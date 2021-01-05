---
title: Utiliser Azure AD et le RBAC Kubernetes pour les clusters
titleSuffix: Azure Kubernetes Service
description: Découvrez comment utiliser l’appartenance de groupe Azure Active Directory pour restreindre l’accès aux ressources de cluster à l’aide du contrôle d’accès en fonction du rôle Kubernetes (RBAC Kubernetes) dans Azure Kubernetes Service (ACS)
services: container-service
ms.topic: article
ms.date: 07/21/2020
ms.openlocfilehash: 89bf785a082ff479dcd824a773123fcd061dc22d
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96576109"
---
# <a name="control-access-to-cluster-resources-using-kubernetes-role-based-access-control-and-azure-active-directory-identities-in-azure-kubernetes-service"></a>Contrôler l’accès aux ressources de cluster à l’aide du contrôle d’accès en fonction du rôle Kubernetes et des identités Azure Active Directory dans Azure Kubernetes Service

Azure Kubernetes Service (AKS) peut être configuré pour utiliser Azure Active Directory (AD) pour l’authentification utilisateur. Dans cette configuration, vous vous connectez à un cluster AKS en utilisant un jeton d’authentification Azure AD. Vous pouvez également configurer le contrôle d’accès en fonction du rôle Kubernetes (RBAC Kubernetes) pour limiter l’accès aux ressources de cluster en fonction de l’identité ou de l’appartenance à un groupe d’un utilisateur.

Cet article montre comment utiliser l’appartenance de groupe Azure AD pour contrôler l’accès aux espaces de noms et ressources de cluster à l’aide du contrôle d’accès en fonction du rôle (RBAC) de Kubernetes dans un cluster AKS. Des exemples de groupes et d’utilisateurs sont créés dans Azure AD, puis des Roles et RoleBindings sont créés dans le cluster AKS pour accorder les autorisations appropriées pour créer et afficher des ressources.

## <a name="before-you-begin"></a>Avant de commencer

Cet article part du principe que vous disposez d’un cluster AKS existant activé avec l’intégration d’Azure AD. Si vous avez besoin d’un cluster AKS, consultez [Intégrer Azure Active Directory à AKS][azure-ad-aks-cli].

L’interface de ligne de commande Azure (Azure CLI) version 2.0.61 ou une version ultérieure doit avoir été installée et configurée. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI][install-azure-cli].

## <a name="create-demo-groups-in-azure-ad"></a>Créer des groupes de démonstration dans Azure AD

Dans le cadre de cet article, nous allons créer deux rôles d’utilisateur utilisables pour montrer comment le contrôle d’accès en fonction du rôle (RBAC) de Kubernetes et Azure AD contrôlent l’accès aux ressources de cluster. Les deux exemples de rôles suivants sont utilisés :

* **Développeur d’application**
    * Utilisateur nommé *aksdev* faisant partie du groupe *appdev*.
* **Ingénieur de fiabilité de site**
    * Utilisateur nommé *akssre* faisant partie du groupe *opssre*.

Dans des environnements de production, vous pouvez utiliser des utilisateurs et groupes existants au sein d’un locataire Azure AD.

Commencez par obtenir l’ID de ressource de votre cluster AKS à l’aide de la commande [az aks show][az-aks-show]. Affectez l’ID de ressource à une variable nommée *AKS_ID* afin de pouvoir y faire référence dans d’autres commandes.

```azurecli-interactive
AKS_ID=$(az aks show \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --query id -o tsv)
```

Créez le premier exemple de groupe dans Azure AD pour les développeurs d’applications à l’aide de la commande [az ad group create][az-ad-group-create]. L’exemple suivant crée un groupe nommé *appdev* :

```azurecli-interactive
APPDEV_ID=$(az ad group create --display-name appdev --mail-nickname appdev --query objectId -o tsv)
```

Créez à présent une attribution de rôle Azure pour le groupe *appdev* à l’aide de la commande [az role assignment create][az-role-assignment-create]. Cette attribution permet à tout membre du groupe d’utiliser `kubectl` pour interagir avec un cluster AKS en lui octroyant le *Rôle utilisateur de cluster du service Azure Kubernetes*.

