---
title: Attachement d’application MSIX Windows Virtual Desktop - Azure
description: Comment configurer l’attachement d’application MSIX pour Windows Virtual Desktop.
author: Heidilohr
ms.topic: how-to
ms.date: 06/16/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 3b02be8f35ff33f758aebe03c89287c51c9ffef7
ms.sourcegitcommit: d2222681e14700bdd65baef97de223fa91c22c55
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/07/2020
ms.locfileid: "91816330"
---
# <a name="set-up-msix-app-attach"></a>Configurer l’attachement d’application MSIX

> [!IMPORTANT]
> L’application MSIX est actuellement disponible en préversion publique.
> Cette préversion est fournie sans contrat de niveau de service, c’est pourquoi nous déconseillons son utilisation pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge.
> Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Cette rubrique vous guide tout au long de la configuration de l’attachement d’application MSIX dans un environnement Windows Virtual Desktop.

## <a name="requirements"></a>Spécifications

Avant de commencer, voici ce dont vous avez besoin pour configurer l’attachement d’application MSIX :

- Accès au portail Windows Insider pour obtenir la version de Windows 10 avec prise en charge des API d’attachement d’application MSIX.
- Un déploiement Windows Virtual Desktop opérationnel. Pour apprendre à déployer Windows Virtual Desktop (classique), consultez [Création d’un locataire dans Windows Virtual Desktop](./virtual-desktop-fall-2019/tenant-setup-azure-active-directory.md). Pour apprendre à déployer Windows Virtual Desktop avec intégration Azure Resource Manager, consultez [Création d’un pool d’hôtes avec le Portail Azure](./create-host-pools-azure-marketplace.md).
- Outil de d’empaquetage MSIX.
- Partage réseau dans votre déploiement Windows Virtual Desktop où le package MSIX sera stocké.

## <a name="get-the-os-image"></a>Obtenir l’image du système d’exploitation

Tout d’abord, vous devez obtenir l’image du système d’exploitation. Vous pouvez obtenir l’image du système d’exploitation via le Portail Azure. Toutefois, si vous êtes membre du programme Windows Insider, vous avez la possibilité d’utiliser le portail Windows Insider à la place.

### <a name="get-the-os-image-from-the-azure-portal"></a>Obtenir l’image du système d’exploitation à partir du portail Azure

Pour obtenir l’image du système d’exploitation à partir du portail Azure :

