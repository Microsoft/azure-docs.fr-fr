---
title: Créer un équilibreur de charge public avec IPv6 – Azure CLI
titleSuffix: Azure Load Balancer
description: Dans ce parcours d’apprentissage, familiarisez-vous avec la création d’un équilibreur de charge public avec IPv6 à l’aide d’Azure CLI.
services: load-balancer
documentationcenter: na
author: asudbring
keywords: IPv6, équilibreur de charge azure, double pile, adresse ip publique, ipv6 natif, mobile, iot
ms.service: load-balancer
ms.devlang: na
ms.topic: how-to
ms.custom: seodec18, devx-track-azurecli
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/25/2018
ms.author: allensu
ms.openlocfilehash: 75226f92995794221635ced7ee0e285ac824b6e2
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94696861"
---
# <a name="create-a-public-load-balancer-with-ipv6-using-azure-cli"></a>Créer un équilibreur de charge public avec IPv6 à l’aide d’Azure CLI

>[!NOTE] 
>Cet article décrit une fonctionnalité IPv6 qui permet aux équilibreurs de charge de base de fournir une connectivité IPv4 et IPv6. Une connectivité IPv6 complète est maintenant disponible avec [IPv6 pour les réseaux virtuels Azure](../virtual-network/ipv6-overview.md), qui intègre la connectivité IPv6 aux réseaux virtuels et inclut des fonctionnalités clés comme les règles de groupe de sécurité réseau IPv6, le routage défini par l’utilisateur IPv6, l’équilibrage de charge IPv6 de base et standard, et plus encore.  Le protocole IPv6 pour les réseaux virtuels Azure est la norme recommandée pour les applications IPv6 dans Azure. Consultez [IPv6 pour le déploiement PowerShell de réseau virtuel Azure](../virtual-network/virtual-network-ipv4-ipv6-dual-stack-standard-load-balancer-powershell.md). 

Un équilibrage de charge Azure est de type Couche 4 (TCP, UDP). Les équilibreurs de charge offrent une disponibilité élevée en distribuant le trafic entrant parmi les instances de service saines dans les services cloud ou les machines virtuelles dans un jeu d’équilibreur de charge. Les équilibreurs de charge peuvent également présenter ces services sur plusieurs ports, plusieurs adresses IP ou les deux.

## <a name="example-deployment-scenario"></a>Exemple de scénario de déploiement

Le diagramme suivant illustre la solution d’équilibrage de charge déployée à l’aide de l’exemple de modèle décrit dans cet article.

![Scénario d’équilibreur de charge](./media/load-balancer-ipv6-internet-cli/lb-ipv6-scenario-cli.png)

Dans ce scénario, vous créez les ressources Azure suivantes :

* Deux machines virtuelles
* Une interface de réseau virtuel pour chaque machine virtuelle avec des adresses IPv4 et IPv6 affectées
* Un équilibreur de charge public avec une adresse IP publique IPv4 et IPv6
* Un groupe à haute disponibilité contenant les deux machines virtuelles
* Deux règles d’équilibrage de charge pour mapper les adresses IP virtuelles publiques sur les points de terminaison privés

## <a name="deploy-the-solution-by-using-azure-cli"></a>Déployer la solution à l’aide d’Azure CLI

Les étapes suivantes expliquent comment créer un équilibreur de charge public à l’aide d’Azure CLI. Avec CLI, vous créez et configurez chaque objet individuellement, puis vous les rassemblez pour créer une ressource.

Créez et configurez les objets suivants pour déployer un équilibreur de charge :

* **Configuration d’adresses IP frontales** : contient les adresses IP publiques pour le trafic réseau entrant.
* **Pool d’adresses principales** : contient des interfaces réseau pour que les machines virtuelles puissent recevoir le trafic réseau de l’équilibreur de charge.
* **Règles d’équilibrage de charge** : contient des règles qui mappent un port public situé sur l’équilibreur de charge à un port du pool d’adresses principales.
* **Règles NAT entrantes**: contient des règles de traduction d’adresses réseau (NAT) qui mappent un port public situé sur l’équilibreur de charge vers le port d’une machine virtuelle spécifique située dans le pool d’adresses principales.
* **Sondes** : contient les sondes d’intégrité utilisées pour vérifier la disponibilité des instances de machines virtuelles du pool d’adresses principales.

