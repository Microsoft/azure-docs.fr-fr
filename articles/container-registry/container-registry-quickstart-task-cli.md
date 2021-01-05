---
title: 'Démarrage rapide : créer une image conteneur à la demande dans Azure'
description: Utilisez les commandes Azure Container Registry pour générer, envoyer (push) et exécuter rapidement une image conteneur Docker à la demande, dans le cloud Azure.
ms.topic: quickstart
ms.date: 09/25/2020
ms.custom: contperf-fy21q1, devx-track-azurecli
ms.openlocfilehash: c6fe1fc246d112218b492072155175b2db99c8c9
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97032947"
---
# <a name="quickstart-build-and-run-a-container-image-using-azure-container-registry-tasks"></a>Démarrage rapide : générer et exécuter une image conteneur avec Azure Container Registry Tasks

Dans ce guide de démarrage rapide, vous avez utilisé les commandes [Azure Container Registry Tasks][container-registry-tasks-overview] pour générer, envoyer (push) et exécuter rapidement une image conteneur Docker en mode natif dans Azure, sans installation Docker locale. ACR Tasks est une suite de fonctionnalités dans Azure Container Registry qui vous aide à gérer et à modifier des images conteneur tout au long du cycle de vie du conteneur. Cet exemple montre comment décharger le cycle de développement d’image de conteneur « boucle interne » dans le cloud avec des builds à la demande à l’aide d’un Dockerfile local. 

Après avoir suivi ce guide de démarrage rapide, explorez les fonctionnalités plus avancées d’ACR Tasks à l’aide des [tutoriels](container-registry-tutorial-quick-task.md). ACR Tasks peut automatiser les builds d’image en fonction de commits de code ou de mises à jour de l’image de base, ou tester plusieurs conteneurs, en parallèle, entre autres scénarios. 

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]
    
- Pour ce guide de démarrage rapide, vous devez avoir la version 2.0.58 ou ultérieure de l’interface Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Si vous ne disposez pas d’un registre de conteneurs, commencez par créer un groupe de ressources avec la commande [az group create][az-group-create]. Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure sont déployées et gérées.

L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *eastus*.

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

## <a name="create-a-container-registry"></a>Créer un registre de conteneur

Créez un registre de conteneurs à l’aide de la commande [az acr create][az-acr-create]. Le nom du registre doit être unique dans Azure et contenir entre 5 et 50 caractères alphanumériques. Dans l’exemple suivant, nous utilisons le nom *myContainerRegistry008*. Mettez à jour le nom de façon à utiliser une valeur unique.

```azurecli-interactive
az acr create --resource-group myResourceGroup \
  --name myContainerRegistry008 --sku Basic
```

Cet exemple crée un registre *De base*, option économique pour les développeurs qui apprennent à se servir d’Azure Container Registry. Pour plus d’informations sur les niveaux de service disponibles, consultez [Niveaux de service des registres de conteneurs][container-registry-skus].

## <a name="build-and-push-image-from-a-dockerfile"></a>Générer et envoyer (push) une image à partir d’un Dockerfile

Maintenant, utilisez Azure Container Registry pour générer et envoyer (push) une image. Tout d’abord, créez un répertoire de travail local, puis un Dockerfile nommé *Dockerfile* avec la ligne unique : `FROM mcr.microsoft.com/hello-world`. Il s’agit d’un exemple simple de création d’une image conteneur Linux à partir de l’image `hello-world` hébergée sur Microsoft Container Registry. Vous pouvez créer votre propre Dockerfile standard et générer des images pour d’autres plateformes. Si vous utilisez un interpréteur de commandes bash, créez le Dockerfile à l’aide de la commande suivante :

```bash
echo FROM mcr.microsoft.com/hello-world > Dockerfile
```

