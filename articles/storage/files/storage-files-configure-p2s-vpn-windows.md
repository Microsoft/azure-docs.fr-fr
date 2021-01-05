---
title: Configurer un VPN point à site (P2S) sur Windows pour l’utiliser avec Azure Files | Microsoft Docs
description: Guide pratique pour configurer un VPN point à site (P2S) sur Windows pour une utilisation avec Azure Files
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 10/19/2019
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 6253deb53229172cd499a6aa14b8d8f19bc07b63
ms.sourcegitcommit: 9826fb9575dcc1d49f16dd8c7794c7b471bd3109
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2020
ms.locfileid: "94629255"
---
# <a name="configure-a-point-to-site-p2s-vpn-on-windows-for-use-with-azure-files"></a>Configurer un VPN point à site (P2S) sur Windows pour l’utiliser avec Azure Files
Vous pouvez utiliser une connexion VPN point à site (P2S) pour monter vos partages de fichiers Azure sur SMB en dehors d’Azure, sans ouvrir le port 445. Une connexion VPN point à site est une connexion VPN entre Azure et un client individuel. Pour utiliser une connexion VPN P2S avec Azure Files, une connexion VPN P2S doit être configurée pour chaque client qui souhaite se connecter. Si de nombreux clients doivent se connecter à vos partages de fichiers Azure depuis votre réseau local, vous pouvez utiliser une connexion VPN site à site (S2S) au lieu d’une connexion point à site pour chaque client. Pour plus d’informations, consultez [Configurer un VPN site à site pour une utilisation avec Azure Files](storage-files-configure-s2s-vpn.md).

Nous vous recommandons fortement de lire [Considérations sur le réseau pour l’accès direct à un partage de fichiers Azure](storage-files-networking-overview.md) avant de poursuivre avec cet article de procédure pour une présentation complète des options réseau disponibles pour Azure Files.

L’article décrit en détail la procédure à suivre pour configurer un VPN point à site sur Windows (client Windows et Windows Server) afin de monter directement des partages de fichiers Azure locaux. Si vous souhaitez router le trafic Azure File Sync via un VPN, consultez [Configuration les paramètres de proxy et de pare-feu d’Azure File Sync](storage-sync-files-firewall-and-proxy.md).

## <a name="prerequisites"></a>Prérequis
- La version la plus récente du module Azure PowerShell. Pour plus d’informations sur l’installation d’Azure PowerShell, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps) et sélectionnez votre système d’exploitation. Si vous préférez utiliser Azure CLI sur Windows, vous pouvez le faire ; les instructions ci-dessous s’appliquent cependant à Azure PowerShell.

- Un partage de fichiers Azure que vous voulez monter localement. Les partages de fichiers Azure sont déployés sur des comptes de stockage. Ces comptes sont des constructions de gestion représentant un pool de stockage partagé dans lequel vous pouvez déployer plusieurs partages de fichiers, ainsi que d’autres ressources de stockage, telles que des conteneurs d’objets blob ou des files d’attente. Pour plus d’informations sur le déploiement des partages de fichiers et des comptes de stockage Azure, consultez [Créer un partage de fichiers Azure](storage-how-to-create-file-share.md).

- Un point de terminaison privé pour le compte de stockage sur lequel se trouve le partage de fichiers Azure que vous souhaitez monter localement. Pour plus d’informations sur la création d’un point de terminaison privé, consultez [Configuration des points de terminaison réseau Azure Files](storage-files-networking-endpoints.md?tabs=azure-powershell). 

## <a name="deploy-a-virtual-network"></a>Déployer un réseau virtuel
Pour accéder à votre partage de fichiers Azure et à d’autres ressources Azure depuis un environnement local via un VPN point à site, vous devez créer un réseau virtuel. La connexion VPN point à site que vous allez créer automatiquement constitue un pont entre votre machine Windows locale et ce réseau virtuel Azure.

Le script PowerShell suivant crée un réseau virtuel Azure avec trois sous-réseaux : un pour le point de terminaison de service de votre compte de stockage, un pour le point de terminaison privé de votre compte de stockage (celui-ci est nécessaire pour accéder localement au compte de stockage sans créer de routage personnalisé pour l’adresse IP publique du compte de stockage, qui peut changer), et un pour votre passerelle de réseau virtuel qui fournit le service VPN. 

N’oubliez pas de remplacer `<region>`, `<resource-group>` et `<desired-vnet-name>` par les valeurs correspondant à votre environnement.

