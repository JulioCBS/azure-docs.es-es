---
title: Adición de un origen de eventos de IoT Hub a Azure Time Series Insights | Microsoft Docs
description: En este artículo se describe cómo agregar un origen de eventos que está conectado a un centro de IoT Hub a su entorno de Time Series Insights.
ms.service: time-series-insights
services: time-series-insights
author: ashannon7
ms.author: dpalled
manager: cshankar
ms.reviewer: v-mamcge, jasonh, kfile
ms.workload: big-data
ms.topic: conceptual
ms.date: 08/06/2019
ms.custom: seodec18
ms.openlocfilehash: 234fe28cb8bd9dcb97e307836961e2f587e15181
ms.sourcegitcommit: 670c38d85ef97bf236b45850fd4750e3b98c8899
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/08/2019
ms.locfileid: "68846641"
---
# <a name="add-an-iot-hub-event-source-to-your-time-series-insights-environment"></a>Adición de un origen de eventos de IoT Hub al entorno de Time Series Insights

Este artículo describe cómo usar Azure Portal para agregar un origen de eventos que lee datos de Azure IoT Hub en su entorno de Time Series Insights.

> [!NOTE]
> Las instrucciones de este artículo se aplican tanto a los entornos de Azure Time Series Insights con disponibilidad general como a los entornos en versión preliminar.

## <a name="prerequisites"></a>Requisitos previos

* Creación de un [entorno de Azure Time Series Insights](time-series-insights-update-create-environment.md).
* [Creación de una instancia de IoT Hub mediante Azure Portal](../iot-hub/iot-hub-create-through-portal.md).
* El centro de IoT Hub debe tener eventos de mensajes activos en proceso de envío.
* Cree un grupo de consumidores dedicado en el centro de IoT Hub para que el entorno de Time Series Insight los consuma. Cada origen del evento de Time Series Insights tiene que tener su propio grupo de consumidores dedicado que no se comparte con ningún otro consumidor. Si varios lectores consumen eventos desde el mismo grupo de consumidores, es probable que todos los lectores vean errores. Para más información, consulte la [Guía para desarrolladores de IoT Hub](../iot-hub/iot-hub-devguide.md).

### <a name="add-a-consumer-group-to-your-iot-hub"></a>Adición de un grupo de consumidores a IoT Hub

Las aplicaciones usan grupos de consumidores para extraer datos de Azure IoT Hub. Para leer datos de manera confiable del centro de IoT Hub, proporcione un grupo de consumidores dedicado, solo para su uso en este entorno de Time Series Insights.

Para agregar un nuevo grupo de consumidores a la instancia de IoT Hub:

1. En Azure Portal, busque y abra el centro de IoT Hub.

