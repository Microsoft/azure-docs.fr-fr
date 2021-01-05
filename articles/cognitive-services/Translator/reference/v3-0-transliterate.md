---
title: Méthode Transliterate de Translator
titleSuffix: Azure Cognitive Services
description: Convertissez du texte dans une même langue depuis un script vers un autre à l’aide de la méthode Transliterate de Translator.
services: cognitive-services
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: reference
ms.date: 02/01/2019
ms.author: swmachan
ms.openlocfilehash: 4b985d7b2eb07ca8bbd3556a237fbcaba5392fe5
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95016464"
---
# <a name="translator-30-transliterate"></a>Translator 3.0 : Transliterate

Convertit du texte dans une langue d’un script vers un autre script.

## <a name="request-url"></a>URL de la demande

Envoyez une demande `POST` à :

```HTTP
https://api.cognitive.microsofttranslator.com/transliterate?api-version=3.0
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
  <tr>
    <td>langage</td>
    <td>*Paramètre obligatoire*.<br/>Spécifie la langue du texte à convertir d’un script à un autre. Les langues possibles sont répertoriées dans l’étendue `transliteration` obtenue en interrogeant le service pour connaître les [langues prises en charge](./v3-0-languages.md).</td>
  </tr>
  <tr>
    <td>fromScript</td>
    <td>*Paramètre obligatoire*.<br/>Spécifie le script utilisé par le texte d’entrée. Recherchez les [langues prises en charge](./v3-0-languages.md) à l’aide de l’étendue `transliteration`, afin de trouver des scripts d’entrée disponibles pour la langue sélectionnée.</td>
  </tr>
  <tr>
    <td>toScript</td>
    <td>*Paramètre obligatoire*.<br/>Spécifie le script de sortie. Recherchez les [langues prises en charge](./v3-0-languages.md) à l’aide de l’étendue `transliteration`, afin de trouver des scripts de sortie disponibles pour l’association de langue et de script d’entrée sélectionnée.</td>
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

Le corps de la demande est un tableau JSON. Chaque élément du tableau est un objet JSON avec une propriété de chaîne nommée `Text`, qui représente la chaîne à convertir.

```json
[
    {"Text":"こんにちは"},
    {"Text":"さようなら"}
]
```

Les limites suivantes s'appliquent :

* Le tableau ne peut pas compter plus de 10 éléments.
* La valeur de texte d’un élément de tableau ne peut pas dépasser 1 000 caractères, espaces compris.
* L’intégralité du texte inclus dans la requête ne peut pas dépasser 5 000 caractères, espaces compris.

## <a name="response-body"></a>Response body

Une réponse correcte est un tableau JSON avec un résultat pour chaque élément dans le tableau d’entrée. Un objet de résultat inclut les propriétés suivantes :

  * `text`: chaîne qui est le résultat de la conversion de la chaîne d’entrée en script de sortie.
  
  * `script`: chaîne spécifiant le script utilisé dans la sortie.

Exemple de réponse JSON :

```json
[
    {"text":"konnnichiha","script":"Latn"},
    {"text":"sayounara","script":"Latn"}
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

L’exemple suivant montre comment convertir deux chaînes de japonais en japonais romanisé.

La charge utile JSON de la requête dans cet exemple :

```json
[{"text":"こんにちは","script":"jpan"},{"text":"さようなら","script":"jpan"}]
```

Si vous utilisez cUrl dans une fenêtre de ligne de commande ne prenant pas en charge les caractères Unicode, utilisez la charge utile JSON suivante et enregistrez-la dans un fichier nommé `request.txt`. Veillez à enregistrer le fichier avec l’encodage `UTF-8`.

```
curl -X POST "https://api.cognitive.microsofttranslator.com/transliterate?api-version=3.0&language=ja&fromScript=Jpan&toScript=Latn" -H "X-ClientTraceId: 875030C7-5380-40B8-8A03-63DACCF69C11" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json" -d @request.txt
```