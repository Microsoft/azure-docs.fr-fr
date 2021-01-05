---
title: 'Résoudre les problèmes de connexion VPN de site à site Azure : Passerelle VPN Azure'
description: Découvrez comment résoudre un problème de connexion VPN de site à site qui cesse soudainement de fonctionner sans possibilité de reconnexion.
services: vpn-gateway
author: chadmath
ms.service: vpn-gateway
ms.topic: troubleshooting
ms.date: 09/16/2019
ms.author: genli
ms.openlocfilehash: 674b8ab8266921a4eef8bbf212f3c556b1e587d7
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94658347"
---
# <a name="troubleshooting-an-azure-site-to-site-vpn-connection-cannot-connect-and-stops-working"></a>Résolution des problèmes : la connexion VPN site à site Azure cesse de fonctionner

Après avoir configuré une connexion VPN de site à site entre un réseau local et un réseau virtuel Azure, la connexion VPN cesse soudainement de fonctionner et la reconnexion est impossible. Cet article fournit les étapes requises pour vous aider à résoudre ce problème. 

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="troubleshooting-steps"></a>Étapes de dépannage

Pour résoudre le problème, essayez d’abord de [réinitialiser la passerelle VPN Azure](./reset-gateway.md) et de réinitialiser le tunnel à partir du périphérique VPN local. Si le problème persiste, procédez comme suit pour en identifier la cause.

### <a name="prerequisite-step"></a>Étape des conditions préalables

Vérifiez le type de passerelle VPN Azure utilisée.

1. Accédez au [portail Azure](https://portal.azure.com).

2. Vérifiez les informations de type dans la page **Vue d’ensemble** de la passerelle VPN.
    
    ![Vue d’ensemble de la passerelle](media/vpn-gateway-troubleshoot-site-to-site-cannot-connect/gatewayoverview.png)

### <a name="step-1-check-whether-the-on-premises-vpn-device-is-validated"></a>Étape 1. Vérifier si le périphérique VPN local est validé

1. Vérifiez si vous utilisez un [appareil VPN et une version de système d’exploitation validés](vpn-gateway-about-vpn-devices.md#devicetable). Si l’appareil n’est pas un périphérique VPN validé, vous devez contacter son fabricant pour en vérifier la compatibilité.

2. Assurez-vous que l’appareil VPN est correctement configuré. Pour plus d’informations, consultez la page [Modification des exemples de configuration de périphérique](vpn-gateway-about-vpn-devices.md#editing).

### <a name="step-2-verify-the-shared-key"></a>Étape 2. Vérifier la clé partagée

Comparez la clé partagée du périphérique VPN local et celle du VPN de réseau virtuel pour vous assurer que les clés correspondent. 

Pour afficher la clé partagée dans l’optique de la connexion VPN Azure, utilisez l’une des méthodes suivantes :

**Azure portal**

1. Accédez à la connexion de site à site de passerelle VPN que vous avez créée.

2. Dans la section **Paramètres**, cliquez sur **Clé partagée**.
    
    ![Clé partagée](media/vpn-gateway-troubleshoot-site-to-site-cannot-connect/sharedkey.png)

**Azure PowerShell**

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Pour le modèle de déploiement Azure Resource Manager :

```azurepowershell
Get-AzVirtualNetworkGatewayConnectionSharedKey -Name <Connection name> -ResourceGroupName <Resource group name>
```

Pour le modèle de déploiement classique :

```azurepowershell
Get-AzureVNetGatewayKey -VNetName -LocalNetworkSiteName
```

### <a name="step-3-verify-the-vpn-peer-ips"></a>Étape 3. Vérifier les adresses IP d’homologue VPN

-   La définition de l’adresse IP dans l’objet **Passerelle de réseau local** dans Azure doit correspondre à l’adresse IP de l’appareil local.
-   La définition de l’adresse IP de la passerelle Azure qui est configurée sur l’appareil local doit correspondre à l’adresse IP de la passerelle Azure.

### <a name="step-4-check-udr-and-nsgs-on-the-gateway-subnet"></a>Étape 4. Vérifier les paramètres d’itinéraire défini par l’utilisateur et des groupes de sécurité réseau sur le sous-réseau de passerelle

Recherchez et supprimez l’itinéraire défini par l’utilisateur (UDR) ou les groupes de sécurité réseau (NSG) sur le sous-réseau de passerelle, puis testez le résultat. Si le problème est résolu, validez les paramètres de l’itinéraire défini par l’utilisateur ou des groupes de sécurité réseau appliqués.

### <a name="step-5-check-the-on-premises-vpn-device-external-interface-address"></a>Étape 5. Vérifier l’adresse d’interface externe du périphérique VPN local

- Si l’adresse IP du périphérique VPN, accessible sur Internet, est incluse dans la définition du **réseau local** dans Azure, il se peut que vous subissiez des déconnexions occasionnelles.
- L’interface externe de l’appareil doit être directement liée à Internet. Il ne doit y avoir aucune traduction d’adresses réseau (NAT) ni aucun pare-feu entre Internet et l’appareil.
- Pour configurer le clustering de pare-feu dans le but d’obtenir une adresse IP virtuelle, vous devez détruire le cluster et exposer l’appliance VPN directement à une interface publique susceptible de s’interfacer avec la passerelle.

### <a name="step-6-verify-that-the-subnets-match-exactly-azure-policy-based-gateways"></a>Étape 6. Vérifier la correspondance des sous-réseaux (passerelles Azure basées sur des stratégies)

-   Vérifiez que les espaces d’adressage du réseau virtuel correspondent exactement entre le réseau virtuel Azure et les définitions locales.
-   Vérifiez que les sous-réseaux correspondent exactement dans **la passerelle réseau locale** et dans les définitions locales pour le réseau local.

### <a name="step-7-verify-the-azure-gateway-health-probe"></a>Étape 7. Vérifier la sonde d’intégrité de la passerelle Azure

1. Ouvrez la sonde d’intégrité en accédant à l’URL suivante :

    `https://<YourVirtualNetworkGatewayIP>:8081/healthprobe`

2. Cliquez sur l’avertissement de certificat.
3. Si vous recevez une réponse, cela signifie que la passerelle VPN est considérée comme saine. Vous ne recevez pas de réponse, cela signifie que la passerelle n’est peut-être pas saine ou qu’un groupe de sécurité réseau sur le sous-réseau de passerelle pose problème. Voici un exemple de réponse :

    ```xml
    <?xml version="1.0"?>
    <string xmlns="http://schemas.microsoft.com/2003/10/Serialization/">Primary Instance: GatewayTenantWorker_IN_1 GatewayTenantVersion: 14.7.24.6</string>
    ```

### <a name="step-8-check-whether-the-on-premises-vpn-device-has-the-perfect-forward-secrecy-feature-enabled"></a>Étape 8 : Vérifier l’activation de la fonctionnalité PFS (Perfect Forward Secrecy) sur le périphérique VPN local

La fonctionnalité Perfect Forward Secrecy peut provoquer des problèmes de déconnexion. Si la fonctionnalité Perfect Forward Secrecy du périphérique VPN est activée, désactivez-la. Mettez ensuite à jour la stratégie IPsec de la passerelle de réseau virtuel.

## <a name="next-steps"></a>Étapes suivantes

-   [Création d’une connexion de site à site dans le portail Azure](vpn-gateway-howto-site-to-site-resource-manager-portal.md)
-   [Configurer la stratégie IPsec/IKE pour des connexions VPN S2S ou de réseau virtuel à réseau virtuel](vpn-gateway-ipsecikepolicy-rm-powershell.md)