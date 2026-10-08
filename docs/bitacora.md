# Bitácora de refactorización

**Nombre:** Eduardo Cedillo
**Matrícula:** 430895
**Fecha:** 8 de octubre de 2026

Registra aquí **cada refactorización** que realices con Claude Code. Copia el
prompt tal cual lo escribiste (o un resumen fiel si fue una conversación larga),
describe el cambio que se aplicó al código y justifica por qué mejora la calidad.
Después de cada cambio ejecuta `pytest` y anota el resultado.

## Punto de partida y configuración

- **Línea base:** `pytest` → 20 passed; `ruff check src` → 20 errores.
- **Configuración de Claude Code:** antes de refactorizar se creó `CLAUDE.md`
  (propósito, estructura, convenciones, comandos de validación, restricciones
  obligatorias y flujo de trabajo) y `.claudeignore` (entornos virtuales,
  bytecode, cachés de pytest/ruff, `.DS_Store` y `.git/`).
- **Fase de exploración:** se pidió a Claude Code un diagnóstico de *code smells*
  de `src/` y un plan priorizado, sin modificar archivos.
- **Estrategia común a todos los prompts:** una sola refactorización a la vez;
  buscar referencias y analizar antes de modificar cuando fuera necesario;
  preservar el comportamiento observable; no modificar `tests/` ni
  `pyproject.toml`; ejecutar `pytest` y `ruff check src` después de cada cambio;
  no usar `ruff --fix`; no avanzar a la siguiente refactorización hasta
  completar la validación y aprobarla.

## Refactorizaciones

