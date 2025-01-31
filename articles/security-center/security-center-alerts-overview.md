---
title: Alertas de seguridad en Azure Security Center | Microsoft Docs
description: En este tema se explica qué son las alertas de seguridad y los distintos tipos disponibles en Azure Security Center.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 1b71e8ad-3bd8-4475-b735-79ca9963b823
ms.service: security-center
ms.topic: conceptual
ms.date: 08/25/2019
ms.author: memildin
ms.openlocfilehash: 3b4b02574c028822d25d841376b127a718243b2e
ms.sourcegitcommit: 8a717170b04df64bd1ddd521e899ac7749627350
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/23/2019
ms.locfileid: "71202566"
---
# <a name="security-alerts-in-azure-security-center"></a>Alertas de seguridad en Azure Security Center

En Azure Security Center, existe una amplia variedad de alertas para los numerosos y diversos tipos de recursos. Security Center genera alertas para los recursos implementados en Azure, así como para los recursos implementados en entornos locales y entornos en la nube híbrida.

El nivel estándar de Azure Security Center dispone de medidas de detección avanzadas. Hay una evaluación gratuita disponible. Si desea actualizar la versión, puede seleccionar las opciones disponibles en el Plan de tarifa de la [Directiva de seguridad](security-center-pricing.md). Para más información sobre los precios, visite la [página de Security Center](https://azure.microsoft.com/pricing/details/security-center/) .

## Respuesta a las amenazas actuales <a name="respond-threats"> </a>

En los últimos 20 años, las amenazas han cambiado considerablemente. Por lo general, en el pasado, las compañías solo tenían que preocuparse por las alteraciones que atacantes individuales realizaban en sus sitios web, quienes normalmente solo pretendían comprobar “qué es lo que eran capaces de hacer”. Hoy en día, los atacantes son mucho más sofisticados y están más organizados. Normalmente, se mueven por un fin económico o estratégico. También disponen de más recursos, ya que a veces están financiados por estados o por el crimen organizado.

Estas realidades cambiantes han propiciado un nivel de profesionalidad entre los atacantes que no tiene precedentes. Ya no les interesa alterar los sitios web. Ahora, lo que buscan es robar información, cuentas financieras y datos privados, que pueden usar con fines lucrativos en el mercado libre o para sacar provecho de una determinada situación política, militar o empresarial. Si los atacantes con un objetivo financiero son motivo de preocupación, aún lo son más los que pretenden abrir una brecha en las redes para dañar la infraestructura o infligir daños personales.

Como respuesta, las organizaciones suelen implementar varias soluciones específicas, que se centran en defender el perímetro o los puntos de conexión de la empresa mediante la búsqueda de firmas de ataque conocidas. Estas soluciones suelen generar un gran número de alertas con un bajo nivel de confiabilidad, lo que requiere que un analista de seguridad las cribe e investigue. La mayor parte de las organizaciones no disponen de la experiencia ni del tiempo necesarios para responder a estas alertas, por lo que muchas quedan desatendidas.  

Además, los atacantes han mejorado sus métodos para debilitar muchas de las defensas basadas en firmas y para [adaptarse a los entornos en la nube](https://azure.microsoft.com/blog/detecting-threats-with-azure-security-center/). Por tanto, se necesitan nuevos enfoques que permitan identificar con mayor rapidez las nuevas amenazas, así como acelerar su detección y respuesta.

## <a name="what-are-security-alerts"></a>¿Qué son las alertas de seguridad?

Las alertas son notificaciones que Security Center genera cuando detecta amenazas en los recursos. Security Center asigna prioridades y enumera las alertas, junto con la información necesaria para que pueda investigar rápidamente el problema. Security Center también proporciona recomendaciones sobre el modo en que puede corregir un ataque.

## ¿Cómo detecta Security Center las amenazas? <a name="detect-threats"></a>

Los investigadores de seguridad de Microsoft trabajan sin descanso para localizar amenazas. Dada la presencia global de Microsoft en la nube y en sistemas locales, tienen acceso a un amplio conjunto de recursos de telemetría. La amplitud y diversidad de estos conjuntos de datos permite detectar nuevos patrones y tendencias de ataque tanto en sus productos locales, destinados a particulares y empresas, como en sus servicios en línea. Como resultado, Security Center es capaz de actualizar rápidamente los algoritmos de detección a medida que los atacantes idean nuevas y más sofisticadas vulnerabilidades de seguridad. Este enfoque le ayuda a mantenerse al día en entornos llenos de amenazas que cambian continuamente.

Para detectar amenazas reales y reducir los falsos positivos, Security Center recopila, analiza e integra los datos de registro de los recursos de Azure y de la red. También funciona con soluciones de asociados conectados, como firewalls y soluciones de protección de puntos de conexión. Security Center analiza estos datos (a menudo, relacionando la información de diferentes orígenes) para identificar las amenazas.

![Recopilación y presentación de datos de Security Center](./media/security-center-alerts-overview/security-center-detection-capabilities.png)

Security Center utiliza análisis avanzados que superan con creces los enfoques basados en firmas. Los grandes avances registrados en las tecnologías de macrodatos y [aprendizaje automático](https://azure.microsoft.com/blog/machine-learning-in-azure-security-center/) se utilizan para evaluar eventos en todo el tejido de la nube, lo que permite detectar amenazas que no se podrían identificar mediante enfoques manuales, así como predecir la evolución de los ataques. Estas técnicas de análisis son:

* **Información integrada sobre amenazas**: busca actores malintencionados conocidos aprovechando la información global sobre amenazas de los productos y servicios de Microsoft, la Unidad de crímenes digitales de Microsoft (DCU), Microsoft Security Response Center (MSRC) y otras fuentes externas.
* **Análisis del comportamiento**: aplica patrones conocidos para identificar comportamientos malintencionados.
* **Detección de anomalías**: usa la generación de perfiles estadísticos para crear una base de referencia histórica. Alerta de las desviaciones de las referencias que se ajustan a un posible vector de ataque.

En los temas siguientes se describe cada uno de estos análisis con más detalle.

### <a name="integrated-threat-intelligence"></a>Información integrada sobre amenazas

Microsoft dispone de una ingente cantidad de información sobre amenazas globales. Los recursos telemétricos proceden de diferentes fuentes, como Azure, Office 365, Microsoft CRM Online, Microsoft Dynamics AX, outlook.com, MSN.com, la Unidad de crímenes digitales de Microsoft (DCU) y Microsoft Security Response Center (MSRC). Los investigadores también cuentan con la información sobre amenazas que comparten los principales proveedores de servicios en la nube y que procede de fuentes de terceros. Azure Security Center puede usar todos estos datos para avisarle de las amenazas procedentes de actores malintencionados conocidos.

### <a name="behavioral-analytics"></a>Análisis del comportamiento

El análisis del comportamiento es una técnica que analiza datos y los compara con una serie de patrones conocidos. No obstante, estos patrones no son simples firmas, sino que están determinados por unos complejos algoritmos de aprendizaje automático que se aplican a conjuntos de datos masivos. También se determinan por medio de un análisis cuidadoso, llevado a cabo por analistas expertos, de los comportamientos malintencionados. Azure Security Center puede utilizar el análisis de comportamiento para identificar recursos en peligro a partir del análisis de registros de máquinas virtuales, registros de dispositivos de redes virtuales, registros de tejidos, volcados de memoria y otros orígenes.

Además, existe una correlación con otras señales que se comprueban para fundamentar las pruebas de que se trata una campaña de gran difusión. Dicha correlación permite identificar eventos que se ajustan a unos indicadores de peligro establecidos. 

### <a name="anomaly-detection"></a>Detección de anomalías

Azure Security Center también utiliza la detección de anomalías para identificar amenazas. A diferencia del análisis del comportamiento, que depende de patrones conocidos obtenidos a partir de grandes conjuntos de datos, la detección de anomalías es una técnica más “personalizada” y se basa en referencias que son específicas de las implementaciones. El aprendizaje automático se aplica para determinar la actividad normal de las implementaciones. A partir de ahí, se generan reglas para definir condiciones de valores atípicos que podrían constituir un evento de seguridad.

## <a name="continuous-monitoring-and-assessments"></a>Supervisión y evaluaciones continuas

Azure Security Center se beneficia de la existencia de equipos de científicos de datos e investigadores de seguridad de Microsoft que supervisan sin descanso los cambios que se registran en el terreno de las amenazas. Estos son algunas de las iniciativas que llevan a cabo:

* **Supervisión de la inteligencia sobre amenazas**: la inteligencia sobre amenazas incluye mecanismos, indicadores, implicaciones y notificaciones para las amenazas nuevas o existentes. Esta información se comparte con la comunidad de seguridad, y Microsoft supervisa sin descanso las fuentes de orígenes internos y externos.
* **Uso compartido de señales**: la información que recopilan los equipos de seguridad en la amplia cartera de servicios tanto locales como en la nube, de servidores y de dispositivos cliente de punto de conexión de Microsoft se comparte y se analiza.
* **Especialistas en seguridad de Microsoft** colaboración continua con equipos de Microsoft que trabajan en campos de seguridad especializados, como análisis forense y detección de ataques web.
* **Ajuste de la detección**: los algoritmos se ejecutan en conjuntos de datos de clientes reales y los investigadores de seguridad trabajan en conjunción con los clientes para validar los resultados. Los falsos positivos y los positivos verdaderos se utilizan para perfeccionar los algoritmos de aprendizaje automático.

Toda esta combinación de esfuerzos culmina en nuevas y mejoradas técnicas de detección, de las que puede beneficiarse al instante sin ninguna acción por su parte.

## Tipos de alertas de seguridad <a name="security-alert-types"> </a>

Los temas siguientes le guiarán a través de las diferentes alertas según los tipos de recursos:

* [Alertas de servidores y máquinas virtuales de IaaS](security-center-alerts-iaas.md)
* [Alertas de procesos nativos](security-center-alerts-compute.md)
* [Alertas de servicios de datos](security-center-alerts-data-services.md)

Los temas siguientes explican cómo Security Center usa los diferentes datos de telemetría que recopila de la integración con la infraestructura de Azure a fin de aplicar capas de protección adicionales para los recursos implementados en Azure:

* [Alertas de la capa de servicios](security-center-alerts-service-layer.md)
* [Integración con productos de seguridad de Azure](security-center-alerts-integration.md)

## <a name="what-are-security-incidents"></a>¿Qué son los incidentes de seguridad?

Un incidente de seguridad es una recopilación de alertas relacionadas, en lugar de una enumeración de alertas individuales. Security Center usa la [correlación de alertas inteligentes en la nube](security-center-alerts-cloud-smart.md) para poner en correlación diferentes alertas y señales de baja fidelidad en incidentes de seguridad.

Utilizando los incidentes, Security Center proporciona una vista única de una campaña de ataques y todas las alertas relacionadas. Esta vista le permite comprender rápidamente las acciones que ha realizado el atacante y qué recursos se han visto afectados. Para obtener más información, consulte [Correlación de alertas inteligentes en la nube](security-center-alerts-cloud-smart.md).

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido obtenido información sobre los distintos tipos de alertas disponibles en Security Center. Para más información, consulte:

* [Guía de planeamiento y operaciones de Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-planning-and-operations-guide)
* [Preguntas más frecuentes sobre Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-faq)

