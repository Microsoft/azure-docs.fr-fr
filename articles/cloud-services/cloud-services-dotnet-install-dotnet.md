---
title: Installer .NET sur des rôles d’Azure Cloud Services | Microsoft Docs
description: Cet article explique comment installer manuellement .NET Framework sur vos rôles web et rôles de travail de Cloud Services.
services: cloud-services
documentationcenter: .net
author: tgore03
manager: carmonm
ms.service: cloud-services
ms.devlang: dotnet
ms.custom: devx-track-dotnet
ms.topic: article
ms.date: 06/22/2018
ms.author: tagore
ms.openlocfilehash: 6de4b79560557fc86edb9e1a25e32a6a1983ceb0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88932234"
---
# <a name="install-net-on-azure-cloud-services-roles"></a>Installer .NET sur des rôles d’Azure Cloud Services
Cet article décrit comment installer des versions de .NET Framework qui ne sont fournies avec le SE invité Azure. Vous pouvez utiliser .NET sur le SE invité pour configurer vos rôles web et rôles de travail de Cloud Services.

Par exemple, vous pouvez installer .NET Framework 4.6.2 sur la famille de SE invités 4, qui n’est fournie avec une version de .NET Framework 4.6. (La famille de SE invités 5 est fournie avec .NET Framework 4.6.) Pour obtenir les dernières informations sur les versions de SE invité Azure, consultez les [Actualités concernant les versions de SE invité Azure](cloud-services-guestos-update-matrix.md). 

