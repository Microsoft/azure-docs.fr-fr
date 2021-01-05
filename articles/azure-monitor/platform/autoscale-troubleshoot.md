---
title: Résolution des problèmes de mise à l’échelle automatique Azure
description: Le suivi des problèmes de mise à l’échelle automatique Azure utilisé dans Service Fabric, les machines virtuelles, les applications web et les services cloud.
ms.topic: conceptual
ms.date: 11/4/2019
ms.subservice: autoscale
ms.openlocfilehash: a29b5d11a6ea06af9d5b6a8b5120c6f0caa6601e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90979049"
---
# <a name="troubleshooting-azure-autoscale"></a>Résolution des problèmes de mise à l’échelle automatique Azure
 
La mise à l’échelle automatique d’Azure Monitor vous permet de disposer de la bonne quantité de ressources en cours d’exécution pour gérer la charge sur votre application. Elle vous permet d’ajouter des ressources pour gérer les augmentations de charge et d’économiser de l’argent en supprimant les ressources qui sont inactives. Vous pouvez mettre à l’échelle en fonction d’une planification, d’une date/heure fixe ou d’une métrique de ressource de votre choix. Pour plus d’informations, consultez [Vue d’ensemble de la mise à l’échelle automatique](autoscale-overview.md).

Le service de mise à l’échelle automatique vous fournit des métriques et des journaux pour comprendre les actions de mise à l’échelle qui se sont produites et l’évaluation des conditions qui ont conduit à ces actions. Vous pouvez trouver des réponses à des questions telles que :

- Pourquoi mon service a-t-il été mis à l’échelle ?
- Pourquoi mon service n’a-t-il pas été mis à l’échelle ?
- Pourquoi une action de mise à l’échelle automatique a-t-elle échoué ?
- Pourquoi une action de mise à l’échelle automatique prend-elle du temps ?
  
## <a name="autoscale-metrics"></a>Métriques de mise à l’échelle automatique

