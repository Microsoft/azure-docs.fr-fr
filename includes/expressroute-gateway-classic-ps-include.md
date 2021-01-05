---
title: Fichier Include
description: Fichier Include
services: expressroute
author: cherylmc
ms.service: expressroute
ms.topic: include
ms.date: 12/13/2018
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: c0c05eb88df27ba9e71c0f61bad75457d893a7ce
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95561705"
---
> [!NOTE]
> Ces exemples ne s’appliquent pas aux configurations de coexistence S2S/ExpressRoute.
> Pour plus d’informations sur l’utilisation de passerelles dans une configuration de coexistence, consultez [Configurer la coexistence de connexions.](../articles/expressroute/expressroute-howto-coexist-classic.md#gw)

## <a name="add-a-gateway"></a>Ajout d’une passerelle

Lorsque vous ajoutez une passerelle à un réseau virtuel à l’aide du modèle de ressource classique, vous modifiez le fichier de configuration réseau directement avant de créer la passerelle. Les valeurs dans les exemples ci-dessous doivent être présentes dans le fichier pour créer une passerelle. Si votre réseau virtuel avait déjà une passerelle qui lui était associée, certaines de ces valeurs seront déjà présentes. Modifiez le fichier pour appliquer les valeurs ci-dessous.

### <a name="download-the-network-configuration-file"></a>Télécharger le fichier de configuration réseau

1. Téléchargez le fichier de configuration réseau à l’aide de la procédure décrite dans l’article [Fichier de configuration réseau](/previous-versions/azure/virtual-network/virtual-networks-using-network-configuration-file). Ouvrez le fichier dans un éditeur de texte.
2. Ajouter un site réseau local au fichier. Vous pouvez utiliser n’importe quel préfixe d’adresse valide. Vous pouvez ajouter n’importe quelle adresse IP valide pour la passerelle VPN. Les valeurs d’adresse dans cette section ne sont pas utilisées pour les opérations d’ExpressRoute, mais sont requises pour la validation du fichier. Dans l’exemple, « branch1 » est le nom du site. Vous pouvez utiliser un autre nom, mais veillez à utiliser la même valeur dans la section Passerelle du fichier.

   ```
   <VirtualNetworkConfiguration>
    <Dns />
    <LocalNetworkSites>
      <LocalNetworkSite name="branch1">
        <AddressSpace>
          <AddressPrefix>165.3.1.0/27</AddressPrefix>
        </AddressSpace>
        <VPNGatewayAddress>3.2.1.4</VPNGatewayAddress>
    </LocalNetworkSite>
   ```
3. Accédez à VirtualNetworkSites et modifiez les champs.

   * Vérifiez que le sous-réseau de passerelle existe pour votre réseau virtuel. Dans le cas contraire, vous pouvez en ajouter un maintenant. Le nom doit être « GatewaySubnet ».
   * Vérifiez que la section Passerelle du fichier existe. Si ce n’est pas le cas, ajoutez-la. Cette section est nécessaire pour associer le réseau virtuel au site de réseau local (qui représente le réseau auquel vous vous connectez).
   * Vérifiez que le type de connexion = Dedicated (dédié). Cela est nécessaire pour les connexions ExpressRoute.

   ```
   </LocalNetworkSites>
    <VirtualNetworkSites>
      <VirtualNetworkSite name="myAzureVNET" Location="East US">
        <AddressSpace>
          <AddressPrefix>10.0.0.0/16</AddressPrefix>
        </AddressSpace>
        <Subnets>
          <Subnet name="default">
            <AddressPrefix>10.0.0.0/24</AddressPrefix>
          </Subnet>
          <Subnet name="GatewaySubnet">
            <AddressPrefix>10.0.1.0/27</AddressPrefix>
          </Subnet>
        </Subnets>
        <Gateway>
          <ConnectionsToLocalNetwork>
            <LocalNetworkSiteRef name="branch1">
              <Connection type="Dedicated" />
            </LocalNetworkSiteRef>
          </ConnectionsToLocalNetwork>
        </Gateway>
      </VirtualNetworkSite>
    </VirtualNetworkSites>
   </VirtualNetworkConfiguration>
   </NetworkConfiguration>
   ```
4. Enregistrez le fichier et chargez-le sur Azure.

### <a name="create-the-gateway"></a>Créer la passerelle

Utilisez la commande suivante pour créer une passerelle. Veillez à remplacer les valeurs pas les vôtres.

```powershell
New-AzureVNetGateway -VNetName "MyAzureVNET" -GatewayType DynamicRouting -GatewaySKU  Standard
```

## <a name="verify-the-gateway-was-created"></a>Vérification de la création de la passerelle

Utilisez la commande suivante pour vérifier que la passerelle a été créée. Cette commande récupère également l’ID de passerelle dont vous avez besoin pour d’autres opérations.

```powershell
Get-AzureVNetGateway
```

## <a name="resize-a-gateway"></a>Redimensionner une passerelle

Il existe un certain nombre de [Références (SKU) de passerelle](../articles/expressroute/expressroute-about-virtual-network-gateways.md). Vous pouvez utiliser la commande suivante pour modifier la référence de passerelle à tout moment.

> [!IMPORTANT]
> Cette commande ne fonctionne pas pour la passerelle UltraPerformance. Pour modifier votre passerelle en passerelle UltraPerformance, commencez par supprimer la passerelle ExpressRoute, puis créez une passerelle UltraPerformance. Pour mettre à niveau votre passerelle à partir d’une passerelle UltraPerformance, commencez par supprimer la passerelle UltraPerformance, puis créez-en une.
>
>

```powershell
Resize-AzureVNetGateway -GatewayId <Gateway ID> -GatewaySKU HighPerformance
```

## <a name="remove-a-gateway"></a>Supprimer une passerelle

Utilisez la commande suivante pour supprimer une passerelle.

```powershell
Remove-AzureVnetGateway -GatewayId <Gateway ID>
```