---
title: Schémas de suivi X12 pour les messages B2B
description: Créer des schémas de suivi pour analyser les messages X12 pour Azure Logic Apps
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, logicappspm
ms.topic: article
ms.date: 01/01/2020
ms.openlocfilehash: 5b2df194761ebc167e67498a985960a4fce35f19
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "76905301"
---
# <a name="create-schemas-for-tracking-x12-messages-in-azure-logic-apps"></a>Créer des schémas pour le suivi des messages X12 dans Azure Logic Apps

Pour surveiller le succès, les erreurs et les propriétés des messages lors de transactions interentreprises (B2B), vous pouvez utiliser ces schémas de suivi X12 dans votre compte d’intégration :

* Schéma de suivi de document informatisé X12
* Schéma de suivi d’accusé de réception de document informatisé X12
* Schéma de suivi d’échange X12
* Schéma de suivi d’accusé de réception d’échange X12
* Schéma de suivi de groupe fonctionnel X12
* Schéma de suivi d’accusé de réception de groupe fonctionnel X12

## <a name="x12-transaction-set-tracking-schema"></a>Schéma de suivi de document informatisé X12

```json
{
   "agreementProperties": {
      "senderPartnerName": "",
      "receiverPartnerName": "",
      "senderQualifier": "",
      "senderIdentifier": "",
      "receiverQualifier": "",
      "receiverIdentifier": "",
      "agreementName": ""
   },
   "messageProperties": {
      "direction": "",
      "interchangeControlNumber": "",
      "functionalGroupControlNumber": "",
      "transactionSetControlNumber": "",
      "CorrelationMessageId": "",
      "messageType": "",
      "isMessageFailed": "",
      "isTechnicalAcknowledgmentExpected": "",
      "isFunctionalAcknowledgmentExpected": "",
      "needAk2LoopForValidMessages": "",
      "segmentsCount": ""
   }
}
```

| Propriété | Obligatoire | Type | Description |
|----------|----------|------|-------------|
| senderPartnerName | Non | String | Nom de partenaire de l’expéditeur du message X12 |
| receiverPartnerName | Non | String | Nom de partenaire du destinataire du message X12 |
| senderQualifier | Oui | String | Qualificateur du partenaire d’envoi |
| senderIdentifier | Oui | String | Identificateur du partenaire d’envoi |
| receiverQualifier | Oui | String | Qualificateur du partenaire de réception |
| receiverIdentifier | Oui | String | Identificateur du partenaire de réception |
| agreementName | Non | String | Nom du contrat X12 dans lequel les messages sont résolus. |
| direction | Oui | Enum | Direction du flux de messages, à savoir `receive` ou `send` |
| interchangeControlNumber | Non | String | Numéro de contrôle de l’échange |
| functionalGroupControlNumber | Non | String | Numéro de contrôle fonctionnel |
| transactionSetControlNumber | Non | String | Numéro de contrôle de document informatisé |
| CorrelationMessageId | Non | String | ID de message de corrélation correspondant à {AgreementName} *{GroupControlNumber}* {TransactionSetControlNumber} |
| messageType | Non | String | Type de document ou de document informatisé |
| isMessageFailed | Oui | Boolean | Indique si le message X12 a échoué |
| isTechnicalAcknowledgmentExpected | Oui | Boolean | Indique si l’accusé de réception technique est configuré dans le contrat X12 |
| isFunctionalAcknowledgmentExpected | Oui | Boolean | Indique si l’accusé de réception fonctionnel est configuré dans le contrat X12 |
| needAk2LoopForValidMessages | Oui | Boolean | Indique si la boucle AK2 est nécessaire pour un message valide |
| segmentsCount | Non | Integer | Nombre de segments dans le document informatisé X12 |
|||||

## <a name="x12-transaction-set-acknowledgment-tracking-schema"></a>Schéma de suivi d’accusé de réception de document informatisé X12

