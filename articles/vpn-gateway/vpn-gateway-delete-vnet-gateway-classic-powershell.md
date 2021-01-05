---
title: 'Supprimer une passerelle de réseau virtuel : Azure Classic'
description: Supprimez une passerelle de réseau virtuel avec PowerShell dans le modèle de déploiement classique.
titleSuffix: Azure VPN Gateway
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 10/08/2020
ms.author: cherylmc
ms.openlocfilehash: 39585a68c5cddc50cd04e82caca71209270f7b68
ms.sourcegitcommit: fbb620e0c47f49a8cf0a568ba704edefd0e30f81
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91874116"
---
# <a name="delete-a-virtual-network-gateway-using-powershell-classic"></a>Supprimer une passerelle de réseau virtuel avec PowerShell (classique)

> [!div class="op_single_selector"]
> * [Resource Manager - Portail Azure](vpn-gateway-delete-vnet-gateway-portal.md)
> * [Resource Manager - PowerShell](vpn-gateway-delete-vnet-gateway-powershell.md)
> * [Classic - PowerShell](vpn-gateway-delete-vnet-gateway-classic-powershell.md)
>

Cet article vous aide à supprimer une passerelle VPN dans le modèle de déploiement classique à l’aide de PowerShell. Une fois la passerelle de réseau virtuel supprimée, modifiez le fichier de configuration réseau pour supprimer les éléments que vous n’utilisez plus.

## <a name="step-1-connect-to-azure"></a><a name="connect"></a>Étape 1 : Connexion à Azure

### <a name="1-install-the-latest-powershell-cmdlets"></a>1. Installez les dernières applets de commande PowerShell.

[!INCLUDE [vpn-gateway-classic-powershell](../../includes/vpn-gateway-powershell-classic-locally.md)]

### <a name="2-connect-to-your-azure-account"></a>2. Connectez-vous à votre compte Azure.

Ouvrez la console PowerShell avec des droits élevés et connectez-vous à votre compte. Utilisez l’exemple suivant pour faciliter votre connexion :

1. Ouvrez la console PowerShell avec des droits élevés.
2. Se connecter à votre compte. Utilisez l’exemple suivant pour faciliter votre connexion :

   ```powershell
   Add-AzureAccount
   ```

## <a name="step-2-export-and-view-the-network-configuration-file"></a><a name="export"></a>Étape 2 : Exportez et affichez le fichier de configuration réseau

Créez un répertoire sur votre ordinateur, puis exportez le fichier de configuration réseau dans ce répertoire. Vous utilisez ce fichier aussi bien pour afficher les informations de configuration actuelles que pour modifier la configuration réseau.

Dans cet exemple, le fichier de configuration réseau est exporté vers C:\AzureNet.

```powershell
Get-AzureVNetConfig -ExportToFile C:\AzureNet\NetworkConfig.xml
```

Dans un éditeur de texte, ouvrez le fichier, puis affichez le nom de votre réseau virtuel classique. Quand vous créez un réseau virtuel dans le portail Azure, le nom complet utilisé par Azure n’est pas visible dans le portail. Par exemple, un réseau virtuel qui figure sous le nom « ClassicVNet1 » dans le portail Azure peut avoir un nom beaucoup plus long dans le fichier de configuration réseau. Le nom peut ressembler à ceci : « Group ClassicRG1 ClassicVNet1 ». Les noms des réseaux virtuels sont répertoriés sous la forme **'VirtualNetworkSite name ='** . Utilisez les noms indiqués dans le fichier de configuration réseau lors de l’exécution de vos applets de commande PowerShell.

## <a name="step-3-delete-the-virtual-network-gateway"></a><a name="delete"></a>Étape 3 : Supprimez la passerelle de réseau virtuel

Lorsque vous supprimez une passerelle de réseau virtuel, toutes les connexions au réseau virtuel via la passerelle sont déconnectées. Si vous disposez de clients P2S connectés au réseau virtuel, ils sont déconnectés sans avertissement.

Cet exemple supprime la passerelle de réseau virtuel. Vérifiez que vous utilisez le nom complet du réseau virtuel indiqué dans le fichier de configuration réseau.

```powershell
Remove-AzureVNetGateway -VNetName "Group ClassicRG1 ClassicVNet1"
```

Si l’opération réussit, la valeur de retour s’affiche :

```
Status : Successful
```

## <a name="step-4-modify-the-network-configuration-file"></a><a name="modify"></a>Étape 4 : Modifiez le fichier de configuration réseau

