---
title: Créer des pipelines de données prédictifs
description: Découvrez la procédure de création d’un pipeline prédictif à l’aide d’Azure Machine Learning Studio (classique) - Activité d’exécution par lot dans Azure Data Factory.
author: nabhishek
ms.author: abnarain
manager: shwang
services: data-factory
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 07/16/2020
ms.openlocfilehash: 50ef97bca0a5359c49ba2f18b1ec789ab076350a
ms.sourcegitcommit: fb3c846de147cc2e3515cd8219d8c84790e3a442
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92637732"
---
# <a name="create-a-predictive-pipeline-using-azure-machine-learning-studio-classic-and-azure-data-factory"></a>Créer un pipeline prédictif à l’aide d’Azure Machine Learning Studio (classique) et Azure Data Factory

> [!div class="op_single_selector" title1="Sélectionnez la version du service Data Factory que vous utilisez :"]
> * [Version 1](v1/data-factory-azure-ml-batch-execution-activity.md)
> * [Version actuelle](transform-data-using-machine-learning.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

[Azure Machine Learning Studio (classique)](https://azure.microsoft.com/documentation/services/machine-learning/) vous permet de générer, tester et déployer des solutions d’analyse prédictive. D’un point de vue très général, cela s’effectue en trois étapes :

1. **Créez une expérience de formation** . Vous effectuez cette étape à l’aide d’Azure Machine Learning Studio (classique). Azure Machine Learning Studio (classique) est un environnement de développement visuel collaboratif qui vous permet d’entraîner et de tester un modèle d’analyse prédictive à l’aide de données d’entraînement.
2. **Convertissez-la en une expérience prédictive** . Une fois que votre modèle a été formé avec des données existantes et que vous êtes prêt à l’utiliser pour la notation de nouvelles données, vous préparez et simplifiez votre expérience de notation.
3. **Déployez-la en tant que service web** . Vous pouvez publier votre expérience de notation comme un service web Azure. Vous pouvez envoyer des données à votre modèle via ce point de terminaison de service web et recevoir des prédictions de résultats du modèle.

### <a name="data-factory-and-azure-machine-learning-studio-classic-together"></a>Data Factory et Azure Machine Learning Studio (classique) ensemble
Azure Data Factory vous permet de créer facilement des pipelines qui utilisent un service web [Azure Machine Learning Studio (classique)](https://azure.microsoft.com/documentation/services/machine-learning) publié pour l’analyse prédictive. À l’aide de l’ **activité d’exécution du lot** dans un pipeline Azure Data Factory, vous pouvez appeler un service web Azure Machine Learning Studio (classique) pour effectuer des prédictions sur les données par lots.

Au fil du temps, les modèles prédictifs dans les expériences de scoring Azure Machine Learning Studio (classique) doivent être réentraînés à l’aide de nouveaux jeux de données d’entrée. Vous pouvez réentraîner un modèle à partir d’un pipeline Data Factory en effectuant les étapes suivantes :

1. Publiez l’expérience de formation (et non l’expérience prédictive) comme un service web. Vous effectuez cette étape dans Azure Machine Learning Studio (classique) comme vous l’avez fait pour exposer l’expérience prédictive en tant que service web dans le scénario précédent.
2. Utilisez l’activité d’exécution du lot Azure Machine Learning Studio (classique) pour appeler le service web pour l’expérience d’entraînement. En fait, vous pouvez utiliser l’activité d’exécution du lot Azure Machine Learning Studio (classique) pour appeler à la fois le service web d’entraînement et le service web de scoring.

Une fois que vous avez fini le réentraînement, mettez à jour le service web de scoring (expérience prédictive exposée comme service web) avec le modèle qui vient d’être entraîné à l’aide de l’ **activité des ressources de mise à jour Azure Machine Learning Studio (classique)** . Consultez l’article [Mise à jour des modèles à l’aide de l’activité des ressources de mise à jour](update-machine-learning-models.md) pour plus d’informations.

## <a name="azure-machine-learning-studio-classic-linked-service"></a>Service lié Azure Machine Learning Studio (classique)

Vous créez un service lié **Azure Machine Learning Studio (classique)** pour lier un service web Azure Machine Learning Studio (classique) à une fabrique de données Azure. Le service lié est utilisé par l'Activité d'exécution par lot et l'[Activité des ressources de mise à jour](update-machine-learning-models.md) Azure Machine Learning Studio (classique).

```JSON
{
    "type" : "linkedServices",
    "name": "AzureMLLinkedService",
    "properties": {
        "type": "AzureML",
        "typeProperties": {
            "mlEndpoint": "URL to Azure ML Predictive Web Service",
            "apiKey": {
                "type": "SecureString",
                "value": "api key"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

Consultez l’article sur les [services liés de calcul](compute-linked-services.md) pour obtenir les descriptions des propriétés dans la définition JSON.

Azure Machine Learning Studio (classique) prend à la fois en charge les services web classiques et les nouveaux services web pour votre expérience prédictive. Vous pouvez choisir celui qui vous convient à partir de Data Factory. Pour obtenir les informations requises afin de créer le service lié Azure Machine Learning Studio (classique), accédez à https://services.azureml.net, où tous vos (nouveaux) services web et les services web classiques sont répertoriés. Cliquez sur le service web auquel vous voulez accéder, puis cliquez sur la page **Consommer** . Copiez la **clé primaire** de la propriété **apiKey** , puis les **requêtes de lots** de la propriété **mlEndpoint** .

![Services web Azure Machine Learning Studio (classique)](./media/transform-data-using-machine-learning/web-services.png)

## <a name="azure-machine-learning-studio-classic-batch-execution-activity"></a>Activité d’exécution par lot Azure Machine Learning Studio (classique)

L’extrait de code JSON suivant définit une activité d’exécution par lot Azure Machine Learning Studio (classique). La définition de l’activité comporte une référence au service lié Azure Machine Learning Studio (classique) que vous avez créé précédemment.

```JSON
{
    "name": "AzureMLExecutionActivityTemplate",
    "description": "description",
    "type": "AzureMLBatchExecution",
    "linkedServiceName": {
        "referenceName": "AzureMLLinkedService",
        "type": "LinkedServiceReference"
    },
    "typeProperties": {
        "webServiceInputs": {
            "<web service input name 1>": {
                "LinkedServiceName":{
                    "referenceName": "AzureStorageLinkedService1",
                    "type": "LinkedServiceReference"
                },
                "FilePath":"path1"
            },
            "<web service input name 2>": {
                "LinkedServiceName":{
                    "referenceName": "AzureStorageLinkedService1",
                    "type": "LinkedServiceReference"
                },
                "FilePath":"path2"
            }
        },
        "webServiceOutputs": {
            "<web service output name 1>": {
                "LinkedServiceName":{
                    "referenceName": "AzureStorageLinkedService2",
                    "type": "LinkedServiceReference"
                },
                "FilePath":"path3"
            },
            "<web service output name 2>": {
                "LinkedServiceName":{
                    "referenceName": "AzureStorageLinkedService2",
                    "type": "LinkedServiceReference"
                },
                "FilePath":"path4"
            }
        },
        "globalParameters": {
            "<Parameter 1 Name>": "<parameter value>",
            "<parameter 2 name>": "<parameter 2 value>"
        }
    }
}
```

| Propriété          | Description                              | Obligatoire |
| :---------------- | :--------------------------------------- | :------- |
| name              | Nom de l’activité dans le pipeline     | Oui      |
| description       | Texte décrivant l’activité.  | Non       |
| type              | Pour l'activité U-SQL de Data Lake Analytics, le type d'activité est **AzureMLBatchExecution** . | Oui      |
| linkedServiceName | Services liés au service lié Azure Machine Learning Studio (classique) Pour en savoir plus sur ce service lié, consultez l’article [Services liés de calcul](compute-linked-services.md). | Oui      |
| webServiceInputs  | Paires clé/valeur, caractérisant les noms des entrées du service web Azure Machine Learning Studio (classique). La clé doit correspondre aux paramètres d’entrée définis dans le service web Azure Machine Learning Studio (classique) publié. La valeur est une paire de propriétés FilePath de services liés de stockage Azure spécifiant les emplacements d’objets blob d’entrée. | Non       |
| webServiceOutputs | Paires clé/valeur caractérisant les noms des sorties du service web Azure Machine Learning Studio (classique). La clé doit correspondre aux paramètres de sortie définis dans le service web Azure Machine Learning Studio (classique) publié. La valeur est une paire de propriétés FilePath de services liés de stockage Azure spécifiant les emplacements d’objets blob de sortie. | Non       |
| globalParameters  | Paires clé/valeur à passer au point de terminaison du service d’exécution de lots Azure Machine Learning Studio (classique). Les clés doivent correspondre aux noms des paramètres de service web définis dans le service web Azure Machine Learning Studio (classique) publié. Les valeurs sont passées dans la propriété GlobalParameters de la demande d’exécution par lot Azure Machine Learning Studio (classique). | Non       |

### <a name="scenario-1-experiments-using-web-service-inputsoutputs-that-refer-to-data-in-azure-blob-storage"></a>Scénario 1 : Expériences qui utilisent des entrées/sorties de service web qui font référence à des données dans Stockage Blob Azure

Dans ce scénario, le service web Azure Machine Learning Studio (classique) effectue des prédictions à l’aide des données d’un fichier dans un stockage d’objets blob Azure et stocke les résultats des prédictions dans le stockage d’objets blob. Le code JSON suivant définit un pipeline Data Factory avec une activité AzureMLBatchExecution. Les données d’entrée et de sortie dans le stockage d’objets blob Azure sont référencées à l’aide d’une paire LinkedName/FilePath. Dans l’exemple, le service lié des entrées et des sorties est différent. Vous pouvez utiliser des services liés différents pour chacune de vos entrées/sorties pour que Data Factory puisse récupérer les bons fichiers et les envoyer au service web Azure Machine Learning Studio (classique).

> [!IMPORTANT]
> Dans votre expérience Azure Machine Learning Studio (classique), les ports et paramètres globaux de l’entrée et la sortie du service web ont des noms par défaut (« input1 », « input2 ») que vous pouvez personnaliser. Les noms que vous utilisez pour les paramètres globalParameters, webServiceOutputs et webServiceInputs doivent correspondre exactement aux noms utilisés dans les expériences. Vous pouvez afficher la charge utile de l’exemple de requête dans la page d’aide relative à l’exécution par lots pour votre point de terminaison Azure Machine Learning Studio (classique) afin de vérifier le mappage attendu.


```JSON
{
    "name": "AzureMLExecutionActivityTemplate",
    "description": "description",
    "type": "AzureMLBatchExecution",
    "linkedServiceName": {
        "referenceName": "AzureMLLinkedService",
        "type": "LinkedServiceReference"
    },
    "typeProperties": {
        "webServiceInputs": {
            "input1": {
                "LinkedServiceName":{
                    "referenceName": "AzureStorageLinkedService1",
                    "type": "LinkedServiceReference"
                },
                "FilePath":"amltest/input/in1.csv"
            },
            "input2": {
                "LinkedServiceName":{
                    "referenceName": "AzureStorageLinkedService1",
                    "type": "LinkedServiceReference"
                },
                "FilePath":"amltest/input/in2.csv"
            }
        },
        "webServiceOutputs": {
            "outputName1": {
                "LinkedServiceName":{
                    "referenceName": "AzureStorageLinkedService2",
                    "type": "LinkedServiceReference"
                },
                "FilePath":"amltest2/output/out1.csv"
            },
            "outputName2": {
                "LinkedServiceName":{
                    "referenceName": "AzureStorageLinkedService2",
                    "type": "LinkedServiceReference"
                },
                "FilePath":"amltest2/output/out2.csv"
            }
        }
    }
}
```
### <a name="scenario-2-experiments-using-readerwriter-modules-to-refer-to-data-in-various-storages"></a>Scénario 2 : Expériences qui utilisent des modules lecteur/enregistreur pour faire référence à des données dans différents stockages
Un autre scénario courant lors de la création d’expériences Azure Machine Learning Studio (classique) consiste à utiliser des modules de données d’importation et de données de sortie. Le module de données d’importation permet de charger des données dans une expérience, tandis que le module de données de sortie sert à enregistrer les données issues de cette expérience. Pour plus d’informations sur les modules de données d’importation et de données de sortie, consultez les rubriques [Données d’importation](/azure/machine-learning/studio-module-reference/import-data) et [Données de sortie](/azure/machine-learning/studio-module-reference/export-data) sur MSDN Library.

Quand vous utilisez les modules de données d’importation et de données de sortie, il est conseillé d’utiliser un paramètre de service web pour chaque propriété de ces modules. Ces paramètres web permettent de configurer les valeurs pendant l’exécution. Par exemple, vous pouvez créer une expérience avec un module d’importation de données qui utilise une base de données Azure SQL Database : XXX.database.windows.net. Une fois le service web déployé, vous pouvez autoriser les consommateurs du service web à spécifier un autre serveur SQL logique appelé `YYY.database.windows.net`. Vous pouvez utiliser un paramètre de service web pour permettre à cette valeur d’être configurée.

> [!NOTE]
> L’entrée et la sortie du service web diffèrent des paramètres de service web. Dans le premier scénario, vous avez vu comment une entrée et une sortie peuvent être spécifiées pour un service web Azure Machine Learning Studio (classique). Dans ce scénario, vous passez pour un service web des paramètres qui correspondent aux propriétés des modules de données d’importation/de sortie.

Examinons un scénario d’utilisation de paramètres de service web. Vous avez déployé un service web Azure Machine Learning Studio (classique) qui utilise un module lecteur pour lire les données de l’une des sources de données prises en charge par Azure Machine Learning Studio (classique) (par exemple : Azure SQL Database). Après l’exécution par lots, les résultats sont écrits à l’aide d’un module enregistreur (Azure SQL Database).  Aucune entrée ou sortie de service web n’est définie dans les expériences. Dans ce cas, nous vous recommandons de configurer les paramètres de service web appropriés pour les modules lecteur et enregistreur. Cela permet la configuration des modules lecteur/enregistreur quand vous utilisez l’activité AzureMLBatchExecution. Spécifiez les paramètres de service web dans la section **globalParameters** du code JSON de l’activité comme suit.

```JSON
"typeProperties": {
    "globalParameters": {
        "Database server name": "<myserver>.database.windows.net",
        "Database name": "<database>",
        "Server user account name": "<user name>",
        "Server user account password": "<password>"
    }
}
```

> [!NOTE]
> Les paramètres de service web respectent la casse ; veillez donc à ce que les noms que vous indiquez dans le script JSON de l'activité correspondent à ceux exposés par le service web.

Une fois que vous avez fini le réentraînement, mettez à jour le service web de scoring (expérience prédictive exposée comme service web) avec le modèle qui vient d’être entraîné à l’aide de l’ **activité des ressources de mise à jour Azure Machine Learning Studio (classique)** . Consultez l’article [Mise à jour des modèles à l’aide de l’activité des ressources de mise à jour](update-machine-learning-models.md) pour plus d’informations.

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