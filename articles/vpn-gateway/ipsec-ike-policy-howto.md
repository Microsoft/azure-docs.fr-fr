---
title: 'Stratégie IPsec/IKE pour des connexions VPN S2S ou de réseau virtuel à réseau virtuel : Portail Azure'
titleSuffix: Azure VPN Gateway
description: Configurez la stratégie IPSec/IKE pour des connexions VPN S2S ou de réseau virtuel à réseau virtuel avec des passerelles VPN Azure à l’aide d’Azure Resource Manager et du portail Azure.
services: vpn-gateway
author: yushwang
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/18/2020
ms.author: yushwang
ms.openlocfilehash: eda920640667abc6620c5c90ee7d04a44789353e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90992659"
---
# <a name="configure-ipsecike-policy-for-s2s-vpn-or-vnet-to-vnet-connections-azure-portal"></a>Configurer la stratégie IPsec/IKE pour des connexions VPN S2S ou de réseau virtuel à réseau virtuel : Portail Azure

Cet article vous guide dans les étapes de configuration de la stratégie IPsec/IKE pour les connexions de passerelle VPN site à site ou de réseau virtuel à réseau virtuel à l’aide du portail Azure. Les sections suivantes vous aident à créer et à configurer une stratégie IPsec/IKE, et à appliquer la stratégie à une connexion nouvelle ou existante.

## <a name="about-ipsec-and-ike-policy-parameters"></a><a name="about"></a>À propos des paramètres de stratégie IPsec et IKE

La norme de protocole IPsec et IKE standard prend en charge un vaste éventail d’algorithmes de chiffrement dans différentes combinaisons. Pour découvrir comment cela peut vous aider à vous assurer que la connectivité entre sites locaux et de réseau virtuel à réseau virtuel répond à vos besoins de conformité ou de sécurité, consultez [À propos des exigences de chiffrement et des passerelles VPN Azure](vpn-gateway-about-compliance-crypto.md).

Cet article fournit des instructions sur la création et la configuration d’une stratégie IPsec/IKE ainsi que sur son application à une connexion de passerelle VPN nouvelle ou existante.

### <a name="considerations"></a>Considérations

* La stratégie IPsec/IKE fonctionne uniquement sur les références (SKU) de passerelle suivantes :
  * ***VpnGw1~5 et VpnGw1AZ~5AZ***
  * ***Standard*** et ***HighPerformance***
* Vous pouvez uniquement spécifier ***une*** combinaison de stratégie pour une connexion donnée.
* Vous devez spécifier tous les algorithmes et paramètres pour IKE (mode principal) et IPsec (mode rapide). Vous n’êtes pas en droit de spécifier de stratégie partielle.
* Consulter les spécifications de votre fournisseur de périphérique VPN pour vous assurer que la stratégie est prise en charge sur vos périphériques VPN locaux. Des connexions S2S ou de réseau virtuel à réseau virtuel ne peuvent pas être établies si les stratégies ne sont pas compatibles.

## <a name="workflow"></a><a name ="workflow"></a>Flux de travail

Cette section décrit le flux de travail de la création et de la mise à jour d’une stratégie IPsec/IKE sur une connexion VPN S2S ou de réseau virtuel à réseau virtuel :

1. créer un réseau virtuel et une passerelle VPN ;
2. créer une passerelle de réseau local pour une connexion entre sites locaux, ou un autre réseau virtuel et une passerelle pour une connexion de réseau virtuel à réseau virtuel ;
3. créer une connexion (IPsec ou VNet2VNet) ;
4. configurer/mettre à jour/supprimer la stratégie IPsec/IKE sur les ressources de connexion.

Les instructions fournies dans cet article vous aideront à établir et à configurer des stratégies IPsec/IKE comme indiqué dans le diagramme :

:::image type="content" source="./media/ipsec-ike-policy-howto/policy-diagram.png" alt-text="Diagramme de stratégie IPsec/IKE" border="false":::

## <a name="supported-cryptographic-algorithms--key-strengths"></a><a name ="params"></a>Algorithmes de chiffrement pris en charge et avantages clés

### <a name="algorithms-and-keys"></a><a name ="table1"></a>Algorithmes et clés

Le tableau suivant répertorie les algorithmes de chiffrement et les forces de clé pris en charge et configurables par les clients :

