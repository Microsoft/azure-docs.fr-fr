---
title: Guide pratique pour spécifier un modèle de reconnaissance – Visage
titleSuffix: Azure Cognitive Services
description: Cet article vous montre comment choisir le modèle de reconnaissance à utiliser avec votre application Azure Visage.
services: cognitive-services
author: longli0
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: conceptual
ms.date: 12/03/2019
ms.author: longl
ms.custom: devx-track-csharp
ms.openlocfilehash: 016b8bf010f597e963e0901d1ec48486f79bbb35
ms.sourcegitcommit: d76108b476259fe3f5f20a91ed2c237c1577df14
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92913124"
---
# <a name="specify-a-face-recognition-model"></a>Spécifier un modèle de reconnaissance faciale

Ce guide vous montre comment spécifier un modèle de reconnaissance faciale pour la détection des visages, l’identification et la recherche de similarités à l’aide du service Azure Visage.

Le service Visage utilise des modèles Machine Learning pour effectuer des opérations sur les visages humains présents dans les images. Nous continuons d’améliorer la précision de nos modèles en fonction des commentaires de nos clients et des progrès de la recherche, et nous intégrons ces améliorations sous forme de mises à jour de modèles. Les développeurs peuvent spécifier la version du modèle de reconnaissance faciale qu'ils souhaitent utiliser ; ils peuvent choisir le modèle qui correspond le mieux à leur cas d'utilisation.

Le service Visage d’Azure dispose de trois modèles de reconnaissance. Les modèles _recognition_01_ (publié en 2017) et _recognition_02_ (publié en 2019) sont continuellement pris en charge pour assurer la compatibilité descendante pour les clients utilisant les objets FaceList ou **PersonGroup** créés avec ces modèles. Un objet **FaceList** ou **Persongroup** utilise toujours le modèle de reconnaissance avec lequel il a été créé et de nouveaux visages sont associés à ce modèle lorsqu'ils y sont ajoutés. Ceci ne peut pas être modifié après la création et les clients doivent utiliser le modèle de reconnaissance correspondant avec l’objet **FaceList** ou **PersonGroup** correspondant.

Vous pouvez passer à des modèles de reconnaissance ultérieurs à votre convenance. Toutefois, vous devrez créer de nouveaux objets FaceList et PersonGroup avec le modèle de reconnaissance de votre choix.

Le modèle _recognition_03_ (publié en 2020) est le modèle le plus précis actuellement disponible. Si vous êtes un nouveau client, nous vous recommandons d’utiliser ce modèle. _Recognition_03_ offrira une meilleure précision pour les comparaisons de similarité et les comparaisons de correspondance de personne. Notez que chaque modèle fonctionne indépendamment des autres et qu’un seuil de confiance défini pour un modèle n’est pas destiné à être comparé avec les autres modèles de reconnaissance.

