---
category: [literature]
status: draft
created: 2026-07-05
updated: 2026-07-05
tags: [paper, cloth-parameters, denim]
---

# Unphased Wrinkles: Estimating cloth elasticity parameters using a frequency-based loss (2022)

arXiv: https://arxiv.org/pdf/2212.08790

## Por qué importa
Reporta valores reales de Young's modulus (E_u, E_v), rigidez a cizalla (μ) y bending (b)
comparando **denim vs algodón vs poliéster**. Es la referencia más directa para calibrar
[[PBD_Cloth_Solver]] hacia mezclilla.

## Hallazgo clave
El método determina correctamente Young's modulus más alto para denim (material rígido)
que para algodón/poliéster, y mayor rigidez a flexión en denim que en cotton/polyester.

## Usado en
- [[PBD_Cloth_Config_Denim]] (nota de configuración derivada)

## Relacionado
- [[PBD_Cloth_Solver]]
