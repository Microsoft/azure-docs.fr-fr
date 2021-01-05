---
title: Connexion d’un service cloud à un contrôleur de domaine personnalisé | Microsoft Docs
description: Découvrez comment connecter vos rôles web/de travail à un domaine Active Directory personnalisé à l’aide de Powershell et de l’extension de domaine Active Directory
services: cloud-services
author: tgore03
ms.service: cloud-services
ms.topic: article
ms.date: 07/18/2017
ms.author: tagore
ms.openlocfilehash: fa918a3a6894205ed36c4b576608e7a71e523a92
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87092709"
---
# <a name="connecting-azure-cloud-services-roles-to-a-custom-ad-domain-controller-hosted-in-azure"></a>Connexion des rôles de services cloud Azure à un contrôleur de domaine Active Directory personnalisé hébergé dans Azure
Nous allons tout d’abord définir un réseau virtuel (VNet) dans Azure. Nous allons ensuite ajouter un contrôleur de domaine Active Directory (hébergé sur une machine virtuelle Azure) sur le réseau virtuel. Ensuite, nous ajouterons des rôles de service cloud existants sur le réseau virtuel créé au préalable, puis les connecterons au contrôleur de domaine.

Avant de commencer, quelques aspects à prendre en compte :

1. Ce didacticiel utilise Powershell. Par conséquent, vérifiez qu’Azure Powershell est installé et prêt à l’emploi. Pour obtenir de l’aide sur la configuration d’Azure Powershell, consultez l’article [Installation et configuration d’Azure PowerShell](/powershell/azure/).
2. Vos instances de contrôleur de domaine AD et de rôle web/de travail doivent se trouver dans le réseau virtuel.

Suivez ce guide pas à pas et, si vous rencontrez des problèmes, laissez-nous un commentaire à la fin de l’article. Nous reviendrons vers vous (oui, nous lisons les commentaires).

Le réseau qui est référencé par le service cloud doit être un **réseau virtuel classique**.

## <a name="create-a-virtual-network"></a>Création d'un réseau virtuel
Vous pouvez créer un réseau virtuel dans Azure via le portail Azure ou Powershell. Ce didacticiel utilise PowerShell. Pour créer un réseau virtuel à l’aide du Portail Azure, consultez la page [Créer un réseau virtuel](../virtual-network/quick-create-portal.md). L’article aborde la création d’un réseau virtuel (Resource Manager), mais vous devez créer un réseau virtuel (Classic) pour les services cloud. Pour cela, sélectionnez **Créer une ressource** sur le portail, tapez *réseau virtuel* dans la zone **Recherche**, puis appuyez sur **Entrée**. Dans les résultats de la recherche, sélectionnez **Réseaux virtuels** sous **Tout**. Sous **Sélectionner un modèle de déploiement**, sélectionnez **Classic**, puis **Créer**. Vous pouvez maintenant suivre les étapes de l’article.

```powershell
#Create Virtual Network

$vnetStr =
@"<?xml version="1.0" encoding="utf-8"?>
<NetworkConfiguration xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/ServiceHosting/2011/07/NetworkConfiguration">
    <VirtualNetworkConfiguration>
    <VirtualNetworkSites>
        <VirtualNetworkSite name="[your-vnet-name]" Location="West US">
        <AddressSpace>
            <AddressPrefix>[your-address-prefix]</AddressPrefix>
        </AddressSpace>
        <Subnets>
            <Subnet name="[your-subnet-name]">
            <AddressPrefix>[your-subnet-range]</AddressPrefix>
            </Subnet>
        </Subnets>
        </VirtualNetworkSite>
    </VirtualNetworkSites>
    </VirtualNetworkConfiguration>
</NetworkConfiguration>
"@;

$vnetConfigPath = "<path-to-vnet-config>"
Set-AzureVNetConfig -ConfigurationPath $vnetConfigPath
```

## <a name="create-a-virtual-machine"></a>Création d'une machine virtuelle
Une fois que vous avez terminé la configuration du réseau virtuel, vous devrez créer un contrôleur de domaine Active Directory. Pour ce didacticiel, nous configurerons un contrôleur de domaine Active Directory sur une machine virtuelle Azure.

Pour ce faire, créez une machine virtuelle via Powershell à l’aide des commandes suivantes :

