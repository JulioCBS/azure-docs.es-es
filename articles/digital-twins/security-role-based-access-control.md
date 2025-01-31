---
title: Control de acceso basado en rol de Azure Digital Twins | Microsoft Docs
description: Obtenga información sobre autenticación en Digital Twins con el control de acceso basado en roles.
author: lyrana
manager: alinast
ms.service: digital-twins
services: digital-twins
ms.topic: conceptual
ms.date: 07/29/2019
ms.author: lyhughes
ms.openlocfilehash: 6602a4df2c6e313156010102780240ba19fbac95
ms.sourcegitcommit: c8a102b9f76f355556b03b62f3c79dc5e3bae305
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2019
ms.locfileid: "68815572"
---
# <a name="role-based-access-control-in-azure-digital-twins"></a>Control de acceso basado en rol en Azure Digital Twins

Azure Digital Twins permite el control de acceso preciso a datos, recursos y acciones específicos en un grafo espacial. Para ello, se usa la administración pormenorizada de roles y permisos que se conoce como control de acceso basado en rol (RBAC). RBAC consta de _roles_ y _asignaciones de roles_. Los roles identifican el nivel de permiso. Las asignaciones de roles asocian un rol a un usuario o dispositivo.

Con RBAC, se puede conceder permiso a:

- Un usuario
- Un dispositivo
- Una entidad de servicio
- Una función definida por el usuario
- Todos los usuarios que pertenezcan a un dominio
- Un inquilino

También puede ajustarse el grado de acceso.

El control de acceso basado en rol es único en el sentido de que los permisos se heredan hacia abajo del grafo espacial.

## <a name="what-can-i-do-with-rbac"></a>¿Qué puedo hacer con RBAC?

Un programador puede usar RBAC para:

- Conceder a un usuario la capacidad de administrar los dispositivos de todo un edificio, o de una sala o planta determinadas.
- Conceder a un administrador acceso global a todos los nodos del grafo espacial para un grafo completo o solo para una sección del grafo.
- Conceder a especialista en soporte técnico acceso de lectura al grafo, excepto a las claves de acceso.
- Conceder a todos los miembros de un dominio acceso de lectura a todos los objetos del grafo.

## <a name="rbac-best-practices"></a>Procedimientos recomendados de RBAC

[!INCLUDE [digital-twins-permissions](../../includes/digital-twins-rbac-best-practices.md)]

## <a name="roles"></a>Roles

### <a name="role-definitions"></a>Definiciones de roles

Una definición de rol es una colección de permisos y otros atributos que constituyen un rol. Una definición de roles enumera las operaciones permitidas, como *CREAR*, *LEER*, *ACTUALIZAR* y *ELIMINAR* que cualquier objeto con ese rol puede realizar. También especifica a qué tipos de objeto se aplican los permisos.

[!INCLUDE [digital-twins-roles](../../includes/digital-twins-roles.md)]

>[!NOTE]
> Para recuperar las definiciones completas de los roles anteriores, consulte la API de sistema o roles.
> Para obtener más información, lea [Creación y administración de asignaciones de roles](./security-create-manage-role-assignments.md#all).

### <a name="object-identifier-types"></a>Tipo de identificador de objeto

[!INCLUDE [digital-twins-object-types](../../includes/digital-twins-object-id-types.md)]

>[!TIP]
> Descubra cómo conceder permisos a la entidad de servicio leyendo [Creación y administración de asignaciones de roles](./security-create-manage-role-assignments.md#grant).

Estos artículos de documentación de referencia describen lo siguiente:

- Cómo [consultar el identificador de objeto para un usuario](https://docs.microsoft.com/powershell/module/azuread/get-azureaduser?view=azureadps-2.0).
- Cómo [obtener el identificador de objeto para una entidad de servicio](https://docs.microsoft.com/powershell/module/az.resources/get-azadserviceprincipal).
- C7 [recuperar el identificador de objeto de un inquilino de Azure AD](../active-directory/develop/quickstart-create-new-tenant.md).

## <a name="role-assignments"></a>Asignaciones de roles

Una asignación de roles de Azure Digital Twins asocia un objeto, como un usuario o un inquilino de Azure AD, a un rol y un espacio. Los permisos se conceden a todos los objetos que pertenecen a dicho espacio. El espacio incluye todo el grafo espacial que hay debajo.

Por ejemplo, un usuario recibe una asignación de roles con el rol `DeviceInstaller` para el nodo raíz de un grafo espacial, que representa un edificio. Así que el usuario puede leer y actualizar los dispositivos de ese nodo y todos los demás espacios secundarios del edificio.

Para conceder permisos a un destinatario, cree una asignación de roles. Para revocar permisos, quite la asignación de roles.

>[!IMPORTANT]
> Para obtener más información sobre las asignaciones de roles, lea [Creación y administración de asignaciones de roles](./security-create-manage-role-assignments.md).

## <a name="next-steps"></a>Pasos siguientes

- Para obtener más información sobre la creación y administración de asignaciones de roles de Azure Digital Twins, lea [Creación y administración de asignaciones de roles](./security-create-manage-role-assignments.md).
