---
title: Utiliser .NET pour gérer les propriétés et les métadonnées d’un conteneur d’objets blob
titleSuffix: Azure Storage
description: Découvrez comment définir et récupérer des propriétés système et stocker des métadonnées personnalisées dans des conteneurs d’objets blob dans votre compte de stockage Azure à l’aide de la bibliothèque cliente .NET.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 07/01/2020
ms.author: tamram
ms.custom: devx-track-csharp
ms.openlocfilehash: 9fb77179a00969da7a3dc372dc70c99cfe4220ca
ms.sourcegitcommit: 30505c01d43ef71dac08138a960903c2b53f2499
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92090996"
---
# <a name="manage-container-properties-and-metadata-with-net"></a>Gérer les métadonnées et les propriétés de conteneur avec .NET

Les conteneurs d’objets blob prennent en charge des propriétés système et des métadonnées définies par l’utilisateur, en plus des données qu’ils contiennent. Cet article présente la gestion des propriétés système et des métadonnées définies par l’utilisateur avec la [bibliothèque cliente Stockage Azure pour .NET](/dotnet/api/overview/azure/storage).

## <a name="about-properties-and-metadata"></a>À propos des propriétés et des métadonnées

- **Propriétés système** : Propriétés système existant sur chaque ressource de stockage blob. Certaines d'entre elles peuvent être lues ou configurées, alors que d'autres sont en lecture seule. En arrière-plan, certaines propriétés système correspondent à certains en-têtes HTTP standard. La bibliothèque cliente de Stockage Azure pour .NET gère ces propriétés pour vous.

- **Métadonnées définies par l’utilisateur** : ces métadonnées se composent d’une ou plusieurs paires nom/valeur, que vous spécifiez pour une ressource de stockage d’objets blob. Vous pouvez les utiliser pour stocker des valeurs supplémentaires avec la ressource. Les valeurs de métadonnées sont destinées à votre usage personnel et n’affectent pas le comportement de la ressource.

Les paires nom/valeur de métadonnées sont des en-têtes HTTP valides ; elles doivent donc respecter toutes les restrictions régissant les en-têtes HTTP. Les noms de métadonnées doivent être des noms d’en-tête HTTP et identificateurs C# valides, peuvent contenir uniquement des caractères ASCII, et doivent être considérés comme sensibles à la casse. Les valeurs de métadonnées contenant des caractères non-ASCII doivent être codées URL ou Base64.

## <a name="retrieve-container-properties"></a>Récupérer les propriétés de conteneur

# <a name="net-v12"></a>[.NET v12](#tab/dotnet)

Pour récupérer les propriétés d’un conteneur, appelez une des méthodes suivantes :

- [GetProperties](/dotnet/api/azure.storage.blobs.blobcontainerclient.getproperties)
- [GetPropertiesAsync](/dotnet/api/azure.storage.blobs.blobcontainerclient.getpropertiesasync)

L’exemple de code suivant récupère les propriétés système d’un conteneur et écrit certaines des valeurs des propriétés dans une fenêtre de console :

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Metadata.cs" id="Snippet_ReadContainerProperties":::

# <a name="net-v11"></a>[.NET v11](#tab/dotnet11)

La récupération des valeurs des propriétés et des métadonnées d’une ressource de stockage d’objet blob se déroule en deux étapes. Pour pouvoir lire ces valeurs, vous devez les récupérer explicitement en appelant l’une des méthodes **FetchAttributesAsync** ou **FetchAttributesAsync**. L’exception à cette règle est que les méthodes **Exists** et **ExistsAsync** appellent la méthode **FetchAttributes** appropriée en coulisses. Lorsque vous appelez une de ces méthodes, vous n’avez pas besoin d’appeler **FetchAttributes**.

> [!IMPORTANT]
> Si vous constatez que les valeurs de propriété ou de métadonnées pour une ressource de stockage n’ont pas été complétées, vérifiez que votre code appelle l’une des méthodes **FetchAttributes** ou **FetchAttributesAsync**.

Pour récupérer les propriétés d’un conteneur, appelez une des méthodes suivantes :

- [FetchAttributes](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.fetchattributes)
- [FetchAttributesAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.fetchattributesasync)

L’exemple de code suivant récupère les propriétés système d’un conteneur et écrit certaines des valeurs des propriétés dans une fenêtre de console :

```csharp
private static async Task ReadContainerPropertiesAsync(CloudBlobContainer container)
{
    try
    {
        // Fetch some container properties and write out their values.
        await container.FetchAttributesAsync();
        Console.WriteLine("Properties for container {0}", container.StorageUri.PrimaryUri);
        Console.WriteLine("Public access level: {0}", container.Properties.PublicAccess);
        Console.WriteLine("Last modified time in UTC: {0}", container.Properties.LastModified);
    }
    catch (StorageException e)
    {
        Console.WriteLine("HTTP error code {0}: {1}",
                            e.RequestInformation.HttpStatusCode,
                            e.RequestInformation.ErrorCode);
        Console.WriteLine(e.Message);
        Console.ReadLine();
    }
}
```

---

