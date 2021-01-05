---
title: 'Modèle : Paramètres dans une définition de stratégie'
description: Ce modèle Azure Policy fournit un exemple d’utilisation des paramètres de chaîne et de tableau dans une définition de stratégie, ainsi que du paramétrage de l’effet.
ms.date: 08/17/2020
ms.topic: sample
ms.openlocfilehash: 7bb991c43a1deee39dd047aa7d814c124712fe69
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89649944"
---
# <a name="azure-policy-pattern-parameters"></a>Modèle Azure Policy : paramètres

Une définition de stratégie peut être rendue dynamique afin de réduire le nombre de définitions de stratégie nécessaires à l’aide des [paramètres](../concepts/definition-structure.md#parameters). Un paramètre est défini lors de l’attribution de la stratégie. Les paramètres ont un ensemble de propriétés prédéfinies qui les décrivent, ainsi que la façon dont ils sont utilisés.

## <a name="sample-1-string-parameters"></a>Exemple 1 : Paramètre de chaîne

Cette définition de stratégie utilise deux paramètres (**tagName** et **tagValue**) pour définir ce que l’attribution de stratégie doit rechercher dans les ressources. Ce format permet à la définition de stratégie d’être utilisée avec autant de combinaisons nom-valeur d’étiquette que souhaité. Toutefois, il ne conserve qu’une seule définition de stratégie.

> [!NOTE]
> Pour obtenir un exemple d’étiquette qui utilise **mode** _All_ et fonctionne avec un groupe de ressources, consultez [Modèle : Étiquettes - Exemple no1](./pattern-tags.md#sample-1-parameterize-tags).

:::code language="json" source="~/policy-templates/patterns/pattern-parameters-1.json":::

### <a name="sample-1-explanation"></a>Exemple 1 : Explication

:::code language="json" source="~/policy-templates/patterns/pattern-parameters-1.json" range="8-13":::

Dans cette partie de la définition de stratégie, le paramètre **tagName** est défini en tant que _chaîne_ et une description est fournie pour son utilisation.

Le paramètre est ensuite utilisé dans le bloc **policyRule.if** pour rendre la stratégie dynamique. Ici, il est utilisé pour définir le champ évalué, qui correspond à une étiquette ayant la valeur **tagName**.

:::code language="json" source="~/policy-templates/patterns/pattern-parameters-1.json" range="22-27" highlight="3":::

## <a name="sample-2-array-parameters"></a>Exemple 2 : Paramètres de tableau

Cette définition de stratégie utilise un seul paramètre (**listOfBandwidthinMbps**) pour vérifier si la ressource de circuit ExpressRoute a configuré le paramètre de bande passante sur l’une des valeurs approuvées. Si ce n’est pas le cas, la création ou la mise à jour de la ressource est [refusée](../concepts/effects.md#deny).

:::code language="json" source="~/policy-templates/patterns/pattern-parameters-2.json":::

### <a name="sample-2-explanation"></a>Exemple 2 : Explication

:::code language="json" source="~/policy-templates/patterns/pattern-parameters-2.json" range="6-12":::

Dans cette partie de la définition de stratégie, le paramètre **listOfBandwidthinMbps** est défini en tant que _tableau_ et une description est fournie pour son utilisation. En tant que _tableau_, il comprend plusieurs valeurs à rechercher.

Le paramètre est ensuite utilisé dans le bloc **policyRule.if**. En tant que paramètre de _tableau_, le **in** ou le **notIn** de la [condition](../concepts/definition-structure.md#conditions) d’un _tableau_
 doivent être utilisés.
Ici, il est utilisé sur l’alias **serviceProvider.bandwidthInMbps** comme l’une des valeurs définies.

:::code language="json" source="~/policy-templates/patterns/pattern-parameters-2.json" range="21-24" highlight="3":::

## <a name="sample-3-parameterized-effect"></a>Exemple 3 : Effet paramétrable

Une façon courante de rendre les définitions de stratégie réutilisables consiste à paramétrer l’effet lui-même. Cet exemple utilise un seul paramètre, **effect**. Le paramétrage de l’effet permet d’assigner la même définition à différentes étendues avec des effets différents.

:::code language="json" source="~/policy-templates/patterns/pattern-parameters-3.json":::

### <a name="sample-3-explanation"></a>Exemple 3 : Explication

:::code language="json" source="~/policy-templates/patterns/pattern-parameters-3.json" range="11-25":::

Dans cette partie de la définition de stratégie, le paramètre **effect** est défini en tant que _chaîne_. La définition de stratégie définit la valeur par défaut d’une affectation sur _audit_, et limite les autres options à _disabled_ et _deny_.

Le paramètre est ensuite utilisé dans le bloc **policyRule.then** pour obtenir _effect_.

:::code language="json" source="~/policy-templates/patterns/pattern-parameters-3.json" range="38-40" highlight="2":::

## <a name="next-steps"></a>Étapes suivantes

- Passez en revue les autres [modèles et définitions intégrées](./index.md).
- Consultez la [Structure de définition Azure Policy](../concepts/definition-structure.md).
- Consultez la page [Compréhension des effets de Policy](../concepts/effects.md).