---
title: Alertes d’Azure Monitor pour machines virtuelles
description: Décrit comment créer des règles d’alerte à partir des données de performances collectées par Azure Monitor pour machines virtuelles.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 11/10/2020
ms.openlocfilehash: f582f0dc7547a607351fcfc4ff9d39e8c5a077df
ms.sourcegitcommit: c157b830430f9937a7fa7a3a6666dcb66caa338b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94686175"
---
# <a name="how-to-create-alerts-from-azure-monitor-for-vms"></a>Comment créer des alertes à partir d’Azure Monitor pour machines virtuelles
[Les alertes dans Azure Monitor](../platform/alerts-overview.md) vous informent de façon proactive des données et des modèles intéressants dans vos données de surveillance. Azure Monitor pour machines virtuelles n’inclut pas de règles d’alerte préconfigurées, mais vous pouvez créer vos propres règles en fonction des données collectées. Cet article fournit des conseils sur la création de règles d’alerte, y compris un ensemble d’exemples de requêtes.

> [!IMPORTANT]
> Les alertes décrites dans cet article sont basées sur des requêtes de journal à partir de données collectées par Azure Monitor pour machines virtuelles. Cela diffère des alertes créées par [Intégrité des invités Azure Monitor pour machines virtuelles](vminsights-health-overview.md), qui est une fonctionnalité actuellement en préversion publique. À mesure que cette fonctionnalité se rapproche de la disponibilité générale, des conseils sur les alertes seront consolidés.


