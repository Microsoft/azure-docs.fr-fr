---
title: Exécution de tâche rapide à l’aide d’un modèle
description: Mettre en file d’attente une exécution de tâche ACR pour générer une image à l’aide d’un modèle Azure Resource Manager
ms.topic: article
ms.date: 04/22/2020
ms.openlocfilehash: 7ad40d2e925d5e1443af9bce4115d45b0e8c06e1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "82927766"
---
# <a name="run-acr-tasks-using-resource-manager-templates"></a>Exécuter ACR Tasks à l’aide de modèles Resource Manager

[ACR Tasks](container-registry-tasks-overview.md) est une suite de fonctionnalités dans Azure Container Registry qui vous aide à gérer et à modifier des images conteneur tout au long du cycle de vie du conteneur. 

Cet article montre des exemples de modèles Azure Resource Manager permettant de mettre en file d’attente une exécution de tâche rapide, semblable à celle que vous pouvez créer manuellement à l’aide de la commande [az acr build][az-acr-build].

Un modèle Resource Manager permettant de mettre en file d’attente une exécution de tâche est utile dans les scénarios d’automatisation et étend la fonctionnalité de `az acr build`. Par exemple :

* Utilisez un modèle pour créer un registre de conteneurs et mettre immédiatement en file d’attente une exécution de tâche visant à générer et à envoyer une image conteneur
* Créez ou activez des ressources supplémentaires que vous pouvez utiliser dans une exécution de tâche rapide, telle qu’une identité managée pour les ressources Azure

## <a name="limitations"></a>Limites

