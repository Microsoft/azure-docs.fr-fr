---
title: Utiliser l’attribut HeadPose
titleSuffix: Azure Cognitive Services
description: Découvrez comment utiliser l’attribut HeadPose pour faire pivoter automatiquement le rectangle du visage ou détecter les mouvements de tête dans un flux vidéo.
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: sample
ms.date: 05/29/2019
ms.author: pafarley
ms.custom: devx-track-csharp
ms.openlocfilehash: fe9cc44542e97880b076d871dde82f5a02a58957
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88928970"
---
# <a name="use-the-headpose-attribute"></a>Utiliser l’attribut HeadPose

Dans ce guide, vous allez voir comment utiliser l’attribut HeadPose d’un visage détecté pour certains scénarios clés.

## <a name="rotate-the-face-rectangle"></a>Faire pivoter le rectangle de visage

Le rectangle de visage, retourné avec chaque visage détecté, marque l’emplacement et la taille du visage dans l’image. Par défaut, ce rectangle est toujours aligné avec l’image (ses côtés sont verticaux et horizontaux). Cela peut s’avérer inefficace pour encadrer des visages qui n’apparaissent pas en entier. Dans les situations où vous souhaitez découper programmatiquement des visages dans une image, il est préférable d’être en mesure de faire pivoter le rectangle à rogner.

L’exemple d’application [WPF Visage de Cognitive Services](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/app-samples/Cognitive-Services-Face-WPF) utilise l’attribut HeadPose pour faire pivoter les rectangles des visages détectés.

### <a name="explore-the-sample-code"></a>Explorer l’exemple de code

Vous pouvez faire pivoter programmatiquement le rectangle de visage à l’aide de l’attribut HeadPose. Si vous spécifiez cet attribut lors de la détection des visages (voir [Guide pratique pour détecter des visages](HowtoDetectFacesinImage.md)), vous serez en mesure de l’interroger plus tard. La méthode suivante à partir de l’application [WPF Visage de Cognitive Services](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/app-samples/Cognitive-Services-Face-WPF) accepte une liste d’objets **DetectedFace** et retourne une liste d’objets **[Face](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/blob/master/app-samples/Cognitive-Services-Face-WPF/Sample-WPF/Controls/Face.cs)** . Ici, **Face** est une classe personnalisée qui stocke les données de visage, notamment les coordonnées actualisées du rectangle. De nouvelles valeurs sont calculées pour **top**, **left**, **width** et **height**, et un nouveau champ **FaceAngle** spécifie la rotation.

