---
title: Bien démarrer avec Stockage Blob à l’aide de Visual Studio (services cloud)
description: Comment prendre en main le stockage d’objets blob Azure dans un projet service cloud dans Visual Studio après s’être connecté à un compte de stockage à l’aide des services connectés de Visual Studio
services: storage
author: ghogen
manager: jillfra
ms.assetid: 1144a958-f75a-4466-bb21-320b7ae8f304
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.custom: vs-azure, devx-track-csharp
ms.workload: azure-vs
ms.topic: conceptual
ms.date: 12/02/2016
ms.author: ghogen
ROBOTS: NOINDEX,NOFOLLOW
ms.openlocfilehash: 89e0d6873ebfd8f8396c36185730c57a66af0dd9
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "96007027"
---
# <a name="get-started-with-azure-blob-storage-and-visual-studio-connected-services-cloud-services-projects"></a>Prendre en main le stockage d’objets blob Azure et les services connectés de Visual Studio (projets services cloud)
[!INCLUDE [storage-try-azure-tools-blobs](../../includes/storage-try-azure-tools-blobs.md)]

## <a name="overview"></a>Vue d’ensemble
Cet article décrit comment prendre en main Azure Blob Storage après avoir créé ou référencé un compte de stockage Azure via la boîte de dialogue **Ajouter des services connectés** de Visual Studio, dans un projet Visual Studio Cloud Services. Nous allons vous montrer comment créer des conteneurs d’objets blob, y accéder et effectuer des tâches courantes comme télécharger, répertorier et charger des objets blob. Les exemples sont écrits en code C\# et utilisent la bibliothèque cliente [Stockage Microsoft Azure pour .NET](/previous-versions/azure/dn261237(v=azure.100)).

Le stockage d’objets blob Azure est un service permettant de stocker de grandes quantités de données non structurées accessibles depuis n’importe où dans le monde via HTTP ou HTTPS. Les objets blob peuvent être de toutes tailles. Il peut s'agir d'images, de fichiers audio ou vidéo, de données brutes ou de fichiers de documents.

De la même manière que les fichiers résident dans des dossiers, le stockage des objets blob s'effectue dans des conteneurs. Après avoir créé un stockage, créez un ou plusieurs conteneurs dans le stockage. Par exemple, dans un stockage appelé « Scrapbook », vous pouvez créer des conteneurs dans le stockage : un conteneur nommé « images » pour stocker des photos et un autre nommé « audio » pour stocker des fichiers audio. Une fois que vous avez créé les conteneurs, vous pouvez y charger des fichiers blob.

