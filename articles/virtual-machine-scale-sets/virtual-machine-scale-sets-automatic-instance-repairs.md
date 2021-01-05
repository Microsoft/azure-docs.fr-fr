---
title: Réparations automatiques d’instances avec les groupes de machines virtuelles identiques Azure
description: Découvrez comment configurer la stratégie de réparation automatique pour les instances de machine virtuelle dans un groupe identique
author: avirishuv
ms.author: avverma
ms.topic: conceptual
ms.service: virtual-machine-scale-sets
ms.subservice: availability
ms.date: 02/28/2020
ms.reviewer: jushiman
ms.custom: avverma, devx-track-azurecli
ms.openlocfilehash: ae508754775d4eb622d8e91ef58eb0d6e1c45692
ms.sourcegitcommit: 230d5656b525a2c6a6717525b68a10135c568d67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94889012"
---
# <a name="automatic-instance-repairs-for-azure-virtual-machine-scale-sets"></a>Réparations automatiques d’instances pour les groupes de machines virtuelles identiques Azure

L’activation des réparations automatiques d’instance pour les groupes de machines virtuelles identiques Azure permet d’obtenir une haute disponibilité pour les applications en conservant un ensemble d’instances saines. Si une instance du groupe identique n’est pas saine, comme indiqué par l’[extension d’intégrité de l’application](./virtual-machine-scale-sets-health-extension.md) ou les [sondes d’intégrité de l’équilibreur de charge](../load-balancer/load-balancer-custom-probe-overview.md), cette fonctionnalité effectue automatiquement une réparation de l’instance en supprimant l’instance non saine et en en créant une nouvelle pour la remplacer.

## <a name="requirements-for-using-automatic-instance-repairs"></a>Conditions requises pour l’utilisation de réparations automatiques d’instances

**Activer l’analyse de l’intégrité des applications pour le groupe identique**

Le groupe identique doit permettre l’analyse de l’intégrité des applications pour les instances activées. Pour ce faire, vous pouvez utiliser l’[extension d’intégrité d’application](./virtual-machine-scale-sets-health-extension.md) ou les [sondes d’intégrité de l’équilibreur de charge](../load-balancer/load-balancer-custom-probe-overview.md). Un seul de ces outils peut être activé à la fois. L’extension d’intégrité d’application ou les sondes de l’équilibreur de charge effectuent un test ping sur le point de terminaison d’application configuré sur les instances de machine virtuelle pour déterminer l’état d’intégrité de l’application. Cet état d’intégrité est utilisé par l’orchestrateur du groupe identique pour analyser l’intégrité de l’instance et effectuer des réparations quand cela est nécessaire.

**Configurer un point de terminaison pour fournir l’état d’intégrité**

Avant d’activer la stratégie de réparation automatique d’instances, vérifiez que le point de terminaison d’application est configuré pour les instances du groupe identique afin d’émettre l’état d’intégrité de l’application. Lorsqu’une instance retourne l’état 200 (OK) sur ce point de terminaison d’application, l’instance est marquée comme « saine ». Dans tous les autres cas, l’instance est marquée comme « non saine », y compris dans les scénarios suivants :

- Lorsqu’aucun point de terminaison d’application n’est configuré à l’intérieur des instances de machine virtuelle pour fournir l’état d’intégrité de l’application
- Lorsque le point de terminaison d’application n’est pas configuré correctement
- Lorsque le point de terminaison d’application n’est pas accessible

Pour les instances marquées comme non saines, des réparations automatiques sont déclenchées par le groupe identique. Assurez-vous que le point de terminaison d’application est correctement configuré avant d’activer la stratégie de réparation automatique afin d’éviter les réparations d’instances involontaires pendant la configuration du point de terminaison.

**Nombre maximal d'instances dans le groupe identique**

Cette fonction n'est actuellement disponible que pour les groupes identiques composés d'un maximum de 200 instances. Le groupe identique peut être déployé en tant que groupe à placement unique ou groupe à placements multiples, mais le nombre d'instances ne peut pas être supérieur à 200 si les réparations automatiques d'instances sont activées pour le groupe identique.

**Version d’API**

