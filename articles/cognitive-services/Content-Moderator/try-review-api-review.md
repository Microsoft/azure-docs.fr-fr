---
title: Créer des révisions de modération avec la console d’API REST - Content Moderator
titleSuffix: Azure Cognitive Services
description: Utilisez les API de révision Azure Content Moderator afin de créer des révisions de texte ou d'image pour la modération humaine.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: conceptual
ms.date: 03/18/2019
ms.author: pafarley
ms.openlocfilehash: 479c7c455f07d098edd327196803e85df24dfb6d
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96905129"
---
# <a name="create-human-reviews-api-console"></a>Créer des révisions humaines (console d’API)

Les [révisions](./review-api.md#reviews) stockent et affichent le contenu que les modérateurs humains vont évaluer. Lorsqu’un utilisateur termine une révision, les résultats sont envoyés à un point de terminaison de rappel spécifié. Dans ce guide, vous apprendrez à configurer des révisions à l’aide des API REST de révision via la console d’API. Une fois que vous aurez compris la structure des API, vous pourrez facilement déplacer ces appels vers n’importe quelle plateforme compatible avec REST.

## <a name="prerequisites"></a>Prérequis

- Connectez-vous ou créez un compte sur le site de l’[outil de révision](https://contentmoderator.cognitive.microsoft.com/) de Content Moderator.

## <a name="create-a-review"></a>Créer une révision

Pour créer une révision, accédez à la page de référence de l’API **[Révision - Créer](https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/580519483f9b0709fc47f9c4)** et sélectionnez le bouton de votre région clé (que vous pouvez trouver dans l’URL de point de terminaison à la page **Informations d’identification** de l’[Outil de révision](https://contentmoderator.cognitive.microsoft.com/)). La console d’API, qui permet de construire et d’exécuter des appels d’API REST, se lance.

![Sélection d’une région Révision - Get](images/test-drive-region.png)

### <a name="enter-rest-call-parameters"></a>Entrer les paramètres d’appel REST

Entrez les valeurs de **teamName** et **Ocp-Apim-Subscription-Key** :

- **teamName** : ID d’équipe que vous avez créé lors de la configuration de votre compte [Outil de révision](https://contentmoderator.cognitive.microsoft.com/) (dans le champ **Id**, sur l’écran Informations d’identification de votre Outil de révision).
- **Ocp-Apim-Subscription-Key** : votre clé Content Moderator. Vous la trouverez sous l’onglet **Paramètres** de [l’Outil de révision](https://contentmoderator.cognitive.microsoft.com).

### <a name="enter-a-review-definition"></a>Entrez une définition de révision

Modifiez la zone **Corps de la demande** pour entrer la requête JSON avec les champs suivants :

- **Métadonnées** : paires clé-valeur personnalisées à retourner à votre point de terminaison de rappel. Si la clé est un code court qui est défini dans l’[outil de révision](https://contentmoderator.cognitive.microsoft.com), elle s’affiche sous forme d’une balise.
- **Contenu** : Dans le cas de contenu de type Image et Vidéo, il s’agit d’une chaîne URL pointant vers le contenu. Pour le contenu de type Texte, il s’agit de la chaîne de texte réelle.
- **ContentId** : chaîne d’identificateur personnalisée. cette chaîne est transmise à l'API et retournée via le rappel. Elle est utile pour associer des métadonnées ou des identificateurs internes aux résultats d’un travail de modération.
- **CallbackEndpoint** : (Facultatif) URL permettant de recevoir les informations de rappel une fois la révision terminée.

Le corps de la demande par défaut présente des exemples des différents types de révisions que vous pouvez créer :

```json
[Image]
[
  {
    "Metadata": [
      {
        "Key": "string",
        "Value": "string"
      }
    ],
    "Type": "Image",
    "Content": "<Content Url>",
    "ContentId": "<Your identifier for this content>",
    "CallbackEndpoint": "<Url where you would receive callbacks>"
  }
]
[Text]
[
  {
    "Metadata": [
      {
        "Key": "string",
        "Value": "string"
      }
    ],
    "Type": "Text",
    "Content": "<Your Text Content>",
    "ContentId": "<Your identifier for this content>",
    "CallbackEndpoint": "<Url where you would receive callbacks>"
  }
]
[Video]
[
  {
    "VideoFrames":[
      {
          "Id": "<Frame Id>",
          "Timestamp": "<Frame Timestamp",
          "FrameImage":"<Frame Image URL",
          "Metadata": [
            {
              "Key": "<Key>",
              "Value": "<Value"
            }
          ],
          "ReviewerResultTags": [
          ]
    ], 
    "Metadata": [
      {
        "Key": "string",
        "Value": "string"
      },
      //For encrypted Videos
        {
          "Key": "protectedType",
          "Value": "AES or FairPlay or Widevine or Playready"
        },
        {
          "Key": "authenticationToken",
          "Value": "your viewtoken(In case of Video Indexer AES encryption type, this value is viewtoken from breakdown json)"
        },
      //For FairPlay encrypted type video include certificateUrl as well
        {
          "Key": "certificateUrl",
          "Value": "your certificate url"
        }
    ],
    "Type": "Video",
    "Content": "<Stream Url>",
    "ContentId": "<Your identifier for this content>",
    "CallbackEndpoint": "<Url where you would receive callbacks>",
    [Optional]
    "Timescale": "<Timescale of the video>
  }
]
```

### <a name="submit-your-request"></a>Soumettre votre requête
  
Sélectionnez **Envoyer**. Si l’opération réussit, l’**État de la réponse** est `200 OK` et la zone **Contenu de la réponse** affiche l’ID de la révision. Copiez cet ID pour l’utiliser dans les étapes suivantes.

![La zone Contenu de la réponse de la console Review - Create (Révision - Créer) affiche l’ID de révision](images/test-drive-review-2.PNG)

### <a name="examine-the-new-review"></a>Examiner la nouvelle révision

Dans l’[outil de révision](https://contentmoderator.cognitive.microsoft.com), sélectionnez **Révision** > **Image**/**Texte**/**Vidéo** (selon le contenu que vous avez utilisé). Le contenu que vous avez chargé doit apparaître, prêt pour la révision humaine.

![Image de l’outil de révision présentant un ballon de football](images/test-drive-review-5.PNG)

## <a name="get-review-details"></a>Obtenir les informations d’une révision

Pour récupérer des informations sur une révision existante, accédez à la page de référence d’API [Révision - Get](https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/580519483f9b0709fc47f9c2) et sélectionnez le bouton de votre région (la région dans laquelle votre clé est administrée).

![Flux de travail - Sélection de région](images/test-drive-region.png)

Entrez les paramètres d’appel REST, comme dans la section ci-dessus. Pour cette étape, **reviewId** représente la chaîne d’ID unique que vous avez reçue lorsque vous avez créé la révision.

![Obtention des résultats dans la console Review - Create (Révision - Créer)](images/test-drive-review-3.PNG)
  
Sélectionnez **Envoyer**. Si l’opération réussit, l’**État de la réponse** est `200 OK` et la zone **Contenu de la réponse** affiche les informations de la révision au format JSON, comme suit :

```json
{  
  "reviewId":"201712i46950138c61a4740b118a43cac33f434",
  "subTeam":"public",
  "status":"Complete",
  "reviewerResultTags":[  
    {  
      "key":"a",
      "value":"False"
    },
    {  
      "key":"r",
      "value":"True"
    },
    {  
      "key":"sc",
      "value":"True"
    }
  ],
  "createdBy":"<teamname>",
  "metadata":[  
    {  
      "key":"sc",
      "value":"true"
    }
  ],
  "type":"Image",
  "content":"https://reviewcontentprod.blob.core.windows.net/<teamname>/IMG_201712i46950138c61a4740b118a43cac33f434",
  "contentId":"0",
  "callbackEndpoint":"<callbackUrl>"
}
```

Tenez compte des champs suivants dans la réponse :

- **statut**
- **reviewerResultTags** : s’affiche si toutes les balises ont été ajoutées manuellement par l’équipe de révision humaine (affichage du champ **createdBy**).
- **metadata** : montre les balises initialement ajoutées lors de la révision, avant les modifications apportées par l’équipe de révision manuelle.

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide, vous avez appris à créer des révisions de modération du contenu à l’aide de l’API REST. Nous allons maintenant intégrer des révisions à un scénario de modération de bout en bout en utilisant le tutoriel sur la [modération dans l’e-commerce](./ecommerce-retail-catalog-moderation.md).