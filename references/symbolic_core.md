# Symbolic Core v2: Fundamentos y Restricciones Operativas

## 1) Alcance de este documento

Este archivo fija el marco teorico para el trabajo simbolico exacto en el skill.

Incluye:
- definiciones minimas necesarias,
- precondiciones de algoritmos,
- limites practicos,
- claims trazables a bibliografia primaria u oficial.

No incluye narrativa historica extensa; para referencias, ver `bibliography_primary.md`.

## 2) Notacion base

- Anillo: `R = k[x1, ..., xn]`.
- Ideal: `I = <f1, ..., fm> subset R`.
- Orden monomial global: `>`.
- Termino lider de `f`: `LT(f)`.
- Monomio lider: `LM(f)`.

## 3) Base de Groebner: definicion operativa

**Claim CLM-SYM-001**: Un conjunto `G subset I` es base de Groebner para `I` (respecto a `>`) si `<LT(G)> = <LT(I)>`. [BIB-BUCHBERGER-2006]

**Claim CLM-SYM-002**: Con una base de Groebner fija, el residuo de division multivariada es unico para un ordering fijo; en particular, `NF(f, G)=0` caracteriza membresia (`f in I`) en ese contexto. [BIB-CLO-IVA]

## 4) Ordenamientos y costo computacional

**Claim CLM-SYM-003**: `degrevlex` suele ser mas eficiente para computo inicial de GB, mientras `lex` es preferido para eliminacion/triangularizacion cuando aplica. [BIB-CLO-IVA]

**Claim CLM-SYM-004**: El cambio de ordering por FGLM evita recalcular desde cero una GB en `lex` en el caso 0-dimensional. [BIB-FGLM-1993]

## 5) Algoritmo de Buchberger y criterio-S

**Claim CLM-SYM-005**: El criterio-S de Buchberger caracteriza cuando un conjunto es GB mediante reducciones a cero de S-polinomios de todos los pares relevantes. [BIB-BUCHBERGER-2006]

Uso en el skill:
- baseline robusto,
- fallback cuando variantes avanzadas no aplican.

Riesgo operativo:
- crecimiento de terminos y coeficientes en sistemas grandes.

## 6) F4 y F5: alcance realista

**Claim CLM-SYM-006**: F4 reorganiza reducciones de pares criticos via algebra lineal matricial (tipo Macaulay), lo que puede mejorar rendimiento practico frente a Buchberger clasico en muchos casos. [BIB-F4-1999]

**Claim CLM-SYM-007**: F5 introduce firmas para evitar clases de reducciones inutiles; su efectividad depende del tipo de sistema y de la implementacion concreta. No se debe afirmar mejora universal incondicional. [BIB-F5-2002]

Politica del skill:
- reportar `algorithm` usado,
- evitar claims absolutos de superioridad sin benchmark local.

## 7) FGLM y solucion 0-dimensional

**Claim CLM-SYM-008**: FGLM esta disenado para ideales 0-dimensionales y conversion de ordering mediante algebra lineal en el cociente `R/I`. [BIB-FGLM-1993]

Precondiciones operativas:
- ideal 0-dimensional,
- orderings globales compatibles,
- base inicial valida.

Fallo tipico:
- si no es 0-dimensional, `fglm` no aplica de forma directa.

## 8) Eliminacion

**Claim CLM-SYM-009**: Con ordering de eliminacion (o procedimientos de eliminacion sobre una GB), se obtiene informacion proyectada en subconjuntos de variables. [BIB-CLO-IVA]

Uso operativo:
- proyecciones,
- ecuaciones implicitas,
- preprocesamiento antes de primaria/radical.

## 9) Radical, primaria y saturacion

**Claim CLM-SYM-010**: El radical `sqrt(I)` describe la misma variedad algebraica que `I` sobre clausura algebraica, ignorando multiplicidades. [BIB-CLO-IVA]

**Claim CLM-SYM-011**: La descomposicion primaria separa componentes algebraicas; algoritmos tipo GTZ son un estandar clasico en CAS. [BIB-GTZ-1988]

**Claim CLM-SYM-012**: La saturacion `I:J^infty` elimina componentes contenidas en la subvariedad definida por `J`. [BIB-CLO-IVA]

## 10) Syzygies y resoluciones

**Claim CLM-SYM-013**: El modulo de syzygies captura relaciones entre generadores y es base para construir resoluciones libres. [BIB-EISENBUD-1995]

Uso operativo:
- diagnostico estructural,
- invariantes homolgicos,
- soporte para analisis de presentaciones.

## 11) Hilbert y contexto graduado

**Claim CLM-SYM-014**: La serie de Hilbert es un invariante graduado; su interpretacion requiere compatibilidad de graduacion (homogeneidad/pesos). [BIB-CLO-IVA]

Riesgo operativo:
- aplicar ruta `hilbert` en ideales no homogeneos produce resultados no alineados con el objetivo.

## 12) Complejidad: como reportarla sin exagerar

**Claim CLM-SYM-015**: El peor caso de Groebner puede ser extremadamente costoso; existen resultados de alta complejidad en literatura teorica. [BIB-MAYR-MEYER-1982]

**Claim CLM-SYM-016**: Ese peor caso no implica que todo problema practico sea intratable; la estructura del sistema y el ordering dominan el rendimiento real. [BIB-MAYR-RITSCHER-2013]

Politica de reporte:
- no usar rankings absolutos de software,
- reportar benchmark local reproducible.

## 13) Aritmetica modular y swell de coeficientes

**Claim CLM-SYM-017**: El uso de metodos modulares puede mitigar crecimiento de coeficientes en calculos sobre `QQ`, segun implementacion y caso. [DOC-SINGULAR]

Regla del skill:
- si hay crecimiento severo, probar `algorithm="modular"` cuando el backend lo soporte.

## 14) Capas de validacion exigidas por el skill

1. Validacion algebraica minima:
- `grobner_basis_check` si aplica.
- `grobner_normal_form` para membresia/consistencia.

2. Validacion de precondiciones:
- `fglm` solo en 0-dimensional,
- `hilbert` solo en contexto graduado.

3. Validacion de interpretacion:
- resultado matematico legible, no solo output crudo.

## 15) Mapeo rapido de claims -> evidencia

- Claims simbolicos basicos: ver `BIB-BUCHBERGER-2006`, `BIB-CLO-IVA`.
- F4/F5/FGLM: ver `BIB-F4-1999`, `BIB-F5-2002`, `BIB-FGLM-1993`.
- Primaria GTZ: ver `BIB-GTZ-1988`.
- Complejidad: ver `BIB-MAYR-MEYER-1982`, `BIB-MAYR-RITSCHER-2013`.
- Implementacion oficial: ver `DOC-OSCAR-GB`, `DOC-SINGULAR`.
