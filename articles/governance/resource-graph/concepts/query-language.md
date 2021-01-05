---
title: Comprendre le langage de requête
description: Décrit les tables Resource Graph et les fonctions, opérateurs et types de données Kusto disponibles, utilisables avec Azure Resource Graph.
ms.date: 11/18/2020
ms.topic: conceptual
ms.openlocfilehash: 34aaaa60ed9d757cc1a63ffaebb2225900cff61f
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94966681"
---
# <a name="understanding-the-azure-resource-graph-query-language"></a>Présentation du langage de requête Azure Resource Graph

Le langage de requête pour Azure Resource Graph prend en charge un certain nombre d’opérateurs et de fonctions. Leur fonctionnement et leur utilisation s’appuient sur le [langage de requête Kusto (KQL, Kusto Query Language)](/azure/kusto/query/index). Pour en savoir plus sur le langage de requête utilisé par Resource Graph, commencez par suivre le [tutoriel KQL](/azure/kusto/query/tutorial).

Cet article traite des composants de langage pris en charge par Resource Graph :

- [Tables Resource Graph](#resource-graph-tables)
- [Éléments de langage personnalisés Resource Graph](#resource-graph-custom-language-elements)
- [Éléments du langage KQL pris en charge](#supported-kql-language-elements)
- [Étendue de la requête](#query-scope)
- [Caractères d’échappement](#escape-characters)

## <a name="resource-graph-tables"></a>Tables Resource Graph

Resource Graph fournit plusieurs tables contenant les données qu’il stocke sur les types de ressources Azure Resource Manager et leurs propriétés. Vous pouvez utiliser certaines tables avec l’opérateur `join` ou `union` pour récupérer des propriétés à partir des types de ressources associés. Voici la liste des tables disponibles dans Resource Graph :

|Table Resource Graph |`join` ? |Description |
|---|---|
|Ressources |Oui |Table par défaut si aucune table n’est définie dans la requête. Elle contient la plupart des types de ressources et propriétés Resource Manager. |
|ResourceContainers |Oui |Inclut les données et les types de ressources d’abonnement (en préversion : `Microsoft.Resources/subscriptions`) et de groupe de ressources (`Microsoft.Resources/subscriptions/resourcegroups`). |
|AdvisorResources |Non |Inclut les ressources _associées_ à `Microsoft.Advisor`. |
|AlertsManagementResources |Non |Inclut les ressources _associées_ à `Microsoft.AlertsManagement`. |
|GuestConfigurationResources |Non |Inclut les ressources _associées_ à `Microsoft.GuestConfiguration`. |
|MaintenanceResources |Non |Inclut les ressources _associées_ à `Microsoft.Maintenance`. |
|PolicyResources |Non |Inclut les ressources _associées_ à `Microsoft.PolicyInsights`. (**Préversion**)|
|SecurityResources |Non |Inclut les ressources _associées_ à `Microsoft.Security`. |
|ServiceHealthResources |Non |Inclut les ressources _associées_ à `Microsoft.ResourceHealth`. |

Pour obtenir une liste complète des types de ressources, consultez [Référence : Tables et types de ressources pris en charge](../reference/supported-tables-resources.md).

> [!NOTE]
> La table _Resources_  est la table par défaut. Quand vous interrogez la table _Resources_ , il n’est pas nécessaire d’indiquer le nom de la table, sauf si vous utilisez `join` ou `union`. Toutefois, la pratique recommandée consiste à toujours inclure la table initiale dans la requête.

Utilisez l’Explorateur Resource Graph dans le portail pour découvrir les types de ressources disponibles dans chaque table. Vous pouvez également utiliser une requête telle que `<tableName> | distinct type` pour obtenir la liste des types de ressources existant dans votre environnement, pris en charge par la table Resource Graph concernée.

La requête suivante illustre un opérateur `join` simple. Le résultat de la requête fusionne les colonnes, et le suffixe **1** est ajouté à tous les noms de colonnes dupliqués de la table jointe (_ResourceContainers_ dans cet exemple). Comme la table _ResourceContainers_ possède des types pour les abonnements et les groupes de ressources, l’un ou l’autre de ces types peut être utilisé pour la jointure à la ressource à partir de la table _Resources_.

```kusto
Resources
| join ResourceContainers on subscriptionId
| limit 1
```

La requête suivante illustre une utilisation plus complexe de l’opérateur `join`. La requête limite la table jointe aux ressources d’abonnement et utilise `project` pour inclure uniquement le champ d’origine _subscriptionId_ et le champ _name_ renommé _SubName_. Le renommage de champ évite que `join` ne l’ajoute en tant que _name1_, puisque le champ existe déjà dans _Resources_. La table d’origine est filtrée avec `where` et le `project` suivant comprend des colonnes des deux tables. Le résultat de la requête est un coffre de clés unique affichant le type, le nom du coffre de clés et le nom de l’abonnement dans lequel il se trouve.

```kusto
Resources
| where type == 'microsoft.keyvault/vaults'
| join (ResourceContainers | where type=='microsoft.resources/subscriptions' | project SubName=name, subscriptionId) on subscriptionId
| project type, name, SubName
| limit 1
```

> [!NOTE]
> Quand vous limitez les résultats de `join` avec `project`, la propriété utilisée par `join` pour associer les deux tables (_subscriptionId_  dans l’exemple ci-dessus) doit être incluse dans `project`.

## <a name="extended-properties-preview"></a><a name="extended-properties"></a>Propriétés étendues (préversion)

En tant que fonctionnalité de _préversion_, certains types de ressources dans Resource Graph ont des propriétés supplémentaires liées au type, disponibles pour effectuer une requête au-delà des propriétés fournies par Azure Resource Manager. Cet ensemble de valeurs, connu sous le nom de _propriétés étendues_, existe sur un type de ressource pris en charge dans `properties.extended`. Pour voir quels types de ressources ont des _propriétés étendues_, utilisez la requête suivante :

```kusto
Resources
| where isnotnull(properties.extended)
| distinct type
| order by type asc
```

Exemple : Obtenir le nombre total de machines virtuelles à l’aide de `instanceView.powerState.code` :

```kusto
Resources
| where type == 'microsoft.compute/virtualmachines'
| summarize count() by tostring(properties.extended.instanceView.powerState.code)
```

## <a name="resource-graph-custom-language-elements"></a>Éléments de langage personnalisés Resource Graph

### <a name="shared-query-syntax-preview"></a><a name="shared-query-syntax"></a>Syntaxe des requêtes partagées (préversion)

La fonctionnalité d’évaluation [requête partagée](../tutorials/create-share-query.md) est accessible directement dans une requête Resource Graph. Ce scénario permet de créer des requêtes standard comme requêtes partagées et de les réutiliser. Pour appeler une requête partagée à l’intérieur d’une requête Resource Graph, utilisez la syntaxe `{{shared-query-uri}}`. L’URI de la requête partagée correspond à son _ID de ressource_ sur la page **Paramètres** correspondante. Dans cet exemple, l’URI de la requête partagée est `/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/SharedQueries/providers/Microsoft.ResourceGraph/queries/Count VMs by OS`.
Il pointe vers l’abonnement, le groupe de ressources et le nom complet de la requête partagée à laquelle nous souhaitons faire référence dans une autre requête. Cette requête est identique à celle créée dans [Tutoriel : Création et partage d’une requête](../tutorials/create-share-query.md).

> [!NOTE]
> Il n’est pas possible d’enregistrer comme requête partagée une requête qui fait référence à une requête partagée.

Exemple 1 : Utiliser uniquement la requête partagée

Les résultats de cette requête Resource Graph sont les mêmes que ceux de la requête stockée dans la requête partagée.

```kusto
{{/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/SharedQueries/providers/Microsoft.ResourceGraph/queries/Count VMs by OS}}
```

Exemple 2 : Inclure la requête partagée dans une requête plus large

Cette requête utilise d’abord la requête partagée, puis se sert de `limit` pour limiter encore les résultats.

```kusto
{{/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/SharedQueries/providers/Microsoft.ResourceGraph/queries/Count VMs by OS}}
| where properties_storageProfile_osDisk_osType =~ 'Windows'
```

## <a name="supported-kql-language-elements"></a>Éléments du langage KQL pris en charge

Resource Graph prend en charge un sous-ensemble des [types de données](/azure/kusto/query/scalar-data-types/), [fonctions scalaires](/azure/kusto/query/scalarfunctions), [opérateurs scalaires](/azure/kusto/query/binoperators) et [fonctions d’agrégation](/azure/kusto/query/any-aggfunction) du langage KQL. Des [opérateurs tabulaires](/azure/kusto/query/queries) spécifiques sont pris en charge par Resource Graph, dont certains présentent différents comportements.

### <a name="supported-tabulartop-level-operators"></a>Opérateurs tabulaires/de niveau supérieur pris en charge

Voici la liste des opérateurs tabulaires KQL pris en charge par Resource Graph avec des exemples spécifiques :

|KQL |Exemple de requête Resource Graph |Notes |
|---|---|---|
|[count](/azure/kusto/query/countoperator) |[Compter les coffres de clés](../samples/starter.md#count-keyvaults) | |
|[distinct](/azure/kusto/query/distinctoperator) |[Afficher des valeurs distinctes pour un alias spécifique](../samples/starter.md#distinct-alias-values) | |
|[extend](/azure/kusto/query/extendoperator) |[Compter les machines virtuelles par type de système d’exploitation](../samples/starter.md#count-os) | |
|[join](/azure/kusto/query/joinoperator) |[Coffre de clés avec nom d’abonnement](../samples/advanced.md#join) |Variantes de jointure prises en charge : [innerunique](/azure/kusto/query/joinoperator#default-join-flavor), [inner](/azure/kusto/query/joinoperator#inner-join), [leftouter](/azure/kusto/query/joinoperator#left-outer-join). Limite de 3 `join` dans une requête unique. Les stratégies de jointure personnalisées comme la jointure de diffusion ne sont pas autorisées. Pour savoir quelles tables peuvent utiliser `join`, consultez [Tables Resource Graph](#resource-graph-tables). |
|[limit](/azure/kusto/query/limitoperator) |[Lister toutes les adresses IP publiques](../samples/starter.md#list-publicip) |Identique à `take`. Ne fonctionne pas avec [Ignorer](./work-with-data.md#skipping-records). |
|[mvexpand](/azure/kusto/query/mvexpandoperator) | | Opérateur hérité, utilisez `mv-expand` à la place. Valeur _RowLimit_ maximale de 400. La valeur par défaut est 128. |
|[mv-expand](/azure/kusto/query/mvexpandoperator) |[Lister Cosmos DB avec des emplacements d’écriture spécifiques](../samples/advanced.md#mvexpand-cosmosdb) |Valeur _RowLimit_ maximale de 400. La valeur par défaut est 128. |
|[order](/azure/kusto/query/orderoperator) |[Lister les ressources triées par nom](../samples/starter.md#list-resources) |Identique à `sort` |
|[project](/azure/kusto/query/projectoperator) |[Lister les ressources triées par nom](../samples/starter.md#list-resources) | |
|[project-away](/azure/kusto/query/projectawayoperator) |[Supprimer des colonnes des résultats](../samples/advanced.md#remove-column) | |
|[sort](/azure/kusto/query/sortoperator) |[Lister les ressources triées par nom](../samples/starter.md#list-resources) |Identique à `order` |
|[summarize](/azure/kusto/query/summarizeoperator) |[Compter les ressources Azure](../samples/starter.md#count-resources) |Première page simplifiée uniquement |
|[take](/azure/kusto/query/takeoperator) |[Lister toutes les adresses IP publiques](../samples/starter.md#list-publicip) |Identique à `limit`. Ne fonctionne pas avec [Ignorer](./work-with-data.md#skipping-records). |
|[top](/azure/kusto/query/topoperator) |[Afficher les cinq premières machines virtuelles par nom et leur type de système d’exploitation](../samples/starter.md#show-sorted) | |
|[union](/azure/kusto/query/unionoperator) |[Combiner les résultats de deux requêtes en un résultat unique](../samples/advanced.md#unionresults) |Table unique autorisée : _T_ `| union` \[`kind=` `inner`\|`outer`\] \[`withsource=`_ColumnName_\] _Table_. Limite de 3 sections `union` dans une requête unique. La résolution approximative des tables avec sections `union` n’est pas autorisée. Peut être utilisé dans une table unique ou entre les tables _Resources_ et _ResourceContainers_. |
|[where](/azure/kusto/query/whereoperator) |[Afficher les ressources contenant storage](../samples/starter.md#show-storage) | |

## <a name="query-scope"></a>Étendue de requête

L’étendue des abonnements à partir desquels les ressources sont retournées par une requête dépend de la méthode d’accès à Azure Resource Graph. Azure CLI et Azure PowerShell remplissent la liste des abonnements à inclure dans la demande en fonction du contexte de l’utilisateur autorisé. La liste des abonnements peut être définie manuellement pour chacun respectivement avec des **abonnements** des **paramètres d’abonnement**.
Dans l’API REST et tous les autres kits de développement logiciel (SDK), la liste des abonnements dont inclure les ressources doit être définie explicitement dans le cadre de la demande.

En guide d’**aperçu**, la version de l’API REST `2020-04-01-preview` ajoute une propriété pour étendre la requête à un [groupe d’administration](../../management-groups/overview.md). Cette API d’aperçu rend également la propriété d’abonnement facultative. Si un groupe d’administration ou une liste d’abonnements ne sont pas définis, l’étendue de la requête correspond à l’ensemble des ressources, qui incluent les ressources [Azure Lighthouse](../../../lighthouse/concepts/azure-delegated-resource-management.md) déléguées, auxquelles l’utilisateur authentifié peut accéder. La nouvelle propriété `managementGroupId` prend l’ID du groupe d’administration, qui est différent du nom du groupe d’administration. Quand `managementGroupId` est spécifié, les ressources des 5 000 premiers abonnements dans ou sous la hiérarchie du groupe d’administration spécifié sont incluses. `managementGroupId` ne peut pas être utilisé en même temps que `subscriptions`.

Exemple : Interroger toutes les ressources dans la hiérarchie du groupe d’administration nommé « Mon groupe d’administration » avec l’ID « myMG ».

- URI de l’API REST

  ```http
  POST https://management.azure.com/providers/Microsoft.ResourceGraph/resources?api-version=2020-04-01-preview
  ```

- Corps de la requête

  ```json
  {
      "query": "Resources | summarize count()",
      "managementGroupId": "myMG"
  }
  ```

## <a name="escape-characters"></a>Caractères d'échappement

Certains noms propres, tels que ceux qui incluent un `.` ou `$`, doivent être inclus dans un wrapper ou échappés dans la requête, sinon le nom de la propriété est interprété de façon incorrecte et les résultats attendus ne sont pas obtenus.

- `.`- Encapsulez le nom de la propriété de cette façon : `['propertyname.withaperiod']`
  
  Exemple de requête qui encapsule la propriété _odata.type_ :

  ```kusto
  where type=~'Microsoft.Insights/alertRules' | project name, properties.condition.['odata.type']
  ```

- `$` - Échappez le caractère dans le nom de la propriété. Le caractère d’échappement utilisé dépend de l’interpréteur de commandes Resource Graph sur lequel il est exécuté.

  - **bash** - `\`

    Exemple de requête qui échappe la propriété _\$type_ dans bash :

    ```kusto
    where type=~'Microsoft.Insights/alertRules' | project name, properties.condition.\$type
    ```

  - **cmd** - ne pas échapper le caractère `$`.

  - **PowerShell** - ``` ` ```

    Exemple de requête qui échappe la propriété _\$type_ dans PowerShell :

    ```kusto
    where type=~'Microsoft.Insights/alertRules' | project name, properties.condition.`$type
    ```

## <a name="next-steps"></a>Étapes suivantes

- Examinez le langage utilisé dans les [requêtes de démarrage](../samples/starter.md).
- Examinez les utilisations avancées dans les [Requêtes avancées](../samples/advanced.md).
- Découvrez plus en détails comment [explorer des ressources](explore-resources.md).
