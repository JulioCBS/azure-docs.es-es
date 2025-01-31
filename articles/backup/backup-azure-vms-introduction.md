---
title: Acerca de la copia de seguridad de máquina virtual de Azure
description: Obtenga información sobre la copia de seguridad de máquina virtual de Azure y tenga en cuenta algunos procedimientos recomendados.
author: dcurwin
manager: carmonm
ms.service: backup
ms.topic: conceptual
ms.date: 09/13/2019
ms.author: dacurwin
ms.openlocfilehash: db3e4b8a8abea4718f5779790906bf45591d221c
ms.sourcegitcommit: 71db032bd5680c9287a7867b923bf6471ba8f6be
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/16/2019
ms.locfileid: "71018701"
---
# <a name="an-overview-of-azure-vm-backup"></a>Información general sobre la copia de seguridad de máquinas virtuales de Azure

En este artículo se describe cómo el [servicio Azure Backup](backup-introduction-to-azure-backup.md) realiza una copia de seguridad de las máquinas virtuales (VM) de Azure.

## <a name="backup-process"></a>Proceso de copia de seguridad

Aquí se explica cómo Azure Backup completa una copia de seguridad para las VM de Azure:

1. Para VM de Azure que están seleccionadas para copia de seguridad, Azure Backup inicia un trabajo de copia de seguridad según la programación que especifique el usuario.
1. Durante la primera copia de seguridad, se instala una extensión de copia de seguridad en la VM si esta se encuentra en ejecución.
    - En VM de Windows, se instala la extensión _VMSnapshot_.
    - En VM de Linux, se instala la extensión _VMSnapshotLinux_.
1. En el caso de VM Windows en ejecución, Azure Backup coordina con el Servicio de instantáneas de volumen (VSS) de Windows para obtener una instantánea coherente con la aplicación de la VM.
    - De forma predeterminada, Azure Backup realiza copias de seguridad de VSS completas.
    - Si Azure Backup no puede realizar una instantánea coherente con la aplicación, realiza una instantánea coherente con archivos del almacenamiento subyacente (ya que no se produce ninguna escritura en la aplicación mientras la VM está detenida).
1. En VM Linux, Azure Backup hace una copia de seguridad coherente con archivos. Para obtener instantáneas coherentes con la aplicación, debe personalizar manualmente los scripts previos y posteriores.
1. Después de que Azure Backup toma la instantánea, transfiere los datos al almacén.
    - La copia de seguridad se optimiza al hacer una copia de seguridad de cada disco de VM en paralelo.
    - Para cada disco del que se hace una copia de seguridad, Azure Backup lee los bloques en el disco e identifica y transfiere solo los bloques de datos que han cambiado (delta) desde la copia de seguridad anterior.
    - Es posible que los datos de las instantáneas no se copien inmediatamente en el almacén. Podrían tardar horas en momentos de máxima actividad. El tiempo total de copia de seguridad de una VM será inferior a 24 horas para las directivas de copia de seguridad diarias.
 1. Los cambios que se hagan en una VM de Windows después de habilitar Azure Backup en ella son los siguientes:
    -   Microsoft Visual C++ 2013 Redistributable(x64): 12.0.40660 instalado en la VM
    -   Tipo de inicio del Servicio de instantáneas de volumen (VSS) cambiado de manual a automático
    -   Se agrega el servicio de Windows IaaSVmProvider

1. Cuando se complete la transferencia de datos, la instantánea se quita y se crea un punto de recuperación.

![Arquitectura de copia de seguridad de máquinas virtuales de Azure](./media/backup-azure-vms-introduction/vmbackup-architecture.png)

## <a name="encryption-of-azure-vm-backups"></a>Cifrado de copias de seguridad de VM de Azure

Al realizar la copia de seguridad de máquinas virtuales de Azure con Azure Backup, las máquinas virtuales se cifran en reposo con Storage Service Encryption (SSE). Azure Backup también puede hacer una copia de seguridad de las VM de Azure cifradas mediante Azure Disk Encryption (ADE).

