---
title: Démarrage rapide pour créer un coffre Recovery Services Azure avec un modèle Azure Resource Manager.
description: Dans ce guide de démarrage rapide, vous allez apprendre à créer un coffre Recovery Services Azure à l’aide d’un modèle Azure Resource Manager (modèle ARM).
ms.topic: quickstart
ms.custom: subject-armqs, devx-track-azurecli
ms.date: 04/29/2020
ms.openlocfilehash: c1c6d625c67130a4e3608cee6134b8c81404c01b
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92745872"
---
# <a name="quickstart-create-a-recovery-services-vault-using-an-arm-template"></a>Démarrage rapide : Créer un coffre Recovery Services avec un modèle ARM

Ce guide de démarrage rapide explique comment configurer un coffre Recovery Services à l’aide d’un modèle Azure Resource Manager (modèle ARM). Le service [Azure Site Recovery](site-recovery-overview.md) contribue à votre stratégie de continuité d’activité et de reprise d’activité afin que vos applications métier demeurent en ligne pendant les interruptions planifiées et non planifiées. Site Recovery gère la reprise d’activité des machines locales et des machines virtuelles Azure, notamment la réplication, le basculement et la récupération.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si votre environnement remplit les prérequis et que vous êtes déjà familiarisé avec l’utilisation des modèles ARM, sélectionnez le bouton **Déployer sur Azure** . Le modèle s’ouvre dans le portail Azure.

[![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-recovery-services-vault-create%2Fazuredeploy.json)

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure actif, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="review-the-template"></a>Vérifier le modèle

Le modèle utilisé dans ce démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/101-recovery-services-vault-create/).

:::code language="json" source="~/quickstart-templates/101-recovery-services-vault-create/azuredeploy.json":::

Deux ressources Azure sont définies dans le modèle :

- [Microsoft.RecoveryServices/vaults](/azure/templates/microsoft.recoveryservices/vaults) : crée le coffre.
- [Microsoft.RecoveryServices/vaults/backupstorageconfig](/rest/api/backup/backupresourcestorageconfigs) : configure les paramètres de redondance de sauvegarde du coffre.

Le modèle comprend des paramètres facultatifs pour la configuration de sauvegarde du coffre. Les paramètres de redondance de stockage sont le stockage localement redondant (LRS) ou le stockage géoredondant (GRS). Pour plus d’informations, consultez [Définir la redondance de stockage](../backup/backup-create-rs-vault.md#set-storage-redundancy).

Pour plus d’informations sur les modèles Azure Recovery Services, consultez [Modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Recoveryservices&pageNumber=1&sort=Popular).

## <a name="deploy-the-template"></a>Déployer le modèle

Pour déployer le modèle, vous devez définir l’ **Abonnement** , le **Groupe de ressources** et le **Nom du coffre** .

1. Pour vous connecter à Azure et ouvrir le modèle, sélectionnez l’image **Déployer sur Azure** .

   [![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-recovery-services-vault-create%2Fazuredeploy.json)

1. Sélectionnez ou entrez les valeurs suivantes :

   :::image type="content" source="media/quickstart-create-vault-template/create-vault-template.png" alt-text="Modèle pour créer un coffre Recovery Services.":::

   - **Abonnement** : sélectionnez votre abonnement Azure.
   - **Groupe de ressources**  : sélectionnez un groupe existant ou sélectionnez **Créer** pour ajouter un groupe.
   - **Emplacement**  : correspond par défaut à la localisation du groupe de ressources et devient indisponible dès qu’un groupe de ressources est sélectionné.
   - **Nom du coffre**  : spécifiez un nom pour le coffre.
   - **Changer le type de stockage**  : La valeur par défaut est **false** . Sélectionnez **true** uniquement si vous devez changer le type de stockage du coffre.
   - **Type de stockage du coffre**  : La valeur par défaut est **GloballyRedundant** . Si le type de stockage est défini sur **true** , sélectionnez **LocallyRedundant** .
   - **Emplacement**  : la fonction `[resourceGroup().location]` correspond par défaut à la localisation du groupe de ressources. Pour changer la localisation, entrez une valeur telle que **westus** .
   - Cochez la case **J’accepte les termes et conditions mentionnés ci-dessus** .

1. Pour commencer le déploiement du coffre, sélectionnez le bouton **Acheter** . Une fois le déploiement réussi, une notification s’affiche.

   :::image type="content" source="media/quickstart-create-vault-template/deployment-success.png" alt-text="Modèle pour créer un coffre Recovery Services.":::

## <a name="validate-the-deployment"></a>Valider le déploiement

Pour vérifier que le coffre a été créé, utilisez Azure CLI ou Azure PowerShell.

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/CLI)

```azurecli-interactive
echo "Enter the resource group name:" &&
read resourceGroupName &&
echo "Enter the vault name:" &&
read vaultName &&
az backup vault show --name $vaultName --resource-group $resourceGroupName &&
az backup vault backup-properties show --name $vaultName --resource-group $resourceGroupName &&
echo "Press [ENTER] to continue ..."
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell-interactive
$resouceGroupName = Read-Host -Prompt "Enter the resource group name"
$vaultName = Read-Host -Prompt "Enter the vault name"
$vaultBackupConfig = Get-AzRecoveryServicesVault -Name $vaultName
Get-AzRecoveryServicesVault -ResourceGroupName $resouceGroupName -Name $vaultName
Get-AzRecoveryServicesBackupProperty -Vault $vaultBackupConfig
Write-Host "Press [ENTER] to continue..."
```

---

La sortie suivante est un extrait des informations du coffre :

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/CLI)

```Output
"id": "/subscriptions/<Subscription Id>/resourceGroups/myResourceGroup
         /providers/Microsoft.RecoveryServices/vaults/myVault"
"location": "eastus"
"name": "myVault"
"resourceGroup": "myResourceGroup"

"storageModelType": "GeoRedundant"
"storageType": "GeoRedundant"
"type": "Microsoft.RecoveryServices/vaults/backupstorageconfig"
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```Output
Name              : myVault
Type              : Microsoft.RecoveryServices/vaults
Location          : eastus
ResourceGroupName : myResourceGroup
SubscriptionId    : <Subscription Id>

BackupStorageRedundancy
-----------------------
GeoRedundant
```

---

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous envisagez d’utiliser les nouvelles ressources, aucune action n’est nécessaire. Dans le cas contraire, vous pouvez supprimer le groupe de ressources et le coffre créé dans ce guide de démarrage rapide. Pour supprimer le groupe de ressources et ses ressources, utilisez Azure CLI ou Azure PowerShell.

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/CLI)

```azurecli-interactive
echo "Enter the resource group name:" &&
read resourceGroupName &&
az group delete --name $resourceGroupName &&
echo "Press [ENTER] to continue ..."
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell-interactive
$resouceGroupName = Read-Host -Prompt "Enter the resource group name"
Remove-AzResourceGroup -Name $resouceGroupName
Write-Host "Press [ENTER] to continue..."
```

---

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé un coffre Recovery Services. Pour en savoir plus sur la reprise d’activité, passez à l’article de démarrage rapide suivant.

> [!div class="nextstepaction"]
> [Configurer une récupération d’urgence](azure-to-azure-quickstart.md)
