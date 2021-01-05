---
title: Créer un groupe identique Azure utilisant les zones de disponibilité
description: Découvrir comment créer des groupes identiques de machines virtuelles Azure qui utilisent les zones de disponibilité pour augmenter la redondance contre les pannes
author: mimckitt
ms.author: mimckitt
ms.topic: conceptual
ms.service: virtual-machine-scale-sets
ms.subservice: availability
ms.date: 08/08/2018
ms.reviewer: jushiman
ms.custom: mimckitt, devx-track-azurecli
ms.openlocfilehash: c5ddd5846be91e9fc99a251d6ad45ade8bde2937
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96016656"
---
# <a name="create-a-virtual-machine-scale-set-that-uses-availability-zones"></a>Créer un groupe identique de machines virtuelles qui utilise les zones de disponibilité

Pour protéger vos groupes de machines virtuelles identiques contre les défaillances au niveau du centre de données, vous pouvez créer un groupe identique à travers les zones de disponibilité. Les régions Azure qui prennent en charge les zones de disponibilité comportent au minimum trois zones distinctes, chacune avec leurs propres source d’alimentation, réseau et système de refroidissement. Pour plus d'informations, consultez [Vue d’ensemble des zones de disponibilité](../availability-zones/az-overview.md).

## <a name="availability-considerations"></a>Considérations relatives à la disponibilité

Lorsque vous déployez un groupe identique régional (non zonal) dans une ou plusieurs zones en tant que version d’API *2017-12-01*, vos options de disponibilité sont les suivantes :
- Diffusion maximale (platformFaultDomainCount = 1)
- Diffusion fixe statique (platformFaultDomainCount = 5)
- Diffusion alignée sur les domaines d’erreur de disque de stockage (platforFaultDomainCount = 2 ou 3)

Avec « max spreading » (répartition max), le groupe identique répartit vos machines virtuelles sur autant de domaines d’erreur possibles au sein de chaque zone. Cette répartition peut s’effectuer sur plus ou moins de cinq domaines d’erreur par zone. Avec une diffusion fixe statique, le groupe identique répartit vos machines virtuelles dans exactement cinq domaines d’erreur par zone. Si le groupe identique ne parvient pas à trouver cinq domaines d’erreur distincts par zone pour satisfaire la requête de répartition, la requête échoue.

**Nous vous recommandons de déployer avec « max spreading » (répartition max.) pour la plupart des charges de travail**, car cette approche fournit la meilleure répartition dans la plupart des cas. Si vous avez besoin que les réplica soient répartis sur des unités matérielles d’isolation distinctes, nous vous recommandons de les répartir sur plusieurs zones de disponibilité et d’utiliser « max spreading » (répartition max.) au sein de chaque zone.

> [!NOTE]
> Avec « max spreading » (répartition max.), un seul domaine d’erreur s’affiche dans la vue d’instance de machine virtuelle ainsi que dans les métadonnées de l’instance, quel que soit le nombre de domaines d’erreur sur lesquels les machines virtuelles sont réparties. La répartition dans chaque zone est implicite.

### <a name="placement-groups"></a>Groupes de placement

Lorsque vous déployez un groupe identique, vous avez également la possibilité de déployer avec un seul [groupe de placement](./virtual-machine-scale-sets-placement-groups.md) par zone de disponibilité, ou plusieurs groupes par zone. Pour les groupes identiques régionaux (non-zonaux), le choix consiste à avoir un groupe de placement unique dans la région ou plusieurs groupes dans la région. Pour la plupart des charges de travail, nous recommandons d’utiliser plusieurs groupes de placement, qui permettent une plus grande mise à l’échelle. Dans la version d’API *2017-12-01*, pour les groupes identiques de zone unique et inter-zones l’option par défaut est plusieurs groupes de placement, mais pour les groupes identiques régionaux (non-zonaux) l’option par défaut est un seul groupe de placement.

> [!NOTE]
> Si vous utilisez « max spreading » (répartition max), vous devez utiliser plusieurs groupes de placement.

### <a name="zone-balancing"></a>Équilibrage des zones

Enfin, pour les groupes identiques déployés sur plusieurs zones, vous avez également la possibilité de choisir « best effort zone balance» (meilleur équilibre des zones) ou « strict zone balance » (équilibre des zones strict). Un groupe identique est considéré comme « équilibré » si le nombre de machines virtuelles de chaque zone est égal à celui de toutes les autres zones du groupe identique, à une machine virtuelle près. Par exemple :

