---
title: Copier de façon incrémentielle des données avec la capture des changements de données
description: Dans ce tutoriel, vous allez créer un pipeline Azure Data Factory qui copie de façon incrémentielle les données delta d’une table de base de données Azure SQL Managed Instance vers le Stockage Azure.
services: data-factory
ms.author: nihurt
author: hurtn
manager: ''
ms.reviewer: ''
ms.service: data-factory
ms.workload: data-services
ms.topic: tutorial
ms.custom: ''
ms.date: 05/04/2020
ms.openlocfilehash: fd9e78b6bc3513f79b05c9522e891d346e3d31a0
ms.sourcegitcommit: fb3c846de147cc2e3515cd8219d8c84790e3a442
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92637511"
---
# <a name="incrementally-load-data-from-azure-sql-managed-instance-to-azure-storage-using-change-data-capture-cdc"></a>Charger de façon incrémentielle des données d’Azure SQL Managed Instance sur le Stockage Azure à l’aide de la capture des changements de données

Dans ce tutoriel, vous allez créer une fabrique de données Azure avec un pipeline qui charge des données delta basées sur des informations de **capture des changements de données (CDC)** de la base de données source Azure SQL Managed Instance vers un stockage Blob Azure.  

Dans ce tutoriel, vous allez effectuer les étapes suivantes :

> [!div class="checklist"]
> * Préparer le magasin de données source
> * Créer une fabrique de données.
> * créez des services liés.
> * Création de jeux de données source et récepteur.
> * Créer, déboguer et exécuter le pipeline pour rechercher les données modifiées
> * Modifier des données dans la table source
> * Créer, exécuter et superviser le pipeline de copie incrémentielle complet

## <a name="overview"></a>Vue d’ensemble
La technologie de capture des changements de données prise en charge par les magasins de données que sont notamment Azure SQL Managed Instance (MI) et SQL Server peut être utilisée pour identifier les données modifiées.  Ce tutoriel explique comment utiliser Azure Data Factory avec la technologie de capture des changements de données pour charger de façon incrémentielle des données delta d’Azure SQL Managed Instance vers un Stockage Blob Azure.  Pour des informations plus concrètes sur la technologie de capture des changements de données SQL, consultez [Capture des changements de données dans SQL Server](/sql/relational-databases/track-changes/about-change-data-capture-sql-server).

## <a name="end-to-end-workflow"></a>Workflow de bout en bout
Voici les étapes de bout en bout du workflow type permettant de charger de façon incrémentielle des données en utilisant la technologie de capture des changements de données.

> [!NOTE]
> Azure SQL MI et SQL Server prennent tous deux en charge la technologie de capture des changements de données. Ce tutoriel utilise Azure SQL Managed Instance comme magasin de données source. Vous pouvez également utiliser un SQL Server local.

## <a name="high-level-solution"></a>Solution générale
Dans ce tutoriel, vous allez créer un pipeline qui effectue les opérations suivantes :  

   1. Créer une **activité de recherche** pour compter le nombre d’enregistrements modifiés dans la table CDC SQL Database et passer ce nombre à une activité Condition IF.
   2. Créez une **Condition If** pour vérifier s’il existe des enregistrements modifiés et, le cas échéant, appeler l’activité de copie.
   3. Créez une **activité de copie** pour copier les données insérées/mises à jour/supprimées entre la table CDC dans le Stockage Blob Azure.

Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis
* **Azure SQL Database Managed Instance** . Vous utilisez la base de données comme magasin de données **sources** . Si vous n’avez pas d’instance managée Azure SQL Database, consultez l’article [Créer une instance managée Azure SQL Database](../azure-sql/managed-instance/instance-create-quickstart.md) pour savoir comme en créer une.
* **Compte Stockage Azure** . Vous utilisez le stockage Blob comme magasin de données **récepteur** . Si vous n’avez pas de compte de stockage Azure, consultez l’article [Créer un compte de stockage](../storage/common/storage-account-create.md) pour découvrir comment en créer un. Créez un conteneur sous le nom **raw** . 

### <a name="create-a-data-source-table-in-azure-sql-database"></a>Créer une table de source de données dans Azure SQL Database

