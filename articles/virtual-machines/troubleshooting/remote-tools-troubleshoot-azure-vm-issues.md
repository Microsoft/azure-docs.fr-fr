---
title: Utiliser des outils à distance pour résoudre les problèmes de machine virtuelle Azure | Microsoft Docs
description: Découvrez PsExec, les scripts PowerShell et d’autres outils distants que vous pouvez utiliser pour résoudre les problèmes des machines virtuelles Azure distantes sans utiliser RDP.
services: virtual-machines-windows
documentationcenter: ''
author: Deland-Han
manager: dcscontentpm
editor: ''
tags: ''
ms.service: virtual-machines
ms.topic: troubleshooting
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: azurecli
ms.date: 01/11/2018
ms.author: delhan
ms.openlocfilehash: c5974388c096c9bc8693c5fc2cf918989c6eadd3
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96488729"
---
# <a name="use-remote-tools-to-troubleshoot-azure-vm-issues"></a>Utiliser des outils à distance pour résoudre les problèmes de machine virtuelle Azure

Quand vous résolvez des problèmes sur une machine virtuelle Azure, vous pouvez vous connecter à celle-ci à l’aide des outils à distance qui sont abordés dans cet article au lieu d’utiliser le protocole RDP (Remote Desktop Protocol).

## <a name="serial-console"></a>Console série

Utilisez une [console série pour machines virtuelles Azure](serial-console-windows.md) afin d’exécuter des commandes sur la machine virtuelle Azure distante.

## <a name="remote-cmd"></a>CMD à distance

Téléchargez [PsExec](/sysinternals/downloads/psexec). Connectez-vous à la machine virtuelle en exécutant la commande suivante :

```cmd
psexec \\<computer>-u user -s cmd
```

>[!NOTE]
>* Cette commande doit être exécutée sur un ordinateur figurant dans le même réseau virtuel.
>* Vous pouvez utiliser DIP ou HostName pour remplacer \<computer>.
>* Le paramètre -s permet de s’assurer que la commande est appelée à l’aide du compte système (autorisation d’administrateur).
>* PsExec utilise les ports TCP 135 et 445. Par conséquent, ces deux ports doivent être ouverts sur le pare-feu.

## <a name="run-command"></a>Commande d’exécution

Pour plus d’informations sur la façon d’utiliser la fonctionnalité Run Command pour exécuter des scripts sur la machine virtuelle, consultez [Exécuter des scripts PowerShell dans votre machine virtuelle Windows à l’aide de Run Command](../windows/run-command.md).

## <a name="custom-script-extension"></a>Extension de script personnalisé

Vous pouvez utiliser la fonctionnalité Extension de script personnalisé pour exécuter un script personnalisé sur la machine virtuelle cible. Pour utiliser cette fonctionnalité, les conditions suivantes doivent être remplies :

* La machine virtuelle dispose d’une connectivité
* L’agent de machine virtuelle Azure est installé et fonctionne comme prévu sur la machine virtuelle.
* L’extension n’a encore jamais été installée sur la machine virtuelle.
 
  L’extension injecte le script uniquement la première fois qu’elle est utilisée. Si vous utilisez cette fonctionnalité ultérieurement, l’extension reconnaît qu’elle a déjà été utilisée et ne charge pas le nouveau script.

Chargez votre script sur un compte de stockage et générez son propre conteneur. Ensuite, exécutez le script suivant dans Azure PowerShell sur un ordinateur qui dispose d’une connectivité à la machine virtuelle.

### <a name="for-classic-deployment-model-vms"></a>Pour les machines virtuelles de modèle de déploiement classique

[!INCLUDE [classic-vm-deprecation](../../../includes/classic-vm-deprecation.md)]


```powershell
#Set up the basic variables.
$subscriptionID = "<<SUBSCRIPTION ID>>" 
$storageAccount = "<<STORAGE ACCOUNT>>" 
$localScript = "<<FULL PATH OF THE PS1 FILE TO EXECUTE ON THE VM>>" 
$blobName = "file.ps1" #Name you want for the blob in the storage.
$vmName = "<<VM NAME>>" 
$vmCloudService = "<<CLOUD SERVICE>>" #Resource group or cloud service where the VM is hosted. For example, for "demo305.cloudapp.net" the cloud service is going to be demo305.

#Set up the Azure PowerShell module, and ensure the access to the subscription.
Import-Module Azure
Add-AzureAccount  #Ensure login with the account associated with the subscription ID.
Get-AzureSubscription -SubscriptionId $subscriptionID | Select-AzureSubscription

#Set up the access to the storage account, and upload the script.
$storageKey = (Get-AzureStorageKey -StorageAccountName $storageAccount).Primary
$context = New-AzureStorageContext -StorageAccountName $storageAccount -StorageAccountKey $storageKey
$container = "cse" + (Get-Date -Format yyyyMMddhhmmss)<
New-AzureStorageContainer -Name $container -Permission Off -Context $context
Set-AzureStorageBlobContent -File $localScript -Container $container -Blob $blobName  -Context $context

#Push the script into the VM.
$vm = Get-AzureVM -ServiceName $vmCloudService -Name $vmName
Set-AzureVMCustomScriptExtension "CustomScriptExtension" -VM $vm -StorageAccountName $storageAccount -StorageAccountKey $storagekey -ContainerName $container -FileName $blobName -Run $blobName | Update-AzureVM
```

