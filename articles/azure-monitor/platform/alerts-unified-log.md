---
title: Alertes de journal dans Azure Monitor
description: Déclenchez des e-mails, des notifications, des URL de sites web d'appel (webhooks) ou une automatisation lorsque la condition de la requête de journal que vous spécifiez est remplie
author: yanivlavi
ms.author: yalavi
ms.topic: conceptual
ms.date: 5/31/2019
ms.subservice: alerts
ms.openlocfilehash: 9f8004b41e8048dfc97fb61bb67a634963c0c575
ms.sourcegitcommit: e5f9126c1b04ffe55a2e0eb04b043e2c9e895e48
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96317552"
---
# <a name="log-alerts-in-azure-monitor"></a>Alertes de journal dans Azure Monitor

## <a name="overview"></a>Vue d’ensemble

Les alertes de journal sont l’un des types d’alerte pris en charge dans [Alertes Azure](./alerts-overview.md). Les alertes de journal permettent aux utilisateurs d’utiliser une requête [Log Analytics](../log-query/log-analytics-tutorial.md) pour évaluer les journaux de ressources à chaque fréquence définie, et de déclencher une alerte en fonction des résultats. Les règles peuvent déclencher une ou plusieurs actions à l’aide des [groupes d’actions](./action-groups.md).

> [!NOTE]
> Les données de journal d'un [espace de travail Log Analytics](../log-query/log-analytics-tutorial.md) peuvent être envoyées au magasin de métriques Azure Monitor. Les alertes de métriques ont [un comportement différent](alerts-metric-overview.md), qui peut être plus adapté en fonction des données que vous utilisez. Pour savoir ce que sont les journaux et comment les acheminer vers les bases de données de métriques, voir [Créer des alertes de métriques de journaux d’activité dans Azure Monitor](alerts-metric-logs.md).

> [!NOTE]
> Il n’existe actuellement aucun frais supplémentaire pour la version `2020-05-01-preview` de l’API et les alertes de journal centrées sur les ressources.  La tarification des fonctionnalités en préversion sera annoncée à l’avenir et un avis sera fourni avant le début de la facturation. Si vous choisissez de continuer à utiliser la nouvelle version de l’API et les alertes de journal centrées sur les ressources après la période de notification, vous serez facturé au tarif en vigueur.

## <a name="prerequisites"></a>Prérequis

Les alertes de journal exécutent des requêtes sur les données Log Analytics. Vous devez d'abord [collecter les données de journal](resource-logs.md), puis les interroger pour détecter les problèmes. Vous pouvez utiliser la [rubrique d'exemples de requêtes d'alerte](../log-query/example-queries.md) de Log Analytics pour en savoir plus sur ce que vous pouvez découvrir ou [commencer à écrire votre propre requête](../log-query/log-analytics-tutorial.md).

[Contributeur de surveillance Azure](./roles-permissions-security.md) est un rôle courant qui est nécessaire pour créer, modifier et mettre à jour les alertes de journal. Des droits d'accès et d'exécution de requêtes sont également nécessaires pour les journaux de ressources. Un accès partiel aux journaux de ressources peut faire échouer des requêtes ou renvoyer des résultats partiels. [Découvrez-en plus sur la configuration des alertes de journal dans Azure](./alerts-log.md).

