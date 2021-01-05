---
title: 'Configurer le protocole BGP pour une passerelle VPN : Portail'
titleSuffix: Azure VPN Gateway
description: Cet article vous guide dans la configuration du protocole BGP avec une passerelle VPN Azure à l’aide de PowerShell.
services: vpn-gateway
author: yushwang
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/18/2020
ms.author: yushwang
ms.openlocfilehash: f52d684d1e6ef63fdf4287c610608061f30395f8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90992678"
---
# <a name="how-to-configure-bgp-on-azure-vpn-gateways"></a>Guide pratique pour configurer le protocole BGP sur des passerelles VPN Azure

Cet article vous guide pas à pas dans l’activation du protocole BGP sur une connexion VPN site à site (S2S) et une connexion de réseau virtuel à réseau virtuel à l’aide du portail Azure.

## <a name="about-bgp"></a><a name="about"></a>À propos du protocole BGP

BGP est le protocole de routage standard couramment utilisé sur Internet pour échanger des informations de routage et d’accessibilité entre plusieurs réseaux. Il permet aux passerelles VPN Azure et à vos périphériques VPN locaux (appelés voisins ou homologues BGP) d’échanger des « itinéraires » informant les deux passerelles sur la disponibilité et l’accessibilité de ces préfixes à travers les passerelles ou routeurs impliqués. Le protocole BGP assure également le routage de transit entre plusieurs réseaux en propageant les itinéraires qu’une passerelle BGP obtient d’un homologue BGP à tous les autres homologues BGP.

Pour plus d’informations sur les avantages du protocole BGP et pour comprendre les exigences techniques et considérations d’utilisation du protocole BGP, consultez [Vue d’ensemble du protocole BGP avec les passerelles VPN Azure](vpn-gateway-bgp-overview.md).

## <a name="getting-started"></a>Prise en main

Chaque partie de cet article vous aide à constituer un bloc de construction pour activer le protocole BGP dans votre connectivité réseau. Si vous terminez ces trois parties, vous générez la topologie illustrée dans le Schéma 1.

**Schéma 1**

:::image type="content" source="./media/bgp-howto/bgp-crosspremises-v2v.png" alt-text="Schéma montrant l’architecture et les paramètres réseau" border="false":::

Vous pouvez les combiner pour créer un réseau de transit plus complexe, à plusieurs tronçons, qui répond à vos besoins.

### <a name="prerequisites"></a>Prérequis