La mise à l’échelle automatique vous fournit [quatre métriques](metrics-supported.md#microsoftinsightsautoscalesettings) pour comprendre son fonctionnement. 

- **Valeur de métrique observée** : la valeur de la métrique sur laquelle vous avez choisi d’effectuer l’action de mise à l’échelle, telle qu’elle apparaît ou est calculée par le moteur de mise à l’échelle automatique. Étant donné qu’un seul paramètre de mise à l’échelle automatique peut avoir plusieurs règles et, par conséquent, plusieurs sources de métriques, vous pouvez filtrer en utilisant « Source de métriques » comme dimension.
- **Seuil de mesure** : le seuil que vous avez défini pour effectuer l’action de mise à l’échelle. Étant donné qu’un seul paramètre de mise à l’échelle automatique peut avoir plusieurs règles et, par conséquent, plusieurs sources de métriques, vous pouvez filtrer en utilisant « Règle de métrique » comme dimension.
- **Capacité observée** : le nombre d’instances actives de la ressource cible, tel qu’il est vu par le moteur de mise à l’échelle automatique.
- **Actions de mise à l’échelle initiées** : le nombre d’actions de mise à l’échelle initiées par le moteur de mise à l’échelle automatique. Vous pouvez filtrer par actions de scale-in ou de scale-out.

Vous pouvez utiliser [Metrics Explorer](metrics-getting-started.md) pour représenter visuellement les métriques ci-dessus dans un même emplacement. Le graphique doit afficher :

  - la métrique réelle
  - la métrique affichée/calculée par le moteur de mise à l’échelle automatique
  - le seuil pour une action de mise à l’échelle
  - la modification de capacité 

## <a name="example-1---analyzing-a-simple-autoscale-rule"></a>Exemple 1 : Analyse d’une règle de mise à l’échelle automatique simple 

Nous disposons d’un paramètre de mise à l’échelle automatique simple pour un groupe de machines virtuelles identiques qui :

- monte en charge lorsque le pourcentage moyen d’utilisation de l’UC d’un groupe est supérieur à 70 % pendant 10 minutes 
- diminue la taille lorsque le pourcentage d’UC du groupe est inférieur à 5 % pendant plus de 10 minutes. 

Passons en revue les métriques du service de mise à l’échelle automatique.
 
![Capture d’écran montrant un exemple de pourcentage d’UC de groupe de machines virtuelles identiques.](media/autoscale-troubleshoot/autoscale-vmss-CPU-ex-full-1.png)

![Exemple de pourcentage d’UC d’un groupe de machines virtuelles identiques](media/autoscale-troubleshoot/autoscale-vmss-CPU-ex-full-2.png)

***Figure 1a - Métrique Pourcentage de l’UC pour le groupe de machines virtuelles identiques et valeur de métrique observée pour le paramètre de mise à l’échelle automatique***

![Seuil de métrique et capacité observée](media/autoscale-troubleshoot/autoscale-metric-threshold-capacity-ex-full.png)

***Figure 1b - Seuil de métrique et capacité observée***

Dans la figure 1b, le **seuil de métrique** (ligne bleu clair) de la règle de montée en charge est de 70.  La **capacité observée** (ligne bleu foncé) indique le nombre d’instances actives, actuellement 3. 

> [!NOTE]
> Vous devez filtrer le **seuil de métrique** par règle de déclenchement de scale-out (augmentation) pour afficher le seuil de scale-out et par règle de scale-in (diminution). 

## <a name="example-2---advanced-autoscaling-for-a-virtual-machine-scale-set"></a>Exemple 2 : Mise à l’échelle automatique avancée pour un groupe de machines virtuelles identiques

Nous disposons d’un paramètre de mise à l’échelle automatique qui permet à une ressource de groupe de machines virtuelles identiques d’effectuer un scale-out en fonction de sa propre métrique **Flux sortants**. Notez que l’option **diviser la métrique par le nombre d’instances** pour le seuil de métrique est cochée. 

La règle d’action de mise à l’échelle est : 

Si la valeur de **Flux sortants par instance** est supérieure à 10, le service de mise à l’échelle automatique doit effectuer un scale-out de 1 instance. 

Dans ce cas, la valeur de métrique observée du moteur de mise à l’échelle automatique est calculée en tant que valeur métrique réelle divisée par le nombre d’instances. Si la valeur de métrique observée est inférieure au seuil, aucune action de montée en charge n’est initiée. 
 
![Capture d’écran montrant la page des flux sortants moyens avec un exemple de graphiques de métriques de mise à l’échelle automatique de groupe de machines virtuelles identiques.](media/autoscale-troubleshoot/autoscale-vmss-metric-chart-ex-1.png)

![Exemple de graphiques de métriques de mise à l’échelle automatique d’un groupe de machines virtuelles identiques](media/autoscale-troubleshoot/autoscale-vmss-metric-chart-ex-2.png)

***Figure 2 - Exemple de graphiques de métriques de mise à l’échelle automatique d’un groupe de machines virtuelles identiques***

Dans la figure 2, vous pouvez voir deux graphiques de métrique. 

Le graphique en haut affiche la valeur réelle de la métrique **Flux sortants**. La valeur réelle est 6. 

Le graphique en bas affiche plusieurs valeurs. 
 - La **valeur de métrique observée** (bleu clair) est 3, car il y a 2 instances actives, et qu’on divise 6 par 2 pour obtenir 3. 
 - La **capacité observée** (violet) indique le nombre d’instances vu par le moteur de mise à l’échelle automatique. 
 - Le **seuil de métrique** (vert clair) est défini sur 10. 

S’il existe plusieurs règles d’action de mise à l’échelle, vous pouvez utiliser le fractionnement ou l’option **ajouter un filtre** dans le graphique de l’explorateur de métriques pour examiner la métrique sur une source ou règle spécifique. Pour plus d’informations sur le fractionnement d’un graphique de métriques, consultez [Fonctionnalités avancées des graphiques de métrique - Fractionnement](metrics-charts.md#apply-splitting-to-a-chart)

## <a name="example-3---understanding-autoscale-events"></a>Exemple 3 : Comprendre les événements de mise à l’échelle automatique

Dans l’écran de paramètres de mise à l’échelle automatique, accédez à l’onglet **Historique des exécutions** pour afficher les actions de mise à l’échelle les plus récentes. L’onglet affiche également l’évolution de **Capacité observée** dans le temps. Pour plus d’informations sur les actions de mise à l’échelle automatique, y compris des opérations telles que la mise à jour/suppression de paramètres de mise à l’échelle automatique, consultez le journal d’activité et filtrez par opération de mise à l’échelle automatique.

![Historique d’exécution des paramètres de mise à l’échelle automatique](media/autoscale-troubleshoot/autoscale-setting-run-history-smaller.png)

## <a name="autoscale-resource-logs"></a>Journaux de ressources de mise à l’échelle automatique

Comme pour toute autre ressource Azure, le service de mise à l’échelle automatique fournit des [journaux de ressources](platform-logs-overview.md). Il existe deux catégories de journaux.

- **Évaluations de mise à l’échelle automatique** : le moteur de mise à l’échelle automatique enregistre des entrées de journal pour chaque évaluation de condition unique chaque fois qu’il effectue une vérification.  L’entrée comprend des détails sur les valeurs observées des métriques, les règles évaluées et si l’évaluation a entraîné une action de mise à l’échelle.

- **Actions de mise à l’échelle automatique** : le moteur enregistre les événements d’action de mise à l’échelle initiés par le service de mise à l’échelle automatique et les résultats de ces actions d’échelle (réussite, échec, ainsi que l’ampleur de la mise à l’échelle observée par le service de mise à l’échelle automatique).

Comme pour tout service Azure Monitor pris en charge, vous pouvez utiliser les [Paramètres de diagnostic](diagnostic-settings.md) pour acheminer ces journaux :

- dans votre espace de travail Log Analytics pour une analyse détaillée
- dans Event Hubs, puis dans des outils non Azure
- sur votre compte de stockage Azure à des fins d’archivage  

![Paramètres de diagnostic de la mise à l’échelle automatique](media/autoscale-troubleshoot/diagnostic-settings.png)

L’image précédente montre les paramètres de diagnostic de mise à l’échelle automatique dans le portail Azure. Vous pouvez sélectionner l’onglet Journaux de diagnostic/ressources et activer la collecte et l’acheminement des journaux. Vous pouvez également effectuer la même action à l’aide de l’API REST, de l’interface CLI, de PowerShell et des modèles Resource Manager pour les paramètres de diagnostic en choisissant le type de ressource *Microsoft.Insights/AutoscaleSettings*. 

## <a name="troubleshooting-using-autoscale-logs"></a>Résolution des problèmes avec les journaux de mise à l’échelle automatique 

Pour une meilleure expérience de dépannage, nous vous recommandons d’acheminer vos journaux vers Azure Monitor Logs (Log Analytics) dans un espace de travail lorsque vous créez le paramètre de mise à l’échelle automatique. Ce processus est illustré dans l’image de la section précédente. Vous pourrez valider les évaluations et actions de mise à l’échelle plus facilement à l’aide de Log Analytics.

Une fois que vous avez configuré les journaux de mise à l’échelle automatique pour qu’ils soient envoyés à l’espace de travail Log Analytics, vous pouvez exécuter les requêtes suivantes pour vérifier les journaux. 

Pour commencer, essayez cette requête pour afficher les journaux d’évaluation de mise à l’échelle automatique les plus récents :

```Kusto
AutoscaleEvaluationsLog
| limit 50
```

Ou exécutez la requête suivante pour afficher les journaux des actions de mise à l’échelle les plus récents :

```Kusto
AutoscaleScaleActionsLog
| limit 50
```

Utilisez les sections suivantes pour répondre à ces questions. 

## <a name="a-scale-action-occurred-that-i-didnt-expect"></a>Une action de mise à l’échelle que je n’attendais pas s’est produite

Exécutez d’abord la requête pour l’action de mise à l’échelle afin de trouver l’action de mise à l’échelle qui vous intéresse. S’il s’agit de la dernière action de mise à l’échelle, utilisez la requête suivante :

```Kusto
AutoscaleScaleActionsLog
| take 1
```

Sélectionnez le champ CorrelationId dans le journal des actions de mise à l’échelle. Utilisez l’ID de corrélation pour trouver le journal d’évaluation approprié. L’exécution de la requête ci-dessous affiche toutes les règles et conditions évaluées menant à cette action de mise à l’échelle.

```Kusto
AutoscaleEvaluationsLog
| where CorrelationId = "<correliationId>"
```

## <a name="what-profile-caused-a-scale-action"></a>Quel profil a entraîné une action de mise à l’échelle ?

Une action de mise à l’échelle s’est produite, mais vous avez des règles et des profils qui se chevauchent et vous devez effectuer le suivi de ce qui a été à l’origine de l’action. 

Recherchez le correlationId de l’action de mise à l’échelle (comme expliqué dans l’exemple 1), puis exécutez la requête sur les journaux d’évaluation pour en savoir plus sur le profil.

```Kusto
AutoscaleEvaluationsLog
| where CorrelationId = "<correliationId_Guid>"
| where ProfileSelected == true
| project ProfileEvaluationTime, Profile, ProfileSelected, EvaluationResult
```

L’évaluation du profil dans son ensemble peut également être mieux comprise à l’aide de la requête suivante

```Kusto
AutoscaleEvaluationsLog
| where TimeGenerated > ago(2h)
| where OperationName contains == "profileEvaluation"
| project OperationName, Profile, ProfileEvaluationTime, ProfileSelected, EvaluationResult
```

## <a name="a-scale-action-did-not-occur"></a>Aucune action de mise à l’échelle n’a eu lieu

Je m’attendais à une action de mise à l’échelle, mais cela ne s’est pas produit. Il peut ne pas y avoir d’événements ou de journaux d’action de mise à l’échelle.

Passez en revue les métriques de mise à l’échelle automatique si vous utilisez une règle de mise à l’échelle basée sur les métriques. Il est possible que la **valeur de métrique observée** ou la **capacité observée** étaient différentes de ce à quoi vous vous attendiez et que la règle de mise à l’échelle n’ait donc pas été déclenchée. Vous pouvez toujours voir les évaluations, mais pas de règle de montée en charge. Il est également possible que le temps de recharge ait empêché une action de mise à l’échelle. 

 Examinez les journaux d’évaluation de la mise à l’échelle automatique pendant la période pendant laquelle vous vous attendiez à une action de mise à l’échelle. Examinez toutes les évaluations effectuées et pourquoi le système a décidé de ne pas déclencher une action de mise à l’échelle.


```Kusto
AutoscaleEvaluationsLog
| where TimeGenerated > ago(2h)
| where OperationName == "MetricEvaluation" or OperationName == "ScaleRuleEvaluation"
| project OperationName, MetricData, ObservedValue, Threshold, EstimateScaleResult
```

## <a name="scale-action-failed"></a>Échec de l'action de mise à l'échelle

Il peut y avoir un cas où le service de mise à l’échelle automatique a effectué une action de mise à l’échelle, mais le système a décidé de ne pas mettre à l’échelle ou n’a pas pu terminer l’action de mise à l’échelle. Utilisez cette requête pour rechercher les actions de mise à l’échelle qui ont échoué.

```Kusto
AutoscaleScaleActionsLog
| where ResultType == "Failed"
| project ResultDescription
```

Créez des règles d’alerte pour être informé des actions de mise à l’échelle automatique ou des échecs. Vous pouvez également créer des règles d’alerte pour recevoir des notifications sur les événements de mise à l’échelle automatique.

## <a name="schema-of-autoscale-resource-logs"></a>Schéma des journaux de ressources de mise à l’échelle automatique

Pour plus d’informations, consultez [Journaux de ressource de mise à l’échelle automatique](autoscale-resource-log-schema.md)

## <a name="next-steps"></a>Étapes suivantes
Lisez des informations sur les [Meilleures pratiques relatives à la mise à l’échelle automatique](autoscale-best-practices.md). 
