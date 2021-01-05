---
title: Utiliser Azure Monitor pour les conteneurs afin de consulter les mesures en temps réel | Microsoft Docs
description: Cet article décrit l’affichage en temps réel des mesures sans l’utilisation de kubectl avec Azure Monitor pour les conteneurs.
ms.topic: conceptual
ms.date: 10/15/2019
ms.custom: references_regions
ms.openlocfilehash: 81d7210778fd6b5d75fb4b4fa8e066d2e015174f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85338027"
---
# <a name="how-to-view-metrics-in-real-time"></a>Comment afficher les mesures en temps réel

La fonctionnalité Azure Monitor Live Data pour les conteneurs vous permet de visualiser les mesures relatives à l’état des nœuds et des pods dans un cluster en temps réel. Elle émule l’accès direct aux commandes `kubectl top nodes`, `kubectl get pods –all-namespaces`et `kubectl get nodes` pour appeler, analyser et visualiser les données dans les graphiques de performances inclus dans cette recommandation analytique.

Cet article fournit une vue d’ensemble détaillée et vous aide à mieux comprendre l’utilisation de cette fonctionnalité.

>[!NOTE]
>Les clusters AKS activés en tant que [clusters privés](https://azure.microsoft.com/updates/aks-private-cluster/) ne sont pas pris en charge par cette fonctionnalité. Cette fonctionnalité repose sur l’accès direct à l’API Kubernetes via un serveur proxy depuis votre navigateur. L’activation de la sécurité réseau pour bloquer l’API Kubernetes depuis ce proxy bloque ce trafic.

Pour obtenir de l’aide concernant la configuration ou la résolution des problèmes se rapportant à la fonctionnalité Live Data (préversion), consultez notre [guide de configuration](container-insights-livedata-setup.md).

## <a name="how-it-works"></a>Fonctionnement

Comme la fonctionnalité Live Data (préversion) accède directement à l’API Kubernetes, des informations supplémentaires sur le modèle d’authentification sont mises à disposition [ici](https://kubernetes.io/docs/concepts/overview/kubernetes-api/).

Cette fonctionnalité effectue une opération d’interrogation sur les points de terminaison de mesures (notamment `/api/v1/nodes`, `/apis/metrics.k8s.io/v1beta1/nodes` et `/api/v1/pods`), toutes les 5 secondes par défaut. Ces données sont mises en cache dans votre navigateur et sont représentées sous forme de graphique dans les quatre graphiques de performances inclus dans Azure Monitor pour les conteneurs, sous l’onglet **Cluster** en sélectionnant **Démarrer (préversion)** . Chaque interrogation ultérieure est représentée sous forme de graphique dans une fenêtre de visualisation toutes les 5 minutes.

![Option Démarrer dans la vue Cluster](./media/container-insights-livedata-metrics/cluster-view-go-live-example-01.png)

La fréquence d’interrogation est configurée dans la liste déroulante **Définir l’intervalle**. Vous pouvez ainsi définir la durée entre chaque interrogation : toutes les 1, 5, 15 ou 30 secondes.

![Menu déroulant pour définir l’intervalle d’interrogation Démarrer](./media/container-insights-livedata-metrics/cluster-view-polling-interval-dropdown.png)

>[!IMPORTANT]
>Lorsque vous essayez de résoudre un problème, nous vous recommandons de définir la fréquence d’interrogation sur 1 seconde pendant la durée de vos tentatives de résolution. Ces requêtes peuvent avoir un impact sur la disponibilité et la limitation de l’API Kubernetes sur votre cluster. Ensuite, définissez un intervalle d’interrogation plus long.

>[!IMPORTANT]
>Aucune donnée n’est stockée de façon permanente pendant l’exécution de cette fonctionnalité. Toutes les informations capturées lors de cette session sont immédiatement supprimées lorsque vous la quittez, ou que vous fermez votre navigateur. Les données restent uniquement présentes pour la visualisation dans le laps de temps de 5 minutes. Toutes les mesures datant de plus de 5 minutes sont également supprimées.

Ces graphiques ne peuvent pas être épinglés au dernier tableau de bord Azure que vous avez affiché en mode réel.

## <a name="metrics-captured"></a>Mesures capturées

### <a name="node-cpu-utilization---node-memory-utilization-"></a>Pourcentage d’utilisation de l’UC du nœud / Pourcentage d’utilisation de la mémoire du nœud

Ces deux graphiques de performances mappent vers un équivalent de l’appel de `kubectl top nodes`, ainsi qu’à la capture des résultats des colonnes **CPU%** (Pourcentage d’UC) et **MEMORY%** (Pourcentage de mémoire) dans leurs graphiques respectifs.

![Exemples de résultats de la commande Kubectl top nodes](./media/container-insights-livedata-metrics/kubectl-top-nodes-example.png)

![Graphique Pourcentages d’utilisation de l’UC de nœud](./media/container-insights-livedata-metrics/cluster-view-node-cpu-util.png)

![Graphique Pourcentages d’utilisation de la mémoire du nœud](./media/container-insights-livedata-metrics/cluster-view-node-memory-util.png)

Les calculs de centile fonctionnent dans les clusters de grande taille pour aider à identifier les nœuds hors norme dans votre cluster. Par exemple, pour comprendre si les nœuds sont sous-utilisés à des fins de descente en puissance. À l’aide de l’agrégation **Min**, vous pouvez voir les nœuds peu utilisés dans le cluster. Pour approfondir vos investigations, sélectionnez l’onglet **Nœuds**, puis triez les entrées de la grille par utilisation de l’UC ou de la mémoire.

Cela vous permet aussi de comprendre quels nœuds atteignent leurs limites et si vous devez envisager une montée en charge. L’utilisation des agrégations **Max** et **P95** peut vous aider à déterminer si certains nœuds du cluster utilisent beaucoup de ressources. Pour approfondir votre investigation, basculez vers l’onglet **Nœuds**.

### <a name="node-count"></a>Nombre de nœuds

Ce graphique de performances mappe vers un équivalent de l’appel de `kubectl get nodes`, ainsi que du mappage de la colonne **STATUS** (État) vers un graphique groupé par types d’états.

![Exemples de résultat de la commande Kubectl get nodes](./media/container-insights-livedata-metrics/kubectl-get-nodes-example.png)

![Graphique du nombre de nœuds](./media/container-insights-livedata-metrics/cluster-view-node-count-01.png)

Les nœuds sont définis sur l’état **Prêt** ou **Pas prêt**. Ils sont comptés (et un nombre total est créé) et les résultats de ces deux agrégations sont représentés sous forme de graphique.
Cela vous permet notamment de savoir si vos nœuds sont définis sur l’état Échec. L’agrégation **Pas prêt** vous permet de voir rapidement le nombre de nœuds dans votre cluster actuellement à l’état **Pas prêt**.

### <a name="active-pod-count"></a>Nombre de pods actifs

Ce graphique de performances mappe vers un équivalent de l’appel de `kubectl get pods –all-namespaces`, ainsi que vers la colonne **STATUS** (État) vers un graphique groupé par types d’états.

![Exemples de résultat de la commande Kubectl get pods](./media/container-insights-livedata-metrics/kubectl-get-pods-example.png)

![Graphique du nombres de nœuds du pod](./media/container-insights-livedata-metrics/cluster-view-node-pod-count.png)

>[!NOTE]
>Il est possible que les noms d’état tels qu’ils sont interprétés par `kubectl` ne correspondent pas exactement au graphique.

## <a name="next-steps"></a>Étapes suivantes

Consultez [Exemples de requêtes de journal](container-insights-log-search.md#search-logs-to-analyze-data) pour voir des requêtes et des exemples prédéfinis permettant de créer des alertes, des visualisations, ou d’effectuer une analyse plus poussée de vos clusters.
