---
title: Supervision de Kubernetes avec Azure Monitor pour conteneurs | Microsoft Docs
description: Cet article décrit comment voir et analyser les performances d’un cluster Kubernetes avec Azure Monitor pour conteneurs.
ms.topic: conceptual
ms.date: 03/26/2020
ms.openlocfilehash: a1f661089b3a6357abb3eed584401e6a8ae2e2fb
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96905704"
---
# <a name="monitor-your-kubernetes-cluster-performance-with-azure-monitor-for-containers"></a>Superviser les performances de votre cluster Kubernetes avec Azure Monitor pour conteneurs

Avec Azure Monitor pour les conteneurs, vous pouvez utiliser les graphiques de performances et l’état d’intégrité pour surveiller la charge de travail de vos clusters Kubernetes hébergés sur Azure Kubernetes Service (AKS), Azure Stack ou un autre environnement selon deux perspectives. La surveillance peut s’effectuer directement à partir du cluster ou vous pouvez afficher tous les clusters d’un abonnement à partir d’Azure Monitor. L’affichage d’Azure Container Instances est également possible lors de la surveillance d’un cluster AKS spécifique.

Cet article vous aidera à comprendre les deux perspectives et comment Azure Monitor vous aide à évaluer, examiner et résoudre rapidement les problèmes détectés.

Pour plus d’informations sur la procédure d’activation d’Azure Monitor pour les conteneurs, consultez [Onboard Azure Monitor for containers](container-insights-onboard.md) (Intégrer Azure Monitor pour les conteneurs).

Azure Monitor fournit une vue multi-cluster qui affiche l’état d’intégrité de tous les clusters Kubernetes surveillés exécutant Linux et Windows Server 2019, déployés dans les groupes de ressources de vos abonnements. Il montre les clusters découverts sur tous les environnements et qui ne sont pas surveillés par la solution. Vous pouvez immédiatement comprendre l’intégrité du cluster et, à partir de là, vous pouvez explorer le nœud et la page de performances du contrôleur, ou naviguer pour voir les graphiques de performances du cluster. Pour les clusters AKS qui ont été détectés et identifiés comme non surveillés, vous pouvez activer leur surveillance à tout moment.

