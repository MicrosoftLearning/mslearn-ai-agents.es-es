---
lab:
  title: "Desarrollo de un agente de IA con la extensión de VS\_Code"
  description: "Use la extensión Microsoft Foundry para VS\_Code para crear un agente de IA."
---

# Desarrollo de un agente de IA con la extensión de VS Code

En este ejercicio, usará la extensión Microsoft Foundry para VS Code crear un agente que pueda usar herramientas de servidor del Protocolo de contexto de modelo (MCP) para acceder a APY y orígenes de datos externos. El agente podrá recuperar información actualizada e interactuar con varios servicios a través de herramientas de MCP.

Este ejercicio debería tardar en completarse **30** minutos aproximadamente.

> **Nota**: algunas de las tecnologías que se usan en este ejercicio se encuentran en versión preliminar o en desarrollo activo. Puede que se produzcan algunos comportamientos, advertencias o errores inesperados.

## Requisitos previos

Antes de iniciar este ejercicio, compruebe lo siguiente:
- Visual Studio Code instalado
- Una suscripción a Azure activa

## Instalación de la extensión Foundry para VS Code

Comencemos instalando y configurando la extensión de VS Code.

1. Abra Visual Studio Code.

1. Seleccione **Extensiones** en el panel izquierdo (o presione **Ctrl+Mayús+X**).

1. En la barra de búsqueda, escriba **Foundry** y presione Entrar.

1. Seleccione la extensión **Foundry** de Microsoft y haga clic en **Instalar**.

1. Una vez completada la instalación, compruebe que la extensión aparece en la barra de navegación principal del lado izquierdo de Visual Studio Code.

## Inicio de sesión en Azure y creación del proyecto

Ahora se conectará a los recursos de Azure y creará un nuevo proyecto de Fundición de IA.

1. En la barra lateral de VS Code, seleccione el icono de la extensión **Foundry**.

1. En la vista de recursos de Azure, seleccione **Iniciar sesión en Azure...** y siga los mensajes de autenticación.

1. Después de iniciar sesión, seleccione la suscripción de Azure en la lista desplegable.

1. Cree un nuevo proyecto de Foundry seleccionando el icono (más) **+** situado junto a **Recursos** en la vista Extensión de Foundry.

1. Elija si desea crear un nuevo grupo de recursos o usar uno existente:
   
   **Para crear un nuevo grupo de recursos:**
   - Seleccione **Crear nuevo grupo de recursos** y presione Entrar.
   - Escriba un nombre para el grupo de recursos (por ejemplo, "rg-ai-agents-lab") y presione Entrar.
   - Seleccione una ubicación de entre las opciones disponibles y presione Entrar.
   
   **Para usar un grupo de recursos existente:**
   - seleccione el grupo de recursos que desea usar en la lista y presione Entrar.

1. Escriba un nombre para el proyecto de Foundry (por ejemplo, "ai-agents-project") en el cuadro de texto y presione Entrar.

1. Espere a que se complete la implementación del proyecto. Aparecerá un mensaje emergente que dice "El proyecto se implementó correctamente".

## Implementación de un modelo

Necesitará un modelo implementado para usarlo con el agente.

1. Cuando aparezca el mensaje emergente "El proyecto se ha implementado correctamente", seleccione el botón **Implementar un modelo**. Se abrirá el catálogo de modelos.

   > **Sugerencia**: También puede acceder al catálogo de modelos seleccionando el icono **+** situado junto a **Modelos** en la sección Recursos, o presionando **F1** y ejecutando el comando **Foundry: Abra el catálogo de modelos**.

1. En el catálogo de modelos, busque el modelo **gpt-4o** (puede usar la barra de búsqueda para encontrarlo rápidamente).

    ![Recorte de pantalla del catálogo de modelos en la extensión Foundry para VS Code.](Media/vs-code-model.png)

1. Seleccione **Implementar en Azure** junto al modelo gpt-4o.

1. Configure las opciones de implementación:
   - **Nombre de implementación**: escriba un nombre como "gpt-4o-deployment".
   - **Tipo de implementación**: seleccione **Estándar global** (o **Estándar** si la primera opción no está disponible).
   - **Versión del modelo**: deje el valor predeterminado.
   - **Tokens por minuto**: deje el valor predeterminado.

1. Seleccione **Implementar en Foundry** en la esquina inferior izquierda.

1. En el cuadro de diálogo de confirmación, seleccione **Implementar** para implementar el modelo.

1. Espere a que la implementación se complete. El modelo implementado aparecerá en la sección **Modelos** de la vista Recursos.

## Creación de un agente de IA con la vista del diseñador

Ahora creará un agente de IA mediante la interfaz del diseñador visual.

1. En la vista de la extensión de Foundry, busque la sección **Recursos**.

1. Seleccione el icono (más) **+** situado junto a la subsección **Agentes** para crear un nuevo agente de IA.

    ![Recorte de pantalla de la creación de un agente en la extensión Foundry para VS Code.](Media/vs-code-new-agent.png)

1. Ciando se le solicite, elija una ubicación para guardar los archivos del agente.

1. La vista del diseñador del agente se abrirá junto con un archivo de configuración `.yaml`.

### Configuración del agente en el diseñador

