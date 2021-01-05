---
title: Problèmes connus et résolution des problèmes d’Azure Application Insights Agent | Microsoft Docs
description: Problèmes connus liés à Application Insights Agent et exemples de résolution. Surveillez les performances de site web sans avoir à redéployer le site web. Fonctionne avec les applications web ASP.NET hébergées en local, dans des machines virtuelles ou sur Azure.
ms.topic: conceptual
author: TimothyMothra
ms.author: tilee
ms.date: 04/23/2019
ms.openlocfilehash: df8ebd97ed9e34e0fce85a50297c595732057115
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91761462"
---
# <a name="troubleshooting-application-insights-agent-formerly-named-status-monitor-v2"></a>Résolution des problèmes d’Application Insights Agent (anciennement appelé Status Monitor v2)

Lorsque vous activez la surveillance, vous pouvez rencontrer des problèmes qui empêchent la collecte de données.
Cet article répertorie tous les problèmes connus et propose des exemples de résolution des problèmes.

## <a name="known-issues"></a>Problèmes connus

### <a name="conflicting-dlls-in-an-apps-bin-directory"></a>DLL en conflit dans le répertoire bin de l’application

Si une de ces DLL est présente dans le répertoire bin, la surveillance risque d’échouer :

- Microsoft.ApplicationInsights.dll
- Microsoft.AspNet.TelemetryCorrelation.dll
- System.Diagnostics.DiagnosticSource.dll

Certaines de ces DLL sont incluses dans les modèles d’applications Visual Studio par défaut, même si votre application ne les utilise pas.
Vous pouvez utiliser les outils de résolution des problèmes pour constater un comportement symptomatique :

- PerfView :
    ```
    ThreadID="7,500" 
    ProcessorNumber="0" 
    msg="Found 'System.Diagnostics.DiagnosticSource, Version=4.0.2.1, Culture=neutral, PublicKeyToken=cc7b13ffcd2ddd51' assembly, skipping attaching redfield binaries" 
    ExtVer="2.8.13.5972" 
    SubscriptionId="" 
    AppName="" 
    FormattedMessage="Found 'System.Diagnostics.DiagnosticSource, Version=4.0.2.1, Culture=neutral, PublicKeyToken=cc7b13ffcd2ddd51' assembly, skipping attaching redfield binaries" 
    ```

- IISReset et chargement d’application (sans les données de télémétrie). Examiner avec Sysinternals (Handle.exe et ListDLLs.exe) :
    ```
    .\handle64.exe -p w3wp | findstr /I "InstrumentationEngine AI. ApplicationInsights"
    E54: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.ApplicationInsights.RedfieldIISModule.dll

    .\Listdlls64.exe w3wp | findstr /I "InstrumentationEngine AI ApplicationInsights"
    0x0000000009be0000  0x127000  C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Instrumentation64\MicrosoftInstrumentationEngine_x64.dll
    0x0000000009b90000  0x4f000   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Instrumentation64\Microsoft.ApplicationInsights.ExtensionsHost_x64.dll
    0x0000000004d20000  0xb2000   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Instrumentation64\Microsoft.ApplicationInsights.Extensions.Base_x64.dll
    ```

### <a name="conflict-with-iis-shared-configuration"></a>Conflit avec la configuration partagée IIS

Si vous avez un cluster de serveurs web, il se peut que vous utilisiez une [configuration partagée](/iis/web-hosting/configuring-servers-in-the-windows-web-platform/shared-configuration_211).
HttpModule ne peut pas être injecté dans cette configuration partagée.
Exécutez la commande Enable sur chaque serveur web pour installer la DLL dans chaque GAC (Global Assembly Cache).

Après avoir exécuté la commande Enable, procédez comme suit :
1. Accédez au répertoire de configuration partagé et recherchez le fichier applicationHost.config.
2. Ajoutez cette ligne à la section des modules de votre configuration :
    ```
    <modules>
        <!-- Registered global managed http module handler. The 'Microsoft.AppInsights.IIS.ManagedHttpModuleHelper.dll' must be installed in the GAC before this config is applied. -->
        <add name="ManagedHttpModuleHelper" type="Microsoft.AppInsights.IIS.ManagedHttpModuleHelper.ManagedHttpModuleHelper, Microsoft.AppInsights.IIS.ManagedHttpModuleHelper, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35" preCondition="managedHandler,runtimeVersionv4.0" />
    </modules>
    ```

### <a name="iis-nested-applications"></a>Applications imbriquées IIS

Nous n’instrumentons pas les applications imbriquées dans IIS dans la version 1.0.

### <a name="advanced-sdk-configuration-isnt-available"></a>La configuration du kit de développement logiciel (SDK) avancé n’est pas disponible.

La configuration du kit de développement logiciel (SDK) n’est pas exposée à l’utilisateur final dans la version 1.0.

    
    
## <a name="troubleshooting"></a>Dépannage
    
