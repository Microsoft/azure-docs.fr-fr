---
title: Actions GitHub & Azure Kubernetes Service (préversion)
services: azure-dev-spaces
ms.date: 04/03/2020
ms.topic: conceptual
description: Passez en revue et testez les modifications à partir d’une demande de tirage (pull request) directement dans Azure Kubernetes Service avec des actions GitHub et Azure Dev Spaces.
keywords: Docker, Kubernetes, Azure, AKS, Azure Kubernetes Service, conteneurs, Actions GitHub, Helm, service Mesh, routage du service Mesh, kubectl, k8s
manager: gwallace
ms.custom: devx-track-js, devx-track-azurecli
ms.openlocfilehash: 9bed61861c80f141270e50b644b32ae42fbe8e77
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95995560"
---
# <a name="github-actions--azure-kubernetes-service-preview"></a>Actions GitHub & Azure Kubernetes Service (préversion)

[!INCLUDE [Azure Dev Spaces deprecation](../../../includes/dev-spaces-deprecation.md)]

Azure Dev Spaces fournit un flux de travail en utilisant GitHub Actions, qui vous permet de tester les modifications d’une demande de tirage (pull request) directement dans AKS avant de la fusionner dans la branche principale de votre dépôt. Avoir une application en cours d’exécution pour examiner les modifications apportées à une demande de tirage peut accroître la confiance du développeur et des membres de l’équipe. Cette application en cours d’exécution peut également aider les membres de son équipe, tels que les chefs de produits et les concepteurs, à faire partie du processus de révision lors des premières étapes du développement.

Dans ce guide, vous allez apprendre à :

* Configurer Azure Dev Spaces sur un cluster Kubernetes managé dans Azure.
* Déployer une vaste application avec plusieurs microservices dans un espace de développement.
* Configurez CI/CD avec des actions GitHub.
* Tester un microservice unique dans un espace de développement isolé dans le contexte de l’application complète.

