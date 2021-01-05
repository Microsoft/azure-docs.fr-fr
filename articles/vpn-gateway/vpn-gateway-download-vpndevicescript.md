---
title: Télécharger des scripts de configuration de périphérique VPN pour les connexions VPN S2S
description: Cet article vous guide dans le téléchargement de scripts de configuration de périphérique VPN pour les connexions VPN S2S avec des passerelles VPN Azure à l’aide d’Azure Resource Manager.
titleSuffix: Azure VPN Gateway
services: vpn-gateway
author: yushwang
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/02/2020
ms.author: yushwang
ms.openlocfilehash: 70d9a8069979a299ad3031de37c525438ab0159d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89394599"
---
# <a name="download-vpn-device-configuration-scripts-for-s2s-vpn-connections"></a>Télécharger des scripts de configuration de périphérique VPN pour les connexions VPN S2S

Cet article vous guide dans le téléchargement de scripts de configuration de périphérique VPN pour les connexions VPN S2S avec des passerelles VPN Azure à l’aide d’Azure Resource Manager. Le schéma qui suit présente le flux de travail général.

![télécharger-script](./media/vpn-gateway-download-vpndevicescript/downloaddevicescript.png)

Les appareils suivants ont des scripts disponibles :

[!INCLUDE [scripts](../../includes/vpn-gateway-device-configuration-scripts.md)]

## <a name="about-vpn-device-configuration-scripts"></a><a name="about"></a>À propos du téléchargement des scripts de configuration de périphérique VPN

Un réseau VPN intersite se compose d’une passerelle VPN Azure, d’un périphérique VPN local et d’un tunnel VPN IPsec S2S entre les deux. Le flux de travail type inclut les étapes suivantes :

1. Créer et configurer une passerelle VPN Azure (passerelle de réseau virtuel)
2. Créer et configurer une passerelle de réseau local Azure qui représente votre réseau local et le périphérique VPN
3. Créer et configurer une connexion VPN Azure entre la passerelle VPN Azure et la passerelle de réseau local
4. Configurer le périphérique VPN local représenté par la passerelle de réseau local pour établir le tunnel VPN S2S réel avec la passerelle VPN Azure

Vous pouvez effectuer les étapes 1 à 3 sur le [portail](vpn-gateway-howto-site-to-site-resource-manager-portal.md) Azure, [PowerShell](vpn-gateway-create-site-to-site-rm-powershell.md) ou la [CLI](vpn-gateway-howto-site-to-site-resource-manager-cli.md). La dernière étape consiste à configurer les périphériques VPN locaux en dehors d’Azure. Cette fonctionnalité vous permet de télécharger un script de configuration pour votre périphérique VPN avec les valeurs correspondantes de votre passerelle VPN Azure, du réseau virtuel et des préfixes d’adresse du réseau local, ainsi que des propriétés de connexion VPN, etc. déjà renseignées. Vous pouvez utiliser le script comme point de départ ou appliquer le script directement à vos périphériques VPN locaux au moyen de la console de configuration.

> [!IMPORTANT]
> * La syntaxe de chaque script de configuration de périphérique VPN est différente et repose sur les modèles et les versions du microprogramme. Faites particulièrement attention au modèle et aux informations de version de votre périphérique par rapport aux modèles disponibles.
> * Certaines valeurs de paramètre doivent être uniques sur le périphérique et ne peuvent pas être déterminées sans un accès au périphérique. Les scripts de configuration générés par Azure renseignent automatiquement ces valeurs, mais vous devez vérifier que les valeurs fournies sont valides sur votre périphérique. Exemples :
>    * Numéros d’interface
>    * Numéros d’ACL
>    * Noms ou numéros de stratégie, etc.
> * Recherchez le mot clé « **REPLACE** », incorporé dans le script pour retrouver les paramètres à vérifier avant d’appliquer le script.
> * Certains modèles comprennent une section « **CLEANUP** » (NETTOYAGE) que vous pouvez appliquer pour supprimer les configurations. Les sections de nettoyage sont commentées par défaut.

## <a name="download-the-configuration-script-from-azure-portal"></a>Télécharger le script de configuration sur le portail Azure

Créez une passerelle VPN Azure, une passerelle de réseau local et une ressource de connexion entre les deux. La page suivante va vous guider tout au long des étapes :

* [Créer une connexion de site à site dans le portail Azure](vpn-gateway-howto-site-to-site-resource-manager-portal.md)

Une fois que la ressource de connexion est créée, suivez les instructions ci-dessous pour télécharger les scripts de configuration de périphérique VPN :

1. Dans un navigateur, accédez au [portail Azure](https://portal.azure.com) et, si nécessaire, connectez-vous avec votre compte Azure
2. Accédez à la ressource de connexion que vous avez créée. Vous trouverez la liste de toutes les ressources de connexion en cliquant sur « Tous les services », puis « RÉSEAU » et « Connexions ».

    ![liste-connexions](./media/vpn-gateway-download-vpndevicescript/connectionlist.png)

3. Cliquez sur la connexion à configurer.

    ![aperçu-connexion](./media/vpn-gateway-download-vpndevicescript/connectionoverview.png)

4. Cliquez sur le lien « Télécharger la configuration » qui s’affiche en rouge dans la page Aperçu de la connexion afin d’ouvrir la page « Télécharger la configuration ».

    ![télécharger-script-1](./media/vpn-gateway-download-vpndevicescript/downloadscript-1.png)

5. Sélectionnez la famille du modèle et la version du microprogramme pour votre périphérique VPN, puis cliquez sur le bouton « Télécharger la configuration ».

    ![télécharger66-script-2](./media/vpn-gateway-download-vpndevicescript/downloadscript-2.PNG)

6. Vous êtes invité à enregistrer le script téléchargé (fichier texte) à partir de votre navigateur.
7. Une fois que vous avez téléchargé le script de configuration, ouvrez-le dans un éditeur de texte et recherchez le mot clé « REPLACE » pour identifier et examiner les paramètres à remplacer.

    ![modifier-script](./media/vpn-gateway-download-vpndevicescript/editscript.png)

## <a name="download-the-configuration-script-using-azure-powershell"></a>Télécharger le script de configuration à l’aide d’Azure PowerShell



Vous pouvez également télécharger le script de configuration à l’aide d’Azure PowerShell, comme indiqué dans l’exemple suivant :

```azurepowershell-interactive
$RG          = "TestRG1"
$GWName      = "VNet1GW"
$Connection  = "VNet1toSite1"

# List the available VPN device models and versions
Get-AzVirtualNetworkGatewaySupportedVpnDevice -Name $GWName -ResourceGroupName $RG

# Download the configuration script for the connection
Get-AzVirtualNetworkGatewayConnectionVpnDeviceConfigScript -Name $Connection -ResourceGroupName $RG -DeviceVendor Juniper -DeviceFamily Juniper_SRX_GA -FirmwareVersion Juniper_SRX_12.x_GA
```

## <a name="apply-the-configuration-script-to-your-vpn-device"></a>Appliquer le script de configuration à votre périphérique VPN

Une fois que vous avez téléchargé et validé le script de configuration, l’étape suivante consiste à appliquer le script à votre périphérique VPN. La procédure réelle varie en fonction de la marque et du modèle du périphérique VPN. Consultez les guides d’utilisation ou les pages d’instruction de vos périphériques VPN.

## <a name="next-steps"></a>Étapes suivantes

Poursuivez la configuration de votre [connexion de site à site](vpn-gateway-howto-site-to-site-resource-manager-portal.md).
