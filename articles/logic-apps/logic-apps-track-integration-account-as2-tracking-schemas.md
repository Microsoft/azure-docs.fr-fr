---
title: Schémas de suivi AS2 pour les messages B2B
description: Créer des schémas de suivi pour analyser les messages AS2 dans Azure Logic Apps
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, logicappspm
ms.topic: article
ms.date: 01/01/2020
ms.openlocfilehash: bccf69362279afd9e8148b20b61ff3ea9b472a03
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "76906962"
---
# <a name="create-schemas-for-tracking-as2-messages-in-azure-logic-apps"></a>Créer des schémas pour le suivi des messages AS2 dans Azure Logic Apps

Pour surveiller le succès, les erreurs et les propriétés des messages lors de transactions interentreprises (B2B), vous pouvez utiliser ces schémas de suivi AS2 dans votre compte d’intégration :

* Schéma de suivi de message AS2
* Schéma de suivi de notification d’état du message (MDN) AS2

## <a name="as2-message-tracking-schema"></a>Schéma de suivi de message AS2

```json
{
   "agreementProperties": {
      "senderPartnerName": "",
      "receiverPartnerName": "",
      "as2To": "",
      "as2From": "",
      "agreementName": ""
   },
   "messageProperties": {
      "direction": "",
      "messageId": "",
      "dispositionType": "",
      "fileName": "",
      "isMessageFailed": "",
      "isMessageSigned": "",
      "isMessageEncrypted": "",
      "isMessageCompressed": "",
      "correlationMessageId": "",
      "incomingHeaders": {},
      "outgoingHeaders": {},
      "isNrrEnabled": "",
      "isMdnExpected": "",
      "mdnType": ""
    }
}
```

| Propriété | Obligatoire | Type | Description |
|----------|----------|------|-------------|
| senderPartnerName | Non | String | Nom de partenaire de l’expéditeur du message AS2 |
| receiverPartnerName | Non | String | Nom de partenaire du destinataire du message AS2 |
| as2To | Oui | String | Nom du destinataire du message AS2 extrait des en-têtes du message AS2 |
| as2From | Oui | String | Nom de l’expéditeur du message AS2 extrait des en-têtes du message AS2 |
| agreementName | Non | String | Nom du contrat AS2 dans lequel les messages sont résolus. |
| direction | Oui | String | Direction du flux de messages, à savoir `receive` ou `send` |
| messageId | Non | String | ID du message AS2 extrait des en-têtes du message AS2 |
| dispositionType | Non | String | Valeur de type de disposition de la notification d’état du message (MDN) |
| fileName | Non | String | Nom de fichier extrait de l’en-tête du message AS2 |
| isMessageFailed | Oui | Boolean | Indique si le message AS2 a échoué |
| isMessageSigned | Oui | Boolean | Indique si le message AS2 a été signé |
| isMessageEncrypted | Oui | Boolean | Indique si le message AS2 a été chiffré |
| isMessageCompressed | Oui | Boolean | Indique si le message AS2 a été compressé |
| correlationMessageId | Non | String | ID du message AS2 permettant de mettre en corrélation des messages avec des MDN |
| incomingHeaders | Non | Dictionnaire de JToken | Détails de l’en-tête de message AS2 entrant |
| outgoingHeaders | Non | Dictionnaire de JToken | Détails de l’en-tête de message AS2 sortant |
| isNrrEnabled | Oui | Boolean | Indique s’il faut utiliser une valeur par défaut si la valeur n’est pas connue |
| isMdnExpected | Oui | Boolean | Indique s’il faut utiliser la valeur par défaut si la valeur n’est pas connue |
| mdnType | Oui | Enum | Valeurs autorisées : `NotConfigured`, `Sync` et `Async` |
|||||

## <a name="as2-mdn-tracking-schema"></a>Schéma de suivi de MDN AS2

```json
{
   "agreementProperties": {
      "senderPartnerName": "",
      "receiverPartnerName": "",
      "as2To": "",
      "as2From": "",
      "agreementName": ""
   },
   "messageProperties": {
      "direction": "",
      "messageId": "",
      "originalMessageId": "",
      "dispositionType": "",
      "isMessageFailed": "",
      "isMessageSigned": "",
      "isNrrEnabled": "",
      "statusCode": "",
      "micVerificationStatus": "",
      "correlationMessageId": "",
      "incomingHeaders": {
      },
      "outgoingHeaders": {
      }
   }
}
```

| Propriété | Obligatoire | Type | Description |
|----------|----------|------|-------------|
| senderPartnerName | Non | String | Nom de partenaire de l’expéditeur du message AS2 |
| receiverPartnerName | Non | String | Nom de partenaire du destinataire du message AS2 |
| as2To | Oui | String | Nom du partenaire qui reçoit le message AS2 |
| as2From | Oui | String | Nom du partenaire qui envoie le message AS2 |
| agreementName | Non | String | Nom du contrat AS2 dans lequel les messages sont résolus. |
| direction | Oui | String | Direction du flux de messages, à savoir `receive` ou `send` |
| messageId | Non | String | ID du message AS2 |
| originalMessageId | Non | String | ID du message AS2 d’origine |
| dispositionType | Non | String | Valeur du type de disposition MDN |
| isMessageFailed | Oui | Boolean | Indique si le message AS2 a échoué |
| isMessageSigned | Oui | Boolean | Indique si le message AS2 a été signé |
| isNrrEnabled | Oui | Boolean | Indique s’il faut utiliser la valeur par défaut si la valeur n’est pas connue |
| statusCode | Oui | Enum | Valeurs autorisées : `Accepted`, `Rejected` et `AcceptedWithErrors` |
| micVerificationStatus | Oui | Enum | Valeurs autorisées : `NotApplicable`, `Succeeded` et `Failed` |
| correlationMessageId | Non | String | ID de corrélation, à savoir l’ID du message d’origine avec la MDN configurée |
| incomingHeaders | Non | Dictionnaire de JToken | Détails de l’en-tête de message entrant |
| outgoingHeaders | Non | Dictionnaire de JToken | Détails de l’en-tête de message sortant |
|||||

## <a name="b2b-protocol-tracking-schemas"></a>Schémas de suivi de protocole B2B

Pour plus d’informations sur les schémas de suivi de protocole B2B, consultez :

* [Schémas de suivi X12](logic-apps-track-integration-account-x12-tracking-schema.md)
* [Schémas de suivi B2B personnalisés](logic-apps-track-integration-account-custom-tracking-schema.md)

## <a name="next-steps"></a>Étapes suivantes

* [Superviser les messages B2B avec les journaux d’activité Azure Monitor](../logic-apps/monitor-b2b-messages-log-analytics.md)