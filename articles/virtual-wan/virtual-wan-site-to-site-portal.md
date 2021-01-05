---
title: 'Tutoriel : Utiliser Azure Virtual WAN pour créer des connexions de site à site'
description: Dans ce didacticiel, découvrez comment utiliser le WAN virtuel Azure pour créer une connexion VPN de site à site vers Azure.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: tutorial
ms.date: 10/08/2020
ms.author: cherylmc
Customer intent: As someone with a networking background, I want to connect my local site to my VNets using Virtual WAN and I don't want to go through a Virtual WAN partner.
ms.openlocfilehash: 7ba0f1b6f37da923e389964b99a02295dc3d6050
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94359525"
---
# <a name="tutorial-create-a-site-to-site-connection-using-azure-virtual-wan"></a>Tutoriel : Créer une connexion de site à site à l’aide d’Azure Virtual WAN

Ce didacticiel vous montre comment utiliser Virtual WAN pour vous connecter à vos ressources dans Azure via une connexion VPN IPsec/IKE (IKEv1 et IKEv2). Ce type de connexion requiert un périphérique VPN local disposant d’une adresse IP publique exposée en externe. Pour plus d’informations sur Azure Virtual WAN, consultez l’article [Vue d’ensemble d’Azure Virtual WAN](virtual-wan-about.md).

Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * Créer un WAN virtuel
> * Créer un hub
> * Créer un site
> * Connecter un site à un hub
> * Connecter un site VPN à un hub
> * Connecter un réseau virtuel à un hub
> * Télécharger un fichier de configuration
> * Configurer votre passerelle VPN

