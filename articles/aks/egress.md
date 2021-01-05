---
title: Utiliser une adresse IP statique pour le trafic de sortie
titleSuffix: Azure Kubernetes Service
description: Découvrez comment créer et utiliser une adresse IP publique statique pour le trafic de sortie dans un cluster Azure Kubernetes Service (AKS)
services: container-service
ms.topic: article
ms.date: 03/04/2019
ms.openlocfilehash: 81b99478358ec3d670e8d783fba27603483614ea
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87563243"
---
# <a name="use-a-static-public-ip-address-for-egress-traffic-with-a-basic-sku-load-balancer-in-azure-kubernetes-service-aks"></a>Utiliser une IP publique statique pour le trafic de sortie avec un équilibreur de charge de niveau tarifaire *De base* dans Azure Kubernetes Service (AKS)

Par défaut, l’adresse IP de sortie d’un cluster Azure Kubernetes Service (AKS) est affectée aléatoirement. Cette configuration n’est pas idéale quand vous avez besoin d’identifier une adresse IP par exemple pour accéder à des services externes. Au lieu de cela, il peut être nécessaire d’attribuer une adresse IP statique à ajouter à une liste d’adresses autorisées pour l’accès au service.

Cet article vous montre comment créer et utiliser une adresse IP publique statique pour le trafic de sortie dans un cluster AKS.

## <a name="before-you-begin"></a>Avant de commencer

Cet article suppose que vous utilisez l’équilibreur de charge Azure De base.  Nous vous recommandons d’utiliser l’[équilibreur de charge Azure Standard](../load-balancer/load-balancer-overview.md), et vous pouvez utiliser des fonctionnalités plus avancées pour [contrôler le trafic de sortie d’AKS](./limit-egress-traffic.md).

Cet article suppose que vous avez un cluster AKS existant. Si vous avez besoin d’un cluster AKS, consultez le guide de démarrage rapide d’AKS [avec Azure CLI][aks-quickstart-cli]ou avec le [Portail Azure][aks-quickstart-portal].

Azure CLI 2.0.59 (ou une version ultérieure) doit également être installé et configuré. Exécutez  `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, consultez  [Installation d’Azure CLI][install-azure-cli].

> [!IMPORTANT]
> Cet article utilise l’équilibreur de charge des références SKU *De base* dans un seul pool de nœuds. Cette configuration n’est pas disponible pour les pools de nœuds multiples, car l’équilibreur de charge des références SKU *De base* n’est pas pris en charge dans de tels scénarios. Consultez [Utiliser un équilibreur de charge Standard public dans Azure Kubernetes Service (AKS)][slb] pour plus d’informations sur l’utilisation de l’équilibreur de charge des références SKU *Standard*.

## <a name="egress-traffic-overview"></a>Vue d’ensemble du trafic de sortie

Le trafic sortant provenant d’un cluster AKS suit les [conventions d’Azure Load Balancer][outbound-connections]. Avant la création du premier service Kubernetes de type `LoadBalancer`, les nœuds d’agent d’un cluster AKS ne font partie d’aucun pool d’Azure Load Balancer. Dans cette configuration, les nœuds n’ont pas d’adresse IP publique au niveau de l’instance. Azure translate le flux sortant vers une adresse IP publique source qui n’est pas configurable ou déterministe.

Après la création d’un service Kubernetes de type `LoadBalancer`, les nœuds d’agent sont ajoutés à un pool d’équilibrage de charge Azure. Azure translate le flux sortant vers la première adresse IP publique configurée sur l’équilibreur de charge. Cette adresse IP publique est valide seulement pour la durée de vie de cette ressource. Si vous supprimez le service Load Balancer Kubernetes, l’adresse IP et l’équilibreur de charge associés sont également supprimés. Si vous voulez affecter une adresse IP spécifique ou conserver une adresse IP pour des services Kubernetes redéployés, vous pouvez créer et utiliser une adresse IP publique statique.

## <a name="create-a-static-public-ip"></a>Créer une adresse IP publique statique

Obtenez le nom du groupe de ressources avec la commande [az aks show][az-aks-show] et ajoutez le paramètre de requête `--query nodeResourceGroup`. L’exemple suivant obtient les le groupe de ressources du nœud pour le cluster AKS nommé *myAKSCluster* dans le groupe de ressources nommé *myResourceGroup* :

```azurecli-interactive
$ az aks show --resource-group myResourceGroup --name myAKSCluster --query nodeResourceGroup -o tsv