| #  | Prompt usado | Cambio realizado | Justificación | Tests OK |
|----|--------------|------------------|---------------|----------|
| 1  | Aplicar solo la refactorización #1 en `src/gestor.py`: introducir constantes con nombre para umbrales y tasas de descuento por volumen y para el IVA; extraer un helper privado para el descuento por volumen y otro para el IVA (solo si conserva la aritmética exacta); reutilizarlos en `registrar_venta()` y `cotizar()`. Sin aplanar el condicional VIP ni dividir `registrar_venta`; `cotizar()` no debe adquirir el descuento VIP; preservar el orden de las operaciones de punto flotante. Explicar antes de editar qué líneas cambian y por qué se preserva el comportamiento. | Commit `ed9a1c2` — *refactor: extract shared pricing logic*. En `src/gestor.py` se agregaron las constantes `UMBRAL_DESCUENTO_MAYOR`, `TASA_DESCUENTO_MAYOR`, `UMBRAL_DESCUENTO_MEDIO`, `TASA_DESCUENTO_MEDIO` y `TASA_IVA`, y los helpers privados `_descuento_por_volumen()` y `_calcular_iva()`. `registrar_venta()` y `cotizar()` los reutilizan; `cotizar()` sigue sin descuento VIP. | Elimina la regla de precios duplicada en dos funciones (un cambio de tasa o umbral ahora se hace en un solo lugar) y reemplaza números mágicos por nombres que documentan la regla de negocio. Se conservó el mismo orden de operaciones para no alterar los totales. **Ruff: 20 → 18** (desapareció SIM108; además `registrar_venta` dejó de reportar C901). | 20/20 PASSED |
| 2  | Aplicar solo la refactorización #2: simplificar el condicional VIP anidado de `registrar_venta()`. Antes de editar, mostrar la condición actual, la propuesta y justificar la equivalencia para `cliente == ""`, `cliente is None`, clientes no VIP y el umbral del monto. No cambiar prefijo, umbral ni tasa VIP; no tocar los helpers de la #1 ni otras partes. | Commit `f3880e8` — *refactor: simplify VIP discount condition*. Cuatro `if` anidados (vacío/None, longitud ≥ 3, prefijo `"VIP"`, monto > 200) se reemplazaron por una sola condición: `if cliente and cliente.startswith("VIP") and aux - desc > 200:`. El cálculo `desc = desc + aux * 0.02` quedó idéntico. | Reduce el anidamiento de 4 niveles a 1 y hace legible la regla VIP. La verificación de longitud era redundante porque `startswith("VIP")` la implica; el cortocircuito de `and` conserva el orden de evaluación y evita llamar a `startswith` sobre `None`. **Ruff: 18 → 15** (desaparecieron los tres SIM102). | 20/20 PASSED |
| 3  | Aplicar solo la refactorización #3: eliminar código muerto u obsoleto. Antes de borrar, buscar en todo el proyecto (incluyendo `src/` y `tests/`) referencias a cada candidato y reportar dónde se define, si tiene referencias y si es seguro eliminarlo. Eliminar solo lo verificado como no usado; no tocar manejo de archivos, nombres, `menu` ni lógica de negocio. | Commit `1860190` — *refactor: remove dead and obsolete code*. Se eliminaron `calcular_descuento_viejo()`, `MODO_DEBUG`, el bloque comentado `exportar_txt`, `reporteViejoCSV()`, el `import os` sin uso de `reportes.py` y las declaraciones `# -*- coding: utf-8 -*-` de los cuatro archivos de `src/`. 35 eliminaciones, 0 adiciones. | El código muerto confunde al lector y genera mantenimiento innecesario; ninguno de los elementos tenía referencias en `src/` ni en `tests/`, y el historial de Git conserva lo eliminado. Las declaraciones de codificación son innecesarias en Python 3. **Ruff: 15 → 8** (UP009 ×4, F401, N802 de `reporteViejoCSV` y SIM115 de `reportes.py`). | 20/20 PASSED |
| 4  | Aplicar solo la refactorización #4 en `src/almacen.py`: reemplazar el manejo manual `open()`/`close()` por context managers en `guardar_datos()` y `cargar_datos()` y quitar el modo `"r"` explícito. Antes de editar, explicar cómo funciona cada operación actual y por qué la nueva versión la preserva. No ampliar ni restringir los tipos de excepción; no cambiar la estructura JSON ni tocar `hayArchivo`. | Commit `3a57457` — *refactor: use context managers for file handling*. `guardar_datos()` y `cargar_datos()` usan `with open(...) as f:`; se quitó el modo `"r"` explícito. Se conservaron el `except Exception`, los mensajes de `ultimo_error`, los valores de retorno y el orden (el estado global se actualiza después de cerrar el archivo). | Garantiza que los archivos se cierren aun si ocurre una excepción y elimina las llamadas `close()` repetidas en varias rutas, reduciendo el riesgo de fugas de recursos. **Ruff: 8 → 5** (SIM115 ×2 y UP015). | 20/20 PASSED |
| 5  | Aplicar solo la refactorización #5: buscar todas las referencias a `hayArchivo` en el proyecto, renombrarla a `hay_archivo`, actualizar únicamente las referencias necesarias y simplificar su cuerpo para regresar directamente la condición. Antes de editar, reportar las referencias y explicar por qué devolver `os.path.exists(ruta)` es equivalente. Sin ordenar imports ni tocar `menu`. | Commit `b844435` — *refactor: rename and simplify file existence check*. `hayArchivo` → `hay_archivo` en `src/almacen.py`; su `if/else` que regresaba `True`/`False` se sustituyó por `return os.path.exists(ruta)`. Se actualizó la única llamada en `src/main.py`. | Unifica el estilo de nombres en snake_case (PEP 8) y elimina un `if/else` redundante: `os.path.exists()` ya devuelve un `bool`. Los tests no usaban esa función, por lo que el renombre era seguro. **Ruff: 5 → 3** (N802 y SIM103). | 20/20 PASSED |
| 6  | Aplicar solo la refactorización #6: reducir la complejidad de `main.menu()` extrayendo responsabilidades cohesivas a helpers privados. Antes de editar, analizar la función, identificar la responsabilidad de cada opción, proponer el conjunto mínimo de helpers para bajar de la complejidad máxima de 10 y explicar cómo se preservan prompts, salidas, orden de interacción, `break`, guardado/carga y manejo de errores. Modificar solo `src/main.py`; sin ordenar imports. | Commit `ee54879` — *refactor: decompose menu into option handlers*. Se extrajeron `_opcion_agregar_producto()`, `_opcion_registrar_venta()`, `_opcion_cotizar()`, `_opcion_mas_vendidos()` y `_opcion_stock_bajo()`, y un diccionario `_ACCIONES_MENU` que asocia las opciones `"1"`–`"7"` con su manejador. `menu()` conserva el saludo, la carga inicial, la impresión de opciones, la opción 8 (guardar y salir) y el mensaje de opción no válida. | `menu()` tenía una cadena `if/elif` de 8 ramas que mezclaba entrada, lógica y salida; ahora cada opción vive en una función pequeña y enfocada, y agregar una opción no aumenta la complejidad de `menu()`. Opciones, números, prompts y textos impresos no cambiaron. **Ruff: 3 → 2** (desapareció C901 de `menu`). | 20/20 PASSED |
| 7  | Aplicar solo la refactorización #7: renombrar la variable global `contadorVentas` a `contador_ventas`. Antes de editar, buscar y reportar todas sus referencias, explicar por qué deben actualizarse juntas y cómo se preservan la clave JSON `"contador"` y la continuidad de folios. Sin cambiar la clave JSON ni la generación de folios; sin ordenar imports. | Commit `94a8ced` — *refactor: rename sales counter to snake case*. `contadorVentas` → `contador_ventas` en sus 8 referencias: 6 en `src/gestor.py` y 2 en `src/almacen.py`. La clave persistida en JSON se mantuvo exactamente como `"contador"`. | Cumple la convención snake_case para variables. Las referencias de ambos módulos debían cambiar juntas porque `almacen.py` lee y asigna `gestor.contador_ventas`; un renombre parcial habría roto el guardado o reiniciado los folios en silencio. Se preservó la continuidad de folios (`test_el_folio_continua_despues_de_recargar` pasó). **Ruff: 2 → 1** (desapareció N816). | 20/20 PASSED |
| 8  | Aplicar solo la refactorización #8: corregir el orden de imports de `src/main.py` para resolver el único hallazgo restante (I001). Antes de editar, mostrar el orden actual y el propuesto y confirmar que no se agregan ni eliminan imports. Cambiar únicamente el orden; sin `ruff --fix`. | Commit `66204ad` — *style: organize main module imports*. Los imports de `src/main.py` se reordenaron alfabéticamente (`almacen`, `gestor`, `reportes`) según la configuración de Ruff. No se agregaron ni eliminaron imports. | Un orden de imports consistente y predecible facilita la lectura y evita diferencias innecesarias en revisiones. Con este cambio el código de `src/` cumple toda la configuración de calidad del reto. **Ruff: 1 → 0** (*All checks passed!*). | 20/20 PASSED |

