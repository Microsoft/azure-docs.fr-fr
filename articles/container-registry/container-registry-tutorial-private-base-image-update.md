---
title: Tutoriel – Déclencher la génération d’images par mise à jour d’images de base privées
description: Dans ce tutoriel, vous allez configurer une tâche Azure Container Registry pour déclencher automatiquement la génération d’images conteneur dans le cloud lorsqu’une image de base est mise à jour dans un autre registre de conteneurs Azure privé.
ms.topic: tutorial
ms.date: 11/20/2020
ms.custom: devx-track-js, devx-track-azurecli
ms.openlocfilehash: 50eb89ccfafa27a7dcb0e97f21d14feec0ef9525
ms.sourcegitcommit: 2e9643d74eb9e1357bc7c6b2bca14dbdd9faa436
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96030434"
---
# <a name="tutorial-automate-container-image-builds-when-a-base-image-is-updated-in-another-private-container-registry"></a>Tutoriel : Automatiser la génération des images conteneur quand une image de base est mise à jour dans un autre registre de conteneurs privé 

[ACR Tasks](container-registry-tasks-overview.md) prend en charge la génération automatisée d’images quand l’[image de base d’un conteneur est mise à jour](container-registry-tasks-base-images.md), par exemple quand vous corrigez le système d’exploitation ou l’infrastructure d’application dans l’une de vos images de base. 

Dans ce tutoriel, vous allez découvrir comment créer une tâche ACR qui déclenche une génération dans le cloud lorsque l’image de base d’un conteneur est envoyée (push) vers un autre registre de conteneurs Azure. Vous pouvez également essayer de suivre un tutoriel pour créer une tâche ACR qui déclenche une génération d’image quand une image de base est envoyée (push) au [même registre de conteneurs Azure](container-registry-tutorial-base-image-update.md).

Dans ce tutoriel, vous allez :

> [!div class="checklist"]
> * Générer l’image de base dans un registre de base
> * Créer une tâche de génération d’application dans un autre registre pour suivre l’image de base 
> * Mettre à jour l’image de base pour déclencher la génération d’une tâche d’image d’application
> * Afficher la tâche déclenchée
> * Vérifier l’image d’application mise à jour

## <a name="prerequisites"></a>Prérequis

### <a name="complete-the-previous-tutorials"></a>Terminer les didacticiels précédents

Pour suivre ce tutoriel, vous devez avoir déjà configuré votre environnement et effectué les étapes des deux premiers tutoriels de la série, dans lesquels vous avez appris à :

* Créer un registre de conteneurs Azure
* Dupliquer l’exemple de référentiel
* Cloner l’exemple de référentiel
* Créer un jeton d’accès personnel GitHub

Si vous ne l’avez pas encore fait, suivez les tutoriels ci-après avant de continuer :

[Générer des images de conteneur dans le cloud avec Azure Container Registry Tasks](container-registry-tutorial-quick-task.md)

[Automatiser la génération des images de conteneur avec Azure Container Registry Tasks](container-registry-tutorial-build-task.md)

En plus du registre de conteneurs créé pour les tutoriels précédents, vous devez créer un registre pour stocker les images de base. Si vous le souhaitez, créez le deuxième registre à un autre emplacement que le registre d’origine.

### <a name="configure-the-environment"></a>Configurer l’environnement

Renseignez ces variables d’environnement shell avec des valeurs appropriées pour votre environnement. Cette étape n’est pas strictement obligatoire, mais facilite un peu l’exécution des commandes multilignes de l’interface de ligne de commande Azure de ce didacticiel. Si vous ne renseignez pas ces variables d’environnement, vous devrez remplacer manuellement chaque valeur à chaque fois qu’elle apparaît dans les exemples de commandes.

```azurecli
BASE_ACR=<base-registry-name>   # The name of your Azure container registry for base images
ACR_NAME=<registry-name>        # The name of your Azure container registry for application images
GIT_USER=<github-username>      # Your GitHub user account name
GIT_PAT=<personal-access-token> # The PAT you generated in the second tutorial
```

