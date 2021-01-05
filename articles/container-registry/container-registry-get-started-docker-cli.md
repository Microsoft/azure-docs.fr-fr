---
title: Envoyer (push) et tirer (pull) une image Docker
description: Transmission et extraction des images Docker à un Registre de conteneur privé dans Azure à l’aide de l’interface de ligne de commande (CLI) Docker
ms.topic: article
ms.date: 01/23/2019
ms.custom: seodec18, H1Hack27Feb2017, devx-track-azurecli
ms.openlocfilehash: 067b5749332c2c41b86d22e7de60083d5f61a442
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96020056"
---
# <a name="push-your-first-image-to-a-private-docker-container-registry-using-the-docker-cli"></a>Transmission de votre première image vers un Registre de conteneur Docker privé à l’aide de l’interface de ligne de commande (CLI) Docker

Un Registre de conteneur Azure stocke et gère les images privées du conteneur [Docker](https://hub.docker.com), de la même manière que [Docker Hub](https://hub.docker.com/) stocke les images publiques du Docker. Vous pouvez utiliser [l’interface de ligne de commande Docker](https://docs.docker.com/engine/reference/commandline/cli/) (interface CLI Docker) pour la [connexion (login)](https://docs.docker.com/engine/reference/commandline/login/), la [poussée (push)](https://docs.docker.com/engine/reference/commandline/push/), le [tirage (pull)](https://docs.docker.com/engine/reference/commandline/pull/) et autres opérations sur le registre du conteneur.

Dans les étapes suivantes, téléchargez une [image Nginx](https://store.docker.com/images/nginx) officielle à partir du registre Docker Hub public, étiquetez-la pour votre registre de conteneurs Azure privé, poussez-la dans votre registre, puis tirez-la du registre.

## <a name="prerequisites"></a>Prérequis

* **Azure Container Registry** : créez un Registre de conteneur dans votre abonnement Azure. Par exemple, utilisez le [portail Azure](container-registry-get-started-portal.md) ou [Azure CLI](container-registry-get-started-azure-cli.md).
* **Docker CLI** : Docker doit également être installé en local. Docker fournit des packages qui le configurent facilement sur n’importe quel système [macOS][docker-mac], [Windows][docker-windows] ou [Linux][docker-linux].

## <a name="log-in-to-a-registry"></a>Se connecter à un Registre

Il existe [plusieurs façons de s’authentifier](container-registry-authentication.md) auprès de votre registre de conteneurs privé. La méthode recommandée avec une ligne de commande consiste à utiliser la commande Azure CLI [az acr login](/cli/azure/acr?view=azure-cli-latest#az-acr-login). Par exemple, pour vous connecter à un registre nommé *myregistry* :

```azurecli
az acr login --name myregistry
```

Vous pouvez également vous connecter avec la commande [docker login](https://docs.docker.com/engine/reference/commandline/login/). Par exemple, vous pouvez avoir [affecté un principal du service](container-registry-authentication.md#service-principal) à votre registre dans un scénario d’automatisation. Lorsque vous exécutez la commande suivante, fournissez de manière interactive l'appID (nom d'utilisateur) et le mot de passe du principal du service dès que vous y êtes invité. Pour connaître les meilleures pratiques de gestion des informations d'identification, consultez la référence de la commande [docker login](https://docs.docker.com/engine/reference/commandline/login/) :

```
docker login myregistry.azurecr.io
```

Les deux commandes retournent `Login Succeeded` une fois terminées.

> [!TIP]
> Spécifiez toujours le nom complet du registre (tout en minuscules) lorsque vous utilisez `docker login` et lorsque vous étiquetez des images à pousser dans votre registre. Dans les exemples de cet article, le nom complet est *myregistry.azurecr.io*.

## <a name="pull-the-official-nginx-image"></a>Tirer (pull) l’image Nginx officielle

Tout d’abord, tirez l’image Nginx publique sur votre ordinateur local.

```
docker pull nginx
```

## <a name="run-the-container-locally"></a>Exécutez localement le conteneur

Exécutez la commande [docker run](https://docs.docker.com/engine/reference/run/) suivante pour démarrer une instance locale du conteneur Nginx de manière interactive (`-it`) sur le port 8080. L’argument `--rm` spécifie que le conteneur doit être supprimé lorsque vous l’arrêtez.

```
docker run -it --rm -p 8080:80 nginx
```

Accédez à `http://localhost:8080` pour afficher la page web par défaut servie par Nginx dans le conteneur en cours d’exécution. Une page similaire à celle ci-dessous doit s'afficher :

![Nginx sur un ordinateur local](./media/container-registry-get-started-docker-cli/nginx.png)

Étant donné que vous avez démarré le conteneur de manière interactive avec `-it`, vous pouvez voir la sortie du serveur Nginx sur la ligne de commande après y avoir accédé dans votre navigateur.

Pour arrêter et supprimer le conteneur, appuyez sur `Control`+`C`.

## <a name="create-an-alias-of-the-image"></a>Créer un alias de l’image

Utilisez une [balise Docker](https://docs.docker.com/engine/reference/commandline/tag/) pour créer un alias de l’image, avec un chemin complet vers votre registre. Cet exemple spécifie l’espace de noms `samples` pour éviter l’encombrement à la racine du Registre.

```
docker tag nginx myregistry.azurecr.io/samples/nginx
```

Pour plus d’informations sur le balisage avec des espaces de noms, consultez la section [Espaces de noms du référentiel](container-registry-best-practices.md#repository-namespaces) dans [Bonnes pratiques pour Azure Container Registry](container-registry-best-practices.md).

## <a name="push-the-image-to-your-registry"></a>Pousser (push) l’image dans votre registre

Maintenant que vous avez étiqueté l’image avec le chemin complet de votre registre privé, vous pouvez la pousser dans le registre avec [docker push](https://docs.docker.com/engine/reference/commandline/push/) :

```
docker push myregistry.azurecr.io/samples/nginx
```

## <a name="pull-the-image-from-your-registry"></a>Tirer (pull) l’image de votre registre

Utilisez la commande [docker pull](https://docs.docker.com/engine/reference/commandline/pull/) pour tirer l’image de votre registre :

```
docker pull myregistry.azurecr.io/samples/nginx
```

## <a name="start-the-nginx-container"></a>Démarrer le conteneur Nginx

Utilisez la commande [docker run](https://docs.docker.com/engine/reference/run/) pour exécuter l’image que vous avez tirée de votre registre :

```
docker run -it --rm -p 8080:80 myregistry.azurecr.io/samples/nginx
```

Accédez à `http://localhost:8080` pour afficher le conteneur en cours d’exécution.

Pour arrêter et supprimer le conteneur, appuyez sur `Control`+`C`.

## <a name="remove-the-image-optional"></a>Supprimer l’image (facultatif)

Si vous n’avez plus besoin de l’image Nginx, vous pouvez la supprimer localement à l’aide de la commande [docker rmi](https://docs.docker.com/engine/reference/commandline/rmi/).

```
docker rmi myregistry.azurecr.io/samples/nginx
```

Pour supprimer des images de votre registre de conteneurs Azure, vous pouvez utiliser la commande Azure CLI [az acr repository delete](/cli/azure/acr/repository#az-acr-repository-delete). Par exemple, la commande suivante supprime le manifeste référencé par l'étiquette `samples/nginx:latest`, toutes les données de couche uniques et toutes les autres étiquettes référençant le manifeste.

```azurecli
az acr repository delete --name myregistry --image samples/nginx:latest
```

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous connaissez les principes de base, vous êtes prêt à utiliser votre registre. Par exemple, déployez des images de conteneur à partir de votre Registre vers :

* [Azure Kubernetes Service (AKS)](../aks/tutorial-kubernetes-prepare-app.md)
* [Azure Container Instances](../container-instances/container-instances-tutorial-prepare-app.md)
* [Service Fabric](../service-fabric/service-fabric-tutorial-create-container-images.md)

Si vous le souhaitez, vous pouvez installer l’[extension Docker pour Visual Studio Code](https://code.visualstudio.com/docs/azure/docker) et l’extension [Compte Azure](https://marketplace.visualstudio.com/items?itemName=ms-vscode.azure-account) pour utiliser vos registres de conteneurs Azure. Dans Visual Studio Code, vous pouvez tirer (pull) et envoyer (push) des images vers un registre de conteneurs Azure, et exécuter ACR Tasks.


<!-- LINKS - external -->
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-windows]: https://docs.docker.com/docker-for-windows/
