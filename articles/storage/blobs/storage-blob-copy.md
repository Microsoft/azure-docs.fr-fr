---
title: Copier un objet blob avec les API Stockage Azure
description: Découvrez comment copier un objet blob à l’aide des bibliothèques clientes Stockage Azure.
services: storage
author: mhopkins-msft
ms.author: mhopkins
ms.date: 09/10/2020
ms.service: storage
ms.subservice: blobs
ms.topic: how-to
ms.custom: devx-track-csharp, devx-track-python
ms.openlocfilehash: 2c474ed4d4158356075f861c3c0d5ace69173255
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90014648"
---
# <a name="copy-a-blob-with-azure-storage-client-libraries"></a>Copier un objet blob avec des bibliothèques clientes Stockage Azure

Cet article explique comment copier un objet blob dans un compte de stockage Azure. Il montre également comment abandonner une opération de copie asynchrone. L’exemple de code utilise les bibliothèques clientes de stockage Azure.

## <a name="about-copying-blobs"></a>À propos de la copie d’objets blob

Lorsque vous copiez un objet blob dans le même compte de stockage, il s’agit d’une opération synchrone. Lorsque vous effectuez la copie d’un compte à un autre, il s’agit d’une opération asynchrone.

L’objet blob source d’une opération de copie peut être un objet blob de blocs, un objet blob d’ajout, un objet blob de pages ou un instantané. Si l’objet blob de destination existe déjà, il doit être du même type d’objet blob que l’objet blob source. Tout objet blob de destination existant sera remplacé.

L’objet blob de destination ne peut pas être modifié pendant une opération de copie. Un objet blob de destination ne peut avoir qu’une seule opération de copie en attente. En d’autres termes, un objet blob ne peut pas être la destination de plusieurs opérations de copie en attente.

L’intégralité du fichier ou de l’objet blob source est toujours copiée. La copie d’une plage d’octets ou d’un ensemble de blocs n’est pas prise en charge.

Lorsqu’un objet blob est copié, ses propriétés système sont copiées dans l’objet blob de destination avec les mêmes valeurs.

Une opération de copie peut prendre l’une des formes suivantes :

- Copiez un objet blob source vers un objet blob de destination avec un nom différent. L’objet blob de destination peut être un objet blob existant du même type d’objet blob (bloc, ajout ou page), ou il peut s’agir d’un nouvel objet blob créé par l’opération de copie.
- Copiez un objet blob source vers un objet blob de destination portant le même nom, ce qui a pour effet de remplacer l’objet blob de destination. Cette opération supprime tous les blocs non validés et remplace les métadonnées de l’objet blob de destination.
- Copiez un fichier source figurant dans le service Azure File vers un objet blob de destination. L’objet blob de destination peut être un objet blob existant ou un nouvel objet blob de blocs créé par l’opération de copie. La copie de fichiers vers des objets blob de pages ou d’ajout n’est pas prise en charge.
- Copiez un instantané sur son objet blob de base. En plaçant un instantané à la place d'un objet blob de base, vous pouvez restaurer une version antérieure de l’objet blob.
- Copiez un instantané sur un objet blob de destination avec un nom différent. L’objet blob de destination obtenu est un objet blob modifiable et non pas un instantané.

## <a name="copy-a-blob"></a>Copier un objet blob

# <a name="net-v12"></a>[.NET v12](#tab/dotnet)

Pour copier un objet blob, appelez l’une des méthodes suivantes :

- [StartCopyFromUri](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.startcopyfromuri)
- [StartCopyFromUriAsync](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.startcopyfromuriasync)

Les méthodes `StartCopyFromUri` et `StartCopyFromUriAsync` retournent un objet [CopyFromUriOperation](/dotnet/api/azure.storage.blobs.models.copyfromurioperation) contenant des informations sur l’opération de copie.

L’exemple de code suivant obtient un client [BlobClient](/dotnet/api/azure.storage.blobs.blobclient) qui représente un objet blob créé précédemment et le copie dans un nouvel objet blob dans le même conteneur :

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/CopyBlob.cs" id="Snippet_CopyBlob":::

