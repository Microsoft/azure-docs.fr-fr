---
title: Guide de configuration d’un modèle d’ordinateur Windows | Microsoft Docs
description: Étapes génériques pour préparer un modèle d’ordinateur Windows dans Lab Services.  Ces étapes comprennent le paramétrage de la planification de Windows Update, l’installation de OneDrive et l’installation d’Office.
author: EMaher
ms.topic: article
ms.date: 06/26/2020
ms.author: enewman
ms.openlocfilehash: 216dc843b31eac355e1d818014f3d70b2ef83132
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94647900"
---
# <a name="guide-to-setting-up-a-windows-template-machine-in-azure-lab-services"></a>Guide de configuration d’un modèle d’ordinateur Windows dans Azure Lab Services

Si vous configurez un modèle d’ordinateur Windows 10 pour Azure Lab Services, voici quelques meilleures pratiques et conseils à prendre en compte. Les étapes de configuration ci-dessous sont toutes facultatives.  Toutefois, ces étapes préparatoires peuvent aider vos étudiants à être plus productifs, à réduire les interruptions en classe et à s’assurer qu’ils utilisent les technologies les plus récentes.

>[!IMPORTANT]
>Cet article contient des extraits de code PowerShell permettant de simplifier le processus de modification du modèle d’ordinateur.  Pour tous les scripts PowerShell affichés, vous devrez les exécuter dans Windows PowerShell avec des privilèges administrateur. Sous Windows 10, cliquer avec le bouton droit sur le menu Démarrer et choisir « Windows PowerShell (Admin) » est un moyen rapide d’y parvenir.

## <a name="install-and-configure-onedrive"></a>Installer et configurer OneDrive

Pour empêcher la perte des données des étudiants en cas de réinitialisation d’une machine virtuelle, nous recommandons aux étudiants de sauvegarder leurs données dans le cloud.  Microsoft OneDrive peut aider les étudiants à protéger leurs données.  

### <a name="install-onedrive"></a>Installer OneDrive

Pour télécharger et installer manuellement OneDrive, consultez les pages de téléchargement [OneDrive](https://onedrive.live.com/about/download/) ou [OneDrive Entreprise](https://onedrive.live.com/about/business/).

Vous pouvez également utiliser le script PowerShell suivant.  Il télécharge et installe automatiquement la dernière version de OneDrive.  Une fois le client OneDrive installé, exécutez le programme d’installation.  Dans notre exemple, nous utilisons le commutateur `/allUsers` pour installer OneDrive pour tous les utilisateurs sur l’ordinateur. Nous utilisons également le commutateur `/silent` pour installer OneDrive en mode silencieux.

```powershell
Write-Host "Downloading OneDrive Client..."
$DownloadPath = "$env:USERPROFILE/Downloads/OneDriveSetup.exe"
if((Test-Path $DownloadPath) -eq $False )
{
    Write-Host "Downloading OneDrive..."
    $web = new-object System.Net.WebClient
    $web.DownloadFile("https://go.microsoft.com/fwlink/p/?LinkId=248256",$DownloadPath)
} else {
    Write-Host "OneDrive installer already exists at " $DownloadPath
}

Write-Host "Installing OneDrive..."
& $env:USERPROFILE/Downloads/OneDriveSetup.exe /allUsers /silent
```

### <a name="onedrive-customizations"></a>Personnalisations de OneDrive

[OneDrive peut être personnalisé de nombreuses façons](/onedrive/use-group-policy). Nous allons aborder quelques-unes des personnalisations les plus courantes.

#### <a name="silently-move-windows-known-folders-to-onedrive"></a>Déplacer silencieusement les dossiers connus de Windows vers OneDrive

Les dossiers comme Documents, Téléchargements et Images sont souvent utilisés pour stocker les fichiers des étudiants. Pour vous assurer que ces dossiers sont sauvegardés dans OneDrive, nous vous recommandons de les déplacer vers OneDrive.

Si vous utilisez un ordinateur qui n’utilise pas Active Directory, les utilisateurs peuvent déplacer manuellement ces dossiers vers OneDrive une fois qu’ils s’authentifient sur OneDrive.

1. Ouvrir l’Explorateur de fichiers
2. Cliquez avec le bouton droit sur le dossier Documents, Téléchargements ou Images.
3. Accédez à Propriétés > Emplacement.  Déplacez le dossier vers un nouveau dossier sur le répertoire OneDrive.

Si votre machine virtuelle est connectée à Active Directory, vous pouvez configurer le modèle d’ordinateur de façon à ce qu’il demande automatiquement aux étudiants de déplacer les dossiers connus vers OneDrive.  

Vous devez d’abord récupérer l’ID de votre organisation.  Pour obtenir des instructions supplémentaires, consultez [Trouver l’ID de votre organisation Microsoft 365](/onedrive/find-your-office-365-tenant-id).  Vous pouvez également obtenir l’ID de l’organisation à l’aide de la cmdlet PowerShell suivante.

```powershell
Install-Module MSOnline -Confirm
Connect-MsolService
$officeTenantID = Get-MSOLCompanyInformation |
    Select-Object -expand objectID |
    Select-Object -expand Guid
```

Une fois que vous disposez de l’ID de votre organisation, paramétrez OneDrive de sorte qu’il invite à déplacer les dossiers connus vers OneDrive à l’aide de la cmdlet PowerShell suivante.

```powershell
if ($officeTenantID -eq $null)
{
        Write-Error "Variable `$officeTenantId must be set to your Office Tenant Id before continuing."
}
New-Item -Path "HKLM:\SOFTWARE\Policies\Microsoft\OneDrive"
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\OneDrive"
    -Name "KFMSilentOptIn" -Value $officeTenantID -PropertyType STRING