### <a name="base-image-update-scenario"></a>Scénario de mise à jour d’une image de base

Ce didacticiel vous guide dans un scénario de mise à jour d’une image de base. Ce scénario reflète un flux de travail de développement pour gérer les images de base dans un registre de conteneurs privé commun lors de la création d’images d’application dans d’autres registres. Les images de base peuvent spécifier des systèmes d’exploitation et des infrastructures communs utilisés par une équipe, ou même des composants de service communs.

Par exemple, les développeurs qui développent des images d’application dans leurs propres registres peuvent accéder à un ensemble d’images de base conservées dans le registre de base commun. Le registre de base peut être dans une autre région, voire même géorépliqué.

[L’exemple de code][code-sample] comprend deux fichiers Dockerfile : une image d’application et une image spécifiée comme image de base. Dans les sections suivantes, vous allez créer une tâche ACR qui déclenche automatiquement une génération de l’image d’application lorsqu’une nouvelle version de l’image de base est envoyée (push) vers un autre registre de conteneurs Azure.

* [Dockerfile-app][dockerfile-app] : Une petite application web Node.js qui restitue une page web statique en affichant la version de Node.js sur laquelle elle est basée. La chaîne de version est simulée : elle affiche le contenu d’une variable d’environnement, `NODE_VERSION`, qui est définie dans l’image de base.

* [Dockerfile-base][dockerfile-base] : Image spécifiée par `Dockerfile-app` comme son image de base. Elle est elle-même basée sur une image [Nœud][base-node] et inclut la variable d’environnement `NODE_VERSION`.

Dans les sections suivantes, vous allez créer une tâche, mettre à jour la valeur `NODE_VERSION` dans le fichier Dockerfile d’image de base, puis utiliser ACR Tasks pour générer l’image de base. Lorsqu’ACR Tasks envoie la nouvelle image de base dans le registre, il déclenche automatiquement une génération de l’image de l’application. Si vous le souhaitez, vous pouvez exécuter l’image de conteneur d’application en local pour voir les différentes chaînes de version dans les images générées.

Dans ce tutoriel, votre tâche ACR génère et envoie (push) une image conteneur d’application spécifiée dans un Dockerfile. ACR Tasks peut également exécuter des [tâches multiétapes](container-registry-tasks-multi-step.md) à l’aide d’un fichier YAML pour définir les étapes permettant de générer, d’envoyer en mode push et éventuellement de tester plusieurs conteneurs.

## <a name="build-the-base-image"></a>Générer l’image de base

Commencez par générer l’image de base avec une *tâche rapide* d’ACR Tasks, en utilisant [az acr build][az-acr-build]. Comme indiqué dans le [premier didacticiel](container-registry-tutorial-quick-task.md) de la série, ce processus génère non seulement l’image, mais l’envoie vers le registre de conteneurs si la génération est réussie. Dans cet exemple, l’image est envoyée en mode push dans le registre d’images de base.

```azurecli
az acr build --registry $BASE_ACR --image baseimages/node:15-alpine --file Dockerfile-base .
```

## <a name="create-a-task-to-track-the-private-base-image"></a>Créer une tâche pour effectuer le suivi de l’image de base privée

Ensuite, créez une tâche dans le registre d’images d’application avec [az acr task create][az-acr-task-create], en activant une [identité managée](container-registry-tasks-authentication-managed-identity.md). L’identité managée est utilisée dans les étapes ultérieures afin que la tâche s’authentifie avec le registre d’images de base. 

Cet exemple utilise une identité affectée par le système, mais vous pouvez créer et activer une identité managée affectée par l’utilisateur pour certains scénarios. Pour plus de détails, consultez [Authentification multiregistre dans une tâche ACR à l’aide d’une identité managée par Azure](container-registry-tasks-cross-registry-authentication.md).

```azurecli
az acr task create \
    --registry $ACR_NAME \
    --name baseexample2 \
    --image helloworld:{{.Run.ID}} \
    --context https://github.com/$GIT_USER/acr-build-helloworld-node.git \
    --file Dockerfile-app \
    --git-access-token $GIT_PAT \
    --arg REGISTRY_NAME=$BASE_ACR.azurecr.io \
    --assign-identity
```