Assurez-vous de disposer d’un abonnement Azure. Si vous ne disposez pas déjà d’un abonnement Azure, vous pouvez activer vos [avantages abonnés MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) ou créer un [compte gratuit](https://azure.microsoft.com/pricing/free-trial/).

## <a name="part-1-configure-bgp-on-the-virtual-network-gateway"></a><a name ="config"></a>Partie 1 : Configurer le protocole BGP sur la passerelle de réseau virtuel

Dans cette section, vous allez créer et configurer un réseau virtuel, créer et configurer une passerelle de réseau virtuel avec les paramètres du protocole BGP, et obtenir les adresses IP d’homologue BGP Azure. Le Schéma 2 présente les paramètres de configuration à utiliser dans le cadre des étapes décrites dans cette section.

**Schéma 2 :**

:::image type="content" source="./media/bgp-howto/bgp-gateway.png" alt-text="Schéma montrant l’architecture et les paramètres réseau" border="false":::

### <a name="1-create-and-configure-testvnet1"></a>1. Créer et configurer TestVNet1

Dans le cadre de cette étape, vous allez créer et configurer un réseau TestVNet1. Suivez les étapes décrites dans le [didacticiel Créer une passerelle](vpn-gateway-tutorial-create-gateway-powershell.md) pour créer et configurer votre réseau virtuel Azure et votre passerelle VPN. Utilisez les paramètres de référence dans les captures d’écran ci-dessous.

* Réseau virtuel :

   :::image type="content" source="./media/bgp-howto/testvnet-1.png" alt-text="Schéma montrant l’architecture et les paramètres réseau":::

* Sous-réseaux :

   :::image type="content" source="./media/bgp-howto/testvnet-1-subnets.png" alt-text="Schéma montrant l’architecture et les paramètres réseau":::

### <a name="2-create-the-vpn-gateway-for-testvnet1-with-bgp-parameters"></a>2. créer la passerelle VPN de TestVNet1 avec les paramètres BGP

Dans le cadre de cette étape, vous allez créer une passerelle VPN avec les paramètres de protocole BGP correspondants.

1. Dans le portail Azure, accédez à la ressource **Passerelle de réseau virtuel** à partir de la Place de marché, puis sélectionnez **Créer**.

1. Définissez les paramètres comme ci-dessous :

   :::image type="content" source="./media/bgp-howto/create-gateway-1.png" alt-text="Schéma montrant l’architecture et les paramètres réseau":::

1. Dans la section en surbrillance **Configurer BGP** de la page, configurez les paramètres suivants :

   :::image type="content" source="./media/bgp-howto/create-gateway-1-bgp.png" alt-text="Schéma montrant l’architecture et les paramètres réseau":::

   * Sélectionnez **Configurer BGP** - **Activé** pour afficher la section de configuration du protocole BGP.

   * Entrez votre NSA (numéro de système autonome).

   * Le champ **Adresse IP BGP Azure APIPA** est facultatif. Si vos appareils VPN locaux utilisent l’adresse APIPA pour BGP, vous devez sélectionner une adresse dans la plage d’adresses APIPA pour VPN réservées Azure, qui est comprise entre **169.254.21.0** et **169.254.22.255**. Cet exemple utilise l’adresse 169.254.21.11.

   * Si vous créez une passerelle VPN en mode actif/actif, la section BGP affiche une **Deuxième adresse IP BGP Azure personnalisée** supplémentaire. Spécifiez une autre adresse de la plage APIPA autorisée (**169.254.21.0** à **169.254.22.255**).

   > [!IMPORTANT]
   >
   > * Par défaut, Azure affecte automatiquement une adresse IP privée de la plage de préfixes GatewaySubnet en tant qu’adresse IP BGP Azure sur la passerelle VPN Azure. L’adresse BGP Azure APIPA personnalisée est nécessaire lorsque vos appareils VPN locaux utilisent une adresse APIPA (169.254.0.1 à 169.254.255.254) comme adresse IP BGP. La passerelle VPN Azure choisit l’adresse APIPA personnalisée si la ressource de passerelle de réseau local correspondante (réseau local) a une adresse APIPA en tant qu’adresse IP d’homologue BGP. Si la passerelle de réseau local utilise une adresse IP normale (non APIPA), la passerelle VPN Azure revient à l’adresse IP privée de la plage GatewaySubnet.
   >
   > * Les adresses BGP APIPA ne doivent pas se chevaucher entre les appareils VPN locaux et toutes les passerelles VPN Azure connectées.
   >

1. Sélectionnez **Vérifier + créer** pour exécuter la validation. Une fois la validation réussie, sélectionnez **Créer** pour déployer la passerelle VPN. La création et le déploiement complets d’une passerelle peuvent prendre jusqu’à 45 minutes. Vous pouvez voir l’état du déploiement dans la page Vue d’ensemble pour votre passerelle.

### <a name="3-obtain-the-azure-bgp-peer-ip-addresses"></a>3. Obtenir les adresses IP d’homologue BGP Azure

Une fois la passerelle créée, vous pouvez obtenir l’adresse IP d’homologue BGP sur la passerelle VPN Azure. Ces adresses sont nécessaires pour configurer vos appareils VPN locaux afin d’établir des sessions BGP avec la passerelle VPN Azure.

1. Accédez à la ressource de passerelle de réseau virtuel et sélectionnez la page **Configuration** pour afficher les informations de configuration du protocole BGP, comme dans la capture d’écran suivante. Sur cette page, vous pouvez afficher toutes les informations de configuration du protocole BGP sur votre passerelle VPN Azure : NSA, adresse IP publique et adresses IP d’homologue BGP correspondantes côté Azure (par défaut et APIPA).

   :::image type="content" source="./media/bgp-howto/vnet-1-gw-bgp.png" alt-text="Schéma montrant l’architecture et les paramètres réseau":::

1. Dans la page **Configuration**, vous pouvez apporter les modifications de configuration suivantes :

   * Si nécessaire, vous pouvez mettre à jour le NSA ou l’adresse IP BGP APIPA.
   * Si vous disposez d’une passerelle VPN en mode actif/actif, cette page affiche l’adresse IP publique, la valeur par défaut et les adresses IP BGP APIPA de la deuxième instance de passerelle VPN Azure.

1. Si vous avez apporté des modifications, sélectionnez **Enregistrer** pour valider les modifications apportées à votre passerelle VPN Azure.

## <a name="part-2-configure-bgp-on-cross-premises-s2s-connections"></a><a name ="crosspremises"></a>Partie 2 : Configurer le protocole BGP sur des connexions S2S intersites

Pour établir une connexion intersite, vous devez créer une *passerelle de réseau local* pour représenter votre appareil VPN local, et une *connexion* entre la passerelle VPN et la passerelle de réseau local, comme décrit dans [Créer une connexion site à site](vpn-gateway-howto-site-to-site-resource-manager-portal.md). Cet article contient les propriétés supplémentaires nécessaires pour spécifier les paramètres de configuration du protocole BGP.

**Schéma 3**

:::image type="content" source="./media/bgp-howto/bgp-crosspremises.png" alt-text="Schéma montrant l’architecture et les paramètres réseau" border="false":::

### <a name="1-configure-bgp-on-the-local-network-gateway"></a>1. Configurer le protocole BGP sur la passerelle de réseau local

Dans le cadre de cette étape, vous allez configurer le protocole BGP sur la passerelle de réseau local. Utilisez la capture d’écran suivante à titre d’exemple. La capture d’écran montre la passerelle de réseau local (Site5) avec les paramètres spécifiés dans le Schéma 3.

:::image type="content" source="./media/bgp-howto/create-local-bgp.png" alt-text="Schéma montrant l’architecture et les paramètres réseau":::

#### <a name="important-configuration-considerations"></a>Considérations importantes relatives à la configuration

* Le NSA et l’adresse IP d’homologue BGP doivent correspondre à la configuration de votre routeur VPN local.
* Vous pouvez laisser l’**Espace d’adressage** vide uniquement si vous utilisez le protocole BGP pour vous connecter à ce réseau. La passerelle VPN Azure ajoute en interne un itinéraire de votre adresse IP d’homologue BGP au tunnel IPsec correspondant. Si vous n’utilisez **PAS** le protocole BGP entre la passerelle VPN Azure et ce réseau particulier, vous **devez** fournir une liste de préfixes d’adresses valides pour l’**espace d’adressage**.
* Vous pouvez éventuellement utiliser une **adresse IP APIPA** (169.254.x.x) comme adresse IP d’homologue BGP local si nécessaire. Toutefois, vous devez également spécifier une adresse IP APIPA comme décrit précédemment dans cet article pour votre passerelle VPN Azure. Dans le cas contraire, la session BGP ne peut pas établir cette connexion.
* Vous pouvez entrer les informations de configuration du protocole BGP lors de la création de la passerelle de réseau local, ou ajouter ou modifier la configuration du protocole BGP à partir de la page **Configuration** de la ressource de passerelle de réseau local.

**Exemple**

Cet exemple utilise une adresse APIPA (169.254.100.1) en tant qu’adresse IP d’homologue BGP local :

:::image type="content" source="./media/bgp-howto/local-apipa.png" alt-text="Schéma montrant l’architecture et les paramètres réseau":::

### <a name="2-configure-a-s2s-connection-with-bgp-enabled"></a>2. Configurer une connexion S2S avec le protocole BGP activé

Dans le cadre de cette étape, vous allez créer une connexion avec le protocole BGP activé. Si vous disposez déjà d’une connexion et souhaitez activer le protocole BGP sur cette dernière, vous pouvez [mettre à jour une connexion existante](#update).

#### <a name="to-create-a-connection-with-bgp-enabled"></a>Pour créer une connexion avec le protocole BGP activé

Pour créer une connexion avec le protocole BGP activé, dans la page **Ajouter une connexion**, renseignez les valeurs, puis activez l’option **Activer BGP** pour activer le protocole BGP sur cette connexion. Sélectionnez **OK** pour établir la connexion.

:::image type="content" source="./media/bgp-howto/ipsec-connection-bgp.png" alt-text="Schéma montrant l’architecture et les paramètres réseau":::

#### <a name="to-update-an-existing-connection"></a><a name ="update"></a>Pour mettre à jour une connexion existante

Si vous souhaitez modifier l’option BGP sur une connexion, accédez à la page **Configuration** de la ressource de connexion, puis basculez l’option **BGP** en surbrillance dans l’exemple suivant. Sélectionnez **Save** (Enregistrer) pour enregistrer les modifications.

:::image type="content" source="./media/bgp-howto/update-bgp.png" alt-text="Schéma montrant l’architecture et les paramètres réseau":::

## <a name="part-3-configure-bgp-on-vnet-to-vnet-connections"></a><a name ="v2v"></a>Partie 3 : Configurer le protocole BGP sur des connexions de réseau virtuel à réseau virtuel

Les étapes d’activation ou de désactivation du protocole BGP sur une connexion de réseau virtuel à réseau virtuel sont les mêmes que les étapes S2S décrites dans la [Partie 2](#crosspremises). Vous pouvez activer le protocole BGP lors de la création de la connexion ou mettre à jour la configuration sur une connexion de réseau virtuel à réseau virtuel existante.

>[!NOTE] 
>Une connexion de réseau virtuel à réseau virtuel sans le protocole BGP limite la communication aux deux réseaux virtuels connectés. Activez le protocole BGP pour autoriser le routage de transit vers d’autres connexions S2S ou de réseau virtuel à réseau virtuel de ces deux réseaux virtuels.
>

Pour le contexte, consultez le **Schéma 4**. Si le protocole BGP devait être désactivé entre les réseaux TestVNet2 et TestVNet1, le réseau TestVNet2 n’apprendrait pas les itinéraires pour le réseau local, Site5, et ne pourrait donc pas communiquer avec le site 5. Une fois que vous activez le protocole BGP, comme indiqué dans le Schéma 4, les trois réseaux sont en mesure de communiquer via les connexions IPsec et de réseau virtuel à réseau virtuel.

**Schéma 4**

:::image type="content" source="./media/bgp-howto/bgp-crosspremises-v2v.png" alt-text="Schéma montrant l’architecture et les paramètres réseau" border="false":::

## <a name="next-steps"></a>Étapes suivantes

Une fois la connexion achevée, vous pouvez ajouter des machines virtuelles à vos réseaux virtuels. Consultez [Création d’une machine virtuelle](../virtual-machines/windows/quick-create-portal.md) pour connaître les différentes étapes.
