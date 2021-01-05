---
title: Didacticiel Kubernetes sur Azure – Déployer un cluster
description: Dans ce didacticiel Azure Kubernetes Service (AKS), vous créez un cluster AKS et utilisez kubectl pour vous connecter au nœud principal Kubernetes.
services: container-service
ms.topic: tutorial
ms.date: 09/30/2020
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 0e034ebede39a3fd9046ced9716323d0c7d874df
ms.sourcegitcommit: c157b830430f9937a7fa7a3a6666dcb66caa338b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94684068"
---
# <a name="tutorial-deploy-an-azure-kubernetes-service-aks-cluster"></a>Tutoriel : Déployer un cluster Azure Kubernetes Service (AKS)

Kubernetes fournit une plateforme distribuée destinée aux applications en conteneur. Avec AKS, vous pouvez créer rapidement un cluster Kubernetes prêt pour la production. Dans ce tutoriel (troisième d’une série de sept), un cluster Kubernetes est déployé dans AKS. Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Déployer un cluster Kubernetes AKS qui peut s’authentifier auprès d’un registre de conteneurs Azure
> * Installer l’interface de ligne de commande Kubernetes (kubectl)
> * Configurer kubectl pour se connecter à votre cluster AKS

Dans d’autres tutoriels, l’application Azure Vote est déployée sur le cluster, mise à l’échelle et mise à jour.

## <a name="before-you-begin"></a>Avant de commencer

Dans les tutoriels précédents, une image conteneur a été créée et chargée dans une instance Azure Container Registry. Si vous n’avez pas effectué ces étapes et que vous souhaitez suivre cette procédure, commencez par le [Tutoriel 1 : Créer des images conteneur][aks-tutorial-prepare-app].

Ce didacticiel nécessite l’exécution de l’interface de ligne de commande Azure CLI version 2.0.53 ou ultérieure. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI][azure-cli-install].

## <a name="create-a-kubernetes-cluster"></a>Créer un cluster Kubernetes

Les clusters AKS peuvent utiliser le contrôle d’accès en fonction du rôle Kubernetes (RBAC Kubernetes). Ces contrôles vous permettent de définir l’accès aux ressources en fonction des rôles attribués aux utilisateurs. Des autorisations sont combinées si plusieurs rôles sont attribués à un utilisateur, et les autorisations peuvent être limitées à un seul espace de noms ou accordées à l’ensemble du cluster. Par défaut, l’interface Azure CLI active automatiquement RBAC Kubernetes lorsque vous créez un cluster AKS.

Créez un cluster AKS à l’aide de [az aks create][]. L’exemple suivant crée un cluster nommé *myAKSCluster* dans le groupe de ressources nommé *myResourceGroup*. Vous avez créé ce groupe de ressources dans le [tutoriel précédent][aks-tutorial-prepare-acr] dans la région *eastus*. L’exemple suivant ne spécifie pas de région ; le cluster AKS est donc également créé dans la région *eastus*. Pour plus d’informations sur les limites de ressources et la disponibilité des régions pour AKS, consultez [Quotas, restrictions de taille de machine virtuelle et disponibilité des régions dans Azure Kubernetes Service (AKS)][quotas-skus-regions].

Pour permettre à un cluster AKS d’interagir avec d’autres ressources Azure, un principal de service Azure Active Directory est automatiquement créé, dans la mesure où vous n’en avez pas spécifié. Ici, ce principal de service [se voit accorder le droit d’extraire des images][container-registry-integration] à partir de l’instance Azure Container Registry (ACR) que vous avez créée dans le tutoriel précédent. Notez que vous pouvez utiliser une [identité managée](use-managed-identity.md) au lieu d’un principal de service pour une gestion simplifiée.

```azurecli
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --node-count 2 \
    --generate-ssh-keys \
    --attach-acr <acrName>
```

Vous pouvez également configurer manuellement un principal de service pour extraire des images d’ACR. Pour plus d’informations, consultez [Authentification ACR à l’aide de principaux de service](../container-registry/container-registry-auth-service-principal.md) ou [S’authentifier à partir de Kubernetes avec un secret de tirage (pull)](../container-registry/container-registry-auth-kubernetes.md).

Après quelques minutes, le déploiement se termine et retourne des informations au format JSON concernant le déploiement AKS.

> [!NOTE]
> Pour garantir un fonctionnement fiable de votre cluster, vous devez exécuter au moins 2 (deux) nœuds.

## <a name="install-the-kubernetes-cli"></a>Installer l’interface de ligne de commande Kubernetes

Pour vous connecter au cluster Kubernetes à partir de votre ordinateur local, utilisez [kubectl][kubectl], le client de ligne de commande Kubernetes.

Si vous utilisez Azure Cloud Shell, `kubectl` est déjà installé. Vous pouvez également l’installer en local à l’aide de la commande [az aks install-cli][] :

```azurecli
az aks install-cli
```

## <a name="connect-to-cluster-using-kubectl"></a>Se connecter au cluster à l’aide de kubectl

Pour configurer `kubectl` afin de vous connecter à votre cluster Kubernetes, exécutez la commande [az aks get-credentials][]. L’exemple suivant obtient les informations d’identification du cluster AKS nommé *myAKSCluster* dans le groupe de ressources *myResourceGroup* :

```azurecli
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

Pour vérifier la connexion à votre cluster, exécutez la commande [kubectl get nodes][kubectl-get] pour retourner la liste des nœuds de cluster :

```
$ kubectl get nodes

NAME                       STATUS   ROLES   AGE   VERSION
aks-nodepool1-12345678-0   Ready    agent   32m   v1.14.8
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, un cluster Kubernetes a été déployé dans ACS, et vous avez configuré `kubectl` pour qu’il s’y connecte. Vous avez appris à :

> [!div class="checklist"]
> * Déployer un cluster Kubernetes AKS qui peut s’authentifier auprès d’un registre de conteneurs Azure
> * Installer l’interface de ligne de commande Kubernetes (kubectl)
> * Configurer kubectl pour se connecter à votre cluster AKS

Passez au didacticiel suivant pour savoir comment déployer une application dans le cluster.

> [!div class="nextstepaction"]
> [Déployer une application dans Kubernetes][aks-tutorial-deploy-app]

<!-- LINKS - external -->
[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get

<!-- LINKS - internal -->
[aks-tutorial-deploy-app]: ./tutorial-kubernetes-deploy-application.md
[aks-tutorial-prepare-acr]: ./tutorial-kubernetes-prepare-acr.md
[aks-tutorial-prepare-app]: ./tutorial-kubernetes-prepare-app.md
[az ad sp create-for-rbac]: /cli/azure/ad/sp#az-ad-sp-create-for-rbac
[az acr show]: /cli/azure/acr#az-acr-show
[az role assignment create]: /cli/azure/role/assignment#az-role-assignment-create
[az aks create]: /cli/azure/aks#az-aks-create
[az aks install-cli]: /cli/azure/aks#az-aks-install-cli
[az aks get-credentials]: /cli/azure/aks#az-aks-get-credentials
[azure-cli-install]: /cli/azure/install-azure-cli
[container-registry-integration]: ./cluster-container-registry-integration.md
[quotas-skus-regions]: quotas-skus-regions.md
