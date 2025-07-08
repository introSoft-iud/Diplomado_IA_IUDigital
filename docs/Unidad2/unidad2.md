

# Módulo 2: Cadenas y Memoria


## Introducción al módulo

Bienvenidos al segundo módulo de nuestro diplomado sobre construcción de aplicaciones asistidas por LLM. En el primer módulo aprendiste a confeccionar instrucciones reutilizables para LLM, los prompt templates, y exploraste cómo acoplar estas instrucciones en cadenas con especificadores de formato llamados output parsers. Sin embargo, estas cadenas de ejecución eran cadenas de un solo turno de interacción entre la IA y el usuario humano. En este módulo aprenderás a darle memoria y contexto a tus cadenas de ejecución. Profundizaremos aún más en el funcionamiento de las cadenas y como limitar su memoria en el contexto del LCEL. Finalmente, pondrás a prueba tu conocimiento creando un chatbot que asiste las labores de un médico realizando tareas secuenciales y que tiene como contexto en su memoria los datos específicos de un paciente.

¡Comencemos!


## Resultados de aprendizaje
Al finalizar esta módulo, estarás en capacidad de:

- Usar cadenas dotadas de memoria usando el LECL.
- Limitar el tamaño del contexto cargado en la memoria de tus cadenas.


## Cronograma de actividades - Módulo 2
| Actividad de aprendizaje       | Evidencia de aprendizaje | Semana       | Ponderación |
|--------------------------------|---------------------------|--------------|--------------|
| EA1:  Cadenas y memoria         | EA1: Cadenas y memoria| Semana 4 y 5 | 25%         |
| **Total**                      |                           |              | **25 %**     |


<!--
Desarrollo temático
-->

## Cadenas

Hay varias maneras de instanciar cadenas de ejecución en LangChain, algunas de las cuales fueron exploradas en el módulo 1. Recientemente, LangChain introdujo LangChain Expression Language (LCEL) como el estándar para construir cadenas. Revisemos más detalladamente de qué se trata:
!!! warning "Para tener en cuenta"
    Recuerda cargar tu llave en las variables de sistema si no lo has hecho:
    ```python
    import os
    import openai
    from dotenv import load_dotenv, find_dotenv

    # Cargar el archivo .env local
    _ = load_dotenv(find_dotenv()) 
    openai.api_key = os.environ['OPENAI_API_KEY']
    ```


### LangChain Expression Language (LCEL)

LangChain Expression Language (LCEL) es una sintaxis para definir cadenas. Permite componer objetos ejecutables (runnables)—objetos que pueden ser ejecutados o encadenados—usando el operador **pipe** (|). Un runnable es cualquier componente que implementa la interfaz Runnable, lo que significa que puede procesar entradas y producir salidas. Ejemplos incluyen:

- Plantillas de prompt (ChatPromptTemplate), como lo hicimos en el módulo 1.

El operador de tubería (|) conecta estos componentes, pasando la salida de un Runnable como la entrada al siguiente. Por ejemplo:

```python
chain = prompt | llm | output_parser
```

Veamos un ejemplo en detalle:

=== "Código"
    ```python
    from langchain_core.prompts import ChatPromptTemplate
    from langchain_openai import ChatOpenAI
    from langchain_core.output_parsers import StrOutputParser

    # Define the prompt template
    prompt = ChatPromptTemplate.from_messages([
        ("system", "You are a world-class technical documentation writer."),
        ("user", "{input}")
    ])

    # Initialize the LLM
    llm = ChatOpenAI(model="gpt-4", temperature=0.7)

    # Create the output parser
    output_parser = StrOutputParser()

    # Compose the chain
    chain = prompt | llm | output_parser

    # Invoke the chain
    response = chain.invoke({"input": "How can LangSmith help with testing?"})
    print(response)
    ```

=== "Salida"
    ```bash
    LangSmith provides a suite of tools designed to assist with software testing, particularly in the realm of language translation and localization. It can be invaluable for teams developing software that needs to function in multiple languages and regions.

    1. **Automated Testing:** LangSmith can automatically test your software’s language functionality, ensuring translations are accurate, context-appropriate, and that all elements of the user interface are correctly localized.

    2. **Quality Assurance:** By checking translations against a comprehensive database, LangSmith aids in maintaining high translation quality, reducing the risk of miscommunication or confusion for users.

    3. **Regression Testing:** When updates or changes are made to the software, LangSmith can help ensure that these changes have not negatively affected the language functionality.

    4. **Cultural Accuracy:** Beyond simple language translation, LangSmith can also verify that cultural nuances and local customs are appropriately considered, which can greatly improve user experience.

    5. **Reporting and Analytics:** LangSmith provides detailed reports on testing outcomes, highlighting any issues or potential areas for improvement. This can provide valuable insights for your development team.

    By integrating LangSmith into your testing process, you can more effectively ensure the quality and accuracy of your software's multilingual and multicultural features, improving overall user experience and satisfaction.
    ```

### Funciones (envueltas con RunnableLambda)

Podemos usar cadenas con funciones al comienzo de la línea de ejecución, un cierto tipo de función muy versátil son las funciones de la clase RunnableLambda. Estas están diseñadas para integrar funciones personalizadas de Python en cadenas de LangChain. Permiten a los desarrolladores envolver funciones de Python arbitrarias (o funciones lambda) en un objeto ejecutable, haciéndolas compatibles con la sintaxis del LCEL. Es decir, convierte una función de Python en un componente encadenable.

