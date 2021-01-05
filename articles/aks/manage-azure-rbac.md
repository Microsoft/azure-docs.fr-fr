---
title: Gérer le RBAC Azure dans Kubernetes à partir d’Azure
titleSuffix: Azure Kubernetes Service
description: Découvrez comment utiliser Azure RBAC pour l’autorisation Kubernetes avec AKS (Azure Kubernetes Service).
services: container-service
ms.topic: article
ms.date: 09/21/2020
ms.author: jpalma
author: palma21
ms.openlocfilehash: a2a385b2be4e1005a7aabd76261b3190ecd2a506
ms.sourcegitcommit: c157b830430f9937a7fa7a3a6666dcb66caa338b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94684217"
---
# <a name="use-azure-rbac-for-kubernetes-authorization-preview"></a>Utiliser Azure RBAC pour l’autorisation Kubernetes (préversion)

Aujourd’hui, vous pouvez déjà tirer parti de l’[authentification intégrée entre Azure AD (Azure Active Directory) et AKS](managed-aad.md). Lorsqu’elle est activée, cette intégration permet aux clients d’utiliser des utilisateurs, groupes ou principaux de service Azure AD en tant qu’objets dans Kubernetes RBAC (voir [ici](azure-ad-rbac.md) pour plus d’informations).
Grâce à cette fonctionnalité, vous n’avez plus à gérer séparément les identités et les informations d’identification des utilisateurs pour Kubernetes. Toutefois, vous devez toujours configurer et gérer séparément Azure RBAC et Kubernetes RBAC. Pour plus d’informations sur l’authentification et l’autorisation avec un RBAC dans AKS, voir [ici](concepts-identity.md).

Ce document présente une nouvelle approche visant à unifier la gestion et le contrôle d’accès entre les ressources Azure, AKS et les ressources Kubernetes.

## <a name="before-you-begin"></a>Avant de commencer

