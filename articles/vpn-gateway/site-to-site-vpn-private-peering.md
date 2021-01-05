---
title: Connexions VPN site à site via un peering privé ExpressRoute
description: Cet article vous aide à activer le VPN site à site via un peering privé ExpressRoute afin de chiffrer le trafic.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 11/16/2020
ms.author: cherylmc
ms.openlocfilehash: 62ceafad0210065700e5c9734cfe9a055208ef35
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94657888"
---
# <a name="configure-a-site-to-site-vpn-connection-over-expressroute-private-peering"></a>Configurer une connexion VPN site à site via un peering privé ExpressRoute

Vous pouvez configurer un VPN site à site vers une passerelle de réseau virtuel via un peering privé ExpressRoute à l’aide d’une adresse IP de la RFC 1918. Cette configuration offre les avantages suivants :

* Le trafic via le peering privé est chiffré.

* Les utilisateurs point à site qui se connectent à une passerelle de réseau virtuel peuvent utiliser ExpressRoute (via le tunnel site à site) pour accéder aux ressources locales.

>[!NOTE]
>Cette fonctionnalité est uniquement prise en charge sur les passerelles redondantes interzones. Par exemple, VpnGw1AZ, VpnGw2AZ, etc.
>

Pour effectuer cette configuration, veillez à respecter les prérequis suivants :

* Vous disposez d’un circuit ExpressRoute fonctionnel qui est lié au réseau virtuel dans lequel la passerelle VPN est (ou sera) créée.

* Vous pouvez atteindre les ressources sur l’adresse IP RFC 1918 (privée) dans le réseau virtuel via le circuit ExpressRoute.

## <a name="routing"></a><a name="routing"></a>Routage

La **figure 1** illustre un exemple de connectivité VPN via un peering privé ExpressRoute. Dans cet exemple, vous voyez un réseau du réseau local connecté à la passerelle VPN du hub Azure via un peering privé ExpressRoute. L’un des aspects importants de cette configuration est le routage entre les réseaux locaux et Azure sur le chemin VPN et le chemin ExpressRoute.

La figure 1

:::image type="content" source="media/site-to-site-vpn-private-peering/routing.png" alt-text="Figure 1":::

L’établissement de la connectivité est simple :

1. Établir une connectivité ExpressRoute avec un circuit ExpressRoute et un Peering privé.

1. Établir la connectivité VPN en suivant les étapes décrites dans cet article.

### <a name="traffic-from-on-premises-networks-to-azure"></a>Trafic de réseaux locaux vers Azure

Pour le trafic de réseaux locaux vers Azure, les préfixes Azure sont publiés via le protocole BGP de peering privé ExpressRoute et le protocole BGP de VPN. Cela aboutit à deux itinéraires réseau (chemins) vers Azure à partir des réseaux locaux :

• Un itinéraire réseau sur le chemin protégé par IPsec.

• Un itinéraire réseau directement via ExpressRoute sans protection IPsec.

Pour appliquer le chiffrement à la communication, vous devez vous assurer que les itinéraires Azure qui passent par la passerelle VPN locale sont préférés au chemin ExpressRoute direct pour le réseau connecté par VPN dans la **figure 1**.

### <a name="traffic-from-azure-to-on-premises-networks"></a>Trafic d’Azure vers des réseaux locaux

La même condition s’applique au trafic d’Azure vers les réseaux locaux. Il existe deux moyens de faire en sorte que le chemin IPsec soit préféré au chemin ExpressRoute direct (sans IPsec) :

• **Publiez des préfixes plus spécifiques sur la session BGP VPN pour le réseau connecté par VPN**. Vous pouvez publier une plus grande plage qui englobe le réseau connecté par VPN par Peering privé ExpressRoute, puis des plages plus spécifiques dans la session BGP VPN. Par exemple, publiez 10.0.0.0/16 par ExpressRoute et 10.0.1.0/24 par VPN.

• **Publiez des préfixes disjoints pour VPN et ExpressRoute**. Si les plages de réseaux connectés par VPN sont disjointes d’autres réseaux connectés par ExpressRoute, vous pouvez publier les préfixes respectivement dans les sessions VPN et BGP ExpressRoute. Par exemple, publiez 10.0.0.0/24 par ExpressRoute et 10.0.1.0/24 par VPN.

Dans ces deux exemples, Azure envoie le trafic à 10.0.1.0/24 via la connexion VPN, plutôt que directement par ExpressRoute sans protection VPN.

