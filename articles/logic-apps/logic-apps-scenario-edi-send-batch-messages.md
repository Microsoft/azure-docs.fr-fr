---
title: Traiter par lots les messages EDI en tant que groupe
description: Envoyer et recevoir des messages EDI sous forme de lots, de groupes ou de collections à l’aide du traitement par lots dans Azure Logic Apps
services: logic-apps
author: divyaswarnkar
ms.author: divswa
ms.reviewer: estfan, logicappspm
ms.topic: article
ms.date: 08/19/2018
ms.openlocfilehash: 6fc0833f70e3e9cd98100f193b52e5a1bfa4d651
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96000400"
---
# <a name="exchange-edi-messages-as-batches-or-groups-between-trading-partners-in-azure-logic-apps"></a>Échanger des messages EDI sous forme de lots ou de groupes entre partenaires commerciaux dans Azure Logic Apps

Dans les scénarios d’entreprise à entreprise (B2B), les partenaires échangent souvent des messages par groupes ou par *lots*. Lorsque vous générez une solution de traitement par lot avec Logic Apps, vous pouvez envoyer des messages aux partenaires commerciaux et traiter ces messages par lot. Cet article explique comment traiter des messages EDI par lot (sur la base de l’exemple X12) en créant des applications logiques d’envoi et de réception par lot. 

Le traitement par lot des messages X12 fonctionne de la même manière que le traitement par lot d’autres messages. Vous devez utiliser un déclencheur de lot qui collecte les messages dans un lot et une action de traitement par lot qui envoie des messages au lot. En outre, le traitement par lot X12 prévoit une étape d’encodage avant d’envoyer les messages au partenaire commercial ou à une autre destination. Pour plus d’informations sur le déclencheur et l’opération de traitement par lots, consultez [Traiter par lots des messages](../logic-apps/logic-apps-batch-process-send-receive-messages.md).

Dans cet article, vous allez concevoir une solution de traitement par lots en créant deux applications logiques dans le même abonnement et la même région Azure, en suivant cet ordre précis :

