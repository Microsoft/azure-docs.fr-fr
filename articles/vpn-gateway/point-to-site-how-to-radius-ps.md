---
title: 'Connecter un ordinateur à un réseau virtuel à l’aide d’une connexion point à site et d’une authentification RADIUS : PowerShell | Azure'
description: Connectez des clients Windows et OS X en toute sécurité à un réseau virtuel à l’aide d’une connexion point à site et de l’authentification RADIUS.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 11/18/2020
ms.author: cherylmc
ms.openlocfilehash: 9d962d3a4757b4c7b2d217f91aaf73d6ad4164d3
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94964845"
---
# <a name="configure-a-point-to-site-connection-to-a-vnet-using-radius-authentication-powershell"></a>Configurer une connexion point à site à un réseau virtuel à l’aide d’une authentification RADIUS : PowerShell

Cet article explique comment créer un réseau virtuel avec une connexion point à site utilisant l’authentification RADIUS. Cette configuration n’est disponible que pour le modèle de déploiement Resource Manager.

Une connexion par passerelle VPN point à site (P2S) vous permet de créer une connexion sécurisée à votre réseau virtuel à partir d’un ordinateur de client individuel. Les connexions VPN point à site sont utiles quand vous souhaitez vous connecter à votre réseau virtuel à partir d’un emplacement distant, par exemple quand vous travaillez à distance de votre domicile ou en conférence. De même, l’utilisation d’un VPN P2S est une solution utile qui constitue une alternative au VPN Site à Site lorsqu’un nombre restreint de clients doivent se connecter à un réseau virtuel.

Une connexion VPN P2S est démarrée à partir d’appareils Windows et Mac. Les clients de connexion peuvent utiliser les méthodes d’authentification suivantes : 

* Serveur RADIUS
* Authentification par certificat native de la passerelle VPN
* Authentification Azure Active Directory native (Windows 10 uniquement)

Cet article vous aide à configurer une configuration P2S avec authentification à l’aide d’un serveur RADIUS. En revanche, si vous souhaitez vous authentifier à l’aide de certificats générés et d’une authentification par certificat native de passerelle VPN, consultez [Configurer une connexion point à site à un réseau virtuel à l’aide d’une authentification par certificat native de passerelle VPN](vpn-gateway-howto-point-to-site-rm-ps.md) ou [Créer un locataire Azure Active Directory pour les connexions de protocole OpenVPN P2S](openvpn-azure-ad-tenant.md) pour l’authentification Azure Active Directory.

![Diagramme illustrant la configuration point à site avec authentification à l’aide d’un serveur RADIUS.](./media/point-to-site-how-to-radius-ps/p2sradius.png)

Les connexions de point à site ne nécessitent pas de périphérique VPN ou d’adresse IP publique. La connexion P2S crée la connexion VPN via SSTP (Secure Socket Tunneling Protocol), OpenVPN ou IKEv2.

* SSTP est un tunnel VPN basé sur le protocole TLS qui n’est pris en charge que sur les plateformes clientes Windows. Il peut pénétrer des pare-feux, ce qui en fait une option idéale pour se connecter à Azure à partir de n’importe quel endroit. Côté serveur, nous prenons en charge SSTP, versions 1.0, 1.1 et 1.2. Le client détermine la version à utiliser. Pour Windows 8.1 et supérieur, SSTP utilise la version 1.2 par défaut.

* Protocole OpenVPN®, un protocole VPN basé sur SSL/TLS. Une solution VPN TLS peut pénétrer des pare-feu, puisque la plupart des pare-feu ouvrent le port de sortie TCP 443 utilisé par le protocole TLS. Vous pouvez utiliser OpenVPN pour vous connecter à partir d’appareils Android, iOS (versions 11.0 et versions ultérieures), Windows, Linux et Mac (OSX 10.13 et versions ultérieures).

* Un VPN IKEv2 est une solution VPN IPsec basée sur des normes. Un VPN IKEv2 peut être utilisé pour se connecter à partir d’appareils Mac (OSX 10.11 et versions ultérieures).

Les connexions P2S requièrent les éléments suivants :

