# Guia Maestra v1: Uso Integral del Skill Groebner Sistemas Ecuaciones

## Tabla de contenidos

1. Proposito, audiencia y alcance
2. Como invocar la skill
3. Requisitos de entorno y operacion
4. Modelo de trabajo unificado
5. Arbol de decision operativo
6. Catalogo de metodos Groebner (MethodSpec)
7. Ejemplos completos end-to-end
8. Benchmarking y reproducibilidad
9. Troubleshooting integral
10. Buenas practicas academicas
11. Glosario
12. Apendices

---

## 1) Proposito, audiencia y alcance

### 1.1 Proposito

Esta guia explica de extremo a extremo como usar el skill `grobner-sistemas-ecuaciones` para resolver sistemas polinomiales con un flujo:
- operativo,
- reproducible,
- auditable,
- trazable bibliograficamente.

### 1.2 Audiencia

- Investigadores en algebra computacional.
- Usuarios de OSCAR/Singular/Sage que quieren estrategia reproducible.
- Usuarios que necesitan decidir cuando cambiar de simbolico exacto a numerico.

### 1.3 Alcance

Incluye:
- invocacion del skill,
- contratos documentales (`SkillInvocationSpec`, `ProblemSpec`, `RunSpec`, `MethodSpec`, `ValidationSpec`, `ReportSpec`, `CitationSpec`),
- seleccion de metodo,
- validacion,
- troubleshooting,
- ejemplos ejecutables por toolchain.

No incluye:
- implementacion interna de nuevos algoritmos en backend,
- prueba formal completa de todos los teoremas citados.

### 1.4 Principios de esta guia

1. Confiabilidad antes que velocidad.
2. Hechos separados de heuristicas.
3. Sin afirmaciones absolutas sin fuente.
4. Resultado matematico interpretable, no solo salida cruda.

---

## 2) Como invocar la skill

### 2.1 Regla general de activacion

La skill se activa cuando:
1. la nombras explicitamente,
2. el objetivo del mensaje coincide con su dominio (Groebner para sistemas polinomiales).

### 2.2 SkillInvocationSpec

```json
{
  "invoke_mode": "skill_name | natural_language",
  "trigger_text": "texto del usuario",
  "user_goal": "solve_zero_dim | eliminate | radical | primary | saturation | syzygies | resolution | hilbert | membership",
  "context_payload": {
    "problem_spec": "opcional",
    "run_spec": "opcional",
    "notes": "opcional"
  }
}
```

### 2.3 Tres formas practicas de invocacion

#### Forma A: por nombre de skill

Prompt tipo:
- `Usa la skill grobner-sistemas-ecuaciones para resolver este sistema...`

Uso recomendado:
- cuando quieres forzar un flujo completo y auditable.

#### Forma B: por objetivo en lenguaje natural

Prompt tipo:
- `Elimina x de este sistema y dame la relacion implicita en y,z`.
- `Separa ramas de solucion con primaria y valida`.

Uso recomendado:
- cuando priorizas rapidez y la tarea ya encaja naturalmente en la skill.

#### Forma C: por especificacion estructurada

Prompt tipo:
- incluir `ProblemSpec` y opcionalmente `RunSpec`.

Uso recomendado:
- trabajo de investigacion, repetibilidad y benchmarking.

### 2.4 Patrones recomendados de prompt

1. Definir objetivo algebraico explicitamente.
2. Declarar campo base y variables.
3. Declarar restricciones de tiempo/memoria.
4. Pedir validacion minima obligatoria.
5. Pedir citas para claims fuertes.

Ejemplo:

```text
Usa grobner-sistemas-ecuaciones.
Problema: base_field=QQ, vars=[x,y,z], generators=[y-x^2, z-x^3].
Objetivo: eliminate x.
Quiero: toolchain, validacion por normal form, y referencias de claims.
```

### 2.5 Anti-patrones de prompt

