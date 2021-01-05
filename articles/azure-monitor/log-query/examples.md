---
title: Exemples de requêtes de journal Azure Monitor | Microsoft Docs
description: Exemples de requêtes de journal Azure Monitor utilisant le langage de requête Kusto.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 03/16/2020
ms.openlocfilehash: 736daa8a09a8f08721c7b7d9c20f012f274b384a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87045522"
---
# <a name="azure-monitor-log-query-examples"></a>Exemples de requêtes de journal dans Azure Monitor
Cet article inclut divers exemples de [requêtes](log-query-overview.md) utilisant le [langage de requête Kusto](/azure/kusto/query/) pour récupérer différents types de données de journal à partir d'Azure Monitor. Plusieurs méthodes sont utilisées pour consolider et analyser les données. Vous pouvez utiliser ces exemples pour identifier les stratégies qui vous conviennent.  

Consultez les [informations de référence sur le langage Kusto](/azure/kusto/query/) pour plus de détails sur les différents mots clés utilisés dans ces exemples. Si vous ne connaissez pas Azure Monitor, lisez une [leçon sur la création de requêtes](get-started-queries.md).

## <a name="events"></a>Événements

### <a name="search-application-level-events-described-as-cryptographic"></a>Rechercher des événements de niveau application décrits comme « Cryptographic »
Cet exemple lance une recherche dans la table **Event** pour les enregistrements dans lesquels **EventLog** a la valeur _Application_ et **RenderedDescription** contient _cryptographic_. Il porte sur les enregistrements des dernières 24 heures.

```Kusto
Event
| where EventLog == "Application" 
| where TimeGenerated > ago(24h) 
| where RenderedDescription contains "cryptographic"
```

### <a name="search-events-related-to-unmarshaling"></a>Rechercher des événements liés à l’unmarshaling
Cet exemple lance une recherche dans les tables **Event** et **SecurityEvents** pour trouver les enregistrements qui mentionnent _unmarshaling_.

```Kusto
search in (Event, SecurityEvent) "unmarshaling"
```

## <a name="heartbeat"></a>Heartbeat

### <a name="chart-a-week-over-week-view-of-the-number-of-computers-sending-data"></a>Obtenir une vue hebdomadaire du nombre d’ordinateurs envoyant des données

L’exemple suivant affiche (sous forme de graphique) le nombre d’ordinateurs distincts qui ont envoyé des pulsations, chaque semaine.

```Kusto
Heartbeat
| where TimeGenerated >= startofweek(ago(21d))
| summarize dcount(Computer) by endofweek(TimeGenerated) | render barchart kind=default
```

### <a name="find-stale-computers"></a>Rechercher des ordinateurs obsolètes

L’exemple suivant recherche les ordinateurs qui étaient actifs les dernières 24 heures, mais n’ont pas envoyé de pulsations depuis une heure.

```Kusto
Heartbeat
| where TimeGenerated > ago(1d)
| summarize LastHeartbeat = max(TimeGenerated) by Computer
| where isnotempty(Computer)
| where LastHeartbeat < ago(1h)
```

### <a name="get-the-latest-heartbeat-record-per-computer-ip"></a>Obtenir le dernier enregistrement de pulsation par adresse IP d’ordinateur

Cet exemple renvoie le dernier enregistrement de pulsation pour chaque adresse IP d’ordinateur.
```Kusto
Heartbeat
| summarize arg_max(TimeGenerated, *) by ComputerIP
```

### <a name="match-protected-status-records-with-heartbeat-records"></a>Faire correspondre des enregistrements d’état protégé avec les enregistrements de pulsation

Cet exemple recherche les enregistrements d’état de protection et les enregistrements de pulsation associés, mis en correspondance en fonction de l’ordinateur et de l’heure.
Notez que le champ d’heure est arrondi à la minute la plus proche. Nous avons utilisé le calcul bin de runtime pour ce faire : `round_time=bin(TimeGenerated, 1m)`.

```Kusto
let protection_data = ProtectionStatus
    | project Computer, DetectionId, round_time=bin(TimeGenerated, 1m);
let heartbeat_data = Heartbeat
    | project Computer, Category, round_time=bin(TimeGenerated, 1m);
protection_data | join (heartbeat_data) on Computer, round_time
```

