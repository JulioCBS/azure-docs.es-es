---
title: Subdominios personalizados
titleSuffix: Azure Cognitive Services
description: Azure Cognitive Services usa nombres de subdominio personalizados para cada recurso creado a través de Azure Portal, Azure Cloud Shell o la CLI de Azure. A diferencia de los puntos de conexión regionales, que son comunes para todos los clientes de una región específica de Azure, los nombres de subdominio personalizados son únicos en el recurso. Los nombres de subdominio personalizados son necesarios para habilitar características como Azure Active Directory (Azure AD) para la autenticación.
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.topic: conceptual
ms.date: 07/24/2019
ms.author: erhopf
ms.openlocfilehash: 7153735052f96bef65bf3daaccde4eab3e61b0f9
ms.sourcegitcommit: c72ddb56b5657b2adeb3c4608c3d4c56e3421f2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2019
ms.locfileid: "68473013"
---
# <a name="custom-subdomain-names-for-cognitive-services"></a>Nombres de subdominios personalizados para Cognitive Services

Azure Cognitive Services usa nombres de subdominio personalizados para cada recurso creado a través de [Azure Portal](https://portal.azure.com), [Azure Cloud Shell](https://azure.microsoft.com/features/cloud-shell/) o la [CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli). A diferencia de los puntos de conexión regionales, que son comunes para todos los clientes de una región específica de Azure, los nombres de subdominio personalizados son únicos en el recurso. Los nombres de subdominio personalizados son necesarios para habilitar características como Azure Active Directory (Azure AD) para la autenticación.

## <a name="how-does-this-impact-existing-resources"></a>¿Cómo afecta esto a los recursos existentes?

Los recursos de Cognitive Services creados antes del 1 de julio de 2019 usarán los puntos de conexión regionales del servicio asociado. Estos puntos de conexión funcionarán con recursos nuevos y existentes.

Si quiere migrar un recurso existente para aprovechar los nombres de subdominio personalizado, de modo que pueda habilitar características como Azure AD, siga estas instrucciones:

1. Inicie sesión en Azure Portal y busque el recurso de Cognitive Services al que le gustaría agregar un nombre de subdominio personalizado.
2. En la hoja de **información general**, busque y seleccione la opción para **generar un nombre de dominio personalizado**.
3. Se abrirá un panel con instrucciones para crear un subdominio personalizado único para el recurso.
   > [!WARNING]
   > Después de crear un nombre de subdominio personalizado, **no** se puede cambiar.

## <a name="do-i-need-to-update-my-existing-resources"></a>¿Es necesario actualizar mis recursos existentes?

No. El punto de conexión regional continuará funcionando para las instancias de Cognitive Services nuevas y existentes; asimismo, el nombre de subdominio personalizado es opcional. Incluso si se agrega un nombre de subdominio personalizado, el punto de conexión regional continuará funcionando con el recurso.

## <a name="what-if-an-sdk-asks-me-for-the-region-for-a-resource"></a>¿Qué ocurre si un SDK me pide la región de un recurso?

> [!WARNING]
> En este momento, los servicios de voz **no** admiten subdominios personalizados. Use los puntos de conexión regionales cuando use los servicios de voz y los SDK asociados.

Los puntos de conexión regionales y los nombres de subdominio personalizados se admiten y se pueden usar indistintamente. Sin embargo, se requiere el punto de conexión completo.

La información de la región está disponible en la hoja de **información general** del recurso de [Azure Portal](https://portal.azure.com). Para obtener una lista completa de los puntos de conexión regionales, consulte [¿Hay alguna lista de puntos de conexión regionales?](#is-there-a-list-of-regional-endpoints)

## <a name="are-custom-subdomain-names-regional"></a>¿Son regionales los nombres de subdominios personalizados?

Sí. El uso de un nombre de subdominio personalizado no cambia ninguno de los aspectos regionales del recurso de Cognitive Services.

## <a name="what-are-the-requirements-for-a-custom-subdomain-name"></a>¿Cuáles son los requisitos para un nombre de subdominio personalizado?

Un nombre de subdominio personalizado es único para el recurso. El nombre solo puede incluir caracteres alfanuméricos y el carácter `-`; además, debe tener entre 2 y 64 caracteres de longitud y no puede terminar con `-`.

## <a name="can-i-change-a-custom-domain-name"></a>¿Se puede cambiar un nombre de dominio personalizado?

No. Una vez que se crea un nombre de subdominio personalizado y se asocia a un recurso, no se puede cambiar.

## <a name="can-i-reuse-a-custom-domain-name"></a>¿Puedo reutilizar un nombre de dominio personalizado?

Cada nombre de subdominio personalizado es único, por lo que para reutilizar un nombre de subdominio personalizado que haya sido asignado a un recurso de Cognitive Services, deberá eliminar el recurso existente. Una vez eliminado el recurso, puede volver a usar el nombre de subdominio personalizado.

## <a name="is-there-a-list-of-regional-endpoints"></a>¿Hay una lista de puntos de conexión regionales?

Sí. Esta es una lista de puntos de conexión regionales que puede usar con los recursos de Cognitive Services de Azure.

> [!NOTE]
> Las API de Translator Text y Bing Search usan puntos de conexión globales.

| Tipo de punto de conexión | Region | Punto de conexión |
|---------------|--------|----------|
| Público | Global (Translator Text y Bing) | `https://api.cognitive.microsoft.com` |
| | Este de Australia | `https://australiaeast.api.cognitive.microsoft.com` |
| | Sur de Brasil | `https://brazilsouth.api.cognitive.microsoft.com` |
| | Centro de Canadá | `https://canadacentral.api.cognitive.microsoft.com` |
| | Centro de EE. UU. | `https://centralus.api.cognitive.microsoft.com` |
| | Asia oriental | `https://eastasia.api.cognitive.microsoft.com` |
| | East US | `https://eastus.api.cognitive.microsoft.com` |
| | Este de EE. UU. 2 | `https://eastus2.api.cognitive.microsoft.com` |
| | Centro de Francia | `https://francecentral.api.cognitive.microsoft.com` |
| | India central | `https://centralindia.api.cognitive.microsoft.com` |
| | Este de Japón | `https://japaneast.api.cognitive.microsoft.com` |
| | Corea Central | `https://koreacentral.api.cognitive.microsoft.com` |
| | Centro-Norte de EE. UU | `https://northcentralus.api.cognitive.microsoft.com` |
| | Europa del Norte | `https://northeurope.api.cognitive.microsoft.com` |
| | Norte de Sudáfrica | `https://southafricanorth.api.cognitive.microsoft.com` |
| | Centro-Sur de EE. UU | `https://southcentralus.api.cognitive.microsoft.com` |
| | Sudeste asiático | `https://southeastasia.api.cognitive.microsoft.com` |
| | Sur de Reino Unido 2 | `https://uksouth.api.cognitive.microsoft.com` |
| | Centro occidental de EE.UU. | `https://westcentralus.api.cognitive.microsoft.com` |
| | Europa occidental | `https://westeurope.api.cognitive.microsoft.com` |
| | Oeste de EE. UU. | `https://westus.api.cognitive.microsoft.com` |
| | Oeste de EE. UU. 2 | `https://westus2.api.cognitive.microsoft.com` |
| Gobierno de EE. UU. | Gobierno de EE. UU. - Virginia | `https://virginia.api.cognitive.microsoft.us` |
| China | Este de China 2 | `https://chinaeast2.api.cognitive.azure.cn` |
| | Norte de China | `https://chinanorth.api.cognitive.azure.cn` |

## <a name="see-also"></a>Otras referencias

* [¿Qué es Cognitive Services?](Welcome.md)
* [Autenticación](authentication.md)