## <a name="set-up-azure-cli"></a>Configuration de l’interface de ligne de commande Azure CLI

Dans cet exemple, vous exécutez les outils Azure CLI dans une fenêtre de commande PowerShell. Pour améliorer la lisibilité et la réutilisation, vous utilisez les fonctionnalités de script de PowerShell et non les cmdlets Azure PowerShell.

1. [Installez et configurez l’interface Azure CLI](/cli/azure/install-azure-cli?view=azure-cli-latest) en suivant les étapes de l’article lié, puis connectez-vous à votre compte Azure.

2. Configurez des variables PowerShell à utiliser avec les commandes Azure CLI :

    ```powershell
    $subscriptionid = "########-####-####-####-############"  # enter subscription id
    $location = "southcentralus"
    $rgName = "pscontosorg1southctrlus09152016"
    $vnetName = "contosoIPv4Vnet"
    $vnetPrefix = "10.0.0.0/16"
    $subnet1Name = "clicontosoIPv4Subnet1"
    $subnet1Prefix = "10.0.0.0/24"
    $subnet2Name = "clicontosoIPv4Subnet2"
    $subnet2Prefix = "10.0.1.0/24"
    $dnsLabel = "contoso09152016"
    $lbName = "myIPv4IPv6Lb"
    ```

## <a name="create-a-resource-group-a-load-balancer-a-virtual-network-and-subnets"></a>Créer un groupe de ressources, un équilibrage de charge, un réseau virtuel et des sous-réseaux

1. Créez un groupe de ressources :

    ```azurecli
    az group create --name $rgName --location $location
    ```

2. Créez un équilibreur de charge :

    ```azurecli
    $lb = az network lb create --resource-group $rgname --location $location --name $lbName
    ```

3. Créez un réseau virtuel :

    ```azurecli
    $vnet = az network vnet create  --resource-group $rgname --name $vnetName --location $location --address-prefixes $vnetPrefix
    ```

4. Dans ce réseau virtuel, créez deux sous-réseaux :

    ```azurecli
    $subnet1 = az network vnet subnet create --resource-group $rgname --name $subnet1Name --address-prefix $subnet1Prefix --vnet-name $vnetName
    $subnet2 = az network vnet subnet create --resource-group $rgname --name $subnet2Name --address-prefix $subnet2Prefix --vnet-name $vnetName
    ```

## <a name="create-public-ip-addresses-for-the-front-end-pool"></a>Créer des adresses IP publiques pour le pool frontal

1. Configurez les variables PowerShell :

    ```powershell
    $publicIpv4Name = "myIPv4Vip"
    $publicIpv6Name = "myIPv6Vip"
    ```

2. Créez une adresse IP publique pour le pool frontal :

    ```azurecli
    $publicipV4 = az network public-ip create --resource-group $rgname --name $publicIpv4Name --location $location --version IPv4 --allocation-method Dynamic --dns-name $dnsLabel
    $publicipV6 = az network public-ip create --resource-group $rgname --name $publicIpv6Name --location $location --version IPv6 --allocation-method Dynamic --dns-name $dnsLabel
    ```

    > [!IMPORTANT]
    > L’équilibreur de charge utilise l’étiquette du domaine de l’adresse IP publique en tant que nom de domaine complet (FQDN). Cet usage diffère d’un déploiement classique qui utilise le service cloud en tant que nom de domaine complet de l’équilibrage de charge.
    >
    > Dans cet exemple, le nom de domaine complet est *contoso09152016.southcentralus.cloudapp.azure.com*.

## <a name="create-front-end-and-back-end-pools"></a>Créer ds pools frontaux et principaux

Dans cette section, vous créez les pools IP suivants :
* Le pool IP frontal qui reçoit le trafic réseau entrant sur l’équilibreur de charge.
* Le pool d’IP principal où le pool frontal envoie le trafic réseau dont la charge a été équilibrée.

