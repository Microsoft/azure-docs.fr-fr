---
title: Remplacement de sous-réseau Azure Traffic Manager avec Azure CLI | Microsoft Docs
description: Cet article va vous aider à comprendre comment le remplacement de sous-réseau Traffic Manager permet de remplacer la méthode de routage d’un profil Traffic Manager pour diriger le trafic vers un point de terminaison en fonction de l’adresse IP de l’utilisateur final via une plage d’adresses IP prédéfinie pour les mappages de point de terminaison.
services: traffic-manager
documentationcenter: ''
author: duongau
ms.topic: how-to
ms.service: traffic-manager
ms.date: 09/18/2019
ms.author: duau
ms.openlocfilehash: 984cdfb3e60717f7632ea1fdcd05a4681dd6a1fd
ms.sourcegitcommit: 5831eebdecaa68c3e006069b3a00f724bea0875a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94517935"
---
# <a name="traffic-manager-subnet-override-using-azure-cli"></a>Remplacement de sous-réseau Traffic Manager avec Azure CLI

Le remplacement du sous-réseau Traffic Manager vous permet de modifier la méthode de routage d’un profil.  L’ajout d’un remplacement permet de diriger le trafic en fonction de l’adresse IP de l’utilisateur final avec une plage d’adresses IP prédéfinie pour le mappage de point de terminaison. 

## <a name="how-subnet-override-works"></a>Comment fonctionne le remplacement de sous-réseau

Quand des remplacements de sous-réseau sont ajoutés à un profil Traffic Manager, Traffic Manager vérifie d’abord s’il existe un remplacement de sous-réseau pour l’adresse IP de l’utilisateur final. Si tel est le cas, la requête DNS de l’utilisateur est dirigée vers le point de terminaison correspondant.  En l’absence de mappage, Traffic Manager revient à la méthode de routage d’origine du profil. 

Les plages d’adresses IP peuvent être spécifiées sous la forme de plages CIDR (par exemple 1.2.3.0/24) ou de plages d’adresses (par exemple 1.2.3.4-5.6.7.8). Les plages d’adresses IP associées à chaque point de terminaison doivent être uniques pour ce point de terminaison. Tout chevauchement de plages d’adresses IP entre différents points de terminaison entraîne le rejet du profil par Traffic Manager.

Il existe deux types de profil de routage qui prennent en charge les remplacements de sous-réseau :

* **Géographique** - Si Traffic Manager trouve un remplacement de sous-réseau pour l’adresse IP de la requête DNS, il route cette requête vers le point de terminaison, quel que soit son intégrité.
* **Performances** - Si Traffic Manager trouve un remplacement de sous-réseau pour l’adresse IP de la requête DNS, il route le trafic vers le point de terminaison, uniquement s’il est sain.  Traffic Manager revient à l’heuristique de routage des performances si le point de terminaison de remplacement de sous-réseau n’est pas sain.

## <a name="create-a-traffic-manager-subnet-override"></a>Créer un remplacement de sous-réseau Traffic Manager

Pour créer un remplacement de sous-réseau Traffic Manager, vous pouvez utiliser Azure CLI afin d’ajouter les sous-réseaux du remplacement au point de terminaison Traffic Manager.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Cet article nécessite la version 2.0.28 ou ultérieure d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="update-the-traffic-manager-endpoint-with-subnet-override"></a>Mettez à jour le point de terminaison Traffic Manager avec le remplacement de sous-réseau.
Utilisez Azure CLI pour mettre à jour votre point de terminaison avec [az network traffic-manager endpoint update](https://docs.microsoft.com/cli/azure/network/traffic-manager/endpoint?view=azure-cli-latest#az-network-traffic-manager-endpoint-update).

```azurecli-interactive
### Add a range of IPs ###
az network traffic-manager endpoint update \
    --name MyEndpoint \
    --profile-name MyTmProfile \
    --resource-group MyResourceGroup \
    --subnets 1.2.3.4-5.6.7.8 \
    --type AzureEndpoints

### Add a subnet ###
az network traffic-manager endpoint update \
    --name MyEndpoint \
    --profile-name MyTmProfile \
    --resource-group MyResourceGroup \
    --subnets 9.10.11.0:24 \
    --type AzureEndpoints
```

Vous pouvez supprimer les plages d’adresses IP en exécutant [az network traffic-manager endpoint update](https://docs.microsoft.com/cli/azure/network/traffic-manager/endpoint?view=azure-cli-latest#az-network-traffic-manager-endpoint-update) avec l’option **--remove**.

```azurecli-interactive
az network traffic-manager endpoint update \
    --name MyEndpoint \
    --profile-name MyTmProfile \
    --resource-group MyResourceGroup \
    --remove subnets \
    --type AzureEndpoints
```

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les [méthodes de routage du trafic](traffic-manager-routing-methods.md)de Traffic Manager.

En savoir plus sur la [méthode de routage du trafic de sous-réseau](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-routing-methods#subnet-traffic-routing-method)
