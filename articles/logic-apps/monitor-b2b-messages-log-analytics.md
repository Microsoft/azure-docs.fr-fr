---
title: Superviser les messages B2B à l'aide d'Azure Monitor
description: Détecter les problèmes liés aux messages AS2, X12 et EDIFACT en configurant les journaux d'activité Azure Monitor et en collectant des données de diagnostic pour Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: divswa, logicappspm
ms.topic: article
ms.date: 01/30/2020
ms.openlocfilehash: 5baa4d4d968adb25b5520ca91149970f5c5578e9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86536264"
---
# <a name="set-up-azure-monitor-logs-and-collect-diagnostics-data-for-b2b-messages-in-azure-logic-apps"></a>Configurer les journaux Azure Monitor et collecter des données de diagnostic pour des messages B2B dans Azure Logic Apps

Après avoir configuré la communication B2B entre les partenaires commerciaux dans votre compte d’intégration, ces partenaires peuvent échanger des messages à l'aide de protocoles comme AS2, X12 et EDIFACT. Pour vérifier le bon fonctionnement de cette communication, vous pouvez configurer les [journaux Azure Monitor](../azure-monitor/platform/data-platform-logs.md) pour votre compte d’intégration. [Azure Monitor](../azure-monitor/overview.md) vous permet de superviser vos environnements cloud et locaux afin de vous aider à garantir plus facilement leur disponibilité et leurs performances. Les journaux Azure Monitor vous permettent d'enregistrer et de stocker des informations relatives aux données et événements d'exécution, tels que les événements déclencheurs, événements d’exécution et événements d’action dans un [espace de travail Log Analytics](../azure-monitor/platform/resource-logs.md#send-to-log-analytics-workspace). Pour les messages, la journalisation collecte également des informations, notamment :

* Nombre et état des messages
* État des accusés de réception
* Corrélations entre les messages et les accusés de réception
* Descriptions détaillées des erreurs en cas d’échec

