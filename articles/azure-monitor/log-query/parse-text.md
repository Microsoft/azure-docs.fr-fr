---
title: Analyser des données de texte dans les journaux d’activité Azure Monitor | Microsoft Docs
description: Décrit différentes options pour l'analyse des données de journal dans les enregistrements Azure Monitor lorsque les données sont ingérées et lorsqu'elles sont extraites par une requête, en comparant les avantages de chacune.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 12/04/2018
ms.openlocfilehash: c76cf939959d198d2c28181295a695a65ae46af9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86505701"
---
# <a name="parse-text-data-in-azure-monitor-logs"></a>Analyser des données de texte dans les journaux d’activité Azure Monitor
Certaines données de journal collectées par Azure Monitor incluront plusieurs informations au sein d'une même propriété. L’analyse de ces données dans plusieurs propriétés simplifie leur utilisation dans des requêtes. Un exemple courant est le [journal personnalisé](../platform/data-sources-custom-logs.md) qui collecte une entrée de journal entière avec plusieurs valeurs dans une seule propriété. En créant des propriétés distinctes pour les différentes valeurs, vous pouvez faire des recherches et des agrégations sur chacune d’elles.

Cet article décrit différentes options pour l'analyse des données de journal dans Azure Monitor lorsque les données sont ingérées et lorsqu'elles sont extraites par une requête, en comparant les avantages de chacune.


## <a name="parsing-methods"></a>Méthode d’analyse
Vous pouvez analyser les données au moment de l’ingestion (lorsqu’elles sont collectées) ou bien lors de la requête (lorsqu’elles sont analysées avec une requête). Chaque stratégie présente ses propres avantages, comme décrit ci-dessous.

### <a name="parse-data-at-collection-time"></a>Analyser les données au moment de la collection
Lorsque vous analysez des données au moment de la collection, vous configurez des [champs personnalisés](../platform/custom-fields.md) qui créent de nouvelles propriétés au sein de la table. Les requêtes n’ont pas nécessairement besoin d’inclure une logique d’analyse et elles utilisent simplement ces propriétés comme n’importe quel autre champ de la table.

Les avantages de cette méthode sont les suivants :

- Elle facilite l’interrogation des données collectées, dans la mesure où vous n’avez pas besoin d’inclure des commandes d’analyse dans la requête.
- Elle offre de meilleures performances de requête, étant donné que la requête n’a pas besoin d’effectuer l’analyse.
 
Les inconvénients de cette méthode sont les suivants :

- Elle doit être définie à l’avance. Elle ne peut pas inclure de données déjà collectées.
- Une modification de la logique d’analyse s’applique uniquement aux nouvelles données.
- Il y a moins d’options d’analyse disponibles avec les requêtes.
- Elle augmente le temps de latence pour la collecte des données.
- Les erreurs peuvent être difficiles à traiter.


### <a name="parse-data-at-query-time"></a>Analyser les données au moment de la requête
Lorsque vous analysez des données au moment de la requête, vous incluez une logique dans votre requête pour analyser les données dans plusieurs champs. La table elle-même n’est pas modifiée.

Les avantages de cette méthode sont les suivants :

- Elle s’applique à toutes les données, y compris celles déjà collectées.
- Les modifications de la logique peuvent être immédiatement appliquées à toutes les données.
- Les options d’analyse sont flexibles, incluant une logique prédéfinie pour les structures de données particulières.
 
Les inconvénients de cette méthode sont les suivants :

