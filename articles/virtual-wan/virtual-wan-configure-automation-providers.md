---
title: Conseils d’automatisation pour les des partenaires Azure Virtual WAN | Microsoft Docs
description: Configurez un environnement d’automatisation pour la connexion et la configuration d’un réseau privé virtuel (VPN) local ou SD-WAN, ou un appareil de branche, pour Azure Virtual WAN.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 06/29/2020
ms.author: cherylmc
ms.openlocfilehash: 29fff3a6a430e3bc1a0b3a13876b55d22f7cb545
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94566467"
---
# <a name="automation-guidelines-for-virtual-wan-partners"></a>Conseils d’automatisation pour les partenaires Virtual WAN

Cet article vous aide à comprendre comment configurer l’environnement d’automatisation pour connecter et configurer un appareil de branche (un périphérique VPN client local ou SDWAN CPE) pour Azure Virtual WAN. Si vous fournissez des appareils de branche qui peuvent accommoder une connectivité VPN par IPsec/IKEv2 ou IPsec/IKEv1, cet article est pour vous.

Un appareil de branche (un périphérique VPN client local ou SDWAN CPE) utilise généralement un tableau de bord contrôleur/appareil pour le provisionnement. Les administrateurs de solutions SD-WAN peuvent souvent utiliser une console de gestion pour préprovisionner un appareil avant de le brancher au réseau. Ce périphérique compatible VPN obtient sa logique de plan de contrôle à partir d’un contrôleur. Le contrôleur du périphérique VPN ou SD-WAN peut utiliser des API Azure pour automatiser la connectivité à Azure Virtual WAN. Ce type de connexion nécessite que le périphérique local dispose d’une adresse IP publique exposée en externe.

## <a name="before-you-begin-automating"></a><a name ="before"></a>Avant de commencer l’automatisation

