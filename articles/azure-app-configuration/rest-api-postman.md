---
title: API REST Azure Active Directory - Test avec Postman
description: Utiliser Postman pour tester l’API REST Azure App Configuration
author: AlexandraKemperMS
ms.author: alkemper
ms.service: azure-app-configuration
ms.topic: reference
ms.date: 08/17/2020
ms.openlocfilehash: 0b4444b049d181c2f66f8b02a5202dd17a3f4b6b
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96932487"
---
# <a name="test-the-azure-app-configuration-rest-api-using-postman"></a>Tester l’API REST Azure App Configuration à l’aide de Postman

Pour tester l’API REST à l’aide de [Postman](https://www.getpostman.com/), vous devez inclure les en-têtes HTTP requis pour l’[authentification](./rest-api-authentication-hmac.md) dans vos requêtes. Voici la procédure à suivre pour configurer Postman de manière à tester l’API REST en générant automatiquement les en-têtes d’authentification :

1. Créer une [requête](https://learning.getpostman.com/docs/postman/sending_api_requests/requests/)
1. Ajoutez la fonction `signRequest` à partir de l’[exemple d’authentification JavaScript](./rest-api-authentication-hmac.md#javascript) au [script de pré-requête](https://learning.getpostman.com/docs/postman/scripts/pre_request_scripts/) pour la requête
1. Ajoutez le code suivant à la fin du script de pré-requête. Mettre à jour la clé d’accès comme indiqué par le commentaire TODO

    ```js
    // TODO: Replace the following placeholders with your access key
    var credential = "<Credential>"; // Id
    var secret = "<Secret>"; // Value

    var isBodyEmpty = pm.request.body === null || pm.request.body === undefined || pm.request.body.isEmpty();

    var headers = signRequest(
        pm.request.url.getHost(),
        pm.request.method,
        pm.request.url.getPathWithQuery(),
        isBodyEmpty ? undefined : pm.request.body.toString(),
        credential,
        secret);

    // Add headers to the request
    headers.forEach(header => {
        pm.request.headers.upsert({key: header.name, value: header.value});
    })
    ```

1. Envoyer la demande
