---
author: erhopf
ms.service: cognitive-services
ms.topic: include
ms.date: 08/06/2019
ms.author: erhopf
ms.custom: devx-track-javascript
ms.openlocfilehash: dfca8bb6f897f532081df7d141112ddbb190d109
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/29/2020
ms.locfileid: "87406540"
---
[!INCLUDE [Prerequisites](prerequisites-nodejs.md)]

[!INCLUDE [Setup and use environment variables](setup-env-variables.md)]

## <a name="create-a-project-and-import-required-modules"></a>Créez un projet et importez les modules requis

Créez un nouveau projet dans votre IDE ou votre éditeur favori. Copiez cet extrait de code dans votre projet, dans un fichier nommé `detect.js`.

```javascript
const request = require('request');
const uuidv4 = require('uuid/v4');
```

> [!NOTE]
> Si vous n’avez jamais utilisé ces modules auparavant, vous devrez les installer avant d’exécuter votre programme. Pour installer ces packages, exécuter `npm install request uuidv4`.

Ces modules sont requis pour construire la requête HTTP et créer un identificateur unique pour l’en-tête `'X-ClientTraceId'`.

## <a name="set-the-subscription-key-and-endpoint"></a>Définir la clé d’abonnement et le point de terminaison

Cet exemple tente de lire la clé d’abonnement et le point de terminaison Translator depuis ces variables d’environnement : `TRANSLATOR_TEXT_SUBSCRIPTION_KEY` et `TRANSLATOR_TEXT_ENDPOINT`. Si vous n’êtes pas familiarisé avec les variables d’environnement, vous pouvez définir `subscriptionKey` et `endpoint` en tant que chaînes et commenter les instructions conditionnelles.

Copiez ce code dans votre projet :

```javascript
var key_var = 'TRANSLATOR_TEXT_SUBSCRIPTION_KEY';
if (!process.env[key_var]) {
    throw new Error('Please set/export the following environment variable: ' + key_var);
}
var subscriptionKey = process.env[key_var];
var endpoint_var = 'TRANSLATOR_TEXT_ENDPOINT';
if (!process.env[endpoint_var]) {
    throw new Error('Please set/export the following environment variable: ' + endpoint_var);
}
var endpoint = process.env[endpoint_var];
```

## <a name="configure-the-request"></a>Configurer la requête

La méthode `request()`, accessible via le module de requête, permet de transférer la méthode HTTP, l’URL, les paramètres de requête, les en-têtes et le code JSON en tant qu’objet `options`. Dans cet extrait de code, nous allons configurer la requête :

```javascript
let options = {
    method: 'POST',
    baseUrl: endpoint,
    url: 'detect',
    qs: {
      'api-version': '3.0',
    },
    headers: {
      'Ocp-Apim-Subscription-Key': subscriptionKey,
      'Content-type': 'application/json',
      'X-ClientTraceId': uuidv4().toString()
    },
    body: [{
          'text': 'Salve, mondo!'
    }],
    json: true,
};
```
La manière la plus simple d’authentifier une requête consiste à transmettre votre clé d’abonnement comme `Ocp-Apim-Subscription-Key`en-tête. C’est ce que nous avons fait dans ce modèle. Vous pouvez aussi échanger votre clé d’abonnement pour un jeton d’accès et transmettre le jeton d’accès en tant qu’`Authorization`en-tête pour valider votre requête.

Si vous utilisez un abonnement multiservice Cognitive Services, vous devez également inclure la `Ocp-Apim-Subscription-Region` dans vos en-têtes de requête.

Pour en savoir plus, consultez [Authentification](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-reference#authentication).

## <a name="make-the-request-and-print-the-response"></a>Effectuer la requête et imprimer la réponse

Ensuite, nous allons créer la requête à l’aide de la méthode `request()`. Cette méthode utilise l’objet `options` que nous avons créé dans la section précédente en tant premier argument, puis imprime la réponse JSON agrémentée.

```javascript
request(options, function(err, res, body){
    console.log(JSON.stringify(body, null, 4));
});
```

>[!NOTE]
> Dans cet exemple, nous définissons la requête HTTP dans l’objet `options`. Toutefois, le module de requête prend également en charge des méthodes pratiques, comme `.post` et `.get`. Pour plus d’informations, consultez la section consacrée aux [méthodes pratiques](https://github.com/request/request#convenience-methods).

## <a name="put-it-all-together"></a>Assemblage

Voilà, vous avez installé un programme simple qui appellera Translator et enverra une réponse JSON. Il est maintenant temps d’exécuter votre programme :

```console
node detect.js
```

## <a name="sample-response"></a>Exemple de réponse

Après avoir exécuté l’exemple, vous devriez voir ce qui suit sur le terminal :

> [!NOTE]
> Recherchez l’abréviation du pays/de la région dans cette [liste de langues](https://docs.microsoft.com/azure/cognitive-services/translator/language-support).

```json
[
    {
        "alternatives": [
            {
                "isTranslationSupported": true,
                "isTransliterationSupported": false,
                "language": "pt",
                "score": 1.0
            },
            {
                "isTranslationSupported": true,
                "isTransliterationSupported": false,
                "language": "en",
                "score": 1.0
            }
        ],
        "isTranslationSupported": true,
        "isTransliterationSupported": false,
        "language": "it",
        "score": 1.0
    }
]
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous avez codé en dur votre clé d’abonnement dans votre programme, veillez à supprimer la clé d’abonnement une fois ce démarrage rapide terminé.

## <a name="next-steps"></a>Étapes suivantes

Consultez les informations de référence sur l’API pour comprendre tout ce que vous pouvez faire avec Translator.

> [!div class="nextstepaction"]
> [Informations de référence sur l'API](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-reference)
