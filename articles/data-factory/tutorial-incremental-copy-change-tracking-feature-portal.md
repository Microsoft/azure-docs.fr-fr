---
title: Copier de façon incrémentielle des données avec Change Tracking sur le portail Azure
description: Dans ce tutoriel, vous créez une fabrique de données Azure Data Factory avec un pipeline qui charge les données delta basées sur des informations de suivi des modifications dans la base de données source dans Azure SQL Database vers un stockage Blob Azure.
services: data-factory
ms.author: yexu
author: dearandyxu
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: tutorial
ms.custom: seo-lt-2019; seo-dt-2019
ms.date: 01/12/2018
ms.openlocfilehash: 3ea231258f7a60ce90ec119803b5abc8b6e525fe
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97510615"
---
# <a name="incrementally-load-data-from-azure-sql-database-to-azure-blob-storage-using-change-tracking-information-using-the-azure-portal"></a>Charger de façon incrémentielle des données d’Azure SQL Database sur le Stockage Blob Azure en utilisant les informations de suivi des modifications sur le portail Azure

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Dans ce tutoriel, vous créez une fabrique de données Azure Data Factory avec un pipeline qui charge les données delta basées sur des informations de **suivi des modifications** dans la base de données source dans Azure SQL Database vers un stockage Blob Azure.  

Dans ce tutoriel, vous allez effectuer les étapes suivantes :

> [!div class="checklist"]
> * Préparer le magasin de données source
> * Créer une fabrique de données.
> * créez des services liés.
> * Créez des jeux de données source, récepteur et de suivi des modifications.
> * Créer, exécuter et surveiller le pipeline de copie complète
> * Ajouter ou mettre à jour des données dans la table source
> * Créer, exécuter et surveiller le pipeline de copie incrémentielle

## <a name="overview"></a>Vue d’ensemble
Dans une solution d’intégration de données, le chargement incrémentiel de données après des chargements de données initiaux est un scénario largement utilisé. Dans certains cas, les données modifiées pendant une période dans votre magasin de données source peuvent être facilement découpées (par exemple, LastModifyTime, CreationTime). Dans certains cas, il n’existe pas de manière explicite pour identifier les données delta depuis le dernier traitement des données. La technologie Change Tracking prise en charge par les magasins de données tels qu’Azure SQL Database et SQL Server peut être utilisée pour identifier les données delta.  Ce tutoriel explique comment utiliser Azure Data Factory avec la technologie Change Tracking SQL afin de charger de façon incrémentielle des données delta d’Azure SQL Database dans Stockage Blob Azure.  Pour des informations plus concrètes sur la technologie Change Tracking SQL, consultez [Change Tracking dans SQL Server](/sql/relational-databases/track-changes/about-change-tracking-sql-server).

## <a name="end-to-end-workflow"></a>Workflow de bout en bout
Voici les étapes de workflow de bout en bout classiques pour charger de façon incrémentielle des données à l’aide de la technologie Change Tracking.

> [!NOTE]
> Azure SQL Database et SQL Server prennent en charge la technologie Change Tracking. Ce tutoriel utilise Azure SQL Database comme magasin de données source. Vous pouvez également utiliser une instance SQL Server.

1. **Chargement initial de données d’historique** (exécuter une fois) :
    1. Activez la technologie Change Tracking dans la base de données source dans Azure SQL Database.
    2. Obtenez la valeur initiale de SYS_CHANGE_VERSION dans la base de données comme base de référence pour la capture des données modifiées.
    3. Chargez les données complètes de la base de données source vers un compte de stockage blob Azure.
2. **Chargement incrémentiel de données delta selon une planification** (exécuter périodiquement après le chargement initial des données) :
    1. Obtenez les valeurs SYS_CHANGE_VERSION anciennes et nouvelles.
    3. Chargez les données delta en associant les clés primaires des lignes modifiées (entre deux valeurs SYS_CHANGE_VERSION) de **sys.change_tracking_tables** avec des données dans la **table source**, puis déplacez les données delta vers la destination.
    4. Mettez à jour SYS_CHANGE_VERSION pour le chargement delta suivant.

## <a name="high-level-solution"></a>Solution générale
Dans ce didacticiel, vous créez deux pipelines qui effectuent les deux opérations suivantes :  

1. **Chargement initial :** vous créez un pipeline avec une activité de copie qui copie l’ensemble des données du magasin de données source (Azure SQL Database) dans le magasin de données de destination (Stockage Blob Azure).

    ![Chargement complet des données](media/tutorial-incremental-copy-change-tracking-feature-portal/full-load-flow-diagram.png)
