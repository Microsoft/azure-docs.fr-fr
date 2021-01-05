---
title: Optimiser les requêtes de journal dans Azure Monitor
description: Bonnes pratiques pour l’optimisation des requêtes de journal dans Azure Monitor.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 03/30/2019
ms.openlocfilehash: a817c12a367d7c14f693389920e49b368a35cc06
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95522870"
---
# <a name="optimize-log-queries-in-azure-monitor"></a>Optimiser les requêtes de journal dans Azure Monitor
Journaux Azure Monitor utilise [Azure Data Explorer (ADX)](/azure/data-explorer/) pour stocker les données de journal et exécuter des requêtes afin d’analyser ces données. Elle crée et gère les clusters ADX, et les optimise pour votre charge de travail de l’analyse des journaux. Quand vous exécutez une requête, elle est optimisée et routée vers le cluster ADX approprié qui stocke les données de l’espace de travail. Journaux Azure Monitor et Azure Data Explorer utilisent de nombreux mécanismes d’optimisation automatique des requêtes. Bien que les optimisations automatiques apportent une amélioration significative, dans certains cas, vous pouvez améliorer considérablement les performances de vos requêtes. Cet article explique les considérations relatives aux performances et plusieurs techniques permettant de les corriger.

La plupart des techniques sont communes aux requêtes qui sont exécutées directement sur Azure Data Explorer et Journaux Azure Monitor, bien que plusieurs considérations sur Journaux Azure Monitor soient abordées ici. Pour plus d’informations sur les conseils d’optimisation propres à Azure Data Explorer, consultez [Bonnes pratiques pour les requêtes](/azure/kusto/query/best-practices).

Les requêtes optimisées :

- S’exécutent plus rapidement, avec une réduction de la durée d’exécution globale des requêtes.
- Ont moins de risques d’être limitées ou rejetées.

Vous devez accorder une attention particulière aux requêtes utilisées de façon récurrente ou en rafale, comme dans le cadre des tableaux de bord, des alertes, de Logic Apps et de Power BI. Dans ces cas, l’impact d’une requête inefficace est substantiel.

## <a name="query-performance-pane"></a>Volet des performances des requêtes
Après avoir exécuté une requête dans Log Analytics, cliquez sur la flèche vers le bas située au-dessus des résultats de la requête afin de voir le volet des performances de la requête qui affiche les résultats de plusieurs indicateurs de performance pour la requête. Chacun de ces indicateurs de performance est décrit dans la section suivante.

![Volet des performances des requêtes](media/query-optimization/query-performance-pane.png)


## <a name="query-performance-indicators"></a>Indicateurs de performances de requête

Les indicateurs de performances de requête suivants sont disponibles pour chaque requête exécutée :

