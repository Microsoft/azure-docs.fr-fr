---
title: Installer des modules linguistiques sur des machines virtuelles Windows 10 dans Windows Virtual Desktop – Azure
description: Comment installer des modules linguistiques pour les machines virtuelles multisessions Windows 10 dans Windows Virtual Desktop.
author: Heidilohr
ms.topic: how-to
ms.date: 12/03/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 780324d1a6f7d9edfb552377c3e966e6a186c231
ms.sourcegitcommit: fa807e40d729bf066b9b81c76a0e8c5b1c03b536
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97347929"
---
# <a name="add-language-packs-to-a-windows-10-multi-session-image"></a>Ajouter des modules linguistiques à une image Windows 10 multisession

Windows Virtual Desktop est un service que vos utilisateurs peuvent déployer à tout moment et en tout lieu. Dès lors, il est important que vos utilisateurs puissent personnaliser la langue affichée par l’image Windows 10 Entreprise multisession.

Il existe deux façons de prendre en charge les besoins linguistiques de vos utilisateurs :

- Créer des pools d’hôtes dédiés avec une image personnalisée pour chaque langue.
- Regrouper les utilisateurs présentant des exigences différentes en termes de langue et de localisation dans le même pool d’hôtes, mais personnaliser leurs images pour leur permettre de sélectionner la langue dont ils ont besoin.

Cette dernière méthode s’avère bien plus efficace et rentable. Cela étant, il vous appartient de choisir la méthode la mieux adaptée à vos besoins. Cet article vous montre comment personnaliser les langues de vos images.

## <a name="prerequisites"></a>Prérequis

Pour personnaliser vos images Windows 10 Entreprise multisession afin d’ajouter plusieurs langues, vous avez besoin de ce qui suit :

- Une machine virtuelle Azure avec Windows 10 Entreprise multisession, version 1903 ou ultérieure

