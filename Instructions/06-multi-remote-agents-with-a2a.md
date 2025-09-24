---
lab:
  title: "Conexión a agentes remotos con el protocolo\_A2A"
  description: "Use el protocolo\_A2A para colaborar con agentes remotos."
---

# Conexión a agentes remotos con el protocolo A2A

En este ejercicio, usará el servicio Agente de Azure AI con el protocolo A2A para crear agentes remotos sencillos que interactúen entre sí. Estos agentes ayudarán a los escritores técnicos a preparar sus entradas de blog para desarrolladores. Un agente de título generará un titular y un agente de esquema usará el título para desarrollar un esquema conciso para el artículo. Comencemos.

> **Sugerencia**: El código usado en este ejercicio se basa en el SDK de Fundición de IA de Azure para Python. Puedes desarrollar soluciones similares mediante los SDK de Microsoft .NET, JavaScript y Java. Consulta las [bibliotecas cliente de SDK de la Fundición de IA de Azure](https://learn.microsoft.com/azure/ai-foundry/how-to/develop/sdk-overview) para más información.

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
    - **Región**: *seleccione cualquiera (se recomienda Fundición de IA\*).

    > \* Algunos de los recursos de Azure AI están restringidos por cuotas de modelo regionales. En caso de que se alcance un límite de cuota más adelante en el ejercicio, es posible que tengas que crear otro recurso en otra región.

1. Selecciona **Crear** y espera a que tu proyecto se cree.
1. Si se te solicita, implementa un modelo **gpt-4o** mediante la opción de implementación *Estándar global* o *Estándar* (en función de la disponibilidad de la cuota).

    >**Nota**: Si la cuota está disponible, se puede implementar automáticamente un modelo base GPT-4o al crear el agente y el proyecto.

1. Cuando se crea el proyecto, se abre el sitio de prueba de agentes.

1. En el panel de navegación de la izquierda, selecciona **Información general** para ver la página principal del proyecto; que tiene este aspecto:

    ![Captura de pantalla de la página de información general de un proyecto de Fundición de IA de Azure.](./Media/ai-foundry-project.png)

1. Copia los valores del **punto de conexión del proyecto de Fundición de IA de Azure** en un Bloc de notas, ya que los usarás para conectarte a tu proyecto en una aplicación cliente.

## Creación de aplicación de A2A

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
   cd ai-agents/Labfiles/06-build-remote-agents-with-a2a/python
   ls -a -l
    ```

    Los archivos proporcionados incluyen lo siguiente:
    ```output
    python
    ├── outline_agent/
    │   ├── agent.py
    │   ├── agent_executor.py
    │   └── server.py
    ├── routing_agent/
    │   ├── agent.py
    │   └── server.py
    ├── title_agent/
    │   ├── agent.py
    |   ├── agent_executor.py
    │   └── server.py
    ├── client.py
    └── run_all.py
    ```

    Cada carpeta del agente contiene el código del agente de Azure AI y un servidor para hospedar el agente. El **agente de enrutamiento** se encarga de detectar y comunicarse con los agentes de **título** y **esquema**. El **cliente** permite a los usuarios enviar mensajes al agente de enrutamiento. `run_all.py` inicia todos los servidores y ejecuta el cliente.

### Configuración de la aplicación

1. En el panel de la línea de comandos de Cloud Shell, escribe el siguiente comando para instalar las bibliotecas que vas a usar:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-projects a2a-sdk
    ```

1. Escribe el siguiente comando para editar el archivo de configuración que se ha proporcionado:

    ```
   code .env
    ```

    El archivo se abre en un editor de código.

1. En el archivo de código, reemplaza el marcador de posición **your_project_endpoint** con el punto de conexión del proyecto (copiado de la página **Información general** del proyecto en el portal de la Función de IA de Azure) y asegúrate de que la variable MODEL_DEPLOYMENT_NAME está configurada con el nombre de la implementación de tu modelo (debe ser *gpt-4o*).
1. Después de reemplazar los marcadores de posición, usa el comando **CTRL+S** para guardar los cambios y después usa el comando **CTRL+Q** para cerrar el editor de código mientras mantienes abierta la línea de comandos de Cloud Shell.

### Creación de un agente reconocible

En esta tarea, creará el agente de título que ayuda a los escritores a crear titulares de tendencia para sus artículos. También definirá las aptitudes y la tarjeta del agente que necesita el protocolo A2A para que el agente se pueda detectar.

1. Vaya al directorio `title_agent`:

    ```
   cd title_agent
    ```

> **Sugerencia**: al agregar código, asegúrate de mantener la sangría correcta. Usa los niveles de sangría de comentario como guía.

1. Escribe el siguiente comando para editar el archivo de código que se ha proporcionado:

    ```
   code agent.py
    ```

1. Busque el comentario **Crear al cliente del agente** y agregue el código siguiente para conectarse al proyecto de Azure AI:

    > **Sugerencia**: ten cuidado de mantener el nivel de sangría correcto.

    ```python
   # Create the agents client
   self.client = AgentsClient(
       endpoint=os.environ['PROJECT_ENDPOINT'],
       credential=DefaultAzureCredential(
           exclude_environment_credential=True,
           exclude_managed_identity_credential=True
       )
   )
    ```

1. Busque el comentario **Crear el agente de título** y agregue el código siguiente para crear el agente:

    ```python
   # Create the title agent
   self.agent = self.client.create_agent(
       model=os.environ['MODEL_DEPLOYMENT_NAME'],
       name='title-agent',
       instructions="""
       You are a helpful writing assistant.
       Given a topic the user wants to write about, suggest a single clear and catchy blog post title.
       """,
   )
    ```

1. Busque el comentario **Crear una conversación para la sesión de chat** y agregue el código siguiente para crear la conversación de chat:

    ```python
   # Create a thread for the chat session
   thread = self.client.threads.create()
    ```

1. Busque el comentario **Enviar mensaje de usuario** y agregue este código para enviar el mensaje del usuario:

    ```python
   # Send user message
   self.client.messages.create(thread_id=thread.id, role=MessageRole.USER, content=user_message)
    ```

1. En el comentario **Crear y ejecutar el agente**, agregue el código siguiente para iniciar la generación de respuestas del agente:

    ```python
   # Create and run the agent
   run = self.client.runs.create_and_process(thread_id=thread.id, agent_id=self.agent.id)
    ```

    El código proporcionado en el resto del archivo procesará y devolverá la respuesta del agente. 

1. Guarde el archivo de código (*CTRL+S*). Ahora ya puede compartir las aptitudes y la tarjeta del agente con el protocolo A2A. 

1. Escribe el siguiente comando para editar el archivo `server.py` del agente de título  

    ```
   code server.py
    ```

1. Busque el comentario **Definir aptitudes del agente** y agregue el código siguiente para especificar la funcionalidad del agente:

    ```python
   # Define agent skills
   skills = [
       AgentSkill(
           id='generate_blog_title',
           name='Generate Blog Title',
           description='Generates a blog title based on a topic',
           tags=['title'],
           examples=[
               'Can you give me a title for this article?',
           ],
       ),
   ]
    ```

1. Busque el comentario **Crear tarjeta del agente** y agregue este código para definir los metadatos que hacen que el agente sea reconocible:

    ```python
   # Create agent card
   agent_card = AgentCard(
       name='AI Foundry Title Agent',
       description='An intelligent title generator agent powered by Azure AI Foundry. '
       'I can help you generate catchy titles for your articles.',
       url=f'http://{host}:{port}/',
       version='1.0.0',
       default_input_modes=['text'],
       default_output_modes=['text'],
       capabilities=AgentCapabilities(),
       skills=skills,
   )
    ```

1. Busque el comentario **Crear ejecutor del agente** y agregue el código siguiente para inicializar el ejecutor del agente mediante la tarjeta del agente:

    ```python
   # Create agent executor
   agent_executor = create_foundry_agent_executor(agent_card)
    ```

    El ejecutor del agente actuará como contenedor para el agente de título que ha creado.

1. Busque el comentario **Crear controlador de solicitudes** y agregue lo siguiente para controlar las solicitudes entrantes mediante el ejecutor:

    ```python
   # Create request handler
   request_handler = DefaultRequestHandler(
       agent_executor=agent_executor, task_store=InMemoryTaskStore()
   )
    ```

1. En el comentario **Crear aplicación de A2A**, agregue este código para crear la instancia de aplicación compatible con A2A:

    ```python
   # Create A2A application
   a2a_app = A2AStarletteApplication(
       agent_card=agent_card, http_handler=request_handler
   )
    ```
    
    Este código crea un servidor A2A que compartirá la información del agente de título y controlará las solicitudes entrantes de este agente mediante el ejecutor del agente de título.

1. Guarda el archivo de código (*CTRL+S*) cuando hayas terminado.

### Habilitación de mensajes entre los agentes

En esta tarea, se usa el protocolo A2A para permitir que el agente de enrutamiento envíe mensajes a los otros agentes. También permitirá que el agente de título reciba mensajes mediante la implementación de la clase del ejecutor del agente.

1. Vaya al directorio `routing_agent`:

    ```
   cd ../routing_agent
    ```

1. Escribe el siguiente comando para editar el archivo de código que se ha proporcionado:

    ```
   code agent.py
    ```

    El agente de enrutamiento actúa como orquestador que controla los mensajes de usuario y determina qué agente remoto debe procesar la solicitud.

    Cuando se recibe un mensaje del usuario, el agente de enrutamiento:
    - Inicia un subproceso de conversación.
    - Usa el método `create_and_process` para evaluar el agente que mejor coincida con el mensaje del usuario.
    - El mensaje se enruta al agente adecuado a través de HTTP mediante la función `send_message`.
    - El agente remoto procesa el mensaje y devuelve una respuesta.

    Por último, el agente de enrutamiento captura la respuesta y la devuelve al usuario en la conversación.

    Tenga en cuenta que el método `send_message` es asincrónico y debe esperarse a que la ejecución del agente se complete correctamente.

1. Agregue el código siguiente en el comentario **Recuperar el cliente A2A del agente remoto con el nombre del agente**:

    ```python
   # Retrieve the remote agent's A2A client using the agent name 
   client = self.remote_agent_connections[agent_name]
    ```

1. Busque el comentario **Construir la carga para enviar al agente remoto** y agregue el código siguiente:

    ```python
   # Construct the payload to send to the remote agent
   payload: dict[str, Any] = {
       'message': {
           'role': 'user',
           'parts': [{'kind': 'text', 'text': task}],
           'messageId': message_id,
       },
   }
    ```

1. Busque el comentario **Encapsular la carga en un objeto SendMessageRequest** y agregue el código siguiente:

    ```python
   # Wrap the payload in a SendMessageRequest object
   message_request = SendMessageRequest(id=message_id, params=MessageSendParams.model_validate(payload))
    ```

1. Agregue el código siguiente en el comentario **Enviar el mensaje al cliente del agente remoto y esperar la respuesta**:

    ```python
   # Send the message to the remote agent client and await the response
   send_response: SendMessageResponse = await client.send_message(message_request=message_request)
    ```


1. Guarda el archivo de código (*CTRL+S*) cuando hayas terminado. Ahora el agente de enrutamiento puede detectar y enviar mensajes al agente de título. Ahora se creará el código del ejecutor del agente para controlar esos mensajes entrantes desde el agente de enrutamiento.

1. Vaya al directorio `title_agent`:

    ```
   cd ../title_agent
    ```

1. Escribe el siguiente comando para editar el archivo de código que se ha proporcionado:

    ```
   code agent_executor.py
    ```

    La implementación de la clase `AgentExecutor` debe contener los métodos `execute` y `cancel`. El método cancel se ha proporcionado automáticamente. El método `execute` incluye un objeto `TaskUpdater` que administra eventos y que indica al autor de la llamada cuando se completa la tarea. Ahora se agregará la lógica para la ejecución de tareas.

1. En el método `execute`, agregue el código siguiente en el comentario **Procesar la solicitud**:

    ```python
   # Process the request
   await self._process_request(context.message.parts, context.context_id, updater)
    ```

1. En el método `_process_request`, agregue el código siguiente en el comentario **Obtener el agente de título**:

    ```python
   # Get the title agent
   agent = await self._get_or_create_agent()
    ```

1. Agregue el código siguiente debajo del comentario **Actualizar el estado de la tarea**:

    ```python
   # Update the task status
   await task_updater.update_status(
       TaskState.working,
       message=new_agent_text_message('Title Agent is processing your request...', context_id=context_id),
   )
    ```

1. Busque el comentario **Ejecutar la conversación del agente** y agregue el código siguiente:

    ```python
   # Run the agent conversation
   responses = await agent.run_conversation(user_message)
    ```

1. Busque el comentario **Actualizar la tarea con las respuestas** y agregue el código siguiente:

    ```python
   # Update the task with the responses
   for response in responses:
       await task_updater.update_status(
           TaskState.working,
           message=new_agent_text_message(response, context_id=context_id),
       )
    ```

1. Busque el comentario **Marcar la tarea como completa** y agregue el código siguiente:

    ```python
   # Mark the task as complete
   final_message = responses[-1] if responses else 'Task completed.'
   await task_updater.complete(
       message=new_agent_text_message(final_message, context_id=context_id)
   )
    ```

    Ahora el agente de título se ha ajustado con un ejecutor del agente que usará el protocolo A2A para controlar los mensajes. ¡Excelente trabajo!

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
    cd ..
    python run_all.py
    ```
    
    La aplicación se ejecuta con las credenciales de la sesión de Azure autenticada para conectarse al proyecto y crear y ejecutar el agente. Debería ver algunos resultados de cada servidor a medida que se inicia.

1. Espere hasta que aparezca la solicitud de entrada y escriba un mensaje como el siguiente:

    ```
   Create a title and outline for an article about React programming.
    ```

    Después de unos instantes, debería ver una respuesta del agente con los resultados.

1. Escriba `quit` para salir del programa y detener los servidores.
    
## Resumen

En este ejercicio, ha usado el SDK del servicio Agente de Azure AI y el SDK de Python de A2A para crear una solución remota de varios agentes. Ha creado un agente compatible con A2A reconocible y ha configurado un agente de enrutamiento para acceder a las aptitudes del agente. También ha implementado un ejecutor de agente para procesar mensajes A2A entrantes y administrar tareas. ¡Excelente trabajo!

## Limpieza

Si has terminado de explorar Agente de servicio de IA de Azure, debes eliminar los recursos que has creado en este ejercicio para evitar incurrir en costes innecesarios de Azure.

1. Vuelve a la pestaña del explorador que contiene Azure Portal (o vuelve a abrir [Azure Portal](https://portal.azure.com) en `https://portal.azure.com` en una nueva pestaña del explorador) y mira el contenido del grupo de recursos donde implementó los recursos usados en este ejercicio.
1. Selecciona **Eliminar grupo de recursos** en la barra de herramientas.
1. Escribe el nombre del grupo de recursos y confirma que deseas eliminarlo.