La función debe aceptar una entrada compatible con la salida del paso anterior y producir una salida compatible con el siguiente paso.

Por ejemplo:

```python
from langchain_core.runnables import RunnableLambda

# Wrap a function
runnable_sumaUno = RunnableLambda(lambda x: x + 1)  # suma 1 a la entrada
runnable_cuadrado = RunnableLambda(lambda x: x**2)  # eleva al cuadrado la entrada

# La cadena de ejecución sería
chain = runnable_sumaUno | runnable_cuadrado  # encadena las funciones
result = chain.invoke(5)  # (5 + 1) ** 2 = 36
print(result)  # Output: 36
```
Vemos otro ejemplo:

```python
from langchain_core.runnables import RunnableLambda

# Define funciones lambda individuales
add_prefix = RunnableLambda(lambda x: f"Hello, {x}!")
to_upper = RunnableLambda(lambda x: x.upper())

# Construir la cadena
chain = add_prefix | to_upper

# Invocar la cadena
result = chain.invoke("Alice")
print(result)  # Salida: HELLO, ALICE!
```

Veamos cómo construir una cadena que toma una consulta de usuario, la formatea en un prompt, la procesa con un LLM y extrae la primera palabra de la respuesta.

```python
from langchain_core.runnables import RunnableLambda
from langchain_core.prompts import PromptTemplate
from langchain_openai import ChatOpenAI  # o usar otro LLM

# Definir componentes

# Crear un prompt template y llenarlo
format_prompt = RunnableLambda(
    lambda x: PromptTemplate.from_template("Answer briefly: {query}").format(query=x)
)

# Extraer la primera palabra de la respuesta
extract_first_word = RunnableLambda(
    lambda x: x.content.split()[0] if hasattr(x, 'content') else x.split()[0]
)

llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)

# Construir la cadena
chain = format_prompt | llm | extract_first_word

# Invocar la cadena
result = chain.invoke("¿Cómo se llama el presidente de Colombia?")
print(result)  # Salida: Iván (recuerda que gpt-3.5 fue entrenado en datos hasta octubre de 2023)
```

La cadena procesa la consulta de entrada paso a paso: formatea la consulta, la procesa con el LLM y extrae la primera palabra de la respuesta. Puedes reemplazar `ChatOpenAI` con otro modelo (por ejemplo, `HuggingFaceHub`) si es necesario.
Language models (ChatOpenAI).

## Memoria

La memoria es el mecanismo por el cual le damos al LLM contexto de nuestras interacciones previas.


Para explorar el uso de la memoria, instanciaremos una cadena de conversación a partir de la clase preconstruida `ConversationChain`. Importamos los módulos necesarios:

```python
from langchain.chains.conversation.base import ConversationChain
from langchain.memory import ConversationBufferMemory
from langchain_openai import ChatOpenAI
```

Usaremos GPT-4 y el modelo de chat de OpenAI:

```python
chat_model = ChatOpenAI(model_name="gpt-4", temperature=0, streaming=True)
```

Para instanciar una cadena de conversación de la clase `ConversationChain`, necesitamos instanciar primero el objeto de clase `Memory` que nos servirá para administrar la memoria de las interacciones en la conversación. Lo hacemos de la siguiente manera:

```python
memory = ConversationBufferMemory() # Es una instancia de la clase que contiene los controles de la memoria
```

Así, la cadena de conversación la instanciaremos como:

```python
chain = ConversationChain(llm=chat_model, memory=memory, verbose=True)
```
Ahora está todo listo para que comencemos nuestra conversación:

=== "Código"
    ```python
    chain.invoke("Hola, mi nombre es Juan, ¿cómo estás?")
    ```

=== "Salida"
    ```bash
    > Entering new ConversationChain chain...
    Prompt after formatting:
    The following is a friendly conversation between a human and an AI. The AI is talkative and provides lots of specific details from its context. If the AI does not know the answer to a question, it truthfully says it does not know.

    Current conversation:

    Human: Hola, mi nombre es Juan, ¿cómo estás?
    AI:

    > Finished chain.

        {'input': 'Hola, mi nombre es Juan, ¿cómo estás?',
    'history': '',
    'response': '¡Hola, Juan! Estoy muy bien, gracias por preguntar. Soy una inteligencia artificial, así que no tengo emociones como los humanos, pero estoy aquí para ayudarte y conversar contigo. ¿En qué puedo asistirte hoy?'}
    ```
Si pregunto aluna cosa adicional, por ejeplo cuando es 2 + 2. No olvidará mi nombre:

=== "Código"
    ```python
    chain.invoke("cuanto es 2 + 2?")
    chain.invoke("cual es mi nombre?")
    ```

