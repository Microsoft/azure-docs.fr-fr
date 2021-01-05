---
title: Définitions de métriques pour Azure Communication Services
titleSuffix: An Azure Communication Services concept document
description: Ce document couvre les définitions de métriques disponibles dans le portail Azure.
author: mikben
manager: jken
services: azure-communication-services
ms.author: mikben
ms.date: 05/19/2020
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: 25c7016f6639df46a9279ef9a9aab2736efd4f95
ms.sourcegitcommit: 230d5656b525a2c6a6717525b68a10135c568d67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94888655"
---
# <a name="metrics-overview"></a>Vue d’ensemble des métriques

Azure Communication Services fournit des métriques pour la conversation et les SMS. [Azure Metrics Explorer](../../azure-monitor/platform/metrics-getting-started.md) peut être utilisé pour tracer vos propres graphiques, examiner les anomalies dans vos valeurs de métrique et comprendre le trafic de vos API à l’aide des données de métriques émises par les demandes des services de conversation et SMS.

## <a name="where-to-find-metrics"></a>Où trouver les métriques

Les services de conversation et de SMS dans Azure Communication Services émettent des métriques pour les demandes des API. Ces métriques se trouvent dans le panneau Métriques sous votre ressource Communication Services. Vous pouvez également créer des tableaux de bord permanents à l’aide du panneau de classeurs sous votre ressource Communication Services.

## <a name="metric-definitions"></a>Définitions de métriques

Deux types de demandes sont représentés dans les métriques Communication Services : **Demandes d’API de conversation** et **demandes d’API SMS**.

Les métriques de demande d’API SMS et de conversation contiennent trois dimensions que vous pouvez utiliser pour filtrer vos données de métriques. Ces dimensions peuvent être agrégées ensemble à l’aide du type d’agrégation `Count` et prennent en charge toutes les séries chronologiques d’agrégation Azure standard, notamment `Sum`, `Average`, `Min` et `Max`.

Vous trouverez plus d’informations sur les types d’agrégation pris en charge et les agrégations de série chronologique dans [Fonctionnalités avancées d’Azure Metrics Explorer](../../azure-monitor/platform/metrics-charts.md#changing-aggregation).

- **Opération** : toutes les opérations ou routes qui peuvent être appelées sur la passerelle de conversation ACS.
- **Code d’état** : réponse de code d’état envoyée après la demande.
- **StatusSubClass** : série de codes d’état envoyée après la réponse. 


### <a name="chat-api-request-metric-operations"></a>Opérations pour les métriques des demandes d’API de conversation

Les opérations suivantes sont disponibles sur les métriques des demande d’API de conversation :

| Operation/Route    | Description                                                                                    |
| -------------------- | ---------------------------------------------------------------------------------------------- |
| GetChatMessage       | Obtient un message par ID de message. |
| ListChatMessages     | Obtient une liste de messages de conversation à partir d’un fil. |
| SendChatMessage      | Envoie un message de conversation à un fil. |
| UpdateChatMessage    | Met à jour un message de conversation. |
| DeleteChatMessage    | Supprime un message de conversation. |
| GetChatThread        | Obtient un fil de conversation. |
| ListChatThreads      | Obtient la liste des fils de conversation d’un utilisateur. |
| UpdateChatThread     | Met à jour les propriétés d’un fil de conversation. |
| CreateChatThread     | Crée un fil de conversation. |
| DeleteChatThread     | Supprime un fil. |
| GetReadReceipts      | Obtient les confirmations de lecture d’un fil. |
| SendReadReceipt      | Envoie un événement de confirmation de lecture à un fil, pour le compte d’un utilisateur. |
| SendTypingIndicator           | Publie un événement de frappe sur un fil, pour le compte d’un utilisateur. |
| ListChatThreadParticipants    | Obtient les membres d’un fil. |
| AddChatThreadParticipants     | Ajoute des membres de fil à un fil. Si les membres existent déjà, aucune modification ne se produit. |
| RemoveChatThreadParticipant   | Supprimer un membre d’un fil. |

:::image type="content" source="./media/chat-metric.png" alt-text="Métrique de demande d’API de conversation":::

Si une demande est envoyée à une opération qui n’est pas reconnue, vous recevez une réponse dont la valeur est « Bad Route » (route incorrecte).

### <a name="sms-api-requests"></a>Demandes d’API SMS

Les opérations suivantes sont disponibles sur les métriques des demandes d’API SMS :

| Operation/Route    | Description                                                                                    |
| -------------------- | ---------------------------------------------------------------------------------------------- |
| SMSMessageSent       | Envoie un SMS. |
| SMSDeliveryReportsReceived     | Obtient les rapports de remise de SMS |
| SMSMessagesReceived      | Obtient les SMS. |


:::image type="content" source="./media/sms-metric.png" alt-text="Métrique de demande d’API SMS":::

## <a name="next-steps"></a>Étapes suivantes

- Découvrez-en plus sur les [métriques de la plateforme de données](../../azure-monitor/platform/data-platform-metrics.md).