### <a name="for-azure-resource-manager-vms"></a>Pour les machines virtuelles Azure Resource Manager

 

```powershell
#Set up the basic variables.
$subscriptionID = "<<SUBSCRIPTION ID>>"
$storageAccount = "<<STORAGE ACCOUNT>>"
$storageRG = "<<RESOURCE GROUP OF THE STORAGE ACCOUNT>>" 
$localScript = "<<FULL PATH OF THE PS1 FILE TO EXECUTE ON THE VM>>" 
$blobName = "file.ps1" #Name you want for the blob in the storage.
$vmName = "<<VM NAME>>" 
$vmResourceGroup = "<<RESOURCE GROUP>>"
$vmLocation = "<<DATACENTER>>" 
 
#Set up the Azure PowerShell module, and ensure the access to the subscription.
Login-AzAccount #Ensure login with the account associated with the subscription ID.
Get-AzSubscription -SubscriptionId $subscriptionID | Select-AzSubscription

#Set up the access to the storage account, and upload the script.
$storageKey = (Get-AzStorageAccountKey -ResourceGroupName $storageRG -Name $storageAccount).Value[0]
$context = New-AzureStorageContext -StorageAccountName $storageAccount -StorageAccountKey $storageKey
$container = "cse" + (Get-Date -Format yyyyMMddhhmmss)
New-AzureStorageContainer -Name $container -Permission Off -Context $context
Set-AzureStorageBlobContent -File $localScript -Container $container -Blob $blobName  -Context $context

#Push the script into the VM.
Set-AzVMCustomScriptExtension -Name "CustomScriptExtension" -ResourceGroupName $vmResourceGroup -VMName $vmName -Location $vmLocation -StorageAccountName $storageAccount -StorageAccountKey $storagekey -ContainerName $container -FileName $blobName -Run $blobName
```

## <a name="remote-powershell"></a>PowerShell à distance

>[!NOTE]
>Le port TCP 5986 (HTTPS) doit être ouvert pour que vous puissiez utiliser cette option.
>
>Pour les machines virtuelles Azure Resource Manager, vous devez ouvrir le port 5986 sur le groupe de sécurité réseau (NSG). Pour plus d’informations, consultez Groupes de sécurité. 
>
>Pour les machines virtuelles RDFE, vous devez avoir un point de terminaison ayant un port privé (5986) et un port public. Ensuite, vous devez également ouvrir ce port public sur le groupe de sécurité réseau.

### <a name="set-up-the-client-computer"></a>Configurer l’ordinateur client

Pour utiliser PowerShell pour vous connecter à distance à la machine virtuelle, vous devez d’abord configurer l’ordinateur client afin d’autoriser la connexion. Pour ce faire, ajoutez la machine virtuelle à la liste des hôtes approuvés PowerShell en exécutant la commande suivante, selon le cas.

Pour ajouter une machine virtuelle à la liste des hôtes approuvés :

```powershell
Set-Item wsman:\localhost\Client\TrustedHosts -value <ComputerName>
```

Pour ajouter plusieurs machines virtuelles à la liste des hôtes approuvés :

```powershell
Set-Item wsman:\localhost\Client\TrustedHosts -value <ComputerName1>,<ComputerName2>
```

Pour ajouter tous les ordinateurs à la liste des hôtes approuvés :

```powershell
Set-Item wsman:\localhost\Client\TrustedHosts -value *
```

### <a name="enable-remoteps-on-the-vm"></a>Activer RemotePS sur la machine virtuelle

Pour les machines virtuelles créées à l’aide du modèle de déploiement classique, utilisez l’extension de script personnalisé pour exécuter le script suivant :

```powershell
Enable-PSRemoting -Force
New-NetFirewallRule -Name "Allow WinRM HTTPS" -DisplayName "WinRM HTTPS" -Enabled True -Profile Any -Action Allow -Direction Inbound -LocalPort 5986 -Protocol TCP
$thumbprint = (New-SelfSignedCertificate -DnsName $env:COMPUTERNAME -CertStoreLocation Cert:\LocalMachine\My).Thumbprint
$command = "winrm create winrm/config/Listener?Address=*+Transport=HTTPS @{Hostname=""$env:computername""; CertificateThumbprint=""$thumbprint""}"
cmd.exe /C $command
```

Pour les machines virtuelles Azure Resource Manager, utilisez les commandes d’exécution à partir du portail pour exécuter le script EnableRemotePS :

![Commande d’exécution](./media/remote-tools-troubleshoot-azure-vm-issues/run-command.png)

### <a name="connect-to-the-vm"></a>Connexion à la machine virtuelle

Exécutez la commande suivante en fonction de l’emplacement de l’ordinateur client :

