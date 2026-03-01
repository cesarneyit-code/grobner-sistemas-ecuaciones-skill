# Quickstart Cheatsheet: grobner-sistemas-ecuaciones

## 1) Activacion rapida (3 formas)

1. Por nombre de skill:
- `Usa grobner-sistemas-ecuaciones para este sistema...`

2. Por objetivo natural:
- `Elimina x de este sistema y valida con normal form`.

3. Por especificacion estructurada:
- enviar `ProblemSpec` (+ `RunSpec` opcional).

---

## 2) Plantilla minima de ProblemSpec

```json
{
  "base_field": "QQ",
  "variables": ["x", "y", "z"],
  "generators": ["y-x^2", "z-x^3"],
  "goal": "eliminate",
  "eliminate_vars": ["x"],
  "expected_dimension": "unknown",
  "constraints": {
    "exact_certificate_required": true,
    "time_budget_s": 120,
    "memory_budget_mb": 4096
  }
}
```

---

## 3) Secuencia base de tools (7 pasos)

1. `grobner_ring_create`
2. `grobner_ordering_create(kind="degrevlex")`
3. `grobner_ideal_create`
4. `grobner_basis_compute`
5. `grobner_basis_check`
6. tool por objetivo (`grobner_eliminate` o `grobner_fglm` o `grobner_radical`...)
7. `grobner_normal_form` para validacion minima

---

## 4) Mapa rapido objetivo -> metodo

1. Resolver 0-dimensional:
- GB en `degrevlex` + `grobner_fglm` a `lex`.

2. Eliminacion/proyeccion:
- `grobner_eliminate`.

3. Geometria sin multiplicidad:
- `grobner_radical`.

4. Separar ramas:
- `grobner_primary_decomposition`.

5. Remover componentes prohibidas:
- `grobner_saturation(I,J)`.

6. Relaciones estructurales:
- `grobner_syzygies`, `grobner_free_resolution`.

7. Invariante graduado:
- `grobner_hilbert_series` (solo si homogeneo/gradado).

---

## 5) Validacion minima obligatoria

1. Confirmar precondiciones del metodo.
2. Ejecutar `grobner_basis_check` cuando aplique.
3. Ejecutar `grobner_normal_form` en polinomios control.
4. Si hay ruta numerica: validar residual y tolerancia declarada.

---

## 6) Fallos frecuentes -> solucion

| fallo | causa tipica | solucion primaria | fallback |
|---|---|---|---|
| `fglm` falla | ideal no 0-dimensional | revisar dimension/objetivo | eliminacion o primaria |
| tiempo alto | ordering/algoritmo | cambiar ordering o algoritmo | eliminar por etapas |
| hilbert inconsistente | no homogeneidad | homogeneizar | usar ruta no graduada |
| saturacion no limpia | filtro `J` pobre | redefinir `J` | saturacion iterativa |
| residual numerico alto | tolerancia/metodo | ajustar tolerancias | cambiar homotopia |

---

## 7) Plantilla minima de ReportSpec

```json
{
  "assumptions": ["base field QQ", "ordering degrevlex"],
  "tools_called": ["grobner_ring_create", "grobner_ideal_create", "grobner_basis_compute"],
  "result_interpretation": "texto matematico interpretable",
  "validation": ["basis_check=true", "normal_form(...)=0"],
  "warnings": ["..."],
  "citations": ["BIB-FGLM-1993", "DOC-OSCAR-GB"]
}
```

---

## 8) Citas esenciales a mano

- `BIB-BUCHBERGER-2006`
- `BIB-F4-1999`
- `BIB-FGLM-1993`
- `BIB-CLO-IVA`
- `BIB-GTZ-1988`
- `DOC-OSCAR-GB`
- `DOC-SINGULAR`
- `DOC-HCJL`

Para detalle completo, ver `references/master_guide.md`.
