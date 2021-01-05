---
title: Copier des données de Stockage Blob Azure vers Azure SQL Database
description: Ce tutoriel fournit les instructions pas à pas permettant de copier des données à partir de Stockage Blob Azure vers Azure SQL Database.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: tutorial
ms.date: 11/08/2019
ms.author: jingwang
ms.openlocfilehash: b2293c0dd74903921abb58037afd8eb5db3659d9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85513271"
---
# <a name="copy-data-from-azure-blob-to-azure-sql-database-using-azure-data-factory"></a>Copier des données à partir d’un objet blob Azure vers Azure SQL Database à l’aide d’Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Dans ce tutoriel, vous créez un pipeline Azure Data Factory qui copie des données depuis le Stockage Blob Azure vers Azure SQL Database. Le modèle de configuration de ce didacticiel s’applique à la copie depuis un magasin de données de fichiers vers un magasin de données relationnelles. Pour obtenir la liste des magasins de données pris en charge en tant que sources et récepteurs, consultez [Magasins de données et formats pris en charge](copy-activity-overview.md#supported-data-stores-and-formats).

Dans ce didacticiel, vous allez effectuer les étapes suivantes :

> [!div class="checklist"]
> * Créer une fabrique de données.
> * Créer les services liés Stockage Azure et Azure SQL Database.
> * Créer des jeux de données Blob Azure et Azure SQL Database.
> * Créer un pipeline contenant une activité de copie.
> * Démarrer une exécution de pipeline.
> * Surveiller les exécutions de pipeline et d’activité.

Ce didacticiel utilise le kit .NET SDK. Vous pouvez utiliser d’autres mécanismes pour interagir avec Azure Data Factory. Consultez les exemples dans **Démarrages rapides**.

