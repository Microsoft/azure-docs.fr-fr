---
title: Accéder anonymement aux conteneurs et objets blob publics avec .NET
titleSuffix: Azure Storage
description: Utilisez la bibliothèque cliente de Stockage Azure pour .NET afin d’accéder anonymement aux conteneurs et objets blob publics.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 08/02/2020
ms.author: tamram
ms.reviewer: fryu
ms.subservice: blobs
ms.custom: devx-track-csharp
ms.openlocfilehash: 2437c5b3272163b3931d7417c84e761c591aec85
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90088814"
---
# <a name="access-public-containers-and-blobs-anonymously-with-net"></a>Accéder anonymement aux conteneurs et objets blob publics avec .NET

Stockage Azure prend en charge l’accès en lecture public facultatif pour les conteneurs et les blobs. Les clients peuvent accéder aux conteneurs et objets blob publics de manière anonyme à l’aide des bibliothèques clientes de Stockage Azure, ainsi qu’avec d’autres outils et utilitaires qui prennent en charge l’accès aux données dans le Stockage Azure.

Cet article montre comment accéder à un conteneur ou à un objet blob public à partir de .NET. Pour plus d’informations sur la configuration de l’accès en lecture anonyme sur un conteneur, consultez [Configurer l’accès en lecture public anonyme pour les conteneurs et les objets blob](anonymous-read-access-configure.md). Pour plus d’informations sur la façon d’empêcher tout accès anonyme à un compte de stockage, consultez [Empêcher l’accès en lecture public anonyme aux conteneurs et aux blobs](anonymous-read-access-prevent.md).

Un client ayant un accès anonyme aux conteneurs et aux objets Blob peut utiliser des constructeurs qui ne nécessitent pas d’informations d’identification. Les exemples suivants illustrent différentes manières de référencer des conteneurs et objets blob de façon anonyme.

## <a name="create-an-anonymous-client-object"></a>Créer un objet de client anonyme

Vous pouvez créer un objet de client de service pour permettre un accès anonyme en spécifiant le point de terminaison du stockage d’objets blob associé au compte. Toutefois, vous devez également connaître le nom d’un conteneur attaché à ce compte qui est disponible pour un accès anonyme.

# <a name="net-v12-sdk"></a>[SDK \.NET v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Security.cs" id="Snippet_CreateAnonymousBlobClient":::

# <a name="net-v11-sdk"></a>[SDK \.NET v11](#tab/dotnet11)

```csharp
public static void CreateAnonymousBlobClient()
{
    // Create the client object using the Blob storage endpoint for your account.
    CloudBlobClient blobClient = new CloudBlobClient(
        new Uri(@"https://storagesamples.blob.core.windows.net"));

    // Get a reference to a container that's available for anonymous access.
    CloudBlobContainer container = blobClient.GetContainerReference("sample-container");

    // Read the container's properties. 
    // Note this is only possible when the container supports full public read access.
    container.FetchAttributes();
    Console.WriteLine(container.Properties.LastModified);
    Console.WriteLine(container.Properties.ETag);
}
```

---

## <a name="reference-a-container-anonymously"></a>Référencer un conteneur de manière anonyme

Si vous disposez de l’URL permettant d’accéder à un conteneur accessible de manière anonyme, vous pouvez l’utiliser pour référencer directement le conteneur.

# <a name="net-v12-sdk"></a>[SDK \.NET v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Security.cs" id="Snippet_ListBlobsAnonymously":::

# <a name="net-v11-sdk"></a>[SDK \.NET v11](#tab/dotnet11)

```csharp
public static void ListBlobsAnonymously()
{
    // Get a reference to a container that's available for anonymous access.
    CloudBlobContainer container = new CloudBlobContainer(
        new Uri(@"https://storagesamples.blob.core.windows.net/sample-container"));

    // List blobs in the container.
    // Note this is only possible when the container supports full public read access.
    foreach (IListBlobItem blobItem in container.ListBlobs())
    {
        Console.WriteLine(blobItem.Uri);
    }
}
```

---

## <a name="reference-a-blob-anonymously"></a>Référencer un objet Blob de façon anonyme

Si vous disposez de l’URL permettant d’accéder à un objet Blob accessible de manière anonyme, vous pouvez l’utiliser pour référencer directement l’objet Blob :

# <a name="net-v12-sdk"></a>[SDK \.NET v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Security.cs" id="Snippet_DownloadBlobAnonymously":::

# <a name="net-v11-sdk"></a>[SDK \.NET v11](#tab/dotnet11)

```csharp
public static void DownloadBlobAnonymously()
{
    CloudBlockBlob blob = new CloudBlockBlob(
        new Uri(@"https://storagesamples.blob.core.windows.net/sample-container/logfile.txt"));
    blob.DownloadToFile(@"C:\Temp\logfile.txt", FileMode.Create);
}
```

---

## <a name="next-steps"></a>Étapes suivantes

- [Configurer l’accès en lecture public anonyme pour les conteneurs et les objets blob](anonymous-read-access-configure.md)
- [Empêcher l’accès en lecture public anonyme aux conteneurs et aux objets blob](anonymous-read-access-prevent.md)
- [Autorisation d’accès au Stockage Azure](../common/storage-auth.md)
