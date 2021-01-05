---
title: 'Modèle : Opérateur count dans une définition de stratégie'
description: Ce modèle Azure Policy fournit un exemple d’utilisation de l’opérateur count dans une définition de stratégie.
ms.date: 10/14/2020
ms.topic: sample
ms.openlocfilehash: 1339dff7f8bc92a8e38ec5635690cc2069dd8df4
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96005416"
---
# <a name="azure-policy-pattern-the-count-operator"></a>Modèle Azure Policy : Opérateur count

L’opérateur [count](../concepts/definition-structure.md#count) évalue les membres d’un alias \[\*\].

## <a name="sample-policy-definition"></a>Exemple de définition de stratégie

Cette définition de stratégie [audite](../concepts/effects.md#audit) des groupes de sécurité réseau configurés pour autoriser le trafic entrant du protocole RDP (Remote Desktop Protocol).

:::code language="json" source="~/policy-templates/patterns/pattern-count-operator.json":::

### <a name="explanation"></a>Explication

Les principaux composants de l’opérateur **count** sont _field_, _where_ et la condition. Chacun est mis en surbrillance dans l’extrait de code ci-dessous.

- _field_ indique à count de quel [alias](../concepts/definition-structure.md#aliases) il faut évaluer les membres. Ici, nous examinons l’alias **securityRules\[\*\]** _array_ du groupe de sécurité réseau.
- _where_ utilise le langage de stratégie pour définir quels membres d’_array_ remplissent les critères. Dans cet exemple, un opérateur logique **allOf** regroupe trois évaluations de condition différentes de propriétés d’alias _array_ : _direction_, _access_ et _destinationPortRange_.
- La condition count dans cet exemple est **greater**. Count prend la valeur true quand un ou plusieurs membres de l’alias _array_ correspondent à la clause _where_.

:::code language="json" source="~/policy-templates/patterns/pattern-count-operator.json" range="12-32" highlight="3,4,20":::

## <a name="next-steps"></a>Étapes suivantes

- Passez en revue les autres [modèles et définitions intégrées](./index.md).
- Consultez la [Structure de définition Azure Policy](../concepts/definition-structure.md).
- Consultez la page [Compréhension des effets de Policy](../concepts/effects.md).