---
title: 'Modèle : Déployer des ressources avec une définition de stratégie'
description: Ce modèle Azure Policy fournit un exemple de déploiement de ressources à l’aide d’une définition de stratégie deployIfNotExists.
ms.date: 08/17/2020
ms.topic: sample
ms.openlocfilehash: 0a9eec54954b8963f38b3f19a0d0cabffe1092e5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89649975"
---
# <a name="azure-policy-pattern-deploy-resources"></a>Modèle Azure Policy : Déployer des ressources

L’effet [deployIfNotExists](../concepts/effects.md#deployifnotexists) permet de déployer un [modèle Azure Resource Manager](../../../azure-resource-manager/templates/overview.md) (modèle ARM) lors de la création ou de la mise à jour d’une ressource qui n’est pas conforme. Cette approche peut être préférable à l’utilisation de l’effet [deny](../concepts/effects.md#deny), car elle permet de continuer à créer des ressources tout en garantissant que les modifications nécessaires sont apportées afin de les rendre conformes.

## <a name="sample-policy-definition"></a>Exemple de définition de stratégie

Cette définition de stratégie utilise l’opérateur **field** pour évaluer le `type` de la ressource créée ou mise à jour. Quand cette ressource est un _Microsoft.Network/virtualNetworks_, la stratégie recherche un observateur réseau à l’emplacement de la ressource nouvelle ou mise à jour. Si aucun observateur réseau correspondant n’est trouvé, le modèle ARM est déployé pour créer la ressource manquante.

:::code language="json" source="~/policy-templates/patterns/pattern-deploy-resources.json":::

### <a name="explanation"></a>Explication

#### <a name="existencecondition"></a>existenceCondition

:::code language="json" source="~/policy-templates/patterns/pattern-deploy-resources.json" range="18-23":::

Le bloc **properties.policyRule.then.details** indique à Azure Policy ce qu’il faut rechercher en rapport avec la ressource créée ou mise à jour dans le bloc **properties.policyRule.if**. Dans cet exemple, un observateur réseau dans le groupe de ressources **networkWatcherRG** doit exister avec **field** `location` égal à l’emplacement de la ressource nouvelle ou mise à jour. L’utilisation de la fonction `field()` permet à l’**existenceCondition** d’accéder aux propriétés de la ressource nouvelle ou mise à jour, en particulier à la propriété `location`.

#### <a name="roledefinitionids"></a>roleDefinitionIds

:::code language="json" source="~/policy-templates/patterns/pattern-deploy-resources.json" range="24-26":::

La propriété _array_ **roleDefinitionIds** dans le bloc **properties.policyRule.then.details** indique à la définition de stratégie les droits dont l’identité managée a besoin pour déployer le modèle Resource Manager inclus. Cette propriété doit être définie pour inclure des rôles disposant des autorisations nécessaires au déploiement de modèle, mais doit utiliser le concept de « principe des privilèges minimum » et n’avoir accès qu’aux opérations nécessaires, et rien de plus.

#### <a name="deployment-template"></a>Modèle de déploiement

La partie **deployment** de la définition de stratégie a un bloc de **properties** qui définit les trois composants principaux :

- **mode** : cette propriété définit le [mode de déploiement](../../../azure-resource-manager/templates/deployment-modes.md) du modèle.

- **template** : cette propriété inclut le modèle lui-même. Dans cet exemple, le paramètre de modèle **location** définit l’emplacement de la nouvelle ressource d’observateur réseau.

  :::code language="json" source="~/policy-templates/patterns/pattern-deploy-resources.json" range="30-44":::
  
- **parameters** : cette propriété définit les paramètres qui sont fournis au **template**. Les noms des paramètres doivent correspondre à ce qui est défini dans **template**. Dans cet exemple, le paramètre se nomme **location** afin qu’il y ait correspondance. La valeur de **location** utilise de nouveau la fonction `field()` pour récupérer la valeur de la ressource évaluée, qui est le réseau virtuel dans le bloc **policyRule.if**.

  :::code language="json" source="~/policy-templates/patterns/pattern-deploy-resources.json" range="45-49":::

## <a name="next-steps"></a>Étapes suivantes

- Passez en revue les autres [modèles et définitions intégrées](./index.md).
- Consultez la [Structure de définition Azure Policy](../concepts/definition-structure.md).
- Consultez la page [Compréhension des effets de Policy](../concepts/effects.md).