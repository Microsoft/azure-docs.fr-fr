---
title: À propos de la passerelle VPN Azure
description: Découvrez ce qu’est une passerelle VPN et comment l’utiliser pour vous connecter à des réseaux virtuels IPsec IKE de site à site, de réseau virtuel à réseau virtuel et de point à site.
services: vpn-gateway
author: cherylmc
Customer intent: As someone with a basic network background, but is new to Azure, I want to understand the capabilities of Azure VPN Gateway so that I can securely connect to my Azure virtual networks.
ms.service: vpn-gateway
ms.topic: overview
ms.date: 08/27/2020
ms.author: cherylmc
ms.custom: contperf-fy21q1
ms.openlocfilehash: c19e1156f897873defaba4cebaabb904d034ead3
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97032743"
---
# <a name="what-is-vpn-gateway"></a>Qu’est-ce qu’une passerelle VPN ?

Une passerelle VPN est un type spécifique de passerelle de réseau virtuel qui est utilisé pour envoyer du trafic chiffré entre un réseau virtuel Azure et un emplacement sur site via l’Internet public. Vous pouvez également utiliser des passerelles VPN pour envoyer du trafic chiffré entre les réseaux virtuels Azure sur le réseau Microsoft. Chaque réseau virtuel ne peut posséder qu’une seule passerelle VPN. Toutefois, vous pouvez créer plusieurs connexions à la même passerelle VPN. Lorsque vous créez plusieurs connexions à la même passerelle VPN, tous les tunnels VPN partagent la bande passante de passerelle disponible.

## <a name="what-is-a-virtual-network-gateway"></a><a name="whatis"></a>Qu’est-ce qu’une passerelle de réseau virtuel ?

Une passerelle de réseau virtuel est composée de deux machines virtuelles ou plus déployées sur un sous-réseau spécifique que vous créez, appelé *sous-réseau de la passerelle*. Les machines virtuelles de passerelle de réseau virtuel contiennent des tables de routage et exécutent des services de passerelle spécifiques. Ces machines virtuelles sont créées lorsque vous créez la passerelle de réseau virtuel. Vous ne pouvez pas configurer directement les machines virtuelles qui font partie de la passerelle de réseau virtuel.

