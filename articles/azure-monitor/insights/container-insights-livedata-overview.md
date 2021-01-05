---
title: Voir Live Data (préversion) avec Azure Monitor pour conteneurs | Microsoft Docs
description: Cet article décrit la vue en temps réel des journaux, métriques de pod et événements Kubernetes, sans l’utilisation de kubectl, dans Azure Monitor pour conteneurs.
ms.topic: conceptual
ms.date: 12/17/2020
ms.custom: references_regions
ms.openlocfilehash: 3655ff8e5879aa4113753b5529c1e484fb079401
ms.sourcegitcommit: d79513b2589a62c52bddd9c7bd0b4d6498805dbe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/18/2020
ms.locfileid: "97672866"
---
# <a name="how-to-view-kubernetes-logs-events-and-pod-metrics-in-real-time"></a>Comment voir les journaux, métriques de pod et événements Kubernetes en temps réel

Azure Monitor pour conteneurs comprend la fonctionnalité Live Data (préversion), une fonctionnalité de diagnostic avancé qui vous permet d’accéder directement aux journaux (stdout/stderror), métriques de pod et événements de conteneur Azure Kubernetes Service (AKS). Elle expose un accès direct aux `kubectl logs -c`, aux événements `kubectl get` et aux `kubectl top pods`. Un volet de la console affiche les journaux, les métriques et les événements générés par le moteur de conteneur, afin de faciliter la résolution des problèmes en temps réel.

Cet article fournit une vue d’ensemble détaillée et vous aide à mieux comprendre l’utilisation de cette fonctionnalité.

