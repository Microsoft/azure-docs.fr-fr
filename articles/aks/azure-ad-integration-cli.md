---
title: Intégrer Azure Active Directory à Azure Kubernetes Service (hérité)
description: Découvrez comment utiliser Azure CLI pour créer un cluster Azure Kubernetes Service (AKS) compatible avec Azure Active Directory (hérité)
services: container-service
author: TomGeske
ms.topic: article
ms.date: 07/20/2020
ms.author: thomasge
ms.openlocfilehash: 4aa63493bb14db69821ac04db1d2c5a846de7dbe
ms.sourcegitcommit: c157b830430f9937a7fa7a3a6666dcb66caa338b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94682466"
---
# <a name="integrate-azure-active-directory-with-azure-kubernetes-service-using-the-azure-cli-legacy"></a>Intégrer Azure Active Directory à Azure Kubernetes Service à l’aide d’Azure CLI (hérité)

Azure Kubernetes Service (AKS) peut être configuré pour utiliser Azure Active Directory (AD) pour l’authentification utilisateur. Dans cette configuration, vous vous connectez à un cluster AKS en utilisant un jeton d’authentification Azure AD. Les opérateurs de cluster peuvent également configurer le contrôle d’accès en fonction du rôle Kubernetes (Kubernetes RBAC) basé sur une identité utilisateur ou l’appartenance à un groupe de répertoires.

Cet article explique comment créer les composants Azure AD requis, puis déployer un cluster compatible Azure AD et créer un rôle Kubernetes de base dans le cluster AKS.

Pour l’exemple de script complet utilisé dans cet article, voir [Exemples Azure CLI – Intégration d’AKS avec Azure AD][complete-script].

> [!Important]
> AKS propose une nouvelle expérience [Azure AD gérée par AKS][managed-aad] améliorée qui ne nécessite pas que vous gériez le serveur ou l’application cliente. Si vous souhaitez effectuer une migration, suivez les instructions [ici][managed-aad-migrate].

## <a name="the-following-limitations-apply"></a>Les limites suivantes s'appliquent :

- Azure AD peut uniquement être activé sur un cluster sur lequel Kubernetes RBAC est activé.
- L’intégration héritée d’Azure AD ne peut être activée que lors de la création du cluster.

## <a name="before-you-begin"></a>Avant de commencer

L’interface de ligne de commande Azure (Azure CLI) version 2.0.61 ou une version ultérieure doit avoir été installée et configurée. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI][install-azure-cli].

