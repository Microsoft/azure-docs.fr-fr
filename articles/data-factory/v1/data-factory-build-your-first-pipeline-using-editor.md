---
title: Créer votre première fabrique de données (portail Azure)
description: Dans ce didacticiel, vous allez créer un exemple de pipeline Azure Data Factory à l’aide de Data Factory Editor dans le portail Azure.
services: data-factory
documentationcenter: ''
author: dcstwh
ms.author: weetok
manager: jroth
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: tutorial
ms.date: 01/22/2018
ms.openlocfilehash: 360fbc3e1bfe7890f1f3b05899eb95ce7a3a72c2
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96496617"
---
# <a name="tutorial-build-your-first-data-factory-by-using-the-azure-portal"></a>Tutoriel : Créer votre première fabrique de données à l’aide du portail Azure
> [!div class="op_single_selector"]
> * [Vue d’ensemble et composants requis](data-factory-build-your-first-pipeline.md)
> * [Visual Studio](data-factory-build-your-first-pipeline-using-vs.md)
> * [PowerShell](data-factory-build-your-first-pipeline-using-powershell.md)
> * [Modèle Azure Resource Manager](data-factory-build-your-first-pipeline-using-arm.md)
> * [REST API](data-factory-build-your-first-pipeline-using-rest-api.md)


> [!NOTE]
> Cet article s’applique à la version 1 d’Azure Data Factory, qui est mise à la disposition générale. Si vous utilisez la version actuelle du service Data Factory, consultez [Démarrage rapide : créer une fabrique de données à l’aide de Data Factory](../quickstart-create-data-factory-dot-net.md).

> [!WARNING]
> L’éditeur JSON dans le portail Azure pour la création et le déploiement de pipelines ADF v1 va être désactivé le 31 juillet 2019. Après le 31 juillet 2019, vous pourrez continuer à utiliser les [applets de commande PowerShell ADF v1](/powershell/module/az.datafactory/?view=azps-2.4.0&viewFallbackFrom=azps-2.3.2), le [kit SDK .Net ADF v1](/dotnet/api/microsoft.azure.management.datafactories.models?view=azure-dotnet) et les [API REST ADF v1](/rest/api/datafactory/) pour créer et déployer vos pipelines ADF v1.