> Se realizaron 8 refactorizaciones; se agregaron filas a la tabla de la plantilla.

## Evidencia de validación incremental

| Iteración | Commit | `ruff check src` | `pytest` |
|-----------|--------|------------------|----------|
| Línea base | — | 20 errores | 20 passed |
| Refactorización 1 | `ed9a1c2` | 18 | 20 passed |
| Refactorización 2 | `f3880e8` | 15 | 20 passed |
| Refactorización 3 | `1860190` | 8 | 20 passed |
| Refactorización 4 | `3a57457` | 5 | 20 passed |
| Refactorización 5 | `b844435` | 3 | 20 passed |
| Refactorización 6 | `ee54879` | 2 | 20 passed |
| Refactorización 7 | `94a8ced` | 1 | 20 passed |
| Refactorización 8 | `66204ad` | 0 — *All checks passed!* | 20 passed |

- `pytest` se mantuvo en **20/20 pruebas aprobadas** durante todas las
  iteraciones de refactorización verificadas.
- Las pruebas se dejaron **intencionalmente sin modificar**, porque `README.md`
  establece que los archivos de `tests/` no deben modificarse. Tampoco se
  modificó `pyproject.toml`.
- No se utilizó `ruff --fix`: cada hallazgo se resolvió con un cambio revisado.

## Reflexión final (10-15 líneas)

La reflexión detallada se mantiene en [`docs/reflexion.md`](reflexion.md).
