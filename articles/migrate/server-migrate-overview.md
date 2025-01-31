---
title: Selección de una opción de migración de VMware con Azure Migrate Server Migration | Microsoft Docs
description: Proporciona información general sobre las opciones para migrar máquinas virtuales de VMware a Azure con Azure Migrate Server Migration.
author: rayne-wiselman
ms.service: azure-migrate
ms.topic: conceptual
ms.date: 07/09/2019
ms.author: raynew
ms.openlocfilehash: f27982b4e310d9865e497a3e1e10be9948beb928
ms.sourcegitcommit: 3877b77e7daae26a5b367a5097b19934eb136350
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2019
ms.locfileid: "68640733"
---
# <a name="select-a-vmware-migration-option"></a>Selección de una opción de migración de VMware

Puede migrar las máquinas virtuales de VMware a Azure mediante la herramienta Azure Migrate Server Migration. Esta herramienta ofrece un par de opciones para la migración de máquinas virtuales de VMware:

- Migración mediante la replicación sin agente. Las máquinas virtuales se migran sin necesidad de instalar nada en ellas.
- Migración con un agente de replicación. Instale un agente en la máquina virtual para su replicación.




## <a name="compare-migration-methods"></a>Comparación de métodos de migración

Use estas comparaciones seleccionadas para que le ayuden a decidir qué método usar. También puede revisar los requisitos de compatibilidad completos para la migración [sin agente](migrate-support-matrix-vmware.md#agentless-migration-vmware-server-requirements) y [basada en agente](migrate-support-matrix-vmware.md#agent-based-migration-vmware-server-requirements).

**Configuración** | **Sin agente** | **Basado en agente**
--- | --- | ---
**Permisos de Azure** | Necesita permisos para crear un proyecto de Azure Migrate y para registrar aplicaciones de Azure AD creadas al implementar el dispositivo de Azure Migrate. | Necesita permisos de colaborador en la suscripción de Azure. 
**Replicación simultánea** | Se puede replicar un máximo de 100 VM simultáneamente desde una instancia de vCenter Server.<br/> Si tiene más de 50 máquinas virtuales para la migración, cree varios lotes de máquinas.<br/> Si replica más de cada vez, afectará al rendimiento. | N/D
**Implementación del dispositivo** | El [dispositivo de Azure Migrate](migrate-appliance.md) se implementa de forma local. | El [dispositivo de replicación de Azure Migrate](migrate-replication-appliance.md) se implementa de forma local.
**Compatibilidad con Site Recovery** | Compatible. | No se puede replicar con la migración del servidor de Azure Migrate si ha configurado la replicación para una máquina con Site Recovery.
**Disco de destino** | Discos administrados | Discos administrados
**Límites del disco** | Disco del sistema operativo: 2 TB<br/><br/> Disco de datos: 4 TB<br/><br/> Número máximo de discos: 60 | Disco del sistema operativo: 2 TB<br/><br/> Disco de datos: 4 TB<br/><br/> Número máximo de discos: 63
**Discos de acceso directo** | No compatible | Compatible
**Arranque UEFI** | No compatible | La VM migrada en Azure se convertirá automáticamente en una VM de arranque del BIOS.<br/><br/> El disco del sistema operativo debe tener hasta cuatro particiones y los volúmenes deben formatearse con NTFS.


## <a name="deployment-steps-comparison"></a>Comparación de los pasos de implementación

Después de revisar las limitaciones, una descripción de los pasos necesarios para implementar cada solución puede ayudarle a decidir qué opción elegir.

**Task** | **Detalles** |**Sin agente** | **Basado en agente**
--- | --- | --- | ---
**Preparación de máquinas virtuales y servidor de VMware para la migración** | Configure diversas opciones en máquinas virtuales y servidores de VMware. | Obligatorio | Obligatorio
**Incorporación de la herramienta Server Migration** | Agregue la herramienta Azure Migrate Server Migration en el proyecto de Azure Migrate. | Obligatorio | Obligatorio
**Implementación del dispositivo de Azure Migrate** | Configure un dispositivo ligero en una máquina virtual de VMware para la detección y la valoración de máquinas virtuales. | Obligatorio | No se requiere.
**Instalación de Mobility Service en máquinas virtuales** | Instale Mobility Service en cada máquina que quiera replicar. | No se requiere | Obligatorio
**Implementación del dispositivo de replicación de Azure Migrate Server Migration** | Configure un dispositivo en una máquina virtual de VMware para detectar máquinas virtuales y para que actúe como puente entre la instancia de Mobility Service que se ejecuta en las máquinas virtuales y Azure Migrate Server Migration. | No se requiere | Obligatorio
**Replicación de máquinas virtuales**, habilitación de la replicación de máquinas virtuales | Configure las opciones de replicación y seleccione las máquinas virtuales que se van a replicar. | Obligatorio | Obligatorio
**Ejecución de una migración de prueba** | Ejecute una migración de prueba para asegurarse de que todo funciona de la forma esperada. | Obligatorio | Obligatorio
**Ejecución de una migración completa** | Migre las máquinas virtuales. | Obligatorio | Obligatorio




## <a name="next-steps"></a>Pasos siguientes

[Migración de VM de VMware](tutorial-migrate-vmware.md) con migración sin agentes.



