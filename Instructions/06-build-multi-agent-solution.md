---
lab:
  title: Desarrollo de una solución multiagente con Fundición de IA de Azure
  description: Aprende a configurar varios agentes para colaborar con el agente de servicio de Fundición de IA de Azure
---

# Desarrollo de una solución de varios agentes

En este ejercicio, crearás un proyecto que organiza varios agentes de IA mediante el agente de servicio de Fundición de IA de Azure. En este ejercicio, crearás un agente maestro de tareas responsable de guiar un grupo por un calabozo. El grupo está compuesto por agentes de inteligencia artificial conectados que representan un guerrero, un sanador y un explorador. El agente maestro de tareas recibe un escenario del usuario y delega las tareas a los miembros del grupo en consecuencia. Comencemos.

Este ejercicio debería tardar en completarse **30** minutos aproximadamente.

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

    > **Nota**: si se muestra un error de *permisos insuficientes**, usa el botón **Reparar** para resolverlo.

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

1. Escribe el siguiente comando para editar el archivo **agent_quest.py**:

    ```
   code agent_quest.py
    ```

1. Revisa el código del archivo; ten en cuenta que contiene cadenas para cada nombre de agente e instrucciones.

1. Busca el comentario **Agregar referencias** y agrega el código siguiente para importar las clases que necesitarás:

    ```python
    # Add references
    from azure.ai.agents import AgentsClient
    from azure.ai.agents.models import ConnectedAgentTool, MessageRole, ListSortOrder
    from azure.identity import DefaultAzureCredential
    ```

1. Busca el comentario **Crear el agente sanador en el agente de servicio de IA de Azure** y agrega el código siguiente para crear un agente de Azure AI.

    ```python
    # Create the healer agent on the Azure AI agent service
    healer_agent = agents_client.create_agent(
        model=model_deployment,
        name=healer_agent_name,
        instructions=healer_instructions
    )
    ```

    Este código crea la definición del agente en el cliente de agentes de Azure AI.

1. Busca el comentario **Crear una herramienta de agente conectado para el agente sanador** y agrega el código siguiente:

    ```python
    # Create a connected agent tool for the healer agent
    healer_agent_tool = ConnectedAgentTool(
        id=healer_agent.id, 
        name=healer_agent_name, 
        description="Responsible for healing party members and addressing injuries."
    )
    ```

    Ahora vamos a crear los otros agentes miembros del grupo.

1. En el comentario **Crear el agente rastreador y la herramienta conectada**, agrega el código siguiente:
    
    ```python
    # Create the scout agent and connected tool
    scout_agent = agents_client.create_agent(
        model=model_deployment,
        name=scout_agent_name,
        instructions=scout_instructions
    )
    scout_agent_tool = ConnectedAgentTool(
        id=scout_agent.id, 
        name=scout_agent_name, 
        description="Goes ahead of the main party to perform reconnaissance."
    )
    ```

1. En el comentario **Crear el agente guerrero y la herramienta conectada**, agrega el código siguiente:
    
    ```python
    # Create the warrior agent and connected tool
    warrior_agent = agents_client.create_agent(
        model=model_deployment,
        name=warrior_agent_name,
        instructions=warrior_instructions
    )
    warrior_agent_tool = ConnectedAgentTool(
        id=warrior_agent.id, 
        name=warrior_agent_name, 
        description="Responds to combat or physical challenges."
    )
    ```


1. En el comentario **Crear un agente principal con las herramientas del agente conectado**, agrega el código siguiente:
    
    ```python
    # Create a main agent with the Connected Agent tools
    agent = agents_client.create_agent(
        model=model_deployment,
        name="quest_master",
        instructions="""
            You are the Questmaster, the intelligent guide of a three-member adventuring party exploring a short dungeon. 
            Based on the scenario, delegate tasks to the appropriate party member. The current party members are: Warrior, Scout, Healer.
            Only include the party member's response, do not provide an analysis or summary.
        """,
        tools=[
            healer_agent_tool.definitions[0],
            scout_agent_tool.definitions[0],
            warrior_agent_tool.definitions[0]
        ]
    )
    ```

1. Busca el comentario **Crear subproceso para la sesión de chat** y agrega el código siguiente:
    
    ```python
    # Create thread for the chat session
    print("Creating agent thread.")
    thread = agents_client.threads.create()
    ```


1. En el comentario **Crear una solicitud de tarea**, agrega el código siguiente:
    
    ```python
    prompt = "We find a locked door with strange symbols, and the warrior is limping."
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
   python agent_quest.py
    ```

    Deberías ver un resultado parecido a los siguiente:

    ```output
    Creating agent thread.
    Processing agent thread. Please wait.

    MessageRole.USER:
    We find a locked door with strange symbols, and the warrior is limping.

    MessageRole.AGENT:
    - **Scout:** Decipher the celestial patterns of the strange symbols and determine the sequence to unlock the door. 
    - **Healer:** The warrior's injury has been addressed; moderate strain is relieved through healing magic and restorative salve.
    - **Warrior:** Recovered and ready to assist physically or guard the party as we proceed.

    Cleaning up agents:
    Deleted quest master agent.
    Deleted healer agent.
    Deleted scout agent.
    Deleted warrior agent.
    ```

    Puedes intentar modificar la solicitud mediante un escenario diferente para ver cómo colaboran los agentes.

## Limpieza

Si has terminado de explorar Agente de servicio de IA de Azure, debes eliminar los recursos que has creado en este ejercicio para evitar incurrir en costes innecesarios de Azure.

1. Vuelve a la pestaña del explorador que contiene Azure Portal (o vuelve a abrir [Azure Portal](https://portal.azure.com) en `https://portal.azure.com` en una nueva pestaña del explorador) y mira el contenido del grupo de recursos donde implementó los recursos usados en este ejercicio.

1. Selecciona **Eliminar grupo de recursos** en la barra de herramientas.

1. Escribe el nombre del grupo de recursos y confirma que deseas eliminarlo.