* En dehors du réseau virtuel ou du déploiement

  * Pour une machine virtuelle créée à l’aide du modèle de déploiement classique, exécutez la commande suivante :

    ```powershell
    $Skip = New-PSSessionOption -SkipCACheck -SkipCNCheck
    Enter-PSSession -ComputerName  "<<CLOUDSERVICENAME.cloudapp.net>>" -port "<<PUBLIC PORT NUMBER>>" -Credential (Get-Credential) -useSSL -SessionOption $Skip
    ```

  * Pour une machine virtuelle Azure Resource Manager, commencez par ajouter un nom DNS à l’adresse IP publique. Pour obtenir des étapes détaillées, consultez [Créer un nom de domaine complet dans le portail Azure pour une machine virtuelle Windows](../create-fqdn.md). Exécutez ensuite la commande suivante :

    ```powershell
    $Skip = New-PSSessionOption -SkipCACheck -SkipCNCheck
    Enter-PSSession -ComputerName "<<DNSname.DataCenter.cloudapp.azure.com>>" -port "5986" -Credential (Get-Credential) -useSSL -SessionOption $Skip
    ```

* Dans le réseau virtuel ou le déploiement, exécutez la commande suivante :
  
  ```powershell
  $Skip = New-PSSessionOption -SkipCACheck -SkipCNCheck
  Enter-PSSession -ComputerName  "<<HOSTNAME>>" -port 5986 -Credential (Get-Credential) -useSSL -SessionOption $Skip
  ```

>[!NOTE] 
>La définition de l’indicateur SkipCaCheck contourne la nécessité d’importer un certificat sur la machine virtuelle quand vous démarrez la session.

Vous pouvez également utiliser l’applet de commande Invoke-Command pour exécuter un script à distance sur la machine virtuelle.

```powershell
Invoke-Command -ComputerName "<<COMPUTERNAME>" -ScriptBlock {"<<SCRIPT BLOCK>>"}
```

## <a name="remote-registry"></a>Registre distant

>[!NOTE]
>Le port TCP 135 ou 445 doit être ouvert pour que vous puissiez utiliser cette option.
>
>Pour les machines virtuelles Azure Resource Manager, vous devez ouvrir le port 5986 sur le groupe de sécurité réseau. Pour plus d’informations, consultez Groupes de sécurité. 
>
>Pour les machines virtuelles RDFE, vous devez avoir un point de terminaison ayant un port privé 5986 et un port public. Vous devez également ouvrir ce port public sur le groupe de sécurité réseau.

1. À partir d’une autre machine virtuelle figurant dans le même réseau virtuel, ouvrez l’Éditeur du Registre (regedit.exe).

2. Sélectionnez **Fichier** > **Connexion au Registre réseau**.

   ![Éditeur du Registre](./media/remote-tools-troubleshoot-azure-vm-issues/remote-registry.png) 

3. Recherchez la machine virtuelle cible par **nom d’hôte** ou **adresse IP dynamique** (préférable) en l’entrant dans la zone **Entrer le nom de l’objet à sélectionner**.

   ![Zone Entrer le nom de l’objet à sélectionner](./media/remote-tools-troubleshoot-azure-vm-issues/input-computer-name.png) 
 
4. Entrez les informations d’identification pour la machine virtuelle cible.

5. Apportez les modifications nécessaires au Registre.

## <a name="remote-services-console"></a>Console Services à distance

>[!NOTE]
>Le port TCP 135 ou 445 doit être ouvert pour que vous puissiez utiliser cette option.
>
>Pour les machines virtuelles Azure Resource Manager, vous devez ouvrir le port 5986 sur le groupe de sécurité réseau. Pour plus d’informations, consultez Groupes de sécurité. 
>
>Pour les machines virtuelles RDFE, vous devez avoir un point de terminaison ayant un port privé 5986 et un port public. Vous devez également ouvrir ce port public sur le groupe de sécurité réseau.

1. À partir d’une autre machine virtuelle figurant dans le même réseau virtuel, ouvrez une instance de **Services.msc**.

2. Cliquez avec le bouton droit sur **Services (Local)** .

3. Sélectionnez **Se connecter à un autre ordinateur**.

   ![Service distant](./media/remote-tools-troubleshoot-azure-vm-issues/remote-services.png)

4. Entrez l’adresse IP dynamique de la machine virtuelle cible.

   ![Entrer l’adresse IP dynamique](./media/remote-tools-troubleshoot-azure-vm-issues/input-ip-address.png)

5. Apportez les modifications nécessaires aux services.

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur l’applet de commande Enter-PSSession, consultez [Enter-PSSession](/powershell/module/microsoft.powershell.core/enter-pssession?view=powershell-5.1).
- Pour plus d’informations sur l’extension de script personnalisé pour Windows à l’aide du modèle de déploiement classique, consultez [Extension de script personnalisé pour Windows](../extensions/custom-script-windows.md).
- PsExec fait partie de la [Suite PSTools](https://download.sysinternals.com/files/PSTools.zip).
- Pour plus d’informations sur la Suite PSTools, consultez [PSTools](/sysinternals/downloads/pstools).