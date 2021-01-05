---
title: Visualiser les dépendances d’applications avec Azure Monitor pour machines virtuelles
description: Cartographie est une fonctionnalité d’Azure Monitor pour machines virtuelles. Elle détecte automatiquement les composants d’application sur les systèmes Windows et Linux, et cartographie la communication entre les différents services. Cet article explique comment utiliser la fonctionnalité Cartographie dans divers scénarios.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 03/20/2020
ms.openlocfilehash: 32ad92b333eda8cd429303177dba160f0b58af54
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95985796"
---
# <a name="use-the-map-feature-of-azure-monitor-for-vms-to-understand-application-components"></a>Utiliser la fonctionnalité Cartographie d’Azure Monitor pour machines virtuelles afin de mieux comprendre le fonctionnement des composants d’application
Dans Azure Monitor pour machines virtuelles, vous pouvez voir les composants d’application qui ont été détectés sur les machines virtuelles Windows et Linux exécutées dans Azure ou votre environnement. Il existe deux manières d’observer les machines virtuelles : en affichant une carte directement à partir d’une machine virtuelle ou en affichant une carte à partir d’Azure Monitor pour voir les composants dans les groupes de machines virtuelles. Cet article vous aidera à bien comprendre ces deux méthodes de visualisation et à utiliser correctement la fonctionnalité Cartographie. 

Pour plus d’informations sur la configuration d’Azure Monitor pour les machines virtuelles, consultez [Enable Azure Monitor for VMs ](vminsights-enable-overview.md)(Activer Azure Monitor pour les machines virtuelles).

