---
title: Réinitialiser les informations d’identification pour un cluster
titleSuffix: Azure Kubernetes Service
description: Découvrir comment mettre à jour ou réinitialiser les informations d’identification d’un principal de service ou d’une application AAD pour un cluster Azure Kubernetes Service (AKS)
services: container-service
ms.topic: article
ms.date: 03/11/2019
ms.openlocfilehash: c787f172bc03e11c574c4de967aee05da9df18aa
ms.sourcegitcommit: 0dcafc8436a0fe3ba12cb82384d6b69c9a6b9536
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94427511"
---
# <a name="update-or-rotate-the-credentials-for-azure-kubernetes-service-aks"></a>Mettre à jour ou faire tourner les informations d’identification pour Azure Kubernetes Service (AKS)

Par défaut, les clusters AKS sont créés avec un principal de service dont le délai d’expiration est d’un an. À mesure que vous approchez de la date d’expiration, vous pouvez réinitialiser les informations d’identification afin de prolonger le délai d’expiration du principal de service. Vous souhaiterez peut-être également mettre à jour, ou faire pivoter, les informations d’identification dans le cadre d’une stratégie de sécurité définie. Cet article explique comment mettre à jour ces informations d’identification pour un cluster AKS.

Vous avez peut-être également [intégré votre cluster AKS à Azure Active Directory][aad-integration] et l’utilisez comme fournisseur d’authentification pour votre cluster. Dans ce cas, vous aurez 2 autres identités créées pour votre cluster, l’application de serveur AAD et l’application de client AAD ; vous pouvez également réinitialiser ces informations d’identification.

Pour les autorisations, vous pouvez également utiliser une identité managée au lieu d’un principal de service. Les identités managées sont plus faciles à gérer que les principaux de service et ne nécessitent pas de mises à jour ni de rotations. Pour plus d’informations, consultez [Utiliser des identités managées](use-managed-identity.md).

## <a name="before-you-begin"></a>Avant de commencer

Azure CLI version 2.0.65 ou ultérieure doit être installé et configuré. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI][install-azure-cli].

## <a name="update-or-create-a-new-service-principal-for-your-aks-cluster"></a>Mise à jour ou création d’un principal de service pour le cluster AKS

Lorsque vous souhaitez mettre à jour les informations d’identification d’un cluster AKS, vous pouvez choisir entre les solutions suivantes :

* Mettre à jour les informations d’identification du principal de service existant
* Créer un principal de service et mettre à jour le cluster pour qu’il utilise ces nouvelles informations d’identification 

> [!WARNING]
> Si vous choisissez de créer un *nouveau* principal de service, la mise à jour d’un cluster AKS de grande taille de sorte qu’il utilise ces informations d’identification est susceptible de prendre beaucoup de temps.

### <a name="check-the-expiration-date-of-your-service-principal"></a>Vérifier la date d’expiration de votre principal de service

Pour vérifier la date d’expiration de votre principal de service, utilisez la commande [az ad sp credential list][az-ad-sp-credential-list]. L’exemple suivant obtient l’ID du principal de service pour le cluster nommé *myAKSCluster* dans le groupe de ressources *myResourceGroup* à l’aide de la commande [az aks show][az-aks-show]. L’ID du principal de service est défini en tant que variable nommée *SP_ID* pour être utilisé dans la commande [az ad sp credential list][az-ad-sp-credential-list].

```azurecli
SP_ID=$(az aks show --resource-group myResourceGroup --name myAKSCluster \
    --query servicePrincipalProfile.clientId -o tsv)
az ad sp credential list --id $SP_ID --query "[].endDate" -o tsv
```

### <a name="reset-the-existing-service-principal-credential"></a>Réinitialisation des informations d’identification du principal de service existant

Pour mettre à jour les informations d’identification du principal de service existant, obtenez l’ID du principal de service de votre cluster à l’aide de la commande [az aks show][az-aks-show]. L’exemple suivant permet d’obtenir l’ID du cluster *myAKSCluster* dans le groupe de ressources *myResourceGroup*. L’ID du principal du service est défini en tant que variable nommée *SP_ID* pour être utilisé dans d’autres commandes. Ces commandes utilisent la syntaxe Bash.

```azurecli-interactive
SP_ID=$(az aks show --resource-group myResourceGroup --name myAKSCluster \
    --query servicePrincipalProfile.clientId -o tsv)
```

Avec un jeu de variables contenant l’ID du principal de service, réinitialisez les informations d’identification en utilisant [az ad sp credential reset][az-ad-sp-credential-reset]. L’exemple suivant permet à la plateforme Azure de générer un nouveau secret sécurisé pour le principal du service. Ce nouveau secret sécurisé est également stocké dans une variable.

```azurecli-interactive
SP_SECRET=$(az ad sp credential reset --name $SP_ID --query password -o tsv)
```

