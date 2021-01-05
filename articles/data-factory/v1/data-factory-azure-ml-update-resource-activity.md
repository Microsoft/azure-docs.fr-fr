---
title: Mettre à jour des modèles d’apprentissage automatique à l’aide d’Azure Data Factory
description: Explique comment créer des pipelines prédictifs avec Azure Data Factory v1 et Azure Machine Learning studio (classique).
services: data-factory
documentationcenter: ''
author: dcstwh
ms.author: weetok
manager: jroth
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/22/2018
ms.openlocfilehash: 556936eb6e8c1c1c2dd1fab4ce7dfc1b648710b7
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96496600"
---
# <a name="updating-azure-machine-learning-studio-classic-models-using-update-resource-activity"></a>Mise à jour des modèles Azure Machine Learning studio (classique) avec l’activité Mettre à jour une ressource

> [!div class="op_single_selector" title1="Activités de transformation"]
> * [Activité Hive](data-factory-hive-activity.md) 
> * [Activité pig](data-factory-pig-activity.md)
> * [Activité MapReduce](data-factory-map-reduce.md)
> * [Activité de diffusion en continu Hadoop](data-factory-hadoop-streaming-activity.md)
> * [Activité Spark](data-factory-spark.md)
> * [Activité Exécution par lots Azure Machine Learning studio (classique)](data-factory-azure-ml-batch-execution-activity.md)
> * [Activité Mettre à jour une ressource Azure Machine Learning studio (classique)](data-factory-azure-ml-update-resource-activity.md)
> * [Activité de procédure stockée](data-factory-stored-proc-activity.md)
> * [Activité U-SQL Data Lake Analytics](data-factory-usql-activity.md)
> * [Activité personnalisée .NET](data-factory-use-custom-activities.md)


> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez [Mettre à jour des modèles d’apprentissage automatique dans Data Factory](../update-machine-learning-models.md).

Cet article vient compléter l’article principal sur l’intégration Azure Data Factory - Azure Machine Learning Studio (classique) : [Créez un pipeline prédictif à l’aide d’Azure Machine Learning Studio (classique) et Azure Data Factory](data-factory-azure-ml-batch-execution-activity.md). Si vous ne l’avez pas encore fait, consultez l’article principal avant de lire cet article. 

## <a name="overview"></a>Vue d’ensemble
Au fil du temps, les modèles prédictifs dans les expériences de scoring Azure Machine Learning Studio (classique) doivent être réentraînés à l’aide de nouveaux jeux de données d’entrée. Une fois que vous avez fini la reformation, vous souhaitez mettre à jour le service web de notation avec le modèle ML reformé. Voici la procédure classique pour permettre le réentraînement et la mise à jour de modèles studio (classique) au moyen de services web :