```PowerShell
$region = "<region>"
$resourceGroupName = "<resource-group>" 
$virtualNetworkName = "<desired-vnet-name>"

$virtualNetwork = New-AzVirtualNetwork `
    -ResourceGroupName $resourceGroupName `
    -Name $virtualNetworkName `
    -Location $region `
    -AddressPrefix "192.168.0.0/16"

Add-AzVirtualNetworkSubnetConfig `
    -Name "ServiceEndpointSubnet" `
    -AddressPrefix "192.168.0.0/24" `
    -VirtualNetwork $virtualNetwork `
    -ServiceEndpoint "Microsoft.Storage" `
    -WarningAction SilentlyContinue | Out-Null

Add-AzVirtualNetworkSubnetConfig `
    -Name "PrivateEndpointSubnet" `
    -AddressPrefix "192.168.1.0/24" `
    -VirtualNetwork $virtualNetwork `
    -WarningAction SilentlyContinue | Out-Null

Add-AzVirtualNetworkSubnetConfig `
    -Name "GatewaySubnet" `
    -AddressPrefix "192.168.2.0/24" `
    -VirtualNetwork $virtualNetwork `
    -WarningAction SilentlyContinue | Out-Null

$virtualNetwork | Set-AzVirtualNetwork | Out-Null
$virtualNetwork = Get-AzVirtualNetwork `
    -ResourceGroupName $resourceGroupName `
    -Name $virtualNetworkName

$serviceEndpointSubnet = $virtualNetwork.Subnets | `
    Where-Object { $_.Name -eq "ServiceEndpointSubnet" }
$privateEndpointSubnet = $virtualNetwork.Subnets | `
    Where-Object { $_.Name -eq "PrivateEndpointSubnet" }
$gatewaySubnet = $virtualNetwork.Subnets | ` 
    Where-Object { $_.Name -eq "GatewaySubnet" }
```

## <a name="create-root-certificate-for-vpn-authentication"></a>Créer un certificat racine pour l’authentification VPN
Pour que les connexions VPN de vos ordinateurs Windows locaux soient authentifiées afin d’accéder à votre réseau virtuel, vous devez créer deux certificats : un certificat racine, qui sera fourni à la passerelle de machine virtuelle, et un certificat client, qui sera signé avec le certificat racine. Le code PowerShell suivant crée le certificat racine ; le certificat client est créé après la création de la passerelle de réseau virtuel Azure avec les informations de la passerelle. 

```PowerShell
$rootcertname = "CN=P2SRootCert"
$certLocation = "Cert:\CurrentUser\My"
$vpnTemp = "C:\vpn-temp\"
$exportedencodedrootcertpath = $vpnTemp + "P2SRootCertencoded.cer"
$exportedrootcertpath = $vpnTemp + "P2SRootCert.cer"

if (-Not (Test-Path $vpnTemp)) {
    New-Item -ItemType Directory -Force -Path $vpnTemp | Out-Null
}

if ($PSVersionTable.PSVersion -ge [System.Version]::new(6, 0)) {
    Install-Module WindowsCompatibility
    Import-WinModule PKI
}

$rootcert = New-SelfSignedCertificate `
    -Type Custom `
    -KeySpec Signature `
    -Subject $rootcertname `
    -KeyExportPolicy Exportable `
    -HashAlgorithm sha256 `
    -KeyLength 2048 `
    -CertStoreLocation $certLocation `
    -KeyUsageProperty Sign `
    -KeyUsage CertSign

Export-Certificate `
    -Cert $rootcert `
    -FilePath $exportedencodedrootcertpath `
    -NoClobber | Out-Null

certutil -encode $exportedencodedrootcertpath $exportedrootcertpath | Out-Null

$rawRootCertificate = Get-Content -Path $exportedrootcertpath

[System.String]$rootCertificate = ""
foreach($line in $rawRootCertificate) { 
    if ($line -notlike "*Certificate*") { 
        $rootCertificate += $line 
    } 
}
```

## <a name="deploy-virtual-network-gateway"></a>Déployer une passerelle de réseau virtuel
La passerelle de réseau virtuel Azure est le service auquel vos machines Windows locales se connecteront. Le déploiement de ce service nécessite deux composants de base : une adresse IP publique qui identifiera la passerelle auprès de vos clients où qu’ils se trouvent dans le monde, et un certificat racine que vous avez précédemment créé et qui sera utilisé pour authentifier vos clients.

N’oubliez pas de remplacer `<desired-vpn-name-here>` par le nom que vous souhaitez donner à ces ressources.

> [!Note]  
> Le déploiement de la passerelle de réseau virtuel Azure peut prendre jusqu’à 45 minutes. Pendant le déploiement de cette ressource, ce script PowerShell est bloqué pour permettre l’exécution du déploiement. Ceci est normal.

