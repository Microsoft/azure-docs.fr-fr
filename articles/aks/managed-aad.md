---
title: Utiliser Azure AD dans Azure Kubernetes Service
description: Découvrez comment utiliser Azure AD dans Azure Kubernetes Service (AKS)
services: container-service
ms.topic: article
ms.date: 08/26/2020
ms.author: thomasge
ms.openlocfilehash: f229075d0bad4f9522e02e30bdabc1d42bb086cf
ms.sourcegitcommit: c157b830430f9937a7fa7a3a6666dcb66caa338b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94684183"
---
# <a name="aks-managed-azure-active-directory-integration"></a>Intégration d’Azure Active Directory géré par AKS

L’intégration d’Azure AD géré par AKS est conçue pour simplifier l’expérience d’intégration d’Azure AD, où les utilisateurs devaient auparavant créer une application cliente et une application serveur et où le locataire Azure AD devait accorder des autorisations de lecture du répertoire. Dans la nouvelle version, le fournisseur de ressources AKS gère les applications cliente et serveur pour vous.

## <a name="azure-ad-authentication-overview"></a>Vue d’ensemble de l’authentification Azure AD

Les administrateurs de cluster peuvent configurer le contrôle d’accès en fonction du rôle Kubernetes (RBAC Kubernetes) en se basant sur l’identité ou l’appartenance à un groupe d’annuaires de l’utilisateur. L’authentification Azure AD est fournie aux clusters AKS à l’aide d’OpenID Connect. OpenID Connect est une couche d’identité basée sur le protocole OAuth 2.0. Pour plus d’informations sur OpenID Connect, voir la [documentation sur Open ID Connect][open-id-connect].