* Vérifiez que votre appareil prend en charge IPsec IKEv1/IKEv2. Voir [Stratégies par défaut](#default).
* Consultez les [API REST](#additional) à utiliser pour automatiser la connectivité à Azure Virtual WAN.
* Testez le portail d’Azure Virtual WAN.
* Ensuite, décidez quelle partie de la procédure de connexion vous souhaitez automatiser. Au minimum, nous vous recommandons d’automatiser les actions suivantes :

  * Contrôle d’accès
  * Chargement des informations sur l’appareil de branche dans Azure Virtual WAN
  * Téléchargement de la configuration Azure et configuration de la connectivité à partir de l’appareil de branche dans Azure Virtual WAN

### <a name="additional-information"></a><a name ="additional"></a>Informations supplémentaires

* [API REST](/rest/api/virtualwan/virtualhubs) pour automatiser la création d’un hub virtuel
* [API REST](/rest/api/virtualwan/vpngateways) pour automatiser la passerelle VPN Azure pour Virtual WAN
* [API REST](/rest/api/virtualwan/vpnconnections) pour connecter un VPNSite à un hub VPN Azure
* [Stratégies IPsec par défaut](#default)

## <a name="customer-experience"></a><a name ="ae"></a>Expérience client

Réfléchissez à l’expérience que votre client attend concernant Azure Virtual WAN.

  1. En règle générale, un utilisateur Virtual WAN démarrera le processus en créant une ressource Virtual WAN.
  2. L’utilisateur configurera un accès au groupe de ressources basé sur un principal de service pour le système local (votre contrôleur de branche ou votre logiciel de provisionnement des appareils VPN) pour écrire les informations sur la branche dans Azure Virtual WAN.
  3. À ce stade, l’utilisateur peut décider de se connecter à votre interface utilisateur et de définir les informations d’identification du principal de service. Une fois cette opération terminée, votre contrôleur doit être en mesure de charger les informations de branche avec l’automatisation que vous fournirez. L’équivalent manuel côté Azure est « Créer un site ».
  4. Une fois que les informations sur le site (appareil de filiale) sont disponibles dans Azure, l’utilisateur va connecter le site à un hub. Un hub virtuel est un réseau virtuel géré par Microsoft. Le hub contient différents points de terminaison de service pour activer la connectivité à partir de votre réseau local (vpnsite). Le hub est le cœur de votre réseau au sein d’une région spécifique. Il ne peut exister qu’un seul hub par région Azure et le point de terminaison vpn (vpngateway) qu’il contient est créé au cours de ce processus. La passerelle VPN est une passerelle évolutive dimensionnée en fonction des besoins en bande passante et de connexion. Vous pouvez choisir d’automatiser la création du hub virtuel et de vpngateway à partir du tableau de bord de votre contrôleur d’appareil de branche.
  5. Une fois le hub virtuel associé au site, un fichier de configuration est généré. L’utilisateur doit le télécharger manuellement. C’est à ce moment que votre automatisation entre en jeu, pour simplifier l’expérience utilisateur. Au lieu que l’utilisateur ait à télécharger et à configurer l’appareil de branche manuellement, vous pouvez définir l’automatisation et offrir une expérience nécessitant un minimum d’actions de la part de l’utilisateur sur votre interface utilisateur, limitant ainsi les problèmes de connectivité classiques tels que la non-correspondance de la clé partagée, la non-correspondance des paramètres IPSec, la lisibilité du fichier de configuration, etc.
  6. À la fin de cette étape dans votre solution, l’utilisateur profitera d’une connexion de site à site transparente entre l’appareil de branche et le hub virtuel. Vous pouvez également configurer des connexions supplémentaires entre d’autres hubs. Chaque connexion est un tunnel actif/actif. Votre client peut choisir d’utiliser un fournisseur d’accès différent pour chacun des liens du tunnel.
  7. Envisagez de fournir des fonctionnalités de dépannage et de supervision dans l’interface de gestion CPE. Les scénarios classiques sont par exemple « le client ne peut pas accéder aux ressources Azure en raison d’un problème de CPE », « afficher les paramètres IPsec côté CPE », etc.

## <a name="automation-details"></a><a name ="understand"></a>Détails de l’automatisation

###  <a name="access-control"></a><a name="access"></a>Contrôle d’accès

Les clients doivent pouvoir configurer un contrôle d'accès approprié pour le réseau WAN virtuel dans l’interface utilisateur de l’appareil. L’utilisation d’un principal de service Azure est recommandée. Un accès basé sur le principal de service fournit au contrôleur de l’appareil une authentification adéquate pour charger des informations de branche. Pour plus d’informations, consultez la page [Créer un principal de service](../active-directory/develop/howto-create-service-principal-portal.md#register-an-application-with-azure-ad-and-create-a-service-principal). Bien que cette fonctionnalité ne fasse pas partie de l’offre Azure Virtual WAN, nous répertorions ci-dessous les étapes classiques à suivre pour configurer l’accès dans Azure. Suite à cela, les détails pertinents sont saisis dans le tableau de bord de gestion de périphérique.

* Créez une application Azure Active Directory pour votre contrôleur de périphérique local.
* Obtenir un ID d’application et une clé d’authentification
* Obtenir l’ID de locataire
* Affecter l’application au rôle « Contributeur »

###  <a name="upload-branch-device-information"></a><a name="branch"></a>Charger les informations d’appareil de branche

Il est recommandé de concevoir l’expérience utilisateur pour charger les informations de branche (site local) dans Azure. Vous pouvez utiliser les [API REST](/rest/api/virtualwan/vpnsites) pour VPNSite afin de créer les informations de site dans Virtual WAN. Vous pouvez fournir tous les appareils VPN/SDWAN de branche, ou sélectionner les personnalisations d’appareil adéquates.

### <a name="device-configuration-download-and-connectivity"></a><a name="device"></a>Téléchargement de la configuration de l’appareil et connectivité

Cette étape inclut le téléchargement de la configuration Azure et la configuration de la connectivité à partir de l’appareil de branche dans Azure Virtual WAN. Dans cette étape, un client qui n’utilise pas un fournisseur doit télécharger manuellement la configuration Azure et l’appliquer à son appareil VPN/SDWAN local. En tant que fournisseur, vous devez automatiser cette étape. Pour plus d’informations, consultez la page sur les [API REST](/rest/api/virtualwan/vpnsitesconfiguration/download) de téléchargement. Le contrôleur d’appareil peut appeler l’API REST « GetVpnConfiguration » pour télécharger la configuration Azure.

**Notes sur la configuration**

  * Si les réseaux virtuels Azure sont attachés au hub virtuel, ils apparaîtront en tant que ConnectedSubnets (sous-réseaux connectés).
  * La connectivité VPN utilise une configuration basée sur des routes et prend en charge les protocoles IKEv1 et IKEv2.

## <a name="device-configuration-file"></a><a name="devicefile"></a>Fichier de configuration d’appareil

Le fichier de configuration de périphérique contient les paramètres à utiliser lors de la configuration de votre périphérique VPN sur site. Lorsque vous affichez ce fichier, notez les informations suivantes :

* **vpnSiteConfiguration -** Cette section indique les détails de l’appareil configuré comme un site se connectant au réseau virtuel étendu. Cela inclut le nom et l’adresse IP publique de l’appareil de branche.
* **vpnSiteConnections -** Cette section fournit des informations sur les éléments suivants :

    * **Espace d’adressage** du réseau virtuel du/des hub(s).<br>Exemple :
 
        ```
        "AddressSpace":"10.1.0.0/24"
        ```
    * **Espace d’adressage** des réseaux virtuels qui sont connectés au hub.<br>Exemple :

         ```
        "ConnectedSubnets":["10.2.0.0/16","10.3.0.0/16"]
         ```
    * **Adresses IP** de la passerelle VPN virtuelle. Étant donné que la passerelle VPN a chaque connexion avec 2 tunnels en configuration actif-actif, vous verrez les deux adresses IP répertoriées dans ce fichier. Dans cet exemple, vous voyez « Instance0 » et « Instance1 » pour chaque site.<br>Exemple :

        ``` 
        "Instance0":"104.45.18.186"
        "Instance1":"104.45.13.195"
        ```
    * **Détails de configuration de connexion de passerelle VPN**, comme BGP, une clé prépartagée, etc. La clé PSK est la clé prépartagée automatiquement générée pour vous. Vous pouvez toujours modifier la connexion dans la page Vue d’ensemble pour une clé PSK personnalisée.
  
**Exemple de fichier de configuration d’appareil**

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

## <a name="connectivity-details"></a><a name="default"></a>Détails sur la connectivité

La configuration de votre périphérique VPN/SDWAN local ou SD-WAN doit correspondre aux algorithmes et paramètres suivants spécifiés dans la stratégie IPsec/IKE Azure, ou les contenir.

* Algorithme de chiffrement IKE
* Algorithme d’intégrité IKE
* Groupe DH
* Algorithme de chiffrement IPsec
* Algorithme d’intégrité IPsec
* Groupe PFS

### <a name="default-policies-for-ipsec-connectivity"></a><a name="default"></a>Stratégies par défaut pour la connectivité IPsec

[!INCLUDE [IPsec Default](../../includes/virtual-wan-ipsec-include.md)]

### <a name="custom-policies-for-ipsec-connectivity"></a><a name="custom"></a>Stratégies personnalisées pour la connectivité IPsec

[!INCLUDE [IPsec Custom](../../includes/virtual-wan-ipsec-custom-include.md)]

## <a name="next-steps"></a>Étapes suivantes

Pour en plus sur le réseau WAN virtuel, consultez [À propos du réseau WAN virtuel Azure](virtual-wan-about.md) et la [FAQ sur le réseau WAN virtuel Azure](virtual-wan-faq.md).

Pour toute information complémentaire, envoyez un e-mail à l’adresse <azurevirtualwan@microsoft.com>. Ajoutez le nom de votre société entre crochets « [] » dans la ligne Objet.