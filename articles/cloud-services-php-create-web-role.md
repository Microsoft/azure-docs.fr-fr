---
title: Créer des rôles web et des rôles de travail Azure pour PHP
description: Guide pour la création de rôles web et de travail PHP dans un service cloud Azure et pour la configuration du runtime PHP.
services: ''
documentationcenter: php
author: msangapu-msft
manager: gwallace
ms.assetid: 9f7ccda0-bd96-4f7b-a7af-fb279a9e975b
ms.service: cloud-services
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: PHP
ms.topic: article
ms.date: 04/11/2018
ms.author: msangapu
ms.openlocfilehash: 0b02662658b2199a13f117ad95c6402f9b2974f9
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95519844"
---
# <a name="create-php-web-and-worker-roles"></a>Créer des rôles web et de travail PHP

## <a name="overview"></a>Vue d’ensemble

Ce guide vous montre comment créer des rôles web et de travail PHP dans un environnement de développement Windows, choisir une version spécifique de PHP à partir des versions « intégrées » disponibles, modifier la configuration de PHP, activer les extensions et effectuer un déploiement dans Azure. Il décrit également la façon de configurer un rôle web et de travail afin d'utiliser un runtime PHP (avec une configuration et des extensions personnalisées) que vous fournissez.

Azure propose trois modèles de calcul pour l’exécution d’applications : Azure App Service, Azure Virtual Machines et Azure Cloud Services. Ils prennent tous les trois en charge PHP. Cloud Services, qui inclut les rôles web et de travail, offre une *plateforme PaaS (platform as a service)* . Dans un service cloud, un rôle web fournit un serveur web IIS (Internet Information Services) dédié pour héberger les applications web frontales. Un rôle de travail peut exécuter des tâches asynchrones, sur le long terme ou perpétuelles, indépendamment des entrées ou des interactions de l’utilisateur.

Pour plus d’informations, consultez [Calcul des options d’hébergement fournies par Azure](cloud-services/cloud-services-choose-me.md).

## <a name="download-the-azure-sdk-for-php"></a>Téléchargement du Kit de développement logiciel (SDK) Azure pour PHP

