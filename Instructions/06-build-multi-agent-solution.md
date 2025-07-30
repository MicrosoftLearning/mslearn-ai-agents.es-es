---
lab:
  title: Desarrollo de una solución multiagente con Fundición de IA de Azure
  description: Aprende a configurar varios agentes para colaborar con el agente de servicio de Fundición de IA de Azure
---

# Desarrollo de una solución de varios agentes

En este ejercicio, crearás un proyecto que organiza varios agentes de IA mediante el agente de servicio de Fundición de IA de Azure. Diseñarás una solución de IA que ayude con la evaluación de prioridades de incidencias. Los agentes conectados evaluarán la prioridad de la incidencia, sugerirán la asignación de un equipo y determinarán el nivel de esfuerzo necesario para completar la incidencia. Comencemos.

> **Sugerencia**: El código usado en este ejercicio se basa en el SDK de Fundición de IA de Azure para Python. Puedes desarrollar soluciones similares mediante los SDK de Microsoft .NET, JavaScript y Java. Consulta las [bibliotecas cliente de SDK de la Fundición de IA de Azure](https://learn.microsoft.com/azure/ai-foundry/how-to/develop/sdk-overview) para más información.

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
    - **Región**: *selecciona cualquier ubicación compatible con los servicios de IA***\*

    > \* Algunos de los recursos de Azure AI están restringidos por cuotas de modelo regionales. En caso de que se alcance un límite de cuota más adelante en el ejercicio, es posible que tengas que crear otro recurso en otra región.

1. Selecciona **Crear** y espera a que se cree el proyecto, incluida la implementación del modelo gpt-4 que seleccionaste.
1. Cuando se cree el proyecto, el área de juegos de chat se abrirá automáticamente.

    > **Nota**: La configuración predeterminada de TPM para este modelo puede ser demasiado baja para este ejercicio. Un TPM más bajo ayuda a evitar el uso excesivo de la cuota disponible en la suscripción que estás usando. 

1. En el panel de navegación de la izquierda, selecciona **Modelos y puntos de conexión** y selecciona tu implementación **gpt-4o**.

1. Selecciona **Editar** y después aumenta el **Límite de velocidad de tokens por minuto**.

   > **NOTA**: 40 000 TPM es suficiente para los datos que se usan en este ejercicio. Si la cuota disponible es inferior a esta, podrás completar el ejercicio, pero es posible que tengas que esperar y volver a enviar indicaciones si se supera el límite de velocidad.

1. En el panel de navegación de la izquierda, selecciona **Información general** para ver la página principal del proyecto; que tiene este aspecto:

    ![Captura de pantalla de la página de información general de un proyecto de Fundición de IA de Azure.](./Media/ai-foundry-project.png)

1. Copia el valor del **punto de conexión del proyecto de la Fundición de IA de Azure** en un Bloc de notas, ya que lo usarás para conectarte a tu proyecto en una aplicación cliente.

## Creación de una aplicación cliente del agente de IA

Ahora estás listo para crear una aplicación cliente que defina los agentes y las instrucciones. Se proporciona código para ti en un repositorio de GitHub.

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
   cd ai-agents/Labfiles/06-build-multi-agent-solution/Python
   ls -a -l
    ```

    Los archivos proporcionados incluyen código de aplicación y un archivo para las opciones de configuración.

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

1. En el archivo de código, reemplaza el marcador de posición **your_project_endpoint** con el punto de conexión de tu proyecto (copiado de la página **Información general** del proyecto en el portal de la Fundición de IA de Azure), y el marcador de posición **your_model_deployment** con el nombre que has asignado a la implementación de tu modelo gpt-4o.

1. Después de reemplazar los marcadores de posición, usa el comando **CTRL+S** para guardar los cambios y, a continuación, usa el comando **CTRL+Q** para cerrar el editor de código mientras mantienes abierta la línea de comandos de Cloud Shell.

### Crear agentes de IA

Ahora ya estás listo para crear los agentes para la solución de varios agentes. Comencemos.

1. Escribe el siguiente comando para editar el archivo **agent_triage.py**:

    ```
   code agent_triage.py
    ```

1. Revisa el código del archivo; ten en cuenta que contiene cadenas para cada nombre de agente e instrucciones.

1. Busca el comentario **Agregar referencias** y agrega el código siguiente para importar las clases que necesitarás:

    ```python
    # Add references
    from azure.ai.agents import AgentsClient
    from azure.ai.agents.models import ConnectedAgentTool, MessageRole, ListSortOrder, ToolSet, FunctionTool
    from azure.identity import DefaultAzureCredential
    ```

1. En el comentario **Instrucciones para el agente principal**, escribe el código siguiente:

    ```python
    # Instructions for the primary agent
    triage_agent_instructions = """
    Triage the given ticket. Use the connected tools to determine the ticket's priority, 
    which team it should be assigned to, and how much effort it may take.
    """
    ```

1. Busca el comentario **Crear el agente de prioridad en el servicio Agente de Azure AI** y agrega el código siguiente para crear un agente de Azure AI.

    ```python
    # Create the priority agent on the Azure AI agent service
    priority_agent = agents_client.create_agent(
        model=model_deployment,
        name=priority_agent_name,
        instructions=priority_agent_instructions
    )
    ```

    Este código crea la definición del agente en el cliente de agentes de Azure AI.

1. Busca el comentario **Crear una herramienta de agente conectado para el agente de prioridad** y agrega el código siguiente:

    ```python
    # Create a connected agent tool for the priority agent
    priority_agent_tool = ConnectedAgentTool(
        id=priority_agent.id, 
        name=priority_agent_name, 
        description="Assess the priority of a ticket"
    )
    ```

    Ahora vamos a crear los otros agentes de evaluación de prioridades.

1. En el comentario **Crear el agente de equipo y la herramienta conectada**, agrega el código siguiente:
    
    ```python
    # Create the team agent and connected tool
    team_agent = agents_client.create_agent(
        model=model_deployment,
        name=team_agent_name,
        instructions=team_agent_instructions
    )
    team_agent_tool = ConnectedAgentTool(
        id=team_agent.id, 
        name=team_agent_name, 
        description="Determines which team should take the ticket"
    )
    ```

1. En el comentario **Cree el agente de esfuerzo y la herramienta conectada**, agrega el código siguiente:
    
    ```python
    # Create the effort agent and connected tool
    effort_agent = agents_client.create_agent(
        model=model_deployment,
        name=effort_agent_name,
        instructions=effort_agent_instructions
    )
    effort_agent_tool = ConnectedAgentTool(
        id=effort_agent.id, 
        name=effort_agent_name, 
        description="Determines the effort required to complete the ticket"
    )
    ```


1. En el comentario **Crear un agente principal con las herramientas del agente conectado**, agrega el código siguiente:
    
    ```python
    # Create a main agent with the Connected Agent tools
    agent = agents_client.create_agent(
        model=model_deployment,
        name="triage-agent",
        instructions=triage_agent_instructions,
        tools=[
            priority_agent_tool.definitions[0],
            team_agent_tool.definitions[0],
            effort_agent_tool.definitions[0]
        ]
    )
    ```

1. Busca el comentario **Crear subproceso para la sesión de chat** y agrega el código siguiente:
    
    ```python
    # Create thread for the chat session
    print("Creating agent thread.")
    thread = agents_client.threads.create()
    ```


1. En el comentario **Crear solicitud de incidencia**, agrega el código siguiente:
    
    ```python
    # Create the ticket prompt
    prompt = "Users can't reset their password from the mobile app."

    ```

1. En el comentario **Enviar una solicitud al agente**, agrega el código siguiente:
    
    ```python
    # Send a prompt to the agent
    message = agents_client.messages.create(
        thread_id=thread.id,
        role=MessageRole.USER,
        content=prompt,
    )
    ```

1. En el comentario **Crear y procesar agente se ejecuta en subproceso con herramientas**, agrega el código siguiente:
    
    ```python
    # Create and process Agent run in thread with tools
    print("Processing agent thread. Please wait.")
    run = agents_client.runs.create_and_process(thread_id=thread.id, agent_id=agent.id)
    ```


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
   python agent_triage.py
    ```

    Deberías ver un resultado parecido a los siguiente:

    ```output
    Creating agent thread.
    Processing agent thread. Please wait.

    MessageRole.USER:
    Users can't reset their password from the mobile app.

    MessageRole.AGENT:
    ### Ticket Assessment

    - **Priority:** High — This issue blocks users from resetting their passwords, limiting access to their accounts.
    - **Assigned Team:** Frontend Team — The problem lies in the mobile app's user interface or functionality.
    - **Effort Required:** Medium — Resolving this problem involves identifying the root cause, potentially updating the mobile app functionality, reviewing API/backend integration, and testing to ensure compatibility across Android/iOS platforms.

    Cleaning up agents:
    Deleted triage agent.
    Deleted priority agent.
    Deleted team agent.
    Deleted effort agent.
    ```

    Puedes intentar modificar la solicitud mediante un escenario diferente de incidencia para ver cómo colaboran los agentes. Por ejemplo, "Investigar errores ocasionales 502 desde el punto de conexión de búsqueda".

## Limpieza

Si has terminado de explorar Agente de servicio de IA de Azure, debes eliminar los recursos que has creado en este ejercicio para evitar incurrir en costes innecesarios de Azure.

1. Vuelve a la pestaña del explorador que contiene Azure Portal (o vuelve a abrir [Azure Portal](https://portal.azure.com) en `https://portal.azure.com` en una nueva pestaña del explorador) y mira el contenido del grupo de recursos donde implementó los recursos usados en este ejercicio.

1. Selecciona **Eliminar grupo de recursos** en la barra de herramientas.

1. Escribe el nombre del grupo de recursos y confirma que deseas eliminarlo.
