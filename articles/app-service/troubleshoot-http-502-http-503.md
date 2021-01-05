---
title: Corriger les erreurs HTTP 502 et HTTP 503
description: Corriger les erreurs « 502 Passerelle incorrecte » et « 503 Service indisponible » dans une application hébergée sur Azure App Service
tags: top-support-issue
keywords: 502 Passerelle incorrecte, 503 Service indisponible, erreur 503, erreur 502
ms.assetid: 51cd331a-a3fa-438f-90ef-385e755e50d5
ms.topic: article
ms.date: 07/06/2016
ms.custom: seodec18
ms.openlocfilehash: 6dd67ff4f7346daf7a590ab4fb45a32d227f1c3e
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92147538"
---
# <a name="troubleshoot-http-errors-of-502-bad-gateway-and-503-service-unavailable-in-azure-app-service"></a>Corriger les erreurs HTTP « 502 Passerelle incorrecte » et « 503 Service indisponible » sur Azure App Service
Les erreurs « 502 Passerelle incorrecte » et « 503 Service indisponible » sont courantes dans les applications hébergées sur [Azure App Service](./overview.md). Cet article vous permet de résoudre ces erreurs.

Si vous avez besoin d'aide supplémentaire concernant n'importe quel point de cet article, contactez les experts Azure sur les [forums MSDN Azure et Stack Overflow](https://azure.microsoft.com/support/forums/). Vous pouvez également signaler un incident au support Azure. Accédez au [site de support Azure](https://azure.microsoft.com/support/options/) , puis cliquez sur **Obtenir un support**.

## <a name="symptom"></a>Symptôme
Lorsque vous accédez à l'application, celle-ci renvoie un message d'erreur HTTP « 502 Passerelle incorrecte » ou HTTP « 503 Service non disponible ».

## <a name="cause"></a>Cause
Ce problème est souvent dû à des problèmes au niveau de l’application, tels que :

* demandes exigeant beaucoup de temps ;
* un taux d’utilisation élevé de la mémoire/UC par l’application ;
* un blocage de l’application en raison d’une exception.

## <a name="troubleshooting-steps-to-solve-502-bad-gateway-and-503-service-unavailable-errors"></a>Étapes de dépannage pour résoudre les erreurs « 503 Service indisponible » et « 502 Passerelle incorrecte »
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

![Surveiller l'application pour corriger les erreurs HTTP « 502 Passerelle incorrecte » et « 503 Service indisponible »](./media/app-service-web-troubleshoot-HTTP-502-503/1-monitor-metrics.png)

Pour plus d'informations, consultez les pages suivantes :

* [Surveiller les applications dans Azure App Service](web-sites-monitor.md)
* [Réception de notifications d’alerte](../azure-monitor/platform/alerts-overview.md)

<a name="collect"></a>

### <a name="2-collect-data"></a>2. Collecter les données
#### <a name="use-the-diagnostics-tool"></a>Utiliser l’outil de diagnostic
App Service fournit une expérience interactive et intelligente pour vous aider à résoudre les problèmes liés à votre application sans configuration particulière. Si votre application rencontre des problèmes, l’outil de diagnostic vous en indique la nature afin de vous permettre d’accéder aux informations appropriées pour les résoudre plus facilement et plus rapidement.

Pour accéder aux diagnostics App Service, accédez à votre application App Service ou à votre environnement App Service dans le [portail Azure](https://portal.azure.com). Dans le volet de navigation de gauche, cliquez sur **Diagnostiquer et résoudre les problèmes**.

#### <a name="use-the-kudu-debug-console"></a>Utilisation de la console de débogage Kudu
App Service est fourni avec une console de débogage que vous pouvez utiliser pour déboguer, explorer et charger des fichiers, ainsi que des points de terminaison JSON pour obtenir des informations sur votre environnement. Pour votre application, il s'agit de la *console Kudu* ou du *tableau de bord SCM*.

Vous pouvez accéder à ce tableau de bord avec le lien **https://&lt;Your app name>.scm.azurewebsites.net/** .

Kudu fournit, entre autres, les éléments suivants :

* paramètres d’environnement pour votre application ;
* flux de journal ;
* vidage de diagnostic ;
* console de débogage dans laquelle vous pouvez exécuter les applets de commande Powershell et les commandes DOS de base.

Autre fonctionnalité utile de Kudu, dans le cas où votre application lève des exceptions de première chance, vous pouvez utiliser Kudu et l’outil Procdump de SysInternals pour créer des vidages de mémoire. Ces vidages de mémoire sont des instantanés du processus et peuvent souvent vous aider à résoudre les problèmes plus complexes rencontrés par votre application.

Pour plus d'informations sur les fonctionnalités disponibles dans Kudu, consultez [Outils en ligne de Sites Web Azure que vous devez connaître](https://azure.microsoft.com/blog/windows-azure-websites-online-tools-you-should-know-about/).

<a name="mitigate"></a>

### <a name="3-mitigate-the-issue"></a>3. Résoudre le problème
#### <a name="scale-the-app"></a>Mettre l’application à l’échelle
Dans Azure App Service, pour améliorer les performances et le débit, vous pouvez ajuster l’échelle à laquelle vous exécutez votre application. La mise à l’échelle d’une application implique deux actions associées : l’évolution de votre plan App Service vers un niveau tarifaire supérieur et la configuration de certains paramètres après le passage à ce niveau tarifaire supérieur.

Pour plus d’informations sur la mise à l’échelle, consultez [Mise à l’échelle des applications dans un environnement Azure App Service](manage-scale-up.md).

En outre, vous pouvez choisir d’exécuter votre application sur plusieurs instances. Non seulement cela vous offre plus de capacité de traitement, mais également un certain niveau de tolérance aux pannes. Si le processus s’arrête sur une instance, l’autre instance continue de servir les requêtes.

Vous pouvez définir la mise à l’échelle pour qu’elle soit manuelle ou automatique.

#### <a name="use-autoheal"></a>Utilisation de la correction automatique (AutoHeal)
La correction automatique (AutoHeal) recycle le processus de travail pour votre application en fonction des paramètres que vous choisissez (comme les modifications de configuration, les requêtes, les limites de mémoire ou le temps nécessaire pour exécuter une requête). La plupart du temps, le recyclage du processus est le moyen le plus rapide pour résoudre un problème. Même si vous pouvez toujours redémarrer l'application à partir du portail Azure, la fonctionnalité de correction automatique (AutoHeal) s'en charge automatiquement pour vous. Il vous suffit d’ajouter des déclencheurs dans le fichier web.config racine pour votre application. Notez que ces paramètres fonctionnent de la même façon même si votre application n’est pas une application .NET.

Pour plus d'informations, consultez [Correction automatique de Sites Web Azure](https://azure.microsoft.com/blog/auto-healing-windows-azure-web-sites/).

#### <a name="restart-the-app"></a>Redémarrer l’application
Il s’agit souvent du moyen le plus simple de résoudre des problèmes à usage unique. Sur le [portail Azure](https://portal.azure.com/), vous pouvez arrêter ou redémarrer votre application à partir du panneau de celle-ci.

 ![redémarrer l’application pour résoudre les erreurs HTTP de type « 502 Passerelle incorrecte » et « 503 Service indisponible »](./media/app-service-web-troubleshoot-HTTP-502-503/2-restart.png)

Vous pouvez également gérer votre application à l’aide d’Azure PowerShell. Pour plus d'informations, consultez [Utilisation d'Azure PowerShell avec le Gestionnaire de ressources Azure](../azure-resource-manager/management/manage-resources-powershell.md).