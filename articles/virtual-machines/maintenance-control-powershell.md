---
title: Contrôle de maintenance des machines virtuelles Azure avec PowerShell
description: Découvrez comment contrôler le moment où la maintenance est appliquée à vos machines virtuelles Azure à l’aide du contrôle de maintenance et de PowerShell.
author: cynthn
ms.service: virtual-machines
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 11/19/2020
ms.author: cynthn
ms.openlocfilehash: 2cc935e81e867609159b5c150b6ee7c346bb9f8e
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95026146"
---
# <a name="control-updates-with-maintenance-control-and-azure-powershell"></a>Contrôler les mises à jour avec le contrôle de maintenance et Azure PowerShell

Le contrôle de maintenance vous permet de décider du moment où appliquer les mises à jour de la plateforme à l’infrastructure hôte de vos machines virtuelles isolées et vos hôtes dédiés Azure. Cette rubrique traite des options de contrôle de maintenance d’Azure PowerShell. Pour plus d’informations sur les avantages du contrôle de maintenance, ses limitations et d’autres options de gestion, consultez [Gestion des mises à jour de plateforme avec le contrôle de maintenance](maintenance-control.md).
 
## <a name="enable-the-powershell-module"></a>Activer le module PowerShell

Vérifiez que `PowerShellGet` est à jour.    

```azurepowershell-interactive  
Install-Module -Name PowerShellGet -Repository PSGallery -Force 
``` 

Installez le module PowerShell `Az.Maintenance`.     

```azurepowershell-interactive  
Install-Module -Name Az.Maintenance
``` 

Si vous installez localement, veillez à ouvrir votre invite PowerShell en tant qu’administrateur.

Vous pouvez également être invité à confirmer que vous souhaitez installer à partir d’un *référentiel non approuvé*. Tapez `Y` ou sélectionnez **Oui pour tout** afin d’installer le module.


## <a name="create-a-maintenance-configuration"></a>Créer une configuration de maintenance

Créez un groupe de ressources en tant que conteneur pour votre configuration. Dans cet exemple, un groupe de ressources nommé *myMaintenanceRG* est créé dans *eastus*. Si vous avez déjà un groupe de ressources et que vous souhaitez l’utiliser, vous pouvez ignorer cette partie et remplacer le nom du groupe de ressources par le vôtre dans le reste des exemples.

```azurepowershell-interactive
New-AzResourceGroup `
   -Location eastus `
   -Name myMaintenanceRG
```

Utilisez [New-AzMaintenanceConfiguration](/powershell/module/az.maintenance/new-azmaintenanceconfiguration) pour créer une configuration de maintenance. Cet exemple crée une configuration de maintenance nommée *myConfig*, étendue à l’hôte. 

```azurepowershell-interactive
$config = New-AzMaintenanceConfiguration `
   -ResourceGroup myMaintenanceRG `
   -Name myConfig `
   -MaintenanceScope host `
   -Location  eastus
```

L’utilisation de `-MaintenanceScope host` garantit l’utilisation de la configuration de maintenance pour contrôler les mises à jour de l’hôte.

Si vous essayez de créer une configuration portant le même nom mais dans un autre emplacement, vous obtenez une erreur. Les noms de configuration doivent être uniques dans votre groupe de ressources.

Vous pouvez rechercher les configurations de maintenance disponibles à l’aide de [Get-AzMaintenanceConfiguration](/powershell/module/az.maintenance/get-azmaintenanceconfiguration).

```azurepowershell-interactive
Get-AzMaintenanceConfiguration | Format-Table -Property Name,Id
```

### <a name="create-a-maintenance-configuration-with-scheduled-window"></a>Créer une configuration de maintenance avec une fenêtre planifiée

Vous pouvez également déclarer une fenêtre planifiée quand Azure applique les mises à jour sur vos ressources. Cet exemple crée une configuration de maintenance nommée myConfig avec une fenêtre planifiée de 5 heures, le quatrième lundi de chaque mois. Une fois que vous aurez créé une fenêtre planifiée, vous n’aurez plus besoin d’appliquer les mises à jour manuellement.

```azurepowershell-interactive
$config = New-AzMaintenanceConfiguration `
   -ResourceGroup $RGName `
   -Name $MaintenanceConfig `
   -MaintenanceScope Host `
   -Location $location `
   -StartDateTime "2020-10-01 00:00" `
   -TimeZone "Pacific Standard Time" `
   -Duration "05:00" `
   -RecurEvery "Month Fourth Monday"
```
> [!IMPORTANT]
> La **durée** de maintenance doit être de *2 heures* ou plus. La **récurrence** minimale de la maintenance doit être d’une fois tous les 35 jours.