# <a name="net-v11"></a>[.NET v11](#tab/dotnet11)

Pour copier un objet blob, appelez l’une des méthodes suivantes :

- [StartCopy](/dotnet/api/microsoft.azure.storage.blob.cloudblob.startcopy)
- [StartCopyAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblob.startcopyasync)

Les méthodes `StartCopy` et `StartCopyAsync` retournent une valeur d’ID de copie utilisée pour vérifier l’état ou abandonner l’opération de copie.

L’exemple de code suivant obtient une référence à un objet blob créé précédemment et copie ce dernier dans un nouvel objet blob dans le même conteneur :

```csharp
private static async Task CopyBlockBlobAsync(CloudBlobContainer container)
{
    CloudBlockBlob sourceBlob = null;
    CloudBlockBlob destBlob = null;
    string leaseId = null;

    try
    {
        // Get a block blob from the container to use as the source.
        sourceBlob = container.ListBlobs().OfType<CloudBlockBlob>().FirstOrDefault();

        // Lease the source blob for the copy operation 
        // to prevent another client from modifying it.
        // Specifying null for the lease interval creates an infinite lease.
        leaseId = await sourceBlob.AcquireLeaseAsync(null);

        // Get a reference to a destination blob (in this case, a new blob).
        destBlob = container.GetBlockBlobReference("copy of " + sourceBlob.Name);

        // Ensure that the source blob exists.
        if (await sourceBlob.ExistsAsync())
        {
            // Get the ID of the copy operation.
            string copyId = await destBlob.StartCopyAsync(sourceBlob);

            // Fetch the destination blob's properties before checking the copy state.
            await destBlob.FetchAttributesAsync();

            Console.WriteLine("Status of copy operation: {0}", destBlob.CopyState.Status);
            Console.WriteLine("Completion time: {0}", destBlob.CopyState.CompletionTime);
            Console.WriteLine("Bytes copied: {0}", destBlob.CopyState.BytesCopied.ToString());
            Console.WriteLine("Total bytes: {0}", destBlob.CopyState.TotalBytes.ToString());
        }
    }
    catch (StorageException e)
    {
        Console.WriteLine(e.Message);
        Console.ReadLine();
        throw;
    }
    finally
    {
        // Break the lease on the source blob.
        if (sourceBlob != null)
        {
            await sourceBlob.FetchAttributesAsync();

            if (sourceBlob.Properties.LeaseState != LeaseState.Available)
            {
                await sourceBlob.BreakLeaseAsync(new TimeSpan(0));
            }
        }
    }
}
```

# <a name="python-v12"></a>[Python v12](#tab/python)

