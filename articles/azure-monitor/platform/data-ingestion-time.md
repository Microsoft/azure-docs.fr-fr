---
title: Durée d’ingestion de données de journal dans Azure Monitor | Microsoft Docs
description: Explique les différents facteurs qui affectent la latence lors de la collecte de données de journal dans Azure Monitor.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 07/18/2019
ms.openlocfilehash: 87bfe1109640f158b92f54b945d314ac65a93ddc
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92107910"
---
# <a name="log-data-ingestion-time-in-azure-monitor"></a>Durée d’ingestion de données de journal dans Azure Monitor
Azure Monitor est un service de données à grande échelle servant des milliers de clients envoyant des téraoctets de données chaque mois à un rythme croissant. Les utilisateurs se demandent souvent quel est le délai nécessaire pour que les données de journal soient disponibles une fois qu’elles ont été collectées. Cet article explique les différents facteurs qui affectent cette latence.

## <a name="typical-latency"></a>Latence classique
La latence fait référence au délai qui s’écoule entre l’heure à laquelle les données sont créées sur le système analysé et l’heure à laquelle elles sont disponibles pour analyse dans Azure Monitor. Le temps de latence habituel pour ingérer des données de journal est compris entre 2 et 5 minutes. La latence spécifique pour des données particulières dépend de divers facteurs expliqués ci-dessous.


## <a name="factors-affecting-latency"></a>Facteurs qui affectent la latence
Le temps total d’ingestion pour un jeu de données particulier peut être divisé en plusieurs durées (voir ci-dessous). 

- Délai de l’agent : délai nécessaire pour découvrir un événement, le collecter et l’envoyer au point d’ingestion Azure Monitor sous forme d’enregistrement de journal. Dans la plupart des cas, ce processus est géré par un agent.
- Délai du pipeline : délai nécessaire pour que le pipeline d’ingestion traite l’enregistrement de journal. Cela inclut l’analyse des propriétés de l’événement et l’ajout éventuel des informations calculées.
- Délai d’indexation : délai nécessaire à l’ingestion d’un enregistrement de journal dans le magasin Big Data Azure Monitor.

Des informations détaillées sur les différentes latences introduites dans ce processus sont décrites ci-dessous.

### <a name="agent-collection-latency"></a>Latence de collecte de l’agent
Les solutions de gestion et les agents utilisent différentes stratégies pour collecter des données d’une machine virtuelle, ce qui peut affecter la latence. Voici quelques exemples :

- Les événements Windows, événements syslog et métriques de performances sont collectés immédiatement. Les compteurs de performances Linux sont interrogés à des intervalles de 30 secondes.
- Les journaux d’activité IIS et les journaux d’activité personnalisés sont collectés une fois que leur timestamp change. Pour les journaux d’activité IIS, cela est influencé par le [calendrier de substitution configuré sur IIS](data-sources-iis-logs.md). 
- La solution Active Directory Replication effectue son évaluation tous les cinq jours, tandis que la solution Active Directory Assessment effectue une évaluation hebdomadaire de votre infrastructure Active Directory. L’agent collecte ces journaux d’activité uniquement lorsque l’évaluation est effectuée.

### <a name="agent-upload-frequency"></a>Fréquence de chargement de l’agent
Pour vous assurer que l’agent Log Analytics est léger, l’agent met en mémoire tampon les journaux d’activité et les charge régulièrement dans Azure Monitor. La fréquence de chargement varie entre 30 secondes et 2 minutes en fonction du type de données. La plupart des données sont chargées en moins d’une minute. Les conditions du réseau peuvent affecter négativement la latence nécessaire à ces données pour atteindre le point d’ingestion Azure Monitor.

### <a name="azure-activity-logs-resource-logs-and-metrics"></a>Journaux d’activité Azure, journaux de ressources et métriques
Les données Azure mettent un certain délai à être disponibles au point d’ingestion Log Analytics en vue de leur traitement :

