---
title: Configurer une connexion de passerelle VPN de réseau virtuel à réseau virtuel Portail Azure
description: Créer une connexion de passerelle VPN entre des réseaux virtuels à l’aide de Resource Manager et du portail Azure.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 10/19/2020
ms.author: cherylmc
ms.openlocfilehash: fe0280e302882fd5e50830950b531ea9ca169618
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94660540"
---
# <a name="configure-a-vnet-to-vnet-vpn-gateway-connection-by-using-the-azure-portal"></a>Configurer une connexion de passerelle VPN de réseau virtuel à réseau virtuel à l’aide du portail Azure

Cet article vous explique comment connecter des réseaux virtuels avec une connexion de réseau virtuel à réseau virtuel. Les réseaux virtuels peuvent se trouver dans différentes régions et sous différents abonnements. Lors de la connexion de réseaux virtuels provenant de différents abonnements, les abonnements ne sont pas tenus d’être associés au même locataire Active Directory. 

:::image type="content" source="./media/vpn-gateway-howto-vnet-vnet-resource-manager-portal/vnet-vnet-diagram.png" alt-text="Diagramme de réseau virtuel à réseau virtuel":::

Les étapes mentionnées dans cet article s’appliquent au modèle de déploiement Azure Resource Manager et font appel au portail Azure. Vous pouvez créer cette configuration avec un outil ou un modèle de déploiement différent en utilisant les options décrites dans les articles suivants :

> [!div class="op_single_selector"]
> * [Azure portal](vpn-gateway-howto-vnet-vnet-resource-manager-portal.md)
> * [PowerShell](vpn-gateway-vnet-vnet-rm-ps.md)
> * [Azure CLI](vpn-gateway-howto-vnet-vnet-cli.md)
> * [Portail Azure (classique)](vpn-gateway-howto-vnet-vnet-portal-classic.md)
> * [Connexions entre différents modèles de déploiement - Portail Azure](vpn-gateway-connect-different-deployment-models-portal.md)
> * [Connexions entre différents modèles de déploiement - PowerShell](vpn-gateway-connect-different-deployment-models-powershell.md)
>
>

## <a name="about-connecting-vnets"></a>À propos de la connexion de réseaux virtuels

Les sections ci-dessous décrivent les différentes manières de se connecter à des réseaux virtuels.

### <a name="vnet-to-vnet"></a>Connexion entre deux réseaux virtuels

La configuration d’une connexion de réseau virtuel à réseau virtuel est un moyen simple de se connecter à des réseaux virtuels. La connexion entre deux réseaux virtuels est semblable à la création d’une connexion IPsec de site à site dans un emplacement local. Ces deux types de connectivité font appel à une passerelle VPN pour offrir un tunnel sécurisé avec Ipsec/IKE et communiquent de la même façon. Toutefois, ils diffèrent en termes de configuration de la passerelle réseau locale. 

Lorsque vous créez une connexion de réseau virtuel à réseau virtuel, l’espace d’adressage de la passerelle réseau locale est automatiquement créé et renseigné. Si vous mettez à jour l’espace d’adressage pour un réseau virtuel, l’autre réseau virtuel achemine automatiquement le trafic vers le nouvel espace d’adressage. Il est généralement plus rapide et plus simple de créer une connexion de réseau virtuel à réseau virtuel qu'une connexion de site à site.

### <a name="site-to-site-ipsec"></a>Site à site (IPsec)

Si vous travaillez avec une configuration réseau complexe, vous pouvez connecter vos réseaux virtuels à l’aide d'une connexion de [site à site](vpn-gateway-howto-site-to-site-resource-manager-portal.md). Lorsque vous suivez les étapes IPsec de site à site, vous créez et configurez manuellement les passerelles réseau locales. La passerelle de réseau local pour chaque réseau virtuel traite l’autre réseau virtuel comme un site local. Ces étapes vous permettent de spécifier un espace d’adressage supplémentaire pour la passerelle réseau locale afin d’acheminer le trafic. Si l’espace d’adressage pour un réseau virtuel est modifié, vous devez manuellement mettre à jour la passerelle réseau local correspondante.

### <a name="vnet-peering"></a>Peering de réseaux virtuels

