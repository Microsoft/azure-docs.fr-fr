---
title: 'Passerelle VPN Azure : À propos des périphériques VPN pour les connexions'
description: Cet article traite des périphériques VPN et des paramètres IPsec pour les connexions entre locaux de passerelle VPN S2S. Des liens sont fournis vers des instructions et des exemples de configuration.
services: vpn-gateway
author: yushwang
ms.service: vpn-gateway
ms.topic: article
ms.date: 12/02/2020
ms.author: yushwang
ms.openlocfilehash: fdb7b1964bf4b8e2dbf6afd541e4415c0345347c
ms.sourcegitcommit: 8c3a656f82aa6f9c2792a27b02bbaa634786f42d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97631463"
---
# <a name="about-vpn-devices-and-ipsecike-parameters-for-site-to-site-vpn-gateway-connections"></a>À propos des périphériques VPN et des paramètres IPsec/IKE pour les connexions de passerelle VPN site à site

Un périphérique VPN est requis pour configurer une connexion VPN site à site (S2S) entre locaux à l’aide d’une passerelle VPN. Vous pouvez utiliser des connexions site à site pour créer une solution hybride, ou chaque fois que vous souhaitez disposer de connexions sécurisées entre vos réseaux locaux et vos réseaux virtuels. Cet article fournit une liste des périphériques VPN validés, ainsi qu’une liste des paramètres IPsec/IKE pour les passerelles VPN.

