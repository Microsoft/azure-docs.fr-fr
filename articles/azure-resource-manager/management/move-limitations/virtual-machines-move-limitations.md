---
title: Déplacer des machines virtuelles Azure vers un nouveau groupe d’abonnements ou de ressources
description: Utilisez Azure Resource Manager pour déplacer des machines virtuelles vers un nouveau groupe de ressources ou abonnement.
ms.topic: conceptual
ms.date: 12/01/2020
ms.openlocfilehash: b1032b5a632bcac82cb9ae1f1b3df7b49f5463f5
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96456326"
---
# <a name="move-guidance-for-virtual-machines"></a>Conseils pour le déplacement de machines virtuelles

Cet article décrit les scénarios actuellement non pris en charge et les étapes pour déplacer des machines virtuelles avec sauvegarde.

## <a name="scenarios-not-supported"></a>Scénarios non pris en charge

Les scénarios suivants ne sont pas encore pris en charge :

* Les groupes de machines virtuelles identiques avec un équilibreur de charge de référence SKU Standard ou avec une adresse IP publique de référence SKU Standard ne peuvent pas être déplacés.
* Les machines virtuelles d'un réseau virtuel existant ne peuvent pas être déplacées vers un nouvel abonnement si vous ne déplacez pas toutes les ressources dans le réseau virtuel.
* Les machines virtuelles créées à partir de ressources Place de marché et auxquelles des plans sont associés ne peuvent pas être déplacées entre des abonnements. Pour obtenir une solution de contournement potentielle, consultez [Machines virtuelles associées aux plans de la Place de marché](#virtual-machines-with-marketplace-plans).
* Les machines virtuelles et les groupes de machines virtuelles identiques basse priorité ne peuvent pas être déplacés entre des groupes de ressources ou abonnements.
* Les machines virtuelles d’un groupe à haute disponibilité ne peuvent pas être déplacées individuellement.

## <a name="azure-disk-encryption"></a>Azure Disk Encryption

Vous ne pouvez pas déplacer une machine virtuelle intégrée à un coffre de clés pour implémenter [Azure Disk Encryption pour les machines virtuelles Linux ](../../../virtual-machines/linux/disk-encryption-overview.md) ou [Azure Disk Encryption pour les machines virtuelles Windows](../../../virtual-machines/windows/disk-encryption-overview.md). Pour déplacer la machine virtuelle, vous devez désactiver le chiffrement.

```azurecli-interactive
az vm encryption disable --resource-group demoRG --name myVm1
```

```azurepowershell-interactive
Disable-AzVMDiskEncryption -ResourceGroupName demoRG -VMName myVm1
```

## <a name="virtual-machines-with-marketplace-plans"></a>Machines virtuelles associées aux plans de la Place de marché

Les machines virtuelles créées à partir de ressources Place de marché et auxquelles des plans sont associés ne peuvent pas être déplacées entre des abonnements. Pour contourner cette limitation, vous pouvez déprovisionner la machine virtuelle dans l’abonnement actuel afin de la redéployer dans le nouvel abonnement. Les étapes suivantes vous permettent de recréer la machine virtuelle dans le nouvel abonnement. Toutefois, il est possible qu’elles ne fonctionnent pas pour tous les scénarios. Si le plan n’est plus disponible dans la Place de marché, ces étapes ne fonctionnent pas.

1. Copiez les informations relatives au plan.

1. Clonez le disque du système d’exploitation vers l’abonnement de destination, ou déplacez le disque d’origine après avoir supprimé la machine virtuelle de l’abonnement source.

1. Dans l’abonnement de destination, acceptez les conditions de la Place de marché pour votre plan. Vous pouvez accepter les conditions en exécutant la commande PowerShell suivante :

   ```azurepowershell
   Get-AzMarketplaceTerms -Publisher {publisher} -Product {product/offer} -Name {name/SKU} | Set-AzMarketplaceTerms -Accept
   ```

   Sinon, vous pouvez créer une instance de machine virtuelle basée sur le plan via le portail. Vous pouvez supprimer la machine virtuelle après avoir accepté les conditions du nouvel abonnement.

1. Dans l’abonnement de destination, utilisez PowerShell, l’interface CLI ou un modèle Azure Resource Manager pour recréer la machine virtuelle à partir du disque de système d’exploitation cloné. Incluez le plan de la Place de marché attaché au disque. Les informations relatives au plan doivent correspondre au plan que vous avez acheté dans le nouvel abonnement.

## <a name="virtual-machines-with-azure-backup"></a>Machines virtuelles avec Sauvegarde Azure

Pour déplacer des machines virtuelles configurées avec le service Sauvegarde Azure, vous devez supprimer les points de restauration du coffre.

Si la [suppression réversible](../../../backup/backup-azure-security-feature-cloud.md) est activée pour votre machine virtuelle, vous ne pouvez pas déplacer la machine virtuelle alors que ces points de restauration sont conservés. Vous pouvez [désactiver la suppression réversible](../../../backup/backup-azure-security-feature-cloud.md#enabling-and-disabling-soft-delete) ou attendre 14 jours après la suppression des points de restauration.

### <a name="portal"></a>Portail

1. Arrêtez temporairement la sauvegarde et conservez les données de sauvegarde.
2. Pour déplacer des machines virtuelles configurées avec Sauvegarde Azure, effectuez les étapes suivantes :

   1. Recherchez l’emplacement de votre machine virtuelle.
   2. Recherchez un groupe de ressources dont le modèle de nommage est le suivant : `AzureBackupRG_<VM location>_1`. Par exemple, le nom est au format *AzureBackupRG_westus2_1*.
   3. Dans le Portail Azure, cochez la case **Afficher les types masqués**.
   4. Recherchez la ressource de type **Microsoft. Microsoft.Compute/restorePointCollections** dont le modèle de nommage est `AzureBackup_<VM name>_###########`.
   5. Supprimez cette ressource. Cette opération supprime uniquement les points de récupération instantanée, et non les données sauvegardées dans le coffre.
   6. Une fois l’opération de suppression terminée, vous pouvez déplacer votre machine virtuelle.

3. Déplacez la machine virtuelle vers le groupe de ressources cible.
4. Reprenez la sauvegarde.

### <a name="powershell"></a>PowerShell

1. Recherchez l’emplacement de votre machine virtuelle.

1. Recherchez un groupe de ressources avec le modèle de nommage – `AzureBackupRG_<VM location>_1`. Par exemple, le nom peut être `AzureBackupRG_westus2_1`.

1. Si vous ne déplacez qu’une seule machine virtuelle, obtenez la collection de points de restauration pour cette machine virtuelle.

   ```azurepowershell-interactive
   $restorePointCollection = Get-AzResource -ResourceGroupName AzureBackupRG_<VM location>_1 -name AzureBackup_<VM name>* -ResourceType Microsoft.Compute/restorePointCollections
   ```

   Supprimez cette ressource. Cette opération supprime uniquement les points de récupération instantanée, et non les données sauvegardées dans le coffre.

   ```azurepowershell-interactive
   Remove-AzResource -ResourceId $restorePointCollection.ResourceId -Force
   ```

1. Si vous déplacez toutes les machines virtuelles avec des sauvegardes à cet emplacement, obtenez les collections de points de restauration pour ces machines virtuelles.

   ```azurepowershell-interactive
   $restorePointCollection = Get-AzResource -ResourceGroupName AzureBackupRG_<VM location>_1 -ResourceType Microsoft.Compute/restorePointCollections
   ```

   Supprimez chaque ressource. Cette opération supprime uniquement les points de récupération instantanée, et non les données sauvegardées dans le coffre.

   ```azurepowershell-interactive
   foreach ($restorePoint in $restorePointCollection)
   {
     Remove-AzResource -ResourceId $restorePoint.ResourceId -Force
   }
   ```

### <a name="azure-cli"></a>Azure CLI

1. Recherchez l’emplacement de votre machine virtuelle.

1. Recherchez un groupe de ressources avec le modèle de nommage – `AzureBackupRG_<VM location>_1`. Par exemple, le nom peut être `AzureBackupRG_westus2_1`.

1. Si vous ne déplacez qu’une seule machine virtuelle, obtenez la collection de points de restauration pour cette machine virtuelle.

   ```azurecli-interactive
   RESTOREPOINTCOL=$(az resource list -g AzureBackupRG_<VM location>_1 --resource-type Microsoft.Compute/restorePointCollections --query "[?starts_with(name, 'AzureBackup_<VM name>')].id" --output tsv)
   ```

   Supprimez cette ressource. Cette opération supprime uniquement les points de récupération instantanée, et non les données sauvegardées dans le coffre.

   ```azurecli-interactive
   az resource delete --ids $RESTOREPOINTCOL
   ```

1. Si vous déplacez toutes les machines virtuelles avec des sauvegardes à cet emplacement, obtenez les collections de points de restauration pour ces machines virtuelles.

   ```azurecli-interactive
   RESTOREPOINTCOL=$(az resource list -g AzureBackupRG_<VM location>_1 --resource-type Microsoft.Compute/restorePointCollections)
   ```

   Supprimez chaque ressource. Cette opération supprime uniquement les points de récupération instantanée, et non les données sauvegardées dans le coffre.

   ```azurecli-interactive
   az resource delete --ids $RESTOREPOINTCOL
   ```

## <a name="next-steps"></a>Étapes suivantes

* Pour obtenir les commandes permettant de déplacer des ressources, consultez [Déplacer des ressources vers un nouveau groupe de ressources ou un nouvel abonnement](../move-resource-group-and-subscription.md).

* Pour plus d’informations sur le déplacement des coffres Recovery Services pour la sauvegarde, consultez [Limitations de Recovery Services](../../../backup/backup-azure-move-recovery-services-vault.md?toc=/azure/azure-resource-manager/toc.json).
