---
title: Questions fréquentes (FAQ) sur les performances des applications
description: Obtenez des réponses aux questions fréquentes sur la disponibilité, les performances et les problèmes d’applications dans Azure App Service.
author: genlin
manager: dcscontentpm
tags: top-support-issue
ms.assetid: 2fa5ee6b-51a6-4237-805f-518e6c57d11b
ms.topic: article
ms.date: 10/31/2018
ms.author: genli
ms.custom: seodec18
ms.openlocfilehash: dfaeee6a6e2a9728d7e63fb5681c487fbbd6139e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88958995"
---
# <a name="application-performance-faqs-for-web-apps-in-azure"></a>FAQ sur les performances des applications web Azure dans Azure

> [!NOTE]
> Certaines instructions ci-dessous fonctionnent uniquement sur les App Services Windows ou Linux. Par exemple, les App Services Linux s’exécutent en mode 64 bits par défaut.
>

Cet article fournit des réponses aux questions fréquemment posées (FAQ) sur les problèmes de performances des applications pour la [fonctionnalité Web Apps d’Azure App Service](https://azure.microsoft.com/services/app-service/web/).

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="why-is-my-app-slow"></a>Pourquoi mon application est-elle lente ?

Plusieurs facteurs peuvent ralentir les performances d’une application. Pour obtenir des étapes de dépannage détaillées, consultez [Résoudre les problèmes de lenteur d’une application web](troubleshoot-performance-degradation.md).

## <a name="how-do-i-troubleshoot-a-high-cpu-consumption-scenario"></a>Comment puis-je résoudre un problème de consommation excessive du processeur ?

Dans certaines situations de consommation excessive du processeur, votre application peut vraiment nécessiter davantage de ressources informatiques. Dans ce cas, vous pouvez évoluer vers un niveau de service supérieur afin que l’application obtienne toutes les ressources dont elle a besoin. Dans d’autres cas, une consommation excessive du processeur peut être due à une boucle incorrecte ou à une méthode de codage. Obtenir un aperçu de ce qui déclenche une consommation excessive du processeur est un processus en deux parties. Créez d’abord un vidage du processus puis analysez ce vidage de processus. Pour plus d’informations, consultez [Capturer et analyser un fichier de vidage en cas de consommation excessive du processeur pour Web Apps](/archive/blogs/asiatech/how-to-capture-dump-when-intermittent-high-cpu-happens-on-azure-web-app).

## <a name="how-do-i-troubleshoot-a-high-memory-consumption-scenario"></a>Comment résoudre les problèmes de consommation excessive de la mémoire ?

Dans certaines situations de consommation excessive de mémoire, votre application peut vraiment nécessiter davantage de ressources informatiques. Dans ce cas, vous pouvez évoluer vers un niveau de service supérieur afin que l’application obtienne toutes les ressources dont elle a besoin. À d’autres moments, un bogue dans le code peut entraîner une fuite de mémoire. Une pratique de codage peut également augmenter la consommation de mémoire. Obtenir un aperçu de ce qui déclenche une consommation excessive de mémoire est un processus en deux parties. Créez d’abord un vidage du processus puis analysez ce vidage de processus. L’outil Crash Diagnoser d’Azure Site Extension Gallery peut effectuer efficacement ces deux étapes. Pour plus d’informations, consultez [Capturer et analyser un fichier de vidage en cas de consommation excessive intermittente de mémoire pour Web Apps](/archive/blogs/asiatech/how-to-capture-and-analyze-dump-for-intermittent-high-memory-on-azure-web-app).

## <a name="how-do-i-automate-app-service-web-apps-by-using-powershell"></a>Comment automatiser les applications web App Service à l’aide de PowerShell ?

Vous pouvez utiliser les applets de commande PowerShell pour gérer et mettre à jour des applications web App Service. Dans notre billet de blog [Automatiser des applications web hébergées dans Azure App Service à l’aide de PowerShell](/archive/blogs/puneetgupta/automating-webapps-hosted-in-azure-app-service-through-powershell-arm-way), nous expliquons comment utiliser des applets de commande PowerShell basée sur Azure Resource Manager pour automatiser des tâches courantes. Ce billet de blog fournit également un exemple de code pour diverses tâches de gestion des applications web. Pour des descriptions et la syntaxe de toutes les applets de commande des applications web App Service, consultez [Az.Websites](/powershell/module/az.websites).

## <a name="how-do-i-view-my-web-apps-event-logs"></a>Comment afficher les journaux des événements de mon application web ?

Pour afficher les journaux des événements de votre application web :

1. Connectez-vous à votre **site web Kudu** (`https://*yourwebsitename*.scm.azurewebsites.net`).
2. Dans le menu, sélectionnez **Console de débogage** > **CMD**.
3. Sélectionnez le dossier **LogFiles**.
4. Pour afficher les journaux des événements, sélectionnez l’icône en forme de crayon en regard de **eventlog.xml**.
5. Pour télécharger les journaux d’activité, exécutez l’applet de commande PowerShell `Save-AzureWebSiteLog -Name webappname`.

## <a name="how-do-i-capture-a-user-mode-memory-dump-of-my-web-app"></a>Comment capturer un vidage de mémoire en mode utilisateur de mon application web ?

Pour capturer un vidage de mémoire en mode utilisateur de votre application web :

1. Connectez-vous à votre **site web Kudu** (`https://*yourwebsitename*.scm.azurewebsites.net`).
2. Sélectionnez le menu **Explorateur de processus**.
3. Cliquez avec le bouton droit sur le menu **w3wp.exe** ou sur votre processus WebJob.
4. Sélectionnez **Télécharger le vidage de mémoire** > **Vidage complet**.

## <a name="how-do-i-view-process-level-info-for-my-web-app"></a>Comment afficher les informations au niveau processus pour mon application web ?

Vous avez deux options pour afficher les informations au niveau processus pour votre application web :

*   Dans le portail Azure :
    1. Ouvrez l **’Explorateur de processus** pour l’application web.
    2. Pour afficher les détails, sélectionnez le processus **w3wp.exe**.
*   Dans la console Kudu :
    1. Connectez-vous à votre **site web Kudu** (`https://*yourwebsitename*.scm.azurewebsites.net`).
    2. Sélectionnez le menu **Explorateur de processus**.
    3. Pour le processus **w3wp.exe**, sélectionnez **Propriétés**.

## <a name="when-i-browse-to-my-app-i-see-error-403---this-web-app-is-stopped-how-do-i-resolve-this"></a>Lorsque j’accède à mon application, le message « Erreur 403 - Cette application web est arrêtée. » s’affiche. Comment résoudre ce problème ?

Trois conditions peuvent provoquer cette erreur :

* L’application web a atteint une limite de facturation et votre site a été désactivé.
* L’application web a été arrêtée dans le portail.
* L’application web a atteint une limite de quota de ressources qui peut s’appliquer à un plan de service de mise à l’échelle gratuit ou partagé.

Pour voir ce qui provoque l’erreur et résoudre le problème, suivez les étapes de la rubrique [Web Apps : « Erreur 403 - Cette application web est arrêtée »](/archive/blogs/waws/azure-web-apps-error-403-this-web-app-is-stopped).

## <a name="where-can-i-learn-more-about-quotas-and-limits-for-various-app-service-plans"></a>Où puis-je obtenir plus d’informations sur les quotas et les limites des différents plans App Service ?

Pour plus d’informations sur les quotas et les limites, consultez [Limites App Service](../azure-resource-manager/management/azure-subscription-service-limits.md#app-service-limits). 

## <a name="how-do-i-decrease-the-response-time-for-the-first-request-after-idle-time"></a>Comment réduire les temps de réponse de la première requête après une période d’inactivité ?

Par défaut, les applications web sont déchargées si elles restent inactives pendant un laps de temps défini. De cette manière, le système peut économiser les ressources. L’inconvénient est que la réponse à la première requête une fois que l’application web est déchargée est plus longue, pour permettre à l’application web de charger puis de démarrer le traitement des réponses. Dans les plans de service De base et Standard, vous pouvez activer le paramètre **Always On** afin que l’application reste toujours chargée. Cela raccourcit les délais de chargement une fois que l’application est inactive. Pour modifier le paramètre **Always On** :

1. Dans le portail Azure, accédez à votre application web.
2. Sélectionnez **Configuration**.
3. Sélectionnez **Paramètres généraux**.
4. Pour **Always On**, sélectionnez **On**.

## <a name="how-do-i-turn-on-failed-request-tracing"></a>Comment activer le suivi des demandes ayant échoué ?

Pour activer le suivi des demandes ayant échoué :

1. Dans le portail Azure, accédez à votre application web.
3. Sélectionnez **Tous les paramètres** > **Journaux de diagnostic**.
4. Pour **Suivi des demandes ayant échoué**, sélectionnez **On**.
5. Sélectionnez **Enregistrer**.
6. Dans le panneau des applications web, sélectionnez **Outils**.
7. Sélectionnez **Visual Studio Online**.
8. Si le paramètre n’est pas défini sur **On**, sélectionnez **On**.
9. Sélectionnez **Go**.
10. Sélectionnez **Web.config**.
11. Dans system.webServer, ajoutez cette configuration (pour capturer une URL spécifique) :

    ```xml
    <system.webServer>
    <tracing> <traceFailedRequests>
    <remove path="*api*" />
    <add path="*api*">
    <traceAreas>
    <add provider="ASP" verbosity="Verbose" />
    <add provider="ASPNET" areas="Infrastructure,Module,Page,AppServices" verbosity="Verbose" />
    <add provider="ISAPI Extension" verbosity="Verbose" />
    <add provider="WWW Server" areas="Authentication,Security,Filter,StaticFile,CGI,Compression, Cache,RequestNotifications,Module,FastCGI" verbosity="Verbose" />
    </traceAreas>
    <failureDefinitions statusCodes="200-999" />
    </add> </traceFailedRequests>
    </tracing>
    ```
12. Pour résoudre les problèmes de baisse des performances, ajoutez cette configuration (si la demande de capture prend plus de 30 secondes) :
    ```xml
    <system.webServer>
    <tracing> <traceFailedRequests>
    <remove path="*" />
    <add path="*">
    <traceAreas> <add provider="ASP" verbosity="Verbose" />
    <add provider="ASPNET" areas="Infrastructure,Module,Page,AppServices" verbosity="Verbose" />
    <add provider="ISAPI Extension" verbosity="Verbose" />
    <add provider="WWW Server" areas="Authentication,Security,Filter,StaticFile,CGI,Compression, Cache,RequestNotifications,Module,FastCGI" verbosity="Verbose" />
    </traceAreas>
    <failureDefinitions timeTaken="00:00:30" statusCodes="200-999" />
    </add> </traceFailedRequests>
    </tracing>
    ```
13. Pour télécharger les traces de demandes ayant échoué, dans le [portail](https://portal.azure.com), accédez à votre site web.
15. Sélectionnez **Outils** > **Kudu** > **Go**.
18. Dans le menu, sélectionnez **Console de débogage** > **CMD**.
19. Sélectionnez le dossier **LogFiles** puis le dossier dont le nom commence par **W3SVC**.
20. Pour afficher le fichier XML, sélectionnez l’icône en forme de crayon.

## <a name="i-see-the-message-worker-process-requested-recycle-due-to-percent-memory-limit-how-do-i-address-this-issue"></a>Le message « Worker Process requested recycle due to 'Percent Memory' limit » (un processus de travail a demandé un recyclage en raison d’une limite de pourcentage de mémoire) s’affiche. Comment pour résoudre ce problème ?

La quantité maximale de mémoire disponible pour un processus 32 bits (même sur un système d’exploitation 64 bits) est de 2 Go. Par défaut, le processus de travail est défini sur 32 bits dans App Service (à des fins de compatibilité avec les applications web héritées).

Vous pouvez basculer vers des processus 64 bits pour tirer parti de la mémoire supplémentaire disponible de votre rôle de travail web. Comme cette opération déclenche un redémarrage de l’application web, vous devez la planifier en conséquence.

Notez également qu’un environnement 64 bits nécessite un plan de service De base ou Standard. Les plans Gratuit et Partagé s’exécutent toujours dans un environnement 32 bits.

Pour plus d’informations, consultez [Configurer des applications web dans App Service](configure-common.md).

## <a name="why-does-my-request-time-out-after-230-seconds"></a>Pourquoi ma demande expire-t-elle au bout de 230 secondes ?

L’équilibrage de charge Azure comporte un paramètre de délai d’inactivité par défaut de quatre minutes. Il s’agit généralement d’un délai de réponse raisonnable pour une demande web. Si votre application web requiert un traitement en arrière-plan, nous vous recommandons d’utiliser des tâches Azure WebJob. L’application web Azure peut appeler des tâches WebJob et être avertie lorsque le traitement en arrière-plan est terminé. Vous pouvez choisir parmi plusieurs méthodes pour utiliser des tâches WebJob, y compris des files d’attente et des déclencheurs.

Les tâches WebJob sont conçues pour le traitement en arrière-plan. Vous pouvez effectuer autant de traitement en arrière-plan que vous le souhaitez dans une tâche WebJob. Pour plus d’informations sur les tâches WebJob, consultez [Exécuter des tâches en arrière-plan avec des tâches WebJob](webjobs-create.md).

## <a name="aspnet-core-applications-that-are-hosted-in-app-service-sometimes-stop-responding-how-do-i-fix-this-issue"></a>Les applications ASP.NET Core hébergées dans App Service cessent parfais de répondre. Comment résoudre ce problème ?

Un problème connu avec une [version de Kestrel](https://github.com/aspnet/KestrelHttpServer/issues/1182) antérieure peut amener une application ASP.NET Core 1.0 hébergée dans App Service à cesser de répondre par intermittence. Le message suivant peut également apparaître : « L’application CGI spécifiée a rencontré une erreur et le serveur a mis fin au processus. »

Ce problème est résolu dans Kestrel version 1.0.2. Cette version est incluse dans la mise à jour ASP.NET Core 1.0.3. Pour résoudre ce problème, veillez à mettre à jour les dépendances de votre application afin d’utiliser Kestrel 1.0.2. Vous pouvez également utiliser une des deux solutions de contournement décrites dans le billet de blog [ASP.NET Core 1.0 slow perf issues in App Service web apps](/archive/blogs/waws/asp-net-core-slow-perf-issues-on-azure-websites).


## <a name="i-cant-find-my-log-files-in-the-file-structure-of-my-web-app-how-can-i-find-them"></a>Impossible de trouver mes fichiers journaux dans la structure de fichiers de mon application web. Comment puis-je les trouver ?

La fonctionnalité Cache local d’App Service affecte la structure des dossiers LogFiles et Data de votre instance App Service. Lorsque la fonctionnalité Cache local est utilisée, des sous-dossiers sont créés dans les dossiers de données LogFiles et Data. Ces sous-dossiers utilisent le même modèle d’affectation de noms « identificateur unique » + horodatage. Chaque sous-dossier correspond à une instance de machine virtuelle dans laquelle l’application web est ou a été exécutée.

Pour déterminer si vous utilisez Cache local, ouvrez l’onglet **Paramètres d’application** d’App Service. Si la fonctionnalité Cache local est utilisée, le paramètre d’application `WEBSITE_LOCAL_CACHE_OPTION` est défini sur `Always`.

Si vous n’utilisez pas la fonctionnalité Cache local et que vous rencontrez ce problème, envoyez une demande de support.

## <a name="i-see-the-message-an-attempt-was-made-to-access-a-socket-in-a-way-forbidden-by-its-access-permissions-how-do-i-resolve-this"></a>Le message « Tentative d'accès à un socket de manière interdite par ses autorisations d'accès » s’affiche. Comment résoudre ce problème ?

Cette erreur se produit généralement si les connexions TCP sortantes sur l’instance de la machine virtuelle sont épuisées. Dans App Service, des limites sont appliquées au le nombre maximal de connexions sortantes possibles pour chaque instance de machine virtuelle. Pour plus d’informations, consultez [Limites numériques entre les machines virtuelles](https://github.com/projectkudu/kudu/wiki/Azure-Web-App-sandbox#cross-vm-numerical-limits).

Cette erreur peut également se produire si vous essayez d’accéder à une adresse locale à partir de votre application. Pour plus d’informations, consultez [Demandes d’adresses locales](https://github.com/projectkudu/kudu/wiki/Azure-Web-App-sandbox#local-address-requests).

Pour plus d’informations sur les connexions sortantes dans votre application web, consultez le billet de blog consacré aux [connexions sortantes vers des sites web Azure](https://www.freekpaans.nl/2015/08/starving-outgoing-connections-on-windows-azure-web-sites/).

## <a name="how-do-i-use-visual-studio-to-remote-debug-my-app-service-web-app"></a>Comment utiliser Visual Studio pour déboguer à distance mon application web App Service ?

Pour obtenir une procédure détaillée montrant comment déboguer votre application web à l’aide de Visual Studio, consultez [Déboguer à distance votre application web App Service](/archive/blogs/benjaminperkins/remote-debug-your-azure-app-service-web-app).