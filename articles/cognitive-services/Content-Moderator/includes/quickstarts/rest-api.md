---
title: Guide de démarrage rapide pour l’API REST Content Moderator
titleSuffix: Azure Cognitive Services
description: Dans ce guide de démarrage rapide, vous allez voir comment utiliser l’API REST Azure Content Moderator. Intégrez un logiciel de filtrage de contenu dans votre application afin de vous conformer aux réglementations ou de maintenir l’environnement souhaité pour vos utilisateurs.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: include
ms.date: 12/08/2020
ms.author: pafarley
ms.openlocfilehash: ebef33072b802ffc35b8c011d974dbcd203fa6e1
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97561475"
---
Commencez à utiliser l’API REST Azure Content Moderator. 

Content Moderator est un service d’IA qui vous permet de gérer le contenu potentiellement offensant, risqué ou indésirable. Utilisez le service de modération de contenu alimenté par l’IA pour analyser du texte, des images et des vidéos et appliquer automatiquement des indicateurs de contenu. Ensuite, intégrez votre application à l’outil Review, un environnement de modérateur en ligne pour une équipe de réviseurs humains. Intégrez un logiciel de filtrage de contenu dans votre application afin de vous conformer aux réglementations ou de maintenir l’environnement souhaité pour vos utilisateurs.

Utilisez l’API REST Content Moderator pour :

* Modérer du texte
* Modérer des images

## <a name="prerequisites"></a>Prérequis