Accédez à [https://shell.azure.com](https://shell.azure.com) pour ouvrir Cloud Shell dans votre navigateur.

Par souci de cohérence et pour faciliter l’exécution des commandes décrites dans cet article, créez une variable pour votre nom de cluster AKS souhaité. L’exemple suivant utilise le nom *myakscluster* :

```console
aksname="myakscluster"
```

## <a name="azure-ad-authentication-overview"></a>Vue d’ensemble de l’authentification Azure AD

L’authentification Azure AD est fournie aux clusters AKS à l’aide d’OpenID Connect. OpenID Connect est une couche d’identité basée sur le protocole OAuth 2.0. Pour plus d’informations sur OpenID Connect, voir la [documentation sur Open ID Connect][open-id-connect].

Depuis le cluster Kubernetes, l’authentification par jeton de Webhook est utilisée pour vérifier les jetons d’authentification. L’authentification par jeton de Webhook est configurée et gérée en tant que partie du cluster AKS. Pour plus d’informations sur l’authentification par jeton Webhook, voir la [documentation sur l’authentification de webhook][kubernetes-webhook].

> [!NOTE]
> Lorsque vous configurez Azure AD pour l’authentification AKS, deux applications Azure AD sont configurées. Cette opération doit être effectuée par un administrateur de locataire Azure.

## <a name="create-azure-ad-server-component"></a>Créer un composant serveur Azure AD

Pour intégrer avec AKS, vous créez et utilisez une application Azure AD qui agit en tant que point de terminaison pour les demandes d’identité. La première application Azure AD dont vous avez besoin obtient l’appartenance au groupe Azure AD pour un utilisateur.

Créez le composant d’application serveur avec la commande [az ad app create][az-ad-app-create], puis mettez à jour les revendications d’appartenance au groupe avec la commande [az ad app update][az-ad-app-update]. L’exemple suivant utilise la variable *aksname* définie dans la section [Avant de commencer](#before-you-begin) et crée une variable

```azurecli-interactive
# Create the Azure AD application
serverApplicationId=$(az ad app create \
    --display-name "${aksname}Server" \
    --identifier-uris "https://${aksname}Server" \
    --query appId -o tsv)

# Update the application group membership claims
az ad app update --id $serverApplicationId --set groupMembershipClaims=All
```

Maintenant, créez un principal de service pour l’application serveur avec la commande [az ad sp create][az-ad-sp-create]. Ce principal de service est utilisé pour s’authentifier dans la plateforme Azure. Ensuite, obtenez le secret du principal de service en utilisant la commande [az ad sp credential reset][az-ad-sp-credential-reset] et affectez-le à la variable nommée *serverApplicationSecret* pour l’utiliser dans l’une des étapes suivantes :

```azurecli-interactive
# Create a service principal for the Azure AD application
az ad sp create --id $serverApplicationId

# Get the service principal secret
serverApplicationSecret=$(az ad sp credential reset \
    --name $serverApplicationId \
    --credential-description "AKSPassword" \
    --query password -o tsv)
```

Le principal de service Azure AD a besoin d’autorisations pour effectuer les actions suivantes :

* Lire les données du répertoire
* Connecter et lire le profil utilisateur

Attribuez ces autorisations à l’aide de la commande [az ad app permission add][az-ad-app-permission-add] :

```azurecli-interactive
az ad app permission add \
    --id $serverApplicationId \
    --api 00000003-0000-0000-c000-000000000000 \
    --api-permissions e1fe6dd8-ba31-4d61-89e7-88639da4683d=Scope 06da0dbc-49e2-44d2-8312-53f166ab848a=Scope 7ab1d382-f21e-4acd-a863-ba3e13f7da61=Role
```

Enfin, accordez les autorisations attribuées dans l’étape précédente pour l’application serveur à l’aide de la commande [az ad app permission grant][az-ad-app-permission-grant]. Cette étape échoue si le compte actuel n’est pas un administrateur de locataire. Vous devez également ajouter des autorisations pour que l’application Azure AD demande des informations, qui sinon pourraient nécessiter le consentement de l’administrateur avec [az ad app permission admin-consent][az-ad-app-permission-admin-consent] :

```azurecli-interactive
az ad app permission grant --id $serverApplicationId --api 00000003-0000-0000-c000-000000000000
az ad app permission admin-consent --id  $serverApplicationId
```

## <a name="create-azure-ad-client-component"></a>Créer un composant de client Azure AD

La deuxième application Azure AD est utilisée quand un utilisateur se connecte au cluster AKS avec l’interface de ligne de commande Kubernetes (`kubectl`). Cette application cliente prend la demande d’authentification de l’utilisateur et vérifie les informations d’identification et les autorisations. Créez l’application Azure AD pour le composant de client à l’aide de la commande [az ad app create][az-ad-app-create] :

```azurecli-interactive
clientApplicationId=$(az ad app create \
    --display-name "${aksname}Client" \
    --native-app \
    --reply-urls "https://${aksname}Client" \
    --query appId -o tsv)
```

Créez un principal de service pour l’application cliente à l’aide de la commande [az ad sp create][az-ad-sp-create] :

```azurecli-interactive
az ad sp create --id $clientApplicationId
```

Obtenez l’ID oAuth2 pour l’application serveur afin de permettre le flux d’authentification entre les deux composants d’application à l’aide de la commande [az ad app show][az-ad-app-show]. Cet ID oAuth2 sera utilisé dans l’étape suivante.

```azurecli-interactive
oAuthPermissionId=$(az ad app show --id $serverApplicationId --query "oauth2Permissions[0].id" -o tsv)
```

Ajoutez les autorisations pour les composants d’application cliente et d’application serveur nécessaires pour utiliser le flux de communication oAuth2 à l’aide de la commande [az ad app permission add][az-ad-app-permission-add]. Ensuite, accordez des autorisations pour que l’application cliente communique avec l’application serveur en utilisant la commande [az ad app permission grant][az-ad-app-permission-grant] :

```azurecli-interactive
az ad app permission add --id $clientApplicationId --api $serverApplicationId --api-permissions ${oAuthPermissionId}=Scope
az ad app permission grant --id $clientApplicationId --api $serverApplicationId
```

## <a name="deploy-the-cluster"></a>Déployer le cluster

Les deux applications Azure AD étant créées, créez à présent le cluster AKS. Commencez par créer un groupe de ressources à l’aide de la commande [az group create][az-group-create]. L’exemple suivant crée le groupe de ressources dans la région *EastUS* :

Créez un groupe de ressources pour le cluster :

```azurecli-interactive
az group create --name myResourceGroup --location EastUS
```

Obtenez l’ID de locataire de votre abonnement Azure à l’aide de la commande [az account show][az-account-show]. Ensuite, créez le cluster AKS avec la commande [az aks create][az-aks-create]. La commande pour créer le cluster AKS fournit les ID d’application serveur et cliente, le secret du principal de service de l’application serveur et votre ID de locataire :

```azurecli-interactive
tenantId=$(az account show --query tenantId -o tsv)

az aks create \
    --resource-group myResourceGroup \
    --name $aksname \
    --node-count 1 \
    --generate-ssh-keys \
    --aad-server-app-id $serverApplicationId \
    --aad-server-app-secret $serverApplicationSecret \
    --aad-client-app-id $clientApplicationId \
    --aad-tenant-id $tenantId
```

Enfin, obtenez les informations d’identification d’administrateur du cluster à l’aide de la commande [az aks get-credentials][az-aks-get-credentials]. Dans l’une des étapes suivantes, vous allez obtenir les informations d’identification de cluster *user* ordinaires afin de voir le flux d’authentification Azure AD en action.

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name $aksname --admin
```

## <a name="create-kubernetes-rbac-binding"></a>Créer une liaison Kubernetes RBAC

Avant de pouvoir utiliser un compte Azure Active Directory avec le cluster AKS, une liaison de rôle ou une liaison de rôle de cluster doit être créée. Les *rôles* définissent les permissions à accorder, et les *liaisons* les appliquent aux utilisateurs souhaités. Ces affectations peuvent s’appliquer à un espace de noms donné ou à l’ensemble du cluster. Pour plus d’informations, consultez [Utilisation de l’autorisation Kubernetes RBAC][rbac-authorization].

Obtenez le nom d’utilisateur principal (UPN) de l’utilisateur actuellement connecté à l’aide de la commande [az ad signed-in-user show][az-ad-signed-in-user-show]. Ce compte d’utilisateur est activé pour l’intégration d’Azure AD à l’étape suivante.

```azurecli-interactive
az ad signed-in-user show --query userPrincipalName -o tsv
```

> [!IMPORTANT]
> Si l’utilisateur pour lequel vous accordez la liaison Kubernetes RBAC figure dans le même locataire Azure AD, attribuez les autorisations en fonction d’*userPrincipalName*. Si l’utilisateur se trouve dans un autre locataire Azure AD, recherchez et utilisez plutôt la propriété *objectId*.

Créez un manifeste YAML nommé `basic-azure-ad-binding.yaml`, et collez le contenu suivant. Sur la dernière ligne, remplacez *userPrincipalName_or_objectId* par l’UPN ou l’ID d’objet retournés par la commande précédente :

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: contoso-cluster-admins
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: userPrincipalName_or_objectId
```

Créez la ClusterRoleBinding à l’aide de la commande [kubectl apply][kubectl-apply], et spécifiez le nom de fichier de votre manifeste YAML :

```console
kubectl apply -f basic-azure-ad-binding.yaml
```

## <a name="access-cluster-with-azure-ad"></a>Accéder au cluster avec Azure AD

Testons à présent l’intégration de l’authentification Azure AD pour le cluster AKS. Définissez le contexte de configuration `kubectl` pour utiliser les informations d’identification d’un utilisateur standard. Ce contexte réachemine toutes les demandes d’authentification via Azure AD.

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name $aksname --overwrite-existing
```

Utilisez à présent la commande [kubectl get pods][kubectl-get] pour afficher les pods dans tous les espaces de noms :

```console
kubectl get pods --all-namespaces
```

Vous recevez une invite de connexion vous demandant de vous authentifier en entrant vos informations d’identification Azure AD dans un navigateur web. Une fois que vous êtes authentifié, la commande `kubectl` affiche les pods figurant dans le cluster AKS, comme illustré dans l’exemple de sortie suivant :

```console
kubectl get pods --all-namespaces
```

```output
To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code BYMK7UXVD to authenticate.

NAMESPACE     NAME                                    READY   STATUS    RESTARTS   AGE
kube-system   coredns-754f947b4-2v75r                 1/1     Running   0          23h
kube-system   coredns-754f947b4-tghwh                 1/1     Running   0          23h
kube-system   coredns-autoscaler-6fcdb7d64-4wkvp      1/1     Running   0          23h
kube-system   heapster-5fb7488d97-t5wzk               2/2     Running   0          23h
kube-system   kube-proxy-2nd5m                        1/1     Running   0          23h
kube-system   kube-svc-redirect-swp9r                 2/2     Running   0          23h
kube-system   kubernetes-dashboard-847bb4ddc6-trt7m   1/1     Running   0          23h
kube-system   metrics-server-7b97f9cd9-btxzz          1/1     Running   0          23h
kube-system   tunnelfront-6ff887cffb-xkfmq            1/1     Running   0          23h
```

Le jeton d’authentification reçu pour `kubectl` est mis en cache. Vous n’êtes réinvité à vous connecter que quand le jeton a expiré ou lorsque le fichier de configuration Kubernetes est recréé.

Si vous voyez un message d’erreur d’autorisation s’afficher après que vous vous êtes connecté avec succès à l’aide d’un navigateur web, comme dans l’exemple de sortie ci-après, vérifiez les problèmes possibles suivants :

```output
error: You must be logged in to the server (Unauthorized)
```

* Vous avez défini l’ID d’objet ou l’UPN approprié, selon que le compte d’utilisateur se trouve dans le même locataire Azure AD ou non.
* L'utilisateur n’est pas membre de plus de 200 groupes.
* Le secret défini dans l’inscription d’application pour le serveur correspond à la valeur configurée à l’aide de `--aad-server-app-secret`

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir le script complet contenant les commandes présentées dans cet article, voir le [script d’intégration Azure AD dans le référentiel d’exemples AKS][complete-script].

Pour utiliser des utilisateurs et des groupes Azure AD pour le contrôle d’accès à des ressources de cluster, consultez [Contrôler l’accès aux ressources de cluster à l’aide du contrôle d’accès en fonction du rôle Kubernetes et d’identités Azure AD dans AKS][azure-ad-rbac].

Pour plus d’informations sur la sécurisation des clusters Kubernetes, consultez [Options d’accès et d’identité pour Azure Kubernetes Service (AKS)][rbac-authorization].

Pour découvrir les meilleures pratiques de contrôle des identités et des ressources, consultez [Meilleurs pratiques relatives à l’authentification et à l’autorisation dans Azure Kubernetes Service (AKS)][operator-best-practices-identity].

<!-- LINKS - external -->
[kubernetes-webhook]:https://kubernetes.io/docs/reference/access-authn-authz/authentication/#webhook-token-authentication
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[complete-script]: https://github.com/Azure-Samples/azure-cli-samples/tree/master/aks/azure-ad-integration/azure-ad-integration.sh

<!-- LINKS - internal -->
[az-aks-create]: /cli/azure/aks?view=azure-cli-latest#az-aks-create
[az-aks-get-credentials]: /cli/azure/aks?view=azure-cli-latest#az-aks-get-credentials
[az-group-create]: /cli/azure/group#az-group-create
[open-id-connect]: ../active-directory/develop/v2-protocols-oidc.md
[az-ad-user-show]: /cli/azure/ad/user#az-ad-user-show
[az-ad-app-create]: /cli/azure/ad/app#az-ad-app-create
[az-ad-app-update]: /cli/azure/ad/app#az-ad-app-update
[az-ad-sp-create]: /cli/azure/ad/sp#az-ad-sp-create
[az-ad-app-permission-add]: /cli/azure/ad/app/permission#az-ad-app-permission-add
[az-ad-app-permission-grant]: /cli/azure/ad/app/permission#az-ad-app-permission-grant
[az-ad-app-permission-admin-consent]: /cli/azure/ad/app/permission#az-ad-app-permission-admin-consent
[az-ad-app-show]: /cli/azure/ad/app#az-ad-app-show
[az-group-create]: /cli/azure/group#az-group-create
[az-account-show]: /cli/azure/account#az-account-show
[az-ad-signed-in-user-show]: /cli/azure/ad/signed-in-user#az-ad-signed-in-user-show
[install-azure-cli]: /cli/azure/install-azure-cli
[az-ad-sp-credential-reset]: /cli/azure/ad/sp/credential#az-ad-sp-credential-reset
[rbac-authorization]: concepts-identity.md#kubernetes-role-based-access-control-kubernetes-rbac
[operator-best-practices-identity]: operator-best-practices-identity.md
[azure-ad-rbac]: azure-ad-rbac.md
[managed-aad]: managed-aad.md
[managed-aad-migrate]: managed-aad.md#upgrading-to-aks-managed-azure-ad-integration