* Une application logique [« réceptrice de lots »](#receiver), qui accepte et collecte les messages dans un lot jusqu’à ce que vos critères précisés soient remplis pour mettre en production et traiter ces messages. Dans ce scénario, le récepteur de lots encode également les messages dans le lot au moyen de l’accord X12 spécifié ou des identités partenaires.

  Veillez à commencer par la création du récepteur de lots, afin de pouvoir sélectionner ensuite la destination des lots lors de la création de l’expéditeur de lots.

* Une application logique [« expéditrice de lots »](#sender), qui envoie les messages au récepteur de lots précédemment créé. 

Assurez-vous que votre récepteur et votre expéditeur de lots partagent le même abonnement *et* la même région Azure. Si ce n’est pas le cas, vous ne pouvez pas sélectionner le récepteur de lots lorsque vous créez l’expéditeur de lots, car ils ne sont pas visibles entre eux.

## <a name="prerequisites"></a>Prérequis

Pour suivre cet exemple, vous avez besoin de ce qui suit :

* Un abonnement Azure. Si vous ne disposez d’aucun abonnement, vous pouvez [commencer par créer gratuitement un compte Azure](https://azure.microsoft.com/free/). Vous pouvez aussi [souscrire un abonnement avec paiement à l’utilisation](https://azure.microsoft.com/pricing/purchase-options/).

* Des connaissances de base en [création d’applications logiques](../logic-apps/quickstart-create-first-logic-app-workflow.md)

* Un [compte d’intégration](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md) existant associé à votre abonnement Azure et lié à vos applications logiques

* Au moins deux [partenaires](../logic-apps/logic-apps-enterprise-integration-partners.md) existants dans votre compte d’intégration. Chaque partenaire doit utiliser le qualificateur X12 (Code alphanumérique standard) en tant qu’identité d’entreprise dans les propriétés du partenaire.

* Un [accord X12](../logic-apps/logic-apps-enterprise-integration-x12.md) existant dans votre compte d’intégration

* Pour utiliser Visual Studio plutôt que le portail Azure, veillez à [configurer Visual Studio pour travailler avec Logic Apps](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md).

<a name="receiver"></a>

## <a name="create-x12-batch-receiver"></a>Créer un récepteur de lots X12

Avant de pouvoir envoyer des messages à un lot, ce lot doit d’abord exister en tant que destination vers laquelle envoyer ces messages. Ainsi, vous devez commencer par créer l’application logique « réceptrice de lots », qui démarre avec le déclencheur **Lot**. De cette façon, lorsque vous créez l’application logique « expéditrice de lots », vous pouvez sélectionner l’application logique réceptrice de lots. Le récepteur de lots continue à collecter les messages jusqu’à ce que vos critères spécifiés soient remplis pour mettre en production et traiter ces messages. Alors que les récepteurs de lots n’ont pas besoin de connaître les expéditeurs de lots, les expéditeurs de lots, eux, doivent connaître la destination vers laquelle envoyer les messages. 

Pour ce récepteur de lots, vous devez spécifier le mode et le nom du lot, les critères de mise en production, l’accord X12 et d’autres paramètres. 

1. Dans le [portail Azure](https://portal.azure.com) ou dans Visual Studio, créez une application logique et nommez-la ainsi : "BatchX12Messages"

2. [Liez votre application logique à votre compte d’intégration](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md#link-account).

3. Dans le Concepteur d'applications logiques, ajoutez le déclencheur **Lot** qui démarre le flux de travail de votre application logique. Dans la zone de recherche, entrez « lot » comme filtre. Sélectionner le déclencheur : **Traiter les messages par lots**

   ![Ajout du déclencheur Lot](./media/logic-apps-scenario-EDI-send-batch-messages/add-batch-receiver-trigger.png)

4. Définissez les propriétés du récepteur de lots : 

   | Propriété | Valeur | Notes | 
   |----------|-------|-------|
   | **Mode Batch** | En ligne |  |  
   | **Nom du lot** | TestBatch | Disponible uniquement avec le mode Batch **Inline** | 
   | **Critères de mise en production** | Sur la base du nombre de messages, sur la base de la planification | Disponible uniquement avec le mode Batch **Inline** | 
   | **Nombre de messages** | 10 | Disponible uniquement avec le critère de mise en production **Sur la base du nombre de messages** | 
   | **Intervalle** | 10 | Disponible uniquement avec le critère de mise en production **Sur la base de la planification** | 
   | **Fréquence** | minute | Disponible uniquement avec le critère de mise en production **Sur la base de la planification** | 
   ||| 

   ![Détails à fournir concernant le déclencheur Lot](./media/logic-apps-scenario-EDI-send-batch-messages/batch-receiver-release-criteria.png)

   > [!NOTE]
   > Cet exemple ne configure pas de partition pour le lot, chaque lot utilise donc la même clé de partition. Pour en savoir plus sur les partitions, consultez [Traiter les messages par lots](../logic-apps/logic-apps-batch-process-send-receive-messages.md#batch-sender).

5. Nous allons maintenant ajouter une action qui encode chaque lot : 

   1. Sous le déclencheur Lot, sélectionnez **Nouvelle étape**.

   2. Dans la zone de recherche, entrez « lot X12 » en tant que filtre et sélectionnez cette action (quelle que soit la version) : **Encodage par lots <*version*> - X12** 

      ![Sélection de l’action d’encodage par lots X12](./media/logic-apps-scenario-EDI-send-batch-messages/add-batch-encode-action.png)

   3. Si vous n’êtes pas déjà connecté à votre compte d’intégration, connectez-vous maintenant. Indiquez un nom pour votre connexion, sélectionnez le compte d’intégration, puis choisissez **Créer**.

      ![Établir la connexion entre l’encodeur par lots et le compte d’intégration](./media/logic-apps-scenario-EDI-send-batch-messages/batch-encoder-connect-integration-account.png)

   4. Définissez ces propriétés pour votre action d’encodeur par lots :

      | Propriété | Description |
      |----------|-------------|
      | **Nom de l’accord X12** | Ouvrez la liste, puis sélectionnez votre accord. <p>Si votre liste est vide, assurez-vous d’avoir bien [associé votre application logique au compte d’intégration](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md#link-account) détenant l’accord qui vous intéresse. | 
      | **BatchName** | Cliquez dans cette zone. Lorsque la liste de contenu dynamique s’affiche, sélectionnez le jeton **Nom du lot**. | 
      | **PartitionName** | Cliquez dans cette zone. Lorsque la liste de contenu dynamique s’affiche, sélectionnez le jeton **Nom de la partition**. | 
      | **Éléments** | Fermez la zone d’informations sur l’élément, puis cliquez dans cette zone. Lorsque la liste de contenu dynamique s’affiche, sélectionnez le jeton **Éléments traités par lots**. | 
      ||| 

      ![Détails de l’action d’encodage par lots](./media/logic-apps-scenario-EDI-send-batch-messages/batch-encode-action-details.png)

      Pour la zone **Éléments** :

      ![Éléments de l’action d’encodage par lots](./media/logic-apps-scenario-EDI-send-batch-messages/batch-encode-action-items.png)

6. Enregistrez votre application logique. 

7. Si vous utilisez Visual Studio, veillez à [déployer votre application logique réceptrice de lots sur Azure](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md#deploy-logic-app-to-azure). Sinon, vous ne pouvez pas sélectionner le récepteur de lots lorsque vous créez l’expéditeur de lots.

### <a name="test-your-logic-app"></a>Tester votre application logique

Pour vérifier que votre récepteur de lots fonctionne comme prévu, vous pouvez ajouter une action HTTP à des fins de test et envoyer un message par lot au [service Request Bin](https://requestbin.com/). 

1. Sous l’action d’encodage X12, choisissez **Nouvelle étape**. 

2. Dans la zone de recherche, entrez « http » en tant que filtre. Sélectionnez cette action : **HTTP - HTTP**
    
   ![Sélection de l’action HTTP](./media/logic-apps-scenario-EDI-send-batch-messages/batch-receiver-add-http-action.png)

3. Définissez les propriétés de l’action HTTP :

   | Propriété | Description | 
   |----------|-------------|
   | **Méthode** | Dans cette liste, sélectionnez **POST**. | 
   | **Uri** | Générez un URI pour votre RequestBin, puis saisissez-le dans cette zone. | 
   | **Corps** | Cliquez dans cette zone. Lorsque la liste de contenu dynamique s’affiche, sélectionnez le jeton **Corps**, qui apparaît dans la section **Encodage Batch par nom de contrat**. <p>Si le jeton **Corps** n’apparaît pas, en regard de **Encodage Batch par nom de contrat**, sélectionnez **Afficher plus**. | 
   ||| 

   ![Saisie des détails de l’action HTTP](./media/logic-apps-scenario-EDI-send-batch-messages/batch-receiver-add-http-action-details.png)

4. Enregistrez votre application logique. 

   Votre application logique réceptrice de lots ressemble à l’exemple suivant : 

   ![Enregistrer votre application logique réceptrice de lots](./media/logic-apps-scenario-EDI-send-batch-messages/batch-receiver-finished.png)

<a name="sender"></a>

## <a name="create-x12-batch-sender"></a>Créer l’expéditeur de lots X12

À présent, créez une ou plusieurs applications logiques qui envoient des messages vers l’application logique réceptrice de lots. Dans chaque expéditeur de lots, vous spécifiez l’application logique réceptrice de lots avec le nom du lot, le contenu du message et tout autre paramètre nécessaire. Vous pouvez éventuellement fournir une clé de partition unique pour diviser le lot en sous-ensembles afin de collecter les messages. 

* En vous assurant d’avoir [créé votre récepteur de lots](#receiver) avant de créer votre expéditeur de lots, vous pouvez sélectionner le récepteur existant comme destinataire de lots. Alors que les récepteurs de lots n’ont pas besoin de connaître les expéditeurs de lots, les expéditeurs de lots, eux, doivent savoir où envoyer les messages. 

* Vérifiez que vos récepteur et expéditeur de lots partagent la même région *et* le même abonnement Azure. Si ce n’est pas le cas, vous ne pouvez pas sélectionner le récepteur de lots lorsque vous créez l’expéditeur de lots, car ils ne sont pas visibles entre eux.

1. Créez une autre application logique portant le nom suivant : "SendX12MessagesToBatch" 

2. Dans la zone de recherche, saisissez le filtre « when a http request ». Sélectionner le déclencheur : **Lors de la réception d’une requête HTTP** 
   
   ![Ajout du déclencheur Request](./media/logic-apps-scenario-EDI-send-batch-messages/add-request-trigger-sender.png)

3. Ajoutez une action pour envoyer des messages vers un lot.

   1. Sous l’action de requête HTTP, choisissez **Nouvelle étape**.

   2. Dans la zone de recherche, entrez « lot » comme filtre. 
   Dans la liste **Actions**, sélectionnez cette action : **Choisir un workflow Logic Apps avec déclencheur de lot - Envoyer les messages au lot**

      ![Sélection de « Choisir un workflow Logic Apps avec déclencheur de lot »](./media/logic-apps-scenario-EDI-send-batch-messages/batch-sender-select-batch-trigger.png)

   3. Sélectionnez à présent l’application logique « BatchX12Messages » créée précédemment.

      ![Sélection de l’application logique « BatchReceiver »](./media/logic-apps-scenario-EDI-send-batch-messages/batch-sender-select-batch-receiver.png)

   4. Sélectionnez cette action : **Traiter_les_messages_par_lots - <*votre-récepteur-de-lots*>**

      ![Sélection de l’action « Batch_messages »](./media/logic-apps-scenario-EDI-send-batch-messages/batch-sender-select-batch-messages-action.png)

4. Définissez les propriétés de l’expéditeur de lots.

   | Propriété | Description | 
   |----------|-------------| 
   | **Nom du lot** | Nom défini par l’application logique réceptrice (« TestBatch » dans cet exemple) <p>**Important !** Le nom du lot est validé lors de l’exécution et doit correspondre au nom spécifié par l’application logique réceptrice. Si vous modifiez ce nom, l’expéditeur de lots échoue. | 
   | **Contenu du message** | Contenu du message que vous souhaitez envoyer (jeton **Corps** dans cet exemple) | 
   ||| 
   
   ![Définition des propriétés du lot](./media/logic-apps-scenario-EDI-send-batch-messages/batch-sender-set-batch-properties.png)

5. Enregistrez votre application logique. 

   Votre application logique expéditrice de lots ressemble à l’exemple suivant :

   ![Enregistrement de votre application logique expéditrice de lots](./media/logic-apps-scenario-EDI-send-batch-messages/batch-sender-finished.png)

## <a name="test-your-logic-apps"></a>Tester les applications logiques

Pour tester votre solution de traitement par lots, publiez des messages X12 dans votre application logique expéditrice de lots à partir de [Postman](https://www.getpostman.com/postman) ou d’un outil similaire. Vous allez bientôt commencer à recevoir des messages X12 dans votre RequestBin, soit toutes les 10 minutes, soit sous la forme de lots de 10, tous avec la même clé de partition.

## <a name="next-steps"></a>Étapes suivantes

* [Traiter par lots des messages](../logic-apps/logic-apps-batch-process-send-receive-messages.md) 