1. Configurez les variables PowerShell :

    ```powershell
    $frontendV4Name = "FrontendVipIPv4"
    $frontendV6Name = "FrontendVipIPv6"
    $backendAddressPoolV4Name = "BackendPoolIPv4"
    $backendAddressPoolV6Name = "BackendPoolIPv6"
    ```

2. Créez un pool d’IP frontal et associez-le à l’IP publique créée lors de l’étape précédente et à l’équilibreur de charge.

    ```azurecli
    $frontendV4 = az network lb frontend-ip create --resource-group $rgname --name $frontendV4Name --public-ip-address $publicIpv4Name --lb-name $lbName
    $frontendV6 = az network lb frontend-ip create --resource-group $rgname --name $frontendV6Name --public-ip-address $publicIpv6Name --lb-name $lbName
    $backendAddressPoolV4 = az network lb address-pool create --resource-group $rgname --name $backendAddressPoolV4Name --lb-name $lbName
    $backendAddressPoolV6 = az network lb address-pool create --resource-group $rgname --name $backendAddressPoolV6Name --lb-name $lbName
    ```

## <a name="create-the-probe-nat-rules-and-load-balancer-rules"></a>Créer la sonde, les règles NAT et les règles d’équilibreur de charge

Cet exemple crée les éléments suivants :

* Une règle de sonde dédiée à la détection de connectivité sur le port TCP 80.
* Une règle NAT pour transférer l’ensemble du trafic entrant sur le port 3389 vers le port 3389 pour RDP.\*
* Une règle NAT pour transférer l’ensemble du trafic entrant sur le port 3391 vers le port 3389 pour le protocole RDP (Remote Desktop Protocol).\*
* Une règle d’équilibreur de charge pour équilibrer tout le trafic entrant sur le port 80 vers le port 80 des adresses du pool principal.

\* Les règles NAT sont associées à une instance de machine virtuelle spécifique derrière l’équilibreur de charge. Le trafic réseau arrivant sur le port 3389 est envoyé à la machine virtuelle spécifique sur le port associé à la règle NAT. Vous devez spécifier un protocole (TCP ou UDP) pour une règle NAT. Vous ne pouvez pas attribuer ces deux protocoles au même port.

1. Configurez les variables PowerShell :

    ```powershell
    $probeV4V6Name = "ProbeForIPv4AndIPv6"
    $natRule1V4Name = "NatRule-For-Rdp-VM1"
    $natRule2V4Name = "NatRule-For-Rdp-VM2"
    $lbRule1V4Name = "LBRuleForIPv4-Port80"
    $lbRule1V6Name = "LBRuleForIPv6-Port80"
    ```

2. Créez la sonde.

    L’exemple suivant illustre la création d’une sonde TCP qui détecte la connectivité au port TCP principal 80 toutes les 15 secondes. Après deux échecs consécutifs, elle marque la ressource principale comme indisponible.

    ```azurecli
    $probeV4V6 = az network lb probe create --resource-group $rgname --name $probeV4V6Name --protocol tcp --port 80 --interval 15 --threshold 2 --lb-name $lbName
    ```

3. Créez des règles NAT entrantes qui prennent en charge les connexions RDP aux ressources principales :

    ```azurecli
    $inboundNatRuleRdp1 = az network lb inbound-nat-rule create --resource-group $rgname --name $natRule1V4Name --frontend-ip-name $frontendV4Name --protocol Tcp --frontend-port 3389 --backend-port 3389 --lb-name $lbName
    $inboundNatRuleRdp2 = az network lb inbound-nat-rule create --resource-group $rgname --name $natRule2V4Name --frontend-ip-name $frontendV4Name --protocol Tcp --frontend-port 3391 --backend-port 3389 --lb-name $lbName
    ```

4. Créez des règles d’équilibreur de charge qui transmettent le trafic sur différents ports principaux en fonction du port frontal de réception de la requête.

    ```azurecli
    $lbruleIPv4 = az network lb rule create --resource-group $rgname --name $lbRule1V4Name --frontend-ip-name $frontendV4Name --backend-pool-name $backendAddressPoolV4Name --probe-name $probeV4V6Name --protocol Tcp --frontend-port 80 --backend-port 80 --lb-name $lbName
    $lbruleIPv6 = az network lb rule create --resource-group $rgname --name $lbRule1V6Name --frontend-ip-name $frontendV6Name --backend-pool-name $backendAddressPoolV6Name --probe-name $probeV4V6Name --protocol Tcp --frontend-port 80 --backend-port 8080 --lb-name $lbName
    ```

