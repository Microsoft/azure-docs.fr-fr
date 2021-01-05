---
title: Créer des révisions de vidéos à l’aide de .NET - Content Moderator
titleSuffix: Azure Cognitive Services
description: Cet article fournit des informations et des exemples de code destinés à vous aider à démarrer rapidement avec le SDK Content Moderator avec C# pour créer des révisions de vidéos.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: conceptual
ms.date: 10/24/2019
ms.author: pafarley
ms.custom: devx-track-csharp
ms.openlocfilehash: d9f80de5a18e27de4a9f8e85613e3c2eee6c111c
ms.sourcegitcommit: 48cb2b7d4022a85175309cf3573e72c4e67288f5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96853473"
---
# <a name="create-video-reviews-using-net"></a>Créer des révisions de vidéos à l’aide de .NET

Cet article fournit des informations et des exemples de code destinés à vous aider à démarrer rapidement avec le [SDK Content Moderator avec C#](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.ContentModerator/) pour :

- Créer une révision de vidéo pour des modérateurs humains
- Ajouter des trames à une révision
- Obtenir les trames pour la révision
- Obtenir l’état et les détails de la révision
- Publier la révision

## <a name="prerequisites"></a>Conditions préalables requises

- Connectez-vous ou créez un compte sur le site de l’[outil de révision](https://contentmoderator.cognitive.microsoft.com/) de Content Moderator.
- Cet article suppose que vous avez [modéré la vidéo (consultez le guide de démarrage rapide )](video-moderation-api.md) et que vous disposez des données de la réponse. Vous en avez besoin pour la création de révisions par trames pour les modérateurs humains.

## <a name="ensure-your-api-key-can-call-the-review-api-for-review-creation"></a>Vérifier que votre clé API peut appeler l’API de révision pour la création de révisions

Une fois les étapes précédentes effectuées, il est possible que vous ayez deux clés Content Moderator si vous avez démarré à partir du portail Azure.

Si vous envisagez d’utiliser la clé API fournie par Azure dans votre exemple de SDK, suivez les étapes mentionnées dans la section [Utilisation de la clé Azure avec l’API de révision](review-tool-user-guide/configure.md#use-your-azure-account-with-the-review-apis) pour permettre à votre application d’appeler l’API de révision et de créer des révisions.

Si vous utilisez la clé d’essai générée par l’outil de révision, votre compte d’outil de révision connaît déjà la clé ; aucune étape supplémentaire n’est donc nécessaire.

### <a name="prepare-your-video-and-the-video-frames-for-review"></a>Préparer votre vidéo et les trames vidéo pour la révision

La vidéo et les exemples de trames vidéo à réviser doivent être publiés en ligne, car vous avez besoin de leurs URL.

> [!NOTE]
> Le programme utilise des captures d’écran enregistrées manuellement à partir de la vidéo avec des scores relatifs au contenu osé/pour adulte aléatoires pour illustrer l’utilisation de l’API de révision. Dans une situation réelle, vous utilisez la [sortie de la modération des vidéos](video-moderation-api.md#run-the-program-and-review-the-output) pour créer des images et affecter des scores. 

Pour la vidéo, vous avez besoin d’un point de terminaison de streaming afin que l’outil de révision lise la vidéo dans la vue du lecteur.

![Miniature de la démonstration de vidéo](images/ams-video-demo-view.PNG)

- Copiez **l’URL** sur cette page de [démonstration Azure Media Services](https://aka.ms/azuremediaplayer?url=https%3A%2F%2Famssamples.streaming.mediaservices.windows.net%2F91492735-c523-432b-ba01-faba6c2206a2%2FAzureMediaServicesPromo.ism%2Fmanifest) pour l’URL du manifeste.

Pour les trames vidéo (images), utilisez les images suivantes :

![Miniature de trame vidéo 1](images/ams-video-frame-thumbnails-1.PNG) | ![Miniature de trame vidéo 2](images/ams-video-frame-thumbnails-2.PNG) | ![Miniature de trame vidéo 3](images/ams-video-frame-thumbnails-3.PNG) |
| :---: | :---: | :---: |
[Trame 1](https://blobthebuilder.blob.core.windows.net/sampleframes/ams-video-frame1-00-17.PNG) | [Trame 2](https://blobthebuilder.blob.core.windows.net/sampleframes/ams-video-frame-2-01-04.PNG) | [Trame 3](https://blobthebuilder.blob.core.windows.net/sampleframes/ams-video-frame-3-02-24.PNG) |

## <a name="create-your-visual-studio-project"></a>Créer votre projet Visual Studio

1. Ajoutez un nouveau projet **Console app (.NET Framework)** à votre solution.

1. Nommez le projet **VideoReviews**.

1. Sélectionnez ce projet comme unique projet de démarrage de la solution.

### <a name="install-required-packages"></a>Installer les packages nécessaires

Installez les packages NuGet suivants pour le projet TermLists.

- Microsoft.Azure.CognitiveServices.ContentModerator
- Microsoft.Rest.ClientRuntime
- Microsoft.Rest.ClientRuntime.Azure
- Newtonsoft.Json

### <a name="update-the-programs-using-statements"></a>Mettre à jour les instructions using du programme

Modifiez les instructions using du programme comme suit.

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Threading;
using Microsoft.Azure.CognitiveServices.ContentModerator;
using Microsoft.Azure.CognitiveServices.ContentModerator.Models;
using Newtonsoft.Json;
```

### <a name="add-private-properties"></a>Ajouter des propriétés privées

Ajoutez les propriétés privées suivantes à l’espace de noms **VideoReviews** de la classe **Program**. Mettez à jour les champs `AzureEndpoint` et `CMSubscriptionKey` avec les valeurs de votre URL de point de terminaison et de votre clé d’abonnement. Vous pouvez les trouver sous l’onglet **Démarrage rapide** de votre ressource dans le Portail Azure.


```csharp
namespace VideoReviews
{
    class Program
    {
        // NOTE: Enter a valid endpoint URL
        /// <summary>
        /// The endpoint URL of your subscription
        /// </summary>
        private static readonly string AzureEndpoint = "YOUR ENDPOINT URL";

        // NOTE: Enter a valid subscription key.
        /// <summary>
        /// Your Content Moderator subscription key.
        /// </summary>
        private static readonly string CMSubscriptionKey = "YOUR CONTENT MODERATOR KEY";

        // NOTE: Replace this example team name with your Content Moderator team name.
        /// <summary>
        /// The name of the team to assign the job to.
        /// </summary>
        /// <remarks>This must be the team name you used to create your 
        /// Content Moderator account. You can retrieve your team name from
        /// the Content Moderator web site. Your team name is the Id associated 
        /// with your subscription.</remarks>
        private const string TeamName = "YOUR CONTENT MODERATOR TEAM ID";

        /// <summary>
        /// The minimum amount of time, in milliseconds, to wait between calls
        /// to the Content Moderator APIs.
        /// </summary>
        private const int throttleRate = 2000;
```

### <a name="create-content-moderator-client-object"></a>Créer un objet Client Content Moderator

Ajoutez la définition de méthode suivante à l’espace de noms **VideoReviews** de la classe **Program**.

```csharp
/// <summary>
/// Returns a new Content Moderator client for your subscription.
/// </summary>
/// <returns>The new client.</returns>
/// <remarks>The <see cref="ContentModeratorClient"/> is disposable.
/// When you have finished using the client,
/// you should dispose of it either directly or indirectly. </remarks>
public static ContentModeratorClient NewClient()
{
    return new ContentModeratorClient(new ApiKeyServiceClientCredentials(CMSubscriptionKey))
    {
        Endpoint = AzureEndpoint
    };
}
```

## <a name="create-a-video-review"></a>Créer une révision de vidéo

Créez une révision de vidéo avec **ContentModeratorClient.Reviews.CreateVideoReviews**. Pour plus d’informations, consultez les [informations de référence sur l’API](https://westus.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/580519483f9b0709fc47f9c4).

**CreateVideoReviews** a les paramètres obligatoires suivants :
1. Une chaîne contenant un type MIME, qui doit être « application/json ». 
1. Le nom de votre équipe Content Moderator.
1. Un objet **IList\<CreateVideoReviewsBodyItem>**. Chaque objet **CreateVideoReviewsBodyItem** représente une révision de vidéo. Ce guide de démarrage rapide crée une révision à la fois.

**CreateVideoReviewsBodyItem** a plusieurs propriétés. Vous définissez au moins les propriétés suivantes :
- **Content**. URL de la vidéo à réviser.
- **ContentId**. Un ID à affecter à la révision de la vidéo.
- **Status**. Définissez la valeur sur « Unpublished ». Si vous ne définissez pas cette propriété, sa valeur par défaut est « Pending », ce qui signifie que la révision de la vidéo est publiée et en attente d’une révision par un opérateur humain. Une fois qu’une révision de vidéo est publiée, vous ne pouvez plus y ajouter de trames vidéo, de transcription ou de résultat de la modération des transcriptions.

> [!NOTE]
> **CreateVideoReviews** retourne un IList\<string>. Chacune de ces chaînes contient un ID de révision de vidéo. Ces ID sont des GUID et sont différents de la valeur de la propriété **ContentId**. 

Ajoutez la définition de méthode suivante à la classe Program de l’espace de noms VideoReviews.

```csharp
/// <summary>
/// Create a video review. For more information, see the API reference:
/// https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/580519483f9b0709fc47f9c4 
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="id">The ID to assign to the video review.</param>
/// <param name="content">The URL of the video to review.</param>
/// <returns>The ID of the video review.</returns>
private static string CreateReview(ContentModeratorClient client, string id, string content)
{
    Console.WriteLine("Creating a video review.");

    List<CreateVideoReviewsBodyItem> body = new List<CreateVideoReviewsBodyItem>() {
        new CreateVideoReviewsBodyItem
        {
            Content = content,
            ContentId = id,
            /* Note: to create a published review, set the Status to "Pending".
            However, you cannot add video frames or a transcript to a published review. */
            Status = "Unpublished",
        }
    };

    var result = client.Reviews.CreateVideoReviews("application/json", TeamName, body);

    Thread.Sleep(throttleRate);

    // We created only one review.
    return result[0];
}
```

> [!NOTE]
> Votre clé de service Content Moderator a une limite de fréquence des requêtes par seconde. Si vous dépassez cette limite, le Kit de développement logiciel (SDK) lève une exception avec le code d’erreur 429.
>
> Une clé de niveau gratuit a une limite de fréquence d’une demande par seconde (RSP).

## <a name="add-video-frames-to-the-video-review"></a>Ajouter des trames vidéo à la révision de la vidéo

Vous ajoutez des trames vidéo à une révision de vidéo avec **ContentModeratorClient.Reviews.AddVideoFrameUrl** (si vos trames vidéo sont hébergées en ligne) ou **ContentModeratorClient.Reviews.AddVideoFrameStream** (si vos trames vidéo sont hébergées localement). Ce guide de démarrage rapide suppose que vos trames vidéo sont hébergées en ligne, et utilise donc **AddVideoFrameUrl**. Pour plus d’informations, consultez les [informations de référence sur l’API](https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/59e7b76ae7151f0b10d451fd).

**AddVideoFrameUrl** a les paramètres obligatoires suivants :
1. Une chaîne contenant un type MIME, qui doit être « application/json ».
1. Le nom de votre équipe Content Moderator.
1. L’ID de révision de vidéo retourné par **CreateVideoReviews**.
1. Un objet **IList\<VideoFrameBodyItem>**. Chaque objet **VideoFrameBodyItem** représente une trame vidéo.

**VideoFrameBodyItem** a les propriétés suivantes :
- **Timestamp**. Chaîne qui contient, en secondes, l’heure dans la vidéo à partir de laquelle la trame vidéo a été prise.
- **FrameImage**. URL de la trame vidéo.
- **Metadata**. Un IList\<VideoFrameBodyItemMetadataItem>. **VideoFrameBodyItemMetadataItem** est simplement une paire clé/valeur. Les clés valides sont notamment les suivantes :
- **reviewRecommended**. True si une révision par un opérateur humain de la trame vidéo est recommandée.
- **adultScore**. Valeur comprise entre 0 et 1 qui évalue la gravité du contenu pour adultes dans la trame vidéo.
- **a**. True si la vidéo contient du contenu pour adultes.
- **racyScore**. Valeur comprise entre 0 et 1 qui évalue la gravité du contenu osé dans la trame vidéo.
- **r**. True si la trame vidéo contient du contenu osé.
- **ReviewerResultTags**. Un IList\<VideoFrameBodyItemReviewerResultTagsItem>. **VideoFrameBodyItemReviewerResultTagsItem** est simplement une paire clé/valeur. Une application peut utiliser ces balises pour organiser des trames vidéo.

> [!NOTE]
> Ce guide de démarrage rapide génère des valeurs aléatoires pour les propriétés **adultScore** et **racyScore**. Dans une application de production, vous devez obtenir ces valeurs à partir du [service de modération des vidéos](video-moderation-api.md), déployé comme un espace Azure Media Service.

Ajoutez les définitions de méthode suivantes à la classe Program de l’espace de noms VideoReviews.

```csharp
<summary>
/// Create a video frame to add to a video review after the video review is created.
/// </summary>
/// <param name="url">The URL of the video frame image.</param>
/// <returns>The video frame.</returns>
private static VideoFrameBodyItem CreateFrameToAddToReview(string url, string timestamp_seconds)
{
    // We generate random "adult" and "racy" scores for the video frame.
    Random rand = new Random();

    var frame = new VideoFrameBodyItem
    {
        // The timestamp is measured in milliseconds. Convert from seconds.
        Timestamp = (int.Parse(timestamp_seconds) * 1000).ToString(),
        FrameImage = url,

        Metadata = new List<VideoFrameBodyItemMetadataItem>
        {
            new VideoFrameBodyItemMetadataItem("reviewRecommended", "true"),
            new VideoFrameBodyItemMetadataItem("adultScore", rand.NextDouble().ToString()),
            new VideoFrameBodyItemMetadataItem("a", "false"),
            new VideoFrameBodyItemMetadataItem("racyScore", rand.NextDouble().ToString()),
            new VideoFrameBodyItemMetadataItem("r", "false")
        },

        ReviewerResultTags = new List<VideoFrameBodyItemReviewerResultTagsItem>()
        {
            new VideoFrameBodyItemReviewerResultTagsItem("tag1", "value1")
        }
    };

    return frame;
}
```

```csharp
/// <summary>
/// Add a video frame to the indicated video review. For more information, see the API reference:
/// https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/59e7b76ae7151f0b10d451fd
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="review_id">The video review ID.</param>
/// <param name="url">The URL of the video frame image.</param>
static void AddFrame(ContentModeratorClient client, string review_id, string url, string timestamp_seconds)
{
    Console.WriteLine("Adding a frame to the review with ID {0}.", review_id);

    var frames = new List<VideoFrameBodyItem>()
    {
        CreateFrameToAddToReview(url, timestamp_seconds)
    };
        
    client.Reviews.AddVideoFrameUrl("application/json", TeamName, review_id, frames);

    Thread.Sleep(throttleRate);
```

## <a name="get-video-frames-for-video-review"></a>Obtenir des trames vidéo pour une révision de vidéo

Vous pouvez obtenir les trames vidéo pour une révision de vidéo avec **ContentModeratorClient.Reviews.GetVideoFrames**. **GetVideoFrames** a les paramètres obligatoires suivants :
1. Le nom de votre équipe Content Moderator.
1. L’ID de révision de vidéo retourné par **CreateVideoReviews**.
1. Index de base zéro de la première trame vidéo à obtenir.
1. Nombre de trames vidéo à obtenir.

Ajoutez la définition de méthode suivante à la classe Program de l’espace de noms VideoReviews.

```csharp
/// <summary>
/// Get the video frames assigned to the indicated video review.  For more information, see the API reference:
/// https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/59e7ba43e7151f0b10d45200
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="review_id">The video review ID.</param>
static void GetFrames(ContentModeratorClient client, string review_id)
{
    Console.WriteLine("Getting frames for the review with ID {0}.", review_id);

    Frames result = client.Reviews.GetVideoFrames(TeamName, review_id, 0);
    Console.WriteLine(JsonConvert.SerializeObject(result, Formatting.Indented));

    Thread.Sleep(throttleRate);
}
```

## <a name="get-video-review-information"></a>Obtenir des informations de révision de vidéo

Vous obtenez les informations concernant une révision de vidéo avec **ContentModeratorClient.Reviews.GetReview**. **GetReview** a les paramètres obligatoires suivants :
1. Le nom de votre équipe Content Moderator.
1. L’ID de révision de vidéo retourné par **CreateVideoReviews**.

Ajoutez la définition de méthode suivante à la classe Program de l’espace de noms VideoReviews.

```csharp
/// <summary>
/// Get the information for the indicated video review. For more information, see the reference API:
/// https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/580519483f9b0709fc47f9c2
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="review_id">The video review ID.</param>
private static void GetReview(ContentModeratorClient client, string review_id)
{
    Console.WriteLine("Getting the status for the review with ID {0}.", review_id);

    var result = client.Reviews.GetReview(ModeratorHelper.Clients.TeamName, review_id);
    Console.WriteLine(JsonConvert.SerializeObject(result, Formatting.Indented));

    Thread.Sleep(throttleRate);
}
```

## <a name="publish-video-review"></a>Publier une révision de vidéo

Vous publiez une révision de vidéo avec **ContentModeratorClient.Reviews.PublishVideoReview**. **PublishVideoReview** a les paramètres obligatoires suivants :
1. Le nom de votre équipe Content Moderator.
1. L’ID de révision de vidéo retourné par **CreateVideoReviews**.

Ajoutez la définition de méthode suivante à la classe Program de l’espace de noms VideoReviews.

```csharp
/// <summary>
/// Publish the indicated video review. For more information, see the reference API:
/// https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/59e7bb29e7151f0b10d45201
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="review_id">The video review ID.</param>
private static void PublishReview(ContentModeratorClient client, string review_id)
{
    Console.WriteLine("Publishing the review with ID {0}.", review_id);
    client.Reviews.PublishVideoReview(TeamName, review_id);
    Thread.Sleep(throttleRate);
}
```

## <a name="putting-it-all-together"></a>Exemple complet

Ajoutez la définition de méthode **Main** à la classe Program de l’espace de noms VideoReviews. Pour finir, fermez la classe Program et l’espace de noms VideoReviews.

```csharp
static void Main(string[] args)
{
    using (ContentModeratorClient client = NewClient())
    {
        // Create a review with the content pointing to a streaming endpoint (manifest)
        var streamingcontent = "https://amssamples.streaming.mediaservices.windows.net/91492735-c523-432b-ba01-faba6c2206a2/AzureMediaServicesPromo.ism/manifest";
        string review_id = CreateReview(client, "review1", streamingcontent);

        var frame1_url = "https://blobthebuilder.blob.core.windows.net/sampleframes/ams-video-frame1-00-17.PNG";
        var frame2_url = "https://blobthebuilder.blob.core.windows.net/sampleframes/ams-video-frame-2-01-04.PNG";
        var frame3_url = "https://blobthebuilder.blob.core.windows.net/sampleframes/ams-video-frame-3-02-24.PNG";

        // Add the frames from 17, 64, and 144 seconds.
        AddFrame(client, review_id, frame1_url, "17");
        AddFrame(client, review_id, frame2_url, "64");
        AddFrame(client, review_id, frame3_url, "144");

        // Get frames information and show
        GetFrames(client, review_id);
        GetReview(client, review_id);

        // Publish the review
        PublishReview(client, review_id);

        Console.WriteLine("Open your Content Moderator Dashboard and select Review > Video to see the review.");
        Console.WriteLine("Press any key to close the application.");
        Console.ReadKey();
    }
}
```

## <a name="run-the-program-and-review-the-output"></a>Exécuter le programme et examiner la sortie
Quand vous exécutez l’application, une sortie s’affiche sur les lignes suivantes :

```json
Creating a video review.
Adding a frame to the review with ID 201801v3212bda70ced4928b2cd7459c290c7dc.
Adding a frame to the review with ID 201801v3212bda70ced4928b2cd7459c290c7dc.
Adding a frame to the review with ID 201801v3212bda70ced4928b2cd7459c290c7dc.
Getting frames for the review with ID 201801v3212bda70ced4928b2cd7459c290c7dc.
{
    "ReviewId": "201801v3212bda70ced4928b2cd7459c290c7dc",
    "VideoFrames": [
    {
        "Timestamp": "17000",
        "FrameImage": "https://reviewcontentprod.blob.core.windows.net/testreview6/FRM_201801v3212bda70ced4928b2cd7459c290c7dc_17000.PNG",
        "Metadata": [
        {
            "Key": "reviewRecommended",
            "Value": "true"
        },
        {
            "Key": "adultScore",
            "Value": "0.808312381528463"
        },
        {
            "Key": "a",
            "Value": "false"
        },
        {
            "Key": "racyScore",
            "Value": "0.846378884206702"
        },
        {
            "Key": "r",
            "Value": "false"
        }
        ],
        "ReviewerResultTags": [
        {
            "Key": "tag1",
            "Value": "value1"
        }
    ]
    },
    {
        "Timestamp": "64000",
        "FrameImage": "https://reviewcontentprod.blob.core.windows.net/testreview6/FRM_201801v3212bda70ced4928b2cd7459c290c7dc_64000.PNG",
        "Metadata": [
        {
            "Key": "reviewRecommended",
            "Value": "true"
        },
        {
            "Key": "adultScore",
            "Value": "0.576078300166912"
        },
        {
            "Key": "a",
            "Value": "false"
        },
        {
            "Key": "racyScore",
            "Value": "0.244768953064815"
        },
        {
            "Key": "r",
            "Value": "false"
        }
        ],
        "ReviewerResultTags": [
        {
            "Key": "tag1",
            "Value": "value1"
        }
    ]
    },
    {
        "Timestamp": "144000",
        "FrameImage": "https://reviewcontentprod.blob.core.windows.net/testreview6/FRM_201801v3212bda70ced4928b2cd7459c290c7dc_144000.PNG",
        "Metadata": [
        {
            "Key": "reviewRecommended",
            "Value": "true"
        },
        {
            "Key": "adultScore",
            "Value": "0.664480847150311"
        },
        {
            "Key": "a",
            "Value": "false"
        },
        {
            "Key": "racyScore",
            "Value": "0.933817870418456"
        },
        {
            "Key": "r",
            "Value": "false"
        }
        ],
        "ReviewerResultTags": [
        {
            "Key": "tag1",
            "Value": "value1"
        }
        ]
    }
    ]
}

Getting the status for the review with ID 201801v3212bda70ced4928b2cd7459c290c7dc.
{
    "ReviewId": "201801v3212bda70ced4928b2cd7459c290c7dc",
    "SubTeam": "public",
    "Status": "UnPublished",
    "ReviewerResultTags": [],
    "CreatedBy": "testreview6",
    "Metadata": [
    {
        "Key": "FrameCount",
        "Value": "3"
    }
    ],
    "Type": "Video",
    "Content": "https://amssamples.streaming.mediaservices.windows.net/91492735-c523-432b-ba01-faba6c2206a2/AzureMediaServicesPromo.ism/manifest",
    "ContentId": "review1",
    "CallbackEndpoint": null
}

Publishing the review with ID 201801v3212bda70ced4928b2cd7459c290c7dc.
Open your Content Moderator Dashboard and select Review > Video to see the review.
Press any key to close the application.
```

## <a name="check-out-your-video-review"></a>Consulter votre révision de vidéo

Enfin, la révision de vidéo s’affiche dans votre compte de l’outil de révision Content Moderator, dans l’écran **Révision**>**Vidéo**.

![Révision de vidéo pour les modérateurs humains](images/ams-video-review.PNG)

## <a name="next-steps"></a>Étapes suivantes

Obtenez le [SDK Content Moderator pour .NET](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.ContentModerator/) et la [solution Visual Studio](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/ContentModerator) pour ce guide de démarrage rapide et d’autres guides de démarrage rapide Content Moderator pour .NET.