**Cifrado** | **Detalles** | **Soporte técnico**
--- | --- | ---
**Azure Disk Encryption** | Azure Disk Encryption cifra los discos de datos y de sistema operativo para VM de Azure.<br/><br/> Azure Disk Encryption se integra con las claves de cifrado de BitLocker (BEK), que se protegen en un almacén de claves como secretos. Azure Disk Encryption también se integra con las claves de cifrado de las claves de Azure Key Vault (KEK). | Azure Backup admite la copia de seguridad de VM de Azure administradas y no administradas cifradas con solo BEK solo o con BEK junto con KEK.<br/><br/> Se crea una copia de seguridad tanto de las BEK como las KEK y estas se cifran.<br/><br/> Dado que se crea una copia de seguridad de las KEK y BEK, los usuarios con los permisos necesarios pueden restaurar las claves y los secretos en el almacén de claves, si fuera necesario. Estos usuarios también pueden recuperar la VM cifrada.<br/><br/> Los usuarios no autorizados o Azure no pueden leer las claves y los secretos cifrados.
**SSE** | Con SSE, Azure Storage proporciona cifrado en reposo al cifrar automáticamente los datos antes de almacenarlos. Azure Storage también descifra los datos antes de recuperarlos. | Azure Backup usa SSE para el cifrado en reposo de las VM de Azure.

Para VM de Azure administradas y no administradas, Azure Backup admite VM cifradas con solo BEK o VM cifradas con BEK y KEK.

Las copias de seguridad de las BEK (secretos) y KEK (claves) están cifradas. Se pueden leer y usar solo cuando los usuarios autorizados las vuelven a restaurar en el almacén de claves. Ni los usuarios no autorizados ni Azure pueden leer o usar las claves o los secretos con copia de seguridad.

También se crea una copia de seguridad de las BEK. Por lo tanto, si se pierden las BEK, los usuarios autorizados pueden restaurarlas en el almacén de claves y recuperar las VM cifradas. Solo los usuarios con el nivel adecuado de permisos pueden realizar la copia de seguridad y restauración de las VM cifradas, así como las claves y los secretos.

## <a name="snapshot-creation"></a>Creación de instantáneas

Azure Backup toma instantáneas según la programación de copia de seguridad.