- [Total Unité centrale](#total-cpu) : calcul global utilisé pour traiter la requête sur tous les nœuds de calcul. Il représente le temps utilisé pour le calcul, l’analyse et l’extraction de données. 

- [Données utilisées pour la requête traitée](#data-used-for-processed-query) : données globales consultées pour traiter la requête. Influencé par la taille de la table cible, l’intervalle de temps utilisé, les filtres appliqués et le nombre de colonnes référencées.

- [Intervalle de temps de la requête traitée](#time-span-of-the-processed-query) : intervalle entre les données les plus récentes et les plus anciennes consultées pour traiter la requête. Influencé par l’intervalle de temps explicite spécifié pour la requête.

- [Âge des données traitées](#age-of-processed-data) : intervalle entre maintenant et les données les plus consultées pour traiter la requête. Il influence fortement l’efficacité de l’extraction des données.

- [Nombre d’espaces de travail](#number-of-workspaces) : nombre d’espaces de travail consultés pendant le traitement de la requête en raison d’une sélection implicite ou explicite.

- [Nombre de régions](#number-of-regions) : nombre de régions consultées pendant le traitement de la requête en raison d’une sélection implicite ou explicite d’espaces de travail. Les requêtes multirégions sont bien moins efficaces et les indicateurs de performance offrent une couverture partielle.

- [Parallélisme](#parallelism) : Indique dans quelle mesure le système a pu exécuter cette requête sur plusieurs nœuds. S’applique uniquement aux requêtes qui ont une consommation élevée du processeur. Influencé par l’utilisation de fonctions et d’opérateurs spécifiques.


## <a name="total-cpu"></a>Total Unité centrale
Processeur de calcul réel consacré au traitement de cette requête sur tous les nœuds de traitement des requêtes. Étant donné que la plupart des requêtes sont exécutées sur un grand nombre de nœuds, cette valeur est généralement beaucoup plus grande que la durée réelle de l’exécution de la requête. 

Une requête qui utilise plus de 100 secondes d’UC est considérée comme une requête consommant une quantité excessive de ressources. Une requête qui utilise plus de 1 000 secondes d’UC est considérée comme une requête abusive et peut être limitée.

Le temps de traitement des requêtes est consacré aux opérations suivantes :
- Extraction de données : l’extraction d’anciennes données consomme plus de temps que l’extraction de données récentes.
- Traitement des données : logique et évaluation des données. 

En dehors du temps passé dans les nœuds de traitement des requêtes, Journaux Azure Monitor consacre également du temps aux opérations suivantes : authentifier l’utilisateur et vérifier qu’il est autorisé à accéder à ces données, localiser le magasin de données, analyser la requête et allouer les nœuds de traitement de la requête. Cette durée n’est pas incluse dans le temps processeur total des requêtes.

### <a name="early-filtering-of-records-prior-of-using-high-cpu-functions"></a>Filtrage précoce des enregistrements avant d’utiliser des fonctions à UC élevée

Certaines des commandes et des fonctions de requête sont gourmandes en ressources processeur. Cela est particulièrement vrai pour les commandes qui analysent des éléments JSON et XML ou qui extraient des expressions régulières complexes. Une analyse de ce type peut se produire explicitement par le biais des fonctions [parse_json()](/azure/kusto/query/parsejsonfunction) ou [parse_xml()](/azure/kusto/query/parse-xmlfunction) ou implicitement quand il est fait référence à des colonnes dynamiques.

Ces fonctions consomment des ressources processeur proportionnellement au nombre de lignes qu’elles traitent. L’optimisation la plus efficace consiste à ajouter des conditions WHERE tôt dans la requête, qui peuvent filtrer le plus grand nombre d’enregistrements possible avant l’exécution de la fonction gourmande en ressources processeur.

Par exemple, les requêtes suivantes produisent exactement le même résultat, mais la seconde est beaucoup plus efficace car la condition [where](/azure/kusto/query/whereoperator) avant l’analyse exclut de nombreux enregistrements :

```Kusto
//less efficient
SecurityEvent
| extend Details = parse_xml(EventData)
| extend FilePath = tostring(Details.UserData.RuleAndFileData.FilePath)
| extend FileHash = tostring(Details.UserData.RuleAndFileData.FileHash)
| where FileHash != "" and FilePath !startswith "%SYSTEM32"  // Problem: irrelevant results are filtered after all processing and parsing is done
| summarize count() by FileHash, FilePath
```
```Kusto
//more efficient
SecurityEvent
| where EventID == 8002 //Only this event have FileHash
| where EventData !has "%SYSTEM32" //Early removal of unwanted records
| extend Details = parse_xml(EventData)
| extend FilePath = tostring(Details.UserData.RuleAndFileData.FilePath)
| extend FileHash = tostring(Details.UserData.RuleAndFileData.FileHash)
| where FileHash != "" and FilePath !startswith "%SYSTEM32"  // exact removal of results. Early filter is not accurate enough
| summarize count() by FileHash, FilePath
| where FileHash != "" // No need to filter out %SYSTEM32 here as it was removed before
```

### <a name="avoid-using-evaluated-where-clauses"></a>Évitez d’utiliser des clauses WHERE évaluées

Les requêtes qui contiennent des clauses [where](/azure/kusto/query/whereoperator) sur une colonne évaluée plutôt que sur des colonnes qui sont physiquement présentes dans le jeu de données perdent en efficacité. Le filtrage sur les colonnes évaluées empêche certaines optimisations système quand de grands ensembles de données sont gérés.
Par exemple, les requêtes suivantes produisent exactement le même résultat, mais la seconde est plus efficace car la condition [where](/azure/kusto/query/whereoperator) fait référence à une colonne intégrée :

```Kusto
//less efficient
Syslog
| extend Msg = strcat("Syslog: ",SyslogMessage)
| where  Msg  has "Error"
| count 
```
```Kusto
//more efficient
Syslog
| where  SyslogMessage  has "Error"
| count 
```

Dans certains cas, la colonne évaluée est créée implicitement par le moteur de traitement des requêtes puisque le filtrage ne se fait pas uniquement sur le champ :
```Kusto
//less efficient
SecurityEvent
| where tolower(Process) == "conhost.exe"
| count 
```
```Kusto
//more efficient
SecurityEvent
| where Process =~ "conhost.exe"
| count 
```




### <a name="use-effective-aggregation-commands-and-dimensions-in-summarize-and-join"></a>Utilisez des commandes d’agrégation et des dimensions efficaces dans la synthèse et la jointure

Bien que certaines commandes d’agrégation telles que [max()](/azure/kusto/query/max-aggfunction), [sum()](/azure/kusto/query/sum-aggfunction), [count()](/azure/kusto/query/count-aggfunction) et [avg()](/azure/kusto/query/avg-aggfunction) aient un faible impact sur le processeur en raison de leur logique, d’autres sont plus complexes et incluent des heuristiques et des estimations qui leur permettent d’être exécutées efficacement. Par exemple, [dcount()](/azure/kusto/query/dcount-aggfunction) utilise l’algorithme HyperLogLog pour fournir une estimation proche du nombre distinct de grands jeux de données sans compter réellement chaque valeur ; les fonctions centile réalisent des approximations similaires à l’aide de l’algorithme de rang centile le plus proche. Plusieurs commandes incluent des paramètres facultatifs pour réduire leur impact. Par exemple, la fonction [makeset()](/azure/kusto/query/makeset-aggfunction) a un paramètre facultatif pour définir la taille maximale du jeu, ce qui affecte de manière significative le processeur et la mémoire.

Les commandes [join](/azure/kusto/query/joinoperator?pivots=azuremonitor) and [summarize](/azure/kusto/query/summarizeoperator) peuvent entraîner une utilisation élevée du processeur quand elles traitent un grand ensemble de données. Leur complexité est directement liée au nombre de valeurs possibles, appelé *cardinalité*, des colonnes qui sont utilisées comme `by` dans la commande summarize ou comme attributs dans la commande join. Pour plus d’informations sur les commandes join et summarize et sur leur optimisation, consultez les articles de la documentation et les conseils d’optimisation qui leur sont consacrés.

Par exemple, les requêtes suivantes produisent exactement le même résultat, car **CounterPath** est toujours mappé un-à-un à **CounterName** et **ObjectName**. La seconde est plus efficace, car la dimension d’agrégation est plus petite :

```Kusto
//less efficient
Perf
| summarize avg(CounterValue) 
by CounterName, CounterPath, ObjectName
```
```Kusto
//make the group expression more compact improve the performance
Perf
| summarize avg(CounterValue), any(CounterName), any(ObjectName) 
by CounterPath
```

La consommation du processeur peut également être impactée par les conditions where ou les colonnes étendues nécessitant un traitement intensif. Toutes les comparaisons de chaînes triviales telles que [equal ==](/azure/kusto/query/datatypes-string-operators) et [startswith](/azure/kusto/query/datatypes-string-operators) ont à peu près le même impact sur le processeur, tandis que les correspondances de texte avancées ont plus d’impact. Plus précisément, l’opérateur [has](/azure/kusto/query/datatypes-string-operators) est plus efficace que l’opérateur [contains](/azure/kusto/query/datatypes-string-operators). En raison des techniques de traitement des chaînes, il est plus efficace de rechercher des chaînes qui comportent plus de quatre caractères que des chaînes courtes.

Par exemple, les requêtes suivantes produisent des résultats similaires, en fonction de la stratégie de nommage Computer, mais la deuxième est plus efficace :

```Kusto
//less efficient – due to filter based on contains
Heartbeat
| where Computer contains "Production" 
| summarize count() by ComputerIP 
```
```Kusto
//less efficient – due to filter based on extend
Heartbeat
| extend MyComputer = Computer
| where MyComputer startswith "Production" 
| summarize count() by ComputerIP 
```
```Kusto
//more efficient
Heartbeat
| where Computer startswith "Production" 
| summarize count() by ComputerIP 
```

> [!NOTE]
> Cet indicateur présente uniquement le processeur du cluster immédiat. Dans une requête multirégion, il ne représente qu’une seule des régions. Dans les requêtes à plusieurs espaces de travail, il n’inclut probablement pas tous les espaces de travail.

### <a name="avoid-full-xml-and-json-parsing-when-string-parsing-works"></a>Évitez l’analyse XML et JSON complète lorsque l’analyse de chaîne fonctionne
L’analyse complète d’un objet XML ou JSON peut consommer des ressources de processeur et de mémoire élevées. Dans de nombreux cas, lorsqu’un seul ou deux paramètres sont nécessaires et que les objets XML ou JSON sont simples, il est plus facile de les analyser en tant que chaînes à l’aide de [l’opérateur parse](/azure/kusto/query/parseoperator) ou d’autres [techniques d’analyse de texte](./parse-text.md). L’amélioration des performances sera plus importante à mesure que le nombre d’enregistrements dans l’objet XML ou JSON augmente. Cela est essentiel lorsque le nombre d’enregistrements atteint des dizaines de millions.

Par exemple, la requête suivante renverra exactement les mêmes résultats que les requêtes ci-dessus sans effectuer d’analyse XML complète. Notez qu’elle fait des hypothèses sur la structure du fichier XML, par exemple le fait que l’élément FilePath vient après FileHash et qu’aucun d’entre eux n’a d’attributs. 

```Kusto
//even more efficient
SecurityEvent
| where EventID == 8002 //Only this event have FileHash
| where EventData !has "%SYSTEM32" //Early removal of unwanted records
| parse EventData with * "<FilePath>" FilePath "</FilePath>" * "<FileHash>" FileHash "</FileHash>" *
| summarize count() by FileHash, FilePath
| where FileHash != "" // No need to filter out %SYSTEM32 here as it was removed before
```


## <a name="data-used-for-processed-query"></a>Données utilisées pour la requête traitée

Un facteur critique dans le traitement de la requête est le volume de données analysé et utilisé. Azure Data Explorer utilise des optimisations agressives qui réduisent considérablement le volume de données par rapport aux autres plateformes de données. Toutefois, il existe des facteurs critiques dans la requête qui peuvent impacter le volume de données utilisé.

Une requête qui traite plus de 2 000 Ko de données est considérée comme une requête consommant une quantité excessive de ressources. Une requête qui traite plus de 20 000KB de données est considérée comme une requête abusive et peut être limitée.

Dans Journaux Azure Monitor, la colonne **TimeGenerated** est utilisée comme mode d’indexation des données. Le fait de limiter les valeurs **TimeGenerated** à une plage aussi fine que possible améliore considérablement les performances des requêtes en limitant de manière significative la quantité de données à traiter.

### <a name="avoid-unnecessary-use-of-search-and-union-operators"></a>Évitez l’utilisation inutile des opérateurs de recherche et d’union

Un autre facteur qui augmente les données traitées est l’utilisation d’un grand nombre de tables. Cela se produit généralement quand des commandes `search *` et `union *` sont utilisées. Ces commandes forcent le système à évaluer et à analyser les données de toutes les tables de l’espace de travail. Dans certains cas, l’espace de travail peut comporter des centaines de tables. Essayez d’éviter autant que possible l’utilisation de « search * » ou d’une recherche sans la limiter à une table spécifique.

Par exemple, les requêtes suivantes produisent exactement le même résultat, mais la dernière est de loin la plus efficace :

```Kusto
// This version scans all tables though only Perf has this kind of data
search "Processor Time" 
| summarize count(), avg(CounterValue)  by Computer
```
```Kusto
// This version scans all strings in Perf tables – much more efficient
Perf
| search "Processor Time" 
| summarize count(), avg(CounterValue)  by Computer
```
```Kusto
// This is the most efficient version 
Perf 
| where CounterName == "% Processor Time"  
| summarize count(), avg(CounterValue)  by Computer
```

### <a name="add-early-filters-to-the-query"></a>Ajoutez des filtres précoces à la requête

Une autre méthode pour réduire le volume de données consiste à placer les conditions [where](/azure/kusto/query/whereoperator) tôt dans la requête. La plateforme Azure Data Explorer inclut un cache qui lui permet de savoir quelles partitions incluent des données pertinentes pour une condition where spécifique. Par exemple, si une requête contient `where EventID == 4624`, elle distribue la requête uniquement aux nœuds qui gèrent des partitions avec des événements correspondants.

Les exemples de requêtes suivants produisent exactement le même résultat, mais le second est plus efficace :

```Kusto
//less efficient
SecurityEvent
| summarize LoginSessions = dcount(LogonGuid) by Account
```
```Kusto
//more efficient
SecurityEvent
| where EventID == 4624 //Logon GUID is relevant only for logon event
| summarize LoginSessions = dcount(LogonGuid) by Account
```

### <a name="avoid-multiple-scans-of-same-source-data-using-conditional-aggregation-functions-and-materialize-function"></a>Évitez les analyses multiples des mêmes données sources à l’aide des fonctions d’agrégation conditionnelles et de la fonction materialize
Lorsqu’une requête comporte plusieurs sous-requêtes fusionnées à l’aide d’opérateurs de jointure ou d’union, chaque sous-requête analyse l’intégralité de la source séparément, puis fusionne les résultats. Il s’agit du nombre de fois où les données sont analysées ; un facteur critique dans les jeux de données très volumineux.

Pour éviter cela, vous pouvez utiliser les fonctions d’agrégation conditionnelles. La plupart des [fonctions d’agrégation](/azure/data-explorer/kusto/query/summarizeoperator#list-of-aggregation-functions) utilisées dans l’opérateur summary ont une version conditionnée qui vous permet d’utiliser un seul opérateur de synthèse avec plusieurs conditions. 

Par exemple, les requêtes suivantes affichent le nombre d’événements de connexion et le nombre d’événements d’exécution de processus pour chaque compte. Elles retournent les mêmes résultats, mais la première analyse les données deux fois, la deuxième une seule fois :

```Kusto
//Scans the SecurityEvent table twice and perform expensive join
SecurityEvent
| where EventID == 4624 //Login event
| summarize LoginCount = count() by Account
| join 
(
    SecurityEvent
    | where EventID == 4688 //Process execution event
    | summarize ExecutionCount = count(), ExecutedProcesses = make_set(Process) by Account
) on Account
```

```Kusto
//Scan only once with no join
SecurityEvent
| where EventID == 4624 or EventID == 4688 //early filter
| summarize LoginCount = countif(EventID == 4624), ExecutionCount = countif(EventID == 4688), ExecutedProcesses = make_set_if(Process,EventID == 4688)  by Account
```

Un autre cas où les sous-requêtes ne sont pas nécessaires est le préfiltrage pour [l’opérateur parse](/azure/data-explorer/kusto/query/parseoperator?pivots=azuremonitor) pour s’assurer qu’il traite uniquement les enregistrements qui correspondent à un modèle spécifique. Cela n’est pas nécessaire, car l’opérateur parse et d’autres opérateurs similaires retournent des résultats vides lorsque le modèle ne correspond pas. Voici deux requêtes qui retournent exactement les mêmes résultats, tandis que la deuxième requête analyse les données une seule fois. Dans la deuxième requête, chaque commande parse est pertinente uniquement pour ses événements. L’opérateur Extend montre ensuite comment faire référence à une situation de données vides.

```Kusto
//Scan SecurityEvent table twice
union(
SecurityEvent
| where EventID == 8002 
| parse EventData with * "<FilePath>" FilePath "</FilePath>" * "<FileHash>" FileHash "</FileHash>" *
| distinct FilePath
),(
SecurityEvent
| where EventID == 4799
| parse EventData with * "CallerProcessName\">" CallerProcessName1 "</Data>" * 
| distinct CallerProcessName1
)
```

```Kusto
//Single scan of the SecurityEvent table
SecurityEvent
| where EventID == 8002 or EventID == 4799
| parse EventData with * "<FilePath>" FilePath "</FilePath>" * "<FileHash>" FileHash "</FileHash>" * //Relevant only for event 8002
| parse EventData with * "CallerProcessName\">" CallerProcessName1 "</Data>" *  //Relevant only for event 4799
| extend FilePath = iif(isempty(CallerProcessName1),FilePath,"")
| distinct FilePath, CallerProcessName1
```

Lorsque l’option ci-dessus n’autorise pas l’utilisation de sous-requêtes, une autre technique consiste à indiquer au moteur de requête qu’une seule source de données est utilisée dans chacune d’elles à l’aide de la fonction [materialize()](/azure/data-explorer/kusto/query/materializefunction?pivots=azuremonitor). Cela est utile lorsque les données sources proviennent d’une fonction qui est utilisée plusieurs fois dans la requête. La fonction materialize est efficace lorsque la sortie de la sous-requête est bien plus petite que l’entrée. Le moteur de requête mettra en cache et réutilisera la sortie dans toutes les occurrences.



### <a name="reduce-the-number-of-columns-that-is-retrieved"></a>Réduisez le nombre de colonnes récupérées

Étant donné qu’Azure Data Explorer est un magasin de données en colonnes, la récupération de chaque colonne est indépendante des autres. Le nombre de colonnes récupérées influence directement le volume de données global. Vous ne devez inclure dans la sortie que les colonnes nécessaires ; pour ce faire, [résumez](/azure/kusto/query/summarizeoperator) les résultats ou [projetez](/azure/kusto/query/projectoperator) les colonnes concernées. Azure Data Explorer dispose de plusieurs optimisations pour réduire le nombre de colonnes récupérées. S’il détermine qu’une colonne n’est pas nécessaire, par exemple si elle n’est pas référencée dans la commande [summarize](/azure/kusto/query/summarizeoperator), il ne la récupère pas.

Par exemple, il est possible que la seconde requête traite trois fois plus de données, car elle doit extraire non pas une colonne, mais trois :

```Kusto
//Less columns --> Less data
SecurityEvent
| summarize count() by Computer  
```
```Kusto
//More columns --> More data
SecurityEvent
| summarize count(), dcount(EventID), avg(Level) by Computer  
```

## <a name="time-span-of-the-processed-query"></a>Intervalle de temps de la requête traitée

Tous les journaux dans Journaux Azure Monitor sont partitionnés en fonction de la colonne **TimeGenerated**. Le nombre de partitions accessibles est directement lié à l’intervalle de temps. La réduction de l’intervalle de temps est la méthode la plus efficace pour garantir l’exécution d’une requête d’invite.

Une requête sur une période de plus de 15 jours est considérée comme une requête consommant une quantité excessive de ressources. Une requête sur une période de plus de 90 jours est considérée comme une requête abusive et peut être limitée.

L’intervalle de temps peut être défini à l’aide du sélecteur d’intervalle de temps dans l’écran Log Analytics, comme décrit dans [Étendue de requête de journal et intervalle de temps dans la fonctionnalité Log Analytics d’Azure Monitor](scope.md#time-range). Il s’agit de la méthode recommandée, car l’intervalle de temps sélectionné est transmis au back-end à l’aide des métadonnées de requête. 

Une autre méthode consiste à inclure explicitement une condition [where](/azure/kusto/query/whereoperator) sur **TimeGenerated** dans la requête. Vous devez utiliser cette méthode, car elle garantit que l’intervalle de temps est fixe, même quand la requête est utilisée à partir d’une interface différente.
Vous devez vous assurer que toutes les parties de la requête ont des filtres **TimeGenerated**. Quand une requête a des sous-requêtes qui extraient des données de différentes tables ou de la même table, chacune doit inclure sa propre [where](/azure/kusto/query/whereoperator).

### <a name="make-sure-all-sub-queries-have-timegenerated-filter"></a>Vérifiez que toutes les sous-requêtes ont un filtre TimeGenerated

Par exemple, dans la requête suivante, tandis que la table **Perf** est analysée uniquement sur le dernier jour, la table **Heartbeat** est analysée sur tout son historique, ce qui peut prendre jusqu’à deux ans :

```Kusto
Perf
| where TimeGenerated > ago(1d)
| summarize avg(CounterValue) by Computer, CounterName
| join kind=leftouter (
    Heartbeat
    //No time span filter in this part of the query
    | summarize IPs = makeset(ComputerIP, 10) by  Computer
) on Computer
```

Ce type d’erreur se produit généralement quand [arg_max()](/azure/kusto/query/arg-max-aggfunction) est utilisé pour rechercher l’occurrence la plus récente. Par exemple :

```Kusto
Perf
| where TimeGenerated > ago(1d)
| summarize avg(CounterValue) by Computer, CounterName
| join kind=leftouter (
    Heartbeat
    //No time span filter in this part of the query
    | summarize arg_max(TimeGenerated, *), min(TimeGenerated)   
by Computer
) on Computer
```

Vous pouvez facilement corriger ce problème en ajoutant un filtre de temps dans la requête interne :

```Kusto
Perf
| where TimeGenerated > ago(1d)
| summarize avg(CounterValue) by Computer, CounterName
| join kind=leftouter (
    Heartbeat
    | where TimeGenerated > ago(1d) //filter for this part
    | summarize arg_max(TimeGenerated, *), min(TimeGenerated)   
by Computer
) on Computer
```

Ce problème peut également se produire lorsque vous effectuez un filtrage par période juste après l’[union](/azure/kusto/query/unionoperator?pivots=azuremonitor) de plusieurs tables. Lorsque vous effectuez l’union, chaque sous-requête doit être délimitée. Vous pouvez utiliser l’instruction [let](/azure/kusto/query/letstatement) pour garantir la cohérence de la portée.

Par exemple, la requête suivante analyse toutes les données des tables *Heartbeat* et *Perf*, et pas seulement celles du jour passé :

```Kusto
Heartbeat 
| summarize arg_min(TimeGenerated,*) by Computer
| union (
    Perf 
    | summarize arg_min(TimeGenerated,*) by Computer) 
| where TimeGenerated > ago(1d)
| summarize min(TimeGenerated) by Computer
```

Cette requête doit être corrigée de la façon suivante :

```Kusto
let MinTime = ago(1d);
Heartbeat 
| where TimeGenerated > MinTime
| summarize arg_min(TimeGenerated,*) by Computer
| union (
    Perf 
    | where TimeGenerated > MinTime
    | summarize arg_min(TimeGenerated,*) by Computer) 
| summarize min(TimeGenerated) by Computer
```

### <a name="time-span-measurement-limitations"></a>Limitations de mesure de l’intervalle de temps

La mesure est toujours supérieure à la période spécifiée. Par exemple, si le filtre appliqué à la requête est de 7 jours, le système peut analyser 7,5 ou 8,1 jours. Cela est dû au fait que le système partitionne les données en segments de taille variable. Pour que tous les enregistrements pertinents soient analysés, il analyse l’intégralité de la partition, qui peut couvrir plusieurs heures, voire plus d’une journée.

Il existe plusieurs cas où le système ne peut pas fournir une mesure exacte de l’intervalle de temps. C’est souvent le cas lorsque la période d’une requête est inférieure à un jour ou lorsque les requêtes englobent plusieurs espaces de travail.


> [!IMPORTANT]
> Cet indicateur présente uniquement les données traitées dans le cluster immédiat. Dans une requête multirégion, il ne représente qu’une seule des régions. Dans les requêtes à plusieurs espaces de travail, il n’inclut probablement pas tous les espaces de travail.

## <a name="age-of-processed-data"></a>Âge des données traitées
Azure Data Explorer utilise plusieurs niveaux de stockage : en mémoire, disques SSD locaux et objets blob Azure beaucoup plus lents. Plus les données sont récentes, plus il est probable qu’elles soient stockées dans un niveau plus performant avec une latence plus faible, ce qui réduit la durée de la requête et la consommation des ressources processeur. Outre pour les données elles-mêmes, le système dispose d’un cache pour les métadonnées. Plus les données sont anciennes, moins il est probable que leurs métadonnées se trouvent dans le cache.

Une requête qui traite des données datant de plus de 14 jours est considérée comme une requête consommant une quantité excessive de ressources.


Bien que certaines requêtes nécessitent l’utilisation d’anciennes données, il existe des cas où les anciennes données sont utilisées par erreur. Cela se produit quand les requêtes sont exécutées sans que soit fourni d’intervalle de temps dans leurs métadonnées et que toutes les références de table n’incluent pas le filtre sur la colonne **TimeGenerated**. Dans ces cas, le système analyse toutes les données stockées dans la table concernée. Quand la conservation des données est longue, elle peut couvrir des intervalles de temps longs et, donc, des données aussi anciennes que la période de conservation des données.

Voici certains exemples :

- Le fait de ne pas définir l’intervalle de temps dans Log Analytics avec une sous-requête qui n’est pas limitée. Consultez l'exemple ci-dessus.
- Utilisation de l’API sans les paramètres d’intervalle de temps facultatifs.
- Utilisation d’un client qui ne force pas un intervalle de temps, tel que le connecteur Power BI.

Consultez les exemples et les notes de la section précédente, car ils sont également pertinents dans ce cas.

## <a name="number-of-regions"></a>Nombre de régions
Il existe plusieurs situations où une seule requête peut être exécutée dans différentes régions :

- Quand plusieurs espaces de travail sont listés explicitement et qu’ils se trouvent dans des régions différentes.
- Quand une requête dont l’étendue est limitée à des ressources récupère des données et que les données sont stockées dans plusieurs espaces de travail qui se trouvent dans des régions différentes.

L’exécution d’une requête inter-région nécessite que le système sérialise et transfère dans le back-end de grands segments de données intermédiaires qui sont généralement beaucoup plus volumineux que les résultats finaux de la requête. Elle limite également la capacité du système à effectuer des optimisations, des heuristiques et à utiliser des caches.
S’il n’y a pas vraiment de raison d’analyser toutes ces régions, vous devez ajuster l’étendue afin qu’elle couvre moins de régions. Si l’étendue de ressource est réduite, mais que de nombreuses régions sont toujours utilisées, la raison peut en être une mauvaise configuration. Par exemple, les journaux d’audit et les paramètres de diagnostic sont envoyés à différents espaces de travail dans différentes régions ou il existe plusieurs configurations de paramètres de diagnostic. 

Une requête qui s’étend à plus de 3 régions est considérée comme une requête consommant une quantité excessive de ressources. Une requête qui s’étend à plus de 6 régions est considérée comme une requête abusive et peut être limitée.

> [!IMPORTANT]
> Lorsqu’une requête est exécutée dans plusieurs régions à la fois, les mesures du processeur et des données ne sont pas exactes, car elles représentent uniquement les mesures de l’une des régions.

## <a name="number-of-workspaces"></a>Nombre d’espaces de travail
Les espaces de travail sont des conteneurs logiques qui permettent de séparer et d’administrer des données de journaux. Le back-end optimise le positionnement des espaces de travail sur les clusters physiques au sein de la région sélectionnée.

L’utilisation de plusieurs espaces de travail peut résulter : 

- Du listage explicite de plusieurs espaces de travail.
- De la récupération par une requête dont l’étendue est limitée à des ressources de données qui sont stockées dans plusieurs espaces de travail.
 
L’exécution de requêtes inter-région et inter-cluster nécessite que le système sérialise et transfère dans le back-end de grands segments de données intermédiaires qui sont généralement beaucoup plus volumineux que les résultats finaux de la requête. Elle limite également la capacité du système à effectuer des optimisations, des heuristiques et à utiliser des caches.

Une requête qui s’étend à plus de 5 espaces de travail est considérée comme une requête consommant une quantité excessive de ressources. Les requêtes ne peuvent pas s’étendre à plus de 100 espaces de travail.

> [!IMPORTANT]
> Dans certains scénarios à plusieurs espaces de travail, les mesures du processeur et des données ne sont pas précises et représentent la mesure uniquement de quelques-uns des espaces de travail.

## <a name="parallelism"></a>Parallélisme
Pour exécuter des requêtes, les journaux Azure Monitor utilisent de grands clusters Azure Data Explorer. L’échelle de ces clusters peut varier et atteindre plusieurs dizaines de nœuds de calcul. Le système met automatiquement à l’échelle les clusters en fonction de la capacité et de la logique de positionnement de l’espace de travail.

Pour qu’une requête soit exécutée efficacement, elle est partitionnée et distribuée aux nœuds de calcul en fonction des données nécessaires à son traitement. Dans certains cas, le système ne peut pas effectuer cette opération de manière efficace. Cela peut entraîner une requête de longue durée. 

Les comportements de requête qui peuvent réduire le parallélisme sont les suivants :

- Utilisation de fonctions de sérialisation et de fenêtre, telles que les fonctions [next()](/azure/kusto/query/nextfunction), [prev()](/azure/kusto/query/prevfunction) et [row](/azure/kusto/query/rowcumsumfunction) ainsi que l’[opérateur serialize](/azure/kusto/query/serializeoperator). Les fonctions de série chronologique et d’analytique utilisateur peuvent être utilisées dans certains de ces cas. Une sérialisation inefficace peut également se produire si les opérateurs suivants ne sont pas utilisés à la fin de la requête : [range](/azure/kusto/query/rangeoperator), [sort](/azure/kusto/query/sortoperator), [order](/azure/kusto/query/orderoperator), [top](/azure/kusto/query/topoperator), [top-hitters](/azure/kusto/query/tophittersoperator), [getschema](/azure/kusto/query/getschemaoperator).
-    L’utilisation de la fonction d’agrégation [dcount()](/azure/kusto/query/dcount-aggfunction) force le système à avoir une copie centrale des différentes valeurs. Quand l’échelle de données est élevée, envisagez d’utiliser les paramètres facultatifs de la fonction dcount pour réduire la précision.
-    Dans de nombreux cas, l’opérateur [join](/azure/kusto/query/joinoperator?pivots=azuremonitor) diminue le parallélisme global. Envisagez une jointure aléatoire comme alternative quand les performances sont problématiques.
-    Dans les requêtes dont l’étendue est limitée à des ressources, les vérifications Kubernetes RBAC ou Azure RBAC en préexécution peuvent traîner en longueur en présence d’un très grand nombre d’attributions de rôle Azure. Cela peut entraîner des contrôles plus longs susceptibles de réduire le parallélisme. Par exemple, une requête est exécutée sur un abonnement qui comprend des milliers de ressources, et de nombreuses attributions de rôle sont définies au niveau de chaque ressource, et non sur l’abonnement ou sur le groupe de ressources.
-    Si une requête traite de petits blocs de données, son parallélisme est faible, car le système ne la répartit pas sur de nombreux nœuds de calcul.



## <a name="next-steps"></a>Étapes suivantes

- [Documentation de référence sur le langage de requête Kusto](/azure/kusto/query/).
