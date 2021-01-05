---
title: Copier des données de SQL Server dans un stockage Blob avec PowerShell
description: Découvrez comment copier les données d’un magasin de données local dans le cloud Azure en utilisant le runtime d’intégration auto-hébergé d’Azure Data Factory.
services: data-factory
author: nabhishek
ms.author: abnarain
manager: anandsub
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: tutorial
ms.custom: seo-lt-2019; seo-dt-2019, devx-track-azurepowershell
ms.date: 01/22/2018
ms.openlocfilehash: bed497dfdc25ec2815f51795a1f40847586ce798
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97510309"
---
# <a name="tutorial-copy-data-from-a-sql-server-database-to-azure-blob-storage"></a>Tutoriel : Copier des données d’une base de données SQL Server vers un stockage Blob Azure

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Dans ce tutoriel, vous allez utiliser Azure PowerShell pour créer un pipeline Data Factory qui copie les données d’une base de données SQL Server dans un stockage Blob Azure. Vous allez créer et utiliser un runtime d’intégration auto-hébergé, qui déplace les données entre les banques de données locales et cloud.

> [!NOTE]
> Cet article ne fournit pas de présentation détaillée du service Data Factory. Pour plus d’informations, consultez [Présentation d’Azure Data Factory](introduction.md).

Dans ce tutoriel, vous effectuerez les étapes suivantes :

> [!div class="checklist"]
> * Créer une fabrique de données.
> * Créez un runtime d’intégration auto-hébergé.
> * Créer des services liés SQL Server et au Stockage Azure.
> * Créer des jeux de données SQL Server et Blob Azure.
> * Créer un pipeline avec une activité de copie pour déplacer les données.
> * Démarrer une exécution de pipeline.
> * Surveiller l’exécution du pipeline.

