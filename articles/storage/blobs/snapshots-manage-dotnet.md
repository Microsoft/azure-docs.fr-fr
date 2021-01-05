---
title: Créer et gérer un instantané blob dans .NET
titleSuffix: Azure Storage
description: Découvrez comment utiliser la bibliothèque cliente .NET pour créer un instantané en lecture seule d’un objet blob et sauvegarder ainsi les données de l’objet blob à un instant T.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 08/27/2020
ms.author: tamram
ms.subservice: blobs
ms.custom: devx-track-csharp
ms.openlocfilehash: ae1d72c30fb7c45c92f24ec5132859a45c6ae953
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95543372"
---
# <a name="create-and-manage-a-blob-snapshot-in-net"></a>Créer et gérer un instantané blob dans .NET

Un instantané est une version en lecture seule d'un objet blob capturé à un instant donné. Cet article explique comment créer ou gérer des instantanés blob à l’aide de la [bibliothèque de client de Stockage Azure pour .NET](/dotnet/api/overview/azure/storage).

Pour plus d’informations sur les instantanés d’objet blob dans le Stockage Azure, consultez [Instantanés d’objet blob](snapshots-overview.md).

## <a name="create-a-snapshot"></a>Créer un instantané

# <a name="net-v12"></a>[.NET v12](#tab/dotnet)

Pour créer un instantané d’un objet blob de blocs à l’aide de la version 12.x de la bibliothèque de client du stockage Azure pour .NET, utilisez l’une des méthodes suivantes :

- [CreateSnapshot](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.createsnapshot)
- [CreateSnapshotAsync](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.createsnapshotasync)

