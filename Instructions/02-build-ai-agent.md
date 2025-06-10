---
lab:
  title: Desarrollo de un agente de IA
  description: Usa Agente de servicio de IA de Azure para desarrollar un agente que use herramientas integradas.
---

# Desarrollo de un agente de IA

En este ejercicio, usarás Agente de servicio de IA de Azure para crear un agente sencillo que analice datos y cree gráficos. El agente usa la herramienta de *intérprete de código* integrada para generar dinámicamente el código necesario para crear gráficos como imágenes y, después, guarda las imágenes del gráfico resultantes.

Este ejercicio debería tardar en completarse **30** minutos aproximadamente.

> **Nota**: algunas de las tecnologías que se usan en este ejercicio se encuentran en versión preliminar o en desarrollo activo. Puede que se produzcan algunos comportamientos, advertencias o errores inesperados.

## Creación de un proyecto de Fundición de IA de Azure

Comencemos creando un proyecto de Fundición de IA de Azure.

1. En un explorador web, abre el [Portal de la Fundición de IA de Azure](https://ai.azure.com) en `https://ai.azure.com` e inicia sesión con tus credenciales de Azure. Cierra las sugerencias o paneles de inicio rápido que se abran la primera vez que inicias sesión y, si es necesario, usa el logotipo de **Fundición de IA de Azure** en la parte superior izquierda para navegar a la página principal, que es similar a la siguiente imagen (cierra el panel **Ayuda** si está abierto):

    ![Captura de pantalla del Portal de la Fundición de IA de Azure.](./Media/ai-foundry-home.png)

1. En la página principal, selecciona **Crear un agente**.
1. Cuando se te pida que crees un proyecto, escribe un nombre válido para el proyecto y expande **Opciones avanzadas**.
1. Confirma los siguientes ajustes para tu proyecto:
    - **Recurso de Fundición de IA de Azure**: *un nombre válido para el recurso de Fundición de IA de Azure*
    - **Suscripción**: *suscripción a Azure*
    - **Grupo de recursos**: *crea o selecciona un grupo de recursos*
    - **Región**: *selecciona cualquier ubicación compatible con los servicios de IA***\*

    > \* Algunos de los recursos de Azure AI están restringidos por cuotas de modelo regionales. En caso de que se alcance un límite de cuota más adelante en el ejercicio, es posible que tengas que crear otro recurso en otra región.

1. Selecciona **Crear** y espera a que tu proyecto se cree.
1. Cuando se cree el proyecto, el área de juegos de agentes se abrirá automáticamente para que puedas seleccionar o implementar un modelo:

    ![Captura de pantalla del área de juegos de agentes en Fundición de IA de Azure.](./Media/ai-foundry-agents-playground.png)

    >**Nota**: Un modelo base GPT-4o se implementa automáticamente al crear el agente y el proyecto.

1. En el panel de navegación de la izquierda, selecciona **Información general** para ver la página principal del proyecto; que tiene este aspecto:

    > **Nota**: si se muestra un error de *permisos insuficientes**, usa el botón **Reparar** para resolverlo.

    ![Captura de pantalla de la página de información general de un proyecto de Fundición de IA de Azure.](./Media/ai-foundry-project.png)

1. Copia los valores del **punto de conexión del proyecto de Fundición de IA de Azure** en un Bloc de notas, ya que los usarás para conectarte a tu proyecto en una aplicación cliente.

## Creación de una aplicación cliente del agente

Ahora estás listo para crear una aplicación cliente que use un agente. Parte del código se ha proporcionado en un repositorio de GitHub.

### Clonación del repositorio que contiene el código de la aplicación

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

1. Escribe el siguiente comando para cambiar el directorio de trabajo a la carpeta que contiene los archivos de código y enumerarlos todos.

    ```
   cd ai-agents/Labfiles/02-build-ai-agent/Python
   ls -a -l
    ```

    Los archivos proporcionados incluyen el código de aplicación, los valores de configuración y los datos.

### Configuración de la aplicación

1. En el panel de la línea de comandos de Cloud Shell, escribe el siguiente comando para instalar las bibliotecas que vas a usar:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-projects
    ```

1. Escribe el siguiente comando para editar el archivo de configuración que se ha proporcionado:

    ```
   code .env
    ```

    El archivo se abre en un editor de código.

1. En el archivo de código, reemplaza el marcador de posición **your_project_endpoint** por el punto de conexión de tu proyecto (copiado de la página **Información general** del proyecto del portal de la Fundición de IA de Azure).
1. Después de reemplazar los marcadores de posición, usa el comando **CTRL+S** para guardar los cambios y después usa el comando **CTRL+Q** para cerrar el editor de código mientras mantienes abierta la línea de comandos de Cloud Shell.

### Escritura de código para una aplicación de agente

> **Sugerencia**: al agregar código, asegúrate de mantener la sangría correcta. Usa los niveles de sangría de comentario como guía.

1. Escribe el siguiente comando para editar el archivo de código que se ha proporcionado:

    ```
   code agent.py
    ```

1. Revisa el código existente, que recupera los valores de configuración de aplicación y carga los datos de *data.txt* que se van a analizar. El resto del archivo incluye comentarios en los que agregarás el código necesario para implementar el agente de análisis de datos.
1. Busca el comentario **Agregar referencias** y agrega el código siguiente para importar las clases que necesitará para crear un agente de Azure AI que use la herramienta de intérprete de código integrada:

    ```python
   # Add references
   from azure.identity import DefaultAzureCredential
   from azure.ai.agents import AgentsClient
   from azure.ai.agents.models import FilePurpose, CodeInterpreterTool, ListSortOrder, MessageRole
    ```

1. Busca el comentario **Conectarse al cliente del agente** y agrega el código siguiente para conectarte al proyecto de Azure AI.

    > **Sugerencia**: ten cuidado de mantener el nivel de sangría correcto.

    ```python
   # Connect to the Agent client
   agent_client = AgentsClient(
       endpoint=project_endpoint,
       credential=DefaultAzureCredential
           (exclude_environment_credential=True,
            exclude_managed_identity_credential=True)
   )
   with agent_client:
    ```

    El código se conecta al proyecto de la Fundición de IA de Azure mediante las credenciales actuales de Azure. La última instrucción *with agent_client* inicia un bloque de código que define el ámbito del cliente, asegurándose de que se limpia cuando finaliza el código del bloque.

1. Busca el comentario **Cargar el archivo de datos y crear un CodeInterpreterTool**, en el bloque *with agent_client* y agrega el código siguiente para cargar el archivo de datos al proyecto y crear un CodeInterpreterTool que puede acceder a los datos en él:

    ```python
   # Upload the data file and create a CodeInterpreterTool
   file = agent_client.files.upload_and_poll(
        file_path=file_path, purpose=FilePurpose.AGENTS
   )
   print(f"Uploaded {file.filename}")

   code_interpreter = CodeInterpreterTool(file_ids=[file.id])
    ```
    
1. Busca el comentario **Definir un agente que use la CodeInterpreterTool** y agrega el código siguiente para definir un agente de IA que analice datos y pueda usar la herramienta de intérprete de código que definiste anteriormente:

    ```python
   # Define an agent that uses the CodeInterpreterTool
   agent = agent_client.create_agent(
        model=model_deployment,
        name="data-agent",
        instructions="You are an AI agent that analyzes the data in the file that has been uploaded. If the user requests a chart, create it and save it as a .png file.",
        tools=code_interpreter.definitions,
        tool_resources=code_interpreter.resources,
   )
   print(f"Using agent: {agent.name}")
    ```

1. Busca el comentario **Crear un hilo para la conversación** y agrega el código siguiente para iniciar un hilo en el que se ejecutará la sesión de chat con el agente:

    ```python
   # Create a thread for the conversation
   thread = agent_client.threads.create()
    ```
    
1. Ten en cuenta que la siguiente sección de código configura un bucle para que un usuario escriba una indicación y termine cuando el usuario escriba "salir".

1. Busca el comentario **Enviar una indicación al agente** y agrega el código siguiente para agregar un mensaje de usuario a la indicación (junto con los datos del archivo que se cargó anteriormente) y, después, ejecuta el hilo con el agente.

    ```python
   # Send a prompt to the agent
   message = agent_client.messages.create(
        thread_id=thread.id,
        role="user",
        content=user_prompt,
    )

   run = agent_client.runs.create_and_process(thread_id=thread.id, agent_id=agent.id)
     
   # Check the run status for failures
   if run.status == "failed":
        print(f"Run failed: {run.last_error}")
    ```

1. Busca el comentario **Mostrar la respuesta más reciente del agente** y agrega el código siguiente para recuperar los mensajes del hilo completado y mostrar el último mensaje enviado por el agente.

    ```python
   # Show the latest response from the agent
   last_msg = agent_client.messages.get_last_message_text_by_role(
       thread_id=thread.id,
       role=MessageRole.AGENT,
   )
   if last_msg:
       print(f"Last Message: {last_msg.text.value}")
    ```

1. Busca el comentario **Obtener el historial de conversaciones**, que se produce después de que finaliza el bucle y agrega el código siguiente para imprimir los mensajes del hilo de conversación; revirtiendo el orden para mostrarlos en secuencia cronológica.

    ```python
   # Get the conversation history
   print("\nConversation Log:\n")
   messages = agent_client.messages.list(thread_id=thread.id, order=ListSortOrder.ASCENDING)
   for message in messages:
       if message.text_messages:
           last_msg = message.text_messages[-1]
           print(f"{message.role}: {last_msg.text.value}\n")
    ```

1. Busca el comentario **Obtener los archivos generados** y agrega el código siguiente para obtener las anotaciones de ruta de acceso de archivo de los mensajes (lo que indica que el agente guardó un archivo en su almacenamiento interno) y copia los archivos en la carpeta de la aplicación. _NOTA_: Actualmente el sistema no dispone del contenido de la imagen.

    ```python
   # Get any generated files
   for msg in messages:
       # Save every image file in the message
       for img in msg.image_contents:
           file_id = img.image_file.file_id
           file_name = f"{file_id}_image_file.png"
           agent_client.files.save(file_id=file_id, file_name=file_name)
           print(f"Saved image file to: {Path.cwd() / file_name}")
    ```

1. Busca el comentario **Limpiar** y agrega el código siguiente para eliminar el agente y el hilo cuando ya no sea necesario.

    ```python
   # Clean up
   agent_client.delete_agent(agent.id)
    ```

1. Revisa el código mediante los comentarios para comprender cómo:
    - Se conecta al proyecto de Fundición de IA.
    - Carga el archivo de datos y crea una herramienta de intérprete de código que puede acceder a él.
    - Crea un nuevo agente que usa la herramienta de intérprete de código y tiene instrucciones explícitas para analizar los datos y crear gráficos como archivos .png.
    - Ejecuta un hilo con un mensaje de indicación del usuario junto con los datos que se van a analizar.
    - Comprueba el estado de la ejecución en caso de que se produzca un error.
    - Recupera los mensajes del hilo completado y muestra el último mensaje enviado por el agente.
    - Muestra el historial de conversaciones
    - Guarda cada archivo que se generó.
    - Elimina el agente y el hilo cuando ya no son necesarios.

1. Guarda el archivo de código (*CTRL+S*) cuando hayas terminado. También puede cerrar el editor de código (*CTRL+Q*); aunque es posible que desees mantenerlo abierto en caso de que tengas modificar el código que agregaste. En cualquier caso, no cierres el panel de línea de comandos de Cloud Shell.

### Inicie sesión en Azure y ejecuta la aplicación.

1. En el panel de línea de comandos de Cloud Shell, escribe el siguiente comando para iniciar sesión en Azure.

    ```
    az login
    ```

    **<font color="red">Debes iniciar sesión en Azure, aunque la sesión de Cloud Shell ya esté autenticada.</font>**

    > **Nota**: en la mayoría de los escenarios, el uso de *inicio de sesión de az* será suficiente. Sin embargo, si tienes suscripciones en varios inquilinos, es posible que tengas que especificar el inquilino mediante el parámetro *--tenant*. Consulta [Inicio de sesión en Azure de forma interactiva mediante la CLI de Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively) para obtener más información.
    
1. Cuando se te solicite, sigue las instrucciones para abrir la página de inicio de sesión en una nueva pestaña y escribe el código de autenticación proporcionado y las credenciales de Azure. A continuación, completa el proceso de inicio de sesión en la línea de comandos y selecciona la suscripción que contiene el centro de Fundición de IA de Azure si se te solicita.
1. Después de iniciar sesión, escribe el siguiente comando para ejecutar la aplicación:

    ```
    python agent.py
    ```
    
    La aplicación se ejecuta con las credenciales de la sesión de Azure autenticada para conectarse al proyecto y crear y ejecutar el agente.

1. Cuando se te solicite, visualiza los datos que la aplicación ha cargado desde el archivo de texto *data.txt*. Después, escribe una indicación como:

    ```
   What's the category with the highest cost?
    ```

    > **Sugerencia**: si se produce un error en la aplicación porque se supera el límite de velocidad. Espere unos segundos y vuelve a intentarlo. Si no hay cuota suficiente disponible en la suscripción, es posible que el modelo no pueda responder.

1. Visualiza la respuesta. A continuación, escribe otra indicación, esta vez solicitando un gráfico:

    ```
   Create a pie chart showing cost by category
    ```

    El agente debe usar selectivamente la herramienta de intérprete de código según sea necesario, en este caso para crear un gráfico basado en tu solicitud.

1. Puedes continuar la conversación si lo deseas. El hilo está *con estado*, por lo que conserva el historial de conversaciones, lo que significa que el agente tiene el contexto completo para cada respuesta. Cuando hayas terminado, escribe `quit`.
1. Revisa los mensajes de la conversación que se recuperaron del hilo y los archivos que se generaron.

1. Una vez finalizada la aplicación, usa el comando **download** de Cloud Shell para descargar cada archivo .png que se guardó en la carpeta de la aplicación. Por ejemplo:

    ```
   download ./<file_name>.png
    ```

    El comando download crea un vínculo emergente en la parte inferior derecha del explorador, que puedes seleccionar para descargar y abrir el archivo.

## Resumen

En este ejercicio, has usado el SDK de Agente de servicio de IA de Azure para crear una aplicación cliente que use un agente de IA. El agente usa la herramienta de intérprete de código integrada para ejecutar código dinámico que crea imágenes.

## Limpieza

Si has terminado de explorar Agente de servicio de IA de Azure, debes eliminar los recursos que has creado en este ejercicio para evitar incurrir en costes innecesarios de Azure.

1. Vuelve a la pestaña del explorador que contiene Azure Portal (o vuelve a abrir [Azure Portal](https://portal.azure.com) en `https://portal.azure.com` en una nueva pestaña del explorador) y mira el contenido del grupo de recursos donde implementó los recursos usados en este ejercicio.
1. Selecciona **Eliminar grupo de recursos** en la barra de herramientas.
1. Escribe el nombre del grupo de recursos y confirma que deseas eliminarlo.