=== "Salida"
    ```bash
    > Entering new ConversationChain chain...
    Prompt after formatting:
    The following is a friendly conversation between a human and an AI. The AI is talkative and provides lots of specific details from its context. If the AI does not know the answer to a question, it truthfully says it does not know.

    Current conversation:
    Human: Hola, mi nombre es Juan, ¿cómo estás?
    AI: ¡Hola, Juan! Estoy muy bien, gracias por preguntar. Soy una inteligencia artificial, así que no tengo emociones como los humanos, pero estoy aquí para ayudarte y conversar contigo. ¿En qué puedo asistirte hoy?
    Human: cuanto es 2 + 2?
    AI: 2 + 2 es igual a 4. Es una de las operaciones matemáticas más básicas y es un buen ejemplo de cómo funcionan las sumas. Si tienes más preguntas de matemáticas o cualquier otro tema, estaré encantado de ayudarte.
    Human: ¿Cuál es mi nombre?
    AI:

    {'input': '¿Cuál es mi nombre?',
    'history': 'Human: Hola, mi nombre es Juan, ¿cómo estás?\nAI: ¡Hola, Juan! Estoy muy bien, gracias por preguntar. Soy una inteligencia artificial, así que no tengo emociones como los humanos, pero estoy aquí para ayudarte y conversar contigo. ¿En qué puedo asistirte hoy?\nHuman: cuanto es 2 + 2?\nAI: 2 + 2 es igual a 4. Es una de las operaciones matemáticas más básicas y es un buen ejemplo de cómo funcionan las sumas. Si tienes más preguntas de matemáticas o cualquier otro tema, estaré encantado de ayudarte.',
    'response': 'Tu nombre es Juan. Me lo dijiste al comienzo de nuestra conversación. Si tienes más preguntas o necesitas ayuda con algo más, no dudes en decírmelo.'}
    ```
La IA responde que *Tu nombre es Juan. Me lo dijiste al comienzo de nuestra conversación*. Lo cual no era posible en el módulo 1 cuando invocábamos el modelo sin memoria.

## La ventana de contexto

La memoria es un recurso costoso, pues los modelos tienen una capacidad limitada para guardar el contexto de las conversaciones. Esta característica es llamada **la ventana de contexto**. A medida que los modelos se han vuelto más avanzados, las ventanas de contexto ofrecidas son cada vez más grandes. 

- GPT-3: 4096 tokens
- GPT-3.5-turbo: 4096 tokens
- GPT-4: 8192 tokens (con una versión extendida de 32768 tokens)

Sin embargo, debemos tener en cuenta que el modelo siempre tratará de relacionar sus respuestas con el contexto de la conversación. Esto puede ser beneficioso, pero al mismo tiempo, puede introducir ruido a la conversación, y el modelo puede generar respuestas menos precisas si está distraído en un contexto muy amplio. Por este motivo, resulta conveniente tener la posibilidad de limitar la memoria que queremos dar a las aplicaciones asistidas por IA. Veremos ahora algunos tipos de memoria y sus usos.

## Tipos de Memoria en LangChain

LangChain ofrece varios tipos de memoria que se pueden utilizar para gestionar el contexto en las aplicaciones de inteligencia artificial

crearemos una pequena encapsulación para ilistrar el uso de las dos clases principales:

```python
class ChatBot:
    def __init__(self, memory):
        self.chat_model  =  ChatOpenAI(model_name= "gpt-4o", temperature= 0, streaming= True)
        self.memory = memory
        self.chain = ConversationChain(llm=self.chat_model, memory=self.memory, verbose=True)
        
# metodo para responder
    def pregunta(self, pregunta: str):        
        response = self.chain.predict(input = pregunta)
        print(response)
        return 0
```
## Tipos de Memoria en LangChain

**ConversationBufferMemory**: 
   - **Uso**: Esta memoria almacena todo el historial de la conversación sin ningún límite. Es útil cuando se desea mantener un registro completo de todas las interacciones previas, como lo hicimos en el ejemplo anterior.

   Por ejemplo, un chat con memoria ilimitada se crearía de la siguiente manera:

   ```python
   from langchain.memory import ConversationBufferMemory

   mucha_memoria = ConversationBufferMemory()
   genioBot = ChatBot(mucha_memoria)
   ```
**ConversationBufferWindowMemory**:
   - **Uso**: Similar a `ConversationBufferMemory`, pero con un parámetro llamado `window_size` que permite recordar solo un número fijo de interacciones recientes.

   Creamos un bot muy inteligente pero con memoria limitada usando `ConversationBufferWindowMemory`:

=== "Código"
    ```python
    from langchain.memory import ConversationBufferWindowMemory

    # Memoria con ventana de 1
    poca_memoria = ConversationBufferWindowMemory(k=1)

    # Instanciamos un bot con poca memoria
    olvidoBot = ChatBot(poca_memoria)
    olvidoBot.pregunta("Hola, mi nombre es Juan, ¿cómo estás?")
    olvidoBot.pregunta("¿Cuánto es 2 + 5?")
    olvidoBot.pregunta("¿Cuál es mi nombre?") # No sabe el nombre
    ```

=== "Salida"
    ```bash hl_lines="9"
    > Entering new ChatBot session...
    Human: Hola, mi nombre es Juan, ¿cómo estás?
    AI: ¡Hola, Juan! Estoy muy bien, gracias por preguntar. Soy una inteligencia artificial diseñada para ayudarte con información y responder a tus preguntas. ¿En qué puedo asistirte hoy?

    Human: ¿Cuánto es 2 + 5?
    AI: 2 + 5 es igual a 7. Si tienes más preguntas de matemáticas o cualquier otra cosa en mente, ¡estaré encantado de ayudarte!

    Human: ¿Cuál es mi nombre?
    AI: Lo siento, no tengo la capacidad de saber tu nombre a menos que me lo hayas dicho antes en esta conversación. Si quieres, puedes decírmelo ahora y lo recordaré para el resto de nuestra charla.
    ```
