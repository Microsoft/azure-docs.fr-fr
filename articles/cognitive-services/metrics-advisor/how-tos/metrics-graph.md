---
title: Graphe de métriques de Metrics Advisor
titleSuffix: Azure Cognitive Services
description: Guide pratique pour configurer votre graphe de métriques et visualiser les anomalies associées dans vos données.
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.subservice: metrics-advisor
ms.topic: conceptual
ms.date: 09/08/2020
ms.author: mbullwin
ms.openlocfilehash: bcedef4a1339dacaff3fe841e97f985c42320819
ms.sourcegitcommit: 2e72661f4853cd42bb4f0b2ded4271b22dc10a52
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92043151"
---
# <a name="how-to-build-a-metrics-graph-to-analyze-related-metrics"></a>Procédure : Créer un graphe de métriques pour analyser les métriques associées

Chacune des métriques de Metrics Advisor est analysée séparément par un modèle qui apprend à partir de données historiques pour prédire les tendances futures. Un modèle distinct est appliqué à chacune. Toutefois, il peut arriver que plusieurs métriques soient liées les unes aux autres et qu’il soit nécessaire d’analyser les anomalies sur plusieurs métriques. Le **graphe de métriques** est utile de ce point de vue. 

Par exemple, s’il existe différents flux de données de télémétrie dans des métriques distinctes, Metrics Advisor les analyse séparément. Si des anomalies présentes dans une métrique provoquent des anomalies dans d’autres, il peut être utile de rechercher ces relations et la cause racine dans les données lors de la gestion des incidents. Le graphe de métriques permet de créer un graphe topologique visuel des anomalies détectées. 

## <a name="select-a-metric-to-put-the-first-node-to-the-graph"></a>Sélection d’une métrique pour placer le premier nœud dans le graphe

Cliquez sur l’onglet **Graphe de métriques** dans la barre de navigation. La première étape de la création d’un graphe de métriques consiste à placer un nœud sur le graphe. Sélectionnez un flux de données et une métrique en haut de la page. Un nœud apparaît dans le volet inférieur. 

:::image type="content" source="../media/graph/metrics-graph-select.png" alt-text="Sélectionner la métrique":::

## <a name="add-a-noderelation-on-existing-node"></a>Ajout d’un nœud/d’une relation sur un nœud existant

Ensuite, vous devez ajouter un autre nœud et spécifier une relation avec un ou plusieurs nœuds existants. Sélectionnez un nœud et cliquez dessus avec le bouton droit. Le menu contextuel qui s’affiche propose plusieurs options. 

Cliquez sur **Ajouter une relation** : vous pouvez choisir une autre métrique et spécifier le type de relation entre les deux nœuds. Vous pouvez également appliquer des filtres de dimension spécifiques. 

:::image type="content" source="../media/graph/metrics-graph-node-action.png" alt-text="Sélectionner la métrique":::

Après avoir répété les étapes ci-dessus, vous disposerez d’un graphe de métriques décrivant les relations entre toutes les métriques associées.
**Indicateur concernant la couleur des nœuds**
> [!TIP]
> - Lorsque vous sélectionnez une métrique et un filtre de dimension, tous les nœuds du graphe présentant la même métrique et le même filtre de dimension sont colorés en **<font color=blue>bleu</font>** .
> - Les nœuds non sélectionnés qui représentent une métrique dans le graphe sont colorés en **<font color=green>vert</font>** .
> - Si une anomalie est observée dans la métrique actuelle, le nœud est coloré en **<font color=red>rouge</font>** .

## <a name="view-related-metrics-anomaly-status-in-incident-hub"></a>Affichage de l’état d’anomalie des métriques associées dans le hub d’incidents

Une fois le graphe de métriques créé, chaque fois qu’une anomalie est détectée sur ses métriques, vous pouvez afficher l’état de l’anomalie et obtenir une vue d’ensemble de l’incident. 

Cliquez sur un incident au sein du graphe et faites défiler jusqu’à **Analyse croisée des métriques**, sous les informations de diagnostic.

:::image type="content" source="../media/graph/metrics-graph-cross-metrics-analysis.png" alt-text="Sélectionner la métrique":::

## <a name="next-steps"></a>Étapes suivantes

- [Ajustement de la détection d’anomalie selon les commentaires](anomaly-feedback.md)
- [Diagnostic d’un incident](diagnose-incident.md)
- [Configuration des métriques et affinage de la configuration de la détection](configure-metrics.md)