1. Lancez **SQL Server Management Studio** et connectez-vous à votre serveur d’instances managées Azure SQL.
2. Dans l’ **Explorateur de serveurs** , cliquez avec le bouton droit sur votre **base de données** et choisissez **Nouvelle requête** .
3. Exécutez la commande SQL suivante sur votre base de données d’instances managées Azure SQL pour créer une table sous le nom `customers` en tant que magasin de source de données.  

    ```sql
    create table customers 
    (
    customer_id int, 
    first_name varchar(50), 
    last_name varchar(50), 
    email varchar(100), 
    city varchar(50), CONSTRAINT "PK_Customers" PRIMARY KEY CLUSTERED ("customer_id") 
     );
    ```
4. Activez le mécanisme **Capture des changements de données** sur votre base de données et la table source (customers) en exécutant la requête SQL suivante :

    > [!NOTE]
    > - Remplacez le &lt;nom de votre schéma source&gt; par le schéma de votre instance Azure SQL MI qui contient la table customers.
    > - La capture des changements de données n’agit pas dans le cadre des transactions qui modifient la table suivie. En revanche, les opérations d’insertion, de mise à jour et de suppression sont écrites dans le journal des transactions. La taille des données déposées dans les tables de modifications augmentera de manière ingérable si vous n'effectuez pas un nettoyage périodique et systématique de ces données. Pour plus d’informations, consultez [Activer la capture des changements de données pour une base de données](/sql/relational-databases/track-changes/enable-and-disable-change-data-capture-sql-server?enable-change-data-capture-for-a-database=&view=sql-server-ver15)

    ```sql
    EXEC sys.sp_cdc_enable_db 
    
    EXEC sys.sp_cdc_enable_table
    @source_schema = 'dbo',
    @source_name = 'customers', 
    @role_name = 'null',
    @supports_net_changes = 1
    ```
5. Insérez des données dans la table customers en exécutant la commande suivante :

    ```sql
     insert into customers 
        (customer_id, first_name, last_name, email, city) 
     values 
        (1, 'Chevy', 'Leward', 'cleward0@mapy.cz', 'Reading'),
        (2, 'Sayre', 'Ateggart', 'sateggart1@nih.gov', 'Portsmouth'),
        (3, 'Nathalia', 'Seckom', 'nseckom2@blogger.com', 'Portsmouth');
    ```

    > [!NOTE]
    > Les modifications historiques apportées à la table ne sont pas capturées tant que la capture des changements de données n’est pas activée.

## <a name="create-a-data-factory"></a>Créer une fabrique de données

1. Lancez le navigateur web **Microsoft Edge** ou **Google Chrome** . L’interface utilisateur de Data Factory n’est actuellement prise en charge que par les navigateurs web Microsoft Edge et Google Chrome.
1. Dans le menu de gauche, sélectionnez **Créer une ressource** > **Données + Analytique** > **Data Factory**  :

   ![Sélection Data Factory dans le volet « Nouveau »](./media/tutorial-incremental-copy-change-data-capture-feature-portal/new-azure-data-factory-menu.png)

2. Dans la page **Nouvelle fabrique de données** , entrez **ADFTutorialDataFactory** comme **nom** .

     ![Page Nouvelle fabrique de données](./media/tutorial-incremental-copy-change-data-capture-feature-portal/new-azure-data-factory.png)

   Le nom de la fabrique de données Azure doit être un nom **global unique** . Si l’erreur suivante s’affiche, changez le nom de la fabrique de données (par exemple, votrenomADFTutorialDataFactory), puis tentez de la recréer. Consultez l’article [Data Factory - Règles d’affectation des noms](naming-rules.md) pour savoir comment nommer les artefacts Data Factory.

    *Le nom de fabrique de données « ADFTutorialDataFactory » n’est pas disponible* .
3. Sélectionnez **V2** pour la **version** .
4. Sélectionnez l’ **abonnement** Azure dans lequel vous voulez créer la fabrique de données.
5. Pour le **groupe de ressources** , effectuez l’une des opérations suivantes :

   1. Sélectionnez **Utiliser l’existant** , puis sélectionnez un groupe de ressources existant dans la liste déroulante.
   2. Sélectionnez **Créer** , puis entrez le nom d’un groupe de ressources.   
         
    Pour plus d’informations sur les groupes de ressources, consultez [Utilisation des groupes de ressources pour gérer vos ressources Azure](../azure-resource-manager/management/overview.md).  