```csharp
/// <summary>
/// Calculate the rendering face rectangle
/// </summary>
/// <param name="faces">Detected face from service</param>
/// <param name="maxSize">Image rendering size</param>
/// <param name="imageInfo">Image width and height</param>
/// <returns>Face structure for rendering</returns>
public static IEnumerable<Face> CalculateFaceRectangleForRendering(IList<DetectedFace> faces, int maxSize, Tuple<int, int> imageInfo)
{
    var imageWidth = imageInfo.Item1;
    var imageHeight = imageInfo.Item2;
    var ratio = (float)imageWidth / imageHeight;
    int uiWidth = 0;
    int uiHeight = 0;
    if (ratio > 1.0)
    {
        uiWidth = maxSize;
        uiHeight = (int)(maxSize / ratio);
    }
    else
    {
        uiHeight = maxSize;
        uiWidth = (int)(ratio * uiHeight);
    }

    var uiXOffset = (maxSize - uiWidth) / 2;
    var uiYOffset = (maxSize - uiHeight) / 2;
    var scale = (float)uiWidth / imageWidth;

    foreach (var face in faces)
    {
        var left = (int)(face.FaceRectangle.Left * scale + uiXOffset);
        var top = (int)(face.FaceRectangle.Top * scale + uiYOffset);

        // Angle of face rectangles, default value is 0 (not rotated).
        double faceAngle = 0;

        // If head pose attributes have been obtained, re-calculate the left & top (X & Y) positions.
        if (face.FaceAttributes?.HeadPose != null)
        {
            // Head pose's roll value acts directly as the face angle.
            faceAngle = face.FaceAttributes.HeadPose.Roll;
            var angleToPi = Math.Abs((faceAngle / 180) * Math.PI);

            // _____       | / \ |
            // |____|  =>  |/   /|
            //             | \ / |
            // Re-calculate the face rectangle's left & top (X & Y) positions.
            var newLeft = face.FaceRectangle.Left +
                face.FaceRectangle.Width / 2 -
                (face.FaceRectangle.Width * Math.Sin(angleToPi) + face.FaceRectangle.Height * Math.Cos(angleToPi)) / 2;

            var newTop = face.FaceRectangle.Top +
                face.FaceRectangle.Height / 2 -
                (face.FaceRectangle.Height * Math.Sin(angleToPi) + face.FaceRectangle.Width * Math.Cos(angleToPi)) / 2;

            left = (int)(newLeft * scale + uiXOffset);
            top = (int)(newTop * scale + uiYOffset);
        }

        yield return new Face()
        {
            FaceId = face.FaceId?.ToString(),
            Left = left,
            Top = top,
            OriginalLeft = (int)(face.FaceRectangle.Left * scale + uiXOffset),
            OriginalTop = (int)(face.FaceRectangle.Top * scale + uiYOffset),
            Height = (int)(face.FaceRectangle.Height * scale),
            Width = (int)(face.FaceRectangle.Width * scale),
            FaceAngle = faceAngle,
        };
    }
}
```

### <a name="display-the-updated-rectangle"></a>Afficher le rectangle actualisé

À ce stade, vous pouvez utiliser les objets **Face** retournés dans votre affichage. Les lignes suivantes de [FaceDetectionPage.xaml](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/blob/master/app-samples/Cognitive-Services-Face-WPF/Sample-WPF/Controls/FaceDetectionPage.xaml) montrent comment le nouveau rectangle est rendu à partir de ces données :

```xaml
 <DataTemplate>
    <Rectangle Width="{Binding Width}" Height="{Binding Height}" Stroke="#FF26B8F4" StrokeThickness="1">
        <Rectangle.LayoutTransform>
            <RotateTransform Angle="{Binding FaceAngle}"/>
        </Rectangle.LayoutTransform>
    </Rectangle>
</DataTemplate>
```

## <a name="detect-head-gestures"></a>Détecter les mouvements de tête

Vous pouvez détecter des mouvements de tête tels qu’un signe de tête ou un hochement de tête en effectuant le suivi en temps réel des modifications de HeadPose. Vous pouvez utiliser cette fonctionnalité comme détecteur d’activité personnalisé.

La détection d’activité est la tâche consistant à déterminer qu’un sujet est une personne réelle et non pas une représentation graphique ou vidéo. Un détecteur de mouvements de tête peut contribuer à vérifier l’activité du sujet, en particulier, par opposition à une représentation graphique d’une personne.

> [!CAUTION]
> Pour détecter les mouvements de tête en temps réel, vous devez appeler l’API Visage à une fréquence élevée (plus d’une fois par seconde). Cela n’est pas possible avec un abonnement de niveau gratuit (f0). Si vous disposez d’un abonnement payant, veillez à calculer les coûts d’appels d’API fréquents pour la détection des mouvements de tête.

Pour obtenir un exemple opérationnel de détection des mouvements de tête, consultez l’[exemple HeadPose du service Visage](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/app-samples/FaceAPIHeadPoseSample) sur GitHub.

## <a name="next-steps"></a>Étapes suivantes

Consultez l’application [WPF Visage de Cognitive Services](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/app-samples/Cognitive-Services-Face-WPF) sur GitHub pour obtenir un exemple opérationnel de rectangles de visages tournés. Vous pouvez aussi consulter l’application [Exemple HeadPose du service Visage](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/app-samples), qui permet de suivre en temps réel l’attribut HeadPose pour détecter les mouvements de tête.