---
title: Synchroniser des données à partir d’Azure SQL Edge à l’aide d’Azure Data Factory
description: Découvrez comment synchroniser des données entre Azure SQL Edge et Stockage Blob Azure.
keywords: SQL Edge, synchroniser des données à partir de SQL Edge, fabrique de données SQL Edge
services: sql-edge
ms.service: sql-edge
ms.topic: tutorial
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 05/19/2020
ms.openlocfilehash: b83201ae864d1f1eb9124af5268360bb1748f6c8
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97507606"
---
# <a name="tutorial-sync-data-from-sql-edge-to-azure-blob-storage-by-using-azure-data-factory"></a>Tutoriel : Synchroniser des données de SQL Edge sur le Stockage Blob Azure à l’aide d’Azure Data Factory

Ce tutoriel explique comment utiliser Azure Data Factory pour synchroniser des données de façon incrémentielle d’une table appartenant à une instance Azure SQL Edge vers le Stockage Blob Azure.

## <a name="before-you-begin"></a>Avant de commencer

Si vous n’avez pas encore créé de base de données ou de table dans votre déploiement Azure SQL Edge, créez-en une à l’aide de l’une des méthodes suivantes :

* Utilisez [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms/) ou [Azure Data Studio](/sql/azure-data-studio/download/) pour vous connecter à SQL Edge. Exécutez un script SQL pour créer la base de données et la table.
* Créez une base de données et une table à l’aide de [SQLCMD](/sql/tools/sqlcmd-utility/) en vous connectant directement au module SQL Edge. Pour plus d’informations, consultez [Se connecter au moteur de base de données avec sqlcmd](/sql/ssms/scripting/sqlcmd-connect-to-the-database-engine/).
* Utilisez SQLPackage.exe pour déployer un fichier de package DAC sur le conteneur SQL Edge. Vous pouvez automatiser ce processus en spécifiant l’URI du fichier SqlPackage dans le cadre de la configuration des propriétés souhaitées du module. Vous pouvez également utiliser directement l’outil client SqlPackage.exe pour déployer un package DAC sur SQL Edge.

    Pour plus d’informations sur le téléchargement de SqlPackage.exe, consultez [Télécharger et installer sqlpackage](/sql/tools/sqlpackage-download/). Voici quelques exemples de commandes pour SqlPackage.exe. Pour plus d’informations, consultez la documentation de SqlPackage.exe.

    **Créer un package DAC**

    ```cmd
    sqlpackage /Action:Extract /SourceConnectionString:"Data Source=<Server_Name>,<port>;Initial Catalog=<DB_name>;User ID=<user>;Password=<password>" /TargetFile:<dacpac_file_name>
    ```

    **Appliquer un package DAC**

    ```cmd
    sqlpackage /Action:Publish /Sourcefile:<dacpac_file_name> /TargetServerName:<Server_Name>,<port> /TargetDatabaseName:<DB_Name> /TargetUser:<user> /TargetPassword:<password>
    ```

## <a name="create-a-sql-table-and-procedure-to-store-and-update-the-watermark-levels"></a>Créer une table et une procédure SQL pour stocker et mettre à jour les niveaux de filigrane

Une table de filigranes est utilisée pour stocker la date et l’heure auxquelles les données ont été synchronisées pour la dernière fois avec le stockage Azure. Une procédure stockée Transact-SQL (T-SQL) est utilisée pour mettre à jour la table de filigranes après chaque synchronisation.

Exécutez ces commandes sur l’instance SQL Edge :

```sql
    Create table [dbo].[watermarktable]
    (
    TableName varchar(255),
    WatermarkValue datetime,
    )
    GO

    CREATE PROCEDURE usp_write_watermark @timestamp datetime, @TableName varchar(50)  
    AS  
    BEGIN
    
    UPDATE [dbo].[watermarktable]
    SET [WatermarkValue] = @timestamp
    WHERE [TableName] = @TableName

    END
    Go
```

