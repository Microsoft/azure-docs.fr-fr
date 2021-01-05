---
title: Transférer des données avec la bibliothèque de déplacement des données pour .NET
titleSuffix: Azure Storage
description: Utilisez la bibliothèque de déplacement des données pour déplacer ou copier des données vers ou à partir du contenu d’objets blob et de fichiers. Copiez des données vers Azure Storage à partir de fichiers locaux ou copiez des données dans ou entre des comptes de stockage. Migrez facilement vos données vers Azure Storage.
services: storage
author: tamram
ms.service: storage
ms.devlang: dotnet
ms.topic: how-to
ms.date: 06/16/2020
ms.author: tamram
ms.subservice: common
ms.custom: devx-track-csharp
ms.openlocfilehash: 8ec35a651d4d3ef9e0877463329a654bc7491f4c
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96023412"
---
# <a name="transfer-data-with-the-data-movement-library"></a>Transférer des données avec la bibliothèque de déplacement de données

La bibliothèque de déplacement des données Stockage Azure est une bibliothèque multiplateforme open source conçue pour charger, télécharger et copier des objets blob et des fichiers avec des performances élevées. La bibliothèque de déplacement des données fournit des méthodes utiles qui ne sont pas disponibles dans la bibliothèque de client Stockage Azure pour .NET. Ces méthodes offrent la capacité à définir le nombre d’opérations parallèles, à suivre la progression des transferts, à reprendre facilement un transfert annulé et bien plus encore.