```azurecli-interactive
az role assignment create \
  --assignee $APPDEV_ID \
  --role "Azure Kubernetes Service Cluster User Role" \
  --scope $AKS_ID
```

> [!TIP]
> Si vous recevez une erreur telle que `Principal 35bfec9328bd4d8d9b54dea6dac57b82 does not exist in the directory a5443dcd-cd0e-494d-a387-3039b419f0d5.`, attendez quelques secondes que l’ID d’objet de groupe Azure AD se propage dans l’annuaire, puis réexécutez la commande `az role assignment create`.

Créez un deuxième exemple de groupe pour les ingénieurs de fiabilité de site, nommé *opssre* :

```azurecli-interactive
OPSSRE_ID=$(az ad group create --display-name opssre --mail-nickname opssre --query objectId -o tsv)
```

Là encore, créez une attribution de rôle Azure pour accorder aux membres du groupe le *Rôle utilisateur de cluster d’Azure Kubernetes Service* :

```azurecli-interactive
az role assignment create \
  --assignee $OPSSRE_ID \
  --role "Azure Kubernetes Service Cluster User Role" \
  --scope $AKS_ID
```

## <a name="create-demo-users-in-azure-ad"></a>Créer des utilisateurs de démonstration dans Azure AD

Avec les deux exemples de groupes créés dans Azure AD pour nos développeurs d’applications et ingénieurs de fiabilité de site, créons à présent deux exemples d’utilisateurs. Pour tester l’intégration du RBAC Kubernetes à la fin de l’article, vous allez vous connecter au cluster AKS avec ces comptes.

Créez le premier compte d’utilisateur dans Azure AD à l’aide de la commande [az ad user create][az-ad-user-create].

L’exemple suivant crée un utilisateur dont le nom d’affichage est *AKS Dev* et le nom d’utilisateur principal (UPN) `aksdev@contoso.com`. Mettez à jour l’UPN afin d’inclure un domaine vérifié pour votre locataire Azure AD (remplacez *contoso.com* par votre propre domaine) et fournissez votre propres informations d’identification `--password` sécurisées :

```azurecli-interactive
AKSDEV_ID=$(az ad user create \
  --display-name "AKS Dev" \
  --user-principal-name aksdev@contoso.com \
  --password P@ssw0rd1 \
  --query objectId -o tsv)
```

Ajoutez maintenant l’utilisateur au groupe *appdev* créé dans la section précédente à l’aide de la commande [az ad group member add][az-ad-group-member-add] :

```azurecli-interactive
az ad group member add --group appdev --member-id $AKSDEV_ID
```

Créez un deuxième compte d’utilisateur. L’exemple suivant crée un utilisateur dont le nom d’affichage est *AKS SRE* et le nom d’utilisateur principal (UPN) `akssre@contoso.com`. Là encore, mettez à jour l’UPN afin d’inclure un domaine vérifié pour votre locataire Azure AD (remplacez *contoso.com* par votre propre domaine) et fournissez votre propres informations d’identification `--password` sécurisées :

```azurecli-interactive
# Create a user for the SRE role
AKSSRE_ID=$(az ad user create \
  --display-name "AKS SRE" \
  --user-principal-name akssre@contoso.com \
  --password P@ssw0rd1 \
  --query objectId -o tsv)

# Add the user to the opssre Azure AD group
az ad group member add --group opssre --member-id $AKSSRE_ID
```

## <a name="create-the-aks-cluster-resources-for-app-devs"></a>Créer les ressources de cluster AKS pour les développeurs d’applications

Les groupes et utilisateurs Azure AD sont maintenant créés. Des attributions de rôles Azure ont été créées pour permettre aux membres du groupe de se connecter à un cluster AKS en tant qu’utilisateurs standards. Maintenant, configurons le cluster AKS pour autoriser ces différents groupes à accéder à des ressources spécifiques.

