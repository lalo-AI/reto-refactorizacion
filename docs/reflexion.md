# Reflexión final

**Nombre:** Eduardo Cedillo
**Matrícula:** 430895
**Fecha:** 8 de octubre de 2026

Claude Code me resultó muy útil, sobre todo en la fase de análisis. Antes de
modificar cualquier archivo, revisó todo `src/` y produjo un diagnóstico
ordenado: lógica de precios duplicada entre `registrar_venta` y `cotizar`,
números mágicos, condicionales anidados, código muerto, archivos abiertos sin
`with`, nombres inconsistentes y un `menu()` con complejidad excesiva. Con ese
diagnóstico armamos un plan priorizado y lo ejecutamos en ocho refactorizaciones
incrementales, que llevaron los hallazgos de Ruff de 20 a 0
(20 → 18 → 15 → 8 → 5 → 3 → 2 → 1 → 0, terminando en *All checks passed!*),
mientras las 20 pruebas siguieron pasando en cada iteración.

La IA señaló detalles que yo no había considerado. Por ejemplo, notó que
`cotizar` y `registrar_venta` no validan en el mismo orden, por lo que el
mensaje de error para un código vacío es distinto, y que `cotizar` no aplica el
descuento VIP. Unificarlas "por limpieza" habría cambiado el comportamiento.
También advirtió que renombrar `contadorVentas` solo en `gestor.py` no habría
fallado de inmediato: `almacen.py` habría creado en silencio un atributo nuevo y
los folios se habrían reiniciado después de recargar los datos.

Mi papel fue sobre todo delimitar y verificar. Cada prompt se restringió a una
sola preocupación, y le indiqué explícitamente que no modificara `tests/` ni
`pyproject.toml` y que no usara `ruff --fix`. Antes de los renombres riesgosos
pedí buscar primero todas las referencias, y al renombrar el contador exigí
conservar la clave JSON `"contador"`. Al migrar el manejo de archivos a context
managers decidí mantener el `except Exception` de `cargar_datos`, aunque es
demasiado amplio, porque esa iteración se limitaba a los context managers y
restringirlo habría cambiado el comportamiento. Las pruebas originales nunca se
tocaron, porque `README.md` lo prohíbe expresamente.

También tuve que ajustar el plan sobre la marcha. Al extraer la lógica de
precios compartida desapareció el hallazgo C901 de `registrar_venta`, así que
adapté la siguiente iteración para atender solo el condicional VIP anidado, en
lugar de hacer un cambio de complejidad que ya no era necesario.

Lo principal que aprendí es que la IA es una muy buena asistente de análisis e
implementación, pero no sustituye el criterio del desarrollador. Definir el
alcance, verificar cada propuesta, proteger el comportamiento observable,
interpretar los resultados de `pytest` y Ruff y decidir cuándo *no* aplicar una
mejora sugerida siguió siendo mi responsabilidad. Trabajar en pasos pequeños y
validados permitió que el proceso fuera predecible: ninguna iteración rompió las
pruebas, y cada commit representa un cambio fácil de revisar y, si hiciera
falta, de revertir.