```

### <a name="use-onedrive-files-on-demand"></a>Utiliser des fichiers OneDrive à la demande

Les étudiants peuvent avoir de nombreux fichiers dans leurs comptes OneDrive. Pour libérer de l’espace sur l’ordinateur et réduire le temps de téléchargement, nous vous recommandons de faire en sorte que tous les fichiers stockés dans le compte OneDrive de l’étudiant soient accessibles à la demande.  Les fichiers à la demande sont téléchargés uniquement lorsqu’un utilisateur accède au fichier.

```powershell
New-Item -Path "HKLM:\SOFTWARE\Policies\Microsoft\OneDrive" -Force
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\OneDrive"
    -Name "FilesOnDemandEnabled" -Value "00000001" -PropertyType DWORD
```

### <a name="silently-sign-in-users-to-onedrive"></a>Connexion silencieuse des utilisateurs à OneDrive

OneDrive peut être configuré pour se connecter automatiquement avec les informations d’identification Windows de l’utilisateur connecté.  La connexion automatique est utile pour les classes où l’étudiant se connecte avec ses informations d’identification scolaires.

```powershell
New-Item -Path "HKLM:\SOFTWARE\Policies\Microsoft\OneDrive"
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\OneDrive"
    -Name "SilentAccountConfig" -Value "00000001" -PropertyType DWORD
```

### <a name="disable-the-tutorial-that-appears-at-the-end-of-onedrive-setup"></a>Désactiver le tutoriel qui apparaît à la fin de l’installation de OneDrive

Ce paramètre vous permet d’empêcher le tutoriel de se lancer dans un navigateur web à la fin de l’installation de OneDrive.

```powershell
New-Item -Path "HKLM:\SOFTWARE\Policies\Microsoft\OneDrive" -Force
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\OneDrive"
    -Name "DisableTutorial" -Value "00000001" -PropertyType DWORD -Force
```

### <a name="set-the-maximum-size-of-a-file-that-to-be-download-automatically"></a>Définir la taille maximale d’un fichier à télécharger automatiquement

Ce paramètre est utilisé conjointement avec le paramètre Connexion silencieuse des utilisateurs au client de synchronisation OneDrive avec leurs informations d’identification Windows sur les appareils dont les fichiers OneDrive ne sont pas activés à la demande. Tout utilisateur dont la taille du OneDrive est supérieure au seuil spécifié (en Mo) est invité à choisir les dossiers à synchroniser avant que le client de synchronisation OneDrive (OneDrive.exe) ne télécharge les fichiers.  Dans notre exemple, « 1111-2222-3333-4444 » est l’ID de l’organisation et 0005000 définit un seuil de 5 Go.

```powershell
New-Item -Path "HKLM:\SOFTWARE\Policies\Microsoft\OneDrive"
New-Item -Path "HKLM:\SOFTWARE\Policies\Microsoft\OneDrive\DiskSpaceCheckThresholdMB"
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\OneDrive\DiskSpaceCheckThresholdMB"
    -Name "1111-2222-3333-4444" -Value "0005000" -PropertyType DWORD
