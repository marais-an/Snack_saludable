# Changelog — SnackCheck

Los cambios se registran siguiendo SemVer (`MAJOR.MINOR.PATCH`). Esta es la primera versión documentada; las correcciones realizadas durante su desarrollo se incluyen en ella, sin atribuir versiones anteriores que no se registraron.

## [1.0.0] - 2026-10-11

### Añadido

- Flujo n8n para recibir solicitudes POST en `/nutrition-check` con un código de barras y consultar Open Food Facts sin clave API.
- Validación de código de barras ausente o no numérico, producto inexistente y datos nutricionales insuficientes.
- Clasificación de azúcar, sal y grasa por 100 g, cálculo de puntuación de preocupación y semáforo nutricional.
- Evaluación del Nutri-Score y selección del peor resultado entre este y el semáforo. Uso exclusivo del semáforo cuando el Nutri-Score falta o es `unknown`.
- Tres ramas condicionales de IA para los veredictos Saludable, Moderado y No saludable, con tonos diferentes y el modelo Groq `openai/gpt-oss-20b`. La IA genera la explicación a partir del veredicto calculado por el flujo.
- Respuesta final en texto plano con `Content-Type: text/plain; charset=utf-8` y manejo de respuestas con HTTP 200, 400 y 404 según el caso.
- README con las seis secciones requeridas y registro `TESTING.md` con casos TC-001 a TC-012 de tipo funcional, integración, error y rendimiento.

### Corregido

- Referencia del nodo Code al campo `Nivel de azucar`, sin tilde, para contar correctamente el azúcar alto. En Nutella, la puntuación de preocupación pasó de 1 a 2.
- Conexión de la salida false de `Validacion - Nutriscore disponible` para que el flujo continúe la clasificación por semáforo cuando no existe Nutri-Score. La prueba con Dark Chocolate Blanxart devolvió Moderado.
- Cierre de bloques de código y encabezados Markdown del README.

### Cambiado

- Nombre del nodo de cálculo a `Calculo - Puntuacion de preocupacion`.
- Nombre del nodo de modelo a `Modelo - Groq`.

### Verificación

- Pruebas realizadas de las tres clasificaciones, Coca-Cola, Nutri-Score `unknown`, datos insuficientes y entradas erróneas.
- Recorrido end-to-end con Nutella desde PowerShell hasta la respuesta generada mediante Groq.
- Tiempo observado en una medición end-to-end: 2.8502927 segundos. No se registró un umbral de aceptación ni una prueba de carga.

Los resultados y las precisiones de la evidencia se detallan en `TESTING.md`.
