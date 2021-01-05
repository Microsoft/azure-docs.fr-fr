---
title: Authentification externe à partir d’une tâche ACR
description: Configurez une tâche Azure Container Registry (tâche ACR) pour lire les informations d’identification Docker Hub stockées dans un coffre de clés Azure à l’aide d’une identité managée pour les ressources Azure.
ms.topic: article
ms.date: 07/06/2020
ms.openlocfilehash: 0bc43f958a14016146160a06372af0b36a9fff75
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86058127"
---
# <a name="external-authentication-in-an-acr-task-using-an-azure-managed-identity"></a>Authentification externe dans une tâche ACR à l’aide d’une identité managée par Azure 

Dans une [tâche ACR](container-registry-tasks-overview.md), vous pouvez [activer une identité managée pour les ressources Azure](container-registry-tasks-authentication-managed-identity.md). La tâche peut utiliser l’identité pour accéder à d’autres ressources Azure, sans qu’il soit nécessaire de fournir ni de gérer des informations d’identification. 

Dans cet article, vous allez apprendre à activer une identité managée dans une tâche qui accède à des secrets stockés dans un coffre de clés Azure. 

Pour créer les ressources Azure, il est nécessaire dans le cadre de cet article d’exécuter Azure CLI version 2.0.68 ou ultérieure. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI][azure-cli].

## <a name="scenario-overview"></a>Présentation du scénario

L’exemple de tâche lit les informations d’identification Docker Hub stockées dans un coffre de clés Azure. Les informations d’identification sont pour un compte Docker Hub doté d’autorisations en écriture (push) sur un référentiel Docker Hub privé. Pour lire les informations d’identification, configurez la tâche avec une identité managée et attribuez-lui les autorisations appropriées. La tâche associée à l’identité génère une image et se connecte à Docker Hub pour envoyer l’image au dépôt privé. 

Cet exemple montre les étapes à effectuer à l’aide d’une identité managée affectée par l’utilisateur ou par le système. Votre choix d’identité dépend des besoins de votre organisation.

Dans un scénario concret, une entreprise peut publier des images sur un dépôt privé dans Docker Hub dans le cadre d’un processus de génération. 

## <a name="prerequisites"></a>Prérequis

Vous avez besoin d’un registre de conteneurs Azure dans lequel vous exécutez la tâche. Dans cet article, ce registre est nommé *myregistry*. Remplacez-le par votre propre nom de registre dans les étapes suivantes.

Si vous ne disposez pas d’un registre de conteneurs Azure, consultez [Démarrage rapide : Créer un registre de conteneurs privé avec Azure CLI](container-registry-get-started-azure-cli.md). Vous n’avez pas encore besoin d’envoyer (push) des images au registre.

Vous avez également besoin d’un dépôt privé dans Docker Hub, et d’un compte Docker Hub disposant des autorisations nécessaires pour écrire dans le dépôt. Dans cet exemple, ce dépôt est nommé *hubuser/hubrepo*. 

## <a name="create-a-key-vault-and-store-secrets"></a>Créer un coffre de clés et stocker des secrets

Commencez, si nécessaire, par créer un groupe de ressources nommé *myResourceGroup* à l’emplacement *eastus* à l’aide de la commande [az group create][az-group-create] suivante :

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Utilisez la commande [az keyvault create][az-keyvault-create] pour créer un coffre de clés. Veillez à spécifier un nom de coffre de clés unique. 

```azurecli-interactive
az keyvault create --name mykeyvault --resource-group myResourceGroup --location eastus
```

Stockez les informations d’identification Docker Hub requises dans le coffre de clés à l’aide de la commande [az keyvault secret set][az-keyvault-secret-set] : Dans ces commandes, les valeurs sont passées dans les variables d’environnement :

```azurecli
# Store Docker Hub user name
az keyvault secret set \
  --name UserName \
  --value $USERNAME \
  --vault-name mykeyvault

# Store Docker Hub password
az keyvault secret set \
  --name Password \
  --value $PASSWORD \
  --vault-name mykeyvault
```

Dans un scénario concret, les secrets seraient probablement définis et maintenus dans un processus distinct.

## <a name="define-task-steps-in-yaml-file"></a>Définir des étapes de tâche dans le fichier YAML

Les étapes de cet exemple de tâche sont définies dans un [fichier YAML](container-registry-tasks-reference-yaml.md). Créez un fichier nommé `dockerhubtask.yaml` dans un répertoire de travail local et collez le contenu suivant. Veillez à remplacer le nom du coffre de clés dans le fichier par le nom de votre coffre de clés.