## <a name="create-a-data-factory-pipeline"></a>Créer un pipeline Data Factory

Dans cette section, vous allez créer un pipeline Azure Data Factory pour synchroniser des données d’une table dans Azure SQL Edge vers le Stockage Blob Azure.

### <a name="create-a-data-factory-by-using-the-data-factory-ui"></a>Créer une fabrique de données avec l’interface utilisateur Data Factory

Créez une fabrique de données en suivant les instructions de ce [tutoriel](../data-factory/quickstart-create-data-factory-portal.md#create-a-data-factory).

### <a name="create-a-data-factory-pipeline"></a>Créer un pipeline Data Factory

1. Dans la page **Prise en main** de l’interface utilisateur Data Factory, sélectionnez **Créer un pipeline**.

    ![Créer un pipeline Data Factory](media/tutorial-sync-data-factory/data-factory-get-started.png)

2. Dans la page **Général** de la fenêtre **Propriétés** du pipeline, entrez le nom **PeriodicSync**.

3. Ajoutez l’activité Recherche pour obtenir l’ancienne valeur de filigrane. Dans le volet **Activités**, développez **Général** et faites glisser l’activité **Recherche** vers la surface du concepteur de pipeline. Remplacez le nom de l’activité par **OldWatermark**.

    ![Ajouter la recherche de l’ancien filigrane](media/tutorial-sync-data-factory/create-old-watermark-lookup.png)

4. Basculez vers l’onglet **Paramètres** et sélectionnez **Nouveau** pour **Jeu de données source**. Vous créez maintenant un jeu de données pour représenter les données de la table de filigranes. Cette table contient l’ancien filigrane utilisé dans l’opération de copie précédente.

5. Dans la fenêtre **Nouveau jeu de données**, sélectionnez **Azure SQL Server**, puis **Continuer**.  

6. Dans la fenêtre **Définir les propriétés** du jeu de données, sous **Nom**, entrez **WatermarkDataset**.

7. Pour **Service lié**, sélectionnez **Nouveau**, puis effectuez ces étapes :

    1. Sous **Nom**, entrez **SQLDBEdgeLinkedService**.

    2. Sous **Nom du serveur**, entrez les détails de votre serveur SQL Edge.

    3. Sélectionnez le **Nom de la base de données** dans la liste déroulante.

    4. Entrez votre **nom d’utilisateur** et votre **mot de passe**.

    5. Pour tester la connexion à l’instance SQL Edge, sélectionnez **Tester la connexion**.

    6. Sélectionnez **Create** (Créer).

    ![Créer un service lié](media/tutorial-sync-data-factory/create-linked-service.png)

    7. Sélectionnez **OK**.

8. Sous l’onglet **Paramètres**, sélectionnez **Modifier**.

9. Sous l’onglet **Connexion**, sélectionnez **[dbo].[watermarktable]** pour la **Table**. Si vous souhaitez voir un aperçu des données de la table, sélectionnez **Aperçu des données**.

10. Basculez vers l’éditeur de pipeline en sélectionnant l’onglet Pipeline en haut ou en sélectionnant le nom du pipeline dans l’arborescence à gauche. Dans la fenêtre de propriétés de l’activité Recherche, vérifiez que **WatermarkDataset** est sélectionné dans la liste **Jeu de données source**.

11. Dans le volet **Activités**, développez **Général** et faites glisser une autre activité **Recherche** vers la surface du concepteur de pipeline. Définissez le nom sur **NewWatermark** sous l’onglet **Général** de la fenêtre de propriétés. Cette activité de recherche obtient la nouvelle valeur de filigrane dans la table qui contient les données sources pour la copier dans la destination.

12. Dans la fenêtre de propriétés de la deuxième activité Recherche, basculez vers l’onglet **Paramètres** et sélectionnez **Nouveau** pour créer un jeu de données qui pointe vers la table source contenant la nouvelle valeur de filigrane.

13. Dans la fenêtre **Nouveau jeu de données**, sélectionnez **Instance SQL Edge**, puis **Continuer**.

    1. Dans la fenêtre **Définir les propriétés**, sous **Nom**, entrez **SourceDataset**. Sous **Service lié**, sélectionnez **SQLDBEdgeLinkedService**.

    2. Sous **Table**, sélectionnez la table à synchroniser. Vous pouvez également spécifier une requête pour ce jeu de données, comme indiqué plus loin dans le tutoriel. La requête a la priorité sur la table spécifiée à cette étape.

    3. Sélectionnez **OK**.

14. Basculez vers l’éditeur de pipeline en sélectionnant l’onglet Pipeline en haut ou en sélectionnant le nom du pipeline dans l’arborescence à gauche. Dans la fenêtre de propriétés de l’activité de recherche, vérifiez que **SourceDataset** est sélectionné dans la liste **Jeu de données source**.

15. Sélectionnez **Requête** sous **Utiliser la requête**. Mettez à jour le nom de la table dans la requête suivante, puis entrez la requête. Vous sélectionnez uniquement la valeur maximale de `timestamp` dans la table. Veillez à sélectionner **Première ligne uniquement**.

    ```sql
    select MAX(timestamp) as NewWatermarkvalue from [TableName]
    ```

    ![sélectionner la requête](media/tutorial-sync-data-factory/select-query-data-factory.png)

16. Dans le volet **Activités**, développez **Déplacer et transformer**, puis faites glisser l’activité **Copie** du volet **Activités** vers la surface du concepteur. Définissez le nom de l’activité sur **IncrementalCopy**.

17. Connectez les deux activités Recherche à l’activité Copie en faisant glisser le bouton vert attaché aux activités Recherche vers l’activité Copie. Relâchez le bouton de la souris lorsque la couleur de bordure de l’activité Copie passe au bleu.

18. Sélectionnez l’activité Copie et vérifiez que vous voyez les propriétés de l’activité dans la fenêtre **Propriétés**.

19. Basculez vers l’onglet **Source** dans la fenêtre **Propriétés**, et effectuez les étapes suivantes :

    1. Dans la zone **Jeu de données source**, sélectionnez **SourceDataset**.

    2. Sous **Utiliser la requête**, sélectionnez **Requête**.

    3. Entrez la requête SQL dans la zone **Requête**. Voici un exemple de requête :

    ```sql
    select * from TemperatureSensor where timestamp > '@{activity('OldWaterMark').output.firstRow.WatermarkValue}' and timestamp <= '@{activity('NewWaterMark').output.firstRow.NewWatermarkvalue}'
    ```

20. Sous l’onglet **Récepteur**, sélectionnez **Nouveau** sous **Jeu de données récepteur**.

21. Dans ce tutoriel, le magasin de données récepteur est un stockage Blob Azure. Sélectionnez **Stockage Blob Azure**, puis **Continuer** dans la fenêtre **Nouveau jeu de données**.

22. Dans la fenêtre **Sélectionner le format**, sélectionnez le format de vos données, puis **Continuer**.

23. Dans la fenêtre **Définir les propriétés**, sous **Nom**, entrez **SinkDataset**. Sous **Service lié**, sélectionnez **Nouveau**. Vous créez maintenant une connexion (un service lié) à votre stockage Blob Azure.

24. Dans la fenêtre **Nouveau service lié (stockage Blob Azure)** , effectuez ces étapes :

    1. Dans la zone **Nom**, entrez **AzureStorageLinkedService**.

    2. Sous **Nom du compte de stockage**, sélectionnez le compte de stockage Azure de votre abonnement Azure.

    3. Testez la connexion, puis sélectionnez **Terminer**.

25. Dans la fenêtre **Définir les propriétés**, vérifiez que **AzureStorageLinkedService** est sélectionné sous **Service lié**. Sélectionnez **Créer** et **OK**.

26. Sous l’onglet **Récepteur**, sélectionnez **Modifier**.

27. Accédez à l’onglet **Connexion** de SinkDataset et effectuez ces étapes :

    1. Sous **Chemin de fichier**, entrez *asdedatasync/incrementalcopy*, où *asdedatasync* est le nom du conteneur d’objets blob et *incrementalcopy* est le nom de dossier. Créez le conteneur s’il n’existe pas ou utilisez le nom d’un conteneur existant. Azure Data Factory crée automatiquement le dossier de sortie *incrementalcopy* s’il n’existe pas. Vous pouvez également utiliser le bouton **Parcourir** pour le **chemin d’accès du fichier** afin d’accéder à un dossier dans un conteneur d’objets blob.

    2. Pour la partie **Fichier** du **Chemin de fichier**, sélectionnez **Ajouter du contenu dynamique [Alt+P]** , puis entrez **@CONCAT('Incremental-', pipeline().RunId, '.txt')** dans la fenêtre qui s’ouvre. Sélectionnez **Terminer**. Le nom de fichier est généré dynamiquement par l’expression. Chaque exécution de pipeline possède un ID unique. L’activité de copie utilise l’ID d’exécution pour générer le nom de fichier.

28. Basculez vers l’éditeur de pipeline en sélectionnant l’onglet Pipeline en haut ou en sélectionnant le nom du pipeline dans l’arborescence à gauche.

29. Dans le volet **Activités**, développez **Général** et faites glisser l’activité **Procédure stockée** du volet **Activités** vers la surface du concepteur de pipeline. Connectez le résultat vert (succès) de l’activité Copie à l’activité Procédure stockée.

30. Sélectionnez **Activité de procédure stockée** dans le concepteur de pipeline et remplacez son nom par **SPtoUpdateWatermarkActivity**.

31. Passez à l’onglet **Compte SQL** et sélectionnez **_QLDBEdgeLinkedService_* sous **Service lié**.

32. Basculez vers l’onglet **Procédure stockée** et effectuez ces étapes :

    1. Sous **Nom de la procédure stockée**, sélectionnez **[dbo].[usp_write_watermark]** .

    2. Pour spécifier les valeurs des paramètres de procédure stockée, sélectionnez **Paramètre d’importation** et entrez ces valeurs pour les paramètres :

    |Nom|Type|Valeur|
    |-----|----|-----|
    |LastModifiedtime|DateTime|@{activity('NewWaterMark').output.firstRow.NewWatermarkvalue}|
    |TableName|String|@{activity('OldWaterMark').output.firstRow.TableName}|

33. Pour valider les paramètres du pipeline, sélectionnez **Valider** dans la barre d’outils. Vérifiez qu’il n’y a aucune erreur de validation. Pour fermer la fenêtre **Rapport de validation de pipeline**, sélectionnez **>>** .

34. Publiez des entités (services liés, jeux de données et pipelines) sur le service Azure Data Factory en sélectionnant le bouton **Tout publier**. Attendez le message de confirmation de réussite de l’opération de publication.

## <a name="trigger-a-pipeline-based-on-a-schedule"></a>Déclencher un pipeline en fonction d’une planification

1. Dans la barre d’outils du pipeline, sélectionnez **Ajouter un déclencheur**, sélectionnez **Nouveau/Modifier**, puis **Nouveau**.

2. Nommez votre déclencheur **HourlySync**. Sous **Type**, sélectionnez **Planification**. Définissez la **périodicité** sur Toutes les heures.

3. Sélectionnez **OK**.

4. Sélectionnez **Publier tout**.

5. Sélectionnez **Déclencher maintenant**.

6. Basculez vers l’onglet **Surveiller** sur la gauche. Vous pouvez voir l’état de l’exécution du pipeline déclenchée par le déclencheur manuel. Sélectionnez **Actualiser** pour actualiser la liste.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, le pipeline Azure Data Factory copie les données d’une table sur un instance SQL Edge vers un emplacement du Stockage Blob Azure toutes les heures. Pour plus d’informations sur l’utilisation de Data Factory dans d’autres scénarios, consultez ces [tutoriels](../data-factory/tutorial-copy-data-portal.md).
