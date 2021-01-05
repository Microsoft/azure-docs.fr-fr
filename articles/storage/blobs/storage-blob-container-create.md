---
title: Créer ou supprimer un conteneur d’objets BLOB avec .NET - Stockage Azure
description: Découvrez comment créer ou supprimer les conteneurs d’objets BLOB dans votre compte de stockage Azure à l’aide de la bibliothèque de client .NET.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 07/22/2020
ms.author: tamram
ms.subservice: blobs
ms.custom: devx-track-csharp
ms.openlocfilehash: 3035e0036e5d35729637e35ad8cb1412a0da959f
ms.sourcegitcommit: 30505c01d43ef71dac08138a960903c2b53f2499
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92091030"
---
# <a name="create-or-delete-a-container-in-azure-storage-with-net"></a>Créer ou supprimer un conteneur dans le stockage Azure avec .NET

Les objets BLOB dans le stockage Azure sont organisés en conteneurs. Avant de pouvoir télécharger un objet BLOB, vous devez d’abord créer un conteneur. Cet article explique comment créer ou supprimer les conteneurs à l’aide de la [bibliothèque de client de stockage Azure pour .NET](/dotnet/api/overview/azure/storage).

## <a name="name-a-container"></a>Nommer un conteneur

Un nom de conteneur doit être un nom DNS valide, car il fait partie de l’URI unique utilisé pour adresser le conteneur ou ses objets BLOB. Suivez ces règles lorsque vous nommez un conteneur :

- Les noms de conteneur doivent comprendre entre 3 et 63 caractères.
- Les noms de conteneur doivent commencer par une lettre ou un chiffre, et peuvent comporter uniquement des lettres minuscules, des chiffres et des tirets (-).
- Deux tirets consécutifs ou plus ne sont pas autorisés dans les noms de conteneurs.

L’URI d’un conteneur est au format suivant :

`https://myaccount.blob.core.windows.net/mycontainer`

## <a name="create-a-container"></a>Créez un conteneur.

Pour créer un conteneur, utilisez l’une des méthodes suivantes :

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

- [Créer](/dotnet/api/azure.storage.blobs.blobcontainerclient.create)
- [CreateAsync](/dotnet/api/azure.storage.blobs.blobcontainerclient.createasync)
- [CreateIfNotExists](/dotnet/api/azure.storage.blobs.blobcontainerclient.createifnotexists)
- [CreateIfNotExistsAsync](/dotnet/api/azure.storage.blobs.blobcontainerclient.createifnotexistsasync)

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

- [Créer](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.create)
- [CreateAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.createasync)
- [CreateIfNotExists](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.createifnotexists)
- [CreateIfNotExistsAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.createifnotexistsasync)
---

Les méthodes **Créer** et **CreateAsync** lèvent une exception si un conteneur portant le même nom existe déjà.

Les méthodes **CreateIfNotExists** et **CreateIfNotExistsAsync** retournent une valeur booléenne indiquant si le conteneur a été créé. Si un conteneur portant le même nom existe déjà, ces méthodes retournent la mention **False** pour indiquer qu’un nouveau conteneur n’a pas été créé.

Les conteneurs sont créés immédiatement sous le compte de stockage. Il n’est pas possible d’imbriquer un conteneur sous un autre.

L’exemple suivant permet de créer un conteneur asynchrone :

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Containers.cs" id="CreateSampleContainerAsync":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

```csharp
private static async Task<CloudBlobContainer> CreateSampleContainerAsync(CloudBlobClient blobClient)
{
    // Name the sample container based on new GUID, to ensure uniqueness.
    // The container name must be lowercase.
    string containerName = "container-" + Guid.NewGuid();

    // Get a reference to a sample container.
    CloudBlobContainer container = blobClient.GetContainerReference(containerName);

    try
    {
        // Create the container if it does not already exist.
        bool result = await container.CreateIfNotExistsAsync();
        if (result == true)
        {
            Console.WriteLine("Created container {0}", container.Name);
        }
    }
    catch (StorageException e)
    {
        Console.WriteLine("HTTP error code {0}: {1}",
                            e.RequestInformation.HttpStatusCode,
                            e.RequestInformation.ErrorCode);
        Console.WriteLine(e.Message);
    }

    return container;
}
```
---

