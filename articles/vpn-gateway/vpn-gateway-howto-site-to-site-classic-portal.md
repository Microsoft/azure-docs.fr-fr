---
title: 'Connectez votre réseau local à un réseau virtuel Azure : VPN site à site (classique) : Portail | Microsoft Docs'
description: Créez une connexion IPsec entre votre réseau local et un réseau virtuel Azure classique via l’Internet public.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 10/08/2020
ms.author: cherylmc
ms.openlocfilehash: 346536f5797841a850ef97dc4667110eafa721a4
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94656954"
---
# <a name="create-a-site-to-site-connection-using-the-azure-portal-classic"></a>Création d’une connexion de site à site à l’aide du portail Azure (Classic)

Cet article vous explique comment utiliser le portail Azure pour créer une connexion de passerelle VPN de site à site à partir de votre réseau local vers le réseau virtuel. Les étapes décrites dans cet article s’appliquent au modèle de déploiement classique mais pas au modèle de déploiement actuel, celui de Resource Manager. Vous pouvez également créer cette configuration à l’aide d’un autre outil ou modèle de déploiement en sélectionnant une option différente dans la liste suivante :

> [!div class="op_single_selector"]
> * [Azure portal](vpn-gateway-howto-site-to-site-resource-manager-portal.md)
> * [PowerShell](vpn-gateway-create-site-to-site-rm-powershell.md)
> * [INTERFACE DE LIGNE DE COMMANDE](vpn-gateway-howto-site-to-site-resource-manager-cli.md)
> * [Portail Azure (classique)](vpn-gateway-howto-site-to-site-classic-portal.md)
> 

Une connexion de passerelle VPN de site à site permet de connecter votre réseau local à un réseau virtuel Azure via un tunnel VPN IPsec/IKE (IKEv1 ou IKEv2). Ce type de connexion requiert un périphérique VPN local disposant d’une adresse IP publique exposée en externe. Pour plus d’informations sur les passerelles VPN, consultez l’article [À propos de la passerelle VPN](vpn-gateway-about-vpngateways.md).

![Schéma de connexion intersite d’une passerelle VPN site à site](./media/vpn-gateway-howto-site-to-site-classic-portal/site-to-site-diagram.png)

## <a name="before-you-begin"></a><a name="before"></a>Avant de commencer

Vérifiez que vous disposez des éléments ci-dessous avant de commencer votre configuration :

* Assurez-vous de vouloir utiliser le modèle de déploiement classique. Si vous souhaitez utiliser le modèle de déploiement Resource Manager, voir [Create a Site-to-Site connection (Resource Manager) (Créer une connexion de site à site [Resource Manager])](vpn-gateway-howto-site-to-site-resource-manager-portal.md). Nous vous recommandons d’utiliser le modèle de déploiement Resource Manager, car le modèle classique est hérité.
* Veillez à disposer d’un périphérique VPN compatible et à être entouré d’une personne en mesure de le configurer. Pour plus d’informations sur les périphériques VPN compatibles et la configuration de votre périphérique, consultez l’article [À propos des périphériques VPN](vpn-gateway-about-vpn-devices.md).
* Vérifiez que vous disposez d’une adresse IPv4 publique exposée en externe pour votre périphérique VPN.
* Si vous ne maîtrisez pas les plages d’adresses IP situées dans votre configuration de réseau local, vous devez contacter une personne en mesure de vous aider. Lorsque vous créez cette configuration, vous devez spécifier les préfixes des plages d’adresses IP qu’Azure acheminera vers votre emplacement local. Aucun des sous-réseaux de votre réseau local ne peut chevaucher les sous-réseaux du réseau virtuel auquel vous souhaitez vous connecter.
* PowerShell est requis pour spécifier la clé partagée et créer la connexion de passerelle VPN. [!INCLUDE [vpn-gateway-classic-powershell](../../includes/vpn-gateway-powershell-classic-locally.md)]

### <a name="sample-configuration-values-for-this-exercise"></a><a name="values"></a>Exemples de valeurs de configuration pour cet exercice

Nous utilisons les valeurs suivantes dans les exemples de cet article. Vous pouvez utiliser ces valeurs pour créer un environnement de test ou vous y référer pour mieux comprendre les exemples de cet article. En règle générale, lorsque vous utilisez des valeurs d’adresse IP pour l’espace d’adressage, vous devez vous coordonner avec votre administrateur réseau afin d’éviter le chevauchement des espaces d’adressage, qui peut perturber le routage. Dans ce cas, remplacez les valeurs d’adresse IP par les vôtres si vous souhaitez créer une connexion qui fonctionne.

* **Groupe de ressources :** TestRG1
* **Nom du réseau virtuel :** TestVNet1
* **Espace d’adressage :** 10.11.0.0/16
* **Nom du sous-réseau** : FrontEnd
* **Plage d'adresses du sous-réseau :** 10.11.0.0/24
* **GatewaySubnet :** 10.11.255.0/27
* **Région :** (États-Unis) USA Est
* **Nom du site local :** Site2
* **Espace d’adressage du client :** Espace d’adressage qui se trouve sur votre site local.

