---
title: Configurer et valider des connexions de réseaux virtuels ou de VPN
description: Guide pas à pas pour configurer et valider différents déploiements de VPN et de réseaux virtuels Azure
services: virtual-network
documentationcenter: na
author: v-miegge
manager: dcscontentpm
editor: ''
ms.assetid: 0433a4f4-b5a0-476d-b398-1506c57eafa2
ms.service: virtual-network
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/28/2019
ms.author: kaushika
ms.openlocfilehash: d6532747c50311ada4df6a0038bd0e05f4d9ce31
ms.sourcegitcommit: 30505c01d43ef71dac08138a960903c2b53f2499
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92089687"
---
# <a name="configure-and-validate-virtual-network-or-vpn-connections"></a>Configurer et valider des connexions de réseaux virtuels ou de VPN

Ce guide pas à pas fournit des instructions pour configurer et valider différents déploiements de VPN et de réseaux virtuels Azure. Les scénarios incluent les itinéraires de transit, les connexions réseau à réseau, le protocole BGP (Border Gateway Protocol), les connexions multisites et les connexions point à site.

Les passerelles VPN Azure offrent une certaine flexibilité quant à l'agencement de pratiquement tout type de topologie de réseau virtuel connecté dans Azure. Par exemple, vous pouvez connecter des réseaux virtuels :

- Situés dans des régions différentes
- Appartenant à des types de réseaux virtuels différents (Azure Resource Manager ou classique)
- Dans Azure ou dans un environnement hybride local
- Dans le cadre d'abonnements différents 

## <a name="network-to-network-vpn-connection"></a>Connexion VPN réseau à réseau

La connexion d'un réseau virtuel à un autre réseau virtuel (réseau à réseau) via un VPN est semblable à la connexion d'un réseau virtuel à un emplacement local. Les deux types de connectivité utilisent une passerelle VPN pour fournir un tunnel sécurisé via IPsec et IKE. Les réseaux virtuels peuvent être situés dans des régions identiques ou différentes et appartenir à des abonnements identiques ou différents.

![Connexion VPN réseau à réseau avec IPsec](./media/virtual-network-configure-vnet-connections/4034386_en_2.png)
 
Si vos réseaux virtuels se trouvent dans la même région, vous pouvez envisager de les connecter à l'aide de l'appairage de réseaux virtuels. L'appairage de réseaux virtuels n'utilise pas de passerelle VPN. Il augmente le débit et diminue la latence. Pour configurer une connexion d'appairage de réseaux virtuels, sélectionnez **Configurer et valider l'appairage de réseaux virtuels** .

Si vos réseaux virtuels ont été créés à l'aide du modèle de déploiement Azure Resource Manager, sélectionnez **Configurer et valider un réseau virtuel Resource Manager sur une connexion de réseau virtuel Resource Manager** pour configurer une connexion VPN.

Si l'un des réseaux virtuels a été créé à l'aide du modèle de déploiement Azure Classic et l'autre via Resource Manager, sélectionnez **Configurer et valider un réseau virtuel classique sur une connexion de réseau virtuel Resource Manager** pour configurer une connexion VPN.

### <a name="configure-virtual-network-peering-for-two-virtual-networks-in-the-same-region"></a>Configurer l'appairage de réseaux virtuels pour deux réseaux virtuels de la même région

Avant de commencer à implémenter et à configurer l'appairage de réseaux virtuels Azure, assurez-vous que vous remplissez les conditions préalables suivantes :

* Les réseaux virtuels homologués doivent se trouver dans la même région Azure.
* Les réseaux virtuels appairés doivent disposer d'espaces d'adressage IP qui ne se chevauchent pas.
* Le peering se fait entre deux réseaux virtuels. Il n’y a aucune relation transitive dérivée entre les appairages. Par exemple, si VNetA est appairé à VNetB et que VNetB est appairé à VNetC, VNetA n'est *pas* appairé à VNetC.

