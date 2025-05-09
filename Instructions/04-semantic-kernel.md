---
lab:
  title: Desarrollo de un agente de Azure AI con el SDK de Kernel semántico
  description: Aprende a usar el SDK de kernel semántico para crear y usar un agente del Agente de servicio de IA de Azure.
---

# Desarrollo de un agente de Azure AI con el SDK de Kernel semántico

En este ejercicio, usarás Agente de servicio de IA de Azure y kernel semántico para crear un agente de IA que cree un correo electrónico de reclamación de gastos.

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
    - **Ubicación**: selecciona cualquiera de las siguientes regiones:\*
        - estado
        - eastus2
        - swedencentral
        - westus
        - westus3
    - **Conectar Servicios de Azure AI o Azure OpenAI**: *crea un nuevo recurso de servicios de IA*
    - **Conectar Búsqueda de Azure AI**: omite la conexión

    > \* En el momento de escribir este ejercicio, estas regiones admitían el modelo gpt-4o para usarlo en los agentes. La disponibilidad del modelo está restringida por cuotas regionales. En caso de que se alcance un límite de cuota más adelante en el ejercicio, es posible que tengas que crear otro proyecto en otra región.

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

## Creación de una aplicación cliente del agente

Ahora estás listo para crear una aplicación cliente que defina un agente y una función personalizada. Parte del código se ha proporcionado en un repositorio de GitHub.

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
   cd ai-agents/Labfiles/04-semantic-kernel/python
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

1. En el archivo de código, reemplaza el marcador de posición **your_project_connection_string** por la cadena de conexión del proyecto (copiado de la página **Información general** del proyecto en el Portal de la Fundición de IA de Azure) y el marcador de posición **your_model_deployment** por el nombre que asignaste a tu implementación de modelo gpt-4o.
1. Después de reemplazar los marcadores de posición, usa el comando **CTRL+S** para guardar los cambios y, a continuación, usa el comando **CTRL+Q** para cerrar el editor de código mientras mantienes abierta la línea de comandos de Cloud Shell.

### Escritura de código para una aplicación de agente

> **Sugerencia**: al agregar código, asegúrate de mantener la sangría correcta. Usa los comentarios existentes como guía al escribir el nuevo código con el mismo nivel de sangría.

1. Escribe el siguiente comando para editar el archivo de código de agente que se ha proporcionado:

    ```
   code semantic-kernel.py
    ```

1. Revisa el código de este archivo. Contiene:
    - Algunas instrucciones de **importación** para agregar referencias a espacios de nombres usados habitualmente
    - Una función *principal* que define los datos de una notificación de gastos (en una aplicación real, esto probablemente se enviaría como un archivo) y, a continuación, llama a...
    - Una función **create_expense_claim** en la que se debe agregar el código para crear y usar tu agente
    - Una clase **EmailPlugin** que incluye una función kernel denominada **send_email**; que usará tu agente para simular la funcionalidad usada para enviar un correo electrónico.

1. En la parte superior del archivo, después de la instrucción **import** existente, busca el comentario **Agregar referencias**, y agrega el siguiente código para hacer referencia a los espacios de nombres en las bibliotecas que necesitarás para implementar tu agente:

    ```python
   # Add references
   from dotenv import load_dotenv
   from azure.identity.aio import DefaultAzureCredential
   from semantic_kernel.agents import AzureAIAgent, AzureAIAgentSettings, AzureAIAgentThread
   from semantic_kernel.functions import kernel_function
   from typing import Annotated
    ```

1. Cerca de la parte inferior del archivo, busca el comentario **Crear un complemento para la funcionalidad de correo electrónico**, y agrega el siguiente código para definir una clase para un complemento que contenga una función que tu agente usará para enviar correo electrónico (los complementos son una forma de agregar funcionalidad personalizada a los agentes de kernel semántico)

    ```python
   # Create a Plugin for the email functionality
   class EmailPlugin:
       """A Plugin to simulate email functionality."""
    
       @kernel_function(description="Sends an email.")
       def send_email(self,
                      to: Annotated[str, "Who to send the email to"],
                      subject: Annotated[str, "The subject of the email."],
                      body: Annotated[str, "The text body of the email."]):
           print("\nTo:", to)
           print("Subject:", subject)
           print(body, "\n")
    ```

    > **Nota**: la función *simula* el envío de un correo electrónico mediante la impresión en la consola. En una aplicación real, usarías un servicio SMTP o similar para enviar realmente el correo electrónico.

1. Realiza una copia de seguridad sobre el código de la nueva clase **EmailPlugin**, en la función **create_expense_claim**, busca el comentario **Obtener ajustes de configuración**, y agrega el siguiente código para cargar el archivo de configuración y crear un objeto **AzureAIAgentSettings** (que incluirá automáticamente los ajustes del agente de Azure AI de la configuración).

    (Asegúrate de mantener el nivel de sangría)

    ```python
   # Get configuration settings
   load_dotenv()
   ai_agent_settings = AzureAIAgentSettings()
    ```

