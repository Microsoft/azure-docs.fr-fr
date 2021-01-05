---
title: Créer et chiffrer une machine virtuelle Windows avec Azure PowerShell
description: Dans ce guide de démarrage rapide, vous allez apprendre à utiliser Azure PowerShell pour créer et chiffrer une machine virtuelle Windows
author: msmbaldwin
ms.author: mbaldwin
ms.service: virtual-machines-windows
ms.subservice: security
ms.topic: quickstart
ms.date: 05/17/2019
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: a50b62a7c6064672dfbf7d609b6053d7be6fdb77
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "89079487"
---
# <a name="quickstart-create-and-encrypt-a-windows-virtual-machine-in-azure-with-powershell"></a>Démarrage rapide : Créer et chiffrer une machine virtuelle Windows dans Azure avec PowerShell

Le module Azure PowerShell est utilisé pour créer et gérer des ressources Azure à partir de la ligne de commande PowerShell ou dans des scripts. Ce guide de démarrage rapide vous montre comment utiliser le module Azure PowerShell pour créer une machine virtuelle Windows, créer un coffre de clés pour le stockage des clés de chiffrement, et chiffrer la machine virtuelle. 

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.


## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Créez un groupe de ressources Azure avec [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Un groupe de ressources est un conteneur logique dans lequel les ressources Azure sont déployées et gérées :

```powershell
New-AzResourceGroup -Name "myResourceGroup" -Location "EastUS"
```

## <a name="create-a-virtual-machine"></a>Création d'une machine virtuelle

Créez une machine virtuelle Azure avec [New-AzVM](/powershell/module/az.compute/new-azvm). Vous devez fournir les informations d’identification à l’applet de commande. 

```powershell
$cred = Get-Credential 

New-AzVM -Name MyVm -Credential $cred -ResourceGroupName MyResourceGroup -Image win2016datacenter -Size Standard_D2S_V3
```

Quelques minutes sont nécessaires pour le déploiement de votre machine virtuelle. 

## <a name="create-a-key-vault-configured-for-encryption-keys"></a>Créer un coffre de clés configuré pour les clés de chiffrement

Azure Disk Encryption stocke sa clé de chiffrement dans Azure Key Vault. Créez un coffre de clés avec [New-AzKeyvault](/powershell/module/az.keyvault/new-azkeyvault). Pour activer le stockage des clés de chiffrement dans Key Vault, utilisez le paramètre -EnabledForDiskEncryption.

> [!Important]
> Chaque coffre de clés doit avoir un nom unique. L’exemple suivant crée un coffre de clés nommé *myKV*, mais vous devez nommer le vôtre différemment.

```powershell
New-AzKeyvault -name MyKV -ResourceGroupName myResourceGroup -Location EastUS -EnabledForDiskEncryption
```

## <a name="encrypt-the-virtual-machine"></a>Chiffrer la machine virtuelle

Chiffrez votre machine virtuelle avec [Set-AzVmDiskEncryptionExtension](/powershell/module/az.compute/set-azvmdiskencryptionextension). 

Set-AzVmDiskEncryptionExtension requiert des valeurs de votre objet de coffre de clés. Vous pouvez obtenir ces valeurs en passant le nom unique de votre coffre de clés à [Get-AzKeyvault](/powershell/module/az.keyvault/get-azkeyvault).

```powershell
$KeyVault = Get-AzKeyVault -VaultName MyKV -ResourceGroupName MyResourceGroup

Set-AzVMDiskEncryptionExtension -ResourceGroupName MyResourceGroup -VMName MyVM -DiskEncryptionKeyVaultUrl $KeyVault.VaultUri -DiskEncryptionKeyVaultId $KeyVault.ResourceId
```

Après quelques minutes, le processus retourne les éléments suivants :

```
RequestId IsSuccessStatusCode StatusCode ReasonPhrase
--------- ------------------- ---------- ------------
                         True         OK OK
```

Vous pouvez vérifier le processus de chiffrement en exécutant [Get-AzVmDiskEncryptionStatus](/powershell/module/az.compute/Get-AzVMDiskEncryptionStatus).

```powershell
Get-AzVmDiskEncryptionStatus -VMName MyVM -ResourceGroupName MyResourceGroup
```

Lorsque le chiffrement est activé, vous verrez les éléments suivants dans la sortie retournée :

```
OsVolumeEncrypted          : Encrypted
DataVolumesEncrypted       : NoDiskFound
OsVolumeEncryptionSettings : Microsoft.Azure.Management.Compute.Models.DiskEncryptionSettings
ProgressMessage            : Provisioning succeeded
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, vous pouvez utiliser la cmdlet [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) pour supprimer le groupe de ressources, la machine virtuelle et toutes les ressources associées :

```powershell
Remove-AzResourceGroup -Name "myResourceGroup"
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé une machine virtuelle, créé un coffre de clés qui a été activé pour les clés de chiffrement, et vous avez chiffré la machine virtuelle.  Passez à l’article suivant pour en savoir plus sur la configuration requise d’Azure Disk Encryption pour les machines virtuelles IaaS.

> [!div class="nextstepaction"]
> [Vue d’ensemble d’Azure Disk Encryption](disk-encryption-overview.md)
