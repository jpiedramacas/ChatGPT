# The Chat Format

En este cuaderno, explorarás cómo puedes utilizar el formato de chat para tener conversaciones extendidas con chatbots personalizados o especializados para tareas o comportamientos específicos.

## Configuración

```python
import os
import openai
from dotenv import load_dotenv, find_dotenv
_ = load_dotenv(find_dotenv()) # leer el archivo .env local

openai.api_key  = os.getenv('OPENAI_API_KEY')

def get_completion(prompt, model="gpt-3.5-turbo"):
    messages = [{"role": "user", "content": prompt}]
    response = openai.ChatCompletion.create(
        model=model,
        messages=messages,
        temperature=0, # este es el grado de aleatoriedad en la respuesta del modelo
    )
    return response.choices[0].message["content"]

def get_completion_from_messages(messages, model="gpt-3.5-turbo", temperature=0):
    response = openai.ChatCompletion.create(
        model=model,
        messages=messages,
        temperature=temperature, # este es el grado de aleatoriedad en la respuesta del modelo
    )
#     print(str(response.choices[0].message))
    return response.choices[0].message["content"]

# Ejemplo 1: Asistente que habla como Shakespeare
messages = [  
    {'role':'system', 'content':'You are an assistant that speaks like Shakespeare.'},    
    {'role':'user', 'content':'tell me a joke'},   
    {'role':'assistant', 'content':'Why did the chicken cross the road'},   
    {'role':'user', 'content':'I don\'t know'}  
]

response = get_completion_from_messages(messages, temperature=1)
print(response)

# Ejemplo 2: Chatbot amistoso
messages = [  
    {'role':'system', 'content':'You are a friendly chatbot.'},    
    {'role':'user', 'content':'Hi, my name is Isa'}  
]
response = get_completion_from_messages(messages, temperature=1)
print(response)

messages = [  
    {'role':'system', 'content':'You are a friendly chatbot.'},    
    {'role':'user', 'content':'Yes,  can you remind me, What is my name?'}  
]
response = get_completion_from_messages(messages, temperature=1)
print(response)

messages = [  
    {'role':'system', 'content':'You are a friendly chatbot.'},
    {'role':'user', 'content':'Hi, my name is Isa'},
    {'role':'assistant', 'content': "Hi Isa! It's nice to meet you. \
    Is there anything I can help you with today?"},
    {'role':'user', 'content':'Yes, you can remind me, What is my name?'}  
]
response = get_completion_from_messages(messages, temperature=1)
print(response)
```

## OrderBot

Podemos automatizar la recopilación de mensajes de usuario y respuestas del asistente para construir un OrderBot. El OrderBot tomará pedidos en un restaurante de pizzas.

```python
def collect_messages(_):
    prompt = inp.value_input
    inp.value = ''
    context.append({'role':'user', 'content':f"{prompt}"})
    response = get_completion_from_messages(context) 
    context.append({'role':'assistant', 'content':f"{response}"})
    panels.append(
        pn.Row('User:', pn.pane.Markdown(prompt, width=600)))
    panels.append(
        pn.Row('Assistant:', pn.pane.Markdown(response, width=600, style={'background-color': '#F6F6F6'})))
 
    return pn.Column(*panels)

import panel as pn  # GUI
pn.extension()

panels = [] # recopilar pantallas 

context = [ {'role':'system', 'content':"""
You are OrderBot, an automated service to collect orders for a pizza restaurant. \
You first greet the customer, then collect the order, \
and then ask if it's a pickup or delivery. \
You wait to collect the entire order, then summarize it and check for a final \
time if the customer wants to add anything else. \
If it's a delivery, you ask for an address. \
Finally you collect the payment.\
Make sure to clarify all options, extras and sizes to uniquely \
identify the item from the menu.\
You respond in a short, very conversational friendly style. \
The menu includes \
pepperoni pizza  12.95, 10.00, 7.00 \
cheese pizza   10.95, 9.25, 6.50 \
eggplant pizza   11.95, 9.75, 6.75 \
fries 4.50, 3.50 \
greek salad 7.25 \
Toppings: \
extra cheese 2.00, \
mushrooms 1.50 \
sausage 3.00 \
canadian bacon 3.50 \
AI sauce 1.50 \
peppers 1.00 \
Drinks: \
coke 3.00, 2.00, 1.00 \
sprite 3.00, 2.00, 1.00 \
bottled water 5.00 \
"""} ]  # acumular mensajes

inp = pn.widgets.TextInput(value="Hi", placeholder='Enter text here…')
button_conversation = pn.widgets.Button(name="Chat!")

interactive_conversation = pn.bind(collect_messages, button_conversation)

dashboard = pn.Column(
    inp,
    pn.Row(button_conversation),
    pn.panel(interactive_conversation, loading_indicator=True, height=300),
)

dashboard
```

## Resumen de la orden en JSON

```python
messages = context.copy()
messages.append(
    {'role':'system', 'content':'create a json summary of the previous food order. Itemize the price for each item\
    The fields should be 1) pizza, include size 2) list of toppings 3) list of drinks, include size  4) list of sides include size  5)total price '},    
)

response = get_completion_from_messages(messages, temperature=0)
print(response)
```

## ¡Experimenta por tu cuenta!

Puedes modificar el menú o las instrucciones para crear tu propio orderbot.

