---
title: Analyser des vidéos en quasi temps réel - Vision par ordinateur
titleSuffix: Azure Cognitive Services
description: Découvrez comment effectuer une analyse en quasi temps réel des images d’un flux vidéo en direct avec l’API Vision par ordinateur.
services: cognitive-services
author: KellyDF
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: sample
ms.date: 09/09/2019
ms.author: kefre
ms.custom: seodec18, devx-track-csharp
ms.openlocfilehash: c86b06ff46f1ddc8d22d2ab7ec4bc8620a8c862f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88933169"
---
# <a name="analyze-videos-in-near-real-time"></a>Analyser des vidéos en quasi temps réel

Cet article explique comment effectuer une analyse en quasi temps réel des images d’un flux vidéo en direct avec l’API Vision par ordinateur. Les éléments de base d’une telle analyse sont les suivants :

- Acquisition d’images à partir d’une source vidéo.
- Sélection des images à analyser.
- Envoi de ces images à l’API.
- Utilisation de chaque résultat de l’analyse retourné par l’appel de l’API.

Les exemples de cet article sont écrits en C#. Pour voir le code, accédez à la page [Exemple d’analyse d’images vidéo](https://github.com/Microsoft/Cognitive-Samples-VideoFrameAnalysis/) sur GitHub.

## <a name="approaches-to-running-near-real-time-analysis"></a>Approches de l’exécution de l’analyse en quasi temps réel

Vous pouvez résoudre le problème de l’exécution d’une analyse en quasi temps réel sur des flux vidéo avec différentes approches. Cet article décrit trois d’entre elles, selon un niveau de sophistication croissant.

### <a name="design-an-infinite-loop"></a>Concevoir une boucle infinie

La conception la plus simple pour l’analyse en quasi temps réel est une boucle infinie. Dans chaque itération de cette boucle, vous prenez une image, vous l’analysez, puis vous utilisez le résultat :

```csharp
while (true)
{
    Frame f = GrabFrame();
    if (ShouldAnalyze(f))
    {
        AnalysisResult r = await Analyze(f);
        ConsumeResult(r);
    }
}
```

Si notre analyse était constituée d’un algorithme léger côté client, cette approche conviendrait. Cependant, quand l’analyse se fait dans le cloud, la latence résultante signifie qu’un appel d’API peut prendre plusieurs secondes. Durant ce laps de temps, vous ne capturez pas d’images et votre thread ne fait rien. Votre fréquence maximale d’images est limitée par la latence des appels d’API.

### <a name="allow-the-api-calls-to-run-in-parallel"></a>Autoriser l’exécution des appels d’API en parallèle

Si une simple boucle monothread convient pour un algorithme léger côté client, elle ne convient pas bien pour la latence d’un appel d’API cloud. La solution à ce problème consiste à autoriser l’exécution de l’appel d’API d’exécution longue en parallèle avec la capture des images. En C#, vous pouvez faire cela en utilisant le parallélisme basé sur les tâches. Par exemple, vous pouvez exécuter le code suivant :

```csharp
while (true)
{
    Frame f = GrabFrame();
    if (ShouldAnalyze(f))
    {
        var t = Task.Run(async () =>
        {
            AnalysisResult r = await Analyze(f);
            ConsumeResult(r);
        }
    }
}
```

Avec cette approche, vous lancez chaque analyse dans une tâche distincte. La tâche peut s’exécuter en arrière-plan pendant que vous continuez à capturer de nouvelles images. L’approche évite de bloquer le thread principal pendant que vous attendez le retour d’un appel d’API. Cependant, l’approche peut présenter certains inconvénients :
* Cela vous coûte une partie des garanties fournies par la version simple. En l’occurrence, plusieurs appels d’API peuvent avoir lieu en parallèle, et les résultats peuvent être retournés dans un ordre incorrect. 
* De plus, plusieurs threads risquent d’entrer dans la fonction ConsumeResult() simultanément, ce qui peut être dangereux si la fonction n’est pas thread-safe. 
* Enfin, ce code simple n’effectuant pas le suivi des tâches créées, les exceptions vont disparaître sans se manifester. Vous devez donc ajouter un thread « consommateur » qui effectue le suivi des tâches d’analyse, lève des exceptions, tue les tâches à exécution longue et vérifie que les résultats sont utilisés dans le bon ordre, un à la fois.

### <a name="design-a-producer-consumer-system"></a>Concevoir un système producteur-consommateur

Pour votre approche finale, qui est la conception d’un système « producteur-consommateur », vous créez un thread producteur qui est similaire à la boucle infinie mentionnée précédemment. Cependant, au lieu d’utiliser les résultats de l’analyse dès qu’ils sont disponibles, le producteur place simplement les tâches dans une file d’attente afin d’effectuer leur suivi.

