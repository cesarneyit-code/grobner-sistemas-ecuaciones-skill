# Claims Register v2: Claim -> Evidencia

## Proposito

Registro de afirmaciones tecnicas fuertes usadas por el skill.
Cada claim debe tener:
- identificador estable,
- fuente primaria u oficial,
- nivel de confianza,
- estado de verificacion.

## Convenciones

- `status`: `verified`, `needs_review`, `deprecated`.
- `source_type`: `primary` o `official_doc`.
- `confidence`: `high`, `medium`, `low`.

## Claims activos

| claim_id | claim_text (resumen) | source_type | source_ref | confidence | status |
|---|---|---|---|---|---|
| CLM-SYM-001 | Definicion operativa de GB por ideal de terminos lideres | primary | BIB-BUCHBERGER-2006 | high | verified |
| CLM-SYM-002 | `NF(f,G)=0` caracteriza membresia en contexto de GB/ordering fijo | primary | BIB-CLO-IVA | high | verified |
| CLM-SYM-003 | `degrevlex` suele ser mejor para computo inicial; `lex` para eliminacion/lectura | primary | BIB-CLO-IVA | medium | verified |
| CLM-SYM-004 | FGLM convierte ordering en caso 0-dimensional | primary | BIB-FGLM-1993 | high | verified |
| CLM-SYM-005 | Criterio-S de Buchberger para verificar GB | primary | BIB-BUCHBERGER-2006 | high | verified |
| CLM-SYM-006 | F4 usa reduccion matricial y puede mejorar rendimiento practico | primary | BIB-F4-1999 | high | verified |
| CLM-SYM-007 | F5 usa firmas; no implica superioridad universal incondicional | primary | BIB-F5-2002 | medium | verified |
| CLM-SYM-008 | FGLM requiere ideal 0-dimensional | primary | BIB-FGLM-1993 | high | verified |
| CLM-SYM-009 | Eliminacion via ordering/GB proyecta relaciones en subconjuntos de variables | primary | BIB-CLO-IVA | high | verified |
| CLM-SYM-010 | Radical preserva variedad y elimina multiplicidad algebraica | primary | BIB-CLO-IVA | high | verified |
| CLM-SYM-011 | GTZ es referencia clasica para descomposicion primaria | primary | BIB-GTZ-1988 | high | verified |
| CLM-SYM-012 | Saturacion `I:J^infty` remueve componentes sobre `V(J)` | primary | BIB-CLO-IVA | high | verified |
| CLM-SYM-013 | Syzygies modelan relaciones y soportan resoluciones libres | primary | BIB-EISENBUD-1995 | high | verified |
| CLM-SYM-014 | Serie de Hilbert requiere marco graduado interpretable | primary | BIB-CLO-IVA | medium | verified |
| CLM-SYM-015 | El peor caso de Groebner puede tener complejidad muy alta | primary | BIB-MAYR-MEYER-1982 | high | verified |
| CLM-SYM-016 | El rendimiento practico depende de estructura y no solo del peor caso | primary | BIB-MAYR-RITSCHER-2013 | medium | verified |
| CLM-SYM-017 | Metodos modulares mitigan swell de coeficientes en muchos escenarios | official_doc | DOC-SINGULAR | medium | verified |
| CLM-HYB-001 | Homotopia es alternativa escalable para aproximacion en problemas grandes | official_doc | DOC-HCJL | medium | verified |
| CLM-HYB-002 | Resolver en complejo y filtrar reales es patron usual en homotopia | official_doc | DOC-HCJL | medium | verified |
| CLM-HYB-003 | Validacion por residual+tolerancia es obligatoria en resultados numericos | official_doc | DOC-HCJL | high | verified |
| CLM-BENCH-001 | Ordering/algoritmo/estructura dominan rendimiento empirico | primary | BIB-F4-1999 | medium | verified |
| CLM-BENCH-002 | Comparaciones sin control de entorno sesgan conclusiones | official_doc | DOC-OSCAR-GB | medium | verified |

## Checklist de mantenimiento

1. Antes de agregar un claim nuevo:
- crear `claim_id` unico,
- agregar fuente en `bibliography_primary.md`,
- marcar `status=verified` solo si hay evidencia directa.

2. Si un claim no tiene fuente fuerte:
- marcar `needs_review`,
- no usarlo como conclusion principal en respuestas.
