<!--  Nombre de la Unidad __
--># Unidad 1. Introducción a la construcción de aplicaciones con LLMs

## Introducción a la unidad

Bienvenidos a la primera unidad. En esta unidad aprenderás, de manera general, cómo funciona un modelo de lenguaje.  
Comenzaremos usando la API de OpenAI y exploraremos cómo conectar sus servicios con nuestras aplicaciones.  
Luego aprenderás a utilizar esta misma API a través del framework LangChain.

Finalmente, introduciremos los aspectos fundamentales de la interacción con los LLMs usando LangChain: **prompts**, **templates** y **output parsers**.

Como actividad práctica, elaborarás un sistema asistido por IA para extraer datos de comentarios de usuarios en un e-commerce.

¡Comencemos!


<!--
Introducción a la unidad
Teniendo en cuenta que cada unidad es un saber específico, en la introducción se destaca la importancia y relevancia del saber que se abordará en función de los resultados de aprendizaje planteados. Describe brevemente cómo el tema central de la unidad se integra en el panorama más amplio del aprendizaje y de la vida cotidiana o profesional del estudiante. Su propósito es despertar la curiosidad y el interés del estudiante sobre los contenidos que explorará.

En definitiva, se trata de responder a las preguntas: ¿Qué va a aprender el estudiante? ¿Cómo lo va a aprender? ¿Para qué lo va a aprender?  




Recomendaciones:
Inicia presentando al estudiante cómo se relaciona el conocimiento de la unidad con su contexto. 
Incluye el propósito de la unidad y lo que el estudiante aprenderá mediante su estudio.
Vale la pena destacar algunos de los temas más importantes que se abordarán.
Procura no superar las 300 palabras (1500 caracteres) al redactar la introducción.

-->

<!-- Resultados de aprendizaje
Los objetos de aprendizaje se asumen como aquello que los estudiantes serán capaces de hacer a partir de lo que aprendieron a lo largo de la unidad.


Recomendaciones:
Formula máximo dos resultados por unidad. 
Asegúrate que tengan relación con los objetivos de aprendizaje planteados en la carta descriptiva. 
Redacta los resultados a partir de tres elementos: qué, cómo y para qué.
Recuerda que los resultados se establecen en función del aprendizaje, no de la enseñanza. 
Utiliza verbos conjugados en presente (describen la acción).  
Los resultados deben ser medibles y alcanzables. 




-->

## Resultados de aprendizaje


<!--Cronograma de actividades de la unidad  
Permite la proyección de los contenidos tanto teóricos como prácticos, la ubicación temporal dentro del curso y los porcentajes que corresponden a la evidencia de aprendizaje.


Recomendaciones:
Toma del cronograma general que realizaste (plantillas preliminares) las actividades que correspondan a la presente unidad.
Plantea una evidencia de aprendizaje por unidad, y otra más para el cierre del curso. 

 -->

## Cronograma de actividades de la unidad  

## Cronograma de actividades - Unidad 1
| Actividad de aprendizaje       | Evidencia de aprendizaje | Semana       | Ponderación |
|-------------------------------|---------------------------|--------------|--------------|
| Actividades de aprendizaje 1 y 2 | EA1                      | Semana 1 y 2 | 25%         |
| **Total**                      |                           |              | **25 %**     |



<!--
Desarrollo temático
Aquí comienza la elaboración del contenido que hará parte de la unidad temática. Para ello, es preciso identificar qué requiere el estudiante para aprender y comprender aquello que debe explorar, desarrollar nuevas habilidades, aplicar el conocimiento y cumplir con los resultados de aprendizaje. 

Los textos se construyen con cohesión y claridad, de manera que facilite al estudiante apropiarse del conocimiento de manera efectiva. Esta elaboración debe estar respaldada por enfoques didácticos, garantizando un proceso de aprendizaje sólido y bien fundamentado.