`olvidoBot` solo recordará una interacción a la vez debido a `window_size=1`. Cuando preguntamos "¿Cuál es mi nombre?", ya no recuerda la interacción donde le dijimos nuestro nombre, por lo que no puede recordarlo.

`ConversationSummaryMemory` almacena un resumen de la conversación, ideal para interacciones muy largas donde no es necesario retener todos los detalles. <!--  NOta terminar esta parte, poner un ejemplo aquí-->

!!! warning "Para tener en cuenta"
    Las guías más recientes recomiendan utilizar la función `trim_messages`, que proporciona una forma flexible de gestionar el historial de la conversación. Las funcionalidades aquí expuestas están siendo migradas a la plataforma de LangGraph, por lo que están fuera del alcance de este curso. Verás el mensaje `LangChainDeprecationWarning: Please see the migration guide at: https://python.langchain.com/docs/versions/migrating_memory/`.


# Entendiendo la Función `trim_messages`

La función `trim_messages` es una utilidad en LangChain diseñada para reducir el tamaño de un historial de chat a un número específico de tokens o mensajes, asegurando que el historial recortado siga siendo válido para los modelos de chat. Un historial de chat válido típicamente:

- **Comienza con:**
  - Un `HumanMessage`, o
  - Un `SystemMessage` seguido de un `HumanMessage`.

- **Termina con:**
  - Un `HumanMessage`, o
  - Un `ToolMessage` (común en conversaciones basadas en agentes).

 Al recortar mensajes más antiguos o menos relevantes, `trim_messages` ayuda a enfocar el modelo en el contexto reciente y pertinente, evitanto información que que pueda distraer al modelo (no simepre un contexto grande es mejor).

La función `trim_messages` opera tomando varios parámetros para controlar cómo se realiza el recorte:

| Parámetro        | Descripción                                                                                                                                                   |
|------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `messages`       | La secuencia de mensajes (por ejemplo, `HumanMessage`, `AIMessage`, `SystemMessage`) a recortar.                                                              |
| `max_tokens`     | El número máximo de tokens que deben tener los mensajes recortados.                                                                                           |
| `strategy`       | La estrategia de recorte: "first" (mantiene los primeros mensajes) o "last" (mantiene los más recientes, a menudo preferido para conversaciones).              |
| `token_counter`  | Una función o LLM utilizado para contar tokens en los mensajes, como el método de conteo de tokens incorporado de un LLM.                                      |
| `include_system` | Un booleano (por defecto: `False`) que especifica si se debe mantener el `SystemMessage` al principio si está presente.                                        |
| `allow_partial`  | Un booleano (por defecto: `False`) que permite dividir un mensaje si solo parte de él puede incluirse para cumplir con el límite de tokens.                    |

La función devuelve una lista de mensajes recortados que se ajustan a los límites especificados mientras mantienen la coherencia del contexto.

Consideremos un ejemplo práctico donde tenemos un chatbot que utiliza una cadena para procesar consultas de usuario. El chatbot ha estado funcionando durante varias interacciones, y el historial de chat se ha alargado. Necesitamos recortar el historial para ajustarlo a un límite de 100 tokens, manteniendo los mensajes más recientes y el `SystemMessage`.

Primero, definamos un historial de chat de ejemplo:

```python
from langchain_core.messages import AIMessage, HumanMessage, SystemMessage

chat_history = [
    SystemMessage(content="You are a helpful assistant."),
    HumanMessage(content="Hello, my name is Juan how are you?"),
    AIMessage(content="I'm doing well, thank you. How can I help you today?"),
    HumanMessage(content="Can you tell me about the weather today?"),
    AIMessage(content="Sure, let me check that for you. [checks weather] It's sunny with a high of 75 degrees."),
    HumanMessage(content="What's the capital of France?"),
    AIMessage(content="The capital of France is Paris."),
]
```

Ahora, usaremos `trim_messages` para recortar este historial a 100 tokens, manteniendo los mensajes más recientes e incluyendo el `SystemMessage`:

```python
from langchain_core.messages.utils import trim_messages
from langchain_openai import OpenAI

# Inicializar un LLM para el conteo de tokens
llm = OpenAI(model="gpt-3.5-turbo-1106")

# Recortar los mensajes
trimmed_history = trim_messages(
    messages=chat_history,
    max_tokens=100,
    strategy="last",
    token_counter=llm.get_num_tokens_from_messages,
    include_system=True, # para mantener el SystemMessage en la memoria
)
```

Si el conteo total de tokens del historial original excede los 100, el historial recortado podría verse así:

```python
[
    SystemMessage(content="You are a helpful assistant."),
    HumanMessage(content="What's the capital of France?"),
    AIMessage(content="The capital of France is Paris."),
]
```

Esto mantiene la conversación enfocada y dentro de la ventana de contexto especificada, pero no recordará nuestro nombre. A continuación, se presenta el código completo:

=== "Código"
    ```python
    from langchain_core.messages import AIMessage, HumanMessage, SystemMessage
    from langchain_core.messages.utils import trim_messages
    from langchain_core.runnables import RunnableLambda
    from langchain_core.prompts import ChatPromptTemplate
    from langchain_openai import ChatOpenAI

    # Inicializar el LLM
    # Reemplazar 'your-openai-api-key' con tu clave API real de OpenAI
    llm = ChatOpenAI(model="gpt-3.5-turbo-1106")

    # Definir la plantilla de prompt
    # La plantilla incluye un mensaje de sistema, un marcador de posición para el historial de chat y la entrada del usuario
    prompt = ChatPromptTemplate.from_messages([
        ("system", "You are a helpful assistant."),
        ("placeholder", "{chat_history}"),  # Importante: este es el marcador de posición para el historial de chat
        ("human", "{input}")
    ])

    # Definir una función para recortar mensajes
    # Esta función recorta el historial de chat para ajustarse a un límite de tokens especificado
    def trim_chat_history(messages):
        return trim_messages(
            messages=messages,
            max_tokens=100,  # Limitar a 100 tokens para ajustarse al contexto del modelo
            strategy="last",  # Conservar los mensajes más recientes
            token_counter=llm.get_num_tokens_from_messages,  # Usar el contador de tokens del LLM
            include_system=True  # Preservar el mensaje del sistema
        )
    ```

=== "Salida"
    ```bash
    LLM Response: You haven't provided your name. Can you please share it with me?
    ```

!!! tip "📖 Para aprender más"
    El `("placeholder", "{chat_history}")` en el `ChatPromptTemplate` es un componente clave del sistema de plantillas de prompt de LangChain, utilizado para definir un espacio en el prompt donde se insertará una secuencia de mensajes (por ejemplo, el historial de la conversación).

    === "Detalles"
        message type and its content or template.

        Tipos comunes de mensajes incluyen:
        - `("system", "...")`: Un mensaje del sistema que define el rol o las instrucciones del asistente.
        - `("human", "...")`: Un mensaje de entrada del usuario.
        - `("ai", "...")`: Un mensaje de respuesta del asistente.
        - `("placeholder", "{variable_name}")`: Un marcador de posición para una secuencia de mensajes o datos que se proporcionarán más tarde.

        La tupla `("placeholder", "{chat_history}")` indica que la variable llamada `chat_history` contendrá una lista de mensajes (por ejemplo, `SystemMessage`, `HumanMessage`, `AIMessage`) que se insertarán en esa posición en el prompt.
Esta cadena procesa una lista única de mensajes como contexto, pero no es propiamente una memoria de la conversación, ya que es fija. Idealmente, queremos que la memoria sea persistente, es decir, que recuerde un cierto número de interacciones. Para este propósito, podemos integrar `RunnableWithMessageHistory` a nuestra cadena para almacenar el historial de la conversación.

## Añadiendo Memoria Persistente para Chatbots

En el ejemplo anterior, la cadena procesa una lista plana de mensajes sin mantener el estado entre invocaciones:

```python
chain = (
    RunnableLambda(trim_chat_history)  # Recorta los mensajes directamente
    | prompt  # Formatea los mensajes recortados en el prompt
    | llm  # Genera una respuesta usando el LLM
)
```

Para añadir memoria persistente, envolveremos esta cadena con `RunnableWithMessageHistory`. Al implementar `RunnableWithMessageHistory` en LCEL, necesitamos definir una pipeline de base, y esta a su vez se define a partir de una plantilla de prompt y un LLM. Comencemos definiendo un nuevo prompt template; en este caso, usaremos `from_message`, para lo cual importaremos los módulos:

```python
from langchain.prompts import (
    SystemMessagePromptTemplate, 
    HumanMessagePromptTemplate,
    MessagesPlaceholder,
    ChatPromptTemplate
)
from langchain_openai import ChatOpenAI
from langchain_core.messages import AIMessage, HumanMessage
from langchain_core.chat_history import InMemoryChatMessageHistory
from langchain_core.runnables.history import RunnableWithMessageHistory
```

Dividiremos el proceso en los siguientes pasos:

1. **Crear la pipeline base.** Como lo hemos hecho antes, definimos el prompt y un LLM. Por ejemplo:

    ```python
    # Definir el prompt del sistema al estilo de Don Quijote
    system_prompt = "Eres un asistente útil que responde en una sola oración en español, al estilo de Don Quijote de la Mancha."

    # Crear la plantilla de prompt para el chat
    prompt_template = ChatPromptTemplate.from_messages([
        SystemMessagePromptTemplate.from_template(system_prompt),
        MessagesPlaceholder(variable_name="history"),
        HumanMessagePromptTemplate.from_template("{query}"),
    ])

    # Inicializar el LLM
    llm = ChatOpenAI(model_name="gpt-3.5-turbo", temperature=0)

    # Crear la pipeline base
    pipeline = prompt_template | llm
    ```

    Esta es nuestra pipeline base sobre la cual añadiremos memoria usando `RunnableWithMessageHistory` y de esta forma retener el historial de conversaciones. Necesitaremos configurar un almacén de historial de chat, puede ser un simple diccionario:

2. **Configurar la Gestión del Historial de Chat.** Para añadir memoria, necesitamos un mecanismo para almacenar y recuperar el historial de conversaciones para diferentes sesiones de usuario. Nuestro `RunnableWithMessageHistory` requiere que nuestra pipeline esté envuelta en un objeto `RunnableWithMessageHistory`. Este objeto necesita algunos parámetros de entrada. Uno de ellos es `get_session_history`, que requiere una función que devuelva un objeto `ChatMessageHistory` basado en un ID de sesión. Definimos esta función nosotros mismos:

    ```python
    # Definir el historial de chat
    chat_history = [
        HumanMessage(content="Saludos, mi nombre es Juan, ¿cómo te hallas?"),
        AIMessage(content="En verdad, me hallo en buen estado, y estoy presto a servirte, ¿cuál es tu deseo?"),
        HumanMessage(content="¿Puedes hablarme del tiempo que hoy nos acompaña?"),
        AIMessage(content="Ciertamente, el día se muestra soleado con un calor apacible que alcanza los 24 grados."),
        HumanMessage(content="¿Cuál es la capital de Francia?"),
        AIMessage(content="La capital de Francia es París, noble villa de gran renombre."),
    ]

    # Configurar la gestión del historial de chat con un historial predefinido
    chat_dict = {}
    session_id = "id_123"
    chat_dict[session_id] = InMemoryChatMessageHistory()
    chat_dict[session_id].add_messages(chat_history)

    def get_chat_history(session_id: str) -> InMemoryChatMessageHistory:
        if session_id not in chat_dict:
            chat_dict[session_id] = InMemoryChatMessageHistory()
        return chat_dict[session_id]
    ```

    `chat_dict` es un diccionario que asigna IDs de sesión a objetos `InMemoryChatMessageHistory`, los cuales almacenan secuencias de mensajes (por ejemplo, mensajes del usuario y respuestas del asistente).

    `get_chat_history` es una función que:
    - Toma un `session_id` (por ejemplo, "id_123") como entrada.
    - Verifica si existe un historial de chat para ese `session_id` en `chat_dict`.
    - Si no, crea un nuevo `InMemoryChatMessageHistory` y lo almacena en `chat_dict`.
    - Luego, devuelve el objeto de historial de chat correspondiente.

    !!! Tip "Para aprender más"
        Esta configuración asegura que cada sesión de usuario tenga su propio historial de chat aislado, permitiendo que múltiples usuarios interactúen simultáneamente sin mezclar conversaciones. Sin embargo, para entornos de producción, el almacenamiento en memoria no es ideal debido a su volatilidad (los datos se pierden al reiniciar la aplicación). Las alternativas incluyen:
        - `RedisChatMessageHistory` para almacenamiento escalable y persistente con Redis.
        - `PostgresChatMessageHistory` para almacenamiento respaldado por bases de datos.
        - `FileChatMessageHistory` para persistencia basada en archivos.

        Por ejemplo, para usar Redis, debes usar el paquete Redis (`%pip install --upgrade --quiet redis`), iniciar un servidor Redis (por ejemplo, a través de Docker) y definir un `get_message_history` con `RedisChatMessageHistory`, como se muestra en la documentación de LangChain sobre Integraciones de Memoria.

3. **Envolver la Pipeline Base con `RunnableWithMessageHistory`.** Para añadir memoria, envolvemos la pipeline base con `RunnableWithMessageHistory`, que gestiona el historial de chat para el `Runnable`:

    ```python
    # Envolver la pipeline con RunnableWithMessageHistory
    pipeline_with_history = RunnableWithMessageHistory(
        pipeline,
        get_session_history=get_chat_history,
        input_messages_key="query",
        history_messages_key="history"
    )
    ```

    Aquí, `pipeline` es el `Runnable` base que vamos a envolver (en este caso, `prompt_template | llm`). `get_session_history` es la función (`get_chat_history`) que devuelve el historial de chat para un ID de sesión dado. Esta función debe tomar un `session_id` y devolver una instancia de `BaseChatMessageHistory`. `input_messages_key` especifica la clave en el diccionario de entrada que contiene el mensaje actual del usuario. En el código, es "query", lo que significa que la entrada espera algo como `{"query": "What is my name again?"}`. `history_messages_key` especifica la clave donde el historial de la conversación debe ser inyectado en la entrada. En el código, es "history", lo que significa que el historial (una lista de objetos `BaseMessage`) se añade bajo esta variable de entrada.

    Al invocar, `RunnableWithMessageHistory` recupera el historial de chat para el ID de sesión usando `get_chat_history`. Aumenta el diccionario de entrada añadiendo el historial bajo la clave `history_messages_key` (por ejemplo, `{"query": "What is my name again?", "history": [...]}`). La entrada aumentada se pasa a la pipeline base, que incluye el historial en el prompt a través de `MessagesPlaceholder`. Después de que la pipeline genera una respuesta, el historial de chat se actualiza con el nuevo mensaje del usuario y la respuesta del asistente.

Y listo, nuestro historial de chat ahora se memorizará y recuperará cada vez que invoquemos nuestro runnable con el mismo ID de sesión.

=== "Código"
    ```python
    # Invocar la pipeline para demostrar la memoria
    result1 = pipeline_with_history.invoke(
        {"query": "¿Cuál es mi nombre?"},
        config={"session_id": "id_123"}
    )
    print(result1.content)  # Esperado: "Vuestro nombre, según me habéis dicho, es Juan."

    result2 = pipeline_with_history.invoke(
        {"query": "¿Qué más sabes de Francia?"},
        config={"session_id": "id_123"}
    )
    print(result2.content)
    ```

