---
title: Détecter les visages dans une image – Visage
titleSuffix: Azure Cognitive Services
description: Ce guide explique comment utiliser la détection des visages pour extraire des attributs tels que le sexe, l’âge ou la posture à partir d’une image donnée.
services: cognitive-services
author: SteveMSFT
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: conceptual
ms.date: 04/18/2019
ms.author: sbowles
ms.custom: devx-track-csharp
ms.openlocfilehash: f7a740b1015bda80000f65180eda2c5e618670da
ms.sourcegitcommit: d76108b476259fe3f5f20a91ed2c237c1577df14
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92911237"
---
# <a name="get-face-detection-data"></a>Obtenir des données de détection des visages

Ce guide explique comment utiliser la détection des visages pour extraire des attributs tels que le sexe, l’âge ou la posture à partir d’une image donnée. Les extraits de code de ce guide sont écrits en C# en utilisant la bibliothèque cliente Visage d’Azure Cognitive Services. La même fonctionnalité est disponible via l’[API REST](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236).

Ce guide vous montre comment :

- Obtenir les emplacements et les dimensions des visages dans une image.
- Obtenir les emplacements des différents points de repère du visage, tels que les pupilles, le nez et la bouche, dans une image.
- Deviner le sexe, l’âge, l’émotion et d’autres attributs d’un visage détecté.

## <a name="setup"></a>Programme d’installation

Ce guide suppose que vous avez déjà construit un objet [FaceClient](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.faceclient?view=azure-dotnet), nommé `faceClient`, avec une URL de point de terminaison et une clé d’abonnement Visage. À ce stade, vous pouvez utiliser la fonctionnalité de détection des visages en appelant [DetectWithUrlAsync](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.faceoperationsextensions.detectwithurlasync?view=azure-dotnet), qui est utilisé dans ce guide, ou [DetectWithStreamAsync](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.faceoperationsextensions.detectwithstreamasync?view=azure-dotnet). Pour obtenir des instructions sur la configuration de cette fonctionnalité, suivez un des guides de démarrage rapide.

Ce guide se concentre sur les spécificités de l’appel de détection, telles que les arguments que vous pouvez transmettre et ce que vous pouvez faire avec les données retournées. Nous vous recommandons d’interroger uniquement les fonctionnalités dont vous avez besoin. Chaque opération nécessite du temps supplémentaire.

## <a name="get-basic-face-data"></a>Obtenir des données de visage de base

Pour rechercher des visages et savoir où ils se trouvent sur une image, appelez la méthode [DetectWithUrlAsync](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.faceoperationsextensions.detectwithurlasync?view=azure-dotnet) ou [DetectWithStreamAsync](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.faceoperationsextensions.detectwithstreamasync?view=azure-dotnet) avec le paramètre _returnFaceId_ défini sur **true**. Il s’agit du paramètre par défaut.

:::code language="csharp" source="~/cognitive-services-quickstart-code/dotnet/Face/sdk/detect.cs" id="basic1":::

Vous pouvez interroger les objets [DetectedFace](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.models.detectedface?view=azure-dotnet) retournés pour obtenir leurs ID uniques et un rectangle qui fournit les coordonnées en pixels du visage.

:::code language="csharp" source="~/cognitive-services-quickstart-code/dotnet/Face/sdk/detect.cs" id="basic2":::

Pour plus d’informations sur la façon d’analyser l’emplacement et les dimensions du visage, consultez [FaceRectangle](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.models.facerectangle?view=azure-dotnet). En règle générale, ce rectangle contient les yeux, les sourcils, le nez et la bouche. Le sommet du crâne, les oreilles et le menton ne sont pas nécessairement inclus. Pour utiliser le rectangle du visage pour rogner une tête complète ou obtenir un portrait en plan moyen, peut-être pour une image de type photo d’identité, vous pouvez développer le rectangle dans chaque direction.

## <a name="get-face-landmarks"></a>Obtenir les points de repère du visage

Les [points de repère de visage](../concepts/face-detection.md#face-landmarks) sont un ensemble de points faciles à trouver sur un visage, tels que les pupilles ou la pointe du nez. Pour obtenir les données des points de repère de visage, définissez le paramètre _detectionModel_ sur **DetectionModel.Detection01** et le paramètre _returnFaceLandmarks_ sur **true**.

:::code language="csharp" source="~/cognitive-services-quickstart-code/dotnet/Face/sdk/detect.cs" id="landmarks1":::

Le code suivant montre comment récupérer les emplacements du nez et des pupilles :

:::code language="csharp" source="~/cognitive-services-quickstart-code/dotnet/Face/sdk/detect.cs" id="landmarks2":::

Vous pouvez également utiliser les données des points de repère de visage pour calculer avec précision l’orientation du visage. Par exemple, vous pouvez définir la rotation du visage en tant que vecteur allant du centre de la bouche jusqu’au centre des yeux. Le code suivant calcule ce vecteur :

:::code language="csharp" source="~/cognitive-services-quickstart-code/dotnet/Face/sdk/detect.cs" id="direction":::

Lorsque vous connaissez l’orientation du visage, vous pouvez faire pivoter le cadre rectangulaire du visage pour mieux l’aligner. Pour rogner les visages dans une image, vous pouvez faire pivoter l’image par programmation afin que les visages apparaissent toujours verticaux.

## <a name="get-face-attributes"></a>Obtenir les attributs du visage

Outre les points de repère et les rectangles des visages, l’API de détection des visages peut analyser plusieurs attributs conceptuels d’un visage. Pour obtenir la liste complète, consultez la section conceptuelle [Attributs du visage](../concepts/face-detection.md#attributes).

Pour analyser les attributs d’un visage, définissez le paramètre _detectionModel_ sur **DetectionModel.Detection01** et le paramètre _returnFaceAttributes_ sur une liste de valeurs [enum FaceAttributeType](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.models.faceattributetype?view=azure-dotnet).

:::code language="csharp" source="~/cognitive-services-quickstart-code/dotnet/Face/sdk/detect.cs" id="attributes1":::

Ensuite, obtenez des références aux données retournées et effectuez des opérations supplémentaires selon vos besoins.

:::code language="csharp" source="~/cognitive-services-quickstart-code/dotnet/Face/sdk/detect.cs" id="attributes2":::

Pour en savoir plus sur les différents attributs, consultez le guide conceptuel [Détection et attributs de visage](../concepts/face-detection.md).

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide, vous avez appris à utiliser les différentes fonctionnalités de détection des visages. Ensuite, intégrez ces fonctionnalités dans votre application en suivant un tutoriel approfondi.

- [Tutoriel : Créer une application WPF pour afficher les données des visages dans une image](../Tutorials/FaceAPIinCSharpTutorial.md)

## <a name="related-topics"></a>Rubriques connexes

- [Documentation de référence (REST)](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236)
- [Documentation de référence (kit SDK .NET)](/dotnet/api/overview/azure/cognitiveservices/client/faceapi?view=azure-dotnet)