Tout d’abord, obtenez les informations d’identification d’administrateur du cluster à l’aide de la commande [az aks get-credentials][az-aks-get-credentials]. Dans l’une des sections suivantes, vous obtenez les informations d’identification de cluster *utilisateur* ordinaires pour voir le flux d’authentification Azure AD en action.

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster --admin
```

Créez un espace de noms dans le cluster AKS à l’aide de la commande [kubectl create namespace][kubectl-create]. L’exemple suivant crée un espace de noms nommé *dev* :

```console
kubectl create namespace dev
```

Dans Kubernetes, les *Roles* définissent les autorisations à accorder, et les *RoleBindings* les appliquent aux utilisateurs ou groupes souhaités. Ces affectations peuvent s’appliquer à un espace de noms donné ou à l’ensemble du cluster. Pour plus d’informations, consultez [Utilisation de l’autorisation Kubernetes RBAC][rbac-authorization].

Commencez par créer un rôle pour l’espace de noms *dev*. Ce rôle accorde des autorisations complètes pour l’espace de noms. Dans des environnements de production, vous pouvez spécifier des autorisations plus granulaires pour différents utilisateurs ou groupes.

Créez un fichier nommé `role-dev-namespace.yaml` et collez le manifeste YAML suivant :

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: dev-user-full-access
  namespace: dev
rules:
- apiGroups: ["", "extensions", "apps"]
  resources: ["*"]
  verbs: ["*"]
- apiGroups: ["batch"]
  resources:
  - jobs
  - cronjobs
  verbs: ["*"]
```

Créez le Role à l’aide de la commande [kubectl apply][kubectl-apply] et spécifiez le nom de fichier de votre manifeste YAML :

```console
kubectl apply -f role-dev-namespace.yaml
```

Ensuite, obtenez l’ID de ressource du groupe *appdev* à l’aide de la commande [az ad group show][az-ad-group-show]. Ce groupe est défini en tant qu’objet d’un RoleBinding à l’étape suivante.

```azurecli-interactive
az ad group show --group appdev --query objectId -o tsv
```

Maintenant, créez un RoleBinding pour le groupe *appdev* afin d’utiliser le Role créé précédemment pour l’accès à l’espace de noms. Créez un fichier nommé `rolebinding-dev-namespace.yaml` et collez le manifeste YAML suivant. Sur la dernière ligne, remplacez *groupObjectId* par l’ID d’objet de groupe retourné par la commande précédente :

```yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: dev-user-access
  namespace: dev
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: dev-user-full-access
subjects:
- kind: Group
  namespace: dev
  name: groupObjectId
```

Créez le RoleBinding à l’aide de la commande [kubectl apply][kubectl-apply] et spécifiez le nom de fichier de votre manifeste YAML :

```console
kubectl apply -f rolebinding-dev-namespace.yaml
```

## <a name="create-the-aks-cluster-resources-for-sres"></a>Créer les ressources de cluster AKS pour les ingénieurs de fiabilité de site

Répétez les étapes précédentes pour créer un espace de noms, Role et RoleBinding pour les ingénieurs de fiabilité de site.

Commencez par créer un espace de noms *sre* à l’aide de la [kubectl create namespace][kubectl-create] :

```console
kubectl create namespace sre
```

Créez un fichier nommé `role-sre-namespace.yaml` et collez le manifeste YAML suivant :

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: sre-user-full-access
  namespace: sre
rules:
- apiGroups: ["", "extensions", "apps"]
  resources: ["*"]
  verbs: ["*"]
- apiGroups: ["batch"]
  resources:
  - jobs
  - cronjobs
  verbs: ["*"]
```

Créez le Role à l’aide de la commande [kubectl apply][kubectl-apply] et spécifiez le nom de fichier de votre manifeste YAML :

```console
kubectl apply -f role-sre-namespace.yaml
```

Obtenez l’ID de ressource du groupe *opssre* à l’aide de la commande [az ad group show][az-ad-group-show] :

```azurecli-interactive
az ad group show --group opssre --query objectId -o tsv
```

Créez un RoleBinding pour le groupe *opssre* afin d’utiliser le Role créé précédemment pour l’accès à l’espace de noms. Créez un fichier nommé `rolebinding-sre-namespace.yaml` et collez le manifeste YAML suivant. Sur la dernière ligne, remplacez *groupObjectId* par l’ID d’objet de groupe retourné par la commande précédente :

```yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: sre-user-access
  namespace: sre
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: sre-user-full-access
subjects:
- kind: Group
  namespace: sre
  name: groupObjectId
