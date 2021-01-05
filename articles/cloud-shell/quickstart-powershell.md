---
title: Démarrage rapide pour Azure Cloud Shell – PowerShell
description: Découvrez comment utiliser PowerShell dans votre navigateur avec Azure Cloud Shell.
author: maertendmsft
ms.author: damaerte
tags: azure-resource-manager
ms.service: azure
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.topic: article
ms.date: 10/18/2018
ms.openlocfilehash: d4a7f1453ec686cfa16d260101ba81f429ce1da0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89469454"
---
# <a name="quickstart-for-powershell-in-azure-cloud-shell"></a>Démarrage rapide de PowerShell dans Azure Cloud Shell

Ce document explique comment utiliser PowerShell dans Cloud Shell dans le [portail Azure](https://portal.azure.com/).

> [!NOTE]
> Un démarrage rapide de [Bash dans Azure Cloud Shell](quickstart.md) est également disponible.

## <a name="start-cloud-shell"></a>Démarrer Cloud Shell

1. Cliquez sur le bouton **Cloud Shell** dans la barre de navigation supérieure du portail Azure

   ![Capture d’écran montrant comment démarrer Azure Cloud Shell à partir du portail Azure.](media/quickstart-powershell/shell-icon.png)

2. Sélectionnez l’environnement PowerShell à partir de la liste déroulante pour basculer vers le lecteur Azure`(Azure:)`

   ![Capture d’écran montrant comment sélectionner l’environnement PowerShell pour Azure Cloud Shell.](media/quickstart-powershell/environment-ps.png)

## <a name="run-powershell-commands"></a>Exécuter des commandes PowerShell

Exécuter des commandes PowerShell normales dans Cloud Shell, telles que :

```azurepowershell-interactive
PS Azure:\> Get-Date

# Expected Output
Friday, July 27, 2018 7:08:48 AM

PS Azure:\> Get-AzVM -Status

# Expected Output
ResourceGroupName       Name       Location                VmSize   OsType     ProvisioningState  PowerState
-----------------       ----       --------                ------   ------     -----------------  ----------
MyResourceGroup2        Demo        westus         Standard_DS1_v2  Windows    Succeeded           running
MyResourceGroup         MyVM1       eastus            Standard_DS1  Windows    Succeeded           running
MyResourceGroup         MyVM2       eastus   Standard_DS2_v2_Promo  Windows    Succeeded           deallocated
```

## <a name="navigate-azure-resources"></a>Parcourir les ressources Azure

 1. Répertorier tous vos abonnements à partir du lecteur `Azure`

    ```azurepowershell-interactive
    PS Azure:\> dir
    ```

 2. `cd` sur votre abonnement préféré

    ```azurepowershell-interactive
    PS Azure:\> cd MySubscriptionName
    PS Azure:\MySubscriptionName>
    ```

 3. Afficher toutes vos ressources Azure sous l’abonnement actuel

    Saisir `dir` pour répertorier plusieurs vues de vos ressources Azure.

    ```azurepowershell-interactive
    PS Azure:\MySubscriptionName> dir

        Directory: azure:\MySubscriptionName

    Mode Name
    ---- ----
    +    AllResources
    +    ResourceGroups
    +    StorageAccounts
    +    VirtualMachines
    +    WebApps
    ```

### <a name="allresources-view"></a>Vue AllResources

Saisissez `dir` sous le répertoire `AllResources` pour afficher vos ressources Azure.

```azurepowershell-interactive
PS Azure:\MySubscriptionName> dir AllResources
```

### <a name="explore-resource-groups"></a>Explorer les groupes de ressources

 Vous pouvez accéder au répertoire `ResourceGroups` et à l’intérieur d’un groupe de ressources spécifique rechercher des machines virtuelles.

```azurepowershell-interactive
PS Azure:\MySubscriptionName> cd ResourceGroups\MyResourceGroup1\Microsoft.Compute\virtualMachines

PS Azure:\MySubscriptionName\ResourceGroups\MyResourceGroup1\Microsoft.Compute\virtualMachines> dir


    Directory: Azure:\MySubscriptionName\ResourceGroups\MyResourceGroup1\Microsoft.Compute\virtualMachines


VMName    Location   ProvisioningState VMSize          OS            SKU             OSVersion AdminUserName  NetworkInterfaceName
------    --------   ----------------- ------          --            ---             --------- -------------  --------------------
TestVm1   westus     Succeeded         Standard_DS2_v2 WindowsServer 2016-Datacenter Latest    AdminUser      demo371
TestVm2   westus     Succeeded         Standard_DS1_v2 WindowsServer 2016-Datacenter Latest    AdminUser      demo271
```

> [!NOTE]
> Vous pouvez remarquer que la deuxième fois que vous tapez `dir`, Cloud Shell est en mesure d’afficher les éléments beaucoup plus rapidement.
> Cela est rendu possible par les éléments enfants mis en mémoire cache pour améliorer l’expérience utilisateur.
Toutefois, vous pouvez toujours utiliser `dir -Force` pour obtenir des données actualisées.

### <a name="navigate-storage-resources"></a>Parcourir les ressources de stockage

En accédant au répertoire `StorageAccounts`, vous pouvez facilement parcourir la totalité de vos ressources de stockage.

```azurepowershell-interactive
PS Azure:\MySubscriptionName\StorageAccounts\MyStorageAccountName\Files> dir

    Directory: Azure:\MySubscriptionNameStorageAccounts\MyStorageAccountName\Files

Name          ConnectionString
----          ----------------
MyFileShare1  \\MyStorageAccountName.file.core.windows.net\MyFileShare1;AccountName=MyStorageAccountName AccountKey=<key>
MyFileShare2  \\MyStorageAccountName.file.core.windows.net\MyFileShare2;AccountName=MyStorageAccountName AccountKey=<key>
MyFileShare3  \\MyStorageAccountName.file.core.windows.net\MyFileShare3;AccountName=MyStorageAccountName AccountKey=<key>
```

La chaîne de connexion vous permet d’utiliser la commande suivante pour monter le partage Azure Files.

```azurepowershell-interactive
net use <DesiredDriveLetter>: \\<MyStorageAccountName>.file.core.windows.net\<MyFileShareName> <AccountKey> /user:Azure\<MyStorageAccountName>
```

Pour en savoir plus, consultez [Montage d'un partage Azure Files et accès au partage dans Windows][azmount].

Vous pouvez également parcourir les répertoires sous le partage Azure Files comme suit :

```azurepowershell-interactive
PS Azure:\MySubscriptionName\StorageAccounts\MyStorageAccountName\Files> cd .\MyFileShare1\
PS Azure:\MySubscriptionName\StorageAccounts\MyStorageAccountName\Files\MyFileShare1> dir

Mode  Name
----  ----
+     TestFolder
.     hello.ps1
```

### <a name="interact-with-virtual-machines"></a>Interagir avec des machines virtuelles

Vous pouvez trouver toutes vos machines virtuelles sous l’abonnement actuel via le répertoire `VirtualMachines`.

```azurepowershell-interactive
PS Azure:\MySubscriptionName\VirtualMachines> dir

    Directory: Azure:\MySubscriptionName\VirtualMachines


Name       ResourceGroupName  Location  VmSize          OsType              NIC ProvisioningState  PowerState
----       -----------------  --------  ------          ------              --- -----------------  ----------
TestVm1    MyResourceGroup1   westus    Standard_DS2_v2 Windows       my2008r213         Succeeded     stopped
TestVm2    MyResourceGroup1   westus    Standard_DS1_v2 Windows          jpstest         Succeeded deallocated
TestVm10   MyResourceGroup2   eastus    Standard_DS1_v2 Windows           mytest         Succeeded     running
```

#### <a name="invoke-powershell-script-across-remote-vms"></a>Appeler un script PowerShell sur les machines virtuelles à distance

 > [!WARNING]
 > Reportez-vous à [Résolution des problèmes de gestion à distance des machines virtuelles Azure](troubleshooting.md#troubleshooting-remote-management-of-azure-vms).

  En supposant que vous avez une machine virtuelle, MyVM1, utilisons `Invoke-AzVMCommand` pour appeler un bloc de script PowerShell sur la machine distante.

  ```azurepowershell-interactive
  Enable-AzVMPSRemoting -Name MyVM1 -ResourceGroupname MyResourceGroup
  Invoke-AzVMCommand -Name MyVM1 -ResourceGroupName MyResourceGroup -Scriptblock {Get-ComputerInfo} -Credential (Get-Credential)
  ```

  Vous pouvez également commencer par accéder au répertoire virtualMachines, puis exécuter `Invoke-AzVMCommand` comme suit.

  ```azurepowershell-interactive
  PS Azure:\> cd MySubscriptionName\ResourceGroups\MyResourceGroup\Microsoft.Compute\virtualMachines
  PS Azure:\MySubscriptionName\ResourceGroups\MyResourceGroup\Microsoft.Compute\virtualMachines> Get-Item MyVM1 | Invoke-AzVMCommand -Scriptblock {Get-ComputerInfo} -Credential (Get-Credential)

  # You will see output similar to the following:

  PSComputerName                                          : 65.52.28.207
  RunspaceId                                              : 2c2b60da-f9b9-4f42-a282-93316cb06fe1
  WindowsBuildLabEx                                       : 14393.1066.amd64fre.rs1_release_sec.170327-1835
  WindowsCurrentVersion                                   : 6.3
  WindowsEditionId                                        : ServerDatacenter
  WindowsInstallationType                                 : Server
  WindowsInstallDateFromRegistry                          : 5/18/2017 11:26:08 PM
  WindowsProductId                                        : 00376-40000-00000-AA947
  WindowsProductName                                      : Windows Server 2016 Datacenter
  WindowsRegisteredOrganization                           :
   ...
  ```

#### <a name="interactively-log-on-to-a-remote-vm"></a>Se connecter de manière interactive à une machine virtuelle à distance

Vous pouvez utiliser `Enter-AzVM` pour la connexion interactive à une machine virtuelle s’exécutant dans Azure.

  ```azurepowershell-interactive
  PS Azure:\> Enter-AzVM -Name MyVM1 -ResourceGroupName MyResourceGroup -Credential (Get-Credential)
  ```

Vous pouvez également commencer par naviguer vers le répertoire `VirtualMachines` et exécuter `Enter-AzVM` comme suit.

  ```azurepowershell-interactive
 PS Azure:\MySubscriptionName\ResourceGroups\MyResourceGroup\Microsoft.Compute\virtualMachines> Get-Item MyVM1 | Enter-AzVM -Credential (Get-Credential)
 ```

### <a name="discover-webapps"></a>Découvrir les applications web

En accédant au répertoire `WebApps`, vous pouvez facilement parcourir vos ressources d’applications web.

```azurepowershell-interactive
PS Azure:\MySubscriptionName> dir .\WebApps\

    Directory: Azure:\MySubscriptionName\WebApps

Name            State    ResourceGroup      EnabledHostNames                  Location
----            -----    -------------      ----------------                  --------
mywebapp1       Stopped  MyResourceGroup1   {mywebapp1.azurewebsites.net...   West US
mywebapp2       Running  MyResourceGroup2   {mywebapp2.azurewebsites.net...   West Europe
mywebapp3       Running  MyResourceGroup3   {mywebapp3.azurewebsites.net...   South Central US

# You can use Azure cmdlets to Start/Stop your web apps
PS Azure:\MySubscriptionName\WebApps> Start-AzWebApp -Name mywebapp1 -ResourceGroupName MyResourceGroup1

Name           State    ResourceGroup        EnabledHostNames                   Location
----           -----    -------------        ----------------                   --------
mywebapp1      Running  MyResourceGroup1     {mywebapp1.azurewebsites.net ...   West US

# Refresh the current state with -Force
PS Azure:\MySubscriptionName\WebApps> dir -Force

    Directory: Azure:\MySubscriptionName\WebApps

Name            State    ResourceGroup      EnabledHostNames                  Location
----            -----    -------------      ----------------                  --------
mywebapp1       Running  MyResourceGroup1   {mywebapp1.azurewebsites.net...   West US
mywebapp2       Running  MyResourceGroup2   {mywebapp2.azurewebsites.net...   West Europe
mywebapp3       Running  MyResourceGroup3   {mywebapp3.azurewebsites.net...   South Central US
```

## <a name="ssh"></a>SSH

Pour vous authentifier auprès des serveurs ou machines virtuelles à l’aide de SSH, générez la paire de clés publique/privée dans le Cloud Shell et publiez la clé publique dans `authorized_keys` sur l’ordinateur distant, tel que `/home/user/.ssh/authorized_keys`.

> [!NOTE]
> Vous pouvez créer des clés publique/privée SSH à l’aide de `ssh-keygen` et les publier sur `$env:USERPROFILE\.ssh` dans le Cloud Shell.

### <a name="using-ssh"></a>Utilisation de SSH

Suivez les instructions fournies [ici](../virtual-machines/linux/quick-create-powershell.md) pour créer une configuration de machine virtuelle à l'aide de cmdlets Azure PowerShell.
Avant d’appeler `New-AzVM` pour lancer le déploiement, ajoutez la clé publique SSH à la configuration de machine virtuelle.
La machine virtuelle créée contient la clé publique à l’emplacement `~\.ssh\authorized_keys` ; vous pouvez ainsi ouvrir une session SSH sur la machine virtuelle sans recourir à des informations d’identification.

```azurepowershell-interactive
# Create VM config object - $vmConfig using instructions on linked page above

# Generate SSH keys in Cloud Shell
ssh-keygen -t rsa -b 2048 -f $HOME\.ssh\id_rsa 

# Ensure VM config is updated with SSH keys
$sshPublicKey = Get-Content "$HOME\.ssh\id_rsa.pub"
Add-AzVMSshPublicKey -VM $vmConfig -KeyData $sshPublicKey -Path "/home/azureuser/.ssh/authorized_keys"

# Create a virtual machine
New-AzVM -ResourceGroupName <yourResourceGroup> -Location <vmLocation> -VM $vmConfig

# SSH to the VM
ssh azureuser@MyVM.Domain.Com
```

## <a name="list-available-commands"></a>Répertorier les commandes disponibles

Sous le lecteur `Azure`, tapez `Get-AzCommand` pour obtenir les commandes Azure spécifiques au contexte.

Vous pouvez également continuer d’utiliser `Get-Command *az* -Module Az.*` pour rechercher les commandes Azure disponibles.

## <a name="install-custom-modules"></a>Installer des modules personnalisés

Vous pouvez exécuter `Install-Module` pour installer les modules à partir de [PowerShell Gallery][gallery].

## <a name="get-help"></a>Get-Help

Tapez `Get-Help` pour obtenir des informations sur PowerShell dans Azure Cloud Shell.

```azurepowershell-interactive
Get-Help
```

Pour une commande spécifique, vous pouvez toujours exécuter `Get-Help` suivi d’une applet de commande.

```azurepowershell-interactive
Get-Help Get-AzVM
```

## <a name="use-azure-files-to-store-your-data"></a>Utiliser Azure Files pour stocker vos données

Vous pouvez créer un script, par exemple `helloworld.ps1`, et l’enregistrer dans votre `clouddrive` pour l’utiliser dans les sessions d’interpréteur de commandes.

```azurepowershell-interactive
cd $HOME\clouddrive
# Create a new file in clouddrive directory
New-Item helloworld.ps1
# Open the new file for editing
code .\helloworld.ps1
# Add the content, such as 'Hello World!'
.\helloworld.ps1
Hello World!
```

La prochaine fois que vous utiliserez PowerShell dans Cloud Shell, le fichier `helloworld.ps1` existera sous le répertoire `$HOME\clouddrive` qui monte le partage de fichiers Azure.

## <a name="use-custom-profile"></a>Utiliser le profil personnalisé

Vous pouvez personnaliser votre environnement PowerShell en créant un ou plusieurs profils PowerShell `profile.ps1` (ou `Microsoft.PowerShell_profile.ps1`).
Enregistrez ces profils sous `$profile.CurrentUserAllHosts` (ou `$profile.CurrentUserAllHosts`) pour qu’ils puissent être chargés dans chaque session PowerShell dans Cloud Shell.

Pour savoir comment créer un profil, consultez [À propos des profils][profile].

## <a name="use-git"></a>Utiliser Git

Pour cloner un référentiel Git dans Cloud Shell, vous devez créer un [jeton d'accès personnel][githubtoken] et l'utiliser comme nom d'utilisateur. Dès que vous disposez de votre jeton, clonez le dépôt comme suit :

```azurepowershell-interactive
  git clone https://<your-access-token>@github.com/username/repo.git
```

## <a name="exit-the-shell"></a>Quittez l’interpréteur de commandes.

Entrez `exit` pour mettre fin à la session.

[bashqs]:quickstart.md
[gallery]:https://www.powershellgallery.com/
[customex]:https://docs.microsoft.com/azure/virtual-machines/windows/extensions-customscript
[profile]: /powershell/module/microsoft.powershell.core/about/about_profiles
[azmount]: ../storage/files/storage-how-to-use-files-windows.md
[githubtoken]: https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/