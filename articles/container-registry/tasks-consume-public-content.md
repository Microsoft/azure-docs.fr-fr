---
title: Workflow de tâches pour gérer le contenu de registre public
description: Créez un workflow Azure Container Registry Tasks automatisé pour suivre, gérer et utiliser des images publiques dans un registre de conteneurs Azure privé.
author: SteveLasker
ms.topic: article
ms.author: stevelas
ms.date: 10/29/2020
ms.custom: ''
ms.openlocfilehash: 4fba6290b4973e797c13943fc9be4fadb19f3274
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96349280"
---
# <a name="how-to-consume-and-maintain-public-content-with-azure-container-registry-tasks"></a>Comment utiliser et gérer le contenu public à l’aide d’Azure Container Registry Tasks

Cet article fournit un exemple de workflow dans Azure Container Registry pour vous aider à gérer la consommation et la maintenance des contenus publics :

1. Importer des copies locales d’images publiques dépendantes.
1. Valider les images publiques par le biais de l’analyse de sécurité et des tests fonctionnels.
1. Promouvoir les images en registres privés pour une utilisation interne.
1. Déclencher les mises à jour des images de base pour les applications dépendantes de contenus publics.
1. Utiliser [Azure Container Registry Tasks](container-registry-tasks-overview.md) pour automatiser ce workflow.

Le workflow se résume à l’image suivante :

![Workflow d’utilisation de contenus publics](./media/tasks-consume-public-content/consuming-public-content-workflow.png)

Le workflow d’importation contrôlé permet de gérer les dépendances de votre organisation vis-à-vis des artefacts gérés de manière externe, par exemple, les images provenant de registres publics, notamment [Docker Hub][docker-hub], [GCR][gcr], [Quay][quay], [GitHub Container Registry][ghcr], [Microsoft Container Registry][mcr], ou même d’autres [registres de conteneurs Azure][acr]. 

Pour plus d’informations sur les risques introduits par les dépendances vis-à-vis d’un contenu public et sur l’utilisation d’Azure Container Registry pour les atténuer, consultez le [billet de blog OCI Consuming Public Content][oci-consuming-public-content] et [Gérer le contenu public à l’aide d’Azure Container Registry](buffer-gate-public-content.md).

Pour suivre cette procédure, vous pouvez utiliser Azure Cloud Shell ou une installation locale d’Azure CLI. Azure CLI version 2.10 ou ultérieure est recommandé. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI][install-cli].

## <a name="scenario-overview"></a>Présentation du scénario

![Composants d’un workflow d’importation](./media/tasks-consume-public-content/consuming-public-content-objects.png)

Cette procédure pas à pas définit les éléments suivants :

1. Trois **registres de conteneurs**, représentant :
   * Une simulation de [Docker Hub][docker-hub] (`publicregistry`) pour prendre en charge la modification de l’image de base
   * Le registre de l’équipe (`contoso`) pour partager des images privées
   * Le registre partagé de l’entreprise ou de l’équipe (`baseartifacts`) pour le contenu public importé
1. Une **tâche ACR** dans chaque registre. Les tâches :  
   1. créent une simulation d’image `node` publique
   1. importent et valident l’image `node` dans le registre partagé de l’entreprise ou de l’équipe
   1. génèrent et déploient l’image `hello-world`
1. **Les définitions de tâche ACR**, notamment les configurations pour :
1. Une collection d’**informations d’identification de registre**, qui sont des pointeurs vers un coffre de clés
1. Une collection de **secrets**, disponibles dans un fichier `acr-task.yaml`, qui sont des pointeurs vers un coffre de clés
1. Une collection de **valeurs configurées** utilisées dans un fichier `acr-task.yaml`
1. Un **coffre de clés Azure** pour sécuriser tous les secrets
1. Une **instance de conteneur Azure**, qui héberge l’application de build `hello-world`

## <a name="prerequisites"></a>Prérequis

Les étapes suivantes configurent des valeurs pour les ressources créées et utilisées dans la procédure pas à pas.

### <a name="set-environment-variables"></a>Définir des variables d’environnement

Configurez des variables propres à votre environnement. Nous suivons les meilleures pratiques pour placer les ressources ayant un contenu durable dans leur propre groupe de ressources afin de réduire le nombre de suppressions accidentelles. Toutefois, vous pouvez les placer dans un seul groupe de ressources si vous le souhaitez.

