---
title: Activer Azure DS Domain Services à l’aide de PowerShell | Microsoft Docs
description: Découvrez comment configurer et activer Azure Active Directory Domain Services via Azure AD PowerShell et Azure PowerShell.
services: active-directory-ds
author: justinha
manager: daveba
ms.assetid: d4bc5583-6537-4cd9-bc4b-7712fdd9272a
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: sample
ms.date: 10/02/2020
ms.author: justinha
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 89061af04147d7cfaa0fdb3a6b1a8fb1cd8c8da7
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96619145"
---
# <a name="enable-azure-active-directory-domain-services-using-powershell"></a>Activer Azure Active Directory Domain Services à l’aide de PowerShell

Azure Active Directory Domain Services (Azure AD DS) fournit des services de domaine managés, comme la jonction de domaine, la stratégie de groupe, le protocole LDAP, et l’authentification Kerberos/NTLM entièrement compatible avec Windows Server Active Directory. Vous consommez ces services de domaine sans déployer, gérer et mettre à jour avec des correctifs les contrôleurs de domaine vous-même. Azure AD DS s’intègre à votre locataire Azure AD existant. Cette intégration permet aux utilisateurs de se connecter en utilisant leurs informations d’identification d’entreprise, et vous pouvez utiliser des groupes et des comptes d’utilisateur existants pour sécuriser l’accès aux ressources.

Cet article vous montre comment activer Azure à l’aide de PowerShell.

