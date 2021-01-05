---
title: 'Configurer le chiffrement ExpressRoute : IPsec sur ExpressRoute pour Azure Virtual WAN'
description: Découvrez comment utiliser Azure Virtual WAN pour créer une connexion VPN site à site par peering privé ExpressRoute.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: how-to
ms.date: 09/22/2020
ms.author: cherylmc
ms.openlocfilehash: b8dde3ed76587e2343edaec8626287853ec6ef9b
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96487505"
---
# <a name="expressroute-encryption-ipsec-over-expressroute-for-virtual-wan"></a>Chiffrement ExpressRoute : IPsec sur ExpressRoute pour Virtual WAN

Cet article explique comment utiliser Azure Virtual WAN pour établir une connexion VPN IPsec/IKE d’un réseau local vers Azure par Peering privé d’un circuit Azure ExpressRoute. Cette technique peut assurer un transit chiffré entre les réseaux locaux et les réseaux virtuels Azure par ExpressRoute, sans passer par l’Internet public ni utiliser des IP publiques.

## <a name="topology-and-routing"></a>Topologie et routage

Le diagramme suivant illustre un exemple de connectivité VPN par Peering privé ExpressRoute :

:::image type="content" source="./media/vpn-over-expressroute/vwan-vpn-over-er.png" alt-text="VPN par ExpressRoute":::

Le diagramme illustre un réseau du réseau local connecté à la passerelle VPN du hub Azure par peering privé ExpressRoute. L’établissement de la connectivité est simple :

1. Établir une connectivité ExpressRoute avec un circuit ExpressRoute et un Peering privé.
2. Établir la connectivité VPN comme l’explique cet article.

L’un des aspects importants de cette configuration est le routage entre les réseaux locaux et Azure sur le chemin VPN et le chemin ExpressRoute.

### <a name="traffic-from-on-premises-networks-to-azure"></a>Trafic de réseaux locaux vers Azure

Pour le trafic de réseaux locaux vers Azure, les préfixes Azure (y compris le hub virtuel et tous les réseaux virtuels spoke qui y sont connectés) sont publiés via le protocole BGP de Peering privé ExpressRoute et le protocole BGP de VPN. Cela aboutit à deux itinéraires réseau (chemins d’accès) vers Azure à partir des réseaux locaux :

- Un sur le chemin d’accès protégé par IPsec
- Un directement sur ExpressRoute *sans* protection IPsec 

Pour appliquer le chiffrement à la communication, vous devez vous assurer que les itinéraires Azure qui passent par la passerelle VPN locale sont préférés au chemin ExpressRoute direct pour le réseau connecté par VPN dans le diagramme.

### <a name="traffic-from-azure-to-on-premises-networks"></a>Trafic d’Azure vers des réseaux locaux

La même condition s’applique au trafic d’Azure vers les réseaux locaux. Il existe deux moyens de faire en sorte que le chemin IPsec soit préféré au chemin ExpressRoute direct (sans IPsec) :

- Publiez des préfixes plus spécifiques sur la session BGP VPN pour le réseau connecté par VPN : Vous pouvez publier une plus grande plage qui englobe le réseau connecté par VPN par Peering privé ExpressRoute, puis des plages plus spécifiques dans la session BGP VPN. Par exemple, publiez 10.0.0.0/16 par ExpressRoute et 10.0.1.0/24 par VPN.

- Publiez des préfixes disjoints pour VPN et ExpressRoute. Si les plages de réseaux connectés par VPN sont disjointes d’autres réseaux connectés par ExpressRoute, vous pouvez publier les préfixes respectivement dans les sessions VPN et BGP ExpressRoute. Par exemple, publiez 10.0.0.0/24 par ExpressRoute et 10.0.1.0/24 par VPN.

Dans ces deux exemples, Azure envoie le trafic à 10.0.1.0/24 via la connexion VPN, plutôt que directement par ExpressRoute sans protection VPN.

> [!WARNING]
> Si vous publiez les *mêmes* préfixes sur les connexions ExpressRoute et VPN, Azure utilisera directement le chemin ExpressRoute sans protection VPN.
>

## <a name="before-you-begin"></a>Avant de commencer

[!INCLUDE [Before you begin](../../includes/virtual-wan-tutorial-vwan-before-include.md)]

## <a name="1-create-a-virtual-wan-and-hub-with-gateways"></a><a name="openvwan"></a>1. Créer un réseau Virtual WAN et un hub avec des passerelles

Avant de continuer, les ressources Azure suivantes et les configurations locales correspondantes doivent être en place :

