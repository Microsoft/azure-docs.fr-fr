---
title: Créer votre première fabrique de données Azure (Visual Studio)
description: Dans ce didacticiel, vous allez créer un exemple de pipeline Azure Data Factory avec Visual Studio.
services: data-factory
documentationcenter: ''
author: dcstwh
ms.author: weetok
manager: jroth
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: tutorial
ms.custom: vs-azure
ms.date: 01/22/2018
ms.openlocfilehash: b0532e75ac581bad3b5dd8669a591186dc320a86
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96496566"
---
# <a name="tutorial-create-a-data-factory-by-using-visual-studio"></a>Didacticiel : Créer une fabrique de données à l’aide de Visual Studio
> [!div class="op_single_selector" title="Tools/SDKs"]
> * [Vue d’ensemble et composants requis](data-factory-build-your-first-pipeline.md)
> * [Visual Studio](data-factory-build-your-first-pipeline-using-vs.md)
> * [PowerShell](data-factory-build-your-first-pipeline-using-powershell.md)
> * [Modèle Resource Manager](data-factory-build-your-first-pipeline-using-arm.md)
> * [REST API](data-factory-build-your-first-pipeline-using-rest-api.md)


> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez [Démarrage rapide : Créer une fabrique de données à l’aide d’Azure Data Factory](../quickstart-create-data-factory-dot-net.md).

Ce tutoriel vous explique comment créer une fabrique de données Azure Data Factory à l’aide de Visual Studio. Vous allez créer un projet Visual Studio à l’aide du modèle de projet Data Factory, puis définir des entités Data Factory (services liés, jeux de données et pipeline) au format JSON et vous allez terminer en publiant/déployant ces entités dans le cloud. 

Le pipeline dans ce didacticiel a une activité : **Activité HDInsight Hive**. Cette activité exécute un script Hive sur un cluster HDInsight qui transforme des données d’entrée pour produire des données de sortie. Le pipeline est programmé pour s’exécuter une fois par mois entre les heures de début et de fin spécifiées. 

> [!NOTE]
> Ce didacticiel n’explique pas comment copier des données à l’aide d’Azure Data Factory. Pour un didacticiel sur la copie de données à l’aide d’Azure Data Factory, consultez [Copie de données Blob Storage vers une base de données SQL à l’aide de Data Factory](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).
> 
> Un pipeline peut contenir plusieurs activités. En outre, vous pouvez chaîner deux activités (une après l’autre) en configurant le jeu de données de sortie d’une activité en tant que jeu de données d’entrée de l’autre activité. Pour plus d’informations, consultez [Planification et exécution dans Data Factory](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline).


## <a name="walkthrough-create-and-publish-data-factory-entities"></a>Procédure pas à pas : Création et publication d’entités Data Factory
Voici les étapes à effectuer dans le cadre de cette procédure pas à pas :

1. Créez deux services liés : **AzureStorageLinkedService1** et **HDInsightOnDemandLinkedService1**. 
   
    Dans ce didacticiel, les données d’entrée et de sortie de l’activité Hive se trouvent dans le même Stockage Blob Azure. Vous utilisez un cluster HDInsight à la demande pour traiter les données d’entrée existantes afin de produire les données de sortie. Le cluster HDInsight à la demande est automatiquement créé pour vous par Azure Data Factory au moment de l’exécution, lorsque les données d’entrée sont prêtes à être traitées. Vous devez lier vos magasins de données ou vos services de calcul à votre fabrique de données, de façon que le service Data Factory puisse s’y connecter au moment de l’exécution. Par conséquent, vous liez votre compte de stockage Azure à la fabrique de données à l’aide d’AzureStorageLinkedService1 et vous liez un cluster HDInsight à la demande à l’aide de HDInsightOnDemandLinkedService1. Lors de la publication, vous spécifiez le nom de la fabrique de données à créer ou une fabrique de données existante.  
2. Créez deux jeux de données : **InputDataset** et **OutputDataset**, qui représentent les données d’entrée/de sortie stockées dans le Stockage Blob Azure. 
   
    Ces définitions de jeu de données font référence au service lié Azure Storage que vous avez créé à l’étape précédente. Pour InputDataset, vous spécifiez le conteneur de blobs (adfgetstarted) et le dossier (inputdata) qui contient un blob avec les données d’entrée. Pour OutputDataset, vous spécifiez le conteneur de blobs (adfgetstarted) et le dossier (partitioneddata) qui contient les données de sortie. Vous spécifiez également d’autres propriétés telles que la structure, la disponibilité et la stratégie.
3. Créez un pipeline nommé **MyFirstPipeline**. 
  
    Dans cette procédure pas à pas, le pipeline a une seule activité : **Activité HDInsight Hive**. Cette activité transforme des données d’entrée pour produire des données de sortie en exécutant un script Hive sur un cluster HDInsight à la demande. Pour en savoir plus sur l’activité Hive, consultez [Transformer des données à l’aide d’une activité Hive dans Azure Data Factory](data-factory-hive-activity.md). 
4. Créez une fabrique de données nommée **DataFactoryUsingVS**. Déployez la fabrique de données et toutes les entités Data Factory (services liés, tables et pipeline).
5. Après la publication, utilisez les panneaux du portail Azure et l’application de surveillance et gestion pour surveiller le pipeline. 
  
### <a name="prerequisites"></a>Conditions préalables requises

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

