---
title: Désactiver et réactiver le module complémentaire Application Gateway Ingress Controller pour le cluster Azure Kubernetes Service
description: Cet article fournit des informations sur la façon de désactiver et de réactiver le module complémentaire AGIC pour votre cluster AKS
services: application-gateway
author: caya
ms.service: application-gateway
ms.topic: how-to
ms.date: 06/10/2020
ms.author: caya
ms.openlocfilehash: fe4da0435731c536a723cb2cb43428166456360b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84807957"
---
# <a name="disable-and-re-enable-agic-add-on-for-your-aks-cluster"></a>Désactiver et réactiver le module complémentaire AGIC pour votre cluster AKS
AGIC (Application Gateway Ingress Controller) déployé en tant que module complémentaire AKS vous permet d’activer et de désactiver le module complémentaire avec une seule ligne dans Azure CLI. Le cycle de vie de l’instance Application Gateway varie quand vous désactivez le module complémentaire AGIC, selon qu’elle a été créée par le module complémentaire AGIC ou qu’elle a été déployée séparément du module complémentaire AGIC. Vous pouvez exécuter la même commande pour réactiver le module complémentaire AGIC si vous le désactivez, ou pour activer le module complémentaire AGIC à l’aide d’un cluster AKS et une instance Application Gateway existants.

## <a name="disabling-agic-add-on-with-associated-application-gateway"></a>Désactivation du module complémentaire AGIC avec l’instance Application Gateway associée 
Si le module complémentaire AGIC a automatiquement déployé l’instance Application Gateway la première fois que vous avez effectué la configuration de tous les éléments, la désactivation du module complémentaire AGIC supprime par défaut l’instance Application Gateway en fonction de certains critères. Il y a deux critères que le module complémentaire AGIC recherche pour déterminer s’il doit supprimer l’instance Application Gateway associée quand vous le désactivez :
- L’instance Application Gateway à laquelle le module complémentaire AGIC est associé est-il déployé dans le groupe de ressources du nœud MC_* ? 
- L’instance Application Gateway à laquelle le module complémentaire AGIC est associé a-t-elle l’étiquette « created-by: ingress-appgw » ? L’étiquette est utilisée par AGIC pour déterminer si l’instance Application Gateway a été, ou non, déployée par le module complémentaire. 

Si les deux critères sont satisfaits, le module complémentaire AGIC supprime l’instance Application Gateway qu’il a créée quand il est désactivé. Toutefois, il ne supprime pas l’adresse IP publique ou le sous-réseau avec lequel ou dans lequel l’instance Application Gateway a été déployée. Si le premier critère n’est pas satisfait, peu importe que l’instance Application Gateway ait l’étiquette « created-by: ingress-appgw » : la désactivation du module complémentaire ne supprime pas l’instance Application Gateway. De même, si le deuxième critère n’est pas satisfait, c’est-à-dire si l’instance Application Gateway n’a pas cette étiquette, la désactivation du module complémentaire ne supprime pas l’instance Application Gateway dans le groupe de ressources du nœud MC_*. 

> [!TIP] 
> Si vous ne voulez pas que l’instance Application Gateway soit supprimée lors de la désactivation du module complémentaire, mais qu’elle satisfait les deux critères, supprimez l’étiquette « created-by: ingress-appgw » pour empêcher le module complémentaire de supprimer votre instance Application Gateway. 

Pour désactiver le module complémentaire AGIC, exécutez la commande suivante : 
```azurecli-interactive
az aks disable-addons -n <AKS-cluster-name> -g <AKS-resource-group-name> -a ingress-appgw 
```

## <a name="enable-agic-add-on-on-existing-application-gateway-and-aks-cluster"></a>Activez le module complémentaire AGIC sur une instance Application Gateway et un cluster AKS existants
Si vous désactivez le module complémentaire AGIC et que vous devez le réactiver, ou que vous voulez l’activer à l’aide d’une instance Application Gateway et d’un cluster AKS existants, exécutez la commande suivante :

```azurecli-interactive
appgwId=$(az network application-gateway show -n <application-gateway-name> -g <resource-group-name> -o tsv --query "id") 
az aks enable-addons -n <AKS-cluster-name> -g <AKS-cluster-resource-group> -a ingress-appgw --appgw-id $appgwId
```

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur l’activation du module complémentaire AGIC à l’aide d’une instance Application Gateway et d’un cluster AKS existants, consultez [Déploiement « brownfield » en tant que module complémentaire AGIC](tutorial-ingress-controller-add-on-existing.md).