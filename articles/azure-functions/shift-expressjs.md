---
title: Passage d’Express.js à Azure Functions
description: Apprenez à refactoriser les points de terminaison Express.js pour Azure Functions.
author: craigshoemaker
ms.topic: conceptual
ms.date: 07/31/2020
ms.author: cshoe
ms.openlocfilehash: 266df5371ff5f47526fa9d6567c62e31d51ebb05
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87810222"
---
# <a name="shifting-from-expressjs-to-azure-functions"></a>Passage d’Express.js à Azure Functions

Express.js est l’une des infrastructures node. js les plus populaires pour les développeurs Web et demeure un excellent choix pour créer des applications qui servent des points de terminaison d’API.

Lors de la migration du code vers une architecture sans serveur, la refactorisation des points de terminaison Express.js affecte les domaines suivants :

- **Middleware** : Express.js offre une collection robuste d’intergiciels. De nombreux modules d’intergiciel ne sont plus requis à la lumière des Azure Functions et les fonctionnalités de [Gestion des API Azure](../api-management/api-management-key-concepts.md). Assurez-vous que vous pouvez répliquer ou remplacer toute logique gérée par l’intergiciel essentiel avant de migrer les points de terminaison.

- **API différentes** : L’API utilisée pour traiter les demandes et les réponses diffère entre Azure Functions et Express.js. L’exemple suivant détaille les modifications requises.

- **Route par défaut** : Par défaut, les points de terminaison Azure Functions sont exposés sous l’itinéraire `api`. Les règles de routage sont configurables par le biais de [`routePrefix` dans le fichier _host.json_ ](./functions-bindings-http-webhook-output.md#hostjson-settings).

- **Configuration et conventions** : Une application de fonctions utilise le fichier _function.json_ pour définir des verbes HTTP, définir des stratégies de sécurité et configurer les [d’entrée et de sortie](./functions-triggers-bindings.md) de la fonction. Par défaut, le nom du dossier qui contient les fichiers de fonctions définit le nom du point de terminaison, mais vous pouvez modifier le nom via la propriété `route` dans le fichier [function.json](./functions-bindings-http-webhook-trigger.md#customize-the-http-endpoint).

> [!TIP]
> Pour plus d’informations, consultez le tutoriel interactif [Refactoriser des API node.js et Express vers des API sans serveur avec Azure Functions](/learn/modules/shift-nodejs-express-apis-serverless/).

## <a name="example"></a>Exemple

### <a name="expressjs"></a>Express.js

L’exemple ci-dessous montre un point de terminaison Express.js `GET` classique.

```javascript
// server.js
app.get('/hello', (req, res) => {
  try {
    res.send("Success!");
  } catch(error) {
    const err = JSON.stringify(error);
    res.status(500).send(`Request error. ${err}`);
  }
});
```

Quand une demande de `GET` est envoyée à `/hello`, une réponse `HTTP 200` contenant `Success` est retournée. Si le point de terminaison rencontre une erreur, la réponse est une `HTTP 500` avec les détails de l’erreur.

### <a name="azure-functions"></a>Azure Functions

Azure Functions organise les fichiers de configuration et de code dans un dossier unique pour chaque fonction. Par défaut, le nom du dossier dicte le nom de la fonction.

Par exemple, une fonction nommée `hello` a un dossier avec les fichiers suivants.

``` files
| - hello
|  - function.json
|  - index.js
```

L’exemple suivant implémente le même résultat que le point de terminaison Express.js ci-dessus, mais avec Azure Functions.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
// hello/index.js
module.exports = async function (context, req) {
  try {
    context.res = { body: "Success!" };
  } catch(error) {
    const err = JSON.stringify(error);
    context.res = {
      status: 500,
      body: `Request error. ${err}`
    };
  }
};
```

# <a name="typescript"></a>[TypeScript](#tab/typescript)

```typescript
// hello/index.ts
import { AzureFunction, Context, HttpRequest } from "@azure/functions";

const httpTrigger: AzureFunction = async function (context: Context, req: HttpRequest): Promise<void> {
  try {
    context.res = { body: "Success!" };
  } catch (error) {
    const err = JSON.stringify(error);
    context.res = {
      status: 500,
      body: `Request error. ${err}`,
    };
  }
};

export default httpTrigger;
```

---

Lors du déplacement vers Functions, les modifications suivantes sont apportées :

- **Module :** Le code de fonction est implémenté en tant que module JavaScript.

- **Contexte et objet de réponse** : Le [`context`](./functions-reference-node.md#context-object) vous permet de communiquer avec le runtime Function. À partir du contexte, vous pouvez lire les données de la demande et définir la réponse de la fonction. Le code synchrone vous oblige à appeler `context.done()` pour terminer l’exécution, tandis que les fonctions `asyc` résolvent la demande de manière implicite.

- **Conventions d’affectation de noms** : Le nom de dossier utilisé pour contenir les fichiers Azure Functions est utilisé comme nom de point de terminaison par défaut (cela peut être substitué dans le [function.json](./functions-bindings-http-webhook-trigger.md#customize-the-http-endpoint)).

- **Configuration** : Vous définissez les verbes HTTP dans le fichier [function.json](./functions-bindings-http-webhook-trigger.md#customize-the-http-endpoint) tel que `POST` ou `PUT`.

Le fichier _function. json_ suivant contient les informations de configuration de la fonction.

```json
{
  "bindings": [
    {
      "authLevel": "function",
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": ["get"]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "res"
    }
  ]
}
```

En définissant `get` dans le tableau de `methods`, la fonction est disponible pour les demandes HTTP `GET`. Si vous souhaitez que votre API accepte les demandes de support `POST`, vous pouvez également ajouter des `post` au tableau.

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations, consultez le tutoriel interactif [Refactoriser des API node.js et Express vers des API sans serveur avec Azure Functions](/learn/modules/shift-nodejs-express-apis-serverless/).