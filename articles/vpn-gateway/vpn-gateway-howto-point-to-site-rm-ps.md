---
title: 'Se connecter à un réseau virtuel depuis un ordinateur – Réseau virtuel P2S et authentification native par certificat Azure : PowerShell'
description: Connectez des clients Windows et macOS de façon sécurisée à un réseau virtuel Azure à l’aide d’une connexion P2S et de certificats auto-signés ou délivrés par une autorité de certification. Cet article se base sur PowerShell.
titleSuffix: Azure VPN Gateway
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 10/29/2020
ms.author: cherylmc
ms.openlocfilehash: ed0a60c88c33af70b7d780d6c4735c5f8e65b35b
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94660404"
---
# <a name="configure-a-point-to-site-vpn-connection-to-a-vnet-using-native-azure-certificate-authentication-powershell"></a>Configurez une connexion point à site à un réseau virtuel à l’aide de l’authentification par certificat Azure native : PowerShell

Cet article vous permet de connecter de façon sécurisée des clients individuels qui exécutent Windows, Linux ou macOS à un réseau virtuel Azure. Les connexions VPN point à site sont utiles quand vous souhaitez vous connecter à votre réseau virtuel à distance, par exemple quand vous télétravaillez à domicile ou que vous êtes en conférence. La connexion P2S est une solution alternative au VPN de site à site lorsque seul un nombre restreint de clients doivent se connecter à un réseau virtuel. Les connexions point à site ne nécessitent pas de périphérique VPN ou d’adresse IP publique. La connexion P2S crée la connexion VPN via SSTP (Secure Socket Tunneling Protocol) ou IKEv2.

:::image type="content" source="./media/vpn-gateway-how-to-point-to-site-rm-ps/point-to-site-diagram.png" alt-text="Diagramme de connexion point à site d’un ordinateur à un réseau virtuel Azure":::

Pour plus d’informations sur le VPN point à site, consultez [À propos du VPN point à site](point-to-site-about.md). Pour créer cette configuration à l’aide du portail Azure, consultez [Configurer un VPN point à site à l’aide du portail Azure](vpn-gateway-howto-point-to-site-resource-manager-portal.md).

[!INCLUDE [P2S basic architecture](../../includes/vpn-gateway-p2s-architecture.md)]

## <a name="prerequisites"></a>Prérequis

