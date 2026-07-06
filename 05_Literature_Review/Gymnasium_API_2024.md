---
category: [literature, training]
status: validated
created: 2026-07-06
updated: 2026-07-06
tags: [paper, gymnasium, gym-api, vectorization, spec, reference]
---

# Gymnasium: A Standardized Interface for Reinforcement Learning Environments (2024)

arXiv: https://arxiv.org/abs/2407.17032
Autores: Mark Towers et al. (Farama Foundation, U. Southampton, Meta AI FAIR, MILA)
Venue: preprint arXiv 2407.17032v4, 2024 — >18M descargas acumuladas

> **Status: validated** — spec oficial del estándar que deben implementar
> [[Gym_Wrapper_Genesis]], [[RoboVerse_2025]] (Handler+Env) y [[ManiSkill_HAB_2024]].

## Por qué importa para el proyecto

Define la **interfaz exacta** que debe exponer [[Gym_Wrapper_Genesis]]. Documenta la
distinción `terminated`/`truncated` (crítica para bootstrap correcto en RL), los tres
modos de vectorización, y los requisitos mínimos para registrar un env custom.
Sucesor mantenido de OpenAI Gym (abandonado en oct. 2022).

## Especificación reset()

```python
observation, info = env.reset(seed: int | None = None,
                              options: dict | None = None)
```

| Argumento | Tipo | Semántica |
|-----------|------|-----------|
| `seed` | int o None | Siembra toda fuente de aleatoriedad del env — Genesis RNG incluido |
| `options` | dict o None | Config adicional (ej. pose inicial de la tela, variante de material) |
| → `observation` | conforme a `observation_space` | Estado inicial del episodio |
| → `info` | dict | Metadata auxiliar; no usar para lógica crítica de entrenamiento |

## Especificación step()

```python
observation, reward, terminated, truncated, info = env.step(action)
```

| Retorno | Tipo | Semántica |
|---------|------|-----------|
| `observation` | conforme a `observation_space` | Estado siguiente |
| `reward` | float | Señal escalar de recompensa |
| `terminated` | bool | True si el env llegó a un estado terminal **por la tarea** |
| `truncated` | bool | True si el episodio se cortó por razón **externa** (ej. TimeLimit) |
| `info` | dict | Metadata auxiliar |

## Distinción terminated vs truncated (crítica para RL)

**`terminated = True`**: la tarea terminó — éxito o fallo definitivo. El agente no
puede alcanzar más estados desde aquí. Valor futuro = 0.

**`truncated = True`**: el episodio se cortó artificialmente (agotó pasos, fallo del
sistema, etc.). La dinámica *podría* continuar. El agente *debe* estimar el valor
futuro restante (bootstrap).

**Impacto en el cálculo del retorno:**

```python
# Cálculo correcto con Gymnasium:
target = reward + gamma * (1 - terminated) * V(next_obs)
#                          ^^^^^^^^^^^^^^^^^
#  truncated no cancela el bootstrap — terminated sí

# Error clásico con OpenAI Gym (done único):
target = reward + gamma * (1 - done) * V(next_obs)
#  done = terminated OR truncated → cancela bootstrap en truncation → sesgo
```

**Por qué OpenAI Gym lo hacía mal:** `done` era la unión de ambos casos, lo que
cancelaba el bootstrap en episodios truncados — sesgo sistemático en el valor aprendido.
Gymnasium los separa y hace el manejo correcto **obligatorio** por la firma de la API.

**Para [[Gym_Wrapper_Genesis]]:**
- `terminated` ← cobertura de tela ≥ umbral (éxito) o tiempo ≤ 0 sin cobertura (fallo).
- `truncated` ← `TimeLimit` wrapper de Gymnasium (no implementar en `step()`).

## Especificación render()

```python
frame = env.render()  # render_mode declarado en __init__
```

| `render_mode` | Retorna | Uso |
|--------------|---------|-----|
| `"human"` | None | Ventana gráfica en tiempo real |
| `"rgb_array"` | ndarray (H, W, 3) uint8 | Grabación de video, depuración |
| `"ansi"` | str | Entornos text-based |

Se llama *después* de `step()`, típicamente en el loop de episodio. No es necesario
llamar durante entrenamiento distribuido.

## Wrappers de vectorización

### SyncVectorEnv — secuencial, bajo overhead

```python
env = SyncVectorEnv([lambda: make_cloth_env(i) for i in range(N)])
obs, info = env.reset()           # shape: (N, *obs_shape)
obs, rew, term, trunc, info = env.step(actions)  # actions shape: (N, *act_shape)
```

Ejecuta cada sub-env en secuencia; sin subprocesos. Adecuado cuando el env es rápido
(ej. Genesis GPU-accelerated) — el overhead de subprocesos de `Async` sería mayor.

### AsyncVectorEnv — subprocesos paralelos

```python
env = AsyncVectorEnv([lambda: make_cloth_env(i) for i in range(N)])
```

Cada sub-env en su propio proceso. Mejor cuando el env es lento en CPU. **No recomendado
con Genesis GPU**: el paralelismo ya está en el motor; subprocesos añaden latencia.