### <a name="troubleshooting-powershell"></a>Résolution des problèmes liés à PowerShell

#### <a name="determine-which-modules-are-available"></a>Déterminer les modules disponibles
Vous pouvez utiliser la commande `Get-Module -ListAvailable` pour déterminer les modules installés.

#### <a name="import-a-module-into-the-current-session"></a>Importer un module dans la session actuelle
Si un module n’a pas été chargé dans une session PowerShell, vous pouvez le charger manuellement à l’aide de la commande `Import-Module <path to psd1>`.


### <a name="troubleshooting-the-application-insights-agent-module"></a>Résolution des problèmes du module Application Insights Agent

#### <a name="list-the-commands-available-in-the-application-insights-agent-module"></a>Lister les commandes disponibles dans le module Application Insights Agent
Exécutez la commande `Get-Command -Module Az.ApplicationMonitor` pour obtenir les commandes disponibles :

```
CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Cmdlet          Disable-ApplicationInsightsMonitoring              0.4.0      Az.ApplicationMonitor
Cmdlet          Disable-InstrumentationEngine                      0.4.0      Az.ApplicationMonitor
Cmdlet          Enable-ApplicationInsightsMonitoring               0.4.0      Az.ApplicationMonitor
Cmdlet          Enable-InstrumentationEngine                       0.4.0      Az.ApplicationMonitor
Cmdlet          Get-ApplicationInsightsMonitoringConfig            0.4.0      Az.ApplicationMonitor
Cmdlet          Get-ApplicationInsightsMonitoringStatus            0.4.0      Az.ApplicationMonitor
Cmdlet          Set-ApplicationInsightsMonitoringConfig            0.4.0      Az.ApplicationMonitor
Cmdlet          Start-ApplicationInsightsMonitoringTrace           0.4.0      Az.ApplicationMonitor
```

#### <a name="determine-the-current-version-of-the-application-insights-agent-module"></a>Déterminer la version actuelle du module Application Insights Agent
Exécutez la commande `Get-ApplicationInsightsMonitoringStatus -PowerShellModule` pour afficher les informations suivantes sur le module :
   - Version du module PowerShell
   - Version du Kit de développement logiciel (SDK) Application Insights
   - Chemin d’accès du module PowerShell
    
Consultez la [documentation de référence de l’API](status-monitor-v2-api-reference.md) pour obtenir une description détaillée de l’utilisation de cette cmdlet.


### <a name="troubleshooting-running-processes"></a>Résolution des problèmes liés aux processus en cours d’exécution

Vous pouvez inspecter les processus sur l’ordinateur instrumenté pour déterminer si toutes les DLL sont chargées.
Si la surveillance fonctionne, au moins 12 DLL doivent être chargées.

Utilisez la commande `Get-ApplicationInsightsMonitoringStatus -InspectProcess` pour vérifier les DLL.

Consultez la [documentation de référence de l’API](status-monitor-v2-api-reference.md) pour obtenir une description détaillée de l’utilisation de cette cmdlet.


### <a name="collect-etw-logs-by-using-perfview"></a>Collecter les journaux ETW à l’aide de PerfView

#### <a name="setup"></a>Installation

1. Télécharger PerfView.exe et PerfView64.exe depuis [GitHub](https://github.com/Microsoft/perfview/releases).
2. Lancez PerfView64.exe.
3. Développez **Options avancées**.
4. Désactivez les cases à cocher suivantes :
    - **Compresser**
    - **Fusionner**
    - **Symbol Collection .NET**
5. Définissez-les **fournisseurs supplémentaires** suivants : `61f6ca3b-4b5f-5602-fa60-759a2a2d1fbd,323adc25-e39b-5c87-8658-2c1af1a92dc5,925fa42b-9ef6-5fa7-10b8-56449d7a2040,f7d60e07-e910-5aca-bdd2-9de45b46c560,7c739bb9-7861-412e-ba50-bf30d95eae36,61f6ca3b-4b5f-5602-fa60-759a2a2d1fbd,323adc25-e39b-5c87-8658-2c1af1a92dc5,252e28f4-43f9-5771-197a-e8c7e750a984`


#### <a name="collecting-logs"></a>Collecte des journaux d’activité

1. Dans une console de commande avec des privilèges d’administrateur, exécutez la commande `iisreset /stop` pour désactiver IIS et toutes les applications web.
2. Dans PerfView, sélectionnez **Démarrer la collecte**.
3. Dans une console de commande avec des privilèges Administrateur, exécutez la commande `iisreset /start` pour démarrer IIS.
4. Essayez d’accéder à votre application.
5. Une fois l’application est chargée, revenez à PerfView et sélectionnez **Arrêter la collecte**.



## <a name="next-steps"></a>Étapes suivantes

- Consultez la [documentation de référence de l’API](status-monitor-v2-overview.md#powershell-api-reference) pour en savoir plus sur les paramètres que vous avez omis.