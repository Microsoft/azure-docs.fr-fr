---
title: Fichier include
description: Fichier include
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 12/05/2019
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 6684c4269f14b91ded651dadff3f0a2e0878a4f2
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96025741"
---
### <a name="is-custom-ipsecike-policy-supported-on-all-azure-vpn-gateway-skus"></a>La stratégie personnalisée IPsec/IKE est-elle prise en charge sur toutes les références de passerelle VPN Azure ?
La stratégie IPsec/IKE personnalisée est prise en charge sur toutes les références SKU Azure, à l’exception de la référence SKU De base.

### <a name="how-many-policies-can-i-specify-on-a-connection"></a>Combien de stratégies puis-je spécifier pour une connexion ?
Vous pouvez uniquement spécifier ***une** _ combinaison de stratégies pour une connexion donnée.

### <a name="can-i-specify-a-partial-policy-on-a-connection-for-example-only-ike-algorithms-but-not-ipsec"></a>Puis-je spécifier une stratégie partielle pour une connexion ? (Par exemple, uniquement les algorithmes IKE, et non IPsec)
Non, vous devez spécifier tous les algorithmes et paramètres pour IKE (mode principal) et IPsec (mode rapide). Vous n’êtes pas en droit de spécifier de stratégie partielle.

### <a name="what-are-the-algorithms-and-key-strengths-supported-in-the-custom-policy"></a>Quels sont les algorithmes et les forces de clé pris en charge dans la stratégie personnalisée ?
Le tableau suivant répertorie les algorithmes de chiffrement et les puissances de clé pris en charge et configurables par les clients. Vous devez sélectionner une option pour chaque champ.

| _ *IPsec/IKEv2**  | **Options**                                                                   |
| ---              | ---                                                                           |
| Chiffrement IKEv2 | AES256, AES192, AES128, DES3, DES                                             |
| Intégrité IKEv2  | SHA384, SHA256, SHA1, MD5                                                     |
| Groupe DH         | DHGroup24, ECP384, ECP256, DHGroup14 (DHGroup2048), DHGroup2, DHGroup1, Aucun  |
| Chiffrement IPsec | GCMAES256, GCMAES192, GCMAES128, AES256, AES192, AES128, DES3, DES, Aucun      |
| Intégrité IPsec  | GCMAES256, GCMAES192, GCMAES128, SHA256, SHA1, MD5                            |
| Groupe PFS        | PFS24, ECP384, ECP256, PFS2048, PFS2, PFS1, Aucun                              |
| Durée de vie de l’AS en mode rapide   | Secondes (entier ; **min 300** /par défaut 27 000 secondes)<br>Kilo-octets (entier ; **min 1024** /par défaut 102 400 000 Ko)           |
| Sélecteur de trafic | UsePolicyBasedTrafficSelectors ($True/$False; default $False)                 |
|                  |                                                                               |

