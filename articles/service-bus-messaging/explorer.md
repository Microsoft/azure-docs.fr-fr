---
title: Utiliser Azure Service Bus Explorer pour effectuer des opérations de données sur Service Bus (préversion)
description: Cet article fournit des informations sur l’utilisation de l’Azure Service Bus Explorer basé sur le portail pour accéder à des données Azure Service Bus.
ms.topic: conceptual
ms.date: 06/23/2020
ms.openlocfilehash: e5e97c6860c2cc01048f4f7caf9f40f9e07592d0
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91295598"
---
# <a name="use-service-bus-explorer-to-perform-data-operations-on-service-bus-preview"></a>Utiliser l’Explorateur Service Bus pour effectuer des opérations de données sur Service Bus (préversion)

## <a name="overview"></a>Vue d’ensemble

Azure Service Bus permet aux applications clientes expéditrices et réceptrices de dissocier leur logique métier à l’aide de la sémantique familière de point à point (File d’attente) et de publication-abonnement (Rubrique-Abonnement).

Les opérations effectuées sur un espace de noms Azure Service Bus sont de deux sortes 
   * Opérations de gestion : création, mise à jour, suppression d’espace de noms, de files d’attente, de rubriques et d’abonnements Service Bus.
   * Opérations de données : échange de messages avec des files d’attente, des rubriques et des abonnements.

L’Explorateur Azure Service Bus étend la fonctionnalité du portail au-delà des opérations de gestion pour prendre en charge les opérations de données (envoi, réception, aperçu) sur les files d’attente, rubriques et abonnements (et leurs sous-entités de lettres mortes), directement à partir du portail Azure.

