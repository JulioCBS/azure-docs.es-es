---
title: Migración de bases de datos desde una instancia de SQL Server a Instancia administrada de Azure SQL Database | Microsoft Docs
description: Aprenda a migrar una base de datos desde una instancia de SQL Server a Instancia administrada de Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: migration
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: bonova
ms.author: bonova
ms.reviewer: douglas, carlrab
ms.date: 07/11/2019
ms.openlocfilehash: 9508ce927ef03c83f1c4ef7bf28d2fc02b831a99
ms.sourcegitcommit: aa042d4341054f437f3190da7c8a718729eb675e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/09/2019
ms.locfileid: "68879935"
---
# <a name="sql-server-instance-migration-to-azure-sql-database-managed-instance"></a>Migración de una instancia de SQL Server a Instancia administrada de Azure SQL Database

En este artículo se describen los métodos para migrar una instancia de SQL Server 2005 o versiones posteriores a [Instancia administrada de Azure SQL Database](sql-database-managed-instance.md). Para obtener información sobre cómo migrar a una base de datos única o a un grupo elástico, consulte [Migración a una base de datos única o a una base de datos agrupada](sql-database-cloud-migrate.md). Para obtener información sobre la migración desde otras plataformas, vea [Guía de Azure Database Migration](https://datamigration.microsoft.com/).

> [!NOTE]
> Si desea empezar rápidamente y probar la instancia administrada, puede ir a [la guía de inicio rápido](sql-database-managed-instance-quickstart-guide.md) en lugar de a esta página. 

En un nivel alto, el proceso de migración de la base de datos es parecido a este:

![proceso de migración](./media/sql-database-managed-instance-migration/migration-process.png)

- [Evalúe la compatibilidad de la instancia administrada](#assess-managed-instance-compatibility) en la que debe asegurarse de que no haya problemas de bloqueo que puedan impedir las migraciones.
  - Este paso también incluye la creación de la [base de referencia del rendimiento](#create-performance-baseline) para determinar el uso de los recursos en la instancia de SQL Server de origen. Este paso es necesario si desea implementar correctamente el tamaño de la Instancia administrada y comprobar que el rendimiento después de la migración no se ve afectado.
- [Elección de las opciones de conectividad de las aplicaciones](sql-database-managed-instance-connect-app.md)
- [Implemente en una Instancia administrada de tamaño óptimo](#deploy-to-an-optimally-sized-managed-instance), donde elegirá las características técnicas (número de núcleos virtuales, cantidad de memoria) y el nivel de rendimiento (Crítico para la empresa, De uso general) de la Instancia administrada.
- [Seleccione el método de migración y migre](#select-migration-method-and-migrate) donde vaya a migrar las bases de datos con la migración sin conexión (copia de seguridad o restauración nativas, importación y exportación de bases de datos) o con la migración en línea (Data Migration Service, replicación transaccional).
- [Supervise las aplicaciones](#monitor-applications) para asegurarse de que logra el rendimiento esperado.

> [!NOTE]
> Para migrar una única base de datos a una sola base de datos o a un grupo elástico, consulte [Migración de una base de datos de SQL Server a Azure SQL Database](sql-database-single-database-migrate.md).

## <a name="assess-managed-instance-compatibility"></a>Evaluación de la compatibilidad de Instancia administrada

En primer lugar, determine si Instancia administrada es compatible con los requisitos de base de datos de su aplicación. La opción de implementación de Instancia administrada se ha diseñado para poder migrar mediante lift-and-shift la mayoría de las aplicaciones existentes que usan instancias de SQL Server locales o en máquinas virtuales. Sin embargo, a veces podría necesitar características o funcionalidades que todavía no se admiten y el costo de implementar una solución alternativa es demasiado alto.

Use [Data Migration Assistant (DMA)](https://docs.microsoft.com/sql/dma/dma-overview) para detectar posibles problemas de compatibilidad que afecten a la funcionalidad de la base de datos en Azure SQL Database. DMA aún no admite Instancia administrada como destino de migración, pero se recomienda ejecutar la evaluación en Azure SQL Database y revisar atentamente la lista de problemas notificados sobre compatibilidad y paridad de las características con la documentación del producto. Consulte las [características de Azure SQL Database](sql-database-features.md) para comprobar si hay algún problema de bloqueo notificado que no bloquee la instancia administrada, puesto que la mayoría de los problemas de bloqueo que evitaban el proceso de migración a Azure SQL Database se han eliminado con Instancia administrada. Por ejemplo, características tales como consultas entre bases de datos, transacciones entre bases de datos dentro de la misma instancia, servidor vinculado a otros orígenes SQL, CLR, tablas temporales globales, vistas de nivel de instancia, Service Broker y similares están disponibles en Instancia administrada.

Si se ha notificado algún problema de bloqueo que no se haya eliminado con la opción de implementación de Instancia administrada, quizá deba considerar una alternativa, como [SQL Server en máquinas virtuales de Azure](https://azure.microsoft.com/services/virtual-machines/sql-server/). Estos son algunos ejemplos:

- Si necesita acceso directo al sistema operativo o al sistema de archivos, por ejemplo, para instalar agentes personalizados o de otros fabricantes en la misma máquina virtual con SQL Server.
- Si las características tienen dependencias estrictas que aún no se admiten, por ejemplo, FileStream o FileTable, PolyBase y transacciones entre instancias.
- Si es imprescindible que permanezca en una versión específica de SQL Server (2012, por ejemplo).
- Si los requisitos de proceso son mucho menores que los que la instancia administrada ofrece (un núcleo virtual, por ejemplo) y la consolidación de la base de datos no es una opción aceptable.

Si ha resuelto todos los bloqueadores identificados de la migración y va a continuar con la migración a Instancia administrada, tenga en cuenta que algunos de los cambios podrían afectar al rendimiento de la carga de trabajo:
- El modelo de recuperación completa obligatorio y la programación de copias de seguridad automatizadas y regulares podría afectar al rendimiento de la carga de trabajo o al de las acciones de mantenimiento o de extracción, transformación y carga de datos si ha empleado periódicamente el modelo simple o el modelo para cargas masivas de registros, o detenido copias de seguridad a petición.
- Distintas configuraciones en el nivel de servidor o de base de datos como marcas de seguimiento o niveles de compatibilidad
- Las nuevas características que usa como Cifrado de datos transparente (TDE) o los grupos de conmutación por error automática podrían afectar a la CPU y al uso de E/S.

Instancia administrada garantiza el 99,99 % de disponibilidad incluso en escenarios críticos, por lo que la sobrecarga provocada por estas características no se puede deshabilitar. Para más información, consulte [las causas principales que podrían provocar un rendimiento diferente en SQL Server e Instancia administrada](https://azure.microsoft.com/blog/key-causes-of-performance-differences-between-sql-managed-instance-and-sql-server/).

### <a name="create-performance-baseline"></a>Creación de base de referencia de rendimiento

Si necesita comparar el rendimiento de la carga de trabajo en Instancia administrada con la carga de trabajo original que se ejecuta en SQL Server, deberá crear una base de referencia de rendimiento que se usará para la comparación. 

La base de referencia del rendimiento es un conjunto de parámetros como el uso de CPU promedio/máximo, la latencia de E/S de disco promedio/máxima, el rendimiento, IOPS, la duración prevista promedio/máxima de la página o el tamaño promedio máximo de tempdb. Es conveniente tener parámetros similares o incluso mejores después de la migración, por lo que es importante medir y registrar los valores de base de referencia para estos parámetros. Además de los parámetros del sistema, deberá seleccionar un conjunto de consultas representativas o las consultas más importantes de la carga de trabajo y medir la duración mínima, promedio y máxima, y el uso de CPU para las consultas seleccionadas. Estos valores le permitirían comparar el rendimiento de la carga de trabajo que se ejecuta en la instancia administrada con los valores originales del servidor SQL Server de origen.

Algunos de los parámetros que necesitará medir en su instancia de SQL Server son: 
- [Supervisión del uso de la CPU en la instancia de SQL Server](https://techcommunity.microsoft.com/t5/Azure-SQL-Database/Monitor-CPU-usage-on-SQL-Server/ba-p/680777#M131) y registro del uso medio y uso máximo de la CPU.
- [Supervisión del uso de memoria en la instancia de SQL Server](https://docs.microsoft.com/sql/relational-databases/performance-monitor/monitor-memory-usage) y determinación de la cantidad de memoria utilizada por los distintos componentes como el grupo de búferes, la caché de planes, el grupo de almacenes de columnas, [OLTP en memoria](https://docs.microsoft.com/sql/relational-databases/in-memory-oltp/monitor-and-troubleshoot-memory-usage?view=sql-server-2017), etc. Además, debe buscar los valores promedio y máximo del contador de rendimiento de la memoria de duración prevista de la página.
- Supervise el uso de E/S de disco en la instancia de SQL Server de origen mediante la vista [sys.dm_io_virtual_file_stats](https://docs.microsoft.com/sql/relational-databases/system-dynamic-management-views/sys-dm-io-virtual-file-stats-transact-sql) o los [contadores de rendimiento](https://docs.microsoft.com/sql/relational-databases/performance-monitor/monitor-disk-usage).
- Supervise el rendimiento de la carga de trabajo y de la consulta o la instancia de SQL Server mediante el análisis de vistas de administración dinámica o el almacén de consultas si va a migrar desde la versión SQL Server 2016 o versiones posteriores. Identifique el promedio de duración y el uso de CPU de las consultas más importantes de la carga de trabajo para compararlos con las consultas que se ejecutan en la instancia administrada.

> [!Note]
> Si observa algún problema con la carga de trabajo de SQL Server como un uso elevado de CPU, presión de memoria constante, problemas de tempdb o de parametrización, debe intentar resolverlos en la instancia de SQL Server de origen antes tomar la base de referencia y realizar la migración. La migración de problemas conocidos a cualquier sistema nuevo podría provocar resultados inesperados e invalidar cualquier comparación de rendimiento.

Como resultado de esta actividad debería tener documentados los valores promedio y máximo de la CPU, la memoria y el uso de E/S en el sistema de origen, así como la duración promedio y la duración máxima y el uso de la CPU de la consulta dominante y la consulta más crítica de la carga de trabajo. Deberá usar estos valores más adelante para comparar el rendimiento de la carga de trabajo en Instancia administrada con el rendimiento de la base de referencia de la carga de trabajo de la instancia de SQL Server de origen.

## <a name="deploy-to-an-optimally-sized-managed-instance"></a>Implementación en una instancia administrada con tamaño óptimo

Instancia administrada se ha diseñado para cargas de trabajo locales que se van a mover a la nube. Presenta un [nuevo modelo de compra](sql-database-service-tiers-vcore.md) que ofrece mayor flexibilidad para seleccionar el nivel adecuado de recursos para las cargas de trabajo. En el mundo local, probablemente está acostumbrado a ajustar el tamaño de estas cargas de trabajo mediante el uso de núcleos físicos y ancho de banda de E/S. El modelo de compra de Instancia administrada se basa en núcleos virtuales con almacenamiento adicional y E/S disponible por separado. El modelo de núcleos virtuales es una manera sencilla de comprender los requisitos de proceso en la nube en comparación con lo que usa en su entorno local hoy en día. Este nuevo modelo permite elegir el tamaño adecuado para el entorno de destino en la nube. Aquí se describen algunas directrices generales que pueden ayudarle a elegir las características y el nivel de servicio correctos:
- Según el uso de CPU de la base, puede aprovisionar una Instancia administrada que coincida con el número de núcleos que usa en SQL Server, teniendo en cuenta que puede que las características de la CPU deban escalarse para que coincidan con las [características de la máquina virtual en la que está instalada la Instancia administrada](sql-database-managed-instance-resource-limits.md#hardware-generation-characteristics).
- En función del uso de memoria de la base de referencia, elija [el nivel de servicio que se ajuste a la memoria](sql-database-managed-instance-resource-limits.md#hardware-generation-characteristics). La cantidad de memoria no se puede elegir directamente, por lo que tendría que seleccionar la Instancia administrada con la cantidad de núcleos virtuales que coincida con la memoria (por ejemplo, 5,1 GB/núcleo virtual en Gen5). 
- En función de la latencia de E/S de la base de referencia del subsistema de archivos, elija entre los niveles de servicio De uso general (latencia mayor que 5 ms) y Crítico para la empresa (latencia inferior a 3 ms).
- Según el rendimiento de la base de referencia, asigne previamente el tamaño de los archivos de datos o de registro para obtener el rendimiento de E/S esperado.

Puede seleccionar recursos de almacenamiento y proceso en el momento de la implementación y, posteriormente, cambiarlos sin provocar tiempo de inactividad de la aplicación con [Azure Portal](sql-database-scale-resources.md):

![ajuste de tamaño de la instancia administrada](./media/sql-database-managed-instance-migration/managed-instance-sizing.png)

Para más información sobre la creación de la infraestructura de red virtual y una instancia administrada, consulte [Creación de una Instancia administrada](sql-database-managed-instance-get-started.md).

> [!IMPORTANT]
> Es importante que la red virtual y la subred de destino cumplan siempre los [requisitos de red virtual de Instancia administrada de Azure SQL Database](sql-database-managed-instance-connectivity-architecture.md#network-requirements). Cualquier incompatibilidad puede impedir crear nuevas instancias o usar las que ya ha creado. Obtenga más información sobre la [creación de nuevas redes](sql-database-managed-instance-create-vnet-subnet.md) y la [configuración de las ya existentes](sql-database-managed-instance-configure-vnet-subnet.md).

## <a name="select-migration-method-and-migrate"></a>Selección del método de migración y migración

La opción de implementación de Instancia administrada está diseñada para escenarios de usuario que requieren migración masiva de bases de datos desde implementaciones locales o de base de datos de IaaS. Son la opción ideal si se necesita migrar mediante lift-and-shift las aplicaciones back-end que periódicamente usan funcionalidades del nivel de instancia o entre bases de datos. Si este es su caso, puede mover una instancia completa al entorno correspondiente en Azure sin necesidad de rediseñar sus aplicaciones.

Para mover las instancias de SQL, deberá planear cuidadosamente:

- La migración de todas las bases de datos que deben situarse juntas (que se ejecutan en la misma instancia)
- La migración de objetos de nivel de instancia de los que depende la aplicación, incluidos inicios de sesión, credenciales, trabajos y operadores del Agente SQL, y desencadenadores de nivel de servidor.

Instancia administrada es un servicio administrado que permite delegar algunas de las actividades DBA normales a la plataforma a medida que se crean. Por lo tanto, algunos datos de nivel de instancia no tienen que migrarse, por ejemplo, los trabajos de mantenimiento de copias de seguridad periódicas o la configuración Always On, porque integra [alta disponibilidad](sql-database-high-availability.md).

Instancia administrada admite las siguientes opciones de migración de base de datos (actualmente son los únicos métodos de migración admitidos):

- Azure Database Migration Service: migración prácticamente sin tiempo de inactividad.
- `RESTORE DATABASE FROM URL` nativo: usa copias de seguridad nativas de SQL Server y requiere tiempo de inactividad.

### <a name="azure-database-migration-service"></a>Azure Database Migration Service

[Azure Database Migration Service (DMS)](../dms/dms-overview.md) es un servicio totalmente administrado diseñado para permitir migraciones completas desde varios orígenes de base de datos hasta las plataformas de datos de Azure con un tiempo de inactividad mínimo. Este servicio simplifica las tareas necesarias para mover las bases de datos existentes de SQL Server y de terceros a Azure. En la versión preliminar pública, las opciones de implementación incluyen bases de datos de Azure SQL Database, bases de datos de SQL Server en una instancia de Azure Virtual Machine. DMS es el método recomendado de migración para las cargas de trabajo de la empresa.

Si usa SQL Server Integration Services (SSIS) en su servidor de SQL Server local, tenga en cuenta que el servicio DMS aún no admite la migración del catálogo de SSIS (SSISDB) que almacena paquetes SSIS; de todos modos, puede aprovisionar Azure-SSIS Integration Runtime (IR) en Azure Data Factory (ADF), para crear un nuevo SSISDB en una instancia administrada y, luego, puede volver a implementar sus paquetes en esta opción. Para ello, consulte [Creación de una instancia de Integration Runtime de SSIS de Azure en Azure Data Factory](https://docs.microsoft.com/azure/data-factory/create-azure-ssis-integration-runtime).

Para más información acerca de este escenario y los pasos de configuración para DMS, consulte cómo [migrar una base de datos local a Instancia administrada con DMS](../dms/tutorial-sql-server-to-managed-instance.md).  

### <a name="native-restore-from-url"></a>Restauración de copias de seguridad nativas desde la dirección URL

La restauración de copias de seguridad nativas (archivos .bak) realizadas desde instancias locales de SQL Server o [SQL Server en Virtual Machines](https://azure.microsoft.com/services/virtual-machines/sql-server/), disponible en [Azure Storage](https://azure.microsoft.com/services/storage/), es una de las principales funcionalidades clave de la opción de implementación de Instancia administrada que permite una migración rápida y sencilla de la base de datos sin conexión.

El siguiente diagrama proporciona una introducción general del proceso:

![flujo de migración](./media/sql-database-managed-instance-migration/migration-flow.png)

La siguiente tabla proporciona más información sobre los métodos que puede usar según la versión de SQL Server de origen que esté ejecutando:

|Paso|Motor y versión de SQL|Método de copia de seguridad y restauración|
|---|---|---|
|Colocar la copia de seguridad en Azure Storage|Anteriores a SQL 2012 SP1 CU2|Carga del archivo .bak directamente a Azure Storage|
||2012 SP1 CU2 - 2016|Copia de seguridad directa mediante la sintaxis [WITH CREDENTIAL](https://docs.microsoft.com/sql/t-sql/statements/restore-statements-transact-sql), ya en desuso.|
||2016 y posteriores|Copia de seguridad directa con [WITH SAS CREDENTIAL](https://docs.microsoft.com/sql/relational-databases/backup-restore/sql-server-backup-to-url).|
|Restaurar de Azure Storage a Instancia administrada|[RESTORE FROM URL WITH SAS CREDENTIAL](sql-database-managed-instance-get-started-restore.md)|

> [!IMPORTANT]
> - Al migrar una base de datos protegida mediante [Cifrado de datos transparente](transparent-data-encryption-azure-sql.md) a una instancia administrada con la opción de restauración nativa, se debe migrar el certificado correspondiente de SQL Server local o IaaS antes de restaurar la base de datos. Para consultar los pasos detallados, consulte [Migración de un certificado TDE a Instancia administrada](sql-database-managed-instance-migrate-tde-certificate.md)
> - No se permite restaurar bases de datos del sistema. Para migrar objetos de nivel de instancia (almacenados en bases de datos maestras o msdb), se recomienda generar scripts y ejecutar scripts de T-SQL en la instancia de destino.

Para obtener una guía rápida que incluya la manera de restaurar una copia de seguridad de base de datos en una instancia administrada mediante las credenciales SAS, consulte el artículo de [Restauración de una base de datos en una instancia administrada](sql-database-managed-instance-get-started-restore.md).

> [!VIDEO https://www.youtube.com/embed/RxWYojo_Y3Q]


## <a name="monitor-applications"></a>Supervisión de aplicaciones

Una vez haya completado la migración a Instancia administrada, debe realizar un seguimiento del comportamiento de la aplicación y del rendimiento de la carga de trabajo. Este proceso incluye las siguientes actividades:
- [Comparar el rendimiento de la carga de trabajo que se ejecuta en la instancia administrada](#compare-performance-with-the-baseline) con la [base de referencia de rendimiento que ha creado en la instancia de SQL Server de origen](#create-performance-baseline).
- [Supervisar el rendimiento de la carga de trabajo](#monitor-performance) continuamente para identificar posibles problemas y mejoras.

### <a name="compare-performance-with-the-baseline"></a>Comparación del rendimiento con la base de referencia

La primera actividad que necesita llevar a cabo inmediatamente después de realizar una migración correcta es comparar el rendimiento de la carga de trabajo con el de la carga de trabajo de la base de referencia. El objetivo de esta actividad es confirmar que el rendimiento de la carga de trabajo en la instancia administrada satisface sus necesidades. 

En la mayoría de los casos, la migración de base de datos a Instancia administrada mantiene la configuración de la base de datos y el nivel de compatibilidad original. La configuración original se conserva siempre que es posible para reducir el riesgo de algunas degradaciones del rendimiento en comparación con el de la instancia de SQL Server de origen. Si el nivel de compatibilidad de una base de datos de usuario era 100 o superior antes de la migración, permanece igual después de la migración. Si el nivel de compatibilidad de una base de datos de usuario era 90 antes de la migración, el nivel de compatibilidad de la base de datos actualizada se establece en 100, que es el nivel de compatibilidad mínimo admitido en Instancia administrada. El nivel de compatibilidad de las bases de datos del sistema es 140. Puesto que la migración a Instancia administrada consiste realmente en migrar a la última versión del motor de base de datos de SQL Server, debe tener en cuenta que deberá volver a probar el rendimiento de la carga de trabajo para evitar algunos problemas de rendimiento inesperados.

Como requisito previo, asegúrese de que ha completado las siguientes actividades:
- Alinear la configuración de Instancia administrada con la configuración de la instancia de SQL Server de origen investigando diversos valores de configuración de la instancia, la base de datos y tempdb. Asegúrese de que no ha cambiado valores como los de los niveles de compatibilidad o el cifrado antes de ejecutar la primera comparación de rendimiento, o acepte el riesgo de que algunas de las nuevas características que ha habilitado afecten a algunas consultas. Para reducir los riesgos de la migración, cambie el nivel de compatibilidad de la base de datos solo después de supervisar el rendimiento.
- Implementar [directrices de procedimientos recomendados de almacenamiento de uso general](https://techcommunity.microsoft.com/t5/DataCAT/Storage-performance-best-practices-and-considerations-for-Azure/ba-p/305525) como preasignar el tamaño de los archivos para obtener el mejor rendimiento.
- Obtener más información sobre las [principales diferencias del entorno que podrían producir diferencias de rendimiento entre Instancia administrada y SQL Server]( https://azure.microsoft.com/blog/key-causes-of-performance-differences-between-sql-managed-instance-and-sql-server/) e identificar los riesgos que podrían afectar al rendimiento.
- Asegurarse de que tiene habilitado Almacén de consultas y Ajuste automático en la instancia administrada. Estas características le permiten medir el rendimiento de carga de trabajo y corregir automáticamente los posibles problemas de rendimiento. Aprenda a usar Almacén de consultas como herramienta ideal para obtener información sobre el rendimiento de la carga de trabajo antes y después de cambiar el nivel de compatibilidad de la base de datos, tal y como se explica en [Mantener la estabilidad del rendimiento al actualizar a una versión más reciente de SQL Server](https://docs.microsoft.com/sql/relational-databases/performance/query-store-usage-scenarios#CEUpgrade).
Una vez que haya preparado el entorno de forma que sea lo más parecido posible al entorno local, puede empezar a ejecutar la carga de trabajo y a medir el rendimiento. El proceso de medición debe incluir los mismos parámetros que midió [al crear la base de referencia de rendimiento de la carga de trabajo en la instancia de origen de SQL Server](#create-performance-baseline).
Como resultado, debe comparar los parámetros de rendimiento con la base de referencia e identificar las diferencias más importantes.

> [!NOTE]
> En muchos casos, no podrá obtener un rendimiento que coincida exactamente en Instancia administrada y en SQL Server. Instancia administrada es un motor de base de datos de SQL Server pero la infraestructura y la configuración de alta disponibilidad de ella pueden suponer alguna diferencia. Posiblemente algunas consultas serán más rápidas mientras que otras pueden ser más lentas. El objetivo de la comparación es comprobar que el rendimiento de la carga de trabajo en Instancia administrada coincide con el rendimiento en SQL Server (el promedio) e identificar si hay alguna consulta crítica con un rendimiento que no coincida con el rendimiento original.

El resultado de la comparación de rendimiento puede ser:
- El rendimiento de la carga de trabajo en Instancia administrada está alineado o es mejor que el rendimiento en SQL Server. En este caso ha confirmado satisfactoriamente que la migración ha sido correcta.
- La mayoría de los parámetros de rendimiento y las consultas de la carga de trabajo funcionan bien, con algunas excepciones que tienen un rendimiento reducido. En este caso, deberá identificar las diferencias y su importancia. Si hay algunas consultas importantes con rendimiento reducido, debería investigar si hay planes subyacentes de SQL que hayan cambiado o si las consultas están traspasando algunos de los límites de los recursos. La mitigación en este caso consistiría en aplicar algunas sugerencias a las consultas críticas (por ejemplo, un nuevo nivel de compatibilidad o estimador de cardinalidad heredado) ya sea directamente o mediante guías de planes, y crear o recrear estadísticas e índices que pudieran afectar a los planes. 
- La mayoría de las consultas son más lentas en Instancia administrada en comparación con las de la instancia de SQL Server de origen. En este caso, intente identificar las causas principales de la diferencia como, por ejemplo, [haber alcanzado el límite de algunos recursos]( sql-database-managed-instance-resource-limits.md#instance-level-resource-limits) como los límites de E/S, el límite de memoria, el límite de tasa de registro de instancias, etc. Si no hay ningún límite de recursos que pueda provocar la diferencia, pruebe a cambiar el nivel de compatibilidad de la base de datos o algún valor de configuración de esta como la estimación de cardinalidad heredada y vuelva a iniciar la prueba. Revise las recomendaciones proporcionadas por las vistas de Instancia administrada o Almacén de consultas para identificar las consultas en las que se redujo el rendimiento.

> [!IMPORTANT]
> Instancia administrada tiene una característica de corrección automática de planes integrada que está habilitada de forma predeterminada. Esta característica garantiza que las consultas que funcionaban bien en el pasado no se degradarán en el futuro. Asegúrese de que esta característica está habilitada y que ha ejecutado la carga de trabajo durante el tiempo suficiente con la configuración anterior antes de cambiar a la nueva configuración para permitir que Instancia administrada obtenga más información sobre la base de referencia de rendimiento y los planes.

Cambie los parámetros o actualice los niveles de servicio para que converjan en la configuración óptima hasta que consiga que el rendimiento de la carga de trabajo se ajuste a sus necesidades.

### <a name="monitor-performance"></a>Supervisión del rendimiento

Instancia administrada proporciona una gran cantidad de herramientas avanzadas para la supervisión y solución de problemas, y debe utilizarlas para supervisar el rendimiento de la instancia. Algunos de los parámetros que tendría que supervisar son:
- El uso de CPU en la instancia para determinar que el número de núcleos virtuales que ha aprovisionado es la opción adecuada para la carga de trabajo.
- La duración prevista de la página en la Instancia administrada para determinar si [necesita memoria adicional](https://techcommunity.microsoft.com/t5/Azure-SQL-Database/Do-you-need-more-memory-on-Azure-SQL-Managed-Instance/ba-p/563444).
- Espere estadísticas como `INSTANCE_LOG_GOVERNOR` o `PAGEIOLATCH`, que le indicarán que tiene problemas de E/S de almacenamiento, especialmente en el nivel De uso general en el que podría tener que asignar previamente archivos para obtener un mejor rendimiento de E/S.

## <a name="leverage-advanced-paas-features"></a>Aprovechamiento de las características avanzadas de PaaS

Una vez que se encuentra en una plataforma totalmente administrada y ha comprobado que el rendimiento de la carga de trabajo coincide con el rendimiento de la carga de trabajo de SQL Server, aproveche las ventajas que se proporcionan automáticamente como parte del servicio SQL Database. 

Incluso si no ha realizado cambios en la instancia administrada durante la migración, es muy probable que haya activado algunas características nuevas mientras trabaja con la instancia para sacar partido a algunas de las mejoras más recientes del motor de base de datos. Algunos cambios se habilitan solo después de [cambiar el nivel de compatibilidad de la base de datos](https://docs.microsoft.com/sql/relational-databases/databases/view-or-change-the-compatibility-level-of-a-database).


Por ejemplo, no tiene que crear copias de seguridad en Instancia administrada; el servicio realiza las copias de seguridad automáticamente. Ya no debe preocuparse de programar, seguir y administrar las copias de seguridad. Instancia administrada permite restaurar a un momento dado dentro de este período de retención mediante la [recuperación a un momento dado (PITR)](sql-database-recovery-using-backups.md#point-in-time-restore). Además, no es necesario preocuparse por la configuración de la alta disponibilidad porque la [alta disponibilidad](sql-database-high-availability.md) está integrada.

Para reforzar la seguridad, considere la posibilidad de usar la [autenticación de Azure Active Directory](sql-database-security-overview.md), la [auditoría](sql-database-managed-instance-auditing.md), la [detección de amenazas](sql-database-advanced-data-security.md), la [seguridad de nivel de fila](https://docs.microsoft.com/sql/relational-databases/security/row-level-security) y el [enmascaramiento dinámico de datos](https://docs.microsoft.com/sql/relational-databases/security/dynamic-data-masking)).

Además de la administración avanzada y las características de seguridad, Instancia administrada ofrece un conjunto de herramientas avanzadas que pueden ayudarle a [supervisar y optimizar la carga de trabajo](sql-database-monitor-tune-overview.md). [Azure SQL Analytics](https://docs.microsoft.com/azure/azure-monitor/insights/azure-sql) le permite supervisar un conjunto grande de instancias administradas y centralizar la supervisión de un gran número de instancias y bases de datos. [Ajuste automático](https://docs.microsoft.com/sql/relational-databases/automatic-tuning/automatic-tuning#automatic-plan-correction) de Instancia administrada supervisa continuamente el rendimiento de las estadísticas de ejecución del plan de SQL y corrige automáticamente los problemas de rendimiento identificados.

## <a name="next-steps"></a>Pasos siguientes

- Para más información acerca de las instancias administradas, consulte [¿Qué es una instancia administrada?](sql-database-managed-instance.md)
- Para ver un tutorial que incluye una restauración desde una copia de seguridad, consulte [Creación de una instancia administrada](sql-database-managed-instance-get-started.md).
- Para ver un tutorial que muestra la migración mediante DMS, consulte cómo [migrar una base de datos local a Instancia administrada con DMS](../dms/tutorial-sql-server-to-managed-instance.md).  