Azure Monitor vous permet de créer des [requêtes de journal](../azure-monitor/log-query/log-query-overview.md) pour vous aider à rechercher et à consulter ces informations. Vous pouvez également [utiliser ces données de diagnostic avec d'autres services Azure](../logic-apps/monitor-logic-apps-log-analytics.md#extend-data), tels que Stockage Azure et Azure Event Hubs.

Pour configurer la journalisation pour votre compte d’intégration, [installez la solution Logic Apps B2B](#install-b2b-solution) dans le portail Azure. Cette solution fournit des informations agrégées pour les événements de message B2B. Ensuite, pour activer la journalisation et créer des requêtes propres à ces informations, configurez les [journaux Azure Monitor](#set-up-resource-logs).

Cet article explique comment activer la journalisation Azure Monitor pour votre compte d’intégration.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="prerequisites"></a>Conditions préalables requises

* Un espace de travail Log Analytics. Si vous n’avez pas d’espace de travail Log Analytics, découvrez [comment créer un espace de travail Log Analytics](../azure-monitor/learn/quick-create-workspace.md).

* Application logique configurée avec la journalisation Azure Monitor, qui envoie ces informations à un espace de travail Log Analytics. Découvrez [comment configurer les journaux Azure Monitor pour votre application logique](../logic-apps/monitor-logic-apps.md).

* Un compte d’intégration lié à votre application logique. Découvrez [comment lier votre compte d’intégration à votre application logique](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md).

<a name="install-b2b-solution"></a>

## <a name="install-logic-apps-b2b-solution"></a>Installation de la solution Logic Apps B2B

Pour suivre les messages B2B liés à votre application logique avec les journaux Azure Monitor, vous devez ajouter la solution **Logic Apps B2B** à votre espace de travail Log Analytics.

1. Dans la zone de recherche du [portail Azure](https://portal.azure.com), entrez `log analytics workspaces`, puis sélectionnez **Espaces de travail Log Analytics**.

   ![Sélectionnez « Espaces de travail Log Analytics »](./media/monitor-b2b-messages-log-analytics/find-select-log-analytics-workspaces.png)

1. Sous **Espaces de travail Log Analytics**, sélectionnez votre espace de travail.

   ![Sélectionnez votre espace de travail Log Analytics.](./media/monitor-b2b-messages-log-analytics/select-log-analytics-workspace.png)

1. Dans le volet Vue d’ensemble, sous **Bien démarrer avec Log Analytics** > **Configurer des solutions de monitoring**, sélectionnez **Afficher les solutions**.

   ![Dans le volet Vue d’ensemble, sélectionner « Afficher les solutions ».](./media/monitor-b2b-messages-log-analytics/log-analytics-workspace.png)

1. Dans le volet Vue d’ensemble, sélectionnez **Ajouter**.

   ![Dans le volet Vue d'ensemble, ajoutez une nouvelle solution.](./media/monitor-b2b-messages-log-analytics/add-logic-apps-management-solution.png)

1. Une fois la **Place de marché** ouverte, dans la zone de recherche, entrez `logic apps b2b`, puis sélectionnez **Logic Apps B2B**.

   ![Sur la Place de marché, sélectionnez « Logic Apps Management ».](./media/monitor-b2b-messages-log-analytics/select-logic-apps-b2b-solution.png)

1. Dans le volet de description de la solution, sélectionnez **Créer**.

   ![Sélectionnez « Créer » pour ajouter la solution « Logic Apps B2B ».](./media/monitor-b2b-messages-log-analytics/create-logic-apps-b2b-solution.png)

1. Vérifiez et confirmez l’espace de travail Log Analytics dans lequel vous souhaitez installer la solution, puis sélectionnez à nouveau **Créer**.

   ![Sélectionnez « Créer » pour « Logic Apps B2B »](./media/monitor-b2b-messages-log-analytics/confirm-log-analytics-workspace.png)

   Une fois qu’Azure a déployé la solution sur le groupe de ressources Azure qui contient votre espace de travail Log Analytics, la solution apparaît dans le volet récapitulatif de votre espace de travail. Lorsque des messages B2B sont traités, le nombre de messages de ce volet est mis à jour.

   ![Volet récapitulatif de l'espace de travail](./media/monitor-b2b-messages-log-analytics/b2b-overview-messages-summary.png)

<a name="set-up-resource-logs"></a>

## <a name="set-up-azure-monitor-logs"></a>Configurer les journaux Azure Monitor

Vous pouvez activer la journalisation Azure Monitor directement à partir de votre compte d’intégration.

1. Dans le [portail Azure](https://portal.azure.com), recherchez et sélectionnez votre compte d’intégration.

   ![Trouvez et sélectionnez votre compte d’intégration](./media/monitor-b2b-messages-log-analytics/find-integration-account.png)

1. Dans le menu de votre compte d’intégration, sous **Supervision**, sélectionnez **Paramètres de diagnostic**. Sélectionnez **Ajouter le paramètre de diagnostic**.

   ![Sous « Supervision », sélectionnez « Paramètres de diagnostic ».](./media/monitor-b2b-messages-log-analytics/monitor-diagnostics-settings.png)

1. Pour créer le paramètre, procédez comme suit :

   1. Donnez un nom au paramètre.

   1. Sélectionnez **Envoyer à Log Analytics**.

   1. Dans **Abonnement**, sélectionnez l'abonnement Azure associé à votre espace de travail Log Analytics.

   1. Dans **Espace de travail Log Analytics**, sélectionnez l'espace de travail que vous souhaitez utiliser.

   1. Sous **journal**, sélectionnez la catégorie **WorkflowRuntime**, qui spécifie la catégorie d'événement que vous souhaitez enregistrer.

   1. Quand vous avez terminé, sélectionnez **Enregistrer**.

   Par exemple : 

   ![Configurer les journaux Azure Monitor pour collecter des données de diagnostic](./media/monitor-b2b-messages-log-analytics/send-diagnostics-data-log-analytics-workspace.png)

<a name="view-message-status"></a>

## <a name="view-message-status"></a>Afficher l’état du message

Après les exécutions de votre application logique, vous pouvez consulter l'état et les données relatifs à ces messages dans votre espace de travail Log Analytics.

1. Dans la zone de recherche du [portail Azure](https://portal.azure.com), recherchez et ouvrez votre espace de travail Log Analytics.

1. Dans le menu de votre espace de travail, sélectionnez **Résumé de l'espace de travail** > **Logic Apps B2B**.

   ![Volet récapitulatif de l'espace de travail](./media/monitor-b2b-messages-log-analytics/b2b-overview-messages-summary.png)

   > [!NOTE]
   > Si la vignette Logic Apps B2B n'affiche pas de résultats immédiatement après une exécution, sélectionnez **Actualiser** ou attendez un peu avant de réessayer.

   Par défaut, la vignette **Logic Apps B2B** affiche des données d’une journée. Pour modifier l’étendue des données en définissant un intervalle différent, sélectionnez le contrôle de l’étendue en haut de la page :

   ![Modification de l’intervalle](./media/monitor-b2b-messages-log-analytics/change-summary-interval.png)

1. Un fois le tableau de bord de l’état du message affiché, vous pouvez afficher davantage de détails sur un type de message spécifique qui affiche des données d’une journée. Sélectionnez la vignette **AS2**, **X12** ou **EDIFACT**.

   ![Afficher les états des messages](./media/monitor-b2b-messages-log-analytics/workspace-summary-b2b-messages.png)

   Une liste de messages s’affiche pour la vignette choisie. Par exemple, voici comment peut se présenter une liste de messages AS2 :

   ![États et détails des messages AS2](./media/monitor-b2b-messages-log-analytics/as2-message-results-list.png)

   Pour en savoir plus sur les propriétés de chaque type de message, consultez les descriptions de propriété de message suivantes :

   * [Propriétés de message AS2](#as2-message-properties)
   * [Propriétés de message X12](#x12-message-properties)
   * [Propriétés de message EDIFACT](#EDIFACT-message-properties)

<!--
1. To view or export the inputs and outputs for specific messages, select those messages, and select **Download**. When you're prompted, save the .zip file to your local computer, and then extract that file.

   The extracted folder includes a folder for each selected message. If you set up acknowledgements, the message folder also includes files with acknowledgement details. Each message folder has at least these files:
   
   * Human-readable files with the input payload and output payload details
   * Encoded files with the inputs and outputs

   For each message type, you can find the folder and file name formats here:

   * [AS2 folder and file name formats](#as2-folder-file-names)
   * [X12 folder and file name formats](#x12-folder-file-names)
   * [EDIFACT folder and file name formats](#edifact-folder-file-names)

   ![Download message files](./media/monitor-b2b-messages-log-analytics/download-messages.png)

1. To view all actions that have the same run ID, on the **Log Search** page, select a message from the message list.

   You can sort these actions by column, or search for specific results.

   * To search results with prebuilt queries, select **Favorites**.

   * Learn [how to build queries by adding filters](../logic-apps/create-monitoring-tracking-queries.md). Or learn more about [how to find data with log searches in Azure Monitor logs](../azure-monitor/log-query/log-query-overview.md).

   * To change query in the search box, update the query with the columns and values that you want to use as filters.
-->

<a name="message-list-property-descriptions"></a>

## <a name="property-descriptions-and-name-formats-for-as2-x12-and-edifact-messages"></a>Formats de noms et descriptions de propriétés pour les messages AS2, X 12 et EDIFACT

Pour chaque type de message, voici les formats de noms et descriptions de propriétés pour les fichiers de message téléchargés.

<a name="as2-message-properties"></a>

### <a name="as2-message-property-descriptions"></a>Description de propriétés des messages AS2

Voici les descriptions de propriété pour chaque message AS2.

| Propriété | Description |
|----------|-------------|
| **Expéditeur** | Partenaire invité spécifié dans **Paramètres de réception**, ou partenaire hôte spécifié dans **Paramètres d’envoi** pour un accord AS2 |
| **Destinataire** | Partenaire hôte spécifié dans **Paramètres de réception**, ou partenaire invité spécifié dans **Paramètres d’envoi** pour un accord AS2 |
| **Application logique** | Application logique dans laquelle les actions AS2 sont configurées |
| **État** | État du message AS2 <br>Success = a reçu ou envoyé un message AS2 valide. Aucun MDN n’est configuré. <br>Success = a reçu ou envoyé un message AS2 valide. MDN est configuré et reçu ou envoyé. <br>Failed = a reçu un message AS2 non valide. Aucun MDN n’est configuré. <br>Pending = a reçu ou envoyé un message AS2 valide. Un MDN est configuré et attendu. |
| **ACK** | État du message MDN <br>Accepted = a reçu ou envoyé un MDN positif. <br>Pending = en attente de réception ou d’envoi d’un MDN. <br>Rejected = a reçu ou envoyé un MDN négatif. <br>Not Required = aucun MDN n’est configuré dans l’accord. |
| **Sens** | Direction du message AS2 |
| **ID de suivi** | ID qui établit la corrélation de l’ensemble des déclencheurs et actions dans une application logique |
| **ID de message** | ID de message AS2 extrait des en-têtes de message AS2 |
| **Timestamp** | Heure à laquelle l’action AS2 a traité le message |
|||

<!--
<a name="as2-folder-file-names"></a>

### AS2 name formats for downloaded message files

Here are the name formats for each downloaded AS2 message folder and files.

| Folder or file | Name format |
|----------------|-------------|
| Message folder | [sender]\_[receiver]\_AS2\_[correlation-ID]\_[message-ID]\_[timestamp] |
| Input, output, and if set up, acknowledgement files | **Input payload**: [sender]\_[receiver]\_AS2\_[correlation-ID]\_input_payload.txt </p>**Output payload**: [sender]\_[receiver]\_AS2\_[correlation-ID]\_output\_payload.txt </p></p>**Inputs**: [sender]\_[receiver]\_AS2\_[correlation-ID]\_inputs.txt </p></p>**Outputs**: [sender]\_[receiver]\_AS2\_[correlation-ID]\_outputs.txt |
|||
-->

<a name="x12-message-properties"></a>

### <a name="x12-message-property-descriptions"></a>Description de propriétés des messages X12

Voici les descriptions de propriété pour chaque message X12.

| Propriété | Description |
|----------|-------------|
| **Expéditeur** | Partenaire invité spécifié dans **Paramètres de réception**, ou partenaire hôte spécifié dans **Paramètres d’envoi** pour un accord X12 |
| **Destinataire** | Partenaire hôte spécifié dans **Paramètres de réception**, ou partenaire invité spécifié dans **Paramètres d’envoi** pour un accord X12 |
| **Application logique** | Application logique dans laquelle les actions X12 sont configurées |
| **État** | État du message X12 <br>Success = a reçu ou envoyé un message X12 valide. Aucun accusé de réception fonctionnel configuré. <br>Success = a reçu ou envoyé un message X12 valide. Accusé de réception fonctionnel configuré et reçu ou envoyé. <br>Failed = a reçu ou envoyé un message X12 non valide. <br>Pending = a reçu ou envoyé un message X12 valide. Accusé de réception fonctionnel configuré et attendu. |
| **ACK** | État de l’accusé de réception fonctionnel (997) <br>Accepted = a reçu ou envoyé un accusé de réception positif. <br>Rejected = a reçu ou envoyé un accusé de réception négatif. <br>Pending = attendait un accusé de réception fonctionnel mais ne l’a pas reçu. <br>Pending = a généré un accusé de réception fonctionnel mais ne peut pas l’envoyer au partenaire. <br>Not Required = accusé de réception fonctionnel non configuré. |
| **Sens** | Direction du message X12 |
| **ID de suivi** | ID qui établit la corrélation de l’ensemble des déclencheurs et actions dans une application logique |
| **Type de message** | Type de message X12 EDI |
| **ICN** | Numéro de contrôle d’échange pour le message X12 |
| **TSCN** | Numéro de contrôle de document informatisé pour le message X12 |
| **Timestamp** | Heure à laquelle l’action X12 a traité le message |
|||

<!--
<a name="x12-folder-file-names"></a>

### X12 name formats for downloaded message files

Here are the name formats for each downloaded X12 message folder and files.

| Folder or file | Name format |
|----------------|-------------|
| Message folder | [sender]\_[receiver]\_X12\_[interchange-control-number]\_[global-control-number]\_[transaction-set-control-number]\_[timestamp] |
| Input, output, and if set up, acknowledgement files | **Input payload**: [sender]\_[receiver]\_X12\_[interchange-control-number]\_input_payload.txt </p>**Output payload**: [sender]\_[receiver]\_X12\_[interchange-control-number]\_output\_payload.txt </p></p>**Inputs**: [sender]\_[receiver]\_X12\_[interchange-control-number]\_inputs.txt </p></p>**Outputs**: [sender]\_[receiver]\_X12\_[interchange-control-number]\_outputs.txt |
|||
-->

<a name="EDIFACT-message-properties"></a>

### <a name="edifact-message-property-descriptions"></a>Description de propriétés des messages EDIFACT

Voici les descriptions de propriété pour chaque message EDIFACT.

| Propriété | Description |
|----------|-------------|
| **Expéditeur** | Partenaire invité spécifié dans **Paramètres de réception**, ou partenaire hôte spécifié dans **Paramètres d’envoi** pour un accord EDIFACT |
| **Destinataire** | Partenaire hôte spécifié dans **Paramètres de réception**, ou partenaire invité spécifié dans **Paramètres d’envoi** pour un accord EDIFACT |
| **Application logique** | Application logique dans laquelle les actions EDIFACT sont configurées |
| **État** | État du message EDIFACT <br>Success = a reçu ou envoyé un message EDIFACT valide. Aucun accusé de réception fonctionnel configuré. <br>Success = a reçu ou envoyé un message EDIFACT valide. Accusé de réception fonctionnel configuré et reçu ou envoyé. <br>Failed = a reçu ou envoyé un message EDIFACT non valide. <br>Pending = a reçu ou envoyé un message EDIFACT valide. Accusé de réception fonctionnel configuré et attendu. |
| **ACK** | État de l'accusé de réception fonctionnel (CONTRL) <br>Accepted = a reçu ou envoyé un accusé de réception positif. <br>Rejected = a reçu ou envoyé un accusé de réception négatif. <br>Pending = attendait un accusé de réception fonctionnel mais ne l’a pas reçu. <br>Pending = a généré un accusé de réception fonctionnel mais ne peut pas l’envoyer au partenaire. <br>Not Required = accusé de réception fonctionnel non configuré. |
| **Sens** | Direction du message EDIFACT |
| **ID de suivi** | ID qui établit la corrélation de l’ensemble des déclencheurs et actions dans une application logique |
| **Type de message** | Type de message EDIFACT |
| **ICN** | Numéro de contrôle de l’échange pour le message EDIFACT |
| **TSCN** | Numéro de contrôle de document informatisé pour le message EDIFACT |
| **Timestamp** | Heure à laquelle l’action EDIFACT a traité le message |
|||

<!--
<a name="edifact-folder-file-names"></a>

### EDIFACT name formats for downloaded message files

Here are the name formats for each downloaded EDIFACT message folder and files.

| Folder or file | Name format |
|----------------|-------------|
| Message folder | [sender]\_[receiver]\_EDIFACT\_[interchange-control-number]\_[global-control-number]\_[transaction-set-control-number]\_[timestamp] |
| Input, output, and if set up, acknowledgement files | **Input payload**: [sender]\_[receiver]\_EDIFACT\_[interchange-control-number]\_input_payload.txt </p>**Output payload**: [sender]\_[receiver]\_EDIFACT\_[interchange-control-number]\_output\_payload.txt </p></p>**Inputs**: [sender]\_[receiver]\_EDIFACT\_[interchange-control-number]\_inputs.txt </p></p>**Outputs**: [sender]\_[receiver]\_EDIFACT\_[interchange-control-number]\_outputs.txt |
|||
-->

## <a name="next-steps"></a>Étapes suivantes

* [Créer des requêtes de supervision et de suivi](../logic-apps/create-monitoring-tracking-queries.md)
