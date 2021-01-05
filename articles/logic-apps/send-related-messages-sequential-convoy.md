---
title: Envoi de messages corrélés dans l’ordre en suivant un convoi séquentiel
description: Envoyez des messages connexes dans l’ordre en suivant le mode de convoi séquentiel dans Azure Logic Apps avec Azure Service Bus.
services: logic-apps
ms.suite: integration
ms.reviewer: apseth, divswa, logicappspm
ms.topic: conceptual
ms.date: 05/29/2020
ms.openlocfilehash: 8c00d2e4f622bcfad7b2468013336f0d936e318c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87048662"
---
# <a name="send-related-messages-in-order-by-using-a-sequential-convoy-in-azure-logic-apps-with-azure-service-bus"></a>Envoi de messages connexes dans l’ordre en suivant un convoi séquentiel dans Azure Logic Apps avec Azure Service Bus

Lorsque vous devez envoyer des messages corrélés dans un ordre précis, vous pouvez suivre le [mode de *convoi séquentiel*](/azure/architecture/patterns/sequential-convoy) si vous utilisez [Azure Logic Apps](../logic-apps/logic-apps-overview.md) à l’aide du [connecteur Azure Service Bus](../connectors/connectors-create-api-servicebus.md). Les messages corrélés possèdent une propriété qui définit la relation qu’ils entretiennent, comme l’ID de la [session](../service-bus-messaging/message-sessions.md) dans Service Bus.

Supposons, par exemple, que vous ayez 10 messages pour une session nommée « Session 1 » et 5 autres pour une session nommée « Session 2 », qui sont tous envoyés à la même [file d’attente Service Bus](../service-bus-messaging/service-bus-queues-topics-subscriptions.md). Vous pouvez créer une application logique qui traite les messages de la file d’attente de sorte que tous ceux de la « Session 1 » soient gérés par une seule exécution de déclencheur et que tous ceux de la « Session 2 » soient gérés par la suivante.

![Mode de convoi séquentiel général](./media/send-related-messages-sequential-convoy/sequential-convoy-pattern-general.png)

Cet article explique comment créer une application logique qui implémente ce mode à l’aide du modèle **Distribution corrélée dans l’ordre à l’aide de sessions Service Bus**. Ce modèle définit un flux de travail d’application logique qui commence par le déclencheur **À la réception d’un message dans une file d’attente (peek-lock)** du connecteur Service Bus, recevant les messages d’une [file d’attente Service Bus](../service-bus-messaging/service-bus-queues-topics-subscriptions.md). Voici les grandes étapes suivies par cette application logique :

* Initialise une session en fonction d’un message lu par le déclencheur à partir de la file d’attente Service Bus.

* Lit et traite tous les messages de la même session dans la file d’attente pendant l’exécution du flux de travail en cours.

