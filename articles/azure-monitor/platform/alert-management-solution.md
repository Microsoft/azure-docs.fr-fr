---
title: Solution Alert Management dans Azure Log Analytics | Microsoft Docs
description: La solution de gestion des alertes dans Log Analytics vous permet d’analyser toutes les alertes qui se produisent dans votre environnement.  Outre la centralisation des alertes générées dans Log Analytics, elle importe les alertes de groupes d’administration System Center Operations Manager connectés dans Log Analytics.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 01/19/2018
ms.openlocfilehash: adc29916c6b674531d7b0e8fcdd4e151b4a17bde
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95997254"
---
# <a name="alert-management-solution-in-azure-log-analytics"></a>Solution Alert Management dans Azure Log Analytics

![Icône de gestion des alertes](media/alert-management-solution/icon.png)

La solution de gestion des alertes vous permet d’analyser toutes les alertes qui se produisent dans votre référentiel Log Analytics.  Ces alertes peuvent provenir de diverses sources, y compris celles [créées par Log Analytics](./alerts-overview.md) ou [importées à partir de Nagios ou Zabbix](../learn/quick-collect-linux-computer.md). La solution importe également les alertes de tous les [groupes d’administration System Center Operations Manager connectés](./om-agents.md).

## <a name="prerequisites"></a>Prérequis
La solution fonctionnant avec tous les enregistrements dans le référentiel Log Analytics ayant le type **Alerte**, vous devez donc effectuer la configuration nécessaire pour collecter ces enregistrements.

- Pour les alertes Log Analytics, [créez des règles d’alerte](./alerts-overview.md) pour créer des enregistrements d’alerte directement dans le référentiel.
- Pour les alertes Nagios et Zabbix, [configurez ces serveurs](../learn/quick-collect-linux-computer.md) pour envoyer des alertes à Log Analytics.
- Pour les alertes System Center Operations Manager, [connectez votre groupe d’administration Operations Manager à votre espace de travail Log Analytics](./om-agents.md).  Toutes les alertes créées dans System Center Operations Manager sont importées dans Log Analytics.  

## <a name="configuration"></a>Configuration
Ajoutez la solution Alert Management à votre espace de travail Log Analytics en suivant la procédure décrite dans [Ajouter des solutions](../insights/solutions.md). Aucune configuration supplémentaire n’est requise.

## <a name="management-packs"></a>Packs d’administration
Si votre groupe d’administration System Center Operations Manager est connecté à votre espace de travail Log Analytics, les packs d’administration suivants sont installés dans System Center Operations Manager lorsque vous ajoutez cette solution.  Ces packs d’administration ne nécessitent aucune opération de configuration ou de maintenance.

* Microsoft System Center Advisor Alert Management (Microsoft.IntelligencePacks.AlertManagement)

Pour plus d’informations sur la façon dont ces packs d’administration de solution sont mis à jour, consultez [Connecter Operations Manager à Log Analytics](./om-agents.md).

## <a name="data-collection"></a>Collecte de données
### <a name="agents"></a>Agents
Le tableau suivant décrit les sources connectées qui sont prises en charge par cette solution.

| Source connectée | Support | Description |
|:--- |:--- |:--- |
| [Agents Windows](agent-windows.md) | Non |Les agents Windows directs ne génèrent pas d’alertes.  Des alertes Log Analytics peuvent être créées à partir d’événements et de données de performances collectées à partir des agents Windows. |
| [Agents Linux](../learn/quick-collect-linux-computer.md) | Non |Les agents Linux directs ne génèrent pas d’alertes.  Des alertes Log Analytics peuvent être créées à partir d’événements et de données de performances collectées à partir des agents Linux.  Les alertes Nagios et Zabbix sont collectées à partir de ces serveurs qui requièrent l’agent Linux. |
| [Groupe d’administration de Microsoft System Center Operations Manager](./om-agents.md) |Oui |Les alertes générées sur des agents Operations Manager sont remises au groupe d’administration, puis transférées à Log Analytics.<br><br>Une connexion directe entre des agents Operations Manager et Log Analytics n’est pas obligatoire. Les données des alertes sont transférées du groupe d’administration dans Log Analytics. |


### <a name="collection-frequency"></a>Fréquence de collecte
- Les enregistrements d’alerte sont disponibles pour la solution dès qu’ils sont stockés dans le référentiel.
- Les données des alertes sont envoyées du groupe d’administration Operations Manager à Log Analytics toutes les 3 minutes.  

## <a name="using-the-solution"></a>Utilisation de la solution
Quand vous ajoutez la solution Alert Management à votre espace de travail Log Analytics, la vignette **Gestion des alertes** est ajoutée à votre tableau de bord.  Cette vignette affiche le nombre d’alertes actuellement actives qui ont été générées au cours des dernières 24 heures, ainsi qu’une représentation graphique.  Vous ne pouvez pas modifier cet intervalle de temps.

![Vignette de gestion des alertes](media/alert-management-solution/tile.png)

Cliquez sur la vignette **Gestion des alertes** pour ouvrir le tableau de bord **Gestion des alertes**.  Le tableau de bord comprend les colonnes figurant dans le tableau suivant.  Chaque colonne répertorie les 10 premières alertes (classées par nombre d’alertes) correspondant aux critères de cette colonne pour l’étendue et l’intervalle de temps spécifiés.  Vous pouvez exécuter une recherche dans les journaux qui fournit la liste complète. Pour cela, cliquez sur **Afficher tout** en bas de la colonne ou cliquez sur l’en-tête de colonne.

