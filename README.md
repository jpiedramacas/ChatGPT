# Guía Profesional para Redacción de Prompts

En esta lección, practicarás dos principios fundamentales de redacción de prompts y sus tácticas relacionadas para escribir instrucciones efectivas para modelos de lenguaje a gran escala.

## Configuración
#### Cargar la clave API y las bibliotecas de Python relevantes

En este curso, hemos proporcionado un código que carga la clave API de OpenAI por ti.

```python
import openai
import os

from dotenv import load_dotenv, find_dotenv
_ = load_dotenv(find_dotenv())

openai.api_key  = os.getenv('OPENAI_API_KEY')
```

#### Función auxiliar
A lo largo de este curso, utilizaremos el modelo `gpt-3.5-turbo` de OpenAI y el [endpoint de chat completions](https://platform.openai.com/docs/guides/chat).

Esta función auxiliar facilitará el uso de prompts y la visualización de los resultados generados.
**Nota**: En junio de 2023, OpenAI actualizó gpt-3.5-turbo. Los resultados que ves en el cuaderno pueden ser ligeramente diferentes a los del video. Algunos de los prompts también han sido ligeramente modificados para producir los resultados deseados.

```python
def get_completion(prompt, model="gpt-3.5-turbo"):
    messages = [{"role": "user", "content": prompt}]
    response = openai.ChatCompletion.create(
        model=model,
        messages=messages,
        temperature=0, # este es el grado de aleatoriedad en la salida del modelo
    )
    return response.choices[0].message["content"]
```

**Nota:** Este y todos los demás cuadernos de laboratorio de este curso usan la versión `0.27.0` de la biblioteca de OpenAI.

Para usar la versión `1.0.0` de la biblioteca de OpenAI, aquí está el código que usarías en su lugar para la función `get_completion`:

```python
client = openai.OpenAI()

def get_completion(prompt, model="gpt-3.5-turbo"):
    messages = [{"role": "user", "content": prompt}]
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0
    )
    return response.choices[0].message.content
```

## Principios de Redacción de Prompts
- **Principio 1: Escribe instrucciones claras y específicas**
- **Principio 2: Dale tiempo al modelo para "pensar"**

### Tácticas

#### Táctica 1: Usa delimitadores para indicar claramente partes distintas del input
- Los delimitadores pueden ser cualquier cosa como: ```, """, < >, `<tag> </tag>`, `:`

```python
text = f"""
Debes expresar lo que quieres que haga un modelo proporcionando instrucciones lo más claras y específicas posible. Esto guiará al modelo hacia el resultado deseado y reducirá las posibilidades de recibir respuestas irrelevantes o incorrectas. No confundas escribir un prompt claro con escribir un prompt corto. En muchos casos, los prompts más largos proporcionan más claridad y contexto para el modelo, lo que puede llevar a resultados más detallados y relevantes.
"""
prompt = f"""
Resume el texto delimitado por comillas triples en una sola oración.
```{text}```
"""
response = get_completion(prompt)
print(response)
```

#### Táctica 2: Pide una salida estructurada
- JSON, HTML

```python
prompt = f"""
Genera una lista de tres títulos de libros ficticios junto con sus autores y géneros. Proporciónalos en formato JSON con las siguientes claves: book_id, title, author, genre.
"""
response = get_completion(prompt)
print(response)
```

#### Táctica 3: Pide al modelo que verifique si se cumplen las condiciones

```python
text_1 = f"""
¡Hacer una taza de té es fácil! Primero, necesitas hervir agua. Mientras tanto, toma una taza y pon una bolsa de té en ella. Una vez que el agua esté lo suficientemente caliente, simplemente viértela sobre la bolsa de té. Déjala reposar un poco para que el té se infusione. Después de unos minutos, saca la bolsa de té. Si lo deseas, puedes agregar un poco de azúcar o leche al gusto. ¡Y eso es todo! Tienes una deliciosa taza de té para disfrutar.
"""
prompt = f"""
Se te proporcionará un texto delimitado por comillas triples. Si contiene una secuencia de instrucciones, reescribe esas instrucciones en el siguiente formato:

Paso 1 - ...
Paso 2 - …
…
Paso N - …

Si el texto no contiene una secuencia de instrucciones, simplemente escribe "No se proporcionaron pasos."

\"\"\"{text_1}\"\"\"
"""
response = get_completion(prompt)
print("Resultado para el Texto 1:")
print(response)

text_2 = f"""
El sol brilla intensamente hoy, y los pájaros cantan. Es un día hermoso para dar un paseo por el parque. Las flores están en plena floración, y los árboles se balancean suavemente con la brisa. La gente está fuera, disfrutando del buen tiempo. Algunos están haciendo picnics, mientras que otros juegan o simplemente se relajan en el césped. Es un día perfecto para pasar tiempo al aire libre y apreciar la belleza de la naturaleza.
"""
prompt = f"""
Se te proporcionará un texto delimitado por comillas triples. Si contiene una secuencia de instrucciones, reescribe esas instrucciones en el siguiente formato:

Paso 1 - ...
Paso 2 - …
…
Paso N - …

Si el texto no contiene una secuencia de instrucciones, simplemente escribe "No se proporcionaron pasos."

\"\"\"{text_2}\"\"\"
"""
response = get_completion(prompt)
print("Resultado para el Texto 2:")
print(response)
```

#### Táctica 4: Prompting de "pocos disparos" (Few-shot prompting)

```python
prompt = f"""
Tu tarea es responder en un estilo consistente.

<niño>: Enséñame sobre la paciencia.

<abuelo>: El río que talla el valle más profundo fluye de un modesto manantial; la sinfonía más grandiosa se origina en una sola nota; el tapiz más intrincado comienza con un solo hilo.

<niño>: Enséñame sobre la resiliencia.
"""
response = get_completion(prompt)
print(response)
```

### Principio 2: Dale tiempo al modelo para “pensar”

#### Táctica 1: Especifica los pasos necesarios para completar una tarea

```python
text = f"""
En un encantador pueblo, los hermanos Jack y Jill emprendieron una búsqueda para recoger agua de un pozo en la cima de una colina. Mientras subían, cantando alegremente, la desgracia los golpeó: Jack tropezó con una piedra y rodó colina abajo, seguido por Jill. Aunque un poco magullados, regresaron a casa con abrazos reconfortantes. A pesar del percance, su espíritu aventurero permaneció intacto, y continuaron explorando con deleite.
"""
# ejemplo 1
prompt_1 = f"""
Realiza las siguientes acciones:
1 - Resume el siguiente texto delimitado por comillas triples en una sola oración.
2 - Traduce el resumen al francés.
3 - Enumera cada nombre en el resumen en francés.
4 - Produce un objeto JSON que contenga las siguientes claves: french_summary, num_names.

Separa tus respuestas con saltos de línea.

Texto:
```{text}```
"""
response = get_completion(prompt_1)
print("Resultado para el prompt 1:")
print(response)

# ejemplo 2
prompt_2 = f"""
Tu tarea es realizar las siguientes acciones:
1 - Resume el siguiente texto delimitado por <> en una sola oración.
2 - Traduce el resumen al francés.
3 - Enumera cada nombre en el resumen en francés.
4 - Produce un objeto JSON que contenga las claves french_summary y num_names.

Usa el siguiente formato:
Texto: <texto a resumir>
Resumen: <resumen>
Traducción: <traducción del resumen>
Nombres: <lista de nombres en el resumen>
JSON de salida: <json con resumen y num_names>

Texto: <{text}>
"""
response = get_completion(prompt_2)
print("\nResultado para el prompt 2:")
print(response)
```

#### Táctica 2: Indicar al modelo que trabaje su propia solución antes de apresurarse a una conclusión

```python
prompt = f"""
Determina si la solución del estudiante es correcta o no.

Pregunta:
Estoy construyendo una instalación de energía solar y necesito ayuda con los cálculos financieros.
- El terreno cuesta $100 / pie cuadrado
- Puedo comprar paneles solares por $250 / pie cuadrado
- Negocié un contrato de mantenimiento que me costará $100k al año, y $10 / pie cuadrado adicionales
¿Cuál es el costo total para el primer año de operaciones en función del número de pies cuadrados?

Solución del estudiante:
Sea x el tamaño de la instalación en pies cuadrados.
Costos:
1. Costo del terreno: 100x
2. Costo de los paneles solares: 250x
3. Costo de mantenimiento: 100,000 + 10x
Costo total: 100x + 250x + 100,000 + 10x = 360x + 100,000
"""
response = get_completion(prompt)
print(response)

# Indicar al modelo que resuelva el problema antes de evaluar la solución del estudiante

prompt = f"""
Tu tarea es determinar si la solución del estudiante

 es correcta o no.
Para resolver el problema haz lo siguiente:
- Primero, resuelve tú mismo el problema incluyendo el costo total final.
- Luego compara tu solución con la del estudiante y evalúa si la solución del estudiante es correcta o no.
No decidas si la solución del estudiante es correcta hasta que hayas resuelto el problema tú mismo.

Usa el siguiente formato:
Pregunta:
```
pregunta aquí
```
Solución del estudiante:
```
solución del estudiante aquí
```
Solución real:
```
pasos para resolver el problema y tu solución aquí
```
¿Es la solución del estudiante la misma que la solución real recién calculada?
```
sí o no
```
Calificación del estudiante:
```
correcto o incorrecto
```

Pregunta:
```
Estoy construyendo una instalación de energía solar y necesito ayuda con los cálculos financieros.
- El terreno cuesta $100 / pie cuadrado
- Puedo comprar paneles solares por $250 / pie cuadrado
- Negocié un contrato de mantenimiento que me costará $100k al año, y $10 / pie cuadrado adicionales
¿Cuál es el costo total para el primer año de operaciones en función del número de pies cuadrados?
```
Solución del estudiante:
```
Sea x el tamaño de la instalación en pies cuadrados.
Costos:
1. Costo del terreno: 100x
2. Costo de los paneles solares: 250x
3. Costo de mantenimiento: 100,000 + 10x
Costo total: 100x + 250x + 100,000 + 10x = 360x + 100,000
```
Solución real:
"""
response = get_completion(prompt)
print(response)
```

## Limitaciones del Modelo: Alucinaciones
- Boie es una empresa real, pero el nombre del producto no es real.

```python
prompt = f"""
Cuéntame sobre el cepillo de dientes inteligente AeroGlide UltraSlim de Boie.
"""
response = get_completion(prompt)
print(response)
```

## Prueba a experimentar por tu cuenta

#### Notas sobre el uso de la API de OpenAI fuera de este aula

Para instalar la biblioteca de Python de OpenAI:
```
!pip install openai
```

La biblioteca necesita ser configurada con la clave secreta de tu cuenta, la cual está disponible en el [sitio web](https://platform.openai.com/account/api-keys).

Puedes establecerla como la variable de entorno `OPENAI_API_KEY` antes de usar la biblioteca:
```
!export OPENAI_API_KEY='sk-...'
```

O, establece `openai.api_key` a su valor:

```python
import openai
openai.api_key = "sk-..."
```

#### Una nota sobre la barra invertida
- En el curso, estamos usando una barra invertida `\` para hacer que el texto quepa en la pantalla sin insertar caracteres de nueva línea '\n'.
- GPT-3 no se ve realmente afectado si insertas o no caracteres de nueva línea. Pero al trabajar con LLMs en general, puedes considerar si los caracteres de nueva línea en tu prompt pueden afectar el rendimiento del modelo.