## <a name="create-a-virtual-network"></a><a name="CreatVNet"></a>Créer un réseau virtuel

Lorsque vous créez un réseau virtuel qui sera utilisé pour une connexion de site à site, vous devez vous assurer que les espaces d’adressage que vous spécifiez ne se chevauchent pas avec les espaces d’adressage du client pour les sites locaux auxquels vous souhaitez vous connecter. Si vos sous-réseaux se chevauchent, votre connexion ne fonctionnera pas correctement.

* Si vous disposez déjà d’un réseau virtuel, vérifiez que les paramètres sont compatibles avec la conception de votre passerelle VPN, avec une attention particulière pour tous les sous-réseaux qui pourraient chevaucher d’autres réseaux.

* Si vous n’avez pas de réseau virtuel, créez-en un. Les captures d’écran sont fournies à titre d’exemple. Assurez-vous de remplacer ces valeurs par les vôtres.

### <a name="to-create-a-virtual-network"></a>Pour créer un réseau virtuel

[!INCLUDE [basic classic vnet](../../includes/vpn-gateway-vnet-classic.md)]

[!INCLUDE [basic classic DNS](../../includes/vpn-gateway-dns-classic.md)]

## <a name="configure-the-site-and-gateway"></a><a name="localsite"></a>Configurer le site et la passerelle

### <a name="to-configure-the-site"></a>Pour configurer le site :

Le site local fait généralement référence à votre emplacement local. Il contient l’adresse IP du périphérique VPN avec lequel vous allez créer une connexion et les plages d’adresses IP qui seront acheminées via la passerelle VPN vers le périphérique VPN.

