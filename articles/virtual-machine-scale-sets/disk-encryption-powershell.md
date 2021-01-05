---
title: Chiffrer des disques pour des groupes identiques Azure avec Azure PowerShell
description: Découvrez comment utiliser Azure PowerShell pour chiffrer les instances de machine virtuelle et les disques attachés dans un groupe de machines virtuelles identiques Windows
author: ju-shim
ms.author: jushiman
ms.topic: how-to
ms.service: virtual-machine-scale-sets
ms.subservice: disks
ms.date: 10/15/2019
ms.reviewer: mimckitt
ms.custom: mimckitt, devx-track-azurepowershell
ms.openlocfilehash: cac2f57ccb5fd5d0aa251533bd4a5dd1179ec058
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89069747"
---
# <a name="encrypt-os-and-attached-data-disks-in-a-virtual-machine-scale-set-with-azure-powershell"></a>Chiffrer des disques de données joints et de systèmes d’exploitation dans un groupe de machines virtuelles identiques avec Azure PowerShell

Le module Azure PowerShell est utilisé pour créer et gérer des ressources Azure à partir de la ligne de commande PowerShell ou dans des scripts.  Cet article vous montre comment utiliser Azure PowerShell pour créer et chiffrer un groupe de machines virtuelles identiques. Pour plus d’informations sur l’application d’Azure Disk Encryption à un groupe de machines virtuelles identiques, consultez [Azure Disk Encryption pour les groupes de machines virtuelles identiques](disk-encryption-overview.md).

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-an-azure-key-vault-enabled-for-disk-encryption"></a>Créer un coffre de clés Azure activé pour le chiffrement de disque

Un coffre de clés Azure peut stocker des clés, des clés secrètes ou des mots de passe vous permettant de les implémenter en toute sécurité dans vos applications et services. Les clés de chiffrement sont stockées dans le coffre de clés Azure à l’aide d’une protection logicielle, mais vous pouvez importer ou générer vos clés dans des modules de sécurité matériels (HSM) certifiés conformes aux normes FIPS 140-2 de niveau 2. Ces clés de chiffrement servent à chiffrer et à déchiffrer les disques virtuels connectés à votre machine virtuelle. Vous gardez le contrôle de ces clés de chiffrement et pouvez effectuer un audit de leur utilisation.

Créez un coffre de clés avec [New-AzKeyVault](/powershell/module/az.keyvault/new-azkeyvault). Pour autoriser l’utilisation du coffre de clés pour le chiffrement de disque, définissez le paramètre *EnabledForDiskEncryption*. L’exemple suivant définit également les variables pour le nom du groupe de ressources, le nom du coffre de clés et l’emplacement. Indiquez un nom unique pour votre coffre de clés :

```azurepowershell-interactive
$rgName="myResourceGroup"
$vaultName="myuniquekeyvault"
$location = "EastUS"

New-AzResourceGroup -Name $rgName -Location $location
New-AzKeyVault -VaultName $vaultName -ResourceGroupName $rgName -Location $location -EnabledForDiskEncryption
```

### <a name="use-an-existing-key-vault"></a>Utiliser un coffre de clés existant

Cette étape est requise uniquement si vous souhaitez utiliser un coffre de clés existant avec le chiffrement de disque. Ignorez cette étape si vous avez créé un coffre de clés dans la section précédente.

Vous pouvez activer un coffre de clés existant dans le même abonnement et la même région que le groupe identique pour le chiffrement de disque avec [Set-AzKeyVaultAccessPolicy](/powershell/module/az.keyvault/Set-AzKeyVaultAccessPolicy). Définissez le nom du coffre de clés existant dans la variable *$vaultName*, comme ceci :


```azurepowershell-interactive
$vaultName="myexistingkeyvault"
Set-AzKeyVaultAccessPolicy -VaultName $vaultName -EnabledForDiskEncryption
```

## <a name="create-a-scale-set"></a>Créer un groupe identique

Tout d’abord, définissez un nom d’utilisateur administrateur et un mot de passe pour les instances de machine virtuelle avec [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential?view=powershell-5.1) :

```azurepowershell-interactive
$cred = Get-Credential
```

À présent, créez un groupe de machines virtuelles identiques avec [New-AzVmss](/powershell/module/az.compute/new-azvmss). Pour distribuer le trafic aux différentes instances de machine virtuelle, un équilibreur de charge est également créé. L’équilibreur de charge inclut des règles pour distribuer le trafic sur le port TCP 80, ainsi que pour autoriser le trafic Bureau à distance sur le port TCP 3389 et le trafic Accès distant PowerShell sur le port TCP 5985 :

```azurepowershell-interactive
$vmssName="myScaleSet"

New-AzVmss `
    -ResourceGroupName $rgName `
    -VMScaleSetName $vmssName `
    -Location $location `
    -VirtualNetworkName "myVnet" `
    -SubnetName "mySubnet" `
    -PublicIpAddressName "myPublicIPAddress" `
    -LoadBalancerName "myLoadBalancer" `
    -UpgradePolicy "Automatic" `
    -Credential $cred
