---
title: Controles de acceso a la red para Azure SQL Database y Data Warehouse | Microsoft Docs
description: Información general sobre los controles de acceso a la red para Azure SQL Database y Data Warehouse para administrar el acceso y configurar una base de datos única o agrupada.
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: rohitnayakmsft
ms.author: rohitna
ms.reviewer: vanto
ms.date: 08/05/2019
ms.openlocfilehash: 2d7cc217ff8ae45491c0f9d6b54ea8afea19cd2e
ms.sourcegitcommit: 6d2a147a7e729f05d65ea4735b880c005f62530f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2019
ms.locfileid: "69981233"
---
# <a name="azure-sql-database-and-data-warehouse-network-access-controls"></a>Controles de acceso a la red para Azure SQL Database y Data Warehouse

> [!NOTE]
> Este artículo se aplica a Azure SQL Server y tanto a las bases de datos de SQL Database como a SQL Data Warehouse que se crean en el servidor de Azure SQL. Para simplificar, SQL Database se utiliza cuando se hace referencia tanto a SQL Database como a SQL Data Warehouse.

> [!IMPORTANT]
> Este artículo *no* es válido para las **instancias administradas de Azure SQL Database**. Para más información sobre la configuración de red, consulte el artículo de [conexión a una instancia administrada](sql-database-managed-instance-connect-app.md).

Al crear una nueva instancia de Azure SQL Server [en Azure Portal](sql-database-single-database-get-started.md), el resultado es un punto de conexión público con el formato *nombreDelServidor.database.windows.net*. Por diseño, se deniega todo acceso al punto de conexión público. Después, puede usar los siguientes controles de acceso a la red para permitir el acceso a la base de datos SQL de forma selectiva mediante el punto de conexión público.
- Allow Azure Services (Permitir servicios de Azure): cuando se establece en ACTIVADO, otros recursos dentro de los límites de Azure, por ejemplo, una máquina virtual de Azure, pueden acceder a SQL Database.

- Reglas de firewall de IP: use esta característica para permitir explícitamente las conexiones desde una dirección IP específica, por ejemplo, desde máquinas locales.

- Reglas de firewall de la red virtual: use esta característica para permitir el tráfico desde una red virtual específica dentro de los límites de Azure.

