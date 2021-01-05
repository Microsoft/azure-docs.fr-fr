---
title: Plages d’adresses IP autorisées par le serveur d’API dans Azure Kubernetes Service (AKS)
description: Apprenez à sécuriser votre cluster à l’aide d’une plage d’adresses IP pour l’accès au serveur d’API dans Azure Kubernetes Service (AKS)
services: container-service
ms.topic: article
ms.date: 09/21/2020
ms.openlocfilehash: 9828682fa71d023356b174d528c2137ed29f368d
ms.sourcegitcommit: c157b830430f9937a7fa7a3a6666dcb66caa338b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94682500"
---
# <a name="secure-access-to-the-api-server-using-authorized-ip-address-ranges-in-azure-kubernetes-service-aks"></a>Sécuriser l’accès au serveur d’API à l’aide de plages d’adresses IP autorisées dans Azure Kubernetes Service (AKS)

Dans Kubernetes, le serveur d’API reçoit des requêtes pour effectuer des actions dans le cluster, comme la création de ressources ou la mise à l’échelle du nombre de nœuds. Le serveur d’API est le moyen centralisé pour interagir avec et gérer un cluster. Pour améliorer la sécurité du cluster et minimiser les attaques, le serveur d’API doit uniquement être accessible à partir d’un ensemble limité de plages d’adresses IP.

Cet article vous montre comment utiliser des plages d’adresses IP autorisées pour le serveur d’API pour définir quelles adresses IP et CIDR peuvent accéder plan de contrôle.

## <a name="before-you-begin"></a>Avant de commencer

Cet article vous montre comment créer un cluster AKS à l’aide d’Azure CLI.

La version 2.0.76 d’Azure CLI (ou ultérieure) doit être installée et configurée. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI][install-azure-cli].

### <a name="limitations"></a>Limites

