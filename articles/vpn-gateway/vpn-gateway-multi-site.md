---
title: 'Connecter un réseau virtuel à plusieurs sites en utilisant une passerelle VPN : Classique'
description: Connectez plusieurs sites locaux à un réseau virtuel classique à l’aide d’une passerelle VPN.
services: vpn-gateway
titleSuffix: Azure VPN Gateway
author: yushwang
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/03/2020
ms.author: yushwang
ms.openlocfilehash: 168bb9e06c73ec27ec1304813023889c9549b8e6
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94660693"
---
# <a name="add-a-site-to-site-connection-to-a-vnet-with-an-existing-vpn-gateway-connection-classic"></a>Ajouter une connexion de site à site à un réseau virtuel avec une connexion de passerelle VPN existante (Classic)

[!INCLUDE [deployment models](../../includes/vpn-gateway-classic-deployment-model-include.md)]

> [!div class="op_single_selector"]
> * [Azure portal](vpn-gateway-howto-multi-site-to-site-resource-manager-portal.md)
> * [PowerShell (classique)](vpn-gateway-multi-site.md)
>
>

Cet article vous explique comment utiliser PowerShell pour ajouter des connexions de site à site (S2S) à une passerelle VPN qui dispose déjà d’une connexion. Pour ce type de connexion, on fait souvent référence à une configuration « multisite ». Les étapes de cet article s’appliquent aux réseaux virtuels créés à l’aide du modèle de déploiement Classic (également connu comme le modèle Gestion des services). Ces étapes ne s’appliquent pas aux configurations de connexion coexistante ExpressRoute/site à site.

### <a name="deployment-models-and-methods"></a>Outils et modèles de déploiement

[!INCLUDE [vpn-gateway-classic-rm](../../includes/vpn-gateway-classic-rm-include.md)]

Nous mettons à jour ce tableau à mesure que de nouveaux articles et des outils supplémentaires sont disponibles pour cette configuration. Quand un article est disponible, le lien vers cet article est ajouté à ce tableau.

[!INCLUDE [vpn-gateway-table-multi-site](../../includes/vpn-gateway-table-multisite-include.md)]

## <a name="about-connecting"></a>À propos de la connexion

Vous pouvez connecter plusieurs sites locaux à un même réseau virtuel. Cela est particulièrement intéressant pour la création de solutions de cloud hybrides. La création d’une connexion sur plusieurs sites à votre passerelle de réseau virtuel Azure ressemble à la création d’autres connexions de site à site. En fait, vous pouvez utiliser une passerelle VPN Azure existante tant que la passerelle est dynamique (basée sur un itinéraire).

Si une passerelle statique est déjà connectée à votre réseau virtuel, vous pouvez changer le type de passerelle en dynamique sans avoir à régénérer le réseau virtuel pour prendre en charge plusieurs sites. Dans ce cas, assurez-vous que votre passerelle VPN locale prend en charge les configurations VPN basées sur un itinéraire.

![Diagramme multisite](./media/vpn-gateway-multi-site/multisite.png "multisite")

## <a name="points-to-consider"></a>Éléments à prendre en considération

**Vous ne pouvez pas utiliser le portail pour apporter des modifications à ce réseau virtuel.** Vous devez apporter des modifications au fichier de configuration réseau au lieu d’utiliser le portail. Si vous apportez des modifications dans le portail, ces dernières affectent vos paramètres de référence multisite pour ce réseau virtuel.

Vous aurez le temps de vous familiariser avec le fichier de configuration réseau au terme de la procédure multisite. Toutefois, si plusieurs personnes travaillent sur la configuration de votre réseau, vous devez vous assurer que tout le monde a connaissance de cette limitation. Cela ne signifie pas que vous ne pouvez pas du tout utiliser le portail. Vous pouvez l’utiliser pour toutes les procédures, à l’exception des modifications de configuration de ce réseau virtuel spécifique.

## <a name="before-you-begin"></a>Avant de commencer

Avant de commencer la configuration, vérifiez que les conditions suivantes sont remplies :

* Matériel VPN compatible pour chaque emplacement local. Voir [À propos des périphériques VPN pour la connectivité au réseau virtuel](vpn-gateway-about-vpn-devices.md) pour vérifier si l’appareil que vous souhaitez utiliser est bien compatible.
* Une adresse IP IPv4 publique exposée en externe pour chaque périphérique VPN. L’adresse IP ne peut pas se trouver derrière un NAT. Cela est obligatoire.
* Un expert en configuration du matériel VPN. Vous devez avoir une bonne maîtrise de la façon de configurer votre périphérique VPN ou de collaborer avec quelqu’un disposant des connaissances nécessaires.
* Les plages d’adresses IP que vous souhaitez utiliser pour votre réseau virtuel (si vous n’en avez pas déjà créé).
* Les plages d’adresses IP pour chacun des sites du réseau local auxquels vous vous connecterez. Vous devez vous assurer que les plages d’adresses IP pour chacun des sites du réseau local auxquels vous souhaitez vous connecter ne se chevauchent ne pas. Dans le cas contraire, le portail ou l’API REST rejette la configuration en cours de chargement.<br>Par exemple, si vous avez deux sites de réseau local contenant la plage d’adresse IP 10.2.3.0/24 et que vous disposez d’un package avec une adresse de destination 10.2.3.3, Azure ne saura pas à quel site envoyer le package, car les plages d’adresses se chevauchent. Pour éviter les problèmes de routage, Azure ne vous permet pas de télécharger un fichier de configuration ayant des plages qui se chevauchent.

