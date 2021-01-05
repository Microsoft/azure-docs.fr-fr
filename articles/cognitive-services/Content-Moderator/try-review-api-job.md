---
title: Utiliser des travaux de modération avec la console d’API REST – Content Moderator
titleSuffix: Azure Cognitive Services
description: Utilisez les opérations de travail de l’API Review (Révision) pour initier des travaux de modération du contenu de bout en bout pour le contenu d’image ou de texte dans Azure Content Moderator.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: conceptual
ms.date: 10/24/2019
ms.author: pafarley
ms.openlocfilehash: 924c21037a464770fac13c9b45ddcf261ff5a058
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96905142"
---
# <a name="define-and-use-moderation-jobs-api-console"></a>Définir et utiliser des travaux de modération (console d’API)

Un travail de modération est une forme de wrapper des fonctionnalités de modération du contenu, de workflows et de révision. Ce guide explique comment utiliser les API REST pour lancer et vérifier des travaux de modération du contenu. Une fois que vous aurez compris la structure des API, vous pourrez facilement déplacer ces appels vers n’importe quelle plateforme compatible avec REST.

## <a name="prerequisites"></a>Prérequis

- Connectez-vous ou créez un compte sur le site de l’[outil de révision](https://contentmoderator.cognitive.microsoft.com/) de Content Moderator.
- (Facultatif) [Définissez un flux de travail personnalisé](./Review-Tool-User-Guide/Workflows.md) à utiliser avec votre travail ; vous pouvez également utiliser le flux de travail par défaut.

## <a name="create-a-job"></a>Créer un travail

Pour créer un travail de modération, accédez à la page de référence d’API [Travail - Créer](https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/580519483f9b0709fc47f9c5) et sélectionnez le bouton correspondant à la région de votre abonnement. Vous pouvez trouver votre région dans l’URL du point de terminaison dans la page **Informations d’identification** de l’[outil de révision](https://contentmoderator.cognitive.microsoft.com/). La console d’API, qui permet de construire et d’exécuter des appels d’API REST, se lance.

![Sélection d’une région sur la page Travail – Créer](images/test-drive-job-1.png)

### <a name="enter-rest-call-parameters"></a>Entrer les paramètres d’appel REST

Entrez les valeurs suivantes pour construire l’appel REST :

- **teamName** : ID d’équipe que vous avez créé lors de la configuration de votre compte [Outil de révision](https://contentmoderator.cognitive.microsoft.com/) (dans le champ **Id**, sur l’écran Informations d’identification de votre Outil de révision).
- **ContentType** : « Image », « Text » ou « Video ».
- **ContentId** : chaîne d’identificateur personnalisée. cette chaîne est transmise à l'API et retournée via le rappel. Elle est utile pour associer des métadonnées ou des identificateurs internes aux résultats d’un travail de modération.
- **Workflowname** : nom du flux de travail que vous avez créé (ou « default » pour le flux de travail par défaut).
- **CallbackEndpoint** : (Facultatif) URL permettant de recevoir les informations de rappel une fois la révision terminée.
- **Ocp-Apim-Subscription-Key** : votre clé Content Moderator. Vous la trouverez sous l’onglet **Paramètres** de l’[outil de révision](https://contentmoderator.cognitive.microsoft.com).

### <a name="fill-in-the-request-body"></a>Remplir le corps de la demande

Le corps de votre appel REST contient un seul champ, **ContentValue**. Collez le contenu texte brut si vous modérez du texte, ou entrez une URL d’image ou de vidéo s’il s’agit d’une image ou d’une vidéo. Vous pouvez utiliser l’URL de l’exemple d’image suivant : [https://moderatorsampleimages.blob.core.windows.net/samples/sample2.jpg](https://moderatorsampleimages.blob.core.windows.net/samples/sample2.jpg).

![Paramètres de la requête, en-têtes et zone Corps de la demande dans la console Travail - Créer](images/job-api-console-inputs.PNG)

### <a name="submit-your-request"></a>Soumettre votre requête

Sélectionnez **Envoyer**. Si l’opération réussit, **l’État de la réponse** est `200 OK` et la zone **Contenu de la réponse** indique l’ID du travail. Copiez cet ID pour l’utiliser dans les étapes suivantes.

![La zone Contenu de la réponse de la console Review - Create (Révision - Créer) affiche l’ID de révision](images/test-drive-job-3.PNG)

## <a name="get-job-status"></a>Obtenir l’état de la tâche

Pour récupérer l’état et des informations sur un travail en cours d’exécution ou terminé, accédez à la page de référence API [Travail – Get](https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/580519483f9b0709fc47f9c3) et sélectionnez le bouton de votre région (celle dans laquelle votre clé est administrée).

![Sélection d’une région Révision – Get](images/test-drive-region.png)

Entrez les paramètres d’appel REST, comme dans la section ci-dessus. À cette étape, **JobId** correspond à la chaîne d’ID unique que vous avez reçue à la création du travail. Sélectionnez **Envoyer**. Si l’opération réussit, **l’État de la réponse** est `200 OK` et la zone **Contenu de la réponse** indique le travail au format JSON :

```json
{  
  "Id":"2018014caceddebfe9446fab29056fd8d31ffe",
  "TeamName":"some team name",
  "Status":"Complete",
  "WorkflowId":"OCR",
  "Type":"Image",
  "CallBackEndpoint":"",
  "ReviewId":"201801i28fc0f7cbf424447846e509af853ea54",
  "ResultMetaData":[  
    {  
      "Key":"hasText",
      "Value":"True"
    },
    {  
      "Key":"ocrText",
      "Value":"IF WE DID \r\nALL \r\nTHE THINGS \r\nWE ARE \r\nCAPABLE \r\nOF DOING, \r\nWE WOULD \r\nLITERALLY \r\nASTOUND \r\nOURSELVE \r\n"
    }
  ],
  "JobExecutionReport":[  
    {  
      "Ts":"2018-01-07T00:38:29.3238715",
      "Msg":"Posted results to the Callbackendpoint: https://requestb.in/vxke1mvx"
    },
    {  
      "Ts":"2018-01-07T00:38:29.2928416",
      "Msg":"Job marked completed and job content has been removed"
    },
    {  
      "Ts":"2018-01-07T00:38:29.0856472",
      "Msg":"Execution Complete"
    },
    {  
      "Ts":"2018-01-07T00:38:26.7714671",
      "Msg":"Successfully got hasText response from Moderator"
    },
    {  
      "Ts":"2018-01-07T00:38:26.4181346",
      "Msg":"Getting hasText from Moderator"
    },
    {  
      "Ts":"2018-01-07T00:38:25.5122828",
      "Msg":"Starting Execution - Try 1"
    }
  ]
}
```

![Réponse à l’appel REST Travail – Get](images/test-drive-job-5.png)

### <a name="examine-the-new-reviews"></a>Examiner la ou les nouvelles révisions

Si votre travail de contenu a entraîné la création d’une révision, vous pouvez l’afficher dans [l’Outil de révision](https://contentmoderator.cognitive.microsoft.com). Sélectionnez **Révision** > **Image**/**Texte**/**Vidéo** (selon le contenu que vous avez utilisé). Le contenu apparaît, prêt à faire l’objet d’une révision humaine. Une fois qu’un modérateur a examiné les tags affectés automatiquement ainsi que les données de prédiction et soumis une décision finale, l’API des travaux soumet toutes ces informations au point de terminaison de rappel désigné.

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide, vous avez appris à créer et à interroger des révisions de modération du contenu avec l’API REST. Intégrez maintenant ces travaux à un scénario de modération de bout en bout, par exemple en suivant le tutoriel sur la [modération dans l’e-commerce](./ecommerce-retail-catalog-moderation.md).