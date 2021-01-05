---
title: Profiler des applications Azure App Service en production avec Application Insights | Microsoft Docs
description: Profilez des applications en production sur Azure App Service avec Application Insights Profiler.
ms.topic: conceptual
author: cweining
ms.author: cweining
ms.date: 08/06/2018
ms.reviewer: mbullwin
ms.openlocfilehash: 41aec98c2a8776ae128389679ad0acb324596ec4
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96546826"
---
# <a name="profile-live-azure-app-service-apps-with-application-insights"></a>Profiler des applications Azure App Service en production avec Application Insights

Vous pouvez exécuter Profiler sur des applications ASP.NET et ASP.NET Core qui s’exécutent sur Azure App Service avec le niveau de service De base ou un niveau supérieur. À l’heure actuelle, l’activation de Profiler sur Linux est uniquement possible via [cette méthode](profiler-aspnetcore-linux.md).

## <a name="enable-profiler-for-your-app"></a><a id="installation"></a> Activer Profiler pour votre application
Pour activer Profiler pour une application, suivez les instructions ci-dessous. Si vous exécutez un autre type de service Azure, voici les instructions permettant d’activer Profiler sur d’autres plateformes prises en charge :
* [Cloud Services](./profiler-cloudservice.md?toc=%2fazure%2fazure-monitor%2ftoc.json)
* [Applications Service Fabric](./profiler-servicefabric.md?toc=%2fazure%2fazure-monitor%2ftoc.json)
* [Machines virtuelles](./profiler-vm.md?toc=%2fazure%2fazure-monitor%2ftoc.json)

Application Insights Profiler est préinstallé dans le cadre de l’exécution d’App Service. Les étapes ci-dessous vous montrent comment l’activer pour votre App Service. Suivez ces étapes même si vous avez inclus le SDK App Insights dans votre application au moment de la génération.

1. Accédez au panneau de configuration Azure pour votre App Service.
1. Activez le paramètre « Always On » pour votre service d’application. Pour accéder à ce paramètre, allez dans la page **Paramètres**, **Configuration** (voir la capture d’écran de l’étape suivante), puis cliquez sur l’onglet **Paramètres généraux**.
1. Accédez à la page **Paramètres > Application Insights**.

   ![Activer App Insights sur le portail App Services](./media/profiler/AppInsights-AppServices.png)

1. Suivez les instructions dans le volet pour créer une ressource ou sélectionnez une ressource App Insights existante pour superviser votre application. Vérifiez également que le profileur est **activé**. Si votre ressource Application Insights se trouve dans un autre abonnement que votre App Service, vous ne pouvez pas utiliser cette page pour configurer Application Insights. Toutefois, vous pouvez toujours le faire manuellement en créant manuellement les paramètres d’application nécessaires. [La section suivante contient des instructions permettant d’activer manuellement Profiler.](#enable-profiler-manually-or-with-azure-resource-manager) 

   ![Ajouter l’extension de site Application Insights][Enablement UI]

1. Le profileur est maintenant activé à l’aide d’un paramètre d’application App Services.

    ![Paramètre d’application pour Profiler][profiler-app-setting]

## <a name="enable-profiler-manually-or-with-azure-resource-manager"></a>Activer Profiler manuellement ou avec Azure Resource Manager
Application Insights Profiler peut être activé en créant des paramètres d’application pour votre Azure App Service. La page avec les options affichées ci-dessus crée automatiquement ces paramètres d’application. Mais vous pouvez automatiser la création de ces paramètres à l’aide d’un modèle ou d’autres moyens. Ces paramètres fonctionnent également si votre ressource Application Insights se trouve dans un autre abonnement que votre Azure App Service.
Voici les paramètres nécessaires pour activer le profileur :

|Paramètre d'application    | Valeur    |
|---------------|----------|
|APPINSIGHTS_INSTRUMENTATIONKEY         | iKey pour votre ressource Application Insights    |
|APPINSIGHTS_PROFILERFEATURE_VERSION | 1.0.0 |
|DiagnosticServices_EXTENSION_VERSION | ~3 |


Vous pouvez définir ces valeurs à l’aide de [modèles Azure Resource Manager](./azure-web-apps.md#app-service-application-settings-with-azure-resource-manager), d’[Azure PowerShell](/powershell/module/az.websites/set-azwebapp) ou d’[Azure CLI](/cli/azure/webapp/config/appsettings?view=azure-cli-latest).

### <a name="enabling-profiler-for-other-clouds-manually"></a>Activation manuelle de Profiler pour d’autres clouds

Si vous souhaitez activer Profiler pour d’autres clouds, utilisez les paramètres d’application ci-dessous.

|Paramètre d'application    | US Government Values| China Cloud |   
|---------------|---------------------|-------------|
|ApplicationInsightsProfilerEndpoint         | `https://profiler.monitor.azure.us`    | `https://profiler.monitor.azure.cn` |
|ApplicationInsightsEndpoint | `https://dc.applicationinsights.us` | `https://dc.applicationinsights.azure.cn` |

## <a name="disable-profiler"></a>Désactiver Profiler

Pour arrêter ou redémarrer Profiler pour l’instance d’une application individuelle, sous **WebJobs**, arrêtez ou redémarrez la tâche web nommée ApplicationInsightsProfiler3. Même si le profileur est désactivé à l’aide du commutateur sur la page Application Insights comme décrit ci-dessus, le processus du profileur s’exécutera toujours. Le profileur vérifiera s’il est activé. S’il est désactivé, il passe en mode veille pendant un certain temps avant de vérifier à nouveau. Il n’effectue aucun profilage s’il est désactivé. Si vous désactivez cette tâche web, le processus du profileur ne s’exécutera pas du tout, même pour vérifier s’il est activé.

  ![Désactiver Profiler pour une tâche web][disable-profiler-webjob]

Nous vous recommandons d’activer dès que possible Profiler sur toutes vos applications afin de découvrir d’éventuels problèmes de performance.

Les fichiers de Profiler peuvent être supprimés quand vous utilisez WebDeploy pour déployer des changements sur votre application web. Vous pouvez empêcher la suppression en excluant le dossier App_Data des éléments étant supprimés au cours du déploiement. 


## <a name="next-steps"></a>Étapes suivantes

* [Utilisation d’Application Insights dans Visual Studio](./visual-studio.md)

[Enablement UI]: ./media/profiler/Enablement_UI.png
[profiler-app-setting]:./media/profiler/profiler-app-setting.png
[disable-profiler-webjob]: ./media/profiler/disable-profiler-webjob.png

