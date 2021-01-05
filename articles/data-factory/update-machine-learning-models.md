---
title: Mettre à jour des modèles Azure Machine Learning Studio (classique) à l’aide d’Azure Data Factory
description: Décrit comment créer des pipelines prédictifs à l’aide d’Azure Data Factory et d’Azure Machine Learning Studio (classique)
services: data-factory
documentationcenter: ''
author: dcstwh
ms.author: weetok
manager: jroth
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 07/16/2020
ms.openlocfilehash: bec300414483181617a7aa009157a4c4a332c745
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96496736"
---
# <a name="update-azure-machine-learning-studio-classic-models-by-using-update-resource-activity"></a>Mettre à jour des modèles Azure Machine Learning Studio (classique) à l’aide de l’activité des ressources de mise à jour

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Cet article vient compléter l’article principal sur l’intégration Azure Data Factory - Azure Machine Learning Studio (classique) : [Créez un pipeline prédictif à l’aide d’Azure Machine Learning Studio (classique) et Azure Data Factory](transform-data-using-machine-learning.md). Si vous ne l’avez pas encore fait, consultez l’article principal avant de lire cet article.

## <a name="overview"></a>Vue d’ensemble
Dans le cadre du processus de mise en place de modèles Azure Machine Learning Studio (classique), votre modèle est formé et enregistré. Vous l’utilisez ensuite pour créer un service web prédictif. Le service web peut ensuite être utilisé dans des sites web, des tableaux de bord et des applications mobiles.

Les modèles que vous créez à l’aide d’Azure Machine Learning Studio (classique) ne sont généralement pas statiques. Lorsque de nouvelles données sont disponibles ou lorsque le consommateur de l’API a ses propres données, il faut effectuer à nouveau l’apprentissage du modèle. 

Il peut être très fréquent d’avoir à effectuer à nouveau l’apprentissage du modèle. Avec l’activité d’exécution par lot et l’activité des ressources de mise à jour, vous pouvez mettre en place le modèle Azure Machine Learning Studio (classique) qui reforme et met à jour le service web prédictif à l’aide de Data Factory.

L’image suivante illustre la relation entre les services web d’apprentissage et de prévision.

![SERVICES WEB](./media/update-machine-learning-models/web-services.png)

## <a name="azure-machine-learning-studio-classic-update-resource-activity"></a>Activité de ressource de mise à jour Azure Machine Learning Studio (classique)

L’extrait de code JSON suivant définit une activité d’exécution par lot Azure Machine Learning Studio (classique).

```json
{
    "name": "amlUpdateResource",
    "type": "AzureMLUpdateResource",
    "description": "description",
    "linkedServiceName": {
        "type": "LinkedServiceReference",
        "referenceName": "updatableScoringEndpoint2"
    },
    "typeProperties": {
        "trainedModelName": "ModelName",
        "trainedModelLinkedServiceName": {
                    "type": "LinkedServiceReference",
                    "referenceName": "StorageLinkedService"
                },
        "trainedModelFilePath": "ilearner file path"
    }
}
```

| Propriété                      | Description                              | Obligatoire |
| :---------------------------- | :--------------------------------------- | :------- |
| name                          | Nom de l’activité dans le pipeline     | Oui      |
| description                   | Texte décrivant l’activité.  | Non       |
| type                          | Pour l’activité des ressources de mise à jour Azure Machine Learning Studio (classique), le type d’activité est **AzureMLUpdateResource**. | Oui      |
| linkedServiceName             | Service lié Azure Machine Learning Studio (classique) qui contient la propriété updateResourceEndpoint. | Oui      |
| trainedModelName              | Nom du module de modèle formé dans l’expérience du service web à mettre à jour. | Oui      |
| trainedModelLinkedServiceName | Nom du service lié de stockage Azure contenant le fichier ilearner chargé par l’opération de mise à jour. | Oui      |
| trainedModelFilePath          | Chemin relatif du fichier dans trainedModelLinkedService pour représenter le fichier ilearner chargé par l’opération de mise à jour. | Oui      |

## <a name="end-to-end-workflow"></a>Workflow de bout en bout

L’ensemble du processus de mise en place de la reformation d’un modèle et de la mise à jour des services web prédictifs implique les étapes suivantes :