La stratégie de réparation automatique est prise en charge pour l’API de calcul version 2018-10-01 ou ultérieure.

**Restrictions relatives aux déplacements de ressources ou d’abonnements**

Les déplacements de ressources ou d’abonnements ne sont actuellement pas pris en charge pour les groupes identiques lorsque la fonctionnalité de réparation automatique est activée.

**Restriction pour les groupes identiques Service Fabric**

Cette fonctionnalité n’est actuellement pas prise en charge pour les groupes identiques Service Fabric.

## <a name="how-do-automatic-instance-repairs-work"></a>Comment fonctionnent les réparations automatiques d’instances ?

La fonctionnalité de réparation automatique d’instances s’appuie sur l’analyse de l’intégrité d’instances individuelles dans un groupe identique. Les instances de machine virtuelle dans un groupe identique peuvent être configurées de façon à émettre l’état d’intégrité de l’application à l’aide de l’[extension d’intégrité d’application](./virtual-machine-scale-sets-health-extension.md) ou des [sondes d’intégrité de l’équilibreur de charge](../load-balancer/load-balancer-custom-probe-overview.md). Si une instance est détectée non saine, le groupe identique effectue une action de réparation en supprimant l’instance non saine et en en créant une nouvelle pour la remplacer. Le dernier modèle de groupe de machines virtuelles identiques est utilisé pour créer la nouvelle instance. Cette fonctionnalité peut être activée dans le modèle de groupe de machines virtuelles identiques à l’aide de l’objet *automaticRepairsPolicy*.

### <a name="batching"></a>Traitement par lot

Les opérations de réparation automatique d’instances sont effectuées par lots. À un moment donné, pas plus de 5 % des instances du groupe identique sont réparées par la stratégie de réparation automatique. Cela permet d’éviter la suppression et la recréation simultanées d’un grand nombre d’instances si elles sont détectées non saines en même temps.

### <a name="grace-period"></a>Période de grâce

Lorsqu’une instance passe par une opération de changement d’état en raison d’une action PUT, PATCH ou POST effectuée sur le groupe identique (par exemple, une réinitialisation, un redéploiement, une mise à jour, etc.), toute action de réparation sur cette instance est exécutée uniquement après l’attente de la période de grâce. La période de grâce correspond à la durée nécessaire pour permettre à l’instance de retrouver son état d’intégrité. La période de grâce commence une fois que le changement d’état est terminé. Cela permet d’éviter toute opération de réparation prématurée ou accidentelle. La période de grâce est honorée pour toute instance nouvellement créée dans le groupe identique (y compris celle créée à la suite d’une opération de réparation). La période de grâce est spécifiée en minutes au format ISO 8601 et peut être définie à l’aide de la propriété *automaticRepairsPolicy.gracePeriod*. La période de grâce peut être comprise entre 30 minutes et 90 minutes, 30 minutes étant la valeur par défaut.

### <a name="suspension-of-repairs"></a>Interruption des réparations 

Les groupes de machines virtuelles identiques permettent de suspendre temporairement des réparations automatiques d’instances si nécessaire. Le paramètre *serviceState* pour les réparations automatiques sous la propriété *orchestrationServices* dans la vue des instances du groupe de machines virtuelles identiques indique l’état actuel des réparations automatiques. Lorsqu’un groupe identique est paramétré pour des réparations automatiques, la valeur du paramètre *serviceState* est définie sur *Running*. Lorsque les réparations automatiques sont interrompues pour un groupe identique, le paramètre *serviceState* est défini sur *Suspended*. Si *automaticRepairsPolicy* est défini sur un groupe identique, mais que la fonctionnalité des réparations automatiques n’est pas activée, le paramètre *serviceState* est défini sur *Not Running*.

