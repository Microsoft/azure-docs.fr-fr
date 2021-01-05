---
title: 'Tutoriel - Connecter un réseau local à un réseau virtuel Azure : Portail Azure'
description: Créez une connexion IPsec de passerelle VPN de site à site entre votre réseau local et un réseau virtuel Azure via l’Internet public à l’aide du portail.
titleSuffix: Azure VPN Gateway
author: cherylmc
ms.author: cherylmc
ms.service: vpn-gateway
ms.topic: tutorial
ms.date: 12/04/2020
ms.openlocfilehash: d078d4b106136c95d67eb467bc16e22e2bb91c5c
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96747283"
---
# <a name="tutorial-create-a-site-to-site-connection-in-the-azure-portal"></a>Tutoriel : Création d’une connexion de site à site dans le portail Azure

Les passerelles VPN Azure fournissent une connectivité entre les locaux du client et Azure. Ce tutoriel vous explique comment utiliser le portail Azure pour créer une connexion de passerelle VPN de site à site à partir de votre réseau local vers le réseau virtuel. Vous pouvez également créer cette configuration en utilisant [Azure PowerShell](vpn-gateway-create-site-to-site-rm-powershell.md) ou [Azure CLI](vpn-gateway-howto-site-to-site-resource-manager-cli.md).

:::image type="content" source="./media/tutorial-site-to-site-portal/diagram.png" alt-text="Schéma de connexion intersite d’une passerelle VPN site à site":::

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créez un réseau virtuel
> * Créer une passerelle VPN
> * Créer une passerelle de réseau local
> * Créer une connexion VPN
> * Vérifier la connexion
> * Connexion à une machine virtuelle

## <a name="prerequisites"></a>Prérequis

* Compte Azure avec un abonnement actif. Si vous n’en avez pas, [créez-en une gratuitement](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).
* Veillez à disposer d’un périphérique VPN compatible et à être entouré d’une personne en mesure de le configurer. Pour plus d’informations sur les périphériques VPN compatibles et la configuration de votre périphérique, consultez l’article [À propos des périphériques VPN](vpn-gateway-about-vpn-devices.md).
* Vérifiez que vous disposez d’une adresse IPv4 publique exposée en externe pour votre périphérique VPN.
* Si vous ne maîtrisez pas les plages d’adresses IP situées dans votre configuration de réseau local, vous devez contacter une personne en mesure de vous aider. Lorsque vous créez cette configuration, vous devez spécifier les préfixes des plages d’adresses IP qu’Azure acheminera vers votre emplacement local. Aucun des sous-réseaux de votre réseau local ne peut chevaucher les sous-réseaux du réseau virtuel auquel vous souhaitez vous connecter.

## <a name="create-a-virtual-network"></a><a name="CreatVNet"></a>Créer un réseau virtuel

Créez un réseau virtuel à l’aide des valeurs suivantes :

* **Groupe de ressources :** TestRG1
* **Nom :** VNet1
* **Région :** (États-Unis) USA Est
* **Espace d’adressage IPv4 :** 10.1.0.0/16
* **Nom du sous-réseau** : FrontEnd
* **Espace d’adressage du sous-réseau :** 10.1.0.0/24

[!INCLUDE [About cross-premises addresses](../../includes/vpn-gateway-cross-premises.md)]

[!INCLUDE [Create a virtual network](../../includes/vpn-gateway-basic-vnet-rm-portal-include.md)]

## <a name="create-a-vpn-gateway"></a><a name="VNetGateway"></a>Créer une passerelle VPN

Dans cette étape, vous créez la passerelle de réseau virtuel de votre réseau virtuel. La création d’une passerelle nécessite généralement au moins 45 minutes, selon la référence SKU de passerelle sélectionnée.

### <a name="about-the-gateway-subnet"></a>À propos du sous-réseau de passerelle

[!INCLUDE [About gateway subnets](../../includes/vpn-gateway-about-gwsubnet-portal-include.md)]

### <a name="create-the-gateway"></a>Créer la passerelle

Créez une passerelle VPN à l’aide des valeurs suivantes :