> [!NOTE]
> Les alertes de journal de Log Analytics étaient auparavant gérées à l'aide de l'[API d'alerte Log Analytics](api-alerts.md) héritée. [En savoir plus sur le basculement sur l’API ScheduledQueryRules actuelle](alerts-log-api-switch.md).

## <a name="query-evaluation-definition"></a>Définition de l'évaluation des requêtes

La définition des conditions des règles de recherche dans les journaux commence par ceci :

- Quelle requête exécuter ?
- Comment utiliser les résultats ?

Les sections suivantes décrivent les différents paramètres que vous pouvez utiliser pour définir la logique ci-dessus.

### <a name="log-query"></a>Requête de journal
Requête [Log Analytics](../log-query/log-analytics-tutorial.md) utilisée pour évaluer la règle. Les résultats renvoyés par cette requête permettent de déterminer si une alerte doit être déclenchée. La requête peut être étendue à :

- Une ressource spécifique, telle qu'une machine virtuelle.
- Une ressource à grande échelle, comme un abonnement ou un groupe de ressources.
- Plusieurs ressources à l'aide d'une [requête inter-ressources](../log-query/cross-workspace-query.md#querying-across-log-analytics-workspaces-and-from-application-insights). 
 
> [!IMPORTANT]
> Des contraintes s'appliquent aux requêtes d'alerte pour garantir des performances optimales et des résultats pertinents. [En savoir plus ici](./alerts-log-query.md).

> [!IMPORTANT]
> Les requêtes centrées sur les ressources et les [requêtes inter-ressources](../log-query/cross-workspace-query.md#querying-across-log-analytics-workspaces-and-from-application-insights) sont uniquement prises en charge à l'aide de l'API SchedulerQueryRules actuelle. Si vous utilisez l'[API d'alerte Log Analytics](api-alerts.md) héritée, vous devrez opérer un basculement. [En savoir plus sur le basculement](./alerts-log-api-switch.md)

#### <a name="query-time-range"></a>Intervalle de temps de requête

L'intervalle de temps est déterminé dans la définition de la condition de la règle. Dans les espaces de travail et Application Insights, cet intervalle est désigné sous le nom de **Période**. Dans tous les autres types de ressources, il s'agit de **Remplacer l'intervalle de temps de requête**.

Comme dans Log Analytics, l'intervalle de temps limite les données de requête à la plage spécifiée. Même si la commande **ago** est utilisée dans la requête, l’intervalle de temps s’applique.

Par exemple, une requête analyse 60 minutes lorsque l’intervalle de temps est de 60 minutes, et ce, même si le texte contient **ago(1d)** . L’intervalle de temps et le filtrage du temps de requête doivent correspondre. Dans cet exemple, la modification de la **période** / **Remplacer l’intervalle de temps de la requête** à une journée fonctionnera comme prévu.

### <a name="measure"></a>Measure

Les alertes de journal transforment le journal en valeurs numériques qui peuvent être évaluées. Vous pouvez mesurer deux choses :

#### <a name="count-of-the-results-table-rows"></a>Nombre de lignes de tableau dans les résultats

Le nombre de résultats est la mesure par défaut. Idéal pour utiliser des événements tels que les journaux d'événements Windows, les journaux d'activité Syslog et les exceptions d'application. Se déclenche lorsque des enregistrements de journal se produisent ou ne se produisent pas dans la fenêtre de temps évaluée.

Les alertes de journal fonctionnent mieux lorsque vous tentez de détecter des données dans le journal. Cela fonctionne moins bien lorsque vous tentez de détecter un manque de données dans les journaux. Par exemple, l’alerte sur la pulsation de la machine virtuelle.

Pour les espaces de travail et Application Insights, il s'appelle **Basé sur** avec la sélection **Nombre de résultats**. Dans tous les autres types de ressources, il s'agit de **Mesure** avec la sélection **Lignes de tableau**.

> [!NOTE]
> Comme les journaux sont des données semi-structurées, ils sont intrinsèquement plus latents que les métriques. Vous pouvez rencontrer des ratés lorsque vous essayez de détecter une absence de données dans les journaux, et vous devez envisager d'utiliser des [alertes liées aux métriques](alerts-metric-overview.md). Vous pouvez envoyer des données au magasin de métriques à partir de journaux à l’aide d’[alertes de métrique pour les journaux](alerts-metric-logs.md).

##### <a name="example-of-results-table-rows-count-use-case"></a>Exemple de cas d'usage du nombre de lignes de tableau dans les résultats

Vous souhaitez savoir quand votre application a répondu avec le code d'erreur 500 (Erreur interne du serveur). Il conviendrait de créer une règle d’alerte paramétrée comme suit :

- **Requête :** 

```Kusto
requests
| where resultCode == "500"
```

- **Période / Précision d’agrégation :** 15 minutes
- **Fréquence des alertes :** 15 minutes
- **Valeur de seuil :** Supérieur à 0

Les règles d'alerte surveillent ensuite toutes les demandes qui se terminent par le code d'erreur 500. La requête s'exécute toutes les 15 minutes, au cours des 15 dernières minutes. Même si un seul enregistrement est trouvé, elle déclenche l'alerte et lance les actions configurées.

#### <a name="calculation-of-measure-based-on-a-numeric-column-such-as-cpu-counter-value"></a>Calcul de la mesure basé sur une colonne numérique (telle que la valeur du compteur du processeur)

Pour les espaces de travail et Application Insights, il s'appelle **Basé sur** avec la sélection **Mesure métrique**. Dans tous les autres types de ressources, il s'agit de **Mesure** avec la sélection du nom d'une colonne numérique.

### <a name="aggregation-type"></a>Type d’agrégation

Le calcul est effectué sur plusieurs enregistrements pour les agréger en une seule valeur numérique. Par exemple :
- **Count** renvoie le nombre d'enregistrements de la requête
- **Average** renvoie la moyenne de la colonne de mesure [**Précision d'agrégation**](#aggregation-granularity) définie.

Dans les espaces de travail et Application Insights, il est uniquement pris en charge dans le type de mesure **Mesure métrique**. Le résultat de la requête doit contenir une colonne appelée AggregatedValue qui fournit une valeur numérique après une agrégation définie par l'utilisateur. Dans tous les autres types de ressources, **Type d'agrégation** est sélectionné dans le champ de ce nom.

### <a name="aggregation-granularity"></a>Précision d'agrégation

Détermine l'intervalle utilisé pour agréger plusieurs enregistrements en une valeur numérique. Par exemple, si vous avez spécifié **5 minutes**, les enregistrements seront regroupés par intervalles de 5 minutes à l'aide du **Type d'agrégation** spécifié.

Dans les espaces de travail et Application Insights, il est uniquement pris en charge dans le type de mesure **Mesure métrique**. Le résultat de la requête doit contenir la fonction [bin()](/azure/kusto/query/binfunction) qui définit l'intervalle. Dans tous les autres types de ressources, le champ qui contrôle ce paramètre est appelé **Précision d'agrégation**.

> [!NOTE]
> Comme la fonction [bin()](/azure/kusto/query/binfunction) peut entraîner des intervalles de temps inégaux, le service d'alerte convertira automatiquement la fonction [bin()](/azure/kusto/query/binfunction) en fonction [bin_at()](/azure/kusto/query/binatfunction) avec l'heure appropriée au moment de l'exécution, pour garantir des résultats avec un point fixe.

### <a name="split-by-alert-dimensions"></a>Fractionner par dimensions d'alerte

Fractionnez les alertes par colonnes numériques ou de chaînes en alertes distinctes en les regroupant en combinaisons uniques. Lors de la création d'alertes centrées sur les ressources à grande échelle (étendue d'abonnement ou de groupe de ressources), vous pouvez fractionner par colonne d'ID de ressource Azure. Le fractionnement sur la colonne d'ID de ressource Azure remplacera la cible de l'alerte par la ressource spécifiée.

Dans les espaces de travail et Application Insights, il est uniquement pris en charge dans le type de mesure **Mesure métrique**. Le champ s'appelle **Agréger sur**. Il est limité à trois colonnes. La présence de plus de trois groupes par colonnes dans la requête peut donner des résultats inattendus. Dans tous les autres types de ressources, il est configuré dans la section **Fractionner par dimensions** de la condition (limitée à six fractionnements).

#### <a name="example-of-splitting-by-alert-dimensions"></a>Exemple de fractionnement par dimensions d'alerte

Par exemple, vous souhaitez analyser les erreurs de plusieurs machines virtuelles exécutant votre application/site web dans un groupe de ressources spécifique. Vous pouvez le faire à l'aide d'une règle d'alerte de journal, comme suit :

- **Requête :** 

    ```Kusto
    // Reported errors
    union Event, Syslog // Event table stores Windows event records, Syslog stores Linux records
    | where EventLevelName == "Error" // EventLevelName is used in the Event (Windows) records
    or SeverityLevel== "err" // SeverityLevel is used in Syslog (Linux) records
    ```

    Lorsque vous utilisez des espaces de travail et Application Insights avec la logique d'alerte **Mesure métrique**, cette ligne doit être ajoutée au texte de la requête :

    ```Kusto
    | summarize AggregatedValue = count() by Computer, bin(TimeGenerated, 15m)
    ```

- **Colonne d'ID de ressource :** _ResourceId (le fractionnement par colonne d'ID de ressource dans les règles d'alerte n'est actuellement disponible que pour les abonnements et les groupes de ressources)
- **Dimensions / Agrégé sur :**
  - Computer = VM1, VM2 (Ordinateur = Machine virtuelle 1, Machine virtuelle 2 - Le filtrage des valeurs dans la définition de la règle d'alerte n'est actuellement pas disponible pour les espaces de travail et Application Insights. Il s'effectue dans le texte de la requête.)
- **Période / Précision d’agrégation :** 15 minutes
- **Fréquence des alertes :** 15 minutes
- **Valeur de seuil :** Supérieur à 0

Cette règle détermine si une machine virtuelle a rencontré des erreurs au cours des 15 dernières minutes. Chaque machine virtuelle est analysée séparément et déclenche des actions individuellement.

> [!NOTE]
> Fractionner par dimensions d'alerte est uniquement disponible pour l'API scheduledQueryRules actuelle. Si vous utilisez l'[API d'alerte Log Analytics](api-alerts.md) héritée, vous devrez opérer un basculement. [En savoir plus sur le basculement](./alerts-log-api-switch.md) Les alertes centrées sur les ressources à grande échelle ne sont prises en charge que dans les versions `2020-05-01-preview` et ultérieures de l'API.

## <a name="alert-logic-definition"></a>Définition de la logique d'alerte

Après avoir défini la requête à exécuter et l'évaluation des résultats, vous devez définir la logique d'alerte et le moment où les actions doivent être déclenchées. Les sections suivantes décrivent les différents paramètres que vous pouvez utiliser :

### <a name="threshold-and-operator"></a>Seuil et opérateur

Les résultats de la requête sont transformés en valeur numérique qui est comparée au seuil et à l'opérateur.

### <a name="frequency"></a>Fréquence

Intervalle dans lequel la requête est exécutée. La valeur peut être comprise entre 5 minutes et 1 jour. Elle doit être inférieure ou égale à l'[intervalle de temps de requête](#query-time-range) pour ne pas manquer les enregistrements de journal.

Par exemple, si vous définissez la période sur 30 minutes et la fréquence sur 1 heure :  Si la requête est exécutée à 00h00, les enregistrements compris entre 23h30 et 00h00 sont renvoyés. La requête s'exécute ensuite à 01h00, pour renvoyer les enregistrements compris entre 00h30 et 01h00. Les enregistrements créés entre 00h00 et 00h30 ne sont jamais évalués.

### <a name="number-of-violations-to-trigger-alert"></a>Nombre de violations à partir duquel déclencher l'alerte

Vous pouvez spécifier la période d'évaluation des alertes et le nombre d'échecs nécessaires pour déclencher une alerte. Pour vous permettre de mieux définir un temps d'impact pour déclencher une alerte. 

Par exemple, si votre règle [**Précision d'agrégation**](#aggregation-granularity) est définie sur « 5 minutes », vous ne pouvez déclencher une alerte que si trois échecs (15 minutes) se sont produits au cours de la dernière heure. Ce paramètre est défini par la stratégie d'entreprise de votre application.

## <a name="state-and-resolving-alerts"></a>État et résolution des alertes

Les alertes de journal sont sans état. Les alertes se déclenchent à chaque fois que la condition est remplie, même si elles ont déjà été déclenchées. Les alertes déclenchées ne sont pas résolues. Vous pouvez [marquer l'alerte comme fermée](alerts-managing-alert-states.md). Vous pouvez également désactiver des actions pour les empêcher de se déclencher pendant un certain temps après le déclenchement d'une règle d'alerte.

Dans les espaces de travail et Application Insights, ceci est appelé **Supprimer les alertes**. Dans tous les autres types de ressources, il s'agit de **Mettre les actions en sourdine**. 

Consultez cet exemple d'évaluation d'alerte :

| Temps    | Évaluation de la condition de journal | Résultats 
| ------- | ----------| ----------| ------- 
| 00:05 | false | L'alerte ne se déclenche pas. Aucune action n’est appelée.
| 00:10 | true  | L’alerte se déclenche et les groupes d’actions sont appelés. Nouvel état d'alerte ACTIF.
| 00:15 | true  | L’alerte se déclenche et les groupes d’actions sont appelés. Nouvel état d'alerte ACTIF.
| 00:20 | false | L'alerte ne se déclenche pas. Aucune action n’est appelée. L'état d'alerte précédent reste ACTIF.

## <a name="pricing-and-billing-of-log-alerts"></a>Tarification et facturation des alertes de journal

Pour obtenir des informations de tarification, consultez la page [Tarification Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/). Les alertes de journal sont répertoriées sous le fournisseur de ressources `microsoft.insights/scheduledqueryrules` avec :

- Alertes de journal sur Application Insights, affichées avec le nom exact de la ressource, ainsi que le groupe de ressources et les propriétés de l'alerte.
- Alertes de journal sur Log Analytics, affichées avec le nom exact de la ressource ainsi que le groupe de ressources et les propriétés de l'alerte (création avec l'[API scheduledQueryRules](/rest/api/monitor/scheduledqueryrules)).
- Les alertes de journal créées à partir de l'[API Log Analytics héritée](./api-alerts.md) ne sont pas des [ressources Azure](../../azure-resource-manager/management/overview.md) suivies et ne disposent pas de noms de ressources uniques. Ces alertes sont toujours créées sur `microsoft.insights/scheduledqueryrules` en tant que ressources masquées, avec cette structure de dénomination de ressources `<WorkspaceName>|<savedSearchId>|<scheduleId>|<ActionId>`. Les alertes de journal de l'API héritée sont affichées avec le nom de la ressource masquée ci-dessus, ainsi que le groupe de ressources et les propriétés de l'alerte.

> [!NOTE]
> Les caractères de ressource non pris en charge, tels que `<, >, %, &, \, ?, /`, sont remplacés par `_` dans les noms des ressources masquées et cela se reflète également dans les informations de facturation.

> [!NOTE]
> Les alertes de journal de Log Analytics étaient auparavant gérées à l'aide de l'[API d'alerte Log Analytics](api-alerts.md) héritée et des modèles hérités des [alertes et recherches Log Analytics enregistrées](../insights/solutions.md). [En savoir plus sur le basculement sur l’API ScheduledQueryRules actuelle](alerts-log-api-switch.md). Une règle d'alerte doit être gérée à l'aide de l'[API Log Analytics héritée](api-alerts.md) jusqu'à ce que vous décidiez de basculer et qu'il ne soit plus possible d'utiliser les ressources masquées.

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur la [création d’alertes de journal dans Azure](./alerts-log.md).
* Comprendre les [webhooks dans les alertes de journal dans Azure](alerts-log-webhook.md).
* En savoir plus sur [Alertes Azure](./alerts-overview.md).
* En savoir plus sur [Log Analytics](../log-query/log-query-overview.md).