```csharp
// Queue that will contain the API call tasks.
var taskQueue = new BlockingCollection<Task<ResultWrapper>>();

// Producer thread.
while (true)
{
    // Grab a frame.
    Frame f = GrabFrame();

    // Decide whether to analyze the frame.
    if (ShouldAnalyze(f))
    {
        // Start a task that will run in parallel with this thread.
        var analysisTask = Task.Run(async () =>
        {
            // Put the frame, and the result/exception into a wrapper object.
            var output = new ResultWrapper(f);
            try
            {
                output.Analysis = await Analyze(f);
            }
            catch (Exception e)
            {
                output.Exception = e;
            }
            return output;
        }

        // Push the task onto the queue.
        taskQueue.Add(analysisTask);
    }
}
```

Vous créez aussi un thread consommateur qui prend des tâches de la file d’attente, attend qu’elles se terminent et affiche le résultat ou lève l’exception qui a été déclenchée. Grâce à la file d’attente, vous pouvez garantir que les résultats sont utilisés un à la fois, dans le bon ordre, sans limiter la fréquence maximale d’images du système.

```csharp
// Consumer thread.
while (true)
{
    // Get the oldest task.
    Task<ResultWrapper> analysisTask = taskQueue.Take();
 
    // Wait until the task is completed.
    var output = await analysisTask;

    // Consume the exception or result.
    if (output.Exception != null)
    {
        throw output.Exception;
    }
    else
    {
        ConsumeResult(output.Analysis);
    }
}
```

## <a name="implement-the-solution"></a>Implémenter la solution

### <a name="get-started-quickly"></a>Prise en main rapide