* **Nom :** VNet1GW
* **Région :** USA Est
* **Type de passerelle :** VPN
* **Type de VPN :** basé sur la route
* **SKU :** VpnGw1
* **Génération :** Génération1
* **Réseau virtuel :** VNet1
* **Plage d’adresses du sous-réseau de passerelle :** 10.1.255.0/27
* **Adresse IP publique :** Création
* **Nom de l'adresse IP publique :** VNet1GWpip
* **Activer le mode actif/actif :** Désactivé
* **Configurer BGP :** Désactivé

[!INCLUDE [Create a vpn gateway](../../includes/vpn-gateway-add-gw-rm-portal-include.md)]

[!INCLUDE [NSG warning](../../includes/vpn-gateway-no-nsg-include.md)]

### <a name="view-the-public-ip-address"></a><a name="view"></a>Afficher l’adresse IP publique

Vous pouvez voir l’adresse IP publique de la passerelle dans la page **Vue d’ensemble** de votre passerelle.

:::image type="content" source="./media/tutorial-create-gateway-portal/address.png" alt-text="Page de présentation":::

Pour voir d’autres informations sur l’objet d’adresse IP publique, cliquez sur le lien de nom/adresse IP à côté de **Adresse IP publique**.

## <a name="create-a-local-network-gateway"></a><a name="LocalNetworkGateway"></a>Créer une passerelle de réseau local

La passerelle réseau locale est un objet spécifique qui représente votre emplacement local (le site) à des fins de routage. Donnez au site un nom auquel Azure pourra se référer, puis spécifiez l’adresse IP du périphérique VPN local vers lequel vous allez créer une connexion. Spécifiez également les préfixes d’adresses IP qui seront acheminés via la passerelle VPN vers le périphérique VPN. Les préfixes d’adresses que vous spécifiez sont les préfixes situés sur votre réseau local. En cas de modification du réseau ou si vous devez modifier l’adresse IP publique de l’appareil VPN, il est simple de mettre à jour les valeurs ultérieurement.

Créez une passerelle de réseau local à l’aide des valeurs suivantes :

* **Nom :** Site1
* **Groupe de ressources :** TestRG1
* **Emplacement :** USA Est

[!INCLUDE [Add a local network gateway](../../includes/vpn-gateway-add-local-network-gateway-portal-include.md)]

## <a name="configure-your-vpn-device"></a><a name="VPNDevice"></a>Configurer votre périphérique VPN

Les connexions site à site vers un réseau local nécessitent un périphérique VPN. Dans cette étape, vous configurez votre périphérique VPN. Lorsque vous configurez votre périphérique VPN, vous avez besoin des valeurs suivantes :

* Une clé partagée. Il s’agit de la clé partagée spécifiée lors de la création de la connexion VPN de site à site. Dans nos exemples, nous utilisons une clé partagée basique. Nous vous conseillons de générer une clé plus complexe.
* L’adresse IP publique de votre passerelle de réseau virtuel. Vous pouvez afficher l’adresse IP publique à l’aide du portail Azure, de PowerShell ou de l’interface de ligne de commande. Pour rechercher l’adresse IP publique de votre passerelle VPN à l’aide du portail Azure, accédez à **Passerelles de réseau virtuel**, puis sélectionnez le nom de votre passerelle.

[!INCLUDE [Configure a VPN device](../../includes/vpn-gateway-configure-vpn-device-include.md)]

## <a name="create-a-vpn-connection"></a><a name="CreateConnection"></a>Créer une connexion VPN

Créez la connexion VPN de site à site entre votre passerelle de réseau virtuel et votre périphérique VPN local.

Créez une connexion à l’aide des valeurs suivantes :

* **Nom de passerelle de réseau local :** Site1
* **Nom de la connexion :** VNet1toSite1
* **Clé partagée :** pour cet exemple, nous utilisons abc123. Toutefois, vous pouvez utiliser n’importe quelle valeur compatible avec votre matériel VPN. L’important est que les valeurs soient les mêmes de part et d’autre de la connexion.

[!INCLUDE [Add a site-to-site connection](../../includes/vpn-gateway-add-site-to-site-connection-portal-include.md)]

## <a name="verify-the-vpn-connection"></a><a name="VerifyConnection"></a>Vérifier la connexion VPN

[!INCLUDE [Verify the connection](../../includes/vpn-gateway-verify-connection-portal-include.md)]

## <a name="connect-to-a-virtual-machine"></a><a name="connectVM"></a>Connexion à une machine virtuelle

