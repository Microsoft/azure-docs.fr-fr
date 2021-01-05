---
title: Bien démarrer avec Azure Application Insights Agent | Microsoft Docs
description: Guide de démarrage rapide pour Application Insights Agent. Surveillez les performances de site web sans avoir à redéployer le site web. Fonctionne avec les applications web ASP.NET hébergées en local, dans des machines virtuelles ou sur Azure.
ms.topic: conceptual
author: TimothyMothra
ms.author: tilee
ms.date: 04/23/2019
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 690304ecef80d988f9a554cd10ce4689f5c72133
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89070137"
---
# <a name="get-started-with-azure-monitor-application-insights-agent-for-on-premises-servers"></a>Bien démarrer avec Azure Monitor Application Insights Agent pour les serveurs locaux

Cet article contient les commandes de démarrage rapide susceptibles de fonctionner pour la plupart des environnements.
Les instructions dépendent de PowerShell Gallery pour distribuer les mises à jour.
Ces commandes prennent en charge le paramètre `-Proxy` PowerShell.

Pour obtenir une explication de ces commandes, des instructions de personnalisation et des informations sur le dépannage, consultez les [instructions détaillées](status-monitor-v2-detailed-instructions.md).

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="download-and-install-via-powershell-gallery"></a>Télécharger et installer via PowerShell Gallery

### <a name="install-prerequisites"></a>Prérequis à installer
Exécutez PowerShell en tant qu’administrateur.
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope Process -Force
Install-PackageProvider -Name NuGet -MinimumVersion 2.8.5.201 -Force
Set-PSRepository -Name "PSGallery" -InstallationPolicy Trusted
Install-Module -Name PowerShellGet -Force
``` 
Fermez PowerShell.

### <a name="install-application-insights-agent"></a>Installer Application Insights Agent
Exécutez PowerShell en tant qu’administrateur.
```powershell   
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope Process -Force
Install-Module -Name Az.ApplicationMonitor -AllowPrerelease -AcceptLicense
``` 

### <a name="enable-monitoring"></a>Activer la supervision
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope Process -Force
Enable-ApplicationInsightsMonitoring -InstrumentationKey xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```
    
        
## <a name="download-and-install-manually-offline-option"></a>Télécharger et installer manuellement (option hors connexion)
### <a name="download-the-module"></a>Télécharger le module
Téléchargez manuellement la dernière version du module à partir de [PowerShell Gallery](https://www.powershellgallery.com/packages/Az.ApplicationMonitor).

### <a name="unzip-and-install-application-insights-agent"></a>Décompresser et installer Application Insights Agent
```powershell
$pathToNupkg = "C:\Users\t\Desktop\Az.ApplicationMonitor.0.3.0-alpha.nupkg"
$pathToZip = ([io.path]::ChangeExtension($pathToNupkg, "zip"))
$pathToNupkg | rename-item -newname $pathToZip
$pathInstalledModule = "$Env:ProgramFiles\WindowsPowerShell\Modules\Az.ApplicationMonitor"
Expand-Archive -LiteralPath $pathToZip -DestinationPath $pathInstalledModule
```
### <a name="enable-monitoring"></a>Activer la supervision
```powershell
Enable-ApplicationInsightsMonitoring -InstrumentationKey xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```



## <a name="next-steps"></a>Étapes suivantes

 Affichez vos données de télémétrie :

- [Explorez les métriques](../platform/metrics-charts.md) pour surveiller les performances et l’utilisation.
- [Effectuez des recherches dans les événements et les journaux](./diagnostic-search.md) pour diagnostiquer les problèmes.
- [Utilisez la fonctionnalité Analytics](../log-query/log-query-overview.md) pour des requêtes plus élaborées.
- [Créez des tableaux de bord](./overview-dashboard.md).

 Ajoutez des données de télémétrie :

- [Créez des tests web](monitor-web-app-availability.md) pour vous assurer que votre site reste actif.
- [Ajoutez la télémétrie de client web](./javascript.md) pour afficher les exceptions à partir du code de la page web et activer le suivi des appels.
- [Ajoutez le kit de développement logiciel (SDK) Application Insights à votre code](./asp-net.md) afin de pouvoir insérer un suivi et journaliser les appels.

En faire plus avec Application Insights Agent :

- Examinez les [instructions détaillées](status-monitor-v2-detailed-instructions.md) pour obtenir une explication des commandes répertoriées ici.
- Utilisez notre guide pour [résoudre les problèmes](status-monitor-v2-troubleshoot.md) d’Application Insights Agent.