| **IPsec/IKE**    | **Options**    |
| ---              | ---            |
| Chiffrement IKE   | AES256, AES192, AES128, DES3, DES                  |
| Intégrité IKE    | SHA384, SHA256, SHA1, MD5                          |
| Groupe DH         | DHGroup24, ECP384, ECP256, DHGroup14, DHGroup2048, DHGroup2, DHGroup1, Aucun |
| Chiffrement IPsec | GCMAES256, GCMAES192, GCMAES128, AES256, AES192, AES128, DES3, DES, Aucun    |
| Intégrité IPsec  | GCMASE256, GCMAES192, GCMAES128, SHA256, SHA1, MD5 |
| Groupe PFS        | PFS24, ECP384, ECP256, PFS2048, PFS2, PFS1, Aucun   |
| Durée de vie de l’AS en mode rapide   | (**Facultatif** : les valeurs par défaut sont utilisées si aucun valeur n’est indiquée)<br>Secondes (entier ; **min 300** /par défaut 27 000 secondes)<br>Kilo-octets (entier ; **min 1024** /par défaut 102 400 000 Ko)    |
| Sélecteur de trafic | UsePolicyBasedTrafficSelectors** ($True/$False ; **facultatif**, $False par défaut si aucune valeur n’est indiquée)    |
| Expiration DPD      | Secondes (entier ; min. 9/max. 3600 ; 45 secondes par défaut) |
|  |  |

#### <a name="important-requirements"></a>Exigences importantes

* La configuration de votre périphérique VPN local doit correspondre aux algorithmes et paramètres suivants, spécifiés dans la stratégie IPsec/IKE Azure ou les contenir :
  * Algorithme de chiffrement IKE (Mode principal / Phase 1)
  * Algorithme d’intégrité IKE (Mode principal / Phase 1)
  * Groupe DH (Mode principal / Phase 1)
  * Algorithme de chiffrement IPsec (Mode rapide / Phase 2)
  * Algorithme d’intégrité IPsec (Mode rapide / Phase 2)
  * Groupe PFS (Mode rapide/Phase 2) > * Sélecteur de trafic (si UsePolicyBasedTrafficSelectors est utilisé)
  * Les durées de vie de l’AS sont uniquement des spécifications locales, elles n’ont pas besoin de correspondre.

* Si GCMAES est utilisé pour l’algorithme de chiffrement IPsec, vous devez sélectionner le même algorithme GCMAES et la même longueur de clé pour l’intégrité IPsec. Par exemple, en utilisant GCMAES128 pour les deux.

