---
title: Gestion de l’accès concurrentiel dans le Stockage Blob
titleSuffix: Azure Storage
description: Apprenez à gérer plusieurs opérations d’écriture sur un objet blob en implémentant l’accès concurrentiel optimiste ou pessimiste dans votre application. L’accès concurrentiel optimiste vérifie la valeur ETag d’un objet blob et la compare à la valeur ETag fournie. L’accès concurrentiel pessimiste utilise un bail exclusif pour verrouiller l’objet blob sur d’autres opérations d’écriture.
services: storage
author: tamram
ms.service: storage
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 12/01/2020
ms.author: tamram
ms.subservice: common
ms.custom: devx-track-csharp
ms.openlocfilehash: ea0bed0884a3a03e2cd15b274b2afb0f054b0cbd
ms.sourcegitcommit: 84e3db454ad2bccf529dabba518558bd28e2a4e6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96523271"
---
# <a name="managing-concurrency-in-blob-storage"></a>Gestion de l’accès concurrentiel dans Blob Storage

Dans les applications modernes, les données sont souvent consultées et mises à jour par plusieurs utilisateurs à la fois. Les développeurs d'applications doivent donc bien réfléchir à la manière de proposer une expérience prévisible à leurs utilisateurs finaux, notamment lorsque plusieurs utilisateurs peuvent mettre à jour les mêmes données. Les développeurs prennent généralement en compte trois grandes stratégies d’accès concurrentiel aux données :  

- **Accès concurrentiel optimiste** : Une application procédant à une mise à jour vérifie, dans le cadre de la mise à jour, si les données n'ont pas été modifiées depuis la dernière lecture. Par exemple, si deux utilisateurs qui consultent une page wiki procèdent à une mise à jour de cette page, la plateforme wiki doit veiller à ce que la deuxième mise à jour n'écrase pas la première. Ils doivent également veiller à ce que les deux utilisateurs sachent si leur mise à jour a fonctionné ou non. Cette stratégie est la plus souvent utilisée dans les applications web.

- **Accès concurrentiel pessimiste** : Une application qui cherche à procéder à une mise à jour verrouille l'objet, ce qui empêche les autres utilisateurs de mettre les données à jour jusqu'à ce qu'elles soient déverrouillées. Par exemple, dans un scénario de réplication de données avec réplicas principal/secondaire où seul le réplica principal procède aux mises à jour, celui-ci verrouille généralement les données de manière exclusive pendant une période prolongée de manière à ce que personne d'autre ne puisse les mettre à jour.

- **Dernière écriture prioritaire** : Une approche permettant aux opérations de mise à jour de continuer sans déterminer d’abord si une autre application a mis à jour les données depuis leur lecture. Cette approche est généralement utilisée lorsque les données sont partitionnées de manière à ce que plusieurs utilisateurs n’accèdent pas simultanément aux mêmes données. Elle peut également être utile lors du traitement de flux de données à durée de vie limitée.

Stockage Azure prend en charge les trois stratégies, bien qu’il se distingue par sa capacité à prendre en charge pleinement l’accès concurrentiel optimiste et pessimiste. Stockage Azure a été conçu pour appliquer un modèle de cohérence fort qui garantit qu’une fois que le service a effectué une opération d’insertion ou de mise à jour, les opérations de lecture suivantes retournent la dernière mise à jour.

Parallèlement à la sélection d'une stratégie d'accès concurrentiel adaptée, les développeurs doivent savoir comment la plateforme de stockage isole les changements, notamment ceux apportés à un même objet au fil des transactions. Stockage Azure utilise l’isolement de capture instantanée pour permettre l’exécution simultanée des opérations de lecture et d’écriture au sein d’une même partition. L’isolement d’instantané garantit que toutes les opérations de lecture retournent un instantané cohérent des données, même pendant les mises à jour.

Vous pouvez choisir d’utiliser des modèles d’accès concurrentiel optimiste ou pessimiste pour gérer l’accès aux objets blob et aux conteneurs. Si vous ne spécifiez pas une stratégie de manière explicite, la dernière écriture prévaut par défaut.  

## <a name="optimistic-concurrency"></a>Accès concurrentiel optimiste

Stockage Azure attribue un identificateur à chaque objet stocké. Cet identificateur est mis à jour chaque fois qu'une opération d’écriture est effectuée sur un objet. L'identificateur est renvoyé au client en tant que réponse HTTP GET dans l'en-tête ETag défini par le protocole HTTP.

