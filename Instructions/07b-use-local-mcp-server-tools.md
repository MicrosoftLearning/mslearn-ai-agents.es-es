# Conexión de agentes de IA a herramientas mediante el Protocolo de contexto de modelo (MCP)

En este ejercicio, creará un agente capaz de conectarse a un servidor MCP y de detectar automáticamente funciones invocables.

Creará un agente de evaluación de inventario sencillo para un minorista de cosméticos. Con el servidor MCP, el agente podrá recuperar información sobre el inventario y realizar sugerencias de reposición o liquidación.

> **Sugerencia**: El código usado en este ejercicio se basa en los SDK de Fundición de IA de Azure y MCP para Python. Puede desarrollar soluciones similares mediante los SDK de Microsoft .NET. Consulte las [bibliotecas cliente del SDK de Fundición de IA de Azure](https://learn.microsoft.com/azure/ai-foundry/how-to/develop/sdk-overview) y el [SDK de C# de MCP](https://modelcontextprotocol.github.io/csharp-sdk/api/ModelContextProtocol.html) para más información.

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
1. Si se le solicita, implemente un modelo **gpt-4o** mediante la opción de implementación *Estándar global* o *Estándar* (en función de la disponibilidad de la cuota).

    >**Nota**: Si hay cuota disponible, se puede implementar automáticamente un modelo base GPT-4o al crear el agente y el proyecto.

1. Cuando se crea el proyecto, se abre e área de juegos de los agentes.

1. En el panel de navegación de la izquierda, selecciona **Información general** para ver la página principal del proyecto; que tiene este aspecto:

    ![Captura de pantalla de la página de información general de un proyecto de Fundición de IA de Azure.](./Media/ai-foundry-project.png)

1. Copia el valor del **punto de conexión del proyecto de la Fundición de IA de Azure** en un Bloc de notas, ya que lo usarás para conectarte a tu proyecto en una aplicación cliente.

## Desarrollo de un agente que usa herramientas de funciones de MCP

Ahora que ha creado el proyecto en Fundición de IA, vamos a desarrollar una aplicación que integre un agente de IA con un servidor MCP.

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
   cd ai-agents/Labfiles/07-use-agent-tools-with-mcp/Python
   ls -a -l
    ```

    Los archivos proporcionados incluyen el código de la aplicación cliente y servidor. El Protocolo de contexto de modelo proporciona una manera estandarizada de conectar modelos de IA a diferentes orígenes de datos y herramientas. Separamos `client.py` y `server.py` para mantener la lógica del agente y las definiciones de herramientas modulares y simular la arquitectura del mundo real. 
    
    `server.py` define las herramientas que el agente puede usar, simulando servicios back-end o lógica de negocios. 
    `client.py` controla la configuración del agente de IA, las indicaciones del usuario y la llamada a las herramientas cuando es necesario.

### Configuración de la aplicación

1. En el panel de la línea de comandos de Cloud Shell, escribe el siguiente comando para instalar las bibliotecas que vas a usar:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-projects mcp
    ```

    >**Nota:** puedes ignorar los mensajes de error o advertencia que se muestran durante la instalación de la biblioteca.

1. Escribe el siguiente comando para editar el archivo de configuración que se ha proporcionado:

    ```
   code .env
    ```

    El archivo se abre en un editor de código.

1. En el archivo de código, reemplace el marcador de posición **your_project_endpoint** por el punto de conexión del proyecto (copiado de la página **Información general** del proyecto en el portal de Función de IA de Azure) y asegúrese de que la variable MODEL_DEPLOYMENT_NAME está configurada con el nombre de la implementación de su modelo (debe ser *gpt-4o*).

1. Después de reemplazar los marcadores de posición, usa el comando **CTRL+S** para guardar los cambios y después usa el comando **CTRL+Q** para cerrar el editor de código mientras mantienes abierta la línea de comandos de Cloud Shell.

### Implementación de un servidor MCP

Un servidor de Protocolo de contexto de modelo (MCP) es un componente que hospeda herramientas invocables. Estas herramientas son funciones de Python que se pueden exponer a agentes de inteligencia artificial. Cuando las herramientas se anotan con `@mcp.tool()`, se vuelven reconocibles para el cliente, lo que permite a un agente de IA llamarlas dinámicamente durante una conversación o tarea. En esta tarea, agregará algunas herramientas que permitirán al agente realizar comprobaciones de inventario.

1. Escribe el siguiente comando para editar el archivo de código que se ha proporcionado para el código de función:

    ```
   code server.py
    ```

    En este archivo de código, definirá las herramientas que el agente puede usar para simular un servicio back-end para la tienda minorista. Observe el código de configuración del servidor en la parte superior del archivo. Usa `FastMCP` para poner en marcha rápidamente una instancia de servidor MCP denominada "Inventory". Este servidor hospedará las herramientas que defina y las hará accesibles para el agente durante el laboratorio.

1. Busque el comentario **Agregar una herramienta de comprobación de inventario** y agregue el código siguiente:

    ```python
   # Add an inventory check tool
   @mcp.tool()
   def get_inventory_levels() -> dict:
        """Returns current inventory for all products."""
        return {
            "Moisturizer": 6,
            "Shampoo": 8,
            "Body Spray": 28,
            "Hair Gel": 5, 
            "Lip Balm": 12,
            "Skin Serum": 9,
            "Cleanser": 30,
            "Conditioner": 3,
            "Setting Powder": 17,
            "Dry Shampoo": 45
        }
    ```

    Este diccionario representa un inventario de muestra. La anotación `@mcp.tool()` permitirá que el LLM detecte la función. 

1. Busque el comentario **Agregar una herramienta de ventas semanales** y agregue el código siguiente:

    ```python
   # Add a weekly sales tool
   @mcp.tool()
   def get_weekly_sales() -> dict:
        """Returns number of units sold last week."""
        return {
            "Moisturizer": 22,
            "Shampoo": 18,
            "Body Spray": 3,
            "Hair Gel": 2,
            "Lip Balm": 14,
            "Skin Serum": 19,
            "Cleanser": 4,
            "Conditioner": 1,
            "Setting Powder": 13,
            "Dry Shampoo": 17
        }
    ```

1. Guarde el archivo (*CTRL + S*).

### Implementación de un cliente MCP

Un cliente MCP es el componente que se conecta al servidor MCP para detectar y llamar a las herramientas. Puede considerarlo como el puente entre el agente y las funciones hospedadas en el servidor, lo que permite el uso de herramientas dinámicas en respuesta a las indicaciones del usuario.

1. Escriba el siguiente comando para empezar a editar el código del cliente.

    ```
   code client.py
    ```

    > **Sugerencia**: al agregar código al archivo de código, asegúrate de mantener la sangría correcta.

1. Busque el comentario **Agregar referencias** y agregue el código siguiente para importar las clases:

    ```python
   # Add references
   from mcp import ClientSession, StdioServerParameters
   from mcp.client.stdio import stdio_client
   from azure.ai.agents import AgentsClient
   from azure.ai.agents.models import FunctionTool, MessageRole, ListSortOrder
   from azure.identity import DefaultAzureCredential
    ```

1. Busque el comentario **Iniciar el servidor MCP** y agregue el código siguiente:

    ```python
   # Start the MCP server
   stdio_transport = await exit_stack.enter_async_context(stdio_client(server_params))
   stdio, write = stdio_transport
    ```

    En una configuración de producción estándar, el servidor se ejecutaría por separado del cliente. Pero, en este laboratorio, el cliente es responsable de iniciar el servidor mediante el transporte de entrada/salida estándar. De esta forma, se crea un canal de comunicación ligero entre los dos componentes y se simplifica la configuración de desarrollo local.

1. Busque el comentario **Crear una sesión de cliente MCP** y agregue el código siguiente:

    ```python
   # Create an MCP client session
   session = await exit_stack.enter_async_context(ClientSession(stdio, write))
   await session.initialize()
    ```

    Se crea una nueva sesión de cliente mediante los flujos de entrada y salida del paso anterior. La llamada a `session.initialize` prepara la sesión para detectar y llamar a las herramientas registradas en el servidor MCP.

1. En el comentario **Enumerar las herramientas disponibles**, agregue el código siguiente para comprobar que el cliente se ha conectado al servidor:

    ```python
   # List available tools
   response = await session.list_tools()
   tools = response.tools
   print("\nConnected to server with tools:", [tool.name for tool in tools]) 
    ```

    Ahora la sesión de cliente está lista para su uso con el agente de Azure AI.

### Conexión de las herramientas de MCP al agente

En esta tarea, preparará el agente de IA, aceptará las indicaciones del usuario e invocará las herramientas de función.

1. Busque el comentario **Conectar con el cliente de los agentes** y agregue el código siguiente para conectarse al proyecto de Azure AI mediante las credenciales actuales de Azure.

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

1. En el comentario **Enumerar las herramientas disponibles en el servidor**, agregue el código siguiente:

    ```python
   # List tools available on the server
   response = await session.list_tools()
   tools = response.tools
    ```

1. En el comentario **Compilar una función para cada herramienta**, agregue el código siguiente:

    ```python
   # Build a function for each tool
   def make_tool_func(tool_name):
        async def tool_func(**kwargs):
            result = await session.call_tool(tool_name, kwargs)
            return result
        
        tool_func.__name__ = tool_name
        return tool_func

   functions_dict = {tool.name: make_tool_func(tool.name) for tool in tools}
   mcp_function_tool = FunctionTool(functions=list(functions_dict.values()))
    ```

    Este código encapsula dinámicamente las herramientas disponibles en el servidor MCP para que el agente de IA pueda invocarlas. Cada herramienta se convierte en una función asincrónica y después se agrupa en `FunctionTool` para que el agente la use.

1. Busque el comentario **Crear el agente** y agregue el código siguiente:

    ```python
   # Create the agent
   agent = agents_client.create_agent(
        model=model_deployment,
        name="inventory-agent",
        instructions="""
        You are an inventory assistant. Here are some general guidelines:
        - Recommend restock if item inventory < 10  and weekly sales > 15
        - Recommend clearance if item inventory > 20 and weekly sales < 5
        """,
        tools=mcp_function_tool.definitions
   )
    ```

1. Busque el comentario **Habilitar llamada de función automática** y agregue el código siguiente:

    ```python
   # Enable auto function calling
   agents_client.enable_auto_function_calls(tools=mcp_function_tool)
    ```

1. En el comentario **Crear un subproceso para la sesión de chat**, agregue el código siguiente:

    ```python
   # Create a thread for the chat session
   thread = agents_client.threads.create()
    ```

1. Busque el comentario **Invocar la indicación** y agregue el código siguiente:

    ```python
   # Invoke the prompt
   message = agents_client.messages.create(
        thread_id=thread.id,
        role=MessageRole.USER,
        content=user_input,
   )
   run = agents_client.runs.create(thread_id=thread.id, agent_id=agent.id)
    ```

1. Busque el comentario **Recuperar la herramienta de función coincidente** y agregue el código siguiente:

    ```python
   # Retrieve the matching function tool
   function_name = tool_call.function.name
   args_json = tool_call.function.arguments
   kwargs = json.loads(args_json)
   required_function = functions_dict.get(function_name)

   # Invoke the function
   output = await required_function(**kwargs)
    ```

    Este código usa la información de la llamada a la herramienta del subproceso del agente. El nombre y los argumentos de la función se recuperan y se usan para invocar la función coincidente.

1. En el comentario **Anexar el texto de salida**, agregue el código siguiente:

    ```python
   # Append the output text
   tool_outputs.append({
        "tool_call_id": tool_call.id,
        "output": output.content[0].text,
   })
    ```

1. En el comentario **Enviar la salida de la llamada a la herramienta**, agregue el código siguiente:

    ```python
   # Submit the tool call output
   agents_client.runs.submit_tool_outputs(thread_id=thread.id, run_id=run.id, tool_outputs=tool_outputs)
    ```

    Este código indicará al subproceso del agente que la acción necesaria se ha completado y actualizará las salidas de la llamada a la herramienta.

1. Busque el comentario **Mostrar la respuesta** y agregue el código siguiente:

    ```python
   # Display the response
   messages = agents_client.messages.list(thread_id=thread.id, order=ListSortOrder.ASCENDING)
   for message in messages:
        if message.text_messages:
            last_msg = message.text_messages[-1]
            print(f"{message.role}:\n{last_msg.text.value}\n")
    ```

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

1. Cuando se le solicite, escriba una consulta como esta:

    ```
   What are the current inventory levels?
    ```

    > **Sugerencia**: si se produce un error en la aplicación porque se supera el límite de velocidad. Espere unos segundos y vuelve a intentarlo. Si no hay cuota suficiente disponible en la suscripción, es posible que el modelo no pueda responder.

    Debería ver un resultado parecido al siguiente:

    ```
    MessageRole.AGENT:
    Here are the current inventory levels:

    - Moisturizer: 6
    - Shampoo: 8
    - Body Spray: 28
    - Hair Gel: 5
    - Lip Balm: 12
    - Skin Serum: 9
    - Cleanser: 30
    - Conditioner: 3
    - Setting Powder: 17
    - Dry Shampoo: 45
    ```

1. Puedes continuar la conversación si lo deseas. El hilo está *con estado*, por lo que conserva el historial de conversaciones, lo que significa que el agente tiene el contexto completo para cada respuesta. 

    Prueba a escribir indicaciones como estas:

    ```
   Are there any products that should be restocked?
    ```

    ```
   Which products would you recommend for clearance?
    ```

    ```
   What are the best sellers this week?
    ```

    Cuando hayas terminado, escribe `quit`.

## Limpieza

Ahora que has terminado el ejercicio, debes eliminar los recursos en la nube que has creado para evitar el uso innecesario de recursos.

1. Abre [Azure Portal](https://portal.azure.com) en `https://portal.azure.com` y visualiza el contenido del grupo de recursos donde implementaste los recursos del centro usados en este ejercicio.
1. Selecciona **Eliminar grupo de recursos** en la barra de herramientas.
1. Escribe el nombre del grupo de recursos y confirma que deseas eliminarlo.
