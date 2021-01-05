---
title: Métriques et journaux d’activité
titleSuffix: Azure Virtual WAN
description: En savoir plus sur les journaux et métriques Azure Virtual WAN
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: how-to
ms.date: 06/05/2020
ms.author: cherylmc
ms.openlocfilehash: 53b8d74d6eb35347d6ac5b27d12fa5b5eaed2582
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94566365"
---
# <a name="azure-virtual-wan-logs-and-metrics"></a>Journaux et métriques Azure Virtual WAN

Vous pouvez surveiller Azure Virtual WAN à l’aide d’Azure Monitor. Virtual WAN est un service de mise en réseau qui combine un grand nombre de fonctionnalités de mise en réseau, de sécurité et de routage pour fournir une interface opérationnelle unique. Les passerelles VPN Virtual WAN, les passerelles ExpressRoute et Pare-feu Azure disposent de journaux et de métriques disponibles via Azure Monitor. Pour plus d’informations sur Pare-feu Azure, consultez [Journaux et métriques du pare-feu Azure](../firewall/logs-and-metrics.md).

Cet article présente les métriques et les diagnostics disponibles via le portail. Les métriques sont légères et peuvent prendre en charge des scénarios en quasi-temps réel, ce qui les rend utiles pour les alertes et la détection rapide de problèmes.

## <a name="metrics"></a>Mesures

Les métriques dans Azure Monitor sont des valeurs numériques décrivant certains aspects d’un système à un moment donné. Les métriques sont collectées toutes les minutes et sont utiles pour les alertes, car elles peuvent être échantillonnées fréquemment. Une alerte peut être déclenchée rapidement avec une logique relativement simple.

### <a name="site-to-site-vpn-gateways"></a>Passerelles VPN site à site

Les métriques suivantes sont disponibles pour les passerelles VPN site à site Azure :

* **Bande passante de la passerelle** : bande passante agrégée site à site moyenne d’une passerelle en octets par seconde.
* **Bande passante du tunnel** : bande passante moyenne d’un tunnel en octets par seconde.
* **Octets de sortie du tunnel** : octets sortants d’un tunnel. 
* **Paquets de sortie de tunnel** : nombre de paquets sortants d’un tunnel. 
* **Rejet de paquets TS de sortie de tunnel pour incompatibilité** : nombre de rejets de paquets sortants du sélecteur de trafic pour incompatibilité dans un tunnel. 
* **Octets d’entrée de tunnel** : octets entrants d’un tunnel. 
* **Paquets d’entrée de tunnel** : nombre de paquets entrants d’un tunnel. 
* **Rejet de paquets TS d’entrée de tunnel pour incompatibilité** : nombre de rejets de paquets entrants du sélecteur de trafic pour incompatibilité dans un tunnel. 

### <a name="point-to-site-vpn-gateways"></a>Passerelles VPN point à site

Les métriques suivantes sont disponibles pour les passerelles VPN point à site Azure :

* **Bande passante P2S de la passerelle** : bande passante agrégée point à site moyenne d’une passerelle en octets par seconde.
* **Nombre de connexions P2S** : nombre de connexions point à site d’une passerelle.

### <a name="azure-expressroute-gateways"></a>Passerelles ExpressRoute Azure

Les métriques suivantes sont disponibles pour les passerelles ExpressRoute Azure :

* **BitsInPerSecond** : bits entrants dans Azure par seconde.
* **BitsInPerSecond** : bits entrant dans Azure par seconde.

### <a name="view-gateway-metrics"></a><a name="metrics-steps"></a>Afficher les métriques de passerelle

Les étapes suivantes vous aident à localiser et à afficher les métriques :

1. Dans le portail, accédez au hub virtuel qui contient la passerelle.

2. Sélectionnez **VPN (site à site)** pour localiser une passerelle site à site, **ExpressRoute** pour localiser une passerelle ExpressRoute ou **VPN utilisateur (point à site)** pour localiser une passerelle point à site. Sur la page, vous pouvez voir les informations de la passerelle. Copiez ces informations. Vous les utiliserez ultérieurement pour afficher les diagnostics à l’aide d’Azure Monitor.

3. Sélectionnez **Métriques**.

   :::image type="content" source="./media/logs-metrics/metrics.png" alt-text="Capture d'écran d'un volet de VPN site à site dans lequel Afficher dans Azure Monitor est sélectionné.":::

4. Dans la page **Métriques**, vous pouvez afficher les métriques qui vous intéressent.

   :::image type="content" source="./media/logs-metrics/metrics-page.png" alt-text="Capture d’écran montrant la page « Métriques » avec les catégories en surbrillance.":::

## <a name="diagnostic-logs"></a><a name="diagnostic"></a>Journaux de diagnostic

