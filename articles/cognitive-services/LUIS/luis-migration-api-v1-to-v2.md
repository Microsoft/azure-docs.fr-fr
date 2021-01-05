---
title: Migration de l’API v1 vers v2
titleSuffix: Azure Cognitive Services
description: Les API de point de terminaison et de création Language Understanding de version 1 sont dépréciées. Utilisez ce guide pour comprendre comment migrer vers la version 2 des API de point de terminaison et de création.
services: cognitive-services
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: how-to
ms.date: 04/02/2019
ms.openlocfilehash: 867ae2cc7567077786bb0840cd11c47b786be423
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95018750"
---
# <a name="api-v1-to-v2-migration-guide-for-luis-apps"></a>Guide de migration d’API v1 vers v2 pour les applications LUIS
Les API de [point de terminaison](https://aka.ms/v1-endpoint-api-docs) et de [création](https://aka.ms/v1-authoring-api-docs) de version 1 sont dépréciées. Utilisez ce guide pour comprendre comment migrer vers la version 2 des API de [point de terminaison](https://go.microsoft.com/fwlink/?linkid=2092356) et de [création](https://go.microsoft.com/fwlink/?linkid=2092087).

## <a name="new-azure-regions"></a>Nouvelles régions Azure
LUIS fournit de nouvelles [régions](./luis-reference-regions.md) pour les API LUIS. LUIS fournit un autre portail Web pour les groupes de régions. L’application doit être créée dans la région que vous vous attendez à interroger. Les applications ne migrent pas automatiquement vers d’autres régions. Vous exportez l’application à partir d’une région, puis vous l’importez dans une autre pour qu’elle soit disponible dans une nouvelle région.

## <a name="authoring-route-changes"></a>Création de modifications d’itinéraires
L’itinéraire de création d’API passe de l’utilisation de l’itinéraire **prog** à l’utilisation de l’itinéraire **api**.


| version | itinéraire |
|--|--|
|1|/luis/v1.0/**prog**/apps|
|2|/luis/**api**/v2.0/apps|


## <a name="endpoint-route-changes"></a>Modifications de l’itinéraire de point de terminaison
L’API du point de terminaison a de nouveaux paramètres de chaîne de requête, ainsi qu’une autre réponse. Si l’indicateur verbose est true, toutes les intentions, quel que soit le score, sont retournées dans un tableau nommé Intentions, outre le topScoringIntent.

| version | OBTENIR un itinéraire |
|--|--|
|1|/luis/v1/application?ID={appId}&q={q}|
|2|/luis/v2.0/apps/{appId}?q={q}[&timezoneOffset][&verbose][&spellCheck][&staging][&bing-spell-check-subscription-key][&log]|


Réponse de réussite du point de terminaison v1 :
```json
{
  "odata.metadata":"https://dialogice.cloudapp.net/odata/$metadata#domain","value":[
    {
      "id":"bccb84ee-4bd6-4460-a340-0595b12db294","q":"turn on the camera","response":"[{\"intent\":\"OpenCamera\",\"score\":0.976928055},{\"intent\":\"None\",\"score\":0.0230718572}]"
    }
  ]
}
```

Réponse de réussite du point de terminaison v2 :
```json
{
  "query": "forward to frank 30 dollars through HSBC",
  "topScoringIntent": {
    "intent": "give",
    "score": 0.3964121
  },
  "entities": [
    {
      "entity": "30",
      "type": "builtin.number",
      "startIndex": 17,
      "endIndex": 18,
      "resolution": {
        "value": "30"
      }
    },
    {
      "entity": "frank",
      "type": "frank",
      "startIndex": 11,
      "endIndex": 15,
      "score": 0.935219169
    },
    {
      "entity": "30 dollars",
      "type": "builtin.currency",
      "startIndex": 17,
      "endIndex": 26,
      "resolution": {
        "unit": "Dollar",
        "value": "30"
      }
    },
    {
      "entity": "hsbc",
      "type": "Bank",
      "startIndex": 36,
      "endIndex": 39,
      "resolution": {
        "values": [
          "BankeName"
        ]
      }
    }
  ]
}
```

## <a name="key-management-no-longer-in-api"></a>La gestion de clés n’est plus dans l’API
Les API de la clé du point de terminaison de l’abonnement sont déconseillées et retournent 410 GONE.

| version | itinéraire |
|--|--|
|1|/luis/v1.0/prog/subscriptions|
|1|/luis/v1.0/prog/subscriptions/{subscriptionKey}|

Les [clés de point de terminaison](luis-how-to-azure-subscription.md) Azure sont générées sur le Portail Azure. Vous affectez la clé à une application LUIS à la page **[Publier](luis-how-to-azure-subscription.md)** . Vous n’avez pas besoin de connaître la valeur de clé réelle. LUIS utilise le nom de l’abonnement pour effectuer l’affectation.

## <a name="new-versioning-route"></a>Nouvel itinéraire de contrôle de version
Le modèle v2 est maintenant contenu dans une [version](luis-how-to-manage-versions.md). Un nom de version a 10 caractères dans l’itinéraire. La version par défaut est « 0.1 ».

| version | itinéraire |
|--|--|
|1|/luis/v1.0/**prog**/apps/{appId}/entities|
|2|/luis/**api**/v2.0/apps/{appId}/**versions**/{versionId}/entities|

## <a name="metadata-renamed"></a>Métadonnées renommées
Plusieurs API qui retournent des métadonnées de LUIS ont été renommées.

| Nom de l’itinéraire v1 | Nom de l’itinéraire v2 |
|--|--|
|PersonalAssistantApps |assistants|
|applicationcultures|cultures|
|applicationdomains|domaines|
|applicationusagescenarios|usagescenarios|


## <a name="sample-renamed-to-suggest"></a>« Exemple » est renommé « Suggérer »
LUIS suggère des énoncés à partir d’[énoncés de point de terminaison](luis-how-to-review-endpoint-utterances.md) qui peuvent améliorer le modèle. Dans la version précédente, cela s’appelait **exemple**. Dans la nouvelle version, le nom est modifié, passant d’exemple à **suggérer**. Cela s’appelle **[Réviser les énoncés de point de terminaison](luis-how-to-review-endpoint-utterances.md)** sur le site web LUIS.

| version | itinéraire |
|--|--|
|1|/luis/v1.0/**prog**/apps/{appId}/entities/{entityId}/**sample**|
|1|/luis/v1.0/**prog**/apps/{appId}/intents/{intentId}/**sample**|
|2|/luis/**api**/v2.0/apps/{appId}/**versions**/{versionId}/entities/{entityId}/**suggest**|
|2|/luis/**api**/v2.0/apps/{appId}/**versions**/{versionId}/intents/{intentId}/**suggest**|


## <a name="create-app-from-prebuilt-domains"></a>Créer une application à partir d’un domaine prédéfini
Les [Domaines prédéfinis](./howto-add-prebuilt-models.md) fournissent un modèle de domaine prédéfini. Les domaines prédéfinis permettent de développer rapidement votre application LUIS pour les domaines courants. Cette API permet de créer une application basée sur un domaine prédéfini. La réponse est le nouvel appID.

|Itinéraire v2|verbe|
|--|--|
|/luis/api/v2.0/apps/customprebuiltdomains  |Get, post|
|/luis/api/v2.0/apps/customprebuiltdomains/{culture}  |get|

## <a name="importing-1x-app-into-2x"></a>Importation d’une application 1.x dans 2.x
Le fichier JSON 1.x exporté de l’application inclut certaines zones que vous devez modifier avant l’importation dans [LUIS][LUIS] 2.0.

### <a name="prebuilt-entities"></a>Entités prédéfinies
Les [entités prédéfinies](./howto-add-prebuilt-models.md) ont changé. Vérifiez que vous utilisez les entités prédéfinies V2. Cela inclut l’utilisation de [datetimeV2](luis-reference-prebuilt-datetimev2.md), au lieu de datetime.

### <a name="actions"></a>Actions
La propriété actions n’est plus valide. Elle devrait être vide

### <a name="labeled-utterances"></a>Énoncés étiquetés
V1 autorisait les énoncés étiquetés pour inclure des espaces au début ou à la fin du mot ou de l’expression. Espaces supprimés.

## <a name="common-reasons-for-http-response-status-codes"></a>Raisons courantes pour les codes d’état de la réponse HTTP
Voir [Codes de réponse des API LUIS](luis-reference-response-codes.md).

## <a name="next-steps"></a>Étapes suivantes

Utilisez la documentation de l’API v2 pour mettre à jour des appels REST existants vers des API LUIS de [point de terminaison](https://go.microsoft.com/fwlink/?linkid=2092356) et de [création](https://go.microsoft.com/fwlink/?linkid=2092087).

[LUIS]: ./luis-reference-regions.md