* Dans le [tableau des algorithmes et des clés](#table1) ci-dessus :
  * IKEv2 correspond au Mode principal ou à la Phase 1.
  * IPsec correspond au Mode rapide ou à la Phase 2
  * Groupe DH spécifie le groupe Diffie-Hellmen utilisé dans le Mode principal ou à la Phase 1
  * Groupe PFS spécifie le groupe Diffie-Hellmen utilisé dans le Mode rapide ou à la Phase 2

* La durée de vie de l’association de sécurité en mode principal IKE est fixée à 28 800 secondes pour les passerelles VPN Azure.

* La définition du paramètre **UsePolicyBasedTrafficSelectors** sur $True sur une connexion a pour effet de configurer la passerelle VPN Azure pour se connecter à un pare-feu VPN basé sur une stratégie en local. Si vous activez UsePolicyBasedTrafficSelectors, vous devez vous assurer que votre périphérique VPN dispose des sélecteurs de trafic correspondant définis avec toutes les combinaisons de préfixes de réseau local (passerelle réseau locale) à destination et à partir des préfixes du réseau virtuel Azure, plutôt que de manière indifférenciée. Par exemple, si les préfixes de votre réseau local sont 10.1.0.0/16 et 10.2.0.0/16 et si les préfixes de votre réseau virtuel sont 192.168.0.0/16 et 172.16.0.0/16, vous devez spécifier les sélecteurs de trafic suivants :
  * 10.1.0.0/16 <====> 192.168.0.0/16
  * 10.1.0.0/16 <====> 172.16.0.0/16
  * 10.2.0.0/16 <====> 192.168.0.0/16
  * 10.2.0.0/16 <====> 172.16.0.0/16

   Pour plus d’informations sur les sélecteurs de trafic basés sur une stratégie, voir [Connecter plusieurs périphériques VPN basés sur une stratégie locale](vpn-gateway-connect-multiple-policybased-rm-ps.md).

* Expiration DPD – La valeur par défaut est de 45 secondes sur les passerelles VPN Azure. Le fait de définir le délai d’expiration sur des périodes plus courtes entraîne un renouvellement de clé IKE de manière plus agressive, qui a pour effet que la connexion apparaît déconnectée dans certains cas. Cela peut ne pas être souhaitable si vos emplacements locaux sont plus éloignés de la région Azure dans laquelle réside la passerelle VPN, ou si la condition de liaison physique risque d’entraîner une perte de paquet. La recommandation générale consiste à définir un délai d’expiration compris entre **30 et 45** secondes.

### <a name="diffie-hellman-groups"></a>Groupes Diffie-Hellman

Le tableau suivant répertorie les groupes Diffie-Hellman correspondants pris en charge par la stratégie personnalisée :

| **Groupe Diffie-Hellman**  | **DHGroup**              | **PFSGroup** | **Longueur de clé** |
| --- | --- | --- | --- |
| 1                         | DHGroup1                 | PFS1         | MODP 768 bits   |
| 2                         | DHGroup2                 | PFS2         | MODP 1 024 bits  |
| 14                        | DHGroup14<br>DHGroup2048 | PFS2048      | MODP 2 048 bits  |
| 19                        | ECP256                   | ECP256       | ECP 256 bits    |
| 20                        | ECP384                   | ECP384       | ECP 384 bits    |
| 24                        | DHGroup24                | PFS24        | MODP 2 048 bits  |

Reportez-vous à [RFC3526](https://tools.ietf.org/html/rfc3526) et [RFC5114](https://tools.ietf.org/html/rfc5114) pour plus d’informations.

## <a name="s2s-vpn-with-ipsecike-policy"></a><a name ="S2S"></a>VPN S2S avec stratégie IPsec/IKE

Cette section vous guide au fil des étapes de création d’une connexion VPN site à site avec une stratégie IPsec/IKE. Les étapes suivantes créent la connexion, comme illustré dans le diagramme suivant :

:::image type="content" source="./media/ipsec-ike-policy-howto/site-to-site-diagram.png" alt-text="Diagramme de stratégie IPsec/IKE" border="false":::

### <a name="step-1---create-the-virtual-network-vpn-gateway-and-local-network-gateway"></a><a name="createvnet1"></a>Étape 1 : création du réseau virtuel, de la passerelle VPN et de la passerelle de réseau local

Créez les ressources suivantes, comme dans les captures d’écran ci-dessous. Pour connaître les étapes à suivre, consultez [Créer une connexion VPN de site à site](vpn-gateway-howto-site-to-site-resource-manager-portal.md).

* **Réseau virtuel :**  TestVNet1

   :::image type="content" source="./media/ipsec-ike-policy-howto/testvnet-1.png" alt-text="Diagramme de stratégie IPsec/IKE":::

* **Passerelle VPN :** VNet1GW

   :::image type="content" source="./media/ipsec-ike-policy-howto/vnet-1-gateway.png" alt-text="Diagramme de stratégie IPsec/IKE":::

* **Passerelle de réseau local :** Site6

   :::image type="content" source="./media/ipsec-ike-policy-howto/lng-site-6.png" alt-text="Diagramme de stratégie IPsec/IKE":::

* **Connection :** VNet1 à Site6

    :::image type="content" source="./media/ipsec-ike-policy-howto/connection-site-6.png" alt-text="Diagramme de stratégie IPsec/IKE":::

### <a name="step-2---configure-ipsecike-policy-on-the-s2s-vpn-connection"></a><a name="s2sconnection"></a>Étape 2 : configuration de stratégie IPsec/IKE sur la connexion VPN S2S

Dans cette section, vous allez configurer une stratégie IPsec/IKE avec les algorithmes et paramètres suivants :

* IKE :   AES256, SHA384, DHGroup24, expiration DPD de 45 secondes
* IPsec : AES256, SHA256, pas de PFS, durée de vie de l’association de sécurité de 30000 secondes et 102 400 000 Ko

1. Accédez à la ressource de connexion, **VNet1toSite6**, dans le portail Azure. Sélectionnez la page **Configuration** et sélectionnez une stratégie IPsec/IKE **Personnalisée** pour afficher toutes les options de configuration. La capture d’écran ci-dessous montre la configuration en fonction de la liste :

    :::image type="content" source="./media/ipsec-ike-policy-howto/policy-site-6.png" alt-text="Diagramme de stratégie IPsec/IKE":::

1. Si vous utilisez des algorithmes GCMAES pour IPsec, vous devez utiliser les mêmes algorithme GCMAES et longueur de clé pour le chiffrement IPsec et l’intégrité IPsec. Par exemple, la capture d’écran ci-dessous spécifie GCMAES128 pour le chiffrement IPsec et l’intégrité IPsec :

   :::image type="content" source="./media/ipsec-ike-policy-howto/gcmaes.png" alt-text="Diagramme de stratégie IPsec/IKE":::

1. Vous pouvez éventuellement sélectionner **Activer** pour l’option **Utiliser des sélecteurs de trafic basés sur une stratégie** afin de permettre à la passerelle VPN Azure de se connecter à des VPN basés sur une stratégie localement, comme décrit ci-dessus.

   :::image type="content" source="./media/ipsec-ike-policy-howto/policy-based-selector.png" alt-text="Diagramme de stratégie IPsec/IKE":::

1. Une fois toutes les options sélectionnées, sélectionnez **Enregistrer** pour valider les modifications apportées à la ressource de connexion. La stratégie sera appliquée en environ une minute.

> [!IMPORTANT]
>
> * Une fois qu’une stratégie IPsec/IKE est spécifiée sur une connexion, la passerelle VPN Azure ne fait qu’envoyer ou accepter la proposition IPsec/IKE avec les algorithmes de chiffrement et puissances de clé spécifiés sur cette connexion particulière. Assurez-vous que votre périphérique VPN local pour la connexion utilise ou accepte la combinaison de stratégies exacte. À défaut, le tunnel VPN S2S ne peut pas être établi.
>
> * Les options **Sélecteur de trafic basé sur une stratégie** et **Délai d’expiration DPD** peuvent être spécifiées à l’aide d’une stratégie **Par défaut**, sans la stratégie IPsec/IKE personnalisée, comme indiqué dans la capture d’écran ci-dessus.
>

## <a name="vnet-to-vnet-with-ipsecike-policy"></a><a name ="vnet2vnet"></a>Connexion de réseau virtuel à réseau virtuel avec stratégie IPsec/IKE

Les étapes de création d’une connexion de réseau virtuel à réseau virtuel avec une stratégie IPsec/IKE sont similaires à celles d’une connexion VPN S2S.

:::image type="content" source="./media/ipsec-ike-policy-howto/vnet-policy.png" alt-text="Diagramme de stratégie IPsec/IKE" border="false":::

1. Pour créer votre connexion de réseau virtuel à réseau virtuel, procédez de la manière décrite dans [Créer une connexion de réseau virtuel à réseau virtuel](vpn-gateway-vnet-vnet-rm-ps.md).

2. Une fois les étapes accomplies, vous verrez deux connexions de réseau virtuel à réseau virtuel, comme dans la capture d’écran ci-dessous de la ressource VNet2GW :

   :::image type="content" source="./media/ipsec-ike-policy-howto/vnet-vnet-connections.png" alt-text="Diagramme de stratégie IPsec/IKE":::

3. Accédez à la ressource de connexion, puis à la page **Configuration** sur le portail. Sélectionnez **Personnalisée** dans **Stratégie IPsec/IKE** pour afficher les options de stratégie personnalisée. Sélectionnez les algorithmes de chiffrement avec les longueurs de clé correspondantes.

   La capture d’écran montre une stratégie IPsec/IKE différente avec les algorithmes et paramètres suivants :
   * IKE : AES128, SHA1, DHGroup14, délai d’expiration DPD de 45 secondes
   * IPsec : GCMAES128, GCMAES128, PFS14, SA Lifetime 14400 seconds & 102400000KB

   :::image type="content" source="./media/ipsec-ike-policy-howto/vnet-vnet-policy.png" alt-text="Diagramme de stratégie IPsec/IKE":::

4. Sélectionnez **Enregistrer** pour appliquer les modifications apportées à la stratégie à la ressource de connexion.

5. Appliquez la même stratégie à l’autre ressource de connexion, VNet2toVNet1. Si vous ne le faites pas, le tunnel VPN IPsec/IKE ne se connecte pas en raison d’une incompatibilité de stratégie.

   > [!IMPORTANT]
   > Une fois qu’une stratégie IPsec/IKE est spécifiée sur une connexion, la passerelle VPN Azure ne fait qu’envoyer ou accepter la proposition IPsec/IKE avec les algorithmes de chiffrement et puissances de clé spécifiés sur cette connexion particulière. Assurez-vous que les stratégies IPsec pour les deux connexions sont identiques. À défaut, la connexion de réseau virtuel à réseau virtuel ne peut pas être établie.

6. Une fois ces étapes accomplies, la connexion est établie en quelques minutes, et vous disposez de la topologie de réseau suivante :

    :::image type="content" source="./media/ipsec-ike-policy-howto/policy-diagram.png" alt-text="Diagramme de stratégie IPsec/IKE" border="false":::

## <a name="to-remove-custom-ipsecike-policy-from-a-connection"></a><a name ="deletepolicy"></a>Pour supprimer une stratégie IPsec/IKE personnalisée d’une connexion

1. Pour supprimer une stratégie personnalisée d’une connexion, accédez à la ressource de connexion, ainsi qu’à la page **Configuration** pour afficher la stratégie en cours.

2. Sélectionnez **Par défaut** sous l’option **Stratégie IPsec/IKE**. Cette opération supprime toutes les stratégies personnalisées précédemment spécifiées sur la connexion, et restaure les paramètres IPsec/IKE par défaut sur cette connexion :

   :::image type="content" source="./media/ipsec-ike-policy-howto/delete-policy.png" alt-text="Diagramme de stratégie IPsec/IKE":::

3. Sélectionnez **Enregistrer** pour supprimer la stratégie personnalisée et restaurer les paramètres IPsec/IKE par défaut sur la connexion.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les sélecteurs de trafic basés sur une stratégie, voir [Connecter plusieurs périphériques VPN basés sur un stratégie locale](vpn-gateway-connect-multiple-policybased-rm-ps.md).
