---
title: Tutoriel - Générer une image lors de la validation du code
description: Dans ce didacticiel, vous allez découvrir comment configurer une tâche Azure Container Registry Task pour déclencher automatiquement la génération des images de conteneur dans le cloud lorsque vous validez le code source dans un référentiel Git.
ms.topic: tutorial
ms.date: 11/24/2020
ms.custom: seodec18, mvc, devx-track-azurecli
ms.openlocfilehash: 7f2e6d7f304977d3e6d92a778dba5bf026343707
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97562903"
---
# <a name="tutorial-automate-container-image-builds-in-the-cloud-when-you-commit-source-code"></a>Tutoriel : automatiser la génération des images de conteneur dans le cloud lorsque vous validez le code source

En plus d'une [tâche rapide](container-registry-tutorial-quick-task.md), ACR Tasks prend en charge la génération d'images de conteneur Docker dans le cloud lorsque vous validez le code source dans un référentiel Git. Les contextes Git pris en charge par ACR Tasks incluent les dépôts GitHub publics ou privés et Azure Repos.

> [!NOTE]
> Actuellement, ACR Tasks ne prend pas en charge les déclencheurs de validation et de demande de tirage (pull request) dans les référentiels GitHub Enterprise.

Dans ce tutoriel, votre tâche ACR génère et envoie (push) une seule image conteneur spécifiée dans un Dockerfile lorsque vous validez le code source dans un référentiel Git. Pour créer une [tâche à plusieurs étapes](container-registry-tasks-multi-step.md) utilisant un fichier YAML pour définir les étapes permettant de créer, d’envoyer (push) et éventuellement de tester plusieurs conteneurs lors de la validation du code, consultez [Tutoriel : Exécuter un workflow de conteneur à plusieurs étapes dans le cloud lorsque vous validez le code source](container-registry-tutorial-multistep-task.md). Pour une vue d’ensemble d’ACR Tasks, consultez [Automatiser les mises à jour correctives du système d’exploitation et de l'infrastructure avec ACR Tasks](container-registry-tasks-overview.md).

Dans ce tutoriel, vous allez :

> [!div class="checklist"]
> * Créer une tâche
> * Tester la tâche
> * Affichage de l’état de la tâche
> * Déclencher la tâche avec une validation de code

Pour suivre ce didacticiel, vous devez avoir déjà effectué les étapes du [didacticiel précédent](container-registry-tutorial-quick-task.md). Si ce n’est déjà fait, effectuez les étapes de la section [Conditions préalables](container-registry-tutorial-quick-task.md#prerequisites) du didacticiel précédent avant de continuer.

[!INCLUDE [container-registry-task-tutorial-prereq.md](../../includes/container-registry-task-tutorial-prereq.md)]
[!INCLUDE [azure-cli-prepare-your-environment-h3.md](../../includes/azure-cli-prepare-your-environment-h3.md)]

## <a name="create-the-build-task"></a>Créer la tâche de génération

Maintenant que vous avez terminé les étapes requises pour permettre à ACR Tasks de lire l’état de validation et de créer des webhooks dans un référentiel, vous pouvez créer une tâche qui déclenche une génération d’image conteneur lors de la validation dans le référentiel.

Tout d’abord, renseignez ces variables d’environnement shell avec des valeurs appropriées pour votre environnement. Cette étape n’est pas strictement obligatoire, mais facilite un peu l’exécution des commandes multilignes de l’interface de ligne de commande Azure de ce didacticiel. Si vous ne renseignez pas ces variables d’environnement, vous devrez remplacer manuellement chaque valeur à chaque fois qu’elle apparaît dans les exemples de commandes.

```console
ACR_NAME=<registry-name>        # The name of your Azure container registry
GIT_USER=<github-username>      # Your GitHub user account name
GIT_PAT=<personal-access-token> # The PAT you generated in the previous section
```

Créez maintenant la tâche en exécutant la commande [az acr task create][az-acr-task-create] suivante :

```azurecli
az acr task create \
    --registry $ACR_NAME \
    --name taskhelloworld \
    --image helloworld:{{.Run.ID}} \
    --context https://github.com/$GIT_USER/acr-build-helloworld-node.git \
    --file Dockerfile \
    --git-access-token $GIT_PAT
```


Cette tâche spécifie que tout code temporel est validé dans la branche *primaire* du dépôt spécifié par `--context` et qu’ACR Tasks va générer l’image conteneur à partir du code de cette branche. Le Dockerfile spécifié par `--file` à la racine du dépôt est utilisé pour générer l’image. L’argument `--image` spécifie une valeur paramétrable de `{{.Run.ID}}` pour la partie « version » de la balise de l’image, garantissant ainsi que l’image générée est en corrélation avec une build spécifique et qu’elle est référencée de façon unique.

La sortie d’une commande [az acr task create][az-acr-task-create] réussie se présente ainsi :

```output
{
  "agentConfiguration": {
    "cpu": 2
  },
  "creationDate": "2010-11-19T22:42:32.972298+00:00",
  "id": "/subscriptions/<Subscription ID>/resourceGroups/myregistry/providers/Microsoft.ContainerRegistry/registries/myregistry/tasks/taskhelloworld",
  "location": "westcentralus",
  "name": "taskhelloworld",
  "platform": {
    "architecture": "amd64",
    "os": "Linux",
    "variant": null
  },
  "provisioningState": "Succeeded",
  "resourceGroup": "myregistry",
  "status": "Enabled",
  "step": {
    "arguments": [],
    "baseImageDependencies": null,
    "contextPath": "https://github.com/gituser/acr-build-helloworld-node",
    "dockerFilePath": "Dockerfile",
    "imageNames": [
      "helloworld:{{.Run.ID}}"
    ],
    "isPushEnabled": true,
    "noCache": false,
    "type": "Docker"
  },
  "tags": null,
  "timeout": 3600,
  "trigger": {
    "baseImageTrigger": {
      "baseImageTriggerType": "Runtime",
      "name": "defaultBaseimageTriggerName",
      "status": "Enabled"
    },
    "sourceTriggers": [
      {
        "name": "defaultSourceTriggerName",
        "sourceRepository": {
          "branch": "main",
          "repositoryUrl": "https://github.com/gituser/acr-build-helloworld-node",
          "sourceControlAuthProperties": null,
          "sourceControlType": "GitHub"
        },
        "sourceTriggerEvents": [
          "commit"
        ],
        "status": "Enabled"
      }
    ]
  },
  "type": "Microsoft.ContainerRegistry/registries/tasks"
}
```

## <a name="test-the-build-task"></a>Tester la tâche de génération

Vous disposez maintenant d’une tâche qui définit votre build. Pour tester le pipeline de build, déclenchez une génération manuellement en exécutant la commande [az acr task run][az-acr-task-run] :

```azurecli
az acr task run --registry $ACR_NAME --name taskhelloworld
```

Par défaut, la commande `az acr task run` diffuse en continu la sortie du journal vers votre console lorsque vous exécutez la commande. La sortie est condensée pour montrer les principales étapes.

```output
2020/11/19 22:51:00 Using acb_vol_9ee1f28c-4fd4-43c8-a651-f0ed027bbf0e as the home volume
2020/11/19 22:51:00 Setting up Docker configuration...
2020/11/19 22:51:02 Successfully set up Docker configuration
2020/11/19 22:51:02 Logging in to registry: myregistry.azurecr.io
2020/11/19 22:51:03 Successfully logged in
2020/11/19 22:51:03 Executing step: build
2020/11/19 22:51:03 Obtaining source code and scanning for dependencies...
2020/11/19 22:51:05 Successfully obtained source code and scanned for dependencies
Sending build context to Docker daemon  23.04kB
Step 1/5 : FROM node:15-alpine
[...]
Step 5/5 : CMD ["node", "/src/server.js"]
 ---> Running in 7382eea2a56a
Removing intermediate container 7382eea2a56a
 ---> e33cd684027b
Successfully built e33cd684027b
Successfully tagged myregistry.azurecr.io/helloworld:da2
2020/11/19 22:51:11 Executing step: push
2020/11/19 22:51:11 Pushing image: myregistry.azurecr.io/helloworld:da2, attempt 1
The push refers to repository [myregistry.azurecr.io/helloworld]
4a853682c993: Preparing
[...]
4a853682c993: Pushed
[...]
da2: digest: sha256:c24e62fd848544a5a87f06ea60109dbef9624d03b1124bfe03e1d2c11fd62419 size: 1366
2020/11/19 22:51:21 Successfully pushed image: myregistry.azurecr.io/helloworld:da2
2020/11/19 22:51:21 Step id: build marked as successful (elapsed time in seconds: 7.198937)
2020/11/19 22:51:21 Populating digests for step id: build...
2020/11/19 22:51:22 Successfully populated digests for step id: build
2020/11/19 22:51:22 Step id: push marked as successful (elapsed time in seconds: 10.180456)
The following dependencies were found:
- image:
    registry: myregistry.azurecr.io
    repository: helloworld
    tag: da2
    digest: sha256:c24e62fd848544a5a87f06ea60109dbef9624d03b1124bfe03e1d2c11fd62419
  runtime-dependency:
    registry: registry.hub.docker.com
    repository: library/node
    tag: 9-alpine
    digest: sha256:8dafc0968fb4d62834d9b826d85a8feecc69bd72cd51723c62c7db67c6dec6fa
  git:
    git-head-revision: 68cdf2a37cdae0873b8e2f1c4d80ca60541029bf


Run ID: ca6 was successful after 27s
```

## <a name="trigger-a-build-with-a-commit"></a>Déclencher une génération avec une validation

Maintenant que vous avez testé la tâche en l’exécutant manuellement, déclenchez-la automatiquement avec une modification du code source.

Vérifiez d’abord que vous êtes dans répertoire contenant votre clone local du [dépôt][sample-repo] :

```console
cd acr-build-helloworld-node
```

Exécutez alors les commandes suivantes pour créer, valider et envoyer un nouveau fichier vers votre duplication du référentiel sur GitHub :

```console
echo "Hello World!" > hello.txt
git add hello.txt
git commit -m "Testing ACR Tasks"
git push origin main
```

Vous pouvez être invité à fournir vos informations d’identification GitHub lorsque vous exécutez la commande `git push`. Indiquez votre nom d’utilisateur GitHub et entrez le jeton d’accès personnel (PAT) que vous avez créé précédemment pour le mot de passe.

```azurecli
Username for 'https://github.com': <github-username>
Password for 'https://githubuser@github.com': <personal-access-token>
```

Une fois que vous avez envoyé une validation vers votre référentiel, le webhook créé par ACR Tasks déclenche et démarre une génération dans Azure Container Registry. Affichez les journaux d’activité de la tâche en cours d’exécution pour vérifier et surveiller l’avancement de la génération :

```azurecli
az acr task logs --registry $ACR_NAME
```

La sortie est similaire à ce qui suit, affichant la tâche en cours d’exécution (ou exécutée en dernier lieu) :

```output
Showing logs of the last created run.
Run ID: ca7

[...]

Run ID: ca7 was successful after 38s
```

## <a name="list-builds"></a>Répertorier les générations

Pour afficher la liste des exécutions de tâche effectuées par ACR Tasks pour votre registre, exécutez la commande [az acr task list-runs][az-acr-task-list-runs] :

```azurecli
az acr task list-runs --registry $ACR_NAME --output table
```

La sortie de la commande doit ressembler à ceci : Les exécutions d’ACR Tasks sont affichées, et « Validation Git » s’affiche dans la colonne DÉCLENCHEUR de la dernière tâche :

```output
RUN ID    TASK            PLATFORM    STATUS     TRIGGER    STARTED               DURATION
--------  --------------  ----------  ---------  ---------  --------------------  ----------
ca7       taskhelloworld  linux       Succeeded  Commit     2020-11-19T22:54:34Z  00:00:29
ca6       taskhelloworld  linux       Succeeded  Manual     2020-11-19T22:51:47Z  00:00:24
ca5                       linux       Succeeded  Manual     2020-11-19T22:23:42Z  00:00:23
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez découvert comment utiliser une tâche pour déclencher automatiquement la génération des images de conteneur dans Azure lorsque vous validez le code source dans un référentiel Git. Passez au didacticiel suivant pour découvrir comment créer des tâches qui déclenchent des générations lorsqu’une image de base d’une image conteneur est mise à jour.

> [!div class="nextstepaction"]
> [Automatiser les générations en fonction de la mise à jour d’une image de base](container-registry-tutorial-base-image-update.md)

<!-- LINKS - External -->
[sample-repo]: https://github.com/Azure-Samples/acr-build-helloworld-node

<!-- LINKS - Internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-task]: /cli/azure/acr/task
[az-acr-task-create]: /cli/azure/acr/task#az-acr-task-create
[az-acr-task-run]: /cli/azure/acr/task#az-acr-task-run
[az-acr-task-list-runs]: /cli/azure/acr/task#az-acr-task-list-runs
[az-login]: /cli/azure/reference-index#az-login



