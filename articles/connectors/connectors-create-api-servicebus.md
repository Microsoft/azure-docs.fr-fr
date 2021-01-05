---
title: Échanger des messages avec Azure Service Bus
description: Créer des tâches et des flux de travail automatisés pour envoyer et recevoir des messages en utilisant Azure Service Bus dans Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: logicappspm
ms.topic: conceptual
ms.date: 10/22/2020
tags: connectors
ms.openlocfilehash: b6276ff940d8b156a671cb5386ce53ede30dd879
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96019705"
---
# <a name="exchange-messages-in-the-cloud-by-using-azure-logic-apps-and-azure-service-bus"></a>Échanger des messages dans le cloud en utilisant Azure Logic Apps et Azure Service Bus

Avec [Azure Logic Apps](../logic-apps/logic-apps-overview.md) et le connecteur [Azure Service Bus](../service-bus-messaging/service-bus-messaging-overview.md), vous pouvez créer des tâches et des workflows automatisés pour transférer des données, comme des documents de vente, des bons de commande, des journaux ou des mouvements de stock, entre des applications de votre organisation. Non seulement le connecteur surveille, envoie et gère les messages, mais il effectue également des actions relatives aux files d’attente, sessions, rubriques, abonnements, etc. Par exemple :

* Surveillance de la réception des messages (semi-automatique ou passage furtif) dans les files d’attente, les rubriques et les abonnements aux rubriques
* Envoi de messages
* Création et suppression des abonnements de rubrique
* Gestion des messages dans les files d’attente et les abonnements aux rubriques, par exemple : obtention, obtention différée, exécution, report, abandon et lettre morte
* Renouvellement des verrous sur les messages et les sessions dans les files d’attente et les abonnements aux rubriques
* Clôture des sessions dans les rubriques et les files d’attente

Vous pouvez utiliser des déclencheurs afin d’obtenir des réponses de Service Bus et mettre la sortie à la disposition d’autres actions dans vos applications logiques. Vous pouvez également faire en sorte que d’autres actions utilisent la sortie d’actions Service Bus. Si vous ne connaissez pas Service Bus et Logic Apps, consultez [Qu’est-ce qu’Azure Service Bus ?](../service-bus-messaging/service-bus-messaging-overview.md) et [Qu’est-ce qu’Azure Logic Apps ?](../logic-apps/logic-apps-overview.md)

[!INCLUDE [Warning about creating infinite loops](../../includes/connectors-infinite-loops.md)]

## <a name="prerequisites"></a>Prérequis

