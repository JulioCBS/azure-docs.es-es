---
title: azcopy logout | Microsoft Docs
description: En este artículo se proporciona información de referencia del comando azcopy logout.
author: normesta
ms.service: storage
ms.topic: reference
ms.date: 08/26/2019
ms.author: normesta
ms.subservice: common
ms.reviewer: zezha-msft
ms.openlocfilehash: f42e2517f8c40855e56bd062fe8bc9b22634d4dc
ms.sourcegitcommit: 532335f703ac7f6e1d2cc1b155c69fc258816ede
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/30/2019
ms.locfileid: "70196802"
---
# <a name="azcopy-logout"></a>azcopy logout

Cierra la sesión del usuario y finaliza el acceso a los recursos de Azure Storage.

## <a name="synopsis"></a>Sinopsis

Este comando quitará toda la información de inicio de sesión almacenada en caché del usuario actual.

```azcopy
azcopy logout [flags]
```

## <a name="options"></a>Opciones

|Opción|DESCRIPCIÓN|
|--|--|
|-h, --help|Muestra el contenido de la ayuda para el comando logout.|

## <a name="options-inherited-from-parent-commands"></a>Opciones heredadas de comandos primarios

|Opción|DESCRIPCIÓN|
|---|---|
|--cap-mbps uint32|Limita la velocidad de transferencia, en megabits por segundo. El rendimiento en un momento dado puede variar ligeramente del límite. Si esta opción se establece en cero o se omite, el rendimiento no se limita.|
|--output-type string|Formato de la salida del comando. Las opciones incluyen: text, json. El valor predeterminado es "text".|

## <a name="see-also"></a>Otras referencias

- [azcopy](storage-ref-azcopy.md)