Pour permettre à votre application d’être opérationnelle aussi vite que possible, nous avons implémenté le système décrit dans la section précédente. Il est conçu pour être suffisamment flexible afin de s’adapter à de nombreux scénarios, tout en étant facile à utiliser. Pour voir le code, accédez à la page [Exemple d’analyse d’images vidéo](https://github.com/Microsoft/Cognitive-Samples-VideoFrameAnalysis/) sur GitHub.

La bibliothèque contient la classe `FrameGrabber`, qui implémente le système producteur-consommateur précédemment décrit pour traiter les images vidéo provenant d’une webcam. Les utilisateurs peuvent spécifier la forme exacte de l’appel d’API. La classe utilise des événements pour informer le code appelant de l’acquisition d’une nouvelle image ou de la disponibilité d’un nouveau résultat d’analyse.

Pour illustrer certaines des possibilités, nous avons fourni deux exemples d’applications qui utilisent la bibliothèque. 

Le premier exemple d’application est une application console simple qui récupère des images de la webcam par défaut, puis les envoie au service Visage pour la détection des visages. Une version simplifiée de l’application est reproduite dans le code suivant :

```csharp
using System;
using System.Linq;
using Microsoft.Azure.CognitiveServices.Vision.Face;
using Microsoft.Azure.CognitiveServices.Vision.Face.Models;
using VideoFrameAnalyzer;

namespace BasicConsoleSample
{
    internal class Program
    {
        const string ApiKey = "<your API key>";
        const string Endpoint = "https://<your API region>.api.cognitive.microsoft.com";

        private static async Task Main(string[] args)
        {
            // Create grabber.
            FrameGrabber<DetectedFace[]> grabber = new FrameGrabber<DetectedFace[]>();

            // Create Face Client.
            FaceClient faceClient = new FaceClient(new ApiKeyServiceClientCredentials(ApiKey))
            {
                Endpoint = Endpoint
            };

            // Set up a listener for when we acquire a new frame.
            grabber.NewFrameProvided += (s, e) =>
            {
                Console.WriteLine($"New frame acquired at {e.Frame.Metadata.Timestamp}");
            };

            // Set up a Face API call.
            grabber.AnalysisFunction = async frame =>
            {
                Console.WriteLine($"Submitting frame acquired at {frame.Metadata.Timestamp}");
                // Encode image and submit to Face service.
                return (await faceClient.Face.DetectWithStreamAsync(frame.Image.ToMemoryStream(".jpg"))).ToArray();
            };

            // Set up a listener for when we receive a new result from an API call.
            grabber.NewResultAvailable += (s, e) =>
            {
                if (e.TimedOut)
                    Console.WriteLine("API call timed out.");
                else if (e.Exception != null)
                    Console.WriteLine("API call threw an exception.");
                else
                    Console.WriteLine($"New result received for frame acquired at {e.Frame.Metadata.Timestamp}. {e.Analysis.Length} faces detected");
            };

            // Tell grabber when to call the API.
            // See also TriggerAnalysisOnPredicate
            grabber.TriggerAnalysisOnInterval(TimeSpan.FromMilliseconds(3000));

            // Start running in the background.
            await grabber.StartProcessingCameraAsync();

            // Wait for key press to stop.
            Console.WriteLine("Press any key to stop...");
            Console.ReadKey();

            // Stop, blocking until done.
            await grabber.StopProcessingAsync();
        }
    }
}
```

Le deuxième exemple d’application est un peu plus intéressant. Il vous permet de choisir l’API à appeler sur les images vidéo. Sur le côté gauche, l’application affiche un aperçu de la vidéo en direct. Sur la droite, elle superpose le résultat le plus récent de l’API sur l’image correspondante.

Dans la plupart des modes, il y a un délai visible entre la vidéo en direct à gauche et l’analyse visualisée à droite. Ce délai correspond au temps nécessaire pour effectuer l’appel d’API. Une exception à cela est le mode « EmotionsWithClientFaceDetect », qui effectue la détection de visage localement sur l’ordinateur client en utilisant OpenCV avant d’envoyer les images à Cognitive Services. 

Grâce à cette approche, vous pouvez visualiser immédiatement le visage détecté. Vous pouvez alors mettre à jour les émotions plus tard, après le retour de l’appel d’API. Ceci montre la possibilité d’une approche « hybride ». Ainsi, un traitement simple peut être effectué sur le client, puis les API Cognitive Services peuvent être utilisées pour compléter ce traitement avec une analyse plus approfondie si nécessaire.

![L’application LiveCameraSample affichant une image avec des étiquettes](../../Video/Images/FramebyFrame.jpg)

### <a name="integrate-the-samples-into-your-codebase"></a>Intégrer les exemples dans votre code base

Pour commencer avec cet exemple, effectuez ceci :

1. Créez un [compte Azure](https://azure.microsoft.com/free/cognitive-services/). Si vous en avez déjà un, vous pouvez passer directement à l’étape suivante.
2. Créez des ressources Vision par ordinateur et Visage dans le Portail Azure afin d’obtenir votre clé et votre point de terminaison. Veillez à sélectionner le niveau gratuit (F0) durant l’installation.
   - [Vision par ordinateur](https://portal.azure.com/#create/Microsoft.CognitiveServicesComputerVision)
   - [Visage](https://portal.azure.com/#create/Microsoft.CognitiveServicesFace) Une fois les ressources déployées, cliquez sur **Accéder à la ressource** afin d’obtenir votre clé et votre point de terminaison pour chaque ressource. 
3. Clonez le dépôt GitHub [Cognitive-Samples-VideoFrameAnalysis](https://github.com/Microsoft/Cognitive-Samples-VideoFrameAnalysis/).
4. Ouvrez l’exemple dans Visual Studio 2015 ou ultérieur, puis générez et exécutez les exemples d’applications :
    - Pour BasicConsoleSample, la clé Visage est codée en dur directement dans [BasicConsoleSample/Program.cs](https://github.com/Microsoft/Cognitive-Samples-VideoFrameAnalysis/blob/master/Windows/BasicConsoleSample/Program.cs).
    - Pour LiveCameraSample, entrez les clés dans le volet **Paramètres** de l’application. Les clés sont conservées d’une session à l’autre en tant que données utilisateur.

Quand vous êtes prêt à intégrer les exemples, référencez la bibliothèque VideoFrameAnalyzer depuis vos propres projets.

Les fonctionnalités de compréhension d’image, de voix, de vidéo et de texte de VideoFrameAnalyzer utilisent Azure Cognitive Services. Microsoft reçoit les images, l’audio , la vidéo et d’autres données que vous chargez (via cette application) et peut les utiliser à des fins d’amélioration du service. Nous sollicitons votre aide afin de protéger les personnes dont votre application envoie les données à Azure Cognitive Services.

## <a name="summary"></a>Résumé

Dans cet article, vous avez découvert comment exécuter une analyse en quasi temps réel sur des streams vidéo en direct à l’aide des services Visage et Vision par ordinateur. Vous avez également découvert comment vous pouvez utiliser notre exemple de code pour commencer.

N’hésitez pas à fournir un feedback et des suggestions dans le [dépôt GitHub](https://github.com/Microsoft/Cognitive-Samples-VideoFrameAnalysis/). Pour fournir un feedback plus général sur l’API, accédez à notre [site UserVoice](https://cognitive.uservoice.com/).

