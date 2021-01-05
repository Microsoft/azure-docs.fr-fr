---
title: Configurer un VPN point à site (P2S) sous Linux pour une utilisation avec Azure Files | Microsoft Docs
description: Configuration d'un VPN point à site (P2S) sous Linux pour une utilisation avec Azure Files
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 10/19/2019
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 74422318718e318a00d7bd7ebaf8e4093ef75aa6
ms.sourcegitcommit: 9826fb9575dcc1d49f16dd8c7794c7b471bd3109
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2020
ms.locfileid: "94629272"
---
# <a name="configure-a-point-to-site-p2s-vpn-on-linux-for-use-with-azure-files"></a>Configurer un VPN point à site (P2S) sous Linux pour une utilisation avec Azure Files
Vous pouvez utiliser une connexion VPN point à site (P2S) pour monter vos partages de fichiers Azure sur SMB en dehors d’Azure, sans ouvrir le port 445. Une connexion VPN point à site est une connexion VPN entre Azure et un client individuel. Pour utiliser une connexion VPN P2S avec Azure Files, une connexion VPN P2S doit être configurée pour chaque client qui souhaite se connecter. Si de nombreux clients doivent se connecter à vos partages de fichiers Azure depuis votre réseau local, vous pouvez utiliser une connexion VPN site à site (S2S) au lieu d’une connexion point à site pour chaque client. Pour plus d’informations, consultez [Configurer un VPN site à site pour une utilisation avec Azure Files](storage-files-configure-s2s-vpn.md).

Avant de poursuivre cet article, nous vous recommandons vivement de lire l’article [Vue d’ensemble de la mise en réseau Azure Files](storage-files-networking-overview.md), qui fournit une présentation complète des options de mise en réseau disponibles pour Azure Files.

L'article décrit en détail la procédure à suivre pour configurer un VPN point à site sous Linux afin de monter des partages de fichiers Azure localement. Si vous souhaitez router le trafic Azure File Sync via un VPN, consultez [Configuration les paramètres de proxy et de pare-feu d’Azure File Sync](storage-sync-files-firewall-and-proxy.md).