### <a name="server-availability-rate"></a>Taux de disponibilité du serveur

Calculez le taux de disponibilité du serveur en fonction des enregistrements de pulsation. La disponibilité est définie comme « au moins 1 pulsation par heure ».
Par conséquent, si un serveur a été disponible 98 heures sur 100, le taux de disponibilité est de 98 %.

```Kusto
let start_time=startofday(datetime("2018-03-01"));
let end_time=now();
Heartbeat
| where TimeGenerated > start_time and TimeGenerated < end_time
| summarize heartbeat_per_hour=count() by bin_at(TimeGenerated, 1h, start_time), Computer
| extend available_per_hour=iff(heartbeat_per_hour>0, true, false)
| summarize total_available_hours=countif(available_per_hour==true) by Computer 
| extend total_number_of_buckets=round((end_time-start_time)/1h)+1
| extend availability_rate=total_available_hours*100/total_number_of_buckets
```


## <a name="multiple-data-types"></a>Types de données multiples

### <a name="chart-the-record-count-per-table"></a>Tracer le nombre d’enregistrements par table
L’exemple suivant collecte tous les enregistrements de toutes les tables des 5 dernières heures et compte le nombre d’enregistrements dans chaque table. Ces résultats sont présentés dans un graphique temporel.

```Kusto
union withsource=sourceTable *
| where TimeGenerated > ago(5h) 
| summarize count() by bin(TimeGenerated,10m), sourceTable
| render timechart
```

### <a name="count-all-logs-collected-over-the-last-hour-by-type"></a>Compter tous les journaux d’activité collectés pendant la dernière heure écoulée, par type
L’exemple suivant recherche tous les éléments signalés au cours de la dernière heure écoulée et compte les enregistrements de chaque table par **type**. Par défaut, les résultats sont affichés dans un histogramme.

```Kusto
search *
| where TimeGenerated > ago(1h) 
| summarize CountOfRecords = count() by Type
| render barchart
```

## <a name="azurediagnostics"></a>AzureDiagnostics

### <a name="count-azure-diagnostics-records-per-category"></a>Compter les enregistrements des diagnostics Azure par catégorie
Cet exemple compte tous les enregistrements des diagnostics Azure pour chaque catégorie unique.

```Kusto
AzureDiagnostics 
| where TimeGenerated > ago(1d)
| summarize count() by Category
```

### <a name="get-a-random-record-for-each-unique-category"></a>Obtenir un enregistrement aléatoire pour chaque catégorie unique
Cet exemple obtient un seul enregistrement aléatoire des diagnostics Azure pour chaque catégorie unique.

```Kusto
AzureDiagnostics
| where TimeGenerated > ago(1d) 
| summarize any(*) by Category
```

### <a name="get-the-latest-record-per-category"></a>Obtenir le dernier enregistrement par catégorie
Cet exemple obtient le dernier enregistrement des diagnostics Azure pour chaque catégorie unique.

```Kusto
AzureDiagnostics
| where TimeGenerated > ago(1d) 
| summarize arg_max(TimeGenerated, *) by Category
```

## <a name="network-monitoring"></a>Analyse du réseau

### <a name="computers-with-unhealthy-latency"></a>Ordinateurs avec une latence inquiétante
Cet exemple répertorie les ordinateurs présentant une latence inquiétante.

```Kusto
NetworkMonitoring 
| where LatencyHealthState <> "Healthy" 
| where Computer != "" 
| distinct Computer
```

## <a name="performance"></a>Performances

### <a name="join-computer-perf-records-to-correlate-memory-and-cpu"></a>Joindre les enregistrements de performances informatiques pour mettre en corrélation la mémoire et l’UC
Cet exemple met en corrélation les enregistrements de **performances** d’un ordinateur particulier et crée deux graphiques temporels (un pour l’utilisation moyenne de l’UC et pour la mémoire maximale).