Si vous remplissez les conditions requises, vous pouvez suivre le [Didacticiel : Connecter des réseaux virtuels à l'aide de l'appairage de réseaux virtuels à partir du portail Azure](https://docs.microsoft.com/azure/virtual-network/virtual-network-create-peering) pour créer et configurer l'appairage.

Pour vérifier la configuration de l'appairage, utilisez la méthode suivante :

1. Connectez-vous au [portail Azure](https://portal.azure.com/) à l'aide d'un compte doté des [rôles et autorisations](virtual-network-manage-peering.md#permissions) nécessaires.
2. Dans la boîte de dialogue contenant le texte **Rechercher des ressources** en haut du portail Azure, tapez **réseaux virtuels** . Quand la mention **Réseaux virtuels** apparaît dans les résultats de recherche, sélectionnez-la.
3. Dans le panneau **Réseaux virtuels** qui s'affiche, sélectionnez le réseau virtuel pour lequel vous souhaitez créer un appairage.
4. Dans le volet qui s'affiche pour le réseau virtuel, accédez à la section **Paramètres** et sélectionnez **Appairages** .
5. Sélectionnez un appairage et consultez les résultats de la configuration.

![Sélections pour vérifier la configuration d'un appairage de réseaux virtuels](./media/virtual-network-configure-vnet-connections/4034496_en_1.png)
 
Pour Azure PowerShell, exécutez la commande [Get-AzureRmVirtualNetworkPeering](https://docs.microsoft.com/powershell/module/azurerm.network/get-azurermvirtualnetworkpeering?view=azurermps-4.1.0) afin d'obtenir l'appairage de réseaux virtuels. Voici un exemple :

```
PS C:\Users\User1> Get-AzureRmVirtualNetworkPeering -VirtualNetworkName Vnet10-01 -ResourceGroupName dev-vnets
Name                             : LinkToVNET10-02
Id                               : /subscriptions/GUID/resourceGroups/dev-vnets/providers/Microsoft.Network/virtualNetworks/VNET10-01/virtualNetworkPeerings/LinkToVNET10-0
2
Etag                             : W/"GUID"
ResourceGroupName                : dev-vnets
VirtualNetworkName               : vnet10-01
ProvisioningState                : Succeeded
RemoteVirtualNetwork             : {
                                  "Id": "/subscriptions/GUID/resourceGroups/DEV-VNET
                                   s/providers/Microsoft.Network/virtualNetworks/VNET10-02"
                                   }
AllowVirtualNetworkAccess        : True
AllowForwardedTraffic            : False
AllowGatewayTransit              : False
UseRemoteGateways                : False
RemoteGateways                   : null
RemoteVirtualNetworkAddressSpace : null
```

### <a name="connect-a-resource-manager-virtual-network-to-another-resource-manager-virtual-network"></a>Connecter un réseau virtuel Resource Manager à un autre réseau virtuel Resource Manager

Vous pouvez directement configurer une connexion entre deux réseaux virtuels Resource Manager. Ou vous pouvez configurer la connexion à l'aide d'IPsec.

### <a name="configure-a-vpn-connection-between-resource-manager-virtual-networks"></a>Configurer une connexion VPN entre des réseaux virtuels Resource Manager

Pour configurer une connexion entre des réseaux virtuels Resource Manager sans IPSec, consultez [Configurer une connexion de passerelle VPN réseau à réseau à l'aide du portail Azure](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-howto-vnet-vnet-resource-manager-portal).

Pour configurer une connexion entre deux réseaux virtuels Resource Manager à l'aide d'IPSec, suivez les étapes 1 à 5 de la section [Créer une connexion site à site dans le portail Azure](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal) pour chaque réseau virtuel.

> [!Note]
> Ces étapes s'appliquent uniquement aux réseaux virtuels associés au même abonnement. Si vos réseaux virtuels figurent dans des abonnements différents, vous devrez utiliser PowerShell pour établir la connexion. Consultez l’article concernant [PowerShell](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-vnet-vnet-rm-ps).

### <a name="validate-the-vpn-connection-between-resource-manager-virtual-networks"></a>Valider une connexion VPN entre réseaux virtuels Resource Manager

![Connexion d'un réseau virtuel classique à un réseau virtuel Azure Resource Manager](./media/virtual-network-configure-vnet-connections/4034493_en_2.png)

Pour vérifier que votre connexion VPN est correctement configurée, suivez les instructions ci-dessous.

> [!Note] 
> Les chiffres qui suivent les composants de réseau virtuel correspondent à ceux du diagramme précédent.

1. Assurez-vous que les espaces d'adressage ne se chevauchent pas dans les réseaux virtuels connectés.
2. Vérifiez que la plage d'adresses du réseau virtuel Azure Resource Manager (1) est correctement définie dans l'instance **Objet de connexion** (4).
3. Vérifiez que la plage d'adresses du réseau virtuel Azure Resource Manager (6) est correctement définie dans l'instance **Objet de connexion** (3).
4. Vérifiez que les clés prépartagées correspondent aux objets de connexion.
5. Vérifiez que l'adresse IP virtuelle de la passerelle de réseau virtuel Azure Resource Manager (2) est correctement définie dans l'instance **Objet de connexion** (4).
6. Vérifiez que l'adresse IP virtuelle de la passerelle de réseau virtuel Azure Resource Manager (5) est correctement définie dans l'instance **Objet de connexion** (3).

### <a name="connect-a-classic-virtual-network-to-a-resource-manager-virtual-network"></a>Connecter un réseau virtuel classique à un réseau virtuel Resource Manager

Vous pouvez établir une connexion entre des réseaux virtuels associés à des abonnements différents et situés dans des régions différentes. Vous pouvez également connecter des réseaux virtuels déjà connectés à des réseaux locaux, à condition que le type de passerelle soit basé sur l'itinéraire.

Pour configurer une connexion entre un réseau virtuel classique et un réseau virtuel Resource Manager, consultez [Connecter des réseaux virtuels à partir de différents modèles de déploiement à l'aide du portail Azure](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-connect-different-deployment-models-portal).

![Diagramme représentant une connexion de réseau virtuel classique à un réseau virtuel Azure Resource Manager.](./media/virtual-network-configure-vnet-connections/4034389_en_2.png)

Pour vérifier la configuration lors de la connexion d'un réseau virtuel classique à un réseau virtuel Azure Resource Manager, suivez les instructions ci-dessous.

> [!Note] 
> Les chiffres qui suivent les composants de réseau virtuel correspondent à ceux du diagramme précédent. 

1. Assurez-vous que les espaces d'adressage ne se chevauchent pas dans les réseaux virtuels connectés.
2. Vérifiez que la plage d'adresses du réseau virtuel Azure Resource Manager (6) est correctement définie dans la définition du réseau local classique (3).
3. Vérifiez que la plage d'adresses du réseau virtuel classique (1) est correctement définie dans l'instance **Objet de connexion** d'Azure Resource Manager (4).
4. Vérifiez que l'adresse IP virtuelle de la passerelle de réseau virtuel classique (2) est correctement définie dans l'instance **Objet de connexion** d'Azure Resource Manager (4).
5. Vérifiez que la passerelle de réseau virtuel Azure Resource Manager (5) est correctement définie dans l'instance **Définition du réseau local** classique (3).
6. Vérifiez que les clés prépartagées correspondent sur les deux réseaux virtuels connectés :
   - Réseau virtuel classique : **Définition du réseau local** (3)
   - Réseau virtuel Azure Resource Manager : **Objet de connexion** (4)

## <a name="create-a-point-to-site-vpn-connection"></a>Créer une connexion VPN point à site

Une configuration point à site ( *P2S* dans le diagramme suivant) vous permet de connecter de manière sécurisée un ordinateur client individuel à un réseau virtuel. Les connexions point à site vous permettent de vous connecter à votre réseau virtuel à partir d'un emplacement distant, par exemple depuis votre domicile ou une salle de conférence. Elles sont également utiles lorsque seuls quelques clients doivent se connecter à un réseau virtuel. 

La connexion VPN point à site est initiée à partir de l'ordinateur client à l'aide du client VPN Windows natif. Les clients qui se connectent utilisent des certificats pour l’authentification.

![Connexion de point à site](./media/virtual-network-configure-vnet-connections/4034387_en_3.png)

Les connexions point à site ne nécessitent aucun périphérique VPN. Elles établissent la connexion VPN via le protocole SSTP (Secure Socket Tunneling Protocol). Vous pouvez établir une connexion point à site à un réseau virtuel à l'aide de divers outils et modèles de déploiement :

* [Configurer une connexion point à site à un réseau virtuel à l'aide du portail Azure](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal)
* [Configurer une connexion point à site à un réseau virtuel à l'aide du portail Azure (classique)](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-howto-point-to-site-classic-azure-portal)
* [Configurer une connexion point à site à un réseau virtuel à l'aide de PowerShell](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-howto-point-to-site-rm-ps)

### <a name="validate-your-point-to-site-connection"></a>Valider votre connexion point à site

L'article [Dépannage : problèmes de connexion point à site Azure](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-troubleshoot-vpn-point-to-site-connection-problems) passe en revue les problèmes courants de connexion point à site.

## <a name="create-a-multisite-vpn-connection"></a>Créer une connexion VPN multisite

Vous pouvez ajouter une connexion site à site ( *S2S* dans le diagramme suivant) à un réseau virtuel qui dispose déjà d'une connexion site à site, d'une connexion point à site ou d'une connexion réseau à réseau. Ce type de connexion est souvent appelé configuration *multisite* . 

![Connexion multisite](./media/virtual-network-configure-vnet-connections/4034497_en_2.png)

Actuellement, Azure utilise deux modèles de déploiement : Resource Manager et le déploiement classique. Les deux modèles ne sont pas entièrement compatibles entre eux. Pour configurer une connexion multisite avec des modèles différents, consultez les articles suivants :

* [Ajouter une connexion site à site à un réseau virtuel avec une connexion de passerelle VPN existante](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-howto-multi-site-to-site-resource-manager-portal)
* [Ajouter une connexion site à site à un réseau virtuel avec une connexion de passerelle VPN existante (classique)](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-multi-site)

> [!Note]
> Les étapes décrites dans ces articles ne s'appliquent pas aux configurations avec coexistence de connexions site à site et Azure ExpressRoute. Pour plus d'informations, consultez [Configurer la coexistence de connexions site à site et ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-howto-coexist-resource-manager).

## <a name="configure-transit-routing"></a>Configurer un itinéraire de transit

Un itinéraire de transit est un scénario de routage spécifique dans lequel vous connectez plusieurs réseaux dans le cadre d'une topologie de connexion en chaîne. Ce routage permet à des ressources de réseaux virtuels situées à chaque extrémité d'une chaîne de communiquer entre elles par le biais de réseaux virtuels. Sans itinéraire de transit, des réseaux ou appareils appairés via un hub ne peuvent pas communiquer.

### <a name="configure-transit-routing-in-a-point-to-site-connection"></a>Configurer un itinéraire de transit dans le cadre d'une connexion point à site

Imaginez un scénario dans lequel vous souhaitez configurer une connexion VPN site à site entre VNetA et VNetB. Vous pouvez également configurer un VPN point à site pour que le client se connecte à la passerelle de VNetA. Ensuite, vous pouvez activer un itinéraire de transit pour que les clients point à site puissent se connecter au réseau virtuel VNetB qui transite par le réseau virtuel VNetA. 

Ce scénario est pris en charge lorsque le protocole BGP est activé sur le VPN site à site entre les réseaux virtuels VNetA et VNetB. Pour plus d'informations, consultez [À propos du routage VPN point à site](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-point-to-site-routing).

### <a name="configure-transit-routing-in-an-expressroute-connection"></a>Configurer un itinéraire de transit dans une connexion ExpressRoute

Azure ExpressRoute vous permet d’étendre vos réseaux locaux au cloud de Microsoft par le biais d’une connexion privée dédiée assurée par un fournisseur de connectivité. Avec ExpressRoute, vous pouvez établir des connexions aux services de cloud computing Microsoft, par exemple Microsoft Azure, Microsoft 365 et Dynamics 365. Pour plus d’informations, voir [Appairage ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-introduction).

![Connexion d'un appairage privé ExpressRoute à des réseaux virtuels Azure](./media/virtual-network-configure-vnet-connections/4034395_en_1.png)

> [!Note]
> Si les réseaux virtuels VNetA et VNetB se trouvent dans la même région géopolitique, nous vous recommandons de [lier les deux réseaux virtuels au circuit ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-howto-linkvnet-arm) au lieu de configurer un itinéraire de transit. Si vos réseaux virtuels se trouvent dans des régions géopolitiques différentes, vous pouvez également les lier directement à votre circuit, à condition de disposer d'[ExpressRoute Premium](https://docs.microsoft.com/azure/expressroute/expressroute-faqs#expressroute-premium). 

Si vous disposez d'ExpressRoute et d'une coexistence site à site, l'itinéraire de transit n'est pas pris en charge. Pour plus d'informations, consultez [Configurer ExpressRoute et le mode site à site à l'aide de PowerShell](https://docs.microsoft.com/azure/expressroute/expressroute-howto-coexist-resource-manager).

Si vous avez activé ExpressRoute pour connecter vos réseaux locaux à un réseau virtuel Azure, vous pouvez activer un appairage des réseaux virtuels pour lesquels vous souhaitez disposer d'un itinéraire de transit. Pour permettre à vos réseaux locaux de se connecter au réseau virtuel distant, vous devez configurer un [appairage de réseaux virtuels](https://docs.microsoft.com/azure/virtual-network/virtual-network-peering-overview#gateways-and-on-premises-connectivity). 

> [!Note]
> L'appairage de réseaux virtuels est uniquement disponible pour les réseaux virtuels d'une même région.

Pour savoir si vous avez configuré un itinéraire de transit pour l'appairage de réseaux virtuels, suivez les instructions ci-dessous :

1. Connectez-vous au [portail Azure](https://portal.azure.com/) à l'aide d'un compte doté des [rôles et autorisations](virtual-network-manage-peering.md#permissions) nécessaires.
2. [Créez un appairage entre VNetA et VNetB](https://docs.microsoft.com/azure/virtual-network/virtual-network-create-peering), comme indiqué dans le diagramme précédent. 
3. Dans le volet qui s'affiche pour le réseau virtuel, accédez à la section **Paramètres** et sélectionnez **Appairages** .
4. Sélectionnez l'appairage que vous souhaitez afficher. Puis sélectionnez **Configuration** pour confirmer que vous avez activé l'option **Autoriser le transit par passerelle** sur le réseau virtuel VNetA connecté au circuit ExpressRoute et l'option **Utiliser la passerelle distante** sur le réseau virtuel VNetB distant non connecté au circuit ExpressRoute.

### <a name="configure-transit-routing-in-a-virtual-network-peering-connection"></a>Configurer un itinéraire de transit dans le cadre d'une connexion d'appairage de réseaux virtuels

Lorsque des réseaux virtuels sont homologués, vous pouvez également configurer la passerelle du réseau virtuel homologué en tant que point de transit vers un réseau local. Pour configurer un itinéraire de transit dans le cadre d'un appairage de réseaux virtuels, consultez [Connexions réseau à réseau](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-vnet-vnet-rm-ps?toc=/azure/virtual-network/toc.json).

> [!Note]
> Le transit de la passerelle n’est pas pris en charge dans la relation d’appairage entre des réseaux virtuels créés via des modèles de déploiement différents. Les deux réseaux virtuels de la relation d'appairage doivent avoir été créés via Resource Manager pour que le transit par passerelle fonctionne.

Pour savoir si vous avez configuré un itinéraire de transit pour l'appairage de réseaux virtuels, suivez les instructions ci-dessous :

1. Connectez-vous au [portail Azure](https://portal.azure.com/) à l'aide d'un compte doté des [rôles et autorisations](virtual-network-manage-peering.md#permissions) nécessaires.
2. Dans la boîte de dialogue contenant le texte **Rechercher des ressources** en haut du portail Azure, tapez **réseaux virtuels** . Quand la mention **Réseaux virtuels** apparaît dans les résultats de recherche, sélectionnez-la.
3. Dans le panneau **Réseaux virtuels** qui s'affiche, sélectionnez le réseau virtuel pour lequel vous souhaitez vérifier le paramètre d'appairage.
4. Dans le volet qui s'affiche pour le réseau virtuel que vous avez sélectionné, accédez à la section **Paramètres** et sélectionnez **Appairages** .
5. Sélectionnez l'appairage que vous souhaitez afficher. Vérifiez que vous avez activé les options **Autoriser le transit par passerelle** et **Utiliser la passerelle distante** sous **Configuration** .

![Sélections pour vérifier que vous avez configuré un itinéraire de transit pour l'appairage de réseaux virtuels](./media/virtual-network-configure-vnet-connections/4035414_en_1.png)

### <a name="configure-transit-routing-in-a-network-to-network-connection"></a>Configurer un itinéraire de transit dans le cadre d'une connexion réseau à réseau

Pour configurer un itinéraire de transit entre réseaux virtuels, vous devez activer le protocole BGP sur toutes les connexions réseau à réseau intermédiaires à l'aide du modèle de déploiement Resource Manager et de PowerShell. Pour obtenir des instructions, consultez [Configurer le protocole BGP sur des passerelles VPN Azure à l'aide de PowerShell](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-bgp-resource-manager-ps).

Le trafic de transit par le biais de passerelles VPN Azure est possible à l'aide du modèle de déploiement classique, mais il s'appuie sur des espaces d'adressage définis de manière statique dans le fichier de configuration réseau. Le protocole BGP n'est pas encore pris en charge par les réseaux virtuels et passerelles VPN Azure via le modèle de déploiement classique. Sans le protocole BGP, la définition manuelle d'espaces d'adressage de transit peut entraîner des erreurs et n'est donc pas recommandée.

> [!Note]
> Vous devez configurer les connexions réseau à réseau classiques à l'aide du portail Azure Classic ou d'un fichier de configuration réseau sur le portail Classic. Vous ne pouvez pas créer ou modifier un réseau virtuel classique via le modèle de déploiement Azure Resource Manager ou le portail Azure. Pour plus d'informations sur l'itinéraire de transit relatif aux réseaux virtuels classiques, consultez le [blog des développeurs Microsoft](https://blogs.msdn.microsoft.com/igorpag/2015/10/01/hubspoke-daisy-chain-and-full-mesh-vnet-topologies-in-azure-arm-using-vpn-v1/).

### <a name="configure-transit-routing-in-a-site-to-site-connection"></a>Configurer un itinéraire de transit dans le cadre d'une connexion site à site

Pour configurer un itinéraire de transit entre votre réseau local et un réseau virtuel via une connexion site à site, vous devez activer le protocole BGP sur toutes les connexions site à site intermédiaires à l'aide du modèle de déploiement Resource Manager et de PowerShell. Pour obtenir des instructions, consultez [Configurer le protocole BGP sur des passerelles VPN Azure à l'aide de PowerShell](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-bgp-resource-manager-ps).

Le trafic de transit par le biais de passerelles VPN Azure est possible à l'aide du modèle de déploiement classique, mais il s'appuie sur des espaces d'adressage définis de manière statique dans le fichier de configuration réseau. Le protocole BGP n'est pas encore pris en charge par les réseaux virtuels et passerelles VPN Azure via le modèle de déploiement classique. Sans le protocole BGP, la définition manuelle d'espaces d'adressage de transit peut entraîner des erreurs et n'est donc pas recommandée.

> [!Note]
> Vous devez configurer les connexions site à site classiques à l'aide du portail Azure Classic ou d'un fichier de configuration réseau sur le portail Classic. Vous ne pouvez pas créer ou modifier un réseau virtuel classique via le modèle de déploiement Azure Resource Manager ou le portail Azure. Pour plus d'informations sur l'itinéraire de transit relatif aux réseaux virtuels classiques, consultez le [blog des développeurs Microsoft](https://blogs.msdn.microsoft.com/igorpag/2015/10/01/hubspoke-daisy-chain-and-full-mesh-vnet-topologies-in-azure-arm-using-vpn-v1/).

## <a name="configure-bgp-for-a-vpn-gateway"></a>Configurer BGP pour une passerelle VPN

BGP est le protocole de routage standard utilisé sur Internet pour échanger des informations de routage et d'accessibilité entre plusieurs réseaux. Lorsque le protocole BGP est utilisé dans le contexte des réseaux virtuels Azure, il active les passerelles VPN Azure et vos périphériques VPN locaux, que l'on appelle des pairs ou voisins BGP. Ceux-ci échangent des « itinéraires » informent les deux passerelles sur la disponibilité et l’accessibilité des préfixes pour le passage à travers les passerelles ou routeurs impliqués. 

Le protocole BGP assure également le routage de transit entre plusieurs réseaux en propageant les itinéraires qu'une passerelle BGP obtient d'un pair BGP à tous les autres pairs BGP. Pour plus d'informations, consultez [Vue d'ensemble du protocole BGP avec les passerelles VPN Azure](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-bgp-overview).

### <a name="configure-bgp-for-a-vpn-connection"></a>Configurer BGP pour une connexion VPN

Pour configurer une connexion VPN utilisant le protocole BGP, consultez [Configurer le protocole BGP sur des passerelles VPN Azure à l'aide de PowerShell](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-bgp-resource-manager-ps).

Activez le protocole BGP sur la passerelle de réseau virtuel en créant un numéro de système autonome. Les passerelles de base ne prennent pas en charge le protocole BGP. Pour vérifier la référence SKU de la passerelle, accédez à la section **Vue d'ensemble** du panneau **Passerelle VPN** dans le portail Azure. Si vous disposez d'une référence SKU **De base** , vous devez la remplacer (voir [Redimensionner la passerelle](https://docs.microsoft.com/powershell/module/azurerm.network/resize-azurermvirtualnetworkgateway?view=azurermps-4.1.0&viewFallbackFrom=azurermps-4.0.0)) par **VpnGw1** . 

La vérification de la référence SKU peut entraîner un temps d'arrêt de 20 à 30 minutes. Dès que la passerelle dispose de la référence SKU qui convient, vous pouvez ajouter le numéro de système autonome via la cmdlet PowerShell [Set-AzureRmVirtualNetworkGateway](https://docs.microsoft.com/powershell/module/azurerm.network/set-azurermvirtualnetworkgateway?view=azurermps-3.8.0). Une fois le numéro AS configuré, une adresse IP de pair BGP pour la passerelle est fournie automatiquement.

Vous devez fournir manuellement à `LocalNetworkGateway` un numéro de système autonome et une adresse de pair BGP. Vous pouvez définir les valeurs `ASN` et `-BgpPeeringAddress` à l'aide de la cmdlet PowerShell [New-AzureRmLocalNetworkGateway](https://docs.microsoft.com/powershell/module/azurerm.network/new-azurermlocalnetworkgateway?view=azurermps-4.1.0) ou [Set-AzureRmLocalNetworkGateway](https://docs.microsoft.com/powershell/module/azurerm.network/set-azurermlocalnetworkgateway?view=azurermps-4.1.0). Certains numéros de système autonome sont réservés à Azure et vous ne pouvez pas les utiliser de la manière décrite dans [À propos du protocole BGP avec une passerelle VPN Azure](../vpn-gateway/vpn-gateway-bgp-overview.md#faq).

Le protocole BGP doit être activé sur l'objet de connexion. Vous pouvez définir la valeur `-EnableBGP` sur `$True` via la cmdlet [New-AzureRmVirtualNetworkGatewayConnection](https://docs.microsoft.com/powershell/module/azurerm.network/new-azurermvirtualnetworkgatewayconnection?view=azurermps-4.1.0) ou [Set-AzureRmVirtualNetworkGatewayConnection](https://docs.microsoft.com/powershell/module/azurerm.network/set-azurermvirtualnetworkgatewayconnection?view=azurermps-4.1.0).

### <a name="validate-the-bgp-configuration"></a>Valider la configuration BGP

Pour savoir si le protocole BGP est correctement configuré, vous pouvez exécuter les cmdlets `get-AzureRmVirtualNetworkGateway` et `get-AzureRmLocalNetworkGateway`. Vous verrez ensuite une sortie liée au protocole BGP dans la partie `BgpSettingsText`. Par exemple :

```
{

"Asn": AsnNumber,

"BgpPeeringAddress": "IP address",

"PeerWeight": 0

}
```

## <a name="create-a-highly-available-activeactive-vpn-connection"></a>Créer une connexion VPN active/active hautement disponible

Les principales différences entre les passerelles en modes actif/actif et actif/passif sont les suivantes :

* Vous devez créer deux configurations IP de passerelle avec deux adresses IP publiques.
* Vous devez définir l'indicateur **EnableActiveActiveFeature** .
* La référence SKU de la passerelle doit être **VpnGw1** , **VpnGw2** ou **VpnGw3** .

Pour obtenir une haute disponibilité des connexions intersites et réseau à réseau, vous devez déployer plusieurs passerelles VPN, et établir différentes connexions parallèles entre vos réseaux et Azure. Pour une vue d'ensemble des options de connectivité et de topologie, consultez [Configuration haute disponibilité pour la connectivité intersites et réseau à réseau](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-highlyavailable).

Pour créer des connexions intersites et réseau à réseau en mode actif/actif, suivez les instructions de la section [Configurer des connexions VPN S2S en mode actif/actif avec des passerelles VPN Azure](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-activeactive-rm-powershell) afin de configurer une passerelle VPN Azure en mode actif/actif.

> [!Note]  
> * Lorsque vous ajoutez des adresses à la passerelle de réseau local pour le mode actif/actif activé pour le protocole BGP, *ajoutez uniquement les adresses /32 des pairs BGP* . Si vous ajoutez des autres adresses, celles-ci sont considérées comme des itinéraires statiques et ont priorité sur les itinéraires BGP.
> * Vous devez utiliser des numéros de système autonome BGP différents pour les réseaux locaux qui se connectent à Azure. (En cas de numéros de système autonome identiques, vous devez modifier celui de votre réseau virtuel si votre périphérique VPN local l'utilise déjà pour s'appairer à des voisins BGP.)

## <a name="change-an-azure-vpn-gateway-type-after-deployment"></a>Modifier un type de passerelle VPN Azure après le déploiement

Vous ne pouvez pas procéder au remplacement direct d'un type de passerelle de réseau virtuel Azure basé sur une stratégie par un type basé sur un itinéraire, ou inversement. Vous devez d'abord supprimer la passerelle. Suite à cela, l'adresse IP et la clé prépartagée ne seront pas conservées. Vous pourrez alors créer une nouvelle passerelle du type souhaité. 

Pour supprimer et créer une passerelle, procédez comme suit :

1. Supprimez toutes les connexions associées à la passerelle d’origine.
2. Supprimez la passerelle à l'aide du portail Azure, de PowerShell ou de PowerShell classique : 
   * [Supprimer une passerelle de réseau virtuel à l'aide du portail Azure](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-delete-vnet-gateway-portal)
   * [Supprimer une passerelle de réseau virtuel à l'aide de PowerShell](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-delete-vnet-gateway-powershell)
   * [Supprimer une passerelle de réseau virtuel à l'aide de PowerShell (classique)](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-delete-vnet-gateway-classic-powershell)
3. Suivez les étapes de la section [Créer la passerelle VPN](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md#VNetGateway) pour créer la nouvelle passerelle du type souhaité et terminer l'installation du VPN.

> [!Note]
> Ce processus prend environ 60 minutes.

## <a name="next-steps"></a>Étapes suivantes

* [Résolution des problèmes de connectivité entre machines virtuelles Azure](https://docs.microsoft.com/azure/virtual-network/virtual-network-troubleshoot-connectivity-problem-between-vms)

