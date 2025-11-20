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
   cd ai-agents/Labfiles/03b-build-multi-agent-solution/Python
   ls -a -l
    ```

    Los archivos proporcionados incluyen código de aplicación y un archivo para las opciones de configuración.

### Configuración de la aplicación

1. En el panel de la línea de comandos de Cloud Shell, escribe el siguiente comando para instalar las bibliotecas que vas a usar:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-projects azure-ai-agents
    ```

1. Escribe el siguiente comando para editar el archivo de configuración que se ha proporcionado:

    ```
   code .env
    ```

    El archivo se abre en un editor de código.

1. En el archivo de código, reemplace el marcador de posición **your_project_endpoint** por el punto de conexión de su proyecto (copiado de la página **Información general** del proyecto en el Portal de la Fundición de IA de Azure) y el marcador de posición **your_model_deployment** por el nombre que asignó a la implementación del modelo gpt-4o (que es `gpt-4o` de manera predeterminada).

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

1. Tenga en cuenta que se ha proporcionado código para cargar el punto de conexión del proyecto y el nombre del modelo de las variables de entorno.

1. Busque el comentario **Conectarse al cliente de agentes** y agregue el código siguiente para crear un elemento AgentsClient conectado al proyecto:

    ```python
   # Connect to the agents client
   agents_client = AgentsClient(
        endpoint=project_endpoint,
        credential=DefaultAzureCredential(
            exclude_environment_credential=True, 
            exclude_managed_identity_credential=True
        ),
   )
    ```

    Ahora agregará código que usa AgentsClient para crear varios agentes, cada uno con un rol específico para procesar una incidencia de soporte técnico.

    > **Sugerencia**: Al agregar código posteriormente, asegúrese de mantener el nivel correcto de sangría en la instrucción `using agents_client:`.

1. Busque el comentario **Crear un agente para priorizar las incidencias de soporte técnico** y escriba el código siguiente (teniendo cuidado de conservar el nivel correcto de sangría):

    ```python
   # Create an agent to prioritize support tickets
   priority_agent_name = "priority_agent"
   priority_agent_instructions = """
   Assess how urgent a ticket is based on its description.

   Respond with one of the following levels:
   - High: User-facing or blocking issues
   - Medium: Time-sensitive but not breaking anything
   - Low: Cosmetic or non-urgent tasks

   Only output the urgency level and a very brief explanation.
   """

   priority_agent = agents_client.create_agent(
        model=model_deployment,
        name=priority_agent_name,
        instructions=priority_agent_instructions
   )
    ```

1. Busque el comentario **Crear un agente para asignar vales al equipo adecuado** y escriba el código siguiente:

    ```python
   # Create an agent to assign tickets to the appropriate team
   team_agent_name = "team_agent"
   team_agent_instructions = """
   Decide which team should own each ticket.

   Choose from the following teams:
   - Frontend
   - Backend
   - Infrastructure
   - Marketing

   Base your answer on the content of the ticket. Respond with the team name and a very brief explanation.
   """

   team_agent = agents_client.create_agent(
        model=model_deployment,
        name=team_agent_name,
        instructions=team_agent_instructions
   )
    ```

1. Busque el comentario **Crear un agente para calcular el trabajo de una incidencia de soporte técnico** y escriba el código siguiente:

    ```python
   # Create an agent to estimate effort for a support ticket
   effort_agent_name = "effort_agent"
   effort_agent_instructions = """
   Estimate how much work each ticket will require.

   Use the following scale:
   - Small: Can be completed in a day
   - Medium: 2-3 days of work
   - Large: Multi-day or cross-team effort

   Base your estimate on the complexity implied by the ticket. Respond with the effort level and a brief justification.
   """

   effort_agent = agents_client.create_agent(
        model=model_deployment,
        name=effort_agent_name,
        instructions=effort_agent_instructions
   )
    ```

    Hasta ahora, ha creado tres agentes; cada uno de los cuales tiene un rol específico en la evaluación de prioridades de una incidencia de soporte técnico. Ahora vamos a crear objetos ConnectedAgentTool para cada uno de estos agentes para que otros agentes puedan usarlos.

1. Busque el comentario **Crear herramientas de agente conectado para los agentes de soporte técnico** y escriba el código siguiente:

    ```python
   # Create connected agent tools for the support agents
   priority_agent_tool = ConnectedAgentTool(
        id=priority_agent.id, 
        name=priority_agent_name, 
        description="Assess the priority of a ticket"
   )
    
   team_agent_tool = ConnectedAgentTool(
        id=team_agent.id, 
        name=team_agent_name, 
        description="Determines which team should take the ticket"
   )
    
   effort_agent_tool = ConnectedAgentTool(
        id=effort_agent.id, 
        name=effort_agent_name, 
        description="Determines the effort required to complete the ticket"
   )
    ```

    Ahora está listo para crear un agente principal que coordinará el proceso de evaluación de prioridades de vales mediante los agentes conectados según sea necesario.

1. Busque el comentario **Crear un agente para evaluar prioridades en el procesamiento de incidencias de soporte técnico mediante agentes conectados**y escriba el código siguiente:

    ```python
   # Create an agent to triage support ticket processing by using connected agents
   triage_agent_name = "triage-agent"
   triage_agent_instructions = """
   Triage the given ticket. Use the connected tools to determine the ticket's priority, 
   which team it should be assigned to, and how much effort it may take.
   """

   triage_agent = agents_client.create_agent(
        model=model_deployment,
        name=triage_agent_name,
        instructions=triage_agent_instructions,
        tools=[
            priority_agent_tool.definitions[0],
            team_agent_tool.definitions[0],
            effort_agent_tool.definitions[0]
        ]
   )
    ```

    Ahora que ha definido un agente principal, puede enviarle una indicación y hacer que use los otros agentes para evaluar la prioridad de un problema de soporte técnico.

1. Busque el comentario **Usar los agentes para evaluar la prioridad de un problema de soporte técnico** y escriba el código siguiente:

    ```python
   # Use the agents to triage a support issue
   print("Creating agent thread.")
   thread = agents_client.threads.create()  

   # Create the ticket prompt
   prompt = input("\nWhat's the support problem you need to resolve?: ")
    
   # Send a prompt to the agent
   message = agents_client.messages.create(
        thread_id=thread.id,
        role=MessageRole.USER,
        content=prompt,
   )   
    
   # Run the thread usng the primary agent
   print("\nProcessing agent thread. Please wait.")
   run = agents_client.runs.create_and_process(thread_id=thread.id, agent_id=triage_agent.id)
        
   if run.status == "failed":
        print(f"Run failed: {run.last_error}")

   # Fetch and display messages
   messages = agents_client.messages.list(thread_id=thread.id, order=ListSortOrder.ASCENDING)
   for message in messages:
        if message.text_messages:
            last_msg = message.text_messages[-1]
            print(f"{message.role}:\n{last_msg.text.value}\n")
   
    ```

1. Busque el comentario **Limpiar**y escriba el código siguiente para eliminar los agentes cuando ya no sean necesarios:

    ```python
   # Clean up
   print("Cleaning up agents:")
   agents_client.delete_agent(triage_agent.id)
   print("Deleted triage agent.")
   agents_client.delete_agent(priority_agent.id)
   print("Deleted priority agent.")
   agents_client.delete_agent(team_agent.id)
   print("Deleted team agent.")
   agents_client.delete_agent(effort_agent.id)
   print("Deleted effort agent.")
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

1. Escriba un mensaje, como `Users can't reset their password from the mobile app.`.

    Después de que los agentes procesen la indicación, debería ver una salida similar a la siguiente:

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
