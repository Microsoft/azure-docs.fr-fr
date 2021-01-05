---
title: Fichier include
description: Fichier include
services: container-registry
author: dlepow
ms.service: container-registry
ms.topic: include
ms.date: 08/04/2020
ms.author: danlep
ms.custom: include file
ms.openlocfilehash: 6a16106495e584c9acbc02d380242df665f35ce5
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96019998"
---
## <a name="push-image-to-registry"></a>Envoyer l’image au registre

Pour envoyer une image dans un registre Azure Container Registry, vous devez tout d’abord disposer d’une image. Si vous n’avez pas encore d’images conteneur locales, exécutez la commande [docker pull][docker-pull] suivante pour extraire (pull) une image existante du Docker Hub. Pour cet exemple, tirez l’image `hello-world`.

```
docker pull hello-world
```

Avant d’envoyer (push) une image vers le registre, vous devez la marquer avec le nom complet de votre serveur de connexion au registre. Le nom du serveur de connexion est au format *\<registry-name\>.azurecr.io* (obligatoirement tout en minuscules). Par exemple : *mycontainerregistry.azurecr.io*.

Étiquetez l’image en utilisant la commande [docker tag][docker-tag]. Remplacez `<login-server>` par le nom du serveur de connexion de votre instance ACR.

```
docker tag hello-world <login-server>/hello-world:v1
```

Exemple :

```
docker tag hello-world mycontainerregistry.azurecr.io/hello-world:v1
```


Pour finir, utilisez la commande [docker push][docker-push] pour envoyer l’image vers l’instance du registre. Remplacez `<login-server>` par le nom du serveur de connexion de votre instance de registre. Cet exemple crée le référentiel **hello-world** qui contient l’image `hello-world:v1`.

```
docker push <login-server>/hello-world:v1
```

Après avoir envoyé (push) l’image à votre registre de conteneurs, supprimez l’image `hello-world:v1` de votre environnement Docker local. (Notez que cette commande [docker rmi][docker-rmi] ne supprime pas l’image du référentiel **hello-world** dans votre registre de conteneurs Azure.)

```
docker rmi <login-server>/hello-world:v1
```

<!-- LINKS - External -->
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-pull]: https://docs.docker.com/engine/reference/commandline/pull/
[docker-rmi]: https://docs.docker.com/engine/reference/commandline/rmi/
[docker-run]: https://docs.docker.com/engine/reference/commandline/run/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/

<!-- LINKS - Internal -->