5. Vérifiez vos paramètres :

    ```azurecli
    az network lb show --resource-group $rgName --name $lbName
    ```

    Sortie attendue :

    ```output
    info:    Executing command network lb show
    info:    Looking up the load balancer "myIPv4IPv6Lb"
    data:    Id                              : /subscriptions/########-####-####-####-############/resourceGroups/pscontosorg1southctrlus09152016/providers/Microsoft.Network/loadBalancers/myIPv4IPv6Lb
    data:    Name                            : myIPv4IPv6Lb
    data:    Type                            : Microsoft.Network/loadBalancers
    data:    Location                        : southcentralus
    data:    Provisioning state              : Succeeded
    data:
    data:    Frontend IP configurations:
    data:    Name             Provisioning state  Private IP allocation  Private IP   Subnet  Public IP
    data:    ---------------  ------------------  ---------------------  -----------  ------  ---------
    data:    FrontendVipIPv4  Succeeded           Dynamic                                     myIPv4Vip
    data:    FrontendVipIPv6  Succeeded           Dynamic                                     myIPv6Vip
    data:
    data:    Probes:
    data:    Name                 Provisioning state  Protocol  Port  Path  Interval  Count
    data:    -------------------  ------------------  --------  ----  ----  --------  -----
    data:    ProbeForIPv4AndIPv6  Succeeded           Tcp       80          15        2
    data:
    data:    Backend Address Pools:
    data:    Name             Provisioning state
    data:    ---------------  ------------------
    data:    BackendPoolIPv4  Succeeded
    data:    BackendPoolIPv6  Succeeded
    data:
    data:    Load Balancing Rules:
    data:    Name                  Provisioning state  Load distribution  Protocol  Frontend port  Backend port  Enable floating IP  Idle timeout in minutes
    data:    --------------------  ------------------  -----------------  --------  -------------  ------------  ------------------  -----------------------
    data:    LBRuleForIPv4-Port80  Succeeded           Default            Tcp       80             80            false               4
    data:    LBRuleForIPv6-Port80  Succeeded           Default            Tcp       80             8080          false               4
    data:
    data:    Inbound NAT Rules:
    data:    Name                 Provisioning state  Protocol  Frontend port  Backend port  Enable floating IP  Idle timeout in minutes
    data:    -------------------  ------------------  --------  -------------  ------------  ------------------  -----------------------
    data:    NatRule-For-Rdp-VM1  Succeeded           Tcp       3389           3389          false               4
    data:    NatRule-For-Rdp-VM2  Succeeded           Tcp       3391           3389          false               4
    info:    network lb show
    ```

## <a name="create-nics"></a>Créer des cartes réseau

Créez des cartes réseau et associez-les à des règles NAT, des règles d’équilibreur de charge et des sondes.

1. Configurez les variables PowerShell :

    ```powershell
    $nic1Name = "myIPv4IPv6Nic1"
    $nic2Name = "myIPv4IPv6Nic2"
    $subnet1Id = "/subscriptions/$subscriptionid/resourceGroups/$rgName/providers/Microsoft.Network/VirtualNetworks/$vnetName/subnets/$subnet1Name"
    $subnet2Id = "/subscriptions/$subscriptionid/resourceGroups/$rgName/providers/Microsoft.Network/VirtualNetworks/$vnetName/subnets/$subnet2Name"
    $backendAddressPoolV4Id = "/subscriptions/$subscriptionid/resourceGroups/$rgname/providers/Microsoft.Network/loadbalancers/$lbName/backendAddressPools/$backendAddressPoolV4Name"
    $backendAddressPoolV6Id = "/subscriptions/$subscriptionid/resourceGroups/$rgname/providers/Microsoft.Network/loadbalancers/$lbName/backendAddressPools/$backendAddressPoolV6Name"
    $natRule1V4Id = "/subscriptions/$subscriptionid/resourceGroups/$rgname/providers/Microsoft.Network/loadbalancers/$lbName/inboundNatRules/$natRule1V4Name"
    $natRule2V4Id = "/subscriptions/$subscriptionid/resourceGroups/$rgname/providers/Microsoft.Network/loadbalancers/$lbName/inboundNatRules/$natRule2V4Name"
    ```