1. `Resuelve esto` sin generadores ni campo base.
2. Pedir FGLM sin indicar si el ideal es 0-dimensional.
3. Mezclar objetivos incompatibles sin prioridad.
4. Pedir rendimiento absoluto sin benchmark local.

---

## 3) Requisitos de entorno y operacion

### 3.1 Componentes del stack

1. Shell de trabajo: `zsh` (o `bash` si tu entorno lo requiere).
2. Python para entorno MCP.
3. Julia + OSCAR.
4. SageMath (si se usa en flujos auxiliares).
5. Singular (backend complementario).

### 3.2 Checklist de verificacion de versiones

Ejecuta (adaptando rutas locales):

```bash
zsh --version
python3 --version
julia --version
sage --version
```

Para OSCAR (Julia):

```bash
julia -e 'using Oscar; println("OSCAR_OK")'
```

Para Singular:

```bash
Singular --version
```

### 3.3 Checklist operativo del MCP

1. Activar entorno virtual correcto del servidor MCP.
2. Verificar que el servidor expone herramientas Groebner esperadas.
3. Confirmar que el proceso cliente puede invocarlas.

### 3.4 Politica de higiene de entorno

1. No mezclar entornos con PATH ambiguo.
2. Registrar versiones en reportes de benchmark.
3. Si cambias version de OSCAR/Singular, revalidar recetas criticas.

---

## 4) Modelo de trabajo unificado

### 4.1 ProblemSpec

Contrato principal de entrada.

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

### 4.2 RunSpec

Contrato de ejecucion.

```json
{
  "ordering_start": "degrevlex",
  "algorithm": "buchberger | f4 | modular | hilbert",
  "ordering_target": "lex | degrevlex | weighted",
  "validation_checks": ["basis_check", "normal_form_zero_for_membership"],
  "fallback_policy": ["change_algorithm", "change_ordering", "eliminate_in_stages", "hybrid_bridge"]
}
```

### 4.3 MethodSpec

Contrato documental por metodo.

```json
{
  "method_name": "fglm",
  "preconditions": ["ideal 0-dimensional", "ordering global compatible"],
  "toolchain": ["grobner_basis_compute", "grobner_fglm"],
  "expected_output": "base convertida al ordering objetivo",
  "failure_modes": ["ideal no 0-dimensional", "ordering incompatible"],
  "fallbacks": ["elimination", "primary_decomposition", "keep_degrevlex"]
}
```

### 4.4 ValidationSpec

Contrato de validacion.

```json
{
  "algebraic_checks": [
    "grobner_basis_check",
    "normal_form_membership",
    "precondition_checks"
  ],
  "numeric_checks": [
    "residual_norm_if_hybrid",
    "stability_across_runs"
  ],
  "acceptance_thresholds": {
    "tol_residual": 1e-10,
    "max_warning_count": 3
  }
}
```

### 4.5 ReportSpec

Contrato de salida obligatoria.

```json
{
  "assumptions": ["base field QQ", "ordering degrevlex"],
  "tools_called": ["grobner_ring_create", "grobner_ideal_create", "grobner_basis_compute"],
  "result_interpretation": "texto matematico interpretable",
  "validation": ["basis_check=true", "NF(f)=0 para f en I"],
  "warnings": ["fglm requiere 0-dimensional"],
  "citations": ["BIB-FGLM-1993", "DOC-OSCAR-GB"]
}
```

### 4.6 CitationSpec

```json
{
  "claim_id": "CLM-SYM-008",
  "claim_text": "FGLM requiere ideal 0-dimensional",
  "source_type": "primary",
  "url_or_ref": "BIB-FGLM-1993",
  "confidence": "high"
}
```

---

## 5) Arbol de decision operativo

### 5.1 Preflight obligatorio

Antes de computar:
1. campo base,
2. variables,
3. objetivo,
4. restriccion de exactitud,
5. presupuesto de recursos.

### 5.2 Seleccion de ruta