* Abonnement Azure - [En créer un gratuitement](https://azure.microsoft.com/free/cognitive-services/)
* Une fois que vous avez votre abonnement Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesContentModerator"  title="Créer une ressource Content Moderator"  target="_blank">créez une ressource Content Moderator<span class="docon docon-navigate-external x-hidden-focus"></span></a> sur le portail Azure pour obtenir votre clé et votre point de terminaison. Attendez qu’elle se déploie, puis cliquez sur le bouton **Accéder à la ressource**.
    * Vous aurez besoin de la clé et du point de terminaison de la ressource que vous créez pour connecter votre application à Content Moderator. Vous collerez votre clé et votre point de terminaison dans le code ci-dessous plus loin dans le guide de démarrage rapide.
    * Vous pouvez utiliser le niveau tarifaire Gratuit (`F0`) pour tester le service, puis passer par la suite à un niveau payant pour la production.


## <a name="moderate-text"></a>Modérer du texte

Vous allez utiliser une commande comme celle qui suit pour appeler l’API Content Moderator en vue d’analyser un corps de texte et d’afficher les résultats dans la console.

:::code language="shell" source="~/cognitive-services-quickstart-code/curl/content-moderator/quickstart.sh" ID="textmod":::

Copiez la commande dans un éditeur de texte et apportez les modifications suivantes :

1. Affectez `Ocp-Apim-Subscription-Key` à votre clé d’abonnement Visage valide.
1. Modifiez la première partie de l’URL de requête pour qu’elle corresponde au point de terminaison associé à votre clé d’abonnement.
   [!INCLUDE [subdomains-note](../../../../../includes/cognitive-services-custom-subdomains-note.md)]
1. Si vous le souhaitez, vous pouvez remplacer le corps de la requête par n’importe quelle chaîne de texte que vous souhaitez analyser.

Après avoir apporté vos changements, ouvrez une invite de commandes et entrez la nouvelle commande. 

### <a name="examine-the-results"></a>Examiner les résultats

Les résultats de la modération du texte doivent s’afficher sous la forme de données JSON dans la fenêtre de console. Par exemple :

```json
{
  "OriginalText": "Is this a crap email abcdef@abcd.com, phone: 6657789887, IP: 255.255.255.255,\n1 Microsoft Way, Redmond, WA 98052\n",
  "NormalizedText": "Is this a crap email abide@ abed. com, phone: 6657789887, IP: 255. 255. 255. 255, \n1 Microsoft Way, Redmond, WA 98052",
  "AutoCorrectedText": "Is this a crap email abide@ abed. com, phone: 6657789887, IP: 255. 255. 255. 255, \n1 Microsoft Way, Redmond, WA 98052",
  "Misrepresentation": null,
  "PII": {
    "Email": [
      {
        "Detected": "abcdef@abcd.com",
        "SubType": "Regular",
        "Text": "abcdef@abcd.com",
        "Index": 21
      }
    ],
    "IPA": [
      {
        "SubType": "IPV4",
        "Text": "255.255.255.255",
        "Index": 61
      }
    ],
    "Phone": [
      {
        "CountryCode": "US",
        "Text": "6657789887",
        "Index": 45
      }
    ],
    "Address": [
      {
        "Text": "1 Microsoft Way, Redmond, WA 98052",
        "Index": 78
      }
    ]
  },
 "Classification": {
    "Category1": 
    {
      "Score": 0.5
    },
    "Category2": 
    {
      "Score": 0.6
    },
    "Category3": 
    {
      "Score": 0.5
    },
    "ReviewRecommended": true
  },
  "Language": "eng",
  "Terms": [
    {
      "Index": 10,
      "OriginalIndex": 10,
      "ListId": 0,
      "Term": "crap"
    }
  ],
  "Status": {
    "Code": 3000,
    "Description": "OK",
    "Exception": null
  },
  "TrackingId": "1717c837-cfb5-4fc0-9adc-24859bfd7fac"
}
```

Pour plus d’informations sur les attributs de texte que Content Moderator peut filtrer, consultez le guide [Concepts relatifs à la modération de texte](../../text-moderation-api.md).

## <a name="moderate-images"></a>Modérer des images

Vous allez utiliser une commande comme celle qui suit pour appeler l’API Content Moderator en vue de modérer une image distante et d’afficher les résultats dans la console.

:::code language="shell" source="~/cognitive-services-quickstart-code/curl/content-moderator/quickstart.sh" ID="imagemod":::

Copiez la commande dans un éditeur de texte et apportez les modifications suivantes :

1. Affectez `Ocp-Apim-Subscription-Key` à votre clé d’abonnement Visage valide.
1. Modifiez la première partie de l’URL de requête pour qu’elle corresponde au point de terminaison associé à votre clé d’abonnement.
1. Si vous le souhaitez, vous pouvez modifier l’URL de `"Value"` dans le corps de la requête pour qu’elle pointe vers l’image distante à modérer.

> [!TIP]
> Vous pouvez également modérer les images locales en passant leurs données d’octets dans le corps de la requête. Pour savoir comment procéder, consultez la [documentation de référence](https://westus.dev.cognitive.microsoft.com/docs/services/57cf753a3f9b070c105bd2c1/operations/57cf753a3f9b070868a1f66c).

Après avoir apporté vos changements, ouvrez une invite de commandes et entrez la nouvelle commande. 

### <a name="examine-the-results"></a>Examiner les résultats

Les résultats de la modération des images doivent s’afficher sous la forme de données JSON dans la fenêtre de console. 

```json
{
  "AdultClassificationScore": x.xxx,
  "IsImageAdultClassified": <Bool>,
  "RacyClassificationScore": x.xxx,
  "IsImageRacyClassified": <Bool>,
  "AdvancedInfo": [],
  "Result": false,
  "Status": {
    "Code": 3000,
    "Description": "OK",
    "Exception": null
  },
  "TrackingId": "<Request Tracking Id>"
}
```

Pour plus d’informations sur les attributs d’image que Content Moderator peut filtrer, consultez le guide [Concepts relatifs à la modération d’image](../../image-moderation-api.md).

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous souhaitez nettoyer et supprimer un abonnement Cognitive Services, vous pouvez supprimer la ressource ou le groupe de ressources. La suppression du groupe de ressources efface également les autres ressources qui y sont associées.

* [Portail](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [Azure CLI](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez appris à utiliser l’API REST Content Moderator pour effectuer des tâches de modération. Pour plus d’informations sur la modération des images ou d’autres éléments multimédias, consultez le guide conceptuel.

* [Concepts liés à la modération d’image](../../image-moderation-api.md)
* [Concepts de la modération de texte](../../text-moderation-api.md)
* [Qu’est-ce qu’Azure Content Moderator ?](../../overview.md)