```PowerShell
$vpnName = "<desired-vpn-name-here>" 
$publicIpAddressName = "$vpnName-PublicIP"

$publicIPAddress = New-AzPublicIpAddress `
    -ResourceGroupName $resourceGroupName `
    -Name $publicIpAddressName `
    -Location $region `
    -Sku Basic `
    -AllocationMethod Dynamic

$gatewayIpConfig = New-AzVirtualNetworkGatewayIpConfig `
    -Name "vnetGatewayConfig" `
    -SubnetId $gatewaySubnet.Id `
    -PublicIpAddressId $publicIPAddress.Id

$azRootCertificate = New-AzVpnClientRootCertificate `
    -Name "P2SRootCert" `
    -PublicCertData $rootCertificate

$vpn = New-AzVirtualNetworkGateway `
    -ResourceGroupName $resourceGroupName `
    -Name $vpnName `
    -Location $region `
    -GatewaySku VpnGw1 `
    -GatewayType Vpn `
    -VpnType RouteBased `
    -IpConfigurations $gatewayIpConfig `
    -VpnClientAddressPool "172.16.201.0/24" `
    -VpnClientProtocol IkeV2 `
    -VpnClientRootCertificates $azRootCertificate
```

## <a name="create-client-certificate"></a>Créer un certificat client
Le certificat client est créé avec l’URI de la passerelle de réseau virtuel. Ce certificat est signé avec le certificat racine que vous avez créé précédemment.

```PowerShell
$clientcertpassword = "1234"

$vpnClientConfiguration = New-AzVpnClientConfiguration `
    -ResourceGroupName $resourceGroupName `
    -Name $vpnName `
    -AuthenticationMethod EAPTLS

Invoke-WebRequest `
    -Uri $vpnClientConfiguration.VpnProfileSASUrl `
    -OutFile "$vpnTemp\vpnclientconfiguration.zip"

Expand-Archive `
    -Path "$vpnTemp\vpnclientconfiguration.zip" `
    -DestinationPath "$vpnTemp\vpnclientconfiguration"

$vpnGeneric = "$vpnTemp\vpnclientconfiguration\Generic"
$vpnProfile = ([xml](Get-Content -Path "$vpnGeneric\VpnSettings.xml")).VpnProfile

$exportedclientcertpath = $vpnTemp + "P2SClientCert.pfx"
$clientcertname = "CN=" + $vpnProfile.VpnServer

$clientcert = New-SelfSignedCertificate `
    -Type Custom `
    -DnsName $vpnProfile.VpnServer `
    -KeySpec Signature `
    -Subject $clientcertname `
    -KeyExportPolicy Exportable `
    -HashAlgorithm sha256 `
    -KeyLength 2048 `
    -CertStoreLocation $certLocation `
    -Signer $rootcert `
    -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.2")

$mypwd = ConvertTo-SecureString -String $clientcertpassword -Force -AsPlainText

Export-PfxCertificate `
    -FilePath $exportedclientcertpath `
    -Password $mypwd `
    -Cert $clientcert | Out-Null
```

## <a name="configure-the-vpn-client"></a>Configurer le client VPN
La passerelle de réseau virtuel Azure crée un package téléchargeable avec les fichiers de configuration nécessaires à l’initialisation de la connexion VPN sur votre machine Windows locale. Nous allons configurer la connexion VPN avec la fonctionnalité [VPN Always On](/windows-server/remote/remote-access/vpn/always-on-vpn/) de Windows 10/Windows Server 2016+. Ce package contient également des packages exécutables qui vont configurer si vous le souhaitez le client VPN Windows hérité. Ce guide utilise VPN Always On au lieu du client VPN Windows hérité, car le client VPN Always On permet aux utilisateurs finaux de se connecter au VPN Azure et de s’en déconnecter sans avoir d’autorisations d’administrateur sur leur machine. 

Le script suivant installe le certificat client nécessaire pour l’authentification auprès de la passerelle de réseau virtuel, et télécharge et installe le package VPN. N’oubliez pas de remplacer `<computer1>` et `<computer2>` par les ordinateurs appropriés. Vous pouvez exécuter ce script sur autant de machines que vous le souhaitez en ajoutant des sessions PowerShell au tableau `$sessions`. Votre compte d’utilisation doit être un administrateur sur chacune de ces machines. Si une de ces machines est la machine locale à partir de laquelle vous exécutez le script, vous devez exécuter le script à partir d’une session PowerShell avec élévation de privilèges. 

