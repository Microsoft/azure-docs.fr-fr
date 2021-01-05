---
title: 'Configurer la coexistence de connexions à ExpressRoute et S2S VPN : classique'
description: Cet article vous guide tout au long de la configuration d’une connexion ExpressRoute et d’une connexion VPN de site à site pouvant coexister pour le modèle de déploiement classique.
documentationcenter: na
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 12/06/2019
ms.author: duau
ms.openlocfilehash: 70407d726b697dd6c4dfb6777cf6c445092d3b73
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92206323"
---
# <a name="configure-expressroute-and-site-to-site-coexisting-connections-classic"></a>Configurer la coexistence de connexions de site à site et ExpressRoute (classique)
> [!div class="op_single_selector"]
> * [PowerShell - Resource Manager](expressroute-howto-coexist-resource-manager.md)
> * [PowerShell - Classique](expressroute-howto-coexist-classic.md)
> 
> 

Cet article vous aide à configurer la coexistence de connexions ExpressRoute et VPN de site à site. La possibilité de configurer des connexions VPN de site à site et ExpressRoute présente plusieurs avantages. Vous pouvez configurer un VPN de site à site en tant que chemin de basculement sécurisé pour ExpressRoute, ou utiliser des VPN de site à site pour vous connecter à des sites qui ne sont pas connectés via ExpressRoute. Dans cet article, nous décrirons les étapes de configuration des deux scénarios. Cet article s’applique au modèle de déploiement classique. Cette configuration n'est pas disponible dans le portail.

[!INCLUDE [expressroute-classic-end-include](../../includes/expressroute-classic-end-include.md)]

**À propos des modèles de déploiement Azure**

[!INCLUDE [vpn-gateway-classic-rm](../../includes/vpn-gateway-classic-rm-include.md)]

> [!IMPORTANT]
> Les circuits ExpressRoute doivent être préconfigurés avant que vous suiviez les instructions ci-dessous. Assurez-vous que vous avez suivi les guides [Créer un circuit ExpressRoute](expressroute-howto-circuit-classic.md) et [Configurer le routage](expressroute-howto-routing-classic.md) avant de suivre les étapes ci-dessous.
> 
> 

## <a name="limits-and-limitations"></a>Limites et limitations
* **Le routage de transit n’est pas pris en charge.** Vous ne pouvez effectuer de routage (via Azure) entre votre réseau local connecté via le réseau VPN de site à site et votre réseau local connecté via ExpressRoute.
* **Le routage point à site n’est pas pris en charge.** Vous ne pouvez pas activer de connexions VPN point à site avec le même réseau virtuel connecté à ExpressRoute. Des connexions VPN point à site et ExpressRoute ne peuvent pas coexister pour le même réseau virtuel.
* **Le tunneling forcé ne peut pas être activé sur la passerelle VPN de site à site.** Vous pouvez uniquement « forcer » tout le trafic Internet vers votre réseau local via ExpressRoute.
* **La passerelle de référence de base n’est pas prise en charge.** Vous devez utiliser une passerelle de référence SKU autre que De base pour la [passerelle ExpressRoute](expressroute-about-virtual-network-gateways.md) et la [passerelle VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md).
* **Seule la passerelle VPN basée sur un itinéraire est prise en charge.** Vous devez utiliser une [passerelle VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md) basée sur un itinéraire.
* L’**itinéraire statique doit être configuré pour votre passerelle VPN.** Si votre réseau local est connecté à la fois à ExpressRoute et à un VPN de site à site, vous devez avoir configuré un itinéraire statique sur votre réseau local pour acheminer la connexion VPN de site à site vers l’Internet public.
* La **passerelle ExpressRoute doit être configurée en premier.** Vous devez commencer par créer la passerelle ExpressRoute avant d’ajouter la passerelle VPN de site à site.