- Un réseau Azure Virtual WAN
- Un hub Virtual WAN avec une passerelle [ExpressRoute](virtual-wan-expressroute-portal.md) et une [passerelle VPN](virtual-wan-site-to-site-portal.md)

Pour connaître les étapes de création d’un réseau étendu (WAN) virtuel Azure et d’un hub avec une association ExpressRoute, consultez [Créer une association ExpressRoute à l’aide d’Azure Virtual WAN](virtual-wan-expressroute-portal.md). Pour connaître les étapes de création d’une passerelle VPN dans le WAN virtuel, consultez [Créer une connexion site à site à l’aide d’Azure Virtual WAN](virtual-wan-site-to-site-portal.md).

## <a name="2-create-a-site-for-the-on-premises-network"></a><a name="site"></a>2. Créer un site pour le réseau local

La ressource de site est la même que les sites VPN non ExpressRoute pour un WAN virtuel. L’adresse IP du périphérique VPN local peut désormais être une IP privée, ou une IP publique dans le réseau local accessible par le Peering privé ExpressRoute créée à l’étape 1.

> [!NOTE]
> L’adresse IP du périphérique VPN local *doit* faire partie des préfixes d’adresse publiés sur le hub de WAN virtuel par Peering privé Azure ExpressRoute.
>

1. Dans votre navigateur, accédez au Portail Azure. 
1. Sélectionnez le hub que vous avez créé. Sur la page du hub Réseau étendu virtuel, sous **Connectivité**, sélectionnez **Sites VPN**.
1. Sur la page **Sites VPN**, sélectionnez **+Créer un site**.
1. Sur la page **Créer un site**, renseignez les champs suivants :
   * **Abonnement**: Vérifiez l’abonnement.
   * **Groupe de ressources** : Sélectionnez ou créez le groupe de ressources à utiliser.
   * **Région** : Entrez la région Azure pour la ressource de site VPN.
   * **Name** : Entrez le nom par lequel vous souhaitez faire référence à votre site local.
   * **Fournisseur de l’appareil** : Entrez le fournisseur du périphérique VPN local.
   * **Border Gateway Protocol** : Sélectionnez « Activer » si votre réseau local utilise le protocole BGP.
   * **Espace d’adressage privé** : Entrez l’espace d’adressage IP qui se trouve sur votre site local. Le trafic destiné à cet espace d’adressage est routé vers le réseau local via la passerelle VPN.
   * **Hubs** : Sélectionnez un ou plusieurs hubs pour connecter ce site VPN. Les hubs sélectionnés doivent disposer déjà de passerelles VPN.
1. Sélectionnez **Suivant : Liens >** pour les paramètres de liaison VPN :
   * **Nom de la liaison** : Nom par lequel vous souhaitez faire référence à cette connexion.
   * **Nom du fournisseur** : Nom du fournisseur de services Internet de ce site. Dans le cas d’un réseau local ExpressRoute, il s’agit du nom du fournisseur de services ExpressRoute.
   * **Vitesse** : Vitesse de la liaison du service Internet ou du circuit ExpressRoute.
   * **Adresse IP** : IP publique du périphérique VPN qui réside sur votre site local. Ou, dans le cas d’un réseau local ExpressRoute, IP privée du périphérique VPN par ExpressRoute.

   Si le protocole BGP est activé, il s’applique à toutes les connexions créées pour ce site dans Azure. La configuration du protocole BGP sur un WAN virtuel équivaut à configurer le protocole BGP sur une passerelle VPN Azure. 
   
   L’adresse du peer BGP local *ne doit pas* être identique à l’adresse IP du réseau VPN vers l’appareil ou à l’espace d’adressage du réseau virtuel du site VPN. Utilisez une adresse IP différente sur le périphérique VPN de votre adresse IP BGP homologue. Il peut s’agir d’une adresse affectée à l’interface de bouclage sur le périphérique. Toutefois, il *ne peut pas* s’agir d’une adresse APIPA (169.254.*x*.*x*). Spécifiez cette adresse sur le site VPN correspondant qui représente l’emplacement. Pour les conditions préalables BGP, consultez [À propos de BGP avec la passerelle VPN Azure](../vpn-gateway/vpn-gateway-bgp-overview.md).

1. Sélectionnez **Suivant : Vérifier + créer >** pour vérifier les valeurs des paramètres et créer le site VPN. Si vous avez sélectionné **Hubs** pour vous connecter, la connexion sera établie entre le réseau local et la passerelle VPN du hub.

## <a name="3-update-the-vpn-connection-setting-to-use-expressroute"></a><a name="hub"></a>3. Mettre à jour le paramètre de connexion VPN de façon à utiliser ExpressRoute

