---
title: Restaurer la clé et le secret du Key Vault pour les machines virtuelles chiffrées
description: Découvrez comment restaurer la clé et la clé secrète du Key Vault dans Azure Backup à l’aide de PowerShell
ms.topic: conceptual
ms.date: 08/28/2017
ms.openlocfilehash: 456ce18f253ffa02cd6b13826a7839f18beecba7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88827084"
---
# <a name="restore-key-vault-key-and-secret-for-encrypted-vms-using-azure-backup"></a>Restaurer la clé et le secret du Key Vault pour les machines virtuelles chiffrées à l’aide d’Azure Backup

Cet article décrit l'utilisation du service de sauvegarde de machines virtuelles Azure pour restaurer des machines virtuelles Azure chiffrées si votre clé et votre secret ne figurent pas dans le coffre de clés. Vous pouvez également suivre cette procédure si vous souhaitez conserver une copie distincte de la clé (clé de chiffrement à clé) et du secret (clé de chiffrement BitLocker) pour la machine Virtuelle restaurée.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prérequis

* **Sauvegarder les machines virtuelles chiffrées** : les machines virtuelles Azure chiffrées ont été sauvegardées à l’aide d’Azure Backup. Pour plus d'informations sur la sauvegarde de machines virtuelles Azure chiffrées, consultez [Gérer la sauvegarde et la restauration de machines virtuelles Azure à l'aide de PowerShell](backup-azure-vms-automation.md).
* **Configurer Azure Key Vault**: assurez-vous que le Key Vault dans lequel les clés et secrets doivent être restaurés est déjà présent. Pour plus d'informations sur la gestion du coffre de clés, consultez [Prise en main d'Azure Key Vault](../key-vault/general/overview.md).
* **Restauration du disque** : vérifiez que vous avez déclenché le travail de restauration pour restaurer les disques d'une machine virtuelle chiffrée à l'aide des [étapes PowerShell](backup-azure-vms-automation.md#restore-an-azure-vm). Ceci parce que ce travail génère un fichier JSON dans votre compte de stockage contenant les clés et les secrets de la machine virtuelle chiffrée à restaurer.

## <a name="get-key-and-secret-from-azure-backup"></a>Obtenir la clé et le secret à partir d’Azure Backup

> [!NOTE]
> Une fois le disque restauré pour la machine virtuelle chiffrée, vérifiez que :
>
> * $details est alimenté avec les détails du travail du disque de restauration, comme indiqué dans [étapes PowerShell dans la section Restaurer les disques](backup-azure-vms-automation.md#restore-an-azure-vm)
> * La machine virtuelle doit être créée à partir de disques restaurées uniquement **après la restauration de la clé et du secret selon le coffre de clés**.

Interrogez les propriétés des disques restaurés pour obtenir les détails du travail.

```powershell
$properties = $details.properties
$storageAccountName = $properties["Target Storage Account Name"]
$containerName = $properties["Config Blob Container Name"]
$encryptedBlobName = $properties["Encryption Info Blob Name"]
```

Définissez le contexte de stockage Azure et restaurez le fichier de configuration JSON contenant les détails de la clé et du secret pour la machine virtuelle chiffrée.

```powershell
Set-AzCurrentStorageAccount -Name $storageaccountname -ResourceGroupName '<rg-name>'
$destination_path = 'C:\vmencryption_config.json'
Get-AzStorageBlobContent -Blob $encryptedBlobName -Container $containerName -Destination $destination_path
$encryptionObject = Get-Content -Path $destination_path  | ConvertFrom-Json
```

## <a name="restore-key"></a>Restaurer la clé

Une fois le fichier JSON généré dans le chemin d’accès de destination mentionné ci-dessus, générez un fichier blob clé à partir de JSON et alimentez-le pour restaurer l’applet de commande clé pour replacer la clé (KEK) dans le coffre de clés.

```powershell
$keyDestination = 'C:\keyDetails.blob'
[io.file]::WriteAllBytes($keyDestination, [System.Convert]::FromBase64String($encryptionObject.OsDiskKeyAndSecretDetails.KeyBackupData))
Restore-AzureKeyVaultKey -VaultName '<target_key_vault_name>' -InputFile $keyDestination
```

## <a name="restore-secret"></a>Restaurer le secret

Utilisez le fichier JSON généré ci-dessus pour obtenir le nom et la valeur secrets et alimentez-le pour définir l’applet de commande secrète pour replacer le secret (BEK) dans le coffre de clés. Utilisez ces cmdlets si votre **machine virtuelle est chiffrée à l’aide de BEK et KEK**.

Utilisez ces cmdlets si votre **machine virtuelle Windows est chiffrée à l’aide de BEK et KEK**.

```powershell
$secretdata = $encryptionObject.OsDiskKeyAndSecretDetails.SecretData
$Secret = ConvertTo-SecureString -String $secretdata -AsPlainText -Force
$secretname = 'B3284AAA-DAAA-4AAA-B393-60CAA848AAAA'
$Tags = @{'DiskEncryptionKeyEncryptionAlgorithm' = 'RSA-OAEP';'DiskEncryptionKeyFileName' = 'B3284AAA-DAAA-4AAA-B393-60CAA848AAAA.BEK';'DiskEncryptionKeyEncryptionKeyURL' = $encryptionObject.OsDiskKeyAndSecretDetails.KeyUrl;'MachineName' = 'vm-name'}
Set-AzureKeyVaultSecret -VaultName '<target_key_vault_name>' -Name $secretname -SecretValue $Secret -ContentType  'Wrapped BEK' -Tags $Tags
```

Utilisez ces cmdlets si votre **machine virtuelle Linux est chiffrée à l’aide de BEK et KEK**.

```powershell
$secretdata = $encryptionObject.OsDiskKeyAndSecretDetails.SecretData
$Secret = ConvertTo-SecureString -String $secretdata -AsPlainText -Force
$secretname = 'B3284AAA-DAAA-4AAA-B393-60CAA848AAAA'
$Tags = @{'DiskEncryptionKeyEncryptionAlgorithm' = 'RSA-OAEP';'DiskEncryptionKeyFileName' = 'LinuxPassPhraseFileName';'DiskEncryptionKeyEncryptionKeyURL' = <Key_url_of_newly_restored_key>;'MachineName' = 'vm-name'}
Set-AzureKeyVaultSecret -VaultName '<target_key_vault_name>' -Name $secretname -SecretValue $Secret -ContentType  'Wrapped BEK' -Tags $Tags
```

Utilisez le fichier JSON généré ci-dessus pour obtenir le nom et la valeur secrets et alimentez-le pour définir l’applet de commande secrète pour replacer le secret (BEK) dans le coffre de clés. Utilisez ces cmdlets si votre **machine virtuelle est chiffrée à l’aide de BEK** uniquement.

```powershell
$secretDestination = 'C:\secret.blob'
[io.file]::WriteAllBytes($secretDestination, [System.Convert]::FromBase64String($encryptionObject.OsDiskKeyAndSecretDetails.KeyVaultSecretBackupData))
Restore-AzureKeyVaultSecret -VaultName '<target_key_vault_name>' -InputFile $secretDestination -Verbose
  ```

> [!NOTE]
>
> * La valeur de $secretname peut être obtenue par référence à la sortie de $encryptionObject.OsDiskKeyAndSecretDetails.SecretUrl et en utilisant le texte après secrets/ Par exemple, l'URL du secret de sortie est `https://keyvaultname.vault.azure.net/secrets/B3284AAA-DAAA-4AAA-B393-60CAA848AAAA/xx000000xx0849999f3xx30000003163` et le nom du secret est B3284AAA-DAAA-4AAA-B393-60CAA848AAAA.
> * La valeur de la balise DiskEncryptionKeyFileName est identique au nom du secret.
>
>

## <a name="create-virtual-machine-from-restored-disk"></a>Créer une machine virtuelle à partir du disque restauré

Si vous avez sauvegardé une machine virtuelle chiffrée à l'aide du service de sauvegarde de machines virtuelles Azure, les cmdlets PowerShell mentionnées plus haut peuvent vous aider à restaurer la clé et le secret dans le coffre de clés. Après leur restauration, consultez l'article [Gérer la sauvegarde et la restauration de machines virtuelles Azure à l'aide de PowerShell](backup-azure-vms-automation.md#create-a-vm-from-restored-disks) pour créer des machines virtuelles chiffrées à partir du disque, de la clé et du secret restaurés.

## <a name="legacy-approach"></a>Approche héritée

La méthode ci-dessus fonctionne pour tous les points de récupération. Toutefois, l’ancienne méthode d’obtention des informations relatives à la clé et au secret à partir du point de récupération est toujours valide pour les points de récupération antérieurs au 11 juillet 2017, sur les machines virtuelles chiffrées à l’aide de BEK et KEK. Une fois le travail de restauration du disque terminé pour la machine virtuelle chiffrée à l’aide des [étapes PowerShell](backup-azure-vms-automation.md#restore-an-azure-vm), assurez-vous que $rp est alimenté avec une valeur valide.

### <a name="restore-key-legacy-approach"></a>Restaurer la clé (approche héritée)

Utilisez les applets de commande suivantes pour obtenir des informations de clés (KEK) à partir du point de récupération et alimentez-les pour restaurer l’applet de commande clé pour la replacer dans le coffre de clés.

```powershell
$rp1 = Get-AzRecoveryServicesBackupRecoveryPoint -RecoveryPointId $rp[0].RecoveryPointId -Item $backupItem -KeyFileDownloadLocation 'C:\Users\downloads'
Restore-AzureKeyVaultKey -VaultName '<target_key_vault_name>' -InputFile 'C:\Users\downloads'
```

### <a name="restore-secret-legacy-approach"></a>Restaurer le secret (approche héritée)

Utilisez les applets de commande suivantes pour obtenir des informations de secrets (KEK) à partir du point de récupération et alimentez-les pour restaurer l’applet de commande secrète pour la replacer dans le coffre de clés.

```powershell
$secretname = 'B3284AAA-DAAA-4AAA-B393-60CAA848AAAA'
$secretdata = $rp1.KeyAndSecretDetails.SecretData
$Secret = ConvertTo-SecureString -String $secretdata -AsPlainText -Force
$Tags = @{'DiskEncryptionKeyEncryptionAlgorithm' = 'RSA-OAEP';'DiskEncryptionKeyFileName' = 'B3284AAA-DAAA-4AAA-B393-60CAA848AAAA.BEK';'DiskEncryptionKeyEncryptionKeyURL' = 'https://mykeyvault.vault.azure.net:443/keys/KeyName/84daaac999949999030bf99aaa5a9f9';'MachineName' = 'vm-name'}
Set-AzureKeyVaultSecret -VaultName '<target_key_vault_name>' -Name $secretname -SecretValue $secret -Tags $Tags -SecretValue $Secret -ContentType  'Wrapped BEK'
```

> [!NOTE]
>
> * La valeur de $secretname peut être obtenue par référence à la sortie de $rp1.KeyAndSecretDetails.SecretUrl et en utilisant le texte après secrets/ Par exemple, l'URL du secret de sortie est `https://keyvaultname.vault.azure.net/secrets/B3284AAA-DAAA-4AAA-B393-60CAA848AAAA/xx000000xx0849999f3xx30000003163` et le nom du secret est B3284AAA-DAAA-4AAA-B393-60CAA848AAAA
> * La valeur de la balise DiskEncryptionKeyFileName est identique au nom du secret.
> * Vous pouvez obtenir la valeur de DiskEncryptionKeyEncryptionKeyURL à partir du Key Vault après la restauration des clés, et en utilisant l’applet de commande [Get-AzureKeyVaultKey](/powershell/module/azurerm.keyvault/get-azurekeyvaultkey).
>
>

## <a name="next-steps"></a>Étapes suivantes

Après la restauration de la clé et du secret dans le coffre de clés, consultez l'article [Gérer la sauvegarde et la restauration de machines virtuelles Azure à l'aide de PowerShell](backup-azure-vms-automation.md#create-a-vm-from-restored-disks) pour créer des machines virtuelles chiffrées à partir du disque, de la clé et du secret restaurés.
