---
title: Résoudre les problèmes de détérioration des performances
description: Découvrez comment résoudre les problèmes de lenteur des applications dans Azure App Service, notamment en supervisant leur comportement, en collectant des données et en atténuant ces problèmes.
tags: top-support-issue
keywords: performances d'application web, application lente
ms.assetid: b8783c10-3a4a-4dd6-af8c-856baafbdde5
ms.topic: article
ms.date: 08/03/2016
ms.custom: seodec18
ms.openlocfilehash: cf8b0e1fda03a74d30ec77c911d705bf12cf0126
ms.sourcegitcommit: d6e92295e1f161a547da33999ad66c94cf334563
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96763806"
---
# <a name="troubleshoot-slow-app-performance-issues-in-azure-app-service"></a>Résoudre les problèmes de baisse de performances d’une application dans Azure App Service
Cet article vous aide à résoudre les problèmes de baisse de performances d’une application dans [Azure App Service](./overview.md).

Si vous avez besoin d'aide supplémentaire concernant n'importe quel point de cet article, contactez les experts Azure sur les [forums MSDN Azure et Stack Overflow](https://azure.microsoft.com/support/forums/). Vous pouvez également signaler un incident au support Azure. Accédez au [site de support Azure](https://azure.microsoft.com/support/options/) , puis cliquez sur **Obtenir un support**.

## <a name="symptom"></a>Symptôme
Lorsque vous parcourez l’application, le chargement des pages est lent et il est possible que le délai soit dépassé.

## <a name="cause"></a>Cause
Ce problème est souvent dû à des problèmes au niveau de l’application, tels que :

* requêtes réseau chronophages ;
* interrogations inefficaces du code ou de la base de données de l’application ;
* un taux d’utilisation élevé de la mémoire/UC par l’application ;
* blocage de l’application en raison d’une exception.

## <a name="troubleshooting-steps"></a>Étapes de dépannage
Le dépannage peut être divisé en trois tâches distinctes, dans un ordre séquentiel :

1. [Observer et contrôler le comportement de l’application](#observe)
2. [Collecter les données](#collect)
3. [Résoudre le problème](#mitigate)

À chaque étape, [App Service](overview.md) vous offre différentes options.

<a name="observe"></a>

### <a name="1-observe-and-monitor-application-behavior"></a>1. Observer et contrôler le comportement de l'application
#### <a name="track-service-health"></a>Suivi de l’état du service
Microsoft Azure publie chaque interruption du service et chaque dégradation des performances. Vous pouvez assurer le suivi de l’état du service sur le [portail Azure](https://portal.azure.com/). Pour plus d’informations, consultez la rubrique [Suivi de l’état du service](../service-health/service-notifications.md).

#### <a name="monitor-your-app"></a>Surveiller votre application
Cette option vous permet de savoir si votre application rencontre des problèmes. Dans le panneau de votre application, cliquez sur la vignette **Demandes et erreurs**. Le panneau **Mesure** affiche toutes les mesures que vous pouvez ajouter.

Voici quelques-unes des mesures disponibles pour surveiller votre application :

* Plage de travail moyenne de la mémoire
* Temps de réponse moyen
* Temps processeur
* Plage de travail de la mémoire
* Demandes

![surveiller les performances d’une application](./media/app-service-web-troubleshoot-performance-degradation/1-monitor-metrics.png)

Pour plus d'informations, consultez les pages suivantes :

* [Surveiller les applications dans Azure App Service](web-sites-monitor.md)
* [Réception de notifications d’alerte](../azure-monitor/platform/alerts-overview.md)

#### <a name="monitor-web-endpoint-status"></a>Surveillance de l'état d'un point de terminaison Web
Si vous exécutez votre application sur le niveau de tarification **Standard**, App Service vous permet de surveiller les deux points de terminaison à partir de trois régions géographiques.

Elle paramètre des tests Web à partir de différents emplacements où sont évalués les temps de réponse et de disponibilité des URL. Ce type de test lance une requête HTTP get sur l’URL Web pour déterminer les temps de réponse et de disponibilité depuis chaque emplacement. Chaque emplacement configuré exécute un test toutes les cinq minutes.

La disponibilité est contrôlée à l'aide des codes réponse HTTP et le temps de réponse est mesuré en millisecondes. Un test de surveillance échoue si le code de réponse HTTP est supérieur ou égal à 400 ou si la réponse prend plus de 30 secondes. Un point de terminaison est considéré comme disponible s'il passe avec succès les différents tests de surveillance à partir de tous les emplacements spécifiés.

Pour le configurer, voir [Surveillance des applications dans Azure App Service](web-sites-monitor.md).

Consultez également [Assurer la gestion des sites Web Azure et la surveillance des points de terminaison, avec Stefan Schackow](https://channel9.msdn.com/Shows/Azure-Friday/Keeping-Azure-Web-Sites-up-plus-Endpoint-Monitoring-with-Stefan-Schackow) pour obtenir une vidéo décrivant la surveillance d’un point de terminaison.

#### <a name="application-performance-monitoring-using-extensions"></a>Analyse des performances des applications à l’aide d’Extensions
Vous pouvez également analyser les performances de votre application à l’aide d’une *extension de site*.

Chaque application d’App Service fournit un point de terminaison de gestion extensible qui vous permet d’utiliser un ensemble performant d’outils déployés en tant qu’extensions de site. Les extensions incluent : 

- Éditeurs de code source tels que [Azure DevOps](https://www.visualstudio.com/products/what-is-visual-studio-online-vs.aspx). 
- Des outils de gestion pour les ressources connectées, comme une base de données MySQL connectée à une application.

[Azure Application Insights](https://azure.microsoft.com/services/application-insights/) est une extension de site d’analyse des performances qui est également disponible. Pour utiliser Application Insights, vous régénérez votre code avec un Kit de développement logiciel (SDK). Vous pouvez également installer une extension qui fournit l’accès à des données supplémentaires. Le Kit de développement logiciel (SDK) vous permet d'écrire un code pour surveiller plus en détail l'utilisation et les performances de votre application. Pour plus d’informations, consultez l’article [Analyse des performances dans les applications web](../azure-monitor/app/app-insights-overview.md).

<a name="collect"></a>

### <a name="2-collect-data"></a>2. Collecter les données
App Service fournit des fonctionnalités de diagnostic pour les informations de journalisation provenant du serveur Web et de l’application web. Ces informations sont réparties en diagnostics de serveur web et diagnostics d’application.

#### <a name="enable-web-server-diagnostics"></a>Activer les diagnostics de serveur web
Vous pouvez activer ou désactiver les types de journaux d’activité suivants :

* **Messages d’erreur détaillés** : informations d’erreur détaillées pour les codes d’état HTTP qui indiquent un échec (code d’état 400 ou supérieur). Il peut s'agir d'informations qui vous aident à déterminer la raison pour laquelle le serveur a renvoyé le code d'erreur.
* **Suivi des demandes ayant échoué** : informations détaillées sur les demandes qui ont échoué, y compris une trace des composants IIS utilisés pour traiter la demande et la durée dans chaque composant. Cela peut se révéler utile si vous essayez d’améliorer le fonctionnement des performances d’application ou d’isoler la cause d’une erreur HTTP spécifique.
* **Journalisation du serveur Web** : informations sur les transactions HTTP à l’aide du format de fichier journal étendu W3C. Ce rapport se révèle utile pour déterminer les mesures globales d’une application, comme le nombre de demandes traitées ou le nombre de demandes émanant d’une adresse IP spécifique.

#### <a name="enable-application-diagnostics"></a>Activer les diagnostics d’application
Il existe plusieurs façons de collecter des données de performances d’application à partir d’App Service, d’activer votre application à partir de Visual Studio, ou de modifier votre code d’application afin de consigner plus d’informations et de traces. Vous pouvez faire votre choix en fonction de l’accès à l’application et de vos observations sur les outils de surveillance.

##### <a name="use-application-insights-profiler"></a>Utiliser Application Insights Profiler
Vous pouvez activer Application Insights Profiler pour commencer à capturer des traces de performances détaillées. Vous pouvez accéder aux traces capturées jusqu’à cinq jours auparavant lorsque vous devez examiner les problèmes ayant eu lieu. Vous pouvez choisir cette option si vous avez accès à la ressource Application Insights de l’application sur le portail Azure.

Application Insights Profiler fournit des statistiques sur les temps de réponse de chaque appel web, et des traces qui indiquent la ligne de code responsable des réponses lentes. Parfois, l’application App Service est lente, car un code spécifique n’a pas été écrit de manière performante. Il peut s’agir par exemple d’un code séquentiel exécuté en parallèle et de conflits indésirables de verrouillage de base de données. Pour augmenter les performances de l’application, il faut supprimer ces goulots d’étranglement dans le code. Mais ceux-ci sont difficiles à détecter si vous ne configurez par des journaux d’activité et des suivis élaborés. Les traces collectées par Application Insights Profiler permettent d’identifier les lignes de code qui ralentissent l’application, et de résoudre ce problème pour les applications App Service.

 Pour plus d’informations, consultez [Profilage d’applications en direct dans Azure App Service avec Application Insights](../azure-monitor/app/profiler.md).

##### <a name="use-remote-profiling"></a>Utiliser le profilage distant
Dans Azure App Service, les applications web, les API, les back ends mobiles et les WebJobs peuvent être profilés à distance. Choisissez cette option si vous avez accès à la ressource d’application et si vous savez comment reproduire le problème, ou si vous connaissez l’intervalle de temps exact auquel le problème de performances se produit.

Le profilage à distance est pertinent si l’utilisation de l’UC du processus est élevée et si votre processus s’exécute plus lentement que prévu, ou si la latence des requêtes HTTP est supérieure à la normale : vous pouvez analyser à distance le profil de vos processus et faire en sorte que les piles d’appel d’exemple analysent l’activité du processus et les chemins actifs de code.

Pour plus d’informations, consultez [Prise en charge d’analyse de profil distant dans Azure App Service](https://azure.microsoft.com/blog/remote-profiling-support-in-azure-app-service).

##### <a name="set-up-diagnostic-traces-manually"></a>Configurer manuellement des traces de diagnostic
Si vous avez accès au code source de l’application web, le diagnostic d’application vous permet de recueillir des informations générées par une application web. Les applications ASP.NET peuvent utiliser la classe `System.Diagnostics.Trace` pour enregistrer des informations dans le journal de diagnostic d’application. Toutefois, vous devez modifier le code ou redéployer votre application. Cette méthode est recommandée si votre application s’exécute sur un environnement de test.

Pour obtenir des instructions détaillées sur la façon de configurer votre application de journalisation, consultez [Activer la journalisation des diagnostics pour les applications dans Azure App Service](troubleshoot-diagnostic-logs.md).

#### <a name="use-the-diagnostics-tool"></a>Utiliser l’outil de diagnostic
App Service fournit une expérience interactive et intelligente pour vous aider à résoudre les problèmes liés à votre application sans configuration particulière. Si votre application rencontre des problèmes, l’outil de diagnostic vous en indique la nature afin de vous permettre d’accéder aux informations appropriées pour les résoudre plus facilement et plus rapidement.

Pour accéder aux diagnostics App Service, accédez à votre application App Service ou à votre environnement App Service dans le [portail Azure](https://portal.azure.com). Dans le volet de navigation de gauche, cliquez sur **Diagnostiquer et résoudre les problèmes**.

#### <a name="use-the-kudu-debug-console"></a>Utilisation de la console de débogage Kudu
App Service est fourni avec une console de débogage que vous pouvez utiliser pour déboguer, explorer et charger des fichiers, ainsi que des points de terminaison JSON pour obtenir des informations sur votre environnement. Il s’agit de la *console Kudu* ou du *tableau de bord SCM* pour votre application.

Vous pouvez accéder à ce tableau de bord avec le lien **https://&lt;Your app name>.scm.azurewebsites.net/** .

Kudu fournit, entre autres, les éléments suivants :

* paramètres d’environnement pour votre application ;
* flux de journal ;
* vidage de diagnostic ;
* console de débogage dans laquelle vous pouvez exécuter les cmdlets PowerShell et les commandes DOS de base.

Autre fonctionnalité utile de Kudu, dans le cas où votre application lève des exceptions de première chance, vous pouvez utiliser Kudu et l’outil Procdump de SysInternals pour créer des vidages de mémoire. Ces vidages de mémoire sont des instantanés du processus et peuvent souvent vous aider à résoudre les problèmes plus complexes rencontrés par votre application.

Pour plus d'informations sur les fonctionnalités disponibles dans Kudu, voir [Outils Azure DevOps que vous devez connaître](https://azure.microsoft.com/blog/windows-azure-websites-online-tools-you-should-know-about/).

<a name="mitigate"></a>

### <a name="3-mitigate-the-issue"></a>3. Résoudre le problème
#### <a name="scale-the-app"></a>Mettre l’application à l’échelle
Dans Azure App Service, pour améliorer les performances et le débit, vous pouvez ajuster l’échelle à laquelle vous exécutez votre application. La mise à l’échelle d’une application implique deux actions associées : l’évolution de votre plan App Service vers un niveau tarifaire supérieur et la configuration de certains paramètres après le passage à ce niveau tarifaire supérieur.

Pour plus d’informations sur la mise à l’échelle, consultez [Mise à l’échelle des applications dans un environnement Azure App Service](manage-scale-up.md).

En outre, vous pouvez choisir d’exécuter votre application sur plusieurs instances. La montée en charge vous offre plus de capacité de traitement, mais également un certain niveau de tolérance aux pannes. Si le processus s’arrête sur une instance, les autres instances continuent à servir les requêtes.

Vous pouvez définir la mise à l’échelle pour qu’elle soit manuelle ou automatique.

#### <a name="use-autoheal"></a>Utilisation de la correction automatique (AutoHeal)
La correction automatique (AutoHeal) recycle le processus de travail pour votre application en fonction des paramètres que vous choisissez (comme les modifications de configuration, les requêtes, les limites de mémoire ou le temps nécessaire pour exécuter une requête). La plupart du temps, le recyclage du processus est le moyen le plus rapide pour résoudre un problème. Même si vous pouvez toujours redémarrer l’application directement dans le portail Azure, AutoHeal le fait automatiquement pour vous. Il vous suffit d’ajouter des déclencheurs dans le fichier web.config racine pour votre application. Ces paramètres fonctionnent de la même façon même si votre application n’est pas une application .NET.

Pour plus d'informations, consultez [Correction automatique de Sites Web Azure](https://azure.microsoft.com/blog/auto-healing-windows-azure-web-sites/).

#### <a name="restart-the-app"></a>Redémarrer l’application
Il s’agit souvent du moyen le plus simple de résoudre des problèmes ponctuels. Sur le [portail Azure](https://portal.azure.com/), vous pouvez arrêter ou redémarrer votre application à partir du panneau de celle-ci.

 ![redémarrer une application pour résoudre les problèmes de performances](./media/app-service-web-troubleshoot-performance-degradation/2-restart.png)

Vous pouvez également gérer votre application à l’aide d’Azure PowerShell. Pour plus d'informations, consultez [Utilisation d'Azure PowerShell avec le Gestionnaire de ressources Azure](../azure-resource-manager/management/manage-resources-powershell.md).