---
title: Développement pour Azure Files avec .NET | Microsoft Docs
description: Découvrez comment développer des services et applications .NET qui utilisent Azure Files pour stocker les données.
author: roygara
ms.service: storage
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 10/02/2020
ms.author: rogarana
ms.subservice: files
ms.custom: devx-track-csharp
ms.openlocfilehash: e112060db4a44884d3094a939b03ff106ba72e65
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96492197"
---
# <a name="develop-for-azure-files-with-net"></a>Développer pour Azure Files avec .NET

[!INCLUDE [storage-selector-file-include](../../../includes/storage-selector-file-include.md)]

Découvrez les bases du développement d’applications .NET qui utilisent [Azure Files](storage-files-introduction.md) pour stocker des données. Cet article explique comment créer une application console simple pour effectuer les opérations suivantes avec .NET et Azure Files :

- Obtenir le contenu d’un fichier
- Définir la taille maximale, ou quota, pour un partage de fichiers.
- Créer une signature d’accès partagé (SAP) pour un fichier.
- Copier un fichier dans un autre fichier au sein du même compte de stockage
- Copier un fichier dans un objet blob au sein du même compte de stockage.
- Créer un instantané d’un partage de fichiers.
- Restaurer un fichier à partir d’un instantané de partage de fichiers.
- Utiliser Azure Storage Metrics pour la résolution des problèmes

Pour en savoir plus sur Azure Files, consultez [Qu’est-ce qu’Azure Files ?](storage-files-introduction.md).

[!INCLUDE [storage-check-out-samples-dotnet](../../../includes/storage-check-out-samples-dotnet.md)]

## <a name="understanding-the-net-apis"></a>Compréhension des API .NET

Azure Files fournit deux grandes approches pour les applications clientes : Server Message Block (SMB) et REST. Dans .NET, l’abstraction de ces approches est fournie par les API `System.IO` et `Azure.Storage.Files.Shares`.

API | Quand l’utiliser | Notes
----|-------------|------
[System.IO](/dotnet/api/system.io) | Votre application : <ul><li>Doit lire/écrire des fichiers à l’aide de SMB.</li><li>Est en cours d’exécution sur un périphérique qui a accès via le port 445 à votre compte Azure Files</li><li>N’a pas besoin de gérer les paramètres d’administration du partage de fichiers</li></ul> | Les E/S de fichiers implémentées avec Azure Files sur SMB sont généralement similaires aux E/S avec un partage de fichiers réseau ou un périphérique de stockage local. Pour découvrir une présentation de certaines fonctionnalités de .NET, notamment des E/S de fichiers, consultez le tutoriel [Application console](/dotnet/csharp/tutorials/console-teleprompter).
[Azure.Storage.Files.Shares](/dotnet/api/azure.storage.files.shares) | Votre application : <ul><li>Ne peut pas accéder à Azure Files à l’aide de SMB sur le port 445 en raison des contraintes de pare-feu ou d’ISP.</li><li>Nécessite une fonctionnalité d’administration, telle que la capacité de définir le quota d’un partage de fichier ou de créer une signature d’accès partagé</li></ul> | Cet article décrit l’utilisation de `Azure.Storage.Files.Shares` pour les E/S de fichiers utilisant REST plutôt que SMB et la gestion du partage de fichiers.

## <a name="create-the-console-application-and-obtain-the-assembly"></a>Création de l’application console et obtention de l’assembly

Vous pouvez utiliser la bibliothèque de client Azure Files dans n’importe quel type d’application .NET. Ces applications incluent les applications cloud Azure, web, de bureau et mobiles. Dans ce guide, nous allons créer une application console pour rester simple.

Dans Visual Studio, créez une application de console Windows. Les étapes suivantes vous montrent comment créer une application console dans Visual Studio 2019. Les étapes sont semblables pour d’autres versions de Visual Studio.

1. Démarrez Visual Studio et sélectionnez **Créer un projet**.
1. Dans **Créer un projet**, choisissez **Application console (.NET Framework)** pour C#, puis sélectionnez **Suivant**.
1. Dans **Configurer votre nouveau projet**, entrez un nom pour l’application, puis sélectionnez **Créer**.

Ajoutez tous les exemples de code de cet article à la classe `Program` dans le fichier *Program.cs*.

