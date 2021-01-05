---
title: Créer un serveur SQL Server sur une machine virtuelle Windows avec Azure PowerShell | Microsoft Docs
description: Ce tutoriel montre comment utiliser Azure PowerShell pour créer une machine virtuelle Windows exécutant SQL Server 2017.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
tags: azure-resource-manager
ms.service: virtual-machines-sql
ms.subservice: deployment
ms.topic: quickstart
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: infrastructure-services
ms.date: 12/21/2018
ms.author: mathoma
ms.reviewer: jroth
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 350df1aede076120fd14dc46da351835c894c5b4
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97356835"
---
# <a name="quickstart-create-sql-server-on-a-windows-virtual-machine-with-azure-powershell"></a>Démarrage rapide : Créer un serveur SQL Server sur une machine virtuelle Windows avec Azure PowerShell

[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Ce guide de démarrage rapide décrit les étapes de création d’une machine virtuelle SQL Server avec Azure PowerShell.

> [!TIP]
> - Ce démarrage rapide vous présente les étapes de mise en service et de connexion rapide d’une machine virtuelle SQL. Pour plus d’informations sur les autres options Azure PowerShell pour la création de machines virtuelles SQL, consultez le [guide de mise en service des machines virtuelles SQL Server avec Azure PowerShell](create-sql-vm-powershell.md).
> - Si vous avez des questions sur les machines virtuelles SQL Server, consultez le [Forum aux Questions](frequently-asked-questions-faq.md).

## <a name="get-an-azure-subscription"></a><a id="subscription"></a> Obtenir un abonnement Azure

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.


## <a name="get-azure-powershell"></a><a id="powershell"></a> Obtenir Azure PowerShell

[!INCLUDE [updated-for-az.md](../../../../includes/updated-for-az.md)]

## <a name="configure-powershell"></a>Configurer PowerShell

1. Ouvrez PowerShell et accédez à votre compte Azure en exécutant la commande **Connect-AzAccount**.

   ```powershell
   Connect-AzAccount
   ```

1. Quand la fenêtre de connexion s’affiche, entrez vos informations d’identification. Utilisez l'adresse électronique et le mot de passe que vous utilisez pour vous connecter au portail Azure.

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

1. Définissez une variable avec un nom unique de groupe de ressources. Pour simplifier la suite de ce guide de démarrage rapide, les commandes restantes utilisent ce nom comme base pour d’autres noms de ressources.

   ```powershell
   $ResourceGroupName = "sqlvm1"
   ```

1. Définissez un emplacement de région Azure cible pour l’ensemble des ressources de machine virtuelle.

   ```powershell
   $Location = "East US"
   ```

1. Créez le groupe de ressources.

   ```powershell
   New-AzResourceGroup -Name $ResourceGroupName -Location $Location
   ```

## <a name="configure-network-settings"></a>Configurer les paramètres réseau

1. Créez un réseau virtuel, un sous-réseau et une adresse IP publique. Ces ressources sont utilisées pour fournir une connectivité réseau à la machine virtuelle et la connecter à Internet.

   ``` PowerShell
   $SubnetName = $ResourceGroupName + "subnet"
   $VnetName = $ResourceGroupName + "vnet"
   $PipName = $ResourceGroupName + $(Get-Random)

   # Create a subnet configuration
   $SubnetConfig = New-AzVirtualNetworkSubnetConfig -Name $SubnetName -AddressPrefix 192.168.1.0/24

   # Create a virtual network
   $Vnet = New-AzVirtualNetwork -ResourceGroupName $ResourceGroupName -Location $Location `
      -Name $VnetName -AddressPrefix 192.168.0.0/16 -Subnet $SubnetConfig

   # Create a public IP address and specify a DNS name
   $Pip = New-AzPublicIpAddress -ResourceGroupName $ResourceGroupName -Location $Location `
      -AllocationMethod Static -IdleTimeoutInMinutes 4 -Name $PipName
   ```

1. Créer des groupes de sécurité réseau Configurez des règles autorisant les connexions du Bureau à distance et de SQL Server.

   ```powershell
   # Rule to allow remote desktop (RDP)
   $NsgRuleRDP = New-AzNetworkSecurityRuleConfig -Name "RDPRule" -Protocol Tcp `
      -Direction Inbound -Priority 1000 -SourceAddressPrefix * -SourcePortRange * `
      -DestinationAddressPrefix * -DestinationPortRange 3389 -Access Allow

   #Rule to allow SQL Server connections on port 1433
   $NsgRuleSQL = New-AzNetworkSecurityRuleConfig -Name "MSSQLRule"  -Protocol Tcp `
      -Direction Inbound -Priority 1001 -SourceAddressPrefix * -SourcePortRange * `
      -DestinationAddressPrefix * -DestinationPortRange 1433 -Access Allow

   # Create the network security group
   $NsgName = $ResourceGroupName + "nsg"
   $Nsg = New-AzNetworkSecurityGroup -ResourceGroupName $ResourceGroupName `
      -Location $Location -Name $NsgName `
      -SecurityRules $NsgRuleRDP,$NsgRuleSQL
   ```

1. Créez l’interface réseau.

   ```powershell
   $InterfaceName = $ResourceGroupName + "int"
   $Interface = New-AzNetworkInterface -Name $InterfaceName `
      -ResourceGroupName $ResourceGroupName -Location $Location `
      -SubnetId $VNet.Subnets[0].Id -PublicIpAddressId $Pip.Id `
      -NetworkSecurityGroupId $Nsg.Id
   ```

## <a name="create-the-sql-vm"></a>Créer la machine virtuelle SQL

1. Définissez les informations d’identification qui vous permettent de vous connecter à la machine virtuelle. Le nom d’utilisateur est « azureadmin ». Veillez à remplacer \<password> avant d’exécuter la commande.

   ``` PowerShell
   # Define a credential object
   $SecurePassword = ConvertTo-SecureString '<password>' `
      -AsPlainText -Force
   $Cred = New-Object System.Management.Automation.PSCredential ("azureadmin", $securePassword)
   ```

1. Créez un objet de configuration de machine virtuelle, puis créez la machine virtuelle. La commande suivante crée une machine virtuelle SQL Server 2017 Developer Edition sur Windows Server 2016.

   ```powershell
   # Create a virtual machine configuration
   $VMName = $ResourceGroupName + "VM"
   $VMConfig = New-AzVMConfig -VMName $VMName -VMSize Standard_DS13_V2 |
      Set-AzVMOperatingSystem -Windows -ComputerName $VMName -Credential $Cred -ProvisionVMAgent -EnableAutoUpdate |
      Set-AzVMSourceImage -PublisherName "MicrosoftSQLServer" -Offer "SQL2017-WS2016" -Skus "SQLDEV" -Version "latest" |
      Add-AzVMNetworkInterface -Id $Interface.Id
   
   # Create the VM
   New-AzVM -ResourceGroupName $ResourceGroupName -Location $Location -VM $VMConfig
   ```

   > [!TIP]
   > La création de la machine virtuelle nécessite quelques minutes.

## <a name="register-with-sql-vm-rp"></a>S’inscrire auprès de SQL VM RP 

Pour connaître les fonctionnalités d’intégration au portail et de machine virtuelle SQL, vous devez vous inscrire auprès de l’[extension SQL IaaS Agent](sql-agent-extension-manually-register-single-vm.md).

Pour obtenir toutes les fonctionnalités, vous devez vous inscrire auprès de l’extension en mode complet. Toutefois, cette opération redémarre le service SQL Server. L’approche recommandée consiste donc à s’inscrire en mode léger, puis à effectuer une mise à niveau vers le mode complet au cours d’une fenêtre de maintenance. 

Inscrivez d’abord la machine virtuelle SQL Server en mode léger : 

```powershell-interactive
# Get the existing compute VM
$vm = Get-AzVM -Name <vm_name> -ResourceGroupName <resource_group_name>
        
# Register SQL VM with 'Lightweight' SQL IaaS agent
New-AzSqlVM -Name $vm.Name -ResourceGroupName $vm.ResourceGroupName -Location $vm.Location `
  -LicenseType PAYG -SqlManagementType LightWeight
```

Ensuite, pendant une fenêtre de maintenance, effectuez une mise à niveau vers le mode complet : 

```powershell-interactive
# Get the existing Compute VM
$vm = Get-AzVM -Name <vm_name> -ResourceGroupName <resource_group_name>
      
# Register with SQL IaaS Agent extension in full mode
Update-AzSqlVM -Name $vm.Name -ResourceGroupName $vm.ResourceGroupName -SqlManagementType Full
```



## <a name="remote-desktop-into-the-vm"></a>Bureau à distance dans la machine virtuelle

1. La commande suivante permet de récupérer l’adresse IP publique de la nouvelle machine virtuelle.

   ```powershell
   Get-AzPublicIpAddress -ResourceGroupName $ResourceGroupName | Select IpAddress
   ```

1. Passez l’adresse IP retournée en tant que paramètre de ligne de commande à **mstsc** pour démarrer une session Bureau à distance dans la nouvelle machine virtuelle.

   ```
   mstsc /v:<publicIpAddress>
   ```

1. Lorsque l’invite de saisie des informations d’identification s’ouvre, entrez les données correspondant à un compte différent. Entrez le nom d’utilisateur en le faisant précéder d’une barre oblique inverse (par exemple `\azureadmin`) ainsi que le mot de passe défini dans ce guide de démarrage rapide.

## <a name="connect-to-sql-server"></a>Se connecter à SQL Server

1. Une fois connecté à la session Bureau à distance, lancez **SQL Server Management Studio 2017** à partir du menu Démarrer.

1. Dans la boîte de dialogue **Se connecter au serveur**, conservez les valeurs par défaut. Le nom du serveur est le nom de la machine virtuelle. L’authentification est définie sur **Authentification Windows**. Sélectionnez **Connecter**.

Vous êtes désormais connecté localement à SQL Server. Pour vous connecter à distance, vous devez [configurer la connectivité](ways-to-connect-to-sql.md) à partir du portail Azure ou manuellement.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous n’avez pas besoin que la machine virtuelle fonctionne en permanence, vous pouvez éviter les frais inutiles en l’arrêtant quand vous ne vous en servez pas. La commande suivante arrête la machine virtuelle, tout en la laissant disponible pour une utilisation future.

```powershell
Stop-AzVM -Name $VMName -ResourceGroupName $ResourceGroupName
```

Vous pouvez aussi supprimer définitivement toutes les ressources associées à la machine virtuelle à l’aide de la commande **Remove-AzResourceGroup**. Dans la mesure où cette commande supprime la machine virtuelle de façon définitive, utilisez-la avec précaution.

## <a name="next-steps"></a>Étapes suivantes

Dans ce démarrage rapide, vous avez créé une machine virtuelle SQL Server 2017 à l’aide d’Azure PowerShell. Pour en savoir plus sur la façon de migrer vos données vers la nouvelle instance SQL Server, consultez l’article suivant.

> [!div class="nextstepaction"]
> [Migrer une base de données SQL Server vers SQL Server dans une machine virtuelle Azure](migrate-to-vm-from-sql-server.md)
