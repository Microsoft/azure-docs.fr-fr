---
title: 'Modèle : Propriétés de champs dans une définition de stratégie'
description: Ce modèle Azure Policy fournit un exemple d’utilisation des propriétés de champs dans une définition de stratégie.
ms.date: 10/14/2020
ms.topic: sample
ms.openlocfilehash: 267c687f78f0bbb100843faee40ab6f3d3cbb64c
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92072966"
---
# <a name="azure-policy-pattern-field-properties"></a>Modèle Azure Policy : Propriétés de champs

L’opérateur [field](../concepts/definition-structure.md#fields) évalue l’[alias](../concepts/definition-structure.md#aliases) ou la propriété spécifié(e) par rapport à une valeur fournie pour une [ condition](../concepts/definition-structure.md#conditions) donnée.

## <a name="sample-policy-definition"></a>Exemple de définition de stratégie

Cette définition de stratégie vous permet de définir des régions autorisées qui répondent aux exigences de géolocalisation de votre organisation. Les ressources autorisées sont définies dans le paramètre **listOfAllowedLocations** (_array_). Les ressources qui correspondent à la définition sont [refusées](../concepts/effects.md#deny).

:::code language="json" source="~/policy-templates/patterns/pattern-fields.json":::

### <a name="explanation"></a>Explication

:::code language="json" source="~/policy-templates/patterns/pattern-fields.json" range="18-36" highlight="3,7,11":::

L’opérateur **field** est utilisé trois fois dans l’[opérateur logique](../concepts/definition-structure.md#logical-operators) **allOf**.

- La première utilisation évalue la propriété `location` avec la condition **notIn** pour le paramètre **listOfAllowedLocations**. **notIn** fonctionne, car elle attend un tableau (_array_) et le paramètre en est un. Si le `location` de la ressource créée ou mise à jour ne figure pas dans la liste approuvée, cet élément prend la valeur true.
- La deuxième utilisation évalue également la propriété `location`, mais utilise la condition **notEquals** pour vérifier si la ressource est _globale_. Si le `location` de la ressource créée ou mise à jour n’est pas _global_, cet élément prend la valeur true.
- La dernière utilisation évalue la propriété `type` et utilise la condition **notEquals** pour valider le fait que le type de ressource n’est pas _Microsoft. AzureActiveDirectory/b2cDirectories_. Si ce n’est pas le cas, cet élément prend la valeur true.

Si les trois instructions de condition de l’opérateur logique **allOf** sont évaluées à la valeur true, la création ou la mise à jour de ressource est bloquée par Azure Policy.

## <a name="next-steps"></a>Étapes suivantes

- Passez en revue les autres [modèles et définitions intégrées](./index.md).
- Consultez la [Structure de définition Azure Policy](../concepts/definition-structure.md).
- Consultez la page [Compréhension des effets de Policy](../concepts/effects.md).