Cette tâche est similaire à la tâche créée dans le [tutoriel précédent](container-registry-tutorial-build-task.md). Elle demande à ACR Tasks de déclencher une génération d’image lorsque les validations sont envoyées vers le référentiel spécifié par `--context`. Alors que le fichier Dockerfile utilisé pour générer l’image dans le tutoriel précédent spécifie une image de base publique (`FROM node:15-alpine`), le fichier Dockerfile dans cette tâche, [Dockerfile-app][dockerfile-app], spécifie une image de base dans le registre d’images de base :

```Dockerfile
FROM ${REGISTRY_NAME}/baseimages/node:15-alpine
```

Cette configuration facilite la simulation d’un correctif de framework dans l’image de base ultérieurement dans ce tutoriel.

## <a name="give-identity-pull-permissions-to-base-registry"></a>Accorder des autorisations d’extraction d’identité au registre de base

Pour accorder à l’identité managée de la tâche l’autorisation d’extraire des images du registre d’images de base, commencez par exécuter [az acr task show][az-acr-task-show] pour obtenir l’ID de principal de service de l’identité. Ensuite, exécutez [az acr show][az-acr-show] pour obtenir l’ID de ressource du registre de base :

```azurecli
# Get service principal ID of the task
principalID=$(az acr task show --name baseexample2 --registry $ACR_NAME --query identity.principalId --output tsv) 

# Get resource ID of the base registry
baseregID=$(az acr show --name $BASE_ACR --query id --output tsv) 
```
 
Affectez les autorisations d’extraction d’identité managée au registre en exécutant [az role assignment create][az-role-assignment-create] :

```azurecli
az role assignment create \
  --assignee $principalID \
  --scope $baseregID --role acrpull 
```

## <a name="add-target-registry-credentials-to-the-task"></a>Ajouter des informations d’identification de registre cible à la tâche

Exécutez [az acr task credential add][az-acr-task-credential-add] pour ajouter des informations d’identification à la tâche. Transmettez le paramètre `--use-identity [system]` pour indiquer que l’identité managée affectée par le système de la tâche peut accéder aux informations d’identification.

```azurecli
az acr task credential add \
  --name baseexample2 \
  --registry $ACR_NAME \
  --login-server $BASE_ACR.azurecr.io \
  --use-identity [system] 
```

## <a name="manually-run-the-task"></a>Exécuter manuellement la tâche

Utilisez la commande [az acr task run][az-acr-task-run] pour déclencher manuellement la tâche et générer l’image de l’application. Cette étape est requise pour que la tâche suive la dépendance de l’image d’application par rapport à l’image de base.

```azurecli
az acr task run --registry $ACR_NAME --name baseexample2
```

Une fois la tâche terminée, prenez note de l’**ID d’exécution** (par exemple, « da6 ») si vous souhaitez terminer l’étape facultative suivante.

### <a name="optional-run-application-container-locally"></a>Facultatif : Exécuter localement un conteneur d’application

Si vous travaillez localement (pas dans Cloud Shell) et si vous avez installé Docker, exécutez le conteneur pour voir l’application affichée dans un navigateur web avant de régénérer son image de base. Si vous utilisez Cloud Shell, ignorez cette section (Cloud Shell ne prend en charge ni `az acr login` ni `docker run`).

Tout d’abord, connectez-vous au registre de conteneurs à l’aide de la commande [az acr login][az-acr-login] :

```azurecli
az acr login --name $ACR_NAME
```

À présent, exécutez localement le conteneur avec `docker run`. Remplacez **\<run-id\>** par l’ID d’exécution trouvé dans la sortie de l’étape précédente (par exemple, « da6 »). Cet exemple nomme le conteneur `myapp` et inclut le paramètre `--rm` pour supprimer le conteneur lorsque vous l’arrêtez.

```bash
docker run -d -p 8080:80 --name myapp --rm $ACR_NAME.azurecr.io/helloworld:<run-id>
```

