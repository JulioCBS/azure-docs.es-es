---
title: Habilitación del acceso de lectura público para los contenedores y blobs en Azure Blob Storage | Microsoft Docs
description: Obtenga información acerca de cómo permitir el acceso anónimo a contenedores y blobs y cómo tener acceso a ellos mediante programación.
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 04/30/2019
ms.author: tamram
ms.reviewer: cbrooks
ms.openlocfilehash: 6293fc84969c4e246c05da4482f76142263db230
ms.sourcegitcommit: 5b76581fa8b5eaebcb06d7604a40672e7b557348
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2019
ms.locfileid: "68985559"
---
# <a name="manage-anonymous-read-access-to-containers-and-blobs"></a>Administración del acceso de lectura anónimo a contenedores y blobs

Puede habilitar el acceso de lectura anónimo y público a un contenedor y sus blobs en Azure Blob Storage. Al hacerlo, puede conceder acceso de solo lectura a estos recursos sin compartir la clave de cuenta y sin necesidad de una firma de acceso compartido (SAS).

El acceso de lectura público es mejor para escenarios donde desea que ciertos blobs estén siempre disponibles para el acceso de lectura anónimo. Para un control más minucioso, puede crear una firma de acceso compartido. Las firmas de acceso compartido le permiten proporcionar acceso restringido con distintos permisos para un período específico. Para obtener más información sobre la creación de firmas de acceso compartido, vea [Uso de firmas de acceso compartido (SAS) en Azure Storage](../common/storage-sas-overview.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).

## <a name="grant-anonymous-users-permissions-to-containers-and-blobs"></a>Concesión de permisos a usuarios anónimos a contenedores y blobs

De forma predeterminada, solo un usuario al que se han concedido los permisos adecuados puede acceder a un contenedor y a los blobs dentro de él. Para conceder a usuarios anónimos accedo de lectura a un contenedor y sus blobs, puede establecer el nivel de acceso público del contenedor. Cuando concede acceso público a un contenedor, los usuarios anónimos pueden leer los blobs dentro de un contenedor con acceso público sin necesidad de tener que autorizar la solicitud.

Puede configurar un contenedor con los permisos siguientes:

* **Sin acceso de lectura público:** Solo puede acceder al contenedor y a sus blobs el propietario de cuenta de almacenamiento. Este es el valor predeterminado para todos los contenedores nuevos.
* **Acceso de lectura público solo para blobs:** Los blobs dentro de este contenedor pueden leerse a través de una solicitud anónima, pero los datos del contenedor no están disponibles. Los clientes anónimos no pueden enumerar los blobs dentro del contenedor.
* **Acceso de lectura público para contenedores y blobs:** Todos los datos del contenedor y de los blobs se pueden leer mediante una solicitud anónima. Los clientes pueden enumerar los blobs del contenedor a través de una solicitud anónima, pero no pueden enumerar los contenedores que están en la cuenta de almacenamiento.

Puede usar lo siguiente para establecer permisos de contenedor:

* [Azure Portal](https://portal.azure.com)
* [Azure PowerShell](../common/storage-powershell-guide-full.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)
* [CLI de Azure](../common/storage-azure-cli.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#create-and-manage-blobs)
* Mediante programación, con una de las bibliotecas de cliente de almacenamiento o la API de REST.

### <a name="set-container-public-access-level-in-the-azure-portal"></a>Establecer el nivel de acceso público de contenedor en Azure Portal

Desde [Azure Portal](https://portal.azure.com), puede actualizar el nivel de acceso público para uno o varios contenedores:

1. Vaya a la cuenta de almacenamiento en Azure Portal.
1. En **Blob service** en la hoja de menú, seleccione **Blobs**.
1. Seleccione los contenedores para el que quiere establecer el nivel de acceso público.
1. Use el botón **Cambiar nivel de acceso** para mostrar la configuración de acceso público.
1. Seleccione el nivel de acceso público deseado en la lista desplegable **Nivel de acceso público** y haga clic en el botón Aceptar para aplicar el cambio a los contenedores seleccionados.

En la siguiente captura de pantalla se muestra cómo cambiar el nivel de acceso público para los contenedores seleccionados.

![Captura de pantalla que muestra cómo establecer el nivel de acceso público en el portal](./media/storage-manage-access-to-resources/storage-manage-access-to-resources-0.png)

> [!NOTE]
> No puede cambiar el nivel de acceso público para un blob individual. El nivel de acceso público se establece en el nivel de contenedor.

### <a name="set-container-public-access-level-with-net"></a>Establecer el nivel de acceso público de contenedor con .NET

Para configurar permisos para un contenedor con C# y la Biblioteca del cliente de almacenamiento para .NET, primero recupere los permisos existentes del contenedor llamando al método **GetPermissions**. A continuación, establezca la propiedad **PublicAccess** para el objeto **BlobContainerPermissions** devuelto por el método **GetPermissions**. Por último, llame al método **SetPermissions** con los permisos actualizados.

El ejemplo siguiente establece los permisos del contenedor para el acceso de lectura público completo. Para establecer permisos de acceso de lectura público solo para blobs, establezca la propiedad **PublicAccess** en **BlobContainerPublicAccessType.Blob**. Para quitar todos los permisos para los usuarios anónimos, establezca la propiedad en **BlobContainerPublicAccessType.Off**.

```csharp
public static void SetPublicContainerPermissions(CloudBlobContainer container)
{
    BlobContainerPermissions permissions = container.GetPermissions();
    permissions.PublicAccess = BlobContainerPublicAccessType.Container;
    container.SetPermissions(permissions);
}
```

## <a name="access-containers-and-blobs-anonymously"></a>Acceso anónimo a contenedores y blobs

Un cliente que tiene acceso a contenedores y blobs de forma anónima puede utilizar constructores que no requieren credenciales. En los ejemplos siguientes se muestran varias maneras diferentes de hacer referencia a contenedores y blobs de forma anónima.

### <a name="create-an-anonymous-client-object"></a>Creación de un objeto de cliente anónimo

Puede crear un nuevo objeto de cliente de servicio para el acceso anónimo proporcionando el punto de conexión de almacenamiento de Blob para la cuenta. Sin embargo, también debe conocer el nombre de un contenedor en esa cuenta que esté disponible para el acceso anónimo.

```csharp
public static void CreateAnonymousBlobClient()
{
    // Create the client object using the Blob storage endpoint.
    CloudBlobClient blobClient = new CloudBlobClient(new Uri(@"https://storagesample.blob.core.windows.net"));

    // Get a reference to a container that's available for anonymous access.
    CloudBlobContainer container = blobClient.GetContainerReference("sample-container");

    // Read the container's properties. Note this is only possible when the container supports full public read access.
    container.FetchAttributes();
    Console.WriteLine(container.Properties.LastModified);
    Console.WriteLine(container.Properties.ETag);
}
```

### <a name="reference-a-container-anonymously"></a>Referencia a un contenedor de forma anónima

Si tiene la dirección URL de un contenedor que está disponible de forma anónima, puede utilizarla para hacer referencia al contenedor directamente.

```csharp
public static void ListBlobsAnonymously()
{
    // Get a reference to a container that's available for anonymous access.
    CloudBlobContainer container = new CloudBlobContainer(new Uri(@"https://storagesample.blob.core.windows.net/sample-container"));

    // List blobs in the container.
    foreach (IListBlobItem blobItem in container.ListBlobs())
    {
        Console.WriteLine(blobItem.Uri);
    }
}
```

### <a name="reference-a-blob-anonymously"></a>Referencia a un blob de forma anónima

Si tiene la dirección URL a un blob que está disponible para el acceso anónimo, puede hacer referencia al blob directamente con esa dirección URL:

```csharp
public static void DownloadBlobAnonymously()
{
    CloudBlockBlob blob = new CloudBlockBlob(new Uri(@"https://storagesample.blob.core.windows.net/sample-container/logfile.txt"));
    blob.DownloadToFile(@"C:\Temp\logfile.txt", System.IO.FileMode.Create);
}
```

## <a name="features-available-to-anonymous-users"></a>Características disponibles para los usuarios anónimos

En la siguiente tabla se indican las operaciones a las que puede llamar de forma anónima cuando un contenedor se ha configurado para el acceso público.

| Operación REST | Acceso de lectura público a un contenedor | Acceso de lectura público solo a los blobs |
| --- | --- | --- |
| List Containers | Solo las solicitudes autorizadas | Solo las solicitudes autorizadas |
| Create Container | Solo las solicitudes autorizadas | Solo las solicitudes autorizadas |
| Get Container Properties | Se permiten solicitudes anónimas | Solo las solicitudes autorizadas |
| Get Container Metadata | Se permiten solicitudes anónimas | Solo las solicitudes autorizadas |
| Set Container Metadata | Solo las solicitudes autorizadas | Solo las solicitudes autorizadas |
| Get Container ACL | Solo las solicitudes autorizadas | Solo las solicitudes autorizadas |
| Set Container ACL | Solo las solicitudes autorizadas | Solo las solicitudes autorizadas |
| Delete Container | Solo las solicitudes autorizadas | Solo las solicitudes autorizadas |
| List Blobs | Se permiten solicitudes anónimas | Solo las solicitudes autorizadas |
| Put Blob | Solo las solicitudes autorizadas | Solo las solicitudes autorizadas |
| Get Blob | Se permiten solicitudes anónimas | Se permiten solicitudes anónimas |
| Get Blob Properties | Se permiten solicitudes anónimas | Se permiten solicitudes anónimas |
| Set Blob Properties | Solo las solicitudes autorizadas | Solo las solicitudes autorizadas |
| Get Blob Metadata | Se permiten solicitudes anónimas | Se permiten solicitudes anónimas |
| Set Blob Metadata | Solo las solicitudes autorizadas | Solo las solicitudes autorizadas |
| Put Block | Solo las solicitudes autorizadas | Solo las solicitudes autorizadas |
| Get Block List (solo bloques confirmados) | Se permiten solicitudes anónimas | Se permiten solicitudes anónimas |
| Get Block List (solo bloques sin confirmar o todos los bloques) | Solo las solicitudes autorizadas | Solo las solicitudes autorizadas |
| Put Block List | Solo las solicitudes autorizadas | Solo las solicitudes autorizadas |
| Delete Blob | Solo las solicitudes autorizadas | Solo las solicitudes autorizadas |
| Copia de blobs | Solo las solicitudes autorizadas | Solo las solicitudes autorizadas |
| Instantánea de blob | Solo las solicitudes autorizadas | Solo las solicitudes autorizadas |
| Lease Blob | Solo las solicitudes autorizadas | Solo las solicitudes autorizadas |
| Put Page | Solo las solicitudes autorizadas | Solo las solicitudes autorizadas |
| Get Page Ranges | Se permiten solicitudes anónimas | Se permiten solicitudes anónimas |
| Append Blob | Solo las solicitudes autorizadas | Solo las solicitudes autorizadas |

## <a name="next-steps"></a>Pasos siguientes

* [Authorization for the Azure Storage Services](https://docs.microsoft.com/rest/api/storageservices/authorization-for-the-azure-storage-services) (Autorización para los servicios de Azure Storage)
* [Uso de Firmas de acceso compartido (SAS)](../common/storage-sas-overview.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)