## <a name="use-nuget-to-install-the-required-packages"></a>Utiliser NuGet pour installer les packages requis

Reportez-vous à ces packages dans votre projet :

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

- [Bibliothèque Azure Core pour .NET](https://www.nuget.org/packages/Azure.Core/) : Ce package est l’implémentation du pipeline client Azure.
- [Bibliothèque de client Stockage Blob Azure pour .NET](https://www.nuget.org/packages/Azure.Storage.Blobs/): Ce package fournit un accès par programmation aux ressources d’objets blob de votre compte de stockage.
- [Bibliothèque de client Fichiers du Stockage Azure pour .NET](https://www.nuget.org/packages/Azure.Storage.Files.Shares/) : Ce package fournit un accès par programmation aux ressources de fichiers de votre compte de stockage.
- [Bibliothèque System Configuration Manager pour .NET](https://www.nuget.org/packages/System.Configuration.ConfigurationManager/) : Ce package fournit une classe qui stocke et récupère des valeurs dans un fichier de configuration.

Vous pouvez utiliser NuGet pour obtenir ces packages. Procédez comme suit :

1. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur le projet, puis choisissez **Gérer les packages NuGet**.
1. Dans le **Gestionnaire de package NuGet**, sélectionnez **Parcourir**. Ensuite, recherchez et choisissez **Azure.Core**, puis sélectionnez **Installer**.

   Cette étape installe le package et ses dépendances.

1. Recherchez et installez ces packages :

   - **Azure.Storage.Blobs**
   - **Azure.Storage.Files.Shares**
   - **System.Configuration.ConfigurationManager**

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

- [Bibliothèque commune Microsoft Azure Stockage pour .NET](https://www.nuget.org/packages/Microsoft.Azure.Storage.Common/) : ce package fournit un accès programmatique aux ressources communes de votre compte de stockage.
- [Bibliothèque Microsoft Azure Storage Blob pour .NET](https://www.nuget.org/packages/Microsoft.Azure.Storage.Blob/) : Ce package fournit un accès par programmation aux ressources d’objets blob de votre compte de stockage.
- [Bibliothèque Microsoft Azure Storage File pour .NET](https://www.nuget.org/packages/Microsoft.Azure.Storage.File/) : Ce package fournit un accès par programmation aux ressources de fichiers de votre compte de stockage.
- [Bibliothèque Microsoft Azure Configuration Manager pour .NET](https://www.nuget.org/packages/Microsoft.Azure.ConfigurationManager/) : Ce package fournit une classe pour l’analyse d’une chaîne de connexion dans un fichier de configuration, quel que soit l’emplacement d’exécution de votre application.

Vous pouvez utiliser NuGet pour obtenir ces packages. Procédez comme suit :

1. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur le projet, puis choisissez **Gérer les packages NuGet**.
1. Dans le **Gestionnaire de package NuGet**, sélectionnez **Parcourir**. Ensuite, recherchez et choisissez **Microsoft.Azure.Storage.Blob**, puis sélectionnez **Installer**.

   Cette étape installe le package et ses dépendances.
1. Recherchez et installez ces packages :

   - **Microsoft.Azure.Storage.Common**
   - **Microsoft.Azure.Storage.File**
   - **Microsoft.Azure.ConfigurationManager**

---

## <a name="save-your-storage-account-credentials-to-the-appconfig-file"></a>Enregistrer les informations d’identification de votre compte de stockage dans le fichier App.config

Enregistrez ensuite vos informations d’identification dans le fichier *App.config* de votre projet. Dans l’**Explorateur de solutions**, double-cliquez sur `App.config`, puis modifiez le fichier afin qu’il soit similaire à l’exemple suivant.

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

Remplacez `myaccount` par le nom de votre compte de stockage et `mykey` par la clé de votre compte de stockage.

:::code language="xml" source="~/azure-storage-snippets/files/howto/dotnet/dotnet-v12/app.config" highlight="5,6,7":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

Remplacez `myaccount` par le nom de votre compte de stockage et `StorageAccountKeyEndingIn==` par la clé de votre compte de stockage.

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <startup>
    <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.5" />
  </startup>
  <appSettings>
    <add key="StorageConnectionString"
      value="DefaultEndpointsProtocol=https;AccountName=myaccount;AccountKey=StorageAccountKeyEndingIn==" />
  </appSettings>
</configuration>
```

---

> [!NOTE]
> L’émulateur de stockage Azurite ne prend actuellement pas en charge Azure Files. Votre chaîne de connexion doit cibler un compte de stockage Azure dans le cloud pour fonctionner avec Azure Files.

## <a name="add-using-directives"></a>Ajouter des directives d’utilisation

Dans l’**Explorateur de solutions**, ouvrez le fichier *Program.cs*, puis ajoutez les directives using suivantes en haut du fichier.

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/files/howto/dotnet/dotnet-v12/FileShare.cs" id="snippet_UsingStatements":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

```csharp
using Microsoft.Azure; // Namespace for Azure Configuration Manager
using Microsoft.Azure.Storage; // Namespace for Storage Client Library
using Microsoft.Azure.Storage.Blob; // Namespace for Azure Blobs
using Microsoft.Azure.Storage.File; // Namespace for Azure Files
```

[!INCLUDE [storage-cloud-configuration-manager-include](../../../includes/storage-cloud-configuration-manager-include.md)]

---

## <a name="access-the-file-share-programmatically"></a>Accès au partage de fichiers par programmation

Dans le fichier *Program.cs*, ajoutez le code suivant pour accéder au partage de fichiers programmatiquement.

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

La méthode suivante crée un partage de fichiers, s’il n’en existe pas déjà un. La méthode commence par créer un objet [ShareClient](/dotnet/api/azure.storage.files.shares.shareclient) à partir d’une chaîne de connexion. L’exemple tente ensuite de télécharger un fichier que nous avons créé précédemment. Appelez cette méthode à partir de `Main()`.

:::code language="csharp" source="~/azure-storage-snippets/files/howto/dotnet/dotnet-v12/FileShare.cs" id="snippet_CreateShare":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

Ajoutez ensuite le contenu suivant à la méthode `Main()`, après le code indiqué ci-dessus, pour récupérer la chaîne de connexion. Ce code obtient une référence au fichier créé plus tôt et retourne son contenu.

```csharp
// Create a CloudFileClient object for credentialed access to Azure Files.
CloudFileClient fileClient = storageAccount.CreateCloudFileClient();

// Get a reference to the file share we created previously.
CloudFileShare share = fileClient.GetShareReference("logs");

// Ensure that the share exists.
if (share.Exists())
{
    // Get a reference to the root directory for the share.
    CloudFileDirectory rootDir = share.GetRootDirectoryReference();

    // Get a reference to the directory we created previously.
    CloudFileDirectory sampleDir = rootDir.GetDirectoryReference("CustomLogs");

    // Ensure that the directory exists.
    if (sampleDir.Exists())
    {
        // Get a reference to the file we created previously.
        CloudFile file = sampleDir.GetFileReference("Log1.txt");

        // Ensure that the file exists.
        if (file.Exists())
        {
            // Write the contents of the file to the console window.
            Console.WriteLine(file.DownloadTextAsync().Result);
        }
    }
}
```

Exécutez l’application console pour voir le résultat.

---

## <a name="set-the-maximum-size-for-a-file-share"></a>Définition de la taille maximale d’un partage de fichiers

À compter de la version 5.x de la bibliothèque de client Azure Files, vous pouvez définir le quota (taille maximale) d’un partage de fichiers. Vous pouvez également vérifier la quantité de données actuellement stockée sur le partage.

Le fait de définir le quota pour un partage limite la taille totale des fichiers stockés sur ce partage. Si la taille totale des fichiers sur le partage dépasse le quota, les clients ne peuvent pas augmenter la taille des fichiers existants. Les clients ne peuvent pas non plus créer de nouveaux fichiers, sauf si ces fichiers sont vides.

L’exemple ci-dessous illustre comment vérifier l’utilisation actuelle pour un partage et comment définir le quota pour le partage.

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/files/howto/dotnet/dotnet-v12/FileShare.cs" id="snippet_SetMaxShareSize":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

```csharp
// Parse the connection string for the storage account.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    Microsoft.Azure.CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create a CloudFileClient object for credentialed access to Azure Files.
CloudFileClient fileClient = storageAccount.CreateCloudFileClient();

// Get a reference to the file share we created previously.
CloudFileShare share = fileClient.GetShareReference("logs");

// Ensure that the share exists.
if (share.Exists())
{
    // Check current usage stats for the share.
    // Note that the ShareStats object is part of the protocol layer for the File service.
    Microsoft.Azure.Storage.File.Protocol.ShareStats stats = share.GetStats();
    Console.WriteLine("Current share usage: {0} GB", stats.Usage.ToString());

    // Specify the maximum size of the share, in GB.
    // This line sets the quota to be 10 GB greater than the current usage of the share.
    share.Properties.Quota = 10 + stats.Usage;
    share.SetProperties();

    // Now check the quota for the share. Call FetchAttributes() to populate the share's properties.
    share.FetchAttributes();
    Console.WriteLine("Current share quota: {0} GB", share.Properties.Quota);
}
```

---

### <a name="generate-a-shared-access-signature-for-a-file-or-file-share"></a>Génération d’une signature d’accès partagé pour un fichier ou partage de fichiers

À compter de la version 5.x de la bibliothèque de client Azure Files, vous pouvez générer une signature d’accès partagé (SAP) pour un partage de fichiers ou un fichier individuel.

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

L’exemple de méthode suivant retourne une signature d’accès partagé sur un fichier du partage spécifié.

:::code language="csharp" source="~/azure-storage-snippets/files/howto/dotnet/dotnet-v12/FileShare.cs" id="snippet_GetFileSasUri":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

Vous pouvez également créer une stratégie d’accès stockée sur un partage de fichiers pour gérer les signatures d’accès partagé. Nous vous recommandons de créer une stratégie d’accès stockée, car elle vous permet de révoquer la SAS si elle est compromise. L’exemple suivant crée une stratégie d’accès stockée sur un partage. Il utilise cette stratégie pour fournir les contraintes pour une SAP sur un fichier du partage.

```csharp
// Parse the connection string for the storage account.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    Microsoft.Azure.CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create a CloudFileClient object for credentialed access to Azure Files.
CloudFileClient fileClient = storageAccount.CreateCloudFileClient();

// Get a reference to the file share we created previously.
CloudFileShare share = fileClient.GetShareReference("logs");

// Ensure that the share exists.
if (share.Exists())
{
    string policyName = "sampleSharePolicy" + DateTime.UtcNow.Ticks;

    // Create a new stored access policy and define its constraints.
    SharedAccessFilePolicy sharedPolicy = new SharedAccessFilePolicy()
        {
            SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24),
            Permissions = SharedAccessFilePermissions.Read | SharedAccessFilePermissions.Write
        };

    // Get existing permissions for the share.
    FileSharePermissions permissions = share.GetPermissions();

    // Add the stored access policy to the share's policies. Note that each policy must have a unique name.
    permissions.SharedAccessPolicies.Add(policyName, sharedPolicy);
    share.SetPermissions(permissions);

    // Generate a SAS for a file in the share and associate this access policy with it.
    CloudFileDirectory rootDir = share.GetRootDirectoryReference();
    CloudFileDirectory sampleDir = rootDir.GetDirectoryReference("CustomLogs");
    CloudFile file = sampleDir.GetFileReference("Log1.txt");
    string sasToken = file.GetSharedAccessSignature(null, policyName);
    Uri fileSasUri = new Uri(file.StorageUri.PrimaryUri.ToString() + sasToken);

    // Create a new CloudFile object from the SAS, and write some text to the file.
    CloudFile fileSas = new CloudFile(fileSasUri);
    fileSas.UploadText("This write operation is authorized via SAS.");
    Console.WriteLine(fileSas.DownloadText());
}
```

---

Pour plus d’informations sur la création et l’utilisation de signatures d’accès partagé, consultez [Fonctionnement d’une signature d’accès partagé](../common/storage-sas-overview.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json#how-a-shared-access-signature-works).

## <a name="copy-files"></a>Copie des fichiers

À compter de la version 5.x de la bibliothèque de client Azure Files, vous pouvez copier un fichier dans un autre fichier, un fichier dans un objet blob ou un objet blob dans un fichier.

Vous pouvez également utiliser AzCopy pour copier un fichier dans un autre ou pour copier un objet blob dans un fichier, ou inversement. Consultez [Bien démarrer avec AzCopy](../common/storage-use-azcopy-v10.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json).

> [!NOTE]
> Si vous copiez un objet blob dans un fichier ou un fichier dans un objet blob, vous devez utiliser une signature d’accès partagé (SAP) pour autoriser l’accès à l’objet source, même si vous effectuez la copie dans le même compte de stockage.

### <a name="copy-a-file-to-another-file"></a>Copier un fichier dans un autre

L’exemple suivant copie un fichier dans un autre fichier au sein du même partage. Vous pouvez utiliser l’[Authentification par clé partagée](/rest/api/storageservices/authorize-with-shared-key) pour effectuer la copie car cette opération copie des fichiers dans le même compte de stockage.

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/files/howto/dotnet/dotnet-v12/FileShare.cs" id="snippet_CopyFile":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

```csharp
// Parse the connection string for the storage account.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    Microsoft.Azure.CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create a CloudFileClient object for credentialed access to Azure Files.
CloudFileClient fileClient = storageAccount.CreateCloudFileClient();

// Get a reference to the file share we created previously.
CloudFileShare share = fileClient.GetShareReference("logs");

// Ensure that the share exists.
if (share.Exists())
{
    // Get a reference to the root directory for the share.
    CloudFileDirectory rootDir = share.GetRootDirectoryReference();

    // Get a reference to the directory we created previously.
    CloudFileDirectory sampleDir = rootDir.GetDirectoryReference("CustomLogs");

    // Ensure that the directory exists.
    if (sampleDir.Exists())
    {
        // Get a reference to the file we created previously.
        CloudFile sourceFile = sampleDir.GetFileReference("Log1.txt");

        // Ensure that the source file exists.
        if (sourceFile.Exists())
        {
            // Get a reference to the destination file.
            CloudFile destFile = sampleDir.GetFileReference("Log1Copy.txt");

            // Start the copy operation.
            destFile.StartCopy(sourceFile);

            // Write the contents of the destination file to the console window.
            Console.WriteLine(destFile.DownloadText());
        }
    }
}
```

---

### <a name="copy-a-file-to-a-blob"></a>Copier un fichier dans un objet blob

L’exemple ci-dessous crée un fichier et le copie dans un objet blob au sein du même compte de stockage. L’exemple crée une SAP pour le fichier source, que le service utilise pour autoriser l’accès au fichier source pendant l’opération de copie.

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/files/howto/dotnet/dotnet-v12/FileShare.cs" id="snippet_CopyFileToBlob":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

```csharp
// Parse the connection string for the storage account.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    Microsoft.Azure.CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create a CloudFileClient object for credentialed access to Azure Files.
CloudFileClient fileClient = storageAccount.CreateCloudFileClient();

// Create a new file share, if it does not already exist.
CloudFileShare share = fileClient.GetShareReference("sample-share");
share.CreateIfNotExists();

// Create a new file in the root directory.
CloudFile sourceFile = share.GetRootDirectoryReference().GetFileReference("sample-file.txt");
sourceFile.UploadText("A sample file in the root directory.");

// Get a reference to the blob to which the file will be copied.
CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();
CloudBlobContainer container = blobClient.GetContainerReference("sample-container");
container.CreateIfNotExists();
CloudBlockBlob destBlob = container.GetBlockBlobReference("sample-blob.txt");

// Create a SAS for the file that's valid for 24 hours.
// Note that when you are copying a file to a blob, or a blob to a file, you must use a SAS
// to authorize access to the source object, even if you are copying within the same
// storage account.
string fileSas = sourceFile.GetSharedAccessSignature(new SharedAccessFilePolicy()
{
    // Only read permissions are required for the source file.
    Permissions = SharedAccessFilePermissions.Read,
    SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24)
});

// Construct the URI to the source file, including the SAS token.
Uri fileSasUri = new Uri(sourceFile.StorageUri.PrimaryUri.ToString() + fileSas);

// Copy the file to the blob.
destBlob.StartCopy(fileSasUri);

// Write the contents of the file to the console window.
Console.WriteLine("Source file contents: {0}", sourceFile.DownloadText());
Console.WriteLine("Destination blob contents: {0}", destBlob.DownloadText());
```

---

Vous pouvez copier un objet blob dans un fichier de la même façon. Si l’objet source est un objet blob, créez une SAP pour authentifier l’accès à cet objet blob pendant l’opération de copie.

## <a name="share-snapshots"></a>Instantanés de partage

À compter de la version 8.5 de la bibliothèque de client Azure Files, vous pouvez créer un instantané de partage. Vous pouvez également répertorier ou parcourir des instantanés de partage et les supprimer. Une fois créés, les instantanés de partage sont en lecture seule.

### <a name="create-share-snapshots"></a>Créer des instantanés de partage

L’exemple suivant permet de créer un instantané de partage de fichier.

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/files/howto/dotnet/dotnet-v12/FileShare.cs" id="snippet_CreateShareSnapshot":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

```csharp
storageAccount = CloudStorageAccount.Parse(ConnectionString); 
fClient = storageAccount.CreateCloudFileClient(); 
string baseShareName = "myazurefileshare"; 
CloudFileShare myShare = fClient.GetShareReference(baseShareName); 
var snapshotShare = myShare.Snapshot();

```

---

### <a name="list-share-snapshots"></a>Répertorier les instantanés de partage

L’exemple suivant liste les instantanés présents sur un partage.

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/files/howto/dotnet/dotnet-v12/FileShare.cs" id="snippet_ListShareSnapshots":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

```csharp
var shares = fClient.ListShares(baseShareName, ShareListingDetails.All);
```

---

### <a name="list-files-and-directories-within-share-snapshots"></a>Lister les fichiers et les répertoires figurant dans des instantanés de partage

L’exemple suivant parcourt les fichiers et les répertoires figurant dans des instantanés de partage.

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/files/howto/dotnet/dotnet-v12/FileShare.cs" id="snippet_ListSnapshotContents":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

```csharp
CloudFileShare mySnapshot = fClient.GetShareReference(baseShareName, snapshotTime); 
var rootDirectory = mySnapshot.GetRootDirectoryReference(); 
var items = rootDirectory.ListFilesAndDirectories();
```

---

### <a name="restore-file-shares-or-files-from-share-snapshots"></a>Restaurer des partages de fichiers ou des fichiers à partir d’instantanés de partage

Prendre un instantané d’un partage de fichiers vous permet de récupérer des fichiers ou l’intégralité du partage de fichiers.

Vous pouvez restaurer un fichier à partir d’un instantané de partage de fichiers en interrogeant les instantanés de partage d’un partage de fichiers. Vous pouvez ensuite récupérer un fichier qui appartient à un instantané de partage particulier. Utilisez cette version pour restaurer ou pour lire directement le fichier.

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/files/howto/dotnet/dotnet-v12/FileShare.cs" id="snippet_RestoreFileFromSnapshot":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

```csharp
CloudFileShare liveShare = fClient.GetShareReference(baseShareName);
var rootDirOfliveShare = liveShare.GetRootDirectoryReference();
var dirInliveShare = rootDirOfliveShare.GetDirectoryReference(dirName);
var fileInliveShare = dirInliveShare.GetFileReference(fileName);

CloudFileShare snapshot = fClient.GetShareReference(baseShareName, snapshotTime);
var rootDirOfSnapshot = snapshot.GetRootDirectoryReference();
var dirInSnapshot = rootDirOfSnapshot.GetDirectoryReference(dirName);
var fileInSnapshot = dir1InSnapshot.GetFileReference(fileName);

string sasContainerToken = string.Empty;
SharedAccessFilePolicy sasConstraints = new SharedAccessFilePolicy();
sasConstraints.SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24);
sasConstraints.Permissions = SharedAccessFilePermissions.Read;

//Generate the shared access signature on the container, setting the constraints directly on the signature.
sasContainerToken = fileInSnapshot.GetSharedAccessSignature(sasConstraints);

string sourceUri = (fileInSnapshot.Uri.ToString() + sasContainerToken + "&" + fileInSnapshot.SnapshotTime.ToString()); ;
fileInliveShare.StartCopyAsync(new Uri(sourceUri));
```

---

### <a name="delete-share-snapshots"></a>Supprimer des instantanés de partage

L’exemple suivant permet de supprimer un instantané de partage de fichier.

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/files/howto/dotnet/dotnet-v12/FileShare.cs" id="snippet_DeleteSnapshot":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

```csharp
CloudFileShare mySnapshot = fClient.GetShareReference(baseShareName, snapshotTime); mySnapshot.Delete(null, null, null);
```

---

## <a name="troubleshoot-azure-files-by-using-metrics"></a>Résoudre les problèmes liés à Azure Files à l’aide de métriques<a name="troubleshooting-azure-files-using-metrics"></a>

Azure Storage Analytics prend en charge les métriques pour Azure Files. Avec les données de métriques, vous pouvez suivre les demandes et diagnostiquer les problèmes.

Vous pouvez activer les métriques pour Azure Files par le biais du [Portail Azure](https://portal.azure.com). Vous pouvez également activer les métriques par programmation en appelant l’opération [Définition des propriétés du service Fichier](/rest/api/storageservices/set-file-service-properties) avec l’API REST ou l’un de ses équivalents dans la bibliothèque de client Azure Files.

L’exemple de code suivant montre comment utiliser la bibliothèque de client .NET afin d’activer les métriques pour Azure Files.

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/files/howto/dotnet/dotnet-v12/FileShare.cs" id="snippet_UseMetrics":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

Commencez par ajouter les directives `using` suivantes à votre fichier *Program.cs*, en plus de celles que vous avez ajoutées ci-dessus :

```csharp
using Microsoft.Azure.Storage.File.Protocol;
using Microsoft.Azure.Storage.Shared.Protocol;
```

Bien qu’Azure Blobs, Azure Tables et Azure Queues utilisent le type `ServiceProperties` partagé dans l’espace de noms `Microsoft.Azure.Storage.Shared.Protocol`, Azure Files utilise son propre type (`FileServiceProperties`) dans l’espace de noms `Microsoft.Azure.Storage.File.Protocol`. Pour pouvoir compiler le code suivant, vous devez cependant référencer les deux espaces de noms à partir de votre code.

```csharp
// Parse your storage connection string from your application's configuration file.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        Microsoft.Azure.CloudConfigurationManager.GetSetting("StorageConnectionString"));
// Create the File service client.
CloudFileClient fileClient = storageAccount.CreateCloudFileClient();

// Set metrics properties for File service.
// Note that the File service currently uses its own service properties type,
// available in the Microsoft.Azure.Storage.File.Protocol namespace.
fileClient.SetServiceProperties(new FileServiceProperties()
{
    // Set hour metrics
    HourMetrics = new MetricsProperties()
    {
        MetricsLevel = MetricsLevel.ServiceAndApi,
        RetentionDays = 14,
        Version = "1.0"
    },
    // Set minute metrics
    MinuteMetrics = new MetricsProperties()
    {
        MetricsLevel = MetricsLevel.ServiceAndApi,
        RetentionDays = 7,
        Version = "1.0"
    }
});

// Read the metrics properties we just set.
FileServiceProperties serviceProperties = fileClient.GetServiceProperties();
Console.WriteLine("Hour metrics:");
Console.WriteLine(serviceProperties.HourMetrics.MetricsLevel);
Console.WriteLine(serviceProperties.HourMetrics.RetentionDays);
Console.WriteLine(serviceProperties.HourMetrics.Version);
Console.WriteLine();
Console.WriteLine("Minute metrics:");
Console.WriteLine(serviceProperties.MinuteMetrics.MetricsLevel);
Console.WriteLine(serviceProperties.MinuteMetrics.RetentionDays);
Console.WriteLine(serviceProperties.MinuteMetrics.Version);
```

---

Si vous rencontrez des problèmes, consultez [Résoudre les problèmes liés à Azure Files sous Windows](storage-troubleshoot-windows-file-connection-problems.md).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur Azure Files, consultez les ressources suivantes :

### <a name="conceptual-articles-and-videos"></a>Vidéos et articles conceptuels

- [Azure Files : un système de fichiers SMB cloud transparent pour Windows et Linux](https://azure.microsoft.com/documentation/videos/azurecon-2015-azure-files-storage-a-frictionless-cloud-smb-file-system-for-windows-and-linux/)
- [Utiliser Azure Files avec Linux](storage-how-to-use-files-linux.md)

### <a name="tooling-support-for-file-storage"></a>Outils pris en charge pour le stockage de fichiers

- [Bien démarrer avec AzCopy](../common/storage-use-azcopy-v10.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json)
- [Résoudre les problèmes liés à Azure Files sous Windows](./storage-troubleshoot-windows-file-connection-problems.md)

### <a name="reference"></a>Informations de référence

- [API de stockage Azure pour .NET](/dotnet/api/overview/azure/storage)
- [API REST du service de fichiers](/rest/api/storageservices/File-Service-REST-API)