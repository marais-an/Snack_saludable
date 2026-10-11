# SnackCheck — ¿Es saludable este snack?

## Propósito

Este flujo permite analizar un producto a partir de su código de barras y devolver un veredicto sencillo sobre si es saludable, moderado o no saludable.

El flujo consulta Open Food Facts para obtener información nutricional del producto, analiza azúcar, sal y grasa por cada 100 g, tiene en cuenta el Nutri-Score cuando está disponible y genera una explicación breve con IA.

## Cómo funciona

El flujo recibe una solicitud POST en el endpoint `/nutrition-check` con un campo `barcode`.

Primero valida que el código de barras exista y que contenga solo números.

Después consulta Open Food Facts para comprobar si el producto existe y recuperar sus datos nutricionales.

Si el producto tiene suficiente información, clasifica los nutrientes según estas reglas:

### Azúcar

- Bajo: menos de 5 g/100 g
- Medio: entre 5 y 22.5 g/100 g
- Alto: más de 22.5 g/100 g

### Sal

- Alta: más de 1.5 g/100 g
- No alta: 1.5 g/100 g o menos

### Grasa

- Alta: más de 17.5 g/100 g
- No alta: 17.5 g/100 g o menos

Después calcula una puntuación de preocupación contando cuántos nutrientes están en nivel alto.

El resultado del semáforo se obtiene de esta forma:

- 0 nutrientes altos: Saludable
- 1 nutriente alto: Moderado
- 2 o 3 nutrientes altos: No saludable

También se evalúa el Nutri-Score:

- A o B: Saludable
- C: Moderado
- D o E: No saludable

Cuando existe Nutri-Score, el veredicto final utiliza el peor resultado entre el semáforo nutricional y el Nutri-Score.

Si el Nutri-Score no está disponible o aparece como `unknown`, el flujo utiliza únicamente el resultado del semáforo.

Finalmente, el flujo dirige el producto a una de tres ramas de IA según el veredicto final:

- Saludable: tono positivo y alentador
- Moderado: tono positivo y equilibrado
- No saludable: tono cauteloso y amable

La IA genera un párrafo breve utilizando los datos reales del producto y el flujo devuelve una respuesta final en texto plano.

## Configuración

El flujo utiliza los siguientes elementos:

- Webhook POST `/nutrition-check`
- Open Food Facts API
- Groq como proveedor del modelo de lenguaje
- Nodos IF para validaciones y clasificación
- Nodos Edit Fields para preparar y transformar datos
- Un nodo Code para calcular la puntuación de preocupación
- Nodos Merge para unir las distintas ramas
- Respond to Webhook para devolver las respuestas finales

La consulta a Open Food Facts no requiere una clave API.

Para utilizar la parte de IA es necesario configurar en n8n una credencial válida de Groq.

La consulta a Open Food Facts utiliza el código de barras recibido en el webhook para solicitar los campos necesarios del producto:

- `product_name`
- `brands`
- `nutriscore_grade`
- `nutriments`

Dentro de `nutriments` se utilizan:

- `sugars_100g`
- `salt_100g`
- `fat_100g`
- `energy-kcal_100g`

## Uso

El flujo recibe solicitudes POST con un cuerpo JSON que contiene el campo `barcode` como cadena de texto para conservar posibles ceros iniciales.

Ejemplo:

```json
{
  "barcode": "3017620422003"
}
```

El flujo analiza el producto y devuelve:

- nombre del producto
- veredicto final
- azúcar por 100 g
- sal por 100 g
- grasa por 100 g
- explicación generada por IA

Ejemplo ilustrativo de una respuesta final (los valores pueden variar según los datos de Open Food Facts):

```text
🍫 ¿Es saludable este snack?

Producto: Nutella
Veredicto: 🔴 No saludable

🔴 Azúcar: 56.3 g/100 g
🟢 Sal: 0.107 g/100 g
🔴 Grasa: 30.9 g/100 g

💬 Explicación generada por IA...
```

Dependiendo del resultado, el veredicto puede ser:

- 🟢 Saludable
- 🟡 Moderado
- 🔴 No saludable

## Manejo de errores

El flujo controla distintos casos para evitar que una petición incorrecta o unos datos incompletos detengan la automatización.

### Código de barras ausente

Si no se recibe el campo `barcode`, el flujo devuelve una respuesta estructurada con HTTP 400 indicando que el código de barras es obligatorio.
También incluye un ejemplo de uso correcto.

### Código de barras no numérico

Si el campo `barcode` contiene letras u otros caracteres no numéricos, el flujo devuelve HTTP 400 con un mensaje explicando que debe contener solo números.

### Cuerpo vacío

Si la solicitud se envía con un cuerpo vacío, el flujo detecta que falta el código de barras y devuelve HTTP 400.

### Producto no encontrado

Si Open Food Facts no encuentra ningún producto para el código recibido, el flujo devuelve HTTP 404 con un mensaje claro indicando que el producto no fue encontrado.
### Datos nutricionales insuficientes

Si el producto existe pero no dispone de datos suficientes de azúcar, sal y grasa por 100 g, el flujo no genera un veredicto.
En ese caso devuelve una respuesta con estado `sin_datos` y HTTP 200, explicando que no hay información nutricional suficiente para evaluar el producto de forma confiable.

### Nutri-Score no disponible

Si `nutriscore_grade` está vacío o aparece como `unknown`, el flujo no se detiene.
En ese caso ignora la clasificación por Nutri-Score y utiliza únicamente el resultado del semáforo nutricional.

## Limitaciones

El resultado depende de la información disponible en Open Food Facts.
Algunos productos pueden tener datos incompletos, nombres faltantes o valores desactualizados.
El flujo solo puede generar un veredicto cuando dispone de los datos necesarios de azúcar, sal y grasa por 100 g.
El tiempo de respuesta puede variar porque el flujo depende de servicios externos como Open Food Facts y Groq.
La explicación generada por IA utiliza los datos calculados previamente por el flujo. La IA no decide la clasificación nutricional ni vuelve a calcular el veredicto.

La clasificación utilizada responde a las reglas definidas para este proyecto y no sustituye una evaluación nutricional profesional.

## Mejoras futuras

- Añadir un manejo específico de fallos de conexión y tiempos de espera de Open Food Facts y Groq.
- Ampliar el análisis con otros datos nutricionales cuando estén disponibles.

Estas mejoras son propuestas y no funcionalidades implementadas.
