---
title: Protection des instances pour les groupes de machines virtuelles identiques Azure
description: Découvrez comment protéger les instances de groupes de machines virtuelles identiques des opérations scale-in et scale-set.
author: avirishuv
ms.author: avverma
ms.topic: conceptual
ms.service: virtual-machine-scale-sets
ms.subservice: availability
ms.date: 02/26/2020
ms.reviewer: jushiman
ms.custom: avverma, devx-track-azurecli
ms.openlocfilehash: 59720a3e89db29ba3eb0829084dd5252e27e9cc9
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92746794"
---
# <a name="instance-protection-for-azure-virtual-machine-scale-set-instances"></a>Protection des instances pour les groupes de machines virtuelles identiques Azure

Les groupes de machines virtuelles identiques Azure offrent une meilleure élasticité en termes de charge de travail via la [mise à l’échelle automatique](virtual-machine-scale-sets-autoscale-overview.md) pour vous permettre de configurer la mise à l'échelle de votre infrastructure. En outre, les groupes identiques vous permettent de gérer, configurer et mettre à jour un grand nombre de machines virtuelles de manière centralisée à l'aide de différents paramètres de [stratégie de mise à niveau](virtual-machine-scale-sets-upgrade-scale-set.md#how-to-bring-vms-up-to-date-with-the-latest-scale-set-model). Vous pouvez configurer une mise à jour sur le modèle de groupe identique, et la nouvelle configuration s'applique automatiquement à chaque instance de groupe identique si vous avez défini la stratégie de mise à niveau sur Automatique ou Continue.

Alors que votre application traite le trafic, vous pouvez souhaiter que des instances spécifiques soient traitées différemment du reste de l'instance de groupe identique. Par exemple, si certaines instances du groupe identique effectuent des opérations de longue durée, vous préférerez peut-être attendre qu'elles soient terminées avant de mettre à l'échelle ces instances. Il est également possible que des instances spécialisées du groupe identique effectuent des tâches supplémentaires ou différentes de celles des autres membres du groupe. Et dès lors, vous pouvez ne pas souhaiter que ces machines virtuelles « spéciales » soient modifiées en même temps que les autres instances du groupe. La protection des instances met à disposition des contrôles supplémentaires pour permettre de tels scénarios.

Cet article vous explique comment appliquer et utiliser les différentes fonctionnalités de protection des instances avec les instances de groupe identique.

## <a name="types-of-instance-protection"></a>Types de protection des instances
Les groupes identiques fournissent deux types de fonctionnalités de protection des instances :

-   **Protection contre les opérations scale-in**
    - Activée par le biais de la propriété **protectFromScaleIn** sur l'instance de groupe identique
    - Protège l’instance contre les opérations scale-in initiées par la mise à l'échelle automatique
    - Les opérations d'instance initiées par l'utilisateur (y compris la suppression d'une instance) ne sont **pas bloquées**
    - Les opérations initiées sur le groupe identique (mise à niveau, réinitialisation, désallocation, etc.) ne sont **pas bloquées**

-   **Protection contre les actions de groupe identique**
    - Activée par le biais de la propriété **protectFromScaleSetActions** sur l'instance de groupe identique
    - Protège l’instance contre les opérations scale-in initiées par la mise à l'échelle automatique
    - Protège l’instance contre les opérations initiées sur le groupe identique (mise à niveau, réinitialisation, désallocation, etc.)
    - Les opérations d'instance initiées par l'utilisateur (y compris la suppression d'une instance) ne sont **pas bloquées**
    - La suppression du groupe identique dans son intégralité n'est **pas bloquée**

## <a name="protect-from-scale-in"></a>Protection contre les opérations scale-in
La protection des instances peut être appliquée aux instances de groupe identique une fois ces instances créées. La protection est uniquement appliquée et modifiée sur le [modèle d’instance](virtual-machine-scale-sets-upgrade-scale-set.md#the-scale-set-vm-model-view) et non sur le [modèle de groupe identique](virtual-machine-scale-sets-upgrade-scale-set.md#the-scale-set-model).

Il existe plusieurs façons d'appliquer la protection contre les opérations scale-in sur vos instances de groupe identique, comme détaillé dans les exemples ci-dessous.

### <a name="azure-portal"></a>Portail Azure

Vous pouvez appliquer une protection contre les opérations scale-in à une instance du groupe identique via le portail Azure. Vous ne pouvez pas ajuster plusieurs instances à la fois. Répétez les étapes pour chacune des instances que vous souhaitez protéger.
 
1. Accédez à un groupe de machines virtuelles identiques existant.
1. Sélectionnez **Instances** dans le menu de gauche, sous **Paramètres** .
1. Sélectionnez le nom de l'instance à protéger.
1. Sélectionnez l'onglet **Stratégie de protection** .
1. Dans le panneau **Stratégie de protection** , sélectionnez l'option **Protection contre les opérations scale-in** .
1. Sélectionnez **Enregistrer** . 

### <a name="rest-api"></a>API REST

L’exemple suivant applique une protection contre les opérations scale-in à une instance du groupe identique.

```
PUT on `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmScaleSetName}/virtualMachines/{instance-id}?api-version=2019-03-01`
```

```json
{
  "properties": {
    "protectionPolicy": {
      "protectFromScaleIn": true
    }
  }        
}

```

> [!NOTE]
>La protection des instances est uniquement prise en charge par la version 2019-03-01 de l'API (ou une version ultérieure)

### <a name="azure-powershell"></a>Azure PowerShell

Utilisez la cmdlet [Update-AzVmssVM](/powershell/module/az.compute/update-azvmssvm) pour appliquer la protection contre les opérations scale-in à votre instance de groupe identique.

L’exemple suivant applique la protection contre les opérations scale-in à une instance du groupe identique contenant l'ID d'instance 0.

```azurepowershell-interactive
Update-AzVmssVM `
  -ResourceGroupName "myResourceGroup" `
  -VMScaleSetName "myVMScaleSet" `
  -InstanceId 0 `
  -ProtectFromScaleIn $true
```

### <a name="azure-cli-20"></a>Azure CLI 2.0

Utilisez [az vmss update](/cli/azure/vmss#az-vmss-update) pour appliquer la protection contre les opérations scale-in à votre instance de groupe identique.

L’exemple suivant applique la protection contre les opérations scale-in à une instance du groupe identique contenant l'ID d'instance 0.

```azurecli-interactive
az vmss update \  
  --resource-group <myResourceGroup> \
  --name <myVMScaleSet> \
  --instance-id 0 \
  --protect-from-scale-in true
```

## <a name="protect-from-scale-set-actions"></a>Protection contre les actions de groupe identique
La protection des instances peut être appliquée aux instances de groupe identique une fois ces instances créées. La protection est uniquement appliquée et modifiée sur le [modèle d’instance](virtual-machine-scale-sets-upgrade-scale-set.md#the-scale-set-vm-model-view) et non sur le [modèle de groupe identique](virtual-machine-scale-sets-upgrade-scale-set.md#the-scale-set-model).

Protéger une instance contre les actions de groupe identique protège aussi l'instance contre les opérations scale-in initiées par la mise à l'échelle automatique.

Il existe plusieurs façons d'appliquer la protection contre les actions de groupe identique sur vos instances de groupe identique, comme détaillé dans les exemples ci-dessous.

### <a name="azure-portal"></a>Portail Azure

Vous pouvez appliquer une protection contre les actions de groupe identique à une instance du groupe identique via le portail Azure. Vous ne pouvez pas ajuster plusieurs instances à la fois. Répétez les étapes pour chacune des instances que vous souhaitez protéger.
 
1. Accédez à un groupe de machines virtuelles identiques existant.
1. Sélectionnez **Instances** dans le menu de gauche, sous **Paramètres** .
1. Sélectionnez le nom de l'instance à protéger.
1. Sélectionnez l'onglet **Stratégie de protection** .
1. Dans le panneau **Stratégie de protection** , sélectionnez l'option **Protection contre les actions de groupe identique** .
1. Sélectionnez **Enregistrer** . 

### <a name="rest-api"></a>API REST

L’exemple suivant applique une protection contre les actions de groupe identique à une instance du groupe identique.

```
PUT on `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vMScaleSetName}/virtualMachines/{instance-id}?api-version=2019-03-01`
```

```json
{
  "properties": {
    "protectionPolicy": {
      "protectFromScaleIn": true,
      "protectFromScaleSetActions": true
    }
  }        
}

```

> [!NOTE]
>La protection des instances est uniquement prise en charge par la version 2019-03-01 de l'API (ou une version ultérieure).</br>
Protéger une instance contre les actions de groupe identique protège aussi l'instance contre les opérations scale-in initiées par la mise à l'échelle automatique. Vous ne pouvez pas spécifier « protectFromScaleIn » : false lorsque vous définissez « protectFromScaleSetActions » : true

### <a name="azure-powershell"></a>Azure PowerShell

Utilisez la cmdlet [Update-AzVmssVM](/powershell/module/az.compute/update-azvmssvm) pour appliquer la protection contre les actions de groupe identique à votre instance de groupe identique.

L’exemple suivant applique une protection contre les actions de groupe identique à une instance du groupe identique contenant l'ID d'instance 0.

```azurepowershell-interactive
Update-AzVmssVM `
  -ResourceGroupName "myResourceGroup" `
  -VMScaleSetName "myVMScaleSet" `
  -InstanceId 0 `
  -ProtectFromScaleIn $true `
  -ProtectFromScaleSetAction $true
```

### <a name="azure-cli-20"></a>Azure CLI 2.0

Utilisez [az vmss update](/cli/azure/vmss#az-vmss-update) pour appliquer la protection contre les actions de groupe identique à votre instance de groupe identique.

L’exemple suivant applique une protection contre les actions de groupe identique à une instance du groupe identique contenant l'ID d'instance 0.

```azurecli-interactive
az vmss update \  
  --resource-group <myResourceGroup> \
  --name <myVMScaleSet> \
  --instance-id 0 \
  --protect-from-scale-in true \
  --protect-from-scale-set-actions true
```

## <a name="troubleshoot"></a>Dépanner
### <a name="no-protectionpolicy-on-scale-set-model"></a>protectionPolicy ne s'applique pas au modèle de groupe identique
La protection des instances s'applique uniquement aux instances de groupe identique, et pas au modèle de groupe identique.

### <a name="no-protectionpolicy-on-scale-set-instance-model"></a>protectionPolicy ne s'applique pas au modèle d'instance de groupe identique
Par défaut, la stratégie de protection ne s'applique pas à une instance au moment de sa création.

Vous pouvez appliquer la protection des instances aux instances de groupe identique une fois ces instances créées.

### <a name="not-able-to-apply-instance-protection"></a>Impossible d’appliquer la protection des instances
La protection des instances est uniquement prise en charge par la version 2019-03-01 de l'API (ou une version ultérieure). Vérifiez la version de l'API utilisée et mettez-la à jour, si besoin. Vous serez peut-être également amené à mettre à jour PowerShell ou l'interface CLI vers la dernière version.

## <a name="next-steps"></a>Étapes suivantes
Découvrez comment [déployer votre application](virtual-machine-scale-sets-deploy-app.md) sur des groupes de machines virtuelles identiques.
