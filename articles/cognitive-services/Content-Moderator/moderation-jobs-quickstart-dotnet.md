---
title: Utiliser des travaux de modération à l’aide de .NET - Content Moderator
titleSuffix: Azure Cognitive Services
description: Utilisez le SDK .NET Content Moderator pour initier des travaux de modération du contenu de bout en bout pour le contenu d’image ou de texte dans Azure Content Moderator.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: conceptual
ms.date: 10/24/2019
ms.author: pafarley
ms.custom: devx-track-csharp
ms.openlocfilehash: ec101786f33aa6f2525d685993d6b6c891ab2e9a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88936314"
---
# <a name="define-and-use-moderation-jobs-net"></a>Définir et utiliser des travaux de modération (.NET)

Un travail de modération est une forme de wrapper des fonctionnalités de modération du contenu, de flux de travail et de révision. Ce guide fournit des informations et des exemples de code qui vont vous aider à prendre en main le [kit de développement logiciel (SDK) Content Moderator pour .NET](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.ContentModerator/) afin d’effectuer les opérations suivantes :

- Lancez une tâche de modération pour analyser et créer des révisions pour des modérateurs humains
- Obtenir l’état de la révision en attente
- Suivre et obtenir l’état final de la révision
- Envoyer les résultats de la révision à l’URL de rappel

## <a name="prerequisites"></a>Conditions préalables requises

