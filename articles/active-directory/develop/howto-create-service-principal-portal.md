---
title: Creación de la identidad de la aplicación de Azure en el portal | Microsoft Docs
description: Describe cómo crear una nueva aplicación de Azure Active Directory y una entidad de servicio que puede utilizarse con el control de acceso basado en rol en Azure Resource Manager para administrar el acceso a los recursos.
services: active-directory
documentationcenter: na
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/17/2019
ms.author: ryanwi
ms.reviewer: tomfitz
ms.custom: aaddev, seoapril2019, identityplatformtop40
ms.collection: M365-identity-device-management
ms.openlocfilehash: a28354f54978e8ba776d8b0da294652ff462a05f
ms.sourcegitcommit: 670c38d85ef97bf236b45850fd4750e3b98c8899
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/08/2019
ms.locfileid: "68853451"
---
# <a name="how-to-use-the-portal-to-create-an-azure-ad-application-and-service-principal-that-can-access-resources"></a>Procedimientos para: Uso del portal para crear una aplicación de Azure AD y una entidad de servicio con acceso a los recursos

En este artículo se muestra cómo crear una nueva entidad de servicio y aplicación de Azure Active Directory (Azure AD) que puede usar con el control de acceso basado en rol. Cuando haya código que deba tener acceso a ciertos recursos o modificarlos, puede crear una identidad para la aplicación. Esta identidad se conoce como una entidad de servicio. Entonces puede asignar los permisos necesarios a la entidad de servicio. En este artículo se muestra cómo usar el portal para crear la entidad de servicio. Se centra en una aplicación de un único inquilino donde la aplicación está diseñada para ejecutarse en una sola organización. Normalmente, utiliza aplicaciones de inquilino único para aplicaciones de línea de negocio que se ejecutan dentro de su organización.

> [!IMPORTANT]
> En lugar de crear una entidad de servicio, considere el uso de identidades administradas para recursos de Azure para la identidad de la aplicación. Si el código se ejecuta en un servicio que admite identidades administradas y tiene acceso a recursos que admiten la autenticación de Azure AD, las identidades administradas son la opción ideal para usted. Para obtener más información sobre las identidades administradas para recursos de Azure, incluidos los servicios que actualmente lo admiten, consulte [¿Qué es Managed Identities for Azure Resources?](../managed-identities-azure-resources/overview.md).

## <a name="create-an-azure-active-directory-application"></a>Creación de una aplicación de Azure Active Directory