1. Ouvrez le [portail Azure](https://portal.azure.com) et connectez-vous.

2. Accédez à **Créer une machine virtuelle**.

3. Dans l’onglet **De base**, sélectionnez **Windows 10 Entreprise multisession, version 2004**.

4. Suivez les autres instructions pour terminer la création de la machine virtuelle.

     >[!NOTE]
     >Vous pouvez utiliser cette machine virtuelle pour tester directement l’attachement de l’application MSIX. Pour en savoir plus, consultez directement [Générer un package VHD ou VHDX pour MSIX](#generate-a-vhd-or-vhdx-package-for-msix). Sinon, poursuivez la lecture de cette section.

### <a name="get-the-os-image-from-the-windows-insider-portal"></a>Obtenir l’image du système d’exploitation à partir du portail Windows Insider

Pour récupérer l’image du système d’exploitation à partir du portail Windows Insider :

1. Ouvrez le [portail Windows Insider](https://www.microsoft.com/software-download/windowsinsiderpreviewadvanced?wa=wsignin1.0) et connectez-vous.

     >[!NOTE]
     >Vous devez être membre du programme Windows Insider pour accéder au portail Windows Insider. Pour en savoir plus sur le programme Windows Insider, consultez notre [Documentation sur Windows Insider](/windows-insider/at-home/).

2. Faites défiler jusqu’à la section **Sélectionner l’édition** et sélectionnez **Windows 10 Insider Preview Enterprise (FAST) – Build 19041** ou version ultérieure.

3. Sélectionnez **Confirmer**, sélectionnez la langue que vous souhaitez utiliser, puis sélectionnez **Confirmer**.

     >[!NOTE]
     >À l’heure actuelle, l’anglais est la seule langue qui a été testée avec la fonctionnalité. Vous pouvez sélectionner d’autres langues, mais elles risquent de ne pas s’afficher comme prévu.

4. Une fois le lien de téléchargement généré, sélectionnez le **Téléchargement 64 bits** et enregistrez le fichier sur votre disque dur local.

## <a name="prepare-the-vhd-image-for-azure"></a>Préparer l’image de disque dur virtuel pour Azure

Ensuite, vous devez créer une image de disque dur virtuel maître. Si vous n’avez pas encore créé votre image de disque dur virtuel principale, accédez à [Préparer et personnaliser une image de disque dur virtuel principale](set-up-customize-master-image.md) et suivez les instructions qui s’y trouvent.

Une fois que vous avez créé votre image de disque dur virtuel principale, vous devez désactiver les mises à jour automatiques pour les applications d’attachement d’application MSIX. Pour désactiver les mises à jour automatiques, vous devez exécuter les commandes suivantes dans une invite de commandes avec élévation de privilèges :

```cmd
rem Disable Store auto update:

reg add HKLM\Software\Policies\Microsoft\WindowsStore /v AutoDownload /t REG_DWORD /d 0 /f
Schtasks /Change /Tn "\Microsoft\Windows\WindowsUpdate\Automatic app update" /Disable
Schtasks /Change /Tn "\Microsoft\Windows\WindowsUpdate\Scheduled Start" /Disable

rem Disable Content Delivery auto download apps that they want to promote to users:

reg add HKCU\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager /v PreInstalledAppsEnabled /t REG_DWORD /d 0 /f

reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\ContentDeliveryManager\Debug /v ContentDeliveryAllowedOverride /t REG_DWORD /d 0x2 /f

rem Disable Windows Update:

sc config wuauserv start=disabled
```

Une fois que vous avez désactivé les mises à jour automatiques, vous devez activer Hyper-V parce que vous allez utiliser la commande Mount-VHD pour l’indexation et la commande Dismount-VHD pour le retrait.

```powershell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```
>[!NOTE]
>Cette modification va nécessiter le redémarrage de la machine virtuelle.

Ensuite, préparez le disque dur virtuel de la machine virtuelle pour Azure et chargez le disque dur virtuel résultant sur Azure. Pour en savoir plus, consultez [Préparer et personnaliser une image VHD principale](set-up-customize-master-image.md).

Une fois que vous avez téléchargé le disque dur virtuel dans Azure, créez un pool d’ordinateurs hôtes basé sur cette nouvelle image en suivant les instructions du didacticiel [Créer un pool d’hôtes en utilisant la Place de marché Azure](create-host-pools-azure-marketplace.md).

## <a name="prepare-the-application-for-msix-app-attach"></a>Préparer l’application pour l’attachement d’application MSIX

Si vous disposez déjà d’un package MSIX, passez directement à la [Configurer l’infrastructure Windows Virtual Desktop](#configure-windows-virtual-desktop-infrastructure). Si vous souhaitez tester des applications héritées, suivez les instructions de [Créer un package MSIX à partir d’un programme d’installation de bureau](/windows/msix/packaging-tool/create-app-package-msi-vm/) pour convertir l’application héritée en package MSIX.

## <a name="generate-a-vhd-or-vhdx-package-for-msix"></a>Générer un package VHD ou VHDX pour MSIX

Les packages sont au format VHD ou VHDX pour optimiser les performances. MSIX requiert des packages VHD ou VHDX pour fonctionner correctement.

Pour générer un package VHD ou VHDX pour MSIX :

1. [Téléchargez l’outil msixmgr](https://aka.ms/msixmgr) et enregistrez le dossier .zip dans un dossier au sein d’une machine virtuelle hôte de session.

2. Décompressez le dossier .zip de l’outil msixmgr.

3. Placez le package MSIX source dans le même dossier que celui où vous avez décompressé l’outil msixmgr.

4. Exécutez l’applet de commande suivante dans PowerShell pour créer un disque dur virtuel :

    ```powershell
    New-VHD -SizeBytes <size>MB -Path c:\temp\<name>.vhd -Dynamic -Confirm:$false
    ```

    >[!NOTE]
    >Assurez-vous que la taille du disque dur virtuel est suffisamment grande pour contenir le MSIX développé.*

5. Exécutez l’applet de commande suivante pour monter le disque dur virtuel nouvellement créé :

    ```powershell
    $vhdObject = Mount-VHD c:\temp\<name>.vhd -Passthru
    ```

6. Exécutez cette applet de commande pour initialiser le disque dur virtuel :

    ```powershell
    $disk = Initialize-Disk -Passthru -Number $vhdObject.Number
    ```

7. Exécutez cette applet de commande pour créer une nouvelle partition :

    ```powershell
    $partition = New-Partition -AssignDriveLetter -UseMaximumSize -DiskNumber $disk.Number
    ```

8. Exécutez cette applet de commande pour formater la partition :

    ```powershell
    Format-Volume -FileSystem NTFS -Confirm:$false -DriveLetter $partition.DriveLetter -Force
    ```

9. Créez un dossier parent sur le disque dur virtuel monté. Cette étape est obligatoire, car l’attachement de l’application MSIX requiert un dossier parent. Vous pouvez nommer le dossier parent comme vous le souhaitez.

### <a name="expand-msix"></a>Développer MSIX

Après cela, vous devez « développer » l’image MSIX en la décompressant. Pour décompresser l’image MSIX :

1. Ouvrez une invite de commandes en tant qu’administrateur et accédez au dossier dans lequel vous avez téléchargé et décompressé l’outil msixmgr.

2. Exécutez l’applet de commande suivante pour décompresser le MSIX dans le disque dur virtuel que vous avez créé et monté dans la section précédente.

    ```powershell
    msixmgr.exe -Unpack -packagePath <package>.msix -destination "f:\<name of folder you created earlier>" -applyacls
    ```

    Le message suivant doit apparaître une fois la décompression terminée :

    `Successfully unpacked and applied ACLs for package: <package name>.msix`

    >[!NOTE]
    > Si vous utilisez des packages du Microsoft Store pour Entreprises (ou Éducation) au sein de votre réseau, ou sur des appareils qui ne sont pas connectés à Internet, vous devez obtenir les licences de package à partir du Store et les installer pour exécuter l’application avec succès. Consultez [Utiliser des packages hors connexion](#use-packages-offline).

3. Accédez au disque dur virtuel monté, puis ouvrez le dossier de l’application et vérifiez que le contenu du package est présent.

4. Démontez le disque dur virtuel.

## <a name="configure-windows-virtual-desktop-infrastructure"></a>Configurer l’infrastructure Windows Virtual Desktop

Par défaut, un package développé MSIX unique (le disque dur virtuel que vous avez créé dans la section précédente) peut être partagé entre plusieurs machines virtuelles hôtes de session, car les disques durs virtuels sont attachés en mode lecture seule.

Avant de commencer, assurez-vous que votre partage réseau répond à la configuration requise suivante :

- Le partage est compatible avec SMB.
- Les machines virtuelles qui font partie du pool hôte de session disposent d’autorisations NTFS sur le partage.

### <a name="set-up-an-msix-app-attach-share"></a>Configurer un partage d’attachement d’application MSIX

Dans votre environnement Windows Virtual Desktop, créez un partage réseau et déplacez-y le package.

>[!NOTE]
> La meilleure pratique pour créer des partages réseau MSIX consiste à configurer le partage réseau avec des autorisations NTFS en lecture seule.

## <a name="install-certificates"></a>Installer des certificats

Si votre application utilise un certificat qui n’est pas approuvé publiquement ou qui a été auto-signé, procédez comme suit pour l’installer :

1. Cliquez avec le bouton droit sur le package et sélectionnez **Propriétés**.
2. Dans la fenêtre qui s’affiche, sélectionnez l’onglet **Signatures numériques**. Il ne doit y avoir qu’un seul élément dans la liste de l’onglet, comme illustré dans l’image suivante. Sélectionnez cet élément pour le mettre en surbrillance, puis sélectionnez **Détails**.
3. Lorsque la fenêtre Détails de la signature numérique s’affiche, sélectionnez l’onglet **Général**, **Afficher le certificat**, puis **Installer le certificat**.
4. Quand le programme d’installation s’ouvre, sélectionnez **Machine locale** comme emplacement de stockage, puis sélectionnez **Suivant**.
5. Si le programme d’installation vous demande si vous souhaitez autoriser l’application à apporter des modifications à votre appareil, sélectionnez **Oui**.
6. Sélectionnez **Placer tous les certificats dans le magasin suivant**, puis sélectionnez **Parcourir**.
7. Lorsque la fenêtre Sélectionner un magasin de certificats s’affiche, sélectionnez **Personnes autorisées**, puis sélectionnez **OK**.
8. Sélectionnez **Suivant** et **Terminer**.

## <a name="prepare-powershell-scripts-for-msix-app-attach"></a>Préparer les scripts PowerShell pour l’attachement de l’application MSIX

L’attachement d’application MSIX comprend quatre phases distinctes qui doivent être exécutées dans l’ordre suivant :

1. Étape
2. Inscrire
3. Désinscrire
4. Retrait

Chaque phase crée un script PowerShell. Des exemples de scripts pour chaque phase sont disponibles [ici](https://github.com/Azure/RDS-Templates/tree/master/msix-app-attach).

### <a name="stage-powershell-script"></a>Indexer un script PowerShell

Avant de mettre à jour les scripts PowerShell, vérifiez que vous disposez du GUID de volume dans le disque dur virtuel. Pour récupérer le GUID du volume :

1.  Ouvrez le partage réseau où se trouve le disque dur virtuel à l’intérieur de la machine virtuelle où vous allez exécuter le script.

2.  Cliquez avec le bouton droit sur le disque dur virtuel, puis sélectionnez **Monter**. Le disque dur virtuel est alors monté sur une lettre de lecteur.

3.  Une fois le disque dur virtuel monté, la fenêtre de l’**Explorateur de fichiers** s’ouvre. Capturer le dossier parent et mettre à jour la variable **$parentFolder**

    >[!NOTE]
    >Si vous ne voyez pas de dossier parent, cela signifie que le MSIX n’a pas été développé correctement. Répétez la section précédente et réessayez.

4.  Ouvrez le dossier parent. S’il est correctement développé, vous verrez un dossier portant le même nom que le package. Mettez à jour la variable **$packageName** pour qu'elle corresponde au nom de ce dossier.

    Par exemple : `VSCodeUserSetup-x64-1.38.1_1.38.1.0_x64__8wekyb3d8bbwe`.

5.  Ouvrez une invite de commandes et entrez **mountvol**. Cette commande affiche une liste de volumes et leurs GUID. Copiez le GUID du volume sur lequel la lettre de lecteur correspond au lecteur sur lequel vous avez monté votre disque dur virtuel à l’étape 2.

    Par exemple, dans cet exemple de sortie pour la commande mountvol, si vous avez monté votre disque dur virtuel sur le lecteur C, vous souhaiterez copier la valeur au-dessus de `C:\` :

    ```cmd
    Possible values for VolumeName along with current mount points are:

    \\?\Volume{a12b3456-0000-0000-0000-10000000000}\
    *** NO MOUNT POINTS ***

    \\?\Volume{c78d9012-0000-0000-0000-20000000000}\
        E:\

    \\?\Volume{d34e5678-0000-0000-0000-30000000000}\
        C:\

    ```


6.  Mettez à jour la variable **$volumeGuid** avec le GUID de volume que vous venez de copier.

7. Ouvrez une invite admin PowerShell et mettez à jour le script PowerShell suivant avec les variables qui s’appliquent à votre environnement.

    ```powershell
    #MSIX app attach staging sample

    #region variables
    $vhdSrc="<path to vhd>"
    $packageName = "<package name>"
    $parentFolder = "<package parent folder>"
    $parentFolder = "\" + $parentFolder + "\"
    $volumeGuid = "<vol guid>"
    $msixJunction = "C:\temp\AppAttach\"
    #endregion

    #region mountvhd
    try
    {
          Mount-Diskimage -ImagePath $vhdSrc -NoDriveLetter -Access ReadOnly
          Write-Host ("Mounting of " + $vhdSrc + " was completed!") -BackgroundColor Green
    }
    catch
    {
          Write-Host ("Mounting of " + $vhdSrc + " has failed!") -BackgroundColor Red
    }
    #endregion

    #region makelink
    $msixDest = "\\?\Volume{" + $volumeGuid + "}\"
    if (!(Test-Path $msixJunction))
    {
         md $msixJunction
    }

    $msixJunction = $msixJunction + $packageName
    cmd.exe /c mklink /j $msixJunction $msixDest
    #endregion

    #region stage
    [Windows.Management.Deployment.PackageManager,Windows.Management.Deployment,ContentType=WindowsRuntime] | Out-Null
    Add-Type -AssemblyName System.Runtime.WindowsRuntime
    $asTask = ([System.WindowsRuntimeSystemExtensions].GetMethods() | Where { $_.ToString() -eq 'System.Threading.Tasks.Task`1[TResult] AsTask[TResult,TProgress](Windows.Foundation.IAsyncOperationWithProgress`2[TResult,TProgress])'})[0]
    $asTaskAsyncOperation = $asTask.MakeGenericMethod([Windows.Management.Deployment.DeploymentResult], [Windows.Management.Deployment.DeploymentProgress])
    $packageManager = [Windows.Management.Deployment.PackageManager]::new()
    $path = $msixJunction + $parentFolder + $packageName # needed if we do the pbisigned.vhd
    $path = ([System.Uri]$path).AbsoluteUri
    $asyncOperation = $packageManager.StagePackageAsync($path, $null, "StageInPlace")
    $task = $asTaskAsyncOperation.Invoke($null, @($asyncOperation))
    $task
    #endregion
    ```

### <a name="register-powershell-script"></a>Inscrire un script PowerShell

Pour exécuter le script d’inscription, exécutez les applets de commande PowerShell suivantes avec les valeurs d’espace réservé remplacées par les valeurs qui s’appliquent à votre environnement.

```powershell
#MSIX app attach registration sample

#region variables
$packageName = "<package name>"
$path = "C:\Program Files\WindowsApps\" + $packageName + "\AppxManifest.xml"
#endregion

#region register
Add-AppxPackage -Path $path -DisableDevelopmentMode -Register
#endregion
```

### <a name="deregister-powershell-script"></a>Désinscription du script PowerShell

Pour ce script, remplacez l'espace réservé de **$packageName** par le nom du package que vous testez.

```powershell
#MSIX app attach deregistration sample

#region variables
$packageName = "<package name>"
#endregion

#region deregister
Remove-AppxPackage -PreserveRoamableApplicationData $packageName
#endregion
```

### <a name="destage-powershell-script"></a>Retirer le script PowerShell

Pour ce script, remplacez l'espace réservé de **$packageName** par le nom du package que vous testez. Dans un déploiement de production, il serait préférable de l’exécuter à l’arrêt.

```powershell
#MSIX app attach de staging sample

$vhdSrc="<path to vhd>"

#region variables
$packageName = "<package name>"
$msixJunction = "C:\temp\AppAttach"
#endregion

#region deregister
Remove-AppxPackage -AllUsers -Package $packageName
Remove-Item "$msixJunction\$packageName" -Recurse -Force -Verbose
#endregion

#region Detach VHD
Dismount-DiskImage -ImagePath $vhdSrc -Confirm:$false
#endregion
```

## <a name="set-up-simulation-scripts-for-the-msix-app-attach-agent"></a>Configurer des scripts de simulation pour l’agent d’attachement de l’application MSIX

Une fois que vous avez créé les scripts, les utilisateurs peuvent les exécuter manuellement ou les configurer pour qu’ils s’exécutent automatiquement comme des scripts de démarrage, d’ouverture de session, de fermeture de session et d’arrêt. Pour en savoir plus sur ces types de scripts, consultez [Utiliser des scripts de démarrage, d’arrêt, d’ouverture de session et de fermeture de session dans une stratégie de groupe](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn789196(v=ws.11)/).

Chacun de ces scripts automatiques exécute une phase des scripts d’attachement d’application :

- Le script de démarrage exécute le script de stockage.
- Le script d’ouverture de session exécute le script d’inscription.
- Le script de fermeture de session exécute le script de désinscription.
- Le script d’arrêt exécute le script de retrait.

## <a name="use-packages-offline"></a>Utiliser des packages hors connexion

Si vous utilisez des packages du [Microsoft Store pour Entreprises](https://businessstore.microsoft.com/) ou du [Microsoft Store pour l’Éducation](https://educationstore.microsoft.com/) au sein de votre réseau ou sur des appareils qui ne sont pas connectés à Internet, vous devez obtenir les licences de package à partir du Microsoft Store et les installer sur votre appareil pour exécuter l’application avec succès. Si votre appareil est en ligne et peut se connecter au Microsoft Store pour Entreprises, les licences requises devraient être téléchargées automatiquement, mais si vous êtes hors connexion, vous devez configurer les licences manuellement.

Pour installer les fichiers de licence, vous devez utiliser un script PowerShell qui appelle la classe MDM_EnterpriseModernAppManagement_StoreLicenses02_01 dans le fournisseur de pont WMI.

Voici comment configurer les licences pour une utilisation hors connexion :

1. Téléchargez le package d’application, les licences et les frameworks requis à partir du Microsoft Store pour Entreprises. Vous avez besoin à la fois des fichiers de licence encodés et non codés. Vous trouverez des instructions de téléchargement détaillées [ici](/microsoft-store/distribute-offline-apps#download-an-offline-licensed-app).
2. Mettez à jour les variables suivantes dans le script de l’étape 3 :
      1. `$contentID` est la valeur ContentID du fichier de licence non codé (.xml). Vous pouvez ouvrir le fichier de licence dans l’éditeur de texte de votre choix.
      2. `$licenseBlob` est la chaîne entière pour l’objet Blob de licence dans le fichier de licence encodé (.bin). Vous pouvez ouvrir le fichier de licence encodé dans l’éditeur de texte de votre choix.
3. Exécutez le script suivant depuis une invite admin PowerShell ISE. Un bon emplacement pour effectuer l’installation de la licence est à la fin du [script de stockage](#stage-powershell-script) qui doit également être exécuté à partir d’une invite admin.

```powershell
$namespaceName = "root\cimv2\mdm\dmmap"
$className = "MDM_EnterpriseModernAppManagement_StoreLicenses02_01"
$methodName = "AddLicenseMethod"
$parentID = "./Vendor/MSFT/EnterpriseModernAppManagement/AppLicenses/StoreLicenses"

#TODO - Update $contentID with the ContentID value from the unencoded license file (.xml)
$contentID = "{'ContentID'_in_unencoded_license_file}"

#TODO - Update $licenseBlob with the entire String in the encoded license file (.bin)
$licenseBlob = "{Entire_String_in_encoded_license_file}"

$session = New-CimSession

#The final string passed into the AddLicenseMethod should be of the form <License Content="encoded license blob" />
$licenseString = '<License Content='+ '"' + $licenseBlob +'"' + ' />'

$params = New-Object Microsoft.Management.Infrastructure.CimMethodParametersCollection
$param = [Microsoft.Management.Infrastructure.CimMethodParameter]::Create("param",$licenseString ,"String", "In")
$params.Add($param)


try
{
   $instance = New-CimInstance -Namespace $namespaceName -ClassName $className -Property @{ParentID=$parentID;InstanceID=$contentID}
   $session.InvokeMethod($namespaceName, $instance, $methodName, $params)

}
catch [Exception]
{
     write-host $_ | out-string
}
```

## <a name="next-steps"></a>Étapes suivantes

Cette fonctionnalité n’est pas prise en charge actuellement, mais vous pouvez poser des questions à la communauté sur la [TechCommunity Windows Virtual Desktop](https://techcommunity.microsoft.com/t5/Windows-Virtual-Desktop/bd-p/WindowsVirtualDesktop).

Vous pouvez également laisser vos commentaires sur Windows Virtual Desktop sur le [Hub de commentaires Windows Virtual Desktop](https://support.microsoft.com/help/4021566/windows-10-send-feedback-to-microsoft-with-feedback-hub-app).
