---
title: Utiliser de grands jeux de données
description: Découvrez comment obtenir, mettre en forme, paginer et ignorer des enregistrements dans des jeux de données volumineux quand vous utilisez Azure Resource Graph.
ms.date: 09/30/2020
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.openlocfilehash: 59f69738bf9fe25cb739539b7a1f93e4499d781a
ms.sourcegitcommit: e7179fa4708c3af01f9246b5c99ab87a6f0df11c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/30/2020
ms.locfileid: "97826034"
---
# <a name="working-with-large-azure-resource-data-sets"></a>Utilisation de jeux de données volumineux d’Azure Resource Graph

Azure Resource Graph est conçu pour obtenir et utiliser des informations sur des ressources dans votre environnement Azure. Resource Graph permet d’obtenir ces données rapidement, même en cas d’interrogation de milliers d’enregistrements. Resource Graph offre plusieurs options pour utiliser ces jeux de données volumineux.

Pour obtenir des conseils sur l’utilisation des requêtes à une fréquence élevée, voir [Instructions pour les requêtes limitées](./guidance-for-throttled-requests.md).

## <a name="data-set-result-size"></a>Taille du jeu de résultats

Par défaut, Resource Graph limite à **100** le nombre d’enregistrements que retourne toute requête. Ce contrôle protège l’utilisateur et le service contre des requêtes involontaires qui auraient pour effet de retourner des jeux de données volumineux. Cela se produit le plus souvent quand un client expérimente des requêtes pour rechercher et filtrer des ressources d’une manière correspondant à ses besoins particuliers. Ce contrôle diffère de l’utilisation des opérateurs linguistiques [top](/azure/kusto/query/topoperator) ou [limit](/azure/kusto/query/limitoperator) d’Azure Data Explorer pour limiter les résultats.

> [!NOTE]
> Lorsque vous utilisez **First**, il est recommandé d’ordonner les résultats sur au moins une colonne avec `asc` ou `desc`. À défaut de tri, les résultats retournés sont aléatoires et non reproductibles.

La limite par défaut peut être modifiée via toutes les méthodes d’interaction avec Resource Graph. Les exemples suivants montrent comment modifier la limite de taille de jeu de données en la définissant sur _200_ :

```azurecli-interactive
az graph query -q "Resources | project name | order by name asc" --first 200 --output table
```

```azurepowershell-interactive
Search-AzGraph -Query "Resources | project name | order by name asc" -First 200
```

Dans l’[API REST](/rest/api/azureresourcegraph/resourcegraph(2019-04-01)/resources/resources), le contrôle est **$top** et fait partie de **QueryRequestOptions**.

Le contrôle _le plus restrictif_ l’emporte. Par exemple, si votre requête utilise les opérateurs **top** ou **limit** et aurait pour effet de retourner plus d’enregistrements qu’avec l’opérateur **First**, le nombre maximal d’enregistrements retournés est égal à celui obtenu avec l’opérateur **First**. De même, si les opérateurs **top** ou **limit** ont pour effet de retourner un nombre d’enregistrements inférieur à celui que retournerait l’opérateur **First**, le jeu d’enregistrements retourné est celui comprenant le plus petit nombre d’enregistrements retournés par les opérateurs **top** ou **limit**.

