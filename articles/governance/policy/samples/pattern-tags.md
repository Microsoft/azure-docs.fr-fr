---
title: 'Modèle : Utilisation d’étiquettes dans une définition de stratégie'
description: Ce modèle Azure Policy fournit des exemples sur la façon d’ajouter des étiquettes paramétrables ou d’hériter d’étiquettes à partir d’un groupe de ressources dans une définition de stratégie.
ms.date: 08/17/2020
ms.topic: sample
ms.openlocfilehash: 3016fc7889f68fd13e993c67ca645a4af055c651
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88545554"
---
# <a name="azure-policy-pattern-tags"></a>Modèle Azure Policy : étiquettes

Les [étiquettes](../../..//azure-resource-manager/management/tag-resources.md) représentent un élément important dans la gestion, l’organisation et l’administration de vos ressources Azure. Azure Policy permet de configurer à grande échelle des étiquettes sur vos ressources nouvelles et existantes au moyen de l’effet [modify](../concepts/effects.md#modify) et des [tâches de correction](../how-to/remediate-resources.md).

## <a name="sample-1-parameterize-tags"></a>Exemple 1 : Paramétrer des étiquettes

Cette définition de stratégie utilise deux paramètres (**tagName** et **tagValue**) pour définir ce que l’attribution de stratégie doit rechercher dans les groupes de ressources. Ce format permet à la définition de stratégie d’être utilisée avec autant de combinaisons nom-valeur d’étiquette que souhaité. Toutefois, il ne conserve qu’une seule définition de stratégie.

> [!NOTE]
> Bien que ce modèle de définition de stratégie soit similaire à celui de [Modèle : Paramètres – Exemple no1](./pattern-parameters.md#sample-1-string-parameters), cet exemple utilise **mode** _All_ et cible des groupes de ressources.

:::code language="json" source="~/policy-templates/patterns/pattern-tags-1.json":::

### <a name="sample-1-explanation"></a>Exemple 1 : Explication

:::code language="json" source="~/policy-templates/patterns/pattern-tags-1.json" range="2-8" highlight="3":::

Dans cet exemple, **mode** a la valeur _All_, car il cible un groupe de ressources. Dans la plupart des cas, **mode** doit avoir la valeur _Indexed_ lors de l’utilisation d’étiquettes. Pour plus d’informations, consultez [modes](../concepts/definition-structure.md#resource-manager-modes).

:::code language="json" source="~/policy-templates/patterns/pattern-tags-1.json" range="26-36" highlight="7-8":::

Dans cette partie de la définition de stratégie, `concat` associe le paramètre **tagName** paramétrable au format `tags['name']` pour indiquer à **field** d’évaluer cette étiquette pour le paramètre **tagValue**.
Lorsque **notEquals** est utilisé, si **tags\[tagName\]** n’est pas égal à **tagValue**, l’effet  **modify** est déclenché.

:::code language="json" source="~/policy-templates/patterns/pattern-tags-1.json" range="43-47" highlight="3-4":::

Ici, pour créer l’étiquette ou la mettre à jour à la valeur souhaitée sur le groupe de ressources évalué, le même format que celui de l’utilisation de valeurs d’étiquettes paramétrables est utilisé par l’opération **addOrReplace**.

## <a name="sample-2-inherit-tag-value-from-resource-group"></a>Exemple 2 : Hériter du groupe de ressources la valeur d’étiquette

Cette définition de stratégie utilise le paramètre **tagName** pour déterminer de quelle valeur de l’étiquette hériter à partir du groupe de ressources parent.

:::code language="json" source="~/policy-templates/patterns/pattern-tags-2.json":::

### <a name="sample-2-explanation"></a>Exemple 2 : Explication

:::code language="json" source="~/policy-templates/patterns/pattern-tags-2.json" range="2-8" highlight="3":::

Dans cet exemple, **mode** a la valeur _Indexed_, car il ne cible pas un groupe de ressources ni un abonnement, bien qu’il obtienne la valeur d’un groupe de ressources. Pour plus d’informations, consultez [modes](../concepts/definition-structure.md#resource-manager-modes).

:::code language="json" source="~/policy-templates/patterns/pattern-tags-2.json" range="19-29" highlight="3-4,7-8":::

**policyRule.if** utilise `concat`, comme dans [Exemple no1](#sample-1-parameterize-tags), pour évaluer la valeur de **tagName**, mais se sert de la fonction `resourceGroup()` pour la comparer à la valeur de la même étiquette sur le groupe de ressources parent. La seconde clause ici vérifie que l’étiquette sur le groupe de ressources a une valeur, et qu’elle n’est pas nulle.

:::code language="json" source="~/policy-templates/patterns/pattern-tags-2.json" range="36-40" highlight="3-4":::

Ici, la valeur assignée à l’étiquette **tagName** sur la ressource utilise également la fonction `resourceGroup()` pour récupérer la valeur à partir du groupe de ressources parent. Ainsi, vous pouvez hériter d’étiquettes provenant de groupes de ressources parent. Si vous avez déjà créé la ressource, mais que vous n’avez pas ajouté l’étiquette, cette même définition de stratégie et une [tâche de correction](../how-to/remediate-resources.md) peuvent mettre à jour les ressources existantes.

## <a name="next-steps"></a>Étapes suivantes

- Passez en revue les autres [modèles et définitions intégrées](./index.md).
- Consultez la [Structure de définition Azure Policy](../concepts/definition-structure.md).
- Consultez la page [Compréhension des effets de Policy](../concepts/effects.md).