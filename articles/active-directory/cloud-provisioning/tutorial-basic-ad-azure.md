---
title: Tutoriel - Environnement Active Directory local et Azure AD simple
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: tutorial
ms.date: 12/02/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0b6acaa76d21cb1321feb5466c4fb7a9bacfa846
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96348107"
---
# <a name="tutorial-basic-active-directory-environment"></a>Tutoriel : Environnement Azure Active Directory simple

Ce tutoriel vous guide dans la création d’un environnement Active Directory simple. 

![Schéma illustrant un environnement Azure AD de base.](media/tutorial-single-forest/diagram1.png)

Vous pouvez utiliser l’environnement que vous créez dans le tutoriel pour tester divers aspects des scénarios d’identité hybride et en faire un prérequis pour certains tutoriels.  Si vous disposez déjà d’un environnement Active Directory existant, vous pouvez l’utiliser comme substitut.  Ces informations sont fournies aux personnes qui commencent à partir de rien.

Ce tutoriel se compose des éléments suivants
## <a name="prerequisites"></a>Prérequis
Voici les conditions préalables requises pour suivre ce didacticiel.
- Un ordinateur où [Hyper-V](/windows-server/virtualization/hyper-v/hyper-v-technology-overview) est installé.  Il est recommandé d’utiliser [Windows 10](/virtualization/hyper-v-on-windows/about/supported-guest-os) ou [Windows Server 2016](/windows-server/virtualization/hyper-v/supported-windows-guest-operating-systems-for-hyper-v-on-windows).
- Une [carte réseau externe](/virtualization/hyper-v-on-windows/quick-start/connect-to-network) pour autoriser la machine virtuelle à communiquer avec Internet.
- Un [abonnement Azure](https://azure.microsoft.com/free)
- Une copie de Windows Server 2016
- [Microsoft .NET Framework 4.7.1](https://www.microsoft.com/download/details.aspx?id=56115)

> [!NOTE]
> Ce didacticiel utilise des scripts PowerShell pour vous permettre de créer l’environnement le plus vite possible.  Chacun des scripts utilise les variables déclarées au début des scripts.  Vous pouvez et devez modifier les variables pour qu’elles reflètent votre environnement.
>
>Les scripts utilisés créent un environnement Active Directory général avant d’installer le provisionnement cloud Azure AD Connect.  Elles sont pertinentes pour l’ensemble des didacticiels.
>
> Des copies des scripts PowerShell utilisés dans ce didacticiel sont disponibles sur GitHub, [ici](https://github.com/billmath/tutorial-phs).

## <a name="create-a-virtual-machine"></a>Création d'une machine virtuelle
Tout d’abord, pour obtenir un environnement d’identité hybride fonctionnel, vous devez créer une machine virtuelle qui sera utilisée en tant que serveur Active Directory local.  Effectuez les actions suivantes :

1. Ouvrez PowerShell ISE en tant qu’administrateur.
2. Exécutez le script suivant.

    ```powershell
    #Declare variables
    $VMName = 'DC1'
    $Switch = 'External'
    $InstallMedia = 'D:\ISO\en_windows_server_2016_updated_feb_2018_x64_dvd_11636692.iso'
    $Path = 'D:\VM'
    $VHDPath = 'D:\VM\DC1\DC1.vhdx'
    $VHDSize = '64424509440'

    #Create New Virtual Machine
    New-VM -Name $VMName -MemoryStartupBytes 16GB -BootDevice VHD -Path $Path -NewVHDPath $VHDPath -NewVHDSizeBytes $VHDSize  -Generation 2 -Switch $Switch  

    #Set the memory to be non-dynamic
    Set-VMMemory $VMName -DynamicMemoryEnabled $false

    #Add DVD Drive to Virtual Machine
    Add-VMDvdDrive -VMName $VMName -ControllerNumber 0 -ControllerLocation 1 -Path $InstallMedia

    #Mount Installation Media
    $DVDDrive = Get-VMDvdDrive -VMName $VMName

    #Configure Virtual Machine to Boot from DVD
    Set-VMFirmware -VMName $VMName -FirstBootDevice $DVDDrive 
    ```

## <a name="complete-the-operating-system-deployment"></a>Terminer le déploiement du système d’exploitation
Pour terminer la création de la machine virtuelle, vous devez terminer l’installation du système d’exploitation.

1. Gestionnaire Hyper-V, double-clic sur la machine virtuelle
2. Cliquez sur le bouton Démarrer.
3. Vous êtes invité à appuyer sur n’importe quelle touche pour démarrer à partir du CD ou DVD. Faites-le.
4. Sur l’écran de démarrage Windows Server, sélectionnez votre langue et cliquez sur **Suivant**.
5. Cliquez sur **Installer maintenant**.
6. Saisissez votre clé de licence et cliquez sur **Suivant**.
7. Acceptez les termes du contrat de licence et cliquez sur **Suivant**.
8. Sélectionnez **Personnalisé :  installer Windows uniquement (avancé)**
9. Cliquez sur **Suivant**.
10. Une fois l’installation terminée, redémarrez la machine virtuelle, connectez-vous et exécutez les mises à jour Windows pour vous assurer que la machine virtuelle est à jour.  Installez les dernières mises à jour.

## <a name="install-active-directory-prerequisites"></a>Installer les prérequis pour Active Directory
Maintenant que la machine virtuelle est en cours d’exécution,vous devez effectuer quelques opérations avant d’installer Active Directory.  Autrement dit, vous devez renommer la machine virtuelle, définir une adresse IP statique et des informations DNS, et installer les outils d’administration de serveur distant.   Effectuez les actions suivantes :

1. Ouvrez PowerShell ISE en tant qu’administrateur.
2. Exécutez le script suivant.

    ```powershell
    #Declare variables
    $ipaddress = "10.0.1.117" 
    $ipprefix = "24" 
    $ipgw = "10.0.1.1" 
    $ipdns = "10.0.1.117"
    $ipdns2 = "8.8.8.8" 
    $ipif = (Get-NetAdapter).ifIndex 
    $featureLogPath = "c:\poshlog\featurelog.txt" 
    $newname = "DC1"
    $addsTools = "RSAT-AD-Tools" 

    #Set static IP address
    New-NetIPAddress -IPAddress $ipaddress -PrefixLength $ipprefix -InterfaceIndex $ipif -DefaultGateway $ipgw 

    # Set the DNS servers
    Set-DnsClientServerAddress -InterfaceIndex $ipif -ServerAddresses ($ipdns, $ipdns2)

    #Rename the computer 
    Rename-Computer -NewName $newname -force 

    #Install features 
    New-Item $featureLogPath -ItemType file -Force 
    Add-WindowsFeature $addsTools 
    Get-WindowsFeature | Where installed >>$featureLogPath 

    #Restart the computer 
    Restart-Computer
    ```

## <a name="create-a-windows-server-ad-environment"></a>Créer un environnement Windows Server AD
Maintenant que la machine virtuelle a été créée et renommée, et qu’elle dispose d’une adresse IP statique, vous pouvez installer et configurer Active Directory Domain Services.  Effectuez les actions suivantes :

1. Ouvrez PowerShell ISE en tant qu’administrateur.
2. Exécutez le script suivant.

    ```powershell 
    #Declare variables
    $DatabasePath = "c:\windows\NTDS"
    $DomainMode = "WinThreshold"
    $DomainName = "contoso.com"
    $DomaninNetBIOSName = "CONTOSO"
    $ForestMode = "WinThreshold"
    $LogPath = "c:\windows\NTDS"
    $SysVolPath = "c:\windows\SYSVOL"
    $featureLogPath = "c:\poshlog\featurelog.txt" 
    $Password = "Pass1w0rd"
    $SecureString = ConvertTo-SecureString $Password -AsPlainText -Force

    #Install AD DS, DNS and GPMC 
    start-job -Name addFeature -ScriptBlock { 
    Add-WindowsFeature -Name "ad-domain-services" -IncludeAllSubFeature -IncludeManagementTools 
    Add-WindowsFeature -Name "dns" -IncludeAllSubFeature -IncludeManagementTools 
    Add-WindowsFeature -Name "gpmc" -IncludeAllSubFeature -IncludeManagementTools } 
    Wait-Job -Name addFeature 
    Get-WindowsFeature | Where installed >>$featureLogPath

    #Create New AD Forest
    Install-ADDSForest -CreateDnsDelegation:$false -DatabasePath $DatabasePath -DomainMode $DomainMode -DomainName $DomainName -SafeModeAdministratorPassword $SecureString -DomainNetbiosName $DomainNetBIOSName -ForestMode $ForestMode -InstallDns:$true -LogPath $LogPath -NoRebootOnCompletion:$false -SysvolPath $SysVolPath -Force:$true
    ```

## <a name="create-a-windows-server-ad-user"></a>Créer un utilisateur Windows Server AD
Maintenant que vous avez un environnement Active Directory, vous avez besoin d’un compte de test.  Ce compte sera créé dans l’environnement Active Directory local, puis synchronisé avec Azure AD.  Effectuez les actions suivantes :

1. Ouvrez PowerShell ISE en tant qu’administrateur.
2. Exécutez le script suivant.

    ```powershell 
    # Filename:    4_CreateUser.ps1
    # Description: Creates a user in Active Directory.  This is part of
    #              the Azure AD Connect password hash sync tutorial.
    #
    # DISCLAIMER:
    # Copyright (c) Microsoft Corporation. All rights reserved. This 
    # script is made available to you without any express, implied or 
    # statutory warranty, not even the implied warranty of 
    # merchantability or fitness for a particular purpose, or the 
    # warranty of title or non-infringement. The entire risk of the 
    # use or the results from the use of this script remains with you.
    #
    #
    #
    #
    #Declare variables
    $Givenname = "Allie"
    $Surname = "McCray"
    $Displayname = "Allie McCray"
    $Name = "amccray"
    $Password = "Pass1w0rd"
    $Identity = "CN=ammccray,CN=Users,DC=contoso,DC=com"
    $SecureString = ConvertTo-SecureString $Password -AsPlainText -Force


    #Create the user
    New-ADUser -Name $Name -GivenName $Givenname -Surname $Surname -DisplayName $Displayname -AccountPassword $SecureString

    #Set the password to never expire
    Set-ADUser -Identity $Identity -PasswordNeverExpires $true -ChangePasswordAtLogon $false -Enabled $true
    ```


## <a name="create-an-azure-ad-tenant"></a>Créer un locataire Azure AD
Vous devez à présent créer un locataire Azure AD pour synchroniser vos utilisateurs sur le cloud.  Pour créer un client Azure AD, procédez comme suit.

1. Accédez au [portail Azure](https://portal.azure.com) et connectez-vous avec un compte qui dispose d’un abonnement Azure.
2. Cliquez sur l’**icône plus (+)** et recherchez **Azure Active Directory**.
3. Dans la liste des résultats, sélectionnez sur **Azure Active Directory**.
4. Sélectionnez **Create** (Créer).</br>
![Capture d’écran montrant la page Azure Active Directory dans le portail Azure.](media/tutorial-single-forest/create1.png)</br>
5. Indiquez le **nom de l’organisation** avec le **nom de domaine initial**. Sélectionnez ensuite **Créer**. Votre annuaire est alors créé.
6. Une fois cette opération terminée, cliquez sur **ce lien** pour gérer l’annuaire.

## <a name="create-a-global-administrator-in-azure-ad"></a>Créer un administrateur général dans Azure AD
Maintenant que vous disposez d’un locataire Azure AD, vous allez créer un compte d’administrateur général.  Pour créer le compte d’administrateur général, procédez comme suit.

1.  Sous **Gérer**, sélectionnez **Utilisateurs**.</br>
![Capture d’écran montrant le menu « Vue d’ensemble » avec l’option « Utilisateurs » sélectionnée.](media/tutorial-single-forest/administrator1.png)</br>
2.  Sélectionnez **Tous les utilisateurs**, puis **+ Nouvel utilisateur**.
3.  Renseignez un nom et un nom d’utilisateur pour cet utilisateur. Il s’agit de votre administrateur général pour le locataire. Vous devez également définir le **rôle d’annuaire** sur **Administrateur général**. Vous pouvez également afficher le mot de passe temporaire. Lorsque vous avez terminé, sélectionnez **Créer**.</br>
![Créer](media/tutorial-single-forest/administrator2.png)</br>
4. Une fois cette opération terminée, ouvrez une nouvelle fenêtre de navigateur web et connectez-vous à myapps.microsoft.com en utilisant le nouveau compte d’administrateur général et le mot de passe temporaire.
5. Remplacez le mot de passe de l’administrateur général par quelque chose de facile à retenir.

## <a name="optional--additional-server-and-forest"></a>Facultatif :  Forêt et serveur supplémentaires
Vous trouverez ci-dessous une section facultative qui fournit les étapes de la création d’un serveur supplémentaire, et d’une forêt le cas échéant.  Elle peut être utilisée dans certains tutoriels plus avancés, tels que [Piloter Azure AD Connect pour le provisionnement cloud](tutorial-pilot-aadc-aadccp.md).

Si vous avez uniquement besoin d’un serveur supplémentaire, vous pouvez arrêter après l’étape **Créer la machine virtuelle** et joindre le serveur au domaine existant qui a été créé précédemment.  

### <a name="create-a-virtual-machine"></a>Création d'une machine virtuelle

1. Ouvrez PowerShell ISE en tant qu’administrateur.
2. Exécutez le script suivant.

    ```powershell
    # Filename:    1_CreateVM_CP.ps1
    # Description: Creates a VM to be used in the tutorial.
    #
    # DISCLAIMER:
    # Copyright (c) Microsoft Corporation. All rights reserved. #This script is made available to you without any express, implied or statutory warranty, not even the implied warranty of merchantability or fitness for a particular purpose, or the warranty of title or non-infringement. The entire risk of the use or the results from the use of this script remains with you.
    #
    #
    #
    #
    #Declare variables
    $VMName = 'CP1'
    $Switch = 'External'
    $InstallMedia = 'D:\ISO\en_windows_server_2016_updated_feb_2018_x64_dvd_11636692.iso'
    $Path = 'D:\VM'
    $VHDPath = 'D:\VM\CP1\CP1.vhdx'
    $VHDSize = '64424509440'

    #Create New Virtual Machine
    New-VM -Name $VMName -MemoryStartupBytes 16GB -BootDevice VHD -Path $Path -NewVHDPath $VHDPath -NewVHDSizeBytes $VHDSize  -Generation 2 -Switch $Switch  

    #Set the memory to be non-dynamic
    Set-VMMemory $VMName -DynamicMemoryEnabled $false

    #Add DVD Drive to Virtual Machine
    Add-VMDvdDrive -VMName $VMName -ControllerNumber 0 -ControllerLocation 1 -Path $InstallMedia

    #Mount Installation Media
    $DVDDrive = Get-VMDvdDrive -VMName $VMName

    #Configure Virtual Machine to Boot from DVD
    Set-VMFirmware -VMName $VMName -FirstBootDevice $DVDDrive
    ```

### <a name="complete-the-operating-system-deployment"></a>Terminer le déploiement du système d’exploitation
Pour terminer la création de la machine virtuelle, vous devez terminer l’installation du système d’exploitation.

1. Gestionnaire Hyper-V, double-clic sur la machine virtuelle
2. Cliquez sur le bouton Démarrer.
3. Vous êtes invité à appuyer sur n’importe quelle touche pour démarrer à partir du CD ou DVD. Faites-le.
4. Sur l’écran de démarrage Windows Server, sélectionnez votre langue et cliquez sur **Suivant**.
5. Cliquez sur **Installer maintenant**.
6. Saisissez votre clé de licence et cliquez sur **Suivant**.
7. Acceptez les termes du contrat de licence et cliquez sur **Suivant**.
8. Sélectionnez **Personnalisé :  installer Windows uniquement (avancé)**
9. Cliquez sur **Suivant**.
10. Une fois l’installation terminée, redémarrez la machine virtuelle, connectez-vous et exécutez les mises à jour Windows pour vous assurer que la machine virtuelle est à jour.  Installez les dernières mises à jour.

### <a name="install-active-directory-prerequisites"></a>Installer les prérequis pour Active Directory
Maintenant que la machine virtuelle est en cours d’exécution,vous devez effectuer quelques opérations avant d’installer Active Directory.  Autrement dit, vous devez renommer la machine virtuelle, définir une adresse IP statique et des informations DNS, et installer les outils d’administration de serveur distant.   Effectuez les actions suivantes :

1. Ouvrez PowerShell ISE en tant qu’administrateur.
2. Exécutez le script suivant.

    ```powershell
    # Filename:    2_ADPrep_CP.ps1
    # Description: Prepares your environment for Active Directory.  This is part of
    #              the Azure AD Connect password hash sync tutorial.
    #
    # DISCLAIMER:
    # Copyright (c) Microsoft Corporation. All rights reserved. This 
    # script is made available to you without any express, implied or 
    # statutory warranty, not even the implied warranty of 
    # merchantability or fitness for a particular purpose, or the 
    # warranty of title or non-infringement. The entire risk of the 
    # use or the results from the use of this script remains with you.
    #
    #
    #
    #
    #Declare variables
    $ipaddress = "10.0.1.118" 
    $ipprefix = "24" 
    $ipgw = "10.0.1.1" 
    $ipdns = "10.0.1.118"
    $ipdns2 = "8.8.8.8" 
    $ipif = (Get-NetAdapter).ifIndex 
    $featureLogPath = "c:\poshlog\featurelog.txt" 
    $newname = "CP1"
    $addsTools = "RSAT-AD-Tools" 

    #Set static IP address
    New-NetIPAddress -IPAddress $ipaddress -PrefixLength $ipprefix -InterfaceIndex $ipif -DefaultGateway $ipgw 

    #Set the DNS servers
    Set-DnsClientServerAddress -InterfaceIndex $ipif -ServerAddresses ($ipdns, $ipdns2)

    #Rename the computer 
    Rename-Computer -NewName $newname -force 

    #Install features 
    New-Item $featureLogPath -ItemType file -Force 
    Add-WindowsFeature $addsTools 
    Get-WindowsFeature | Where installed >>$featureLogPath 

    #Restart the computer 
    Restart-Computer
    ```
### <a name="create-a-windows-server-ad-environment"></a>Créer un environnement Windows Server AD
Maintenant que la machine virtuelle a été créée et renommée, et qu’elle dispose d’une adresse IP statique, vous pouvez installer et configurer Active Directory Domain Services.  Effectuez les actions suivantes :

1. Ouvrez PowerShell ISE en tant qu’administrateur.
2. Exécutez le script suivant.

    ```powershell
    # Filename:    3_InstallAD_CP.ps1
    # Description: Creates an on-premises AD environment.  This is part of
    #              the Azure AD Connect password hash sync tutorial.
    #
    # DISCLAIMER:
    # Copyright (c) Microsoft Corporation. All rights reserved. This 
    # script is made available to you without any express, implied or 
    # statutory warranty, not even the implied warranty of 
    # merchantability or fitness for a particular purpose, or the 
    # warranty of title or non-infringement. The entire risk of the 
    # use or the results from the use of this script remains with you.
    #
    #
    #
    #
    #Declare variables
    $DatabasePath = "c:\windows\NTDS"
    $DomainMode = "WinThreshold"
    $DomainName = "fabrikam.com"
    $DomaninNetBIOSName = "FABRIKAM"
    $ForestMode = "WinThreshold"
    $LogPath = "c:\windows\NTDS"
    $SysVolPath = "c:\windows\SYSVOL"
    $featureLogPath = "c:\poshlog\featurelog.txt" 
    $Password = "Pass1w0rd"
    $SecureString = ConvertTo-SecureString $Password -AsPlainText -Force

    #Install AD DS, DNS and GPMC 
    start-job -Name addFeature -ScriptBlock { 
    Add-WindowsFeature -Name "ad-domain-services" -IncludeAllSubFeature -IncludeManagementTools 
    Add-WindowsFeature -Name "dns" -IncludeAllSubFeature -IncludeManagementTools 
    Add-WindowsFeature -Name "gpmc" -IncludeAllSubFeature -IncludeManagementTools } 
    Wait-Job -Name addFeature 
    Get-WindowsFeature | Where installed >>$featureLogPath

    #Create New AD Forest
    Install-ADDSForest -CreateDnsDelegation:$false -DatabasePath $DatabasePath -DomainMode $DomainMode -DomainName $DomainName -SafeModeAdministratorPassword $SecureString -DomainNetbiosName $DomainNetBIOSName -ForestMode $ForestMode -InstallDns:$true -LogPath $LogPath -NoRebootOnCompletion:$false -SysvolPath $SysVolPath -Force:$true
    ```

### <a name="create-a-windows-server-ad-user"></a>Créer un utilisateur Windows Server AD
Maintenant que vous avez un environnement Active Directory, vous avez besoin d’un compte de test.  Ce compte sera créé dans l’environnement Active Directory local, puis synchronisé avec Azure AD.  Effectuez les actions suivantes :

1. Ouvrez PowerShell ISE en tant qu’administrateur.
2. Exécutez le script suivant.

    ```powershell 
    # Filename:    4_CreateUser_CP.ps1
    # Description: Creates a user in Active Directory.  This is part of
    #              the Azure AD Connect password hash sync tutorial.
    #
    # DISCLAIMER:
    # Copyright (c) Microsoft Corporation. All rights reserved. This 
    # script is made available to you without any express, implied or 
    # statutory warranty, not even the implied warranty of 
    # merchantability or fitness for a particular purpose, or the 
    # warranty of title or non-infringement. The entire risk of the 
    # use or the results from the use of this script remains with you.
    #
    #
    #
    #
    #Declare variables
    $Givenname = "Anna"
    $Surname = "Ringdal"
    $Displayname = "Anna Ringdal"
    $Name = "aringdal"
    $Password = "Pass1w0rd"
    $Identity = "CN=aringdal,CN=Users,DC=fabrikam,DC=com"
    $SecureString = ConvertTo-SecureString $Password -AsPlainText -Force


    #Create the user
    New-ADUser -Name $Name -GivenName $Givenname -Surname $Surname -DisplayName $Displayname -AccountPassword $SecureString

    #Set the password to never expire
    Set-ADUser -Identity $Identity -PasswordNeverExpires $true -ChangePasswordAtLogon $false -Enabled $true
    ```

## <a name="conclusion"></a>Conclusion
Vous disposez maintenant d’un environnement qui peut être utilisé pour les tutoriels existants, et servir à tester les fonctionnalités supplémentaires fournies par le provisionnement cloud.

## <a name="next-steps"></a>Étapes suivantes 

- [Présentation du provisionnement](what-is-provisioning.md)
- [Présentation du provisionnement cloud Azure AD Connect](what-is-cloud-provisioning.md)