Accédez à `http://localhost:8080` dans le navigateur ; vous devez voir le numéro de version Node.js affiché dans la page web, comme dans l’exemple suivant. Dans une étape ultérieure, vous allez augmenter le numéro de version en ajoutant un « a » à la chaîne de version.

:::image type="content" source="media/container-registry-tutorial-base-image-update/base-update-01.png" alt-text="Capture d’écran de l’exemple d’application dans le navigateur":::

Pour arrêter et supprimer le conteneur, exécutez la commande suivante :

```bash
docker stop myapp
```

## <a name="list-the-builds"></a>Répertorier les builds

Répertoriez ensuite les exécutions de tâche effectuées par ACR Tasks pour votre registre à l’aide de la commande [az acr task list-runs][az-acr-task-list-runs] :

```azurecli
az acr task list-runs --registry $ACR_NAME --output table
```

Si vous avez terminé le didacticiel précédent (sans supprimer le registre), vous devez pouvoir observer une sortie similaire à ce qui suit. Prenez note du nombre d’exécutions de tâches et du dernier ID d’exécution afin de pouvoir comparer la sortie après avoir mis à jour l’image de base dans la section suivante.

```console
$ az acr task list-runs --registry $ACR_NAME --output table

UN ID    TASK            PLATFORM    STATUS     TRIGGER       STARTED               DURATION
--------  --------------  ----------  ---------  ------------  --------------------  ----------
ca12      baseexample2    linux       Succeeded  Manual        2020-11-21T00:00:56Z  00:00:36
ca11      baseexample1    linux       Succeeded  Image Update  2020-11-20T23:38:24Z  00:00:34
ca10      taskhelloworld  linux       Succeeded  Image Update  2020-11-20T23:38:24Z  00:00:24
cay                       linux       Succeeded  Manual        2020-11-20T23:38:08Z  00:00:22
cax       baseexample1    linux       Succeeded  Manual        2020-11-20T23:33:12Z  00:00:30
caw       taskhelloworld  linux       Succeeded  Commit        2020-11-20T23:16:07Z  00:00:29
```

## <a name="update-the-base-image"></a>Mettre à jour l’image de base

Ici, vous allez simuler un correctif d’infrastructure dans l’image de base. Modifiez **Dockerfile-base** et ajoutez un « a » après le numéro de version défini dans `NODE_VERSION` :

```Dockerfile
ENV NODE_VERSION 15.2.1a
```

Exécutez une tâche rapide dans ACR Tasks pour générer l’image de base modifiée. Prenez note de l’**ID d’exécution** dans la sortie.

```azurecli
az acr build --registry $BASE_ACR --image baseimages/node:15-alpine --file Dockerfile-base .
```

Lorsque la génération est terminée et qu’ACR Tasks a envoyé la nouvelle image de base dans le registre, il déclenche une génération de l’image de l’application. Quelques minutes peuvent être nécessaires à la tâche créée précédemment pour déclencher la génération d’image de l’application, car elle doit détecter l’image de base qui vient d’être générée et envoyée.

## <a name="list-updated-build"></a>Répertorier le build mis à jour

Maintenant que vous avez mis à jour l’image de base, répertoriez à nouveau vos exécutions de tâches pour les comparer à la liste précédente. Si dans un premier temps, la sortie ne diffère pas, exécutez régulièrement la commande pour afficher la nouvelle exécution de tâche dans la liste.

```azurecli
az acr task list-runs --registry $ACR_NAME --output table
```

Le résultat ressemble à ce qui suit. Le DÉCLENCHEUR de la dernière génération exécutée doit être « Mise à jour de l’image », ce qui indique que la tâche a été lancée par la tâche rapide de l’image de base.

