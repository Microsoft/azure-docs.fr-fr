---
title: Déployer des modèles ML sur Azure Functions Apps (préversion)
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser Azure Machine Learning pour empaqueter et déployer un modèle en tant que service web dans une application Azure Functions.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: vaidyas
author: vaidya-s
ms.reviewer: larryfr
ms.date: 03/06/2020
ms.topic: conceptual
ms.custom: how-to, racking-python, devx-track-azurecli
ms.openlocfilehash: 901e4d458cc2d77d4e7f13c1782b86c8532ca499
ms.sourcegitcommit: 4295037553d1e407edeb719a3699f0567ebf4293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96327166"
---
# <a name="deploy-a-machine-learning-model-to-azure-functions-preview"></a>Déployer des modèles Machine Learning sur Azure Functions (préversion)


Découvrez comment déployer un modèle à partir d’Azure Machine Learning en tant qu’application de fonction dans Azure App Service.

> [!IMPORTANT]
> Bien qu’Azure Machine Learning et Azure Functions soient mis à la disposition générale, la possibilité d’empaqueter un modèle Machine Learning Service vers Functions est disponible en préversion.

Avec Azure Machine Learning, vous pouvez créer des images Docker à partir de modèles Machine Learning entraînés. Azure Machine Learning dispose à présent de la fonctionnalité en préversion pour créer ces modèles Machine Learning dans des applications de fonction, qui peuvent être [déployées dans Azure Functions](../azure-functions/functions-deployment-technologies.md#docker-container).

## <a name="prerequisites"></a>Conditions préalables requises

* Un espace de travail Azure Machine Learning. Pour plus d’informations, consultez l’article [Créer un espace de travail](how-to-manage-workspace.md).
* [Interface de ligne de commande Azure](/cli/azure/install-azure-cli?preserve-view=true&view=azure-cli-latest).
* Un modèle Machine Learning entraîné inscrit dans votre espace de travail. Si vous n’avez pas de modèle, utilisez le [tutoriel de classification d’image : entraîner un modèle](tutorial-train-models-with-aml.md) pour entraîner et inscrire un modèle.

    > [!IMPORTANT]
    > Les extraits de code de cet article partent du principe que vous avez défini les variables suivantes :
    >
    > * `ws` - Votre espace de travail Azure Machine Learning.
    > * `model` - Modèle inscrit qui sera déployé.
    > * `inference_config` - Configuration d’inférence pour le modèle.
    >
    > Pour plus d’informations sur la définition de ces variables, consultez [Déployer des modèles avec Azure Machine Learning](how-to-deploy-and-where.md).

## <a name="prepare-for-deployment"></a>Préparer le déploiement

Avant le déploiement, vous devez définir ce qui est nécessaire pour exécuter le modèle en tant que service web. La liste suivante décrit les éléments principaux nécessaires pour un déploiement :

* __Script d’entrée__. Ce script accepte les requêtes, évalue la requête à l’aide du modèle et renvoie les résultats.

    > [!IMPORTANT]
    > Le script d’entrée est spécifique à votre modèle. Il doit comprendre le format des données de la requête entrante, le format des données attendues par votre modèle et le format des données renvoyées aux clients.
    >
    > Si le format des données de la requête n'est pas utilisable par votre modèle, le script peut les convertir à un format acceptable. Il peut également transformer la réponse avant de la renvoyer au client.
    >
    > Par défaut, lors de l’empaquetage des fonctions, l’entrée est traitée comme du texte. Si vous souhaitez consommer les octets bruts de l’entrée (par exemple pour les déclencheurs de blob), vous devez utiliser [AMLRequest pour accepter les données brutes](./how-to-deploy-advanced-entry-script.md#binary-data).

Pour plus d’informations sur le script d’entrée, consultez [Définir le code de scoring](./how-to-deploy-and-where.md#define-an-entry-script).

* **Dépendances**, comme les scripts d’assistance ou les packages Python/Conda nécessaires à l’exécution du script d’entrée ou du modèle

Ces entités sont encapsulées dans une __configuration d'inférence__. La configuration d’inférence référence le script d’entrée et d’autres dépendances.

> [!IMPORTANT]
> Lors de la création d’une configuration d’inférence à utiliser avec Azure Functions, vous devez utiliser un objet [Environnement](/python/api/azureml-core/azureml.core.environment%28class%29?preserve-view=true&view=azure-ml-py). Notez que, si vous définissez un environnement personnalisé, vous devez ajouter azureml-defaults avec une version supérieure ou égale à 1.0.45 comme dépendance pip. Ce package contient les fonctionnalités nécessaires pour héberger le modèle en tant que service web. L’exemple suivant illustre la création d’un objet d’environnement et son utilisation avec une configuration d’inférence :
>
> ```python
> from azureml.core.environment import Environment
> from azureml.core.conda_dependencies import CondaDependencies
>
> # Create an environment and add conda dependencies to it
> myenv = Environment(name="myenv")
> # Enable Docker based environment
> myenv.docker.enabled = True
> # Build conda dependencies
> myenv.python.conda_dependencies = CondaDependencies.create(conda_packages=['scikit-learn'],
>                                                            pip_packages=['azureml-defaults'])
> inference_config = InferenceConfig(entry_script="score.py", environment=myenv)
> ```

Pour plus d’informations sur les environnements , consultez [Créer et gérer des environnements pour la formation et le déploiement](how-to-use-environments.md).

Pour plus d’informations sur la configuration de l’inférence, consultez [Déployer des modèles avec Azure Machine Learning](how-to-deploy-and-where.md).

> [!IMPORTANT]
> Lors du déploiement sur Azure Functions, il n’est pas nécessaire de créer une __configuration de déploiement__.

## <a name="install-the-sdk-preview-package-for-functions-support"></a>Installer le package de la préversion du SDK pour la prise en charge des fonctions

Pour générer des packages pour Azure Functions, vous devez installer le package de la préversion du SDK.

```bash
pip install azureml-contrib-functions
```

## <a name="create-the-image"></a>Création de l’image

Pour créer l’image Docker qui est déployée sur Azure Functions, utilisez [azureml.contrib.functions.package](/python/api/azureml-contrib-functions/azureml.contrib.functions?preserve-view=true&view=azure-ml-py) ou la fonction de package spécifique pour le déclencheur que vous souhaitez utiliser. L’extrait de code suivant montre comment créer un nouveau package avec un déclencheur de blob à partir de la configuration du modèle et de l’inférence :

> [!NOTE]
> L’extrait de code suppose que `model` contient un modèle inscrit et que `inference_config` contient la configuration de l’environnement d’inférence. Pour plus d’informations, consultez [Déployer des modèles avec Azure Machine Learning](how-to-deploy-and-where.md).

```python
from azureml.contrib.functions import package
from azureml.contrib.functions import BLOB_TRIGGER
blob = package(ws, [model], inference_config, functions_enabled=True, trigger=BLOB_TRIGGER, input_path="input/{blobname}.json", output_path="output/{blobname}_out.json")
blob.wait_for_creation(show_output=True)
# Display the package location/ACR path
print(blob.location)
```

Si la condition est `show_output=True`, la sortie du processus de génération Docker s’affiche. Une fois le processus terminé, l’image a été créée dans le registre Azure Container Registry pour votre espace de travail. Une fois que l’image a été créée, son emplacement dans Azure Container Registry s’affiche. L’emplacement est retourné au format `<acrinstance>.azurecr.io/package@sha256:<imagename>`.

> [!NOTE]
> L’empaquetage des fonctions prend actuellement en charge les déclencheurs HTTP, les déclencheurs de blob et les déclencheurs Service Bus. Pour plus d’informations sur les déclencheurs, consultez [Liaisons Azure Functions](../azure-functions/functions-bindings-storage-blob-trigger.md#blob-name-patterns).

> [!IMPORTANT]
> Enregistrez les informations concernant l’emplacement, car vous en aurez besoin lors du déploiement de l’image.

## <a name="deploy-image-as-a-web-app"></a>Déployer une image en tant qu’application web

1. Utilisez la commande suivante pour obtenir les informations d’identification de connexion de l’instance d’Azure Container Registry qui contient l’image. Remplacez `<myacr>` par la valeur retournée précédemment par `blob.location` : 

    ```azurecli-interactive
    az acr credential show --name <myacr>
    ```

    La sortie de cette commande ressemble au document JSON suivant :

    ```json
    {
    "passwords": [
        {
        "name": "password",
        "value": "Iv0lRZQ9762LUJrFiffo3P4sWgk4q+nW"
        },
        {
        "name": "password2",
        "value": "=pKCxHatX96jeoYBWZLsPR6opszr==mg"
        }
    ],
    "username": "myml08024f78fd10"
    }
    ```

    Enregistrez le nom d’utilisateur (__username__), ainsi que l’un des mots de passe (__passwords__).

1. Si vous ne disposez pas déjà d’un groupe de ressources ou d’un plan App Service pour déployer le service, les commandes suivantes montrent comment créer ces deux éléments :

    ```azurecli-interactive
    az group create --name myresourcegroup --location "West Europe"
    az appservice plan create --name myplanname --resource-group myresourcegroup --sku B1 --is-linux
    ```

    Cet exemple utilise le niveau tarifaire _Linux De base_ (`--sku B1`).

    > [!IMPORTANT]
    > Les images créées par Azure Machine Learning utilisent Linux. Vous devez donc utiliser le paramètre `--is-linux`.

1. Créez le compte de stockage à utiliser pour le stockage de tâches web et récupérez sa chaîne de connexion. Remplacez `<webjobStorage>` par le nom que vous souhaitez utiliser.

    ```azurecli-interactive
    az storage account create --name <webjobStorage> --location westeurope --resource-group myresourcegroup --sku Standard_LRS
    ```
    ```azurecli-interactive
    az storage account show-connection-string --resource-group myresourcegroup --name <webJobStorage> --query connectionString --output tsv
    ```

1. Pour créer une application de fonction, utilisez la commande suivante. Remplacez `<app-name>` par le nom que vous souhaitez utiliser. Remplacez `<acrinstance>` et `<imagename>` par les valeurs du `package.location` retourné précédemment. Remplacez `<webjobStorage>` par le nom du compte de stockage de l’étape précédente :

    ```azurecli-interactive
    az functionapp create --resource-group myresourcegroup --plan myplanname --name <app-name> --deployment-container-image-name <acrinstance>.azurecr.io/package:<imagename> --storage-account <webjobStorage>
    ```

    > [!IMPORTANT]
    > À ce stade, l’application de fonction a été créée. Toutefois, étant donné que vous n’avez pas de chaîne de connexion pour le déclencheur de blob et que vous n’avez pas fourni les informations d’identification à l’instance Azure Container Registry qui contient l’image, l’application de fonction n’est pas active. Dans l’étape qui suit, vous allez fournir la chaîne de connexion et les informations d’authentification pour le registre de conteneurs. 

1. Créez le compte de stockage à utiliser pour le stockage de déclencheurs blob et récupérez sa chaîne de connexion. Remplacez `<triggerStorage>` par le nom que vous souhaitez utiliser.

    ```azurecli-interactive
    az storage account create --name <triggerStorage> --location westeurope --resource-group myresourcegroup --sku Standard_LRS
    ```
    ```azurecli-interactive
    az storage account show-connection-string --resource-group myresourcegroup --name <triggerStorage> --query connectionString --output tsv
    ```
    Enregistrez cette chaîne de connexion à fournir à l’application de fonction. Nous l’utiliserons plus tard quand nous vous demanderons `<triggerConnectionString>`

1. Créez les conteneurs pour l’entrée et la sortie dans le compte de stockage. Remplacez `<triggerConnectionString>` par la chaîne de connexion retournée précédemment :

    ```azurecli-interactive
    az storage container create -n input --connection-string <triggerConnectionString>
    ```
    ```azurecli-interactive
    az storage container create -n output --connection-string <triggerConnectionString>
    ```

1. Pour associer la chaîne de connexion du déclencheur à l’application de fonction, utilisez la commande suivante. Remplacez `<app-name>` par le nom de la fonction. Remplacez `<triggerConnectionString>` par la chaîne de connexion retournée précédemment :

    ```azurecli-interactive
    az functionapp config appsettings set --name <app-name> --resource-group myresourcegroup --settings "TriggerConnectionString=<triggerConnectionString>"
    ```
1. Vous devez récupérer la balise associée au conteneur créé à l’aide de la commande suivante. Remplacez `<username>` par le nom d’utilisateur retourné précédemment à partir du registre de conteneurs :

    ```azurecli-interactive
    az acr repository show-tags --repository package --name <username> --output tsv
    ```
    Enregistrez la valeur retournée, elle sera utilisée en tant que `imagetag` à l’étape suivante.

1. Pour fournir à l’application de fonction les informations d’identification nécessaires pour accéder au registre de conteneurs, utilisez la commande suivante. Remplacez `<app-name>` par le nom de la fonction. Remplacez `<acrinstance>` et `<imagetag>` par les valeurs de l’appel de l’interface de commande AZ CLI à l’étape précédente. Remplacez `<username>` et `<password>` par les informations de connexion ACR récupérées précédemment :

    ```azurecli-interactive
    az functionapp config container set --name <app-name> --resource-group myresourcegroup --docker-custom-image-name <acrinstance>.azurecr.io/package:<imagetag> --docker-registry-server-url https://<acrinstance>.azurecr.io --docker-registry-server-user <username> --docker-registry-server-password <password>
    ```

    Cette commande retourne des informations semblables à celles du document JSON suivant :

    ```json
    [
    {
        "name": "WEBSITES_ENABLE_APP_SERVICE_STORAGE",
        "slotSetting": false,
        "value": "false"
    },
    {
        "name": "DOCKER_REGISTRY_SERVER_URL",
        "slotSetting": false,
        "value": "https://myml08024f78fd10.azurecr.io"
    },
    {
        "name": "DOCKER_REGISTRY_SERVER_USERNAME",
        "slotSetting": false,
        "value": "myml08024f78fd10"
    },
    {
        "name": "DOCKER_REGISTRY_SERVER_PASSWORD",
        "slotSetting": false,
        "value": null
    },
    {
        "name": "DOCKER_CUSTOM_IMAGE_NAME",
        "value": "DOCKER|myml08024f78fd10.azurecr.io/package:20190827195524"
    }
    ]
    ```

À ce stade, l’application de fonction commence à charger l’image.

> [!IMPORTANT]
> Le chargement de l’image peut prendre plusieurs minutes. Vous pouvez surveiller la progression à l’aide du portail Azure.

## <a name="test-the-deployment"></a>test du déploiement

Une fois l’image chargée et l’application disponible, procédez comme suit pour déclencher l’application :

1. Créez un fichier texte contenant les données qu’attend le fichier score.py. L’exemple suivant fonctionne avec un fichier score.py qui attend un tableau de 10 nombres :

    ```json
    {"data": [[1, 2, 3, 4, 5, 6, 7, 8, 9, 10], [10, 9, 8, 7, 6, 5, 4, 3, 2, 1]]}
    ```

    > [!IMPORTANT]
    > Le format des données dépend de ce qu’attendent votre fichier score.py et votre modèle.

2. Utilisez la commande suivante pour charger ce fichier dans le conteneur d’entrée de l’objet blob de stockage de déclencheur créé précédemment. Remplacez `<file>` par le nom du fichier contenant les données. Remplacez `<triggerConnectionString>` par la chaîne de connexion retournée précédemment. Dans cet exemple, `input` est le nom du conteneur d’entrée créé précédemment. Si vous avez utilisé un autre nom, remplacez cette valeur :

    ```azurecli-interactive
    az storage blob upload --container-name input --file <file> --name <file> --connection-string <triggerConnectionString>
    ```

    Le résultat de cette commande doit ressembler au JSON suivant :

    ```json
    {
    "etag": "\"0x8D7C21528E08844\"",
    "lastModified": "2020-03-06T21:27:23+00:00"
    }
    ```

3. Pour afficher la sortie produite par la fonction, utilisez la commande suivante pour répertorier les fichiers de sortie générés. Remplacez `<triggerConnectionString>` par la chaîne de connexion retournée précédemment. Dans cet exemple, `output` est le nom du conteneur de sortie créé précédemment. Si vous avez utilisé un autre nom, remplacez cette valeur :

    ```azurecli-interactive
    az storage blob list --container-name output --connection-string <triggerConnectionString> --query '[].name' --output tsv
    ```

    La sortie de cette commande doit être similaire à `sample_input_out.json`.

4. Pour télécharger le fichier et inspecter son contenu, utilisez la commande suivante. Remplacez `<file>` par le nom de fichier retourné par la commande précédente. Remplacez `<triggerConnectionString>` par la chaîne de connexion retournée précédemment : 

    ```azurecli-interactive
    az storage blob download --container-name output --file <file> --name <file> --connection-string <triggerConnectionString>
    ```

    Une fois la commande exécutée, ouvrez le fichier. Il contient les données retournées par le modèle.

Pour plus d’informations sur l’utilisation des déclencheurs d’objets blob, voir l’article [Créer une fonction déclenchée par un stockage Blob Azure](../azure-functions/functions-create-storage-blob-triggered-function.md).

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment configurer votre application de fonction dans la documentation sur [Functions](../azure-functions/functions-create-function-linux-custom-image.md).
* Pour en savoir plus sur les déclencheurs de Stockage Blob, consultez [Liaisons Stockage Blob Azure](../azure-functions/functions-bindings-storage-blob.md).
* [Déployez votre modèle sur Azure App Service](how-to-deploy-app-service.md).
* [Utiliser un modèle ML déployé en tant que service web](how-to-consume-web-service.md)
* [Référence sur l’API](/python/api/azureml-contrib-functions/azureml.contrib.functions?preserve-view=true&view=azure-ml-py)