Recomendaciones:
Ten en cuenta las siguientes recomendaciones para desarrollar las temáticas de la actividad de aprendizaje:
Lee el documento “Manual del contenidista” en el cual encontrarás consejos para redactar los contenidos.
Ten a la mano el “Manual de redacción” para resolver dudas o inquietudes sobre el uso de las normas APA para citas y referencias. 
Cada unidad debe tener una cantidad mínima de 30 páginas de contenido temático. Esto equivale aproximadamente a 8500 palabras, en Arial 12, espaciado 1.0 y texto justificado. 
El desarrollo del contenido requiere un 70 % de producción propia y un 30 % para contenidos de terceros (fuentes primarias). Monitorea permanentemente tu documento con ayuda de la herramienta Turnitin para revisar el porcentaje de similitud.
Los textos e imágenes de terceros obligatoriamente se deben citar y referenciar, procurando que no superen el porcentaje exigido (30 %). Debes suministrar los enlaces de los recursos digitales empleados (PDF, sitios web, artículos online, videos, imágenes, etc.). Todos estos recursos deben ser de uso libre.
No incluir artículos, tesis, textos o documentos propios que han sido previamente publicados o presentados a otra institución. 
Los recursos como imágenes, infografías, ilustraciones, tablas, etc., no hacen parte de las 30 páginas del desarrollo de contenido.
Las figuras propias deben ser editables y se entregan en una carpeta aparte, cuidando que tengan el nombre y número correspondiente. 
Las fuentes se pueden tomar de bases de datos de suscripción como EBSCO o de uso libre como Redalyc y Google Académico, las cuales cuentas con licencia Creative Commons (LCC) para su reproducción (solicitar el acceso a los repositorios en caso de no tenerlo).
Organiza y jerarquiza los temas y subtemas numéricamente.

-->
## ¿Qué es un modelo de lenguaje?

Un modelo de lenguaje es un sistema basado en *deep learning* que encapsula información sobre uno o varios lenguajes. Este sistema es entrenado para predecir qué tan probable es que una palabra aparezca en un determinado contexto.

Por ejemplo, dado el contexto:

> "Mi plato favorito es el ____"

un modelo de lenguaje que codifique el español de Antioquia podría predecir "sancocho" con más frecuencia que "ajiaco".
### Tokens

La unidad básica de predicción de un modelo de lenguaje es el **token**, y el **tokenizador** es el software que utiliza el modelo para dividir los textos en tokens.

Por ejemplo, el tokenizador de GPT-4 divide la frase:

> "El sol está brillando intensamente"

de la siguiente manera:

<!--WARNING: El numerado automático de figuras no está funcionando. Arreglar-->

<figure>
  <img src="assets/images/tokenizer.png" alt="División en tokens de una frase en GPT-4" width="600">
  <figcaption>División en tokens de una frase utilizando el tokenizador de GPT-4. Fuente: <a href="https://platform.openai.com/tokenizer">OpenAI Tokenizer</a>.</figcaption>
</figure>

!!! warning "Para tener en cuenta"
    Hay varias razones por las que los modelos de lenguaje utilizan **tokens** en lugar de palabras completas o caracteres individuales.

    A diferencia de un simple carácter, un token permite dividir una palabra en componentes con significado propio. Por ejemplo, la palabra **"intensamente"** puede ser dividida por el tokenizador en "intens" y "amente", y cada uno de estos componentes aporta parte del significado de la palabra completa.

    Esto también implica que hay **menos tokens únicos que palabras únicas**, lo que hace que el vocabulario del modelo sea más pequeño y, por lo tanto, más eficiente.

    Finalmente, los tokens permiten al modelo **entender palabras desconocidas**. Por ejemplo, si se le presenta la palabra *"WhatsAppeando"*, el modelo puede inferir su significado a partir del contexto en que aparecen los tokens "WhatsApp" y "ando".
# ¿Qué son los grandes modelos de lenguaje (LLM)?

Lo que diferencia un **LLM** (Large Language Model) de un modelo de lenguaje tradicional es el **número de parámetros**. Los parámetros son los pesos que el modelo ajusta durante el proceso de entrenamiento, y que determinan cómo interpreta y genera texto a partir de los datos.

Por supuesto, el concepto de "grande" es relativo. ¿A partir de cuántos parámetros puede considerarse que un modelo es grande? Veámoslo así:

- El **GPT** lanzado por OpenAI en 2018 tenía **117 millones de parámetros**, y ya era considerado un modelo grande en su época.
- En 2019, **GPT-2** aumentó ese número a **1.5 billones de parámetros**.
- Hasta abril de 2025, el modelo de lenguaje más grande conocido públicamente es **GPT-4** de OpenAI, con aproximadamente **1.76 billones de parámetros**.

