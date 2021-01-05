---
title: 'Azure Virtual WAN : Créer une appliance virtuelle réseau (NVA) dans le hub'
description: Apprenez à déployer une appliance virtuelle réseau dans le hub Virtual WAN.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: how-to
ms.date: 09/22/2020
ms.author: cherylmc
Customer intent: As someone with a networking background, I want to create a Network Virtual Appliance (NVA) in my Virtual WAN hub.
ms.openlocfilehash: 0e174f9b78606e0bea5cded3a176f4207745bb5c
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92077301"
---
# <a name="how-to-create-a-network-virtual-appliance-in-an-azure-virtual-wan-hub-preview"></a>Comment créer une appliance virtuelle réseau dans un hub Azure Virtual WAN (préversion)

Cet article vous montre comment utiliser Azure Virtual WAN pour vous connecter à vos ressources dans Azure via une **appliance virtuelle réseau** (NVA) dans Azure. Ce type de connexion requiert un périphérique VPN local disposant d’une adresse IP publique exposée en externe. Pour plus d’informations sur Virtual WAN, consultez la [présentation de Virtual WAN](virtual-wan-about.md).

Les étapes décrites dans cet article vous aident à créer une appliance virtuelle réseau **Barracuda CloudGen WAN** dans le hub Virtual WAN. Pour effectuer cet exercice, vous devez disposer d’un appareil Barracuda Cloud Premise Device (CPE) et d’une licence pour l’appliance Barracuda CloudGen WAN que vous déployez dans le hub avant de commencer.

Pour obtenir de la documentation sur le déploiement de **Cisco SD-WAN** au sein d’Azure Virtual WAN, envoyez un e-mail à Cisco à l’adresse e-mail suivante : vwan_public_preview@external.cisco.com


## <a name="prerequisites"></a>Prérequis

Vérifiez que vous disposez des éléments ci-dessous avant de commencer votre configuration :