## <a name="sign-in-to-azure"></a>Connexion à Azure
Connectez-vous au [portail Azure](https://portal.azure.com).

## <a name="introduction-to-the-map-experience"></a>Présentation de l’expérience avec la fonctionnalité Cartographie
Avant d’entrer dans les détails de l’expérience avec la fonctionnalité Cartographie, vous devez comprendre comment cette fonctionnalité présente et affiche les informations. Que vous sélectionniez la fonctionnalité Cartographie directement à partir d’une machine virtuelle ou à partir d’Azure Monitor, cette fonctionnalité offre une expérience similaire. La seule différence est qu’à partir d’Azure Monitor, vous voyez sur une carte unique tous les membres d’un cluster ou d’une application multiniveau.

La fonctionnalité Cartographie affiche les dépendances de machine virtuelle en détectant les processus en cours d’exécution qui présentent : 

- Des connexions réseau actives entre serveurs.
- Une latence des connexions entrantes et sortantes.
- Des ports sur n’importe quelle architecture connectée par TCP sur un intervalle de temps spécifié.  
 
Développez une machine virtuelle pour visualiser les détails uniquement des processus qui communiquent avec cette machine virtuelle. Le groupe de clients indique le nombre de clients front-end qui se connectent à la machine virtuelle. Les groupes de serveurs-ports affichent le nombre de serveurs back-end auxquels la machine virtuelle se connecte. Développez un groupe de serveurs-ports pour voir la liste détaillée des serveurs qui se connectent sur ce port.  

Quand vous sélectionnez la machine virtuelle, le volet **Propriétés** de droite affiche les propriétés de la machine virtuelle. Les propriétés incluent les informations système retournées par le système d’exploitation, les propriétés de la machine virtuelle Azure ainsi qu’un graphique en anneau qui récapitule les connexions détectées. 

![Volet Propriétés](./media/vminsights-maps/properties-pane-01.png)

Sur le côté droit du volet, sélectionnez **Événements de journal** pour afficher une liste des données que la machine virtuelle a envoyées à Azure Monitor. Ces données peuvent être interrogées.  Sélectionnez un type d’enregistrement pour ouvrir la page **Journaux**, où sont affichés les résultats trouvés pour ce type d’enregistrement. Dans cette page, vous voyez également une requête préconfigurée qui est filtrée sur la machine virtuelle sélectionnée.  

![Volet Événements de journal](./media/vminsights-maps/properties-pane-logs-01.png)

Fermez la page **Journaux** et revenez au volet **Propriétés**. Dans ce volet, sélectionnez **Alertes** pour voir les alertes sur les critères d’intégrité de la machine virtuelle. La fonctionnalité Cartographie intègre les alertes Azure pour afficher les alertes afférentes au serveur sélectionné sur l’intervalle de temps sélectionné. Le serveur affiche une icône pour les alertes actives, et le volet **Alertes de la machine** liste toutes les alertes. 

![Volet Alertes](./media/vminsights-maps/properties-pane-alerts-01.png)

Pour que la fonctionnalité Cartographie affiche des alertes pertinentes, créez une règle d’alerte qui s’applique à un ordinateur spécifique :

- Incluez une clause de regroupement des alertes par ordinateur (par exemple, **by Computer interval 1 minute**).
- Créez l’alerte sur la base d’une métrique.

Pour plus d’informations sur les alertes Azure et la création de règles d’alerte, consultez [Alertes unifiées dans Azure Monitor](../platform/alerts-overview.md).

Située dans le coin supérieur droit, l’option **Légende** décrit les symboles et les rôles cartographiés. Pour voir la carte plus en détail et vous déplacer autour, utilisez les contrôles de zoom disponibles en bas à droite. Vous pouvez définir le niveau de zoom et adapter la carte à la taille de la page.  

## <a name="connection-metrics"></a>Métriques de connexion
Le volet **Connexions** affiche les métriques standard pour la connexion sélectionnée de la machine virtuelle sur le port TCP. Les métriques comprennent le temps de réponse, le nombre de requêtes par minute, le débit du trafic et les liens.  

![Graphiques des connexions réseau dans le volet Connexions](./media/vminsights-maps/map-group-network-conn-pane-01.png)  

### <a name="failed-connections"></a>Connexions ayant échoué
La carte affiche les connexions ayant échoué pour les processus et les ordinateurs. Une ligne rouge en pointillés indique qu’un système client n’a pas réussi à accéder à un processus ou un port. Sur les systèmes qui utilisent Dependency Agent, cet agent signale les tentatives de connexion ayant échoué. La fonctionnalité Cartographie supervise un processus en observant les sockets TCP qui ne parviennent pas à établir une connexion. Cet échec peut être dû à un pare-feu, à une configuration incorrecte du client ou du serveur, ou à un service distant non disponible.

![Carte illustrant une connexion ayant échoué](./media/vminsights-maps/map-group-failed-connection-01.png)

En ayant une bonne compréhension des échecs de connexions, vous pourrez plus facilement résoudre les problèmes rencontrés, valider la migration, analyser la sécurité et comprendre l’architecture globale du service. Les échecs de connexions sont parfois sans incidence, mais ils révèlent souvent un problème sous-jacent. Les connexions peuvent échouer, par exemple, quand un environnement de basculement devient subitement inaccessible ou lorsque deux niveaux d’application ne peuvent plus communiquer entre eux après une migration vers le cloud.

### <a name="client-groups"></a>Groupes de clients
Sur la carte, les groupes de clients représentent les machines clientes qui se connectent à la machine cartographiée. Un groupe de clients représente les clients d’un seul processus ou d’une seule machine.

![Carte illustrant un groupe de clients](./media/vminsights-maps/map-group-client-groups-01.png)

Pour voir les adresses IP et clients supervisés des systèmes dans un groupe de clients, sélectionnez le groupe. Le contenu du groupe est illustré ci-dessous.  

![Carte montrant la liste des adresses IP d’un groupe de clients](./media/vminsights-maps/map-group-client-group-iplist-01.png)

Si le groupe inclut des clients supervisés et non supervisés, vous pouvez sélectionner la section appropriée dans le graphique en anneau du groupe pour filtrer les clients.

### <a name="server-port-groups"></a>Groupes de serveurs-ports
Les groupes de serveurs-ports représentent les ports sur les serveurs qui reçoivent des connexions établies à partir de la machine cartographiée. Le groupe indique le port du serveur ainsi que le nombre de serveurs qui ont des connexions à ce port. Sélectionnez le groupe pour voir les différents serveurs et connexions. 

![Carte illustrant un groupe de serveurs-ports](./media/vminsights-maps/map-group-server-port-groups-01.png)  

Si le groupe inclut des serveurs supervisés et non supervisés, vous pouvez sélectionner la section appropriée dans le graphique en anneau du groupe pour filtrer les serveurs.

## <a name="view-a-map-from-a-vm"></a>Afficher une carte à partir d’une machine virtuelle 

Pour accéder à Azure Monitor pour machines virtuelles directement à partir d’une machine virtuelle :

1. Dans le portail Azure, sélectionnez **Machines virtuelles**. 
2. Dans la liste, sélectionnez une machine virtuelle. Dans la section **Supervision**, choisissez **Insights**.  
3. Sélectionnez l’onglet **Carte**.

La carte affiche les dépendances de la machine virtuelle en détectant les processus et groupes de processus en cours d’exécution qui ont des connexions réseau actives sur l’intervalle de temps spécifié.  

Par défaut, la carte affiche les 30 dernières minutes. Si vous souhaitez voir les dépendances sur une plus grande période passée, vous pouvez exécuter une requête sur un intervalle de temps historique allant jusqu’à une heure. Pour exécuter la requête, utilisez le sélecteur **TimeRange** dans le coin supérieur gauche. Vous pouvez exécuter une requête, par exemple, durant un incident ou pour afficher l’état avant une modification.  

![Capture d’écran de l’onglet Carte dans la section Insights d’analyse du portail Azure, montrant un diagramme des dépendances entre les machines virtuelles.](./media/vminsights-maps/map-direct-vm-01.png)

## <a name="view-a-map-from-a-virtual-machine-scale-set"></a>Afficher une carte à partir d’un groupe de machines virtuelles identiques

Pour accéder à Azure Monitor pour machines virtuelles directement à partir d’un groupe de machines virtuelles identiques :

1. Dans le Portail Microsoft Azure, sélectionnez **Groupes de machines virtuelles identiques**.
2. Dans la liste, sélectionnez une machine virtuelle. Ensuite, dans la section **Supervision**, choisissez **Insights**.  
3. Sélectionnez l’onglet **Carte**.

La carte affiche toutes les instances dans le groupe identique sous forme de nœud de groupe présentant toutes les dépendances du groupe. Le nœud développé liste les instances dans le groupe identique. Vous pouvez faire défiler jusqu’à 10 instances à la fois. 

Pour charger la carte d’une instance spécifique, sélectionnez d’abord cette instance sur la carte. Ensuite, sélectionnez le bouton représentant des **points de suspension** (...) à droite et choisissez **Charger la carte des serveurs**. Sur la carte qui s’affiche, vous voyez tous les processus et groupes de processus qui ont des connexions réseau actives sur l’intervalle de temps spécifié. 

Par défaut, la carte affiche les 30 dernières minutes. Si vous souhaitez voir les dépendances sur une plus grande période passée, vous pouvez exécuter une requête sur un intervalle de temps historique allant jusqu’à une heure. Pour exécuter la requête, utilisez le sélecteur **TimeRange**. Vous pouvez exécuter une requête, par exemple, durant un incident ou pour afficher l’état avant une modification.

![Capture d’écran de l’onglet Carte dans la section Insights d’analyse du portail Azure, montrant un diagramme des dépendances entre les groupes de machines virtuelles identiques.](./media/vminsights-maps/map-direct-vmss-01.png)

>[!NOTE]
>Vous pouvez également accéder à la carte d’une instance spécifique à partir de la vue **Instances** de votre groupe de machines virtuelles identiques. Dans la section **Paramètres**, accédez à **Instances** > **Insights**.

## <a name="view-a-map-from-azure-monitor"></a>Afficher une carte à partir d’Azure Monitor

Dans Azure Monitor, la fonctionnalité Cartographie offre une vue globale de vos machines virtuelles et de leurs dépendances. Pour accéder à la fonctionnalité Cartographie dans Azure Monitor :

1. Dans le Portail Azure, sélectionnez **Surveiller**. 
2. Dans la section **Insights**, choisissez **Machines virtuelles**.
3. Sélectionnez l’onglet **Carte**.

   ![Carte globale d’Azure Monitor des différentes machines virtuelles](./media/vminsights-maps/map-multivm-azure-monitor-01.png)

Choisissez un espace de travail à l’aide du sélecteur **Espace de travail** en haut de la page. Si vous avez plusieurs espaces de travail Log Analytics, choisissez l’espace de travail qui est activé avec la solution et qui contient des machines virtuelles supervisées. 

Le sélecteur **Groupe** affiche les abonnements, les groupes de ressources, les [groupes d’ordinateurs](../platform/computer-groups.md) et les groupes de machines virtuelles identiques qui sont associés à l’espace de travail sélectionné. Votre sélection s’applique uniquement à la fonctionnalité Cartographie, et n’est pas reportée sur la fonctionnalité Performances ou Intégrité.

Par défaut, la carte affiche les 30 dernières minutes. Si vous souhaitez voir les dépendances sur une plus grande période passée, vous pouvez exécuter une requête sur un intervalle de temps historique allant jusqu’à une heure. Pour exécuter la requête, utilisez le sélecteur **TimeRange**. Vous pouvez exécuter une requête, par exemple, durant un incident ou pour afficher l’état avant une modification.  

## <a name="next-steps"></a>Étapes suivantes

Pour identifier les goulots d’étranglement, superviser les performances et comprendre l’utilisation globale de vos machines virtuelles, consultez [Afficher les performances avec Azure Monitor pour machines virtuelles](vminsights-performance.md). 

