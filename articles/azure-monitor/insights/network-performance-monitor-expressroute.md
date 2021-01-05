---
title: Solution Analyseur de performances réseau dans Azure Log Analytics | Microsoft Docs
description: Utilisez la fonctionnalité Moniteur ExpressRoute de Network Performance Monitor afin de surveiller les performances et la connectivité de bout en bout entre vos succursales et Azure, via Azure ExpressRoute.
ms.subservice: logs
ms.topic: conceptual
author: abshamsft
ms.author: absha
ms.date: 11/27/2018
ms.openlocfilehash: a9f268e5080a5c04c5cdb4767f2db5fd2d77cd3c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87326152"
---
# <a name="expressroute-monitor"></a>Moniteur ExpressRoute

Vous pouvez utiliser la fonctionnalité Moniteur Azure ExpressRoute de [Network Performance Monitor](network-performance-monitor.md) pour surveiller les performances et la connectivité de bout en bout entre vos succursales et Azure, via Azure ExpressRoute. Les avantages clés sont les suivants : 

- Détection automatique des circuits ExpressRoute associés à votre abonnement.
- Suivi de l’utilisation de la bande passante, perte et latence au niveau du circuit, peering et niveau du réseau virtuel Azure d’ExpressRoute.
- Découverte de la topologie réseau de vos circuits ExpressRoute.

![Moniteur ExpressRoute](media/network-performance-monitor-expressroute/expressroute-intro.png)

## <a name="configuration"></a>Configuration 
Pour ouvrir la configuration de Network Performance Monitor, ouvrez la [solution Network Performance Monitor](network-performance-monitor.md) et sélectionnez **Configurer**.

### <a name="configure-network-security-group-rules"></a>Configurer les règles du groupe de sécurité réseau 
Pour les serveurs se trouvant dans Azure utilisés pour la surveillance via Network Performance Monitor, configurez les règles du groupe de sécurité réseau (NSG) pour autoriser le trafic TCP sur un port utilisé par Network Performance Monitor pour les transactions synthétiques. Par défaut, il s’agit du port 8084. Cette configuration permet à l’agent Log Analytics installé sur les machines virtuelles Azure de communiquer avec un agent de surveillance local. 

Pour plus d’informations sur les Groupes de sécurité réseau, voir  [Groupes de sécurité réseau](../../virtual-network/manage-network-security-group.md). 

>[!NOTE]
> Avant de poursuivre, installez l’agent de serveur local et l’agent de serveur Azure, et exécutez le script PowerShell EnableRules.ps1. 

 
### <a name="discover-expressroute-peering-connections"></a>Détecter les connexions de peering ExpressRoute 
 
1. Sélectionnez la vue **Peerings ExpressRoute**.
2. Sélectionnez **Découvrir maintenant** pour découvrir tous les peerings privés ExpressRoute connectés aux réseaux virtuels dans l’abonnement Azure lié à cet espace de travail Azure Log Analytics.

    >[!NOTE]
    > La solution ne découvre pour le moment que les peerings privés ExpressRoute. 

    >[!NOTE]
    > Ces peerings privés, connectés aux réseaux virtuels associés à l’abonnement lié à cet espace de travail Log Analytics, sont les seuls à être découverts. Si ExpressRoute est connecté aux réseaux virtuels en dehors de l’abonnement lié à cet espace de travail, créez un espace de travail Log Analytics dans ces abonnements. Utilisez ensuite Network Performance Monitor pour surveiller ces peerings. 

    ![Configuration du moniteur ExpressRoute](media/network-performance-monitor-expressroute/expressroute-configure.png)
 
   Une fois la détection terminée, les connexions de peerings privés sont répertoriées dans une table. L’analyse de ces peerings est initialement à l’état désactivée. 

### <a name="enable-monitoring-of-the-expressroute-peering-connections"></a>Activer la surveillance des connexions de peering ExpressRoute 