> [!VIDEO https://channel9.msdn.com/Shows/Data-Exposed/Data-Exposed--SQL-Database-Connectivity-Explained/player?WT.mc_id=dataexposed-c9-niner]

## <a name="allow-azure-services"></a>Allow Azure Services (Permitir servicios de Azure) 
Durante la creación de una nueva instancia de Azure SQL Server [en Azure Portal](sql-database-single-database-get-started.md), esta configuración se deja desactivada.

 ![Captura de pantalla de creación del nuevo servidor][1]

También puede cambiar esta configuración desde el panel de firewall después de crear la instancia de Azure SQL Server como sigue.
  
 ![Captura de pantalla de la administración del firewall del servidor][2]

Cuando se establece en **ACTIVADO**, Azure SQL Server permite la comunicación de todos los recursos dentro de los límites de Azure, que pueden o no formar parte de la suscripción.

En muchos casos, el valor **ACTIVADO** es más permisivo que el que desean la mayoría de los clientes. Es posible que deseen establecer esta opción en **DESACTIVADO** y reemplazarla por reglas de firewall de IP más restrictivas o reglas de firewall de red virtual. Esto afecta a las siguientes características:

### <a name="import-export-service"></a>Import Export Service

El servicio de importación y exportación de Azure SQL Database se ejecuta en VM de Azure. Estas máquinas virtuales no están en la red virtual y, por tanto, obtienen una dirección IP de Azure al conectarse a la base de datos. Al deshabilitar **Permitir que los servicios de Azure accedan al servidor**, estas máquinas virtuales no podrán acceder a las bases de datos.
Puede evitar el problema con la ejecución de la importación o exportación de BACPAC directamente en el código mediante la API de DACFx.

### <a name="sql-database-query-editor"></a>Editor de consultas de SQL Database

El editor de consultas de Azure SQL Database se implementa en máquinas virtuales de Azure. Estas máquinas virtuales no están en la red virtual. Por tanto, las máquinas virtuales obtienen una dirección IP de Azure cuando se conectan a la base de datos. Al deshabilitar **Permitir que los servicios de Azure accedan al servidor**, estas máquinas virtuales no podrán acceder a las bases de datos.

### <a name="table-auditing"></a>Auditoría de tablas

En la actualidad hay dos maneras de habilitar la auditoría en SQL Database. La auditoría de tablas produce un error después de haber habilitado los puntos de conexión de servicio en el servidor de Azure SQL Server. La solución en este caso consiste en cambiar a la auditoría de blobs.

### <a name="impact-on-data-sync"></a>Impacto en la sincronización de datos

Azure SQL Database tiene la característica Sincronización de datos, que se conecta a las bases de datos con direcciones IP de Azure. Al utilizar los puntos de conexión de servicio, desactivará la opción **Permitir que los servicios de Azure accedan al servidor** de acceso al servidor de SQL Database e interrumpirá la característica Data Sync.

## <a name="ip-firewall-rules"></a>Reglas de firewall de IP
El firewall basado en IP es una característica de Azure SQL Server que impide todo acceso al servidor de bases de datos hasta que se agregan explícitamente las [direcciones IP](sql-database-server-level-firewall-rule.md) de las máquinas cliente.


## <a name="virtual-network-firewall-rules"></a>Reglas de firewall de red virtual

Además de las reglas de IP, el firewall de Azure SQL Server permite definir *reglas de red virtual*.  
Para más información, consulte [Reglas y puntos de conexión de servicio de red virtual para Azure SQL Database](sql-database-vnet-service-endpoint-rule-overview.md).

 ### <a name="azure-networking-terminology"></a>Terminología de redes en Azure  
Tenga en cuenta los siguientes términos de redes en Azure a medida que explora las reglas de firewall de red virtual

**Red virtual:** puede tener redes virtuales asociadas a la suscripción de Azure. 

**Subred:** una red virtual contiene **subredes**. Cualquier máquina virtual (VM) de Azure que tenga se asignará a las subredes. Una subred puede contener varias máquinas virtuales u otros nodos de proceso. Los nodos de proceso que se encuentran fuera de la red virtual no pueden tener acceso a su red virtual a menos que configure la seguridad para que permita el acceso.

**Punto de conexión de servicio de red virtual:** un [punto de conexión de servicio de red virtual][vm-virtual-network-service-endpoints-overview-649d] es una subred cuyos valores de propiedad incluyen uno o más nombres formales de tipo de servicio de Azure. En este artículo nos interesa el nombre de tipo de **Microsoft.Sql**, que hace referencia al servicio de Azure denominado SQL Database.

**Regla de red virtual:** una regla de red virtual para el servidor de SQL Database es una subred que se muestra en la lista de control de acceso (ACL) del servidor de SQL Database. Para estar en la ACL de su SQL Database, la subred debe contener el nombre de tipo **Microsoft.Sql**. Una regla de red virtual indica a su servidor de SQL Database que acepte las comunicaciones procedentes de todos los nodos que están en la subred.


## <a name="ip-vs-virtual-network-firewall-rules"></a>Reglas de IP frente a reglas de firewall de red virtual

El firewall de Azure SQL Server le permite especificar intervalos de direcciones IP desde los que se aceptan las comunicaciones en SQL Database. Este enfoque es preciso para las direcciones IP estables que están fuera de la red privada de Azure. Sin embargo, las máquinas virtuales de la red privada de Azure se configuran con direcciones IP *dinámicas*. Las direcciones IP dinámicas pueden cambiar cuando se reinicia la máquina virtual y, a su vez, invalidar la regla de firewall basada en IP. Sería una locura especificar una dirección IP dinámica en una regla de firewall, en un entorno de producción.

Para superar esta limitación, puede obtener una dirección IP *estática* para la máquina virtual. Para más información, consulte [Configuración de direcciones IP privadas para una máquina virtual mediante Azure Portal][vm-configure-private-ip-addresses-for-a-virtual-machine-using-the-azure-portal-321w]. Sin embargo, el enfoque de IP estática puede resultar difícil de administrar y es costoso cuando se hace a escala. 

Las reglas de red virtual son alternativas más fáciles de establecer y su acceso, de administrar desde una subred específica que contenga las máquinas virtuales.

> [!NOTE]
> Aún no se puede disponer de SQL Database en una subred. Si su servidor de Azure SQL Database fuera un nodo de una subred de la red virtual, todos los nodos de la red virtual podrían comunicarse con su instancia de SQL Database. En este caso, las máquinas virtuales podrían comunicarse con SQL Database sin necesitar ninguna regla IP ni regla de red virtual.

## <a name="next-steps"></a>Pasos siguientes

- Para consultar un inicio rápido sobre cómo crear una regla de firewall por IP de nivel de servidor, consulte [Creación de una base de datos de Azure SQL](sql-database-single-database-get-started.md).

- Para un inicio rápido sobre la creación de una regla de firewall de red virtual de nivel de servidor, consulte [Puntos de conexión de servicio de red virtual y reglas para Azure SQL Database](sql-database-vnet-service-endpoint-rule-overview.md).

- Si desea obtener ayuda para conectarse a una base de datos de Azure SQL desde aplicaciones de código abierto o de terceros, consulte [Ejemplos de código de inicio rápido de cliente para SQL Database](https://msdn.microsoft.com/library/azure/ee336282.aspx).

- Para información sobre los puertos adicionales que puede necesitar abrir, vea la sección **SQL Database: fuera frente a dentro**: de la sección [Puertos más allá de 1433 para ADO.NET 4.5 y SQL Database](sql-database-develop-direct-route-ports-adonet-v12.md)

- Para información general sobre la connectividad de Azure SQL Database, consulte [Arquitectura de conectividad de Azure SQL](sql-database-connectivity-architecture.md).

- Para obtener información general sobre la seguridad de Azure SQL Database, vea [Protección de bases de datos SQL](sql-database-security-overview.md).

<!--Image references-->
[1]: ./media/sql-database-get-started-portal/new-server2.png
[2]: ./media/sql-database-get-started-portal/manage-server-firewall.png
