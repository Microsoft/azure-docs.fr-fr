---
title: Configurer un VPN sur votre appareil Azure Stack Edge Pro R à l’aide d’Azure PowerShell
description: Décrit comment configurer un VPN sur votre appareil Azure Stack Edge Pro R à l’aide d’un script Azure PowerShell pour créer des ressources Azure.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: article
ms.date: 10/23/2020
ms.author: alkohli
ms.openlocfilehash: 2139080367cdce9a5f018afab0970a7bd0e7504c
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96465326"
---
# <a name="configure-vpn-on-your-azure-stack-edge-pro-r-device-via-azure-powershell"></a>Configurer un VPN sur votre appareil Azure Stack Edge Pro R via Azure PowerShell

<!--[!INCLUDE [applies-to-r-skus](../../includes/azure-stack-edge-applies-to-r-sku.md)]-->

L’option VPN fournit une deuxième couche de chiffrement pour les données en mouvement via *TLS* entre votre appareil Azure Stack Edge Pro R et Azure. Vous pouvez configurer un VPN sur votre appareil Azure Stack Edge Pro R via le Portail Azure ou via Azure PowerShell.

Cet article décrit les étapes nécessaires à la configuration d’un VPN sur votre appareil Azure Stack Edge Pro R à l’aide d’un script Azure PowerShell pour créer la configuration dans le cloud.

## <a name="about-vpn-setup"></a>À propos de l’installation d’un VPN

Un réseau VPN intersite se compose d’une passerelle VPN Azure, d’un périphérique VPN local et d’un tunnel VPN IPsec S2S entre les deux. Le flux de travail type inclut les étapes suivantes :

- Configurez les prérequis.
- Configurez les ressources nécessaires sur Azure.
    - Créez et configurez une passerelle VPN Azure (passerelle de réseau virtuel).
    - Créez et configurez une passerelle de réseau local Azure qui représente votre réseau local et le périphérique VPN.
    - Créez et configurez une connexion VPN Azure entre la passerelle VPN Azure et la passerelle de réseau local.
    - Configurez le Pare-feu Azure et ajoutez des règles de réseau et d’application.
    - Créez une table de routage Azure et ajoutez des routes.
- Configurez le VPN dans l’interface utilisateur web locale de l’appareil. Configurez le périphérique VPN local représenté par la passerelle de réseau local pour établir le tunnel VPN S2S réel avec la passerelle VPN Azure.

Les étapes détaillées sont fournies dans les sections suivantes.

## <a name="configure-prerequisites"></a>Configuration préalable requise

1. Vous devez avoir accès à un appareil Azure Stack Edge Pro R installé conformément aux instructions fournies dans [Installer votre appareil Azure Stack Edge Pro R](azure-stack-edge-pro-r-deploy-install.md). Cet appareil sera utilisé comme appareil VPN local pour créer la connexion VPN avec Azure. 

2. Votre appareil VPN doit avoir une adresse IP publique statique (externe). Cette adresse ne doit pas être de type NAT.

3. Vous devez avoir accès à un abonnement Azure valide qui est activé pour le service Azure Stack Edge dans Azure. Utilisez cet abonnement pour créer une ressource correspondante dans Azure afin de gérer votre appareil Azure Stack Edge Pro R.  