1. Sur la page de votre réseau virtuel, sous **Paramètres**, sélectionnez **Connexions de site à site**.
1. Dans la page Connexions de site à site, sélectionnez **+ Ajouter**.
1. Dans la page **Configurer une connexion VPN et une passerelle**, pour **Type de connexion**, gardez l’option **Site à site** sélectionnée. Pour cet exercice, vous devrez utiliser un mélange des [valeurs d’exemple](#values) et de vos propres valeurs.

   * **Adresse IP de la passerelle VPN :** Adresse IP publique du périphérique VPN pour votre réseau local. Le périphérique VPN requiert une adresse IP IPv4 publique. Spécifiez une adresse IP publique valide pour le périphérique VPN auquel vous souhaitez vous connecter. Il doit être accessible par Azure. Si vous ne connaissez pas l’adresse IP de votre périphérique VPN, vous pouvez toujours placer une valeur d’espace réservé (à condition qu’elle soit au format d’une adresse IP publique valide) et la modifier ultérieurement.

   * **Espace d’adressage du client :** Listez les plages d’adresses IP que vous voulez router vers le réseau local par le biais de cette passerelle. Vous pouvez ajouter plusieurs plages d’espaces d’adressage. Assurez-vous que les plages que vous spécifiez ici ne se chevauchent pas avec des plages d’adresses d’autres réseaux auxquels votre réseau virtuel se connecte, ou avec les propres plages d’adresses du réseau virtuel.
1. En bas de la page, NE sélectionnez PAS passer Vérifier + créer. Au lieu de cela, sélectionnez **Suivant : Passerelle >** .

### <a name="to-configure-the-virtual-network-gateway"></a><a name="sku"></a>Pour configurer la passerelle de réseau virtuel

1. Sur la page **Passerelle**, sélectionnez les valeurs suivantes :

   * **Taille :** Il s’agit de la référence de passerelle que vous utilisez pour créer votre passerelle de réseau virtuel. Les passerelles VPN classiques utilisent les anciennes références SKU de passerelles. Pour en savoir plus sur les anciennes références SKU de passerelle, consultez la section [Utilisation des références SKU de passerelle de réseau virtuel (anciennes références SKU)](vpn-gateway-about-skus-legacy.md). Vous pouvez sélectionner **Standard** pour cet exercice.

   * **Type de routage :** Sélectionnez le type de routage pour votre passerelle. Cela correspond également au type de VPN. Il est important de sélectionner le type approprié, car vous ne pouvez pas convertir la passerelle d’un type à un autre. Votre périphérique VPN doit être compatible avec le type de routage que vous sélectionnez. Pour plus d’informations sur le type de routage, consultez [À propos des paramètres de la passerelle VPN](vpn-gateway-about-vpn-gateway-settings.md#vpntype). Certains articles font référence aux types de VPN « RouteBased » et « PolicyBased ». « RouteBased » correspond à un routage dynamique, et « PolicyBased » à un routage statique. En général, vous souhaitez un routage dynamique.

   * **Sous-réseau de passerelle** : La taille du sous-réseau de passerelle que vous spécifiez dépend de la configuration de la passerelle VPN que vous souhaitez créer. Bien qu’il soit possible de créer un sous-réseau de passerelle aussi petit que /29, nous vous recommandons d’utiliser un sous-réseau /27 ou /28. Cette opération crée un sous-réseau plus grand qui inclut plusieurs adresses. En choisissant un sous-réseau de passerelle plus vaste, vous disposez de suffisamment d’adresses IP pour prendre en charge d’éventuelles configurations futures.

1. Au bas de la page, sélectionnez **Vérifier + créer** pour confirmer vos paramètres. Sélectionnez **Créer** pour effectuer le déploiement. Selon la référence SKU que vous sélectionnez, cela peut prendre jusqu’à 45 minutes pour créer une passerelle de réseau virtuel.

## <a name="configure-your-vpn-device"></a><a name="vpndevice"></a>Configurer votre périphérique VPN

Les connexions site à site vers un réseau local nécessitent un périphérique VPN. Dans cette étape, vous configurez votre périphérique VPN. Lorsque vous configurez votre périphérique VPN, vous avez besoin des valeurs suivantes :

* Une clé partagée. Il s’agit de la clé partagée spécifiée lors de la création de la connexion VPN de site à site. Dans nos exemples, nous utilisons une clé partagée basique. Nous vous conseillons de générer une clé plus complexe.
* L’adresse IP publique de votre passerelle de réseau virtuel. Vous pouvez afficher l’adresse IP publique à l’aide du portail Azure, de PowerShell ou de l’interface de ligne de commande.

[!INCLUDE [vpn-gateway-configure-vpn-device-rm](../../includes/vpn-gateway-configure-vpn-device-rm-include.md)]

## <a name="retrieve-values"></a><a name="getvalues"></a>Récupérer les valeurs

[!INCLUDE [retrieve values](../../includes/vpn-gateway-values-classic.md)]

## <a name="create-the-connection"></a><a name="CreateConnection"></a>Créer la connexion

> [!NOTE]
> Pour le modèle de déploiement classique, cette étape n’est pas disponible dans le portail Azure ni via Azure Cloud Shell. Vous devez utiliser la version Management des services (SM) des cmdlets Azure PowerShell localement à partir de votre bureau.
>

Dans cette étape, en utilisant les valeurs des étapes précédentes, vous définissez la clé partagée et créez la connexion. La clé que vous définissez doit être la même que celle qui a été utilisée lors la configuration de votre périphérique VPN.

1. Définissez la clé partagée et créez la connexion.

   * Modifiez les valeurs -VNetName et -LocalNetworkSiteName. Lorsque vous spécifiez un nom qui contient des espaces, encadrez la valeur avec des guillemets.
   * « -SharedKey » est une valeur que vous pouvez générer, puis spécifier. Dans l’exemple, nous avons utilisé « abc123 », mais vous pouvez (et devriez) générer une valeur plus complexe. L’important, c’est que la valeur que vous spécifiez ici doit être identique à celle spécifiée lors de la configuration de votre périphérique VPN.

   ```powershell
   Set-AzureVNetGatewayKey -VNetName 'Group TestRG1 TestVNet1' `
   -LocalNetworkSiteName '6C74F6E6_Site2' -SharedKey abc123
   ```

1. Quand la connexion est créée, le résultat est : **État : Réussi**.

## <a name="verify-your-connection"></a><a name="verify"></a>Vérifier votre connexion

[!INCLUDE [vpn-gateway-verify-connection-azureportal-classic](../../includes/vpn-gateway-verify-connection-azureportal-classic-include.md)]

Si vous rencontrez des problèmes de connexion, consultez la section **Dépanner** dans la table des matières du volet gauche.

## <a name="how-to-reset-a-vpn-gateway"></a><a name="reset"></a>Réinitialisation d’une passerelle VPN

La réinitialisation d’une passerelle VPN Azure est utile si vous perdez la connectivité VPN entre différents locaux sur un ou plusieurs tunnels VPN de site à site. Dans ce cas, vos périphériques VPN sur site fonctionnent tous correctement, mais ils ne sont pas en mesure d’établir des tunnels IPsec avec les passerelles VPN Azure. Pour obtenir la procédure, consultez [Réinitialiser une passerelle VPN](./reset-gateway.md#resetclassic).

## <a name="how-to-change-a-gateway-sku"></a><a name="changesku"></a>Modification d’une référence SKU de passerelle

Pour obtenir la procédure permettant de modifier une référence SKU de passerelle, consultez [Redimensionner une référence SKU de passerelle](vpn-gateway-about-SKUS-legacy.md#classicresize).

## <a name="next-steps"></a>Étapes suivantes

* Une fois la connexion achevée, vous pouvez ajouter des machines virtuelles à vos réseaux virtuels. Pour plus d’informations, consultez [Machines virtuelles](../index.yml).
* Pour plus d’informations sur le tunneling forcé, consultez [Configuration du tunneling forcé à l’aide du modèle de déploiement classique](vpn-gateway-about-forced-tunneling.md).