* Vous devez spécifier un contexte distant tel, qu’un référentiel GitHub, comme [emplacement source](container-registry-tasks-overview.md#context-locations) pour votre exécution de tâche. Vous ne pouvez pas utiliser un contexte de source locale.
* Pour les exécutions de tâche utilisant une identité managée, seule une identité managée *attribuée par l’utilisateur* est autorisée.

## <a name="prerequisites"></a>Prérequis

* **Action GitHub** : créez un compte sur https://github.com si vous n’en avez pas encore. 
* **Dupliquer (fork) un exemple de référentiel** : pour les exemples de tâches illustrés ici, utilisez l’interface utilisateur GitHub pour dupliquer (fork) l’exemple de référentiel suivant dans votre compte GitHub : https://github.com/Azure-Samples/acr-build-helloworld-node. Ce référentiel contient des exemples de Dockerfiles et de code source permettant de créer de petites images conteneur.

## <a name="example-create-registry-and-queue-task-run"></a>Exemple : Création d’un registre et mise en file d’attente d’une exécution de tâche

Cet exemple utilise un [exemple de modèle](https://github.com/Azure/acr/tree/master/docs/tasks/run-as-deployment/quickdockerbuild) pour créer un registre de conteneurs et mettre en file d’attente une exécution de tâche qui génère et envoie une image. 

### <a name="template-parameters"></a>Paramètres de modèle

Pour cet exemple, fournissez des valeurs pour les paramètres de modèle suivants :

|Paramètre  |Valeur  |
|---------|---------|
|registryName     |Nom unique du registre créé         |
|repository     |Référentiel cible pour la tâche de génération        |
|taskRunName     |Nom de l’exécution de tâche, qui spécifie une balise d’image |
|sourceLocation     |Contexte distant pour la tâche de génération, par exemple, https://github.com/Azure-Samples/acr-build-helloworld-node. Le Dockerfile de la racine du référentiel génère une image conteneur pour une petite application web Node.js. Si vous le souhaitez, utilisez votre fourche du référentiel comme contexte de génération.         |

### <a name="deploy-the-template"></a>Déployer le modèle

Déployez ensuite le modèle avec la commande [az deployment group create][az-deployment-group-create]. Cet exemple génère et envoie l’image *helloworld-node:testrun* dans un registre nommé *mycontainerregistry*.

```azurecli
az deployment group create \
  --resource-group myResourceGroup \
  --template-uri https://raw.githubusercontent.com/Azure/acr/master/docs/tasks/run-as-deployment/quickdockerbuild/azuredeploy.json \
  --parameters \
    registryName=mycontainerregistry \
    repository=helloworld-node \
    taskRunName=testrun \
    sourceLocation=https://github.com/Azure-Samples/acr-build-helloworld-node.git
 ```

La commande précédente transmet les paramètres sur la ligne de commande. Si vous le souhaitez, transmettez-les dans un [fichier de paramètres](../azure-resource-manager/templates/parameter-files.md).

### <a name="verify-deployment"></a>Vérifier le déploiement

Une fois le déploiement terminé, vérifiez que l’image est générée en exécutant la commande [az acr repository show-tags][az-acr-repository-show-tags] :

```azurecli
az acr repository show-tags \
  --name mycontainerregistry \
  --repository helloworld-node --output table
```

Sortie :

```console
Result
--------
testrun
```

### <a name="view-run-log"></a>Afficher le journal d’exécution

Pour afficher les détails de l’exécution de tâche, affichez le journal d’exécution.

Tout d’abord, récupérez l’ID d’exécution à l’aide de la commande [az acr task list-runs][az-acr-task-list-runs].
```azurecli
az acr task list-runs \
  --registry mycontainerregistry --output table
```

Le résultat se présente ainsi :

```console
RUN ID    TASK    PLATFORM    STATUS     TRIGGER    STARTED               DURATION
--------  ------  ----------  ---------  ---------  --------------------  ----------
ca1               linux       Succeeded  Manual     2020-03-23T17:54:28Z  00:00:48
```

Exécutez [az acr task logs][az-acr-task-logs] pour afficher les journaux d’exécution de tâche de l’ID d’exécution, dans ce cas *ca1* :

```azurecli
az acr task logs \
  --registry mycontainerregistry \
  --run-id ca1
```

La sortie affiche le journal d’exécution de tâche.

Vous pouvez également afficher le journal d’exécution de tâche dans le Portail Azure. 

1. Accédez à votre registre de conteneurs.
2. Sous **Services**, sélectionnez **Tâches** > **Exécutions**.
3. Sélectionnez l’ID d’exécution, dans ce cas *ca1*. 

Le portail affiche le journal d’exécution de tâche.

## <a name="example-task-run-with-managed-identity"></a>Exemple : Exécution de tâche avec une identité managée

Utilisez un [exemple de modèle](https://github.com/Azure/acr/tree/master/docs/tasks/run-as-deployment/quickdockerbuildwithidentity) pour mettre en file d’attente une exécution de tâche qui active une identité managée affectée par l’utilisateur. Au cours de l’exécution de tâche, l’identité s’authentifie pour extraire une image d’un autre registre de conteneurs Azure. 

Ce scénario est similaire à [l’authentification multiregistre dans une tâche ACR à l’aide d’une identité managée par Azure](container-registry-tasks-cross-registry-authentication.md). Par exemple, une organisation peut conserver un registre centralisé avec des images de base accessibles par plusieurs équipes de développement.

### <a name="prepare-base-registry"></a>Préparer le registre de base

À des fins de démonstration, créez un registre de conteneurs distinct comme registre de base et envoyez une image de base Node.js extraite de Docker Hub.

1. Créez un deuxième registre de conteneurs, par exemple *mybaseregistry*, pour stocker les images de base.
1. Extrayez l’image `node:9-alpine` à partir de Docker Hub, balisez-la pour votre registre de base, puis envoyez-la vers le registre de base :

  ```azurecli
  docker pull node:9-alpine
  docker tag node:9-alpine mybaseregistry.azurecr.io/baseimages/node:9-alpine
  az acr login -n mybaseregistry
  docker push mybaseregistry.azurecr.io/baseimages/node:9-alpine
  ```

### <a name="create-new-dockerfile"></a>Créer un Dockerfile

Créez un Dockerfile qui extrait l’image de base de votre registre de base. Procédez comme suit dans votre fork locale du référentiel GitHub, par exemple, `https://github.com/myGitHubID/acr-build-helloworld-node.git`.

1. Dans l’interface utilisateur GitHub, sélectionnez **Create new file** (Créer un fichier).
1. Nommez votre fichier *Dockerfile-test* et collez le contenu suivant. Remplacez le nom de votre registre par *mybaseregistry*.
    ```
    FROM mybaseregistry.azurecr.io/baseimages/node:9-alpine
    COPY . /src
    RUN cd /src && npm install
    EXPOSE 80
    CMD ["node", "/src/server.js"]
    ```
 1. Sélectionnez **Commit new file** (Valider le nouveau fichier).

[!INCLUDE [container-registry-tasks-user-assigned-id](../../includes/container-registry-tasks-user-assigned-id.md)]

### <a name="give-identity-pull-permissions-to-the-base-registry"></a>Accorder des autorisations d’extraction d’identité au registre de base

Accordez à l’identité managée les autorisations d’extraction à partir du registre de base, *mybaseregistry*.

Utilisez la commande [az acr show][az-acr-show] pour obtenir l’ID de ressource du registre de base et le stocker dans une variable :

```azurecli
baseregID=$(az acr show \
  --name mybaseregistry \
  --query id --output tsv)
```

Utilisez la commande [az role assignment create][az-role-assignment-create] pour attribuer le rôle Acrpull au registre de base. Ce rôle dispose uniquement des autorisations pour extraire des images du registre.

```azurecli
az role assignment create \
  --assignee $principalID \
  --scope $baseregID \
  --role acrpull
```

### <a name="template-parameters"></a>Paramètres de modèle

Pour cet exemple, fournissez des valeurs pour les paramètres de modèle suivants :

|Paramètre  |Valeur  |
|---------|---------|
|registryName     |Nom du registre dans lequel l’image est générée  |
|repository     |Référentiel cible pour la tâche de génération        |
|taskRunName     |Nom de l’exécution de tâche, qui spécifie une balise d’image |
|userAssignedIdentity |ID de ressource de l’identité attribuée par l’utilisateur qui est activée dans la tâche|
|customRegistryIdentity | ID client de l’identité attribuée par l’utilisateur qui est activée dans la tâche, utilisé pour l’authentification auprès du registre personnalisé |
|customRegistry |Nom du serveur de connexion du registre personnalisé auquel la tâche accède, par exemple, *mybaseregistry.azurecr.io*|
|sourceLocation     |Contexte distant pour la tâche de génération, par exemple, *https://github.com/\<your-GitHub-ID\> /acr-build-helloworld-node* |
|dockerFilePath | Chemin d’accès au Dockerfile dans le contexte distant, utilisé pour générer l’image |

### <a name="deploy-the-template"></a>Déployer le modèle

Déployez ensuite le modèle avec la commande [az deployment group create][az-deployment-group-create]. Cet exemple génère et envoie l’image *helloworld-node:testrun* dans un registre nommé *mycontainerregistry*. L’image de base est extraite de *mybaseregistry.azurecr.io*.

```azurecli
az deployment group create \
  --resource-group myResourceGroup \
  --template-uri https://raw.githubusercontent.com/Azure/acr/master/docs/tasks/run-as-deployment/quickdockerbuildwithidentity/azuredeploy.json \
  --parameters \
    registryName=mycontainerregistry \
    repository=helloworld-node \
    taskRunName=basetask \
    userAssignedIdentity=$resourceID \
    customRegistryIdentity=$clientID \
    sourceLocation=https://github.com/<your-GitHub-ID>/acr-build-helloworld-node.git \
    dockerFilePath=Dockerfile-test \
    customRegistry=mybaseregistry.azurecr.io
```

La commande précédente transmet les paramètres sur la ligne de commande. Si vous le souhaitez, transmettez-les dans un [fichier de paramètres](../azure-resource-manager/templates/parameter-files.md).

### <a name="verify-deployment"></a>Vérifier le déploiement

Une fois le déploiement terminé, vérifiez que l’image est générée en exécutant la commande [az acr repository show-tags][az-acr-repository-show-tags] :

```azurecli
az acr repository show-tags \
  --name mycontainerregistry \
  --repository helloworld-node --output table
```

Sortie :

```console
Result
--------
basetask
```

### <a name="view-run-log"></a>Afficher le journal d’exécution

Pour afficher le journal d’exécution, consultez les étapes de la [section précédente](#view-run-log).

## <a name="next-steps"></a>Étapes suivantes

 * D’autres exemples de modèles sont accessibles dans le[référentiel GitHub ACR](https://github.com/Azure/acr/tree/master/docs/tasks/run-as-deployment).
 * Pour plus d’informations sur les propriétés des modèles, consultez la documentation de référence des modèles pour les [exécutions de tâche](/azure/templates/microsoft.containerregistry/2019-06-01-preview/registries/taskruns) et les [tâches](/azure/templates/microsoft.containerregistry/2019-06-01-preview/registries/tasks).


<!-- LINKS - Internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-build]: /cli/azure/acr#az-acr-build
[az-acr-show]: /cli/azure/acr#az-acr-show
[az-acr-task-run]: /cli/azure/acr/task#az-acr-task-run
[az-acr-task-logs]: /cli/azure/acr/task#az-acr-task-logs
[az-acr-repository-show-tags]: /cli/azure/acr/repository#az-acr-repository-show-tags
[az-acr-task-list-runs]: /cli/azure/acr/task#az-acr-task-list-runs
[az-deployment-group-create]: /cli/azure/deployment/group#az-deployment-group-create
[az-identity-create]: /cli/azure/identity#az-identity-create
[az-identity-show]: /cli/azure/identity#az-identity-show
[az-role-assignment-create]: /cli/azure/role/assignment#az-role-assignment-create