=== "Salida"
    ```bash
    Vuestro nombre es Juan, valeroso caballero que busca conocimiento y respuestas.
    Francia es tierra de exquisita gastronomía, arte refinado y hermosos paisajes, dignos de ser explorados y admirados.
    ```

Nuestro chat ahora tiene la capacidad de recordar todas las interacciones. Vemos:

```python
result2 = pipeline_with_history.invoke(
    {"query": "El nombre de mi madre es Maria"},
    config={"session_id": "id_123"}
)
print(result2.content) 

result2 = pipeline_with_history.invoke(
    {"query": "¿Quién es Maria en mi vida?"},
    config={"session_id": "id_123"}
)

chat = get_chat_history('id_123')

for msg in chat.messages:
    print(msg.content)
```

=== "Salida"
    ```bash
    Saludos, mi nombre es Juan, ¿cómo te hallas?
    En verdad, me hallo en buen estado, y estoy presto a servirte, ¿cuál es tu deseo?
    ¿Puedes hablarme del tiempo que hoy nos acompaña?
    Ciertamente, el día se muestra soleado con un calor apacible que alcanza los 24 grados.
    ¿Cuál es la capital de Francia?
    La capital de Francia es París, noble villa de gran renombre.
    ¿Cuál es mi nombre?
    Vuestro nombre es Juan, valeroso caballero que busca conocimiento y respuestas.
    ¿Qué más sabes de Francia?
    Francia es tierra de exquisita gastronomía, arte refinado y hermosos paisajes, dignos de ser explorados y admirados.
    El nombre de mi madre es Maria
    Vuestra madre, María, posee un nombre tan puro y bello como el de la Virgen Santa.
    ¿Quién es Maria?
    María es un nombre común entre las mujeres, pero también es el nombre de la madre de Jesús, la Virgen María, figura importante en la religión católica.
    ¿Quién es Maria en mi vida?
    María, en vuestra vida, es la mujer que os dio la vida, os cuidó con amor y os guió en vuestro camino, como una luz en la oscuridad.
    ```