- Appelez le **service web de formation** à l’aide de l’**activité d’exécution par lot**. Un appel de service web de formation est identique à un appel de service web prédictif décrit dans [Créer des pipelines prédictifs avec Azure Machine Learning Studio (classique) et l’activité d’exécution par lot Data Factory](transform-data-using-machine-learning.md). La sortie du service web de formation est un fichier iLearner que vous pouvez utiliser pour mettre à jour le service web prédictif.
- Appelez le **point de terminaison des ressources de mise à jour** du **service web prédictif** à l’aide de l’**activité des ressources de mise à jour** pour mettre à jour le service web avec le modèle qui vient d’être formé.

## <a name="azure-machine-learning-studio-classic-linked-service"></a>Service lié Azure Machine Learning Studio (classique)

Pour que le flux de travail de bout en bout mentionné ci-dessus fonctionne, vous devez créer deux services liés Azure Machine Learning Studio (classique) :

1. Un service lié Azure Machine Learning Studio (classique) pour le service web d’apprentissage. Ce service lié est utilisé par l’activité d’exécution par lot de la même façon que ce qui est mentionné dans [Créer des pipelines prédictifs avec Azure Machine Learning Studio (classique) et l’activité d’exécution par lot Data Factory](transform-data-using-machine-learning.md). La différence est que la sortie du service web d’apprentissage est un fichier iLearner qui est ensuite utilisé par l’activité des ressources de mise à jour pour mettre à jour le service web prédictif.
2. Un service lié Azure Machine Learning Studio (classique) pour le point de terminaison des ressources de mise à jour du service web prédictif. Ce service lié est utilisé par l’activité des ressources de mise à jour pour mettre à jour le service web prédictif à l’aide du fichier iLearner retourné par l’étape ci-dessus.

Pour le second service lié Azure Machine Learning Studio (classique), la configuration est différente quand votre service web Azure Machine Learning Studio (classique) est un service web classique ou un nouveau service web. Les différences sont expliquées distinctement dans les sections suivantes.

## <a name="web-service-is-new-azure-resource-manager-web-service"></a>Le service web est un nouveau service web Azure Resource Manager

Si le service web est un nouveau type de service web qui expose un point de terminaison Azure Resource Manager, vous n’avez pas besoin ajouter le second point de terminaison, **qui n’est pas celui par défaut** . Le **updateResourceEndpoint** du service lié est au format :

```
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resource-group-name}/providers/Microsoft.MachineLearning/webServices/{web-service-name}?api-version=2016-05-01-preview
```