```

Créez le RoleBinding à l’aide de la commande [kubectl apply][kubectl-apply] et spécifiez le nom de fichier de votre manifeste YAML :

```console
kubectl apply -f rolebinding-sre-namespace.yaml
```

## <a name="interact-with-cluster-resources-using-azure-ad-identities"></a>Interagir avec les ressources de cluster à l’aide d’identités Azure AD

Maintenant, testons le fonctionnement des autorisations attendu lorsque vous créez et gérez des ressources dans un cluster AKS. Dans ces exemples, vous allez planifier et afficher des pods dans l’espace de noms attribué de l’utilisateur. Ensuite, vous allez tenter de planifier et d’afficher les pods en dehors de l’espace de noms attribué.

Commencez par réinitialiser le contexte *kubeconfig* à l’aide de la commande [az aks get-credentials][az-aks-get-credentials]. Dans une section précédente, vous avez défini le contexte en utilisant les informations d’identification d’administrateur de cluster. L’utilisateur administrateur ignore les invites de connexion d’Azure AD. Sans le paramètre `--admin`, le contexte utilisateur appliqué requiert que toutes les demandes soient authentifiées à l’aide d’Azure AD.

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster --overwrite-existing
```

Planifiez un pod NGINX de base à l’aide de la commande [kubectl run][kubectl-run] dans l’espace de noms *dev* :

```console
kubectl run nginx-dev --image=nginx --namespace dev
```

En tant qu’invite de connexion, entrez les informations d’identification de votre propre compte `appdev@contoso.com` créé au début de l’article. Une fois que vous êtes correctement connecté, le jeton de compte est mis en cache pour des commandes `kubectl` futures. Le pod NGINX est correctement planifié, comme le montre l’exemple de sortie suivant :

```console
$ kubectl run nginx-dev --image=nginx --namespace dev

To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code B24ZD6FP8 to authenticate.

pod/nginx-dev created
```

Utilisez à présent la commande [kubectl get pods][kubectl-get] pour afficher tous les pods dans l’espace de noms *dev*.

```console
kubectl get pods --namespace dev
```

Comme le montre l’exemple de sortie suivant, le pod NGINX s’*exécute* correctement :

```console
$ kubectl get pods --namespace dev

NAME        READY   STATUS    RESTARTS   AGE
nginx-dev   1/1     Running   0          4m
```

### <a name="create-and-view-cluster-resources-outside-of-the-assigned-namespace"></a>Créer et afficher des ressources de cluster en dehors de l’espace de noms attribué

Essayez maintenant d’afficher des pods en dehors de l’espace de noms *dev*. Utilisez de nouveau la commande [kubectl get pods][kubectl-get], cette fois pour voir `--all-namespaces` comme suit :

```console
kubectl get pods --all-namespaces
```

L’appartenance de groupe de l’utilisateur n’a pas de Role Kubernetes autorisant cette action, comme le montre l’exemple de sortie suivant :

```console
$ kubectl get pods --all-namespaces

Error from server (Forbidden): pods is forbidden: User "aksdev@contoso.com" cannot list resource "pods" in API group "" at the cluster scope
```

De la même façon, essayez de planifier un pod dans un espace de noms différent, tel que *sre*. L’appartenance de groupe de l’utilisateur ne s’aligne pas avec un Role et des RoleBinding Kubernetes pour accorder ces autorisations, comme le montre l’exemple de sortie suivant :

```console
$ kubectl run nginx-dev --image=nginx --namespace sre

Error from server (Forbidden): pods is forbidden: User "aksdev@contoso.com" cannot create resource "pods" in API group "" in the namespace "sre"
```

### <a name="test-the-sre-access-to-the-aks-cluster-resources"></a>Tester l’accès d’ingénieur de fiabilité de site aux ressources de cluster AKS

Pour vérifier que notre appartenance de groupe Azure AD et le contrôle d’accès en fonction du rôle (RBAC) Kubernetes fonctionnent correctement entre les différents utilisateurs et groupes, essayez les commandes précédentes lorsque vous êtes connecté en tant qu’utilisateur *opssre*.

