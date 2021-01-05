---
title: 'Modèle : Opérateurs logiques d’une définition de stratégie'
description: Ce modèle Azure Policy fournit des exemples d’utilisation des opérateurs logiques dans une définition de stratégie.
ms.date: 08/17/2020
ms.topic: sample
ms.openlocfilehash: 3f644cdbfc45b06d1ad5db8e7727c0fa69742f00
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88545588"
---
# <a name="azure-policy-pattern-logical-operators"></a>Modèle Azure Policy : opérateurs logiques

Une définition de stratégie peut contenir plusieurs instructions conditionnelles. Vous aurez peut-être besoin que chaque instruction ait la valeur true ou que certaines d’entre elles aient la valeur true. Pour répondre à ces besoins, le langage comprend des [opérateurs logiques](../concepts/definition-structure.md#logical-operators) **not**, **allOf** et **anyOf**. Ils sont facultatifs et peuvent être imbriqués pour créer des scénarios complexes.

## <a name="sample-1-one-logical-operator"></a>Exemple 1 : Un opérateur logique

Cette définition de stratégie évalue les comptes CosmosDB pour voir si les basculements automatiques et les emplacements à plusieurs écritures sont configurés. Si ce n’est pas le cas, l’[audit](../concepts/effects.md#audit) crée une entrée de journal lorsque la ressource non conforme est créée ou mise à jour.

:::code language="json" source="~/policy-templates/patterns/pattern-logical-operators-1.json":::

### <a name="sample-1-explanation"></a>Exemple 1 : Explication

:::code language="json" source="~/policy-templates/patterns/pattern-logical-operators-1.json" range="6-22" highlight="3":::

Le bloc **policyRule.if** utilise un seul **allOf** pour garantir que les trois conditions sont true.
C’est uniquement lorsque toutes ces conditions ont la valeur true que l’effet **audit** est déclenché.

## <a name="sample-2-multiple-logical-operators"></a>Exemple 2 : Plusieurs opérateurs logiques

Cette définition de stratégie évalue les ressources à la recherche d’un modèle de nommage. Si une ressource ne correspond pas, elle est [refusée](../concepts/effects.md#deny).

:::code language="json" source="~/policy-templates/patterns/pattern-logical-operators-2.json":::

### <a name="sample-2-explanation"></a>Exemple 2 : Explication

:::code language="json" source="~/policy-templates/patterns/pattern-logical-operators-2.json" range="7-21" highlight="2,3,9":::

Ce bloc **policyRule.if** comprend également un **allOf**, mais chacune des conditions est wrappée avec l’opérateur logique **not**. L’opérateur conditionnel à l’intérieur de l’opérateur logique **not** est évalué en premier, puis **not** est évalué afin de déterminer si la clause entière est true ou false. Si les deux opérateurs logiques **not** ont la valeur true, l’effet de stratégie est déclenché.

## <a name="sample-3-combining-logical-operators"></a>Exemple 3 : Combinaison d’opérateurs logiques

Cette définition de stratégie évalue les comptes Java Spring pour déterminer si la trace n’est pas activée ou si elle n’est pas dans un état de réussite.

:::code language="json" source="~/policy-templates/patterns/pattern-logical-operators-3.json":::

### <a name="sample-3-explanation"></a>Exemple 3 : Explication

:::code language="json" source="~/policy-templates/patterns/pattern-logical-operators-3.json" range="6-28" highlight="3,8":::

Ce bloc **policyRule.if** inclut les deux opérateurs logiques **allOf** et **anyOf**. L’opérateur logique **anyOf** prend la valeur true tant qu’une condition incluse a la valeur true. Étant donné que le _type_ se trouve au cœur de l’opérateur **allOf**, il doit toujours prendre la valeur true. Si le _type_ et l’une des conditions de l’opérateur **anyOf** a la valeur true, l’effet de stratégie est déclenché.

## <a name="next-steps"></a>Étapes suivantes

- Passez en revue les autres [modèles et définitions intégrées](./index.md).
- Consultez la [Structure de définition Azure Policy](../concepts/definition-structure.md).
- Consultez la page [Compréhension des effets de Policy](../concepts/effects.md).