```json
{
   "agreementProperties": {
      "senderPartnerName": "",
      "receiverPartnerName": "",
      "senderQualifier": "",
      "senderIdentifier": "",
      "receiverQualifier": "",
      "receiverIdentifier": "",
      "agreementName": ""
   },
   "messageProperties": {
      "direction": "",
      "interchangeControlNumber": "",
      "functionalGroupControlNumber": "",
      "isaSegment": "",
      "gsSegment": "",
      "respondingfunctionalGroupControlNumber": "",
      "respondingFunctionalGroupId": "",
      "respondingtransactionSetControlNumber": "",
      "respondingTransactionSetId": "",
      "statusCode": "",
      "processingStatus": "",
      "CorrelationMessageId": "",
      "isMessageFailed": "",
      "ak2Segment": "",
      "ak3Segment": "",
      "ak5Segment": ""
   }
}
```

| Propriété | Obligatoire | Type | Description |
|----------|----------|------|-------------|
| senderPartnerName | Non | String | Nom de partenaire de l’expéditeur du message X12 |
| receiverPartnerName | Non | String | Nom de partenaire du destinataire du message X12 |
| senderQualifier | Oui | String | Qualificateur du partenaire d’envoi |
| senderIdentifier | Oui | String | Identificateur du partenaire d’envoi |
| receiverQualifier | Oui | String | Qualificateur du partenaire de réception |
| receiverIdentifier | Oui | String | Identificateur du partenaire de réception |
| agreementName | Non | String | Nom du contrat X12 dans lequel les messages sont résolus. |
| direction | Oui | Enum | Direction du flux de messages, à savoir `receive` ou `send` |
| interchangeControlNumber | Non | String | Numéro de contrôle de l’échange de l’accusé de réception fonctionnel Valeur renseignée uniquement pour l’envoi si un accusé de réception fonctionnel a été reçu pour les messages envoyés au partenaire. |
| functionalGroupControlNumber | Non | String | Numéro de contrôle de groupe fonctionnel de l’accusé de réception fonctionnel. Valeur renseignée uniquement pour l’envoi si un accusé de réception fonctionnel a été reçu pour les messages envoyés au partenaire |
| isaSegment | Non | String | Segment ISA du message. Valeur renseignée uniquement pour l’envoi si un accusé de réception fonctionnel a été reçu pour les messages envoyés au partenaire |
| gsSegment | Non | String | Segment GS du message. Valeur renseignée uniquement pour l’envoi si un accusé de réception fonctionnel a été reçu pour les messages envoyés au partenaire |
| respondingfunctionalGroupControlNumber | Non | String | Numéro de contrôle de l’échange de réponse |
| respondingFunctionalGroupId | Non | String | ID du groupe fonctionnel de réponse qui est mappé à AK101 dans l’accusé de réception |
| respondingtransactionSetControlNumber | Non | String | Numéro de contrôle de document informatisé de réponse |
| respondingTransactionSetId | Non | String | ID du document informatisé de réponse, qui est mappé à AK201 dans l’accusé de réception |
| statusCode | Oui | Boolean | Code d’état de l’accusé de réception du document informatisé |
| segmentsCount | Oui | Enum | Code d’état de l’accusé de réception avec les valeurs autorisées suivantes : `Accepted`, `Rejected` et `AcceptedWithErrors` |
| processingStatus | Oui | Enum | État de traitement de l’accusé de réception avec les valeurs autorisées suivantes : `Received`, `Generated` et `Sent` |
| CorrelationMessageId | Non | String | ID de message de corrélation correspondant à {AgreementName} *{GroupControlNumber}* {TransactionSetControlNumber} |
| isMessageFailed | Oui | Boolean | Indique si le message X12 a échoué |
| ak2Segment | Non | String | Accusé de réception pour un document informatisé dans le groupe fonctionnel reçu |
| ak3Segment | Non | String | Signale les erreurs dans un segment de données |
| ak5Segment | Non | String | Signale si le document informatisé identifié dans le segment AK2 est accepté ou rejeté et pourquoi |
|||||

