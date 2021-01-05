---
title: Résolution des erreurs de démarrage BitLocker dans une machine virtuelle Azure | Microsoft Docs
description: Découvrez comment résoudre les erreurs de démarrage BitLocker dans une machine virtuelle Azure.
services: virtual-machines-windows
documentationCenter: ''
author: genlin
manager: dcscontentpm
editor: v-jesits
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 11/20/2020
ms.author: genli
ms.custom: has-adal-ref
ms.openlocfilehash: 00095eed3fe6d143d9ed7a0c748c4702028f4632
ms.sourcegitcommit: beacda0b2b4b3a415b16ac2f58ddfb03dd1a04cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/31/2020
ms.locfileid: "97832058"
---
# <a name="bitlocker-boot-errors-on-an-azure-vm"></a>Erreurs de démarrage BitLocker dans une machine virtuelle Azure

 Cet article décrit les erreurs BitLocker que vous pouvez rencontrer lorsque vous démarrez une machine virtuelle Windows dans Microsoft Azure.

 

## <a name="symptom"></a>Symptôme

 Une machine virtuelle Windows ne démarre pas. Lorsque vous regardez les captures d’écran de la fenêtre [Diagnostics de démarrage](./boot-diagnostics.md), vous voyez l’un des messages d’erreur suivants :

- Connectez le pilote USB qui contient la clé BitLocker.

- Votre compte est verrouillé. Entrez la clé de récupération pour poursuivre (disposition du clavier : américain) Le nombre maximal de tentatives de connexion a été atteint. Votre PC a été verrouillé pour des raisons de sécurité. Pour récupérer la clé de récupération, accédez à https://windows.microsoft.com/recoverykeyfaq à partir d’un autre PC ou appareil mobile. Si vous en avez besoin, voici l’ID de la clé : XXXXXXX. Vous pouvez aussi réinitialiser votre PC.

- Entrez le mot de passe pour déverrouiller le lecteur [] Appuyez sur la touche Insertion pour afficher le mot de passe à mesure que vous le tapez.
- Entrez votre clé de récupération. Chargez votre clé de récupération à partir d’un périphérique USB.

## <a name="cause"></a>Cause

Ce problème peut se produire lorsque la machine virtuelle ne peut pas localiser le fichier BEK pour déchiffrer le disque chiffré.

## <a name="solution"></a>Solution

Pour résoudre ce problème, arrêtez et libérez la machine virtuelle, et puis démarrez-la. Cette opération force la machine virtuelle à récupérer le fichier BEK à partir d’Azure Key Vault, puis à la placer sur le disque chiffré. 

Si cette méthode ne résout pas le problème, effectuez les étapes suivantes pour restaurer le fichier BEK manuellement :

1. Prenez un instantané du disque système de la machine virtuelle affectée en guise de sauvegarde. Pour plus d’informations, consultez [Créer un instantané](../windows/snapshot-copy-managed-disk.md).
2. [Attachez le disque système à une machine virtuelle de récupération](troubleshoot-recovery-disks-portal-windows.md). Pour exécuter la commande [manage-bde](/windows-server/administration/windows-commands/manage-bde) de l’étape 7, la fonction de **chiffrement de lecteur BitLocker** doit être activée dans la machine virtuelle de récupération.

    Lorsque vous attachez un disque managé, vous pouvez recevoir un message d’erreur du type « Contient des paramètres de chiffrement et ne peut donc pas être utilisé comme disque de données ». Dans ce cas, exécutez le script suivant pour réessayer d’attacher le disque :

    ```Powershell
    $rgName = "myResourceGroup"
    $osDiskName = "ProblemOsDisk"
    # Set the EncryptionSettingsEnabled property to false, so you can attach the disk to the recovery VM.
    New-AzDiskUpdateConfig -EncryptionSettingsEnabled $false |Update-AzDisk -diskName $osDiskName -ResourceGroupName $rgName

    $recoveryVMName = "myRecoveryVM" 
    $recoveryVMRG = "RecoveryVMRG" 
    $OSDisk = Get-AzDisk -ResourceGroupName $rgName -DiskName $osDiskName;

    $vm = get-AzVM -ResourceGroupName $recoveryVMRG -Name $recoveryVMName 

    Add-AzVMDataDisk -VM $vm -Name $osDiskName -ManagedDiskId $osDisk.Id -Caching None -Lun 3 -CreateOption Attach 

    Update-AzVM -VM $vm -ResourceGroupName $recoveryVMRG
    ```
     Vous ne pouvez pas attacher un disque managé à une machine virtuelle qui a été restaurée à partir d’une image d’objet blob.

