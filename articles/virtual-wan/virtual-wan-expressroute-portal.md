---
title: 'Tutoriel : Créer des connexions ExpressRoute à l’aide d’Azure Virtual WAN'
description: Dans ce tutoriel, découvrez comment utiliser Azure Virtual WAN pour établir des connexions ExpressRoute à des environnements Azure et locaux.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: tutorial
ms.date: 10/07/2020
ms.author: cherylmc
Customer intent: As someone with a networking background, I want to connect my corporate on-premises network(s) to my VNets using Virtual WAN and ExpressRoute.
ms.openlocfilehash: 07053c096ce001b322e5f05556bd041519ca9d2e
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92102474"
---
# <a name="tutorial-create-an-expressroute-association-using-azure-virtual-wan"></a>Tutoriel : Créer une association ExpressRoute avec Azure Virtual WAN

Ce didacticiel vous montre comment utiliser Azure Virtual WAN pour vous connecter à vos ressources dans Azure via un circuit ExpressRoute. Pour plus d’informations sur Azure Virtual WAN et les ressources Azure Virtual WAN, consultez [Vue d’ensemble d’Azure Virtual WAN](virtual-wan-about.md).

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer un WAN virtuel
> * Créer un hub et une passerelle
> * Connecter un réseau virtuel à un hub
> * Connecter un circuit à une passerelle de hub
> * Tester la connectivité
> * Modifier la taille d’une passerelle
> * Publier un itinéraire par défaut

## <a name="prerequisites"></a>Prérequis

Vérifiez que vous disposez des éléments ci-dessous avant de commencer votre configuration :

* Vous avez un réseau virtuel auquel vous souhaitez vous connecter. Vérifiez qu’aucun des sous-réseaux de votre réseau local ne chevauche les réseaux virtuels auxquels vous souhaitez vous connecter. Pour créer un réseau virtuel dans le portail Azure, consultez le [guide de démarrage rapide](../virtual-network/quick-create-portal.md).

* Votre réseau virtuel n’a pas de passerelle de réseau virtuel. Si votre réseau virtuel dispose d’une passerelle (VPN ou ExpressRoute), vous devez supprimer toutes les passerelles. Cette configuration nécessite que les réseaux virtuels soient connectés à la passerelle hub Virtual WAN.

* Obtenez une plage d’adresses IP pour la région de votre hub. Le hub est un réseau virtuel qui est créé et utilisé par Virtual WAN. La plage d’adresses que vous spécifiez pour le hub ne peut pas chevaucher les réseaux virtuels existants auxquels vous vous connectez. Elle ne peut pas non plus chevaucher vos plages d’adresses auxquelles vous vous connectez en local. Si vous ne maîtrisez pas les plages d’adresses IP situées dans votre configuration de réseau local, contactez une personne en mesure de vous aider.

* Le circuit ExpressRoute doit être un circuit Premium ou Standard pour pouvoir se connecter à la passerelle du hub.

* Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="create-a-virtual-wan"></a><a name="openvwan"></a>Créer un WAN virtuel