Vous pouvez également connecter vos réseaux virtuels à l’aide du peering de réseaux virtuels. VNet Peering n’utilise pas de passerelle VPN et a d’autres contraintes En outre, la [tarification de VNet Peering](https://azure.microsoft.com/pricing/details/virtual-network) est différente de la [tarification de la passerelle VPN de réseau virtuel à réseau virtuel](https://azure.microsoft.com/pricing/details/vpn-gateway). Pour plus d’informations, consultez l’article [Peering de réseaux virtuels](../virtual-network/virtual-network-peering-overview.md).

## <a name="why-create-a-vnet-to-vnet-connection"></a>Pourquoi créer une connexion de réseau virtuel à réseau virtuel ?

Vous pouvez décider de connecter des réseaux virtuels à l’aide d’une connexion de réseau virtuel à réseau virtuel pour les raisons suivantes :

### <a name="cross-region-geo-redundancy-and-geo-presence"></a>Géo-redondance et présence géographique dans plusieurs régions

* Vous pouvez configurer la géoréplication ou la synchronisation avec une connectivité sécurisée sans passer par les points de terminaison accessibles sur Internet.
* Avec Azure Traffic Manager et l’équilibreur de charge Azure, vous pouvez configurer une charge de travail hautement disponible avec la géo-redondance dans plusieurs régions Azure. Par exemple, vous pouvez définir des groupes de disponibilité SQL Server Always On dans plusieurs régions Azure.

### <a name="regional-multi-tier-applications-with-isolation-or-administrative-boundaries"></a>Applications multiniveaux régionales avec limites d’isolement ou administratives

* Dans la même région, vous pouvez configurer des applications multiniveaux avec plusieurs réseaux virtuels interconnectés pour des besoins d’isolement ou d’administration.

Vous pouvez combiner une communication de réseau virtuel à réseau virtuel avec des configurations multisites. Ces configurations vous permettent d'établir des topologies réseau qui combinent une connectivité entre différents locaux et une connectivité entre différents réseaux virtuels, comme indiqué dans le schéma suivant :

:::image type="content" source="./media/vpn-gateway-howto-vnet-vnet-resource-manager-portal/connections-diagram.png" alt-text="Diagramme de connexions de réseau virtuel":::

Cet article explique comment connecter des réseaux virtuels avec une connexion de réseau virtuel à réseau virtuel. Lorsque vous suivez ces étapes dans le cadre d’un exercice, vous pouvez vous servir des valeurs des paramètres d’exemple suivantes. Dans l’exemple, les réseaux virtuels se trouvent dans le même abonnement, mais dans des groupes de ressources différents. Si vos réseaux virtuels existent dans des abonnements différents, vous ne pourrez pas créer la connexion dans le portail. Utilisez plutôt [PowerShell](vpn-gateway-vnet-vnet-rm-ps.md) ou [CLI](vpn-gateway-howto-vnet-vnet-cli.md). Pour plus d’informations sur les connexions de réseau virtuel à réseau virtuel, consultez le [Forum Aux Questions sur l’interconnexion de réseaux virtuels](#vnet-to-vnet-faq).

### <a name="example-settings"></a>Exemples de paramètres

**Valeurs pour VNet1 :**

* **Paramètres de réseau virtuel**
  * **Name** : VNet1
  * **Espace d’adressage** : 10.1.0.0/16
  * **Abonnement**: Sélectionnez l’abonnement que vous souhaitez utiliser.
  * **Groupe de ressources** : TestRG1
  * **Emplacement** : USA Est
  * **Sous-réseau**
    * **Name** : FrontEnd
    * **Plage d’adresses** : 10.1.0.0/24

* **Paramètres de passerelle de réseau virtuel**
  * **Name** : VNet1GW
  * **Groupe de ressources** : USA Est
  * **Génération** : Génération 1
  * **Type de passerelle** : Sélectionnez **VPN**.
  * **Type de VPN** : Sélectionnez **Route*based**.
  * **Référence (SKU)** : VpnGw1
  * **Réseau virtuel** : VNet1
  * **Plage d’adresses de sous-réseau de la passerelle** : 10.1.255.0/27
  * **Adresse IP publique** : Création
  * **Nom de l’adresse IP publique** : VNet1GWpip

* **Connection**
  * **Name** : VNet1toVNet4
  * **Clé partagée** : Vous pouvez créer la clé partagée vous-même. Lorsque vous créez la connexion entre les réseaux virtuels, les valeurs doivent correspondre. Pour cet exercice, utilisez abc123.

**Valeurs pour VNet4 :**

* **Paramètres de réseau virtuel**
  * **Name** : VNet4
  * **Espace d’adressage** : 10.41.0.0/16
  * **Abonnement**: Sélectionnez l’abonnement que vous souhaitez utiliser.
  * **Groupe de ressources** : TestRG4
  * **Emplacement** : USA Ouest
  * **Sous-réseau**
  * **Name** : FrontEnd
  * **Plage d’adresses** : 10.41.0.0/24

* **Paramètres de passerelle de réseau virtuel**
  * **Name** : VNet4GW
  * **Groupe de ressources** : USA Ouest
  * **Génération** : Génération 1
  * **Type de passerelle** : Sélectionnez **VPN**.
  * **Type de VPN** : Sélectionnez **Route-based**.
  * **Référence (SKU)** : VpnGw1
  * **Réseau virtuel** : VNet4
  * **Plage d’adresses de sous-réseau de la passerelle** : 10.41.255.0/27
  * **Adresse IP publique** : Création
  * **Nom de l’adresse IP publique** : VNet4GWpip

* **Connection**
  * **Name** : VNet4toVNet1
  * **Clé partagée** : Vous pouvez créer la clé partagée vous-même. Lorsque vous créez la connexion entre les réseaux virtuels, les valeurs doivent correspondre. Pour cet exercice, utilisez abc123.

## <a name="create-and-configure-vnet1"></a>Créer et configurer VNet1

Si vous disposez déjà d’un réseau virtuel, vérifiez que les paramètres sont compatibles avec la conception de votre passerelle VPN, avec une attention particulière pour tous les sous-réseaux qui pourraient chevaucher d’autres réseaux. Si vos sous-réseaux se chevauchent, votre connexion ne fonctionnera pas correctement.

### <a name="to-create-a-virtual-network"></a>Pour créer un réseau virtuel

[!INCLUDE [About cross-premises addresses](../../includes/vpn-gateway-cross-premises.md)]

[!INCLUDE [Create a virtual network](../../includes/vpn-gateway-basic-vnet-rm-portal-include.md)]

## <a name="create-the-vnet1-gateway"></a>Créer la passerelle VNet1

Dans cette étape, vous créez la passerelle de réseau virtuel de votre réseau virtuel. La création d’une passerelle nécessite généralement au moins 45 minutes, selon la référence SKU de passerelle sélectionnée. Si vous créez cette configuration dans le cadre d’un exercice, consultez les [Exemples de paramètres](#example-settings).

[!INCLUDE [About gateway subnets](../../includes/vpn-gateway-about-gwsubnet-portal-include.md)]

### <a name="to-create-a-virtual-network-gateway"></a>Pour créer une passerelle de réseau virtuel

[!INCLUDE [Create a gateway](../../includes/vpn-gateway-add-gw-rm-portal-include.md)]

[!INCLUDE [NSG warning](../../includes/vpn-gateway-no-nsg-include.md)]

## <a name="create-and-configure-vnet4"></a>Créer et configurer VNet4

Après avoir configuré VNet1, créez VNet4 et la passerelle VNet4 en répétant les étapes précédentes et en remplaçant les valeurs par les valeurs de VNet4. Vous n’avez pas besoin d’attendre que la création de la passerelle de réseau virtuel pour VNet1 soit terminée pour configurer VNet4. Si vous utilisez vos propres valeurs, assurez-vous que les espaces d’adressage ne chevauchent pas les réseaux virtuels auxquels vous souhaitez vous connecter.

## <a name="configure-the-vnet1-gateway-connection"></a>Configurer la connexion de passerelle VNet1

Lorsque les passerelles de réseau virtuel pour VNet1 et VNet4 sont terminées, vous pouvez créer vos connexions de passerelle de réseau virtuel. Dans cette section, vous allez créer une connexion de VNet1 à VNet4. Ces étapes s’appliquent uniquement aux réseaux virtuels situés dans le même abonnement. Si vos réseaux virtuels figurent dans des abonnements différents, vous devrez utiliser [PowerShell](vpn-gateway-vnet-vnet-rm-ps.md) pour établir la connexion. Toutefois, si vos réseaux virtuels se trouvent dans des groupes de ressources différents au sein du même abonnement, vous pouvez les connecter à l’aide du portail.

1. Dans le portail Azure, sélectionnez **Toutes les ressources**, entrez *passerelle de réseau virtuel* dans la recherche de zone, puis accédez à la passerelle de votre réseau virtuel. Par exemple, **VNet1GW**. Sélectionnez la passerelle pour ouvrir la page **Passerelle de réseau virtuel**.
1. Sur la page Passerelle, accédez à **Paramètres -> Connexions**. Sélectionnez ensuite **Ajouter**.

   :::image type="content" source="./media/vpn-gateway-howto-vnet-vnet-resource-manager-portal/connections.png" alt-text="Page Connexions":::
1. La page **Ajouter une connexion** s’ouvre.

   :::image type="content" source="./media/vpn-gateway-howto-vnet-vnet-resource-manager-portal/vnet1-vnet4.png" alt-text="Ajouter une connexion":::

   Sur la page **Ajouter une connexion**, renseignez les valeurs correspondant à votre connexion :

   * **Name** : Entrez un nom pour votre connexion. Par exemple, *VNet1toVNet4*.

   * **Type de connexion** : Sélectionnez **Réseau virtuel à réseau virtuel** dans la liste déroulante.

   * **Première passerelle de réseau virtuel** : La valeur de ce champ est automatiquement renseignée parce que vous créez cette connexion à partir de la passerelle de réseau virtuel spécifiée.

   * **Deuxième passerelle de réseau virtuel** : Ce champ correspond à la passerelle de réseau virtuel sur laquelle vous souhaitez créer une connexion. Sélectionnez **Choisir une autre passerelle de réseau virtuel** pour ouvrir la page **Choisir la passerelle de réseau virtuel**.

      :::image type="content" source="./media/vpn-gateway-howto-vnet-vnet-resource-manager-portal/choose.png" alt-text="Choisir une passerelle":::

     * Cette page répertorie les différentes passerelles de réseau virtuel disponibles. Notez que seules les passerelles de réseau virtuel incluses dans votre abonnement sont répertoriées. Si vous souhaitez vous connecter à une passerelle de réseau virtuel qui ne se trouve pas dans votre abonnement, utilisez [PowerShell](vpn-gateway-vnet-vnet-rm-ps.md).

     * Sélectionnez la passerelle de réseau virtuel à laquelle vous souhaitez vous connecter.

   * **Clé partagée** : Dans ce champ, entrez une clé partagée pour votre connexion. Vous pouvez générer ou créer cette clé vous-même. Dans une connexion de site à site, la clé que vous utilisez est exactement la même pour votre appareil local et votre connexion de passerelle de réseau virtuel. Le concept est similaire, sauf qu’au lieu de se connecter à un périphérique VPN, vous vous connectez à une autre passerelle de réseau virtuel.
1. Sélectionnez **OK** pour enregistrer vos modifications.

## <a name="configure-the-vnet4-gateway-connection"></a>Configurer la connexion de passerelle VNet4

Créez ensuite une connexion de VNet4 à VNet1. Dans le portail, recherchez la passerelle de réseau virtuel associée à VNet4. Suivez les étapes de la section précédente, en remplaçant les valeurs pour créer une connexion de VNet4 à VNet1. Vérifiez que vous utilisez la même clé partagée.

## <a name="verify-your-connections"></a>Vérifiez vos connexions

1. Recherchez la passerelle de réseau virtuel dans le portail Azure. 
1. Sur la page **Passerelle de réseau virtuel**, sélectionnez **Connexion** pour afficher la page **Connexion** de la passerelle de réseau virtuel. Une fois la connexion établie, la valeur d’**État** devient **Connecté**.

   :::image type="content" source="./media/vpn-gateway-howto-vnet-vnet-resource-manager-portal/view-connections.png" alt-text="Vérifier les connexions":::
1. Dans la colonne **Nom**, sélectionnez l’une des connexions pour afficher plus d’informations. Lorsque les données commencent à circuler, des valeurs apparaissent pour **Données entrantes** et **Données sortantes**.

   :::image type="content" source="./media/vpn-gateway-howto-vnet-vnet-resource-manager-portal/status.png" alt-text="Capture d’écran montrant un groupe de ressources contenant des valeurs pour les données entrantes et les données sortantes":::

## <a name="add-additional-connections"></a>Ajouter des connexions supplémentaires

Si vous souhaitez ajouter des connexions supplémentaires, accédez à la passerelle de réseau virtuel à partir de laquelle créer la connexion, puis sélectionnez **Connexions**. Vous pouvez créer une autre connexion de réseau virtuel à réseau virtuel, ou créer une connexion de site à site IPsec vers un emplacement local. Veillez à ajuster le **type de connexion** en fonction du type de connexion que vous souhaitez créer. Avant de créer des connexions supplémentaires, vérifiez que l’espace d’adressage de votre réseau virtuel ne chevauche aucun des espaces d’adressage auxquels vous souhaitez vous connecter. Pour connaître la procédure de création d’une connexion site à site, consultez [Créer une connexion de site à site](vpn-gateway-howto-site-to-site-resource-manager-portal.md).

## <a name="vnet-to-vnet-faq"></a>Forum Aux Questions sur l’interconnexion de réseaux virtuels

Consultez les détails du Forum Aux Questions pour plus d’informations sur les connexions de réseau virtuel à réseau virtuel.

[!INCLUDE [vpn-gateway-vnet-vnet-faq](../../includes/vpn-gateway-faq-vnet-vnet-include.md)]

## <a name="next-steps"></a>Étapes suivantes

* Pour savoir comment limiter le trafic réseau aux ressources d’un réseau virtuel, consultez [Sécurité du réseau](../virtual-network/network-security-groups-overview.md).

* Pour plus d’informations sur la façon dont Azure achemine le trafic entre les ressources Azure, locales et Internet, consultez [Routage du trafic de réseau virtuel](../virtual-network/virtual-networks-udr-overview.md).