| Colonne | Description |
|:--- |:--- |
| Alertes critiques |Toutes les alertes ayant le niveau de gravité Critique, regroupées selon le nom de l’alerte.  Cliquez sur un nom d’alerte pour exécuter une recherche dans les journaux retournant tous les enregistrements pour cette alerte. |
| Alertes d'avertissement |Toutes les alertes ayant le niveau de gravité Avertissement, regroupées selon le nom de l’alerte.  Cliquez sur un nom d’alerte pour exécuter une recherche dans les journaux retournant tous les enregistrements pour cette alerte. |
| Alertes actives de System Center Operations Manager |Toutes les alertes recueillies depuis Operations Manager n’ayant pas l’état *Fermé* , regroupées selon la source ayant généré l’alerte. |
| Toutes les alertes actives |Toutes les alertes, quel que soit leur niveau de gravité, regroupées selon le nom de l’alerte. Seules les alertes Operations Manager n’ayant pas l’état *Fermé*. |

Si vous faites défiler l’écran vers la droite, le tableau de bord répertorie plusieurs requêtes courantes sur lesquelles vous pouvez cliquer pour effectuer une [recherche dans les journaux](../log-query/log-query-overview.md) afin d’obtenir les données des alertes.

![Tableau de bord de gestion des alertes](media/alert-management-solution/dashboard.png)


## <a name="log-analytics-records"></a>Enregistrements Log Analytics
La solution de gestion des alertes analyse tous les enregistrements de type **Alerte**.  Les alertes créées par Log Analytics ou collectées à partir de Nagios ou Zabbix ne sont pas collectées directement par la solution.

Cette solution importe également les alertes de System Center Operations Manager et crée pour chacune d’entre elles un enregistrement correspondant avec comme propriétés Type et SourceSystem les valeurs **Alert** et **OpsManager**, respectivement.  Les propriétés de ces enregistrements sont décrites dans le tableau suivant :  

| Propriété | Description |
|:--- |:--- |
| `Type` |*Alert* |
| `SourceSystem` |*OpsManager* |
| `AlertContext` |Détails de l’élément de données à l’origine de l’alerte générée au format XML. |
| `AlertDescription` |Description détaillée de l’alerte. |
| `AlertId` |GUID de l’alerte. |
| `AlertName` |Nom de l’alerte. |
| `AlertPriority` |Niveau de priorité de l’alerte. |
| `AlertSeverity` |Niveau de gravité de l’alerte. |
| `AlertState` |Dernier état de résolution de l’alerte. |
| `LastModifiedBy` |Nom de l’utilisateur ayant apporté la dernière modification à l’alerte. |
| `ManagementGroupName` |Nom du groupe d’administration dans lequel l’alerte a été générée. |
| `RepeatCount` |Nombre de fois que l’alerte a été générée pour le même objet analysé depuis sa résolution. |
| `ResolvedBy` |Nom de l’utilisateur ayant résolu l’alerte. Vide si l’alerte n’a pas encore été résolue. |
| `SourceDisplayName` |Nom d’affichage de l’objet d’analyse ayant généré l’alerte. |
| `SourceFullName` |Nom complet de l’objet d’analyse ayant généré l’alerte. |
| `TicketId` |ID de ticket de l’alerte si l’environnement System Center Operations Manager est intégré à un processus d’affectation de tickets pour les alertes.  Vide si aucun ID ticket n’est affecté. |
| `TimeGenerated` |Date et heure de la création de l’alerte. |
| `TimeLastModified` |Date et heure de la dernière modification de l’alerte. |
| `TimeRaised` |Date et heure de la génération de l’alerte. |
| `TimeResolved` |Date et heure de la résolution de l’alerte. Vide si l’alerte n’a pas encore été résolue. |

## <a name="sample-log-searches"></a>Exemples de recherches dans les journaux
Le tableau suivant fournit des exemples de recherches dans les journaux pour les enregistrements d’alerte collectés par cette solution : 

| Requête | Description |
|:---|:---|
| Alerte &#124; où SourceSystem == « OpsManager » et AlertSeverity == « erreur » et TimeRaised > ago(24 h) |Alertes critiques déclenchées au cours des dernières 24 heures |
| Alerte &#124; où AlertSeverity == « avertissement » et TimeRaised > ago(24 h) |Alertes d’avertissement déclenchées au cours des dernières 24 heures |
| Alerte &#124; où SourceSystem == « OpsManager » et AlertState ! = « Fermé » et TimeRaised > ago(24 h) &#124; résumer Count = count() par SourceDisplayName |Sources avec des alertes actives déclenchées au cours des dernières 24 heures |
| Alerte &#124; où SourceSystem == « OpsManager » et AlertSeverity == « erreur » et TimeRaised > ago(24 h) et AlertState != « Fermé » |Alertes critiques déclenchées au cours des dernières 24 heures et toujours actives |
| Alerte &#124; où SourceSystem == « OpsManager », TimeRaised > ago(24 h) et AlertState != « Fermé » |Alertes déclenchées au cours des dernières 24 heures et désormais fermées |
| Alerte &#124; où SourceSystem == « OpsManager » et TimeRaised > ago(1d) &#124; résumer Count = count() par AlertSeverity |Alertes déclenchées au cours de la journée précédente et regroupées selon leur niveau de gravité |
| Alerte &#124; où SourceSystem == « OpsManager » et TimeRaised > ago(1d) &#124; résumer Count = count() par RepeatCount desc |Alertes déclenchées au cours de la journée précédente et triées selon leur valeur de répétition |



## <a name="next-steps"></a>Étapes suivantes
* Consultez [Alertes dans Log Analytics](./alerts-overview.md) pour obtenir des informations sur la génération d’alertes à partir de Log Analytics.