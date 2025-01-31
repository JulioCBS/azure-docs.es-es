---
title: 'Conexión de VM de cliente: Instancia administrada de Azure SQL Database| Microsoft Docs'
description: Conéctese a una Instancia administrada de Azure SQL Database mediante SQL Server Management Studio desde una máquina virtual de Azure.
services: sql-database
ms.service: sql-database
ms.subservice: managed-instance
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: jovanpop-msft
ms.author: jovanpop
ms.reviewer: sstein, carlrab, srbozovi, bonova
ms.date: 02/18/2019
ms.openlocfilehash: ddcac56671e145728f02d31bf23c657ea172e4c0
ms.sourcegitcommit: 7c4de3e22b8e9d71c579f31cbfcea9f22d43721a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/26/2019
ms.locfileid: "68567663"
---
# <a name="quickstart-configure-azure-vm-to-connect-to-an-azure-sql-database-managed-instance"></a>Guía de inicio rápido: Configurar una máquina virtual de Azure para la conexión a una Instancia administrada de Azure SQL Database

Esta guía de inicio rápido muestra cómo configurar una máquina virtual de Azure para conectarse a una Instancia administrada de Azure SQL Database mediante SQL Server Management Studio (SSMS). Para obtener una guía de inicio rápido en la que se muestra cómo conectarse desde un equipo cliente local mediante una conexión de punto a sitio, vea [Configuración de una conexión de punto a sitio](sql-database-managed-instance-configure-p2s.md).

## <a name="prerequisites"></a>Requisitos previos