Réinitialisez le contexte *kubeconfig* à l’aide de la commande [az aks get-credentials][az-aks-get-credentials] qui efface le jeton d’authentification précédemment mis en cache pour l’utilisateur *aksdev* :

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster --overwrite-existing
```

Essayez de planifier et d’afficher les pods dans l’espace de noms *sre* attribué. Lorsque vous y êtes invité, connectez-vous avec vos propres informations d’identification `opssre@contoso.com` créées au début de l’article :

```console
kubectl run nginx-sre --image=nginx --namespace sre
kubectl get pods --namespace sre
```

Comme le montre l’exemple de sortie suivant, vous pouvez correctement créer et afficher les pods :

```console
$ kubectl run nginx-sre --image=nginx --namespace sre

To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code BM4RHP3FD to authenticate.

pod/nginx-sre created

$ kubectl get pods --namespace sre

NAME        READY   STATUS    RESTARTS   AGE
nginx-sre   1/1     Running   0
```

À présent, essayez d’afficher ou de planifier des pods en dehors de l’espace de noms SRE attribué :

```console
kubectl get pods --all-namespaces
kubectl run nginx-sre --image=nginx --namespace dev
```

Ces commandes `kubectl` échouent, comme le montre l’exemple de sortie suivant. L’appartenance de groupe de l’utilisateur et le Role et les RoleBindings Kubernetes n’accordent pas d’autorisations pour créer ou gérer des ressources dans d’autres espaces de noms :

```console
$ kubectl get pods --all-namespaces
Error from server (Forbidden): pods is forbidden: User "akssre@contoso.com" cannot list pods at the cluster scope

$ kubectl run nginx-sre --image=nginx --namespace dev
Error from server (Forbidden): pods is forbidden: User "akssre@contoso.com" cannot create pods in the namespace "dev"
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Dans cet article, vous avez créé des ressources dans le cluster AKS, et des utilisateurs et groupes dans Azure AD. Pour nettoyer toutes ces ressources, exécutez les commandes suivantes :

```azurecli-interactive
# Get the admin kubeconfig context to delete the necessary cluster resources
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster --admin

# Delete the dev and sre namespaces. This also deletes the pods, Roles, and RoleBindings
kubectl delete namespace dev
kubectl delete namespace sre

# Delete the Azure AD user accounts for aksdev and akssre
az ad user delete --upn-or-object-id $AKSDEV_ID
az ad user delete --upn-or-object-id $AKSSRE_ID

# Delete the Azure AD groups for appdev and opssre. This also deletes the Azure role assignments.
az ad group delete --group appdev
az ad group delete --group opssre
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la sécurisation des clusters Kubernetes, consultez [Options d’accès et d’identité pour Azure Kubernetes Service (AKS)][rbac-authorization].

Pour découvrir les meilleures pratiques de contrôle des identités et des ressources, consultez [Meilleurs pratiques relatives à l’authentification et à l’autorisation dans Azure Kubernetes Service (AKS)][operator-best-practices-identity].

<!-- LINKS - external -->
[kubectl-create]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#create
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-run]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#run

<!-- LINKS - internal -->
[az-aks-get-credentials]: /cli/azure/aks?view=azure-cli-latest#az-aks-get-credentials
[install-azure-cli]: /cli/azure/install-azure-cli
[azure-ad-aks-cli]: azure-ad-integration-cli.md
[az-aks-show]: /cli/azure/aks#az-aks-show
[az-ad-group-create]: /cli/azure/ad/group#az-ad-group-create
[az-role-assignment-create]: /cli/azure/role/assignment#az-role-assignment-create
[az-ad-user-create]: /cli/azure/ad/user#az-ad-user-create
[az-ad-group-member-add]: /cli/azure/ad/group/member#az-ad-group-member-add
[az-ad-group-show]: /cli/azure/ad/group#az-ad-group-show
[rbac-authorization]: concepts-identity.md#kubernetes-role-based-access-control-kubernetes-rbac
[operator-best-practices-identity]: operator-best-practices-identity.md