```Kusto
let StartTime = now()-5d;
let EndTime = now()-4d;
Perf
| where CounterName == "% Processor Time"  
| where TimeGenerated > StartTime and TimeGenerated < EndTime
| project TimeGenerated, Computer, cpu=CounterValue 
| join kind= inner (
   Perf
    | where CounterName == "% Used Memory"  
    | where TimeGenerated > StartTime and TimeGenerated < EndTime
    | project TimeGenerated , Computer, mem=CounterValue 
) on TimeGenerated, Computer
| summarize avgCpu=avg(cpu), maxMem=max(mem) by TimeGenerated bin=30m  
| render timechart
```

### <a name="perf-cpu-utilization-graph-per-computer"></a>Graphique de performances pour l’utilisation de l’UC par ordinateur
Cet exemple calcule et trace l’utilisation de l’UC des ordinateurs qui commencent par _Contoso_.

```Kusto
Perf
| where TimeGenerated > ago(4h)
| where Computer startswith "Contoso" 
| where CounterName == @"% Processor Time"
| summarize avg(CounterValue) by Computer, bin(TimeGenerated, 15m) 
| render timechart
```

## <a name="protection-status"></a>État de la protection

### <a name="computers-with-non-reporting-protection-status-duration"></a>Ordinateurs avec durée d’état protection sans rapport
Cet exemple répertorie les ordinateurs avec l’état de protection _Not Reporting_ et la durée de cet état.

```Kusto
ProtectionStatus
| where ProtectionStatus == "Not Reporting"
| summarize count(), startNotReporting = min(TimeGenerated), endNotReporting = max(TimeGenerated) by Computer, ProtectionStatusDetails
| join ProtectionStatus on Computer
| summarize lastReporting = max(TimeGenerated), startNotReporting = any(startNotReporting), endNotReporting = any(endNotReporting) by Computer
| extend durationNotReporting = endNotReporting - startNotReporting
```

### <a name="match-protected-status-records-with-heartbeat-records"></a>Faire correspondre des enregistrements d’état protégé avec les enregistrements de pulsation
Cet exemple recherche les enregistrements d’état de protection et les enregistrements de pulsation associés, mis en correspondance en fonction de l’ordinateur et de l’heure.
Le champ d’heure est arrondi à la minute la plus proche à l’aide de **bin**.

```Kusto
let protection_data = ProtectionStatus
    | project Computer, DetectionId, round_time=bin(TimeGenerated, 1m);
let heartbeat_data = Heartbeat
    | project Computer, Category, round_time=bin(TimeGenerated, 1m);
protection_data | join (heartbeat_data) on Computer, round_time
```


## <a name="security-records"></a>Enregistrements de sécurité

### <a name="count-security-events-by-activity-id"></a>Compter les événements de sécurité par ID d’activité


Cet exemple s’appuie sur la structure fixe de la colonne **Activity** : \<ID\>-\<Name\>.
Il analyse la valeur **Activity** dans deux nouvelles colonnes et compte les occurrences de chaque élément **activityID**.

```Kusto
SecurityEvent
| where TimeGenerated > ago(30m) 
| project Activity 
| parse Activity with activityID " - " activityDesc
| summarize count() by activityID
```

### <a name="count-security-events-related-to-permissions"></a>Compter les événements de sécurité liés aux autorisations
Cet exemple montre le nombre d’enregistrements **securityEvent** dans lesquels la colonne **Activité** contient le terme _Permissions_. La requête s’applique aux enregistrements créés au cours des 30 dernières minutes.

```Kusto
SecurityEvent
| where TimeGenerated > ago(30m)
| summarize EventCount = countif(Activity has "Permissions")
```

### <a name="find-accounts-that-failed-to-log-in-from-computers-with-a-security-detection"></a>Rechercher des comptes dont la connexion a échoué à partir d’ordinateurs avec une détection de sécurité
Cet exemple recherche et comptabilise les comptes sont la connexion a échoué à partir d’ordinateurs sur lesquels il existe une détection de sécurité.

```Kusto
let detections = toscalar(SecurityDetection
| summarize makeset(Computer));
SecurityEvent
| where Computer in (detections) and EventID == 4624
| summarize count() by Account
```

