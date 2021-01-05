---
title: Guide pratique pour mettre à jour des règles d’alerte ou des règles d’action quand la ressource cible est déplacée vers une autre région Azure
description: Informations générales et instructions relatives à la mise à jour des règles d’alerte ou des règles d’action quand la ressource cible est déplacée vers une autre région Azure.
author: harelbr
ms.author: harelbr
ms.topic: how-to
ms.custom: subject-moving-resources
ms.date: 06/26/2020
ms.subservice: alerts
ms.openlocfilehash: 4ea5c8552d35db67a1d2caf20c0143c74cdd642e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86505480"
---
# <a name="how-to-update-alert-rules-or-action-rules-when-their-target-resource-moves-to-a-different-azure-region"></a>Guide pratique pour mettre à jour des règles d’alerte ou des règles d’action quand la ressource cible est déplacée vers une autre région Azure

Cet article explique pourquoi les [règles d’alerte](./alerts-overview.md) et [règles d’action](./alerts-action-rules.md) existantes peuvent être affectées quand vous déplacez d’autres ressources Azure entre des régions, et comment identifier et résoudre ces problèmes. Pour plus d’informations sur les cas où le déplacement des ressources entre les régions est utile et pour obtenir une liste de vérification de la conception d’un processus de déplacement, consultez la [documentation principale relative au déplacement de ressources](../../azure-resource-manager/management/move-region.md).

## <a name="why-the-problem-exists"></a>Pourquoi le problème existe-il ?

Les règles d’alerte et les règles d’action référencent d’autres ressources Azure, par exemple, des [machines virtuelles Azure](../../site-recovery/azure-to-azure-tutorial-migrate.md), [Azure SQL](../../azure-sql/database/move-resources-across-regions.md) et [Stockage Azure](../../storage/common/storage-account-move.md). Quand vous déplacez les ressources auxquelles ces règles font référence, les règles risquent de cesser de fonctionner correctement, car elles ne peuvent pas trouver les ressources qu’elles référencent.

Il existe deux raisons principales pour lesquelles vos règles peuvent cesser de fonctionner après le déplacement des ressources cibles :

- L’étendue de votre règle fait explicitement référence à l’ancienne ressource
- Votre règle d’alerte est basée sur des métriques

## <a name="rule-scope-explicitly-refers-to-the-old-resource"></a>L’étendue de la règle référence explicitement l’ancienne ressource

Quand vous déplacez une ressource, son ID de ressource change dans la plupart des cas. En arrière-plan, le système réplique la ressource dans la nouvelle région avant de la supprimer de l’ancienne région. Ce processus nécessite deux ressources ; il existe donc deux ID de ressources différents pendant un laps de temps limité. Étant donné que les ID de ressources doivent être uniques, un nouvel ID doit être créé au cours du processus. 

**Comment le déplacement de la ressource affecte-t-il les règles existantes ?**

Les règles d’alerte et les règles d’action s’appliquent à une étendue de ressources spécifique. L’étendue peut être l’intégralité d’un abonnement, un groupe de ressources, ou une ou plusieurs ressources spécifiques.
Voici par exemple une règle avec une étendue comportant deux ressources (deux machines virtuelles) :

![Règle d’alerte à ressources multiples](media/alerts-resource-move/multi-resource-alert-rule.png)

Si l’étendue de la règle mentionne explicitement une ressource, et que cette ressource a été déplacée et n’a plus le même ID de ressource, cette règle recherche une ressource incorrecte ou inexistante, et par conséquent elle échoue.

**Comment corriger le problème ?**

Mettez à jour ou recréez la règle affectée pour qu’elle pointe vers la nouvelle ressource. Vous trouverez une description du processus de mise à jour de l’étendue plus loin dans cet article.

Le problème s’applique à ces types de règles :