* Un compte et un abonnement Azure. Si vous n’avez pas d’abonnement Azure, [inscrivez-vous pour bénéficier d’un compte Azure gratuit](https://azure.microsoft.com/free/).

* Un espace de noms et une entité de messagerie Service Bus, telle une file d’attente. Ces éléments et votre application logique doivent utiliser le même abonnement Azure. Si vous ne disposez pas de ces éléments, découvrez comment [créer votre espace de noms Service Bus et une file d’attente](../service-bus-messaging/service-bus-create-namespace-portal.md).

* Des connaissances de base en [création d’applications logiques](../logic-apps/quickstart-create-first-logic-app-workflow.md)

* L’application logique dans laquelle vous utilisez l’espace de noms Service Bus et l’entité de messagerie. Votre application logique et Service Bus doivent utiliser le même abonnement Azure. Pour démarrer votre workflow avec un déclencheur Service Bus, [créez une application logique vide](../logic-apps/quickstart-create-first-logic-app-workflow.md). Pour utiliser une action Service Bus dans votre workflow, démarrez votre application logique avec un autre déclencheur, par exemple, le [déclencheur Périodicité](../connectors/connectors-native-recurrence.md).

<a name="permissions-connection-string"></a>

## <a name="check-permissions"></a>Vérifier les autorisations

Vérifiez que votre application logique dispose des autorisations pour accéder à l’espace de noms Service Bus.

1. Sur le [portail Azure](https://portal.azure.com) connectez-vous avec votre compte Azure.

1. Accédez à votre *espace de noms* Service Bus. Dans la page de l’espace de noms, sous **Paramètres**, sélectionnez **Stratégies d’accès partagé**. Sous **Revendications**, vérifiez que vous disposez des autorisations **Gérer** pour cet espace de noms.

   ![Gérer les autorisations pour l’espace de noms Service Bus](./media/connectors-create-api-azure-service-bus/azure-service-bus-namespace.png)

1. Récupérez la chaîne de connexion pour votre espace de noms Service Bus. Vous avez besoin de cette chaîne au moment où vous fournissez les informations de connexion dans votre application logique.

   1. Dans le volet **Stratégies d’accès partagé**, sélectionnez **RootManageSharedAccessKey**.

   1. En regard de votre chaîne de connexion principale, sélectionnez le bouton de copie. Enregistrez la chaîne de connexion en vue d’une utilisation ultérieure.

      ![Copier la chaîne de connexion de l’espace de noms Service Bus](./media/connectors-create-api-azure-service-bus/find-service-bus-connection-string.png)

   > [!TIP]
   > Pour vous assurer que votre chaîne de connexion est bien associée à votre espace de noms Service Bus ou à une entité de messagerie (une file d'attente par exemple), recherchez la chaîne de connexion pour le paramètre `EntityPath`. Si vous trouvez ce paramètre, la chaîne de connexion correspond à une entité spécifique. De fait, il ne s’agit pas de la chaîne appropriée à utiliser avec votre application logique.

## <a name="add-service-bus-trigger"></a>Ajouter un déclencheur Service Bus

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

1. Connectez-vous au [portail Azure](https://portal.azure.com) et ouvrez votre application logique vide dans le concepteur d’application logique.

1. Dans le champ de recherche du portail, entrez `azure service bus`. Dans la liste des déclencheurs qui apparaît, sélectionnez celui qui vous intéresse.

   Par exemple, pour déclencher votre application logique quand un nouvel élément est envoyé à une file d’attente Service Bus, sélectionnez le déclencheur **Quand une file d’attente reçoit un message (saisie semi-automatique)** .

   ![Sélectionner un déclencheur Service Bus](./media/connectors-create-api-azure-service-bus/select-service-bus-trigger.png)

   Voici quelques éléments à prendre en compte lorsque vous utilisez un déclencheur Service Bus :

   * Tous les déclencheurs Service Bus sont des déclencheurs à *interrogation longue*. Cette description signifie que lorsqu’un déclencheur est activé, celui-ci traite tous les messages et attend 30 secondes le temps qu’un plus grand nombre de messages apparaissent dans la file d’attente ou l’abonnement à la rubrique. Si aucun message n’apparaît au bout de 30 secondes, l’exécution du déclencheur est ignorée. Dans le cas contraire, le déclencheur poursuit la lecture des messages jusqu’à ce que la rubrique d’abonnement ou la file d’attente soit vide. La prochaine interrogation de déclencheur est basée sur l’intervalle de récurrence spécifié dans les propriétés du déclencheur.

   * Certains déclencheurs (par exemple, **Quand un ou plusieurs messages arrivent dans une file d’attente (autocomplétion)** ) peuvent retourner un ou plusieurs messages. Quand ces déclencheurs sont activés, ils retournent entre un et le nombre de messages spécifié par la propriété **Nombre maximal de messages** du déclencheur.

     > [!NOTE]
     > Le déclencheur d’autocomplétion rédige automatiquement un message, mais la saisie semi-automatique se produit uniquement lors de l’appel suivant à Service Bus. Ce comportement peut affecter la conception de votre application logique. Par exemple, évitez de modifier la concurrence sur le déclencheur d’autocomplétion, car cette modification peut entraîner la duplication de messages si votre application logique entre dans un état limité. La modification du contrôle de la concurrence crée les conditions suivantes : les déclencheurs limités sont ignorés avec le code `WorkflowRunInProgress`, l’opération d’achèvement n’a pas lieu et l’exécution du déclencheur suivant se produit après l’intervalle d’interrogation. Vous devez définir la durée de verrouillage du bus de service sur une valeur supérieure à la fréquence d’interrogation. Toutefois, malgré ce réglage, le message peut ne pas être complet si votre application logique reste à un état limité à l’intervalle d’interrogation suivant.

   * Si vous [activez le paramètre de concurrence](../logic-apps/logic-apps-workflow-actions-triggers.md#change-trigger-concurrency) pour un déclencheur Service Bus, la valeur par défaut de la propriété `maximumWaitingRuns` est 10. En fonction du paramètre de la durée de verrouillage de l'entité Service Bus et de la durée d'exécution de votre instance d'application logique, cette valeur par défaut peut être trop élevée et entraîner une exception de type « verrouillage perdu ». Afin de trouver la valeur optimale dans votre scénario, commencez par essayer une valeur de 1 ou 2 pour la propriété `maximumWaitingRuns`. Pour modifier la valeur maximale des exécutions en attente, consultez [Modifier la limite des exécutions en attente](../logic-apps/logic-apps-workflow-actions-triggers.md#change-waiting-runs).

1. Si votre déclencheur se connecte à votre espace de noms Service Bus pour la première fois, suivez ces étapes quand le concepteur d’application logique vous invite à fournir vos informations de connexion.

   1. Fournissez un nom pour votre connexion, puis sélectionnez votre espace de noms Service Bus.

      ![Capture d’écran montrant le nom de la connexion et la sélection d’un espace de noms Service Bus](./media/connectors-create-api-azure-service-bus/create-service-bus-connection-trigger-1.png)

      Pour entrer manuellement la chaîne de connexion, sélectionnez **Entrer manuellement les informations de connexion**. Si vous ne disposez pas de chaîne de connexion, découvrez [comment obtenir votre chaîne de connexion](#permissions-connection-string).

   1. Sélectionnez votre stratégie Service Bus, puis **Créer**.

      ![Capture d’écran montrant comment sélectionner une stratégie Service Bus](./media/connectors-create-api-azure-service-bus/create-service-bus-connection-trigger-2.png)

   1. Sélectionnez l’entité de messagerie souhaitée, comme une file d’attente ou une rubrique. Pour cet exemple, sélectionnez votre file d’attente Service Bus.
   
      ![Capture d’écran montrant comment sélectionner une file d’attente Service Bus](./media/connectors-create-api-azure-service-bus/service-bus-select-queue-trigger.png)

1. Fournissez les informations nécessaires pour le déclencheur sélectionné. Pour ajouter d’autres propriétés disponibles à l’action, ouvrez la liste **Ajouter un nouveau paramètre**, puis sélectionnez les propriétés souhaitées.

   Pour le déclencheur de cet exemple, sélectionnez l’intervalle d’interrogation et la fréquence de vérification de la file d’attente.

   ![Capture d’écran montrant la définition d’une fréquence d’interrogation sur le déclencheur du Service Bus](./media/connectors-create-api-azure-service-bus/service-bus-trigger-details.png)

   Pour plus d’informations sur les déclencheurs et les propriétés disponibles, consultez la page d’[informations de référence](/connectors/servicebus/) du connecteur.

1. Poursuivez la création de votre application logique en ajoutant les actions de votre choix.

   Par exemple, vous pouvez ajouter une action qui envoie un e-mail lorsqu’un nouveau message arrive. Lorsque votre déclencheur vérifie la file d’attente et trouve un nouveau message, l’application logique exécute les actions que vous avez sélectionnées pour le message trouvé.

## <a name="add-service-bus-action"></a>Ajouter une action Service Bus

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

1. Dans le [portail Azure](https://portal.azure.com), ouvrez votre application logique dans le Concepteur d’applications logiques.

1. En dessous l’étape où vous voulez ajouter une action, sélectionnez **Nouvelle étape**.

   Ou bien, pour ajouter une action entre deux étapes, placez votre pointeur sur la flèche qui les sépare. Sélectionnez le signe plus ( **+** ) qui s’affiche, puis sélectionnez **Ajouter une action**.

1. Sous **Choisir une action**, dans la zone de recherche, entrez `azure service bus`. Dans la liste des actions qui apparaît, sélectionnez celle qui vous intéresse. 

   Pour cet exemple, sélectionnez l’action **Envoyer un message**.

   ![Capture d’écran montrant l’action du Service Bus](./media/connectors-create-api-azure-service-bus/select-service-bus-send-message-action.png) 

1. Si votre action se connecte à votre espace de noms Service Bus pour la première fois, suivez ces étapes quand le concepteur d’application logique vous invite à fournir vos informations de connexion.

   1. Fournissez un nom pour votre connexion, puis sélectionnez votre espace de noms Service Bus.

      ![Capture d’écran montrant un nom de connexion et la sélection d’un espace de noms Service Bus](./media/connectors-create-api-azure-service-bus/create-service-bus-connection-action-1.png)

      Pour entrer manuellement la chaîne de connexion, sélectionnez **Entrer manuellement les informations de connexion**. Si vous ne disposez pas de chaîne de connexion, découvrez [comment obtenir votre chaîne de connexion](#permissions-connection-string).

   1. Sélectionnez votre stratégie Service Bus, puis **Créer**.

      ![Capture d’écran montrant la sélection d’une stratégie Service Bus et la sélection du bouton Créer](./media/connectors-create-api-azure-service-bus/create-service-bus-connection-action-2.png)

   1. Sélectionnez l’entité de messagerie souhaitée, comme une file d’attente ou une rubrique. Pour cet exemple, sélectionnez votre file d’attente Service Bus.

      ![Capture d’écran montrant la sélection d’une file d’attente Service Bus](./media/connectors-create-api-azure-service-bus/service-bus-select-queue-action.png)

1. Fournissez les informations nécessaires pour l’action sélectionnée. Pour ajouter d’autres propriétés disponibles à l’action, ouvrez la liste **Ajouter un nouveau paramètre**, puis sélectionnez les propriétés souhaitées.

   Par exemple, sélectionnez les propriétés **Contenu** et **Type de contenu** pour pouvoir les ajouter à l’action. Ensuite, spécifiez le contenu du message que vous voulez envoyer.

   ![Capture d’écran montrant le type de contenu du message et des détails](./media/connectors-create-api-azure-service-bus/service-bus-send-message-details.png)

   Pour plus d’informations sur les actions disponibles et leurs propriétés, consultez la page d’[informations de référence](/connectors/servicebus/) du connecteur.

1. Poursuivez la création de votre application logique en ajoutant éventuellement d’autres actions.

   Par exemple, vous pouvez ajouter une action qui envoie un e-mail confirmant que votre message a bien été envoyé.

1. Enregistrez votre application logique. Dans la barre d’outils du Concepteur, sélectionnez **Enregistrer**.

<a name="sequential-convoy"></a>

## <a name="send-correlated-messages-in-order"></a>Envoyer des messages corrélés dans l'ordre

Lorsque vous devez envoyer des messages connexes dans un ordre précis, vous pouvez utiliser le modèle de [*convoi séquentiel*](/azure/architecture/patterns/sequential-convoy) à l'aide du [connecteur Azure Service Bus](../connectors/connectors-create-api-servicebus.md). Les messages corrélés possèdent une propriété qui définit la relation entre ces messages, comme l'ID de la [session](../service-bus-messaging/message-sessions.md) dans Service Bus.

Lorsque vous créez une application logique, vous pouvez sélectionner le modèle **Livraison corrélée dans l'ordre à l'aide de sessions Service Bus**, qui implémente le modèle de convoi séquentiel. Pour plus d'informations, consultez [Envoyer des messages connexes dans l'ordre](../logic-apps/send-related-messages-sequential-convoy.md).

## <a name="delays-in-updates-to-your-logic-app-taking-effect"></a>Retards dans l’entrée en vigueur des mises à jour de votre application logique

Si l’intervalle d’interrogation d’un déclencheur Service Bus est faible, par exemple 10 secondes, les mises à jour de votre application logique peuvent ne pas prendre effet avant 10 minutes. Pour contourner ce problème, vous pouvez augmenter temporairement l’intervalle d’interrogation à une valeur plus élevée, par exemple 30 secondes ou 1 minute, avant de mettre à jour votre application logique. Après avoir effectué la mise à jour, vous pouvez rétablir la valeur d’origine de l’intervalle d’interrogation. 

<a name="connector-reference"></a>

## <a name="connector-reference"></a>Référence de connecteur

À partir d’un bus de service, le connecteur Service Bus peut enregistrer jusqu’à 1 500 sessions uniques à la fois dans le cache du connecteur, par [entité de messagerie Service Bus, comme un abonnement ou une rubrique](../service-bus-messaging/service-bus-queues-topics-subscriptions.md). Si le nombre de sessions dépasse cette limite, les anciennes sessions sont supprimées du cache. Pour plus d’informations, consultez les [sessions de messages](../service-bus-messaging/message-sessions.md).

Pour obtenir d'autres détails techniques sur les déclencheurs, les actions et les limites, qui sont fournis par la description OpenAPI du connecteur (anciennement Swagger), consultez la [page de référence du connecteur](/connectors/servicebus/). Pour plus d’informations sur la messagerie Azure Service Bus, consultez [qu’est-ce que Azure Service Bus](../service-bus-messaging/service-bus-messaging-overview.md) ?

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les autres [connecteurs d’applications logiques](../connectors/apis-list.md)
