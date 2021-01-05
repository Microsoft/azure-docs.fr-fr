---
title: Créer une configuration à ID de sécurité multiple pour SAP à haute disponibilité dans Azure
description: Apprenez à créer une configuration de haute disponibilité pour SAP NetWeaver multi-SID sur des machines virtuelles Windows.
services: virtual-machines-windows, virtual-network, storage
documentationcenter: saponazure
author: rdeltcheva
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: 0b89b4f8-6d6c-45d7-8d20-fe93430217ca
ms.service: virtual-machines-windows
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 05/05/2017
ms.author: radeltch
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 0d9e611ea3bd1243ef93bd5731199c549ed7e431
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94951296"
---
# <a name="create-a-high-availability-sap-netweaver-multi-sid-configuration"></a>Créer une configuration à ID de sécurité multiple pour SAP NetWeaver à haute disponibilité

[load-balancer-multivip-overview]:../../../load-balancer/load-balancer-multivip-overview.md
[sap-ha-guide]:sap-high-availability-guide.md 
[sap-ha-guide-figure-6001]:./media/virtual-machines-shared-sap-high-availability-guide/6001-sap-multi-sid-ascs-scs-sid1.png
[sap-ha-guide-figure-6002]:./media/virtual-machines-shared-sap-high-availability-guide/6002-sap-multi-sid-ascs-scs.png
[sap-ha-guide-figure-6003]:./media/virtual-machines-shared-sap-high-availability-guide/6003-sap-multi-sid-full-landscape.png
[sap-ha-guide-figure-6004]:./media/virtual-machines-shared-sap-high-availability-guide/6004-sap-multi-sid-dns.png
[sap-ha-guide-figure-6005]:./media/virtual-machines-shared-sap-high-availability-guide/6005-sap-multi-sid-azure-portal.png
[sap-ha-guide-figure-6006]:./media/virtual-machines-shared-sap-high-availability-guide/6006-sap-multi-sid-sios-replication.png
[networking-limits-azure-resource-manager]:../../../azure-resource-manager/management/azure-subscription-service-limits.md#azure-resource-manager-virtual-networking-limits
[sap-ha-guide-9.1.1]:sap-high-availability-guide.md#a97ad604-9094-44fe-a364-f89cb39bf097 
[sap-ha-guide-8.8]:sap-high-availability-guide.md#f19bd997-154d-4583-a46e-7f5a69d0153c
[sap-ha-guide-8.12.3.3]:sap-high-availability-guide.md#d9c1fc8e-8710-4dff-bec2-1f535db7b006 
[sap-ha-guide-9]:sap-high-availability-guide.md#a06f0b49-8a7a-42bf-8b0d-c12026c5746b
[sap-ha-guide-9.1]:sap-high-availability-guide.md#31c6bd4f-51df-4057-9fdf-3fcbc619c170 
[sap-ha-guide-9.1.2]:sap-high-availability-guide.md#eb5af918-b42f-4803-bb50-eff41f84b0b0 
[sap-ha-guide-9.1.3]:sap-high-availability-guide.md#e4caaab2-e90f-4f2c-bc84-2cd2e12a9556 
[sap-ha-guide-9.1.4]:sap-high-availability-guide.md#10822f4f-32e7-4871-b63a-9b86c76ce761 
[sap-ha-guide-9.4]:sap-high-availability-guide.md#094bc895-31d4-4471-91cc-1513b64e406a 
[sap-ha-guide-9.5]:sap-high-availability-guide.md#2477e58f-c5a7-4a5d-9ae3-7b91022cafb5 
[sap-ha-guide-9.6]:sap-high-availability-guide.md#0ba4a6c1-cc37-4bcf-a8dc-025de4263772 
[sap-ha-guide-10]:sap-high-availability-guide.md#18aa2b9d-92d2-4c0e-8ddd-5acaabda99e9

En septembre 2016, Microsoft a publié une fonctionnalité vous permettant de gérer plusieurs adresses IP virtuelles à l’aide d’un équilibrage de charge interne Azure. Cette fonctionnalité existe déjà dans l’équilibrage de charge externe Azure.

