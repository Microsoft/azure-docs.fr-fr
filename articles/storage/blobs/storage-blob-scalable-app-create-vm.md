---
title: Créer une machine virtuelle et un compte de stockage pour une application scalable dans Azure
description: Découvrir comment déployer une machine virtuelle à utiliser pour exécuter une application évolutive à l’aide du Stockage blob Azure
author: roygara
ms.service: storage
ms.topic: tutorial
ms.date: 02/20/2018
ms.author: rogarana
ms.subservice: blobs
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 4b8c52b03cb6dec6096565e9eac26b7b2c4a30e4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89073248"
---
# <a name="create-a-virtual-machine-and-storage-account-for-a-scalable-application"></a>Créer une machine virtuelle et un compte de stockage pour une application évolutive

Ce tutoriel est la première partie d’une série d’étapes. Ce didacticiel décrit le déploiement d’une application qui charge et télécharge une grande quantité de données aléatoires avec un compte de stockage Azure. Une fois que vous avez terminé, vous avez une application console qui s’exécute sur une machine virtuelle à partir de laquelle vous chargez et téléchargez de grandes quantités de données dans un compte de stockage.

Dans ce premier volet, vous apprenez à :

> [!div class="checklist"]
> * Créez un compte de stockage.
> * Création d'une machine virtuelle
> * Configurer une extension de script personnalisé

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Si vous choisissez d’installer et d’utiliser PowerShell en local, vous devez exécuter le module Azure PowerShell Az version 0.7 ou version ultérieure pour les besoins de ce tutoriel. Exécutez `Get-Module -ListAvailable Az` pour trouver la version. Si vous devez effectuer une mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-Az-ps). Si vous exécutez PowerShell en local, vous devez également lancer `Connect-AzAccount` pour créer une connexion avec Azure.

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Créez un groupe de ressources Azure avec [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Un groupe de ressources est un conteneur logique dans lequel les ressources Azure sont déployées et gérées.

```azurepowershell-interactive
New-AzResourceGroup -Name myResourceGroup -Location EastUS
```

## <a name="create-a-storage-account"></a>Créez un compte de stockage.
 
L’exemple charge 50 fichiers volumineux dans un conteneur blob sur un compte de stockage Azure. Le compte de stockage Azure fournit un espace de noms unique pour stocker les objets de données de Stockage Azure et y accéder. Créez un compte de stockage dans le groupe de ressources que vous avez créé à l’aide de la commande [New-AzStorageAccount](/powershell/module/az.Storage/New-azStorageAccount).

Dans la commande suivante, indiquez le nom global unique de votre compte de stockage Blob dans l’espace réservé `<blob_storage_account>`.

```powershell-interactive
$storageAccount = New-AzStorageAccount -ResourceGroupName myResourceGroup `
  -Name "<blob_storage_account>" `
  -Location EastUS `
  -SkuName Standard_LRS `
  -Kind Storage `
```

## <a name="create-a-virtual-machine"></a>Création d'une machine virtuelle

Créez une configuration de machine virtuelle. Cette configuration inclut les paramètres qui sont utilisés lors du déploiement de la machine virtuelle, comme une image de machine virtuelle, la taille et la configuration de l’authentification. Lors de l’exécution de cette étape, vous êtes invité à saisir vos informations d’identification. Les valeurs que vous saisissez sont configurées comme le nom d’utilisateur et le mot de passe pour la machine virtuelle.

Créez la machine virtuelle avec [New-AzVM](/powershell/module/az.compute/new-azvm).

```azurepowershell-interactive
# Variables for common values
$resourceGroup = "myResourceGroup"
$location = "eastus"
$vmName = "myVM"

# Create user object
$cred = Get-Credential -Message "Enter a username and password for the virtual machine."

# Create a subnet configuration
$subnetConfig = New-AzVirtualNetworkSubnetConfig -Name mySubnet -AddressPrefix 192.168.1.0/24

# Create a virtual network
$vnet = New-AzVirtualNetwork -ResourceGroupName $resourceGroup -Location $location `
  -Name MYvNET -AddressPrefix 192.168.0.0/16 -Subnet $subnetConfig

# Create a public IP address and specify a DNS name
$pip = New-AzPublicIpAddress -ResourceGroupName $resourceGroup -Location $location `
  -Name "mypublicdns$(Get-Random)" -AllocationMethod Static -IdleTimeoutInMinutes 4

# Create a virtual network card and associate with public IP address
$nic = New-AzNetworkInterface -Name myNic -ResourceGroupName $resourceGroup -Location $location `
  -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id

# Create a virtual machine configuration
$vmConfig = New-AzVMConfig -VMName myVM -VMSize Standard_DS14_v2 | `
    Set-AzVMOperatingSystem -Windows -ComputerName myVM -Credential $cred | `
    Set-AzVMSourceImage -PublisherName MicrosoftWindowsServer -Offer WindowsServer `
    -Skus 2016-Datacenter -Version latest | Add-AzVMNetworkInterface -Id $nic.Id

# Create a virtual machine
New-AzVM -ResourceGroupName $resourceGroup -Location $location -VM $vmConfig

Write-host "Your public IP address is $($pip.IpAddress)"
```

## <a name="deploy-configuration"></a>Déployer la configuration

Pour ce didacticiel, vous devez installer des prérequis sur la machine virtuelle. L’extension de script personnalisé est utilisée pour exécuter un script PowerShell qui effectue les tâches suivantes :

> [!div class="checklist"]
> * Installer .NET Core 2.0
> * Installer chocolatey
> * Installer GIT
> * Cloner l’exemple de dépôt
> * Restaurer des packages NuGet
> * Créer 50 fichiers de 1 Go avec des données aléatoires

Exécutez l’applet de commande suivante pour finaliser la configuration de la machine virtuelle. Cette étape dure 5 à 15 minutes.

```azurepowershell-interactive
# Start a CustomScript extension to use a simple PowerShell script to install .NET core, dependencies, and pre-create the files to upload.
Set-AzVMCustomScriptExtension -ResourceGroupName myResourceGroup `
    -VMName myVM `
    -Location EastUS `
    -FileUri https://raw.githubusercontent.com/azure-samples/storage-dotnet-perf-scale-app/master/setup_env.ps1 `
    -Run 'setup_env.ps1' `
    -Name DemoScriptExtension
```

## <a name="next-steps"></a>Étapes suivantes

Dans la première partie de la série, vous avez appris à créer un compte de stockage, déployer une machine virtuelle et configurer la machine virtuelle avec les prérequis nécessaires, notamment comment :

> [!div class="checklist"]
> * Créez un compte de stockage.
> * Création d'une machine virtuelle
> * Configurer une extension de script personnalisé

Passez à la deuxième partie de la série pour charger de grandes quantités de données dans un compte de stockage à l’aide d’un parallélisme et d’un nombre de nouvelles tentatives exponentiels.

> [!div class="nextstepaction"]
> [Charger en parallèle de grandes quantités de fichiers volumineux dans un compte de stockage](storage-blob-scalable-app-upload-files.md)