### <a name="is-my-security-data-available"></a>Mes données de sécurité sont-elles disponibles ?
L’exploration de données commence souvent par la vérification de la disponibilité des données. Cet exemple montre le nombre d’enregistrements **SecurityEvent** au cours des 30 dernières minutes.

```Kusto
SecurityEvent 
| where TimeGenerated  > ago(30m) 
| count
```

### <a name="parse-activity-name-and-id"></a>Analyser le nom d’activité et l’ID
Les deux exemples ci-dessous s’appuient sur la structure fixe de la colonne **Activity** : \<ID\>-\<Name\>. Le premier exemple utilise l’opérateur **parse** pour affecter des valeurs à deux nouvelles colonnes : **activityID** et **activityDesc**.

```Kusto
SecurityEvent
| take 100
| project Activity 
| parse Activity with activityID " - " activityDesc
```

Cet exemple utilise l’opérateur **split** pour créer un tableau de valeurs distinctes.
```Kusto
SecurityEvent
| take 100
| project Activity 
| extend activityArr=split(Activity, " - ") 
| project Activity , activityArr, activityId=activityArr[0]
```

### <a name="explicit-credentials-processes"></a>Processus d’informations d’identification explicites
L’exemple suivant montre un graphique à secteurs des processus qui ont utilisé des informations d’identification explicites la semaine précédente.

```Kusto
SecurityEvent
| where TimeGenerated > ago(7d)
// filter by id 4648 ("A logon was attempted using explicit credentials")
| where EventID == 4648
| summarize count() by Process
| render piechart 
```

### <a name="top-running-processes"></a>Processus les plus exécutés

L’exemple suivant montre une chronologie de l’activité pour les cinq processus les plus exécutés au cours des trois derniers jours.

```Kusto
// Find all processes that started in the last three days. ID 4688: A new process has been created.
let RunProcesses = 
    SecurityEvent
    | where TimeGenerated > ago(3d)
    | where EventID == "4688";
// Find the 5 processes that were run the most
let Top5Processes =
RunProcesses
| summarize count() by Process
| top 5 by count_;
// Create a time chart of these 5 processes - hour by hour
RunProcesses 
| where Process in (Top5Processes) 
| summarize count() by bin (TimeGenerated, 1h), Process
| render timechart
```


### <a name="find-repeating-failed-login-attempts-by-the-same-account-from-different-ips"></a>Rechercher la répétition de tentatives de connexion ayant échoué pour le même compte à partir de différentes adresses IP

L’exemple suivant recherche les tentatives de connexion ayant échoué pour le même compte à partir de plus de cinq adresses IP différentes au cours des six dernières heures.

```Kusto
SecurityEvent 
| where AccountType == "User" and EventID == 4625 and TimeGenerated > ago(6h) 
| summarize IPCount = dcount(IpAddress), makeset(IpAddress)  by Account
| where IPCount > 5
| sort by IPCount desc
```

### <a name="find-user-accounts-that-failed-to-log-in"></a>Rechercher les comptes d’utilisateur qui n’ont pas pu se connecter 
L’exemple suivant identifie les comptes d’utilisateur qui n’ont pas pu se connecter plus de cinq fois au cours des dernières 24 heures, et l’heure de la dernière tentative de connexion.

```Kusto
let timeframe = 1d;
SecurityEvent
| where TimeGenerated > ago(1d)
| where AccountType == 'User' and EventID == 4625 // 4625 - failed log in
| summarize failed_login_attempts=count(), latest_failed_login=arg_max(TimeGenerated, Account) by Account 
| where failed_login_attempts > 5
| project-away Account1
```

À l’aide des instructions **join** et **let**, nous pouvons vérifier si les mêmes comptes suspects ont été ultérieurement en mesure de se connecter.

```Kusto
let timeframe = 1d;
let suspicious_users = 
    SecurityEvent
    | where TimeGenerated > ago(timeframe)
    | where AccountType == 'User' and EventID == 4625 // 4625 - failed login
    | summarize failed_login_attempts=count(), latest_failed_login=arg_max(TimeGenerated, Account) by Account 
    | where failed_login_attempts > 5
    | project-away Account1;
let suspicious_users_that_later_logged_in = 
    suspicious_users 
    | join kind=innerunique (
        SecurityEvent
        | where TimeGenerated > ago(timeframe)
        | where AccountType == 'User' and EventID == 4624 // 4624 - successful login,
        | summarize latest_successful_login=arg_max(TimeGenerated, Account) by Account
    ) on Account
    | extend was_login_after_failures = iif(latest_successful_login>latest_failed_login, 1, 0)
    | where was_login_after_failures == 1
;
suspicious_users_that_later_logged_in
```