```

## <a name="install-and-configure-microsoft-365"></a>Installer et configurer Microsoft 365

### <a name="install-microsoft-365"></a>Installer Microsoft 365

Si votre modèle d’ordinateur a besoin d’Office, nous vous recommandons d’installer Office par le biais de l’[outil Déploiement d’Office (ODT)](https://www.microsoft.com/download/details.aspx?id=49117). Vous devez créer un fichier config réutilisable à l’aide du [Centre d’administration Microsoft 365 Apps](https://config.office.com/) pour choisir l’architecture, les fonctionnalités dont vous aurez besoin et la fréquence de mise à jour.

1. Accédez au [Centre d’administration Microsoft 365 Apps](https://config.office.com/) et téléchargez votre propre fichier de configuration.
2. Téléchargez [l’outil Déploiement d’Office](https://www.microsoft.com/download/details.aspx?id=49117).  Le fichier téléchargé sera `setup.exe`.
3. Exécutez `setup.exe /download configuration.xml` pour télécharger les composants Office.
4. Exécutez `setup.exe /configure configuration.xml` pour installer les composants Office.

### <a name="change-the-microsoft-365-update-channel"></a>Modifier le canal de mise à jour de Microsoft 365

À l’aide de l’outil de configuration d’Office, vous pouvez définir la fréquence à laquelle Office reçoit les mises à jour. Toutefois, si vous avez besoin de modifier cette fréquence après l'installation, vous pouvez modifier l'URL du canal de mise à jour. Pour accéder aux adresses URL du canal de mise à jour, consultez [Modifier le canal de mise à jour de Microsoft 365 Apps pour les appareils de votre organisation](/deployoffice/change-update-channels). L’exemple ci-dessous montre comment configurer Microsoft 365 pour utiliser le canal de mise à jour mensuelle.

```powershell
# Update to the Microsoft 365 Monthly Channel
Set-ItemProperty
    -Path "HKLM:\SOFTWARE\Microsoft\Office\ClickToRun\Configuration\CDNBaseUrl"
    -Name "CDNBaseUrl"
    -Value "http://officecdn.microsoft.com/pr/492350f6-3a01-4f97-b9c0-c7c6ddf67d60"
```

## <a name="install-and-configure-updates"></a>Installer et configurer des mises à jour

### <a name="install-the-latest-windows-updates"></a>Installer les dernières mises à jour Windows

Nous vous recommandons d’installer les dernières mises à jour Microsoft sur le modèle d’ordinateur à des fins de sécurité avant de publier le modèle de machine virtuelle.  Cela évite également aux étudiants d’être perturbés dans leur travail lorsque les mises à jour s’exécutent à des moments inattendus.

1. Dans le menu Démarrer, lancez **Paramètres**
2. Cliquez sur **Mise à jour et sécurité**
3. Cliquez sur **Rechercher des mises à jour**
4. Les mises à jour seront téléchargées et installées.

Vous pouvez également utiliser PowerShell pour mettre à jour le modèle d’ordinateur.

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force
Install-PackageProvider -Name NuGet -MinimumVersion 2.8.5.201 -Confirm
Install-Module PSWindowsUpdate -Confirm
Install-WindowsUpdate -MicrosoftUpdate
Set-ExecutionPolicy default -Force
```

>[!NOTE]
>Certaines mises à jour peuvent nécessiter de redémarrer l’ordinateur.  Si un redémarrage est nécessaire, vous serez invité à le faire.

### <a name="install-the-latest-updates-for-microsoft-store-apps"></a>Installer les dernières mises à jour pour les applications Microsoft Store

Nous vous recommandons de mettre à jour toutes les applications Microsoft Store vers leurs versions les plus récentes.  Voici les instructions permettant de mettre à jour manuellement des applications à partir de Microsoft Store.  

