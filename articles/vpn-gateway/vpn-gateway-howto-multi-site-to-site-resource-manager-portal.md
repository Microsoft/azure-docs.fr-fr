---
title: 'Ajouter plusieurs connexions site à site de passerelle VPN à un réseau virtuel : Portail Azure'
description: Ajouter des connexions S2S multisites à une passerelle VPN qui dispose déjà d’une connexion
services: vpn-gateway
titleSuffix: Azure VPN Gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 10/27/2020
ms.author: cherylmc
ms.openlocfilehash: 92d39b0d39511571701fd092f641cb8ca3ae42c7
ms.sourcegitcommit: 9826fb9575dcc1d49f16dd8c7794c7b471bd3109
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2020
ms.locfileid: "92890146"
---
# <a name="add-additional-s2s-connections-to-a-vnet-azure-portal"></a>Ajouter des connexions S2S à un réseau virtuel : Portail Azure

> [!div class="op_single_selector"]
> * [Azure portal](vpn-gateway-howto-multi-site-to-site-resource-manager-portal.md)
> * [PowerShell (classique)](vpn-gateway-multi-site.md)
>

Cet article vous aide à ajouter des connexions site à site (S2S) à une passerelle VPN qui dispose déjà d’une connexion. Cette architecture est souvent appelée configuration « multisite ». Vous pouvez ajouter une connexion S2S à un réseau virtuel qui possède déjà une connexion S2S, une connexion de point à site ou une connexion de réseau virtuel à réseau virtuel. L’ajout de connexions est soumis à certaines limitations. Consultez la section [Configuration requise](#before) de cet article pour effectuer des vérifications avant de commencer votre configuration.

Cet article s’applique aux réseaux virtuels de Resource Manager qui contiennent une passerelle VPN RouteBased. Ces étapes ne s’appliquent pas aux nouvelles configurations de connexions coexistantes ExpressRoute/site à site. Toutefois, vous pouvez les suivre si vous ajoutez simplement une nouvelle connexion VPN à une configuration de coexistence. Consultez l’article [ExpressRoute/S2S coexisting connections](../expressroute/expressroute-howto-coexist-resource-manager.md) (Connexions coexistantes ExpressRoute/S2S) pour en savoir plus sur les connexions coexistantes.

## <a name="prerequisites"></a><a name="before"></a>Configuration requise

Vérifiez les points suivants :

* Vous n’effectuez pas une nouvelle configuration de connexions coexistantes ExpressRoute/passerelle VPN.
* Vous disposez d’un réseau virtuel qui a été créé selon le modèle de déploiement Resource Manager avec une connexion existante.
* La passerelle de réseau virtuel de votre réseau virtuel est de type RouteBased. Si vous avez une passerelle VPN de type PolicyBased, vous devez supprimer la passerelle de réseau virtuel et créer une passerelle VPN de type RouteBased.
* Les plages d’adresses ne se chevauchent pour aucun des réseaux virtuels auxquels ce réseau virtuel se connecte.
* Vous disposez d’un périphérique VPN compatible et d’une personne capable de le configurer. Consultez [À propos des périphériques VPN](vpn-gateway-about-vpn-devices.md). Si vous ne maîtrisez pas la configuration de votre appareil VPN ou les plages d’adresses IP mentionnées dans la configuration de votre réseau local, vous devez contacter une personne qui peut vous fournir ces informations.
* Vous disposez d’une adresse IP publique exposée en externe pour votre périphérique VPN.

## <a name="configure-a-connection"></a><a name="configure"></a>Configurer une connexion

1. Dans un navigateur, accédez au [portail Azure](https://portal.azure.com) et, si nécessaire, connectez-vous avec votre compte Azure.
1. Cliquez sur **Toutes les ressources**, repérez votre **passerelle de réseau virtuel** dans la liste des ressources, puis sélectionnez-la.
1. Dans la page **Passerelle de réseau virtuel**, sélectionnez **Connexions**.

   :::image type="content" source="./media/vpn-gateway-howto-multi-site-to-site-resource-manager-portal/connections.png" alt-text="Connexions de passerelle VPN":::
1. Dans la page **Connexions**, sélectionnez **+Ajouter**.
1. La boîte de dialogue **Ajouter une connexion** s’ouvre.

   :::image type="content" source="./media/vpn-gateway-howto-multi-site-to-site-resource-manager-portal/add-connection.png" alt-text="Page Ajouter une connexion":::
1. Dans la page **Ajouter une connexion**, renseignez les champs suivants :

   * **Nom** : nom que vous voulez donner au site pour lequel vous créez la connexion.
   * **Type de connexion :** sélectionnez **Site à site (IPsec)** .

## <a name="add-a-local-network-gateway"></a><a name="local"></a>Ajouter une passerelle de réseau local

1. Pour le champ **Passerelle de réseau local**, sélectionnez **_Choisir une passerelle de réseau local_ *_. La page _* Choisir une passerelle de réseau local** s’ouvre.
1. Sélectionnez **+ Créer** pour ouvrir la page **Créer une passerelle de réseau local**.

   :::image type="content" source="./media/vpn-gateway-howto-multi-site-to-site-resource-manager-portal/create-local-network-gateway.png" alt-text="Page Créer une passerelle de réseau local":::
1. Dans la page **Créer une passerelle de réseau local**, renseignez les champs suivants :

   * **Nom** : nom que vous voulez attribuer à la ressource de passerelle de réseau local.
   * **Point de terminaison :** adresse IP publique du périphérique VPN sur le site auquel vous voulez vous connecter, ou FQDN du point de terminaison.
   * **Espace d’adressage** : espace d’adressage qui doit être acheminé vers le nouveau site de réseau local.
1. Cliquez sur **OK** dans la page **Créer une passerelle de réseau local** pour enregistrer les modifications.

## <a name="add-the-shared-key"></a><a name="part3"></a>Ajouter la clé partagée

1. Après avoir créé la passerelle de réseau local, revenez à la page **Ajouter une connexion**.
1. Renseignez les champs restants. Pour la **Clé partagée (PSK)** , vous pouvez soit obtenir la clé partagée à partir de votre périphérique VPN, soit en créer une ici et configurer votre périphérique VPN de sorte qu’il utilise la même clé partagée. Il importe que les clés soient exactement les mêmes.

## <a name="create-the-connection"></a><a name="create"></a>Créer la connexion

1. Au bas de la page, sélectionnez **OK** pour créer la connexion. La création de la connexion commence immédiatement.
1. Une fois la connexion terminée, vous pouvez l’afficher et la vérifier.

## <a name="view-and-verify-the-vpn-connection"></a><a name="verify"></a>Afficher et vérifier la connexion VPN

[!INCLUDE [Verify the connection](../../includes/vpn-gateway-verify-connection-portal-include.md)]

## <a name="next-steps"></a>Étapes suivantes

Une fois la connexion achevée, vous pouvez ajouter des machines virtuelles à vos réseaux virtuels. Pour plus d’informations, consultez [Parcours d’apprentissage des machines virtuelles](/learn/paths/deploy-a-website-with-azure-virtual-machines/).
