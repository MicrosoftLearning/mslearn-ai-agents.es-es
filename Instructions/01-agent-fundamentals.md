---
lab:
  title: Exploración del desarrollo del agente de IA
  description: Sigue los primeros pasos para desarrollar agentes de IA mientras exploras el Agente de servicio de IA de Azure en el Portal de la Fundición de IA de Azure.
---

# Exploración del desarrollo del agente de IA

En este ejercicio, usarás el Agente de servicio de IA de Azure en el Portal de la Fundición de IA de Azure para crear un agente de IA sencillo que ayude a los empleados con las reclamaciones de gastos.

Este ejercicio dura aproximadamente **30** minutos.

> **Nota**: Algunas de las tecnologías que se usan en este ejercicio se encuentran en versión preliminar o en desarrollo activo. Puede que se produzcan algunos comportamientos, advertencias o errores inesperados.

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

## Creación de un agente

Ahora que has implementado un modelo, estás listo para crear un agente de IA. En este ejercicio, crearás un agente sencillo que responde a preguntas basadas en una directiva de gastos corporativos. Descargarás el documento de la directiva de gastos y lo usarás como datos de *fundamentación* para el agente.

1. Abre otra pestaña del explorador y descarga el documento [Expenses_policy.docx](https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-agents/main/Labfiles/01-agent-fundamentals/Expenses_Policy.docx) de `https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-agents/main/Labfiles/01-agent-fundamentals/Expenses_Policy.docx` y guárdalo localmente. Este documento contiene detalles de la directiva de gastos de la empresa ficticia Contoso.
1. Vuelve a la pestaña del explorador que contiene el Portal de la Fundición de IA de Azure y, en el panel de navegación de la izquierda, en la sección **Compilación y personalización**, seleccione la página **Agentes**.
1. Si se te solicita, selecciona el recurso de Azure OpenAI Service y ve.

    Se debería crear automáticamente el nuevo agente con un nombre como *Agent123* (si no es así, use el botón **+ Nuevo agente** para crear uno).

1. Selecciona el nuevo agente. A continuación, en el panel **Configuración** del nuevo agente, establece el **Nombre del agente** en `ExpensesAgent`, asegúrate de que la implementación de modelo gpt-4o que creaste anteriormente está seleccionada y establece las **Instrucciones** en:

    ```prompt
   You are an AI assistant for corporate expenses.
   You answer questions about expenses based on the expenses policy data.
   If a user wants to submit an expense claim, you get their email address, a description of the claim, and the amount to be claimed and write the claim details to a text file that the user can download.
    ```

    ![Captura de pantalla de la página de configuración del agente de IA del Portal de la Fundición de IA de Azure.](./Media/ai-agent-setup.png)

1. Más abajo, en el panel **Configuración**, junto al encabezado **Conocimientos**, selecciona **+ Agregar**. A continuación, en el cuadro de diálogo **Agregar conocimientos**, selecciona **Archivos**.
1. En el cuadro de diálogo **Agregar archivos**, crea un nuevo almacén de vectores denominado `Expenses_Vector_Store`, cargando y guardando el archivo local **Expenses_policy.docx** que descargaste anteriormente.

    ![Captura de pantalla del cuadro de diálogo Agregar datos en el Portal de la Fundición de IA de Azure.](./Media/ai-agent-add-files.png)

1. En el panel **Configuración**, en la sección **Conocimientos**, comprueba que aparece **Expenses_Vector_Store** y que muestra que contiene 1 archivo.

1. Debajo de la sección **Conocimientos**, junto a **Acciones**, selecciona **+ Agregar**. A continuación, en el cuadro de diálogo **Agregar acción**, selecciona **Intérprete de código** y, a continuación, selecciona **Guardar** (no es necesario cargar ningún archivo para el intérprete de código).

    El agente usará el documento que cargaste como origen de conocimiento para *fundamentar* sus respuestas (es decir, responderá a preguntas basadas en el contenido de este documento). Usará la herramienta de intérprete de código según sea necesario para realizar acciones mediante la generación y ejecución de su propio código de Python.

## Prueba del agente

Ahora que has creado un agente, puedes probarlo en el área de juegos del Portal de la Fundición de IA de Azure.

1. En la parte superior del panel **Configuración** del agente, selecciona **Probar en el área de juegos**.
1. En el área de juegos, escribe la indicación: `What's the maximum I can claim for meals?` y revisa la respuesta del agente, que debe estar basada en la información del documento de la directiva de gastos que agregaste como conocimientos al configurar el agente.

    > **Nota**: si el agente no responde es porque se ha superado el límite de frecuencia. Espere unos segundos y vuelve a intentarlo. Si no hay cuota suficiente disponible en la suscripción, es posible que el modelo no pueda responder.

1. Prueba la siguiente solicitud de seguimiento: `I'd like to submit a claim for a meal.` y revisa la respuesta. El agente debe pedirte la información necesaria para enviar una notificación.
1. Proporciona al agente una dirección de correo electrónico; por ejemplo, `fred@contoso.com`. El agente debe confirmar la respuesta y solicitar la información restante necesaria para la notificación de gastos (descripción e importe).
1. Envía un mensaje que describa la notificación y el importe; por ejemplo, `Breakfast cost me $20`.
1. El agente debe usar el intérprete de código para preparar el archivo de texto de notificación de gastos y proporcionar un vínculo para que puedas descargarlo.

    ![Captura de pantalla del área de juegos del agente en el Portal de la Fundición de IA de Azure.](./Media/ai-agent-playground.png)

1. Descarga y abre el documento de texto para ver los detalles de la notificación de gastos.

## Limpieza

Ahora que has terminado el ejercicio, debes eliminar los recursos en la nube que has creado para evitar el uso innecesario de recursos.

1. Abre [Azure Portal](https://portal.azure.com) en `https://portal.azure.com` y visualiza el contenido del grupo de recursos donde implementaste los recursos del centro usados en este ejercicio.
1. Selecciona **Eliminar grupo de recursos** en la barra de herramientas.
1. Escribe el nombre del grupo de recursos y confirma que deseas eliminarlo.
