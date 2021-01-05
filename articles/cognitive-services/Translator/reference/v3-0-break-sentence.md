---
title: Méthode BreakSentence de Translator
titleSuffix: Azure Cognitive Services
description: La méthode BreakSentence de Translator identifie le positionnement des limites de phrases dans du texte.
services: cognitive-services
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: reference
ms.date: 08/06/2020
ms.author: swmachan
ms.openlocfilehash: 27069cf38da2567a3af3ed0fb65baead43665186
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94369475"
---
# <a name="translator-30-breaksentence"></a>Translator 3.0 : BreakSentence

Identifie le positionnement des limites de phrases dans du texte.

## <a name="request-url"></a>URL de la demande

Envoyez une demande `POST` à :

```HTTP
https://api.cognitive.microsofttranslator.com/breaksentence?api-version=3.0
```

## <a name="request-parameters"></a>Paramètres de la demande

Les paramètres de demande transmis à la chaîne de requête sont les suivants :

| Paramètre de requête | Description |
| -------| ----------- |
| api-version <img width=200/>   | **Paramètre de requête obligatoire**.<br/>Version de l’API demandée par le client. La valeur doit être `3.0`. |
| langage | **Paramètre de requête facultatif**.<br/>Balise de langue identifiant la langue du texte d’entrée. Si aucun code n’est spécifié, la détection automatique de la langue est appliquée. |
| script    | **Paramètre de requête facultatif**.<br/>Balise de script qui identifie le script utilisé par le texte d’entrée. Si aucun script n’est spécifié, le script par défaut de la langue est supposé.  | 

Les en-têtes de demande sont les suivants :

| headers | Description |
| ------- | ----------- |
| En-tête(s) d’authentification <img width=200/>  | **En-tête de demande obligatoire**.<br/>Voir les <a href="/azure/cognitive-services/translator/reference/v3-0-reference#authentication">options disponibles pour l’authentification</a>. |
| Content-Type | **En-tête de demande obligatoire**.<br/>Spécifie le type de contenu de la charge utile. Les valeurs possibles sont les suivantes : `application/json`. |
| Content-Length    | **En-tête de demande obligatoire**.<br/>Longueur du corps de la demande.  | 
| X-ClientTraceId   | **Facultatif**.<br/>GUID généré par le client pour identifier de façon unique la demande. Vous pouvez omettre cet en-tête si vous incluez l’ID de trace dans la chaîne de requête à l’aide d’un paramètre de requête nommé `ClientTraceId`.  | 

## <a name="request-body"></a>Corps de la demande

Le corps de la demande est un tableau JSON. Chaque élément du tableau est un objet JSON avec une propriété de chaîne nommée `Text`. Les limites de phrases sont calculées pour la valeur de la propriété `Text`. Un exemple de corps de demande avec un seul élément de texte se présente ainsi :

```json
[
    { "Text": "How are you? I am fine. What did you do today?" }
]
```

Les limites suivantes s'appliquent :

* Le tableau ne peut pas compter plus de 100 éléments.
* La valeur texte d’un élément de tableau ne peut pas dépasser 50 000 caractères, espaces comprises.
* L’intégralité du texte inclus dans la demande ne peut pas dépasser 50 000 caractères, espaces compris.
* Si le paramètre de requête `language` est spécifié, tous les éléments du tableau doivent être dans la même langue. Sinon, la détection automatique de la langue est appliquée indépendamment à chaque élément du tableau.

## <a name="response-body"></a>Response body

Une réponse correcte est un tableau JSON avec un résultat pour chaque chaîne dans le tableau d’entrée. Un objet de résultat inclut les propriétés suivantes :

  * `sentLen`: tableau d’entiers représentant les longueurs des phrases dans l'élément de texte. La longueur du tableau correspond au nombre de phrases, et les valeurs sont les longueurs des phrases. 

  * `detectedLanguage`: objet décrivant la langue détectée au moyen des propriétés suivantes :

     * `language`: code de la langue détectée.

     * `score`: valeur flottante indiquant le niveau de confiance dans le résultat. Le score est compris entre zéro et un, un score faible indiquant un niveau de confiance bas.
     
    Notez que la propriété `detectedLanguage` n’est présente dans l’objet de résultat que quand la détection automatique de la langue est demandée.

Exemple de réponse JSON :

```json
[
  {
    "sentLen": [ 13, 11, 22 ]
    "detectedLanguage": {
      "language": "en",
      "score": 401
    },
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

L’exemple suivant montre comment obtenir les limites d’une phrase unique. La langue de la phrase est détectée automatiquement par le service.

```curl
curl -X POST "https://api.cognitive.microsofttranslator.com/breaksentence?api-version=3.0" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json" -d "[{'Text':'How are you? I am fine. What did you do today?'}]"
```