- Un groupe identique composé de 2 machines virtuelles dans la zone 1, de 3 machines virtuelles dans la zone 2 et de 3 machines virtuelles dans la zone 3 est considéré comme étant équilibré. Une seule de ces zones présente un nombre de machines virtuelles différent, et ce nombre n’est inférieur que de 1 à celui des autres zones. 
- Un groupe identique composé d’1 machine virtuelle dans la zone 1, de 3 machines virtuelles dans la zone 2 et de 3 machines virtuelles dans la zone 3 est considéré comme non équilibré. En effet, la zone 1 comporte 2 machines virtuelles de moins que les zones 2 et 3.

Il est possible que les machines virtuelles du groupe identique soient correctement créées, mais que le déploiement d’extensions sur ces machines virtuelles échoue. Ces machines virtuelles avec des erreurs d’extension sont toujours comptabilisées lorsque l’équilibre d’un groupe identique est évalué. Par exemple, un groupe identique composé de 3 machines virtuelles en zone 1, 3 machines virtuelles en zone 2 et 3 machines virtuelles en zone 3 est considéré comme étant équilibré même si toutes les extensions ont échoué en zone 1 et toutes les extensions ont réussi en zones 2 et 3.

Avec « best effort zone balance » (meilleur équilibre des zones), le groupe identique tente de diminuer et d’augmenter la taille des instances tout en maintenant l’équilibre. Toutefois, si pour une raison quelconque cela n’est pas possible (par exemple, si une zone tombe en panne, le groupe identique ne peut pas créer de nouvelle machine virtuelle dans cette zone), le groupe identique autorise un déséquilibre temporaire pour effectuer correctement un scale-in ou un scale-out. Lors des tentatives suivantes d’augmentation de la taille des instances, le groupe identique ajoute des machines virtuelles aux zones qui ont besoin de davantage de machines virtuelles pour équilibrer le groupe identique. De même, lors des tentatives suivantes de diminution de la taille des instances, le groupe identique retire des machines virtuelles aux zones qui ont besoin de moins de machines virtuelles pour équilibrer le groupe identique. Avec « strict zone balance » (équilibre des zones strict), toute tentative du groupe identique pour diminuer et augmenter la taille des instances échoue si cela crée un déséquilibre.

Pour utiliser « best effort zone balance » (meilleur équilibre des zones), définissez *zoneBalance* sur *false*. Il s’agit du paramètre par défaut dans la version d’API *2017-12-01*. Pour utiliser « strict zone balance » (équilibre des zones strict), définissez la valeur *zoneBalance* sur *true*.

## <a name="single-zone-and-zone-redundant-scale-sets"></a>Groupes identiques dans une zone unique et redondants interzone

Quand vous déployez un groupe de machines virtuelles identiques, vous pouvez utiliser une seule zone de disponibilité dans une région, ou bien plusieurs zones.

Quand vous créez un groupe identique dans une zone unique, vous contrôlez la zone dans laquelle toutes ces instances de machine virtuelle s’exécutent, et le groupe identique est géré et automatiquement mis à l’échelle dans cette zone uniquement. Un groupe identique redondant interzone permet de créer un groupe identique unique qui couvre plusieurs zones. Au fur et à mesure que les instances de machine virtuelle sont créées, elles sont, par défaut, uniformément réparties sur les différentes zones. En cas d’interruption dans l’une de ces zones, le groupe identique ne se met pas automatiquement à l’échelle pour augmenter la capacité. Une meilleure pratique serait de configurer des règles de mise à l’échelle automatique en fonction de l’utilisation du processeur ou de la mémoire. Les règles de mise à l’échelle automatique permettraient au groupe identique de réagir en cas de perte d’instances de machine virtuelle dans cette zone en augmentant la taille des instances dans les zones opérationnelles restantes.

Pour utiliser les zones de disponibilité, votre groupe identique doit être créé dans une [région Azure prise en charge](../availability-zones/az-region.md). Vous pouvez créer un groupe identique qui utilise des zones de disponibilité avec l’une des méthodes suivantes :

