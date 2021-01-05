---
title: Prise en main des services cloud Azure et Python | Microsoft Docs
description: Présentation des outils Python pour Visual Studio pour la création de services cloud Azure comprenant des rôles web et de travail.
services: cloud-services
documentationcenter: python
author: tgore03
ms.service: cloud-services
ms.devlang: python
ms.topic: conceptual
ms.date: 07/18/2017
ms.author: tagore
ms.custom: devx-track-python
ms.openlocfilehash: 955498c663560351c01f1894fb348b89f72d549b
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92079239"
---
# <a name="python-web-and-worker-roles-with-python-tools-for-visual-studio"></a>Rôles Web et rôles de travail Python avec Python Tools pour Visual Studio

Cet article fournit une vue d’ensemble de l’utilisation des rôles Web et de travail Python avec [Python Tools pour Visual Studio][Python Tools for Visual Studio]. Apprenez à utiliser Visual Studio pour créer et déployer un service cloud de base utilisant Python.

## <a name="prerequisites"></a>Prérequis
* [Visual Studio 2013, 2015 ou 2017](https://www.visualstudio.com/)
* [Python Tools pour Visual Studio][Python Tools for Visual Studio] (PTVS)
* [Azure SDK Tools pour VS 2013][Azure SDK Tools for VS 2013] ou  
[Azure SDK Tools pour VS 2015][Azure SDK Tools for VS 2015] ou  
[Azure SDK Tools pour VS 2017][Azure SDK Tools for VS 2017]
* [Python 2.7 32 bits][Python 2.7 32-bit] ou [Python 3.5 32 bits][Python 3.5 32-bit]

[!INCLUDE [create-account-and-websites-note](../../includes/create-account-and-websites-note.md)]

## <a name="what-are-python-web-and-worker-roles"></a>Présentation des rôles web et de travail Python
Azure propose trois modèles de calcul pour l’exécution des applications : [Fonctionnalité Web Apps dans Azure App Service][execution model-web sites], [Machines virtuelles Azure][execution model-vms] et [Azure Cloud Services][execution model-cloud services]. Ils prennent tous les trois en charge Python. Azure Cloud Services, qui inclut les rôles Web et de travail, fournit la fonctionnalité *PaaS (Platform as a Service)* . Au sein d'un service cloud, un rôle web fournit un serveur web IIS (Internet Information Services) dédié permettant d'héberger des applications web frontales, tandis qu'un rôle de travail peut exécuter des tâches asynchrones, de longue durée ou perpétuelles indépendamment de l’interaction ou de la saisie de l’utilisateur.

Pour en savoir plus, voir [Présentation d’un service Cloud].

> [!NOTE]
> *Vous voulez créer un simple site web ?*
> Si votre scénario ne comporte qu'un simple composant frontal web, envisagez d'utiliser la fonctionnalité Web Apps légère dans Azure App Service. Vous pouvez facilement passer à un service cloud en fonction de l'évolution de votre site et de vos besoins. Consultez le [Centre de développement Python](https://azure.microsoft.com/develop/python/) pour découvrir des articles consacrés au développement de la fonctionnalité Web Apps dans Azure App Service.
> <br />
> 
> 

## <a name="project-creation"></a>Création du projet
Dans Visual Studio, vous pouvez sélectionner **Azure Cloud Service** dans la boîte de dialogue **Nouveau projet**, sous **Python**.

![Boîte de dialogue Nouveau projet](./media/cloud-services-python-ptvs/new-project-cloud-service.png)

Dans l'Assistant Service Cloud Azure, vous pouvez créer les rôles web et de travail.

![Azure Cloud Service Dialog](./media/cloud-services-python-ptvs/new-service-wizard.png)

Le modèle de rôle de travail est fourni avec un code réutilisable permettant de se connecter à un compte de stockage Azure ou à Azure Service Bus.

![Cloud Service Solution](./media/cloud-services-python-ptvs/worker.png)

Vous pouvez ajouter des rôles web ou des rôles de travail quand vous le souhaitez à un service cloud existant.  Vous pouvez choisir d'ajouter des projets existants à votre solution ou bien d'en créer de nouveaux.

![Add Role Command](./media/cloud-services-python-ptvs/add-new-or-existing-role.png)

Votre service cloud peut contenir des rôles dans différents langages.  Par exemple, vous pouvez avoir un rôle web Python implémenté à l'aide de Django, avec des rôles de travail Python ou C#.  Vous pouvez assurer la communication entre les rôles grâce aux files d'attente Service Bus ou aux files d'attente de stockage.

## <a name="install-python-on-the-cloud-service"></a>Installer Python sur le service cloud
> [!WARNING]
> Au moment de la dernière mise à jour de cet article, les scripts d’installation qui sont installés avec Visual Studio ne fonctionnaient pas. Cette section décrit comment contourner le problème.
> 
> 

Le principal problème inhérent aux scripts d’installation réside dans le fait qu’ils n’installent pas Python. Commencez par définir deux [tâches de démarrage](cloud-services-startup-tasks.md) dans le fichier [ServiceDefinition.csdef](cloud-services-model-and-package.md#servicedefinitioncsdef). La première tâche (**PrepPython.ps1**) télécharge et installe le runtime Python. La seconde tâche (**PipInstaller.ps1**) exécute les paramètres d’initialisation de programme (pip) pour installer toutes vos dépendances éventuelles.

Les scripts ci-après ont été écrits pour Python 3.5. Si vous souhaitez utiliser la version 2.x de Python, définissez le fichier de variables **PYTHON2** sur **on** pour les deux tâches de démarrage et la tâche d’exécution : `<Variable name="PYTHON2" value="<mark>on</mark>" />`.

```xml
<Startup>

  <Task executionContext="elevated" taskType="simple" commandLine="bin\ps.cmd PrepPython.ps1">
    <Environment>
      <Variable name="EMULATED">
        <RoleInstanceValue xpath="/RoleEnvironment/Deployment/@emulated" />
      </Variable>
      <Variable name="PYTHON2" value="off" />
    </Environment>
  </Task>

  <Task executionContext="elevated" taskType="simple" commandLine="bin\ps.cmd PipInstaller.ps1">
    <Environment>
      <Variable name="EMULATED">
        <RoleInstanceValue xpath="/RoleEnvironment/Deployment/@emulated" />
      </Variable>
      <Variable name="PYTHON2" value="off" />
    </Environment>

  </Task>

</Startup>
```

Les variables **PYTHON2** et **PYPATH** doivent être ajoutées à la tâche de démarrage du rôle de travail. La variable **PYPATH** est uniquement utilisée si la variable **PYTHON2** est définie sur **on**.

```xml
<Runtime>
  <Environment>
    <Variable name="EMULATED">
      <RoleInstanceValue xpath="/RoleEnvironment/Deployment/@emulated" />
    </Variable>
    <Variable name="PYTHON2" value="off" />
    <Variable name="PYPATH" value="%SystemDrive%\Python27" />
  </Environment>
  <EntryPoint>
    <ProgramEntryPoint commandLine="bin\ps.cmd LaunchWorker.ps1" setReadyOnProcessStart="true" />
  </EntryPoint>
</Runtime>
```

#### <a name="sample-servicedefinitioncsdef"></a>Exemple de fichier ServiceDefinition.csdef
```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceDefinition name="AzureCloudServicePython" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition" schemaVersion="2015-04.2.6">
  <WorkerRole name="WorkerRole1" vmsize="Small">
    <ConfigurationSettings>
      <Setting name="Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString" />
      <Setting name="Python2" />
    </ConfigurationSettings>
    <Startup>
      <Task executionContext="elevated" taskType="simple" commandLine="bin\ps.cmd PrepPython.ps1">
        <Environment>
          <Variable name="EMULATED">
            <RoleInstanceValue xpath="/RoleEnvironment/Deployment/@emulated" />
          </Variable>
          <Variable name="PYTHON2" value="off" />
        </Environment>
      </Task>
      <Task executionContext="elevated" taskType="simple" commandLine="bin\ps.cmd PipInstaller.ps1">
        <Environment>
          <Variable name="EMULATED">
            <RoleInstanceValue xpath="/RoleEnvironment/Deployment/@emulated" />
          </Variable>
          <Variable name="PYTHON2" value="off" />
        </Environment>
      </Task>
    </Startup>
    <Runtime>
      <Environment>
        <Variable name="EMULATED">
          <RoleInstanceValue xpath="/RoleEnvironment/Deployment/@emulated" />
        </Variable>
        <Variable name="PYTHON2" value="off" />
        <Variable name="PYPATH" value="%SystemDrive%\Python27" />
      </Environment>
      <EntryPoint>
        <ProgramEntryPoint commandLine="bin\ps.cmd LaunchWorker.ps1" setReadyOnProcessStart="true" />
      </EntryPoint>
    </Runtime>
    <Imports>
      <Import moduleName="RemoteAccess" />
      <Import moduleName="RemoteForwarder" />
    </Imports>
  </WorkerRole>
</ServiceDefinition>
```



Ensuite, créez les fichiers **PrepPython.ps1** et **PipInstaller.ps1** dans le dossier **./bin** de votre rôle.

#### <a name="preppythonps1"></a>PrepPython.ps1
Ce script installe Python. Si la variable d’environnement **PYTHON2** est définie sur **on**, Python 2.7 est installé. Dans le cas contraire, le script installé est Python 3.5.

```powershell
[Net.ServicePointManager]::SecurityProtocol = "tls12, tls11, tls"
$is_emulated = $env:EMULATED -eq "true"
$is_python2 = $env:PYTHON2 -eq "on"
$nl = [Environment]::NewLine

if (-not $is_emulated){
    Write-Output "Checking if python is installed...$nl"
    if ($is_python2) {
        & "${env:SystemDrive}\Python27\python.exe"  -V | Out-Null
    }
    else {
        py -V | Out-Null
    }

    if (-not $?) {

        $url = "https://www.python.org/ftp/python/3.5.2/python-3.5.2-amd64.exe"
        $outFile = "${env:TEMP}\python-3.5.2-amd64.exe"

        if ($is_python2) {
            $url = "https://www.python.org/ftp/python/2.7.12/python-2.7.12.amd64.msi"
            $outFile = "${env:TEMP}\python-2.7.12.amd64.msi"
        }

        Write-Output "Not found, downloading $url to $outFile$nl"
        Invoke-WebRequest $url -OutFile $outFile
        Write-Output "Installing$nl"

        if ($is_python2) {
            Start-Process msiexec.exe -ArgumentList "/q", "/i", "$outFile", "ALLUSERS=1" -Wait
        }
        else {
            Start-Process "$outFile" -ArgumentList "/quiet", "InstallAllUsers=1" -Wait
        }

        Write-Output "Done$nl"
    }
    else {
        Write-Output "Already installed"
    }
}
```

#### <a name="pipinstallerps1"></a>PipInstaller.ps1
Ce script appelle les paramètres d’initialisation de programme et installe toutes les dépendances dans le fichier **requirements.txt**. Si la variable d’environnement **PYTHON2** est définie sur **on**, Python 2.7 est installé. Dans le cas contraire, le script installé est Python 3.5.

```powershell
$is_emulated = $env:EMULATED -eq "true"
$is_python2 = $env:PYTHON2 -eq "on"
$nl = [Environment]::NewLine

if (-not $is_emulated){
    Write-Output "Checking if requirements.txt exists$nl"
    if (Test-Path ..\requirements.txt) {
        Write-Output "Found. Processing pip$nl"

        if ($is_python2) {
            & "${env:SystemDrive}\Python27\python.exe" -m pip install -r ..\requirements.txt
        }
        else {
            py -m pip install -r ..\requirements.txt
        }

        Write-Output "Done$nl"
    }
    else {
        Write-Output "Not found$nl"
    }
}
```

#### <a name="modify-launchworkerps1"></a>Modifier LaunchWorker.ps1
> [!NOTE]
> Dans le cas d’un projet de **rôle de travail**, le fichier **LauncherWorker.ps1** est requis pour exécuter le fichier de démarrage. Dans un projet de **rôle Web**, le fichier de démarrage est quant à lui défini dans les propriétés du projet.
> 
> 

À l’origine, le fichier **bin\LaunchWorker.ps1** avait été créé pour effectuer une grande partie du travail de préparation, mais cela ne fonctionne pas vraiment. Remplacez le contenu de ce fichier par le script ci-après.

Ce script appelle le fichier **worker.py** à partir de votre projet Python. Si la variable d’environnement **PYTHON2** est définie sur **on**, Python 2.7 est installé. Dans le cas contraire, le script installé est Python 3.5.

```powershell
$is_emulated = $env:EMULATED -eq "true"
$is_python2 = $env:PYTHON2 -eq "on"
$nl = [Environment]::NewLine

if (-not $is_emulated)
{
    Write-Output "Running worker.py$nl"

    if ($is_python2) {
        cd..
        iex "$env:PYPATH\python.exe worker.py"
    }
    else {
        cd..
        iex "py worker.py"
    }
}
else
{
    Write-Output "Running (EMULATED) worker.py$nl"

    # Customize to your local dev environment

    if ($is_python2) {
        cd..
        iex "$env:PYPATH\python.exe worker.py"
    }
    else {
        cd..
        iex "py worker.py"
    }
}
```

#### <a name="pscmd"></a>ps.cmd
Les modèles Visual Studio doivent avoir créé un fichier **ps.cmd** dans le dossier **./bin**. Ce script shell appelle les scripts de wrapper PowerShell ci-dessus et assure la journalisation en fonction du nom du wrapper PowerShell appelé. Si ce fichier n’a pas été créé, voici ce qu’il doit contenir. 

```cmd
@echo off

cd /D %~dp0

if not exist "%DiagnosticStore%\LogFiles" mkdir "%DiagnosticStore%\LogFiles"
%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe -ExecutionPolicy Unrestricted -File %* >> "%DiagnosticStore%\LogFiles\%~n1.txt" 2>> "%DiagnosticStore%\LogFiles\%~n1.err.txt"
```



## <a name="run-locally"></a>Exécution locale
Si vous définissez votre projet de service cloud comme projet de démarrage et que vous appuyez sur F5, le service cloud s'exécute dans l'émulateur Azure local.

Bien que PTVS puisse être lancé dans l'émulateur, le débogage (ex : points d'arrêt) ne fonctionne pas.

Pour déboguer vos rôles web et vos rôles de travail, vous pouvez définir le projet de rôle comme projet de démarrage, et déboguer ce projet.  Vous pouvez aussi définir plusieurs projets de démarrage.  Cliquez avec le bouton droit sur la solution, puis sélectionnez **Définir comme projet de démarrage**.

![Solution Startup Project Properties](./media/cloud-services-python-ptvs/startup.png)

## <a name="publish-to-azure"></a>Publication dans Azure
Pour la publication, cliquez avec le bouton droit sur le projet de service cloud dans la solution, puis sélectionnez **Publier**.

![Microsoft Azure Publish Sign In](./media/cloud-services-python-ptvs/publish-sign-in.png)

Suivez les instructions de l’Assistant. En cas de besoin, activez le Bureau à distance. Le Bureau à distance vous est utile lorsque vous avez besoin de procéder à un débogage.

Une fois que vous avez terminé la configuration des paramètres, cliquez sur **Publier**.

La progression s'affiche alors dans la fenêtre Sortie, puis vous verrez la fenêtre du Journal d’activité Microsoft Azure.

![Microsoft Azure Activity Log Window](./media/cloud-services-python-ptvs/publish-activity-log.png)

Le déploiement prend plusieurs minutes. Ensuite, vos rôles web et/ou de travail sont exécutés sur Azure !

### <a name="investigate-logs"></a>Examiner les journaux d’activité
Une fois que la machine virtuelle du service cloud a démarré et installé Python, vous pouvez examiner les journaux d’activité pour y rechercher d’éventuels messages d’échec. Ces journaux d’activité sont stockés dans le dossier **C:\Resources\Directory\\{role}\LogFiles**. **PrepPython.err.txt** contient au moins une erreur à partir du moment ou le script tente de détecter l’installation de Python. Il est possible que**PipInstaller.err.txt** signale l’existence d’une version obsolète de pip.

## <a name="next-steps"></a>Étapes suivantes
Pour plus d'informations sur l'utilisation des rôles web et de travail dans les outils Python pour Visual Studio, consultez la documentation PTVS :

* [Projets de service cloud][Cloud Service Projects]

Pour plus d’informations sur l’utilisation des services Azure à partir de vos rôles web et de travail, telles que l’utilisation d’Azure Storage ou Service Bus, consultez les articles suivants :

* [Service BLOB][Blob Service]
* [Service de Table][Table Service]
* [Service de File d’attente][Queue Service]
* [Files d’attente Service Bus][Service Bus Queues]
* [Rubriques Service Bus][Service Bus Topics]

<!--Link references-->

[Présentation d’un service Cloud]: cloud-services-choose-me.md
[execution model-web sites]: ../app-service/overview.md
[execution model-vms]:../virtual-machines/windows/overview.md
[execution model-cloud services]: cloud-services-choose-me.md
[Python Developer Center]: /develop/python/

[Blob Service]:../storage/blobs/storage-python-how-to-use-blob-storage.md
[Queue Service]: ../storage/queues/storage-python-how-to-use-queue-storage.md
[Table Service]:../cosmos-db/table-storage-how-to-use-python.md
[Service Bus Queues]: ../service-bus-messaging/service-bus-python-how-to-use-queues.md
[Service Bus Topics]: ../service-bus-messaging/service-bus-python-how-to-use-topics-subscriptions.md


<!--External Link references-->

[Python Tools for Visual Studio]: https://aka.ms/ptvs
[Python Tools for Visual Studio Documentation]: /visualstudio/python/
[Cloud Service Projects]: /visualstudio/python/python-azure-cloud-service-project-template
[Azure SDK Tools for VS 2013]: https://go.microsoft.com/fwlink/?LinkId=746482
[Azure SDK Tools for VS 2015]: https://go.microsoft.com/fwlink/?LinkId=746481
[Azure SDK Tools for VS 2017]: https://go.microsoft.com/fwlink/?LinkId=746483
[Python 2.7 32-bit]: https://www.python.org/downloads/
[Python 3.5 32-bit]: https://www.python.org/downloads/