## <a name="x12-interchange-tracking-schema"></a>Schéma de suivi d’échange X12

```json
{
   "agreementProperties": {
      "senderPartnerName": "",
      "receiverPartnerName": "",
      "senderQualifier": "",
      "senderIdentifier": "",
      "receiverQualifier": "",
      "receiverIdentifier": "",
      "agreementName": ""
   },
   "messageProperties": {
      "direction": "",
      "interchangeControlNumber": "",
      "isaSegment": "",
      "isTechnicalAcknowledgmentExpected": "",
      "isMessageFailed": "",
      "isa09": "",
      "isa10": "",
      "isa11": "",
      "isa12": "",
      "isa14": "",
      "isa15": "",
      "isa16": ""
   }
}
```

| Propriété | Obligatoire | Type | Description |
|----------|----------|------|-------------|
| senderPartnerName | Non | String | Nom de partenaire de l’expéditeur du message X12 |
| receiverPartnerName | Non | String | Nom de partenaire du destinataire du message X12 |
| senderQualifier | Oui | String | Qualificateur du partenaire d’envoi |
| senderIdentifier | Oui | String | Identificateur du partenaire d’envoi |
| receiverQualifier | Oui | String | Qualificateur du partenaire de réception |
| receiverIdentifier | Oui | String | Identificateur du partenaire de réception |
| agreementName | Non | String | Nom du contrat X12 dans lequel les messages sont résolus. |
| direction | Oui | Enum | Direction du flux de messages, à savoir `receive` ou `send` |
| interchangeControlNumber | Non | String | Numéro de contrôle de l’échange |
| isaSegment | Non | String | Segment ISA du message |
| isTechnicalAcknowledgmentExpected | Boolean | Indique si l’accusé de réception technique est configuré dans le contrat X12  |
| isMessageFailed | Oui | Boolean | Indique si le message X12 a échoué |
| isa09 | Non | String | Date d’échange du document X12 |
| isa10 | Non | String | Heure d’échange du document X12 |
| isa11 | Non | String | Identificateur des normes de contrôle d’échange X12 |
| isa12 | Non | String | Numéro de version du contrôle d’échange X12 |
| isa14 | Non | String | Un accusé de réception X12 est exigé |
| isa15 | Non | String | Indicateur de test ou de production |
| isa16 | Non | String | Séparateur d'éléments |
|||||

## <a name="x12-interchange-acknowledgment-tracking-schema"></a>Schéma de suivi d’accusé de réception d’échange X12

```json
{
   "agreementProperties": {
      "senderPartnerName": "",
      "receiverPartnerName": "",
      "senderQualifier": "",
      "senderIdentifier": "",
      "receiverQualifier": "",
      "receiverIdentifier": "",
      "agreementName": ""
   },
   "messageProperties": {
      "direction": "",
      "interchangeControlNumber": "",
      "isaSegment": "",
      "respondingInterchangeControlNumber": "",
      "isMessageFailed": "",
      "statusCode": "",
      "processingStatus": "",
      "ta102": "",
      "ta103": "",
      "ta105": ""
   }
}
```

