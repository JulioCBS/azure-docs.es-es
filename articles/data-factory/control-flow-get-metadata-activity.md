---
title: Actividad de obtención de metadatos en Azure Data Factory | Microsoft Docs
description: Obtenga información sobre cómo usar la actividad de obtención de metadatos en una canalización de Data Factory.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.reviewer: ''
ms.assetid: 1c46ed69-4049-44ec-9b46-e90e964a4a8e
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 08/12/2019
ms.author: jingwang
ms.openlocfilehash: 081d7219407decac5dd36a06f289436aa0da627b
ms.sourcegitcommit: 388c8f24434cc96c990f3819d2f38f46ee72c4d8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2019
ms.locfileid: "70061536"
---
# <a name="get-metadata-activity-in-azure-data-factory"></a>Actividad de obtención de metadatos en Azure Data Factory

La actividad de obtención de metadatos se puede usar para recuperar metadatos de cualquier dato en Azure Data Factory. Esta actividad se puede usar en los siguientes escenarios:

- Validar los metadatos de cualquier dato.
- Desencadenar una canalización cuando los datos estén listos/disponibles.

La siguiente funcionalidad está disponible en el flujo de control:

- Puede usar la salida de la actividad de obtención de metadatos en expresiones condicionales para llevar a cabo una validación.
- Puede desencadenar una canalización si se satisface la condición mediante un bucle Do Until.

## <a name="capabilities"></a>Capacidades

La actividad de obtención de metadatos toma un conjunto de datos como entrada y genera información de metadatos como salida. Actualmente, se admiten los siguientes conectores y los metadatos recuperables correspondientes. El tamaño máximo de los metadatos devueltos es 1 MB.

>[!NOTE]
>Si ejecuta la actividad de obtención de metadatos en un entorno de ejecución de integración autohospedado, se admite la funcionalidad más reciente en la versión 3.6 o posterior.

### <a name="supported-connectors"></a>Conectores compatibles

**Almacenamiento de archivos**

| Conector/Metadatos | itemName<br>(archivo/carpeta) | itemType<br>(archivo/carpeta) | size<br>(archivo) | created<br>(archivo/carpeta) | lastModified<br>(archivo/carpeta) |childItems<br>(carpeta) |contentMD5<br>(archivo) | structure<br/>(archivo) | columnCount<br>(archivo) | exists<br>(archivo/carpeta) |
|:--- |:--- |:--- |:--- |:--- |:--- |:--- |:--- |:--- |:--- |:--- |
| [Amazon S3](connector-amazon-simple-storage-service.md) | √/√ | √/√ | √ | x/x | √/√* | √ | x | √ | √ | √/√* |
| [Google Cloud Storage](connector-google-cloud-storage.md) | √/√ | √/√ | √ | x/x | √/√* | √ | x | √ | √ | √/√* |
| [Almacenamiento de blobs de Azure](connector-azure-blob-storage.md) | √/√ | √/√ | √ | x/x | √/√* | √ | √ | √ | √ | √/√ |
| [Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md) | √/√ | √/√ | √ | x/x | √/√ | √ | x | √ | √ | √/√ |
| [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md) | √/√ | √/√ | √ | x/x | √/√ | √ | x | √ | √ | √/√ |
| [Archivos de Azure](connector-azure-file-storage.md) | √/√ | √/√ | √ | √/√ | √/√ | √ | x | √ | √ | √/√ |
| [Sistema de archivos](connector-file-system.md) | √/√ | √/√ | √ | √/√ | √/√ | √ | x | √ | √ | √/√ |
| [SFTP](connector-sftp.md) | √/√ | √/√ | √ | x/x | √/√ | √ | x | √ | √ | √/√ |
| [FTP](connector-ftp.md) | √/√ | √/√ | √ | x/x | √/√ | √ | x | √ | √ | √/√ |

- En el caso de Amazon S3 y Google Cloud Storage, `lastModified` se aplica al cubo y a la clave, pero no a la carpeta virtual, y `exists` se aplica al cubo y a la clave, pero no al prefijo ni a la carpeta virtual.
- En el caso de Azure Blob Storage, `lastModified` se aplica al contenedor y al blob, pero no a la carpeta virtual.

**Base de datos relacional**

| Conector/Metadatos | structure | columnCount | exists |
|:--- |:--- |:--- |:--- |
| [Azure SQL Database](connector-azure-sql-database.md) | √ | √ | √ |
| [Instancia administrada de Azure SQL Database](connector-azure-sql-database-managed-instance.md) | √ | √ | √ |
| [Azure SQL Data Warehouse](connector-azure-sql-data-warehouse.md) | √ | √ | √ |
| [SQL Server](connector-sql-server.md) | √ | √ | √ |

### <a name="metadata-options"></a>Opciones de metadatos

Puede especificar los siguientes tipos de metadatos en la lista de campos de la actividad de obtención de metadatos para recuperar la información correspondiente:

| Tipo de metadatos | DESCRIPCIÓN |
|:--- |:--- |
| itemName | Nombre del archivo o la carpeta. |
| itemType | Tipo del archivo o la carpeta. El valor devuelto es `File` o `Folder`. |
| size | Tamaño del archivo, en bytes. Se aplica solo a los archivos. |
| created | Fecha y hora de creación del archivo o la carpeta. |
| lastModified | Fecha y hora de la última modificación del archivo o la carpeta. |
| childItems | Lista de subcarpetas y archivos dentro de la carpeta determinada. Se aplica solo a las carpetas. El valor devuelto es una lista del nombre y el tipo de cada elemento secundario. |
| contentMD5 | MD5 del archivo. Se aplica solo a los archivos. |
| structure | Estructura de datos del archivo o la tabla de base de datos relacional. El valor devuelto es una lista de nombres de columna y tipos de columna. |
| columnCount | Número de columnas en el archivo o la tabla relacional. |
| exists| Si un archivo, carpeta o tabla existe. Tenga en cuenta que si se especifica `exists` en la lista del campo de obtención de metadatos, la actividad no producirá un error aunque el archivo, la carpeta o la tabla no existan. En su lugar, se devuelve `exists: false` en la salida. |

>[!TIP]
>Si desea validar que existe un archivo, una carpeta o una tabla, especifique `exists` en la lista de campos de la actividad de obtención de metadatos. A continuación, puede comprobar el resultado `exists: true/false` en la salida de la actividad. Si `exists` no se especifica en la lista de campos, se producirá un error en la actividad de obtención de metadatos si no se encuentra el objeto.

>[!NOTE]
>Si obtiene metadatos de almacenes de archivos y configura `modifiedDatetimeStart` o `modifiedDatetimeEnd`, el elemento `childItems` de la salida incluirá solo los archivos de la ruta determinada que tengan una hora de última modificación que se encuentre dentro del intervalo especificado. No incluirá los elementos de las subcarpetas.

## <a name="syntax"></a>Sintaxis

**Actividad de obtención de metadatos**

```json
{
    "name": "MyActivity",
    "type": "GetMetadata",
    "typeProperties": {
        "fieldList" : ["size", "lastModified", "structure"],
        "dataset": {
            "referenceName": "MyDataset",
            "type": "DatasetReference"
        }
    }
}
```

**Dataset**

```json
{
    "name": "MyDataset",
    "properties": {
    "type": "AzureBlob",
        "linkedService": {
            "referenceName": "StorageLinkedService",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "folderPath":"container/folder",
            "filename": "file.json",
            "format":{
                "type":"JsonFormat"
            }
        }
    }
}
```

## <a name="type-properties"></a>Propiedades de tipo

Actualmente, la actividad de obtención de metadatos puede capturar los siguientes tipos de información de metadatos:

Propiedad | DESCRIPCIÓN | Obligatorio
-------- | ----------- | --------
fieldList | Los tipos de información de metadatos necesarios. Para más información sobre los metadatos admitidos, consulte la sección [Opciones de metadatos](#metadata-options) de este artículo. | Sí 
dataset | Conjunto de datos de referencia cuyos metadatos debe recuperar la actividad de obtención de metadatos. Consulte la sección[Capacidades](#capabilities) para obtener información sobre los conectores admitidos. Consulte los temas específicos del conector para obtener información detallada sobre la sintaxis del conjunto de datos. | Sí
formatSettings | Se aplica cuando se usa el tipo de formato dataset. | Sin
storeSettings | Se aplica cuando se usa el tipo de formato dataset. | Sin

## <a name="sample-output"></a>Salida de ejemplo

Los resultados de la obtención de metadatos se muestran en la salida de la actividad. A continuación se muestran dos ejemplos que muestran amplias opciones de metadatos. Para utilizar los resultados en una actividad posterior, utilice este patrón: `@{activity('MyGetMetadataActivity').output.itemName}`.

### <a name="get-a-files-metadata"></a>Obtener los metadatos de un archivo

```json
{
  "exists": true,
  "itemName": "test.csv",
  "itemType": "File",
  "size": 104857600,
  "lastModified": "2017-02-23T06:17:09Z",
  "created": "2017-02-23T06:17:09Z",
  "contentMD5": "cMauY+Kz5zDm3eWa9VpoyQ==",
  "structure": [
    {
        "name": "id",
        "type": "Int64"
    },
    {
        "name": "name",
        "type": "String"
    }
  ],
  "columnCount": 2
}
```

### <a name="get-a-folders-metadata"></a>Obtener los metadatos de una carpeta

```json
{
  "exists": true,
  "itemName": "testFolder",
  "itemType": "Folder",
  "lastModified": "2017-02-23T06:17:09Z",
  "created": "2017-02-23T06:17:09Z",
  "childItems": [
    {
      "name": "test.avro",
      "type": "File"
    },
    {
      "name": "folder hello",
      "type": "Folder"
    }
  ]
}
```

## <a name="next-steps"></a>Pasos siguientes
Consulte otras actividades de flujo de control compatibles con Data Factory:

- [Actividad de ejecución de canalización](control-flow-execute-pipeline-activity.md)
- [Actividad ForEach](control-flow-for-each-activity.md)
- [Actividad de búsqueda](control-flow-lookup-activity.md)
- [Actividad web](control-flow-web-activity.md)