Dans le cas d’un déploiement SAP, vous pouvez utiliser un équilibrage de charge interne pour créer une configuration de cluster Windows pour SAP ASCS/SCS, comme indiqué dans le [Guide de haute disponibilité SAP NetWeaver sur machines virtuelles Windows][sap-ha-guide].

Cet article met l’accent sur le passage d’une installation ASCS/SCS unique à une configuration SAP multi-SID en installant des instances SAP ASCS/SCS en cluster supplémentaires dans un cluster de basculement Windows Server (WSFC) existant. Lorsque ce processus est terminé, vous aurez configuré un cluster multi-SID SAP.

[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prérequis
Vous avez déjà configuré un cluster WSFC qui est utilisé pour une instance SAP ASCS/SCS, comme indiqué dans le [Guide de haute disponibilité SAP NetWeaver sur machines virtuelles Windows][sap-ha-guide] et comme illustré dans ce diagramme.

![Instance SAP ASCS/SCS à haute disponibilité][sap-ha-guide-figure-6001]

## <a name="target-architecture"></a>Architecture cible

L’objectif est d’installer plusieurs instances en cluster SAP ABAP ASCS ou SAP Java SCS dans un même cluster WSFC, comme indiqué ici :

![Plusieurs instances SAP ASCS/SCS en cluster dans Azure][sap-ha-guide-figure-6002]

> [!NOTE]
>Il existe une limite au nombre d’adresses IP frontales privées pour chaque équilibrage de charge interne Azure.
>
>Le nombre maximal d’instances SAP ASCS/SCS dans un cluster WSFC est égal au nombre maximal d’adresses IP frontales privées pour chaque équilibrage de charge interne Azure.
>

Pour plus d’informations sur les limites de l’équilibreur de charge, consultez « Adresse IP frontale privée par équilibreur de charge » dans [Limites de réseau : Azure Resource Manager][networking-limits-azure-resource-manager].

Voici une vue d’ensemble avec deux systèmes SAP à haute disponibilité :

![Configuration SAP multi-SID haute disponibilité avec deux SID système SAP][sap-ha-guide-figure-6003]

> [!IMPORTANT]
> La configuration doit répondre aux conditions suivantes :
> - Les instances SAP ASCS/SCS partagent le même cluster WSFC.
> - Chaque SID DBMS a son propre cluster WSFC dédié.
> - Les serveurs d’applications SAP appartenant au système SAP SID utilisent leurs propres machines virtuelles.


## <a name="prepare-the-infrastructure"></a>Préparer l’infrastructure
Pour préparer votre infrastructure, vous pouvez installer une instance SAP ASCS/SCS supplémentaire avec les paramètres suivants :

| Nom du paramètre | Valeur |
| --- | --- |
| SID ASCS/SCS SAP |pr1-lb-ascs |
| Équilibrage de charge interne du SGBD SAP | PR5 |
| Nom d’hôte virtuel SAP | pr5-sap-cl |
| Adresse IP de l’hôte virtuel SAP ASCS/SCS (adresse IP d’équilibrage de charge Azure supplémentaire) | 10.0.0.50 |
| Numéro de l’instance SAP ASCS/SCS | 50 |
| Port de sonde d’équilibrage de charge interne pour l’instance SAP ASCS/SCS supplémentaire | 62350 |

> [!NOTE]
> Pour des instances SAP ASCS/SCS en cluster, chaque adresse IP doit avoir un port de sonde unique. Par exemple, si une adresse IP sur un équilibrage de charge interne Azure utilise le port de sonde 62300, aucune autre adresse IP de cet équilibrage de charge ne peut utiliser le port 62300 de la sonde.
>
>Pour nos besoins, car le port de sonde 62300 est déjà réservé, nous utilisons le port de sonde 62350.

Vous pouvez installer des instances SAP ASCS/SCS supplémentaire dans le cluster WSFC existant avec deux nœuds :

| Rôle de la machine virtuelle | Nom d’hôte de la machine virtuelle | Adresse IP statique |
| --- | --- | --- |
| 1er nœud de cluster pour l’instance ASCS/SCS |pr1-ascs-0 |10.0.0.10 |
| 2e nœud de cluster pour l’instance ASCS/SCS |pr1-ascs-1 |10.0.0.9 |

### <a name="create-a-virtual-host-name-for-the-clustered-sap-ascsscs-instance-on-the-dns-server"></a>Créer un nom d’hôte virtuel pour l’instance SAP ASCS/SCS en cluster sur le serveur DNS

Vous pouvez créer une entrée DNS pour le nom d’hôte virtuel de l’instance ASCS/SCS en utilisant les paramètres suivants :

| Nouveau nom d’hôte virtuel SAP ASCS/SCS | Adresse IP associée |
| --- | --- |
|pr5-sap-cl |10.0.0.50 |

Le nouveau nom d’hôte et l’adresse IP apparaissent dans le Gestionnaire DNS, comme illustré dans la capture d’écran suivante :

![Liste du Gestionnaire DNS mettant en surbrillance l’entrée DNS définie pour le nom virtuel et l’adresse TCP/IP du nouveau cluster SAP ASCS/SCS][sap-ha-guide-figure-6004]

La procédure de création d’une entrée DNS est également décrite en détail dans le principal [guide de haute disponibilité SAP NetWeaver sur des machines virtuelles Windows][sap-ha-guide-9.1.1].

> [!NOTE]
> La nouvelle adresse IP que vous affectez au nom d’hôte virtuel de l’instance ASCS/SCS supplémentaire doit être identique à la nouvelle adresse IP que vous avez affectée à l’équilibrage de charge SAP Azure.
>
>Dans notre scénario, l’adresse IP est 10.0.0.50.

### <a name="add-an-ip-address-to-an-existing-azure-internal-load-balancer-by-using-powershell"></a>Ajouter une adresse IP à un équilibrage de charge interne Azure existant à l’aide de PowerShell

Pour créer plusieurs instances SAP ASCS/SCS dans le même cluster WSFC, utilisez PowerShell pour ajouter une adresse IP à un équilibrage de charge interne Azure existant. Chaque adresse IP requiert une règle d’équilibrage de charge, un port de sonde, un pool IP frontal et un pool principal propres.

Le script suivant ajoute une nouvelle adresse IP à un équilibreur de charge existant. Mettez à jour les variables PowerShell de votre environnement. Le script crée toutes les règles d’équilibrage de charge requises pour tous les ports SAP ASCS/SCS.

```powershell

# Select-AzSubscription -SubscriptionId <xxxxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx>
Clear-Host
$ResourceGroupName = "SAP-MULTI-SID-Landscape"      # Existing resource group name
$VNetName = "pr2-vnet"                        # Existing virtual network name
$SubnetName = "Subnet"                        # Existing subnet name
$ILBName = "pr2-lb-ascs"                      # Existing ILB name                      
$ILBIP = "10.0.0.50"                          # New IP address
$VMNames = "pr2-ascs-0","pr2-ascs-1"          # Existing cluster virtual machine names
$SAPInstanceNumber = 50                       # SAP ASCS/SCS instance number: must be a unique value for each cluster
[int]$ProbePort = "623$SAPInstanceNumber"     # Probe port: must be a unique value for each IP and load balancer

$ILB = Get-AzLoadBalancer -Name $ILBName -ResourceGroupName $ResourceGroupName

$count = $ILB.FrontendIpConfigurations.Count + 1
$FrontEndConfigurationName ="lbFrontendASCS$count"
$LBProbeName = "lbProbeASCS$count"

# Get the Azure VNet and subnet
$VNet = Get-AzVirtualNetwork -Name $VNetName -ResourceGroupName $ResourceGroupName
$Subnet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $VNet -Name $SubnetName

# Add second front-end and probe configuration
Write-Host "Adding new front end IP Pool '$FrontEndConfigurationName' ..." -ForegroundColor Green
$ILB | Add-AzLoadBalancerFrontendIpConfig -Name $FrontEndConfigurationName -PrivateIpAddress $ILBIP -SubnetId $Subnet.Id
$ILB | Add-AzLoadBalancerProbeConfig -Name $LBProbeName  -Protocol Tcp -Port $Probeport -ProbeCount 2 -IntervalInSeconds 10  | Set-AzLoadBalancer

# Get new updated configuration
$ILB = Get-AzLoadBalancer -Name $ILBname -ResourceGroupName $ResourceGroupName
# Get new updated LP FrontendIP COnfig
$FEConfig = Get-AzLoadBalancerFrontendIpConfig -Name $FrontEndConfigurationName -LoadBalancer $ILB
$HealthProbe  = Get-AzLoadBalancerProbeConfig -Name $LBProbeName -LoadBalancer $ILB

# Add new back-end configuration into existing ILB
$BackEndConfigurationName  = "backendPoolASCS$count"
Write-Host "Adding new backend Pool '$BackEndConfigurationName' ..." -ForegroundColor Green
$BEConfig = Add-AzLoadBalancerBackendAddressPoolConfig -Name $BackEndConfigurationName -LoadBalancer $ILB | Set-AzLoadBalancer

# Get new updated config
$ILB = Get-AzLoadBalancer -Name $ILBname -ResourceGroupName $ResourceGroupName

# Assign VM NICs to backend pool
$BEPool = Get-AzLoadBalancerBackendAddressPoolConfig -Name $BackEndConfigurationName -LoadBalancer $ILB
foreach($VMName in $VMNames){
        $VM = Get-AzVM -ResourceGroupName $ResourceGroupName -Name $VMName
        $NICName = ($VM.NetworkInterfaceIDs[0].Split('/') | select -last 1)        
        $NIC = Get-AzNetworkInterface -name $NICName -ResourceGroupName $ResourceGroupName                
        $NIC.IpConfigurations[0].LoadBalancerBackendAddressPools += $BEPool
        Write-Host "Assigning network card '$NICName' of the '$VMName' VM to the backend pool '$BackEndConfigurationName' ..." -ForegroundColor Green
        Set-AzNetworkInterface -NetworkInterface $NIC
        #start-AzVM -ResourceGroupName $ResourceGroupName -Name $VM.Name
}


# Create load-balancing rules
$Ports = "445","32$SAPInstanceNumber","33$SAPInstanceNumber","36$SAPInstanceNumber","39$SAPInstanceNumber","5985","81$SAPInstanceNumber","5$SAPInstanceNumber`13","5$SAPInstanceNumber`14","5$SAPInstanceNumber`16"
$ILB = Get-AzLoadBalancer -Name $ILBname -ResourceGroupName $ResourceGroupName
$FEConfig = get-AzLoadBalancerFrontendIpConfig -Name $FrontEndConfigurationName -LoadBalancer $ILB
$BEConfig = Get-AzLoadBalancerBackendAddressPoolConfig -Name $BackEndConfigurationName -LoadBalancer $ILB
$HealthProbe  = Get-AzLoadBalancerProbeConfig -Name $LBProbeName -LoadBalancer $ILB

Write-Host "Creating load balancing rules for the ports: '$Ports' ... " -ForegroundColor Green

foreach ($Port in $Ports) {

        $LBConfigrulename = "lbrule$Port" + "_$count"
        Write-Host "Creating load balancing rule '$LBConfigrulename' for the port '$Port' ..." -ForegroundColor Green

        $ILB | Add-AzLoadBalancerRuleConfig -Name $LBConfigRuleName -FrontendIpConfiguration $FEConfig  -BackendAddressPool $BEConfig -Probe $HealthProbe -Protocol tcp -FrontendPort  $Port -BackendPort $Port -IdleTimeoutInMinutes 30 -LoadDistribution Default -EnableFloatingIP   
}

$ILB | Set-AzLoadBalancer

Write-Host "Successfully added new IP '$ILBIP' to the internal load balancer '$ILBName'!" -ForegroundColor Green

```
Une fois le script exécuté, les résultats sont affichés dans le portail Azure, comme indiqué dans la capture d’écran suivante :

![Nouveau pool IP frontal dans le portail Azure][sap-ha-guide-figure-6005]

### <a name="add-disks-to-cluster-machines-and-configure-the-sios-cluster-share-disk"></a>Ajouter des disques aux machines du cluster et configurer le disque de partage de cluster SIOS

Vous devez ajouter un nouveau disque de partage de cluster pour chaque instance SAP ASCS/SCS supplémentaire. Pour Windows Server 2012 R2, le disque de partage de cluster WSFC utilisé actuellement est la solution logicielle SIOS DataKeeper.

Effectuez les actions suivantes :
1. Ajoutez un ou plusieurs disques supplémentaires de même taille (que vous devrez entrelacer) à chacun des nœuds du cluster et formatez-les.
2. Configurez la réplication de stockage avec SIOS DataKeeper.

Cette procédure suppose que vous avez déjà installé SIOS DataKeeper sur les machines du cluster WSFC. Si vous l’avez installé, vous devez maintenant configurer la réplication entre les machines. La procédure est décrite en détail dans le principal [guide de haute disponibilité SAP NetWeaver sur des machines virtuelles Windows][sap-ha-guide-8.12.3.3].  

![Mise en miroir synchrone de DataKeeper pour le nouveau disque de partage SAP ASCS/SCS][sap-ha-guide-figure-6006]

### <a name="deploy-vms-for-sap-application-servers-and-dbms-cluster"></a>Déployer des machines virtuelles pour les serveurs d’applications SAP et le cluster DBMS

Pour terminer la préparation de l’infrastructure pour le deuxième système SAP, procédez comme suit :

1. Déployer des machines virtuelles dédiées pour les serveurs d’applications SAP et les placer dans leur groupe de disponibilité dédié.
2. Déployer des machines virtuelles dédiées pour le cluster DBMS et les placer dans leur groupe de disponibilité dédié.


## <a name="install-the-second-sap-sid2-netweaver-system"></a>Installer le deuxième système SAP SID2 NetWeaver

Le processus complet d’installation d’un deuxième système SAP SID2 est décrit dans le principal [guide de haute disponibilité SAP NetWeaver sur des machines virtuelles Windows][sap-ha-guide-9].

La procédure détaillée est la suivante :

1. [Installer le premier nœud de cluster SAP][sap-ha-guide-9.1.2].  
 Dans cette étape, vous installez SAP avec une instance ASCS/SCS à haute disponibilité sur le **nœud de cluster WSFC existant 1**.

2. [Modifier le profil SAP de l’instance ASCS/SCS][sap-ha-guide-9.1.3].

3. [Configurer un port de sonde][sap-ha-guide-9.1.4].  
 Dans cette étape, vous configurez le port de sonde SAP-SID2-IP d’une ressource de cluster SAP à l’aide de PowerShell. Exécutez cette configuration sur un des nœuds de cluster SAP ASCS/SCS.

4. [Installer l’instance de base de données][sap-ha-guide-9.2].  
 Dans cette étape, vous installez SGBD sur un cluster WSFC dédié.

5. [Installer le deuxième nœud de cluster][sap-ha-guide-9.3].  
 Dans cette étape, vous installez SAP avec une instance ASCS/SCS à haute disponibilité sur le nœud de cluster WSFC existant 2.

6. Ouvrez les ports du pare-feu Windows de l’instance SAP ASCS/SCS et ProbePort.  
 Sur les deux nœuds de cluster utilisés pour l’instance SAP ASCS/SCS, vous ouvrez tous les ports du pare-feu Windows utilisés par SAP ASCS/SCS. Ces ports sont répertoriés dans le [guide de haute disponibilité SAP NetWeaver sur des machines virtuelles Windows][sap-ha-guide-8.8].  
 Ouvrez également le port de sonde de l’équilibrage de charge interne Azure, 62350 dans notre scénario.

7. [Modifiez le type de démarrage de l’instance de service Windows SAP ERS][sap-ha-guide-9.4].

8. [Installez le serveur d’applications principal SAP][sap-ha-guide-9.5] sur la nouvelle machine virtuelle dédiée.

9. [Installez le serveur d’applications SAP supplémentaire][sap-ha-guide-9.6] sur la nouvelle machine virtuelle dédiée.

10. [Testez le basculement et la réplication SIOS de l’instance SAP ASCS/SCS][sap-ha-guide-10].

## <a name="next-steps"></a>Étapes suivantes

- [Limites de mise en réseau : Azure Resource Manager][networking-limits-azure-resource-manager]
- [Adresses IP virtuelles multiples pour Azure Load Balancer][load-balancer-multivip-overview]
- [Guide de haute disponibilité SAP NetWeaver sur des machines virtuelles Windows][sap-ha-guide]
