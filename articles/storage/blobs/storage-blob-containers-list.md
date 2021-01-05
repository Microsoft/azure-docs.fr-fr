---
title: Répertorier les conteneurs d’objets BLOB avec .NET - Stockage Azure
description: Découvrez comment répertorier les conteneurs d’objets BLOB dans votre compte de stockage Azure à l’aide de la bibliothèque de client .NET.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 10/14/2020
ms.author: tamram
ms.subservice: blobs
ms.custom: devx-track-csharp
ms.openlocfilehash: a12fc991734fe74e450aa14a477f3a4500ba659c
ms.sourcegitcommit: dea56e0dd919ad4250dde03c11d5406530c21c28
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96937251"
---
# <a name="list-blob-containers-with-net"></a>Répertoriez les conteneurs d’objets BLOB avec .NET

Lorsque vous répertoriez les conteneurs dans un compte de stockage Azure à partir de votre code, vous pouvez spécifier un certain nombre d’options pour gérer la façon dont les résultats sont retournés à partir du stockage Azure. Cet article explique comment répertorier les conteneurs à l’aide de la [bibliothèque de client de stockage Azure pour .NET](/dotnet/api/overview/azure/storage).  

## <a name="understand-container-listing-options"></a>Comprenez les options de la liste des conteneurs

Pour répertorier les conteneurs dans votre compte de stockage, appelez l’une des méthodes suivantes :

# <a name="net-v12"></a>[.NET v12](#tab/dotnet)

- [GetBlobContainers](/dotnet/api/azure.storage.blobs.blobserviceclient.getblobcontainers)
- [GetBlobContainersAsync](/dotnet/api/azure.storage.blobs.blobserviceclient.getblobcontainersasync)

# <a name="net-v11"></a>[.NET v11](#tab/dotnet11)

- [ListContainersSegmented](/dotnet/api/microsoft.azure.storage.blob.cloudblobclient.listcontainerssegmented)
- [ListContainersSegmentedAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobclient.listcontainerssegmentedasync)

---

Les surcharges de ces méthodes fournissent des options supplémentaires pour gérer la façon dont les conteneurs sont retournés par l’opération de dressage de liste. Les étapes de cette procédure sont décrites dans les sections suivantes.

### <a name="manage-how-many-results-are-returned"></a>Gérez le nombre de résultats retournés

Par défaut, une opération de dressage de liste renvoie jusqu’à 5 000 résultats à la fois. Pour retourner un ensemble de résultats plus petit, fournissez une valeur différente de zéro pour la taille de la page de résultats à retourner.

Si votre compte de stockage contient plus de 5000 conteneurs, ou si vous avez spécifié une taille de page telle que l’opération de création de liste retourne un sous-ensemble de conteneurs dans le compte de stockage, le Stockage Azure retourne un *jeton de continuation* avec la liste des conteneurs. Un jeton de continuation est une valeur opaque que vous pouvez utiliser pour récupérer le jeu de résultats suivant à partir du stockage Azure.

Dans votre code, vérifiez la valeur du jeton de continuation pour déterminer s’il est vide (pour .NET v12) ou s’agit d’une valeur Null (pour .NET v11 et versions antérieures). Lorsque le jeton de continuation a la valeur Null, l’ensemble de résultats est complété. Si le jeton de continuation n’a pas la valeur Null, appelez à nouveau la méthode de création de liste, en lui passant le jeton de continuation pour récupérer l’ensemble de résultats suivant, jusqu’à ce que le jeton de continuation ait la valeur Null.

### <a name="filter-results-with-a-prefix"></a>Filtrez les résultats avec un préfixe

Pour filtrer la liste des conteneurs, spécifiez une chaîne pour le paramètre `prefix`. La chaîne de préfixe peut inclure un ou plusieurs caractères. Le stockage Azure retourne alors uniquement les conteneurs dont les noms commencent par ce préfixe.

### <a name="return-metadata"></a>Retourner les métadonnées

Pour retourner les métadonnées de conteneur avec les résultats, spécifiez la valeur **Métadonnées** pour l’énumération [BlobContainerTraits](/dotnet/api/azure.storage.blobs.models.blobcontainertraits) (pour .NET v12) ou l’énumération [ContainerListingDetails](/dotnet/api/microsoft.azure.storage.blob.containerlistingdetails) (pour .NET v11 et versions antérieures). Le Stockage Azure comprend des métadonnées avec chaque conteneur retourné. Vous n’avez donc pas besoin de récupérer également les métadonnées de conteneur.

## <a name="example-list-containers"></a>Exemple : Répertorier les conteneurs

L’exemple suivant répertorie de manière asynchrone les conteneurs dans un compte de stockage qui commencent par un préfixe spécifié. L’exemple répertorie les conteneurs qui commencent par le préfixe spécifié et retourne le nombre de résultats spécifié par appel de l’opération de création liste. Il utilise ensuite le jeton de continuation pour obtenir le segment suivant des résultats. L’exemple retourne également les métadonnées de conteneur avec les résultats.

# <a name="net-v12"></a>[.NET v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Containers.cs" id="Snippet_ListContainers":::

# <a name="net-v11"></a>[.NET v11](#tab/dotnet11)

```csharp
private static async Task ListContainersWithPrefixAsync(CloudBlobClient blobClient,
                                                        string prefix,
                                                        int? segmentSize)
{
    Console.WriteLine("List containers beginning with prefix {0}, plus container metadata:", prefix);

    BlobContinuationToken continuationToken = null;
    ContainerResultSegment resultSegment;

    try
    {
        do
        {
            // List containers beginning with the specified prefix,
            // returning segments of 5 results each.
            // Passing in null for the maxResults parameter returns the maximum number of results (up to 5000).
            // Requesting the container's metadata as part of the listing operation populates the metadata,
            // so it's not necessary to call FetchAttributes() to read the metadata.
            resultSegment = await blobClient.ListContainersSegmentedAsync(
                prefix, ContainerListingDetails.Metadata, segmentSize, continuationToken, null, null);

            // Enumerate the containers returned.
            foreach (var container in resultSegment.Results)
            {
                Console.WriteLine("\tContainer:" + container.Name);

                // Write the container's metadata keys and values.
                foreach (var metadataItem in container.Metadata)
                {
                    Console.WriteLine("\t\tMetadata key: " + metadataItem.Key);
                    Console.WriteLine("\t\tMetadata value: " + metadataItem.Value);
                }
            }

            // Get the continuation token.
            continuationToken = resultSegment.ContinuationToken;

        } while (continuationToken != null);

        Console.WriteLine();
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

[!INCLUDE [storage-blob-dotnet-resources-include](../../../includes/storage-blob-dotnet-resources-include.md)]

## <a name="see-also"></a>Voir aussi

- [Lister des conteneurs](/rest/api/storageservices/list-containers2)
- [Énumération des ressources d’objets blob](/rest/api/storageservices/enumerating-blob-resources)
