---
title: Authentification multiregistre à partir d’une tâche ACR
description: Configurez une tâche Azure Container Registry (Tâche ACR) pour accéder à un autre registre de conteneurs Azure privé avec une identité managée pour les ressources Azure.
ms.topic: article
ms.date: 07/06/2020
ms.openlocfilehash: 9a460102eafa5c1eda2f37330887d985387d5df5
ms.sourcegitcommit: daab0491bbc05c43035a3693a96a451845ff193b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "93026256"
---
# <a name="cross-registry-authentication-in-an-acr-task-using-an-azure-managed-identity"></a>Authentification multiregistre dans une tâche ACR à l’aide d’une identité managée par Azure 

Dans une [tâche ACR](container-registry-tasks-overview.md), vous pouvez [activer une identité managée pour les ressources Azure](container-registry-tasks-authentication-managed-identity.md). La tâche peut utiliser l’identité pour accéder à d’autres ressources Azure, sans qu’il soit nécessaire de fournir ni de gérer des informations d’identification. 

Dans cet article, vous allez apprendre à activer une identité managée dans une tâche pour extraire une image à partir d’un registre différent de celui utilisé pour exécuter la tâche.

Pour créer les ressources Azure, il est nécessaire dans le cadre de cet article d’exécuter Azure CLI version 2.0.68 ou ultérieure. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI][azure-cli].

## <a name="scenario-overview"></a>Présentation du scénario

L’exemple de tâche extrait une image de base à partir d’un autre registre de conteneurs Azure pour générer et transmettre une image d’application. Pour extraire l’image de base, configurez la tâche avec une identité managée et attribuez-lui les autorisations appropriées. 

Cet exemple montre les étapes à effectuer à l’aide d’une identité managée affectée par l’utilisateur ou par le système. Votre choix d’identité dépend des besoins de votre organisation.

Dans un scénario réel, une organisation peut conserver un ensemble d’images de base utilisées par toutes les équipes de développement pour créer leurs applications. Ces images de base sont stockées dans un registre de l’entreprise, chaque équipe de développement ne disposant que de droits d’extraction. 

## <a name="prerequisites"></a>Prérequis

Pour cet article, vous avez besoin de deux registres de conteneurs Azure :

* Vous utilisez le premier registre pour créer et exécuter les tâches ACR. Dans cet article, ce registre est nommé *myregistry*. 
* Le deuxième Registre héberge une image de base utilisée pour la tâche afin de générer une image. Dans cet article, ce deuxième registre est nommé *mybaseregistry*. 

Remplacez-les par vos propres noms de registre dans les étapes suivantes.

Si vous n’avez pas encore les registres de conteneurs Azure requis, consultez [Démarrage rapide : Créer un registre de conteneurs privé avec Azure CLI](container-registry-get-started-azure-cli.md). Vous n’avez pas encore besoin d’envoyer (push) des images au registre.

## <a name="prepare-base-registry"></a>Préparer le registre de base

À des fins de démonstration, dans le cadre d’une opération unique, exécutez [az acr import][az-acr-import] pour importer une image Node.js publique de Docker Hub vers votre registre de base. Dans la pratique, une autre équipe ou un autre processus de l’organisation peut conserver des images dans le registre de base.

```azurecli
az acr import --name mybaseregistry \
  --source docker.io/library/node:9-alpine \
  --image baseimages/node:9-alpine 
```

## <a name="define-task-steps-in-yaml-file"></a>Définir des étapes de tâche dans le fichier YAML

Les étapes de cet exemple de [tâche multiétapes](container-registry-tasks-multi-step.md) sont définies dans un [fichier YAML](container-registry-tasks-reference-yaml.md). Créez un fichier nommé `helloworldtask.yaml` dans votre répertoire de travail local et collez le contenu suivant. Mettez à jour la valeur `REGISTRY_NAME` dans l’étape de génération avec le nom du serveur de votre registre de base.

```yml
version: v1.1.0
steps:
# Replace mybaseregistry with the name of your registry containing the base image
  - build: -t $Registry/hello-world:$ID  https://github.com/Azure-Samples/acr-build-helloworld-node.git -f Dockerfile-app --build-arg REGISTRY_NAME=mybaseregistry.azurecr.io
  - push: ["$Registry/hello-world:$ID"]
```

