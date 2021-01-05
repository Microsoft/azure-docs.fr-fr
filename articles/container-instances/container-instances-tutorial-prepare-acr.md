---
title: Tutoriel - Préparer le registre de conteneurs pour déployer une image
description: Tutoriel Azure Container Instances (partie 2 sur 3) - Préparer un registre de conteneurs Azure et envoyer une image
ms.topic: tutorial
ms.date: 12/18/2019
ms.custom: seodec18, mvc, devx-track-azurecli
ms.openlocfilehash: 2eda960c53fc7ba851ffcfbe96bd8e9a48844910
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92746941"
---
# <a name="tutorial-create-an-azure-container-registry-and-push-a-container-image"></a>Tutoriel : Créer un registre de conteneurs Azure et envoyer une image conteneur

Il s’agit de la deuxième partie d’un didacticiel en trois parties. Dans la [première partie](container-instances-tutorial-prepare-app.md) du didacticiel, vous avez créé une image de conteneur Docker pour une application web Node.js. Dans ce didacticiel, vous envoyez cette image à Azure Container Registry. Si vous n’avez pas encore créé l’image de conteneur, retournez au [Didacticiel 1 : Créer une image conteneur](container-instances-tutorial-prepare-app.md).

Azure Container Registry est votre registre Docker privé dans Azure. Dans ce tutoriel, la deuxième partie de la série, vous allez :

> [!div class="checklist"]
> * Créer une instance Azure Container Registry avec Azure CLI.
> * Baliser une image conteneur pour votre Azure Container Registry
> * Charger l’image dans votre registre

Dans ce dernier article de la série, vous allez déployer le conteneur de votre registre privé vers Azure Container Instances.

## <a name="before-you-begin"></a>Avant de commencer

[!INCLUDE [container-instances-tutorial-prerequisites](../../includes/container-instances-tutorial-prerequisites.md)]

[!INCLUDE [container-instances-create-registry](../../includes/container-instances-create-registry.md)]

## <a name="tag-container-image"></a>Baliser l’image de conteneur

Pour envoyer une image de conteneur sur un registre privé comme Azure Container Registry, vous devez d’abord étiqueter l’image avec le nom complet du serveur de connexion du registre.

Tout d’abord, obtenez le nom complet du serveur de connexion pour votre registre de conteneurs Azure. Exécutez la commande [az acr show][az-acr-show] suivante et remplacez `<acrName>` par le nom du registre que vous venez de créer :

```azurecli
az acr show --name <acrName> --query loginServer --output table
```

Par exemple, si votre registre est nommé *mycontainerregistry082* :

```azurecli
az acr show --name mycontainerregistry082 --query loginServer --output table
```

```output
Result
------------------------
mycontainerregistry082.azurecr.io
```

Maintenant, affichez la liste de vos images locales avec la commande [docker images][docker-images] :

```bash
docker images
```

Vous devriez voir l’image *aci-didacticiel-app* que vous avez créé dans le [didacticiel précédent](container-instances-tutorial-prepare-app.md) en même temps que toutes les autres images vous avez sur votre ordinateur :

```console
$ docker images
REPOSITORY          TAG       IMAGE ID        CREATED           SIZE
aci-tutorial-app    latest    5c745774dfa9    39 minutes ago    68.1 MB
```

Étiquetez l’image *aci-tutorial-app* à l’aide du serveur de connexion de votre registre de conteneurs. Ajoutez également la balise `:v1` à la fin du nom d’image pour indiquer le numéro de version de l’image. Remplacez `<acrLoginServer>` par le résultat de la commande [az acr show][az-acr-show] que vous avez exécutée précédemment.

```bash
docker tag aci-tutorial-app <acrLoginServer>/aci-tutorial-app:v1
```

Exécutez `docker images` de nouveau pour vérifier l’opération de marquage :

```console
$ docker images
REPOSITORY                                            TAG       IMAGE ID        CREATED           SIZE
aci-tutorial-app                                      latest    5c745774dfa9    39 minutes ago    68.1 MB
mycontainerregistry082.azurecr.io/aci-tutorial-app    v1        5c745774dfa9    7 minutes ago     68.1 MB
```