## <a name="create-the-root-container"></a>Créer le conteneur racine

Un conteneur racine sert de conteneur par défaut pour votre compte de stockage. Chaque compte de stockage peut avoir un conteneur racine, qui doit être nommé *$root*. Le conteneur racine doit être créé ou supprimé de manière explicite.

Vous pouvez référencer un objet BLOB stocké dans le conteneur racine sans inclure le nom du conteneur racine. Le conteneur racine vous permet de référencer un objet BLOB au niveau supérieur de la hiérarchie du compte de stockage. Par exemple, vous pouvez référencer un blob qui réside dans le conteneur racine de la manière suivante :

`https://myaccount.blob.core.windows.net/default.html`

L’exemple suivant permet de créer un conteneur racine synchrone :

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Containers.cs" id="CreateRootContainer":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

```csharp
private static void CreateRootContainer(CloudBlobClient blobClient)
{
    try
    {
        // Create the root container if it does not already exist.
        CloudBlobContainer container = blobClient.GetContainerReference("$root");
        if (container.CreateIfNotExists())
        {
            Console.WriteLine("Created root container.");
        }
        else
        {
            Console.WriteLine("Root container already exists.");
        }
    }
    catch (StorageException e)
    {
        Console.WriteLine("HTTP error code {0}: {1}",
                            e.RequestInformation.HttpStatusCode,
                            e.RequestInformation.ErrorCode);
        Console.WriteLine(e.Message);
    }
}
```
---

## <a name="delete-a-container"></a>Supprimer un conteneur

Pour supprimer un conteneur dans .NET, utilisez l’une des méthodes suivantes :

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

- [Supprimer](/dotnet/api/azure.storage.blobs.blobcontainerclient.delete)
- [DeleteAsync](/dotnet/api/azure.storage.blobs.blobcontainerclient.deleteasync)
- [DeleteIfExists](/dotnet/api/azure.storage.blobs.blobcontainerclient.deleteifexists)
- [DeleteIfExistsAsync](/dotnet/api/azure.storage.blobs.blobcontainerclient.deleteifexistsasync)

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

- [Supprimer](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.delete)
- [DeleteAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.deleteasync)
- [DeleteIfExists](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.deleteifexists)
- [DeleteIfExistsAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.deleteifexistsasync)
---

Les méthodes **Delete** et **DeleteAsync** lèvent une exception si le conteneur n’existe pas.

Les méthodes **DeleteIfExists** et **DeleteIfExistsAsync** retournent une valeur booléenne indiquant si le conteneur a été supprimé. Si le conteneur spécifié n’existe pas, ces méthodes retournent la mention **False** pour indiquer que le conteneur n’a pas été supprimé.

Après avoir supprimé un conteneur, vous ne pouvez pas créer un conteneur portant le même nom pendant *au moins* 30 secondes. Une tentative de création d’un conteneur portant le même nom échoue avec le code d’erreur HTTP 409 (conflit). Toutes les autres opérations sur le conteneur ou les blobs qu’il contient échouent avec le code d’erreur HTTP 404 (introuvable).

L’exemple suivant supprime le conteneur spécifié et traite l’exception si le conteneur n’existe pas :

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Containers.cs" id="DeleteSampleContainerAsync":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

```csharp
private static async Task DeleteSampleContainerAsync(CloudBlobClient blobClient, string containerName)
{
    CloudBlobContainer container = blobClient.GetContainerReference(containerName);

    try
    {
        // Delete the specified container and handle the exception.
        await container.DeleteAsync();
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

L’exemple suivant montre comment supprimer tous les conteneurs qui commencent par un préfixe spécifié.

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Containers.cs" id="DeleteContainersWithPrefixAsync":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

```csharp
private static async Task DeleteContainersWithPrefixAsync(CloudBlobClient blobClient, string prefix)
{
    Console.WriteLine("Delete all containers beginning with the specified prefix");
    try
    {
        foreach (var container in blobClient.ListContainers(prefix))
        {
            Console.WriteLine("\tContainer:" + container.Name);
            await container.DeleteAsync();
        }

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

- [Opération Créer le conteneur](/rest/api/storageservices/create-container)
- [Opération Supprimer le conteneur](/rest/api/storageservices/delete-container)