Assurez-vous de disposer d’un abonnement Azure. Si vous ne disposez pas déjà d’un abonnement Azure, vous pouvez activer vos [avantages abonnés MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details) ou créer un [compte gratuit](https://azure.microsoft.com/pricing/free-trial).

### <a name="azure-powershell"></a>Azure PowerShell

>[!IMPORTANT]
> Vous pouvez utiliser Azure Cloud Shell dans bon nombre des étapes de cet article. Toutefois, vous ne pouvez pas utiliser Cloud Shell pour générer des certificats. Par ailleurs, pour charger la clé publique du certificat racine, vous devez utiliser Azure PowerShell localement ou le portail Azure.
>

[!INCLUDE [PowerShell](../../includes/vpn-gateway-cloud-shell-powershell-about.md)]

## <a name="1-sign-in"></a><a name="signin"></a>1. Se connecter

[!INCLUDE [sign in](../../includes/vpn-gateway-cloud-shell-ps-login.md)]

## <a name="2-declare-variables"></a><a name="declare"></a>2. Déclarer des variables

Nous utilisons des variables dans cet article pour vous permettre d’utiliser facilement des valeurs propres à votre environnement sans avoir à modifier les exemples. Déclarez les variables que vous souhaitez utiliser. Vous pouvez utiliser l’exemple suivant avec vos propres valeurs si nécessaire. Si vous fermez votre session PowerShell/Cloud Shell au cours de l’exercice, copiez et collez les valeurs une nouvelle fois pour redéclarer les variables.

```azurepowershell-interactive
$VNetName  = "VNet1"
$FESubName = "FrontEnd"
$GWSubName = "GatewaySubnet"
$VNetPrefix = "10.1.0.0/16"
$FESubPrefix = "10.1.0.0/24"
$GWSubPrefix = "10.1.255.0/27"
$VPNClientAddressPool = "172.16.201.0/24"
$RG = "TestRG1"
$Location = "EastUS"
$GWName = "VNet1GW"
$GWIPName = "VNet1GWpip"
$GWIPconfName = "gwipconf"
$DNS = "10.2.1.4"
```

## <a name="3-configure-a-vnet"></a><a name="ConfigureVNet"></a>3. Configurer un réseau virtuel

1. Créez un groupe de ressources.

   ```azurepowershell-interactive
   New-AzResourceGroup -Name $RG -Location $Location
   ```

1. Créez les configurations de sous-réseau du réseau virtuel et nommez-les *FrontEnd* et *GatewaySubnet*. Ces préfixes doivent faire partie de l’espace d’adressage du réseau virtuel que vous avez déclaré.

   ```azurepowershell-interactive
   $fesub = New-AzVirtualNetworkSubnetConfig -Name $FESubName -AddressPrefix $FESubPrefix
   $gwsub = New-AzVirtualNetworkSubnetConfig -Name $GWSubName -AddressPrefix $GWSubPrefix
   ```

1. Création du réseau virtuel.

   Dans cet exemple, le paramètre du serveur -DnsServer est facultatif. La définition d’une valeur n’entraîne pas la création de serveur DNS. Le serveur DNS dont vous spécifiez l’adresse IP doit pouvoir résoudre les noms des ressources auxquelles vous vous connectez depuis votre réseau virtuel. Cet exemple utilise une adresse IP privée, mais il ne s’agit probablement pas de l’adresse IP de votre serveur DNS. Veillez à utiliser vos propres valeurs. La valeur que vous spécifiez est utilisée par les ressources que vous déployez sur le réseau virtuel, et non par la connexion P2S ou le client VPN.

   ```azurepowershell-interactive
       New-AzVirtualNetwork `
      -ResourceGroupName $RG `
      -Location $Location `
      -Name $VNetName `
      -AddressPrefix $VNetPrefix `
      -Subnet $fesub, $gwsub `
      -DnsServer $DNS
   ```

1. Spécifiez les variables pour le réseau virtuel que vous avez créé.

   ```azurepowershell-interactive
   $vnet = Get-AzVirtualNetwork -Name $VNetName -ResourceGroupName $RG
   $subnet = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet
   ```

1. Une passerelle VPN doit avoir une adresse IP publique. Vous commencez par demander la ressource d’adresse IP, puis vous y faites référence lors de la création de votre passerelle de réseau virtuel. L’adresse IP est affectée dynamiquement à la ressource lors de la création de la passerelle VPN. Actuellement, la passerelle VPN prend uniquement en charge l’allocation d’adresses IP publiques *dynamiques*. Vous ne pouvez pas demander d’affectation d’adresse IP publique statique. Toutefois, cela ne signifie pas que l’adresse IP change après son affectation à votre passerelle VPN. L’adresse IP publique change uniquement lorsque la passerelle est supprimée, puis recréée. Elle n’est pas modifiée lors du redimensionnement, de la réinitialisation ou des autres opérations de maintenance/mise à niveau internes de votre passerelle VPN.

   Demandez une adresse IP publique attribuée dynamiquement.

   ```azurepowershell-interactive
   $pip = New-AzPublicIpAddress -Name $GWIPName -ResourceGroupName $RG -Location $Location -AllocationMethod Dynamic
   $ipconf = New-AzVirtualNetworkGatewayIpConfig -Name $GWIPconfName -Subnet $subnet -PublicIpAddress $pip
   ```

## <a name="4-create-the-vpn-gateway"></a><a name="creategateway"></a>4. Créer la passerelle VPN

Dans cette étape, vous configurez et créez la passerelle de réseau virtuel pour votre réseau virtuel.

* Le paramètre -GatewayType doit être défini sur la valeur **Vpn**, tandis que le paramètre -VpnType doit être défini sur la valeur **RouteBased**.
* Le paramètre -VpnClientProtocol est utilisé pour spécifier les types de tunnels que vous souhaitez activer. Les types de tunnels disponibles **OpenVPN, SSTP** et **IKEv2**. Vous pouvez choisir d’en activer l'un deux, voire une combinaison prise en charge. Si vous souhaitez activer plusieurs types, vous devez spécifier les noms séparés par une virgule. OpenVPN et SSTP ne peuvent pas être activés conjointement. Le client strongSwan sur Android et Linux et le client VPN IKEv2 natif sur iOS et OSX n’utiliseront que le tunnel IKEv2 pour se connecter. Les clients Windows essaient IKEv2 en premier lieu. En cas d’échec de la connexion, ils utilisent SSTP. Vous pouvez utiliser le client OpenVPN pour la connexion au type de tunnel OpenVPN.
* La référence SKU De base de la passerelle de réseau virtuel ne prend pas en charge IKEv2, OpenVPN ou l’authentification RADIUS. Si vous envisagez de connecter des clients Mac à votre réseau virtuel, n’utilisez pas la référence SKU de base.
* L’achèvement d’une passerelle VPN peut prendre jusqu’à 45 minutes en fonction de la [référence de passerelle](vpn-gateway-about-vpn-gateway-settings.md) que vous sélectionnez. Cet exemple utilise IKEv2.

1. Configurez et créez la passerelle de réseau virtuel pour votre réseau virtuel. La création de la passerelle prend environ 45 minutes.

   ```azurepowershell-interactive
   New-AzVirtualNetworkGateway -Name $GWName -ResourceGroupName $RG `
   -Location $Location -IpConfigurations $ipconf -GatewayType Vpn `
   -VpnType RouteBased -EnableBgp $false -GatewaySku VpnGw1 -VpnClientProtocol "IKEv2"
   ```

1. Une fois votre passerelle créée, vous pouvez l’afficher à l’aide de l’exemple suivant. Si vous avez fermé PowerShell ou que le délai d’attente a expiré lors de la création de votre passerelle, vous pouvez [redéclarer vos variables](#declare).

   ```azurepowershell-interactive
   Get-AzVirtualNetworkGateway -Name $GWName -ResourceGroup $RG
   ```

## <a name="5-add-the-vpn-client-address-pool"></a><a name="addresspool"></a>5. Ajouter le pool d’adresses des clients VPN

Après la création par la passerelle VPN, vous pouvez ajouter le pool d’adresses des clients VPN. Il s’agit de la plage à partir de laquelle les clients VPN reçoivent une adresse IP lorsqu’ils se connectent. Utilisez une plage d’adresses IP privées qui ne chevauche ni l’emplacement local à partir duquel vous vous connectez ni le réseau virtuel auquel vous souhaitez vous connecter.

Dans cet exemple, le pool d’adresses de clients VPN est déclaré comme une [variable](#declare) dans une étape précédente.

```azurepowershell-interactive
$Gateway = Get-AzVirtualNetworkGateway -ResourceGroupName $RG -Name $GWName
Set-AzVirtualNetworkGateway -VirtualNetworkGateway $Gateway -VpnClientAddressPool $VPNClientAddressPool
```

## <a name="6-generate-certificates"></a><a name="Certificates"></a>6. Générer des certificats

>[!IMPORTANT]
> Vous ne pouvez pas générer de certificats avec Azure Cloud Shell. Vous devez utiliser l’une des méthodes présentées dans cette section. Si vous souhaitez utiliser PowerShell, vous devez l’installer localement.
>

Les certificats sont utilisés par Azure afin d’authentifier les clients VPN pour les VPN point à site. Vous chargez les informations de la clé publique du certificat racine vers Azure. La clé publique est alors considérée comme « approuvée ». Les certificats clients doivent être générés à partir du certificat racine approuvé, puis installés sur chaque ordinateur client dans le magasin de certificats Utilisateur actuel/Personnel. Le certificat permet d’authentifier le client lorsqu’il établit une connexion avec le réseau virtuel. 

Si vous utilisez des certificats auto-signés, ceux-ci doivent être créés à l’aide de paramètres spécifiques. Vous pouvez créer un certificat auto-signé en suivant les instructions pour [PowerShell et Windows 10](vpn-gateway-certificates-point-to-site.md). Si vous n’avez pas Windows 10, vous pouvez utiliser [MakeCert](vpn-gateway-certificates-point-to-site-makecert.md). Il est important de suivre les étapes décrites dans les instructions lorsque vous générez des certificats racines auto-signés et des certificats clients. Dans le cas contraire, les certificats que vous générez ne seront pas compatibles avec les connexions P2S, ce qui entraînera une erreur de connexion.

### <a name="root-certificate"></a><a name="cer"></a>Certificat racine

1. [!INCLUDE [Root certificate](../../includes/vpn-gateway-p2s-rootcert-include.md)]

1. Après avoir créé le certificat racine, [exportez](vpn-gateway-certificates-point-to-site.md#cer) les données de certificat public (pas la clé privée) en tant que fichier .cer X.509 encodé en Base64.

### <a name="client-certificate"></a><a name="generate"></a>Certificat client

1. [!INCLUDE [Generate a client certificate](../../includes/vpn-gateway-p2s-clientcert-include.md)]

1. Après avoir généré le certificat client, [exportez](vpn-gateway-certificates-point-to-site.md#clientexport)-le. Le certificat client est alors distribué aux ordinateurs clients qui se connectent.

## <a name="7-upload-the-root-certificate-public-key-information"></a><a name="upload"></a>7. Charger les informations de la clé publique du certificat racine

Vérifiez que votre passerelle VPN a terminé la création. Une fois terminée, vous pouvez charger le fichier .cer (qui contient les informations de clé publique) pour un certificat racine approuvé dans Azure. Une fois qu’un fichier .cer est chargé, Azure peut l’utiliser pour authentifier les clients qui ont installé un certificat client généré à partir du certificat racine approuvé. Vous pouvez charger ultérieurement d’autres fichiers de certificat racine approuvé, jusqu’à un total de 20, si nécessaire.

>[!NOTE]
> Vous ne pouvez pas charger le fichier .cer à l’aide d’Azure Cloud Shell. Vous pouvez soit utiliser PowerShell localement sur votre ordinateur, soit utiliser les [étapes du Portail Azure](vpn-gateway-howto-point-to-site-resource-manager-portal.md#uploadfile).
>

1. Déclarez la variable pour le nom de votre certificat, en remplaçant la valeur par la vôtre.

   ```azurepowershell
   $P2SRootCertName = "P2SRootCert.cer"
   ```

1. Remplacez le chemin d’accès de fichier par le vôtre et exécutez les applets de commande.

   ```azurepowershell
   $filePathForCert = "C:\cert\P2SRootCert.cer"
   $cert = new-object System.Security.Cryptography.X509Certificates.X509Certificate2($filePathForCert)
   $CertBase64 = [system.convert]::ToBase64String($cert.RawData)
   ```

1. Chargez les informations de clé publique vers Azure. Une fois les informations de certificat chargées, Azure considère qu’il s’agit d’un certificat racine approuvé. Lors du chargement, assurez-vous d’exécuter PowerShell localement sur votre ordinateur, ou à la place, vous pouvez utiliser les [étapes du Portail Azure](vpn-gateway-howto-point-to-site-resource-manager-portal.md#uploadfile). Vous ne pouvez pas effectuer le chargement à l’aide d’Azure Cloud Shell.

   ```azurepowershell
   Add-AzVpnClientRootCertificate -VpnClientRootCertificateName $P2SRootCertName -VirtualNetworkGatewayname "VNet1GW" -ResourceGroupName "TestRG1" -PublicCertData $CertBase64
   ```

## <a name="8-install-an-exported-client-certificate"></a><a name="clientcertificate"></a>8. Installer un certificat client exporté

Les étapes suivantes vous permettent d’effectuer l’installation sur un client Windows. Pour obtenir des clients supplémentaires et plus d’informations, consultez [Installer un certificat client](point-to-site-how-to-vpn-client-install-azure-cert.md).

[!INCLUDE [Install on Windows](../../includes/vpn-gateway-certificates-install-client-cert-include.md)]

Assurez-vous que le certificat client a été exporté dans un fichier .pfx avec la totalité de la chaîne du certificat (qui est la valeur par défaut). Dans le cas contraire, les informations du certificat racine ne sont pas présentes sur l’ordinateur client et le client ne pourra pas s’authentifier correctement.

## <a name="9-configure-the-vpn-client"></a><a name="clientconfig"></a>9. Configurer le client VPN

Dans cette section, vous configurez le client natif pour que votre ordinateur se connecte à la passerelle de réseau virtuel. Par exemple, quand vous accédez aux paramètres VPN sur votre ordinateur Windows, vous pouvez ajouter des connexions VPN. Une connexion point à site nécessite des paramètres de configuration spécifiques. Ces étapes vous permettent de créer un package avec les paramètres spécifiques dont votre client VPN natif a besoin pour se connecter au réseau virtuel au moyen d’une connexion point à site.

Vous pouvez utiliser les exemples rapides suivants pour générer et installer le package de configuration du client. Pour obtenir plus d’informations sur le contenu d’un package et des instructions supplémentaires en vue de générer et d’installer les fichiers de configuration du client VPN, consultez [Créer et installer les fichiers de configuration du client VPN](point-to-site-vpn-client-configuration-azure-cert.md).

Si vous avez besoin de redéclarer vos variables, vous pouvez les trouver [ici](#declare).

### <a name="to-generate-configuration-files"></a>Pour générer des fichiers de configuration

```azurepowershell-interactive
$profile=New-AzVpnClientConfiguration -ResourceGroupName $RG -Name $GWName -AuthenticationMethod "EapTls"

$profile.VPNProfileSASUrl
```

### <a name="to-install-the-client-configuration-package"></a>Pour installer le package de configuration du client

[!INCLUDE [Windows instructions](../../includes/vpn-gateway-p2s-client-configuration-windows.md)]

## <a name="10-connect-to-azure"></a><a name="connect"></a>10. Connexion à Azure

### <a name="windows-vpn-client"></a>Client VPN Windows

[!INCLUDE [Connect from Windows client](../../includes/vpn-gateway-p2s-connect-windows-client.md)]

[!INCLUDE [Client certificates](../../includes/vpn-gateway-certificates-verify-client-cert-include.md)]

### <a name="mac-vpn-client"></a>Client VPN Mac

À partir de la boîte de dialogue Réseau, recherchez le profil de client que vous souhaitez utiliser, puis cliquez sur **Connexion**.
Pour obtenir des instructions détaillées, consultez la section [Installer - Mac (OS X)](./point-to-site-vpn-client-configuration-azure-cert.md#installmac). Si vous rencontrez des problèmes de connexion, vérifiez que la passerelle de réseau virtuel n’utilise pas une référence SKU de base. La référence SKU de base n’est pas prise en charge pour les clients Mac.

  ![Connexion Mac](./media/vpn-gateway-howto-point-to-site-rm-ps/applyconnect.png)

## <a name="to-verify-a-connection"></a><a name="verify"></a>Pour vérifier une connexion

Ces instructions s’appliquent aux clients Windows.

1. Pour vérifier que votre connexion VPN est active, ouvrez une invite de commandes avec élévation de privilèges, puis exécutez *ipconfig/all*.
2. Affichez les résultats. Notez que l’adresse IP que vous avez reçue est l’une des adresses du pool d’adresses de client VPN point à site que vous avez spécifiées dans votre configuration. Les résultats ressemblent à l’exemple qui suit :

   ```
   PPP adapter VNet1:
      Connection-specific DNS Suffix .:
      Description.....................: VNet1
      Physical Address................:
      DHCP Enabled....................: No
      Autoconfiguration Enabled.......: Yes
      IPv4 Address....................: 172.16.201.13(Preferred)
      Subnet Mask.....................: 255.255.255.255
      Default Gateway.................:
      NetBIOS over Tcpip..............: Enabled
   ```

## <a name="to-connect-to-a-virtual-machine"></a><a name="connectVM"></a>Se connecter à une machine virtuelle

Ces instructions s’appliquent aux clients Windows.

[!INCLUDE [Connect to a VM](../../includes/vpn-gateway-connect-vm.md)]

* Vérifiez que le package de configuration du client VPN a été généré après que les adresses IP du serveur DNS ont été spécifiées pour le réseau virtuel. Si vous avez mis à jour les adresses IP du serveur DNS, générez et installez un package de configuration du client VPN.

* Utilisez « ipconfig » pour vérifier l’adresse IPv4 attribuée à l’adaptateur Ethernet sur l’ordinateur à partir duquel vous vous connectez. Si l’adresse IP est comprise dans la plage d’adresses du réseau virtuel auquel vous vous connectez, ou dans la plage d’adresses de votre VPNClientAddressPool, cette situation est désignée sous le terme d’espaces d’adressage qui se chevauchent. Lorsque vos espaces d’adressage se chevauchent de cette façon, le trafic réseau n’atteint pas Azure et reste sur le réseau local.

## <a name="to-add-or-remove-a-root-certificate"></a><a name="addremovecert"></a>Pour ajouter ou supprimer un certificat racine

Vous pouvez ajouter et supprimer des certificats racines approuvés à partir d'Azure. Lorsque vous supprimez un certificat racine, les clients qui possèdent un certificat généré à partir de la racine ne peuvent pas s’authentifier, et ne seront donc pas en mesure de se connecter. Si vous souhaitez que des clients s’authentifient et se connectent, vous devez installer un nouveau certificat client généré à partir d’un certificat racine approuvé (téléchargé) dans Azure. Ces étapes nécessitent l’installation loacale d’applets de commande Azure PowerShell sur votre ordinateur (et non pas Azure Cloud Shell). Vous pouvez également utiliser le portail Azure pour ajouter des certificats racines.

**Pour ajouter :**

Vous pouvez ajouter jusqu’à 20 fichiers .cer de certificat racine dans Azure. La procédure suivante vous permet d’ajouter un certificat racine. 

1. Préparez le fichier .cer à télécharger :

   ```azurepowershell
   $filePathForCert = "C:\cert\P2SRootCert3.cer"
   $cert = new-object System.Security.Cryptography.X509Certificates.X509Certificate2($filePathForCert)
   $CertBase64_3 = [system.convert]::ToBase64String($cert.RawData)
   ```

1. Chargez le fichier. Vous ne pouvez charger qu’un seul fichier à la fois.

   ```azurepowershell
   Add-AzVpnClientRootCertificate -VpnClientRootCertificateName $P2SRootCertName -VirtualNetworkGatewayname "VNet1GW" -ResourceGroupName "TestRG1" -PublicCertData $CertBase64_3
   ```

1. Pour vérifier que le fichier de certificat est chargé :

   ```azurepowershell
   Get-AzVpnClientRootCertificate -ResourceGroupName "TestRG1" `
   -VirtualNetworkGatewayName "VNet1GW"
   ```

**Pour supprimer :**

1. Déclarez les variables. Modifiez les variables de l’exemple pour qu’elles correspondent au certificat que vous souhaitez supprimer.

   ```azurepowershell-interactive
   $GWName = "Name_of_virtual_network_gateway"
   $RG = "Name_of_resource_group"
   $P2SRootCertName2 = "ARMP2SRootCert2.cer"
   $MyP2SCertPubKeyBase64_2 = "MIIC/zCCAeugAwIBAgIQKazxzFjMkp9JRiX+tkTfSzAJBgUrDgMCHQUAMBgxFjAUBgNVBAMTDU15UDJTUm9vdENlcnQwHhcNMTUxMjE5MDI1MTIxWhcNMzkxMjMxMjM1OTU5WjAYMRYwFAYDVQQDEw1NeVAyU1Jvb3RDZXJ0MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAyjIXoWy8xE/GF1OSIvUaA0bxBjZ1PJfcXkMWsHPzvhWc2esOKrVQtgFgDz4ggAnOUFEkFaszjiHdnXv3mjzE2SpmAVIZPf2/yPWqkoHwkmrp6BpOvNVOpKxaGPOuK8+dql1xcL0eCkt69g4lxy0FGRFkBcSIgVTViS9wjuuS7LPo5+OXgyFkAY3pSDiMzQCkRGNFgw5WGMHRDAiruDQF1ciLNojAQCsDdLnI3pDYsvRW73HZEhmOqRRnJQe6VekvBYKLvnKaxUTKhFIYwuymHBB96nMFdRUKCZIiWRIy8Hc8+sQEsAML2EItAjQv4+fqgYiFdSWqnQCPf/7IZbotgQIDAQABo00wSzBJBgNVHQEEQjBAgBAkuVrWvFsCJAdK5pb/eoCNoRowGDEWMBQGA1UEAxMNTXlQMlNSb290Q2VydIIQKazxzFjMkp9JRiX+tkTfSzAJBgUrDgMCHQUAA4IBAQA223veAZEIar9N12ubNH2+HwZASNzDVNqspkPKD97TXfKHlPlIcS43TaYkTz38eVrwI6E0yDk4jAuPaKnPuPYFRj9w540SvY6PdOUwDoEqpIcAVp+b4VYwxPL6oyEQ8wnOYuoAK1hhh20lCbo8h9mMy9ofU+RP6HJ7lTqupLfXdID/XevI8tW6Dm+C/wCeV3EmIlO9KUoblD/e24zlo3YzOtbyXwTIh34T0fO/zQvUuBqZMcIPfM1cDvqcqiEFLWvWKoAnxbzckye2uk1gHO52d8AVL3mGiX8wBJkjc/pMdxrEvvCzJkltBmqxTM6XjDJALuVh16qFlqgTWCIcb7ju"
   ```

1. Supprimez le certificat.

   ```azurepowershell-interactive
   Remove-AzVpnClientRootCertificate -VpnClientRootCertificateName $P2SRootCertName2 -VirtualNetworkGatewayName $GWName -ResourceGroupName $RG -PublicCertData $MyP2SCertPubKeyBase64_2
   ```

1. Utilisez l'exemple suivant pour vérifier que le certificat a été supprimé avec succès.

   ```azurepowershell-interactive
   Get-AzVpnClientRootCertificate -ResourceGroupName "TestRG1" `
   -VirtualNetworkGatewayName "VNet1GW"
   ```

## <a name="to-revoke-or-reinstate-a-client-certificate"></a><a name="revoke"></a>Pour révoquer ou rétablir un certificat client

Vous pouvez révoquer des certificats clients. La liste de révocation de certificat vous permet de refuser sélectivement la connexion point à site en fonction des certificats clients individuels. Cela est différent de la suppression d’un certificat racine approuvé. Si vous supprimez un fichier .cer de certificat racine approuvé d’Azure, vous révoquez l’accès pour tous les certificats clients générés/signés par le certificat racine révoqué. Le fait de révoquer un certificat client plutôt que le certificat racine permet de continuer à utiliser les autres certificats générés à partir du certificat racine pour l’authentification.

La pratique courante consiste à utiliser le certificat racine pour gérer l'accès au niveaux de l'équipe ou de l'organisation, tout en utilisant des certificats clients révoqués pour le contrôle d'accès précis des utilisateurs individuels.

**Pour révoquer :**

1. Récupérez l’empreinte du certificat client. Pour plus d’informations, consultez l’article [Comment : récupérer l’empreinte numérique d’un certificat](/dotnet/framework/wcf/feature-details/how-to-retrieve-the-thumbprint-of-a-certificate).

1. Copiez les informations dans un éditeur de texte et supprimez tous les espaces afin d’obtenir une chaîne continue. Cette chaîne est déclarée sous la forme d’une variable à l’étape suivante.

1. Déclarez les variables. Veillez à déclarer l’empreinte numérique que vous avez récupérée à l’étape précédente.

   ```azurepowershell-interactive
   $RevokedClientCert1 = "NameofCertificate"
   $RevokedThumbprint1 = "‎51ab1edd8da4cfed77e20061c5eb6d2ef2f778c7"
   $GWName = "Name_of_virtual_network_gateway"
   $RG = "Name_of_resource_group"
   ```

1. Ajoutez l’empreinte numérique à la liste des certificats révoqués. Une fois l’empreinte numérique ajoutée, le message « Réussi » s’affiche.

   ```azurepowershell-interactive
   Add-AzVpnClientRevokedCertificate -VpnClientRevokedCertificateName $RevokedClientCert1 `
   -VirtualNetworkGatewayName $GWName -ResourceGroupName $RG `
   -Thumbprint $RevokedThumbprint1
   ```

1. Vérifiez que l'empreinte numérique a été ajoutée à la liste de révocation de certificats.

   ```azurepowershell-interactive
   Get-AzVpnClientRevokedCertificate -VirtualNetworkGatewayName $GWName -ResourceGroupName $RG
   ```

1. Après avoir ajouté l’empreinte numérique, le certificat ne peut plus être utilisé pour se connecter. Les clients qui tentent de se connecter à l’aide de ce certificat reçoivent un message indiquant que le certificat n’est plus valide.

**Pour rétablir :**

Vous pouvez réactiver un certificat client en supprimant l'empreinte numérique de la liste des certificats clients révoqués.

1. Déclarez les variables. Assurez-vous de déclarer la bonne empreinte numérique pour le certificat que vous souhaitez rétablir.

   ```azurepowershell-interactive
   $RevokedClientCert1 = "NameofCertificate"
   $RevokedThumbprint1 = "‎51ab1edd8da4cfed77e20061c5eb6d2ef2f778c7"
   $GWName = "Name_of_virtual_network_gateway"
   $RG = "Name_of_resource_group"
   ```

1. Supprimez l’empreinte numérique du certificat de la liste de révocation de certificats.

   ```azurepowershell-interactive
   Remove-AzVpnClientRevokedCertificate -VpnClientRevokedCertificateName $RevokedClientCert1 `
   -VirtualNetworkGatewayName $GWName -ResourceGroupName $RG -Thumbprint $RevokedThumbprint1
   ```

1. Vérifiez si l'empreinte numérique est supprimée de la liste de révocation.

   ```azurepowershell-interactive
   Get-AzVpnClientRevokedCertificate -VirtualNetworkGatewayName $GWName -ResourceGroupName $RG
   ```

## <a name="point-to-site-faq"></a><a name="faq"></a>Forum Aux Questions sur les connexions point à site

Pour plus d’informations, consultez les [questions fréquentes (FAQ) sur les passerelles VPN point à site](vpn-gateway-vpn-faq.md#P2S)

## <a name="next-steps"></a>Étapes suivantes

Une fois la connexion achevée, vous pouvez ajouter des machines virtuelles à vos réseaux virtuels. Pour plus d’informations, consultez [Machines virtuelles](../index.yml). Pour plus d’informations sur la mise en réseau et les machines virtuelles, consultez [Vue d’ensemble du réseau de machines virtuelles Azure et Linux](../virtual-machines/network-overview.md).

Pour plus d’informations sur la résolution des problèmes liés aux connexions de point à site, consultez l’article [Résolution des problèmes de connexion de point à site Azure](vpn-gateway-troubleshoot-vpn-point-to-site-connection-problems.md).