1. Ruta simbolica exacta (default).
2. Puente numerico si no se requiere certificado exacto y hay costo prohibitivo.

### 5.3 Mapa rapido objetivo -> metodo

1. `solve_zero_dim`: `degrevlex` + GB + `fglm` a `lex`.
2. `eliminate`: `grobner_eliminate` (posible por etapas).
3. `radical`: `grobner_radical`.
4. `primary`: `grobner_primary_decomposition`.
5. `saturation`: `grobner_saturation(I,J)`.
6. `syzygies`: `grobner_syzygies`.
7. `resolution`: `grobner_free_resolution`.
8. `hilbert`: metodo graduado + `grobner_hilbert_series`.
9. `membership`: `grobner_normal_form`.

### 5.4 Claims de soporte

- CLM-SYM-008 (FGLM 0-dimensional) [BIB-FGLM-1993]
- CLM-SYM-012 (saturacion) [BIB-CLO-IVA]
- CLM-HYB-001 (puente homotopico) [DOC-HCJL, BIB-SOMMESE-WAMPLER-2005]

---

## 6) Catalogo de metodos Groebner (MethodSpec)

## 6.1 Buchberger

### MethodSpec

```json
{
  "method_name": "buchberger",
  "preconditions": ["anillo e ideal definidos"],
  "toolchain": ["grobner_basis_compute(algorithm=\"buchberger\")"],
  "expected_output": "GB valida",
  "failure_modes": ["timeout", "coefficient swell"],
  "fallbacks": ["f4", "modular", "elimination_in_stages"]
}
```

### Nota

Metodo baseline robusto y fallback universal. [BIB-BUCHBERGER-2006]

## 6.2 F4

### MethodSpec

```json
{
  "method_name": "f4",
  "preconditions": ["backend con soporte f4", "ideal no trivial"],
  "toolchain": ["grobner_basis_compute(algorithm=\"f4\")"],
  "expected_output": "GB con posible mejora de tiempo",
  "failure_modes": ["no soporte para el dominio", "limitacion de memoria"],
  "fallbacks": ["buchberger", "modular"]
}
```

### Nota

Puede mejorar rendimiento practico via reduccion matricial; no es garantia universal. [BIB-F4-1999]

## 6.3 Modular

### MethodSpec

```json
{
  "method_name": "modular",
  "preconditions": ["dominio compatible", "backend habilitado"],
  "toolchain": ["grobner_basis_compute(algorithm=\"modular\")"],
  "expected_output": "GB con menor swell de coeficientes en casos favorables",
  "failure_modes": ["primos no favorables", "reconstruccion incompleta"],
  "fallbacks": ["buchberger", "f4"]
}
```

### Nota

Util para mitigar crecimiento de coeficientes. [DOC-SINGULAR]

## 6.4 FGLM (cambio de ordering)

### MethodSpec

```json
{
  "method_name": "fglm",
  "preconditions": ["ideal 0-dimensional", "ordering global"],
  "toolchain": ["grobner_basis_compute", "grobner_fglm"],
  "expected_output": "GB en ordering destino (tipicamente lex)",
  "failure_modes": ["ideal no 0-dimensional", "ordering incompatible"],
  "fallbacks": ["elimination", "decompose_then_convert"]
}
```

### Nota

Metodo estandar de conversion en 0-dimensional. [BIB-FGLM-1993]

## 6.5 Eliminacion

### MethodSpec

```json
{
  "method_name": "elimination",
  "preconditions": ["variables a eliminar definidas"],
  "toolchain": ["grobner_basis_compute", "grobner_eliminate"],
  "expected_output": "ideal proyectado en variables remanentes",
  "failure_modes": ["GB muy grande", "seleccion incorrecta de variables"],
  "fallbacks": ["elimination_in_stages", "change_ordering"]
}
```

### Nota

Base para relaciones implicitas y proyecciones. [BIB-CLO-IVA]

## 6.6 Radical

