---
title: Supervision de Key Vault avec Azure Event Grid
description: Utilisez Azure Event Grid pour vous abonner aux événements Key Vault
services: key-vault
author: msmbaldwin
manager: rkarlin
ms.service: key-vault
ms.subservice: general
ms.topic: conceptual
ms.date: 11/12/2019
ms.author: mbaldwin
ms.openlocfilehash: 1ee38196f0b1a6e00d385dc0d2c88d45a4291d3b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90087434"
---
# <a name="monitoring-key-vault-with-azure-event-grid"></a>Supervision de Key Vault avec Azure Event Grid

L’intégration entre Key Vault et Event Grid permet aux utilisateurs d’être avertis en cas de modification de l’état d’un secret stocké dans le coffre de clés. Un changement d’état se définit comme étant un secret sur le point d’expirer (30 jours après l’expiration), un secret ayant expiré ou un secret ayant une nouvelle version disponible. Les notifications relatives aux trois types de secret (clé, certificat et secret) sont prises en charge.

Les applications peuvent réagir à ces événements à l’aide d’architectures modernes serverless, sans qu’il soit nécessaire de recourir à du code compliqué ou à des services d’interrogation coûteux et inefficaces. Les événements sont envoyés (push) via [Azure Event Grid](https://azure.microsoft.com/services/event-grid/) à des gestionnaires d’événements tels qu’[Azure Functions](https://azure.microsoft.com/services/functions/), [Azure Logic Apps](https://azure.microsoft.com/services/logic-apps/) ou même votre propre Webhook. Vous payez uniquement pour ce que vous utilisez. Pour plus d’informations sur la tarification, consultez la page [Tarification Event Grid](https://azure.microsoft.com/pricing/details/event-grid/).

## <a name="key-vault-events-and-schemas"></a>Événements et schémas Key Vault

Event Grid utilise les [abonnements aux événements](../../event-grid/concepts.md#event-subscriptions) pour acheminer les messages d’événements vers les abonnés. Les événements Key Vault contiennent toutes les informations dont vous avez besoin pour répondre aux changements apportés à vos données. Vous pouvez facilement identifier un événement Key Vault, car la propriété eventType commence par « Microsoft.KeyVault ».

Pour plus d’informations, consultez le [schéma d’événement Key Vault](../../event-grid/event-schema-key-vault.md).

> [!WARNING]
> Les événements de notification sont déclenchés uniquement sur les nouvelles versions des secrets, clés et certificats, et vous devez d’abord vous abonner à l’événement sur votre coffre de clés pour recevoir ces notifications.

## <a name="practices-for-consuming-events"></a>Pratiques pour la consommation d’événements

Les applications qui gèrent les événements Key Vault doivent suivre certaines pratiques recommandées :

* Plusieurs abonnements peuvent être configurés pour acheminer les événements vers le même gestionnaire d'événements. Il est important de ne pas considérer que les événements proviennent d’une source particulière, mais de vérifier le sujet du message pour avoir la certitude qu’il provient du coffre de clés attendu.
* De même, vérifiez que vous êtes prêt à traiter son eventType, et ne supposez pas que tous les événements reçus seront aux types que vous attendez.
* Ignorez les champs que vous ne comprenez pas.  Cette pratique vous aidera à prendre en charge les nouvelles fonctionnalités qui peuvent être ajoutées à l’avenir.
* Utilisez le préfixe « subject « et les correspondances de suffixe pour limiter les événements à un événement particulier.

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble d’Azure Key Vault](overview.md)
- [Vue d’ensemble d’Azure Event Grid](../../event-grid/overview.md)
- Procédure : [Routez les événements Key Vault vers le runbook Automation](event-grid-tutorial.md).
- Procédure : [Recevoir un e-mail en cas de changement d’un secret de coffre de clés](event-grid-logicapps.md)
- [Schéma des événements Azure Event Grid pour Azure Key Vault](../../event-grid/event-schema-key-vault.md)
- [Vue d’ensemble d’Azure Automation](../../automation/index.yml)