!!! tip "📖 Para aprender más"
    Hasta aquí hemos reproducido el comportamiento de la clase `ConversationBufferMemory` que describimos al principio. Sin embargo, esta clase será deprecada en las versiones futuras de LangChain. Para ver cómo crear *wrappers* de los demás tipos de memoria, puedes consultar el siguiente material:
    **Artículo**: Introducción a los Tipos de Memoria en LangChain  
    **URL**: [https://www.aurelio.ai/learn/langchain-memory-types](https://www.aurelio.ai/learn/langchain-memory-types)

¡Felicidades por llegar al final del módulo 2! Has aprendido cómo crear cadenas con memoria utilizando el LCEL. Ahora estás en capacidad de crear chatbots funcionales. Te invito a realizar la actividad de aprendizaje para que pongas en práctica lo aprendido.

 **Glosario**

- **Configurable runnables**: En LangChain, son funciones ejecutables que pueden personalizarse dinámicamente en tiempo de ejecución utilizando un objeto `RunnableConfig`. Esto permite pasar parámetros como el nombre de la ejecución, etiquetas o metadatos para controlar el comportamiento, como los límites de concurrencia o recursión.

- **Context window**: La cantidad máxima de tokens de entrada (texto, datos, etc.) que un modelo de chat puede procesar en una sola interacción, determinada por la arquitectura del modelo.

- **langchain**: Un paquete de Python que proporciona componentes de alto nivel para construir aplicaciones con modelos de lenguaje, como cadenas preconstruidas y herramientas para tareas comunes.

- **langchain-community**: Una colección de componentes e integraciones contribuidas por la comunidad para LangChain, que extiende su funcionalidad con herramientas de terceros.

- **langchain-core**: El paquete fundamental de LangChain, que contiene interfaces centrales, abstracciones base e implementaciones en memoria para construir cadenas y funciones ejecutables.

- **langgraph**: Una extensión de LangChain para orquestar flujos de trabajo y pipelines complejos, permitiendo una gestión avanzada de estado y procesos de múltiples pasos.

- **langserve**: Una herramienta para desplegar funciones ejecutables de LangChain como endpoints de API REST utilizando FastAPI. Principalmente soporta funciones ejecutables de LangChain, con compatibilidad limitada para LangGraph.

- **Managing chat history**: Métodos y técnicas para almacenar, recuperar y mantener el contexto conversacional a través de múltiples interacciones en una aplicación basada en chat.

- **RunnableConfig**: Un objeto de configuración en LangChain para pasar parámetros de tiempo de ejecución a funciones ejecutables, incluyendo `run_name`, `run_id`, etiquetas, metadatos, `max_concurrency`, `recursion_limit` y otras configuraciones personalizables.



## Evidencia de Aprendizaje                 


| **Módulo 2** | **Cadenas y Memoria** |
|--------------|-------------------------------------------------------------|
| **EA1.**     |   **Generación de Informes de Salud Utilizando Archivos CSV **|


En este proyecto practicarás el uso de cadenas para desrrollar un sistema que reliza tareas secuenciales y ramificadas, cargarás los resitado de estas operacones en el bufer de memoria de un chatbot.
## Instrucciones
Descarga el archivo [healthcare_report.csv](../assets/resources/healthcare_report.csv) proporcionado y, usando LECL, desarrolla un sistema capaz de:
1. **Procesar un informe de salud original en español**: lee el informe desde el archivo CSV.
2. **Traducir el informe al inglés**: usa una cadena para traducir el texto.
3. **Resumir el informe traducido**: genera un resumen breve en inglés.
4. **Extraer indicadores clave de salud del resumen**: identifica elementos clave (e.g., síntomas, duración).
5. **Generar un plan de tratamiento basado en los indicadores clave**: propón pasos de tratamiento.
6. **Detectar el idioma original del informe**: determina si el informe original está en español.
7. **Generar una recomendación de seguimiento en el idioma detectado**: devuelve una recomendación en español.

Finalmete carga el infome médico del paciente en la memoria y crea un chat bot que esté en capacidad de responder preguntas sobre el tratamiento indicado. Demuestra su uso con algunas llamas al chat


---

Guarda los documentos con la siguiente nomenclatura:

- **Apellido_Nombre del estudiante.ipynb**  
**Ejemplo:**  
- López_Karla.ipynb

Finalmente, haz clic en el botón **Cargar Tarea**, sube tu archivo y presiona el botón **Enviar** para remitirlo a tu profesor con el fin de que lo evalúe y retroalimente. |

!!! tip "📖 Nota"
    Conoce los criterios de evaluación de esta evidencia de aprendizaje consultando la rúbrica que encontrarás a continuación.

| **Criterios**             | **Ponderación** |                       |                       |                       |                       | **Totales** |
|---------------------------|------------------|-----------------------|-----------------------|-----------------------|-----------------------|------------|
|                           | **70**           | **50**                | **5**                 | **0**                 |                       |            |
| **Calidad de las Soluciones** | Las soluciones a los ejercicios son correctas, demostrando una implementación adecuada de los conceptos y técnicas requeridos. El estudiante muestra un dominio completo de los temas abordados. | Aunque las soluciones no son completamente correctas, se observa un entendimiento y aplicación adecuada de los conceptos y técnicas involucradas. Hay evidencia de esfuerzo y comprensión de los temas. | Las soluciones presentadas son en su mayoría incorrectas. Se percibe un intento de resolver los ejercicios, pero hay una falta de comprensión de los conceptos y técnicas esenciales. | No realiza la entrega |                       | **70**      |
| **Calidad de la entrega** | El notebook es claro y fácil de seguir, incluyendo comentarios detallados sobre el funcionamiento del código en las celdas Markdown, lo que facilita la comprensión de las soluciones propuestas. | El notebook no es particularmente fácil de leer, pero aún así incluye comentarios que explican el funcionamiento del código en las celdas Markdown, mostrando un esfuerzo por aclarar la lógica detrás del código. | El notebook carece de comentarios acerca del funcionamiento del código en las celdas Markdown, lo que dificulta la comprensión de las soluciones implementadas. | No realiza la entrega |                       | **20**      |
| **Tiempo de la entrega**  | La entrega se realiza a tiempo, cumpliendo con el plazo establecido para la presentación de la actividad. | La entrega se realiza con una semana de atraso. Aunque fuera del plazo original, se considera adecuada para evaluar el trabajo presentado. | La entrega se realiza con más de una semana de atraso, lo que indica un retraso significativo en la presentación de la actividad. | No realiza la entrega |                       | **10**      |
|                           |                  |                       |                       |                       | **Ponderación de la actividad** | **100 puntos** |

# Referencias

Aurelio AI. (s.f.). *LangChain Course*. Recuperado el 21 de mayo de 2025, de [https://www.aurelio.ai/course/langchain](https://www.aurelio.ai/course/langchain)

Chase, H., & Ng, A. (2023). *LangChain for LLM Application Development* [Curso en línea]. DeepLearning.AI. Disponible en [https://www.deeplearning.ai/short-courses/langchain-for-llm-application-development/](https://www.deeplearning.ai/short-courses/langchain-for-llm-application-development/)


---
# Lecturas y material complementario

## 📚 Lecturas recomendadas

### **Título:** *How Trellix Uses LangChain to Enhance Cybersecurity*  
**Autor:** [LangChain]  
**Fecha de recuperación:** 21 de mayo de 2025  
**URL:** [How Trellix Uses LangChain to Enhance Cybersecurity](https://blog.langchain.dev/customers-trellix/)
## 🎥 Videos recomendados

### **Título:** *LangChain: Prompts, Parsers and Chaining | for Beginners*  
**Autor:** [Anub Gupta on Learn4Tarakki]  
**URL:** [LangChain: Prompts, Parsers and Chaining | for Beginners](https://www.youtube.com/watch?v=FHhJYxuIIA0)  
Este video ofrece una introducción amigable para principiantes sobre cómo crear plantillas de prompts, utilizar parsers y encadenar componentes en LangChain.

### Título: Interrupt 2025 Keynote | Harrison Chase | LangChain
**Autor:** [LangChain]
**URL:** Interrupt 2025 Keynote | Harrison Chase | LangChain

Este video presenta la keynote de Harrison Chase en la conferencia Interrupt 2025 de LangChain, donde se discute la evolución de la ingeniería de agentes y la visión de la compañía para agentes inteligentes. Incluye reflexiones sobre la trayectoria de LangChain y anuncios de nuevas herramientas de desarrollo.




























