---
title: 'Ejemplo de script de Azure PowerShell: creación de una definición de aplicación administrada | Microsoft Docs'
description: 'Ejemplo de script de Azure PowerShell: creación de una definición de aplicación administrada'
services: managed-applications
documentationcenter: na
author: tfitzmac
manager: timlt
ms.service: managed-applications
ms.devlang: powershell
ms.topic: sample
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/27/2017
ms.author: tomfitz
ms.openlocfilehash: 35f1b805323da03f1622e5c355d6ffa6d960b275
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/23/2019
ms.locfileid: "66171501"
---
# <a name="create-a-managed-application-definition-with-powershell"></a>Creación de una definición de aplicación administrada con PowerShell

Este script publica una definición de aplicación administrada en un catálogo de servicios.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Script de ejemplo

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[!code-powershell[main](../../../powershell_scripts/managed-applications/create-definition/create-definition.ps1 "Create definition")]


## <a name="script-explanation"></a>Explicación del script

Este script usa el siguiente comando para crear la definición de aplicación administrada. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [New-AzManagedApplicationDefinition](https://docs.microsoft.com/powershell/module/az.resources/new-azmanagedapplicationdefinition) | Cree una definición de aplicación administrada. Proporcione el paquete que contiene los archivos necesarios. |


## <a name="next-steps"></a>Pasos siguientes

* Para una introducción a las aplicaciones administradas, consulte la [introducción a las aplicaciones administradas de Azure](../overview.md).
* Para más información sobre PowerShell, consulte la [documentación de Azure PowerShell](https://docs.microsoft.com/powershell/azure/get-started-azureps).
