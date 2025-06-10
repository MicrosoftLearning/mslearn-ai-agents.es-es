---
lab:
  title: Desarrollo de una solución de varios agentes
  description: Aprende a configurar varios agentes para colaborar mediante el SDK de kernel semántico
---

# Desarrollo de una solución de varios agentes

En este ejercicio, crearás un proyecto que organiza dos agentes de IA mediante el SDK de kernel semántico. Un agente del *Administrador de incidentes* analizará los archivos de registro del servicio en busca de problemas. Si se encuentra un problema, el Administrador de incidentes recomendará una acción de resolución y un agente de *DevOps Assistant* recibirá la recomendación e invocará la función correctiva y realizará la resolución. Después, el agente del Administrador de incidentes revisará los registros actualizados para asegurarse de que la resolución se ha realizado correctamente.

En este ejercicio se proporcionan cuatro archivos de registro de ejemplo. El código del agente de DevOps Assistant solo actualiza los archivos de registro de ejemplo con algunos mensajes de registro de ejemplo.

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

1. En el panel **Configuración**, anota el nombre de la implementación del modelo; que debe ser **gpt-4o**. Para confirmarlo, mira la implementación en la página **Modelos y puntos de conexión** (simplemente abre esa página en el panel de navegación de la izquierda).
1. En el panel de navegación de la izquierda, selecciona **Información general** para ver la página principal del proyecto; que tiene este aspecto:

    > **Nota**: si se muestra un error de *permisos insuficientes**, usa el botón **Reparar** para resolverlo.

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
   pip install python-dotenv azure-identity semantic-kernel[azure] 
    ```

    > **Nota**: al instalar *semantic-kernel[azure]* se instala automáticamente una versión de *azure-ai-projects* compatible con kernel semántico.

1. Escribe el siguiente comando para editar el archivo de configuración que se ha proporcionado:

    ```
   code .env
    ```

    El archivo se abre en un editor de código.

1. En el archivo de código, reemplaza el marcador de posición **your_project_endpoint** con el punto de conexión de tu proyecto (copiado de la página **Información general** del proyecto en el portal de la Fundición de IA de Azure), y el marcador de posición **your_model_deployment** con el nombre que has asignado a la implementación de tu modelo gpt-4o.

1. Después de reemplazar los marcadores de posición, usa el comando **CTRL+S** para guardar los cambios y, a continuación, usa el comando **CTRL+Q** para cerrar el editor de código mientras mantienes abierta la línea de comandos de Cloud Shell.

### Crear agentes de IA

Ahora ya estás listo para crear los agentes para la solución de varios agentes. Comencemos.

1. Escribe el siguiente comando para editar el archivo **agent_chat.py**:

    ```
   code agent_chat.py
    ```

1. Revisa el código del archivo; ten en cuenta que contiene:
    - Constantes que definen los nombres e instrucciones de los dos agentes.
    - Una función **main** en la que se agregará la mayor parte del código para implementar la solución de varios agentes.
    - Una clase **SelectionStrategy**, que usarás para implementar la lógica necesaria para determinar qué agente se debe seleccionar para cada turno de la conversación.
    - Una clase **ApprovalTerminationStrategy**, que usarás para implementar la lógica necesaria para determinar cuándo finaliza la conversación.
    - Una clase **DevopsPlugin** que contiene funciones para realizar operaciones de DevOps.
    - Una clase **LogFilePlugin** que contiene funciones para leer y escribir archivos de registro.

    En primer lugar, crearás el agente del *Administrador de incidentes*, que analizará los archivos de registro del servicio, identificará posibles pro
blemas y recomendará acciones de resolución o escalará los problemas cuando sea necesario.

1. Anota la cadena **INCIDENT_MANAGER_INSTRUCTIONS**. Estas son las instrucciones para tu agente.

1. En la función **main**, busca el comentario **Crear el agente del administrador de incidentes en el Agente de servicio de IA de Azure** y agrega el código siguiente para crear un agente de Azure AI.

    ```python
   # Create the incident manager agent on the Azure AI agent service
   incident_agent_definition = await client.agents.create_agent(
        model=ai_agent_settings.model_deployment_name,
        name=INCIDENT_MANAGER,
        instructions=INCIDENT_MANAGER_INSTRUCTIONS
   )
    ```

    Este código crea la definición del agente en el cliente del proyecto de Azure AI.

1. Busca el comentario **Crear un agente de kernel semántico para el agente del administrador de incidentes de Azure AI** y agrega el código siguiente para crear un agente de kernel semántico basado en la definición del agente de Azure AI.

    ```python
   # Create a Semantic Kernel agent for the Azure AI incident manager agent
   agent_incident = AzureAIAgent(
        client=client,
        definition=incident_agent_definition,
        plugins=[LogFilePlugin()]
   )
    ```

    Este código crea el agente de kernel semántico con acceso a **LogFilePlugin**. Este complemento permite al agente leer el contenido del archivo de registro.

    Ahora vamos a crear el segundo agente, que responderá a los problemas y realizará operaciones de DevOps para resolverlos.

1. En la parte superior del archivo de código, dedica un momento a observar la cadena **DEVOPS_ASSISTANT_INSTRUCTIONS**. Estas son las instrucciones que proporcionarás al nuevo agente del asistente de DevOps.

1. Busca el comentario **Create the devops agent on the Azure AI agent service** y añade el código siguiente para crear una definición del agente de Azure AI:
    
    ```python
   # Create the devops agent on the Azure AI agent service
   devops_agent_definition = await client.agents.create_agent(
        model=ai_agent_settings.model_deployment_name,
        name=DEVOPS_ASSISTANT,
        instructions=DEVOPS_ASSISTANT_INSTRUCTIONS,
   )
    ```

1. Busca el comentario **Create a Semantic Kernel agent for the devops Azure AI agent** y añade el código siguiente para crear un agente de kernel semántico basado en la definición del agente de Azure AI.
    
    ```python
   # Create a Semantic Kernel agent for the devops Azure AI agent
   agent_devops = AzureAIAgent(
        client=client,
        definition=devops_agent_definition,
        plugins=[DevopsPlugin()]
   )
    ```

    El complemento **DevopsPlugin** permite al agente simular tareas de DevOps, como reiniciar el servicio o revertir una transacción.

### Definición de estrategias de chat grupal

Ahora debes proporcionar la lógica utilizada para determinar qué agente se debe seleccionar para tomar el siguiente turno en una conversación y cuándo se debe finalizar la conversación.

Comencemos con **SelectionStrategy**, que identifica qué agente debe tomar el siguiente turno.

1. En la clase **SelectionStrategy** (debajo de la función **main**), busca el comentario **Select the next agent that should take the next turn in the chat** y añade el código siguiente para definir una función de selección:

    ```python
   # Select the next agent that should take the next turn in the chat
   async def select_agent(self, agents, history):
        """"Check which agent should take the next turn in the chat."""

        # The Incident Manager should go after the User or the Devops Assistant
        if (history[-1].name == DEVOPS_ASSISTANT or history[-1].role == AuthorRole.USER):
            agent_name = INCIDENT_MANAGER
            return next((agent for agent in agents if agent.name == agent_name), None)
        
        # Otherwise it is the Devops Assistant's turn
        return next((agent for agent in agents if agent.name == DEVOPS_ASSISTANT), None)
    ```

    Este código se ejecuta en cada turno para determinar qué agente debe responder, comprobando el historial de chats para ver quién respondió por última vez.

    Ahora vamos a implementar la clase **ApprovalTerminationStrategy** para ayudar a indicar cuándo se completa el objetivo y se puede finalizar la conversación.

1. En la clase **ApprovalTerminationStrategy**, busca el comentario **End the chat if the agent has indicated there is no action needed** y añade el código siguiente para definir la función de terminación:

    ```python
   # End the chat if the agent has indicated there is no action needed
   async def should_agent_terminate(self, agent, history):
        """Check if the agent should terminate."""
        return "no action needed" in history[-1].content.lower()
    ```

    El kernel invoca esta función después de la respuesta del agente para determinar si se cumplen los criterios de finalización. En este caso, el objetivo se cumple cuando el administrador de incidentes responde con "No action needed". Esta frase está definida en las instrucciones del agente del administrador de incidentes.

### Implementación del chat grupal

Ahora que tienes dos agentes y estrategias para ayudarles a tomar turnos y finalizar un chat, puedes implementar el chat grupal.

1. Haz una copia de seguridad en la función principal, busca el comentario **Add the agents to a group chat with a custom termination and selection strategy** y añade el código siguiente para crear el chat grupal:

    ```python
   # Add the agents to a group chat with a custom termination and selection strategy
   chat = AgentGroupChat(
        agents=[agent_incident, agent_devops],
        termination_strategy=ApprovalTerminationStrategy(
            agents=[agent_incident], 
            maximum_iterations=10, 
            automatic_reset=True
        ),
        selection_strategy=SelectionStrategy(agents=[agent_incident,agent_devops]),      
   )
    ```

    En este código, crearás un objeto de chat de grupo de agentes con el administrador de incidentes y los agentes de DevOps. También defines las estrategias de terminación y selección para el chat. Ten en cuenta que **ApprovalTerminationStrategy** solo está vinculado al agente del administrador de incidentes y no al agente de DevOps. Esto hace que el agente del administrador de incidentes sea responsable de indicar el final del chat. **SelectionStrategy** incluye todos los agentes que deben tomar un turno en el chat.

    Ten en cuenta que la marca de restablecimiento automático borrará automáticamente el chat cuando finalice. De este modo, el agente puede seguir analizando los archivos sin el objeto de historial de chats con demasiados tokens innecesarios. 

1. Busca el comentario **Append the current log file to the chat** y añade el código siguiente para agregar el texto del archivo de registro de lectura más reciente al chat:

    ```python
   # Append the current log file to the chat
   await chat.add_chat_message(logfile_msg)
   print()
    ```

1. Busca el comentario **Invoke a response from the agents** y añade el código siguiente para invocar el chat grupal:

    ```python
   # Invoke a response from the agents
   async for response in chat.invoke():
        if response is None or not response.name:
            continue
        print(f"{response.content}")
    ```

    Este es el código que desencadena el chat. Dado que el texto del archivo de registro se ha agregado como un mensaje, la estrategia de selección determinará qué agente debe leer y responder y, a continuación, la conversación continuará entre los agentes hasta que se cumplan las condiciones de la estrategia de terminación o se alcance el número máximo de iteraciones.

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
   python agent_chat.py
    ```

    Deberías ver un resultado parecido a los siguiente:

    ```output
    
    INCIDENT_MANAGER > /home/.../logs/log1.log | Restart service ServiceX
    DEVOPS_ASSISTANT > Service ServiceX restarted successfully.
    INCIDENT_MANAGER > No action needed.

    INCIDENT_MANAGER > /home/.../logs/log2.log | Rollback transaction for transaction ID 987654.
    DEVOPS_ASSISTANT > Transaction rolled back successfully.
    INCIDENT_MANAGER > No action needed.

    INCIDENT_MANAGER > /home/.../logs/log3.log | Increase quota.
    DEVOPS_ASSISTANT > Successfully increased quota.
    (continued)
    ```

    > **Nota**: La aplicación incluye código para esperar tras el procesamiento de cada archivo de registro e intentar reducir el riesgo de que se supere el límite de velocidad del TPM y el control de excepciones en caso de que se produzca de todos modos. Si no hay cuota suficiente disponible en la suscripción, es posible que el modelo no pueda responder.