Si vous n’avez pas d’abonnement Azure, créez un [compte Azure gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

* *Compte Stockage Azure*. Vous utilisez le stockage blob comme magasins de données *source*. Si vous n’avez pas de compte de stockage Azure, consultez [Créer un compte de stockage universel](../storage/common/storage-account-create.md).
* *Azure SQL Database*. Vous utilisez la base de données en tant que magasin de données *récepteur*. Si vous n’avez pas de base de données dans Azure SQL Database, consultez [Créer une base de données dans Azure SQL Database](../azure-sql/database/single-database-create-quickstart.md).
* *Visual Studio*. La procédure pas à pas décrite dans cet article utilise Visual Studio 2019.
* *[SDK Azure pour .NET](/dotnet/azure/dotnet-tools)* .
* *Application Azure Active Directory*. Si vous n’avez pas d’application Azure Active Directory, consultez la section [Créer une application Azure Active Directory](../active-directory/develop/howto-create-service-principal-portal.md#register-an-application-with-azure-ad-and-create-a-service-principal) dans [Procédure : Utiliser le portail pour créer une application Azure AD](../active-directory/develop/howto-create-service-principal-portal.md). Copiez les valeurs suivantes pour les utiliser dans les étapes ultérieures : **ID d’application (client)** , **Clé d’authentification** et **ID de l’annuaire (locataire)** . Affectez l’application au rôle **Contributeur** en suivant les instructions données dans le même article.

### <a name="create-a-blob-and-a-sql-table"></a>Créer un objet blob et une table SQL

À présent, préparez votre objet blob Azure et Azure SQL Database au tutoriel en créant un objet blob source et une table SQL de récepteur.

#### <a name="create-a-source-blob"></a>Créer un objet blob source

Tout d’abord, créez un objet blob source en créant un conteneur et en y chargeant un fichier texte d’entrée :

1. Ouvrez le Bloc-notes. Copiez le texte suivant et enregistrez-le localement dans un fichier nommé *inputEmp.txt*.

    ```inputEmp.txt
    John|Doe
    Jane|Doe
    ```

2. Utilisez un outil comme [Explorateur Stockage Azure](https://azure.microsoft.com/features/storage-explorer/) pour créer le conteneur *adfv2tutorial* et charger le fichier *inputEmp.txt* sur ce dernier.

#### <a name="create-a-sink-sql-table"></a>Créer une table SQL de récepteur

Ensuite, créez une table SQL de récepteur.

1. Utilisez le script SQL suivant pour créer la table *dbo.emp* dans Azure SQL Database.

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

2. Autorisez les services Azure à accéder à SQL Database. Veillez à autoriser l’accès aux services Azure sur votre serveur pour que le service Data Factory puisse écrire des données sur SQL Database. Pour vérifier et activer ce paramètre, procédez comme suit :

    1. Accédez au [portail Azure](https://portal.azure.com) pour gérer votre serveur SQL. Recherchez et sélectionnez les **serveurs SQL**.

    2. Sélectionnez votre serveur.

    3. Sous le titre **Sécurité** du menu du serveur SQL, sélectionnez **Pare-feu et réseaux virtuels**.

    4. Dans la page **Pare-feu et réseaux virtuels**, sous **Autoriser les services et ressources Azure à accéder à ce serveur**, sélectionnez **ACTIVÉ**.

## <a name="create-a-visual-studio-project"></a>Créer un projet Visual Studio

À l’aide de Visual Studio, créez une application console C# .NET.

1. Ouvrez Visual Studio.
2. Dans la fenêtre **Démarrer**, sélectionnez **Créer un projet**.
3. Dans la fenêtre **Créer un projet**, choisissez la version C# de l’**application console (.NET Framework)** dans la liste des types de projets. Sélectionnez ensuite **Suivant**.
4. Dans la fenêtre **Configurer votre nouveau projet**, entrez le **nom de projet** *ADFv2Tutorial*. Pour **Emplacement**, accédez au répertoire dans lequel enregistrer le projet et/ou créez-le. Sélectionnez ensuite **Créer**. Le nouveau projet apparaît dans l’environnement de développement intégré (IDE) de Visual Studio.

## <a name="install-nuget-packages"></a>Installer les packages NuGet

Ensuite, installez les packages de bibliothèque nécessaires à l’aide du gestionnaire de package NuGet.

1. Dans la barre de menus, choisissez **Outils** > **Gestionnaire de package NuGet** > **Console du gestionnaire de package**.
2. Dans le volet **Console du Gestionnaire de package**, exécutez les commandes suivantes pour installer les packages. Pour plus d’informations sur le package NuGet Azure Data Factory, consultez [Microsoft.Azure.Management.DataFactory](https://www.nuget.org/packages/Microsoft.Azure.Management.DataFactory/).

    ```package manager console
    Install-Package Microsoft.Azure.Management.DataFactory
    Install-Package Microsoft.Azure.Management.ResourceManager -PreRelease
    Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory
    ```

## <a name="create-a-data-factory-client"></a>Créer un client de fabrique de données

Suivez ces étapes pour créer un client de fabrique de données.

1. Ouvrez *Program.cs*, puis remplacez les instructions `using` existantes par le code suivant pour ajouter des références aux espaces de noms.

    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using Microsoft.Rest;
    using Microsoft.Rest.Serialization;
    using Microsoft.Azure.Management.ResourceManager;
    using Microsoft.Azure.Management.DataFactory;
    using Microsoft.Azure.Management.DataFactory.Models;
    using Microsoft.IdentityModel.Clients.ActiveDirectory;
    ```

2. Ajoutez le code suivant à la méthode `Main` qui définit les variables. Remplacez les 14 espaces réservés par vos valeurs.

    Pour voir la liste des régions Azure dans lesquelles Data Factory est actuellement disponible, consultez [Disponibilité des produits par région](https://azure.microsoft.com/global-infrastructure/services/). Dans la liste déroulante **Produits**, choisissez **parcourir** > **Analytics** > **Data Factory**. Ensuite, dans la liste déroulante **Régions**, choisissez les régions qui vous intéressent. Une grille apparaît avec l’état de disponibilité des produits Data Factory pour les régions sélectionnées.

    > [!NOTE]
    > Les magasins de données, tels que Stockage Azure et Azure SQL Database, et les calculs, tels que HDInsight, que Data Factory utilise peuvent se trouver dans d’autres régions que celles que vous choisissez pour Data Factory.

    ```csharp
    // Set variables
    string tenantID = "<your tenant ID>";
    string applicationId = "<your application ID>";
    string authenticationKey = "<your authentication key for the application>";
    string subscriptionId = "<your subscription ID to create the factory>";
    string resourceGroup = "<your resource group to create the factory>";

    string region = "<location to create the data factory in, such as East US>";
    string dataFactoryName = "<name of data factory to create (must be globally unique)>";

    // Specify the source Azure Blob information
    string storageAccount = "<your storage account name to copy data>";
    string storageKey = "<your storage account key>";
    string inputBlobPath = "adfv2tutorial/";
    string inputBlobName = "inputEmp.txt";

    // Specify the sink Azure SQL Database information
    string azureSqlConnString =
        "Server=tcp:<your server name>.database.windows.net,1433;" +
        "Database=<your database name>;" +
        "User ID=<your username>@<your server name>;" +
        "Password=<your password>;" +
        "Trusted_Connection=False;Encrypt=True;Connection Timeout=30";
    string azureSqlTableName = "dbo.emp";

    string storageLinkedServiceName = "AzureStorageLinkedService";
    string sqlDbLinkedServiceName = "AzureSqlDbLinkedService";
    string blobDatasetName = "BlobDataset";
    string sqlDatasetName = "SqlDataset";
    string pipelineName = "Adfv2TutorialBlobToSqlCopy";
    ```

3. Ajoutez le code suivant à la méthode `Main` qui crée une instance de la classe `DataFactoryManagementClient`. Cet objet vous permet de créer une fabrique de données, un service lié, des jeux de données ainsi qu’un pipeline. Cet objet vous permet également de surveiller les détails de l’exécution du pipeline.

    ```csharp
    // Authenticate and create a data factory management client
    var context = new AuthenticationContext("https://login.windows.net/" + tenantID);
    ClientCredential cc = new ClientCredential(applicationId, authenticationKey);
    AuthenticationResult result = context.AcquireTokenAsync(
        "https://management.azure.com/", cc
    ).Result;
    ServiceClientCredentials cred = new TokenCredentials(result.AccessToken);
    var client = new DataFactoryManagementClient(cred) { SubscriptionId = subscriptionId };
    ```

## <a name="create-a-data-factory"></a>Créer une fabrique de données

Ajoutez le code suivant à la méthode `Main` qui crée une *fabrique de données*.

```csharp
// Create a data factory
Console.WriteLine("Creating a data factory " + dataFactoryName + "...");
Factory dataFactory = new Factory
{
    Location = region,
    Identity = new FactoryIdentity()
};

client.Factories.CreateOrUpdate(resourceGroup, dataFactoryName, dataFactory);
Console.WriteLine(
    SafeJsonConvert.SerializeObject(dataFactory, client.SerializationSettings)
);

while (
    client.Factories.Get(
        resourceGroup, dataFactoryName
    ).ProvisioningState == "PendingCreation"
)
{
    System.Threading.Thread.Sleep(1000);
}
```

## <a name="create-linked-services"></a>Créez des services liés

Dans ce tutoriel, vous créez deux services liés pour la source et le récepteur, respectivement.

### <a name="create-an-azure-storage-linked-service"></a>Créer un service lié Stockage Azure

Ajoutez le code suivant à la méthode `Main` qui crée un *service lié Stockage Azure*. Pour plus d’informations sur les propriétés prises en charge, consultez [Propriétés du service lié Objet blob Azure](connector-azure-blob-storage.md#linked-service-properties).

```csharp
// Create an Azure Storage linked service
Console.WriteLine("Creating linked service " + storageLinkedServiceName + "...");

LinkedServiceResource storageLinkedService = new LinkedServiceResource(
    new AzureStorageLinkedService
    {
        ConnectionString = new SecureString(
            "DefaultEndpointsProtocol=https;AccountName=" + storageAccount +
            ";AccountKey=" + storageKey
        )
    }
);

client.LinkedServices.CreateOrUpdate(
    resourceGroup, dataFactoryName, storageLinkedServiceName, storageLinkedService
);
Console.WriteLine(
    SafeJsonConvert.SerializeObject(storageLinkedService, client.SerializationSettings)
);
```

### <a name="create-an-azure-sql-database-linked-service"></a>Créer un service lié Azure SQL Database

Ajoutez le code suivant à la méthode `Main` qui crée un *service lié Azure SQL Database*. Pour plus d’informations sur les propriétés prises en charge, consultez [Propriétés du service lié Azure SQL Database](connector-azure-sql-database.md#linked-service-properties).

```csharp
// Create an Azure SQL Database linked service
Console.WriteLine("Creating linked service " + sqlDbLinkedServiceName + "...");

LinkedServiceResource sqlDbLinkedService = new LinkedServiceResource(
    new AzureSqlDatabaseLinkedService
    {
        ConnectionString = new SecureString(azureSqlConnString)
    }
);

client.LinkedServices.CreateOrUpdate(
    resourceGroup, dataFactoryName, sqlDbLinkedServiceName, sqlDbLinkedService
);
Console.WriteLine(
    SafeJsonConvert.SerializeObject(sqlDbLinkedService, client.SerializationSettings)
);
```

## <a name="create-datasets"></a>Créez les jeux de données

Dans cette section, vous créez deux jeux de données : un pour la source, l’autre pour le récepteur.

### <a name="create-a-dataset-for-source-azure-blob"></a>Créer un jeu de données pour l’objet blob Azure source

Ajoutez le code suivant à la méthode `Main` qui crée un *jeu de données d’objet blob Azure*. Pour plus d’informations sur les propriétés prises en charge, consultez [Propriétés du jeu de données d’objet blob Azure](connector-azure-blob-storage.md#dataset-properties).

Vous définissez un jeu de données qui représente les données sources dans l’objet blob Azure. Ce jeu de données d’objet blob fait référence au service lié Stockage Azure que vous avez créé à l’étape précédente et décrit :

- Emplacement de l’objet blob à partir duquel copier : `FolderPath` et `FileName`
- Le format d’objet blob qui indique comment analyser le contenu : `TextFormat` et ses paramètres (par exemple, délimiteur de colonne)
- La structure de données, comprenant les noms de colonne et les types de données qui dans ce cas sont mappés à la table SQL du récepteur

```csharp
// Create an Azure Blob dataset
Console.WriteLine("Creating dataset " + blobDatasetName + "...");
DatasetResource blobDataset = new DatasetResource(
    new AzureBlobDataset
    {
        LinkedServiceName = new LinkedServiceReference {
            ReferenceName = storageLinkedServiceName
        },
        FolderPath = inputBlobPath,
        FileName = inputBlobName,
        Format = new TextFormat { ColumnDelimiter = "|" },
        Structure = new List<DatasetDataElement>
        {
            new DatasetDataElement { Name = "FirstName", Type = "String" },
            new DatasetDataElement { Name = "LastName", Type = "String" }
        }
    }
);

client.Datasets.CreateOrUpdate(
    resourceGroup, dataFactoryName, blobDatasetName, blobDataset
);
Console.WriteLine(
    SafeJsonConvert.SerializeObject(blobDataset, client.SerializationSettings)
);
```

### <a name="create-a-dataset-for-sink-azure-sql-database"></a>Créer un jeu de données pour Azure SQL Database récepteur

Ajoutez le code suivant à la méthode `Main` qui crée un *jeu de données Azure SQL Database*. Pour plus d’informations sur les propriétés prises en charge, consultez [Propriétés du jeu de données Azure SQL Database](connector-azure-sql-database.md#dataset-properties).

Vous définissez un jeu de données qui représente les données du récepteur dans Azure SQL Database. Ce jeu de données fait référence au service lié Azure SQL Database que vous avez créé à l’étape précédente. Il spécifie également la table SQL qui contient les données copiées.

```csharp
// Create an Azure SQL Database dataset
Console.WriteLine("Creating dataset " + sqlDatasetName + "...");
DatasetResource sqlDataset = new DatasetResource(
    new AzureSqlTableDataset
    {
        LinkedServiceName = new LinkedServiceReference
        {
            ReferenceName = sqlDbLinkedServiceName
        },
        TableName = azureSqlTableName
    }
);

client.Datasets.CreateOrUpdate(
    resourceGroup, dataFactoryName, sqlDatasetName, sqlDataset
);
Console.WriteLine(
    SafeJsonConvert.SerializeObject(sqlDataset, client.SerializationSettings)
);
```

## <a name="create-a-pipeline"></a>Créer un pipeline

Ajoutez le code suivant à la méthode `Main` qui crée un *pipeline avec une activité de copie*. Dans ce tutoriel, ce pipeline contient une seule activité : `CopyActivity`, qui accepte le jeu de données d’objet blob en tant que source et le jeu de données SQL en tant que récepteur. Pour plus d’informations sur l’activité de copie, consultez [Activité de copie dans Azure Data Factory](copy-activity-overview.md).

```csharp
// Create a pipeline with copy activity
Console.WriteLine("Creating pipeline " + pipelineName + "...");
PipelineResource pipeline = new PipelineResource
{
    Activities = new List<Activity>
    {
        new CopyActivity
        {
            Name = "CopyFromBlobToSQL",
            Inputs = new List<DatasetReference>
            {
                new DatasetReference() { ReferenceName = blobDatasetName }
            },
            Outputs = new List<DatasetReference>
            {
                new DatasetReference { ReferenceName = sqlDatasetName }
            },
            Source = new BlobSource { },
            Sink = new SqlSink { }
        }
    }
};

client.Pipelines.CreateOrUpdate(resourceGroup, dataFactoryName, pipelineName, pipeline);
Console.WriteLine(
    SafeJsonConvert.SerializeObject(pipeline, client.SerializationSettings)
);
```

## <a name="create-a-pipeline-run"></a>Créer une exécution du pipeline

Ajoutez le code suivant à la méthode `Main` qui *déclenche une exécution du pipeline*.

```csharp
// Create a pipeline run
Console.WriteLine("Creating pipeline run...");
CreateRunResponse runResponse = client.Pipelines.CreateRunWithHttpMessagesAsync(
    resourceGroup, dataFactoryName, pipelineName
).Result.Body;
Console.WriteLine("Pipeline run ID: " + runResponse.RunId);
```

## <a name="monitor-a-pipeline-run"></a>Surveiller une exécution du pipeline

À présent, insérez le code pour vérifier les états de l’exécution du pipeline et obtenir des détails sur l’exécution de l’activité de copie.

1. Ajoutez le code suivant à la méthode `Main` afin de vérifier en permanence les états de l’exécution du pipeline jusqu’à la fin de la copie des données.

    ```csharp
    // Monitor the pipeline run
    Console.WriteLine("Checking pipeline run status...");
    PipelineRun pipelineRun;
    while (true)
    {
        pipelineRun = client.PipelineRuns.Get(
            resourceGroup, dataFactoryName, runResponse.RunId
        );
        Console.WriteLine("Status: " + pipelineRun.Status);
        if (pipelineRun.Status == "InProgress")
            System.Threading.Thread.Sleep(15000);
        else
            break;
    }
    ```

2. Ajoutez le code suivant à la méthode `Main` qui récupère les détails de l’exécution de l’activité de copie, tels que la taille des données lues ou écrites.

    ```csharp
    // Check the copy activity run details
    Console.WriteLine("Checking copy activity run details...");

    RunFilterParameters filterParams = new RunFilterParameters(
        DateTime.UtcNow.AddMinutes(-10), DateTime.UtcNow.AddMinutes(10)
    );

    ActivityRunsQueryResponse queryResponse = client.ActivityRuns.QueryByPipelineRun(
        resourceGroup, dataFactoryName, runResponse.RunId, filterParams
    );

    if (pipelineRun.Status == "Succeeded")
    {
        Console.WriteLine(queryResponse.Value.First().Output);
    }
    else
        Console.WriteLine(queryResponse.Value.First().Error);

    Console.WriteLine("\nPress any key to exit...");
    Console.ReadKey();
    ```

## <a name="run-the-code"></a>Exécuter le code

Générez l’application en choisissant **Générer** > **Générer la solution**. Ensuite, démarrez l’application en choisissant **Déboguer** > **Démarrer le débogage**, puis vérifiez l’exécution du pipeline.

La console affiche la progression de la création d’une fabrique de données, d’un service lié, de jeux de données, du pipeline et de l’exécution du pipeline. Elle vérifie ensuite l’état de l’exécution du pipeline. Patientez jusqu’à l’affichage des détails de l’exécution de l’activité de copie avec la taille des données lues/écrites. Ensuite, à l’aide d’outils comme SQL Server Management Studio (SSMS) ou Visual Studio, vous pouvez vous connecter à votre base de données Azure SQL Database de destination pour vérifier si la table de destination que vous avez spécifiée contient les données copiées.

### <a name="sample-output"></a>Exemple de sortie

```json
Creating a data factory AdfV2Tutorial...
{
  "identity": {
    "type": "SystemAssigned"
  },
  "location": "East US"
}
Creating linked service AzureStorageLinkedService...
{
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": {
        "type": "SecureString",
        "value": "DefaultEndpointsProtocol=https;AccountName=<accountName>;AccountKey=<accountKey>"
      }
    }
  }
}
Creating linked service AzureSqlDbLinkedService...
{
  "properties": {
    "type": "AzureSqlDatabase",
    "typeProperties": {
      "connectionString": {
        "type": "SecureString",
        "value": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
      }
    }
  }
}
Creating dataset BlobDataset...
{
  "properties": {
    "type": "AzureBlob",
    "typeProperties": {
      "folderPath": "adfv2tutorial/",
      "fileName": "inputEmp.txt",
      "format": {
        "type": "TextFormat",
        "columnDelimiter": "|"
      }
    },
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
    "linkedServiceName": {
      "type": "LinkedServiceReference",
      "referenceName": "AzureStorageLinkedService"
    }
  }
}
Creating dataset SqlDataset...
{
  "properties": {
    "type": "AzureSqlTable",
    "typeProperties": {
      "tableName": "dbo.emp"
    },
    "linkedServiceName": {
      "type": "LinkedServiceReference",
      "referenceName": "AzureSqlDbLinkedService"
    }
  }
}
Creating pipeline Adfv2TutorialBlobToSqlCopy...
{
  "properties": {
    "activities": [
      {
        "type": "Copy",
        "typeProperties": {
          "source": {
            "type": "BlobSource"
          },
          "sink": {
            "type": "SqlSink"
          }
        },
        "inputs": [
          {
            "type": "DatasetReference",
            "referenceName": "BlobDataset"
          }
        ],
        "outputs": [
          {
            "type": "DatasetReference",
            "referenceName": "SqlDataset"
          }
        ],
        "name": "CopyFromBlobToSQL"
      }
    ]
  }
}
Creating pipeline run...
Pipeline run ID: 1cd03653-88a0-4c90-aabc-ae12d843e252
Checking pipeline run status...
Status: InProgress
Status: InProgress
Status: Succeeded
Checking copy activity run details...
{
  "dataRead": 18,
  "dataWritten": 28,
  "rowsCopied": 2,
  "copyDuration": 2,
  "throughput": 0.01,
  "errors": [],
  "effectiveIntegrationRuntime": "DefaultIntegrationRuntime (East US)",
  "usedDataIntegrationUnits": 2,
  "billedDuration": 2
}

Press any key to exit...
```

## <a name="next-steps"></a>Étapes suivantes

Dans cet exemple, le pipeline copie les données d’un emplacement vers un autre dans un stockage Blob Azure. Vous avez appris à :

> [!div class="checklist"]
> * Créer une fabrique de données.
> * Créer les services liés Stockage Azure et Azure SQL Database.
> * Créer des jeux de données Blob Azure et Azure SQL Database.
> * Créer un pipeline contenant une activité de copie.
> * Démarrer une exécution de pipeline.
> * Surveiller les exécutions de pipeline et d’activité.

Passez au tutoriel suivant pour en savoir plus sur la copie des données locales vers le cloud :

> [!div class="nextstepaction"]
>[Copier des données locales vers le cloud](tutorial-hybrid-copy-powershell.md)
