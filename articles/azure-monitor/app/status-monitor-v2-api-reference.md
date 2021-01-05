---
title: Informations de référence sur l’API de l’agent .Net d’Azure Application Insights
description: Référence API d’Application Insights Agent. Surveillez les performances de site web sans avoir à redéployer le site web. Fonctionne avec les applications web ASP.NET hébergées en local, dans des machines virtuelles ou sur Azure.
ms.topic: conceptual
author: TimothyMothra
ms.author: tilee
ms.date: 04/23/2019
ms.openlocfilehash: 2f814c54aeca8a337f786beb8da1b98accbeef7e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87318995"
---
# <a name="azure-monitor-application-insights-agent-api-reference"></a>Informations de référence sur l’API de l’agent Azure Monitor Application Insights

Cet article décrit une cmdlet appartenant au [module PowerShell Az.ApplicationMonitor](https://www.powershellgallery.com/packages/Az.ApplicationMonitor/).

> [!NOTE] 
> - Pour commencer, vous avez besoin d’une clé d’instrumentation. Pour plus d’informations, consultez [Create an Application Insights resource](create-new-resource.md#copy-the-instrumentation-key) (Création d’une ressource Application Insights).
> - Cette cmdlet nécessite que vous lisiez et acceptiez nos licence et déclaration de confidentialité.

> [!IMPORTANT] 
> Cette cmdlet requiert une session PowerShell avec des autorisations d’administrateur, ainsi qu’une stratégie d’exécution avec élévation des privilèges. Pour plus d’informations, consultez [Run PowerShell as Admin with an elevated execution policy](status-monitor-v2-detailed-instructions.md#run-powershell-as-admin-with-an-elevated-execution-policy) (Exécuter PowerShell en tant qu’administrateur avec une stratégie d’exécution avec élévation de privilèges).
> - Cette cmdlet nécessite que vous lisiez et acceptiez nos licence et déclaration de confidentialité.
> - Le moteur d’instrumentation ajoute une surcharge et est désactivé par défaut.


## <a name="enable-instrumentationengine"></a>Enable-InstrumentationEngine

Active le moteur d’instrumentation en définissant certaines clés de Registre.
Redémarrez IIS pour que les modifications soient prises en compte.

Le moteur d’instrumentation peut compléter les données collectées par les kits de développement logiciel (SDK) .NET.
Il collecte les événements et les messages qui décrivent l’exécution d’un processus managé. Ces événements et messages incluent les codes de résultat de dépendance, les verbes HTTP et le [texte de commande SQL](asp-net-dependencies.md#advanced-sql-tracking-to-get-full-sql-query).

Activez le moteur d’instrumentation si :
- Vous avez déjà activé la supervision avec la cmdlet Enable mais n’avez pas activé le moteur d’instrumentation.
- Vous avez instrumenté manuellement votre application avec les kits de développement (SDK) .NET, et souhaitez collecter des données de télémétrie supplémentaires.

### <a name="examples"></a>Exemples

```powershell
PS C:\> Enable-InstrumentationEngine
```

### <a name="parameters"></a>Paramètres

#### <a name="-acceptlicense"></a>-AcceptLicense
**Facultatif.** Utilisez cette instruction switch pour accepter la licence et la déclaration de confidentialité dans les installations sans affichage.

#### <a name="-verbose"></a>-Verbose
**Paramètre commun.** Utilisez ce commutateur pour générer des journaux détaillés.

### <a name="output"></a>Output


##### <a name="example-output-from-successfully-enabling-the-instrumentation-engine"></a>Exemple de sortie résultant d’une activation réussie du moteur d’instrumentation

```
Configuring IIS Environment for instrumentation engine...
Configuring registry for instrumentation engine...
```

## <a name="enable-applicationinsightsmonitoring"></a>Enable-ApplicationInsightsMonitoring

Permet la surveillance sans code des applications IIS sur un ordinateur cible.

Cette cmdlet modifie l’élément applicationHost.config de l’application IIS et définit des clés de Registre.
Il crée également un fichier applicationinsights.ikey.config, qui définit la clé d’instrumentation utilisée par chaque application.
IIS charge RedfieldModule au démarrage, qui injecte le Kit de développement logiciel (SDK) Application Insights dans les applications à mesure qu’elles démarrent.
Redémarrez IIS pour que vos modifications prennent effet.

Après avoir activé la surveillance, nous vous recommandons d’utiliser [Métriques en temps réel](live-stream.md) pour vérifier rapidement si votre application nous envoie des données de télémétrie.

### <a name="examples"></a>Exemples

#### <a name="example-with-a-single-instrumentation-key"></a>Exemple avec une clé d’instrumentation unique
Dans cet exemple, toutes les applications sur l’ordinateur actuel reçoivent une clé d’instrumentation unique.

```powershell
PS C:\> Enable-ApplicationInsightsMonitoring -InstrumentationKey xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

#### <a name="example-with-an-instrumentation-key-map"></a>Exemple avec un mappage de clé d’instrumentation
Dans cet exemple :
- `MachineFilter` effectue la correspondance avec l’ordinateur actuel en utilisant le caractère générique `'.*'`.
- `AppFilter='WebAppExclude'` fournit une clé d’instrumentation `null`. L’application spécifiée ne sera pas instrumentée.
- `AppFilter='WebAppOne'` attribue à l’application spécifiée une clé d’instrumentation unique.
- `AppFilter='WebAppTwo'` attribue à l’application spécifiée une clé d’instrumentation unique.
- Enfin, `AppFilter` utilise également le caractère générique `'.*'` pour établir les correspondances de toutes les applications web pour lesquelles les règles précédentes n’ont effectué aucune correspondance, et attribuer une clé d’instrumentation par défaut.
- Des espaces ont été ajoutées pour faciliter la lisibilité.

```powershell
PS C:\> Enable-ApplicationInsightsMonitoring -InstrumentationKeyMap 
    @(@{MachineFilter='.*';AppFilter='WebAppExclude'},
      @{MachineFilter='.*';AppFilter='WebAppOne';InstrumentationSettings=@{InstrumentationKey='xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx1'}},
      @{MachineFilter='.*';AppFilter='WebAppTwo';InstrumentationSettings=@{InstrumentationKey='xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx2'}},
      @{MachineFilter='.*';AppFilter='.*';InstrumentationSettings=@{InstrumentationKey='xxxxxxxx-xxxx-xxxx-xxxx-xxxxxdefault'}})

```


### <a name="parameters"></a>Paramètres

#### <a name="-instrumentationkey"></a>-InstrumentationKey
**Obligatoire.** Utilisez ce paramètre pour fournir une clé d’instrumentation unique que l’ensemble des applications utiliseront sur l’ordinateur cible.

#### <a name="-instrumentationkeymap"></a>-InstrumentationKeyMap
**Obligatoire.** Utilisez ce paramètre pour fournir plusieurs clés d’instrumentation et un mappage des clés d’instrumentation utilisées par chaque application.
Vous pouvez créer un script d’installation unique pour plusieurs ordinateurs en définissant le paramètre `MachineFilter`.

> [!IMPORTANT]
> Les applications effectueront le mappage avec les règles dans l’ordre d’apparition de ces dernières. Par conséquent, vous devez d’abord spécifier les règles les plus spécifiques et finir par les règles les plus génériques.

##### <a name="schema"></a>schéma
`@(@{MachineFilter='.*';AppFilter='.*';InstrumentationSettings=@{InstrumentationKey='xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'}})`

- **MachineFilter** est une expression régulière C# obligatoire associée au nom de la machine virtuelle ou de l’ordinateur.
    - ’.*’ effectue la correspondance avec tous les éléments
    - ’ComputerName’ effectue la correspondance avec les ordinateurs présentant le nom exact spécifié uniquement.
- **AppFilter** est une expression C# régulière obligatoire du nom du site IIS. Vous pouvez obtenir la liste des sites sur votre serveur en exécutant la commande [get-iissite](/powershell/module/iisadministration/get-iissite).
    - ’.*’ effectue la correspondance avec tous les éléments
    - « SiteName » correspond uniquement au site IIS portant le nom exact spécifié.
- **InstrumentationKey** est requis pour activer la surveillance des applications qui correspondent aux deux filtres précédents.
    - Conservez la valeur null si vous souhaitez définir des règles pour exclure la surveillance.


#### <a name="-enableinstrumentationengine"></a>-EnableInstrumentationEngine
**Facultatif.** Utilisez cette instruction switch pour permettre au moteur d’instrumentation de collecter des événements et des messages sur ce qui se passe pendant l’exécution d’un processus managé. Ces événements et messages incluent les codes de résultat de dépendance, les verbes HTTP et le texte de commande SQL.

Le moteur d’instrumentation ajoute une surcharge et est désactivé par défaut.

#### <a name="-acceptlicense"></a>-AcceptLicense
**Facultatif.** Utilisez cette instruction switch pour accepter la licence et la déclaration de confidentialité dans les installations sans affichage.

#### <a name="-ignoresharedconfig"></a>-IgnoreSharedConfig
Lorsque vous avez un cluster de serveurs web, il se peut que vous utilisiez une [configuration partagée](/iis/web-hosting/configuring-servers-in-the-windows-web-platform/shared-configuration_211).
HttpModule ne peut pas être injecté dans cette configuration partagée.
Ce script échoue avec le message indiquant que des étapes d’installation supplémentaires sont requises.
Utilisez ce commutateur pour ignorer cette vérification et poursuivre l’installation des prérequis. Pour plus d’informations, consultez [known conflict-with-iis-shared-configuration](status-monitor-v2-troubleshoot.md#conflict-with-iis-shared-configuration)

#### <a name="-verbose"></a>-Verbose
**Paramètre commun.** Utilisez ce commutateur pour afficher les journaux détaillés.

#### <a name="-whatif"></a>-WhatIf 
**Paramètre commun.** Utilisez cette instruction switch pour tester et valider vos paramètres d’entrée sans réellement activer la surveillance.

### <a name="output"></a>Output

#### <a name="example-output-from-a-successful-enablement"></a>Exemple de sortie à partir d’une activation réussie

```powershell
Initiating Disable Process
Applying transformation to 'C:\Windows\System32\inetsrv\config\applicationHost.config'
'C:\Windows\System32\inetsrv\config\applicationHost.config' backed up to 'C:\Windows\System32\inetsrv\config\applicationHost.config.backup-2019-03-26_08-59-52z'
in :1,237
No element in the source document matches '/configuration/location[@path='']/system.webServer/modules/add[@name='ManagedHttpModuleHelper']'
Not executing RemoveAll (transform line 1, 546)
Transformation to 'C:\Windows\System32\inetsrv\config\applicationHost.config' was successfully applied. Operation: 'disable'
GAC Module will not be removed, since this operation might cause IIS instabilities
Configuring IIS Environment for codeless attach...
Registry: skipping non-existent 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\IISADMIN[Environment]
Registry: skipping non-existent 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\W3SVC[Environment]
Registry: skipping non-existent 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WAS[Environment]
Configuring IIS Environment for instrumentation engine...
Registry: skipping non-existent 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\IISADMIN[Environment]
Registry: skipping non-existent 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\W3SVC[Environment]
Registry: skipping non-existent 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WAS[Environment]
Configuring registry for instrumentation engine...
Successfully disabled Application Insights Status Monitor
Installing GAC module 'C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\0.2.0\content\Runtime\Microsoft.AppInsights.IIS.ManagedHttpModuleHelper.dll'
Applying transformation to 'C:\Windows\System32\inetsrv\config\applicationHost.config'
Found GAC module Microsoft.AppInsights.IIS.ManagedHttpModuleHelper.ManagedHttpModuleHelper, Microsoft.AppInsights.IIS.ManagedHttpModuleHelper, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35
'C:\Windows\System32\inetsrv\config\applicationHost.config' backed up to 'C:\Windows\System32\inetsrv\config\applicationHost.config.backup-2019-03-26_08-59-52z_1'
Transformation to 'C:\Windows\System32\inetsrv\config\applicationHost.config' was successfully applied. Operation: 'enable'
Configuring IIS Environment for codeless attach...
Configuring IIS Environment for instrumentation engine...
Configuring registry for instrumentation engine...
Updating app pool permissions...
Successfully enabled Application Insights Status Monitor
```

## <a name="disable-instrumentationengine"></a>Disable-InstrumentationEngine

Désactive le moteur d’instrumentation en supprimant certaines clés de Registre.
Redémarrez IIS pour que les modifications soient prises en compte.

### <a name="examples"></a>Exemples

```powershell
PS C:\> Disable-InstrumentationEngine
```

### <a name="parameters"></a>Paramètres 

#### <a name="-verbose"></a>-Verbose
**Paramètre commun.** Utilisez ce commutateur pour générer des journaux détaillés.

### <a name="output"></a>Output


##### <a name="example-output-from-successfully-disabling-the-instrumentation-engine"></a>Exemple de sortie à partir d’une désactivation réussie du moteur d’instrumentation

```powershell
Configuring IIS Environment for instrumentation engine...
Registry: removing 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\IISADMIN[Environment]'
Registry: removing 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\W3SVC[Environment]'
Registry: removing 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WAS[Environment]'
Configuring registry for instrumentation engine...
```

## <a name="disable-applicationinsightsmonitoring"></a>Disable-ApplicationInsightsMonitoring

Désactive l’analyse sur l’ordinateur cible.
Cette applet de commande supprime les modifications apportées à l’élément applicationHost.config de l’application IIS et supprime les clés de Registre.

### <a name="examples"></a>Exemples

```powershell
PS C:\> Disable-ApplicationInsightsMonitoring
```

### <a name="parameters"></a>Paramètres 

#### <a name="-verbose"></a>-Verbose
**Paramètre commun.** Utilisez ce commutateur pour afficher les journaux détaillés.

### <a name="output"></a>Output


##### <a name="example-output-from-successfully-disabling-monitoring"></a>Exemple de sortie à partir d’une désactivation réussie de la surveillance

```powershell
Initiating Disable Process
Applying transformation to 'C:\Windows\System32\inetsrv\config\applicationHost.config'
'C:\Windows\System32\inetsrv\config\applicationHost.config' backed up to 'C:\Windows\System32\inetsrv\config\applicationHost.config.backup-2019-03-26_08-59-00z'
in :1,237
No element in the source document matches '/configuration/location[@path='']/system.webServer/modules/add[@name='ManagedHttpModuleHelper']'
Not executing RemoveAll (transform line 1, 546)
Transformation to 'C:\Windows\System32\inetsrv\config\applicationHost.config' was successfully applied. Operation: 'disable'
GAC Module will not be removed, since this operation might cause IIS instabilities
Configuring IIS Environment for codeless attach...
Registry: skipping non-existent 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\IISADMIN[Environment]
Registry: skipping non-existent 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\W3SVC[Environment]
Registry: skipping non-existent 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WAS[Environment]
Configuring IIS Environment for instrumentation engine...
Registry: skipping non-existent 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\IISADMIN[Environment]
Registry: skipping non-existent 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\W3SVC[Environment]
Registry: skipping non-existent 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WAS[Environment]
Configuring registry for instrumentation engine...
Successfully disabled Application Insights Status Monitor
```


## <a name="get-applicationinsightsmonitoringconfig"></a>Get-ApplicationInsightsMonitoringConfig

Obtient le fichier de configuration et imprime les valeurs dans la console.

### <a name="examples"></a>Exemples

```powershell
PS C:\> Get-ApplicationInsightsMonitoringConfig
```

### <a name="parameters"></a>Paramètres

Aucun paramètre requis.

### <a name="output"></a>Output


##### <a name="example-output-from-reading-the-config-file"></a>Exemple de sortie à partir de la lecture du fichier de configuration

```
RedfieldConfiguration:
Filters:
0)InstrumentationKey:  AppFilter: WebAppExclude MachineFilter: .*
1)InstrumentationKey: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx2 AppFilter: WebAppTwo MachineFilter: .*
2)InstrumentationKey: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxdefault AppFilter: .* MachineFilter: .*
```

## <a name="get-applicationinsightsmonitoringstatus"></a>Get-ApplicationInsightsMonitoringStatus

Ce cmdlet fournit des informations de dépannage sur Status Monitor.
Utilisez ce cmdlet pour analyser l’état de l’analyse, la version du module PowerShell et pour inspecter le processus en cours d’exécution.
Cette applet de commande signale les informations de version et les informations sur les fichiers de clés nécessaires à la supervision.

### <a name="examples"></a>Exemples

#### <a name="example-application-status"></a>Exemple : État de l’application

Exécutez la commande `Get-ApplicationInsightsMonitoringStatus` pour afficher l’état d’analyse des sites Web.

```powershell

PS C:\Windows\system32> Get-ApplicationInsightsMonitoringStatus

IIS Websites:

SiteName               : Default Web Site
ApplicationPoolName    : DefaultAppPool
SiteId                 : 1
SiteState              : Stopped

SiteName               : DemoWebApp111
ApplicationPoolName    : DemoWebApp111
SiteId                 : 2
SiteState              : Started
ProcessId              : not found

SiteName               : DemoWebApp222
ApplicationPoolName    : DemoWebApp222
SiteId                 : 3
SiteState              : Started
ProcessId              : 2024
Instrumented           : true
InstrumentationKey     : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx123

SiteName               : DemoWebApp333
ApplicationPoolName    : DemoWebApp333
SiteId                 : 4
SiteState              : Started
ProcessId              : 5184
AppAlreadyInstrumented : true
```

Dans cet exemple,
- l’**identificateur de la machine** est un ID anonyme utilisé pour identifier de manière unique votre serveur. Si vous créez une demande d’assistance, nous aurons besoin de cet ID pour rechercher les journaux de votre serveur.
- **Le site Web par défaut** est arrêté dans IIS
- **DemoWebApp111** a été démarré dans IIS, mais n’a reçu aucune demande. Ce rapport indique qu’il n’y a aucun processus en cours d’exécution (ProcessId: not found).
- **DemoWebApp222** est en cours d’exécution et est en cours d’analyse (Instrumented: true). En fonction de la configuration utilisateur, la clé d’instrumentation xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx123 a été mise en correspondance pour ce site.
- **DemoWebApp333** a été instrumenté manuellement à l’aide du kit de développement logiciel (SDK) Application Insights. Status Monitor a détecté le kit de développement logiciel (SDK) et n’analyse pas ce site.


#### <a name="example-powershell-module-information"></a>Exemple : Informations sur le module PowerShell

Exécutez la commande `Get-ApplicationInsightsMonitoringStatus -PowerShellModule` pour afficher des informations sur le module actuel :

```powershell

PS C:\> Get-ApplicationInsightsMonitoringStatus -PowerShellModule

PowerShell Module version:
0.4.0-alpha

Application Insights SDK version:
2.9.0.3872

Executing PowerShell Module Assembly:
Microsoft.ApplicationInsights.Redfield.Configurator.PowerShell, Version=2.8.14.11432, Culture=neutral, PublicKeyToken=31bf3856ad364e35

PowerShell Module Directory:
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\0.2.2\content\PowerShell

Runtime Paths:
ParentDirectory (Exists: True)
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content

ConfigurationPath (Exists: True)
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\applicationInsights.ikey.config

ManagedHttpModuleHelperPath (Exists: True)
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.AppInsights.IIS.ManagedHttpModuleHelper.dll

RedfieldIISModulePath (Exists: True)
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.ApplicationInsights.RedfieldIISModule.dll

InstrumentationEngine86Path (Exists: True)
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Instrumentation32\MicrosoftInstrumentationEngine_x86.dll

InstrumentationEngine64Path (Exists: True)
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Instrumentation64\MicrosoftInstrumentationEngine_x64.dll

InstrumentationEngineExtensionHost86Path (Exists: True)
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Instrumentation32\Microsoft.ApplicationInsights.ExtensionsHost_x86.dll

InstrumentationEngineExtensionHost64Path (Exists: True)
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Instrumentation64\Microsoft.ApplicationInsights.ExtensionsHost_x64.dll

InstrumentationEngineExtensionConfig86Path (Exists: True)
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Instrumentation32\Microsoft.InstrumentationEngine.Extensions.config

InstrumentationEngineExtensionConfig64Path (Exists: True)
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Instrumentation64\Microsoft.InstrumentationEngine.Extensions.config

ApplicationInsightsSdkPath (Exists: True)
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.ApplicationInsights.dll
```

#### <a name="example-runtime-status"></a>Exemple : État d’exécution

Vous pouvez inspecter le processus sur l’ordinateur instrumenté pour vérifier si toutes les DLL sont chargées. Si la surveillance fonctionne, au moins 12 DLL doivent être chargées.

Exécutez la commande suivante `Get-ApplicationInsightsMonitoringStatus -InspectProcess` :


```
PS C:\> Get-ApplicationInsightsMonitoringStatus -InspectProcess

iisreset.exe /status
Status for IIS Admin Service ( IISADMIN ) : Running
Status for Windows Process Activation Service ( WAS ) : Running
Status for Net.Msmq Listener Adapter ( NetMsmqActivator ) : Running
Status for Net.Pipe Listener Adapter ( NetPipeActivator ) : Running
Status for Net.Tcp Listener Adapter ( NetTcpActivator ) : Running
Status for World Wide Web Publishing Service ( W3SVC ) : Running

handle64.exe -accepteula -p w3wp
BF0: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.AI.ServerTelemetryChannel.dll
C58: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.AI.AzureAppServices.dll
C68: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.AI.DependencyCollector.dll
C78: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.AI.WindowsServer.dll
C98: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.AI.Web.dll
CBC: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.AI.PerfCounterCollector.dll
DB0: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.AI.Agent.Intercept.dll
B98: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.ApplicationInsights.RedfieldIISModule.dll
BB4: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.ApplicationInsights.RedfieldIISModule.Contracts.dll
BCC: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.ApplicationInsights.Redfield.Lightup.dll
BE0: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.ApplicationInsights.dll

listdlls64.exe -accepteula w3wp
0x0000000019ac0000  0x127000  C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Instrumentation64\MicrosoftInstrumentationEngine_x64.dll
0x00000000198b0000  0x4f000   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Instrumentation64\Microsoft.ApplicationInsights.ExtensionsHost_x64.dll
0x000000000c460000  0xb2000   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Instrumentation64\Microsoft.ApplicationInsights.Extensions.Base_x64.dll
0x000000000ad60000  0x108000  C:\Windows\TEMP\2.4.0.0.Microsoft.ApplicationInsights.Extensions.Intercept_x64.dll
```

### <a name="parameters"></a>Paramètres

#### <a name="no-parameters"></a>(Aucun paramètre)

Par défaut, ce cmdlet signale l’état d’analyse des applications Web.
Utilisez cette option pour vérifier si votre application a été correctement instrumentée.
Vous pouvez également examiner la clé d’instrumentation qui a été mise en correspondance avec votre site.


#### <a name="-powershellmodule"></a>-PowerShellModule
**Facultatif**. Utilisez ce commutateur pour signaler les numéros de version et les chemins des DLL nécessaires à l’analyse.
Utilisez cette option si vous avez besoin identifier la version de n’importe quelle DLL, y compris le SDK Application Insights.

#### <a name="-inspectprocess"></a>-InspectProcess

**Facultatif**. Utilisez ce commutateur pour signaler si IIS est en cours d’exécution.
Il télécharge également des outils externes pour déterminer si les DLL sont chargées dans le runtime d’IIS.


Si ce processus échoue pour une raison quelconque, vous pouvez exécuter ces commandes manuellement :
- iisreset.exe /status
- [handle64.exe](/sysinternals/downloads/handle) -p w3wp | findstr /I "InstrumentationEngine AI. ApplicationInsights"
- [listdlls64.exe](/sysinternals/downloads/listdlls) w3wp | findstr /I "InstrumentationEngine AI ApplicationInsights"


#### <a name="-force"></a>-Force

**Facultatif**. Utilisé uniquement avec InspectProcess. Utilisez ce commutateur pour ignorer l’invite utilisateur qui apparaît avant le téléchargement d’outils supplémentaires.


## <a name="set-applicationinsightsmonitoringconfig"></a>Set-ApplicationInsightsMonitoringConfig

Définit le fichier de configuration sans effectuer de réinstallation complète.
Redémarrez IIS pour que vos modifications prennent effet.

> [!IMPORTANT] 
> Cette cmdlet requiert une session PowerShell avec des autorisations d’administrateur.


### <a name="examples"></a>Exemples

#### <a name="example-with-a-single-instrumentation-key"></a>Exemple avec une clé d’instrumentation unique
Dans cet exemple, toutes les applications sur l’ordinateur actuel recevront une clé d’instrumentation unique.

```powershell
PS C:\> Enable-ApplicationInsightsMonitoring -InstrumentationKey xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

#### <a name="example-with-an-instrumentation-key-map"></a>Exemple avec un mappage de clé d’instrumentation
Dans cet exemple :
- `MachineFilter` effectue la correspondance avec l’ordinateur actuel en utilisant le caractère générique `'.*'`.
- `AppFilter='WebAppExclude'` fournit une clé d’instrumentation `null`. L’application spécifiée ne sera pas instrumentée.
- `AppFilter='WebAppOne'` attribue à l’application spécifiée une clé d’instrumentation unique.
- `AppFilter='WebAppTwo'` attribue à l’application spécifiée une clé d’instrumentation unique.
- Enfin, `AppFilter` utilise également le caractère générique `'.*'` pour établir les correspondances de toutes les applications web pour lesquelles les règles précédentes n’ont effectué aucune correspondance, et attribuer une clé d’instrumentation par défaut.
- Des espaces ont été ajoutées pour faciliter la lisibilité.

```powershell
Enable-ApplicationInsightsMonitoring -InstrumentationKeyMap `
       @(@{MachineFilter='.*';AppFilter='WebAppExclude'},
          @{MachineFilter='.*';AppFilter='WebAppOne';InstrumentationSettings=@{InstrumentationKey='xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx1'}},
          @{MachineFilter='.*';AppFilter='WebAppTwo';InstrumentationSettings=@{InstrumentationKey='xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx2'}},
          @{MachineFilter='.*';AppFilter='.*';InstrumentationSettings=@{InstrumentationKey='xxxxxxxx-xxxx-xxxx-xxxx-xxxxxdefault'}})
```

### <a name="parameters"></a>Paramètres

#### <a name="-instrumentationkey"></a>-InstrumentationKey
**Obligatoire.** Utilisez ce paramètre pour fournir une clé d’instrumentation unique que l’ensemble des applications utiliseront sur l’ordinateur cible.

#### <a name="-instrumentationkeymap"></a>-InstrumentationKeyMap
**Obligatoire.** Utilisez ce paramètre pour fournir plusieurs clés d’instrumentation et un mappage des clés d’instrumentation utilisées par chaque application.
Vous pouvez créer un script d’installation unique pour plusieurs ordinateurs en définissant le paramètre `MachineFilter`.

> [!IMPORTANT]
> Les applications effectueront le mappage avec les règles dans l’ordre d’apparition de ces dernières. Par conséquent, vous devez d’abord spécifier les règles les plus spécifiques et finir par les règles les plus génériques.

##### <a name="schema"></a>schéma
`@(@{MachineFilter='.*';AppFilter='.*';InstrumentationKey='xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'})`

- **MachineFilter** est une expression régulière C# obligatoire associée au nom de la machine virtuelle ou de l’ordinateur.
    - ’.*’ effectue la correspondance avec tous les éléments
    - « ComputerName » effectue la correspondance avec les ordinateurs présentant le nom spécifié uniquement.
- **AppFilter** est une expression régulière C# obligatoire associée au nom de la machine virtuelle ou de l’ordinateur.
    - ’.*’ effectue la correspondance avec tous les éléments
    - « ComputerName » effectue la correspondance avec les applications IIS présentant le nom spécifié uniquement.
- Le paramètre **Clé d’instrumentation** est requis pour activer la surveillance des applications qui correspondent aux deux filtres précédents.
    - Conservez la valeur null si vous souhaitez définir des règles pour exclure la surveillance.


#### <a name="-verbose"></a>-Verbose
**Paramètre commun.** Utilisez ce commutateur pour afficher les journaux détaillés.


### <a name="output"></a>Output

Par défaut, aucune sortie.

##### <a name="example-verbose-output-from-setting-the-config-file-via--instrumentationkey"></a>Exemple détaillé de la sortie à partir de la définition du fichier de configuration via -InstrumentationKey

```
VERBOSE: Operation: InstallWithIkey
VERBOSE: InstrumentationKeyMap parsed:
Filters:
0)InstrumentationKey: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx AppFilter: .* MachineFilter: .*
VERBOSE: set config file
VERBOSE: Config File Path:
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\applicationInsights.ikey.config
```

##### <a name="example-verbose-output-from-setting-the-config-file-via--instrumentationkeymap"></a>Exemple détaillé de la sortie à partir de la définition du fichier de configuration via -InstrumentationKeyMap

```
VERBOSE: Operation: InstallWithIkeyMap
VERBOSE: InstrumentationKeyMap parsed:
Filters:
0)InstrumentationKey:  AppFilter: WebAppExclude MachineFilter: .*
1)InstrumentationKey: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx2 AppFilter: WebAppTwo MachineFilter: .*
2)InstrumentationKey: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxdefault AppFilter: .* MachineFilter: .*
VERBOSE: set config file
VERBOSE: Config File Path:
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\applicationInsights.ikey.config
```

## <a name="start-applicationinsightsmonitoringtrace"></a>Start-ApplicationInsightsMonitoringTrace

Collecte des [événements ETW](/windows/desktop/etw/event-tracing-portal) à partir du runtime joint sans code. Cette cmdlet est une alternative à l’exécution de [PerfView](https://github.com/microsoft/perfview).

Les événements collectés seront imprimés dans la console en temps réel et enregistrés dans un fichier ETL. Vous pouvez ouvrir le fichier ETL de sortie avec [PerfView](https://github.com/microsoft/perfview) pour un examen approfondi.

Cette cmdlet s’exécute jusqu’à la fin du délai d’expiration (5 minutes par défaut) ou vous pouvez l’arrêter manuellement (`Ctrl + C`).

### <a name="examples"></a>Exemples

#### <a name="how-to-collect-events"></a>Comment collecter des événements

Normalement, nous vous demandons de collecter des événements pour examiner pourquoi votre application n’est pas instrumentée actuellement.

Le runtime joint sans code émettra des événements ETW au démarrage d’IIS et au lancement de votre application.

Pour collecter ces événements, procédez comme suit :
1. Dans une console de commande avec des privilèges d’administrateur, exécutez la commande `iisreset /stop` pour désactiver IIS et toutes les applications web.
2. Exécuter cette cmdlet
3. Dans une console de commande avec des privilèges d’administrateur, exécutez la commande `iisreset /start` pour démarrer IIS.
4. Essayez d’accéder à votre application.
5. Une fois votre application chargée, vous pouvez l’arrêter manuellement (`Ctrl + C`) ou attendre la fin du délai d’attente.

#### <a name="what-events-to-collect"></a>Événements à collecter

Vous disposez de trois options lors de la collecte des événements :
1. Utiliser le commutateur `-CollectSdkEvents` pour collecter les événements émis par le kit de développement logiciel (SDK) Application Insights.
2. Utiliser le commutateur `-CollectRedfieldEvents` pour collecter les événements émis par Status Monitor et le runtime Redfield. Ces journaux sont utiles pour le diagnostic d’IIS et le démarrage de l’application.
3. Utiliser les deux commutateurs pour collecter les deux types d’événements.
4. Par défaut, si aucun commutateur n’est spécifié, les deux types d’événements seront collectés.


### <a name="parameters"></a>Paramètres

#### <a name="-maxdurationinminutes"></a>-MaxDurationInMinutes
**Facultatif.** Utilisez ce paramètre pour définir la durée pendant laquelle ce script doit collecter les événements. La valeur par défaut est de 5 minutes.

#### <a name="-logdirectory"></a>-LogDirectory
**Facultatif.** Utilisez ce commutateur pour définir le répertoire de sortie du fichier ETL. Par défaut, ce fichier sera être créé dans le répertoire des modules PowerShell. Le chemin d’accès complet sera affiché lors de l’exécution du script.


#### <a name="-collectsdkevents"></a>-CollectSdkEvents
**Facultatif.** Utilisez ce commutateur pour collecter les événements du kit de développement logiciel (SDK) Application Insights.

#### <a name="-collectredfieldevents"></a>-CollectRedfieldEvents
**Facultatif.** Utiliser ce commutateur pour collecter les événements émis par Status Monitor et le runtime Redfield.

#### <a name="-verbose"></a>-Verbose
**Paramètre commun.** Utilisez ce commutateur pour générer des journaux détaillés.



### <a name="output"></a>Output


#### <a name="example-of-application-startup-logs"></a>Exemple de journaux de démarrage d’application
```powershell
PS C:\Windows\system32> Start-ApplicationInsightsMonitoringTrace -ColectRedfieldEvents
Starting...
Log File: C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\logs\20190627_144217_ApplicationInsights_ETW_Trace.etl
Tracing enabled, waiting for events.
Tracing will timeout in 5 minutes. Press CTRL+C to cancel.