Exécutez la commande [az acr build][az-acr-build], qui génère l’image et, une fois l’image correctement générée, l’envoie (push) à votre registre. L’exemple suivant génère et envoie (push) l’image `sample/hello-world:v1`. Le `.` à la fin de la commande définit l’emplacement du fichier Dockerfile, dans ce cas, le répertoire actif.

```azurecli-interactive
az acr build --image sample/hello-world:v1 \
  --registry myContainerRegistry008 \
  --file Dockerfile . 
```

La sortie d’une opération de génération et d’envoi (push) réussie se présente ainsi :

```console
Packing source code into tar to upload...
Uploading archived source code from '/tmp/build_archive_b0bc1e5d361b44f0833xxxx41b78c24e.tar.gz'...
Sending context (1.856 KiB) to registry: mycontainerregistry008...
Queued a build with ID: ca8
Waiting for agent...
2019/03/18 21:56:57 Using acb_vol_4c7ffa31-c862-4be3-xxxx-ab8e615c55c4 as the home volume
2019/03/18 21:56:57 Setting up Docker configuration...
2019/03/18 21:56:58 Successfully set up Docker configuration
2019/03/18 21:56:58 Logging in to registry: mycontainerregistry008.azurecr.io
2019/03/18 21:56:59 Successfully logged into mycontainerregistry008.azurecr.io
2019/03/18 21:56:59 Executing step ID: build. Working directory: '', Network: ''
2019/03/18 21:56:59 Obtaining source code and scanning for dependencies...
2019/03/18 21:57:00 Successfully obtained source code and scanned for dependencies
2019/03/18 21:57:00 Launching container with name: build
Sending build context to Docker daemon  13.82kB
Step 1/1 : FROM mcr.microsoft.com/hello-world
latest: Pulling from hello-world
Digest: sha256:2557e3c07ed1e38f26e389462d03ed943586fxxxx21577a99efb77324b0fe535
Successfully built fce289e99eb9
Successfully tagged mycontainerregistry008.azurecr.io/sample/hello-world:v1
2019/03/18 21:57:01 Successfully executed container: build
2019/03/18 21:57:01 Executing step ID: push. Working directory: '', Network: ''
2019/03/18 21:57:01 Pushing image: mycontainerregistry008.azurecr.io/sample/hello-world:v1, attempt 1
The push refers to repository [mycontainerregistry008.azurecr.io/sample/hello-world]
af0b15c8625b: Preparing
af0b15c8625b: Layer already exists
v1: digest: sha256:92c7f9c92844bbbb5d0a101b22f7c2a7949e40f8ea90c8b3bc396879d95e899a size: 524
2019/03/18 21:57:03 Successfully pushed image: mycontainerregistry008.azurecr.io/sample/hello-world:v1
2019/03/18 21:57:03 Step ID: build marked as successful (elapsed time in seconds: 2.543040)
2019/03/18 21:57:03 Populating digests for step ID: build...
2019/03/18 21:57:05 Successfully populated digests for step ID: build
2019/03/18 21:57:05 Step ID: push marked as successful (elapsed time in seconds: 1.473581)
2019/03/18 21:57:05 The following dependencies were found:
2019/03/18 21:57:05
- image:
    registry: mycontainerregistry008.azurecr.io
    repository: sample/hello-world
    tag: v1
    digest: sha256:92c7f9c92844bbbb5d0a101b22f7c2a7949e40f8ea90c8b3bc396879d95e899a
  runtime-dependency:
    registry: registry.hub.docker.com
    repository: library/hello-world
    tag: v1
    digest: sha256:2557e3c07ed1e38f26e389462d03ed943586f744621577a99efb77324b0fe535
  git: {}

Run ID: ca8 was successful after 10s
```

## <a name="run-the-image"></a>Exécuter l’image

Maintenant, exécutez rapidement l’image que vous avez générée et envoyée (push) à votre registre. Là, vous utilisez [az acr run][az-acr-run] pour exécuter la commande de conteneur. Dans votre workflow de développement de conteneur, il peut s’agir d’une étape de validation avant le déploiement de l’image, ou vous pourriez inclure la commande dans un [fichier YAML multi-étape][container-registry-tasks-multi-step]. 

