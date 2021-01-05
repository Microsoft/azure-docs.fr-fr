---
title: Limiter l’accès à kubeconfig dans Azure Kubernetes Service (AKS)
description: Découvrir comment contrôler l’accès au fichier de configuration Kubernetes (kubeconfig) pour les administrateurs et utilisateurs de cluster
services: container-service
ms.topic: article
ms.date: 05/06/2020
ms.openlocfilehash: 371628b02ebecee23697e996ee0d484688167875
ms.sourcegitcommit: c157b830430f9937a7fa7a3a6666dcb66caa338b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94684812"
---
# <a name="use-azure-role-based-access-control-to-define-access-to-the-kubernetes-configuration-file-in-azure-kubernetes-service-aks"></a>Utiliser le contrôle d’accès en fonction du rôle Azure pour définir l’accès au fichier config Kubernetes dans Azure Kubernetes Service (AKS)

Vous pouvez interagir avec les clusters Kubernetes à l’aide de l’outil `kubectl`. L’interface Azure CLI fournit un moyen simple qui permet d’obtenir les informations de configuration et d’identification de l’accès pour vous connecter à vos clusters AKS avec `kubectl`. Pour limiter les utilisateurs pouvant obtenir ces informations de configuration Kubernetes (*kubeconfig*) et limiter les autorisations qui en découlent, vous pouvez utiliser le contrôle d’accès en fonction du rôle Azure (Azure RBAC).

Cet article vous montre comment attribuer des rôles Azure qui limitent les utilisateurs pouvant récupérer les informations de configuration d’un cluster AKS.

## <a name="before-you-begin"></a>Avant de commencer

Cet article suppose que vous avez un cluster AKS existant. Si vous avez besoin d’un cluster AKS, consultez le guide de démarrage rapide d’AKS [avec Azure CLI][aks-quickstart-cli]ou avec le [Portail Azure][aks-quickstart-portal].

Pour les besoins de cet article, vous devez aussi exécuter Azure CLI version 2.0.65 ou ultérieure. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI][azure-cli-install].

## <a name="available-cluster-roles-permissions"></a>Autorisations des rôles de cluster disponibles

Lorsque vous interagissez avec un cluster AKS par l’intermédiaire de l’outil `kubectl`, un fichier de configuration est utilisé pour définir les informations de connexion au cluster. Ce fichier de configuration est généralement stocké dans *~/.kube/config*. Plusieurs clusters peuvent être définis dans ce fichier *kubeconfig*. Vous basculez entre les clusters à l’aide de la commande [kubectl config use-context][kubectl-config-use-context].

La commande [az aks get-credentials][az-aks-get-credentials] vous permet d’obtenir les informations d’identification de l’accès à un cluster AKS et les fusionne dans le fichier *kubeconfig*. Vous pouvez utiliser le contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour contrôler l’accès à ces informations d’identification. Ces rôles Azure vous permettent de définir les utilisateurs qui peuvent récupérer le fichier *kubeconfig* et les autorisations qui en découlent au sein du cluster.

Les deux rôles intégrés sont :

* **Rôle d’administrateur de cluster Azure Kubernetes Service**  
  * Permet l’accès à l’appel d’API *Microsoft.ContainerService/managedClusters/listClusterAdminCredential/action*. Cet appel d’API [répertorie les informations d’identification de l’administrateur de cluster][api-cluster-admin].
  * Télécharge *kubeconfig* pour le rôle *clusterAdmin*.
* **Rôle d’utilisateur de cluster Azure Kubernetes Service**
  * Permet l’accès à l’appel d’API *Microsoft.ContainerService/managedClusters/listClusterUserCredential/action*. Cet appel d’API [répertorie les informations d’identification de l’utilisateur de cluster][api-cluster-user].
  * Télécharge *kubeconfig* pour le rôle *clusterUser*.

Ces rôles Azure peuvent être appliqués à un utilisateur ou groupe Azure Active Directory (AD).

> [!NOTE]
> Sur les clusters qui utilisent Azure AD, les utilisateurs disposant du rôle *clusterUser* ont un fichier *kubeconfig* vide qui les invite à se connecter. Une fois connectés, les utilisateurs disposent d’un accès en fonction de leurs paramètres utilisateur ou de groupe Azure AD. Les utilisateurs disposant du rôle *clusterAdmin* ont un accès administrateur.
>
> Les clusters qui n’utilisent pas Azure AD utilisent uniquement le rôle *clusterAdmin*.

## <a name="assign-role-permissions-to-a-user-or-group"></a>Affecter des autorisations de rôle à un utilisateur ou groupe

Pour affecter un des rôles disponibles, vous devez obtenir l’ID de ressource du cluster AKS et l’ID du compte d’utilisateur ou groupe Azure AD. L’exemple suivant :

* Obtient l’ID de la ressource du cluster à l’aide de la commande [az aks show][az-aks-show] pour le cluster nommé *myAKSCluster* dans le groupe de ressources *myResourceGroup*. Fournissez votre propre nom de groupe de ressources et de cluster, au besoin.
* Utilisez les commandes [az account show][az-account-show] et [az ad user show][az-ad-user-show] pour obtenir votre ID d’utilisateur.
* Enfin, attribuez un rôle à l’aide de la commande [az role assignment create][az-role-assignment-create].

L’exemple suivant affecte le *Rôle d’administrateur de cluster Azure Kubernetes Service* à un compte d’utilisateur individuel :

