---
title: Utiliser des points de terminaison privés pour créer un pipeline Azure Data Factory
description: Ce didacticiel fournit des instructions détaillées sur l’utilisation d’un portail Azure pour créer une fabrique de données avec un pipeline. Le pipeline utilise l’activité de copie pour copier des données du Stockage Blob Azure vers une base de données Azure SQL.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: tutorial
ms.custom: seo-lt-2019
ms.date: 05/15/2020
ms.author: jingwang
ms.openlocfilehash: 4f5d691ef99ac4647d2031d6588d0b3922edd8cf
ms.sourcegitcommit: 4bee52a3601b226cfc4e6eac71c1cb3b4b0eafe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94505986"
---
# <a name="copy-data-securely-from-azure-blob-storage-to-a-sql-database-by-using-private-endpoints"></a>Copier des données du stockage Blob Azure vers une base de données SQL de manière sécurisée en utilisant des points de terminaison privés

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Dans ce didacticiel, vous créez une fabrique de données à l’aide de l’interface utilisateur (IU) d’Azure Data Factory. *Le pipeline de cette fabrique de données copie de manière sécurisée les données du Stockage Blob Azure dans une base de données Azure SQL (les deux n’autorisant l’accès qu’aux réseaux sélectionnés) en utilisant des points de terminaison privés dans un [réseau virtuel managé Azure Data Factory](managed-virtual-network-private-endpoint.md).* Le modèle de configuration de ce didacticiel s’applique à la copie depuis un magasin de données de fichiers vers un magasin de données relationnelles. Pour connaître la liste des magasins de données pris en charge comme sources et récepteurs, consultez le tableau [Magasins de données et formats pris en charge](./copy-activity-overview.md).

> [!NOTE]
> Si vous débutez avec Data Factory, consultez [Présentation d’Azure Data Factory](./introduction.md).

Dans ce tutoriel, vous effectuez les étapes suivantes :

* Créer une fabrique de données.
* Créer un pipeline avec une activité de copie.


