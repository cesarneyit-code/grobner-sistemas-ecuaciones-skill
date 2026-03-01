# Hybrid Bridge v2: De Simbolico Exacto a Homotopia Numerica

## 1) Proposito

Este documento define **cuando** y **como** pasar de una ruta Groebner exacta a una ruta numerica (homotopia) sin perder trazabilidad.

Principio:
- simbolico para certificacion exacta,
- numerico para escalabilidad aproximada cuando el costo simbolico es prohibitivo y el problema lo permite.

## 2) Trigger de cambio (decision operativa)

Cambiar a puente numerico si se cumplen simultaneamente:
1. `exact_certificate_required = false`.
2. Timeouts o presion de memoria repetidos en ruta simbolica.
3. El objetivo admite tolerancia numerica controlada.

No cambiar si:
- hay exigencia de prueba algebraica exacta final,
- el usuario pide igualdad/membresia exacta de ideales como salida central.

## 3) Riesgos controlados

Riesgos al cambiar:
- perdida de exactitud exacta,
- sensibilidad a tolerancias,
- soluciones espurias por condicionamiento.

Mitigaciones:
- reportar tolerancia y residual,
- repetir corrida con configuracion estable,
- validar soluciones candidatas por sustitucion y residual.

## 4) Flujo puente recomendado

1. **Pre-simplificacion simbolica** (opcional pero recomendada).
- eliminacion parcial,
- limpieza por saturacion/radical cuando tenga sentido.

2. **Exportacion del sistema**.
- fijar variables y orden,
- fijar precision/tolerancia inicial,
- registrar semilla y parametros de corrida.

3. **Resolucion homotopica**.
- usar homotopia adecuada (total degree, polyhedral, parameter),
- seguir trayectorias hasta soluciones candidatas.

4. **Filtrado y verificacion**.
- clasificar soluciones reales/complejas,
- aceptar solo candidatas con residual bajo umbral.

5. **Retorno al reporte unificado**.
- integrar resultados numericos en `ReportSpec`.
- incluir advertencia de naturaleza aproximada.

## 5) Criterios de aceptacion numerica

Minimos obligatorios:
1. Residuo por ecuacion por debajo de `tol_residual`.
2. Estabilidad en al menos dos corridas (si el costo lo permite).
3. Registro de parametros: precision, metodo, semilla, tolerancia.

Ejemplo de declaracion:
```json
{
  "numeric_validation": {
    "tol_residual": 1e-10,
    "max_residual_observed": 4.2e-12,
    "method": "polyhedral_homotopy",
    "seed": 12345
  }
}
```

## 6) Integracion con salida del skill

Si hubo puente numerico, `ReportSpec` debe incluir:
- por que se cambio de ruta,
- que solucionador se uso,
- tipo de homotopia,
- tolerancias,
- residual maximo,
- advertencia explicita: resultado aproximado.

## 7) Interoperabilidad simbolico <-> numerico

Patrones utiles:
1. Simbolico primero, numerico despues.
- usar eliminacion/simplificacion para reducir dimension.

2. Numerico primero, simbolico despues.
- usar candidatos numericos para guiar busqueda de estructura exacta en subproblemas.

3. Pipeline mixto iterativo.
- alternar simplificacion simbolica y refinamiento numerico.

## 8) Herramientas recomendadas

- HomotopyContinuation.jl (documentacion y ecosistema Julia).
- PHCpack/Bertini en contextos legacy.

Nota:
- este skill no reemplaza un skill numerico dedicado; define el criterio de puente y el contrato de reporte.

## 9) Claims trazables

- CLM-HYB-001: homotopia puede escalar mejor en problemas grandes de soluciones aproximadas. [DOC-HCJL, BIB-SOMMESE-WAMPLER-2005]
- CLM-HYB-002: el uso de trayectorias en `C` y filtrado posterior de reales es patron estandar en homotopia. [DOC-HCJL]
- CLM-HYB-003: validacion por residual y tolerancia declarada es obligatoria para resultados numericos confiables. [DOC-HCJL]