L’exemple suivant utilise `$Registry` pour spécifier le registre dans lequel vous exécutez la commande :

```azurecli-interactive
az acr run --registry myContainerRegistry008 \
  --cmd '$Registry/sample/hello-world:v1' /dev/null
```

Le paramètre `cmd` dans cet exemple exécute le conteneur dans sa configuration par défaut, mais `cmd` prend en charge des paramètres `docker run` supplémentaires ou même d’autres commandes `docker`.

Le résultat ressemble à ce qui suit :

```console
Packing source code into tar to upload...
Uploading archived source code from '/tmp/run_archive_ebf74da7fcb04683867b129e2ccad5e1.tar.gz'...
Sending context (1.855 KiB) to registry: mycontainerre...
Queued a run with ID: cab
Waiting for an agent...
2019/03/19 19:01:53 Using acb_vol_60e9a538-b466-475f-9565-80c5b93eaa15 as the home volume
2019/03/19 19:01:53 Creating Docker network: acb_default_network, driver: 'bridge'
2019/03/19 19:01:53 Successfully set up Docker network: acb_default_network
2019/03/19 19:01:53 Setting up Docker configuration...
2019/03/19 19:01:54 Successfully set up Docker configuration
2019/03/19 19:01:54 Logging in to registry: mycontainerregistry008.azurecr.io
2019/03/19 19:01:55 Successfully logged into mycontainerregistry008.azurecr.io
2019/03/19 19:01:55 Executing step ID: acb_step_0. Working directory: '', Network: 'acb_default_network'
2019/03/19 19:01:55 Launching container with name: acb_step_0

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

2019/03/19 19:01:56 Successfully executed container: acb_step_0
2019/03/19 19:01:56 Step ID: acb_step_0 marked as successful (elapsed time in seconds: 0.843801)

Run ID: cab was successful after 6s
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Vous pouvez utiliser la commande [az group delete][az-group-delete] pour supprimer le groupe de ressources, le registre de conteneurs et les images conteneur stockées à cet endroit quand vous n’en avez plus besoin.

```azurecli
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez utilisé des fonctionnalités d’ACR Tasks pour générer, envoyer (push) et exécuter rapidement une image conteneur Docker en mode natif dans Azure, sans installation Docker locale. Passez aux tutoriels Azure Container Registry Tasks pour savoir comment automatiser les mises à jour et les builds d’image avec ACR Tasks.

> [!div class="nextstepaction"]
> [Tutoriels Azure Container Registry Tasks][container-registry-tutorial-quick-task]

<!-- LINKS - external -->
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-pull]: https://docs.docker.com/engine/reference/commandline/pull/
[docker-rmi]: https://docs.docker.com/engine/reference/commandline/rmi/
[docker-run]: https://docs.docker.com/engine/reference/commandline/run/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[docker-windows]: https://docs.docker.com/docker-for-windows/
[azure-account]: https://azure.microsoft.com/free/

<!-- LINKS - internal -->
[az-acr-create]: /cli/azure/acr#az-acr-create
[az-acr-build]: /cli/azure/acr#az-acr-build
[az-acr-run]: /cli/azure/acr#az-acr-run
[az-group-create]: /cli/azure/group#az-group-create
[az-group-delete]: /cli/azure/group#az-group-delete
[azure-cli]: /cli/azure/install-azure-cli
[container-registry-tasks-overview]: container-registry-tasks-overview.md
[container-registry-tasks-multi-step]: container-registry-tasks-multi-step.md
[container-registry-tutorial-quick-task]: container-registry-tutorial-quick-task.md
[container-registry-skus]: container-registry-skus.md
[azure-cli-install]: /cli/azure/install-azure-cli
