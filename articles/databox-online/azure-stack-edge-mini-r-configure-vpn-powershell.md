---
title: Configurer un VPN sur votre appareil Azure Stack Edge Mini R à l’aide d’Azure PowerShell
description: Décrit comment configurer un VPN sur votre appareil Azure Stack Edge Mini R à l’aide d’un script Azure PowerShell pour créer des ressources Azure.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: article
ms.date: 11/17/2020
ms.author: alkohli
ms.openlocfilehash: 763ccd397d8cd704ca161032e65f17979bccb53b
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96465512"
---
# <a name="configure-vpn-on-your-azure-stack-edge-mini-r-device-via-azure-powershell"></a>Configurer un VPN sur votre appareil Azure Stack Edge Mini R via Azure PowerShell

<!--[!INCLUDE [applies-to-r-skus](../../includes/azure-stack-edge-applies-to-r-sku.md)]-->

L’option VPN fournit une deuxième couche de chiffrement pour les données en mouvement via *TLS* entre votre appareil Azure Stack Edge Mini R ou Azure Stack Edge Pro R et Azure. Vous pouvez configurer un VPN sur votre appareil Azure Stack Edge Mini R via le Portail Azure ou via Azure PowerShell. 

Cet article décrit les étapes nécessaires à la configuration d’un VPN de point à site (P2S) sur votre appareil Azure Stack Edge Mini R à l’aide d’un script Azure PowerShell pour créer la configuration dans le cloud. La configuration sur l’appareil Azure Stack Edge est effectuée via l’interface utilisateur locale.

## <a name="about-vpn-setup"></a>À propos de l’installation d’un VPN

Une connexion par passerelle VPN P2S vous permet de créer une connexion sécurisée à votre réseau virtuel à partir d’un ordinateur client individuel ou votre appareil Azure Stack Edge Mini R. Vous démarrez la connexion P2S à partir de l’ordinateur client ou de l’appareil. Dans ce cas, la connexion P2S utilise le VPN IKEv2, une solution VPN IPsec basée sur des normes.

Le flux de travail type inclut les étapes suivantes :

1. Configurer les prérequis.
2. Configurez les ressources nécessaires sur Azure.
    1. Créez et configurez un réseau virtuel et les sous-réseaux requis. 
    2. Créez et configurez une passerelle VPN Azure (passerelle de réseau virtuel).
    3. Configurez le Pare-feu Azure et ajoutez des règles de réseau et d’application.
    4. Créez des tables de routage Azure et ajoutez des routes.
    5. Activez le point à site dans la passerelle VPN.
        1. Ajoutez le pool d’adresses des clients.
        2. Configurez le type de tunnel.
        3. Configurez le type d’authentification.
        4. Créez un certificat.
        5. Chargez un certificat.
    6. Téléchargez l’annuaire téléphonique.
3. Configurez le VPN dans l’interface utilisateur web locale de l’appareil. 
    1. Fournissez l’annuaire téléphonique.
    2. Fournissez le fichier de balises de service (json).


Les étapes détaillées sont fournies dans les sections suivantes.

## <a name="configure-prerequisites"></a>Configuration préalable requise

- Vous devez avoir accès à un appareil Azure Stack Edge Mini R installé conformément aux instructions fournies dans [Installer votre appareil Azure Stack Edge Mini R](azure-stack-edge-mini-r-deploy-install.md). Ce périphérique établira une connexion P2S avec Azure. 

- Vous devez avoir accès à un abonnement Azure valide qui est activé pour le service Azure Stack Edge dans Azure. Utilisez cet abonnement pour créer une ressource correspondante dans Azure afin de gérer votre appareil Azure Stack Edge Mini R.  

