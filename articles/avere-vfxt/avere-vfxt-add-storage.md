---
title: Configurar el almacenamiento de Avere vFXT - Azure
description: Cómo agregar un sistema de almacenamiento back-end a Avere vFXT para Azure
author: ekpgh
ms.service: avere-vfxt
ms.topic: conceptual
ms.date: 01/29/2019
ms.author: v-erkell
ms.openlocfilehash: 6d35d5cdeafb80a36f910d71393802a3affb4df8
ms.sourcegitcommit: 41ca82b5f95d2e07b0c7f9025b912daf0ab21909
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/13/2019
ms.locfileid: "60515905"
---
# <a name="configure-storage"></a>Configurar el almacenamiento

En este paso podrá configurar un sistema de almacenamiento de back-end para el clúster de vFXT.

> [!TIP]
> Si creó un contenedor de blobs de Azure junto con el clúster de Avere vFXT, ese contenedor ya está configurado para su uso y no es necesario agregar el almacenamiento.

Siga estas instrucciones si no creó un contenedor de blobs con el clúster o si quiere agregar un hardware adicional o un sistema de almacenamiento basado en la nube.

Hay dos tareas principales que debe realizar:

1. [Crear un archivador principal](#create-a-core-filer) para conectar el clúster de vFXT a un sistema de almacenamiento existente o a una cuenta de Azure Storage.

1. [Crear una unión de espacio de nombres](#create-a-junction) para definir la ruta de acceso que usarán los clientes.

Tenga en cuenta que en estos pasos usará el panel de control de Avere. Lea [Access the vFXT cluster](avere-vfxt-cluster-gui.md) (Obtener acceso al clúster de vFXT) para aprender a usarlo.

## <a name="create-a-core-filer"></a>Crear un archivador principal

Un "Archivador principal" es un término de vFXT que hace referencia a un sistema de almacenamiento de back-end. El almacenamiento puede ser un dispositivo NAS de hardware como NetApp o Isilon, o puede ser un almacén de objetos en la nube. Puede encontrar más información acerca de los archivadores principales en la [guía de configuración de clústeres de Avere](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/settings_overview.html#managing-core-filers).

Para agregar un archivador principal, elija uno de los dos tipos principales que tiene disponibles:

  * [Archivador principal de NAS](#nas-core-filer): describe cómo agregar un archivador principal de NAS. 
  * [Archivador principal en la nube de Azure Storage](#azure-storage-cloud-core-filer): describe cómo agregar una cuenta de Azure Storage como un archivador principal en la nube.

### <a name="nas-core-filer"></a>Archivador principal de NAS

Un archivador principal de NAS puede ser una instancia local de NetApp o Isilon o un punto de conexión de NAS en la nube. El sistema de almacenamiento debe tener una conexión confiable de alta velocidad al clúster de Avere vFXT; por ejemplo, una conexión de ExpressRoute de 1 Gbps (no una VPN). Asimismo, debe proporcionar acceso a la raíz del clúster a las exportaciones de NAS que se vayan a usar.

Los siguientes pasos agregan un archivador principal de NAS:

1. En el panel de control de Avere, haga clic en la ficha **Settings** (Configuración) que se encuentra en la parte superior.

1. Haga clic en **Core Filer (Archivador principal)**  > **Manage Core Filers (Administrar archivadores principales)** a la izquierda.

1. Haga clic en **Create**(Crear).

   ![Captura de pantalla de la página para agregar un nuevo archivador principal con un cursor sobre el botón Crear](media/avere-vfxt-add-core-filer-start.png)

1. Rellene la información necesaria en el asistente: 

   * Póngale nombre a su archivador principal.
   * Proporcione el nombre de dominio completo (FQDN) si estuviera disponible. En caso contrario, proporcione una dirección IP o el nombre de host que se resuelve en su archivador principal.
   * Elija la clase de archivador de la lista. Si no está seguro, elija **Other** (Otro).

     ![Captura de pantalla de la página para agregar un nuevo archivador principal con el nombre del archivador principal y su nombre de dominio completo](media/avere-vfxt-add-core-filer.png)
  
   * A continuación, haga clic en **Next** (Siguiente) y elija una directiva de caché. 
   * Haga clic en **Add filer** (Agregar archivador).
   * Para obtener más información, consulte [Adding a new NAS core filer](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/new_core_filer_nas.html) (Agregar un nuevo archivador principal de NAS) en la guía de configuración del clúster de Avere.

A continuación, [cree una unión](#create-a-junction).  

### <a name="azure-storage-cloud-core-filer"></a>Archivador principal en la nube de Azure Storage

Para usar Azure Blob Storage como almacenamiento de back-end del clúster de vFXT, debe tener un contenedor vacío para poder agregarlo como archivador principal.

> [!TIP] 
> Si decide crear un contenedor de blobs al mismo tiempo que crea el clúster de Avere vFXT, el script o la plantilla de implementación crean un contenedor de almacenamiento, lo definen como un archivador principal y crean la unión del espacio de nombres como parte de la creación del clúster de vFXT. La plantilla también crea un punto de conexión de servicio de almacenamiento dentro de la red virtual del clúster. 

Para poder agregar Blob Storage al clúster debe realizar estas tareas:

* Crear una cuenta de almacenamiento (paso 1, que se detalla a continuación).
* Crear un contenedor de blobs vacío (pasos 2 y 3).
* Agregar la clave de acceso de almacenamiento como una credencial en la nube para el clúster de vFXT (pasos 4 a 6).
* Agregar el contenedor de blobs como un archivador principal para el clúster de vFXT (pasos 7 a 9).
* Crear una unión de espacio de nombres que los clientes usarán para montar el archivador principal (Sección [Crear una unión](#create-a-junction), que es la misma tanto para el almacenamiento de hardware como para el almacenamiento en la nube).

Para agregar Blob Storage después de crear el clúster, siga estos pasos. 

1. Cree una cuenta de almacenamiento V2 de uso general mediante la siguiente configuración:

   * **Suscripción**: la misma que usó para el clúster de vFXT.
   * **Grupo de recursos**: el mismo que usó para el grupo de clústeres de vFXT (opcional).
   * **Ubicación**: la misma que usó para el clúster de vFXT.
   * **Rendimiento**: estándar (no se admite el almacenamiento Premium).
   * **Tipo de cuenta**: uso general V2 (StorageV2).
   * **Replicación**: almacenamiento con redundancia local (LRS).
   * **Nivel de acceso**: frecuente.
   * **Es necesaria una transferencia segura**: deshabilite esta opción (valor no predeterminado).
   * **Redes virtuales** : no requeridas.

   Puede usar Azure Portal o hacer clic en el botón "Implementar en Azure".

   [![botón para crear la cuenta de almacenamiento](media/deploytoazure.png)](https://ms.portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2FAvere%2Fmaster%2Fsrc%2Fvfxt%2Fstorageaccount%2Fazuredeploy.json)

1. Una vez creada la cuenta, vaya a la página de la cuenta de almacenamiento.

   ![Nueva cuenta de almacenamiento en Azure Portal](media/avere-vfxt-new-storage-acct.png)

1. Cree un contenedor de blobs haciendo clic en **Blobs** en la página de información general y, a continuación, haga clic en **+ Contenedor**. Use cualquier nombre de contenedor y asegúrese de que el acceso está establecido como **Privado**.

   ![Página de blobs de almacenamiento sin ningún contenedor](media/avere-vfxt-blob-no-container.png)

1. Obtenga la clave de cuenta de Azure Storage; para ello, haga clic en la opción **Claves de acceso** de la **configuración**:

   ![GUI de Azure Portal para copiar la clave](media/avere-vfxt-copy-storage-key.png) 

1. Abra el panel de control de Avere para el clúster. Haga clic en **Settings** (Configuración) y abra la opción **Cluster (Clúster)**  > **Cloud Credentials (Credenciales en la nube)** en el panel de navegación izquierdo. En la página de credenciales en la nube, haga clic en **Add Credential** (Agregar credencial).

   ![Haga clic en el botón Add Credential (Agregar credencial); este se encuentra en la página de configuración de credenciales en la nube](media/avere-vfxt-new-credential-button.png)

1. Rellene la información siguiente para crear una credencial para el archivador principal en la nube: 

   | Campo | Valor |
   | --- | --- |
   | Nombre de credencial | cualquier nombre descriptivo |
   | Tipo de servicio | (seleccione la clave de acceso de Azure Storage) |
   | Inquilino | el nombre de cuenta de almacenamiento |
   | Subscription | Id. de suscripción |
   | Clave de acceso de almacenamiento | Clave de cuenta de Azure Storage (copiado en el paso anterior) | 

   Haga clic en **Enviar**.

   ![Se completó el formulario de credenciales en la nube en el panel de control de Avere](media/avere-vfxt-new-credential-submit.png)

1. A continuación, cree el archivador principal. En el lado izquierdo del panel de control Avere, haga clic en **Core Filer (Archivador principal)**  >  **Manage Core Filers (Administrar archivadores principales)** . 

1. Haga clic en el botón **Create** (Crear) situado en la página de configuración **Manage Core Filers** (Administrar archivadores principales).

1. Complete los pasos del asistente:

   * Seleccione la **nube** del tipo de archivador. 
   * Póngale nombre al nuevo archivador principal y haga clic en **Next** (Siguiente).
   * Acepte la directiva de caché predeterminada y vaya a la tercera página.
   * En **Service type** (Tipo de servicio), elija **Azure Storage**. 
   * Elija la credencial que creó anteriormente.
   * Establezca el **contenido de cubo** como **Empty** (Vacío).
   * Cambie la**comprobación del certificado** a **Disabled** (Deshabilitado).
   * Cambie el **modo de compresión** a **None** (Ninguno).  
   * Haga clic en **Next**.
   * En la cuarta página, escriba el nombre del contenedor en **Bucket name (Nombre de cubo)** como *storage_account_name*/*container_name*.
   * Además, puede establecer el **tipo de cifrado** a **None** (Ninguno).  Azure Storage se cifra de forma predeterminada.
   * Haga clic en **Add filer** (Agregar archivador).

   Para obtener más información, consulte [Adding a new cloud core filer](<https://azure.github.io/Avere/legacy/ops_guide/4_7/html/new_core_filer_cloud.html>) (Agregar un nuevo archivador principal en la nube) en la guía de configuración del clúster de Avere. 

La página se actualizará (o puede actualizarla usted mismo) para mostrar el nuevo archivador principal.

A continuación, debe [crear una unión](#create-a-junction).

## <a name="create-a-junction"></a>Crear una unión

Una unión es una ruta de acceso que se crea para los clientes. Los clientes deben usar la ruta de acceso para llegar al destino que usted elija.

Por ejemplo, puede crear `/avere/files` para asignarlo a la exportación del archivador principal de NetApp `/vol0/data` y al subdirectorio `/project/resources`.

Puede encontrar más información sobre las uniones en la [sección del espacio de nombres de la guía de configuración del clúster de Avere](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gui_namespace.html).

Siga estos pasos en la interfaz de configuración del panel de control de Avere:

* Haga clic en **VServer** > **Namespace (Espacio de nombres)** en la parte superior izquierda.
* Debe iniciar la ruta de acceso del espacio de nombres con una barra diagonal (/), como ``/avere/data``.
* Elija su archivador principal.
* Elija la exportación del archivador principal.
* Haga clic en **Next**.

  ![Captura de pantalla de la página "Agregar nueva unión" con los campos completados para la unión, el archivador principal y la exportación](media/avere-vfxt-add-junction.png)

La unión aparecerá pasados unos segundos. Recuerde que puede crear uniones adicionales si fuera necesario.

Una vez creada la unión, los clientes pueden [usar el clúster de Avere vFXT](avere-vfxt-mount-clients.md) para obtener acceso al sistema de archivos.
