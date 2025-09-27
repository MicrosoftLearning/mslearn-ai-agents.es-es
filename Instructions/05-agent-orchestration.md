---
lab:
  title: Desarrollo de una solución multiagente con kernel semántico
  description: Aprende a configurar varios agentes para colaborar mediante el SDK de kernel semántico
---

# Desarrollo de una solución de varios agentes

En este ejercicio, practicará el uso del patrón de orquestación secuencial del SDK de kernel semántico. Creará una canalización sencilla de tres agentes que funcionan conjuntamente para procesar los comentarios de los clientes y sugerir los pasos siguientes. Creará los siguientes agentes:

- El agente generador de resumen condensará los comentarios sin procesar en una oración breve y neutral.
- El agente clasificador clasificará los comentarios como positivo, negativo o una solicitud de característica.
- Por último, el agente de acción recomendada recomendará un paso de seguimiento adecuado.

Aprenderá a usar el SDK de kernel semántico para desglosar un problema, dirigirlo a través de los agentes adecuados y generar resultados accionables. Comencemos.

> **Sugerencia**: El código usado en este ejercicio se basa en el SDK de kernel semántico para Python. Puedes desarrollar soluciones similares mediante los SDK para Microsoft .NET y Java. Consulta [Lenguajes semánticos de kernel admitidos](https://learn.microsoft.com/semantic-kernel/get-started/supported-languages) para obtener más información.

Este ejercicio debería tardar en completarse **30** minutos aproximadamente.

> **Nota**: algunas de las tecnologías que se usan en este ejercicio se encuentran en versión preliminar o en desarrollo activo. Puede que se produzcan algunos comportamientos, advertencias o errores inesperados.

## Implementación de un modelo en un proyecto de Fundición de IA de Azure

Comencemos con la implementación de un modelo en un proyecto de Fundición de IA de Azure.

1. En un explorador web, abre el [Portal de la Fundición de IA de Azure](https://ai.azure.com) en `https://ai.azure.com` e inicia sesión con tus credenciales de Azure. Cierra las sugerencias o paneles de inicio rápido que se abran la primera vez que inicias sesión y, si es necesario, usa el logotipo de **Fundición de IA de Azure** en la parte superior izquierda para navegar a la página principal, que es similar a la siguiente imagen (cierra el panel **Ayuda** si está abierto):

    ![Captura de pantalla del Portal de la Fundición de IA de Azure.](./Media/ai-foundry-home.png)

1. En la página principal, en la sección **Explorar modelos y funcionalidades**, busca el modelo `gpt-4o`, que usaremos en nuestro proyecto.
1. En los resultados de la búsqueda, selecciona el modelo **gpt-4o** para ver sus detalles y, a continuación, en la parte superior de la página del modelo, selecciona **Usar este modelo**.
1. Cuando se te pida que crees un proyecto, escribe un nombre válido para el proyecto y expande **Opciones avanzadas**.
1. Confirma los siguientes ajustes para tu proyecto:
    - **Recurso de Fundición de IA de Azure**: *un nombre válido para el recurso de Fundición de IA de Azure*
    - **Suscripción**: *suscripción a Azure*
    - **Grupo de recursos**: *crea o selecciona un grupo de recursos*
    - **Región**: *seleccione cualquiera (se recomienda Fundición de IA\*).

    > \* Algunos de los recursos de Azure AI están restringidos por cuotas de modelo regionales. En caso de que se alcance un límite de cuota más adelante en el ejercicio, es posible que tengas que crear otro recurso en otra región.

1. Selecciona **Crear** y espera a que se cree el proyecto, incluida la implementación del modelo gpt-4 que seleccionaste.

1. Cuando se cree el proyecto, el área de juegos de chat se abrirá automáticamente.

1. En el panel de navegación de la izquierda, selecciona **Modelos y puntos de conexión** y selecciona tu implementación **gpt-4o**.

1. En el panel **Configuración**, anota el nombre de la implementación del modelo; que debe ser **gpt-4o**. Para confirmarlo, mira la implementación en la página **Modelos y puntos de conexión** (simplemente abre esa página en el panel de navegación de la izquierda).
1. En el panel de navegación de la izquierda, selecciona **Información general** para ver la página principal del proyecto; que tiene este aspecto:

    ![Captura de pantalla de los detalles de un proyecto de Azure AI en el Portal de la Fundición de IA de Azure.](./Media/ai-foundry-project.png)

## Creación de una aplicación cliente del agente de IA

Ahora estás listo para crear una aplicación cliente que defina un agente y una función personalizada. Se proporciona código para ti en un repositorio de GitHub.

### Preparación del entorno

1. Abre una nueva pestaña del explorador (mantén el Portal de la Fundición de IA de Azure abierto en la pestaña existente). En la nueva pestaña, explora [Azure Portal](https://portal.azure.com) en `https://portal.azure.com` e inicia sesión con tus credenciales de Azure, si se te solicita.

    Cierra las notificaciones de bienvenida para ver la página principal de Azure Portal.

1. Usa el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, para lo que deberás seleccionar un entorno de ***PowerShell*** sin almacenamiento en tu suscripción.

    Cloud Shell proporciona una interfaz de la línea de comandos en un panel situado en la parte inferior de Azure Portal. Puedes cambiar el tamaño o maximizar este panel para facilitar el trabajo.

    > **Nota**: si has creado anteriormente una instancia de Cloud Shell que usa un entorno de *Bash*, cámbiala a ***PowerShell***.

1. En la barra de herramientas de Cloud Shell, en el menú **Configuración**, selecciona **Ir a la versión clásica** (esto es necesario para usar el editor de código).

    **<font color="red">Asegúrate de que has cambiado a la versión clásica de Cloud Shell antes de continuar.</font>**

1. En el panel de Cloud Shell, escribe los siguientes comandos para clonar el repositorio de GitHub que contiene los archivos de código de este ejercicio (escribe el comando o cópialo en el Portapapeles y haz clic con el botón derecho en la línea de comandos y pega como texto sin formato):

    ```
   rm -r ai-agents -f
   git clone https://github.com/MicrosoftLearning/mslearn-ai-agents ai-agents
    ```

    > **Sugerencia**: al escribir comandos en CloudShell, la salida puede ocupar una gran cantidad del búfer de pantalla y el cursor en la línea actual puede estar atenuado. Puedes despejar la pantalla al escribir el comando `cls` para que te resulte más fácil centrarte en cada tarea.

1. Cuando se haya clonado el repositorio, escribe el siguiente comando para cambiar el directorio de trabajo a la carpeta que contiene los archivos de código y enumerarlos todos.

    ```
   cd ai-agents/Labfiles/05-agent-orchestration/Python
   ls -a -l
    ```

    Los archivos proporcionados incluyen código de aplicación y un archivo para las opciones de configuración.

### Configuración de la aplicación

1. En el panel de la línea de comandos de Cloud Shell, escribe el siguiente comando para instalar las bibliotecas que vas a usar:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity semantic-kernel --upgrade
    ```

    > **Nota**: Con la instalación del *kernel semántico*, se instala automáticamente una versión compatible con el kernel semántico de *azure-ai-projects*.

1. Escribe el siguiente comando para editar el archivo de configuración que se ha proporcionado:

    ```
   code .env
    ```

    El archivo se abre en un editor de código.

1. En el archivo de código, reemplace el marcador de posición **your_openai_endpoint** por el punto de conexión de Azure Open AI del proyecto (copiado de la página **Información general** del proyecto en el Portal de la Fundición de IA de Azure en **Azure OpenAI**). Reemplace el marcador de posición **your_openai_api_key** por la clave de API del proyecto y asegúrese de que la variable MODEL_DEPLOYMENT_NAME esté establecida en el nombre de implementación del modelo (que debe ser *gpt-4o*).

1. Después de reemplazar los marcadores de posición, usa el comando **CTRL+S** para guardar los cambios y, a continuación, usa el comando **CTRL+Q** para cerrar el editor de código mientras mantienes abierta la línea de comandos de Cloud Shell.

### Crear agentes de IA

Ahora ya estás listo para crear los agentes para la solución de varios agentes. Comencemos.

1. Escriba el siguiente comando para editar el archivo **agents.py**:

    ```
   code agents.py
    ```

1. En la parte superior del archivo en el comentario **Agregar referencias**, agregue el código siguiente para hacer referencia a los espacios de nombres de las bibliotecas que necesitará para implementar el agente:

    ```python
   # Add references
   import asyncio
   from semantic_kernel.agents import Agent, ChatCompletionAgent, SequentialOrchestration
   from semantic_kernel.agents.runtime import InProcessRuntime
   from semantic_kernel.connectors.ai.open_ai import AzureChatCompletion
   from semantic_kernel.contents import ChatMessageContent
    ```


1. En la función **get_agents**, agregue el código siguiente en el comentario **Crear un agente de resumen**:

    ```python
   # Create a summarizer agent
   summarizer_agent = ChatCompletionAgent(
       name="SummarizerAgent",
       instructions="""
       Summarize the customer's feedback in one short sentence. Keep it neutral and concise.
       Example output:
       App crashes during photo upload.
       User praises dark mode feature.
       """,
       service=AzureChatCompletion(),
   )
    ```

1. Agregue el código siguiente en el comentario **Crear un agente clasificador**:

    ```python
   # Create a classifier agent
   classifier_agent = ChatCompletionAgent(
       name="ClassifierAgent",
       instructions="""
       Classify the feedback as one of the following: Positive, Negative, or Feature request.
       """,
       service=AzureChatCompletion(),
   )
    ```

1. Agregue el código siguiente en el comentario **Crear un agente de acción recomendado**:

    ```python
   # Create a recommended action agent
   action_agent = ChatCompletionAgent(
       name="ActionAgent",
       instructions="""
       Based on the summary and classification, suggest the next action in one short sentence.
       Example output:
       Escalate as a high-priority bug for the mobile team.
       Log as positive feedback to share with design and marketing.
       Log as enhancement request for product backlog.
       """,
       service=AzureChatCompletion(),
   )
    ```

1. Agregue el código siguiente en el comentario **Devolver una lista de agentes**:

    ```python
   # Return a list of agents
   return [summarizer_agent, classifier_agent, action_agent]
    ```

    El orden de los agentes de esta lista será el orden en que se seleccionen durante la orquestación.

## Creación de una orquestación secuencial

1. En la función **main**, busque el comentario **Inicializar la tarea de entrada** y agregue el código siguiente:
    
    ```python
   # Initialize the input task
   task="""
   I tried updating my profile picture several times today, but the app kept freezing halfway through the process. 
   I had to restart it three times, and in the end, the picture still wouldn't upload. 
   It's really frustrating and makes the app feel unreliable.
   """
    ```

1. En el comentario **Crear una orquestación secuencial**, agregue el código siguiente para definir una orquestación secuencial con una devolución de llamada de respuesta:

    ```python
   # Create a sequential orchestration
   sequential_orchestration = SequentialOrchestration(
       members=get_agents(),
       agent_response_callback=agent_response_callback,
   )
    ```

    `agent_response_callback` le permitirá ver la respuesta de cada agente durante la orquestación.

1. Agregue el código siguiente en el comentario **Crear un entorno de ejecución e iniciarlo**:

    ```python
   # Create a runtime and start it
   runtime = InProcessRuntime()
   runtime.start()
    ```

1. Agregue el código siguiente en el comentario **Invocar la orquestación con una tarea y el entorno de ejecución**:

    ```python
   # Invoke the orchestration with a task and the runtime
   orchestration_result = await sequential_orchestration.invoke(
       task=task,
       runtime=runtime,
   )
    ```

1. Agregue el código siguiente en el comentario **Esperar los resultados**:

    ```python
   # Wait for the results
   value = await orchestration_result.get(timeout=20)
   print(f"\n****** Task Input ******{task}")
   print(f"***** Final Result *****\n{value}")
    ```

    En este código, recuperará y mostrará el resultado de la orquestación. Si la orquestación no se completa dentro del tiempo de espera especificado, se producirá una excepción de tiempo de espera.

1. Busque el comentario **Detener el tiempo de ejecución cuando esté inactivo** y agregue el código siguiente:

    ```python
   # Stop the runtime when idle
   await runtime.stop_when_idle()
    ```

    Una vez completado el procesamiento, detenga el tiempo de ejecución para limpiar los recursos.

1. Usa el comando **CTRL+S** para guardar los cambios en el archivo de código. Puedes mantenerlo abierto (en caso de que tengas que editar el código para corregir los errores) o usar el comando **CTRL+Q** para cerrar el editor de código mientras mantienes abierta la línea de comandos de Cloud Shell.

### Inicie sesión en Azure y ejecuta la aplicación.

Ahora estás listo para ejecutar el código y ver cómo colaboran tus agentes de IA.

1. En el panel de línea de comandos de Cloud Shell, escribe el siguiente comando para iniciar sesión en Azure.

    ```
   az login
    ```

    **<font color="red">Debes iniciar sesión en Azure, aunque la sesión de Cloud Shell ya esté autenticada.</font>**

    > **Nota**: en la mayoría de los escenarios, el uso de *inicio de sesión de az* será suficiente. Sin embargo, si tienes suscripciones en varios inquilinos, es posible que tengas que especificar el inquilino mediante el parámetro *--tenant*. Consulta [Inicio de sesión en Azure de forma interactiva mediante la CLI de Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively) para obtener más información.

1. Cuando se te solicite, sigue las instrucciones para abrir la página de inicio de sesión en una nueva pestaña y escribe el código de autenticación proporcionado y las credenciales de Azure. A continuación, completa el proceso de inicio de sesión en la línea de comandos y selecciona la suscripción que contiene el centro de Fundición de IA de Azure si se te solicita.

1. Después de iniciar sesión, escribe el siguiente comando para ejecutar la aplicación:

    ```
   python agents.py
    ```

    Deberías ver un resultado parecido a los siguiente:

    ```output
    # SummarizerAgent
    App freezes during profile picture upload, preventing completion.
    # ClassifierAgent
    Negative
    # ActionAgent
    Escalate as a high-priority bug for the development team.

    ****** Task Input ******
    I tried updating my profile picture several times today, but the app kept freezing halfway through the process.
    I had to restart it three times, and in the end, the picture still wouldn't upload.
    It's really frustrating and makes the app feel unreliable.

    ***** Final Result *****
    Escalate as a high-priority bug for the development team.
    ```

1. Opcionalmente, puede intentar ejecutar el código mediante distintas entradas de tarea, como:

    ```output
    I use the dashboard every day to monitor metrics, and it works well overall. But when I'm working late at night, the bright screen is really harsh on my eyes. If you added a dark mode option, it would make the experience much more comfortable.
    ```
    ```output
    I reached out to your customer support yesterday because I couldn't access my account. The representative responded almost immediately, was polite and professional, and fixed the issue within minutes. Honestly, it was one of the best support experiences I've ever had.
    ```

## Resumen

En este ejercicio, ha practicado la orquestación secuencial con el SDK de kernel semántico, combinando varios agentes en un único flujo de trabajo simplificado. ¡Excelente trabajo!

## Limpieza

Si has terminado de explorar Agente de servicio de IA de Azure, debes eliminar los recursos que has creado en este ejercicio para evitar incurrir en costes innecesarios de Azure.

1. Vuelve a la pestaña del explorador que contiene Azure Portal (o vuelve a abrir [Azure Portal](https://portal.azure.com) en `https://portal.azure.com` en una nueva pestaña del explorador) y mira el contenido del grupo de recursos donde implementó los recursos usados en este ejercicio.

1. Selecciona **Eliminar grupo de recursos** en la barra de herramientas.

1. Escribe el nombre del grupo de recursos y confirma que deseas eliminarlo.