Pour en savoir plus sur le flux de l’intégration d’Azure AD, consultez la [documentation sur les concepts d’intégration d’Azure Active Directory](concepts-identity.md#azure-active-directory-integration).

## <a name="limitations"></a>Limites 

* L’intégration d’Azure AD géré par AKS ne peut pas être désactivée.
* Les clusters sans RBAC Kubernetes ne sont pas pris en charge pour l’intégration d’Azure AD managé par AKS
* La modification de l’abonné Azure AD associé à l’intégration d’Azure AD géré par AKS n’est pas prise en charge

## <a name="prerequisites"></a>Prérequis

* Azure CLI 2.11.0 ou une version ultérieure
* Kubectl avec une version minimale [1.18.1](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.18.md#v1181) ou [kubelogin](https://github.com/Azure/kubelogin)
* Si vous utilisez [helm](https://github.com/helm/helm), version minimale de helm 3.3.

> [!Important]
> Vous devez utiliser Kubectl avec une version minimale 1.18.1 ou kubelogin. Si vous n’utilisez pas la version appropriée, vous remarquerez des problèmes d’authentification.

Pour installer kubectl et kubelogin, utilisez les commandes suivantes :

```azurecli-interactive
sudo az aks install-cli
kubectl version --client
kubelogin --version
```

Pour d’autres systèmes d’exploitation, utilisez [ces instructions](https://kubernetes.io/docs/tasks/tools/install-kubectl/).


## <a name="before-you-begin"></a>Avant de commencer

Pour votre cluster, vous avez besoin d’un groupe Azure AD. Le cluster a besoin de ce groupe comme groupe d’administration afin d’accorder des autorisations d’administrateur de cluster. Vous pouvez utiliser un groupe Azure AD existant ou en créer un. Enregistrez l’ID d’objet de votre groupe Azure AD.

```azurecli-interactive
# List existing groups in the directory
az ad group list --filter "displayname eq '<group-name>'" -o table
```

Pour créer un groupe Azure AD pour vos administrateurs de cluster, utilisez la commande suivante :

```azurecli-interactive
# Create an Azure AD group
az ad group create --display-name myAKSAdminGroup --mail-nickname myAKSAdminGroup
```

## <a name="create-an-aks-cluster-with-azure-ad-enabled"></a>Créer un cluster AKS compatible Azure AD

Créez un cluster AKS à l’aide des commandes CLI suivantes.

Créez un groupe de ressources Azure :

```azurecli-interactive
# Create an Azure resource group
az group create --name myResourceGroup --location centralus
```

Créer un cluster AKS et activer l’accès administrateur pour votre groupe Azure AD

```azurecli-interactive
# Create an AKS-managed Azure AD cluster
az aks create -g myResourceGroup -n myManagedCluster --enable-aad --aad-admin-group-object-ids <id> [--aad-tenant-id <id>]
```

Lorsqu’un cluster Azure AD géré par AKS a bien été créé, vous voyez la section suivante dans le corps de la réponse.
```output
"AADProfile": {
    "adminGroupObjectIds": [
      "5d24****-****-****-****-****afa27aed"
    ],
    "clientAppId": null,
    "managed": true,
    "serverAppId": null,
    "serverAppSecret": null,
    "tenantId": "72f9****-****-****-****-****d011db47"
  }
```

Une fois le cluster créé, vous pouvez commencer à y accéder.

## <a name="access-an-azure-ad-enabled-cluster"></a>Accéder à un cluster compatible Azure AD

Pour effectuer les étapes ci-dessous, vous avez besoin du rôle intégré [Azure Kubernetes Service Cluster User](../role-based-access-control/built-in-roles.md#azure-kubernetes-service-cluster-user-role) (Utilisateur de cluster Azure Kubernetes Service).

Récupérez les informations d’identification d’utilisateur permettant d’accéder au cluster :
 
```azurecli-interactive
 az aks get-credentials --resource-group myResourceGroup --name myManagedCluster
```
Suivez les instructions pour vous connecter.

Utilisez la commande kubectl get nodes pour voir les nœuds figurant dans le cluster :

```azurecli-interactive
kubectl get nodes

NAME                       STATUS   ROLES   AGE    VERSION
aks-nodepool1-15306047-0   Ready    agent   102m   v1.15.10
aks-nodepool1-15306047-1   Ready    agent   102m   v1.15.10
aks-nodepool1-15306047-2   Ready    agent   102m   v1.15.10
```
Configurez le [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](./azure-ad-rbac.md) afin de configurer des groupes de sécurité supplémentaires pour vos clusters.

## <a name="troubleshooting-access-issues-with-azure-ad"></a>Résolution des problèmes d’accès avec Azure AD

> [!Important]
> Les étapes décrites ci-dessous contournent l’authentification normale des groupes Azure AD. Effectuez ces étapes uniquement en cas d’urgence.

Si vous restez bloqué sans pouvoir accéder à aucun groupe Azure AD valide ayant accès à votre cluster, vous pouvez obtenir les informations d’identification d’administrateur qui vous permettront d’accéder au cluster directement.

Pour effectuer ces étapes, vous devez avoir accès au rôle intégré [Azure Kubernetes Service Cluster Admin](../role-based-access-control/built-in-roles.md#azure-kubernetes-service-cluster-admin-role).

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myManagedCluster --admin
```

## <a name="enable-aks-managed-azure-ad-integration-on-your-existing-cluster"></a>Activer l’intégration de Azure AD géré par AKS sur votre cluster existant

Vous pouvez activer l’intégration d’Azure AD géré par AKS sur votre cluster avec RBAC Kubernetes existant. Veillez à définir votre groupe d’administration pour conserver l’accès à votre cluster.

```azurecli-interactive
az aks update -g MyResourceGroup -n MyManagedCluster --enable-aad --aad-admin-group-object-ids <id-1> [--aad-tenant-id <id>]
```

Lorsqu’un cluster Azure AD géré par AKS a bien été activé, vous voyez la section suivante dans le corps de la réponse

```output
"AADProfile": {
    "adminGroupObjectIds": [
      "5d24****-****-****-****-****afa27aed"
    ],
    "clientAppId": null,
    "managed": true,
    "serverAppId": null,
    "serverAppSecret": null,
    "tenantId": "72f9****-****-****-****-****d011db47"
  }
```

Téléchargez à nouveau les informations d’identification de l’utilisateur pour accéder à votre cluster en suivant les étapes [ici][access-cluster].

## <a name="upgrading-to-aks-managed-azure-ad-integration"></a>Mise à niveau vers l’intégration d’Azure AD géré par AKS

Si votre cluster utilise l’intégration d’Azure AD héritée, vous pouvez effectuer la mise à niveau vers l’intégration d’Azure AD géré par AKS.

```azurecli-interactive
az aks update -g myResourceGroup -n myManagedCluster --enable-aad --aad-admin-group-object-ids <id> [--aad-tenant-id <id>]
```

Lorsqu’un cluster Azure AD géré par AKS a bien été migré, vous voyez la section suivante dans le corps de la réponse.

```output
"AADProfile": {
    "adminGroupObjectIds": [
      "5d24****-****-****-****-****afa27aed"
    ],
    "clientAppId": null,
    "managed": true,
    "serverAppId": null,
    "serverAppSecret": null,
    "tenantId": "72f9****-****-****-****-****d011db47"
  }
```

Si vous souhaitez accéder au cluster, suivez les étapes décrites [ici][access-cluster].

## <a name="non-interactive-sign-in-with-kubelogin"></a>Connexion non interactive avec kubelogin

Certains scénarios non interactifs, tels que les pipelines d’intégration continue, ne sont pas possibles avec kubectl. Utilisez [`kubelogin`](https://github.com/Azure/kubelogin) pour accéder au cluster à l’aide d’une connexion non interactive avec le principal de service.

## <a name="next-steps"></a>Étapes suivantes

* Apprenez-en davantage sur l’[intégration d’Azure RBAC pour l’autorisation Kubernetes][azure-rbac-integration].
* Apprenez-en davantage sur l’[intégration d’Azure AD avec Kubernetes RBAC][azure-ad-rbac].
* Utilisez [kubelogin](https://github.com/Azure/kubelogin) pour accéder aux fonctionnalités de l’authentification Azure non disponibles dans kubectl.
* Apprenez-en davantage sur les [concepts d’identité AKS et Kubernetes][aks-concepts-identity].
* Utilisez des [modèles ARM (Azure Resource Manager)][aks-arm-template] pour créer des clusters avec Azure AD géré par AKS.

<!-- LINKS - external -->
[kubernetes-webhook]:https://kubernetes.io/docs/reference/access-authn-authz/authentication/#webhook-token-authentication
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[aks-arm-template]: /azure/templates/microsoft.containerservice/managedclusters

<!-- LINKS - Internal -->
[azure-rbac-integration]: manage-azure-rbac.md
[aks-concepts-identity]: concepts-identity.md
[azure-ad-rbac]: azure-ad-rbac.md
[az-aks-create]: /cli/azure/aks?view=azure-cli-latest#az-aks-create
[az-aks-get-credentials]: /cli/azure/aks?view=azure-cli-latest#az-aks-get-credentials
[az-group-create]: /cli/azure/group#az-group-create
[open-id-connect]:../active-directory/develop/v2-protocols-oidc.md
[az-ad-user-show]: /cli/azure/ad/user#az-ad-user-show
[rbac-authorization]: concepts-identity.md#role-based-access-controls-rbac
[operator-best-practices-identity]: operator-best-practices-identity.md
[azure-ad-rbac]: azure-ad-rbac.md
[azure-ad-cli]: azure-ad-integration-cli.md
[access-cluster]: #access-an-azure-ad-enabled-cluster
[aad-migrate]: #upgrading-to-aks-managed-azure-ad-integration