| Propriété | Obligatoire | Type | Description |
|----------|----------|------|-------------|
| senderPartnerName | Non | String | Nom de partenaire de l’expéditeur du message X12 |
| receiverPartnerName | Non | String | Nom de partenaire du destinataire du message X12 |
| senderQualifier | Oui | String | Qualificateur du partenaire d’envoi |
| senderIdentifier | Oui | String | Identificateur du partenaire d’envoi |
| receiverQualifier | Oui | String | Qualificateur du partenaire de réception |
| receiverIdentifier | Oui | String | Identificateur du partenaire de réception |
| agreementName | Non | String | Nom du contrat X12 dans lequel les messages sont résolus. |
| direction | Oui | Enum | Direction du flux de messages, à savoir `receive` ou `send` |
| interchangeControlNumber | Non | String | Numéro de contrôle de l’échange de l’accusé de réception technique reçu de partenaires |
| isaSegment | Non | String | Segment ISA de l’accusé de réception technique reçu de partenaires |
| respondingInterchangeControlNumber | Non | String | Numéro de contrôle de l’échange de l’accusé de réception technique reçu de partenaires |
| isMessageFailed | Oui | Boolean | Indique si le message X12 a échoué |
| statusCode | Oui | Enum | Code d’état de l’accusé de réception d'échange avec les valeurs autorisées suivantes : `Accepted`, `Rejected` et `AcceptedWithErrors` |
| processingStatus | Oui | Enum | État de l’accusé de réception avec les valeurs autorisées suivantes : `Received`, `Generated` et `Sent` |
| ta102 | Non | String | Date de l’échange |
| ta103 | Non | String | Heure de l’échange |
| ta105 | Non | String | Code de note de l’échange |
|||||

## <a name="x12-functional-group-tracking-schema"></a>Schéma de suivi de groupe fonctionnel X12

```json
{
   "agreementProperties": {
      "senderPartnerName": "",
      "receiverPartnerName": "",
      "senderQualifier": "",
      "senderIdentifier": "",
      "receiverQualifier": "",
      "receiverIdentifier": "",
      "agreementName": ""
   },
   "messageProperties": {
      "direction": "",
      "interchangeControlNumber": "",
      "functionalGroupControlNumber": "",
      "gsSegment": "",
      "isTechnicalAcknowledgmentExpected": "",
      "isFunctionalAcknowledgmentExpected": "",
      "isMessageFailed": "",
      "gs01": "",
      "gs02": "",
      "gs03": "",
      "gs04": "",
      "gs05": "",
      "gs07": "",
      "gs08": ""
   }
}
```

| Propriété | Obligatoire | Type | Description |
|----------|----------|------|-------------|
| senderPartnerName | Non | String | Nom de partenaire de l’expéditeur du message X12 |
| receiverPartnerName | Non | String | Nom de partenaire du destinataire du message X12 |
| senderQualifier | Oui | String | Qualificateur du partenaire d’envoi |
| senderIdentifier | Oui | String | Identificateur du partenaire d’envoi |
| receiverQualifier | Oui | String | Qualificateur du partenaire de réception |
| receiverIdentifier | Oui | String | Identificateur du partenaire de réception |
| agreementName | Non | String | Nom du contrat X12 dans lequel les messages sont résolus |
| direction | Oui | Enum | Direction du flux de messages (envoi ou réception) |
| interchangeControlNumber | Non | String | Numéro de contrôle de l’échange |
| functionalGroupControlNumber | Non | String | Numéro de contrôle fonctionnel |
| gsSegment | Non | String | Segment GS de message |
| isTechnicalAcknowledgmentExpected | Oui | Boolean | Indique si l’accusé de réception technique est configuré dans le contrat X12 |
| isFunctionalAcknowledgmentExpected | Oui | Boolean | Indique si l’accusé de réception fonctionnel est configuré dans le contrat X12 |
| isMessageFailed | Oui | Boolean | Indique si le message X12 a échoué |
| gs01 | Non | String | Code d’identificateur fonctionnel |
| gs02 | Non | String | Code de l’expéditeur de l’application |
| gs03 | Non | String | Code du destinataire de l’application |
| gs04 | Non | String | Date du groupe fonctionnel |
| gs05 | Non | String | Heure du groupe fonctionnel |
| gs07 | Non | String | Code de l’agence responsable |
| gs08 | Non | String | Code identificateur de la version, de la mise en production ou de l'activité |
|||||

