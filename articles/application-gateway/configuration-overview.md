---
title: Introducción a la configuración de Azure Application Gateway
description: En este artículo se describe cómo configurar los componentes de Azure Application Gateway
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: article
ms.date: 6/1/2019
ms.author: absha
ms.openlocfilehash: 65cf71140d1706b8607e721ac323b1a97ae272fa
ms.sourcegitcommit: d3dced0ff3ba8e78d003060d9dafb56763184d69
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2019
ms.locfileid: "69898445"
---
# <a name="application-gateway-configuration-overview"></a>Introducción a la configuración de Application Gateway

Azure Application Gateway consta de varios componentes que se pueden configurar de varias maneras para diferentes escenarios. En este artículo se muestra cómo configurar cada componente.

![Diagrama de flujo de los componentes de Application Gateway](./media/configuration-overview/configuration-overview1.png)

Esta imagen muestra una aplicación que tiene tres clientes de escucha. Los dos primeros son clientes de escucha multisitio para `http://acme.com/*` y `http://fabrikam.com/*`, respectivamente. Ambos escuchan en el puerto 80. El tercero es un cliente de escucha básico que tiene la terminación de la capa de sockets seguros (SSL) de un extremo a otro.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Requisitos previos

### <a name="azure-virtual-network-and-dedicated-subnet"></a>Red virtual de Azure y subred dedicada

Una puerta de enlace de aplicaciones es una implementación dedicada en su red virtual. Dentro de la red virtual, es necesaria una subred dedicada para la puerta de enlace de aplicaciones. Puede tener varias instancias de una implementación de puerta de enlace de aplicaciones determinada en una subred. También se pueden implementar otras puertas de enlace de aplicaciones en la subred. Pero no se puede implementar ningún otro recurso en la subred de la puerta de enlace de aplicaciones.

> [!NOTE]
> No se pueden combinar Application Gateway Standard y Standard_v2 en la misma subred.

#### <a name="size-of-the-subnet"></a>Tamaño de la subred

Application Gateway consume una dirección IP privada por instancia, más otra dirección IP privada si se configura una dirección IP de front-end privada.

Además, Azure se reserva 5 direcciones IP en cada subred para uso interno: las 4 primeras direcciones IP y la última de cada subred. Por ejemplo, suponga que hay 15 instancias de Application Gateway sin ninguna dirección IP de front-end privada. Necesitará al menos 20 direcciones IP para esta subred: 5 para uso interno y 15 para las instancias de Application Gateway. Por lo tanto, necesita una subred de tamaño /27 o superior.

Ahora imagine una subred que tiene 27 instancias de Application Gateway y una dirección IP de front-end privada. En este caso, necesita 33 direcciones IP: 27 para las instancias de Application Gateway, 1 para la dirección IP de front-end privada y 5 para uso interno. Por lo tanto, necesita una subred de tamaño /26 o superior.

Se recomienda que utilice un tamaño /28 como mínimo. Este tamaño le ofrece 11 direcciones IP utilizables. Si la carga de la aplicación requiere más de 10 direcciones IP, considere la posibilidad de usar un tamaño de subred /27 o/26.

#### <a name="network-security-groups-on-the-application-gateway-subnet"></a>Grupos de seguridad de red en la subred de Application Gateway

Los grupos de seguridad se admiten en Application Gateway. Sin embargo, hay varias restricciones:

- Debe incluir excepciones para el tráfico entrante en los puertos 65503-65534 para la SKU v1 de Application Gateway y los puertos 65200-65535 para la SKU v2. Este intervalo de puertos es necesario para la comunicación de la infraestructura de Azure. Estos puertos están protegidos (bloqueados) mediante certificados de Azure. Las entidades externas, incluidos los clientes de esas puertas de enlace, no podrán iniciar ningún cambio en esos puntos de conexión sin los certificados adecuados en vigor.

- No puede bloquearse la conectividad saliente de Internet. Las reglas de salida predeterminadas del grupo de seguridad de red permiten la conectividad a Internet. Se recomienda que:

  - No elimine las reglas de salida predeterminadas.
  - No cree otras reglas de salida que denieguen la conectividad saliente de Internet.

- Se debe permitir el tráfico de la etiqueta **AzureLoadBalancer**.

##### <a name="allow-application-gateway-access-to-a-few-source-ips"></a>Permitir que Application Gateway acceda a varias direcciones IP de origen

En este escenario, puede usar grupos de seguridad de red en la subred de Application Gateway. Ponga las restricciones siguientes en la subred en este orden de prioridad:

1. Permitir el tráfico entrante de una IP o intervalo IP de origen.
2. Permita las solicitudes entrantes de todos los orígenes a los puertos 65503-65534 para la SKU de Application Gateway v1 y a los puertos 65200-65535 para la SKU v2 para la [comunicación de mantenimiento de back-end](https://docs.microsoft.com/azure/application-gateway/application-gateway-diagnostics). Este intervalo de puertos es necesario para la comunicación de la infraestructura de Azure. Estos puertos están protegidos (bloqueados) mediante certificados de Azure. Sin los certificados adecuados en vigor, las entidades externas no podrán iniciar cambios en esos puntos de conexión.
3. Permitir sondeos entrantes de Azure Load Balancer (con la etiqueta *AzureLoadBalancer*) y el tráfico de red virtual entrante (con la etiqueta *VirtualNetwork*) en el [grupo de seguridad de red](https://docs.microsoft.com/azure/virtual-network/security-overview).
4. Bloquear todo el tráfico entrante restante mediante una regla Denegar todo.
5. Permitir el tráfico saliente a Internet para todos los destinos.

#### <a name="user-defined-routes-supported-on-the-application-gateway-subnet"></a>Rutas definidas por el usuario admitidas en la subred de Application Gateway

Para la SKU v1, las rutas definidas por el usuario (UDR) se admiten en la subred de la puerta de enlace de aplicaciones, siempre que no alteren la comunicación de solicitud y respuesta de un extremo a otro. Por ejemplo, puede configurar una ruta definida por el usuario en la subred de Application Gateway para que apunte a un dispositivo de firewall para la inspección de paquetes. Debe asegurarse de que el paquete puede llegar a su destino previsto después de la inspección. El no hacerlo podría resultar en un sondeo del estado o en un comportamiento de enrutamiento de tráfico incorrectos. Esto incluye las rutas aprendidas o las rutas 0.0.0.0/0 predeterminadas que se propagan por las puertas de enlace de VPN o de Azure ExpressRoute en la red virtual.

Las UDR en la subred de Application Gateway no son compatibles con la SKU v2. Para más información, consulte [Azure Application Gateway v2 SKU](application-gateway-autoscaling-zone-redundant.md#differences-with-v1-sku).

> [!NOTE]
> Las UDR no son compatibles con la SKU v2.  Si necesita UDR, debe seguir implementando la SKU v1.

> [!NOTE]
> El uso de rutas definidas por el usuario en la subred de Application Gateway hace que el estado de mantenimiento en la [vista de mantenimiento de back-end](https://docs.microsoft.com/azure/application-gateway/application-gateway-diagnostics#back-end-health) aparezca como "Desconocido". Esto también hace que se produzcan errores en la generación de registros y métricas de Application Gateway. Se recomienda que no use rutas definidas por el usuario en la subred de Application Gateway para que pueda ver el estado, los registros y las métricas del back-end.

## <a name="front-end-ip"></a>Dirección IP de front-end

Puede configurar la puerta de enlace de aplicaciones para que tenga una dirección IP pública, una dirección IP privada o ambas. Se necesita una dirección IP pública si hospeda un back-end al que los clientes deben acceder desde Internet mediante una IP virtual (VIP) accesible desde Internet. 

No se necesita ninguna dirección IP pública para un punto de conexión interno que no está expuesto a Internet. Eso se conoce como un punto de conexión de *equilibrador de carga interno* (ILB). Un equilibrador de carga interno de puerta de enlace de aplicaciones es útil para aplicaciones de línea de negocio internas no expuestas a Internet. También es útil para los servicios y niveles de una aplicación de varios niveles dentro de un límite de seguridad que no están expuestos a Internet, pero que siguen necesitando distribución de carga round robin, permanencia de sesión o terminación SSL.

Se admite solo 1 dirección IP pública o 1 dirección IP privada. Puede elegir la dirección IP de front-end cuando cree la puerta de enlace de aplicaciones.

- Para una dirección IP pública, puede crear una nueva dirección IP pública o usar una ya existente en la misma ubicación que la puerta de enlace de aplicaciones. Si crea una nueva, el tipo de dirección IP que seleccione (estática o dinámica) no se podrá cambiar más adelante. Para más información, consulte esta comparación entre [direcciones IP públicas estáticas y dinámicas](https://docs.microsoft.com/azure/application-gateway/application-gateway-components#static-versus-dynamic-public-ip-address).

- Para una dirección IP privada, puede especificar una dirección IP privada de la subred en la que se creó la puerta de enlace de aplicaciones. Si no especifica ninguna, se seleccionará automáticamente una dirección IP aleatoria de la subred. Para más información, consulte [Creación de una puerta de enlace de aplicaciones con un equilibrador de carga interno](https://docs.microsoft.com/azure/application-gateway/application-gateway-ilb-arm).

Una dirección IP de front-end está asociada a un *cliente de escucha* que comprueba las solicitudes entrantes en esta dirección IP.

## <a name="listeners"></a>Clientes de escucha

Un cliente de escucha es una entidad lógica que comprueba las solicitudes de conexión entrantes mediante el puerto, protocolo, host y dirección IP. Al configurar el cliente de escucha, debe especificar valores que coincidan con los valores correspondientes de la solicitud entrante de la puerta de enlace.

Cuando crea una puerta de enlace de aplicaciones mediante Azure Portal, también puede crear un cliente de escucha predeterminado eligiendo el protocolo y el puerto de este cliente. Puede elegir si habilita compatibilidad con HTTP2 en el cliente de escucha o no. Después de crear la puerta de enlace de aplicaciones, puede editar la configuración de ese cliente de escucha predeterminado (*appGatewayHttpListener*/*appGatewayHttpsListener*) o crear otros clientes nuevos.

### <a name="listener-type"></a>Tipo de cliente de escucha

Cuando crea un nuevo cliente de escucha, puede elegir entre [*básico* y *multisitio*](https://docs.microsoft.com/azure/application-gateway/application-gateway-components#types-of-listeners).

- Si va a hospedar un único sitio detrás de una puerta de enlace de aplicaciones, elija básico. Aprenda a [crear una puerta de enlace de aplicaciones con un cliente de escucha básico](https://docs.microsoft.com/azure/application-gateway/quick-create-portal).

- Si va a configurar más de una aplicación web o varios subdominios del mismo dominio primario en la misma instancia de la puerta de enlace de aplicaciones, elija el cliente de escucha multisitio. Para un cliente de escucha multisitio, también debe especificar un nombre de host. Esto se debe a que Application Gateway se basa en los encabezados de host HTTP 1.1 para hospedar más de un sitio web en la misma dirección IP pública y en el mismo puerto.

#### <a name="order-of-processing-listeners"></a>Orden de procesamiento de los clientes de escucha

Para la SKU v1, los clientes de escucha se procesan en el orden en que aparecen. Si un cliente de escucha coincide con una solicitud entrante, el cliente de escucha procesa esa solicitud primero. Por tanto, configure clientes de escucha multisitio en vez de básicos para asegurarse de que el tráfico se enruta al back-end correcto.

Para la SKU v2, los clientes de escucha multisitio se procesan antes que los básicos.

### <a name="front-end-ip"></a>Dirección IP de front-end

Elija la dirección IP de front-end que va a asociar a este cliente de escucha. El cliente de escucha escuchará las solicitudes entrantes en esta dirección IP.

### <a name="front-end-port"></a>Puerto de front-end

Elija el puerto de front-end. Seleccione un puerto existente o cree uno nuevo. Elija cualquier valor del [intervalo de puertos permitidos](https://docs.microsoft.com/azure/application-gateway/application-gateway-components#ports). Puede usar no solo los puertos conocidos, como el 80 y el 443, sino cualquier puerto permitido personalizado que sea adecuado. Se puede usar un puerto para los clientes de escucha de acceso público y los de acceso privado.

### <a name="protocol"></a>Protocolo

Elija HTTP o HTTPS:

- Si elige HTTP, se descifrará el tráfico entre el cliente y la puerta de enlace de aplicaciones.

- Seleccione HTTPS si desea [terminación SSL](https://docs.microsoft.com/azure/application-gateway/overview#secure-sockets-layer-ssltls-termination) o [cifrado SSL de un extremo a otro](https://docs.microsoft.com/azure/application-gateway/ssl-overview). Si lo hace, se cifrará el tráfico entre el cliente y la puerta de enlace de aplicaciones. Y la conexión SSL termina en la puerta de enlace de aplicaciones. Si desea cifrado SSL de un extremo a otro, debe elegir HTTPS y configurar el valor de **HTTP de back-end**. Esto garantiza que el tráfico se volverá a cifrar cuando vaya desde la puerta de enlace de aplicaciones al back-end.

Para configurar la terminación SSL y el cifrado de un extremo a otro, debe agregar un certificado al cliente de escucha para permitir a la puerta de enlace de aplicaciones derivar una clave simétrica. Esto viene determinado por la especificación del protocolo SSL. La clave simétrica se usa para cifrar y descifrar el tráfico que se envía a la puerta de enlace. El certificado de la puerta de enlace debe estar en formato de Intercambio de información personal (PFX). Este formato le permite exportar la clave privada que usa la puerta de enlace para cifrar y descifrar el tráfico.

#### <a name="supported-certificates"></a>Certificados admitidos

Consulte los [certificados compatibles con la terminación SSL](https://docs.microsoft.com/azure/application-gateway/ssl-overview#certificates-supported-for-ssl-termination).

### <a name="additional-protocol-support"></a>Compatibilidad con protocolos adicionales

#### <a name="http2-support"></a>Compatibilidad con HTTP2

La compatibilidad con el protocolo HTTP/2 está disponible únicamente para los clientes que se conectan a los clientes de escucha de la puerta de aplicaciones. La comunicación con granjas de servidores back-end es a través de HTTP/1.1. De forma predeterminada, HTTP/2 está deshabilitado. El siguiente fragmento de código de Azure PowerShell muestra cómo habilitarlo:

```azurepowershell
$gw = Get-AzApplicationGateway -Name test -ResourceGroupName hm

$gw.EnableHttp2 = $true

Set-AzApplicationGateway -ApplicationGateway $gw
```

#### <a name="websocket-support"></a>Compatibilidad con WebSocket

La compatibilidad con WebSocket está habilitada de forma predeterminada. No hay ninguna opción de configuración que permita al usuario habilitarla o deshabilitarla. Puede usar WebSockets con clientes de escucha HTTP y HTTPS.

### <a name="custom-error-pages"></a>Páginas de error personalizadas

Puede definir el error personalizado en el nivel global o en el nivel del cliente de escucha. Actualmente no se admite la creación de páginas de error personalizadas en el nivel global desde Azure Portal. Puede configurar una página de error personalizada para un error 403 de firewall de aplicaciones web o una página de mantenimiento 502 en el nivel del cliente de escucha. También debe especificar una dirección URL para el blob accesible públicamente para el código de estado de error especificado. Para más información, consulte [Create Application Gateway custom error pages](https://docs.microsoft.com/azure/application-gateway/custom-error) (Creación de páginas de error personalizadas de Application Gateway).

![Códigos de error de Application Gateway](https://docs.microsoft.com/azure/application-gateway/media/custom-error/ag-error-codes.png)

Para configurar una página de error personalizada global, consulte [Configuración de Azure PowerShell](https://docs.microsoft.com/azure/application-gateway/custom-error#azure-powershell-configuration).

### <a name="ssl-policy"></a>Directiva SSL

Puede centralizar la administración de certificados SSL y reducir la sobrecarga de cifrado y descifrado de una granja de servidores back-end. El control centralizado de SSL también le permite especificar una directiva SSL central que se adapte a los requisitos de seguridad. Puede elegir la directiva SSL *predeterminada*, *predefinida* o *personalizada*.

Puede configurar la directiva SSL para controlar las versiones del protocolo SSL. Puede configurar una puerta de enlace de aplicaciones para denegar TLS1.0, TLS1.1 y TLS1.2. De forma predeterminada, SSL 2.0 y 3.0 están deshabilitadas y no se pueden configurar. Para más información, consulte [Introducción a la directiva SSL de Application Gateway](https://docs.microsoft.com/azure/application-gateway/application-gateway-ssl-policy-overview).

Después de crear un cliente de escucha, puede asociarlo a una regla de enrutamiento de solicitud. Esta regla determina cómo se enrutan las solicitudes que se reciben en el cliente de escucha hacia el back-end.

## <a name="request-routing-rules"></a>Reglas de enrutamiento de solicitudes

Si crea una puerta de enlace de aplicaciones mediante Azure Portal, puede crear una regla predeterminada (*rule1*). Esta regla enlaza el cliente de escucha predeterminado (*appGatewayHttpListener*) con el grupo de servidores back-end predeterminado (*appGatewayBackendPool*) y la configuración de HTTP de back-end predeterminada (*appGatewayBackendHttpSettings*). Después de crear la puerta de enlace, puede modificar la configuración de la regla predeterminada o crear nuevas reglas.

### <a name="rule-type"></a>Tipo de regla

Cuando se crea una regla, puede elegir entre [*básica* y *basada en ruta de acceso*](https://docs.microsoft.com/azure/application-gateway/application-gateway-components#request-routing-rules).

- Elija básica si desea reenviar todas las solicitudes del cliente de escucha asociado (por ejemplo, *blog<i></i>.contoso.com/\*)* a un único grupo de servidores back-end.
- Elija basada en ruta de acceso si desea enrutar las solicitudes desde determinadas rutas de acceso URL a grupos de servidores back-end específicos. El patrón basado en rutas de acceso se aplica solo a la ruta de acceso de la dirección URL, no a sus parámetros de consulta.

#### <a name="order-of-processing-rules"></a>Orden de procesamiento de las reglas

Para la SKU v1, la coincidencia de patrones de las solicitudes entrantes se procesa en el orden en que se enumeran las rutas en la asignación de rutas de acceso de la dirección URL de la regla basada en ruta de acceso. Si una solicitud coincide con el patrón en dos o más rutas de la asignación de rutas, se selecciona la primera ruta de la lista. Y la solicitud se reenvía al back-end que está asociado con esa ruta de acceso.

En el caso de la SKU v2, una coincidencia exacta tiene mayor prioridad que el orden de las rutas en la asignación de rutas de acceso URL. Si una solicitud coincide con el patrón en dos o más rutas de acceso, la solicitud se reenvía al back-end que está asociado con la ruta de acceso que coincide exactamente con la solicitud. Si la ruta de acceso de la solicitud entrante no coincide exactamente con ninguna ruta de acceso de la asignación, la coincidencia de patrones de la solicitud se procesa en la lista de pedidos de asignación de rutas de acceso de la regla basada en ruta de acceso.

### <a name="associated-listener"></a>Cliente de escucha asociado

Asocie un cliente de escucha a la regla para que la *regla de enrutamiento de solicitudes* que está asociada con él realice una evaluación para determinar el grupo de servidores back-end al que enrutar la solicitud.

### <a name="associated-back-end-pool"></a>Grupo de servidores back-end asociado

Asocie la regla al grupo de servidores back-end que contiene los back-end de destino que se encargan de las solicitudes que recibe el cliente de escucha.

 - En una regla básica, se permite un único grupo de servidores back-end. Todas las solicitudes del cliente de escucha asociado se reenviarán a ese grupo de servidores back-end.

 - En el caso de una regla basada en ruta de acceso, agregue varios grupos de servidores back-end que se correspondan con cada ruta de acceso URL. Aquellas solicitudes que coincidan con la ruta de acceso especificada se reenviarán al grupo de servidores back-end correspondiente. Agregue también un grupo de servidores back-end predeterminado. Las solicitudes que no coincidan con ninguna ruta de acceso URL de la regla se reenviarán a ese grupo.

### <a name="associated-back-end-http-setting"></a>Configuración de HTTP del back-end asociado

Agregue una configuración de HTTP de back-end para cada regla. Las solicitudes se enrutarán desde la puerta de enlace de aplicaciones a los back-end de destino mediante el número de puerto, el protocolo y cualquier otra información que se especifique en esta configuración.

En una regla básica, se permite solo una configuración de HTTP de back-end. Todas las solicitudes del cliente de escucha asociado se reenviarán a los back-end de destino correspondientes mediante esta configuración de HTTP.

Agregue una configuración de HTTP de back-end para cada regla. Las solicitudes se enrutarán desde la puerta de enlace de aplicaciones a los back-end de destino mediante el número de puerto, el protocolo y cualquier otra información que se especifique en esta configuración.

En una regla básica, se permite solo una configuración de HTTP de back-end. Todas las solicitudes del cliente de escucha asociado se reenviarán a los back-end de destino correspondientes mediante esta configuración de HTTP.

En el caso de una regla basada en ruta de acceso, agregue varias configuraciones HTTP de back-end que se correspondan con cada ruta de acceso URL. Las solicitudes que coincidan con la ruta de acceso URL de esta configuración se reenviarán a los back-end de destino correspondientes mediante la configuración de HTTP que se corresponda con cada ruta de acceso URL. Agregue también una configuración de HTTP predeterminada. Las solicitudes que no coincidan con ninguna ruta de acceso URL de esta regla se reenviarán al grupo de servidores back-end predeterminado mediante la configuración de HTTP predeterminada.

### <a name="redirection-setting"></a>Configuración de redireccionamiento

Si se configura el redireccionamiento para una regla básica, todas las solicitudes del cliente de escucha asociado se reenviarán al destino. Esto es un redireccionamiento *global*. Si se configura el redireccionamiento para una regla basada en ruta de acceso, solo se redirigirán aquellas solicitudes de una determinada área del sitio. Un ejemplo es un área de carro de la compra que se indica mediante */cart/\** . Esto es un redireccionamiento *basado en ruta de acceso*.

Para más información acerca de los redireccionamientos, consulte [Introducción a la redirección de Application Gateway](https://docs.microsoft.com/azure/application-gateway/redirect-overview).

#### <a name="redirection-type"></a>Tipo de redireccionamiento

Elija el tipo de redireccionamiento necesario: *Permanente (301)* , *Temporal (307)* , *Encontrado (302)* o *Ver otro (303)* .

#### <a name="redirection-target"></a>Destino del redireccionamiento

Elija otro cliente de escucha o un sitio externo como destino del redireccionamiento.

##### <a name="listener"></a>Agente de escucha

Elige el cliente de escucha como destino de redireccionamiento para redirigir el tráfico desde un cliente de escucha a otro en la puerta de enlace. Esta configuración es necesaria si se desea habilitar el redireccionamiento de HTTP a HTTPS. Redirige el tráfico desde el cliente de escucha de origen que comprueba las solicitudes HTTP entrantes al cliente de escucha de destino que comprueba las solicitudes HTTPS entrantes. También puede decidir incluir la cadena de consulta y la ruta de acceso de la solicitud original en la solicitud que se reenviará al destino de redireccionamiento.

![Cuadro de diálogo de componentes de Application Gateway](./media/configuration-overview/configure-redirection.png)

Para más información sobre el redireccionamiento de HTTP a HTTPS, consulte:
- [HTTP-to-HTTPS redirection by using the Azure portal](https://docs.microsoft.com/azure/application-gateway/redirect-http-to-https-portal) (Redireccionamiento de HTTP a HTTPS mediante Azure Portal)
- [HTTP-to-HTTPS redirection by using PowerShell](https://docs.microsoft.com/azure/application-gateway/redirect-http-to-https-powershell) (Redireccionamiento de HTTP a HTTPS mediante PowerShell)
- [HTTP-to-HTTPS redirection by using the Azure CLI](https://docs.microsoft.com/azure/application-gateway/redirect-http-to-https-cli) (Redireccionamiento de HTTP a HTTPS mediante la CLI de Azure)

##### <a name="external-site"></a>Sitio externo

Elija un sitio externo cuando desee redirigir el tráfico del cliente de escucha asociado con esta regla a un sitio externo. Puede decidir incluir la cadena de consulta de la solicitud original en la solicitud que se reenviará al destino de redireccionamiento. No se puede reenviar la ruta de acceso al sitio externo que se encontraba en la solicitud original.

Para más información sobre el redireccionamiento, consulte:
- [Redirect traffic to an external site by using PowerShell](https://docs.microsoft.com/azure/application-gateway/redirect-external-site-powershell) (Redireccionamiento del tráfico a un sitio externo mediante PowerShell)
- [Redirect traffic to an external site by using the CLI](https://docs.microsoft.com/azure/application-gateway/redirect-external-site-cli) (Redireccionamiento del tráfico a un sitio externo mediante la CLI)

#### <a name="rewrite-the-http-header-setting"></a>Reescritura del valor del encabezado HTTP

Este valor permite agregar, quitar o actualizar los encabezados de respuesta y de solicitudes HTTP, mientras los paquetes de solicitudes y respuestas se mueven entre el cliente y los grupos de servidores back-end. Solo se puede configurar esta funcionalidad mediante PowerShell. La compatibilidad con Azure Portal y la CLI aún no está disponible. Para más información, consulte:

 - [Introducción a la reescritura de encabezados HTTP](https://docs.microsoft.com/azure/application-gateway/rewrite-http-headers)
 - [Configuración de la reescritura de encabezados HTTP](https://docs.microsoft.com/azure/application-gateway/add-http-header-rewrite-rule-powershell#specify-the-http-header-rewrite-rule-configuration)

## <a name="http-settings"></a>Configuración de HTTP

La puerta de enlace de aplicaciones enrutará el tráfico a los servidores back-end mediante la configuración que especifique aquí. Después de crear una configuración de HTTP, debe asociarla con una o varias reglas de enrutamiento de solicitudes.

### <a name="cookie-based-affinity"></a>Afinidad basada en cookies

Esta característica es útil cuando se quiere mantener una sesión de usuario en el mismo servidor. Las cookies administradas por la puerta de enlace permiten a esta dirigir el tráfico posterior de una sesión de usuario hasta el mismo servidor para su procesamiento. Esto es importante en aquellos casos en los que se guarda el estado de la sesión de forma local en el servidor para una sesión de usuario. Si la aplicación no puede administrar la afinidad basada en cookies, no podrá usar esta característica. Para poder utilizarla, asegúrese de que los clientes admiten cookies.

### <a name="connection-draining"></a>Purga de la conexión

La purga de conexión ayuda a la correcta eliminación de miembros del grupo de servidores back-end durante las actualizaciones de servicio planeadas. Puede aplicar esta configuración a todos los miembros de un grupo de servidores back-end durante la creación de reglas. Se asegura de que todas las instancias de anulación de registro de un grupo de servidores back-end no reciben solicitudes nuevas. Mientras tanto, a las solicitudes existentes se les permite finalizar dentro de un límite de tiempo que se puede configurar. La purga de conexión se aplica a instancias de back-end que se eliminan explícitamente del grupo de servidores back-end mediante una llamada API. También se aplica a las instancias de back-end que el sondeo de estado notifica como *incorrectas*.

### <a name="protocol"></a>Protocolo

Application Gateway admite HTTP y HTTPS para las solicitudes de enrutamiento a los servidores back-end. Si elige HTTP, el tráfico a los servidores back-end no estará cifrado. Si la comunicación sin cifrar no es aceptable, seleccione HTTPS.

Esta configuración combinada con HTTPS en el cliente de escucha admite [SSL de un extremo a otro](https://docs.microsoft.com/azure/application-gateway/ssl-overview). Esto le permite transmitir de forma segura información confidencial cifrada al back-end. Cada servidor back-end del grupo con SSL de un extremo a otro habilitada debe configurarse con un certificado para permitir la comunicación segura.

### <a name="port"></a>Port

Este valor especifica el puerto en el que escuchan los servidores back-end el tráfico de la puerta de enlace de aplicaciones. Puede configurar los puertos comprendidos entre 1 y 65535.

### <a name="request-timeout"></a>Tiempo de espera de solicitud

Este valor es el número de segundos que la puerta de enlace de aplicaciones espera para recibir una respuesta del grupo de servidores back-end antes de devolver el mensaje de error "Se agotó el tiempo de espera de la conexión".

### <a name="override-back-end-path"></a>Ruta de acceso de back-end de sustitución

Este valor le permite configurar una ruta de reenvío personalizada opcional que se usará cuando la solicitud se reenvíe al servidor back-end. Cualquier parte de la ruta de acceso entrante que coincida con la ruta de acceso personalizada del campo **ruta de acceso de back-end de sustitución** se copiará en la ruta de acceso reenviada. En la tabla siguiente se muestra cómo funciona esta característica:

- Cuando la configuración de HTTP se asocia a una regla básica de enrutamiento de solicitudes:

  | Solicitud original  | Ruta de acceso de back-end de sustitución | Solicitud que se reenvía al back-end |
  | ----------------- | --------------------- | ---------------------------- |
  | /home/            | /override/            | /override/home/              |
  | /home/secondhome/ | /override/            | /override/home/secondhome/   |

- Cuando la configuración de HTTP se asocia a una regla de enrutamiento de solicitudes basada en una ruta de acceso:

  | Solicitud original           | Regla de la ruta       | Ruta de acceso de back-end de sustitución | Solicitud que se reenvía al back-end |
  | -------------------------- | --------------- | --------------------- | ---------------------------- |
  | /pathrule/home/            | /pathrule*      | /override/            | /override/home/              |
  | /pathrule/home/secondhome/ | /pathrule*      | /override/            | /override/home/secondhome/   |
  | /home/                     | /pathrule*      | /override/            | /override/home/              |
  | /home/secondhome/          | /pathrule*      | /override/            | /override/home/secondhome/   |
  | /pathrule/home/            | /pathrule/home* | /override/            | /override/                   |
  | /pathrule/home/secondhome/ | /pathrule/home* | /override/            | /override/secondhome/        |
  | /pathrule/                 | /pathrule/      | /override/            | /override/                   |

### <a name="use-for-app-service"></a>Uso para App Service

Se trata de un acceso directo de la interfaz de usuario que selecciona los dos valores necesarios para el back-end de Azure App Service. Habilita la opción **Seleccionar nombre de host de la dirección de back-end** y crea un nuevo sondeo personalizado. (Para más información, consulte la sección dedicada a la opción [Seleccionar nombre de host de la dirección de back-end](#pick) de este artículo). Se crea un nuevo sondeo y se selecciona el encabezado de este a partir de la dirección de un miembro del back-end.

### <a name="use-custom-probe"></a>Usar sondeo personalizado

Esta opción asocia un [sondeo personalizado](https://docs.microsoft.com/azure/application-gateway/application-gateway-probe-overview#custom-health-probe) con una configuración de HTTP. Solo puede asociar un sondeo personalizado con una configuración de HTTP. Si no asocia explícitamente un sondeo personalizado, se empleará el [sondeo predeterminado](https://docs.microsoft.com/azure/application-gateway/application-gateway-probe-overview#default-health-probe-settings) para supervisar el estado del back-end. Es recomendable crear un sondeo personalizado para un mayor control sobre la supervisión del estado de los servidores back-end.

> [!NOTE]
> El sondeo personalizado no supervisa el estado del grupo de servidores back-end a menos que la configuración de HTTP correspondiente esté explícitamente asociada a un cliente de escucha.

### <a id="pick"/></a>Seleccionar nombre de host de la dirección de back-end

Esta funcionalidad establece dinámicamente el encabezado *host* de la solicitud en el nombre de host del grupo de servidores back-end. Usa una dirección IP o FQDN.

Esta característica resulta útil cuando el nombre de dominio del back-end es diferente del nombre DNS de la puerta de enlace de aplicaciones y el back-end se basa en un encabezado de host específico o extensión de Indicación de nombre de servidor (SNI) para resolver en el punto de conexión correcto.

Un ejemplo podría ser el uso de servicios multiinquilino como back-end. Una instancia de App Service es un servicio multiinquilino que usa un espacio compartido con una sola dirección IP. Por lo tanto, solo se puede acceder a una instancia de App Service mediante los nombres de host que se establecen en la configuración del dominio personalizado.

De forma predeterminada, el nombre de dominio personalizado es *example.azurewebsites.<i></i>net*. Para acceder a la instancia de App Service con una puerta de enlace de aplicaciones mediante un nombre de host que no está explícitamente registrado en dicha instancia o mediante el FQDN de la puerta de enlace de aplicaciones, debe reemplazar el nombre de host de la solicitud original por el nombre de host de la instancia de App Service. Para ello, habilite la opción **Seleccionar nombre de host de la dirección de back-end**.

Para un dominio personalizado cuyo nombre DNS personalizado ya existente está asignado a la instancia de App Service, no es necesario que habilite esta opción.

> [!NOTE]
> Esta opción no es necesaria para App Service Environment para PowerApps, ya que es una implementación dedicada.

### <a name="host-name-override"></a>Sustitución del nombre de host

Esta funcionalidad reemplaza el encabezado *host* de la solicitud entrante en la puerta de enlace de aplicaciones por el nombre de host que especifique.

Por ejemplo, si se especifica *www.contoso<i></i>.com* en la opción **Nombre de host**, la solicitud original *https:/<i></i>/appgw.eastus.cloudapp.net/path1* cambia a *https:/<i></i>/www.contoso.com/path1* cuando la solicitud se reenvía al servidor back-end.

## <a name="back-end-pool"></a>Grupo de servidores back-end

Puede apuntar un grupo de servidores back-end a cuatro tipos de miembros de back-end: una máquina virtual específica, un conjunto de escalado de máquinas virtuales, una dirección IP o FQDN, o una instancia de App Service. Cada grupo de servidores back-end puede apuntar a varios miembros del mismo tipo. No se admite el apuntar a miembros de diferentes tipos en el mismo grupo de servidores back-end.

Después de crear un grupo de servidores back-end, debe asociarlo con una o varias reglas de enrutamiento de solicitudes. También debe configurar sondeos de estado para cada grupo de servidores back-end de la puerta de enlace de aplicaciones. Cuando se cumple una condición de la regla de enrutamiento de solicitudes, la puerta de enlace de aplicaciones reenvía el tráfico a los servidores correctos (los que determinen los sondeos de estado) del grupo de back-end correspondiente.

## <a name="health-probes"></a>Sondeos de estado

Una puerta de enlace de aplicaciones supervisa el estado de todos los recursos en su back-end de forma predeterminada. Pero se recomienda encarecidamente que cree un sondeo personalizado para cada configuración de HTTP de back-end para conseguir un mayor control sobre la supervisión del estado. Para más información sobre cómo configurar un sondeo personalizado, consulte [Configuración de sondeo de estado personalizado](https://docs.microsoft.com/azure/application-gateway/application-gateway-probe-overview#custom-health-probe-settings).

> [!NOTE]
> Después de crear un sondeo de estado personalizado, debe asociarlo a una configuración de HTTP de back-end. Un sondeo personalizado no supervisará el estado del grupo de servidores back-end a menos que la configuración de HTTP correspondiente esté explícitamente asociada a un cliente de escucha.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ya conoce los componentes de Application Gateway, puede:

- [Crear una puerta de enlace de aplicaciones mediante Azure Portal](quick-create-portal.md)
- [Crear una puerta de enlace de aplicaciones mediante PowerShell](quick-create-powershell.md)
- [Crear una puerta de enlace de aplicaciones mediante la CLI de Azure](quick-create-cli.md)