- La mise à disposition des données des journaux de ressources prend 2 à 15 minutes, suivant le service Azure. Consultez la [requête ci-dessous](#checking-ingestion-time) pour examiner cette latence dans votre environnement.
- L’envoi des métriques de la plateforme Azure au point d’ingestion Log Analytics prend 3 minutes.
- L’envoi des données du journal d’activité au point d’ingestion Log Analytics prend entre 10 et 15 minutes.

Une fois disponibles au point d’ingestion, les données peuvent être interrogées au terme d’un délai supplémentaire de 2 à 5 minutes.

### <a name="management-solutions-collection"></a>Collecte des solutions de gestion
Certaines solutions ne collectent pas leurs données à partir d’un agent et peuvent utiliser une méthode de collecte qui introduit une latence supplémentaire. Certaines solutions collectent des données à intervalles réguliers sans tenter d’effectuer une collecte en temps quasi-réel. Voici quelques exemples :

- La solution Microsoft 365 interroge les journaux d'activité à l'aide de l'API d'activité de gestion, qui ne fournit actuellement aucune garantie concernant une latence en temps quasi-réel.
- Les données des solutions Windows Analytics (Update Compliance par exemple) sont collectées par la solution à une fréquence quotidienne.

Reportez-vous à la documentation de chaque solution afin de déterminer sa fréquence de collecte.

### <a name="pipeline-process-time"></a>Délai du processus de pipeline
Une fois que les enregistrements de journal sont ingérés dans le pipeline de Azure Monitor (tel qu’il est identifié dans la propriété [_TimeReceived](./log-standard-columns.md#_timereceived)), ils sont écrits dans un stockage temporaire pour garantir l’isolation des locataires et pour vous assurer que les données ne sont pas perdues. Ce processus ajoute généralement 5 à 15 secondes. Certaines solutions de gestion implémentent des algorithmes plus lourds pour agréger les données et dériver des insights à mesure que les données affluent. Par exemple, Network Performance Monitor agrège les données entrantes toutes les 3 minutes, en ajoutant au final une latence de 3 minutes. Un autre processus qui ajoute une latence est le processus qui gère les journaux d’activité personnalisés. Dans certains cas, ce processus peut ajouter quelques minutes de latence aux journaux d’activité qui sont collectés à partir de fichiers par l’agent.

### <a name="new-custom-data-types-provisioning"></a>Provisionnement des nouveaux types de données personnalisées
Quand un type de données personnalisées est créé à partir d’un [journal personnalisé](data-sources-custom-logs.md) ou de l’[API Collecteur de données](data-collector-api.md), le système crée un conteneur de stockage dédié. Il s’agit d’une surcharge à usage unique qui se produit uniquement à la première apparition de ce type de données.

### <a name="surge-protection"></a>Protection en cas de pics de données
La priorité numéro 1 d'Azure Monitor est de s’assurer que les données client ne sont pas perdues. Le système dispose donc d’une protection intégrée pour les pics de données. Cela inclut des mémoires tampons pour veiller à ce que, même sous une charge considérable, le système reste opérationnel. Sous une charge normale, ces contrôles ajoutent moins d’une minute, mais dans des conditions extrêmes et lors de défaillances, ils peuvent ajouter un délai considérable tout en garantissant la sécurité des données.

### <a name="indexing-time"></a>Délai d’indexation
Chaque plateforme Big Data propose un équilibre entre la fourniture d’analytiques et de fonctionnalités de recherche avancées et la fourniture d’un accès immédiat aux données. Azure Monitor vous permet d’exécuter des requêtes puissantes sur des milliards d’enregistrements et d’obtenir des résultats au bout de quelques secondes. Cela est possible, car l’infrastructure transforme les données de façon spectaculaire lors de leur ingestion et les stocke dans des structures compactes uniques. Le système met en mémoire tampon les données jusqu’à ce que suffisamment de données soient disponibles pour créer ces structures. Cette opération doit être effectuée avant que l’enregistrement de journal s’affiche dans les résultats de la recherche.

Ce processus prend environ 5 minutes avec un faible volume de données, mais moins de temps à un débit plus élevé. Cela paraît contre-intuitif, mais ce processus permet l’optimisation de la latence pour les charges de travail de production impliquant de gros volume.



## <a name="checking-ingestion-time"></a>Vérification de la durée d’ingestion
La durée d’ingestion peut varier pour différentes ressources dans différentes circonstances. Vous pouvez utiliser des requêtes de journal pour identifier un comportement spécifique de votre environnement. Le tableau suivant spécifie la manière dont vous pouvez déterminer les différentes heures d’un enregistrement lors de sa création et de son envoi à Azure Monitor.

| Étape | Propriété ou fonction | Commentaires |
|:---|:---|:---|
| Enregistrement créé au niveau de la source de données | [TimeGenerated](./log-standard-columns.md#timegenerated-and-timestamp) <br>Si la source de données ne définit pas cette valeur, elle est définie à la même heure que _TimeReceived. |
| Enregistrement reçu par le point de terminaison d’ingestion Azure Monitor | [_TimeReceived](./log-standard-columns.md#_timereceived) | |
| Enregistrement stocké dans l’espace de travail et disponible pour les requêtes | [ingestion_time()](/azure/kusto/query/ingestiontimefunction) | |

### <a name="ingestion-latency-delays"></a>Délais de latence d’ingestion
Vous pouvez mesurer la latence d’un enregistrement spécifique en comparant le résultat de la fonction [ingestion_time()](/azure/kusto/query/ingestiontimefunction) à la propriété _TimeGenerated_. Ces données peuvent être utilisées avec différentes agrégations afin de déterminer le comportement de latence d’ingestion. Examinez certains centiles de la durée d’ingestion pour obtenir des insights sur une grande quantité de données. 

Par exemple, la requête suivante montre quels ordinateurs ont eu la durée d’ingestion la plus élevée durant les 8 heures précédentes : 

``` Kusto
Heartbeat
| where TimeGenerated > ago(8h) 
| extend E2EIngestionLatency = ingestion_time() - TimeGenerated 
| extend AgentLatency = _TimeReceived - TimeGenerated 
| summarize percentiles(E2EIngestionLatency,50,95), percentiles(AgentLatency,50,95) by Computer 
| top 20 by percentile_E2EIngestionLatency_95 desc
```

Les contrôles de centile précédents conviennent pour rechercher des tendances générales en matière de latence. Pour identifier un pic à court terme en matière de latence, l’utilisation de la valeur maximale (`max()`) peut s’avérer plus efficace.

Si vous souhaitez explorer au niveau du détail la durée d’ingestion pour un ordinateur spécifique sur une période donnée, utilisez la requête suivante, qui affiche également les données de la journée précédente dans un graphe : 


``` Kusto
Heartbeat 
| where TimeGenerated > ago(24h) //and Computer == "ContosoWeb2-Linux"  
| extend E2EIngestionLatencyMin = todouble(datetime_diff("Second",ingestion_time(),TimeGenerated))/60 
| extend AgentLatencyMin = todouble(datetime_diff("Second",_TimeReceived,TimeGenerated))/60 
| summarize percentiles(E2EIngestionLatencyMin,50,95), percentiles(AgentLatencyMin,50,95) by bin(TimeGenerated,30m) 
| render timechart
```
 
Utilisez la requête suivante pour afficher la durée d’ingestion des ordinateurs en fonction du pays/de la région où ils se trouvent, d’après leur adresse IP : 

``` Kusto
Heartbeat 
| where TimeGenerated > ago(8h) 
| extend E2EIngestionLatency = ingestion_time() - TimeGenerated 
| extend AgentLatency = _TimeReceived - TimeGenerated 
| summarize percentiles(E2EIngestionLatency,50,95),percentiles(AgentLatency,50,95) by RemoteIPCountry 
```
 
Différents types de données provenant de l’agent peuvent avoir des latences d’ingestion différentes ; ainsi, les requêtes précédentes pourraient être utilisées avec d’autres types. Utilisez la requête suivante pour examiner la durée d’ingestion de divers services Azure : 

``` Kusto
AzureDiagnostics 
| where TimeGenerated > ago(8h) 
| extend E2EIngestionLatency = ingestion_time() - TimeGenerated 
| extend AgentLatency = _TimeReceived - TimeGenerated 
| summarize percentiles(E2EIngestionLatency,50,95), percentiles(AgentLatency,50,95) by ResourceProvider
```

### <a name="resources-that-stop-responding"></a>Ressources qui ne répondent plus 
Dans certains cas, une ressource peut cesser d’envoyer des données. Pour savoir si une ressource envoie des données ou non, examinez son enregistrement le plus récent, identifiable par le champ _TimeGenerated_ standard.  

Utilisez la table _Heartbeat_ pour vérifier la disponibilité d’une machine virtuelle, dans la mesure où une pulsation est envoyée une fois par minute par l’agent. Utilisez la requête suivante pour lister les ordinateurs actifs qui n’ont pas signalé de pulsation récemment : 

``` Kusto
Heartbeat  
| where TimeGenerated > ago(1d) //show only VMs that were active in the last day 
| summarize NoHeartbeatPeriod = now() - max(TimeGenerated) by Computer  
| top 20 by NoHeartbeatPeriod desc 
```

## <a name="next-steps"></a>Étapes suivantes
* Consultez le [contrat de niveau de service (SLA)](https://azure.microsoft.com/support/legal/sla/log-analytics/v1_1/) Azure Monitor.