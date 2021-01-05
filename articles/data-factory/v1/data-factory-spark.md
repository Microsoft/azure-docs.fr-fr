---
title: Appeler des programmes Spark à partir d’Azure Data Factory
description: Apprenez à appeler des programmes Spark à partir d’une fabrique de données Azure à l’aide de l’activité MapReduce.
services: data-factory
documentationcenter: ''
author: dcstwh
ms.author: weetok
manager: jroth
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/10/2018
ms.openlocfilehash: 417306e09a9424b302bb226aea5dd2c1debe96f5
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97508422"
---
# <a name="invoke-spark-programs-from-azure-data-factory-pipelines"></a>Appeler des programmes Spark à partir des pipelines Azure Data Factory

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
> Cet article s’applique à la version 1 d’Azure Data Factory, qui est mise à la disposition générale. Si vous utilisez la version actuelle du service Data Factory, consultez [Transformer des données à l’aide d’une activité Apache Spark dans Data Factory](../transform-data-using-spark.md).

## <a name="introduction"></a>Introduction
L’activité Spark est l’une des [activités de transformation des données](data-factory-data-transformation-activities.md) prises en charge par Data Factory. Cette activité exécute le programme Spark spécifié sur votre cluster Spark dans Azure HDInsight.

> [!IMPORTANT]
> - L’activité Spark ne prend pas en charge les clusters Spark HDInsight qui utilisent Azure Data Lake Store en tant que stockage principal.
> - L’activité Spark prend en charge uniquement les clusters Spark HDInsight existants (c’est-à-dire vos propres clusters). Elle ne prend pas en charge les services liés HDInsight à la demande.

## <a name="walkthrough-create-a-pipeline-with-a-spark-activity"></a>Procédure pas à pas : Créer un pipeline avec une activité Spark
Voici les étapes classiques pour créer un pipeline de fabrique de données avec une activité Spark :

* Créer une fabrique de données.
* Créez un service lié Stockage Azure pour lier à la fabrique de données le stockage qui est associé à votre cluster Spark HDInsight.
* Créez un service lié HDInsight pour lier à la fabrique de données votre cluster Spark dans HDInsight.
* Créez un jeu de données faisant référence au service lié Stockage. Actuellement, vous devez spécifier un jeu de données de sortie d’une activité même si aucune sortie n’est produite.
* Créez un pipeline avec une activité Spark faisant référence au service lié HDInsight que vous avez créé. L’activité est configurée avec le jeu de données que vous avez créé à l’étape précédente comme un jeu de données de sortie. Le jeu de données de sortie pilote la planification (horaire, quotidienne). Par conséquent, vous devez spécifier le jeu de données de sortie même si l’activité ne produit pas vraiment de sortie.

### <a name="prerequisites"></a>Prérequis
1. Créez un compte de stockage à usage général en suivant les instructions fournies dans [Créer un compte de stockage](../../storage/common/storage-account-create.md).

1. Créez un cluster Spark dans HDInsight en suivant les instructions fournies dans le didacticiel [Créer un cluster Spark dans HDInsight](../../hdinsight/spark/apache-spark-jupyter-spark-sql.md). Associez le compte de stockage que vous avez créé à l’étape 1 à ce cluster.