[!INCLUDE [Connect to a VM](../../includes/vpn-gateway-connect-vm.md)]

## <a name="optional-steps"></a>Étapes facultatives

### <a name="add-additional-connections-to-the-gateway"></a><a name="addconnect"></a>Ajouter des connexions supplémentaires à la passerelle

Vous pouvez ajouter des connexions supplémentaires, à condition que les espaces d’adressage des connexions ne se chevauchent pas.

1. Pour ajouter une connexion, accédez à la passerelle VPN, puis sélectionnez **Connexions** pour ouvrir la page homonyme.
1. Cliquez sur **+Ajouter** pour ajouter votre connexion. Définissez le type de connexion sur Réseau virtuel à réseau virtuel (pour une connexion à une autre passerelle de réseau virtuel) ou sur Site à site, selon vos besoins.
1. Si vous voulez ajouter une connexion Site à site et que vous n’avez pas encore créé de passerelle de réseau local pour le site auquel vous souhaitez vous connecter, vous pouvez en créer une.
1. Spécifiez la clé partagée à utiliser, puis cliquez sur **OK** pour créer la connexion.

### <a name="resize-a-gateway-sku"></a><a name="resize"></a>Redimensionner une référence SKU de passerelle

Il existe des règles spécifiques relatives au redimensionnement et à la modification d’une référence SKU de passerelle. Dans cette section, nous allons redimensionner la référence SKU. Pour plus d’informations, consultez [Paramètres de la passerelle : Redimensionnement ou modification d’une référence SKU](vpn-gateway-about-vpn-gateway-settings.md#resizechange).

[!INCLUDE [resize a gateway](../../includes/vpn-gateway-resize-gw-portal-include.md)]

### <a name="reset-a-gateway"></a><a name="reset"></a>Réinitialiser une passerelle

La réinitialisation d’une passerelle VPN Azure est utile si vous perdez la connectivité VPN entre différents locaux sur un ou plusieurs tunnels VPN de site à site. Dans ce cas, vos périphériques VPN sur site fonctionnent tous correctement, mais ils ne sont pas en mesure d’établir des tunnels IPsec avec les passerelles VPN Azure.

[!INCLUDE [reset a gateway](../../includes/vpn-gateway-reset-gw-portal-include.md)]

### <a name="additional-configuration-considerations"></a><a name="connectVM"></a>Considérations importantes relatives à la configuration

Les configurations S2S peuvent être personnalisées de différentes façons. Pour plus d’informations, consultez les articles suivants :

* Pour plus d’informations sur le protocole BGP, consultez les articles [Vue d’ensemble du protocole BGP](vpn-gateway-bgp-overview.md) et [Comment configurer BGP](vpn-gateway-bgp-resource-manager-ps.md).
* Pour plus d’informations sur le tunneling forcé, consultez [À propos du tunneling forcé](vpn-gateway-forced-tunneling-rm.md).
* Pour plus d’informations sur les connexions haut actif-actif, consultez [Configuration haute disponibilité pour la connectivité entre les réseaux locaux et la connectivité entre deux réseaux virtuels](vpn-gateway-highlyavailable.md).
* Pour savoir comment limiter le trafic réseau aux ressources d’un réseau virtuel, consultez [Sécurité du réseau](../virtual-network/network-security-groups-overview.md).
* Pour plus d’informations sur la façon dont Azure achemine le trafic entre les ressources Azure, locales et Internet, consultez [Routage du trafic de réseau virtuel](../virtual-network/virtual-networks-udr-overview.md).

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous n’envisagez pas de continuer à utiliser cette application ni de passer au tutoriel suivant, supprimez ces ressources en effectuant les étapes suivantes :

1. Entrez le nom de votre groupe de ressources dans la zone **Rechercher** en haut du portail et sélectionnez-le dans les résultats de la recherche.

1. Sélectionnez **Supprimer le groupe de ressources**.

1. Entrez votre groupe de ressources dans **TAPER LE NOM DU GROUPE DE RESSOURCES**, puis sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Une fois que vous avez configuré une connexion S2S, vous pouvez ajouter une connexion P2S à la même passerelle.

> [!div class="nextstepaction"]
> [Connexions VPN de point à site](vpn-gateway-howto-point-to-site-resource-manager-portal.md)
