---
title: Surveiller une application web ASP.NET active avec Azure Application Insights | Microsoft Docs
description: Analysez les performances d'un site web sans le redéployer. Fonctionne avec les applications web ASP.NET hébergées localement ou dans des machines virtuelles.
ms.topic: conceptual
ms.date: 08/26/2019
ms.custom: devx-track-dotnet
ms.openlocfilehash: 53dbcc341fdd4bc194d34d40cdd2a975df496376
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96186301"
---
# <a name="instrument-web-apps-at-runtime-with-application-insights-codeless-attach"></a>Instrumenter des applications web au runtime avec jonction sans code Application Insights

> [!IMPORTANT]
> L’utilisation de Status Monitor n’est plus recommandée ; **à partir du 1er juin 2021**, cette version de Status Monitor ne sera plus prise en charge. Il a été remplacé par Azure Monitor Application Insights Agent (anciennement appelé Status Monitor v2). Consultez notre documentation sur les [déploiements de serveurs locaux](./status-monitor-v2-overview.md) ou sur les [déploiements de machines virtuelles et de groupes de machines virtuelles identiques Azure](./azure-vm-vmss-apps.md).

Vous pouvez instrumenter une application web dynamique avec Azure Application Insights, sans avoir à modifier ou à redéployer votre code. Cette opération nécessite un abonnement [Microsoft Azure](https://azure.com) .

Status Monitor est utilisé pour instrumenter une application .NET hébergée dans IIS localement ou dans une machine virtuelle.

- Si votre application est déployée dans une machine Azure ou un groupe de machines virtuelles identiques Azure, suivez [ces instructions](azure-vm-vmss-apps.md).
- Si votre application est déployée dans Azure App Services, suivez [les instructions ci-dessous](azure-web-apps.md).
- Si votre application est déployé dans une machine virtuelle Azure, vous pouvez activer l’analyse Application Insights à partir du panneau de configuration Azure.
- (Des articles distincts sont également consacrés à l’instrumentation d’[Azure Cloud Services](./cloudservices.md).)


![Capture d’écran des graphiques de vue d’ensemble App Insights qui contiennent des informations sur les demandes ayant échoué, le temps de réponse du serveur et les requêtes serveur](./media/monitor-performance-live-website-now/overview-graphs.png)

Deux méthodes sont disponibles pour appliquer Application Insights à vos applications web .NET :

* **En cours de création :** [Ajoutez le kit de développement logiciel (SDK) Application Insights][greenbrown] au code de votre application web.
* **En cours d’exécution :** Instrumentez votre application web sur le serveur, comme décrit ci-dessous, sans régénérer ni redéployer le code.

> [!NOTE]
> Si vous utilisez l’instrumentation au moment de la génération, l’instrumentation au moment de l’exécution ne va pas fonctionner, même si elle est activée.

Voici un résumé de ce que vous apporte chaque méthode :

|  | En cours de création | En cours d’exécution |
| --- | --- | --- |
| **Requêtes et exceptions** |Oui |Oui |
| **[Exceptions plus détaillées](./asp-net-exceptions.md)** | |Oui |
| **[Diagnostics de dépendance](./asp-net-dependencies.md)** |Sur .NET 4.6 +, mais moins détaillé |Oui, tous les détails : codes de résultat, texte de commande SQL, verbe HTTP|
| **[Compteurs de performances système](./performance-counters.md)** |Oui |Oui |
| **[API pour la télémétrie personnalisée][api]** |Oui |Non |
| **[Intégration des journaux de suivi](./asp-net-trace-logs.md)** |Oui |Non |
| **[Mode Page et données utilisateur](./javascript.md)** |Oui |Non |
| **Nécessité de regénérer le code** |Oui | Non |



## <a name="monitor-a-live-iis-web-app"></a>Surveiller une application web IIS active

Si votre application est hébergée sur un serveur IIS, activez Application Insights à l’aide de Status Monitor.

1. Sur votre serveur web IIS, connectez-vous avec vos informations d’identification d’administrateur.
2. Si Application Insights Status Monitor n’est pas encore installé, téléchargez et exécutez le [programme d’installation](#download).
3. Dans Status Monitor, sélectionnez l’application web installée ou le site Web à surveiller. Connectez-vous avec vos informations d’identification Azure.

    Configurez la ressource où vous souhaitez afficher les résultats dans le portail Application Insights. (En règle générale, il est préférable de créer une ressource. Sélectionnez une ressource existante si vous avez déjà configuré des [tests web][availability] ou une [supervision du client][client] pour cette application.) 

    ![Choisissez une application et une ressource.](./media/monitor-performance-live-website-now/appinsights-036-configAIC.png)

4. Redémarrez IIS.

    ![Cliquez sur Redémarrer en haut de la boîte de dialogue.](./media/monitor-performance-live-website-now/appinsights-036-restart.png)

    Votre service web sera interrompu pendant une courte période.

## <a name="customize-monitoring-options"></a>Personnaliser les options de surveillance

L’activation d’Application Insights ajoute des DLL et ApplicationInsights.config à votre application web. Vous pouvez [modifier le fichier .config](./configuration-with-applicationinsights-config.md) pour changer certaines options.

## <a name="when-you-re-publish-your-app-re-enable-application-insights"></a>Lorsque vous republiez votre application, réactivez Application Insights

Avant de republier votre application, vous pouvez [ajouter Application Insights au code dans Visual Studio][greenbrown]. Vous obtiendrez des données de télémétrie plus détaillées et aurez la possibilité de personnaliser la télémétrie à l’aide de code.

Si vous souhaitez procéder à la republication sans ajouter Application Insights au code, sachez que le processus de déploiement peut supprimer les DLL et ApplicationInsights.config du site web publié. Par conséquent :

1. Si vous avez modifié le fichier ApplicationInsights.config, copiez-le avant de republier votre application.
2. Republiez votre application.
3. Réactivez la surveillance d’Application Insights. (Utilisez la méthode appropriée : le panneau de configuration de l’application web Azure ou Status Monitor sur un hôte IIS.)
4. Rétablissez les éventuelles modifications que vous avez effectuées dans le fichier .config.


## <a name="troubleshooting"></a><a name="troubleshoot"></a>Résolution des problèmes

### <a name="confirm-a-valid-installation"></a>Vérifier la validité d'une installation 

Voici quelques étapes à suivre pour vérifier que votre installation a réussi.

- Vérifiez que le fichier applicationinsights.config est présent dans le répertoire de l'application cible et contient votre iKey.

- Si vous pensez qu’il manque des données, vous pouvez exécuter une requête dans [Analytics](../log-query/log-analytics-tutorial.md) pour lister tous les rôles cloud qui envoient actuellement des données de télémétrie.
  ```Kusto
  union * | summarize count() by cloud_RoleName, cloud_RoleInstance
  ```

- Si vous devez vérifier qu’Application Insights est correctement attaché, vous pouvez exécuter [Sysinternals Handle](/sysinternals/downloads/handle) dans une fenêtre Commande pour vous assurer que applicationinsights.dll a été chargé par IIS.

  ```console
  handle.exe /p w3wp.exe
  ```


### <a name="cant-connect-no-telemetry"></a>Vous n’arrivez pas à vous connecter ? Vous n’obtenez aucune donnée de télémétrie ?

* Ouvrez [les ports sortants requis](./ip-addresses.md#outgoing-ports) dans le pare-feu de votre serveur pour permettre à Status Monitor de fonctionner.

### <a name="unable-to-login"></a>Impossible d'établir une connexion

Si Status Monitor ne parvient pas à se connecter, procédez à une installation par ligne de commande. Status Monitor tente de se connecter pour récupérer votre iKey, mais vous pouvez la lui fournir manuellement à l'aide de la commande :

```powershell
Import-Module 'C:\Program Files\Microsoft Application Insights\Status Monitor\PowerShell\Microsoft.Diagnostics.Agent.StatusMonitor.PowerShell.dll'
Start-ApplicationInsightsMonitoring -Name appName -InstrumentationKey 00000000-000-000-000-0000000
```

### <a name="could-not-load-file-or-assembly-systemdiagnosticsdiagnosticsource"></a>Could not load file or assembly 'System.Diagnostics.DiagnosticSource'

Cette erreur peut s'afficher après avoir activé Application Insights. Elle est dû au fait que le programme d'installation remplace ce dll dans votre répertoire bin.
Pour y remédier, mettez à jour votre fichier web.config :

```xml
<dependentAssembly>
    <assemblyIdentity name="System.Diagnostics.DiagnosticSource" publicKeyToken="cc7b13ffcd2ddd51"/>
    <bindingRedirect oldVersion="0.0.0.0-4.*.*.*" newVersion="4.0.2.1"/>
</dependentAssembly>
```

Ce problème est suivi [ici](https://github.com/MohanGsk/ApplicationInsights-Home).


### <a name="application-diagnostic-messages"></a>Messages de diagnostic d’application

* Ouvrez Status Monitor et sélectionnez votre application dans le volet gauche. Vérifiez la présence de messages de diagnostic pour cette application dans la section « Notifications de configuration » :

  ![Ouvrez le panneau des performances pour afficher la demande, le temps de réponse, la dépendance et d’autres données.](./media/monitor-performance-live-website-now/appinsights-status-monitor-diagnostics-message.png)
  
### <a name="detailed-logs"></a>Journaux d’activité détaillés

* Par défaut, Status Monitor génère des journaux de diagnostic dans : `C:\Program Files\Microsoft Application Insights\Status Monitor\diagnostics.log`

* Pour disposer de journaux d’activité détaillés, modifiez le fichier de configuration : `C:\Program Files\Microsoft Application Insights\Status Monitor\Microsoft.Diagnostics.Agent.StatusMonitor.exe.config` et ajoutez `<add key="TraceLevel" value="All" />` au `appsettings`.
Redémarrez ensuite Status Monitor.

* Status Monitor est une application .NET, vous pouvez donc également activer [le traçage .NET en ajoutant les diagnostics appropriés au fichier config](/dotnet/framework/configure-apps/file-schema/trace-debug/system-diagnostics-element). Par exemple, dans certains scénarios, il peut être utile de voir ce qu’il se passe au niveau du réseau en [configurant le traçage réseau](/dotnet/framework/network-programming/how-to-configure-network-tracing).

### <a name="insufficient-permissions"></a>Autorisations insuffisantes
  
* Si un message relatif à des « autorisations insuffisantes » s’affiche sur le serveur, procédez comme suit :
  * Dans le Gestionnaire des services Internet, sélectionnez votre pool d’applications, ouvrez **Paramètres avancés** puis, sous **Modèle de processus**, notez l’identité.
  * Dans le panneau de configuration relatif à la gestion de l’ordinateur, ajoutez cette identité au groupe Utilisateurs de l’Analyseur de performances.

### <a name="conflict-with-systems-center-operations-manager"></a>Conflit avec Systems Center Operations Manager

* Si des services MMA/SCOM (Systems Center Operations Manager) sont installés sur votre serveur, certaines versions peuvent entrer en conflit. Désinstallez à la fois SCOM et Moniteur d’état, puis réinstallez des versions les plus récentes.

### <a name="failed-or-incomplete-installation"></a>Installation incomplète ou échec d'installation

En cas d'échec de Status Monitor lors d'une installation, cette dernière risque de ne pas aboutir, sans que Status Monitor puisse y remédier. Une réinitialisation manuelle est alors nécessaire.

Supprimez ces fichiers dans le répertoire de l’application :
- Tous les fichiers DLL de votre répertoire bin commençant par « Microsoft.AI ». ou « Microsoft.ApplicationInsights. »
- Ce fichier DLL de votre répertoire bin « Microsoft.Web.Infrastructure.dll »
- Ce fichier DLL de votre répertoire bin « System.Diagnostics.DiagnosticSource.dll »
- Dans le répertoire de l’application, supprimez « App_Data\packages »
- Dans le répertoire de l’application, supprimez « applicationinsights.config »


### <a name="additional-troubleshooting"></a>Résolution de problèmes supplémentaires

* Consultez [Résolution de problèmes supplémentaires][qna].

## <a name="system-requirements"></a>Configuration requise
Prise en charge du système d’exploitation pour Application Insights Status Monitor sur le serveur :

* Windows Server 2008
* Windows Server 2008 R2
* Windows Server 2012
* Windows Server 2012 R2
* Windows Server 2016

avec la dernière version de Service Pack et .NET Framework 4.5 (Status Monitor est basé sur cette version de l’infrastructure)

Côté client : Windows 7, 8, 8.1 et 10, également avec .NET Framework 4.5

Prise en charge IIS : IIS 7, 7.5, 8, 8.5 (IIS est requis)

## <a name="automation-with-powershell"></a>Automation avec PowerShell
Vous pouvez démarrer et arrêter la surveillance à l’aide de PowerShell sur votre serveur IIS.

Tout d’abord, importez le module Application Insights :

```powershell
Import-Module 'C:\Program Files\Microsoft Application Insights\Status Monitor\PowerShell\Microsoft.Diagnostics.Agent.StatusMonitor.PowerShell.dll'
```

Identifiez les applications qui sont surveillées :

`Get-ApplicationInsightsMonitoringStatus [-Name appName]`

* `-Name` (Facultatif) Nom d’une application web.
* Affiche l’état de la surveillance Application Insights pour chaque application web (ou l’application nommée) dans ce serveur IIS.
* Retourne `ApplicationInsightsApplication` pour chaque application :

  * `SdkState==EnabledAfterDeployment`: L’application est surveillée et a été instrumentée lors de l’exécution par l’outil Status Monitor ou par `Start-ApplicationInsightsMonitoring`.
  * `SdkState==Disabled`: L’application n’est pas instrumentée pour Application Insights. Soit elle n’a jamais été instrumentée, soit la surveillance lors de l’exécution a été désactivée avec l’outil Status Monitor ou avec `Stop-ApplicationInsightsMonitoring`.
  * `SdkState==EnabledByCodeInstrumentation`: L’application a été instrumentée en ajoutant le Kit de développement logiciel (SDK) au code source. Son Kit de développement logiciel (SDK) ne peut pas être mis à jour ou arrêté.
  * `SdkVersion` affiche la version utilisée pour surveiller cette application.
  * `LatestAvailableSdkVersion`affiche la version actuellement disponible dans la galerie NuGet. Pour mettre à niveau l’application vers cette version, utilisez `Update-ApplicationInsightsMonitoring`.

`Start-ApplicationInsightsMonitoring -Name appName -InstrumentationKey 00000000-000-000-000-0000000`

* `-Name` Nom de l’application dans IIS
* `-InstrumentationKey` Ikey de la ressource Application Insights où vous souhaitez afficher les résultats.
* Cette applet de commande affecte uniquement les applications qui ne sont pas déjà instrumentées, c’est-à-dire SdkState==NotInstrumented.

    L’applet de commande n’affecte pas une application qui est déjà instrumentée, que ce soit au moment de la création en ajoutant le Kit de développement logiciel (SDK) au code ou au moment de l’exécution via une utilisation préalable de cette applet de commande.

    La version du Kit de développement logiciel (SDK) utilisée pour instrumenter l’application est la version la plus récente téléchargée sur ce serveur.

    Pour télécharger la version la plus récente, utilisez Update-ApplicationInsightsVersion.
* Retourne `ApplicationInsightsApplication` en cas de réussite. En cas d’échec, il consigne un suivi sur stderr.

   ```output
   Name                      : Default Web Site/WebApp1
   InstrumentationKey        : 00000000-0000-0000-0000-000000000000
   ProfilerState             : ApplicationInsights
   SdkState                  : EnabledAfterDeployment
   SdkVersion                : 1.2.1
   LatestAvailableSdkVersion : 1.2.3
   ```

`Stop-ApplicationInsightsMonitoring [-Name appName | -All]`

* `-Name` Nom d’une application dans IIS
* `-All` Arrête la surveillance de toutes les applications de ce serveur IIS pour lequel `SdkState==EnabledAfterDeployment`
* Arrête la surveillance des applications spécifiées et supprime l’instrumentation. Cela ne fonctionne que pour les applications qui ont été instrumentées lors de l’exécution avec l’outil Status Monitor ou avec Start-ApplicationInsightsApplication. (`SdkState==EnabledAfterDeployment`)
* Renvoie ApplicationInsightsApplication.

`Update-ApplicationInsightsMonitoring -Name appName [-InstrumentationKey "0000000-0000-000-000-0000"`]

* `-Name`: Nom d’une application web dans IIS.
* `-InstrumentationKey` (Facultatif.) Permet de modifier la ressource à laquelle la télémétrie de l’application est envoyée.
* Cette applet de commande :
  * Met à niveau l’application nommée vers la version la plus récente du Kit de développement logiciel (SDK) téléchargée sur cette machine. (Ne fonctionne que si `SdkState==EnabledAfterDeployment`)
  * Si vous fournissez une clé d’instrumentation, l’application nommée est reconfigurée pour envoyer la télémétrie à la ressource avec cette clé. (Fonctionne si `SdkState != Disabled`)

`Update-ApplicationInsightsVersion`

* Télécharge la version la plus récente du Kit de développement logiciel (SDK) Application Insights sur le serveur.

## <a name="questions-about-status-monitor"></a><a name="questions"></a>Questions sur Status Monitor

### <a name="what-is-status-monitor"></a>Qu’est-ce que Status Monitor ?

Il s’agit d’une application de bureau que vous installez sur votre serveur web IIS. Il vous permet d’instrumenter et de configurer des applications web. 

### <a name="when-do-i-use-status-monitor"></a>Quand dois-je utiliser Status Monitor ?

* Vous pouvez l’utiliser pour instrumenter une application web qui s’exécute sur votre serveur IIS, même si elle est déjà en cours d’exécution.
* Vous pouvez également vous en servir pour activer la télémétrie supplémentaire pour les applications web qui ont été [générées avec le Kit de développement logiciel (SDK) Application Insights](./asp-net.md) au moment de la compilation. 

### <a name="can-i-close-it-after-it-runs"></a>Puis-je le fermer il après son exécution ?

Oui. Une fois qu’il a instrumenté les sites web que vous avez sélectionnés, vous pouvez le fermer.

Il ne collecte pas la télémétrie par lui-même. Il configure simplement les applications web et définit certaines autorisations.

### <a name="what-does-status-monitor-do"></a>Que fait Status Monitor ?

Lorsque vous sélectionnez une application web que Status Monitor doit instrumenter :

* Celui-ci télécharge et place les assemblys Application Insights et le fichier .config dans le dossier de fichiers binaires de l’application web.
* Il active le profilage CLR pour collecter les appels de dépendance.

### <a name="what-version-of-application-insights-sdk-does-status-monitor-install"></a>Quelle version du SDK d’Application Insights est installée par Status Monitor ?

À partir de maintenant, Status Monitor peut uniquement installer les versions 2.3 ou 2.4 du SDK d’Application Insights. 

Le kit de développement logiciel (SDK) 2.4 d’Application Insights est la [dernière version prenant en charge .NET 4.0](https://github.com/microsoft/ApplicationInsights-dotnet/releases/tag/v2.5.0-beta1) qui est arrivé en [fin de vie en janvier 2016](https://devblogs.microsoft.com/dotnet/support-ending-for-the-net-framework-4-4-5-and-4-5-1/). Par conséquent, Status Monitor peut dorénavant être utilisé pour instrumenter une application .NET 4.0. 

### <a name="do-i-need-to-run-status-monitor-whenever-i-update-the-app"></a>Dois-je exécuter Status Monitor à chaque mise à jour d’une application ?

Cela n’est pas nécessaire si vous la redéployez de façon incrémentielle. 

Si vous sélectionnez l’option « Supprimer les fichiers existants » dans le processus de publication, vous devrez à nouveau exécuter Status Monitor pour configurer Application Insights.

### <a name="what-telemetry-is-collected"></a>Quel type de télémétrie est collecté ?

Pour les applications que vous instrumentez uniquement au moment de l’exécution à l’aide de Status Monitor :

* Des requêtes HTTP
* Des appels de dépendances
* Exceptions
* Compteurs de performance

Pour les applications déjà instrumentées au moment de la compilation :

 * Des compteurs de processus.
 * Des appels de dépendances (.NET 4.5) ; des valeurs de retour dans des appels de dépendances (.NET 4.6).
 * Des valeurs d’arborescence des appels de procédure d’exception.

[En savoir plus](https://apmtips.com/posts/2016-11-18-how-application-insights-status-monitor-not-monitors-dependencies/)

## <a name="video"></a>Vidéo

> [!VIDEO https://channel9.msdn.com/events/Connect/2016/100/player]

## <a name="download-status-monitor"></a><a name="download"></a>Télécharger Status Monitor

- Utiliser le nouveau [module PowerShell](./status-monitor-v2-overview.md)
- Téléchargez et exécutez le [programme d’installation Status Monitor](https://go.microsoft.com/fwlink/?LinkId=506648).
- Ou exécutez [Web Platform Installer](https://www.microsoft.com/web/downloads/platform.aspx) et recherchez-y Application Insights Status Monitor.

## <a name="next-steps"></a><a name="next"></a>Étapes suivantes

Affichez vos données de télémétrie :

* [Explorez les mesures](../platform/metrics-charts.md) pour surveiller les performances et l’utilisation
* [Effectuez des recherches dans les événements et les journaux][diagnostic] pour diagnostiquer les problèmes
* [Utilisez la fonctionnalité Analytics](../log-query/log-query-overview.md) pour des requêtes plus élaborées

Ajoutez des données de télémétrie :

* [Créez des tests web][availability] pour vous assurer que votre site reste actif.
* [Ajoutez la télémétrie de client web][usage] pour afficher les exceptions à partir du code de la page web et vous permettre d’insérer un suivi des appels.
* [Ajoutez le kit de développement logiciel (SDK) Application Insights à votre code][greenbrown] afin de pouvoir insérer un suivi et des appels de journaux

<!--Link references-->

[api]: ./api-custom-events-metrics.md
[availability]: monitor-web-app-availability.md
[client]: ./javascript.md
[diagnostic]: ./diagnostic-search.md
[greenbrown]: ./asp-net.md
[qna]: ../faq.md
[roles]: ./resources-roles-access-control.md
[usage]: ./javascript.md