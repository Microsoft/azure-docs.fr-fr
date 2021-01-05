---
title: Tutoriel - Génération rapide d’une image de conteneur
description: Dans ce didacticiel, vous allez apprendre comment générer une image de conteneur Docker dans Azure avec Azure Container Registry Tasks (ACR Tasks), puis la déployer dans Azure Container Instances.
ms.topic: tutorial
ms.date: 11/24/2020
ms.custom: seodec18, mvc, devx-track-azurecli
ms.openlocfilehash: b218f47348d5a26297f14c4bc788a6cf6b78cc60
ms.sourcegitcommit: 2e9643d74eb9e1357bc7c6b2bca14dbdd9faa436
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96030325"
---
# <a name="tutorial-build-and-deploy-container-images-in-the-cloud-with-azure-container-registry-tasks"></a>Tutoriel : Générer et déployer des images conteneurs dans le cloud avec Azure Container Registry Tasks

[ACR Tasks](container-registry-tasks-overview.md) est une suite de fonctionnalités d’Azure Container Registry qui fournit des builds d’images de conteneur Docker rationalisés et efficaces dans Azure. Dans cet article, vous allez apprendre comment utiliser la fonctionnalité de *tâche rapide* d’ACR Tasks.

Le cycle de développement « en boucle interne » est le processus itératif d’écriture de code, de création et de test de votre application avant de procéder au contrôle de code source. La tâche rapide étend votre boucle interne sur le cloud, en prenant en charge la validation de l’exécution des builds et la transmission automatique des images générées à votre registre de conteneurs. Vos images sont générées de manière native dans le cloud, à proximité de votre registre, pour un déploiement plus rapide.

La totalité de votre expertise Dockerfile est directement transférable à ACR Tasks. Vous n’avez aucune obligation de modifier vos fichiers Dockerfiles pour générer dans le cloud avec ACR Tasks, vous devez simplement modifier la commande que vous exécutez. 

Ce didacticiel est la première partie d’une série d’étapes :

> [!div class="checklist"]
> * Récupérer le code source de l'exemple d'application
> * Générer une image de conteneur dans Azure
> * Déployer un conteneur sur Azure Container Instances

Dans les didacticiels suivants, vous apprendrez comment utiliser ACR Tasks pour les générations automatisées d’images conteneur lors de la validation du code et la mise à jour des images de base. ACR Tasks peut également exécuter des [tâches multiétapes](container-registry-tasks-multi-step.md) à l’aide d’un fichier YAML pour définir les étapes permettant de générer, d’envoyer en mode push et éventuellement de tester plusieurs conteneurs.

## <a name="prerequisites"></a>Prérequis

### <a name="github-account"></a>Compte GitHub

Si vous n’en disposez pas encore, créez un compte sur https://github.com. Cette suite de didacticiels utilise un référentiel GitHub pour représenter les générations automatisées d’images dans ACR Tasks.

### <a name="fork-sample-repository"></a>Dupliquer l’exemple de référentiel

Ensuite, utilisez l’IU GitHub pour dupliquer l’exemple de référentiel dans votre compte GitHub. Dans ce didacticiel, vous générez une image de conteneur à partir de la source dans le référentiel, et dans le didacticiel suivant, vous transmettez une validation vers votre fourche du référentiel afin de lancer une tâche automatisée.

Dupliquez ce référentiel : https://github.com/Azure-Samples/acr-build-helloworld-node

![Capture d’écran du bouton de duplication (mis en surbrillance) dans GitHub][quick-build-01-fork]

### <a name="clone-your-fork"></a>Cloner votre fourche

Une fois le référentiel dupliqué, clonez votre fourche et saisissez le répertoire contenant votre clone local.

Clonez le dépôt avec `git`, remplacez **\<your-github-username\>** par votre nom d’utilisateur GitHub :

```console
git clone https://github.com/<your-github-username>/acr-build-helloworld-node
```

Saisissez le répertoire contenant le code source :

```console
cd acr-build-helloworld-node
```

### <a name="bash-shell"></a>Interpréteur de commandes Bash

Les commandes de cette suite de didacticiels sont formatées pour l’interpréteur de commandes Bash. Si vous préférez utiliser PowerShell, l’invite de commandes ou un autre interpréteur de commandes, il vous faudra éventuellement ajuster en conséquence la continuation de ligne et le format de la variable d’environnement.

[!INCLUDE [azure-cli-prepare-your-environment-h3.md](../../includes/azure-cli-prepare-your-environment-h3.md)]

## <a name="build-in-azure-with-acr-tasks"></a>Générer dans Azure avec ACR Tasks

Maintenant que vous avez enregistré le code source sur votre machine, exécutez cette procédure pour créer un registre de conteneurs et générer l’image conteneur avec ACR Tasks.

