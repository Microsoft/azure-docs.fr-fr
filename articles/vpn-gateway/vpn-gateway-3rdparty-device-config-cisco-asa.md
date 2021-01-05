---
title: Exemple de configuration pour la connexion d’appareils Cisco ASA à des passerelles VPN Azure
description: Cet article fournit un exemple de configuration pour la connexion de périphériques Cisco ASA à des passerelles VPN Azure.
services: vpn-gateway
author: yushwang
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/03/2020
ms.author: yushwang
ms.openlocfilehash: 82e00e0b519040c57fdfd798d8992086c61e5481
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89426168"
---
# <a name="sample-configuration-cisco-asa-device-ikev2no-bgp"></a>Exemple de configuration : appareil Cisco ASA (IKEv2/sans BGP)
Cet article fournit des exemples de configurations pour la connexion de périphériques Cisco ASA (Adaptive Security Appliance) à des passerelles VPN Azure. L’exemple s’applique à des périphériques Cisco ASA qui exécutent IKEv2 sans protocole de passerelle frontière (BGP). 

## <a name="device-at-a-glance"></a>Aperçu de l’appareil

* Fournisseur de l’appareil : **Cisco**
* Modèle de l’appareil : **ASA**           
* Version cible : **8.4 et versions ultérieures**
* Modèle testé : **ASA 5505**
* Version testée : **9.2**             
* Version IKE : **IKEv2**                  
* BGP : **Non**      
* Type de passerelle VPN Azure : **Passerelle VPN basée sur le routage**

> [!NOTE]
> L’exemple de configuration connecte un périphérique Cisco ASA à une passerelle VPN **basée sur le routage** Azure. Cette connexion utilise une stratégie IPsec/IKE personnalisée avec l’option **UsePolicyBasedTrafficSelectors**, comme décrit dans [cet article](vpn-gateway-connect-multiple-policybased-rm-ps.md).
>
> L’exemple exige que les périphériques ASA utilisent la stratégie **IKEv2** avec des configurations basées sur une liste d’accès, et non sur une interface virtuelle de tunnel (VTI). Consultez les spécifications du fournisseur de votre périphérique VPN pour vérifier que la stratégie IKEv2 est prise en charge sur vos périphériques VPN locaux.


## <a name="vpn-device-requirements"></a>Configuration requise du périphérique VPN
Les passerelles VPN Azure utilisent des suites de protocoles IPsec/IKE standard pour établir des tunnels VPN de site à site (S2S). Consultez [À propos des périphériques VPN](vpn-gateway-about-vpn-devices.md) pour connaître les paramètres détaillés du protocole IPsec/IKE et les algorithmes de chiffrement par défaut des passerelles VPN Azure.

> [!NOTE]
> Vous pouvez éventuellement spécifier une combinaison exacte d’algorithmes de chiffrement et de forces de clé souhaitée d’une connexion spécifique, comme décrit dans la rubrique [À propos des exigences de chiffrement](vpn-gateway-about-compliance-crypto.md). Si vous spécifiez une combinaison exacte d’algorithmes et de forces de clé, veillez à utiliser les spécifications correspondantes sur vos périphériques VPN.

## <a name="single-vpn-tunnel"></a>Tunnel VPN unique
Cette configuration présente un tunnel VPN S2S unique entre une passerelle VPN Azure et un périphérique VPN local. Vous pouvez éventuellement configurer le protocole BGP sur le tunnel VPN.

![Tunnel VPN S2S unique](./media/vpn-gateway-3rdparty-device-config-cisco-asa/singletunnel.png)