La fonctionnalité des plages d’adresses IP autorisées par le serveur d’API présente les limitations suivantes :
- Sur les clusters créés après que les plages d’adresses IP autorisées par le serveur d’API ont été supprimées de la préversion en octobre 2019, les plages d’adresses IP autorisées par le serveur d’API sont uniquement prises en charge sur l’équilibreur de charge du SKU *Standard*. Les clusters existants avec l’équilibreur de charge SKU *De base* et les plages d’adresses IP autorisées du serveur d’API sont configurés pour continuer à fonctionner normalement, mais ils ne peuvent pas être migrés vers un équilibreur de charge SKU *Standard*. Ces clusters existants continueront de fonctionner si leur version de Kubernetes ou leur plan de contrôle sont mis à niveau. Les plages d’adresses IP autorisées par le serveur d’API ne sont pas prises en charge pour les clusters privés.
- Cette fonctionnalité n’est pas compatible avec les clusters qui utilisent la [fonctionnalité d’évaluation des pools de nœuds à une IP publique par nœud](use-multiple-node-pools.md#assign-a-public-ip-per-node-for-your-node-pools-preview).

## <a name="overview-of-api-server-authorized-ip-ranges"></a>Vue d’ensemble des plages d’adresses IP pour le serveur d’API

Le serveur d’API Kubernetes détermine la façon dont les API Kubernetes sous-jacentes sont exposées. Ce composant fournit l’interaction des outils de gestion, tels que `kubectl` ou le tableau de bord Kubernetes. AKS fournit un plan de contrôle de cluster monolocataire doté d’un serveur d’API dédié. Par défaut, le serveur d’API reçoit une IP publique et vous devez contrôler l’accès à l’aide du contrôle d’accès en fonction du rôle Kubernetes (Kubernetes RBAC) ou d’Azure RBAC.

Pour sécuriser l’accès au plan de contrôle AKS/serveur d’API, normalement accessible, vous pouvez activer et utiliser des plages d’adresses IP autorisées. Ces plages d’adresses IP autorisées autorisent uniquement les plages d’adresses IP définies à communiquer avec le serveur d’API. Une requête adressée au serveur API depuis une adresse IP qui ne fait pas partie de ces plages d’adresses IP autorisées est bloquée. Continuez à utiliser Kubernetes RBAC ou Azure RBAC pour autoriser des utilisateurs et les actions qu’ils demandent.

Pour plus d’informations sur le serveur d’API et les autres composants de cluster, consultez [Concepts de base de Kubernetes pour AKS][concepts-clusters-workloads].

## <a name="create-an-aks-cluster-with-api-server-authorized-ip-ranges-enabled"></a>Créer un cluster AKS dont les plages d’adresses IP autorisées du serveur d’API sont activées

Créez un cluster au moyen de la commande [az aks create][az-aks-create] et spécifiez le paramètre *`--api-server-authorized-ip-ranges`* pour fournir une liste de plages d’adresses IP autorisées. Ces plages d’adresses IP sont généralement des plages d’adresses utilisées par vos réseaux locaux ou IP publiques. Lorsque vous spécifiez une plage CIDR, commencez par la première adresse IP dans la plage. Par exemple, *137.117.106.90/29* est une plage valide, mais assurez-vous que vous spécifiez la première adresse IP dans la plage, par exemple *137.117.106.88/29*.

> [!IMPORTANT]
> Par défaut, votre cluster utilise l’[équilibreur de charge de la référence SKU Standard][standard-sku-lb], vous pouvez utiliser celui-ci pour configurer la passerelle sortante. Lorsque vous activez des plages d’adresses IP autorisées pour le serveur d’API lors de la création du cluster, l’adresse IP publique de votre cluster est également autorisée par défaut en plus des plages que vous indiquez. Si vous spécifiez *""* ou aucune valeur pour *`--api-server-authorized-ip-ranges`* , les plages d’adresses IP autorisées pour le serveur API seront autorisées. Notez que si vous utilisez PowerShell, utilisez *`--api-server-authorized-ip-ranges=""`* (avec le signe égal) pour éviter tout problème d’analyse.

L’exemple suivant crée un cluster à nœud unique nommé *myAKSCluster* dans le groupe de ressources *myResourceGroup*, avec les plages d’adresses IP autorisées du serveur d’API activées. Les plages d’adresses IP autorisées sont *73.140.245.0/24* :

```azurecli-interactive
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --node-count 1 \
    --vm-set-type VirtualMachineScaleSets \
    --load-balancer-sku standard \
    --api-server-authorized-ip-ranges 73.140.245.0/24 \
    --generate-ssh-keys
```

> [!NOTE]
> Vous devez ajouter ces plages à une liste verte :
> - Adresse IP publique du pare-feu
> - Toute plage qui représente les réseaux à partir desquels vous allez administrer le cluster
> - Si vous utilisez Azure Dev Spaces sur votre cluster AKS, vous devez autoriser des [plages supplémentaires en fonction de votre région][dev-spaces-ranges].
>
> La limite supérieure du nombre de plages d’adresses IP que vous pouvez spécifier est 200.
>
> La propagation des règles peut prendre jusqu’à deux minutes. Veuillez patienter pendant le test de la connexion.

### <a name="specify-the-outbound-ips-for-the-standard-sku-load-balancer"></a>Spécifier les adresses IP sortantes pour l’équilibreur de charge de la référence SKU Standard

Lors de la création d’un cluster AKS, si vous indiquez des préfixes ou des adresses IP sortantes pour le cluster, ceux-ci sont également autorisés. Par exemple :

```azurecli-interactive
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --node-count 1 \
    --vm-set-type VirtualMachineScaleSets \
    --load-balancer-sku standard \
    --api-server-authorized-ip-ranges 73.140.245.0/24 \
    --load-balancer-outbound-ips <publicIpId1>,<publicIpId2> \
    --generate-ssh-keys
```

Dans l’exemple ci-dessus, toutes les adresses IP fournies dans le paramètre *`--load-balancer-outbound-ip-prefixes`* sont autorisées avec les adresses IP dans le paramètre *`--api-server-authorized-ip-ranges`* .

Au lieu de cela, vous pouvez spécifier le paramètre *`--load-balancer-outbound-ip-prefixes`* pour autoriser les préfixes des adresses IP sortantes de l’équilibreur de charge.

### <a name="allow-only-the-outbound-public-ip-of-the-standard-sku-load-balancer"></a>Autoriser uniquement l’adresse IP publique sortante de l’équilibreur de charge de la référence SKU Standard

Lorsque vous activez les plages d’adresses IP autorisées du serveur d’API à la création du cluster, l’adresse IP publique sortante pour l’équilibreur de charge de la référence SKU Standard de votre cluster est également autorisée par défaut, en plus des plages que vous indiquez. Pour autoriser uniquement l’adresse IP publique sortante de l’équilibreur de charge de la référence SKU Standard, utilisez *0.0.0.0/32* lors de la spécification du paramètre *`--api-server-authorized-ip-ranges`* .

Dans l’exemple suivant, seule l’adresse IP publique sortante de l’équilibreur de charge de la référence SKU Standard est autorisée, et vous ne pouvez accéder au serveur d’API qu’à partir des nœuds du cluster.

```azurecli-interactive
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --node-count 1 \
    --vm-set-type VirtualMachineScaleSets \
    --load-balancer-sku standard \
    --api-server-authorized-ip-ranges 0.0.0.0/32 \
    --generate-ssh-keys
```

## <a name="update-a-clusters-api-server-authorized-ip-ranges"></a>Mettre à jour les plages d’adresses IP autorisées du serveur d’API sur un cluster

Pour mettre à jour les plages d’adresses IP autorisées du serveur API sur un cluster existant, utilisez la commande [az aks update][az-aks-update] avec les paramètres *`--api-server-authorized-ip-ranges`* ,--load-balancer-outbound-ip-prefixes *, *`--load-balancer-outbound-ips`* , ou --load-balancer-outbound-ip-prefixes*.

L’exemple suivant met à jour les plages d’adresses IP autorisées du serveur d’API sur le cluster nommé *myAKSCluster* dans le groupe de ressources *myResourceGroup*. La plage d’adresses IP à autoriser est *73.140.245.0/24* :

```azurecli-interactive
az aks update \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --api-server-authorized-ip-ranges  73.140.245.0/24
```

Vous pouvez également utiliser *0.0.0.0/32* quand vous spécifiez le paramètre *`--api-server-authorized-ip-ranges`* pour autoriser uniquement l’adresse IP publique de l’équilibreur de charge de la référence SKU Standard.

## <a name="disable-authorized-ip-ranges"></a>Désactiver les plages d’adresses IP autorisées

Pour désactiver les plages d’adresses IP autorisées, utilisez [az aks update][az-aks-update] et spécifiez une plage vide pour désactiver les plages d’adresses IP autorisées du serveur d’API. Par exemple :

```azurecli-interactive
az aks update \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --api-server-authorized-ip-ranges ""
```

## <a name="how-to-find-my-ip-to-include-in---api-server-authorized-ip-ranges"></a>Comment trouver mon adresse IP à inclure dans `--api-server-authorized-ip-ranges` ?

Vous devez ajouter vos ordinateurs de développement, vos outils ou vos adresses IP Automation à la liste de clusters AKS des plages d’adresses IP approuvées pour accéder au serveur d’API à partir de ces emplacements. 

Une autre option est de configurer une jumpbox avec les outils nécessaires à l’intérieur d’un sous-réseau distinct dans le réseau virtuel du pare-feu. Cela suppose que votre environnement dispose d’un pare-feu avec le réseau correspondant et que vous avez ajouté les adresses IP du pare-feu aux plages autorisées. De même, si vous avez forcé le tunneling du sous-réseau AKS vers le sous-réseau du pare-feu, avoir une jumpbox dans le sous-réseau du cluster est également une bonne chose.

Ajoutez une autre adresse IP aux plages approuvées à l’aide de la commande suivante.

```bash
# Retrieve your IP address
CURRENT_IP=$(dig @resolver1.opendns.com ANY myip.opendns.com +short)
# Add to AKS approved list
az aks update -g $RG -n $AKSNAME --api-server-authorized-ip-ranges $CURRENT_IP/32
```

>> [!NOTE]
> L’exemple ci-dessus ajoute les plages d’adresses IP autorisées par le serveur d’API sur le cluster. Pour désactiver les plages d’adresses IP autorisées, utilisez la commande « az aks update » et spécifiez une plage vide "". 

Une autre option consiste à utiliser la commande ci-dessous sur les systèmes Windows pour obtenir l’adresse IPv4 publique, ou vous pouvez suivre les étapes décrites dans [Rechercher votre adresse IP](https://support.microsoft.com/en-gb/help/4026518/windows-10-find-your-ip-address).

```azurepowershell-interactive
Invoke-RestMethod http://ipinfo.io/json | Select -exp ip
```

Vous pouvez également trouver cette adresse en recherchant « quelle est mon adresse IP » dans un navigateur Internet.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez activé des plages d’adresses IP autorisées pour le serveur d’API. Cette approche est une partie de la façon dont vous pouvez exécuter un cluster AKS sécurisé.

Pour plus d’informations, consultez [Concepts de sécurité pour les applications et les clusters dans AKS][concepts-security] et [Meilleures pratiques relatives aux mises à jour et à la sécurité du cluster dans Azure Kubernetes Service][operator-best-practices-cluster-security].

<!-- LINKS - external -->
[cni-networking]: https://github.com/Azure/azure-container-networking/blob/master/docs/cni.md
[dev-spaces-ranges]: ../dev-spaces/configure-networking.md#aks-cluster-network-requirements
[kubenet]: https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/#kubenet

<!-- LINKS - internal -->
[az-aks-update]: /cli/azure/ext/aks-preview/aks#ext-aks-preview-az-aks-update
[az-aks-create]: /cli/azure/aks#az-aks-create
[az-network-public-ip-list]: /cli/azure/network/public-ip#az-network-public-ip-list
[concepts-clusters-workloads]: concepts-clusters-workloads.md
[concepts-security]: concepts-security.md
[install-azure-cli]: /cli/azure/install-azure-cli
[operator-best-practices-cluster-security]: operator-best-practices-cluster-security.md
[route-tables]: ../virtual-network/manage-route-table.md
[standard-sku-lb]: load-balancer-standard.md