## <a name="alert-rule-types"></a>Types de règles d’alerte
Azure Monitor propose [différents types de règles d’alerte](../platform/alerts-overview.md#what-you-can-alert-on) en fonction des données utilisées pour créer l’alerte. Toutes les données collectées par Azure Monitor pour machines virtuelles sont stockées dans des journaux Azure Monitor, avec prise en charge des [alertes de journal](../platform/alerts-log.md). Vous ne pouvez actuellement pas utiliser [d’alertes de métriques](../platform/alerts-log.md) avec les données de performances collectées à partir d’Azure Monitor pour machines virtuelles, car les données ne sont pas collectées dans des métriques Azure Monitor. Pour collecter des données pour les alertes de métriques, installez [l’extension Diagnostics](../platform/diagnostics-extension-overview.md) pour les machines virtuelles Windows ou [l’agent Telegraf](../platform/collect-custom-metrics-linux-telegraf.md) pour les machines virtuelles Linux afin de collecter les données de performances dans des métriques.

Il existe deux types d’alertes de journal dans Azure Monitor :

- Les [alertes sur le nombre de résultats](../platform/alerts-unified-log.md#count-of-the-results-table-rows) créent une alerte unique lorsqu’une requête retourne au moins un nombre spécifié d’enregistrements. Elles sont idéales pour les données non numériques, comme les événements Windows et syslog collectés par [l’agent Log Analytics](../platform/log-analytics-agent.md) ou pour l’analyse des tendances de performances sur plusieurs ordinateurs.
- Les [alertes de mesure de métrique](../platform/alerts-unified-log.md#calculation-of-measure-based-on-a-numeric-column-such-as-cpu-counter-value) créent une alerte distincte pour chaque enregistrement dans une requête dont la valeur dépasse un seuil défini dans la règle d’alerte. Ces règles d’alerte sont idéales pour les données de performances collectées par Azure Monitor pour machines virtuelles, car elles peuvent créer des alertes individuelles pour chaque ordinateur.


## <a name="alert-rule-walkthrough"></a>Procédure pas à pas pour les règles d’alerte
Cette section décrit la création d’une règle d’alerte de mesure de métrique à l’aide des données de performances d’Azure Monitor pour machines virtuelles. Vous pouvez utiliser ce processus de base avec diverses requêtes de journal pour alerter sur différents compteurs de performance.

Commencez par créer une nouvelle règle d’alerte en suivant la procédure décrite dans [Créer, afficher et gérer des alertes de journal à l’aide d’Azure Monitor](../platform/alerts-log.md). Pour la **Ressource**, sélectionnez l’espace de travail Log Analytics que les machines virtuelles Azure Monitor utilisent dans votre abonnement. Étant donné que la ressource cible pour les règles d’alerte de journal est toujours un espace de travail Log Analytics, la requête de journal doit inclure un filtre pour des machines virtuelles spécifiques ou des groupes de machines virtuelles identiques. 

Pour la **Condition** de la règle d’alerte, utilisez l’une des requêtes de la [section ci-dessous](#sample-alert-queries) comme **Requête de recherche**. La requête doit retourner une propriété numérique appelée *AggregatedValue*. Elle doit résumer les données par ordinateur pour que vous puissiez créer une alerte distincte pour chaque machine virtuelle qui dépasse le seuil.

Dans la **Logique d’alerte**, sélectionnez **Mesure de métriques** puis indiquez une **Valeur de seuil**. Dans **Déclencher l’alerte selon**, spécifiez le nombre de fois où le seuil doit être dépassé avant la création d’une alerte. Par exemple, cela ne vous pose probablement pas de problème si le processeur dépasse un seuil une fois puis revient à la normale, mais cela peut être inquiétant s’il continue de dépasser le seuil sur plusieurs mesures consécutives.

La section **Évaluées sur la base de** définit la fréquence d’exécution de la requête et la fenêtre de temps de la requête. Dans l’exemple ci-dessous, la requête est exécutée toutes les 15 minutes et évalue les valeurs de performances collectées au cours des 15 dernières minutes.


![Règle d’alerte Mesure métrique](media/vminsights-alerts/metric-measurement-alert.png)

## <a name="sample-alert-queries"></a>Exemples de requêtes d’alerte
Les requêtes suivantes peuvent être utilisées avec une règle d’alerte de mesure de métrique à l’aide des données de performances collectées par Azure Monitor pour machines virtuelles. Chacune synthétise les données par ordinateur afin qu’une alerte soit créée pour chaque ordinateur avec une valeur qui dépasse le seuil.

### <a name="cpu-utilization"></a>Utilisation du processeur

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms" 
| where Namespace == "Processor" and Name == "UtilizationPercentage" 
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), Computer, _ResourceId
```

### <a name="available-memory-in-mb"></a>Mémoire disponible en Mo

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where Namespace == "Memory" and Name == "AvailableMB"
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), Computer, _ResourceId
```

### <a name="available-memory-in-percentage"></a>Mémoire disponible en pourcentage

```kusto
InsightsMetrics 
| where Origin == "vm.azm.ms" 
| where Namespace == "Memory" and Name == "AvailableMB" 
| extend TotalMemory = toreal(todynamic(Tags)["vm.azm.ms/memorySizeMB"])
| extend AvailableMemoryPercentage = (toreal(Val) / TotalMemory) * 100.0 
| summarize AggregatedValue = avg(AvailableMemoryPercentage) by bin(TimeGenerated, 15m), Computer, _ResourceId 
```

### <a name="logical-disk-used---all-disks-on-each-computer"></a>Disque logique utilisé : tous les disques sur chaque ordinateur

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where Namespace == "LogicalDisk" and Name == "FreeSpacePercentage"
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), Computer, _ResourceId 
```

### <a name="logical-disk-used---individual-disks"></a>Disque logique utilisé : disques individuels

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where Namespace == "LogicalDisk" and Name == "FreeSpacePercentage"
| extend Disk=tostring(todynamic(Tags)["vm.azm.ms/mountId"])
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), Computer, _ResourceId, Disk
```

### <a name="logical-disk-iops"></a>Disque logique : IOPS

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms" 
| where Namespace == "LogicalDisk" and Name == "TransfersPerSecond"
| extend Disk=tostring(todynamic(Tags)["vm.azm.ms/mountId"])
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m) ), Computer, _ResourceId, Disk
```

### <a name="logical-disk-data-rate"></a>Débit de données du disque logique

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms" 
| where Namespace == "LogicalDisk" and Name == "BytesPerSecond"
| extend Disk=tostring(todynamic(Tags)["vm.azm.ms/mountId"])
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m) , Computer, _ResourceId, Disk
```