Pour consulter le fichier JSON de ce modèle, consultez [GitHub : service-bus-sessions.json](https://github.com/Azure/logicapps/blob/master/templates/service-bus-sessions.json).

Pour plus d’informations, consultez [Mode de convoi séquentiel – Modes de conception cloud de l’architecture Azure](/azure/architecture/patterns/sequential-convoy).

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas encore d’abonnement, vous pouvez [vous inscrire pour obtenir un compte Azure gratuitement](https://azure.microsoft.com/free/).

* Un espace de noms Service Bus et une [file d’attente Service Bus](../service-bus-messaging/service-bus-queues-topics-subscriptions.md), entité de messagerie que vous allez utiliser dans votre application logique. Ces éléments et votre application logique doivent utiliser le même abonnement Azure. Veillez à sélectionner **Activer les sessions** lors de la création de la file d’attente. Si vous ne disposez pas de ces éléments, découvrez comment [créer votre espace de noms et une file d’attente Service Bus](../service-bus-messaging/service-bus-create-namespace-portal.md).

  [!INCLUDE [Warning about creating infinite loops](../../includes/connectors-infinite-loops.md)]

* Des connaissances de base en création d’applications logiques. Si vous débutez avec Azure Logic Apps, suivez le guide de démarrage rapide [Création d’un premier flux de travail automatisé](../logic-apps/quickstart-create-first-logic-app-workflow.md).

<a name="permissions-connection-string"></a>

## <a name="check-access-to-service-bus-namespace"></a>Vérification de l’accès total à l’espace de noms Service Bus

Si vous ne savez pas si votre application logique dispose des autorisations nécessaires pour accéder à votre espace de noms Service Bus, confirmez ces autorisations.

1. Connectez-vous au [portail Azure](https://portal.azure.com). Recherchez et sélectionnez votre *espace de noms* Service Bus.

1. Dans le menu de l’espace de noms, sous **Paramètres**, sélectionnez **Stratégies d’accès partagé**. Sous **Revendications**, vérifiez que vous disposez des autorisations **Gérer** pour cet espace de noms.

   ![Gérer les autorisations pour l’espace de noms Service Bus](./media/send-related-messages-sequential-convoy/check-service-bus-permissions.png)

1. Récupérez maintenant la chaîne de connexion de votre espace de noms Service Bus. Vous pourrez l’utiliser plus tard, au moment de créer une connexion à l’espace de noms à partir de votre application logique.

   1. Dans le volet **Stratégies d’accès partagé**, sélectionnez **RootManageSharedAccessKey** sous **Stratégie**.
   
   1. En regard de votre chaîne de connexion principale, sélectionnez le bouton de copie. Enregistrez la chaîne de connexion en vue d’une utilisation ultérieure.

      ![Copier la chaîne de connexion de l’espace de noms Service Bus](./media/send-related-messages-sequential-convoy/copy-service-bus-connection-string.png)

   > [!TIP]
   > Pour vous assurer que votre chaîne de connexion est bien associée à votre espace de noms Service Bus ou à une entité de messagerie (une file d’attente par exemple), recherchez la chaîne de connexion pour le paramètre `EntityPath` . Si vous trouvez ce paramètre, la chaîne de connexion correspond à une entité spécifique. De fait, il ne s’agit pas de la chaîne appropriée à utiliser avec votre application logique.

## <a name="create-logic-app"></a>Créer une application logique

Dans cette section, vous allez créer une application logique en suivant le modèle **Distribution corrélée dans l’ordre à l’aide de sessions Service Bus**, qui comprend le déclencheur et les actions permettant d’implémenter ce mode de flux de travail. Vous allez également créer une connexion à votre espace de noms Service Bus et spécifier le nom de la file d’attente Service Bus à utiliser.

1. Dans le [portail Azure](https://portal.azure.com), créez une application logique vide. Sur la page d’accueil Azure, sélectionnez **Créer une ressource** > **Intégration** > **Application logique**.

1. Lorsque la Galerie de modèles s’affiche, faites défiler la page après la section des vidéos et des déclencheurs courants. Dans la section **Modèles**, sélectionnez le modèle **Distribution corrélée dans l’ordre à l’aide de sessions Service Bus**.

   ![Sélectionner le modèle « Distribution corrélée dans l’ordre à l’aide de sessions Service Bus »](./media/send-related-messages-sequential-convoy/select-correlated-in-order-delivery-template.png)

1. Quand la boîte de confirmation s’affiche, sélectionnez **Utiliser ce modèle**.

1. Dans le Concepteur d’applications logiques, sélectionnez **Continuer** dans la forme **Service Bus**, puis sélectionnez le signe plus ( **+** ) qui apparaît dans la forme.

   ![Sélectionner « Continuer » pour se connecter à Azure Service Bus](./media/send-related-messages-sequential-convoy/connect-to-service-bus.png)

1. Créez maintenant une connexion Service Bus en choisissant l’une des options suivantes :

   * Pour utiliser la chaîne de connexion que vous avez copiée à partir de votre espace de noms Service Bus, procédez de la façon suivante :

     1. Sélectionnez **Entrer manuellement les informations de connexion**.

     1. Dans **Nom de la connexion**, donnez un nom à votre connexion. Dans **Chaîne de connexion**, collez la chaîne de connexion de votre espace de noms, puis sélectionnez **Créer**, par exemple :

        ![Entrer le nom de la connexion et la chaîne de connexion Service Bus](./media/send-related-messages-sequential-convoy/provide-service-bus-connection-string.png)

        > [!TIP]
        > Si vous ne disposez pas de cette chaîne de connexion, découvrez comment [Rechercher et copier la chaîne de connexion de l’espace de noms Service Bus](#permissions-connection-string).

   * Pour sélectionner un espace de noms Service Bus à partir de votre abonnement Azure actuel, procédez de la façon suivante :

     1. Dans **Nom de la connexion**, donnez un nom à votre connexion. Dans **Espace de noms Service Bus**, sélectionnez votre espace de noms Service Bus, par exemple :

        ![Entrer le nom de la connexion et sélectionner l’espace de noms Service Bus](./media/send-related-messages-sequential-convoy/create-service-bus-connection.png)

     1. Lorsque le volet suivant s’affiche, sélectionnez votre stratégie Service Bus, puis **Créer**.

        ![Sélectionner la stratégie Service Bus, puis « Créer »](./media/send-related-messages-sequential-convoy/create-service-bus-connection-2.png)

1. Sélectionnez **Continuer** lorsque vous avez terminé.

   Le Concepteur d’applications logiques affiche maintenant le modèle **Distribution corrélée dans l’ordre à l’aide de sessions Service Bus**, qui contient un flux de travail prérempli avec un déclencheur et des actions, y compris deux étendues qui implémentent une gestion des erreurs suivant le mode `Try-Catch`.

Vous pouvez maintenant vous renseigner sur le déclencheur et les actions du modèle ou [indiquer directement les valeurs du modèle d’application logique](#complete-template).

<a name="template-summary"></a>

## <a name="template-summary"></a>Résumé du modèle

Voici les grandes étapes du flux de travail du modèle **Distribution corrélée dans l’ordre à l’aide de sessions Service Bus** une fois les détails réduits :

![Grandes étapes du flux de travail du modèle](./media/send-related-messages-sequential-convoy/template-top-level-flow.png)

| Nom | Description |
|------|-------------|
| **`When a message is received in a queue (peek-lock)`** | Ce déclencheur Service Bus vérifie la présence de messages dans la file d’attente Service Bus spécifiée selon la périodicité indiquée. S’il existe un message dans la file d’attente, le déclencheur s’active, ce qui crée et exécute une instance de flux de travail. <p><p>Le terme *peek-lock* signifie que le déclencheur envoie une demande de récupération d’un message de la file d’attente. S’il existe un message, le déclencheur le récupère et le verrouille afin qu’aucun autre traitement n’y soit effectué avant expiration de la période de verrouillage. Pour plus d’informations, consultez [Initialisation de la session](#initialize-session). |
| **`Init isDone`** | Cette [action **Initialisation de variable**](../logic-apps/logic-apps-create-variables-store-values.md#initialize-variable) crée une variable booléenne dont la valeur est `false` et qui indique si les conditions suivantes sont vraies : <p><p>- Il n’y a plus aucun message à lire dans la session. <br>- Le verrou de session n’a plus besoin d’être renouvelé de sorte que l’instance de flux de travail actuelle peut se terminer. <p><p>Pour plus d’informations, consultez [Initialisation de la session](#initialize-session). |
| **`Try`** | Cette [action **Étendue**](../logic-apps/logic-apps-control-flow-run-steps-group-scopes.md) contient les actions qui s’exécutent pour traiter un message. Si un problème se produit dans l’étendue `Try`, c’est l’action **Portée** `Catch` suivante qui le gère. Pour plus d’informations, consultez [Étendue « Try »](#try-scope). |
| **`Catch`**| Cette [action **Étendue**](../logic-apps/logic-apps-control-flow-run-steps-group-scopes.md) contient les actions qui s’exécutent si un problème se produit dans l’étendue `Try` précédente. Pour plus d’informations, consultez [Étendue « Catch »](#catch-scope). |
|||

<a name="try-scope"></a>

### <a name="try-scope"></a>Étendue « Try »

Voici les grandes étapes du flux de [l’action d’étendue](../logic-apps/logic-apps-control-flow-run-steps-group-scopes.md) `Try` une fois les détails réduits :

![Flux de travail de l’action d’étendue « Try »](./media/send-related-messages-sequential-convoy/try-scope-action.png)

| Nom | Description |
|------|-------------|
| **`Send initial message to topic`** | Vous pouvez remplacer cette action par celle qui devra gérer le premier message de la session dans la file d’attente. L’ID de session spécifie la session. <p><p>Dans ce modèle, une action Service Bus envoie le premier message à une rubrique Service Bus. Pour plus d’informations, consultez [Gestion du message initial](#handle-initial-message). |
| (branche parallèle) | Cette [action branche parallèle](../logic-apps/logic-apps-control-flow-branches.md) crée deux chemins : <p><p>- Branch 1 : Poursuit le traitement du message. Pour plus d’informations, consultez [Branche 1 : Finaliser le message initial dans la file d’attente](#complete-initial-message). <p><p>- Branche 2 : Abandonne le message en cas de problème et le libère pour qu’il soit collecté par une autre exécution du déclencheur. Pour plus d’informations, consultez [Branche 2 : Abandonner le message initial de la file d’attente](#abandon-initial-message). <p><p>Les deux chemins se rejoignent plus tard dans l’action **Fermer une session dans une file d’attente et réussir**, décrite dans la ligne suivante. |
| **`Close a session in a queue and succeed`** | Cette action Service Bus joint les branches décrites précédemment et ferme la session dans la file d’attente après l’un des événements suivants : <p><p>- Le flux de travail termine le traitement des messages disponibles dans la file d’attente. <br>- Le flux de travail abandonne le message initial en raison d’un problème. <p><p>Pour plus d’informations, consultez [Fermer une session dans une file d’attente et réussir](#close-session-succeed). |
|||

<a name="complete-initial-message"></a>

#### <a name="branch-1-complete-initial-message-in-queue"></a>Branche 1 : Finaliser le message initial dans la file d’attente

| Nom | Description |
|------|-------------|
| `Complete initial message in queue` | Cette action Service Bus marque un message récupéré comme finalisé et le supprime de la file d’attente pour empêcher le retraitement. Pour plus d’informations, consultez [Gestion du message initial](#handle-initial-message). |
| `While there are more messages for the session in the queue` | Cette [boucle **Tant que**](../logic-apps/logic-apps-control-flow-loops.md#until-loop) continue à recevoir les messages tant qu’il en existe ou jusqu’à ce qu’une heure se soit écoulée. Pour plus d’informations sur les actions de cette boucle, consultez [Tant qu’il y a d’autres messages pour la session dans la file d’attente](#while-more-messages-for-session). |
| **`Set isDone = true`** | Lorsqu’il n’y a plus de messages, cette [action **Définir la variable**](../logic-apps/logic-apps-create-variables-store-values.md#set-variable) définit `isDone` sur `true`. |
| **`Renew session lock until cancelled`** | Cette [boucle **Tant que**](../logic-apps/logic-apps-control-flow-loops.md#until-loop) fait en sorte que le verrou de session soit maintenu par cette application logique tant qu’il existe des messages ou jusqu’à ce qu’une heure se soit écoulée. Pour plus d’informations sur les actions de cette boucle, consultez [Renouveler le verrou de session jusqu’à annulation](#renew-session-while-messages-exist). |
|||

<a name="abandon-initial-message"></a>

#### <a name="branch-2-abandon-initial-message-from-the-queue"></a>Branche 2 : Abandonner le message initial de la file d’attente

Si l’action qui gère le premier message échoue, l’action Service Bus **Abandonner le message initial de la file d’attente** le libère pour qu’il soit collecté et traité par une autre instance de flux de travail. Pour plus d’informations, consultez [Gestion du message initial](#handle-initial-message).

<a name="catch-scope"></a>

### <a name="catch-scope"></a>Étendue « catch »

L’application logique doit fermer la session même si les actions de l’étendue `Try` échouent. [L’action d’étendue](../logic-apps/logic-apps-control-flow-run-steps-group-scopes.md) `Catch` s’exécute lorsque l’action d’étendue `Try` entraîne l’état `Failed`, `Skipped` ou `TimedOut`. L’étendue retourne un message d’erreur qui comprend l’ID de session où le problème s’est produit et met fin à l’application logique.

Voici les grandes étapes du flux de l’action d’étendue `Catch` une fois les détails réduits :

![Flux de travail de l’action d’étendue « Catch »](./media/send-related-messages-sequential-convoy/catch-scope-action.png)

| Nom | Description |
|------|-------------|
| **`Close a session in a queue and fail`** | Cette action Service Bus ferme la session dans la file d’attente afin que le verrou de session ne reste pas ouvert. Pour plus d’informations, consultez [Fermer une session dans une file d’attente et échouer](#close-session-fail). |
| **`Find failure msg from 'Try' block`** | Cette [action **Filtrage de tableau**](../logic-apps/logic-apps-perform-data-operations.md#filter-array-action) crée un tableau à partir des entrées et des sorties de toutes les actions qui se trouvent dans l’étendue `Try` en fonction des critères spécifiés. Dans ce cas, elle retourne les sorties des actions qui ont abouti à l’état `Failed`. Pour plus d’informations, consultez [Trouver le message d’échec du bloc « Try »](#find-failure-message). |
| **`Select error details`** | Cette [action **Sélectionner**](../logic-apps/logic-apps-perform-data-operations.md#select-action) crée un tableau qui contient des objets JSON en fonction des critères spécifiés. Ces objets JSON sont construits à partir des valeurs du tableau créé par l’action précédente, `Find failure msg from 'Try' block`. Dans ce cas, cette action retourne un tableau qui contient un objet JSON créé à partir des détails de l’erreur retournés par l’action précédente. Pour plus d’informations, consultez [Sélectionner les détails de l’erreur](#select-error-details). |
| **`Terminate`** | Cette [action **Arrêt**](../logic-apps/logic-apps-workflow-actions-triggers.md#terminate-action) arrête l’exécution du flux de travail, annule toutes les actions en cours, ignore les actions restantes et retourne l’état spécifié, l’ID de session et le résultat d’erreur de l’action `Select error details`. Pour plus d’informations, consultez [Arrêt d’une application logique](#terminate-logic-app). |
|||

<a name="complete-template"></a>

## <a name="complete-the-template"></a>Finalisation du modèle

Pour indiquer les valeurs du déclencheur et des actions dans le modèle **Distribution corrélée dans l’ordre à l’aide de sessions Service Bus**, procédez de la façon suivante. Vous devez fournir toutes les valeurs requises, qui sont marquées d’un astérisque ( **\*** ), pour pouvoir enregistrer votre application logique.

<a name="initialize-session"></a>

### <a name="initialize-the-session"></a>Initialiser la session

* Pour le déclencheur **À la réception d’un message dans une file d’attente (peek-lock)** , indiquez ces informations pour que le modèle puisse initialiser une session à l’aide de la propriété **ID de session**, par exemple :

  ![Détails du déclencheur Service Bus pour « À la réception d’un message dans une file d’attente (peek-lock) »](./media/send-related-messages-sequential-convoy/service-bus-check-message-peek-lock-trigger.png)

  > [!NOTE]
  > Initialement, la fréquence d’interrogation est définie sur trois minutes afin que l’application logique ne s’exécute pas plus souvent que prévu au risque d’entraîner des frais de facturation non anticipés. Dans l’idéal, définissez l’intervalle et la fréquence sur 30 secondes pour que l’application logique se déclenche immédiatement lorsqu’un message arrive.

  | Propriété | Obligatoire pour ce scénario | Valeur | Description |
  |----------|----------------------------|-------|-------------|
  | **Nom de la file d’attente** | Oui | <*nom-file d’attente*> | Nom de la file d’attente Service Bus créée, « Fabrikam-Service-Bus-Queue » dans cet exemple. |
  | **Type de file d'attente** | Oui | **Main** | File d’attente Service Bus primaire. |
  | **ID de session** | Oui | **Suivant disponible** | Cette option permet d’obtenir une session pour chaque exécution de déclencheur, en fonction de l’ID de session du message dans la file d’attente Service Bus. La session est également verrouillée, de sorte qu’aucune autre application logique ni aucun autre client ne peut traiter les messages liés à cette session. Les actions suivantes du flux de travail traitent tous les messages associés à cette session (cf. suite de cet article). <p><p>Voici quelques informations complémentaires sur les autres options **ID de session** : <p>- **Aucun** : option par défaut, qui n’aboutit à aucune session et ne peut pas être utilisée pour implémenter le mode de convoi séquentiel. <p>- **Entrer une valeur personnalisée** : choisissez cette option si vous connaissez l’ID de session à utiliser et que vous souhaitez toujours exécuter le déclencheur pour cet ID de session. <p>**Remarque** : Le connecteur Service Bus peut enregistrer un nombre limité de sessions uniques à la fois entre Azure Service Bus et le cache du connecteur. Si le nombre de sessions dépasse cette limite, les anciennes sessions sont supprimées du cache. Pour plus d’informations, consulter [Échange de messages dans le cloud avec Azure Logic Apps et Azure Service Bus](../connectors/connectors-create-api-servicebus.md#connector-reference). |
  | **Intervalle** | Oui | <*nombre-intervalles*> | Nombre d’unités de temps de la périodicité de consultation des messages. |
  | **Fréquence** | Oui | **Seconde**, **Minute**, **Heure**, **Jour**, **Semaine** ou **Mois** | Unité de temps de la périodicité à appliquer pour la consultation des messages. <p>**Conseil** : Pour ajouter un **Fuseau horaire** ou une **Heure de début**, sélectionnez ces propriétés dans la liste **Ajouter un nouveau paramètre**. |
  |||||

  Pour plus d’informations sur les déclencheurs, consultez [Service Bus – À la réception d’un message dans une file d’attente (peek-lock)](/connectors/servicebus/#when-a-message-is-received-in-a-queue-(peek-lock)). Le déclencheur génère un [ServiceBusMessage](/connectors/servicebus/#servicebusmessage).

Après l’initialisation de la session, le flux de travail utilise l’action **Initialisation de variable** pour créer une variable booléenne dont la valeur initiale est `false` et qui indique si les conditions suivantes sont remplies : 

* Il n’y a plus aucun message à lire dans la session.

* Le verrou de session n’a plus besoin d’être renouvelé de sorte que l’instance de flux de travail actuelle peut se terminer.

![Détails de l’action « Initialisation de variable » pour « Init isDone »](./media/send-related-messages-sequential-convoy/init-is-done-variable.png)

Ensuite, dans le bloc **Try**, le flux de travail effectue des actions sur le premier message lu.

<a name="handle-initial-message"></a>

### <a name="handle-the-initial-message"></a>Gestion du message initial

La première action est un espace réservé à une action Service Bus, **Envoyer le message initial à la rubrique**, que vous pouvez remplacer par celle qui devra gérer le premier message de la session dans la file d’attente. L’ID de session spécifie la session à l’origine du message.

L’espace réservé à une action Service Bus envoie le premier message à une rubrique Service Bus spécifiée par la propriété **ID de session**. Ainsi, tous les messages associés à une session spécifique sont dirigés vers la même rubrique. Toutes les propriétés **ID de session** des actions suivantes de ce modèle utilisent la même valeur d’ID de session.

![Détails de l’action Service Bus « Envoyer le message initial à la rubrique »](./media/send-related-messages-sequential-convoy/send-initial-message-to-topic-action.png)

1. Dans l’action Service Bus **Finaliser le message initial dans la file d’attente**, indiquez le nom de votre file d’attente Service Bus, puis conservez la valeur par défaut de toutes les autres propriétés de l’action.

   ![Détails de l’action Service Bus « Finaliser le message initial dans la file d’attente »](./media/send-related-messages-sequential-convoy/complete-initial-message-queue.png)

1. Dans l’action Service Bus **Abandonner le message initial de la file d’attente**, indiquez le nom de votre file d’attente Service Bus, puis conservez la valeur par défaut de toutes les autres propriétés de l’action.

   ![Détails de l’action Service Bus « Abandonner le message initial de la file d’attente »](./media/send-related-messages-sequential-convoy/abandon-initial-message-from-queue.png)

Vous allez maintenant indiquer les informations nécessaires pour les actions qui suivent **Finaliser le message initial dans la file d’attente**. Commencez par les actions de la boucle **Tant qu’il y a d’autres messages pour la session dans la file d’attente**.

<a name="while-more-messages-for-session"></a>

### <a name="while-there-are-more-messages-for-the-session-in-the-queue"></a>Tant qu’il y a d’autres messages pour la session dans la file d’attente

Cette [boucle **Tant que**](../logic-apps/logic-apps-control-flow-loops.md#until-loop) exécute ces actions tant qu’il existe des messages dans la file d’attente ou jusqu’à ce qu’une heure se soit écoulée. Pour modifier la limite de temps de la boucle, changez la valeur de la propriété **Délai d’expiration** de la boucle.

* Récupérer des messages supplémentaires de la file d’attente tant qu’il existe des messages.

* Vérifier le nombre de messages restants. S’il existe toujours des messages, poursuivre le traitement des messages. S’il n’y a plus de messages, le flux de travail donne à la variable `isDone` la valeur `true`et ferme la boucle.

![Boucle Tant que – Traiter les messages tant qu’il y en a dans la file d’attente](./media/send-related-messages-sequential-convoy/while-more-messages-for-session-in-queue.png)

1. Dans l’action Service Bus **Récupérer des messages supplémentaires de la session**, indiquez le nom de votre file d’attente Service Bus. Sinon, conservez la valeur par défaut de toutes les autres propriétés de l’action.

   > [!NOTE]
   > Par défaut, le nombre maximal de messages est défini sur `175`, mais cette limite est affectée par les propriétés de taille et de taille maximale du message dans Service Bus. Pour plus d’informations, consultez les détails relatifs à la [taille d’un message en file d’attente](../service-bus-messaging/service-bus-quotas.md).

   ![Action Service Bus – « Récupérer des messages supplémentaires de la session »](./media/send-related-messages-sequential-convoy/get-additional-messages-from-session.png)

   Ensuite, le flux de travail se divise en deux branches parallèles :

   * Si une erreur ou un échec se produit lors de la recherche de messages supplémentaires, donner à la variable `isDone` la valeur `true`.

   * La condition **Traiter les messages s’il y en a** vérifie si le nombre de messages restants est égal à zéro. Si la valeur est false et qu’il y a d’autres messages, continuer le traitement. Si la valeur est true et qu’il n’y a plus de messages, le flux de travail donne à la variable `isDone` la valeur `true`.

   ![Condition – Traiter les messages s’il y en a](./media/send-related-messages-sequential-convoy/process-messages-if-any.png)

   Dans la section **Si la valeur est false**, une boucle **Pour chaque** traite chaque message dans l’ordre FIFO (premier entré, premier sorti). Dans les **Paramètres** de la boucle, le paramètre **Contrôle de concurrence** a la valeur `1`, de sorte qu’un seul message est traité à la fois.

   ![Boucle « Pour chaque » – Traiter les messages un par un](./media/send-related-messages-sequential-convoy/for-each-additional-message.png)

1. Pour les actions Service Bus **Finaliser le message dans une file d’attente** et **Abandonner le message dans une file d’attente**, indiquez le nom de votre file d’attente Service Bus.

   ![Actions Service Bus – « Terminer le message dans une file d’attente » et « Abandonner le message dans une file d’attente »](./media/send-related-messages-sequential-convoy/abandon-or-complete-message-in-queue.png)

   Une fois **Tant qu’il y a d’autres messages pour la session dans la file d’attente** terminé, le flux de travail donne à la variable `isDone` la valeur `true`.

Vous allez maintenant indiquer les informations nécessaires pour les actions de la boucle **Renouveler le verrou de session jusqu’à annulation**.

<a name="renew-session-while-messages-exist"></a>

### <a name="renew-session-lock-until-cancelled"></a>Renouveler le verrou de session jusqu’à annulation

Cette [boucle **Tant que**](../logic-apps/logic-apps-control-flow-loops.md#until-loop) fait en sorte que le verrou de session soit maintenu par cette application logique tant qu’il existe des messages dans la file d’attente ou jusqu’à ce qu’une heure se soit écoulée en effectuant ces actions. Pour modifier la limite de temps de la boucle, changez la valeur de la propriété **Délai d’expiration** de la boucle.

* Fixer un délai de 25 secondes ou d’une durée inférieure à celle du 	dépassement de délai d’attente de verrou pour la file d’attente en cours de traitement. La plus petite durée de verrouillage étant de 30 secondes, la valeur par défaut est suffisante. Toutefois, vous pouvez optimiser le nombre d’exécutions de la boucle en l’ajustant selon vos besoins.

* Vérifier si la variable `isDone` a la valeur `true`.

  * Si `isDone` n’a pas la valeur `true`, le flux de travail traite toujours les messages, de sorte qu’il renouvelle le verrou sur la session dans la file d’attente et vérifie à nouveau la condition de la boucle.

    Vous devez indiquer le nom de votre file d’attente Service Bus dans l’action Service Bus [**Renouveler le verrou sur la session dans une file d’attente**](#renew-lock-on-session).

  * Si `isDone` a la valeur `true`, le flux de travail ne renouvelle pas le verrou sur la session dans la file d’attente et quitte la boucle.

  ![Boucle Tant que – « Renouveler le verrou de session jusqu’à annulation »](./media/send-related-messages-sequential-convoy/renew-lock-until-session-cancelled.png)

<a name="renew-lock-on-session"></a>

#### <a name="renew-lock-on-the-session-in-a-queue"></a>Renouveler le verrou sur la session dans une file d’attente

Cette action Service Bus renouvelle le verrou sur la session dans la file d’attente tant que le flux de travail traite des messages.

* Dans l’action Service Bus **Renouveler le verrou sur la session dans une file d’attente**, indiquez le nom de votre file d’attente Service Bus.

  ![Action Service Bus – « Renouveler le verrou sur la session dans une file d’attente »](./media/send-related-messages-sequential-convoy/renew-lock-on-session-in-queue.png)

Vous allez maintenant indiquer les informations nécessaires pour l’action Service Bus **Fermer une session dans une file d’attente et réussir**.

<a name="close-session-succeed"></a>

### <a name="close-a-session-in-a-queue-and-succeed"></a>Fermer une session dans une file d’attente et réussir

Cette action Service Bus ferme la session dans la file d’attente une fois que le flux de travail a terminé le traitement de tous les messages disponibles dans la file d’attente ou qu’il a abandonné le message initial.

* Dans l’action Service Bus **Fermer une session dans une file d’attente et réussir**, indiquez le nom de votre file d’attente Service Bus.

  ![Action Service Bus – « Fermer une session dans une file d’attente et réussir »](./media/send-related-messages-sequential-convoy/close-session-in-queue-succeed.png)

Les sections suivantes décrivent les actions de la section `Catch`, chargées de gérer les erreurs et les exceptions qui se produisent dans le flux de travail.

<a name="close-session-fail"></a>

### <a name="close-a-session-in-a-queue-and-fail"></a>Fermer une session dans une file d’attente et échouer

Cette action Service Bus est toujours la première à s’exécuter dans l’étendue `Catch` et ferme la session dans la file d’attente.

* Dans l’action Service Bus **Fermer une session dans une file d’attente et échouer**, indiquez le nom de votre file d’attente Service Bus.

  ![Action Service Bus – « Fermer une session dans une file d’attente et échouer »](./media/send-related-messages-sequential-convoy/close-session-in-queue-fail.png)

Ensuite, le flux de travail crée un tableau qui contient les entrées et les sorties de toutes les actions de l’étendue `Try`, afin que l’application logique puisse accéder aux informations sur l’erreur ou l’échec qui s’est produit.

<a name="find-failure-message"></a>

### <a name="find-failure-msg-from-try-block"></a>Trouver le message d’échec du bloc « Try »

Cette [action **Filtrage de tableau**](../logic-apps/logic-apps-perform-data-operations.md#filter-array-action) crée un tableau comportant les entrées et les sorties de toutes les actions qui se trouvent dans l’étendue `Try` en fonction des critères spécifiés à l’aide de la [fonction `result()`](../logic-apps/workflow-definition-language-functions-reference.md#result). Dans ce cas, cette action retourne les sorties des actions dont l’état est `Failed` à l’aide de la [fonction `equals()`](../logic-apps/workflow-definition-language-functions-reference.md#equals) et de la [fonction `item()`](../logic-apps/workflow-definition-language-functions-reference.md#item).

![Action Filtrage de tableau – « Trouver le message d’échec du bloc « Try » »](./media/send-related-messages-sequential-convoy/find-failure-message.png)

Voici la définition JSON de cette action :

```json
"Find_failure_msg_from_'Try'_block": {
   "inputs": {
      "from": "@Result('Try')",
      "where": "@equals(item()['status'], 'Failed')"
   },
   "runAfter": {
      "Close_the_session_in_the_queue_and_fail": [
         "Succeeded"
      ]
   },
   "type": "Query"
},
```

Ensuite, le flux de travail crée un tableau avec un objet JSON qui contient les informations d’erreur dans le tableau retourné à partir de l’action `Find failure msg from 'Try' block`.

<a name="select-error-details"></a>

### <a name="select-error-details"></a>Sélection des détails de l’erreur

Cette [action **Sélection**](../logic-apps/logic-apps-perform-data-operations.md#select-action) crée un tableau contenant des objets JSON en fonction du tableau d’entrée obtenu en sortie de l’action précédente, `Find failure msg from 'Try' block`. Plus précisément, cette action retourne un tableau qui contient uniquement les propriétés spécifiées pour chacun des objets du tableau. Dans ce cas, le tableau comporte le nom de l’action et les propriétés des résultats d’erreur.

![Action Sélection – « Sélectionner les détails de l’erreur »](./media/send-related-messages-sequential-convoy/select-error-details.png)

Voici la définition JSON de cette action :

```json
"Select_error_details": {
   "inputs": {
      "from": "@body('Find_failure_msg_from_''Try''_block')[0]['outputs']",
      "select": {
         "action": "@item()['name']",
         "errorResult": "@item()"
      }
   },
   "runAfter": {
      "Find_failure_msg_from_'Try'_block": [
         "Succeeded"
      ]
   },
   "type": "Select"
},
```

Ensuite, le flux de travail arrête l’exécution de l’application logique et retourne l’état d’exécution avec d’autres informations sur l’erreur ou l’échec qui s’est produit.

<a name="terminate-logic-app"></a>

### <a name="terminate-logic-app-run"></a>Arrêt de l’exécution de l’application logique

Cette [action **Arrêt**](../logic-apps/logic-apps-workflow-actions-triggers.md#terminate-action) arrête l’exécution de l’application logique et retourne `Failed` comme état de l’exécution de l’application logique, ainsi que l’ID de session et le résultat d’erreur de l’action `Select error details`.

![Action Arrêt pour arrêter l’exécution de l’application logique](./media/send-related-messages-sequential-convoy/terminate-logic-app-run.png)

Voici la définition JSON de cette action :

```json
"Terminate": {
   "description": "This Failure Termination only runs if the Close Session upon Failure action runs - otherwise the LA will be terminated as Success",
   "inputs": {
      "runError": {
         "code": "",
         "message": "There was an error processing messages for Session ID @{triggerBody()?['SessionId']}. The following error(s) occurred: @{body('Select_error_details')['errorResult']}"
         },
         "runStatus": "Failed"
      },
      "runAfter": {
         "Select_error_details": [
            "Succeeded"
         ]
      },
      "type": "Terminate"
   }
},
```

## <a name="save-and-run-logic-app"></a>Enregistrement et exécution de l’application logique

Maintenant que le modèle est finalisé, vous pouvez enregistrer votre application logique. Dans la barre d’outils du Concepteur, sélectionnez **Enregistrer**.

Pour tester votre application logique, envoyez des messages à votre file d’attente Service Bus. 

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les [déclencheurs et actions du connecteur Service Bus](/connectors/servicebus/)
