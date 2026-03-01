# Benchmarks v2: Politica de Medicion Reproducible

## 1) Objetivo

Medir rendimiento y robustez del flujo Groebner del skill sin afirmaciones absolutas.

El benchmark se usa para:
- comparar estrategias (`buchberger`, `f4`, `modular`, `degrevlex->fglm`),
- justificar recomendaciones practicas,
- detectar regresiones.

## 2) BenchmarkSpec (contrato)

```json
{
  "case_id": "cyclic6",
  "base_field": "QQ",
  "variables": ["x1", "x2", "x3", "x4", "x5", "x6"],
  "generators": ["..."],
  "goal": "solve_zero_dim",
  "run_matrix": [
    {"algorithm": "buchberger", "ordering": "degrevlex"},
    {"algorithm": "f4", "ordering": "degrevlex"},
    {"algorithm": "buchberger", "ordering": "lex"}
  ],
  "limits": {
    "time_budget_s": 300,
    "memory_budget_mb": 8192
  }
}
```

## 3) Casos minimos requeridos

### Grupo A: 0-dimensional
1. `cyclic6` o variante manejable.
2. `katsura6` o variante manejable.
3. Caso pequeno didactico para `degrevlex -> fglm -> lex`.

### Grupo B: Eliminacion
1. Curva implicita por eliminacion (`y-x^2`, `z-x^3`).
2. Eliminacion por etapas con al menos 4 variables.

### Grupo C: Geometria del ideal
1. Radical en ejemplo con multiplicidad.
2. Primaria en ideal factorizable simple (`<x*y>`).
3. Saturacion con filtro no trivial.

### Grupo D: Estructura
1. Syzygies de un ideal pequeno.
2. Resolucion libre corta.
3. Caso homogeno para Hilbert.

## 4) Metricas obligatorias

Para cada corrida, registrar:
- `elapsed_ms`,
- exito/fallo y tipo de error,
- tamano de GB (numero de generadores),
- grado maximo observado en generadores (si disponible),
- validaciones pasadas (`basis_check`, `normal_form`),
- memoria pico (si el entorno lo permite).

## 5) Reporte comparativo

Formato recomendado por caso:

```text
case_id, algorithm, ordering, elapsed_ms, success, gb_size, max_degree, checks_passed, warnings
```

Minimo:
- 3 repeticiones por configuracion cuando el costo lo permita,
- reportar mediana y desviacion basica de tiempo.

## 6) Criterios de aceptacion

1. Ninguna tecnica recomendada sin al menos un benchmark local reproducible.
2. Cada caso debe terminar con validacion algebraica o explicacion de por que no aplica.
3. Si una estrategia falla, debe existir fallback documentado y medido.
4. Las recomendaciones del skill deben citar datos propios del benchmark, no solo literatura.

## 7) Politica de fairness

- Mantener mismo hardware y limites por bloque comparado.
- Mantener mismo parseo/normalizacion de entrada.
- No mezclar resultados exactos con aproximados sin etiquetado claro.

## 8) Plantilla de metadatos de entorno

```json
{
  "machine": "hostname/model",
  "os": "macOS/Linux",
  "cpu": "...",
  "ram_gb": 32,
  "oscar_version": "...",
  "singular_version": "...",
  "julia_version": "...",
  "date_utc": "YYYY-MM-DD"
}
```

## 9) Claims trazables

- CLM-BENCH-001: rendimiento de Groebner depende fuertemente de ordering, algoritmo y estructura del sistema. [BIB-CLO-IVA, BIB-F4-1999]
- CLM-BENCH-002: comparaciones sin control de entorno pueden inducir conclusiones falsas. [DOC-SINGULAR, DOC-OSCAR-GB]