Les exemples présentés dans cet article sont mis en forme pour l’interpréteur de commandes Bash.

```bash
# Set the three registry names, must be globally unique:
REGISTRY_PUBLIC=publicregistry
REGISTRY_BASE_ARTIFACTS=contosobaseartifacts
REGISTRY=contoso

# set the location all resources will be created in:
RESOURCE_GROUP_LOCATION=eastus

# default resource groups
REGISTRY_PUBLIC_RG=${REGISTRY_PUBLIC}-rg
REGISTRY_BASE_ARTIFACTS_RG=${REGISTRY_BASE_ARTIFACTS}-rg
REGISTRY_RG=${REGISTRY}-rg

# fully qualified registry urls
REGISTRY_DOCKERHUB_URL=docker.io
REGISTRY_PUBLIC_URL=${REGISTRY_PUBLIC}.azurecr.io
REGISTRY_BASE_ARTIFACTS_URL=${REGISTRY_BASE_ARTIFACTS}.azurecr.io
REGISTRY_URL=${REGISTRY}.azurecr.io

# Azure key vault for storing secrets, name must be globally unique
AKV=acr-task-credentials
AKV_RG=${AKV}-rg

# ACI for hosting the deployed application
ACI=hello-world-aci
ACI_RG=${ACI}-rg
```

### <a name="git-repositories-and-tokens"></a>Référentiels Git et jetons

Pour simuler votre environnement, dupliquez (fork) chacun des référentiels Git suivants dans les référentiels que vous pouvez gérer. 

* https://github.com/importing-public-content/base-image-node.git
* https://github.com/importing-public-content/import-baseimage-node.git
* https://github.com/importing-public-content/hello-world.git

Ensuite, mettez à jour les variables suivantes pour vos référentiels dupliqués.

La variable `:main` ajoutée à la fin des URL Git représente la branche de référentiel par défaut.

```bash
GIT_BASE_IMAGE_NODE=https://github.com/<your-fork>/base-image-node.git#main
GIT_NODE_IMPORT=https://github.com/<your-fork>/import-baseimage-node.git#main
GIT_HELLO_WORLD=https://github.com/<your-fork>/hello-world.git#main
```