1. Créer une expérience dans [Azure Machine Learning Studio (classique)](https://studio.azureml.net).
2. Lorsque vous êtes satisfait du modèle, utilisez Azure Machine Learning Studio (classique) pour publier des services web à la fois pour l’**expérience de formation** et l’**expérience prédictive**/de scoring.

Le tableau suivant décrit les services web utilisés dans cet exemple.  Pour plus de détails, consultez [Réentraîner les modèles Azure Machine Learning Studio (classique) programmatiquement](../../machine-learning/classic/retrain-machine-learning-model.md).

- **Service Web de formation** - Reçoit les données d’apprentissage et produit les modèles formés. La sortie de la reformation est un fichier .ilearner dans un stockage d’objets blob Azure. Le **point de terminaison par défaut** est automatiquement créé pour vous lorsque vous publiez l’expérience de formation en tant que service web. Vous pouvez créer d’autres points de terminaison, mais l’exemple utilise uniquement le point de terminaison par défaut.
- **Service Web de notation** - Reçoit des exemples de données sans étiquette et effectue des prédictions. La sortie de la prédiction peut prendre plusieurs formes, comme un fichier .csv ou des lignes dans une base de données Azure SQL, selon la configuration de l’expérience. Le point de terminaison par défaut est automatiquement créé pour vous lorsque vous publiez l’expérience prédictive comme un service web. 

Le schéma suivant illustre la relation entre les points de terminaison d’apprentissage et de scoring dans Azure Machine Learning studio (classique).

![SERVICES WEB](./media/data-factory-azure-ml-batch-execution-activity/web-services.png)

Pour appeler le **service web d’apprentissage**, vous pouvez utiliser **l’activité Exécution par lots Azure Machine Learning studio (classique)** . On appelle un service web d’apprentissage de la même manière qu’un service web Azure Machine Learning studio (classique) (service web de scoring) pour le scoring des données. Les sections précédentes expliquent en détail comment appeler un service web Azure Machine Learning studio (classique) à partir d’un pipeline Azure Data Factory. 

Pour appeler le **service web de scoring**, vous pouvez utiliser **l’activité Mettre à jour une ressource Azure Machine Learning studio (classique)** pour mettre à jour le service web avec le modèle qui vient d’être entraîné. Les exemples suivants fournissent les définitions de service associé : 

## <a name="scoring-web-service-is-a-classic-web-service"></a>Service web de notation est un service web classique
Si le service web de notation est un **service classique web**, créez le deuxième **point de terminaison (qui n’est pas le point de terminaison par défaut) pouvant être mis à jour** à l’aide du portail Azure. Pour connaître les étapes, consultez l’article [Créer des points de terminaison](../../machine-learning/classic/create-endpoint.md). Après avoir créé le point de terminaison non par défaut pouvant être mis à jour, procédez comme suit :

* Cliquez sur **EXÉCUTION PAR LOT** pour obtenir la valeur d’URI pour la propriété JSON **mlEndpoint**.
* Cliquez sur le lien **RESSOURCE DE MISE À JOUR** pour obtenir la valeur d’URI pour la propriété JSON **updateResourceEndpoint**. La clé API est sur la page du point de terminaison même (dans le coin inférieur droit).

![point de terminaison pouvant être mis à jour](./media/data-factory-azure-ml-batch-execution-activity/updatable-endpoint.png)

L’exemple suivant présente un exemple de définition JSON pour le service lié AzureML. Le service lié utilise apiKey pour l’authentification.  

```json
{
    "name": "updatableScoringEndpoint2",
    "properties": {
        "type": "AzureML",
        "typeProperties": {
            "mlEndpoint": "https://ussouthcentral.services.azureml.net/workspaces/xxx/services/--scoring experiment--/jobs",
            "apiKey": "endpoint2Key",
            "updateResourceEndpoint": "https://management.azureml.net/workspaces/xxx/webservices/--scoring experiment--/endpoints/endpoint2"
        }
    }
}
```

## <a name="scoring-web-service-is-azure-resource-manager-web-service"></a>Le service web de notification est un service web Azure Resource Manager 
Si le service web est un nouveau type de service web qui expose un point de terminaison Azure Resource Manager, vous n’avez pas besoin ajouter le second point de terminaison, **qui n’est pas celui par défaut** . Le **updateResourceEndpoint** du service lié est au format : 

```
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resource-group-name}/providers/Microsoft.MachineLearning/webServices/{web-service-name}?api-version=2016-05-01-preview. 
```

Vous pouvez obtenir des valeurs pour les espaces réservés dans l’URL lors de l’interrogation du service web sur le [portail des services web Azure Machine Learning Studio (classique)](https://services.azureml.net/). Le nouveau type de point de terminaison de ressource de mise à jour requiert un jeton AAD (Azure Active Directory). Spécifiez **servicePrincipalId** et **servicePrincipalKey** dans le service lié studio (classique). Consultez [how to create service principal and assign permissions to manage Azure resource](../../active-directory/develop/howto-create-service-principal-portal.md) (comment créer le principal de service et affecter des autorisations de gestion de ressources Azure). Voici un exemple de définition de service lié AzureML : 

```json
{
    "name": "AzureMLLinkedService",
    "properties": {
        "type": "AzureML",
        "description": "The linked service for AML web service.",
        "typeProperties": {
            "mlEndpoint": "https://ussouthcentral.services.azureml.net/workspaces/0000000000000000000000000000000000000/services/0000000000000000000000000000000000000/jobs?api-version=2.0",
            "apiKey": "xxxxxxxxxxxx",
            "updateResourceEndpoint": "https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myRG/providers/Microsoft.MachineLearning/webServices/myWebService?api-version=2016-05-01-preview",
            "servicePrincipalId": "000000000-0000-0000-0000-0000000000000",
            "servicePrincipalKey": "xxxxx",
            "tenant": "mycompany.com"
        }
    }
}
```

Le scénario suivant fournit plus de détails. Il comporte un exemple de réentraînement et de mise à jour de modèles studio (classique) à partir d’un pipeline Azure Data Factory.

## <a name="scenario-retraining-and-updating-a-studio-classic-model"></a>Scénario : réentraînement et mise à jour d’un modèle studio (classique)
Cette section fournit un exemple de pipeline qui utilise **l’activité d’exécution par lot Azure Machine Learning Studio (classique)** pour réentraîner un modèle. Le pipeline utilise également l’**activité des ressources de mise à jour Azure Machine Learning Studio (classique)** pour mettre à jour le modèle dans le service web de notation. La section fournit également des extraits de code JSON pour tous les services liés, jeux de données et éléments de pipeline dans l’exemple.

Voici la vue schématique de l’exemple de pipeline. Comme vous pouvez le voir, l’activité Exécution par lots studio (classique) prend l’entrée d’apprentissage et génère une sortie d’apprentissage (fichier iLearner). L’activité Mettre à jour une ressource studio (classique) récupère cette sortie d’apprentissage et met à jour le modèle dans le point de terminaison de service web de scoring. L’activité des ressources de mise à jour ne génère aucune sortie. placeholderBlob est simplement un jeu de données de sortie factice requis par le service Azure Data Factory pour exécuter le pipeline.

![schéma du pipeline](./media/data-factory-azure-ml-batch-execution-activity/update-activity-pipeline-diagram.png)

### <a name="azure-blob-storage-linked-service"></a>Service lié Azure Blob Storage :
Azure Storage contient les données suivantes :

* Données de formation. Les données d’entrée du service web d’apprentissage studio (classique).  
* Fichier iLearner. La sortie du service web d’apprentissage studio (classique). Ce fichier est également l’entrée de l’activité des ressources de mise à jour.  

Voici la définition d’exemple JSON du service lié :

```JSON
{
    "name": "StorageLinkedService",
      "properties": {
        "type": "AzureStorage",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=name;AccountKey=key"
        }
    }
}
```

### <a name="training-input-dataset"></a>Jeu de données d’entrée de formation
Le jeu de données suivant représente les données d’apprentissage d’entrée du service web d’apprentissage studio (classique). L’activité Exécution par lots studio (classique) prend ce jeu de données en entrée.

```JSON
{
    "name": "trainingData",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "StorageLinkedService",
        "typeProperties": {
            "folderPath": "labeledexamples",
            "fileName": "labeledexamples.arff",
            "format": {
                "type": "TextFormat"
            }
        },
        "availability": {
            "frequency": "Week",
            "interval": 1
        },
        "policy": {          
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```

### <a name="training-output-dataset"></a>Jeu de données de sortie de formation :
Le jeu de données suivant représente le fichier iLearner de sortie du service web d’apprentissage Azure Machine Learning studio (classique). C’est l’activité Exécution par lots Azure Machine Learning studio (classique) qui produit ce jeu de données. Il constitue également l’entrée de l’activité Mettre à jour une ressource Azure Machine Learning studio (classique).

```JSON
{
    "name": "trainedModelBlob",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "StorageLinkedService",
        "typeProperties": {
            "folderPath": "trainingoutput",
            "fileName": "model.ilearner",
            "format": {
                "type": "TextFormat"
            }
        },
        "availability": {
            "frequency": "Week",
            "interval": 1
        }
    }
}
```

### <a name="linked-service-for-studio-classic-training-endpoint"></a>Service lié pour le point de terminaison d’apprentissage Azure Machine Learning studio (classique)
L’extrait de code JSON suivant définit un service lié studio (classique) qui pointe vers le point de terminaison par défaut du service web d’apprentissage.

```JSON
{    
    "name": "trainingEndpoint",
      "properties": {
        "type": "AzureML",
        "typeProperties": {
            "mlEndpoint": "https://ussouthcentral.services.azureml.net/workspaces/xxx/services/--training experiment--/jobs",
              "apiKey": "myKey"
        }
      }
}
```

Dans **Azure Machine Learning Studio (classique)** , procédez comme suit afin d’obtenir les valeurs pour **mlEndpoint** et **apiKey** :

1. Cliquez sur **SERVICES WEB** dans le menu de gauche.
2. Cliquez sur le **service web de formation** dans la liste des services web.
3. Cliquez sur Copier regard de la zone de texte **Clé API** . Collez la clé copiée dans l’éditeur JSON Data Factory.
4. Dans **Azure Machine Learning Studio (classique)** , cliquez sur le lien **EXÉCUTION PAR LOTS**.
5. Copiez l’**URI de demande** à partir de la section **Demande**, et collez-le dans l’éditeur JSON Data Factory.   

### <a name="linked-service-for-studio-classic-updatable-scoring-endpoint"></a>Service lié du point de terminaison de scoring studio (classique) pouvant être mis à jour :
L’extrait de code JSON suivant définit un service lié studio (classique) qui pointe vers le point de terminaison (autre que le point de terminaison par défaut) pouvant être mis à jour du service web de scoring.  

```JSON
{
    "name": "updatableScoringEndpoint2",
    "properties": {
        "type": "AzureML",
        "typeProperties": {
            "mlEndpoint": "https://ussouthcentral.services.azureml.net/workspaces/00000000eb0abe4d6bbb1d7886062747d7/services/00000000026734a5889e02fbb1f65cefd/jobs?api-version=2.0",
            "apiKey": "sooooooooooh3WvG1hBfKS2BNNcfwSO7hhY6dY98noLfOdqQydYDIXyf2KoIaN3JpALu/AKtflHWMOCuicm/Q==",
            "updateResourceEndpoint": "https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/Default-MachineLearning-SouthCentralUS/providers/Microsoft.MachineLearning/webServices/myWebService?api-version=2016-05-01-preview",
            "servicePrincipalId": "fe200044-c008-4008-a005-94000000731",
            "servicePrincipalKey": "zWa0000000000Tp6FjtZOspK/WMA2tQ08c8U+gZRBlw=",
            "tenant": "mycompany.com"
        }
    }
}
```

### <a name="placeholder-output-dataset"></a>Jeu de données de sortie de l’espace réservé
L’activité Mettre à jour une ressource studio (classique) ne génère aucune sortie. Toutefois, Azure Data Factory requiert un jeu de données de sortie pour que la planification d’un pipeline fonctionne. Par conséquent, nous utilisons dans cet exemple un jeu de données factice/paramètre fictif.  

```JSON
{
    "name": "placeholderBlob",
    "properties": {
        "availability": {
            "frequency": "Week",
            "interval": 1
        },
        "type": "AzureBlob",
        "linkedServiceName": "StorageLinkedService",
        "typeProperties": {
            "folderPath": "any",
            "format": {
                "type": "TextFormat"
            }
        }
    }
}
```

### <a name="pipeline"></a>Pipeline
Le pipeline a deux activités : **AzureMLBatchExecution** et **AzureMLUpdateResource**. L’activité Exécution par lots Azure Machine Learning studio (classique) prend les données d’apprentissage en entrée et produit un fichier .iLearner en sortie. L’activité appelle le service web de formation (expérience de formation exposée comme un service web) avec les données de formation d’entrée et reçoit le fichier iLearner du service web. placeholderBlob est simplement un jeu de données de sortie factice requis par le service Azure Data Factory pour exécuter le pipeline.

![schéma du pipeline](./media/data-factory-azure-ml-batch-execution-activity/update-activity-pipeline-diagram.png)

```JSON
{
    "name": "pipeline",
    "properties": {
        "activities": [
            {
                "name": "retraining",
                "type": "AzureMLBatchExecution",
                "inputs": [
                    {
                        "name": "trainingData"
                    }
                ],
                "outputs": [
                    {
                        "name": "trainedModelBlob"
                    }
                ],
                "typeProperties": {
                    "webServiceInput": "trainingData",
                    "webServiceOutputs": {
                        "output1": "trainedModelBlob"
                    }              
                 },
                "linkedServiceName": "trainingEndpoint",
                "policy": {
                    "concurrency": 1,
                    "executionPriorityOrder": "NewestFirst",
                    "retry": 1,
                    "timeout": "02:00:00"
                }
            },
            {
                "type": "AzureMLUpdateResource",
                "typeProperties": {
                    "trainedModelName": "Training Exp for ADF ML [trained model]",
                    "trainedModelDatasetName" :  "trainedModelBlob"
                },
                "inputs": [
                    {
                        "name": "trainedModelBlob"
                    }
                ],
                "outputs": [
                    {
                        "name": "placeholderBlob"
                    }
                ],
                "policy": {
                    "timeout": "01:00:00",
                    "concurrency": 1,
                    "retry": 3
                },
                "name": "AzureML Update Resource",
                "linkedServiceName": "updatableScoringEndpoint2"
            }
        ],
        "start": "2016-02-13T00:00:00Z",
           "end": "2016-02-14T00:00:00Z"
    }
}
```