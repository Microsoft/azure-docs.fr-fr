---
title: Interroger le graphe de jumeaux
titleSuffix: Azure Digital Twins
description: Découvrez comment interroger le graphe de jumeaux Azure Digital Twins.
author: baanders
ms.author: baanders
ms.date: 11/19/2020
ms.topic: how-to
ms.service: digital-twins
ms.custom: contperf-fy21q2
ms.openlocfilehash: df7462cf047dd113c34669d9a5f68f2589cc50f4
ms.sourcegitcommit: d79513b2589a62c52bddd9c7bd0b4d6498805dbe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/18/2020
ms.locfileid: "97672991"
---
# <a name="query-the-azure-digital-twins-twin-graph"></a>Interroger le graphe de jumeaux Azure Digital Twins

Cet article fournit des exemples de requête et des instructions plus détaillées sur l’utilisation du **langage de requête Azure Digital Twins** pour interroger votre [graphe de jumeaux](concepts-twins-graph.md) afin d’obtenir des informations. (Pour une présentation du langage de requête et la liste complète de ses fonctionnalités, consultez [*Concepts : langage de requête*](concepts-query-language.md).)

Cet article commence par des exemples de requête qui illustrent la structure du langage de requête et les opérations de requête courantes pour des jumeaux numériques. Il décrit ensuite comment exécuter vos requêtes une fois que vous les avez écrites, à l’aide de l’[API de requête](/rest/api/digital-twins/dataplane/query) ou d’un [SDK](how-to-use-apis-sdks.md#overview-data-plane-apis) Azure Digital Twins.

> [!TIP]
> Si vous exécutez les exemples de requête ci-dessous avec un appel d’API ou de SDK, vous devez condenser le texte de la requête sur une seule ligne.

## <a name="show-all-digital-twins"></a>Montrer tous les jumeaux numériques

Voici la requête de base qui retourne la liste de tous les jumeaux numériques dans l’instance :

```sql
SELECT *
FROM DIGITALTWINS
```

## <a name="query-by-property"></a>Requête par propriété

Obtenir les jumeaux numériques d’après leurs **propriétés** (y compris l’ID et les métadonnées) :

```sql
SELECT  *
FROM DigitalTwins T  
WHERE T.firmwareVersion = '1.1'
AND T.$dtId in ['123', '456']
AND T.Temperature = 70
```

> [!TIP]
> L’ID d’un jumeau numérique s’interroge à l’aide du champ de métadonnées `$dtId`.

Vous pouvez également obtenir des jumeaux en fonction de **la définition ou non d’une certaine propriété**. Voici une requête qui récupère les jumeaux dont la propriété *Location* a été définie :

```sql
SELECT * FROM DIGITALTWINS WHERE IS_DEFINED(Location)
```

Cela vous permet d’obtenir des jumeaux par le biais de leurs propriétés *tag*, comme décrit dans [Ajouter des étiquettes à des jumeaux numériques](how-to-use-tags.md). Voici une requête qui récupère tous les jumeaux étiquetés avec *red* :

```sql
SELECT * FROM DIGITALTWINS WHERE IS_DEFINED(tags.red)
```

Vous pouvez également obtenir des jumeaux selon le **type d’une propriété**. Voici une requête qui récupère les jumeaux dont la propriété *Temperature* est un nombre :

```sql
SELECT * FROM DIGITALTWINS T WHERE IS_NUMBER(T.Temperature)
```

## <a name="query-by-model"></a>Requête par modèle

L’opérateur `IS_OF_MODEL` peut être utilisé pour filtrer en fonction du [**modèle**](concepts-models.md) de jumeau.

Il prend en compte l’[héritage](concepts-models.md#model-inheritance) et la [gestion des versions](how-to-manage-model.md#update-models) de modèle, et prend la valeur **true** pour un jumeau donné si le jumeau remplit l’une des conditions suivantes :

* Le jumeau implémente directement le modèle fourni à `IS_OF_MODEL()`, et le numéro de version du modèle sur le jumeau est *supérieur ou égal* au numéro de version du modèle fourni.
* Le jumeau implémente un modèle qui *étend* le modèle fourni à `IS_OF_MODEL()`, et le numéro de version du modèle étendu du jumeau est *supérieur ou égal* au numéro de version du modèle fourni.

Par exemple, si vous interrogez les jumeaux du modèle `dtmi:example:widget;4`, la requête retourne tous les jumeaux basés sur la **version 4 ou ultérieure** du modèle **widget**, ainsi que les jumeaux basés sur la version **4 ou ultérieure** de tous les **modèles qui héritent de widget**.

`IS_OF_MODEL` peut prendre plusieurs paramètres différents, et le reste de cette section est dédié à ses différentes options de surcharge.

L’utilisation la plus simple de `IS_OF_MODEL` ne prend qu’un paramètre `twinTypeName` : `IS_OF_MODEL(twinTypeName)`.
Voici un exemple de requête qui transmet une valeur dans ce paramètre :

```sql
SELECT * FROM DIGITALTWINS WHERE IS_OF_MODEL('dtmi:example:thing;1')
```

Pour spécifier une collection de jumeaux à rechercher lorsqu’il en existe plusieurs (par exemple, lorsqu’un `JOIN` est utilisé), ajoutez le paramètre `twinCollection` : `IS_OF_MODEL(twinCollection, twinTypeName)`.
Voici un exemple de requête qui transmet une valeur dans ce paramètre :

```sql
SELECT * FROM DIGITALTWINS DT WHERE IS_OF_MODEL(DT, 'dtmi:example:thing;1')
```

Pour effectuer une correspondance exacte, ajoutez le paramètre `exact` : `IS_OF_MODEL(twinTypeName, exact)`.
Voici un exemple de requête qui transmet une valeur dans ce paramètre :

```sql
SELECT * FROM DIGITALTWINS WHERE IS_OF_MODEL('dtmi:example:thing;1', exact)
```

Vous pouvez également passer les trois arguments ensemble : `IS_OF_MODEL(twinCollection, twinTypeName, exact)`.
Voici un exemple de requête spécifiant une valeur pour les trois paramètres :

```sql
SELECT ROOM FROM DIGITALTWINS DT WHERE IS_OF_MODEL(DT, 'dtmi:example:thing;1', exact)
```

## <a name="query-by-relationship"></a>Requête par relation

Lorsque vous effectuez une interrogation basée sur les **relations** des jumeaux numériques, sachez que le langage du magasin de requêtes d’Azure Digital Twins présente une syntaxe spéciale.

Les relations sont tirées (pull) puis ajoutées à l’étendue de la requête dans la clause `FROM`. Ce qui le distingue principalement des langages de type SQL « classiques » est que chaque expression de la clause `FROM` n’est pas une table. En effet, la clause `FROM` exprime une traversée de relation entre les entités, et elle est écrite avec une version Azure Digital Twins de `JOIN`.

Rappelez-vous qu’avec les fonctionnalités de [modèle](concepts-models.md) d’Azure Digital Twins, les relations n’existent pas indépendamment des jumeaux. Cela signifie que le `JOIN` du langage d’Azure Digital Twins est légèrement différent du `JOIN` SQL général, puisque les relations ne peuvent pas être interrogées de manière indépendante et doivent être liées à un jumeau.
Pour incorporer cette différence, le mot clé `RELATED` est utilisé dans la clause `JOIN` afin de référencer l’ensemble de relations d’un jumeau.

La section suivante fournit plusieurs exemples.

> [!TIP]
> D’un point de vue conceptuel, cette fonctionnalité imite les fonctionnalités orientées document de CosmosDB, où `JOIN` peut être effectué sur les objets enfants d’un document. CosmosDB utilise le mot clé `IN` pour indiquer que `JOIN` est destiné à itérer au sein des éléments de tableau du document de contexte actuel.

### <a name="relationship-based-query-examples"></a>Exemples de requêtes basées sur les relations

Pour obtenir un jeu de données comprenant des relations, utilisez une instruction `FROM` suivie de N instructions `JOIN`, où les instructions `JOIN` expriment les relations basées sur le résultat d’une instruction `FROM` ou `JOIN` précédente.

Voici un exemple de requête basée sur les relations. Cet extrait de code sélectionne tous les jumeaux numériques dont la propriété *ID* a la valeur « ABC », ainsi que tous les jumeaux numériques associés à ces jumeaux via une relation de *contenance*.

```sql
SELECT T, CT
FROM DIGITALTWINS T
JOIN CT RELATED T.contains
WHERE T.$dtId = 'ABC'
```

> [!NOTE]
> Le développeur n’a pas besoin de mettre en corrélation ce `JOIN` avec une valeur de clé dans la clause `WHERE` (ni de spécifier une valeur de clé inline avec la définition `JOIN`). Cette corrélation est calculée automatiquement par le système, puisque les propriétés de relation identifient l’entité cible.

### <a name="query-the-properties-of-a-relationship"></a>Interroger les propriétés d’une relation

De même qu’il est possible de décrire les propriétés des jumeaux numériques via DTDL, les relations peuvent elles aussi avoir des propriétés. Vous pouvez interroger les jumeaux **selon les propriétés de leurs relations**.
Le langage d’Azure Digital Twins permet le filtrage et la projection des relations, en affectant un alias à la relation dans la clause `JOIN`.

Prenons l’exemple d’une relation *servicedBy* qui comprendrait une propriété *reportedCondition*. Dans la requête ci-dessous, cette relation se voit attribuer l’alias « R » afin de référencer sa propriété.

```sql
SELECT T, SBT, R
FROM DIGITALTWINS T
JOIN SBT RELATED T.servicedBy R
WHERE T.$dtId = 'ABC'
AND R.reportedCondition = 'clean'
```

Dans l’exemple ci-dessus, notez que *reportedCondition* est une propriété de la relation *servicedBy* (et non d’un jumeau numérique ayant une relation *servicedBy*).

### <a name="query-with-multiple-joins"></a>Interroger avec plusieurs clauses JOIN

Jusqu’à cinq `JOIN`s sont pris en charge dans une requête unique. Cela vous permet de parcourir plusieurs niveaux de relations à la fois.

Voici un exemple de requête à plusieurs JOIN, qui permet d’obtenir toutes les ampoules contenues dans les panneaux lumineux des pièces 1 et 2.

```sql
SELECT LightBulb
FROM DIGITALTWINS Room
JOIN LightPanel RELATED Room.contains
JOIN LightBulb RELATED LightPanel.contains
WHERE IS_OF_MODEL(LightPanel, 'dtmi:contoso:com:lightpanel;1')
AND IS_OF_MODEL(LightBulb, 'dtmi:contoso:com:lightbulb ;1')
AND Room.$dtId IN ['room1', 'room2']
```

## <a name="count-items"></a>Compter les éléments

Vous pouvez compter le nombre d’éléments dans un jeu de résultats à l’aide de la clause `Select COUNT` :

```sql
SELECT COUNT()
FROM DIGITALTWINS
```

Ajoutez une clause `WHERE` pour compter le nombre d’éléments qui répondent à un certain critère. Voici quelques exemples de comptage avec un filtre appliqué en fonction du type de modèle de jumeau (pour plus d’informations sur cette syntaxe, consultez [*Requête par modèle*](#query-by-model) ci-dessous) :

```sql
SELECT COUNT()
FROM DIGITALTWINS
WHERE IS_OF_MODEL('dtmi:sample:Room;1')

SELECT COUNT()
FROM DIGITALTWINS c
WHERE IS_OF_MODEL('dtmi:sample:Room;1') AND c.Capacity > 20
```

Vous pouvez également utiliser `COUNT` avec la clause `JOIN` . Voici une requête qui compte toutes les ampoules contenues dans les panneaux lumineux des salles 1 et 2 :

```sql
SELECT COUNT()  
FROM DIGITALTWINS Room  
JOIN LightPanel RELATED Room.contains  
JOIN LightBulb RELATED LightPanel.contains  
WHERE IS_OF_MODEL(LightPanel, 'dtmi:contoso:com:lightpanel;1')  
AND IS_OF_MODEL(LightBulb, 'dtmi:contoso:com:lightbulb ;1')  
AND Room.$dtId IN ['room1', 'room2']
```

## <a name="filter-results-select-top-items"></a>Filtrer les résultats : sélectionner les meilleurs éléments

Vous pouvez sélectionner les meilleurs éléments dans une requête à l’aide de la clause `Select TOP`.

```sql
SELECT TOP (5)
FROM DIGITALTWINS
WHERE ...
```

## <a name="filter-results-specify-return-set-with-projections"></a>Filtrer les résultats : spécifier un jeu de retour avec des projections

En utilisant des projections dans l’instruction `SELECT`, vous pouvez choisir les colonnes qui sont renvoyées par une requête.

>[!NOTE]
>À ce stade, les propriétés complexes ne sont pas prises en charge. Pour vous assurer que les propriétés de projection sont valides, combinez les projections avec un contrôle `IS_PRIMITIVE`.

Voici un exemple de requête qui utilise la projection pour retourner les jumeaux et les relations. La requête ci-dessous projette *Consumer*, *Factory* et *Edge* dans un scénario où une *Factory* avec l’ID *ABC* est liée au *Consumer* via une relation *Factory.customer*, et cette relation est présentée en tant que *Edge*.

```sql
SELECT Consumer, Factory, Edge
FROM DIGITALTWINS Factory
JOIN Consumer RELATED Factory.customer Edge
WHERE Factory.$dtId = 'ABC'
```

Vous pouvez également utiliser la projection pour retourner une propriété d’un jumeau. La requête suivante projette la propriété *Name* des *Consumers* associés à la *Factory* avec l’ID *ABC* via une relation de *Factory.customer*.

```sql
SELECT Consumer.name
FROM DIGITALTWINS Factory
JOIN Consumer RELATED Factory.customer Edge
WHERE Factory.$dtId = 'ABC'
AND IS_PRIMITIVE(Consumer.name)
```

Vous pouvez également utiliser la projection pour retourner une propriété d’une relation. Comme dans l’exemple précédent, la requête suivante projette la propriété *Name* des *Consumers* associés à la *Factory* avec un ID *ABC* via une relation de *Factory.customer*. Elle retourne maintenant également deux propriétés de cette relation : *Prop1* et *Prop2*. Pour ce faire, elle nomme la relation *Edge* et collecte ses propriétés.  

```sql
SELECT Consumer.name, Edge.prop1, Edge.prop2, Factory.area
FROM DIGITALTWINS Factory
JOIN Consumer RELATED Factory.customer Edge
WHERE Factory.$dtId = 'ABC'
AND IS_PRIMITIVE(Factory.area) AND IS_PRIMITIVE(Consumer.name) AND IS_PRIMITIVE(Edge.prop1) AND IS_PRIMITIVE(Edge.prop2)
```

Vous pouvez également utiliser des alias pour simplifier les requêtes avec projection.

La requête suivante effectue les mêmes opérations que l’exemple précédent, mais elle associe les noms de propriété à `consumerName`, `first`, `second` et `factoryArea`.

```sql
SELECT Consumer.name AS consumerName, Edge.prop1 AS first, Edge.prop2 AS second, Factory.area AS factoryArea
FROM DIGITALTWINS Factory
JOIN Consumer RELATED Factory.customer Edge
WHERE Factory.$dtId = 'ABC'
AND IS_PRIMITIVE(Factory.area) AND IS_PRIMITIVE(Consumer.name) AND IS_PRIMITIVE(Edge.prop1) AND IS_PRIMITIVE(Edge.prop2)
```

Voici une requête similaire qui interroge le même jeu que ci-dessus, mais projette uniquement la propriété *Consumer.name* en tant que `consumerName` et projette le *Factory* complet en tant que jumeau.

```sql
SELECT Consumer.name AS consumerName, Factory
FROM DIGITALTWINS Factory
JOIN Consumer RELATED Factory.customer Edge
WHERE Factory.$dtId = 'ABC'
AND IS_PRIMITIVE(Factory.area) AND IS_PRIMITIVE(Consumer.name)
```

## <a name="build-efficient-queries-with-the-in-operator"></a>Créer des requêtes efficaces avec l’opérateur IN

Vous pouvez réduire considérablement le nombre de requêtes dont vous avez besoin en générant un tableau de jumeaux et en l’interrogeant à l’aide de l’opérateur `IN`. 

Par exemple, imaginons un scénario dans lequel des *immeubles* comprennent des *étages*, et ces *étages* comprennent des *pièces*. Pour rechercher des salles chauffées dans un immeuble, vous pouvez suivre ces étapes.

1. Recherchez les étages de l’immeuble en fonction de la relation `contains`.

    ```sql
    SELECT Floor
    FROM DIGITALTWINS Building
    JOIN Floor RELATED Building.contains
    WHERE Building.$dtId = @buildingId
    ```

2. Pour trouver des pièces, au lieu de prendre les étages un par un et d’exécuter une requête `JOIN` afin de rechercher les pièces de chacun d’eux, vous pouvez interroger une collection d’étages de l’immeuble (nommée *Floor* dans la requête ci-dessous).

    Dans l’application cliente :
    
    ```csharp
    var floors = "['floor1','floor2', ..'floorn']"; 
    ```
    
    Dans la requête :
    
    ```sql
    
    SELECT Room
    FROM DIGITALTWINS Floor
    JOIN Room RELATED Floor.contains
    WHERE Floor.$dtId IN ['floor1','floor2', ..'floorn']
    AND Room. Temperature > 72
    AND IS_OF_MODEL(Room, 'dtmi:com:contoso:Room;1')
    
    ```

## <a name="other-compound-query-examples"></a>Autres exemples de requêtes composées

Vous pouvez **combiner** un des types de requête ci-dessus avec des opérateurs de combinaison pour inclure plus de détails dans une seule requête. Voici quelques exemples supplémentaires de requêtes composées qui interrogent plusieurs types de descripteur de jumeau à la fois.

| Description | Requête |
| --- | --- |
| Parmi les appareils qui équipent *Room 123* (la salle 123), retournez les appareils MxChip qui remplissent le rôle d’Operator (opérateur) | `SELECT device`<br>`FROM DigitalTwins space`<br>`JOIN device RELATED space.has`<br>`WHERE space.$dtid = 'Room 123'`<br>`AND device.$metadata.model = 'dtmi:contoso:com:DigitalTwins:MxChip:3'`<br>`AND has.role = 'Operator'` |
| Récupérez les jumeaux qui présentent une relation nommée *Contains* (Contient) avec un autre jumeau dont l’ID est *id1* | `SELECT Room`<br>`FROM DIGITALTWINS Room`<br>`JOIN Thermostat RELATED Room.Contains`<br>`WHERE Thermostat.$dtId = 'id1'` |
| Récupérez toutes les salles de ce modèle de salle qui sont contenues dans *floor11* (l’étage11) | `SELECT Room`<br>`FROM DIGITALTWINS Floor`<br>`JOIN Room RELATED Floor.Contains`<br>`WHERE Floor.$dtId = 'floor11'`<br>`AND IS_OF_MODEL(Room, 'dtmi:contoso:com:DigitalTwins:Room;1')` |

## <a name="run-queries-with-the-api"></a>Exécuter des requêtes avec l’API

Une fois que vous avez choisi une chaîne de requête, exécutez-la en appelant l’[**API de requête**](/rest/api/digital-twins/dataplane/query).

Vous pouvez appeler l’API directement ou utiliser l’un des [SDK](how-to-use-apis-sdks.md#overview-data-plane-apis) disponibles pour Azure Digital Twins.

L’extrait de code suivant illustre l’appel au [SDK .NET (C#)](/dotnet/api/overview/azure/digitaltwins/client?view=azure-dotnet&preserve-view=true) à partir de l’application cliente :

```csharp
    string adtInstanceEndpoint = "https://<your-instance-hostname>";

    var credential = new DefaultAzureCredential();
    DigitalTwinsClient client = new DigitalTwinsClient(new Uri(adtInstanceEndpoint), credential);

    // Run a query for all twins   
    string query = "SELECT * FROM DIGITALTWINS";
    AsyncPageable<BasicDigitalTwin> result = client.QueryAsync<BasicDigitalTwin>(query);
```

Cet appel retourne les résultats de la requête sous la forme d’un objet [BasicDigitalTwin](/dotnet/api/azure.digitaltwins.core.basicdigitaltwin?view=azure-dotnet&preserve-view=true).

Les appels de requête prennent en charge la pagination. Voici un exemple complet qui utilise `BasicDigitalTwin` en tant que type de résultat de requête avec gestion des erreurs et pagination :

```csharp
try
{
    await foreach(BasicDigitalTwin twin in result)
        {
            // You can include your own logic to print the result
            // The logic below prints the twin's ID and contents
            Console.WriteLine($"Twin ID: {twin.Id} \nTwin data");
            IDictionary<string, object> contents = twin.Contents;
            foreach (KeyValuePair<string, object> kvp in contents)
            {
                Console.WriteLine($"{kvp.Key}  {kvp.Value}");
            }
        }
}
catch (RequestFailedException e)
{
    Console.WriteLine($"Error {e.Status}: {e.Message}");
    throw;
}
```

## <a name="next-steps"></a>Étapes suivantes

Découvrez les [API et SDK Azure Digital Twins](how-to-use-apis-sdks.md), y compris l’API de requête qui est utilisée pour exécuter les requêtes de cet article.