- Règles d’alerte de journal d’activité
- Règles d’action
- Alertes classiques
- Alertes de métrique : pour plus d’informations, consultez la section suivante [Règles d’alerte basées sur des métriques](#alert-rules-based-on-metrics).

> [!NOTE]
> Les règles d’alerte de recherche dans les journaux et les règles d’alerte de détecteur intelligent ne sont pas affectées, car leur étendue est un espace de travail ou Application Insights. Aucune de ces étendues ne prend actuellement en charge les déplacements d’une région à une autre.

## <a name="alert-rules-based-on-metrics"></a>Règles d’alerte basées sur des métriques

Les métriques émises par les ressources Azure sont régionales. Chaque fois qu’une ressource est déplacée vers une nouvelle région, elle commence à émettre ses métriques dans cette nouvelle région. Par conséquent, toutes les règles d’alerte basées sur des métriques doivent être mises à jour ou recréées afin qu’elles pointent vers le flux de métriques actuel dans la région correcte.

Cette explication s’applique à la fois aux [règles d’alerte de métrique](alerts-metric-overview.md) et aux [règles d’alerte de test de disponibilité](../app/monitor-web-app-availability.md).

Si **toutes** les ressources de l’étendue ont été déplacées, vous n’avez pas besoin de recréer la règle. Il vous suffit de mettre à jour n’importe quel champ de la règle d’alerte, par exemple sa description, et de l’enregistrer.
Si **seules certaines ressources** de l’étendue ont été déplacées, vous devez supprimer les ressources déplacées de la règle existante et créer une nouvelle règle qui couvre uniquement les ressources déplacées.

## <a name="procedures-to-fix-problems"></a>Procédures de résolution des problèmes

### <a name="identifying-rules-associated-with-a-moved-resource-from-the-azure-portal"></a>Identification des règles associées à une ressource déplacée à partir du portail Azure

- **Pour les règles d’alerte** : accédez à Alertes > Gérer les règles d’alerte > filtrez en fonction de l’abonnement conteneur et de la ressource déplacée.
> [!NOTE]
> Les règles d’alerte de journal d’activité ne prennent pas en charge ce processus. Il n’est pas possible de mettre à jour l’étendue d’une règle d’alerte de journal d’activité et de la faire pointer vers une ressource d’un autre abonnement. Au lieu de cela, vous pouvez créer une nouvelle règle qui remplacera l’ancienne.

- **Pour les règles d’action** : accédez à Alertes > Gérer les actions > Règles d’action (préversion) > filtrez en fonction de l’abonnement conteneur et de la ressource déplacée.

### <a name="change-scope-of-a-rule-from-the-azure-portal"></a>Modifier l’étendue d’une règle à partir du portail Azure

1. Ouvrez la règle que vous avez identifiée à l’étape précédente en cliquant dessus.
2. Sous **Ressources**, cliquez sur **Modifier** et ajustez l’étendue.
3. Ajustez les autres propriétés de la règle en fonction des besoins.
4. Cliquez sur **Enregistrer**.

![Modifier l’étendue d’une règle d’alerte](media/alerts-resource-move/change-alert-rule-scope.png)

### <a name="change-the-scope-of-a-rule-using-azure-resource-manager-templates"></a>Modifier l’étendue d’une règle à l’aide de modèles Azure Resource Manager

1. Récupérez le modèle Azure Resource Manager de la règle.  Pour exporter le modèle d’une règle à partir du portail Azure :
   1. Accédez à la section Groupes de ressources dans le portail et ouvrez le groupe de ressources contenant la règle.
   2. Dans la section Vue d’ensemble, cochez la case **Afficher les types masqués**, puis filtrez en fonction du type de règle approprié.
   3. Sélectionnez la règle concernée pour afficher ses détails.
   4. Sous **Paramètres**, sélectionnez **Exporter le modèle**.
2. Modifiez le modèle. Si nécessaire, fractionnez en deux règles (pertinent pour certains cas d’alertes de métrique, comme indiqué ci-dessus).
3. Redéployez le modèle.

### <a name="change-scope-of-a-rule-using-rest-api"></a>Modifier l’étendue d’une règle à l’aide de l’API REST

1. Récupérez la règle existante ([alertes de métrique](/rest/api/monitor/metricalerts/get), [alertes de journal d’activité](/rest/api/monitor/activitylogalerts/get)).
2. Modifiez l’étendue ([alertes de journal d’activité](/rest/api/monitor/activitylogalerts/update)).
3. Redéployez la règle ([alertes de métrique](/rest/api/monitor/metricalerts/createorupdate), [alertes de journal d’activité](/rest/api/monitor/activitylogalerts/createorupdate)).

### <a name="change-scope-of-a-rule-using-powershell"></a>Modifier l’étendue d’une règle à l’aide de PowerShell

1. Récupérez la règle existante ([alertes de métrique](/powershell/module/az.monitor/get-azmetricalertrulev2), [alertes de journal d’activité](/powershell/module/az.monitor/get-azactivitylogalert), [règles d’action](/powershell/module/az.alertsmanagement/get-azactionrule)).
2. Modifiez l’étendue. Si nécessaire, fractionnez en deux règles (pertinent pour certains cas d’alertes de métrique, comme indiqué ci-dessus).
3. Redéployez la règle ([alertes de métrique](/powershell/module/az.monitor/add-azmetricalertrulev2), [alertes de journal d’activité](/powershell/module/az.monitor/enable-azactivitylogalert), [règles d’action](/powershell/module/az.alertsmanagement/set-azactionrule)).

### <a name="change-the-scope-of-a-rule-using-azure-cli"></a>Modifier l’étendue d’une règle à l’aide d’Azure CLI

1.  Récupérez la règle existante ([alertes de métrique](/cli/azure/monitor/metrics/alert?view=azure-cli-latest#az-monitor-metrics-alert-show), [alertes de journal d’activité](/cli/azure/monitor/activity-log/alert#az-monitor-activity-log-alert-list)).
2.  Mettez directement à jour l’étendue de la règle ([alertes de métrique](/cli/azure/monitor/metrics/alert#az-monitor-metrics-alert-update), [alertes de journal d’activité](/cli/azure/monitor/activity-log/alert/scope)).
3.  Si nécessaire, fractionnez en deux règles (pertinent pour certains cas d’alertes de métrique, comme indiqué ci-dessus).

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage sur la résolution d’autres problèmes liés aux [notifications d’alerte](alerts-troubleshoot.md), aux [alertes de métrique](alerts-troubleshoot-metric.md) et aux [alertes de journal](alerts-troubleshoot-log.md). 
