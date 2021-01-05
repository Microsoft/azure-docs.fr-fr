---
title: 'Didacticiel : Utiliser Docker Compose pour déployer plusieurs conteneurs'
description: Utilisez Docker Compose pour générer et exécuter une application à plusieurs conteneurs, puis intégrez l’application à Azure Container Instances
ms.topic: tutorial
ms.date: 10/28/2020
ms.custom: ''
ms.openlocfilehash: a71ff438feaef555a85c33d818c287c64621d40d
ms.sourcegitcommit: d76108b476259fe3f5f20a91ed2c237c1577df14
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92913838"
---
# <a name="tutorial-deploy-a-multi-container-group-using-docker-compose"></a>Tutoriel : Déployer plusieurs conteneurs à l’aide de Docker Compose 

Dans ce didacticiel, vous allez utiliser [Docker Compose](https://docs.docker.com/compose/) pour définir et exécuter une application à plusieurs conteneurs localement, puis la déployer en tant que [groupe de conteneurs](container-instances-container-groups.md) dans Azure Container Instances. 

Exécutez des conteneurs dans Azure Container Instances à la demande lorsque vous développez des applications cloud natives avec Docker et que vous souhaitez passer de façon fluide d’un développement local à un déploiement cloud. Cette fonctionnalité est activée par l’[intégration entre Docker et Azure](https://docs.docker.com/engine/context/aci-integration/). Vous pouvez utiliser des commandes Docker natives pour exécuter [une instance de conteneur unique](quickstart-docker-cli.md) ou plusieurs conteneurs dans Azure.

> [!IMPORTANT]
> Toutes les fonctionnalités d’Azure Container Instances ne sont pas prises en charge. Fournissez des commentaires sur l’intégration Docker-Azure en créant un problème dans le référentiel GitHub [Docker ACI Integration](https://github.com/docker/aci-integration-beta).

> [!TIP]
> Vous pouvez utiliser l’[extension Docker pour Visual Studio Code](https://aka.ms/VSCodeDocker) afin de bénéficier d’une expérience intégrée de développement, d’exécution et de gestion des conteneurs, des images et des contextes.

Dans cet article, vous découvrirez comment :

> [!div class="checklist"]
> * Créer un registre de conteneurs Azure
> * Cloner le code source de l’application à partir de GitHub.
> * Utiliser Docker Compose pour générer une image et exécuter une application à plusieurs conteneurs en local
> * Envoyer l’image de l’application à votre registre de conteneurs
> * Créer un contexte Azure pour Docker
> * Intégrer l’application dans Azure Container Instances

## <a name="prerequisites"></a>Prérequis

* **Azure CLI**  : Azure CLI doit être installé sur votre ordinateur local. La version 2.10.1 ou ultérieure est recommandée. Exécutez `az --version` pour trouver la version. Si vous devez effectuer une installation ou une mise à niveau, consultez [Installer Azure CLI](/cli/azure/install-azure-cli).

* **Station de travail Docker**  : Docker version 2.3.0.5 ou ultérieure doit être installé, disponible pour [Windows](https://desktop.docker.com/win/edge/Docker%20Desktop%20Installer.exe) ou [macOS](https://desktop.docker.com/mac/edge/Docker.dmg). Vous pouvez également installer [Docker ACI Integration CLI pour Linux](https://docs.docker.com/engine/context/aci-integration/#install-the-docker-aci-integration-cli-on-linux).

[!INCLUDE [container-instances-create-registry](../../includes/container-instances-create-registry.md)]

## <a name="get-application-code"></a>Obtenir le code d’application

L’exemple d’application utilisé dans ce didacticiel est une application de votes de base. L’application est constituée d’un composant web frontal et d’une instance Redis principale. Le composant web est empaqueté dans une image conteneur personnalisée. L’instance Redis utilise une image non modifiée de Docker Hub.

Utilisez [git](https://git-scm.com/downloads) pour cloner l’exemple d’application dans votre environnement de développement :

```console
git clone https://github.com/Azure-Samples/azure-voting-app-redis.git
```

Passez au répertoire cloné.

```console
cd azure-voting-app-redis
```

Dans le répertoire se trouvent le code source de l’application, un fichier Docker Compose précréé et docker-compose.yaml.

## <a name="modify-docker-compose-file"></a>Modifier le fichier Docker Compose

Ouvrez docker-compose.yaml dans un éditeur de texte. Le fichier configure les services `azure-vote-back` et `azure-vote-front`.

```yml
version: '3'
services:
  azure-vote-back:
    image: mcr.microsoft.com/oss/bitnami/redis:6.0.8
    container_name: azure-vote-back
    environment:
      ALLOW_EMPTY_PASSWORD: "yes"
    ports:
        - "6379:6379"

  azure-vote-front:
    build: ./azure-vote
    image: mcr.microsoft.com/azuredocs/azure-vote-front:v1
    container_name: azure-vote-front
    environment:
      REDIS: azure-vote-back
    ports:
        - "8080:80"
```

Dans la configuration `azure-vote-front`, apportez les modifications suivantes :

1. Mettez à jour la propriété `image` dans le service `azure-vote-front`. Préfixez le nom de l’image avec le nom du serveur de connexion de votre registre de conteneurs Azure, \<acrName\>.azurecr.io. Par exemple, si votre registre est nommé *myregistry* , le nom du serveur de connexion est *myregistry.azurecr.io* (tout en minuscules) et la propriété de l’image est alors `myregistry.azurecr.io/azure-vote-front`.
1. Modifiez le mappage de `ports` en `80:80`. Enregistrez le fichier.

Le fichier mis à jour doit ressembler au suivant :

```yml
version: '3'
services:
  azure-vote-back:
    image: mcr.microsoft.com/oss/bitnami/redis:6.0.8
    container_name: azure-vote-back
    environment:
      ALLOW_EMPTY_PASSWORD: "yes"
    ports:
        - "6379:6379"

  azure-vote-front:
    build: ./azure-vote
    image: myregistry.azurecr.io/azure-vote-front
    container_name: azure-vote-front
    environment:
      REDIS: azure-vote-back
    ports:
        - "80:80"
```

Si vous effectuez ces substitutions, l’image `azure-vote-front` que vous créez à l’étape suivante est marquée pour votre registre de conteneurs Azure et l’image peut être extraite pour s’exécuter dans Azure Container Instances.

> [!TIP]
> Vous ne devez pas utiliser de registre de conteneurs Azure pour ce scénario. Par exemple, vous pouvez choisir un référentiel privé dans Docker Hub pour héberger l’image de votre application. Si vous choisissez un autre registre, mettez à jour la propriété d’image de manière appropriée.

## <a name="run-multi-container-application-locally"></a>Exécuter une application à plusieurs conteneurs en local

Exécutez [docker-compose](https://docs.docker.com/compose/reference/up/), qui utilise l’exemple de fichier `docker-compose.yaml` pour générer l’image conteneur, télécharger l’image Redis, puis démarrez l’application :

```console
docker-compose up --build -d
```

Une fois terminé, utilisez la commande [docker images](https://docs.docker.com/engine/reference/commandline/images/) pour afficher les images créées. Trois images ont été téléchargées ou créées. L’image `azure-vote-front` contient l’application de serveur frontal, qui utilise l’image `uwsgi-nginx-flask` comme base. L’image `redis` est utilisée pour démarrer une instance Redis.

```
$ docker images

REPOSITORY                                TAG        IMAGE ID            CREATED             SIZE
myregistry.azurecr.io/azure-vote-front    latest     9cc914e25834        40 seconds ago      944MB
mcr.microsoft.com/oss/bitnami/redis       6.0.8      3a54a920bb6c        4 weeks ago          103MB
tiangolo/uwsgi-nginx-flask                python3.6  788ca94b2313        9 months ago        9444MB
```

Exécutez la commande [docker ps](https://docs.docker.com/engine/reference/commandline/ps/) pour voir les conteneurs en cours d’exécution :

```
$ docker ps

CONTAINER ID        IMAGE                                      COMMAND                  CREATED             STATUS              PORTS                           NAMES
82411933e8f9        myregistry.azurecr.io/azure-vote-front     "/entrypoint.sh /sta…"   57 seconds ago      Up 30 seconds       443/tcp, 0.0.0.0:80->80/tcp   azure-vote-front
b62b47a7d313        mcr.microsoft.com/oss/bitnami/redis:6.0.8  "/opt/bitnami/script…"   57 seconds ago      Up 30 seconds       0.0.0.0:6379->6379/tcp          azure-vote-back
```

Pour voir l’application en cours d’exécution, entrez `http://localhost:80` dans un navigateur web local. L’exemple d’application est chargée, comme indiqué dans l’exemple suivant :

:::image type="content" source="media/tutorial-docker-compose/azure-vote.png" alt-text="Image de l’application Voting":::

Après avoir essayé l’application locale, exécutez [docker-compose](https://docs.docker.com/compose/reference/down/) pour arrêter l’application et supprimer les conteneurs.

```console
docker-compose down
```

## <a name="push-image-to-container-registry"></a>Envoyer l’image au registre de conteneurs

Pour déployer l’application dans Azure Container Instances, vous devez envoyer l’image `azure-vote-front` à votre registre de conteneurs. Exécutez [docker-compose](https://docs.docker.com/compose/reference/push) pour envoyer l’image :

```console
docker-compose push
```

L’envoi vers le registre peut prendre quelques minutes.

Pour vérifier que l’image est stockée dans votre registre, exécutez la commande [az acr repository show](/cli/azure/acr/repository#az-acr-repository-show) :

```azurecli
az acr repository show --name <acrName> --repository azure-vote-front
```

[!INCLUDE [container-instances-create-docker-context](../../includes/container-instances-create-docker-context.md)]

## <a name="deploy-application-to-azure-container-instances"></a>Déployer une application dans Azure Container Instances

Ensuite, accédez au contexte ACI. Les commandes Docker ultérieures s’exécutent dans ce contexte.

```console
docker context use myacicontext
```

Exécutez `docker compose up` pour démarrer l’application dans Azure Container Instances. L’image `azure-vote-front` est extraite de votre registre de conteneurs et le groupe de conteneurs est créé dans Azure Container Instances.

```console
docker compose up
```

> [!NOTE]
> Les commandes Docker Compose actuellement disponibles dans un contexte ACI sont `docker compose up` et `docker compose down`. Il n’y a pas de trait d’union entre `docker` et `compose` dans ces commandes.

Le groupe de conteneurs est déployé très rapidement. Exemple de sortie :

```
[+] Running 3/3
 ⠿ Group azurevotingappredis  Created                          3.6s
 ⠿ azure-vote-back            Done                             10.6s
 ⠿ azure-vote-front           Done                             10.6s
```

Exécutez `docker ps` pour voir les conteneurs en cours d’exécution et l’adresse IP assignée au groupe de conteneurs.

```console
docker ps
```

Exemple de sortie :

```
CONTAINER ID                           IMAGE                                         COMMAND             STATUS              PORTS
azurevotingappredis_azure-vote-back    mcr.microsoft.com/oss/bitnami/redis:6.0.8                         Running             52.179.23.131:6379->6379/tcp
azurevotingappredis_azure-vote-front   myregistry.azurecr.io/azure-vote-front                            Running             52.179.23.131:80->80/tcp
```

Pour afficher l’application en cours d’exécution dans le cloud, entrez l’adresse IP affichée dans un navigateur web local. Dans cet exemple, entrez `52.179.23.131`. L’exemple d’application est chargée, comme indiqué dans l’exemple suivant :

:::image type="content" source="media/tutorial-docker-compose/azure-vote-aci.png" alt-text="Image de l’application Voting":::

Pour afficher les journaux du conteneur frontal, exécutez la commande [docker logs](https://docs.docker.com/engine/reference/commandline/logs). Exemple :

```console
docker logs azurevotingappredis_azure-vote-front
```

Vous pouvez également utiliser le portail Azure ou d’autres outils Azure pour afficher les propriétés et l’état du groupe de conteneurs que vous avez déployé.

Lorsque vous avez terminé de tester l’application, arrêtez l’application et les conteneurs avec `docker compose down` :

```console
docker compose down
```

Cette commande supprime le groupe de conteneurs dans Azure Container Instances.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez utilisé Docker Compose pour passer de l’exécution d’une application à plusieurs conteneurs en local à l’exécution dans Azure Container Instances. Vous avez appris à :

> [!div class="checklist"]
> * Créer un registre de conteneurs Azure
> * Cloner le code source de l’application à partir de GitHub.
> * Utiliser Docker Compose pour générer une image et exécuter une application à plusieurs conteneurs en local
> * Envoyer l’image de l’application à votre registre de conteneurs
> * Créer un contexte Azure pour Docker
> * Intégrer l’application dans Azure Container Instances

Vous pouvez également utiliser l’[extension Docker pour Visual Studio Code](https://aka.ms/VSCodeDocker) afin de bénéficier d’une expérience intégrée de développement, d’exécution et de gestion des conteneurs, des images et des contextes.

Si vous souhaitez tirer parti d’un plus grand nombre de fonctionnalités dans Azure Container Instances, utilisez les outils Azure pour spécifier un groupe de plusieurs conteneurs. Par exemple, consultez les didacticiels pour déployer un groupe de conteneurs à l’aide d’Azure CLI avec un [fichier YAML](container-instances-multi-container-yaml.md) ou déployer à l’aide d’un [modèle Azure Resource Manager](container-instances-multi-container-group.md). 