Un client qui effectue une mise à jour peut envoyer la valeur ETag d’origine avec un en-tête conditionnel pour s’assurer qu’une mise à jour se produira uniquement si une certaine condition est remplie. Par exemple, si l’en-tête **If-Match** est spécifié, Stockage Azure vérifie que la valeur ETag spécifiée dans la demande de mise à jour est identique à la valeur ETag de l’objet en cours de mise à jour. Pour plus d’informations sur les en-têtes conditionnels, consultez [Spécification des en-têtes conditionnels pour les opérations du service Blob](/rest/api/storageservices/specifying-conditional-headers-for-blob-service-operations).

Ce processus se déroule comme suit :  

1. Récupérez un objet blob à partir de Stockage Azure. La réponse inclut une valeur d'en-tête ETag HTTP qui identifie la version actuelle de l'objet.
1. Lorsque vous mettez l'objet blob à jour, incluez la valeur ETag reçue à l'étape 1 dans l'en-tête conditionnel **If-Match** de la demande d’écriture. Stockage Azure compare la valeur ETag de la demande à la valeur ETag de l'objet blob.
1. Si la valeur ETag actuelle de l’objet blob diffère de celle spécifiée dans l’en-tête conditionnel **If-Match** fourni dans la demande, Stockage Azure retourne le code d’état HTTP 412 (échec de la précondition). Cette erreur indique au client que l’objet blob a été mis à jour par un autre processus depuis la première récupération par le client.
1. Si la valeur ETag actuelle de l'objet blob est la même que la balise ETag dans l'en-tête conditionnel **If-Match** de la demande, Stockage Azure effectue l'opération demandée et met la valeur ETag de l'objet blob à jour.  

Les exemples de code suivants montrent comment construire une condition **If-Match** sur la demande d’écriture qui vérifie la valeur ETag pour un objet blob. Stockage Azure évalue si la valeur ETag actuelle de l’objet blob est identique à la valeur ETag fournie dans la demande, puis exécute l’opération d’écriture uniquement si les deux valeurs ETag correspondent. Si l’objet blob a été mis à jour entre-temps par un autre processus, Stockage Azure retourne un message d’état HTTP 412 (Échec de la condition préalable).  

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Concurrency.cs" id="Snippet_DemonstrateOptimisticConcurrencyBlob":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

```csharp
public void DemonstrateOptimisticConcurrencyBlob(string containerName, string blobName)
{
    Console.WriteLine("Demonstrate optimistic concurrency");

    // Parse connection string and create container.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(ConnectionString);
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();
    CloudBlobContainer container = blobClient.GetContainerReference(containerName);
    container.CreateIfNotExists();

    // Create test blob. The default strategy is last writer wins, so
    // write operation will overwrite existing blob if present.
    CloudBlockBlob blockBlob = container.GetBlockBlobReference(blobName);
    blockBlob.UploadText("Hello World!");

    // Retrieve the ETag from the newly created blob.
    string originalETag = blockBlob.Properties.ETag;
    Console.WriteLine("Blob added. Original ETag = {0}", originalETag);

    /// This code simulates an update by another client.
    string helloText = "Blob updated by another client.";
    // No ETag was provided, so original blob is overwritten and ETag updated.
    blockBlob.UploadText(helloText);
    Console.WriteLine("Blob updated. Updated ETag = {0}", blockBlob.Properties.ETag);

    // Now try to update the blob using the original ETag value.
    try
    {
        Console.WriteLine(@"Attempt to update blob using original ETag
                            to generate if-match access condition");
        blockBlob.UploadText(helloText, accessCondition: AccessCondition.GenerateIfMatchCondition(originalETag));
    }
    catch (StorageException ex)
    {
        if (ex.RequestInformation.HttpStatusCode == (int)HttpStatusCode.PreconditionFailed)
        {
            Console.WriteLine(@"Precondition failure as expected.
                                Blob's ETag does not match.");
        }
        else
        {
            throw;
        }
    }
    Console.WriteLine();
}
```

---

Stockage Azure prend également en charge d’autres en-têtes conditionnels, notamment **If-Modified-Since**, **If-Unmodified-Since** et **If-None-Match**. Pour plus d’informations, consultez [Spécification des en-têtes conditionnels pour les opérations du service Blob](/rest/api/storageservices/specifying-conditional-headers-for-blob-service-operations).  

## <a name="pessimistic-concurrency-for-blobs"></a>Accès concurrentiel pessimiste pour les objets blob

