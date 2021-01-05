---
title: Créer, voir et gérer des alertes de journal à l’aide d’Azure Monitor | Microsoft Docs
description: Utiliser Azure Monitor pour créer, afficher et gérer les règles d’alerte de journal
author: yanivlavi
ms.author: yalavi
ms.topic: conceptual
ms.date: 07/29/2019
ms.subservice: alerts
ms.openlocfilehash: a8dbadd3af9dc21ced54af151e4de705f854e011
ms.sourcegitcommit: b6267bc931ef1a4bd33d67ba76895e14b9d0c661
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/19/2020
ms.locfileid: "97696057"
---
# <a name="create-view-and-manage-log-alerts-using-azure-monitor"></a>Créer, afficher et gérer des alertes de journal à l’aide d’Azure Monitor

## <a name="overview"></a>Vue d’ensemble

Les alertes de journal permettent aux utilisateurs d’utiliser une requête [Log Analytics](../log-query/log-analytics-tutorial.md) pour évaluer les journaux de ressources à chaque fréquence définie, et de déclencher une alerte en fonction des résultats. Les règles peuvent déclencher une ou plusieurs actions à l’aide des [groupes d’actions](./action-groups.md). [En savoir plus sur les fonctionnalités et la terminologie des alertes de journal](alerts-unified-log.md).

Cet article explique comment créer et gérer des alertes de journal à l’aide d’Azure Monitor. Les règles d’alerte sont définies par trois composants :
- Cible : ressource Azure spécifique à surveiller.
- Critères : logique à évaluer. Si les critères sont réunis, l’alerte se déclenche.  
- Action : notifications ou automation, e-mail, SMS, webhook, etc.

Vous pouvez également créer des règles d’alerte de journal à l’aide de modèles Azure Resource Manager qui sont décrits dans un [autre article](alerts-log-create-templates.md).

> [!NOTE]
> Les données de journal d’un [espace de travail Log Analytics](../log-query/log-analytics-tutorial.md) peuvent être envoyées au magasin de métriques Azure Monitor. Les alertes de métriques ont [un comportement différent](alerts-metric-overview.md), qui peut être plus adapté en fonction des données que vous utilisez. Pour savoir ce que sont les journaux et comment les acheminer vers les bases de données de métriques, voir [Créer des alertes de métriques de journaux d’activité dans Azure Monitor](alerts-metric-logs.md).

## <a name="create-a-log-alert-rule-with-the-azure-portal"></a>Créer une règle d’alerte de journal avec le portail Azure

Pour commencer à écrire des requêtes pour les alertes, procédez comme suit :

1. Accédez à la ressource pour laquelle vous souhaitez créer une alerte.
1. Sous **Surveiller**, sélectionnez **Journaux**.
1. Interrogez les données de journal qui peuvent indiquer le problème. Pour savoir ce que vous pouvez découvrir ou [commencer à écrire votre propre requête](../log-query/log-analytics-tutorial.md), vous pouvez utiliser la [rubrique d’exemples de requêtes d’alerte](../log-query/example-queries.md). En outre, [découvrez comment créer des requêtes d’alerte optimisées](alerts-log-query.md).
1. Appuyez sur le bouton « + Nouvelle règle d’alerte » pour démarrer le flux de création d’alerte.

    ![Log Analytics - Définir l’alerte](media/alerts-log/AlertsAnalyticsCreate.png)

