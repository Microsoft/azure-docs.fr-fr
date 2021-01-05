---
title: Traiter le flux de modification dans Stockage Blob Azure | Microsoft Docs
description: Découvrez comment traiter les journaux de flux de modification dans une application cliente .NET
author: normesta
ms.author: normesta
ms.date: 09/08/2020
ms.topic: article
ms.service: storage
ms.subservice: blobs
ms.reviewer: sadodd
ms.custom: devx-track-csharp
ms.openlocfilehash: f0e89fdfba852fc056cf48efd1b92daabb272cf0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89568249"
---
# <a name="process-change-feed-in-azure-blob-storage"></a>Traiter le flux de modification dans Stockage Blob Azure

Le flux de modification fournit des journaux des transactions de toutes les modifications apportées aux objets blob et aux métadonnées d’objets blob dans votre compte de stockage. Cet article explique comment lire les enregistrements de flux de modification à l’aide de la bibliothèque du processeur de flux de modification d’objet blob.

Pour en savoir plus sur le flux de modification, consultez [Flux de modification dans Stockage Blob Azure](storage-blob-change-feed.md).

## <a name="get-the-blob-change-feed-processor-library"></a>Obtenir la bibliothèque du processeur de flux de modification d’objet blob

1. Ouvrez une fenêtre de commande (par exemple : Windows PowerShell).
2. À partir du répertoire de votre projet, installez le package NuGet [**Azure.Storage.Blobs.Changefeed**](https://www.nuget.org/packages/Azure.Storage.Blobs.ChangeFeed/).

```console
dotnet add package Azure.Storage.Blobs --version 12.5.1
dotnet add package Azure.Storage.Blobs.ChangeFeed --version 12.0.0-preview.4
```
## <a name="read-records"></a>Lire les enregistrements

> [!NOTE]
> Le flux de modification est une entité immuable et en lecture seule dans votre compte de stockage. Un nombre quelconque d’applications peut lire et traiter le flux de modification simultanément et de manière indépendante. Les enregistrements ne sont pas supprimés du flux de modification quand une application les lit. L’état de lecture ou d’itération de chaque lecteur de consommation est indépendant et géré uniquement par votre application.

Cet exemple procède à l'itération de tous les enregistrements du flux de modification, les ajoute à une liste, puis renvoie cette liste à l'appelant.
 
```csharp
public async Task<List<BlobChangeFeedEvent>> ChangeFeedAsync(string connectionString)
{
    // Get a new blob service client.
    BlobServiceClient blobServiceClient = new BlobServiceClient(connectionString);

    // Get a new change feed client.
    BlobChangeFeedClient changeFeedClient = blobServiceClient.GetChangeFeedClient();

    List<BlobChangeFeedEvent> changeFeedEvents = new List<BlobChangeFeedEvent>();

    // Get all the events in the change feed. 
    await foreach (BlobChangeFeedEvent changeFeedEvent in changeFeedClient.GetChangesAsync())
    {
        changeFeedEvents.Add(changeFeedEvent);
    }

    return changeFeedEvents;
}
```

Cet exemple imprime sur la console quelques valeurs de chaque enregistrement de la liste. 

```csharp
public void showEventData(List<BlobChangeFeedEvent> changeFeedEvents)
{
    foreach (BlobChangeFeedEvent changeFeedEvent in changeFeedEvents)
    {
        string subject = changeFeedEvent.Subject;
        string eventType = changeFeedEvent.EventType.ToString();
        string api = changeFeedEvent.EventData.Api;

        Console.WriteLine("Subject: " + subject + "\n" +
        "Event Type: " + eventType + "\n" +
        "Api: " + api);
    }
}
```

## <a name="resume-reading-records-from-a-saved-position"></a>Reprendre la lecture des enregistrements à partir d'une position enregistrée

Vous pouvez choisir d'enregistrer votre position de lecture dans le flux de modification, puis de reprendre ultérieurement l'itération des enregistrements. Vous pouvez enregistrer la position de lecture en obtenant le curseur du flux de modification. Le curseur est une **chaîne** et votre application peut enregistrer cette chaîne de la manière qui convient le mieux à la conception de votre application (par exemple : dans un fichier ou une base de données).

Cet exemple procède à l'itération de tous les enregistrements du flux de modification, les ajoute à une liste et enregistre le curseur. La liste et le curseur sont renvoyés à l'appelant. 

```csharp
public async Task<(string, List<BlobChangeFeedEvent>)> ChangeFeedResumeWithCursorAsync
    (string connectionString,  string cursor)
{
    // Get a new blob service client.
    BlobServiceClient blobServiceClient = new BlobServiceClient(connectionString);

    // Get a new change feed client.
    BlobChangeFeedClient changeFeedClient = blobServiceClient.GetChangeFeedClient();
    List<BlobChangeFeedEvent> changeFeedEvents = new List<BlobChangeFeedEvent>();

    IAsyncEnumerator<Page<BlobChangeFeedEvent>> enumerator = changeFeedClient
        .GetChangesAsync(continuation: cursor)
        .AsPages(pageSizeHint: 10)
        .GetAsyncEnumerator();

    await enumerator.MoveNextAsync();

    foreach (BlobChangeFeedEvent changeFeedEvent in enumerator.Current.Values)
    {
    
        changeFeedEvents.Add(changeFeedEvent);             
    }
    
    // Update the change feed cursor.  The cursor is not required to get each page of events,
    // it is intended to be saved and used to resume iterating at a later date.
    cursor = enumerator.Current.ContinuationToken;
    return (cursor, changeFeedEvents);
}
```

## <a name="stream-processing-of-records"></a>Traitement des flux de données d’enregistrement

Vous pouvez choisir de traiter les enregistrements du flux de modification à mesure qu'ils sont validés dans le flux de modification. Voir [Spécifications](storage-blob-change-feed.md#specifications). Les événements de modification sont publiés dans le flux de modification à une fréquence de 60 secondes en moyenne. Nous vous recommandons d'interroger les nouvelles modifications en tenant compte de cette fréquence lorsque vous spécifiez votre intervalle d'interrogation.

Cet exemple interroge périodiquement les modifications.  S'il existe des enregistrements de modification, ce code les traite et enregistre le curseur du flux de modification. Ainsi, si le processus est arrêté puis redémarré, l'application peut utiliser le curseur pour reprendre le traitement des enregistrements là où il s'était interrompu. Cet exemple enregistre le curseur dans le fichier de configuration d'une application locale, mais votre application peut l'enregistrer sous la forme qui convient le mieux à votre scénario. 

```csharp
public async Task ChangeFeedStreamAsync
    (string connectionString, int waitTimeMs, string cursor)
{
    // Get a new blob service client.
    BlobServiceClient blobServiceClient = new BlobServiceClient(connectionString);

    // Get a new change feed client.
    BlobChangeFeedClient changeFeedClient = blobServiceClient.GetChangeFeedClient();

    while (true)
    {
        IAsyncEnumerator<Page<BlobChangeFeedEvent>> enumerator = changeFeedClient
        .GetChangesAsync(continuation: cursor).AsPages().GetAsyncEnumerator();

        while (true) 
        {
            var result = await enumerator.MoveNextAsync();

            if (result)
            {
                foreach (BlobChangeFeedEvent changeFeedEvent in enumerator.Current.Values)
                {
                    string subject = changeFeedEvent.Subject;
                    string eventType = changeFeedEvent.EventType.ToString();
                    string api = changeFeedEvent.EventData.Api;

                    Console.WriteLine("Subject: " + subject + "\n" +
                        "Event Type: " + eventType + "\n" +
                        "Api: " + api);
                }
            
                // helper method to save cursor. 
                SaveCursor(enumerator.Current.ContinuationToken);
            }
            else
            {
                break;
            }

        }
        await Task.Delay(waitTimeMs);
    }

}

public void SaveCursor(string cursor)
{
    System.Configuration.Configuration config = 
        ConfigurationManager.OpenExeConfiguration
        (ConfigurationUserLevel.None);

    config.AppSettings.Settings.Clear();
    config.AppSettings.Settings.Add("Cursor", cursor);
    config.Save(ConfigurationSaveMode.Modified);
}
```

## <a name="reading-records-within-a-time-range"></a>Lecture d’enregistrements dans un intervalle de temps

Vous pouvez lire des enregistrements situés dans un intervalle de temps spécifique. Cet exemple procède à l'itération de tous les enregistrements du flux de modification qui se situent entre 15h00 le 2 mars 2020 et 2h00 le 7 août 2020, les ajoute à une liste, puis renvoie cette liste à l'appelant.

### <a name="selecting-segments-for-a-time-range"></a>Sélection de segments pour un intervalle de temps

```csharp
public async Task<List<BlobChangeFeedEvent>> ChangeFeedBetweenDatesAsync(string connectionString)
{
    // Get a new blob service client.
    BlobServiceClient blobServiceClient = new BlobServiceClient(connectionString);

    // Get a new change feed client.
    BlobChangeFeedClient changeFeedClient = blobServiceClient.GetChangeFeedClient();
    List<BlobChangeFeedEvent> changeFeedEvents = new List<BlobChangeFeedEvent>();

    // Create the start and end time.  The change feed client will round start time down to
    // the nearest hour, and round endTime up to the next hour if you provide DateTimeOffsets
    // with minutes and seconds.
    DateTimeOffset startTime = new DateTimeOffset(2020, 3, 2, 15, 0, 0, TimeSpan.Zero);
    DateTimeOffset endTime = new DateTimeOffset(2020, 8, 7, 2, 0, 0, TimeSpan.Zero);

    // You can also provide just a start or end time.
    await foreach (BlobChangeFeedEvent changeFeedEvent in changeFeedClient.GetChangesAsync(
        start: startTime,
        end: endTime))
    {
        changeFeedEvents.Add(changeFeedEvent);
    }

    return changeFeedEvents;
}
```

L'heure de début que vous indiquez est arrondie à l'heure inférieure, et l'heure de fin est arrondie à l'heure supérieure. Les utilisateurs peuvent donc voir des événements survenus avant l'heure de début et après l'heure de fin. Il est également possible que certains événements survenus entre l'heure de début et l'heure de fin n'apparaissent pas. En effet, des événements peuvent être enregistrés pendant l'heure précédant l'heure de début ou pendant l'heure suivant l'heure de fin.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les journaux de flux de modification. Consultez [Flux de modification dans Stockage Blob Azure](storage-blob-change-feed.md)
