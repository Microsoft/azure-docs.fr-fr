---
title: Méthode Detect de Translator
titleSuffix: Azure Cognitive Services
description: Identifiez la langue d’un fragment de texte avec la méthode Detect d’Azure Cognitive Services Translator.
services: cognitive-services
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: reference
ms.date: 02/01/2019
ms.author: swmachan
ms.openlocfilehash: 12029d7a77eb7cbbb10cb5330eb18d6316520a2d
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95021318"
---
# <a name="translator-30-detect"></a>Translator 3.0 : Detect

Identifie la langue d’un texte.

## <a name="request-url"></a>URL de la demande

Envoyez une demande `POST` à :

```HTTP
https://api.cognitive.microsofttranslator.com/detect?api-version=3.0
```

## <a name="request-parameters"></a>Paramètres de la demande

Les paramètres de demande transmis à la chaîne de requête sont les suivants :

<table width="100%">
  <th width="20%">Paramètre de requête.</th>
  <th>Description</th>
  <tr>
    <td>api-version</td>
    <td>*Paramètre obligatoire*.<br/>Version de l’API demandée par le client. La valeur doit être `3.0`.</td>
  </tr>
</table> 

Les en-têtes de demande sont les suivants :

<table width="100%">
  <th width="20%">headers</th>
  <th>Description</th>
  <tr>
    <td>En-tête(s) d’authentification</td>
    <td><em>En-tête de demande obligatoire</em>.<br/>Voir les <a href="/azure/cognitive-services/translator/reference/v3-0-reference#authentication">options disponibles pour l’authentification</a>.</td>
  </tr>
  <tr>
    <td>Content-Type</td>
    <td>*En-tête de demande obligatoire*.<br/>Spécifie le type de contenu de la charge utile. Les valeurs possibles sont les suivantes : `application/json`.</td>
  </tr>
  <tr>
    <td>Content-Length</td>
    <td>*En-tête de demande obligatoire*.<br/>Longueur du corps de la demande.</td>
  </tr>
  <tr>
    <td>X-ClientTraceId</td>
    <td>*Facultatif*.<br/>GUID généré par le client pour identifier de façon unique la demande. Vous pouvez omettre cet en-tête si vous incluez l’ID de trace dans la chaîne de requête à l’aide d’un paramètre de requête nommé `ClientTraceId`.</td>
  </tr>
</table> 

## <a name="request-body"></a>Corps de la demande

Le corps de la demande est un tableau JSON. Chaque élément du tableau est un objet JSON avec une propriété de chaîne nommée `Text`. La détection de la langue est appliquée à la valeur de la propriété `Text`. La détection automatique de la langue fonctionne mieux avec un texte d’entrée plus long. Voici un exemple de corps de la demande :

```json
[
    { "Text": "Ich würde wirklich gern Ihr Auto um den Block fahren ein paar Mal." }
]
```

Les limites suivantes s'appliquent :

* Le tableau ne peut pas compter plus de 100 éléments.
* L’intégralité du texte inclus dans la demande ne peut pas dépasser 50 000 caractères, espaces compris.

## <a name="response-body"></a>Response body

Une réponse correcte est un tableau JSON avec un résultat pour chaque chaîne dans le tableau d’entrée. Un objet de résultat inclut les propriétés suivantes :

  * `language`: code de la langue détectée.

  * `score`: valeur flottante indiquant le niveau de confiance dans le résultat. Le score est compris entre zéro et un, un score faible indiquant un niveau de confiance bas.

  * `isTranslationSupported`: valeur booléenne true si la langue détectée est l’une des langues prises en charge pour la traduction de texte.

  * `isTransliterationSupported`: valeur booléenne true si la langue détectée est l’une des langues prises en charge pour la translittération.
  
  * `alternatives`: tableau des autres langues possibles. Chaque élément du tableau est un autre objet avec les mêmes propriétés que celles répertoriées ci-dessus : `language`, `score`, `isTranslationSupported` et `isTransliterationSupported`.

Exemple de réponse JSON :

```json
[
  {
    "language": "de",
    "score": 0.92,
    "isTranslationSupported": true,
    "isTransliterationSupported": false,
    "alternatives": [
      {
        "language": "pt",
        "score": 0.23,
        "isTranslationSupported": true,
        "isTransliterationSupported": false
      },
      {
        "language": "sk",
        "score": 0.23,
        "isTranslationSupported": true,
        "isTransliterationSupported": false
      }
    ]
  }
]
```

## <a name="response-headers"></a>En-têtes de réponse

<table width="100%">
  <th width="20%">headers</th>
  <th>Description</th>
  <tr>
    <td>X-RequestId</td>
    <td>Valeur générée par le service pour identifier la demande. Elle sert à des fins de dépannage.</td>
  </tr>
</table> 

## <a name="response-status-codes"></a>Codes d’état de réponse

Voici les codes d’état HTTP qu’une demande peut retourner. 

<table width="100%">
  <th width="20%">Code d’état</th>
  <th>Description</th>
  <tr>
    <td>200</td>
    <td>Réussite.</td>
  </tr>
  <tr>
    <td>400</td>
    <td>L’un des paramètres de requête est manquant ou non valide. Corrigez les paramètres de demande avant de réessayer.</td>
  </tr>
  <tr>
    <td>401</td>
    <td>Il n’a pas été possible d’authentifier la demande. Vérifiez que les informations d’identification sont spécifiées et valides.</td>
  </tr>
  <tr>
    <td>403</td>
    <td>La demande n’est pas autorisée. Vérifiez le message d’erreur détaillé. Cela indique souvent que toutes les traductions gratuites fournies avec un abonnement d’essai ont été utilisées.</td>
  </tr>
  <tr>
    <td>429</td>
    <td>Le serveur a rejeté la requête, car le client a dépassé les limites de requête.</td>
  </tr>
  <tr>
    <td>500</td>
    <td>Une erreur inattendue s’est produite. Si l’erreur persiste, signalez-la en fournissant les informations suivantes : date et heure de la défaillance, identificateur de la demande dans l’en-tête de réponse,`X-RequestId` et identificateur du client dans l’en-tête de demande `X-ClientTraceId`.</td>
  </tr>
  <tr>
    <td>503</td>
    <td>Serveur temporairement indisponible. Relancez la requête. Si l’erreur persiste, signalez-la en fournissant les informations suivantes : date et heure de la défaillance, identificateur de la demande dans l’en-tête de réponse,`X-RequestId` et identificateur du client dans l’en-tête de demande `X-ClientTraceId`.</td>
  </tr>
</table> 

Si une erreur se produit, la requête renvoie également une réponse d'erreur JSON. Le code d’erreur est un nombre à 6 chiffres qui combine le code d’état HTTP à 3 chiffres et un nombre à 3 chiffres qui sert à catégoriser plus précisément l’erreur. Vous trouverez les codes d’erreur les plus courants sur la [page Référence de Translator v3](./v3-0-reference.md#errors). 

## <a name="examples"></a>Exemples

L’exemple suivant montre comment récupérer les langues prises en charge pour la traduction du texte.

```curl
curl -X POST "https://api.cognitive.microsofttranslator.com/detect?api-version=3.0" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json" -d "[{'Text':'What language is this text written in?'}]"
```