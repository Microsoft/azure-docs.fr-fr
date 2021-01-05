---
title: Exemples de scripts Azure Disk Encryption
description: Cet article constitue l’annexe de Microsoft Azure Disk Encryption pour les machines virtuelles Windows.
author: msmbaldwin
ms.service: virtual-machines-windows
ms.subservice: security
ms.topic: how-to
ms.author: mbaldwin
ms.date: 08/06/2019
ms.custom: seodec18
ms.openlocfilehash: 255e284cf8d54a9be59f09f5613cb2728417d234
ms.sourcegitcommit: d76108b476259fe3f5f20a91ed2c237c1577df14
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92912036"
---
# <a name="azure-disk-encryption-sample-scripts"></a>Exemples de scripts Azure Disk Encryption 

Cet article fournit des exemples de scripts pour la préparation de disques durs virtuels préchiffrés et d’autres tâches.

> [!NOTE]
> Sauf mention contraire, tous les scripts font référence à la dernière version hors AAD d’ADE.

## <a name="sample-powershell-scripts-for-azure-disk-encryption"></a>Exemples de scripts PowerShell pour Azure Disk Encryption 


- **Répertorier toutes les machines virtuelles chiffrées dans votre abonnement**

  Vous pouvez rechercher toutes les machines virtuelles chiffrées avec ADE et la version de l’extension, dans tous les groupes de ressources d’un abonnement, à l’aide de [ce script PowerShell](https://raw.githubusercontent.com/Azure/azure-powershell/master/src/Compute/Compute/Extension/AzureDiskEncryption/Scripts/Find_1passAdeVersion_VM.ps1).

  À l’inverse, ces cmdlets affichent toutes les machines virtuelles chiffrées avec ADE (sans la version de l’extension) :

    ```azurepowershell-interactive
    $osVolEncrypted = {(Get-AzVMDiskEncryptionStatus -ResourceGroupName $_.ResourceGroupName -VMName $_.Name).OsVolumeEncrypted}
    $dataVolEncrypted= {(Get-AzVMDiskEncryptionStatus -ResourceGroupName $_.ResourceGroupName -VMName $_.Name).DataVolumesEncrypted}
    Get-AzVm | Format-Table @{Label="MachineName"; Expression={$_.Name}}, @{Label="OsVolumeEncrypted"; Expression=$osVolEncrypted}, @{Label="DataVolumesEncrypted"; Expression=$dataVolEncrypted}
    ```

- **Répertorier toutes les instances VMSS chiffrées de votre abonnement**
    
    Vous pouvez rechercher toutes les instances VMSS chiffrées avec ADE et la version de l’extension, dans tous les groupes de ressources d’un abonnement, à l’aide de [ce script PowerShell](https://raw.githubusercontent.com/Azure/azure-powershell/master/src/Compute/Compute/Extension/AzureDiskEncryption/Scripts/Find_1passAdeVersion_VMSS.ps1).
 
- **Répertorier tous les secrets de chiffrement de disque utilisées pour chiffrer les machines virtuelles dans un coffre de clés**

```azurepowershell-interactive
Get-AzKeyVaultSecret -VaultName $KeyVaultName | where {$_.Tags.ContainsKey('DiskEncryptionKeyFileName')} | format-table @{Label="MachineName"; Expression={$_.Tags['MachineName']}}, @{Label="VolumeLetter"; Expression={$_.Tags['VolumeLetter']}}, @{Label="EncryptionKeyURL"; Expression={$_.Id}}
```

### <a name="using-the-azure-disk-encryption-prerequisites-powershell-script"></a>Utilisation du script PowerShell des prérequis Azure Disk Encryption

Si vous êtes déjà familiarisé avec les prérequis d’Azure Disk Encryption, vous pouvez utiliser le [script PowerShell des prérequis d’Azure Disk Encryption](https://raw.githubusercontent.com/Azure/azure-powershell/master/src/Compute/Compute/Extension/AzureDiskEncryption/Scripts/AzureDiskEncryptionPreRequisiteSetup.ps1 ). Vous trouverez un exemple d’utilisation de ce script PowerShell dans [Démarrage rapide du chiffrement d’une machine virtuelle](disk-encryption-powershell-quickstart.md). Vous pouvez supprimer les commentaires d’une section du script, en commençant à la ligne 211, pour chiffrer tous les disques des machines virtuelles d’un groupe de ressources existant. 

Le tableau suivant présente les paramètres pouvant être utilisés dans le script PowerShell : 

|Paramètre|Description|Obligatoire ?|
|------|------|------|
|$resourceGroupName| Nom du groupe de ressources auquel appartient le coffre de clés.  Un groupe de ressources portant ce nom sera créé s’il n’existe pas encore.| True|
|$keyVaultName|Nom du coffre de clés dans lequel les clés de chiffrement doivent être placées. Un coffre portant ce nom sera créé s’il n’existe pas encore.| True|
|$location|Emplacement du coffre de clés. Assurez-vous que le coffre de clés et les machines virtuelles à chiffrer se trouvent au même emplacement. Obtenez une liste des emplacements avec `Get-AzLocation`.|True|
|$subscriptionId|Identificateur de l’abonnement Azure à utiliser.  Vous pouvez obtenir votre ID d’abonnement avec `Get-AzSubscription`.|True|
|$aadAppName|Nom de l’application Azure AD qui sera utilisée pour écrire des secrets dans le coffre de clés. S’il n’existe aucune application de ce nom, une nouvelle application sera créée en utilisant le nom que vous avez saisi. Si cette application existe déjà, appliquez le paramètre aadClientSecret au script.|False|
|$aadClientSecret|Clé secrète client de l’application Azure AD qui a été créée précédemment.|False|
|$keyEncryptionKeyName|Nom de la clé de chiffrement principale facultative dans KeyVault. Une clé portant ce nom sera créée si elle n’existe pas encore.|False|

## <a name="resource-manager-templates"></a>Modèles Resource Manager

### <a name="encrypt-or-decrypt-vms-without-an-azure-ad-app"></a>Chiffrer ou déchiffrer des machines virtuelles sans application Azure AD

- [Activer le chiffrement de disque sur une machine virtuelle Windows existante ou en cours d’exécution](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-windows-vm-without-aad)  
- [Désactiver le chiffrement sur une machine virtuelle Windows en cours d’exécution](https://github.com/Azure/azure-quickstart-templates/tree/master/201-decrypt-running-windows-vm-without-aad) 

### <a name="encrypt-or-decrypt-vms-with-an-azure-ad-app-previous-release"></a>Chiffrer ou déchiffrer des machines virtuelles avec une application Azure AD (version précédente) 
 
- [Activer le chiffrement de disque sur une machine virtuelle Windows existante ou en cours d’exécution](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-windows-vm)    
- [Désactiver le chiffrement sur une machine virtuelle Windows en cours d’exécution](https://github.com/Azure/azure-quickstart-templates/tree/master/201-decrypt-running-windows-vm) 
- [Créer un disque managé chiffré à partir d’un disque dur virtuel/objet blob de stockage préchiffré](https://github.com/Azure/azure-quickstart-templates/tree/master/201-create-encrypted-managed-disk)
    - Crée un disque managé chiffré fourni par un disque dur virtuel préchiffré et ses paramètres de chiffrement correspondants

## <a name="prepare-a-pre-encrypted-windows-vhd"></a>Préparer un disque dur virtuel Windows déjà chiffré
Les sections qui suivent sont nécessaires pour préparer un disque dur virtuel Windows déjà chiffré qui sera déployé comme disque dur virtuel chiffré dans Azure IaaS. Utilisez ces informations pour préparer et démarrer une nouvelle machine virtuelle Windows (disque dur virtuel) sur Azure Site Recovery ou Azure. Pour plus d’informations sur la préparation et le chargement d’un disque dur virtuel, consultez [Charger un disque dur virtuel généralisé et l’utiliser pour créer des machines virtuelles dans Azure](upload-generalized-managed.md).

### <a name="update-group-policy-to-allow-non-tpm-for-os-protection"></a>Mettre à jour la stratégie de groupe pour permettre la protection autre que par module de plateforme sécurisée pour la protection du système d’exploitation
Configurez le paramètre de stratégie de groupe BitLocker **Chiffrement de lecteur BitLocker** qui se trouve sous **Stratégie de l’ordinateur local** > **Configuration ordinateur** > **Modèles d’administration** > **Composants Windows**. Remplacez ce paramètre par **Lecteurs du système d’exploitation** > **Exiger une authentification supplémentaire au démarrage** > **Autoriser BitLocker sans un module de plateforme sécurisée compatible** comme indiqué dans la figure ci-dessous :

![Microsoft Antimalware dans Azure](../media/disk-encryption/disk-encryption-fig8.png)

### <a name="install-bitlocker-feature-components"></a>Installer les composants de fonctionnalité BitLocker
Pour Windows Server 2012 ou version ultérieure, utilisez la commande suivante :

```console
dism /online /Enable-Feature /all /FeatureName:BitLocker /quiet /norestart
```

Pour Windows Server 2008 R2, utilisez la commande suivante :

```console
ServerManagerCmd -install BitLockers
```

### <a name="prepare-the-os-volume-for-bitlocker-by-using-bdehdcfg"></a>Préparer le volume du système d’exploitation pour BitLocker à l’aide de `bdehdcfg`
Pour compresser la partition du système d’exploitation et préparer la machine pour BitLocker, exécutez si nécessaire [bdehdcfg](/windows/security/information-protection/bitlocker/bitlocker-basic-deployment) :

```console
bdehdcfg -target c: shrink -quiet 
```

### <a name="protect-the-os-volume-by-using-bitlocker"></a>Protéger le volume du système d’exploitation à l’aide de BitLocker
Utilisez la commande [`manage-bde`](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/ff829849(v=ws.11)) pour activer le chiffrement sur le volume de démarrage à l’aide d’un protecteur de clé externe. Placez également la clé externe (fichier .bek) sur le disque ou le volume externe. Le chiffrement sera activé sur le volume système/de démarrage au prochain redémarrage.

```console
manage-bde -on %systemdrive% -sk [ExternalDriveOrVolume]
reboot
```

> [!NOTE]
> Préparez la machine virtuelle avec un disque dur virtuel de données/de ressources distinct pour obtenir la clé externe à l’aide de BitLocker.

## <a name="upload-encrypted-vhd-to-an-azure-storage-account"></a>Télécharger des disques durs virtuels cryptés dans un compte de stockage Azure
Une fois le chiffrement DM-Crypt activé, le disque dur virtuel chiffré local doit être chargé sur votre compte de stockage.
```powershell
    Add-AzVhd [-Destination] <Uri> [-LocalFilePath] <FileInfo> [[-NumberOfUploaderThreads] <Int32> ] [[-BaseImageUriToPatch] <Uri> ] [[-OverWrite]] [ <CommonParameters>]
```

## <a name="upload-the-secret-for-the-pre-encrypted-vm-to-your-key-vault"></a>Charger le secret pour la machine virtuelle préchiffrée dans votre coffre de clés
Le secret de chiffrement de disque obtenu précédemment doit être chargé en tant que secret dans votre coffre de clés.  Pour ce faire, vous devez accorder l’autorisation de définition de secret et l’autorisation wrapkey au compte qui chargera les secrets.

```powershell 
# Typically, account Id is the user principal name (in user@domain.com format)
$upn = (Get-AzureRmContext).Account.Id
Set-AzKeyVaultAccessPolicy -VaultName $kvname -UserPrincipalName $acctid -PermissionsToKeys wrapKey -PermissionsToSecrets set

# In cloud shell, the account ID is a managed service identity, so specify the username directly 
# $upn = "user@domain.com" 
# Set-AzKeyVaultAccessPolicy -VaultName $kvname -UserPrincipalName $acctid -PermissionsToKeys wrapKey -PermissionsToSecrets set

# When running as a service principal, retrieve the service principal ID from the account ID, and set access policy to that 
# $acctid = (Get-AzureRmContext).Account.Id
# $spoid = (Get-AzureRmADServicePrincipal -ServicePrincipalName $acctid).Id
# Set-AzKeyVaultAccessPolicy -VaultName $kvname -ObjectId $spoid -BypassObjectIdValidation -PermissionsToKeys wrapKey -PermissionsToSecrets set

```

### <a name="disk-encryption-secret-not-encrypted-with-a-kek"></a>La clé secrète de chiffrement de disque non chiffré avec une clé de chiffrement à clé KEK
Utilisez [Set-AzKeyVaultSecret](/powershell/module/az.keyvault/set-azkeyvaultsecret) pour configurer la clé secrète dans votre coffre de clés. La phrase secrète est encodée sous forme de chaîne en base64, puis chargée dans le coffre de clés. Assurez-vous également que les balises suivantes sont définies lors de la création de la clé secrète dans le coffre de clés.

```powershell

 # This is the passphrase that was provided for encryption during the distribution installation
 $passphrase = "contoso-password"

 $tags = @{"DiskEncryptionKeyEncryptionAlgorithm" = "RSA-OAEP"; "DiskEncryptionKeyFileName" = "LinuxPassPhraseFileName"}
 $secretName = [guid]::NewGuid().ToString()
 $secretValue = [Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes($passphrase))
 $secureSecretValue = ConvertTo-SecureString $secretValue -AsPlainText -Force

 $secret = Set-AzKeyVaultSecret -VaultName $KeyVaultName -Name $secretName -SecretValue $secureSecretValue -tags $tags
 $secretUrl = $secret.Id
```


Utilisez `$secretUrl` à l’étape suivante pour [attacher le lecteur du système d’exploitation sans utiliser de clé de chiffrement à clé](#without-using-a-kek).

### <a name="disk-encryption-secret-encrypted-with-a-kek"></a>Disque chiffré avec une clé secrète de chiffrement de disque à clé KEK
Avant de télécharger la clé secrète dans le coffre de clés, vous pouvez éventuellement la chiffrer à l’aide d’une clé de chiffrement à clé. Utilisez [l’API](/rest/api/keyvault/wrapkey) de retour à la ligne pour chiffrer d’abord la clé secrète à l’aide de la clé de chiffrement à clé. La sortie de cette opération de retour à la ligne est une chaîne d’URL encodée en base64 que vous pouvez ensuite charger comme clé secrète à l’aide de l’applet de commande [`Set-AzKeyVaultSecret`](/powershell/module/az.keyvault/set-azkeyvaultsecret).

```powershell
    # This is the passphrase that was provided for encryption during the distribution installation
    $passphrase = "contoso-password"

    Add-AzKeyVaultKey -VaultName $KeyVaultName -Name "keyencryptionkey" -Destination Software
    $KeyEncryptionKey = Get-AzKeyVaultKey -VaultName $KeyVault.OriginalVault.Name -Name "keyencryptionkey"

    $apiversion = "2015-06-01"

    ##############################
    # Get Auth URI
    ##############################

    $uri = $KeyVault.VaultUri + "/keys"
    $headers = @{}

    $response = try { Invoke-RestMethod -Method GET -Uri $uri -Headers $headers } catch { $_.Exception.Response }

    $authHeader = $response.Headers["www-authenticate"]
    $authUri = [regex]::match($authHeader, 'authorization="(.*?)"').Groups[1].Value

    Write-Host "Got Auth URI successfully"

    ##############################
    # Get Auth Token
    ##############################

    $uri = $authUri + "/oauth2/token"
    $body = "grant_type=client_credentials"
    $body += "&client_id=" + $AadClientId
    $body += "&client_secret=" + [Uri]::EscapeDataString($AadClientSecret)
    $body += "&resource=" + [Uri]::EscapeDataString("https://vault.azure.net")
    $headers = @{}

    $response = Invoke-RestMethod -Method POST -Uri $uri -Headers $headers -Body $body

    $access_token = $response.access_token

    Write-Host "Got Auth Token successfully"

    ##############################
    # Get KEK info
    ##############################

    $uri = $KeyEncryptionKey.Id + "?api-version=" + $apiversion
    $headers = @{"Authorization" = "Bearer " + $access_token}

    $response = Invoke-RestMethod -Method GET -Uri $uri -Headers $headers

    $keyid = $response.key.kid

    Write-Host "Got KEK info successfully"

    ##############################
    # Encrypt passphrase using KEK
    ##############################

    $passphraseB64 = [Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes($Passphrase))
    $uri = $keyid + "/encrypt?api-version=" + $apiversion
    $headers = @{"Authorization" = "Bearer " + $access_token; "Content-Type" = "application/json"}
    $bodyObj = @{"alg" = "RSA-OAEP"; "value" = $passphraseB64}
    $body = $bodyObj | ConvertTo-Json

    $response = Invoke-RestMethod -Method POST -Uri $uri -Headers $headers -Body $body

    $wrappedSecret = $response.value

    Write-Host "Encrypted passphrase successfully"

    ##############################
    # Store secret
    ##############################

    $secretName = [guid]::NewGuid().ToString()
    $uri = $KeyVault.VaultUri + "/secrets/" + $secretName + "?api-version=" + $apiversion
    $secretAttributes = @{"enabled" = $true}
    $secretTags = @{"DiskEncryptionKeyEncryptionAlgorithm" = "RSA-OAEP"; "DiskEncryptionKeyFileName" = "LinuxPassPhraseFileName"}
    $headers = @{"Authorization" = "Bearer " + $access_token; "Content-Type" = "application/json"}
    $bodyObj = @{"value" = $wrappedSecret; "attributes" = $secretAttributes; "tags" = $secretTags}
    $body = $bodyObj | ConvertTo-Json

    $response = Invoke-RestMethod -Method PUT -Uri $uri -Headers $headers -Body $body

    Write-Host "Stored secret successfully"

    $secretUrl = $response.id
```

Utilisez `$KeyEncryptionKey` et `$secretUrl` à l’étape suivante pour [attacher le lecteur du système d’exploitation à l’aide de la clé de chiffrement à clé](#using-a-kek).

##  <a name="specify-a-secret-url-when-you-attach-an-os-disk"></a>Spécifier une URL secrète lorsque vous attachez un lecteur de système d’exploitation

###  <a name="without-using-a-kek"></a>Sans utiliser de clé de chiffrement à clé KEK
Lorsque vous attachez le lecteur du système d’exploitation, exécutez la commande `$secretUrl`. L’URL a été générée dans la section « La clé secrète de chiffrement de disque non chiffrée avec une clé de chiffrement à clé ».
```powershell
    Set-AzVMOSDisk `
            -VM $VirtualMachine `
            -Name $OSDiskName `
            -SourceImageUri $VhdUri `
            -VhdUri $OSDiskUri `
            -Windows `
            -CreateOption FromImage `
            -DiskEncryptionKeyVaultId $KeyVault.ResourceId `
            -DiskEncryptionKeyUrl $SecretUrl
```
### <a name="using-a-kek"></a>À l’aide d’une clé de chiffrement à clé KEK
Lorsque vous attachez le lecteur du système d’exploitation, exécutez les commandes `$KeyEncryptionKey` et `$secretUrl`. L’URL a été générée dans la section « Secret de chiffrement de disque chiffré avec une clé KEK ».
```powershell
    Set-AzVMOSDisk `
            -VM $VirtualMachine `
            -Name $OSDiskName `
            -SourceImageUri $CopiedTemplateBlobUri `
            -VhdUri $OSDiskUri `
            -Windows `
            -CreateOption FromImage `
            -DiskEncryptionKeyVaultId $KeyVault.ResourceId `
            -DiskEncryptionKeyUrl $SecretUrl `
            -KeyEncryptionKeyVaultId $KeyVault.ResourceId `
            -KeyEncryptionKeyURL $KeyEncryptionKey.Id
```