1. Sélectionnez la connexion de peering privé à surveiller.
2. Dans le volet de droite, cochez la case **Surveiller ce peering**. 
3. Si vous envisagez de créer des événements d’intégrité pour cette connexion, sélectionnez **Activer le monitoring d’intégrité pour ce peering**. 
4. Choisissez les conditions d’analyse. Vous pouvez définir des seuils personnalisés pour la génération d’événements d’intégrité en entrant des valeurs de seuil. Chaque fois que la valeur d’une condition dépasse son seuil sélectionné pour la connexion de peering, un événement d’intégrité est généré. 
5. Sélectionnez **Ajouter des agents** pour choisir les agents de surveillance que vous souhaitez utiliser pour surveiller cette connexion de peering. Assurez-vous que vous ajoutez des agents aux deux extrémités de la connexion. Vous avez besoin d’au moins un agent dans le réseau virtuel connecté à ce peering. Vous avez également besoin d’au moins un agent local connecté à ce peering. 
6. Sélectionnez **Enregistrer** pour enregistrer la configuration. 

   ![Configuration de la surveillance ExpressRoute](media/network-performance-monitor-expressroute/expressroute-configure-discovery.png)


Après avoir activé les règles et sélectionné les valeurs et les agents, patientez 30 à 60 minutes pour que les valeurs soient renseignées et que les vignettes de **surveillance ExpressRoute** s’affichent. Lorsque vous voyez les vignettes de surveillance, vos circuits ExpressRoute et ressources de connexion sont surveillés par Network Performance Monitor. 

>[!NOTE]
> Cette fonctionnalité s’exécute de façon fiable sur les espaces de travail mis à niveau vers le nouveau langage de requête.

## <a name="walkthrough"></a>Procédure pas à pas 

Le tableau de bord Network Performance Monitor présente une vue d’ensemble de l’intégrité des circuits ExpressRoute et des connexions de peering. 

![Tableau de bord de l’Analyseur de performances réseau](media/network-performance-monitor-expressroute/npm-dashboard-expressroute.png) 

### <a name="circuits-list"></a>Liste des circuits 

Pour afficher une liste de tous les circuits ExpressRoute surveillés, sélectionnez la vignette Circuits ExpressRoute. Vous pouvez sélectionner un circuit et afficher son état d’intégrité, des graphiques de tendances pour la perte de paquets, l’utilisation de la bande passante et la latence. Les graphiques sont interactifs. Vous pouvez sélectionner une fenêtre de temps personnalisée pour tracer les graphiques. Faites glisser la souris sur une zone sur le graphique pour zoomer et voir des points de données précis. 

![Liste des circuits ExpressRoute](media/network-performance-monitor-expressroute/expressroute-circuits.png) 

### <a name="trends-of-loss-latency-and-throughput"></a>Tendances concernant les pertes, la latence et le débit 

Les graphiques représentant l’utilisation de la bande passante, la latence et la perte sont interactifs. Vous pouvez zoomer sur n’importe quelle section de ces graphiques, à l’aide des contrôles de la souris. Vous pouvez également voir la bande passante, la latence et la perte de données pour les autres intervalles. Dans le coin supérieur gauche, sous le bouton **Actions**, sélectionnez **Date/Heure**. 

![Latence d’ExpressRoute](media/network-performance-monitor-expressroute/expressroute-latency.png) 

### <a name="peerings-list"></a>Liste des peerings 

Sélectionnez la vignette **Peerings privés** du tableau de bord pour afficher une liste de toutes les connexions à des réseaux virtuels sur le peering privé. Ici, vous pouvez sélectionner une connexion de réseau virtuel et afficher son état d’intégrité, des graphiques de tendances pour la perte de paquets, l’utilisation de la bande passante et la latence. 

![Peerings ExpressRoute](media/network-performance-monitor-expressroute/expressroute-peerings.png) 

### <a name="circuit-topology"></a>Topologie de circuit 