## <a name="prerequisites"></a>Prérequis
- Version la plus récente de l'interface Azure CLI. Pour plus d'informations sur l'installation d'Azure CLI, consultez [Installer l'interface CLI Azure PowerShell](/cli/azure/install-azure-cli) et sélectionnez votre système d'exploitation. Si vous préférez utiliser le module Azure PowerShell sous Linux, rien ne vous en empêche. Sachez toutefois que les instructions ci-dessous s'appliquent à Azure CLI.

- Un partage de fichiers Azure que vous voulez monter localement. Les partages de fichiers Azure sont déployés sur des comptes de stockage. Ces comptes sont des constructions de gestion représentant un pool de stockage partagé dans lequel vous pouvez déployer plusieurs partages de fichiers, ainsi que d’autres ressources de stockage, telles que des conteneurs d’objets blob ou des files d’attente. Pour plus d’informations sur le déploiement des partages de fichiers et des comptes de stockage Azure, consultez [Créer un partage de fichiers Azure](storage-how-to-create-file-share.md).

- Un point de terminaison privé pour le compte de stockage sur lequel se trouve le partage de fichiers Azure que vous souhaitez monter localement. Pour plus d’informations sur la création d’un point de terminaison privé, consultez [Configuration des points de terminaison réseau Azure Files](storage-files-networking-endpoints.md?tabs=azure-cli). 

## <a name="install-required-software"></a>Installer les logiciels nécessaires
La passerelle de réseau virtuel Azure peut fournir des connexions VPN par le biais de plusieurs protocoles VPN, dont IPsec et OpenVPN. Ce guide explique comment utiliser IPsec et utilise le package strongSwan pour assurer la prise en charge sous Linux. 

> Vérifié avec Ubuntu 18.10.

```bash
sudo apt install strongswan strongswan-pki libstrongswan-extra-plugins curl libxml2-utils cifs-utils

installDir="/etc/"
```

### <a name="deploy-a-virtual-network"></a>Déployer un réseau virtuel 
Pour accéder à votre partage de fichiers Azure et à d’autres ressources Azure depuis un environnement local via un VPN point à site, vous devez créer un réseau virtuel. La connexion VPN point à site que vous allez créer automatiquement constitue un pont entre votre ordinateur Linux local et ce réseau virtuel Azure.

Le script suivant crée un réseau virtuel Azure doté de trois sous-réseaux : un pour le point de terminaison de service de votre compte de stockage, un pour le point de terminaison privé de votre compte de stockage (celui-ci est nécessaire pour accéder localement au compte de stockage sans créer de routage personnalisé pour l'adresse IP publique du compte de stockage, qui peut changer), et un pour votre passerelle de réseau virtuel qui fournit le service VPN. 

N’oubliez pas de remplacer `<region>`, `<resource-group>` et `<desired-vnet-name>` par les valeurs correspondant à votre environnement.

```bash
region="<region>"
resourceGroupName="<resource-group>"
virtualNetworkName="<desired-vnet-name>"

virtualNetwork=$(az network vnet create \
    --resource-group $resourceGroupName \
    --name $virtualNetworkName \
    --location $region \
    --address-prefixes "192.168.0.0/16" \
    --query "newVNet.id" | tr -d '"')

serviceEndpointSubnet=$(az network vnet subnet create \
    --resource-group $resourceGroupName \
    --vnet-name $virtualNetworkName \
    --name "ServiceEndpointSubnet" \
    --address-prefixes "192.168.0.0/24" \
    --service-endpoints "Microsoft.Storage" \
    --query "id" | tr -d '"')

privateEndpointSubnet=$(az network vnet subnet create \
    --resource-group $resourceGroupName \
    --vnet-name $virtualNetworkName \
    --name "PrivateEndpointSubnet" \
    --address-prefixes "192.168.1.0/24" \
    --query "id" | tr -d '"')

gatewaySubnet=$(az network vnet subnet create \
    --resource-group $resourceGroupName \
    --vnet-name $virtualNetworkName \
    --name "GatewaySubnet" \
    --address-prefixes "192.168.2.0/24" \
    --query "id" | tr -d '"')
```

## <a name="create-certificates-for-vpn-authentication"></a>Créer des certificats pour l'authentification VPN
Pour que les connexions VPN de vos ordinateurs Linux locaux soient authentifiées afin d'accéder à votre réseau virtuel, vous devez créer deux certificats : un certificat racine, qui sera fourni à la passerelle de machine virtuelle, et un certificat client, qui sera signé avec le certificat racine. Le script suivant crée les certificats requis.

```bash
rootCertName="P2SRootCert"
username="client"
password="1234"

mkdir temp
cd temp

sudo ipsec pki --gen --outform pem > rootKey.pem
sudo ipsec pki --self --in rootKey.pem --dn "CN=$rootCertName" --ca --outform pem > rootCert.pem

rootCertificate=$(openssl x509 -in rootCert.pem -outform der | base64 -w0 ; echo)

sudo ipsec pki --gen --size 4096 --outform pem > "clientKey.pem"
sudo ipsec pki --pub --in "clientKey.pem" | \
    sudo ipsec pki \
        --issue \
        --cacert rootCert.pem \
        --cakey rootKey.pem \
        --dn "CN=$username" \
        --san $username \
        --flag clientAuth \
        --outform pem > "clientCert.pem"

openssl pkcs12 -in "clientCert.pem" -inkey "clientKey.pem" -certfile rootCert.pem -export -out "client.p12" -password "pass:$password"
```

## <a name="deploy-virtual-network-gateway"></a>Déployer une passerelle de réseau virtuel
La passerelle de réseau virtuel Azure correspond au service auquel vos ordinateurs Linux locaux se connecteront. Le déploiement de ce service requiert deux composants de base : une adresse IP publique qui identifiera la passerelle auprès de vos clients où qu'ils se trouvent dans le monde, et un certificat racine que vous avez précédemment créé et qui sera utilisé pour authentifier vos clients.

N’oubliez pas de remplacer `<desired-vpn-name-here>` par le nom que vous souhaitez donner à ces ressources.

> [!Note]  
> Le déploiement de la passerelle de réseau virtuel Azure peut prendre jusqu’à 45 minutes. Pendant le déploiement de cette ressource, ce script bash sera bloqué pour permettre l'exécution du déploiement.
>
> Les connexions P2S IKEv2/OpenVPN ne sont pas prises en charge avec la référence SKU **De base**. Ce script utilise donc la référence SKU **VpnGw1** pour la passerelle de réseau virtuel.

```bash
vpnName="<desired-vpn-name-here>"
publicIpAddressName="$vpnName-PublicIP"

publicIpAddress=$(az network public-ip create \
    --resource-group $resourceGroupName \
    --name $publicIpAddressName \
    --location $region \
    --sku "Basic" \
    --allocation-method "Dynamic" \
    --query "publicIp.id" | tr -d '"')

az network vnet-gateway create \
    --resource-group $resourceGroupName \
    --name $vpnName \
    --vnet $virtualNetworkName \
    --public-ip-addresses $publicIpAddress \
    --location $region \
    --sku "VpnGw1" \
    --gateway-typ "Vpn" \
    --vpn-type "RouteBased" \
    --address-prefixes "172.16.201.0/24" \
    --client-protocol "IkeV2" > /dev/null

az network vnet-gateway root-cert create \
    --resource-group $resourceGroupName \
    --gateway-name $vpnName \
    --name $rootCertName \
    --public-cert-data $rootCertificate \
    --output none
```

## <a name="configure-the-vpn-client"></a>Configurer le client VPN
La passerelle de réseau virtuel Azure crée un package téléchargeable avec les fichiers de configuration nécessaires à l'initialisation de la connexion VPN sur votre ordinateur Linux local. Le script suivant place les certificats que vous avez créés à l'emplacement approprié et configure le fichier `ipsec.conf` avec les valeurs qui conviennent à partir du fichier de configuration contenu dans le package téléchargeable.

```bash
vpnClient=$(az network vnet-gateway vpn-client generate \
    --resource-group $resourceGroupName \
    --name $vpnName \
    --authentication-method EAPTLS | tr -d '"')

curl $vpnClient --output vpnClient.zip
unzip vpnClient.zip

vpnServer=$(xmllint --xpath "string(/VpnProfile/VpnServer)" Generic/VpnSettings.xml)
vpnType=$(xmllint --xpath "string(/VpnProfile/VpnType)" Generic/VpnSettings.xml | tr '[:upper:]' '[:lower:]')
routes=$(xmllint --xpath "string(/VpnProfile/Routes)" Generic/VpnSettings.xml)

sudo cp "${installDir}ipsec.conf" "${installDir}ipsec.conf.backup"
sudo cp "Generic/VpnServerRoot.cer" "${installDir}ipsec.d/cacerts"
sudo cp "${username}.p12" "${installDir}ipsec.d/private" 

echo -e "\nconn $virtualNetworkName" | sudo tee -a "${installDir}ipsec.conf" > /dev/null
echo -e "\tkeyexchange=$vpnType" | sudo tee -a "${installDir}ipsec.conf" > /dev/null
echo -e "\ttype=tunnel" | sudo tee -a "${installDir}ipsec.conf" > /dev/null
echo -e "\tleftfirewall=yes" | sudo tee -a "${installDir}ipsec.conf" > /dev/null
echo -e "\tleft=%any" | sudo tee -a "${installDir}ipsec.conf" > /dev/null
echo -e "\tleftauth=eap-tls" | sudo tee -a "${installDir}ipsec.conf" > /dev/null
echo -e "\tleftid=%client" | sudo tee -a "${installDir}ipsec.conf" > /dev/null
echo -e "\tright=$vpnServer" | sudo tee -a "${installDir}ipsec.conf" > /dev/null
echo -e "\trightid=%$vpnServer" | sudo tee -a "${installDir}ipsec.conf" > /dev/null
echo -e "\trightsubnet=$routes" | sudo tee -a "${installDir}ipsec.conf" > /dev/null
echo -e "\tleftsourceip=%config" | sudo tee -a "${installDir}ipsec.conf" > /dev/null 
echo -e "\tauto=add" | sudo tee -a "${installDir}ipsec.conf" > /dev/null

echo ": P12 client.p12 '$password'" | sudo tee -a "${installDir}ipsec.secrets" > /dev/null

sudo ipsec restart
sudo ipsec up $virtualNetworkName 
```

## <a name="mount-azure-file-share"></a>Monter le partage de fichiers Azure
Maintenant que vous avez configuré votre VPN point à site, vous pouvez monter votre partage de fichiers Azure. L'exemple suivant permet de monter le partage de manière non persistante. Pour un montage persistant, consultez [Utiliser un partage de fichiers Azure avec Linux](storage-how-to-use-files-linux.md). 

```bash
fileShareName="myshare"

mntPath="/mnt/$storageAccountName/$fileShareName"
sudo mkdir -p $mntPath

storageAccountKey=$(az storage account keys list \
    --resource-group $resourceGroupName \
    --account-name $storageAccountName \
    --query "[0].value" | tr -d '"')

smbPath="//$storageAccountPrivateIP/$fileShareName"
sudo mount -t cifs $smbPath $mntPath -o vers=3.0,username=$storageAccountName,password=$storageAccountKey,serverino
```

## <a name="see-also"></a>Voir aussi
- [Vue d’ensemble de la mise en réseau Azure Files](storage-files-networking-overview.md)
- [Configurer un VPN point à site (P2S) sous Windows pour une utilisation avec Azure Files](storage-files-configure-p2s-vpn-windows.md)
- [Configurer un VPN site à site (S2S) pour une utilisation avec Azure Files](storage-files-configure-s2s-vpn.md)