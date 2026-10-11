# SnackCheck — Registro de pruebas

## Alcance y evidencia

Registro de las pruebas realizadas sobre el flujo n8n de SnackCheck. Los resultados se transcriben del resumen de traspaso de la sesión de trabajo; no se han vuelto a ejecutar para elaborar este documento. PASS indica que el resultado observado coincide con el esperado documentado.

El flujo recibe una solicitud POST en `/nutrition-check`, consulta Open Food Facts, aplica las reglas nutricionales y utiliza tres ramas de IA con Groq (`openai/gpt-oss-20b`) para generar la explicación. La IA no decide el veredicto.

## Registro TC-XXX

| ID | Tipo | Caso y entrada | Resultado esperado | Resultado observado | Estado |
| --- | --- | --- | --- | --- | --- |
| TC-001 | Funcional | Nutella: `3017620422003` | No saludable | No saludable; puntuación de preocupación corregida a 2 | PASS |
| TC-002 | Funcional | Pain de mie Seigle & Graines: `3760049798609` | Saludable | Saludable | PASS |
| TC-003 | Funcional | Gerblé/Sésamo: `3175680011480` | Moderado | Moderado | PASS |
| TC-004 | Funcional | Coca-Cola Original: `5449000000996` | Aplicar el peor resultado entre semáforo y Nutri-Score | No saludable por el peor resultado con Nutri-Score | PASS |
| TC-005 | Funcional | Dark Chocolate Blanxart: `8436006832468`, Nutri-Score `unknown` | Utilizar solo el semáforo y continuar hasta la respuesta | Moderado utilizando solo el semáforo | PASS |
| TC-006 | Error | Producto existente con nutrientes incompletos | No emitir un veredicto; devolver `sin_datos` con HTTP 200 | `sin_datos`, HTTP 200 | PASS |
| TC-007 | Error | Producto inexistente: `98765432109876543210987654321` | Error de producto no encontrado, HTTP 404 | `PRODUCTO_NO_ENCONTRADO`, HTTP 404 | PASS |
| TC-008 | Error | `{"barcode": ""}` | Error de código de barras faltante, HTTP 400 | `CODIGO_BARRAS_FALTANTE`, HTTP 400 | PASS |
| TC-009 | Error | `{"barcode": "30A1762"}` | Rechazar caracteres no numéricos con error estructurado, HTTP 400 | Error estructurado, HTTP 400 | PASS |
| TC-010 | Error | Cuerpo JSON vacío: `{}` | Error de código de barras faltante, HTTP 400 | `CODIGO_BARRAS_FALTANTE`, HTTP 400 | PASS |
| TC-011 | Integración | Nutella desde PowerShell → Webhook → Open Food Facts → lógica → Groq → respuesta | Completar el recorrido y devolver el resultado final | Recorrido completo correcto | PASS |
| TC-012 | Rendimiento | Nutella end-to-end, medición con `Measure-Command` | Registrar el tiempo del recorrido completo | 2.8502927 segundos | Tiempo registrado |

## Precisiones del registro

- **TC-006:** el resumen confirma la prueba y su resultado, pero no conserva el código de barras utilizado. Se mantiene el resultado comprobado sin inventar la entrada.
- **TC-009:** no se conserva el identificador exacto del error; sí se confirma la respuesta estructurada con HTTP 400.
- **TC-012:** corresponde a una medición individual. No consta un umbral de aceptación, una media ni una prueba de carga; el tiempo registrado no permite afirmar un cumplimiento de rendimiento adicional.
- Para las respuestas válidas, el flujo está configurado para devolver texto plano con HTTP 200 y `Content-Type: text/plain; charset=utf-8`.

## Correcciones verificadas durante las pruebas

### Puntuación de preocupación

El nodo Code buscaba `Nivel de azúcar`, pero el campo real es `Nivel de azucar`, sin tilde. Se corrigió la referencia y Nutella pasó de una puntuación incorrecta de 1 a `puntuacion_preocupacion = 2`.

El nodo final se denomina `Calculo - Puntuacion de preocupacion`.

### Continuidad sin Nutri-Score

La salida false de `Validacion - Nutriscore disponible` estaba sin conectar. Se conectó a `Clasificacion - Veredicto final no saludable` para continuar la lógica de clasificación usando `Resultado semaforo` cuando falta Nutri-Score.

TC-005 verificó el resultado Moderado por semáforo tras la corrección. El nombre del nodo de destino no implica que todos los productos sin Nutri-Score se clasifiquen como No saludable.

## Casos exploratorios excluidos de la evidencia principal

| Entrada | Observación | Motivo de exclusión |
| --- | --- | --- |
| `6111242100985` — Queso fresco Jaouda | Recorrió la rama Saludable, pero faltaba el nombre en la respuesta | El caso saludable definitivo es TC-002 |
| `6111252421582` — Aquafina | Nutri-Score `unknown`, azúcar y grasa `null`; terminó en datos insuficientes | No demuestra un veredicto basado solo en semáforo con nutrientes completos |
| `9999999999999` | Open Food Facts devolvió un producto llamado Salatgurke | No es un código válido para probar producto inexistente |