> [!IMPORTANT]
> 1. DHGroup2048 et PFS2048 sont identiques en tant que groupe Diffie-Hellman **14** dans IKE et IPsec PFS. Voir [Groupes Diffie-Hellman](#DH) pour accéder aux mappages complets.
> 2. Pour les algorithmes GCMAES, vous devez spécifier le même algorithme GCMAES et la longueur de clé pour le chiffrement IPsec et l’intégrité IPsec.
> 3. La durée de vie de l’AS en mode principal IKEv2 est fixée à 28 800 secondes pour les passerelles VPN Azure.
> 4. Les durées de vie de l’AS en mode rapide sont des paramètres facultatifs. Si rien n’a été spécifié, les valeurs par défaut de 27 000 secondes (7,5 heures) et de 102 400 000 kilo-octets (102 Go) sont utilisées.
> 5. UsePolicyBasedTrafficSelector est un paramètre d’option sur la connexion. Consultez le point suivant de la FAQ pour « UsePolicyBasedTrafficSelectors ».

### <a name="does-everything-need-to-match-between-the-azure-vpn-gateway-policy-and-my-on-premises-vpn-device-configurations"></a>Tous les éléments entre la stratégie de passerelle VPN Azure et mes configurations de périphérique VPN local doivent-ils correspondre ?
La configuration de votre périphérique VPN local doit correspondre aux algorithmes et paramètres suivants, spécifiés dans la stratégie IPsec/IKE Azure ou les contenir :

* Algorithme de chiffrement IKE
* Algorithme d’intégrité IKE
* Groupe DH
* Algorithme de chiffrement IPsec
* Algorithme d’intégrité IPsec
* Groupe PFS
* Sélecteur de trafic (*)

Les durées de vie de l’AS sont uniquement des spécifications locales, elles n’ont pas besoin de correspondre.

Si vous activez **UsePolicyBasedTrafficSelectors**, vous devez vous assurer que votre périphérique VPN dispose des sélecteurs de trafic correspondant définis avec toutes les combinaisons de préfixes (passerelle réseau locale) de votre réseau local vers et depuis les préfixes Azure, plutôt que de manière individuelle. Par exemple, si les préfixes de votre réseau local sont 10.1.0.0/16 et 10.2.0.0/16 et si les préfixes de votre réseau virtuel sont 192.168.0.0/16 et 172.16.0.0/16, vous devez spécifier les sélecteurs de trafic suivants :
* 10.1.0.0/16 <====> 192.168.0.0/16
* 10.1.0.0/16 <====> 172.16.0.0/16
* 10.2.0.0/16 <====> 192.168.0.0/16
* 10.2.0.0/16 <====> 172.16.0.0/16

Pour en savoir plus, consultez la section relative à la [connexion de plusieurs appareils VPN basés sur des stratégies locales](../articles/vpn-gateway/vpn-gateway-connect-multiple-policybased-rm-ps.md).

### <a name="which-diffie-hellman-groups-are-supported"></a><a name ="DH"></a>Quels groupes Diffie-Hellman sont pris en charge ?
Le tableau ci-dessous répertorie les groupes Diffie-Hellman pris en charge pour le protocole IKE (DHGroup) et IPsec (PFSGroup) :

| **Groupe Diffie-Hellman**  | **DHGroup**              | **PFSGroup** | **Longueur de clé** |
| ---                       | ---                      | ---          | ---            |
| 1                         | DHGroup1                 | PFS1         | MODP 768 bits   |
| 2                         | DHGroup2                 | PFS2         | MODP 1 024 bits  |
| 14                        | DHGroup14<br>DHGroup2048 | PFS2048      | MODP 2 048 bits  |
| 19                        | ECP256                   | ECP256       | ECP 256 bits    |
| 20                        | ECP384                   | ECP384       | ECP 384 bits    |
| 24                        | DHGroup24                | PFS24        | MODP 2 048 bits  |
|                           |                          |              |                |

Pour en savoir plus, voir [RFC3526](https://tools.ietf.org/html/rfc3526) et [RFC5114](https://tools.ietf.org/html/rfc5114).

### <a name="does-the-custom-policy-replace-the-default-ipsecike-policy-sets-for-azure-vpn-gateways"></a>La stratégie personnalisée remplace-t-elle les jeux de stratégie IPsec/IKE par défaut pour les passerelles VPN Azure ?
Oui, une fois qu’une stratégie personnalisée est spécifiée pour une connexion, la passerelle VPN Azure utilisera uniquement la stratégie pour la connexion, à la fois en tant qu’initiateur IKE et que répondeur IKE.

### <a name="if-i-remove-a-custom-ipsecike-policy-does-the-connection-become-unprotected"></a>Si je supprime une stratégie IPsec/IKE personnalisée, la connexion devient-elle non protégée ?
Non, la connexion restera protégée par IPsec/IKE. Dès que vous supprimez la stratégie personnalisée d’une connexion, la passerelle VPN Azure revient à la [liste par défaut de propositions IPsec/IKE](../articles/vpn-gateway/vpn-gateway-about-vpn-devices.md) et relance la négociation IKE avec votre appareil VPN local.

### <a name="would-adding-or-updating-an-ipsecike-policy-disrupt-my-vpn-connection"></a>L’ajout ou la mise à jour d’une stratégie IPsec/IKE risquent-ils de perturber ma connexion VPN ?
Oui, cela pourrait entraîner une courte interruption (de quelques secondes), puisque la passerelle VPN Azure va éliminer la connexion existante et relancer la négociation IKE pour rétablir le tunnel IPsec avec les nouveaux algorithmes de chiffrement et paramètres. Vérifiez que votre appareil VPN local est également configuré avec les algorithmes et les puissances de clé correspondants pour réduire l’interruption.

### <a name="can-i-use-different-policies-on-different-connections"></a>Puis-je utiliser des stratégies différentes pour des connexions distinctes ?
Oui. Une stratégie personnalisée est appliquée sur une base par connexion. Vous pouvez créer et appliquer des stratégies IPsec/IKE différentes pour des connexions distinctes. Vous pouvez également choisir d’appliquer des stratégies personnalisées pour un sous-ensemble de connexions. Les autres connexions utiliseront les jeux de stratégie IPsec/IKE par défaut d’Azure.

### <a name="can-i-use-the-custom-policy-on-vnet-to-vnet-connection-as-well"></a>Puis-je également utiliser la stratégie personnalisée pour une connexion entre des réseaux virtuels ?
Oui, vous pouvez appliquer la stratégie personnalisée pour des connexions IPsec entre différents sites locaux ou des connexions entre des réseaux virtuels.

### <a name="do-i-need-to-specify-the-same-policy-on-both-vnet-to-vnet-connection-resources"></a>Dois-je spécifier la même stratégie pour les ressources de connexion entre des réseaux virtuels ?
Oui. Un tunnel entre des réseaux virtuels se compose de deux ressources de connexion dans Azure, une pour chaque direction. Assurez-vous que les deux ressources de connexion ont la même stratégie, sinon la connexion de réseau virtuel à réseau virtuel ne pourra pas être établie.

### <a name="what-is-the-default-dpd-timeout-value-can-i-specify-a-different-dpd-timeout"></a>Quelle est la valeur du délai d’expiration DPD par défaut ? Est-ce que je peux spécifier un autre délai d’expiration DPD ?
Le délai d’expiration DPD par défaut est de 45 secondes. Vous pouvez spécifier une autre valeur de délai d’expiration DPD sur chaque connexion IPsec ou de réseau virtuel à réseau virtuel comprise entre 9 et 3 600 secondes.

### <a name="does-custom-ipsecike-policy-work-on-expressroute-connection"></a>La stratégie IPsec/IKE est-elle compatible avec une connexion ExpressRoute ?
Non. La stratégie IPsec/IKE est uniquement compatible avec les connexions S2S VPN et entre des réseaux virtuels via les passerelles VPN Azure.

### <a name="how-do-i-create-connections-with-ikev1-or-ikev2-protocol-type"></a>Comment créer des connexions avec le type de protocole IKEv1 ou IKEv2 ?
Les connexions IKEv1 peuvent être créées sur toutes les références SKU de type VPN RouteBased, à l’exception de la référence SKU De base, la référence SKU Standard et autres [références SKU héritées](../articles/vpn-gateway/vpn-gateway-about-skus-legacy.md#gwsku). Vous pouvez spécifier un type de protocole de connexion IKEv1 ou IKEv2 lors de la création de connexions. Si vous ne spécifiez pas de type de protocole de connexion, IKEv2 est utilisé comme option par défaut là où c’est applicable. Pour plus d’informations, consultez la documentation de l’[applet de commande PowerShell](/powershell/module/az.network/new-azvirtualnetworkgatewayconnection). Pour les types SKU et la prise en charge de IKEv1/IKEv2, consultez [Connecter des passerelles à des appareils VPN basés sur des stratégies](../articles/vpn-gateway/vpn-gateway-connect-multiple-policybased-rm-ps.md).

### <a name="is-transit-between-between-ikev1-and-ikev2-connections-allowed"></a>Le transit entre les connexions IKEv1 et IKEv2 est-il autorisé ?
Oui. Le transit entre les connexions IKEv1 et IKEv2 est pris en charge.

### <a name="can-i-have-ikev1-site-to-site-connections-on-basic-skus-of-routebased-vpn-type"></a>Puis-je avoir des connexions de site à site IKEv1 sur les références SKU de base de type VPN RouteBased ?
Non. La référence SKU de base ne prend pas cela en charge.

### <a name="can-i-change-the-connection-protocol-type-after-the-connection-is-created-ikev1-to-ikev2-and-vice-versa"></a>Puis-je changer le type de protocole de connexion une fois la connexion créée (IKEv1 en IKEv2 et vice versa) ?
Non. Une fois la connexion créée, les protocoles IKEv1/IKEv2 ne peuvent pas être changés. Vous devez supprimer et recréer une connexion avec le type de protocole souhaité.

### <a name="where-can-i-find-more-configuration-information-for-ipsec"></a>Où puis-je trouver des informations de configuration supplémentaires pour IPsec ?
Consultez [Configurer la stratégie IPsec/IKE pour des connexions S2S ou de réseau virtuel à réseau virtuel](../articles/vpn-gateway/vpn-gateway-ipsecikepolicy-rm-powershell.md)