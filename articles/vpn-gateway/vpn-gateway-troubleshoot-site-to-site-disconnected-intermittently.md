---
title: Détecter un problème de déconnexion intermittente du VPN site à site Azure
description: Découvrez comment résoudre les problèmes de déconnexion régulière du VPN de site à site.
services: vpn-gateway
titleSuffix: Azure VPN Gateway
author: chadmath
ms.service: vpn-gateway
ms.topic: troubleshooting
ms.date: 10/30/2018
ms.author: genli
ms.openlocfilehash: 82054099a5a496e99c49135ab98ee1163af19784
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75862558"
---
# <a name="troubleshooting-azure-site-to-site-vpn-disconnects-intermittently"></a>Résolution des problèmes : déconnexion intermittente du VPN de site à site Azure

Il se peut que vous rencontriez un problème d’instabilité ou de déconnexion régulière de la connexion VPN de site à site Microsoft Azure. Cet article fournit les étapes pour vous aider à identifier et résoudre la cause du problème. 

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="troubleshooting-steps"></a>Étapes de dépannage

### <a name="prerequisite-step"></a>Étape des conditions préalables

Vérifiez le type de passerelle de réseau virtuel Azure :

1. Accédez au [portail Azure](https://portal.azure.com).
2. Vérifiez les informations de type dans la page **Vue d’ensemble** de la passerelle de réseau virtuel.
    
    ![Vue d’ensemble de la passerelle](media/vpn-gateway-troubleshoot-site-to-site-disconnected-intermittently/gatewayoverview.png)

### <a name="step-1-check-whether-the-on-premises-vpn-device-is-validated"></a>Étape 1 : Vérifier si l’appareil VPN local est validé

1. Vérifiez si vous utilisez un [appareil VPN et une version de système d’exploitation validés](vpn-gateway-about-vpn-devices.md#devicetable). Si l’appareil VPN n’est pas validé, vous devez contacter son fabricant pour vérifier la compatibilité.
2. Assurez-vous que l’appareil VPN est correctement configuré. Pour plus d’informations, consultez [Modification des exemples de configuration de périphérique](vpn-gateway-about-vpn-devices.md#editing).

### <a name="step-2-check-the-security-association-settingsfor-policy-based-azure-virtual-network-gateways"></a>Étape 2 : Vérifier les paramètres d’association de sécurité (pour les passerelles de réseau virtuel Azure basées sur une stratégie)

1. Assurez-vous que le réseau virtuel, les sous-réseaux et les plages spécifiés dans la définition de **passerelle de réseau local** de Microsoft Azure sont identiques à la configuration sur l’appareil VPN local.
2. Vérifiez que les paramètres d’association de sécurité correspondent.

### <a name="step-3-check-for-user-defined-routes-or-network-security-groups-on-gateway-subnet"></a>Étape 3 : Vérifier les itinéraires définis par l’utilisateur ou les groupes de sécurité réseau sur le sous-réseau de passerelle

Un itinéraire défini par l’utilisateur sur le sous-réseau de passerelle risque de limiter une partie du trafic et d’autoriser le reste du trafic. C’est pourquoi la connexion VPN peut sembler manquer de fiabilité pour une partie du trafic et pas pour le reste du trafic. 

### <a name="step-4-check-the-one-vpn-tunnel-per-subnet-pair-setting-for-policy-based-virtual-network-gateways"></a>Étape 4 : Vérifier le paramètre « un tunnel VPN par paire de sous-réseau » (pour les passerelles de réseau virtuel basées sur une stratégie)

Assurez-vous que l’appareil VPN local est défini pour **qu’un tunnel VPN par paire de sous-réseau** soit configuré pour les passerelles de réseau virtuel basées sur une stratégie.

### <a name="step-5-check-for-security-association-limitation-for-policy-based-virtual-network-gateways"></a>Étape 5 : Vérifier la limite d’association de sécurité (pour les passerelles de réseau virtuel Azure basées sur une stratégie)

La passerelle de réseau virtuel basée sur une stratégie est limitée à 200 paires d’association de sécurité de sous-réseau. Si le nombre de sous-réseaux du réseau virtuel Azure multiplié par le nombre de sous-réseaux locaux est supérieur à 200, vous constaterez que les sous-réseaux se déconnectent sporadiquement.

### <a name="step-6-check-on-premises-vpn-device-external-interface-address"></a>Étape 6 : Vérifier l’adresse d’interface externe de l’appareil VPN local

- Si l’adresse IP accessible sur Internet de l’appareil VPN est incluse dans la définition de la **passerelle de réseau local** dans Azure, il se peut que vous subissiez des déconnexions occasionnelles.
- L’interface externe de l’appareil doit être directement liée à Internet. Il ne doit y avoir aucune traduction d’adresses réseau (NAT) ni aucun pare-feu entre Internet et l’appareil.
-  Si vous configurez le clustering de pare-feu dans le but d’obtenir une adresse IP virtuelle, vous devez détruire le cluster et exposer l’appliance VPN directement à une interface publique qui peut s’interfacer avec la passerelle.

### <a name="step-7-check-whether-the-on-premises-vpn-device-has-perfect-forward-secrecy-enabled"></a>Étape 7 : Vérifier l’activation de la fonctionnalité PFS (Perfect Forward Secrecy) sur l’appareil VPN local

La fonctionnalité **Perfect Forward Secrecy** peut provoquer les problèmes de déconnexion. Si la fonctionnalité **Perfect Forward Secrecy** de l’appareil VPN est activée, désactivez-la. [Mettez ensuite à jour la stratégie IPsec de la passerelle de réseau virtuel](vpn-gateway-ipsecikepolicy-rm-powershell.md#managepolicy).

## <a name="next-steps"></a>Étapes suivantes

- [Création d’une connexion de site à site dans le portail Azure](vpn-gateway-howto-site-to-site-resource-manager-portal.md)
- [Configure IPsec/IKE policy for S2S VPN or VNet-to-VNet connections](vpn-gateway-ipsecikepolicy-rm-powershell.md) (Configurer la stratégie IPsec/IKE pour les connexions VPN de site à site ou de réseau virtuel à réseau virtuel)

