---
title: Bien démarrer avec les requêtes de journal dans Azure Monitor | Microsoft Docs
description: Cet article fournit un tutoriel pour commencer à écrire des requêtes de journal dans Azure Monitor.
ms.subservice: logs
ms.topic: tutorial
author: bwren
ms.author: bwren
ms.date: 10/24/2019
ms.openlocfilehash: a949c9b34e299e0dc4eccbb62f4b4ebb38d6ccb9
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96186709"
---
# <a name="get-started-with-log-queries-in-azure-monitor"></a>Bien démarrer avec les requêtes de journal dans Azure Monitor

> [!NOTE]
> Vous pouvez effectuer cet exercice dans votre propre environnement si vous collectez des données à partir d’au moins une machine virtuelle. Si ce n’est pas le cas, utilisez notre [environnement de démonstration](https://ms.portal.azure.com/#blade/Microsoft_Azure_Monitoring_Logs/DemoLogsBlade), qui comporte de nombreux exemples de données.  Si vous savez déjà comment interroger dans KQL, mais que vous devez simplement créer rapidement des requêtes utiles basées sur le ou les types de ressources, consultez le [volet des exemples de requêtes enregistrées](./example-queries.md).

Dans ce didacticiel, vous allez apprendre à écrire des requêtes de journal dans Azure Monitor. Au terme du tutoriel, vous saurez :

- Comprendre la structure d’une requête
- Trier les résultats d’une requête
- Filtrer les résultats d’une requête
- Spécifier un intervalle de temps
- Sélectionner les champs à inclure dans les résultats
- Définir et utiliser des champs personnalisés
- Agréger et regrouper des résultats

Pour obtenir un didacticiel sur l’utilisation de Log Analytics dans le portail Azure, consultez [Bien démarrer avec Azure Monitor Log Analytics](./log-analytics-tutorial.md).<br>
Pour plus d’informations sur les requêtes de journal dans Azure Monitor, consultez [Vue d’ensemble des requêtes de journal dans Azure Monitor](log-query-overview.md).

Suivez une version vidéo de ce tutoriel ci-dessous :

> [!VIDEO https://www.microsoft.com/videoplayer/embed/RE42pGX]

## <a name="writing-a-new-query"></a>Écriture d’une nouvelle requête

Les requêtes peuvent commencer par un nom de table ou la commande *search*. Vous devez commencer par un nom de table, dans la mesure où il définit une étendue précise pour la requête et améliore les performances de la requête et la pertinence des résultats.

> [!NOTE]
> Le langage de requête Kusto utilisé par Azure Monitor est sensible à la casse. Les mots clés du langage sont généralement écrits en minuscules. Quand vous utilisez des noms de tables ou de colonnes dans une requête, veillez à utiliser la casse adéquate, comme indiqué dans le volet Schéma.

### <a name="table-based-queries"></a>Requêtes basées sur une table

Azure Monitor organise les données de journal en tables, chacune composée de plusieurs colonnes. Toutes les tables et colonnes sont affichées dans le volet Schéma de Log Analytics sur le portail Analytics. Identifiez une table qui vous intéresse et jetez un œil à une partie des données :

```Kusto
SecurityEvent
| take 10
```

La requête ci-dessus retourne 10 résultats à partir de la table *SecurityEvent*, sans ordre spécifique. Il s’agit d’une méthode très courante pour avoir un aperçu d’une table et comprendre sa structure et son contenu. Examinons la façon dont elle est construite :

* La requête commence par le nom de table *SecurityEvent* : cette partie définit l’étendue de la requête.
* Le caractère barre verticale (|) sépare les commandes ; ainsi, la sortie de la première commande est l’entrée de la commande suivante. Vous pouvez enchaîner un nombre quelconque d’éléments avec le caractère barre verticale.
* Le caractère barre verticale est suivi de la commande **take**, qui retourne un nombre spécifique d’enregistrements arbitraires à partir de la table.

Nous pourrions exécuter la requête sans ajouter `| take 10` : elle serait toujours valide, mais pourrait retourner jusqu’à 10 000 résultats.

### <a name="search-queries"></a>Requêtes de recherche

Les requêtes de recherche sont moins structurées et généralement plus adaptées pour rechercher des enregistrements qui incluent une valeur spécifique dans une de leurs colonnes :

```Kusto
search in (SecurityEvent) "Cryptographic"
| take 10
```

Cette requête recherche dans la table *SecurityEvent* les enregistrements qui contiennent l’expression « Cryptographic ». De ces enregistrements, dix sont retournés et affichés. Si nous omettons la partie `in (SecurityEvent)` et exécutons simplement `search "Cryptographic"`, la recherche parcourt *toutes* les tables, opération qui peut prendre plus de temps et être moins efficace.

> [!WARNING]
> Les requêtes de recherche sont généralement plus lentes que les requêtes basées sur une table car elles doivent traiter plus de données. 

## <a name="sort-and-top"></a>Sort et top
Bien que **take** soit utile pour obtenir quelques enregistrements, les résultats ne sont pas sélectionnés et affichés dans un ordre particulier. Pour obtenir un affichage ordonné, vous pourriez **trier** (sort) en fonction de la colonne par défaut :

```Kusto
SecurityEvent   
| sort by TimeGenerated desc
```

Cette opération pourrait cependant retourner trop de résultats et également prendre un certain temps. La requête ci-dessus trie *l’intégralité* de la table SecurityEvent en fonction de la colonne TimeGenerated. Ensuite, le portail Analytics limite l’affichage à 10 000 enregistrements uniquement. Bien sûr, cette approche n’est pas optimale.

La meilleure façon d’obtenir uniquement les 10 enregistrements les plus récents consiste à utiliser **top**, qui trie la table entière côté serveur, puis retourne les premiers enregistrements :

```Kusto
SecurityEvent
| top 10 by TimeGenerated
```

L’ordre décroissant étant l’ordre de tri par défaut, nous omettons généralement l’argument **desc**. La sortie doit ressembler à ceci :

![Top 10](media/get-started-queries/top10.png)


## <a name="where-filtering-on-a-condition"></a>Where : filtrage sur une condition
Les filtres, comme leur nom l’indique, filtrent les données en fonction d’une condition spécifique. Il s’agit de la méthode la plus courante pour limiter les résultats d’une requête aux informations pertinentes.

Pour ajouter un filtre à une requête, utilisez l’opérateur **where** suivi d’une ou plusieurs conditions. Par exemple, la requête suivante retourne uniquement des enregistrements *SecurityEvent* dans lesquels _Level_ est égal à _8_ :

```Kusto
SecurityEvent
| where Level == 8
```

Quand vous écrivez des conditions de filtre, vous pouvez utiliser les expressions suivantes :

| Expression | Description | Exemple |
|:---|:---|:---|
| == | Vérifier l’égalité<br>(avec respect de la casse) | `Level == 8` |
| =~ | Vérifier l’égalité<br>(sans respect de la casse) | `EventSourceName =~ "microsoft-windows-security-auditing"` |
| !=, <> | Vérifier l’inégalité<br>(les deux expressions sont identiques) | `Level != 4` |
| *and*, *or* | Requis entre les conditions| `Level == 16 or CommandLine != ""` |

Pour filtrer en fonction de plusieurs conditions, vous pouvez utiliser **and** :

```Kusto
SecurityEvent
| where Level == 8 and EventID == 4672
```

Ou enchaîner plusieurs éléments **where** avec le caractère barre verticale :

```Kusto
SecurityEvent
| where Level == 8 
| where EventID == 4672
```
    
> [!NOTE]
> Les valeurs pouvant avoir différents types, vous devrez peut-être les caster pour effectuer une comparaison sur le type correct. Par exemple, la colonne *Level* de la table SecurityEvent est de type String ; vous devez donc la caster en un type numérique tel que *int* ou *long* avant de pouvoir lui appliquer des opérateurs numériques : `SecurityEvent | where toint(Level) >= 10`

## <a name="specify-a-time-range"></a>Spécifier un intervalle de temps

### <a name="time-picker"></a>Sélecteur d’heure

Le sélecteur d’heure se trouve près du bouton Exécuter et indique que notre recherche ne porte que sur les enregistrements générés au cours des dernières 24 heures. Il s’agit de l’intervalle de temps par défaut appliqué à toutes les requêtes. Pour obtenir uniquement les enregistrements générés au cours de la dernière heure, sélectionnez _Dernière heure_ et réexécutez la requête.

![Sélecteur d’heure](media/get-started-queries/timepicker.png)


### <a name="time-filter-in-query"></a>Filtre de temps dans la requête

Vous pouvez également définir votre propre intervalle de temps en ajoutant un filtre de temps à la requête. Il est préférable de placer le filtre de temps immédiatement après le nom de la table : 

```Kusto
SecurityEvent
| where TimeGenerated > ago(30m) 
| where toint(Level) >= 10
```

Dans le filtre de temps ci-dessus, `ago(30m)` signifie « il y a 30 minutes » ; cette requête ne retourne donc que les enregistrements générés au cours des 30 dernières minutes. Il existe d’autres unités de temps, par exemple les jours (2d), les minutes (25m) et les secondes (10s).


## <a name="project-and-extend-select-and-compute-columns"></a>Project et Extend : sélectionner et calculer des colonnes

Utilisez **project** pour sélectionner les colonnes à inclure dans les résultats :

```Kusto
SecurityEvent 
| top 10 by TimeGenerated 
| project TimeGenerated, Computer, Activity
```

L’exemple précédent génère cette sortie :

![Résultats de la requête project](media/get-started-queries/project.png)

Vous pouvez également utiliser **project** pour renommer des colonnes et en définir de nouvelles. L’exemple suivant utilise project pour effectuer les opérations suivantes :

* Sélectionner uniquement les colonnes d’origine *Computer* et *TimeGenerated*.
* Affiche la colonne *Activity* en tant que *EventDetails*.
* Créer une colonne nommée *EventCode*. La fonction **substring()** est utilisée pour obtenir uniquement les quatre premiers caractères du champ Activity.


```Kusto
SecurityEvent
| top 10 by TimeGenerated 
| project Computer, TimeGenerated, EventDetails=Activity, EventCode=substring(Activity, 0, 4)
```

**extend** conserve toutes les colonnes d’origine dans le jeu de résultats et en définit de nouvelles. La requête suivante utilise **extend** pour ajouter la colonne *EventCode*. Notez que cette colonne peut ne pas s’afficher à la fin des résultats de la table, auquel cas vous devrez développer les détails d’un enregistrement pour l’afficher.

```Kusto
SecurityEvent
| top 10 by TimeGenerated
| extend EventCode=substring(Activity, 0, 4)
```

## <a name="summarize-aggregate-groups-of-rows"></a>Summarize : agréger des groupes de lignes
Utilisez **summarize** pour identifier des groupes d’enregistrements, en fonction d’une ou plusieurs colonnes, et leur appliquer des agrégations. L’utilisation la plus courante de **summarize** est *count*, qui retourne le nombre de résultats contenus dans chaque groupe.

La requête suivante passe en revue tous les enregistrements *Perf* générés au cours de la dernière heure, les regroupe par *ObjectName* et compte les enregistrements dans chaque groupe : 
```Kusto
Perf
| where TimeGenerated > ago(1h)
| summarize count() by ObjectName
```

Parfois, il est judicieux de définir les groupes au moyen de plusieurs dimensions. Chaque combinaison unique de ces valeurs définit un groupe distinct :

```Kusto
Perf
| where TimeGenerated > ago(1h)
| summarize count() by ObjectName, CounterName
```

Une autre utilisation courante consiste à effectuer des calculs mathématiques ou statistiques sur chaque groupe. Par exemple, ce qui suit calcule la moyenne de *CounterValue* pour chaque ordinateur :

```Kusto
Perf
| where TimeGenerated > ago(1h)
| summarize avg(CounterValue) by Computer
```

Malheureusement, les résultats de cette requête n’ont aucune signification dans la mesure où nous avons combiné différents compteurs de performances. Pour que les résultats de cette requête soient plus explicites, nous devons calculer la moyenne séparément pour chaque combinaison de *CounterName* et *Computer* :

```Kusto
Perf
| where TimeGenerated > ago(1h)
| summarize avg(CounterValue) by Computer, CounterName
```

### <a name="summarize-by-a-time-column"></a>Résumer en fonction d’une colonne d’heure
Le regroupement des résultats peut également reposer sur une colonne de temps ou une autre valeur continue. Cependant, une simple agrégation `by TimeGenerated` créerait des groupes pour chaque milliseconde de la plage de temps, car ce sont des valeurs uniques. 

Pour créer des groupes basés sur des valeurs continues, il convient de diviser la plage en unités gérables à l’aide de **bin**. La requête suivante analyse les enregistrements *Perf* qui mesurent la mémoire disponible (*Available MBytes*) sur un ordinateur spécifique. Elle calcule la valeur moyenne pour chaque période de 1 heure, au cours des 7 derniers jours :

```Kusto
Perf 
| where TimeGenerated > ago(7d)
| where Computer == "ContosoAzADDS2" 
| where CounterName == "Available MBytes" 
| summarize avg(CounterValue) by bin(TimeGenerated, 1h)
```

Pour rendre la sortie plus claire, vous décidez de l’afficher sous la forme d’un graphique de temps, qui indique la mémoire disponible au fil du temps :

![Mémoire d'interrogation au fil du temps](media/get-started-queries/chart.png)



## <a name="next-steps"></a>Étapes suivantes

- Apprenez-en davantage sur l’utilisation des données de chaîne dans une requête de journal avec [Utilisation de chaînes dans des requêtes de journal Azure Monitor](/azure/data-explorer/kusto/query/samples?&pivots=azuremonitor#string-operations).
- Apprenez-en davantage sur l’agrégation de données dans une requête de journal avec [Agrégations avancées dans des requêtes de journal Azure Monitor](/azure/data-explorer/write-queries#advanced-aggregations).
- Découvrez comment joindre des données provenant de plusieurs tables avec [Jointures dans les requêtes de journal Azure Monitor](/azure/data-explorer/kusto/query/samples?&pivots=azuremonitor#joins).
- Procurez-vous la documentation sur l’intégralité du langage de requête Kusto dans [Informations de référence sur le langage KQL](/azure/kusto/query/).