## <a name="x12-functional-group-acknowledgment-tracking-schema"></a>Schéma de suivi d’accusé de réception de groupe fonctionnel X12

```json
{
   "agreementProperties": {
      "senderPartnerName": "",
      "receiverPartnerName": "",
      "senderQualifier": "",
      "senderIdentifier": "",
      "receiverQualifier": "",
      "receiverIdentifier": "",
      "agreementName": ""
   },
   "messageProperties": {
      "direction": "",
      "interchangeControlNumber": "",
      "functionalGroupControlNumber": "",
      "isaSegment": "",
      "gsSegment": "",
      "respondingfunctionalGroupControlNumber": "",
      "respondingFunctionalGroupId": "",
      "isMessageFailed": "",
      "statusCode": "",
      "processingStatus": "",
      "ak903": "",
      "ak904": "",
      "ak9Segment": ""
   }
}
```

| Propriété | Obligatoire | Type | Description |
|----------|----------|------|-------------|
| senderPartnerName | Non | String | Nom de partenaire de l’expéditeur du message X12 |
| receiverPartnerName | Non | String | Nom de partenaire du destinataire du message X12 |
| senderQualifier | Oui | String | Qualificateur du partenaire d’envoi |
| senderIdentifier | Oui | String | Identificateur du partenaire d’envoi |
| receiverQualifier | Oui | String | Qualificateur du partenaire de réception |
| receiverIdentifier | Oui | String | Identificateur du partenaire de réception |
| agreementName | Non | String | Nom du contrat X12 dans lequel les messages sont résolus. |
| direction | Oui | Enum | Direction du flux de messages, à savoir `receive` ou `send` |
| interchangeControlNumber | Non | String | Numéro de contrôle de l’échange, qui renseigne le côté envoi lors de la réception d’un accusé de réception technique de partenaires |
| functionalGroupControlNumber | Non | String | Numéro de contrôle de groupe fonctionnel de l’accusé de réception technique, qui renseigne le côté envoi lors de la réception d’un accusé de réception technique de partenaires |
| isaSegment | Non | String | Identique au numéro de contrôle de l’échange, mais renseigné uniquement dans des cas spécifiques |
| gsSegment | Non | String | Identique au numéro de contrôle de groupe fonctionnel, mais renseigné uniquement dans des cas spécifiques |
| respondingfunctionalGroupControlNumber | Non | String | Numéro de contrôle du groupe fonctionnel d’origine |
| respondingFunctionalGroupId | Non | String | Mappé à AK101 dans l’ID de groupe fonctionnel de l’accusé de réception |
| isMessageFailed | Boolean | Indique si le message X12 a échoué |
| statusCode | Oui | Enum | Code d’état de l’accusé de réception avec les valeurs autorisées suivantes : `Accepted`, `Rejected` et `AcceptedWithErrors` |
| processingStatus | Oui | Enum | État de traitement de l’accusé de réception avec les valeurs autorisées suivantes : `Received`, `Generated` et `Sent` |
| ak903 | Non | String | Nombre de documents informatisés reçus |
| ak904 | Non | String | Nombre de documents informatisés acceptés dans le groupe fonctionnel identifié |
| ak9Segment | Non | String | Indique si le groupe fonctionnel identifié dans le segment AK1 est accepté ou rejeté et pourquoi |
|||||

## <a name="b2b-protocol-tracking-schemas"></a>Schémas de suivi de protocole B2B

Pour plus d’informations sur les schémas de suivi de protocole B2B, consultez :

* [Schémas de suivi AS2](../logic-apps/logic-apps-track-integration-account-as2-tracking-schemas.md)
* [Schémas de suivi B2B personnalisés](logic-apps-track-integration-account-custom-tracking-schema.md)

## <a name="next-steps"></a>Étapes suivantes

* [Superviser les messages B2B avec les journaux d’activité Azure Monitor](../logic-apps/monitor-b2b-messages-log-analytics.md)