Lorsque vous supprimez une passerelle de réseau virtuel, l’applet de commande ne modifie pas le fichier de configuration réseau. Vous devez le modifier pour supprimer les éléments qui ne sont plus utilisés. Les sections suivantes vous aident à modifier le fichier de configuration réseau que vous avez téléchargé.

### <a name="local-network-site-references"></a><a name="lnsref"></a>Informations de référence de site de réseau local

Pour supprimer les informations de référence de site, apportez des modifications de configuration à **ConnectionsToLocalNetwork/LocalNetworkSiteRef**. La suppression d’une référence de site local déclenche la suppression d’un tunnel par Azure. En fonction de la configuration que vous avez créée, il se peut qu’une **LocalNetworkSiteRef** ne soit pas répertoriée.

```
<Gateway>
   <ConnectionsToLocalNetwork>
     <LocalNetworkSiteRef name="D1BFC9CB_Site2">
       <Connection type="IPsec" />
     </LocalNetworkSiteRef>
   </ConnectionsToLocalNetwork>
 </Gateway>
```

Exemple :

```
<Gateway>
   <ConnectionsToLocalNetwork>
   </ConnectionsToLocalNetwork>
 </Gateway>
```

### <a name="local-network-sites"></a><a name="lns"></a>Sites de réseau local

Supprimez les sites locaux que vous n’utilisez plus. Selon la configuration que vous avez créée, il est possible qu’un **LocalNetworkSite** ne soit pas répertorié.

```
<LocalNetworkSites>
  <LocalNetworkSite name="Site1">
    <AddressSpace>
      <AddressPrefix>192.168.0.0/16</AddressPrefix>
    </AddressSpace>
    <VPNGatewayAddress>5.4.3.2</VPNGatewayAddress>
  </LocalNetworkSite>
  <LocalNetworkSite name="Site3">
    <AddressSpace>
      <AddressPrefix>192.168.0.0/16</AddressPrefix>
    </AddressSpace>
    <VPNGatewayAddress>57.179.18.164</VPNGatewayAddress>
  </LocalNetworkSite>
 </LocalNetworkSites>
```

Dans cet exemple, nous avons supprimé uniquement Site3.

```
<LocalNetworkSites>
  <LocalNetworkSite name="Site1">
    <AddressSpace>
      <AddressPrefix>192.168.0.0/16</AddressPrefix>
    </AddressSpace>
    <VPNGatewayAddress>5.4.3.2</VPNGatewayAddress>
  </LocalNetworkSite>
 </LocalNetworkSites>
```

### <a name="client-addresspool"></a><a name="clientaddresss"></a>Pool d’adresses de client

Si vous aviez une connexion P2S à votre réseau virtuel, vous avez un **VPNClientAddressPool**. Supprimez les pools d’adresses de client qui correspondent à la passerelle de réseau virtuel que vous avez supprimée.

```
<Gateway>
    <VPNClientAddressPool>
      <AddressPrefix>10.1.0.0/24</AddressPrefix>
    </VPNClientAddressPool>
  <ConnectionsToLocalNetwork />
 </Gateway>
```

Exemple :

```
<Gateway>
  <ConnectionsToLocalNetwork />
 </Gateway>
```

### <a name="gatewaysubnet"></a><a name="gwsub"></a>Sous-réseau de passerelle

Supprimez le **GatewaySubnet** qui correspond au réseau virtuel.

```
<Subnets>
   <Subnet name="FrontEnd">
     <AddressPrefix>10.11.0.0/24</AddressPrefix>
   </Subnet>
   <Subnet name="GatewaySubnet">
     <AddressPrefix>10.11.1.0/29</AddressPrefix>
   </Subnet>
 </Subnets>
```

Exemple :

```
<Subnets>
   <Subnet name="FrontEnd">
     <AddressPrefix>10.11.0.0/24</AddressPrefix>
   </Subnet>
 </Subnets>
```

## <a name="step-5-upload-the-network-configuration-file"></a><a name="upload"></a>Étape 5 : Chargez le fichier de configuration réseau

Enregistrez vos modifications et chargez le fichier de configuration réseau dans Azure. Assurez-vous que vous modifiez le chemin d'accès selon les besoins de votre environnement.

```powershell
Set-AzureVNetConfig -ConfigurationPath C:\AzureNet\NetworkConfig.xml
```

Si l’opération réussit, la valeur de retour affiche un contenu semblable à cet exemple :

```
OperationDescription        OperationId                      OperationStatus                                                
--------------------        -----------                      ---------------                                           
Set-AzureVNetConfig         e0ee6e66-9167-cfa7-a746-7casb9   Succeeded
```