Après avoir créé le site VPN et établi la connexion au hub, suivez les étapes ci-dessous pour configurer la connexion de façon à utiliser le Peering privé ExpressRoute :

1. Revenez à la page des ressources du WAN virtuel, puis sélectionnez la ressource du hub ou accédez au hub connecté à partir du site VPN.

   :::image type="content" source="./media/vpn-over-expressroute/hub-selection.png" alt-text="Sélectionner un hub":::
1. Sous **Connectivité**, sélectionnez **VPN (site à site)**.

   :::image type="content" source="./media/vpn-over-expressroute/vpn-select.png" alt-text="Sélectionnez VPN (site à site)":::
1. Sélectionnez les points de suspension (**…**) sur le site VPN par ExpressRoute, puis **Modifier la connexion VPN à ce hub**.

   :::image type="content" source="./media/vpn-over-expressroute/config-menu.png" alt-text="Menu Entrer les configurations":::
1. Pour **Utiliser une adresse IP privée Azure**, sélectionnez **Oui**. Le paramètre configure la passerelle VPN du hub de façon à utiliser des adresses IP privées situées dans la plage d’adresses du hub sur la passerelle pour cette connexion, au lieu des adresses IP publiques. Ainsi, le trafic provenant du réseau local parcourra les chemins de Peering privé ExpressRoute plutôt que d’utiliser l’Internet public pour cette connexion VPN. La capture d’écran qui suit présente le réglage :

   :::image type="content" source="./media/vpn-over-expressroute/vpn-link-configuration.png" alt-text="Réglage de l’utilisation d’une adresse IP privée pour la connexion VPN" border="false":::
1. Sélectionnez **Enregistrer**.

Une fois que vous avez enregistré vos modifications, la passerelle VPN du hub utilisera les adresses IP privées pour établir les connexions IPsec/IKE avec le périphérique VPN local par ExpressRoute.

## <a name="4-get-the-private-ip-addresses-for-the-hub-vpn-gateway"></a><a name="associate"></a>4. Récupérer les adresses IP privées de la passerelle VPN du hub

Téléchargez la configuration du périphérique VPN pour obtenir les adresses IP privées de la passerelle VPN du hub. Vous avez besoin de ces adresses pour configurer le périphérique VPN local.

1. Sur la page de votre hub, sélectionnez **VPN (site à site)** sous **Connectivité**.
1. En haut de la page **Vue d’ensemble**, sélectionnez **Télécharger la configuration VPN**. 

   Azure crée un compte de stockage dans le groupe de ressources « microsoft-network-[location] », où *location* est l’emplacement du réseau étendu. Une fois que vous avez appliqué la configuration à vos périphériques VPN, vous pouvez supprimer ce compte de stockage.
1. Une fois le fichier créé, sélectionnez le lien pour le télécharger.
1. Appliquez la configuration à votre périphérique VPN.

### <a name="vpn-device-configuration-file"></a>Fichier config du périphérique VPN

Le fichier config du périphérique contient les paramètres à utiliser lors de la configuration de votre périphérique VPN local. Lorsque vous affichez ce fichier, notez les informations suivantes :

* **vpnSiteConfiguration** : Cette section indique les détails de l’appareil configuré comme un site se connectant au WAN virtuel. Cela inclut le nom et l’IP publique du périphérique de branche.
* **vpnSiteConnections** : Cette section fournit des informations sur les paramètres suivants :

    * Espace d’adressage du réseau virtuel du hub virtuel.<br/>Exemple :
           ```
           "AddressSpace":"10.51.230.0/24"
           ```
    * Espace d’adressage des réseaux virtuels qui sont connectés au hub.<br>Exemple :
           ```
           "ConnectedSubnets":["10.51.231.0/24"]
            ```
    * Adresses IP de la passerelle VPN du hub virtuel. Étant donné que chaque connexion de la passerelle VPN est composée de deux tunnels en configuration actif-actif, vous pouvez voir les deux adresses IP répertoriées dans ce fichier. Dans cet exemple, `Instance0` et `Instance1` apparaissent pour chaque site ; il s’agit d’adresses IP privées et non d’IP publiques.<br>Exemple :
           ``` 
           "Instance0":"10.51.230.4"
           "Instance1":"10.51.230.5"
           ```
    * Détails de configuration de la connexion de la passerelle VPN, comme un protocole BGP et une clé prépartagée. La clé prépartagée est automatiquement générée pour vous. Vous pouvez toujours modifier la connexion dans la page **Vue d’ensemble** pour une clé prépartagée personnalisée.
  
### <a name="example-device-configuration-file"></a>Exemple de fichier de configuration de périphérique

