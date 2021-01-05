---
title: Traiter par lots les messages en tant que groupe
description: Envoyer et recevoir des messages dans des groupes entre vos workflows à l’aide d’un traitement par lots dans Azure Logic Apps
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: estfan, jonfan, logicappspm
ms.topic: article
ms.date: 07/31/2020
ms.openlocfilehash: 0985afe3ddfd0d9de3c36ad6b030b6f259708c88
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87458257"
---
# <a name="send-receive-and-batch-process-messages-in-azure-logic-apps"></a>Envoyer, recevoir et traiter par lots des messages dans Azure Logic Apps

Pour envoyer et traiter des messages ensemble de façon spécifique en tant que groupes, vous pouvez créer une solution de traitement par lot. Cette solution collecte des messages dans un *lot* et attend que les critères spécifiés soient remplis avant de mettre en production et de traiter les messages traités par lot. Le traitement par lots peut réduire la fréquence à laquelle votre application logique traite les messages.

Cet article montre comment concevoir une solution de traitement par lot en créant deux applications logiques dans le même abonnement, dans la même région Azure et dans cet ordre :

1. L’application logique [« réceptrice de lots »](#batch-receiver), qui accepte et collecte les messages dans un lot jusqu’à ce que vos critères précisés soient remplis pour mettre en production et traiter ces messages. Veillez à commencer par la création du récepteur de lots, afin de pouvoir sélectionner ultérieurement la destination des lots lors de la création de l’expéditeur de lots.

1. Une ou plusieurs applications logiques [« expéditrices de lots »](#batch-sender), qui envoient les messages au récepteur de lots précédemment créé.

   Vous pouvez également spécifier une clé unique (telle qu’un numéro de client), qui *partitionne* ou divise en sous-ensembles logiques le lot cible, en fonction de cette clé. De cette façon, l’application réceptrice peut collecter tous les éléments avec la même clé et les traiter ensemble.

Votre récepteur et votre expéditeur de lots doivent partager le même abonnement *et* la même région Azure. Si ce n’est pas le cas, vous ne pouvez pas sélectionner le récepteur de lots lorsque vous créez l’expéditeur de lots, car ils ne sont pas visibles entre eux.

## <a name="prerequisites"></a>Prérequis

* Un compte et un abonnement Azure. Si vous ne disposez d’aucun abonnement, vous pouvez [commencer par créer gratuitement un compte Azure](https://azure.microsoft.com/free/). Vous pouvez aussi [souscrire un abonnement avec paiement à l’utilisation](https://azure.microsoft.com/pricing/purchase-options/).

* Un compte de courrier de n’importe quel [fournisseur de messagerie pris en charge par Azure Logic Apps](../connectors/apis-list.md)

  > [!IMPORTANT]
  > Si vous souhaitez utiliser le connecteur Gmail, seuls les comptes professionnels G-Suite peuvent utiliser ce connecteur sans restriction dans les applications logiques. Si vous disposez d’un compte de consommateur Gmail, vous pouvez utiliser ce connecteur uniquement avec certains services approuvés par Google, ou vous pouvez [créer une application cliente Google pour servir lors de l’authentification avec votre connecteur Gmail](/connectors/gmail/#authentication-and-bring-your-own-application). Pour plus d’informations, consultez [Stratégies de confidentialité et de sécurité des données pour les connecteurs Google dans Azure Logic Apps](../connectors/connectors-google-data-security-privacy-policy.md).

* Des connaissances de base en [création d’applications logiques](../logic-apps/quickstart-create-first-logic-app-workflow.md)

* Pour utiliser Visual Studio plutôt que le portail Azure, veillez à [configurer Visual Studio pour l’utilisation de Logic Apps](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md).

<a name="batch-receiver"></a>

## <a name="create-batch-receiver"></a>Créer le récepteur de lots

Avant de pouvoir envoyer des messages à un lot, ce lot doit d’abord exister en tant que destination vers laquelle envoyer ces messages. Ainsi, vous devez commencer par créer l’application logique « réceptrice de lots », qui démarre avec le déclencheur **Lot**. De cette façon, lorsque vous créez l’application logique « expéditrice de lots », vous pouvez sélectionner l’application logique réceptrice de lots. Le récepteur de lots continue à collecter les messages jusqu’à ce que vos critères spécifiés soient remplis pour mettre en production et traiter ces messages. Alors que les récepteurs de lots n’ont pas besoin de connaître les expéditeurs de lots, les expéditeurs de lots, eux, doivent connaître la destination vers laquelle envoyer les messages.

1. Dans le [portail Azure](https://portal.azure.com) ou dans Visual Studio, créez une application logique et nommez-la ainsi : `BatchReceiver`.

1. Dans le concepteur d’applications logiques, ajoutez le déclencheur **Lot** qui démarre le workflow de votre application logique. Dans la zone de recherche, entrez `batch`, puis sélectionnez ce déclencheur : **Traiter les messages par lots**

   ![Ajout d’un déclencheur « Traiter les messages par lots »](./media/logic-apps-batch-process-send-receive-messages/add-batch-receiver-trigger.png)

1. Définissez les propriétés suivantes pour le récepteur de lots :

   | Propriété | Description |
   |----------|-------------|
   | **Mode Batch** | - **Inline** : Pour définir des critères de mise en production à l’intérieur du déclencheur de lots <br>- **Compte d’intégration** : Pour définir plusieurs configurations de critères de mise en production via un [compte d’intégration](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md). Avec un compte d’intégration, vous pouvez conserver ces configurations toutes au même endroit, plutôt que dans des applications logiques distincts. |
   | **Nom du lot** | Le nom de votre lot, qui est « TestBatch » dans cet exemple, et qui s’applique uniquement au mode batch **Inline** |
   | **Critères de mise en production** | S’applique seulement au mode batch **Inline** et sélectionne les critères à respecter avant de traiter chaque lot : <p>- **Sur la base du nombre de messages** : Mettez en production le lot en fonction du nombre de messages qu’il a collectés. <br>- **En fonction de la taille** : Mettez en production le lot en fonction de la taille totale (en octets) de tous les messages qu’il a collectés. <br>- **Planification** : Mettez en production le lot selon une planification de récurrence, avec un intervalle et une fréquence. Dans les options avancées, vous pouvez également sélectionner un fuseau horaire, et fournir une date et une heure de début. <br>- **Tout sélectionner** : Utilisez tous les critères spécifiés. |
   | **Nombre de messages** | Nombre de messages à collecter dans le lot, par exemple 10 messages. Pour un lot, la limite est de 8 000 messages. |
   | **Taille de lot** | Taille totale à collecter dans le lot (en octets), par exemple, 10 Mo. La taille limite d’un lot est de 80 Mo. |
   | **Planification** | Intervalle et fréquence entre les mises en production de lots, par exemple 10 minutes. La périodicité minimale est de 60 secondes ou 1 minute. Les fractions de minutes sont arrondies à 1 minute. Pour spécifier un fuseau horaire ou une date et une heure de début, ouvrez la liste **Ajouter un nouveau paramètre**, puis sélectionnez les propriétés correspondantes. |
   |||

   > [!NOTE]
   >
   > Si vous modifiez les critères de mise en production alors que le déclencheur a toujours des messages traités par lot mais non envoyés, celui-ci utilise les nouveaux critères de mise en production pour gérer les messages non envoyés.

   Cet exemple montre tous les critères. Toutefois, pour vos propres tests, vous pouvez n’utiliser qu’un seul critère :

   ![Détails à fournir concernant le déclencheur Lot](./media/logic-apps-batch-process-send-receive-messages/batch-receiver-criteria.png)

1. Ajoutez maintenant une ou plusieurs actions qui traitent chaque lot.

   Pour cet exemple, ajoutez une action qui envoie un e-mail lorsque le déclencheur de lot est activé. Le déclencheur s’exécute et envoie un e-mail lorsque le lot contient 10 messages, atteint 10 Mo, ou au bout de 10 minutes.

   1. Sous le déclencheur de lot, sélectionnez **Nouvelle étape**.

   1. Dans la zone de recherche, entrez `send email` en guise de filtre. Sélectionnez un connecteur de messagerie en fonction de votre fournisseur de messagerie.

      Par exemple, si vous avez un compte professionnel ou scolaire, comme @fabrikam.com ou @fabrikam.onmicrosoft.com, sélectionnez le connecteur **Microsoft 365 Outlook**. Si vous avez un compte personnel, tel que @outlook.com ou @hotmail.com, sélectionnez le connecteur **Outlook.com**. Cet exemple utilise le connecteur Microsoft 365 Outlook.

   1. Sélectionnez l’action « Envoyer un e-mail » pour votre fournisseur, par exemple :

      ![Sélection de l’action « Envoyer un message électronique » pour votre fournisseur de messagerie](./media/logic-apps-batch-process-send-receive-messages/batch-receiver-send-email-action.png)

1. Si vous y êtes invité, connectez-vous à votre compte e-mail.

1. Définissez les propriétés de l’action que vous avez ajoutée.

   * Dans la zone **À**, entrez l’adresse e-mail du destinataire. À des fins de test, vous pouvez utiliser votre propre adresse e-mail.

   * Dans la zone **Objet**, lorsque la liste de contenu dynamique s’affiche, sélectionnez le champ **Nom de partition**.

     ![Dans la liste de contenu dynamique, sélection du « Nom de partition »](./media/logic-apps-batch-process-send-receive-messages/send-email-action-details.png)

     Plus tard, dans l’expéditeur de lots, vous pouvez spécifier une clé de partition unique, afin de diviser le lot cible en plusieurs sous-ensembles logiques vers lesquels envoyer des messages. Chaque ensemble est associé à un numéro unique qui est généré par l’application logique expéditrice de lots. Cette fonctionnalité permet d’utiliser un lot comprenant plusieurs sous-ensembles et de définir chaque sous-ensemble avec le nom que vous fournissez.

     > [!IMPORTANT]
     > Une partition a une limite de 5 000 messages ou 80 Mo. Si l’une ou l’autre des conditions est remplie, Logic Apps peut mettre en production le lot, même lorsque votre condition de mise en production définie n’est pas remplie.

   * Dans la zone **Corps**, lorsque la liste de contenu dynamique s’affiche, sélectionnez le champ **ID de message**.

     Le concepteur d’applications logiques ajoute automatiquement une boucle **Pour tous** autour de l’action d’envoi d’e-mail, car cette action traite la sortie de l’action précédente comme une collection plutôt que comme un lot.

     ![Pour « Corps », sélectionnez « ID de message »](./media/logic-apps-batch-process-send-receive-messages/send-email-action-details-for-each.png)

1. Enregistrez votre application logique. Vous venez de créer un récepteur de lots.

    ![Enregistrer votre application logique](./media/logic-apps-batch-process-send-receive-messages/save-batch-receiver-logic-app.png)

   > [!IMPORTANT]
   > Si vous utilisez Visual Studio, avant de passer à la section suivante, assurez-vous d’abord de [*déployer* votre application logique de récepteur de lots sur Azure](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md#deploy-logic-app-to-azure). Sinon, vous ne pouvez pas sélectionner le récepteur de lots lorsque vous créez l’expéditeur de lots.

<a name="batch-sender"></a>

## <a name="create-batch-sender"></a>Créer l’expéditeur de lots

À présent, créez une ou plusieurs applications logiques expéditrices de lots qui envoient des messages vers l’application logique réceptrice de lots. Dans chaque expéditeur de lots, vous spécifiez le récepteur de lots avec le nom du lot, le contenu du message et tout autre paramètre nécessaire. Vous pouvez éventuellement fournir une clé de partition unique pour diviser le lot en sous-ensembles logiques pour la collecte des messages avec cette clé.

* En vous assurant d’avoir [créé et déployé votre récepteur de lots](#batch-receiver) avant de créer votre expéditeur de lots, vous pouvez sélectionner le récepteur existant comme destinataire de lots. Alors que les récepteurs de lots n’ont pas besoin de connaître les expéditeurs de lots, les expéditeurs de lots, eux, doivent savoir où envoyer les messages.

* Vérifiez que vos récepteur et expéditeur de lots partagent la même région *et* le même abonnement Azure. Si ce n’est pas le cas, vous ne pouvez pas sélectionner le récepteur de lots lorsque vous créez l’expéditeur de lots, car ils ne sont pas visibles entre eux.

1. Créez une autre application logique portant le nom suivant : `BatchSender`.

   1. Dans la zone de recherche, entrez `recurrence` en guise de filtre. Dans la liste des déclencheurs, sélectionnez ce déclencheur : **Périodicité**

      ![Ajouter le déclencheur de périodicité](./media/logic-apps-batch-process-send-receive-messages/add-schedule-trigger-batch-sender.png)

   1. Définissez l’intervalle et la fréquence d’exécution de l’application logique expéditrice sur une minute.

      ![Définition de la fréquence et de l’intervalle pour le déclencheur Périodicité](./media/logic-apps-batch-process-send-receive-messages/recurrence-trigger-batch-sender-details.png)

1. Ajoutez une nouvelle étape pour envoyer des messages vers un lot.

   1. Sous le déclencheur **Périodicité**, choisissez **Nouvelle étape**.

   1. Dans la zone de recherche, entrez `batch` comme filtre, puis sélectionnez cette action : **Choisir un workflow Logic Apps avec déclencheur de lot**

      ![Sélection de « Choisir un workflow Logic Apps avec déclencheur de lot »](./media/logic-apps-batch-process-send-receive-messages/send-messages-batch-action.png)

      Une liste apparaît et affiche uniquement les applications logiques qui ont des déclencheurs de lot et qui existent dans la même région Azure *et* dans le même abonnement Azure que votre application logique d’expéditeur de lots.

   1. Dans la liste des applications logiques, sélectionnez l’application logique réceptrice de lots que vous avez créée précédemment.

      ![Sélectionner votre application logique réceptrice de lots](./media/logic-apps-batch-process-send-receive-messages/batch-sender-select-batch-receiver.png)

      > [!IMPORTANT]
      > Si vous utilisez Visual Studio et que vous ne voyez aucun récepteur de lot à sélectionner, vérifiez que vous avez déjà créé et déployé votre récepteur de lots sur Azure. Si ce n’est pas le cas, découvrez comment [déployer votre application logique réceptrice de lots sur Azure](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md#deploy-logic-app-to-azure).

   1. Dans la liste des actions, sélectionnez cette action : **Batch_messages – <*nom-de-votre-application-logique*>**

      ![Sélectionnez cette action : "Traiter_les_messages_par_lots - <votre-application-logique>"](./media/logic-apps-batch-process-send-receive-messages/batch-sender-select-batch.png)

1. Définissez les propriétés de l’expéditeur de lots :

   | Propriété | Description |
   |----------|-------------|
   | **Nom du lot** | Nom du lot défini par l’application logique réceptrice (`TestBatch` dans cet exemple) <p>**Important !** Le nom du lot est validé lors de l’exécution et doit correspondre au nom spécifié par l’application logique réceptrice. Si vous modifiez ce nom, l’expéditeur de lots échoue. |
   | **Contenu du message** | Contenu du message que vous voulez envoyer |
   |||

   > [!NOTE]
   > Les valeurs de propriété **Nom du déclencheur** et **Workflow** sont renseignées automatiquement à partir de l’application logique sélectionnée.

   Pour cet exemple, ajoutez l’expression suivante qui insère la date et l’heure actuelles dans le contenu du message que vous envoyez au lot :

   1. Cliquez dans la zone **Contenu du Message**.

   1. Lorsque la liste de contenu dynamique s’affiche, sélectionnez **Expression**.

   1. Entrez l’expression `utcnow()`, puis sélectionnez **OK**.

      ![Dans « Contenu du message », sélectionnez « Expression », entrez « utcnow() » et sélectionnez « OK ».](./media/logic-apps-batch-process-send-receive-messages/batch-sender-details.png)

1. À présent, définissez une partition pour le lot. Dans l’action `BatchReceiver`, ouvrez la liste **Ajouter un nouveau paramètre**, puis sélectionnez les propriétés suivantes :

   | Propriété | Description |
   |----------|-------------|
   | **Nom de partition** | Clé de partition unique facultative, à utiliser pour diviser le lot cible en sous-ensembles logiques et collecter les messages en fonction de cette clé |
   | **ID de message** | Identificateur de message facultatif, qui est un identificateur global unique généré (GUID) lorsqu’il est vide |
   |||

   Pour cet exemple, dans la zone **Nom de partition**, ajoutez une expression qui génère un nombre aléatoire compris entre 1 et 5. Laissez la zone **ID de message** vide.

   1. Cliquez dans la zone **Nom de partition** pour afficher la liste de contenu dynamique.

   1. Dans la liste à contenu dynamique, sélectionnez **Expression**.

   1. Entrez l’expression `rand(1,6)`, puis sélectionnez **OK**.

      ![Définition d’une partition pour le lot cible](./media/logic-apps-batch-process-send-receive-messages/batch-sender-partition-advanced-options.png)

      Cette fonction **rand** génère un nombre compris entre 1 et 5. Vous divisez donc ce lot en cinq partitions numérotées, définies dynamiquement par cette expression.

1. Enregistrez votre application logique. Votre application logique expéditrice doit désormais ressembler à ceci :

   ![Enregistrement de l’application logique](./media/logic-apps-batch-process-send-receive-messages/batch-sender-finished.png)

## <a name="test-your-logic-apps"></a>Tester les applications logiques

Pour tester votre solution de traitement par lots, laissez vos applications logiques s’exécuter pendant quelques minutes. Vous allez commencer à recevoir des e-mails par groupes de cinq, tous avec la même clé de partition.

Votre application logique expéditrice de lots s’exécute toutes les minutes, elle génère un nombre aléatoire compris entre 1 et 5 et utilise ce numéro généré comme clé de partition pour le lot cible où sont envoyés les messages. Chaque fois que le lot comporte cinq éléments avec la même clé de partition, l’application logique réceptrice de lots se déclenche et envoie un e-mail pour chaque message.

> [!IMPORTANT]
> Lorsque vous avez terminé vos tests, veillez à désactiver l’application logique `BatchSender` afin d’arrêter l’envoi de messages et éviter la surcharge de la boîte de réception.

## <a name="next-steps"></a>Étapes suivantes

* [Mettre en lot des messages EDI et les envoyer](../logic-apps/logic-apps-scenario-edi-send-batch-messages.md)
* [Créer des définitions d’application logique à l’aide de JSON](../logic-apps/logic-apps-author-definitions.md)
* [Créer une application sans serveur dans Visual Studio avec Azure Logic Apps et Azure Functions](../logic-apps/logic-apps-serverless-get-started-vs.md)
* [Gestion des exceptions et journalisation des erreurs pour les applications logiques](../logic-apps/logic-apps-scenario-error-and-exception-handling.md)
