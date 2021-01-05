---
title: Déployer des modèles ML sur Azure App Service (préversion)
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser Azure Machine Learning pour déployer un modèle ML formé sur une application web dans Azure App Service.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: aashishb
author: aashishb
ms.reviewer: larryfr
ms.date: 06/23/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python, deploy, devx-track-azurecli
ms.openlocfilehash: 5f3861d43715ed68116d25206efe4347ea96805c
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96452143"
---
# <a name="deploy-a-machine-learning-model-to-azure-app-service-preview"></a>Déployer des modèles Machine Learning sur Azure App Service (préversion)


Découvrez comment déployer un modèle à partir d’Azure Machine Learning en tant qu’application web dans Azure App Service.

> [!IMPORTANT]
> Bien qu’Azure Machine Learning et Azure App Service soient mis à la disposition générale, la possibilité de déployer un modèle du service Machine Learning vers App Service est en préversion.

Avec Azure Machine Learning, vous pouvez créer des images Docker à partir de modèles Machine Learning entraînés. Cette image contient un service web qui reçoit des données, les soumet au modèle, puis retourne la réponse. Azure App Service peut être utilisé pour déployer l’image et fournit les fonctionnalités suivantes :

* [Authentification](../app-service/configure-authentication-provider-aad.md) avancée pour une sécurité améliorée. Les méthodes d’authentification incluent Azure Active Directory et l’authentification multifacteur.
* [Effectuer une mise à l’échelle automatique](../azure-monitor/platform/autoscale-get-started.md?toc=%2fazure%2fapp-service%2ftoc.json) sans nécessité de redéploiement.
* [Prise en charge TLS](../app-service/configure-ssl-certificate-in-code.md) pour les communications sécurisées entre les clients et le service.

Pour plus d’informations sur les fonctionnalités fournies par Azure App Service, consultez la [présentation d’App Service](../app-service/overview.md).

> [!IMPORTANT]
> Si vous avez besoin de la possibilité de consigner les données de scoring utilisées avec votre modèle déployé, ou les résultats de scoring, il est préférable d’effectuer le déploiement vers Azure Kubernetes Service. Pour plus d’informations, consultez [Collecter des données relatives à vos modèles de production](how-to-enable-data-collection.md).