1. Téléchargez et passez en revue le fichier de script Python **test.py** se trouvant à l’emplacement [https://adftutorialfiles.blob.core.windows.net/sparktutorial/test.py](https://adftutorialfiles.blob.core.windows.net/sparktutorial/test.py).

1. Chargez le fichier **test.py** dans le dossier **pyFiles** du conteneur **adfspark** figurant dans votre stockage Blob. Créez le conteneur et le dossier s’ils n’existent pas.

### <a name="create-a-data-factory"></a>Créer une fabrique de données
Pour créer une fabrique de données, procédez comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. Sélectionnez **Nouveau** > **Données + Analytique** > **Data Factory**.

1. Dans le panneau **Nouvelle fabrique de données**, entrez **SparkDF** dans le champ **Nom**.

   > [!IMPORTANT]
   > Le nom de la fabrique de données Azure doit être un nom global unique. Si l’erreur « Data factory name SparkDF is not available » (Le nom de fabrique de données SparkDF n’est pas disponible) s’affiche, changez le nom de la fabrique de données. Par exemple, utilisez votrenomSparkDF et recréez la fabrique de données. Pour plus d’informations sur les règles de nommage, consultez [Data Factory : règles de nommage](data-factory-naming-rules.md).

1. Sous **Abonnement**, sélectionnez l’abonnement Azure dans lequel vous souhaitez créer la fabrique de données.

1. Sélectionnez un groupe de ressources Azure existant ou créez-en un.

1. Cochez la case **Épingler au tableau de bord**.

1. Sélectionnez **Create** (Créer).

   > [!IMPORTANT]
   > Pour créer des instances Data Factory, vous devez avoir un rôle de [contributeur de fabrique de données](../../role-based-access-control/built-in-roles.md#data-factory-contributor) au niveau de l’abonnement/du groupe de ressources.

1. La fabrique de données apparaît comme en cours de création dans le tableau de bord du portail Azure.

1. Une fois la fabrique de données créée, la page **Fabrique de données** correspondante s’affiche avec son contenu. Si vous ne voyez pas la page **Fabrique de données**, sélectionnez la vignette de votre fabrique de données sur le tableau de bord.

    ![Panneau Data Factory](./media/data-factory-spark/data-factory-blade.png)

### <a name="create-linked-services"></a>Créez des services liés
Dans cette étape, vous créez deux services liés. Un service relie votre cluster Spark à votre fabrique de données, et l’autre service relie votre stockage à votre fabrique de données.

#### <a name="create-a-storage-linked-service"></a>Créer un service lié pour le stockage
Dans cette étape, vous liez votre compte de stockage à votre fabrique de données. Un jeu de données que vous allez créer plus loin dans cette procédure fait référence à ce service lié. Le service lié HDInsight que vous définissez dans l’étape suivante fait également référence à ce service lié.

1. Dans le panneau **Fabrique de données**, sélectionnez **Créer et déployer**. Le Data Factory Editor apparaît.

1. Sélectionnez **Nouveau magasin de données** et choisissez **Stockage Azure**.

   ![Nouveau magasin de données](./media/data-factory-spark/new-data-store-azure-storage-menu.png)

1. Le script JSON que vous utilisez pour créer un service lié Stockage dans l’éditeur apparaît.

   ![AzureStorageLinkedService](./media/data-factory-build-your-first-pipeline-using-editor/azure-storage-linked-service.png)

1. Remplacez **account name** et **account key** par le nom et la clé d’accès de votre compte de stockage. Pour découvrir comment obtenir votre clé d’accès au stockage, consultez [Gérer les clés d’accès au compte de stockage](../../storage/common/storage-account-keys-manage.md).

1. Pour déployer le service lié, sélectionnez **Déployer** dans la barre de commandes. Une fois le service lié déployé, la fenêtre Draft-1 disparaît. **AzureStorageLinkedService** apparaît dans l’arborescence à gauche.

#### <a name="create-an-hdinsight-linked-service"></a>Créer un service lié HDInsight
Dans cette étape, vous créez un service lié HDInsight pour lier à la fabrique de données votre cluster Spark HDInsight. Le service lié HDInsight est utilisé pour exécuter le programme Spark spécifié dans l’activité Spark du pipeline de cet exemple.

1. Dans Data Factory Editor, sélectionnez **Plus** > **Nouveau calcul** > **Cluster HDInsight**.

    ![Créer un service lié Azure HDInsight](media/data-factory-spark/new-hdinsight-linked-service.png)

1. Copiez et collez l’extrait ci-dessous dans la fenêtre Draft-1. Dans l’éditeur JSON, procédez comme suit :

    1. Spécifier l’URI du cluster Spark HDInsight. Par exemple : `https://<sparkclustername>.azurehdinsight.net/`.

    1. Spécifiez le nom de l’utilisateur qui a accès au cluster Spark.

    1. Spécifiez le mot de passe de l’utilisateur.

    1. Spécifiez le service lié Stockage Azure associé au cluster Spark HDInsight. Dans cet exemple, il s’agit de AzureStorageLinkedService.

    ```json
    {
        "name": "HDInsightLinkedService",
        "properties": {
            "type": "HDInsight",
            "typeProperties": {
                "clusterUri": "https://<sparkclustername>.azurehdinsight.net/",
                "userName": "admin",
                "password": "**********",
                "linkedServiceName": "AzureStorageLinkedService"
            }
        }
    }
    ```

    > [!IMPORTANT]
    > - L’activité Spark ne prend pas en charge les clusters Spark HDInsight qui utilisent Azure Data Lake Store en tant que stockage principal.
    > - L’activité Spark prend en charge uniquement les clusters Spark HDInsight existants (c’est-à-dire vos propres clusters). Elle ne prend pas en charge les services liés HDInsight à la demande.

    Pour plus d’informations sur le service lié HDInsight, consultez la page consacrée au [service lié HDInsight](data-factory-compute-linked-services.md#azure-hdinsight-linked-service).

1. Pour déployer le service lié, sélectionnez **Déployer** dans la barre de commandes.

### <a name="create-the-output-dataset"></a>Créer le jeu de données de sortie
Le jeu de données de sortie pilote la planification (horaire, quotidienne). Par conséquent, vous devez spécifier un jeu de données de sortie pour l’activité Spark du pipeline, même si l’activité ne produit pas de sortie. Vous n’êtes pas obligé de spécifier un jeu de données d’entrée pour l’activité.

1. Dans Data Factory Editor, sélectionnez **Plus** > **Nouveau jeu de données** > **Stockage Blob Azure**.

1. Copiez et collez l’extrait ci-dessous dans la fenêtre Draft-1. L’extrait de code JSON définit un jeu de données appelé **OutputDataset**. En outre, vous indiquez que les résultats sont stockés dans le conteneur d’objets blob nommé **adfspark** et dans le dossier nommé **pyFiles/output**. Comme mentionné précédemment, ce jeu de données est un jeu de données factice. Le programme Spark, dans cet exemple, ne produit pas de sortie. La section **availability** spécifie que le jeu de données de sortie est produit tous les jours.

    ```json
    {
        "name": "OutputDataset",
        "properties": {
            "type": "AzureBlob",
            "linkedServiceName": "AzureStorageLinkedService",
            "typeProperties": {
                "fileName": "sparkoutput.txt",
                "folderPath": "adfspark/pyFiles/output",
                "format": {
                    "type": "TextFormat",
                    "columnDelimiter": "\t"
                }
            },
            "availability": {
                "frequency": "Day",
                "interval": 1
            }
        }
    }
    ```
1. Pour déployer le jeu de données, sélectionnez **Déployer** dans la barre de commandes.


### <a name="create-a-pipeline"></a>Créer un pipeline
Dans cette étape, vous créez un pipeline avec une activité HDInsightSpark. À ce stade, c’est le jeu de données de sortie qui pilote la planification : vous devez donc créer un jeu de données de sortie même si l’activité ne produit pas de sortie. Si l’activité ne prend aucune entrée, vous pouvez ignorer la création du jeu de données d’entrée. Par conséquent, aucun jeu de données d’entrée n’est spécifié dans cet exemple.

1. Dans Data Factory Editor, sélectionnez **Plus** > **Nouveau pipeline**.

1. Remplacez le script de la fenêtre Draft-1 par le script suivant :

    ```json
    {
        "name": "SparkPipeline",
        "properties": {
            "activities": [
                {
                    "type": "HDInsightSpark",
                    "typeProperties": {
                        "rootPath": "adfspark\\pyFiles",
                        "entryFilePath": "test.py",
                        "getDebugInfo": "Always"
                    },
                    "outputs": [
                        {
                            "name": "OutputDataset"
                        }
                    ],
                    "name": "MySparkActivity",
                    "linkedServiceName": "HDInsightLinkedService"
                }
            ],
            "start": "2017-02-05T00:00:00Z",
            "end": "2017-02-06T00:00:00Z"
        }
    }
    ```

    Notez les points suivants :

    1. La propriété **type** est définie sur **HDInsightSpark**.

    1. La propriété **rootPath** est définie sur **adfspark\\pyFiles**, où adfspark est le conteneur d’objets Blob et pyFiles est le dossier de fichiers dans ce conteneur. Dans cet exemple, le stockage Blob est celui qui est associé au cluster Spark. Vous pouvez charger le fichier vers un autre compte de stockage. Si vous procédez ainsi, créez un service lié Stockage pour lier à la fabrique de données ce compte de stockage. Ensuite, spécifiez le nom du service lié en tant que valeur pour la propriété **sparkJobLinkedService**. Pour plus d’informations sur cette propriété et d’autres propriétés prises en charge par l’activité Spark, consultez [Propriétés de l’activité Spark](#spark-activity-properties).

    1. La propriété **entryFilePath** est définie sur **test.py**, c’est-à-dire le fichier Python.

    1. La propriété **getDebugInfo** est définie sur **Always**, ce qui signifie que les fichiers journaux sont toujours générés (succès ou échec).

       > [!IMPORTANT]
       > Nous vous recommandons de ne pas définir cette propriété sur `Always` dans un environnement de production, sauf si vous résolvez un problème.

    1. La section **outputs** possède un jeu de données de sortie. Vous devez spécifier un jeu de données de sortie même si le programme Spark ne produit pas de sortie. Le jeu de données de sortie pilote la planification du pipeline (horaire, quotidienne).

    Pour plus d’informations sur les propriétés prises en charge par l’activité Spark, consultez la section [Propriétés de l’activité Spark](#spark-activity-properties).

1. Pour déployer le pipeline, sélectionnez **Déployer** dans la barre de commandes.

### <a name="monitor-a-pipeline"></a>Surveiller un pipeline
1. Dans le panneau **Fabrique de données**, sélectionnez **Surveiller et gérer** pour démarrer l’application de surveillance dans un autre onglet.

    ![Vignette Surveiller et gérer](media/data-factory-spark/monitor-and-manage-tile.png)

1. Modifiez le filtre de **début** en haut de la page sur **2/1/2017**, puis sélectionnez **Appliquer**.

1. Une seule fenêtre d’activité apparaît puisqu’il n’y a qu’un seul jour entre le début (2017-02-01) et la fin (2017-02-02) du pipeline. Vérifiez que la tranche de données est à l’état **Prêt**.

    ![Surveiller le pipeline](media/data-factory-spark/monitor-and-manage-app.png)

1. Dans la liste des **fenêtres d’activité**, sélectionnez une exécution d’activité pour en afficher les détails. S’il y a une erreur, vous pouvez en afficher les détails dans le volet droit.

### <a name="verify-the-results"></a>Vérifier les résultats

1. Ouvrez le bloc-notes Jupyter de votre cluster Spark HDInsight en accédant à `https://CLUSTERNAME.azurehdinsight.net/jupyter`. Vous pouvez également ouvrir un tableau de bord de votre cluster Spark HDInsight, puis ouvrir le bloc-notes Jupyter.

1. Sélectionnez **Nouveau** > **PySpark** pour ouvrir un nouveau bloc-notes.

    ![Nouveau bloc-notes Jupyter](media/data-factory-spark/jupyter-new-book.png)

1. Exécutez la commande suivante en copiant et en collant le texte et en appuyant sur les touches Maj + Entrée à la fin de la deuxième instruction :

    ```sql
    %%sql

    SELECT buildingID, (targettemp - actualtemp) AS temp_diff, date FROM hvac WHERE date = \"6/1/13\"
    ```
1. Vérifiez que les données figurent dans la table hvac.

    ![Résultats de la requête Jupyter](media/data-factory-spark/jupyter-notebook-results.png)

<!-- Removed bookmark #run-a-hive-query-using-spark-sql since it doesn't exist in the target article -->
Pour obtenir des instructions détaillées, consultez la section [Exécuter une requête Spark SQL](../../hdinsight/spark/apache-spark-jupyter-spark-sql.md).

### <a name="troubleshooting"></a>Dépannage
Étant donné que vous définissez getDebugInfo sur **Toujours**, un sous-dossier log apparaît dans le dossier pyFiles de votre conteneur d’objets Blob. Le fichier journal figurant dans ce dossier fournit des informations supplémentaires. Ce fichier journal est particulièrement utile en cas d’erreur. Dans un environnement de production, vous souhaiterez peut-être définir cette propriété sur **Échec**.

Pour résoudre des problèmes, procédez comme suit :


1. Atteindre `https://<CLUSTERNAME>.azurehdinsight.net/yarnui/hn/cluster`.

    ![Application d’interface utilisateur YARN](media/data-factory-spark/yarnui-application.png)

1. Sélectionnez **Journaux d’activité** pour l’une des tentatives d’exécution.

    ![Page d’application](media/data-factory-spark/yarn-applications.png)

1. Vous voyez d’autres informations d’erreur sur la page du journal :

    ![Erreur du journal](media/data-factory-spark/yarnui-application-error.png)

Les sections suivantes fournissent des informations sur les entités de fabrique de données pour utiliser le cluster Spark et l’activité Spark dans votre fabrique de données.

## <a name="spark-activity-properties"></a>Propriétés de l'activité Spark
Voici l’exemple de définition JSON d’un pipeline avec une activité Spark :

```json
{
    "name": "SparkPipeline",
    "properties": {
        "activities": [
            {
                "type": "HDInsightSpark",
                "typeProperties": {
                    "rootPath": "adfspark\\pyFiles",
                    "entryFilePath": "test.py",
                    "arguments": [ "arg1", "arg2" ],
                    "sparkConfig": {
                        "spark.python.worker.memory": "512m"
                    },
                    "getDebugInfo": "Always"
                },
                "outputs": [
                    {
                        "name": "OutputDataset"
                    }
                ],
                "name": "MySparkActivity",
                "description": "This activity invokes the Spark program",
                "linkedServiceName": "HDInsightLinkedService"
            }
        ],
        "start": "2017-02-01T00:00:00Z",
        "end": "2017-02-02T00:00:00Z"
    }
}
```

Le tableau suivant décrit les propriétés JSON utilisées dans la définition JSON.

| Propriété | Description | Obligatoire |
| -------- | ----------- | -------- |
| name | Nom de l'activité dans le pipeline. | Oui |
| description | Texte qui décrit l’activité. | Non  |
| type | Cette propriété doit être définie sur HDInsightSpark. | Oui |
| linkedServiceName | Nom d’un service lié HDInsight sur lequel s’exécute le programme Spark. | Oui |
| rootPath | Conteneur d’objets Blob et dossier contenant le fichier Spark. Le nom de fichier est sensible à la casse. | Oui |
| entryFilePath | Chemin d’accès relatif au dossier racine du code/package Spark. | Oui |
| ClassName | Classe principale Java/Spark de l’application. | Non  |
| arguments | Liste d’arguments de ligne de commande du programme Spark. | Non |
| proxyUser | Compte d’utilisateur à emprunter pour exécuter le programme Spark. | Non  |
| sparkConfig | Spécifiez les valeurs des propriétés de configuration de Spark lisétes dans la rubrique [Configuration Spark : Propriétés de l’application](https://spark.apache.org/docs/latest/configuration.html#available-properties). | Non  |
| getDebugInfo | Spécifie quand les fichiers journaux de Spark sont copiés vers le stockage utilisé par le cluster HDInsight (ou) spécifié par sparkJobLinkedService. Les valeurs autorisées sont Aucun, Toujours ou Échec. La valeur par défaut est Aucun. | Non  |
| sparkJobLinkedService | Service lié Stockage qui contient le fichier de travail, les dépendances et les journaux d’activité Spark. Si vous ne spécifiez pas de valeur pour cette propriété, le stockage associé au cluster HDInsight est utilisé. | Non  |

## <a name="folder-structure"></a>Structure de dossiers
L’activité Spark ne prend pas en charge un script en ligne, contrairement aux activités Pig et Hive. Les travaux Spark sont également plus extensibles que les travaux Pig/Hive. Pour les travaux Spark, vous pouvez fournir plusieurs dépendances, telles que des packages jar (placés dans le CLASSPATH Java), des fichiers Python (placés dans le PYTHONPATH) et tout autre fichier.

Créez la structure de dossiers suivante dans le stockage Blob référencé par le service lié HDInsight. Chargez ensuite les fichiers dépendants dans les sous-dossiers appropriés dans le dossier racine représenté par **entryFilePath**. Par exemple, chargez les fichiers Python dans le sous-dossier pyFiles et les fichiers jar dans le sous-dossier jars du dossier racine. Lors de l’exécution, le service Data Factory attend la structure de dossiers suivante dans le stockage Blob :

| Path | Description | Obligatoire | Type |
| ---- | ----------- | -------- | ---- |
| . | Chemin d’accès racine du travail Spark dans le service lié de stockage. | Oui | Dossier |
| &lt;défini par l’utilisateur &gt; | Chemin d’accès qui pointe vers le fichier d’entrée du travail Spark. | Oui | Fichier |
| ./jars | Tous les fichiers dans ce dossier sont chargés et placés dans le classpath Java du cluster. | Non  | Dossier |
| ./pyFiles | Tous les fichiers dans ce dossier sont chargés et placés dans le PYTHONPATH du cluster. | Non  | Dossier |
| ./files | Tous les fichiers dans ce dossier sont chargés et placés dans le répertoire de travail de l’exécuteur. | Non  | Dossier |
| ./archives | Tous les fichiers dans ce dossier sont décompressés. | Non  | Dossier |
| ./logs | Dossier dans lequel sont stockés les journaux d’activité du cluster Spark| Non  | Dossier |

Voici un exemple de stockage qui contient deux fichiers de travail Spark dans le stockage Blob référencé par le service lié HDInsight :

```
SparkJob1
    main.jar
    files
        input1.txt
        input2.txt
    jars
        package1.jar
        package2.jar
    logs

SparkJob2
    main.py
    pyFiles
        scrip1.py
        script2.py
    logs
```