### <a name="working-with-azure-powershell"></a>Utilisation d’Azure PowerShell

[!INCLUDE [vpn-gateway-classic-powershell](../../includes/vpn-gateway-powershell-classic-locally.md)]

## <a name="1-create-a-site-to-site-vpn"></a>1. Créer un VPN de site à site
Si vous avez déjà un VPN de site à site avec une passerelle de routage dynamique, bravo ! Vous pouvez [Exporter les paramètres de configuration de réseau virtuel](#export). Sinon, procédez ainsi :

### <a name="if-you-already-have-a-site-to-site-virtual-network-but-it-has-a-static-policy-based-routing-gateway"></a>Si vous avez déjà un réseau virtuel de site à site, mais que sa passerelle de routage est statique :
1. Modifiez le type de passerelle en routage dynamique. Un VPN multisite requiert une passerelle de routage dynamique (ou basée sur un itinéraire). Pour modifier le type de passerelle, vous devez d’abord supprimer la passerelle existante, puis en créer une nouvelle.
2. Configurez votre nouvelle passerelle et créez votre tunnel VPN. Pour obtenir des instructions, consultez [Spécifier la référence et le type de VPN](vpn-gateway-howto-site-to-site-classic-portal.md#sku). Veillez à spécifier « Dynamique » comme type de routage.

### <a name="if-you-dont-have-a-site-to-site-virtual-network"></a>Si vous n’avez pas de réseau virtuel de site à site :
1. Créez votre réseau virtuel de site à site à l’aide des instructions suivantes : [Créer un réseau virtuel avec une connexion VPN de site à site](./vpn-gateway-howto-site-to-site-classic-portal.md).  
2. Configurez une passerelle de routage dynamique à l’aide des instructions suivantes : [Configurer une passerelle VPN](./vpn-gateway-howto-site-to-site-classic-portal.md). Veillez à sélectionner le **routage dynamique** pour le type de passerelle.

## <a name="2-export-the-network-configuration-file"></a><a name="export"></a>2. Exporter le fichier de configuration réseau

Ouvrez la console PowerShell avec des privilèges élevés. Pour passer au management des services, utilisez cette commande :

```powershell
azure config mode asm
```

Se connecter à votre compte. Utilisez l’exemple suivant pour faciliter votre connexion :

```powershell
Add-AzureAccount
```

Exportez votre fichier de configuration réseau Azure en exécutant la commande suivante. Vous pouvez modifier l’emplacement d’exportation du fichier si nécessaire.

```powershell
Get-AzureVNetConfig -ExportToFile C:\AzureNet\NetworkConfig.xml
```

## <a name="3-open-the-network-configuration-file"></a>3. Ouvrir le fichier de configuration réseau
Ouvrez le fichier de configuration réseau que vous avez téléchargé à l’étape précédente. Utilisez l’éditeur xml de votre choix. Le fichier doit se présenter ainsi :

```xml
<NetworkConfiguration xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/ServiceHosting/2011/07/NetworkConfiguration">
  <VirtualNetworkConfiguration>
    <LocalNetworkSites>
      <LocalNetworkSite name="Site1">
        <AddressSpace>
          <AddressPrefix>10.0.0.0/16</AddressPrefix>
          <AddressPrefix>10.1.0.0/16</AddressPrefix>
        </AddressSpace>
        <VPNGatewayAddress>131.2.3.4</VPNGatewayAddress>
      </LocalNetworkSite>
      <LocalNetworkSite name="Site2">
        <AddressSpace>
          <AddressPrefix>10.2.0.0/16</AddressPrefix>
          <AddressPrefix>10.3.0.0/16</AddressPrefix>
        </AddressSpace>
        <VPNGatewayAddress>131.4.5.6</VPNGatewayAddress>
      </LocalNetworkSite>
    </LocalNetworkSites>
    <VirtualNetworkSites>
      <VirtualNetworkSite name="VNet1" AffinityGroup="USWest">
        <AddressSpace>
          <AddressPrefix>10.20.0.0/16</AddressPrefix>
          <AddressPrefix>10.21.0.0/16</AddressPrefix>
        </AddressSpace>
        <Subnets>
          <Subnet name="FE">
            <AddressPrefix>10.20.0.0/24</AddressPrefix>
          </Subnet>
          <Subnet name="BE">
            <AddressPrefix>10.20.1.0/24</AddressPrefix>
          </Subnet>
          <Subnet name="GatewaySubnet">
            <AddressPrefix>10.20.2.0/29</AddressPrefix>
          </Subnet>
        </Subnets>
        <Gateway>
          <ConnectionsToLocalNetwork>
            <LocalNetworkSiteRef name="Site1">
              <Connection type="IPsec" />
            </LocalNetworkSiteRef>
          </ConnectionsToLocalNetwork>
        </Gateway>
      </VirtualNetworkSite>
    </VirtualNetworkSites>
  </VirtualNetworkConfiguration>
</NetworkConfiguration>
```

## <a name="4-add-multiple-site-references"></a>4. Ajouter plusieurs références de site
Lorsque vous ajoutez ou supprimez les informations de référence de site, des modifications de configuration sont apportées à ConnectionsToLocalNetwork/LocalNetworkSiteRef. À l’ajout d’une nouvelle référence de site local, Azure crée un nouveau tunnel. Dans l’exemple ci-dessous, la configuration du réseau correspond à une connexion de site unique. Enregistrez le fichier une fois que vous avez terminé d’apporter vos modifications.

```xml
  <Gateway>
    <ConnectionsToLocalNetwork>
      <LocalNetworkSiteRef name="Site1"><Connection type="IPsec" /></LocalNetworkSiteRef>
    </ConnectionsToLocalNetwork>
  </Gateway>
```

Pour ajouter des références de site supplémentaires (créer une configuration multisite), ajoutez simplement des lignes « LocalNetworkSiteRef », comme illustré dans l’exemple ci-dessous :

```xml
  <Gateway>
    <ConnectionsToLocalNetwork>
      <LocalNetworkSiteRef name="Site1"><Connection type="IPsec" /></LocalNetworkSiteRef>
      <LocalNetworkSiteRef name="Site2"><Connection type="IPsec" /></LocalNetworkSiteRef>
    </ConnectionsToLocalNetwork>
  </Gateway>
```

## <a name="5-import-the-network-configuration-file"></a>5. Importer le fichier de configuration réseau
Importez le fichier de configuration réseau. Lorsque vous importez ce fichier avec les modifications, les nouveaux tunnels sont ajoutés. Les tunnels utilisent la passerelle dynamique que vous avez créée précédemment. Vous pouvez utiliser PowerShell pour importer le fichier.

## <a name="6-download-keys"></a>6. Télécharger les clés
Une fois les tunnels ajoutés, utilisez la cmdlet PowerShell Get-AzureVNetGatewayKey pour obtenir les clés prépartagées IPsec/IKE de chaque tunnel.

Par exemple :

```powershell
Get-AzureVNetGatewayKey –VNetName "VNet1" –LocalNetworkSiteName "Site1"
Get-AzureVNetGatewayKey –VNetName "VNet1" –LocalNetworkSiteName "Site2"
```

Si vous préférez, vous pouvez également utiliser l’API REST *Obtenir la clé partagée de la passerelle de réseau virtuel* pour obtenir les clés prépartagées.

## <a name="7-verify-your-connections"></a>7. Vérifiez vos connexions
Vérifiez l’état de tunnel multisite. Après avoir téléchargé les clés de chaque tunnel, vérifiez les connexions. Utilisez Get-AzureVnetConnection pour obtenir la liste des tunnels de réseau virtuel, comme indiqué dans l’exemple ci-dessous. VNet1 est le nom du réseau virtuel.

```powershell
Get-AzureVnetConnection -VNetName VNET1
```

L’exemple renvoie :

```
    ConnectivityState         : Connected
    EgressBytesTransferred    : 661530
    IngressBytesTransferred   : 519207
    LastConnectionEstablished : 5/2/2014 2:51:40 PM
    LastEventID               : 23401
    LastEventMessage          : The connectivity state for the local network site 'Site1' changed from Not Connected to Connected.
    LastEventTimeStamp        : 5/2/2014 2:51:40 PM
    LocalNetworkSiteName      : Site1
    OperationDescription      : Get-AzureVNetConnection
    OperationId               : 7f68a8e6-51e9-9db4-88c2-16b8067fed7f
    OperationStatus           : Succeeded

    ConnectivityState         : Connected
    EgressBytesTransferred    : 789398
    IngressBytesTransferred   : 143908
    LastConnectionEstablished : 5/2/2014 3:20:40 PM
    LastEventID               : 23401
    LastEventMessage          : The connectivity state for the local network site 'Site2' changed from Not Connected to Connected.
    LastEventTimeStamp        : 5/2/2014 2:51:40 PM
    LocalNetworkSiteName      : Site2
    OperationDescription      : Get-AzureVNetConnection
    OperationId               : 7893b329-51e9-9db4-88c2-16b8067fed7f
    OperationStatus           : Succeeded
```

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les passerelles VPN, consultez [À propos des passerelles VPN](vpn-gateway-about-vpngateways.md).