Pour afficher la topologie de circuit, sélectionnez la vignette **Topologie**. Cette action vous permet d’accéder à l’affichage de la topologie du circuit ou du peering sélectionné. Le diagramme de topologie fournit la latence pour chaque segment sur le réseau, et chaque tronçon de couche 3 est représenté par un nœud du diagramme. Sélectionnez un tronçon pour obtenir plus de détails sur ce dernier. Pour augmenter le niveau de visibilité et inclure des tronçons locaux, déplacez le curseur sous **FILTRES**. Déplacez le curseur vers la droite ou vers la gauche pour augmenter/diminuer le nombre de tronçons dans le graphique de la topologie. La latence pour chaque segment est visible, ce qui permet une isolation plus rapide des segments à latence élevée sur votre réseau.

![Topologie d’ExpressRoute](media/network-performance-monitor-expressroute/expressroute-topology.png)

### <a name="detailed-topology-view-of-a-circuit"></a>Vue détaillée de la topologie d’un circuit 

Cette vue affiche les connexions de réseau virtuel. 

![Connexions de réseau virtuel ExpressRoute](media/network-performance-monitor-expressroute/expressroute-vnet.png)
 
## <a name="diagnostics"></a>Diagnostics 

Network Performance Monitor vous permet de diagnostiquer plusieurs problèmes de connectivité de circuit. Voici quelques-uns de ces problèmes.

Vous pouvez afficher les codes de notification et définir des alertes avec **Log Analytics**. La page **Diagnostics NPM** présente la description de tous les messages de diagnostic déclenchés.

| Code de notification (journaux d’activité) | Description |
| --- | --- |
| 5501 | Impossible de passer par la connexion secondaire du circuit ExpressRoute |
| 5502 | Impossible de passer par la connexion principale du circuit ExpressRoute |
| 5503 | Aucun circuit trouvé pour l’abonnement lié à l’espace de travail | 
| 5508 | Impossible de déterminer si le trafic passe par un ou des circuits pour le chemin d’accès |
| 5510 | Le trafic ne passe pas par le circuit prévu | 
| 5511 | Le trafic ne passe pas par le réseau virtuel prévu | 

**Le circuit est arrêté.** Network Performance Monitor vous informe dès que la connectivité entre vos ressources locales et les réseaux virtuels Azure est perdue. Cette notification vous permet d’agir de façon proactive avant de recevoir les remontées des utilisateurs et de réduire le temps d’arrêt.

![Circuit ExpressRoute arrêté](media/network-performance-monitor-expressroute/expressroute-circuit-down.png)
 

**Le trafic n’est pas transmis via le circuit prévu.** Network Performance Monitor vous envoie une notification dès que le trafic ne passe pas par le circuit ExpressRoute prévu. Ce problème peut se produire si le circuit est arrêté et que le trafic est transmis via le routage de secours. Il peut également survenir s’il existe un problème de routage. Ces informations vous permettent de gérer proactivement les problèmes de configuration de vos stratégies de routage et de vous assurer que vous utilisez l’itinéraire le plus optimal et le plus sécurisé. 

 

**Le trafic n’est pas transmis via le circuit principal.** Network Performance Monitor vous avertit lorsque le trafic est transmis via le circuit ExpressRoute secondaire. Même si vous ne rencontrez aucun problème de connectivité dans ce cas, la résolution proactive des problèmes avec le circuit principal vous permet d’être mieux préparé. 

 
![Flux de trafic ExpressRoute](media/network-performance-monitor-expressroute/expressroute-traffic-flow.png)


**Dégradation due à un pic d’utilisation.** Vous pouvez mettre en corrélation la tendance d’utilisation de la bande passante avec la tendance de latence pour déterminer si la dégradation de la charge de travail Azure est due à un pic d’utilisation de la bande passante ou non, et agir en conséquence.

![Utilisation de la bande passante par ExpressRoute](media/network-performance-monitor-expressroute/expressroute-peak-utilization.png)

 

## <a name="next-steps"></a>Étapes suivantes
[Rechercher dans les journaux d’activité](../log-query/log-query-overview.md) pour afficher des enregistrements de données détaillées sur les performances réseau.