Pour simplifier l’exécution des exemples de commandes, les didacticiels de cette suite utilisent les variables d’environnement de l’interpréteur de commandes. Exécutez la commande suivante pour définir la variable `ACR_NAME`. Remplacez **\<registry-name\>** par un nom unique à associer à votre nouveau registre de conteneurs. Le nom de registre doit être unique dans Azure et contenir uniquement des lettres minuscules et entre 5 et 50 caractères alphanumériques. Les autres ressources créées dans ce didacticiel étant basées sur ce nom, vous devriez avoir besoin de modifier uniquement cette première variable.

```console
ACR_NAME=<registry-name>
```

Avec cette variable d’environnement de registre de conteneurs renseignée, vous devriez être en mesure de copier-coller le reste des commandes dans le didacticiel, sans modifier aucune valeur. Exécutez les commandes suivantes afin de créer un groupe de ressources et un registre de conteneurs :

```azurecli
RES_GROUP=$ACR_NAME # Resource Group name

az group create --resource-group $RES_GROUP --location eastus
az acr create --resource-group $RES_GROUP --name $ACR_NAME --sku Standard --location eastus
```

Maintenant que vous disposez d’un registre, utilisez ACR Tasks pour générer une image conteneur à partir de l’exemple de code. Exécutez la commande [az acr build][az-acr-build] afin d’effectuer une *tâche rapide* :

```azurecli
az acr build --registry $ACR_NAME --image helloacrtasks:v1 .
```

La sortie de la commande [az acr build][az-acr-build] est similaire à ce qui suit. Vous avez accès au chargement du code source (le « contexte ») dans Azure, et aux détails de l’opération `docker build` exécutée dans le cloud par la tâche ACR. Dans la mesure où ACR Tasks utilise `docker build` pour générer vos images, il n’est pas nécessaire de modifier vos fichiers Dockerfiles pour commencer à utiliser ACR Tasks.

```output
Packing source code into tar file to upload...
Sending build context (4.813 KiB) to ACR...
Queued a build with build ID: da1
Waiting for build agent...
2020/11/18 18:31:42 Using acb_vol_01185991-be5f-42f0-9403-a36bb997ff35 as the home volume
2020/11/18 18:31:42 Setting up Docker configuration...
2020/11/18 18:31:43 Successfully set up Docker configuration
2020/11/18 18:31:43 Logging in to registry: myregistry.azurecr.io
2020/11/18 18:31:55 Successfully logged in
Sending build context to Docker daemon   21.5kB
Step 1/5 : FROM node:15-alpine
15-alpine: Pulling from library/node
Digest: sha256:8dafc0968fb4d62834d9b826d85a8feecc69bd72cd51723c62c7db67c6dec6fa
Status: Image is up to date for node:15-alpine
 ---> a56170f59699
Step 2/5 : COPY . /src
 ---> 88087d7e709a
Step 3/5 : RUN cd /src && npm install
 ---> Running in e80e1263ce9a
npm notice created a lockfile as package-lock.json. You should commit this file.
npm WARN helloworld@1.0.0 No repository field.

up to date in 0.1s
Removing intermediate container e80e1263ce9a
 ---> 26aac291c02e
Step 4/5 : EXPOSE 80
 ---> Running in 318fb4c124ac
Removing intermediate container 318fb4c124ac
 ---> 113e157d0d5a
Step 5/5 : CMD ["node", "/src/server.js"]
 ---> Running in fe7027a11787
Removing intermediate container fe7027a11787
 ---> 20a27b90eb29
Successfully built 20a27b90eb29
Successfully tagged myregistry.azurecr.io/helloacrtasks:v1
2020/11/18 18:32:11 Pushing image: myregistry.azurecr.io/helloacrtasks:v1, attempt 1
The push refers to repository [myregistry.azurecr.io/helloacrtasks]
6428a18b7034: Preparing
c44b9827df52: Preparing
172ed8ca5e43: Preparing
8c9992f4e5dd: Preparing
8dfad2055603: Preparing
c44b9827df52: Pushed
172ed8ca5e43: Pushed
8dfad2055603: Pushed
6428a18b7034: Pushed
8c9992f4e5dd: Pushed
v1: digest: sha256:b038dcaa72b2889f56deaff7fa675f58c7c666041584f706c783a3958c4ac8d1 size: 1366
2020/11/18 18:32:43 Successfully pushed image: myregistry.azurecr.io/helloacrtasks:v1
2020/11/18 18:32:43 Step ID acb_step_0 marked as successful (elapsed time in seconds: 15.648945)
The following dependencies were found:
- image:
    registry: myregistry.azurecr.io
    repository: helloacrtasks
    tag: v1
    digest: sha256:b038dcaa72b2889f56deaff7fa675f58c7c666041584f706c783a3958c4ac8d1
  runtime-dependency:
    registry: registry.hub.docker.com
    repository: library/node
    tag: 15-alpine
    digest: sha256:8dafc0968fb4d62834d9b826d85a8feecc69bd72cd51723c62c7db67c6dec6fa
  git: {}

Run ID: da1 was successful after 1m9.970148252s
```

