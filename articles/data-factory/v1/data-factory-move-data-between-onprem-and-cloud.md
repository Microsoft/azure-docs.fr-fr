---
title: Déplacement de données – Passerelle de gestion des données
description: Mettez en place une passerelle de données pour déplacer vos données entre un emplacement local et le cloud. Utilisez la passerelle de gestion des données dans Azure Data Factory pour déplacer vos données.
services: data-factory
documentationcenter: ''
author: nabhishek
manager: anandsub
ms.assetid: 7bf6d8fd-04b5-499d-bd19-eff217aa4a9c
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: abnarain
robots: noindex
ms.openlocfilehash: 7f07f08cd320d94495403b0f5ae65d60d8dc93b5
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96019682"
---
# <a name="move-data-between-on-premises-sources-and-the-cloud-with-data-management-gateway"></a>Déplacement de données entre des sources locales et le cloud à l’aide de la passerelle de gestion des données
> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez [Copier des données entre un emplacement local et le cloud](../tutorial-hybrid-copy-powershell.md).

Cet article présente l’intégration des données entre les magasins de données locaux et les magasins de données cloud à l’aide de Data Factory. Il s’appuie sur l’article [Activités de déplacement des données](data-factory-data-movement-activities.md) et d’autres articles sur les principaux concepts Data Factory : les [jeux de données](data-factory-create-datasets.md) et les [pipelines](data-factory-create-pipelines.md).

## <a name="data-management-gateway"></a>Passerelle de gestion de données
Vous devez installer la passerelle de gestion des données sur votre ordinateur local pour activer le déplacement des données vers/depuis un magasin de données local. La passerelle peut être installée sur le même ordinateur que le magasin de données ou sur un autre ordinateur, pourvu qu’elle puisse se connecter au magasin de données.

> [!IMPORTANT]
> Consultez l’article [Passerelle de gestion des données](data-factory-data-management-gateway.md) pour obtenir des informations détaillées sur la passerelle de gestion des données.

La procédure pas à pas suivante explique comment créer une fabrique de données avec un pipeline qui déplace les données d’une base de données **SQL Server** locale vers un Stockage Blob Azure. Dans le cadre de la procédure pas à pas, vous installez et configurez la passerelle de gestion des données sur votre ordinateur.

## <a name="walkthrough-copy-on-premises-data-to-cloud"></a>Procédure pas à pas : copier des données locales vers le cloud
Dans cette procédure pas à pas, vous effectuez les étapes suivantes :

1. Créer une fabrique de données.
2. Créer une passerelle de gestion des données.
3. Créer des services liés pour les magasins de données des sources et récepteurs.
4. Créer des jeux de données pour représenter les données d’entrée et de sortie.
5. Créer un pipeline avec une activité de copie pour déplacer les données.

## <a name="prerequisites-for-the-tutorial"></a>Configuration requise pour le didacticiel
Avant de commencer ce guide, vous devez disposer des éléments suivants :

* **Abonnement Azure**.  Si vous n'êtes pas abonné, vous pouvez créer un compte d'essai gratuit en quelques minutes. Consultez l'article [Essai gratuit](https://azure.microsoft.com/pricing/free-trial/) pour plus d'informations.
* **Compte Azure Storage**. Dans le cadre de ce didacticiel, le stockage d’objets blob est utilisé comme magasin de données **destination/réception**. Si vous n’avez pas de compte de stockage Azure, consultez l’article [Créer un compte de stockage](../../storage/common/storage-account-create.md) pour découvrir comment en créer un.
* **SQL Server**. Dans le cadre de ce didacticiel, vous utilisez une base de données SQL Server locale comme magasin de données **source**.

## <a name="create-data-factory"></a>Créer une fabrique de données
Dans cette étape, vous allez utiliser le portail Azure pour créer une instance Azure Data Factory nommée **ADFTutorialOnPremDF**.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Cliquez sur **Créer une ressource**, sur **Décisionnel + analytique**, puis sur **Fabrique de données**.

   ![Nouveau -> DataFactory](./media/data-factory-move-data-between-onprem-and-cloud/NewDataFactoryMenu.png)  