L’opérateur **First** a actuellement une valeur maximale autorisée de _5000_, qu’il obtient en [paginant les résultats](#paging-results), à raison de _1000_ enregistrements à la fois.

> [!IMPORTANT]
> Lorsque l’opérateur **First** est configuré sur une valeur supérieure à _1000_ enregistrements, la requête doit **projeter** le champ **id** pour que la pagination fonctionne. Si ce champ ne figure pas dans la requête, la réponse n’est pas [paginée](#paging-results) et les résultats sont limités à _1000_ enregistrements.

## <a name="skipping-records"></a>Omission d’enregistrements

L’option suivante pour la manipulation de jeux de données volumineux est l’utilisation du contrôle **Skip**. Ce contrôle permet à votre requête d’omettre ou d’ignorer le nombre défini d’enregistrements avant de retourner les résultats. Le contrôle **Skip** est utile pour les requêtes qui trient les résultats de façon explicite lorsque l’intention est d’accéder à des enregistrements quelque part au milieu de jeu de résultats. Si les résultats nécessaires figurent à la fin du jeu de données retourné, il est plus efficace d’utiliser une configuration de tri différente afin de récupérer à la place les résultats à partir de la partie supérieure du jeu de données.

> [!NOTE]
> Lorsque vous utilisez **Skip**, il est recommandé d’ordonner les résultats sur au moins une colonne avec `asc` ou `desc`. À défaut de tri, les résultats retournés sont aléatoires et non reproductibles. Si `limit` ou `take` sont utilisés dans la requête, **Skip** (Ignorer) est ignoré.

Les exemples suivants montrent comment ignorer les _10_ premiers enregistrements qu’une requête retournerait, en retournant le jeu de résultats à partir du 11e enregistrement :

```azurecli-interactive
az graph query -q "Resources | project name | order by name asc" --skip 10 --output table
```

```azurepowershell-interactive
Search-AzGraph -Query "Resources | project name | order by name asc" -Skip 10
```

Dans l’[API REST](/rest/api/azureresourcegraph/resourcegraph(2019-04-01)/resources/resources), le contrôle est **$skip** et fait partie de **QueryRequestOptions**.

## <a name="paging-results"></a>Résultats de pagination

S’il est nécessaire de diviser un jeu de résultats en jeux d’enregistrements plus petits à des fins de traitement ou parce que le jeu de résultats dépasse la valeur maximale autorisée de _1000_ enregistrements retournés, utilisez une pagination. Pour indiquer qu’un jeu de résultats a été divisé, l’[API REST](/rest/api/azureresourcegraph/resourcegraph(2019-04-01)/resources/resources)
**QueryResponse** fournit les valeurs **resultTruncated** et **$skipToken**. La valeur **resultTruncated** est une valeur booléenne qui informe l’utilisateur de l’existence d’enregistrements supplémentaires non retournés dans la réponse. Cette condition peut également être identifiée lorsque la valeur de la propriété **count** est inférieure à celle de la propriété **totalRecords**. La valeur **totalRecords** définit le nombre d’enregistrements correspondant à la requête.

 **resultTruncated** a pour valeur **true** quand la pagination est désactivée ou qu’elle est impossible en raison de l’absence de colonne `id` ou de l’insuffisance de ressources pour l’exécution d’une requête. Quand la valeur **resultTruncated** est **true**, la propriété **$skipToken** n’est pas définie.

Les exemples suivants montrent comment **ignorer** (skip) les 3 000 premiers enregistrements et retourner les 1 000 **premiers** (first) enregistrements suivants avec Azure CLI et Azure PowerShell :

```azurecli-interactive
az graph query -q "Resources | project id, name | order by id asc" --first 1000 --skip 3000
```

```azurepowershell-interactive
Search-AzGraph -Query "Resources | project id, name | order by id asc" -First 1000 -Skip 3000
```

> [!IMPORTANT]
> Pour que la pagination fonctionne, la requête doit **projeter** le champ **id**. S’il est absent de la requête, la réponse n’inclut pas le **$skipToken**.

Pour obtenir un exemple, voir [Requête Page suivante](/rest/api/azureresourcegraph/resourcegraph(2019-04-01)/resources/resources#next-page-query) dans la documentation de l’API REST.

## <a name="formatting-results"></a>Mise en forme des résultats

Les résultats d’une requête Resource Graph sont fournis aux formats _Table_  et _ObjectArray_ . Le format est configuré avec le paramètre **resultFormat** dans le cadre des options de requête. Le format _Table_ est la valeur par défaut du paramètre **resultFormat**.

Les résultats de l’interface Azure CLI sont fournis au format JSON par défaut. Les résultats dans Azure PowerShell prennent la forme d’un objet **PSCustomObject** par défaut. Cependant, ils peuvent être convertis rapidement au format JSON à l’aide de l’applet de commande `ConvertTo-Json`. Pour les autres kits SDK, les résultats de la requête peuvent être configurés pour sortir au format _ObjectArray_.

### <a name="format---table"></a>Format - Table

Le format par défaut (_Table_ ) retourne les résultats dans un format JSON conçu pour mettre en évidence la conception des colonnes et les valeurs de lignes des propriétés retournées par la requête. Ce format se rapproche beaucoup de celui d’une table de données structurée ou d’une feuille de calcul avec les colonnes identifiées en premier et chaque ligne représentant les données alignées à ces colonnes.

Voici un exemple de résultat de requête au format _Table_ :

```json
{
    "totalRecords": 47,
    "count": 1,
    "data": {
        "columns": [{
                "name": "name",
                "type": "string"
            },
            {
                "name": "type",
                "type": "string"
            },
            {
                "name": "location",
                "type": "string"
            },
            {
                "name": "subscriptionId",
                "type": "string"
            }
        ],
        "rows": [
            [
                "veryscaryvm2-nsg",
                "microsoft.network/networksecuritygroups",
                "eastus",
                "11111111-1111-1111-1111-111111111111"
            ]
        ]
    },
    "facets": [],
    "resultTruncated": "true"
}
```

### <a name="format---objectarray"></a>Format - ObjectArray

Le format _ObjectArray_ retourne également les résultats au format JSON. Toutefois, cette conception s’aligne sur la relation de paire clé/valeur courante du format JSON, où les données de colonne et de ligne sont mises en correspondance dans des groupes de tableaux.

Voici un exemple de résultat de requête au format _ObjectArray_ :

```json
{
    "totalRecords": 47,
    "count": 1,
    "data": [{
        "name": "veryscaryvm2-nsg",
        "type": "microsoft.network/networksecuritygroups",
        "location": "eastus",
        "subscriptionId": "11111111-1111-1111-1111-111111111111"
    }],
    "facets": [],
    "resultTruncated": "true"
}
```

Voici quelques exemples montrant comment définir le paramètre **resultFormat** pour utiliser le format _ObjectArray_ :

```csharp
var requestOptions = new QueryRequestOptions( resultFormat: ResultFormat.ObjectArray);
var request = new QueryRequest(subscriptions, "Resources | limit 1", options: requestOptions);
```

```python
request_options = QueryRequestOptions(
    result_format=ResultFormat.object_array
)
request = QueryRequest(query="Resources | limit 1", subscriptions=subs_list, options=request_options)
response = client.resources(request)
```

## <a name="next-steps"></a>Étapes suivantes

- Examinez le langage utilisé dans les [requêtes de démarrage](../samples/starter.md).
- Examinez les utilisations avancées dans les [Requêtes avancées](../samples/advanced.md).
- Découvrez plus en détails comment [explorer des ressources](explore-resources.md).
