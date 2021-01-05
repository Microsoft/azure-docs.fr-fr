---
title: Comprenez le fonctionnement des alertes de métrique dans Azure Monitor.
description: Obtenez un aperçu des actions possibles avec les alertes de métriques et de leur fonctionnement dans Azure Monitor.
ms.date: 09/30/2020
ms.topic: conceptual
ms.subservice: alerts
ms.openlocfilehash: 066ffff5979e658a7c06fe10bd668a9fac839a14
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96460843"
---
# <a name="understand-how-metric-alerts-work-in-azure-monitor"></a>Comprendre le fonctionnement des alertes de métrique dans Azure Monitor

Les alertes de métrique dans Azure Monitor fonctionnent en complément des métriques multidimensionnelles. Il peut s’agir de [métriques de la plateforme](alerts-metric-near-real-time.md#metrics-and-dimensions-supported), de [métriques personnalisées](./metrics-custom-overview.md), de [journaux d’activité courants d’Azure Monitor convertis en métriques](./alerts-metric-logs.md) et de métriques Application Insights. Les alertes de métrique vérifient à intervalles réguliers que les conditions d’une ou de plusieurs séries chronologiques de métriques sont remplies et vous avertissent quand c’est le cas. Les alertes de métrique sont avec état ; autrement dit, elles envoient uniquement des notifications lorsque l’état change.

## <a name="how-do-metric-alerts-work"></a>Fonctionnement des alertes de métrique

Vous pouvez définir une règle d’alerte de métrique en spécifiant une ressource cible à superviser, un nom de métrique, un type de condition (statique ou dynamique) et la condition (un opérateur et un seuil/une sensibilité), ainsi qu’un groupe d’actions à effectuer lorsque la règle d’alerte se déclenche. Les types de condition affectent la manière de déterminer les seuils. [Découvrez-en plus sur le type de condition des seuils dynamiques et les options de sensibilité](alerts-dynamic-thresholds.md).

### <a name="alert-rule-with-static-condition-type"></a>Règle d’alerte avec le type de condition statique

Supposons que vous avez créé une règle d’alerte de métrique de seuil statique simple comme suit :

- Ressource cible (ressource Azure à surveiller) : myVM
- Mesure : Percentage CPU
- Type de condition : statique
- Agrégation du temps (statistique exécutée sur des valeurs de mesures brutes ; Les [agrégations de temps prises en charge](metrics-charts.md#changing-aggregation) sont Min, Max, Moy, Total, Nombre) : Average
- Période (fenêtre de vérification des valeurs de métrique) : Au cours des 5 dernières minutes
- Fréquence (fréquence à laquelle l’alerte de métrique vérifie si les conditions sont remplies) : 1 minute
- Opérateur : Supérieur à
- Seuil : 70

À partir de la création de la règle d’alerte, l’analyse s’exécute chaque minute. Elle examine les valeurs de métrique des 5 dernières minutes et vérifie si la moyenne de ces valeurs est supérieure à 70. Si la condition est remplie (autrement dit, le pourcentage d’UC moyen pour les 5 dernières minutes est supérieur à 70), la règle d’alerte déclenche une notification d’activation. Si vous avez configuré un e-mail ou une action de webhook dans le groupe d’actions associé à la règle d’alerte, vous recevrez une notification sur les deux.

Si vous utilisez plusieurs conditions dans une seule règle, la règle « ajoute » les conditions ensemble. Autrement dit, une alerte se déclenche lorsque toutes les conditions de la règle correspondante sont évaluées comme vraies et résolues lorsque l’une des conditions n’est plus vraie. Un exemple de ce type de règle d’alerte consiste à surveiller une machine virtuelle Azure et à alerter lorsque le « pourcentage d’UC est supérieur à 90 % » et la « longueur de la file d’attente comprend plus de 300 éléments ».

### <a name="alert-rule-with-dynamic-condition-type"></a>Règle d’alerte avec le type de condition dynamique

Supposons que vous avez créé une règle d’alerte de métrique de seuil dynamique simple comme suit :

- Ressource cible (ressource Azure à surveiller) : myVM
- Mesure : Percentage CPU
- Type de condition : Dynamique
- Agrégation du temps (statistique exécutée sur des valeurs de mesures brutes ; Les [agrégations de temps prises en charge](metrics-charts.md#changing-aggregation) sont Min, Max, Moy, Total, Nombre) : Average
- Période (fenêtre de vérification des valeurs de métrique) : Au cours des 5 dernières minutes
- Fréquence (fréquence à laquelle l’alerte de métrique vérifie si les conditions sont remplies) : 1 minute
- Opérateur : Supérieur à
- Sensibilité : Moyenne
- Périodes passées à vérifier : 4
- Nombre de violations : 4

Une fois que la règle d’alerte est créée, l’algorithme de machine learning des seuils dynamiques acquiert les données d’historique disponibles, calcule le seuil qui correspond le mieux au modèle de comportement de la série de métriques et continue d’apprendre de nouvelles données pour rendre le seuil plus précis.

À partir du moment où la règle d’alerte est créée, l’analyse s’exécute toutes les minutes et examine les valeurs de métrique des 20 dernières minutes regroupées par période de 5 minutes pour déterminer si la moyenne des valeurs de chacune des 4 périodes dépasse le seuil attendu. Si la condition est remplie, autrement dit, si le pourcentage d’UC moyen des 20 dernières minutes (quatre périodes de 5 minutes) s’est écarté du comportement attendu à quatre reprises, la règle d’alerte déclenche une notification d’activation. Si vous avez configuré un e-mail ou une action de webhook dans le groupe d’actions associé à la règle d’alerte, vous recevrez une notification sur les deux.

### <a name="view-and-resolution-of-fired-alerts"></a>Affichage et résolution des alertes déclenchées

Les exemples ci-dessus de déclenchement des règles d’alerte sont également consultables dans le portail Azure dans le panneau **Toutes les alertes**.

Par exemple, si l’utilisation de « myVM » demeure au-dessus du seuil lors les vérifications suivantes, la règle d’alerte ne sera pas déclenchée à nouveau tant que les conditions ne seront pas résolues.

Au bout d'un moment, l'utilisation de « myVM » revient à la normale (en dessous du seuil). la règle d’alerte surveille l’état deux fois de plus, puis envoie une notification de résolution. La règle d’alerte envoie un message de résolution/désactivation lorsque la condition d’alerte n’est pas remplie pendant trois périodes consécutives pour réduire le bruit en cas de flottement de conditions.

Comme la notification de résolution est envoyée par le biais de webhooks ou d’un e-mail, l’état de l’instance d’alerte (état de l’analyse) dans le portail Azure est également défini comme résolu.

### <a name="using-dimensions"></a>Utilisation de dimensions

Les alertes de métrique dans Azure Monitor prennent également en charge la surveillance de plusieurs combinaisons de valeurs de dimension avec une seule règle. Essayons de comprendre pourquoi vous pouvez utiliser plusieurs combinaisons de dimension à l’aide d’un exemple.

Supposons que vous avez un plan App Service pour votre site web. Vous souhaitez surveiller l’utilisation du processeur sur plusieurs instances de votre site web/application. Vous pouvez le faire à l’aide d’une règle d’alerte de métrique, comme suit :

- Ressource cible : myAppServicePlan
- Mesure : Percentage CPU
- Type de condition : statique
- Dimensions
  - Instance = InstanceName1, InstanceName2
- Agrégation de temps : Average
- Période : Au cours des 5 dernières minutes
- Fréquence : 1 minute
- Opérateur : GreaterThan
- Seuil : 70

Comme précédemment, cette règle contrôle si l’utilisation moyenne du processeur pendant les 5 dernières minutes est supérieure à 70 %. Toutefois, avec la même règle, vous pouvez surveiller deux instances de votre site web. Chaque instance est surveillée individuellement, et vous recevez des notifications séparées.

Admettons que vous avez une application web soumise à une très forte demande et que vous devez ajouter d’autres instances. Cette règle ne surveille toujours que deux instances. Toutefois, vous pouvez créer une règle comme suit :

- Ressource cible : myAppServicePlan
- Mesure : Percentage CPU
- Type de condition : statique
- Dimensions
  - Instance = *
- Agrégation de temps : Average
- Période : Au cours des 5 dernières minutes
- Fréquence : 1 minute
- Opérateur : GreaterThan
- Seuil : 70

Cette règle surveille automatiquement toutes les valeurs pour de l’instance, c’est-à-dire que vous pouvez surveiller les nouvelles instances dès leur ajout sans avoir à modifier votre règle d’alerte de métrique.

Quand vous supervisez plusieurs dimensions, la règle d’alerte de seuil dynamique peut créer des seuils adaptés à des centaines de séries de métriques à la fois. Un seuil dynamique réduit le nombre de règles d’alerte à gérer et le temps passé à gérer et créer ces règles.

Supposons que vous ayez une application web avec de nombreuses instances et que vous ne connaissiez pas le seuil le plus approprié. Les règles ci-dessus utilisent toujours le seuil de 70 %. Toutefois, vous pouvez créer une règle comme suit :

- Ressource cible : myAppServicePlan
- Mesure : Percentage CPU
- Type de condition : Dynamique
- Dimensions
  - Instance = *
- Agrégation de temps : Average
- Période : Au cours des 5 dernières minutes
- Fréquence : 1 minute
- Opérateur : GreaterThan
- Sensibilité : Moyenne
- Périodes passées à vérifier : 1
- Nombre de violations : 1

Cette règle surveille si l’utilisation moyenne du processeur pendant les 5 dernières minutes dépasse le comportement attendu pour chaque instance. Vous pouvez superviser les instances dès leur ajout sans avoir à remodifier votre règle d’alerte de métrique. Chaque instance obtient un seuil qui s’adapte au modèle de comportement de la série de métriques et qui change en permanence selon les nouvelles données pour rendre le seuil plus précis. Comme avant, chaque instance est supervisée individuellement et vous recevez des notifications séparées.

L’allongement des périodes passées à vérifier et le nombre de violations peuvent aussi vous permettre de filtrer les alertes de sorte à seulement recevoir une conformément à votre définition d’un écart significatif. [Découvrez-en plus sur les options avancées des seuils dynamiques](alerts-dynamic-thresholds.md#what-do-the-advanced-settings-in-dynamic-thresholds-mean).

> [!NOTE]
>
> Nous vous recommandons de choisir une *granularité d’agrégation (période)* supérieure à la *fréquence d’évaluation* afin de réduire la probabilité de manquer la première évaluation de la série chronologique ajoutée dans les cas suivants :
> - Règle d’alerte métrique qui surveille plusieurs dimensions : quand une combinaison de valeurs de dimension est ajoutée
> - Règle d’alerte métrique qui surveille plusieurs ressources : quand une ressource est ajoutée à l’étendue
> - Règle d’alerte métrique qui surveille une métrique qui n’est pas émise en continu (métrique éparse) : lorsque la métrique est émise après une période de plus de 24 heures pendant laquelle elle n’a pas été émise



## <a name="monitoring-at-scale-using-metric-alerts-in-azure-monitor"></a>Surveillance à l’échelle à l’aide d’alertes de métrique dans Azure Monitor

Pour l'instant, vous avez vu la manière d'utiliser une alerte de métrique unique pour surveiller une ou plusieurs séries chronologiques de métriques liées à une seule ressource Azure. Vous apprécierez de pouvoir appliquer la même règle d'alerte à de nombreuses ressources. Azure Monitor prend également en charge la supervision de plusieurs ressources (du même type) avec une seule règle d’alerte de métrique, pour les ressources qui existent dans la même région Azure. 

Cette fonctionnalité est actuellement prise en charge pour les métriques de plateforme (et non pour les métriques personnalisées) pour les services suivants dans les clouds Azure suivants :

| Service | Cloud public Azure | Gouvernement américain | Chine |
|:--------|:--------|:--------|:--------|
| Machines virtuelles<sup>1</sup>  | **Oui** | Non | Non |
| Bases de données SQL Server | **Oui** | **Oui** | **Oui** |
| Pools élastiques SQL Server | **Oui** | **Oui** | **Oui** |
| Pools de capacité NetApp Files | **Oui** | **Oui** | **Oui** |
| Volumes NetApp Files | **Oui** | **Oui** | **Oui** |
| Coffres de clés | **Oui** | **Oui** | **Oui** |
| Cache Azure pour Redis | **Oui** | **Oui** | **Oui** |
| Appareils Data Box Edge | **Oui** | **Oui** | **Oui** |

<sup>1</sup> Pas pris en charge pour les métriques de réseau des machines virtuelles (Octets entrants réseau totaux, Octets sortants réseau totaux, Flux entrants, Flux sortants, Taux de création maximal de flux entrants, Taux de création maximal de flux sortants).

Vous disposez de trois méthodes pour spécifier l’étendue de la supervision par une règle d’alerte de métrique unique. Par exemple, avec des machines virtuelles, vous pouvez spécifier l’étendue comme suit :  

- sous la forme d’une liste de machines virtuelles (dans une seule région Azure) au sein d’un abonnement
- pour toutes les machines virtuelles (dans une seule région Azure) dans un ou plusieurs groupes de ressources d’un abonnement ;
- pour toutes les machines virtuelles (dans une seule région Azure) d’un abonnement

> [!NOTE]
>
> L’étendue d’une règle d’alerte de métrique multiressource doit contenir au moins une ressource du type de ressource sélectionné.

La création de règles d'alerte de métrique surveillant plusieurs ressources est similaire à la [création de toute autre alerte de métrique](alerts-metric.md) surveillant une seule ressource, à la différence près que vous sélectionnez toutes les ressources que vous souhaitez surveiller. Vous pouvez également créer ces règles par le biais des [modèles Azure Resource Manager](./alerts-metric-create-templates.md#template-for-a-metric-alert-that-monitors-multiple-resources). Vous recevrez des notifications individuelles pour chaque ressource supervisée.

> [!NOTE]
>
> Dans une règle d’alerte métrique qui surveille plusieurs ressources, une seule condition est autorisée.

## <a name="typical-latency"></a>Latence classique

Pour les alertes de métrique, vous êtes généralement informé en moins de 5 minutes si vous définissez la fréquence de règle d’alerte sur 1 minute. En cas de forte charge sur les systèmes de notification, vous pouvez observer une latence plus longue.

## <a name="supported-resource-types-for-metric-alerts"></a>Types de ressources pris en charge pour les alertes de métrique

Vous trouverez la liste complète des types de ressources pris en charge dans cet [article](./alerts-metric-near-real-time.md#metrics-and-dimensions-supported).


## <a name="next-steps"></a>Étapes suivantes

- [Découvrir comment créer, afficher et gérer des alertes de métrique dans Azure](alerts-metric.md)
- [Découvrir comment créer des alertes dans Azure Monitor Metrics Explorer](./metrics-charts.md#create-alert-rules)
- [Découvrir comment déployer des alertes de métrique à l’aide de modèles Azure Resource Manager](./alerts-metric-create-templates.md)
- [En savoir plus sur les groupes d’actions](action-groups.md)
- [En savoir plus sur le type de condition des seuils dynamiques](alerts-dynamic-thresholds.md)
- [En savoir plus sur la résolution des problèmes liés aux alertes de métrique](alerts-troubleshoot-metric.md)

