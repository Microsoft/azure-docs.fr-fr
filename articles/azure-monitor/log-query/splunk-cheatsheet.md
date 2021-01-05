---
title: De Splunk à la requête de journal Azure Monitor | Microsoft Docs
description: Aide à l’apprentissage des requêtes de journal Azure Monitor pour les utilisateurs familiarisés avec Splunk.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 08/21/2018
ms.openlocfilehash: c59b5646e011afa6b8487e8145a1cb07e6e2a8ff
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95015577"
---
# <a name="splunk-to-azure-monitor-log-query"></a>De Splunk à la requête de journal Azure Monitor

Cet article a pour but d’aider les utilisateurs qui connaissent Splunk, à apprendre le langage de requête Kusto pour écrire des requêtes de journal dans Azure Monitor. Des comparaisons directes sont établies entre les deux pour vous permettre de comprendre les principales différences et les similitudes, et éventuellement d’approfondir vos connaissances.

## <a name="structure-and-concepts"></a>Structure et concepts

Le tableau suivant compare les concepts et les structures de données de Splunk et des journaux d’activité Azure Monitor.

 | Concept  | Splunk | Azure Monitor |  Commentaire
 | --- | --- | --- | ---
 | Unité de déploiement  | cluster |  cluster |  Azure Monitor autorise les requêtes entre clusters arbitraires. Ce n’est pas le cas de Splunk. |
 | Caches de données |  compartiments  |  Stratégies de rétention et mise en cache |  Contrôle la période et le niveau de mise en cache des données. Ce paramètre a un impact direct sur les performances des requêtes et le coût du déploiement. |
 | Partition logique des données  |  index  |  database  |  Permet une séparation logique des données. Les deux implémentations autorisent les unions et les jointures entre ces partitions. |
 | Métadonnées d’événement structurées | N/A | table |  Splunk n’a pas le concept exposé au langage de recherche de métadonnées d’événement. Les journaux d’activité Azure Monitor disposent du concept de table comportant des colonnes. Chaque instance d’événement est mappée à une ligne. |
 | Enregistrement de données | événement | ligne |  Changement de terminologie uniquement. |
 | Attribut d’enregistrement de données | field |  colonne |  Dans Azure Monitor, elle est prédéfinie dans le cadre de la structure de la table. Dans Splunk, chaque événement possède son propre ensemble de champs. |
 | Types | type de données |  type de données |  Les types de données Azure Monitor sont plus explicites, car ils sont définis sur les colonnes. Les deux ont la possibilité d’utiliser dynamiquement des types de données et un ensemble de types de données à peu près équivalent, notamment la prise en charge JSON. |
 | Requête et recherche  | recherche | query |  Les concepts sont essentiellement les mêmes entre Azure Monitor et Splunk. |
 | Durée d’ingestion des événements | Temps système | ingestion_time() |  Dans Splunk, chaque événement obtient un horodatage système de l’heure à laquelle l’événement a été indexé. Dans Azure Monitor, vous pouvez définir une stratégie nommée ingestion_time qui expose une colonne système à laquelle la fonction ingestion_time() fait référence. |

## <a name="functions"></a>Fonctions

Le tableau suivant spécifie les équivalences entre les fonctions Azure Monitor et Splunk.

|Splunk | Azure Monitor |Commentaire
|---|---|---
|strcat | strcat()| (1) |
|split  | split() | (1) |
|if     | iff()   | (1) |
|tonumber | todouble()<br>tolong()<br>toint() | (1) |
|upper<br>lower |toupper()<br>tolower()|(1) |
| remplacer | replace() | (1)<br> Notez également que si `replace()` prend trois paramètres dans les deux produits, les paramètres sont différents. |
| substr | substring() | (1)<br>Notez également que Splunk utilise des index de base un. Azure Monitor note des index de base zéro. |
| tolower |  tolower() | (1) |
| toupper | toupper() | (1) |
| match | matches regex |  (2)  |
| regex | matches regex | Dans Splunk, `regex` est un opérateur. Dans Azure Monitor, il s’agit d’un opérateur relationnel. |
| searchmatch | == | Dans Splunk, `searchmatch` permet de rechercher la chaîne exacte.
| random | rand()<br>rand(n) | La fonction de Splunk retourne un nombre compris entre zéro et 2<sup>31</sup>-1. La fonction Azure Monitor retourne un nombre compris entre 0,0 et 1,0 ou, si un paramètre est fourni, entre 0 et n-1.
| now | now() | (1)
| relative_time | totimespan() | (1)<br>Dans Azure Monitor, l’équivalent Splunk de relative_time(datetimeVal, offsetVal) est datetimeVal + totimespan(offsetVal).<br>Par exemple, <code>search &#124; eval n=relative_time(now(), "-1d@d")</code> devient <code>...  &#124; extend myTime = now() - totimespan("1d")</code>.