## <a name="configuration-designs"></a>Modèles de configuration
### <a name="configure-a-site-to-site-vpn-as-a-failover-path-for-expressroute"></a>Configurer un réseau VPN de site à site comme un chemin d’accès de basculement pour ExpressRoute
Vous pouvez configurer une connexion VPN de site à site en tant que sauvegarde pour ExpressRoute. Cela s’applique uniquement aux réseaux virtuels liés au chemin de peering privé Azure. Il n’existe aucune solution de basculement basée sur des réseaux VPN pour les services accessibles via les peerings Azure public et Microsoft. Le circuit ExpressRoute est toujours le lien principal. Les données circulent via le chemin d’accès du réseau VPN de site à site uniquement si le circuit ExpressRoute échoue. 

> [!NOTE]
> Bien que le circuit ExpressRoute soit préférable au réseau VPN de site à site lorsque les deux itinéraires sont identiques, Azure utilise la correspondance de préfixe la plus longue pour choisir l’itinéraire vers la destination du paquet.
> 
> 

![Diagramme montrant une connexion VPN de site à site en tant que sauvegarde pour ExpressRoute.](media/expressroute-howto-coexist-classic/scenario1.jpg)

### <a name="configure-a-site-to-site-vpn-to-connect-to-sites-not-connected-through-expressroute"></a>Configurer un réseau VPN de site à site pour se connecter à des sites non connectés via ExpressRoute
Vous pouvez configurer votre réseau là où certains sites se connectent directement à Azure via des réseaux VPN de site à site ou via ExpressRoute. 

![Coexister](media/expressroute-howto-coexist-classic/scenario2.jpg)

> [!NOTE]
> Vous ne pouvez pas configurer un réseau virtuel comme un routeur de transit.
> 
> 

## <a name="selecting-the-steps-to-use"></a>Sélection des étapes à suivre
Vous pouvez choisir entre deux procédures différentes pour configurer vos connexions afin qu’elles coexistent. La procédure de configuration que vous sélectionnez varie selon que vous disposez déjà d’un réseau virtuel auquel vous connecter ou que vous voulez créer un réseau virtuel.