```powershell
# Initialize variables
# VNet and subnet must be classic virtual network resources, not Azure Resource Manager resources.

$vnetname = '<your-vnet-name>'
$subnetname = '<your-subnet-name>'
$vmsvc1 = '<your-hosted-service>'
$vm1 = '<your-vm-name>'
$username = '<your-username>'
$password = '<your-password>'
$affgrp = '<your- affgrp>'

# Create a VM and add it to the Virtual Network

New-AzureQuickVM -Windows -ServiceName $vmsvc1 -Name $vm1 -ImageName $imgname -AdminUsername $username -Password $password -AffinityGroup $affgrp -SubnetNames $subnetname -VNetName $vnetname
```

## <a name="promote-your-virtual-machine-to-a-domain-controller"></a>Promotion de votre machine virtuelle vers un contrôleur de domaine
Pour configurer la machine virtuelle comme un contrôleur de domaine Active Directory, vous devez ouvrir une session sur la machine virtuelle et la configurer.

Pour vous connecter à la machine virtuelle, vous pouvez obtenir le fichier RDP via Powershell. Utilisez les commandes suivantes :

```powershell
# Get RDP file
Get-AzureRemoteDesktopFile -ServiceName $vmsvc1 -Name $vm1 -LocalPath <rdp-file-path>
```

Une fois connecté à la machine virtuelle, configurez votre machine virtuelle comme un contrôleur de domaine Active Directory en suivant le guide pas à pas de la rubrique [Configuration de votre contrôleur de domaine Active Directory de client](https://social.technet.microsoft.com/wiki/contents/articles/12370.windows-server-2012-set-up-your-first-domain-controller-step-by-step.aspx).

## <a name="add-your-cloud-service-to-the-virtual-network"></a>Ajout de votre service cloud au réseau virtuel
Ensuite, vous devez ajouter le déploiement de votre service cloud sur le nouveau réseau virtuel. Pour ce faire, modifiez votre cscfg de service cloud en ajoutant les sections pertinentes à votre cscfg à l'aide de Visual Studio ou de l'éditeur de votre choix.

```xml
<ServiceConfiguration serviceName="[hosted-service-name]" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceConfiguration" osFamily="[os-family]" osVersion="*">
    <Role name="[role-name]">
    <Instances count="[number-of-instances]" />
    </Role>
    <NetworkConfiguration>

    <!--optional-->
    <Dns>
        <DnsServers><DnsServer name="[dns-server-name]" IPAddress="[ip-address]" /></DnsServers>
    </Dns>
    <!--optional-->

    <!--VNet settings
        VNet and subnet must be classic virtual network resources, not Azure Resource Manager resources.-->
    <VirtualNetworkSite name="[virtual-network-name]" />
    <AddressAssignments>
        <InstanceAddress roleName="[role-name]">
        <Subnets>
            <Subnet name="[subnet-name]" />
        </Subnets>
        </InstanceAddress>
    </AddressAssignments>
    <!--VNet settings-->

    </NetworkConfiguration>
</ServiceConfiguration>
```

Générez ensuite votre projet de services cloud et déployez-le dans Azure. Pour obtenir de l'aide pour le déploiement de votre package de services cloud dans Azure, consultez l'article [Création et déploiement d'un service cloud](cloud-services-how-to-create-deploy-portal.md)

## <a name="connect-your-webworker-roles-to-the-domain"></a>Connexion de vos rôles web/de travail au domaine
Une fois votre projet de service cloud déployé sur Azure, connectez vos instances de rôle pour le domaine Active Directory personnalisé à l'aide de l'extension de domaine Active Directory. Pour ajouter l’extension de domaine Active Directory à votre déploiement de services cloud existant et rejoindre le domaine personnalisé, exécutez les commandes suivantes dans Powershell :

```powershell
# Initialize domain variables

$domain = '<your-domain-name>'
$dmuser = '$domain\<your-username>'
$dmpswd = '<your-domain-password>'
$dmspwd = ConvertTo-SecureString $dmpswd -AsPlainText -Force
$dmcred = New-Object System.Management.Automation.PSCredential ($dmuser, $dmspwd)

# Add AD Domain Extension to the cloud service roles

Set-AzureServiceADDomainExtension -Service <your-cloud-service-hosted-service-name> -Role <your-role-name> -Slot <staging-or-production> -DomainName $domain -Credential $dmcred -JoinOption 35
```

Et c’est terminé !

Vos services cloud doivent être joints à votre contrôleur de domaine personnalisé. Si vous souhaitez en savoir plus sur les différentes options disponibles pour la configuration de l’extension de domaine Active Directory, utilisez l’aide PowerShell. Voici quelques exemples :

```powershell
help Set-AzureServiceADDomainExtension
help New-AzureServiceADDomainExtensionConfig
```



