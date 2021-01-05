---
title: Déployer un modèle Machine Learning sur Azure Functions à l’aide d’Azure Cache pour Redis
description: 'Dans cet article, vous allez déployer un modèle d’Azure Machine Learning en tant qu’application de fonction dans Azure Functions à l’aide d’une instance Azure Cache pour Redis. Azure Cache pour Redis est extrêmement performant et évolutif : lorsqu’il est associé à un modèle Azure Machine Learning, vous obtenez une latence faible et un débit élevé dans votre application.'
author: curib
ms.author: cauribeg
ms.service: cache
ms.topic: conceptual
ms.date: 09/30/2020
ms.openlocfilehash: d9731455edf0afbe4c0768ae40a51316ac71ad94
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92537573"
---
# <a name="deploy-a-machine-learning-model-to-azure-functions-with-azure-cache-for-redis"></a>Déployer un modèle Machine Learning sur Azure Functions à l’aide d’Azure Cache pour Redis 

Dans cet article, vous allez déployer un modèle d’Azure Machine Learning en tant qu’application de fonction dans Azure Functions à l’aide d’une instance Azure Cache pour Redis.  

Azure Cache pour Redis est extrêmement performant et évolutif : lorsqu’il est associé à un modèle Azure Machine Learning, vous obtenez une latence faible et un débit élevé dans votre application. Parmi les scénarios où un cache est particulièrement utile, on peut citer l’inférence des données et les résultats de l’inférence du modèle réel. Dans les deux cas, les métadonnées ou les résultats sont stockés en mémoire, ce qui augmente les performances. 

> [!NOTE]
> Bien qu’Azure Machine Learning et Azure Functions soient mis à la disposition générale, la possibilité d’empaqueter un modèle Machine Learning Service vers Functions est disponible en préversion.  
>