2. Créez une carte réseau pour chaque instance principale et ajoutez une configuration IPv6 :

    ```azurecli
    $nic1 = az network nic create --name $nic1Name --resource-group $rgname --location $location --private-ip-address-version "IPv4" --subnet $subnet1Id --lb-address-pools $backendAddressPoolV4Id --lb-inbound-nat-rules $natRule1V4Id
    $nic1IPv6 = az network nic ip-config create --resource-group $rgname --name "IPv6IPConfig" --private-ip-address-version "IPv6" --lb-address-pools $backendAddressPoolV6Id --nic-name $nic1Name

    $nic2 = az network nic create --name $nic2Name --resource-group $rgname --location $location --private-ip-address-version "IPv4" --subnet $subnet2Id --lb-address-pools $backendAddressPoolV4Id --lb-inbound-nat-rules $natRule2V4Id
    $nic2IPv6 = az network nic ip-config create --resource-group $rgname --name "IPv6IPConfig" --private-ip-address-version "IPv6" --lb-address-pools $backendAddressPoolV6Id --nic-name $nic2Name
    ```

## <a name="create-the-back-end-vm-resources-and-attach-each-nic"></a>Créer les ressources de machines virtuelles principales et associer chaque carte réseau

Pour créer des machines virtuelles, vous devez disposer d’un compte de stockage. Pour l’équilibrage de charge, les machines virtuelles doivent être membres d’un groupe à haute disponibilité. Pour plus d’informations sur la création des machines virtuelles, consultez [Créer une machine virtuelle Windows avec PowerShell](../virtual-machines/windows/quick-create-powershell.md?toc=%2fazure%2fload-balancer%2ftoc.json).

1. Configurez les variables PowerShell :

    ```powershell
    $availabilitySetName = "myIPv4IPv6AvailabilitySet"
    $vm1Name = "myIPv4IPv6VM1"
    $vm2Name = "myIPv4IPv6VM2"
    $nic1Id = "/subscriptions/$subscriptionid/resourceGroups/$rgname/providers/Microsoft.Network/networkInterfaces/$nic1Name"
    $nic2Id = "/subscriptions/$subscriptionid/resourceGroups/$rgname/providers/Microsoft.Network/networkInterfaces/$nic2Name"
    $imageurn = "MicrosoftWindowsServer:WindowsServer:2012-R2-Datacenter:latest"
    $vmUserName = "vmUser"
    $mySecurePassword = "PlainTextPassword*1"
    ```

    > [!WARNING]
    > Cet exemple utilise le nom d’utilisateur et le mot de passe pour les machines virtuelles, en texte en clair. Prenez les mesures nécessaires lorsque vous utilisez ces informations d’identification en texte en clair. Pour découvrir une méthode plus sécurisée de traitement des informations d’identification dans PowerShell, consultez la cmdlet [`Get-Credential`](/powershell/module/microsoft.powershell.security/get-credential).

2. Créez le groupe à haute disponibilité :

    ```azurecli
    $availabilitySet = az vm availability-set create --name $availabilitySetName --resource-group $rgName --location $location
    ```

3. Créez les machines virtuelles avec les cartes réseau associées :

    ```azurecli
    az vm create --resource-group $rgname --name $vm1Name --image $imageurn --admin-username $vmUserName --admin-password $mySecurePassword --nics $nic1Id --location $location --availability-set $availabilitySetName --size "Standard_A1" 

    az vm create --resource-group $rgname --name $vm2Name --image $imageurn --admin-username $vmUserName --admin-password $mySecurePassword --nics $nic2Id --location $location --availability-set $availabilitySetName --size "Standard_A1" 
    ```