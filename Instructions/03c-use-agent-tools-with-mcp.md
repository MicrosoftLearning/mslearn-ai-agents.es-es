---
lab:
  title: Conexión de agentes de IA a un servidor MCP remoto
  description: Obtenga información sobre cómo integrar herramientas del Protocolo de contexto de modelo (MCP) con agentes de IA
---

# Conexión de agentes de IA a herramientas mediante el Protocolo de contexto de modelo (MCP)

En este ejercicio, creará un agente que se conecta a un servidor MCP hospedado en la nube. El agente usará la búsqueda con tecnología de inteligencia artificial para ayudar a los desarrolladores a encontrar respuestas precisas y en tiempo real de la documentación oficial de Microsoft. Esto es útil para crear asistentes que aporten a los desarrolladores instrucciones actualizadas sobre herramientas como Azure, .NET y Microsoft 365. El agente usará las herramientas de MCP disponibles para consultar la documentación y devolver los resultados pertinentes.

> **Sugerencia**: El código usado en este ejercicio se basa en el repositorio de ejemplos de compatibilidad con MCP del servicio Agente de Azure AI. Consulte [Demostraciones de Azure OpenAI](https://github.com/retkowsky/Azure-OpenAI-demos/blob/main/Azure%20Agent%20Service/9%20Azure%20AI%20Agent%20service%20-%20MCP%20support.ipynb) o visite [Conexión a servidores de Protocolo de contexto de modelo](https://learn.microsoft.com/en-us/azure/ai-foundry/agents/how-to/tools/model-context-protocol) para más información.

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
    - **Región**: *Seleccione cualquiera de las siguientes ubicaciones admitidas:* \*
      * Oeste de EE. UU. 2
      * Oeste de EE. UU.
      * Este de Noruega
      * Norte de Suiza
      * Norte de Emiratos Árabes Unidos
      * Sur de la India

    > \* Algunos de los recursos de Azure AI están restringidos por cuotas de modelo regionales. En caso de que se alcance un límite de cuota más adelante en el ejercicio, es posible que tengas que crear otro recurso en otra región.

1. Selecciona **Crear** y espera a que tu proyecto se cree.
1. Si se te solicita, implementa un modelo **gpt-4o** mediante la opción de implementación *Estándar global* o *Estándar* (en función de la disponibilidad de la cuota).

    >**Nota**: Si la cuota está disponible, se puede implementar automáticamente un modelo base GPT-4o al crear el agente y el proyecto.

1. Cuando se crea el proyecto, se abre el sitio de prueba de agentes.

1. En el panel de navegación de la izquierda, selecciona **Información general** para ver la página principal del proyecto; que tiene este aspecto:

    ![Captura de pantalla de la página de información general de un proyecto de Fundición de IA de Azure.](./Media/ai-foundry-project.png)

1. Copie el valor del **punto de conexión del proyecto de Fundición de IA de Azure**. Lo usará para conectarse al proyecto en una aplicación cliente.

## Desarrollo de un agente que usa herramientas de funciones de MCP

Ahora que has creado el proyecto en Fundición de IA, vamos a desarrollar una aplicación que integre un agente de IA con un servidor MCP.

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
   cd ai-agents/Labfiles/03c-use-agent-tools-with-mcp/Python
   ls -a -l
    ```

### Configuración de la aplicación

1. En el panel de la línea de comandos de Cloud Shell, escribe el siguiente comando para instalar las bibliotecas que vas a usar:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt --pre azure-ai-projects azure-ai-agents mcp
    ```

    >**Nota:** puedes ignorar los mensajes de error o advertencia que se muestran durante la instalación de la biblioteca.

1. Escribe el siguiente comando para editar el archivo de configuración que se ha proporcionado:

    ```
   code .env
    ```

    El archivo se abre en un editor de código.

1. En el archivo de código, reemplaza el marcador de posición **your_project_endpoint** con el punto de conexión del proyecto (copiado de la página **Información general** del proyecto en el portal de la Función de IA de Azure) y asegúrate de que la variable MODEL_DEPLOYMENT_NAME está configurada con el nombre de la implementación de tu modelo (debe ser *gpt-4o*).

1. Después de reemplazar los marcadores de posición, usa el comando **CTRL+S** para guardar los cambios y después usa el comando **CTRL+Q** para cerrar el editor de código mientras mantienes abierta la línea de comandos de Cloud Shell.

### Conexión de un agente de Azure AI a un servidor MCP remoto

En esta tarea, se conectará a un servidor MCP remoto, preparará el agente de IA y ejecutará un mensaje de usuario.

1. Escribe el siguiente comando para editar el archivo de código que se ha proporcionado:

    ```
   code client.py
    ```

    El archivo se abre en el editor de código.

1. Busca el comentario **Agregar referencias** y agrega el código siguiente para importar las clases:

    ```python
   # Add references
   from azure.identity import DefaultAzureCredential
   from azure.ai.agents import AgentsClient
   from azure.ai.agents.models import McpTool, ToolSet, ListSortOrder
    ```

1. Busca el comentario **Conectar con el cliente de los agentes** y agrega el código siguiente para conectarte al proyecto de Azure AI mediante las credenciales actuales de Azure.

    ```python
   # Connect to the agents client
   agents_client = AgentsClient(
        endpoint=project_endpoint,
        credential=DefaultAzureCredential(
            exclude_environment_credential=True,
            exclude_managed_identity_credential=True
        )
   )
    ```

1. En el comentario **Inicializar la herramienta MCP del agente**, agregue el código siguiente:

    ```python
   # Initialize agent MCP tool
   mcp_tool = McpTool(
        server_label=mcp_server_label,
        server_url=mcp_server_url,
   )
    
   mcp_tool.set_approval_mode("never")
    
   toolset = ToolSet()
   toolset.add(mcp_tool)
    ```

    Este código se conectará al servidor MCP remoto de Microsft Learn Docs. Se trata de un servicio hospedado en la nube que permite a los clientes acceder a información de confianza y actualizada directamente desde la documentación oficial de Microsoft.

1. En el comentario **Crear un nuevo agente**, agregue el código siguiente:

    ```python
   # Create a new agent
   agent = agents_client.create_agent(
        model=model_deployment,
        name="my-mcp-agent",
        instructions="""
        You have access to an MCP server called `microsoft.docs.mcp` - this tool allows you to 
        search through Microsoft's latest official documentation. Use the available MCP tools 
        to answer questions and perform tasks."""
   )
    ```

    En este código, debe proporcionar instrucciones para el agente y las definiciones de herramientas de MCP.

1. Busque el comentario **Crear conversación para la comunicación** y agregue el código siguiente:

    ```python
   # Create thread for communication
   thread = agents_client.threads.create()
   print(f"Created thread, ID: {thread.id}")
    ```

1. Busque el comentario **Crear un mensaje en la conversación** y agregue el código siguiente:

    ```python
   # Create a message on the thread
   prompt = input("\nHow can I help?: ")
   message = agents_client.messages.create(
        thread_id=thread.id,
        role="user",
        content=prompt,
   )
   print(f"Created message, ID: {message.id}")
    ```

1. Busque el comentario **Establecer modo de aprobación** y agregue el código siguiente:

    ```python
    # Set approval mode
    mcp_tool.set_approval_mode("never")
    ```

    Esto permite que el agente invoque automáticamente las herramientas de MCP sin necesidad de aprobación del usuario. Si desea requerir aprobación, debe proporcionar un valor de encabezado mediante `mcp_tool.update_headers`.

1. Busque el comentario **Crear y procesar la ejecución del agente en el subproceso con herramientas de MCP** y agregue el código siguiente:

    ```python
   # Create and process agent run in thread with MCP tools
   run = agents_client.runs.create_and_process(thread_id=thread.id, agent_id=agent.id, toolset=toolset)
   print(f"Created run, ID: {run.id}")
    ```
    
    El agente de IA invoca automáticamente las herramientas de MCP conectadas para procesar la solicitud de mensaje. Para ilustrar este proceso, el código proporcionado en el comentario **Mostrar pasos de ejecución y llamadas a herramientas** generará las herramientas invocadas desde el servidor MCP.

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
   python client.py
    ```

1. Cuando se le solicite, escriba una solicitud de información técnica como:

    ```
    Give me the Azure CLI commands to create an Azure Container App with a managed identity.
    ```

1. Espere a que el agente procese la indicación, usando el servidor MCP para encontrar una herramienta adecuada para recuperar la información solicitada. Deberías ver un resultado parecido a los siguiente:

    ```
    Created agent, ID: <<agent-id>>
    MCP Server: mslearn at https://learn.microsoft.com/api/mcp
    Created thread, ID: <<thread-id>>
    Created message, ID: <<message-id>>
    Created run, ID: <<run-id>>
    Run completed with status: RunStatus.COMPLETED
    Step <<step1-id>> status: completed

    Step <<step2-id>> status: completed
    MCP Tool calls:
        Tool Call ID: <<tool-call-id>>
        Type: mcp
        Type: microsoft_code_sample_search


    Conversation:
    --------------------------------------------------
    ASSISTANT: You can use Azure CLI to create an Azure Container App with a managed identity (either system-assigned or user-assigned). Below are the relevant commands and workflow:

    ---

    ### **1. Create a Resource Group**
    '''azurecli
    az group create --name myResourceGroup --location eastus
    '''
    

    {{continued...}}

    By following these steps, you can deploy an Azure Container App with either system-assigned or user-assigned managed identities to integrate seamlessly with other Azure services.
    --------------------------------------------------
    USER: Give me the Azure CLI commands to create an Azure Container App with a managed identity.
    --------------------------------------------------
    Deleted agent
    ```

    Observe que el agente ha podido invocar automáticamente la herramienta de MCP `microsoft_code_sample_search` para cumplir la solicitud.

1. Puede volver a ejecutar la aplicación (con el comando `python client.py`) para solicitar información diferente. En cada caso, el agente intentará encontrar documentación técnica mediante la herramienta de MCP.

## Limpieza

Ahora que has terminado el ejercicio, debes eliminar los recursos en la nube que has creado para evitar el uso innecesario de recursos.

1. Abre [Azure Portal](https://portal.azure.com) en `https://portal.azure.com` y visualiza el contenido del grupo de recursos donde implementaste los recursos del centro usados en este ejercicio.
1. Selecciona **Eliminar grupo de recursos** en la barra de herramientas.
1. Escribe el nombre del grupo de recursos y confirma que deseas eliminarlo.
