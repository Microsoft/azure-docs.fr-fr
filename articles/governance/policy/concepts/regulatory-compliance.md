---
title: Conformité réglementaire dans les définitions d’initiative
description: Décrit comment utiliser une définition d’initiative pour regrouper des stratégies par domaine réglementaire, par exemple le contrôle d’accès, la gestion de la configuration, etc.
ms.date: 08/17/2020
ms.topic: conceptual
ms.openlocfilehash: 15b03b5a40cba1290859b02883df70e3df54c5cf
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89645527"
---
# <a name="regulatory-compliance-in-azure-policy"></a>Conformité réglementaire dans Azure Policy

La conformité réglementaire dans Azure Policy fournit des définitions d’initiative intégrées qui permettent de voir la liste des **contrôles** et des **domaines de conformité** en fonction de la responsabilité (_Client_, _Microsoft_, _Partagé_).
Pour les contrôles dont Microsoft est responsable, nous fournissons les détails supplémentaires sur nos résultats d’audit en fonction d’une attestation tierce ainsi que les détails d’implémentation permettant d’atteindre cette conformité.
Les contrôles dont Microsoft est responsable sont `type` [statiques](./definition-structure.md#type).

> [!NOTE]
> La conformité réglementaire est une fonctionnalité en préversion. Pour les mises à jour intégrées, les contrôles d’initiatives sont mappés au standard de conformité correspondant. Les initiatives existantes relatives aux standards de conformité sont en cours de mise à jour pour permettre la prise en charge de la conformité réglementaire.

## <a name="regulatory-compliance-defined"></a>Conformité réglementaire définie

La conformité réglementaire repose sur la partie [regroupement](./initiative-definition-structure.md#policy-definition-groups) d’une définition d’initiative. Dans les fonctionnalités intégrées, chaque regroupement au sein de la définition d’initiative définit un nom (**contrôle**), une catégorie (**domaine de conformité**) et fournit une référence à l’objet [policyMetadata](./initiative-definition-structure.md#metadata-objects) qui contient des informations sur ce **contrôle**. Dans une définition d’initiative de conformité réglementaire, la propriété `category` doit avoir la valeur **Conformité réglementaire**. Dans le cadre d’une définition d’initiative standard, les initiatives de conformité réglementaires prennent en charge des [paramètres](./initiative-definition-structure.md#parameters) pour permettre la création d’affectations dynamiques.

Les clients peuvent créer leurs propres initiatives de conformité réglementaire. Ces définitions peuvent être spécifiques ou copiées à partir de définitions intégrées existantes. Si vous utilisez une définition d’initiative de conformité réglementaire intégrée en tant que référence, il est recommandé de superviser la source des définitions de conformité réglementaire dans le [dépôt GitHub Azure Policy](https://github.com/Azure/azure-policy/tree/master/built-in-policies/policySetDefinitions/Regulatory%20Compliance).

Pour lier une initiative de conformité réglementaire personnalisée à votre tableau de bord Azure Security Center, consultez [Azure Security Center - Utilisation de stratégies de sécurité personnalisées](../../../security-center/custom-security-policies.md).

## <a name="regulatory-compliance-in-portal"></a>Conformité réglementaire dans le portail

Quand une définition d’initiative est créée avec des [groupes](./initiative-definition-structure.md#policy-definition-groups), la page de détails **Conformité** correspondant à cette initiative dans le portail contient des informations supplémentaires. 

Un nouvel onglet, **Contrôles** est ajouté à la page. Un filtrage est disponible par **domaine de conformité**. De plus, les définitions de stratégie sont regroupées en fonction du champ `title` de l’objet **policyMetadata**. Chaque ligne représente un **contrôle** qui indique son état de conformité, le **domaine de conformité** dont il fait partie, les informations de responsabilité ainsi que le nombre de définitions de stratégie non conformes et conformes qui composent ce **contrôle**.

:::image type="content" source="../media/regulatory-compliance/regulatory-compliance-overview.png" alt-text="Capture d’écran de la vue d’ensemble de la conformité réglementaire pour la définition intégrée NIST SP 800-53 R4, montrant des contrôles conformes et non conformes.":::

La sélection d’un **contrôle** entraîne l’ouverture d’une page de détails pour ce contrôle. La **vue d’ensemble** contient les informations de `description` et `requirements`. Sous l’onglet **Stratégies** se trouvent toutes les définitions de stratégie individuelles qui font partie de l’initiative et qui contribuent à ce **contrôle**. L’onglet **Conformité des ressources** fournit une vue précise de chaque ressource évaluée par une stratégie membre du contrôle **affiché**.

> [!NOTE]
> Le type d’évaluation **Géré par Microsoft** correspond au `type` de définition de stratégie [statique](./definition-structure.md#type).

:::image type="content" source="../media/regulatory-compliance/regulatory-compliance-policies.png" alt-text="Capture d’écran de la vue d’ensemble de la conformité réglementaire pour la définition intégrée NIST SP 800-53 R4, montrant des contrôles conformes et non conformes.":::

À partir de la même page de **contrôle**, le passage à l’onglet **Conformité des ressources** permet d’afficher toutes les ressources incluses dans les définitions de stratégie de ce **contrôle**. Des filtres sont disponibles pour le nom ou l’ID, l’état de conformité, le type de ressource ainsi que l’emplacement.

:::image type="content" source="../media/regulatory-compliance/regulatory-compliance-resources.png" alt-text="Capture d’écran de la vue d’ensemble de la conformité réglementaire pour la définition intégrée NIST SP 800-53 R4, montrant des contrôles conformes et non conformes.":::

## <a name="regulatory-compliance-in-sdk"></a>Conformité réglementaire dans le kit SDK

Si la conformité réglementaire est activée pour une définition d’initiative, les enregistrements d’analyse d’évaluation, les événements et les états de stratégie du kit SDK retournent chacun des propriétés supplémentaires. Ces propriétés supplémentaires sont regroupées par état de conformité et fournissent des informations sur le nombre de groupes de chaque état.

Le code suivant est un exemple de résultats ajoutés à partir d’un appel de `summarize` :

```json
"policyGroupDetails": [{
        "complianceState": "noncompliant",
        "count": 4
    },
    {
        "complianceState": "compliant",
        "count": 76
    }
]
```

## <a name="next-steps"></a>Étapes suivantes

- Voir [Structure d’une définition d’initiative](./initiative-definition-structure.md)
- Consultez des exemples à la page [Exemples Azure Policy](../samples/index.md).
- Consultez la page [Compréhension des effets de Policy](./effects.md).
- Découvrez comment [corriger des ressources non conformes](../how-to/remediate-resources.md).
