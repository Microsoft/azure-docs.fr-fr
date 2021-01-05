---
title: Paramètres des messages AS2
description: Guide de référence des paramètres d'envoi et de réception AS2 d'Azure Logic Apps avec Enterprise Integration Pack
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, logicappspm
ms.topic: article
ms.date: 04/22/2019
ms.openlocfilehash: ad047a30b901d71604c775e9882b0f242f094638
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "74793031"
---
# <a name="reference-for-as2-message-settings-in-azure-logic-apps-with-enterprise-integration-pack"></a>Référence relative aux paramètres des messages AS2 dans Azure Logic Apps avec Enterprise Integration Pack

Ce guide de référence décrit les propriétés que vous pouvez définir pour spécifier comment un accord AS2 doit gérer les messages envoyés et reçus entre partenaires commerciaux. Configurez ces propriétés selon l'accord conclu avec le partenaire qui échange des messages avec vous.

<a name="AS2-incoming-messages"></a>

## <a name="as2-receive-settings"></a>Paramètres de réception AS2

![Sélectionnez « Paramètres de réception »](./media/logic-apps-enterprise-integration-as2-message-settings/receive-settings.png)

| Propriété | Obligatoire | Description |
|----------|----------|-------------|
| **Remplacer les propriétés du message** | Non | Remplace les propriétés des messages entrants par vos paramètres de propriété. |
| **Le message doit être signé** | Non | Spécifie si tous les messages entrants doivent être signés numériquement. Si une signature est nécessaire, dans la liste **Certificat**, sélectionnez un certificat public de partenaire invité existant pour valider la signature dans les messages. Si vous n'avez pas de certificat, apprenez-en davantage sur l'[ajout de certificats](../logic-apps/logic-apps-enterprise-integration-certificates.md). |
| **Le message doit être chiffré** | Non | Spécifie si tous les messages entrants doivent être chiffrés. Les messages non chiffrés sont rejetés. Si un chiffrement est nécessaire, dans la liste **Certificat**, sélectionnez un certificat privé de partenaire hôte existant pour déchiffrer les messages entrants. Si vous n'avez pas de certificat, apprenez-en davantage sur l'[ajout de certificats](../logic-apps/logic-apps-enterprise-integration-certificates.md). |
| **Le message doit être compressé** | Non | Spécifie si tous les messages entrants doivent être compressés. Les messages non compressés sont rejetés. |
| **Interdire les ID de message en double** | Non | Spécifie si les messages associés à des ID en double doivent être autorisés. Si vous interdisez les ID en double, sélectionnez l'intervalle de vérification (en nombre de jours). Vous pouvez également choisir de suspendre les doublons. |
| **Texte du MDN** | Non | Spécifie la notification de réception du message (MDN) par défaut à envoyer à l'expéditeur du message. |
| **Envoyer une MDN** | Non | Spécifie si des MDN synchrones doivent être envoyées pour les messages reçus.  |
| **Envoyer une MDN signée** | Non | Spécifie si des MDN signées doivent être envoyées pour les messages reçus. Si une signature est nécessaire, dans la liste **Algorithme MIC**, sélectionnez l'algorithme à utiliser pour signer les messages. |
| **Envoyer une MDN asynchrone** | Non | Spécifie les MDN doivent être envoyées en mode asynchrone. Si vous sélectionnez des MDN asynchrones, dans la zone **URL**, indiquez l'URL à laquelle envoyer les MDN. |
||||

<a name="AS2-outgoing-messages"></a>

## <a name="as2-send-settings"></a>Paramètres d'envoi AS2

![Sélectionnez « Paramètres d'envoi »](./media/logic-apps-enterprise-integration-as2-message-settings/send-settings.png)

| Propriété | Obligatoire | Description |
|----------|----------|-------------|
| **Activer la signature des messages** | Non | Spécifie si tous les messages sortants doivent être signés numériquement. Si une signature est nécessaire, sélectionnez les valeurs suivantes : <p>- Dans la liste **Algorithme de signature**, sélectionnez l'algorithme à utiliser pour signer les messages. <br>- Dans la liste **Certificat**, sélectionnez un certificat privé de partenaire hôte existant pour signer les messages. Si vous n'avez pas de certificat, apprenez-en davantage sur l'[ajout de certificats](../logic-apps/logic-apps-enterprise-integration-certificates.md). |
| **Activer le chiffrement des messages** | Non | Spécifie si tous les messages sortants doivent être chiffrés. Si un chiffrement est nécessaire, sélectionnez les valeurs suivantes : <p>- Dans la liste **Algorithme de chiffrement**, sélectionnez l'algorithme de certificat public de partenaire invité à utiliser pour chiffrer les messages. <br>- Dans la liste **Certificat**, sélectionnez un certificat privé de partenaire invité existant pour chiffrer les messages sortants. Si vous n'avez pas de certificat, apprenez-en davantage sur l'[ajout de certificats](../logic-apps/logic-apps-enterprise-integration-certificates.md). |
| **Activer la compression des messages** | Non | Spécifie si tous les messages sortants doivent être compressés. |
| **Dérouler les en-têtes HTTP** | Non | Place l'en-tête HTTP `content-type` sur une seule ligne. |
| **Transmettre le nom du fichier dans l'en-tête MIME** | Non | Spécifie si le nom du fichier doit être inclus dans l'en-tête MIME. |
| **Exiger la MDN** | Non | Spécifie si des notifications de réception de message (MDN) doivent être reçues pour tous les messages sortants. |
| **Exiger la MDN signée** | Non | Spécifie si des MDN signées doivent être reçues pour tous les messages sortants. Si une signature est nécessaire, dans la liste **Algorithme MIC**, sélectionnez l'algorithme à utiliser pour signer les messages. |
| **Exiger la MDN asynchrone** | Non | Spécifie si les MDN doivent être reçues en mode asynchrone. Si vous sélectionnez des MDN asynchrones, dans la zone **URL**, indiquez l'URL à laquelle envoyer les MDN. |
| **Activer NRR** | Non | Spécifie si une non-répudiation de réception (NRR) est requise. Cet attribut de communication fournit une preuve de réception des données. |
| **Format de l'algorithme SHA2** | Non | Spécifie le format de 'algorithme MIC à utiliser pour la signature dans les en-têtes des MDN ou des messages AS2 sortants. |
||||

## <a name="next-steps"></a>Étapes suivantes

[Échanger des messages AS2](../logic-apps/logic-apps-enterprise-integration-as2.md)