La **récurrence** de la maintenance peut être exprimée quotidiennement, hebdomadairement ou mensuellement. Quelques exemples :
 - **quotidiennement** : RecurEvery « Jour » **ou** « 3Jours » 
 - **hebdomadairement** : RecurEvery « 3Jours » **ou** « Semaine samedi,dimanche » 
 - **mensuellement** : RecurEvery « Lundi jour23,jour24 » **ou** « Mois Dernier dimanche » **ou** « Mois Quatrième lundi »  
      

## <a name="assign-the-configuration"></a>Affecter la configuration

Utilisez [New-AzConfigurationAssignment](/powershell/module/az.maintenance/new-azconfigurationassignment) pour affecter la configuration à votre machine virtuelle isolée ou à Azure Dedicated Host.

### <a name="isolated-vm"></a>Machine virtuelle isolée

Appliquez la configuration à une machine virtuelle à l’aide de l’ID de la configuration. Spécifiez `-ResourceType VirtualMachines` et fournissez le nom de la machine virtuelle pour `-ResourceName` et le groupe de ressources de la machine virtuelle pour `-ResourceGroupName` . 

```azurepowershell-interactive
New-AzConfigurationAssignment `
   -ResourceGroupName myResourceGroup `
   -Location eastus `
   -ResourceName myVM `
   -ResourceType VirtualMachines `
   -ProviderName Microsoft.Compute `
   -ConfigurationAssignmentName $config.Name `
   -MaintenanceConfigurationId $config.Id
```

### <a name="dedicated-host"></a>Hôte dédié

Pour appliquer une configuration à un hôte dédié, vous devez aussi inclure `-ResourceType hosts`, `-ResourceParentName` avec le nom du groupe hôte et `-ResourceParentType hostGroups`. 


```azurepowershell-interactive
New-AzConfigurationAssignment `
   -ResourceGroupName myResourceGroup `
   -Location eastus `
   -ResourceName myHost `
   -ResourceType hosts `
   -ResourceParentName myHostGroup `
   -ResourceParentType hostGroups `
   -ProviderName Microsoft.Compute `
   -ConfigurationAssignmentName $config.Name `
   -MaintenanceConfigurationId $config.Id
```

## <a name="check-for-pending-updates"></a>Rechercher les mises à jour en attente

Utilisez [Get-AzMaintenanceUpdate](/powershell/module/az.maintenance/get-azmaintenanceupdate) pour voir s’il existe des mises à jour en attente. Utilisez `-subscription` pour spécifier l’abonnement Azure de la machine virtuelle s’il est différent de celui auquel vous êtes connecté.

S’il n’y a aucune mise à jour à afficher, cette commande ne retourne rien. Sinon, elle retourne un objet PSApplyUpdate :

```json
{
   "maintenanceScope": "Host",
   "impactType": "Freeze",
   "status": "Pending",
   "impactDurationInSec": 9,
   "notBefore": "2020-02-21T16:47:44.8728029Z",
   "properties": {
      "resourceId": "/subscriptions/39c6cced-4d6c-4dd5-af86-57499cd3f846/resourcegroups/Ignite2019/providers/Microsoft.Compute/virtualMachines/MCDemo3"
} 
```

### <a name="isolated-vm"></a>Machine virtuelle isolée

Recherchez les mises à jour en attente pour une machine virtuelle isolée. Dans cet exemple, la sortie est mise en forme en tant que tableau pour faciliter la lecture.

```azurepowershell-interactive
Get-AzMaintenanceUpdate `
  -ResourceGroupName myResourceGroup `
  -ResourceName myVM `
  -ResourceType VirtualMachines `
  -ProviderName Microsoft.Compute | Format-Table
```


### <a name="dedicated-host"></a>Hôte dédié

Pour rechercher les mises à jour en attente pour un hôte dédié. Dans cet exemple, la sortie est mise en forme en tant que tableau pour faciliter la lecture. Remplacez les valeurs des ressources par les vôtres.

```azurepowershell-interactive
Get-AzMaintenanceUpdate `
   -ResourceGroupName myResourceGroup `
   -ResourceName myHost `
   -ResourceType hosts `
   -ResourceParentName myHostGroup `
   -ResourceParentType hostGroups `
   -ProviderName Microsoft.Compute | Format-Table
```