Vous avez besoin d’un [jeton d’accès GitHub (PAT)][git-token] pour ACR Tasks afin de cloner et d’établir des webhooks Git. Pour connaître les étapes à suivre pour créer un jeton avec les autorisations requises pour un référentiel privé, consultez [Créer un jeton d’accès GitHub](container-registry-tutorial-build-task.md#create-a-github-personal-access-token). 

```bash
GIT_TOKEN=<set-git-token-here>
```

### <a name="docker-hub-credentials"></a>Informations d’identification Docker Hub  
Pour éviter la limitation de requêtes et les requêtes d’identité lors de l’extraction d’images à partir de Docker Hub, créez un [jeton Docker Hub][docker-hub-tokens]. Ensuite, définissez les variables d’environnement suivantes :

```bash
REGISTRY_DOCKERHUB_USER=<yourusername>
REGISTRY_DOCKERHUB_PASSWORD=<yourtoken>
```

### <a name="create-registries"></a>Créer des registres

À l’aide de commandes Azure CLI, créez trois registres de conteneurs de niveau Premium, chacun dans son propre groupe de ressources :

```azurecli-interactive
az group create --name $REGISTRY_PUBLIC_RG --location $RESOURCE_GROUP_LOCATION
az acr create --resource-group $REGISTRY_PUBLIC_RG --name $REGISTRY_PUBLIC --sku Premium

az group create --name $REGISTRY_BASE_ARTIFACTS_RG --location $RESOURCE_GROUP_LOCATION
az acr create --resource-group $REGISTRY_BASE_ARTIFACTS_RG --name $REGISTRY_BASE_ARTIFACTS --sku Premium

az group create --name $REGISTRY_RG --location $RESOURCE_GROUP_LOCATION
az acr create --resource-group $REGISTRY_RG --name $REGISTRY --sku Premium
```

### <a name="create-key-vault-and-set-secrets"></a>Créer un coffre de clés et définir des secrets

Créez un coffre de clés :

```azurecli-interactive
az group create --name $AKV_RG --location $RESOURCE_GROUP_LOCATION
az keyvault create --resource-group $AKV_RG --name $AKV
```

Définissez le nom d’utilisateur et le jeton Docker Hub dans le coffre de clés :

```azurecli-interactive
az keyvault secret set \
--vault-name $AKV \
--name registry-dockerhub-user \
--value $REGISTRY_DOCKERHUB_USER

az keyvault secret set \
--vault-name $AKV \
--name registry-dockerhub-password \
--value $REGISTRY_DOCKERHUB_PASSWORD
```

Définissez et vérifiez un PAT Git dans le coffre de clés :

```azurecli-interactive
az keyvault secret set --vault-name $AKV --name github-token --value $GIT_TOKEN

az keyvault secret show --vault-name $AKV --name github-token --query value -o tsv
```

### <a name="create-resource-group-for-an-azure-container-instance"></a>Créer un groupe de ressources pour une instance de conteneur Azure

Ce groupe de ressources est utilisé dans une tâche ultérieure lors du déploiement de l’image `hello-world`.

```azurecli-interactive
az group create --name $ACI_RG --location $RESOURCE_GROUP_LOCATION
```

## <a name="create-public-node-base-image"></a>Créer une image de base `node` publique

Pour simuler l’image `node` sur Docker Hub, créez une [tâche ACR][acr-task] pour créer et gérer l’image publique. Cette configuration permet de simuler les modifications apportées par les responsables de l’image `node`.

```azurecli-interactive
az acr task create \
  --name node-public \
  -r $REGISTRY_PUBLIC \
  -f acr-task.yaml \
  --context $GIT_BASE_IMAGE_NODE \
  --git-access-token $(az keyvault secret show \
                        --vault-name $AKV \
                        --name github-token \
                        --query value -o tsv) \
  --set REGISTRY_FROM_URL=${REGISTRY_DOCKERHUB_URL}/ \
  --assign-identity
```

Pour éviter la limitation de Docker, ajoutez des [informations d’identification Docker Hub][docker-hub-tokens] à la tâche. La commande [acr task credentials][acr-task-credentials] peut être utilisée pour transmettre les informations d’identification Docker à n’importe quel registre, y compris Docker Hub.

```azurecli-interactive
az acr task credential add \
  -n node-public \
  -r $REGISTRY_PUBLIC \
  --login-server $REGISTRY_DOCKERHUB_URL \
  -u https://${AKV}.vault.azure.net/secrets/registry-dockerhub-user \
  -p https://${AKV}.vault.azure.net/secrets/registry-dockerhub-password \
  --use-identity [system]
```

Accordez à la tâche l’accès en lecture aux valeurs du coffre de clés :

```azurecli-interactive
az keyvault set-policy \
  --name $AKV \
  --resource-group $AKV_RG \
  --object-id $(az acr task show \
                  --name node-public \
                  --registry $REGISTRY_PUBLIC \
                  --query identity.principalId --output tsv) \
  --secret-permissions get
```

[Les tâches peuvent être déclenchées][acr-task-triggers] par des validations Git, des mises à jour d’images de base, des temporisations ou des exécutions manuelles. 

Exécutez la tâche manuellement pour générer l’image `node` :

```azurecli-interactive
az acr task run -r $REGISTRY_PUBLIC -n node-public
```

Répertoriez l’image dans le registre public simulé :

```azurecli-interactive
az acr repository show-tags -n $REGISTRY_PUBLIC --repository node
```

## <a name="create-the-hello-world-image"></a>Créer l’image `hello-world`

En fonction de la simulation de l’image `node` publique, générez une image `hello-world`.

### <a name="create-token-for-pull-access-to-simulated-public-registry"></a>Créer un jeton pour l’accès pull au registre public simulé

Créez un [jeton d’accès][acr-tokens] pour le registre public simulé, dont la portée est `pull`. Ensuite, définissez-le dans le coffre de clés :

```azurecli-interactive
az keyvault secret set \
  --vault-name $AKV \
  --name "registry-${REGISTRY_PUBLIC}-user" \
  --value "registry-${REGISTRY_PUBLIC}-user"

az keyvault secret set \
  --vault-name $AKV \
  --name "registry-${REGISTRY_PUBLIC}-password" \
  --value $(az acr token create \
              --name "registry-${REGISTRY_PUBLIC}-user" \
              --registry $REGISTRY_PUBLIC \
              --scope-map _repositories_pull \
              -o tsv \
              --query credentials.passwords[0].value)
```

### <a name="create-token-for-pull-access-by-azure-container-instances"></a>Créer un jeton pour l’accès pull d’Azure Container Instances

Créez un [jeton d’accès][acr-tokens] pour le registre hébergeant l’image `hello-world`, dont la portée est « pull ». Ensuite, définissez-le dans le coffre de clés :

```azurecli-interactive
az keyvault secret set \
  --vault-name $AKV \
  --name "registry-${REGISTRY}-user" \
  --value "registry-${REGISTRY}-user"

az keyvault secret set \
  --vault-name $AKV \
  --name "registry-${REGISTRY}-password" \
  --value $(az acr token create \
              --name "registry-${REGISTRY}-user" \
              --registry $REGISTRY \
              --repository hello-world content/read \
              -o tsv \
              --query credentials.passwords[0].value)
```

### <a name="create-task-to-build-and-maintain-hello-world-image"></a>Créer une tâche pour générer et gérer une image `hello-world`

La commande suivante crée une tâche à partir de la définition dans `acr-tasks.yaml` dans le référentiel `hello-world`. Les étapes de la tâche génèrent l’image `hello-world`, puis la déploient sur Azure Container Instances. Le groupe de ressources pour Azure Container Instances a été créé dans une section précédente. En appelant `az container create` dans la tâche avec seulement une différence dans `image:tag`, la tâche est déployée dans la même instance tout au long de cette procédure.

```azurecli-interactive
az acr task create \
  -n hello-world \
  -r $REGISTRY \
  -f acr-task.yaml \
  --context $GIT_HELLO_WORLD \
  --git-access-token $(az keyvault secret show \
                        --vault-name $AKV \
                        --name github-token \
                        --query value -o tsv) \
  --set REGISTRY_FROM_URL=${REGISTRY_PUBLIC_URL}/ \
  --set KEYVAULT=$AKV \
  --set ACI=$ACI \
  --set ACI_RG=$ACI_RG \
  --assign-identity
```

Ajoutez des informations d’identification à la tâche pour le registre public simulé :

```azurecli-interactive
az acr task credential add \
  -n hello-world \
  -r $REGISTRY \
  --login-server $REGISTRY_PUBLIC_URL \
  -u https://${AKV}.vault.azure.net/secrets/registry-${REGISTRY_PUBLIC}-user \
  -p https://${AKV}.vault.azure.net/secrets/registry-${REGISTRY_PUBLIC}-password \
  --use-identity [system]
```

Accordez à la tâche l’accès en lecture aux valeurs du coffre de clés :

```azurecli-interactive
az keyvault set-policy \
  --name $AKV \
  --resource-group $AKV_RG \
  --object-id $(az acr task show \
                  --name hello-world \
                  --registry $REGISTRY \
                  --query identity.principalId --output tsv) \
  --secret-permissions get
```

Accordez l’accès à la tâche pour créer et gérer Azure Container Instances en accordant l’accès au groupe de ressources :

```azurecli-interactive
az role assignment create \
  --assignee $(az acr task show \
  --name hello-world \
  --registry $REGISTRY \
  --query identity.principalId --output tsv) \
  --scope $(az group show -n $ACI_RG --query id -o tsv) \
  --role owner
```

Une fois la tâche créée et configurée, exécutez la tâche pour générer et déployer l’image `hello-world` :

```azurecli-interactive
az acr task run -r $REGISTRY -n hello-world
```

Une fois qu’elle a été créée, récupérez l’adresse IP du conteneur hébergeant l’image `hello-world`.

```azurecli-interactive
az container show \
  --resource-group $ACI_RG \
  --name ${ACI} \
  --query ipAddress.ip \
  --out tsv
```

Dans votre navigateur, accédez à l’adresse IP pour voir l’application en cours d’exécution.

## <a name="update-the-base-image-with-a-questionable-change"></a>Mettre à jour l’image de base avec une modification « douteuse »

Cette section simule une modification de l’image de base qui peut provoquer des problèmes dans l’environnement.

1. Ouvrez `Dockerfile` dans le référentiel `base-image-node` dupliqué.
1. Définissez `BACKGROUND_COLOR` sur `Orange` pour simuler la modification.

```Dockerfile
ARG REGISTRY_NAME=
FROM ${REGISTRY_NAME}node:15-alpine
ENV NODE_VERSION 15-alpine
ENV BACKGROUND_COLOR Orange
```

Validez la modification et vérifiez que les tâches ACR se créent automatiquement.

Exécutez une commande watch pour que la tâche commence à s’exécuter :

```azurecli-interactive
watch -n1 az acr task list-runs -r $REGISTRY_PUBLIC -o table
```

Vous devez finalement voir STATUS `Succeeded` en fonction d’une valeur `Commit` pour TRIGGER :

```azurecli-interactive
RUN ID    TASK      PLATFORM    STATUS     TRIGGER    STARTED               DURATION
--------  --------  ----------  ---------  ---------  --------------------  ----------
ca4       hub-node  linux       Succeeded  Commit     2020-10-24T05:02:29Z  00:00:22
```

Appuyez sur **Ctrl+C** pour quitter la commande espion, puis consultez les journaux de la dernière exécution :

```azurecli-interactive
az acr task logs -r $REGISTRY_PUBLIC
```

Une fois que l’image `node` a terminé, exécutez une commande `watch` pour que les tâches ACR commencent automatiquement à générer l’image `hello-world` :

```azurecli-interactive
watch -n1 az acr task list-runs -r $REGISTRY -o table
```

Vous devez finalement voir STATUS `Succeeded` en fonction d’une valeur `Image Update` pour TRIGGER :

```azurecli-interactive
RUN ID    TASK         PLATFORM    STATUS     TRIGGER       STARTED               DURATION
--------  -----------  ----------  ---------  ------------  --------------------  ----------
dau       hello-world  linux       Succeeded  Image Update  2020-10-24T05:08:45Z  00:00:31
```

Appuyez sur **Ctrl+C** pour quitter la commande espion, puis consultez les journaux de la dernière exécution :

```azurecli-interactive
az acr task logs -r $REGISTRY
```

Une fois l’opération terminée, récupérez l’adresse IP du site hébergeant l’image `hello-world` mise à jour :

```azurecli-interactive
az container show \
  --resource-group $ACI_RG \
  --name ${ACI} \
  --query ipAddress.ip \
  --out tsv
```

Dans votre navigateur, accédez au site, qui doit avoir un arrière-plan orange (paramètre douteux).

### <a name="checking-in"></a>Enregistrement

À ce stade, vous avez créé une image `hello-world` qui est automatiquement générée sur les validations Git et les modifications apportées à l’image `node` de base. Dans cet exemple, la tâche est générée par rapport à une image de base dans Azure Container Registry, mais vous pouvez utiliser n’importe quel registre pris en charge.

La mise à jour de l’image de base redéclenche automatiquement l’exécution de la tâche lors de la mise à jour de l’image `node`. Comme nous l’avons vu ici, toutes les mises à jour ne sont pas voulues.

## <a name="gated-imports-of-public-content"></a>Importations par portail du contenu public

Pour empêcher que des modifications en amont n’arrêtent des charges de travail critiques, il est possible d’ajouter des tests fonctionnels et une analyse de sécurité.

Dans cette section, vous allez créer une tâche ACR pour effectuer les opérations suivantes :

* Créer une image de test
* Exécuter un script de test fonctionnel `./test.sh` sur l’image de test
* Si l’image réussit les tests, importez l’image publique dans le registre **baseimages**

### <a name="add-automation-testing"></a>Ajouter des tests d’automatisation

Pour contrôler tout contenu en amont, des tests automatisés sont mis en place. Dans cet exemple, un `test.sh` est fourni qui vérifie le paramètre `$BACKGROUND_COLOR`. Si le test échoue, un `EXIT_CODE` de `1` est retourné, ce qui entraîne l’échec de cette étape de tâche ACR et met fin à l’exécution de la tâche. Les tests peuvent être développés avec n’importe quelle forme d’outils, y compris la journalisation des résultats. La porte est gérée par une réponse de type « réussite/échec » dans le script, reproduite ici :

```bash
if [ ""$(echo $BACKGROUND_COLOR | tr '[:lower:]' '[:upper:]') = 'RED' ]; then
    echo -e "\e[31mERROR: Invalid Color:\e[0m" ${BACKGROUND_COLOR}
    EXIT_CODE=1
else
  echo -e "\e[32mValidation Complete - No Known Errors\e[0m"
fi
exit ${EXIT_CODE}
```
### <a name="task-yaml"></a>YAML de la tâche 

Vérifiez `acr-task.yaml` dans le référentiel `import-baseimage-node`, qui effectue les étapes suivantes :

1. Générer l’image de base de test à l’aide des Dockerfile suivants :
    ```dockerfile
    ARG REGISTRY_FROM_URL=
    FROM ${REGISTRY_FROM_URL}node:15-alpine
    WORKDIR /test
    COPY ./test.sh .
    CMD ./test.sh
    ```
1. Une fois l’opération terminée, valider l’image en exécutant le conteneur, ce qui s’exécute `./test.sh`
1. Uniquement en cas de réussite de l’opération, exécutez les étapes d’importation, lesquelles sont contrôlées avec `when: ['validate-base-image']`

```yaml
version: v1.1.0
steps:
  - id: build-test-base-image
    # Build off the base image we'll track
    # Add a test script to do unit test validations
    # Note: the test validation image isn't saved to the registry
    # but the task logs captures log validation results
    build: >
      --build-arg REGISTRY_FROM_URL={{.Values.REGISTRY_FROM_URL}}
      -f ./Dockerfile
      -t {{.Run.Registry}}/node-import:test
      .
  - id: validate-base-image
    # only continues if node-import:test returns a non-zero code
    when: ['build-test-base-image']
    cmd: "{{.Run.Registry}}/node-import:test"
  - id: pull-base-image
    # import the public image to base-artifacts
    # Override the stable tag,
    # and create a unique tag to enable rollback
    # to a previously working image
    when: ['validate-base-image']
    cmd: >
        docker pull {{.Values.REGISTRY_FROM_URL}}node:15-alpine
  - id: retag-base-image
    when: ['pull-base-image']
    cmd: docker tag {{.Values.REGISTRY_FROM_URL}}node:15-alpine {{.Run.Registry}}/node:15-alpine
  - id: retag-base-image-unique-tag
    when: ['pull-base-image']
    cmd: docker tag {{.Values.REGISTRY_FROM_URL}}node:15-alpine {{.Run.Registry}}/node:15-alpine-{{.Run.ID}}
  - id: push-base-image
    when: ['retag-base-image', 'retag-base-image-unique-tag']
    push:
    - "{{.Run.Registry}}/node:15-alpine"
    - "{{.Run.Registry}}/node:15-alpine-{{.Run.ID}}"
```

### <a name="create-task-to-import-and-test-base-image"></a>Créer une tâche pour importer et tester l’image de base

```azurecli-interactive
  az acr task create \
  --name base-import-node \
  -f acr-task.yaml \
  -r $REGISTRY_BASE_ARTIFACTS \
  --context $GIT_NODE_IMPORT \
  --git-access-token $(az keyvault secret show \
                        --vault-name $AKV \
                        --name github-token \
                        --query value -o tsv) \
  --set REGISTRY_FROM_URL=${REGISTRY_PUBLIC_URL}/ \
  --assign-identity
```

Ajoutez des informations d’identification à la tâche pour le registre public simulé :

```azurecli-interactive
az acr task credential add \
  -n base-import-node \
  -r $REGISTRY_BASE_ARTIFACTS \
  --login-server $REGISTRY_PUBLIC_URL \
  -u https://${AKV}.vault.azure.net/secrets/registry-${REGISTRY_PUBLIC}-user \
  -p https://${AKV}.vault.azure.net/secrets/registry-${REGISTRY_PUBLIC}-password \
  --use-identity [system]
```

Accordez à la tâche l’accès en lecture aux valeurs du coffre de clés :

```azurecli-interactive
az keyvault set-policy \
  --name $AKV \
  --resource-group $AKV_RG \
  --object-id $(az acr task show \
                  --name base-import-node \
                  --registry $REGISTRY_BASE_ARTIFACTS \
                  --query identity.principalId --output tsv) \
  --secret-permissions get
```

Exécutez la tâche d’importation :

```azurecli-interactive
az acr task run -n base-import-node -r $REGISTRY_BASE_ARTIFACTS
```

> [!NOTE]
> Si la tâche échoue en raison de `./test.sh: Permission denied`, assurez-vous que le script dispose des autorisations d’exécution et validez dans le référentiel Git :
>```bash
>chmod +x ./test.sh
>```

## <a name="update-hello-world-image-to-build-from-gated-node-image"></a>Mettre à jour l’image `hello-world` à générer à partir d’une image `node` contrôlée

Créez un [jeton d’accès][acr-tokens] pour accéder au registre « base-artifacts », limité à `read` du référentiel `node`. Ensuite, définissez-le dans le coffre de clés :

```azurecli-interactive
az keyvault secret set \
  --vault-name $AKV \
  --name "registry-${REGISTRY_BASE_ARTIFACTS}-user" \
  --value "registry-${REGISTRY_BASE_ARTIFACTS}-user"

az keyvault secret set \
  --vault-name $AKV \
  --name "registry-${REGISTRY_BASE_ARTIFACTS}-password" \
  --value $(az acr token create \
              --name "registry-${REGISTRY_BASE_ARTIFACTS}-user" \
              --registry $REGISTRY_BASE_ARTIFACTS \
              --repository node content/read \
              -o tsv \
              --query credentials.passwords[0].value)
```

Ajoutez des informations d’identification à la tâche **hello-world** pour le registre « base-artifacts » :

```azurecli-interactive
az acr task credential add \
  -n hello-world \
  -r $REGISTRY \
  --login-server $REGISTRY_BASE_ARTIFACTS_URL \
  -u https://${AKV}.vault.azure.net/secrets/registry-${REGISTRY_BASE_ARTIFACTS}-user \
  -p https://${AKV}.vault.azure.net/secrets/registry-${REGISTRY_BASE_ARTIFACTS}-password \
  --use-identity [system]
```

Mettez à jour la tâche pour modifier `REGISTRY_FROM_URL` afin d’utiliser le registre `BASE_ARTIFACTS`.

```azurecli-interactive
az acr task update \
  -n hello-world \
  -r $REGISTRY \
  --set KEYVAULT=$AKV \
  --set REGISTRY_FROM_URL=${REGISTRY_BASE_ARTIFACTS_URL}/ \
  --set ACI=$ACI \
  --set ACI_RG=$ACI_RG
```

Exécutez la tâche **hello-world** pour modifier sa dépendance à l’image de base :

```azurecli-interactive
az acr task run -r $REGISTRY -n hello-world
```

## <a name="update-the-base-image-with-a-valid-change"></a>Mettre à jour l’image de base avec une modification « valide »

1. Ouvrez le `Dockerfile` dans le référentiel `base-image-node`.
1. Définissez `BACKGROUND_COLOR` sur `Green` pour simuler une modification valide.

```Dockerfile
ARG REGISTRY_NAME=
FROM ${REGISTRY_NAME}node:15-alpine
ENV NODE_VERSION 15-alpine
ENV BACKGROUND_COLOR Green
```

Validez la modification et surveillez la séquence des mises à jour :

```azurecli-interactive
watch -n1 az acr task list-runs -r $REGISTRY_PUBLIC -o table
```

Lorsque l’exécution est lancée, appuyez sur **Ctrl+C** et surveillez les journaux :

```azurecli-interactive
az acr task logs -r $REGISTRY_PUBLIC
```

Une fois l’opération terminée, surveillez la tâche **base-image-import** :

```azurecli-interactive
watch -n1 az acr task list-runs -r $REGISTRY_BASE_ARTIFACTS -o table
```

Lorsque l’exécution est lancée, appuyez sur **Ctrl+C** et surveillez les journaux :

```azurecli-interactive
az acr task logs -r $REGISTRY_BASE_ARTIFACTS
```

Une fois l’opération terminée, surveillez la tâche **hello-world** :

```azurecli-interactive
watch -n1 az acr task list-runs -r $REGISTRY -o table
```

Lorsque l’exécution est lancée, appuyez sur **Ctrl+C** et surveillez les journaux :

```azurecli-interactive
az acr task logs -r $REGISTRY
```

Une fois l’opération terminée, récupérez l’adresse IP du site hébergeant l’image `hello-world` mise à jour :

```azurecli-interactive
az container show \
  --resource-group $ACI_RG \
  --name ${ACI} \
  --query ipAddress.ip \
  --out tsv
```

Dans votre navigateur, accédez au site, qui doit avoir un arrière-plan vert (paramètre valide).

### <a name="view-the-gated-workflow"></a>Afficher le workflow contrôlé

Effectuez à nouveau les étapes de la section précédente, avec une couleur d’arrière-plan rouge.

1. Ouvrez le `Dockerfile` dans le référentiel `base-image-node`.
1. Définissez `BACKGROUND_COLOR` sur `Red` pour simuler une modification non valide.

```Dockerfile
ARG REGISTRY_NAME=
FROM ${REGISTRY_NAME}node:15-alpine
ENV NODE_VERSION 15-alpine
ENV BACKGROUND_COLOR Red
```

Validez la modification et surveillez la séquence des mises à jour :

```azurecli-interactive
watch -n1 az acr task list-runs -r $REGISTRY_PUBLIC -o table
```

Lorsque l’exécution est lancée, appuyez sur **Ctrl+C** et surveillez les journaux :

```azurecli-interactive
az acr task logs -r $REGISTRY_PUBLIC
```

Une fois l’opération terminée, surveillez la tâche **base-image-import** :

```azurecli-interactive
watch -n1 az acr task list-runs -r $REGISTRY_BASE_ARTIFACTS -o table
```

Lorsque l’exécution est lancée, appuyez sur **Ctrl+C** et surveillez les journaux :

```azurecli-interactive
az acr task logs -r $REGISTRY_BASE_ARTIFACTS
```

À ce stade, vous devez constater l’échec de la validation de la tâche **base-import-node** et arrêter la séquence de publication d’une mise à jour de `hello-world`. Le résultat se présente ainsi :

```console
[...]
2020/10/30 03:57:39 Launching container with name: validate-base-image
Validating Image
NODE_VERSION: 15-alpine
BACKGROUND_COLOR: Red
ERROR: Invalid Color: Red
2020/10/30 03:57:40 Container failed during run: validate-base-image. No retries remaining.
failed to run step ID: validate-base-image: exit status 1
```

### <a name="publish-an-update-to-hello-world"></a>Publier une mise à jour sur `hello-world`

Les modifications apportées à l’image `hello-world` continueront à utiliser l’image `node` validée la plus récente.

Toutes les modifications supplémentaires apportées à l’image `node` de base qui réussissent les validations contrôlées déclencheront des mises à jour d’image de base sur l’image `hello-world`.

## <a name="cleaning-up"></a>Nettoyage

Quand vous n’en avez plus besoin, supprimez les ressources utilisées dans cet article.

```azurecli-interactive
az group delete -n $REGISTRY_RG --no-wait -y
az group delete -n $REGISTRY_PUBLIC_RG --no-wait -y
az group delete -n $REGISTRY_BASE_ARTIFACTS_RG --no-wait -y
az group delete -n $AKV_RG --no-wait -y
az group delete -n $ACI_RG --no-wait -y
```

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez utilisé des tâches ACR pour créer un workflow contrôlé automatisé afin d’introduire des images de base mises à jour dans votre environnement. Consultez les informations connexes pour gérer des images dans Azure Container Registry.


* [Suggestions pour la création d’étiquettes et de versions pour les images conteneur](container-registry-image-tag-version.md)
* [Verrouiller une image conteneur dans un registre de conteneurs Azure](container-registry-image-lock.md)

[install-cli]:                  /cli/azure/install-azure-cli
[acr]:                          https://aka.ms/acr
[acr-repo-permissions]:         ./container-registry-repository-scoped-permissions.md
[acr-task]:                     ./container-registry-tasks-overview.md
[acr-task-triggers]:            container-registry-tasks-overview.md#task-scenarios
[acr-task-credentials]:       container-registry-tasks-authentication-managed-identity.md#4-optional-add-credentials-to-the-task
[acr-tokens]:                   ./container-registry-repository-scoped-permissions.md
[aci]:                          https://aka.ms/aci
[alpine-public-image]:          https://hub.docker.com/_/alpine
[docker-hub]:                   https://hub.docker.com
[docker-hub-tokens]:            https://hub.docker.com/settings/security
[git-token]:                    https://github.com/settings/tokens
[gcr]:                          https://cloud.google.com/container-registry
[ghcr]:                         https://docs.github.com/en/free-pro-team@latest/packages/getting-started-with-github-container-registry/about-github-container-registry
[helm-charts]:                  https://helm.sh
[mcr]:                          https://aka.ms/mcr
[nginx-public-image]:           https://hub.docker.com/_/nginx
[oci-artifacts]:                ./container-registry-oci-artifacts.md
[oci-consuming-public-content]: https://opencontainers.org/posts/blog/2020-10-30-consuming-public-content/
[opa]:                          https://www.openpolicyagent.org/
[quay]:                         https://quay.io