### MethodSpec

```json
{
  "method_name": "radical",
  "preconditions": ["objetivo geometrico"],
  "toolchain": ["grobner_radical"],
  "expected_output": "ideal radical",
  "failure_modes": ["alto costo computacional"],
  "fallbacks": ["simplify_then_radical", "partial_elimination"]
}
```

### Nota

Preserva variedad y elimina multiplicidad algebraica. [BIB-CLO-IVA]

## 6.7 Primaria

### MethodSpec

```json
{
  "method_name": "primary_decomposition",
  "preconditions": ["interes en ramas/componentes"],
  "toolchain": ["grobner_primary_decomposition(algorithm=\"GTZ\"|\"SY\")"],
  "expected_output": "lista de componentes primarias",
  "failure_modes": ["explosion combinatoria", "componentes grandes"],
  "fallbacks": ["switch_algorithm", "pre_elimination", "radical_then_primary"]
}
```

### Nota

GTZ es referencia clasica. [BIB-GTZ-1988]

## 6.8 Saturacion

### MethodSpec

```json
{
  "method_name": "saturation",
  "preconditions": ["ideal principal I y filtro J"],
  "toolchain": ["grobner_saturation(ideal_or_module_id=I, with_id=J)"],
  "expected_output": "I:J^infinity",
  "failure_modes": ["filtro J mal planteado", "iteracion insuficiente"],
  "fallbacks": ["iterative_saturation", "refine_filter"]
}
```

### Nota

Quita componentes sobre `V(J)`. [BIB-CLO-IVA]

## 6.9 Syzygies

### MethodSpec

```json
{
  "method_name": "syzygies",
  "preconditions": ["generadores de ideal/modulo"],
  "toolchain": ["grobner_syzygies"],
  "expected_output": "relaciones entre generadores",
  "failure_modes": ["presentacion demasiado grande"],
  "fallbacks": ["reduce_generators", "work_over_simpler_field"]
}
```

### Nota

Base estructural para resoluciones. [BIB-EISENBUD-1995]

## 6.10 Resoluciones libres

### MethodSpec

```json
{
  "method_name": "free_resolution",
  "preconditions": ["ideal/modulo depurado"],
  "toolchain": ["grobner_free_resolution"],
  "expected_output": "complejo libre truncado o completo",
  "failure_modes": ["longitud excesiva", "recursos insuficientes"],
  "fallbacks": ["shorter_length", "simplify_target"]
}
```

### Nota

Para informacion homologica y complejidad estructural. [BIB-EISENBUD-1995]

## 6.11 Hilbert

### MethodSpec

```json
{
  "method_name": "hilbert",
  "preconditions": ["contexto graduado/homogeneo"],
  "toolchain": ["grobner_basis_compute(algorithm=\"hilbert\")", "grobner_hilbert_series"],
  "expected_output": "serie de Hilbert interpretable",
  "failure_modes": ["no homogeneidad", "pesos incompatibles"],
  "fallbacks": ["homogenize", "use_standard_basis_then_estimate"]
}
```

### Nota

Aplicar solo con precondiciones de graduacion claras. [BIB-CLO-IVA]

---

## 7) Ejemplos completos y ejecutables (end-to-end)

Cada ejemplo sigue:
1. `ProblemSpec`
2. `RunSpec`
3. Toolchain
4. Resultado esperado
5. Validacion
6. Fallback si falla
7. Citas

## 7.1 Ejemplo A: 0-dimensional con FGLM

### ProblemSpec

```json
{
  "base_field": "QQ",
  "variables": ["x", "y"],
  "generators": ["x^2-2", "y-x"],
  "goal": "solve_zero_dim",
  "expected_dimension": "0",
  "constraints": {"exact_certificate_required": true, "time_budget_s": 120}
}
```

### RunSpec