## <a name="push-image-to-azure-container-registry"></a>Envoyer l’image à Azure Container Registry

Maintenant que vous avez étiqueté l’image *aci-tutorial-app* avec le nom complet du serveur de connexion de votre registre privé, vous pouvez envoyer (push) l’image au registre avec la commande [docker push][docker-push]. Remplacez `<acrLoginServer>` par le nom complet du serveur de connexion que vous avez obtenu à l’étape précédente.

```bash
docker push <acrLoginServer>/aci-tutorial-app:v1
```

L’opération `push` peut prendre de quelques secondes à quelques minutes selon votre connexion Internet, et la sortie est semblable à ce qui suit :

```console
$ docker push mycontainerregistry082.azurecr.io/aci-tutorial-app:v1
The push refers to a repository [mycontainerregistry082.azurecr.io/aci-tutorial-app]
3db9cac20d49: Pushed
13f653351004: Pushed
4cd158165f4d: Pushed
d8fbd47558a8: Pushed
44ab46125c35: Pushed
5bef08742407: Pushed
v1: digest: sha256:ed67fff971da47175856505585dcd92d1270c3b37543e8afd46014d328f05715 size: 1576
```

## <a name="list-images-in-azure-container-registry"></a>Énumérer les images dans Azure Container Registry

Pour vérifier que l’image que vous venez d’envoyer est bien dans le registre de conteneurs Azure, listez les images de votre registre avec la commande [az acr repository list][az-acr-repository-list]. Remplacez `<acrName>` par le nom de votre registre de conteneurs.

```azurecli
az acr repository list --name <acrName> --output table
```

Par exemple :

```azurecli
az acr repository list --name mycontainerregistry082 --output table
```

```output
Result
----------------
aci-tutorial-app
```

Pour voir les *étiquettes* d’une image spécifique, utilisez la commande [az acr repository show-tags][az-acr-repository-show-tags].

```azurecli
az acr repository show-tags --name <acrName> --repository aci-tutorial-app --output table
```

Vous devez obtenir une sortie similaire à la suivante :

```console
az acr repository show-tags --name mycontainerregistry082 --repository aci-tutorial-app --output table
Result
--------
v1
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez préparé un registre de conteneurs Azure pour une utilisation avec Azure Container Instances, et envoyé une image conteneur vers le registre. Les étapes suivantes ont été effectuées :

> [!div class="checklist"]
> * Création d’une instance Azure Container Registry avec Azure CLI
> * Marquage d’une image conteneur pour Azure Container Registry
> * Chargement d’une image dans Azure Container Registry

Passez au didacticiel suivant pour découvrir comment déployer le conteneur sur Azure à l’aide d’Azure Container Instances :

> [!div class="nextstepaction"]
> [Déployer un conteneur sur Azure Container Instances](container-instances-tutorial-deploy-app.md)

<!-- LINKS - External -->
[docker-build]: https://docs.docker.com/engine/reference/commandline/build/
[docker-get-started]: https://docs.docker.com/get-started/
[docker-hub-nodeimage]: https://store.docker.com/images/node
[docker-images]: https://docs.docker.com/engine/reference/commandline/images/
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-login]: https://docs.docker.com/engine/reference/commandline/login/
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[docker-windows]: https://docs.docker.com/docker-for-windows/
[nodejs]: https://nodejs.org

<!-- LINKS - Internal -->
[az-acr-create]: /cli/azure/acr#az-acr-create
[az-acr-login]: /cli/azure/acr#az-acr-login
[az-acr-repository-list]: /cli/azure/acr/repository
[az-acr-repository-show-tags]: /cli/azure/acr/repository#az-acr-repository-show-tags
[az-acr-show]: /cli/azure/acr#az-acr-show
[az-group-create]: /cli/azure/group#az-group-create
[azure-cli-install]: /cli/azure/install-azure-cli
