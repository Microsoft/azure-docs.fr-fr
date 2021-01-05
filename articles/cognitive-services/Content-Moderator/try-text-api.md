---
title: Modérer du texte à l’aide de l’API Modération du texte - Content Moderator
titleSuffix: Azure Cognitive Services
description: Évaluez la modération du texte à l’aide de l’API Modération de texte dans la console en ligne.
services: cognitive-services
author: PatrickFarley
ms.author: pafarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: conceptual
ms.date: 05/29/2019
ms.openlocfilehash: 664c4289cbfa1f6ce2fce9f9f83b0240bd2d592c
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96001998"
---
# <a name="moderate-text-from-the-api-console"></a>Modérer du texte à partir de la console d’API

Utilisez l’[API Modération de texte](https://westus.dev.cognitive.microsoft.com/docs/services/57cf753a3f9b070c105bd2c1/operations/57cf753a3f9b070868a1f66f) dans Azure Content Moderator pour analyser votre contenu de texte à la recherche de propos injurieux et le comparer aux listes personnalisées et partagées.

## <a name="get-your-api-key"></a>Obtenir votre clé API

Avant de pouvoir tester l’API dans la console en ligne, vous avez besoin de votre clé d’abonnement. Elle se trouve sous l’onglet **Paramètres**, dans la zone **Ocp-Apim-Subscription-Key**. Pour plus d’informations, consultez [What is Content Moderator?](overview.md) (Présentation de Content Moderator).

## <a name="navigate-to-the-api-reference"></a>Accéder aux informations de référence sur l’API

Accédez aux [Informations de référence sur l’API Modération du texte](https://westus.dev.cognitive.microsoft.com/docs/services/57cf753a3f9b070c105bd2c1/operations/57cf753a3f9b070868a1f66f). 

  La page **Text - Screen** (Texte - Passer au crible) s’ouvre.

## <a name="open-the-api-console"></a>Ouvrir la console d’API

Pour l’option **Open API testing console** (Ouvrir la console de test d’API), sélectionnez la région qui décrit le mieux votre emplacement. 

  ![Sélection d’une région dans la page Text - Screen (Texte - Passer au crible)](images/test-drive-region.png)

  La console d’API **Text - Screen** (Texte - Passer au crible) s’ouvre.

## <a name="select-the-inputs"></a>Sélectionner les entrées

### <a name="parameters"></a>Paramètres

Sélectionnez les paramètres de requête que vous voulez utiliser dans le passage au crible de votre texte. Pour cet exemple, utilisez la valeur par défaut pour le paramètre **language**. Vous pouvez également le laisser vide, car l’opération détecte automatiquement la langue probable dans le cadre de son exécution.

> [!NOTE]
> Pour le paramètre **langue**, attribuez `eng` ou laissez-le vide pour voir la réponse de la **classification** assistée par ordinateur (fonctionnalité en préversion). **Cette fonctionnalité prend en charge uniquement l’anglais**.
>
> Pour la détection des **termes injurieux**, utilisez le [code ISO 639-3](http://www-01.sil.org/iso639-3/codes.asp) des langues prises en charge répertoriées dans cet article, ou laissez le paramètre vide.

Pour les paramètres **autocorrect**, **PII** et **classify** (préversion), sélectionnez **true**. Laissez le champ **ListId** vide.

  ![Paramètres de requête de la console Text - Screen (Texte - Passer au crible)](images/text-api-console-inputs.PNG)

### <a name="content-type"></a>Type de contenu

Pour **Content-Type**, sélectionnez le type de contenu à passer au crible. Pour cet exemple, utilisez le type de contenu **text/plain** par défaut. Dans la zone **Ocp-Apim-abonnement-Key**, entrez votre clé d’abonnement.

### <a name="sample-text-to-scan"></a>Exemple de texte à analyser

Dans la zone **Request body** (Corps de la requête), entrez du texte. L’exemple suivant montre une faute de frappe intentionnelle dans le texte.

```
Is this a grabage or crap email abcdef@abcd.com, phone: 4255550111, IP: 
255.255.255.255, 1234 Main Boulevard, Panapolis WA 96555.
```

## <a name="analyze-the-response"></a>Analyser la réponse

La réponse suivante montre les différentes informations fournies par l’API. Elle contient les propos injurieux potentiels, les données personnelles, la classification (préversion) et la version corrigée automatiquement.

> [!NOTE]
> La fonctionnalité « Classification » assistée par ordinateur est en préversion et prend en charge uniquement l’anglais.

```json
{
   "original_text":"Is this a grabage or crap email abcdef@abcd.com, phone: 
   6657789887, IP: 255.255.255.255, 1 Microsoft Way, Redmond, WA 98052.",
   "normalized_text":"   grabage  crap email abcdef@abcd.com, phone: 
   6657789887, IP: 255.255.255.255, 1 Microsoft Way, Redmond, WA 98052.",
   "auto_corrected_text":"Is this a garbage or crap email abcdef@abcd.com, phone: 
   6657789887, IP: 255.255.255.255, 1 Microsoft Way, Redmond, WA 98052.",
   "status":{
      "code":3000,
      "description":"OK"
   },
   "pii":{
      "email":[
         {
            "detected":"abcdef@abcd.com",
            "sub_type":"Regular",
            "text":"abcdef@abcd.com",
            "index":32
         }
      ],
      "ssn":[

      ],
      "ipa":[
         {
            "sub_type":"IPV4",
            "text":"255.255.255.255",
            "index":72
         }
      ],
      "phone":[
         {
            "country_code":"US",
            "text":"6657789887",
            "index":56
         }
      ],
      "address":[
         {
            "text":"1 Microsoft Way, Redmond, WA 98052",
            "index":89
         }
      ]
   },
   "language":"eng",
   "terms":[
      {
         "index":12,
         "original_index":21,
         "list_id":0,
         "term":"crap"
      }
   ],
   "tracking_id":"WU_ibiza_65a1016d-0f67-45d2-b838-b8f373d6d52e_ContentModerator.
   F0_fe000d38-8ecd-47b5-a8b0-4764df00e3b5"
}
```

Pour obtenir une explication détaillée de toutes les sections de la réponse JSON, reportez-vous au guide conceptuel [Modération du texte](text-moderation-api.md).

## <a name="next-steps"></a>Étapes suivantes

Utilisez l’API REST dans votre code ou suivez le [Démarrage rapide du kit SDK .NET](./client-libraries.md?pivots=programming-language-csharp%253fpivots%253dprogramming-language-csharp) pour l’intégrer à votre application.