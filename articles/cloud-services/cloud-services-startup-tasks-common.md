---
title: Tâches de démarrage courantes pour les services cloud | Microsoft Docs
description: Fournit des exemples courants de tâches de démarrage que vous pouvez effectuer dans votre rôle de travail ou web de services cloud.
services: cloud-services
documentationcenter: ''
author: tgore03
ms.service: cloud-services
ms.topic: article
ms.date: 07/18/2017
ms.author: tagore
ms.openlocfilehash: 77cea7ebd333b958675438aaeb5e0e2a326a5866
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92075176"
---
# <a name="common-cloud-service-startup-tasks"></a>Tâches courantes de démarrage dans le service cloud
Cet article fournit des exemples courants de tâches de démarrage que vous pouvez effectuer dans votre service cloud. Vous pouvez utiliser des tâches de démarrage pour exécuter des opérations avant le démarrage d’un rôle. Parmi les opérations que vous pouvez effectuer figurent l’installation d’un composant, l’enregistrement de composants COM, la définition des clés du Registre ou le démarrage d’un processus de longue durée. 

Consultez [cet article](cloud-services-startup-tasks.md) pour comprendre comment fonctionnent les tâches de démarrage et, en particulier, comment créer des entrées qui définissent une tâche de démarrage.

> [!NOTE]
> Les tâches de démarrage ne s’appliquent pas aux rôles de machine virtuelle ; elles ne concernent que les rôles web de service cloud et de travail.
> 

## <a name="define-environment-variables-before-a-role-starts"></a>Définir des variables d’environnement avant le démarrage d’un rôle
Si des variables d’environnement doivent être définies pour une tâche spécifique, utilisez l’élément [Environment] à l’intérieur de l’élément [Tâche].

```xml
<ServiceDefinition name="MyService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition">
    <WorkerRole name="WorkerRole1">
        ...
        <Startup>
            <Task commandLine="Startup.cmd" executionContext="limited" taskType="simple">
                <Environment>
                    <Variable name="MyEnvironmentVariable" value="MyVariableValue" />
                </Environment>
            </Task>
        </Startup>
    </WorkerRole>
</ServiceDefinition>
```

Les variables peuvent également utiliser une [valeur Azure XPath valide](cloud-services-role-config-xpath.md) pour faire référence à des informations sur le déploiement. Au lieu d’utiliser l’attribut `value` , définissez un élément enfant [RoleInstanceValue] .

```xml
<Variable name="PathToStartupStorage">
    <RoleInstanceValue xpath="/RoleEnvironment/CurrentInstance/LocalResources/LocalResource[@name='StartupLocalStorage']/@path" />
</Variable>
```


## <a name="configure-iis-startup-with-appcmdexe"></a>Configurer le démarrage d’IIS avec AppCmd.exe
L’outil de ligne de commande [AppCmd.exe](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/jj635852(v=ws.11)) peut être utilisé pour gérer les paramètres IIS au démarrage sur Azure. *AppCmd.exe* fournit un accès en ligne de commande pratique aux paramètres de configuration à utiliser dans les tâches de démarrage sur Azure. *AppCmd.exe*permet d’ajouter, de modifier ou de supprimer des paramètres de site web pour les applications et les sites.

Toutefois, tenez compte des points suivants quand vous utilisez *AppCmd.exe* en tant que tâche de démarrage :

* Les tâches de démarrage peuvent être exécutées plusieurs fois entre les redémarrages. Par exemple, lorsqu’un rôle est recyclé.
* Si une action *AppCmd.exe* est exécutée plusieurs fois, cela peut générer une erreur. Par exemple, une double tentative d’ajout d’une section à *Web.config* peut générer une erreur.
* Les tâches de démarrage échouent si elles retournent un code de sortie non nul ou **errorlevel**. Par exemple, lorsque *AppCmd.exe* génère une erreur.

Il est recommandé de vérifier le paramètre **errorlevel** arès avoir appelé *AppCmd.exe*, opération que vous pouvez effectuez facilement si vous encapsulez l’appel à *AppCmd.exe* avec un fichier *.cmd*. Si vous détectez une réponse **errorlevel** connue, vous pouvez l’ignorer, ou bien la retransmettre.