### <a name="site-to-site-vpn-gateways"></a>Passerelles VPN site à site

Les diagnostics suivants sont disponibles pour les passerelles VPN site à site Azure :

* **Journaux de diagnostic de la passerelle** : diagnostics spécifiques à la passerelle, tels que l’intégrité, la configuration, les mises à jour du service, ainsi que des diagnostics supplémentaires.
* **Journaux de diagnostic du tunnel** : il s’agit de journaux relatifs au tunnel IPsec, tels que les événements de connexion et de déconnexion pour un tunnel IPsec site à site, les associations de sécurité négociées, les raisons de déconnexion, ainsi que des diagnostics supplémentaires.
* **Journaux de diagnostic d’itinéraires** : il s’agit des journaux relatifs aux événements pour les itinéraires statiques, Border Gateway Protocol, les mises à jour des itinéraires, ainsi que des diagnostics supplémentaires.
* **Journaux de diagnostic IKE** : diagnostics spécifiques à IKE pour les connexions IPsec.

### <a name="point-to-site-vpn-gateways"></a>Passerelles VPN point à site

Les diagnostics suivants sont disponibles pour les passerelles VPN point à site Azure :

* **Journaux de diagnostic de la passerelle** : diagnostics spécifiques à la passerelle, tels que l’intégrité, la configuration, les mises à jour du service, ainsi que d’autres diagnostics.
* **Journaux de diagnostic IKE** : diagnostics spécifiques à IKE pour les connexions IPsec.
* **Journaux de diagnostic P2S** : il s’agit des événements de client et de configuration P2S (point à site) du VPN des utilisateurs. Il s’agit notamment de la connexion/déconnexion des clients, de l’allocation d’adresses de clients VPN, ainsi que d’autres Diagnostics.

### <a name="view-diagnostic-logs"></a><a name="diagnostic-steps"></a>Afficher les journaux de diagnostic

Les étapes suivantes vous aident à localiser et à afficher les diagnostics :

1. Dans le portail, accédez à votre ressource Virtual WAN. Dans la section **Vue d’ensemble** de la page Virtual WAN du portail, sélectionnez **Essentiels** pour développer la vue et obtenir des informations sur le groupe de ressources. Copiez les informations du groupe de ressources.

   :::image type="content" source="./media/logs-metrics/3.png" alt-text="Capture d’écran montrant la section « Vue d’ensemble » avec une flèche pointant vers le bouton « Copier ».":::

2. Dans la section Surveillance, accédez au groupe de ressources. Sélectionnez **Paramètres de diagnostic**, puis entrez les informations sur les ressources. Il s’agit des informations sur les ressources que vous avez copiées à l’étape 2 de la section [Afficher les métriques de passerelle](#metrics-steps), plus haut dans cet article.

   :::image type="content" source="./media/logs-metrics/4.png" alt-text="Capture d’écran montrant la section « Surveillance » avec une flèche pointant vers la liste déroulante « Ressource ».":::

3. Dans la page des résultats, sélectionnez **+ Ajouter un paramètre de diagnostic**, puis sélectionnez une option. Vous pouvez choisir d’envoyer à Log Analytics, de diffuser vers un Event Hub ou de simplement archiver dans un compte de stockage.

   :::image type="content" source="./media/logs-metrics/5.png" alt-text="page métriques":::

### <a name="log-analytics-sample-query"></a><a name="sample-query"></a>Exemple de requête Log Analytics

Les journaux se trouvent dans **Espace de travail Log Analytics Azure**. Vous pouvez configurer une requête dans Log Analytics. L’exemple suivant contient une requête visant à obtenir des diagnostics d’itinéraire site à site.

```AzureDiagnostics | where Category == "RouteDiagnosticLog"```

Remplacez les valeurs ci-dessous, après le caractère **==** , si nécessaire.

* "GatewayDiagnosticLog"
* "IKEDiagnosticLog"
* "P2SDiagnosticLog”
* "TunnelDiagnosticLog"
* "RouteDiagnosticLog"

## <a name="activity-logs"></a><a name="activity-logs"></a>Journaux d’activité

Les entrées du **journal d’activité** sont recueillies par défaut et peuvent être affichées dans le portail Azure. Vous pouvez utiliser les journaux d’activité Azure (anciennement *journaux des opérations* et *journaux d’audit*) pour afficher toutes les opérations soumises à votre abonnement Azure.

## <a name="next-steps"></a>Étapes suivantes

* Pour savoir comment superviser les métriques et les journaux de Pare-feu Azure, consultez le [Tutoriel : Surveiller les journaux d’activité de pare-feu Azure](../firewall/firewall-diagnostics.md).
* Pour en savoir plus sur les métriques dans Azure Monitor, consultez [Métriques dans Azure Monitor](../azure-monitor/platform/data-platform-metrics.md).