1. Lancez l’application **Microsoft Store**.
2. Cliquez sur les points de suspension (…) à côté de votre photo d’utilisateur dans le coin supérieur de l’application.
3. Sélectionnez **Téléchargements et mises à jour** dans le menu déroulant.
4. Cliquez sur le bouton **Obtenir des mises à jour**.

Vous pouvez également utiliser PowerShell pour mettre à jour les applications Microsoft Store déjà installées.

```powershell
(Get-WmiObject -Namespace "root\cimv2\mdm\dmmap" -Class "MDM_EnterpriseModernAppManagement_AppManagement01").UpdateScanMethod()
```

### <a name="stop-automatic-windows-updates"></a>Arrêter les mises à jour Windows automatiques

Après la mise à jour de Windows vers la dernière version, vous pouvez envisager d’arrêter les mises à jour Windows.  Les mises à jour automatiques peuvent potentiellement interférer avec les heures de classe.  Si votre cours est long, demandez aux étudiants de vérifier manuellement les mises à jour ou de paramétrer les mises à jour automatiques à une heure en dehors des heures de cours.  Pour plus d’informations sur les options de personnalisation de Windows Update, consultez [Gérer les paramètres supplémentaires de Windows Update](/windows/deployment/update/waas-wu-settings).

Les mises à jour Windows automatiques peuvent être arrêtées à l’aide du script PowerShell suivant.

```powershell
New-Item -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\AU"
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\AU"
    -Name "NoAutoUpdate" -Value "1" -PropertyType DWORD
```

## <a name="install-foreign-language-packs"></a>Installer des modules de langue étrangère

Si vous avez besoin d’installer des langues supplémentaires sur la machine virtuelle, vous pouvez les ajouter par l’intermédiaire de Microsoft Store.

1. Lancez Microsoft Store
2. Recherchez « module linguistique »
3. Choisissez la langue à installer

Si vous êtes déjà connecté au modèle de machine virtuelle, utilisez le raccourci « Installer un module linguistique » (`ms-settings:regionlanguage?activationSource=SMC-IA-4027670`) pour accéder directement à la page appropriée des paramètres.

## <a name="remove-unneeded-built-in-apps"></a>Supprimer les applications intégrées inutiles

Windows 10 est fourni avec de nombreuses applications intégrées qui peuvent ne pas être nécessaires pour votre cours. Afin de simplifier l’image d’ordinateur pour les étudiants, vous pouvez désinstaller certaines applications de votre modèle d’ordinateur.  Pour afficher la liste des applications installées, utilisez la cmdlet `Get-AppxPackage` PowerShell.  L’exemple ci-dessous montre toutes les applications installées qui peuvent être supprimées.

```powershell
Get-AppxPackage | Where {$_.NonRemovable -eq $false} | select Name
```

Pour supprimer une application, utilisez la cmdlet Remove-Appx.  L’exemple ci-dessous montre comment supprimer tout ce qui est lié à XBox.

```powershell
Get-AppxPackage -Name *xbox* | foreach { if (-not $_.NonRemovable) { Remove-AppxPackage $_} }
```

## <a name="install-common-teaching-related-applications"></a>Installer des applications courantes liées à l’enseignement

Installez d’autres applications couramment utilisées pour l’enseignement via l’application Microsoft Store. Les suggestions incluent des applications telles que [l’application Microsoft Whiteboard](https://www.microsoft.com/store/productId/9MSPC6MP8FM4), [Microsoft Teams](https://www.microsoft.com/store/productId/9MSPC6MP8FM4) et [Minecraft Education Edition](https://education.minecraft.net/). Ces applications doivent être installées manuellement sur le modèle de machine virtuelle via Microsoft Store ou par le biais de leurs sites web respectifs.

## <a name="conclusion"></a>Conclusion

Cet article vous a montré quelques étapes facultatives pour préparer votre modèle de machine virtuelle Windows pour des cours efficaces.  Les étapes incluent l’installation de OneDrive et de Microsoft 365, ainsi que l’installation des mises à jour pour Windows et les applications Microsoft Store.  Nous avons également expliqué comment paramétrer une heure de mise à jour qui convient le mieux pour votre classe.  

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur la gestion des coûts, consultez l’article sur la façon de contrôler le comportement d’arrêt de Windows : [Guide de contrôle du comportement d’arrêt de Windows](how-to-windows-shutdown.md)