---
title: Guide d’utilisation d’Azure PowerShell pour approvisionner SQL Server sur une machine virtuelle Azure
description: Fournit une procédure et des commandes PowerShell permettant de créer une machine virtuelle Azure à l’aide des images de la galerie de machines virtuelles SQL Server.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
editor: ''
tags: azure-resource-manager
ms.assetid: 98d50dd8-48ad-444f-9031-5378d8270d7b
ms.service: virtual-machines-sql
ms.subservice: deployment
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 12/21/2018
ms.author: mathoma
ms.reviewer: jroth
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: a3f51a07b274320d1cd9f12b33703d8ec7f21f49
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97359657"
---
# <a name="how-to-use-azure-powershell-to-provision-sql-server-on-azure-virtual-machines"></a>Comment utiliser Azure PowerShell pour approvisionner SQL Server sur des machines virtuelles Azure

[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Ce guide couvre les options d’utilisation de PowerShell pour approvisionner SQL Server sur des machines virtuelles Azure. Pour obtenir un exemple Azure PowerShell simplifié qui repose sur les valeurs par défaut, consultez la section [Démarrage rapide d’Azure PowerShell pour des machines virtuelles SQL](sql-vm-create-powershell-quickstart.md).

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

[!INCLUDE [updated-for-az.md](../../../../includes/updated-for-az.md)]

## <a name="configure-your-subscription"></a>Configurer votre abonnement

1. Ouvrez PowerShell et accédez à votre compte Azure en exécutant la commande **Connect-AzAccount**.

   ```powershell
   Connect-AzAccount
   ```

1. À l’invite, entrez vos informations d’identification. Utilisez l'adresse électronique et le mot de passe que vous utilisez pour vous connecter au portail Azure.

## <a name="define-image-variables"></a>Définir des variables d’image

Pour réutiliser des valeurs et simplifier la création de scripts, commencez par définir quelques variables. Modifiez les valeurs des paramètres comme vous le souhaitez, mais sachez que des restrictions s’appliquent à la longueur des noms et aux caractères spéciaux en cas de modification des valeurs fournies.

### <a name="location-and-resource-group"></a>Emplacement et groupe de ressources

Définissez la région de données et le groupe de ressources dans lesquels vous souhaitez créer les autres ressources de machine virtuelle.

Apportez les modifications souhaitées, puis exécutez ces cmdlets pour initialiser ces variables.

```powershell
$Location = "SouthCentralUS"
$ResourceGroupName = "sqlvm2"
```

### <a name="storage-properties"></a>Propriétés de stockage

Définissez le compte de stockage et le type de stockage que la machine virtuelle doit utiliser.

Apportez les modifications souhaitées, puis exécutez la cmdlet suivante pour initialiser ces variables. Nous vous recommandons d'utiliser des [disques SSD Premium](../../../virtual-machines/disks-types.md#premium-ssd) pour les charges de travail de production.

```powershell
$StorageName = $ResourceGroupName + "storage"
$StorageSku = "Premium_LRS"
```

### <a name="network-properties"></a>Propriétés réseau

Définissez les propriétés que le réseau doit utiliser dans la machine virtuelle. 

- interface réseau
- Méthode d’allocation TCP/IP
- Nom du réseau virtuel
- Nom du sous-réseau virtuel
- Plage d’adresses IP du réseau virtuel
- Plage d’adresses IP du sous-réseau
- Étiquette de nom de domaine public

Apportez les modifications souhaitées, puis exécutez cette cmdlet pour initialiser ces variables.

```powershell
$InterfaceName = $ResourceGroupName + "ServerInterface"
$NsgName = $ResourceGroupName + "nsg"
$TCPIPAllocationMethod = "Dynamic"
$VNetName = $ResourceGroupName + "VNet"
$SubnetName = "Default"
$VNetAddressPrefix = "10.0.0.0/16"
$VNetSubnetAddressPrefix = "10.0.0.0/24"
$DomainName = $ResourceGroupName
```

### <a name="virtual-machine-properties"></a>Propriétés de machine virtuelle

Définissez les propriétés suivantes :

- Nom de la machine virtuelle
- Nom de l’ordinateur
- Taille de la machine virtuelle
- Nom du disque du système d’exploitation pour la machine virtuelle

Apportez les modifications souhaitées, puis exécutez cette cmdlet pour initialiser ces variables.

```powershell
$VMName = $ResourceGroupName + "VM"
$ComputerName = $ResourceGroupName + "Server"
$VMSize = "Standard_DS13"
$OSDiskName = $VMName + "OSDisk"
```

### <a name="choose-a-sql-server-image"></a>Choisir une image SQL Server

Utilisez les variables suivantes pour définir l’image SQL Server à utiliser pour la machine virtuelle. 

1. Tout d’abord, listez toutes les offres d’image SQL Server avec la commande `Get-AzVMImageOffer`. Cette commande dresse la liste des images actuelles disponibles sur le portail Azure, ainsi que des images plus anciennes que vous ne pouvez installer qu'avec PowerShell :

   ```powershell
   Get-AzVMImageOffer -Location $Location -Publisher 'MicrosoftSQLServer'
   ```

1. Dans ce didacticiel, utilisez les variables suivantes pour spécifier SQL Server 2017 sur Windows Server 2016.

   ```powershell
   $OfferName = "SQL2017-WS2016"
   $PublisherName = "MicrosoftSQLServer"
   $Version = "latest"
   ```

1. Ensuite, listez toutes les éditions disponibles pour votre offre.

   ```powershell
   Get-AzVMImageSku -Location $Location -Publisher 'MicrosoftSQLServer' -Offer $OfferName | Select Skus
   ```

1. Dans ce didacticiel, utilisez l’édition Developer de SQL Server 2017 (**SQLDEV**). La Developer Edition est librement concédée sous licence pour le développement et le test, et vous ne payez que pour le coût d’exécution de la machine virtuelle.

   ```powershell
   $Sku = "SQLDEV"
   ```

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Avec le modèle de déploiement Resource Manager, le premier objet que vous créez est le groupe de ressources. Utilisez la cmdlet [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) pour créer un groupe de ressources Azure et ses ressources. Spécifiez les variables que vous avez précédemment initialisées pour le nom et l’emplacement du groupe de ressources.

Exécutez cette applet de commande pour créer votre groupe de ressources.

```powershell
New-AzResourceGroup -Name $ResourceGroupName -Location $Location
```

## <a name="create-a-storage-account"></a>Créez un compte de stockage.

La machine virtuelle nécessite des ressources de stockage pour le disque du système d’exploitation ainsi que pour les données et fichiers journaux de SQL Server. Pour plus de simplicité, vous allez créer un seul disque pour les deux. Vous pourrez attacher des disques supplémentaires ultérieurement, à l’aide de l’applet de commande [Add-Azure Disk](/powershell/module/servicemanagement/azure.service/add-azuredisk), pour placer vos données et fichiers journaux SQL Server sur des disques dédiés. Utilisez la cmdlet [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount) pour créer un compte de stockage standard dans votre nouveau groupe de ressources. Spécifiez les variables que vous avez précédemment initialisées pour le nom du compte de stockage, le nom de la référence SKU de stockage et l’emplacement.

Exécutez cette applet de commande pour créer votre compte de stockage.

```powershell
$StorageAccount = New-AzStorageAccount -ResourceGroupName $ResourceGroupName `
   -Name $StorageName -SkuName $StorageSku `
   -Kind "Storage" -Location $Location
```

> [!TIP]
> La création du compte de stockage peut prendre quelques minutes.

## <a name="create-network-resources"></a>Créer des ressources réseau

La machine virtuelle requiert plusieurs ressources réseau pour la connectivité réseau.

* Chaque machine virtuelle requiert un réseau virtuel.
* Un réseau virtuel doit avoir au moins un sous-réseau.
* Une interface réseau doit être définie avec une adresse IP privée ou publique.

### <a name="create-a-virtual-network-subnet-configuration"></a>Créer une configuration de sous-réseau de réseau virtuel

Commencez par créer une configuration de sous-réseau pour votre réseau virtuel. Dans le cadre de ce tutoriel, créez un sous-réseau par défaut à l’aide de la cmdlet [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig). Spécifiez les variables que vous avez précédemment initialisées pour le nom du sous-réseau et le préfixe d’adresse.

> [!NOTE]
> Vous pouvez définir des propriétés supplémentaires dans la configuration de sous-réseau de réseau virtuel à l’aide de cette applet de commande, mais cela sort du cadre de ce didacticiel.

Exécutez cette applet de commande pour créer votre configuration de sous-réseau virtuel.

```powershell
$SubnetConfig = New-AzVirtualNetworkSubnetConfig -Name $SubnetName -AddressPrefix $VNetSubnetAddressPrefix
```

### <a name="create-a-virtual-network"></a>Créez un réseau virtuel

Ensuite, créez votre réseau virtuel dans votre nouveau groupe de ressources à l’aide de la cmdlet [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork). Spécifiez les variables que vous avez précédemment initialisées pour le nom, l’emplacement et le préfixe d’adresse. Utilisez la configuration de sous-réseau que vous avez définie à l’étape précédente.

Exécutez cette applet de commande pour créer votre réseau virtuel.

```powershell
$VNet = New-AzVirtualNetwork -Name $VNetName `
   -ResourceGroupName $ResourceGroupName -Location $Location `
   -AddressPrefix $VNetAddressPrefix -Subnet $SubnetConfig
```

### <a name="create-the-public-ip-address"></a>Créer une adresse IP publique

Maintenant que votre réseau virtuel est défini, vous devez configurer une adresse IP pour la connectivité à la machine virtuelle. Dans le cadre de ce tutoriel, créez une adresse IP publique à l’aide de l’adressage IP dynamique pour prendre en charge la connectivité Internet. Utilisez la cmdlet [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) pour créer l’adresse IP publique dans votre nouveau groupe de ressources. Spécifiez les variables que vous avez précédemment initialisées pour le nom, l’emplacement, la méthode d’allocation et l’étiquette du nom de domaine DNS.

> [!NOTE]
> Vous pouvez définir des propriétés supplémentaires de l’adresse IP publique à l’aide de cette applet de commande, mais cela sort du cadre de ce didacticiel. Vous pouvez également créer une adresse privée ou une adresse avec une adresse statique, mais cela sort du cadre de ce didacticiel.

Exécutez cette applet de commande pour créer votre adresse IP publique.

```powershell
$PublicIp = New-AzPublicIpAddress -Name $InterfaceName `
   -ResourceGroupName $ResourceGroupName -Location $Location `
   -AllocationMethod $TCPIPAllocationMethod -DomainNameLabel $DomainName
```

### <a name="create-the-network-security-group"></a>Créer le groupe de sécurité réseau

Pour sécuriser le trafic de la machine virtuelle et de SQL Server, créez un groupe de sécurité réseau.

1. Tout d’abord, créez une règle de groupe de sécurité réseau pour le bureau à distance (RDP) pour autoriser les connexions RDP.

   ```powershell
   $NsgRuleRDP = New-AzNetworkSecurityRuleConfig -Name "RDPRule" -Protocol Tcp `
      -Direction Inbound -Priority 1000 -SourceAddressPrefix * -SourcePortRange * `
      -DestinationAddressPrefix * -DestinationPortRange 3389 -Access Allow
   ```
1. Configurez une règle de groupe de sécurité réseau qui autorise le trafic sur le port TCP 1433. Celle-ci permet de se connecter à SQL Server par Internet.

   ```powershell
   $NsgRuleSQL = New-AzNetworkSecurityRuleConfig -Name "MSSQLRule"  -Protocol Tcp `
      -Direction Inbound -Priority 1001 -SourceAddressPrefix * -SourcePortRange * `
      -DestinationAddressPrefix * -DestinationPortRange 1433 -Access Allow
   ```

1. Créez le groupe de sécurité réseau.

   ```powershell
   $Nsg = New-AzNetworkSecurityGroup -ResourceGroupName $ResourceGroupName `
      -Location $Location -Name $NsgName `
      -SecurityRules $NsgRuleRDP,$NsgRuleSQL
   ```

### <a name="create-the-network-interface"></a>Créer l’interface réseau

Vous êtes maintenant prêt à créer l’interface réseau de votre machine virtuelle. Utilisez la cmdlet [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface) pour créer l’interface réseau dans votre nouveau groupe de ressources. Spécifiez le nom, l’emplacement, le sous-réseau et l’adresse IP publique précédemment définis.

Exécutez cette applet de commande pour créer votre interface réseau.

```powershell
$Interface = New-AzNetworkInterface -Name $InterfaceName `
   -ResourceGroupName $ResourceGroupName -Location $Location `
   -SubnetId $VNet.Subnets[0].Id -PublicIpAddressId $PublicIp.Id `
   -NetworkSecurityGroupId $Nsg.Id
```

## <a name="configure-a-vm-object"></a>Configurer un objet de machine virtuelle

Maintenant que les ressources réseau et de stockage sont définies, vous pouvez définir les ressources de calcul de la machine virtuelle.

- Spécifiez la taille de la machine virtuelle et diverses propriétés de système d’exploitation.
- Spécifiez l’interface réseau que vous avez précédemment créée.
- Définissez le stockage d’objets blob.
- Spécifiez le disque du système d’exploitation.

### <a name="create-the-vm-object"></a>Créer l’objet de machine virtuelle

Commencez par spécifier la taille de la machine virtuelle. Dans le cadre de ce tutoriel, spécifiez DS13. Utilisez la cmdlet [New-AzVMConfig](/powershell/module/az.compute/new-azvmconfig) pour créer un objet de machine virtuelle configurable. Spécifiez les variables que vous avez précédemment initialisées pour le nom et la taille.

Exécutez cette applet de commande pour créer l’objet de machine virtuel.

```powershell
$VirtualMachine = New-AzVMConfig -VMName $VMName -VMSize $VMSize
```

### <a name="create-a-credential-object-to-hold-the-name-and-password-for-the-local-administrator-credentials"></a>Créer un objet d’informations d’identification pour stocker le nom et le mot de passe de l’administrateur local

Avant de définir les propriétés du système d’exploitation de la machine virtuelle, vous devez indiquer les informations d’identification du compte d’administrateur local sous la forme d’une chaîne de caractères sécurisée. Pour ce faire, utilisez l’applet de commande [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential).

Exécutez l’applet de commande suivante. Vous devez taper le nom et le mot de passe de l’administrateur local de la machine virtuelle dans la fenêtre de demande d’informations d’identification de PowerShell.

```powershell
$Credential = Get-Credential -Message "Type the name and password of the local administrator account."
```

### <a name="set-the-operating-system-properties-for-the-virtual-machine"></a>Configurer les propriétés du système d’exploitation de la machine virtuelle

Vous êtes maintenant prêt à définir les propriétés du système d’exploitation de la machine virtuelle avec la cmdlet [Set-AzVMOperatingSystem](/powershell/module/az.compute/set-azvmoperatingsystem).

- Définissez le type de système d’exploitation Windows.
- Exigez que l’[agent de machine virtuelle](../../../virtual-machines/extensions/agent-windows.md) soit installé.
- Spécifiez que l’applet de commande autorise la mise à jour automatique.
- Spécifiez les variables que vous avez précédemment initialisées pour le nom de la machine virtuelle, le nom de l’ordinateur et les informations d’identification.

Exécutez cette applet de commande pour définir les propriétés du système d’exploitation de votre machine virtuelle.

```powershell
$VirtualMachine = Set-AzVMOperatingSystem -VM $VirtualMachine `
   -Windows -ComputerName $ComputerName -Credential $Credential `
   -ProvisionVMAgent -EnableAutoUpdate
```

### <a name="add-the-network-interface-to-the-virtual-machine"></a>Ajouter l’interface réseau à la machine virtuelle

Ensuite, utilisez la cmdlet [Add-AzVMNetworkInterface](/powershell/module/az.compute/add-azvmnetworkinterface) pour ajouter l’interface réseau à l’aide de la variable que vous avez précédemment définie.

Exécutez cette applet de commande pour définir l’interface réseau de votre machine virtuelle.

```powershell
$VirtualMachine = Add-AzVMNetworkInterface -VM $VirtualMachine -Id $Interface.Id
```

### <a name="set-the-blob-storage-location-for-the-disk-to-be-used-by-the-virtual-machine"></a>Définir l’emplacement de Blob Storage du disque à utiliser par la machine virtuelle

Ensuite, définissez l’emplacement du stockage d’objets blob pour le disque de la machine virtuelle en utilisant les variables que vous avez définies précédemment.

Exécutez cette applet de commande pour définir l’emplacement du stockage d’objets blob.

```powershell
$OSDiskUri = $StorageAccount.PrimaryEndpoints.Blob.ToString() + "vhds/" + $OSDiskName + ".vhd"
```

### <a name="set-the-operating-system-disk-properties-for-the-virtual-machine"></a>Configurer les propriétés du disque du système d’exploitation de la machine virtuelle

Ensuite, définissez les propriétés du disque du système d’exploitation de la machine virtuelle à l’aide de la cmdlet [Set-AzVMOSDisk](/powershell/module/az.compute/set-azvmosdisk). 

- Spécifiez que le système d’exploitation de la machine virtuelle provient d’une image.
- Définissez une mise en cache en lecture seule (car SQL Server est installé sur le même disque).
- Spécifiez les variables que vous avez précédemment initialisées pour le nom de la machine virtuelle et le disque du système d’exploitation.

Exécutez cette applet de commande pour définir les propriétés du disque du système d’exploitation de votre machine virtuelle.

```powershell
$VirtualMachine = Set-AzVMOSDisk -VM $VirtualMachine -Name `
   $OSDiskName -VhdUri $OSDiskUri -Caching ReadOnly -CreateOption FromImage
```

### <a name="specify-the-platform-image-for-the-virtual-machine"></a>Spécifier l’image de plateforme de la machine virtuelle

La dernière étape de configuration consiste à spécifier l’image de plateforme de votre machine virtuelle. Dans le cadre de ce tutoriel, utilisez la dernière image de SQL Server 2016 CTP. Utilisez la cmdlet [Set-AzVMSourceImage](/powershell/module/az.compute/set-azvmsourceimage) pour utiliser cette image avec les variables que vous avez définies précédemment.

Exécutez cette applet de commande pour spécifier l’image de plateforme de votre machine virtuelle.

```powershell
$VirtualMachine = Set-AzVMSourceImage -VM $VirtualMachine `
   -PublisherName $PublisherName -Offer $OfferName `
   -Skus $Sku -Version $Version
```

## <a name="create-the-sql-vm"></a>Créer la machine virtuelle SQL

Maintenant que vous avez terminé les étapes de configuration, vous pouvez créer la machine virtuelle. Utilisez la cmdlet [New-AzVM](/powershell/module/az.compute/new-azvm) pour créer la machine virtuelle à l’aide des variables que vous avez définies.

> [!TIP]
> La création de la machine virtuelle peut prendre quelques minutes.

Exécutez cette applet de commande pour créer votre machine virtuelle.

```powershell
New-AzVM -ResourceGroupName $ResourceGroupName -Location $Location -VM $VirtualMachine
```

La machine virtuelle est créée.

> [!NOTE]
> Si vous obtenez une erreur liée aux diagnostics de démarrage, vous pouvez l’ignorer. Un compte de stockage standard est créé pour les diagnostics de démarrage, car le compte de stockage spécifié pour le disque de la machine virtuelle est un compte de stockage Premium.

## <a name="install-the-sql-iaas-agent"></a>Installer l’agent IaaS SQL

Les machines virtuelles SQL Server prennent en charge les fonctionnalités de gestion automatisées avec l’[Extension de l’Agent IaaS SQL Server](sql-server-iaas-agent-extension-automate-management.md). Pour inscrire votre instance SQL auprès de l’extension, exécutez la commande [New-AzSqlVM](/powershell/module/az.sqlvirtualmachine/new-azsqlvm) après la création de la machine virtuelle. Spécifiez le type de licence pour votre machine virtuelle SQL Server, en choisissant entre le paiement à l’utilisation ou BYOL par le biais d’[Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-benefit/). Pour plus d’informations sur les licences, consultez [Modèle de licence](licensing-model-azure-hybrid-benefit-ahb-change.md). 


   ```powershell
   New-AzSqlVM -ResourceGroupName $ResourceGroupName -Name $VMName -Location $Location -LicenseType <PAYG/AHUB> 
   ```

Il existe trois façons de s’inscrire auprès de l’extension : 
- [Automatiquement pour toutes les machines virtuelles actuelles et futures dans un abonnement](sql-agent-extension-automatic-registration-all-vms.md)
- [Manuellement pour une seule machine virtuelle](sql-agent-extension-manually-register-single-vm.md)
- [Manuellement pour plusieurs machines virtuelles en bloc](sql-agent-extension-manually-register-vms-bulk.md)


## <a name="stop-or-remove-a-vm"></a>Arrêter ou supprimer une machine virtuelle

Si vous n’avez pas besoin que la machine virtuelle fonctionne en permanence, vous pouvez éviter les frais inutiles en l’arrêtant quand vous ne vous en servez pas. La commande suivante arrête la machine virtuelle, tout en la laissant disponible pour une utilisation future.

```powershell
Stop-AzVM -Name $VMName -ResourceGroupName $ResourceGroupName
```

Vous pouvez aussi supprimer définitivement toutes les ressources associées à la machine virtuelle à l’aide de la commande **Remove-AzResourceGroup**. Dans la mesure où cette commande supprime la machine virtuelle de façon définitive, utilisez-la avec précaution.

## <a name="example-script"></a>Exemple de script

Le script suivant contient le script PowerShell complet pour ce didacticiel. Nous supposons que vous avez déjà configuré l’abonnement Azure à utiliser avec les commandes **Connect-AzAccount** et **Select-AzSubscription**.

```powershell
# Variables

## Global
$Location = "SouthCentralUS"
$ResourceGroupName = "sqlvm2"

## Storage
$StorageName = $ResourceGroupName + "storage"
$StorageSku = "Premium_LRS"

## Network
$InterfaceName = $ResourceGroupName + "ServerInterface"
$NsgName = $ResourceGroupName + "nsg"
$VNetName = $ResourceGroupName + "VNet"
$SubnetName = "Default"
$VNetAddressPrefix = "10.0.0.0/16"
$VNetSubnetAddressPrefix = "10.0.0.0/24"
$TCPIPAllocationMethod = "Dynamic"
$DomainName = $ResourceGroupName

##Compute
$VMName = $ResourceGroupName + "VM"
$ComputerName = $ResourceGroupName + "Server"
$VMSize = "Standard_DS13"
$OSDiskName = $VMName + "OSDisk"

##Image
$PublisherName = "MicrosoftSQLServer"
$OfferName = "SQL2017-WS2016"
$Sku = "SQLDEV"
$Version = "latest"

# Resource Group
New-AzResourceGroup -Name $ResourceGroupName -Location $Location

# Storage
$StorageAccount = New-AzStorageAccount -ResourceGroupName $ResourceGroupName -Name $StorageName -SkuName $StorageSku -Kind "Storage" -Location $Location

# Network
$SubnetConfig = New-AzVirtualNetworkSubnetConfig -Name $SubnetName -AddressPrefix $VNetSubnetAddressPrefix
$VNet = New-AzVirtualNetwork -Name $VNetName -ResourceGroupName $ResourceGroupName -Location $Location -AddressPrefix $VNetAddressPrefix -Subnet $SubnetConfig
$PublicIp = New-AzPublicIpAddress -Name $InterfaceName -ResourceGroupName $ResourceGroupName -Location $Location -AllocationMethod $TCPIPAllocationMethod -DomainNameLabel $DomainName
$NsgRuleRDP = New-AzNetworkSecurityRuleConfig -Name "RDPRule" -Protocol Tcp -Direction Inbound -Priority 1000 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * -DestinationPortRange 3389 -Access Allow
$NsgRuleSQL = New-AzNetworkSecurityRuleConfig -Name "MSSQLRule"  -Protocol Tcp -Direction Inbound -Priority 1001 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * -DestinationPortRange 1433 -Access Allow
$Nsg = New-AzNetworkSecurityGroup -ResourceGroupName $ResourceGroupName -Location $Location -Name $NsgName -SecurityRules $NsgRuleRDP,$NsgRuleSQL
$Interface = New-AzNetworkInterface -Name $InterfaceName -ResourceGroupName $ResourceGroupName -Location $Location -SubnetId $VNet.Subnets[0].Id -PublicIpAddressId $PublicIp.Id -NetworkSecurityGroupId $Nsg.Id

# Compute
$VirtualMachine = New-AzVMConfig -VMName $VMName -VMSize $VMSize
$Credential = Get-Credential -Message "Type the name and password of the local administrator account."
$VirtualMachine = Set-AzVMOperatingSystem -VM $VirtualMachine -Windows -ComputerName $ComputerName -Credential $Credential -ProvisionVMAgent -EnableAutoUpdate #-TimeZone = $TimeZone
$VirtualMachine = Add-AzVMNetworkInterface -VM $VirtualMachine -Id $Interface.Id
$OSDiskUri = $StorageAccount.PrimaryEndpoints.Blob.ToString() + "vhds/" + $OSDiskName + ".vhd"
$VirtualMachine = Set-AzVMOSDisk -VM $VirtualMachine -Name $OSDiskName -VhdUri $OSDiskUri -Caching ReadOnly -CreateOption FromImage

# Image
$VirtualMachine = Set-AzVMSourceImage -VM $VirtualMachine -PublisherName $PublisherName -Offer $OfferName -Skus $Sku -Version $Version

# Create the VM in Azure
New-AzVM -ResourceGroupName $ResourceGroupName -Location $Location -VM $VirtualMachine

# Add the SQL IaaS Extension, and choose the license type
New-AzSqlVM -ResourceGroupName $ResourceGroupName -Name $VMName -Location $Location -LicenseType <PAYG/AHUB> 
```

## <a name="next-steps"></a>Étapes suivantes

Une fois la machine virtuelle créée, vous pouvez :

- Vous connecter à la machine virtuelle à l’aide du protocole RDP (Remote Desktop Protocol)
- Configurer les paramètres de SQL Server dans le portail de votre machine virtuelle, notamment :
   - les [paramètres de stockage](storage-configuration.md) ; 
   - les [tâche de gestion automatisées](sql-server-iaas-agent-extension-automate-management.md).
- [Configurer la connectivité](ways-to-connect-to-sql.md)
- Connecter des clients et applications à la nouvelle instance de SQL Server