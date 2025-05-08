---
lab:
  title: Uso de una función personalizada en un agente de IA
  description: Descubre cómo usar funciones para agregar funcionalidades personalizadas a los agentes.
---

# Uso de una función personalizada en un agente de IA

En este ejercicio, explorarás la creación de un agente que puede usar funciones personalizadas como herramienta para completar tareas.

Crearás un agente de soporte técnico sencillo que puede recopilar los detalles de un problema técnico y generar una incidencia de soporte técnico.

Este ejercicio debería tardar en completarse **30** minutos aproximadamente.

> **Nota**: algunas de las tecnologías que se usan en este ejercicio se encuentran en versión preliminar o en desarrollo activo. Puede que se produzcan algunos comportamientos, advertencias o errores inesperados.

## Creación de un proyecto de Fundición de IA de Azure

Comencemos creando un proyecto de Fundición de IA de Azure.

1. En un explorador web, abre el [Portal de la Fundición de IA de Azure](https://ai.azure.com) en `https://ai.azure.com` e inicia sesión con tus credenciales de Azure. Cierra las sugerencias o paneles de inicio rápido que se abran la primera vez que inicias sesión y, si es necesario, usa el logotipo de **Fundición de IA de Azure** en la parte superior izquierda para navegar a la página principal, que es similar a la siguiente imagen (cierra el panel **Ayuda** si está abierto):

    ![Captura de pantalla del Portal de la Fundición de IA de Azure.](./Media/ai-foundry-home.png)

1. En la página principal, selecciona **+Crear proyecto**.
1. En el asistente para **crear un proyecto**, escribe un nombre válido y si se te sugiere un centro existente, elige la opción para crear uno nuevo. A continuación, revisa los recursos de Azure que se crearán automáticamente para admitir el centro y el proyecto.
1. Selecciona **Personalizar** y especifica la siguiente configuración para el centro:
    - **Nombre del centro**: *un nombre válido para el centro*
    - **Suscripción**: *suscripción a Azure*
    - **Grupo de recursos**: *crea o selecciona un grupo de recursos*
    - **Ubicación**: selecciona una región de entre las siguientes:\*
        - estado
        - eastus2
        - swedencentral
        - westus
        - westus3
    - **Conectar Servicios de Azure AI o Azure OpenAI**: *crea un nuevo recurso de servicios de IA*
    - **Conectar Búsqueda de Azure AI**: omite la conexión

    > \* En el momento de escribir, estas regiones admiten el modelo gpt-4o para su uso en agentes. La disponibilidad del modelo está restringida por cuotas regionales. En caso de que se alcance un límite de cuota más adelante en el ejercicio, es posible que tengas que crear otro proyecto en otra región.

1. Selecciona **Siguiente** y revisa tu configuración. Luego, selecciona **Crear** y espera a que se complete el proceso.
1. Cuando se cree el proyecto, cierra las sugerencias que se muestran y revisa la página del proyecto en el Portal de la Fundición de IA de Azure, que debe tener un aspecto similar a la siguiente imagen:

    ![Captura de pantalla de los detalles de un proyecto de Azure AI en el Portal de la Fundición de IA de Azure.](./Media/ai-foundry-project.png)

## Implementación de un modelo de IA generativa

Ahora ya estás listo para implementar un modelo de lenguaje de IA generativa compatible con el agente.

1. En el panel de la izquierda de tu proyecto, en la sección **Mis recursos**, selecciona la página **Modelos y puntos de conexión**.
1. En la página **Modelos y puntos de conexión**, en la pestaña **Implementaciones de modelos**, en el menú **+ Implementar modelo**, selecciona **Implementar modelo base**.
1. Busca el modelo **gpt-4o** en la lista, selecciona y confirma.
1. Implementa el modelo con la siguiente configuración mediante la selección de **Personalizar** en los detalles de implementación:
    - **Nombre de implementación**: *nombre válido para la implementación de modelo*
    - **Tipo de implementación**: estándar global
    - **Actualización automática de la versión**: habilitado
    - **** Versión del modelo: *selecciona la versión disponible más reciente*
    - **Recurso de IA conectado**: *selecciona tu conexión de recursos de Azure OpenAI*
    - **Límite de velocidad de tokens por minuto (miles):** 50 000 *(o el máximo disponible en la suscripción si es inferior a 50 000)*
    - **Filtro de contenido**: DefaultV2

    > **Nota**: reducir el TPM ayuda a evitar el uso excesivo de la cuota disponible en la suscripción que está usando. 50 000 TPM deben ser suficientes para los datos que se usan en este ejercicio. Si la cuota disponible es inferior a esta, podrás completar el ejercicio, pero es posible que tengas que esperar y volver a enviar indicaciones si se supera el límite de velocidad.

1. Espera a que la implementación se complete.

## Desarrollo de un agente que usa herramientas de funciones

Ahora que has creado el proyecto en Fundición de IA, vamos a desarrollar una aplicación que implemente un agente mediante herramientas de funciones personalizadas.

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

    > **Sugerencia**: al escribir comandos en CloudShell, la salida puede ocupar una gran cantidad del búfer de pantalla y el cursor de la línea actual puede quedar oculto. Puedes despejar la pantalla al escribir el comando `cls` para que te resulte más fácil centrarte en cada tarea.

1. Escribe el siguiente comando para cambiar el directorio de trabajo a la carpeta que contiene los archivos de código y enumerarlos todos.

    ```
   cd ai-agents/Labfiles/03-ai-agent-functions/Python
   ls -a -l
    ```

    Los archivos proporcionados incluyen código de aplicación y un archivo para las opciones de configuración.

### Configuración de la aplicación

1. En el panel de la línea de comandos de Cloud Shell, escribe el siguiente comando para instalar las bibliotecas que vas a usar:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity azure-ai-projects
    ```

    >**Nota:** puedes ignorar los mensajes de error o advertencia que se muestran durante la instalación de la biblioteca.

1. Escribe el siguiente comando para editar el archivo de configuración que se ha proporcionado:

    ```
   code .env
    ```

    El archivo se abre en un editor de código.

1. En el archivo de código, reemplaza el marcador de posición **your_project_connection_string** por la cadena de conexión del proyecto (copiado de la página **Información general** del proyecto en el Portal de la Fundición de IA de Azure) y el marcador de posición **your_model_deployment** por el nombre que asignaste a tu implementación del modelo gpt-4o.
1. Después de reemplazar los marcadores de posición, usa el comando **CTRL+S** para guardar los cambios y, a continuación, usa el comando **CTRL+Q** para cerrar el editor de código mientras mantienes abierta la línea de comandos de Cloud Shell.

### Definición de una función personalizada

1. Escribe el siguiente comando para editar el archivo de código que se ha proporcionado para el código de función:

    ```
   code user_functions.py
    ```

1. Busca el comentario **Crear una función para enviar una incidencia de soporte técnico** y agrega el código siguiente, que generará un número de vale y guarda la incidencia de soporte técnico como un archivo de texto.

    ```python
   # Create a function to submit a support ticket
   def submit_support_ticket(email_address: str, description: str) -> str:
        script_dir = Path(__file__).parent  # Get the directory of the script
        ticket_number = str(uuid.uuid4()).replace('-', '')[:6]
        file_name = f"ticket-{ticket_number}.txt"
        file_path = script_dir / file_name
        text = f"Support ticket: {ticket_number}\nSubmitted by: {email_address}\nDescription:\n{description}"
        file_path.write_text(text)
    
        message_json = json.dumps({"message": f"Support ticket {ticket_number} submitted. The ticket file is saved as {file_name}"})
        return message_json
    ```

1. Busca el comentario **Definir un conjunto de funciones invocables** y agrega el código siguiente, que define estáticamente un conjunto de funciones invocables en este archivo de código (en este caso, solo hay una, pero en una solución real puedes tener varias funciones a las que puede llamar el agente):

    ```python
   # Define a set of callable functions
   user_functions: Set[Callable[..., Any]] = {
        submit_support_ticket
    }
    ```
1. Guarda el archivo (*CTRL + S*).

### Escritura de código para implementar un agente que pueda usar la función

1. Escribe el siguiente comando para empezar a editar el código del agente.

    ```
    code agent.py
    ```

    > **Sugerencia**: al agregar código al archivo de código, asegúrate de mantener la sangría correcta.

1. Revisa el código existente, que recupera los valores de configuración de la aplicación y configura un bucle en el que el usuario puede escribir indicaciones para el agente. El resto del archivo incluye comentarios en los que agregarás el código necesario para implementar el agente de soporte técnico.
1. Busca el comentario **Agregar referencias** y agrega el código siguiente para importar las clases que necesitarás para crear un agente de Azure AI que use el código de función como herramienta:

    ```python
   # Add references
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
   from azure.ai.projects.models import FunctionTool, ToolSet
   from user_functions import user_functions
    ```

1. Busca el comentario **Conectar el proyecto a Fundición de IA de Azure** y agrega el siguiente código para conectarte al proyecto de Azure AI mediante las credenciales de Azure actuales.

    > **Sugerencia**: ten cuidado de mantener el nivel de sangría correcto.

    ```python
   # Connect to the Azure AI Foundry project
   project_client = AIProjectClient.from_connection_string(
        credential=DefaultAzureCredential
            (exclude_environment_credential=True,
             exclude_managed_identity_credential=True),
        conn_str=PROJECT_CONNECTION_STRING
   )
    ```
    
1. Busca la sección del comentario **Definir un agente que pueda usar las funciones personalizadas** y agrega el código siguiente para agregar el código de función como un conjunto de herramientas. A continuación, crea un agente que pueda usar el conjunto de herramientas y un hilo en el que ejecutar la sesión de chat.

    ```python
   # Define an agent that can use the custom functions
   with project_client:

        functions = FunctionTool(user_functions)
        toolset = ToolSet()
        toolset.add(functions)
        project_client.agents.enable_auto_function_calls(toolset=toolset)
            
        agent = project_client.agents.create_agent(
            model=MODEL_DEPLOYMENT,
            name="support-agent",
            instructions="""You are a technical support agent.
                            When a user has a technical issue, you get their email address and a description of the issue.
                            Then you use those values to submit a support ticket using the function available to you.
                            If a file is saved, tell the user the file name.
                         """,
            toolset=toolset
        )

        thread = project_client.agents.create_thread()
        print(f"You're chatting with: {agent.name} ({agent.id})")

    ```

1. Busca el comentario **Enviar una indicación al agente** y agrega el código siguiente para agregar la indicación del usuario como mensaje y ejecutar la conversación.

    ```python
   # Send a prompt to the agent
   message = project_client.agents.create_message(
        thread_id=thread.id,
        role="user",
        content=user_prompt
   )
   run = project_client.agents.create_and_process_run(thread_id=thread.id, agent_id=agent.id)
    ```

    > **Nota**: el uso del método **create_and_process_run** para ejecutar el hilo permite al agente buscar automáticamente las funciones y elegir usarlas en función de sus nombres y parámetros. Como alternativa, puedes usar el método **create_run**, en cuyo caso serías responsable de escribir código para sondear el estado de ejecución con el fin de determinar cuándo se requiere una llamada a la función para llamarla y devolver los resultados al agente.

1. Busca el comentario **Comprobar el estado de ejecución de los errores** y agrega el código siguiente para mostrar los errores que se producen.

    ```python
   # Check the run status for failures
   if run.status == "failed":
        print(f"Run failed: {run.last_error}")
    ```

1. Busca el comentario **Mostrar la respuesta más reciente del agente** y agrega el código siguiente para recuperar los mensajes del hilo completado y mostrar el último mensaje enviado por el agente.

    ```python
   # Show the latest response from the agent
   messages = project_client.agents.list_messages(thread_id=thread.id)
   last_msg = messages.get_last_text_message_by_role("assistant")
   if last_msg:
        print(f"Last Message: {last_msg.text.value}")
    ```

1. Busca el comentario **Obtener el historial de conversaciones**, que se produce después de que finaliza el bucle, y agrega el código siguiente para imprimir los mensajes del hilo de conversación, revirtiendo el orden para mostrarlos en secuencia cronológica.

    ```python
   # Get the conversation history
   print("\nConversation Log:\n")
   messages = project_client.agents.list_messages(thread_id=thread.id)
   for message_data in reversed(messages.data):
        last_message_content = message_data.content[-1]
        print(f"{message_data.role}: {last_message_content.text.value}\n")
    ```

1. Busca el comentario **Limpiar** y agrega el código siguiente para eliminar el agente y el hilo cuando ya no sea necesario.

    ```python
   # Clean up
   project_client.agents.delete_agent(agent.id)
   project_client.agents.delete_thread(thread.id)
    ```

1. Revisa el código mediante los comentarios para comprender cómo:
    - Agrega el conjunto de funciones personalizadas a un conjunto de herramientas
    - Crea un agente que usa el conjunto de herramientas.
    - Ejecuta un hilo con un mensaje de indicación del usuario.
    - Comprueba el estado de la ejecución en caso de que se produzca un error.
    - Recupera los mensajes del hilo completado y muestra el último mensaje enviado por el agente.
    - Muestra el historial de conversaciones
    - Elimina el agente y el hilo cuando ya no son necesarios.

1. Guarda el archivo de código (*CTRL+S*) cuando hayas terminado. También puedes cerrar el editor de código (*CTRL+Q*); aunque es posible que desees mantenerlo abierto en caso de que tengas modificar el código que has agregado. En cualquier caso, no cierres el panel de línea de comandos de Cloud Shell.

### Inicia sesión en Azure y ejecuta la aplicación

1. En el panel de línea de comandos de Cloud Shell, escribe el siguiente comando para iniciar sesión en Azure.

    ```
    az login
    ```

    **<font color="red">Debes iniciar sesión en Azure, aunque la sesión de Cloud Shell ya esté autenticada.</font>**

    > **Nota**: En la mayoría de los escenarios, será suficiente con usar *az login*. Sin embargo, si tienes suscripciones en varios inquilinos, es posible que tengas que especificar el inquilino mediante el parámetro *--tenant*. Consulta [Iniciar sesión en Azure de forma interactiva mediante la CLI de Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively) para obtener más información.
    
1. Cuando se te solicite, siga las instrucciones para abrir la página de inicio de sesión en una nueva pestaña y escribe el código de autenticación proporcionado y las credenciales de Azure. Después, completa el proceso de inicio de sesión en la línea de comandos y selecciona la suscripción que contiene tu centro de Fundición de IA de Azure si se te solicita.
1. Después de iniciar sesión, escribe el siguiente comando para ejecutar la aplicación:

    ```
   python agent.py
    ```
    
    La aplicación se ejecuta con las credenciales de la sesión de Azure autenticada para conectarse al proyecto y crear y ejecutar el agente.

1. Cuando se te solicite, escribe una indicación como:

    ```
   I have a technical problem
    ```

    > **Sugerencia**: si se produce un error en la aplicación porque se supera el límite de velocidad. Espere unos segundos y vuelve a intentarlo. Si no hay cuota suficiente disponible en la suscripción, es posible que el modelo no pueda responder.

1. Visualiza la respuesta. El agente puede pedir tu dirección de correo electrónico y una descripción del problema. Puedes usar cualquier dirección de correo electrónico (por ejemplo, `alex@contoso.com`) y cualquier descripción del problema (como `my computer won't start`).

    Cuando tengas suficiente información, el agente debe elegir usar la función según sea necesario.

1. Puedes continuar la conversación si lo deseas. El hilo está *con estado*, por lo que conserva el historial de conversaciones, lo que significa que el agente tiene el contexto completo para cada respuesta. Cuando hayas terminado, escribe `quit`.
1. Revisa los mensajes de la conversación que se recuperaron del hilo y los vales que se generaron.
1. La herramienta debe haber guardado incidencias de soporte técnico en la carpeta de la aplicación. Puedes usar el comando `ls` para comprobar y usar el comando `cat` para ver el contenido del archivo, de la siguiente manera:

    ```
   cat ticket-<ticket_num>.txt
    ```

## Limpieza

Ahora que has terminado el ejercicio, debes eliminar los recursos en la nube que has creado para evitar el uso innecesario de recursos.

1. Abre [Azure Portal](https://portal.azure.com) en `https://portal.azure.com` y visualiza el contenido del grupo de recursos donde implementaste los recursos del centro usados en este ejercicio.
1. Selecciona **Eliminar grupo de recursos** en la barra de herramientas.
1. Escribe el nombre del grupo de recursos y confirma que deseas eliminarlo.