Quand vous configurez une passerelle de réseau virtuel, vous configurez un paramètre qui spécifie le type de passerelle. Le type de passerelle détermine la manière dont la passerelle de réseau virtuel est utilisée et les actions qu’elle exécute. Le type de passerelle « Vpn » spécifie que le type de passerelle de réseau virtuel créé est une « passerelle VPN ». Ceci la distingue d’une passerelle ExpressRoute, qui utilise un type de passerelle différent. Un réseau virtuel peut avoir deux passerelles de réseau virtuel : une passerelle VPN et une passerelle ExpressRoute. Pour plus d’informations, consultez [Types de passerelle](vpn-gateway-about-vpn-gateway-settings.md#gwtype).

La création d’une passerelle de réseau virtuel peut prendre jusqu’à 45 minutes. Lors de la création d’une passerelle de réseau virtuel, les machines virtuelles de passerelle sont déployées sur le sous-réseau de la passerelle et configurées avec les paramètres que vous spécifiez. Après avoir créé une passerelle VPN, vous pouvez créer une connexion de tunnel IPsec/IKE VPN entre cette passerelle VPN et une autre (de réseau virtuel à réseau virtuel), ou créer une connexion de tunnel IPsec/IKE VPN intersite entre la passerelle VPN et un périphérique VPN local (de site à site). Vous pouvez également créer une connexion VPN de point à site (VPN sur OpenVPN, IKEv2 ou SSTP), ce qui vous permet de vous connecter à votre réseau virtuel à partir d’un emplacement distant, par exemple une salle de conférence ou votre domicile.

## <a name="configuring-a-vpn-gateway"></a><a name="configuring"></a>Configuration d’une passerelle VPN

Une connexion par passerelle VPN s’appuie sur plusieurs ressources qui sont configurées avec des paramètres spécifiques. La plupart des ressources peuvent être configurées séparément, mais certaines d’entre elles doivent être configurées dans un certain ordre.

### <a name="design"></a><a name="diagrams"></a>Conception

Il est important de savoir qu’il existe différentes configurations disponibles pour les connexions aux passerelles VPN. Vous devez déterminer la configuration qui correspond le mieux à vos besoins. Par exemple, les instructions et exigences de configuration sont différentes entre les connexions point à site, site à site et ExpressRoute/site à site coexistantes. Pour plus d’informations sur la conception et pour voir des diagrammes de topologie de connexion, consultez [Conception](design.md).

### <a name="planning-table"></a><a name="planningtable"></a>Tableau de planification

Le tableau suivant peut vous aider à déterminer la meilleure option de connectivité pour votre solution.

[!INCLUDE [cross-premises](../../includes/vpn-gateway-cross-premises-include.md)]

### <a name="settings"></a><a name="settings"></a>Paramètres

Les paramètres que vous avez choisis pour chaque ressource sont essentiels à la création d’une connexion réussie. Pour plus d’informations sur les ressources et paramètres spécifiques pour la passerelle VPN, consultez [À propos des paramètres de passerelle VPN](vpn-gateway-about-vpn-gateway-settings.md). L’article contient des informations pour vous aider à comprendre les types de passerelle, les références SKU des passerelles, les types de VPN, les types de connexion, les sous-réseaux de passerelle, les passerelles de réseau local et divers autres paramètres de ressource que vous pouvez vouloir prendre en compte.

### <a name="deployment-tools"></a><a name="tools"></a>Outils de déploiement

Vous pouvez commencer par créer et configurer des ressources à l’aide de l’un des outils de configuration, comme le portail Azure. Vous pouvez décider de passer à un autre outil, tel que PowerShell, pour configurer des ressources supplémentaires ou pour modifier les ressources existantes, le cas échéant. Il n’est pour le moment pas possible de configurer toutes les ressources et tous les paramètres des ressources dans le portail Azure. Les instructions fournies dans les articles dédiés à chaque topologie de connexion indiquent si un outil de configuration spécifique est requis.

## <a name="gateway-skus"></a><a name="gwsku"></a>SKU de passerelle

Lorsque vous créez une passerelle de réseau virtuel, vous spécifiez la référence SKU de passerelle que vous voulez utiliser. Sélectionnez la référence SKU qui répond à vos besoins en fonction des types de charges de travail, des débits, des fonctionnalités et des contrats de niveau de service.

* Pour plus d’informations sur les références SKU de passerelle, notamment les fonctionnalités prises en charge, la production et le développement-test, et les étapes de configuration, consultez l’article [Paramètres de la passerelle VPN - Références SKU de passerelle](vpn-gateway-about-vpn-gateway-settings.md#gwsku).
* Pour plus d’informations sur les références SKU héritées, consultez [Utilisation de références SKU héritées](vpn-gateway-about-skus-legacy.md).

### <a name="gateway-skus-by-tunnel-connection-and-throughput"></a><a name="benchmark"></a>Références SKU de passerelle par tunnel, connexion et débit

[!INCLUDE [Aggregated throughput by SKU](../../includes/vpn-gateway-table-gwtype-aggtput-include.md)]

## <a name="availability-zones"></a><a name="availability"></a> Zones de disponibilité

Des passerelles VPN peuvent être déployées dans des zones de disponibilité Azure. Cela apporte de la résilience, de l’extensibilité et une plus grande disponibilité aux passerelles de réseau virtuel. Le déploiement de passerelles dans les zones de disponibilité Azure sépare les passerelles physiquement et logiquement au sein d’une région, tout en protégeant votre connectivité de réseau local à Azure à partir d’échecs au niveau de la zone. Consultez [À propos des passerelles de réseau virtuel redondantes interzone dans les Zones de disponibilité Azure](about-zone-redundant-vnet-gateways.md).

## <a name="pricing"></a><a name="pricing"></a>Tarifs

[!INCLUDE [vpn-gateway-about-pricing-include](../../includes/vpn-gateway-about-pricing-include.md)]

Pour plus d’informations sur les références de passerelle pour la passerelle VPN, consultez [Références (SKU) de passerelle](vpn-gateway-about-vpn-gateway-settings.md#gwsku).

## <a name="faq"></a><a name="faq"></a>Forum Aux Questions

Pour les questions fréquemment posées sur la passerelle VPN, consultez le [Forum aux questions sur la passerelle VPN](vpn-gateway-vpn-faq.md).

## <a name="whats-new"></a><a name="new"></a>Nouveautés

Abonnez-vous au flux RSS et consultez les dernières mises à jour des fonctionnalités Passerelle VPN dans la page [Mises à jour Azure](https://azure.microsoft.com/updates/?category=networking&query=VPN%20Gateway).

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations, consultez la [FAQ sur la passerelle VPN](vpn-gateway-vpn-faq.md).
- Consultez les [Limites du service et de l’abonnement](../azure-resource-manager/management/azure-subscription-service-limits.md#networking-limits).
- Découvrez certaines des autres [fonctionnalités de réseau](../networking/networking-overview.md) clés d’Azure.