3. Dans la page **Nouvelle fabrique de données**, dans le champ Nom, entrez **ADFTutorialOnPremDF**.

    ![Ajouter au Tableau d'accueil](./media/data-factory-move-data-between-onprem-and-cloud/OnPremNewDataFactoryAddToStartboard.png)

   > [!IMPORTANT]
   > Le nom de la fabrique de données Azure doit être un nom global unique. Si vous recevez le message d’erreur : **Le nom de la fabrique de données « ADFTutorialOnPremDF » n’est pas disponible**, changez le nom de la fabrique de données (par exemple votrenomADFTutorialOnPremDF), puis réessayez de la créer. Utilisez ce nom à la place d'ADFTutorialOnPremDF quand vous effectuez les étapes restantes de ce didacticiel.
   >
   > Le nom de la fabrique de données pourra être enregistré en tant que nom **DNS** et devenir ainsi visible publiquement.
   >
   >
4. Sélectionnez l’ **abonnement Azure** où vous voulez que la fabrique de données soit créée.
5. Sélectionnez un **groupe de ressources** existant ou créez-en un. Pour ce tutoriel, créez un groupe de ressources nommé : **ADFTutorialResourceGroup**.
6. Cliquez sur **Créer** dans la page **Nouvelle fabrique de données**.

   > [!IMPORTANT]
   > Pour créer des instances Data Factory, vous devez avoir un rôle de [collaborateur de fabrique de données](../../role-based-access-control/built-in-roles.md#data-factory-contributor) au niveau de l’abonnement/du groupe de ressources.
   >
   >
7. Une fois la création terminée, la page **Data Factory** s’affiche comme sur l’image suivante :

   ![Page d’accueil Data Factory](./media/data-factory-move-data-between-onprem-and-cloud/OnPremDataFactoryHomePage.png)

## <a name="create-gateway"></a>Créer une passerelle
1. Dans la page **Data Factory**, cliquez sur la vignette **Créer et déployer** pour lancer l’**éditeur** de la fabrique de données.

    ![Vignette Créer et déployer](./media/data-factory-move-data-between-onprem-and-cloud/author-deploy-tile.png)
2. Dans Data Factory Editor, dans la barre d’outils, cliquez sur **... Plus**, puis cliquez sur **Nouvelle passerelle de données**. Vous pouvez également cliquer avec le bouton droit sur **Passerelles de données** dans l’arborescence, puis cliquer sur **Nouvelle passerelle de données**.

   ![Nouvelle passerelle de données sur la barre d’outils](./media/data-factory-move-data-between-onprem-and-cloud/NewDataGateway.png)
3. Dans la page **Créer**, entrez **adftutorialgateway** dans le champ **Nom**, puis cliquez sur **OK**.     

    ![Page Créer une passerelle](./media/data-factory-move-data-between-onprem-and-cloud/OnPremCreateGatewayBlade.png)

    > [!NOTE]
    > Dans cette procédure pas à pas, vous créez la passerelle logique avec un seul nœud (ordinateur Windows local). Vous pouvez effectuer un scale-out d’une passerelle de gestion des données en associant plusieurs machines locales avec la passerelle. Vous pouvez monter en puissance une passerelle en augmentant le nombre de travaux de déplacement des données qui peuvent s’exécuter simultanément sur un nœud. Cette fonctionnalité est également disponible pour une passerelle logique à nœud unique. Consultez l’article [Mise à l’échelle de la passerelle de gestion des données dans Azure Data Factory](data-factory-data-management-gateway-high-availability-scalability.md) pour plus d’informations.  
4. Dans la page **Configurer**, cliquez sur **Installer directement sur cet ordinateur**. Cette action télécharge le package d’installation de la passerelle, installe, configure et inscrit la passerelle sur l’ordinateur.  

   > [!NOTE]
   > Utilisez Internet Explorer ou un navigateur web compatible Microsoft ClickOnce.
   >
   > Si vous utilisez Chrome, accédez au [Chrome Web Store](https://chrome.google.com/webstore/), faites une recherche sur le mot-clé « ClickOnce », choisissez l’une des extensions ClickOnce, puis installez-la.
   >
   > Faites de même pour Firefox (installez un complément). Cliquez sur le bouton **Ouvrir le menu** dans la barre d’outils (**trois lignes horizontales** en haut à droite), cliquez sur **Modules complémentaires**, effectuez une recherche avec le mot-clé « ClickOnce », choisissez l’une des extensions de ClickOnce et installez le programme.    
   >
   >

    ![Passerelle - Page Configurer](./media/data-factory-move-data-between-onprem-and-cloud/OnPremGatewayConfigureBlade.png)

    Il s’agit de la méthode la plus simple (un clic) pour télécharger, installer, configurer et inscrire la passerelle en une seule étape. Vous pouvez voir que l’application **Gestionnaire de configuration de la passerelle de gestion de données Microsoft** est installée sur votre ordinateur. Vous pouvez aussi trouver l’exécutable **ConfigManager.exe** dans le dossier : **C:\Program Files\Microsoft Data Management Gateway\2.0\Shared**.

    Vous pouvez également télécharger et installer manuellement la passerelle en utilisant les liens de cette page et l’inscrire à l’aide de la clé indiquée dans la zone de texte **NOUVELLE CLÉ**.

    Consultez l’article [Data Management Gateway](data-factory-data-management-gateway.md) (Passerelle de gestion des données) pour obtenir des informations détaillées sur la passerelle.

   > [!NOTE]
   > Vous devez être administrateur sur l’ordinateur local pour pouvoir installer et configurer la passerelle de gestion des données avec succès. Vous pouvez ajouter des utilisateurs supplémentaires au groupe Windows local **Utilisateurs de la passerelle de gestion des données** . Les membres de ce groupe peuvent utiliser l’outil Gestionnaire de configuration de la passerelle de gestion des données pour configurer la passerelle.
   >
   >
5. Attendez quelques minutes, ou patientez jusqu’à ce que le message de notification suivant s’affiche :

    ![Gateway installation successful (Installation réussie de la passerelle)](./media/data-factory-move-data-between-onprem-and-cloud/gateway-install-success.png)
6. Lancez l’application **Gestionnaire de configuration de passerelle de gestion des données** sur votre ordinateur. Dans la fenêtre **Rechercher**, saisissez **passerelle de gestion de données** pour accéder à cet utilitaire. Vous pouvez aussi trouver l’exécutable **ConfigManager.exe** dans le dossier : **C:\Program Files\Microsoft Data Management Gateway\2.0\Shared**

    ![Gestionnaire de configuration de la passerelle](./media/data-factory-move-data-between-onprem-and-cloud/OnPremDMGConfigurationManager.png)
7. Vérifiez que le message `adftutorialgateway is connected to the cloud service` s’affiche. La barre d’état située au bas de l’écran affiche le message **Connecté au service de cloud** accompagné d’une **coche verte**.

    Sous l’onglet **Accueil**, vous pouvez également effectuer les opérations suivantes :

   * **Inscrire** une passerelle avec une clé obtenue à partir du portail Azure en utilisant le bouton Inscrire.
   * **Arrêter** le service hôte de la passerelle de gestion des données en cours d’exécution sur la machine passerelle.
   * **Planifier des mises à jour** à installer à une heure spécifique de la journée.
   * Voir l’heure de **dernière mise à jour** de la passerelle.
   * Spécifier l’heure à laquelle une mise à jour de la passerelle peut être installée.
8. Basculez vers l’onglet **Paramètres** . Le certificat spécifié dans la section **Certificat** est utilisé pour chiffrer/déchiffrer les informations d’identification du magasin de données local que vous fournissez dans le portail (facultatif). Cliquez sur **Modifier** pour utiliser votre propre certificat à la place. Par défaut, la passerelle utilise le certificat généré automatiquement par le service Data Factory.

    ![Configuration de certificat de la passerelle](./media/data-factory-move-data-between-onprem-and-cloud/gateway-certificate.png)

    Vous pouvez également effectuer les actions suivantes sous l’onglet **Paramètres**:

   * Afficher ou exporter le certificat utilisé par la passerelle.
   * Modifier le point de terminaison HTTPS utilisé par la passerelle.    
   * Définir un proxy HTTP que la passerelle peut utiliser.     
9. (facultatif) Basculez sur l’onglet **Diagnostics**, et cochez l’option **Activer la journalisation détaillée** si vous souhaitez activer la journalisation détaillée à utiliser pour résoudre les problèmes de passerelle. Vous trouverez les informations de journalisation dans **l’Observateur d’événements** sous le nœud **Journaux des applications et des services** -> **Passerelle de gestion des données**.

    ![Onglet Diagnostic](./media/data-factory-move-data-between-onprem-and-cloud/diagnostics-tab.png)

    Vous pouvez également effectuer les actions suivantes dans l’onglet **Diagnostics** :

   * Utilisez la section **Tester la connexion** à une source de données locale à l’aide de la passerelle.
   * Cliquez sur **Afficher les journaux d’activité** pour consulter le journal d’activité de la passerelle de gestion des données dans une fenêtre de l’Observateur d’événements.
   * Cliquez sur **Envoyer des journaux d’activité** pour charger un fichier zip contenant les journaux d’activité des sept derniers jours et l’envoyer à Microsoft pour faciliter le dépannage en cas de problèmes.
10. Sous l’onglet **Diagnostics**, dans la section **Tester la connexion**, sélectionnez **SqlServer** pour le type de magasin de données, entrez le nom du serveur de base de données et le nom de la base de données, spécifiez le type d’authentification, entrez un nom d’utilisateur et un mot de passe, puis cliquez sur **Tester** pour tester si la passerelle peut se connecter à la base de données.
11. Basculez vers le navigateur web, puis, dans le **portail Azure**, cliquez sur **OK** dans la page **Configurer**, puis dans la page **Nouvelle passerelle de données**.
12. Vous devez voir **adftutorialgateway** sous **Passerelles de données** dans l’arborescence de gauche.  Si vous cliquez dessus, vous devez voir le code JSON associé.

## <a name="create-linked-services"></a>Créez des services liés
Dans cette étape, vous créez deux services liés : **AzureStorageLinkedService** et **SqlServerLinkedService**. Le service lié **SqlServerLinkedService** associe une base de données SQL Server, et le service lié **AzureStorageLinkedService** associe un magasin d’objets blob Azure à la fabrique de données. Plus loin dans cette procédure pas à pas, vous allez créer un pipeline qui copie les données de la base de données SQL Server vers le magasin d’objets blob Azure.

#### <a name="add-a-linked-service-to-a-sql-server-database"></a>Ajout d’un service lié à une base de données SQL Server
1. Dans **Data Factory Editor**, cliquez sur **Nouvelle banque de données** sur la barre d’outils, puis sélectionnez **SQL Server**.

   ![Nouveau service lié SQL Server](./media/data-factory-move-data-between-onprem-and-cloud/NewSQLServer.png)
2. Dans l’**éditeur JSON** à droite, procédez comme suit :

   1. Pour **gatewayName**, spécifiez **adftutorialgateway**.    
   2. Dans **connectionString**, procédez comme suit :    

      1. Pour **servername**, entrez le nom du serveur qui héberge la base de données SQL Server.
      2. Pour **databasename**, entrez le nom de la base de données.
      3. Cliquez sur le bouton **Chiffrer** dans la barre d’outils. L’application Gestionnaire des informations d’identification apparaît.

         ![Application Gestionnaire des informations d’identification](./media/data-factory-move-data-between-onprem-and-cloud/credentials-manager-application.png)
      4. Dans la boîte de dialogue **Définition des informations d’identification**, spécifiez le type d’authentification, le nom d’utilisateur et le mot de passe, puis cliquez sur **OK**. Si la connexion est réussie, les informations d’identification chiffrées sont stockées dans le JSON, et la boîte de dialogue se ferme.
      5. Fermez l’onglet de navigateur vide qui a lancé la boîte de dialogue s’il ne se ferme pas automatiquement, puis revenez à l’onglet du portail Azure.

         Sur la machine passerelle, ces informations d’identification sont **chiffrées** à l’aide d’un certificat appartenant au service Data Factory. Si vous préférez utiliser le certificat qui est associé à la passerelle de gestion des données, consultez Set credentials securely (Configuration des informations d’identification de manière sécurisée).    
   3. Cliquez sur l’option **Déployer** de la barre de commandes pour déployer le service lié SQL Server. Vous devez voir le service lié dans l’arborescence.

      ![Service lié SQL Server dans l’arborescence](./media/data-factory-move-data-between-onprem-and-cloud/sql-linked-service-in-tree-view.png)    

#### <a name="add-a-linked-service-for-an-azure-storage-account"></a>Ajout d’un service lié pour un compte de stockage Azure
1. Dans **Data Factory Editor**, dans la barre de commandes, cliquez sur **Nouvelle banque de données**, puis sur **Stockage Azure**.
2. Entrez le nom de votre compte de stockage Azure dans le champ **Nom du compte**.
3. Entrez la clé de votre compte de stockage Azure dans le champ **Clé du compte**.
4. Cliquez sur **Déployer** pour déployer le service lié **AzureStorageLinkedService**.

## <a name="create-datasets"></a>Créez les jeux de données
Dans cette étape, vous allez créer des jeux de données d’entrée et de sortie qui représentent les données d’entrée et de sortie pour l’opération de copie (base de données SQL Server locale = > stockage d’objets blob Azure). Avant de créer des jeux de données, procédez comme suit (des étapes détaillées suivent la liste) :

* Créez une table nommée **emp** dans la base de données SQL Server que vous avez ajoutée en tant que service lié à la fabrique de données, puis insérez quelques exemples d’entrées dans la table.
* Créez un conteneur d’objets blobs nommé **adftutorial** dans le compte de stockage d’objets blobs Azure que vous avez ajouté en tant que service associé à la fabrique de données.

### <a name="prepare-on-premises-sql-server-for-the-tutorial"></a>Préparation du serveur SQL Server local pour le didacticiel
1. Dans la base de données que vous avez spécifiée pour le service lié SQL Server (**SqlServerLinkedService**), utilisez le script SQL suivant pour créer la table **emp** dans la base de données.

    ```SQL   
    CREATE TABLE dbo.emp
    (
        ID int IDENTITY(1,1) NOT NULL,
        FirstName varchar(50),
        LastName varchar(50),
        CONSTRAINT PK_emp PRIMARY KEY (ID)
    )
    GO
    ```
2. Insérer des exemples dans la table :

    ```SQL
    INSERT INTO emp VALUES ('John', 'Doe')
    INSERT INTO emp VALUES ('Jane', 'Doe')
    ```

### <a name="create-input-dataset"></a>Créer le jeu de données d’entrée

1. Dans **Data Factory Editor**, cliquez sur **... Plus**, dans la barre de commande, cliquez sur **Nouveau jeu de données**, puis sur **Table SQL Server**.
2. Remplacez le code JSON du volet droit par le texte suivant :

    ```JSON   
    {        
        "name": "EmpOnPremSQLTable",
        "properties": {
            "type": "SqlServerTable",
            "linkedServiceName": "SqlServerLinkedService",
            "typeProperties": {
                "tableName": "emp"
            },
            "external": true,
            "availability": {
                "frequency": "Hour",
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
   Notez les points suivants :

   * Le **type** est défini sur **SqlServerTable**.
   * **tableName** est défini sur **emp**.
   * Le paramètre **linkedServiceName** est défini sur **SqlServerLinkedService** (vous avez créé ce service lié précédemment dans le cadre de la procédure pas à pas).
   * Pour un jeu de données d’entrée non généré par un autre pipeline dans Azure Data Factory, vous devez définir **external** sur **true**. Cela signifie que les données d’entrée sont produites à l’extérieur du service Azure Data Factory. Vous pouvez éventuellement spécifier des stratégies de données externes à l’aide de l’élément **externalData** dans la section **Policy**.    

   Pour plus de détails sur les propriétés JSON, voir [Déplacer des données vers/à partir SQL Server](data-factory-sqlserver-connector.md).
3. Cliquez sur **Déployer** dans la barre de commandes pour déployer le jeu de données.  

### <a name="create-output-dataset"></a>Créer un jeu de données de sortie

1. Dans **Data Factory Editor**, cliquez sur **Nouveau jeu de données** dans la barre de commandes, puis sur **Stockage Blob Azure**.
2. Remplacez le code JSON du volet droit par le texte suivant :

    ```JSON   
    {
        "name": "OutputBlobTable",
        "properties": {
            "type": "AzureBlob",
            "linkedServiceName": "AzureStorageLinkedService",
            "typeProperties": {
                "folderPath": "adftutorial/outfromonpremdf",
                "format": {
                    "type": "TextFormat",
                    "columnDelimiter": ","
                }
            },
            "availability": {
                "frequency": "Hour",
                "interval": 1
            }
        }
     }
    ```   
   Notez les points suivants :

   * **type** est défini sur **AzureBlob**.
   * Le paramètre **linkedServiceName** est défini sur **AzureStorageLinkedService** (que vous avez créé à l’étape 2).
   * Le paramètre **folderPath** est défini sur **adftutorial/outfromonpremdf**, où « outfromonpremdf » est le dossier dans le conteneur adftutorial. S’il n’existe pas déjà, créez le conteneur **adftutorial** .
   * **availability** est défini sur **hourly** (**frequency** a la valeur **hour** et **interval** est défini sur **1**).  Le service Data Factory génère une tranche de données de sortie toutes les heures dans la table **emp** de la base de données Azure SQL.

   Si vous ne spécifiez pas **fileName** pour une **table de sortie**, les fichiers générés dans **folderPath** sont nommés au format suivant : `Data.<Guid>.txt` (par exemple Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt.).

   Pour affecter une valeur à **folderPath** et **fileName** de manière dynamique en fonction de l’heure de **SliceStart**, utilisez la propriété partitionedBy. Dans l’exemple suivant, folderPath utilise les valeurs Year, Month et Day à partir de SliceStart (heure de début de la partie en cours de traitement), alors que fileName utilise la valeur Hour à partir de SliceStart. Par exemple, si une partie est produite pour 2014-10-20T08:00:00, la valeur folderName est wikidatagateway/wikisampledataout/2014/10/20, alors que la valeur de fileName est 08.csv.

    ```JSON
    "folderPath": "wikidatagateway/wikisampledataout/{Year}/{Month}/{Day}",
    "fileName": "{Hour}.csv",
    "partitionedBy":
    [

        { "name": "Year", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyy" } },
        { "name": "Month", "value": { "type": "DateTime", "date": "SliceStart", "format": "MM" } },
        { "name": "Day", "value": { "type": "DateTime", "date": "SliceStart", "format": "dd" } },
        { "name": "Hour", "value": { "type": "DateTime", "date": "SliceStart", "format": "hh" } }
    ],
    ```

    Pour plus de détails sur les propriétés JSON, voir [Déplacer des données vers/à partir de Stockage Blob Azure](data-factory-azure-blob-connector.md).
3. Cliquez sur **Déployer** dans la barre de commandes pour déployer le jeu de données. Vérifiez que les jeux de données s’affichent dans l’arborescence.  

## <a name="create-pipeline"></a>Création d’un pipeline
Dans cette étape, vous créez un **pipeline** avec une **activité Copier l’activité** qui utilise **EmpOnPremSQLTable** en tant qu’entrée et **OutputBlobTable** en tant que sortie.

1. Dans Data Factory Editor, cliquez sur **... Plus**, puis sur **Nouveau pipeline**.
2. Remplacez le code JSON du volet droit par le texte suivant :    

    ```JSON   
     {
         "name": "ADFTutorialPipelineOnPrem",
         "properties": {
         "description": "This pipeline has one Copy activity that copies data from an on premises SQL to Azure blob",
         "activities": [
           {
             "name": "CopyFromSQLtoBlob",
             "description": "Copy data from on premises SQL server to blob",
             "type": "Copy",
             "inputs": [
               {
                 "name": "EmpOnPremSQLTable"
               }
             ],
             "outputs": [
               {
                 "name": "OutputBlobTable"
               }
             ],
             "typeProperties": {
               "source": {
                 "type": "SqlSource",
                 "sqlReaderQuery": "select * from emp"
               },
               "sink": {
                 "type": "BlobSink"
               }
             },
             "Policy": {
               "concurrency": 1,
               "executionPriorityOrder": "NewestFirst",
               "style": "StartOfInterval",
               "retry": 0,
               "timeout": "01:00:00"
             }
           }
         ],
         "start": "2016-07-05T00:00:00Z",
         "end": "2016-07-06T00:00:00Z",
         "isPaused": false
       }
     }
    ```   
   > [!IMPORTANT]
   > Remplacez la valeur de la propriété **start** par le jour actuel et la valeur **end**, par le jour suivant.
   >
   >

   Notez les points suivants :

   * Dans la section des activités, toutes les activités ont le **type** **Copy**.
   * **L’entrée** de l’activité est définie sur **EmpOnPremSQLTable** et la **sortie** de l’activité, sur **OutputBlobTable**.
   * Dans la section **typeProperties**, **SqlSource** est spécifié en tant que **Type de source** et **BlobSink** en tant que **Type de récepteur**.
   * La requête SQL `select * from emp` est spécifiée pour la propriété **sqlReaderQuery** de **SqlSource**.

   Les dates/heures de début et de fin doivent toutes deux être au [format ISO](https://en.wikipedia.org/wiki/ISO_8601). Par exemple : 2014-10-14T16:32:41Z. L’heure de fin ( **end** ) est facultative, mais nous allons l’utiliser dans ce didacticiel.

   Si vous ne spécifiez aucune valeur pour la propriété **end**, cette dernière est calculée comme suit : « **start + 48 heures** ». Pour exécuter le pipeline indéfiniment, spécifiez **9/9/9999** comme valeur pour la propriété **end**.

   Vous définissez la durée pendant laquelle les tranches de données seront traitées en fonction des propriétés de **Disponibilité** définies pour chaque jeu de données Azure Data Factory.

   Dans l’exemple ci-dessus, il existe 24 tranches de données, car une tranche est générée par heure.        
3. Cliquez sur l’option **Déployer** de la barre de commandes pour déployer le jeu de données (la table est un jeu de données rectangulaire). Vérifiez que le pipeline s’affiche dans l’arborescence sous le nœud **Pipelines**.  
4. Maintenant, cliquez sur **X** à deux reprises pour fermer la page et revenir à la page **Data Factory** pour **ADFTutorialOnPremDF**.

**Félicitations !** Vous avez correctement créé une fabrique de données Azure, des services liés, des jeux de données et un pipeline, et planifié le pipeline.

#### <a name="view-the-data-factory-in-a-diagram-view"></a>Afficher une vue schématique d'une fabrique de données
1. Dans le **portail Azure**, cliquez sur la vignette **Diagramme** sur la page d’accueil de la fabrique de données **ADFTutorialOnPremDF**. :

    ![Lien Diagramme](./media/data-factory-move-data-between-onprem-and-cloud/OnPremDiagramLink.png)
2. Le diagramme devrait ressembler à l’image suivante :

    ![Vue du diagramme](./media/data-factory-move-data-between-onprem-and-cloud/OnPremDiagramView.png)

    Vous pouvez faire un zoom avant, un zoom arrière, un zoom à 100 %, un zoom pour ajuster, positionner automatiquement les pipelines et les jeux de données, et afficher les informations de lignage (mise en surbrillance des éléments en amont et en aval des éléments sélectionnés).  Vous pouvez double-cliquer sur un objet (jeu de données d’entrée/de sortie) pour afficher ses propriétés.

## <a name="monitor-pipeline"></a>Surveillance d’un pipeline
Dans cette étape, vous utilisez le portail Azure pour surveiller ce qui se passe dans une fabrique de données Azure. Vous pouvez également utiliser les applets de commande PowerShell pour surveiller les jeux de données et les pipelines. Pour plus de détails sur la surveillance, consultez [Surveillance et gestion des pipelines](data-factory-monitor-manage-pipelines.md).

1. Dans le diagramme, double-cliquez sur **EmpOnPremSQLTable**.  

    ![Tranches EmpOnPremSQLTable](./media/data-factory-move-data-between-onprem-and-cloud/OnPremSQLTableSlicesBlade.png)
2. Notez que toutes les tranches de données sont dans l’état **Prêt** parce que la durée du pipeline (de l’heure de début à l’heure de fin) s’inscrit dans le passé. Cela est dû au fait que vous avez inséré les données dans la base de données SQL Server et qu’elles y sont tout le temps. Vérifiez qu’aucune tranche n’apparaît dans la section **Tranches problématiques** , sur la partie inférieure de la fenêtre. Pour afficher toutes les tranches, cliquez sur **Afficher plus** en bas de la liste des tranches.
3. À présent, dans la page **Jeux de données**, cliquez sur **OutputBlobTable**.

    ![Tranches OputputBlobTable](./media/data-factory-move-data-between-onprem-and-cloud/OutputBlobTableSlicesBlade.png)
4. Cliquez sur une tranche de données dans la liste pour afficher la page **Tranche de données**. Les activités exécutées pour cette tranche s’affichent. Généralement, une seule activité exécutée s’affiche.  

    ![Panneau Tranche de données](./media/data-factory-move-data-between-onprem-and-cloud/DataSlice.png)

    Si la tranche n’a pas l’état **Prêt**, vous pouvez voir les tranches en amont qui ne sont pas prêtes et qui empêchent l’exécution de la tranche actuelle dans la liste **Tranches en amont qui ne sont pas prêtes**.
5. Cliquez sur **l’exécution d’activité** dans la liste de la partie inférieure de la fenêtre pour afficher les **détails sur l’exécution d’activité**.

   ![Page Détails de l’exécution d’activité](./media/data-factory-move-data-between-onprem-and-cloud/ActivityRunDetailsBlade.png)

   Vous devriez voir des informations telles que le débit, la durée et la passerelle utilisée pour transférer les données.
6. Cliquez sur **X** pour fermer toutes les pages jusqu’à ce que vous
7. reveniez à la page d’accueil de l’élément **ADFTutorialOnPremDF**.
8. (facultatif) Cliquez sur **Pipelines**, sur **ADFTutorialOnPremDF**, puis accédez aux tables d’entrée (**Consommé**) ou aux jeux de données de sortie (**Produit**).
9. Utilisez des outils tels que [Microsoft Storage Explorer](https://storageexplorer.com/) pour vérifier qu’un objet blob/fichier est créé pour chaque heure.

   ![Explorateur de stockage Azure](./media/data-factory-move-data-between-onprem-and-cloud/OnPremAzureStorageExplorer.png)

## <a name="next-steps"></a>Étapes suivantes
* Consultez l’article [Data Management Gateway](data-factory-data-management-gateway.md) (Passerelle de gestion des données) pour obtenir des informations détaillées sur la passerelle de gestion des données.
* Consultez l’article [Copie de données d’un objet blob vers Azure SQL](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) pour en savoir plus sur l’utilisation de l’activité de copie pour déplacer des données d’une banque de données source vers une banque de données récepteur.
