---
title: Utiliser le point de terminaison de prédiction pour tester les images par programmation avec un classifieur - Custom Vision
titleSuffix: Azure Cognitive Services
description: Découvrez comment utiliser l’API pour tester par programmation des images avec votre classifieur Service Vision personnalisée.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: custom-vision
ms.topic: conceptual
ms.date: 04/02/2019
ms.author: pafarley
ms.custom: devx-track-csharp
ms.openlocfilehash: 7f1939536e033d2cf964dd2f4ee562e4ee20061b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88934750"
---
# <a name="use-your-model-with-the-prediction-api"></a>Utiliser votre modèle avec l’API de prédiction

Après avoir entraîné votre modèle, vous pouvez tester par programmation les images en les soumettant au point de terminaison de l’API Prediction.

> [!NOTE]
> Ce document démontre l’utilisation de C# pour soumettre une image à l’API Prediction. Pour en savoir plus et obtenir des exemples, consultez la [référence sur l’API Prediction](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Prediction_3.0/operations/5c82db60bf6a2b11a8247c15).

## <a name="publish-your-trained-iteration"></a>Publier votre itération formée

À partir de la [page Web Custom Vision](https://customvision.ai), sélectionnez votre projet, puis sélectionnez l’onglet __Performance__.

Pour envoyer des images à l’API de prédiction, vous devez d’abord publier votre itération pour la prédiction, ce que vous pouvez faire en sélectionnant __Publier__ et en spécifiant un nom pour l’itération publiée. Cela rend votre modèle accessible à l’API de prédiction de votre ressource Azure Custom Vision.

![L’onglet des performances s’affiche avec un rectangle rouge entourant le bouton Publier.](./media/use-prediction-api/unpublished-iteration.png)

Une fois votre modèle publié avec succès, vous verrez une étiquette « Publié » s’afficher à côté de votre itération dans la barre latérale gauche, et son nom s’affiche dans la description de l’itération.

![L’onglet Performances s’affiche avec un rectangle rouge autour de l’étiquette Publié et le nom de l’itération publiée.](./media/use-prediction-api/published-iteration.png)

## <a name="get-the-url-and-prediction-key"></a>Obtenir l’URL et la clé de prédiction

Une fois que votre modèle a été publié, vous pouvez récupérer les informations requises en sélectionnant __URL de prédiction__. Cela ouvrira une boîte de dialogue avec les informations sur l’utilisation de l’API de prédiction, y compris __l’URL de prédiction__ et la __Prediction-Key__.

![L’onglet des performances s’affiche avec un rectangle rouge entourant le bouton URL prédite.](./media/use-prediction-api/published-iteration-prediction-url.png)

![L’onglet Performances s’affiche avec un rectangle rouge autour de la valeur de l’URL de prédiction pour l’utilisation d’un fichier image et la valeur de Prediction-Key.](./media/use-prediction-api/prediction-api-info.png)


Dans ce guide, vous utiliserez une image locale, aussi copiez l’URL sous **Si vous avez un fichier image** à un emplacement temporaire. Copiez aussi la valeur __Prediction-Key__ correspondante.

## <a name="create-the-application"></a>Création de l'application

1. Dans Visual Studio, créez une application console C#.

1. Utilisez le code suivant comme corps du fichier __Program.cs__.

    ```csharp
    using System;
    using System.IO;
    using System.Net.Http;
    using System.Net.Http.Headers;
    using System.Threading.Tasks;

    namespace CVSPredictionSample
    {
        public static class Program
        {
            public static void Main()
            {
                Console.Write("Enter image file path: ");
                string imageFilePath = Console.ReadLine();

                MakePredictionRequest(imageFilePath).Wait();

                Console.WriteLine("\n\nHit ENTER to exit...");
                Console.ReadLine();
            }

            public static async Task MakePredictionRequest(string imageFilePath)
            {
                var client = new HttpClient();

                // Request headers - replace this example key with your valid Prediction-Key.
                client.DefaultRequestHeaders.Add("Prediction-Key", "<Your prediction key>");

                // Prediction URL - replace this example URL with your valid Prediction URL.
                string url = "<Your prediction URL>";

                HttpResponseMessage response;

                // Request body. Try this sample with a locally stored image.
                byte[] byteData = GetImageAsByteArray(imageFilePath);

                using (var content = new ByteArrayContent(byteData))
                {
                    content.Headers.ContentType = new MediaTypeHeaderValue("application/octet-stream");
                    response = await client.PostAsync(url, content);
                    Console.WriteLine(await response.Content.ReadAsStringAsync());
                }
            }

            private static byte[] GetImageAsByteArray(string imageFilePath)
            {
                FileStream fileStream = new FileStream(imageFilePath, FileMode.Open, FileAccess.Read);
                BinaryReader binaryReader = new BinaryReader(fileStream);
                return binaryReader.ReadBytes((int)fileStream.Length);
            }
        }
    }
    ```

1. Modifiez les informations suivantes :
   * Définissez le champ `namespace` sur le nom de votre projet.
   * Remplacez l’espace réservé `<Your prediction key>` avec la valeur de clé que vous avez récupérée précédemment.
   * Remplacez l’espace réservé `<Your prediction URL>` avec l’URL que vous avez récupérée précédemment.

## <a name="run-the-application"></a>Exécution de l'application

Lorsque vous exécutez l’application, vous êtes invité à saisir un chemin d’accès vers un fichier image dans la console. L’image est ensuite envoyée à l’API de prédiction, et les résultats de prédiction sont renvoyés sous forme de chaîne au format JSON. Voici un exemple de réponse.

```json
{
    "id":"7796df8e-acbc-45fc-90b4-1b0c81b73639",
    "project":"8622c779-471c-4b6e-842c-67a11deffd7b",
    "iteration":"59ec199d-f3fb-443a-b708-4bca79e1b7f7",
    "created":"2019-03-20T16:47:31.322Z",
    "predictions":[
        {"tagId":"d9cb3fa5-1ff3-4e98-8d47-2ef42d7fb373","tagName":"cat", "probability":1.0},
        {"tagId":"9a8d63fb-b6ed-4462-bcff-77ff72084d99","tagName":"dog", "probability":0.1087869}
    ]
}
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide, vous avez appris à envoyer des images sur votre détecteur/classifieur d’images personnalisé et recevoir une réponse par programmation avec le SDK C#. Nous allons maintenant apprendre à exécuter des scénarios de bout en bout avec C#, ou commencer à utiliser une autre langue de SDK.

* [Démarrage rapide : SDK Custom Vision](quickstarts/image-classification.md)