4. Vous avez accès à un client Windows que vous utiliserez pour accéder à votre appareil Azure Stack Edge Pro R. Vous utiliserez ce client pour créer par programmation la configuration dans le cloud.

    1. Pour installer la version requise de PowerShell sur votre client Windows, exécutez les commandes suivantes :

        ```azurepowershell
        Install-Module -Name Az -AllowClobber -Scope CurrentUser 
        Import-Module Az.Accounts
        ```
    2. Pour vous connecter à votre compte Azure et à votre abonnement, exécutez les commandes suivantes :

        ```azurepowershell
        Connect-AzAccount 
        Set-AzContext -Subscription "<Your subscription name>"
        ```
        Indiquez le nom de l’abonnement Azure que vous utilisez avec votre appareil Azure Stack Edge Pro R pour configurer le VPN.

    3. [Téléchargez le script](https://aka.ms/ase-vpn-deployment) requis pour créer la configuration dans le cloud. Voici ce que va faire ce script :
        
        - Créer un réseau virtuel Azure et les sous-réseaux suivants : *GatewaySubnet* et *AzureFirewallSubnet*.
        - Créer et configurer une passerelle VPN Azure.
        - Créer et configurer une passerelle de réseau local Azure.
        - Créer et configurer une connexion VPN Azure entre la passerelle VPN Azure et la passerelle de réseau local.
        - Créer un Pare-feu Azure et ajouter des règles de réseau, des règles d’application.
        - Créer une table de routage Azure et ajouter des routes à celle-ci.


## <a name="use-the-script"></a>Utiliser le script

Tout d’abord, vous modifiez le fichier `parameters.json` pour entrer vos paramètres. Ensuite, vous exécutez le script à l’aide du fichier JSON modifié.

Chacune de ces étapes est abordée dans les sections suivantes.

### <a name="download-service-tags-file"></a>Télécharger le fichier d’étiquettes de service

Vous avez peut-être déjà un fichier `ServiceTags.json` dans le dossier où vous avez téléchargé le script. Si ce n’est pas le cas, vous pouvez télécharger le fichier d’étiquettes de service.

[!INCLUDE [azure-stack-edge-gateway-download-service-tags](../../includes/azure-stack-edge-gateway-download-service-tags.md)]

### <a name="modify-parameters-file"></a>Modifier le fichier de paramètres

La première étape consiste à modifier le fichier `parameters.json` et à enregistrer les modifications. 


Pour les ressources Azure que vous créez, vous devez fournir les noms suivants :

|Nom du paramètre  |Description  |
|---------|---------|
|virtualNetworks_vnet_name    | Nom du réseau virtuel Azure        |
|azureFirewalls_firewall_name     | Nom du Pare-feu Azure        |
|routeTables_routetable_name     | Nom de la table de routage Azure        |
|publicIPAddresses_VNGW_public_ip_name     | Nom de l’adresse IP publique de votre passerelle de réseau virtuel       |
|virtualNetworkGateways_VNGW_name    | Nom de la passerelle VPN Azure (passerelle de réseau virtuel)        |
|publicIPAddresses_firewall_public_ip_name     | Nom de l’adresse IP publique de votre Pare-feu Azure         |
|localNetworkGateways_LNGW_name    |Nom de la passerelle du réseau local Azure          |
|connections_vngw_lngw_name    | Nom de la connexion VPN Azure. Il s’agit de la connexion entre votre passerelle de réseau virtuel et la passerelle de réseau local.       |
|location     |Il s’agit de la région dans laquelle vous souhaitez créer votre réseau virtuel. Sélectionnez la même région que celle associée à votre appareil.         |

Les adresses IP et les espaces d’adressage suivants se rapportent aux ressources Azure créées, notamment le réseau virtuel et les sous-réseaux associés (default, firewall, GatewaySubnet).

|Nom du paramètre  |Description  |
|---------|---------|
|VnetIPv4AddressSpace    | Il s’agit de l’espace d’adressage associé à votre réseau virtuel.       |
|DefaultSubnetIPv4AddressSpace    |Il s’agit de l’espace d’adressage associé au sous-réseau `Default` de votre réseau virtuel.         |
|FirewallSubnetIPv4AddressSpace    |Il s’agit de l’espace d’adressage associé au sous-réseau `Firewall` de votre réseau virtuel.          |
|GatewaySubnetIPv4AddressSpace    |Il s’agit de l’espace d’adressage associé au `GatewaySubnet` de votre réseau virtuel.          |
|GatewaySubnetIPv4bgpPeeringAddress    | Il s’agit de l’adresse IP réservée à la communication BGP et qui est basée sur l’espace d’adressage associé au `GatewaySubnet` de votre réseau virtuel.          |

Les adresses IP et les espaces d’adressage suivants se rapportent au réseau local (où votre appareil Azure Stack Edge Pro R est déployé).

|Nom du paramètre  |Description  |
|---------|---------|
|CustomerNetworkAddressSpace    |  Il s’agit de l’espace d’adressage pour votre adresse IP privée.       |
|CustomerPublicNetworkAddressSpace    |  Il s’agit de l’espace d’adressage pour votre adresse IP publique.       |
|DbeIOTNetworkAddressSpace    |Cette adresse IP est réservée par le service IoT. Ne modifiez pas ce paramètre.        |
|AzureVPNsharedKey    |Cette clé partagée est utilisée lors de la création de la ressource de connexion VPN Azure. Cette clé est également fournie en tant que secret partagé VPN pendant la configuration VPN de l’interface utilisateur web locale.         |
|DBE-Gateway-ipaddress   | Adresse IP publique de votre appareil Azure Stack Edge Pro R. Celle-ci peut ne pas être connue et vous pouvez exécuter le script avec un espace réservé d’adresse IP. Modifiez ultérieurement la passerelle de réseau local avec l’adresse IP réelle.     |

#### <a name="caveats-to-keep-in-mind"></a>Tenez compte des avertissements suivants :

- Vous ne disposez pas de l’adresse IP de l’appareil Azure Stack Edge Pro R. Vous pouvez utiliser un espace réservé d’adresse IP pour créer votre ressource et modifier ultérieurement la passerelle de réseau local Azure pour affecter l’adresse IP réelle du périphérique et l’espace d’adressage du réseau local pour l’appareil.
- En fonction des instructions de l’IETF sur Internet Assigned Numbers Authority (IANA), utilisez un sous-réseau figurant entre 172.16.x.y et 172.24.z.a. Nous nous réservons les plages d’adresses IPv4 172.24 pour le réseau Azure.

### <a name="run-the-script"></a>Exécuter le script

Procédez comme suit pour utiliser le `parameters.json` modifié et exécuter le script afin de créer des ressources Azure.

1. Exécutez PowerShell ISE en tant qu’administrateur.

2. Basculez vers le répertoire où se trouve le script.

3. Exécutez le script.

    `.\AzDeployVpn.ps1 -Location <Location> -AzureAppRuleFilePath "appRule.json" -AzureIPRangesFilePath "<Service tag json file>"  -ResourceGroupName "<Resource group name>" -AzureDeploymentName "<Deployment name>" -NetworkRuleCollectionName "<Name for collection of network rules>" -Priority 115 -AppRuleCollectionName "<Name for collection of app rules>"`

    Voici un exemple de sortie obtenue.

    `.\AzDeployVpn.ps1 -Location eastus -AzureAppRuleFilePath "appRule.json" -AzureIPRangesFilePath "ServiceTags_Public_20191216.json"  -ResourceGroupName "devtestrg4" -AzureDeploymentName "dbetestdeployment20" -NetworkRuleCollectionName "testnrc20" -Priority 115 -AppRuleCollectionName "testarc20"`

    L’exécution du script prend environ 90 minutes. Une fois le script terminé, un journal de déploiement est généré dans le même dossier que celui où se trouve le script.


## <a name="verify-the-azure-resources"></a>Vérifier les ressources Azure

Une fois que vous avez exécuté le script avec succès, vérifiez que toutes les ressources ont été créées dans Azure.

Vous allez ensuite configurer le VPN sur l’interface utilisateur web locale de votre appareil.


## <a name="vpn-configuration-on-the-device"></a>Configuration du VPN sur l’appareil

[!INCLUDE [azure-stack-edge-gateway-configure-vpn-local-ui](../../includes/azure-stack-edge-gateway-configure-vpn-local-ui.md)]


## <a name="verify-vpn"></a>Vérifier le VPN

[!INCLUDE [azure-stack-edge-gateway-verify-vpn](../../includes/azure-stack-edge-gateway-verify-vpn.md)]

## <a name="validate-data-transfer-through-vpn"></a>Valider le transfert de données via VPN

Pour confirmer que le VPN fonctionne, copiez les données vers un partage SMB. Suivez les étapes décrites dans [Ajouter un partage](azure-stack-edge-j-series-manage-shares.md#add-a-share) sur votre appareil Azure Stack Edge Pro R. 

1. Copiez un fichier, par exemple \data\pictures\waterfall.jpg, dans le partage SMB que vous avez monté sur votre système client. 
2. Vérifiez que ce fichier s’affiche dans votre compte de stockage dans le cloud.

Pour vérifier que les données passent par un VPN :

1. Ouvrez la ressource de connexion présente dans le groupe de ressources. 

2. Vérifiez la valeur des données d’entrée et de sortie.
 
3. Ouvrez la passerelle de réseau virtuel dans votre groupe de ressources. Affichez les graphiques **Total des entrées du tunnel** et **Total des sorties du tunnel**.
 
## <a name="debug-issues"></a>Résoudre les problèmes

Pour déboguer des problèmes, utilisez les commandes suivantes :

```
Get-AzResourceGroupDeployment -DeploymentName $deploymentName -ResourceGroupName $ResourceGroupName
```

Voici un exemple de sortie obtenue :


```azurepowershell
PS C:\Projects\VPN\Azure-VpnDeployment> Get-AzResourceGroupDeployment -DeploymentName "aseprorvpnrg14_deployment" -ResourceGroupName "aseprorvpnrg14"


DeploymentName          : aseprorvpnrg14_deployment
ResourceGroupName       : aseprorvpnrg14
ProvisioningState       : Succeeded
Timestamp               : 10/21/2020 6:23:13 PM
Mode                    : Incremental
TemplateLink            :
Parameters              :
                          Name                                         Type                       Value
                          ===========================================  =========================  ==========
                          virtualNetworks_vnet_name                    String                     aseprorvpnrg14_vnet
                          azureFirewalls_firewall_name                 String                     aseprorvpnrg14_firewall
                          routeTables_routetable_name                  String                     aseprorvpnrg14_routetable
                          publicIPAddresses_VNGW_public_ip_name        String                     aseprorvpnrg14_vngwpublicip
                          virtualNetworkGateways_VNGW_name             String                     aseprorvpnrg14_vngw
                          publicIPAddresses_firewall_public_ip_name    String                     aseprorvpnrg14_fwpip
                          localNetworkGateways_LNGW_name               String                     aseprorvpnrg14_lngw
                          connections_vngw_lngw_name                   String                     aseprorvpnrg14_connection
                          location                                     String                     East US
                          vnetIPv4AddressSpace                         String                     172.24.0.0/16
                          defaultSubnetIPv4AddressSpace                String                     172.24.0.0/24
                          firewallSubnetIPv4AddressSpace               String                     172.24.1.0/24
                          gatewaySubnetIPv4AddressSpace                String                     172.24.2.0/24
                          gatewaySubnetIPv4bgpPeeringAddress           String                     172.24.2.254
                          customerNetworkAddressSpace                  String                     10.0.0.0/18
                          customerPublicNetworkAddressSpace            String                     207.68.128.0/24
                          dbeIOTNetworkAddressSpace                    String                     10.139.218.0/24
                          azureVPNsharedKey                            String                     1234567890
                          dbE-Gateway-ipaddress                        String                     207.68.128.113

Outputs                 :
                          Name                     Type                       Value
                          =======================  =========================  ==========
                          virtualNetwork           Object                     {
                            "provisioningState": "Succeeded",
                            "resourceGuid": "dcf673d3-5c73-4764-b077-77125eda1303",
                            "addressSpace": {
                              "addressPrefixes": [
                                "172.24.0.0/16"
                              ]
================= CUT ============================= CUT ===========================
```


```
Get-AzResourceGroupDeploymentOperation -ResourceGroupName $ResourceGroupName -DeploymentName $AzureDeploymentName
```

## <a name="next-steps"></a>Étapes suivantes

[Configurer un VPN via l’interface utilisateur locale sur votre appareil Azure Stack Edge Pro R](azure-stack-edge-pro-r-deploy-configure-certificates-vpn-encryption.md#configure-vpn)