Vous pouvez obtenir des valeurs pour les espaces réservés dans l’URL lors de l’interrogation du service web sur le [portail des services web Azure Machine Learning Studio (classique)](https://services.azureml.net/).

Le nouveau type de point de terminaison des ressources de mise à jour exige une authentification du principal du service. Pour utiliser une authentification du principal du service, inscrivez une entité d’application dans Azure Active Directory (Azure AD) et octroyez-lui le rôle de **contributeur** ou de **propriétaire** de l’abonnement ou du groupe de ressources auquel le service web appartient. Consultez [how to create service principal and assign permissions to manage Azure resource](../active-directory/develop/howto-create-service-principal-portal.md) (comment créer le principal de service et affecter des autorisations de gestion de ressources Azure). Prenez note des valeurs suivantes, qui vous permettent de définir le service lié :

- ID de l'application
- Clé de l'application
- ID client

Voici un exemple de définition de service lié :

```json
{
    "name": "AzureMLLinkedService",
    "properties": {
        "type": "AzureML",
        "description": "The linked service for AML web service.",
        "typeProperties": {
            "mlEndpoint": "https://ussouthcentral.services.azureml.net/workspaces/0000000000000000  000000000000000000000/services/0000000000000000000000000000000000000/jobs?api-version=2.0",
            "apiKey": {
                "type": "SecureString",
                "value": "APIKeyOfEndpoint1"
            },
            "updateResourceEndpoint": "https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resource-group-name}/providers/Microsoft.MachineLearning/webServices/{web-service-name}?api-version=2016-05-01-preview",
            "servicePrincipalId": "000000000-0000-0000-0000-0000000000000",
            "servicePrincipalKey": {
                "type": "SecureString",
                "value": "servicePrincipalKey"
            },
            "tenant": "mycompany.com"
        }
    }
}
```

Le scénario suivant fournit plus de détails. Il présente un exemple de réentraînement et de mise à jour de modèles Machine Learning Studio (classique) à partir d’un pipeline Azure Data Factory.


## <a name="sample-retraining-and-updating-an-azure-machine-learning-studio-classic-model"></a>Exemple : Réentraînement et mise à jour d’un modèle Azure Machine Learning Studio (classique)

Cette section fournit un exemple de pipeline qui utilise **l’activité d’exécution par lot Azure Machine Learning Studio (classique)** pour réentraîner un modèle. Le pipeline utilise également l’**activité des ressources de mise à jour Azure Machine Learning Studio (classique)** pour mettre à jour le modèle dans le service web de notation. La section fournit également des extraits de code JSON pour tous les services liés, jeux de données et éléments de pipeline dans l’exemple.

### <a name="azure-blob-storage-linked-service"></a>Service lié Azure Blob Storage :
Azure Storage contient les données suivantes :

* Données de formation. Les données d’entrée pour le service web d’entraînement Azure Machine Learning Studio (classique).
* Fichier iLearner. La sortie du service web d’entraînement Azure Machine Learning Studio (classique). Ce fichier est également l’entrée de l’activité des ressources de mise à jour.

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

### <a name="linked-service-for-azure-machine-learning-studio-classic-training-endpoint"></a>Service lié pour le point de terminaison d’entraînement Azure Machine Learning Studio (classique)
L’extrait de code JSON suivant définit un service lié Azure Machine Learning Studio (classique) qui pointe vers le point de terminaison par défaut du service web de formation.

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

### <a name="linked-service-for-azure-machine-learning-studio-classic-updatable-scoring-endpoint"></a>Service lié pour le point de terminaison de notation pouvant être mis à jour Azure Machine Learning Studio (classique) :
L’extrait de code JSON suivant définit un service lié Azure Machine Learning Studio (classique) qui pointe vers le point de terminaison pouvant être mis à jour du service web de notation.

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

### <a name="pipeline"></a>Pipeline
Le pipeline a deux activités : **AzureMLBatchExecution** et **AzureMLUpdateResource**. L’activité d’exécution par lot prend les données d’apprentissage comme entrée et génère un fichier .iLearner comme sortie. L’activité des ressources de mise à jour prend alors ce fichier iLearner et l’utilise pour mettre à jour le service web prédictif.

```JSON
{
    "name": "LookupPipelineDemo",
    "properties": {
        "activities": [
            {
                "name": "amlBEGetilearner",
                "description": "Use AML BES to get the ileaner file from training web service",
                "type": "AzureMLBatchExecution",
                "linkedServiceName": {
                    "referenceName": "trainingEndpoint",
                    "type": "LinkedServiceReference"
                },
                "typeProperties": {
                    "webServiceInputs": {
                        "input1": {
                            "LinkedServiceName":{
                                "referenceName": "StorageLinkedService",
                                "type": "LinkedServiceReference"
                            },
                            "FilePath":"azuremltesting/input"
                        },
                        "input2": {
                            "LinkedServiceName":{
                                "referenceName": "StorageLinkedService",
                                "type": "LinkedServiceReference"
                            },
                            "FilePath":"azuremltesting/input"
                        }
                    },
                    "webServiceOutputs": {
                        "output1": {
                            "LinkedServiceName":{
                                "referenceName": "StorageLinkedService",
                                "type": "LinkedServiceReference"
                            },
                            "FilePath":"azuremltesting/output"
                        }
                    }
                }
            },
            {
                "name": "amlUpdateResource",
                "type": "AzureMLUpdateResource",
                "description": "Use AML Update Resource to update the predict web service",
                "linkedServiceName": {
                    "type": "LinkedServiceReference",
                    "referenceName": "updatableScoringEndpoint2"
                },
                "typeProperties": {
                    "trainedModelName": "ADFV2Sample Model [trained model]",
                    "trainedModelLinkedServiceName": {
                        "type": "LinkedServiceReference",
                        "referenceName": "StorageLinkedService"
                    },
                    "trainedModelFilePath": "azuremltesting/output/newModelForArm.ilearner"
                },
                "dependsOn": [
                    {
                        "activity": "amlbeGetilearner",
                        "dependencyConditions": [ "Succeeded" ]
                    }
                ]
            }
        ]
    }
}
```
## <a name="next-steps"></a>Étapes suivantes
Consultez les articles suivants qui expliquent comment transformer des données par d’autres moyens :

* [Activité U-SQL](transform-data-using-data-lake-analytics.md)
* [Activité Hive](transform-data-using-hadoop-hive.md)
* [Activité pig](transform-data-using-hadoop-pig.md)
* [Activité MapReduce](transform-data-using-hadoop-map-reduce.md)
* [Activité de diffusion en continu Hadoop](transform-data-using-hadoop-streaming.md)
* [Activité Spark](transform-data-using-spark.md)
* [Activité personnalisée .NET](transform-data-using-dotnet-custom-activity.md)
* [Activité de procédure stockée](transform-data-using-stored-procedure.md)