* Pour plus d’informations sur la manipulation des objets blob par programme, consultez la page [Prise en main du stockage d’objets blob Azure à l’aide de .NET](../storage/blobs/storage-quickstart-blobs-dotnet.md).
* Pour obtenir des informations générales sur Azure Storage, consultez [Documentation du stockage](https://azure.microsoft.com/documentation/services/storage/).
* Pour obtenir des informations générales sur Azure Cloud Services, consultez la [Documentation Azure Cloud Services](https://azure.microsoft.com/documentation/services/cloud-services/).
* Pour plus d’informations sur la programmation d’applications ASP.NET, consultez [ASP.NET](https://www.asp.net).

## <a name="access-blob-containers-in-code"></a>Accès aux conteneurs d'objets blob dans le code
Pour accéder par programmation aux objets blob dans des projets de service cloud, vous devez ajouter les éléments suivants s’ils ne sont pas déjà présents.

1. Ajoutez les déclarations d'espace de noms suivantes en haut de chaque fichier C# pour lequel vous souhaitez accéder à Azure Storage par programmation :
   
    ```csharp
    using Microsoft.Framework.Configuration;
    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Blob;
    using System.Threading.Tasks;
    using LogLevel = Microsoft.Framework.Logging.LogLevel;
    ```
2. Obtenez un objet **CloudStorageAccount** qui représente les informations de votre compte de stockage. Le code suivant permet d'obtenir la chaîne de connexion de stockage et les informations de compte de stockage à partir de la configuration du service Azure.
   
    ```csharp
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("<storage account name>_AzureStorageConnectionString"));
    ```
3. Obtenez un objet **CloudBlobClient** pour référencer un conteneur existant dans votre compte de stockage.
   
    ```csharp
    // Create a blob client.
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();
    ```
4. Obtenez un objet **CloudBlobContainer** pour référencer un conteneur d’objets blob spécifique.
   
    ```csharp
    // Get a reference to a container named "mycontainer."
    CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");
    ```

> [!NOTE]
> Utilisez l’ensemble du code montré dans la procédure précédente en haut du code des sections suivantes.
> 
> 

## <a name="create-a-container-in-code"></a>Création d'un conteneur dans le code
> [!NOTE]
> Certaines API qui effectuent des appels à Azure Storage sont asynchrones. Pour plus d’informations, consultez l’article [Programmation asynchrone avec Async et Await](/previous-versions/hh191443(v=vs.140)) . Le code de l’exemple suivant suppose que vous utilisez des méthodes de programmation asynchrones.
> 
> 

Pour créer un conteneur dans votre compte de stockage, il suffit d’ajouter un appel à **CreateIfNotExistsAsync** , comme dans le code suivant :

```csharp
// If "mycontainer" doesn't exist, create it.
await container.CreateIfNotExistsAsync();
```


Pour rendre publics les fichiers présents dans le conteneur, vous pouvez configurer le conteneur en conséquence à l’aide du code suivant :

```csharp
await container.SetPermissionsAsync(new BlobContainerPermissions
{
    PublicAccess = BlobContainerPublicAccessType.Blob
});
```


Tous les utilisateurs d’Internet peuvent afficher les objets blob d’un conteneur public, mais seuls ceux possédant la clé d’accès adéquate peuvent les modifier ou les supprimer.

## <a name="upload-a-blob-into-a-container"></a>Charger un objet blob dans un conteneur
Azure Storage prend en charge les objets blob de blocs et de pages. Dans la plupart des cas, il est recommandé d’utiliser le type d’objet blob de blocs.

Pour télécharger un fichier vers un objet blob de blocs, obtenez une référence de conteneur et utilisez-la pour obtenir une référence d’objet blob de blocs. Lorsque vous disposez d’une référence d’objet blob, vous pouvez télécharger un flux de données vers cet objet en appelant la méthode **UploadFromStream**. Cette opération crée l’objet blob s’il n’existait pas. S’il existe, il est remplacé. L’exemple suivant illustre le chargement d’un objet blob dans un conteneur en partant du principe que le conteneur existe déjà.

```csharp
// Retrieve a reference to a blob named "myblob".
CloudBlockBlob blockBlob = container.GetBlockBlobReference("myblob");

// Create or overwrite the "myblob" blob with contents from a local file.
using (var fileStream = System.IO.File.OpenRead(@"path\myfile"))
{
    blockBlob.UploadFromStream(fileStream);
}
```

## <a name="list-the-blobs-in-a-container"></a>Créer la liste des objets blob d’un conteneur
Pour créer une liste d’objets blob dans un conteneur, commencez par obtenir une référence pointant vers un conteneur. Vous pouvez ensuite utiliser la méthode **ListBlobs** du conteneur pour récupérer les objets blob et/ou les répertoires qu’il contient. Pour accéder aux nombreuses propriétés et méthodes d’une **IListBlobItem** renvoyée, vous devez l’appeler vers un objet **CloudBlockBlob**, **CloudPageBlob** ou **CloudBlobDirectory**. Si vous ne connaissez pas le type, vous pouvez lancer une vérification de type pour déterminer la cible de l’appel. Le code suivant illustre la récupération et la génération de l’URI de chaque élément du conteneur **photos** :

```csharp
// Loop over items within the container and output the length and URI.
foreach (IListBlobItem item in container.ListBlobs(null, false))
{
    if (item.GetType() == typeof(CloudBlockBlob))
    {
        CloudBlockBlob blob = (CloudBlockBlob)item;

        Console.WriteLine("Block blob of length {0}: {1}", blob.Properties.Length, blob.Uri);

    }
    else if (item.GetType() == typeof(CloudPageBlob))
    {
        CloudPageBlob pageBlob = (CloudPageBlob)item;

        Console.WriteLine("Page blob of length {0}: {1}", pageBlob.Properties.Length, pageBlob.Uri);

    }
    else if (item.GetType() == typeof(CloudBlobDirectory))
    {
        CloudBlobDirectory directory = (CloudBlobDirectory)item;

        Console.WriteLine("Directory: {0}", directory.Uri);
    }
}
```

Comme le montre l’exemple de code précédent, le service blob intègre également le concept de répertoires dans des conteneurs. Vous pouvez donc organiser vos objets blob selon une structure proche de celle des dossiers. Par exemple, prenez l’ensemble d’objets blob de blocs suivant, situé dans un conteneur nommé **photos**:

```output
photo1.jpg
2010/architecture/description.txt
2010/architecture/photo3.jpg
2010/architecture/photo4.jpg
2011/architecture/photo5.jpg
2011/architecture/photo6.jpg
2011/architecture/description.txt
2011/photo7.jpg
```

Quand vous appelez **ListBlobs** pour le conteneur (comme dans l’exemple précédent), la collection renvoyée contient les objets **CloudBlobDirectory** et **CloudBlockBlob** qui représentent les répertoires et objets blob contenus au niveau supérieur. Voici la sortie résultante :

```output
Directory: https://<accountname>.blob.core.windows.net/photos/2010/
Directory: https://<accountname>.blob.core.windows.net/photos/2011/
Block blob of length 505623: https://<accountname>.blob.core.windows.net/photos/photo1.jpg
```


En option, vous pouvez définir le paramètre **UseFlatBlobListing** de la méthode **ListBlobs** sur la valeur **true**. Cette opération se traduit par le renvoi de chaque objet blob en tant que **CloudBlockBlob**, indépendamment du répertoire. Voici l’appel à **ListBlobs**:

```csharp
// Loop over items within the container and output the length and URI.
foreach (IListBlobItem item in container.ListBlobs(null, true))
{
    ...
}
```

et voici les résultats :

```output
Block blob of length 4: https://<accountname>.blob.core.windows.net/photos/2010/architecture/description.txt
Block blob of length 314618: https://<accountname>.blob.core.windows.net/photos/2010/architecture/photo3.jpg
Block blob of length 522713: https://<accountname>.blob.core.windows.net/photos/2010/architecture/photo4.jpg
Block blob of length 4: https://<accountname>.blob.core.windows.net/photos/2011/architecture/description.txt
Block blob of length 419048: https://<accountname>.blob.core.windows.net/photos/2011/architecture/photo5.jpg
Block blob of length 506388: https://<accountname>.blob.core.windows.net/photos/2011/architecture/photo6.jpg
Block blob of length 399751: https://<accountname>.blob.core.windows.net/photos/2011/photo7.jpg
Block blob of length 505623: https://<accountname>.blob.core.windows.net/photos/photo1.jpg
```

Pour plus d’informations, consultez [CloudBlobContainer.ListBlobs](/rest/api/storageservices/List-Blobs).

## <a name="download-blobs"></a>Télécharger des objets blob
Pour télécharger des objets blob, commencez par récupérer une référence d’objet blob, puis appelez la méthode **DownloadToStream** . L’exemple suivant utilise la méthode **DownloadToStream** pour transférer les contenus d’objets blob vers un objet de flux pouvant être rendu persistant dans un fichier local.

```csharp
// Get a reference to a blob named "photo1.jpg".
CloudBlockBlob blockBlob = container.GetBlockBlobReference("photo1.jpg");

// Save blob contents to a file.
using (var fileStream = System.IO.File.OpenWrite(@"path\myfile"))
{
    blockBlob.DownloadToStream(fileStream);
}
```

Vous pouvez également utiliser la méthode **DownloadToStream** pour télécharger les contenus d’un objet blob en tant que chaîne de texte.

```csharp
// Get a reference to a blob named "myblob.txt"
CloudBlockBlob blockBlob2 = container.GetBlockBlobReference("myblob.txt");

string text;
using (var memoryStream = new MemoryStream())
{
    blockBlob2.DownloadToStream(memoryStream);
    text = System.Text.Encoding.UTF8.GetString(memoryStream.ToArray());
}
```

## <a name="delete-blobs"></a>Suppression d’objets blob
Pour supprimer un objet blob, commencez par obtenir une référence d’objet blob, puis appelez la méthode **Delete** .

```csharp
// Get a reference to a blob named "myblob.txt".
CloudBlockBlob blockBlob = container.GetBlockBlobReference("myblob.txt");

// Delete the blob.
blockBlob.Delete();
```


## <a name="list-blobs-in-pages-asynchronously"></a>Création d’une liste d’objets blob dans des pages de manière asynchrone
Si vous répertoriez un grand nombre d’objets blob ou que vous souhaitez contrôler le nombre de résultats renvoyés lors d’une opération de création de liste, vous pouvez répertorier les objets blob dans des pages de résultats. Cet exemple montre comment renvoyer les résultats dans des pages de manière asynchrone afin que l’exécution ne soit pas bloquée lorsqu’un ensemble volumineux de résultats est attendu.

Cet exemple montre une liste plate d’objets blob. Vous pouvez également avoir une liste hiérarchique en définissant le paramètre **useFlatBlobListing** de la méthode **ListBlobsSegmentedAsync** sur **false**.

Comme l’exemple de méthode appelle une méthode asynchrone, il doit être précédé du mot clé **async** et doit renvoyer un objet **Task**. Le mot clé d’attente spécifié pour la méthode **ListBlobsSegmentedAsync** suspend l’exécution de l’exemple de méthode jusqu’à ce que la tâche de création de liste soit terminée.

```csharp
async public static Task ListBlobsSegmentedInFlatListing(CloudBlobContainer container)
{
    // List blobs to the console window, with paging.
    Console.WriteLine("List blobs in pages:");

    int i = 0;
    BlobContinuationToken continuationToken = null;
    BlobResultSegment resultSegment = null;

    // Call ListBlobsSegmentedAsync and enumerate the result segment returned, while the continuation token is non-null.
    // When the continuation token is null, the last page has been returned and execution can exit the loop.
    do
    {
        // This overload allows control of the page size. You can return all remaining results by passing null for the maxResults parameter,
        // or by calling a different overload.
        resultSegment = await container.ListBlobsSegmentedAsync("", true, BlobListingDetails.All, 10, continuationToken, null, null);
        if (resultSegment.Results.Count<IListBlobItem>() > 0) { Console.WriteLine("Page {0}:", ++i); }
        foreach (var blobItem in resultSegment.Results)
        {
            Console.WriteLine("\t{0}", blobItem.StorageUri.PrimaryUri);
        }
        Console.WriteLine();

        //Get the continuation token.
        continuationToken = resultSegment.ContinuationToken;
    }
    while (continuationToken != null);
}
```

## <a name="next-steps"></a>Étapes suivantes
[!INCLUDE [vs-storage-dotnet-blobs-next-steps](../../includes/vs-storage-dotnet-blobs-next-steps.md)]