```json
{
  "ordering_start": "degrevlex",
  "algorithm": "buchberger",
  "ordering_target": "lex",
  "validation_checks": ["basis_check", "normal_form_zero_for_membership"],
  "fallback_policy": ["change_algorithm", "eliminate_in_stages"]
}
```

### Toolchain

1. `grobner_ring_create`
2. `grobner_ordering_create(kind="degrevlex")`
3. `grobner_ideal_create`
4. `grobner_basis_compute`
5. `grobner_basis_check`
6. `grobner_ordering_create(kind="lex")`
7. `grobner_fglm`
8. `grobner_normal_form` sobre generadores originales

### Resultado esperado

- base en `lex` con polinomio univariado y ecuacion de sustitucion.

### Validacion minima

- `basis_check=true`
- residuos cero de los generadores originales respecto a GB en `lex`.

### Fallback

- Si `fglm` falla, verificar dimension y pasar a eliminacion parcial.

### Citas

- CLM-SYM-004, CLM-SYM-008 [BIB-FGLM-1993]

## 7.2 Ejemplo B: Eliminacion de variable

### ProblemSpec

```json
{
  "base_field": "QQ",
  "variables": ["x", "y", "z"],
  "generators": ["y-x^2", "z-x^3"],
  "goal": "eliminate",
  "eliminate_vars": ["x"],
  "constraints": {"exact_certificate_required": true, "time_budget_s": 120}
}
```

### RunSpec

```json
{
  "ordering_start": "degrevlex",
  "algorithm": "f4",
  "ordering_target": "degrevlex",
  "validation_checks": ["normal_form_zero_for_membership"],
  "fallback_policy": ["change_algorithm", "eliminate_in_stages"]
}
```

### Toolchain

1. `grobner_ring_create`
2. `grobner_ideal_create`
3. `grobner_basis_compute(algorithm="f4")`
4. `grobner_eliminate(vars_or_indices=["x"])`

### Resultado esperado

- relacion eliminada equivalente a `y^3-z^2`.

### Validacion minima

- sustituir parametrizacion y verificar anulacion.

### Fallback

- Si `f4` no aplica, volver a `buchberger`.
- Si se dispara complejidad, eliminar por etapas.

### Citas

- CLM-SYM-006 [BIB-F4-1999]
- CLM-SYM-009 [BIB-CLO-IVA]

## 7.3 Ejemplo C: Radical + primaria

### ProblemSpec

```json
{
  "base_field": "QQ",
  "variables": ["x", "y"],
  "generators": ["x*y"],
  "goal": "primary",
  "constraints": {"exact_certificate_required": true, "time_budget_s": 120}
}
```

### RunSpec

```json
{
  "ordering_start": "degrevlex",
  "algorithm": "buchberger",
  "ordering_target": "degrevlex",
  "validation_checks": ["basis_check"],
  "fallback_policy": ["switch_primary_algorithm", "pre_elimination"]
}
```

### Toolchain

1. `grobner_ring_create`
2. `grobner_ideal_create`
3. `grobner_radical`
4. `grobner_primary_decomposition(algorithm="GTZ")`

### Resultado esperado

- componentes equivalentes a `(x)` y `(y)` en el ejemplo.

### Validacion minima

- la interseccion de componentes recupera ideal de partida en este caso.

### Fallback

- cambiar `GTZ <-> SY`.

### Citas

- CLM-SYM-010 [BIB-CLO-IVA]
- CLM-SYM-011 [BIB-GTZ-1988]

## 7.4 Ejemplo D: Saturacion para remover componente espuria

### ProblemSpec

```json
{
  "base_field": "QQ",
  "variables": ["x", "y"],
  "generators": ["x*y"],
  "goal": "saturation",
  "constraints": {"exact_certificate_required": true, "time_budget_s": 120}
}
```

### RunSpec

```json
{
  "ordering_start": "degrevlex",
  "algorithm": "buchberger",
  "ordering_target": "degrevlex",
  "validation_checks": ["normal_form_zero_for_membership"],
  "fallback_policy": ["iterative_saturation", "refine_filter"]
}
```

