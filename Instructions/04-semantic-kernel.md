---
lab:
  title: Desarrollo de un agente de Azure AI con el SDK de Microsoft Agent Framework
  description: Aprenda a usar el SDK de Microsoft Agent Framework para crear y usar un agente de chat de Azure AI.
---

# Desarrollo de un agente de chat de Azure AI con el SDK de Microsoft Agent Framework

En este ejercicio, usará el servicio Agente de Azure AI y Microsoft Agent Framework para crear un agente de IA que procese notificaciones de gastos.

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
    - **Región**: *seleccione cualquiera (se recomienda Fundición de IA\*).

    > \* Algunos de los recursos de Azure AI están restringidos por cuotas de modelo regionales. En caso de que se alcance un límite de cuota más adelante en el ejercicio, es posible que tengas que crear otro recurso en otra región.

1. Selecciona **Crear** y espera a que se cree el proyecto, incluida la implementación del modelo gpt-4 que seleccionaste.
1. Cuando se cree el proyecto, el área de juegos de chat se abrirá automáticamente.
1. En el panel **Configuración**, anota el nombre de la implementación del modelo; que debe ser **gpt-4o**. Para confirmarlo, mira la implementación en la página **Modelos y puntos de conexión** (simplemente abre esa página en el panel de navegación de la izquierda).
1. En el panel de navegación de la izquierda, selecciona **Información general** para ver la página principal del proyecto; que tiene este aspecto:

    ![Captura de pantalla de los detalles de un proyecto de Azure AI en el Portal de la Fundición de IA de Azure.](./Media/ai-foundry-project.png)

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
   cd ai-agents/Labfiles/04-agent-framework/python
   ls -a -l
    ```

    Los archivos proporcionados incluyen código de aplicación un archivo para las opciones de configuración y un archivo que contiene datos de gastos.

### Configuración de la aplicación

1. En el panel de la línea de comandos de Cloud Shell, escribe el siguiente comando para instalar las bibliotecas que vas a usar:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install azure-identity agent-framework
    ```

1. Escribe el siguiente comando para editar el archivo de configuración que se ha proporcionado:

    ```
   code .env
    ```

    El archivo se abre en un editor de código.

1. En el archivo de código, reemplaza el marcador de posición **your_project_endpoint** con el punto de conexión de tu proyecto (copiado de la página **Información general** del proyecto en el portal de la Fundición de IA de Azure), y el marcador de posición **your_model_deployment** con el nombre que has asignado a la implementación de tu modelo gpt-4o.
1. Después de reemplazar los marcadores de posición, usa el comando **CTRL+S** para guardar los cambios y, a continuación, usa el comando **CTRL+Q** para cerrar el editor de código mientras mantienes abierta la línea de comandos de Cloud Shell.

### Escritura de código para una aplicación de agente

> **Sugerencia**: al agregar código, asegúrate de mantener la sangría correcta. Usa los comentarios existentes como guía al escribir el nuevo código con el mismo nivel de sangría.

1. Escribe el siguiente comando para editar el archivo de código de agente que se ha proporcionado:

    ```
   code agent-framework.py
    ```

1. Revisa el código de este archivo. Contiene:
    - Algunas instrucciones**de importación** para agregar referencias a espacios de nombres usados habitualmente
    - Una función *principal* que carga un archivo que contiene datos de gastos, pide instrucciones al usuario y después llama a...
    - Una función **process_expenses_data** en la que se debe agregar el código para crear y usar tu agente

1. En la parte superior del archivo, después de la instrucción **import** existente, busca el comentario **Agregar referencias**, y agrega el siguiente código para hacer referencia a los espacios de nombres en las bibliotecas que necesitarás para implementar tu agente:

    ```python
   # Add references
   from agent_framework import AgentThread, ChatAgent
   from agent_framework.foundry import FoundryChatClient
   from azure.identity.aio import AzureCliCredential
   from pydantic import Field
   from typing import Annotated
    ```

1. Cerca de la parte inferior del archivo, busque el comentario **Crear una función de herramienta para la funcionalidad de correo electrónico** y agregue el código siguiente para definir una función que el agente usará para enviar correo electrónico (las herramientas son una manera de agregar funcionalidad personalizada a los agentes).

    ```python
   # Create a tool function for the email functionality
   def send_email(
    to: Annotated[str, Field(description="Who to send the email to")],
    subject: Annotated[str, Field(description="The subject of the email.")],
    body: Annotated[str, Field(description="The text body of the email.")]):
        print("\nTo:", to)
        print("Subject:", subject)
        print(body, "\n")
    ```

    > **Nota**: la función *simula* el envío de un correo electrónico mediante la impresión en la consola. En una aplicación real, usarías un servicio SMTP o similar para enviar realmente el correo electrónico.

