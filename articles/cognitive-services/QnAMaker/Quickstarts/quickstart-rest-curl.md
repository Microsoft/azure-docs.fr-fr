---
title: 'Démarrage rapide : Utiliser cURL et REST pour gérer la base de connaissances - QnA Maker'
description: Ce guide de démarrage rapide vous montre comment créer, publier et interroger votre base de connaissances à l’aide des API REST.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.date: 11/09/2020
ROBOTS: NOINDEX,NOFOLLOW
ms.custom: RESTCURL2020FEB27
ms.topic: quickstart
ms.openlocfilehash: 7592b9fb509f39504ad2399d0e939ceca1156221
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96351093"
---
# <a name="quickstart-use-curl-and-rest-to-manage-knowledge-base"></a>Démarrage rapide : Utiliser cURL et REST pour gérer la base de connaissances

Ce guide de démarrage rapide vous guide tout au long de la création, la publication et l’interrogation de votre base de connaissances. QnA Maker extrait automatiquement les questions et les réponses à partir du contenu semi-structuré, telles que les FAQ, de [sources de données](../index.yml). Le modèle de la base de connaissances est défini dans le code JSON envoyé dans le corps de la requête d’API.

[!INCLUDE [Custom subdomains notice](../../../../includes/cognitive-services-custom-subdomains-note.md)]

## <a name="prerequisites"></a>Prérequis

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)

