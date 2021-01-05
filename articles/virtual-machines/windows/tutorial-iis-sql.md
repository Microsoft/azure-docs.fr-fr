---
title: 'Tutoriel : Créer des machines virtuelles exécutant une pile SQL, IIS ou .NET dans Azure'
description: Dans ce tutoriel, vous allez apprendre à installer la pile Azure SQL, IIS, .NET sur une machine virtuelle Windows dans Azure.
author: cynthn
ms.service: virtual-machines-windows
ms.topic: tutorial
ms.workload: infrastructure
ms.date: 12/05/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 1c53194bd345c18ac582acd538f1e8f8e1e34d54
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87027850"
---
# <a name="tutorial-install-the-sql-iis-net-stack-in-a-windows-vm-with-azure-powershell"></a>Tutoriel : Installer la pile SQL, IIS, .NET dans une machine virtuelle Windows avec Azure PowerShell

Dans ce tutoriel, nous installons une pile SQL, IIS, .NET avec Azure PowerShell. Cette pile se compose de deux machines virtuelles exécutant Windows Server 2016 : une avec IIS et .NET, et l’autre avec SQL Server.

> [!div class="checklist"]
> * Créer une machine virtuelle 
> * Installer IIS et le kit SDK .NET Core sur la machine virtuelle
> * Créer une machine virtuelle exécutant SQL Server
> * Installer l’extension SQL Server

## <a name="launch-azure-cloud-shell"></a>Lancement d’Azure Cloud Shell

Azure Cloud Shell est un interpréteur de commandes interactif et gratuit que vous pouvez utiliser pour exécuter les étapes de cet article. Il contient des outils Azure courants préinstallés et configurés pour être utilisés avec votre compte. 

Pour ouvrir Cloud Shell, sélectionnez simplement **Essayer** en haut à droite d’un bloc de code. Vous pouvez également lancer Cloud Shell dans un onglet distinct du navigateur en accédant à [https://shell.azure.com/powershell](https://shell.azure.com/powershell). Sélectionnez **Copier** pour copier les blocs de code, collez-les dans Cloud Shell, puis appuyez sur Entrée pour les exécuter.

## <a name="create-an-iis-vm"></a>Créer une machine virtuelle IIS 

Dans cet exemple, nous utilisons l’applet de commande [New-AzVM](/powershell/module/az.compute/new-azvm) de PowerShell Cloud Shell pour créer rapidement une machine virtuelle Windows Server 2016, puis installer IIS et le .NET Framework. Les machines virtuelles IIS et SQL partagent un groupe de ressources et le réseau virtuel. Nous allons donc créer des variables pour ces noms.


```azurepowershell-interactive
$vmName = "IISVM"
$vNetName = "myIISSQLvNet"
$resourceGroup = "myIISSQLGroup"
New-AzVm `
    -ResourceGroupName $resourceGroup `
    -Name $vmName `
    -Location "East US" `
    -VirtualNetworkName $vNetName `
    -SubnetName "myIISSubnet" `
    -SecurityGroupName "myNetworkSecurityGroup" `
    -AddressPrefix 192.168.0.0/16 `
    -PublicIpAddressName "myIISPublicIpAddress" `
    -OpenPorts 80,3389 
```

Installez IIS et le .NET Framework avec l’extension de script personnalisé à l’aide de l’applet de commande [Set-AzVMExtension](/powershell/module/az.compute/set-azvmextension).

```azurepowershell-interactive
Set-AzVMExtension `
    -ResourceGroupName $resourceGroup `
    -ExtensionName IIS `
    -VMName $vmName `
    -Publisher Microsoft.Compute `
    -ExtensionType CustomScriptExtension `
    -TypeHandlerVersion 1.4 `
    -SettingString '{"commandToExecute":"powershell Add-WindowsFeature Web-Server,Web-Asp-Net45,NET-Framework-Features"}' `
    -Location EastUS
```

## <a name="create-another-subnet"></a>Créer un autre sous-réseau

Créez un second sous-réseau pour la machine virtuelle SQL. Obtenez le réseau virtuel à l’aide de [Get-AzVirtualNetwork]{/powershell/module/az.network/get-azvirtualnetwork}.

```azurepowershell-interactive
$vNet = Get-AzVirtualNetwork `
   -Name $vNetName `
   -ResourceGroupName $resourceGroup
```

Créez une configuration pour le sous-réseau en utilisant [Add-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/add-azvirtualnetworksubnetconfig).


```azurepowershell-interactive
Add-AzVirtualNetworkSubnetConfig `
   -AddressPrefix 192.168.0.0/24 `
   -Name mySQLSubnet `
   -VirtualNetwork $vNet `
   -ServiceEndpoint Microsoft.Sql
```

Mettre à jour le réseau virtuel avec les informations du nouveau sous-réseau en utilisant [Set-AzVirtualNetwork](/powershell/module/az.network/set-azvirtualnetwork)
   
```azurepowershell-interactive   
$vNet | Set-AzVirtualNetwork
```

## <a name="azure-sql-vm"></a>Machine virtuelle Azure SQL

Utilisez une image préconfigurée d’Azure marketplace d’un serveur SQL Server pour créer la machine virtuelle SQL. Nous créons d’abord la machine virtuelle, puis nous installons l’extension SQL Server dessus. 


```azurepowershell-interactive
New-AzVm `
    -ResourceGroupName $resourceGroup `
    -Name "mySQLVM" `
    -ImageName "MicrosoftSQLServer:SQL2016SP1-WS2016:Enterprise:latest" `
    -Location eastus `
    -VirtualNetworkName $vNetName `
    -SubnetName "mySQLSubnet" `
    -SecurityGroupName "myNetworkSecurityGroup" `
    -PublicIpAddressName "mySQLPublicIpAddress" `
    -OpenPorts 3389,1401 
```

Utilisez [Set-AzVMSqlServerExtension](/powershell/module/az.compute/set-azvmsqlserverextension) pour ajouter l’[extension SQL Server](../../azure-sql/virtual-machines/windows/sql-server-iaas-agent-extension-automate-management.md) à la machine virtuelle SQL.

```azurepowershell-interactive
Set-AzVMSqlServerExtension `
   -ResourceGroupName $resourceGroup  `
   -VMName mySQLVM `
   -Name "SQLExtension" `
   -Location "EastUS"
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez installé une pile SQL&#92;IIS&#92;.NET avec Azure PowerShell. Vous avez appris à :

> [!div class="checklist"]
> * Créer une machine virtuelle 
> * Installer IIS et le kit SDK .NET Core sur la machine virtuelle
> * Créer une machine virtuelle exécutant SQL Server
> * Installer l’extension SQL Server

Passez au tutoriel suivant pour savoir comment sécuriser un serveur web IIS avec des certificats TLS/SSL.

> [!div class="nextstepaction"]
> [Sécuriser un serveur web IIS à l’aide de certificats TLS/SSL](tutorial-secure-web-server.md)