>[!Warning]
>Si vous publiez les mêmes préfixes sur les connexions ExpressRoute et VPN, Azure utilisera directement le chemin ExpressRoute sans protection VPN.
>

## <a name="portal-steps"></a><a name="portal"></a>Étapes dans le portail

1. Configurez une connexion site à site. Pour connaître les étapes à suivre, consultez l’article [Configuration de site à site](vpn-gateway-howto-site-to-site-resource-manager-portal.md). Veillez à sélectionner une référence SKU de passerelle redondante interzone pour la passerelle. 

   Les références SKU redondantes interzones comportent « AZ » à la fin de la référence SKU. Par exemple, **VpnGw1AZ**. Les passerelles redondantes interzone sont uniquement disponibles dans les régions où le service de zones de disponibilité est proposé. Pour plus d'informations sur les régions dans lesquelles nous prenons en charge les zones de disponibilité, consultez [Régions prenant en charge les zones de disponibilité](../availability-zones/az-region.md).

   :::image type="content" source="media/site-to-site-vpn-private-peering/gateway.png" alt-text="Adresses IP privées de passerelle":::
1. Activez les adresses IP privées sur la passerelle. Sélectionnez **Configuration**, puis définissez **Adresses IP privées de passerelle** sur **Activé**. Cliquez sur **Enregistrer** pour enregistrer vos modifications.
1. Dans la page **Vue d’ensemble**, sélectionnez **Afficher plus** pour afficher l’adresse IP privée. Notez ces informations pour une utilisation ultérieure dans les étapes de configuration.

   :::image type="content" source="media/site-to-site-vpn-private-peering/gateway-overview.png" alt-text="Page de présentation" lightbox="media/site-to-site-vpn-private-peering/gateway-overview.png":::
1. Pour activer **Utiliser l’adresse IP privée Azure** sur la connexion, sélectionnez **Configuration**. Définissez **Utiliser l’adresse IP privée Azure** sur **Activé**, puis sélectionnez **Enregistrer**.

   :::image type="content" source="media/site-to-site-vpn-private-peering/connection.png" alt-text="Adresses IP privées de passerelle - Activé":::
1. À partir de votre pare-feu, exécutez une commande ping sur l’adresse IP privée que vous avez notée à l’étape 3. L’adresse IP privée doit être accessible via le peering privé ExpressRoute.
1. Utilisez cette adresse IP privée comme adresse IP distante sur votre pare-feu local pour établir le tunnel site à site via le peering privé ExpressRoute.

## <a name="powershell-steps"></a><a name="powershell"></a>étapes PowerShell

1. Configurez une connexion site à site. Pour connaître les étapes à suivre, consultez l’article [Configurer un VPN site à site](vpn-gateway-howto-site-to-site-resource-manager-portal.md). Veillez à sélectionner une référence SKU de passerelle redondante interzone pour la passerelle. Les références SKU redondantes interzones comportent « AZ » à la fin de la référence SKU. Par exemple, VpnGw1AZ.
1. Définissez l’indicateur pour utiliser l’adresse IP privée sur la passerelle à l’aide des commandes PowerShell suivantes :

   ```azurepowershell-interactive
   $Gateway = Get-AzVirtualNetworkGateway -Name <name of gateway> -ResourceGroup <name of resource group>

   Set-AzVirtualNetworkGateway -VirtualNetworkGateway $Gateway -EnablePrivateIpAddress $true
   ```

   Vous devez voir une adresse IP privée et publique. Notez l’adresse IP sous la section « TunnelIpAddresses » de la sortie. Vous utiliserez cette information dans une prochaine étape.
1. Définissez la connexion pour utiliser l’adresse IP privée à l’aide de la commande PowerShell suivante :

   ```azurepowershell-interactive
   $Connection = get-AzVirtualNetworkGatewayConnection -Name <name of the connection> -ResourceGroupName <name of resource group>

   Set-AzVirtualNetworkGatewayConnection --VirtualNetworkGatewayConnection $Connection -UseLocalAzureIpAddress $true
   ```
1. À partir de votre pare-feu, exécutez une commande ping sur l’adresse IP privée que vous avez notée à l’étape 2. Elle doit être accessible via l’appairage privé ExpressRoute.
1. Utilisez cette adresse IP privée comme adresse IP distante sur votre pare-feu local pour établir le tunnel site à site via le peering privé ExpressRoute.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la passerelle VPN, consultez [Qu’est-ce qu’une passerelle VPN ?](vpn-gateway-about-vpngateways.md)