Peu avant la fin de la tâche, ACR Tasks affiche les dépendances découvertes pour votre image. Ce faisant, ACR Tasks est en mesure d’automatiser les générations d’images sur les mises à jour des images de base, comme lors de la mise à jour d’une image avec des correctifs du système d’exploitation ou de l’infrastructure. Plus loin dans cette suite de didacticiels, vous approfondirez la prise en charge des mises à jour d’images de base par ACR Tasks.

## <a name="deploy-to-azure-container-instances"></a>Déployer vers Azure Container Instances

ACR Tasks transfère automatiquement les images générées vers votre registre par défaut, ce qui vous permet de les déployer immédiatement.

Dans cette section, vous créez une instance Azure Key Vault et un principal du service, puis déployez le conteneur vers Azure Container Instances (ACI) à l’aide des informations d’identification du principal.

### <a name="configure-registry-authentication"></a>Configurer l’authentification du registre

Tous les scénarios de production doivent utiliser les [principaux du service][service-principal-auth] pour accéder à un registre de conteneurs Azure. Les principaux de service vous permettent de fournir un contrôle d’accès basé sur des rôles à vos images conteneur. Par exemple, vous pouvez configurer un principal de service avec uniquement un accès d’extraction à un registre.

#### <a name="create-a-key-vault"></a>Création d’un coffre de clés

Si vous n’avez pas encore un coffre dans [Azure Key Vault](../key-vault/index.yml), créez-en un avec Azure CLI à l’aide des commandes suivantes.

```azurecli
AKV_NAME=$ACR_NAME-vault

az keyvault create --resource-group $RES_GROUP --name $AKV_NAME
```

#### <a name="create-a-service-principal-and-store-credentials"></a>Créer un principal de service et stocker les informations d’identification

Vous devez maintenant créer un principal de service et stocker ses informations d’identification dans votre coffre de clés.

Utilisez la commande [az ad sp create-for-rbac][az-ad-sp-create-for-rbac] pour créer le principal du service, et [az keyvault secret set][az-keyvault-secret-set] pour stocker le **mot de passe** du principal de service dans le coffre :

```azurecli
# Create service principal, store its password in AKV (the registry *password*)
az keyvault secret set \
  --vault-name $AKV_NAME \
  --name $ACR_NAME-pull-pwd \
  --value $(az ad sp create-for-rbac \
                --name $ACR_NAME-pull \
                --scopes $(az acr show --name $ACR_NAME --query id --output tsv) \
                --role acrpull \
                --query password \
                --output tsv)
```

L’argument `--role` dans la commande précédente configure le principal de service avec le rôle *acrpull*, ce qui lui accorde uniquement un accès d’extraction au registre. Pour accorder les accès push et pull (envoi et tirage), affectez à l’argument `--role` la valeur *acrpush*.

Ensuite, stockez le *appId* du principal de service dans le coffre, qui est le **nom d’utilisateur** que vous passez à Azure Container Registry pour l’authentification :

```azurecli
# Store service principal ID in AKV (the registry *username*)
az keyvault secret set \
    --vault-name $AKV_NAME \
    --name $ACR_NAME-pull-usr \
    --value $(az ad sp show --id http://$ACR_NAME-pull --query appId --output tsv)
```

Vous avez créé un coffre de clés Azure et il contient deux secrets :

* `$ACR_NAME-pull-usr`: ID de principal du service, pour une utilisation comme **nom d’utilisateur** du registre de conteneurs.
* `$ACR_NAME-pull-pwd`: mot de passe du principal du service, pour une utilisation comme **mot de passe** du registre de conteneurs.

Vous pouvez maintenant référencer ces secrets par nom lorsque vous ou vos applications et services extrayez des images du registre.

### <a name="deploy-a-container-with-azure-cli"></a>Déployer un conteneur avec Azure CLI

Maintenant que les informations d’identification du principal de service sont stockées en tant que secrets de Azure Key Vault, vos applications et vos services peuvent les utiliser pour accéder à votre registre privé.

Exécutez la commande [az container create][az-container-create]suivante pour déployer une instance de conteneur. La commande utilise les informations d’identification du principal du service stockées dans Azure Key Vault pour s’authentifier sur votre registre de conteneurs.