1. Lisez l’article [Vue d’ensemble du didacticiel](data-factory-build-your-first-pipeline.md) et effectuez les **étapes préalables requises** . Vous pouvez également sélectionner l’option **Vue d’ensemble et étapes préalables requises** de la liste déroulante figurant en haut de la page pour basculer vers l’article correspondant. Une fois les étapes préalables suivies, revenez à cet article en sélectionnant l’option **Visual Studio** dans la liste déroulante.
2. Pour créer des instances Data Factory, vous devez avoir un rôle de [collaborateur de fabrique de données](../../role-based-access-control/built-in-roles.md#data-factory-contributor) au niveau de l’abonnement/du groupe de ressources.  
3. Les composants suivants doivent être installés sur votre ordinateur :
   * Visual Studio 2013 ou Visual Studio 2015
   * Téléchargez le Kit de développement logiciel (SDK) Azure pour Visual Studio 2013 ou Visual Studio 2015. Accédez à la [page de téléchargement d’Azure](https://azure.microsoft.com/downloads/), puis cliquez sur **VS 2013** ou **VS 2015** dans la section  **.NET**.
   * Téléchargez le dernier plug-in Azure Data Factory pour Visual Studio : [VS 2013](https://visualstudiogallery.msdn.microsoft.com/754d998c-8f92-4aa7-835b-e89c8c954aa5) ou [VS 2015](https://visualstudiogallery.msdn.microsoft.com/371a4cf9-0093-40fa-b7dd-be3c74f49005). Vous pouvez également mettre à jour le plug-in en procédant comme suit : dans le menu, cliquez sur **Outils** -> **Extensions et mises à jour** -> **En ligne** -> **Galerie Visual Studio** -> **Outils Microsoft Azure Data Factory pour Visual Studio** -> **Mettre à jour**.

À présent, utilisons Visual Studio pour créer une fabrique de données Azure Data Factory.

### <a name="create-visual-studio-project"></a>Création d’un projet Visual Studio
1. Lancez **Visual Studio 2013** ou **Visual Studio 2015**. Cliquez sur **Fichier**, pointez le curseur de la souris sur **Nouveau**, puis cliquez sur **Projet**. La boîte de dialogue **Nouveau projet** doit s’afficher.  
2. Dans la boîte de dialogue **Nouveau projet**, sélectionnez le modèle **DataFactory**, puis cliquez sur **Projet Data Factory vide**.   

    ![Boîte de dialogue Nouveau projet](./media/data-factory-build-your-first-pipeline-using-vs/new-project-dialog.png)
3. Entrez le **nom** du projet, son **emplacement** et le nom de la **solution**, puis cliquez sur **OK**.

    ![Explorateur de solutions](./media/data-factory-build-your-first-pipeline-using-vs/solution-explorer.png)

### <a name="create-linked-services"></a>Créez des services liés
Au cours de cette étape, vous allez créer deux services liés : **Stockage Azure** et **HDInsight à la demande**. 

Le service lié Stockage Azure lie votre compte de stockage Azure à la fabrique de données en fournissant les informations de connexion. Le service Data Factory utilise la chaîne de connexion à partir du paramètre de service lié pour se connecter au stockage Azure lors de l’exécution. Ce stockage contient les données d’entrée et de sortie pour le pipeline et le fichier de script Hive utilisé par l’activité Hive. 

Avec le service lié HDInsight à la demande, le cluster HDInsight à la demande est automatiquement créé au moment de l’exécution, lorsque les données d’entrée sont prêtes à être traitées. Le cluster est supprimé une fois le traitement terminé et au terme du délai d’inactivité spécifié. 

> [!NOTE]
> Vous créez une fabrique de données en spécifiant son nom et ses paramètres au moment de la publication de votre solution Data Factory.

#### <a name="create-azure-storage-linked-service"></a>Créer le service lié Stockage Azure
1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur **Services liés**, pointez sur **Ajouter**, puis cliquez sur **Nouvel élément**.      
2. Dans la boîte de dialogue **Ajouter un nouvel élément**, sélectionnez **Service lié Azure Storage** dans la liste, puis cliquez sur **Ajouter**.
    ![Capture d’écran mettant en évidence le service lié Stockage Azure dans la liste.](./media/data-factory-build-your-first-pipeline-using-vs/new-azure-storage-linked-service.png)
3. Remplacez `<accountname>` et `<accountkey>` par le nom de votre compte de stockage Azure et par sa clé. Pour découvrir comment obtenir votre clé d’accès au stockage, consultez [Gérer les clés d’accès au compte de stockage](../../storage/common/storage-account-keys-manage.md).
    ![Service lié Azure Storage](./media/data-factory-build-your-first-pipeline-using-vs/azure-storage-linked-service.png)
4. Enregistrez le fichier **AzureStorageLinkedService1.json** .

#### <a name="create-azure-hdinsight-linked-service"></a>Créer le service lié Azure HDInsight
1. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur **Services liés**, pointez sur **Ajouter** puis cliquez sur **Nouvel élément**.
2. Sélectionnez **Service lié à la demande HDInsight** puis cliquez sur **Ajouter**.
3. Remplacez le code **JSON** par le code JSON suivant :

     ```json
    {
        "name": "HDInsightOnDemandLinkedService",
        "properties": {
        "type": "HDInsightOnDemand",
            "typeProperties": {
                "version": "3.5",
                "clusterSize": 1,
                "timeToLive": "00:05:00",
                "osType": "Linux",
                "linkedServiceName": "AzureStorageLinkedService1"
            }
        }
    }
    ```

    Le tableau suivant décrit les propriétés JSON utilisées dans l'extrait de code :

    Propriété | Description
    -------- | ----------- 
    clusterSize | Spécifie la taille du cluster HDInsight Hadoop.
    timeToLive | Spécifie la durée d’inactivité du cluster HDInsight avant sa suppression.
    linkedServiceName | Spécifie le compte de stockage utilisé pour stocker les journaux d’activité générés par le cluster HDInsight Hadoop. 

    > [!IMPORTANT]
    > Le cluster HDInsight crée un **conteneur par défaut** dans le Stockage Blob que vous avez spécifié dans le code JSON (linkedServiceName). HDInsight ne supprime pas ce conteneur lorsque le cluster est supprimé. Ce comportement est normal. Avec le service lié HDInsight à la demande, un cluster HDInsight est créé dès qu’une tranche est traitée, à moins qu’il n’existe un cluster actif (timeToLive). Ce cluster est supprimé, une fois le traitement terminé.
    > 
    > Comme un nombre croissant de tranches sont traitées, vous voyez un grand nombre de conteneurs dans votre stockage d’objets blob Azure. Si vous n’en avez pas besoin pour dépanner les travaux, il se peut que vous deviez les supprimer pour réduire les frais de stockage. Les noms de ces conteneurs sont conformes au modèle suivant : `adf<yourdatafactoryname>-<linkedservicename>-datetimestamp`. Utilisez des outils tels que l’[Explorateur Stockage Microsoft Azure](https://storageexplorer.com/) pour supprimer des conteneurs dans votre stockage Blob Azure.

    Pour plus d’informations sur les propriétés JSON, consultez [Service lié à la demande Azure HDInsight](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service). 
4. Enregistrez le fichier **HDInsightOnDemandLinkedService1.json** .

### <a name="create-datasets"></a>Créez les jeux de données
Dans cette étape, vous créez des jeux de données afin de représenter les données d’entrée et de sortie pour le traitement Hive. Ces jeux de données font référence au service **AzureStorageLinkedService1** que vous avez créé précédemment dans ce didacticiel. Le service lié pointe vers un compte de stockage Azure, et les jeux de données spécifient le conteneur, le dossier et le nom de fichier dans le stockage qui contient les données d’entrée et de sortie.   

#### <a name="create-input-dataset"></a>Créer le jeu de données d’entrée
1. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur **Tables**, pointez sur **Ajouter**, puis cliquez sur **Nouvel élément**.
2. Sélectionnez **Objet blob Azure** dans la liste, changez le nom du fichier en **InputDataSet.json**, puis cliquez sur **Ajouter**.
3. Remplacez le code **JSON** dans l’éditeur par l’extrait de code JSON suivant :

    ```json
    {
        "name": "AzureBlobInput",
        "properties": {
            "type": "AzureBlob",
            "linkedServiceName": "AzureStorageLinkedService1",
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
    Cet extrait de code JSON définit un jeu de données appelé **AzureBlobInput** qui représente les données d’entrée de l’activité Hive dans le pipeline. Vous indiquez que les données d’entrée se trouvent dans le conteneur de blobs nommé `adfgetstarted` et dans le dossier nommé `inputdata`.

    Le tableau suivant décrit les propriétés JSON utilisées dans l'extrait de code :

    Propriété | Description |
    -------- | ----------- |
    type |La propriété de type est définie sur **AzureBlob**, car les données se trouvent dans le Stockage Blob Azure.
    linkedServiceName | Fait référence au service AzureStorageLinkedService1 que vous avez créé précédemment.
    fileName |Cette propriété est facultative. Si vous omettez cette propriété, tous les fichiers spécifiés dans le paramètre folderPath sont récupérés. Dans le cas présent, seul le fichier input.log est traité.
    type | Les fichiers journaux sont au format texte : nous utilisons donc TextFormat. |
    columnDelimiter | Les colonnes des fichiers journaux sont délimitées par une virgule (`,`)
    frequency/interval | La fréquence est définie sur Mois et l’intervalle est 1, ce qui signifie que les segments d’entrée sont disponibles mensuellement.
    external | Cette propriété a la valeur true si les données d’entrée de l’activité ne sont pas générées par le pipeline. Cette propriété est uniquement spécifiée sur les jeux de données d’entrée. Pour le jeu de données d’entrée de la première activité, choisissez toujours la valeur true.
4. Enregistrez le fichier **InputDataset.json** .

#### <a name="create-output-dataset"></a>Créer un jeu de données de sortie
Vous allez maintenant créer le jeu de données de sortie pour représenter les données de sortie stockées dans le Stockage Blob Azure.

1. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur **Tables**, pointez sur **Ajouter**, puis cliquez sur **Nouvel élément**.
2. Sélectionnez **Objet blob Azure** dans la liste, changez le nom du fichier en **OutputDataset.json**, puis cliquez sur **Ajouter**.
3. Remplacez le code **JSON** dans l’éditeur par le code JSON suivant :
    
    ```json
    {
        "name": "AzureBlobOutput",
        "properties": {
            "type": "AzureBlob",
            "linkedServiceName": "AzureStorageLinkedService1",
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
    L’extrait de code JSON définit un jeu de données appelé **AzureBlobOuput** qui représente les données de sortie produites par l’activité Hive dans le pipeline. Vous indiquez que les données de sortie sont produites par l’activité Hive et placées dans le conteneur de blobs nommé `adfgetstarted` et dans le dossier nommé `partitioneddata`. 
    
    La section **availability** spécifie que le jeu de données de sortie est produit tous les mois. Le jeu de données de sortie détermine la programmation du pipeline. Le pipeline s’exécute tous les mois entre ses heures de début et de fin. 

    Consultez la section **Créer le jeu de données d’entrée** pour obtenir une description de ces propriétés. Vous ne définissez pas la propriété externe sur un jeu de données de sortie, car le jeu de données est produit par le pipeline.
4. Enregistrez le fichier **OutputDataset.json** .

### <a name="create-pipeline"></a>Création d’un pipeline
Jusqu’à présent, vous avez créé le service lié Stockage Azure et les jeux de données d’entrée et de sortie. Vous allez maintenant créer un pipeline avec une activité **HDInsightHive**. **L’entrée** de l’activité Hive a la valeur **AzureBlobInput** et la **sortie** a la valeur **AzureBlobOutput**. Une tranche d’un jeu de données d’entrée est disponible chaque mois (fréquence : mois, intervalle : 1), et la tranche de sortie est générée chaque mois également. 

1. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur **Pipelines**, pointez sur **Ajouter**, puis cliquez sur **Nouvel élément.**
2. Sélectionnez **Pipeline de transformation Hive** dans la liste, puis cliquez sur **Ajouter**.
3. Remplacez le code **JSON** par l’extrait de code suivant :

    > [!IMPORTANT]
    > Remplacez `<storageaccountname>` par le nom de votre compte de stockage.

    ```json
    {
        "name": "MyFirstPipeline",
        "properties": {
            "description": "My first Azure Data Factory pipeline",
            "activities": [
                {
                    "type": "HDInsightHive",
                    "typeProperties": {
                        "scriptPath": "adfgetstarted/script/partitionweblogs.hql",
                        "scriptLinkedService": "AzureStorageLinkedService1",
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
            "start": "2016-04-01T00:00:00Z",
            "end": "2016-04-02T00:00:00Z",
            "isPaused": false
        }
    }
    ```

    > [!IMPORTANT]
    > Remplacez `<storageaccountname>` par le nom de votre compte de stockage.

    L’extrait de code JSON définit un pipeline qui se compose d’une seule activité (activité Hive). Cette activité exécute un script Hive pour traiter les données d’entrée sur un cluster HDInsight à la demande afin de produire des données de sortie. Dans la section des activités du pipeline JSON, vous ne voyez qu’une seule activité dans le tableau avec la valeur de type définie sur **HDInsightHive**. 

    Dans les propriétés de type qui sont spécifiques à l’activité HDInsight Hive, vous indiquez le service lié Stockage Azure disposant du fichier de script Hive, le chemin d’accès au fichier de script et les paramètres au fichier de script. 

    Le fichier de script Hive, **partitionweblogs.hql**, est stocké dans le compte de stockage Azure (spécifié par scriptLinkedService) et dans le dossier `script` du conteneur `adfgetstarted`.

    La section `defines` est utilisée pour spécifier les paramètres d’exécution transmis au script Hive comme valeurs de configuration Hive (p. ex. `${hiveconf:inputtable}`, `${hiveconf:partitionedtable})`.

    Les propriétés **start** et **end** du pipeline spécifient la période active du pipeline. Vous avez configuré le jeu de données à produire tous les mois, par conséquent, la tranche est produite une fois par le pipeline (car le mois est identique dans les dates de début et de fin).

    Dans l’activité JSON, vous spécifiez que le script Hive s’exécute sur le calcul spécifié par le service **linkedServiceName** – **HDInsightOnDemandLinkedService**.
4. Enregistrez le fichier **HiveActivity1.json** .

### <a name="add-partitionweblogshql-and-inputlog-as-a-dependency"></a>Ajouter partitionweblogs.hql et input.log comme dépendance
1. Cliquez avec le bouton droit sur **Dépendances** dans la fenêtre **Explorateur de solutions**, pointez sur **Ajouter**, puis cliquez sur **Élément existant**.  
2. Accédez au dossier **C:\ADFGettingStarted**, sélectionnez les fichiers **partitionweblogs.hql** et **input.log**, puis cliquez sur **Ajouter**. Vous avez créé ces deux fichiers dans le cadre des étapes préalables indiquées dans la [Vue d’ensemble du didacticiel](data-factory-build-your-first-pipeline.md).

Quand vous publiez la solution à l’étape suivante, le fichier **partitionweblogs.hql** est chargé dans le dossier **script** du conteneur de blobs `adfgetstarted`.   

### <a name="publishdeploy-data-factory-entities"></a>Publier/Déployer des entités Data Factory
Dans cette étape, vous allez publier les entités Data Factory (services liés, jeux de données et pipeline) dans votre projet pour le service Azure Data Factory. Lors du processus de publication, vous spécifiez le nom de votre fabrique de données. 

1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet, puis cliquez sur **Publier**.
2. Si la boîte de dialogue **Connectez-vous à votre compte Microsoft** s’affiche, saisissez vos informations d’identification pour le compte associé à l’abonnement Azure, puis cliquez sur **Se connecter**.
3. La boîte de dialogue suivante doit s’afficher :

   ![Boîte de dialogue Publier](./media/data-factory-build-your-first-pipeline-using-vs/publish.png)
4. Dans la page **Configurer une fabrique de données**, procédez comme suit :

    ![Publier - Nouveau paramètres de fabrique de données](media/data-factory-build-your-first-pipeline-using-vs/publish-new-data-factory.png)

   1. Sélectionnez l'option **Créer une fabrique de données** .
   2. Entrez un **nom** unique pour la fabrique de données. Par exemple : **DataFactoryUsingVS09152016**. Le nom doit être globalement unique.
   3. Sélectionnez l’abonnement approprié pour le champ **Abonnement** . 
        > [!IMPORTANT]
        > Si vous ne voyez pas les abonnements, vérifiez que vous êtes connecté à l’aide d’un compte administrateur ou coadministrateur de l’abonnement.
   4. Sélectionnez le **groupe de ressources** pour la fabrique de données à créer.
   5. Sélectionnez la **région** pour la fabrique de données.
   6. Cliquez sur **Suivant** pour basculer vers la page **Publier des éléments**. (Utilisez la touche **TABULATION** pour passer au champ Nom si le bouton **Suivant** est désactivé.)

      > [!IMPORTANT]
      > Si vous recevez l’erreur **Le nom de la fabrique de données « DataFactoryUsingVS » n’est pas disponible** au moment de la publication, changez le nom (par exemple en votrenomDataFactoryUsingVS). Consultez la rubrique [Data Factory - Règles d’affectation des noms](data-factory-naming-rules.md) pour savoir comment nommer les artefacts Data Factory.   
1. Dans la page **Publier des éléments**, vérifiez que toutes les entités de fabriques de données sont sélectionnées, puis cliquez sur **Suivant** pour basculer vers la page **Résumé**.

    ![Page Publier des éléments](media/data-factory-build-your-first-pipeline-using-vs/publish-items-page.png)     
2. Passez en revue le résumé, puis cliquez sur **Suivant** pour démarrer le processus de déploiement et afficher l’**état du déploiement**.

    ![Page de résumé](media/data-factory-build-your-first-pipeline-using-vs/summary-page.png)
3. Dans la page **État du déploiement** , vous devez voir l’état du processus de déploiement. Une fois le déploiement terminé, cliquez sur Terminer.

Quelques points importants à prendre en compte :

- Si vous recevez le message d’erreur : « **L’abonnement n’est pas inscrit pour utiliser l’espace de noms Microsoft.DataFactory** », effectuez l’une des opérations suivantes et essayez de relancer la publication :
    - Dans Azure PowerShell, exécutez la commande suivante pour enregistrer le fournisseur Data Factory.
        ```powershell   
        Register-AzResourceProvider -ProviderNamespace Microsoft.DataFactory
        ```
        Vous pouvez exécuter la commande suivante pour confirmer que le fournisseur Data Factory est bien enregistré.

        ```powershell
        Get-AzResourceProvider
        ```
    - Connectez-vous au [portail Azure](https://portal.azure.com) à l’aide de l’abonnement Azure et accédez à un panneau Data Factory (ou) créez une fabrique de données dans le portail Azure. Cette action enregistre automatiquement le fournisseur.
- Le nom de la fabrique de données pourra être enregistré en tant que nom DNS et devenir ainsi visible publiquement.
- Pour créer des instances Data Factory, vous devez être administrateur ou co-administrateur de l’abonnement Azure

### <a name="monitor-pipeline"></a>Surveillance d’un pipeline
Au cours de cette étape, vous allez surveiller le pipeline à l’aide de la Vue de diagramme de la fabrique de données. 

#### <a name="monitor-pipeline-using-diagram-view"></a>Surveillance d’un pipeline à l’aide de la Vue de diagramme
1. Connectez-vous au [portail Azure](https://portal.azure.com/) et procédez comme suit :
   1. Cliquez sur **Plus de services**, puis sur **Fabriques de données**.
       
        ![Parcourir les fabriques de données](./media/data-factory-build-your-first-pipeline-using-vs/browse-datafactories.png)
   2. Sélectionnez le nom de votre fabrique de données (par exemple : **DataFactoryUsingVS09152016**) dans la liste des fabriques de données.
   
       ![Sélectionner votre fabrique de données](./media/data-factory-build-your-first-pipeline-using-vs/select-first-data-factory.png)
2. Dans la page d’accueil de votre fabrique de données, cliquez sur **Diagramme**.

    ![Vignette de diagramme](./media/data-factory-build-your-first-pipeline-using-vs/diagram-tile.png)
3. Dans la Vue de diagramme, une vue d’ensemble des pipelines et des jeux de données utilisés dans ce didacticiel s’affiche.

    ![Vue du diagramme](./media/data-factory-build-your-first-pipeline-using-vs/diagram-view-2.png)
4. Pour afficher toutes les activités du pipeline, cliquez avec le bouton droit sur le pipeline dans le diagramme, puis cliquez sur Ouvrir un pipeline.

    ![Menu Ouvrir un pipeline](./media/data-factory-build-your-first-pipeline-using-vs/open-pipeline-menu.png)
5. Vérifiez que l’activité HDInsightHive est bien dans le pipeline.

    ![Vue Ouvrir un pipeline](./media/data-factory-build-your-first-pipeline-using-vs/open-pipeline-view.png)

    Pour revenir à la vue précédente, cliquez sur **Fabrique de données** dans le menu de navigation du haut.
6. Dans la **Vue de diagramme**, double-cliquez sur le jeu de données **AzureBlobInput**. Vérifiez que l’état du segment est **Prêt** . Plusieurs minutes peuvent être nécessaires avant que le segment n’apparaisse avec l’état Prêt. Si rien ne se produit au bout d’un moment, vérifiez que le fichier d’entrée (input.log) est bien placé dans le conteneur (`adfgetstarted`) et le dossier (`inputdata`) appropriés. Et assurez-vous que la propriété **externe** du jeu de données d’entrée a la valeur **true**. 

   ![Segment d’entrée dans l’état Prêt](./media/data-factory-build-your-first-pipeline-using-vs/input-slice-ready.png)
7. Cliquez sur **X** pour fermer le panneau **AzureBlobInput**.
8. Dans la **Vue de diagramme**, double-cliquez sur le jeu de données **AzureBlobOutput**. La tranche est en cours de traitement.

   ![Capture d’écran mettant en évidence le jeu de données AzureBlobOutput.](./media/data-factory-build-your-first-pipeline-using-vs/dataset-blade.png)
9. Quand le traitement est terminé, l’état de la tranche est **Prêt** .

   > [!IMPORTANT]
   > La création d’un cluster HDInsight à la demande prend généralement un certain temps (environ 20 minutes). Le pipeline devrait donc traiter la tranche en **30 minutes environ** .  
   
    ![Dataset](./media/data-factory-build-your-first-pipeline-using-vs/dataset-slice-ready.png)    
10. Lorsque la tranche indique l’état **Prêt**, vérifiez le dossier `partitioneddata` dans le conteneur `adfgetstarted` de votre stockage d’objets blob pour les données de sortie.  

    ![données de sortie](./media/data-factory-build-your-first-pipeline-using-vs/three-ouptut-files.png)
11. Cliquez sur la tranche pour en afficher les détails dans le panneau **Tranche de données** .

    ![Détails de la tranche](./media/data-factory-build-your-first-pipeline-using-vs/data-slice-details.png)  
12. Cliquez sur une exécution d’activité (activité Hive dans notre scénario) dans la **liste Exécutions d’activité** pour en afficher les détails dans la fenêtre **Détails de l’exécution d’activité**. 
  
    ![Détails de l'exécution d'activité](./media/data-factory-build-your-first-pipeline-using-vs/activity-window-blade.png)    

    Dans les fichiers journaux, vous pouvez voir la requête Hive qui a été exécutée et son état. Ces journaux d’activité sont utiles pour résoudre les problèmes.  

Consultez [Surveiller les jeux de données et le pipeline](data-factory-monitor-manage-pipelines.md) pour obtenir des instructions sur l’utilisation du portail Azure afin de surveiller le pipeline et les jeux de données que vous avez créés dans ce didacticiel.

#### <a name="monitor-pipeline-using-monitor--manage-app"></a>Surveiller le pipeline à l’aide de l’application de surveillance et de gestion
Vous pouvez également utiliser l’application de surveillance et de gestion pour surveiller vos pipelines. Pour en savoir plus sur l’utilisation de cette application, consultez l’article [Surveiller et gérer les pipelines Azure Data Factory à l’aide de l’application de surveillance et de gestion](data-factory-monitor-manage-app.md).

1. Cliquez sur la vignette Surveiller et gérer.

    ![Vignette Surveiller et gérer](./media/data-factory-build-your-first-pipeline-using-vs/monitor-and-manage-tile.png)
2. L’application de surveillance et de gestion devrait s’afficher. Modifiez l’**heure de début** et l’**heure de fin** pour qu’elles correspondent aux heures de début (04-01-2016 12:00 AM) et de fin (04-02-2016 12:00 AM) de votre pipeline, puis cliquez sur **Appliquer**.

    ![Application de surveillance et gestion](./media/data-factory-build-your-first-pipeline-using-vs/monitor-and-manage-app.png)
3. Pour afficher des informations sur une fenêtre d’activité, sélectionnez-la dans la **liste des fenêtres d’activité**.
    ![Détails de la fenêtre d’activité](./media/data-factory-build-your-first-pipeline-using-vs/activity-window-details.png)

> [!IMPORTANT]
> Le fichier d’entrée sera supprimé lorsque la tranche est traitée avec succès. Par conséquent, si vous souhaitez réexécuter la tranche ou refaire le didacticiel, chargez le fichier d’entrée (input.log) dans le dossier `inputdata` du conteneur `adfgetstarted`.

### <a name="additional-notes"></a>Remarques supplémentaires
- Une fabrique de données peut avoir un ou plusieurs pipelines. Un pipeline peut contenir une ou plusieurs activités. Par exemple, une activité de copie censée copier des données d’un magasin de données source vers un magasin de données de destination, et une activité Hive HDInsight pour exécuter un script Hive pour transformer des données d’entrée. Pour connaître l’ensemble des sources et des récepteurs pris en charge par l’activité de copie, consultez [Banques de données et formats pris en charge](data-factory-data-movement-activities.md#supported-data-stores-and-formats) . Pour obtenir la liste des services de calcul pris en charge par Data Factory, consultez [Services liés de calcul](data-factory-compute-linked-services.md) .
- Les services liés se chargent de lier des magasins de données ou des services de calcul à une fabrique de données Azure Data Factory. Pour connaître l’ensemble des sources et des récepteurs pris en charge par l’activité de copie, consultez [Banques de données et formats pris en charge](data-factory-data-movement-activities.md#supported-data-stores-and-formats) . Pour obtenir la liste des services de calcul pris en charge par Data Factory et les [activités de transformation](data-factory-data-transformation-activities.md) qui peuvent s’exécuter sur ceux-ci, consultez [Services liés de calcul](data-factory-compute-linked-services.md).
- Consultez [Déplacer des données depuis et vers le Stockage Blob Azure à l’aide d’Azure Data Factory](data-factory-azure-blob-connector.md#azure-storage-linked-service) pour plus d’informations sur les propriétés JSON utilisées dans la définition de service lié Stockage Azure.
- Vous pouvez utiliser votre propre cluster HDInsight au lieu d’utiliser un cluster HDInsight à la demande. Pour plus d’informations, consultez [Services de calcul liés](data-factory-compute-linked-services.md) .
-  La fabrique de données crée pour vous un cluster HDInsight **Linux** avec le code JSON précédent. Pour plus d’informations, voir [Service lié à la demande Azure HDInsight](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service) .
- Le cluster HDInsight crée un **conteneur par défaut** dans le Stockage Blob que vous avez spécifié dans le code JSON (linkedServiceName). HDInsight ne supprime pas ce conteneur lorsque le cluster est supprimé. Ce comportement est normal. Avec le service lié HDInsight à la demande, un cluster HDInsight est créé dès qu’une tranche est traitée, à moins qu’il n’existe un cluster actif (timeToLive). Ce cluster est supprimé, une fois le traitement terminé.
    
    Comme un nombre croissant de tranches sont traitées, vous voyez un grand nombre de conteneurs dans votre stockage d’objets blob Azure. Si vous n’en avez pas besoin pour dépanner les travaux, il se peut que vous deviez les supprimer pour réduire les frais de stockage. Les noms de ces conteneurs sont conformes au modèle suivant : `adf**yourdatafactoryname**-**linkedservicename**-datetimestamp`. Utilisez des outils tels que l’[Explorateur Stockage Microsoft Azure](https://storageexplorer.com/) pour supprimer des conteneurs dans votre stockage Blob Azure.
- À ce stade, c'est le jeu de données de sortie qui pilote la planification : vous devez donc créer un jeu de données de sortie même si l’activité ne génère aucune sortie. Si l’activité ne prend aucune entrée, vous pouvez ignorer la création du jeu de données d’entrée. 
- Ce didacticiel n’explique pas comment copier des données à l’aide d’Azure Data Factory. Pour un didacticiel sur la copie de données à l’aide d’Azure Data Factory, consultez [Copie de données Blob Storage vers une base de données SQL à l’aide de Data Factory](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).


## <a name="use-server-explorer-to-view-data-factories"></a>Utiliser l’Explorateur de serveurs pour passer en revue la fabrique des données
1. Dans **Visual Studio**, cliquez sur **Affichage** dans le menu, puis sur **Explorateur de serveurs**.
2. Dans la fenêtre Explorateur de serveurs, développez **Azure**, puis **Data Factory**. Si la boîte de dialogue **Connectez-vous à Visual Studio** s’affiche, saisissez le **compte** associé à votre abonnement Azure, puis cliquez sur **Continuer**. Saisissez le **mot de passe**, puis cliquez sur **Se connecter**. Visual Studio essaie d’obtenir des informations sur toutes les fabriques de données Azure contenues dans votre abonnement. L’état de cette opération s’affiche dans la fenêtre **Liste des tâches de Data Factory** .

    ![Explorateur de serveurs](./media/data-factory-build-your-first-pipeline-using-vs/server-explorer.png)
3. Vous pouvez cliquer avec le bouton droit sur une fabrique de données et sélectionner **Exporter la fabrique de données vers le nouveau projet** pour créer un projet Visual Studio basé sur une fabrique de données existante.

    ![Exporter la fabrique de données](./media/data-factory-build-your-first-pipeline-using-vs/export-data-factory-menu.png)

## <a name="update-data-factory-tools-for-visual-studio"></a>Mettre à jour des outils Data Factory pour Visual Studio
Pour mettre à jour des outils Azure Data Factory pour Visual Studio, procédez comme suit :

1. Dans le menu, cliquez sur **Outils**, puis sélectionnez **Extensions et mises à jour**.
2. Dans le volet de gauche, sélectionnez **Mises à jour**, puis **Galerie Visual Studio**.
3. Sélectionnez **Outils Azure Data Factory pour Visual Studio**, puis cliquez sur **Mettre à jour**. Si cette entrée n’est pas affichée, c’est que vous possédez déjà la dernière version de ces outils.

## <a name="use-configuration-files"></a>Utiliser des fichiers de configuration
Vous pouvez utiliser des fichiers de configuration dans Visual Studio pour configurer les propriétés des services/tableaux/pipelines liés différemment pour chaque environnement.

Examinez la définition JSON suivante pour un service lié Azure Storage. Spécifiez **connectionString** avec différentes valeurs pour accountname et accountkey, en fonction de l’environnement (dév./test/production) sur lequel vous déployez des entités Data Factory. Vous pouvez parvenir à ce comportement en utilisant un fichier de configuration distinct pour chaque environnement.

```json
{
    "name": "StorageLinkedService",
    "properties": {
        "type": "AzureStorage",
        "description": "",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
        }
    }
}
```

### <a name="add-a-configuration-file"></a>Ajouter un fichier de configuration
Ajoutez un fichier de configuration pour chaque environnement en effectuant les opérations suivantes :   

1. Cliquez avec le bouton droit de la souris sur le projet Data Factory dans votre solution Visual Studio, pointez sur **Ajouter**, puis cliquez sur **Nouvel élément**.
2. Sélectionnez **Config** dans la liste des modèles installés sur la gauche, choisissez **Fichier de configuration**, entrez un **nom** pour ce fichier, puis cliquez sur **Ajouter**.

    ![Ajouter un fichier de configuration](./media/data-factory-build-your-first-pipeline-using-vs/add-config-file.png)
3. Ajoutez les paramètres de configuration et leurs valeurs au format suivant :

    ```json
    {
        "$schema": "http://datafactories.schema.management.azure.com/vsschemas/V1/Microsoft.DataFactory.Config.json",
        "AzureStorageLinkedService1": [
            {
                "name": "$.properties.typeProperties.connectionString",
                "value": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
            }
        ],
        "AzureSqlLinkedService1": [
            {
                "name": "$.properties.typeProperties.connectionString",
                "value":  "Server=tcp:<logical SQL server name>.database.windows.net,1433;Database=<Azure Sql database>;User ID=<user name>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
            }
        ]
    }
    ```

    Cet exemple configure la propriété connectionString d’un service lié Azure Storage et d’un service lié SQL Azure. Notez que la syntaxe de spécification du nom est [JsonPath](https://goessner.net/articles/JsonPath/).   

    Si JSON est doté d’une propriété ayant un tableau de valeurs comme indiqué dans le code suivant :  

    ```json
    "structure": [
          {
              "name": "FirstName",
            "type": "String"
          },
          {
            "name": "LastName",
            "type": "String"
        }
    ],
    ```

    Configurez les propriétés comme indiqué dans le fichier de configuration suivant (utilisez indexation de base zéro) :

    ```json
    {
        "name": "$.properties.structure[0].name",
        "value": "FirstName"
    }
    {
        "name": "$.properties.structure[0].type",
        "value": "String"
    }
    {
        "name": "$.properties.structure[1].name",
        "value": "LastName"
    }
    {
        "name": "$.properties.structure[1].type",
        "value": "String"
    }
    ```

### <a name="property-names-with-spaces"></a>Noms de propriétés avec des espaces
Si un nom de propriété comporte des espaces, utilisez des crochets comme indiqué dans l’exemple suivant (nom de serveur de base de données) :

```json
 {
     "name": "$.properties.activities[1].typeProperties.webServiceParameters.['Database server name']",
     "value": "MyAsqlServer.database.windows.net"
 }
```

### <a name="deploy-solution-using-a-configuration"></a>Déployer une solution à l’aide d’une configuration
Lorsque vous publiez des entités Azure Data Factory dans Visual Studio, vous pouvez spécifier la configuration que vous souhaitez utiliser pour cette opération de publication.

Pour publier des entités dans un projet Azure Data Factory à l’aide d’un fichier de configuration :   

1. Cliquez avec le bouton droit sur le projet Data Factory, puis cliquez sur **Publier** pour afficher la boîte de dialogue **Publier des éléments**.
2. Dans la page **Configurer une fabrique de données**, sélectionnez une fabrique de données existante ou spécifiez les valeurs pour en créer une, puis cliquez sur **Suivant**.   
3. La page **Publier des éléments** contient une liste déroulante avec les configurations disponibles pour le champ **Sélectionner une configuration de déploiement**.

    ![Sélectionner un fichier de config](./media/data-factory-build-your-first-pipeline-using-vs/select-config-file.png)
4. Sélectionnez le **fichier de configuration** que vous souhaitez utiliser, puis cliquez sur **Suivant**.
5. Vérifiez que vous voyez bien le nom du fichier JSON dans la page **Résumé**, puis cliquez sur **Suivant**.
6. Une fois l’opération de déploiement terminée, cliquez sur **Terminer** .

Au cours du déploiement, les valeurs du fichier de configuration sont utilisées pour définir celles des propriétés des fichiers JSON avant que les entités ne soient déployées sur le service Azure Data Factory.   

## <a name="use-azure-key-vault"></a>Utiliser Azure Key Vault
Il n’est pas recommandé et souvent déconseillé vis-à-vis de la stratégie de sécurité pour valider des données sensibles telles que des chaînes de connexion au référentiel de code. Consultez l’exemple [ADF Secure Publish](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/ADFSecurePublish) sur GitHub pour en savoir plus sur le stockage d’informations sensibles dans Azure Key Vault et son utilisation lors de la publication des entités Data Factory. L’extension Secure Publish pour Visual Studio permet de stocker les secrets dans Key Vault, et seules les références à ceux-ci sont spécifiés dans des services / configurations de déploiement liés. Ces références sont résolues lorsque vous publiez des entités Data Factory dans Azure. Ces fichiers peuvent ensuite être validés sur le référentiel source sans exposer les secrets.

## <a name="summary"></a>Résumé
Dans ce tutoriel, vous avez créé une fabrique de données Azure Data Factory pour traiter des données en exécutant le script Hive sur un cluster HDInsight Hadoop. Vous avez effectué les étapes suivantes dans le portail Azure à l’aide de Data Factory Editor :  

1. Création d’une **fabrique de données** Azure.
2. Création de deux **services liés**:
   1. **Azure Storage** pour lier à la fabrique de données votre stockage d’objets blob Azure contenant les fichiers d’entrée/sortie.
   2. **Azure HDInsight** à la demande pour lier à la fabrique de données un cluster Hadoop HDInsight à la demande. Azure Data Factory crée un cluster Hadoop HDInsight juste-à-temps pour traiter les données d’entrée et produire des données de sortie.
3. Création de deux **jeux de données** qui décrivent les données d’entrée et de sortie pour l’activité HDInsight Hive dans le pipeline.
4. Création d’un **pipeline** avec une activité **Hive HDInsight**.  

## <a name="next-steps"></a>Étapes suivantes
Dans cet article, vous avez créé un pipeline avec une activité de transformation (Activité HDInsight) qui exécute un script Hive sur un cluster HDInsight à la demande. Pour voir comment utiliser une activité de copie pour copier des données depuis un objet blob Azure vers Azure SQL, consultez le [Didacticiel : copie de données depuis un objet blob Azure vers Azure SQL](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).

Vous pouvez chaîner deux activités (une après l’autre) en configurant le jeu de données de sortie d’une activité en tant que jeu de données d’entrée de l’autre activité. Pour plus d’informations, voir [Planification et exécution dans Data Factory](data-factory-scheduling-and-execution.md). 


## <a name="see-also"></a>Voir aussi

| Rubrique | Description |
|:--- |:--- |
| [Pipelines](data-factory-create-pipelines.md) |Cet article vous aide à comprendre les pipelines et les activités dans Azure Data Factory, et à les utiliser dans l’optique de créer des workflows pilotés par les données pour votre scénario ou votre entreprise. |
| [Groupes de données](data-factory-create-datasets.md) |Cet article vous aide à comprendre les jeux de données dans Azure Data Factory. |
| [Activités de transformation des données](data-factory-data-transformation-activities.md) |Cet article fournit une liste des activités de transformation de données (par exemple, la transformation Hive HDInsight que vous avez utilisée dans ce didacticiel) prises en charge par Azure Data Factory. |
| [Planification et exécution](data-factory-scheduling-and-execution.md) |Cet article explique les aspects de la planification et de l’exécution du modèle d’application Azure Data Factory. |
| [Surveiller et gérer les pipelines Azure Data Factory à l’aide de la nouvelle application de surveillance et gestion.](data-factory-monitor-manage-app.md) |Cet article décrit comment surveiller, gérer et déboguer les pipelines à l’aide de l’application de surveillance et gestion. |