La possibilité de gérer RBAC pour les ressources Kubernetes à partir d’Azure vous donne le choix de gérer RBAC pour les ressources de cluster à l’aide d’Azure ou des mécanismes Kubernetes natifs. Quand cette option est activée, les principaux Azure AD sont validés exclusivement par Azure RBAC, tandis que les comptes de service et utilisateurs Kubernetes standard sont validés exclusivement par Kubernetes RBAC. Pour plus d’informations sur l’authentification et l’autorisation avec un RBAC sur AKS, voir [ici](concepts-identity.md#azure-rbac-for-kubernetes-authorization-preview).

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

### <a name="prerequisites"></a>Prérequis 
- Vous devez aussi disposer d’Azure CLI version 2.9.0 ou ultérieure
- Vérifiez que l’indicateur de fonctionnalité `EnableAzureRBACPreview` est activé.
- Vérifiez que l’[extension CLI][az-extension-add] `aks-preview` version 0.4.55 ou ultérieure est installée.
- Vérifiez que l’outil [kubectl v1.18.3+][az-aks-install-cli] est installé.

#### <a name="register-enableazurerbacpreview-preview-feature"></a>Inscrire la fonctionnalité d’évaluation `EnableAzureRBACPreview`

Pour créer un cluster AKS qui utilise Azure RBAC pour l’autorisation Kubernetes, vous devez activer l’indicateur des fonctionnalités `EnableAzureRBACPreview` sur votre abonnement.

Inscrivez l’indicateur de fonctionnalité `EnableAzureRBACPreview` à l’aide de la commande [az feature register][az-feature-register], comme indiqué dans l’exemple suivant :

```azurecli-interactive
az feature register --namespace "Microsoft.ContainerService" --name "EnableAzureRBACPreview"
```

 Vous pouvez vérifier l’état de l’enregistrement à l’aide de la commande [az feature list][az-feature-list] :

```azurecli-interactive
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/EnableAzureRBACPreview')].{Name:name,State:properties.state}"
```

Lorsque vous êtes prêt, actualisez l’inscription du fournisseur de ressources *Microsoft.ContainerService* à l’aide de la commande [az provider register][az-provider-register] :

```azurecli-interactive
az provider register --namespace Microsoft.ContainerService
```

#### <a name="install-aks-preview-cli-extension"></a>Installer l’extension CLI de préversion d’aks

Pour créer un cluster AKS qui utilise Azure RBAC, vous avez besoin de l’extension CLI *aks-preview* version 0.4.55 ou ultérieure. Installez l’extension Azure CLI *aks-preview* à l’aide de la commande [az extension add][az-extension-add] ou installez toutes les mises à jour disponibles à l’aide de la commande [az extension update][az-extension-update] :

```azurecli-interactive
# Install the aks-preview extension
az extension add --name aks-preview

# Update the extension to make sure you have the latest version installed
az extension update --name aks-preview
```

### <a name="limitations"></a>Limites

- Nécessite l’[intégration managée d’Azure AD](managed-aad.md).
- Vous ne pouvez pas intégrer Azure RBAC pour l’autorisation Kubernetes dans les clusters existants durant la phase de préversion, mais vous serez en mesure de le faire au moment de la mise à disposition générale.
- Utilisez [kubectl v1.18.3+][az-aks-install-cli].
- Si vous avez des définitions CRD et que vous créez des définitions de rôle personnalisées, le seul moyen actuellement de couvrir les CRD est de spécifier `Microsoft.ContainerService/managedClusters/*/read`. AKS travaille actuellement à fournir des autorisations plus précises pour les définitions CRD. Pour les autres objets, vous pouvez utiliser les groupes d’API spécifiques, par exemple : `Microsoft.ContainerService/apps/deployments/read`.
- Les nouvelles attributions de rôles peuvent prendre jusqu’à 5 min pour se propager et être mises à jour par le serveur d’autorisation.
- Nécessite que le locataire Azure AD configuré pour l’authentification soit le même que le locataire de l’abonnement qui contient le cluster AKS. 

## <a name="create-a-new-cluster-using-azure-rbac-and-managed-azure-ad-integration"></a>Créer un cluster en utilisant Azure RBAC et l’intégration managée d’Azure AD

Créez un cluster AKS à l’aide des commandes CLI suivantes.

Créez un groupe de ressources Azure :

```azurecli-interactive
# Create an Azure resource group
az group create --name myResourceGroup --location westus2
```

Créez le cluster AKS avec l’intégration managée d’Azure AD et Azure RBAC pour l’autorisation Kubernetes.

```azurecli-interactive
# Create an AKS-managed Azure AD cluster
az aks create -g MyResourceGroup -n MyManagedCluster --enable-aad --enable-azure-rbac
```

Quand la création d’un cluster avec l’intégration d’Azure AD et Azure RBAC pour l’autorisation Kubernetes a réussi, vous voyez la section suivante dans le corps de la réponse :

```json
"AADProfile": {
    "adminGroupObjectIds": null,
    "clientAppId": null,
    "enableAzureRbac": true,
    "managed": true,
    "serverAppId": null,
    "serverAppSecret": null,
    "tenantId": "****-****-****-****-****"
  }
```

## <a name="create-role-assignments-for-users-to-access-cluster"></a>Créer des attributions de rôles pour permettre aux utilisateurs d’accéder au cluster

AKS fournit les quatre rôles intégrés décrits ci-après :


| Role                                | Description  |
|-------------------------------------|--------------|
| Azure Kubernetes Service RBAC Viewer  | Autorise l’accès en lecture seule pour voir la plupart des objets dans un espace de noms. Ce rôle n’autorise pas l’affichage des rôles et des liaisons de rôles. Il n’autorise pas l’affichage des `Secrets`, car la lecture du contenu de Secrets donne accès aux informations d’identification ServiceAccount dans l’espace de noms, ce qui permet l’accès aux API comme n’importe quel ServiceAccount dans l’espace de noms (une forme d’élévation de privilèges)  |
| Azure Kubernetes Service RBAC Writer | Autorise l’accès en lecture/écriture pour la plupart des objets dans un espace de noms. Ce rôle n’autorise pas l’affichage ni la modification des rôles et des liaisons de rôles. Toutefois, il permet d’accéder à `Secrets` et d’exécuter des pods comme tout ServiceAccount dans l’espace de noms. Il peut donc être utilisé pour obtenir les niveaux d’accès API de n’importe quel ServiceAccount dans l’espace de noms. |
| Azure Kubernetes Service RBAC Admin  | Autorise l’accès administrateur, normalement accordé au sein d’un espace de noms. Autorise l’accès en lecture/écriture à la plupart des ressources dans un espace de noms (ou dans l’étendue du cluster), y compris la possibilité de créer des rôles et des liaisons de rôles dans l’espace de noms. Ce rôle n’autorise pas l’accès en écriture au quota de ressources ou à l’espace de noms lui-même. |
| Azure Kubernetes Service RBAC Cluster Admin  | Autorise l’accès de super utilisateur qui permet d’effectuer n’importe quelle action sur toutes les ressources. Ce rôle donne un contrôle total sur l’ensemble des ressources dans le cluster et dans tous les espaces de noms. |


Les attributions de rôles étendues au **cluster AKS entier** peuvent être effectuées à partir du panneau Contrôle d’accès (IAM) de la ressource de cluster sur le portail Azure ou en utilisant les commandes Azure CLI indiquées ci-dessous :

```bash
# Get your AKS Resource ID
AKS_ID=$(az aks show -g MyResourceGroup -n MyManagedCluster --query id -o tsv)
```

```azurecli-interactive
az role assignment create --role "Azure Kubernetes Service RBAC Admin" --assignee <AAD-ENTITY-ID> --scope $AKS_ID
```

où `<AAD-ENTITY-ID>` peut être un nom d’utilisateur (par exemple, user@contoso.com) ou même le ClientID d’un principal de service.

Vous pouvez également créer des attributions de rôles étendues à un **espace de noms** spécifique au sein du cluster :

```azurecli-interactive
az role assignment create --role "Azure Kubernetes Service RBAC Viewer" --assignee <AAD-ENTITY-ID> --scope $AKS_ID/namespaces/<namespace-name>
```

Actuellement, les attributions de rôles étendues aux espaces de noms doivent être configurées via Azure CLI.


### <a name="create-custom-roles-definitions"></a>Créer des définitions de rôles personnalisées

Si vous le souhaitez, vous pouvez créer votre propre définition de rôle, puis l’attribuer comme indiqué ci-dessus.

Voici un exemple de définition de rôle qui permet à un utilisateur de lire uniquement les déploiements et rien d’autre. La liste complète des actions possibles est disponible [ici](../role-based-access-control/resource-provider-operations.md#microsoftcontainerservice).


Copiez le code JSON ci-dessous dans un fichier appelé `deploy-view.json`.

```json
{
    "Name": "AKS Deployment Viewer",
    "Description": "Lets you view all deployments in cluster/namespace.",
    "Actions": [],
    "NotActions": [],
    "DataActions": [
        "Microsoft.ContainerService/managedClusters/apps/deployments/read"  
    ],
    "NotDataActions": [],
    "assignableScopes": [
        "/subscriptions/<YOUR SUBSCRIPTION ID>"
    ]   
}
```

Remplacez `<YOUR SUBSCRIPTION ID>` par l’ID de votre abonnement, que vous pouvez obtenir en exécutant cette commande :

```azurecli-interactive
az account show --query id -o tsv
```


Créez maintenant la définition de rôle en exécutant la commande suivante à partir du dossier où vous avez enregistré `deploy-view.json` :

```azurecli-interactive
az role definition create --role-definition @deploy-view.json 
```

Attribuez votre nouvelle définition de rôle à un utilisateur ou à une autre identité de votre choix en exécutant cette commande :

```azurecli-interactive
az role assignment create --role "AKS Deployment Viewer" --assignee <AAD-ENTITY-ID> --scope $AKS_ID
```

## <a name="use-azure-rbac-for-kubernetes-authorization-with-kubectl"></a>Utiliser Azure RBAC pour l’autorisation Kubernetes avec `kubectl`

> [!NOTE]
> Vérifiez que vous avez la dernière version de kubectl en exécutant la commande ci-dessous :
>
> ```azurecli-interactive
> az aks install-cli
> ```
> Vous devrez peut-être l’exécuter avec des privilèges `sudo`. 

Vous avez attribué le rôle et les autorisations souhaités. Vous pouvez maintenant lancer l’appel de l’API Kubernetes, par exemple, à partir de `kubectl`.

Avant cela, obtenez les données kubeconfig du cluster avec la commande ci-dessous :

```azurecli-interactive
az aks get-credentials -g MyResourceGroup -n MyManagedCluster
```

> [!IMPORTANT]
> Pour effectuer l’étape ci-dessus, vous avez besoin du rôle intégré [Azure Kubernetes Service Cluster User](../role-based-access-control/built-in-roles.md#azure-kubernetes-service-cluster-user-role) (Utilisateur de cluster Azure Kubernetes Service).

À présent, vous pouvez utiliser kubectl pour lister les nœuds du cluster, par exemple. À la première exécution de l’outil, vous devrez vous connecter, puis les commandes suivantes utiliseront leur jeton d’accès respectif.

```azurecli-interactive
kubectl get nodes
To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code AAAAAAAAA to authenticate.

NAME                                STATUS   ROLES   AGE    VERSION
aks-nodepool1-93451573-vmss000000   Ready    agent   3h6m   v1.15.11
aks-nodepool1-93451573-vmss000001   Ready    agent   3h6m   v1.15.11
aks-nodepool1-93451573-vmss000002   Ready    agent   3h6m   v1.15.11
```


## <a name="use-azure-rbac-for-kubernetes-authorization-with-kubelogin"></a>Utiliser Azure RBAC pour l’autorisation Kubernetes avec `kubelogin`

Pour permettre d’autres scénarios comme des connexions non interactives, l’utilisation d’anciennes versions de `kubectl` ou l’activation de l’authentification unique sur plusieurs clusters sans avoir besoin de se connecter au nouveau cluster (si votre jeton est toujours valide), AKS a créé un plug-in exec appelé [`kubelogin`](https://github.com/Azure/kubelogin).

Vous pouvez utiliser ce plug-in en exécutant :

```bash
export KUBECONFIG=/path/to/kubeconfig
kubelogin convert-kubeconfig
``` 

La première fois, vous devrez vous connecter de manière interactive comme avec le kubectl standard, mais vous n’aurez plus à le faire par la suite, même pour les nouveaux clusters Azure AD (tant que votre jeton restera valide).

```bash
kubectl get nodes
To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code AAAAAAAAA to authenticate.

NAME                                STATUS   ROLES   AGE    VERSION
aks-nodepool1-93451573-vmss000000   Ready    agent   3h6m   v1.15.11
aks-nodepool1-93451573-vmss000001   Ready    agent   3h6m   v1.15.11
aks-nodepool1-93451573-vmss000002   Ready    agent   3h6m   v1.15.11
```


## <a name="clean-up"></a>Nettoyer

### <a name="clean-role-assignment"></a>Nettoyer les attributions de rôles

```azurecli-interactive
az role assignment list --scope $AKS_ID --query [].id -o tsv
```
Copiez l’ID ou les ID de toutes les attributions que vous avez effectuées, puis exécutez cette commande.

```azurecli-interactive
az role assignment delete --ids <LIST OF ASSIGNMENT IDS>
```

### <a name="clean-up-role-definition"></a>Nettoyer la définition de rôle

```azurecli-interactive
az role definition delete -n "AKS Deployment Viewer"
```

### <a name="delete-cluster-and-resource-group"></a>Nettoyer le cluster et le groupe de ressources

```azurecli-interactive
az group delete -n MyResourceGroup
```

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur l’authentification AKS, l’autorisation, le RBAC Kubernetes et le RBAC Azure, voir [ici](concepts-identity.md).
- Découvrez [ici](../role-based-access-control/overview.md) plus d’informations sur Azure RBAC.
- Découvrez [ici](../role-based-access-control/resource-provider-operations.md#microsoftcontainerservice) plus d’informations sur toutes les actions possibles pour définir avec précision des rôles Azure RBAC personnalisés pour l’autorisation Kubernetes.


<!-- LINKS - Internal -->
[aks-support-policies]: support-policies.md
[aks-faq]: faq.md
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[az-feature-list]: /cli/azure/feature#az-feature-list
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-aks-install-cli]: /cli/azure/aks?view=azure-cli-latest#az-aks-install-cli&preserve-view=true
