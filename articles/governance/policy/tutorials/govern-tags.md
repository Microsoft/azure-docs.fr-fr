---
title: 'Tutoriel : Gérer la gouvernance des balises'
description: Dans ce tutoriel, vous utilisez l’effet modify d’Azure Policy pour créer et appliquer un modèle de gouvernance des balises sur des ressources nouvelles ou existantes.
ms.date: 10/05/2020
ms.topic: tutorial
ms.openlocfilehash: 9efeb27151cd3a32741f1bdb6d1d90d3304c5874
ms.sourcegitcommit: fbb620e0c47f49a8cf0a568ba704edefd0e30f81
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91876275"
---
# <a name="tutorial-manage-tag-governance-with-azure-policy"></a>Tutoriel : Gérer la gouvernance des balises avec Azure Policy

Les [balises](../../../azure-resource-manager/management/tag-resources.md) représentent un aspect essentiel de l’organisation des ressources Azure dans une taxonomie. Dès lors que les [meilleures pratiques de gestion des balises](/azure/cloud-adoption-framework/ready/azure-best-practices/naming-and-tagging#naming-and-tagging-resources) sont suivies, les balises peuvent servir de base à l’application des stratégies d’entreprise avec Azure Policy ou au [suivi des coûts avec Cost Management](../../../cost-management-billing/costs/cost-mgt-best-practices.md#tag-shared-resources).
Quels que soient l’usage et la finalité des balises utilisées, il est important de pouvoir en ajouter, en modifier et en supprimer rapidement sur des ressources Azure. Pour voir si votre ressource Azure prend en charge l’étiquetage, consultez [Prise en charge des étiquettes](../../../azure-resource-manager/management/tag-support.md).

L’effet [modify](../concepts/effects.md#modify) d’Azure Policy est conçu pour faciliter la gouvernance des balises à toutes les phases de gouvernance des ressources. **modify** est utile dans les cas suivants :

- Vous ne connaissez pas encore le cloud et n’avez pas de gouvernance des balises.
- Vous possédez déjà des milliers de ressources sans gouvernance des balises.
- Vous disposez déjà d’une taxonomie que vous devez modifier.

Dans ce didacticiel, vous allez apprendre à effectuer les tâches suivantes :

> [!div class="checklist"]
> - Identifier vos exigences métier
> - Associer chaque spécification à une définition de stratégie
> - Regrouper les stratégies de balises dans une initiative

## <a name="prerequisites"></a>Prérequis

Pour suivre ce tutoriel, vous devez disposer d’un abonnement Azure. Si vous n’en avez pas, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="identify-requirements"></a>Identifier les exigences

Comme pour toute bonne implémentation des contrôles de gouvernance, les exigences doivent venir des besoins professionnels et être bien comprises avant la création de contrôles techniques. Dans le scénario de ce tutoriel, nos exigences métier seront les suivantes :

- Deux balises obligatoires sur toutes les ressources : _CostCenter_ et _Env_
- _CostCenter_ doit apparaître sur tous les conteneurs et toutes les ressources individuelles
  - Les ressources héritent du conteneur dans lequel elles se trouvent, mais peuvent être remplacées individuellement
- _Env_ doit apparaître sur tous les conteneurs et toutes les ressources individuelles
  - Les ressources déterminent l’environnement par le schéma de nommage du conteneur et ne peuvent pas être remplacées
  - Toutes les ressources d’un conteneur font partie du même environnement

## <a name="configure-the-costcenter-tag"></a>Configurer la balise CostCenter

Dans un environnement Azure géré par Azure Policy en particulier, les spécifications d’étiquette _CostCenter_ exigent les mesures suivantes :

- Refuser les groupes de ressources qui ne contiennent pas la balise _CostCenter_
- Modifier les ressources qui ne contiennent pas la balise _CostCenter_ en l’ajoutant à partir du groupe de ressources parent

### <a name="deny-resource-groups-missing-the-costcenter-tag"></a>Refuser les groupes de ressources qui ne contiennent pas la balise CostCenter

Dans la mesure où la balise _CostCenter_ d’un groupe de ressources ne peut pas être déterminée par le nom du groupe, elle doit être définie dans la demande de création du groupe de ressources. La règle de stratégie suivante, qui utilise l’effet [deny](../concepts/effects.md#deny), empêche la création et la mise à jour des groupes de ressources qui ne contiennent pas la balise _CostCenter_ :

```json
"if": {
    "allOf": [{
            "field": "type",
            "equals": "Microsoft.Resources/subscriptions/resourceGroups"
        },
        {
            "field": "tags['CostCenter']",
            "exists": false
        }
    ]
},
"then": {
    "effect": "deny"
}
```

> [!NOTE]
> Comme cette règle de stratégie cible un groupe de ressources, le _mode_ de définition de la stratégie doit être « Tous » au lieu de « Indexé ».

### <a name="modify-resources-to-inherit-the-costcenter-tag-when-missing"></a>Modifier les ressources qui ne contiennent pas la balise CostCenter pour qu’elles en héritent

Le deuxième critère de _CostCenter_ est que toutes les ressources qui ne contiennent pas la balise l’héritent du groupe de ressources parent. Si la balise est déjà définie sur la ressource, elle doit être laissée seule, même si elle est différente du groupe de ressources parent. La règle de stratégie suivante utilise l’effet [modify](../concepts/effects.md#modify) :

```json
"policyRule": {
    "if": {
        "field": "tags['CostCenter']",
        "exists": "false"
    },
    "then": {
        "effect": "modify",
        "details": {
            "roleDefinitionIds": [
                "/providers/microsoft.authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c"
            ],
            "operations": [{
                "operation": "add",
                "field": "tags['CostCenter']",
                "value": "[resourcegroup().tags['CostCenter']]"
            }]
        }
    }
}
```

Cette règle de stratégie utilise l’opération **add** au lieu de **addOrReplace**, car il ne faut pas modifier la valeur de la balise si elle est présente lors de la [correction](../how-to/remediate-resources.md) des ressources existantes. Elle se sert également de la fonction de modèle `[resourcegroup()]` pour récupérer la valeur de la balise du groupe de ressources parent.

> [!NOTE]
> Comme cette règle de stratégie cible des ressources qui acceptent les balises, le _mode_ de définition de la stratégie doit être « Indexé ». Cette configuration garantit également que la stratégie ignore les groupes de ressources.

## <a name="configure-the-env-tag"></a>Configurer la balise Env

Dans un environnement Azure géré par Azure Policy en particulier, les spécifications d’étiquette _Env_ exigent les mesures suivantes :

- Modifier la balise _Env_ sur le groupe de ressources en fonction du schéma de nommage du groupe de ressources
- Modifier la balise _Env_ de toutes les ressources du groupe de ressources pour qu’elle soit identique à celle du groupe de ressources parent

### <a name="modify-resource-groups-env-tag-based-on-name"></a>Modifier la balise Env des groupes de ressources en fonction du nom

Une stratégie [modify](../concepts/effects.md#modify) est nécessaire pour chacun des environnements de l’environnement Azure. Elle se présentera pour chacun comme cette définition de stratégie :

```json
"policyRule": {
    "if": {
        "allOf": [{
            "field": "type",
            "equals": "Microsoft.Resources/subscriptions/resourceGroups"
        },
        {
            "field": "name",
            "like": "prd-*"
        },
        {
            "field": "tags['Env']",
            "notEquals": "Production"
        }

    ]
    },
    "then": {
        "effect": "modify",
        "details": {
            "roleDefinitionIds": [
                "/providers/microsoft.authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c"
            ],
            "operations": [{
                "operation": "addOrReplace",
                "field": "tags['Env']",
                "value": "Production"
            }]
        }
    }
}
```

> [!NOTE]
> Comme cette règle de stratégie cible un groupe de ressources, le _mode_ de définition de la stratégie doit être « Tous » au lieu de « Indexé ».

Cette stratégie correspond uniquement aux groupes de ressources dont l’exemple de schéma de nommage utilisé pour les ressources de production est `prd-`. Il est possible de créer des modèles de nommage plus complexes en ajoutant plusieurs conditions de **correspondance** au lieu de l’unique condition **like** de cet exemple.

### <a name="modify-resources-to-inherit-the-env-tag"></a>Modifier les ressources pour qu’elles héritent de la balise Env

L’exigence métier impose que toutes les ressources contiennent la balise _Env_ de leur groupe de ressources parent. Comme cette balise ne peut pas être remplacée, nous allons utiliser l’opération **addOrReplace** avec l’effet [modify](../concepts/effects.md#modify). L’exemple de stratégie modify se présente comme la règle suivante :

```json
"policyRule": {
    "if": {
        "anyOf": [{
            "field": "tags['Env']",
            "notEquals": "[resourcegroup().tags['Env']]"
        },
        {
            "field": "tags['Env']",
            "exists": false
        }
    ]
    },
    "then": {
        "effect": "modify",
        "details": {
            "roleDefinitionIds": [
                "/providers/microsoft.authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c"
            ],
            "operations": [{
                "operation": "addOrReplace",
                "field": "tags['Env']",
                "value": "[resourcegroup().tags['Env']]"
            }]
        }
    }
}
```

> [!NOTE]
> Comme cette règle de stratégie cible des ressources qui acceptent les balises, le _mode_ de définition de la stratégie doit être « Indexé ». Cette configuration garantit également que la stratégie ignore les groupes de ressources.

Cette règle de stratégie recherche les ressources qui n’ont pas la valeur de leur groupe de ressources parent pour la balise _Env_ ou qui ne contiennent pas la balise _Env_. La balise _Env_ des ressources correspondantes est définie sur la valeur du groupe de ressources parent, même si elle existait déjà sur la ressource avec une valeur différente.

## <a name="assign-the-initiative-and-remediate-resources"></a>Assigner l’initiative et corriger les ressources

Une fois les stratégies de balises ci-dessus créées, groupez-les en une initiative unique de gouvernance des balises et attribuez-les à un groupe d’administration ou à un abonnement. L’initiative et les stratégies incluses évalueront la conformité des ressources existantes et des demandes de modification des nouvelles ressources et des ressources mises à jour qui correspondent à la propriété **if** de la règle de stratégie. Toutefois, la stratégie ne met pas automatiquement à jour les ressources non conformes existantes en intégrant les modifications de balise définies.

Comme les stratégies [deployIfNotExists](../concepts/effects.md#deployifnotexists), la stratégie **modify** utilise des tâches de correction pour modifier les ressources non conformes existantes. Suivez les instructions indiquées dans [Guide pratique pour corriger les ressources](../how-to/remediate-resources.md) pour identifier vos ressources **modify** non conformes et corriger les balises en fonction de la taxonomie définie.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous avez fini d’utiliser les ressources de ce tutoriel, effectuez les étapes suivantes pour supprimer les affectations ou définitions créées ci-dessus :

1. Sélectionnez **Définitions** (ou **Affectations** si vous essayez de supprimer une affectation) sous **Création** dans la partie gauche de la page Azure Policy.

1. Recherchez la nouvelle définition d’initiative ou de stratégie (ou affectation) à supprimer.

1. Cliquez avec le bouton droit sur la liste ou cliquez sur le bouton de sélection en fin de définition (ou d’affectation), puis sélectionnez **Supprimer la définition** (ou **Supprimer l’affectation**).

## <a name="review"></a>Révision

Dans ce tutoriel, vous avez découvert les tâches suivantes :

> [!div class="checklist"]
> - Identifier vos exigences métier
> - Faire correspondre chaque spécification à une définition de stratégie
> - Regrouper les stratégies de balises dans une initiative

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les structures des définitions de stratégie, consultez cet article :

> [!div class="nextstepaction"]
> [Structure de définition Azure Policy](../concepts/definition-structure.md)