- Vous avez accès à un client Windows que vous utiliserez pour accéder à votre appareil Azure Stack Edge Mini R. Vous utiliserez ce client pour créer par programmation la configuration dans le cloud.

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
        Indiquez le nom de l’abonnement Azure que vous utilisez avec votre appareil Azure Stack Edge Mini R pour configurer le VPN.

    3. [Téléchargez le script](https://aka.ms/ase-vpn-deployment) requis pour créer la configuration dans le cloud. Voici ce que va faire ce script :
        
        - Créer un réseau virtuel Azure et les sous-réseaux suivants : *GatewaySubnet* et *AzureFirewallSubnet*.
        - Créer et configurer une passerelle VPN Azure.
        - Créer et configurer une passerelle de réseau local Azure.
        - Créer et configurer une connexion VPN Azure entre la passerelle VPN Azure et la passerelle de réseau local.
        - Créer un Pare-feu Azure et ajouter des règles de réseau, des règles d’application.
        - Créer une table de routage Azure et ajouter des itinéraires à celle-ci.

    4. Créer le groupe de ressources dans le Portail Azure sous lequel vous souhaitez créer les ressources Azure. Accédez à la liste des services dans Portail Azure, sélectionnez **Groupe de ressources**, puis sélectionnez **+ Ajouter**. Fournissez les informations d’abonnement et le nom de votre groupe de ressources, puis sélectionnez **Créer**. Si vous accédez à ce groupe de ressources, il ne doit pas contenir de ressources pour l’instant.

        ![Groupe de ressources Azure](media/azure-stack-edge-mini-r-configure-vpn-powershell/azure-resource-group-1.png)
    
    5. Vous devez disposer d’un certificat encodé base 64 au format `.cer` pour votre appareil Azure Stack Edge Mini R. Ce certificat doit être chargé sur votre appareil Azure Stack Edge comme `pfx` avec une clé privée. Ce certificat doit également être installé à la racine de confiance du magasin sur le client qui essaie d’établir la connexion P2S.

## <a name="use-the-script"></a>Utiliser le script

Tout d’abord, vous modifiez le fichier `parameters-p2s.json` pour entrer vos paramètres. Ensuite, vous exécutez le script à l’aide du fichier JSON modifié.

Chacune de ces étapes est abordée dans les sections suivantes.

### <a name="download-service-tags-file"></a>Télécharger le fichier d’étiquettes de service

Vous avez peut-être déjà un fichier `ServiceTags.json` dans le dossier où vous avez téléchargé le script. Si ce n’est pas le cas, vous pouvez télécharger le fichier d’étiquettes de service.

[!INCLUDE [azure-stack-edge-gateway-download-service-tags](../../includes/azure-stack-edge-gateway-download-service-tags.md)]

### <a name="modify-parameters-file"></a>Modifier le fichier de paramètres

La première étape consiste à modifier le fichier `parameters-p2s.json` et à enregistrer les modifications. 

Pour les ressources Azure que vous créez, vous devez fournir les noms suivants :

|Nom du paramètre  |Description  |
|---------|---------|
|virtualNetworks_vnet_name    | Nom du réseau virtuel Azure        |
|azureFirewalls_firewall_name     | Nom du Pare-feu Azure        |
|routeTables_routetable_name     | Nom de la table de routage Azure        |
|publicIPAddresses_VNGW_public_ip_name     | Nom de l’adresse IP publique de votre passerelle de réseau virtuel       |
|virtualNetworkGateways_VNGW_name    | Nom de la passerelle VPN Azure (passerelle de réseau virtuel)        |
|publicIPAddresses_firewall_public_ip_name     | Nom de l’adresse IP publique de votre Pare-feu Azure         |
|location     |Il s’agit de la région dans laquelle vous souhaitez créer votre réseau virtuel. Sélectionnez la même région que celle associée à votre appareil.         |
|RouteTables_routetable_onprem_name| Il s’agit du nom de la table de routage supplémentaire permettant au pare-feu d’acheminer les paquets vers l’appareil Azure Stack Edge. Le script crée deux itinéraires supplémentaires et associe *default* et *FirewallSubnet* à cette table de routage.|

Fournissez les adresses IP et les espaces d’adressage suivants pour les ressources Azure créées, y compris le réseau virtuel et les sous-réseaux associés (*default*, *firewall*, *GatewaySubnet*).

|Nom du paramètre  |Description  |
|---------|---------|
|VnetIPv4AddressSpace    | Il s’agit de l’espace d’adressage associé à votre réseau virtuel. Fournissez une plage d’adresses IP de réseau virtuel en tant que plage d’adresses IP privées (https://en.wikipedia.org/wiki/Private_network#Private_IPv4_addresses).     |
|DefaultSubnetIPv4AddressSpace    |Il s’agit de l’espace d’adressage associé au sous-réseau `Default` de votre réseau virtuel.         |
|FirewallSubnetIPv4AddressSpace    |Il s’agit de l’espace d’adressage associé au sous-réseau `Firewall` de votre réseau virtuel.          |
|GatewaySubnetIPv4AddressSpace    |Il s’agit de l’espace d’adressage associé au `GatewaySubnet` de votre réseau virtuel.          |
|GatewaySubnetIPv4bgpPeeringAddress    | Il s’agit de l’adresse IP réservée à la communication BGP et qui est basée sur l’espace d’adressage associé au `GatewaySubnet` de votre réseau virtuel.          |
|ClientAddressPool    | Cette adresse IP est utilisée pour le pool d’adresses dans la configuration P2S dans le Portail Azure.         |
|PublicCertData     | Les données de certificat publiques sont utilisées par la passerelle VPN pour authentifier les clients P2S qui se connectent à celle-ci. Pour récupérer les données de certificat, installez le certificat racine. Assurez-vous que le certificat est en base-64 encodé avec une extension. cer. Ouvrez ce certificat et copiez le texte dans le certificat entre ==BEGIN CERTIFICATE== et ==END CERTIFICATE== en une ligne continue.     |



### <a name="run-the-script"></a>Exécuter le script

Procédez comme suit pour utiliser le `parameters-p2s.json` modifié et exécuter le script afin de créer des ressources Azure.

1. Exécutez PowerShell. Basculez vers le répertoire où se trouve le script.

3. Exécutez le script.

    `.\AzDeployVpn.ps1 -Location <Location> -AzureAppRuleFilePath "appRule.json" -AzureIPRangesFilePath "<Service tag json file>"  -ResourceGroupName "<Resource group name>" -AzureDeploymentName "<Deployment name>" -NetworkRuleCollectionName "<Name for collection of network rules>" -Priority 115 -AppRuleCollectionName "<Name for collection of app rules>"`

    > [!NOTE]
    > Dans cette version, le script fonctionne à l’emplacement USA Est uniquement.

    Lorsque vous exécutez le script, vous devez entrer les informations suivantes :

    
    |Paramètre  |Description  |
    |---------|---------|
    |Emplacement     |Il s’agit de la région dans laquelle les ressources Azure doivent être créées.         |
    |AzureAppRuleFilePath     | Il s’agit du chemin d’accès de fichier pour `appRule.json`.       |
    |AzureIPRangesFilePath     |Il s’agit du fichier JSON d’étiquette de service que vous avez téléchargé à l’étape précédente.         |
    |ResourceGroupName     | Il s’agit du nom du groupe de ressources sous lequel toutes les ressources Azure sont créées.        |
    |AzureDeploymentName    |Il s’agit du nom de votre déploiement Azure.         |
    |NetworkRuleCollectionName            | Il s’agit du nom de la collection de toutes les règles de réseau qui sont créées et ajoutées à votre Pare-feu Azure.             |
    |Priority            | Il s’agit de la priorité affectée à toutes les règles de réseau et d’application qui sont créées.              |
    |AppRuleCollectionName            |Il s’agit du nom de la collection de toutes les règles d’application qui sont créées et ajoutées à votre Pare-feu Azure.                |


    Voici un exemple de sortie obtenue.
    
    ```powershell
    PS C:\Offline docs\AzureVpnConfigurationScripts> .\AzDeployVpn.ps1 -Location eastus -AzureAppRuleFilePath "appRule.json" -AzureIPRangesFilePath ".\ServiceTags_Public_20200203.json" -ResourceGroupName "mytmarg3" -AzureDeploymentName "tmap2stestdeploy1" -NetworkRuleCollectionName "testnrc1" -Priority 115 -AppRuleCollectionName "testarc2"
        validating vpn deployment parameters
        Starting vpn deployment
        C:\Offline docs\AzureVpnConfigurationScripts\parameters-p2s.json
        C:\Offline docs\AzureVpnConfigurationScripts\template-p2s.json
        vpn deployment: tmap2stestdeploy1 started and status: Running
        Waiting for vpn deployment completion....
        ==== CUT ==================== CUT ==============
        Adding route 191.236.0.0/18 for AzureCloud.eastus
        Adding route 191.237.0.0/17 for AzureCloud.eastus
        Adding route 191.238.0.0/18 for AzureCloud.eastus
        Total Routes:294, Existing Routes: 74, New Routes Added: 220
        Additional routes getting added
    ```    

    > [!IMPORTANT]
    > - L’exécution du script prend environ 90 minutes. Veillez à vous connecter à votre réseau juste avant le démarrage du script.
    > - Si, pour une raison quelconque, une session a échoué avec le script, veillez à supprimer le groupe de ressources pour supprimer toutes les ressources qu’il a créées.
  
    
    Une fois le script terminé, un journal de déploiement est généré dans le même dossier que celui où se trouve le script.


## <a name="verify-the-azure-resources"></a>Vérifier les ressources Azure

Une fois que vous avez exécuté le script avec succès, vérifiez que toutes les ressources ont été créées dans Azure. Accédez au groupe de ressources que vous avez créé. Vous devez voir les ressources suivantes :

![Ressources Azure](media/azure-stack-edge-mini-r-configure-vpn-powershell/script-resources.png)


<!--## Enable point-to-site in VPN gateway

1. In the Azure portal, go to the resource group and then select the virtual network gateway that you created in the earlier step.

    ![Azure virtual network gateway](media/azure-stack-edge-mini-r-configure-vpn-powershell/azure-virtual-network-gateway.png)

2. Go to **Settings > Point-to-site configuration**. Select **Configure now**.

    ![Enable P2S configuration](media/azure-stack-edge-mini-r-configure-vpn-powershell/enable-p2s.png)


3.  On the **Point-to-site configuration** blade:

    1. You'll add the client address pool. This pool is a range of private IP addresses that you specify. The clients that connect over a P2S VPN dynamically receive an IP address from this range. Use a private IP address range that does not overlap with the on-premises location that you connect from, or the VNet that you want to connect to. 
    2. You can select the tunnel type. For the VPN tunnel, you will use the IKEv2 protocol. 
    3. You will now define the type of authentication. Before Azure accepts a P2S VPN connection, the user has to be authenticated first. In this case, you authenticate using the native Azure certificate authentication. Set **Authentication type** to **Azure certificate**.
    4. You will now create and upload the certificates to Azure and your client. You will need to install a root certificate on your VPN gateway. The VPN gateway validates the client certificates when the P2S connection is being established by the client. The root certificate is required for the validation and must be uploaded to Azure.
    
        The clients trying to establish the P2S connection will have a client certificate that authenticates the connecting user.  To create these certificates, follow the steps in [Generate and export certificates for Point-to-Site using PowerShell](../vpn-gateway/vpn-gateway-certificates-point-to-site.md). 

        To install the root certificate, make sure the certificate is Base-64 encoded with a .cer extension. Open this certificate and copy the text in the certificate between ==BEGIN CERTIFICATE== and ==END CERTIFICATE== in one continuous line in the public certificate data under Root certificates.

        To upload the root certificates, follow the detailed steps in [Upload the root certificate public certificate data](../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md#uploadfile).
    
    5. Save the configuration.

        ![Save P2S configuration](media/azure-stack-edge-mini-r-configure-vpn-powershell/enable-p2s-config.png)-->

## <a name="download-phone-book-for-vpn-profile"></a>Télécharger l’annuaire téléphonique pour le profil VPN

Au cours de cette étape, vous allez télécharger le profil VPN de votre appareil.

1. Dans le Portail Azure, accédez au groupe de ressources, puis sélectionnez la passerelle de réseau virtuel que vous avez créée à l’étape précédente.

    ![Passerelle de réseau virtuel Azure](media/azure-stack-edge-mini-r-configure-vpn-powershell/azure-virtual-network-gateway.png)

2. Accédez à **Paramètres >Configuration de point à site**. Sélectionnez **Téléchargement du client VPN**.

    ![Activer la configuration P2S 1](media/azure-stack-edge-mini-r-configure-vpn-powershell/download-vpn-client.png)

2. Enregistrez le profil zippé et extrayez-le sur votre client Windows.

    ![Activer la configuration P2S 2](media/azure-stack-edge-mini-r-configure-vpn-powershell/save-extract-profile.png)

3. Accédez à dossier *WindowsAmd64* puis extrayez le `.exe` : *VpnClientSetupAmd64.exe*.

    ![Activer la configuration P2S 3](media/azure-stack-edge-mini-r-configure-vpn-powershell/extract-exe.png)

3. Créez un chemin d’accès temporaire. Par exemple :

    `C:\NewTemp\vnet\tmp`

4. Exécutez PowerShell et accédez au répertoire où se trouve le `.exe`. Pour exécuter le `.exe`, tapez :

    `.\VpnClientSetupAmd64.exe /Q /C /T:"C:\NewTemp\vnet\tmp"`

5. Le chemin d’accès temporaire contient de nouveaux fichiers. Voici un exemple de sortie :

    ```powershell
    
    PS C:\windows\system32> cd "C:\Users\Ase\Downloads\vngw5\WindowsAmd64"
    PS C:\Users\Ase\Downloads\vngw5\WindowsAmd64> .\VpnClientSetupAmd64.exe /Q /C /T:"C:\NewTemp\vnet\tmp"
    PS C:\Users\Ase\Downloads\vngw5\WindowsAmd64> cd "C:\NewTemp\vnet"
    PS C:\NewTemp\vnet> ls .\tmp
    
        Directory: C:\NewTemp\vnet\tmp
    
    Mode                LastWriteTime         Length Name
    ----                -------------         ------ ----
    -a----         2/6/2020   6:18 PM            947 8c670077-470b-421a-8dd8-8cedb4f2f08a.cer
    -a----         2/6/2020   6:18 PM            155 8c670077-470b-421a-8dd8-8cedb4f2f08a.cmp
    -a----         2/6/2020   6:18 PM           3564 8c670077-470b-421a-8dd8-8cedb4f2f08a.cms
    -a----         2/6/2020   6:18 PM          11535 8c670077-470b-421a-8dd8-8cedb4f2f08a.inf
    -a----         2/6/2020   6:18 PM           2285 8c670077-470b-421a-8dd8-8cedb4f2f08a.pbk
    -a----         2/6/2020   6:18 PM           5430 azurebox16.ico
    -a----         2/6/2020   6:18 PM           4286 azurebox32.ico
    -a----         2/6/2020   6:18 PM         138934 azurevpnbanner.bmp
    -a----         2/6/2020   6:18 PM          46064 cmroute.dll
    -a----         2/6/2020   6:18 PM            196 routes.txt
    
    PS C:\NewTemp\vnet>
    ```

6. Le fichier *.pbk* est l’annuaire téléphonique pour le profil VPN. Vous allez l’utiliser dans l’interface utilisateur locale.


## <a name="vpn-configuration-on-the-device"></a>Configuration VPN sur l’appareil

Procédez comme suit sur l’interface utilisateur locale de votre appareil Azure Stack Edge.

1. Dans l’interface utilisateur locale, accédez à la page **VPN**. Sous État VPN, sélectionnez **Configurer**.

    ![Configurer VPN 1](media/azure-stack-edge-mini-r-configure-vpn-powershell/configure-vpn-1.png)

2. Dans le panneau **Configurer un VPN** :
    
    1. Dans le fichier de chargement de l’annuaire téléphonique, pointez sur le fichier. pbk que vous avez créé à l’étape précédente.
    2. Dans le fichier de configuration de chargement de liste d’adresses IP publiques, fournissez le fichier JSON de la plage d’adresses IP du centre de données Azure en tant qu’entrée. Vous avez téléchargé ce fichier à l’étape précédente à partir de : [https://www.microsoft.com/download/details.aspx?id=56519](https://www.microsoft.com/download/details.aspx?id=56519).
    3. Sélectionnez **eastus** comme région et sélectionnez **Appliquer**.

    ![Configurer VPN 2](media/azure-stack-edge-mini-r-configure-vpn-powershell/configure-vpn-2.png)

3. Dans la section **Plages d’adresses IP accessibles uniquement à l’aide du VPN** , entrez la plage IPv4 de réseau virtuel que vous avez choisie pour votre réseau virtuel Azure.

    ![Configurer VPN 3](media/azure-stack-edge-mini-r-configure-vpn-powershell/configure-vpn-3.png)

## <a name="verify-client-connection"></a>Vérifier la connexion cliente

1. Dans le Portail Azure, accédez à la passerelle VPN.
2. Accédez à **Paramètres >Configuration de point à site**. Sous **Adresses IP allouées**, l’adresse IP de votre appareil Azure Stack Edge doit s’afficher.

## <a name="validate-data-transfer-through-vpn"></a>Valider le transfert de données via VPN

Pour confirmer que le VPN fonctionne, copiez les données vers un partage SMB. Suivez les étapes décrites dans [Ajouter un partage](azure-stack-edge-j-series-manage-shares.md#add-a-share) sur votre appareil Azure Stack Edge. 

1. Copiez un fichier, par exemple \data\pictures\waterfall.jpg, dans le partage SMB que vous avez monté sur votre système client. 
2. Pour vérifier que les données passent par un VPN, pendant que les données sont copiées :

    1. Accédez à la passerelle VPN dans le Portail Azure. 

    2. Accédez à **Analyse > métriques**.

    3. Dans le volet droit, choisissez l’**Étendue** en tant que passerelle VPN, **Métrique** en tant que bande passante P2S de la passerelle et **Agrégation** en tant que moyenne.

    4. À mesure que les données sont copiées, vous allez voir une augmentation de l’utilisation de la bande passante et une fois la copie des données terminée, l’utilisation de la bande passante va chuter.

        ![Métriques du VPN Azure](media/azure-stack-edge-mini-r-configure-vpn-powershell/vpn-metrics-1.png)

3. Vérifiez que ce fichier s’affiche dans votre compte de stockage dans le cloud.
 
## <a name="debug-issues"></a>Résoudre les problèmes

Pour déboguer des problèmes, utilisez les commandes suivantes :

```
Get-AzResourceGroupDeployment -DeploymentName $deploymentName -ResourceGroupName $ResourceGroupName
```

Voici un exemple de sortie obtenue :


```azurepowershell
PS C:\Projects\TZL\VPN\Azure-VpnDeployment> Get-AzResourceGroupDeployment -DeploymentName "tznvpnrg14_deployment" -ResourceGroupName "tznvpnrg14"


DeploymentName          : tznvpnrg14_deployment
ResourceGroupName       : tznvpnrg14
ProvisioningState       : Succeeded
Timestamp               : 1/21/2020 6:23:13 PM
Mode                    : Incremental
TemplateLink            :
Parameters              :
                          Name                                         Type                       Value
                          ===========================================  =========================  ==========
                          virtualNetworks_vnet_name                    String                     tznvpnrg14_vnet
                          azureFirewalls_firewall_name                 String                     tznvpnrg14_firewall
                          routeTables_routetable_name                  String                     tznvpnrg14_routetable
                          publicIPAddresses_VNGW_public_ip_name        String                     tznvpnrg14_vngwpublicip
                          virtualNetworkGateways_VNGW_name             String                     tznvpnrg14_vngw
                          publicIPAddresses_firewall_public_ip_name    String                     tznvpnrg14_fwpip
                          localNetworkGateways_LNGW_name               String                     tznvpnrg14_lngw
                          connections_vngw_lngw_name                   String                     tznvpnrg14_connection
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

[Configurez un VPN via l’interface utilisateur locale sur votre appareil Azure Stack Edge](azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption.md#configure-vpn).