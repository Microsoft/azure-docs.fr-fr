---
title: Lister les objets blob avec .NET – Stockage Azure
description: Découvrez comment lister les objets blob dans un conteneur dans votre compte de stockage Azure à l’aide de la bibliothèque cliente .NET. Les exemples de code montrent comment répertorier des blobs dans une liste plate ou comment les répertorier hiérarchiquement, comme s’ils étaient organisés dans des répertoires ou des dossiers.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 11/16/2020
ms.author: tamram
ms.subservice: blobs
ms.custom: devx-track-csharp
ms.openlocfilehash: ddd19c90c8c47016497e2c3b00e04595a94e7715
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95543066"
---
# <a name="list-blobs-with-net"></a>Lister les objets blob avec .NET

Lorsque vous listez les objets blob de votre code, vous pouvez spécifier un certain nombre d’options pour gérer la façon dont les résultats sont retournés à partir du stockage Azure. Vous pouvez spécifier le nombre de résultats à retourner dans chaque ensemble de résultats, puis récupérer les ensembles suivants. Vous pouvez spécifier un préfixe pour retourner les blobs dont le nom commence par ce caractère ou cette chaîne. Vous pouvez également répertorier les blobs dans une structure de liste plate, ou hiérarchiquement. Une liste hiérarchique retourne les blobs comme s’ils étaient organisés en dossiers.

Cet article explique comment lister les objets blob à l’aide de la [bibliothèque cliente de stockage Azure pour .NET](/dotnet/api/overview/azure/storage).  

## <a name="understand-blob-listing-options"></a>Présentation des options de liste d’objets blob

Pour lister les objets blob dans un compte de stockage, appelez l’une des méthodes suivantes :

# <a name="net-v12"></a>[.NET v12](#tab/dotnet)

- [BlobContainerClient.GetBlobs](/dotnet/api/azure.storage.blobs.blobcontainerclient.getblobs)
- [BlobContainerClient.GetBlobsAsync](/dotnet/api/azure.storage.blobs.blobcontainerclient.getblobsasync)
- [BlobContainerClient.GetBlobsByHierarchy](/dotnet/api/azure.storage.blobs.blobcontainerclient.getblobsbyhierarchy)
- [BlobContainerClient.GetBlobsByHierarchyAsync](/dotnet/api/azure.storage.blobs.blobcontainerclient.getblobsbyhierarchyasync)

# <a name="net-v11"></a>[.NET v11](#tab/dotnet11)

- [CloudBlobClient.ListBlobs](/dotnet/api/microsoft.azure.storage.blob.cloudblobclient.listblobs)
- [CloudBlobClient.ListBlobsSegmented](/dotnet/api/microsoft.azure.storage.blob.cloudblobclient.listblobssegmented)
- [CloudBlobClient.ListBlobsSegmentedAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobclient.listblobssegmentedasync)

Pour lister les objets blob dans un conteneur, appelez l’une des méthodes suivantes :

- [CloudBlobContainer.ListBlobs](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.listblobs)
- [CloudBlobContainer.ListBlobsSegmented](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.listblobssegmented)
- [CloudBlobContainer.ListBlobsSegmentedAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.listblobssegmentedasync)

Les surcharges de ces méthodes fournissent des options supplémentaires pour gérer la façon dont les objets blob sont retournés par l’opération de création de liste. Les étapes de cette procédure sont décrites dans les sections suivantes.

---

### <a name="manage-how-many-results-are-returned"></a>Gérez le nombre de résultats retournés

Par défaut, une opération de liste renvoie jusqu’à 5 000 résultats à la fois, mais vous pouvez spécifier le nombre de résultats que chaque opération de liste doit renvoyer. Les exemples présentés dans cet article vous montrent comment restituer les résultats en pages.

### <a name="filter-results-with-a-prefix"></a>Filtrez les résultats avec un préfixe

Pour filtrer la liste de blobs, spécifiez une chaîne pour le paramètre `prefix`. La chaîne de préfixe peut inclure un ou plusieurs caractères. Le stockage Azure retourne alors uniquement les objets blob dont les noms commencent par ce préfixe.

### <a name="return-metadata"></a>Retourner les métadonnées

Vous pouvez retourner des métadonnées d’objet blob avec les résultats.

- Si vous utilisez le Kit de développement logiciel (SDK) .NET v12, spécifiez la valeur **Metadata** pour l’énumération [BlobTraits](/dotnet/api/azure.storage.blobs.models.blobtraits).

- Si vous utilisez le Kit de développement logiciel (SDK) .NET v11, spécifiez la valeur **Metadata** pour l’énumération [BlobListingDetails](/dotnet/api/microsoft.azure.storage.blob.bloblistingdetails). Le stockage Azure inclut des métadonnées avec chaque objet blob retourné. Vous n’avez donc pas besoin d’appeler l’une des méthodes **FetchAttributes** dans ce contexte pour récupérer les métadonnées des objets blob.