1. Haga una copia de seguridad sobre el código de **send_email**, en la función **process_expenses_data**, busque el comentario **Crear un agente de chat** y agregue el código siguiente para crear un objeto **ChatAgent** con las herramientas e instrucciones.

    (Asegúrate de mantener el nivel de sangría)

    ```python
   # Create a chat agent
   async with (
       AzureCliCredential() as credential,
       ChatAgent(
           chat_client=FoundryChatClient(async_credential=credential),
           name="expenses_agent",
           instructions="""You are an AI assistant for expense claim submission.
                           When a user submits expenses data and requests an expense claim, use the plug-in function to send an email to expenses@contoso.com with the subject 'Expense Claim`and a body that contains itemized expenses with a total.
                           Then confirm to the user that you've done so.""",
           tools=send_email,
       ) as agent,
   ):
    ```

    Tenga en cuenta que el objeto **AzureCliCredential** incluirá automáticamente las opciones del proyecto de la Fundición de IA de Azure de la configuración.

1. Busca el comentario **Usar el agente para procesar los datos de gastos**, y agrega el siguiente código para crear un subproceso en el que se ejecutará tu agente y después invocarlo con un mensaje de chat.

    (Asegúrate de mantener el nivel de sangría):

    ```python
   # Use the agent to process the expenses data
   try:
       # Add the input prompt to a list of messages to be submitted
       prompt_messages = [f"{prompt}: {expenses_data}"]
       # Invoke the agent for the specified thread with the messages
       response = await agent.run(prompt_messages)
       # Display the response
       print(f"\n# {response.name}:\n{response}")
   except Exception as e:
       # Something went wrong
       print (e)
    ```

1. Revisa que el código completado del agente, con los comentarios que te ayudarán a comprender lo que hace cada bloque de código y después guarda los cambios de código (**CTRL+S**).
1. Mantén abierto el editor de código en caso de que necesites corregir cualquier error tipográfico en el código, pero cambia el tamaño de los paneles para que puedas ver más de la consola de la línea de comandos.

### Inicia sesión en Azure y ejecuta la aplicación

1. En el panel de línea de comandos de Cloud Shell, debajo del editor de código, escribe el siguiente comando para iniciar sesión en Azure.

    ```
    az login
    ```

    **<font color="red">Debes iniciar sesión en Azure, aunque la sesión de Cloud Shell ya esté autenticada.</font>**

    > **Nota**: en la mayoría de los escenarios, el uso de *inicio de sesión de az* será suficiente. Sin embargo, si tienes suscripciones en varios inquilinos, es posible que tengas que especificar el inquilino mediante el parámetro *--tenant*. Consulta [Inicio de sesión en Azure de forma interactiva mediante la CLI de Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively) para obtener más información.
    
1. Cuando se te solicite, sigue las instrucciones para abrir la página de inicio de sesión en una nueva pestaña y escribe el código de autenticación proporcionado y las credenciales de Azure. A continuación, completa el proceso de inicio de sesión en la línea de comandos y selecciona la suscripción que contiene el centro de Fundición de IA de Azure si se te solicita.
1. Después de iniciar sesión, escribe el siguiente comando para ejecutar la aplicación:

    ```
   python agent-framework.py
    ```
    
    La aplicación se ejecuta con las credenciales de la sesión de Azure autenticada para conectarse al proyecto y crear y ejecutar el agente.

1. Cuando se te pregunte qué hacer con los datos de gastos, escribe el siguiente mensaje:

    ```
   Submit an expense claim
    ```

1. Una vez finalizada la aplicación, revisa la salida. El agente debe haber compuesto un correo electrónico para una notificación de gastos en función de los datos proporcionados.

    > **Sugerencia**: si se produce un error en la aplicación porque se supera el límite de velocidad. Espere unos segundos y vuelve a intentarlo. Si no hay cuota suficiente disponible en la suscripción, es posible que el modelo no pueda responder.

## Resumen

En este ejercicio, ha usado el SDK de Microsoft Agent Framework para crear un agente con una herramienta personalizada.

## Limpieza

Si has terminado de explorar Agente de servicio de IA de Azure, debes eliminar los recursos que has creado en este ejercicio para evitar incurrir en costes innecesarios de Azure.

1. Vuelve a la pestaña del explorador que contiene Azure Portal (o vuelve a abrir [Azure Portal](https://portal.azure.com) en `https://portal.azure.com` en una nueva pestaña del explorador) y mira el contenido del grupo de recursos donde implementó los recursos usados en este ejercicio.
1. Selecciona **Eliminar grupo de recursos** en la barra de herramientas.
1. Escribe el nombre del grupo de recursos y confirma que deseas eliminarlo.