5. Sélectionnez **l’emplacement** de la fabrique de données. Seuls les emplacements pris en charge sont affichés dans la liste déroulante. Les magasins de données (Stockage Azure, Azure SQL Database, etc.) et les services de calcul (HDInsight, etc.) utilisés par la fabrique de données peuvent être proposés dans d’autres régions.
6. Désélectionnez **Activer GIT** .     
7. Cliquez sur **Créer** .
8. Une fois le déploiement terminé, cliquez sur **Accéder à la ressource**

   ![Capture d’écran d’un message indiquant que votre déploiement est terminé avec une option pour accéder à la ressource.](./media/tutorial-incremental-copy-change-data-capture-feature-portal/data-factory-deploy-complete.png)
9. Une fois la création terminée, la page **Data Factory** s’affiche comme sur l’image.

   ![Capture d’écran montrant la fabrique de données que vous avez déployée.](./media/tutorial-incremental-copy-change-data-capture-feature-portal/data-factory-home-page.png)
10. Cliquez sur la vignette **Créer et surveiller** pour lancer l’interface utilisateur d’Azure Data Factory dans un onglet séparé.
11. Dans la page **Prise en main** , basculez vers l’onglet **Modifier** dans le volet gauche comme illustré dans l’image suivante :

    ![Bouton Créer un pipeline](./media/tutorial-incremental-copy-change-data-capture-feature-portal/get-started-page.png)

## <a name="create-linked-services"></a>Créez des services liés
Vous allez créer des services liés dans une fabrique de données pour lier vos magasins de données et vos services de calcul à la fabrique de données. Dans cette section, vous allez créer des services liés à votre compte Stockage Azure et à Azure SQL MI.

### <a name="create-azure-storage-linked-service"></a>Créer un service lié Stockage Azure.
Dans cette étape, vous liez votre compte Stockage Azure à la fabrique de données.

1. Cliquez sur **Connexions** , puis sur **+ Nouveau** .

   ![Bouton Nouvelle connexion](./media/tutorial-incremental-copy-change-data-capture-feature-portal/new-connection-button-storage.png)
2. Dans la fenêtre **Nouveau service lié** , sélectionnez **Stockage Blob Azure** , puis cliquez sur **Continuer** .

   ![Sélectionner le stockage Blob Azure](./media/tutorial-incremental-copy-change-data-capture-feature-portal/select-azure-storage.png)
3. Dans la fenêtre **Nouveau service lié** , procédez comme suit :

   1. Entrez **AzureStorageLinkedService** pour **Nom** .
   2. Sélectionnez votre compte de stockage Azure comme **Nom du compte de stockage** .
   3. Cliquez sur **Enregistrer** .

   ![Paramètres du compte de stockage Azure](./media/tutorial-incremental-copy-change-data-capture-feature-portal/azure-storage-linked-service-settings.png)


### <a name="create-azure-sql-mi-database-linked-service"></a>Créez le service lié Base de données Azure SQL MI.
Dans cette étape, vous allez lier votre base de données Azure SQL MI à la fabrique de données.

