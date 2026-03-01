# Bibliografia Primaria y Documentacion Oficial (v2)

## Uso

- Cada claim no trivial del skill debe citar al menos una entrada de este catalogo.
- Prioridad: articulo primario o documentacion oficial.
- Formato de cita recomendado en el reporte: `BIB-*` o `DOC-*`.

## Referencias primarias (simbolico)

### BIB-BUCHBERGER-2006
- Bruno Buchberger.
- *Bruno Buchberger's PhD thesis 1965: An algorithm for finding the basis elements of the residue class ring of a zero dimensional polynomial ideal*.
- Journal of Symbolic Computation 41(3-4), 2006 (traduccion/edicion historica).
- DOI: https://doi.org/10.1016/j.jsc.2005.09.007
- Uso: fundamentos de GB y algoritmo de Buchberger.

### BIB-LAZARD-1983
- Daniel Lazard.
- *Groebner-bases, Gaussian elimination and resolution of systems of algebraic equations*.
- LNCS 162 (EUROSAM 1983).
- DOI: https://doi.org/10.1007/3-540-12868-9_99
- Uso: puente conceptual GB <-> algebra lineal (Macaulay/F4).

### BIB-F4-1999
- Jean-Charles Faugere.
- *A new efficient algorithm for computing Groebner bases (F4)*.
- Journal of Pure and Applied Algebra 139(1-3), 1999.
- DOI: https://doi.org/10.1016/S0022-4049(99)00005-5
- Uso: algoritmo F4 y reduccion matricial.

### BIB-F5-2002
- Jean-Charles Faugere.
- *A new efficient algorithm for computing Groebner bases without reduction to zero (F5)*.
- ISSAC 2002.
- DOI: https://doi.org/10.1145/780506.780516
- Uso: firmas y criterios de descarte.

### BIB-FGLM-1993
- J.-C. Faugere, P. Gianni, D. Lazard, T. Mora.
- *Efficient Computation of Zero-dimensional Groebner Bases by Change of Ordering*.
- Journal of Symbolic Computation 16(4), 1993.
- DOI: https://doi.org/10.1006/jsco.1993.1051
- Uso: conversion de ordering en 0-dimensional.

### BIB-GTZ-1988
- P. Gianni, B. Trager, G. Zacharias.
- *Groebner bases and primary decomposition of polynomial ideals*.
- Journal of Symbolic Computation 6(2-3), 1988.
- DOI: https://doi.org/10.1016/S0747-7171(88)80040-3
- Uso: descomposicion primaria tipo GTZ.

### BIB-ROUILLIER-1999
- Fabrice Rouillier.
- *Solving zero-dimensional systems through the Rational Univariate Representation*.
- Applicable Algebra in Engineering, Communication and Computing 9(5), 1999.
- DOI: https://doi.org/10.1007/s002000050114
- Uso: extraccion de soluciones via RUR.

### BIB-MAYR-MEYER-1982
- Ernst W. Mayr, Albert R. Meyer.
- *The complexity of the word problems for commutative semigroups and polynomial ideals*.
- Advances in Mathematics 46(3), 1982.
- DOI: https://doi.org/10.1016/0001-8708(82)90048-2
- Uso: complejidad de peor caso.

### BIB-MAYR-RITSCHER-2013
- Ernst W. Mayr, Stefan Ritscher.
- *Dimension-dependent bounds for Groebner bases of polynomial ideals*.
- Journal of Symbolic Computation 49, 2013.
- DOI: https://doi.org/10.1016/j.jsc.2011.12.018
- Uso: cotas y dependencia con dimension.

### BIB-BKK-1975
- D. N. Bernstein.
- *The number of roots of a system of equations*.
- Functional Analysis and Its Applications 9(3), 1975.
- DOI: https://doi.org/10.1007/BF01075595
- Uso: cota BKK para metodos poliedrales.

## Referencias base (texto tecnico)

### BIB-CLO-IVA
- David Cox, John Little, Donal O'Shea.
- *Ideals, Varieties, and Algorithms* (4th ed.).
- Springer, 2015.
- URL: https://link.springer.com/book/10.1007/978-3-319-16721-3
- Uso: definiciones operativas, eliminacion, radical, bases GB.

### BIB-EISENBUD-1995
- David Eisenbud.
- *Commutative Algebra with a View Toward Algebraic Geometry*.
- Springer GTM 150, 1995.
- URL: https://link.springer.com/book/10.1007/978-1-4612-5350-1
- Uso: syzygies, resoluciones y estructura modular.

### BIB-SOMMESE-WAMPLER-2005
- A. J. Sommese, C. W. Wampler.
- *The Numerical Solution of Systems of Polynomials Arising in Engineering and Science*.
- World Scientific, 2005.
- URL: https://www.worldscientific.com/worldscibooks/10.1142/5722
- Uso: base de homotopia numerica.

## Documentacion oficial (software)

### DOC-OSCAR-GB
- OSCAR documentation: Groebner bases.
- URL: https://docs.oscar-system.org/stable/CommutativeAlgebra/GroebnerBases/groebner_bases/
- Uso: API, restricciones practicas y funciones disponibles en OSCAR.

### DOC-OSCAR-IDEALS
- OSCAR documentation: Ideals and operations.
- URL: https://docs.oscar-system.org/stable/CommutativeAlgebra/ideals/
- Uso: radical, saturacion, operaciones de ideales (segun version).

### DOC-SINGULAR
- Singular manual (latest).
- URL: https://www.singular.uni-kl.de/Manual/latest/index.htm
- Uso: backend, estrategias modulares y operaciones de algebra computacional.

### DOC-HCJL
- HomotopyContinuation.jl documentation.
- URL: https://www.juliahomotopycontinuation.org/
- Uso: metodos de homotopia, configuracion numerica y validacion por residual.

## Nota de mantenimiento

Si una URL cambia:
1. mantener la clave (`BIB-*` o `DOC-*`),
2. actualizar URL,
3. registrar ajuste en `claims_register.md` si afecta claims activos.