* Une passerelle VPN RouteBased. 
* Un serveur RADIUS pour gérer l’authentification des utilisateurs. Le serveur RADIUS peut être déployé en local ou dans votre réseau virtuel Azure. Vous pouvez également configurer deux serveurs RADIUS pour une haute disponibilité.
* Un package de configuration du client VPN pour les appareils Windows qui seront connectés au réseau virtuel. Un package de configuration du client VPN fournit les paramètres requis pour connecter un client VPN via P2S.

## <a name="about-active-directory-ad-domain-authentication-for-p2s-vpns"></a><a name="aboutad"></a>À propos de l’authentification de domaine Active Directory (AD) pour les VPN P2S

L’authentification de domaine AD permet aux utilisateurs de se connecter à Azure à l’aide des informations d’identification du domaine de l’organisation. Un serveur RADIUS qui s’intègre avec le serveur AD est requis. Les organisations peuvent aussi exploiter un déploiement RADIUS existant.
 
Le serveur RADIUS peut être situé en local ou dans votre réseau virtuel Azure. Lors de l’authentification, la passerelle VPN permet le transfert direct et transfère les messages d’authentification entre le serveur RADIUS et l’appareil de connexion. Il est important que la passerelle VPN puisse accéder au serveur RADIUS. Si le serveur RADIUS est situé en local, une connexion VPN site à site au site local à partir d’Azure est requise.

Un serveur RADIUS permet également l’intégration avec d’autres systèmes d’identité externe, à l’exception d’Active Directory. Cette opération ouvre de nombreuses options d’authentification pour les VPN point à site, notamment les options de MFA. Vérifiez la documentation de votre fournisseur de serveur RADIUS pour obtenir la liste des systèmes d’identité qui y sont intégrés.

![Diagramme de connexions - RADIUS](./media/point-to-site-how-to-radius-ps/radiusimage.png)

> [!IMPORTANT]
>Seule une connexion VPN site à site peut être utilisée pour se connecter à un serveur RADIUS local. Une connexion ExpressRoute ne peut pas être utilisée.
>
>

## <a name="before-beginning"></a><a name="before"></a>Avant tout chose

