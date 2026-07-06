---
category: [simulation]
status: validated
created: 2026-07-06
updated: 2026-07-06
tags: [genesis, installation, backends, requirements, cuda, pytorch]
---

# Genesis — Instalación y Requisitos

Fuente: `README.md` del repo `Genesis-Embodied-AI/Genesis` (rama `main`).

## Requisitos mínimos

| Componente | Versión requerida |
|-----------|------------------|
| **Python** | 3.10 – 3.13 (< 3.14) |
| **PyTorch** | Instalar antes de genesis-world según plataforma |
| **CUDA** | Determinado por la versión de PyTorch elegida (ej. CUDA 12.6 → `cu126`) |

Genesis no impone una versión de CUDA directamente — se hereda de PyTorch.

## Backends soportados (compilador Quadrants)

El compilador **Quadrants** es el motor JIT de Genesis. Se incluye automáticamente con `genesis-world`. Soporta:

| Backend | Hardware | Notas |
|---------|----------|-------|
| **NVIDIA CUDA** | GPU NVIDIA | Estable; recomendado para entrenamiento RL |
| **AMD ROCm (HIP)** | GPU AMD | Usar `Dockerfile.amdgpu` |
| **Apple Metal** | Apple Silicon (M1/M2/M3) | Estable; instalación simple (`uv pip install torch`) |
| **Vulkan** | GPU genérica | Soporte general |
| **x86 CPU** | Intel/AMD | Sin GPU; lento para PBD cloth |
| **ARM64 CPU** | Apple Silicon, RPi, etc. | — |

Instalación standalone del compilador: `pip install quadrants`.

## Métodos de instalación

### Opción 1 — pip (release estable, recomendado)

```bash
# 1. Instalar PyTorch primero
#    NVIDIA CUDA 12.6:
pip install torch --index-url https://download.pytorch.org/whl/cu126
#    CPU:
pip install torch --index-url https://download.pytorch.org/whl/cpu
#    Apple Silicon:
pip install torch

# 2. Instalar Genesis
pip install genesis-world
```

### Opción 2 — desde fuente (para features experimentales o contribuir)

```bash
git clone https://github.com/Genesis-Embodied-AI/genesis-world.git
cd genesis-world
pip install -e ".[dev]"
# Tras hacer git pull / cambiar de rama, repetir el install:
pip install -e ".[dev]"
```

### Opción 3 — uv package manager

```bash
git clone https://github.com/Genesis-Embodied-AI/genesis-world.git
cd genesis-world
uv sync
uv pip install torch --index-url https://download.pytorch.org/whl/cu126  # NVIDIA
```

### Opción 4 — versión de desarrollo sin clonar

```bash
pip install git+https://github.com/Genesis-Embodied-AI/genesis-world.git
```

## Dependencias opcionales

| Feature | Instalación | Restricciones |
|---------|------------|---------------|
| **IPC solver** (`FEM.Cloth`, `IPCCouplerOptions`) | `pip install pyuipc` | Linux/Windows x86 + NVIDIA GPU únicamente |
| **Nyx renderer** (ray tracing avanzado) | `pip install gs-nyx` | — |

> **Implicación directa para el proyecto:** usar `gs.materials.PBD.Cloth` (plan actual
> en [[PBD_Cloth_Config_Denim]]) **no requiere** `pyuipc`. Si en algún punto se migra a
> `gs.materials.FEM.Cloth` (parámetros E, nu directos desde [[Unphased_Wrinkles_2022]]),
> entonces `pip install pyuipc` es obligatorio y restringe el entorno a Linux/Windows NVIDIA.

## Docker

Dockerfile estándar disponible en el repo. Para AMD/ROCm: `Dockerfile.amdgpu`.

## Verificación rápida

```python (genesis_env)
import genesis as gs
gs.init(backend=gs.gpu)   # alternativos: gs.cpu, gs.metal
print(gs.__version__)
```

## Backends y rendimiento para cloth RL

| Backend | Uso recomendado |
|---------|----------------|
| `gs.gpu` (CUDA) | Entrenamiento RL con vectorización (target: 1,024 envs paralelos) |
| `gs.cpu` | Debug y desarrollo; demasiado lento para PBD a escala |
| `gs.metal` | Desarrollo en Mac; sin vectorización masiva verificada para PBD |

El target de rendimiento para [[Gym_Wrapper_Genesis]] es ~4,000 SPS con 1,024 envs
(referencia: [[ManiSkill_HAB_2024]]). Solo alcanzable con backend CUDA.

## Relacionado
- [[Genesis_Simulator]] — arquitectura del motor; qué solvers están disponibles
- [[Genesis_Config_System]] — todas las opciones de configuración y materiales
- [[PBD_Cloth_Solver]] — requiere solo `genesis-world` + PyTorch (sin extras)
- [[Gym_Wrapper_Genesis]] — target de rendimiento (CUDA obligatorio para 1,024 envs)