L’étape de génération utilise le fichier `Dockerfile-app` dans le référentiel [Azure-Samples/acr-build-helloworld-node](https://github.com/Azure-Samples/acr-build-helloworld-node.git) pour générer une image. La commande `--build-arg` référence le registre de base pour extraire l’image de base. Une fois la génération réussie, l’image est envoyée au registre utilisé pour exécuter la tâche.

## <a name="option-1-create-task-with-user-assigned-identity"></a>Option 1 : Créer une tâche avec une identité affectée par l’utilisateur

Les étapes de cette section créent une tâche et activent une identité affectée par l’utilisateur. Si vous souhaitez activer une identité affectée par le système à la place, consultez l’[option 2 : Créer une tâche avec une identité affectée par le système](#option-2-create-task-with-system-assigned-identity). 

[!INCLUDE [container-registry-tasks-user-assigned-id](../../includes/container-registry-tasks-user-assigned-id.md)]

### <a name="create-task"></a>Créer une tâche

Créez la tâche *helloworldtask* en exécutant la commande [az acr task create][az-acr-task-create] suivante. La tâche s’exécute sans contexte de code source et la commande référence le fichier `helloworldtask.yaml` dans le répertoire de travail. Le paramètre `--assign-identity` transmet l’ID de ressource de l’identité affectée par l’utilisateur. 

```azurecli
az acr task create \
  --registry myregistry \
  --name helloworldtask \
  --context /dev/null \
  --file helloworldtask.yaml \
  --assign-identity $resourceID
```

[!INCLUDE [container-registry-tasks-user-id-properties](../../includes/container-registry-tasks-user-id-properties.md)]

### <a name="give-identity-pull-permissions-to-the-base-registry"></a>Accorder des autorisations d’extraction d’identité au registre de base

Dans cette section, vous allez accorder à l’identité gérée les autorisations d’extraction à partirdu registre de base, *mybaseregistry*.

Utilisez la commande [az acr show][az-acr-show] pour obtenir l’ID de ressource du registre de base et le stocker dans une variable :

```azurecli
baseregID=$(az acr show --name mybaseregistry --query id --output tsv)
```

Utilisez la commande [az role assignment create][az-role-assignment-create] pour affecter à l’identité le rôle `acrpull` à chaque registre de base. Ce rôle dispose uniquement des autorisations pour extraire des images du registre.

```azurecli
az role assignment create \
  --assignee $principalID \
  --scope $baseregID \
  --role acrpull
```

Procédez à [Ajouter des informations d’identification de registre cible à la tâche](#add-target-registry-credentials-to-task).

## <a name="option-2-create-task-with-system-assigned-identity"></a>Option n°2 : Créer une tâche avec une identité affectée par le système

Les étapes de cette section créent une tâche et activent une identité affectée par le système. Si vous souhaitez activer une identité affectée par l’utilisateur à la place, consultez l’[Option 1 : Créer une tâche avec une identité affectée par l’utilisateur](#option-1-create-task-with-user-assigned-identity). 

### <a name="create-task"></a>Créer une tâche

Créez la tâche *helloworldtask* en exécutant la commande [az acr task create][az-acr-task-create] suivante. La tâche s’exécute sans contexte de code source et la commande référence le fichier `helloworldtask.yaml` dans le répertoire de travail. Le paramètre `--assign-identity`, sans valeur, active une identité managée affectée par le système sur la tâche. 

```azurecli
az acr task create \
  --registry myregistry \
  --name helloworldtask \
  --context /dev/null \
  --file helloworldtask.yaml \
  --assign-identity 
```
[!INCLUDE [container-registry-tasks-system-id-properties](../../includes/container-registry-tasks-system-id-properties.md)]

### <a name="give-identity-pull-permissions-to-the-base-registry"></a>Accorder des autorisations d’extraction d’identité au registre de base

Dans cette section, vous allez accorder à l’identité gérée les autorisations d’extraction à partirdu registre de base, *mybaseregistry*.

Utilisez la commande [az acr show][az-acr-show] pour obtenir l’ID de ressource du registre de base et le stocker dans une variable :

```azurecli
baseregID=$(az acr show --name mybaseregistry --query id --output tsv)
```

Utilisez la commande [az role assignment create][az-role-assignment-create] pour affecter à l’identité le rôle `acrpull` à chaque registre de base. Ce rôle dispose uniquement des autorisations pour extraire des images du registre.

```azurecli
az role assignment create \
  --assignee $principalID \
  --scope $baseregID \
  --role acrpull
```

## <a name="add-target-registry-credentials-to-task"></a>Ajouter des informations d’identification de registre cible à la tâche

Utilisez maintenant la commande [az acr task credential add][az-acr-task-credential-add] pour permettre à la tâche de s’authentifier auprès du registre de base avec les informations d’identification de l’identité. Exécutez la commande correspondant au type d’identité gérée que vous avez activé dans la tâche. Si vous avez activé une identité affectée par l’utilisateur `--use-identity`, transmettez-la avec l’ID client de l’identité. Si vous avez activé une identité affectée par le système, transmettez la commande `--use-identity [system]`.

```azurecli
# Add credentials for user-assigned identity to the task
az acr task credential add \
  --name helloworldtask \
  --registry myregistry \
  --login-server mybaseregistry.azurecr.io \
  --use-identity $clientID

# Add credentials for system-assigned identity to the task
az acr task credential add \
  --name helloworldtask \
  --registry myregistry \
  --login-server mybaseregistry.azurecr.io \
  --use-identity [system]
```

## <a name="manually-run-the-task"></a>Exécuter manuellement la tâche

Pour vérifier que la tâche dans laquelle vous avez activé une identité managée s’exécute correctement, déclenchez manuellement la tâche à l’aide de la commande [az acr task run][az-acr-task-run]. 

```azurecli
az acr task run \
  --name helloworldtask \
  --registry myregistry
```

Si la tâche s’exécute correctement, vous devez obtenir le résultat suivant :

```
Queued a run with ID: cf10
Waiting for an agent...
2019/06/14 22:47:32 Using acb_vol_dbfbe232-fd76-4ca3-bd4a-687e84cb4ce2 as the home volume
2019/06/14 22:47:39 Creating Docker network: acb_default_network, driver: 'bridge'
2019/06/14 22:47:40 Successfully set up Docker network: acb_default_network
2019/06/14 22:47:40 Setting up Docker configuration...
2019/06/14 22:47:41 Successfully set up Docker configuration
2019/06/14 22:47:41 Logging in to registry: myregistry.azurecr.io
2019/06/14 22:47:42 Successfully logged into myregistry.azurecr.io
2019/06/14 22:47:42 Logging in to registry: mybaseregistry.azurecr.io
2019/06/14 22:47:43 Successfully logged into mybaseregistry.azurecr.io
2019/06/14 22:47:43 Executing step ID: acb_step_0. Timeout(sec): 600, Working directory: '', Network: 'acb_default_network'
2019/06/14 22:47:43 Scanning for dependencies...
2019/06/14 22:47:45 Successfully scanned dependencies
2019/06/14 22:47:45 Launching container with name: acb_step_0
Sending build context to Docker daemon   25.6kB
Step 1/6 : ARG REGISTRY_NAME
Step 2/6 : FROM ${REGISTRY_NAME}/baseimages/node:9-alpine
9-alpine: Pulling from baseimages/node
[...]
Successfully built 41b49a112663
Successfully tagged myregistry.azurecr.io/hello-world:cf10
2019/06/14 22:47:56 Successfully executed container: acb_step_0
2019/06/14 22:47:56 Executing step ID: acb_step_1. Timeout(sec): 600, Working directory: '', Network: 'acb_default_network'
2019/06/14 22:47:56 Pushing image: myregistry.azurecr.io/hello-world:cf10, attempt 1
The push refers to repository [myregistry.azurecr.io/hello-world]
[...]
2019/06/14 22:48:00 Step ID: acb_step_1 marked as successful (elapsed time in seconds: 2.517011)
2019/06/14 22:48:00 The following dependencies were found:
2019/06/14 22:48:00
- image:
    registry: myregistry.azurecr.io
    repository: hello-world
    tag: cf10
    digest: sha256:611cf6e3ae3cb99b23fadcd89fa144e18aa1b1c9171ad4a0da4b62b31b4e38d1
  runtime-dependency:
    registry: mybaseregistry.azurecr.io
    repository: baseimages/node
    tag: 9-alpine
    digest: sha256:e8e92cffd464fce3be9a3eefd1b65dc9cbe2484da31c11e813a4effc6105c00f
  git:
    git-head-revision: 0f988779c97fe0bfc7f2f74b88531617f4421643

Run ID: cf10 was successful after 32s
```

Exécutez la commande [az acr repository show-tags][az-acr-repository-show-tags] pour vérifier que l’image générée a été correctement transmise vers *myregistry*  :

```azurecli
az acr repository show-tags --name myregistry --repository hello-world --output tsv
```

Exemple de sortie :

```console
cf10
```

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur l’[activation d’une identité gérée dans une tâche ACR](container-registry-tasks-authentication-managed-identity.md).
* Consultez la [référence YAML ACR Tasks](container-registry-tasks-reference-yaml.md)

<!-- LINKS - Internal -->
[az-login]: /cli/azure/reference-index#az-login
[az-acr-login]: /cli/azure/acr#az-acr-login
[az-acr-show]: /cli/azure/acr#az-acr-show
[az-acr-build]: /cli/azure/acr#az-acr-build
[az-acr-repository-show-tags]: /cli/azure/acr/repository#az-acr-repository-show-tags
[az-role-assignment-create]: /cli/azure/role/assignment#az-role-assignment-create
[az-acr-login]: /cli/azure/acr#az-acr-login
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-task-create]: /cli/azure/acr/task#az-acr-task-create
[az-acr-task-show]: /cli/azure/acr/task#az-acr-task-show
[az-acr-task-run]: /cli/azure/acr/task#az-acr-task-run
[az-acr-task-list-runs]: /cli/azure/acr/task#az-acr-task-list-runs
[az-acr-task-credential-add]: /cli/azure/acr/task/credential#az-acr-task-credential-add
[az-group-create]: /cli/azure/group?#az-group-create