Assurez-vous de disposer d’un abonnement Azure. Si vous ne disposez pas déjà d’un abonnement Azure, vous pouvez activer vos [avantages abonnés MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details) ou créer un [compte gratuit](https://azure.microsoft.com/pricing/free-trial).

### <a name="working-with-azure-powershell"></a>Utilisation d’Azure PowerShell

[!INCLUDE [PowerShell](../../includes/vpn-gateway-cloud-shell-powershell-about.md)]

### <a name="example-values"></a><a name="example"></a>Exemples de valeurs

Vous pouvez utiliser ces exemples de valeurs pour créer un environnement de test ou vous y référer pour mieux comprendre les exemples de cet article. Vous pouvez suivre les étapes proposées en utilisant les valeurs sans les modifier ou modifier les valeurs pour les adapter à votre environnement.

* **Nom : VNet1**
* **Espace d’adressage :192.168.0.0/16** et **10.254.0.0/16**<br>Pour cet exemple, nous utilisons différents espaces d’adressage afin de démontrer que cette configuration fonctionne avec plusieurs espaces d’adressage. Toutefois, plusieurs espaces d’adressage ne sont pas nécessaires pour cette configuration.
* **Nom du sous-réseau : FrontEnd**
  * **Plage d’adresses de sous-réseau : 192.168.1.0/24**
* **Nom du sous-réseau : BackEnd**
  * **Plage d’adresses de sous-réseau : 10.254.1.0/24**
* **Nom du sous-réseau : GatewaySubnet**<br>Le nom du sous-réseau *GatewaySubnet* est obligatoire pour que la passerelle VPN fonctionne.
  * **Plage d’adresses de GatewaySubnet : 192.168.200.0/24** 
* **Pool d’adresses des clients VPN : 172.16.201.0/24**<br>Les clients VPN qui se connectent au réseau virtuel à l’aide de cette connexion point à site reçoivent une adresse IP de ce pool d’adresses des clients VPN.
* **Abonnement :** vérifiez que vous utilisez l’abonnement approprié si vous en possédez plusieurs.
* **Groupe de ressources : TestRG**
* **Emplacement : USA Est**
* **Serveur DNS : l’adresse IP** du serveur DNS que vous souhaitez utiliser pour la résolution de noms pour votre réseau virtuel. (facultatif)
* **Nom de passerelle : Vnet1GW**
* **Nom d’adresse IP publique : VNet1GWPIP**
* **Type de VPN : RouteBased**

## <a name="1-set-the-variables"></a><a name="signin"></a>1. Définir les variables

Déclarez les variables que vous souhaitez utiliser. Utilisez l’exemple ci-dessous en utilisant vos propres valeurs si nécessaire. Si vous fermez votre session PowerShell/Cloud Shell au cours de l’exercice, copiez et collez les valeurs une nouvelle fois pour redéclarer les variables.

  ```azurepowershell-interactive
  $VNetName  = "VNet1"
  $FESubName = "FrontEnd"
  $BESubName = "Backend"
  $GWSubName = "GatewaySubnet"
  $VNetPrefix1 = "192.168.0.0/16"
  $VNetPrefix2 = "10.254.0.0/16"
  $FESubPrefix = "192.168.1.0/24"
  $BESubPrefix = "10.254.1.0/24"
  $GWSubPrefix = "192.168.200.0/26"
  $VPNClientAddressPool = "172.16.201.0/24"
  $RG = "TestRG"
  $Location = "East US"
  $GWName = "VNet1GW"
  $GWIPName = "VNet1GWPIP"
  $GWIPconfName = "gwipconf"
  ```

## <a name="2-create-the-resource-group-vnet-and-public-ip-address"></a>2. <a name="vnet"></a>Créer le groupe de ressources, le réseau virtuel et l’adresse IP publique

La procédure suivante crée un groupe de ressources et un réseau virtuel comprenant trois sous-réseaux dans le groupe de ressources. Lorsque vous remplacez les valeurs, pensez à toujours nommer votre sous-réseau de passerelle « GatewaySubnet ». Si vous le nommez autrement, la création de votre passerelle échoue.

1. Créez un groupe de ressources.

   ```azurepowershell-interactive
   New-AzResourceGroup -Name "TestRG" -Location "East US"
   ```
2. Créez les configurations de sous-réseau du réseau virtuel en les nommant *FrontEnd*, *BackEnd* et *GatewaySubnet*. Ces préfixes doivent faire partie de l’espace d’adressage du réseau virtuel que vous avez déclaré.

   ```azurepowershell-interactive
   $fesub = New-AzVirtualNetworkSubnetConfig -Name "FrontEnd" -AddressPrefix "192.168.1.0/24"  
   $besub = New-AzVirtualNetworkSubnetConfig -Name "Backend" -AddressPrefix "10.254.1.0/24"  
   $gwsub = New-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -AddressPrefix "192.168.200.0/24"
   ```
3. Création du réseau virtuel.

   Dans cet exemple, le paramètre du serveur -DnsServer est facultatif. La définition d’une valeur n’entraîne pas la création de serveur DNS. Le serveur DNS dont vous spécifiez l’adresse IP doit pouvoir résoudre les noms des ressources auxquelles vous vous connectez depuis votre réseau virtuel. Pour cet exemple, nous avons utilisé une adresse IP privée, mais il ne s’agit probablement pas de l’adresse IP de votre serveur DNS. Veillez à utiliser vos propres valeurs. La valeur que vous spécifiez est utilisée par les ressources que vous déployez sur le réseau virtuel et non par la connexion P2S.

   ```azurepowershell-interactive
   New-AzVirtualNetwork -Name "VNet1" -ResourceGroupName "TestRG" -Location "East US" -AddressPrefix "192.168.0.0/16","10.254.0.0/16" -Subnet $fesub, $besub, $gwsub -DnsServer 10.2.1.3
   ```
4. Une passerelle VPN doit avoir une adresse IP publique. Vous commencez par demander la ressource d’adresse IP, puis vous y faites référence lors de la création de votre passerelle de réseau virtuel. L’adresse IP est affectée dynamiquement à la ressource lors de la création de la passerelle VPN. Actuellement, la passerelle VPN prend uniquement en charge l’allocation d’adresses IP publiques *dynamiques*. Vous ne pouvez pas demander d’affectation d’adresse IP publique statique. Toutefois, cela ne signifie pas que l’adresse IP change après son affectation à votre passerelle VPN. L’adresse IP publique change uniquement lorsque la passerelle est supprimée, puis recréée. Elle n’est pas modifiée lors du redimensionnement, de la réinitialisation ou des autres opérations de maintenance/mise à niveau internes de votre passerelle VPN.

   Spécifiez les variables pour demander une adresse IP publique assignée dynamiquement.

   ```azurepowershell-interactive
   $vnet = Get-AzVirtualNetwork -Name "VNet1" -ResourceGroupName "TestRG"  
   $subnet = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet 
   $pip = New-AzPublicIpAddress -Name "VNet1GWPIP" -ResourceGroupName "TestRG" -Location "East US" -AllocationMethod Dynamic 
   $ipconf = New-AzVirtualNetworkGatewayIpConfig -Name "gwipconf" -Subnet $subnet -PublicIpAddress $pip
   ```

## <a name="3-set-up-your-radius-server"></a>3. <a name="radius"></a>Configurer votre serveur RADIUS

Avant de créer et de configurer la passerelle du réseau virtuel, votre serveur RADIUS doit être configuré correctement pour être authentifié.

1. Si vous n’avez pas de serveur RADIUS déployé, vous devez en déployer un. Pour les étapes du déploiement, consultez le guide d’installation fourni par votre fournisseur RADIUS.  
2. Configurer la passerelle VPN en tant que client RADIUS sur le RADIUS. Lorsque vous ajoutez un client RADIUS, spécifiez le réseau virtuel GatewaySubnet que vous avez créé. 
3. Une fois que le serveur RADIUS est configuré, obtenez l’adresse IP du serveur RADIUS et le secret partagé que les clients RADIUS doivent utiliser pour communiquer avec le serveur RADIUS. Si le serveur RADIUS se trouve dans le réseau virtuel Azure, utilisez l’adresse IP de l’autorité de certification de la machine virtuelle du serveur RADIUS.

L’article relatif au [serveur NPS (Network Policy Server)](/windows-server/networking/technologies/nps/nps-top) fournit des instructions sur la configuration d’un serveur RADIUS Windows (NPS) pour l’authentification de domaine AD.

## <a name="4-create-the-vpn-gateway"></a>4. <a name="creategw"></a>Créer la passerelle VPN

Configurez et créez la passerelle VPN pour votre réseau virtuel.

* Le paramètre -GatewayType doit être défini sur la valeur « Vpn », tandis que le paramètre -VpnType doit être défini sur la valeur RouteBased.
* L’achèvement d’une passerelle VPN peut prendre jusqu’à 45 minutes en fonction de la  [référence SKU de la passerelle](vpn-gateway-about-vpn-gateway-settings.md#gwsku)  que vous sélectionnez.

```azurepowershell-interactive
New-AzVirtualNetworkGateway -Name $GWName -ResourceGroupName $RG `
-Location $Location -IpConfigurations $ipconf -GatewayType Vpn `
-VpnType RouteBased -EnableBgp $false -GatewaySku VpnGw1
```

## <a name="5-add-the-radius-server-and-client-address-pool"></a>5. <a name="addradius"></a>Ajouter le serveur RADIUS et le pool d’adresses client
 
* Le paramètre -RadiusServer peut être spécifié par nom ou par adresse IP. Si vous spécifiez le nom, et si le serveur est situé en local, la passerelle VPN peut ne pas être en mesure de résoudre le nom. Si tel est le cas, il est préférable de spécifier l’adresse IP du serveur. 
* Le paramètre -RadiusSecret doit correspondre à ce qui est configuré sur votre serveur RADIUS.
* Le paramètre -VpnClientAddressPool est la plage à partir de laquelle les clients VPN reçoivent une adresse IP. Utilisez une plage d’adresses IP privées qui ne chevauche ni l’emplacement local à partir duquel vous vous connectez ni le réseau virtuel auquel vous souhaitez vous connecter. Assurez-vous que vous disposez d’un pool d’adresses configuré suffisamment large.  

1. Créez une chaîne sécurisée pour le secret RADIUS.

   ```azurepowershell-interactive
   $Secure_Secret=Read-Host -AsSecureString -Prompt "RadiusSecret"
   ```

2. Vous êtes invité à entrer le secret RADIUS. Les caractères que vous entrez n’apparaissent pas et sont remplacés par le caractère « * ».

   ```azurepowershell-interactive
   RadiusSecret:***
   ```
3. Ajoutez le pool d’adresses client VPN et les informations du serveur RADIUS.

   Pour configurer SSTP :

    ```azurepowershell-interactive
    $Gateway = Get-AzVirtualNetworkGateway -ResourceGroupName $RG -Name $GWName
    Set-AzVirtualNetworkGateway -VirtualNetworkGateway $Gateway `
    -VpnClientAddressPool "172.16.201.0/24" -VpnClientProtocol "SSTP" `
    -RadiusServerAddress "10.51.0.15" -RadiusServerSecret $Secure_Secret
    ```

   Pour les configurations OpenVPN® :

    ```azurepowershell-interactive
    $Gateway = Get-AzVirtualNetworkGateway -ResourceGroupName $RG -Name $GWName
    Set-AzVirtualNetworkGateway -VirtualNetworkGateway $Gateway -VpnClientRootCertificates @()
    Set-AzVirtualNetworkGateway -VirtualNetworkGateway $Gateway `
    -VpnClientAddressPool "172.16.201.0/24" -VpnClientProtocol "OpenVPN" `
    -RadiusServerAddress "10.51.0.15" -RadiusServerSecret $Secure_Secret
    ```


   Pour configurer IKEv2 :

    ```azurepowershell-interactive
    $Gateway = Get-AzVirtualNetworkGateway -ResourceGroupName $RG -Name $GWName
    Set-AzVirtualNetworkGateway -VirtualNetworkGateway $Gateway `
    -VpnClientAddressPool "172.16.201.0/24" -VpnClientProtocol "IKEv2" `
    -RadiusServerAddress "10.51.0.15" -RadiusServerSecret $Secure_Secret
    ```

   Pour SSTP + IKEv2

    ```azurepowershell-interactive
    $Gateway = Get-AzVirtualNetworkGateway -ResourceGroupName $RG -Name $GWName
    Set-AzVirtualNetworkGateway -VirtualNetworkGateway $Gateway `
    -VpnClientAddressPool "172.16.201.0/24" -VpnClientProtocol @( "SSTP", "IkeV2" ) `
    -RadiusServerAddress "10.51.0.15" -RadiusServerSecret $Secure_Secret
    ```

   Pour spécifier **deux** serveurs RADIUS, utilisez la syntaxe suivante. Modifiez la valeur de **-VpnClientProtocol** en fonction des besoins

    ```azurepowershell-interactive
    $radiusServer1 = New-AzRadiusServer -RadiusServerAddress 10.1.0.15 -RadiusServerSecret $radiuspd -RadiusServerScore 30
    $radiusServer2 = New-AzRadiusServer -RadiusServerAddress 10.1.0.16 -RadiusServerSecret $radiuspd -RadiusServerScore 1

    $radiusServers = @( $radiusServer1, $radiusServer2 )

    Set-AzVirtualNetworkGateway -VirtualNetworkGateway $actual -VpnClientAddressPool 201.169.0.0/16 -VpnClientProtocol "IkeV2" -RadiusServerList $radiusServers
    ```

## <a name="6-download-the-vpn-client-configuration-package-and-set-up-the-vpn-client"></a>6. <a name="vpnclient"></a>Téléchargement du package de configuration du client VPN et configuration du client VPN

La configuration du client VPN permet aux appareils de se connecter à un réseau virtuel via une connexion P2S. Pour générer un package de configuration du client VPN et configurer le client VPN, consultez [Créer une configuration du client VPN pour une authentification RADIUS](point-to-site-vpn-client-configuration-radius.md).

## <a name="7-connect-to-azure"></a><a name="connect"></a>7. Connexion à Azure

### <a name="to-connect-from-a-windows-vpn-client"></a>Se connecter à partir d’un client VPN Windows

1. Pour vous connecter à votre réseau virtuel, sur l’ordinateur client, accédez aux connexions VPN et recherchez celle que vous avez créée. Elle porte le même nom que votre réseau virtuel. Entrez vos informations d’identification de domaine, puis cliquez sur « Se connecter ». Un message demandant des droits élevés s’affiche. Acceptez et entrez les informations d’identification.

   ![Connexion du client VPN à Azure](./media/point-to-site-how-to-radius-ps/client.png)
2. Votre connexion est établie.

   ![Connexion établie](./media/point-to-site-how-to-radius-ps/connected.png)

### <a name="connect-from-a-mac-vpn-client"></a>Se connecter à partir d’un client VPN Mac

À partir de la boîte de dialogue Réseau, recherchez le profil de client que vous souhaitez utiliser, puis cliquez sur **Connexion**.

  ![Connexion Mac](./media/vpn-gateway-howto-point-to-site-rm-ps/applyconnect.png)

## <a name="to-verify-your-connection"></a><a name="verify"></a>Pour vérifier votre connexion

1. Pour vérifier que votre connexion VPN est active, ouvrez une invite de commandes avec élévation de privilèges, puis exécutez *ipconfig/all*.
2. Affichez les résultats. Notez que l’adresse IP que vous avez reçue est l’une des adresses du pool d’adresses de client VPN point à site que vous avez spécifiées dans votre configuration. Les résultats ressemblent à l’exemple qui suit :

   ```
   PPP adapter VNet1:
      Connection-specific DNS Suffix .:
      Description.....................: VNet1
      Physical Address................:
      DHCP Enabled....................: No
      Autoconfiguration Enabled.......: Yes
      IPv4 Address....................: 172.16.201.3(Preferred)
      Subnet Mask.....................: 255.255.255.255
      Default Gateway.................:
      NetBIOS over Tcpip..............: Enabled
   ```

Pour résoudre les problèmes liés aux connexions point à site, consultez [Résolution des problèmes de connexion de point à site Azure](vpn-gateway-troubleshoot-vpn-point-to-site-connection-problems.md).

## <a name="to-connect-to-a-virtual-machine"></a><a name="connectVM"></a>Se connecter à une machine virtuelle

[!INCLUDE [Connect to a VM](../../includes/vpn-gateway-connect-vm.md)]

* Vérifiez que le package de configuration du client VPN a été généré après que les adresses IP du serveur DNS ont été spécifiées pour le réseau virtuel. Si vous avez mis à jour les adresses IP du serveur DNS, générez et installez un package de configuration du client VPN.

* Utilisez « ipconfig » pour vérifier l’adresse IPv4 attribuée à l’adaptateur Ethernet sur l’ordinateur à partir duquel vous vous connectez. Si l’adresse IP est comprise dans la plage d’adresses du réseau virtuel auquel vous vous connectez, ou dans la plage d’adresses de votre VPNClientAddressPool, cette situation est désignée sous le terme d’espaces d’adressage qui se chevauchent. Lorsque vos espaces d’adressage se chevauchent de cette façon, le trafic réseau n’atteint pas Azure et reste sur le réseau local.

## <a name="faq"></a><a name="faq"></a>Forum Aux Questions

Ce forum aux questions concerne le P2S à l’aide de l’authentification RADIUS

[!INCLUDE [Point-to-Site RADIUS FAQ](../../includes/vpn-gateway-faq-p2s-radius-include.md)]

## <a name="next-steps"></a>Étapes suivantes

Une fois la connexion achevée, vous pouvez ajouter des machines virtuelles à vos réseaux virtuels. Pour plus d’informations, consultez [Machines virtuelles](../index.yml). Pour plus d’informations sur la mise en réseau et les machines virtuelles, consultez [Vue d’ensemble du réseau de machines virtuelles Azure et Linux](../virtual-machines/network-overview.md).
