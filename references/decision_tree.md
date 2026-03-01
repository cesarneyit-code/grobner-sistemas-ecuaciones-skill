# Decision Tree v2: Objetivo -> Tecnica -> Validacion -> Fallback

## 0) Preflight obligatorio

Antes de elegir tecnica, fija:
- `base_field` (`QQ`, `GF(p)`, `ZZ`),
- variables y generadores,
- objetivo (`goal`),
- si se exige certificado exacto,
- presupuesto de tiempo/memoria.

Si esto no esta claro, no iniciar computo pesado.

## 1) Seleccion de regimen: simbolico o puente numerico

### 1A. Ruta simbolica (default)
Elige simbolico si:
- se requiere certificacion exacta,
- hay necesidad de pruebas algebraicas (membresia/igualdad de ideales),
- el tamano del sistema aun es manejable con Groebner.

### 1B. Ruta puente numerico (homotopia)
Elige puente numerico si:
- **no** se exige certificado exacto final,
- hay alta dimension/grado y costo simbolico prohibitivo,
- se busca aproximacion robusta de todas las soluciones complejas/reales.

Si eliges 1B, ir a seccion 8.

## 2) Objetivo: resolver sistema finito (0-dimensional)

### Preconditions
- Ideal probablemente 0-dimensional.
- Ordenamiento inicial eficiente disponible (`degrevlex`).

### Toolchain recomendado
1. `grobner_ring_create`
2. `grobner_ordering_create(kind="degrevlex")`
3. `grobner_ideal_create`
4. `grobner_basis_compute(algorithm="buchberger"|"f4"|"modular")`
5. `grobner_ordering_create(kind="lex")`
6. `grobner_fglm`
7. `grobner_normal_form` para chequeos finales

### Validacion
- `grobner_basis_check` positivo.
- Forma triangular o al menos presencia de polinomio en menos variables.
- Residuos cero para relaciones esperadas.

### Fallback
- Si `fglm` falla: revisar 0-dimensionalidad; si no, pasar a objetivo 3 o 4.
- Si `lex` directo es caro: mantener `degrevlex` y eliminar por etapas.
- Si `f4`/`modular` falla: volver a `buchberger`.

## 3) Objetivo: eliminacion/proyeccion

### Preconditions
- Variables a eliminar definidas (`eliminate_vars`).

### Toolchain recomendado
1. Base de Groebner en orden eficiente (`degrevlex`).
2. `grobner_eliminate(ideal_id, vars_or_indices=...)`.
3. Repetir por bloques de variables si el problema es grande.

### Validacion
- Generadores eliminados no contienen variables removidas.
- Chequeo de consistencia por sustitucion o `normal_form`.

### Fallback
- Cambiar ordering de inicio.
- Eliminar por subconjuntos.
- Pasar a puente numerico si solo se necesita proyeccion aproximada.

## 4) Objetivo: analisis geometrico del ideal

### 4A. Radical

Precondition:
- Interesa variedad geometrica sin multiplicidad.

Toolchain:
- `grobner_radical`.

Validacion:
- Comprobar inclusion y consistencia via reduccion en casos test.

Fallback:
- Cambiar campo base o estrategia modular si disponible.

### 4B. Descomposicion primaria

Precondition:
- Interesa separar ramas/componentes.

Toolchain:
- `grobner_primary_decomposition(algorithm="GTZ"|"SY")`.

Validacion:
- Interseccion de componentes reconstruye ideal original (doble contencion en casos de prueba).

Fallback:
- Cambiar algoritmo (`GTZ` <-> `SY`).
- Reducir sistema con eliminacion previa.

### 4C. Saturacion

Precondition:
- Hay subvariedad prohibida definida por `J`.

Toolchain:
- `grobner_saturation(ideal_or_module_id=I, with_id=J)`.

Validacion:
- Generadores finales excluyen componentes soportadas en `V(J)`.

Fallback:
- Probar saturacion iterada.
- Reescalar/reestructurar ideal filtro.

## 5) Objetivo: estructura de relaciones (syzygies/resoluciones)

### Preconditions
- Interesa estructura homologica o presentaciones.

### Toolchain recomendado
1. `grobner_syzygies`
2. `grobner_free_resolution`

### Validacion
- Verificar consistencia minima (mapas, longitudes, `is_complete` cuando aplique).

### Fallback
- Reducir numero de generadores.
- Cambiar campo base para exploracion estructural.

## 6) Objetivo: Hilbert y contexto graduado

### Preconditions
- Ideal/modulo homogeno (o con pesos positivos compatibles).

### Toolchain recomendado
1. `grobner_basis_compute(algorithm="hilbert")` si aplica.
2. `grobner_hilbert_series`.

### Validacion
- Coherencia de grados y serie de Hilbert esperada.

### Fallback
- Homogeneizar sistema antes.
- Volver a `buchberger` y computar invariantes por otra ruta.

## 7) Objetivo: membresia y chequeo rapido

### Preconditions
- Base o ideal disponible.

### Toolchain recomendado
- `grobner_normal_form(poly_or_list, ideal_id_or_gb_id)`.

### Validacion
- Residuo 0 <=> pertenencia al ideal (con base fija y ordering fijo).

### Fallback
- Recalcular GB con ordering consistente.
- Revisar parseo de polinomios/campo base.

## 8) Puente numerico (homotopia) cuando aplica

### Trigger de cambio
Pasa de simbolico a numerico si:
- no se requiere certificado exacto,
- hay timeout/memoria en rutas simbolicas repetidas,
- el objetivo principal es aproximacion de soluciones.

### Flujo recomendado
1. Simplificar simbolicamente (si posible): eliminar variables obvias o componentes espurias.
2. Exportar sistema a solucionador de homotopia (ej. HomotopyContinuation.jl).
3. Resolver y filtrar soluciones reales si es necesario.
4. Validar por residuo numerico (`||f(x)||`).
5. Si necesitas exactitud posterior, intentar reconstruccion/racionalizacion selectiva.

### Validacion
- Residuo numerico bajo tolerancia declarada.
- Reproducibilidad con semilla/parametros de corrida.

### Fallback
- Ajustar tolerancias/pasos.
- Cambiar homotopia (total degree -> polyhedral/parameter).
- Regresar a subproblemas simbolicos si hace falta certificacion parcial.

## 9) Politica de salida minima

Toda respuesta debe incluir:
1. `ProblemSpec` resumido.
2. Herramientas y ordenamientos usados.
3. Resultado matematico interpretable.
4. Validacion ejecutada.
5. Fallback aplicado (si hubo).
6. Citas para claims fuertes.