## <a name="prerequisites"></a>Prérequis
* Abonnement Azure : [créez-en un gratuitement](https://azure.microsoft.com/free/).
* Un espace de travail Azure Machine Learning. Pour plus d’informations, consultez l’article [Créer un espace de travail](../machine-learning/how-to-manage-workspace.md).
* [Azure CLI](/cli/azure/install-azure-cli?preserve-view=true&view=azure-cli-latest).
* Un modèle Machine Learning entraîné inscrit dans votre espace de travail. Si vous n’avez pas de modèle, utilisez le [tutoriel de classification d’image : entraîner un modèle](../machine-learning/tutorial-train-models-with-aml.md) pour entraîner et inscrire un modèle.

> [!IMPORTANT]
> Les extraits de code de cet article partent du principe que vous avez défini les variables suivantes :
>
> * `ws` - Votre espace de travail Azure Machine Learning.
> * `model` - Modèle inscrit qui sera déployé.
> * `inference_config` - Configuration d’inférence pour le modèle.
>
> Pour plus d’informations sur la définition de ces variables, consultez [Déployer des modèles avec Azure Machine Learning](../machine-learning/how-to-deploy-and-where.md).

## <a name="create-an-azure-cache-for-redis-instance"></a>Créer une instance Cache Redis Azure 
Vous pourrez déployer un modèle Machine Learning pour Azure Functions avec n’importe quelle instance de cache De base, Standard ou Premium. Pour créer une instance de cache, procédez comme suit.  

1. Accédez à la page d’accueil du portail Azure ou ouvrez le menu latéral, puis sélectionnez **Créer une ressource** . 
   
1. Dans la page **Nouvelle** , sélectionnez **Bases de données** , puis **Azure Cache pour Redis** .

    :::image type="content" source="media/cache-private-link/2-select-cache.png" alt-text="Sélectionnez Azure Cache pour Redis.":::
   
1. Dans la page **Nouveau cache Redis** , configurez les paramètres du nouveau cache.
   
   | Paramètre      | Valeur suggérée  | Description |
   | ------------ |  ------- | -------------------------------------------------- |
   | **Nom DNS** | Entrez un nom globalement unique. | Le nom du cache doit être une chaîne de 1 à 63 caractères ne contenant que des chiffres, des lettres et des traits d’union. Le nom doit commencer et se terminer par un chiffre ou une lettre, et ne peut pas contenir de traits d’union consécutifs. Le *nom d’hôte* de votre instance de cache sera *\<DNS name>.redis.cache.windows.net* . | 
   | **Abonnement** | Dans la liste déroulante, sélectionnez votre abonnement. | Abonnement sous lequel créer cette nouvelle instance d’Azure Cache pour Redis. | 
   | **Groupe de ressources** | Dans la liste déroulante, sélectionnez un groupe de ressources ou choisissez **Créer nouveau** , puis entrez un nouveau nom de groupe de ressources. | Nom du groupe de ressources dans lequel créer votre cache et d’autres ressources. En plaçant toutes les ressources de votre application dans un seul groupe de ressources, vous pouvez facilement les gérer ou les supprimer ensemble. | 
   | **Lieu** | Dans la liste déroulante, sélectionnez un emplacement. | Choisissez une [Région](https://azure.microsoft.com/regions/) proche d’autres services qui utiliseront votre cache. |
   | **Niveau tarifaire** | Sélectionnez un [Niveau tarifaire](https://azure.microsoft.com/pricing/details/cache/). |  Le niveau tarifaire détermine la taille, les performances et les fonctionnalités disponibles pour le cache. Pour plus d’informations, consultez [Présentation du cache Azure pour Redis](cache-overview.md). |

1. Sélectionnez l’onglet **Réseau** ou cliquez sur le bouton **Réseau** au bas de la page.

1. Sous l’onglet **Réseau** , sélectionnez votre méthode de connectivité.

1. Sélectionnez le bouton **Suivant : Avancé** ou cliquez sur le bouton **Suivant : Avancé** en bas de la page.

1. Sous l’onglet **Avancé** d’une instance de cache de base ou standard, sélectionnez Activer/désactiver si vous souhaitez activer un port non-TLS.

1. Sous l’onglet **Avancé** d’une instance de cache premium, configurez les paramètres pour le port non-TLS, le clustering et la persistance des données.

1. Sélectionnez le bouton **Suivant : Étiquettes** ou cliquez sur le bouton **Suivant : Étiquettes** au bas de la page.

1. Si vous le voulez, sous l’onglet **Étiquettes** , entrez le nom et la valeur si vous souhaitez catégoriser la ressource. 

1. Sélectionnez **Revoir + créer** . Vous êtes redirigé vers l’onglet Vérifier + créer où Azure valide votre configuration.

1. Une fois que le message vert Validation réussie s’affiche, sélectionnez **Créer** .

La création du cache prend un certain temps. Vous pouvez surveiller la progression dans la page **Vue d’ensemble** du Azure Cache pour Redis. Lorsque **État** indique **En cours d’exécution** , le cache est prêt pour utilisation. 

## <a name="prepare-for-deployment"></a>Préparer le déploiement

Avant le déploiement, vous devez définir ce qui est nécessaire pour exécuter le modèle en tant que service web. La liste suivante décrit les éléments principaux nécessaires pour un déploiement :

* __Script d’entrée__ . Ce script accepte les requêtes, évalue la requête à l’aide du modèle et renvoie les résultats.

    > [!IMPORTANT]
    > Le script d’entrée est spécifique à votre modèle. Il doit comprendre le format des données de la requête entrante, le format des données attendues par votre modèle et le format des données renvoyées aux clients.
    >
    > Si le format des données de la requête n'est pas utilisable par votre modèle, le script peut les convertir à un format acceptable. Il peut également transformer la réponse avant de la retourner au client.
    >
    > Par défaut, lors de l’empaquetage des fonctions, l’entrée est traitée comme du texte. Si vous souhaitez consommer les octets bruts de l’entrée (par exemple pour les déclencheurs de blob), vous devez utiliser [AMLRequest pour accepter les données brutes](../machine-learning/how-to-deploy-advanced-entry-script.md#binary-data).

Pour la fonction d’exécution, assurez-vous qu’elle se connecte à un point de terminaison Redis.

```python
import json
import numpy as np
import os
import redis
from sklearn.externals import joblib

def init():
    global model
    global azrediscache
    azrediscache = redis.StrictRedis(host='<host_url>', port=6380, password="<access_key>", ssl=True)
    model_path = os.path.join(os.getenv('AZUREML_MODEL_DIR'), 'sklearn_mnist_model.pkl')
    model = joblib.load(model_path)

@input_schema('data', NumpyParameterType(input_sample))
@output_schema(NumpyParameterType(output_sample))
def run(data):
    try:
        input = azrediscache.get(data)
        result = model.predict(input)
        data = np.array(json.loads(data))
        result = model.predict(data)
        # You can return any data type, as long as it is JSON serializable.
        return result.tolist()
    except Exception as e:
        error = str(e)
        return error
```

Pour plus d’informations sur le script d’entrée, consultez [Définir le code de scoring](../machine-learning/how-to-deploy-and-where.md?tabs=python#define-an-entry-script).

* **Dépendances** , comme les scripts d’assistance ou les packages Python/Conda nécessaires à l’exécution du script d’entrée ou du modèle

Ces entités sont encapsulées dans une __configuration d'inférence__ . La configuration d’inférence référence le script d’entrée et d’autres dépendances.

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
>                                                            pip_packages=['azureml-defaults', 'redis'])
> inference_config = InferenceConfig(entry_script="score.py", environment=myenv)
> ```

Pour plus d’informations sur les environnements , consultez [Créer et gérer des environnements pour la formation et le déploiement](../machine-learning/how-to-use-environments.md).

Pour plus d’informations sur la configuration de l’inférence, consultez [Déployer des modèles avec Azure Machine Learning](../machine-learning/how-to-deploy-and-where.md?tabs=python#define-an-inference-configuration).

> [!IMPORTANT]
> Lors du déploiement sur Azure Functions, il n’est pas nécessaire de créer une __configuration de déploiement__ .

## <a name="install-the-sdk-preview-package-for-functions-support"></a>Installer le package de la préversion du SDK pour la prise en charge des fonctions

Pour générer des packages pour Azure Functions, vous devez installer le package de la préversion du SDK.

```bash
pip install azureml-contrib-functions
```

## <a name="create-the-image"></a>Création de l’image

Pour créer l’image Docker qui est déployée sur Azure Functions, utilisez [azureml.contrib.functions.package](/python/api/azureml-contrib-functions/azureml.contrib.functions?preserve-view=true&view=azure-ml-py) ou la fonction de package spécifique pour le déclencheur que vous souhaitez utiliser. L’extrait de code suivant montre comment créer un nouveau package avec un déclencheur HTTP à partir de la configuration du modèle et de l’inférence :

> [!NOTE]
> L’extrait de code suppose que `model` contient un modèle inscrit et que `inference_config` contient la configuration de l’environnement d’inférence. Pour plus d’informations, consultez [Déployer des modèles avec Azure Machine Learning](../machine-learning/how-to-deploy-and-where.md).

```python
from azureml.contrib.functions import package
from azureml.contrib.functions import HTTP_TRIGGER
model_package = package(ws, [model], inference_config, functions_enabled=True, trigger=HTTP_TRIGGER)
model_package.wait_for_creation(show_output=True)
# Display the package location/ACR path
print(model_package.location)
```

Si la condition est `show_output=True`, la sortie du processus de génération Docker s’affiche. Une fois le processus terminé, l’image a été créée dans le registre Azure Container Registry pour votre espace de travail. Une fois que l’image a été créée, son emplacement dans Azure Container Registry s’affiche. L’emplacement est retourné au format `<acrinstance>.azurecr.io/package@sha256:<imagename>`.

> [!NOTE]
> L’empaquetage des fonctions prend actuellement en charge les déclencheurs HTTP, les déclencheurs de blob et les déclencheurs Service Bus. Pour plus d’informations sur les déclencheurs, consultez [Liaisons Azure Functions](../azure-functions/functions-bindings-storage-blob-trigger.md#blob-name-patterns).

> [!IMPORTANT]
> Enregistrez les informations concernant l’emplacement, car vous en aurez besoin lors du déploiement de l’image.

## <a name="deploy-image-as-a-web-app"></a>Déployer une image en tant qu’application web

1. Utilisez la commande suivante pour obtenir les informations d’identification de connexion de l’instance d’Azure Container Registry qui contient l’image. Remplacez `<myacr>` par la valeur retournée précédemment par `package.location` : 

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

    Enregistrez le nom d’utilisateur ( __username__ ), ainsi que l’un des mots de passe ( __passwords__ ).

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
    > À ce stade, l’application de fonction a été créée. Toutefois, étant donné que vous n’avez pas de chaîne de connexion pour le déclencheur HTTP et que vous n’avez pas fourni les informations d’identification à l’instance Azure Container Registry qui contient l’image, l’application de fonction n’est pas active. Dans l’étape qui suit, vous allez fournir la chaîne de connexion et les informations d’authentification pour le registre de conteneurs. 

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

## <a name="test-azure-function-http-trigger"></a>Tester le déclencheur HTTP Azure Functions 

Nous allons maintenant exécuter et tester notre déclencheur HTTP Azure Functions.

1. Accédez à votre application Azure Functions dans le portail Azure.
1. Sous Développeur, sélectionnez **Code + test** . 
1. Sur le côté droit, sélectionnez l’onglet **Entrée** . 
1. Cliquez sur le bouton **Exécuter** pour tester le déclencheur HTTP Azure Functions. 

Vous avez maintenant déployé un modèle d’Azure Machine Learning en tant qu’application de fonction à l’aide d’une instance Azure Cache pour Redis. Pour en savoir plus sur Azure Cache pour Redis, cliquez sur les liens de la section ci-dessous.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous envisagez d’exécuter le didacticiel suivant, vous pouvez conserver les ressources créées dans le cadre de ce démarrage rapide afin de les réutiliser.

Sinon, si vous en avez fini avec le démarrage rapide, vous pouvez supprimer les ressources Azure que vous y avez créées afin d’éviter des frais. 

> [!IMPORTANT]
> La suppression d’un groupe de ressources est irréversible. Quand vous supprimez un groupe de ressources, toutes les ressources qu’il contient sont supprimées définitivement. Veillez à ne pas supprimer accidentellement des ressources ou un groupe de ressources incorrects. Si vous avez créé les ressources pour l’hébergement de cet exemple dans un groupe de ressources existant contenant des ressources que vous souhaitez conserver, vous pouvez supprimer chaque ressource individuellement à partir de son panneau respectif, au lieu de supprimer l’intégralité du groupe de ressources.

### <a name="to-delete-a-resource-group"></a>Pour supprimer un groupe de ressources

1. Connectez-vous au [Portail Azure](https://portal.azure.com), puis sélectionnez **Groupes de ressources** .

2. Dans la zone **Filtrer par nom...** , saisissez le nom de votre groupe de ressources. Sur votre groupe de ressources, dans la liste des résultats, sélectionnez **...** , puis **Supprimer le groupe de ressources** .

Vous êtes invité à confirmer la suppression du groupe de ressources. Saisissez le nom de votre groupe de ressources pour confirmer, puis sélectionnez **Supprimer** .

Après quelques instants, le groupe de ressources et toutes ses ressources sont supprimés.

## <a name="next-steps"></a>Étapes suivantes 

* En savoir plus sur [Azure Cache pour Redis](./cache-overview.md)
* Découvrez comment configurer votre application de fonction dans la documentation sur [Functions](../azure-functions/functions-create-function-linux-custom-image.md).
* [Référence sur l’API](/python/api/azureml-contrib-functions/azureml.contrib.functions?preserve-view=true&view=azure-ml-py) 
* Créer une [application Python qui utilise Azure Cache pour Redis](./cache-python-get-started.md)