Poursuivez la lecture pour en savoir plus sur la détermination d’un modèle spécifique dans différentes opérations de détection de visages, tout en évitant les conflits de modèles. Si vous êtes un utilisateur expérimenté et souhaitez déterminer si vous devez passer au modèle le plus récent, passez à la section [Évaluer des modèles différents](#evaluate-different-models) pour évaluer le nouveau modèle et comparer les résultats en utilisant votre ensemble de données actuel.


## <a name="prerequisites"></a>Prérequis

Vous devez maîtriser les concepts de la détection et de l'identification de visages par intelligence artificielle (AI). Si ce n’est pas les cas, consultez d’abord ces guides :

* [Concepts de détection de visage](../concepts/face-detection.md)
* [Concepts de reconnaissance faciale](../concepts/face-recognition.md)
* [Comment détecter des visages dans une image](HowtoDetectFacesinImage.md)

## <a name="detect-faces-with-specified-model"></a>Détecter des visages avec le modèle spécifié

La détection des visages identifie les repères visuels des visages humains et trouve leur emplacement dans les cadres englobants. Elle extrait également les traits du visage et les stocke pour les utiliser à des fins d’identification. Toutes ces informations forment la représentation d’un visage.

Le modèle de reconnaissance est utilisé lors de l’extraction des traits du visage, de sorte que vous pouvez spécifier une version du modèle lors de l’exécution de l'opération de détection.

Lorsque vous utilisez l’API [Face - Detect], attribuez la version du modèle avec le paramètre `recognitionModel`. Les valeurs disponibles sont :
* recognition_01
* recognition_02
* recognition_03


Si vous le souhaitez, vous pouvez spécifier le paramètre _returnRecognitionModel_ (par défaut **false** ) pour indiquer si _recognitionModel_ doit être retourné comme réponse. Ainsi, une URL de requête pour l’API REST [Face - Detect] se présentera comme suit :

`https://westus.api.cognitive.microsoft.com/face/v1.0/detect[?returnFaceId][&returnFaceLandmarks][&returnFaceAttributes][&recognitionModel][&returnRecognitionModel]&subscription-key=<Subscription key>`

Si vous utilisez la bibliothèque cliente, vous pouvez attribuer la valeur de `recognitionModel` en passant une chaîne représentant la version. Si vous ne lui attribuez aucune valeur, une version du modèle par défaut de `recognition_01` est utilisée. Consultez l'exemple de code suivant pour la bibliothèque cliente .NET.

```csharp
string imageUrl = "https://news.microsoft.com/ceo/assets/photos/06_web.jpg";
var faces = await faceClient.Face.DetectWithUrlAsync(imageUrl, true, true, recognitionModel: "recognition_01", returnRecognitionModel: true);
```

## <a name="identify-faces-with-specified-model"></a>Identifier des visages avec le modèle spécifié

Le service Visage peut extraire les données de visage d’une image et les associer à un objet **Person** (via l’appel d’API [Add face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523b), par exemple), et plusieurs objets **Person** peuvent être stockés ensemble dans un objet **PersonGroup**. Ensuite, un nouveau visage peut être comparé à un objet **PersonGroup** (avec l'appel [Visage - Identifier]), et la personne correspondante dans ce groupe peut être identifiée.

Un objet **PersonGroup** doit avoir un modèle de reconnaissance unique pour toutes les **personnes** , et vous pouvez le spécifier en utilisant le paramètre `recognitionModel` lorsque vous créez le groupe ( [PersonGroup - Créer] ou [LargePersonGroup - Créer]). Si vous ne spécifiez pas ce paramètre, le modèle original `recognition_01` est utilisé. Un groupe utilisera toujours le modèle de reconnaissance avec lequel il a été créé, et de nouveaux visages seront associés à ce modèle lorsqu'ils y seront ajoutés ; ceci ne peut pas être modifié après la création d’un groupe. Pour voir avec quel modèle un objet **PersonGroup** est configuré, utilisez l'API [PersonGroup - Get] avec le jeu de paramètres _returnRecognitionModel_ défini sur **true**.

Consultez l'exemple de code suivant pour la bibliothèque cliente .NET.

```csharp
// Create an empty PersonGroup with "recognition_02" model
string personGroupId = "mypersongroupid";
await faceClient.PersonGroup.CreateAsync(personGroupId, "My Person Group Name", recognitionModel: "recognition_02");
```

Dans ce code, un objet **PersonGroup** avec l’ID `mypersongroupid` est créé, et il est configuré pour utiliser le modèle _recognition_02_ afin d’extraire les traits du visage.

En conséquence, vous devez spécifier quel modèle utiliser lors de la détection des visages à comparer à cet objet **PersonGroup** (via l’API [Face - Detect]). Le modèle que vous utilisez doit toujours être cohérent avec la configuration de l’objet **PersonGroup** ; sinon, l'opération échouera en raison de modèles incompatibles.

Aucun changement n’est apporté à l'API [Visage - Identifier] ; il vous suffit de spécifier la version du modèle lors de la détection.

## <a name="find-similar-faces-with-specified-model"></a>Rechercher des visages avec le modèle spécifié

Vous pouvez également spécifier un modèle de reconnaissance pour la recherche de similarités. Vous pouvez attribuer la version du modèle avec `recognitionModel` lors de la création de la liste des visages avec l’API [FaceList - Create] ou [LargeFaceList - Create]. Si vous ne spécifiez pas ce paramètre, le modèle `recognition_01` est utilisé par défaut. Une liste de visages utilise toujours le modèle de reconnaissance avec lequel il a été créé et de nouveaux visages sont associés à ce modèle lorsqu'ils y sont ajoutés. Vous ne pouvez pas modifier ceci après la création. Pour voir avec quel modèle une liste de visages est configurée, utilisez l'API [FaceList - Get] avec le jeu de paramètres _returnRecognitionModel_ défini sur **true**.

Consultez l'exemple de code suivant pour la bibliothèque cliente .NET.

```csharp
await faceClient.FaceList.CreateAsync(faceListId, "My face collection", recognitionModel: "recognition_03");
```

Ce code crée une liste de visages appelée `My face collection`, en utilisant le modèle _recognition_03_ pour l’extraction des traits de visage. Lorsque vous recherchez dans cette liste des visages similaires à un nouveau visage détecté, ce visage doit avoir été détecté ( [Face - Detect]) en utilisant le modèle _recognition_03_. Comme dans la section précédente, le modèle doit être cohérent.

Aucun changement n’est apporté à l'API [Face - Find Similar] ; vous spécifiez uniquement la version du modèle lors de la détection.

## <a name="verify-faces-with-specified-model"></a>Vérifier des visages avec le modèle spécifié

L’API [Face - Verify] évalue si deux visages appartiennent à la même personne. Aucun changement n’est apporté à l'API Verify au niveau des modèles de reconnaissance, mais vous pouvez uniquement comparer des visages qui ont été détectés avec le même modèle.

## <a name="evaluate-different-models"></a>Évaluer des modèles différents

Si vous souhaitez comparer les performances des différents modèles de reconnaissance sur vos propres données, vous devez :
1. créer trois objets PersonGroup utilisant respectivement _recognition_01_ , _recognition_02_ et _recognition_03_  ;
1. utiliser vos données d’image pour détecter les visages et les relier à des **personne** s au sein de ces trois objets **PersonGroup**  ; 
1. former vos objets PersonGroup à l'aide de l'API PersonGroup - Former ;
1. effectuer un test avec l’API Face - Identify sur les trois objets **PersonGroup** et comparer les résultats.


Si vous spécifiez normalement un seuil de confiance (une valeur comprise entre zéro et une valeur qui détermine le degré de confiance que doit avoir le modèle pour identifier un visage), vous devrez peut-être utiliser différents seuils pour différents modèles. Le seuil d’un modèle n'est pas destiné à être partagé avec un autre modèle, et il ne produira pas nécessairement les mêmes résultats.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris comment spécifier le modèle de reconnaissance à utiliser avec différentes API du service Visage. Voici maintenant un guide de démarrage rapide pour commencer à utiliser la détection des visages.

* [SDK .NET Visage](../quickstarts/client-libraries.md?pivots=programming-language-csharp%253fpivots%253dprogramming-language-csharp)
* [Kit de développement logiciel (SDK) Face Python](../quickstarts/client-libraries.md?pivots=programming-language-python%253fpivots%253dprogramming-language-python)
* [Kit de développement logiciel (SDK) Go face Go](../quickstarts/client-libraries.md?pivots=programming-language-go%253fpivots%253dprogramming-language-go)

[Face - Detect]: https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d
[Face - Find Similar]: https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395237
[Visage - Identifier]: https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239
[Face - Verify]: https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523a
[PersonGroup - Créer]: https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395244
[PersonGroup - Get]: https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395246
[PersonGroup Person - Add Face]: https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523b
[PersonGroup - Train]: https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395249
[LargePersonGroup - Créer]: https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599acdee6ac60f11b48b5a9d
[FaceList - Create]: https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039524b
[FaceList - Get]: https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039524c
[LargeFaceList - Create]: https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/5a157b68d2de3616c086f2cc