Esta guía de inicio rápido usa los recursos creados en [Creación de una Instancia administrada](sql-database-managed-instance-get-started.md) como punto de partida.

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en el [Azure Portal](https://portal.azure.com/).

## <a name="create-a-new-subnet-in-the-managed-instance-vnet"></a>Creación de una nueva subred en la red virtual de Instancia administrada

En los pasos siguientes se crea una nueva subred en la red virtual de Instancia administrada para poder conectar una máquina virtual de Azure a la Instancia administrada. La subred de Instancia administrada se dedica a las Instancias administradas. No puede crear ningún otro recurso, como Azure Virtual Machines, en esa subred.

1. Abra el grupo de recursos para la Instancia administrada que creó en la guía de inicio rápido [Creación de una Instancia administrada](sql-database-managed-instance-get-started.md). Seleccione la red virtual para su Instancia administrada.

   ![Recursos de la Instancia administrada](./media/sql-database-managed-instance-configure-vm/resources.png)

2. Seleccione **Subredes** y, a continuación, seleccione **+ Subred** para crear una nueva subred.

   ![Subredes de Instancia administrada](./media/sql-database-managed-instance-configure-vm/subnets.png)

3. Rellene el formulario con la información de esta tabla:

   | Configuración| Valor sugerido | Descripción |
   | ---------------- | ----------------- | ----------- |
   | **Nombre** | Cualquier nombre válido|Para conocer cuáles son los nombres válidos, consulte el artículo [Convenciones de nomenclatura](https://docs.microsoft.com/azure/architecture/best-practices/naming-conventions).|
   | **Intervalo de direcciones (bloque CIDR)** | Un intervalo válido | El valor predeterminado es adecuado para este inicio rápido.|
   | **Grupo de seguridad de red** | None | El valor predeterminado es adecuado para este inicio rápido.|
   | **Tabla de rutas** | None | El valor predeterminado es adecuado para este inicio rápido.|
   | **Puntos de conexión de servicio** | 0 seleccionado | El valor predeterminado es adecuado para este inicio rápido.|
   | **Delegación de subred** | None | El valor predeterminado es adecuado para este inicio rápido.|

   ![Nueva subred de Instancia administrada de una máquina virtual de cliente](./media/sql-database-managed-instance-configure-vm/new-subnet.png)

4. Seleccione **Aceptar** para crear esta subred adicional en la red virtual de Instancia administrada.

## <a name="create-a-virtual-machine-in-the-new-subnet-in-the-vnet"></a>Creación de una máquina virtual en la nueva subred de la red virtual

En los pasos siguientes se muestra cómo crear una máquina virtual en la nueva subred para la conexión a la Instancia administrada.

## <a name="prepare-the-azure-virtual-machine"></a>Preparación de la máquina virtual de Azure

Dado que la instancia administrada de SQL está colocada en su red virtual privada, es preciso que cree una máquina virtual de Azure con una herramienta de cliente SQL instalada como SQL Server Management Studio o Azure Data Studio. Esta herramienta le permite conectarse a la Instancia administrada y ejecutar consultas. En esta guía de inicio rápido se usa SQL Server Management Studio.

La forma más fácil de crear una máquina virtual cliente con todas las herramientas necesarias es usar las plantillas de Azure Resource Manager.

1. Asegúrese de haber iniciado sesión en Azure Portal en otra pestaña del explorador. A continuación, seleccione el siguiente botón para crear una máquina virtual cliente e instalar SQL Server Management Studio:

    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjovanpop-msft%2Fazure-quickstart-templates%2Fsql-win-vm-w-tools%2F201-vm-win-vnet-sql-tools%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>

2. Rellene el formulario con la información de la siguiente tabla:

   | Configuración| Valor sugerido | DESCRIPCIÓN |
   | ---------------- | ----------------- | ----------- |
   | **Suscripción** | Una suscripción válida | Debe ser una suscripción en la que tiene permiso para crear recursos. |
   | **Grupo de recursos** |Grupo de recursos que especificó en el inicio rápido [Crear Instancia administrada](sql-database-managed-instance-get-started.md).|Este grupo de recursos debe ser el grupo de recursos donde existe la red virtual.|
   | **Ubicación** | Ubicación del grupo de recursos | Este valor se rellena según el grupo de recursos seleccionado. |
   | **Nombre de la máquina virtual**  | Cualquier nombre válido | Para conocer cuáles son los nombres válidos, consulte el artículo [Convenciones de nomenclatura](https://docs.microsoft.com/azure/architecture/best-practices/naming-conventions).|
   |**Nombre de usuario administrador**|Cualquier nombre de usuario válido|Para conocer cuáles son los nombres válidos, consulte el artículo [Convenciones de nomenclatura](https://docs.microsoft.com/azure/architecture/best-practices/naming-conventions). No utilice "serveradmin", ya es un rol de nivel de servidor reservado.<br>Use este nombre de usuario cada vez que [se conecte a la máquina virtual](#connect-to-virtual-machine).|
   |**Contraseña**|Cualquier contraseña válida|La contraseña debe tener al menos 12 caracteres de largo y cumplir con los [requisitos de complejidad definidos](../virtual-machines/windows/faq.md#what-are-the-password-requirements-when-creating-a-vm).<br>Use esta contraseña cada vez que [se conecte a la máquina virtual](#connect-to-virtual-machine).|
   | **Tamaño de máquina virtual** | Cualquier tamaño válido | El valor predeterminado de esta plantilla de **Standard_B2s** es suficiente para esta guía de inicio rápido. |
   | **Ubicación**|[resourceGroup().location].| No cambie este valor. |
   | **Nombre de la red virtual**|La red virtual en la que creó la Instancia administrada.|
   | **Nombre de subred**|El nombre de la subred que creó en el procedimiento anterior| No elija la subred en la que creó la Instancia administrada.|
   | **Ubicación de artefactos** | [deployment().properties.templateLink.uri] | No cambie este valor. |
   | **Token de Sas de ubicación de artefactos** | déjelo en blanco | No cambie este valor. |

   ![crear máquina virtual cliente](./media/sql-database-managed-instance-configure-vm/create-client-sql-vm.png)

   Si usa el nombre de red virtual sugerido y la subred predeterminada de [Crear la Instancia administrada](sql-database-managed-instance-get-started.md), no es necesario cambiar los dos últimos parámetros. En caso contrario, debe cambiar estos valores a los valores que especificó al configurar el entorno de red.

3. Seleccione la casilla **Acepto los términos y condiciones indicados anteriormente**.
4. Seleccione **Comprar** para implementar la máquina virtual de Azure en la red.
5. Seleccione el icono **Notificaciones** para ver el estado de la implementación.

> [!IMPORTANT]
> No continúe hasta aproximadamente 15 minutos después de la creación de la máquina virtual para dar tiempo a que los scripts posteriores a la creación instalen SQL Server Management Studio.

## <a name="connect-to-virtual-machine"></a>Conexión a la máquina virtual

En los pasos siguientes se muestra cómo conectarse a la máquina virtual recién creada mediante una conexión de escritorio remoto.

1. Una vez finalizada la implementación, vaya al recurso de máquina virtual.

    ![máquina virtual](./media/sql-database-managed-instance-configure-vm/vm.png)  

2. Seleccione **Conectar**.

   Aparece un formulario de archivo de Protocolo de escritorio remoto (archivo .rdp) con la dirección IP pública y el número de puerto de la máquina virtual.

   ![Formulario RDP](./media/sql-database-managed-instance-configure-vm/rdp.png)  

3. Seleccione **Descargar archivo RDP**.

   > [!NOTE]
   > También puede usar SSH para conectarse a la máquina virtual.

4. Cierre el formulario **Conexión a la máquina virtual**.
5. Para conectarse a la máquina virtual, abra el archivo RDP descargado.
6. Cuando se le pida, seleccione **Connect** (Conectar). En un equipo Mac, necesita un cliente RDP como este [Cliente de Escritorio remoto](https://itunes.apple.com/us/app/microsoft-remote-desktop/id715768417?mt=12) de Mac App Store.

7. Escriba el nombre de usuario y la contraseña que especificó al crear la máquina virtual y, a continuación, elija **Aceptar**.

8. Puede recibir una advertencia de certificado durante el proceso de inicio de sesión. Elija **Sí** o en **Continuar** para continuar con la conexión.

Está conectado a la máquina virtual en el panel de Administrador del servidor.

## <a name="use-ssms-to-connect-to-the-managed-instance"></a>Uso de SSMS para conectarse a la Instancia administrada

1. En la máquina virtual, abra SQL Server Management Studio (SSMS).

   Tardará unos instantes en abrirse, ya que debe completar su configuración y esta es la primera vez que se inicia SSMS.
2. En el cuadro de diálogo **Conectar a un servidor**, escriba el **nombre de host** completo para su Instancia administrada en el cuadro **Nombre del servidor**. Seleccione **Autenticación de SQL Server**, proporcione su nombre de usuario y contraseña y, a continuación, seleccione **Conectar**.

    ![conexión ssms](./media/sql-database-managed-instance-configure-vm/ssms-connect.png)  

Después de conectarse, puede ver las bases de datos del sistema y de los usuarios en el nodo Bases de datos, así como varios objetos en los nodos Seguridad, Objetos de servidor, Replicación, Administración, Agente SQL Server y XEvent Profiler.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener una guía de inicio rápido en la que se muestra cómo conectarse a una Instancia administrada desde un equipo cliente local mediante una conexión de punto a sitio, vea [Configuración de una conexión de punto a sitio](sql-database-managed-instance-configure-p2s.md).
- Para obtener información general de las opciones de conexión para las aplicaciones, vea [Conexión de la aplicación a Instancia administrada de Azure SQL Database](sql-database-managed-instance-connect-app.md).
- Para restaurar una base de datos SQL Server existente desde el entorno local en una Instancia administrada, puede usar [Azure Database Migration Service (DMS) para la migración](../dms/tutorial-sql-server-to-managed-instance.md), o bien el [comando T-SQL RESTORE](sql-database-managed-instance-get-started-restore.md) para restaurar desde un archivo de copia de seguridad de base de datos.