Dans un navigateur, accédez au [portail Azure](https://portal.azure.com) et connectez-vous avec votre compte Azure.

1. Accédez à la page WAN Virtuel. Dans le portail, cliquez sur **+Créer une ressource** . Tapez **WAN virtuel** dans la zone de recherche et sélectionnez Entrée.
2. Sélectionnez **WAN virtuel** dans les résultats. Dans la page Virtual WAN, cliquez sur **Créer** pour ouvrir la page Créer un WAN.
3. Dans la page **Créer un WAN** , sous l’onglet **Fonctions de base** , renseignez les champs suivants :

   ![Créer un WAN](./media/virtual-wan-expressroute-portal/createwan.png)

   * **Abonnement**  : sélectionnez l’abonnement que vous voulez utiliser.
   * **Groupe de ressources** : Créer en un nouveau ou utiliser un qui existe déjà.
   * **Emplacement du groupe de ressources**  : choisissez un emplacement de la ressource dans la liste déroulante. Un WAN est une ressource globale et ne réside pas dans une région particulière. Toutefois, vous devez sélectionner une région pour gérer et localiser plus facilement la ressource WAN que vous créez.
   * **Nom**  : tapez le nom que vous souhaitez donner à votre réseau étendu.
   * **Type**  : sélectionnez **Standard** . Vous ne pouvez pas créer une passerelle ExpressRoute à l’aide de la référence SKU de base.
4. Quand vous avez fini de renseigner les champs, cliquez sur **Vérifier + Créer** .
5. Après la validation, sélectionnez **Créer** pour créer le WAN virtuel.

## <a name="create-a-virtual-hub-and-gateway"></a><a name="hub"></a>Créer un hub virtuel et une passerelle

Un hub virtuel est un réseau virtuel qui est créé et utilisé par Virtual WAN. Il peut contenir différentes passerelles, telles que VPN et ExpressRoute. Dans cette section, vous allez créer une passerelle ExpressRoute pour votre hub virtuel. Vous pouvez créer la passerelle lorsque vous [créez un hub virtuel](#newhub), ou vous pouvez créer la passerelle dans un [hub existant](#existinghub) en la modifiant. 

Les passerelles ExpressRoute sont approvisionnées en unités de 2 Gbits/s. 1 unité d’échelle = 2 Gbits/s avec prise en charge jusqu’à 10 unités d’échelle = 20 Gbits/s. La création complète d’un hub virtuel et d’une passerelle prend environ 30 minutes.

### <a name="to-create-a-new-virtual-hub-and-a-gateway"></a><a name="newhub"></a>Pour créer un hub virtuel et une passerelle

Créez un hub virtuel. Une fois le hub créé, vous serez facturé, même si vous n’y joignez aucun site.

[!INCLUDE [Create a hub](../../includes/virtual-wan-tutorial-er-hub-include.md)]

### <a name="to-create-a-gateway-in-an-existing-hub"></a><a name="existinghub"></a>Pour créer une passerelle dans un hub existant

Vous pouvez également créer une passerelle dans un hub existant en la modifiant.

1. Accédez au hub virtuel que vous souhaitez modifier et sélectionnez-le.
2. Sur la page **Modifier un hub virtuel** , cochez la case **Inclure la passerelle ExpressRoute** .
3. Cliquez sur **Confirmer** pour confirmer vos modifications. La création complète d’un hub virtuel et des ressources d’un hub prend environ 30 minutes.

   ![hub existant](./media/virtual-wan-expressroute-portal/edithub.png "modifier un hub")

### <a name="to-view-a-gateway"></a>Pour afficher une passerelle

Une fois que vous avez créé une passerelle ExpressRoute, vous pouvez afficher les détails de la passerelle. Accédez au Hub, sélectionnez **ExpressRoute** et affichez la passerelle.

![Afficher la passerelle](./media/virtual-wan-expressroute-portal/viewgw.png "afficher la passerelle")

## <a name="connect-your-vnet-to-the-hub"></a><a name="connectvnet"></a>Connecter votre réseau virtuel au hub

Dans cette section, vous créez la connexion de peering entre votre hub et un réseau virtuel. Répétez ces étapes pour chaque réseau virtuel que vous souhaitez connecter.

1. Dans la page de votre réseau étendu, cliquez sur **Connexion réseau de réseau virtuel** .
2. Dans la page de connexion de réseau virtuel, cliquez sur **+ Ajouter une connexion** .
3. Dans la page **Ajouter une connexion** , renseignez les champs suivants :

    * **Nom de connexion** : nommez votre connexion.
    * **Hubs** : sélectionnez le hub que vous souhaitez associer à cette connexion.
    * **Abonnement** : vérifiez l’abonnement.
    * **Réseau virtuel** : sélectionnez le réseau virtuel que vous souhaitez connecter à ce hub. Le réseau virtuel ne peut pas avoir une passerelle de réseau virtuel déjà existante (ni VPN, ni ExpressRoute).

## <a name="connect-your-circuit-to-the-hub-gateway"></a><a name="connectcircuit"></a>Connecter votre circuit à la passerelle de hub

Une fois la passerelle créée, vous pouvez y connecter un circuit [ExpressRoute](../expressroute/expressroute-howto-circuit-portal-resource-manager.md). Les circuits ExpressRoute Standard ou Premium qui se trouvent dans des emplacements pris en charge par ExpressRoute Global Reach peuvent se connecter à une passerelle ExpressRoute WAN virtuel et bénéficier de toutes les fonctionnalités de transit Virtual WAN (transit VPN à VPN, VPN et ExpressRoute). Les circuits ExpressRoute Standard et Premium qui se trouvent dans des emplacements non Global Reach peuvent se connecter aux ressources Azure, mais ils ne peuvent pas utiliser les fonctionnalités de transit Virtual WAN. Les circuits ExpressRoute Local peuvent être utilisés avec les hubs Azure Virtual WAN, du moment que les réseaux virtuels Spoke qui sont connectés à un hub Virtual WAN se trouvent dans la même région que celui-ci.

### <a name="to-connect-the-circuit-to-the-hub-gateway"></a>Pour connecter votre circuit à la passerelle de hub

Dans le portail, accédez à la page **Hub virtuel -> Connectivité -> ExpressRoute** . Si vous avez accès à un circuit ExpressRoute dans votre abonnement, vous verrez le circuit que vous souhaitez utiliser dans la liste des circuits. Si vous ne voyez aucun circuit, mais que vous l’avez fourni avec une clé d’autorisation et un URI de circuit homologue, vous pouvez échanger et connecter un circuit. Consultez [Rechercher un circuit en utilisant une clé d'autorisation](#authkey).

1. Sélectionnez le circuit.
2. Sélectionnez **Connecter un ou plusieurs circuits** .

   ![connecter des circuits](./media/virtual-wan-expressroute-portal/cktconnect.png "connecter des circuits")

### <a name="to-connect-by-redeeming-an-authorization-key"></a><a name="authkey"></a>Se connecter en utilisant une clé d'autorisation

Utilisez la clé d’autorisation et l’URI de circuit fournis pour vous connecter.

1. Sur la page ExpressRoute, cliquez sur **+Échanger la clé d’autorisation**

   ![Capture d’écran montrant ExpressRoute pour un hub virtuel avec l’option Utiliser la clé d’autorisation sélectionnée.](./media/virtual-wan-expressroute-portal/redeem.png "échanger")
2. Dans la page Échanger la clé d’autorisation, renseignez les valeurs.

   ![échanger des valeurs de clés](./media/virtual-wan-expressroute-portal/redeemkey2.png "échanger des valeurs de clés")
3. Sélectionnez **Ajouter** pour ajouter la clé.
4. Affichez le circuit. Un circuit échangé affiche uniquement le nom (sans le type, le fournisseur et d’autres informations), car il se trouve dans un abonnement différent de celui de l’utilisateur.

## <a name="to-test-connectivity"></a>Pour tester la connectivité

Une fois la connexion au circuit établie, l’état de la connexion du hub indique « Ce hub », signifiant que la connexion est établie à la passerelle ExpressRoute Hub. Attendez environ 5 minutes avant de tester la connectivité à partir d’un client situé derrière votre circuit ExpressRoute, par exemple, une machine virtuelle dans le réseau virtuel que vous avez créée précédemment.

Si vous avez des sites connectés à une passerelle VPN de Virtual WAN dans le même hub que la passerelle ExpressRoute, vous pouvez disposer d’une connectivité bidirectionnelle entre les points de terminaison VPN et ExpressRoute. Le routage dynamique (BGP) est pris en charge. L’ASN des passerelles dans le hub est fixe et ne peut pas être modifié pour l’instant.

## <a name="to-change-the-size-of-a-gateway"></a>Pour modifier la taille d’une passerelle

Si vous souhaitez modifier la taille de votre passerelle ExpressRoute, localisez la passerelle ExpressRoute dans le hub, puis sélectionnez les unités d’échelle dans la liste déroulante. Enregistrez vos modifications. La mise à jour de la passerelle du hub prendra environ 30 minutes.

![modifier la taille d’une passerelle](./media/virtual-wan-expressroute-portal/changescale.png "modifier la taille d’une passerelle")

## <a name="to-advertise-default-route-00000-to-endpoints"></a>Pour publier l’itinéraire par défaut 0.0.0.0/0 vers les points de terminaison

Si vous souhaitez que le hub virtuel Azure publie l’itinéraire par défaut 0.0.0.0/0 sur vos points de terminaison ExpressRoute, vous devez activer « Propager l’itinéraire par défaut ».

1. Sélectionnez votre **Circuit ->...-> Modifie la connexion** .

   ![Modifier la connexion](./media/virtual-wan-expressroute-portal/defaultroute1.png "Modifier la connexion")

2. Sélectionnez **Activer** pour propager l’itinéraire par défaut.

   ![Propager l’itinéraire par défaut](./media/virtual-wan-expressroute-portal/defaultroute2.png "Propager l’itinéraire par défaut")

## <a name="clean-up-resources"></a><a name="cleanup"></a>Supprimer des ressources

Quand vous n’avez plus besoin de ces ressources, vous pouvez utiliser [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) pour supprimer le groupe de ressources et toutes les ressources qu’il contient. Remplacez « myResourceGroup » par le nom de votre groupe de ressources et exécutez la commande PowerShell suivante :

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>Étapes suivantes

Ensuite, pour en savoir plus sur Virtual WAN, consultez :

> [!div class="nextstepaction"]
> * [FAQ sur Virtual WAN](virtual-wan-faq.md)