Passez maintenant à [mettre à jour le cluster AKS avec les nouvelles informations d’identification du principal de service](#update-aks-cluster-with-new-service-principal-credentials). Cette étape est nécessaire pour que les modifications apportées au principal de service soient reflétées sur le cluster AKS.

### <a name="create-a-new-service-principal"></a>Création d’un principal de service

Si vous avez choisi de mettre à jour les informations d’identification du principal du service existantes dans la section précédente, ignorez cette étape. Continuez pour [mettre à jour le cluster AKS avec les nouvelles informations d’identification du principal de service](#update-aks-cluster-with-new-service-principal-credentials).

Pour créer un principal de service puis mettre à jour le cluster AKS pour qu’il utilise ces nouvelles informations d’identification, utilisez la commande [az ad sp create-for-rbac][az-ad-sp-create]. Dans l’exemple suivant, le paramètre `--skip-assignment` empêche toute affectation par défaut supplémentaire :

```azurecli-interactive
az ad sp create-for-rbac --skip-assignment
```

Le résultat ressemble à l’exemple qui suit. Prenez note de vos propres valeurs pour `appId` et `password`. Ces valeurs sont utilisées à l’étape suivante.

```json
{
  "appId": "7d837646-b1f3-443d-874c-fd83c7c739c5",
  "name": "7d837646-b1f3-443d-874c-fd83c7c739c",
  "password": "a5ce83c9-9186-426d-9183-614597c7f2f7",
  "tenant": "a4342dc8-cd0e-4742-a467-3129c469d0e5"
}
```

Définissez maintenant des variables pour l’ID du principal de service et la clé secrète client à l’aide de la sortie de votre propre commande [az ad sp create-for-rbac][az-ad-sp-create], comme indiqué dans l’exemple suivant. *SP_ID* correspond à votre *appId* , et *SP_SECRET* à votre *mot de passe* :

```console
SP_ID=7d837646-b1f3-443d-874c-fd83c7c739c5
SP_SECRET=a5ce83c9-9186-426d-9183-614597c7f2f7
```

Passez maintenant à [mettre à jour le cluster AKS avec les nouvelles informations d’identification du principal de service](#update-aks-cluster-with-new-service-principal-credentials). Cette étape est nécessaire pour que les modifications apportées au principal de service soient reflétées sur le cluster AKS.

## <a name="update-aks-cluster-with-new-service-principal-credentials"></a>Mise à jour du cluster AKS avec les informations d’identification du nouveau principal de service

> [!IMPORTANT]
> La mise à jour d’un cluster AKS de grande taille avec un nouveau principal de service est susceptible de prendre beaucoup de temps.

Que vous ayez choisi de mettre à jour les informations d’identification du principal de service existant ou de créer un principal de service, mettez maintenant à jour le cluster AKS avec vos nouvelles informations d’identification à l’aide de la commande [az aks update-credentials][az-aks-update-credentials]. Les variables pour *--service-principal* et *--client-secret* sont utilisées :

```azurecli-interactive
az aks update-credentials \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --reset-service-principal \
    --service-principal $SP_ID \
    --client-secret $SP_SECRET
```

La mise à jour des informations d’identification du principal de service dans AKS prend quelques instants pour les clusters de petite taille et de taille moyenne.

## <a name="update-aks-cluster-with-new-aad-application-credentials"></a>Mettre à jour le cluster AKS avec les nouvelles informations d’identification de l’application AAD

Vous pouvez créer des applications de serveur et de client AAD en suivant les [Étapes de l’intégration AAD][create-aad-app]. Sinon, réinitialisez vos applications AAD existantes en suivant la [même méthode de réinitialisation que celle du principal de service](#reset-the-existing-service-principal-credential). Après cela, il vous suffit de mettre à jour les informations d’identification de votre application AAD sur le cluster à l’aide de la même commande [az aks update-credentials][az-aks-update-credentials], mais en utilisant les variables *--reset-aad*.

```azurecli-interactive
az aks update-credentials \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --reset-aad \
    --aad-server-app-id <SERVER APPLICATION ID> \
    --aad-server-app-secret <SERVER APPLICATION SECRET> \
    --aad-client-app-id <CLIENT APPLICATION ID>
```


## <a name="next-steps"></a>Étapes suivantes

Dans cet article, le principal de service du cluster AKS lui-même et les applications d’intégration AAD ont été mis à jour. Pour plus d’informations sur la gestion de l’identité des charges de travail dans un cluster, consultez la section [Meilleures pratiques d’authentification et d’autorisation dans AKS][best-practices-identity].

<!-- LINKS - internal -->
[install-azure-cli]: /cli/azure/install-azure-cli
[az-aks-show]: /cli/azure/aks#az-aks-show
[az-aks-update-credentials]: /cli/azure/aks#az-aks-update-credentials
[best-practices-identity]: operator-best-practices-identity.md
[aad-integration]: ./azure-ad-integration-cli.md
[create-aad-app]: ./azure-ad-integration-cli.md#create-azure-ad-server-component
[az-ad-sp-create]: /cli/azure/ad/sp#az-ad-sp-create-for-rbac
[az-ad-sp-credential-list]: /cli/azure/ad/sp/credential#az-ad-sp-credential-list
[az-ad-sp-credential-reset]: /cli/azure/ad/sp/credential#az-ad-sp-credential-reset