```PowerShell
$sessions = [System.Management.Automation.Runspaces.PSSession[]]@()
$sessions += New-PSSession -ComputerName "<computer1>"
$sessions += New-PSSession -ComputerName "<computer2>"

foreach ($session in $sessions) {
    Invoke-Command -Session $session -ArgumentList $vpnTemp -ScriptBlock { 
        $vpnTemp = $args[0]
        if (-Not (Test-Path $vpnTemp)) {
            New-Item `
                -ItemType Directory `
                -Force `
                -Path "C:\vpn-temp" | Out-Null
        }
    }

    Copy-Item `
        -Path $exportedclientcertpath, $exportedrootcertpath, "$vpnTemp\vpnclientconfiguration.zip" `
        -Destination $vpnTemp `
        -ToSession $session

    Invoke-Command `
        -Session $session `
        -ArgumentList `
            $mypwd, `
            $vpnTemp, `
            $virtualNetworkName `
        -ScriptBlock { 
            $mypwd = $args[0] 
            $vpnTemp = $args[1]
            $virtualNetworkName = $args[2]

            Import-PfxCertificate `
                -Exportable `
                -Password $mypwd `
                -CertStoreLocation "Cert:\LocalMachine\My" `
                -FilePath "$vpnTemp\P2SClientCert.pfx" | Out-Null

            Import-Certificate `
                -FilePath "$vpnTemp\P2SRootCert.cer" `
                -CertStoreLocation "Cert:\LocalMachine\Root" | Out-Null

            Expand-Archive `
                -Path "$vpnTemp\vpnclientconfiguration.zip" `
                -DestinationPath "$vpnTemp\vpnclientconfiguration"
            $vpnGeneric = "$vpnTemp\vpnclientconfiguration\Generic"

            $vpnProfile = ([xml](Get-Content -Path "$vpnGeneric\VpnSettings.xml")).VpnProfile

            Add-VpnConnection `
                -Name $virtualNetworkName `
                -ServerAddress $vpnProfile.VpnServer `
                -TunnelType Ikev2 `
                -EncryptionLevel Required `
                -AuthenticationMethod MachineCertificate `
                -SplitTunneling `
                -AllUserConnection

            Add-VpnConnectionRoute `
                -Name $virtualNetworkName `
                -DestinationPrefix $vpnProfile.Routes `
                -AllUserConnection

            Add-VpnConnectionRoute `
                -Name $virtualNetworkName `
                -DestinationPrefix $vpnProfile.VpnClientAddressPool `
                -AllUserConnection

            rasdial $virtualNetworkName
        }
}

Remove-Item -Path $vpnTemp -Recurse
```

## <a name="mount-azure-file-share"></a>Monter le partage de fichiers Azure
Maintenant que vous avez configuré votre VPN de point à site, vous pouvez l’utiliser pour monter le partage de fichiers Azure sur les ordinateurs que vous configurez via PowerShell. L’exemple suivant va monter le partage, lister le contenu du répertoire racine du partage pour prouver que le partage est réellement, puis démonter le partage. Malheureusement, il n’est pas possible de monter le partage de façon persistante via PowerShell à distance. Pour un montage persistant, consultez [Utiliser un partage de fichiers Azure avec Windows](storage-how-to-use-files-windows.md). 

```PowerShell
$myShareToMount = "<file-share>"

$storageAccountKeys = Get-AzStorageAccountKey `
    -ResourceGroupName $resourceGroupName `
    -Name $storageAccountName
$storageAccountKey = ConvertTo-SecureString `
    -String $storageAccountKeys[0].Value `
    -AsPlainText `
    -Force

$nic = Get-AzNetworkInterface -ResourceId $privateEndpoint.NetworkInterfaces[0].Id
$storageAccountPrivateIP = $nic.IpConfigurations[0].PrivateIpAddress

Invoke-Command `
    -Session $sessions `
    -ArgumentList  `
        $storageAccountName, `
        $storageAccountKey, `
        $storageAccountPrivateIP, `
        $myShareToMount `
    -ScriptBlock {
        $storageAccountName = $args[0]
        $storageAccountKey = $args[1]
        $storageAccountPrivateIP = $args[2]
        $myShareToMount = $args[3]

        $credential = [System.Management.Automation.PSCredential]::new(
            "AZURE\$storageAccountName", 
            $storageAccountKey)

        New-PSDrive `
            -Name Z `
            -PSProvider FileSystem `
            -Root "\\$storageAccountPrivateIP\$myShareToMount" `
            -Credential $credential `
            -Persist | Out-Null
        Get-ChildItem -Path Z:\
        Remove-PSDrive -Name Z
    }
```

## <a name="see-also"></a>Voir aussi
- [Considérations réseau pour l’accès direct à un partage réseau Azure](storage-files-networking-overview.md)
- [Configurer un VPN point à site (P2S) sous Linux pour une utilisation avec Azure Files](storage-files-configure-p2s-vpn-linux.md)
- [Configurer un VPN site à site (S2S) pour une utilisation avec Azure Files](storage-files-configure-s2s-vpn.md)