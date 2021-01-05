---
title: Pièces jointes Azure Cosmos DB
description: Cet article présente une vue d’ensemble des pièces jointes Azure Cosmos DB.
author: aliuy
ms.author: andrl
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 08/07/2020
ms.reviewer: sngun
ms.openlocfilehash: b6504c0521328edc356dea1c146fe9aeb6bde55f
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93092736"
---
# <a name="azure-cosmos-db-attachments"></a>Pièces jointes Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Les pièces jointes Azure Cosmos DB sont des éléments spéciaux contenant des références vers des métadonnées associées avec un objet blob ou un fichier multimédia externe.

Azure Cosmos DB prend en charge deux types d’attachements :

* **Les pièces jointes non managées** sont un wrapper autour d’une référence URI à un objet blob stocké dans un service externe (par exemple, stockage Azure, OneDrive, etc.). Cette approche est similaire au stockage d’une propriété URI dans un élément Azure Cosmos DB standard.
* **Les pièces jointes managées** sont des objets blob gérés et stockés en interne par Azure Cosmos DB et exposés via un mediaLink généré par le système.


> [!NOTE]
> La pièce jointe est une fonctionnalité héritée. Sa prise en charge est étendue pour offrir une fonctionnalité continue si vous l’utilisez déjà.
> 
> Au lieu d’utiliser des pièces jointes, nous vous recommandons d’utiliser le stockage Blob Azure en tant que service de stockage d’objets blob conçu pour stocker des données blob. Vous pouvez continuer à stocker des métadonnées relatives aux objets blob ainsi que des liens URI de référence dans Azure Cosmos DB en tant que propriétés d’éléments. Le stockage de ces données dans Azure Cosmos DB permet d’interroger les métadonnées et les liens vers des objets blob stockés dans le stockage Blob Azure.
> 
> Microsoft s’engage à fournir un préavis minimal de 36 mois avant la dépréciation complète des pièces jointes, qui sera annoncée à une date ultérieure.

## <a name="known-limitations"></a>Limitations connues

Les pièces jointes managées d’Azure Cosmos DB sont distinctes de la prise en charge des éléments standard, pour laquelle elle offre une évolutivité illimitée, une distribution globale et une intégration avec d’autres services Azure.

- Les pièces jointes ne sont pas prises en charge dans toutes les versions des kit de développement logiciel (SDK) Azure Cosmos DB.
- Les pièces jointes managées sont limitées à 2 Go de stockage par compte de base de données.
- Les pièces jointes managées ne sont pas compatibles avec la distribution globale d’Azure Cosmos DB, et elles ne sont pas répliquées dans les régions.

## <a name="migrating-attachments-to-azure-blob-storage"></a>Migration de pièces jointes vers le stockage Blob Azure

Nous vous recommandons de migrer les pièces jointes d’Azure Cosmos DB vers le stockage Blib Azure en procédant comme suit :

1. Copiez les données des pièces jointes de votre conteneur Azure Cosmos DB source vers votre conteneur de stockage Blob Azure cible.
2. Validez les données blob chargées dans le conteneur de stockage d’objets Blob Azure cible.
3. Le cas échéant, ajoutez des références URI aux objets blob contenus dans le stockage Blob Azure en tant que propriétés de chaîne dans votre jeu de données Azure Cosmos DB.
4. Refactorisez votre code d’application pour lire et écrire des objets blob à partir du nouveau conteneur de stockage Blob Azure.

L’exemple de code dotnet suivant montre comment copier des pièces jointes d’Azure Cosmos DB vers le stockage Blob Azure dans le cadre d’un flux de migration à l’aide du kit de développement logiciel (SDK) .NET v2 et du kit de développement logiciel (SDK) .NET v12 du stockage Blob Azure. Veillez à remplacer le `<placeholder values>` pour le compte Azure Cosmos DB source et le conteneur de stockage BLOB Azure cible.