Es muy posible que en el futuro estos modelos hoy considerados **LLMs** sean vistos como simples modelos de lenguaje, a medida que la tecnología y los recursos computacionales avancen.

Es muy posible que en el futuro estos modelos hoy considerados **LLMs** sean vistos como simples modelos de lenguaje, a medida que la tecnología y los recursos computacionales avancen.

!!! warning "Para tener en cuenta"
    El crecimiento en la cantidad de parámetros no garantiza una mejora si **no hay suficientes datos** disponibles para el entrenamiento. Entrenar un modelo grande con un conjunto de datos pequeño puede causar **sobreajuste (overfitting)**, lo que significa que el modelo funciona bien con los datos de entrenamiento pero falla al generalizar a nuevos datos. Esto no solo desperdicia recursos computacionales, sino que también produce un modelo con poca utilidad práctica.

    Cuando no se cuenta con grandes volúmenes de datos, se pueden aplicar técnicas como:

    - **[Aprendizaje por transferencia (transfer learning)](https://www.tensorflow.org/tutorials/images/transfer_learning)**  
      Utiliza modelos previamente entrenados para resolver nuevas tareas con pocos datos.

    - **[Aumento de datos (data augmentation)](https://machinelearningmastery.com/how-to-configure-image-data-augmentation-when-training-deep-learning-neural-networks/)**  
      Genera versiones modificadas de los datos existentes para enriquecer el conjunto de entrenamiento.

    - **[Destilación de conocimiento (knowledge distillation)](https://pytorch.org/tutorials/beginner/knowledge_distillation_tutorial.html)**  
      Transfiere el conocimiento de un modelo grande (profesor) a uno más pequeño (estudiante) manteniendo un rendimiento competitivo.

    Estas estrategias permiten que modelos más pequeños logren mejor desempeño, aprovechando conocimiento preexistente o la generación sintética de datos.

## De ML Igeniringa a IA Ingering






<!--  ESTAS SON ALGUNAS ADAPTACIONES DE LAS ADMINICIONES USADAS EN LAS PLATILLAS DE LA IU -->

!!! warning "Para tener en cuenta"
    Asegúrate de evaluar los posibles sesgos en los datos antes de implementar un modelo de IA. Los sesgos no detectados pueden llevar a decisiones injustas, afectando la equidad y la confianza en la tecnología.


!!! tip "📖 Para aprender más"
    Si deseas conocer más sobre [tema], lee el siguiente material:
    Artículo de [nombre del artículo]:
    URL: [enlace]

<!-- Video -->

<div class="grid cards" markdown>

- :material-video-vintage:{ .lg .middle } **Video: Oportunidades en IA**  
  **Autor**: Andrew Ng  
  Para obtener una visión sobre el panorama actual de la inteligencia artificial, te invito a que veas la conferencia del profesor Andrew Ng *Oportunidades en IA*.  
  [Ver Video](https://www.youtube.com/watch?v=5p248yoa3oE){ .md-button .md-button--primary }

</div>

<div class="grid cards" markdown>

- :octicons-megaphone-16:{ .lg .middle } **Sabias que**  
  **Autor**: Andrew Ng  
  Para obtener una visión sobre el panorama actual de la inteligencia artificial, te invito a que veas la conferencia del profesor Andrew Ng *Oportunidades en IA*.  
  [Ver Video](https://www.youtube.com/watch?v=5p248yoa3oE){ .md-button .md-button--primary }

</div>

<div class="grid cards" markdown>

- :fontawesome-solid-gears:{ .lg .middle } **Reto formativo**  
  **Plantemiento**:
  Para obtener una visión sobre el panorama actual de la inteligencia artificial, te invito a que veas la conferencia del profesor Andrew Ng *Oportunidades en IA*.  
  [Ver Video](https://www.youtube.com/watch?v=5p248yoa3oE){ .md-button .md-button--primary }

</div>

<div class="grid cards" markdown>

- :simple-taketwointeractivesoftware:{ .lg .middle } **Recurso formativo**  
  **Plantemiento**:
  Para obtener una visión sobre el panorama actual de la inteligencia artificial, te invito a que veas la conferencia del profesor Andrew Ng *Oportunidades en IA*.  
  [Ver Video](https://www.youtube.com/watch?v=5p248yoa3oE){ .md-button .md-button--primary }

</div>