L’exemple de code suivant montre comment créer un instantané avec la version 12.x. Incluez une référence à la bibliothèque [Azure.Identity](https://www.nuget.org/packages/azure.identity) pour utiliser vos informations d’identification Azure AD pour autoriser les requêtes au service.

```csharp
private static async Task CreateBlockBlobSnapshot(string accountName, string containerName, string blobName, Stream data)
{
    const string blobServiceEndpointSuffix = ".blob.core.windows.net";
    Uri containerUri = new Uri("https://" + accountName + blobServiceEndpointSuffix + "/" + containerName);

    // Get a container client object and create the container.
    BlobContainerClient containerClient = new BlobContainerClient(containerUri,
        new DefaultAzureCredential());
    await containerClient.CreateIfNotExistsAsync();

    // Get a blob client object.
    BlobClient blobClient = containerClient.GetBlobClient(blobName);

    try
    {
        // Upload text to create a block blob.
        await blobClient.UploadAsync(data);

        // Add blob metadata.
        IDictionary<string, string> metadata = new Dictionary<string, string>
        {
            { "ApproxBlobCreatedDate", DateTime.UtcNow.ToString() },
            { "FileType", "text" }
        };
        await blobClient.SetMetadataAsync(metadata);

        // Sleep 5 seconds.
        System.Threading.Thread.Sleep(5000);

        // Create a snapshot of the base blob.
        // You can specify metadata at the time that the snapshot is created.
        // If no metadata is specified, then the blob's metadata is copied to the snapshot.
        await blobClient.CreateSnapshotAsync();
    }
    catch (RequestFailedException e)
    {
        Console.WriteLine(e.Message);
        Console.ReadLine();
        throw;
    }
}
```

# <a name="net-v11"></a>[.NET v11](#tab/dotnet11)

Pour créer un instantané d’un objet blob de blocs à l’aide de la version 11.x de la bibliothèque de client du stockage Azure pour .NET, utilisez l’une des méthodes suivantes :

- [CreateSnapshot](/dotnet/api/microsoft.azure.storage.blob.cloudblockblob.createsnapshot)
- [CreateSnapshotAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblockblob.createsnapshotasync)

L’exemple de code suivant montre comment créer un instantané avec la version 11.x. Cet exemple spécifie des métadonnées supplémentaires pour la capture instantanée lors de sa création.

```csharp
private static async Task CreateBlockBlobSnapshot(CloudBlobContainer container)
{
    // Create a new block blob in the container.
    CloudBlockBlob baseBlob = container.GetBlockBlobReference("sample-base-blob.txt");

    // Add blob metadata.
    baseBlob.Metadata.Add("ApproxBlobCreatedDate", DateTime.UtcNow.ToString());

    try
    {
        // Upload the blob to create it, with its metadata.
        await baseBlob.UploadTextAsync(string.Format("Base blob: {0}", baseBlob.Uri.ToString()));

        // Sleep 5 seconds.
        System.Threading.Thread.Sleep(5000);

        // Create a snapshot of the base blob.
        // You can specify metadata at the time that the snapshot is created.
        // If no metadata is specified, then the blob's metadata is copied to the snapshot.
        Dictionary<string, string> metadata = new Dictionary<string, string>();
        metadata.Add("ApproxSnapshotCreatedDate", DateTime.UtcNow.ToString());
        await baseBlob.CreateSnapshotAsync(metadata, null, null, null);
        Console.WriteLine(snapshot.SnapshotQualifiedStorageUri.PrimaryUri);
    }
    catch (StorageException e)
    {
        Console.WriteLine(e.Message);
        Console.ReadLine();
        throw;
    }
}
```

---

## <a name="delete-snapshots"></a>Suppression d'instantanés

Pour supprimer un blob, vous devez d’abord supprimer tous les instantanés de ce blob. Vous pouvez supprimer un instantané individuellement, ou spécifier que tous les instantanés doivent être supprimés lors de la suppression de l’objet blob source. Si vous essayez de supprimer un objet blob auquel des instantanés sont encore associés, une erreur se produit.

# <a name="net-v12"></a>[.NET v12](#tab/dotnet)

Pour supprimer un objet blob et ses instantanés à l’aide de la version 12.x de la bibliothèque de client du stockage Azure pour .NET, utilisez l’une des méthodes suivantes et incluez l’énumération [DeleteSnapshotsOption](/dotnet/api/azure.storage.blobs.models.deletesnapshotsoption) :

- [Supprimer](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.delete)
- [DeleteAsync](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.deleteasync)
- [DeleteIfExists](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.deleteifexists)
- [DeleteIfExistsAsync](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.deleteifexistsasync)

L’exemple de code suivant montre comment supprimer un objet blob et ses instantanés dans .NET, où `blobClient` est un objet de type [BlobClient](/dotnet/api/azure.storage.blobs.blobclient) :

```csharp
await blobClient.DeleteIfExistsAsync(DeleteSnapshotsOption.IncludeSnapshots, null, default);
```

# <a name="net-v11"></a>[.NET v11](#tab/dotnet11)

Pour supprimer un objet blob et ses instantanés à l’aide de la version 11.x de la bibliothèque de client du stockage Azure pour .NET, utilisez l’une des méthodes de suppression suivantes et incluez l’énumération [DeleteSnapshotsOption](/dotnet/api/microsoft.azure.storage.blob.deletesnapshotsoption) :

- [Supprimer](/dotnet/api/microsoft.azure.storage.blob.cloudblob.delete)
- [DeleteAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblob.deleteasync)
- [DeleteIfExists](/dotnet/api/microsoft.azure.storage.blob.cloudblob.deleteifexists)
- [DeleteIfExistsAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblob.deleteifexistsasync)

L’exemple de code suivant montre comment supprimer un objet blob et ses instantanés dans .NET, où `blockBlob` est un objet de type [CloudBlockBlob][dotnet_CloudBlockBlob] :

```csharp
await blockBlob.DeleteIfExistsAsync(DeleteSnapshotsOption.IncludeSnapshots, null, null, null);
```

---

## <a name="next-steps"></a>Étapes suivantes

- [Instantanés d’objet blob](snapshots-overview.md)
- [Versions d’objets blob](versioning-overview.md)
- [Suppression réversible pour les objets blob](./soft-delete-blob-overview.md)