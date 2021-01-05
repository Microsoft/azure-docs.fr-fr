---
title: Mettre à l’échelle une instance d’Azure SignalR Service
description: Découvrez comment mettre à l’échelle une instance d’Azure SignalR Service pour augmenter ou réduire la capacité, par le biais du portail Azure ou d’Azure CLI.
author: sffamily
ms.service: signalr
ms.topic: conceptual
ms.date: 9/9/2020
ms.author: zhshang
ms.custom: devx-track-azurecli
ms.openlocfilehash: bb01f2f96e5db4c94e759b114818360e6084255f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89595765"
---
# <a name="how-to-scale-an-azure-signalr-service-instance"></a>Comment mettre à l’échelle une instance d’Azure SignalR Service ?
Cet article explique comment mettre à l’échelle votre instance d’Azure SignalR Service. Il existe deux scénarios de mise à l’échelle : le scale-up et le scale-out.

* [Monter en puissance](https://en.wikipedia.org/wiki/Scalability#Horizontal_and_vertical_scaling) : obtenir plus d’unités, de connexions, de messages, etc. Vous pouvez effectuer un scale-up en faisant passer le niveau tarifaire de Gratuit à Standard.
* [Effectuer un scale-out](https://en.wikipedia.org/wiki/Scalability#Horizontal_and_vertical_scaling) : augmenter le nombre d’unités SignalR. Vous pouvez effectuer un scale-out allant jusqu’à 100 unités. Des options d’unités limitées doivent être sélectionnées pour la mise à l’échelle : 1, 2, 5, 10, 20, 50 et 100 unités pour une instance SignalR Service unique.

L’application des paramètres de mise à l’échelle prend quelques minutes. Dans de rares cas, cette opération peut durer environ 30 minutes. Ils ne nécessitent pas de changer votre code ou de redéployer votre application serveur.

Pour plus d’informations sur les prix et les fonctionnalités de chaque instance de SignalR Service, consultez [Tarification du service Azure SignalR](https://azure.microsoft.com/pricing/details/signalr-service/).  

> [!NOTE]
> Quand vous faites passer SignalR Service du niveau **Gratuit** au niveau **Standard**, ou vice versa, l’adresse IP publique du service est changée et la propagation de la modification aux serveurs DNS sur l’ensemble d’Internet prend généralement 30 à 60 minutes. Il est possible que votre service soit inaccessible tant que DNS n’est pas mis à jour. En règle générale, il n’est pas recommandé de changer votre niveau tarifaire trop souvent.


## <a name="scale-on-azure-portal"></a>Effectuer une mise à l’échelle sur le portail Azure

1. Dans votre navigateur, ouvrez le [portail Azure](https://portal.azure.com).

2. Dans la page de votre instance de SignalR Service, dans le menu de gauche, sélectionnez **Mettre à l’échelle**.
   
3. Choisissez votre niveau tarifaire, puis cliquez sur **Sélectionner**. Définissez le nombre d’unités pour le niveau **Standard**.
   
    ![Effectuer une mise à l’échelle sur le portail](./media/signalr-howto-scale/signalr-howto-scale.png)

4. Cliquez sur **Enregistrer**.

## <a name="scale-using-azure-cli"></a>Mise à l’échelle à l’aide de l’interface de ligne de commande Azure

Ce script crée une ressource SignalR Service de niveau **Gratuit** et un groupe de ressources, puis le fait passer au niveau **Standard**. 

```azurecli-interactive
#!/bin/bash

# Generate a unique suffix for the service name
let randomNum=$RANDOM*$RANDOM

# Generate a unique service and group name with the suffix
SignalRName=SignalRTestSvc$randomNum
#resource name must be lowercase
mySignalRSvcName=${SignalRName,,}
myResourceGroupName=$SignalRName"Group"

# Create resource group 
az group create --name $myResourceGroupName --location eastus

# Create the Azure SignalR Service resource
az signalr create \
  --name $mySignalRSvcName \
  --resource-group $myResourceGroupName \
  --sku Free_F1 \
  --service-mode Default

# Scale up to Standard Tier, and scale out to 50 units
az signalr update \
  --name $mySignalRSvcName \
  --resource-group $myResourceGroupName \
  --sku Standard_S1 \
  --unit-count 50
```

Consignez le nom généré pour le nouveau groupe de ressources. Vous utiliserez ce nom de groupe de ressources au moment de la suppression de l’ensemble des ressources du groupe.

[!INCLUDE [cli-script-clean-up](../../includes/cli-script-clean-up.md)]

## <a name="compare-pricing-tiers"></a>Comparer les niveaux tarifaires

Pour obtenir des informations détaillées, telles que les messages et les connexions inclus pour chaque niveau tarifaire, consultez [Tarification du service SignalR](https://azure.microsoft.com/pricing/details/signalr-service/).

Pour voir un tableau des limites, quotas et contraintes du service dans chaque niveau, consultez [Limites de SignalR Service](../azure-resource-manager/management/azure-subscription-service-limits.md#azure-signalr-service-limits).

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide, vous avez appris à mettre à l’échelle une instance de SignalR Service unique.

Plusieurs points de terminaison sont également pris en charge pour les scénarios de mise à l’échelle, de partitionnement et inter-régions.

> [!div class="nextstepaction"]
> [Mettre à l’échelle SignalR Service avec plusieurs instances](./signalr-howto-scale-multi-instances.md)
