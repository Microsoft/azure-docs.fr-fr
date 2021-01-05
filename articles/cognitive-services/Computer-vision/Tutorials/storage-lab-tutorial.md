---
title: 'Tutoriel : Générer des métadonnées pour les images Azure'
titleSuffix: Azure Cognitive Services
description: Ce tutoriel vous montre comment intégrer le service Azure Vision par ordinateur dans une application web afin de générer des métadonnées pour des images.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: tutorial
ms.date: 11/23/2020
ms.author: pafarley
ms.custom: devx-track-csharp
ms.openlocfilehash: b51234ff121071de27ec8c91425f4a769c8d833e
ms.sourcegitcommit: 1bf144dc5d7c496c4abeb95fc2f473cfa0bbed43
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "96008784"
---
# <a name="tutorial-use-computer-vision-to-generate-image-metadata-in-azure-storage"></a>Tutoriel : Utiliser le service Vision par ordinateur pour générer des métadonnées des images dans le stockage Azure

Ce tutoriel vous montre comment intégrer le service Vision par ordinateur d’Azure dans une application web afin de générer ensuite des métadonnées pour les images chargées. Cela est utile pour les scénarios de [gestion des actifs numériques](../overview.md#computer-vision-for-digital-asset-management), par exemple si une entreprise souhaite générer rapidement des sous-titres descriptifs ou des mots clés de recherche pour toutes ses images.

Vous trouverez un guide complet de l’application dans le [lab Azure Storage and Cognitive Services](https://github.com/Microsoft/computerscience/blob/master/Labs/Azure%20Services/Azure%20Storage/Azure%20Storage%20and%20Cognitive%20Services%20(MVC).md) sur GitHub. Ce tutoriel couvre essentiellement l’exercice 5 du lab. Vous pouvez créer l’application complète en suivant chaque étape, mais si vous voulez seulement découvrir comment intégrer Vision par ordinateur à une application web existante, lisez le présent tutoriel.

Ce didacticiel vous explique les procédures suivantes :

> [!div class="checklist"]
> * Créer une ressource Vision par ordinateur dans Azure
> * Effectuer une analyse des images du stockage Azure
> * Attacher des métadonnées aux images du stockage Azure
> * Vérifier les métadonnées des images à l’aide de l’Explorateur Stockage Azure

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/cognitive-services) avant de commencer. 

## <a name="prerequisites"></a>Prérequis

- [Visual Studio 2017 Community](https://www.visualstudio.com/products/visual-studio-community-vs.aspx) ou ultérieur, avec les charges de travail « ASP.NET et développement web » et « Développement Azure » installées.
- Un compte de stockage Azure avec un conteneur d’objets blob configuré pour le stockage d’images (suivez l’[exercice 1 du lab Stockage Azure](https://github.com/Microsoft/computerscience/blob/master/Labs/Azure%20Services/Azure%20Storage/Azure%20Storage%20and%20Cognitive%20Services%20(MVC).md#Exercise1) si vous avez besoin d’aide pour cette étape).
- L’outil Explorateur Stockage Azure (suivez l’[exercice 2 du lab Azure Storage](https://github.com/Microsoft/computerscience/blob/master/Labs/Azure%20Services/Azure%20Storage/Azure%20Storage%20and%20Cognitive%20Services%20(MVC).md#Exercise2) si vous avez besoin d’aide pour cette étape).
- Une application web ASP.NET avec accès au stockage Azure (suivez l’[exercice 3 du lab Azure Storage](https://github.com/Microsoft/computerscience/blob/master/Labs/Azure%20Services/Azure%20Storage/Azure%20Storage%20and%20Cognitive%20Services%20(MVC).md#Exercise3) pour créer une application de ce type rapidement).

## <a name="create-a-computer-vision-resource"></a>Créer une ressource Vision par ordinateur

Vous devez créer une ressource Vision par ordinateur pour votre compte Azure ; cette ressource gère votre accès au service Vision par ordinateur d’Azure. 

1. Pour créer une ressource Vision par ordinateur, suivez les instructions de l'article [Créer une ressource Azure Cognitive Services](../../cognitive-services-apis-create-account.md).

1. Accédez ensuite au menu de votre groupe de ressources et cliquez sur l’abonnement à l’API Vision par ordinateur que vous venez de créer. Copiez l’URL indiquée sous **Point de terminaison** à un endroit où vous pourrez facilement la récupérer un peu plus tard. Cliquez ensuite sur **Afficher les clés d’accès**.

    ![Page du portail Azure avec l’URL du point de terminaison et le lien des clés d’accès entourés](../Images/copy-vision-endpoint.png)
    
    [!INCLUDE [Custom subdomains notice](../../../../includes/cognitive-services-custom-subdomains-note.md)]


1. Dans la fenêtre suivante, copiez la valeur de **KEY 1** dans le Presse-papiers.

    ![Boîte de dialogue Gérer les clés, avec le bouton de copie entouré](../Images/copy-vision-key.png)

## <a name="add-computer-vision-credentials"></a>Ajouter les informations d’identification pour Vision par ordinateur

Vous devez ensuite ajouter les informations d’identification nécessaire à votre application pour pouvoir accéder aux ressources Vision par ordinateur.

Ouvrez votre application web ASP.NET dans Visual Studio et accédez au fichier **Web.config** à la racine du projet. Ajoutez les instructions suivantes dans la section `<appSettings>` du fichier, en remplaçant `VISION_KEY` par la clé que vous aviez copiée à l’étape précédente et `VISION_ENDPOINT` par l’URL que vous avez enregistrée à l’étape précédente.

```xml
<add key="SubscriptionKey" value="VISION_KEY" />
<add key="VisionEndpoint" value="VISION_ENDPOINT" />
```

Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet, puis utilisez la commande **Gérer les packages NuGet** pour installer le package **Microsoft.Azure.CognitiveServices.Vision.ComputerVision**. Ce package contient les types à utiliser pour appeler l’API Vision par ordinateur.

## <a name="add-metadata-generation-code"></a>Ajouter le code de génération des métadonnées

Ensuite, vous allez ajouter le code qui utilise le service Vision par ordinateur pour créer des métadonnées pour les images. Ces étapes sont conçues pour l’application ASP.NET du lab, mais vous pouvez les adapter à votre propre application. À ce stade, l’important est de vous assurer que votre application web ASP.NET peut charger des images dans un conteneur de stockage Azure, lire les images de ce conteneur et afficher les images dans la vue. Si vous avez besoin d’aide sur cette étape, le mieux est de suivre l’[exercice 3 du lab Stockage Azure](https://github.com/Microsoft/computerscience/blob/master/Labs/Azure%20Services/Azure%20Storage/Azure%20Storage%20and%20Cognitive%20Services%20(MVC).md#Exercise3). 

1. Ouvrez le fichier *HomeController.cs* dans le dossier **Contrôleurs** du projet et ajoutez les instructions `using` suivantes en haut du fichier :

    ```csharp
    using Microsoft.Azure.CognitiveServices.Vision.ComputerVision;
    using Microsoft.Azure.CognitiveServices.Vision.ComputerVision.Models;
    ```

1. Accédez ensuite à la méthode **Upload**, qui convertit et charge les images dans un stockage blob. Ajoutez le code suivant immédiatement après le bloc qui commence par `// Generate a thumbnail` (ou à la fin du processus de création de l’objet blob de l’image). Ce code utilise l’objet blob contenant l’image (`photo`) et génère une description de cette image à l’aide de Vision par ordinateur. L’API Vision par ordinateur génère également une liste de mots clés en lien avec l’image. Les mots clés et la description générés sont stockés dans les métadonnées de l’objet blob pour les utiliser plus tard.

    ```csharp
    // Submit the image to Azure's Computer Vision API
    ComputerVisionClient vision = new ComputerVisionClient(
        new ApiKeyServiceClientCredentials(ConfigurationManager.AppSettings["SubscriptionKey"]),
        new System.Net.Http.DelegatingHandler[] { });
    vision.Endpoint = ConfigurationManager.AppSettings["VisionEndpoint"];

    VisualFeatureTypes[] features = new VisualFeatureTypes[] { VisualFeatureTypes.Description };
    var result = await vision.AnalyzeImageAsync(photo.Uri.ToString(), features);

    // Record the image description and tags in blob metadata
    photo.Metadata.Add("Caption", result.Description.Captions[0].Text);

    for (int i = 0; i < result.Description.Tags.Count; i++)
    {
        string key = String.Format("Tag{0}", i);
        photo.Metadata.Add(key, result.Description.Tags[i]);
    }

    await photo.SetMetadataAsync();
    ```

1. Ensuite, accédez à la méthode **Index** dans le même fichier. Cette méthode énumère les objets blob d’image stockés dans le conteneur d’objets blob ciblé (comme les instances de **IListBlobItem**) et les passe à la vue de l’application. Remplacez le bloc `foreach` dans cette méthode par le code suivant. Ce code appelle **CloudBlockBlob.FetchAttributes** pour obtenir les métadonnées attachées à chaque objet blob. Il extrait la description générée par ordinateur (`caption`) à partir des métadonnées et l’ajoute à l’objet **BlobInfo**, qui est alors passé à la vue.
    
    ```csharp
    foreach (IListBlobItem item in container.ListBlobs())
    {
        var blob = item as CloudBlockBlob;
    
        if (blob != null)
        {
            blob.FetchAttributes(); // Get blob metadata
            var caption = blob.Metadata.ContainsKey("Caption") ? blob.Metadata["Caption"] : blob.Name;
    
            blobs.Add(new BlobInfo()
            {
                ImageUri = blob.Uri.ToString(),
                ThumbnailUri = blob.Uri.ToString().Replace("/photos/", "/thumbnails/"),
                Caption = caption
            });
        }
    }
    ```

## <a name="test-the-app"></a>Test de l'application

Enregistrez vos modifications dans Visual Studio, puis appuyez sur **Ctrl+F5** pour lancer l’application dans votre navigateur. Utilisez l’application pour charger quelques images à partir du dossier « photos » des ressources du lab ou à partir de votre propre dossier. Quand vous placez le curseur sur l’une des images dans la vue, une fenêtre d’info-bulle doit apparaître et afficher la légende générée par ordinateur pour l’image.

![Légende générée par ordinateur](../Images/thumbnail-with-tooltip.png)

Pour voir toutes les métadonnées attachées, affichez le conteneur de stockage utilisé pour les images dans l’Explorateur Stockage Azure. Cliquez avec le bouton droit sur un objet blob dans le conteneur, puis sélectionnez **Propriétés**. Dans la boîte de dialogue, vous voyez une liste de paires clé-valeur. La description de l’image générée par ordinateur est stockée dans l’élément « Caption » et les différents mots clés de recherche sont stockés dans « Tag0 », « Tag1 », etc. Quand vous avez terminé, cliquez sur **Annuler** pour fermer la boîte de dialogue.

![Fenêtre de dialogue des propriétés de l’image, avec les balises de métadonnées listées](../Images/blob-metadata.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous souhaitez continuer à travailler sur votre application web, passez à la section [Étapes suivantes](#next-steps). Si vous n’avez plus besoin de cette application, supprimez toutes les ressources propres à l’application. Pour supprimer les ressources, vous pouvez supprimer le groupe de ressources qui contient votre abonnement Stockage Azure et la ressource Vision par ordinateur. Cette opération supprime le compte de stockage, les objets blob qui y ont été chargés ainsi que la ressource App Service utilisée pour vous connecter à l’application web ASP.NET. 

Pour supprimer le groupe de ressources, ouvrez l’onglet **Groupes de ressources** dans le portail, accédez au groupe de ressources que vous avez utilisé dans ce projet, puis cliquez sur **Supprimer le groupe de ressources** en haut de la vue. Vous êtes alors invité à taper le nom du groupe de ressources pour confirmer sa suppression, car une fois qu’il est supprimé, vous ne pouvez pas le récupérer.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez configuré le service Vision par ordinateur d’Azure dans une application web existante pour générer automatiquement des légendes et des mots clés pour les images d’objet blob au fil de leur chargement. Effectuez maintenant l’exercice 6 du lab Azure Storage pour savoir comment ajouter des fonctionnalités de recherche à votre application web. Cet exercice utilise les mots clés de recherche générés par le service Vision par ordinateur.

> [!div class="nextstepaction"]
> [Ajouter une recherche à votre application](https://github.com/Microsoft/computerscience/blob/master/Labs/Azure%20Services/Azure%20Storage/Azure%20Storage%20and%20Cognitive%20Services%20(MVC).md#Exercise6)