1.  **Chargement incrémentiel :** vous créez un pipeline avec les activités suivantes, et vous l’exécutez régulièrement.
    1. Créez **deux activités de recherche** pour obtenir les valeurs SYS_CHANGE_VERSION anciennes et nouvelles dans Azure SQL Database et les transmettre à l’activité de copie.
    2. Créez **une activité de copie** pour copier les données insérées/mises à jour/supprimées entre deux valeurs SYS_CHANGE_VERSION d’Azure SQL Database dans Stockage Blob Azure.
    3. Créez **une activité de procédure stockée** pour mettre à jour la valeur de SYS_CHANGE_VERSION pour la prochaine exécution du pipeline.

    ![Diagramme de flux de chargement incrémentiel](media/tutorial-incremental-copy-change-tracking-feature-portal/incremental-load-flow-diagram.png)


Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis
* **Azure SQL Database**. Vous utilisez la base de données comme magasin de données **sources**. Si vous n’avez pas de base de données dans Azure SQL Database, consultez l’article [Créer une base de données dans Azure SQL Database](../azure-sql/database/single-database-create-quickstart.md) pour savoir comme en créer une.
* **Compte Stockage Azure**. Vous utilisez le stockage Blob comme magasin de données **récepteur**. Si vous n’avez pas de compte de stockage Azure, consultez l’article [Créer un compte de stockage](../storage/common/storage-account-create.md) pour découvrir comment en créer un. Créez un conteneur sous le nom **adftutorial**. 

### <a name="create-a-data-source-table-in-azure-sql-database"></a>Créer une table de source de données dans Azure SQL Database

1. Lancez **SQL Server Management Studio**, puis connectez-vous à SQL Database.
2. Dans l’**Explorateur de serveurs**, cliquez avec le bouton droit sur votre **base de données** et choisissez **Nouvelle requête**.
3. Exécutez la commande SQL suivante sur votre base de données pour créer une table sous le nom `data_source_table` comme magasin de la source de données.  

    ```sql
    create table data_source_table
    (
        PersonID int NOT NULL,
        Name varchar(255),
        Age int
        PRIMARY KEY (PersonID)
    );

    INSERT INTO data_source_table
        (PersonID, Name, Age)
    VALUES
        (1, 'aaaa', 21),
        (2, 'bbbb', 24),
        (3, 'cccc', 20),
        (4, 'dddd', 26),
        (5, 'eeee', 22);

    ```