- Elle nécessite des requêtes plus complexes. L’utilisation de [fonctions simulant une table](#use-function-to-simulate-a-table) permet d’atténuer cet inconvénient.
- Elle nécessite de répliquer une logique d’analyse dans plusieurs requêtes. Elle peut partager une logique via des fonctions.
- Elle peut créer une surcharge lors de l’exécution d’une logique complexe sur de très grands jeux d’enregistrement (comprenant des milliards d’enregistrements).

## <a name="parse-data-as-its-collected"></a>Analyser les données lors de la collecte
Consultez [Créer des champs personnalisés dans Azure Monitor](../platform/custom-fields.md) pour plus d'informations sur l'analyse des données lors de la collecte. Cela crée des propriétés personnalisées dans la table, pouvant être utilisées par les requêtes comme n’importe quelle autre propriété.

## <a name="parse-data-in-query-using-patterns"></a>Analyser les données dans une requête à l’aide de modèles
Lorsque les données que vous souhaitez analyser peuvent être identifiées par un modèle répété sur plusieurs enregistrements, vous pouvez utiliser différents opérateurs du [langage de requête Kusto](/azure/kusto/query/) pour extraire la donnée spécifique dans une ou plusieurs nouvelles propriétés.

### <a name="simple-text-patterns"></a>Modèles de texte simples

Utilisez l’opérateur [d’analyse](/azure/kusto/query/parseoperator) dans votre requête pour créer une ou plusieurs propriétés personnalisées pouvant être extraites à partir d’une expression de chaîne. Vous spécifiez le modèle à identifier et le nom des propriétés à créer. Cela est particulièrement utile pour les données comprenant des chaînes de clé-valeur avec une forme similaire à _clé=valeur_.

Imaginons un journal personnalisé avec des données au format suivant.

```
Time=2018-03-10 01:34:36 Event Code=207 Status=Success Message=Client 05a26a97-272a-4bc9-8f64-269d154b0e39 connected
Time=2018-03-10 01:33:33 Event Code=208 Status=Warning Message=Client ec53d95c-1c88-41ae-8174-92104212de5d disconnected
Time=2018-03-10 01:35:44 Event Code=209 Status=Success Message=Transaction 10d65890-b003-48f8-9cfc-9c74b51189c8 succeeded
Time=2018-03-10 01:38:22 Event Code=302 Status=Error Message=Application could not connect to database
Time=2018-03-10 01:31:34 Event Code=303 Status=Error Message=Application lost connection to database
```

La requête suivante analyse ces données dans des propriétés individuelles. La ligne avec _project_ est ajoutée afin de retourner uniquement les propriétés calculées et pas _RawData_, qui est la seule propriété contenant l’entrée complète à partir du journal personnalisé.

```Kusto
MyCustomLog_CL
| parse RawData with * "Time=" EventTime " Event Code=" Code " Status=" Status " Message=" Message
| project EventTime, Code, Status, Message
```

Voici un autre exemple qui découpe le nom d’utilisateur d’un UPN dans la table _AzureActivity_.

```Kusto
AzureActivity
| parse  Caller with UPNUserPart "@" * 
| where UPNUserPart != "" //Remove non UPN callers (apps, SPNs, etc)
| distinct UPNUserPart, Caller
```


### <a name="regular-expressions"></a>Expressions régulières
Si vos données peuvent être identifiées avec une expression régulière, vous pouvez utiliser des [fonctions utilisant des expressions régulières](/azure/kusto/query/re2) pour extraire les valeurs individuelles. L’exemple suivant utilise [extract](/azure/kusto/query/extractfunction) pour diviser le champ _UPN_ à partir des enregistrements _AzureActivity_, puis retourne des utilisateurs distincts.

```Kusto
AzureActivity
| extend UPNUserPart = extract("([a-z.]*)@", 1, Caller) 
| distinct UPNUserPart, Caller
```

Pour permettre une analyse efficace à grande échelle, Azure Monitor utilise la version re2 des Expressions régulières, qui est similaire mais pas identique à d'autres variantes d'expressions régulières. Reportez-vous à la [syntaxe d’expression re2](https://aka.ms/kql_re2syntax) pour plus d’informations.


## <a name="parse-delimited-data-in-a-query"></a>Analyser des données délimitées dans une requête
Des données délimitées séparent les champs avec un caractère commun, comme une virgule dans un fichier CSV. Utilisez la fonction [split](/azure/kusto/query/splitfunction) (fractionner) pour analyser des données délimitées à l’aide d’un délimiteur que vous spécifiez. Vous pouvez l’utiliser avec l’opérateur [extend](/azure/kusto/query/extendoperator) (étendre) pour retourner tous les champs dans les données ou pour spécifier des champs individuels à inclure dans la sortie.

> [!NOTE]
> Étant donné que le fractionnement retourne un objet dynamique, il est possible que les résultats doivent être explicitement castés en types de données comme une chaîne à utiliser dans les opérateurs et les filtres.

Imaginons un journal personnalisé avec des données au format CSV suivant.

```
2018-03-10 01:34:36, 207,Success,Client 05a26a97-272a-4bc9-8f64-269d154b0e39 connected
2018-03-10 01:33:33, 208,Warning,Client ec53d95c-1c88-41ae-8174-92104212de5d disconnected
2018-03-10 01:35:44, 209,Success,Transaction 10d65890-b003-48f8-9cfc-9c74b51189c8 succeeded
2018-03-10 01:38:22, 302,Error,Application could not connect to database
2018-03-10 01:31:34, 303,Error,Application lost connection to database
```

La requête suivante analyse ces données et les synthétise par deux des propriétés calculées. La première ligne fractionne la propriété _RawData_ dans un groupe de chaînes. Chacune des lignes suivantes donne un nom aux propriétés individuelles et les ajoute à la sortie en utilisant des fonctions pour les convertir en type de données approprié.

```Kusto
MyCustomCSVLog_CL
| extend CSVFields  = split(RawData, ',')
| extend EventTime  = todatetime(CSVFields[0])
| extend Code       = toint(CSVFields[1]) 
| extend Status     = tostring(CSVFields[2]) 
| extend Message    = tostring(CSVFields[3]) 
| where getyear(EventTime) == 2018
| summarize count() by Status,Code
```

## <a name="parse-predefined-structures-in-a-query"></a>Analyser des structures prédéfinies dans une requête
Si vos données sont formatées dans une structure connue, vous pourrez peut-être utiliser l'une des fonctions du [langage de requête Kusto](/azure/kusto/query/) pour analyser des structures prédéfinies :

- [JSON](/azure/kusto/query/parsejsonfunction)
- [XML](/azure/kusto/query/parse-xmlfunction)
- [IPv4](/azure/kusto/query/parse-ipv4function)
- [URL](/azure/kusto/query/parseurlfunction)
- [Requête d’URL](/azure/kusto/query/parseurlqueryfunction)
- [Chemin de fichier](/azure/kusto/query/parsepathfunction)
- [Agent utilisateur](/azure/kusto/query/parse-useragentfunction)
- [Chaîne de version](/azure/kusto/query/parse-versionfunction)

L’exemple de requête suivant analyse le champ _Propriétés_ de la table _AzureActivity_, structurée en JSON. Il enregistre les résultats dans une propriété dynamique appelée _parsedProp_, qui inclut la valeur nommée individuellement en JSON. Ces valeurs sont utilisées pour filtrer et résumer les résultats de requête.

```Kusto
AzureActivity
| extend parsedProp = parse_json(Properties) 
| where parsedProp.isComplianceCheck == "True" 
| summarize count() by ResourceGroup, tostring(parsedProp.tags.businessowner)
```

Ces fonctions d’analyse sont susceptibles d’utiliser le processeur de façon intensive, elles doivent donc être utilisées uniquement lorsque votre requête utilise plusieurs propriétés à partir de données formatées. Autrement, le traitement simple des critères spéciaux sera plus rapide.

L’exemple suivant montre la répartition du type de préauthentification TGT de contrôleur de domaine. Le type existe uniquement dans le champ EventData (une chaîne XML), mais aucune autre donnée venant de ce champ n’est nécessaire. Dans ce cas, [parse](/azure/kusto/query/parseoperator) est utilisé pour extraire les données indispensables.

```Kusto
SecurityEvent
| where EventID == 4768
| parse EventData with * 'PreAuthType">' PreAuthType '</Data>' * 
| summarize count() by PreAuthType
```

## <a name="use-function-to-simulate-a-table"></a>Utiliser une fonction pour simuler une table
Vous pouvez avoir plusieurs requêtes effectuant la même analyse d’une table particulière. Dans ce cas, [créez une fonction](functions.md) qui retourne les données analysées au lieu de répliquer la logique d’analyse dans chaque requête. Vous pouvez ensuite utiliser l’alias de fonction à la place de la table d’origine dans les autres requêtes.

Prenons l’exemple de journal personnalisé délimité par des virgules. Pour pouvoir utiliser les données analysées dans plusieurs requêtes, créez une fonction à l’aide de la requête suivante et enregistrez-la avec l’alias _MyCustomCSVLog_.

```Kusto
MyCustomCSVLog_CL
| extend CSVFields = split(RawData, ',')
| extend DateTime  = tostring(CSVFields[0])
| extend Code      = toint(CSVFields[1]) 
| extend Status    = tostring(CSVFields[2]) 
| extend Message   = tostring(CSVFields[3]) 
```

Vous pouvez maintenant utiliser l’alias _MyCustomCSVLog_ à la place du nom de la table réelle dans les requêtes, comme suit.

```Kusto
MyCustomCSVLog
| summarize count() by Status,Code
```


## <a name="next-steps"></a>Étapes suivantes
* Découvrez les [requêtes dans les journaux](log-query-overview.md) pour analyser les données collectées à partir de sources de données et de solutions.