(1) Dans Splunk, la fonction est appelée avec l’opérateur `eval`. Dans Azure Monitor, elle est utilisée dans le cadre de `extend` ou de `project`.<br>(2) Dans Splunk, la fonction est appelée avec l’opérateur `eval`. Dans Azure Monitor, elle peut être utilisée avec l’opérateur `where`.


## <a name="operators"></a>Opérateurs

Les sections suivantes donnent des exemples d’utilisation de différents opérateurs entre Splunk et Azure Monitor.

> [!NOTE]
> Pour les besoins de l’exemple suivant, le champ Splunk _rule_ est mappé à une table dans Azure Monitor et le timestamp Splunk par défaut est mappé à la colonne _ingestion_time()_ de Log Analytics.

### <a name="search"></a>Recherche
Dans Splunk, vous pouvez omettre le mot clé `search` et spécifier une chaîne sans guillemets. Dans Azure Monitor, chaque requête doit commencer par `find`, une chaîne sans guillemets est un nom de colonne et la valeur de recherche doit être une chaîne entre guillemets. 

| | Opérateur | Exemple |
|:---|:---|:---|
| **Splunk** | **search** | <code>search Session.Id="c8894ffd-e684-43c9-9125-42adc25cd3fc" earliest=-24h</code> |
| **Azure Monitor** | **find** | <code>find Session.Id=="c8894ffd-e684-43c9-9125-42adc25cd3fc" and ingestion_time()> ago(24h)</code> |


### <a name="filter"></a>Filtrer
Les requêtes de journal Azure Monitor commencent à partir d’un jeu de résultats tabulaire où se trouve le filtre. Dans Splunk, le filtrage est l’opération par défaut sur l’index actuel. Vous pouvez également utiliser l’opérateur `where` dans Splunk, mais ce n’est pas recommandé.

| | Opérateur | Exemple |
|:---|:---|:---|
| **Splunk** | **search** | <code>Event.Rule="330009.2" Session.Id="c8894ffd-e684-43c9-9125-42adc25cd3fc" _indextime>-24h</code> |
| **Azure Monitor** | **where** | <code>Office_Hub_OHubBGTaskError<br>&#124; where Session_Id == "c8894ffd-e684-43c9-9125-42adc25cd3fc" and ingestion_time() > ago(24h)</code> |

### <a name="getting-n-eventsrows-for-inspection"></a>Obtention de n événements/lignes pour inspection 
Les requêtes de journal Azure Monitor prennent également en charge `take` comme alias de `limit`. Dans Splunk, si les résultats sont triés, `head` retourne les n premiers résultats. Dans Azure Monitor, la limite n’est pas ordonnée, mais retourne les n premières lignes trouvées.

| | Opérateur | Exemple |
|:---|:---|:---|
| **Splunk** | **head** | <code>Event.Rule=330009.2<br>&#124; head 100</code> |
| **Azure Monitor** | **limit** | <code>Office_Hub_OHubBGTaskError<br>&#124; limit 100</code> |

### <a name="getting-the-first-n-eventsrows-ordered-by-a-fieldcolumn"></a>Obtention des n premiers événements/premières lignes classé(e)s selon un champ/une colonne
Pour les derniers résultats, dans Splunk vous utilisez `tail`. Dans Azure Monitor, la direction de tri est spécifiée avec `asc`.

| | Opérateur | Exemple |
|:---|:---|:---|
| **Splunk** | **head** |  <code>Event.Rule="330009.2"<br>&#124; sort Event.Sequence<br>&#124; head 20</code> |
| **Azure Monitor** | **top** | <code>Office_Hub_OHubBGTaskError<br>&#124; top 20 by Event_Sequence</code> |

### <a name="extending-the-result-set-with-new-fieldscolumns"></a>Extension du jeu de résultats avec de nouveaux champs ou de nouvelles colonnes
Splunk a également une fonction `eval`, qui ne doit pas être comparée à l’opérateur `eval`. Les opérateurs `eval` dans Splunk et `extend` dans Azure Monitor ne prennent en charge que les fonctions scalaires et les opérateurs arithmétiques.

| | Opérateur | Exemple |
|:---|:---|:---|
| **Splunk** | **eval** |  <code>Event.Rule=330009.2<br>&#124; eval state= if(Data.Exception = "0", "success", "error")</code> |
| **Azure Monitor** | **extend** | <code>Office_Hub_OHubBGTaskError<br>&#124; extend state = iif(Data_Exception == 0,"success" ,"error")</code> |

### <a name="rename"></a>Renommer 
Azure Monitor utilise l’opérateur `project-rename` pour renommer un champ. `project-rename` permet à la requête de tirer parti de tous les index prédéfinis pour un champ. Splunk a un opérateur `rename` pour faire cela.

