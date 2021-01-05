---
title: Surveiller les runbooks Azure Automation avec des alertes de métriques
description: Cet article explique comment configurer une alerte de métrique en fonction de l’état d’achèvement du runbook.
services: automation
ms.date: 08/10/2020
ms.topic: article
ms.openlocfilehash: 8767687f0b72d3469bef570770ac81fa8300097f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88055924"
---
# <a name="monitor-runbooks-with-metric-alerts"></a>Surveiller des runbooks avec des alertes de métrique

Dans cet article, vous apprenez à créer une [alerte de métrique](../azure-monitor/platform/alerts-metric-overview.md) en fonction de l’état d’achèvement du runbook.

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au [portail Azure](https://portal.azure.com)

## <a name="create-alert"></a>Créer une alerte

Les alertes vous permettent de définir une condition à surveiller, ainsi qu’une action à entreprendre lorsque cette condition est remplie.

1. Lancez le service Azure Automation dans le portail Azure en cliquant sur **Tous les services**, puis en recherchant et en cliquant sur **Comptes Automation**.

2. Dans la liste des comptes Automation, sélectionnez le compte pour lequel vous souhaitez créer une alerte. 

3. Sous **Supervision**, sélectionnez **Alertes**, puis **+ Nouvelle règle d’alerte**. L’étendue pour la cible est déjà définie et associée à votre compte Automation.

### <a name="configure-alert-criteria"></a>Configurer les critères de l’alerte

1. Cliquez sur **Sélectionner une condition**. Sélectionnez **Métriques** pour le **Type de signal**, puis choisissez **Nombre total de travaux** dans la liste.

2. La page **Configurer la logique du signal** vous permet de définir la logique qui déclenche l’alerte. Sous le graphique d’historique, deux dimensions apparaissent : **Nom du Runbook** et **État**. Les dimensions correspondent aux différentes propriétés d’une métrique qui sont utilisables pour le filtrage des résultats. Pour la dimension **Nom du Runbook**, sélectionnez le runbook pour lequel vous souhaitez créer une alerte, ou laissez la dimension vide si vous voulez définir une alerte pour tous les runbooks. Pour la dimension **État**, sélectionnez dans la liste déroulante l’état que vous souhaitez surveiller. Les valeurs de nom et d’état de runbook qui s’affichent dans la liste déroulante s’appliquent uniquement aux travaux qui se sont exécutés la semaine précédente.

   Si vous souhaitez créer une alerte pour un état ou un runbook qui ne figurent pas dans la liste déroulante, cliquez sur l’option **Ajouter une valeur personnalisée** en regard de la dimension. Cette action ouvre une boîte de dialogue vous permettant de spécifier une valeur personnalisée non émise récemment pour cette dimension. Si vous entrez une valeur de propriété inexistante, votre alerte ne se déclenchera pas.

   > [!NOTE]
   > Lorsque vous ne spécifier pas de nom pour la dimension **RunbookName**, si des runbooks répondent aux critères d’état (cela comprend les runbooks système masqués), vous recevez une alerte.

    Par exemple, pour alerter quand un runbook retourne un état _En échec_, en plus de spécifier le nom du runbook, pour la dimension **État**, ajoutez la valeur de dimension personnalisée **En échec**.

    :::image type="content" source="./media/automation-alert-metric/specify-dimension-custom-value.png" alt-text="Spécifier une valeur de dimension personnalisée" border="false":::

3. Sous **Logique d’alerte**, définissez la condition et le seuil de votre alerte. Un aperçu de la condition que vous avez définie s’affiche en dessous.

4. Sous **Évaluées sur la base de**, sélectionnez l’intervalle de temps et la fréquence d’exécution de la requête. Par exemple, si vous définissez **Période** sur **Au cours des 5 dernières minutes** et **Fréquence** sur **Toutes les minutes**, l’alerte recherche le nombre de runbooks qui répondaient à vos critères au cours des 5 dernières minutes. Cette requête s’exécute toutes les minutes, et une fois que les critères d’alerte que vous avez définis ne sont plus détectés au cours d’une période de 5 minutes, l’alerte se résout d’elle-même. Lorsque vous avez terminé, cliquez sur **Terminé**.

   ![Sélection d’une ressource pour l’alerte](./media/automation-alert-activity-log/configure-signal-logic.png)

### <a name="define-the-action-to-take"></a>Définir l’action à entreprendre

1. Sous **Groupe d’actions**, sélectionnez **Spécifier un groupe d’actions**. Un groupe d’actions est un groupe que vous pouvez utiliser dans plusieurs alertes. Il peut s’agir, sans s’y limiter, de notifications par e-mail, de runbooks, de webhooks et bien plus encore. Pour en savoir plus sur les groupes d’actions et les étapes à suivre pour en créer un qui envoie une notification par e-mail, consultez [Créer et gérer des groupes d’actions](../azure-monitor/platform/action-groups.md).

### <a name="define-alert-details"></a>Définir les détails de l’alerte

1. Sous **Détails de la règle d’alerte**, donnez à l’alerte un nom convivial et une description. Définissez le champ **Gravité** en fonction de la condition de l’alerte. Il existe cinq niveaux de gravité compris entre 0 et 5. Les alertes sont traitées de la même manière quelle que soit leur gravité, et vous pouvez faire correspondre la gravité à votre logique métier.

1. Par défaut, les règles sont activées lors de la création, sauf si vous sélectionnez **Non** pour l’option **Activer la règle d’alerte lors de la création**. Pour les alertes créées dans un état désactivé, vous pouvez les activer à l’avenir lorsque vous êtes prêt. Sélectionnez **Créer une règle d’alerte** pour enregistrer vos modifications.

## <a name="receive-notification"></a>Recevoir une notification

Lorsque les critères d’alerte sont satisfaits, le groupe d’actions exécute l’action définie. Selon l’exemple fourni dans cet article, un e-mail est envoyé. L’image ci-après illustre un exemple d’e-mail que vous recevez après le déclenchement de l’alerte :

![Alerte par courrier électronique](./media/automation-alert-activity-log/alert-email.png)

Une fois que la métrique ne dépasse plus le seuil défini, l’alerte est désactivée et le groupe d’actions exécute l’action définie. Si une action de type e-mail est sélectionnée, le système envoie un e-mail de résolution indiquant que l’alerte a été résolue.

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations, consultez [Utiliser une alerte pour déclencher un runbook Azure Automation](automation-create-alert-triggered-runbook.md).