3. Une fois que le disque est attaché, établissez une connexion entre le Bureau à distance et la machine virtuelle de récupération, de manière à pouvoir exécuter des scripts Azure PowerShell. Vérifiez que vous avez installé [la dernière version d’Azure PowerShell](/powershell/azure/) sur la machine virtuelle de récupération.

4. Ouvrez une session Azure PowerShell avec élévation de privilèges (Exécuter en tant qu’administrateur). Pour vous connecter à l’abonnement Azure, exécutez les commandes suivantes :

    ```Powershell
    Add-AzAccount -SubscriptionID [SubscriptionID]
    ```

5. Exécutez le script suivant pour vérifier le nom du fichier BEK :

    ```powershell
    $vmName = "myVM"
    $vault = "myKeyVault"
    Get-AzKeyVaultSecret -VaultName $vault | where {($_.Tags.MachineName -eq $vmName) -and ($_.ContentType -match 'BEK')} `
            | Sort-Object -Property Created `
            | ft  Created, `
                @{Label="Content Type";Expression={$_.ContentType}}, `
                @{Label ="MachineName"; Expression = {$_.Tags.MachineName}}, `
                @{Label ="Volume"; Expression = {$_.Tags.VolumeLetter}}, `
                @{Label ="DiskEncryptionKeyFileName"; Expression = {$_.Tags.DiskEncryptionKeyFileName}}
    ```

    Voici un exemple de sortie : Dans ce cas, nous supposons que le nom de fichier est EF7B2F5A-50C6-4637-0001-7F599C12F85C.BEK.

    ```
    Created               Content Type Volume MachineName DiskEncryptionKeyFileName
    -------               ------------ ------ ----------- -------------------------
    11/20/2020 7:41:56 AM BEK          C:\    myVM   EF7B2F5A-50C6-4637-0001-7F599C12F85C.BEK
    ```
    Si vous voyez un volume en double, celui dont l’horodatage est le plus récent correspond au fichier BEK qui est actuellement utilisé par la machine virtuelle de récupération.

    Si le **Content Type** (Type de contenu) est **Wrapped BEK** (Clé BEK wrappée), accédez aux [scénarios impliquant des clés KEK (clés de chiffrement principales)](#key-encryption-key-scenario).

    Maintenant que vous connaissez le nom du fichier BEK pour le lecteur, vous devez créer le fichier secret-file-name.BEK pour déverrouiller le lecteur.

6.  Téléchargez le fichier BEK sur le disque de récupération. L’exemple suivant enregistre le fichier BEK dans le dossier C:\BEK. Vérifiez que le chemin `C:\BEK\` existe avant d’exécuter les scripts.

    ```powershell
    $vault = "myKeyVault"
    $bek = "EF7B2F5A-50C6-4637-0001-7F599C12F85C"
    $keyVaultSecret = Get-AzKeyVaultSecret -VaultName $vault -Name $bek
    $bstr = [Runtime.InteropServices.Marshal]::SecureStringToBSTR($keyVaultSecret.SecretValue)
    $bekSecretBase64 = [Runtime.InteropServices.Marshal]::PtrToStringAuto($bstr)
    $bekFileBytes = [Convert]::FromBase64String($bekSecretbase64)
    $path = "C:\BEK\DiskEncryptionKeyFileName.BEK"
    [System.IO.File]::WriteAllBytes($path,$bekFileBytes)
    ```

7.  Pour déverrouiller le disque attaché à l’aide du fichier BEK, exécutez la commande suivante.

    ```powershell
    manage-bde -unlock F: -RecoveryKey "C:\BEK\EF7B2F5A-50C6-4637-0001-7F599C12F85C.BEK
    ```
    Dans cet exemple, le disque du système d’exploitation attaché est le lecteur F. Vérifiez que vous utilisez la bonne lettre de lecteur. 

8. Une fois que le disque a été déverrouillé à l’aide de la clé BEK, détachez le disque de la machine virtuelle de récupération, puis recréez la machine virtuelle à l’aide de ce nouveau disque de système d’exploitation.

    > [!NOTE]
    > L’échange de disque de système d’exploitation n’est pas pris en charge pour les machines virtuelles utilisant le chiffrement de disque.

9. Si la nouvelle machine virtuelle ne peut toujours pas démarrer normalement, essayez l’une des étapes suivantes après avoir déverrouillé le lecteur :

    - Suspendez la protection pour désactiver temporairement BitLocker en exécutant la commande suivante :

    ```console
    manage-bde -protectors -disable F: -rc 0
    ```

    - Déchiffrez entièrement le lecteur. Pour ce faire, exécutez la commande suivante :

    ```console
    manage-bde -off F:
    ```

### <a name="key-encryption-key-scenario"></a>Scénario impliquant une clé de chiffrement principale

Dans un scénario impliquant une clé de chiffrement principale, effectuez les étapes suivantes :

1. Vérifiez que le compte d’utilisateur connecté nécessite l’autorisation « unwrapped » (non wrappée) dans les stratégies d’accès au coffre de clés sous **Utilisateur|Autorisations de clé|Opérations de chiffrement|Ne pas inclure la clé**.
2. Enregistrez le script suivant dans un fichier .PS1 :

    ```powershell
    #Set the Parameters for the script
    param (
            [Parameter(Mandatory=$true)]
            [string] 
            $keyVaultName,
            [Parameter(Mandatory=$true)]
            [string] 
            $kekName,
            [Parameter(Mandatory=$true)]
            [string]
            $secretName,
            [Parameter(Mandatory=$true)]
            [string]
            $bekFilePath,
            [Parameter(Mandatory=$true)]
            [string] 
            $adTenant
            )
    # Load ADAL Assemblies
    $adal = "${env:ProgramFiles}\WindowsPowerShell\Modules\Az.Accounts\$(((dir ${env:ProgramFiles}\WindowsPowerShell\Modules\Az.Accounts).name) | select -last 1)\PreloadAssemblies\Microsoft.IdentityModel.Clients.ActiveDirectory.dll"
    $adalforms = "${env:ProgramFiles}\WindowsPowerShell\Modules\Az.Accounts\$(((dir ${env:ProgramFiles}\WindowsPowerShell\Modules\Az.Accounts).name) | select -last 1)\PreloadAssemblies\Microsoft.IdentityModel.Clients.ActiveDirectory.Platform.dll"
    If ((Test-Path -Path $adal) -and (Test-Path -Path $adalforms)) { 

    [System.Reflection.Assembly]::LoadFrom($adal)
    [System.Reflection.Assembly]::LoadFrom($adalforms)
     }
     else
     {
    $adal="${env:userprofile}\Documents\WindowsPowerShell\Modules\Az.Accounts\$(((dir ${env:userprofile}\Documents\WindowsPowerShell\Modules\Az.Accounts).name) | select -last 1)\PreloadAssemblies\Microsoft.IdentityModel.Clients.ActiveDirectory.dll"
    $adalforms ="${env:userprofile}\Documents\WindowsPowerShell\Modules\Az.Accounts\$(((dir ${env:userprofile}\Documents\WindowsPowerShell\Modules\Az.Accounts).name) | select -last 1)\PreloadAssemblies\Microsoft.IdentityModel.Clients.ActiveDirectory.Platform.dll"
    [System.Reflection.Assembly]::LoadFrom($adal)
    [System.Reflection.Assembly]::LoadFrom($adalforms)
     }  

    # Set well-known client ID for AzurePowerShell
    $clientId = "1950a258-227b-4e31-a9cf-717495945fc2" 
    # Set redirect URI for Azure PowerShell
    $redirectUri = "urn:ietf:wg:oauth:2.0:oob"
    # Set Resource URI to Azure Service Management API
    $resourceAppIdURI = "https://vault.azure.net"
    # Set Authority to Azure AD Tenant
    $authority = "https://login.windows.net/$adtenant"
    # Create Authentication Context tied to Azure AD Tenant
    $authContext = New-Object "Microsoft.IdentityModel.Clients.ActiveDirectory.AuthenticationContext" -ArgumentList $authority
    # Acquire token
    $platformParameters = New-Object "Microsoft.IdentityModel.Clients.ActiveDirectory.PlatformParameters" -ArgumentList "Auto"
    $authResult = $authContext.AcquireTokenAsync($resourceAppIdURI, $clientId, $redirectUri, $platformParameters).result
    # Generate auth header 
    $authHeader = $authResult.CreateAuthorizationHeader()
    # Set HTTP request headers to include Authorization header
    $headers = @{'x-ms-version'='2014-08-01';"Authorization" = $authHeader}

    ########################################################################################################################
    # 1. Retrieve wrapped BEK
    # 2. Make KeyVault REST API call to unwrap the BEK
    # 3. Convert the Base64Url string returned by KeyVault unwrap to Base64 string 
    # 4. Convert Base64 string to bytes and write to the BEK file
    ########################################################################################################################

    #Get wrapped BEK and place it in JSON object to send to KeyVault REST API
    $keyVaultSecret = Get-AzKeyVaultSecret -VaultName $keyVaultName -Name $secretName
    $bstr = [Runtime.InteropServices.Marshal]::SecureStringToBSTR($keyVaultSecret.SecretValue)
    $wrappedBekSecretBase64 = [Runtime.InteropServices.Marshal]::PtrToStringAuto($bstr)
    $jsonObject = @"
    {
    "alg": "RSA-OAEP",
    "value" : "$wrappedBekSecretBase64"
    }
    "@

    #Get KEK Url
    $kekUrl = (Get-AzKeyVaultKey -VaultName $keyVaultName -Name $kekName).Key.Kid;
    $unwrapKeyRequestUrl = $kekUrl+ "/unwrapkey?api-version=2015-06-01";

    #Call KeyVault REST API to Unwrap 
    $result = Invoke-RestMethod -Method POST -Uri $unwrapKeyRequestUrl -Headers $headers -Body $jsonObject -ContentType "application/json" -Debug

    #Convert Base64Url string returned by KeyVault unwrap to Base64 string
    $base64UrlBek = $result.value;
    $base64Bek = $base64UrlBek.Replace('-', '+');
    $base64Bek = $base64Bek.Replace('_', '/');
    if($base64Bek.Length %4 -eq 2)
    {
        $base64Bek+= '==';
    }
    elseif($base64Bek.Length %4 -eq 3)
    {
        $base64Bek+= '=';
    }

    #Convert base64 string to bytes and write to BEK file
    $bekFileBytes = [System.Convert]::FromBase64String($base64Bek);
    [System.IO.File]::WriteAllBytes($bekFilePath,$bekFileBytes)

    #Delete the key from the memory
    [Runtime.InteropServices.Marshal]::ZeroFreeBSTR($bstr)
    clear-variable -name wrappedBekSecretBase64
    ```
3. Définissez les paramètres. Le script va traiter le secret KEK pour créer la clé BEK, puis l’enregistrer dans un dossier local de la machine virtuelle de récupération. Si vous recevez des erreurs lors de l’exécution du script, consultez la section [Résolution des problèmes de script](#script-troubleshooting).

4. La sortie suivante s’affiche au début de l’exécution du script :

    GAC    Version        Emplacement                                                                              
    ---    -------        --------                                                                              
    Faux  v4.0.30319     C:\Program Files\WindowsPowerShell\Modules\Az.Accounts\...  Faux  v4.0.30319     C:\Program Files\WindowsPowerShell\Modules\Az.Accounts\...

    Une fois l’exécution du script terminée, vous voyez la sortie suivante :

    ```output
    VERBOSE: POST https://myvault.vault.azure.net/keys/rondomkey/<KEY-ID>/unwrapkey?api-
    version=2015-06-01 with -1-byte payload
    VERBOSE: received 360-byte response of content type application/json; charset=utf-8
    ```

5. Pour déverrouiller le disque attaché à l’aide du fichier BEK, exécutez la commande suivante :

    ```powershell
    manage-bde -unlock F: -RecoveryKey "C:\BEK\EF7B2F5A-50C6-4637-9F13-7F599C12F85C.BEK
    ```
    Dans cet exemple, le disque du système d’exploitation attaché est le lecteur F. Vérifiez que vous utilisez la bonne lettre de lecteur. 

6. Une fois que le disque a été déverrouillé à l’aide de la clé BEK, détachez le disque de la machine virtuelle de récupération, puis recréez la machine virtuelle à l’aide de ce nouveau disque de système d’exploitation. 

    > [!NOTE]
    > L’échange de disque de système d’exploitation n’est pas pris en charge pour les machines virtuelles utilisant le chiffrement de disque.

7. Si la nouvelle machine virtuelle ne peut toujours pas démarrer normalement, essayez l’une des étapes suivantes après avoir déverrouillé le lecteur :

    - Suspendez la protection pour désactiver temporairement BitLocker en exécutant la commande suivante :

    ```console
    manage-bde -protectors -disable F: -rc 0
    ```

    - Déchiffrez entièrement le lecteur. Pour ce faire, exécutez la commande suivante :

    ```console
    manage-bde -off F:
    ```

## <a name="script-troubleshooting"></a>Résolution des problèmes de script

**Erreur : Impossible de charger le fichier ou l’assembly**

Cette erreur se produit parce que les chemins d’accès des assemblys ADAL sont incorrects. Vous pouvez rechercher le dossier `Az.Accounts` pour trouver le chemin d’accès correct.

**Erreur : Get-AzKeyVaultSecret ou Get-AzKeyVaultSecret n’est pas reconnu en tant que nom d’une cmdlet**

Si vous utilisez l’ancien module AZ PowerShell, vous devez remplacer les deux commandes par `Get-AzureKeyVaultSecret` et `Get-AzureKeyVaultSecret`.

**Exemples de paramètres**

| Paramètres  | Exemple de valeur  |Commentaires   |
|---|---|---|
|  $keyVaultName | myKeyVault2112852926  | Nom du coffre de clés qui stocke la clé |
|$kekName   |mykey   | Nom de la clé utilisée pour chiffrer la machine virtuelle|
|$secretName   |7EB4F531-5FBA-4970-8E2D-C11FD6B0C69D  | Nom du secret de la clé de la machine virtuelle|
|$bekFilePath   |c:\bek\7EB4F531-5FBA-4970-8E2D-C11FD6B0C69D.BEK |Chemin d’accès pour l’écriture du fichier BEK.|
|$adTenant  |contoso.onmicrosoft.com   | Nom de domaine complet ou GUID de votre Azure Active Directory qui héberge le coffre de clés |
