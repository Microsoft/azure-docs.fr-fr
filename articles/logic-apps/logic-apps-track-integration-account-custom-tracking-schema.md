---
title: Schémas de suivi personnalisés pour les messages B2B
description: Créer des schémas de suivi personnalisés pour analyser les messages B2B dans Azure Logic Apps
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, logicappspm
ms.topic: article
ms.date: 01/01/2020
ms.openlocfilehash: c82f9cbfaf2e23ddaa5e4b05f4aac4795d3e16a9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "76903048"
---
# <a name="create-custom-tracking-schemas-that-monitor-end-to-end-workflows-in-azure-logic-a"></a>Créer des schémas de suivi personnalisé qui surveillent des flux de travail de bout en bout dans Azure Logic Apps

Azure Logic Apps est doté d'une fonctionnalité de suivi intégré que vous pouvez activer pour certaines parties de votre flux de travail. Cependant, vous pouvez configurer un suivi personnalisé qui consigne les événements du début à la fin des flux de travail, par exemple, pour les flux de travail incluant une application logique, BizTalk Server, SQL Server ou toute autre couche. Cet article fournit un code personnalisé que vous pouvez utiliser dans les couches en dehors de votre application logique.

## <a name="custom-tracking-schema"></a>Schéma de suivi personnalisé

```json
{
   "sourceType": "",
   "source": {
      "workflow": {
         "systemId": ""
      },
      "runInstance": {
         "runId": ""
      },
      "operation": {
         "operationName": "",
         "repeatItemScopeName": "",
         "repeatItemIndex": ,
         "trackingId": "",
         "correlationId": "",
         "clientRequestId": ""
      }
   },
   "events": [
      {
         "eventLevel": "",
         "eventTime": "",
         "recordType": "",
         "record": {}
      }
   ]
}
```

| Propriété | Obligatoire | Type | Description |
|----------|----------|------|-------------|
| sourceType | Oui | String | Type de la source d’exécution avec les valeurs autorisées suivantes : `Microsoft.Logic/workflows`, `custom` |
| source | Oui | Chaîne ou JToken | Si le type de source est `Microsoft.Logic/workflows`, les informations source doivent suivre ce schéma. Si le type de source est `custom`, le schéma est un JToken. |
| systemId | Oui | String | ID système d’application logique |
| runId | Oui | String | ID d’exécution d’application logique |
| operationName | Oui | String | Nom de l’opération (par exemple action ou déclencheur) |
| repeatItemScopeName | Oui | String | Répéter le nom de l’élément si l’action se trouve au sein d’une boucle `foreach` ou `until` |
| repeatItemIndex | Oui | Integer | Indique que l’action fait partie d’une boucle `foreach` ou `until` et correspond à l’index de l’élément répété. |
| trackingId | Non | String | ID de suivi permettant de mettre en corrélation les messages |
| correlationId | Non | String | ID de corrélation permettant de mettre en corrélation les messages |
| clientRequestId | Non | String | Le client peut remplir cette propriété pour mettre en corrélation les messages |
| eventLevel | Oui | String | Niveau de l’événement |
| eventTime | Oui | DateTime | Heure de l’événement au format UTC : *YYYY-MM-DDTHH:MM:SS.00000Z* |
| recordType | Oui | String | Type d'enregistrement de suivi avec la valeur autorisée suivante uniquement : `custom` |
| enregistrement | Oui | JToken | Type d’enregistrement personnalisé au format JToken uniquement |
|||||

## <a name="b2b-protocol-tracking-schemas"></a>Schémas de suivi de protocole B2B

Pour plus d’informations sur les schémas de suivi de protocole B2B, consultez :

* [Schémas de suivi AS2](../logic-apps/logic-apps-track-integration-account-as2-tracking-schemas.md)
* [Schémas de suivi X12](logic-apps-track-integration-account-x12-tracking-schema.md)

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur la [surveillance des messages B2B avec les journaux Azure Monitor](../logic-apps/monitor-b2b-messages-log-analytics.md)