```

## <a name="enable-encryption"></a>Activer le chiffrement

Pour chiffrer des instances de machine virtuelle dans un groupe identique, vous devez d’abord obtenir certaines informations sur l’URI et l’ID de ressources du coffre de clés avec [Get-AzKeyVault](/powershell/module/az.keyvault/Get-AzKeyVault). Les variables obtenues sont ensuite utilisées pour démarrer le processus de chiffrement avec [Set-AzVmssDiskEncryptionExtension](/powershell/module/az.compute/Set-AzVmssDiskEncryptionExtension) :


```azurepowershell-interactive
$diskEncryptionKeyVaultUrl=(Get-AzKeyVault -ResourceGroupName $rgName -Name $vaultName).VaultUri
$keyVaultResourceId=(Get-AzKeyVault -ResourceGroupName $rgName -Name $vaultName).ResourceId

Set-AzVmssDiskEncryptionExtension -ResourceGroupName $rgName -VMScaleSetName $vmssName `
    -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $keyVaultResourceId -VolumeType "All"
```

Quand vous y êtes invité, tapez *y* pour continuer le processus de chiffrement de disque sur les instances de machine virtuelle dans le groupe identique.

### <a name="enable-encryption-using-kek-to-wrap-the-key"></a>Activer le chiffrement à l’aide de KEK pour encapsuler la clé

Vous pouvez également utiliser une clé de chiffrement de clé pour renforcer la sécurité lors du chiffrement du groupe de machines virtuelles identiques.

```azurepowershell-interactive
$diskEncryptionKeyVaultUrl=(Get-AzKeyVault -ResourceGroupName $rgName -Name $vaultName).VaultUri
$keyVaultResourceId=(Get-AzKeyVault -ResourceGroupName $rgName -Name $vaultName).ResourceId
$keyEncryptionKeyUrl = (Get-AzKeyVaultKey -VaultName $vaultName -Name $keyEncryptionKeyName).Key.kid;

Set-AzVmssDiskEncryptionExtension -ResourceGroupName $rgName -VMScaleSetName $vmssName `
    -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $keyVaultResourceId `
    -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $keyVaultResourceId -VolumeType "All"
```

> [!NOTE]
>  La syntaxe de la valeur du paramètre disk-encryption-keyvault est la chaîne de l’identificateur complet :</br>
/subscriptions/[guid-id-abonnement]/resourceGroups/[nom-groupe-ressources]/providers/Microsoft.KeyVault/vaults/[nom-coffre-clés]</br></br>
> La syntaxe de la valeur du paramètre key-encryption-key est l’URI complet vers la clé KEK comme dans :</br>
https://[nom-coffre-clés].vault.azure.net/keys/[nom-clé-kek]/[id-clé-kek-unique]

## <a name="check-encryption-progress"></a>Vérifier la progression du chiffrement

Pour vérifier l’état du chiffrement de disque, utilisez [Get-AzVmssDiskEncryption](/powershell/module/az.compute/Get-AzVmssDiskEncryption) :


```azurepowershell-interactive
Get-AzVmssDiskEncryption -ResourceGroupName $rgName -VMScaleSetName $vmssName
```

Une fois que les instances de machine virtuelle sont chiffrées, la variable *EncryptionSummary* retourne *ProvisioningState/succeeded* comme dans l’exemple de sortie suivant :

```powershell
ResourceGroupName            : myResourceGroup
VmScaleSetName               : myScaleSet
EncryptionSettings           :
  KeyVaultURL                : https://myuniquekeyvault.vault.azure.net/
  KeyEncryptionKeyURL        :
  KeyVaultResourceId         : /subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.KeyVault/vaults/myuniquekeyvault
  KekVaultResourceId         :
  KeyEncryptionAlgorithm     :
  VolumeType                 : All
  EncryptionOperation        : EnableEncryption
EncryptionSummary[0]         :
  Code                       : ProvisioningState/succeeded
  Count                      : 2
EncryptionEnabled            : True
EncryptionExtensionInstalled : True
```

## <a name="disable-encryption"></a>Désactiver le chiffrement

Si vous ne souhaitez plus utiliser les disques d’instances de machine virtuelle chiffrés, désactivez le chiffrement avec [Disable-AzVmssDiskEncryption](/powershell/module/az.compute/Disable-AzVmssDiskEncryption), comme ceci :


```azurepowershell-interactive
Disable-AzVmssDiskEncryption -ResourceGroupName $rgName -VMScaleSetName $vmssName
```

## <a name="next-steps"></a>Étapes suivantes

- Dans cet article, vous avez utilisé Azure PowerShell pour chiffrer un groupe de machines virtuelles identiques. Vous pouvez également utiliser [Azure CLI](disk-encryption-cli.md) ou des [modèles Azure Resource Manager](disk-encryption-azure-resource-manager.md).
- Si vous souhaitez qu’Azure Disk Encryption soit appliqué après l’approvisionnement d’une autre extension, vous pouvez utiliser le [séquencement d’extensions](virtual-machine-scale-sets-extension-sequencing.md).
