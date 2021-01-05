---
title: Concepteur de vues Azure Monitor - Options de conversion en classeurs
description: ''
author: austonli
ms.author: aul
ms.subservice: ''
ms.topic: conceptual
ms.date: 02/07/2020
ms.openlocfilehash: 7bfa831332451718c0c9c05023b90104d2b8b02b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77658708"
---
# <a name="azure-monitor-view-designer-to-workbooks-conversion-options"></a>Concepteur de vues Azure Monitor - Options de conversion en classeurs
Le [concepteur de vues](view-designer.md) est une fonctionnalité d'Azure Monitor qui vous permet de créer des vues personnalisées pour vous aider à visualiser les données de votre espace de travail Log Analytics, avec des graphiques, des listes et des chronologies. Celles-ci sont progressivement supprimées et remplacées par des classeurs qui offrent des fonctionnalités supplémentaires. Cet article compare les concepts fondamentaux entre les deux et les options pour convertir des vues en classeurs.

## <a name="basic-workbook-designs"></a>Conceptions de classeurs de base

Le concepteur de vues a un style de représentation statique fixe, tandis que les classeurs permettent d’inclure et de modifier la façon dont les données sont représentées. Les images ci-dessous montrent deux exemples de la façon dont vous pouvez organiser les classeurs lors de la conversion des vues.

[Classeur vertical](view-designer-conversion-examples.md#vertical)
![Vertical](media/view-designer-conversion-options/view-designer-vertical.png)

[Classeur avec onglets](view-designer-conversion-examples.md#tabbed)
![Onglet Distribution des types de données](media/view-designer-conversion-options/distribution-tab.png)
![Onglet Types de données dans le temps](media/view-designer-conversion-options/over-time-tab.png)

## <a name="tile-conversion"></a>Conversion de vignettes
Le concepteur de vues utilise la fonctionnalité de vignette de vue d’ensemble pour représenter et résumer l’état global. Il est représenté dans sept vignettes, qui montrent des nombres ou des graphiques. Dans les classeurs, les utilisateurs peuvent créer des visualisations similaires et les épingler pour qu’elles ressemblent au style original des vignettes de vue d’ensemble. 

![Galerie](media/view-designer-conversion-options/overview.png)


## <a name="view-dashboard-conversion"></a>Conversion des tableaux de bord des vues
Les vignettes du concepteur de vues se composent généralement de deux sections : une visualisation et une liste qui correspond aux données de la visualisation, par exemple la vignette **Anneau & Liste**.

![Anneau](media/view-designer-conversion-options/donut-example.png)

Avec les classeurs, nous permettons à l’utilisateur à choisir d’interroger une des deux sections de la vue ou les deux. La formulation des requêtes dans les classeurs est un processus simple en deux étapes. D’abord, les données sont générées à partir de la requête, et ensuite, les données sont rendues sous la forme d’une visualisation.  Voici un exemple de recréation de cette vue dans des classeurs :

![Convertir](media/view-designer-conversion-options/convert-donut.png)


## <a name="next-steps"></a>Étapes suivantes
- [Accès aux classeurs et autorisations](view-designer-conversion-access.md)