[!INCLUDE [updated-for-az.md](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prérequis

Pour effectuer ce qui est décrit dans cet article, vous avez besoin des ressources suivantes :

* Installez et configurez Azure PowerShell.
    * Si nécessaire, suivez les instructions pour [installer le module Azure PowerShell et vous connecter à votre abonnement Azure](/powershell/azure/install-az-ps).
    * Veillez à vous connecter à votre abonnement Azure à l’aide de l’applet de commande [Connect-AzAccount][Connect-AzAccount].
* Installez et configurez Azure AD PowerShell.
    * Si nécessaire, suivez les instructions pour [installer le module Azure AD PowerShell et vous connecter à Azure AD](/powershell/azure/active-directory/install-adv2).
    * Veillez à vous connecter à votre locataire Azure AD à l’aide de l’applet de commande [Connect-AzureAD][Connect-AzureAD].
* Vous devez disposer des privilèges d’*Administrateur global* dans votre locataire Azure AD pour activer Azure AD DS.
* Vous avez besoin de privilèges de *Contributeur* dans votre abonnement Azure pour créer les ressources Azure AD DS nécessaires.

## <a name="create-required-azure-ad-resources"></a>Créer les ressources Azure AD nécessaires

Azure AD DS nécessite un principal du service et un groupe Azure AD. Ces ressources permettent au domaine managé Azure AD DS de synchroniser les données et de définir les utilisateurs qui possèdent des autorisations administratives dans le domaine managé.

Tout d’abord, créez un principal du service Azure AD pour permettre à Azure AD DS de communiquer et de s’authentifier. Un ID d’application spécifique est utilisé. Il se nomme *Domain Controller Services* et son ID est *6ba9a5d4-8456-4118-b521-9c5ca10cdf84*. Ne modifiez pas cet ID d’application.

Créez un principal du service Azure AD à l’aide de l’applet de commande [New-AzureADServicePrincipal][New-AzureADServicePrincipal] :

```powershell
New-AzureADServicePrincipal -AppId "6ba9a5d4-8456-4118-b521-9c5ca10cdf84"
```

Créez ensuite un groupe Azure AD nommé *AAD DC Administrators*. Les utilisateurs ajoutés à ce groupe se voient ensuite accorder des autorisations pour effectuer des tâches d’administration dans le domaine managé.

Obtenez d’abord l’ID d’objet du groupe *Administrateurs AAD DC* en utilisant l’applet de commande [Get-AzureADGroup][Get-AzureADGroup]. Si le groupe n’existe pas, créez-le avec le groupe *Administrateurs AAD DC* en utilisant l’applet de commande [New-AzureADGroup][New-AzureADGroup] :

```powershell
# First, retrieve the object ID of the 'AAD DC Administrators' group.
$GroupObjectId = Get-AzureADGroup `
  -Filter "DisplayName eq 'AAD DC Administrators'" | `
  Select-Object ObjectId

# If the group doesn't exist, create it
if (!$GroupObjectId) {
  $GroupObjectId = New-AzureADGroup -DisplayName "AAD DC Administrators" `
    -Description "Delegated group to administer Azure AD Domain Services" `
    -SecurityEnabled $true `
    -MailEnabled $false `
    -MailNickName "AADDCAdministrators"
  }
else {
  Write-Output "Admin group already exists."
}
```

Une fois le groupe *Administrateurs AAD DC* créé, obtenez l’ID d’objet de l’utilisateur souhaité en utilisant l’applet de commande [Get-AzureADUser][Get-AzureADUser], puis ajoutez l’utilisateur au groupe en utilisant l’applet de commande [Add-AzureADGroupMember][Add-AzureADGroupMember].

Dans l’exemple suivant, l’ID d’objet utilisateur du compte a le nom d’utilisateur principal (UPN) `admin@contoso.onmicrosoft.com`. Remplacez ce compte d’utilisateur par l’UPN de l’utilisateur que vous souhaitez ajouter au groupe *AAD DC Administrators* :

```powershell
# Retrieve the object ID of the user you'd like to add to the group.
$UserObjectId = Get-AzureADUser `
  -Filter "UserPrincipalName eq 'admin@contoso.onmicrosoft.com'" | `
  Select-Object ObjectId

# Add the user to the 'AAD DC Administrators' group.
Add-AzureADGroupMember -ObjectId $GroupObjectId.ObjectId -RefObjectId $UserObjectId.ObjectId
```

## <a name="create-network-resources"></a>Créer des ressources réseau

Dans un premier temps, inscrivez le fournisseur de ressources Azure AD Domain Services à l’aide de l’applet de commande [Register-AzResourceProvider][Register-AzResourceProvider] :

```powershell
Register-AzResourceProvider -ProviderNamespace Microsoft.AAD
```

Créez ensuite un groupe de ressources avec l’applet de commande [New-AzResourceGroup][New-AzResourceGroup]. Dans l’exemple suivant, le groupe de ressources est nommé *myResourceGroup* et est créé dans la région *westus*. Utilisez votre propre nom et la région souhaitée :

```powershell
$ResourceGroupName = "myResourceGroup"
$AzureLocation = "westus"

# Create the resource group.
New-AzResourceGroup `
  -Name $ResourceGroupName `
  -Location $AzureLocation
```

Créez le réseau virtuel et des sous-réseaux pour Azure AD Domain Services. Deux sous-réseaux sont créés : un pour *DomainServices* et un autre pour *Workloads*. Azure AD DS est déployé dans le sous-réseau dédié *DomainServices*. Ne déployez pas d’autres applications ou charges de travail dans ce sous-réseau. Utilisez le sous-réseau indépendant *Workloads* ou d’autres sous-réseaux pour le reste de vos machines virtuelles.

Créez les sous-réseaux à l’aide de l’applet de commande [New-AzVirtualNetworkSubnetConfig][New-AzVirtualNetworkSubnetConfig], puis créez le réseau virtuel à l’aide de l’applet de commande [New-AzVirtualNetwork][New-AzVirtualNetwork].

```powershell
$VnetName = "myVnet"

# Create the dedicated subnet for Azure AD Domain Services.
$SubnetName = "DomainServices"
$AaddsSubnet = New-AzVirtualNetworkSubnetConfig `
  -Name $SubnetName `
  -AddressPrefix 10.0.0.0/24

# Create an additional subnet for your own VM workloads
$WorkloadSubnet = New-AzVirtualNetworkSubnetConfig `
  -Name Workloads `
  -AddressPrefix 10.0.1.0/24

# Create the virtual network in which you will enable Azure AD Domain Services.
$Vnet= New-AzVirtualNetwork `
  -ResourceGroupName $ResourceGroupName `
  -Location westus `
  -Name $VnetName `
  -AddressPrefix 10.0.0.0/16 `
  -Subnet $AaddsSubnet,$WorkloadSubnet
```

### <a name="create-a-network-security-group"></a>Créer un groupe de sécurité réseau

Azure AD DS a besoin d’un groupe de sécurité réseau pour sécuriser les ports nécessaires au domaine managé et bloquer tout autre trafic entrant. Un [groupe de sécurité réseau (NSG)][nsg-overview] contient la liste des règles qui autorisent ou rejettent le trafic réseau vers le trafic d’un réseau virtuel Azure. Dans Azure AD DS, le groupe de sécurité réseau agit comme une couche de protection supplémentaire pour verrouiller l’accès au domaine managé. Pour afficher les ports obligatoires, consultez [Groupes de sécurité réseau et ports nécessaires][network-ports].

Les cmdlets PowerShell suivantes utilisent [New-AzNetworkSecurityRuleConfig][New-AzNetworkSecurityRuleConfig] pour créer les règles, puis [New-AzNetworkSecurityGroup][New-AzNetworkSecurityGroup] pour créer le groupe de sécurité réseau. Le groupe de sécurité réseau et les règles sont ensuite associés au sous-réseau de réseau virtuel à l’aide de la cmdlet [Set-AzVirtualNetworkSubnetConfig][Set-AzVirtualNetworkSubnetConfig].

```powershell
$NSGName = "aaddsNSG"

# Create a rule to allow inbound TCP port 443 traffic for synchronization with Azure AD
$nsg101 = New-AzNetworkSecurityRuleConfig `
    -Name AllowSyncWithAzureAD `
    -Access Allow `
    -Protocol Tcp `
    -Direction Inbound `
    -Priority 101 `
    -SourceAddressPrefix AzureActiveDirectoryDomainServices `
    -SourcePortRange * `
    -DestinationAddressPrefix * `
    -DestinationPortRange 443

# Create a rule to allow inbound TCP port 3389 traffic from Microsoft secure access workstations for troubleshooting
$nsg201 = New-AzNetworkSecurityRuleConfig -Name AllowRD `
    -Access Allow `
    -Protocol Tcp `
    -Direction Inbound `
    -Priority 201 `
    -SourceAddressPrefix CorpNetSaw `
    -SourcePortRange * `
    -DestinationAddressPrefix * `
    -DestinationPortRange 3389

# Create a rule to allow TCP port 5986 traffic for PowerShell remote management
$nsg301 = New-AzNetworkSecurityRuleConfig -Name AllowPSRemoting `
    -Access Allow `
    -Protocol Tcp `
    -Direction Inbound `
    -Priority 301 `
    -SourceAddressPrefix AzureActiveDirectoryDomainServices `
    -SourcePortRange * `
    -DestinationAddressPrefix * `
    -DestinationPortRange 5986

# Create the network security group and rules
$nsg = New-AzNetworkSecurityGroup -Name $NSGName `
    -ResourceGroupName $ResourceGroupName `
    -Location $AzureLocation `
    -SecurityRules $nsg101,$nsg201,$nsg301

# Get the existing virtual network resource objects and information
$vnet = Get-AzVirtualNetwork -Name $VnetName -ResourceGroupName $ResourceGroupName
$subnet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name $SubnetName
$addressPrefix = $subnet.AddressPrefix

# Associate the network security group with the virtual network subnet
Set-AzVirtualNetworkSubnetConfig -Name $SubnetName `
    -VirtualNetwork $vnet `
    -AddressPrefix $addressPrefix `
    -NetworkSecurityGroup $nsg
$vnet | Set-AzVirtualNetwork
```

## <a name="create-a-managed-domain"></a>Créer un domaine managé

Créons maintenant un domaine managé. Définissez votre ID d’abonnement Azure, puis attribuez un nom au domaine managé, par exemple *aaddscontoso.com*. Vous pouvez obtenir votre ID d’abonnement à l’aide de l’applet de commande [Get-AzSubscription][Get-AzSubscription].

Si vous choisissez une région qui prend en charge les Zones de disponibilité, les ressources Azure AD DS sont réparties entre les zones pour assurer une redondance supplémentaire.

Les Zones de disponibilité sont des emplacements physiques uniques au sein d’une région Azure. Chaque zone de disponibilité est composée d’un ou de plusieurs centres de données équipés d’une alimentation, d’un système de refroidissement et d’un réseau indépendants. Pour garantir la résilience, un minimum de trois zones distinctes sont activées dans toutes les régions.

Vous ne devez rien configurer pour la répartition d’Azure AD DS entre les zones. La plateforme Azure gère automatiquement la répartition de zone des ressources. Pour plus d’informations et pour connaître la disponibilité régionale, voir [Zones de disponibilité dans Azure][availability-zones].

```powershell
$AzureSubscriptionId = "YOUR_AZURE_SUBSCRIPTION_ID"
$ManagedDomainName = "aaddscontoso.com"

# Enable Azure AD Domain Services for the directory.
New-AzResource -ResourceId "/subscriptions/$AzureSubscriptionId/resourceGroups/$ResourceGroupName/providers/Microsoft.AAD/DomainServices/$ManagedDomainName" `
  -ApiVersion "2017-06-01" `
  -Location $AzureLocation `
  -Properties @{"DomainName"=$ManagedDomainName; `
    "SubnetId"="/subscriptions/$AzureSubscriptionId/resourceGroups/$ResourceGroupName/providers/Microsoft.Network/virtualNetworks/$VnetName/subnets/DomainServices"} `
  -Force -Verbose
```

Créer la ressource et retourner le contrôle à l’invite PowerShell prend quelques minutes. Le provisionnement du domaine managé se poursuit en arrière-plan et le déploiement peut prendre jusqu’à une heure. Dans le portail Azure, la page **Vue d’ensemble** de votre domaine managé indique l’état actuel tout au long de cette phase de déploiement.

Une fois que le portail Azure a indiqué que le provisionnement du domaine managé était terminé, voici les tâches qu’il convient d’effectuer :

* Mettez à jour les paramètres DNS pour le réseau virtuel afin que les machines virtuelles puissent trouver le domaine géré pour l’authentification ou la jonction de domaine.
    * Pour configure le système DNS, sélectionnez votre domaine managé dans le portail. Dans la fenêtre **Vue d’ensemble**, vous êtes invité à configurer automatiquement ces paramètres DNS.
* [Activez la synchronisation de mot de passe avec Azure AD DS](tutorial-create-instance.md#enable-user-accounts-for-azure-ad-ds) afin que les utilisateurs finaux puissent se connecter au domaine managé avec leurs informations d’identification d’entreprise.

## <a name="complete-powershell-script"></a>Compléter le script PowerShell

Le script PowerShell complet suivant regroupe toutes les tâches présentées dans cet article. Copiez le script et enregistrez-le dans un fichier avec une extension `.ps1`. Exécutez le script dans une console PowerShell locale ou [Azure Cloud Shell][cloud-shell].

> [!NOTE]
> Pour activer Azure AD DS, vous devez être administrateur général du locataire Azure AD. Vous avez aussi besoin au moins de privilèges *Contributeur* dans l’abonnement Azure.

```powershell
# Change the following values to match your deployment.
$AaddsAdminUserUpn = "admin@contoso.onmicrosoft.com"
$ResourceGroupName = "myResourceGroup"
$VnetName = "myVnet"
$AzureLocation = "westus"
$AzureSubscriptionId = "YOUR_AZURE_SUBSCRIPTION_ID"
$ManagedDomainName = "aaddscontoso.com"

# Connect to your Azure AD directory.
Connect-AzureAD

# Login to your Azure subscription.
Connect-AzAccount

# Create the service principal for Azure AD Domain Services.
New-AzureADServicePrincipal -AppId "6ba9a5d4-8456-4118-b521-9c5ca10cdf84"

# First, retrieve the object ID of the 'AAD DC Administrators' group.
$GroupObjectId = Get-AzureADGroup `
  -Filter "DisplayName eq 'AAD DC Administrators'" | `
  Select-Object ObjectId

# Create the delegated administration group for Azure AD Domain Services if it doesn't already exist.
if (!$GroupObjectId) {
  $GroupObjectId = New-AzureADGroup -DisplayName "AAD DC Administrators" `
    -Description "Delegated group to administer Azure AD Domain Services" `
    -SecurityEnabled $true `
    -MailEnabled $false `
    -MailNickName "AADDCAdministrators"
  }
else {
  Write-Output "Admin group already exists."
}

# Now, retrieve the object ID of the user you'd like to add to the group.
$UserObjectId = Get-AzureADUser `
  -Filter "UserPrincipalName eq '$AaddsAdminUserUpn'" | `
  Select-Object ObjectId

# Add the user to the 'AAD DC Administrators' group.
Add-AzureADGroupMember -ObjectId $GroupObjectId.ObjectId -RefObjectId $UserObjectId.ObjectId

# Register the resource provider for Azure AD Domain Services with Resource Manager.
Register-AzResourceProvider -ProviderNamespace Microsoft.AAD

# Create the resource group.
New-AzResourceGroup `
  -Name $ResourceGroupName `
  -Location $AzureLocation

# Create the dedicated subnet for AAD Domain Services.
$SubnetName = "DomainServices"
$AaddsSubnet = New-AzVirtualNetworkSubnetConfig `
  -Name DomainServices `
  -AddressPrefix 10.0.0.0/24

$WorkloadSubnet = New-AzVirtualNetworkSubnetConfig `
  -Name Workloads `
  -AddressPrefix 10.0.1.0/24

# Create the virtual network in which you will enable Azure AD Domain Services.
$Vnet=New-AzVirtualNetwork `
  -ResourceGroupName $ResourceGroupName `
  -Location $AzureLocation `
  -Name $VnetName `
  -AddressPrefix 10.0.0.0/16 `
  -Subnet $AaddsSubnet,$WorkloadSubnet
  
$NSGName = "aaddsNSG"

# Create a rule to allow inbound TCP port 443 traffic for synchronization with Azure AD
$nsg101 = New-AzNetworkSecurityRuleConfig `
    -Name AllowSyncWithAzureAD `
    -Access Allow `
    -Protocol Tcp `
    -Direction Inbound `
    -Priority 101 `
    -SourceAddressPrefix AzureActiveDirectoryDomainServices `
    -SourcePortRange * `
    -DestinationAddressPrefix * `
    -DestinationPortRange 443

# Create a rule to allow inbound TCP port 3389 traffic from Microsoft secure access workstations for troubleshooting
$nsg201 = New-AzNetworkSecurityRuleConfig -Name AllowRD `
    -Access Allow `
    -Protocol Tcp `
    -Direction Inbound `
    -Priority 201 `
    -SourceAddressPrefix CorpNetSaw `
    -SourcePortRange * `
    -DestinationAddressPrefix * `
    -DestinationPortRange 3389

# Create a rule to allow TCP port 5986 traffic for PowerShell remote management
$nsg301 = New-AzNetworkSecurityRuleConfig -Name AllowPSRemoting `
    -Access Allow `
    -Protocol Tcp `
    -Direction Inbound `
    -Priority 301 `
    -SourceAddressPrefix AzureActiveDirectoryDomainServices `
    -SourcePortRange * `
    -DestinationAddressPrefix * `
    -DestinationPortRange 5986

# Create the network security group and rules
$nsg = New-AzNetworkSecurityGroup -Name $NSGName `
    -ResourceGroupName $ResourceGroupName `
    -Location $AzureLocation `
    -SecurityRules $nsg101,$nsg201,$nsg301

# Get the existing virtual network resource objects and information
$vnet = Get-AzVirtualNetwork -Name $VnetName -ResourceGroupName $ResourceGroupName
$subnet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name $SubnetName
$addressPrefix = $subnet.AddressPrefix

# Associate the network security group with the virtual network subnet
Set-AzVirtualNetworkSubnetConfig -Name $SubnetName `
    -VirtualNetwork $vnet `
    -AddressPrefix $addressPrefix `
    -NetworkSecurityGroup $nsg
$vnet | Set-AzVirtualNetwork

# Enable Azure AD Domain Services for the directory.
New-AzResource -ResourceId "/subscriptions/$AzureSubscriptionId/resourceGroups/$ResourceGroupName/providers/Microsoft.AAD/DomainServices/$ManagedDomainName" `
  -ApiVersion "2017-06-01" `
  -Location $AzureLocation `
  -Properties @{"DomainName"=$ManagedDomainName; `
    "SubnetId"="/subscriptions/$AzureSubscriptionId/resourceGroups/$ResourceGroupName/providers/Microsoft.Network/virtualNetworks/$VnetName/subnets/DomainServices"} `
  -Force -Verbose
```

Créer la ressource et retourner le contrôle à l’invite PowerShell prend quelques minutes. Le provisionnement du domaine managé se poursuit en arrière-plan et le déploiement peut prendre jusqu’à une heure. Dans le portail Azure, la page **Vue d’ensemble** de votre domaine managé indique l’état actuel tout au long de cette phase de déploiement.

Une fois que le portail Azure a indiqué que le provisionnement du domaine managé était terminé, voici les tâches qu’il convient d’effectuer :

* Mettez à jour les paramètres DNS pour le réseau virtuel afin que les machines virtuelles puissent trouver le domaine géré pour l’authentification ou la jonction de domaine.
    * Pour configure le système DNS, sélectionnez votre domaine managé dans le portail. Dans la fenêtre **Vue d’ensemble**, vous êtes invité à configurer automatiquement ces paramètres DNS.
* [Activez la synchronisation de mot de passe avec Azure AD DS](tutorial-create-instance.md#enable-user-accounts-for-azure-ad-ds) afin que les utilisateurs finaux puissent se connecter au domaine managé avec leurs informations d’identification d’entreprise.

## <a name="next-steps"></a>Étapes suivantes

Pour voir le domaine managé en action, vous pouvez [joindre une machine virtuelle Windows à un domaine][windows-join], [configurer le protocole LDAP sécurisé][tutorial-ldaps] et [configurer la synchronisation du hachage de mot de passe][tutorial-phs].

<!-- INTERNAL LINKS -->
[windows-join]: join-windows-vm.md
[tutorial-ldaps]: tutorial-configure-ldaps.md
[tutorial-phs]: tutorial-configure-password-hash-sync.md
[nsg-overview]: ../virtual-network/network-security-groups-overview.md
[network-ports]: network-considerations.md#network-security-groups-and-required-ports

<!-- EXTERNAL LINKS -->
[Connect-AzAccount]: /powershell/module/Az.Accounts/Connect-AzAccount
[Connect-AzureAD]: /powershell/module/AzureAD/Connect-AzureAD
[New-AzureADServicePrincipal]: /powershell/module/AzureAD/New-AzureADServicePrincipal
[New-AzureADGroup]: /powershell/module/AzureAD/New-AzureADGroup
[Add-AzureADGroupMember]: /powershell/module/AzureAD/Add-AzureADGroupMember
[Get-AzureADGroup]: /powershell/module/AzureAD/Get-AzureADGroup
[Get-AzureADUser]: /powershell/module/AzureAD/Get-AzureADUser
[Register-AzResourceProvider]: /powershell/module/Az.Resources/Register-AzResourceProvider
[New-AzResourceGroup]: /powershell/module/Az.Resources/New-AzResourceGroup
[New-AzVirtualNetworkSubnetConfig]: /powershell/module/Az.Network/New-AzVirtualNetworkSubnetConfig
[New-AzVirtualNetwork]: /powershell/module/Az.Network/New-AzVirtualNetwork
[Get-AzSubscription]: /powershell/module/Az.Accounts/Get-AzSubscription
[cloud-shell]: ../cloud-shell/cloud-shell-windows-users.md
[availability-zones]: ../availability-zones/az-overview.md
[New-AzNetworkSecurityRuleConfig]: /powershell/module/az.network/new-aznetworksecurityruleconfig
[New-AzNetworkSecurityGroup]: /powershell/module/az.network/new-aznetworksecuritygroup
[Set-AzVirtualNetworkSubnetConfig]: /powershell/module/az.network/set-azvirtualnetworksubnetconfig