```console
$ az acr task list-runs --registry $ACR_NAME --output table

         PLATFORM    STATUS     TRIGGER       STARTED               DURATION
--------  --------------  ----------  ---------  ------------  --------------------  ----------
ca13      baseexample2    linux       Succeeded  Image Update  2020-11-21T00:06:00Z  00:00:43
ca12      baseexample2    linux       Succeeded  Manual        2020-11-21T00:00:56Z  00:00:36
ca11      baseexample1    linux       Succeeded  Image Update  2020-11-20T23:38:24Z  00:00:34
ca10      taskhelloworld  linux       Succeeded  Image Update  2020-11-20T23:38:24Z  00:00:24
cay                       linux       Succeeded  Manual        2020-11-20T23:38:08Z  00:00:22
cax       baseexample1    linux       Succeeded  Manual        2020-11-20T23:33:12Z  00:00:30
caw       taskhelloworld  linux       Succeeded  Commit        2020-11-20T23:16:07Z  00:00:29
```

Si vous souhaitez effectuer l’étape facultative suivante qui est d’exécuter le conteneur nouvellement généré pour voir le numéro de version mis à jour, prenez note de la valeur **ID D’EXÉCUTION** de la génération déclenchée par la mise à jour de l’image (dans la sortie précédente, il s’agit de « ca13 »).

### <a name="optional-run-newly-built-image"></a>Facultatif : Exécuter l’image qui vient d’être générée

Si vous travaillez localement (pas dans Cloud Shell) et si vous avez installé Docker, exécutez la nouvelle image de l’application une fois sa génération terminée. Remplacez `<run-id>` par l’ID D’EXÉCUTION obtenu à l’étape précédente. Si vous utilisez Cloud Shell, ignorez cette section (Cloud Shell ne prend pas en charge `docker run`).

```bash
docker run -d -p 8081:80 --name updatedapp --rm $ACR_NAME.azurecr.io/helloworld:<run-id>
```

Accédez à http://localhost:8081 dans le navigateur ; vous devez voir le numéro de version Node.js mis à jour (avec le « a ») affiché dans la page web :

:::image type="content" source="media/container-registry-tutorial-base-image-update/base-update-02.png" alt-text="Capture d’écran de l’exemple d’application mis à jour dans le navigateur":::

Il est important de noter que si vous avez mis à jour votre image de **base** avec un nouveau numéro de version, l’image **d’application** générée en dernier affiche la nouvelle version. ACR Tasks a choisi la modification que vous avez apportée à l’image de base et régénéré automatiquement l’image de l’application.

Pour arrêter et supprimer le conteneur, exécutez la commande suivante :

```bash
docker stop updatedapp
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez découvert comment utiliser une tâche pour déclencher automatiquement la génération des images de conteneur lorsqu’une image de base a été mise à jour. À présent, passez au tutoriel suivant pour découvrir comment déclencher des tâches selon une planification définie.

> [!div class="nextstepaction"]
> [Exécuter une tâche selon une planification](container-registry-tasks-scheduled.md)

<!-- LINKS - External -->
[base-alpine]: https://hub.docker.com/_/alpine/
[base-dotnet]: https://hub.docker.com/r/microsoft/dotnet/
[base-node]: https://hub.docker.com/_/node/
[base-windows]: https://hub.docker.com/r/microsoft/nanoserver/
[code-sample]: https://github.com/Azure-Samples/acr-build-helloworld-node
[dockerfile-app]: https://github.com/Azure-Samples/acr-build-helloworld-node/blob/master/Dockerfile-app
[dockerfile-base]: https://github.com/Azure-Samples/acr-build-helloworld-node/blob/master/Dockerfile-base

<!-- LINKS - Internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-build]: /cli/azure/acr#az-acr-build
[az-acr-task-create]: /cli/azure/acr/task#az-acr-task-create
[az-acr-task-update]: /cli/azure/acr/task#az-acr-task-update
[az-acr-task-run]: /cli/azure/acr/task#az-acr-task-run
[az-acr-task-show]: /cli/azure/acr/task#az-acr-task-show
[az-acr-task-credential-add]: /cli/azure/acr/task/credential#az-acr-task-credential-add
[az-acr-login]: /cli/azure/acr#az-acr-login
[az-acr-task-list-runs]: /cli/azure/acr/task#az-acr-task-list-runs
[az-acr-task]: /cli/azure/acr#az-acr-task
[az-acr-show]: /cli/azure/acr#az-acr-show
[az-role-assignment-create]: /cli/azure/role/assignment#az-role-assignment-create

