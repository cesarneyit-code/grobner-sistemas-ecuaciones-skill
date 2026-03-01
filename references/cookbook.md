# Cookbook v2: Recetas Reproducibles para Sistemas Polinomiales

## Plantilla unica por receta

Cada receta debe contener:
1. Objetivo.
2. Preconditions.
3. Ejemplo minimo (`ProblemSpec`).
4. Toolchain.
5. Salida esperada.
6. Validacion.
7. Fallos tipicos y fallback.

---

## Receta 1: Base de Groebner de trabajo + chequeo

### Objetivo
Construir una base operativa confiable para iniciar cualquier flujo.

### Preconditions
- Campo base definido.
- Generadores parseables.

### Ejemplo minimo
```json
{
  "base_field": "QQ",
  "variables": ["x", "y"],
  "generators": ["x^2-y", "x*y-1"],
  "goal": "membership"
}
```

### Toolchain
1. `grobner_ring_create`
2. `grobner_ordering_create(kind="degrevlex")`
3. `grobner_ideal_create`
4. `grobner_basis_compute(algorithm="buchberger")`
5. `grobner_basis_check`

### Salida esperada
- `gb_id` valido.
- `grobner_basis_check=true`.

### Validacion
- Al menos un polinomio de la GB reduce a 0 contra la propia GB.

### Fallos tipicos y fallback
- Parseo invalido -> normalizar generadores.
- Timeout -> probar `f4` o reducir variables.

---

## Receta 2: Membresia por normal form

### Objetivo
Decidir si un polinomio pertenece al ideal.

### Preconditions
- Ideal o GB disponible con ordering fijo.

### Ejemplo minimo
```json
{
  "base_field": "QQ",
  "variables": ["x", "y"],
  "generators": ["x^2-y", "x*y-1"],
  "goal": "membership"
}
```
Polinomio a probar: `x^2*y-1`.

### Toolchain
1. Construir ideal/GB.
2. `grobner_normal_form(poly_or_list=["x^2*y-1"], ideal_id_or_gb_id=...)`.

### Salida esperada
- Residuo `0` para pertenencia verdadera.

### Validacion
- Repetir con un polinomio no perteneciente y verificar residuo no nulo.

### Fallos tipicos y fallback
- Residuos inconsistentes por ordering distinto -> fijar ordering explicito en todo el flujo.

---

## Receta 3: Eliminacion de variable (curva implicita)

### Objetivo
Eliminar parametro y obtener relacion implicita.

### Preconditions
- Variable a eliminar identificada.

### Ejemplo minimo
```json
{
  "base_field": "QQ",
  "variables": ["x", "y", "z"],
  "generators": ["y-x^2", "z-x^3"],
  "goal": "eliminate",
  "eliminate_vars": ["x"]
}
```

### Toolchain
1. Crear anillo/ideal.
2. `grobner_basis_compute` en ordering eficiente.
3. `grobner_eliminate(ideal_id, vars_or_indices=["x"])`.

### Salida esperada
- Ideal eliminado que contiene `y^3-z^2` (o generador equivalente).

### Validacion
- Sustituir `y=x^2`, `z=x^3` en la relacion eliminada y verificar anulacion.

### Fallos tipicos y fallback
- Base enorme -> eliminar en etapas.
- Resultado vacio inesperado -> revisar indices/variables.

---

## Receta 4: Resolver 0-dimensional con FGLM

### Objetivo
Pasar de orden eficiente a `lex` para lectura mas triangular.

### Preconditions
- Ideal 0-dimensional.
- Ordenamientos globales compatibles.

### Ejemplo minimo
```json
{
  "base_field": "QQ",
  "variables": ["x", "y"],
  "generators": ["x^2-2", "y-x"],
  "goal": "solve_zero_dim",
  "expected_dimension": "0"
}
```

### Toolchain
1. GB en `degrevlex`.
2. Crear ordering destino `lex`.
3. `grobner_fglm`.

### Salida esperada
- Base en `lex` con forma triangular (ej. univariado + ecuacion de sustitucion).

### Validacion
- `grobner_normal_form` de cada generador original respecto a la nueva base da 0.

### Fallos tipicos y fallback
- `fglm` falla por no 0-dimensional -> ir a eliminacion parcial o descomposicion.

---

## Receta 5: Radical para quitar multiplicidades

### Objetivo
Trabajar con variedad geometrica sin multiplicidades algebraicas.

### Preconditions
- Interesa geometria de soluciones, no estructura multiplicativa.

### Ejemplo minimo
```json
{
  "base_field": "QQ",
  "variables": ["x", "y"],
  "generators": ["x^2", "y-x"],
  "goal": "radical"
}
```

### Toolchain
1. Crear ideal `I`.
2. `grobner_radical(I)`.

### Salida esperada
- Ideal radical equivalente geometricamente a `I`.

### Validacion
- Puntos de la variedad coinciden en ejemplos pequenos.

### Fallos tipicos y fallback
- Alto costo -> combinar con eliminacion previa.

---

## Receta 6: Descomposicion primaria en ramas