- L’ISO de langue, le disque 1 Feature on Demand (FOD) et l’ISO Inbox Apps de la version du système d’exploitation utilisée par l’image. Vous pouvez les télécharger ici : 
     
     - ISO de langue :
        - [ISO de module linguistique Windows 10, version 1903 ou 1909](https://software-download.microsoft.com/download/pr/18362.1.190318-1202.19h1_release_CLIENTLANGPACKDVD_OEM_MULTI.iso)
        - [ISO de module linguistique Windows 10, version 2004 ou 20H2](https://software-download.microsoft.com/download/pr/19041.1.191206-1406.vb_release_CLIENTLANGPACKDVD_OEM_MULTI.iso)

     - ISO Disque 1 FOD :
        - [ISO Disque 1 FOD Windows 10, version 1903 ou 1909](https://software-download.microsoft.com/download/pr/18362.1.190318-1202.19h1_release_amd64fre_FOD-PACKAGES_OEM_PT1_amd64fre_MULTI.iso)
        - [ISO Disque 1 FOD Windows 10, version 2004 ou 20H2](https://software-download.microsoft.com/download/pr/19041.1.191206-1406.vb_release_amd64fre_FOD-PACKAGES_OEM_PT1_amd64fre_MULTI.iso)
        
     - ISO Inbox Apps :
        - [ISO Inbox Apps Windows 10, version 1903 ou 1909](https://software-download.microsoft.com/download/pr/18362.1.190318-1202.19h1_release_amd64fre_InboxApps.iso)
        - [ISO Inbox Apps Windows 10, version 2004](https://software-download.microsoft.com/download/pr/19041.1.191206-1406.vb_release_amd64fre_InboxApps.iso)
        - [ISO Inbox Apps Windows 10, version 20H2](https://software-download.microsoft.com/download/pr/19041.508.200905-1327.vb_release_svc_prod1_amd64fre_InboxApps.iso)
     
     - Si vous utilisez des fichiers ISO du module d’expérience locale (LXP) pour localiser vos images, vous devez également télécharger le fichier ISO LXP approprié pour bénéficier d’une expérience de langue optimale.
        - Si vous utilisez Windows 10, version 1903 ou 1909 :
          - [Fichier ISO LXP Windows 10, version 1903 ou 1909](https://software-download.microsoft.com/download/pr/Win_10_1903_32_64_ARM64_MultiLng_LngPkAll_LXP_ONLY.iso)
        - Si vous utilisez Windows 10, version 2004 ou 20H2, utilisez les informations contenues dans [Ajout de langues dans Windows 10 : problèmes connus](/windows-hardware/manufacture/desktop/language-packs-known-issue) pour déterminer lequel des fichiers ISO LXP suivants vous convient :
          - [Fichier ISO LXP Windows 10, version 2004 ou 20H2 **9B**](https://software-download.microsoft.com/download/pr/Win_10_2004_64_ARM64_MultiLang_LangPckAll_LIP_LXP_ONLY)
          - [Fichier ISO LXP Windows 10, version 2004 ou 20H2 **9C**](https://software-download.microsoft.com/download/pr/Win_10_2004_32_64_ARM64_MultiLng_LngPkAll_LIP_9C_LXP_ONLY)
          - [Fichier ISO LXP Windows 10, version 2004 ou 20H2 **10C**](https://software-download.microsoft.com/download/pr/LanguageExperiencePack.2010C.iso)
          - [Windows 10, version 2004 ou 20H2 **11C** LXP ISO](https://software-download.microsoft.com/download/pr/LanguageExperiencePack.2011C.iso)

- Un partage Azure Files ou un partage de fichiers sur une machine virtuelle de serveur de fichiers Windows

>[!NOTE]
>Le partage de fichiers (référentiel) doit être accessible à partir de la machine virtuelle Azure que vous envisagez d’utiliser pour créer l’image personnalisée.

## <a name="create-a-content-repository-for-language-packages-and-features-on-demand"></a>Créer un référentiel de contenu pour les modules linguistiques et les fonctionnalités à la demande

Pour créer le référentiel de contenu pour les modules linguistiques, les FODs et un référentiel pour les packages Inbox Apps :

1. Sur une machine virtuelle Azure, téléchargez les images ISO Windows 10 multilingue, FOD et Inbox Apps pour Windows 10 Entreprise multisession, versions 1903/1909 et 2004, à partir des liens disponibles dans [Prérequis](#prerequisites).

2. Ouvrez et montez les fichiers ISO sur la machine virtuelle.

3. Accédez au module linguistique ISO et copiez le contenu des dossiers **LocalExperiencePacks** et **x64\\Langpacks**, puis collez le contenu dans le partage de fichiers.

4. Accédez au **fichier ISO FOD**, copiez l’intégralité de son contenu, puis collez-le dans le partage de fichiers.
5. Accédez au dossier **amd64fre** dans l’ISO Inbox Apps et copiez le contenu du référentiel pour les applications de boîte de réception que vous avez préparées.

     >[!NOTE]
     > Si vous utilisez un stockage limité, copiez uniquement les fichiers pour les langues dont vos utilisateurs ont besoin. Vous pouvez distinguer les fichiers en examinant les codes de langue dans leurs noms de fichiers. Par exemple, le nom du fichier français présente le code « fr-FR ». Pour obtenir la liste complète des codes de toutes les langues disponibles, consultez [Modules linguistiques disponibles pour Windows](/windows-hardware/manufacture/desktop/available-language-packs-for-windows).

     >[!IMPORTANT]
     > Certaines langues requièrent des polices supplémentaires incluses dans les packages satellites qui suivent différentes conventions d’affectation de noms. Par exemple, les noms de fichiers de police japonais comprennent « Jpan ».
     >
     > [!div class="mx-imgBorder"]
     > ![Exemple de modules linguistiques japonais avec la balise de langue « Jpan » dans leurs noms de fichiers.](media/language-pack-example.png)

6. Définissez les autorisations sur le partage de contenu de la langue afin d’avoir un accès en lecture à partir de la machine virtuelle que vous allez utiliser pour créer l’image personnalisée.

## <a name="create-a-custom-windows-10-enterprise-multi-session-image-manually"></a>Créer manuellement une image Windows 10 Entreprise multisession personnalisée

Pour créer manuellement une image Windows 10 Entreprise multisession personnalisée :

1. Déployez une machine virtuelle Azure, puis accédez à la galerie Azure et sélectionnez la version actuelle de Windows 10 Entreprise multisession que vous utilisez.
2. Après avoir déployé la machine virtuelle, connectez-vous à l’aide du protocole RDP en tant qu’administrateur local.
3. Assurez-vous que votre machine virtuelle dispose de toutes les mises à jour Windows les plus récentes. Si nécessaire, téléchargez les mises à jour et redémarrez la machine virtuelle.
4. Connectez-vous au référentiel de partage de fichiers de module linguistique, FOD et Inbox Apps et montez-le sur un lecteur à lettre (par exemple, le lecteur E).

## <a name="create-a-custom-windows-10-enterprise-multi-session-image-automatically"></a>Créer manuellement une image Windows 10 Entreprise multisession personnalisée

Si vous préférez installer des langues par le biais d’un processus automatisé, vous pouvez configurer un script dans PowerShell. Vous pouvez utiliser l’exemple de script suivant pour installer les modules linguistiques espagnol (Espagne), français (France) et chinois (RPC), ainsi que les packages satellites pour Windows 10 Entreprise Multisession, version 2004. Le script intègre le module d’interface linguistique et tous les packages satellites nécessaires dans l’image. Toutefois, vous pouvez également modifier ce script pour installer d’autres langues. Veillez simplement à exécuter le script à partir d’une session PowerShell avec élévation de privilèges, à défaut de quoi il ne fonctionnera pas.

```powershell
########################################################
## Add Languages to running Windows Image for Capture##
########################################################

##Disable Language Pack Cleanup##
Disable-ScheduledTask -TaskPath "\Microsoft\Windows\AppxDeploymentClient\" -TaskName "Pre-staged app cleanup"

##Set Language Pack Content Stores##
[string]$LIPContent = "E:"

##Spanish##
Add-AppProvisionedPackage -Online -PackagePath $LIPContent\es-es\LanguageExperiencePack.es-es.Neutral.appx -LicensePath $LIPContent\es-es\License.xml
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-Client-Language-Pack_x64_es-es.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-Basic-es-es-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-Handwriting-es-es-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-OCR-es-es-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-Speech-es-es-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-TextToSpeech-es-es-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-NetFx3-OnDemand-Package~31bf3856ad364e35~amd64~es-es~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-InternetExplorer-Optional-Package~31bf3856ad364e35~amd64~es-es~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-MSPaint-FoD-Package~31bf3856ad364e35~amd64~es-es~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-Notepad-FoD-Package~31bf3856ad364e35~amd64~es-es~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-PowerShell-ISE-FOD-Package~31bf3856ad364e35~amd64~es-es~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-Printing-WFS-FoD-Package~31bf3856ad364e35~amd64~es-es~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-StepsRecorder-Package~31bf3856ad364e35~amd64~es-es~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-WordPad-FoD-Package~31bf3856ad364e35~amd64~es-es~.cab
$LanguageList = Get-WinUserLanguageList
$LanguageList.Add("es-es")
Set-WinUserLanguageList $LanguageList -force

##French##
Add-AppProvisionedPackage -Online -PackagePath $LIPContent\fr-fr\LanguageExperiencePack.fr-fr.Neutral.appx -LicensePath $LIPContent\fr-fr\License.xml
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-Client-Language-Pack_x64_fr-fr.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-Basic-fr-fr-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-Handwriting-fr-fr-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-OCR-fr-fr-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-Speech-fr-fr-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-TextToSpeech-fr-fr-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-NetFx3-OnDemand-Package~31bf3856ad364e35~amd64~fr-fr~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-InternetExplorer-Optional-Package~31bf3856ad364e35~amd64~fr-FR~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-MSPaint-FoD-Package~31bf3856ad364e35~amd64~fr-FR~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-Notepad-FoD-Package~31bf3856ad364e35~amd64~fr-FR~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-PowerShell-ISE-FOD-Package~31bf3856ad364e35~amd64~fr-FR~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-Printing-WFS-FoD-Package~31bf3856ad364e35~amd64~fr-FR~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-StepsRecorder-Package~31bf3856ad364e35~amd64~fr-FR~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-WordPad-FoD-Package~31bf3856ad364e35~amd64~fr-FR~.cab
$LanguageList = Get-WinUserLanguageList
$LanguageList.Add("fr-fr")
Set-WinUserLanguageList $LanguageList -force

##Chinese(PRC)##
Add-AppProvisionedPackage -Online -PackagePath $LIPContent\zh-cn\LanguageExperiencePack.zh-cn.Neutral.appx -LicensePath $LIPContent\zh-cn\License.xml
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-Client-Language-Pack_x64_zh-cn.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-Basic-zh-cn-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-Fonts-Hans-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-Handwriting-zh-cn-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-OCR-zh-cn-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-Speech-zh-cn-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-TextToSpeech-zh-cn-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-NetFx3-OnDemand-Package~31bf3856ad364e35~amd64~zh-cn~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-InternetExplorer-Optional-Package~31bf3856ad364e35~amd64~zh-cn~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-MSPaint-FoD-Package~31bf3856ad364e35~amd64~zh-cn~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-Notepad-FoD-Package~31bf3856ad364e35~amd64~zh-cn~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-PowerShell-ISE-FOD-Package~31bf3856ad364e35~amd64~zh-cn~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-Printing-WFS-FoD-Package~31bf3856ad364e35~amd64~zh-cn~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-StepsRecorder-Package~31bf3856ad364e35~amd64~zh-cn~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-WordPad-FoD-Package~31bf3856ad364e35~amd64~zh-cn~.cab
$LanguageList = Get-WinUserLanguageList
$LanguageList.Add("zh-cn")
Set-WinUserLanguageList $LanguageList -force
```

>[!IMPORTANT]
>Les versions 1903 et 1909 de Windows 10 Entreprise ne requièrent pas le fichier de package `Microsoft-Windows-Client-Language-Pack_x64_<language-code>.cab`.

Le script peut prendre un certain temps en fonction du nombre de langues à installer.

Une fois l’exécution du script terminée, assurez-vous que les modules linguistiques sont correctement installés en accédant à **Démarrer** > **Paramètres** > **Heure et langue** > **Langue**. Si les fichiers de langue sont présents, cela signifie qu’ils sont bien installés.

Après avoir ajouté des langues supplémentaires à l’image Windows, vous devez également mettre à jour les applications de boîte de réception pour qu’elles prennent en charge les langues ajoutées. Pour ce faire, vous pouvez actualiser les applications préinstallées avec le contenu de l’ISO Inbox Apps. Pour effectuer cette actualisation dans un environnement déconnecté (aucun accès Internet possible depuis la machine virtuelle), vous pouvez utiliser l’exemple de script PowerShell suivant pour automatiser le processus.

```powershell
#########################################
## Update Inbox Apps for Multi Language##
#########################################
##Set Inbox App Package Content Stores##
[string]$InboxApps = "F:\"
##Update Inbox Store Apps##
$AllAppx = Get-Item $inboxapps\*.appx | Select-Object name
$AllAppxBundles = Get-Item $inboxapps\*.appxbundle | Select-Object name
$allAppxXML = Get-Item $inboxapps\*.xml | Select-Object name
foreach ($Appx in $AllAppx) {
    $appname = $appx.name.substring(0,$Appx.name.length-5)
    $appnamexml = $appname + ".xml"
    $pathappx = $InboxApps + "\" + $appx.Name
    $pathxml = $InboxApps + "\" + $appnamexml
    
    if($allAppxXML.name.Contains($appnamexml)){
    
    Write-Host "Handeling with xml $appname"  
  
    Add-AppxProvisionedPackage -Online -PackagePath $pathappx -LicensePath $pathxml
    } else {
      
      Write-Host "Handeling without xml $appname"
      
      Add-AppxProvisionedPackage -Online -PackagePath $pathappx -skiplicense
    }
}
foreach ($Appx in $AllAppxBundles) {
    $appname = $appx.name.substring(0,$Appx.name.length-11)
    $appnamexml = $appname + ".xml"
    $pathappx = $InboxApps + "\" + $appx.Name
    $pathxml = $InboxApps + "\" + $appnamexml
    
    if($allAppxXML.name.Contains($appnamexml)){
    Write-Host "Handeling with xml $appname"
    
    Add-AppxProvisionedPackage -Online -PackagePath $pathappx -LicensePath $pathxml
    } else {
       Write-Host "Handeling without xml $appname"
      Add-AppxProvisionedPackage -Online -PackagePath $pathappx -skiplicense
    }
}
```

>[!IMPORTANT]
>Les applications de boîte de réception incluses dans le fichier ISO ne sont pas les versions les plus récentes des applications Windows préinstallées. Pour obtenir la version la plus récente de toutes les applications, vous devez mettre à jour les applications à l’aide de l’application du Windows Store et recherchez manuellement les mises à jour après avoir installé les langues supplémentaires.

Lorsque vous avez terminé, veillez à déconnecter le partage.

## <a name="finish-customizing-your-image"></a>Finaliser la personnalisation de votre image

Après avoir installé les modules linguistiques, vous pouvez ajouter d’autres logiciels à votre image personnalisée.

Une fois votre image personnalisée, vous devez exécuter l’outil de préparation du système (sysprep).

Pour exécuter sysprep :

1. Ouvrez une invite de commandes avec élévation de privilèges et exécutez la commande suivante pour généraliser l’image :  
   
     ```cmd
     C:\Windows\System32\Sysprep\sysprep.exe /oobe /generalize /shutdown
     ```

2. Arrêtez la machine virtuelle, puis capturez-la dans une image managée en suivant les instructions fournies dans [Créer une image managée d’une machine virtuelle généralisée dans Azure](../virtual-machines/windows/capture-image-resource.md).

3. Vous pouvez maintenant utiliser l’image personnalisée pour déployer un pool d’hôtes Windows Virtual Desktop. Pour savoir comment déployer un pool d’hôtes, consultez [Tutoriel : Créer un pool d’hôtes avec le portail Azure](create-host-pools-azure-marketplace.md).

## <a name="enable-languages-in-windows-settings-app"></a>Activer les langues dans l’application Paramètres Windows

Enfin, après avoir déployer le pool d’hôtes, vous devez ajouter la langue à la liste des langues de chaque utilisateur afin qu’il puisse sélectionner la langue de son choix dans le menu Paramètres.

Pour permettre à vos utilisateurs de sélectionner les langues que vous avez installées, connectez-vous en tant qu’utilisateur, puis exécutez la cmdlet PowerShell suivante afin d’ajouter les modules linguistiques installés au menu Langues. Vous pouvez également configurer ce script en tant que tâche automatisée ou script d’ouverture qui s’active lorsque l’utilisateur se connecte à sa session.

```powershell
$LanguageList = Get-WinUserLanguageList
$LanguageList.Add("es-es")
$LanguageList.Add("fr-fr")
$LanguageList.Add("zh-cn")
Set-WinUserLanguageList $LanguageList -force
```

Après avoir modifié ses paramètres de langue, un utilisateur doit se déconnecter de sa session Windows Virtual Desktop puis se reconnecter pour que les modifications prennent effet. 

## <a name="next-steps"></a>Étapes suivantes

Si vous souhaitez connaître les problèmes connus liés aux modules linguistiques, consultez [Ajout de modules linguistiques dans Windows 10, version 1803 et versions ultérieures : problèmes connus](/windows-hardware/manufacture/desktop/language-packs-known-issue).

Si vous avez d’autres questions sur Windows 10 Entreprise multisession, consultez notre [FAQ](windows-10-multisession-faq.md).
