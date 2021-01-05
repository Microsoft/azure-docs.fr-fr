---
title: Interprétation de votre carte de performance | Microsoft Docs
description: Découvrez comment interpréter votre carte de performance. L’onglet Carte de performance contient les résultats agrégés et analysés de vos tests.
services: internet-analyzer
author: mattcalder
ms.service: internet-analyzer
ms.topic: how-to
ms.date: 10/16/2019
ms.author: mebeatty
ms.openlocfilehash: f43d094193fb266d1ecec7089b44d8b3fd5e9b43
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91330211"
---
# <a name="interpreting-your-scorecard"></a>Interprétation de votre carte de performance

L’onglet Carte de performance contient les résultats agrégés et analysés de vos tests. Chaque test a ses propres cartes de performance. Les cartes de performance fournissent des résumés rapides et pertinents des résultats de mesures basés sur des données pour vos exigences en matière de réseau. Internet Analyzer s’occupe de l’analyse, ce qui vous permet de vous concentrer sur la décision.

L’onglet Carte de performance se trouve dans le menu des ressources d’Internet Analyzer. 


## <a name="filters"></a>Filtres

* ***Test :*** Sélectionnez le test dont vous souhaitez voir les résultats ; chaque test a sa propre carte de performance. Les données de test s’affichent dès que les données sont suffisantes pour effectuer l’analyse, généralement dans les 24 heures. 
* ***Période et date de fin :*** Trois cartes de performance sont générées quotidiennement : chaque carte reflète une période d’agrégation spécifique : les 24 heures précédentes (jour), les sept jours précédents (semaine) et les 30 jours précédents (mois). Utilisez le filtre « Date de fin » pour sélectionner le dernier jour de la période que vous souhaitez voir. 
* ***Pays :*** Pour chaque pays pour lequel vous avez des utilisateurs finaux, une carte de performance est générée. Le filtre Global contient tous les utilisateurs finaux.

## <a name="measurement-count"></a>Nombre de mesures

Le nombre de mesures impacte la confiance de l’analyse. Plus le nombre est élevé, plus le résultat est précis. Au minimum, les tests doivent viser 100 mesures par point de terminaison par jour. Si le nombre de mesures est trop faible, configurez le client JavaScript afin qu’il s’exécute plus fréquemment dans votre application. Les nombres de mesures pour les points de terminaison A et B doivent être très similaires, bien que de petites différences soient attendues et acceptables. En cas de grandes différences, les résultats ne doivent pas être considérés comme fiables.

## <a name="percentiles"></a>Centiles

La latence, mesurée en millisecondes, est une métrique couramment utilisée pour mesurer la vitesse entre une source et une destination sur Internet. Les données de latence ne sont pas distribuées normalement (c’est-à-dire qu’elles ne suivent pas une « courbe en cloche »), car il existe une « longue traîne » de valeurs de grande latence qui faussent les résultats lors de l’utilisation de statistiques telles que la moyenne arithmétique. En guise d’alternative, les centiles offrent un moyen sans distribution d’analyser les données. À titre d’exemple, la valeur médiane, ou 50e centile, indique le milieu de la distribution : la moitié des valeurs sont au-dessus de cette valeur, l’autre moitié au-dessous. Le 75e centile est supérieur à 75 % de toutes les valeurs de la distribution. Internet Analyzer fait référence aux centiles sous forme abrégée en utilisant P50, P75 et P95.

Les centiles Internet Analyzer sont des _exemples de métriques_. Cela diffère de la _métrique de population_ réelle. Par exemple, la latence médiane quotidienne réelle de la population entre les étudiants de l’Université de Californie du Sud et Microsoft est la valeur de latence médiane de toutes les demandes au cours de la journée considérée. Dans la pratique, la mesure de la valeur de toutes les demandes étant irréalisable, nous supposons qu’un échantillon raisonnablement grand est représentatif de la population réelle.

À des fins d’analyse, P50 (médiane) est utile comme valeur attendue pour une distribution de latence. Les centiles plus grands, tels que P95, sont utiles pour identifier la latence élevée dans le pire des cas. Si vous souhaitez comprendre la latence des clients en général, P50 est la métrique correcte sur laquelle se concentrer. Si votre souci est de comprendre les performances pour les clients les moins performants, vous devez vous concentrer sur P95. P75 est un compromis entre ces deux centiles.


## <a name="deltas"></a>Deltas

Un delta est la différence de valeurs de métriques pour les points de terminaison A et B. Les deltas sont calculés pour montrer l’avantage de B par rapport à A. Des valeurs positives indiquent que B a été plus performant que A, tandis que des valeurs négatives indiquent que les performances de B sont pires. Les deltas peuvent être absolus (par exemple, 10 millisecondes) ou relatifs (5 %).

## <a name="confidence-interval"></a>intervalle de confiance 

Les intervalles de confiance (IC) sont une plage de valeurs qui ont une probabilité de contenir la métrique de population telle que la médiane, P75 ou la moyenne. Nous suivons la convention statistique courante consistant à utiliser l’IC de 95 %.

Pour Internet Analyzer, un intervalle de confiance étroit est parfait, car il montre que l’exemple de métrique est probablement très proche de la métrique de population réelle. Un intervalle de confiance large signifie une certitude moindre que notre exemple de métrique reflète la métrique de population réelle. La meilleure façon d’améliorer l’IC consiste à augmenter le nombre de mesures.

## <a name="time-series"></a>Série chronologique 

Une série chronologique montre comment une métrique change dans le temps. Sur Internet, il existe de nombreux facteurs temporels qui impactent les performances, tels que les jours fériés, les périodes de pic de trafic et les différences de population entre les jours de la semaine et le week-end.


## <a name="next-steps"></a>Étapes suivantes

Si vous souhaitez en savoir plus, veuillez consulter la page [Qu’est-ce qu’Internet Analyzer ? (Préversion)](internet-analyzer-overview.md).