Pour obtenir de l’aide lors de la configuration ou de la résolution des problèmes se rapportant à la fonctionnalité Live Data (préversion), consultez notre [guide de configuration](container-insights-livedata-setup.md). Comme cette fonctionnalité accède directement à l’API Kubernetes, des informations supplémentaires sur le modèle d’authentification sont disponibles [ici](https://kubernetes.io/docs/concepts/overview/kubernetes-api/).

## <a name="view-deployment-live-logs-preview"></a>Afficher des journaux dynamiques de déploiement (préversion)
Utilisez la procédure suivante pour afficher les journaux dynamiques pour des déploiements qui font partie de clusters AKS qui ne sont pas surveillés par Azure Monitor pour conteneurs. Si votre cluster utilise Azure Monitor pour conteneurs, utilisez le processus ci-dessous pour afficher les données actives pour des nœuds, contrôleurs, conteneurs et déploiements.

1. Dans le portail Azure, accédez au groupe de ressources du cluster AKS et sélectionnez votre ressource AKS.

2. Sélectionnez **Charges de travail** dans la section **Ressources Kubernetes** du menu.

3. Sélectionnez un déploiement sous l’onglet **Déploiements**.

4. Sélectionnez **Journaux dynamiques (préversion)** dans le menu du déploiement.

5. Sélectionnez un pod pour démarrer la collecte des données actives.

    [![Journaux dynamiques de déploiement](./media/container-insights-livedata-overview/live-data-deployment.png)](./media/container-insights-livedata-overview/live-data-deployment.png#lightbox)

## <a name="view-logs"></a>Afficher les journaux d’activité

Vous pouvez voir les données de journal en temps réel, à mesure qu’elles sont générées par le moteur de conteneur depuis la vue **Nœuds**, **Contrôleurs** et **Conteneurs**. Pour voir les données de journal, effectuez les étapes suivantes.

1. Dans le portail Azure, accédez au groupe de ressources du cluster AKS et sélectionnez votre ressource AKS.

2. Dans le tableau de bord du cluster AKS, sous **Supervision** à gauche, choisissez **Insights**.

3. Sélectionnez l’onglet **Nœuds**, **Contrôleurs** ou **Conteneurs**.

4. Sélectionnez un objet dans la grille de performances, puis dans le volet des propriétés situé sur le côté droit, sélectionnez l’option **Afficher les données actives (préversion)** . Si le cluster AKS est configuré pour l’authentification unique avec Azure AD, vous êtes invité à vous authentifier lors de la première utilisation sur cette session de navigateur. Sélectionnez votre compte et finalisez l’authentification auprès d’Azure.

    >[!NOTE]
    >Lorsque vous consultez les données depuis votre espace de travail Log Analytics en sélectionnant l’option **Afficher dans Analytics** du volet des propriétés, les résultats de la recherche dans les journaux peuvent éventuellement afficher des **Nœuds**, **Ensembles de démons**, **Jeux de réplicas**, **Travaux**, **Tâches Cron**, **Pods** et **Conteneurs** qui n’existent peut-être plus. Une tentative de recherche dans les journaux d’un conteneur qui n’est pas disponible dans `kubectl` échouera également ici. Pour en savoir plus sur la consultation des journaux, événements et métriques historiques, consultez la fonctionnalité [Voir dans Analytics](container-insights-log-search.md#search-logs-to-analyze-data).

Une fois l’authentification réussie, le volet de la console Live Data (préversion) apparaît sous la grille des données de performances, dans laquelle vous pouvez voir des données de journal en flux continu. Si l’indicateur d’état de récupération affiche une coche verte tout à droite du volet, cela signifie que des données sont récupérables et que leur envoi en streaming vers votre console commence.

![Option Voir les données du volet des propriétés du nœud](./media/container-insights-livedata-overview/node-properties-pane.png)

Le titre du volet indique le nom du pod avec lequel le conteneur est groupé.

## <a name="view-events"></a>Afficher les événements

Vous pouvez voir les données d’événement en temps réel, à mesure qu’elles sont générées par le moteur de conteneur depuis la vue **Nœuds**, **Contrôleurs**, **Conteneurs** et **Déploiements (préversion)** , lorsqu’un conteneur, pod, nœud, jeu de réplicas, ensemble de démons, travail, tâche Cron ou déploiement est sélectionné. Pour voir les événements, effectuez les étapes suivantes.

1. Dans le portail Azure, accédez au groupe de ressources du cluster AKS et sélectionnez votre ressource AKS.

2. Dans le tableau de bord du cluster AKS, sous **Supervision** à gauche, choisissez **Insights**.

3. Sélectionnez l’onglet **Nœuds**, **Contrôleurs**, **Conteneurs** ou **Déploiements (préversion)** .

4. Sélectionnez un objet dans la grille de performances, puis dans le volet des propriétés situé sur le côté droit, sélectionnez l’option **Afficher les données actives (préversion)** . Si le cluster AKS est configuré pour l’authentification unique avec Azure AD, vous êtes invité à vous authentifier lors de la première utilisation sur cette session de navigateur. Sélectionnez votre compte et finalisez l’authentification auprès d’Azure.

    >[!NOTE]
    >Lorsque vous consultez les données depuis votre espace de travail Log Analytics en sélectionnant l’option **Afficher dans Analytics** du volet des propriétés, les résultats de la recherche dans les journaux peuvent éventuellement afficher des **Nœuds**, **Ensembles de démons**, **Jeux de réplicas**, **Travaux**, **Tâches Cron**, **Pods** et **Conteneurs** qui n’existent peut-être plus. Une tentative de recherche dans les journaux d’un conteneur qui n’est pas disponible dans `kubectl` échouera également ici. Pour en savoir plus sur la consultation des journaux, événements et métriques historiques, consultez la fonctionnalité [Voir dans Analytics](container-insights-log-search.md#search-logs-to-analyze-data).

Une fois l’authentification réussie, le volet de la console Live Data (préversion) apparaît sous la grille des données de performances. Si l’indicateur d’état de récupération affiche une coche verte tout à droite du volet, cela signifie que des données sont récupérables et que leur envoi en streaming vers votre console commence.

Si l’objet que vous avez sélectionné était un conteneur, sélectionnez l’option **Événements** dans le volet. Si vous avez sélectionné un nœud, un pod ou un contrôleur, l’affichage des événements est automatiquement sélectionné.

![Événements d’affichage du volet Propriétés du contrôleur](./media/container-insights-livedata-overview/controller-properties-live-event.png)

Le titre du volet indique le nom du pod avec lequel le conteneur est groupé.

### <a name="filter-events"></a>Filtrer les événements

Lors de l’affichage d’événements, vous pouvez également limiter les résultats à l’aide du bouton **Filtrer** à droite de la barre de recherche. Selon la ressource que vous avez sélectionnée, un pod, un espace de noms ou un cluster sont répertoriés à partir desquels choisir.

## <a name="view-metrics"></a>Afficher les mesures

Vous pouvez voir les données de métrique en temps réel, à mesure qu’elles sont générées par le moteur de conteneur depuis la vue **Nœuds** ou **Contrôleurs**, uniquement quand un **Pod** est sélectionné. Pour voir les métriques, effectuez les étapes suivantes.

1. Dans le portail Azure, accédez au groupe de ressources du cluster AKS et sélectionnez votre ressource AKS.

2. Dans le tableau de bord du cluster AKS, sous **Supervision** à gauche, choisissez **Insights**.

3. Sélectionnez l’onglet **Nœuds** ou **Contrôleurs**.

4. Sélectionnez un objet **Pod** dans la grille de performances, puis dans le volet des propriétés situé sur le côté droit, sélectionnez l’option **Afficher les données actives (préversion)** . Si le cluster AKS est configuré pour l’authentification unique avec Azure AD, vous êtes invité à vous authentifier lors de la première utilisation sur cette session de navigateur. Sélectionnez votre compte et finalisez l’authentification auprès d’Azure.

    >[!NOTE]
    >Lorsque vous consultez les données depuis votre espace de travail Log Analytics en sélectionnant l’option **Afficher dans Analytics** du volet des propriétés, les résultats de la recherche dans les journaux peuvent éventuellement afficher des **Nœuds**, **Ensembles de démons**, **Jeux de réplicas**, **Travaux**, **Tâches Cron**, **Pods** et **Conteneurs** qui n’existent peut-être plus. Une tentative de recherche dans les journaux d’un conteneur qui n’est pas disponible dans `kubectl` échouera également ici. Pour en savoir plus sur la consultation des journaux, événements et métriques historiques, consultez la fonctionnalité [Voir dans Analytics](container-insights-log-search.md#search-logs-to-analyze-data).

Une fois l’authentification réussie, le volet de la console Live Data (préversion) apparaît sous la grille des données de performances. Les données de métriques sont récupérées, et leur envoi en streaming vers votre console commence pour leur présentation dans les deux graphiques. Le titre du volet indique le nom du pod avec lequel le conteneur est groupé.

![Exemple de métriques de l’affichage Pod](./media/container-insights-livedata-overview/pod-properties-live-metrics.png)

## <a name="using-live-data-views"></a>Utilisation de vues de données actives
Les sections suivantes décrivent les fonctionnalités que vous pouvez utiliser dans les différentes vues de données actives.

### <a name="search"></a>Rechercher
La fonctionnalité Live Data (préversion) comprend la fonctionnalité de recherche. Dans le champ **Rechercher**, vous pouvez filtrer les résultats en tapant un mot clé ou un terme ; tous les résultats correspondants sont mis en surbrillance pour permettre une consultation rapide. Lors de l’affichage d’événements, vous pouvez également limiter les résultats à l’aide du bouton **Filtrer** à droite de la barre de recherche. Selon la ressource que vous avez sélectionnée, un pod, un espace de noms ou un cluster sont répertoriés à partir desquels choisir.

![Exemple de filtre du volet de la console Live Data](./media/container-insights-livedata-overview/livedata-pane-filter-example.png)

![Exemple de filtre du volet de la console Données actives pour le déploiement](./media/container-insights-livedata-overview/live-data-deployment-search.png)

### <a name="scroll-lock-and-pause"></a>Arrêt défil et Pause

Pour suspendre le défilement automatique, et contrôler le comportement du volet en faisant défiler manuellement les nouvelles données lues, servez-vous de l’option **Faire défiler**. Pour réactiver le défilement automatique, sélectionnez de nouveau l’option **Faire défiler**. Vous pouvez aussi interrompre la récupération des données de journal ou d’événement en sélectionnant l’option **Suspendre**. Lorsque vous êtes prêt à reprendre la récupération, sélectionnez **Lire**.

![Vue en direct interrompue dans le volet de la console Live Data](./media/container-insights-livedata-overview/livedata-pane-scroll-pause-example.png)

![Affichage en direct en pause du volet de la console Données actives pour le déploiement](./media/container-insights-livedata-overview/live-data-deployment-pause.png)



>[!IMPORTANT]
>Nous recommandons uniquement de suspendre ou d’interrompre le défilement automatique pendant une brève période de temps, lors de la résolution d’un problème. Ces requêtes peuvent avoir un impact sur la disponibilité et la limitation de l’API Kubernetes sur votre cluster.

>[!IMPORTANT]
>Aucune donnée n’est stockée de façon permanente pendant l’exécution de cette fonctionnalité. Toutes les informations capturées lors de la session sont supprimées lorsque vous la quittez, ou que vous fermez votre navigateur. Les données restent uniquement présentes pour la visualisation dans le laps de temps de cinq minutes de la fonctionnalité Métriques ; toutes les mesures datant de plus de cinq minutes sont également supprimées. La mémoire tampon de Live Data (préversion) interroge dans des limites raisonnables de l’utilisation de la mémoire.

## <a name="next-steps"></a>Étapes suivantes

- Pour continuer à découvrir comment utiliser Azure Monitor et surveiller les autres aspects de votre cluster AKS, consultez [Connaître l’état d’Azure Kubernetes Service](container-insights-analyze.md).

- Consultez [Exemples de requêtes de journal](container-insights-log-search.md#search-logs-to-analyze-data) pour voir des requêtes et des exemples prédéfinis permettant de créer des alertes, des visualisations, ou d’effectuer une analyse plus poussée de vos clusters.