## <a name="set-and-retrieve-metadata"></a>Définir et récupérer des métadonnées

# <a name="net-v12"></a>[.NET v12](#tab/dotnet)

Vous pouvez indiquer des métadonnées sous la forme de paires nom-valeur sur une ressource d’objet blob ou de conteneur. Pour définir les métadonnées, ajoutez des paires nom-valeur à un objet [IDictionary](/dotnet/api/system.collections.idictionary), puis appelez les méthodes suivantes pour écrire les valeurs :

- [SetMetadata](/dotnet/api/azure.storage.blobs.blobcontainerclient.setmetadata)
- [SetMetadataAsync](/dotnet/api/azure.storage.blobs.blobcontainerclient.setmetadataasync)

Le nom de vos métadonnées doit respecter la convention d’affectation de noms pour les identificateurs C#. Les noms des métadonnées conservent la casse avec laquelle ils ont été créés, mais ils ne respectent pas la casse lorsqu’ils sont définis ou lus. Si deux en-têtes de métadonnées ou plus portant le même nom sont envoyés pour une ressource, le stockage blob sépare et concatène les deux valeurs par une virgule et retourne le code de réponse HTTP 200 (OK).

L’exemple de code suivant définit les métadonnées d’un conteneur.

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Metadata.cs" id="Snippet_AddContainerMetadata":::

Pour récupérer des métadonnées, appelez l’une des méthodes suivantes :

- [GetProperties](/dotnet/api/azure.storage.blobs.blobcontainerclient.getproperties)
- [GetPropertiesAsync](/dotnet/api/azure.storage.blobs.blobcontainerclient.getpropertiesasync).

Ensuite, lisez les valeurs, comme indiqué dans l’exemple ci-dessous.

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Metadata.cs" id="Snippet_ReadContainerMetadata":::

# <a name="net-v11"></a>[.NET v11](#tab/dotnet11)

Vous pouvez indiquer des métadonnées sous la forme de paires nom-valeur sur une ressource d’objet blob ou de conteneur. Pour définir les métadonnées, ajoutez des paires nom-valeur à la collection **Metadata** de la ressource, puis appelez les méthodes suivantes pour écrire les valeurs :

- [SetMetadata](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.setmetadata)
- [SetMetadataAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.setmetadataasync)

Le nom de vos métadonnées doit respecter la convention d’affectation de noms pour les identificateurs C#. Les noms des métadonnées conservent la casse avec laquelle ils ont été créés, mais ils ne respectent pas la casse lorsqu’ils sont définis ou lus. Si deux en-têtes de métadonnées ou plus portant le même nom sont envoyés pour une ressource, le stockage blob sépare et concatène les deux valeurs par une virgule et retourne le code de réponse HTTP 200 (OK).

L’exemple de code suivant définit les métadonnées d’un conteneur. Une valeur est définie à l’aide de la méthode **Add** de la collection. L’autre valeur est définie à l’aide d’une syntaxe implicite clé/valeur. Les deux sont valides.

```csharp
public static async Task AddContainerMetadataAsync(CloudBlobContainer container)
{
    try
    {
        // Add some metadata to the container.
        container.Metadata.Add("docType", "textDocuments");
        container.Metadata["category"] = "guidance";

        // Set the container's metadata.
        await container.SetMetadataAsync();
    }
    catch (StorageException e)
    {
        Console.WriteLine("HTTP error code {0}: {1}",
                            e.RequestInformation.HttpStatusCode,
                            e.RequestInformation.ErrorCode);
        Console.WriteLine(e.Message);
        Console.ReadLine();
    }
}
```

Pour récupérer des métadonnées, appelez l’une des méthodes **FetchAttributes** ou **SetMetadataAsync** sur votre objet blob ou votre conteneur pour remplir la collection **Metadata**, puis lisez les valeurs, comme indiqué dans l’exemple suivant.

```csharp
public static async Task ReadContainerMetadataAsync(CloudBlobContainer container)
{
    try
    {
        // Fetch container attributes in order to populate the container's properties and metadata.
        await container.FetchAttributesAsync();

        // Enumerate the container's metadata.
        Console.WriteLine("Container metadata:");
        foreach (var metadataItem in container.Metadata)
        {
            Console.WriteLine("\tKey: {0}", metadataItem.Key);
            Console.WriteLine("\tValue: {0}", metadataItem.Value);
        }
    }
    catch (StorageException e)
    {
        Console.WriteLine("HTTP error code {0}: {1}",
                            e.RequestInformation.HttpStatusCode,
                            e.RequestInformation.ErrorCode);
        Console.WriteLine(e.Message);
        Console.ReadLine();
    }
}
```

---

[!INCLUDE [storage-blob-dotnet-resources-include](../../../includes/storage-blob-dotnet-resources-include.md)]

## <a name="see-also"></a>Voir aussi

- [Opération de récupération des propriétés du conteneur](/rest/api/storageservices/get-container-properties)
- [Opération de définition des métadonnées du conteneur](/rest/api/storageservices/set-container-metadata)
- [Opération de récupération des métadonnées du conteneur](/rest/api/storageservices/get-container-metadata)
