---
title: Azure Monitor pour réseaux
description: Vue d’ensemble d’Azure Monitor pour réseaux, qui fournit une vue complète de l’intégrité et des métriques pour toutes les ressources réseau déployées sans aucune configuration.
ms.subservice: ''
ms.topic: conceptual
author: KumudD
ms.author: kumud
ms.date: 11/25/2020
ms.openlocfilehash: e4b555f4727d2d1fb5b64814ae07aa15b78da42c
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97510921"
---
# <a name="azure-monitor-for-networks"></a>Azure Monitor pour réseaux

Azure Monitor pour réseaux fournit une vue complète de l’[intégrité](../../service-health/resource-health-checks-resource-types.md) et des [métriques](../platform/metrics-supported.md) pour toutes les ressources réseau déployées, sans nécessiter aucune configuration. Il permet également d’accéder à toutes les capacités de surveillance du réseau, comme [Moniteur de connexion](../../network-watcher/connection-monitor-preview.md), la [journalisation des flux pour les groupes de sécurité réseau (NSG)](../../network-watcher/network-watcher-nsg-flow-logging-overview.md) et [Traffic Analytics](../../network-watcher/traffic-analytics.md). En outre, il fournit d’autres fonctionnalités de [diagnostic](../../network-watcher/network-watcher-monitoring-overview.md#diagnostics) du réseau.

Azure Monitor pour réseaux est structuré autour de ces principaux composants de surveillance :
- [Intégrité de réseau et métriques](#networkhealth)
- [Connectivité](#connectivity)
- [Trafic](#traffic)
- [Boîte à outils de diagnostic](#diagnostictoolkit)

## <a name="network-health-and-metrics"></a><a name="networkhealth"></a>Intégrité du réseau et métriques

La page **Vue d’ensemble** d’Azure Monitor pour réseaux vous permet de visualiser facilement l’inventaire de vos ressources réseau ainsi que les alertes et l’intégrité des ressources. Elle est divisée en quatre principales zones fonctionnelles : recherche et filtrage, intégrité des ressources et métriques, alertes et affichage des dépendances.

[![Capture d’écran de la page Vue d’ensemble](media/network-insights-overview/overview.png)](media/network-insights-overview/overview.png#lightbox)

### <a name="search-and-filtering"></a>Recherche et filtrage
Vous pouvez personnaliser l’affichage des alertes et de l’intégrité des ressources en utilisant des filtres comme **Abonnement**, **Groupe de ressources** et **Type**.

Vous pouvez utiliser la zone de recherche pour rechercher des ressources et leurs ressources associées. Par exemple, une IP publique est associée à une passerelle applicative. La recherche du nom DNS de l’IP publique renverra à la fois l’IP publique et la passerelle applicative associée :

[![Capture d’écran montrant les résultats de recherche d’Azure Monitor pour réseaux.](media/network-insights-overview/search.png)](media/network-insights-overview/search.png#lightbox)


### <a name="resource-health-and-metrics"></a>Intégrité des ressources et métriques
Dans l’exemple suivant, chaque vignette représente un type de ressource. La vignette affiche le nombre d’instances de ce type de ressource déployées dans tous les abonnements sélectionnés. Elle affiche également l’état d’intégrité de la ressource. Dans cet exemple, 105 connexions ER et VPN sont déployées : 103 sont saines et 2 ne sont pas disponibles.

![Capture d’écran montrant l’intégrité et les métriques des ressources dans Azure Monitor pour réseaux.](media/network-insights-overview/resource-health.png)

Si vous sélectionnez les connexions ER et VPN non disponibles, un affichage des métriques s’affiche : 

![Capture d’écran montrant l’affichage des métriques dans Azure Monitor pour réseaux.](media/network-insights-overview/metric-view.png)

Vous pouvez sélectionner n’importe quel élément en mode grille. Sélectionnez l’icône dans la colonne **Intégrité** pour connaître l’intégrité des ressources pour cette connexion. Sélectionnez la valeur dans la colonne **Alerte** pour accéder à la page des alertes et des métriques pour la connexion. 

### <a name="alerts"></a>Alertes
La zone **Alertes** sur le côté droit de la page fournit une vue de toutes les alertes générées pour les ressources sélectionnées dans tous les abonnements. Sélectionnez le nombre d’alertes pour accéder à la page détaillée des alertes.

### <a name="dependency-view"></a>Affichage des dépendances
L’affichage des dépendances vous permet de visualiser la façon dont une ressource est configurée. L’affichage des dépendances est actuellement disponible pour Azure Application Gateway, Azure Virtual WAN et Azure Load Balancer. Par exemple, pour Application Gateway, vous pouvez accéder à l’affichage des dépendances en sélectionnant le nom de la ressource Application Gateway dans la grille des métriques. Vous pouvez effectuer la même opération pour Virtual WAN et Load Balancer.

![Capture montrant la vue Application Gateway dans Azure Monitor pour réseaux.](media/network-insights-overview/application-gateway.png)

L’affichage des dépendances pour Application Gateway fournit une vue simplifiée de la façon dont les adresses IP frontales sont connectées aux écouteurs, aux règles et au pool principal. Les lignes de connexion sont codées par couleur et fournissent des détails supplémentaires en fonction de l’intégrité du pool principal. L’affichage fournit également une vue détaillée des métriques Application Gateway et des métriques pour tous les pools principaux associés, comme les groupes de machines virtuelles identiques et les instances de machine virtuelle.

[![Capture d’écran illustrant l’affichage des dépendances dans Azure Monitor pour réseaux.](media/network-insights-overview/dependency-view.png)](media/network-insights-overview/dependency-view.png#lightbox)

Le graphique de dépendance facilite la navigation vers les paramètres de configuration. Cliquez avec le bouton droit sur un pool principal pour accéder à d’autres informations. Par exemple, si le pool principal est une machine virtuelle, vous pouvez accéder directement aux insights de machine virtuelle et à la résolution des problèmes de connexion Azure Network Watcher pour identifier les problèmes de connectivité :

![Capture d’écran montrant le menu Affichage des dépendances dans Azure Monitor pour réseaux.](media/network-insights-overview/dependency-view-menu.png)

La barre de recherche et de filtre de l’affichage des dépendances permet d’effectuer facilement des recherches dans le graphique. Par exemple, si vous recherchez **AppGWTestRule** dans l’exemple précédent, l’affichage effectue un scale-down sur tous les nœuds connectés via AppGWTestRule :

![Capture d’écran illustrant un exemple de recherche dans Azure Monitor pour réseaux.](media/network-insights-overview/search-example.png)

Différents filtres vous permettent d’effectuer un scale-down sur un chemin d’accès et un état spécifiques. Par exemple, sélectionnez uniquement **Non sain** dans la liste **État d’intégrité** pour afficher toutes les arêtes pour lesquelles l’état est non sain.

Sélectionnez **Afficher les métriques détaillées** pour ouvrir un classeur préconfiguré qui fournit des métriques détaillées pour la passerelle applicative, toutes les ressources du pool principal et les adresses IP frontales. 

## <a name="connectivity"></a><a name="connectivity"></a>Connectivité

L’onglet **Connectivité** offre un moyen facile de visualiser tous les tests configurés à l’aide du [Moniteur de connexion](../../network-watcher/connection-monitor-overview.md) et du Moniteur de connexion (classique) pour l’ensemble d’abonnements sélectionné.

![Capture d’écran montrant l’onglet Connectivité dans Azure Monitor pour réseaux.](media/network-insights-overview/azure-monitor-for-networks-connectivity-tab.png)

Les tests sont regroupés par vignettes **Sources** et **Destinations** et affichent l’état d’accessibilité pour chaque test. Les paramètres accessibles permettent d’accéder facilement aux configurations pour vos critères d’accessibilité en fonction des vérifications ayant échoué (%) et du RTT (ms). Une fois que vous avez défini les valeurs, l’état de chaque test est mis à jour en fonction des critères de sélection.

[![Capture d’écran montrant les tests de connectivité dans Azure Monitor pour réseaux.](media/network-insights-overview/azure-monitor-for-networks-connectivity-tests.png)](media/network-insights-overview/azure-monitor-for-networks-connectivity-tests.png#lightbox)

Vous pouvez sélectionner n’importe quelle vignette source ou de destination pour ouvrir l’affichage d’une métrique :

[![Capture d’écran montrant les métriques de connectivité dans Azure Monitor pour réseaux.](media/network-insights-overview/azure-monitor-for-networks-connectivity-metrics.png)](media/network-insights-overview/azure-monitor-for-networks-connectivity-metrics.png#lightbox)


Vous pouvez sélectionner n’importe quel élément en mode grille. Sélectionnez l’icône dans la colonne **Accessibilité** pour accéder à la page du portail Moniteur de connexion et afficher la topologie tronçon par tronçon et les problèmes de connectivité identifiés. Sélectionnez la valeur dans la colonne **Alerte** pour accéder aux alertes. Sélectionnez les graphiques dans les colonnes **Pourcentage de vérifications ayant échoué** et **Durée aller-retour (ms)** pour accéder à la page des métriques pour le moniteur de connexion sélectionné.

La zone  **Alerte** sur le côté droit de la page fournit une vue de toutes les alertes générées pour les tests de connectivité configurés dans tous les abonnements. Sélectionnez le nombre d’alertes pour accéder à la page détaillée des alertes.

## <a name="traffic"></a><a name="traffic"></a>Trafic
L’onglet **Trafic** permet d’accéder à tous les NSG configurés pour [Journaux de flux NSG](../../network-watcher/network-watcher-nsg-flow-logging-overview.md) et [Traffic Analytics](../../network-watcher/traffic-analytics.md) pour le jeu d’abonnements sélectionné, regroupés par emplacement. La fonctionnalité de recherche fournie sous cet onglet vous permet d’identifier les NSG configurés pour l’adresse IP recherchée. Vous pouvez rechercher n’importe quelle adresse IP dans votre environnement. La vue régionale en mosaïque affiche tous les NSG, ainsi que les journaux de flux NSG et l’état de configuration de Traffic Analytics.

[![Capture d’écran montrant l’onglet Trafic dans Azure Monitor pour réseaux.](media/network-insights-overview/azure-monitor-for-networks-traffic-view.png)](media/network-insights-overview/azure-monitor-for-networks-traffic-view.png#lightbox)

Si vous sélectionnez une vignette de région, l’affichage apparait en mode grille. La grille fournit des journaux de flux NSG et Traffic Analytics dans une vue facile à lire et à configurer :  

[![Capture d’écran montrant l’affichage Région du trafic dans Azure Monitor pour réseaux.](media/network-insights-overview/azure-monitor-for-networks-traffic-region-view.png)](media/network-insights-overview/azure-monitor-for-networks-traffic-region-view.png#lightbox)

Vous pouvez sélectionner n’importe quel élément en mode grille. Sélectionnez l’icône dans la colonne **État de la configuration du journal de flux** pour modifier la configuration des journaux de flux NSG et de Traffic Analytics. Sélectionnez la valeur dans la colonne **Alerte** pour accéder aux alertes de trafic configurées pour le NSG sélectionné. De même, vous pouvez accéder à la vue Traffic Analytics en sélectionnant l’**espace de travail Traffic Analytics**.  

La zone  **Alertes** sur le côté droit de la page fournit une vue de toutes les alertes basées sur l’espace de travail Traffic Analytics dans tous les abonnements. Sélectionnez le nombre d’alertes pour accéder à la page détaillée des alertes.

## <a name="diagnostic-toolkit"></a><a name="diagnostictoolkit"></a> Boîte à outils de diagnostic
La boîte à outils de diagnostic fournit un accès à toutes les fonctionnalités de diagnostic disponibles pour la résolution des problèmes du réseau. Vous pouvez utiliser cette liste déroulante pour accéder à des fonctionnalités telles que la [capture de paquets](../../network-watcher/network-watcher-packet-capture-overview.md), la [résolution des problèmes de VPN](../../network-watcher/network-watcher-troubleshoot-overview.md), la [résolution des problèmes de connexion](../../network-watcher/network-watcher-connectivity-overview.md), le [tronçon suivant](../../network-watcher/network-watcher-next-hop-overview.md) et la [vérification du flux IP](../../network-watcher/network-watcher-ip-flow-verify-overview.md) :

![Capture d’écran montrant l’onglet Boîte à outils de diagnostic.](media/network-insights-overview/azure-monitor-for-networks-diagnostic-toolkit.png)

## <a name="troubleshooting"></a>Dépannage 

Pour obtenir des conseils généraux sur la résolution des problèmes, consultez l’[article de résolution des problèmes](troubleshoot-workbooks.md) dédié aux insights basés sur des classeurs.

Cette section vous aide à diagnostiquer et à résoudre certains problèmes courants que vous pouvez rencontrer lorsque vous utilisez Azure Monitor pour réseaux. 

### <a name="how-do-i-resolve-performance-problems-or-failures"></a>Comment résoudre les problèmes de performances ou les échecs ?

Pour en savoir plus sur la résolution des problèmes liés à la mise en réseau que vous identifiez avec Azure Monitor pour réseaux, consultez la documentation sur la résolution des problèmes de la ressource défectueuse. 

Voici quelques liens vers des articles de dépannage pour les services fréquemment utilisés. Pour plus d’informations sur la résolution des problèmes liés à ces services, consultez les autres articles de la section Résolution des problèmes de la table des matières du service.
* [Réseau virtuel Azure](../../virtual-network/virtual-network-troubleshoot-peering-issues.md)
* [Application Gateway Azure](../../application-gateway/create-gateway-internal-load-balancer-app-service-environment.md)
* [Passerelle VPN Azure](../../vpn-gateway/vpn-gateway-troubleshoot.md)
* [Azure ExpressRoute](../../expressroute/expressroute-troubleshooting-expressroute-overview.md) 
* [Équilibrage de charge Azure](../../load-balancer/load-balancer-troubleshoot.md) 

### <a name="why-dont-i-see-the-resources-for-all-the-subscriptions-ive-selected"></a>Pourquoi ne puis-je pas voir les ressources de tous les abonnements que j’ai sélectionnés ?

Azure Monitor pour réseaux ne peut afficher les ressources que pour cinq abonnements à la fois. 

### <a name="how-do-i-make-changes-or-add-visualizations-to-azure-monitor-for-networks"></a>Comment apporter des modifications ou ajouter des visualisations à Azure Monitor pour réseaux ?

Pour apporter des modifications, sélectionnez **Mode d’édition** afin de modifier le classeur. Vous pouvez ensuite enregistrer vos modifications dans un nouveau classeur qui est lié à un abonnement et à un groupe de ressources désignés.

### <a name="whats-the-time-grain-after-i-pin-any-part-of-the-workbooks"></a>Quel est le fragment de temps après avoir épinglé une partie des classeurs ?

Azure Monitor pour réseaux utilise le fragment de temps **automatique**, de sorte que le fragment de temps est basé sur l’intervalle de temps sélectionné.

### <a name="whats-the-time-range-when-any-part-of-a-workbook-is-pinned"></a>Quel est l’intervalle de temps quand une partie de classeur est épinglée ?

L’intervalle de temps dépend des paramètres du tableau de bord.

### <a name="what-if-i-want-to-see-other-data-or-make-my-own-visualizations-how-can-i-make-changes-to-azure-monitor-for-networks"></a>Que se passe-t-il si je souhaite voir d’autres données ou créer mes propres visualisations ? Comment puis-je apporter des modifications à Azure Monitor pour réseaux ?

Vous pouvez modifier le classeur que vous voyez dans n’importe quel panneau latéral ou affichage de métrique détaillée en utilisant le mode d’édition. Vous pouvez ensuite enregistrer vos modifications dans un nouveau classeur.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur l’analyse des réseaux : [Présentation d’Azure Network Watcher](../../network-watcher/network-watcher-monitoring-overview.md)
- Découvrez les scénarios pour lesquels les classeurs sont conçus, la manière de créer des rapports et de personnaliser les rapports existants, et bien plus encore : [Créer des rapports interactifs avec les classeurs Azure Monitor](../platform/workbooks-overview.md)