### Objetivo
Separar el sistema en componentes primarias.

### Preconditions
- Interesa estructura por ramas.

### Ejemplo minimo
```json
{
  "base_field": "QQ",
  "variables": ["x", "y"],
  "generators": ["x*y"],
  "goal": "primary"
}
```

### Toolchain
1. `grobner_primary_decomposition(ideal_id, algorithm="GTZ")`.
2. Opcional: comparar con `algorithm="SY"`.

### Salida esperada
- Componentes equivalentes a `(x)` y `(y)` (hasta equivalencia de generadores).

### Validacion
- Interseccion de componentes recupera el ideal de partida en el ejemplo.

### Fallos tipicos y fallback
- Explota complejidad -> usar eliminacion y luego primaria por bloques.

---

## Receta 7: Saturacion para excluir componentes espurias

### Objetivo
Quitar componentes soportadas en una subvariedad prohibida.

### Preconditions
- Ideal filtro `J` disponible.

### Ejemplo minimo
```json
{
  "base_field": "QQ",
  "variables": ["x", "y"],
  "generators": ["x*y"],
  "goal": "saturation"
}
```
con filtro `J=<x>`.

### Toolchain
1. Crear `I=<x*y>` y `J=<x>`.
2. `grobner_saturation(ideal_or_module_id=I, with_id=J)`.

### Salida esperada
- Resultado equivalente a `<y>`.

### Validacion
- Verificar que multiples de potencias de `x` de los generadores del resultado vuelven a `I`.

### Fallos tipicos y fallback
- Saturacion no concluyente -> activar modo iterativo.

---

## Receta 8: Syzygies de generadores

### Objetivo
Obtener relaciones entre generadores del ideal.

### Preconditions
- Ideal definido con generadores no triviales.

### Ejemplo minimo
```json
{
  "base_field": "QQ",
  "variables": ["x", "y"],
  "generators": ["x^2", "x*y"],
  "goal": "syzygies"
}
```

### Toolchain
1. `grobner_syzygies(input_type="handle", input_id_or_generators=ideal_id)`.

### Salida esperada
- Relacion equivalente a `y*(x^2) - x*(x*y) = 0`.

### Validacion
- Sustituir syzygia en los generadores y verificar anulacion.

### Fallos tipicos y fallback
- Presentacion demasiado grande -> reducir conjunto de generadores primero.

---

## Receta 9: Resolucion libre (nivel basico)

### Objetivo
Extraer informacion homologica inicial del ideal/modulo.

### Preconditions
- Estructura algebraica ya depurada.

### Ejemplo minimo
```json
{
  "base_field": "QQ",
  "variables": ["x", "y"],
  "generators": ["x", "y"],
  "goal": "resolution"
}
```

### Toolchain
1. `grobner_free_resolution(target_id=ideal_id, kind="ideal", length=2)`.

### Salida esperada
- Objeto `resolution_id` con mapas no triviales.

### Validacion
- Revisar consistencia de longitud y completitud (`is_complete` si existe).

### Fallos tipicos y fallback
- Recurso insuficiente -> limitar longitud o simplificar ideal.

---

## Receta 10: Serie de Hilbert en caso graduado

### Objetivo
Calcular invariante graduado para ideal/modulo homogeno.

### Preconditions
- Generadores homogeneos o esquema de pesos valido.

### Ejemplo minimo
```json
{
  "base_field": "QQ",
  "variables": ["x", "y"],
  "generators": ["x^2", "x*y", "y^2"],
  "goal": "hilbert"
}
```

### Toolchain
1. Confirmar homogeneidad.
2. `grobner_hilbert_series(target_id=ideal_id)`.

### Salida esperada
- Serie de Hilbert racional o polinomica segun contexto.

### Validacion
- Verificar coherencia con conteo de monomios en grados bajos.

### Fallos tipicos y fallback
- No homogeno -> homogeneizar o cambiar estrategia.

---

## Receta 11: Criterio de puente numerico (homotopia)

### Objetivo
Moverse a solucion numerica cuando el costo simbolico no es razonable.

### Preconditions
- No se requiere certificado exacto final.
- Timeouts simbolicos repetidos.

### Ejemplo minimo
```json
{
  "base_field": "QQ",
  "variables": ["x", "y", "z"],
  "generators": ["x^6+y^6+z^6-1", "x^3-y^2+z", "x*y*z-1/10"],
  "goal": "solve_zero_dim",
  "constraints": {"exact_certificate_required": false, "time_budget_s": 60}
}
```

### Toolchain
1. Pre-simplificar simbolicamente si posible.
2. Exportar a solver de homotopia.
3. Resolver numericamente y filtrar soluciones.
4. Verificar residuos.

### Salida esperada
- Lista de soluciones aproximadas + tolerancia.

### Validacion
- Residuo por ecuacion menor que umbral declarado.

### Fallos tipicos y fallback
- Trayectorias inestables -> ajustar homotopia/tolerancias.
- Si se exige exactitud en parte del problema, volver a subproblemas simbolicos.

---

## Plantilla de reporte obligatorio

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