## <a name="prerequisites"></a>Prérequis
### <a name="azure-subscription"></a>Abonnement Azure
Si vous n’avez pas d’abonnement Azure, [créez un compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

### <a name="azure-roles"></a>Rôles Azure
Pour créer des instances de fabrique de données, le compte d’utilisateur que vous utilisez pour vous connecter à Azure doit avoir le rôle de *Contributeur* ou de *Propriétaire*, ou être *administrateur* de l’abonnement Azure.

Dans le portail Azure, cliquez sur votre nom d’utilisateur dans le coin supérieur droit, puis sélectionnez **Autorisations** pour afficher les autorisations dont vous disposez dans l’abonnement. Si vous avez accès à plusieurs abonnements, sélectionnez l’abonnement approprié. Pour obtenir des exemples d’instructions sur l’ajout d’un utilisateur à un rôle, consultez l’article [Ajouter ou supprimer des attributions de rôle Azure à l’aide du portail Azure](../role-based-access-control/role-assignments-portal.md).

### <a name="sql-server-2014-2016-and-2017"></a>SQL Server 2014, 2016 et 2017
Dans le cadre de ce tutoriel, vous allez utiliser une base de données SQL Server comme magasin de données *source*. Le pipeline de la fabrique de données que vous allez créer dans ce tutoriel copie les données de cette base de données SQL Server (source) dans un stockage Blob Azure (récepteur). Créez ensuite une table nommée **emp** dans votre base de données SQL Server, puis insérez quelques exemples d’entrées dans la table.

1. Exécutez SQL Server Management Studio. S’il n’est pas déjà installé sur votre machine, accédez à [Télécharger SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms).

1. Connectez-vous à votre instance SQL Server à l’aide de vos informations d’identification.

1. Créez un exemple de base de données. Dans l’arborescence, cliquez avec le bouton droit sur **Bases de données**, puis sur **Nouvelle base de données**.

1. Dans la fenêtre **Nouvelle base de données**, entrez un nom pour la base de données, puis cliquez sur **OK**.

1. Pour créer la table **emp** et y insérer quelques données d’exemple, exécutez le script de requête suivant sur la base de données. Dans l’arborescence, cliquez avec le bouton droit sur la base de données créée, puis sur **Nouvelle requête**.

    ```sql
    CREATE TABLE dbo.emp
    (
        ID int IDENTITY(1,1) NOT NULL,
        FirstName varchar(50),
        LastName varchar(50)
    )
    GO

    INSERT INTO emp (FirstName, LastName) VALUES ('John', 'Doe')
    INSERT INTO emp (FirstName, LastName) VALUES ('Jane', 'Doe')
    GO
    ```


### <a name="azure-storage-account"></a>Compte de Stockage Azure
Dans ce tutoriel, vous utilisez un compte Stockage Azure à usage général (stockage d’objets Blob Azure spécifiquement) comme banque de données réceptrice/de destination. Si vous ne possédez pas de compte Stockage Azure à usage général, consultez [Créer un compte de stockage](../storage/common/storage-account-create.md). Le pipeline de la fabrique de données que vous allez créer dans ce tutoriel copie les données de la base de données SQL Server (source) dans ce stockage Blob Azure (récepteur). 

#### <a name="get-storage-account-name-and-account-key"></a>Obtenir le nom de compte de stockage et la clé de compte
Dans ce tutoriel, vous spécifiez le nom et la clé de votre compte Stockage Azure. Procédez comme suit pour obtenir le nom et la clé de votre compte de stockage :

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec vos nom d’utilisateur et mot de passe Azure.

1. Dans le volet gauche, sélectionnez **Plus de services**, filtrez à l’aide du mot-clé **Stockage**, puis sélectionnez **Comptes de stockage**.

    ![Rechercher le compte de stockage](media/doc-common-process/search-storage-account.png)

1. Dans la liste des comptes de stockage, appliquez un filtre pour votre compte de stockage (si nécessaire), puis sélectionnez votre compte de stockage.

1. Dans la fenêtre **Compte de stockage**, sélectionnez **Clés d’accès**.

1. Dans les zones **Nom du compte de stockage** et **key1**, copiez les valeurs, puis collez-les dans le bloc-notes ou un autre éditeur pour une utilisation ultérieure dans le tutoriel.

#### <a name="create-the-adftutorial-container"></a>Créer le conteneur adftutorial
Dans cette section, vous allez créer un conteneur d’objets blob nommé **adftutorial** dans votre stockage Blob Azure.

1. Dans la fenêtre **Compte de stockage**, basculez vers **Vue d’ensemble**, puis sélectionnez **Objets blob**.

    ![Sélection de l’option Objets blob](media/tutorial-hybrid-copy-powershell/select-blobs.png)

1. Dans la fenêtre **Service Blob**, sélectionnez **Conteneur**.

1. Dans la fenêtre **Nouveau conteneur**, dans la zone **Nom** , entrez **adftutorial**, puis sélectionnez **OK**.

    ![Saisie du nom du conteneur](media/tutorial-hybrid-copy-powershell/new-container-dialog.png)

1. Cliquez sur **adftutorial** dans la liste des conteneurs.  

1. Gardez la fenêtre **conteneur** de **adftutorial** ouverte. Elle vous permet de vérifier la sortie à la fin du tutoriel. Data Factory crée automatiquement le dossier de sortie de ce conteneur, de sorte que vous n’avez pas besoin d’en créer.


### <a name="windows-powershell"></a>Windows PowerShell

#### <a name="install-azure-powershell"></a>Installation d’Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Installez la dernière version d’Azure PowerShell, si elle n’est pas installée sur votre machine. Pour des instructions détaillées, consultez [Installation et configuration d’Azure PowerShell](/powershell/azure/install-Az-ps).

#### <a name="log-in-to-powershell"></a>Se connecter à PowerShell

1. Démarrez PowerShell sur votre machine et laissez-le ouvert jusqu’à la fin de ce tutoriel de démarrage rapide. Si vous le fermez, puis le rouvrez, vous devez réexécuter ces commandes.

1. Exécutez la commande suivante, puis saisissez le nom d’utilisateur et le mot de passe Azure que vous utilisez pour la connexion au portail Azure :

    ```powershell
    Connect-AzAccount
    ```        

1. Si vous avez plusieurs abonnements Azure, exécutez la commande suivante pour sélectionner l’abonnement avec lequel vous souhaitez travailler. Remplacez **SubscriptionId** par l’ID de votre abonnement Azure :

    ```powershell
    Select-AzSubscription -SubscriptionId "<SubscriptionId>"    
    ```

## <a name="create-a-data-factory"></a>Créer une fabrique de données

1. Définissez une variable pour le nom du groupe de ressources que vous utiliserez ultérieurement dans les commandes PowerShell. Copiez la commande suivante dans PowerShell, spécifiez un nom pour le [groupe de ressources Azure](../azure-resource-manager/management/overview.md) (entre guillemets doubles, par exemple `"adfrg"`), puis exécutez la commande. 
   
    ```powershell
    $resourceGroupName = "ADFTutorialResourceGroup"
    ```

1. Pour créer le groupe de ressources Azure, exécutez la commande suivante :

    ```powershell
    New-AzResourceGroup $resourceGroupName -location 'East US'
    ```

    Si le groupe de ressources existe déjà, vous pouvez ne pas le remplacer. Affectez une valeur différente à la variable `$resourceGroupName` et exécutez à nouveau la commande.

1. Définissez une variable pour le nom de la fabrique de données que vous pourrez utiliser dans les commandes PowerShell plus tard. Les noms doivent commencer par une lettre ou un chiffre, et peuvent comporter uniquement des lettres, des chiffres et des tirets (-).

    > [!IMPORTANT]
    >  Mettez à jour le nom de la fabrique de données afin qu’il soit globalement unique. Par exemple, ADFTutorialFactorySP1127.

    ```powershell
    $dataFactoryName = "ADFTutorialFactory"
    ```

1. Définissez une variable pour l’emplacement de la fabrique de données :

    ```powershell
    $location = "East US"
    ```  

1. Exécutez l’applet de commande `Set-AzDataFactoryV2` suivante pour créer la fabrique de données :

    ```powershell       
    Set-AzDataFactoryV2 -ResourceGroupName $resourceGroupName -Location $location -Name $dataFactoryName
    ```

> [!NOTE]
>
> * Le nom de la fabrique de données doit être un nom global unique. Si vous recevez l’erreur suivante, changez le nom, puis réessayez.
>    ```
>    The specified data factory name 'ADFv2TutorialDataFactory' is already in use. Data factory names must be globally unique.
>    ```
> * Pour créer des instances de fabrique de données, le compte d’utilisateur que vous utilisez pour vous connecter à Azure doit être un membre des rôles *contributeur* ou *propriétaire*, ou un *administrateur* de l’abonnement Azure.
> * Pour obtenir la liste des régions Azure dans lesquelles Data Factory est actuellement disponible, sélectionnez les régions qui vous intéressent dans la page suivante, puis développez **Analytique** pour localiser **Data Factory** : [Disponibilité des produits par région](https://azure.microsoft.com/global-infrastructure/services/). Les magasins de données (Stockage Azure, Azure SQL Database, etc.) et les services de calcul (Azure HDInsight, etc.) utilisés par la fabrique de données peuvent se trouver dans d’autres régions.
>
>

## <a name="create-a-self-hosted-integration-runtime"></a>Créer un runtime d’intégration auto-hébergé

Dans cette section, vous allez créer un runtime d’intégration auto-hébergé et l’associer à un ordinateur local avec la base de données SQL Server. Le runtime d’intégration auto-hébergé est le composant qui copie les données de la base de données SQL Server sur votre machine dans le stockage Blob Azure.

1. Créez une variable pour le nom du runtime d’intégration. Utilisez un nom unique, et notez-le. Vous l’utiliserez ultérieurement dans ce tutoriel.

    ```powershell
   $integrationRuntimeName = "ADFTutorialIR"
    ```

1. Créez un runtime d’intégration auto-hébergé.

    ```powershell
    Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $resourceGroupName -DataFactoryName $dataFactoryName -Name $integrationRuntimeName -Type SelfHosted -Description "selfhosted IR description"
    ```

    Voici l'exemple de sortie :

    ```console
    Name              : ADFTutorialIR
    Type              : SelfHosted
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    Description       : selfhosted IR description
    Id                : /subscriptions/<subscription ID>/resourceGroups/<resourceGroupName>/providers/Microsoft.DataFactory/factories/<dataFactoryName>/integrationruntimes/<integrationRuntimeName>
    ```

1. Exécutez la commande suivante pour récupérer l’état du runtime d’intégration créé :

    ```powershell
   Get-AzDataFactoryV2IntegrationRuntime -name $integrationRuntimeName -ResourceGroupName $resourceGroupName -DataFactoryName $dataFactoryName -Status
    ```

    Voici l'exemple de sortie :

    ```console
    State                     : NeedRegistration
    Version                   :
    CreateTime                : 9/10/2019 3:24:09 AM
    AutoUpdate                : On
    ScheduledUpdateDate       :
    UpdateDelayOffset         :
    LocalTimeZoneOffset       :
    InternalChannelEncryption :
    Capabilities              : {}
    ServiceUrls               : {eu.frontend.clouddatahub.net}
    Nodes                     : {}
    Links                     : {}
    Name                      : <Integration Runtime name>
    Type                      : SelfHosted
    ResourceGroupName         : <resourceGroup name>
    DataFactoryName           : <dataFactory name>
    Description               : selfhosted IR description
    Id                        : /subscriptions/<subscription ID>/resourceGroups/<resourceGroupName>/providers/Microsoft.DataFactory/factories/<dataFactoryName>/integrationruntimes/<integrationRuntimeName>
    ```

1. Exécutez la commande suivante pour récupérer les *clés d’authentification* permettant d’enregistrer le runtime d’intégration auto-hébergé auprès du service de fabrique de données dans le cloud. Copiez l’une des clés (sans les guillemets) pour enregistrer le runtime d’intégration auto-hébergé que vous allez installer sur votre ordinateur à l’étape suivante.

    ```powershell
    Get-AzDataFactoryV2IntegrationRuntimeKey -Name $integrationRuntimeName -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName | ConvertTo-Json
    ```

    Voici l'exemple de sortie :

    ```json
    {
        "AuthKey1":  "IR@0000000000-0000-0000-0000-000000000000@xy0@xy@xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx=",
        "AuthKey2":  "IR@0000000000-0000-0000-0000-000000000000@xy0@xy@yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy="
    }
    ```

## <a name="install-the-integration-runtime"></a>Installer le runtime d’intégration
1. Téléchargez le [runtime d’intégration Azure Data Factory](https://www.microsoft.com/download/details.aspx?id=39717) sur un ordinateur Windows local, puis exécutez l’installation.

1. Sur la page **Bienvenue dans l’assistant Installation de Microsoft Integration Runtime**, cliquez sur **Suivant**.  

1. Dans la fenêtre **Contrat de licence utilisateur final**, acceptez les conditions et le contrat de licence, puis cliquez sur **Suivant**.

1. Dans la fenêtre **Dossier de destination**, cliquez sur **Suivant**.

1. Dans la fenêtre **Prêt à installer Microsoft Integration Runtime**, cliquez sur **Installer**.

1. Dans l’**Assistant Installation de Microsoft Integration Runtime terminé**, cliquez sur **Terminer**.

1. Dans la fenêtre **Inscrire Microsoft Integration Runtime (auto-hébergé)** , collez la clé que vous avez enregistrée dans la section précédente, puis cliquez sur **Inscrire**.

    ![Inscrire le runtime d’intégration](media/tutorial-hybrid-copy-powershell/register-integration-runtime.png)

1. Dans la fenêtre **Nouveau nœud Runtime d’intégration (autohébergé)** , sélectionnez **Terminer**.

    ![Fenêtre Nouveau nœud Integration Runtime](media/tutorial-hybrid-copy-powershell/new-integration-runtime-node-page.png)

 1. Le message suivant s’affiche une fois que le runtime d’intégration auto-hébergé est bien inscrit :

    ![Inscription réussie](media/tutorial-hybrid-copy-powershell/registered-successfully.png)

1. Dans la fenêtre **Inscrire Microsoft Integration Runtime (auto-hébergé)** , cliquez sur **Lancer Configuration Manager**.

1. Le message suivant apparaît une fois que le nœud est connecté au service cloud :

    ![Le nœud est connecté](media/tutorial-hybrid-copy-powershell/node-is-connected.png)

1. Testez la connectivité à votre base de données SQL Server en procédant comme suit :

    a. Dans la fenêtre **Gestionnaire de configuration**, basculez vers l’onglet **Diagnostics**.

    b. Dans la zone **Type de source de données**, sélectionnez **SqlServer**.

    c. Saisissez le nom du serveur.

    d. Saisissez le nom de la base de données.

    e. Sélectionnez le mode d’authentification.

    f. Saisissez le nom d’utilisateur.

    g. Entrez le mot de passe associé au nom d’utilisateur.

    h. Cliquez sur **Tester** pour vérifier que le runtime d’intégration se connecte à SQL Server.  
    ![Connexion établie](media/tutorial-hybrid-copy-powershell/config-manager-diagnostics-tab.png)

    Une coche verte apparaît si la connexion est établie. Sinon, c’est un message d’erreur concernant l’échec qui apparaît. Corrigez les problèmes et assurez-vous que le runtime d’intégration peut se connecter à votre instance SQL Server.

    Notez toutes les valeurs précédentes pour une utilisation ultérieure dans ce tutoriel.

## <a name="create-linked-services"></a>Créez des services liés
Créez des services liés dans la fabrique de données pour lier vos magasins de données et vos services de calcul à la fabrique de données. Dans ce tutoriel, vous liez votre compte Stockage Azure et votre instance SQL Server au magasin de données. Les services liés comportent les informations de connexion utilisées par le service de fabrique de données lors de l’exécution pour s’y connecter.

### <a name="create-an-azure-storage-linked-service-destinationsink"></a>Créer un service lié Stockage Azure (destination/réception)
Dans cette étape, vous liez votre compte Stockage Azure à la fabrique de données.

1. Créez un fichier JSON sous le nom *AzureSqlDWLinkedService.json* dans le dossier *C:\ADFv2Tutorial* avec le code suivant. Créez le dossier *ADFv2Tutorial* s’il n’existe pas déjà.  

    > [!IMPORTANT]
    > Avant d’enregistrer le fichier, remplacez \<accountName> et \<accountKey> par le nom et la clé de votre compte de stockage Azure. Vous les avez notées dans la section [Conditions préalables](#get-storage-account-name-and-account-key).

   ```json
    {
        "name": "AzureStorageLinkedService",
        "properties": {
            "annotations": [],
            "type": "AzureBlobStorage",
            "typeProperties": {
                "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountName>;AccountKey=<accountKey>;EndpointSuffix=core.windows.net"
            }
        }
    }
   ```

1. Dans PowerShell, accédez au dossier *C:\ADFv2Tutorial*.
   ```powershell
   Set-Location 'C:\ADFv2Tutorial'    
   ```

1. Exécutez l’applet de commande `Set-AzDataFactoryV2LinkedService` suivante pour créer le service lié AzureStorageLinkedService :

   ```powershell
   Set-AzDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $ResourceGroupName -Name "AzureStorageLinkedService" -File ".\AzureStorageLinkedService.json"
   ```

   Voici un exemple de sortie :

    ```console
    LinkedServiceName : AzureStorageLinkedService
    ResourceGroupName : <resourceGroup name>
    DataFactoryName   : <dataFactory name>
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureBlobStorageLinkedService
    ```

    Si vous recevez une erreur indiquant « fichier introuvable », vérifiez que le fichier existe en exécutant la commande `dir`. Si le nom du fichier a l’extension *.txt* (par exemple, AzureStorageLinkedService.json.txt), supprimez-la puis exécutez la commande PowerShell à nouveau.

### <a name="create-and-encrypt-a-sql-server-linked-service-source"></a>Créer et chiffrer un service lié SQL Server (source)
Dans cette étape, vous liez votre instance SQL Server à la fabrique de données.

1. Créez un fichier JSON sous le nom *SqlServerLinkedService.json* dans le dossier *C:\ADFv2Tutorial* avec le code suivant :

    > [!IMPORTANT]
    > Sélectionnez la section en fonction de l’authentification utilisée pour établir la connexion à SQL Server.

    **Utilisation de l’authentification SQL :**

    ```json
    {  
        "name":"SqlServerLinkedService",
        "type":"Microsoft.DataFactory/factories/linkedservices",
        "properties":{  
            "annotations":[  

            ],
            "type":"SqlServer",
            "typeProperties":{  
                "connectionString":"integrated security=False;data source=<serverName>;initial catalog=<databaseName>;user id=<userName>;password=<password>"
            },
            "connectVia":{  
                "referenceName":"<integration runtime name> ",
                "type":"IntegrationRuntimeReference"
            }
        }
    }
   ```    

    **Avec l’authentification Windows :**

    ```json
    {  
        "name":"SqlServerLinkedService",
        "type":"Microsoft.DataFactory/factories/linkedservices",
        "properties":{  
            "annotations":[  

            ],
            "type":"SqlServer",
            "typeProperties":{  
                "connectionString":"integrated security=True;data source=<serverName>;initial catalog=<databaseName>",
                "userName":"<username> or <domain>\\<username>",
                "password":{  
                    "type":"SecureString",
                    "value":"<password>"
                }
            },
            "connectVia":{  
                "referenceName":"<integration runtime name>",
                "type":"IntegrationRuntimeReference"
            }
        }
    }
    ```

    > [!IMPORTANT]
    > - Sélectionnez la section en fonction de l’authentification utilisée pour établir la connexion à votre instance SQL Server.
    > - Remplacez **\<integration runtime name>** par le nom de votre runtime d’intégration.
    > - Avant d’enregistrer le fichier, remplacez **\<servername>** , **\<databasename>** , **\<username>** et **\<password>** par les valeurs de votre instance SQL Server.
    > - Si vous avez besoin d’utiliser une barre oblique (\\) dans votre nom de serveur ou de compte d’utilisateur, faites-la précéder d’un caractère d’échappement (\\). Par exemple, utilisez *mydomain\\\\myuser*.

1. Pour chiffrer les données sensibles (nom d’utilisateur, mot de passe etc.) exécutez l’applet de commande `New-AzDataFactoryV2LinkedServiceEncryptedCredential`.  
    Les informations d’identification sont alors chiffrées à l’aide de l’API de protection des données (DPAPI). Les informations d’identification chiffrées sont stockées localement sur le nœud de runtime d’intégration auto-hébergé (ordinateur local). La charge utile de sortie peut être redirigée vers un autre fichier JSON (dans ce cas, *encryptedLinkedService.json*) qui contient les informations d’identification chiffrées.

   ```powershell
   New-AzDataFactoryV2LinkedServiceEncryptedCredential -DataFactoryName $dataFactoryName -ResourceGroupName $ResourceGroupName -IntegrationRuntimeName $integrationRuntimeName -File ".\SQLServerLinkedService.json" > encryptedSQLServerLinkedService.json
   ```

1. Exécutez la commande suivante, qui crée EncryptedSqlServerLinkedService :

   ```powershell
   Set-AzDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $ResourceGroupName -Name "EncryptedSqlServerLinkedService" -File ".\encryptedSqlServerLinkedService.json"
   ```


## <a name="create-datasets"></a>Créez les jeux de données
Dans cette étape, vous allez créer des jeux de données d’entrée et de sortie. Ils représentent les données d’entrée et de sortie pour l’opération de copie, qui copie les données depuis la base de données SQL Server vers le stockage Blob Azure.

### <a name="create-a-dataset-for-the-source-sql-server-database"></a>Créer un jeu de données pour la base de données SQL Server source
Dans cette étape, vous définissez un jeu de données qui représente les données dans l’instance de la base de données SQL Server. Le jeu de données est de type SqlServerTable. Il fait référence au service lié SQL Server que vous avez créé à l’étape précédente. Le service lié comporte les informations de connexion utilisées par le service de fabrique de données pour établir la connexion à l’instance SQL Server lors de l’exécution. Ce jeu de données spécifie la table SQL dans la base de données qui contient les données. Dans ce didacticiel, c’est la table **emp** qui contient les données source.

1. Créez un fichier JSON sous le nom *SqlServerDataset.json* dans le dossier *C:\ADFv2Tutorial* avec le code suivant :  
    ```json
    {  
        "name":"SqlServerDataset",
        "properties":{  
            "linkedServiceName":{  
                "referenceName":"EncryptedSqlServerLinkedService",
                "type":"LinkedServiceReference"
            },
            "annotations":[  

            ],
            "type":"SqlServerTable",
            "schema":[  

            ],
            "typeProperties":{  
                "schema":"dbo",
                "table":"emp"
            }
        }
    }
    ```

1. Pour créer le jeu de données SqlServerDataset, exécutez l’applet de commande `Set-AzDataFactoryV2Dataset`.

    ```powershell
    Set-AzDataFactoryV2Dataset -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "SqlServerDataset" -File ".\SqlServerDataset.json"
    ```

    Voici l'exemple de sortie :

    ```console
    DatasetName       : SqlServerDataset
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    Structure         :
    Properties        : Microsoft.Azure.Management.DataFactory.Models.SqlServerTableDataset
    ```

### <a name="create-a-dataset-for-azure-blob-storage-sink"></a>Créer un jeu de données pour le stockage Blob Azure (récepteur)
Dans cette étape, vous définissez un jeu de données qui représente les données devant être copiées dans le stockage Blob Azure. Le jeu de données est de type AzureBlob. Il fait référence au service lié Stockage Azure que vous avez créé précédemment dans ce didacticiel.

Le service lié comporte les informations de connexion utilisées par le service de fabrique de données lors de l’exécution pour établir la connexion à votre compte Stockage Azure. Ce jeu de données spécifie le dossier du stockage Azure dans lequel les données sont copiées à partir de la base de données SQL Server. Dans ce didacticiel, le dossier est : *adftutorial/fromonprem* où `adftutorial` est le conteneur d’objets blob et `fromonprem` le dossier.

1. Créez un fichier JSON sous le nom *AzureBlobDataset.json* dans le dossier *C:\ADFv2Tutorial* avec le code suivant :

    ```json
    {  
        "name":"AzureBlobDataset",
        "properties":{  
            "linkedServiceName":{  
                "referenceName":"AzureStorageLinkedService",
                "type":"LinkedServiceReference"
            },
            "annotations":[  

            ],
            "type":"DelimitedText",
            "typeProperties":{  
                "location":{  
                    "type":"AzureBlobStorageLocation",
                    "folderPath":"fromonprem",
                    "container":"adftutorial"
                },
                "columnDelimiter":",",
                "escapeChar":"\\",
                "quoteChar":"\""
            },
            "schema":[  

            ]
        },
        "type":"Microsoft.DataFactory/factories/datasets"
    }
    ```

1. Pour créer le jeu de données AzureBlobDataset, exécutez l’applet de commande `Set-AzDataFactoryV2Dataset`.

    ```powershell
    Set-AzDataFactoryV2Dataset -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "AzureBlobDataset" -File ".\AzureBlobDataset.json"
    ```

    Voici l'exemple de sortie :

    ```console
    DatasetName       : AzureBlobDataset
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    Structure         :
    Properties        : Microsoft.Azure.Management.DataFactory.Models.DelimitedTextDataset
    ```

## <a name="create-a-pipeline"></a>Créer un pipeline
Dans ce tutoriel, vous allez créer un pipeline avec une activité de copie. L’activité de copie utilise SqlServerDataset en tant que jeu de données d’entrée et AzureBlobDataset en tant que jeu de données de sortie. Le type de source est défini sur *SqlSource* et le type de récepteur sur *BlobSink*.

1. Créez un fichier JSON sous le nom *SqlServerToBlobPipeline.json* dans le dossier *C:\ADFv2Tutorial* avec le code suivant :

    ```json
    {  
        "name":"SqlServerToBlobPipeline",
        "properties":{  
            "activities":[  
                {  
                    "name":"CopySqlServerToAzureBlobActivity",
                    "type":"Copy",
                    "dependsOn":[  

                    ],
                    "policy":{  
                        "timeout":"7.00:00:00",
                        "retry":0,
                        "retryIntervalInSeconds":30,
                        "secureOutput":false,
                        "secureInput":false
                    },
                    "userProperties":[  

                    ],
                    "typeProperties":{  
                        "source":{  
                            "type":"SqlServerSource"
                        },
                        "sink":{  
                            "type":"DelimitedTextSink",
                            "storeSettings":{  
                                "type":"AzureBlobStorageWriteSettings"
                            },
                            "formatSettings":{  
                                "type":"DelimitedTextWriteSettings",
                                "quoteAllText":true,
                                "fileExtension":".txt"
                            }
                        },
                        "enableStaging":false
                    },
                    "inputs":[  
                        {  
                            "referenceName":"SqlServerDataset",
                            "type":"DatasetReference"
                        }
                    ],
                    "outputs":[  
                        {  
                            "referenceName":"AzureBlobDataset",
                            "type":"DatasetReference"
                        }
                    ]
                }
            ],
            "annotations":[  

            ]
        }
    }
    ```

1. Pour créer le pipeline SqlServerToBlobPipeline, exécutez l’applet de commande `Set-AzDataFactoryV2Pipeline`.

    ```powershell
    Set-AzDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "SQLServerToBlobPipeline" -File ".\SQLServerToBlobPipeline.json"
    ```

    Voici l'exemple de sortie :

    ```console
    PipelineName      : SQLServerToBlobPipeline
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    Activities        : {CopySqlServerToAzureBlobActivity}
    Parameters        :  
    ```

## <a name="create-a-pipeline-run"></a>Créer une exécution du pipeline
Démarrez l’exécution du pipeline SQLServerToBlobPipeline et capturez l’ID d’exécution du pipeline pour une surveillance ultérieure.

```powershell
$runId = Invoke-AzDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineName 'SQLServerToBlobPipeline'
```

## <a name="monitor-the-pipeline-run"></a>Surveiller l’exécution du pipeline.

1. Pour vérifier en permanence l’état d’exécution du pipeline SQLServerToBlobPipeline, exécutez le script suivant dans PowerShell et imprimez le résultat final :

    ```powershell
    while ($True) {
        $result = Get-AzDataFactoryV2ActivityRun -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineRunId $runId -RunStartedAfter (Get-Date).AddMinutes(-30) -RunStartedBefore (Get-Date).AddMinutes(30)

        if (($result | Where-Object { $_.Status -eq "InProgress" } | Measure-Object).count -ne 0) {
            Write-Host "Pipeline run status: In Progress" -foregroundcolor "Yellow"
            Start-Sleep -Seconds 30
        }
        else {
            Write-Host "Pipeline 'SQLServerToBlobPipeline' run finished. Result:" -foregroundcolor "Yellow"
            $result
            break
        }
    }
    ```

    Voici la sortie de l’exemple d’exécution :

    ```console
    ResourceGroupName    : <resourceGroupName>
    DataFactoryName      : <dataFactoryName>
    ActivityRunId        : 24af7cf6-efca-4a95-931d-067c5c921c25
    ActivityName         : CopySqlServerToAzureBlobActivity
    ActivityType         : Copy
    PipelineRunId        : 7b538846-fd4e-409c-99ef-2475329f5729
    PipelineName         : SQLServerToBlobPipeline
    Input                : {source, sink, enableStaging}
    Output               : {dataRead, dataWritten, filesWritten, sourcePeakConnections...}
    LinkedServiceName    :
    ActivityRunStart     : 9/11/2019 7:10:37 AM
    ActivityRunEnd       : 9/11/2019 7:10:58 AM
    DurationInMs         : 21094
    Status               : Succeeded
    Error                : {errorCode, message, failureType, target}
    AdditionalProperties : {[retryAttempt, ], [iterationHash, ], [userProperties, {}], [recoveryStatus, None]...}
    ```

1. Vous pouvez obtenir l’ID d’exécution du pipeline SQLServerToBlobPipeline, puis vérifiez le résultat détaillé de l’exécution d’activité en exécutant la commande suivante :

    ```powershell
    Write-Host "Pipeline 'SQLServerToBlobPipeline' run result:" -foregroundcolor "Yellow"
    ($result | Where-Object {$_.ActivityName -eq "CopySqlServerToAzureBlobActivity"}).Output.ToString()
    ```

    Voici la sortie de l’exemple d’exécution :

    ```json
    {  
        "dataRead":36,
        "dataWritten":32,
        "filesWritten":1,
        "sourcePeakConnections":1,
        "sinkPeakConnections":1,
        "rowsRead":2,
        "rowsCopied":2,
        "copyDuration":18,
        "throughput":0.01,
        "errors":[  

        ],
        "effectiveIntegrationRuntime":"ADFTutorialIR",
        "usedParallelCopies":1,
        "executionDetails":[  
            {  
                "source":{  
                    "type":"SqlServer"
                },
                "sink":{  
                    "type":"AzureBlobStorage",
                    "region":"CentralUS"
                },
                "status":"Succeeded",
                "start":"2019-09-11T07:10:38.2342905Z",
                "duration":18,
                "usedParallelCopies":1,
                "detailedDurations":{  
                    "queuingDuration":6,
                    "timeToFirstByte":0,
                    "transferDuration":5
                }
            }
        ]
    }
    ```

## <a name="verify-the-output"></a>Vérifier la sortie
Le pipeline crée automatiquement le dossier de sortie nommé *fromonprem* dans le conteneur d’objets blob `adftutorial`. Vérifiez que le fichier *dbo.emp.txt* se trouve dans le dossier de sortie.

1. Dans le portail Azure, dans la fenêtre du conteneur **adftutorial**, cliquez sur **Actualiser** pour afficher le dossier de sortie.
1. Sélectionnez `fromonprem` dans la liste des dossiers.
1. Vérifiez que le fichier nommé `dbo.emp.txt` s’affiche.

    ![Fichier de sortie](media/tutorial-hybrid-copy-powershell/fromonprem-file.png)


## <a name="next-steps"></a>Étapes suivantes
Dans cet exemple, le pipeline copie les données d’un emplacement vers un autre dans un stockage Blob Azure. Vous avez appris à :

> [!div class="checklist"]
> * Créer une fabrique de données.
> * Créez un runtime d’intégration auto-hébergé.
> * Créer des services liés SQL Server et au Stockage Azure.
> * Créer des jeux de données SQL Server et Blob Azure.
> * Créer un pipeline avec une activité de copie pour déplacer les données.
> * Démarrer une exécution de pipeline.
> * Surveiller l’exécution du pipeline.

Pour obtenir la liste des magasins de données pris en charge par Data Factory, consultez l’article [Magasins de données pris en charge](copy-activity-overview.md#supported-data-stores-and-formats).

Passez au tutoriel suivant pour découvrir comment copier des données en bloc d’une source vers une destination :

> [!div class="nextstepaction"]
>[Copier des données en bloc](tutorial-bulk-copy.md)