Le [Kit de développement logiciel (SDK) Azure pour PHP](https://github.com/Azure/azure-sdk-for-php) est constitué de plusieurs composants. Cet article fait référence à deux d’entre eux : Azure PowerShell et les émulateurs Azure. Ceux-ci peuvent être installés par le biais de Microsoft Web Platform Installer : Pour plus d’informations, consultez [Installer et configurer Azure PowerShell](/powershell/azure/).

## <a name="create-a-cloud-services-project"></a>Création d'un projet Cloud Services

La première étape de la création d’un rôle web ou de travail PHP consiste à créer un projet de service Azure. Celui-ci fait office de conteneur logique pour les rôles web et de travail, et contient les fichiers de [définition de service (.csdef)] et de [configuration de service (.cscfg)] du projet.

Pour créer un projet de service Azure, exécutez Azure PowerShell en tant qu’administrateur et exécutez la commande suivante :

```powershell
PS C:\>New-AzureServiceProject myProject
```

Cette commande crée un répertoire (`myProject`) dans lequel vous pouvez ajouter des rôles web et de travail.

## <a name="add-php-web-or-worker-roles"></a>Ajout de rôles web et de travail PHP

Pour ajouter un rôle web PHP à un projet, exécutez la commande suivante à partir du répertoire racine du projet :

```powershell
PS C:\myProject> Add-AzurePHPWebRole roleName
```

Pour un rôle de travail, utilisez la commande suivante :

```powershell
PS C:\myProject> Add-AzurePHPWorkerRole roleName
```

> [!NOTE]
> Le paramètre `roleName` est facultatif. S'il est omis, le nom du rôle est généré automatiquement. Le premier rôle web créé est `WebRole1`, le second `WebRole2`, et ainsi de suite. Le premier rôle de travail créé est `WorkerRole1`, le second `WorkerRole2`, et ainsi de suite.
>
>

## <a name="use-your-own-php-runtime"></a>Utilisation de votre propre runtime PHP

Dans certains cas, au lieu de sélectionner un runtime PHP intégré et de le configurer comme indiqué ci-dessus, vous pouvez fournir votre propre runtime PHP. Par exemple, vous pouvez utiliser le même runtime PHP dans un rôle web ou de travail que vous utilisez dans votre environnement de développement, ce qui permet de s’assurer que l’application ne change pas de comportement dans votre environnement de production.

### <a name="configure-a-web-role-to-use-your-own-php-runtime"></a>Configurer un rôle web pour utiliser votre propre runtime PHP

Pour configurer un rôle web pour utiliser un runtime PHP que vous fournissez, procédez comme suit :

1. Créez un projet de service Azure et ajoutez un rôle web PHP comme indiqué précédemment dans cette rubrique.
2. Créez un dossier `php` dans le dossier `bin` se trouvant dans le répertoire racine de votre rôle web, puis ajoutez votre runtime PHP (tous les fichiers binaires, fichiers de configuration, sous-dossiers, etc.) dans le dossier `php`.
3. (FACULTATIF) Si votre runtime PHP utilise les [pilotes Microsoft SQL Server pour PHP][sqlsrv drivers], vous devez configurer votre rôle web pour installer [SQL Server Native Client 2012][sql native client], une fois celui-ci provisionné. Pour cela, ajoutez le [programme d’installation x64 sqlncli.msi] dans le dossier `bin` du répertoire racine de votre rôle web. Le script de démarrage décrit à l'étape suivante exécutera en silence le programme d'installation lorsque le rôle sera mis en service. Si votre runtime PHP n'utilise pas les pilotes Microsoft SQL Server pour PHP, vous pouvez supprimer la ligne suivante dans le script figurant à l'étape suivante :

   ```console
   msiexec /i sqlncli.msi /qn IACCEPTSQLNCLILICENSETERMS=YES
   ```

4. Définissez une tâche de démarrage configurant [Internet Information Services (IIS)][iis.net] afin d’utiliser votre runtime PHP pour gérer les requêtes de pages `.php`. Pour cela, ouvrez le fichier `setup_web.cmd` (dans le fichier `bin` du répertoire racine de votre rôle web) dans un éditeur de texte et remplacez son contenu par le script suivant :

    ```cmd
    @ECHO ON
    cd "%~dp0"

    if "%EMULATED%"=="true" exit /b 0

    msiexec /i sqlncli.msi /qn IACCEPTSQLNCLILICENSETERMS=YES

    SET PHP_FULL_PATH=%~dp0php\php-cgi.exe
    SET NEW_PATH=%PATH%;%RoleRoot%\base\x86

    %WINDIR%\system32\inetsrv\appcmd.exe set config -section:system.webServer/fastCgi /+"[fullPath='%PHP_FULL_PATH%',maxInstances='12',idleTimeout='60000',activityTimeout='3600',requestTimeout='60000',instanceMaxRequests='10000',protocol='NamedPipe',flushNamedPipe='False']" /commit:apphost
    %WINDIR%\system32\inetsrv\appcmd.exe set config -section:system.webServer/fastCgi /+"[fullPath='%PHP_FULL_PATH%'].environmentVariables.[name='PATH',value='%NEW_PATH%']" /commit:apphost
    %WINDIR%\system32\inetsrv\appcmd.exe set config -section:system.webServer/fastCgi /+"[fullPath='%PHP_FULL_PATH%'].environmentVariables.[name='PHP_FCGI_MAX_REQUESTS',value='10000']" /commit:apphost
    %WINDIR%\system32\inetsrv\appcmd.exe set config -section:system.webServer/handlers /+"[name='PHP',path='*.php',verb='GET,HEAD,POST',modules='FastCgiModule',scriptProcessor='%PHP_FULL_PATH%',resourceType='Either',requireAccess='Script']" /commit:apphost
    %WINDIR%\system32\inetsrv\appcmd.exe set config -section:system.webServer/fastCgi /"[fullPath='%PHP_FULL_PATH%'].queueLength:50000"
    ```
5. Ajoutez vos fichiers d'application dans le répertoire racine de votre rôle web. Il s'agira du répertoire racine du serveur web.
6. Publiez votre application en procédant comme indiqué dans la section [Publication de votre application](#publish-your-application) ci-dessous.

> [!NOTE]
> Il est possible de supprimer le script `download.ps1` (dans le dossier `bin` du répertoire racine du rôle web) une fois que vous avez suivi les étapes ci-dessus permettant d’utiliser votre propre runtime PHP.
>
>

### <a name="configure-a-worker-role-to-use-your-own-php-runtime"></a>Configurer un rôle de travail pour utiliser votre propre runtime PHP

Pour configurer un rôle de travail pour utiliser un runtime PHP que vous fournissez, procédez comme suit :

1. Créez un projet de service Azure et ajoutez un rôle de travail PHP comme indiqué précédemment dans cette rubrique.
2. Créez un dossier `php` dans le répertoire racine du rôle de travail, puis ajoutez votre runtime PHP (tous les fichiers binaires, fichiers de configuration, sous-dossiers, etc.) dans le dossier `php`.
3. (FACULTATIF) Si votre runtime PHP utilise les [pilotes Microsoft SQL Server pour PHP][sqlsrv drivers], une fois le rôle de travail mis en service, il vous faudra le configurer pour installer [SQL Server Native Client 2012][sql native client] une fois celui-ci provisionné. Pour cela, ajoutez le [programme d’installation x64 sqlncli.msi] dans le répertoire racine du rôle de travail. Le script de démarrage décrit à l'étape suivante exécutera en silence le programme d'installation lorsque le rôle sera mis en service. Si votre runtime PHP n'utilise pas les pilotes Microsoft SQL Server pour PHP, vous pouvez supprimer la ligne suivante dans le script figurant à l'étape suivante :

   ```console
   msiexec /i sqlncli.msi /qn IACCEPTSQLNCLILICENSETERMS=YES
   ```

4. Définissez une tâche de démarrage ajoutant votre exécutable `php.exe` à l’environnement PATH du rôle de travail une fois ce dernier mis en service. Pour cela, ouvrez le fichier `setup_worker.cmd` (dans le répertoire racine de votre rôle de travail) dans un éditeur de texte et remplacez son contenu par le script suivant :

    ```cmd
    @echo on

    cd "%~dp0"

    echo Granting permissions for Network Service to the web root directory...
    icacls ..\ /grant "Network Service":(OI)(CI)W
    if %ERRORLEVEL% neq 0 goto error
    echo OK

    if "%EMULATED%"=="true" exit /b 0

    msiexec /i sqlncli.msi /qn IACCEPTSQLNCLILICENSETERMS=YES

    setx Path "%PATH%;%~dp0php" /M

    if %ERRORLEVEL% neq 0 goto error

    echo SUCCESS
    exit /b 0

    :error

    echo FAILED
    exit /b -1
    ```
5. Ajoutez vos fichiers d'application dans le répertoire racine de votre rôle de travail.
6. Publiez votre application en procédant comme indiqué dans la section [Publication de votre application](#publish-your-application) ci-dessous.

## <a name="run-your-application-in-the-compute-and-storage-emulators"></a>Exécuter votre application dans les émulateurs de calcul et de stockage

Les émulateurs Azure fournissent un environnement local dans lequel vous pouvez tester votre application Azure avant de la déployer sur le cloud. Il y a quelques différences entre les émulateurs et l'environnement Azure. Pour en savoir plus, consultez [Utiliser l'Émulateur de stockage Azure à des fins de développement et de tests](storage/common/storage-use-emulator.md).

Notez que PHP doit être installé en local pour pouvoir utiliser l’émulateur de calcul. Ce dernier utilise votre installation locale de PHP pour exécuter votre application.

Pour exécuter votre projet dans les émulateurs, exécutez la commande suivante à partir du répertoire racine de votre projet :

```powershell
PS C:\MyProject> Start-AzureEmulator
```

Vous verrez une sortie semblable à ceci :

```output
Creating local package...
Starting Emulator...
Role is running at http://127.0.0.1:81
Started
```

Pour voir votre application exécutée dans l’émulateur, ouvrez un navigateur web et accédez à l’adresse locale illustrée dans le résultat (`http://127.0.0.1:81` dans l’exemple ci-dessus).

Pour arrêter les émulateurs, exécutez la commande suivante :

```powershell
PS C:\MyProject> Stop-AzureEmulator
```

## <a name="publish-your-application"></a>Publication de votre application

Pour publier votre application, vous devez d’abord importer vos paramètres de publication en utilisant l’applet de commande [Import-AzurePublishSettingsFile](/powershell/module/servicemanagement/azure.service/import-azurepublishsettingsfile) . Vous pouvez ensuite publier votre application à l’aide de l’applet de commande [Publish-AzureServiceProject](/powershell/module/servicemanagement/azure.service/publish-azureserviceproject) . Pour plus d’informations sur la connexion, consultez [Installation et configuration d’Azure PowerShell](/powershell/azure/).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez le [Centre pour développeurs PHP](https://azure.microsoft.com/develop/php/).

[install ps and emulators]: https://go.microsoft.com/fwlink/p/?linkid=320376&clcid=0x409
[définition de service (.csdef)]: /previous-versions/azure/reference/ee758711(v=azure.100)
[configuration de service (.cscfg)]: /previous-versions/azure/reference/ee758710(v=azure.100)
[iis.net]: https://www.iis.net/
[sql native client]: /sql/sql-server/sql-server-technical-documentation
[sqlsrv drivers]: https://php.net/sqlsrv
[programme d’installation x64 sqlncli.msi]: https://go.microsoft.com/fwlink/?LinkID=239648