## <a name="usage"></a>Usage

Le type de données `Usage` permet d’assurer le suivi du volume de données ingérées par la solution ou le type de données. Il existe d’autres techniques pour étudier les volumes de données ingérées par [ordinateur](../platform/manage-cost-storage.md#data-volume-by-computer) ou [abonnement, groupe de ressources ou ressource Azure](../platform/manage-cost-storage.md#data-volume-by-azure-resource-resource-group-or-subscription).

#### <a name="data-volume-by-solution"></a>Volume de données par solution

La requête utilisée pour afficher le volume de données facturable par solution au cours du dernier mois (à l’exception du dernier jour partiel) est la suivante :

```kusto
Usage 
| where TimeGenerated > ago(32d)
| where StartTime >= startofday(ago(31d)) and EndTime < startofday(now())
| where IsBillable == true
| summarize BillableDataGB = sum(Quantity) / 1000. by bin(StartTime, 1d), Solution | render barchart
```

Notez que la clause `where IsBillable = true` exclut les types de données de certaines solutions pour lesquels il n'existe aucun frais d'ingestion.  De même, la clause avec `TimeGenerated` vise uniquement à vérifier que l’expérience de requête sur le portail Azure va au-delà des 24 dernières heures par défaut. Quand vous utilisez le type de données Usage, `StartTime` et `EndTime` représentent la période pour laquelle les résultats sont présentés. 

#### <a name="data-volume-by-type"></a>Volume de données par type

Vous pouvez approfondir pour afficher les tendances par type de données :

```kusto
Usage 
| where TimeGenerated > ago(32d)
| where StartTime >= startofday(ago(31d)) and EndTime < startofday(now())
| where IsBillable == true
| summarize BillableDataGB = sum(Quantity) / 1000. by bin(StartTime, 1d), DataType | render barchart
```

Ou pour afficher un tableau par solution et par type pour le mois précédent,

```kusto
Usage 
| where TimeGenerated > ago(32d)
| where StartTime >= startofday(ago(31d)) and EndTime < startofday(now())
| where IsBillable == true
| summarize BillableDataGB = sum(Quantity) / 1000. by Solution, DataType
| sort by Solution asc, DataType asc
```

> [!NOTE]
> Certains champs du type de données Utilisation, bien que faisant partie du schéma, sont maintenant déconseillés et leurs valeurs ne seront plus fournies. Il s’agit de **Computer** et des champs liées à l’ingestion (**TotalBatches**, **BatchesWithinSla**, **BatchesOutsideSla**,  **BatchesCapped** et **AverageProcessingTimeMs**.

## <a name="updates"></a>Mises à jour

### <a name="computers-still-missing-updates"></a>Ordinateurs avec mises à jour encore manquantes
Cet exemple répertorie les ordinateurs sur lesquels il manquait une ou plusieurs mises à jour critiques il y a quelques jours et pour lesquels le problème n’est pas encore résolu.

```Kusto
let ComputersMissingUpdates3DaysAgo = Update
| where TimeGenerated between (ago(30d)..ago(1h))
| where Classification !has "Critical" and UpdateState =~ "Needed"
| summarize makeset(Computer);
Update
| where TimeGenerated > ago(1d)
| where Classification has "Critical" and UpdateState =~ "Needed"
| where Computer in (ComputersMissingUpdates3DaysAgo)
| summarize UniqueUpdatesCount = dcount(Product) by Computer, OSType
```


## <a name="next-steps"></a>Étapes suivantes

- Reportez-vous aux [informations de référence sur le langage Kusto](/azure/kusto/query) pour en savoir plus.
- Suivez une [leçon sur l’écriture de requêtes de journal dans Azure Monitor](get-started-queries.md).