1. En el menú, en **Configuración**, seleccione **Puntos de conexión integrados** y, luego, seleccione el punto de conexión **Eventos**.

   [![Selección del botón Eventos en la página Puntos de conexión integrados](media/time-series-insights-how-to-add-an-event-source-iothub/iothub-one.png)](media/time-series-insights-how-to-add-an-event-source-iothub/iothub-one.png#lightbox)

1. En **Grupos de consumidores**, escriba un nombre único para el grupo de consumidores. Use este mismo nombre en el entorno de Time Series Insights al crear un nuevo origen del evento.

1. Seleccione **Guardar**.

## <a name="add-a-new-event-source"></a>Adición de un nuevo origen del evento

1. Inicie sesión en el [Azure Portal](https://portal.azure.com).

1. En el menú izquierdo, seleccione **Todos los recursos**. Seleccione el entorno de Time Series Insights.

1. En **Topología del entorno** seleccione **Orígenes de eventos**y, a continuación, seleccione **Agregar**.

   [![Selección de Orígenes de eventos y del botón Agregar](media/time-series-insights-how-to-add-an-event-source-iothub/iothub-two.png)](media/time-series-insights-how-to-add-an-event-source-iothub/iothub-two.png#lightbox)

1. En el panel **Nuevo origen del evento**, en **Nombre del origen del evento**, especifique un nombre que sea exclusivo para este entorno de Time Series Insights. Por ejemplo, escriba **event-stream**.

1. En **Origen**, seleccione **IoT Hub**.

1. Seleccione un valor para **Opción de importación**:

   * Si ya tiene un centro de IoT en una de sus suscripciones, seleccione **Usar IoT Hub desde las suscripciones disponibles**. Esta opción presenta el enfoque más sencillo.
   
     [![Selección de opciones en el panel Nuevo origen del evento](media/time-series-insights-how-to-add-an-event-source-iothub/iothub-three.png)](media/time-series-insights-how-to-add-an-event-source-iothub/iothub-three.png#lightbox)

    * En la siguiente tabla se explican las propiedades necesarias para la opción **Usar IoT Hub desde las suscripciones disponibles**:

       [![Panel Nuevo origen del evento: propiedades que hay que establecer en la opción Usar IoT Hub desde la opción de suscripciones disponibles](media/time-series-insights-how-to-add-an-event-source-iothub/iothub-four.png)](media/time-series-insights-how-to-add-an-event-source-iothub/iothub-four.png#lightbox)

       | Propiedad | DESCRIPCIÓN |
       | --- | --- |
       | Id. de suscripción | Seleccione la suscripción en la que se creó el centro de IoT.
       | Nombre de IoT Hub | Seleccione el nombre del centro de IoT.
       | Nombre de la directiva de IoT Hub | Seleccione la directiva de acceso compartido. Puede encontrar esta directiva en la pestaña Configuración de centro de IoT. Cada directiva de acceso compartido tiene un nombre, los permisos establecidos y las claves de acceso. La directiva de acceso compartido para el origen de eventos *debe* tener permisos de **conexión de servicios**.
       | Clave de la directiva de IoT Hub | La clave se rellena previamente.
       | Grupo de consumidores de IoT Hub | El grupo de consumidores que lee eventos del centro de IoT. Es muy recomendable usar un grupo de consumidores dedicado para el origen del evento.
       | Formato de serialización de eventos | Actualmente, JSON es el único formato de serialización disponible. Los mensajes de eventos deberán estar en este formato o no se podrá leer ningún dato. |
       | Nombre de la propiedad de marca de tiempo | Para determinar este valor, es necesario que comprenda el formato de mensaje de los datos del mensaje que se envía al centro de IoT. Este valor es el **nombre** de la propiedad específica del evento en los datos del mensaje que quiere usar como marca de tiempo del evento. El valor distingue mayúsculas de minúsculas. Si se deja en blanco, se usa la **hora de puesta en cola del evento** como marca de tiempo del evento en el origen del evento. |

    * Si el centro de IoT es externo a sus suscripciones, o si desea elegir opciones avanzadas, seleccione **Proporcionar configuración del centro de IoT de forma manual**.

      En la siguiente tabla se explican las propiedades necesarias para la opción **Proporcionar configuración del Centro de IoT de forma manual**:

       | Propiedad | DESCRIPCIÓN |
       | --- | --- |
       | Id. de suscripción | La suscripción en la que se creó este centro de IoT.
       | Resource group | El nombre del grupo de recursos en el que se creó este centro de IoT.
       | Nombre de IoT Hub | El nombre del centro de IoT. Cuando creó el centro de IoT especificó un nombre para este.
       | Nombre de la directiva de IoT Hub | La directiva de acceso compartido. Puede crear esta directiva en la pestaña Configuración de centro de IoT. Cada directiva de acceso compartido tiene un nombre, los permisos establecidos y las claves de acceso. La directiva de acceso compartido para el origen de eventos *debe* tener permisos de **conexión de servicios**.
       | Clave de la directiva de IoT Hub | La clave de acceso compartido que se usa para autenticar el acceso al espacio de nombres de Azure Service Bus. Especifique la clave principal o secundaria aquí.
       | Grupo de consumidores de IoT Hub | El grupo de consumidores que lee eventos del centro de IoT. Es muy recomendable usar un grupo de consumidores dedicado para el origen del evento.
       | Formato de serialización de eventos | Actualmente, JSON es el único formato de serialización disponible. Los mensajes de eventos deberán estar en este formato o no se podrá leer ningún dato. |
       | Nombre de la propiedad de marca de tiempo | Para determinar este valor, es necesario que comprenda el formato de mensaje de los datos del mensaje que se envía al centro de IoT. Este valor es el **nombre** de la propiedad específica del evento en los datos del mensaje que quiere usar como marca de tiempo del evento. El valor distingue mayúsculas de minúsculas. Si se deja en blanco, se usa la **hora de puesta en cola del evento** como marca de tiempo del evento en el origen del evento. |

1. Agregue el nombre del grupo de consumidores dedicado de Time Series Insights que agregó al centro de IoT.

1. Seleccione **Crear**.

   [![El botón Crear](media/time-series-insights-how-to-add-an-event-source-iothub/iothub-five.png)](media/time-series-insights-how-to-add-an-event-source-iothub/iothub-five.png#lightbox)

1. Una vez que se haya creado el origen del evento, Time Series Insights iniciará automáticamente la transmisión de datos al entorno.

## <a name="next-steps"></a>Pasos siguientes

* [Defina las directivas de acceso a datos](time-series-insights-data-access.md) para proteger los datos.

* [Envíe eventos](time-series-insights-send-events.md) al origen de eventos.

* Acceso al entorno en el [explorador de Time Series Insights](https://insights.timeseries.azure.com).