* Je n’ai pas de réseau virtuel et dois en créer un
  
    Si vous ne disposez pas déjà d’un réseau virtuel, cette procédure vous guide dans la création d’un réseau virtuel en utilisant le modèle de déploiement classique et dans l’établissement de nouvelles connexions ExpressRoute et VPN de site à site. Pour procéder à la configuration, suivez les étapes décrites dans la section [Créer un réseau virtuel et des connexions qui coexistent](#new).
* J’ai déjà un réseau virtuel répondant au modèle de déploiement classique
  
    Vous disposez peut-être déjà d’un réseau virtuel avec une connexion VPN de site à site existante ou une connexion ExpressRoute. La section [Configurer des connexions qui coexistent pour un réseau virtuel existant](#add) de l’article vous guide tout au long des étapes de suppression de la passerelle et de création de connexions ExpressRoute et VPN de site à site. Notez que vous devez effectuer les étapes dans un ordre très spécifique lorsque vous créez les connexions. N’utilisez pas les instructions contenues dans d’autres articles pour créer des connexions et des passerelles.
  
    Lors de cette procédure, si vous créez des connexions pouvant coexister, vous devez supprimer votre passerelle, puis configurer de nouvelles passerelles. En d’autres termes, vous subissez un temps d’arrêt pour les connexions entre différents locaux lorsque vous supprimez et recréez la passerelle et les connexions, mais vous ne devez pas migrer les ordinateurs virtuels ou les services vers un nouveau réseau virtuel. Les machines virtuelles et les services sont toujours en mesure de communiquer via l’équilibreur de charge lorsque vous configurez votre passerelle s’ils sont configurés pour ce faire.

## <a name="install-powershell-cmdlets"></a>Installer les applets de commande PowerShell

[!INCLUDE [classic powershell install instructions](../../includes/expressroute-poweshell-classic-install-include.md)]

## <a name="to-create-a-new-virtual-network-and-coexisting-connections"></a><a name="new"></a>Créer un réseau virtuel et des connexions qui coexistent
Cette procédure vous guide dans la création d’un réseau virtuel et dans l’établissement de nouvelles connexions de site à site et ExpressRoute appelées à coexister.

1. Vous aurez besoin d’installer la dernière version des applets de commande PowerShell Azure. Pour plus d’informations sur l’installation des applets de commande PowerShell, consultez [Installation et configuration d’Azure PowerShell](/powershell/azure/). Les applets de commande que vous utiliserez pour cette configuration peuvent être légèrement différentes de celles que vous connaissez. Utilisez les applets de commande spécifiées dans ces instructions. 
2. Créez un schéma pour votre réseau virtuel. Pour plus d’informations sur le schéma de configuration, consultez la page [Schéma de configuration du réseau virtuel Azure](/previous-versions/azure/reference/jj157100(v=azure.100)).
   
    Lorsque vous créez votre schéma, veillez à utiliser les valeurs suivantes :
   
   * Le sous-réseau de la passerelle du réseau virtuel doit être défini sur /27 ou un préfixe plus court (comme /26 ou /25).
   * La connexion à la passerelle est du type « dédié ».
     
    ```xml
    <VirtualNetworkSite name="MyAzureVNET" Location="Central US">
      <AddressSpace>
        <AddressPrefix>10.17.159.192/26</AddressPrefix>
      </AddressSpace>
      <Subnets>
        <Subnet name="Subnet-1">
          <AddressPrefix>10.17.159.192/27</AddressPrefix>
        </Subnet>
        <Subnet name="GatewaySubnet">
          <AddressPrefix>10.17.159.224/27</AddressPrefix>
          /Subnet>
      </Subnets>
      <Gateway>
        <ConnectionsToLocalNetwork>
          <LocalNetworkSiteRef name="MyLocalNetwork">
            <Connection type="Dedicated" />
          </LocalNetworkSiteRef>
        </ConnectionsToLocalNetwork>
      </Gateway>
    </VirtualNetworkSite>
    ```
3. Après avoir créé et configuré votre fichier de schéma xml, chargez-le vers le serveur. Cette opération crée votre réseau virtuel.
   
    Utilisez l’applet de commande suivante pour charger votre fichier vers le serveur en remplaçant la valeur par la vôtre.
   
    ```azurepowershell
    Set-AzureVNetConfig -ConfigurationPath 'C:\NetworkConfig.xml'
    ```
4. <a name="gw"></a>Créez une passerelle ExpressRoute. Veillez à spécifier la valeur *Standard*, *HighPerformance* ou *UltraPerformance* pour le paramètre GatewaySKU, et la valeur *DynamicRouting* pour le paramètre GatewayType.
   
    Utilisez l’exemple ci-dessous en utilisant vos propres valeurs.

    ```azurepowershell
    New-AzureVNetGateway -VNetName MyAzureVNET -GatewayType DynamicRouting -GatewaySKU HighPerformance
    ```
5. Liez la passerelle ExpressRoute au circuit ExpressRoute. Une fois cette étape terminée, la connexion entre votre réseau local et Azure est établie via ExpressRoute.
   
    ```azurepowershell
    New-AzureDedicatedCircuitLink -ServiceKey <service-key> -VNetName MyAzureVNET
    ```
6. <a name="vpngw"></a>Créez ensuite la passerelle VPN de site à site. Vous devez spécifier la valeur *Standard*, *HighPerformance* ou *UltraPerformance* pour le paramètre GatewaySKU, et la valeur *DynamicRouting* pour le paramètre GatewayType.
   
    ```azurepowershell
    New-AzureVirtualNetworkGateway -VNetName MyAzureVNET -GatewayName S2SVPN -GatewayType DynamicRouting -GatewaySKU  HighPerformance
    ```
   
    Pour récupérer les paramètres de la passerelle de réseau virtuel, y compris l’ID de passerelle et l’adresse IP publique, utilisez l’applet de commande `Get-AzureVirtualNetworkGateway`.
   
    ```azurepowershell
    Get-AzureVirtualNetworkGateway
   
    GatewayId            : 348ae011-ffa9-4add-b530-7cb30010565e
    GatewayName          : S2SVPN
    LastEventData        :
    GatewayType          : DynamicRouting
    LastEventTimeStamp   : 5/29/2015 4:41:41 PM
    LastEventMessage     : Successfully created a gateway for the following virtual network: GNSDesMoines
    LastEventID          : 23002
    State                : Provisioned
    VIPAddress           : 104.43.x.y
    DefaultSite          :
    GatewaySKU           : HighPerformance
    Location             :
    VnetId               : 979aabcf-e47f-4136-ab9b-b4780c1e1bd5
    SubnetId             :
    EnableBgp            : False
    OperationDescription : Get-AzureVirtualNetworkGateway
    OperationId          : 42773656-85e1-a6b6-8705-35473f1e6f6a
    OperationStatus      : Succeeded
    ```
7. Créez une entité de passerelle VPN de site local. Cette commande ne configure pas votre passerelle VPN locale. Elle vous permet d’indiquer les paramètres de la passerelle locale, par exemple l’adresse IP publique et l’espace d’adressage local afin que la passerelle VPN Azure puisse s’y connecter.
   
   > [!IMPORTANT]
   > Le site local du réseau privé virtuel de site à site n’est pas défini dans le fichier netcfg. Vous devez utiliser cette applet de commande pour spécifier les paramètres du site local. Vous ne pouvez pas le définir à l’aide du portail ou du fichier netcfg.
   > 
   > 
   
    Utilisez l’exemple suivant en remplaçant les valeurs par les vôtres.
   
    ```azurepowershell
    New-AzureLocalNetworkGateway -GatewayName MyLocalNetwork -IpAddress <MyLocalGatewayIp> -AddressSpace <MyLocalNetworkAddress>
    ```
   
   > [!NOTE]
   > Si votre réseau local possède plusieurs itinéraires, vous pouvez tous les transmettre sous la forme d’un tableau.  $MyLocalNetworkAddress = @("10.1.2.0/24","10.1.3.0/24","10.2.1.0/24")  
   > 
   > 

    Pour récupérer les paramètres de la passerelle de réseau virtuel, y compris l’ID de passerelle et l’adresse IP publique, utilisez l’applet de commande `Get-AzureVirtualNetworkGateway`. Consultez l’exemple qui suit.

    ```azurepowershell
    Get-AzureLocalNetworkGateway

    GatewayId            : 532cb428-8c8c-4596-9a4f-7ae3a9fcd01b
    GatewayName          : MyLocalNetwork
    IpAddress            : 23.39.x.y
    AddressSpace         : {10.1.2.0/24}
    OperationDescription : Get-AzureLocalNetworkGateway
    OperationId          : ddc4bfae-502c-adc7-bd7d-1efbc00b3fe5
    OperationStatus      : Succeeded
    ```


1. Configurez votre périphérique VPN local à connecter à la nouvelle passerelle. Utilisez les informations que vous avez récupérées à l’étape 6 lors de la configuration de votre périphérique VPN. Pour plus d’informations sur la configuration du périphérique VPN, consultez la rubrique [Configuration de périphérique VPN](../vpn-gateway/vpn-gateway-about-vpn-devices.md).
2. Liez la passerelle VPN de site à site dans Azure à la passerelle locale.
   
    Dans cet exemple, connectedEntityId est l’ID de la passerelle locale que vous pouvez trouver en exécutant `Get-AzureLocalNetworkGateway`. Vous pouvez trouver virtualNetworkGatewayId à l’aide de l’applet de commande `Get-AzureVirtualNetworkGateway` . Après cette étape, la connexion entre votre réseau local et Azure est établie via la connexion VPN de site à site.

    ```azurepowershell
    New-AzureVirtualNetworkGatewayConnection -connectedEntityId <local-network-gateway-id> -gatewayConnectionName Azure2Local -gatewayConnectionType IPsec -sharedKey abc123 -virtualNetworkGatewayId <azure-s2s-vpn-gateway-id>
    ```

## <a name="to-configure-coexisting-connections-for-an-already-existing-vnet"></a><a name="add"></a>Pour configurer des connexions coexistantes pour un réseau virtuel existant
Si vous disposez déjà d’un réseau virtuel, vérifiez la taille du sous-réseau de passerelle. Si le sous-réseau de passerelle est /28 ou /29, vous devez tout d’abord supprimer la passerelle de réseau virtuel et augmenter la taille du sous-réseau de passerelle. Les étapes décrites dans cette section vous indiquent la procédure à suivre.

Si le sous-réseau de passerelle est défini sur/27 ou plus et si le réseau virtuel est connecté via ExpressRoute, vous pouvez ignorer les étapes ci-dessous et passer à [« Étape 6 : créer une passerelle VPN de site à site »](#vpngw) dans la section précédente.

> [!NOTE]
> Lorsque vous supprimez la passerelle existante, votre site local perdra la connexion à votre réseau virtuel lorsque vous effectuerez cette configuration.
> 
> 

1. Vous aurez besoin d’installer la dernière version des applets de commande PowerShell Azure Resource Manager. Pour plus d’informations sur l’installation des applets de commande PowerShell, consultez [Installation et configuration d’Azure PowerShell](/powershell/azure/). Les applets de commande que vous utiliserez pour cette configuration peuvent être légèrement différentes de celles que vous connaissez. Utilisez les applets de commande spécifiées dans ces instructions. 
2. Supprimez la passerelle VPN ExpressRoute ou de site à site existante. Utilisez l’applet de commande suivante en remplaçant les valeurs par les vôtres.
   
    ```azurepowershell
    Remove-AzureVNetGateway –VnetName MyAzureVNET
    ```
3. Exportez le schéma du réseau virtuel. Utilisez l’applet de commande PowerShell suivante en remplaçant les valeurs par les vôtres.
   
    ```azurepowershell
    Get-AzureVNetConfig –ExportToFile "C:\NetworkConfig.xml"
    ```
4. Modifiez le schéma du fichier de configuration réseau pour que le sous-réseau de passerelle soit défini sur /27 ou un préfixe plus court (comme /26 ou /25). Consultez l’exemple qui suit. 
   
   > [!NOTE]
   > S’il ne vous reste pas suffisamment d’adresses IP dans votre réseau virtuel pour augmenter la taille du sous-réseau de passerelle, vous devez augmenter l’espace d’adresses IP. Pour plus d’informations sur le schéma de configuration, consultez la page [Schéma de configuration du réseau virtuel Azure](/previous-versions/azure/reference/jj157100(v=azure.100)).
   > 
   > 
   
    ```xml
    <Subnet name="GatewaySubnet">
      <AddressPrefix>10.17.159.224/27</AddressPrefix>
    </Subnet>
    ```
5. Si votre passerelle précédente était une passerelle VPN de site à site, vous devez également modifier le type de connexion et le définir sur **Dédié**.
   
    ```xml
    <Gateway>
      <ConnectionsToLocalNetwork>
        <LocalNetworkSiteRef name="MyLocalNetwork">
          <Connection type="Dedicated" />
        </LocalNetworkSiteRef>
      </ConnectionsToLocalNetwork>
    </Gateway>
    ```
6. À ce stade, vous disposez d’un réseau virtuel sans passerelles. Pour créer de nouvelles passerelles et finaliser vos connexions, vous pouvez passer à l’ [Étape 4 : Créer une passerelle ExpressRoute](#gw), dans les étapes qui précèdent.

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur ExpressRoute, consultez la [FAQ sur ExpressRoute](expressroute-faqs.md)