* Version actuelle de [cURL](https://curl.haxx.se/). Plusieurs commutateurs de ligne de commande sont utilisés dans les guides de démarrage rapide, qui sont indiqués dans la [documentation cURL](https://curl.haxx.se/docs/manpage.html).
* Pour utiliser la clé et le nom de la ressource, vous devez disposer d’une [ressource QnA Maker](../How-To/set-up-qnamaker-service-azure.md). Vous avez entré le **Nom** de la ressource lors de la création de la ressource, puis la clé a été créée pour vous. Le nom de la ressource est utilisé en tant que sous-domaine pour votre point de terminaison. Pour récupérer votre clé et le nom de votre ressource, sélectionnez **Démarrage rapide** pour votre ressource dans le portail Azure. Le nom de la ressource constitue le premier sous-domaine de l’URL du point de terminaison :

    `https://YOUR-RESOURCE-NAME.cognitiveservices.azure.com/qnamaker/v4.0`

> [!CAUTION]
> Les exemples BASH suivants utilisent le caractère de continuation de ligne `\`. Si votre console ou terminal utilise un caractère de continuation de ligne différent, utilisez ce caractère.

# <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)

* Version actuelle de [cURL](https://curl.haxx.se/). Plusieurs commutateurs de ligne de commande sont utilisés dans les guides de démarrage rapide, qui sont indiqués dans la [documentation cURL](https://curl.haxx.se/docs/manpage.html).
* Pour utiliser la clé et le nom de la ressource, vous devez disposer d’une [ressource QnA Maker](../How-To/set-up-qnamaker-service-azure.md). Vous avez entré le **Nom** de la ressource lors de la création de la ressource, puis la clé a été créée pour vous. Le nom de la ressource est utilisé en tant que sous-domaine pour votre point de terminaison. Pour récupérer votre clé et le nom de votre ressource, sélectionnez **Démarrage rapide** pour votre ressource dans le portail Azure. Le nom de la ressource constitue le premier sous-domaine de l’URL du point de terminaison :

    `https://YOUR-RESOURCE-NAME.cognitiveservices.azure.com/qnamaker/v5.0-preview.1`

> [!CAUTION]
> Les exemples BASH suivants utilisent le caractère de continuation de ligne `\`. Si votre console ou terminal utilise un caractère de continuation de ligne différent, utilisez ce caractère.

---

## <a name="create-a-knowledge-base"></a>Créer une base de connaissances

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)

Pour créer une base de connaissances avec les API REST et cURL, vous devez disposer des informations suivantes :

|Information|Configuration cURL|Objectif|
|--|--|--|
|Nom de la ressource QnA Maker|URL|Utilisé pour construire l’URL|
|Clé de la ressource QnA Maker|Param `-h` pour l’en-tête `Ocp-Apim-Subscription-Key`|S’authentifier auprès du service QnA Maker|
|JSON décrivant la base de connaissances|Param `-d`|[Exemples](/rest/api/cognitiveservices/qnamaker/knowledgebase/create#examples) de JSON|
|Taille du JSON en octets|Param `-h` pour l’en-tête `Content-Size`||

La commande cURL est exécutée à partir d’un interpréteur de commandes BASH. Modifiez cette commande avec vos propres informations relatives au nom de ressource, à la clé de ressource ainsi qu’aux valeurs JSON et à la taille de JSON.

```bash
curl https://REPLACE-WITH-YOUR-RESOURCE-NAME.cognitiveservices.azure.com/qnamaker/v4.0/knowledgebases/create \
-X POST \
-H "Ocp-Apim-Subscription-Key: REPLACE-WITH-YOUR-RESOURCE-KEY" \
-H "Content-Type:application/json" \
-H "Content-Size:107" \
-d '{ name: "QnA Maker FAQ",urls: [ "https://docs.microsoft.com/en-in/azure/cognitive-services/qnamaker/faqs"]}'
```

La réponse cURL de QnA Maker comprend `operationId`, qui est requis pour [obtenir l’état de l’opération](#get-status-of-operation).

```json
{
  "operationState": "NotStarted",
  "createdTimestamp": "2020-02-27T04:11:22Z",
  "lastActionTimestamp": "2020-02-27T04:11:22Z",
  "userId": "9596077b3e0441eb93d5080d6a15c64b",
  "operationId": "95a4f700-9899-4c98-bda8-5449af9faef8"
}
```

# <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)


Pour créer une base de connaissances avec les API REST et cURL, vous devez disposer des informations suivantes :

|Information|Configuration cURL|Objectif|
|--|--|--|
|Nom de la ressource QnA Maker|URL|Utilisé pour construire l’URL|
|Clé de la ressource QnA Maker|Param `-h` pour l’en-tête `Ocp-Apim-Subscription-Key`|S’authentifier auprès du service QnA Maker|
|JSON décrivant la base de connaissances|Param `-d`|[Exemples](/rest/api/cognitiveservices/qnamaker/knowledgebase/create#examples) de JSON|
|Taille du JSON en octets|Param `-h` pour l’en-tête `Content-Size`||

La commande cURL est exécutée à partir d’un interpréteur de commandes BASH. Modifiez cette commande avec vos propres informations relatives au nom de ressource, à la clé de ressource ainsi qu’aux valeurs JSON et à la taille de JSON.

```bash
curl https://REPLACE-WITH-YOUR-RESOURCE-NAME.cognitiveservices.azure.com/qnamaker/v5.0-preview.1/knowledgebases/create \
-X POST \
-H "Ocp-Apim-Subscription-Key: REPLACE-WITH-YOUR-RESOURCE-KEY" \
-H "Content-Type:application/json" \
-H "Content-Size:107" \
-d '{ name: "QnA Maker FAQ",urls: [ "https://docs.microsoft.com/en-in/azure/cognitive-services/qnamaker/faqs"]}'
```

La réponse cURL de QnA Maker comprend `operationId`, qui est requis pour [obtenir l’état de l’opération](#get-status-of-operation).

```json
{
  "operationState": "NotStarted",
  "createdTimestamp": "2020-02-27T04:11:22Z",
  "lastActionTimestamp": "2020-02-27T04:11:22Z",
  "userId": "9596077b3e0441eb93d5080d6a15c64b",
  "operationId": "95a4f700-9899-4c98-bda8-5449af9faef8"
}
```
---

## <a name="get-status-of-operation"></a>Obtenir l’état de l’opération

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)

Quand vous créez une base de connaissances, comme l’opération est asynchrone, la réponse contient des informations pour déterminer l’état.

|Information|Configuration cURL|Objectif|
|--|--|--|
|Nom de la ressource QnA Maker|URL|Utilisé pour construire l’URL|
|ID d’opération|Itinéraire d’URL|`/operations/REPLACE-WITH-YOUR-OPERATION-ID`|
|Clé de la ressource QnA Maker|Param `-h` pour l’en-tête `Ocp-Apim-Subscription-Key`|S’authentifier auprès du service QnA Maker|

La commande cURL est exécutée à partir d’un interpréteur de commandes BASH. Modifiez cette commande avec vos propres informations relatives au nom de ressource, à la clé de ressource et à l’ID d’opération.

```bash
curl https://REPLACE-WITH-YOUR-RESOURCE-NAME.cognitiveservices.azure.com/qnamaker/v4.0/operations/REPLACE-WITH-YOUR-OPERATION-ID \
-X GET \
-H "Ocp-Apim-Subscription-Key: REPLACE-WITH-YOUR-RESOURCE-KEY"
```

La réponse cURL comprend l’état. Si l’état de l’opération est Réussi, `resourceLocation` contient l’ID de la base de connaissances.

```json
{
   "operationState": "Succeeded",
   "createdTimestamp": "2020-02-27T04:54:07Z",
   "lastActionTimestamp": "2020-02-27T04:54:19Z",
   "resourceLocation": "/knowledgebases/fe3971b7-cfaa-41fa-8d9f-6ceb673eb865",
   "userId": "f596077b3e0441eb93d5080d6a15c64b",
   "operationId": "f293f218-d080-48f0-a766-47993e9b26a8"
}
```
# <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)


Quand vous créez une base de connaissances, comme l’opération est asynchrone, la réponse contient des informations pour déterminer l’état.

|Information|Configuration cURL|Objectif|
|--|--|--|
|Nom de la ressource QnA Maker|URL|Utilisé pour construire l’URL|
|ID d’opération|Itinéraire d’URL|`/operations/REPLACE-WITH-YOUR-OPERATION-ID`|
|Clé de la ressource QnA Maker|Param `-h` pour l’en-tête `Ocp-Apim-Subscription-Key`|S’authentifier auprès du service QnA Maker|

La commande cURL est exécutée à partir d’un interpréteur de commandes BASH. Modifiez cette commande avec vos propres informations relatives au nom de ressource, à la clé de ressource et à l’ID d’opération.

```bash
curl https://REPLACE-WITH-YOUR-RESOURCE-NAME.cognitiveservices.azure.com/qnamaker/v5.0-preview.1/operations/REPLACE-WITH-YOUR-OPERATION-ID \
-X GET \
-H "Ocp-Apim-Subscription-Key: REPLACE-WITH-YOUR-RESOURCE-KEY"
```

La réponse cURL comprend l’état. Si l’état de l’opération est Réussi, `resourceLocation` contient l’ID de la base de connaissances.

```json
{
   "operationState": "Succeeded",
   "createdTimestamp": "2020-02-27T04:54:07Z",
   "lastActionTimestamp": "2020-02-27T04:54:19Z",
   "resourceLocation": "/knowledgebases/fe3971b7-cfaa-41fa-8d9f-6ceb673eb865",
   "userId": "f596077b3e0441eb93d5080d6a15c64b",
   "operationId": "f293f218-d080-48f0-a766-47993e9b26a8"
}
```

---

## <a name="publish-knowledge-base"></a>Publier une base de connaissances

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)

Avant d’interroger la base de connaissances, vous devez :
* Publier une base de connaissances
* Obtenir la clé de point de terminaison du runtime

Cette tâche permet de publier la base de connaissances. L’obtention de la clé de point de terminaison du runtime est une [tâche distincte](#get-published-knowledge-bases-runtime-endpoint-key).

|Information|Configuration cURL|Objectif|
|--|--|--|
|Nom de la ressource QnA Maker|URL|Utilisé pour construire l’URL|
|Clé de la ressource QnA Maker|Param `-h` pour l’en-tête `Ocp-Apim-Subscription-Key`|S’authentifier auprès du service QnA Maker|
|ID de la base de connaissances|Itinéraire d’URL|`/knowledgebases/REPLACE-WITH-YOUR-KNOWLEDGE-BASE-ID`|

La commande cURL est exécutée à partir d’un interpréteur de commandes BASH. Modifiez cette commande avec vos propres informations relatives au nom de ressource, à la clé de ressource et à l’ID de la base de connaissances.

```bash
curl https://REPLACE-WITH-YOUR-RESOURCE-NAME.cognitiveservices.azure.com/qnamaker/v4.0/knowledgebases/REPLACE-WITH-YOUR-KNOWLEDGE-BASE-ID \
-v \
-X POST \
-H "Ocp-Apim-Subscription-Key: REPLACE-WITH-YOUR-RESOURCE-KEY" \
--data-raw ''
```

L’état de la réponse est 204 sans résultats. Utilisez le paramètre de ligne de commande `-v` pour afficher la sortie détaillée de la commande cURL. Cela inclut l’état HTTP.

# <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)

Avant d’interroger la base de connaissances, vous devez :
* Publier une base de connaissances
* Obtenir la clé de point de terminaison du runtime

Cette tâche permet de publier la base de connaissances. L’obtention de la clé de point de terminaison du runtime est une [tâche distincte](#get-published-knowledge-bases-runtime-endpoint-key).

|Information|Configuration cURL|Objectif|
|--|--|--|
|Nom de la ressource QnA Maker|URL|Utilisé pour construire l’URL|
|Clé de la ressource QnA Maker|Param `-h` pour l’en-tête `Ocp-Apim-Subscription-Key`|S’authentifier auprès du service QnA Maker|
|ID de la base de connaissances|Itinéraire d’URL|`/knowledgebases/REPLACE-WITH-YOUR-KNOWLEDGE-BASE-ID`|

La commande cURL est exécutée à partir d’un interpréteur de commandes BASH. Modifiez cette commande avec vos propres informations relatives au nom de ressource, à la clé de ressource et à l’ID de la base de connaissances.

```bash
curl https://REPLACE-WITH-YOUR-RESOURCE-NAME.cognitiveservices.azure.com/qnamaker/v5.0-preview.1/knowledgebases/REPLACE-WITH-YOUR-KNOWLEDGE-BASE-ID \
-v \
-X POST \
-H "Ocp-Apim-Subscription-Key: REPLACE-WITH-YOUR-RESOURCE-KEY" \
--data-raw ''
```

L’état de la réponse est 204 sans résultats. Utilisez le paramètre de ligne de commande `-v` pour afficher la sortie détaillée de la commande cURL. Cela inclut l’état HTTP.

---

## <a name="get-published-knowledge-bases-runtime-endpoint-key"></a>Obtenir la clé de point de terminaison du runtime de la base de connaissances publiée

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)

Avant d’interroger la base de connaissances, vous devez :
* Publier une base de connaissances
* Obtenir la clé de point de terminaison du runtime

Cette tâche permet d’obtenir la clé de point de terminaison du runtime. La publication de la base de connaissances est une [tâche distincte](#publish-knowledge-base).

La clé de point de terminaison du runtime est la même clé pour toutes les bases de connaissances utilisant la ressource QnA Maker.

|Information|Configuration cURL|Objectif|
|--|--|--|
|Nom de la ressource QnA Maker|URL|Utilisé pour construire l’URL|
|Clé de la ressource QnA Maker|Param `-h` pour l’en-tête `Ocp-Apim-Subscription-Key`|S’authentifier auprès du service QnA Maker|

La commande cURL est exécutée à partir d’un interpréteur de commandes BASH. Modifiez cette commande avec vos propres informations relatives au nom de ressource et à la clé de ressource.

```bash
curl https://REPLACE-WITH-YOUR-RESOURCE-NAME.cognitiveservices.azure.com/qnamaker/v4.0/endpointkeys \
-X GET \
-H "Ocp-Apim-Subscription-Key: REPLACE-WITH-YOUR-RESOURCE-KEY"
```


La réponse cURL comprend les clés de point de terminaison du runtime. Utilisez une seule des clés lors de l’interrogation pour obtenir une réponse de la base de connaissances.

```json
{
  "primaryEndpointKey": "93e88a14-694a-44d5-883b-184a68aa8530",
  "secondaryEndpointKey": "92c98c16-ca31-4294-8626-6c57454a5063",
  "installedVersion": "4.0.5",
  "lastStableVersion": "4.0.6"
}
```
# <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)

Avant d’interroger la base de connaissances, vous devez :
* Publier une base de connaissances
* Obtenir la clé de point de terminaison du runtime

Cette tâche permet d’obtenir la clé de point de terminaison du runtime. La publication de la base de connaissances est une [tâche distincte](#publish-knowledge-base).

La clé de point de terminaison du runtime est la même clé pour toutes les bases de connaissances utilisant la ressource QnA Maker.

|Information|Configuration cURL|Objectif|
|--|--|--|
|Nom de la ressource QnA Maker|URL|Utilisé pour construire l’URL|
|Clé de la ressource QnA Maker|Param `-h` pour l’en-tête `Ocp-Apim-Subscription-Key`|S’authentifier auprès du service QnA Maker|

La commande cURL est exécutée à partir d’un interpréteur de commandes BASH. Modifiez cette commande avec vos propres informations relatives au nom de ressource et à la clé de ressource.

```bash
curl https://REPLACE-WITH-YOUR-RESOURCE-NAME.cognitiveservices.azure.com/qnamaker/v5.0-preview.1/endpointkeys \
-X GET \
-H "Ocp-Apim-Subscription-Key: REPLACE-WITH-YOUR-RESOURCE-KEY"
```


La réponse cURL comprend les clés de point de terminaison du runtime. Utilisez une seule des clés lors de l’interrogation pour obtenir une réponse de la base de connaissances.

```json
{
  "primaryEndpointKey": "93e88a14-694a-44d5-883b-184a68aa8530",
  "secondaryEndpointKey": "92c98c16-ca31-4294-8626-6c57454a5063",
  "installedVersion": "4.0.5",
  "lastStableVersion": "4.0.6"
}
```

---

## <a name="query-for-answer-from-published-knowledge-base"></a>Requête relative à une réponse de la base de connaissances publiée

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)

L’obtention d’une réponse de la base de connaissances se déroule à partir d’un runtime distinct de la gestion de la base de connaissances. Étant donné qu’il s’agit d’un runtime distinct, vous devez vous authentifier à l’aide d’une clé de runtime.

|Information|Configuration cURL|Objectif|
|--|--|--|
|Nom de la ressource QnA Maker|URL|Utilisé pour construire l’URL|
|Clé de runtime QnA Maker|Param `-h` pour l’en-tête `Authorization`|La clé fait partie d’une chaîne qui comprend le mot `Endpointkey `. S’authentifier auprès du service QnA Maker|
|ID de la base de connaissances|Itinéraire d’URL|`/knowledgebases/REPLACE-WITH-YOUR-KNOWLEDGE-BASE-ID`|
|JSON décrivant la requête|Param `-d`|[Paramètres du corps de la demande](/rest/api/cognitiveservices/qnamakerruntime/runtime/generateanswer#request-body) et [exemples](/rest/api/cognitiveservices/qnamakerruntime/runtime/generateanswer#examples) de JSON|
|Taille du JSON en octets|Param `-h` pour l’en-tête `Content-Size`||

La commande cURL est exécutée à partir d’un interpréteur de commandes BASH. Modifiez cette commande avec vos propres informations relatives au nom de ressource, à la clé de ressource et à l’ID de la base de connaissances.

```bash
curl https://REPLACE-WITH-YOUR-RESOURCE-NAME.azurewebsites.net/qnamaker/knowledgebases/REPLACE-WITH-YOUR-KNOWLEDGE-BASE-ID/generateAnswer \
-X POST \
-H "Authorization: EndpointKey REPLACE-WITH-YOUR-RUNTIME-KEY" \
-H "Content-Type:application/json" \
-H "Content-Size:159" \
-d '{"question": "How are QnA Maker and LUIS used together?","top": 6,"isTest": true,  "scoreThreshold": 20, "strictFilters": [], "userId": "sd53lsY="}'
```

Une réponse correcte comprend la meilleure réponse avec d’autres informations dont une application cliente, telle qu’un chatbot, a besoin afin d’afficher une réponse pour l’utilisateur.

# <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)

L’obtention d’une réponse de la base de connaissances se déroule à partir d’un runtime distinct de la gestion de la base de connaissances. Étant donné qu’il s’agit d’un runtime distinct, vous devez vous authentifier à l’aide d’une clé de runtime.

|Information|Configuration cURL|Objectif|
|--|--|--|
|Nom de la ressource QnA Maker|URL|Utilisé pour construire l’URL|
|Clé de la ressource QnA Maker|Param `-h` pour l’en-tête `Ocp-Apim-Subscription-Key`|S’authentifier auprès du service QnA Maker|
|ID de la base de connaissances|Itinéraire d’URL|`/knowledgebases/REPLACE-WITH-YOUR-KNOWLEDGE-BASE-ID`|
|JSON décrivant la requête|Param `-d`|[Paramètres du corps de la demande](/rest/api/cognitiveservices/qnamakerruntime/runtime/generateanswer#request-body) et [exemples](/rest/api/cognitiveservices/qnamakerruntime/runtime/generateanswer#examples) de JSON|
|Taille du JSON en octets|Param `-h` pour l’en-tête `Content-Size`||

La commande cURL est exécutée à partir d’un interpréteur de commandes BASH. Modifiez cette commande avec vos propres informations relatives au nom de ressource, à la clé de ressource et à l’ID de la base de connaissances.

```bash
curl https://REPLACE-WITH-YOUR-RESOURCE-NAME.cognitiveservices.azure.com/qnamaker/v5.0-preview.1/knowledgebases/REPLACE-WITH-YOUR-KNOWLEDGE-BASE-ID/generateAnswer \
-X POST \
-H "Content-Type:application/json" \
-H "Content-Size:159" \
-H "Ocp-Apim-Subscription-Key: REPLACE-WITH-YOUR-RESOURCE-KEY"
-d '{"question": "How are QnA Maker and LUIS used together?","top": 6,"isTest": true,  "scoreThreshold": 20, "strictFilters": [], "userId": "sd53lsY="}'
```

Une réponse correcte comprend la meilleure réponse avec d’autres informations dont une application cliente, telle qu’un chatbot, a besoin afin d’afficher une réponse pour l’utilisateur.


---

## <a name="delete-knowledge-base"></a>Supprimer une base de connaissances

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)

Quand vous avez fini d’utiliser la base de connaissances, supprimez-la.

|Information|Configuration cURL|Objectif|
|--|--|--|
|Nom de la ressource QnA Maker|URL|Utilisé pour construire l’URL|
|Clé de la ressource QnA Maker|Param `-h` pour l’en-tête `Ocp-Apim-Subscription-Key`|S’authentifier auprès du service QnA Maker|
|ID de la base de connaissances|Itinéraire d’URL|`/knowledgebases/REPLACE-WITH-YOUR-KNOWLEDGE-BASE-ID`|

La commande cURL est exécutée à partir d’un interpréteur de commandes BASH. Modifiez cette commande avec vos propres informations relatives au nom de ressource, à la clé de ressource et à l’ID de la base de connaissances.

```bash
curl https://REPLACE-WITH-YOUR-RESOURCE-NAME.cognitiveservices.azure.com/qnamaker/v4.0/knowledgebases/REPLACE-WITH-YOUR-KNOWLEDGE-BASE-ID \
-X DELETE \
-v \
-H "Ocp-Apim-Subscription-Key: REPLACE-WITH-YOUR-RESOURCE-KEY"
```

L’état de la réponse est 204 sans résultats. Utilisez le paramètre de ligne de commande `-v` pour afficher la sortie détaillée de la commande cURL. Cela inclut l’état HTTP.

# <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)

Quand vous avez fini d’utiliser la base de connaissances, supprimez-la.

|Information|Configuration cURL|Objectif|
|--|--|--|
|Nom de la ressource QnA Maker|URL|Utilisé pour construire l’URL|
|Clé de la ressource QnA Maker|Param `-h` pour l’en-tête `Ocp-Apim-Subscription-Key`|S’authentifier auprès du service QnA Maker|
|ID de la base de connaissances|Itinéraire d’URL|`/knowledgebases/REPLACE-WITH-YOUR-KNOWLEDGE-BASE-ID`|

La commande cURL est exécutée à partir d’un interpréteur de commandes BASH. Modifiez cette commande avec vos propres informations relatives au nom de ressource, à la clé de ressource et à l’ID de la base de connaissances.

```bash
curl https://REPLACE-WITH-YOUR-RESOURCE-NAME.cognitiveservices.azure.com/qnamaker/v5.0-preview.1/knowledgebases/REPLACE-WITH-YOUR-KNOWLEDGE-BASE-ID \
-X DELETE \
-v \
-H "Ocp-Apim-Subscription-Key: REPLACE-WITH-YOUR-RESOURCE-KEY"
```

L’état de la réponse est 204 sans résultats. Utilisez le paramètre de ligne de commande `-v` pour afficher la sortie détaillée de la commande cURL. Cela inclut l’état HTTP.

---

## <a name="additional-resources"></a>Ressources supplémentaires

* Documentation de référence sur la [création](/rest/api/cognitiveservices/qnamaker4.0/knowledgebase)
* Documentation de référence sur le [runtime](/rest/api/cognitiveservices/qnamaker4.0/runtime)
* [Exemples de scripts BASH utilisant cURL](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/curl/QnAMaker)

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Documentation de référence pour l’API REST QnA Maker (V4)](/rest/api/cognitiveservices/qnamaker4.0/knowledgebase)