>[!IMPORTANT]
>Azure SDK 2.9 contient une restriction sur le déploiement de .NET Framework 4.6 sur la famille de SE invités 4 ou version antérieure. Un correctif pour la restriction est disponible sur le site [Microsoft Docs](https://github.com/MicrosoftDocs/azure-cloud-services-files/tree/master/Azure%20Targets%20SDK%202.9).

Pour installer .NET sur vos rôles web et rôles de travail, incluez le programme d’installation web de .NET dans le cadre de votre projet de service cloud. Démarrez le programme d’installation dans le cadre des tâches de démarrage des rôles. 

## <a name="add-the-net-installer-to-your-project"></a>Ajouter le programme d'installation de .NET à votre projet
Pour télécharger le programme d'installation web de .NET Framework, sélectionnez la version à installer :

* [Programme d’installation web du .NET Framework 4.8](https://dotnet.microsoft.com/download/thank-you/net48)
* [Programme d’installation web du .NET Framework 4.7.2](https://go.microsoft.com/fwlink/?LinkId=863262)
* [Programme d’installation web du .NET Framework 4.6.2](https://www.microsoft.com/download/details.aspx?id=53345)

Pour ajouter le programme d’installation pour un rôle *web* :
  1. Dans l’**Explorateur de solutions**, dans votre projet de service cloud, sous **Rôles**, cliquez avec le bouton droit sur votre rôle *web*, puis sélectionnez **Ajouter** > **Nouveau dossier**. Créez un dossier intitulé **bin**.
  2. Cliquez avec le bouton droit sur le dossier bin, puis sélectionnez **Ajouter** > **Élément existant**. Sélectionnez le programme d'installation de .NET, puis ajoutez-le au dossier bin.
  
Pour ajouter le programme d’installation pour un rôle *de travail* :
* Cliquez avec le bouton droit sur votre rôle *de travail*, puis sélectionnez **Ajouter** > **Élément existant**. Sélectionnez le programme d'installation de .NET, puis ajoutez-le au rôle. 

Les fichiers ajoutés de cette façon dans le dossier de contenu du rôle sont automatiquement ajoutés à votre package de service cloud et déployés dans un emplacement similaire sur la machine virtuelle. Répétez ce processus pour chaque rôle web et chaque rôle de travail de service cloud afin que tous les rôles disposent d'une copie du programme d'installation.

> [!NOTE]
> Même si votre application cible .NET Framework 4.6.2, nous vous conseillons d’installer .NET Framework 4.6 sur votre rôle de service cloud. Le SE invité inclut les mises à jour [ 3098779](https://support.microsoft.com/kb/3098779) et [3097997](https://support.microsoft.com/kb/3097997) de la base de connaissances. L’installation de .NET Framework 4.6 en plus de ces mises à jour peut causer des problèmes lors de l’exécution de vos applications .NET. Pour éviter ces problèmes, installez .NET Framework 4.6.2 au lieu de la version 4.6. Pour plus d’informations, consultez les [articles sur la base de connaissances 3118750](https://support.microsoft.com/kb/3118750) et [4340191](https://support.microsoft.com/kb/4340191).
> 
> 

![Contenu du rôle avec les fichiers d'installation][1]

## <a name="define-startup-tasks-for-your-roles"></a>Définir des tâches de démarrage pour vos rôles
Vous pouvez utiliser des tâches de démarrage pour exécuter des opérations avant le démarrage d’un rôle. L’installation de .NET Framework dans le cadre de la tâche de démarrage permet de garantir qu’il sera installé avant l’exécution du code de votre application. Pour plus d’informations sur les tâches de démarrage, consultez [Exécuter des tâches de démarrage dans Azure](cloud-services-startup-tasks.md). 

1. Ajoutez le contenu suivant au fichier ServiceDefinition.csdef sous le nœud **WebRole** ou **WorkerRole** pour tous les rôles :
   
    ```xml
    <LocalResources>
      <LocalStorage name="NETFXInstall" sizeInMB="1024" cleanOnRoleRecycle="false" />
    </LocalResources>    
    <Startup>
      <Task commandLine="install.cmd" executionContext="elevated" taskType="simple">
        <Environment>
          <Variable name="PathToNETFXInstall">
            <RoleInstanceValue xpath="/RoleEnvironment/CurrentInstance/LocalResources/LocalResource[@name='NETFXInstall']/@path" />
          </Variable>
          <Variable name="ComputeEmulatorRunning">
            <RoleInstanceValue xpath="/RoleEnvironment/Deployment/@emulated" />
          </Variable>
        </Environment>
      </Task>
    </Startup>
    ```
   
    La configuration précédente exécute la commande de console `install.cmd` avec des privilèges d’administrateur pour installer .NET Framework. La configuration crée également un élément **LocalStorage** nommé **NETFXInstall**. Le script de démarrage configure le dossier temporaire pour qu’il utilise cette ressource de stockage local. 
    
    > [!IMPORTANT]
    > Afin de garantir l’installation correcte de Framework, définissez la taille de cette ressource sur 1 024 Mo au minimum.
    
    Pour plus d’informations sur les tâches de démarrage, consultez [Tâches courantes de démarrage dans Azure Cloud Services](cloud-services-startup-tasks-common.md).

2. Créez un fichier nommé **install.cmd** et ajoutez le script d’installation suivant au fichier.

   Le script vérifie si la version de .NET Framework spécifiée est déjà installée sur l'ordinateur en interrogeant le Registre. Si la version de .NET Framework n'est pas installée, le programme d'installation web de .NET Framework est lancé. Pour résoudre les éventuels problèmes, le script enregistre toutes les activités dans le fichier startuptasklog-(date et heures actuelles).txt qui est conservé dans le stockage local **InstallLogs**.
   
   > [!IMPORTANT]
   > Utilisez un éditeur de texte de base tel que le Bloc-notes de Windows pour créer le fichier install.cmd. Si vous utilisez Visual Studio pour créer un fichier texte dont vous changez ensuite l’extension à .cmd, le fichier risque de toujours contenir une marque d’ordre d’octet UTF-8 qui pourrait générer une erreur lors de l’exécution de la première ligne du script. Pour éviter cette erreur, assurez-vous que la première ligne du script est une instruction REM qui peut être ignorée par le traitement de la marque d’ordre d’octet. 
   > 
   >
   
   ```cmd
   REM Set the value of netfx to install appropriate .NET Framework. 
   REM ***** To install .NET 4.5.2 set the variable netfx to "NDP452" *****
   REM ***** To install .NET 4.6 set the variable netfx to "NDP46" *****
   REM ***** To install .NET 4.6.1 set the variable netfx to "NDP461" ***** https://go.microsoft.com/fwlink/?LinkId=671729
   REM ***** To install .NET 4.6.2 set the variable netfx to "NDP462" ***** https://www.microsoft.com/download/details.aspx?id=53345
   REM ***** To install .NET 4.7 set the variable netfx to "NDP47" ***** 
   REM ***** To install .NET 4.7.1 set the variable netfx to "NDP471" ***** https://go.microsoft.com/fwlink/?LinkId=852095
   REM ***** To install .NET 4.7.2 set the variable netfx to "NDP472" ***** https://go.microsoft.com/fwlink/?LinkId=863262
   set netfx="NDP472"
   REM ***** To install .NET 4.8 set the variable netfx to "NDP48" ***** https://dotnet.microsoft.com/download/thank-you/net48
      
   REM ***** Set script start timestamp *****
   set timehour=%time:~0,2%
   set timestamp=%date:~-4,4%%date:~-10,2%%date:~-7,2%-%timehour: =0%%time:~3,2%
   set "log=install.cmd started %timestamp%."
   
   REM ***** Exit script if running in Emulator *****
   if "%ComputeEmulatorRunning%"=="true" goto exit
   
   REM ***** Needed to correctly install .NET 4.6.1, otherwise you may see an out of disk space error *****
   set TMP=%PathToNETFXInstall%
   set TEMP=%PathToNETFXInstall%
   
   REM ***** Setup .NET filenames and registry keys *****
   if %netfx%=="NDP472" goto NDP472
   if %netfx%=="NDP471" goto NDP471
   if %netfx%=="NDP47" goto NDP47
   if %netfx%=="NDP462" goto NDP462
   if %netfx%=="NDP461" goto NDP461
   if %netfx%=="NDP46" goto NDP46
   
   set "netfxinstallfile=NDP452-KB2901954-Web.exe"
   set netfxregkey="0x5cbf5"
   goto logtimestamp
   
   :NDP46
   set "netfxinstallfile=NDP46-KB3045560-Web.exe"
   set netfxregkey="0x6004f"
   goto logtimestamp
   
   :NDP461
   set "netfxinstallfile=NDP461-KB3102438-Web.exe"
   set netfxregkey="0x6040e"
   goto logtimestamp
   
   :NDP462
   set "netfxinstallfile=NDP462-KB3151802-Web.exe"
   set netfxregkey="0x60632"
   goto logtimestamp
   
   :NDP47
   set "netfxinstallfile=NDP47-KB3186500-Web.exe"
   set netfxregkey="0x707FE"
   goto logtimestamp
   
   :NDP471
   set "netfxinstallfile=NDP471-KB4033344-Web.exe"
   set netfxregkey="0x709fc"
   goto logtimestamp
   
   :NDP472
   set "netfxinstallfile=NDP472-KB4054531-Web.exe"
   set netfxregkey="0x70BF6"
   goto logtimestamp
   
   :logtimestamp
   REM ***** Setup LogFile with timestamp *****
   md "%PathToNETFXInstall%\log"
   set startuptasklog="%PathToNETFXInstall%log\startuptasklog-%timestamp%.txt"
   set netfxinstallerlog="%PathToNETFXInstall%log\NetFXInstallerLog-%timestamp%"
   echo %log% >> %startuptasklog%
   echo Logfile generated at: %startuptasklog% >> %startuptasklog%
   echo TMP set to: %TMP% >> %startuptasklog%
   echo TEMP set to: %TEMP% >> %startuptasklog%
   
   REM ***** Check if .NET is installed *****
   echo Checking if .NET (%netfx%) is installed >> %startuptasklog%
   set /A netfxregkeydecimal=%netfxregkey%
   set foundkey=0
   FOR /F "usebackq skip=2 tokens=1,2*" %%A in (`reg query "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\NET Framework Setup\NDP\v4\Full" /v Release 2^>nul`) do @set /A foundkey=%%C
   echo Minimum required key: %netfxregkeydecimal% -- found key: %foundkey% >> %startuptasklog%
   if %foundkey% GEQ %netfxregkeydecimal% goto installed
   
   REM ***** Installing .NET *****
   echo Installing .NET with commandline: start /wait %~dp0%netfxinstallfile% /q /serialdownload /log %netfxinstallerlog%  /chainingpackage "CloudService Startup Task" >> %startuptasklog%
   start /wait %~dp0%netfxinstallfile% /q /serialdownload /log %netfxinstallerlog% /chainingpackage "CloudService Startup Task" >> %startuptasklog% 2>>&1
   if %ERRORLEVEL%== 0 goto installed
       echo .NET installer exited with code %ERRORLEVEL% >> %startuptasklog%    
       if %ERRORLEVEL%== 3010 goto restart
       if %ERRORLEVEL%== 1641 goto restart
       echo .NET (%netfx%) install failed with Error Code %ERRORLEVEL%. Further logs can be found in %netfxinstallerlog% >> %startuptasklog%
       goto exit
       
   :restart
   echo Restarting to complete .NET (%netfx%) installation >> %startuptasklog%
   shutdown.exe /r /t 5 /c "Installed .NET framework" /f /d p:2:4
   
   :installed
   echo .NET (%netfx%) is installed >> %startuptasklog%
   
   :end
   echo install.cmd completed: %date:~-4,4%%date:~-10,2%%date:~-7,2%-%timehour: =0%%time:~3,2% >> %startuptasklog%
   
   :exit
   EXIT /B 0
   ```

3. Ajoutez le fichier install.cmd à chaque rôle en sélectionnant**Ajouter** > **Élément existant** dans l’**Explorateur de solutions**, tel que décrit précédemment dans cette rubrique. 

    Une fois cette étape réalisée, tous les rôles doivent avoir le fichier du programme d'installation de .NET et le fichier install.cmd.

   ![Contenu du rôle avec tous les fichiers][2]

## <a name="configure-diagnostics-to-transfer-startup-logs-to-blob-storage"></a>Configurer les diagnostics pour transférer les journaux d’activité de démarrage vers le stockage Blob
Pour simplifier la résolution des problèmes d’installation, vous pouvez configurer Diagnostics Azure de façon à transférer tous les fichiers journaux générés par le script de démarrage ou le programme d’installation de .NET vers le stockage Blob Azure. Grâce à cette approche, vous pouvez afficher les journaux d’activité en téléchargeant les fichiers journaux depuis le stockage Blob au lieu d’accéder au rôle par le biais du Bureau à distance.


Pour configurer les diagnostics, ouvrez le fichier diagnostics.wadcfgx et ajoutez le contenu suivant sous le nœud **Directories** : 

```xml 
<DataSources>
 <DirectoryConfiguration containerName="netfx-install">
  <LocalResource name="NETFXInstall" relativePath="log"/>
 </DirectoryConfiguration>
</DataSources>
```

Ce code XML configure Diagnostics de façon à transférer tous les fichiers du répertoire log de la ressource **NETFXInstall** vers le compte de stockage de Diagnostics dans le conteneur d’objets blob **netfx-install**.

## <a name="deploy-your-cloud-service"></a>Déployer votre service cloud
Quand vous déployez votre service cloud, les tâches de démarrage installent .NET Framework, s’il n’est pas déjà installé. Vos rôles de service cloud seront à l’état *Occupé* pendant l’installation de .NET Framework et peuvent même être redémarrés si l’installation le nécessite. 

## <a name="additional-resources"></a>Ressources supplémentaires
* [Installation du .NET Framework][Installing the .NET Framework]
* [Identifier les versions de .NET Framework installées][How to: Determine Which .NET Framework Versions Are Installed]
* [Résolution des problèmes liés aux installations de .NET Framework][Troubleshooting .NET Framework Installations]

[How to: Determine Which .NET Framework Versions Are Installed]: /dotnet/framework/migration-guide/how-to-determine-which-versions-are-installed
[Installing the .NET Framework]: /dotnet/framework/install/guide-for-developers
[Troubleshooting .NET Framework Installations]: /dotnet/framework/install/troubleshoot-blocked-installations-and-uninstallations

<!--Image references-->
[1]: ./media/cloud-services-dotnet-install-dotnet/rolecontentwithinstallerfiles.png
[2]: ./media/cloud-services-dotnet-install-dotnet/rolecontentwithallfiles.png



