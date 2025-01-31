---
title: Control de acceso basado en rol en Azure Cosmos DB con integración de Azure Active Directory
description: Obtenga información sobre cómo Azure Cosmos DB proporciona protección de bases de datos con la integración de Active Directory (RBAC).
author: markjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 05/23/2019
ms.author: mjbrown
ms.openlocfilehash: 971d2ec96906a3309963495dd1af5d293a71f265
ms.sourcegitcommit: d4dfbc34a1f03488e1b7bc5e711a11b72c717ada
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/13/2019
ms.locfileid: "66243515"
---
# <a name="role-based-access-control-in-azure-cosmos-db"></a>Control de acceso basado en rol en Azure Cosmos DB

Azure Cosmos DB proporciona un control de acceso basado en role (RBAC) integrado para escenarios de administración comunes en Azure Cosmos DB. Un usuario que tiene un perfil en Azure Active Directory puede asignar estos roles RBAC a usuarios, grupos, entidades de servicio o identidades administradas para conceder o denegar el acceso a recursos y operaciones en los recursos de Azure Cosmos DB. Las asignaciones de roles están dirigidas únicamente al acceso al plano de control, que incluye el acceso a las cuentas, bases de datos, contenedores y ofertas de Azure Cosmos (rendimiento).

## <a name="built-in-roles"></a>Roles integrados

Los siguientes son los roles integrados compatibles que se admiten en Azure Cosmos DB:

|**Rol integrado**  |**Descripción**  |
|---------|---------|
|[Colaborador de cuentas de DocumentDB](../role-based-access-control/built-in-roles.md#documentdb-account-contributor)   | Puede administrar cuentas de Azure Cosmos DB.  |
|[Lector de cuentas de Cosmos DB](../role-based-access-control/built-in-roles.md#cosmos-db-account-reader-role)  | Puede leer los datos de cuentas de Azure Cosmos DB.        |
|[Operador de copias de seguridad de Cosmos](../role-based-access-control/built-in-roles.md#cosmosbackupoperator)     |  Puede enviar una solicitud de restauración para una base de datos de Azure Cosmos o un contenedor.       |
|[Operador de Cosmos DB](../role-based-access-control/built-in-roles.md#cosmos-db-operator)  | Puede aprovisionar cuentas, bases de datos y contenedores de Azure Cosmos, pero no puede acceder a las claves necesarias para acceder a los datos.         |

> [!IMPORTANT]
> La compatibilidad con RBAC en Azure Cosmos DB se aplica solo a las operaciones del plano de control. Las operaciones del plano de datos se protegen mediante las claves maestras o tokens de recursos. Para más información, consulte [Protección del acceso a los datos de Azure Cosmos DB](secure-access-to-data.md).

## <a name="identity-and-access-management-iam"></a>Administración de identidad y acceso (IAM)

El panel **Control de acceso (IAM)** de Azure Portal se utiliza para configurar el control de acceso basado en rol en los recursos de Azure Cosmos. Los roles se aplican a usuarios, grupos, entidades de servicio e identidades administradas en Active Directory. Puede usar roles integrados o personalizados para usuarios y grupos. En la captura de pantalla siguiente se muestra la integración de Active Directory (RBAC) con control de acceso (IAM) en Azure Portal:

![Control de acceso (IAM) en Azure Portal: demostración de la seguridad de bases de datos](./media/role-based-access-control/database-security-identity-access-management-rbac.png)

## <a name="custom-roles"></a>Roles personalizados

Además de los roles integrados, los usuarios también pueden crear [roles personalizados](../role-based-access-control/custom-roles.md) en Azure y aplicar estos roles a las entidades de servicio en todas las suscripciones dentro de su inquilino de Active Directory. Los roles personalizados proporcionan a los usuarios una forma de crear definiciones de roles RBAC con un conjunto personalizado de operaciones de los proveedores de recursos. Para saber qué operaciones están disponibles para crear roles personalizados para Azure Cosmos DB, consulte [Operaciones del proveedor de recursos de Azure Cosmos DB](../role-based-access-control/resource-provider-operations.md#microsoftdocumentdb).

## <a name="next-steps"></a>Pasos siguientes

- [¿Qué es el control de acceso basado en rol (RBAC) para los recursos de Azure?](../role-based-access-control/overview.md)
- [Roles personalizados en los recursos de Azure](../role-based-access-control/custom-roles.md)
- [Operaciones del proveedor de recursos de Azure Cosmos DB](../role-based-access-control/resource-provider-operations.md#microsoftdocumentdb)