```azurecli-interactive
# Get the resource ID of your AKS cluster
AKS_CLUSTER=$(az aks show --resource-group myResourceGroup --name myAKSCluster --query id -o tsv)

# Get the account credentials for the logged in user
ACCOUNT_UPN=$(az account show --query user.name -o tsv)
ACCOUNT_ID=$(az ad user show --id $ACCOUNT_UPN --query objectId -o tsv)

# Assign the 'Cluster Admin' role to the user
az role assignment create \
    --assignee $ACCOUNT_ID \
    --scope $AKS_CLUSTER \
    --role "Azure Kubernetes Service Cluster Admin Role"
```

> [!TIP]
> Si vous voulez attribuer des autorisations à un groupe Azure AD, mettez à jour le paramètre `--assignee` présenté dans l’exemple précédent avec l’ID d’objet d’un *groupe* et non d’un *utilisateur*. Pour obtenir l’ID d’objet d’un groupe, utilisez la commande [az ad group show][az-ad-group-show]. L’exemple suivant obtient l’ID d’objet du groupe Azure AD nommé *appdev* : `az ad group show --group appdev --query objectId -o tsv`

Vous pouvez modifier l’affectation précédente au *rôle d’utilisateur de cluster*, le cas échéant.

L’exemple de sortie suivant montre que les attributions de rôle ont été créées correctement :

```
{
  "canDelegate": null,
  "id": "/subscriptions/<guid>/resourcegroups/myResourceGroup/providers/Microsoft.ContainerService/managedClusters/myAKSCluster/providers/Microsoft.Authorization/roleAssignments/b2712174-5a41-4ecb-82c5-12b8ad43d4fb",
  "name": "b2712174-5a41-4ecb-82c5-12b8ad43d4fb",
  "principalId": "946016dd-9362-4183-b17d-4c416d1f8f61",
  "resourceGroup": "myResourceGroup",
  "roleDefinitionId": "/subscriptions/<guid>/providers/Microsoft.Authorization/roleDefinitions/0ab01a8-8aac-4efd-b8c2-3ee1fb270be8",
  "scope": "/subscriptions/<guid>/resourcegroups/myResourceGroup/providers/Microsoft.ContainerService/managedClusters/myAKSCluster",
  "type": "Microsoft.Authorization/roleAssignments"
}
```

## <a name="get-and-verify-the-configuration-information"></a>Obtenir et vérifier les informations de configuration

À l’aide des rôles Azure attribués, utilisez la commande [az aks get-credentials][az-aks-get-credentials] pour obtenir la définition *kubeconfig* de votre cluster AKS. L’exemple suivant obtient les informations d’identification *--admin*, ce qui fonctionne correctement si l’utilisateur a reçu le *rôle d’administrateur de cluster* :

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster --admin
```

Vous pouvez ensuite utiliser la commande [kubectl config view][kubectl-config-view] pour vérifier que le *contexte* du cluster indique que les informations de configuration de l’administrateur ont été appliquées :

```
$ kubectl config view

apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://myaksclust-myresourcegroup-19da35-4839be06.hcp.eastus.azmk8s.io:443
  name: myAKSCluster
contexts:
- context:
    cluster: myAKSCluster
    user: clusterAdmin_myResourceGroup_myAKSCluster
  name: myAKSCluster-admin
current-context: myAKSCluster-admin
kind: Config
preferences: {}
users:
- name: clusterAdmin_myResourceGroup_myAKSCluster
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
    token: e9f2f819a4496538b02cefff94e61d35
```

## <a name="remove-role-permissions"></a>Supprimer des autorisations de rôle

Pour supprimer des attributions de rôle, utilisez la commande [az role assignment delete][az-role-assignment-delete]. Spécifiez l’ID de compte et l’ID de ressource de cluster, tels qu’ils ont été obtenus dans les commandes précédentes. Si vous avez attribué le rôle à un groupe plutôt qu’à un utilisateur, spécifiez l’ID d’objet de groupe approprié au lieu de l’ID d’objet de compte pour le paramètre `--assignee` :

```azurecli-interactive
az role assignment delete --assignee $ACCOUNT_ID --scope $AKS_CLUSTER
```

## <a name="next-steps"></a>Étapes suivantes

Pour mieux sécuriser l’accès aux clusters AKS, [intégrez l’authentification Azure Active Directory][aad-integration].

<!-- LINKS - external -->
[kubectl-config-use-context]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#config
[kubectl-config-view]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#config

<!-- LINKS - internal -->
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[azure-cli-install]: /cli/azure/install-azure-cli
[az-aks-get-credentials]: /cli/azure/aks#az-aks-get-credentials
[azure-rbac]: ../role-based-access-control/overview.md
[api-cluster-admin]: /rest/api/aks/managedclusters/listclusteradmincredentials
[api-cluster-user]: /rest/api/aks/managedclusters/listclusterusercredentials
[az-aks-show]: /cli/azure/aks#az-aks-show
[az-account-show]: /cli/azure/account#az-account-show
[az-ad-user-show]: /cli/azure/ad/user#az-ad-user-show
[az-role-assignment-create]: /cli/azure/role/assignment#az-role-assignment-create
[az-role-assignment-delete]: /cli/azure/role/assignment#az-role-assignment-delete
[aad-integration]: ./azure-ad-integration-cli.md
[az-ad-group-show]: /cli/azure/ad/group#az-ad-group-show