2:42:31 PM EVENT: Microsoft-ApplicationInsights-IIS-ManagedHttpModuleHelper Trace Resolved variables to: MicrosoftAppInsights_ManagedHttpModulePath='C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.ApplicationInsights.RedfieldIISModule.dll', MicrosoftAppInsights_ManagedHttpModuleType='Microsoft.ApplicationInsights.RedfieldIISModule.RedfieldIISModule'
2:42:31 PM EVENT: Microsoft-ApplicationInsights-IIS-ManagedHttpModuleHelper Trace Resolved variables to: MicrosoftDiagnosticServices_ManagedHttpModulePath2='', MicrosoftDiagnosticServices_ManagedHttpModuleType2=''
2:42:31 PM EVENT: Microsoft-ApplicationInsights-IIS-ManagedHttpModuleHelper Trace Environment variable 'MicrosoftDiagnosticServices_ManagedHttpModulePath2' or 'MicrosoftDiagnosticServices_ManagedHttpModuleType2' is null, skipping managed dll loading
2:42:31 PM EVENT: Microsoft-ApplicationInsights-IIS-ManagedHttpModuleHelper Trace MulticastHttpModule.constructor, success, 70 ms
2:42:31 PM EVENT: Microsoft-ApplicationInsights-RedfieldIISModule Trace Current assembly 'Microsoft.ApplicationInsights.RedfieldIISModule, Version=2.8.18.27202, Culture=neutral, PublicKeyToken=f23a46de0be5d6f3' location 'C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.ApplicationInsights.RedfieldIISModule.dll'
2:42:31 PM EVENT: Microsoft-ApplicationInsights-RedfieldIISModule Trace Matched filter '.*'~'STATUSMONITORTE', '.*'~'DemoWithSql'
2:42:31 PM EVENT: Microsoft-ApplicationInsights-RedfieldIISModule Trace Lightup assembly calculated path: 'C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.ApplicationInsights.Redfield.Lightup.dll'
2:42:31 PM EVENT: Microsoft-ApplicationInsights-FrameworkLightup Trace Loaded applicationInsights.config from assembly's resource Microsoft.ApplicationInsights.Redfield.Lightup, Version=2.8.18.27202, Culture=neutral, PublicKeyToken=f23a46de0be5d6f3/Microsoft.ApplicationInsights.Redfield.Lightup.ApplicationInsights-recommended.config
2:42:34 PM EVENT: Microsoft-ApplicationInsights-FrameworkLightup Trace Successfully attached ApplicationInsights SDK
2:42:34 PM EVENT: Microsoft-ApplicationInsights-RedfieldIISModule Trace RedfieldIISModule.LoadLightupAssemblyAndGetLightupHttpModuleClass, success, 2687 ms
2:42:34 PM EVENT: Microsoft-ApplicationInsights-RedfieldIISModule Trace RedfieldIISModule.CreateAndInitializeApplicationInsightsHttpModules(lightupHttpModuleClass), success
2:42:34 PM EVENT: Microsoft-ApplicationInsights-IIS-ManagedHttpModuleHelper Trace ManagedHttpModuleHelper, multicastHttpModule.Init() success, 3288 ms
2:42:35 PM EVENT: Microsoft-ApplicationInsights-IIS-ManagedHttpModuleHelper Trace Resolved variables to: MicrosoftAppInsights_ManagedHttpModulePath='C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.ApplicationInsights.RedfieldIISModule.dll', MicrosoftAppInsights_ManagedHttpModuleType='Microsoft.ApplicationInsights.RedfieldIISModule.RedfieldIISModule'
2:42:35 PM EVENT: Microsoft-ApplicationInsights-IIS-ManagedHttpModuleHelper Trace Resolved variables to: MicrosoftDiagnosticServices_ManagedHttpModulePath2='', MicrosoftDiagnosticServices_ManagedHttpModuleType2=''
2:42:35 PM EVENT: Microsoft-ApplicationInsights-IIS-ManagedHttpModuleHelper Trace Environment variable 'MicrosoftDiagnosticServices_ManagedHttpModulePath2' or 'MicrosoftDiagnosticServices_ManagedHttpModuleType2' is null, skipping managed dll loading
2:42:35 PM EVENT: Microsoft-ApplicationInsights-IIS-ManagedHttpModuleHelper Trace MulticastHttpModule.constructor, success, 0 ms
2:42:35 PM EVENT: Microsoft-ApplicationInsights-RedfieldIISModule Trace RedfieldIISModule.CreateAndInitializeApplicationInsightsHttpModules(lightupHttpModuleClass), success
2:42:35 PM EVENT: Microsoft-ApplicationInsights-IIS-ManagedHttpModuleHelper Trace ManagedHttpModuleHelper, multicastHttpModule.Init() success, 0 ms
Timeout Reached. Stopping...
```

## <a name="next-steps"></a>Étapes suivantes

  Affichez vos données de télémétrie :
 - [Explorez les métriques](../platform/metrics-charts.md) pour surveiller les performances et l’utilisation.
- [Effectuez des recherches dans les événements et les journaux](./diagnostic-search.md) pour diagnostiquer les problèmes.
- Utilisez la fonctionnalité [Analytics](../log-query/log-query-overview.md) pour des requêtes plus élaborées.
- [Créez des tableaux de bord](./overview-dashboard.md).
 
 Ajoutez des données de télémétrie :
 - [Créez des tests web](monitor-web-app-availability.md) pour vous assurer que votre site reste actif.
- [Ajoutez la télémétrie de client web](./javascript.md) pour afficher les exceptions à partir du code de la page web et activer le suivi des appels.
- [Ajoutez le kit de développement logiciel (SDK) Application Insights à votre code](./asp-net.md) afin de pouvoir insérer un suivi et journaliser les appels.
 
 En faire plus avec Application Insights Agent :
 - Utilisez notre guide pour [résoudre les problèmes](status-monitor-v2-troubleshoot.md) d’Application Insights Agent.

