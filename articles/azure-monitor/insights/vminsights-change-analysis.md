---
title: Analyse des modifications dans Azure Monitor pour machines virtuelles
description: L’intégration d’Azure Monitor pour machines virtuelles avec Analyse des changements d'application vous permet d’afficher toute modification apportée à une machine virtuelle susceptible d’avoir affecté ses performances.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 09/23/2020
ms.openlocfilehash: 59799a09436d5968a441f6f17655d3138a2d84d8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91710979"
---
# <a name="change-analysis-in-azure-monitor-for-vms"></a>Analyse des modifications dans Azure Monitor pour machines virtuelles
L’intégration d’Azure Monitor pour machines virtuelles avec [Analyse des changements d'application](../app/change-analysis.md) vous permet d’afficher toute modification apportée à une machine virtuelle susceptible d’avoir affecté ses performances.

## <a name="overview"></a>Vue d’ensemble
Supposez que vous avez une machine virtuelle qui s’exécute lentement, et que vous souhaitez vérifier si les modifications récemment apportées à sa configuration ont pu affecter ses performances. Vous visualisez les performances de la machine virtuelle à l’aide d’Azure Monitor pour machines virtuelles et vous constatez une augmentation de l’utilisation de la mémoire au cours de la dernière heure. L’analyse des changements peut vous aider à déterminer si des modifications de configuration apportées à ce moment-là étaient à l’origine de cette augmentation.

Le service Analyse des changements d'application regroupe les modifications d’[Azure Resource Graph](../../governance/resource-graph/how-to/get-resource-changes.md) ainsi que les modifications de propriétés imbriquées telles que les règles de sécurité réseau à partir d’Azure Resource Manager. 

## <a name="enabling-change-analysis"></a>Activation de l’analyse des changements
Pour intégrer l’analyse des changements dans Azure Monitor pour machines virtuelles, vous devez inscrire le fournisseur de *Microsoft.ChangeAnalysis*. La première fois que vous lancez Azure Monitor pour machines virtuelles ou Analyse des changements d'application dans le portail Azure, ce fournisseur de ressources est inscrit automatiquement pour vous. Analyse des changements d'application est un service gratuit qui n’entraîne aucune surcharge des performances sur les ressources.

## <a name="view-change-analysis"></a>Afficher l’analyse des changements
Vous pouvez accéder à l’analyse des changements à partir de l’onglet **Performances** ou **Carte** d’Azure Monitor pour machines virtuelles en sélectionnant l’option **Modifier**. 

[![Enquêter sur les changements](media/vminsights-change-analysis/investigate-changes-screenshot.png)](media/vminsights-change-analysis/investigate-changes-screenshot-zoom.png#lightbox)


Cliquez sur le bouton **Investigate changes** (Enquêter sur les changements) pour lancer la page Analyse des changements d'application filtrée pour la machine virtuelle. Vous pouvez examiner les modifications listées pour voir si l’une d’elles peut être à l’origine du problème. Si vous avez des doutes quant à une modification particulière, vous pouvez référencer la colonne **Change by** pour identifier la personne qui a effectué la modification.

[![Détails des modifications](media/vminsights-change-analysis/change-details-screenshot.png)](media/vminsights-change-analysis/change-details-screenshot.png#lightbox)

## <a name="next-steps"></a>Étapes suivantes
- Apprenez-en davantage sur l’[Analyse des changements d'application](../app/change-analysis.md).
- Apprenez-en davantage sur [Azure Resource Graph](../../governance/resource-graph/how-to/get-resource-changes.md). 