Dans cet article, vous allez utiliser le [portail Azure](https://portal.azure.com/) pour créer votre première fabrique de données. Pour suivre le didacticiel avec d’autres outils/kits de développement logiciel (SDK), sélectionnez une des options dans la liste déroulante. 

Le pipeline dans ce didacticiel comprend une activité : une activité Azure HDInsight Hive. Cette activité exécute un script Hive sur un cluster HDInsight qui transforme des données d’entrée pour produire des données de sortie. Le pipeline est programmé pour s’exécuter une fois par mois entre les heures de début et de fin spécifiées. 

> [!NOTE]
> Dans ce didacticiel, le pipeline de données transforme les données d’entrée pour produire des données de sortie. Pour obtenir un tutoriel sur la copie des données à l’aide de Data Factory, consultez [Tutoriel : Copier des données de Stockage Blob Azure vers Azure SQL Database](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).
> 
> Un pipeline peut contenir plusieurs activités. En outre, vous pouvez chaîner deux activités (une après l’autre) en configurant le jeu de données de sortie d’une activité en tant que jeu de données d’entrée de l’autre activité. Pour plus d’informations, consultez [Planification et exécution dans Data Factory](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline).

## <a name="prerequisites"></a>Prérequis
Consultez la [Vue d’ensemble du didacticiel](data-factory-build-your-first-pipeline.md) et suivez les étapes de la section « Configuration requise ».

Cet article ne fournit pas de vue d’ensemble conceptuelle du service Data Factory. Pour plus d’informations sur le service, consultez [Présentation d’Azure Data Factory](data-factory-introduction.md).  

## <a name="create-a-data-factory"></a>Créer une fabrique de données
Une fabrique de données peut avoir un ou plusieurs pipelines. Un pipeline peut contenir une ou plusieurs activités. Il peut s’agir d’une activité de copie pour copier les données d’une source vers un magasin de données de destination. Il peut également s’agir d’une activité HDInsight Hive pour exécuter un script Hive afin de transformer des données d’entrée en données de sortie. 

Pour créer une fabrique de données, procédez comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. Sélectionnez **Nouveau** > **Données + Analytique** > **Data Factory**.

   ![Panneau Créer](./media/data-factory-build-your-first-pipeline-using-editor/create-blade.png)

1. Dans le panneau **Nouvelle fabrique de données**, entrez **GetStartedDF** dans le champ **Nom**.

   ![Panneau Nouvelle fabrique de données](./media/data-factory-build-your-first-pipeline-using-editor/new-data-factory-blade.png)

   > [!IMPORTANT]
   > Le nom de la fabrique de données doit être un nom global unique. Si l’erreur "Data factory name GetStartedDF is not available" (Le nom de fabrique de données GetStartedDF n’est pas disponible) s’affiche, changez le nom de la fabrique de données. Par exemple, utilisez votrenomGetStartedDF et recréez la fabrique de données. Pour plus d’informations sur les règles de nommage, consultez [Data Factory : règles de nommage](data-factory-naming-rules.md).
   >
   > Le nom de la fabrique de données pourra être enregistré en tant que nom DNS et devenir ainsi visible publiquement.
   >
   >
1. Sous **Abonnement**, sélectionnez l’abonnement Azure dans lequel vous souhaitez créer la fabrique de données.

1. Sélectionnez un groupe de ressources existant ou créez-en un. Pour les besoins de ce didacticiel, créez un groupe de ressources nommé **ADFGetStartedRG**.

1. Sous **Emplacement**, sélectionnez l’emplacement de la fabrique de données. Seules les régions prises en charge par le service Data Factory sont affichées dans la liste déroulante.

1. Cochez la case **Épingler au tableau de bord**.

1. Sélectionnez **Create** (Créer).

   > [!IMPORTANT]
   > Pour créer des instances Data Factory, vous devez avoir un rôle de [contributeur de fabrique de données](../../role-based-access-control/built-in-roles.md#data-factory-contributor) au niveau de l’abonnement/du groupe de ressources.
   >
   >
1. Sur le tableau de bord, vous voyez la vignette suivante avec l’état **Déploiement de Data Factory** :    

   ![État Déploiement de Data Factory](./media/data-factory-build-your-first-pipeline-using-editor/creating-data-factory-image.png)

1. Une fois la fabrique de données créée, la page **Fabrique de données** correspondante s’affiche avec son contenu.     

    ![Panneau Data Factory](./media/data-factory-build-your-first-pipeline-using-editor/data-factory-blade.png)

Avant de créer un pipeline dans la fabrique de données, vous devez créer quelques entités Data Factory. Créez d’abord des services liés pour lier des magasins de données/services de calcul à votre magasin de données. Définissez ensuite des jeux de données d’entrée et de sortie pour représenter les données d’entrée/sortie dans les magasins de données liés. Enfin, créez le pipeline avec une activité qui utilise ces jeux de données.

## <a name="create-linked-services"></a>Créez des services liés
Dans cette étape, vous liez votre compte Stockage Azure et un cluster HDInsight à la demande à votre fabrique de données. Le compte de stockage contient les données d’entrée et de sortie pour le pipeline de cet exemple. Le service lié HDInsight est utilisé pour exécuter le script Hive spécifié dans l’activité du pipeline de cet exemple. Identifiez les [magasins de données](data-factory-data-movement-activities.md)/[services de calcul](data-factory-compute-linked-services.md) qui sont utilisés dans votre scénario. Liez ensuite ces services à la fabrique de données en créant des services liés.  

### <a name="create-a-storage-linked-service"></a>Créer un service lié pour le stockage
Dans cette étape, vous liez votre compte de stockage à votre fabrique de données. Dans ce didacticiel, vous utilisez le même compte de stockage pour stocker les données d’entrée/sortie et le fichier de script HQL.

1. Dans le panneau **Fabrique de données**, sélectionnez **Créer et déployer** pour **GetStartedDF**. Data Factory Editor s’affiche.

   ![Vignette Créer et déployer](./media/data-factory-build-your-first-pipeline-using-editor/data-factory-author-deploy.png)

1. Sélectionnez **Nouveau magasin de données** et choisissez **Stockage Azure**.

   ![Panneau Nouveau magasin de données](./media/data-factory-build-your-first-pipeline-using-editor/new-data-store-azure-storage-menu.png)

1. Le script JSON de création d’un service lié Stockage apparaît dans l’éditeur.

   ![Service lié Stockage](./media/data-factory-build-your-first-pipeline-using-editor/azure-storage-linked-service.png)

1. Remplacez **nom de compte** par le nom de votre compte de stockage. Remplacez **clé de compte** par la clé d’accès du compte de stockage. Pour découvrir comment obtenir votre clé d’accès au stockage, consultez [Gérer les clés d’accès au compte de stockage](../../storage/common/storage-account-keys-manage.md).

1. Sélectionnez **Déployer** dans la barre de commandes pour déployer le service lié.

    ![Bouton déployer](./media/data-factory-build-your-first-pipeline-using-editor/deploy-button.png)

   Une fois le service lié déployé, la fenêtre Draft-1 disparaît. **AzureStorageLinkedService** apparaît dans l’arborescence à gauche.

    ![AzureStorageLinkedService](./media/data-factory-build-your-first-pipeline-using-editor/StorageLinkedServiceInTree.png)    

### <a name="create-an-hdinsight-linked-service"></a>Créer un service lié HDInsight
Dans cette étape, vous liez un cluster HDInsight à la demande à votre fabrique de données. Le cluster HDInsight est créé automatiquement au moment de l’exécution. Le cluster est supprimé une fois le traitement terminé et au terme du délai d’inactivité spécifié.

1. Dans Data Factory Editor, sélectionnez **Plus** > **Nouveau calcul** > **Cluster HDInsight à la demande**.

    ![Nouveau calcul](./media/data-factory-build-your-first-pipeline-using-editor/new-compute-menu.png)

1. Copiez et collez l’extrait ci-dessous dans la fenêtre Draft-1. L’extrait de code JSON décrit les propriétés permettant de créer le cluster HDInsight à la demande.

    ```JSON
    {
        "name": "HDInsightOnDemandLinkedService",
        "properties": {
            "type": "HDInsightOnDemand",
            "typeProperties": {
                "version": "3.5",
                "clusterSize": 1,
                "timeToLive": "00:05:00",
                "osType": "Linux",
                "linkedServiceName": "AzureStorageLinkedService"
            }
        }
    }
    ```

    Le tableau suivant décrit les propriétés JSON utilisées dans l’extrait de code.

   | Propriété | Description |
   |:--- |:--- |
   | clusterSize |Spécifie la taille du cluster HDInsight. |
   | timeToLive | Spécifie la durée d’inactivité du cluster HDInsight avant sa suppression. |
   | linkedServiceName | Spécifie le compte de stockage utilisé pour stocker les journaux d’activité générés par HDInsight. |

    Notez les points suivants :

     a. La fabrique de données crée pour vous un cluster HDInsight Linux avec les propriétés JSON. Pour plus d’informations, voir [Service lié HDInsight à la demande](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service).

     b. Vous pouvez utiliser votre propre cluster HDInsight au lieu d’utiliser un cluster HDInsight à la demande. Pour plus d’informations, voir [Service lié HDInsight](data-factory-compute-linked-services.md#azure-hdinsight-linked-service).

     c. Le cluster HDInsight crée un conteneur par défaut dans le stockage d’objets blob que vous avez spécifié dans la propriété JSON (**linkedServiceName**). HDInsight ne supprime pas ce conteneur lorsque le cluster est supprimé. Ce comportement est normal. Avec le service lié HDInsight à la demande, un cluster HDInsight est créé dès qu’une tranche est traitée, à moins qu’il n’existe un cluster actif (**timeToLive**). Ce cluster est automatiquement supprimé une fois le traitement terminé.

     À mesure que le nombre de tranches traitées augmente, les conteneurs se multiplient dans votre stockage d’objets blob. Si vous n’en avez pas besoin pour dépanner les travaux, vous pouvez les supprimer afin de réduire les frais de stockage. Le nom de ces conteneurs suit un modèle : « adf **nomdevotrefabriquededonnées**-**nomduservicelié**-horodatage ». Utilisez des outils tels que [l’Explorateur Stockage Microsoft Azure](https://storageexplorer.com/) pour supprimer des conteneurs dans votre stockage d’objets blob.

     Pour plus d’informations, voir [Service lié HDInsight à la demande](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service).

1. Sélectionnez **Déployer** dans la barre de commandes pour déployer le service lié.

    ![Option Déployer](./media/data-factory-build-your-first-pipeline-using-editor/ondemand-hdinsight-deploy.png)

1. Confirmez que vous voyez s’afficher à la fois **AzureStorageLinkedService** et **HDInsightOnDemandLinkedService** dans l’arborescence à gauche de l’écran.

    ![Capture d’écran montrant qu’AzureStorageLinkedService et HDInsightOnDemandLinkedService sont liés entre eux.](./media/data-factory-build-your-first-pipeline-using-editor/tree-view-linked-services.png)

## <a name="create-datasets"></a>Créez les jeux de données
Dans cette étape, vous créez des jeux de données afin de représenter les données d’entrée et de sortie pour le traitement Hive. Ces jeux de données font référence au service AzureStorageLinkedService que vous avez créé précédemment dans ce didacticiel. Le service lié mène à un compte de stockage. Les jeux de données spécifient le conteneur, le dossier et le nom du fichier dans le stockage qui contient les données d’entrée et de sortie.   

### <a name="create-the-input-dataset"></a>Créer le jeu de données d’entrée
1. Dans Data Factory Editor, sélectionnez **Plus** > **Nouveau jeu de données** > **Stockage Blob Azure**.

    ![Nouveau jeu de données](./media/data-factory-build-your-first-pipeline-using-editor/new-data-set.png)

1. Copiez et collez l’extrait ci-dessous dans la fenêtre Draft-1. Dans l’extrait de code JSON, vous créez un jeu de données appelé **AzureBlobInput** qui représente les données d’entrée pour une activité dans le pipeline. En outre, vous spécifiez que les données d’entrée se trouvent dans le conteneur d’objets blob nommé **adfgetstarted** et dans le dossier nommé **inputdata**.

    ```JSON
    {
        "name": "AzureBlobInput",
        "properties": {
            "type": "AzureBlob",
            "linkedServiceName": "AzureStorageLinkedService",
            "typeProperties": {
                "fileName": "input.log",
                "folderPath": "adfgetstarted/inputdata",
                "format": {
                    "type": "TextFormat",
                    "columnDelimiter": ","
                }
            },
            "availability": {
                "frequency": "Month",
                "interval": 1
            },
            "external": true,
            "policy": {}
        }
    }
    ```
    Le tableau suivant décrit les propriétés JSON utilisées dans l’extrait de code.

   | Propriété | Imbriqués sous | Description |
   |:--- |:--- |:--- |
   | type | properties |La propriété type est définie sur **AzureBlob**, car les données se trouvent dans le stockage d’objets blob. |
   | linkedServiceName | format |Fait référence au service AzureStorageLinkedService que vous avez créé précédemment. |
   | folderPath | typeProperties | Spécifie le conteneur d’objets blob et le dossier contenant les objets blob d’entrée. | 
   | fileName | typeProperties |Cette propriété est facultative. Si vous omettez cette propriété, tous les fichiers spécifiés dans le paramètre folderPath sont récupérés. Dans ce didacticiel, seul le fichier input.log est traité. |
   | type | format |Les fichiers journaux étant au format texte, sélectionnez **TextFormat**. |
   | columnDelimiter | format |Les colonnes des fichiers journaux sont délimitées par une virgule (`,`). |
   | frequency/interval | availability |La fréquence est définie sur **Mois** et l’intervalle est **1**, ce qui signifie que les tranches d’entrée sont disponibles mensuellement. |
   | external | properties | Cette propriété a la valeur **true** si les données d’entrée ne sont pas générées par ce pipeline. Dans ce didacticiel, le fichier input.log n’est pas généré par ce pipeline. La propriété est donc définie sur **true**. |

    Pour plus d’informations sur ces propriétés JSON, voir [Connecteur de stockage Blob Azure](data-factory-azure-blob-connector.md#dataset-properties).

1. Sélectionnez **Déployer** dans la barre de commandes pour déployer le jeu de données que vous venez de créer. Le jeu de données apparaît dans l’arborescence sur la gauche.

### <a name="create-the-output-dataset"></a>Créer le jeu de données de sortie
Vous allez maintenant créer le jeu de données de sortie pour représenter les données de sortie stockées dans le stockage d’objets blob.

1. Dans Data Factory Editor, sélectionnez **Plus** > **Nouveau jeu de données** > **Stockage Blob Azure**.

1. Copiez et collez l’extrait ci-dessous dans la fenêtre Draft-1. Dans l’extrait de code JSON, vous créez un jeu de données appelé **AzureBlobOutput** et spécifiez la structure des données produites par le script Hive. Vous indiquez aussi que les résultats sont stockés dans le conteneur d’objets blob appelé **adfgetstarted** et dans le dossier appelé **partitioneddata**. La section **availability** indique que le jeu de données de sortie est produit tous les mois.

    ```JSON
    {
      "name": "AzureBlobOutput",
      "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
          "folderPath": "adfgetstarted/partitioneddata",
          "format": {
            "type": "TextFormat",
            "columnDelimiter": ","
          }
        },
        "availability": {
          "frequency": "Month",
          "interval": 1
        }
      }
    }
    ```
    Consultez la section Créer le jeu de données d’entrée pour obtenir une description de ces propriétés. Vous ne définissez pas la propriété externe sur un jeu de données de sortie, car le jeu de données est produit par le service Data Factory.

1. Sélectionnez **Déployer** dans la barre de commandes pour déployer le jeu de données que vous venez de créer.

1. Vérifiez que le jeu de données a été correctement créé.

    ![Arborescence avec les services liés](./media/data-factory-build-your-first-pipeline-using-editor/tree-view-data-set.png)

## <a name="create-a-pipeline"></a>Créer un pipeline
Dans cette étape, vous créez votre premier pipeline avec une activité HDInsight Hive. La tranche d’entrée est disponible mensuellement (fréquence : Mois, intervalle : 1). La tranche de sortie est produite mensuellement. La propriété du planificateur pour l’activité est également définie sur Mensuellement. Les paramètres pour le jeu de données de sortie et le planificateur d’activité doivent correspondre. À ce stade, c’est le jeu de données de sortie qui pilote la planification : vous devez donc créer un jeu de données de sortie même si l’activité ne produit pas de sortie. Si l’activité ne prend aucune entrée, vous pouvez ignorer la création du jeu de données d’entrée. Les propriétés utilisées dans l’extrait de code JSON suivant sont expliquées à la fin de cette section.

1. Dans Data Factory Editor, sélectionnez **Plus** > **Nouveau pipeline**.

    ![Option Nouveau pipeline](./media/data-factory-build-your-first-pipeline-using-editor/new-pipeline-button.png)

1. Copiez et collez l’extrait ci-dessous dans la fenêtre Draft-1.

   > [!IMPORTANT]
   > Dans l’extrait de code JSON, remplacez **storageaccountname** par le nom de votre compte de stockage.
   >
   >

    ```JSON
    {
        "name": "MyFirstPipeline",
        "properties": {
            "description": "My first Azure Data Factory pipeline",
            "activities": [
                {
                    "type": "HDInsightHive",
                    "typeProperties": {
                        "scriptPath": "adfgetstarted/script/partitionweblogs.hql",
                        "scriptLinkedService": "AzureStorageLinkedService",
                        "defines": {
                            "inputtable": "wasb://adfgetstarted@<storageaccountname>.blob.core.windows.net/inputdata",
                            "partitionedtable": "wasb://adfgetstarted@<storageaccountname>.blob.core.windows.net/partitioneddata"
                        }
                    },
                    "inputs": [
                        {
                            "name": "AzureBlobInput"
                        }
                    ],
                    "outputs": [
                        {
                            "name": "AzureBlobOutput"
                        }
                    ],
                    "policy": {
                        "concurrency": 1,
                        "retry": 3
                    },
                    "scheduler": {
                        "frequency": "Month",
                        "interval": 1
                    },
                    "name": "RunSampleHiveActivity",
                    "linkedServiceName": "HDInsightOnDemandLinkedService"
                }
            ],
            "start": "2017-07-01T00:00:00Z",
            "end": "2017-07-02T00:00:00Z",
            "isPaused": false
        }
    }
    ```

    Dans l’extrait de code JSON, vous créez un pipeline qui se compose d’une seule activité utilisant Hive pour traiter des données sur un cluster HDInsight.

    Le fichier script Hive (**partitionweblogs.hql**) est stocké dans le compte de stockage, qui est spécifié par le service scriptLinkedService, appelé **AzureStorageLinkedService1**. Vous pouvez le trouver dans le dossier **script** du conteneur **adfgetstarted**.

    La section **defines** est utilisée pour spécifier les paramètres d’exécution transmis au script Hive comme valeurs de configuration Hive. Exemples : ${hiveconf:inputtable} et ${hiveconf:partitionedtable}.

    Les propriétés **start** et **end** du pipeline spécifient la période active du pipeline.

    Dans l’activité JSON, vous spécifiez que le script Hive s’exécute sur le calcul spécifié par le service **linkedServiceName** : **HDInsightOnDemandLinkedService**.

   > [!NOTE]
   > Pour en savoir plus sur les propriétés JSON utilisées dans l’exemple, consultez la section « Pipeline JSON » de l’article [Pipelines et activités dans Azure Data Factory](data-factory-create-pipelines.md).
   >
   >
1. Vérifiez les éléments suivants :

   a. Le fichier **input.log** existe dans le dossier **inputdata** du conteneur **adfgetstarted** dans le stockage d’objets blob.

   b. Le fichier **partitionweblogs.hql** existe dans le dossier **script** du conteneur **adfgetstarted** dans le stockage d’objets blob. Si vous ne voyez pas ces fichiers, suivez les étapes de la section « Configuration requise » de la [Vue d’ensemble du didacticiel](data-factory-build-your-first-pipeline.md).

   c. Dans le pipeline JSON, vérifiez que vous avez bien remplacé **storageaccountname** par le nom de votre compte de stockage.

1. Sélectionnez **Déployer** dans la barre de commandes pour déployer le pipeline. Étant donné que les valeurs **start** et **end** spécifiées sont antérieures au moment actuel, et que **isPaused** est défini sur **false**, le pipeline (l’activité dans le pipeline) s’exécute immédiatement après son déploiement.

1. Vérifiez que le pipeline apparaît dans l’arborescence.

    ![Arborescence avec pipeline](./media/data-factory-build-your-first-pipeline-using-editor/tree-view-pipeline.png)



## <a name="monitor-a-pipeline"></a>Surveiller un pipeline
### <a name="monitor-a-pipeline-by-using-the-diagram-view"></a>Surveiller un pipeline à l’aide de la vue Diagramme
1. Dans le panneau **Data Factory**, sélectionnez **Diagramme**.

    ![Vignette de diagramme](./media/data-factory-build-your-first-pipeline-using-editor/diagram-tile.png)

1. La vue **Diagramme** contient une vue d’ensemble des pipelines et des jeux de données utilisés dans ce didacticiel.

    ![Vue schématique](./media/data-factory-build-your-first-pipeline-using-editor/diagram-view-2.png)

1. Pour afficher toutes les activités du pipeline, cliquez avec le bouton droit sur le pipeline dans le diagramme, puis sélectionnez **Ouvrir un pipeline**.

    ![Menu Ouvrir un pipeline](./media/data-factory-build-your-first-pipeline-using-editor/open-pipeline-menu.png)

1. Vérifiez que **l’activité Hive** apparaît dans le pipeline.

    ![Vue Ouvrir un pipeline](./media/data-factory-build-your-first-pipeline-using-editor/open-pipeline-view.png)

    Pour revenir à la vue précédente, sélectionnez **Fabrique de données** dans le menu supérieur.

1. Dans la vue **Diagramme**, double-cliquez sur le jeu de données **AzureBlobInput**. Vérifiez que l’état de la tranche est **Prêt**. Plusieurs minutes peuvent être nécessaires avant que la tranche n’apparaisse avec l’état **Prêt**. Si rien ne se produit au bout d’un moment, vérifiez que le fichier d’entrée (**input.log**) est placé dans le conteneur (**adfgetstarted**) et le dossier (**inputdata**) appropriés.

   ![Tranche d’entrée dans l’état Prêt](./media/data-factory-build-your-first-pipeline-using-editor/input-slice-ready.png)

1. Fermez le panneau **AzureBlobInput**.

1. Dans la vue **Diagramme**, double-cliquez sur le jeu de données **AzureBlobOutput**. Vous pouvez voir que la tranche est en cours de traitement.

   ![Jeu de données en cours de traitement](./media/data-factory-build-your-first-pipeline-using-editor/dataset-blade.png)

1. Une fois le traitement terminé, la tranche apparaît à l’état **Prêt**.

   ![Jeu de données dans l’état Prêt](./media/data-factory-build-your-first-pipeline-using-editor/dataset-slice-ready.png)  

   > [!IMPORTANT]
   > La création d’un cluster HDInsight à la demande prend généralement une vingtaine de minutes. Le pipeline devrait donc traiter la tranche en 30 minutes environ.
   >
   >

1. Quand l’état de la tranche est **Prêt**, vérifiez la présence des données de sortie dans le dossier **partitioneddata** du conteneur **adfgetstarted** de votre stockage d’objets blob.  

   ![Données de sortie](./media/data-factory-build-your-first-pipeline-using-editor/three-ouptut-files.png)

1. Sélectionnez la tranche pour en afficher les détails dans le panneau **Tranche de données**.

    ![Détails de la tranche de données](./media/data-factory-build-your-first-pipeline-using-editor/data-slice-details.png)

1. Dans la liste **Exécutions d’activités**, sélectionnez une exécution d’activité pour en afficher les détails. (Dans ce scénario, il s’agit d’une activité Hive.) Les informations s’affichent dans un panneau **Détails de l’exécution d’activité**.   

    ![Fenêtre Détails de l’exécution d’activité](./media/data-factory-build-your-first-pipeline-using-editor/activity-window-blade.png)    

   Dans les fichiers journaux, vous pouvez voir la requête Hive qui a été exécutée et son état. Ces journaux d’activité sont utiles pour résoudre les problèmes.
   Pour en savoir plus, consultez l’article [Surveiller et gérer les pipelines à l’aide des panneaux du portail Azure](data-factory-monitor-manage-pipelines.md).

> [!IMPORTANT]
> Le fichier d’entrée sera supprimé lorsque la tranche est traitée avec succès. Par conséquent, si vous souhaitez réexécuter la tranche ou refaire le didacticiel, chargez le fichier d’entrée (**input.log**) dans le dossier **inputdata** du conteneur **adfgetstarted**.
>
>

### <a name="monitor-a-pipeline-by-using-the-monitor--manage-app"></a>Surveiller un pipeline à l’aide de l’application Surveiller et gérer
Vous pouvez également utiliser l’application Surveiller et gérer pour surveiller vos pipelines. Pour en savoir plus sur l’utilisation de cette application, consultez l’article [Surveiller et gérer les pipelines Azure Data Factory à l’aide de l’application de surveillance et gestion](data-factory-monitor-manage-app.md).

1. Sélectionnez la vignette **Surveiller et gérer** sur la page d’accueil de votre fabrique de données.

    ![Vignette Surveiller et gérer](./media/data-factory-build-your-first-pipeline-using-editor/monitor-and-manage-tile.png)

1. Dans l’application Surveiller et gérer, modifiez les valeurs **Heure de début** et **Heure de fin** pour qu’elles correspondent aux heures de début et de fin de votre pipeline. Sélectionnez **Appliquer**.

    ![Application Surveiller et gérer](./media/data-factory-build-your-first-pipeline-using-editor/monitor-and-manage-app.png)

1. Sélectionnez une fenêtre d’activité dans la liste des **fenêtres d’activité** pour en afficher les détails.

    ![Liste des fenêtres d’activité](./media/data-factory-build-your-first-pipeline-using-editor/activity-window-details.png)

## <a name="summary"></a>Résumé
Dans ce didacticiel, vous avez créé une fabrique de données pour traiter des données en exécutant un script Hive sur un cluster Hadoop HDInsight. Vous avez effectué les étapes suivantes dans le portail Azure à l’aide de Data Factory Editor :  

* Créer une fabrique de données.
* Création de deux services liés :
   * Un service lié Stockage pour lier à la fabrique de données votre stockage d’objets blob contenant les fichiers d’entrée/sortie.
   * Un service lié HDInsight à la demande pour lier à la fabrique de données un cluster Hadoop HDInsight à la demande. Data Factory crée un cluster Hadoop HDInsight juste-à-temps pour traiter les données d’entrée et produire des données de sortie.
* Création de deux jeux de données qui décrivent les données d’entrée et de sortie pour l’activité HDInsight Hive dans le pipeline.
* Création d’un pipeline avec une activité HDInsight Hive.

## <a name="next-steps"></a>Étapes suivantes
Dans cet article, vous avez créé un pipeline avec une activité de transformation (Activité HDInsight) qui exécute un script Hive sur un cluster HDInsight à la demande. Pour voir comment utiliser une activité de copie pour copier des données du Stockage Blob vers Azure SQL Database, consultez [Tutoriel : Copier des données de Stockage Blob vers SQL Database](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).

## <a name="see-also"></a>Voir aussi
| Rubrique | Description |
|:--- |:--- |
| [Pipelines](data-factory-create-pipelines.md) |Cet article vous aide à comprendre les pipelines et les activités dans Data Factory, et à les utiliser dans l’optique de créer des workflows pilotés par les données de bout en bout pour votre scénario ou votre entreprise. |
| [Groupes de données](data-factory-create-datasets.md) |Cet article vous aide à comprendre les jeux de données dans Data Factory. |
| [Planification et exécution](data-factory-scheduling-and-execution.md) |Cet article explique les aspects de la planification et de l’exécution du modèle d’application Data Factory. |
| [Surveiller et gérer les pipelines à l’aide de l’application Surveiller et gérer](data-factory-monitor-manage-app.md) |Cet article décrit comment surveiller, gérer et déboguer les pipelines à l’aide de l’application Surveiller et gérer. |