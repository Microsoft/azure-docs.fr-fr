---
title: Conformité avec Azure Policy
description: Attribuez des stratégies intégrées dans Azure Policy pour auditer la conformité de vos ressources Azure SignalR Service.
author: JialinXin
ms.service: signalr
ms.topic: conceptual
ms.date: 06/17/2020
ms.author: jixin
ms.openlocfilehash: 018033d3a6123948191a7261f5a1ee2ae526e25a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89295015"
---
# <a name="audit-compliance-of-azure-signalr-service-resources-using-azure-policy"></a>Auditer la conformité de ressources Azure SignalR Service à l’aide d’Azure Policy

[Azure Policy](../governance/policy/overview.md) est un service d’Azure que vous utilisez pour créer, affecter et gérer des stratégies. Ces stratégies appliquent différentes règles et effets sur vos ressources, qui restent donc conformes aux normes et aux contrats de niveau de service de l’entreprise.

Cet article présente les stratégies intégrées (préversion) pour Azure SignalR Service. Utilisez ces stratégies pour auditer la conformité des ressources SignalR nouvelles et existantes.

L’utilisation d’Azure Policy est gratuite.

## <a name="built-in-policy-definitions"></a>Définitions de stratégie intégrées

Les définitions de stratégies intégrées suivantes sont spécifiques à Azure SignalR Service :

[!INCLUDE [azure-policy-reference-policies-signalr](../../includes/policy/reference/bycat/policies-signalr.md)]

## <a name="assign-policy-definitions"></a>Attribuer des définitions de stratégie

* Attribuez des définitions de stratégie à l’aide du [portail Azure](../governance/policy/assign-policy-portal.md), d’[Azure CLI](../governance/policy/assign-policy-azurecli.md), d’un [modèle Resource Manager](../governance/policy/assign-policy-template.md) ou des Kits de développement logiciel (SDK) Azure Policy.
* Étendez une attribution de stratégie à un groupe de ressources, à un abonnement ou à un [groupe d’administration Azure](../governance/management-groups/overview.md). Les attributions de stratégie SignalR s’appliquent aux ressources SignalR nouvelles et existantes au sein de l’étendue.
* Activez ou désactivez l’[application de la stratégie](../governance/policy/concepts/assignment-structure.md#enforcement-mode) à tout moment.

> [!NOTE]
> Une fois que vous avez attribué ou mis à jour une stratégie, l’application de l’attribution aux ressources de l’étendue définie prend un certain temps. Consultez les informations relatives aux [déclencheurs d’évaluation de la stratégie](../governance/policy/how-to/get-compliance-data.md#evaluation-triggers).

## <a name="review-policy-compliance"></a>Vérifier la conformité de la stratégie

Accédez aux informations de conformité générées par vos affectations de stratégie à l’aide du Portail Azure, des outils en ligne de commande Azure ou des Kits de développement logiciel (SDK) Azure Policy. Pour plus d’informations, consultez [Obtenir les données de conformité des ressources Azure](../governance/policy/how-to/get-compliance-data.md).

De nombreuses raisons peuvent expliquer une ressource non conforme. Pour en déterminer la raison ou pour trouver la modification en cause, consultez [Déterminer une non-conformité](../governance/policy/how-to/determine-non-compliance.md).

### <a name="policy-compliance-in-the-portal"></a>Conformité de la stratégie dans le portail :

1. Sélectionnez **Tous les services** et recherchez **Stratégie**.
1. Sélectionnez **Conformité**.
1. Utilisez les filtres pour limiter les états de conformité ou pour rechercher des stratégies.
   
    [ ![Policy compliance in portal](./media/signalr-howto-azure-policy/azure-policy-compliance.png) ](./media/signalr-howto-azure-policy/azure-policy-compliance.png#lightbox)
2. Sélectionnez une stratégie pour passer en revue l’ensemble des détails et des événements relatifs à la conformité. Si vous le souhaitez, sélectionnez une ressource SignalR spécifique pour la conformité des ressources.

### <a name="policy-compliance-in-the-azure-cli"></a>Conformité de la stratégie dans Azure CLI

Vous pouvez également utiliser l’interface de ligne de commande Azure pour accéder aux données de conformité. Par exemple, utilisez la commande [az policy assignment list](/cli/azure/policy/assignment#az-policy-assignment-list) dans l’interface CLI pour obtenir les ID des stratégies Azure SignalR Service qui sont appliquées :

```azurecli
az policy assignment list --query "[?contains(displayName,'SignalR')].{name:displayName, ID:id}" --output table
```

Exemple de sortie :

```
Name                                                                                   ID
-------------------------------------------------------------------------------------  --------------------------------------------------------------------------------------------------------------------------------
[Preview]: Azure SignalR Service should use private links  /subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/providers/Microsoft.Authorization/policyAssignments/<assignmentId>
```

Exécutez ensuite [az policy state list](/cli/azure/policy/state#az-policy-state-list) pour retourner l’état de conformité au format JSON pour toutes les ressources sous un groupe de ressources spécifique :

```azurecli
az policy state list --g <resourceGroup>
```

Ou exécutez [az policy state list](/cli/azure/policy/state#az-policy-state-list) pour retourner l’état de conformité au format JSON d’une ressource SignalR spécifique :

```azurecli
az policy state list \
 --resource /subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/providers/Microsoft.SignalRService/SignalR/<resourceName> \
 --namespace Microsoft.SignalRService \
 --resource-group <resourceGroup>
```

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les [définitions](../governance/policy/concepts/definition-structure.md) et les [effets](../governance/policy/concepts/effects.md) Azure Policy

* Créer une [définition de stratégie personnalisée](../governance/policy/tutorials/create-custom-policy-definition.md)

* En savoir plus sur les [capacités de gouvernance](../governance/index.yml) dans Azure


<!-- LINKS - External -->
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/