### <a name="list-blob-versions-or-snapshots"></a>Répertorier les versions ou les instantanés de blobs

Pour répertorier les versions ou les instantanés de blobs avec la bibliothèque de client .NET v12, spécifiez le paramètre [BlobStates](/dotnet/api/azure.storage.blobs.models.blobstates) avec le champ **Version** ou **Instantané**. Les versions et les instantanés sont répertoriés du plus ancien au plus récent. Pour plus d’informations sur la liste des versions, consultez [Répertorier les versions de blobs](versioning-enable.md#list-blob-versions).

### <a name="flat-listing-versus-hierarchical-listing"></a>Création d’une liste plate ou d’une liste hiérarchique

Les objets blob dans le stockage Azure sont organisés en paradigme plat, plutôt qu’en paradigme hiérarchique (comme un système de fichiers standard). Toutefois, vous pouvez organiser les blobs en *répertoires virtuels* afin de simuler une structure de dossiers. Un répertoire virtuel fait partie du nom du blob et est indiqué par le caractère délimiteur.

Pour organiser les objets blob en répertoires virtuels, utilisez un caractère délimiteur dans les noms des objets blob. Le caractère délimiteur par défaut est une barre oblique (/), mais vous pouvez spécifier n’importe quel caractère comme délimiteur.

Si vous nommez vos objets blob en utilisant un délimiteur, vous pouvez choisir de lister les objets blob hiérarchiquement. Pour une opération de création de liste hiérarchique, le stockage Azure retourne tous les répertoires virtuels et les objets blob figurant sous l’objet parent. Vous pouvez appeler l’opération de création de liste de manière récursive pour parcourir la hiérarchie, de la même façon que vous parcourez un système de fichiers standard par programmation.

## <a name="use-a-flat-listing"></a>Utiliser une liste plate

Par défaut, une opération de création de liste retourne les objets blob dans une liste plate. Dans une liste plate, les objets blob ne sont pas organisés par répertoire virtuel.

L’exemple suivant liste les objets blob dans le conteneur spécifié à l’aide d’une liste plate, avec une taille de segment facultative spécifiée, et écrit le nom de l’objet blob dans une fenêtre de console.

Si vous avez activé la fonctionnalité d’espace de noms hiérarchique sur votre compte, les répertoires ne sont pas virtuels. Au lieu de cela, ce sont des objets concrets et indépendants. Par conséquent, les répertoires apparaissent dans la liste sous forme d’objets blob de longueur nulle.

# <a name="net-v12"></a>[.NET v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/CRUD.cs" id="Snippet_ListBlobsFlatListing":::

# <a name="net-v11"></a>[.NET v11](#tab/dotnet11)

Si une opération de liste retourne plus de 5 000 objets blob, ou si le nombre d’objets blob disponibles dépasse le nombre que vous avez spécifié, le stockage Azure retourne un *jeton de continuation* avec la liste d’objets blob. Un jeton de continuation est une valeur opaque que vous pouvez utiliser pour récupérer le jeu de résultats suivant à partir du stockage Azure.

Dans votre code, vérifiez la valeur du jeton de continuation pour déterminer s’il s’agit d’une valeur Null. Lorsque le jeton de continuation a la valeur Null, l’ensemble de résultats est complété. Si le jeton de continuation n’a pas la valeur Null, appelez à nouveau l’opération de création de liste, en lui passant le jeton de continuation pour récupérer l’ensemble de résultats suivant, jusqu’à ce que le jeton de continuation ait la valeur Null.

```csharp
private static async Task ListBlobsFlatListingAsync(CloudBlobContainer container, int? segmentSize)
{
    BlobContinuationToken continuationToken = null;
    CloudBlob blob;

    try
    {
        // Call the listing operation and enumerate the result segment.
        // When the continuation token is null, the last segment has been returned
        // and execution can exit the loop.
        do
        {
            BlobResultSegment resultSegment = await container.ListBlobsSegmentedAsync(string.Empty,
                true, BlobListingDetails.Metadata, segmentSize, continuationToken, null, null);

            foreach (var blobItem in resultSegment.Results)
            {
                blob = (CloudBlob)blobItem;

                // Write out some blob properties.
                Console.WriteLine("Blob name: {0}", blob.Name);
            }

            Console.WriteLine();

           // Get the continuation token and loop until it is null.
           continuationToken = resultSegment.ContinuationToken;

        } while (continuationToken != null);
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

Exemple de sortie :

```
Blob name: FolderA/blob1.txt
Blob name: FolderA/blob2.txt
Blob name: FolderA/blob3.txt
Blob name: FolderA/FolderB/blob1.txt
Blob name: FolderA/FolderB/blob2.txt
Blob name: FolderA/FolderB/blob3.txt
Blob name: FolderA/FolderB/FolderC/blob1.txt
Blob name: FolderA/FolderB/FolderC/blob2.txt
Blob name: FolderA/FolderB/FolderC/blob3.txt
```

## <a name="use-a-hierarchical-listing"></a>Utiliser une liste hiérarchique

Lorsque vous appelez une opération de création de liste hiérarchique, le stockage Azure retourne les répertoires virtuels et les objets blob figurant au premier niveau de la hiérarchie. La propriété [Prefix](/dotnet/api/microsoft.azure.storage.blob.cloudblobdirectory.prefix) de chaque répertoire virtuel est définie pour que vous puissiez passer le préfixe dans un appel récursif afin de récupérer le répertoire suivant.

# <a name="net-v12"></a>[.NET v12](#tab/dotnet)

Pour répertorier les objets blob de façon hiérarchique, appelez la méthode [BlobContainerClient.GetBlobsByHierarchy](/dotnet/api/azure.storage.blobs.blobcontainerclient.getblobsbyhierarchy), ou [BlobContainerClient.GetBlobsByHierarchyAsync](/dotnet/api/azure.storage.blobs.blobcontainerclient.getblobsbyhierarchyasync).

L’exemple suivant liste les objets blob dans le conteneur spécifié à l’aide d’une liste hiérarchique, avec une taille de segment facultative spécifiée, et écrit le nom de l’objet blob dans la fenêtre de console.

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/CRUD.cs" id="Snippet_ListBlobsHierarchicalListing":::

# <a name="net-v11"></a>[.NET v11](#tab/dotnet11)

Pour lister les objets blob hiérarchiquement, définissez le paramètre `useFlatBlobListing` de la méthode de création de liste sur **false**.

L’exemple suivant liste les objets blob dans le conteneur spécifié à l’aide d’une liste plate, avec une taille de segment facultative spécifiée, et écrit le nom de l’objet blob dans la fenêtre de console.

```csharp
private static async Task ListBlobsHierarchicalListingAsync(CloudBlobContainer container, string prefix)
{
    CloudBlobDirectory dir;
    CloudBlob blob;
    BlobContinuationToken continuationToken = null;

    try
    {
        // Call the listing operation and enumerate the result segment.
        // When the continuation token is null, the last segment has been returned and
        // execution can exit the loop.
        do
        {
            BlobResultSegment resultSegment = await container.ListBlobsSegmentedAsync(prefix,
                false, BlobListingDetails.Metadata, null, continuationToken, null, null);
            foreach (var blobItem in resultSegment.Results)
            {
                // A hierarchical listing may return both virtual directories and blobs.
                if (blobItem is CloudBlobDirectory)
                {
                    dir = (CloudBlobDirectory)blobItem;

                    // Write out the prefix of the virtual directory.
                    Console.WriteLine("Virtual directory prefix: {0}", dir.Prefix);

                    // Call recursively with the prefix to traverse the virtual directory.
                    await ListBlobsHierarchicalListingAsync(container, dir.Prefix);
                }
                else
                {
                    // Write out the name of the blob.
                    blob = (CloudBlob)blobItem;

                    Console.WriteLine("Blob name: {0}", blob.Name);
                }
                Console.WriteLine();
            }

            // Get the continuation token and loop until it is null.
            continuationToken = resultSegment.ContinuationToken;

        } while (continuationToken != null);
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

Exemple de sortie :

```
Virtual directory prefix: FolderA/
Blob name: FolderA/blob1.txt
Blob name: FolderA/blob2.txt
Blob name: FolderA/blob3.txt

Virtual directory prefix: FolderA/FolderB/
Blob name: FolderA/FolderB/blob1.txt
Blob name: FolderA/FolderB/blob2.txt
Blob name: FolderA/FolderB/blob3.txt

Virtual directory prefix: FolderA/FolderB/FolderC/
Blob name: FolderA/FolderB/FolderC/blob1.txt
Blob name: FolderA/FolderB/FolderC/blob2.txt
Blob name: FolderA/FolderB/FolderC/blob3.txt
```

> [!NOTE]
> Les instantanés d’objets blob ne peuvent pas être listés dans une opération de création de liste hiérarchique.

[!INCLUDE [storage-blob-dotnet-resources-include](../../../includes/storage-blob-dotnet-resources-include.md)]

## <a name="next-steps"></a>Étapes suivantes

- [Lister les objets blob](/rest/api/storageservices/list-blobs)
- [Énumération des ressources d’objets blob](/rest/api/storageservices/enumerating-blob-resources)