> [!NOTE]
> Il est recommandé de créer des alertes à grande échelle lorsque vous utilisez le mode d’accès aux ressources pour des journaux, qui s’exécute sur plusieurs ressources à l’aide d’un groupe de ressources ou d’une étendue d’abonnement. La génération d’alertes à grande échelle réduit la charge de gestion des règles. Pour pouvoir cibler les ressources, incluez la colonne ID de ressource dans les résultats. [Apprenez-en davantage sur le fractionnement des alertes par dimensions.](alerts-unified-log.md#split-by-alert-dimensions)

### <a name="log-alert-for-log-analytics-and-application-insights"></a>Alerte de journal pour Log Analytics et Application Insights

1. Si la syntaxe de requête est correcte, les données historiques de la requête s’affichent sous forme de graphique, avec la possibilité d’ajuster la période du graphique entre les six dernières heures et la dernière semaine.
 
    Si les résultats de votre requête contiennent des données synthétisées ou des colonnes spécifiques d’un [projet](/azure/kusto/query/projectoperator) sans colonne d’heure, le graphique affiche une seule valeur.

    ![Configurer une règle d’alerte](media/alerts-log/AlertsPreviewAlertLog.png)

1. Choisissez l’intervalle de temps pendant lequel évaluer la condition spécifiée à l’aide de l’option [**Période**](alerts-unified-log.md#query-time-range).

1. Les alertes de journal peuvent reposer sur deux types de [**Mesures**](alerts-unified-log.md#measure) :
    1. **Nombre de résultats** – Nombre d’enregistrements que retourne la requête.
    1. **Mesure métrique** - *Valeur agrégée* calculée à l’aide d’une synthèse regroupée par expressions choisies et sélection de la fonction [bin()](/azure/kusto/query/binfunction). Par exemple :

    ```Kusto
    // Reported errors
    union Event, Syslog // Event table stores Windows event records, Syslog stores Linux records
    | where EventLevelName == "Error" // EventLevelName is used in the Event (Windows) records
    or SeverityLevel== "err" // SeverityLevel is used in Syslog (Linux) records
    | summarize AggregatedValue = count() by Computer, bin(TimeGenerated, 15m)
    ```

1. Pour la logique d’alerte des mesures métriques, vous pouvez éventuellement spécifier la façon de [fractionner les alertes par dimensions](alerts-unified-log.md#split-by-alert-dimensions) à l’aide de l’option **Agréger sur**. L’expression de regroupement de lignes doit être unique et triée.

    > [!NOTE]
    > Comme la fonction [bin()](/azure/kusto/query/binfunction) peut entraîner des intervalles de temps inégaux, le service d’alerte convertit automatiquement la fonction [bin()](/azure/kusto/query/binfunction) en fonction [bin_at()](/azure/kusto/query/binatfunction) avec l’heure appropriée au moment de l’exécution, pour garantir des résultats avec un point fixe.

    > [!NOTE]
    > La commande Fractionner par dimensions d’alerte n’est disponible que pour l’API scheduledQueryRules actuelle. Si vous utilisez l'[API d'alerte Log Analytics](api-alerts.md) héritée, vous devrez opérer un basculement. [En savoir plus sur le basculement](./alerts-log-api-switch.md) Les alertes centrées sur les ressources à grande échelle ne sont prises en charge que dans les versions `2020-05-01-preview` et ultérieures de l’API.

    ![Option Agréger sur](media/alerts-log/aggregate-on.png)

1. Ensuite, en fonction du jeu de données de l’aperçu, définissez l’[**Opérateur**, la **Valeur de seuil**](alerts-unified-log.md#threshold-and-operator) et la [**Fréquence**](alerts-unified-log.md#frequency).

1. Vous pouvez également définir le [nombre de violations pour déclencher une alerte](alerts-unified-log.md#number-of-violations-to-trigger-alert) à l’aide **Total des violations ou Violations consécutives**.

1. Sélectionnez **Terminé**. 

1. Définissez le **Nom de la règle d’alerte**, un **Description**, puis sélectionnez la **Gravité** de l’alerte. Ces détails sont utilisés dans toutes les actions d’alerte. En outre, vous pouvez choisir de ne pas activer la règle d’alerte lors de la création en sélectionnant l’option **Activer la règle lors de sa création**.

1. Indiquez si vous souhaitez supprimer des actions de règle pour une heure postérieure au déclenchement d’une alerte, en utilisant l’option [**Supprimer les alertes**](alerts-unified-log.md#state-and-resolving-alerts). La règle s’exécutera et créera des alertes, mais aucune action ne sera déclenchée pour empêcher le bruit. Pour être effective, la valeur de Mettre les actions en sourdine doit être supérieure à la fréquence d’alerte.

    ![Supprimer des alertes de journal](media/alerts-log/AlertsPreviewSuppress.png)

1. Spécifiez si la règle d’alerte doit déclencher un ou plusieurs [**Croupes d’actions**](action-groups.md#webhook) lorsque la condition d’alerte est remplie.

    > [!NOTE]
    > Consultez l’article [Limites du service d’abonnement Azure](../../azure-resource-manager/management/azure-subscription-service-limits.md) pour connaître les limites des actions pouvant être effectuées.  

1. Vous pouvez éventuellement personnaliser des actions dans les règles d’alerte de journal :

    - **Objet de l’e-mail personnalisé** : remplace l’*objet de l’e-mail* des actions de courrier. Vous ne pouvez pas modifier le corps du message et ce champ **n’est pas destiné aux adresses e-mail**.
    - **Inclure la charge utile JSON personnalisée** : remplace le JSON de webhook utilisé par les groupes d’actions, si le groupe d’actions contient une action webhook. Apprenez-en davantage sur les [actions webhook pour les alertes de journal](./alerts-log-webhook.md).

    ![Remplacements d’actions pour les alertes de journal](media/alerts-log/AlertsPreviewOverrideLog.png)

1. Si tous les champs sont correctement définis, vous pouvez cliquer sur le bouton **Créer une règle d’alerte** pour créer une alerte.

    Après quelques minutes, l’alerte est active et se déclenche comme décrit précédemment.

    ![Création de règles](media/alerts-log/AlertsPreviewCreate.png)

#### <a name="creating-log-alert-for-log-analytics-and-application-insights-from-the-alerts-management"></a>Création d’alerte de journal pour Log Analytics et Application Insights à partir de la gestion des alertes

> [!NOTE]
> La création à partir de la gestion des alertes n’est actuellement pas prise en charge pour les journaux relatifs aux ressources.

1. Dans le [portail](https://portal.azure.com/), sélectionnez **Surveiller**, puis **Alertes**.

    ![Surveillance](media/alerts-log/AlertsPreviewMenu.png)

1. Sélectionnez **Nouvelle règle d’alerte**. 

    ![Ajouter une alerte](media/alerts-log/AlertsPreviewOption.png)

1. Le volet **Créer une alerte** s’affiche. Il se compose de quatre parties : 
    - la ressource à laquelle l’alerte s’applique ;
    - la condition à vérifier ;
    - l’action à exécuter lorsque la condition est remplie.
    - Le nom et la description de l’alerte 

    ![Créer une règle](media/alerts-log/AlertsPreviewAdd.png)

1. Appuyez sur le bouton **Sélectionner une ressource**. Filtrez en choisissant l’*Abonnement*, le *Type de ressource*, puis sélectionnez une ressource. Vérifiez que la ressource a des journaux disponibles.

   ![Sélectionner une ressource](media/alerts-log/Alert-SelectResourceLog.png)

1. Utilisez ensuite le bouton **Condition** pour afficher la liste des options de signal disponibles pour la ressource. Sélectionnez l’option **Recherche personnalisée dans les journaux**.

   ![Sélectionner une ressource : recherche de journal personnalisée](media/alerts-log/AlertsPreviewResourceSelectionLog.png)

   > [!NOTE]
   > Le portail des alertes répertorie les requêtes enregistrées de Log Analytics et d’Application Insights, qui peuvent être utilisées comme modèles de requêtes d’alerte.

1. Après avoir opéré votre sélection, écrivez, collez ou modifiez la requête d’alerte dans le champ **Rechercher la requête**.

1. Passez aux étapes suivantes décrites dans la [dernière section](#log-alert-for-log-analytics-and-application-insights).

### <a name="log-alert-for-all-other-resource-types"></a>Alerte de journal pour tous les autres types de ressources

> [!NOTE]
> Il n’existe actuellement aucun frais supplémentaire pour la version `2020-05-01-preview` de l’API et les alertes de journal centrées sur les ressources.  La tarification des fonctionnalités en préversion sera annoncée à l’avenir et un avis sera fourni avant le début de la facturation. Si vous choisissez de continuer à utiliser la nouvelle version de l’API et les alertes de journal centrées sur les ressources après la période de notification, vous serez facturé au tarif en vigueur.

1. Démarrez à partir de l’onglet **Condition** :

    1. Vérifiez que les valeurs [**Mesure**](alerts-unified-log.md#measure), [**Type d’agrégation**](alerts-unified-log.md#aggregation-type) et [**Granularité d’agrégation**](alerts-unified-log.md#aggregation-granularity) sont correctes. 
        1. Par défaut, la règle compte le nombre de résultats au cours des 5 dernières minutes.
        1. Si nous détectons des résultats de requête résumés, la règle est automatiquement mise à jour en quelques secondes pour capturer cela.

    1. Si nécessaire, choisissez un [fractionnement des alertes par dimensions](alerts-unified-log.md#split-by-alert-dimensions) : 
       - Si elle est détectée, la **colonne ID de la ressource**, est sélectionnée automatiquement, et remplace le contexte de l’alerte déclenchée par la ressource de l’enregistrement. 
       - Vous pouvez désactiver la **colonne ID de la ressource** pour déclencher des alertes sur des groupes d’abonnements ou de ressources. Cette désactivation est utile lorsque les résultats de requête sont basés sur des requêtes interressources. C’est le cas, par exemple, d’une requête qui vérifie si 80 % des machines virtuelles du groupe de ressources sont confrontées à une utilisation élevée du processeur.
       - Vous pouvez également sélectionner jusqu’à six fractionnements supplémentaires pour tout type de colonne de nombre ou de texte à l’aide de la table de dimensions.
       - Les alertes sont déclenchées séparément conformément au fractionnement basé sur des combinaisons uniques, et la charge utile d’alerte contient ces informations.
    
        ![Sélectionner les paramètres d’agrégation et le fractionnement](media/alerts-log/select-aggregation-parameters-and-splitting.png)

    1. Le graphique **Aperçu** affiche les résultats des évaluations de requête dans le temps. Vous pouvez modifier la période du graphique ou sélectionner différentes séries chronologiques résultant du fractionnement des alertes par dimensions.

        ![Graphique en préversion](media/alerts-log/preview-chart.png)

    1. Ensuite, en fonction des données d’aperçu, définissez la **Logique d’alerte** : [**Opérateur**, **Valeur de seuil**](alerts-unified-log.md#threshold-and-operator) et [**Fréquence**](alerts-unified-log.md#frequency).

        ![Graphique Aperçu avec seuil et logique d’alerte](media/alerts-log/chart-and-alert-logic.png)

    1. Vous pouvez éventuellement définir le [**Nombre de violations à partir duquel déclencher l’alerte**](alerts-unified-log.md#number-of-violations-to-trigger-alert) dans la section **Options avancées**.
    
        ![Options avancées](media/alerts-log/advanced-options.png)

1. Sous l’onglet **Actions**, sélectionnez ou créez les [groupes d’actions](action-groups.md) requis.

    ![Onglet Actions](media/alerts-log/actions-tab.png)

1. Sous l’onglet **Détails**, définissez les **Détails des règles d’alerte** et les **Détails du projet**. Vous pouvez éventuellement activer ou désactiver les options **Démarrer l’exécution maintenant** ou [**Mettre les actions en sourdine**](alerts-unified-log.md#state-and-resolving-alerts) pendant une période après le déclenchement de la règle d’alerte.

    > [!NOTE]
    > Les règles d’alerte de journal sont actuellement sans état et déclenchent une action chaque fois qu’une alerte est créée, sauf si la mise en sourdine est définie.

    ![Onglet Détails](media/alerts-log/details-tab.png)

1. Sous l’onglet **Balises**, définissez les balises requises sur la ressource de règle d’alerte.

    ![Onglet balises](media/alerts-log/tags-tab.png)

1. Sous l’onglet **Vérifier + créer**, une validation est effectuée et vous informe de problèmes éventuels. Examinez et approuvez la définition de la règle.
1. Si tous les champs sont corrects, sélectionnez le bouton **Créer** et terminez la création de la règle d’alerte. Vous pouvez examiner toutes les alertes à partir de Gestion des alertes.
 
    ![Onglet Vérifier et créer](media/alerts-log/review-and-create-tab.png)

## <a name="view--manage-log-alerts-in-azure-portal"></a>Afficher et gérer les alertes de journal dans le portail Azure

1. Dans le [portail](https://portal.azure.com/), sélectionnez la ressource appropriée ou le service **Surveiller**. Sélectionnez ensuite **Alertes** dans la section Surveiller.

1. La gestion des alertes affiche toutes les alertes déclenchées. [Apprenez-en davantage sur la gestion des alertes](alerts-managing-alert-instances.md).

    > [!NOTE]
    > Les règles d’alerte de journal sont actuellement [sans état et ne résolvent pas](alerts-unified-log.md#state-and-resolving-alerts).

1. Pour modifier les règles, sélectionnez le bouton **Gérer les règles d’alerte** dans la barre supérieure :

    ![ gérer les règles d’alerte](media/alerts-log/manage-alert-rules.png)

## <a name="managing-log-alerts-using-powershell"></a>Gestion des alertes de journal à l’aide de PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

> [!NOTE]
> PowerShell n’est pas actuellement pris en charge dans la version de l’API `2020-05-01-preview`

Les cmdlets PowerShell répertoriées ci-dessous sont disponibles pour gérer les règles avec l’[API de règles de requêtes planifiées](/rest/api/monitor/scheduledqueryrules/).

- [New-AzScheduledQueryRule](/powershell/module/az.monitor/new-azscheduledqueryrule): cmdlet PowerShell permettant de créer une règle d’alerte de journal.
- [Set-AzScheduledQueryRule](/powershell/module/az.monitor/set-azscheduledqueryrule): cmdlet PowerShell permettant de mettre à jour une règle d’alerte de journal existante.
- [New-AzScheduledQueryRuleSource](/powershell/module/az.monitor/new-azscheduledqueryrulesource): cmdlet PowerShell permettant de créer ou de mettre à jour un objet spécifiant les paramètres sources pour une alerte de journal. Utilisé comme entrée par l’applet de commande [New-AzScheduledQueryRule](/powershell/module/az.monitor/new-azscheduledqueryrule) et [Set-AzScheduledQueryRule](/powershell/module/az.monitor/set-azscheduledqueryrule).
- [New-AzScheduledQueryRuleSchedule](/powershell/module/az.monitor/new-azscheduledqueryruleschedule) : cmdlet PowerShell permettant de créer ou de mettre à jour un objet spécifiant les paramètres de planification pour une alerte de journal. Utilisé comme entrée par l’applet de commande [New-AzScheduledQueryRule](/powershell/module/az.monitor/new-azscheduledqueryrule) et [Set-AzScheduledQueryRule](/powershell/module/az.monitor/set-azscheduledqueryrule).
- [New-AzScheduledQueryRuleAlertingAction](/powershell/module/az.monitor/new-azscheduledqueryrulealertingaction): cmdlet PowerShell permettant de créer ou de mettre à jour un objet spécifiant les paramètres d’action pour une alerte de journal. Utilisé comme entrée par l’applet de commande [New-AzScheduledQueryRule](/powershell/module/az.monitor/new-azscheduledqueryrule) et [Set-AzScheduledQueryRule](/powershell/module/az.monitor/set-azscheduledqueryrule).
- [New-AzScheduledQueryRuleAznsActionGroup](/powershell/module/az.monitor/new-azscheduledqueryruleaznsactiongroup): cmdlet PowerShell permettant de créer ou de mettre à jour un objet spécifiant les paramètres des groupes d’actions pour une alerte de journal. Utilisé comme entrée par l’applet de commande [New-AzScheduledQueryRuleAlertingAction](/powershell/module/az.monitor/new-azscheduledqueryrulealertingaction).
- [New-AzScheduledQueryRuleTriggerCondition](/powershell/module/az.monitor/new-azscheduledqueryruletriggercondition): cmdlet PowerShell permettant de créer ou de mettre à jour un objet spécifiant les paramètres de condition de déclenchement pour une alerte de journal. Utilisé comme entrée par l’applet de commande [New-AzScheduledQueryRuleAlertingAction](/powershell/module/az.monitor/new-azscheduledqueryrulealertingaction).
- [New-AzScheduledQueryRuleLogMetricTrigger](/powershell/module/az.monitor/new-azscheduledqueryrulelogmetrictrigger): cmdlet PowerShell permettant de créer ou de mettre à jour un objet spécifiant les paramètres de condition de déclenchement de métrique pour une [alerte de journal de type mesure de métriques](./alerts-unified-log.md#calculation-of-measure-based-on-a-numeric-column-such-as-cpu-counter-value). Utilisé comme entrée par l’applet de commande [New-AzScheduledQueryRuleTriggerCondition](/powershell/module/az.monitor/new-azscheduledqueryruletriggercondition).
- [Get-AzScheduledQueryRule](/powershell/module/az.monitor/get-azscheduledqueryrule): cmdlet PowerShell permettant de répertorier les règles d’alerte de journal existantes ou une règle d’alerte de journal spécifique.
- [Update-AzScheduledQueryRule](/powershell/module/az.monitor/update-azscheduledqueryrule): cmdlet PowerShell permettant d’activer ou de désactiver une règle d’alerte de journal.
- [Remove-AzScheduledQueryRule](/powershell/module/az.monitor/remove-azscheduledqueryrule) : cmdlet PowerShell permettant de supprimer une règle d’alerte de journal existante.

> [!NOTE]
> Les cmdlets PowerShell ScheduledQueryRules peuvent uniquement gérer des règles créées dans l’[API de règles de requêtes planifiées](/rest/api/monitor/scheduledqueryrules/) actuelle. Les règles d’alerte de journal créées à l’aide de l’[API d’alerte Log Analytics](api-alerts.md) héritée ne peuvent être gérées qu’à l’aide de PowerShell uniquement après [basculement vers l’API de règles de requêtes planifiées](alerts-log-api-switch.md).

Voici des exemples d’étapes de création d’une règle d’alerte de journal utilisant PowerShell :

```powershell
$source = New-AzScheduledQueryRuleSource -Query 'Heartbeat | summarize AggregatedValue = count() by bin(TimeGenerated, 5m), _ResourceId' -DataSourceId "/subscriptions/a123d7efg-123c-1234-5678-a12bc3defgh4/resourceGroups/contosoRG/providers/microsoft.OperationalInsights/workspaces/servicews"

$schedule = New-AzScheduledQueryRuleSchedule -FrequencyInMinutes 15 -TimeWindowInMinutes 30

$metricTrigger = New-AzScheduledQueryRuleLogMetricTrigger -ThresholdOperator "GreaterThan" -Threshold 2 -MetricTriggerType "Consecutive" -MetricColumn "_ResourceId"

$triggerCondition = New-AzScheduledQueryRuleTriggerCondition -ThresholdOperator "LessThan" -Threshold 5 -MetricTrigger $metricTrigger

$aznsActionGroup = New-AzScheduledQueryRuleAznsActionGroup -ActionGroup "/subscriptions/a123d7efg-123c-1234-5678-a12bc3defgh4/resourceGroups/contosoRG/providers/microsoft.insights/actiongroups/sampleAG" -EmailSubject "Custom email subject" -CustomWebhookPayload "{ `"alert`":`"#alertrulename`", `"IncludeSearchResults`":true }"

$alertingAction = New-AzScheduledQueryRuleAlertingAction -AznsAction $aznsActionGroup -Severity "3" -Trigger $triggerCondition

New-AzScheduledQueryRule -ResourceGroupName "contosoRG" -Location "Region Name for your Application Insights App or Log Analytics Workspace" -Action $alertingAction -Enabled $true -Description "Alert description" -Schedule $schedule -Source $source -Name "Alert Name"
```

Vous pouvez également créer l’alerte de journal en utilisant [un modèle et des fichiers de paramètres](./alerts-log-create-templates.md) à l’aide de PowerShell :

```powershell
Connect-AzAccount

Select-AzSubscription -SubscriptionName <yourSubscriptionName>

New-AzResourceGroupDeployment -Name AlertDeployment -ResourceGroupName ResourceGroupofTargetResource `
  -TemplateFile mylogalerttemplate.json -TemplateParameterFile mylogalerttemplate.parameters.json
```

## <a name="managing-log-alerts-using-cli"></a>Gestion des alertes de journal à l’aide de l’interface de ligne de commande

> [!NOTE]
> La prise en charge d’Azure CLI n’est disponible que pour l’API scheduledQueryRules versions `2020-05-01-preview` et ultérieures. La version précédente de l’API peut utiliser l’interface de ligne de commande Azure Resource Manager avec des modèles, comme décrit ci-dessous. Si vous utilisez l’[API d’alerte Log Analytics](api-alerts.md) héritée, vous devrez opérer un basculement vers l’utilisation de l’interface de ligne de commande. [En savoir plus sur le basculement](./alerts-log-api-switch.md).

Les sections précédentes décrivaient comment créer, afficher et gérer des règles d’alerte de journal à l’aide du portail Azure. Cette section décrit comment faire de même à l’aide de l’[interface de ligne de commande Azure](/cli/azure/get-started-with-azure-cli) multiplateforme. Le plus rapide pour commencer à utiliser Azure CLI est de s’appuyer sur [Azure Cloud Shell](../../cloud-shell/overview.md). Pour cet article, nous utiliserons le service Cloud Shell.

1. Accédez au portail Azure, puis sélectionnez **Cloud Shell**.

1. À l’invite, vous pouvez utiliser des commandes avec l’option ``--help`` pour en savoir plus sur la commande et comment l’utiliser. Par exemple, la commande suivante affiche la liste des commandes disponibles pour la création, l’affichage et la gestion des alertes de journal :

    ```azurecli
    az monitor scheduled-query --help
    ```

1. Vous pouvez créer une règle d’alerte de journal qui surveille le nombre d’erreurs d’événement système :

    ```azurecli
    az monitor scheduled-query create -g {ResourceGroup} -n {nameofthealert} --scopes {vm_id} --condition "count \'union Event, Syslog | where TimeGenerated > ago(1h) | where EventLevelName == \"Error\" or SeverityLevel== \"err\"\' > 2" --description {descriptionofthealert}
    ```

1. Vous pouvez afficher toutes les alertes de journal dans un groupe de ressources à l’aide de la commande suivante :

    ```azurecli
    az monitor scheduled-query list -g {ResourceGroup}
    ```

1. Vous pouvez voir les détails d’une règle d’alerte de journal spécifique en utilisant le nom ou l’ID de ressource de la règle :

    ```azurecli
    az monitor scheduled-query show -g {ResourceGroup} -n {AlertRuleName}
    ```

    ```azurecli
    az monitor scheduled-query show --ids {RuleResourceId}
    ```

1. Vous pouvez désactiver une règle d’alerte de journal à l’aide de la commande suivante :

    ```azurecli
    az monitor scheduled-query update -g {ResourceGroup} -n {AlertRuleName} --enabled false
    ```

1. Vous pouvez supprimer une règle d’alerte de journal à l’aide de la commande suivante :

    ```azurecli
    az monitor scheduled-query delete -g {ResourceGroup} -n {AlertRuleName}
    ```

Vous pouvez également utiliser l’interface de ligne de commande d’Azure Resource Manager avec des [modèles](./alerts-log-create-templates.md) de fichiers :

```azurecli
az login

az deployment group create \
    --name AlertDeployment \
    --resource-group ResourceGroupofTargetResource \
    --template-file mylogalerttemplate.json \
    --parameters @mylogalerttemplate.parameters.json
```

En cas de réussite de la création, le code d’état 201 est retourné. En cas de réussite de la mise à jour, le code d’état 200 est retourné.

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les [alertes de journal](./alerts-unified-log.md).
* Créer des alertes de journal à l’aide de [modèles Azure Resource Manager](./alerts-log-create-templates.md).
* Comprendre les [actions webhook pour les alertes de journal](./alerts-log-webhook.md).
* Découvrez plus en détail les [requêtes dans les journaux](../log-query/log-query-overview.md).