---
title: 'Referencia de API: Face API'
titleSuffix: Azure Cognitive Services
description: La referencia de API proporciona información sobre las API de algoritmos Person, LargePersonGroup/PersonGroup, LargeFaceList/FaceList y Face.
services: cognitive-services
author: SteveMSFT
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: reference
ms.date: 03/01/2018
ms.author: sbowles
ms.openlocfilehash: f4258f34bb7d353ee4e76f4675f4ef672a4a8c78
ms.sourcegitcommit: d4dfbc34a1f03488e1b7bc5e711a11b72c717ada
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/13/2019
ms.locfileid: "60816401"
---
# <a name="api-reference"></a>Referencia de API

Azure Face API es una API basada en la nube que proporciona algoritmos para la detección y el reconocimiento de caras. Las API de Face pueden clasificarse en las siguientes categorías:

- API de algoritmos de Face: Abarcan funciones básicas, como [detección](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236), [búsqueda de parecidos](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395237), [verificación](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523a), [identificación](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239) y [agrupación](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395238).
- [API de FaceList](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039524b): se usa para administrar una lista de caras en las [búsquedas de parecidos](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395237).
- [LargePersonGroup Person API](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599adcba3a7b9412a4d53f40): se usa para administrar las caras de los miembros de un grupo numeroso de personas para su [identificación](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239).
- [LargePersonGroup API](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599acdee6ac60f11b48b5a9d): se usa para administrar un conjunto de datos en un grupo numeroso de personas para su [identificación](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239).
- [API de LargeFaceList](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/5a157b68d2de3616c086f2cc): se utiliza para administrar una lista numerosa de caras para [buscar parecidos](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395237).
- [PersonGroup Person API](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523c): se usa para administrar las caras de los miembros de un grupo de personas para su [identificación](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239).
- [PersonGroup API](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395244): se usa para administrar un conjunto de datos de un grupo de personas para su [identificación](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239).
- [Snapshot API](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/snapshot-take): se usa para administrar una instantánea para la migración de datos entre suscripciones.