Pour verrouiller un objet blob de manière à l'utiliser de manière exclusive, vous pouvez obtenir un bail pour l'objet blob. Lorsque vous acquérez le bail, vous spécifiez sa durée. Un bail fini peut être compris entre 15 et 60 secondes. Un bail peut également être infini, ce qui correspond à un verrou exclusif. Vous pouvez renouveler un bail à durée limitée et vous pouvez libérer le bail lorsque vous n'en avez plus besoin. Azure Stockage libère automatiquement les baux à durée limitée quand ils expirent.  

Les baux permettent la prise en charge de différentes stratégies de synchronisation, dont des opérations d'écriture exclusive/de lecture partagée, d'écriture exclusive/de lecture exclusive et d'écriture partagée/de lecture exclusive. Si un bail existe, Stockage Azure applique l’accès exclusif aux opérations d’écriture pour le titulaire de ce bail. Cependant, pour garantir l’exclusivité des opérations de lecture, le développeur doit veiller à ce que toutes les applications clientes utilisent un identificateur de bail et à ce que seul un client à la fois dispose d’un identificateur de bail valable. Les opérations de lecture sans identificateur de bail entraînent l’application d’une stratégie de lecture partagée.  

Les exemples de code suivants montrent comment acquérir un bail exclusif sur un objet blob, mettre à jour le contenu de cet objet blob en fournissant l’ID de bail, puis libérer le bail. Si le bail est actif et que l’ID de bail n’est pas fourni dans une demande d’écriture, l’opération d’écriture échoue avec le code d’erreur 412 (échec de la précondition).  

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Concurrency.cs" id="Snippet_DemonstratePessimisticConcurrencyBlob":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

```csharp
public void DemonstratePessimisticConcurrencyBlob(string containerName, string blobName)
{
    Console.WriteLine("Demonstrate pessimistic concurrency");

    // Parse connection string and create container.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(ConnectionString);
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();
    CloudBlobContainer container = blobClient.GetContainerReference(containerName);
    container.CreateIfNotExists();

    CloudBlockBlob blockBlob = container.GetBlockBlobReference(blobName);
    blockBlob.UploadText("Hello World!");
    Console.WriteLine("Blob added.");

    // Acquire lease for 15 seconds.
    string lease = blockBlob.AcquireLease(TimeSpan.FromSeconds(15), null);
    Console.WriteLine("Blob lease acquired. Lease = {0}", lease);

    // Update blob using lease. This operation should succeed.
    const string helloText = "Blob updated";
    var accessCondition = AccessCondition.GenerateLeaseCondition(lease);
    blockBlob.UploadText(helloText, accessCondition: accessCondition);
    Console.WriteLine("Blob updated using an exclusive lease");

    // Simulate another client attempting to update to blob without providing lease.
    try
    {
        // Operation will fail as no valid lease was provided.
        Console.WriteLine("Now try to update blob without valid lease.");
        blockBlob.UploadText("Update operation will fail without lease.");
    }
    catch (StorageException ex)
    {
        if (ex.RequestInformation.HttpStatusCode == (int)HttpStatusCode.PreconditionFailed)
        {
            Console.WriteLine(@"Precondition failure error as expected.
                                Blob lease not provided.");
        }
        else
        {
            throw;
        }
    }

    // Release lease proactively.
    blockBlob.ReleaseLease(accessCondition);
    Console.WriteLine();
}
```

---

## <a name="pessimistic-concurrency-for-containers"></a>Accès concurrentiel pessimiste pour les conteneurs

Les baux sur des conteneurs permettent les mêmes stratégies de synchronisation prises en charge pour les objets blob, dont des stratégies d'écriture exclusive/de lecture partagée, d'écriture exclusive/de lecture exclusive et d'écriture partagée/de lecture exclusive. Pour les conteneurs, toutefois, le verrou exclusif est appliqué uniquement aux opérations de suppression. Pour supprimer un conteneur avec un bail actif, le client doit inclure l'identificateur du bail actif dans la demande de suppression. Toutes les autres opérations de conteneur réussiront sur un conteneur loué sans ID de bail.  

## <a name="next-steps"></a>Étapes suivantes

* [Spécification des en-têtes conditionnels pour les opérations du service BLOB](/rest/api/storageservices/specifying-conditional-headers-for-blob-service-operations)
* [Lease Container](/rest/api/storageservices/lease-container)
* [Bail Blob](/rest/api/storageservices/lease-blob)