```azurecli
az container create \
    --resource-group $RES_GROUP \
    --name acr-tasks \
    --image $ACR_NAME.azurecr.io/helloacrtasks:v1 \
    --registry-login-server $ACR_NAME.azurecr.io \
    --registry-username $(az keyvault secret show --vault-name $AKV_NAME --name $ACR_NAME-pull-usr --query value -o tsv) \
    --registry-password $(az keyvault secret show --vault-name $AKV_NAME --name $ACR_NAME-pull-pwd --query value -o tsv) \
    --dns-name-label acr-tasks-$ACR_NAME \
    --query "{FQDN:ipAddress.fqdn}" \
    --output table
```

La valeur `--dns-name-label` doit être unique dans Azure, afin que la commande précédente ajoute le nom de votre registre de conteneurs à l’étiquette de nom DNS du conteneur. La sortie pour la commande affiche le nom de domaine complet du conteneur (FQDN), par exemple :

```output
FQDN
----------------------------------------------
acr-tasks-myregistry.eastus.azurecontainer.io
```

Consignez le nom de domaine complet du conteneur, car vous allez vous en servir dans la section suivante.

### <a name="verify-the-deployment"></a>Vérifier le déploiement

Pour contrôler le processus de démarrage du conteneur, utilisez la commande [az container attach][az-container-attach] :

```azurecli
az container attach --resource-group $RES_GROUP --name acr-tasks
```

La sortie `az container attach` affiche dans un premier temps le statut du conteneur quand il extrait l’image et démarre, puis lie les éléments STDOUT et STDERR de votre console locale à ceux du conteneur.

```output
Container 'acr-tasks' is in state 'Running'...
(count: 1) (last timestamp: 2020-11-18 18:39:10+00:00) pulling image "myregistry.azurecr.io/helloacrtasks:v1"
(count: 1) (last timestamp: 2020-11-18 18:39:15+00:00) Successfully pulled image "myregistry.azurecr.io/helloacrtasks:v1"
(count: 1) (last timestamp: 2020-11-18 18:39:17+00:00) Created container
(count: 1) (last timestamp: 2020-11-18 18:39:17+00:00) Started container

Start streaming logs:
Server running at http://localhost:80
```

Lorsque `Server running at http://localhost:80` s’affiche, accédez au nom de domaine complet du conteneur dans votre navigateur afin d’afficher l’application en cours d’exécution. Le nom de domaine complet s’est normalement affiché dans la sortie de la commande `az container create` que vous avez exécutée dans la section précédente.

:::image type="content" source="media/container-registry-tutorial-quick-build/quick-build-02-browser.png" alt-text="Exemple d’application s’exécutant dans le navigateur":::

Pour dissocier votre console du conteneur, appuyez sur `Control+C`.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Arrêtez l’instance de conteneur avec la commande [az container delete][az-container-delete] :

```azurecli
az container delete --resource-group $RES_GROUP --name acr-tasks
```

Pour supprimer *toutes* les ressources créées dans ce didacticiel, notamment le registre de conteneurs, le coffre de clés et le principal du service, exécutez les commandes suivantes. Ces ressources sont utilisées dans le [prochain didacticiel](container-registry-tutorial-build-task.md) de la suite, vous avez donc intérêt à les conserver si vous comptez passer directement à ce prochain didacticiel.

```azurecli
az group delete --resource-group $RES_GROUP
az ad sp delete --id http://$ACR_NAME-pull
```

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez testé votre boucle interne avec une tâche rapide, configurez une **tâche de génération** afin de déclencher des générations d’images conteneurs lorsque vous validez le code source sur un référentiel Git :

> [!div class="nextstepaction"]
> [Déclencher des générations automatiques avec des tâches](container-registry-tutorial-build-task.md)

<!-- LINKS - External -->
[sample-archive]: https://github.com/Azure-Samples/acr-build-helloworld-node/archive/master.zip

<!-- LINKS - Internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-build]: /cli/azure/acr#az-acr-build
[az-ad-sp-create-for-rbac]: /cli/azure/ad/sp#az-ad-sp-create-for-rbac
[az-container-attach]: /cli/azure/container#az-container-attach
[az-container-create]: /cli/azure/container#az-container-create
[az-container-delete]: /cli/azure/container#az-container-delete
[az-keyvault-create]: /cli/azure/keyvault/secret#az-keyvault-create
[az-keyvault-secret-set]: /cli/azure/keyvault/secret#az-keyvault-secret-set
[az-login]: /cli/azure/reference-index#az-login
[service-principal-auth]: container-registry-auth-service-principal.md

<!-- IMAGES -->
[quick-build-01-fork]: ./media/container-registry-tutorial-quick-build/quick-build-01-fork.png
[quick-build-02-browser]: ./media/container-registry-tutorial-quick-build/quick-build-02-browser.png
