---
title: Modifier un groupe de machines virtuelles identiques Azure
description: Découvrir comment modifier et à mettre à jour un groupe de machines virtuelles identiques Azure avec les API REST, Azure PowerShell et Azure CLI
author: ju-shim
ms.author: jushiman
ms.topic: how-to
ms.service: virtual-machine-scale-sets
ms.subservice: management
ms.date: 03/10/2020
ms.reviewer: mimckitt
ms.custom: mimckitt, devx-track-azurecli
ms.openlocfilehash: c255a3d68b1a24e25c1c0e308faa3fd364a15861
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97358739"
---
# <a name="modify-a-virtual-machine-scale-set"></a>Modifier un groupe de machines virtuelles identiques

Tout au long du cycle de vie de vos applications, vous pourrez avoir besoin de modifier ou de mettre à jour votre groupe de machines virtuelles identiques. Ces mises à jour peuvent être liées à la configuration du groupe de machines ou à la modification de la configuration de l’application. Cet article décrit comment modifier un groupe identique avec les API REST, Azure PowerShell ou Azure CLI.

## <a name="fundamental-concepts"></a>Concepts fondamentaux

### <a name="the-scale-set-model"></a>Modèle du groupe identique
Un groupe identique est associé à un modèle qui capture l’état *souhaité* du groupe identique dans son ensemble. Pour interroger le modèle d’un groupe identique, vous pouvez utiliser les 

- API REST avec [compute/virtualmachinescalesets/get](/rest/api/compute/virtualmachinescalesets/get) comme suit :

    ```rest
    GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet?api-version={apiVersion}
    ```

- Azure PowerShell avec [Get-AzVmss](/powershell/module/az.compute/get-azvmss) :

    ```powershell
    Get-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet"
    ```

- Azure CLI avec [az vmss show](/cli/azure/vmss) :

    ```azurecli
    az vmss show --resource-group myResourceGroup --name myScaleSet
    ```