1. Busca el comentario **Conectar con el proyecto Fundición de IA de Azure**, y agrega el siguiente código para conectar con tu proyecto Fundición de IA de Azure mediante las credenciales de Azure con las que has iniciado sesión.

    (Asegúrate de mantener el nivel de sangría)

    ```python
   # Connect to the Azure AI Foundry project
   async with (
        DefaultAzureCredential(
            exclude_environment_credential=True,
            exclude_managed_identity_credential=True) as creds,
        AzureAIAgent.create_client(
            credential=creds
        ) as project_client,
   ):
    ```

1. Busca el comentario **Definir un agente de Azure AI que envíe un correo electrónico de reclamación de gastos**, y agrega el siguiente código para crear una definición de agente de Azure AI para tu agente.

    (Asegúrate de mantener el nivel de sangría)

    ```python
   # Define an Azure AI agent that sends an expense claim email
   expenses_agent_def = await project_client.agents.create_agent(
        model= ai_agent_settings.model_deployment_name,
        name="expenses_agent",
        instructions="""You are an AI assistant for expense claim submission.
                        When a user submits expenses data and requests an expense claim, use the plug-in function to send an email to expenses@contoso.com with the subject 'Expense Claim`and a body that contains itemized expenses with a total.
                        Then confirm to the user that you've done so."""
   )
    ```

1. Busca el comentario **Crear un agente de kernel semántico**, y agrega el siguiente código para crear un objeto de agente de kernel semántico para tu agente de Azure AI, e incluye una referencia al complemento **EmailPlugin**.

    (Asegúrate de mantener el nivel de sangría)

    ```python
   # Create a semantic kernel agent
   expenses_agent = AzureAIAgent(
        client=project_client,
        definition=expenses_agent_def,
        plugins=[EmailPlugin()]
   )
    ```

1. Busca el comentario **Usar el agente para generar un correo electrónico de reclamación de gastos**, y agrega el siguiente código para crear un subproceso en el que se ejecutará tu agente y después invocarlo con un mensaje de chat.

    (Asegúrate de mantener el nivel de sangría):

    ```python
   # Use the agent to generate an expense claim email
   thread: AzureAIAgentThread = AzureAIAgentThread(client=project_client)
   try:
        # Add the input prompt to a list of messages to be submitted
        prompt_messages = [f"Create an expense claim for the following expenses: {expenses_data}"]
        # Invoke the agent for the specified thread with the messages
        response = await expenses_agent.get_response(thread_id=thread.id, messages=prompt_messages)
        # Display the response
        print(f"\n# {response.name}:\n{response}")
   except Exception as e:
        # Something went wrong
        print (e)
   finally:
        # Cleanup: Delete the thread and agent
        await thread.delete() if thread else None
        await project_client.agents.delete_agent(expenses_agent.id)
    ```

1. Revisa el código completado del agente, con los comentarios que te ayudarán a comprender lo que hace cada bloque de código, y después guarda los cambios de código (**CTRL+S**).

### Inicie sesión en Azure y ejecuta la aplicación.

1. En el panel de línea de comandos de Cloud Shell, debajo del editor de código, escribe el siguiente comando para iniciar sesión en Azure.

    ```
    az login
    ```

    **<font color="red">Debes iniciar sesión en Azure, aunque la sesión de Cloud Shell ya esté autenticada.</font>**

    > **Nota**: en la mayoría de los escenarios, el uso de *inicio de sesión de az* será suficiente. Sin embargo, si tienes suscripciones en varios inquilinos, es posible que tengas que especificar el inquilino mediante el parámetro *--tenant*. Consulta [Inicio de sesión en Azure de forma interactiva mediante la CLI de Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively) para obtener más información.
    
1. Cuando se te solicite, sigue las instrucciones para abrir la página de inicio de sesión en una nueva pestaña y escribe el código de autenticación proporcionado y las credenciales de Azure. A continuación, completa el proceso de inicio de sesión en la línea de comandos y selecciona la suscripción que contiene el centro de Fundición de IA de Azure si se te solicita.
1. Después de iniciar sesión, escribe el siguiente comando para ejecutar la aplicación:

    ```
   python semantic-kernel.py
    ```
    
    La aplicación se ejecuta con las credenciales de la sesión de Azure autenticada para conectarse al proyecto y crear y ejecutar el agente.

    > **Sugerencia**: si se produce un error en la aplicación porque se supera el límite de velocidad. Espere unos segundos y vuelve a intentarlo. Si no hay cuota suficiente disponible en la suscripción, es posible que el modelo no pueda responder.

1. Cuando la aplicación acabe, revisa el resultado. El agente debe haber compuesto un correo electrónico para una notificación de gastos basada en los datos proporcionados.

## Resumen

En este ejercicio, has usado el SDK del Agente de servicio de IA de Azure y el kernel semántico para crear un agente.

## Limpieza

Si has terminado de explorar Agente de servicio de IA de Azure, debes eliminar los recursos que has creado en este ejercicio para evitar incurrir en costes innecesarios de Azure.

1. Vuelve a la pestaña del explorador que contiene Azure Portal (o vuelve a abrir [Azure Portal](https://portal.azure.com) en `https://portal.azure.com` en una nueva pestaña del explorador) y mira el contenido del grupo de recursos donde implementó los recursos usados en este ejercicio.
1. Selecciona **Eliminar grupo de recursos** en la barra de herramientas.
1. Escribe el nombre del grupo de recursos y confirma que deseas eliminarlo.
