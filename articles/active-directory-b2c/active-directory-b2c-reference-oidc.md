---
title: 'Inicio de sesión web con OpenID Connect: Azure Active Directory B2C'
description: Creación de aplicaciones web mediante el protocolo de autenticación OpenID Connect de Azure Active Directory B2C.
services: active-directory-b2c
author: mmacy
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 08/22/2019
ms.author: marsma
ms.subservice: B2C
ms.custom: fasttrack-edit
ms.openlocfilehash: 35abb84f92ed9a7295c45afc69b673a3be46be15
ms.sourcegitcommit: b3bad696c2b776d018d9f06b6e27bffaa3c0d9c3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2019
ms.locfileid: "69874128"
---
# <a name="web-sign-in-with-openid-connect-in-azure-active-directory-b2c"></a>Inicio de sesión web con OpenID Connect en Azure Active Directory B2C

OpenID Connect es un protocolo de autenticación basado en OAuth 2.0 que se puede usar para que los usuarios inicien sesión de forma segura en las aplicaciones web. Con la implementación de OpenID Connect de Azure Active Directory B2C (Azure AD B2C) puede externalizar el registro, el inicio de sesión y otras experiencias de administración de identidades en sus aplicaciones web a Azure Active Directory (Azure AD). Esta guía le enseñará cómo hacerlo de manera independiente del lenguaje. En ella se describe cómo enviar y recibir mensajes HTTP sin utilizar ninguna de nuestras bibliotecas de código abierto.