```csharp

using System;
using System.IO;
using System.Threading.Tasks;
using Microsoft.Azure.Documents;
using Microsoft.Azure.Documents.Client;
using Azure.Storage.Blobs;
using Azure.Storage.Blobs.Models;

namespace attachments
{
    class Program
    {
        private static string cosmosAccount = "<Your_Azure_Cosmos_account_URI>";
        private static string cosmosKey = "<Your_Azure_Cosmos_account_PRIMARY_KEY>";
        private static string cosmosDatabaseName = "<Your_Azure_Cosmos_database>";
        private static string cosmosCollectionName = "<Your_Azure_Cosmos_collection>";
        private static string storageConnectionString = "<Your_Azure_Storage_connection_string>";
        private static string storageContainerName = "<Your_Azure_Storage_container_name>";
        private static DocumentClient cosmosClient = new DocumentClient(new Uri(cosmosAccount), cosmosKey);
        private static BlobServiceClient storageClient = new BlobServiceClient(storageConnectionString);
        private static BlobContainerClient storageContainerClient = storageClient.GetBlobContainerClient(storageContainerName);

        static void Main(string[] args)
        {
            CopyAttachmentsToBlobsAsync().Wait();
        }

        private async static Task CopyAttachmentsToBlobsAsync()
        {
            Console.WriteLine("Copying Azure Cosmos DB Attachments to Azure Blob Storage ...");

            int totalCount = 0;
            string docContinuation = null;

            // Iterate through each item (document in v2) in the Azure Cosmos DB container (collection in v2) to look for attachments.
            do
            {
                FeedResponse<dynamic> response = await cosmosClient.ReadDocumentFeedAsync(
                    UriFactory.CreateDocumentCollectionUri(cosmosDatabaseName, cosmosCollectionName),
                    new FeedOptions
                    {
                        MaxItemCount = -1,
                        RequestContinuation = docContinuation
                    });
                docContinuation = response.ResponseContinuation;

                foreach (Document document in response)
                {
                    string attachmentContinuation = null;
                    PartitionKey docPartitionKey = new PartitionKey(document.Id);

                    // Iterate through each attachment within the item (if any).
                    do
                    {
                        FeedResponse<Attachment> attachments = await cosmosClient.ReadAttachmentFeedAsync(
                            document.SelfLink,
                            new FeedOptions
                            {
                                PartitionKey = docPartitionKey,
                                RequestContinuation = attachmentContinuation
                            }
                        );
                        attachmentContinuation = attachments.ResponseContinuation;

                        foreach (var attachment in attachments)
                        {
                            // Download the attachment in to local memory.
                            MediaResponse content = await cosmosClient.ReadMediaAsync(attachment.MediaLink);

                            byte[] buffer = new byte[content.ContentLength];
                            await content.Media.ReadAsync(buffer, 0, buffer.Length);

                            // Upload the locally buffered attachment to blob storage
                            string blobId = String.Concat(document.Id, "-", attachment.Id);

                            Azure.Response<BlobContentInfo> uploadedBob = await storageContainerClient.GetBlobClient(blobId).UploadAsync(
                                new MemoryStream(buffer, writable: false),
                                true
                            );

                            Console.WriteLine("Copied attachment ... Item Id: {0} , Attachment Id: {1}, Blob Id: {2}", document.Id, attachment.Id, blobId);
                            totalCount++;

                            // Clean up attachment from Azure Cosmos DB.
                            // Warning: please verify you've succesfully migrated attachments to blog storage prior to cleaning up Azure Cosmos DB.
                            // await cosmosClient.DeleteAttachmentAsync(
                            //     attachment.SelfLink,
                            //     new RequestOptions { PartitionKey = docPartitionKey }
                            // );

                            // Console.WriteLine("Cleaned up attachment ... Document Id: {0} , Attachment Id: {1}", document.Id, attachment.Id);
                        }

                    } while (!string.IsNullOrEmpty(attachmentContinuation));
                }
            }
            while (!string.IsNullOrEmpty(docContinuation));

            Console.WriteLine("Finished copying {0} attachments to blob storage", totalCount);
        }
    }
}

```

## <a name="next-steps"></a>Étapes suivantes

- Prise en main du [stockage Blob Azure](../storage/blobs/storage-quickstart-blobs-dotnet.md)
- Obtenir des références pour l’utilisation de pièces jointes via le [kit de développement logiciel (SDK) .NET v2 d’Azure Cosmos DB](/dotnet/api/microsoft.azure.documents.attachment?preserve-view=true&view=azure-dotnet)
- Obtenir des références pour l’utilisation de pièces jointes via le [kit de développement logiciel (SDK) Java v2 d’Azure Cosmos DB](/java/api/com.microsoft.azure.documentdb.attachment?preserve-view=true&view=azure-java-stable)
- Obtenir des références pour l’utilisation de pièces jointes via l’[API REST d’Azure Cosmos DB](/rest/api/cosmos-db/attachments)