> [!NOTE]
> Si vous avez de nombreux sites, vous utiliseriez généralement un [partenaire WAN virtuel](https://aka.ms/virtualwan) pour créer cette configuration. Toutefois, vous pouvez créer cette configuration vous-même si vous êtes familiarisé avec la mise en réseau et expert dans la configuration de votre propre périphérique VPN.
>

![Diagramme WAN virtuel](./media/virtual-wan-about/virtualwan.png)

## <a name="prerequisites"></a>Prérequis

Vérifiez que vous disposez des éléments ci-dessous avant de commencer votre configuration :

[!INCLUDE [Before you begin](../../includes/virtual-wan-before-include.md)]

## <a name="create-a-virtual-wan"></a><a name="openvwan"></a>Créer un WAN virtuel

[!INCLUDE [Create a virtual WAN](../../includes/virtual-wan-create-vwan-include.md)]

## <a name="create-a-hub"></a><a name="hub"></a>Créer un hub

Un hub est un réseau virtuel qui peut contenir des passerelles pour offrir des fonctionnalités de site à site, ExpressRoute ou de point à site. Une fois le hub créé, vous serez facturé, même si vous n’y joignez aucun site. La création de la passerelle VPN de site à site prend 30 minutes dans le hub virtuel.

[!INCLUDE [Create a hub](../../includes/virtual-wan-tutorial-s2s-hub-include.md)]

## <a name="create-a-site"></a><a name="site"></a>Créer un site

Vous êtes maintenant prêt à créer les sites correspondant à vos emplacements physiques. Créez autant de sites que nécessaire correspondant à vos emplacements physiques. Par exemple, si vous avez une succursale à New York, une autre à Londres et une autre à LA, vous créeriez trois sites distincts. Ces sites contiennent vos points de terminaison du périphérique VPN local. Vous pouvez créer jusqu’à 1 000 sites par hub virtuel dans Virtual WAN. Si vous aviez plusieurs hubs, vous pouvez créer 1 000 sites pour chacun de ces hubs. Si vous avez un appareil CPE de partenaire Virtual WAN (lien d’insertion), contactez ce dernier pour en savoir plus sur son automatisation sur Azure. En général, l’automatisation permet d’exporter en un seul clic des informations de branche à grande échelle dans Azure, et de configurer la connectivité entre l’appareil CPE et la passerelle VPN Azure Virtual WAN. Pour plus d’informations, consultez [Guide d’automatisation Azure à l’intention des partenaires CPE](virtual-wan-configure-automation-providers.md).

[!INCLUDE [Create a site](../../includes/virtual-wan-tutorial-s2s-site-include.md)]

## <a name="connect-the-vpn-site-to-the-hub"></a><a name="connectsites"></a>Connecter le site VPN au hub

Au cours de cette étape, vous connectez votre site VPN au hub.

[!INCLUDE [Connect VPN sites](../../includes/virtual-wan-tutorial-s2s-connect-vpn-site-include.md)]

## <a name="connect-the-vnet-to-the-hub"></a><a name="vnet"></a>Connecter le réseau virtuel au hub

[!INCLUDE [Connect](../../includes/virtual-wan-connect-vnet-hub-include.md)]

## <a name="download-vpn-configuration"></a><a name="device"></a>Télécharger une configuration VPN

Utilisez la configuration de périphérique VPN pour configurer votre périphérique VPN local.

1. Dans la page de votre réseau étendu virtuel, cliquez sur **Vue d’ensemble**.
2. En haut de la page **Hub ->VPNSite** , cliquez sur **Télécharger la configuration VPN**. Azure crée un compte de stockage dans le groupe de ressources « microsoft-network-[emplacement] », où emplacement est l’emplacement du réseau étendu. Une fois que vous avez appliqué la configuration à vos périphériques VPN, vous pouvez supprimer ce compte de stockage.
3. Une fois que le fichier a terminé la création, vous pouvez cliquer sur le lien pour le télécharger.
4. Appliquez la configuration à votre périphérique VPN local.

### <a name="understanding-the-vpn-device-configuration-file"></a>Comprendre le fichier de configuration de périphérique VPN

Le fichier de configuration de périphérique contient les paramètres à utiliser lors de la configuration de votre périphérique VPN sur site. Lorsque vous affichez ce fichier, notez les informations suivantes :

* **vpnSiteConfiguration -** Cette section indique les détails de l’appareil configuré comme un site se connectant au réseau virtuel étendu. Cela inclut le nom et l’adresse IP publique de l’appareil de branche.
* **vpnSiteConnections -** Cette section fournit des informations sur les paramètres suivants :

    * **Espace d’adressage** du réseau virtuel du/des hub(s)<br>Exemple :
 
        ```
        "AddressSpace":"10.1.0.0/24"
        ```
    * **Espace d’adressage** des réseaux virtuels qui sont connectés au hub<br>Exemple :

         ```
        "ConnectedSubnets":["10.2.0.0/16","10.3.0.0/16"]
         ```
    * **Adresses IP** de la passerelle VPN virtuelle. Étant donné que chaque connexion effectuée avec la passerelle VPN est composée de deux tunnels en configuration actif-actif, vous pouvez voir les deux adresses IP répertoriées dans ce fichier. Dans cet exemple, vous voyez « Instance0 » et « Instance1 » pour chaque site.<br>Exemple :

        ``` 
        "Instance0":"104.45.18.186"
        "Instance1":"104.45.13.195"
        ```
    * **Détails de configuration de connexion de passerelle VPN** , comme BGP, une clé prépartagée, etc. La clé PSK est la clé prépartagée automatiquement générée pour vous. Vous pouvez toujours modifier la connexion dans la page Vue d’ensemble pour une clé PSK personnalisée.
  
### <a name="example-device-configuration-file"></a>Exemple de fichier de configuration de périphérique

  ```
  { 
      "configurationVersion":{ 
         "LastUpdatedTime":"2018-07-03T18:29:49.8405161Z",
         "Version":"r403583d-9c82-4cb8-8570-1cbbcd9983b5"
      },
      "vpnSiteConfiguration":{ 
         "Name":"testsite1",
         "IPAddress":"73.239.3.208"
      },
      "vpnSiteConnections":[ 
         { 
            "hubConfiguration":{ 
               "AddressSpace":"10.1.0.0/24",
               "Region":"West Europe",
               "ConnectedSubnets":[ 
                  "10.2.0.0/16",
                  "10.3.0.0/16"
               ]
            },
            "gatewayConfiguration":{ 
               "IpAddresses":{ 
                  "Instance0":"104.45.18.186",
                  "Instance1":"104.45.13.195"
               }
            },
            "connectionConfiguration":{ 
               "IsBgpEnabled":false,
               "PSK":"bkOWe5dPPqkx0DfFE3tyuP7y3oYqAEbI",
               "IPsecParameters":{ 
                  "SADataSizeInKilobytes":102400000,
                  "SALifeTimeInSeconds":3600
               }
            }
         }
      ]
   },
   { 
      "configurationVersion":{ 
         "LastUpdatedTime":"2018-07-03T18:29:49.8405161Z",
         "Version":"1f33f891-e1ab-42b8-8d8c-c024d337bcac"
      },
      "vpnSiteConfiguration":{ 
         "Name":" testsite2",
         "IPAddress":"66.193.205.122"
      },
      "vpnSiteConnections":[ 
         { 
            "hubConfiguration":{ 
               "AddressSpace":"10.1.0.0/24",
               "Region":"West Europe"
            },
            "gatewayConfiguration":{ 
               "IpAddresses":{ 
                  "Instance0":"104.45.18.187",
                  "Instance1":"104.45.13.195"
               }
            },
            "connectionConfiguration":{ 
               "IsBgpEnabled":false,
               "PSK":"XzODPyAYQqFs4ai9WzrJour0qLzeg7Qg",
               "IPsecParameters":{ 
                  "SADataSizeInKilobytes":102400000,
                  "SALifeTimeInSeconds":3600
               }
            }
         }
      ]
   },
   { 
      "configurationVersion":{ 
         "LastUpdatedTime":"2018-07-03T18:29:49.8405161Z",
         "Version":"cd1e4a23-96bd-43a9-93b5-b51c2a945c7"
      },
      "vpnSiteConfiguration":{ 
         "Name":" testsite3",
         "IPAddress":"182.71.123.228"
      },
      "vpnSiteConnections":[ 
         { 
            "hubConfiguration":{ 
               "AddressSpace":"10.1.0.0/24",
               "Region":"West Europe"
            },
            "gatewayConfiguration":{ 
               "IpAddresses":{ 
                  "Instance0":"104.45.18.187",
                  "Instance1":"104.45.13.195"
               }
            },
            "connectionConfiguration":{ 
               "IsBgpEnabled":false,
               "PSK":"YLkSdSYd4wjjEThR3aIxaXaqNdxUwSo9",
               "IPsecParameters":{ 
                  "SADataSizeInKilobytes":102400000,
                  "SALifeTimeInSeconds":3600
               }
            }
         }
      ]
   }
  ```

### <a name="configuring-your-vpn-device"></a>Configuration de votre périphérique VPN

>[!NOTE]
> Si vous utilisez une solution de partenaire Virtual WAN, la configuration de l’appareil VPN est automatique. Le contrôleur de l’appareil obtient le fichier config d’Azure et l’applique à l’appareil pour configurer la connexion à Azure. Cela signifie que vous n’avez pas besoin de savoir comment configurer manuellement votre périphérique VPN.
>

Si vous avez besoin d’instructions pour configurer votre périphérique, vous pouvez utiliser les instructions de la [page de scripts de configuration de périphérique VPN](~/articles/vpn-gateway/vpn-gateway-about-vpn-devices.md#configscripts) en prenant les précautions suivantes :

* Les instructions sur la page des périphériques VPN ne sont pas écrites pour un réseau étendu virtuel, mais vous pouvez utiliser les valeurs WAN virtuel à partir du fichier de configuration pour configurer manuellement votre périphérique VPN. 
* Les scripts de configuration de périphérique téléchargeables qui sont pour la passerelle VPN ne fonctionnent pas pour le réseau étendu virtuel, étant donné que la configuration est différente.
* Un nouvel élément Virtual WAN peut prendre en charge IKEv1 et IKEv2.
* Virtual WAN peut utiliser des appareils VPN basés sur une stratégie et sur une route, et des instructions d’appareil.

## <a name="configure-your-vpn-gateway"></a><a name="gateway-config"></a>Configurer votre passerelle VPN

Vous pouvez afficher et configurer les paramètres de votre passerelle VPN à tout moment en sélectionnant **Afficher/Configurer**.

:::image type="content" source="media/virtual-wan-site-to-site-portal/view-configuration-1.png" alt-text="Capture d’écran montrant la page « VPN (site à site) » avec une flèche pointant vers l’action « Afficher/configurer »." lightbox="media/virtual-wan-site-to-site-portal/view-configuration-1-expand.png":::

Sur la page **Modifier la passerelle VPN** , vous pouvez voir les paramètres suivants :

* Adresse IP publique de la passerelle VPN (affectée par Azure)
* Adresse IP privée de la passerelle VPN (affectée par Azure)
* Adresse IP BGP par défaut de la passerelle VPN (affectée par Azure)
* Option de configuration de l’adresse IP BGP personnalisée : Ce champ est réservé à APIPA (Automatic Private IP Addressing). Azure prend en charge l’adresse IP BGP dans les plages 169.254.21.* et 169.254.22.*. Azure accepte les connexions BGP dans ces plages, mais établit la connexion avec l’adresse IP BGP par défaut.

   :::image type="content" source="media/virtual-wan-site-to-site-portal/view-configuration-2.png" alt-text="Afficher la configuration" lightbox="media/virtual-wan-site-to-site-portal/view-configuration-2-expand.png":::

## <a name="clean-up-resources"></a><a name="cleanup"></a>Supprimer des ressources

Quand vous n’avez plus besoin de ces ressources, vous pouvez utiliser [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) pour supprimer le groupe de ressources et toutes les ressources qu’il contient. Remplacez « myResourceGroup » par le nom de votre groupe de ressources et exécutez la commande PowerShell suivante :

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>Étapes suivantes

Ensuite, pour en savoir plus sur Virtual WAN, consultez :

> [!div class="nextstepaction"]
> * [FAQ sur Virtual WAN](virtual-wan-faq.md)