## <a name="prerequisites"></a>Prérequis

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

    > [!IMPORTANT]
    > Le Kit de développement logiciel (SDK) Azure Machine Learning n’offre aucun moyen pour le service web d’accéder à votre magasin de données ou à vos jeux de données. Si vous avez besoin du modèle déployé pour accéder aux données stockées en dehors du déploiement, comme dans un compte de stockage Azure, vous devez développer une solution de code personnalisée à l’aide du Kit de développement logiciel (SDK) approprié. Exemple : [Kit de développement logiciel (SDK) Stockage Azure pour Python](https://github.com/Azure/azure-storage-python).
    >
    > Une autre solution possible pour votre scénario consiste à utiliser les [prédictions par lots](./tutorial-pipeline-batch-scoring-classification.md), qui donnent accès aux magasins de travail lors du scoring.

    Pour plus d’informations sur les scripts d’entrée, consultez [Déployer des modèles avec Azure Machine Learning](how-to-deploy-and-where.md).

* **Dépendances**, comme les scripts d’assistance ou les packages Python/Conda nécessaires à l’exécution du script d’entrée ou du modèle

Ces entités sont encapsulées dans une __configuration d'inférence__. La configuration d’inférence référence le script d’entrée et d’autres dépendances.

> [!IMPORTANT]
> Lors de la création d’une configuration d’inférence à utiliser avec Azure App Service, vous devez utiliser un objet [Environnement](/python/api/azureml-core/azureml.core.environment(class)?preserve-view=true&view=azure-ml-py). Notez que, si vous définissez un environnement personnalisé, vous devez ajouter azureml-defaults avec une version supérieure ou égale à 1.0.45 comme dépendance pip. Ce package contient les fonctionnalités nécessaires pour héberger le modèle en tant que service web. L’exemple suivant illustre la création d’un objet d’environnement et son utilisation avec une configuration d’inférence :
>
> ```python
> from azureml.core.environment import Environment
> from azureml.core.conda_dependencies import CondaDependencies
> from azureml.core.model import InferenceConfig
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
> Lors du déploiement vers Azure App Service, il n’est pas nécessaire de créer une __configuration de déploiement__.

## <a name="create-the-image"></a>Création de l’image

Pour créer l’image du Docker qui est déployée sur Azure App Service, utilisez [Model.package](/python/api/azureml-core/azureml.core.model.model?preserve-view=true&view=azure-ml-py). L’extrait de code suivant montre comment construire une nouvelle image à partir de la configuration du modèle et de l’inférence :

> [!NOTE]
> L’extrait de code suppose que `model` contient un modèle inscrit et que `inference_config` contient la configuration de l’environnement d’inférence. Pour plus d’informations, consultez [Déployer des modèles avec Azure Machine Learning](how-to-deploy-and-where.md).

```python
from azureml.core import Model

package = Model.package(ws, [model], inference_config)
package.wait_for_creation(show_output=True)
# Display the package location/ACR path
print(package.location)
```

Si la condition est `show_output=True`, la sortie du processus de génération Docker s’affiche. Une fois le processus terminé, l’image a été créée dans le registre Azure Container Registry pour votre espace de travail. Une fois que l’image a été créée, son emplacement dans Azure Container Registry s’affiche. L’emplacement est retourné au format `<acrinstance>.azurecr.io/package@sha256:<imagename>`. Par exemple : `myml08024f78fd10.azurecr.io/package@sha256:20190827151241`.

> [!IMPORTANT]
> Enregistrez les informations concernant l’emplacement, car vous en aurez besoin lors du déploiement de l’image.

## <a name="deploy-image-as-a-web-app"></a>Déployer une image en tant qu’application web

1. Utilisez la commande suivante pour obtenir les informations d’identification de connexion de l’instance d’Azure Container Registry qui contient l’image. Remplacez `<acrinstance>` par la valeur retournée précédemment par `package.location` :

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

    Dans cet exemple, le niveau tarifaire __De base__ (`--sku B1`) est utilisé.

    > [!IMPORTANT]
    > Les images créées par Azure Machine Learning utilisent Linux. Vous devez donc utiliser le paramètre `--is-linux`.

1. Pour créer une application web, utilisez la commande suivante. Remplacez `<app-name>` par le nom que vous souhaitez utiliser. Remplacez `<acrinstance>` et `<imagename>` par les valeurs du `package.location` retourné précédemment :

    ```azurecli-interactive
    az webapp create --resource-group myresourcegroup --plan myplanname --name <app-name> --deployment-container-image-name <acrinstance>.azurecr.io/package@sha256:<imagename>
    ```

    Cette commande retourne des informations semblables à celles du document JSON suivant :

    ```json
    {
    "adminSiteName": null,
    "appServicePlanName": "myplanname",
    "geoRegion": "West Europe",
    "hostingEnvironmentProfile": null,
    "id": "/subscriptions/0000-0000/resourceGroups/myResourceGroup/providers/Microsoft.Web/serverfarms/myplanname",
    "kind": "linux",
    "location": "West Europe",
    "maximumNumberOfWorkers": 1,
    "name": "myplanname",
    < JSON data removed for brevity. >
    "targetWorkerSizeId": 0,
    "type": "Microsoft.Web/serverfarms",
    "workerTierName": null
    }
    ```

    > [!IMPORTANT]
    > À ce stade, l’application web a été créée. Toutefois, étant donné que vous n’avez pas fourni les informations d’identification à l’instance Azure Container Registry qui contient l’image, l’application web n’est pas active. Dans l’étape qui suit, vous allez fournir les informations d’authentification du registre de conteneurs.

1. Pour fournir à l’application web les informations d’identification nécessaires pour accéder au registre de conteneurs, utilisez la commande suivante. Remplacez `<app-name>` par le nom que vous souhaitez utiliser. Remplacez `<acrinstance>` et `<imagename>` par les valeurs du `package.location` retourné précédemment. Remplacez `<username>` et `<password>` par les informations de connexion ACR récupérées précédemment :

    ```azurecli-interactive
    az webapp config container set --name <app-name> --resource-group myresourcegroup --docker-custom-image-name <acrinstance>.azurecr.io/package@sha256:<imagename> --docker-registry-server-url https://<acrinstance>.azurecr.io --docker-registry-server-user <username> --docker-registry-server-password <password>
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
        "value": "DOCKER|myml08024f78fd10.azurecr.io/package@sha256:20190827195524"
    }
    ]
    ```

À ce stade, l’application web commence à charger l’image.

> [!IMPORTANT]
> Le chargement de l’image peut prendre plusieurs minutes. Pour superviser la progression, utilisez la commande suivante :
>
> ```azurecli-interactive
> az webapp log tail --name <app-name> --resource-group myresourcegroup
> ```
>
> Une fois que l’image a été chargée et que le site est actif, le journal affiche un message qui indique ceci : `Container <container name> for site <app-name> initialized successfully and is ready to serve requests`.

Une fois l’image déployée, vous pouvez trouver le nom d’hôte à l’aide de la commande suivante :

```azurecli-interactive
az webapp show --name <app-name> --resource-group myresourcegroup
```

Cette commande retourne des informations semblables au nom d’hôte suivant : `<app-name>.azurewebsites.net`. Utilisez cette valeur dans le cadre de l’__URL de base__ du service.

## <a name="use-the-web-app"></a>Utiliser l’application web

Le service web qui transmet les demandes au modèle se trouve à l’emplacement `{baseurl}/score`. Par exemple : `https://<app-name>.azurewebsites.net/score`. L’extrait de code Python suivant montre comment envoyer des données à l’URL et afficher la réponse :

```python
import requests
import json

scoring_uri = "https://mywebapp.azurewebsites.net/score"

headers = {'Content-Type':'application/json'}

test_sample = json.dumps({'data': [
    [1,2,3,4,5,6,7,8,9,10],
    [10,9,8,7,6,5,4,3,2,1]
]})

response = requests.post(scoring_uri, data=test_sample, headers=headers)
print(response.status_code)
print(response.elapsed)
print(response.json())
```

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment configurer votre application web dans la documentation sur [App service sur Linux](/azure/app-service/containers/).
* Découvrez la mise à l’échelle dans [Bien démarrer avec la mise à l’échelle automatique dans Azure](../azure-monitor/platform/autoscale-get-started.md?toc=%2fazure%2fapp-service%2ftoc.json).
* [Utiliser un certificat TLS/SSL dans votre Azure App Service](../app-service/configure-ssl-certificate-in-code.md).
* [Configurer votre application App Service pour utiliser une connexion Azure Active Directory](../app-service/configure-authentication-provider-aad.md).
* [Utiliser un modèle ML déployé en tant que service web](how-to-consume-web-service.md)