Pour copier un objet blob, appelez la méthode [start_copy_from_url](/azure/developer/python/sdk/storage/azure-storage-blob/azure.storage.blob.blobclient#start-copy-from-url-source-url--metadata-none--incremental-copy-false----kwargs-). La méthode `start_copy_from_url` retourne un dictionnaire contenant des informations sur l’opération de copie.

L’exemple de code suivant obtient un client [BlobClient](/azure/developer/python/sdk/storage/azure-storage-blob/azure.storage.blob.blobclient) qui représente un objet blob créé précédemment et le copie dans un nouvel objet blob dans le même conteneur :

:::code language="python" source="~/azure-storage-snippets/blobs/howto/python/python-v12/copy_blob.py" id="Snippet_BlobCopy":::

---

## <a name="abort-a-copy-operation"></a>Abandonner une opération de copie

L'abandon d'une opération de copie résulte en un objet blob de destination de longueur nulle. Toutefois, les métadonnées pour l'objet blob de destination auront les nouvelles valeurs copiées de l'objet blob source ou définies explicitement pendant l'opération de copie. Pour conserver les métadonnées d’origine antérieures à la copie, créez un instantané de l’objet blob de destination avant d’appeler une des méthodes de copie.

# <a name="net-v12"></a>[.NET v12](#tab/dotnet)

Vérifiez la propriété [BlobProperties.CopyStatus](/dotnet/api/azure.storage.blobs.models.blobproperties.copystatus) dans l’objet blob de destination pour obtenir l’état de l’opération de copie. L’objet blob final est validé à la fin de la copie.

Lorsque vous abandonnez une opération de copie, la propriété de l’objet blob de destination est définie sur [CopyStatus.Aborted](/dotnet/api/microsoft.azure.storage.blob.copystatus).

Les méthodes [AbortCopyFromUri](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.abortcopyfromuri) et [AbortCopyFromUriAsync](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.abortcopyfromuriasync) annulent une opération de copie en cours.

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/CopyBlob.cs" id="Snippet_StopBlobCopy":::

# <a name="net-v11"></a>[.NET v11](#tab/dotnet11)

Vérifiez la propriété [CopyState.Status](/dotnet/api/microsoft.azure.storage.blob.copystate.status) dans l’objet blob de destination pour obtenir l’état de l’opération de copie. L’objet blob final est validé à la fin de la copie.

Lorsque vous abandonnez une opération de copie, la propriété de l’objet blob de destination est définie sur [CopyStatus.Aborted](/dotnet/api/microsoft.azure.storage.blob.copystatus).

Les méthodes [AbortCopy](/dotnet/api/microsoft.azure.storage.blob.cloudblob.abortcopy) et [AbortCopyAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblob.abortcopyasync) annulent une opération de copie en cours.

```csharp
// Fetch the destination blob's properties before checking the copy state.
await destBlob.FetchAttributesAsync();

// Check the copy status. If it is still pending, abort the copy operation.
if (destBlob.CopyState.Status == CopyStatus.Pending)
{
    await destBlob.AbortCopyAsync(copyId);
    Console.WriteLine("Copy operation {0} has been aborted.", copyId);
}
```

# <a name="python-v12"></a>[Python v12](#tab/python)

Vérifiez l’entrée « status » dans le dictionnaire [CopyProperties](/azure/developer/python/sdk/storage/azure-storage-blob/azure.storage.blob.copyproperties) retourné par la méthode [get_blob_properties](/azure/developer/python/sdk/storage/azure-storage-blob/azure.storage.blob.blobclient#get-blob-properties---kwargs-) pour connaître l’état de l’opération de copie. L’objet blob final est validé à la fin de la copie.

Lorsque vous abandonnez une opération de copie, l’[état](/azure/developer/python/sdk/storage/azure-storage-blob/azure.storage.blob.copyproperties) est défini sur « aborted ».

La méthode [abort_copy](/azure/developer/python/sdk/storage/azure-storage-blob/azure.storage.blob.blobclient#abort-copy-copy-id----kwargs-) annule une opération de copie en cours.

:::code language="python" source="~/azure-storage-snippets/blobs/howto/python/python-v12/copy_blob.py" id="Snippet_StopBlobCopy":::

---

## <a name="azure-sdks"></a>SDK Azure

Obtenez davantage d’informations sur les SDK Azure :

 - [Kit de développement logiciel (SDK) Azure pour .NET](https://github.com/azure/azure-sdk-for-net)
 - [Kit de développement logiciel (SDK) Azure pour Java](https://github.com/azure/azure-sdk-for-java)
 - [Kit de développement logiciel (SDK) Azure pour Python](https://github.com/azure/azure-sdk-for-python)
 - [Kit de développement logiciel (SDK) pour JavaScript](https://github.com/azure/azure-sdk-for-js)

## <a name="next-steps"></a>Étapes suivantes

Les rubriques suivantes contiennent des informations sur la copie des objets blob et sur l’abandon des opérations de copie en cours à l’aide des API REST Azure.

- [Copy Blob](/rest/api/storageservices/copy-blob)
- [Abort Copy Blob](/rest/api/storageservices/abort-copy-blob)