1. Comprueba que los archivos de registro de la carpeta **logs** se actualicen con mensajes de operación de resolución de DevopsAssistant.

    Por ejemplo, log1.log debe tener anexados los siguientes mensajes de registro:

    ```log
    [2025-02-27 12:43:38] ALERT  DevopsAssistant: Multiple failures detected in ServiceX. Restarting service.
    [2025-02-27 12:43:38] INFO  ServiceX: Restart initiated.
    [2025-02-27 12:43:38] INFO  ServiceX: Service restarted successfully.
    ```

## Resumen

En este ejercicio, has usado el Agente de servicio de IA de Azure y el SDK de kernel semántico para crear agentes de incidentes y DevOps de IA que pueden detectar automáticamente problemas y aplicar soluciones. ¡Excelente trabajo!

## Limpieza

Si has terminado de explorar Agente de servicio de IA de Azure, debes eliminar los recursos que has creado en este ejercicio para evitar incurrir en costes innecesarios de Azure.

1. Vuelve a la pestaña del explorador que contiene Azure Portal (o vuelve a abrir [Azure Portal](https://portal.azure.com) en `https://portal.azure.com` en una nueva pestaña del explorador) y mira el contenido del grupo de recursos donde implementó los recursos usados en este ejercicio.

1. Selecciona **Eliminar grupo de recursos** en la barra de herramientas.

1. Escribe el nombre del grupo de recursos y confirma que deseas eliminarlo.