> [!IMPORTANT]
> Actuellement, cette fonctionnalité est uniquement disponible en tant que version préliminaire. Les préversions sont à votre disposition, à condition que vous acceptiez les [conditions d’utilisation supplémentaires](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Certains aspects de cette fonctionnalité sont susceptibles d’être modifiés avant la mise à disposition générale.

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, vous pouvez créer un [compte gratuit](https://azure.microsoft.com/free).
* [Azure CLI][azure-cli-installed].
* [Helm 3 installé][helm-installed].
* Un compte GitHub avec des [actions GitHub activées][github-actions-beta-signup].
* L’exemple d’application [Azure Dev Spaces Bike Sharing](https://github.com/Azure/dev-spaces/tree/master/samples/BikeSharingApp/README.md) s’exécutant sur un cluster AKS.

## <a name="create-an-azure-container-registry"></a>Création d’un Azure Container Registry

Créer un Azure Container Registry (ACR) :

```azurecli
az acr create --resource-group MyResourceGroup --name <acrName> --sku Basic
```

> [!IMPORTANT]
> Le nom de votre ACR doit être unique dans Azure et contenir entre 5 et 50 caractères alphanumériques. Les lettres que vous utilisez doivent être en minuscules.

Enregistrez la valeur *loginServer* à partir de la sortie, car elle est utilisée dans une étape ultérieure.

## <a name="create-a-service-principal-for-authentication"></a>Créer un principal de service pour l’authentification

Utilisez [az ad sp create-for-rbac][az-ad-sp-create-for-rbac] pour créer un principal du service. Par exemple :

```azurecli
az ad sp create-for-rbac --sdk-auth --skip-assignment
```

Enregistrez la sortie JSON, car elle est utilisée à une étape ultérieure.

Utilisez [az aks show][az-aks-show] pour afficher l’*ID* de votre cluster AKS :

```azurecli
az aks show -g MyResourceGroup -n MyAKS  --query id
```

Utilisez [az cr show][az-acr-show] pour afficher l’*ID* de votre ACR :

```azurecli
az acr show --name <acrName> --query id
```

Utilisez [az role assignment create][az-role-assignment-create] pour offrir l’accès *Contributeur* à votre cluster AKS et l’accès *AcrPush* à votre ACR.

```azurecli
az role assignment create --assignee <ClientId> --scope <AKSId> --role Contributor
az role assignment create --assignee <ClientId>  --scope <ACRId> --role AcrPush
```

> [!IMPORTANT]
> Vous devez être le propriétaire de votre cluster AKS et de l’ACR pour permettre à votre principal de service d’accéder à ces ressources.

## <a name="configure-your-github-action"></a>Configurer votre action GitHub

> [!IMPORTANT]
> Les actions GitHub doivent être activées pour votre référentiel. Pour activer les actions GitHub pour votre référentiel, accédez à votre référentiel sur GitHub, cliquez sur l’onglet Actions, puis choisissez d’activer des actions pour ce dépôt.

Accédez à votre référentiel dupliqué, puis cliquez sur *Paramètres*. Cliquez sur *Secrets* dans la barre latérale gauche. Cliquez sur *Ajouter un nouveau secret* pour ajouter chaque nouveau secret ci-dessous :

1. *AZURE_CREDENTIALS* : la totalité de la sortie de la création du principal de service.
1. *RESOURCE_GROUP* : le groupe de ressources de votre cluster AKS, dans cet exemple *MyResourceGroup*.
1. *CLUSTER_NAME* : le nom de votre cluster AKS, dans cet exemple *MyAKS*.
1. *CONTAINER_REGISTRY*: le *loginServer* pour l’ACR.
1. *HOST* : l’hôte pour votre espace de développement, avec la forme *<MASTER_SPACE>.<APP_NAME>.<HOST_SUFFIX>* , dans cet exemple *dev.bikesharingweb.fedcab0987.eus.azds.io*.
1. *IMAGE_PULL_SECRET* : nom de la clé secrète que vous souhaitez utiliser, par exemple *demo-secret*.
1. *MASTER_SPACE* : le nom de votre espace de développement parent, dans cet exemple *dev*.
1. *REGISTRY_USERNAME* : le *clientId* à partir de la sortie JSON de la création du principal de service.
1. *REGISTRY_PASSWORD* : le *clientSecret* à partir de la sortie JSON de la création du principal de service.

> [!NOTE]
> Tous ces secrets sont utilisés par l’action GitHub et sont configurés dans [.gitHub/workflows/bikes.yml][github-action-yaml].

Si vous souhaitez mettre à jour l’espace maître après la fusion de votre demande de tirage (pull request), ajoutez le secret *GATEWAY_HOST* au format *<ESPACE_MAÎTRE>.gateway.<SUFFIXE_HÔTE>* , comme dans l’exemple *dev.gateway.fedcab0987.eus.azds.io*. Une fois que vous avez fusionné vos modifications dans la branche maître de votre duplication (fork), une autre action est exécutée pour regénérer et exécuter l’intégralité de votre application dans l’espace de développement maître. Dans cet exemple, l’espace maître est *dev*. Cette action est configurée dans [.gitHub/workflows/bikesharing.yml][github-action-bikesharing-yaml].

En outre, si vous souhaitez que les modifications apportées à votre demande de tirage s’exécutent dans un espace petit-enfant, mettez à jour les secrets *MASTER_SPACE* et *HOST*. Par exemple, si votre application s’exécute dans *dev* avec un espace enfant *dev/azureuser1*, pour que la demande de tirage s’exécute dans un espace enfant de *dev/azureuser1* :

* Mettez à jour *MASTER_SPACE* sur l’espace enfant souhaité en tant qu’espace parent, dans cet exemple *azureuser1*.
* Mettez à jour *HOST* sur *<GRANDPARENT_SPACE>.<APP_NAME>.<HOST_SUFFIX>* , dans cet exemple *dev.bikesharingweb.fedcab0987.eus.azds.io*.

## <a name="create-a-new-branch-for-code-changes"></a>Créer une branche pour les modifications de code

Accédez à `BikeSharingApp/` et créez une nouvelle branche appelée *bike-images*.

```cmd
cd dev-spaces/samples/BikeSharingApp/
git checkout -b bike-images
```

Modifiez [Bikes/server.js][bikes-server-js] pour supprimer les lignes 232 et 233 :

```javascript
    // Hard code image url *FIX ME*
    theBike.imageUrl = "/static/logo.svg";
```

La section doit maintenant se présenter comme suit :

```javascript
    var theBike = result;
    theBike.id = theBike._id;
    delete theBike._id;
```

Enregistrez le fichier, puis utilisez `git add` et `git commit` pour organiser vos modifications.

```cmd
git add Bikes/server.js 
git commit -m "Removing hard coded imageUrl from /bikes/:id route"
```

## <a name="push-your-changes"></a>Envoyer vos modifications

Utilisez `git push` pour pousser votre nouvelle branche vers votre référentiel dupliqué :

```cmd
git push origin bike-images
```

Une fois l’opération Push terminée, accédez à votre référentiel dupliqué sur GitHub afin de créer une demande de tirage (pull request) avec la branche *master* dans votre référentiel dupliqué comme branche de base, par rapport à la branche *bike-images*.

Une fois la requête de tirage ouverte, accédez à l’onglet *Actions*. Vérifiez qu’une nouvelle action a démarré et qu’elle est en train de créer le service *Bikes*.

## <a name="view-the-child-space-with-your-changes"></a>Afficher l’espace enfant à l’aide de vos modifications

Une fois l’action terminée, vous verrez un commentaire avec une URL vers votre nouvel espace enfant en fonction des modifications apportées à la demande de tirage (pull request).

> [!div class="mx-imgBorder"]
> ![URL d’action GitHub](../media/github-actions/github-action-url.png)

Accédez au service *bikesharingweb* en ouvrant l’URL à partir du commentaire. Sélectionnez *Aurelia Briggs (client)* en tant qu’utilisateur, puis sélectionnez un vélo à louer. Vérifiez que l’image d’espace réservé du vélo n’est plus visible.

Si vous fusionnez vos modifications dans la branche *master* de votre fourche, une autre action est exécutée pour régénérer et exécuter l’intégralité de votre application dans l’espace de développement parent. Dans cet exemple, l’espace parent est *dev*. Cette action est configurée dans [.gitHub/workflows/bikesharing.yml][github-action-bikesharing-yaml].

## <a name="clean-up-your-azure-resources"></a>Nettoyer vos ressources Azure

```azurecli
az group delete --name MyResourceGroup --yes --no-wait
```

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage plus sur le fonctionnement d’Azure Dev Spaces.

> [!div class="nextstepaction"]
> [Fonctionnement d’Azure Dev Spaces](../how-dev-spaces-works.md)

[azure-cli-installed]: /cli/azure/install-azure-cli?view=azure-cli-latest
[az-ad-sp-create-for-rbac]: /cli/azure/ad/sp#az-ad-sp-create-for-rbac
[az-acr-show]: /cli/azure/acr#az-acr-show
[az-aks-show]: /cli/azure/aks?view=azure-cli-latest#az-aks-show
[az-role-assignment-create]: /cli/azure/role/assignment#az-role-assignment-create
[bikes-server-js]: https://github.com/Azure/dev-spaces/blob/master/samples/BikeSharingApp/Bikes/server.js#L232-L233
[bike-sharing-gh]: https://github.com/Azure/dev-spaces/
[bike-sharing-values-yaml]: https://github.com/Azure/dev-spaces/blob/master/samples/BikeSharingApp/charts/values.yaml
[github-actions-beta-signup]: https://github.com/features/actions
[github-action-yaml]: https://github.com/Azure/dev-spaces/blob/master/.github/workflows/bikes.yml
[github-action-bikesharing-yaml]: https://github.com/Azure/dev-spaces/blob/master/.github/workflows/bikesharing.yml
[helm-installed]: https://helm.sh/docs/intro/install/
[supported-regions]: https://azure.microsoft.com/global-infrastructure/services/?products=kubernetes-service
[sp-acr]: ../../container-registry/container-registry-auth-service-principal.md
[sp-aks]: ../../aks/kubernetes-service-principal.md