- Vous pouvez également utiliser [resources.azure.com](https://resources.azure.com) ou les [kits de développement logiciel (SDK) Azure](https://azure.microsoft.com/downloads/) propres à un langage.

La sortie exacte obtenue varie selon les options que vous ajoutez à la commande. L’exemple suivant représente une sortie condensée de l’interface Azure CLI :

```azurecli
az vmss show --resource-group myResourceGroup --name myScaleSet
{
  "location": "westus",
  "overprovision": true,
  "plan": null,
  "singlePlacementGroup": true,
  "sku": {
    "additionalProperties": {},
    "capacity": 1,
    "name": "Standard_D2_v2",
    "tier": "Standard"
  },
}
```

Ces propriétés s’appliquent à l’ensemble du groupe identique.


### <a name="the-scale-set-instance-view"></a>Vue d’instance du groupe identique
Un groupe identique est également associé à une vue d’instance de groupe identique. Cette vue capture l’état actuel de *l’exécution* du groupe identique dans son ensemble. Pour interroger la vue d’instance d’un groupe identique, vous pouvez utiliser :

- Les API REST avec [compute/virtualmachinescalesets/getinstanceview](/rest/api/compute/virtualmachinescalesets/getinstanceview) comme suit :

    ```rest
    GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet/instanceView?api-version={apiVersion}
    ```

- Azure PowerShell avec [Get-AzVmss](/powershell/module/az.compute/get-azvmss) :

    ```powershell
    Get-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceView
    ```

- Azure CLI avec [az vmss get-instance-view](/cli/azure/vmss) :

    ```azurecli
    az vmss get-instance-view --resource-group myResourceGroup --name myScaleSet
    ```

- Vous pouvez également utiliser [resources.azure.com](https://resources.azure.com) ou les [kits de développement logiciel (SDK) Azure](https://azure.microsoft.com/downloads/) propres à un langage.

La sortie exacte obtenue varie selon les options que vous ajoutez à la commande. L’exemple suivant représente une sortie condensée de l’interface Azure CLI :

```azurecli
$ az vmss get-instance-view --resource-group myResourceGroup --name myScaleSet
{
  "statuses": [
    {
      "additionalProperties": {},
      "code": "ProvisioningState/succeeded",
      "displayStatus": "Provisioning succeeded",
      "level": "Info",
      "message": null,
      "time": "{time}"
    }
  ],
  "virtualMachine": {
    "additionalProperties": {},
    "statusesSummary": [
      {
        "additionalProperties": {},
        "code": "ProvisioningState/succeeded",
        "count": 1
      }
    ]
  }
}
```

Ces propriétés fournissent un récapitulatif de l’état actuel de l’exécution des machines virtuelles du groupe identique, notamment l’état des extensions appliquées au groupe identique.


### <a name="the-scale-set-vm-model-view"></a>Vue de modèle des machines virtuelles du groupe identique
Comme le groupe identique, chaque instance de machine virtuelle a sa propre vue de modèle. Pour interroger la vue de modèle d’une instance de machine virtuelle d’un groupe identique, vous pouvez utiliser :

- Les API REST API avec [compute/virtualmachinescalesetvms/get](/rest/api/compute/virtualmachinescalesetvms/get) comme suit :

    ```rest
    GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet/virtualmachines/instanceId?api-version={apiVersion}
    ```

- Azure PowerShell avec [Get-AzVmssVm](/powershell/module/az.compute/get-azvmssvm) :

    ```powershell
    Get-AzVmssVm -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId instanceId
    ```

- Azure CLI avec [az vmss show](/cli/azure/vmss) :

    ```azurecli
    az vmss show --resource-group myResourceGroup --name myScaleSet --instance-id instanceId
    ```

- Vous pouvez également utiliser [resources.azure.com](https://resources.azure.com) ou les [kits de développement logiciel (SDK) Azure](https://azure.microsoft.com/downloads/).

La sortie exacte obtenue varie selon les options que vous ajoutez à la commande. L’exemple suivant représente une sortie condensée de l’interface Azure CLI :

```azurecli
$ az vmss show --resource-group myResourceGroup --name myScaleSet
{
  "location": "westus",
  "name": "{name}",
  "sku": {
    "name": "Standard_D2_v2",
    "tier": "Standard"
  },
}
```

Ces propriétés décrivent la configuration d’une instance de machine virtuelle au sein d’un groupe identique, et non celle du groupe identique dans son ensemble. Par exemple, le modèle du groupe identique a la propriété `overprovision`, contrairement au modèle d’une instance de machine virtuelle d’un groupe identique. En effet, le surprovisionnement est une propriété qui s’applique aux groupes identiques, mais pas aux instances de machine virtuelle qui les constituent (pour plus d’informations sur le surprovisionnement, consultez la section [Considérations relatives à la conception de groupes de machines virtuelles identiques](virtual-machine-scale-sets-design-overview.md#overprovisioning)).


### <a name="the-scale-set-vm-instance-view"></a>Vue d’instance des machines virtuelles d’un groupe identique
Comme le groupe identique, chaque instance de machine virtuelle a sa propre vue d’instance. Pour interroger la vue d’instance d’une instance de machine virtuelle d’un groupe identique, vous pouvez utiliser :

- Les API REST avec [compute/virtualmachinescalesetvms/getinstanceview](/rest/api/compute/virtualmachinescalesetvms/getinstanceview) comme suit :

    ```rest
    GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet/virtualmachines/instanceId/instanceView?api-version={apiVersion}
    ```

- Azure PowerShell avec [Get-AzVmssVm](/powershell/module/az.compute/get-azvmssvm) :

    ```powershell
    Get-AzVmssVm -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId instanceId -InstanceView
    ```

- Azure CLI avec [az vmss get-instance-view](/cli/azure/vmss) :

    ```azurecli
    az vmss get-instance-view --resource-group myResourceGroup --name myScaleSet --instance-id instanceId
    ```

- Vous pouvez également utiliser [resources.azure.com](https://resources.azure.com) ou les [kits de développement logiciel Azure](https://azure.microsoft.com/downloads/)

La sortie exacte obtenue varie selon les options que vous ajoutez à la commande. L’exemple suivant représente une sortie condensée de l’interface Azure CLI :

```azurecli
$ az vmss get-instance-view --resource-group myResourceGroup --name myScaleSet --instance-id instanceId
{
  "additionalProperties": {
    "osName": "ubuntu",
    "osVersion": "16.04"
  },
  "disks": [
    {
      "name": "{name}",
      "statuses": [
        {
          "additionalProperties": {},
          "code": "ProvisioningState/succeeded",
          "displayStatus": "Provisioning succeeded",
          "time": "{time}"
        }
      ]
    }
  ],
  "statuses": [
    {
      "additionalProperties": {},
      "code": "ProvisioningState/succeeded",
      "displayStatus": "Provisioning succeeded",
      "time": "{time}"
    },
    {
      "additionalProperties": {},
      "code": "PowerState/running",
      "displayStatus": "VM running"
    }
  ],
  "vmAgent": {
    "statuses": [
      {
        "additionalProperties": {},
        "code": "ProvisioningState/succeeded",
        "displayStatus": "Ready",
        "level": "Info",
        "message": "Guest Agent is running",
        "time": "{time}"
      }
    ],
    "vmAgentVersion": "{version}"
  },
}
```

Ces propriétés décrivent l’état actuel de l’exécution d’une instance de machine virtuelle au sein d’un groupe identique, qui inclut toute extension appliquée au groupe identique.


## <a name="how-to-update-global-scale-set-properties"></a>Comment mettre à jour les propriétés globales d’un groupe identique
Pour mettre à jour une propriété globale d’un groupe identique, vous devez mettre à jour cette propriété dans le modèle du groupe identique. Vous pouvez effectuer cette modification via :

- Les REST API avec [compute/virtualmachinescalesets/createorupdate](/rest/api/compute/virtualmachinescalesets/createorupdate) comme suit :

    ```rest
    PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet?api-version={apiVersion}
    ```

- Vous pouvez déployer un modèle Resource Manager avec les propriétés de l’API REST pour mettre à jour les propriétés globales du groupe identique.

- Azure PowerShell avec [Update-AzVmss](/powershell/module/az.compute/update-azvmss) :

    ```powershell
    Update-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -VirtualMachineScaleSet {scaleSetConfigPowershellObject}
    ```

- Azure CLI avec [az vmss update](/cli/azure/vmss) :
    - Pour modifier une propriété :

        ```azurecli
        az vmss update --set {propertyPath}={value}
        ```

    - Pour ajouter un objet à la propriété de liste d’un groupe identique : 

        ```azurecli
        az vmss update --add {propertyPath} {JSONObjectToAdd}
        ```

    - Pour supprimer un objet de la propriété de liste d’un groupe identique : 

        ```azurecli
        az vmss update --remove {propertyPath} {indexToRemove}
        ```

    - Si vous avez précédemment déployé le groupe identique à l’aide de la commande `az vmss create`, vous pouvez réexécuter la commande `az vmss create` pour modifier le groupe identique. Vérifiez que toutes les propriétés définies dans la commande `az vmss create` sont les mêmes qu’avant, à l’exception de celles que vous souhaitez modifier.

- Vous pouvez également utiliser [resources.azure.com](https://resources.azure.com) ou les [kits de développement logiciel (SDK) Azure](https://azure.microsoft.com/downloads/).

Une fois que vous avez modifié le modèle du groupe identique, la nouvelle configuration est appliquée à toutes les nouvelles machines virtuelles qui sont créées dans le groupe identique. Toutefois, les modèles des machines virtuelles existantes appartenant au groupe identique doivent toujours être mis à jour à l’aide du dernier modèle global du groupe identique. Le modèle de chaque machine virtuelle contient une propriété booléenne appelée `latestModelApplied`, qui indique si la machine virtuelle est à jour, par rapport au dernier modèle global du groupe identique (`true` signifie que la machine virtuelle est à jour par rapport au dernier modèle).


## <a name="how-to-bring-vms-up-to-date-with-the-latest-scale-set-model"></a>Comment mettre à jour les machines virtuelles avec le dernier modèle du groupe identique
Les groupes identiques ont une « stratégie de mise à niveau » qui détermine la façon dont les machines virtuelles sont mises à jour à l’aide du dernier modèle du groupe identique. Les trois modes de la stratégie de mise à niveau sont les suivants :

- **Automatique** : avec ce mode, le groupe identique ne garantit pas l’ordre dans lequel les machines virtuelles sont arrêtées. Le groupe identique peut arrêter toutes les machines virtuelles en même temps. 
- **Continue** : avec ce mode, le groupe identique déploie la mise à jour par lots, en marquant éventuellement une pause entre chaque lot.
- **Manuelle** : avec ce mode, quand vous mettez à jour le modèle du groupe identique, les machines virtuelles existantes ne sont pas mises à jour.
 
Pour mettre à jour les machines virtuelles existantes, vous devez effectuer une « mise à niveau manuelle » de chaque machine virtuelle existante. Vous pouvez effectuer cette mise à niveau manuelle avec :

- Les API REST avec [compute/virtualmachinescalesets/updateinstances](/rest/api/compute/virtualmachinescalesets/updateinstances) comme suit :

    ```rest
    POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet/manualupgrade?api-version={apiVersion}
    ```

- Azure PowerShell avec [Update-AzVmssInstance](/powershell/module/az.compute/update-azvmssinstance) :
    
    ```powershell
    Update-AzVmssInstance -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId instanceId
    ```

- Azure CLI avec [az vmss update-instances](/cli/azure/vmss) :

    ```azurecli
    az vmss update-instances --resource-group myResourceGroup --name myScaleSet --instance-ids {instanceIds}
    ```

- Vous pouvez également utiliser les [kits de développement logiciel (SDK) Azure](https://azure.microsoft.com/downloads/) propres à un langage.

>[!NOTE]
> Les clusters Service Fabric peuvent uniquement utiliser le mode *Automatique*, mais la mise à jour est gérée différemment. Pour plus d’informations, consultez [Mise à niveau des applications Service Fabric](../service-fabric/service-fabric-application-upgrade.md).

Il existe une méthode de modification des propriétés globales de groupe identique qui ne respecte pas la stratégie de mise à niveau. Les modifications apportées au profil de système d’exploitation et de disque de données du groupe identique (par exemple au nom d’utilisateur ou au mot de passe de l’administrateur) peuvent être apportées uniquement dans la version d’API *2017-12-01* ou une version ultérieure. Ces modifications s’appliquent uniquement aux machines virtuelles qui ont été créées après la modification du modèle du groupe identique. Pour mettre à jour les machines virtuelles existantes, vous devez effectuer un réimageage pour chacune d’elles. Vous pouvez effectuer ce réimageage via :

- Les API REST avec [compute/virtualmachinescalesets/reimage](/rest/api/compute/virtualmachinescalesets/reimage) comme suit :

    ```rest
    POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet/reimage?api-version={apiVersion}
    ```

- Azure PowerShell avec [Set-AzVmssVm](/powershell/module/az.compute/set-azvmssvm) :

    ```powershell
    Set-AzVmssVM -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId instanceId -Reimage
    ```

- Azure CLI avec [az vmss reimage](/cli/azure/vmss) :

    ```azurecli
    az vmss reimage --resource-group myResourceGroup --name myScaleSet --instance-id instanceId
    ```

- Vous pouvez également utiliser les [kits de développement logiciel (SDK) Azure](https://azure.microsoft.com/downloads/) propres à un langage.


## <a name="properties-with-restrictions-on-modification"></a>Propriétés avec restrictions au niveau de la modification

### <a name="create-time-properties"></a>Propriétés définies au moment de la création
Certaines propriétés peuvent être définies uniquement lors de la création du groupe identique. Ces propriétés sont les suivantes :

- Zones de disponibilité
- Image reference publisher
- Image reference offer
- Type géré de compte de stockage de disque de système d’exploitation

### <a name="properties-that-can-only-be-changed-based-on-the-current-value"></a>Propriétés qui peuvent être modifiées uniquement en fonction de la valeur actuelle
La capacité à modifier certaines propriétés peut dépendre de la valeur actuelle. Ces propriétés sont les suivantes :

- **singlePlacementGroup** : si singlePlacementGroup a la valeur true, celle-ci peut être remplacée par false. Toutefois, si singlePlacementGroup a la valeur false, elle **ne peut pas** avoir la valeur true.
- **subnet** : le sous-réseau d’un groupe identique peut être modifié, tant que le sous-réseau d’origine et le nouveau sous-réseau se trouvent dans le même réseau virtuel.
- **imageReferenceSku** : une référence (SKU) d’image peut être mise à jour pour des [distributions Linux](https://docs.microsoft.com/azure/virtual-machines/linux/endorsed-distros) approuvées, des images de serveur/client Windows et des images sans [informations de plan](https://docs.microsoft.com/azure/virtual-machines/linux/cli-ps-findimage#view-plan-properties). 

### <a name="properties-that-require-deallocation-to-change"></a>Propriétés qui nécessitent une désallocation pour être modifiées
La valeur de certaines propriétés peut uniquement être modifiée si les machines virtuelles du groupe identique sont libérées. Ces propriétés sont les suivantes :

- **SKU Name** : si la nouvelle référence (SKU) de machine virtuelle n’est pas prise en charge sur l’appareil sur lequel se trouve le groupe identique, vous devez libérer les machines virtuelles du groupe identique avant de modifier le nom de la référence (SKU). Pour plus d’informations, voir [comment redimensionner une machine virtuelle Azure](../virtual-machines/windows/resize-vm.md). 

## <a name="vm-specific-updates"></a>Mises à jour relatives à certaines machines virtuelles
Certaines modifications peuvent être appliquées uniquement à certaines machines virtuelles, et non aux propriétés globales du groupe identique. Pour le moment, la seule mise à jour applicable à certaines machines virtuelles qui est prise en charge est celle qui permet de joindre et de détacher des disques de données au niveau des machines virtuelles du groupe identique. Cette fonctionnalité est en préversion. Pour plus d’informations, consultez la [documentation relative à la préversion](https://github.com/Azure/vm-scale-sets/tree/master/preview/disk).


## <a name="scenarios"></a>Scénarios

### <a name="application-updates"></a>Mises à jour d’application
Si une application est déployée dans un groupe identique via des extensions, la mise à jour de la configuration de l’extension va entraîner la mise à jour de l’application conformément à la stratégie de mise à niveau. Par exemple, si vous avez une nouvelle version d’un script qui doit s’exécuter dans une extension de script personnalisé, vous pouvez mettre à jour la propriété *fileUris* pour qu’elle pointe vers le nouveau script. Dans certains cas, toutefois, il est nécessaire de forcer une mise à jour, même si cela ne modifie pas la configuration de l’extension (par exemple, si vous avez mis à jour le script sans modifier son URI). Dans ce cas, vous pouvez modifier le *forceUpdateTag* pour qu’il force la mise à jour. La plateforme Azure n’interprète pas cette propriété. Toute modification de cette valeur n’a aucun effet sur l’exécution de l’extension. Sa modification ne fait que forcer la réexécution de l’extension. Pour plus d’informations sur *forceUpdateTag*, consultez la [documentation API REST sur les extensions](/rest/api/compute/virtualmachineextensions/createorupdate). Notez que la balise *forceUpdateTag* peut être utilisée avec toutes les extensions et pas seulement avec celle de script personnalisé.

Il est également courant de déployer des applications à l’aide d’une image personnalisée. Ce scénario est décrit dans la section suivante.

### <a name="os-updates"></a>Mises à jour de système d’exploitation
Si vous utilisez des images de plateforme Azure, vous pouvez les mettre à jour en modifiant *imageReference* (pour plus d’informations, consultez la [documentation de l’API REST](/rest/api/compute/virtualmachinescalesets/createorupdate)).

>[!NOTE]
> Avec les images de plateforme, il est courant de spécifier « latest » (dernière) pour la version de référence d’image. Cela signifie que lors de la création, de l’augmentation de la taille des instances et de la réinitialisation du groupe identique, les machines virtuelles sont créées avec la dernière version disponible. Toutefois, cela **ne signifie pas** que l’image du système d’exploitation sera automatiquement mise à jour à chaque nouvelle version d’image. Une fonction distincte, actuellement en version préliminaire, fournit des mises à niveau automatiques du système d’exploitation. Pour plus d’informations, consultez la [documentation relative aux mises à niveau automatiques du système d’exploitation](virtual-machine-scale-sets-automatic-upgrade.md).

Si vous utilisez des images personnalisées, vous pouvez les mettre à jour en modifiant l’ID *imageReference* (pour plus d’informations, consultez la [documentation de l’API REST](/rest/api/compute/virtualmachinescalesets/createorupdate)).

## <a name="examples"></a>Exemples

### <a name="update-the-os-image-for-your-scale-set"></a>Mettre à jour l’image du système d’exploitation pour un groupe identique
Supposons que vous possédez un groupe identique qui exécute une ancienne version d’Ubuntu LTS 16.04. Vous souhaitez mettre à jour cette version avec une version plus récente d’Ubuntu LTS 16.04 (par exemple, la version *16.04.201801090*. La propriété de version de la référence d’image ne fait pas partie d’une liste. Vous pouvez donc la modifier directement avec l’une des commandes suivantes :

- Azure PowerShell avec [Update-AzVmss](/powershell/module/az.compute/update-azvmss) comme suit :

    ```powershell
    Update-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -ImageReferenceVersion 16.04.201801090
    ```

- Azure CLI avec [az vmss update](/cli/azure/vmss) :

    ```azurecli
    az vmss update --resource-group myResourceGroup --name myScaleSet --set virtualMachineProfile.storageProfile.imageReference.version=16.04.201801090
    ```

Vous pouvez également modifier l'image utilisée par votre groupe identique. Par exemple, vous pouvez mettre à jour ou modifier une image personnalisée utilisée par votre groupe identique. Vous pouvez modifier l'image utilisée par votre groupe identique en mettant à jour la propriété ID de référence de l'image. La propriété ID de référence de limage ne fait partie d'aucune liste. Vous pouvez donc la modifier directement à l'aide d'une des commandes suivantes :

- Azure PowerShell avec [Update-AzVmss](/powershell/module/az.compute/update-azvmss) comme suit :

    ```powershell
    Update-AzVmss `
        -ResourceGroupName "myResourceGroup" `
        -VMScaleSetName "myScaleSet" `
        -ImageReferenceId /subscriptions/{subscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/images/myNewImage
    ```

- Azure CLI avec [az vmss update](/cli/azure/vmss) :

    ```azurecli
    az vmss update \
        --resource-group myResourceGroup \
        --name myScaleSet \
        --set virtualMachineProfile.storageProfile.imageReference.id=/subscriptions/{subscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/images/myNewImage
    ```


### <a name="update-the-load-balancer-for-your-scale-set"></a>Mettre à jour l’équilibreur de charge pour un groupe identique
Supposons que vous utilisiez un groupe identique avec Azure Load Balancer, et que vous souhaitiez remplacer Azure Load Balancer par Azure Application Gateway. Les propriétés de l’équilibreur de charge et du service Application Gateway d’un groupe identique font partie d’une liste. Vous pouvez donc utiliser les commandes de suppression et d’ajout d’éléments de liste, au lieu de modifier directement les propriétés :

- Azure Powershell :

    ```powershell
    # Get the current model of the scale set and store it in a local PowerShell object named $vmss
    $vmss=Get-AzVmss -ResourceGroupName "myResourceGroup" -Name "myScaleSet"
    
    # Create a local PowerShell object for the new desired IP configuration, which includes the reference to the application gateway
    $ipconf = New-AzVmssIPConfig "myNic" -ApplicationGatewayBackendAddressPoolsId /subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Network/applicationGateways/{applicationGatewayName}/backendAddressPools/{applicationGatewayBackendAddressPoolName} -SubnetId $vmss.VirtualMachineProfile.NetworkProfile.NetworkInterfaceConfigurations[0].IpConfigurations[0].Subnet.Id –Name $vmss.VirtualMachineProfile.NetworkProfile.NetworkInterfaceConfigurations[0].IpConfigurations[0].Name
    
    # Replace the existing IP configuration in the local PowerShell object (which contains the references to the current Azure Load Balancer) with the new IP configuration
    $vmss.VirtualMachineProfile.NetworkProfile.NetworkInterfaceConfigurations[0].IpConfigurations[0] = $ipconf
    
    # Update the model of the scale set with the new configuration in the local PowerShell object
    Update-AzVmss -ResourceGroupName "myResourceGroup" -Name "myScaleSet" -virtualMachineScaleSet $vmss
    ```

- Azure CLI :

    ```azurecli
    # Remove the load balancer backend pool from the scale set model
    az vmss update --resource-group myResourceGroup --name myScaleSet --remove virtualMachineProfile.networkProfile.networkInterfaceConfigurations[0].ipConfigurations[0].loadBalancerBackendAddressPools 0
    
    # Remove the load balancer backend pool from the scale set model; only necessary if you have NAT pools configured on the scale set
    az vmss update --resource-group myResourceGroup --name myScaleSet --remove virtualMachineProfile.networkProfile.networkInterfaceConfigurations[0].ipConfigurations[0].loadBalancerInboundNatPools 0
    
    # Add the application gateway backend pool to the scale set model
    az vmss update --resource-group myResourceGroup --name myScaleSet --add virtualMachineProfile.networkProfile.networkInterfaceConfigurations[0].ipConfigurations[0].ApplicationGatewayBackendAddressPools '{"id": "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Network/applicationGateways/{applicationGatewayName}/backendAddressPools/{applicationGatewayBackendPoolName}"}'
    ```

>[!NOTE]
> Ces commandes supposent qu’il n’existe qu’une seule configuration IP et qu’un seul équilibreur de charge dans le groupe identique. S’il en existe plusieurs, vous devrez peut-être utiliser un index de liste différent de *0*.


## <a name="next-steps"></a>Étapes suivantes
Vous pouvez également effectuer des tâches courantes de gestion sur des groupes identiques avec l’interface [Azure CLI](virtual-machine-scale-sets-manage-cli.md) ou [Azure PowerShell](virtual-machine-scale-sets-manage-powershell.md).