* Obtenez une licence pour votre passerelle Barracuda CloudGen WAN. Pour en savoir plus sur la procédure à suivre, consultez la [documentation de Barracuda CloudGen WAN](https://www.barracuda.com/products/cloudgenwan).

* Vous avez un réseau virtuel auquel vous souhaitez vous connecter. Vérifiez qu’aucun des sous-réseaux de votre réseau local ne chevauche les réseaux virtuels auxquels vous souhaitez vous connecter. Pour créer un réseau virtuel dans le portail Azure, consultez le [guide de démarrage rapide](../virtual-network/quick-create-portal.md).

* Votre réseau virtuel n’a pas de passerelle de réseau virtuel. Si votre réseau virtuel dispose d’une passerelle (VPN ou ExpressRoute), vous devez supprimer toutes les passerelles. Cette configuration nécessite que les réseaux virtuels soient connectés à la passerelle hub Virtual WAN.

* Obtenez une plage d’adresses IP pour la région de votre hub. Le hub est un réseau virtuel qui est créé et utilisé par Virtual WAN. La plage d’adresses que vous spécifiez pour le hub ne peut pas chevaucher les réseaux virtuels existants auxquels vous vous connectez. Elle ne peut pas non plus chevaucher vos plages d’adresses auxquelles vous vous connectez en local. Si vous ne maîtrisez pas les plages d’adresses IP situées dans votre configuration de réseau local, contactez une personne en mesure de vous aider.

* Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="create-a-virtual-wan"></a><a name="openvwan"></a>Créer un WAN virtuel

[!INCLUDE [Create virtual WAN](../../includes/virtual-wan-create-vwan-include.md)]

## <a name="create-a-hub"></a><a name="hub"></a>Créer un hub

Un hub est un réseau virtuel qui peut contenir des passerelles pour offrir des fonctionnalités de site à site, ExpressRoute, de point à site ou d’appliance virtuelle réseau. Une fois le hub créé, vous serez facturé, même si vous n’y joignez aucun site. Si vous choisissez de créer une passerelle VPN site à site, il faut 30 minutes pour créer celle-ci dans le hub virtuel. Contrairement aux configurations site à site, ExpressRoute ou point à site, le hub doit d’abord être créé avant de pouvoir déployer une appliance virtuelle réseau dans le réseau virtuel du hub.

1. Localisez l’instance Virtual WAN que vous avez créée. Dans la page **Virtual WAN**, sous la section **Connectivité**, sélectionnez **Hubs**.
1. Dans la page **Hubs**, cliquez sur +Nouveau hub pour ouvrir la page **Créer un hub virtuel**.

   :::image type="content" source="./media/how-to-nva-hub/vwan-hub.png" alt-text="Concepts de base":::
1. Dans la page **Créer un hub virtuel**, sous l’onglet **Fonctions de base**, renseignez les champs suivants :

   **Détails du projet**

   * Région (précédemment appelée Emplacement)
   * Nom
   * Espace d’adressage privé du hub. L’espace d’adressage minimal pour créer un hub est /24. Cela implique que toute plage comprise entre /25 et /32 produit une erreur lors de la création. Le service Azure Virtual WAN, qui est géré par Microsoft, crée les sous-réseaux appropriés dans le hub virtuel pour les différents services/passerelles. (Par exemple : les appliances virtuelles réseau, les passerelles VPN, les passerelles ExpressRoute, les passerelles VPN utilisateur/point à site, le pare-feu, le routage, etc.). L’utilisateur n’a pas besoin de planifier explicitement l’espace d’adressage de sous-réseau pour les services du hub virtuel, car Microsoft s’en charge dans le cadre du service.
1. Sélectionnez **Vérifier + créer** pour valider.
1. Sélectionnez **Créer** pour créer le hub.

## <a name="create-the-network-virtual-appliance-in-the-hub"></a>Créer l’appliance virtuelle réseau dans le hub

Dans cette étape, vous allez créer une appliance virtuelle réseau dans le hub. La procédure pour chaque NVA sera différente pour le produit de chaque partenaire NVA. Pour cet exemple, nous créons une passerelle Barracuda CloudGen WAN.

1. Recherchez le hub Virtual WAN que vous avez créé à l’étape précédente et ouvrez-le.

   :::image type="content" source="./media/how-to-nva-hub/nva-hub.png" alt-text="Concepts de base":::
1. Recherchez la vignette Appliances virtuelles réseau et sélectionnez le lien **Créer**.
1. Dans le panneau **Appliance virtuelle réseau**, sélectionnez **Barracuda CloudGen WAN**, puis cliquez sur le bouton **Créer**.

   :::image type="content" source="./media/how-to-nva-hub/select-nva.png" alt-text="Concepts de base":::
1. Cela vous mène à l’offre de Place de marché Azure pour la passerelle Barracuda CloudGen WAN. Lisez les conditions, puis sélectionnez le bouton **Créer** lorsque vous êtes prêt.

   :::image type="content" source="./media/how-to-nva-hub/barracuda-create-basics.png" alt-text="Concepts de base":::
1. Sur la page **Notions de base**, vous devez fournir les informations suivantes :

   * **Abonnement** : choisissez l’abonnement que vous avez utilisé pour déployer le réseau étendu virtuel et le hub.
   * **Groupe de ressources** : choisissez le groupe de ressources que vous avez utilisé pour déployer le réseau étendu virtuel et le hub.
   * **Région** : choisissez la même région que celle où se trouve la ressource de votre hub virtuel.
   * **Nom de l’application** : Barracuda NextGen WAN est une application managée. Choisissez un nom qui facilite l’identification de cette ressource, car c’est le nom qu’elle portera lorsqu’elle apparaîtra dans votre abonnement.
   * **Groupe de ressources managées** : il s’agit du nom du groupe de ressources managées dans lequel Barracuda déploie les ressources qu’il gère. Le nom doit être pré-rempli.
1. Sélectionnez le bouton **Suivant : Passerelle CloudGen WAN**.

   :::image type="content" source="./media/how-to-nva-hub/barracuda-cloudgen-wan.png" alt-text="Concepts de base":::
1. Fournissez les informations suivantes :

   * **Hub Virtual WAN** : hub Virtual WAN dans lequel vous souhaitez déployer cette NVA.
   * **Unités d’infrastructure NVA** : indique le nombre d’unités d’infrastructure NVA avec lesquelles vous souhaitez déployer cette NVA. Choisissez la capacité de bande passante globale que vous souhaitez fournir à tous les sites de branche qui se connecteront à ce hub via cette NVA.
   * **Jeton** : Barracuda exige que vous fournissiez un jeton d’authentification ici pour vous identifier en tant qu’utilisateur inscrit de ce produit. Vous devez l’obtenir auprès de Barracuda.
1. Sélectionnez le bouton **Vérifier et créer** pour continuer.
1. Sur cette page, vous serez invité à accepter les conditions du contrat d’accès du coadministrateur. C’est la norme avec les applications managées où l’Éditeur aura accès à certaines ressources dans ce déploiement. Cochez la case **J’accepte les conditions générales ci-dessus**, puis sélectionnez **Créer**.

## <a name="connect-the-vnet-to-the-hub"></a><a name="vnet"></a>Connecter le réseau virtuel au hub

[!INCLUDE [Connect](../../includes/virtual-wan-connect-vnet-hub-include.md)]

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur Virtual WAN, consultez la page de [présentation de Virtual WAN](virtual-wan-about.md).
* Pour en savoir plus sur les NVA dans un hub Virtual WAN, consultez [À propos de l’appliance virtuelle réseau dans le hub Virtual WAN (préversion)](about-nva-hub.md).