> [!NOTE]
> Si vous utilisez SQL MI, consultez [ce document](./connector-azure-sql-managed-instance.md#prerequisites) pour obtenir des informations sur l’accès via un point de terminaison public ou privé. Si vous utilisez un point de terminaison privé, vous devez exécuter ce pipeline en utilisant un runtime d’intégration auto-hébergé. Il en va de même si vous exécutez SQL Server en local, dans un scénario de machines virtuelles ou de réseau virtuel.

1. Cliquez sur **Connexions** , puis sur **+ Nouveau** .
2. Dans la fenêtre **Nouveau service lié** , sélectionnez **Azure SQL Database Managed Instance** , puis cliquez sur **Continuer** .
3. Dans la fenêtre **Nouveau service lié** , procédez comme suit :

   1. Entrez **AzureSqlMI1** dans le champ **Nom** .
   2. Sélectionnez votre serveur SQL dans le champ **Nom du serveur** .
   4. Sélectionnez votre base de données SQL dans le champ **Nom de la base de données** .
   5. Entrez le nom de l’utilisateur pour le champ **Nom d’utilisateur** .
   6. Entrez le mot de passe de l’utilisateur pour le champ **Mot de passe** .
   7. Cliquez sur **Tester la connexion** pour tester la connexion.
   8. Cliquez sur **Enregistrer** pour enregistrer le service lié.

   ![Paramètres du service lié Base de données Azure SQL MI](./media/tutorial-incremental-copy-change-data-capture-feature-portal/azure-sql-managed-instance-database-linked-service-settings.png)

## <a name="create-datasets"></a>Créez les jeux de données
Dans cette étape, vous allez créer des jeux de données pour représenter la source de données et la destination des données.

### <a name="create-a-dataset-to-represent-source-data"></a>Créer un jeu de données pour représenter les données sources
Dans cette étape, vous créez un jeu de données pour représenter les données source.

1. Dans l’arborescence, cliquez sur **+ (plus)** , puis sur **Jeu de données** .

   ![Menu Nouveau jeu de données](./media/tutorial-incremental-copy-change-data-capture-feature-portal/new-dataset-menu.png)
2. Sélectionnez **Azure SQL Database Managed Instance** , puis cliquez sur **Continuer** .

   ![Type de jeu de données source - Azure SQL Database](./media/tutorial-incremental-copy-change-data-capture-feature-portal/select-azure-sql-database.png)
   
3. Sous l’onglet **Définir les propriétés** , définissez le nom du jeu de données et les informations de connexion :
 
   1. Sélectionnez **AzureSqlMI1** pour **Service lié** .
   2. Sélectionnez **[dbo].[dbo_customers_CT]** pour **Nom de la table** .  Remarque : Cette table a été créée automatiquement au moment où CDC a été activé pour la table customers. Les données modifiées ne sont jamais interrogées directement à partir de cette table, mais elles sont extraites via les [fonctions CDC](/sql/relational-databases/system-functions/change-data-capture-functions-transact-sql?view=sql-server-ver15).

   ![Connexion source](./media/tutorial-incremental-copy-change-data-capture-feature-portal/source-dataset-configuration.png)

### <a name="create-a-dataset-to-represent-data-copied-to-sink-data-store"></a>Créez un jeu de données pour représenter les données copiées dans le magasin de données récepteur.
Dans cette étape, vous créez un jeu de données pour représenter les données copiées à partir du magasin de données source. Vous avez créé le conteneur de lac de données dans votre Stockage Blob Azure au titre des prérequis. Créez le conteneur s’il n’existe pas (ou) attribuez-lui le nom d’un conteneur existant. Dans ce tutoriel, le nom du fichier de sortie est généré dynamiquement en utilisant l’heure de déclenchement, qui sera configurée ultérieurement.

1. Dans l’arborescence, cliquez sur **+ (plus)** , puis sur **Jeu de données** .

   ![Menu Nouveau jeu de données](./media/tutorial-incremental-copy-change-data-capture-feature-portal/new-dataset-menu.png)
2. Sélectionnez **Stockage Blob Azure** , puis cliquez sur **Continuer** .

   ![Type de jeu de données récepteur - Stockage Blob Azure](./media/tutorial-incremental-copy-change-data-capture-feature-portal/sink-dataset-type.png)
3. Sélectionnez **DelimitedText** , puis cliquez sur **Continuer** .

   ![Format du jeu de données récepteur – DelimitedText](./media/tutorial-incremental-copy-change-data-capture-feature-portal/sink-dataset-format.png)
4. Sous l’onglet **Définir les propriétés** , définissez le nom du jeu de données et les informations de connexion :

   1. Sélectionnez **AzureStorageLinkedService** pour **Service lié** .
   2. Entrez **raw** pour la partie **conteneur** du **chemin du fichier** .
   3. Activez **Utiliser la première ligne comme en-tête**
   4. Cliquez sur **OK**

   ![Jeu de données récepteur - connexion](./media/tutorial-incremental-copy-change-data-capture-feature-portal/sink-dataset-configuration.png)

## <a name="create-a-pipeline-to-copy-the-changed-data"></a>Créer un pipeline pour copier les données modifiées
Dans cette étape, vous allez créer un pipeline, qui vérifie d’abord le nombre d’enregistrements modifiés dans la table de modifications à l’aide d’une **activité de recherche** . Une activité Condition IF vérifie si le nombre d’enregistrements modifiés est supérieur à zéro et exécute une **activité de copie** pour copier les données insérées/mises à jour/supprimées d’Azure SQL Database vers le Stockage Blob Azure. Enfin, un déclencheur de fenêtre bascule est configuré et les heures de début et de fin sont transmises aux activités sous forme de paramètres de fenêtre de début et de fin. 

1. Dans l’interface utilisateur de Data Factory, basculez vers l’onglet **Modifier** . Cliquez sur **+ (plus)** dans le volet gauche, puis cliquez sur **Pipeline** .

    ![Menu Nouveau pipeline](./media/tutorial-incremental-copy-change-data-capture-feature-portal/new-pipeline-menu.png)
2. Vous voyez un nouvel onglet pour configurer le pipeline. Vous voyez également le pipeline dans l’arborescence. Dans la fenêtre **Propriétés** , renommez le pipeline en **IncrementalCopyPipeline** .

    ![Nom du pipeline](./media/tutorial-incremental-copy-change-data-capture-feature-portal/incremental-copy-pipeline-name.png)
3. Développez **Général** dans la boîte à outils **Activités** , puis faites glisser et déposez une activité **Recherche** sur la surface du concepteur de pipeline. Nommez l’activité **GetChangeCount** . Cette activité obtient le nombre d’enregistrements présents dans la table de modifications pour une fenêtre de temps donnée.

    ![Activité de recherche - nom](./media/tutorial-incremental-copy-change-data-capture-feature-portal/first-lookup-activity-name.png)
4. Accédez à **Paramètres** dans la fenêtre **Propriétés**  :
   1. Spécifiez le nom du jeu de données SQL MI dans le champ **Jeu de données source** .
   2. Sélectionnez l’option Requête et entrez ce qui suit dans la zone de requête :
    ```sql
    DECLARE  @from_lsn binary(10), @to_lsn binary(10);  
    SET @from_lsn =sys.fn_cdc_get_min_lsn('dbo_customers');  
    SET @to_lsn = sys.fn_cdc_map_time_to_lsn('largest less than or equal',  GETDATE());
    SELECT count(1) changecount FROM cdc.fn_cdc_get_all_changes_dbo_customers(@from_lsn, @to_lsn, 'all')
    ```
   3. Activez **Utiliser la première ligne comme en-tête**

    ![Activité de recherche - paramètres](./media/tutorial-incremental-copy-change-data-capture-feature-portal/first-lookup-activity-settings.png)
5. Cliquez sur le bouton **Aperçu des données** pour vérifier la validité de la sortie obtenue par l’activité de recherche

    ![Activité de recherche – aperçu](./media/tutorial-incremental-copy-change-data-capture-feature-portal/first-lookup-activity-preview.png)
6. Développez **Itération et conditions** dans la boîte à outils **Activités** , puis glissez-déplacez l’activité **Condition If** vers la surface du concepteur de pipeline. Nommez l’activité **HasChangedRows** . 

    ![Activité Condition If - nom](./media/tutorial-incremental-copy-change-data-capture-feature-portal/if-condition-activity-name.png)
7. Accédez à **Activités** dans la fenêtre **Propriétés**  :

   1. Entrez l’ **expression** suivante
   
    ```adf
    @greater(int(activity('GetChangeCount').output.firstRow.changecount),0)
    ```

   2. Cliquez sur l’icône de crayon pour modifier la condition True.

   ![Activité Condition If - paramètres](./media/tutorial-incremental-copy-change-data-capture-feature-portal/if-condition-activity-setting.png)

   3. Développez **Général** dans la boîte à outils **Activités** , puis glissez-déposez une activité **Attendre** vers la surface du concepteur de pipeline. Il s’agit d’une activité temporaire destinée à déboguer la condition If et sera modifiée par la suite dans le tutoriel. 

   ![Condition If True - attendre](./media/tutorial-incremental-copy-change-data-capture-feature-portal/if-condition-activity-wait.png)

   4. Cliquez sur la barre de navigation IncrementalCopyPipeline pour revenir au pipeline principal.

8. Exécutez le pipeline en mode **Débogage** pour vérifier que le pipeline s’exécute correctement. 

   ![Pipeline - débogage](./media/tutorial-incremental-copy-change-data-capture-feature-portal/incremental-copy-pipeline-debug.png)
9. Ensuite, revenez à l’étape de la condition True et supprimez l’activité **Attendre** . Dans la boîte à outils **Activités** , développez **Déplacer et transformer** et glissez-déposez l’activité **Copier** vers la surface du concepteur de pipeline. Définissez le nom de l’activité sur **IncrementalCopyActivity** . 

   ![Activité de copie - nom](./media/tutorial-incremental-copy-change-data-capture-feature-portal/copy-source-name.png)
10. Basculez vers l’onglet **Source** dans la fenêtre **Propriétés** , et procédez comme suit :

   1. Spécifiez le nom du jeu de données SQL MI dans le champ **Jeu de données source** . 
   2. Sélectionnez **Requête** pour **Utiliser la requête** .
   3. Entrez ce qui suit pour **Requête** .

      ```sql
      DECLARE @from_lsn binary(10), @to_lsn binary(10); 
      SET @from_lsn =sys.fn_cdc_get_min_lsn('dbo_customers'); 
      SET @to_lsn = sys.fn_cdc_map_time_to_lsn('largest less than or equal', GETDATE());
      SELECT * FROM cdc.fn_cdc_get_all_changes_dbo_customers(@from_lsn, @to_lsn, 'all')
      ```

   ![Activité de copie - paramètres de la source](./media/tutorial-incremental-copy-change-data-capture-feature-portal/copy-source-settings.png)

11. Cliquez sur Aperçu pour vérifier que la requête retourne correctement les lignes modifiées.

    ![Capture d’écran montrant un aperçu pour vérifier la requête.](./media/tutorial-incremental-copy-change-data-capture-feature-portal/copy-source-preview.png)
12. Basculez sous l’onglet **Récepteur** et spécifiez le jeu de données Stockage Azure dans le champ **Jeu de données récepteur** .

    ![Capture d’écran montrant l’onglet Récepteur.](./media/tutorial-incremental-copy-change-data-capture-feature-portal/copy-sink-settings.png)
13. Cliquez de nouveau sur le canevas du pipeline principal et connectez l’activité **Recherche** à l’activité **Condition If** une après l’autre. Faites glisser le bouton **vert** attaché à l’activité **Recherche** vers l’activité **Condition If** .

    ![Connecter des activités de recherche et de copie](./media/tutorial-incremental-copy-change-data-capture-feature-portal/connect-lookup-if.png)
14. Cliquez sur **Valider** dans la barre d’outils. Vérifiez qu’il n’y a aucune erreur de validation. Fermez la fenêtre **Rapport de validation de pipeline** en cliquant sur **>>** .

    ![Bouton de validation](./media/tutorial-incremental-copy-change-data-capture-feature-portal/validate-button.png)
15. Cliquez sur Déboguer pour tester le pipeline et vérifier qu’un fichier est généré dans l’emplacement de stockage.

    ![Débogage du pipeline incrémentiel-2](./media/tutorial-incremental-copy-change-data-capture-feature-portal/incremental-copy-pipeline-debug-2.png)
16. Publiez des entités (services liés, jeux de données et pipelines) sur le service Data Factory en cliquant sur le bouton **Publier tout** . Patientez jusqu’à ce que le message **Publication réussie** s’affiche.

    ![Bouton Publier](./media/tutorial-incremental-copy-change-data-capture-feature-portal/publish-button-2.png)    

### <a name="configure-the-tumbling-window-trigger-and-cdc-window-parameters"></a>Configurer le déclencheur de fenêtre bascule et les paramètres de fenêtre CDC 
Dans cette étape, vous allez créer un déclencheur de fenêtre bascule pour exécuter la tâche selon une planification fréquente. Vous allez utiliser les variables système WindowStart et WindowEnd du déclencheur de fenêtre bascule et les transmettre sous forme de paramètres au pipeline à utiliser dans la requête CDC.

1. Accédez à l’onglet **Paramètres** du pipeline **IncrementalCopyPipeline** puis, à l’aide du bouton **+ Nouveau** , ajoutez deux paramètres ( **triggerStartTime** et **triggerEndTime** ) au pipeline, qui représentent l’heure de début et l’heure de fin de la fenêtre bascule. Pour les besoins du débogage, ajoutez les valeurs par défaut au format **AAAA-MM-JJ HH24:MI:SS.FFF** , mais vérifiez que l’heure triggerStartTime n’est pas antérieure à l’heure d’activation de CDC sur la table. Si c’est le cas, cela générera une erreur.

    ![Menu Déclencher maintenant](./media/tutorial-incremental-copy-change-data-capture-feature-portal/incremental-copy-pipeline-parameters.png)
2. Cliquez sur l’onglet Paramètres de l’activité **Recherche** et configurez la requête pour qu’elle utilise les paramètres de début et de fin. Copiez ce qui suit dans la requête :
    ```sql
    @concat('DECLARE @begin_time datetime, @end_time datetime, @from_lsn binary(10), @to_lsn binary(10); 
    SET @begin_time = ''',pipeline().parameters.triggerStartTime,''';
    SET @end_time = ''',pipeline().parameters.triggerEndTime,''';
    SET @from_lsn = sys.fn_cdc_map_time_to_lsn(''smallest greater than or equal'', @begin_time);
    SET @to_lsn = sys.fn_cdc_map_time_to_lsn(''largest less than'', @end_time);
    SELECT count(1) changecount FROM cdc.fn_cdc_get_all_changes_dbo_customers(@from_lsn, @to_lsn, ''all'')')
    ```

3. Accédez à l’activité **Copier** dans le cas True de l’activité **Condition If** et cliquez sur l’onglet **Source** . Copiez ce qui suit dans la requête :
    ```sql
    @concat('DECLARE @begin_time datetime, @end_time datetime, @from_lsn binary(10), @to_lsn binary(10); 
    SET @begin_time = ''',pipeline().parameters.triggerStartTime,''';
    SET @end_time = ''',pipeline().parameters.triggerEndTime,''';
    SET @from_lsn = sys.fn_cdc_map_time_to_lsn(''smallest greater than or equal'', @begin_time);
    SET @to_lsn = sys.fn_cdc_map_time_to_lsn(''largest less than'', @end_time);
    SELECT * FROM cdc.fn_cdc_get_all_changes_dbo_customers(@from_lsn, @to_lsn, ''all'')')
    ```
4. Cliquez sur l’onglet **Récepteur** de l’activité **Copier** , puis cliquez sur **Ouvrir** pour modifier les propriétés du jeu de données. Cliquez sur l’onglet **Paramètres** et ajoutez un nouveau paramètre appelé **triggerStart**    

    ![Capture d’écran montrant l’ajout d’un nouveau paramètre à l’onglet Paramètres.](./media/tutorial-incremental-copy-change-data-capture-feature-portal/sink-dataset-configuration-2.png)
5. Ensuite, configurez les propriétés du jeu de données pour stocker les données dans un sous-répertoire **customers/incremental** avec des partitions basées sur la date.
   1. Cliquez sur l’onglet **Connexion** des propriétés du jeu de données et ajoutez du contenu dynamique pour les sections **Répertoire** et **Fichier** . 
   2. Entrez l’expression suivante dans la section **Répertoire** en cliquant sur le lien de contenu dynamique en dessous de la zone de texte :
    
    ```sql
    @concat('customers/incremental/',formatDateTime(dataset().triggerStart,'yyyy/MM/dd'))
    ```
   3. Entrez l’expression suivante dans la section **Fichier** . Cela aura pour effet de créer des noms de fichiers basés sur la date et l’heure de début du déclencheur, avec l’extension CSV pour suffixe :
    
    ```sql
    @concat(formatDateTime(dataset().triggerStart,'yyyyMMddHHmmssfff'),'.csv')
    ```
    ![Configuration du jeu de données récepteur-3](./media/tutorial-incremental-copy-change-data-capture-feature-portal/sink-dataset-configuration-3.png)

   4. Revenez aux paramètres du **Récepteur**  dans l’activité **Copier** en cliquant sur l’onglet **IncrementalCopyPipeline** . 
   5. Développez les propriétés du jeu de données et entrez du contenu dynamique dans la valeur du paramètre triggerStart avec l’expression suivante :
     ```sql
     @pipeline().parameters.triggerStartTime
     ```
    ![Configuration du jeu de données récepteur-4](./media/tutorial-incremental-copy-change-data-capture-feature-portal/sink-dataset-configuration-4.png)

6. Cliquez sur Déboguer pour tester le pipeline et vérifier que la structure de dossiers et le fichier de sortie sont générés comme prévu. Téléchargez et ouvrez le fichier pour vérifier son contenu. 

    ![Débogage de la copie incrémentielle-3](./media/tutorial-incremental-copy-change-data-capture-feature-portal/incremental-copy-pipeline-debug-3.png)
7. Vérifiez que les paramètres sont injectés dans la requête en examinant les paramètres d’entrée de l’exécution du pipeline.

    ![Débogage de la copie incrémentielle-4](./media/tutorial-incremental-copy-change-data-capture-feature-portal/incremental-copy-pipeline-debug-4.png)
8. Publiez des entités (services liés, jeux de données et pipelines) sur le service Data Factory en cliquant sur le bouton **Publier tout** . Patientez jusqu’à ce que le message **Publication réussie** s’affiche.
9. Enfin, configurez un déclencheur de fenêtre bascule pour exécuter le pipeline à intervalles réguliers et définir les paramètres d’heure de début et de fin. 
   1. Cliquez sur le bouton **Ajouter un déclencheur** , puis sélectionnez **Nouveau/Modifier**

   ![Ajouter un déclencheur](./media/tutorial-incremental-copy-change-data-capture-feature-portal/add-trigger.png)

   2. Entrez un nom de déclencheur et spécifiez une heure de début, qui correspond à l’heure de fin de la fenêtre de débogage ci-dessus.

   ![Déclencheur de fenêtre bascule](./media/tutorial-incremental-copy-change-data-capture-feature-portal/tumbling-window-trigger.png)

   3. Dans l’écran suivant, spécifiez les valeurs ci-dessous pour les paramètres de début et de fin, respectivement.
    ```sql
    @formatDateTime(trigger().outputs.windowStartTime,'yyyy-MM-dd HH:mm:ss.fff')
    @formatDateTime(trigger().outputs.windowEndTime,'yyyy-MM-dd HH:mm:ss.fff')
    ```

   ![Déclencheur de fenêtre bascule-2](./media/tutorial-incremental-copy-change-data-capture-feature-portal/tumbling-window-trigger-2.png)

> [!NOTE]
> Notez que le déclencheur s’exécute une fois qu’il a été publié. Par ailleurs, le comportement attendu de la fenêtre bascule est d’exécuter tous les intervalles historiques de la date de début jusqu’au moment présent. Vous trouverez des informations complémentaires sur les déclencheurs de fenêtre bascule [ici](./how-to-create-tumbling-window-trigger.md). 
  
10. L’utilisation de **SQL Server Management Studio** apporte des modifications supplémentaires à la table customer en exécutant le code SQL suivant :
    ```sql
    insert into customers (customer_id, first_name, last_name, email, city) values (4, 'Farlie', 'Hadigate', 'fhadigate3@zdnet.com', 'Reading');
    insert into customers (customer_id, first_name, last_name, email, city) values (5, 'Anet', 'MacColm', 'amaccolm4@yellowbook.com', 'Portsmouth');
    insert into customers (customer_id, first_name, last_name, email, city) values (6, 'Elonore', 'Bearham', 'ebearham5@ebay.co.uk', 'Portsmouth');
    update customers set first_name='Elon' where customer_id=6;
    delete from customers where customer_id=5;
    ```
11. Cliquez sur le bouton **Publier tout** . Patientez jusqu’à ce que le message **Publication réussie** s’affiche.  
12. Au bout de quelques minutes, le pipeline est déclenché et un nouveau fichier est chargé dans Stockage Azure


### <a name="monitor-the-incremental-copy-pipeline"></a>Surveiller le pipeline de copie incrémentielle
1. Cliquez sur l’onglet **Surveiller** sur la gauche. Vous voyez l’exécution du pipeline dans la liste et son état. Pour actualiser la liste, cliquez sur **Actualiser** . Placez le curseur près du nom du pipeline pour accéder à l’action Réexécuter et au rapport de consommation.

    ![Exécutions de pipeline](./media/tutorial-incremental-copy-change-data-capture-feature-portal/copy-pipeline-runs.png)
2. Pour afficher les exécutions d’activité associées à l’exécution de pipelines, cliquez sur le nom du pipeline. Si des changements de données ont été détectés, trois activités sont présentes, dont celle de copie. Sinon, il n’y a que deux entrées dans la liste. Pour revenir à la vue des exécutions de pipelines, cliquez sur le lien **Tous les pipelines** en haut.

    ![Exécutions d’activités](./media/tutorial-incremental-copy-change-data-capture-feature-portal/copy-activity-runs.png)


### <a name="review-the-results"></a>Passer en revue les résultats.
Vous voyez le second fichier dans le dossier `customers/incremental/YYYY/MM/DD` du conteneur `raw`.

![Fichier de sortie de la copie incrémentielle](media/tutorial-incremental-copy-change-data-capture-feature-portal/incremental-copy-pipeline-run.png)
 

## <a name="next-steps"></a>Étapes suivantes
Passez au tutoriel suivant pour en savoir plus sur la copie des fichiers nouveaux et modifiés uniquement en fonction de leur LastModifiedDate :

> [!div class="nextstepaction"]
>[Copier les nouveaux fichiers par lastmodifieddate](tutorial-incremental-copy-lastmodified-copy-data-tool.md)