### Auto-reset al fin de episodio (comportamiento crítico)

| Modo | Cuándo resetea | Obs final | Recomendación |
|------|---------------|-----------|--------------|
| `next-step` (default Gymnasium) | Paso siguiente al terminal | En `info["final_observation"]` | **Usar este** |
| `same-step` (default OpenAI Gym) | Mismo paso del terminal | Retornada directamente | Evitar — confuso |
| `disabled` | Manual con máscara | N/A | Cuando se controla reset externamente |

En `next-step` mode, cuando `terminated[i] or truncated[i]` es True, la obs retornada
es ya la del nuevo episodio; la obs final del episodio anterior está en
`info["final_observation"][i]`.

## Spaces: tipos disponibles

```python
# Observación continua (partículas PBD de tela):
obs_space = Box(low=-np.inf, high=np.inf, shape=(N_particles, 3), dtype=np.float32)

# Acción continua (pose del gripper):
act_space = Box(low=-1.0, high=1.0, shape=(6,), dtype=np.float32)

# Observación multi-modal (estado + imagen):
obs_space = Dict({
    "particles": Box(...),
    "depth": Box(low=0, high=255, shape=(128, 128, 1), dtype=np.uint8),
    "joints": Box(...),
})
```

Tipos disponibles: `Box`, `Discrete`, `MultiDiscrete`, `MultiBinary`, `Dict`, `Tuple`,
`Text`, `Sequence`, `Graph`, `OneOf`.

## Requisitos mínimos para env custom

```python
import gymnasium as gym
from gymnasium import spaces
import numpy as np

class GenesisClothEnv(gym.Env):
    metadata = {"render_modes": ["rgb_array"], "render_fps": 30}

    def __init__(self, render_mode=None):
        super().__init__()
        # 1. Declarar espacios SIEMPRE antes de reset/step
        self.observation_space = spaces.Dict({
            "particles": spaces.Box(-np.inf, np.inf, (N_PARTICLES, 3), np.float32),
            "joints":    spaces.Box(-np.pi, np.pi, (N_JOINTS,), np.float32),
        })
        self.action_space = spaces.Box(-1.0, 1.0, (6,), np.float32)
        self.render_mode = render_mode
        # self._genesis_scene = ...  inicializar Genesis aquí

    def reset(self, seed=None, options=None):
        super().reset(seed=seed)   # siembra self.np_random
        # reinicializar escena Genesis con self.np_random para reproducibilidad
        obs = self._get_obs()
        info = {}
        return obs, info

    def step(self, action):
        # aplicar acción al gripper en Genesis
        # avanzar simulación PBD
        obs = self._get_obs()
        reward = self._compute_reward()
        terminated = bool(self._coverage() >= COVERAGE_THRESHOLD)
        truncated = False   # dejar a TimeLimit wrapper
        info = {}
        return obs, reward, terminated, truncated, info

    def render(self):
        if self.render_mode == "rgb_array":
            return self._genesis_render_rgb()

    def close(self):
        super().close()
        # liberar recursos Genesis

# Registro:
gym.register(id="GenesisCloth-v0",
             entry_point="my_module:GenesisClothEnv",
             max_episode_steps=500)
```

**Regla crítica:** No implementar límite de pasos en `step()`. Usar `TimeLimit` wrapper
(vía `max_episode_steps` en `register()`) para que `truncated` se gestione
automáticamente y el bootstrap sea correcto.

## FuncEnv: API funcional para JAX/GPU

```python
class FuncEnv:
    def initial(rng) -> state: ...
    def transition(state, action, rng) -> state: ...
    def observation(state) -> obs: ...
    def reward(state, action, next_state) -> float: ...
    def terminal(state) -> bool: ...
```

Diseño sin estado interno — compatible con `jax.vmap` y `jax.jit` para vectorización
masiva en GPU. Referencia: Brax, Pgx, Jumanji lo usan para miles de envs en paralelo.
Alternativa si Genesis expone una API funcional/JAX-compatible.

## Diferencias clave vs. OpenAI Gym

| Aspecto | Gymnasium | OpenAI Gym |
|---------|-----------|-----------|
| Terminación/truncación | `terminated`, `truncated` (separados) | `done` (unión) |
| Mantenimiento | Activo (Farama Foundation) | Abandonado oct. 2022 |
| Vectorización | `Sync/AsyncVectorEnv` + 3 modos auto-reset | Básico |
| Spaces nuevos | Text, Sequence, Graph, OneOf | No disponibles |
| API funcional | FuncEnv (JAX) | No disponible |

## Relacionado
- [[Gym_Wrapper_Genesis]] — implementación concreta de esta spec para Genesis + tela
- [[RoboVerse_2025]] — Handler+Env sigue esta spec en la capa Env
- [[ManiSkill_HAB_2024]] — implementación GPU con SyncVectorEnv, 4,109 SPS
- [[SoftGym_CoRL2020]] — primer benchmark cloth con esta interfaz (basado en Gym, no Gymnasium)
