---
title: Utiliser Azure Policy pour implémenter la gouvernance et les contrôles sur les ressources Azure Cosmos DB
description: Découvrez comment utiliser Azure Policy pour implémenter la gouvernance et les contrôles sur les ressources Azure Cosmos DB.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 09/23/2020
ms.openlocfilehash: 1390f5db6e0f0370788bef60d5a2cafee1e8a96d
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93080649"
---
# <a name="use-azure-policy-to-implement-governance-and-controls-for-azure-cosmos-db-resources"></a>Utiliser Azure Policy pour implémenter la gouvernance et les contrôles sur les ressources Azure Cosmos DB
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

[Azure Policy](../governance/policy/overview.md) permet d’appliquer les normes de gouvernance de l’organisation, d’évaluer la conformité des ressources et d’implémenter la correction automatique. Les cas d’utilisation courants incluent la gestion de la sécurité et des coûts et la cohérence de la configuration.

Azure Policy fournit des définitions de stratégie intégrées. Vous pouvez créer des définitions de stratégie personnalisées pour les scénarios qui ne sont pas traités par les définitions de stratégie intégrées. Pour plus d’informations, consultez la documentation [Azure Policy](../governance/policy/overview.md).

> [!IMPORTANT]
> Azure Policy est appliqué au niveau du fournisseur de ressources pour les services Azure. Les Kits de développement logiciel (SDK) Cosmos DB peuvent effectuer la plupart des opérations de gestion sur les ressources de bases de données, de conteneurs et de débit qui contournent le fournisseur de ressources de Cosmos DB, ignorant ainsi toute stratégie créée à l’aide d’Azure Policy. Pour garantir l’application des stratégies, consultez [Prévention des modifications des Kits de développement logiciel (SDK) Azure Cosmos DB](role-based-access-control.md#prevent-sdk-changes).

## <a name="assign-a-built-in-policy-definition"></a>Affecter une définition de stratégie intégrée

Les définitions de stratégie décrivent les conditions de la conformité des ressources et l’effet à exécuter si une condition est remplie. Les  _affectations_ de stratégie sont créées à partir des _définitions_ de stratégie. Vous pouvez utiliser des définitions de stratégie personnalisées ou intégrées pour vos ressources Azure Cosmos DB. L’étendue des affectations de stratégie se limite à un groupe de gestion Azure, un abonnement Azure ou un groupe de ressources. Elles sont affectées aux ressources au sein de cette étendue. Si vous le souhaitez, vous pouvez exclure des ressources spécifiques de l’étendue.

Vous pouvez créer des attributions de stratégie avec le [Portail Azure](../governance/policy/assign-policy-portal.md), [Azure PowerShell](../governance/policy/assign-policy-powershell.md), [Azure CLI](../governance/policy/assign-policy-azurecli.md) ou [un modèle ARM](../governance/policy/assign-policy-template.md).

Pour créer une affectation de stratégie à partir d’une définition de stratégie intégrée pour Azure Cosmos DB, procédez comme indiqué dans l’article [Démarrage rapide : Créer une affectation de stratégie pour identifier les ressources non conformes](../governance/policy/assign-policy-portal.md).

À l’étape de sélection de la définition de stratégie, entrez `Cosmos DB` dans le champ de recherche pour filtrer la liste des définitions de stratégie intégrées disponibles. Sélectionnez l’une des définitions de stratégie intégrées disponibles, puis choisissez **Sélectionner** pour poursuivre la création de l’attribution de stratégie.

> [!TIP]
> Vous pouvez également utiliser les noms de définition de stratégie intégrée indiqués dans le volet **Définitions disponibles** avec des modèles Azure PowerShell, Azure CLI ou ARM pour créer des attributions de stratégie.

:::image type="content" source="./media/policy/available-definitions.png" alt-text="Recherche de définitions de stratégie intégrée Azure Cosmos DB":::

## <a name="create-a-custom-policy-definition"></a>Créer une définition de stratégie personnalisée

Vous pouvez créer une [définition de stratégie personnalisée](../governance/policy/tutorials/create-custom-policy-definition.md) pour les scénarios qui ne sont pas traités par les définitions de stratégie intégrées. Par la suite, vous créerez une _affectation de stratégie_ à partir de votre _définition_ de stratégie personnalisée.

### <a name="property-types-and-property-aliases-in-policy-rules"></a>Types et alias de propriété dans les règles de stratégie

Utilisez les [étapes du processus de définition de stratégie personnalisée](../governance/policy/tutorials/create-custom-policy-definition.md) pour identifier les propriétés de ressource et les alias de propriété, qui sont nécessaires pour créer des règles de stratégie.

Pour identifier les alias de propriété spécifiques à Azure Cosmos DB, utilisez l’espace de noms `Microsoft.DocumentDB` via l’une des méthodes indiquées dans l’article relatif aux étapes de définition d’une stratégie personnalisée.

#### <a name="use-the-azure-cli"></a>Via Azure CLI :
```azurecli-interactive
# Login first with az login if not using Cloud Shell

# Get Azure Policy aliases for namespace Microsoft.DocumentDB
az provider show --namespace Microsoft.DocumentDB --expand "resourceTypes/aliases" --query "resourceTypes[].aliases[].name"
```

#### <a name="use-azure-powershell"></a>Via Azure PowerShell :
```azurepowershell-interactive
# Login first with Connect-AzAccount if not using Cloud Shell

# Use Get-AzPolicyAlias to list aliases for Microsoft.DocumentDB namespace
(Get-AzPolicyAlias -NamespaceMatch 'Microsoft.DocumentDB').Aliases
```

Ces commandes génèrent la liste des noms d’alias de propriété pour la propriété Azure Cosmos DB. Voici un extrait de la sortie :

```json
[
  "Microsoft.DocumentDB/databaseAccounts/sku.name",
  "Microsoft.DocumentDB/databaseAccounts/virtualNetworkRules[*]",
  "Microsoft.DocumentDB/databaseAccounts/virtualNetworkRules[*].id",
  "Microsoft.DocumentDB/databaseAccounts/isVirtualNetworkFilterEnabled",
  "Microsoft.DocumentDB/databaseAccounts/consistencyPolicy.defaultConsistencyLevel",
  "Microsoft.DocumentDB/databaseAccounts/enableAutomaticFailover",
  "Microsoft.DocumentDB/databaseAccounts/Locations",
  "Microsoft.DocumentDB/databaseAccounts/Locations[*]",
  "Microsoft.DocumentDB/databaseAccounts/Locations[*].locationName",
  "..."
]
```

Vous pouvez utiliser n’importe lequel de ces noms d’alias de propriété dans les [règles de définition de stratégie personnalisée](../governance/policy/tutorials/create-custom-policy-definition.md#policy-rule).

Voici un exemple de définition de stratégie qui vérifie si un compte de Azure Cosmos DB est configuré pour plusieurs emplacements d’écriture. La définition de stratégie personnalisée comprend deux règles : l’une pour vérifier le type spécifique d’alias de propriété, et l’autre pour la propriété spécifique du type, en l’occurrence, le champ qui stocke le paramètre d’emplacement d’écritures multiples. Ces deux règles utilisent des noms d’alias.

```json
"policyRule": {
  "if": {
    "allOf": [
      {
        "field": "type",
        "equals": "Microsoft.DocumentDB/databaseAccounts"
      },
      {
        "field": "Microsoft.DocumentDB/databaseAccounts/enableMultipleWriteLocations",
        "notEquals": true
      }
    ]
  },
  "then": {
    "effect": "Audit"
  }
}
```

Les définitions de stratégie personnalisée peuvent être utilisées pour créer des attributions de stratégie, tout comme les définitions de stratégie intégrées.

## <a name="policy-compliance"></a>Conformité à la stratégie

Une fois les attributions de stratégie créées, Azure Policy évalue les ressources dans l’étendue de l’affectation. Il vérifie la _conformité_ de chaque ressource avec la stratégie. _L’effet_ spécifié dans la stratégie est ensuite appliqué aux ressources non conformes.

Vous pouvez consulter les résultats de conformité et les détails de la correction dans le [Portail Azure](../governance/policy/how-to/get-compliance-data.md#portal) ou via [Azure CLI](../governance/policy/how-to/get-compliance-data.md#command-line) ou encore les [journaux Azure Monitor](../governance/policy/how-to/get-compliance-data.md#azure-monitor-logs).

La capture d’écran suivante montre deux exemples d’affectations de stratégie.

Une affectation est basée sur une définition de stratégie intégrée, qui vérifie que les ressources Azure Cosmos DB sont déployées uniquement dans les régions Azure autorisées. La conformité des ressources affiche le résultat de l’évaluation de la stratégie (conforme ou non conforme) pour les ressources concernées.

L’autre est basée sur une définition de stratégie personnalisée. Cette attribution vérifie que les comptes Cosmos DB sont configurés pour les emplacements d’écriture multiples.

Une fois les affectations de stratégie déployées, le tableau de bord de conformité affiche les résultats de l’évaluation. Notez que cette opération peut prendre jusqu’à 30 minutes après le déploiement d’une affectation de stratégie. En outre, les [analyses d’évaluation de stratégie peuvent être lancées à la demande](../governance/policy/how-to/get-compliance-data.md#on-demand-evaluation-scan) immédiatement après la création d’affectations de stratégie.

La capture d’écran montre les résultats d’évaluation de la conformité suivants pour les comptes Azure Cosmos DB concernés :

- Aucun des deux comptes n’est conforme à une stratégie selon laquelle le filtrage de réseau virtuel doit être configuré.
- Aucun des deux comptes n’est conforme à une stratégie requérant que le compte soit configuré pour plusieurs emplacements d’écriture
- Aucun des deux comptes n’est conforme à une stratégie selon laquelle les ressources ont été déployées vers des régions Azure autorisées.

:::image type="content" source="./media/policy/compliance.png" alt-text="Résultats de conformité pour les affectations d’Azure Policy répertoriées":::

Pour comment corriger les ressources non conformes, consultez [Corriger les ressources non conformes avec Azure Policy](../governance/policy/how-to/remediate-resources.md).

## <a name="next-steps"></a>Étapes suivantes

- [Examinez les exemples de définitions de stratégie personnalisée pour Azure Cosmos DB](https://github.com/Azure/azure-policy/tree/master/samples/CosmosDB), notamment pour les stratégies d’emplacements d’écriture multiples et de filtrage de réseau virtuel présentées ci-dessus.
- [Création d’une affectation de stratégie dans le Portail Azure](../governance/policy/assign-policy-portal.md)
- [Vérification des définitions de stratégie intégrées d’Azure Policy pour Azure Cosmos DB](./policy-reference.md)