```yml
version: v1.1.0
# Replace mykeyvault with the name of your key vault
secrets:
  - id: username
    keyvault: https://mykeyvault.vault.azure.net/secrets/UserName
  - id: password
    keyvault: https://mykeyvault.vault.azure.net/secrets/Password
steps:
# Log in to Docker Hub
  - cmd: bash echo '{{.Secrets.password}}' | docker login --username '{{.Secrets.username}}' --password-stdin 
# Build image
  - build: -t {{.Values.PrivateRepo}}:$ID https://github.com/Azure-Samples/acr-tasks.git -f hello-world.dockerfile
# Push image to private repo in Docker Hub
  - push:
    - {{.Values.PrivateRepo}}:$ID
```

Les étapes de tâche effectuent ce qui suit :

* Gérez les informations d’identification secrètes pour l’authentification auprès de Docker Hub.
* Authentifiez-vous auprès de Docker Hub en transmettant les secrets à la commande `docker login`.
* Générez une image à l’aide d’un exemple de fichier Docker dans le dépôt [Azure-Samples/acr-tasks](https://github.com/Azure-Samples/acr-tasks.git).
* Envoyez par push l’image au dépôt privé Docker Hub.


## <a name="option-1-create-task-with-user-assigned-identity"></a>Option 1 : Créer une tâche avec une identité affectée par l’utilisateur

Les étapes de cette section créent une tâche et activent une identité affectée par l’utilisateur. Si vous souhaitez activer une identité affectée par le système à la place, consultez l’[option 2 : Créer une tâche avec une identité affectée par le système](#option-2-create-task-with-system-assigned-identity). 

[!INCLUDE [container-registry-tasks-user-assigned-id](../../includes/container-registry-tasks-user-assigned-id.md)]

### <a name="create-task"></a>Créer une tâche

Créez la tâche *dockerhubtask* en exécutant la commande [az acr task create][az-acr-task-create] suivante. La tâche s’exécute sans contexte de code source et la commande référence le fichier `dockerhubtask.yaml` dans le répertoire de travail. Le paramètre `--assign-identity` transmet l’ID de ressource de l’identité affectée par l’utilisateur. 

```azurecli
az acr task create \
  --name dockerhubtask \
  --registry myregistry \
  --context /dev/null \
  --file dockerhubtask.yaml \
  --assign-identity $resourceID
```

[!INCLUDE [container-registry-tasks-user-id-properties](../../includes/container-registry-tasks-user-id-properties.md)]


### <a name="grant-identity-access-to-key-vault"></a>Autoriser l’identité à accéder à un coffre de clés

Exécutez la commande [az keyvault set-policy][az-keyvault-set-policy] suivante pour définir une stratégie d’accès sur le coffre de clés. L’exemple suivant permet à l’identité de lire des secrets du coffre de clés. 

```azurecli
az keyvault set-policy --name mykeyvault \
  --resource-group myResourceGroup \
  --object-id $principalID \
  --secret-permissions get
```

Poursuivez avec l’[exécution manuelle de la tâche](#manually-run-the-task).

## <a name="option-2-create-task-with-system-assigned-identity"></a>Option n°2 : Créer une tâche avec une identité affectée par le système

Les étapes de cette section créent une tâche et activent une identité affectée par le système. Si vous souhaitez activer une identité affectée par l’utilisateur à la place, consultez l’[Option 1 : Créer une tâche avec une identité affectée par l’utilisateur](#option-1-create-task-with-user-assigned-identity). 

### <a name="create-task"></a>Créer une tâche

Créez la tâche *dockerhubtask* en exécutant la commande [az acr task create][az-acr-task-create] suivante. La tâche s’exécute sans contexte de code source et la commande référence le fichier `dockerhubtask.yaml` dans le répertoire de travail. Le paramètre `--assign-identity`, sans valeur, active une identité managée affectée par le système sur la tâche.  

```azurecli
az acr task create \
  --name dockerhubtask \
  --registry myregistry \
  --context /dev/null \
  --file dockerhubtask.yaml \
  --assign-identity 
```

[!INCLUDE [container-registry-tasks-system-id-properties](../../includes/container-registry-tasks-system-id-properties.md)]

### <a name="grant-identity-access-to-key-vault"></a>Autoriser l’identité à accéder à un coffre de clés

Exécutez la commande [az keyvault set-policy][az-keyvault-set-policy] suivante pour définir une stratégie d’accès sur le coffre de clés. L’exemple suivant permet à l’identité de lire des secrets du coffre de clés. 

```azurecli
az keyvault set-policy --name mykeyvault \
  --resource-group myResourceGroup \
  --object-id $principalID \
  --secret-permissions get
```

## <a name="manually-run-the-task"></a>Exécuter manuellement la tâche

Pour vérifier que la tâche dans laquelle vous avez activé une identité managée s’exécute correctement, déclenchez manuellement la tâche à l’aide de la commande [az acr task run][az-acr-task-run]. Le paramètre `--set` est utilisé pour transmettre le nom du dépôt privé à la tâche. Dans cet exemple, l’espace réservé du nom de dépôt est *hubuser/hubrepo*.

```azurecli
az acr task run --name dockerhubtask --registry myregistry --set PrivateRepo=hubuser/hubrepo
```

Quand la tâche s’exécute correctement, la sortie indique une authentification réussie auprès de Docker Hub et l’image est correctement générée et envoyée au dépôt privé :

```console
Queued a run with ID: cf24
Waiting for an agent...
2019/06/20 18:05:55 Using acb_vol_b1edae11-30de-4f2b-a9c7-7d743e811101 as the home volume
2019/06/20 18:05:58 Creating Docker network: acb_default_network, driver: 'bridge'
2019/06/20 18:05:58 Successfully set up Docker network: acb_default_network
2019/06/20 18:05:58 Setting up Docker configuration...
2019/06/20 18:05:59 Successfully set up Docker configuration
2019/06/20 18:05:59 Logging in to registry: myregistry.azurecr.io
2019/06/20 18:06:00 Successfully logged into myregistry.azurecr.io
2019/06/20 18:06:00 Executing step ID: acb_step_0. Timeout(sec): 600, Working directory: '', Network: 'acb_default_network'
2019/06/20 18:06:00 Launching container with name: acb_step_0
[...]
Login Succeeded
2019/06/20 18:06:02 Successfully executed container: acb_step_0
2019/06/20 18:06:02 Executing step ID: acb_step_1. Timeout(sec): 600, Working directory: '', Network: 'acb_default_network'
2019/06/20 18:06:02 Scanning for dependencies...
2019/06/20 18:06:04 Successfully scanned dependencies
2019/06/20 18:06:04 Launching container with name: acb_step_1
Sending build context to Docker daemon    129kB
[...]
2019/06/20 18:06:07 Successfully pushed image: hubuser/hubrepo:cf24
2019/06/20 18:06:07 Step ID: acb_step_0 marked as successful (elapsed time in seconds: 2.064353)
2019/06/20 18:06:07 Step ID: acb_step_1 marked as successful (elapsed time in seconds: 2.594061)
2019/06/20 18:06:07 Populating digests for step ID: acb_step_1...
2019/06/20 18:06:09 Successfully populated digests for step ID: acb_step_1
2019/06/20 18:06:09 Step ID: acb_step_2 marked as successful (elapsed time in seconds: 2.743923)
2019/06/20 18:06:09 The following dependencies were found:
2019/06/20 18:06:09
- image:
    registry: registry.hub.docker.com
    repository: hubuser/hubrepo
    tag: cf24
    digest: sha256:92c7f9c92844bbbb5d0a101b22f7c2a7949e40f8ea90c8b3bc396879d95e899a
  runtime-dependency:
    registry: registry.hub.docker.com
    repository: library/hello-world
    tag: latest
    digest: sha256:0e11c388b664df8a27a901dce21eb89f11d8292f7fca1b3e3c4321bf7897bffe
  git:
    git-head-revision: b0ffa6043dd893a4c75644c5fed384c82ebb5f9e

Run ID: cf24 was successful after 15s
```

Pour confirmer que l’image a fait l’objet d’un envoi (push), recherchez la balise (`cf24` dans cet exemple) dans le dépôt privé Docker Hub.

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
[az-identity-create]: /cli/azure/identity#az-identity-create
[az-identity-show]: /cli/azure/identity#az-identity-show
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-task-create]: /cli/azure/acr/task#az-acr-task-create
[az-acr-task-show]: /cli/azure/acr/task#az-acr-task-show
[az-acr-task-run]: /cli/azure/acr/task#az-acr-task-run
[az-acr-task-list-runs]: /cli/azure/acr/task#az-acr-task-list-runs
[az-acr-task-credential-add]: /cli/azure/acr/task/credential#az-acr-task-credential-add
[az-group-create]: /cli/azure/group?#az-group-create
[az-keyvault-create]: /cli/azure/keyvault?#az-keyvault-create
[az-keyvault-secret-set]: /cli/azure/keyvault/secret#az-keyvault-secret-set
[az-keyvault-set-policy]: /cli/azure/keyvault#az-keyvault-set-policy