Si des instances nouvellement créées destinées à remplacer les instances non saines d’un groupe identique continuent d’être non saines, même après des opérations de réparation répétées, alors, par mesure de sécurité, la plateforme met à jour le *serviceState* pour les réparations automatiques sur *Suspended*. Vous pouvez reprendre à nouveau les réparations automatiques en définissant la valeur de *serviceState* pour les réparations automatiques sur *Running*. Des instructions détaillées sont fournies dans la section relative [à l’affichage et à la mise à jour de l’état du service de la stratégie de réparation automatique](#viewing-and-updating-the-service-state-of-automatic-instance-repairs-policy) pour votre groupe identique. 

Le processus de réparation automatique d’instances fonctionne comme suit :

1. L’[extension d’intégrité d’application](./virtual-machine-scale-sets-health-extension.md) ou les [sondes d’intégrité de l’équilibreur de charge](../load-balancer/load-balancer-custom-probe-overview.md) effectuent un test ping sur le point de terminaison d’application à l’intérieur de chaque machine virtuelle du groupe identique pour obtenir l’état d’intégrité d’application de chaque instance.
2. Si le point de terminaison répond avec l’état 200 (OK), l’instance est marquée comme « saine ». Dans tous les autres cas (y compris si le point de terminaison est inaccessible), l’instance est marquée comme « non saine ».
3. Lorsqu’une instance est détectée non saine, le groupe identique effectue une action de réparation en supprimant l’instance non saine et en en créant une nouvelle pour la remplacer.
4. Les réparations d’instances sont effectuées par lots. À un moment donné, pas plus de 5 % du nombre total d’instances du groupe identique sont réparées. Si un groupe identique comporte moins de 20 instances, les réparations sont effectuées sur une instance non saine à la fois.
5. Le processus ci-dessus se poursuit jusqu’à ce que toutes les instances non saines dans le groupe identique soient réparées.

## <a name="instance-protection-and-automatic-repairs"></a>Protection d’instance et réparations automatiques

Si l’instance d’un groupe identique est protégée par l’une des [stratégies de protection](./virtual-machine-scale-sets-instance-protection.md), les réparations automatiques ne sont pas effectuées sur cette instance. Cela s’applique aux deux stratégies de protection : les actions *Protection contre les opérations scale-in* et *Protection contre les opérations de groupe identique*. 

## <a name="terminatenotificationandautomaticrepairs"></a>Notification d’arrêt et réparations automatiques

Si la fonctionnalité de [notification d’arrêt](./virtual-machine-scale-sets-terminate-notification.md) est activée sur un groupe identique, pendant l’opération de réparation automatique, la suppression d’une instance non saine suit la configuration de la notification d’arrêt. Une notification d’arrêt est envoyée via le service de métadonnées Azure (événements planifiés), et la suppression d’instance est retardée pendant la durée du délai configuré. Toutefois, la création d’une nouvelle instance pour remplacer celle qui est non saine n’attend pas la fin du délai.

## <a name="enabling-automatic-repairs-policy-when-creating-a-new-scale-set"></a>Activation de la stratégie de réparation automatique lors de la création d’un groupe identique

Pour activer la stratégie de réparation automatique lors de la création d’un groupe identique, assurez-vous que toutes les [conditions requises](#requirements-for-using-automatic-instance-repairs) pour l’adhésion à cette fonctionnalité sont remplies. Le point de terminaison d’application doit être correctement configuré pour les instances de groupe identique afin d’éviter de déclencher des réparations involontaires pendant la configuration du point de terminaison. Pour les groupes identiques nouvellement créés, toute réparation d’instance est effectuée uniquement après l’expiration de la période de grâce. Pour activer la réparation automatique d’instances dans un groupe identique, utilisez l’objet *automaticRepairsPolicy* dans le modèle de groupe de machines virtuelles identiques.

Vous pouvez également utiliser ce [modèle de démarrage rapide](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-automatic-repairs-slb-health-probe) pour déployer un groupe de machines virtuelles identiques dont la sonde d’intégrité de l’équilibreur de charge et les réparations automatiques d’instances sont activées avec une période de grâce de 30 minutes.

### <a name="azure-portal"></a>Portail Azure
 
Les étapes suivantes permettent d’activer la stratégie de réparation automatique lors de la création d’un groupe identique.
 
1. Accédez à **Groupes identiques de machines virtuelles**.
1. Sélectionnez **+ Ajouter** pour créer un groupe identique.
1. Accédez à l’onglet **Intégrité**. 
1. Recherchez la section **Intégrité**.
1. Activez l’option **Superviser l’intégrité de l’application**.
1. Recherchez la section **Stratégie de réparation automatique**.
1. **Activez** l’option **Réparations automatiques**.
1. Dans **Période de grâce (min)** , spécifiez la période de grâce en minutes ; les valeurs autorisées sont comprises entre 30 et 90 minutes. 
1. Lorsque vous avez fini de créer le groupe identique, sélectionnez le bouton **Vérifier + créer**.

### <a name="rest-api"></a>API REST

L’exemple suivant montre comment activer la réparation automatique d’instances dans un modèle de groupe identique. Utilisez la version d’API 2018-10-01 ou ultérieure.

```
PUT or PATCH on '/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmScaleSetName}?api-version=2019-07-01'
```

```json
{
  "properties": {
    "automaticRepairsPolicy": {
            "enabled": "true",
            "gracePeriod": "PT30M"
        }
    }
}
```

### <a name="azure-powershell"></a>Azure PowerShell

La fonctionnalité de réparation automatique d’instances peut être activée lors de la création d’un groupe identique à l’aide de la cmdlet [New-AzVmssConfig](/powershell/module/az.compute/new-azvmssconfig). Cet exemple de script présente la création d’un groupe identique et les ressources associées à l’aide du fichier config : [Créez un groupe de machines virtuelles identiques complet](./scripts/powershell-sample-create-complete-scale-set.md). Vous pouvez configurer la stratégie de réparation automatique des instances en ajoutant les paramètres *EnableAutomaticRepair* et *AutomaticRepairGracePeriod* à l’objet de configuration servant à la création du groupe identique. L’exemple suivant active la fonctionnalité avec une période de grâce de 30 minutes.

```azurepowershell-interactive
New-AzVmssConfig `
 -Location "EastUS" `
 -SkuCapacity 2 `
 -SkuName "Standard_DS2" `
 -UpgradePolicyMode "Automatic" `
 -EnableAutomaticRepair $true `
 -AutomaticRepairGracePeriod "PT30M"
```

### <a name="azure-cli-20"></a>Azure CLI 2.0

L’exemple suivant active la stratégie de réparation automatique lors de la création d’un groupe identique à l’aide de la commande *[az vmss create](/cli/azure/vmss?view=azure-cli-latest#az-vmss-create)* . Commencez par créer un groupe de ressources, puis créez un groupe identique dont la période de grâce de la stratégie de réparation automatique est définie sur 30 minutes.

```azurecli-interactive
az group create --name <myResourceGroup> --location <VMSSLocation>
az vmss create \
  --resource-group <myResourceGroup> \
  --name <myVMScaleSet> \
  --image UbuntuLTS \
  --admin-username <azureuser> \
  --generate-ssh-keys \
  --load-balancer <existingLoadBalancer> \
  --health-probe <existingHealthProbeUnderLoaderBalancer> \
  --automatic-repairs-grace-period 30
```

L’exemple ci-dessus utilise un équilibreur de charge et une sonde d’intégrité existants pour surveiller l’état d’intégrité d’application des instances. Si vous préférez utiliser une extension d’intégrité d’application pour l’analyse, vous pouvez créer un groupe identique, configurer l’extension d’intégrité d’application, puis activer la stratégie de réparation automatique d’instances à l’aide de la commande *az vmss update*, comme expliqué dans la section suivante.

## <a name="enabling-automatic-repairs-policy-when-updating-an-existing-scale-set"></a>Activation de la stratégie de réparation automatique lors de la mise à jour d’un groupe identique existant

Avant d’activer la stratégie de réparation automatique dans un groupe identique existant, assurez-vous que toutes les [conditions requises](#requirements-for-using-automatic-instance-repairs) pour l’adhésion à cette fonctionnalité sont remplies. Le point de terminaison d’application doit être correctement configuré pour les instances de groupe identique afin d’éviter de déclencher des réparations involontaires pendant la configuration du point de terminaison. Pour activer la réparation automatique d’instances dans un groupe identique, utilisez l’objet *automaticRepairsPolicy* dans le modèle de groupe de machines virtuelles identiques.

Après la mise à jour du modèle d’un groupe identique existant, vérifiez que le modèle le plus récent est appliqué à toutes les instances du groupe identique. Reportez-vous aux instructions relatives à la [mise à jour les machines virtuelles avec le dernier modèle du groupe identique](./virtual-machine-scale-sets-upgrade-scale-set.md#how-to-bring-vms-up-to-date-with-the-latest-scale-set-model).

### <a name="azure-portal"></a>Portail Azure

Vous pouvez modifier la stratégie de réparation automatique d’un groupe identique via le Portail Azure. 
 
1. Accédez à un groupe de machines virtuelles identiques existant.
1. Sous **Paramètres** dans le menu de gauche, sélectionnez **Intégrité et réparation**.
1. Activez l’option **Superviser l’intégrité de l’application**.
1. Recherchez la section **Stratégie de réparation automatique**.
1. **Activez** l’option **Réparations automatiques**.
1. Dans **Période de grâce (min)** , spécifiez la période de grâce en minutes ; les valeurs autorisées sont comprises entre 30 et 90 minutes. 
1. Lorsque vous avez terminé, sélectionnez **Enregistrer**. 

### <a name="rest-api"></a>API REST

L’exemple suivant active la stratégie avec une période de grâce de 40 minutes. Utilisez la version d’API 2018-10-01 ou ultérieure.

```
PUT or PATCH on '/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmScaleSetName}?api-version=2019-07-01'
```

```json
{
  "properties": {
    "automaticRepairsPolicy": {
            "enabled": "true",
            "gracePeriod": "PT40M"
        }
    }
}
```

### <a name="azure-powershell"></a>Azure PowerShell

Utilisez la cmdlet [Update-AzVmss](/powershell/module/az.compute/update-azvmss) pour modifier la configuration de la fonctionnalité de réparation automatique d’instances dans un groupe identique existant. L’exemple suivant met à jour la période de grâce à 40 minutes.

```azurepowershell-interactive
Update-AzVmss `
 -ResourceGroupName "myResourceGroup" `
 -VMScaleSetName "myScaleSet" `
 -EnableAutomaticRepair $true `
 -AutomaticRepairGracePeriod "PT40M"
```

### <a name="azure-cli-20"></a>Azure CLI 2.0

Voici un exemple de mise à jour de la stratégie de réparation automatique d’instances d’un groupe identique existant, à l’aide de la commande *[az vmss update](/cli/azure/vmss?view=azure-cli-latest#az-vmss-update)* .

```azurecli-interactive
az vmss update \  
  --resource-group <myResourceGroup> \
  --name <myVMScaleSet> \
  --enable-automatic-repairs true \
  --automatic-repairs-grace-period 30
```

## <a name="viewing-and-updating-the-service-state-of-automatic-instance-repairs-policy"></a>Affichage et mise à jour de l’état du service de la stratégie de réparation automatique d’instances

### <a name="rest-api"></a>API REST 

Utilisez [Get Instance View](/rest/api/compute/virtualmachinescalesets/getinstanceview) avec la version d’API 2019-12-01 ou ultérieure pour que le groupe de machines virtuelles identiques affichage le paramètre *serviceState* pour les réparations automatiques sous la propriété *orchestrationServices*. 

```http
GET '/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmScaleSetName}/instanceView?api-version=2019-12-01'
```

```json
{
  "orchestrationServices": [
    {
      "serviceName": "AutomaticRepairs",
      "serviceState": "Running"
    }
  ]
}
```

Utilisez l’API *setOrchestrationServiceState* avec la version d’API 2019-12-01 ou ultérieure sur un groupe de machines virtuelles identiques pour définir l’état des réparations automatiques. Une fois que le groupe identique est paramétré pour la fonctionnalité de réparation automatique, vous pouvez utiliser cette API pour interrompre ou reprendre les réparations automatiques pour votre groupe identique. 

 ```http
 POST '/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmScaleSetName}/setOrchestrationServiceState?api-version=2019-12-01'
 ```

```json
{
  "orchestrationServices": [
    {
      "serviceName": "AutomaticRepairs",
      "serviceState": "Suspend"
    }
  ]
}
```

### <a name="azure-cli"></a>Azure CLI 

Utilisez la cmdlet [get-instance-view](/cli/azure/vmss?view=azure-cli-latest#az-vmss-get-instance-view) pour afficher le paramètre *serviceState* pour les réparations automatiques d’instances. 

```azurecli-interactive
az vmss get-instance-view \
    --name MyScaleSet \
    --resource-group MyResourceGroup
```

Utilisez la cmdlet [set-orchestration-service-state](/cli/azure/vmss?view=azure-cli-latest#az-vmss-set-orchestration-service-state) pour mettre à jour le paramètre *serviceState* pour les automatiques d’instances. Une fois que le groupe identique est paramétré pour la fonctionnalité de réparation automatique, vous pouvez utiliser cette cmdlet pour interrompre ou reprendre les réparations automatiques pour votre groupe identique. 

```azurecli-interactive
az vmss set-orchestration-service-state \
    --service-name AutomaticRepairs \
    --action Resume \
    --name MyScaleSet \
    --resource-group MyResourceGroup
```
### <a name="azure-powershell"></a>Azure PowerShell

Utilisez la cmdlet [Get-AzVmss](/powershell/module/az.compute/get-azvmss?view=azps-3.7.0) avec la paramètre *InstanceView* pour afficher le paramètre *serviceState* pour les réparations automatiques d’instances.

```azurepowershell-interactive
Get-AzVmss `
    -ResourceGroupName "myResourceGroup" `
    -VMScaleSetName "myScaleSet" `
    -InstanceView
```

Utilisez la cmdlet Set-AzVmssOrchestrationServiceState pour mettre à jour le paramètre *serviceState* pour les réparations automatiques d’instances. Une fois que le groupe identique est paramétré pour la fonctionnalité de réparation automatique, vous pouvez utiliser cette cmdlet pour interrompre ou reprendre les réparations automatiques pour votre groupe identique.

```azurepowershell-interactive
Set-AzVmssOrchestrationServiceState `
    -ResourceGroupName "myResourceGroup" `
    -VMScaleSetName "myScaleSet" `
    -ServiceName "AutomaticRepairs" `
    -Action "Suspend"
```

## <a name="troubleshoot"></a>Dépanner

**Échec de l’activation de la stratégie de réparation automatique**

Si vous obtenez une erreur « BadRequest » avec un message indiquant que le membre « automaticRepairsPolicy » est introuvable sur l’objet de type « properties », vérifiez la version de l’API utilisée pour le groupe de machines virtuelles identiques. La version d’API 2018-10-01 ou ultérieure est requise pour cette fonctionnalité.

**L’instance n’est pas réparée même lorsque la stratégie est activée**

L’instance peut être en période de grâce. Il s’agit de la durée d’attente après toute modification d’état sur l’instance avant d’effectuer des réparations. Cela permet d’éviter toute réparation prématurée ou accidentelle. L’action de réparation doit se produire une fois que la période de grâce est terminée pour l’instance.

**Affichage de l’état d’intégrité d’application pour les instances de groupe identique**

Vous pouvez utiliser l’[API Get Instance View](/rest/api/compute/virtualmachinescalesetvms/getinstanceview) pour les instances d’un groupe de machines virtuelles identiques pour afficher l’état d’intégrité de l’application. Avec Azure PowerShell, vous pouvez utiliser la cmdlet [Get-AzVmssVM](/powershell/module/az.compute/get-azvmssvm) avec l’indicateur *-InstanceView*. L’état d’intégrité de l’application est fourni sous la propriété *vmHealth*.

Dans le Portail Azure, vous pouvez également voir l’état d’intégrité. Accédez à un groupe identique existant, sélectionnez **Instances** dans le menu de gauche, puis examinez la colonne **État d’intégrité** pour déterminer l’état d’intégrité de chaque instance de groupe identique. 

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment configurer l’[extension d’intégrité d’application](./virtual-machine-scale-sets-health-extension.md) ou les [sondes d’intégrité de l’équilibreur de charge](../load-balancer/load-balancer-custom-probe-overview.md) pour vos groupes identiques.