- **VM de Windows**: para las VM de Windows, el servicio de copia de seguridad coordina con VSS para tomar una instantánea coherente con la aplicación de los discos de la VM.

  - De forma predeterminada, Azure Backup realiza copias de seguridad de VSS completas. [Más información](https://blogs.technet.com/b/filecab/archive/2008/05/21/what-is-the-difference-between-vss-full-backup-and-vss-copy-backup-in-windows-server-2008.aspx).
  - Para cambiar la configuración de modo que Azure Backup realice copias de seguridad de las copias de VSS, establezca la siguiente clave del registro desde un símbolo del sistema:

    **REG ADD "HKLM\SOFTWARE\Microsoft\BcdrAgent" /v USEVSSCOPYBACKUP /t REG_SZ /d TRUE /f**

- **VM de Linux:** para realizar instantáneas coherentes con la aplicación de una VM de Linux, use el marco de script anterior y posterior para escribir scripts personalizados a fin de garantizar la coherencia.

  - Azure Backup solo invoca los scripts anteriores y posteriores que escriba el usuario.
  - Azure Backup marcará el punto de recuperación como coherente con la aplicación si la ejecución de los scripts anteriores y posteriores se realiza correctamente. Sin embargo, el usuario es el responsable final de la coherencia con la aplicación cuando se usan scripts personalizados.
  - [Obtenga más información](backup-azure-linux-app-consistent.md) sobre cómo configurar scripts.

### <a name="snapshot-consistency"></a>Coherencia de instantáneas

En la siguiente tabla se explica los diferentes tipos de coherencia de instantánea:

**Instantánea** | **Detalles** | **Recuperación** | **Consideración**
--- | --- | --- | ---
**Coherente con la aplicación** | Las copias de seguridad coherentes con las aplicaciones capturan el contenido de la memoria y las operaciones de E/S pendientes. Las instantáneas coherentes con la aplicación usan el escritor VSS (o scripts anteriores o posteriores para Linux) para garantizar la coherencia de datos de la aplicación antes de que se produzca una copia de seguridad. | Cuando se va a recuperar una VM con una instantánea coherente con la aplicación, la VM se inicia. No se pierden ni se dañan los datos. Las aplicaciones se inician en un estado coherente. | Windows: Todas las instancias de VSS Writer son correctas<br/><br/> Linux: Los scripts anteriores o posteriores están configurados y son correctos
**Coherencia con el sistema de archivos** | Las copias de seguridad coherentes con el sistema de archivos proporcionan coherencia al hacer una instantánea de todos los archivos al mismo tiempo.<br/><br/> | Cuando se va a recuperar una VM con una instantánea coherente con el sistema de archivos, la VM se inicia. No se pierden ni se dañan los datos. Las aplicaciones deben implementar su propio mecanismo de corrección para asegurarse de que los datos restaurados son coherentes. | Windows: Se produjeron errores en algunas instancias de VSS Writer <br/><br/> Linux: Valor predeterminado (si los scripts anteriores y posteriores no están configurados o tienen errores)
**Coherente frente a bloqueos** | Las instantáneas coherentes con bloqueos suelen producirse si una VM de Azure se apaga en el momento en que se hace la copia de seguridad. Solamente se capturan y se hace una copia de seguridad de los datos que ya existen en el disco en el momento de la copia de seguridad.<br/><br/> Un punto de recuperación coherente con el bloqueo no garantiza la coherencia de datos para el sistema operativo o la aplicación. | Aunque no hay ninguna garantía, la VM suele arrancar y luego iniciar una comprobación de disco para corregir los errores por daños. Los datos en memoria o las operaciones de escritura que no se han transferido al disco antes del bloqueo se pierden. Las aplicaciones implementan su propia comprobación de los datos. Por ejemplo, una aplicación de base de datos puede usar su registro de transacciones para la comprobación. Si el registro de transacciones tiene entradas que no están presentes en la base de datos, el software de base de datos realiza una reversión de las transacciones hasta que los datos sean coherentes. | La VM está en estado de cierre

## <a name="backup-and-restore-considerations"></a>Consideraciones sobre las operaciones de copia de seguridad y restauración

**Consideración** | **Detalles**
--- | ---
**Disco** | La copia de seguridad de los disco de VM se realiza en paralelo. Por ejemplo, si una VM tiene cuatro discos, el servicio intenta hacer una copia de seguridad de los cuatro en paralelo. La copia de seguridad es incremental (solo los datos modificados).
**Programación** |  Para reducir el tráfico de copia de seguridad, haga una copia de seguridad diferentes VM en distintos momentos del día y asegúrese de que los horarios no se superpongan. La copia de seguridad de máquinas virtuales al mismo tiempo causa atascos del tráfico.
**Preparación de copias de seguridad** | Tenga en cuenta el tiempo necesario para preparar la copia de seguridad. El tiempo de preparación incluye la instalación o actualización de la extensión de copia de seguridad y el desencadenamiento de una instantánea según la programación de copia de seguridad.
**Transferencia de datos** | Tenga en cuenta el tiempo necesario para que Azure Backup identifique los cambios incrementales desde la copia de seguridad anterior.<br/><br/> En una copia de seguridad incremental, Azure Backup determina los cambios al calcular la suma de comprobación del bloque. Si un bloque cambia, se marca para su transferencia al almacén. El servicio analiza los bloques identificados para intentar aún más minimizar la cantidad de datos que se van a transferir. Después de evaluar todos los bloques cambiados, Azure Backup transfiere los cambios al almacén.<br/><br/> Puede haber un retraso entre la realización de la instantánea y la copia en el almacén.<br/><br/> En horas punta, el procesamiento de las copias de seguridad puede tardar hasta ocho horas. El tipo de copia de seguridad de una máquina virtual será inferior a 24 horas para las copias de seguridad diarias.
**Copia de seguridad inicial** | Aunque el tiempo total de copia de seguridad para copias de seguridad incrementales es menor que 24 horas, es posible que esto no sea el caso para la primera copia de seguridad. El tiempo necesario para la copia de seguridad inicial dependerá del tamaño de los datos y el momento de procesamiento de la copia de seguridad.
**Restaurar cola** | Azure Backup procesa los trabajos de restauración desde varias cuentas de almacenamiento al mismo tiempo y coloca las solicitudes de restauración en una cola.
**Restaurar copia** | Durante el proceso de restauración, los datos se copian desde el almacén en la cuenta de almacenamiento.<br/><br/> El tiempo total de restauración depende de las operaciones de E/S por segundo (IOPS) y el rendimiento de la cuenta de almacenamiento.<br/><br/> Para reducir el tiempo de copia, seleccione una cuenta de almacenamiento que no esté cargada con otras lecturas y escrituras de aplicaciones.

### <a name="backup-performance"></a>Rendimiento de Backup

Estos escenarios comunes pueden afectar al tiempo total de copia de seguridad:

- **Agregar un disco nuevo a una VM de Azure protegida:** si una VM es sujeto de copia de seguridad incremental y se agrega un disco nuevo, aumentará el tiempo de procesamiento de la copia de seguridad. El tiempo total de procesamiento de la copia de seguridad puede ser más de 24 horas debido a la replicación inicial del nuevo disco, junto con la replicación delta de los discos existentes.
- **Discos fragmentados:** las operaciones de copia de seguridad son más rápidas los cambios en el disco son contiguos. Si los cambios se expanden horizontalmente y se fragmentan a lo largo de un disco, la copia de seguridad será más lenta.
- **Actividad de disco:** si los discos protegidos que están en proceso de copia de seguridad incremental tienen una actividad diaria de más de 200 GB, la copia de seguridad puede tardar mucho tiempo (más de ocho horas) en completarse.
- **Versiones de Azure Backup:** la versión más reciente de Azure Backup (conocida como la versión de restauración instantánea) usa un proceso más optimizado que la comparación de la suma de comprobación para identificar los cambios. Sin embargo, si usa la restauración instantánea y ha eliminado una instantánea de copia de seguridad, la copia de seguridad cambia a la comparación de la suma de comprobación. En este caso, la operación de copia de seguridad será superior a 24 horas (o un sufrirá un error).

## <a name="best-practices"></a>Procedimientos recomendados

Al configurar las copias de seguridad de VM, se recomienda seguir estos procedimientos recomendados:

- Modificar las horas de programación predeterminadas que se establecen en una directiva. Por ejemplo, si la hora predeterminada en la directiva es 00:00 h, incremente el tiempo en varios minutos para que los recursos se usen de forma óptima.
- Si va a restaurar las VM desde un solo almacén, recomendamos encarecidamente que use diferentes [cuentas de almacenamiento de uso general v2](https://docs.microsoft.com/azure/storage/common/storage-account-upgrade) para asegurarse de que no se vea limitada la cuenta de almacenamiento de destino. Por ejemplo, cada VM debe tener una cuenta de almacenamiento diferente. Por ejemplo, si se restauran 10 máquinas virtuales, use 10 cuentas de almacenamiento diferentes.
- Para la copia de seguridad de VM que usan almacenamiento Premium, con restauración instantánea, se recomienda asignar un espacio libre del *50 %* , del espacio de almacenamiento total asignado, que **solo** es necesario para la primera copia de seguridad. El espacio libre del 50 % no es un requisito para las copias de seguridad una vez completada la primera copia de seguridad.
- Las restauraciones de una capa de almacenamiento de uso general v1 (instantánea) se completará en cuestión de minutos, porque la instantánea está en la misma cuenta de almacenamiento. Las restauraciones de la capa de almacenamiento de uso general v2 (almacén) pueden tardar horas. En los casos en los que los datos están disponibles en el almacenamiento de uso general v1, recomendamos que use la característica de [restauración instantánea](backup-instant-restore-capability.md) para lograr restauraciones más rápidas. (Si los datos deben restaurarse desde un almacén, se tardará más tiempo).
- El límite del número de discos por cuenta de almacenamiento es relativo a la frecuencia de acceso a los discos de parte de las aplicaciones que se ejecutan en una VM de infraestructura como servicio (IaaS). Como práctica general, si hay entre 5 y 10 o más en una única cuenta de almacenamiento, mueva algunos discos a cuentas de almacenamiento separadas para equilibrar la carga.

## <a name="backup-costs"></a>Costos de la copia de seguridad

Las máquinas virtuales de Azure incluidas en la copia de seguridad de Azure Backup están sujetas a los [precios de Azure Backup](https://azure.microsoft.com/pricing/details/backup/).

La facturación no se inicia hasta que no se complete la primera copia de seguridad correcta. En este punto, se iniciará la facturación del almacenamiento y las máquinas virtuales protegidas. La facturación continúa siempre y cuando los datos de copia de seguridad de la VM se almacenan en un almacén. Si detiene la protección de una máquina virtual, pero existen datos de copia de seguridad para la máquina virtual en un almacén, la facturación continúa.

La facturación de una máquina virtual especificada solo se suspenderá si se detiene la protección y se eliminan los datos de la copia de seguridad. Cuando no hay ningún trabajo de copia de seguridad activo y se ha detenido la protección, el tamaño de la última copia de seguridad correcta de la máquina virtual se convierte en el tamaño de instancia protegida en el que se basa la factura mensual.

El cálculo del tamaño de instancia protegida se basa en el tamaño *real* de la VM. El tamaño de la VM es la suma de todos los datos que esta contiene, excepto el almacenamiento temporal. Los precios se basan en los datos reales que se almacenan en los discos de datos, no en el tamaño máximo admitido para cada disco de datos conectado a la VM.

De forma similar, la factura de almacenamiento de copia de seguridad se basa en la cantidad de datos almacenados en Azure Backup, que es la suma de los datos reales de cada punto de recuperación.

Por ejemplo, veamos una VM de tamaño estándar A2 con dos discos de datos adicionales con un tamaño máximo de 4 TB cada uno. En la tabla siguiente se proporcionan los datos almacenados en cada uno de estos discos:

**Disco** | **Tamaño máximo** | **Datos reales presentes**
--- | --- | ---
Disco del sistema operativo | 4095 GB | 17 GB
Disco local/temporal | 135 GB | 5 GB (no incluidos en la copia de seguridad)
Disco de datos 1 | 4095 GB | 30 GB
Disco de datos 2 | 4095 GB | 0 GB

El tamaño real de la máquina virtual en este caso es de 17 GB + 30 GB + 0 GB = 47 GB. Este tamaño de instancias protegidas (47 GB) se convierte en la base de la factura mensual. A medida que crece la cantidad de datos en la VM, el tamaño de instancia protegida usado para la facturación también cambia para que coincida.

<a name="limited-public-preview-backup-of-vm-with-disk-sizes-up-to-30tb"></a>
## <a name="public-preview-backup-of-vm-with-disk-sizes-up-to-30-tb"></a>Versión preliminar pública: Copia de seguridad de máquinas virtuales con tamaños de disco de hasta 30 TB

Azure Backup admite ya una versión preliminar pública de instancias de [Azure Managed Disks](https://azure.microsoft.com/blog/larger-more-powerful-managed-disks-for-azure-virtual-machines/) más grandes y potentes de hasta 30 TB de tamaño. Esta versión preliminar proporciona compatibilidad en el nivel de producción con las máquinas virtuales administradas.

Las copias de seguridad de las máquinas virtuales con un tamaño de disco de hasta 30 TB y un máximo de 256 TB combinado para todos los discos de una máquina virtual deben funcionar fluidamente sin afectar las copias de seguridad existentes. No se requiere ninguna acción del usuario para ejecutar las copias de seguridad de los discos de gran tamaño si la máquina virtual ya está configurada con Azure Backup.

Las copias de seguridad de todas las instancias de Máquinas virtuales de Azure con discos grandes que tengan una copia de seguridad configurada se deben realizar correctamente.

## <a name="next-steps"></a>Pasos siguientes

Ahora, [prepárese para la copia de seguridad de VM de Azure](backup-azure-arm-vms-prepare.md).
