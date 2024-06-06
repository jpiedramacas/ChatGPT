# Transformación
En este cuaderno, exploraremos cómo usar los modelos de lenguaje grande para tareas de transformación de texto, como la traducción de idiomas, la corrección ortográfica y gramatical, el ajuste del tono y la conversión de formato.

## Configuración

```python
import openai
import os

from dotenv import load_dotenv, find_dotenv
_ = load_dotenv(find_dotenv()) # leer el archivo .env local

openai.api_key  = os.getenv('OPENAI_API_KEY')

def get_completion(prompt, model="gpt-3.5-turbo", temperature=0): 
    messages = [{"role": "user", "content": prompt}]
    response = openai.ChatCompletion.create(
        model=model,
        messages=messages,
        temperature=temperature, 
    )
    return response.choices[0].message["content"]
```

## Traducción

ChatGPT está entrenado con fuentes en muchos idiomas, lo que le da la capacidad de traducir. Aquí hay algunos ejemplos de cómo usar esta capacidad.

```python
prompt = f"""
Translate the following English text to Spanish: \ 
```Hi, I would like to order a blender```
"""
response = get_completion(prompt)
print(response)
```

```python
prompt = f"""
Tell me which language this is: 
```Combien coûte le lampadaire?```
"""
response = get_completion(prompt)
print(response)
```

```python
prompt = f"""
Translate the following text to French and Spanish
and English pirate: \
```I want to order a basketball```
"""
response = get_completion(prompt)
print(response)
```

```python
prompt = f"""
Translate the following text to Spanish in both the \
formal and informal forms: 
'Would you like to order a pillow?'
"""
response = get_completion(prompt)
print(response)
```

### Traductor universal

Imagina que estás a cargo de TI en una gran empresa multinacional de comercio electrónico. Los usuarios te envían mensajes con problemas de TI en todos sus idiomas nativos. Tu personal es de todo el mundo y solo habla sus idiomas nativos. ¡Necesitas un traductor universal!

```python
user_messages = [
  "La performance du système est plus lente que d'habitude.",  # El rendimiento del sistema es más lento de lo habitual.
  "Mi monitor tiene píxeles que no se iluminan.",              # Mi monitor tiene píxeles que no se iluminan.
  "Il mio mouse non funziona",                                 # Mi mouse no funciona.
  "Mój klawisz Ctrl jest zepsuty",                             # Mi teclado tiene una tecla Ctrl rota.
  "我的屏幕在闪烁"                                               # Mi pantalla está parpadeando.
] 

for issue in user_messages:
    prompt = f"Tell me what language this is: ```{issue}```"
    lang = get_completion(prompt)
    print(f"Original message ({lang}): {issue}")

    prompt = f"""
    Translate the following  text to English \
    and Korean: ```{issue}```
    """
    response = get_completion(prompt)
    print(response, "\n")
```

## ¡Inténtalo tú mismo!
¡Prueba algunas traducciones por tu cuenta!

## Transformación de tono

La escritura puede variar según el público objetivo. ChatGPT puede producir diferentes tonos.

```python
prompt = f"""
Translate the following from slang to a business letter: 
'Dude, This is Joe, check out this spec on this standing lamp.'
"""
response = get_completion(prompt)
print(response)
```

## Conversión de formato

ChatGPT puede traducir entre formatos. El aviso debe describir los formatos de entrada y salida.

```python
data_json = { "resturant employees" :[ 
    {"name":"Shyam", "email":"shyamjaiswal@gmail.com"},
    {"name":"Bob", "email":"bob32@gmail.com"},
    {"name":"Jai", "email":"jai87@gmail.com"}
]}

prompt = f"""
Translate the following python dictionary from JSON to an HTML \
table with column headers and title: {data_json}
"""
response = get_completion(prompt)
print(response)

from IPython.display import display, Markdown, Latex, HTML, JSON
display(HTML(response))
```

## Revisión ortográfica y gramatical

Aquí hay algunos ejemplos de problemas comunes de gramática y ortografía y la respuesta del LLM.

Para señalar al LLM que deseas que revise tu texto, le indicas al modelo que 'revise' o 'revise y corrija'.

```python
text = [ 
  "The girl with the black and white puppies have a ball.",  # The girl has a ball.
  "Yolanda has her notebook.", # ok
  "Its going to be a long day. Does the car need it’s oil changed?",  # Homophones
  "Their goes my freedom. There going to bring they’re suitcases.",  # Homophones
  "Your going to need you’re notebook.",  # Homophones
  "That medicine effects my ability to sleep. Have you heard of the butterfly affect?", # Homophones
  "This phrase is to cherck chatGPT for speling abilitty"  # spelling
]

for t in text:
    prompt = f"""Proofread and correct the following text
    and rewrite the corrected version. If you don't find
    and errors, just say "No errors found". Don't use 
    any punctuation around the text:
    ```{t}```"""
    response = get_completion(prompt)
    print(response)
```

```python
text = f"""
Got this for my daughter for her birthday cuz she keeps taking \
mine from my room.  Yes, adults also like pandas too.  She takes \
it everywhere with her, and it's super soft and cute.  One of the \
ears is a bit lower than the other, and I don't think that was \
designed to be asymmetrical. It's a bit small for what I paid for it \
though. I think there might be other options that are bigger for \
the same price.  It arrived a day earlier than expected, so I got \
to play with it myself before I gave it to my daughter.
"""
prompt = f"proofread and correct this review: ```{text}```"
response = get_completion(prompt)
print(response)

from redlines import Redlines

diff = Redlines(text,response)
display(Markdown(diff.output_markdown))
```

```python
prompt = f"""
proofread and correct this review. Make it more compelling. 
Ensure it follows APA style guide and targets an advanced reader. 
Output in markdown format.
Text: ```{text}```
"""
response = get_completion(prompt)
display(Markdown(response))
```

## ¡Inténtalo tú mismo!
¡Intenta cambiar las instrucciones para formar tu propia revisión!

---

Gracias a los siguientes sitios:

https://writingprompts.com/bad-grammar-examples/