[OpenID Connect](https://openid.net/specs/openid-connect-core-1_0.html) amplía el protocolo de *autorización* de OAuth 2.0 para su uso como protocolo de *autenticación*. Este protocolo de autenticación permite realizar el inicio de sesión único. Presenta el concepto de un *token de identificador*, que permite al cliente comprobar la identidad del usuario y obtener información del perfil básica sobre el usuario.

Al ampliar OAuth 2.0, también permite que las aplicaciones adquieran *tokens de acceso* de forma segura. Se puede usar access_tokens para acceder a los recursos protegidos por un [servidor de autorización](active-directory-b2c-reference-protocols.md). Se recomienda OpenID Connect si va a crear una aplicación web que esté hospedada en un servidor y a la que se accede mediante un explorador. Si desea agregar administración de identidades a sus aplicaciones móviles o de escritorio con Azure AD B2C, debe usar [OAuth 2.0](active-directory-b2c-reference-oauth-code.md), en lugar de OpenID Connect. Para más información sobre los tokens, consulte [Información general de tokens de Azure Active Directory B2C](active-directory-b2c-reference-tokens.md).

Azure AD B2C extiende el protocolo OpenID Connect estándar para realizar algo más que una autorización y autenticación simples. Presenta el [parámetro de flujo de usuario](active-directory-b2c-reference-policies.md), que le permite usar OpenID Connect para agregar experiencias de usuario (como registro, inicio de sesión y administración de perfiles) a su aplicación.

## <a name="send-authentication-requests"></a>Envío de solicitudes de autenticación

Cuando su aplicación web necesite autenticar al usuario y ejecutar el flujo de usuario, puede dirigir al usuario al punto de conexión `/authorize`. El usuario realiza una acción en función del flujo de usuario.

En esta solicitud, el cliente indica los permisos que necesita adquirir del usuario en el parámetro `scope` y especifica el flujo de usuario que se va a ejecutar. Para hacerse una idea de cómo funciona la solicitud, intente pegarla en un explorador y ejecutarla. Reemplace `{tenant}` por el nombre del inquilino. Reemplace `90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6` por el id. de la aplicación que registró previamente en el inquilino. Cambie también el nombre de directiva (`{policy}`) por el que tiene en su inquilino, por ejemplo `b2c_1_sign_in`.

```HTTP
GET https://{tenant}.b2clogin.com/{tenant}.onmicrosoft.com/{policy}/oauth2/v2.0/authorize?
client_id=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6
&response_type=code+id_token
&redirect_uri=https%3A%2F%2Faadb2cplayground.azurewebsites.net%2F
&response_mode=form_post
&scope=openid%20offline_access
&state=arbitrary_data_you_can_receive_in_the_response
&nonce=12345
```

| Parámetro | Obligatorio | DESCRIPCIÓN |
| --------- | -------- | ----------- |
| {tenant} | Sí | Nombre de su inquilino de Azure AD B2C. |
| {policy} | Sí | Flujo de usuario que se va a ejecutar. Especifique el nombre del flujo de usuario que creó en el inquilino de Azure AD B2C. Por ejemplo: `b2c_1_sign_in`, `b2c_1_sign_up` o `b2c_1_edit_profile`. |
| client_id | Sí | Identificador de aplicación que [Azure Portal](https://portal.azure.com/) asignó a la aplicación. |
| valor de seguridad | Sí | Un valor incluido en la solicitud, generada por la aplicación, que se incluirá en el token de identificador resultante como una notificación. La aplicación puede comprobar este valor para mitigar los ataques de reproducción de token. Normalmente, el valor es una cadena única aleatoria que se puede usar para identificar el origen de la solicitud. |
| response_type | Sí | Debe incluir un token de identificador para OpenID Connect. Si su aplicación web también necesita tokens para llamar a una API web, puede usar `code+id_token`. |
| scope | Sí | Una lista de ámbitos separada por espacios. El ámbito `openid` indica un permiso para iniciar sesión con el usuario y obtener los datos del usuario en forma de tokens de identificador. El ámbito `offline_access` es opcional para las aplicaciones web. Indica que la aplicación necesita un *token de actualización* para un acceso ampliado a los recursos. |
| símbolo del sistema | Sin | El tipo de interacción con el usuario que se necesita. El único valor válido en este momento es `login`, que obliga al usuario a escribir sus credenciales en esa solicitud. |
| redirect_uri | Sin | El parámetro `redirect_uri` de su aplicación, donde su aplicación puede enviar y recibir las respuestas de autenticación. Debe coincidir exactamente con uno de los parámetros `redirect_uri` que registró en Azure Portal, con la excepción de que debe estar codificado como URL. |
| response_mode | Sin | El método que se usa para devolver el código de autorización resultante a la aplicación. Puede ser `query`, `form_post` o `fragment`.  Para mayor seguridad se recomienda el modo de respuesta `form_post`. |
| state | Sin | Un valor incluido en la solicitud que también se devolverá en la respuesta del token. Puede ser una cadena de cualquier contenido que desee. Se utiliza normalmente un valor único generado de forma aleatoria para evitar los ataques de falsificación de solicitudes entre sitios. El estado también se usa para codificar información sobre el estado del usuario en la aplicación antes de que se haya producido la solicitud de autenticación, por ejemplo, la página en la que estaban. |

En este punto se pedirá al usuario que complete el flujo de trabajo. Puede que el usuario tenga que escribir su nombre de usuario y contraseña, iniciar sesión con una identidad social o registrarse en el directorio. Podría haber cualquier otro número de pasos en función de cómo se define el flujo de usuario.

Una vez que el usuario haya completado el flujo de usuario, se devuelve una respuesta a la aplicación en el parámetro `redirect_uri` indicado, mediante el método que se especifica en el parámetro `response_mode`. La respuesta es la misma para cada uno de los casos anteriores, con independencia del flujo de usuario.

Una respuesta correcta al usar `response_mode=fragment` tiene el siguiente aspecto:

```HTTP
GET https://aadb2cplayground.azurewebsites.net/#
id_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...
&code=AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrq...
&state=arbitrary_data_you_can_receive_in_the_response
```

| Parámetro | DESCRIPCIÓN |
| --------- | ----------- |
| ID_token | El token de identificador que solicitó la aplicación. Puede usar el token de identificador para comprobar la identidad del usuario y comenzar una sesión con el usuario. |
| código | El código de autorización que solicitó la aplicación, si usó `response_type=code+id_token`. La aplicación puede utilizar el código de autorización con el fin de solicitar un token de acceso para un recurso de destino. Los códigos de autorización normalmente expiran después de unos 10 minutos. |
| state | Si un parámetro `state` está incluido en la solicitud, debería aparecer el mismo valor en la respuesta. La aplicación debe comprobar que los valores `state` de la solicitud y de la respuesta sean idénticos. |

Las respuestas de error también se pueden enviar al parámetro `redirect_uri` para que la aplicación pueda controlarlas de manera adecuada:

```HTTP
GET https://aadb2cplayground.azurewebsites.net/#
error=access_denied
&error_description=the+user+canceled+the+authentication
&state=arbitrary_data_you_can_receive_in_the_response
```

| Parámetro | DESCRIPCIÓN |
| --------- | ----------- |
| error | Un código que puede usarse para clasificar los tipos de errores que se producen. |
| error_description | Un mensaje de error específico que puede ayudarlo a identificar la causa raíz de un error de autenticación. |
| state | Si un parámetro `state` está incluido en la solicitud, debería aparecer el mismo valor en la respuesta. La aplicación debe comprobar que los valores `state` de la solicitud y de la respuesta sean idénticos. |

## <a name="validate-the-id-token"></a>Validar el token de identificador

Recibir un token de identificador no es suficiente para autenticar al usuario. Debe validar la firma del token de identificador y comprobar las notificaciones en el token en función de los requisitos de la aplicación. Azure AD B2C usa [tokens web JSON (JWT)](https://self-issued.info/docs/draft-ietf-oauth-json-web-token.html) y la criptografía de clave pública para firmar los tokens y comprobar que son válidos. Hay muchas bibliotecas de código abierto disponibles para validar los JWT, según el lenguaje de preferencia. Se recomienda explorar esas opciones en lugar de implementar su propia lógica de validación.

Azure AD B2C tiene un extremo de metadatos OpenID Connect, que permite a una aplicación obtener información sobre Azure AD B2C en tiempo de ejecución. En esta información se incluyen los extremos, los contenidos del token y las claves de firma de los token. Hay un documento de metadatos JSON para cada flujo de usuario en su inquilino B2C. Por ejemplo, el documento de metadatos del flujo de usuario `b2c_1_sign_in` en `fabrikamb2c.onmicrosoft.com` se encuentra en:

```HTTP
https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/b2c_1_sign_in/v2.0/.well-known/openid-configuration
```

Una de las propiedades de este documento de configuración es `jwks_uri`, cuyo valor para el mismo flujo de usuario sería:

```HTTP
https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/b2c_1_sign_in/discovery/v2.0/keys
```

Para determinar qué flujo de usuario se usó en la firma de un token de identificador (y de dónde se deben obtener los metadatos), tiene dos opciones. En primer lugar, el nombre del flujo de usuario se incluye en la notificación `acr` del token de identificador. La otra opción consiste en codificar el flujo de usuario en el valor del parámetro `state` al emitir la solicitud y luego descodificarlo para determinar qué flujo de usuario se ha usado. Cualquiera de estos métodos es válido.

Cuando haya adquirido el documento de metadatos del punto de conexión de metadatos de OpenID Connect, podrá usar las claves públicas RSA-256 para validar la firma del token de identificador. Podría haber varias claves enumeradas en este punto de conexión, cada una identificada con una `kid`. El encabezado del token de identificador también contiene una notificación `kid`, que indica cuál de estas claves se usó para firmar el token de identificador.

Para comprobar los tokens de Azure AD B2C, debe generar la clave pública mediante el exponente(e) y el módulo(n). Debe determinar cómo hacerlo con el lenguaje de programación respectivo, según corresponda. La documentación oficial sobre la generación de claves públicas con el protocolo RSA se puede encontrar aquí: https://tools.ietf.org/html/rfc3447#section-3.1.

Cuando haya validado la firma del token de identificador, deberá verificar varias notificaciones. Por ejemplo:

- Valide la notificación `nonce` para evitar ataques de reproducción del token. Su valor debería ser el que especificó en la solicitud de inicio de sesión.
- Valide la notificación `aud` para asegurarse de que se ha generado el token de identificador para su aplicación. Su valor debería ser el identificador de la aplicación.
- Valide las notificaciones `iat` y `exp` para asegurarse de que el token de identificador no ha expirado.

También hay otras validaciones que deberá llevar a cabo. Las validaciones se describen en detalle en la [especificación de OpenID Connect Core](https://openid.net/specs/openid-connect-core-1_0.html). Podría también querer validar notificaciones adicionales, en función de su escenario. Algunas validaciones comunes incluyen:

- Asegurarse de que la organización o el usuario se han registrado en la aplicación.
- Asegurarse de que el usuario tiene la autorización o los privilegios adecuados.
- Asegurarse de que se haya producido un determinado nivel de autenticación, como Azure Multi-Factor Authentication.

Cuando haya validado el token de identificador, podrá iniciar una sesión con el usuario. Puede usar las notificaciones del token de identificador para obtener información sobre el usuario en la aplicación. Entre los usos de esta información se incluye la visualización, los registros y la autorización.

## <a name="get-a-token"></a>Obtención de un token

Si necesita que la aplicación web solo ejecute flujos de usuario, puede omitir las siguientes secciones. Estas secciones son aplicables solo a las aplicaciones web que necesitan efectuar llamadas autenticadas a una API web y que también están protegidas por Azure AD B2C.

Puede canjear el código de autorización que adquirió (mediante `response_type=code+id_token`) por un token para el recurso deseado; para ello, debe enviar una solicitud `POST` al punto de conexión `/token`. En Azure AD B2C, puede [solicitar tokens de acceso para otras API](active-directory-b2c-access-tokens.md#request-a-token) como de costumbre especificando sus ámbitos en la solicitud.

También puede solicitar un token de acceso para su propia API web de back-end de la aplicación mediante la convención del uso del identificador del cliente de la aplicación como el ámbito solicitado (lo que resultará en un token de acceso con ese identificador de cliente como el "público"):

```HTTP
POST {tenant}.onmicrosoft.com/{policy}/oauth2/v2.0/token HTTP/1.1
Host: {tenant}.b2clogin.com
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&client_id=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6&scope=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6 offline_access&code=AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrq...&redirect_uri=urn:ietf:wg:oauth:2.0:oob&client_secret=<your-application-secret>
```

| Parámetro | Obligatorio | DESCRIPCIÓN |
| --------- | -------- | ----------- |
| {tenant} | Sí | Nombre de su inquilino de Azure AD B2C. |
| {policy} | Sí | El flujo de usuario usado para adquirir el código de autorización. No puede usar un flujo de usuario diferente en esta solicitud. Agregue este parámetro a la cadena de consulta, no al cuerpo de POST. |
| client_id | Sí | Identificador de aplicación que [Azure Portal](https://portal.azure.com/) asignó a la aplicación. |
| client_secret | Sí | El secreto de la aplicación se generó en [Azure Portal](https://portal.azure.com/). El secreto de aplicación es un artefacto de seguridad importante. Debe almacenarlo de forma segura en el servidor. Cambie este secreto de cliente de forma periódica. |
| código | Sí | El código de autorización que adquirió al principio del flujo de usuario. |
| grant_type | Sí | El tipo de concesión, que debe ser `authorization_code` para el flujo de código de autorización. |
| redirect_uri | Sí | El parámetro `redirect_uri` de la aplicación en la que recibió el código de autorización. |
| scope | Sin | Una lista de ámbitos separada por espacios. El ámbito `openid` indica un permiso para iniciar sesión con el usuario y obtener datos de este en forma de parámetros id_token. Se puede usar para obtener tokens para la propia API web de back-end de la aplicación, representada por el mismo id. de aplicación que el cliente. El ámbito `offline_access` indica que la aplicación necesita un token de actualización para un acceso ampliado a los recursos. |

Una respuesta correcta del token tiene el siguiente aspecto:

```JSON
{
    "not_before": "1442340812",
    "token_type": "Bearer",
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...",
    "scope": "90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6 offline_access",
    "expires_in": "3600",
    "refresh_token": "AAQfQmvuDy8WtUv-sd0TBwWVQs1rC-Lfxa_NDkLqpg50Cxp5Dxj0VPF1mx2Z...",
}
```

| Parámetro | DESCRIPCIÓN |
| --------- | ----------- |
| not_before | Hora a la que el token se considera válido, en tiempo de época. |
| token_type | El valor del tipo de token. `Bearer` es el único tipo admitido. |
| access_token | El token JWT firmado solicitado. |
| scope | Los ámbitos para los que el token es válido. |
| expires_in | El período de validez del token de acceso (en segundos). |
| refresh_token | Un token de actualización de OAuth 2.0. La aplicación puede usar este token para adquirir tokens adicionales una vez que expire el actual. Los tokens de actualización pueden usarse para conservar el acceso a los recursos durante largos periodos. El ámbito `offline_access` debe haberse usado en las solicitudes de token y de autorización para recibir un token de actualización. |

Las respuestas de error tienen un aspecto similar al siguiente:

```JSON
{
    "error": "access_denied",
    "error_description": "The user revoked access to the app.",
}
```

| Parámetro | DESCRIPCIÓN |
| --------- | ----------- |
| error | Un código que puede usarse para clasificar los tipos de errores que se producen. |
| error_description | Un mensaje que puede ayudar a identificar la causa raíz de un error de autenticación. |

## <a name="use-the-token"></a>Uso del token

Ahora que ha adquirido correctamente un token de acceso, puede usar el token en las solicitudes a las API web de back-end incluyéndolo en el encabezado `Authorization`:

```HTTP
GET /tasks
Host: mytaskwebapi.com
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...
```

## <a name="refresh-the-token"></a>Actualización del token

Los tokens de identificador expiran en un breve período de tiempo. Actulice los token cuando expiren para poder seguir accediendo a los recursos. Puede actualizar un token mediante el envío de otra solicitud `POST` al punto de conexión `/token`. Esta vez, proporcione el parámetro `refresh_token` en lugar del parámetro `code`:

```HTTP
POST {tenant}.onmicrosoft.com/{policy}/oauth2/v2.0/token HTTP/1.1
Host: {tenant}.b2clogin.com
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token&client_id=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6&scope=openid offline_access&refresh_token=AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrq...&redirect_uri=urn:ietf:wg:oauth:2.0:oob&client_secret=<your-application-secret>
```

| Parámetro | Obligatorio | DESCRIPCIÓN |
| --------- | -------- | ----------- |
| {tenant} | Sí | Nombre de su inquilino de Azure AD B2C. |
| {policy} | Sí | El flujo de usuario usado para adquirir el token de actualización original. No puede usar un flujo de usuario diferente en esta solicitud. Agregue este parámetro a la cadena de consulta, no al cuerpo de POST. |
| client_id | Sí | Identificador de aplicación que [Azure Portal](https://portal.azure.com/) asignó a la aplicación. |
| client_secret | Sí | El secreto de la aplicación se generó en [Azure Portal](https://portal.azure.com/). El secreto de aplicación es un artefacto de seguridad importante. Debe almacenarlo de forma segura en el servidor. Cambie este secreto de cliente de forma periódica. |
| grant_type | Sí | El tipo de concesión, que debe ser un token de actualización para esta parte del flujo de código de autorización. |
| refresh_token | Sí | El token de actualización original que adquirió en la segunda parte del flujo. El ámbito `offline_access` debe usarse en las solicitudes de token y de autorización para recibir un token de actualización. |
| redirect_uri | Sin | El parámetro `redirect_uri` de la aplicación en la que recibió el código de autorización. |
| scope | Sin | Una lista de ámbitos separada por espacios. El ámbito `openid` indica un permiso para iniciar sesión con el usuario y obtener los datos del usuario en forma de tokens de identificador. Se puede usar para enviar tokens para la propia API web de back-end de la aplicación, representada por el mismo id. de aplicación que el cliente. El ámbito `offline_access` indica que la aplicación necesita un token de actualización para un acceso ampliado a los recursos. |

Una respuesta correcta del token tiene el siguiente aspecto:

```JSON
{
    "not_before": "1442340812",
    "token_type": "Bearer",
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...",
    "scope": "90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6 offline_access",
    "expires_in": "3600",
    "refresh_token": "AAQfQmvuDy8WtUv-sd0TBwWVQs1rC-Lfxa_NDkLqpg50Cxp5Dxj0VPF1mx2Z...",
}
```

| Parámetro | DESCRIPCIÓN |
| --------- | ----------- |
| not_before | Hora a la que el token se considera válido, en tiempo de época. |
| token_type | El valor del tipo de token. `Bearer` es el único tipo admitido. |
| access_token | El token JWT firmado que se solicitó. |
| scope | El ámbito para el que el token es válido. |
| expires_in | El período de validez del token de acceso (en segundos). |
| refresh_token | Un token de actualización de OAuth 2.0. La aplicación puede usar este token para adquirir tokens adicionales una vez que expire el actual. Los tokens de actualización pueden usarse para conservar el acceso a los recursos durante largos periodos. |

Las respuestas de error tienen un aspecto similar al siguiente:

```JSON
{
    "error": "access_denied",
    "error_description": "The user revoked access to the app.",
}
```

| Parámetro | DESCRIPCIÓN |
| --------- | ----------- |
| error | Un código que puede usarse para clasificar los tipos de errores que se producen. |
| error_description | Un mensaje que puede ayudar a identificar la causa raíz de un error de autenticación. |

## <a name="send-a-sign-out-request"></a>Envío de una solicitud de cierre de sesión

Si desea cerrar la sesión del usuario de la aplicación, no basta con borrar las cookies de la aplicación o finalizar la sesión con el usuario. Redireccione al usuario a Azure AD B2C para cerrar la sesión. Si no lo hace, el usuario podría autenticarse de nuevo en su aplicación sin volver a escribir sus credenciales.

Para cerrar la sesión del usuario, redirija al usuario al punto de conexión `end_session` que aparece en el documento de metadatos de OpenID Connect que se ha descrito anteriormente:

```HTTP
GET https://{tenant}.b2clogin.com/{tenant}.onmicrosoft.com/{policy}/oauth2/v2.0/logout?post_logout_redirect_uri=https%3A%2F%2Faadb2cplayground.azurewebsites.net%2F
```

| Parámetro | Obligatorio | DESCRIPCIÓN |
| --------- | -------- | ----------- |
| {tenant} | Sí | Nombre de su inquilino de Azure AD B2C. |
| {policy} | Sí | El flujo de usuario que quiere usar para cerrar la sesión del usuario en la aplicación. |
| id_token_hint| Sin | Token de id. emitido previamente para pasarse al punto de conexión de cierre de sesión como una sugerencia sobre la sesión autenticada actual del usuario final con el cliente. La pista `id_token_hint` garantiza que el `post_logout_redirect_uri` es una dirección URL de respuesta registrada en la configuración de la aplicación de Azure AD B2C. |
| post_logout_redirect_uri | Sin | La dirección URL a la que se debe redirigir al usuario después de un cierre de sesión correcto. Si no se incluye, Azure AD B2C mostrará un mensaje genérico al usuario. A menos que proporcione un valor `id_token_hint`, no debe registrar esta dirección URL como una dirección URL de respuesta en la configuración de la aplicación de Azure AD B2C. |
| state | Sin | Si un parámetro `state` está incluido en la solicitud, debería aparecer el mismo valor en la respuesta. La aplicación debe comprobar que los valores `state` de la solicitud y de la respuesta sean idénticos. |

### <a name="secure-your-logout-redirect"></a>Protección de la redirección de cierre de sesión

Después del cierre de sesión, se redirige al usuario al URI especificado en el parámetro `post_logout_redirect_uri`, independientemente de las direcciones URL de respuesta que se hayan especificado para la aplicación. Sin embargo, si se pasa una pista `id_token_hint` válida, Azure AD B2C comprueba que el valor de `post_logout_redirect_uri` coincida con uno de los URI de redirección configurados de la aplicación antes de realizar la redirección. Si no se configuró ninguna dirección URL de respuesta coincidente para la aplicación, se muestra un mensaje de error y no se redirige al usuario.

### <a name="external-identity-provider-sign-out"></a>Cierre de sesión con un proveedor de identidades externo

El hecho de dirigir al usuario al punto de conexión `end_session` borrará algún estado de inicio de sesión único del usuario con Azure AD B2C, pero no se cerrará la sesión del proveedor de identidades social (IDP) del usuario. Si el usuario selecciona el mismo IDP durante un inicio de sesión posterior, se volverá a autenticar sin especificar sus credenciales. Si un usuario quiere cerrar sesión en su aplicación, eso no significa necesariamente que quiera cerrar sesión en su cuenta de Facebook. Sin embargo, si se usan cuentas locales, la sesión del usuario finalizará correctamente.