> [!IMPORTANT]
> Si vous rencontrez des problèmes de connectivité entre vos périphériques VPN locaux et les passerelles VPN, consultez [Problèmes de compatibilité connus avec le matériel](#known).
>

### <a name="items-to-note-when-viewing-the-tables"></a>Éléments à noter lorsque vous affichez les tables :

* Une modification de la terminologie a eu lieu pour les passerelles VPN Azure. Seuls les noms ont changé. Aucune modification de fonctionnalité n’est à noter.
  * Routage statique = basé sur des stratégies
  * Routage dynamique = basé sur un itinéraire
* Sauf indication contraire, les spécifications des passerelles VPN HighPerformance sont identiques à celles des passerelles VPN RouteBased. Par exemple, les périphériques VPN validés qui sont compatibles avec les passerelles VPN RouteBased sont également compatibles avec la passerelle VPN HighPerformance.

## <a name="validated-vpn-devices-and-device-configuration-guides"></a><a name="devicetable"></a>Périphériques VPN validés et guides de configuration des périphériques

Nous avons validé un ensemble de périphériques VPN standard en partenariat avec des fournisseurs d’appareils. Tous les appareils appartenant aux familles de la liste suivante doivent fonctionner avec les passerelles VPN. Consultez [À propos des paramètres de configuration de la passerelle VPN](vpn-gateway-about-vpn-gateway-settings.md#vpntype) pour comprendre le type de VPN utilisé (PolicyBased ou RouteBased) pour la solution de passerelle VPN que vous souhaitez configurer.

Pour configurer plus facilement votre périphérique VPN, reportez-vous aux liens qui correspondent à la famille de périphériques appropriée. Les liens vers les instructions de configuration sont fournis dans la mesure du possible. Pour une prise en charge des appareils VPN, contactez le fabricant de votre appareil.

|**Fournisseur**          |**Famille de périphériques**     |**Version de système d’exploitation minimale** |**Instructions de configuration PolicyBased** |**Instructions de configuration RouteBased** |
| ---                | ---                  | ---                   | ---            | ---           |
| A10 Networks, Inc. |Thunder CFW           |ACOS 4.1.1             |Non compatible  |[Guide de configuration](https://www.a10networks.com/wp-content/uploads/A10-DG-16161-EN.pdf)|
| Allied Telesis     |Routeurs VPN série AR |Série AR 5.4.7+               | [Guide de configuration](https://www.alliedtelesis.com/documents/how-to/configure/site-to-site-vpn-between-azure-and-ar-series-router) |[Guide de configuration](https://www.alliedtelesis.com/documents/how-to/configure/site-to-site-vpn-between-azure-and-ar-series-router)|
| Arista | Routeur CloudEOS | vEOS 4.24.0FX | (non testé) | [Guide de configuration](https://www.arista.com/en/cg-veos-router/veos-router-cloudeos-ipsec-connectivity-to-azure-virtual-network-gateway) |
| Barracuda Networks, Inc. |Barracuda CloudGen Firewall |PolicyBased : 5.4.3<br>RouteBased : 6.2.0 |[Guide de configuration](https://campus.barracuda.com/product/cloudgenfirewall/doc/79462887/how-to-configure-an-ikev1-ipsec-site-to-site-vpn-to-the-static-microsoft-azure-vpn-gateway/) |[Guide de configuration](https://campus.barracuda.com/product/cloudgenfirewall/doc/79462889/how-to-configure-bgp-over-ikev2-ipsec-site-to-site-vpn-to-an-azure-vpn-gateway/) |
| Check Point |Passerelle de sécurité |R80.10 |[Guide de configuration](https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk101275) |[Guide de configuration](https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk101275) |
| Cisco              |ASA       |8.3<br>8.4+ (IKEv2*) |Prise en charge |[Guide de configuration*](https://www.cisco.com/c/en/us/support/docs/security/adaptive-security-appliance-asa-software/214109-configure-asa-ipsec-vti-connection-to-az.html) |
| Cisco |ASR |PolicyBased : IOS 15.1<br>RouteBased : IOS 15.2 |Pris en charge |Pris en charge |
| Cisco | CSR | RouteBased : IOS-XE 16.10 | (non testé) | [Script de configuration](vpn-gateway-download-vpndevicescript.md) |
| Cisco |ISR |PolicyBased : IOS 15.0<br>RouteBased* : IOS 15.1 |Pris en charge |Pris en charge |
| Cisco |Meraki (MX) | MX v15.12 |Non compatible | [Guide de configuration](https://documentation.meraki.com/MX/Site-to-site_VPN/Configuring_Site_to_Site_VPN_tunnels_to_Azure_VPN_Gateway) |
| Cisco | vEdge (Viptela OS) | 18.4.0 (mode actif/passif)<br><br>19.2 (mode actif/passif) | Non compatible |  [Configuration manuelle (active/passive)](https://community.cisco.com/t5/networking-documents/how-to-configure-ipsec-vpn-connection-between-cisco-vedge-and/ta-p/3841454)<br><br>[Configuration Cloud Onramp (active/active)](https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/Network-Optimization-and-High-Availability/Network-Optimization-High-Availability-book/b_Network-Optimization-and-HA_chapter_00.html) |
| Citrix |NetScaler MPX, SDX, VPX |10.1 et versions ultérieures |[Guide de configuration](https://docs.citrix.com/en-us/netscaler/11-1/system/cloudbridge-connector-introduction/cloudbridge-connector-azure.html) |Non compatible |
| F5 |Série BIG-IP |12.0 |[Guide de configuration](https://devcentral.f5.com/articles/connecting-to-windows-azure-with-the-big-ip) |[Guide de configuration](https://devcentral.f5.com/articles/big-ip-to-azure-dynamic-ipsec-tunneling) |
| Fortinet |FortiGate |FortiOS 5.6 | (non testé) |[Guide de configuration](https://docs.fortinet.com/document/fortigate/5.6.0/cookbook/255100/ipsec-vpn-to-azure) |
| Hillstone Networks | Next-Generation Firewall (NGFW) | 5.5R7  | (non testé) | [Guide de configuration](https://www.hillstonenet.com/wp-content/uploads/How-to-setup-Site-to-Site-VPN-between-Microsoft-Azure-and-an-on-premise-Hillstone-Networks-Security-Gateway.pdf) |
| Internet Initiative Japan (IIJ) |Série SEIL |SEIL/X 4.60<br>SEIL/B1 4.60<br>SEIL/x86 3.20 |[Guide de configuration](https://www.iij.ad.jp/biz/seil/ConfigAzureSEILVPN.pdf) |Non compatible |
| Juniper |SRX |PolicyBased : JunOS 10.2<br>Routebased : JunOS 11.4 |Prise en charge |[Script de configuration](vpn-gateway-download-vpndevicescript.md) |
| Juniper |Série J |PolicyBased : JunOS 10.4r9<br>RouteBased : JunOS 11.4 |Prise en charge |[Script de configuration](vpn-gateway-download-vpndevicescript.md) |
| Juniper |ISG |ScreenOS 6.3 |Prise en charge |[Script de configuration](vpn-gateway-download-vpndevicescript.md) |
| Juniper |SSG |ScreenOS 6.2 |Prise en charge |[Script de configuration](vpn-gateway-download-vpndevicescript.md) |
| Juniper |MX |JunOS 12.x|Prise en charge |[Script de configuration](vpn-gateway-download-vpndevicescript.md) |
| Microsoft |Service de routage et d’accès à distance |Windows Server 2012 |Non compatible |Prise en charge |
| Open Systems AG |Passerelle Mission Control Security |N/A |[Guide de configuration](https://open-systems.com/wp-content/uploads/2019/12/OpenSystems-AzureVPNSetup-Installation-Guide.pdf) |Non compatible |
| Palo Alto Networks |Tous les périphériques exécutant PAN-OS |PAN-OS<br>PolicyBased : 6.1.5 ou version ultérieure<br>RouteBased : 7.1.4 |Prise en charge |[Guide de configuration](https://knowledgebase.paloaltonetworks.com/KCSArticleDetail?id=kA10g000000Cm6WCAS) |
| Sentrium (développeur) | VyOS | VyOS 1.2.2 | (non testé) | [Guide de configuration](https://docs.vyos.io/en/latest/configexamples/azure-vpn-bgp.html)|
| ShareTech | UTM nouvelle génération (série NU) | 9.0.1.3 | Non compatible | [Guide de configuration](http://www.sharetech.com.tw/images/file/Solution/NU_UTM/S2S_VPN_with_Azure_Route_Based_en.pdf) |
| SonicWall |Série TZ, série NSA<br>Série SuperMassive<br>Série NSA classe E |SonicOS 5.8.x<br>SonicOS 5.9.x<br>SonicOS 6.x |Non compatible |[Guide de configuration](https://www.sonicwall.com/support/knowledge-base/170505320011694) |
| Sophos | Pare-feu XG Next Gen | XG v17 | (non testé) | [Guide de configuration](https://community.sophos.com/kb/127546)<br><br>[Guide de configuration - Associations de sécurité multiples](https://community.sophos.com/kb/en-us/133154) |
| Synology | MR2200ac <br>RT2600ac <br>RT1900ac | SRM1.1.5/VpnPlusServer-1.2.0 | (non testé) | [Guide de configuration](https://www.synology.com/en-global/knowledgebase/SRM/tutorial/VPN/How_to_set_up_Site_to_Site_VPN_between_Synology_Router_and_MS_Azure) |
| Ubiquiti | EdgeRouter | Version 1.10 d’EdgeOS | (non testé) | [Protocole BGP sur IKEv2/IPsec](https://help.ubnt.com/hc/en-us/articles/115012374708)<br><br>[Protocole VTI sur IKEv2/IPsec](https://help.ubnt.com/hc/en-us/articles/115012305347) |
| Ultra | 3E-636L3 | 5.2.0.T3 Build-13  | (non testé) | [Guide de configuration](https://ultra-3eti.com/wp-content/uploads/2020/07/Azure-VPN-636L3-Site-to-Site-Test-Notes.pdf) |
| WatchGuard |Tous |Fireware XTM<br> PolicyBased: v11.11.x<br>RouteBased: v11.12.x |[Guide de configuration](http://watchguardsupport.force.com/publicKB?type=KBArticle&SFDCID=kA2F00000000LI7KAM&lang=en_US) |[Guide de configuration](http://watchguardsupport.force.com/publicKB?type=KBArticle&SFDCID=kA22A000000XZogSAG&lang=en_US)|
| Zyxel |Série ZyWALL USG<br>Série ZyWALL ATP<br>Série ZyWALL VPN | ZLD v4.32 + | (non testé) | [Protocole VTI sur IKEv2/IPsec](https://businessforum.zyxel.com/discussion/2648/)<br><br>[Protocole BGP sur IKEv2/IPsec](https://businessforum.zyxel.com/discussion/2650/)|

> [!NOTE]
>
> (*) Les versions Cisco ASA 8.4+ ajoutent la prise en charge IKEv2. Elles peuvent se connecter à la passerelle VPN Azure à l’aide de la stratégie IPsec/IKE personnalisée avec l’option « UsePolicyBasedTrafficSelectors ». Reportez-vous à cet [article sur les procédures](vpn-gateway-connect-multiple-policybased-rm-ps.md).
>
> (\*\*) Les routeurs de la série ISR 7200 prennent uniquement en charge les VPN basés sur des stratégies.

## <a name="download-vpn-device-configuration-scripts-from-azure"></a><a name="configscripts"></a>Télécharger des script de configuration de périphérique VPN à partir d’Azure

Pour certains appareils, vous pouvez télécharger les scripts de configuration directement à partir d’Azure. Pour plus d’informations et des instructions de téléchargement, consultez la page [Télécharger des script de configuration de périphérique VPN](vpn-gateway-download-vpndevicescript.md).

### <a name="devices-with-available-configuration-scripts"></a>Appareils avec des scripts de configuration disponibles

[!INCLUDE [scripts](../../includes/vpn-gateway-device-configuration-scripts.md)]

## <a name="non-validated-vpn-devices"></a><a name="additionaldevices"></a>Périphériques VPN non validés

Si votre appareil n’est pas répertorié dans le tableau des périphériques VPN validés, il peut tout de même fonctionner avec une connexion site à site. Contactez le fabricant de votre périphérique pour obtenir une prise en charge et des instructions de configuration supplémentaires.

## <a name="editing-device-configuration-samples"></a><a name="editing"></a>Modification des exemples de configuration de périphérique

Après avoir téléchargé l’exemple de configuration de périphérique VPN fourni, vous devrez remplacer certaines des valeurs spécifiées pour qu’elles reflètent les paramètres de votre environnement.

### <a name="to-edit-a-sample"></a>Pour modifier un exemple :

1. Ouvrez l’exemple à l’aide du Bloc-notes.
2. Recherchez et remplacez toutes les chaînes au format <*texte*> par les valeurs qui correspondent à votre environnement. Prenez soin d’inclure < et >. Lorsque vous sélectionnez un nom, assurez-vous qu’il est unique. Si une commande ne fonctionne pas, consultez la documentation du fabricant du périphérique.

| **Texte de l’exemple** | **Valeur de substitution** |
| --- | --- |
| &lt;RP_OnPremisesNetwork&gt; |Nom que vous choisissez pour cet objet. Exemple : MonRéseauLocal |
| &lt;RP_AzureNetwork&gt; |Nom que vous choisissez pour cet objet. Exemple : MonRéseauAzure |
| &lt;RP_AccessList&gt; |Nom que vous choisissez pour cet objet. Exemple : MaListeAccèsAzure |
| &lt;RP_IPSecTransformSet&gt; |Nom que vous choisissez pour cet objet. Exemple : MonJeuTransformationsIPSec |
| &lt;RP_IPSecCryptoMap&gt; |Nom que vous choisissez pour cet objet. Exemple : MaCarteChiffrementIPSec |
| &lt;SP_AzureNetworkIpRange&gt; |Spécifiez une plage. Exemple : 192.168.0.0 |
| &lt;SP_AzureNetworkSubnetMask&gt; |Spécifiez un masque de sous-réseau. Exemple : 255.255.0.0 |
| &lt;SP_OnPremisesNetworkIpRange&gt; |Spécifiez une plage locale. Exemple : 10.2.1.0 |
| &lt;SP_OnPremisesNetworkSubnetMask&gt; |Spécifiez un masque de sous-réseau local. Exemple : 255.255.255.0 |
| &lt;SP_AzureGatewayIpAddress&gt; |Ces informations sont propres à votre réseau virtuel et figurent dans le portail de gestion sous l’intitulé **Adresse IP de la passerelle**. |
| &lt;SP_PresharedKey&gt; |Ces informations sont propres à votre réseau virtuel et figurent dans le Portail de gestion sous l’intitulé Gérer la clé. |

## <a name="default-ipsecike-parameters"></a><a name="ipsec"></a>Paramètres IPsec/IKE par défaut

Les tableaux ci-dessous répertorient les combinaisons d’algorithmes et de paramètres utilisées par les passerelles VPN Azure dans la configuration par défaut (**stratégies par défaut**). Pour les passerelles VPN basées sur le routage créées à l’aide du modèle de déploiement Resource Manager, vous pouvez spécifier une stratégie personnalisée sur chaque connexion. Reportez-vous à [Configurer une stratégie IPsec/IKE](vpn-gateway-ipsecikepolicy-rm-powershell.md) pour obtenir des instructions détaillées.

En outre, vous devez fixer la **taille maximale de segment** du protocole TCP à **1350**. Dans le cas où vos appareils VPN ne prendraient pas en charge le réglage de la taille maximale de segment, vous pouvez à la place définir l’**unité de transmission maximale** dans l’interface de tunnel sur **1400** octets.

Dans les tableaux suivants :

* AS = association de sécurité
* IKE Phase 1 est également appelé « Mode principal »
* IKE Phase 2 est également appelé « Mode rapide »

### <a name="ike-phase-1-main-mode-parameters"></a>Paramètres IKE Phase 1 (Mode principal)

| **Propriété**          |**PolicyBased**    | **RouteBased**    |
| ---                   | ---               | ---               |
| Version IKE           |IKEv1              |IKEv1 et IKEv2    |
| Groupe Diffie-Hellman  |Groupe 2 (1 024 bits) |Groupe 2 (1 024 bits) |
| Méthode d'authentification |Clé prépartagée     |Clé prépartagée     |
| Chiffrement et algorithmes de hachage |1. AES256, SHA256<br>2. AES256, SHA1<br>3. AES128, SHA1<br>4. 3DES, SHA1 |1. AES256, SHA1<br>2. AES256, SHA256<br>3. AES128, SHA1<br>4. AES128, SHA256<br>5. 3DES, SHA1<br>6. 3DES, SHA256 |
| Durée de vie de l’AS           |28 800 secondes     |28 800 secondes     |

### <a name="ike-phase-2-quick-mode-parameters"></a>Paramètres IKE Phase 2 (Mode rapide)

| **Propriété**                  |**PolicyBased**| **RouteBased**                              |
| ---                           | ---           | ---                                         |
| Version IKE                   |IKEv1          |IKEv1 et IKEv2                              |
| Chiffrement et algorithmes de hachage |1. AES256, SHA256<br>2. AES256, SHA1<br>3. AES128, SHA1<br>4. 3DES, SHA1 |[Offres d’AS RouteBased en mode rapide](#RouteBasedOffers) |
| Durée de vie de l’AS (durée)            |3 600 secondes  |27 000 secondes                               |
| Durée de vie de l’AS (octets)           |102 400 000 Ko |102 400 000 Ko                               |
| PFS (Perfect Forward Secrecy) |Non             |[Offres d’AS RouteBased en mode rapide](#RouteBasedOffers) |
| Détection d’homologue mort     |Non pris en charge  |Prise en charge                                    |


### <a name="routebased-vpn-ipsec-security-association-ike-quick-mode-sa-offers"></a><a name ="RouteBasedOffers"></a>Offres d’association de sécurité VPN IPsec RouteBased (AS IKE en mode rapide)

Le tableau suivant répertorie les offres d’association de sécurité IPsec (IKE en mode rapide). Les offres sont énumérées dans l’ordre de préférence dans lequel elles sont présentées ou acceptées.

#### <a name="azure-gateway-as-initiator"></a>Passerelle Azure en tant qu’initiateur

|-  |**Chiffrement**|**Authentification**|**Groupe PFS**|
|---| ---          |---               |---          |
| 1 |GCM AES256    |GCM (AES256)      |None         |
| 2 |AES256        |SHA1              |None         |
| 3 |3DES          |SHA1              |None         |
| 4 |AES256        |SHA256            |None         |
| 5 |AES128        |SHA1              |None         |
| 6 |3DES          |SHA256            |None         |

#### <a name="azure-gateway-as-responder"></a>Passerelle Azure en tant que répondeur

|-  |**Chiffrement**|**Authentification**|**Groupe PFS**|
|---| ---          | ---              |---          |
| 1 |GCM AES256    |GCM (AES256)      |None         |
| 2 |AES256        |SHA1              |None         |
| 3 |3DES          |SHA1              |None         |
| 4 |AES256        |SHA256            |None         |
| 5 |AES128        |SHA1              |None         |
| 6 |3DES          |SHA256            |None         |
| 7 |DES           |SHA1              |None         |
| 8 |AES256        |SHA1              |1            |
| 9 |AES256        |SHA1              |2            |
| 10|AES256        |SHA1              |14           |
| 11|AES128        |SHA1              |1            |
| 12|AES128        |SHA1              |2            |
| 13|AES128        |SHA1              |14           |
| 14|3DES          |SHA1              |1            |
| 15|3DES          |SHA1              |2            |
| 16|3DES          |SHA256            |2            |
| 17|AES256        |SHA256            |1            |
| 18|AES256        |SHA256            |2            |
| 19|AES256        |SHA256            |14           |
| 20|AES256        |SHA1              |24           |
| 21|AES256        |SHA256            |24           |
| 22|AES128        |SHA256            |None         |
| 23|AES128        |SHA256            |1            |
| 24|AES128        |SHA256            |2            |
| 25|AES128        |SHA256            |14           |
| 26|3DES          |SHA1              |14           |

* Vous pouvez spécifier le chiffrement IPsec ESP NULL avec les passerelles VPN RouteBased et HighPerformance. Le chiffrement Null ne fournit pas de protection des données en transit. Il doit être utilisé uniquement lorsqu’un débit maximal et une latence minimale sont requis. Les clients peuvent choisir de l’utiliser dans les scénarios de communication entre les réseaux virtuels ou lorsque le chiffrement est appliqué ailleurs dans la solution.
* Pour les connexions entre locaux par le biais d’Internet, utilisez les paramètres de passerelle VPN Azure par défaut avec les algorithmes de chiffrement et de hachage répertoriés dans les tableaux ci-dessus pour garantir la sécurité de vos communications cruciales.

## <a name="known-device-compatibility-issues"></a><a name="known"></a>Problèmes de compatibilité connus avec le matériel

> [!IMPORTANT]
> Il s’agit de problèmes de compatibilité connus entre les appareils VPN tiers et les passerelles VPN Azure. L’équipe Azure travaille en étroite collaboration avec les fournisseurs pour résoudre les problèmes répertoriés ici. Une fois les problèmes résolus, cette page sera mise à jour avec les informations les plus récentes. Revenez la consulter régulièrement.
>
>

### <a name="feb-16-2017"></a>16 février 2017

**Appareils Palo Alto Networks dont la version est antérieure à la version 7.1.4** pour les VPN Azure basés sur les routes : Si vous utilisez des périphériques VPN de Palo Alto Networks avec une version de PAN-OS antérieure à la version 7.1.4 et si vous rencontrez des problèmes de connectivité pour les passerelles VPN Azure basées sur les routes, procédez comme suit :

1. Vérifiez la version du microprogramme de votre appareil Palo Alto Networks. Si votre version de PAN-OS est antérieure à la version 7.1.4, mettez à niveau vers la version 7.1.4.
2. Sur l’appareil Palo Alto Networks, modifiez la durée de SA de phase 2 (ou SA mode rapide) sur 28 800 secondes (8 heures) au moment de vous connecter à la passerelle VPN Azure.
3. Si vous continuez à rencontrer des problèmes de connectivité, ouvrez une demande de support sur le Portail Azure.