```
[{
      "configurationVersion":{
        "LastUpdatedTime":"2019-10-11T05:57:35.1803187Z",
        "Version":"5b096293-edc3-42f1-8f73-68c14a7c4db3"
      },
      "vpnSiteConfiguration":{
        "Name":"VPN-over-ER-site",
        "IPAddress":"172.24.127.211",
        "LinkName":"VPN-over-ER"
      },
      "vpnSiteConnections":[{
        "hubConfiguration":{
          "AddressSpace":"10.51.230.0/24",
          "Region":"West US 2",
          "ConnectedSubnets":["10.51.231.0/24"]
        },
        "gatewayConfiguration":{
          "IpAddresses":{
            "Instance0":"10.51.230.4",
            "Instance1":"10.51.230.5"
          }
        },
        "connectionConfiguration":{
          "IsBgpEnabled":false,
          "PSK":"abc123",
          "IPsecParameters":{"SADataSizeInKilobytes":102400000,"SALifeTimeInSeconds":3600}
        }
      }]
    },
    {
      "configurationVersion":{
        "LastUpdatedTime":"2019-10-11T05:57:35.1803187Z",
        "Version":"fbdb34ea-45f8-425b-9bc2-4751c2c4fee0"
      },
      "vpnSiteConfiguration":{
        "Name":"VPN-over-INet-site",
        "IPAddress":"13.75.195.234",
        "LinkName":"VPN-over-INet"
      },
      "vpnSiteConnections":[{
        "hubConfiguration":{
          "AddressSpace":"10.51.230.0/24",
          "Region":"West US 2",
          "ConnectedSubnets":["10.51.231.0/24"]
        },
        "gatewayConfiguration":{
          "IpAddresses":{
            "Instance0":"51.143.63.104",
            "Instance1":"52.137.90.89"
          }
        },
        "connectionConfiguration":{
          "IsBgpEnabled":false,
          "PSK":"abc123",
          "IPsecParameters":{"SADataSizeInKilobytes":102400000,"SALifeTimeInSeconds":3600}
        }
      }]
}]
```

### <a name="configuring-your-vpn-device"></a>Configuration de votre périphérique VPN

Si vous avez besoin d’instructions pour configurer votre périphérique, vous pouvez utiliser les instructions de la [page de scripts de configuration de périphérique VPN](~/articles/vpn-gateway/vpn-gateway-about-vpn-devices.md#configscripts) en prenant les précautions suivantes :

* Les instructions figurant sur la page du périphérique VPN ne sont pas écrites pour un WAN virtuel. Toutefois, vous pouvez utiliser les valeurs de WAN virtuel du fichier config pour configurer manuellement votre périphérique VPN. 
* Les scripts téléchargeables de configuration de périphérique qui sont pour la passerelle VPN ne fonctionnent pas pour le WAN virtuel, car la configuration est différente.
* Un nouveau WAN virtuel peut prendre en charge IKEv1 et IKEv2.
* Le WAN virtuel peut uniquement utiliser des périphériques VPN basés sur un itinéraire et des instructions de périphérique.

## <a name="5-view-your-virtual-wan"></a><a name="viewwan"></a>5. Afficher votre WAN virtuel

1. Accédez au WAN virtuel.
1. Sur la page **Vue d’ensemble**, chaque point sur la carte représente un hub.
1. Dans la section **Hubs et connexions**, vous pouvez afficher l’état du hub, du site, de la région et de la connexion VPN. Vous pouvez également afficher les octets entrés et sortis.

## <a name="6-monitor-a-connection"></a><a name="connectmon"></a>6. Surveiller une connexion

Créez une connexion pour surveiller les communications entre une machine virtuelle Azure et un site distant. Pour plus d’informations sur la façon de configurer un contrôleur de connexion, consultez [Surveiller une communication réseau](~/articles/network-watcher/connection-monitor.md). Le champ source est l’adresse IP de la machine virtuelle dans Azure, et l’adresse IP de destination est le l’adresse IP du site.

## <a name="7-clean-up-resources"></a><a name="cleanup"></a>7. Nettoyer les ressources

Quand vous n’avez plus besoin de ces ressources, vous pouvez utiliser [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) pour supprimer le groupe de ressources et toutes les ressources qu’il contient. Exécutez la commande PowerShell suivante et remplacez `myResourceGroup` par le nom de votre groupe de ressources :

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>Étapes suivantes

Cet article explique comment créer une connexion VPN par Peering privé ExpressRoute à l’aide de Virtual WAN. Pour plus d’informations sur Virtual WAN et les fonctionnalités associées, consultez la [Vue d’ensemble de Virtual WAN](virtual-wan-about.md).