4. Activez le mécanisme **Change Tracking** sur votre base de données et la table source (data_source_table) en exécutant la requête SQL suivante :

    > [!NOTE]
    > - Remplacez le &lt;nom de votre base de données&gt; par le nom de celle qui contient data_source_table dans Azure SQL Database.
    > - Dans cet exemple, les données modifiées sont conservées pendant deux jours. Si vous chargez les données modifiées tous les trois jours ou plus, certaines données modifiées ne sont pas incluses.  Vous devez remplacer la valeur de CHANGE_RETENTION par un plus grand nombre. Assurez-vous également que votre période pour charger les données modifiées est de moins de deux jours. Pour plus d’informations, consultez [Activer le suivi des modifications pour une base de données](/sql/relational-databases/track-changes/enable-and-disable-change-tracking-sql-server#enable-change-tracking-for-a-database)

    ```sql
    ALTER DATABASE <your database name>
    SET CHANGE_TRACKING = ON  
    (CHANGE_RETENTION = 2 DAYS, AUTO_CLEANUP = ON)  

    ALTER TABLE data_source_table
    ENABLE CHANGE_TRACKING  
    WITH (TRACK_COLUMNS_UPDATED = ON)
    ```
5. Créez une nouvelle table et stockez la ChangeTracking_version avec une valeur par défaut en exécutant la requête suivante :

    ```sql
    create table table_store_ChangeTracking_version
    (
        TableName varchar(255),
        SYS_CHANGE_VERSION BIGINT,
    );

    DECLARE @ChangeTracking_version BIGINT
    SET @ChangeTracking_version = CHANGE_TRACKING_CURRENT_VERSION();  

    INSERT INTO table_store_ChangeTracking_version
    VALUES ('data_source_table', @ChangeTracking_version)
    ```

    > [!NOTE]
    > Si les données ne sont pas modifiées une fois que vous avez activé le suivi des modifications pour SQL Database, la valeur de la version de suivi des modifications est 0.
6. Exécutez la requête suivante pour créer une procédure stockée dans votre base de données. Le pipeline appelle cette procédure stockée pour mettre à jour la version de suivi des modifications dans la table que vous avez créée à l’étape précédente.

    ```sql
    CREATE PROCEDURE Update_ChangeTracking_Version @CurrentTrackingVersion BIGINT, @TableName varchar(50)
    AS

    BEGIN

    UPDATE table_store_ChangeTracking_version
    SET [SYS_CHANGE_VERSION] = @CurrentTrackingVersion
    WHERE [TableName] = @TableName

    END    
    ```

### <a name="azure-powershell"></a>Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Installez les modules Azure PowerShell les plus récents en suivant les instructions décrites dans [Comment installer et configurer Azure PowerShell](/powershell/azure/install-Az-ps).

## <a name="create-a-data-factory"></a>Créer une fabrique de données

1. Lancez le navigateur web **Microsoft Edge** ou **Google Chrome**. L’interface utilisateur de Data Factory n’est actuellement prise en charge que par les navigateurs web Microsoft Edge et Google Chrome.
1. Dans le menu de gauche, sélectionnez **Créer une ressource** > **Données + Analytique** > **Data Factory** :

   ![Sélection Data Factory dans le volet « Nouveau »](./media/quickstart-create-data-factory-portal/new-azure-data-factory-menu.png)

2. Dans la page **Nouvelle fabrique de données**, entrez **ADFTutorialDataFactory** comme **nom**.

     ![Page Nouvelle fabrique de données](./media/tutorial-incremental-copy-change-tracking-feature-portal/new-azure-data-factory.png)

   Le nom de la fabrique de données Azure doit être **globalement unique**. Si l’erreur suivante s’affiche, changez le nom de la fabrique de données (par exemple, votrenomADFTutorialDataFactory), puis tentez de la recréer. Consultez l’article [Data Factory - Règles d’affectation des noms](naming-rules.md) pour savoir comment nommer les artefacts Data Factory.

   *Le nom de fabrique de données « ADFTutorialDataFactory » n’est pas disponible*
3. Sélectionnez l’**abonnement** Azure dans lequel vous voulez créer la fabrique de données.
4. Pour le **groupe de ressources**, effectuez l’une des opérations suivantes :

      - Sélectionnez **Utiliser l’existant**, puis sélectionnez un groupe de ressources existant dans la liste déroulante.
      - Sélectionnez **Créer**, puis entrez le nom d’un groupe de ressources.   
         
        Pour plus d’informations sur les groupes de ressources, consultez [Utilisation des groupes de ressources pour gérer vos ressources Azure](../azure-resource-manager/management/overview.md).  
4. Sélectionnez **V2 (préversion)** pour la **version**.
5. Sélectionnez **l’emplacement** de la fabrique de données. Seuls les emplacements pris en charge sont affichés dans la liste déroulante. Les magasins de données (Stockage Azure, Azure SQL Database, etc.) et les services de calcul (HDInsight, etc.) utilisés par la fabrique de données peuvent être proposés dans d’autres régions.
6. Sélectionnez **Épingler au tableau de bord**.     
7. Cliquez sur **Créer**.      
8. Sur le tableau de bord, vous voyez la vignette suivante avec l’état : **Déploiement de Data Factory**.

    ![mosaïque déploiement de fabrique de données](media/tutorial-incremental-copy-change-tracking-feature-portal/deploying-data-factory.png)
9. Une fois la création terminée, la page **Data Factory** s’affiche comme sur l’image.

   ![Page d’accueil Data Factory](./media/tutorial-incremental-copy-change-tracking-feature-portal/data-factory-home-page.png)
10. Cliquez sur la vignette **Créer et surveiller** pour lancer l’interface utilisateur d’Azure Data Factory dans un onglet séparé.
11. Dans la page **Prise en main**, basculez vers l’onglet **Modifier** dans le volet gauche comme illustré dans l’image suivante :

    ![Bouton Créer un pipeline](./media/tutorial-incremental-copy-change-tracking-feature-portal/get-started-page.png)

## <a name="create-linked-services"></a>Créez des services liés
Vous allez créer des services liés dans une fabrique de données pour lier vos magasins de données et vos services de calcul à la fabrique de données. Dans cette section, vous allez créer des services liés à votre compte de stockage Azure et à votre base de données dans Azure SQL Database.

### <a name="create-azure-storage-linked-service"></a>Créer un service lié Stockage Azure.
Dans cette étape, vous liez votre compte Stockage Azure à la fabrique de données.

1. Cliquez sur **Connexions**, puis sur **+ Nouveau**.

   ![Bouton Nouvelle connexion](./media/tutorial-incremental-copy-change-tracking-feature-portal/new-connection-button-storage.png)
2. Dans la fenêtre **Nouveau service lié**, sélectionnez **Stockage Blob Azure**, puis cliquez sur **Continuer**.

   ![Sélectionner le stockage Blob Azure](./media/tutorial-incremental-copy-change-tracking-feature-portal/select-azure-storage.png)
3. Dans la fenêtre **Nouveau service lié**, procédez comme suit :

    1. Entrez **AzureStorageLinkedService** pour **Nom**.
    2. Sélectionnez votre compte de stockage Azure comme **Nom du compte de stockage**.
    3. Cliquez sur **Enregistrer**.

   ![Paramètres du compte de stockage Azure](./media/tutorial-incremental-copy-change-tracking-feature-portal/azure-storage-linked-service-settings.png)


### <a name="create-azure-sql-database-linked-service"></a>Créez le service lié Azure SQL Database.
Dans cette étape, vous liez votre base de données à la fabrique de données.

1. Cliquez sur **Connexions**, puis sur **+ Nouveau**.
2. Dans la fenêtre **Nouveau service lié**, sélectionnez **Azure SQL Database**, puis cliquez sur **Continuer**.
3. Dans la fenêtre **Nouveau service lié**, procédez comme suit :

    1. Entrez **AzureSqlDatabaseLinkedService** pour le champ **Nom**.
    2. Sélectionnez votre serveur pour le champ **Nom du serveur**.
    3. Sélectionnez votre base de données pour le champ **Nom de la base de données**.
    4. Entrez le nom de l’utilisateur pour le champ **Nom d’utilisateur**.
    5. Entrez le mot de passe de l’utilisateur pour le champ **Mot de passe**.
    6. Cliquez sur **Tester la connexion** pour tester la connexion.
    7. Cliquez sur **Enregistrer** pour enregistrer le service lié.

       ![Paramètres du service lié Azure SQL Database](./media/tutorial-incremental-copy-change-tracking-feature-portal/azure-sql-database-linked-service-settings.png)

## <a name="create-datasets"></a>Créez les jeux de données
Dans cette étape, vous créez des jeux de données pour représenter la source de données et la destination de données. et l’emplacement pour stocker SYS_CHANGE_VERSION.

### <a name="create-a-dataset-to-represent-source-data"></a>Créer un jeu de données pour représenter les données sources
Dans cette étape, vous créez un jeu de données pour représenter les données source.

1. Dans l’arborescence, cliquez sur **+ (plus)** , puis sur **Jeu de données**.

   ![Menu Nouveau jeu de données](./media/tutorial-incremental-copy-change-tracking-feature-portal/new-dataset-menu.png)
2. Sélectionnez **Azure SQL Database**, puis cliquez sur **Terminer**.

   ![Type de jeu de données source - Azure SQL Database](./media/tutorial-incremental-copy-change-tracking-feature-portal/select-azure-sql-database.png)
3. Vous voyez un nouvel onglet pour configurer le jeu de données. Vous voyez également le jeu de données dans l’arborescence. Dans la fenêtre **Propriétés**, renommez le jeu de données en **SourceDataset**.

   ![Nom du jeu de données source](./media/tutorial-incremental-copy-change-tracking-feature-portal/source-dataset-name.png)    
4. Basculez vers l’onglet **Connexions**, et procédez comme suit :

    1. Sélectionnez **AzureSqlDatabaseLinkedService** pour **Service lié**.
    2. Sélectionnez **[dbo].[ data_source_table]** pour **Table**.

   ![Connexion source](./media/tutorial-incremental-copy-change-tracking-feature-portal/source-dataset-connection.png)

### <a name="create-a-dataset-to-represent-data-copied-to-sink-data-store"></a>Créez un jeu de données pour représenter les données copiées dans le magasin de données récepteur.
Dans cette étape, vous créez un jeu de données pour représenter les données copiées à partir du magasin de données source. Vous avez créé le conteneur adftutorial dans votre stockage Blob Azure dans le cadre des conditions préalables. Créez le conteneur s’il n’existe pas (ou) attribuez-lui le nom d’un conteneur existant. Dans ce didacticiel, le nom de fichier de sortie est généré dynamiquement à l’aide de l’expression : `@CONCAT('Incremental-', pipeline().RunId, '.txt')`.

1. Dans l’arborescence, cliquez sur **+ (plus)** , puis sur **Jeu de données**.

   ![Menu Nouveau jeu de données](./media/tutorial-incremental-copy-change-tracking-feature-portal/new-dataset-menu.png)
2. Sélectionnez **Stockage Blob Azure**, puis cliquez sur **Terminer**.

   ![Type de jeu de données récepteur - Stockage Blob Azure](./media/tutorial-incremental-copy-change-tracking-feature-portal/source-dataset-type.png)
3. Vous voyez un nouvel onglet pour configurer le jeu de données. Vous voyez également le jeu de données dans l’arborescence. Dans la fenêtre **Propriétés**, renommez le jeu de données en **SinkDataset**.

   ![Jeu de données récepteur - nom](./media/tutorial-incremental-copy-change-tracking-feature-portal/sink-dataset-name.png)
4. Basculez vers l’onglet **Connexions** dans la fenêtre Propriétés, et procédez comme suit :

    1. Sélectionnez **AzureStorageLinkedService** pour **Service lié**.
    2. Entrez **adftutorial/incchgtracking** pour la partie **dossier** de **filePath**.
    3. Entrez **\@CONCAT('Incremental-', pipeline().RunId, '.txt')** pour la partie **fichier** de **filePath**.  

       ![Jeu de données récepteur - connexion](./media/tutorial-incremental-copy-change-tracking-feature-portal/sink-dataset-connection.png)

### <a name="create-a-dataset-to-represent-change-tracking-data"></a>Créer un jeu de données pour représenter les données de suivi des modifications
Dans cette étape, vous créez un jeu de données pour stocker la version de suivi des modifications.  Vous avez créé la table table_store_ChangeTracking_version dans le cadre des conditions préalables.

1. Dans l’arborescence, cliquez sur **+ (plus)** , puis sur **Jeu de données**.
2. Sélectionnez **Azure SQL Database**, puis cliquez sur **Terminer**.
3. Vous voyez un nouvel onglet pour configurer le jeu de données. Vous voyez également le jeu de données dans l’arborescence. Dans la fenêtre **Propriétés**, renommez le jeu de données en **ChangeTrackingDataset**.
4. Basculez vers l’onglet **Connexions**, et procédez comme suit :

    1. Sélectionnez **AzureSqlDatabaseLinkedService** pour **Service lié**.
    2. Sélectionnez **[dbo].[table_store_ChangeTracking_version]** pour **Table**.

## <a name="create-a-pipeline-for-the-full-copy"></a>Créer un pipeline pour la copie complète
Dans cette étape, vous créez un pipeline avec une activité de copie qui copie l’ensemble des données du magasin de données source (Azure SQL Database) dans le magasin de données de destination (Stockage Blob Azure).

1. Cliquez sur **+ (plus)** dans le volet gauche, puis cliquez sur **Pipeline**.

    ![Capture d’écran montrant l’option de pipeline pour une fabrique de données.](./media/tutorial-incremental-copy-change-tracking-feature-portal/new-pipeline-menu.png)
2. Vous voyez un nouvel onglet pour configurer le pipeline. Vous voyez également le pipeline dans l’arborescence. Dans la fenêtre **Propriétés**, renommez le pipeline en **FullCopyPipeline**.

    ![Capture d’écran montrant un pipeline avec un nom entré.](./media/tutorial-incremental-copy-change-tracking-feature-portal/full-copy-pipeline-name.png)
3. Dans la boîte à outils **Activités**, développez **Flux de données** et glissez-déposez l’activité **Copie** vers la surface du concepteur de pipeline, puis définissez le nom **FullCopyActivity**.

    ![Activité de copie complète - nom](./media/tutorial-incremental-copy-change-tracking-feature-portal/full-copy-activity-name.png)
4. Basculez vers l’onglet **Source** et sélectionnez **SourceDataset** pour le champ **Jeu de données source**.

    ![Activité de copie - source](./media/tutorial-incremental-copy-change-tracking-feature-portal/copy-activity-source.png)
5. Basculez vers l’onglet **Récepteur** et sélectionnez **SinkDataset** pour le champ **Jeu de données récepteur**.

    ![Activité de copie - récepteur](./media/tutorial-incremental-copy-change-tracking-feature-portal/copy-activity-sink.png)
6. Pour valider la définition du pipeline, cliquez sur **Valider** dans la barre d’outils. Vérifiez qu’il n’y a aucune erreur de validation. Fermez la fenêtre **Rapport de validation de pipeline** en cliquant sur **>>** .

    ![Valider le pipeline](./media/tutorial-incremental-copy-change-tracking-feature-portal/full-copy-pipeline-validate.png)
7. Pour publier des entités (services liés, jeux de données et pipelines), cliquez sur **Publier**. Patientez jusqu’à ce que la publication réussisse.

    ![Capture d’écran affichant la fabrique de données avec le bouton Publier tout sélectionné.](./media/tutorial-incremental-copy-change-tracking-feature-portal/publish-button.png)
8. Patientez jusqu’à voir le message **Publication réussie**.

    ![Publication réussie](./media/tutorial-incremental-copy-change-tracking-feature-portal/publishing-succeeded.png)
9. Vous pouvez également voir des notifications en cliquant sur le bouton **Afficher les Notifications** à gauche. Pour fermer la fenêtre de notifications, cliquez sur **X**.

    ![Afficher les notifications](./media/tutorial-incremental-copy-change-tracking-feature-portal/show-notifications.png)


### <a name="run-the-full-copy-pipeline"></a>Exécuter le pipeline de copie complète
Cliquez sur **Déclencher** dans la barre d’outils du pipeline, puis cliquez sur **Déclencher maintenant**.

![Capture d’écran montre l’option Déclencher maintenant sélectionnée dans le menu du déclencheur.](./media/tutorial-incremental-copy-change-tracking-feature-portal/trigger-now-menu.png)

### <a name="monitor-the-full-copy-pipeline"></a>Surveiller le pipeline de copie complète

1. Cliquez sur l’onglet **Surveiller** sur la gauche. Vous voyez l’exécution du pipeline dans la liste et son état. Pour actualiser la liste, cliquez sur **Actualiser**. Les liens dans la colonne Actions vous permettent de visualiser les exécutions d’activités associées à l’exécution du pipeline et de réexécuter le pipeline.

    ![Capture d’écran montrant les exécutions de pipeline pour une fabrique de données.](./media/tutorial-incremental-copy-change-tracking-feature-portal/monitor-full-copy-pipeline-run.png)
2. Pour afficher les exécutions d’activités associées à l’exécution du pipeline, cliquez sur le lien **Afficher les exécutions d’activités** dans la colonne **Actions**. Il n’y a qu’une seule activité dans le pipeline, vous ne voyez donc qu’une seule entrée dans la liste. Pour revenir à l’affichage des exécutions du pipeline, cliquez sur le lien **Pipelines** en haut.

    ![Capture d’écran montrant les exécutions d’activités pour une fabrique de données avec le lien Pipelines sélectionné.](./media/tutorial-incremental-copy-change-tracking-feature-portal/activity-runs-full-copy.png)

### <a name="review-the-results"></a>Passer en revue les résultats.
Vous voyez un fichier nommé `incremental-<GUID>.txt` dans le dossier `incchgtracking` du conteneur `adftutorial`.

![Fichier de sortie d’une copie complète](media/tutorial-incremental-copy-change-tracking-feature-portal/full-copy-output-file.png)

Le fichier doit contenir les données de votre base de données :

```
1,aaaa,21
2,bbbb,24
3,cccc,20
4,dddd,26
5,eeee,22
```

## <a name="add-more-data-to-the-source-table"></a>Ajouter plus de données à la table source

Exécutez la requête suivante par rapport à votre base de données pour ajouter une ligne et mettre à jour une ligne.

```sql
INSERT INTO data_source_table
(PersonID, Name, Age)
VALUES
(6, 'new','50');


UPDATE data_source_table
SET [Age] = '10', [name]='update' where [PersonID] = 1

```

## <a name="create-a-pipeline-for-the-delta-copy"></a>Créer un pipeline pour la copie delta
Dans cette étape, vous créez un pipeline avec les activités suivantes, et vous l’exécutez régulièrement. Les **activités de recherche** obtiennent les valeurs SYS_CHANGE_VERSION anciennes et nouvelles dans Azure SQL Database et les transmettent à l’activité de copie. L’**activité de copie** copie les données insérées/mises à jour/supprimées entre deux valeurs SYS_CHANGE_VERSION d’Azure SQL Database dans Stockage Blob Azure. L’**activité de procédure stockée** met à jour la valeur de SYS_CHANGE_VERSION pour la prochaine exécution du pipeline.

1. Dans l’interface utilisateur de Data Factory, basculez vers l’onglet **Modifier**. Cliquez sur **+ (plus)** dans le volet gauche, puis cliquez sur **Pipeline**.

    ![Capture d’écran montrant comment créer un pipeline dans une fabrique de données.](./media/tutorial-incremental-copy-change-tracking-feature-portal/new-pipeline-menu-2.png)
2. Vous voyez un nouvel onglet pour configurer le pipeline. Vous voyez également le pipeline dans l’arborescence. Dans la fenêtre **Propriétés**, renommez le pipeline en **IncrementalCopyPipeline**.

    ![Nom du pipeline](./media/tutorial-incremental-copy-change-tracking-feature-portal/incremental-copy-pipeline-name.png)
3. Développez **Général** dans la boîte à outils **Activités**, puis faites glisser et déposez une activité **Recherche** sur la surface du concepteur de pipeline. Définissez le nom de l’activité sur **LookupLastChangeTrackingVersionActivity**. Cette activité permet d’obtenir la version de suivi des modifications utilisée dans la dernière opération de copie qui est stockée dans la table **table_store_ChangeTracking_version**.

    ![Capture d’écran montrant un pipeline avec une activité de recherche.](./media/tutorial-incremental-copy-change-tracking-feature-portal/first-lookup-activity-name.png)
4. Basculez vers **Paramètres** dans la fenêtre **Propriétés**, puis sélectionnez **ChangeTrackingDataset** pour le champ **Jeu de données source**.

    ![Capture d’écran montrant l’onglet Paramètres dans la fenêtre Propriétés.](./media/tutorial-incremental-copy-change-tracking-feature-portal/first-lookup-activity-settings.png)
5. Glissez-déposez l’activité **Recherche** de la boîte à outils **Activités** vers la surface du concepteur de pipeline. Définissez le nom de l’activité sur **LookupCurrentChangeTrackingVersionActivity**. Cette activité permet d’obtenir la version de suivi des modifications en cours.

    ![Capture d’écran montrant un pipeline avec deux activités de recherche.](./media/tutorial-incremental-copy-change-tracking-feature-portal/second-lookup-activity-name.png)
6. Basculez vers les **Paramètres** dans la fenêtre **Propriétés**, et procédez comme suit :

   1. Sélectionnez **SourceDataset** pour le champ **Jeu de données source**.
   2. Sélectionnez **Requête** pour **Utiliser la requête**.
   3. Entrez la requête SQL suivante pour **Requête**.

       ```sql
       SELECT CHANGE_TRACKING_CURRENT_VERSION() as CurrentChangeTrackingVersion
       ```

      ![Capture d’écran montrant une requête ajoutée à l’onglet Paramètres dans la fenêtre Propriétés.](./media/tutorial-incremental-copy-change-tracking-feature-portal/second-lookup-activity-settings.png)
7. Dans la boîte à outils **Activités**, développez **Flux de données** et glissez-déposez l’activité **Copie** vers la surface du concepteur de pipeline. Définissez le nom de l’activité sur **IncrementalCopyActivity**. Cette activité permet de copier les données entre la dernière version de suivi des modifications et la version de suivi des modifications en cours dans le magasin de données de destination.

    ![Activité de copie - nom](./media/tutorial-incremental-copy-change-tracking-feature-portal/incremental-copy-activity-name.png)
8. Basculez vers l’onglet **Source** dans la fenêtre **Propriétés**, et procédez comme suit :

   1. Sélectionnez **SourceDataset** pour **Jeu de données source**.
   2. Sélectionnez **Requête** pour **Utiliser la requête**.
   3. Entrez la requête SQL suivante pour **Requête**.

       ```sql
       select data_source_table.PersonID,data_source_table.Name,data_source_table.Age, CT.SYS_CHANGE_VERSION, SYS_CHANGE_OPERATION from data_source_table RIGHT OUTER JOIN CHANGETABLE(CHANGES data_source_table, @{activity('LookupLastChangeTrackingVersionActivity').output.firstRow.SYS_CHANGE_VERSION}) as CT on data_source_table.PersonID = CT.PersonID where CT.SYS_CHANGE_VERSION <= @{activity('LookupCurrentChangeTrackingVersionActivity').output.firstRow.CurrentChangeTrackingVersion}
       ```

      ![Activité de copie - paramètres de la source](./media/tutorial-incremental-copy-change-tracking-feature-portal/inc-copy-source-settings.png)
9. Basculez vers l’onglet **Récepteur** et sélectionnez **SinkDataset** pour le champ **Jeu de données récepteur**.

    ![Activité de copie - paramètres du récepteur](./media/tutorial-incremental-copy-change-tracking-feature-portal/inc-copy-sink-settings.png)
10. **Connectez les deux activités de recherche à l’activité de copie** une par une. Faites glisser le bouton **vert** associé à l’activité **Recherche** vers l’activité **Copie**.

    ![Connecter des activités de recherche et de copie](./media/tutorial-incremental-copy-change-tracking-feature-portal/connect-lookup-and-copy.png)
11. Glissez-déposez l’activité **Procédure stockée** de la boîte à outils **Activités** vers la surface du concepteur de pipeline. Définissez le nom de l’activité sur **StoredProceduretoUpdateChangeTrackingActivity**. Cette activité permet de mettre à jour la version de suivi des modifications dans la table **table_store_ChangeTracking_version**.

    ![Activité de procédure stockée - nom](./media/tutorial-incremental-copy-change-tracking-feature-portal/stored-procedure-activity-name.png)
12. Basculez vers l’onglet *Compte SQL** et sélectionnez **AzureSqlDatabaseLinkedService** pour **Service lié**.

    ![Activité de procédure stockée - Compte SQL](./media/tutorial-incremental-copy-change-tracking-feature-portal/sql-account-tab.png)
13. Basculez vers l’onglet **Procédure stockée**, et procédez comme suit :

    1. Pour **Nom de la procédure stockée**, sélectionnez **Update_ChangeTracking_Version**.  
    2. Sélectionnez **Import parameter** (Paramètre d’importation).
    3. Dans la section **Paramètres de procédure stockée**, spécifiez les valeurs suivantes pour les paramètres :

        | Nom | Type | Valeur |
        | ---- | ---- | ----- |
        | CurrentTrackingVersion | Int64 | @{activity('LookupCurrentChangeTrackingVersionActivity').output.firstRow.CurrentChangeTrackingVersion} |
        | TableName | String | @{activity('LookupLastChangeTrackingVersionActivity').output.firstRow.TableName} |

        ![Activité de procédure stockée - Paramètres](./media/tutorial-incremental-copy-change-tracking-feature-portal/stored-procedure-parameters.png)
14. **Connectez l’activité de copie à l’activité de procédure stockée**. Glissez-déposez le bouton **vert** associé à l’activité de copie l’activité de procédure stockée.

    ![Connecter les activités de copie et de procédure stockée](./media/tutorial-incremental-copy-change-tracking-feature-portal/connect-copy-stored-procedure.png)
15. Cliquez sur **Valider** dans la barre d’outils. Vérifiez qu’il n’y a aucune erreur de validation. Fermez la fenêtre **Rapport de validation de pipeline** en cliquant sur **>>** .

    ![Bouton de validation](./media/tutorial-incremental-copy-change-tracking-feature-portal/validate-button.png)
16. Publiez des entités (services liés, jeux de données et pipelines) sur le service Data Factory en cliquant sur le bouton **Publish All** (Tout publier). Patientez jusqu’à ce que le message **Publication réussie** s’affiche.

       ![Capture d’écran montrant le bouton Publier tout pour une fabrique de données.](./media/tutorial-incremental-copy-change-tracking-feature-portal/publish-button-2.png)    

### <a name="run-the-incremental-copy-pipeline"></a>Exécuter le pipeline de copie incrémentielle
1. Cliquez sur **Déclencher** dans la barre d’outils du pipeline, puis cliquez sur **Déclencher maintenant**.

    ![Capture d’écran montrant un pipeline avec des activités, avec l’option Déclencher maintenant sélectionnée dans le menu du déclencheur.](./media/tutorial-incremental-copy-change-tracking-feature-portal/trigger-now-menu-2.png)
2. Dans la fenêtre **Exécution du pipeline**, sélectionnez **Terminer**.

### <a name="monitor-the-incremental-copy-pipeline"></a>Surveiller le pipeline de copie incrémentielle
1. Cliquez sur l’onglet **Surveiller** sur la gauche. Vous voyez l’exécution du pipeline dans la liste et son état. Pour actualiser la liste, cliquez sur **Actualiser**. Les liens dans la colonne **Actions** vous permettent de visualiser les exécutions d’activités associées à l’exécution du pipeline et de réexécuter le pipeline.

    ![Capture d’écran montrant les exécutions de pipeline pour une fabrique de données incluant votre pipeline.](./media/tutorial-incremental-copy-change-tracking-feature-portal/inc-copy-pipeline-runs.png)
2. Pour afficher les exécutions d’activités associées à l’exécution du pipeline, cliquez sur le lien **Afficher les exécutions d’activités** dans la colonne **Actions**. Il n’y a qu’une seule activité dans le pipeline, vous ne voyez donc qu’une seule entrée dans la liste. Pour revenir à l’affichage des exécutions du pipeline, cliquez sur le lien **Pipelines** en haut.

    ![Capture d’écran montrant des exécutions de pipeline pour une fabrique de données avec plusieurs exécutions marquées comme ayant réussi.](./media/tutorial-incremental-copy-change-tracking-feature-portal/inc-copy-activity-runs.png)


### <a name="review-the-results"></a>Passer en revue les résultats.
Vous voyez le second fichier dans le dossier `incchgtracking` du conteneur `adftutorial`.

![Fichier de sortie de la copie incrémentielle](media/tutorial-incremental-copy-change-tracking-feature-portal/incremental-copy-output-file.png)

Le fichier ne doit contenir que les données delta de votre base de données. L’enregistrement avec `U` correspond à la ligne mise à jour dans la base de données et `I` à la ligne ajoutée.

```
1,update,10,2,U
6,new,50,1,I
```
Les trois premières colonnes correspondent aux données modifiées de data_source_table. Les deux dernières colonnes correspondent aux métadonnées de la table système de suivi des modifications. La quatrième colonne correspond à SYS_CHANGE_VERSION de chaque ligne modifiée. La cinquième colonne correspond à l’opération :  U = mise à jour, I = insertion.  Pour plus d’informations sur le suivi des modifications, consultez [CHANGETABLE](/sql/relational-databases/system-functions/changetable-transact-sql).

```
==================================================================
PersonID Name    Age    SYS_CHANGE_VERSION    SYS_CHANGE_OPERATION
==================================================================
1        update  10            2                                 U
6        new     50            1                                 I
```

## <a name="next-steps"></a>Étapes suivantes
Passez au tutoriel suivant pour en savoir plus sur la copie des fichiers nouveaux et modifiés uniquement en fonction de leur LastModifiedDate :

> [!div class="nextstepaction"]
> [Copier les nouveaux fichiers par lastmodifieddate](tutorial-incremental-copy-lastmodified-copy-data-tool.md)
