---
title: Guide de démarrage rapide pour l’API REST Visage
description: Utilisez l’API REST Visage avec cURL afin de détecter des visages, rechercher des visages semblables (recherche faciale par image), identifier des visages (recherche avec reconnaissance faciale) et d’effectuer la migration de vos données faciales.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: include
ms.date: 12/06/2020
ms.author: pafarley
ms.openlocfilehash: be942f73ee0a3d5a8850141c937754bad330db90
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96912306"
---
Commencez à utiliser la reconnaissance faciale avec l’API REST Visage. Le service Visage vous donne accès à des algorithmes avancés pour la détection et la reconnaissance des visages dans des images.

Utilisez l’API REST Visage pour :

* [Détecter des visages sur une image](#detect-faces-in-an-image)
* [Rechercher des visages semblables](#find-similar-faces)

> [!NOTE]
> Ce guide de démarrage rapide utilise des commandes cURL pour appeler l’API REST. Vous pouvez également appeler l’API REST à l’aide d’un langage de programmation. Vous pouvez consulter des exemples [C#](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/dotnet/Face/rest), [Python](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/python/Face/rest), [Java](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/java/Face/rest), [JavaScript](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/javascript/Face/rest) et [Go](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/go/Face/rest) sur GitHub.

## <a name="prerequisites"></a>Prérequis

* Abonnement Azure - [En créer un gratuitement](https://azure.microsoft.com/free/cognitive-services/)
* Une fois que vous avez votre abonnement Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesFace"  title="créez une ressource Visage"  target="_blank">créer une ressource Visage <span class="docon docon-navigate-external x-hidden-focus"></span></a> dans le Portail Azure pour obtenir votre clé et votre point de terminaison. Une fois le déploiement effectué, cliquez sur **Accéder à la ressource**.
    * Vous aurez besoin de la clé et du point de terminaison de la ressource que vous créez pour connecter votre application à l’API Visage. Vous collerez votre clé et votre point de terminaison dans le code ci-dessous plus loin dans le guide de démarrage rapide.
    * Vous pouvez utiliser le niveau tarifaire Gratuit (`F0`) pour tester le service, puis passer par la suite à un niveau payant pour la production.


## <a name="detect-faces-in-an-image"></a>Détecter des visages dans une image

Vous allez utiliser une commande similaire à ce qui suit pour appeler l’API Visage et obtenir les données d’attribut de visage d’une image. Pour commencer, copiez le code dans un éditeur de texte. Avant de l’exécuter, vous devez changer certaines parties.

:::code language="shell" source="~/cognitive-services-quickstart-code/curl/face/detect.sh" ID="detection_model_2":::

Effectuez les modifications suivantes :
1. Affectez `Ocp-Apim-Subscription-Key` à votre clé d’abonnement Visage valide.
1. Modifiez la première partie de l’URL de requête pour qu’elle corresponde au point de terminaison associé à votre clé d’abonnement.
   [!INCLUDE [subdomains-note](../../../../../includes/cognitive-services-custom-subdomains-note.md)]
1. Si vous le souhaitez, vous pouvez modifier l’URL dans le corps de la requête pour qu’elle pointe vers une autre image.

Après avoir apporté vos changements, ouvrez une invite de commandes et entrez la nouvelle commande. 

### <a name="examine-the-results"></a>Examiner les résultats

Les informations de visage doivent apparaître en tant que données JSON dans la fenêtre de console. Par exemple :

```json
[
  {
    "faceId": "49d55c17-e018-4a42-ba7b-8cbbdfae7c6f",
    "faceRectangle": {
      "top": 131,
      "left": 177,
      "width": 162,
      "height": 162
    }
  }
]  
```

### <a name="get-face-attributes"></a>Obtenir les attributs du visage
 
Pour extraire les attributs de visage, appelez à nouveau l’API de détection, mais définissez `detectionModel` sur `detection_01`. Ajoutez également le paramètre de requête `returnFaceAttributes`. La commande doit maintenant ressembler à ce qui suit. Comme précédemment, insérez la clé et le point de terminaison de votre abonnement au service Visage.

:::code language="shell" source="~/cognitive-services-quickstart-code/curl/face/detect.sh" ID="detection_model_1":::

### <a name="examine-the-results"></a>Examiner les résultats

Les informations de visage retournées incluent désormais des attributs de visage. Par exemple :

```json
[
  {
    "faceId": "49d55c17-e018-4a42-ba7b-8cbbdfae7c6f",
    "faceRectangle": {
      "top": 131,
      "left": 177,
      "width": 162,
      "height": 162
    },
    "faceAttributes": {
      "smile": 0,
      "headPose": {
        "pitch": 0,
        "roll": 0.1,
        "yaw": -32.9
      },
      "gender": "female",
      "age": 22.9,
      "facialHair": {
        "moustache": 0,
        "beard": 0,
        "sideburns": 0
      },
      "glasses": "NoGlasses",
      "emotion": {
        "anger": 0,
        "contempt": 0,
        "disgust": 0,
        "fear": 0,
        "happiness": 0,
        "neutral": 0.986,
        "sadness": 0.009,
        "surprise": 0.005
      },
      "blur": {
        "blurLevel": "low",
        "value": 0.06
      },
      "exposure": {
        "exposureLevel": "goodExposure",
        "value": 0.67
      },
      "noise": {
        "noiseLevel": "low",
        "value": 0
      },
      "makeup": {
        "eyeMakeup": true,
        "lipMakeup": true
      },
      "accessories": [],
      "occlusion": {
        "foreheadOccluded": false,
        "eyeOccluded": false,
        "mouthOccluded": false
      },
      "hair": {
        "bald": 0,
        "invisible": false,
        "hairColor": [
          {
            "color": "brown",
            "confidence": 1
          },
          {
            "color": "black",
            "confidence": 0.87
          },
          {
            "color": "other",
            "confidence": 0.51
          },
          {
            "color": "blond",
            "confidence": 0.08
          },
          {
            "color": "red",
            "confidence": 0.08
          },
          {
            "color": "gray",
            "confidence": 0.02
          }
        ]
      }
    }
  }
]
```

## <a name="find-similar-faces"></a>Rechercher des visages semblables

Cette opération utilise un seul visage détecté (la source) et recherche un ensemble d’autres visages (la cible) pour trouver des correspondances (recherche faciale par image). Quand il trouve une correspondance, il affiche l’ID du visage correspondant sur la console.

### <a name="detect-faces-for-comparison"></a>Détecter des visages pour les comparer

Avant de pouvoir comparer des images, vous devez d’abord les détecter dans les images. Exécutez cette commande comme vous l’avez fait dans la section [Détecter des visages](#detect-faces-in-an-image). Cette méthode de détection est optimisée pour les opérations de comparaison. Elle n’extrait pas les attributs de visage détaillés comme dans la section ci-dessus, et elle utilise un modèle de détection différent.

:::code language="shell" source="~/cognitive-services-quickstart-code/curl/face/detect.sh" ID="detect_for_similar":::

Recherchez la valeur `"faceId"` dans la réponse JSON et enregistrez-la dans un emplacement temporaire. Ensuite, rappelez la commande ci-dessus pour ces autres URL d’image et enregistrez les ID de visages. Vous utiliserez ces ID pour constituer un groupe cible de visages à partir duquel rechercher un visage similaire.

:::code source="~/cognitive-services-quickstart-code/curl/face/detect.sh" ID="similar_group":::

Enfin, détectez le visage source que vous utiliserez pour la comparaison, et enregistrez son ID. Conservez cet ID séparément des autres.

:::code source="~/cognitive-services-quickstart-code/curl/face/detect.sh" ID="similar_matcher":::

### <a name="find-matches"></a>Rechercher des correspondances

Copiez la commande ci-après dans un éditeur de texte.

:::code language="shell" source="~/cognitive-services-quickstart-code/curl/face/detect.sh" ID="similar":::

Ensuite, effectuez les modifications suivantes :
1. Affectez `Ocp-Apim-Subscription-Key` à votre clé d’abonnement Visage valide.
1. Modifiez la première partie de l’URL de requête pour qu’elle corresponde au point de terminaison associé à votre clé d’abonnement.

Utilisez le contenu JSON suivant pour la valeur `body` :

:::code language="JSON" source="~/cognitive-services-quickstart-code/curl/face/detect.sh" ID="similar_body":::

1. Utilisez l’ID de visage source pour `"faceId"`.
1. Collez les autres ID de visage comme des termes dans le tableau `"faceIds"`.

### <a name="examine-the-results"></a>Examiner les résultats

Vous recevrez une réponse JSON qui liste les ID des visages correspondant au visage de requête.

```json
[
    {
        "persistedFaceId" : "015839fb-fbd9-4f79-ace9-7675fc2f1dd9",
        "confidence" : 0.82
    },
    ...
] 
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous souhaitez nettoyer et supprimer un abonnement Cognitive Services, vous pouvez supprimer la ressource ou le groupe de ressources. La suppression du groupe de ressources efface également les autres ressources qui y sont associées.

* [Portail](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [Azure CLI](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez vu comment utiliser l’API REST Visage afin d’effectuer des tâches de reconnaissance faciale basiques. Pour plus d’informations sur la bibliothèque, reportez-vous à la documentation de référence.

> [!div class="nextstepaction"]
> [Référence sur l’API Visage](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236)

* [Qu’est ce que le service Visage ?](../../overview.md)