### Toolchain

1. `grobner_ring_create`
2. `grobner_ideal_create` para `I=<x*y>`
3. `grobner_ideal_create` para `J=<x>`
4. `grobner_saturation(I,J)`

### Resultado esperado

- resultado equivalente a `<y>`.

### Validacion minima

- verificar condicion de saturacion con potencia de `x`.

### Fallback

- activar modo iterativo o redefinir filtro.

### Citas

- CLM-SYM-012 [BIB-CLO-IVA]

## 7.5 Ejemplo E: Puente numerico controlado (homotopia)

### ProblemSpec

```json
{
  "base_field": "QQ",
  "variables": ["x", "y", "z"],
  "generators": ["x^6+y^6+z^6-1", "x^3-y^2+z", "x*y*z-1/10"],
  "goal": "solve_zero_dim",
  "constraints": {
    "exact_certificate_required": false,
    "time_budget_s": 60,
    "memory_budget_mb": 4096
  }
}
```

### RunSpec

```json
{
  "ordering_start": "degrevlex",
  "algorithm": "buchberger",
  "ordering_target": "degrevlex",
  "validation_checks": ["residual_norm_if_hybrid"],
  "fallback_policy": ["hybrid_bridge", "adjust_tolerances", "return_symbolic_subproblem"]
}
```

### Toolchain

1. Intento simbolico breve (pre-simplificacion).
2. Exportar a solver de homotopia.
3. Resolver, filtrar, validar residual.
4. Reportar tolerancias y semilla.

### Resultado esperado

- conjunto de soluciones aproximadas con residual bajo umbral.

### Validacion minima

- `max_residual_observed <= tol_residual`.

### Fallback

- cambiar tipo de homotopia,
- ajustar tolerancias,
- volver a subproblema simbolico.

### Citas

- CLM-HYB-001, CLM-HYB-002, CLM-HYB-003 [DOC-HCJL]

---

## 8) Benchmarking y reproducibilidad

### 8.1 Metricas obligatorias

Registrar por corrida:
1. `elapsed_ms`
2. `success/failure`
3. tamano de base
4. grado maximo observado (si aplica)
5. checks de validacion
6. warnings

### 8.2 Formato recomendado

```text
case_id,algorithm,ordering,elapsed_ms,success,gb_size,max_degree,checks_passed,warnings
```

### 8.3 Entorno minimo a registrar

```json
{
  "os": "macOS/Linux",
  "python_version": "...",
  "julia_version": "...",
  "sage_version": "...",
  "oscar_version": "...",
  "singular_version": "...",
  "date_utc": "YYYY-MM-DD"
}
```

### 8.4 Recomendacion metodologica

1. comparar estrategias bajo mismos limites,
2. al menos 3 repeticiones cuando sea viable,
3. reportar mediana y variacion.

### 8.5 Citas

- CLM-BENCH-001 [BIB-F4-1999]
- CLM-BENCH-002 [DOC-OSCAR-GB]

---

## 9) Troubleshooting integral

## 9.1 Errores de precondicion

1. FGLM falla.
- Causa tipica: ideal no 0-dimensional.
- Accion: verificar dimension esperada; usar eliminacion o descomposicion.

2. Hilbert falla.
- Causa tipica: falta homogeneidad o pesos validos.
- Accion: homogeneizar o cambiar objetivo.

## 9.2 Timeouts y memoria

1. Reducir alcance por eliminacion en etapas.
2. Cambiar algoritmo `f4/modular -> buchberger` o viceversa segun caso.
3. Reducir numero de generadores mediante pre-simplificacion.
4. Si no hay exigencia exacta, activar puente numerico.

## 9.3 Ordering inadecuado

1. `lex` temprano puede inflar complejidad.
2. Usar `degrevlex` para computo inicial.
3. Convertir a `lex` solo al final via FGLM cuando aplique.

