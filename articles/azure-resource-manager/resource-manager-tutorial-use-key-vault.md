---
title: Integración de Azure Key Vault en la implementación de la plantilla de Resource Manager | Microsoft Docs
description: Aprenda a usar Azure Key Vault para pasar valores de parámetros seguros durante la implementación de la plantilla de Resource Manager
services: azure-resource-manager
documentationcenter: ''
author: mumian
manager: dougeby
editor: tysonn
ms.service: azure-resource-manager
ms.workload: multiple
ms.tgt_pltfrm: na
ms.devlang: na
ms.date: 05/23/2019
ms.topic: tutorial
ms.author: jgao
ms.custom: seodec18
ms.openlocfilehash: b91a3488750795e8ee9df6602bb2b3df8a9b08ec
ms.sourcegitcommit: f56b267b11f23ac8f6284bb662b38c7a8336e99b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/28/2019
ms.locfileid: "67436570"
---
# <a name="tutorial-integrate-azure-key-vault-in-your-resource-manager-template-deployment"></a>Tutorial: Integración de Azure Key Vault en la implementación de la plantilla de Resource Manager

Aprenda a recuperar los secretos de Azure Key Vault y a pasarlos como parámetros durante la implementación de Azure Resource Manager. El valor del parámetro nunca se expone, ya que solo se hace referencia a su identificador de almacén de claves. Para más información, consulte [Uso de Azure Key Vault para pasar el valor de parámetro seguro durante la implementación](./resource-manager-keyvault-parameter.md).

En el tutorial [Establecimiento del orden de implementación de los recursos](./resource-manager-tutorial-create-templates-with-dependent-resources.md), se crea una máquina virtual (VM). Deberá proporcionar el nombre de usuario y la contraseña de administrador de la VM. En lugar de proporcionar la contraseña, puede almacenar previamente la contraseña en Azure Key Vault y luego personalizar la plantilla para recuperar la contraseña desde el almacén de claves durante la implementación.

![Diagrama que muestra la integración de una plantilla de Resource Manager con un almacén de claves](./media/resource-manager-tutorial-use-key-vault/resource-manager-template-key-vault-diagram.png)

En este tutorial se describen las tareas siguientes:

> [!div class="checklist"]
> * Preparación de un almacén de claves
> * Apertura de una plantilla de inicio rápido
> * Edición del archivo de parámetros
> * Implementación de la plantilla
> * Validación de la implementación
> * Limpieza de recursos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Requisitos previos

Para completar este artículo, necesitará lo siguiente:

* [Visual Studio Code](https://code.visualstudio.com/) con la extensión [Resource Manager Tools](./resource-manager-quickstart-create-templates-use-visual-studio-code.md#prerequisites).
* Para mejorar la seguridad, use una contraseña generada para la cuenta de administrador de la VM. Este es un ejemplo para generar una contraseña:

    ```azurecli-interactive
    openssl rand -base64 32
    ```
    Compruebe que la contraseña generada cumple los requisitos de contraseña de la VM. Cada servicio de Azure tiene unos requisitos de contraseña concretos. Para los requisitos de contraseña de una VM, consulte [¿Cuáles son los requisitos de contraseña cuando se crea una VM?](../virtual-machines/windows/faq.md#what-are-the-password-requirements-when-creating-a-vm).

## <a name="prepare-a-key-vault"></a>Preparación de un almacén de claves

En esta sección, creará un almacén de claves y le agregará un secreto, de manera que pueda recuperar el secreto al implementar la plantilla. Un almacén de claves se puede crear de muchas maneras. En este tutorial, se usa Azure PowerShell para implementar una [plantilla de Resource Manager](https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/tutorials-use-key-vault/CreateKeyVault.json). Esta plantilla hace lo siguiente:

* Se crea un almacén de claves con la propiedad `enabledForTemplateDeployment` habilitada. El valor de esta propiedad debe ser *true* para que el proceso de implementación de plantilla pueda acceder a los secretos definidos en este almacén de claves.
* Se agrega un secreto al almacén de claves. El secreto almacena la contraseña de administrador de la VM.

> [!NOTE]
> Como el usuario que implementa la plantilla de máquina virtual, si no es el propietario o un colaborador del almacén de claves, el propietario o colaborador debe conceder acceso al permiso *Microsoft.KeyVault/vaults/deploy/action* para el almacén de claves. Para más información, consulte [Uso de Azure Key Vault para pasar el valor de parámetro seguro durante la implementación](./resource-manager-keyvault-parameter.md).

Para ejecutar el siguiente script de Azure PowerShell, seleccione **Pruébelo** para abrir Azure Cloud Shell. Para pegar el script, haga clic con el botón derecho en el panel del shell y, a continuación, seleccione **Pegar**.

```azurepowershell-interactive
$projectName = Read-Host -Prompt "Enter a project name that is used for generating resource names"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"
$upn = Read-Host -Prompt "Enter your user principal name (email address) used to sign in to Azure"
$secretValue = Read-Host -Prompt "Enter the virtual machine administrator password" -AsSecureString

$resourceGroupName = "${projectName}rg"
$keyVaultName = $projectName
$adUserId = (Get-AzADUser -UserPrincipalName $upn).Id
$templateUri = "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/tutorials-use-key-vault/CreateKeyVault.json"

New-AzResourceGroup -Name $resourceGroupName -Location $location
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri $templateUri -keyVaultName $keyVaultName -adUserId $adUserId -secretValue $secretValue
```

> [!IMPORTANT]
> * Observe que el nombre del grupo de recursos es el nombre del proyecto, con **rg** anexado. Para que resulte más fácil [limpiar los recursos que creó en este tutorial](#clean-up-resources), use el mismo nombre de proyecto y de grupo de recursos al [implementar la siguiente plantilla](#deploy-the-template).
> * El nombre predeterminado para el secreto es **vmAdminPassword**. Está codificado de forma rígida en la plantilla.
> * Para permitir que la plantilla recupere el secreto, debe habilitar una directiva de acceso llamada "Habilitar el acceso a Azure Resource Manager para la implementación de plantillas" para el almacén de claves. Esta directiva está habilitada en la plantilla. Para más información sobre esta directiva de acceso, consulte [Implementación de almacenes de claves y secretos](./resource-manager-keyvault-parameter.md#deploy-key-vaults-and-secrets).

La plantilla tiene un valor de salida denominado *keyVaultId*. Anote el valor de identificador para su uso posterior, cuando se implemente la máquina virtual. El formato del identificador de recurso es:

```json
/subscriptions/<SubscriptionID>/resourceGroups/mykeyvaultdeploymentrg/providers/Microsoft.KeyVault/vaults/<KeyVaultName>
```

Cuando copie y pegue el identificador, este puede dividirse en varias líneas. Combine las líneas y recorte los espacios adicionales.

Para validar la implementación, ejecute el siguiente comando de PowerShell en el mismo panel de shell para recuperar el secreto en texto no cifrado. El comando solo funciona en la misma sesión de shell, ya que usa la variable *$keyVaultName*, que se define en el script de PowerShell anterior.

```azurepowershell
(Get-AzKeyVaultSecret -vaultName $keyVaultName  -name "vmAdminPassword").SecretValueText
```

Ha preparado un almacén de claves y un secreto. En las secciones siguientes se muestra cómo personalizar una plantilla existente para recuperar el secreto durante la implementación.

## <a name="open-a-quickstart-template"></a>Apertura de una plantilla de inicio rápido

Plantillas de inicio rápido de Azure es un repositorio de plantillas de Resource Manager. En lugar de crear una plantilla desde cero, puede buscar una plantilla de ejemplo y personalizarla. La plantilla que se usa en este tutorial se denomina [Deploy a simple Windows VM](https://azure.microsoft.com/resources/templates/101-vm-simple-windows/).

1. En Visual Studio Code, seleccione **Archivo** > **Abrir archivo**.

1. En el cuadro **Nombre de archivo**, pegue la siguiente dirección URL:

    ```url
    https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-simple-windows/azuredeploy.json
    ```

1. Seleccione **Abrir** para abrir el archivo. El escenario es el mismo que el que se usa en [Tutorial: Creación de plantillas de Azure Resource Manager con recursos dependientes](./resource-manager-tutorial-create-templates-with-dependent-resources.md).
   La plantilla define cinco recursos:

   * `Microsoft.Storage/storageAccounts`. Consulte la [referencia de plantilla](https://docs.microsoft.com/azure/templates/Microsoft.Storage/storageAccounts).
   * `Microsoft.Network/publicIPAddresses`. Consulte la [referencia de plantilla](https://docs.microsoft.com/azure/templates/microsoft.network/publicipaddresses).
   * `Microsoft.Network/virtualNetworks`. Consulte la [referencia de plantilla](https://docs.microsoft.com/azure/templates/microsoft.network/virtualnetworks).
   * `Microsoft.Network/networkInterfaces`. Consulte la [referencia de plantilla](https://docs.microsoft.com/azure/templates/microsoft.network/networkinterfaces).
   * `Microsoft.Compute/virtualMachines`. Consulte la [referencia de plantilla](https://docs.microsoft.com/azure/templates/microsoft.compute/virtualmachines).

   Resulta útil disponer cierto conocimientos básicos sobre la plantilla antes de personalizarla.

1. Seleccione **Archivo** > **Guardar como** y guarde una copia del archivo en el equipo local con el nombre *azuredeploy.json*.

1. Repita los pasos de 1 al 3 para abrir la dirección URL siguiente y luego guarde el archivo como *azuredeploy.parameters.json*.

    ```url
    https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-simple-windows/azuredeploy.parameters.json
    ```

## <a name="edit-the-parameters-file"></a>Edición del archivo de parámetros

No es necesario hacer ningún cambio en el archivo de plantilla.

1. En Visual Studio Code, abra *azuredeploy.parameters.json* si aún no está abierto.
1. Actualice el parámetro `adminPassword` a:

    ```json
    "adminPassword": {
        "reference": {
            "keyVault": {
            "id": "/subscriptions/<SubscriptionID>/resourceGroups/mykeyvaultdeploymentrg/providers/Microsoft.KeyVault/vaults/<KeyVaultName>"
            },
            "secretName": "vmAdminPassword"
        }
    },
    ```

    > [!IMPORTANT]
    > Reemplace el valor de **id** por el identificador de recurso del almacén de claves que creó en el procedimiento anterior.

    ![Integrar el almacén de claves y el archivo de parámetros de implementación de máquina virtual de la plantilla de Resource Manager](./media/resource-manager-tutorial-use-key-vault/resource-manager-tutorial-create-vm-parameters-file.png)

1. Actualice los valores siguientes:

    * **adminUsername**: el nombre de la cuenta de administrador de la máquina virtual.
    * **dnsLabelPrefix**: el nombre del valor dnsLabelPrefix.

    Para obtener ejemplos de nombres, consulte la imagen anterior.

1. Guarde los cambios.

## <a name="deploy-the-template"></a>Implementación de la plantilla

Siga las instrucciones de [Implementación de la plantilla](./resource-manager-tutorial-create-templates-with-dependent-resources.md#deploy-the-template). Cargue tanto *azuredeploy.json* como *azuredeploy.parameters.json* en Cloud Shell y luego use el siguiente script de PowerShell para implementar la plantilla:

```azurepowershell
$projectName = Read-Host -Prompt "Enter the same project name that is used for creating the key vault"
$location = Read-Host -Prompt "Enter the same location that is used for creating the key vault (i.e. centralus)"
$resourceGroupName = "${projectName}rg"

New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -TemplateFile "$HOME/azuredeploy.json" `
    -TemplateParameterFile "$HOME/azuredeploy.parameters.json"
```

Al implementar la plantilla, use el mismo grupo de recursos que el que usó en el almacén de claves. Con este método, es más fácil limpiar los recursos, ya que solo se necesita eliminar un único grupo de recursos en lugar de dos.

## <a name="validate-the-deployment"></a>Validación de la implementación

Después de haber implementado correctamente la máquina virtual, pruebe las credenciales de inicio de sesión con la contraseña que se almacena en el almacén de claves.

1. Abra [Azure Portal](https://portal.azure.com).

1. Seleccione **Grupos de recursos** >  **\<*NombreDelGrupoDeRecursos*>**  > **simpleWinVM**.
1. Seleccione **conectar** en la parte superior.
1. Seleccione **Descargar archivo RDP** y siga las instrucciones para iniciar sesión en la máquina virtual con la contraseña almacenada en el almacén de claves.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite los recursos de Azure, elimine el grupo de recursos para limpiar los recursos que implementó.

```azurepowershell-interactive
$projectName = Read-Host -Prompt "Enter the same project name that is used for creating the key vault"
$resourceGroupName = "${projectName}rg"

Remove-AzResourceGroup -Name $resourceGroupName
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, recuperó un secreto de Azure Key Vault. Luego, usó el secreto en la implementación de la plantilla. Para aprender a crear plantillas vinculadas, consulte:

> [!div class="nextstepaction"]
> [Creación de plantillas vinculadas](./resource-manager-tutorial-create-linked-templates.md)
