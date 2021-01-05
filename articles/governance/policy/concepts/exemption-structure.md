---
title: Détails de la structure d’exemption de stratégie
description: Décrit la définition d’exemption de stratégie utilisée par Azure Policy pour exempter des ressources de l’évaluation d’initiatives ou de définitions.
ms.date: 09/22/2020
ms.topic: conceptual
ms.openlocfilehash: e6ced56c1dc65ca68998c5c58d3e985b63873e0b
ms.sourcegitcommit: a2d8acc1b0bf4fba90bfed9241b299dc35753ee6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/12/2020
ms.locfileid: "91950174"
---
# <a name="azure-policy-exemption-structure"></a>Structure d’exemption Azure Policy

La fonctionnalité d’exemption Azure Policy (préversion) est utilisée pour _exempter_ une hiérarchie de ressources ou une ressource individuelle de l’évaluation d’initiatives ou de définitions. Les ressources qui ne sont _exemptes de_ pour la conformité globale, mais ne peuvent pas être évaluées ou comportent une dérogation temporaire. Pour plus d’informations, consultez [Comprendre l’étendue d’Azure Policy](./scope.md). Les exemptions Azure Policy fonctionnent uniquement avec les modes de [gestionnaire des ressources](./definition-structure.md#resource-manager-modes) et ne fonctionnent pas avec [modes de fournisseur de ressources](./definition-structure.md#resource-provider-modes).

> [!IMPORTANT]
> Cette fonctionnalité est gratuite pendant la **préversion**. Pour plus d’informations sur la tarification, consultez la [tarification d’Azure Policy](https://azure.microsoft.com/pricing/details/azure-policy/). Pour plus d’informations sur les préversions, consultez [Conditions d’utilisation supplémentaires pour les préversions de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Vous devez utiliser JSON pour créer une exemption de stratégie. L’exemption de stratégie contient des éléments pour :

- le nom d’affichage
- description
- metadata
- attribution de stratégie
- définitions de stratégie dans une initiative
- catégorie d'exemption
- expiration

> [!NOTE]
> Une exemption de stratégie est créée en tant qu’objet enfant sur la hiérarchie de ressources ou la ressource individuelle a accordé l’exemption, donc la cible n’est pas incluse dans la définition d’exemption.

Par exemple, le code JSON suivant montre une exemption de stratégie dans la catégorie **dispenser** d’une ressource à une affectation d’initiative nommée `resourceShouldBeCompliantInit`. La ressource est _exempte_ des deux seules définitions de stratégie de l’initiative, la `customOrgPolicy`définition de stratégie personnalisée (référence`requiredTags`) et la définition de stratégie intégrée « emplacements autorisés » (ID : `e56962a6-4747-49cd-b67b-bf8b01975c4c`, `allowedLocations`de référence) :

```json
{
    "id": "/subscriptions/{subId}/resourceGroups/ExemptRG/providers/Microsoft.Authorization/policyExemptions/resourceIsNotApplicable",
    "name": "resourceIsNotApplicable",
    "type": "Microsoft.Authorization/policyExemptions",
    "properties": {
        "displayName": "This resource is scheduled for deletion",
        "description": "This resources is planned to be deleted by end of quarter and has been granted a waiver to the policy.",
        "metadata": {
            "requestedBy": "Storage team",
            "approvedBy": "IA",
            "approvedOn": "2020-07-26T08:02:32.0000000Z",
            "ticketRef": "4baf214c-8d54-4646-be3f-eb6ec7b9bc4f"
        },
        "policyAssignmentId": "/subscriptions/{mySubscriptionID}/providers/Microsoft.Authorization/policyAssignments/resourceShouldBeCompliantInit",
        "policyDefinitionReferenceIds": [
            "requiredTags",
            "allowedLocations"
        ],
        "exemptionCategory": "waiver",
        "expiresOn": "2020-12-31T23:59:00.0000000Z"
    }
}
```

Extrait de l’initiative associée avec le `policyDefinitionReferenceIds` correspondant utilisé par l’exemption de stratégie :

```json
"policyDefinitions": [
    {
        "policyDefinitionId": "/subscriptions/{mySubscriptionID}/providers/Microsoft.Authorization/policyDefinitions/customOrgPolicy",
        "policyDefinitionReferenceId": "requiredTags",
        "parameters": {
            "reqTags": {
                "value": "[parameters('init_reqTags')]"
            }
        }
    },
    {
        "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/e56962a6-4747-49cd-b67b-bf8b01975c4c",
        "policyDefinitionReferenceId": "allowedLocations",
        "parameters": {
            "listOfAllowedLocations": {
                "value": "[parameters('init_listOfAllowedLocations')]"
            }
        }
    }
]
```

## <a name="display-name-and-description"></a>Nom d’affichage et description

Vous utilisez **displayName** et **description** pour identifier l’exemption de stratégie et fournir un contexte pour son utilisation avec la ressource spécifique. **displayName** a une longueur maximale de _128_ caractères et **description** a une longueur maximale de _512_ caractères.

## <a name="metadata"></a>Métadonnées

La propriété **métadonnées** permet de créer n’importe quelle propriété enfant nécessaire pour stocker des informations pertinentes. Dans l’exemple ci-dessus, les propriétés **requestedBy**, **approvedBy**, **approvedOn**et **ticketRef** contiennent des valeurs client pour fournir des informations sur les personnes qui ont demandé l’exemption, qui l’a approuvée et le moment, et un ticket de suivi interne pour la demande. Ces propriétés de **métadonnées** sont des exemples, mais elles ne sont pas obligatoires et les **métadonnées** ne sont pas limitées à ces propriétés enfants.

## <a name="policy-assignment-id"></a>ID d'attribution de stratégie

Ce champ correspond au nom du chemin complet d’une attribution de stratégie ou d’une attribution d’initiative.
`policyAssignmentId` est une chaîne et non un tableau. Cette propriété définit l’affectation à partir de laquelle la hiérarchie de ressources parente ou la ressource individuelle est _exemptée_.

## <a name="policy-definition-ids"></a>ID de définition de stratégie

Si le `policyAssignmentId` concerne une attribution d’initiative, la propriété `policyDefinitionReferenceIds` peut être utilisée pour spécifier la ou les définitions de stratégie de l’initiative à laquelle la ressource objet a une exemption. Comme la ressource peut être exemptée d’une ou de plusieurs définitions de stratégie incluses, cette propriété est un _tableau_. Les valeurs doivent correspondre aux valeurs de la définition d’initiative dans les `policyDefinitions.policyDefinitionReferenceId`champs.

## <a name="exemption-category"></a>Catégorie d’exemption

Deux catégories d’exemption existent et sont utilisées pour regrouper les exemptions :

- **Atténuées** : L’exemption est accordée parce que l’intention de stratégie est satisfaite par une autre méthode.
- **Renonciation** : L’exemption est accordée car l’état de non-conformité de la ressource est provisoirement accepté. Une autre raison de l’utilisation de cette catégorie est pour une ressource ou une hiérarchie de ressources qui doit être exclue d’une ou de plusieurs définitions dans une initiative, mais ne doit pas être exclue de l’initiative entière.

## <a name="expiration"></a>Expiration

Pour définir à quel moment une hiérarchie de ressources ou une ressource individuelle n’est plus _exempter_ à une affectation, définissez la propriété `expiresOn`. Cette propriété facultative doit être au format dateHeure ISO 8601 `yyyy-MM-ddTHH:mm:ss.fffffffZ`.

> [!NOTE]
> Les exemptions de stratégie ne sont pas supprimées lorsque la `expiresOn` date est atteinte. L’objet est conservé pour la conservation des enregistrements, mais l’exemption n’est plus honorée.

## <a name="required-permissions"></a>Autorisations requises

Les autorisations Azure RBAC nécessaires pour gérer les objets d’exemption de stratégie se trouvent dans le `Microsoft.Authorization/policyExemptions`groupe d’opérations. Les rôles intégrés de [collaborateur de stratégie de ressource](../../../role-based-access-control/built-in-roles.md#resource-policy-contributor) et d’[administrateur de la sécurité](../../../role-based-access-control/built-in-roles.md#security-admin) disposent des autorisations `read` et `write` l’enregistreur de données [Policy Insights (préversion)](../../../role-based-access-control/built-in-roles.md#policy-insights-data-writer-preview) dispose de l’autorisation `read`.

Les exemptions ont des mesures de sécurité supplémentaires en raison de l’impact de l’octroi d’une exemption. Au-delà de la nécessité de l’`Microsoft.Authorization/policyExemptions/write`opération sur la hiérarchie de ressources ou une ressource individuelle, le créateur d’une exemption doit avoir le verbe `exempt/Action` sur l’affectation cible.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur la [structure des définitions de stratégies](./definition-structure.md)
- Découvrez comment [créer des stratégies par programmation](../how-to/programmatically-create.md).
- Découvrez comment [obtenir des données de conformité](../how-to/get-compliance-data.md).
- Découvrez comment [corriger des ressources non conformes](../how-to/remediate-resources.md).
- Pour en savoir plus sur les groupes d’administration, consultez [Organiser vos ressources avec des groupes d’administration Azure](../../management-groups/overview.md).