> [!NOTE]
> Cet article met en évidence la fonctionnalité de l’Explorateur Azure Service Bus qui se trouve sur le Portail Azure.
>
> L’outil Explorateur Azure Service Bus n’est ***pas*** l’outil OSS [Service Bus Explorer](https://github.com/paolosalvatori/ServiceBusExplorer) détenu par la communauté.
>

## <a name="prerequisites"></a>Prérequis

Pour utiliser l’outil Service Bus Explorer, vous devez approvisionner un espace de noms Azure Service Bus. 

Une fois l’espace de noms Service Bus approvisionné, vous devez créer une file d’attente pour échanger des messages avec une rubrique associée à un abonnement pour tester la fonctionnalité.

Pour en savoir plus sur la façon de créer des files d’attente, rubriques et abonnements, suivez les liens ci-dessous :
   * [Démarrage rapide – Créer des files d’attente](service-bus-quickstart-portal.md)
   * [Démarrage rapide – Créer des rubriques](service-bus-quickstart-topics-subscriptions-portal.md)


## <a name="using-the-service-bus-explorer"></a>Utilisation de Service Bus Explorer

Pour utiliser Azure Service Bus Explorer, vous devez accéder à l’espace de noms Service Bus sur lequel vous souhaitez effectuer des opérations d’envoi, d’aperçu et d’affichage.

Si vous cherchez à effectuer des opérations sur une file d’attente, choisissez **Files d’attente** dans le menu de navigation. Si vous cherchez à effectuer des opérations sur une rubrique (et ses abonnements associés), choisissez **Rubriques**. 

:::image type="content" source="./media/service-bus-explorer/queue-topics-left-navigation.png"alt-text="Sélection d’entité":::

Après avoir choisi **Files d’attente** ou **Rubriques**, sélectionnez la file d’attente ou la rubrique souhaitée.

Sélectionnez **Service Bus Explorer (préversion)** dans le menu de navigation de gauche.

:::image type="content" source="./media/service-bus-explorer/left-navigation-menu-selected.png" alt-text="Sélection d’entité":::

### <a name="sending-a-message-to-a-queue-or-topic"></a>Envoi d’un message à une file d’attente ou à une rubrique

Pour envoyer un message à une **File d’attente** ou à une **Rubrique**, cliquez sur l’onglet ***Envoyer*** dans Service Bus Explorer.

Pour composer un message ici : 

1. Choisissez le **Type de contenu** « Texte/Brut », « Application/Xml » ou « Application/Json ».
2. Ajoutez le **Contenu** du message. Assurez-vous qu’il correspond au **Type de contenu** défini précédemment.
3. Définissez les **Propriétés avancées** (facultatif) : ID de corrélation, ID de message, Étiquette, ReplyTo, Durée de vie (TTL) et Heure planifiée de mise en file d’attente (pour les Messages planifiés).
4. Définissez les **Propriétés personnalisées**. Il peut s’agir de toute propriété utilisateur définie par rapport à une clé de dictionnaire.

Une fois le message composé, appuyez sur Envoyer.

:::image type="content" source="./media/service-bus-explorer/send-experience.png" alt-text="Sélection d’entité":::

Lorsque l’opération d’envoi s’est terminée avec succès, 

* En cas d’envoi à la file d’attente, le compteur de métriques **Messages actifs** est incrémenté.

    :::image type="content" source="./media/service-bus-explorer/queue-after-send-metrics.png" alt-text="Sélection d’entité":::

* En cas d’envoi à la Rubrique, le compteur de métriques **Messages actifs** est incrémenté sur l’abonnement vers lequel le message a été acheminé.

    :::image type="content" source="./media/service-bus-explorer/topic-after-send-metrics.png" alt-text="Sélection d’entité":::

### <a name="receiving-a-message-from-a-queue"></a>Réception d’un message d’une file d’attente

La fonction de réception sur Service Bus Explorer permet de recevoir un seul message à la fois. L’opération de réception s’effectue à l’aide du mode **ReceiveAndDelete**.

> [!IMPORTANT]
> Notez que l’opération Recevoir effectuée par Service Bus Explorer est une ***réception destructrice***, c’est-à-dire que le message est supprimé de la file d’attente quand il est affiché sur l’outil Service Bus Explorer.
>
> Pour parcourir les messages sans les supprimer de la file d’attente, envisagez d’utiliser la fonctionnalité ***Aperçu***.
>

Pour recevoir un message d’une File d’attente (ou de sa sous-file d’attente de lettres mortes) : 

1. Cliquez sur l’onglet ***Recevoir*** du Service Bus Explorer.
2. Vérifiez les métriques pour voir s’il existe des **Messages actifs** ou des **Messages de lettres mortes** à recevoir.

    :::image type="content" source="./media/service-bus-explorer/queue-after-send-metrics.png" alt-text="Sélection d’entité":::

3. Choisissez entre la ***File d’attente*** ou la sous-file d’attente ***Lettre morte***.

    :::image type="content" source="./media/service-bus-explorer/queue-or-deadletter.png" alt-text="Sélection d’entité":::

4. Cliquez sur le bouton ***Recevoir***, puis sur ***Oui*** pour confirmer l’opération de réception et suppression.


Lorsque l’opération de réception aboutit, les détails du message s’affichent dans la grille comme ci-dessous. Vous pouvez sélectionner le message dans la grille pour afficher ses détails.

:::image type="content" source="./media/service-bus-explorer/receive-message-from-queue-2.png" alt-text="Sélection d’entité":::


### <a name="peeking-a-message-from-a-queue"></a>Affichage d’aperçu d’un message d’une file d’attente

La fonctionnalité d’aperçu vous permet d’utiliser Service Bus Explorer pour afficher les 32 premiers messages d’une file d’attente ou la file d’attente de lettres mortes.

1. Pour lire le message dans une file d’attente, cliquez sur l’onglet ***Aperçu*** dans le Service Bus Explorer.

    :::image type="content" source="./media/service-bus-explorer/peek-tab-selected.png" alt-text="Sélection d’entité":::

2. Vérifiez les métriques pour voir s’il existe des **Messages actifs** ou des **Messages de lettres mortes** dont afficher l’aperçu.

    :::image type="content" source="./media/service-bus-explorer/queue-after-send-metrics.png" alt-text="Sélection d’entité":::

3. Choisissez ensuite entre la ***File d’attente*** ou la sous-file d’attente ***Lettre morte***.

    :::image type="content" source="./media/service-bus-explorer/queue-or-deadletter.png" alt-text="Sélection d’entité":::

4. Cliquez sur le bouton ***Aperçu***. 

Une fois l’opération d’affichage de l’aperçu terminée, jusqu’à 32 messages s’affichent dans la grille comme ci-dessous. Pour afficher les détails d’un message particulier, sélectionnez-le dans la grille. 

:::image type="content" source="./media/service-bus-explorer/peek-message-from-queue-2.png" alt-text="Sélection d’entité":::

> [!NOTE]
>
> Comme l’affichage d’aperçu n’est pas une opération destructrice, le message **n’est pas** supprimé de la file d’attente.
>

### <a name="receiving-a-message-from-a-subscription"></a>Réception d’un message d’un abonnement

À l’instar d’une file d’attente, l’opération ***Recevoir*** peut être effectuée par rapport à un abonnement (ou à son entité de lettres mortes). Toutefois, étant donné qu’un abonnement se trouve dans le contexte de la rubrique, l’opération de réception est effectuée en accédant au Service Bus Explorer pour une rubrique donnée.

> [!IMPORTANT]
> Notez que l’opération de réception effectuée par le Service Bus Explorer est une ***réception destructrice***, c’est-à-dire que le message est supprimé de la file d’attente quand il est affiché dans l’outil Service Bus Explorer.
>
> Pour parcourir les messages sans les supprimer de la file d’attente, envisagez d’utiliser la fonctionnalité ***Aperçu***.
>

1. Cliquez sur l’onglet ***Recevoir*** et sélectionnez l’***Abonnement***  spécifique dans le sélecteur de liste déroulante.

    :::image type="content" source="./media/service-bus-explorer/receive-subscription-tab-selected.png" alt-text="Sélection d’entité":::

2. Choisissez entre l’***Abonnement*** ou la sous-entité ***DeadLetter***.

    :::image type="content" source="./media/service-bus-explorer/subscription-or-deadletter.png" alt-text="Sélection d’entité":::

3. Cliquez sur le bouton ***Recevoir***, puis sur ***Oui*** pour confirmer l’opération de réception et suppression.

Lorsque l’opération de réception aboutit, le message reçu s’affiche dans la grille comme ci-dessous. Pour afficher les détails du message, cliquez dessus.

:::image type="content" source="./media/service-bus-explorer/receive-message-from-subscription.png" alt-text="Sélection d’entité":::

### <a name="peeking-a-message-from-a-subscription"></a>Affichage de l’aperçu d’un message d’un abonnement

Pour simplement parcourir les messages d’un abonnement ou de sa sous-entité de lettres mortes, la fonctionnalité ***Aperçu*** peut également être utilisée sur l’abonnement.

1. Cliquez sur l’onglet ***Aperçu*** et sélectionnez l’***Abonnement*** spécifique dans le sélecteur de liste déroulante.

    :::image type="content" source="./media/service-bus-explorer/peek-subscription-tab-selected.png" alt-text="Sélection d’entité":::

2. Choisissez entre l’***Abonnement*** ou la sous-entité ***DeadLetter***.

    :::image type="content" source="./media/service-bus-explorer/subscription-or-deadletter.png" alt-text="Sélection d’entité":::

3. Cliquez sur le bouton ***Aperçu***.

Une fois l’opération d’affichage de l’aperçu terminée, jusqu’à 32 messages s’affichent dans la grille comme ci-dessous. Pour afficher les détails d’un message particulier, sélectionnez-le dans la grille. 

:::image type="content" source="./media/service-bus-explorer/peek-message-from-subscription.png" alt-text="Sélection d’entité":::

> [!NOTE]
>
> Comme l’affichage d’aperçu n’est pas une opération destructrice, le message **n’est pas** supprimé de la file d’attente.
>

## <a name="next-steps"></a>Étapes suivantes

   * Apprenez-en davantage sur les [Files d’attente](service-bus-queues-topics-subscriptions.md#queues) et les [Rubriques](service-bus-queues-topics-subscriptions.md#topics-and-subscriptions) Service Bus
   * Apprenez-en davantage sur la création de [Files d’attente Service Bus via le portail Azure](service-bus-quickstart-portal.md)
   * Apprenez-en davantage sur la création de [Rubriques et abonnements Service Bus via le portail Azure](service-bus-quickstart-topics-subscriptions-portal.md)