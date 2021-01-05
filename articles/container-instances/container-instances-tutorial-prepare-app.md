---
title: Tutoriel - Préparer le registre de conteneurs pour un déploiement
description: Tutoriel Azure Container Instances (partie 1 sur 3) - Préparer une application dans une image conteneur pour le déploiement sur Azure Container Instances
ms.topic: tutorial
ms.date: 03/21/2018
ms.custom: seodec18, mvc, devx-track-js
ms.openlocfilehash: 1eca34fb5da50b15f15322a3b534556bcd3bda09
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91303112"
---
# <a name="tutorial-create-a-container-image-for-deployment-to-azure-container-instances"></a>Tutoriel : Créer une image conteneur à déployer sur Azure Container Instances

Azure Container Instances permet de déployer des conteneurs Docker sur l’infrastructure Azure sans configurer de machines virtuelles ni adopter de service de niveau supérieur. Dans ce tutoriel, vous allez empaqueter une petite application web Node.js dans une image de conteneur pouvant être exécuté avec Azure Container Instances.

Dans ce premier article de la série, vous allez apprendre à :

> [!div class="checklist"]
> * Cloner le code source de l’application à partir de GitHub.
> * Créer une image conteneur à partir de la source de l’application.
> * Tester l’image dans un environnement Docker local.

Dans les parties deux et trois du tutoriel, vous chargez votre image dans un conteneur Azure Container Registry, et vous la déployez dans Azure Container Instances.

## <a name="before-you-begin"></a>Avant de commencer

[!INCLUDE [container-instances-tutorial-prerequisites](../../includes/container-instances-tutorial-prerequisites.md)]

## <a name="get-application-code"></a>Obtenir le code d’application

L’exemple d’application dans ce tutoriel est une application web basique générée dans [Node.js][nodejs]. L’application dessert une page HTML statique et ressemble à la capture d’écran suivante :

![Application du didacticiel affichée dans le navigateur][aci-tutorial-app]

Utiliser Git pour cloner le référentiel de l’exemple d’application :

```bash
git clone https://github.com/Azure-Samples/aci-helloworld.git
```

Vous pouvez également [télécharger l’archive ZIP][aci-helloworld-zip] directement à partir de GitHub.

## <a name="build-the-container-image"></a>Générer l’image de conteneur

Le fichier Dockerfile de l’exemple d’application montre comment le conteneur est généré. La génération du conteneur commence par une [image officielle Node.js][docker-hub-nodeimage] basée sur [Linux Alpine][alpine-linux], une petite distribution parfaitement adaptée aux conteneurs. Les fichiers d’application sont ensuite copiés dans le conteneur, les dépendances sont installées à l’aide de Node Package Manager, et l’application démarre.

```Dockerfile
FROM node:8.9.3-alpine
RUN mkdir -p /usr/src/app
COPY ./app/ /usr/src/app/
WORKDIR /usr/src/app
RUN npm install
CMD node /usr/src/app/index.js
```

Utilisez la commande [docker build][docker-build] pour créer l’image conteneur, et étiquetez-la *aci-tutorial-app* :

```bash
docker build ./aci-helloworld -t aci-tutorial-app
```

Le résultat de la commande [docker build][docker-build] est similaire à ce qui suit (tronqué pour une meilleure lisibilité) :

```console
$ docker build ./aci-helloworld -t aci-tutorial-app
Sending build context to Docker daemon  119.3kB
Step 1/6 : FROM node:8.9.3-alpine
8.9.3-alpine: Pulling from library/node
88286f41530e: Pull complete
84f3a4bf8410: Pull complete
d0d9b2214720: Pull complete
Digest: sha256:c73277ccc763752b42bb2400d1aaecb4e3d32e3a9dbedd0e49885c71bea07354
Status: Downloaded newer image for node:8.9.3-alpine
 ---> 90f5ee24bee2
...
Step 6/6 : CMD node /usr/src/app/index.js
 ---> Running in f4a1ea099eec
 ---> 6edad76d09e9
Removing intermediate container f4a1ea099eec
Successfully built 6edad76d09e9
Successfully tagged aci-tutorial-app:latest
```

Utilisez la commande [docker images][docker-images] pour voir l’image générée :

```bash
docker images
```

L’image que vous venez de créer doit apparaître dans la liste :

```console
$ docker images
REPOSITORY          TAG       IMAGE ID        CREATED           SIZE
aci-tutorial-app    latest    5c745774dfa9    39 seconds ago    68.1 MB
```

## <a name="run-the-container-locally"></a>Exécutez localement le conteneur

Avant de déployer le conteneur dans Azure Container Instances, utilisez [docker run][docker-run] pour l’exécuter localement et vérifiez qu’il fonctionne. Le commutateur `-d` permet l’exécution du conteneur en arrière-plan, tandis que `-p` vous permet de mapper un port arbitraire sur votre ordinateur vers le port 80 du conteneur.

```bash
docker run -d -p 8080:80 aci-tutorial-app
```

La sortie de la commande `docker run` affiche les ID du conteneur en cours d’exécution en cas de réussite de celle-ci :

```console
$ docker run -d -p 8080:80 aci-tutorial-app
a2e3e4435db58ab0c664ce521854c2e1a1bda88c9cf2fcff46aedf48df86cccf
```

Accédez maintenant à l’adresse `http://localhost:8080` dans votre navigateur pour confirmer l’exécution du conteneur. Une page web similaire à celle ci-dessous doit s’afficher :

![Exécution locale de l’application dans le navigateur][aci-tutorial-app-local]

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez créé une image de conteneur qui peut être déployée dans Azure Container Instances et vérifié qu’elle s’exécute localement. Dans ce tutoriel, vous avez effectué les actions suivantes :

> [!div class="checklist"]
> * Clonage de la source de l’application à partir de GitHub
> * Création d’une image conteneur à partir de la source de l’application
> * Test de l’exécution locale du conteneur

Passez au tutoriel suivant de la série pour en savoir plus sur le stockage d’images de conteneur dans Azure Container Registry :

> [!div class="nextstepaction"]
> [Envoyer l’image à Azure Container Registry](container-instances-tutorial-prepare-acr.md)

<!--- IMAGES --->
[aci-tutorial-app]:./media/container-instances-quickstart/aci-app-browser.png
[aci-tutorial-app-local]: ./media/container-instances-tutorial-prepare-app/aci-app-browser-local.png

<!-- LINKS - External -->
[aci-helloworld-zip]: https://github.com/Azure-Samples/aci-helloworld/archive/master.zip
[alpine-linux]: https://alpinelinux.org/
[docker-build]: https://docs.docker.com/engine/reference/commandline/build/
[docker-get-started]: https://docs.docker.com/get-started/
[docker-hub-nodeimage]: https://store.docker.com/images/node
[docker-images]: https://docs.docker.com/engine/reference/commandline/images/
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-login]: https://docs.docker.com/engine/reference/commandline/login/
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-run]: https://docs.docker.com/engine/reference/commandline/run/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[docker-windows]: https://docs.docker.com/docker-for-windows/
[nodejs]: https://nodejs.org

<!-- LINKS - Internal -->
[azure-cli-install]: /cli/azure/install-azure-cli