Cette bibliothèque utilise également .NET Core, ce qui signifie que vous pouvez l’utiliser pour créer des applications .NET pour Windows, Linux et macOS. Pour en savoir plus sur .NET Core, consultez la [Documentation .NET Core](https://dotnet.github.io/). Cette bibliothèque fonctionne également pour les applications .NET Framework classiques pour Windows.

Ce document montre comment créer une application de console .NET Core qui s’exécute sous Windows, Linux et macOS et effectue les scénarios suivants :

- charger des fichiers et des répertoires vers le Stockage Blob ;
- définir le nombre d’opérations parallèles lors du transfert de données ;
- suivre la progression du transfert de données ;
- reprendre les transferts de données annulés ;
- copier des fichiers de l’URL vers le Stockage Blob ;
- copier d’un Stockage Blob à un autre.

## <a name="prerequisites"></a>Conditions préalables requises

- [Visual Studio Code](https://code.visualstudio.com/)
- Un [compte de stockage Azure](storage-account-create.md)

## <a name="setup"></a>Programme d’installation

1. Consultez le [Guide d’installation de .NET Core](https://www.microsoft.com/net/core) pour installer .NET Core. Lorsque vous sélectionnez votre environnement, choisissez l’option de ligne de commande.
2. En ligne de commande, créez un répertoire pour votre projet. Accédez à ce répertoire, puis tapez `dotnet new console -o <sample-project-name>` pour créer un projet de console C#.
3. Ouvrez ce répertoire dans Visual Studio Code. Vous pouvez effectuer rapidement cette étape en tapant `code .` en ligne de commande dans Windows.
4. Installez [l’extension C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp) à partir de la Place de marché Visual Studio. Redémarrez Visual Studio Code.
5. À ce stade, vous devez voir deux invites. L’une permet d’ajouter les « composants requis pour générer et déboguer ». Cliquez sur « Oui ». L’autre invite permet de restaurer les dépendances non résolues. Cliquez sur « Restaurer ».
6. Modifiez `launch.json` sous `.vscode` pour utiliser un terminal externe comme console. Ce paramètre doit alors être `"console": "externalTerminal"`.
7. Visual Studio Code vous permet de déboguer les applications .NET Core. Appuyez sur `F5` pour exécuter votre application et vérifier que votre configuration fonctionne. « Hello World! » doit s’imprimer dans la console.

## <a name="add-the-data-movement-library-to-your-project"></a>Ajouter la bibliothèque de déplacement des données à votre projet

1. Ajoutez la dernière version de la bibliothèque de déplacement des données à la section `dependencies` de votre fichier `<project-name>.csproj`. Au moment de la rédaction de cette page, il s’agit de la version `"Microsoft.Azure.Storage.DataMovement": "0.6.2"`.
2. Une invite doit s’afficher pour restaurer votre projet. Cliquez sur le bouton « Restaurer ». Vous pouvez également restaurer votre projet en ligne de commande en tapant la commande `dotnet restore` à la racine du répertoire de votre projet.

Modifiez `<project-name>.csproj` :

```xml
<Project Sdk="Microsoft.NET.Sdk">
    <PropertyGroup>
        <OutputType>Exe</OutputType>
        <TargetFramework>netcoreapp2.0</TargetFramework>
    </PropertyGroup>
    <ItemGroup>
        <PackageReference Include="Microsoft.Azure.Storage.DataMovement" Version="0.6.2" />
        </ItemGroup>
    </Project>
```

## <a name="set-up-the-skeleton-of-your-application"></a>Définir le squelette de votre application

La première chose à faire est de définir le « squelette » de code de votre application. Ce code nous demande une clé et un nom de compte de Stockage et utilise ces informations d’identification pour créer un objet `CloudStorageAccount`. Cet objet est utilisé pour interagir avec notre compte de Stockage dans tous les scénarios de transfert. Le code nous invite également à choisir le type d’opération de transfert que nous souhaitons exécuter.

Modifiez `Program.cs` :

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;
using System.Diagnostics;
using Microsoft.Azure.Storage;
using Microsoft.Azure.Storage.Blob;
using Microsoft.Azure.Storage.DataMovement;

namespace DMLibSample
{
    public class Program
    {
        public static void Main()
        {
            Console.WriteLine("Enter Storage account name:");
            string accountName = Console.ReadLine();

            Console.WriteLine("\nEnter Storage account key:");
            string accountKey = Console.ReadLine();

            string storageConnectionString = "DefaultEndpointsProtocol=https;AccountName=" + accountName + ";AccountKey=" + accountKey;
            CloudStorageAccount account = CloudStorageAccount.Parse(storageConnectionString);

            ExecuteChoice(account);
        }

        public static void ExecuteChoice(CloudStorageAccount account)
        {
            Console.WriteLine("\nWhat type of transfer would you like to execute?\n1. Local file --> Azure Blob\n2. Local directory --> Azure Blob directory\n3. URL (e.g. Amazon S3 file) --> Azure Blob\n4. Azure Blob --> Azure Blob");
            int choice = int.Parse(Console.ReadLine());

            if(choice == 1)
            {
                TransferLocalFileToAzureBlob(account).Wait();
            }
            else if(choice == 2)
            {
                TransferLocalDirectoryToAzureBlobDirectory(account).Wait();
            }
            else if(choice == 3)
            {
                TransferUrlToAzureBlob(account).Wait();
            }
            else if(choice == 4)
            {
                TransferAzureBlobToAzureBlob(account).Wait();
            }
        }

        public static async Task TransferLocalFileToAzureBlob(CloudStorageAccount account)
        {

        }

        public static async Task TransferLocalDirectoryToAzureBlobDirectory(CloudStorageAccount account)
        {

        }

        public static async Task TransferUrlToAzureBlob(CloudStorageAccount account)
        {

        }

        public static async Task TransferAzureBlobToAzureBlob(CloudStorageAccount account)
        {

        }
    }
}
```

## <a name="upload-a-local-file-to-a-blob"></a>Charger un fichier local vers un objet blob

Ajoutez les méthodes `GetSourcePath` et `GetBlob` à `Program.cs` :

```csharp
public static string GetSourcePath()
{
    Console.WriteLine("\nProvide path for source:");
    string sourcePath = Console.ReadLine();

    return sourcePath;
}

public static CloudBlockBlob GetBlob(CloudStorageAccount account)
{
    CloudBlobClient blobClient = account.CreateCloudBlobClient();

    Console.WriteLine("\nProvide name of Blob container:");
    string containerName = Console.ReadLine();
    CloudBlobContainer container = blobClient.GetContainerReference(containerName);
    container.CreateIfNotExistsAsync().Wait();

    Console.WriteLine("\nProvide name of new Blob:");
    string blobName = Console.ReadLine();
    CloudBlockBlob blob = container.GetBlockBlobReference(blobName);

    return blob;
}
```

Modifiez la méthode `TransferLocalFileToAzureBlob` :

```csharp
public static async Task TransferLocalFileToAzureBlob(CloudStorageAccount account)
{
    string localFilePath = GetSourcePath();
    CloudBlockBlob blob = GetBlob(account);
    Console.WriteLine("\nTransfer started...");
    await TransferManager.UploadAsync(localFilePath, blob);
    Console.WriteLine("\nTransfer operation complete.");
    ExecuteChoice(account);
}
```

Ce code nous demande le chemin d’accès à un fichier local, le nom d’un conteneur nouveau ou existant et le nom d’un nouvel objet blob. La méthode `TransferManager.UploadAsync` effectue le chargement suivant ces informations.

Appuyez sur `F5` pour exécuter votre application. Vous pouvez vérifier que le chargement a bien été effectué en affichant votre compte de Stockage avec [l’Explorateur de Stockage Microsoft Azure](https://storageexplorer.com/).

## <a name="set-the-number-of-parallel-operations"></a>Définir le nombre d’opérations parallèles

L’une des fonctionnalités offertes par la bibliothèque de déplacement des données consiste à définir le nombre d’opérations parallèles pour augmenter le débit du transfert de données. Par défaut, la bibliothèque de déplacement des données fixe le nombre d’opérations parallèles à 8 * le nombre de cœurs sur votre ordinateur.

Remarque : un grand nombre d’opérations parallèles dans un environnement à faible bande passante peut surcharger la connexion réseau et en réalité entraver le bon déroulement des opérations. Vous devrez faire des essais avec ce paramètre pour déterminer ce qui fonctionne le mieux en fonction de votre bande passante réseau disponible.

Nous allons ajouter du code pour pouvoir définir le nombre d’opérations parallèles. Ajoutons du code afin de chronométrer la durée du transfert.

Ajoutez une méthode `SetNumberOfParallelOperations` à `Program.cs` :

```csharp
public static void SetNumberOfParallelOperations()
{
    Console.WriteLine("\nHow many parallel operations would you like to use?");
    string parallelOperations = Console.ReadLine();
    TransferManager.Configurations.ParallelOperations = int.Parse(parallelOperations);
}
```

Modifiez la méthode `ExecuteChoice` de sorte qu’elle utilise `SetNumberOfParallelOperations` :

```csharp
public static void ExecuteChoice(CloudStorageAccount account)
{
    Console.WriteLine("\nWhat type of transfer would you like to execute?\n1. Local file --> Azure Blob\n2. Local directory --> Azure Blob directory\n3. URL (e.g. Amazon S3 file) --> Azure Blob\n4. Azure Blob --> Azure Blob");
    int choice = int.Parse(Console.ReadLine());

    SetNumberOfParallelOperations();

    if(choice == 1)
    {
        TransferLocalFileToAzureBlob(account).Wait();
    }
    else if(choice == 2)
    {
        TransferLocalDirectoryToAzureBlobDirectory(account).Wait();
    }
    else if(choice == 3)
    {
        TransferUrlToAzureBlob(account).Wait();
    }
    else if(choice == 4)
    {
        TransferAzureBlobToAzureBlob(account).Wait();
    }
}
```

Modifiez la méthode `TransferLocalFileToAzureBlob` de sorte qu’elle utilise un minuteur :

```csharp
public static async Task TransferLocalFileToAzureBlob(CloudStorageAccount account)
{
    string localFilePath = GetSourcePath();
    CloudBlockBlob blob = GetBlob(account);
    Console.WriteLine("\nTransfer started...");
    Stopwatch stopWatch = Stopwatch.StartNew();
    await TransferManager.UploadAsync(localFilePath, blob);
    stopWatch.Stop();
    Console.WriteLine("\nTransfer operation completed in " + stopWatch.Elapsed.TotalSeconds + " seconds.");
    ExecuteChoice(account);
}
```

## <a name="track-transfer-progress"></a>Suivre la progression des transferts

Il est très utile de connaître la durée du transfert des données. Toutefois, il serait encore mieux de pouvoir voir la progression du transfert *pendant* l’opération de transfert. Pour appliquer ce scénario, nous devons créer un objet `TransferContext`. L’objet `TransferContext` se présente sous deux formes : `SingleTransferContext` et `DirectoryTransferContext`. La première permet de transférer un seul fichier et la seconde concerne le transfert d’un répertoire de fichiers.

Ajoutez les méthodes `GetSingleTransferContext` et `GetDirectoryTransferContext` à `Program.cs` :

```csharp
public static SingleTransferContext GetSingleTransferContext(TransferCheckpoint checkpoint)
{
    SingleTransferContext context = new SingleTransferContext(checkpoint);

    context.ProgressHandler = new Progress<TransferStatus>((progress) =>
    {
        Console.Write("\rBytes transferred: {0}", progress.BytesTransferred );
    });

    return context;
}

public static DirectoryTransferContext GetDirectoryTransferContext(TransferCheckpoint checkpoint)
{
    DirectoryTransferContext context = new DirectoryTransferContext(checkpoint);

    context.ProgressHandler = new Progress<TransferStatus>((progress) =>
    {
        Console.Write("\rBytes transferred: {0}", progress.BytesTransferred );
    });

    return context;
}
```

Modifiez la méthode `TransferLocalFileToAzureBlob` de sorte qu’elle utilise `GetSingleTransferContext` :

```csharp
public static async Task TransferLocalFileToAzureBlob(CloudStorageAccount account)
{
    string localFilePath = GetSourcePath();
    CloudBlockBlob blob = GetBlob(account);
    TransferCheckpoint checkpoint = null;
    SingleTransferContext context = GetSingleTransferContext(checkpoint);
    Console.WriteLine("\nTransfer started...\n");
    Stopwatch stopWatch = Stopwatch.StartNew();
    await TransferManager.UploadAsync(localFilePath, blob, null, context);
    stopWatch.Stop();
    Console.WriteLine("\nTransfer operation completed in " + stopWatch.Elapsed.TotalSeconds + " seconds.");
    ExecuteChoice(account);
}
```

## <a name="resume-a-canceled-transfer"></a>Reprendre un transfert annulé

Une autre fonctionnalité pratique offerte par la bibliothèque de déplacement des données est de pouvoir reprendre un transfert annulé. Nous allons ajouter du code qui nous permettra d’annuler temporairement le transfert en tapant `c`, puis de le reprendre trois secondes plus tard.

Modifiez `TransferLocalFileToAzureBlob` :

```csharp
public static async Task TransferLocalFileToAzureBlob(CloudStorageAccount account)
{
    string localFilePath = GetSourcePath();
    CloudBlockBlob blob = GetBlob(account);
    TransferCheckpoint checkpoint = null;
    SingleTransferContext context = GetSingleTransferContext(checkpoint);
    CancellationTokenSource cancellationSource = new CancellationTokenSource();
    Console.WriteLine("\nTransfer started...\nPress 'c' to temporarily cancel your transfer...\n");

    Stopwatch stopWatch = Stopwatch.StartNew();
    Task task;
    ConsoleKeyInfo keyinfo;
    try
    {
        task = TransferManager.UploadAsync(localFilePath, blob, null, context, cancellationSource.Token);
        while(!task.IsCompleted)
        {
            if(Console.KeyAvailable)
            {
                keyinfo = Console.ReadKey(true);
                if(keyinfo.Key == ConsoleKey.C)
                {
                    cancellationSource.Cancel();
                }
            }
        }
        await task;
    }
    catch(Exception e)
    {
        Console.WriteLine("\nThe transfer is canceled: {0}", e.Message);
    }

    if(cancellationSource.IsCancellationRequested)
    {
        Console.WriteLine("\nTransfer will resume in 3 seconds...");
        Thread.Sleep(3000);
        checkpoint = context.LastCheckpoint;
        context = GetSingleTransferContext(checkpoint);
        Console.WriteLine("\nResuming transfer...\n");
        await TransferManager.UploadAsync(localFilePath, blob, null, context);
    }

    stopWatch.Stop();
    Console.WriteLine("\nTransfer operation completed in " + stopWatch.Elapsed.TotalSeconds + " seconds.");
    ExecuteChoice(account);
}
```

Jusqu'à présent, notre `checkpoint` a toujours eu la valeur `null`. Maintenant, si nous annulons le transfert, nous récupérons le dernier point de contrôle de notre transfert, puis nous utilisons ce nouveau point de contrôle dans notre contexte de transfert.

## <a name="transfer-a-local-directory-to-blob-storage"></a>Transférer un répertoire local vers le stockage Blob

Il serait décevant que la bibliothèque de déplacement des données ne puisse transférer qu’un fichier à la fois. Heureusement, ce n’est pas le cas. La bibliothèque de déplacement des données offre la capacité à transférer un répertoire de fichiers et tous ses sous-répertoires. Nous allons ajouter du code pour pouvoir effectuer cette opération.

Tout d’abord, ajoutez la méthode `GetBlobDirectory` à `Program.cs` :

```csharp
public static CloudBlobDirectory GetBlobDirectory(CloudStorageAccount account)
{
    CloudBlobClient blobClient = account.CreateCloudBlobClient();

    Console.WriteLine("\nProvide name of Blob container. This can be a new or existing Blob container:");
    string containerName = Console.ReadLine();
    CloudBlobContainer container = blobClient.GetContainerReference(containerName);
    container.CreateIfNotExistsAsync().Wait();

    CloudBlobDirectory blobDirectory = container.GetDirectoryReference("");

    return blobDirectory;
}
```

Ensuite, modifiez `TransferLocalDirectoryToAzureBlobDirectory` :

```csharp
public static async Task TransferLocalDirectoryToAzureBlobDirectory(CloudStorageAccount account)
{
    string localDirectoryPath = GetSourcePath();
    CloudBlobDirectory blobDirectory = GetBlobDirectory(account);
    TransferCheckpoint checkpoint = null;
    DirectoryTransferContext context = GetDirectoryTransferContext(checkpoint);
    CancellationTokenSource cancellationSource = new CancellationTokenSource();
    Console.WriteLine("\nTransfer started...\nPress 'c' to temporarily cancel your transfer...\n");

    Stopwatch stopWatch = Stopwatch.StartNew();
    Task task;
    ConsoleKeyInfo keyinfo;
    UploadDirectoryOptions options = new UploadDirectoryOptions()
    {
        Recursive = true
    };

    try
    {
        task = TransferManager.UploadDirectoryAsync(localDirectoryPath, blobDirectory, options, context, cancellationSource.Token);
        while(!task.IsCompleted)
        {
            if(Console.KeyAvailable)
            {
                keyinfo = Console.ReadKey(true);
                if(keyinfo.Key == ConsoleKey.C)
                {
                    cancellationSource.Cancel();
                }
            }
        }
        await task;
    }
    catch(Exception e)
    {
        Console.WriteLine("\nThe transfer is canceled: {0}", e.Message);
    }

    if(cancellationSource.IsCancellationRequested)
    {
        Console.WriteLine("\nTransfer will resume in 3 seconds...");
        Thread.Sleep(3000);
        checkpoint = context.LastCheckpoint;
        context = GetDirectoryTransferContext(checkpoint);
        Console.WriteLine("\nResuming transfer...\n");
        await TransferManager.UploadDirectoryAsync(localDirectoryPath, blobDirectory, options, context);
    }

    stopWatch.Stop();
    Console.WriteLine("\nTransfer operation completed in " + stopWatch.Elapsed.TotalSeconds + " seconds.");
    ExecuteChoice(account);
}
```

Il existe quelques différences entre cette méthode et la méthode de chargement d’un seul fichier. Nous utilisons maintenant `TransferManager.UploadDirectoryAsync` et la méthode `getDirectoryTransferContext` créée précédemment. En outre, nous fournissons à présent une valeur `options` à notre opération de chargement, ce qui nous permet d’indiquer que nous souhaitons inclure les sous-répertoires à notre chargement.

## <a name="copy-a-file-from-url-to-a-blob"></a>Copier un fichier d’une URL vers un objet blob

À présent, nous allons ajouter du code permettant de copier un fichier à partir d’une URL vers un objet blob Azure.

Modifiez `TransferUrlToAzureBlob` :

```csharp
public static async Task TransferUrlToAzureBlob(CloudStorageAccount account)
{
    Uri uri = new Uri(GetSourcePath());
    CloudBlockBlob blob = GetBlob(account);
    TransferCheckpoint checkpoint = null;
    SingleTransferContext context = GetSingleTransferContext(checkpoint);
    CancellationTokenSource cancellationSource = new CancellationTokenSource();
    Console.WriteLine("\nTransfer started...\nPress 'c' to temporarily cancel your transfer...\n");

    Stopwatch stopWatch = Stopwatch.StartNew();
    Task task;
    ConsoleKeyInfo keyinfo;
    try
    {
        task = TransferManager.CopyAsync(uri, blob, true, null, context, cancellationSource.Token);
        while(!task.IsCompleted)
        {
            if(Console.KeyAvailable)
            {
                keyinfo = Console.ReadKey(true);
                if(keyinfo.Key == ConsoleKey.C)
                {
                    cancellationSource.Cancel();
                }
            }
        }
        await task;
    }
    catch(Exception e)
    {
        Console.WriteLine("\nThe transfer is canceled: {0}", e.Message);
    }

    if(cancellationSource.IsCancellationRequested)
    {
        Console.WriteLine("\nTransfer will resume in 3 seconds...");
        Thread.Sleep(3000);
        checkpoint = context.LastCheckpoint;
        context = GetSingleTransferContext(checkpoint);
        Console.WriteLine("\nResuming transfer...\n");
        await TransferManager.CopyAsync(uri, blob, true, null, context, cancellationSource.Token);
    }

    stopWatch.Stop();
    Console.WriteLine("\nTransfer operation completed in " + stopWatch.Elapsed.TotalSeconds + " seconds.");
    ExecuteChoice(account);
}
```

L’un des cas d’utilisation les plus importants de cette fonctionnalité est le déplacement de données d’un autre service cloud (par exemple, AWS) vers Azure. Tant que vous disposez d’une URL qui vous permet d’accéder à la ressource, vous pouvez facilement déplacer cette ressource dans des objets blob Azure à l’aide de la méthode `TransferManager.CopyAsync`. Cette méthode introduit également un nouveau paramètre booléen. La valeur `true` de ce paramètre indique que nous souhaitons faire une copie asynchrone côté serveur. La valeur `false` de ce paramètre indique une copie synchrone, ce qui signifie que la ressource est d’abord téléchargée sur notre ordinateur local, puis chargée vers le Stockage Blob Azure. Toutefois, la copie synchrone n’est actuellement disponible que pour copier d’une ressource de Stockage Azure vers une autre.

## <a name="copy-a-blob"></a>Copier un objet blob

Une autre fonctionnalité offerte uniquement par la bibliothèque de déplacement des données est la capacité à copier d’une ressource de Stockage Azure vers une autre.

Modifiez `TransferAzureBlobToAzureBlob` :

```csharp
public static async Task TransferAzureBlobToAzureBlob(CloudStorageAccount account)
{
    CloudBlockBlob sourceBlob = GetBlob(account);
    CloudBlockBlob destinationBlob = GetBlob(account);
    TransferCheckpoint checkpoint = null;
    SingleTransferContext context = GetSingleTransferContext(checkpoint);
    CancellationTokenSource cancellationSource = new CancellationTokenSource();
    Console.WriteLine("\nTransfer started...\nPress 'c' to temporarily cancel your transfer...\n");

    Stopwatch stopWatch = Stopwatch.StartNew();
    Task task;
    ConsoleKeyInfo keyinfo;
    try
    {
        task = TransferManager.CopyAsync(sourceBlob, destinationBlob, CopyMethod.ServiceSideAsyncCopy, null, context, cancellationSource.Token);
        while(!task.IsCompleted)
        {
            if(Console.KeyAvailable)
            {
                keyinfo = Console.ReadKey(true);
                if(keyinfo.Key == ConsoleKey.C)
                {
                    cancellationSource.Cancel();
                }
            }
        }
        await task;
    }
    catch(Exception e)
    {
        Console.WriteLine("\nThe transfer is canceled: {0}", e.Message);
    }

    if(cancellationSource.IsCancellationRequested)
    {
        Console.WriteLine("\nTransfer will resume in 3 seconds...");
        Thread.Sleep(3000);
        checkpoint = context.LastCheckpoint;
        context = GetSingleTransferContext(checkpoint);
        Console.WriteLine("\nResuming transfer...\n");
        await TransferManager.CopyAsync(sourceBlob, destinationBlob, false, null, context, cancellationSource.Token);
    }

    stopWatch.Stop();
    Console.WriteLine("\nTransfer operation completed in " + stopWatch.Elapsed.TotalSeconds + " seconds.");
    ExecuteChoice(account);
}
```

Dans cet exemple, nous donnons la valeur `false` au paramètre booléen `TransferManager.CopyAsync` pour indiquer que nous souhaitons effectuer une copie synchrone. Cela signifie que la ressource est d’abord téléchargée sur notre ordinateur local, puis chargée vers le Stockage Blob Azure. L’option de copie synchrone est un excellent moyen de veiller à ce que l’opération de copie ait une vitesse constante. À l’inverse, la vitesse de la copie asynchrone côté serveur dépend de la bande passante réseau disponible sur le serveur, qui peut varier. Toutefois, la copie synchrone peut générer des coûts de sortie supplémentaires par rapport à la copie asynchrone. L’approche recommandée consiste à utiliser la copie synchrone dans une machine virtuelle Azure qui se trouve dans la même région que votre compte de stockage source afin d’éviter les frais de sortie.

L’application de déplacement des données est à présent terminée. [L’exemple de code complet est disponible sur GitHub](https://github.com/azure-samples/storage-dotnet-data-movement-library-app).

## <a name="next-steps"></a>Étapes suivantes

[Documentation de référence de la bibliothèque de déplacement des données du Stockage Azure](https://azure.github.io/azure-storage-net-data-movement).

[!INCLUDE [storage-try-azure-tools-blobs](../../../includes/storage-try-azure-tools-blobs.md)]
