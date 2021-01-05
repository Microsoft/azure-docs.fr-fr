---
title: Spécifier une clé fournie par le client sur une demande au stockage d’objets blob avec .NET - Stockage Azure
description: Découvrez comment spécifier une clé fournie par le client sur une demande au stockage d’objets blob avec .NET.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 07/20/2020
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.custom: devx-track-csharp
ms.openlocfilehash: 50d592d0020ae1b5a704296ef68f5153f0207714
ms.sourcegitcommit: 0dcafc8436a0fe3ba12cb82384d6b69c9a6b9536
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94427573"
---
# <a name="specify-a-customer-provided-key-on-a-request-to-blob-storage-with-net"></a>Spécifier une clé fournie par le client sur une demande au stockage d’objets blob avec .NET

Les clients effectuant des requêtes auprès du stockage d’objets blob Azure ont la possibilité de fournir une clé de chiffrement sur une requête individuelle. L’inclusion de la clé de chiffrement dans la requête offre un contrôle précis des paramètres de chiffrement pour les opérations de stockage d’objets blob. Les clés fournies par le client peuvent être stockées dans Azure Key Vault ou dans un autre magasin de clés.

Cet article explique comment spécifier une clé fournie par le client sur une demande avec .NET.

[!INCLUDE [storage-install-packages-blob-and-identity-include](../../../includes/storage-install-packages-blob-and-identity-include.md)]

Pour en savoir plus sur la façon de s’authentifier avec de la bibliothèque cliente Azure Identity à partir du Stockage Azure, consultez la section intitulée **S’authentifier avec la bibliothèque d’identité Azure** dans [Autoriser l’accès aux objets blob et files d’attente avec Azure Active Directory et les identités managées pour les ressources Azure](../common/storage-auth-aad-msi.md?toc=%2Fazure%2Fstorage%2Fblobs%2Ftoc.json#authenticate-with-the-azure-identity-library).

## <a name="example-use-a-customer-provided-key-to-upload-a-blob"></a>Exemple : Utiliser une clé fournie par le client pour charger un objet blob

L’exemple suivant crée une clé fournie par le client et utilise cette clé pour charger un objet blob. Le code charge un bloc, puis valide la liste de blocs pour écrire l’objet blob dans Stockage Azure.

```csharp
async static Task UploadBlobWithClientKey(string accountName, string containerName,
    string blobName, Stream data, byte[] key)
{
    const string blobServiceEndpointSuffix = ".blob.core.windows.net";
    Uri accountUri = new Uri("https://" + accountName + blobServiceEndpointSuffix);

    // Specify the customer-provided key on the options for the client.
    BlobClientOptions options = new BlobClientOptions()
    {
        CustomerProvidedKey = new CustomerProvidedKey(key)
    };

    // Create a client object for the Blob service, including options.
    BlobServiceClient serviceClient = new BlobServiceClient(accountUri, 
        new DefaultAzureCredential(), options);

    // Create a client object for the container.
    // The container client retains the credential and client options.
    BlobContainerClient containerClient = serviceClient.GetBlobContainerClient(containerName);

    // Create a new block blob client object.
    // The blob client retains the credential and client options.
    BlobClient blobClient = containerClient.GetBlobClient(blobName);

    try
    {
        // Create the container if it does not exist.
        await containerClient.CreateIfNotExistsAsync();

        // Upload the data using the customer-provided key.
        await blobClient.UploadAsync(data);
    }
    catch (RequestFailedException e)
    {
        Console.WriteLine(e.Message);
        Console.ReadLine();
        throw;
    }
}
```

## <a name="next-steps"></a>Étapes suivantes

- [Fournir une clé de chiffrement lors d’une requête au stockage d’objets blob](encryption-customer-provided-keys.md)
- [Chiffrement du stockage Azure pour les données au repos](../common/storage-service-encryption.md)