Pour connaître les instructions pas à pas permettant de créer les configurations Azure, consultez [Configuration de tunnel VPN unique](vpn-gateway-3rdparty-device-config-overview.md#singletunnel).

### <a name="virtual-network-and-vpn-gateway-information"></a>Informations sur le réseau virtuel et la passerelle VPN
Cette section répertorie les paramètres de cet exemple.

| **Paramètre**                | **Valeur**                    |
| ---                          | ---                          |
| Préfixes d’adresse de réseau virtuel        | 10.11.0.0/16<br>10.12.0.0/16 |
| IP de la passerelle VPN Azure         | Azure_Gateway_Public_IP      |
| Préfixes d’adresse locale | 10.51.0.0/16<br>10.52.0.0/16 |
| IP du périphérique VPN local    | OnPrem_Device_Public_IP     |
| * ASN BGP du réseau virtuel                | 65010                        |
| * IP d’homologue BGP Azure           | 10.12.255.30                 |
| * ASN BGP local         | 65050                        |
| * IP d’homologue BGP local     | 10.52.255.254                |
|                              |                              |

\* Paramètre facultatif pour BGP uniquement.

### <a name="ipsecike-policy-and-parameters"></a>Stratégie et paramètres IPsec/IKE
Le tableau suivant répertorie les algorithmes et paramètres IPsec/IKE utilisés dans l’exemple. Consultez les spécifications de votre périphérique VPN pour vérifier les algorithmes pris en charge pour vos modèles de périphérique VPN et vos versions de microprogramme.

| **IPsec/IKEv2**  | **Valeur**                            |
| ---              | ---                                  |
| Chiffrement IKEv2 | AES256                               |
| Intégrité IKEv2  | SHA384                               |
| Groupe DH         | DHGroup24                            |
| * Chiffrement IPsec | AES256                               |
| * Intégrité IPsec  | SHA1                                 |
| Groupe PFS        | PFS24                                |
| Durée de vie de l’AS en mode rapide   | 7 200 secondes                         |
| Sélecteur de trafic | UsePolicyBasedTrafficSelectors $True |
| Clé prépartagée   | PreSharedKey                         |
|                  |                                      |

\* Sur certains périphériques, l’intégrité IPsec doit être une valeur Null quand l’algorithme de chiffrement IPsec est AES-GCM.

### <a name="asa-device-support"></a>Prise en charge des périphériques ASA

* La prise en charge d’IKEv2 exige ASA versions 8.4 et ultérieures.

* La prise en charge des groupes Diffie-Hellman et PFS au-delà du groupe 5 nécessitent la version 9.x d’ASA.

* La prise en charge du chiffrement IPsec avec AES-GCM et l’intégrité IPsec avec SHA-256, SHA-384 ou SHA-512 exigent la version 9.x d’ASA. Cette exigence de prise en charge s’applique aux derniers périphériques ASA. Au moment de la publication, les modèles ASA 5505, 5510, 5520, 5540, 5550 et 5580 ne prennent pas en charge ces algorithmes. Consultez les spécifications de votre périphérique VPN pour vérifier les algorithmes pris en charge pour vos modèles de périphérique VPN et vos versions de microprogramme.


### <a name="sample-device-configuration"></a>Exemple de configuration de périphérique
Le script fournit un exemple qui se base sur la configuration et les paramètres décrits dans les sections précédentes. La configuration du tunnel VPN S2S comprend les éléments suivants :

1. Interfaces et itinéraires
2. Listes d’accès
3. Stratégie et paramètres IKE (phase 1 ou mode principal)
4. Stratégie et paramètres IPsec (phase 2 ou mode rapide)
5. Autres paramètres, tels que clamping du MSS TCP

> [!IMPORTANT]
> Effectuez les étapes suivantes avant d’utiliser l’exemple de script. Remplacez les valeurs d’espace réservé dans le script par les paramètres de périphérique adaptés à votre configuration.

* Spécifiez la configuration des interfaces internes et externes.
* Identifiez les itinéraires de vos réseaux internes/privés et externes/publics.
* Veillez à ce que tous les noms et les numéros de stratégie soient uniques sur le périphérique.
* Vérifiez que les algorithmes de chiffrement sont pris en charge sur votre périphérique.
* Remplacez les **valeurs d’espace réservé** suivantes par les valeurs réelles de votre configuration :
  - Nom de l’interface externe : **outside**
  - **Azure_Gateway_Public_IP**
  - **OnPrem_Device_Public_IP**
  - IKE : **Pre_Shared_Key**
  - Noms du réseau virtuel et de la passerelle de réseau local : **VNetName** et **LNGName**
  - **Préfixes** d’adresse du réseau virtuel et du réseau local
  - **Masques de réseau** appropriés

#### <a name="sample-script"></a>Exemple de script

```
! Sample ASA configuration for connecting to Azure VPN gateway
!
! Tested hardware: ASA 5505
! Tested version:  ASA version 9.2(4)
!
! Replace the following place holders with your actual values:
!   - Interface names - default are "outside" and "inside"
!   - <Azure_Gateway_Public_IP>
!   - <OnPrem_Device_Public_IP>
!   - <Pre_Shared_Key>
!   - <VNetName>*
!   - <LNGName>* ==> LocalNetworkGateway - the Azure resource that represents the
!     on-premises network, specifies network prefixes, device public IP, BGP info, etc.
!   - <PrivateIPAddress> ==> Replace it with a private IP address if applicable
!   - <Netmask> ==> Replace it with appropriate netmasks
!   - <Nexthop> ==> Replace it with the actual nexthop IP address
!
! (*) Must be unique names in the device configuration
!
! ==> Interface & route configurations
!
!     > <OnPrem_Device_Public_IP> address on the outside interface or vlan
!     > <PrivateIPAddress> on the inside interface or vlan; e.g., 10.51.0.1/24
!     > Route to connect to <Azure_Gateway_Public_IP> address
!
!     > Example:
!
!       interface Ethernet0/0
!        switchport access vlan 2
!       exit
!
!       interface vlan 1
!        nameif inside
!        security-level 100
!        ip address <PrivateIPAddress> <Netmask>
!       exit
!
!       interface vlan 2
!        nameif outside
!        security-level 0
!        ip address <OnPrem_Device_Public_IP> <Netmask>
!       exit
!
!       route outside 0.0.0.0 0.0.0.0 <NextHop IP> 1
!
! ==> Access lists
!
!     > Most firewall devices deny all traffic by default. Create access lists to
!       (1) Allow S2S VPN tunnels between the ASA and the Azure gateway public IP address
!       (2) Construct traffic selectors as part of IPsec policy or proposal
!
access-list outside_access_in extended permit ip host <Azure_Gateway_Public_IP> host <OnPrem_Device_Public_IP>
!
!     > Object group that consists of all VNet prefixes (e.g., 10.11.0.0/16 &
!       10.12.0.0/16)
!
object-group network Azure-<VNetName>
 description Azure virtual network <VNetName> prefixes
 network-object 10.11.0.0 255.255.0.0
 network-object 10.12.0.0 255.255.0.0
exit
!
!     > Object group that corresponding to the <LNGName> prefixes.
!       E.g., 10.51.0.0/16 and 10.52.0.0/16. Note that LNG = "local network gateway".
!       In Azure network resource, a local network gateway defines the on-premises
!       network properties (address prefixes, VPN device IP, BGP ASN, etc.)
!
object-group network <LNGName>
 description On-Premises network <LNGName> prefixes
 network-object 10.51.0.0 255.255.0.0
 network-object 10.52.0.0 255.255.0.0
exit
!
!     > Specify the access-list between the Azure VNet and your on-premises network.
!       This access list defines the IPsec SA traffic selectors.
!
access-list Azure-<VNetName>-acl extended permit ip object-group <LNGName> object-group Azure-<VNetName>
!
!     > No NAT required between the on-premises network and Azure VNet
!
nat (inside,outside) source static <LNGName> <LNGName> destination static Azure-<VNetName> Azure-<VNetName>
!
! ==> IKEv2 configuration
!
!     > General IKEv2 configuration - enable IKEv2 for VPN
!
group-policy DfltGrpPolicy attributes
 vpn-tunnel-protocol ikev1 ikev2
exit
!
crypto isakmp identity address
crypto ikev2 enable outside
!
!     > Define IKEv2 Phase 1/Main Mode policy
!       - Make sure the policy number is not used
!       - integrity and prf must be the same
!       - DH group 14 and above require ASA version 9.x.
!
crypto ikev2 policy 1
 encryption       aes-256
 integrity        sha384
 prf              sha384
 group            24
 lifetime seconds 86400
exit
!
!     > Set connection type and pre-shared key
!
tunnel-group <Azure_Gateway_Public_IP> type ipsec-l2l
tunnel-group <Azure_Gateway_Public_IP> ipsec-attributes
 ikev2 remote-authentication pre-shared-key <Pre_Shared_Key> 
 ikev2 local-authentication  pre-shared-key <Pre_Shared_Key> 
exit
!
! ==> IPsec configuration
!
!     > IKEv2 Phase 2/Quick Mode proposal
!       - AES-GCM and SHA-2 requires ASA version 9.x on newer ASA models. ASA
!         5505, 5510, 5520, 5540, 5550, 5580 are not supported.
!       - ESP integrity must be null if AES-GCM is configured as ESP encryption
!
crypto ipsec ikev2 ipsec-proposal AES-256
 protocol esp encryption aes-256
 protocol esp integrity  sha-1
exit
!
!     > Set access list & traffic selectors, PFS, IPsec proposal, SA lifetime
!       - This sample uses "Azure-<VNetName>-map" as the crypto map name
!       - ASA supports only one crypto map per interface, if you already have
!         an existing crypto map assigned to your outside interface, you must use
!         the same crypto map name, but with a different sequence number for
!         this policy
!       - "match address" policy uses the access-list "Azure-<VNetName>-acl" defined 
!         previously
!       - "ipsec-proposal" uses the proposal "AES-256" defined previously 
!       - PFS groups 14 and beyond requires ASA version 9.x.
!
crypto map Azure-<VNetName>-map 1 match address Azure-<VNetName>-acl
crypto map Azure-<VNetName>-map 1 set pfs group24
crypto map Azure-<VNetName>-map 1 set peer <Azure_Gateway_Public_IP>
crypto map Azure-<VNetName>-map 1 set ikev2 ipsec-proposal AES-256
crypto map Azure-<VNetName>-map 1 set security-association lifetime seconds 7200
crypto map Azure-<VNetName>-map interface outside
!
! ==> Set TCP MSS to 1350
!
sysopt connection tcpmss 1350
!
```

## <a name="simple-debugging-commands"></a>Commandes de débogage simples

À des fins de débogage, utilisez les commandes ASA suivantes :

* Affichez l’association de sécurité IPsec ou IKE :
    ```
    show crypto ipsec sa
    show crypto ikev2 sa
    ```

* Passez en mode de débogage :
    ```
    debug crypto ikev2 platform <level>
    debug crypto ikev2 protocol <level>
    ```
    Les commandes `debug` peuvent générer une sortie importante sur la console.

* Affichez les configurations actuelles sur le périphérique :
    ```
    show run
    ```
    Utilisez des sous-commandes `show` pour répertorier des parties spécifiques de la configuration du périphérique, par exemple :
    ```
    show run crypto
    show run access-list
    show run tunnel-group
    ```

## <a name="next-steps"></a>Étapes suivantes
Pour configurer des connexions entre différents locaux et entre deux réseaux virtuels en mode actif/actif, consultez [Configurer des passerelles VPN en mode actif/actif](vpn-gateway-activeactive-rm-powershell.md).