Les principales différences entre la surveillance d’un cluster Windows Server doté d’Azure Monitor pour conteneurs et celle d’un cluster Linux sont décrites [ici](container-insights-overview.md#what-does-azure-monitor-for-containers-provide) dans l’article de présentation.

## <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

Connectez-vous au [portail Azure](https://portal.azure.com).

## <a name="multi-cluster-view-from-azure-monitor"></a>Vue multicluster à partir d’Azure Monitor

Pour afficher l’état d’intégrité de tous les clusters Kubernetes déployés, sélectionnez **Surveiller** dans le volet de gauche du Portail Azure. Sous la section **Insights**, sélectionnez **Conteneurs**.

![Exemple de tableau de bord multicluster dans Azure Monitor](./media/container-insights-analyze/azmon-containers-multiview.png)

Vous pouvez étendre les résultats présentés dans la grille pour afficher les clusters suivants :

* **Azure** - Clusters AKS et AKS-Engine hébergés dans Azure Kubernetes Service
* **Azure Stack (préversion)** - Clusters AKS-Engine hébergés sur Azure Stack
* **Non-Azure (préversion)** - Clusters Kubernetes hébergés localement
* **Tous** - Affichez tous les clusters Kubernetes hébergés dans des environnements Azure, Azure Stack et locaux intégrés à Azure Monitor pour les conteneurs

Pour afficher les clusters d’un environnement spécifique, sélectionnez-les dans la pastille **Environnements** dans le coin supérieur gauche de la page.

![Exemple de sélecteur de pastille d’environnement](./media/container-insights-analyze/clusters-multiview-environment-pill.png)

Dans l’onglet **Monitored clusters** (Clusters surveillés), vous découvrez les informations suivantes :

- Combien de clusters se trouvent dans un état critique ou non intègre, et combien sont sains ou ne rapportent aucune donnée (état inconnu).
- Si tous les déploiements [Azure Kubernetes Engine (AKS-engine)](https://github.com/Azure/aks-engine) sont intègres.
- Combien de nœuds et de pods utilisateur et système sont déployés par cluster.
- Quelle quantité d’espace disque est disponible et s’il y a un problème de capacité.

Les états d’intégrité inclus sont :

* **Sain** : Aucun problème détecté pour la machine virtuelle et le fonctionnement répond aux attentes.
* **Critique** : Un ou plusieurs problèmes critiques ont été détectés et doivent être corrigés afin de restaurer l’état opérationnel normal attendu.
* **Avertissement** : Un ou plusieurs problèmes ont été détectés et doivent être corrigés pour éviter que la condition d’intégrité ne devienne critique.
* **Inconnue** : Si le service n’a pas pu établir de connexion avec le nœud ou le pod, l’état passe à Inconnu.
* **Introuvable** : L’espace de travail, le groupe de ressources ou l’abonnement contenant l’espace de travail pour cette solution a été supprimé.
* **Non autorisé** : L’utilisateur ne dispose pas des autorisations requises pour lire les données dans l’espace de travail.
* **Erreur** : Une erreur s’est produite lors de la tentative de lecture à partir de l’espace de travail.
* **Mal configuré** : Azure Monitor pour les conteneurs n’a pas été configuré correctement dans l’espace de travail spécifié.
* **Aucune donnée** : Aucune donnée n’a été signalée à l’espace de travail au cours des 30 dernières minutes.

L’état d’intégrité calcule l’état global des clusters en fonction du *pire* des trois états à une exception près. Si l’un des trois états est inconnu, l’état global des clusters affiche **Inconnu**.

Le tableau suivant fournit une répartition du calcul pour le contrôle des états d’intégrité pour un cluster surveillé sur la vue multicluster.

| Cluster supervisé |Statut |Disponibilité |
|-------|-------|-----------------|
|**Pod utilisateur**| | |
| |Healthy |100 % |
| |Avertissement |90 - 99 % |
| |Critique |< 90 % |
| |Unknown |Si non signalé dans les 30 dernières minutes |
|**Pod système**| | |
| |Healthy |100 % |
| |Avertissement |N/A |
| |Critique |< 100 % |
| |Unknown |Si non signalé dans les 30 dernières minutes |
|**Nœud** | | |
| |Healthy |> 85 % |
| |Avertissement |60 - 84 % |
| |Critique |< 60 % |
| |Unknown |Si non signalé dans les 30 dernières minutes |

À partir de la liste des clusters, vous pouvez descendre à la page **Cluster** en sélectionnant le nom du cluster. Accédez ensuite à la page de performances des **Nœuds** en sélectionnant le cumul des nœuds dans la colonne **Nœuds** pour ce cluster en particulier. Vous pouvez également descendre dans la hiérarchie jusqu’à la page de performances des **Contrôleurs** en sélectionnant le cumul de la colonne **Pods utilisateur** ou **Pods système**.

## <a name="view-performance-directly-from-a-cluster"></a>Afficher les performances directement à partir d’un cluster

L’accès à Azure Monitor pour les conteneurs est disponible directement à partir d’un cluster AKS en sélectionnant **Insights** > **Cluster** dans le volet gauche, ou en sélectionnant un cluster depuis l’affichage multi-cluster. Les informations sur votre cluster se présentent sous quatre perspectives :

- Cluster
- Nœuds
- Controllers
- Containers

>[!NOTE]
>L’expérience décrite dans le reste de cet article s’applique également à l’affichage de l’état des performances et de l’intégrité de vos clusters Kubernetes hébergés sur Azure Stack ou un autre environnement lorsqu’ils sont sélectionnés à partir de l’affichage multi-cluster.

La page par défaut s’ouvre et affiche quatre graphiques en courbes des performances qui affichent les principales mesures de performance de votre cluster.

![Exemples de graphiques de performances sous l’onglet Cluster](./media/container-insights-analyze/containers-cluster-perfview.png)

Les graphiques de performances affichent quatre métriques de performance :

- **Utilisation de l’UC du nœud&nbsp;%**  : vue agrégée de l’utilisation du processeur pour le cluster entier. Pour filtrer les résultats pour l’intervalle de temps, sélectionnez **Avg** (Moy), **Min**, **50th** (50e), **90th** (90e), **95th** (95e) ou **Max** dans le sélecteur de centiles au-dessus du graphique. Les filtres peuvent être utilisés individuellement ou combinés.
- **Utilisation de mémoire du nœud&nbsp;%** : vue agrégée de l’utilisation de la mémoire pour le cluster entier. Pour filtrer les résultats pour l’intervalle de temps, sélectionnez **Avg** (Moy), **Min**, **50th** (50e), **90th** (90e), **95th** (95e) ou **Max** dans le sélecteur de centiles au-dessus du graphique. Les filtres peuvent être utilisés individuellement ou combinés.
- **Nombre de nœuds** : nombre et état des nœuds fournis par Kubernetes. L’état des nœuds de cluster représentés est Total, Ready (Prêts) ou Not Ready (Non prêts). Il est possible de les filtrer individuellement ou en les combinant à l’aide du sélecteur situé au-dessus du graphique.
- **Nombre de pods actifs** : nombre et état des pods fournis par Kubernetes. L’état des pods représentés est Total, Pending (En attente), Running (En cours d’exécution), Unknown (Inconnus), Succeeded (Réussite) ou Failed (Échec). Il est possible de les filtrer individuellement ou en les combinant à l’aide du sélecteur situé au-dessus du graphique.

Utilisez les touches de direction gauche et droite pour parcourir chaque point de données sur le graphe. Utilisez les touches de direction haut et bas pour faire défiler les lignes de centile. Sélectionnez l’icône d’épingle dans le coin supérieur droit de l’un des graphiques pour épingler le graphique sélectionné au dernier tableau de bord Azure que vous avez consulté. À partir du tableau de bord, vous pouvez redimensionner et repositionner le graphique. En sélectionnant le graphique depuis le tableau de bord, vous êtes redirigé vers Azure Monitor pour conteneurs et chargez l’étendue et la vue appropriées.

Azure Monitor pour conteneurs prend également en charge Azure Monitor [Metrics Explorer](../platform/metrics-getting-started.md), où vous pouvez tracer vos propres graphiques, mettre en corrélation et examiner les tendances, et épingler aux tableaux de bord. Dans Metrics Explorer, vous pouvez également utiliser les critères que vous avez définis pour visualiser vos métriques en tant que base d’une [règle d’alerte basée sur une métrique](../platform/alerts-metric.md).

## <a name="view-container-metrics-in-metrics-explorer"></a>Afficher les métriques du conteneur dans Metrics Explorer

Dans Metrics Explorer, vous pouvez afficher les métriques agrégées d’utilisation de pod et de nœud d’Azure Monitor pour conteneurs. Le tableau suivant récapitule les détails pour vous aider à comprendre comment utiliser les graphiques de métrique pour visualiser les métriques des conteneurs.

|Espace de noms | Métrique | Description |
|----------|--------|-------------|
| insights.container/nodes | |
| | cpuUsageMillicores | Mesure agrégée de l’utilisation du processeur sur le cluster. Il s’agit d’un cœur de processeur divisé en 1 000 unités (millisecondes = 1000). Utilisé pour déterminer l’utilisation des cœurs dans un conteneur où de nombreuses applications peuvent utiliser un seul cœur.|
| | cpuUsagePercentage | Utilisation du processeur moyenne agrégée, mesurée en pourcentage sur le cluster.|
| | memoryRssBytes | Mémoire RSS du conteneur utilisée, en octets.|
| | memoryRssPercentage | Mémoire RSS du conteneur utilisée, en pourcentage.|
| | memoryWorkingSetBytes | Mémoire de plage de travail du conteneur utilisée.|
| | memoryWorkingSetPercentage | Mémoire de plage de travail du conteneur utilisée, en pourcentage. |
| | nodesCount | Nombre de nœuds fournis par Kubernetes.|
| insights.container/pods | |
| | PodCount | Nombre de pods fournis par Kubernetes.|

Vous pouvez [fractionner](../platform/metrics-charts.md#apply-splitting-to-a-chart) une métrique pour l’afficher par dimension et comparer différents segments de celui-ci. Pour un nœud, vous pouvez segmenter le graphique par la dimension *hôte*. À partir d’un pod, vous pouvez le segmenter par les dimensions suivantes :

* Contrôleur
* Espace de noms Kubernetes
* Nœud
* Phase

## <a name="analyze-nodes-controllers-and-container-health"></a>Analyser l’intégrité des conteneurs, des contrôleurs et des nœuds

Quand vous passez à l’onglet **Nodes** (Nœuds), **Controllers** (Contrôleurs) et **Containers** (Conteneurs), un volet des propriétés s’affiche automatiquement à droite de la page. Il présente les propriétés de l’élément sélectionné, notamment les étiquettes que vous définissez pour organiser les objets Kubernetes. Lorsqu’un nœud Linux est sélectionné, la section **Capacité du disque Local** qui représente également l’espace disque disponible et le pourcentage utilisé pour chaque disque présenté au nœud. Sélectionnez le lien **>>** dans le volet pour afficher ou masquer celui-ci.

Quand vous développez les objets dans la hiérarchie, le volet des propriétés est mis à jour en fonction de l’objet sélectionné. Depuis le volet, vous pouvez également afficher les journaux de conteneur Kubernetes (stdout/stderror), les événements et les mesures pod en sélectionnant le lien **Afficher les données actives (préversion)** en haut du volet. Pour plus d’informations sur la configuration requise pour accorder et contrôler l’accès à l’affichage de ces données, consultez [Setup the Live Data (preview)](container-insights-livedata-setup.md) (Configurer les données actives [préversion]). Lorsque vous consultez des ressources de cluster, vous pouvez voir ces informations en temps réel. Pour plus d’informations sur cette fonctionnalité, consultez [How to view Kubernetes logs, events, and pod metrics in real time](container-insights-livedata-overview.md) (Comment afficher les journaux, les événements et les mesures pod Kubernetes en temps réel). Pour afficher les données de journal Kubernetes stockées dans votre espace de travail en fonction de recherches de journal prédéfinies, sélectionnez **Afficher les journaux d’activité du conteneur** dans la liste déroulante **Afficher dans Analytics**. Pour plus d’informations sur ce sujet, consultez [Rechercher des journaux d’activité pour analyser les données](container-insights-log-search.md#search-logs-to-analyze-data).

Utilisez l’option **+ Ajouter un filtre** en haut de la page pour filtrer les résultats de la vue par **Service**, **Nœud** ou **Espace de noms** ou **Pool de nœuds**. Après avoir sélectionné l’étendue de filtre, sélectionnez l’une des valeurs affichées dans le champ **Select value(s)** (Sélectionner une ou plusieurs valeurs). Une fois que le filtre est configuré, il est appliqué globalement lors de l’affichage de n’importe quel point de vue du cluster AKS. La formule prend uniquement en charge le signe égal. Vous pouvez ajouter des filtres supplémentaires au-dessus du premier pour affiner davantage vos résultats. Par exemple, si vous spécifiez un filtre par **Nœud**, vous pouvez uniquement sélectionner **Service** ou **Espace de noms** pour le deuxième filtre.

La spécification d’un filtre dans un onglet continue d’être appliquée quand vous en sélectionnez un autre. Il est supprimé lorsque vous cliquez sur le symbole **x** en regard du filtre spécifié.

Sous l’onglet **Nodes** (Nœuds), la hiérarchie de ligne suit le modèle d’objet Kubernetes qui commence par un nœud dans votre cluster. Développez le nœud pour voir les pods qui y sont actuellement exécutés. Si plusieurs conteneurs sont regroupés dans un pod, ils sont affichés comme la dernière ligne de la hiérarchie. Vous pouvez également voir le nombre de charges de travail non associées à un pod qui sont actuellement exécutées sur l’ordinateur hôte si le processeur ou la mémoire de l’hôte sont très sollicités.

![Exemple de hiérarchie de nœud Kubernetes dans l’affichage des performances](./media/container-insights-analyze/containers-nodes-view.png)

Les conteneurs Windows Server qui exécutent le système d’exploitation Windows Server 2019 sont affichés après tous les nœuds basés sur Linux dans la liste. Lorsque vous développez un nœud Windows Server, vous pouvez afficher un ou plusieurs pods et conteneurs en cours d’exécution sur le nœud. Une fois qu’un nœud est sélectionné, le volet Propriétés affiche les informations de version.

![Exemple de hiérarchie de nœud avec les nœuds Windows Server répertoriés](./media/container-insights-analyze/nodes-view-windows.png)

Les nœuds virtuels Azure Container Instances qui exécutent le système d’exploitation Linux sont affichés après le dernier nœud de cluster AKS sur la liste. Lorsque vous développez un nœud Container Instances, vous pouvez afficher un ou plusieurs pods et conteneurs Container Instances en cours d’exécution sur le nœud. Les métriques ne sont pas collectées et signalées pour les nœuds, uniquement pour les pods.

![Exemple de hiérarchie de nœud avec les instances de conteneur répertoriées](./media/container-insights-analyze/nodes-view-aci.png)

À partir d’un nœud développé, vous pouvez explorer le pod ou le conteneur qui s’exécute sur le nœud jusqu’au contrôleur pour afficher les données de performances filtrées pour ce contrôleur. Sélectionnez la valeur sous la colonne **Contrôleur** du nœud spécifique.

![Capture d’écran montrant le nœud jusqu’au contrôleur dans l’affichage des performances](./media/container-insights-analyze/drill-down-node-controller.png)

Sélectionnez des contrôleurs ou des conteneurs en haut de la page pour examiner l’état et l’utilisation des ressources pour ces objets. Pour afficher l’utilisation de la mémoire, dans la liste déroulante **Metric** (Métrique), sélectionnez **Memory RSS** (Mémoire RSS) ou **Memory working set** (Plage de travail de la mémoire). L’option **Memory RSS** (Mémoire RSS) est uniquement prise en charge par Kubernetes 1.8 et versions ultérieures. Sinon, vous pouvez afficher les valeurs **Min&nbsp;%** sous la forme *NaN&nbsp;%* , qui est une valeur de type données numérique représentant une valeur non définie ou non représentable.

![Affichage des performances des nœuds d’un conteneur](./media/container-insights-analyze/containers-node-metric-dropdown.png)

La **plage de travail de la mémoire** montre la mémoire résidente et la mémoire virtuelle (cache) incluses, et représente le total de ce que l’application utilise. **RSS mémoire** montre uniquement la mémoire principale (qui, en d’autres termes, n’est rien d’autre que la mémoire résidente). Cette métrique montre la capacité réelle de la mémoire disponible. Quelle est la différence entre la mémoire résidente et la mémoire virtuelle ?

- La mémoire résidente, ou mémoire principale, correspond à la quantité réelle de mémoire disponible pour les nœuds du cluster.

- La mémoire virtuelle correspond à l’espace disque réservé (cache) utilisé par le système d’exploitation pour échanger des données de la mémoire sur le disque en cas de sollicitation de la mémoire, puis les replacer en mémoire si nécessaire.

Par défaut, les données de performances sont basées sur les six dernières heures, mais vous pouvez modifier la fenêtre à l’aide de l’option **Plage horaire** située en haut à gauche. Vous pouvez également filtrer les résultats dans l’intervalle de temps en sélectionnant les options du sélecteur de centiles **Min**, **Avg** (Moy), **50th** (50e), **90th** (90e), **95th** (95e) et **Max**.

![Sélection de centile pour le filtrage des données](./media/container-insights-analyze/containers-metric-percentile-filter.png)

Lorsque vous pointez sur le graphique à barres sous la colonne **Tendance**, chaque barre indique l’utilisation du processeur ou de la mémoire, selon la métrique sélectionnée, au sein d’une période d’échantillonnage de 15 minutes. Après avoir sélectionné le graphique de tendances à l’aide d’un clavier, utilisez la combinaison Alt + Pg. préc ou Alt + Pg. suiv pour parcourir chaque barre individuellement. Vous obtenez les mêmes détails, comme vous le feriez en pointant sur la barre.

![Exemple de pointage sur le graphique à barres](./media/container-insights-analyze/containers-metric-trend-bar-01.png)

Dans l’exemple suivant, pour le premier nœud de la liste *aks-nodepool1-* , la valeur de **Containers** (Conteneurs) est 9. Il s’agit du cumul du nombre de conteneurs déployés.

![Cumul des conteneurs pour l’exemple de nœud](./media/container-insights-analyze/containers-nodes-containerstotal.png)

Cette information peut vous aider à déterminer rapidement si le nombre de conteneurs est bien réparti entre les nœuds de votre cluster.

Les informations qui sont affichées dans l’onglet **Nœuds** sont décrites dans le tableau suivant.

| Colonne | Description |
|--------|-------------|
| Nom | Nom de l’hôte. |
| Statut | Vue Kubernetes de l’état du nœud. |
| Min&nbsp;%, Avg&nbsp;% (Moy %), 50th&nbsp;% (50e %), 90th&nbsp;% (90e %), 95th&nbsp;% (95e %), Max&nbsp;%  | Pourcentage moyen du nœud basé sur le centile durant la période spécifiée. |
| Min, Avg (Moy), 50th (50e), 90th (90e), 95th (95e), Max | Valeur réelle moyenne des nœuds basée sur le centile durant la période spécifiée. La valeur moyenne est mesurée à partir de la limite de processeur/mémoire définie pour un nœud. Pour les pods et les conteneurs, il s’agit de la valeur moyenne indiquée par l’hôte. |
| Containers | Nombre de conteneurs. |
| Uptime | Indique le temps écoulé depuis le démarrage ou le redémarrage d’un nœud. |
| Contrôleur | Uniquement pour les conteneurs et les pods. Indique le contrôleur dans lequel l’élément réside. Les pods ne figurent pas tous dans un contrôleur, certains peuvent donc afficher la mention **N/A** (non applicable). |
| Tendance Min&nbsp;%, Avg&nbsp;% (Moy %), 50th&nbsp;% (50e %), 90th&nbsp;% (90e %), 95th&nbsp;% (95e %), Max&nbsp;% | La tendance du graphique à barres indique la métrique de centile en pourcentage du contrôleur. |

Vous remarquerez peut-être une charge de travail après le développement d’un nœud nommé **Autre processus**. Il représente les processus non placés dans des conteneurs qui s’exécutent sur votre nœud, et comprend les éléments suivants :

* Processus autonomes ou gérés par Kubernetes non placés dans des conteneurs

* Processus runtime de conteneur

* Kubelet

* Processus système en cours d’exécution sur votre nœud

* Autres charges de travail hors Kubernetes en cours d’exécution sur le matériel ou la machine virtuelle du nœud

Le calcul est effectué comme suit : *Utilisation totale à partir de CAdvisor* - *Utilisation à partir du processus en conteneur*.

Dans le sélecteur, sélectionnez **Controllers** (Contrôleurs).

![Sélectionner l’affichage des contrôleurs](./media/container-insights-analyze/containers-controllers-tab.png)

Ici, vous pouvez afficher l’intégrité des performances de vos contrôleurs ainsi que les contrôleurs de nœuds virtuels Container Instances ou les pods de nœuds virtuels non connectés à un contrôleur.

![Vue des performances des contrôleurs de \<Nom>](./media/container-insights-analyze/containers-controllers-view.png)

La hiérarchie de ligne commence par un contrôleur. Développer un contrôleur permet d’afficher un ou plusieurs pods. Développez un pod : vous voyez que la dernière ligne affiche le conteneur lié à un pod. À partir d’un contrôleur développé, vous pouvez explorer jusqu’au nœud en cours d’exécution pour afficher les données de performances filtrées pour ce nœud. Les pods Container Instances non connectés à un contrôleur sont répertoriés à la fin de la liste.

![Exemple de hiérarchie de contrôleurs avec les pods d’instances de conteneur répertoriés](./media/container-insights-analyze/controllers-view-aci.png)

Sélectionnez la valeur sous la colonne **Nœud** du contrôleur spécifique.

![Exemple de zoom jusqu’aux nœuds du contrôleur dans la vue des performances](./media/container-insights-analyze/drill-down-controller-node.png)

Les informations qui sont affichées dans les contrôleurs sont décrites dans le tableau suivant.

| Colonne | Description |
|--------|-------------|
| Nom | Nom du contrôleur.|
| Statut | Statut du cumul des conteneurs lorsque l’exécution est terminée, par exemple, *OK*, *Terminated* (Terminé), *Failed* (Échec), *Stopped* (Arrêt) ou *Paused* (Pause). Si le conteneur est en cours d’exécution, mais que l’état n’est pas correctement affiché, ou n’a pas été identifié par l’agent et n’a pas répondu dans un délai de 30 minutes, l’état est *Unknown* (Inconnu). Des détails supplémentaires sur l’icône d’état sont fournis dans le tableau suivant.|
| Min&nbsp;%, Avg&nbsp;% (Moy %), 50th&nbsp;% (50e %), 90th&nbsp;% (90e %), 95th&nbsp;% (95e %), Max&nbsp;%| Moyenne cumulée du pourcentage moyen de chaque entité pour la métrique et le centile sélectionnés. |
| Min, Avg (Moy), 50th (50e), 90th (90e), 95th (95e), Max  | Cumul de la performance moyenne du processeur ou de la mémoire du conteneur pour le centile sélectionné. La valeur moyenne est mesurée à partir de la limite Processeur/Mémoire définie pour un pod. |
| Containers | Nombre total de conteneurs pour le contrôleur ou pod. |
| Restarts | Cumul du nombre de redémarrages à partir des conteneurs. |
| Uptime | Indique le temps écoulé depuis le démarrage d’un conteneur. |
| Nœud | Uniquement pour les conteneurs et les pods. Indique le contrôleur dans lequel l’élément réside. |
| Tendance Min&nbsp;%, Avg&nbsp;% (Moy %), 50th&nbsp;% (50e %), 90th&nbsp;% (90e %), 95th&nbsp;% (95e %), Max&nbsp;% | La tendance du graphique à barres indique la métrique de centile du contrôleur. |

Les icônes dans le champ d’état indiquent le statut en ligne des conteneurs.

| Icône | Statut |
|--------|-------------|
| ![Icône de statut en cours d’exécution prêt](./media/container-insights-analyze/containers-ready-icon.png) | Running (Ready) (En cours d’exécution (Prêt))|
| ![Icône de statut en attente ou en pause](./media/container-insights-analyze/containers-waiting-icon.png) | Waiting or Paused (En attente ou en pause)|
| ![Icône de statut de dernière exécution signalée](./media/container-insights-analyze/containers-grey-icon.png) | Last reported running but hasn’t responded more than 30 minutes (Dernière exécution signalée, mais pas de réponse depuis plus de 30 minutes)|
| ![Icône d’état de réussite](./media/container-insights-analyze/containers-green-icon.png) | Successfully stopped or failed to stop (Réussite ou échec de l’arrêt)|

L’icône d’état affiche un nombre basé sur ce que fournit le pod. Elle montre les deux pires états. Lorsque vous pointez sur un état, elle affiche l’état cumulé de tous les pods du conteneur. Si l’état n’est pas Prêt, il affiche **(0)** .

Dans le sélecteur, sélectionnez **Containers** (Conteneurs).

![Sélectionner l’affichage des conteneurs](./media/container-insights-analyze/containers-containers-tab.png)

Ici, vous pouvez voir l’intégrité des performances de vos conteneurs Azure Kubernetes et Azure Container Instances.

![Vue des performances des conteneurs de \<Nom>](./media/container-insights-analyze/containers-containers-view.png)

À partir d’un conteneur, vous pouvez accéder à un pod ou un nœud pour afficher les données de performance filtrées pour cet objet. Sélectionnez la valeur sous la colonne **Pod** ou **Node** pour le conteneur.

![Exemple de zoom depuis le nœud jusqu’aux conteneurs dans l’affichage des performances](./media/container-insights-analyze/drill-down-controller-node.png)

Les informations qui sont affichées dans les conteneurs sont décrites dans le tableau suivant.

| Colonne | Description |
|--------|-------------|
| Nom | Nom du contrôleur.|
| Statut | Statut des conteneurs, le cas échéant. Des détails supplémentaires sur l’icône d’état sont fournis dans le tableau ci-dessous.|
| Min&nbsp;%, Avg&nbsp;% (Moy %), 50th&nbsp;% (50e %), 90th&nbsp;% (90e %), 95th&nbsp;% (95e %), Max&nbsp;% | Cumul du pourcentage moyen de chaque entité pour la métrique et le centile sélectionnés. |
| Min, Avg (Moy), 50th (50e), 90th (90e), 95th (95e), Max | Cumul de la performance moyenne du processeur ou de la mémoire du conteneur pour le centile sélectionné. La valeur moyenne est mesurée à partir de la limite Processeur/Mémoire définie pour un pod. |
| Pod | Conteneur dans lequel se trouve le pod.|
| Nœud |  Nœud sur lequel se trouve le conteneur. |
| Restarts | Indique le temps écoulé depuis le démarrage d’un conteneur. |
| Uptime | Indique le temps écoulé depuis le démarrage ou le redémarrage d’un conteneur. |
| Tendance Min&nbsp;%, Avg&nbsp;% (Moy %), 50th&nbsp;% (50e %), 90th&nbsp;% (90e %), 95th&nbsp;% (95e %), Max&nbsp;% | La tendance du graphique à barres indique la métrique de centile en pourcentage du conteneur. |

Les icônes du champ d’état indiquent les états en ligne des pods, comme expliqué dans le tableau suivant.

| Icône | Statut |
|--------|-------------|
| ![Icône de statut en cours d’exécution prêt](./media/container-insights-analyze/containers-ready-icon.png) | Running (Ready) (En cours d’exécution (Prêt))|
| ![Icône de statut en attente ou en pause](./media/container-insights-analyze/containers-waiting-icon.png) | Waiting or Paused (En attente ou en pause)|
| ![Icône de statut de dernière exécution signalée](./media/container-insights-analyze/containers-grey-icon.png) | Last reported running but hasn’t responded more than 30 minutes (Dernière exécution signalée, mais pas de réponse depuis plus de 30 minutes)|
| ![Icône de statut Terminé](./media/container-insights-analyze/containers-terminated-icon.png) | Successfully stopped or failed to stop (Réussite ou échec de l’arrêt)|
| ![Icône de statut Échec](./media/container-insights-analyze/containers-failed-icon.png) | Statut Échec |

## <a name="monitor-and-visualize-network-configurations"></a>Analyser et visualiser les configurations réseau
Azure Network Policy Manager comprend des métriques Prometheus informatives qui vous permettent d'analyser et de mieux comprendre vos configurations réseau. Il fournit des visualisations intégrées dans le portail Azure ou les laboratoires Grafana. Pour plus d'informations, consultez [Analyser et visualiser les configurations réseau avec Azure NPM](../../virtual-network/kubernetes-network-policies.md#monitor-and-visualize-network-configurations-with-azure-npm).


## <a name="workbooks"></a>Workbooks

Les classeurs regroupent du texte, des requêtes de journal, des métriques et des paramètres au sein de rapports interactifs complets qui vous permettent d’analyser les performances des clusters. Si vous souhaitez obtenir une description des classeurs Azure Monitor pour conteneurs, consultez [Classeurs Azure Monitor pour conteneurs](container-insights-reports.md).


## <a name="next-steps"></a>Étapes suivantes

- Consultez [Créer des alertes de performances avec Azure Monitor pour conteneurs](./container-insights-log-alerts.md) pour découvrir comment créer des alertes pour une utilisation élevée du processeur et de la mémoire, afin de prendre en charge vos procédures et processus opérationnels ou DevOps.

- Consultez les [exemples de requêtes de journal](container-insights-log-search.md#search-logs-to-analyze-data) pour voir les requêtes prédéfinies et des exemples permettant d’évaluer ou de personnaliser la génération d’alertes, la visualisation ou l’analyse de vos clusters.

- Consultez [Superviser l’intégrité des clusters](./container-insights-overview.md) pour en savoir plus sur l’affichage de l’état d’intégrité de votre cluster Kubernetes.