Vamos a saltar directamente a la creación de la identidad. Si experimenta un problema, compruebe los [permisos necesarios](#required-permissions) para asegurarse de que su cuenta puede crear la identidad.

1. Inicie sesión en su cuenta de Azure mediante [Azure Portal](https://portal.azure.com).
1. Seleccione **Azure Active Directory**.
1. Seleccione **App registrations** (Registros de aplicaciones).
1. Seleccione **Nuevo registro**.
1. Asigne un nombre a la aplicación. Seleccione un tipo de cuenta compatible, que determinará quién puede usar la aplicación. En **URI de redireccionamiento**, seleccione **Web** para indicar el tipo de aplicación que quiere crear. Escriba el URI al que se envía el token de acceso. No se pueden crear credenciales para una [aplicación nativa](../manage-apps/application-proxy-configure-native-client-application.md). No se puede usar ese tipo en una aplicación automatizada. Después de configurar los valores, seleccione **Registrar**.

   ![Escriba un nombre para la aplicación](./media/howto-create-service-principal-portal/create-app.png)

Ha creado una aplicación de Azure AD y una entidad de servicio.

## <a name="assign-the-application-to-a-role"></a>Asignación de la aplicación a un rol

Para acceder a los recursos de la suscripción, debe asignarle a la aplicación un rol. Decida qué rol ofrece los permisos adecuados para la aplicación. Para obtener más información sobre los roles disponibles, vea [RBAC: Roles integrados](../../role-based-access-control/built-in-roles.md).

Puede establecer el ámbito en el nivel de suscripción, grupo de recursos o recurso. Los permisos se heredan en los niveles inferiores del ámbito. Por ejemplo, el hecho de agregar una aplicación al rol Lector para un grupo de recursos significa que esta puede leer el grupo de recursos y los recursos que contenga.

1. Desplácese hasta el nivel de ámbito al que desea asignar la aplicación. Por ejemplo, para asignar un rol en el ámbito de suscripción, seleccione **Todos los servicios** y **Suscripciones**.

   ![Por ejemplo, asigne un rol en el ámbito de la suscripción](./media/howto-create-service-principal-portal/select-subscription.png)

1. Seleccione la suscripción concreta que se asignará a la aplicación.

   ![Seleccionar suscripción para la asignación](./media/howto-create-service-principal-portal/select-one-subscription.png)

   Si no ve la suscripción que busca, seleccione el **filtro de suscripciones globales**. Asegúrese de que la suscripción que desea está seleccionada para el portal.

1. Seleccione **Access Control (IAM)** .
1. Seleccione **Agregar asignación de roles**.
1. Seleccione el rol que quiere asignar a la aplicación. Para permitir que la aplicación pueda ejecutar acciones como **reiniciar**, **iniciar** y **detener** instancias, debe seleccionar el rol **Colaborador**. De manera predeterminada, las aplicaciones de Azure AD no se muestran en las opciones disponibles. Para encontrar la aplicación, busque el nombre y selecciónelo.

   ![Seleccione el rol que se asignará a la aplicación](./media/howto-create-service-principal-portal/select-role.png)

1. Haga clic en **Guardar** para finalizar la asignación del rol. Verá la aplicación en la lista de usuarios asignados a un rol para ese ámbito.

La entidad de servicio está configurada. Puede empezar a usarla para ejecutar aplicaciones o scripts. En la sección siguiente se muestra cómo obtener valores necesarios al iniciar sesión mediante programación.

## <a name="get-values-for-signing-in"></a>Obtención de valores para iniciar sesión

Al iniciar sesión mediante programación, deberá pasar el id. de inquilino con la solicitud de autenticación. También necesitará el identificador de la aplicación y una clave de autenticación. Para obtener estos valores, use los pasos siguientes:

1. Seleccione **Azure Active Directory**.
1. En **Registros de aplicaciones**, en Azure AD, seleccione su aplicación.
1. Copie el identificador del directorio (inquilino) y almacénelo en el código de la aplicación.

    ![Copie directorio (identificador del inquilino) y almacénelo en el código de la aplicación](./media/howto-create-service-principal-portal/copy-tenant-id.png)

1. Copie el **id. de aplicación** y almacénelo en el código de la aplicación.

   ![Copie el identificador de la aplicación (cliente)](./media/howto-create-service-principal-portal/copy-app-id.png)

## <a name="certificates-and-secrets"></a>Certificados y secretos
Las aplicaciones de demonio pueden usar dos tipos de credenciales para realizar la autenticación con Azure AD: certificados y secretos de aplicación.  Se recomienda usar un certificado, pero también puede crear un secreto de aplicación nuevo.

### <a name="upload-a-certificate"></a>Carga de un certificado

Puede usar un certificado existente si lo tiene.  Si quiere, puede crear un certificado autofirmado con fines de prueba. Abra PowerShell y ejecute [New-SelfSignedCertificate](/powershell/module/pkiclient/new-selfsignedcertificate) con los parámetros siguientes para crear un certificado autofirmado en el almacén de certificados del usuario en su equipo: `$cert=New-SelfSignedCertificate -Subject "CN=DaemonConsoleCert" -CertStoreLocation "Cert:\CurrentUser\My"  -KeyExportPolicy Exportable -KeySpec Signature`.  Exporte este certificado con el complemento MMC [Administrar certificado de usuario](/dotnet/framework/wcf/feature-details/how-to-view-certificates-with-the-mmc-snap-in) accesible desde el Panel de control de Windows.

Para cargar el certificado:

1. Seleccione **Certificados y secretos**.
1. Seleccione **Cargar certificado** y seleccione el certificado (un certificado existente o el certificado autofirmado que exportó).

    ![Seleccione Cargar certificado y seleccione el certificado que quiere agregar](./media/howto-create-service-principal-portal/upload-cert.png)

1. Seleccione **Agregar**.

Después de registrar el certificado con la aplicación en el portal de registro de aplicación, debe habilitar el código de la aplicación cliente para usar el certificado.

### <a name="create-a-new-application-secret"></a>Creación de un secreto de aplicación

Si decide no usar un certificado, puede crear un secreto de aplicación nuevo.

1. Seleccione **Certificados y secretos**.
1. Seleccione **Secretos de cliente -> Nuevo secreto de cliente**.
1. Proporcione una descripción y duración del secreto. Cuando haya terminado, seleccione **Agregar**.

   Después de guardar el secreto de cliente, se muestra el valor del mismo. Copie este valor porque no podrá recuperar la clave más adelante. Debe proporcionar el valor de la clave junto con el identificador de la aplicación para iniciar sesión como la aplicación. Guarde el valor de clave donde la aplicación pueda recuperarlo.

   ![Copie el valor del secreto porque no podrá recuperarlo más adelante](./media/howto-create-service-principal-portal/copy-secret.png)

## <a name="required-permissions"></a>Permisos necesarios

Debe tener permisos suficientes para registrar una aplicación en su inquilino de Azure AD y asignar la aplicación a un rol en su suscripción de Azure.

### <a name="check-azure-ad-permissions"></a>Comprobación de los permisos de Azure AD

1. Seleccione **Azure Active Directory**.
1. Anote su rol. Si tiene el rol **Usuario**, debe asegurarse de que los no administradores pueden registrar aplicaciones.

   ![Busque su rol. Si es usuario, asegúrese de que quienes no son administradores pueden registrar aplicaciones](./media/howto-create-service-principal-portal/view-user-info.png)

1. Seleccione **Configuración de usuario**.
1. Compruebe la configuración de **App registrations** (Registros de aplicaciones). Este valor solo puede configurarlo un administrador. Si se configura en **Sí**, cualquier usuario en el inquilino de Azure Active Directory puede registrar una aplicación.

Si la configuración de registro de aplicaciones se establece en **No**, solo los usuarios con un rol de administrador pueden registrar este tipo de aplicaciones. Consulte los [roles disponibles](../users-groups-roles/directory-assign-admin-roles.md#available-roles) y los [permisos de roles](../users-groups-roles/directory-assign-admin-roles.md#role-permissions) para conocer los roles de administrador disponibles y los permisos específicos en Azure AD que se otorgan a cada rol. Si la cuenta está asignada al rol Usuario, pero la opción Registros de aplicaciones está limitada a los administradores, pida al administrador que le asigne un rol de administrador para poder crear y administrar todos los aspectos de los registros de aplicaciones, o que permita a los usuarios registrar las aplicaciones.

### <a name="check-azure-subscription-permissions"></a>Comprobación de los permisos de suscripción de Azure

En su suscripción de Azure, su cuenta debe tener acceso a `Microsoft.Authorization/*/Write` para asignar una aplicación de AD a un rol. Esta acción se concede mediante el rol [Propietario](../../role-based-access-control/built-in-roles.md#owner) o el rol [Administrador de acceso de usuario](../../role-based-access-control/built-in-roles.md#user-access-administrator). Si su cuenta está asignada al rol **Colaborador**, no tiene los permisos adecuados. Al intentar asignar la entidad de servicio a un rol, se muestra un error.

Para comprobar los permisos de su suscripción:

1. Seleccione la cuenta en la esquina superior derecha y, luego, **... -> Mis permisos**.

   ![Seleccione la cuenta y los permisos de usuario](./media/howto-create-service-principal-portal/select-my-permissions.png)

1. En la lista desplegable, seleccione la suscripción en la que desea crear la entidad de servicio. A continuación, seleccione **Click here to view complete access details for this subscription** (Haga clic aquí para ver los detalles de acceso completos para esta suscripción).

   ![Seleccione la suscripción en la que quiere crear la entidad de servicio](./media/howto-create-service-principal-portal/view-details.png)

1. Seleccione **Asignaciones de roles** para ver los roles asignados y determine si tiene los permisos correspondientes para asignar una aplicación de AD a un rol. En caso contrario, pida al administrador de suscripciones que le agregue al rol Administrador de acceso de usuario. En la siguiente imagen, el usuario está asignado al rol Propietario, lo que significa que el usuario tiene los permisos adecuados.

   ![En este ejemplo se muestra el usuario asignado al rol Propietario](./media/howto-create-service-principal-portal/view-user-role.png)

## <a name="next-steps"></a>Pasos siguientes

* Para configurar una aplicación multiinquilino, consulte [Guía del desarrollador para la autorización con la API de Azure Resource Manager](../../azure-resource-manager/resource-manager-api-authentication.md).
* Para obtener información sobre cómo especificar directivas de seguridad, consulte [Control de acceso basado en roles de Azure](../../role-based-access-control/role-assignments-portal.md).  
* Para obtener una lista de las acciones que puede conceder o denegar a los usuarios, consulte [Operaciones del proveedor de recursos de Azure Resource Manager](../../role-based-access-control/resource-provider-operations.md).