## <a name="prerequisites"></a>Prérequis
* **Abonnement Azure**. Si vous n’avez pas d’abonnement Azure, créez un [compte Azure gratuit](https://azure.microsoft.com/free/) avant de commencer.
* **Compte Azure Storage**. Vous utilisez le stockage Blob comme magasin de données *source*. Si vous ne possédez pas de compte de stockage, consultez l’article [Créer un compte de stockage Azure](../storage/common/storage-account-create.md?tabs=azure-portal) pour découvrir comment en créer un. *Vérifiez que le compte de stockage autorise l’accès provenant des réseaux sélectionnés uniquement.* 
* **Azure SQL Database**. Vous utilisez la base de données comme magasin de données *récepteur*. Si vous ne disposez pas d’une base de données Azure SQL, consultez [Créer une base de données SQL](../azure-sql/database/single-database-create-quickstart.md) pour découvrir comment en créer une. *Vérifiez que le compte SQL Database autorise l’accès provenant des réseaux sélectionnés uniquement.* 

### <a name="create-a-blob-and-a-sql-table"></a>Créer un objet blob et une table SQL

Préparez maintenant votre Stockage Blob et votre base de données SQL pour les besoins de ce tutoriel, en procédant comme suit.

#### <a name="create-a-source-blob"></a>Créer un objet blob source

1. Ouvrez le Bloc-notes. Copiez le texte suivant et enregistrez-le comme fichier **emp.txt** sur votre disque :

    ```
    FirstName,LastName
    John,Doe
    Jane,Doe
    ```

1. Créez un conteneur nommé **adftutorial** dans votre Stockage Blob. Créez un dossier nommé **input** dans ce conteneur. Ensuite, chargez le fichier **emp.txt** dans le dossier **input**. Utilisez le portail Azure ou des outils tels que l’[Explorateur Stockage Azure](https://storageexplorer.com/) pour effectuer ces tâches.

#### <a name="create-a-sink-sql-table"></a>Créer une table SQL de récepteur

Utilisez le script SQL suivant pour créer la table **dbo.emp** dans votre base de données SQL :

```sql
CREATE TABLE dbo.emp
(
    ID int IDENTITY(1,1) NOT NULL,
    FirstName varchar(50),
    LastName varchar(50)
)
GO

CREATE CLUSTERED INDEX IX_emp_ID ON dbo.emp (ID);
```

## <a name="create-a-data-factory"></a>Créer une fabrique de données
À cette étape, vous allez créer une fabrique de données et démarrer l’interface utilisateur de Data Factory afin de créer un pipeline dans la fabrique de données.

1. Ouvrez Microsoft Edge ou Google Chrome. Actuellement, seuls les navigateurs web Microsoft Edge et Google Chrome prennent en charge l’interface utilisateur de Data Factory.

1. Dans le menu de gauche, sélectionnez **Créer une ressource** > **Analytics** > **Data Factory**.

1. Sur la page **Nouvelle fabrique de données** , entrez **ADFTutorialDataFactory** dans le champ **Nom**.

   Le nom de la fabrique de données Azure doit être un nom *global unique*. Si vous recevez un message d’erreur concernant la valeur du nom, entrez un autre nom pour la fabrique de données (par exemple, votrenomADFTutorialDataFactory). Consultez l’article [Azure Data Factory - Règles d’affectation des noms](./naming-rules.md) pour savoir comment nommer les règles Data Factory.

1. Sélectionnez l’ **abonnement** Azure dans lequel vous voulez créer la fabrique de données.

1. Pour **Groupe de ressources** , réalisez l’une des opérations suivantes :

    - Sélectionnez **Utiliser l’existant** , puis sélectionnez un groupe de ressources existant dans la liste déroulante.
    - Sélectionnez **Créer** , puis entrez le nom d’un groupe de ressources. 
     
    Pour plus d’informations sur les groupes de ressources, consultez [Utilisation des groupes de ressources pour gérer vos ressources Azure](../azure-resource-manager/management/overview.md). 

1. Sous **Version** , sélectionnez **V2**.

1. Sous **Emplacement** , sélectionnez l’emplacement de la fabrique de données. Seuls les emplacements pris en charge apparaissent dans la liste déroulante. Les magasins de données (comme Stockage Azure et SQL Database) et les services de calcul (comme Azure HDInsight) utilisés par la fabrique de données peuvent se trouver dans d’autres régions.

1. Sélectionnez **Create** (Créer).

1. Une fois la création terminée, la notification apparaît dans le centre de notifications. Sélectionnez **Accéder à la ressource** pour accéder à la page **Data Factory**.

1. Sélectionnez **Créer et surveiller** pour lancer l’interface utilisateur de Data Factory dans un onglet séparé.

## <a name="create-an-azure-integration-runtime-in-data-factory-managed-virtual-network"></a>Création d’un runtime d’intégration Azure sur le réseau virtuel managé Data Factory
À cette étape, vous allez créer un runtime d’intégration Azure et activer le réseau virtuel managé Data Factory.

1. Sur le portail Data Factory, accédez à **Gérer** et sélectionnez **Créer** pour créer un runtime d’intégration Azure.

   ![Capture d’écran montrant la création d’un runtime d’intégration Azure](./media/tutorial-copy-data-portal-private/create-new-azure-ir.png)
1. Choisissez de créer un runtime d’intégration **Azure**.

   ![Capture d’écran montrant un nouveau runtime d’intégration Azure](./media/tutorial-copy-data-portal-private/azure-ir.png)
1. Sous **Configuration de réseau virtuel (préversion)** , sélectionnez **Activer**.

   ![Capture d’écran montrant l’activation d’un nouveau runtime d’intégration Azure](./media/tutorial-copy-data-portal-private/enable-managed-vnet.png)
1. Sélectionnez **Create** (Créer).

## <a name="create-a-pipeline"></a>Créer un pipeline
À cette étape, vous créez un pipeline avec une activité de copie dans la fabrique de données. L’activité de copie permet de copier les données d’un stockage Blob vers une base de données SQL. Dans le [didacticiel de démarrage rapide](./quickstart-create-data-factory-portal.md), vous avez créé un pipeline en procédant comme suit :

1. Créer le service lié.
1. Créer des jeux de données d’entrée et de sortie.
1. Créer un pipeline.

Dans ce tutoriel, vous allez commencer par créer un pipeline. Puis vous créez des jeux de données et des services liés lorsque vous en avez besoin pour configurer le pipeline.

1. Dans la page **Prise en main** , cliquez sur **Créer un pipeline**.

   ![Capture d’écran montrant la création d’un pipeline](./media/doc-common-process/get-started-page.png)
1. Dans le volet Propriétés du pipeline, entrez **CopyPipeline** comme nom du pipeline.

1. Dans la boîte à outils **Activités** , développez la catégorie **Déplacer et transformer** , puis faites glisser l’activité **Copier les données** de la boîte à outils vers l’aire du Concepteur de pipeline. Entrez **CopyFromBlobToSql** comme nom.

    ![Capture d’écran montrant l’activité de copie](./media/tutorial-copy-data-portal-private/drag-drop-copy-activity.png)

### <a name="configure-a-source"></a>Configuration d’une source

>[!TIP]
>Dans ce tutoriel, vous allez utiliser le type d’authentification **Clé de compte** pour votre magasin de données source. Vous pouvez également choisir d’autres méthodes d’authentification prises en charge, par exemple **URI SAP** , **Principal de service** et **Identité managée** si nécessaire. Pour plus d’informations, consultez les sections correspondantes dans [Copie et transformation de données dans le Stockage Blob Azure avec Azure Data Factory](./connector-azure-blob-storage.md#linked-service-properties).
>
>Pour stocker des secrets des magasins de données de manière sécurisée, nous vous recommandons également d’utiliser Azure Key Vault. Pour plus d’informations et d’illustrations, consultez [Stockage des informations d’identification dans Azure Key Vault](./store-credentials-in-key-vault.md).

#### <a name="create-a-source-dataset-and-linked-service"></a>Création d’un jeu de données source et d’un service lié

1. Accédez à l’onglet **Source**. Sélectionnez **+ Nouveau** pour créer un jeu de données source.

1. Dans la boîte de dialogue **Nouveau jeu de données** , sélectionnez **Stockage Blob Azure** , puis **Continuer**. Sachant que les données sources se trouvent dans un stockage Blob, vous devez sélectionner le **Stockage Blob Azure** pour le jeu de données source.

1. Dans la boîte de dialogue **Sélectionner le format** , sélectionnez le type de format de vos données, puis **Continuer**.

1. Dans la boîte de dialogue **Définir les propriétés** , entrez **SourceBlobDataset** comme **Nom**. Cochez la case **Première ligne comme en-tête**. Sous la zone de texte **Service lié** , sélectionnez **+ Nouveau**.

1. Dans la boîte de dialogue **Nouveau service lié (Stockage Blob Azure)** , entrez **AzureStorageLinkedService** comme **Nom** , puis sélectionnez votre compte de stockage dans la liste **Nom du compte de stockage**. 

1. Veillez à activer **Création interactive**. L’activation peut prendre environ une minute.

    ![Capture d’écran montrant Création interactive](./media/tutorial-copy-data-portal-private/interactive-authoring.png)

1. Sélectionnez **Tester la connexion**. Celle-ci doit échouer quand le compte de stockage autorise l’accès provenant des **Réseaux sélectionnés** uniquement et oblige Azure Data Factory à créer un point de terminaison privé vers le compte (le point de terminaison doit être approuvé avant utilisation du compte). Le message d’erreur comporte un lien permettant de créer un point de terminaison privé. Vous pouvez le suivre pour créer un point de terminaison privé managé. Pour créer un point de terminaison privé managé, vous pouvez également accéder directement à l’onglet **Gérer** et appliquer les instructions de la [section suivante](#create-a-managed-private-endpoint).

   > [!NOTE]
   > L’onglet **Gérer** n’est pas nécessairement disponible pour toutes les instances Data Factory. Si vous ne le voyez pas, vous pouvez accéder aux points de terminaison privés en sélectionnant **Auteur** > **Connexions** > **Point de terminaison privé**.
1. Laissez la boîte de dialogue ouverte, puis accédez à votre compte de stockage.

1. Suivez les instructions de [cette section](#approval-of-a-private-link-in-a-storage-account) pour approuver la liaison privée.

1. Revenez à la boîte de dialogue. Sélectionnez de nouveau **Tester la connexion** , puis sélectionnez **Créer** pour déployer le service lié.

1. Après la création du service lié, la page **Définir les propriétés** s’affiche de nouveau. En regard de **Chemin d’accès du fichier** , sélectionnez **Parcourir**.

1. Accédez au dossier **adftutorial/input** et sélectionnez le fichier **emp.txt** , puis **OK**.

1. Sélectionnez **OK**. La page du pipeline s’affiche automatiquement. Dans l’onglet **Source** , vérifiez que **SourceBlobDataset** est sélectionné. Pour afficher un aperçu des données dans cette page, sélectionnez **Aperçu des données**.

    ![Capture d’écran montrant le jeu de données source](./media/tutorial-copy-data-portal-private/source-dataset-selected.png)

#### <a name="create-a-managed-private-endpoint"></a>Créer un point de terminaison privé managé

Si vous n’avez pas sélectionné le lien hypertexte lorsque vous avez testé la connexion, suivez le chemin. Vous devez maintenant créer un point de terminaison privé managé que vous connecterez au service lié créé.

1. Accédez à l’onglet **Gérer**.

   > [!NOTE]
   > L’onglet **Gérer** n’est pas nécessairement disponible pour toutes les instances Data Factory. Si vous ne le voyez pas, vous pouvez accéder aux points de terminaison privés en sélectionnant **Auteur** > **Connexions** > **Point de terminaison privé**.

1. Accédez à la section **Points de terminaison privés managés**.

1. Sélectionnez **+ Nouveau** sous **Points de terminaison privés managés**.

    ![Capture d’écran montrant Points de terminaison privés managés > bouton Nouveau](./media/tutorial-copy-data-portal-private/new-managed-private-endpoint.png) 

1. Sélectionnez la vignette **Stockage Blob Azure** dans la liste, puis **Continuer**.

1. Entrez le nom du compte de stockage que vous avez créé.

1. Sélectionnez **Create** (Créer).

1. Au bout de quelques secondes, la liaison privée créée indique qu’une approbation est nécessaire.

1. Sélectionnez le point de terminaison privé que vous avez créé. Le lien hypertexte qui apparaît vous permet d’approuver le point de terminaison privé au niveau du compte de stockage.

    ![Capture d’écran montrant le volet Points de terminaison privés managés](./media/tutorial-copy-data-portal-private/manage-private-endpoint.png) 

#### <a name="approval-of-a-private-link-in-a-storage-account"></a>Approbation d’une liaison privée dans un compte de stockage
1. Dans la section **Paramètres** du compte de stockage, accédez à **Connexions des points de terminaison privés**.

1. Cochez la case du point de terminaison privé que vous avez créé, puis sélectionnez **Approuver**.

    ![Capture d’écran montrant le bouton Approuver pour le point de terminaison privé](./media/tutorial-copy-data-portal-private/approve-private-endpoint.png)

1. Ajoutez une description, puis sélectionnez **Oui**.
1. Revenez à la section **Points de terminaison privés managés** de l’onglet **Gérer** dans Data Factory.
1. Au bout d’environ une ou deux minutes, l’approbation de votre point de terminaison privé apparaît dans l’interface utilisateur de Data Factory.


### <a name="configure-a-sink"></a>Configuration d’un récepteur
>[!TIP]
>Dans ce tutoriel, vous allez utiliser le type d’authentification **Authentification SQL** pour votre magasin de données récepteur. Vous pouvez également choisir d’autres méthodes d’authentification prises en charge, par exemple **Principal de service** et **Identité managée** si nécessaire. Pour plus d’informations, consultez les sections correspondantes dans [Copie et transformation de données dans Azure SQL Database avec Azure Data Factory](./connector-azure-sql-database.md#linked-service-properties).
>
>Pour stocker des secrets des magasins de données de manière sécurisée, nous vous recommandons également d’utiliser Azure Key Vault. Pour plus d’informations et d’illustrations, consultez [Stockage des informations d’identification dans Azure Key Vault](./store-credentials-in-key-vault.md).

#### <a name="create-a-sink-dataset-and-linked-service"></a>Création d’un jeu de données récepteur et d’un service lié
1. Accédez à l’onglet **Récepteur** , puis sélectionnez **+ Nouveau** pour créer un jeu de données récepteur.

1. Dans la boîte de dialogue **Nouveau jeu de données** , entrez **SQL** dans la zone de recherche pour filtrer les connecteurs. Sélectionnez **Azure SQL Database** , puis **Continuer**. Dans ce didacticiel, vous copiez des données vers une base de données SQL.

1. Dans la boîte de dialogue **Définir les propriétés** , entrez **OutputSqlDataset** comme **Nom**. Dans la liste déroulante **Service lié** , sélectionnez **+ Créer**. Un jeu de données doit être associé à un service lié. Le service lié comporte la chaîne de connexion utilisée par Data Factory pour établir la connexion à la base de données SQL lors de l’exécution. Le jeu de données spécifie le conteneur, le dossier et le fichier (facultatif) dans lequel les données sont copiées.

1. Dans la boîte de dialogue **Nouveau service lié (Azure SQL Database)** , procédez comme suit :

    1. Sous **Nom** , entrez **AzureSqlDatabaseLinkedService**.
    1. Sous **Nom du serveur** , sélectionnez votre instance SQL Server.
    1. Veillez à activer **Création interactive**.
    1. Sous **Nom de la base de données** , sélectionnez votre base de données SQL.
    1. Sous **Nom d’utilisateur** , entrez le nom de l’utilisateur.
    1. Sous **Mot de passe** , entrez le mot de passe de l’utilisateur.
    1. Sélectionnez **Tester la connexion**. Celle-ci doit échouer, car le serveur SQL autorise l’accès provenant des **Réseaux sélectionnés** uniquement et oblige Azure Data Factory à créer un point de terminaison privé vers le compte (le point de terminaison doit être approuvé avant utilisation du compte). Le message d’erreur comporte un lien permettant de créer un point de terminaison privé. Vous pouvez le suivre pour créer un point de terminaison privé managé. Pour créer un point de terminaison privé managé, vous pouvez également accéder directement à l’onglet **Gérer** et appliquer les instructions de la section suivante.
    1. Laissez la boîte de dialogue ouverte, puis accédez au serveur SQL sélectionné.
    1. Suivez les instructions de [cette section](#approval-of-a-private-link-in-sql-server) pour approuver la liaison privée.
    1. Revenez à la boîte de dialogue. Sélectionnez de nouveau **Tester la connexion** , puis sélectionnez **Créer** pour déployer le service lié.

1. La boîte de dialogue **Définir les propriétés** s’affiche automatiquement. Sélectionnez **[dbo].[emp]** dans le champ **Table**. Sélectionnez ensuite **OK**.

1. Accédez à l’onglet avec le pipeline. Ensuite, vérifiez que **OutputSqlDataset** est sélectionné dans **Jeu de données récepteur**.

    ![Capture d’écran montrant l’onglet Pipeline](./media/tutorial-copy-data-portal-private/pipeline-tab-2.png)

Si vous le souhaitez, vous pouvez mapper le schéma de la source au schéma correspondant de la destination en suivant les instructions mentionnées dans [Mappage de schéma dans l’activité de copie](./copy-activity-schema-and-type-mapping.md).

#### <a name="create-a-managed-private-endpoint"></a>Créer un point de terminaison privé managé

Si vous n’avez pas sélectionné le lien hypertexte lorsque vous avez testé la connexion, suivez le chemin. Vous devez maintenant créer un point de terminaison privé managé que vous connecterez au service lié créé.

1. Accédez à l’onglet **Gérer**.
1. Accédez à la section **Points de terminaison privés managés**.
1. Sélectionnez **+ Nouveau** sous **Points de terminaison privés managés**.

    ![Capture d’écran montrant Points de terminaison privés managés > bouton Nouveau](./media/tutorial-copy-data-portal-private/new-managed-private-endpoint.png) 

1. Sélectionnez la vignette **Azure SQL Database** dans la liste, puis **Continuer**.
1. Entrez le nom du serveur SQL que vous avez sélectionné.
1. Sélectionnez **Create** (Créer).
1. Au bout de quelques secondes, la liaison privée créée indique qu’une approbation est nécessaire.
1. Sélectionnez le point de terminaison privé que vous avez créé. Le lien hypertexte qui apparaît vous permet d’approuver le point de terminaison privé au niveau du serveur SQL.


#### <a name="approval-of-a-private-link-in-sql-server"></a>Approbation d’une liaison privée dans SQL Server
1. Dans la section **Paramètres** du serveur SQL, accédez à **Connexions des points de terminaison privés**.
1. Cochez la case du point de terminaison privé que vous avez créé, puis sélectionnez **Approuver**.
1. Ajoutez une description, puis sélectionnez **Oui**.
1. Revenez à la section **Points de terminaison privés managés** de l’onglet **Gérer** dans Data Factory.
1. L’approbation de votre point de terminaison privé prend une ou deux minutes.

#### <a name="debug-and-publish-the-pipeline"></a>Déboguer et publier le pipeline

Vous pouvez déboguer un pipeline avant de publier des artefacts (services liés, jeux de données et pipeline) dans Data Factory ou votre propre référentiel Azure Repos Git.

1. Pour déboguer le pipeline, sélectionnez **Déboguer** dans la barre d’outils. L’état d’exécution du pipeline apparaît dans l’onglet **Sortie** au bas de la fenêtre.
1. Une fois que le pipeline peut s’exécuter correctement, sélectionnez **Tout publier** dans la barre d’outils supérieure. Cette action a pour effet de publier les entités (jeux de données et pipelines) que vous avez créées dans Data Factory.
1. Patientez jusqu’à voir le message **Publication réussie**. Pour voir les messages de notification, sélectionnez **Afficher les notifications** en haut à droite (bouton en forme de cloche).


#### <a name="summary"></a>Récapitulatif
Dans cet exemple, le pipeline copie des données d’un Stockage Blob dans SQL Database en utilisant des points de terminaison privés dans un réseau virtuel managé Data Factory. Vous avez appris à :

* Créer une fabrique de données.
* Créer un pipeline avec une activité de copie.