### <a name="network-interfaces-bytes-received---all-interfaces"></a>Octets reçus sur les interfaces réseau : toutes les interfaces

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where Namespace == "Network" and Name == "ReadBytesPerSecond"
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), Computer, _ResourceId 
```

### <a name="network-interfaces-bytes-received---individual-interfaces"></a>Octets reçus sur les interfaces réseau : interfaces individuelles

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where Namespace == "Network" and Name == "ReadBytesPerSecond"
| extend NetworkInterface=tostring(todynamic(Tags)["vm.azm.ms/networkDeviceId"])
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), Computer, _ResourceId, NetworkInterface
```

### <a name="network-interfaces-bytes-sent---all-interfaces"></a>Octets envoyés sur les interfaces réseau : toutes les interfaces

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where Namespace == "Network" and Name == "WriteBytesPerSecond"
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), Computer, _ResourceId
```

### <a name="network-interfaces-bytes-sent---individual-interfaces"></a>Octets envoyés sur les interfaces réseau : interfaces individuelles

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where Namespace == "Network" and Name == "WriteBytesPerSecond"
| extend NetworkInterface=tostring(todynamic(Tags)["vm.azm.ms/networkDeviceId"])
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), Computer, _ResourceId, NetworkInterface
```

### <a name="virtual-machine-scale-set"></a>Jeu de mise à l’échelle de machine virtuelle
Modifiez avec votre ID d’abonnement, votre groupe de ressources et le nom du groupe de machines virtuelles identiques.

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where _ResourceId startswith "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/my-resource-group/providers/Microsoft.Compute/virtualMachineScaleSets/my-vm-scaleset" 
| where Namespace == "Processor" and Name == "UtilizationPercentage"
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), _ResourceId
```

### <a name="specific-virtual-machine"></a>Machine virtuelle spécifique
Modifiez avec votre ID d’abonnement, votre groupe de ressources et le nom de la machine virtuelle.

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where _ResourceId =~ "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/my-resource-group/providers/Microsoft.Compute/virtualMachines/my-vm" 
| where Namespace == "Processor" and Name == "UtilizationPercentage"
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m)
```

### <a name="cpu-utilization-for-all-compute-resources-in-a-subscription"></a>Utilisation du processeur pour toutes les ressources de calcul dans un abonnement
Remplacez par votre ID d’abonnement.

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where _ResourceId startswith "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" and (_ResourceId contains "/providers/Microsoft.Compute/virtualMachines/" or _ResourceId contains "/providers/Microsoft.Compute/virtualMachineScaleSets/")
| where Namespace == "Processor" and Name == "UtilizationPercentage"
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), _ResourceId
```

### <a name="cpu-utilization-for-all-compute-resources-in-a-resource-group"></a>Utilisation du processeur pour toutes les ressources de calcul dans un groupe de ressources
Modifiez avec votre ID d’abonnement et votre groupe de ressources.

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where _ResourceId startswith "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/my-resource-group/providers/Microsoft.Compute/virtualMachines/"
or _ResourceId startswith "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/my-resource-group/providers/Microsoft.Compute/virtualMachineScaleSets/" 
| where Namespace == "Processor" and Name == "UtilizationPercentage"
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), _ResourceId

```

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur les [alertes dans Azure Monitor](../platform/alerts-overview.md).
- En savoir plus sur les [requêtes de journal à l’aide des données d’Azure Monitor pour machines virtuelles](vminsights-log-search.md).
