---
name: grobner-sistemas-ecuaciones
description: Resolver sistemas de ecuaciones polinomiales con Groebner en oscar-mcp, con flujo reproducible (anillos, ideales, GB, eliminacion, FGLM, radical, primaria, saturacion, syzygies, resoluciones) y criterio explicito para puente numerico por homotopia.
---

# Groebner Sistemas Ecuaciones (v2)

## Objetivo

Resolver sistemas polinomiales con un flujo **operativo + auditable**:
- solucion efectiva del problema del usuario,
- validacion algebraica minima,
- trazabilidad de decisiones (ordering, algoritmo, fallback, fuentes).

## Cuando activar este skill

Activa este skill cuando el usuario pida:
- resolver sistemas polinomiales (exacto o mixto),
- eliminar variables o parametros,
- convertir ordenamientos (`degrevlex -> lex`) para lectura triangular,
- analizar estructura de ideal (radical, primaria, saturacion, syzygies, resoluciones),
- decidir entre via simbolica y puente numerico por homotopia.

## Prioridades de ejecucion

1. `OSCAR-first` para computo simbolico exacto.
2. Validacion algebraica antes de concluir.
3. Puente numerico solo cuando el arbol de decision lo recomiende.
4. Reporte final con `ReportSpec` completo.

## Contratos obligatorios (interfaces de trabajo)

Usa estos esquemas en cada caso, aunque sea de forma resumida.

### `ProblemSpec`

```json
{
  "base_field": "QQ | GF(p) | ZZ",
  "variables": ["x", "y", "z"],
  "generators": ["x^2-y", "x^3-z"],
  "goal": "solve_zero_dim | eliminate | radical | primary | saturation | syzygies | resolution | hilbert | membership",
  "eliminate_vars": ["x"],
  "expected_dimension": "unknown | 0 | >=1",
  "constraints": {
    "exact_certificate_required": true,
    "time_budget_s": 120,
    "memory_budget_mb": 4096
  }
}
```

### `RunSpec`

```json
{
  "ordering_start": "degrevlex",
  "algorithm": "buchberger | f4 | modular | hilbert",
  "ordering_target": "lex | degrevlex | weighted",
  "validation_checks": ["basis_check", "normal_form_zero_for_membership"],
  "fallback_policy": ["change_algorithm", "change_ordering", "eliminate_in_stages", "hybrid_bridge"]
}
```

### `ReportSpec`

```json
{
  "assumptions": ["base field QQ", "ordering degrevlex"],
  "tools_called": ["grobner_ring_create", "grobner_ideal_create", "grobner_basis_compute"],
  "result_interpretation": "texto matematico breve",
  "validation": ["grobner_basis_check=true", "normal_form(...)=0"],
  "warnings": ["FGLM requiere ideal 0-dimensional"],
  "citations": ["BIB-FGLM-1993", "DOC-OSCAR-GB"]
}
```

### `CitationSpec`

```json
{
  "claim_id": "CLM-EXAMPLE-001",
  "claim_text": "FGLM requiere ideal 0-dimensional y ordenamientos globales",
  "source_type": "primary | official_doc",
  "url_or_ref": "https://docs.oscar-system.org/stable/CommutativeAlgebra/GroebnerBases/groebner_bases/",
  "confidence": "high"
}
```

## Flujo operativo minimo (obligatorio)

1. Normalizar `ProblemSpec`.
- Fijar campo, variables, objetivo y restricciones.

2. Elegir ruta en `references/decision_tree.md`.
- Simbolica exacta o puente numerico.

3. Construir objetos algebraicos.
- `grobner_ring_create` -> `ring_id`.
- `grobner_ordering_create` si aplica.
- `grobner_ideal_create` o `grobner_module_create`.

4. Computo central de base.
- `grobner_basis_compute` con algoritmo explicito.
- `grobner_basis_check` cuando aplique.

5. Tecnica por objetivo.
- `eliminate`, `fglm`, `radical`, `primary_decomposition`, `saturation`, `syzygies`, `free_resolution`, `hilbert_series`.

6. Validacion.
- `grobner_normal_form` para pertenencia y chequeos minimos.
- Confirmar precondiciones (ej. FGLM 0-dimensional).

7. Reporte con `ReportSpec`.
- Sin salida cruda sin interpretar.
- Incluir advertencias y fallback usados.

## Reglas de calidad de respuesta

- Separar claramente:
  - hecho matematico,
  - heuristica practica,
  - opinion de rendimiento.
- Evitar afirmaciones absolutas sin fuente.
- Usar terminologia consistente: `degrevlex`, `lex`, 0-dimensional, radical, primaria.
- Si un claim es fuerte, agregar cita de `references/bibliography_primary.md`.

## Fallback estandar

1. `f4/modular -> buchberger` si falla o no aplica.
2. `lex directo -> degrevlex + fglm` para 0-dimensional.
3. eliminacion en etapas para bajar complejidad.
4. radical/primaria para separar ramas antes de seguir.
5. puente numerico (homotopia) solo si no se exige certificado exacto o si el costo simbolico es prohibitivo.

## Referencias internas (carga progresiva)

- Guia maestra (academica + operativa):
  - `references/master_guide.md`

- Guia rapida (uso diario):
  - `references/quickstart_cheatsheet.md`

- Arbol de decision: `references/decision_tree.md`
- Recetas operativas: `references/cookbook.md`
- Marco teorico simbolico: `references/symbolic_core.md`
- Criterio de puente numerico: `references/hybrid_bridge.md`
- Politica de benchmark: `references/benchmarks.md`
- Bibliografia primaria/oficial: `references/bibliography_primary.md`
- Registro claim -> evidencia: `references/claims_register.md`