1. En el diseñador del agente, configure los siguientes campos:
   - **Nombre**: escriba un nombre descriptivo para el agente (por ejemplo, "data-research-agent").
   - **Descripción**: agregue una descripción que explique el propósito del agente.
   - **Modelo**: seleccione la implementación de GPT-4o en la lista desplegable.
   - **Instrucciones**: escriba instrucciones del sistema, como:
     ```
     You are an AI agent that helps users research information from various sources. Use the available tools to access up-to-date information and provide comprehensive responses based on external data sources.
     ```

1. Guarde la configuración seleccionando **Archivo > Guardar** en la barra de menús de VS Code.

## Adición de una herramienta de servidor MCP al agente

Ahora agregará una herramienta de servidor del Protocolo de contexto de modelo (MCP) que permite al agente acceder a API y orígenes de datos externos.

1. En la sección **HERRAMIENTA** del diseñador, seleccione el botón **Agregar herramienta** en la esquina superior derecha.

![Recorte de pantalla de cómo agregar una herramienta a un agente en la extensión Foundry para VS Code.](Media/vs-code-agent-tools.png)

1. En el menú desplegable, elija **Servidor MCP**.

1. Configure la herramienta Servidor MCP con la siguiente información:
   - **Dirección URL del servidor**: escriba la dirección URL de un servidor MCP (por ejemplo, `https://gitmcp.io/Azure/azure-rest-api-specs`).
   - **Etiqueta del servidor**: escriba un identificador único (por ejemplo, "github_docs_server").

1. Deje la lista desplegable **Herramientas permitidas** vacía para permitir todas las herramientas del servidor MCP.

1. Seleccione el botón **Crear herramienta** para agregar la herramienta al agente.

## Implementación del agente en Foundry

1. En la vista del diseñador, seleccione el botón **Crear en Foundry** en la esquina inferior izquierda.

1. Espere a que la implementación se complete.

1. En la barra de navegación de VS Code, actualice la vista **Recursos de Azure**. El agente implementado debería aparecer ahora en la subsección **Agentes**.

## Prueba del agente en el área de juegos

1. Haga clic con el botón derecho en el agente implementado en la subsección **Agentes**.

1. Seleccione **Abrir área de juegos** en el menú contextual.

1. El área de juegos de agentes se abrirá en una nueva pestaña dentro de VS Code.

1. Escriba un mensaje de prueba, como:

   ```output
   Can you help me find documentation about Azure Container Apps and provide an example of how to create one?
   ```

1. Envíe el mensaje y observe los mensajes de autenticación y aprobación de la herramienta Servidor MCP:
    - En este ejercicio, responda **Sin autenticación** al mensaje.
    - Para la preferencia de aprobación de las herramientas de MCP, puede seleccionar **Aprobar siempre**.

1. Revise la respuesta del agente y observe cómo usa la herramienta de servidor MCP para recuperar información externa.

1. Compruebe la sección **Anotaciones del agente** para ver los orígenes de información que usa el agente.

## Generación de código de ejemplo para el agente

1. Haga clic con el botón derecho en el agente implementado y seleccione **Abrir archivo de código** o seleccione el botón **Abrir archivo de código** en la página Preferencias del agente.

1. Elija el SDK preferido en la lista desplegable (Python, .NET, JavaScript o Java).

1. Seleccione el lenguaje de programación que prefiera.

1. Elija el método de autenticación preferido.

1. Revise el código de ejemplo generado que muestra cómo interactuar con el agente mediante programación.

Puede usar este código como punto de partida para compilar aplicaciones que usen el agente de IA.

## Visualización del historial de conversaciones y los subprocesos

1. En la vista **Recursos de Azure**, expanda la subsección **Subprocesos** para ver las conversaciones creadas durante las interacciones del agente.

1. Seleccione un subproceso para ver la página **Detalles del subproceso**, que muestra:
   - Mensajes individuales de la conversación
   - Información de ejecución y detalles de ejecución
   - Respuestas del agente y uso de herramientas

1. Seleccione **Ver información de ejecución** para ver información detallada en JSON sobre cada ejecución.

## Resumen

En este ejercicio, ha usado la extensión Foundry para VS Code para crear un agente de IA con herramientas de servidor MCP. El agente puede acceder a orígenes de datos externos y API a través del Protocolo de contexto de modelo, lo que le permite proporcionar información actualizada e interactuar con varios servicios. También ha aprendido a probar el agente en el área de juegos y a generar código de ejemplo para la interacción mediante programación.

## Limpieza

Cuando haya terminado de explorar la extensión Foundry para VS Code, debe limpiar los recursos para evitar incurrir en costos innecesarios de Azure.

### Eliminación de los agentes

1. En el portal de Foundry, seleccione **Agentes** en el menú de navegación.

1. Seleccione el agente y, a continuación, seleccione el botón **Eliminar**.

### Eliminación de los modelos

1. En VS Code, actualice la vista **Recursos de Azure**.

1. Expanda la subsección **Modelos**.

1. Haga clic con el botón derecho en el modelo implementado y seleccione **Eliminar**.

### Eliminación de otros recursos

1. Abra [Azure Portal](https://portal.azure.com).

1. Vaya al grupo de recursos que contiene los recursos de Fundición de IA.

1. Seleccione **Eliminar grupo de recursos** y confirme la eliminación.