MC_myResourceGroup_myAKSCluster_eastus
```

Créez maintenant une adresse IP publique statique avec la commande [az network public ip create][az-network-public-ip-create]. Spécifiez le nom de groupe de ressources du nœud obtenue dans la commande précédente, puis un nom pour la ressource d’adresse IP, par exemple *myAKSPublicIP* :

```azurecli-interactive
az network public-ip create \
    --resource-group MC_myResourceGroup_myAKSCluster_eastus \
    --name myAKSPublicIP \
    --allocation-method static
```

L’adresse IP apparaît, comme illustré dans l’exemple de sortie condensée suivante :

```json
{
  "publicIp": {
    "dnsSettings": null,
    "etag": "W/\"6b6fb15c-5281-4f64-b332-8f68f46e1358\"",
    "id": "/subscriptions/<SubscriptionID>/resourceGroups/MC_myResourceGroup_myAKSCluster_eastus/providers/Microsoft.Network/publicIPAddresses/myAKSPublicIP",
    "idleTimeoutInMinutes": 4,
    "ipAddress": "40.121.183.52",
    [..]
  }
```

Vous pouvez obtenir ultérieurement l’adresse IP publique avec la commande [az network public-ip list][az-network-public-ip-list]. Spécifiez le nom du groupe de ressources du nœud, puis recherchez *ipAddress* comme indiqué dans l’exemple suivant :

```azurecli-interactive
$ az network public-ip list --resource-group MC_myResourceGroup_myAKSCluster_eastus --query [0].ipAddress --output tsv

40.121.183.52
```

## <a name="create-a-service-with-the-static-ip"></a>Créer un service avec l’adresse IP statique

Pour créer un service avec l’adresse IP publique statique, ajoutez la propriété `loadBalancerIP` et la valeur de l’adresse IP publique statique au manifeste YAML. Créez un fichier nommé `egress-service.yaml` et copiez-y le YAML suivant. Spécifiez votre propre adresse IP publique créée à l’étape précédente.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: azure-egress
spec:
  loadBalancerIP: 40.121.183.52
  type: LoadBalancer
  ports:
  - port: 80
```

Créez le service et le déploiement avec la commande `kubectl apply`.

```console
kubectl apply -f egress-service.yaml
```

Ce service configure une nouvelle adresse IP frontend sur l’équilibreur de charge Azure. Si vous n’avez pas d’autres adresses IP configurées, **tout** le trafic sortant doit désormais utiliser cette adresse. Si plusieurs adresses sont configurées sur Azure Load Balancer, toutes ces adresses sont candidates pour les flux sortants, mais une seule d’entre elles est sélectionnée au hasard.

## <a name="verify-egress-address"></a>Vérifier l’adresse de sortie

Pour vérifier que l’adresse IP publique statique est utilisée, vous pouvez utiliser un service de recherche DNS comme `checkip.dyndns.org`.

Démarrez et attachez à un pod *Debian* de base :

```console
kubectl run -it --rm aks-ip --image=debian
```

Pour accéder à un site web à partir du conteneur, utilisez `apt-get` pour installer `curl` dans le conteneur.

```console
apt-get update && apt-get install curl -y
```

Utilisez maintenant curl pour accéder au site *checkip.dyndns.org*. L’adresse IP de sortie apparaît, comme illustré dans l’exemple de sortie suivant. Cette adresse IP correspond à l’adresse IP publique statique créée et définie pour le service d’équilibreur de charge :

```console
$ curl -s checkip.dyndns.org

<html><head><title>Current IP Check</title></head><body>Current IP Address: 40.121.183.52</body></html>
```

## <a name="next-steps"></a>Étapes suivantes

Pour éviter de devoir gérer plusieurs adresses IP publiques sur l’équilibreur de charge Azure, vous pouvez à la place utiliser un contrôleur d’entrée. Les contrôleurs d’entrée offrent des avantages supplémentaires, comme l’arrêt de SSL/TLS, la prise en charge des réécritures d’URI et le chiffrement SSL/TLS en amont. Pour plus d’informations, consultez [Créer un contrôleur d’entrée de base dans AKS][ingress-aks-cluster].

<!-- LINKS - internal -->
[az-network-public-ip-create]: /cli/azure/network/public-ip#az-network-public-ip-create
[az-network-public-ip-list]: /cli/azure/network/public-ip#az-network-public-ip-list
[az-aks-show]: /cli/azure/aks#az-aks-show
[azure-cli-install]: /cli/azure/install-azure-cli
[ingress-aks-cluster]: ./ingress-basic.md
[outbound-connections]: ../load-balancer/load-balancer-outbound-connections.md#scenarios
[public-ip-create]: /cli/azure/network/public-ip#az-network-public-ip-create
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
[slb]: load-balancer-standard.md