| | Opérateur | Exemple |
|:---|:---|:---|
| **Splunk** | **rename** |  <code>Event.Rule=330009.2<br>&#124; rename Date.Exception as execption</code> |
| **Azure Monitor** | **project-rename** | <code>Office_Hub_OHubBGTaskError<br>&#124; project-rename exception = Date_Exception</code> |

### <a name="format-resultsprojection"></a>Mettre en forme les résultats/Projection
Splunk ne semble pas avoir d’opérateur similaire à `project-away`. Vous pouvez utiliser l’interface utilisateur pour filtrer les champs.

| | Opérateur | Exemple |
|:---|:---|:---|
| **Splunk** | **table** |  <code>Event.Rule=330009.2<br>&#124; table rule, state</code> |
| **Azure Monitor** | **project**<br>**project-away** | <code>Office_Hub_OHubBGTaskError<br>&#124; project exception, state</code> |

### <a name="aggregation"></a>Agrégation
Pour connaître les différentes fonctions d’agrégation, voir [Agrégations dans les requêtes de journal Azure Monitor](/azure/data-explorer/kusto/query/samples?&pivots=azuremonitor#aggregations) .

| | Opérateur | Exemple |
|:---|:---|:---|
| **Splunk** | **stats** |  <code>search (Rule=120502.*)<br>&#124; stats count by OSEnv, Audience</code> |
| **Azure Monitor** | **summarize** | <code>Office_Hub_OHubBGTaskError<br>&#124; summarize count() by App_Platform, Release_Audience</code> |


### <a name="join"></a>Join
La jointure dans Splunk présente des limitations importantes. La sous-requête a une limite de 10 000 résultats (définie dans le fichier de configuration de déploiement), et il existe un nombre limité de variantes de jointure.

| | Opérateur | Exemple |
|:---|:---|:---|
| **Splunk** | **join** |  <code>Event.Rule=120103* &#124; stats by Client.Id, Data.Alias \| join Client.Id max=0 [search earliest=-24h Event.Rule="150310.0" Data.Hresult=-2147221040]</code> |
| **Azure Monitor** | **join** | <code>cluster("OAriaPPT").database("Office PowerPoint").Office_PowerPoint_PPT_Exceptions<br>&#124; where  Data_Hresult== -2147221040<br>&#124; join kind = inner (Office_System_SystemHealthMetadata<br>&#124; summarize by Client_Id, Data_Alias)on Client_Id</code>   |

### <a name="sort"></a>Trier
Dans Splunk, pour trier dans l’ordre croissant, vous devez utiliser l’opérateur `reverse`. Azure Monitor permet également de définir l’emplacement des valeurs Null (au début ou à la fin).

| | Opérateur | Exemple |
|:---|:---|:---|
| **Splunk** | **sort** |  <code>Event.Rule=120103<br>&#124; sort Data.Hresult<br>&#124; reverse</code> |
| **Azure Monitor** | **order by** | <code>Office_Hub_OHubBGTaskError<br>&#124; order by Data_Hresult,  desc</code> |

### <a name="multivalue-expand"></a>Développement à valeurs multiples
Cet opérateur est similaire dans Splunk et dans Azure Monitor.

| | Opérateur | Exemple |
|:---|:---|:---|
| **Splunk** | **mvexpand** |  `mvexpand foo` |
| **Azure Monitor** | **mvexpand** | `mvexpand foo` |

### <a name="results-facets-interesting-fields"></a>Facettes de résultats, champs intéressants
Dans Log Analytics, sur le Portail Azure, seule la première colonne est exposée. Toutes les colonnes sont disponibles via l’API.

| | Opérateur | Exemple |
|:---|:---|:---|
| **Splunk** | **fields** |  <code>Event.Rule=330009.2<br>&#124; fields App.Version, App.Platform</code> |
| **Azure Monitor** | **facets** | <code>Office_Excel_BI_PivotTableCreate<br>&#124; facet by App_Branch, App_Version</code> |

### <a name="de-duplicate"></a>Dédupliquer
Vous pouvez utiliser `summarize arg_min()` à la place pour inverser l’ordre dans lequel l’enregistrement est choisi.

| | Opérateur | Exemple |
|:---|:---|:---|
| **Splunk** | **dedup** |  <code>Event.Rule=330009.2<br>&#124; dedup device_id sortby -batterylife</code> |
| **Azure Monitor** | **summarize arg_max()** | <code>Office_Excel_BI_PivotTableCreate<br>&#124; summarize arg_max(batterylife, *) by device_id</code> |

## <a name="next-steps"></a>Étapes suivantes

- Suivez une leçon [d’écriture de requêtes de journal dans Azure Monitor](get-started-queries.md).