## <a name="apply-updates"></a>Appliquer des mises à jour

Utilisez [New-AzApplyUpdate](/powershell/module/az.maintenance/new-azapplyupdate) pour appliquer les mises à jour en attente.

### <a name="isolated-vm"></a>Machine virtuelle isolée

Créez une demande d’application des mises à jour à une machine virtuelle isolée.

```azurepowershell-interactive
New-AzApplyUpdate `
   -ResourceGroupName myResourceGroup `
   -ResourceName myVM `
   -ResourceType VirtualMachines `
   -ProviderName Microsoft.Compute
```

En cas de réussite, cette commande retourne un objet `PSApplyUpdate`. Vous pouvez utiliser l’attribut Name dans la commande `Get-AzApplyUpdate` pour vérifier l’état de la mise à jour. Consultez [Vérifier l’état des mises à jour](#check-update-status).

### <a name="dedicated-host"></a>Hôte dédié

Appliquez les mises à jour à un hôte dédié.

```azurepowershell-interactive
New-AzApplyUpdate `
   -ResourceGroupName myResourceGroup `
   -ResourceName myHost `
   -ResourceType hosts `
   -ResourceParentName myHostGroup `
   -ResourceParentType hostGroups `
   -ProviderName Microsoft.Compute
```

## <a name="check-update-status"></a>Vérifier l’état de la mise à jour
Utilisez [Get-AzApplyUpdate](/powershell/module/az.maintenance/get-azapplyupdate) pour vérifier l’état d’une mise à jour. Les commandes indiquées ci-dessous indiquent l’état de la dernière mise à jour à l’aide de `default` pour le paramètre `-ApplyUpdateName`. Vous pouvez remplacer le nom de la mise à jour (retourné par la commande [New-AzApplyUpdate](/powershell/module/az.maintenance/new-azapplyupdate)) pour obtenir l’état d’une mise à jour spécifique.

```text
Status         : Completed
ResourceId     : /subscriptions/12ae7457-4a34-465c-94c1-17c058c2bd25/resourcegroups/TestShantS/providers/Microsoft.Comp
ute/virtualMachines/DXT-test-04-iso
LastUpdateTime : 1/1/2020 12:00:00 AM
Id             : /subscriptions/12ae7457-4a34-465c-94c1-17c058c2bd25/resourcegroups/TestShantS/providers/Microsoft.Comp
ute/virtualMachines/DXT-test-04-iso/providers/Microsoft.Maintenance/applyUpdates/default
Name           : default
Type           : Microsoft.Maintenance/applyUpdates
```
LastUpdateTime correspond à l’heure à laquelle la mise à jour est terminée, soit à votre initiative, soit à celle de la plateforme au cas où la fenêtre d’automaintenance n’aurait pas été utilisée. Si aucune mise à jour n’a jamais été appliquée par le biais du contrôle de maintenance, la valeur par défaut s’affiche.

### <a name="isolated-vm"></a>Machine virtuelle isolée

Recherchez les mises à jour d’une machine virtuelle spécifique.

```azurepowershell-interactive
Get-AzApplyUpdate `
   -ResourceGroupName myResourceGroup `
   -ResourceName myVM `
   -ResourceType VirtualMachines `
   -ProviderName Microsoft.Compute `
   -ApplyUpdateName default
```

### <a name="dedicated-host"></a>Hôte dédié

Recherchez les mises à jour d’un hôte dédié.

```azurepowershell-interactive
Get-AzApplyUpdate `
   -ResourceGroupName myResourceGroup `
   -ResourceName myHost `
   -ResourceType hosts `
   -ResourceParentName myHostGroup `
   -ResourceParentType hostGroups `
   -ProviderName Microsoft.Compute `
   -ApplyUpdateName myUpdateName
```

## <a name="remove-a-maintenance-configuration"></a>Supprimer une configuration de maintenance

Utilisez [Remove-AzMaintenanceConfiguration](/powershell/module/az.maintenance/remove-azmaintenanceconfiguration) pour supprimer une configuration de maintenance.

```azurepowershell-interactive
Remove-AzMaintenanceConfiguration `
   -ResourceGroupName myResourceGroup `
   -Name $config.Name
```

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations, consultez [Maintenance et mises à jour](maintenance-and-updates.md).
