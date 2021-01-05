---
title: Diagnostiquer la connectivité locale via une passerelle VPN
titleSuffix: Azure Network Watcher
description: Cet article décrit comment diagnostiquer la connectivité locale via une passerelle VPN avec la résolution des problèmes des ressources d’Azure Network Watcher.
services: network-watcher
documentationcenter: na
author: damendo
ms.assetid: aeffbf3d-fd19-4d61-831d-a7114f7534f9
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: damendo
ms.openlocfilehash: 9014f24918013872ce102d094f62fd5703594ddc
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94961853"
---
# <a name="diagnose-on-premises-connectivity-via-vpn-gateways"></a>Diagnostiquer la connectivité locale par le biais de passerelles VPN

La passerelle VPN Azure vous permet de créer des solutions hybrides qui répondent aux besoins d’une connexion sécurisée entre votre réseau local et votre réseau virtuel Azure. Vos besoins étant uniques, le choix du périphérique VPN local l’est également. Azure prend actuellement en charge [plusieurs périphériques VPN](../vpn-gateway/vpn-gateway-about-vpn-devices.md#devicetable) qui sont validés en permanence en partenariat avec les fournisseurs de périphériques. Passez en revue les paramètres de configuration spécifiques au périphérique avant de configurer votre périphérique VPN local. De même, la passerelle VPN Azure est configurée avec un ensemble de [paramètres IPsec pris en charge](../vpn-gateway/vpn-gateway-about-vpn-devices.md#ipsec) qui sont utilisés pour établir des connexions. Actuellement, il n’existe aucun moyen de spécifier ou de sélectionner une combinaison spécifique de paramètres IPsec à partir de la passerelle VPN Azure. Pour établir une connexion correcte entre le site et Azure, les paramètres du périphérique VPN local doivent être conformes aux paramètres IPsec prescrits par la passerelle VPN Azure. Si les paramètres sont incorrects, vous perdez la connectivité et, jusqu’à maintenant, la résolution de ces problèmes n’est pas simple et plusieurs heures sont généralement nécessaires pour identifier et corriger le problème.

Avec la fonctionnalité de résolution des problèmes d’Azure Network Watcher, vous êtes en mesure de diagnostiquer les problèmes en lien avec vos connexions et votre passerelle. En quelques minutes, vous avez suffisamment d’informations pour prendre une décision éclairée et corriger le problème.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="scenario"></a>Scénario

Vous pouvez configurer une connexion site à site entre Azure et votre site local en utilisant FortiGate comme passerelle VPN locale. Pour réaliser ce scénario, la configuration suivante est requise :

1. Passerelle de réseau virtuel - La passerelle VPN sur Azure
1. Passerelle de réseau local : représentation de la [passerelle VPN (FortiGate) locale](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md#LocalNetworkGateway) dans le cloud Azure
1. Connexion site à site (basée sur l’itinéraire) : [Connexion entre la passerelle VPN et le routeur local](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md#CreateConnection)
1. [Configuration de FortiGate](https://github.com/Azure/Azure-vpn-config-samples/blob/master/Fortinet/Current/Site-to-Site_VPN_using_FortiGate.md)

Vous trouverez des instructions étape par étape détaillées sur la configuration d’une connexion de site à site en consultant [Création d’un réseau virtuel avec une connexion de site à site à l’aide du portail Azure](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md).

L’une des étapes critiques de configuration consiste à configurer les paramètres de communication IPsec. Une mauvaise configuration entraîne une perte de connectivité entre le réseau local et Azure. Les passerelles VPN Azure sont actuellement configurées pour prendre en charge les paramètres IPsec suivants pour la phase 1. Comme vous pouvez le voir dans le tableau ci-dessous, les algorithmes de chiffrement pris en charge par la passerelle VPN Azure sont AES256 AES128 et 3DES.

### <a name="ike-phase-1-setup"></a>Configuration IKE phase 1

| **Propriété** | **PolicyBased** | **Basé sur un itinéraire et passerelle VPN standard ou hautes performances** |
| --- | --- | --- |
| Version IKE |IKEv1 |IKEv2 |
| Groupe Diffie-Hellman |Groupe 2 (1 024 bits) |Groupe 2 (1 024 bits) |
| Méthode d'authentification |Clé prépartagée |Clé prépartagée |
| Algorithmes de chiffrement |AES256 AES128 3DES |AES256 3DES |
| Algorithme de hachage |SHA1(SHA128) |SHA1(SHA128), SHA2(SHA256) |
| Durée de vie d’association de sécurité de phase 1 (temps) |28 800 secondes |28 800 secondes |

En tant qu’utilisateur, vous êtes obligé de configurer FortiGate. Vous trouverez un exemple de configuration sur [GitHub](https://github.com/Azure/Azure-vpn-config-samples/blob/master/Fortinet/Current/fortigate_show%20full-configuration.txt). Sans le savoir, vous avez configuré FortiGate pour qu’il utilise SHA-512 comme algorithme de hachage. Comme cet algorithme n’est pas un algorithme pris en charge pour les connexions basées sur une stratégie, votre connexion VPN ne fonctionne pas.

Ces problèmes sont difficiles à résoudre et les causes premières sont souvent complexes à déterminer. Dans ce cas, vous pouvez ouvrir un ticket de support pour obtenir de l’aide sur la résolution du problème. Toutefois, l’API de résolution des problèmes d’Azure Network Watcher vous permet d’identifier ces problèmes vous-même.

## <a name="troubleshooting-using-azure-network-watcher"></a>Résolution des problèmes à l’aide d’Azure Network Watcher

Pour diagnostiquer votre connexion, connectez-vous à Azure PowerShell et lancez l’applet de commande `Start-AzNetworkWatcherResourceTroubleshooting`. Vous trouverez des détails sur l’utilisation de cette applet de commande à la page [Troubleshoot Virtual Network Gateway and connections - PowerShell](network-watcher-troubleshoot-manage-powershell.md) (Résoudre les problèmes de connexions et de passerelle de réseau virtuel - PowerShell). L’exécution de cette applet de commande peut prendre plusieurs minutes.

Suite à son exécution, vous pouvez accéder à l’emplacement de stockage spécifié dans l’applet de commande pour obtenir des informations détaillées sur le problème et les journaux d’activité. Azure Network Watcher crée un dossier zip qui contient les fichiers journaux suivants :

![1][1]

Ouvrez le fichier appelé IKEErrors.txt. Celui-ci affiche l’erreur suivante, qui indique un problème de mauvaise configuration du paramètre IKE local.

```
Error: On-premises device rejected Quick Mode settings. Check values.
     based on log : Peer sent NO_PROPOSAL_CHOSEN notify
```

Vous pouvez obtenir des informations détaillées sur l’erreur à partir de Scrubbed-wfpdiag.txt. Ici, il est mentionné qu’une erreur `ERROR_IPSEC_IKE_POLICY_MATCH` s’est produite et a entraîné un dysfonctionnement de la connexion.

Un autre problème de configuration courant est la spécification de clés partagées incorrectes. Si dans l’exemple précédent, vous aviez spécifié différentes clés partagées, le fichier IKEErrors.txt affiche l’erreur suivante : `Error: Authentication failed. Check shared key`.

La fonctionnalité de résolution des problèmes d’Azure Network Watcher vous permet de diagnostiquer et de résoudre les problèmes liés à votre passerelle VPN et à votre connexion grâce à une simple applet de commande PowerShell. Actuellement, nous prenons en charge le diagnostic des états suivants et travaillons pour en prendre en charge davantage.

### <a name="gateway"></a>Passerelle

| Type d’erreur | Motif | Journal|
|---|---|---|
| NoFault | Quand aucune erreur n’est détectée. |Oui|
| GatewayNotFound | Passerelle introuvable ou non approvisionnée. |Non|
| PlannedMaintenance |  Instance de passerelle en maintenance.  |Non|
| UserDrivenUpdate | Quand une mise à jour utilisateur est en cours. Il peut s’agir d’une opération de redimensionnement. | Non |
| VipUnResponsive | Impossible d’atteindre l’instance principale de la passerelle. Cela se produit en cas d’échec de la sonde d’intégrité. | Non |
| PlatformInActive | Il existe un problème avec la plateforme. | Non|
| ServiceNotRunning | Le service sous-jacent ne fonctionne pas. | Non|
| NoConnectionsFoundForGateway | Aucune connexion n’existe sur la passerelle. Il s’agit simplement d’un avertissement.| Non|
| ConnectionsNotConnected | Aucune des connexions n’est connectée. Il s’agit simplement d’un avertissement.| Oui|
| GatewayCPUUsageExceeded | L’utilisation du processeur par la passerelle est actuellement supérieure à 95 %. | Oui |

### <a name="connection"></a>Connexion

| Type d’erreur | Motif | Journal|
|---|---|---|
| NoFault | Quand aucune erreur n’est détectée. |Oui|
| GatewayNotFound | Passerelle introuvable ou non approvisionnée. |Non|
| PlannedMaintenance | Instance de passerelle en maintenance.  |Non|
| UserDrivenUpdate | Quand une mise à jour utilisateur est en cours. Il peut s’agir d’une opération de redimensionnement.  | Non |
| VipUnResponsive | Impossible d’atteindre l’instance principale de la passerelle. Cela se produit en cas d’échec de la sonde d’intégrité. | Non |
| ConnectionEntityNotFound | La configuration de la connexion est manquante. | Non |
| ConnectionIsMarkedDisconnected | La connexion est identifiée comme étant « déconnectée ». |Non|
| ConnectionNotConfiguredOnGateway | Le service sous-jacent n’a pas la connexion configurée. | Oui |
| ConnectionMarkedStandby | Le service sous-jacent est identifié comme étant en veille.| Oui|
| Authentication | Non-concordance des clés prépartagées. | Oui|
| PeerReachability | La passerelle homologue n’est pas accessible. | Oui|
| IkePolicyMismatch | Les stratégies IKE de la passerelle homologue ne sont pas prises en charge par Azure. | Oui|
| WfpParse Error | Une erreur s’est produite lors de l’analyse du journal de protection des fichiers Windows. |Oui|

## <a name="next-steps"></a>Étapes suivantes

Apprenez à vérifier la connectivité de la passerelle VPN avec PowerShell et Azure Automation en consultant [Monitor VPN gateways with Network Watcher troubleshooting](network-watcher-monitor-with-azure-automation.md) (Surveiller les passerelles VPN avec la résolution des problèmes Network Watcher).

[1]: ./media/network-watcher-diagnose-on-premises-connectivity/figure1.png