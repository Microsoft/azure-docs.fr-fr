---
title: Guide de démarrage rapide - Sauvegarder une machine virtuelle avec un modèle Resource Manager
description: Découvrez comment sauvegarder vos machines virtuelles avec un modèle Azure Resource Manager
ms.devlang: azurecli
ms.topic: quickstart
ms.date: 05/14/2019
ms.custom: mvc,subject-armqs
ms.openlocfilehash: e64aa10d8ddadd367d04b2b480770a99f3ece1dc
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "88826529"
---
# <a name="quickstart-back-up-a-virtual-machine-in-azure-with-an-arm-template"></a>Démarrage rapide : Sauvegarder une machine virtuelle dans Azure avec un modèle ARM

La [sauvegarde Azure](backup-overview.md) sauvegarde les applications et machines locales, ainsi que les machines virtuelles Azure. Cet article explique comment sauvegarder une machine virtuelle Azure avec un modèle Azure Resource Manager (modèle ARM) et Azure PowerShell. Ce guide de démarrage rapide porte essentiellement sur le déploiement d’un modèle ARM en vue de créer un coffre Recovery Services. Pour plus d’informations sur le développement de modèles ARM, consultez la [documentation Azure Resource Manager](../azure-resource-manager/index.yml) et les [informations de référence sur les modèles](/azure/templates/microsoft.recoveryservices/allversions).

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Un [coffre Recovery Services](backup-azure-recovery-services-vault-overview.md) est un conteneur logique qui stocke des données de sauvegarde pour des ressources protégées, telles que des machines virtuelles Azure. Quand un travail de sauvegarde s’exécute, il crée un point de récupération à l’intérieur du coffre Recovery Services. Vous pouvez ensuite utiliser un de ces points de récupération pour restaurer des données à un moment donné dans le temps. Vous pouvez également sauvegarder une machine virtuelle en utilisant [Azure PowerShell](./quick-backup-vm-powershell.md), l’interface [Azure CLI](quick-backup-vm-cli.md) ou le [portail Azure](quick-backup-vm-portal.md).

Si votre environnement remplit les prérequis et que vous êtes déjà familiarisé avec l’utilisation des modèles ARM, sélectionnez le bouton **Déployer sur Azure**. Le modèle s’ouvre dans le portail Azure.

[![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-recovery-services-create-vm-and-configure-backup%2Fazuredeploy.json)

## <a name="review-the-template"></a>Vérifier le modèle

Le modèle utilisé dans ce guide de démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/101-recovery-services-create-vm-and-configure-backup/). Ce modèle vous permet de déployer une machine virtuelle Windows simple et un coffre Recovery Services configuré avec la stratégie par défaut pour la protection.

:::code language="json" source="~/quickstart-templates/101-recovery-services-create-vm-and-configure-backup/azuredeploy.json":::

Les ressources définies dans le modèle sont les suivantes :

- [**Microsoft.Storage/storageAccounts**](/azure/templates/microsoft.storage/storageaccounts)
- [**Microsoft.Network/publicIPAddresses**](/azure/templates/microsoft.network/publicipaddresses)
- [**Microsoft.Network/networkSecurityGroups**](/azure/templates/microsoft.network/networksecuritygroups)
- [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualnetworks)
- [**Microsoft.Network/networkInterfaces**](/azure/templates/microsoft.network/networkinterfaces)
- [**Microsoft.Compute/virtualMachines**](/azure/templates/microsoft.compute/virtualmachines)
- [**Microsoft.RecoveryServices/vaults**](/azure/templates/microsoft.recoveryservices/2016-06-01/vaults)
- [**Microsoft.RecoveryServices/vaults/backupFabrics/protectionContainers/protectedItems**](/azure/templates/microsoft.recoveryservices/vaults/backupfabrics/protectioncontainers/protecteditems)

## <a name="deploy-the-template"></a>Déployer le modèle

Pour déployer le modèle, sélectionnez **Essayer** afin d’ouvrir Azure Cloud Shell, puis collez le script PowerShell suivant dans la fenêtre de l’interpréteur de commandes. Pour coller le code, cliquez avec le bouton droit sur la fenêtre de l’interpréteur de commandes, puis sélectionnez **Coller**.

```azurepowershell-interactive
$projectName = Read-Host -Prompt "Enter a project name (limited to eight characters) that is used to generate Azure resource names"
$location = Read-Host -Prompt "Enter the location (for example, centralus)"
$adminUsername = Read-Host -Prompt "Enter the administrator username for the virtual machine"
$adminPassword = Read-Host -Prompt "Enter the administrator password for the virtual machine" -AsSecureString
$dnsPrefix = Read-Host -Prompt "Enter the unique DNS Name for the Public IP used to access the virtual machine"

$resourceGroupName = "${projectName}rg"
$templateUri = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-recovery-services-create-vm-and-configure-backup/azuredeploy.json"

New-AzResourceGroup -Name $resourceGroupName -Location $location
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri $templateUri -projectName $projectName -adminUsername $adminUsername -adminPassword $adminPassword -dnsLabelPrefix $dnsPrefix
```

Dans ce guide de démarrage rapide, vous utilisez Azure PowerShell pour déployer le modèle ARM. Le [portail Azure](../azure-resource-manager/templates/deploy-portal.md), l’interface [Azure CLI](../azure-resource-manager/templates/deploy-cli.md) et [l’API Rest](../azure-resource-manager/templates/deploy-rest.md) peuvent également être utilisés pour déployer des modèles.

## <a name="validate-the-deployment"></a>Valider le déploiement

### <a name="start-a-backup-job"></a>Démarrer un travail de sauvegarde

Le modèle crée une machine virtuelle et permet la sauvegarde sur la machine virtuelle. Après avoir déployé le modèle, vous devez démarrer un travail de sauvegarde. Pour plus d’informations, consultez [Démarrer un travail de sauvegarde](./quick-backup-vm-powershell.md#start-a-backup-job).

### <a name="monitor-the-backup-job"></a>Surveiller le travail de sauvegarde

Pour surveiller le travail de sauvegarde, consultez [Surveiller le travail de sauvegarde](./quick-backup-vm-powershell.md#monitor-the-backup-job).

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous n’avez plus besoin de sauvegarder la machine virtuelle, vous pouvez la nettoyer.

- Si vous voulez essayer de restaurer la machine virtuelle, ignorez cette opération de nettoyage.
- Si vous avez utilisé une machine virtuelle existante, vous pouvez ignorer l’applet de commande finale [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) pour laisser en place le groupe de ressources et la machine virtuelle.

Désactivez la protection, supprimez les points de restauration et le coffre. Supprimez ensuite le groupe de ressources et les ressources de machine virtuelle associées comme suit :

```powershell
Disable-AzRecoveryServicesBackupProtection -Item $item -RemoveRecoveryPoints
$vault = Get-AzRecoveryServicesVault -Name "myRecoveryServicesVault"
Remove-AzRecoveryServicesVault -Vault $vault
Remove-AzResourceGroup -Name "myResourceGroup"
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé un coffre Recovery Services, activé la protection sur une machine virtuelle et créé le point de récupération initial.

- [Découvrez comment](tutorial-backup-vm-at-scale.md) sauvegarder des machines virtuelles dans le portail Azure.
- [Découvrez comment](tutorial-restore-disk.md) rapidement restaurer une machine virtuelle
- [Découvrez comment](../azure-resource-manager/templates/template-tutorial-create-first-template.md) créer des modèles ARM.