Les codes errorlevel retournés par *AppCmd.exe* sont répertoriés dans le fichier winerror.h, et peuvent également être consultés sur [MSDN](/windows/desktop/Debug/system-error-codes--0-499-).

### <a name="example-of-managing-the-error-level"></a>Exemple de gestion de niveau d’erreur
Cet exemple ajoute une section de compression et une entrée de compression pour JSON au fichier *Web.config* , avec gestion et journalisation des erreurs.

Les sections pertinentes du fichier [ServiceDefinition.csdef] indiquées ici, avec notamment l’attribut [executionContext](/previous-versions/azure/reference/gg557552(v=azure.100)#task) défini sur `elevated` pour qu’*AppCmd.exe* soit autorisé à modifier les paramètres du fichier *Web.config* :

```xml
<ServiceDefinition name="MyService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition">
    <WorkerRole name="WorkerRole1">
        ...
        <Startup>
            <Task commandLine="Startup.cmd" executionContext="elevated" taskType="simple" />
        </Startup>
    </WorkerRole>
</ServiceDefinition>
```

Le fichier de commandes *Startup.cmd* utilise *AppCmd.exe* pour ajouter une section de compression et une entrée de compression pour JSON au fichier *Web.config*. Le code **errorlevel** attendu 183 est défini sur zéro à l’aide du programme en ligne de commande VERIFY.EXE. Les codes errorlevel inattendus sont enregistrés dans StartupErrorLog.txt.

```cmd
REM   *** Add a compression section to the Web.config file. ***
%windir%\system32\inetsrv\appcmd set config /section:urlCompression /doDynamicCompression:True /commit:apphost >> "%TEMP%\StartupLog.txt" 2>&1

REM   ERRORLEVEL 183 occurs when trying to add a section that already exists. This error is expected if this
REM   batch file were executed twice. This can occur and must be accounted for in an Azure startup
REM   task. To handle this situation, set the ERRORLEVEL to zero by using the Verify command. The Verify
REM   command will safely set the ERRORLEVEL to zero.
IF %ERRORLEVEL% EQU 183 VERIFY > NUL

REM   If the ERRORLEVEL is not zero at this point, some other error occurred.
IF %ERRORLEVEL% NEQ 0 (
    ECHO Error adding a compression section to the Web.config file. >> "%TEMP%\StartupLog.txt" 2>&1
    GOTO ErrorExit
)

REM   *** Add compression for json. ***
%windir%\system32\inetsrv\appcmd set config  -section:system.webServer/httpCompression /+"dynamicTypes.[mimeType='application/json; charset=utf-8',enabled='True']" /commit:apphost >> "%TEMP%\StartupLog.txt" 2>&1
IF %ERRORLEVEL% EQU 183 VERIFY > NUL
IF %ERRORLEVEL% NEQ 0 (
    ECHO Error adding the JSON compression type to the Web.config file. >> "%TEMP%\StartupLog.txt" 2>&1
    GOTO ErrorExit
)

REM   *** Exit batch file. ***
EXIT /b 0

REM   *** Log error and exit ***
:ErrorExit
REM   Report the date, time, and ERRORLEVEL of the error.
DATE /T >> "%TEMP%\StartupLog.txt" 2>&1
TIME /T >> "%TEMP%\StartupLog.txt" 2>&1
ECHO An error occurred during startup. ERRORLEVEL = %ERRORLEVEL% >> "%TEMP%\StartupLog.txt" 2>&1
EXIT %ERRORLEVEL%
```

## <a name="add-firewall-rules"></a>Ajouter des règles de pare-feu
Dans Azure, il existe en fait deux pare-feu. Le premier pare-feu contrôle les connexions entre la machine virtuelle et le monde extérieur. Ce pare-feu est contrôlé par l’élément [Points de terminaison] du fichier [ServiceDefinition.csdef].

Le second pare-feu contrôle les connexions entre la machine virtuelle et les processus au sein de cette dernière. Ce pare-feu peut être contrôlé par l’outil de ligne de commande `netsh advfirewall firewall`.

Azure crée des règles de pare-feu pour les processus démarrés au sein de vos rôles. Par exemple, quand vous démarrez un service ou un programme, Azure crée automatiquement les règles de pare-feu nécessaires pour permettre à ce service de communiquer avec Internet. Toutefois, si vous créez un service qui est démarré par un processus en dehors de votre rôle (par exemple, un service COM+ ou une tâche planifiée Windows), vous devez créer manuellement une règle de pare-feu pour autoriser l’accès à ce service. Ces règles de pare-feu peuvent être créées à l’aide d’une tâche de démarrage.

Une tâche de démarrage qui crée une règle de pare-feu doit avoir [executionContext][tâche] défini sur **elevated**. Ajoutez la tâche de démarrage suivante au fichier [ServiceDefinition.csdef] .

```xml
<ServiceDefinition name="MyService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition">
    <WorkerRole name="WorkerRole1">
        ...
        <Startup>
            <Task commandLine="AddFirewallRules.cmd" executionContext="elevated" taskType="simple" />
        </Startup>
    </WorkerRole>
</ServiceDefinition>
```

Pour ajouter la règle de pare-feu, vous devez utiliser les commandes `netsh advfirewall firewall` appropriées dans votre fichier de commandes de démarrage. Dans cet exemple, la tâche de démarrage nécessite les fonctionnalités de sécurité et de chiffrement pour le port TCP 80.

```cmd
REM   Add a firewall rule in a startup task.

REM   Add an inbound rule requiring security and encryption for TCP port 80 traffic.
netsh advfirewall firewall add rule name="Require Encryption for Inbound TCP/80" protocol=TCP dir=in localport=80 security=authdynenc action=allow >> "%TEMP%\StartupLog.txt" 2>&1

REM   If an error occurred, return the errorlevel.
EXIT /B %errorlevel%
```

## <a name="block-a-specific-ip-address"></a>Bloquer une adresse IP spécifique
Vous pouvez limiter un accès de rôle web Azure à un ensemble d’adresses IP spécifiques en modifiant votre fichier IIS **web.config**. Vous devez également utiliser un fichier de commande qui déverrouille la section **ipSecurity** du fichier **ApplicationHost.config**.

Pour déverrouiller la section **ipSecurity** du fichier **ApplicationHost.config**, créez un fichier de commande qui s’exécute au démarrage du rôle. Créez un dossier à la racine de votre rôle web appelé **startup** et, dans ce dossier, créez un fichier de commandes appelé **startup.cmd**. Ajoutez ce fichier à votre projet Visual Studio et définissez les propriétés sur **Toujours copier** pour vous assurer qu’il est inclus dans votre package.

Ajoutez la tâche de démarrage suivante au fichier [ServiceDefinition.csdef] .

```xml
<ServiceDefinition name="MyService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition">
    <WebRole name="WebRole1">
        ...
        <Startup>
            <Task commandLine="startup.cmd" executionContext="elevated" />
        </Startup>
    </WebRole>
</ServiceDefinition>
```

Ajoutez cette commande au fichier **startup.cmd** :

```cmd
@echo off
@echo Installing "IPv4 Address and Domain Restrictions" feature 
powershell -ExecutionPolicy Unrestricted -command "Install-WindowsFeature Web-IP-Security"
@echo Unlocking configuration for "IPv4 Address and Domain Restrictions" feature 
%windir%\system32\inetsrv\AppCmd.exe unlock config -section:system.webServer/security/ipSecurity
```

Ainsi, le fichier de commandes **startup.cmd** s’exécute chaque fois que le rôle web est initialisé et la section **ipSecurity** nécessaire est systématiquement déverrouillée.

Enfin, modifiez la [section system.webServer](https://www.iis.net/configreference/system.webserver/security/ipsecurity#005) du fichier **web.config** de votre rôle web pour ajouter une liste d’adresses IP bénéficiant d’un droit d’accès, comme illustré dans l’exemple suivant :

Cet exemple de configuration **permet** à toutes les adresses IP d’accéder au serveur, sauf aux deux adresses IP définies.

```xml
<system.webServer>
    <security>
    <!--Unlisted IP addresses are granted access-->
    <ipSecurity>
        <!--The following IP addresses are denied access-->
        <add allowed="false" ipAddress="192.168.100.1" subnetMask="255.255.0.0" />
        <add allowed="false" ipAddress="192.168.100.2" subnetMask="255.255.0.0" />
    </ipSecurity>
    </security>
</system.webServer>
```

Cet exemple de configuration **refuse** à toutes les adresses IP le droit d’accéder au serveur, sauf aux deux adresses IP définies.

```xml
<system.webServer>
    <security>
    <!--Unlisted IP addresses are denied access-->
    <ipSecurity allowUnlisted="false">
        <!--The following IP addresses are granted access-->
        <add allowed="true" ipAddress="192.168.100.1" subnetMask="255.255.0.0" />
        <add allowed="true" ipAddress="192.168.100.2" subnetMask="255.255.0.0" />
    </ipSecurity>
    </security>
</system.webServer>
```

## <a name="create-a-powershell-startup-task"></a>Créer une tâche de démarrage PowerShell
Les scripts Windows PowerShell ne peuvent pas être appelés directement à partir du fichier [ServiceDefinition.csdef] , mais ils peuvent être appelés à partir d’un fichier de commandes de démarrage.

PowerShell (par défaut) n’exécute pas de scripts non signés. Sauf si vous signez votre script, vous devez configurer Windows PowerShell pour exécuter des scripts non signés. Pour exécuter des scripts non signés, **ExecutionPolicy** doit avoir la valeur **Unrestricted**. Le paramètre **ExecutionPolicy** que vous utilisez est basé sur la version de Windows PowerShell.

```cmd
REM   Run an unsigned PowerShell script and log the output
PowerShell -ExecutionPolicy Unrestricted .\startup.ps1 >> "%TEMP%\StartupLog.txt" 2>&1

REM   If an error occurred, return the errorlevel.
EXIT /B %errorlevel%
```

Si vous utilisez un SE invité qui exécute PowerShell 2.0 ou 1.0, vous pouvez forcer la version 2 à s’exécuter et, si elle n’est pas disponible, utiliser la version 1.

```cmd
REM   Attempt to set the execution policy by using PowerShell version 2.0 syntax.
PowerShell -Version 2.0 -ExecutionPolicy Unrestricted .\startup.ps1 >> "%TEMP%\StartupLog.txt" 2>&1

REM   If PowerShell version 2.0 isn't available. Set the execution policy by using the PowerShell
IF %ERRORLEVEL% EQU -393216 (
   PowerShell -Command "Set-ExecutionPolicy Unrestricted" >> "%TEMP%\StartupLog.txt" 2>&1
   PowerShell .\startup.ps1 >> "%TEMP%\StartupLog.txt" 2>&1
)

REM   If an error occurred, return the errorlevel.
EXIT /B %errorlevel%
```

## <a name="create-files-in-local-storage-from-a-startup-task"></a>Créer des fichiers dans le stockage local à partir d’une tâche de démarrage
Vous pouvez utiliser une ressource de stockage local pour stocker les fichiers créés par votre tâche de démarrage afin de les rendre accessibles à votre application.

Pour créer la ressource de stockage local, ajoutez une section [LocalResources] au fichier [ServiceDefinition.csdef], puis ajoutez l’élément enfant [LocalStorage]. Donnez à la ressource de stockage local un nom unique et une taille appropriée pour votre tâche de démarrage.

Pour utiliser une ressource de stockage local dans votre tâche de démarrage, vous devez créer une variable d’environnement pour faire référence à l’emplacement de la ressource de stockage local. Ensuite, la tâche de démarrage et l’application sont en mesure de lire et d’écrire des fichiers dans la ressource de stockage local.

Les sections appropriées du fichier **ServiceDefinition.csdef** sont indiquées ici :

```xml
<ServiceDefinition name="MyService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition">
  <WorkerRole name="WorkerRole1">
    ...

    <LocalResources>
      <LocalStorage name="StartupLocalStorage" sizeInMB="5"/>
    </LocalResources>

    <Startup>
      <Task commandLine="Startup.cmd" executionContext="limited" taskType="simple">
        <Environment>
          <Variable name="PathToStartupStorage">
            <RoleInstanceValue xpath="/RoleEnvironment/CurrentInstance/LocalResources/LocalResource[@name='StartupLocalStorage']/@path" />
          </Variable>
        </Environment>
      </Task>
    </Startup>
  </WorkerRole>
</ServiceDefinition>
```

Par exemple, ce fichier de commandes **Startup.cmd** emploie la variable d’environnement **PathToStartupStorage** pour créer le fichier **MyTest.txt** à l’emplacement du stockage local.

```cmd
REM   Create a simple text file.

ECHO This text will go into the MyTest.txt file which will be in the    >  "%PathToStartupStorage%\MyTest.txt"
ECHO path pointed to by the PathToStartupStorage environment variable.  >> "%PathToStartupStorage%\MyTest.txt"
ECHO The contents of the PathToStartupStorage environment variable is   >> "%PathToStartupStorage%\MyTest.txt"
ECHO "%PathToStartupStorage%".                                          >> "%PathToStartupStorage%\MyTest.txt"

REM   Exit the batch file with ERRORLEVEL 0.

EXIT /b 0
```

Vous pouvez accéder au dossier de stockage local à partir du Kit de développement logiciel (SDK) Azure à l’aide de la méthode [GetLocalResource](/previous-versions/azure/reference/ee772845(v=azure.100)).

```csharp
string localStoragePath = Microsoft.WindowsAzure.ServiceRuntime.RoleEnvironment.GetLocalResource("StartupLocalStorage").RootPath;

string fileContent = System.IO.File.ReadAllText(System.IO.Path.Combine(localStoragePath, "MyTestFile.txt"));
```

## <a name="run-in-the-emulator-or-cloud"></a>Exécution dans l’émulateur ou le cloud
Si vous le souhaitez, les étapes suivies par votre tâche de démarrage peuvent différer selon qu’elle s’exécute dans le cloud ou dans l’émulateur de calcul. Par exemple, vous pouvez utiliser une nouvelle copie de vos données SQL uniquement quand l’exécution a lieu dans l’émulateur. De même, vous pouvez effectuer des opérations d’optimisation de performances pour le cloud qui sont superflues dans l’émulateur.

Pour pouvoir effectuer ainsi différentes actions sur l’émulateur de calcul et sur le cloud, vous devez créer une variable d’environnement dans le fichier [ServiceDefinition.csdef]. Vous testez ensuite une valeur de cette variable d’environnement dans votre tâche de démarrage.

Pour créer la variable d’environnement, ajoutez l’élément [Variable]/[RoleInstanceValue] et créez une valeur XPath`/RoleEnvironment/Deployment/@emulated`. La valeur de la variable d’environnement **%ComputeEmulatorRunning%** est `true` si l’exécution a lieu sur l’émulateur de calcul et `false` sur le cloud.

```xml
<ServiceDefinition name="MyService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition">
  <WorkerRole name="WorkerRole1">

    ...

    <Startup>
      <Task commandLine="Startup.cmd" executionContext="limited" taskType="simple">
        <Environment>
          <Variable name="ComputeEmulatorRunning">
            <RoleInstanceValue xpath="/RoleEnvironment/Deployment/@emulated" />
          </Variable>
        </Environment>
      </Task>
    </Startup>

  </WorkerRole>
</ServiceDefinition>
```

La tâche peut désormais utiliser la variable d’environnement **%ComputeEmulatorRunning%** pour effectuer différentes actions selon que le rôle s’exécute dans le cloud ou l’émulateur. Voici un script shell .cmd qui vérifie cette variable d’environnement.

```cmd
REM   Check if this task is running on the compute emulator.

IF "%ComputeEmulatorRunning%" == "true" (
    REM   This task is running on the compute emulator. Perform tasks that must be run only in the compute emulator.
) ELSE (
    REM   This task is running on the cloud. Perform tasks that must be run only in the cloud.
)
```


## <a name="detect-that-your-task-has-already-run"></a>Détecter que la tâche a déjà été exécutée
Si le rôle se recycle sans qu’un redémarrage ait lieu, vos tâches de démarrage sont de nouveau exécutées. Aucun indicateur ne permet de signaler qu’une tâche a déjà été exécutée sur la machine virtuelle hôte. Il est possible que certaines tâches s’exécutent plusieurs fois sans engendrer de problème. Toutefois, dans certaines situations, vous devez éviter qu’une tâche s’exécute plusieurs fois.

La façon la plus simple de détecter qu’une tâche a déjà été exécutée consiste à créer un fichier dans le dossier **%TEMP%** quand la tâche réussit et à le rechercher au démarrage de la tâche. Voici un exemple de script shell cmd qui effectue cette opération.

```cmd
REM   If Task1_Success.txt exists, then Application 1 is already installed.
IF EXIST "%PathToApp1Install%\Task1_Success.txt" (
  ECHO Application 1 is already installed. Exiting. >> "%TEMP%\StartupLog.txt" 2>&1
  GOTO Finish
)

REM   Run your real exe task
ECHO Running XYZ >> "%TEMP%\StartupLog.txt" 2>&1
"%PathToApp1Install%\setup.exe" >> "%TEMP%\StartupLog.txt" 2>&1

IF %ERRORLEVEL% EQU 0 (
  REM   The application installed without error. Create a file to indicate that the task
  REM   does not need to be run again.

  ECHO This line will create a file to indicate that Application 1 installed correctly. > "%PathToApp1Install%\Task1_Success.txt"

) ELSE (
  REM   An error occurred. Log the error and exit with the error code.

  DATE /T >> "%TEMP%\StartupLog.txt" 2>&1
  TIME /T >> "%TEMP%\StartupLog.txt" 2>&1
  ECHO  An error occurred running task 1. Errorlevel = %ERRORLEVEL%. >> "%TEMP%\StartupLog.txt" 2>&1

  EXIT %ERRORLEVEL%
)

:Finish

REM   Exit normally.
EXIT /B 0
```

## <a name="task-best-practices"></a>Meilleures pratiques pour les tâches
Voici quelques meilleures pratiques à suivre quand vous configurez la tâche pour votre rôle web ou de travail.

### <a name="always-log-startup-activities"></a>Toujours consigner les activités de démarrage
Comme Visual Studio ne fournit pas de débogueur pour parcourir les fichiers de commandes, il est préférable de récupérer autant de données que possible sur le fonctionnement des fichiers de commandes. La consignation de la sortie des fichiers de commandes, **stdout** et **stderr**, peut fournir des informations importantes au moment du débogage et de la correction des fichiers de commandes. Pour consigner **stdout** et **stderr** dans le fichier StartupLog.txt dans le répertoire indiqué par la variable d’environnement **%TEMP%** , ajoutez le texte `>>  "%TEMP%\\StartupLog.txt" 2>&1` à la fin des lignes que vous souhaitez enregistrer. Par exemple, pour exécuter setup.exe dans le répertoire **%PathToApp1Install%** : `"%PathToApp1Install%\setup.exe" >> "%TEMP%\StartupLog.txt" 2>&1`

Pour simplifier votre code xml, vous pouvez créer un fichier *cmd* wrapper qui appelle toutes vos tâches de démarrage avec la journalisation et garantit que chaque tâche enfant partage les mêmes variables d’environnement.

Il peut néanmoins s’avérer ennuyeux d’utiliser `>> "%TEMP%\StartupLog.txt" 2>&1` sur la fin de chaque tâche de démarrage. Vous pouvez appliquer la journalisation des tâches en créant un wrapper qui gère la journalisation pour vous. Ce wrapper appelle le fichier batch réel que vous souhaitez exécuter. Toutes les sorties depuis le fichier batch cible seront redirigées vers le fichier *Startuplog.txt*.

L’exemple suivant montre comment rediriger toute la sortie d’un fichier batch de démarrage. Dans cet exemple, le fichier ServerDefinition.csdef crée une tâche de démarrage qui appelle *logwrap.cmd*. *logwrap.cmd* appelle *Startup2.cmd*, en redirigeant toutes la sortie vers **%TEMP%\\StartupLog.txt**.

ServiceDefinition.cmd :

```xml
<Startup>
    <Task commandLine="logwrap.cmd startup2.cmd" executionContext="limited" taskType="simple" />
</Startup>
```

**logwrap.cmd :**

```cmd
@ECHO OFF

REM   logwrap.cmd calls passed in batch file, redirecting all output to the StartupLog.txt log file.

ECHO [%date% %time%] == START logwrap.cmd ============================================== >> "%TEMP%\StartupLog.txt" 2>&1
ECHO [%date% %time%] Running %1 >> "%TEMP%\StartupLog.txt" 2>&1

REM   Call the child command batch file, redirecting all output to the StartupLog.txt log file.
START /B /WAIT %1 >> "%TEMP%\StartupLog.txt" 2>&1

REM   Log the completion of child command.
ECHO [%date% %time%] Done >> "%TEMP%\StartupLog.txt" 2>&1

IF %ERRORLEVEL% EQU 0 (

   REM   No errors occurred. Exit logwrap.cmd normally.
   ECHO [%date% %time%] == END logwrap.cmd ================================================ >> "%TEMP%\StartupLog.txt" 2>&1
   ECHO.  >> "%TEMP%\StartupLog.txt" 2>&1
   EXIT /B 0

) ELSE (

   REM   Log the error.
   ECHO [%date% %time%] An error occurred. The ERRORLEVEL = %ERRORLEVEL%.  >> "%TEMP%\StartupLog.txt" 2>&1
   ECHO [%date% %time%] == END logwrap.cmd ================================================ >> "%TEMP%\StartupLog.txt" 2>&1
   ECHO.  >> "%TEMP%\StartupLog.txt" 2>&1
   EXIT /B %ERRORLEVEL%

)
```

**Startup2.cmd :**

```cmd
@ECHO OFF

REM   This is the batch file where the startup steps should be performed. Because of the
REM   way Startup2.cmd was called, all commands and their outputs will be stored in the
REM   StartupLog.txt file in the directory pointed to by the TEMP environment variable.

REM   If an error occurs, the following command will pass the ERRORLEVEL back to the
REM   calling batch file.

ECHO [%date% %time%] Some log information about this task
ECHO [%date% %time%] Some more log information about this task

EXIT %ERRORLEVEL%
```

Exemple de sortie dans le fichier **StartupLog.txt** :

```txt
[Mon 10/17/2016 20:24:46.75] == START logwrap.cmd ============================================== 
[Mon 10/17/2016 20:24:46.75] Running command1.cmd 
[Mon 10/17/2016 20:24:46.77] Some log information about this task
[Mon 10/17/2016 20:24:46.77] Some more log information about this task
[Mon 10/17/2016 20:24:46.77] Done 
[Mon 10/17/2016 20:24:46.77] == END logwrap.cmd ================================================ 
```

> [!TIP]
> Le fichier **StartupLog.txt** se trouve dans le dossier *C:\Resources\temp\\{role identifier}\RoleTemp*.
> 
> 

### <a name="set-executioncontext-appropriately-for-startup-tasks"></a>Définir executionContext convenablement pour les tâches de démarrage
Définissez les privilèges convenablement pour la tâche de démarrage. Parfois, les tâches de démarrage doivent s’exécuter avec des privilèges élevés même si le rôle s’exécute avec des privilèges normaux.

L’outil en ligne de commande [executionContext][tâche] définit le niveau de privilège de la tâche de démarrage. Si `executionContext="limited"` est utilisé, la tâche de démarrage a le même niveau de privilège que le rôle. Si `executionContext="elevated"` est utilisé, la tâche de démarrage a des privilèges d’administrateur, ce qui lui permet d’effectuer des tâches d’administrateur sans accorder de privilèges d’administrateur à votre rôle.

Un exemple de tâche de démarrage qui nécessite des privilèges élevés est une tâche de démarrage qui utilise **AppCmd.exe** pour configurer IIS. **AppCmd.exe** nécessite `executionContext="elevated"`.

### <a name="use-the-appropriate-tasktype"></a>Utiliser l’attribut taskType approprié
L’outil en ligne de commande [taskType][tâche] détermine la façon dont la tâche de démarrage est exécutée. Trois valeurs sont possibles : **simple**, **background** et **foreground**. Les tâches en arrière-plan et de premier plan sont lancées de manière asynchrone, puis les tâches simples sont exécutées de façon synchrone une par une.

Dans le cas des tâches de démarrage **simple**, vous pouvez définir l’ordre dans lequel les tâches s’exécutent en fonction de l’ordre dans lequel elles sont répertoriées dans le fichier ServiceDefinition.csdef. Si une tâche **simple** se termine par un code de sortie différent de zéro, la procédure de démarrage s’arrête et le rôle ne démarre pas.

La différence entre une tâche de démarrage **background** et une tâche de démarrage **foreground** est que dans le cas d’une tâche **foreground** le rôle continue de s’exécuter jusqu’au terme de la tâche **foreground**. Cela signifie également qu’en cas de blocage ou de défaillance de la tâche **foreground**, le rôle n’est pas recyclé tant que la tâche **foreground** n’a pas fait l’objet d’une fermeture forcée. Pour cette raison, les tâches **background** sont recommandées pour les tâches de démarrage asynchrones sauf si vous avez besoin de cette fonctionnalité de la tâche **foreground**.

### <a name="end-batch-files-with-exit-b-0"></a>Terminer des fichiers de commandes par EXIT /B 0
Le rôle ne démarre que si le code **errorlevel** de chacune de vos tâches de démarrage simples est égal à zéro. Comme tous les programmes ne définissent pas correctement le code **errorlevel** (code de sortie), le fichier de commandes doit se terminer par un `EXIT /B 0` si tout s’est exécuté correctement.

L’absence d’un `EXIT /B 0` à la fin d’un fichier de commandes de démarrage est une cause courante du non-démarrage des rôles.

> [!NOTE]
> J’ai remarqué que les fichiers batch imbriqués cessent parfois de répondre lors de l’utilisation du paramètre `/B`. Vous souhaitez peut-être vous assurer que ce problème ne se produit pas si un autre fichier batch appelle votre fichier batch actuel, comme lors de l’utilisation du [wrapper de journal](#always-log-startup-activities). Vous pouvez omettre le paramètre `/B` dans ce cas.
> 
> 

### <a name="expect-startup-tasks-to-run-more-than-once"></a>Prévoir plusieurs exécutions des tâches de démarrage
Tous les recyclages de rôle n’incluent pas un redémarrage, mais ils comprennent tous l’exécution de toutes les tâches de démarrage. Cela signifie que les tâches de démarrage doivent être en mesure de s’exécuter plusieurs fois entre les redémarrages sans problème. Ce sujet est abordé dans la [section précédente](#detect-that-your-task-has-already-run).

### <a name="use-local-storage-to-store-files-that-must-be-accessed-in-the-role"></a>Utiliser le stockage local pour stocker les fichiers qui doivent être accessibles dans le rôle
Si vous souhaitez copier ou créer un fichier pendant votre tâche de démarrage qui soit accessible à votre rôle, ce fichier doit être placé dans le stockage local. Voir la [section précédente](#create-files-in-local-storage-from-a-startup-task).

## <a name="next-steps"></a>Étapes suivantes
Examiner [le modèle et le package de service cloud](cloud-services-model-and-package.md)

En savoir plus sur le fonctionnement des [tâches](cloud-services-startup-tasks.md) .

[Créez et déployez](cloud-services-how-to-create-deploy-portal.md) votre package de service cloud.

[ServiceDefinition.csdef]: cloud-services-model-and-package.md#csdef
[Tâche]: /previous-versions/azure/reference/gg557552(v=azure.100)#Task
[Startup]: /previous-versions/azure/reference/gg557552(v=azure.100)#Startup
[Runtime]: /previous-versions/azure/reference/gg557552(v=azure.100)#Runtime
[Environment]: /previous-versions/azure/reference/gg557552(v=azure.100)#Environment
[Variable]: /previous-versions/azure/reference/gg557552(v=azure.100)#Variable
[RoleInstanceValue]: /previous-versions/azure/reference/gg557552(v=azure.100)#RoleInstanceValue
[RoleEnvironment]: /previous-versions/azure/reference/ee773173(v=azure.100)
[Points de terminaison]: /previous-versions/azure/reference/gg557552(v=azure.100)#Endpoints
[LocalStorage]: /previous-versions/azure/reference/gg557552(v=azure.100)#LocalStorage
[LocalResources]: /previous-versions/azure/reference/gg557552(v=azure.100)#LocalResources
[RoleInstanceValue]: /previous-versions/azure/reference/gg557552(v=azure.100)#RoleInstanceValue