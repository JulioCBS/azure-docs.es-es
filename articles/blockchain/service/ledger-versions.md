---
title: Versiones que Azure Blockchain Service admite para los libros de contabilidad, la revisión y la actualización
description: Información general de las versiones admitidas en Azure Blockchain Service para los libros de contabilidad, incluidas las directivas con respecto a los sistemas de aplicación de revisiones y las actualizaciones administradas por el usuario y por el sistema.
services: azure-blockchain
keywords: cadena de bloques
author: PatAltimore
ms.author: patricka
ms.date: 05/02/2019
ms.topic: article
ms.service: azure-blockchain
ms.reviewer: janders
manager: femila
ms.openlocfilehash: 6cded49f14ae90f84b18c0a37059677c6c1fed99
ms.sourcegitcommit: 39d95a11d5937364ca0b01d8ba099752c4128827
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/16/2019
ms.locfileid: "69562793"
---
# <a name="supported-azure-blockchain-service-ledger-versions"></a>Versiones que Azure Blockchain Service admite para los libros de contabilidad

Azure Blockchain Service usa el libro de contabilidad de [Quorum](https://www.goquorum.com/developers) basado en Ethereum y diseñado para el procesamiento de transacciones privadas dentro de un grupo de participantes conocidos, identificado como un consorcio de Azure Blockchain Service.

Actualmente, Azure Blockchain Service admite [Quorum versión 2.2.3](https://github.com/jpmorganchase/quorum/releases/tag/v2.2.3) y el [administrador de transacciones Tessera](https://github.com/jpmorganchase/tessera).

## <a name="managing-updates-and-upgrades"></a>Administración de actualizaciones

El control de versiones en Quorum se realiza con las versiones principal, secundaria y las revisadas. Por ejemplo, si la versión de Quorum es la 2.0.1, el tipo de versión se clasificaría como sigue:

|Principal | Minor  | Revisión  |
| :--- | :----- | :----- |
| 2 | 0 | 1 | 

Azure Blockchain Service actualiza automáticamente las versiones de las revisiones de Quorum para los miembros existentes que se ejecutan dentro de los 30 días posteriores a que estén disponibles desde Quorum.

## <a name="availability-of-new-ledger-versions"></a>Disponibilidad de nuevas versiones del libro de contabilidad

Azure Blockchain Service proporciona las versiones principales y secundarias más recientes del libro de contabilidad de Quorum 60 días después de que el fabricante de Quorum las publique. Se proporcionan cuatro versiones secundarias, como máximo, para que los consorcios elijan cuándo aprovisionar un nuevo miembro y un consorcio. La actualización a una versión principal o secundaria no se admite actualmente.

## <a name="next-steps"></a>Pasos siguientes

[Límites en Azure Blockchain Service](limits.md)
