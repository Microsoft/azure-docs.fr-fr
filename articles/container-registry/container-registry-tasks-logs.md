---
title: Afficher les journaux d’exécution des tâches - Tâches
description: Guide pratique pour afficher et gérer les journaux d’exécution générés par les tâches ACR.
ms.topic: article
ms.date: 03/09/2020
ms.openlocfilehash: b2a10d4a3a2746acf38445673af994c6317c77de
ms.sourcegitcommit: daab0491bbc05c43035a3693a96a451845ff193b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "93027174"
---
# <a name="view-and-manage-task-run-logs"></a>Afficher et gérer les journaux d’exécution des tâches

Chaque exécution de tâche dans [Azure Container Registry](container-registry-tasks-overview.md) génère une sortie de journal que vous pouvez inspecter pour déterminer si les étapes de la tâche ont été exécutées avec succès. 

Cet article explique comment afficher et gérer des journaux d’exécution des tâches.

## <a name="view-streamed-logs"></a>Afficher les journaux diffusés en continu

Quand vous déclenchez une tâche manuellement, la sortie du journal est diffusée directement sur la console. Par exemple, lorsque vous déclenchez une tâche manuellement à l’aide de la commande [az acr build](/cli/azure/acr#az-acr-build), [az acr run](/cli/azure/acr#az-acr-run) ou [az acr task run](/cli/azure/acr/task#az-acr-task-run), la sortie de journal est diffusée en continu sur la console. 

L’exemple de commande [az acr run](/cli/azure/acr#az-acr-run) suivant déclenche manuellement une tâche qui exécute un conteneur extrait du même registre :

```azurecli
az acr run --registry mycontainerregistry1220 \
  --cmd '$Registry/samples/hello-world:v1' /dev/null
```

Journal diffusé en continu :

```console
Queued a run with ID: cf4
Waiting for an agent...
2020/03/09 20:30:10 Alias support enabled for version >= 1.1.0, please see https://aka.ms/acr/tasks/task-aliases for more information.
2020/03/09 20:30:10 Creating Docker network: acb_default_network, driver: 'bridge'
2020/03/09 20:30:10 Successfully set up Docker network: acb_default_network
2020/03/09 20:30:10 Setting up Docker configuration...
2020/03/09 20:30:11 Successfully set up Docker configuration
2020/03/09 20:30:11 Logging in to registry: mycontainerregistry1220azurecr.io
2020/03/09 20:30:12 Successfully logged into mycontainerregistry1220azurecr.io
2020/03/09 20:30:12 Executing step ID: acb_step_0. Timeout(sec): 600, Working directory: '', Network: 'acb_default_network'
2020/03/09 20:30:12 Launching container with name: acb_step_0
Unable to find image 'mycontainerregistry1220azurecr.io/samples/hello-world:v1' locally
v1: Pulling from samples/hello-world
Digest: sha256:92c7f9c92844bbbb5d0a101b22f7c2a7949e40f8ea90c8b3bc396879d95e888a
Status: Downloaded newer image for mycontainerregistry1220azurecr.io/samples/hello-world:v1

Hello from Docker!
This message shows that your installation appears to be working correctly.
[...]

2020/03/09 20:30:13 Successfully executed container: acb_step_0
2020/03/09 20:30:13 Step ID: acb_step_0 marked as successful (elapsed time in seconds: 1.180081)

Run ID: cf4 was successful after 5s
```

## <a name="view-stored-logs"></a>Afficher les journaux stockés 

Azure Container Registry stocke les journaux d’exécution de toutes les tâches. Vous pouvez afficher les journaux d’exécution stockés dans le portail Azure. Ou utilisez la commande [az acr task logs](/cli/azure/acr/task#az-acr-task-logs) pour afficher un journal sélectionné. Par défaut, les journaux d’activité sont conservés 30 jours.

Si une tâche est déclenchée automatiquement, par exemple par une mise à jour du code source, l’accès aux journaux stockés est l’ *unique* moyen d’afficher les journaux d’exécution. Les déclencheurs de tâches automatiques incluent les validations de code source ou les requêtes de tirage, les mises à jour d’image de base et les déclencheurs de minuteur.

Pour afficher les journaux d’exécution dans le portail :

1. Accédez à votre registre de conteneurs.
1. Dans **Services** , sélectionnez **Tâches** > **Exécutions**.
1. Sélectionnez un **ID d’exécution** pour afficher l’état d’exécution et les journaux d’exécution. Le journal contient les mêmes informations qu’un journal diffusé en continu, le cas échéant.

![Afficher l’exécution d’une tâche dans le portail de connexion](./media/container-registry-tasks-logs/portal-task-run-logs.png)

Pour afficher un journal à l’aide de l’interface de ligne de commande Azure, exécutez [az acr task logs](/cli/azure/acr/task#az-acr-task-logs), puis spécifiez un ID d’exécution, un nom de tâche ou une image spécifique créée par une tâche de génération. Si un nom de tâche est spécifié, la commande affiche le journal de la dernière exécution créée.

L’exemple suivant génère le journal pour l’exécution dont l’ID est *CF4*  :

```azurecli
az acr task logs --registry mycontainerregistry1220 \
  --run-id cf4
```

## <a name="alternative-log-storage"></a>Autre stockage de journal

Vous souhaiterez peut-être stocker les journaux d’exécution des tâches sur un système de fichiers local, ou utiliser une autre solution d’archivage telle que le Stockage Azure.

Par exemple, créez un répertoire local *tasklogs* et redirigez la sortie de la commande [az acr task logs](/cli/azure/acr/task#az-acr-task-logs) vers un fichier local :

```azurecli
mkdir ~/tasklogs

az acr task logs --registry mycontainerregistry1220 \
  --run-id cf4 > ~/tasklogs/cf4.log
```

Vous pouvez également enregistrer les fichiers journaux locaux dans le Stockage Azure. Par exemple, utilisez l’[interface de ligne de commande Azure](../storage/blobs/storage-quickstart-blobs-cli.md), le [portail Azure](../storage/blobs/storage-quickstart-blobs-portal.md) ou d’autres méthodes pour charger des fichiers dans un compte de stockage.

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les [tâches Azure Container Registry](container-registry-tasks-overview.md)


<!-- LINKS - Internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-build]: /cli/azure/acr#az-acr-build
[az-acr-pack-build]: /cli/azure/acr/pack#az-acr-pack-build
[az-acr-task]: /cli/azure/acr/task
[az-acr-task-create]: /cli/azure/acr/task#az-acr-task-create
[az-acr-task-run]: /cli/azure/acr/task#az-acr-task-run
[az-acr-task-update]: /cli/azure/acr/task#az-acr-task-update
[az-login]: /cli/azure/reference-index#az-login
[az-login-service-principal]: /cli/azure/authenticate-azure-cli

<!-- IMAGES -->
[quick-build-01-fork]: ./media/container-registry-tutorial-quick-build/quick-build-01-fork.png
[quick-build-02-browser]: ./media/container-registry-tutorial-quick-build/quick-build-02-browser.png