- [Azure portal](#use-the-azure-portal)
- Azure CLI
- [Azure PowerShell](#use-azure-powershell)
- [Modèles Microsoft Azure Resource Manager](#use-azure-resource-manager-templates)

## <a name="use-the-azure-portal"></a>Utilisation du portail Azure

Le processus de création d’un groupe identique qui utilise une zone de disponibilité est identique à celui décrit dans [l’article de prise en main](quick-create-portal.md). Lorsque vous sélectionnez une région Azure prise en charge, vous pouvez créer un groupe identique dans une zone disponible ou plus, comme indiqué dans l’exemple suivant :

![Créer un groupe identique dans une zone de disponibilité unique](media/virtual-machine-scale-sets-use-availability-zones/vmss-az-portal.png)

Le groupe identique et les ressources prises en charge, notamment l’équilibreur de charge Azure et l’adresse IP publique, sont créés dans la même zone que vous spécifiez.

## <a name="use-the-azure-cli"></a>Utilisation de l’interface de ligne de commande Microsoft Azure

Le processus de création d’un groupe identique qui utilise une zone de disponibilité est identique à celui décrit dans [l’article de prise en main](quick-create-cli.md). Pour utiliser les zones de disponibilité, vous devez créer votre groupe identique dans une région Azure prise en charge.

Ajoutez le paramètre `--zones` à la commande [az vmss create](/cli/azure/vmss), puis spécifiez la zone à utiliser (par exemple, zone *1*, *2* ou *3*). L’exemple suivant crée un groupe identique dans une zone unique, nommé *myScaleSet* dans la zone *1* :

```azurecli
az vmss create \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --image UbuntuLTS \
    --upgrade-policy-mode automatic \
    --admin-username azureuser \
    --generate-ssh-keys \
    --zones 1
```

Pour obtenir un exemple complet de groupe identique dans une zone unique et de ressources réseau, consultez [cet exemple de script CLI](https://github.com/Azure/azure-docs-cli-python-samples/blob/master/virtual-machine-scale-sets/create-single-availability-zone/create-single-availability-zone.sh).

### <a name="zone-redundant-scale-set"></a>Groupe identique redondant interzone

Pour créer un groupe identique redondant interzone, vous devez utiliser une adresse IP publique et un équilibreur de charge avec une référence SKU *Standard*. Pour assurer une meilleure redondance, la référence SKU *Standard* crée des ressources réseau redondantes interzone. Pour plus d’informations, consultez les articles [Présentation de Azure Load Balancer Standard](../load-balancer/load-balancer-overview.md) et [Référence Standard de Load Balancer et zones de disponibilité](../load-balancer/load-balancer-standard-availability-zones.md).

Pour créer un groupe identique redondant interzone, spécifiez des zones multiples à l’aide du paramètre `--zones`. Dans l’exemple suivant, un groupe identique redondant interzone, nommé *myScaleSet*, est créé dans les zones *1,2,3* :

```azurecli
az vmss create \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --image UbuntuLTS \
    --upgrade-policy-mode automatic \
    --admin-username azureuser \
    --generate-ssh-keys \
    --zones 1 2 3
```

Quelques minutes sont nécessaires pour créer et configurer l’ensemble des ressources et machines virtuelles du groupe identique dans les zones que vous spécifiez. Pour obtenir un exemple complet de groupe identique redondant interzone et de ressources réseau, consultez [cet exemple de script CLI](https://github.com/Azure/azure-docs-cli-python-samples/blob/master/virtual-machine-scale-sets/create-zone-redundant-scale-set/create-zone-redundant-scale-set.sh).

## <a name="use-azure-powershell"></a>Utilisation d'Azure PowerShell

Pour utiliser les zones de disponibilité, vous devez créer votre groupe identique dans une région Azure prise en charge. Ajoutez le paramètre `-Zone` à la commande [New-AzVmssConfig](/powershell/module/az.compute/new-azvmssconfig), puis spécifiez la zone à utiliser (par exemple, zone *1*, *2* ou *3*).

L’exemple suivant crée un groupe identique Linux dans une zone unique, nommé *myScaleSet* dans la région *USA Est 2*, zone *1*. Les ressources réseau Azure pour le réseau virtuel, l’adresse IP publique et l’équilibreur de charge sont automatiquement créées. Lorsque vous y êtes invité, fournissez vos propres informations d’identification d’administration souhaitées pour les instances de machine virtuelle dans le groupe identique :

```powershell
New-AzVmss `
  -ResourceGroupName "myResourceGroup" `
  -Location "EastUS2" `
  -VMScaleSetName "myScaleSet" `
  -VirtualNetworkName "myVnet" `
  -SubnetName "mySubnet" `
  -PublicIpAddressName "myPublicIPAddress" `
  -LoadBalancerName "myLoadBalancer" `
  -UpgradePolicy "Automatic" `
  -Zone "1"
```

### <a name="zone-redundant-scale-set"></a>Groupe identique redondant interzone

Pour créer un groupe identique redondant interzone, spécifiez des zones multiples à l’aide du paramètre `-Zone`. Dans l’exemple suivant, un groupe identique redondant interzone, nommé *myScaleSet*, est créé dans la région *USA Est 2*, zones *1, 2, 3*. Les ressources réseau Azure redondantes interzone pour le réseau virtuel, l’adresse IP publique et l’équilibreur de charge sont automatiquement créés. Lorsque vous y êtes invité, fournissez vos propres informations d’identification d’administration souhaitées pour les instances de machine virtuelle dans le groupe identique :

```powershell
New-AzVmss `
  -ResourceGroupName "myResourceGroup" `
  -Location "EastUS2" `
  -VMScaleSetName "myScaleSet" `
  -VirtualNetworkName "myVnet" `
  -SubnetName "mySubnet" `
  -PublicIpAddressName "myPublicIPAddress" `
  -LoadBalancerName "myLoadBalancer" `
  -UpgradePolicy "Automatic" `
  -Zone "1", "2", "3"
```

## <a name="use-azure-resource-manager-templates"></a>Utiliser les modèles Azure Resource Manager

Le processus de création d’un groupe identique qui utilise une zone de disponibilité est identique à celui décrit dans l’article de prise en main pour [Linux](quick-create-template-linux.md) ou [Windows](quick-create-template-windows.md). Pour utiliser les zones de disponibilité, vous devez créer votre groupe identique dans une région Azure prise en charge. Ajoutez la propriété `zones` au type de ressource *Microsoft.Compute/virtualMachineScaleSets* dans votre modèle, puis spécifiez la zone à utiliser (par exemple, zone *1*, *2* ou *3*).

L’exemple suivant crée un groupe identique Linux dans une zone unique, nommé *myScaleSet* dans la région *USA Est 2*, zone *1* :

```json
{
  "type": "Microsoft.Compute/virtualMachineScaleSets",
  "name": "myScaleSet",
  "location": "East US 2",
  "apiVersion": "2017-12-01",
  "zones": ["1"],
  "sku": {
    "name": "Standard_A1",
    "capacity": "2"
  },
  "properties": {
    "upgradePolicy": {
      "mode": "Automatic"
    },
    "virtualMachineProfile": {
      "storageProfile": {
        "osDisk": {
          "caching": "ReadWrite",
          "createOption": "FromImage"
        },
        "imageReference":  {
          "publisher": "Canonical",
          "offer": "UbuntuServer",
          "sku": "16.04-LTS",
          "version": "latest"
        }
      },
      "osProfile": {
        "computerNamePrefix": "myvmss",
        "adminUsername": "azureuser",
        "adminPassword": "P@ssw0rd!"
      }
    }
  }
}
```

Pour obtenir un exemple complet de groupe identique dans une zone unique et de ressources réseau, consultez [cet exemple de modèle Resource Manager](https://github.com/Azure/vm-scale-sets/blob/master/preview/zones/singlezone.json).

### <a name="zone-redundant-scale-set"></a>Groupe identique redondant interzone

Pour créer un groupe identique redondant interzone, spécifiez plusieurs valeurs pour la propriété `zones` du type de ressource *Microsoft.Compute/virtualMachineScaleSets*. Dans l’exemple suivant, un groupe identique redondant interzone, nommé *myScaleSet*, est créé dans la région *USA Est 2*, zones *1,2,3* :

```json
{
  "type": "Microsoft.Compute/virtualMachineScaleSets",
  "name": "myScaleSet",
  "location": "East US 2",
  "apiVersion": "2017-12-01",
  "zones": [
        "1",
        "2",
        "3"
      ]
}
```

Si vous créez une adresse IP publique ou un équilibreur de charge, spécifiez la propriété *"sku": { "name": "Standard" }"* pour créer des ressources réseau redondantes interzone. Vous devez également créer un groupe de sécurité réseau et les règles associées pour autoriser tout le trafic. Pour plus d’informations, consultez les articles [Présentation de Azure Load Balancer Standard](../load-balancer/load-balancer-overview.md) et [Référence Standard de Load Balancer et zones de disponibilité](../load-balancer/load-balancer-standard-availability-zones.md).

Pour obtenir un exemple complet de groupe identique redondant interzone et de ressources réseau, consultez [cet exemple de modèle Resource Manager](https://github.com/Azure/vm-scale-sets/blob/master/preview/zones/multizone.json).

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez créé un groupe identique dans une zone de disponibilité, vous pouvez apprendre à [déployer des applications sur des groupes identiques de machines virtuelles](tutorial-install-apps-cli.md) ou à [utiliser la mise à l’échelle automatique avec des groupes identiques de machines virtuelles](tutorial-autoscale-cli.md).