## 9.4 Inconsistencias de salida

1. Confirmar que `base_field` y parseo no cambiaron entre pasos.
2. Confirmar ordering fijo durante validacion.
3. Repetir `normal_form` en polinomios control.

## 9.5 Tabla rapida fallo -> accion

| fallo | probable causa | accion primaria | fallback secundario |
|---|---|---|---|
| `fglm` error | ideal no 0-dimensional | revisar objetivo/dimension | eliminacion o primaria |
| GB demasiado grande | ordering/algoritmo no favorable | cambiar ordering | dividir problema |
| hilbert no coherente | ideal no homogeno | homogeneizar | cambiar a ruta no-graduada |
| saturacion no limpia | filtro `J` mal definido | redefinir `J` | saturacion iterativa |
| residual alto en numerico | tolerancia/metodo | ajustar tolerancias | cambiar homotopia |

---

## 10) Buenas practicas academicas

### 10.1 Disciplina de afirmaciones

Separar siempre:
1. hecho matematico,
2. heuristica practica,
3. opinion de rendimiento.

### 10.2 Regla de citacion

- Cada claim fuerte debe tener `BIB-*` o `DOC-*`.
- Reusar claves de `claims_register.md`.
- Evitar frases absolutas sin evidencia local.

### 10.3 Trazabilidad minima en reportes

Todo reporte serio debe incluir:
1. supuestos algebraicos,
2. toolchain usado,
3. validacion ejecutada,
4. advertencias,
5. citas.

### 10.4 Citas nucleares sugeridas por tema

1. GB/Buchberger: `BIB-BUCHBERGER-2006`
2. F4/F5: `BIB-F4-1999`, `BIB-F5-2002`
3. FGLM: `BIB-FGLM-1993`
4. Primaria: `BIB-GTZ-1988`
5. Puente numerico: `DOC-HCJL`, `BIB-SOMMESE-WAMPLER-2005`

---

## 11) Glosario

1. `GB`: Base de Groebner.
2. `LT(f)`: termino lider de `f`.
3. `LM(f)`: monomio lider de `f`.
4. `NF(f,G)`: normal form de `f` respecto a `G`.
5. `0-dimensional`: ideal con numero finito de soluciones sobre clausura algebraica.
6. `FGLM`: algoritmo de cambio de ordering para caso 0-dimensional.
7. `Saturacion`: `I:J^infinity`.
8. `Primary decomposition`: descomposicion en componentes primarias.
9. `Residual norm`: medida numerica de `||f(x)||` en soluciones aproximadas.

---

## 12) Apendices

## Apendice A: Plantilla integral de solicitud

```text
Usa grobner-sistemas-ecuaciones.
ProblemSpec:
{...}
RunSpec:
{...}
Quiero salida con ReportSpec completo y citas.
```

## Apendice B: Plantilla integral de reporte

```json
{
  "assumptions": ["..."],
  "tools_called": ["..."],
  "result_interpretation": "...",
  "validation": ["..."],
  "warnings": ["..."],
  "citations": ["BIB-...", "DOC-..."]
}
```

## Apendice C: Matriz de cobertura (fase de auditoria)

| seccion maestro | fuente principal reutilizada |
|---|---|
| arbol de decision | `references/decision_tree.md` |
| catalogo de metodos | `references/cookbook.md`, `references/symbolic_core.md` |
| puente numerico | `references/hybrid_bridge.md` |
| benchmarking | `references/benchmarks.md` |
| citas y claims | `references/bibliography_primary.md`, `references/claims_register.md` |

## Apendice D: Fuentes clave

- `BIB-BUCHBERGER-2006`
- `BIB-F4-1999`
- `BIB-F5-2002`
- `BIB-FGLM-1993`
- `BIB-GTZ-1988`
- `BIB-CLO-IVA`
- `DOC-OSCAR-GB`
- `DOC-SINGULAR`
- `DOC-HCJL`