- Connectez-vous ou créez un compte sur le site de l’[outil de révision](https://contentmoderator.cognitive.microsoft.com/) de Content Moderator.

## <a name="ensure-your-api-key-can-call-the-review-api-for-review-creation"></a>Vérifier que votre clé API peut appeler l’API de révision pour la création de révisions

Une fois les étapes précédentes effectuées, il est possible que vous ayez deux clés Content Moderator si vous avez démarré à partir du portail Azure.

Si vous envisagez d’utiliser la clé API fournie par Azure dans votre exemple de SDK, suivez les étapes mentionnées dans la section [Utilisation de la clé Azure avec l’API de révision](./review-tool-user-guide/configure.md#use-your-azure-account-with-the-review-apis) pour permettre à votre application d’appeler l’API de révision et de créer des révisions.

Si vous utilisez la clé d’essai générée par l’outil de révision, votre compte d’outil de révision connaît déjà la clé ; aucune étape supplémentaire n’est donc nécessaire.

## <a name="define-a-custom-moderation-workflow"></a>Définir un workflow de modération personnalisé

Une tâche de modération analyse votre contenu à l’aide des API et utilise un **workflow** pour déterminer s’il faut créer des révisions ou non.
Bien que l’outil de révision contient un workflow par défaut, [définissons un workflow personnalisé](Review-Tool-User-Guide/Workflows.md) pour ce guide de démarrage rapide.

Vous utilisez le nom du workflow dans votre code qui démarre la tâche de modération.

## <a name="create-your-visual-studio-project"></a>Créer votre projet Visual Studio

1. Ajoutez un nouveau projet **Console app (.NET Framework)** à votre solution.

   Dans l’exemple de code, nommez le projet **CreateReviews**.

1. Sélectionnez ce projet comme unique projet de démarrage de la solution.

### <a name="install-required-packages"></a>Installer les packages nécessaires

Installez les packages NuGet suivants :

- Microsoft.Azure.CognitiveServices.ContentModerator
- Microsoft.Rest.ClientRuntime
- Newtonsoft.Json

### <a name="update-the-programs-using-statements"></a>Mettre à jour les instructions using du programme

Mettez à jour les instructions using du programme.

```csharp
using Microsoft.Azure.CognitiveServices.ContentModerator;
using Microsoft.Azure.CognitiveServices.ContentModerator.Models;
using Newtonsoft.Json;
using System;
using System.Collections.Generic;
using System.IO;
using System.Threading;
```

### <a name="create-the-content-moderator-client"></a>Créer le client Content Moderator

Ajoutez le code suivant afin de créer un client Content Moderator pour votre abonnement.

> [!IMPORTANT]
> Mettez à jour les champs **AzureEndpoint** et **CMSubscriptionKey** en leur donnant comme valeurs l’URL de votre point de terminaison et votre clé d’abonnement.

```csharp
/// <summary>
/// Wraps the creation and configuration of a Content Moderator client.
/// </summary>
/// <remarks>This class library contains insecure code. If you adapt this
/// code for use in production, use a secure method of storing and using
/// your Content Moderator subscription key.</remarks>
public static class Clients
{
    /// <summary>
    /// The base URL fragment for Content Moderator calls.
    /// </summary>
    private static readonly string AzureEndpoint = "YOUR ENDPOINT URL";

    /// <summary>
    /// Your Content Moderator subscription key.
    /// </summary>
    private static readonly string CMSubscriptionKey = "YOUR API KEY";

    /// <summary>
    /// Returns a new Content Moderator client for your subscription.
    /// </summary>
    /// <returns>The new client.</returns>
    /// <remarks>The <see cref="ContentModeratorClient"/> is disposable.
    /// When you have finished using the client,
    /// you should dispose of it either directly or indirectly. </remarks>
    public static ContentModeratorClient NewClient()
    {
        // Create and initialize an instance of the Content Moderator API wrapper.
        ContentModeratorClient client = new ContentModeratorClient(new ApiKeyServiceClientCredentials(CMSubscriptionKey));

        client.Endpoint = AzureEndpoint;
        return client;
    }
}
```

### <a name="initialize-application-specific-settings"></a>Initialiser les paramètres spécifiques de l’application

Ajoutez les constantes et champs statiques suivants à la classe **Program** dans Program.cs.

> [!NOTE]
> Vous définissez la constante TeamName par le nom utilisé lorsque vous avez créé votre abonnement à Content Moderator. Vous récupérez TeamName à partir du site Web Content Moderator.
> Une fois connecté, sélectionnez **Informations d'identification** dans le menu (engrenage) **Paramètres**.
>
> Le nom de votre équipe est la valeur du champ **Id** dans la section **API**.

```csharp
/// <summary>
/// The moderation job will use this workflow that you defined earlier.
/// See the quickstart article to learn how to setup custom workflows.
/// </summary>
private const string WorkflowName = "OCR";

/// <summary>
/// The name of the team to assign the job to.
/// </summary>
/// <remarks>This must be the team name you used to create your
/// Content Moderator account. You can retrieve your team name from
/// the Content Moderator web site. Your team name is the Id associated
/// with your subscription.</remarks>
private const string TeamName = "***";

/// <summary>
/// The URL of the image to create a review job for.
/// </summary>
private const string ImageUrl =
    "https://moderatorsampleimages.blob.core.windows.net/samples/sample5.png";

/// <summary>
/// The name of the log file to create.
/// </summary>
/// <remarks>Relative paths are relative to the execution directory.</remarks>
private const string OutputFile = "OutputLog.txt";

/// <summary>
/// The number of seconds to delay after a review has finished before
/// getting the review results from the server.
/// </summary>
private const int latencyDelay = 45;

/// <summary>
/// The callback endpoint for completed reviews.
/// </summary>
/// <remarks>Reviews show up for reviewers on your team.
/// As reviewers complete reviews, results are sent to the
/// callback endpoint using an HTTP POST request.</remarks>
private const string CallbackEndpoint = "";
```

## <a name="add-code-to-auto-moderate-create-a-review-and-get-the-job-details"></a>Ajouter du code pour modérer automatiquement, créer une révision et obtenir les détails d’une tâche

> [!Note]
> En pratique, vous définissez l’URL de rappel **CallbackEndpoint** comme URL qui recevra les résultats de la révision manuelle (via une requête HTTP POST).

Commencez par ajouter le code suivant à la méthode **Main**.

```csharp
using (TextWriter writer = new StreamWriter(OutputFile, false))
{
    using (var client = Clients.NewClient())
    {
        writer.WriteLine("Create review job for an image.");
        var content = new Content(ImageUrl);

        // The WorkflowName contains the name of the workflow defined in the online review tool.
        // See the quickstart article to learn more.
        var jobResult = client.Reviews.CreateJobWithHttpMessagesAsync(
                TeamName, "image", "contentID", WorkflowName, "application/json", content, CallbackEndpoint);

        // Record the job ID.
        var jobId = jobResult.Result.Body.JobIdProperty;

        // Log just the response body from the returned task.
        writer.WriteLine(JsonConvert.SerializeObject(
            jobResult.Result.Body, Formatting.Indented));

        Thread.Sleep(2000);
        writer.WriteLine();

        writer.WriteLine("Get review job status.");
        var jobDetails = client.Reviews.GetJobDetailsWithHttpMessagesAsync(
                TeamName, jobId);

        // Log just the response body from the returned task.
        writer.WriteLine(JsonConvert.SerializeObject(
                jobDetails.Result.Body, Formatting.Indented));

        Console.WriteLine();
        Console.WriteLine("Perform manual reviews on the Content Moderator site.");
        Console.WriteLine("Then, press any key to continue.");
        Console.ReadKey();

        Console.WriteLine();
        Console.WriteLine($"Waiting {latencyDelay} seconds for results to propagate.");
        Thread.Sleep(latencyDelay * 1000);

        writer.WriteLine("Get review details.");
        jobDetails = client.Reviews.GetJobDetailsWithHttpMessagesAsync(
        TeamName, jobId);

        // Log just the response body from the returned task.
        writer.WriteLine(JsonConvert.SerializeObject(
        jobDetails.Result.Body, Formatting.Indented));
    }
    writer.Flush();
    writer.Close();
}
```

> [!NOTE]
> Votre clé de service Content Moderator a une limite de fréquence des demandes par seconde (RPS). Si vous dépassez la limite, le kit SDK lève une exception avec le code d’erreur 429.
>
> Une clé de niveau gratuit a une limite de fréquence d’une demande par seconde (RSP).

## <a name="run-the-program-and-review-the-output"></a>Exécuter le programme et examiner la sortie

L’exemple de sortie suivante s’affiche dans la console :

```console
Perform manual reviews on the Content Moderator site.
Then, press any key to continue.
```

Connectez-vous à l’outil de révision Content Moderator pour consulter la révision d’images en attente.

Utilisez le bouton **Suivant** pour envoyer.

![Révision d’images pour les modérateurs humains](images/ocr-sample-image.PNG)

## <a name="see-the-sample-output-in-the-log-file"></a>Consulter l’exemple de sortie dans le fichier journal

> [!NOTE]
> Dans votre fichier de sortie, les chaînes **Teamname**, **ContentId**, **CallBackEndpoint** et **WorkflowId** reflètent les valeurs que vous avez utilisées précédemment.

```json
Create moderation job for an image.
{
    "JobId": "2018014caceddebfe9446fab29056fd8d31ffe"
}

Get review details.
{
    "Id": "2018014caceddebfe9446fab29056fd8d31ffe",
    "TeamName": "some team name",
    "Status": "InProgress",
    "WorkflowId": "OCR",
    "Type": "Image",
    "CallBackEndpoint": "",
    "ReviewId": "",
    "ResultMetaData": [],
    "JobExecutionReport": [
    {
        "Ts": "2018-01-07T00:38:26.7714671",
        "Msg": "Successfully got hasText response from Moderator"
    },
    {
        "Ts": "2018-01-07T00:38:26.4181346",
        "Msg": "Getting hasText from Moderator"
    },
    {
        "Ts": "2018-01-07T00:38:25.5122828",
        "Msg": "Starting Execution - Try 1"
    }
    ]
}
```

## <a name="your-callback-url-if-provided-receives-this-response"></a>Si fournie, votre URL de rappel reçoit cette réponse

Vous verrez une réponse semblable à celle-ci :

> [!NOTE]
> Dans votre réponse de rappel, les chaînes **ContentId** et **WorkflowId** reflètent les valeurs que vous avez utilisées précédemment.

```json
{
    "JobId": "2018014caceddebfe9446fab29056fd8d31ffe",
    "ReviewId": "201801i28fc0f7cbf424447846e509af853ea54",
    "WorkFlowId": "OCR",
    "Status": "Complete",
    "ContentType": "Image",
    "CallBackType": "Job",
    "ContentId": "contentID",
    "Metadata": {
        "hastext": "True",
        "ocrtext": "IF WE DID \r\nALL \r\nTHE THINGS \r\nWE ARE \r\nCAPABLE \r\nOF DOING, \r\nWE WOULD \r\nLITERALLY \r\nASTOUND \r\nOURSELVE \r\n",
        "imagename": "contentID"
    }
}
```

## <a name="next-steps"></a>Étapes suivantes

Obtenez le [SDK Content Moderator pour .NET](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.ContentModerator/) et la [solution Visual Studio](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/ContentModerator) pour ce guide de démarrage rapide et d’autres guides de démarrage rapide Content Moderator pour .NET, puis commencez votre intégration.
