---
title: Utiliser le volet de paramètres d’Azure Application Insights Profiler | Microsoft Docs
description: Voir l’état de Profiler et démarrer des sessions de profilage
ms.topic: conceptual
author: cweining
ms.author: cweining
ms.date: 08/06/2018
ms.reviewer: mbullwin
ms.openlocfilehash: 9d9cc377ead0c297e8334d34255bd2c7c7cd39fc
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86499407"
---
# <a name="configure-application-insights-profiler"></a>Configurer Application Insights Profiler

## <a name="updated-profiler-agent"></a>Agent du profileur mis à jour
Les fonctionnalités de déclenchement fonctionnent uniquement à partir de la version 2.6 de l’agent du profileur. Si vous exécutez un service Azure App Service, votre agent est mis à jour automatiquement. Vous pouvez voir la version de l’agent en cours d’exécution en accédant à l’URL Kudu de votre web et en ajoutant \DiagnosticServices à la fin de l’URL comme ceci : `https://yourwebsite.scm.azurewebsites.net/diagnosticservices`. La tâche web Application Insights Profiler doit être en version 2.6 ou ultérieure. Vous pouvez forcer une mise à niveau en redémarrant votre application web. 

Si vous exécutez le profileur sur une machine virtuelle ou un service cloud, la version 16.0.4 de l’extension Diagnostics Azure pour Windows (WAD) ou une version plus récente doit être installée. Vous pouvez vérifier la version de WAD en vous connectant à votre machine virtuelle et en recherchant ce répertoire : C:\Packages\Plugins\Microsoft.Azure.Diagnostics.IaaSDiagnostics\1.16.0.4. Le nom du répertoire correspond à la version de WAD installée. L’agent de machine virtuelle Azure met à jour WAD automatiquement quand de nouvelles versions sont disponibles.

## <a name="profiler-settings-page"></a>Page des paramètres de Profiler

Pour ouvrir le volet des paramètres d’Azure Application Insights Profiler, accédez au volet Performances Application Insights, puis sélectionnez le bouton **Configurer Profiler**.

![Lien pour ouvrir la page des paramètres de Profiler][configure-profiler-entry]

Une page se présentant comme suit s’affiche :

![Page des paramètres de Profiler][configure-profiler-page]

La page **Configurer Application Insights Profiler** offre les fonctionnalités suivantes :

| Fonctionnalité | Description |
|-|-|
Profiler maintenant | Démarre des sessions de profilage pour toutes les applications qui sont liées à cette instance d’Application Insights.
Déclencheurs | Vous permet de configurer les déclencheurs entraînant l’exécution du profileur. 
Sessions de profilage récentes | Affiche des informations sur les sessions de profilage effectuées.

## <a name="profile-now"></a>Profiler maintenant
Cette option vous permet de démarrer une session de profilage à la demande. Quand vous cliquez sur ce lien, tous les agents du profileur qui envoient des données à cette instance d’Application Insights commencent à capturer un profil. Après 5 à 10 minutes, la session de profilage s’affiche dans la liste ci-dessous.

Pour déclencher manuellement une session du profileur, l’utilisateur doit au moins disposer d’un accès en « écriture » au composant Application Insights. Dans la plupart des cas, vous bénéficiez automatiquement de cet accès et vous n’avez rien d’autre à faire. En cas de problème, le rôle d’étendue d’abonnement à ajouter est « Contributeur de composant Application Insights ». [En savoir plus sur le contrôle d'accès par rôle avec Azure Monitor](./resources-roles-access-control.md).

## <a name="trigger-settings"></a>Paramètres de déclencheur
![Boîte de dialogue Paramètres de déclencheur][trigger-settings-flyout]

Cliquez sur le bouton Déclencheurs dans la barre de menus pour ouvrir la boîte de dialogue Paramètres de déclencheur. Vous pouvez configurer le déclencheur pour démarrer le profilage quand le pourcentage d’utilisation du processeur ou de la mémoire atteint le niveau que vous avez défini.

| Paramètre | Description |
|-|-|
Bouton Activé/Désactivé | Activé : le profileur peut être démarré avec ce déclencheur ; Désactivé : le profileur ne sera pas démarré avec ce déclencheur.
Seuil de mémoire | Quand l’utilisation de la mémoire atteint ce pourcentage, le profileur est démarré.
Duration | Définit la durée d’exécution du profileur quand il est déclenché.
Temps de recharge | Définit le temps d’attente du profileur avant de vérifier à nouveau l’utilisation de la mémoire ou du processeur après son déclenchement.

## <a name="recent-profiling-sessions"></a>Sessions de profilage récentes
Cette section de la page affiche des informations sur les sessions de profilage récentes. Une session de profilage représente la période pendant laquelle l’agent du profileur a effectué un profil sur l’une des machines hébergeant votre application. Vous pouvez ouvrir les profils à partir d’une session en cliquant sur l’une des lignes. Pour chaque session, les informations suivantes sont présentées :

| Paramètre | Description |
|-|-|
Déclenché par | Mode de démarrage de la session : par un déclencheur, avec le bouton Profiler maintenant ou avec l’échantillonnage par défaut. 
Nom de l'application | Nom de l’application profilée.
Instance de machine | Nom de la machine sur laquelle l’agent du profileur a été exécuté.
Timestamp | Heure à laquelle le profil a été capturé.
Traces | Nombre de traces associées à des requêtes individuelles.
% d’UC | Pourcentage d’utilisation du processeur pendant l’exécution du profileur.
Mémoire (%) | Pourcentage d’utilisation de la mémoire pendant l’exécution du profileur.

## <a name="use-web-performance-tests-to-generate-traffic-to-your-application"></a><a id="profileondemand"></a> Utiliser les tests de performances web pour générer du trafic vers votre application

Vous pouvez déclencher manuellement Profiler en un clic. Supposons que vous exécutez un test de performances web. Vous avez besoin des traces pour comprendre comment votre application web s’exécute avec une certaine charge. Pour savoir quand le test de charge est exécuté, vous avez besoin de contrôler le moment de capture des traces. Toutefois, l’intervalle d’échantillonnage aléatoire ne permet pas de le faire.

Les sections suivantes montrent comment ce scénario fonctionne :

### <a name="step-1-generate-traffic-to-your-web-app-by-starting-a-web-performance-test"></a>Étape 1 : Générer du trafic vers votre application web en démarrant un test de performances web

Si votre application web a déjà du trafic entrant ou si vous voulez simplement générer du trafic manuellement, ignorez cette section et continuez à l’étape 2.

1. Dans le portail Application Insights, sélectionnez **Configurer** > **Test de performances**. 

1. Pour démarrer un nouveau test de performances, sélectionnez le bouton **Nouveau**.

   ![Créer un test de performances][create-performance-test]

1. Dans le volet **Nouveau test de performance**, configurez l’URL cible du test. Acceptez tous les paramètres par défaut, puis sélectionnez **Exécuter le test** pour lancer l’exécution du test de charge.

    ![Configurer le test de charge][configure-performance-test]

    Le nouveau test est mis le premier en file d’attente, suivi de l’état *En cours*.

    ![Le test de charge est envoyé et placé en file d’attente][load-test-queued]

    ![Le test de charge est en cours d’exécution][load-test-in-progress]

### <a name="step-2-start-a-profiler-on-demand-session"></a>Étape 2 : Démarrer une session de Profiler à la demande

1. Quand le test de charge est en cours d’exécution, démarrez Profiler pour capturer les traces de l’application web pendant qu’elle reçoit la charge.

1. Accédez au volet **Configurer Profiler**.


### <a name="step-3-view-traces"></a>Étape 3 : Affichage des traces

Une fois que l’exécution de Profiler est terminée, suivez les instructions de la notification pour accéder au volet Performances et voir les traces.

## <a name="next-steps"></a>Étapes suivantes
[Activer Profiler et voir les traces](profiler-overview.md?toc=/azure/azure-monitor/toc.json)

[profiler-on-demand]: ./media/profiler-settings/Profiler-on-demand.png
[configure-profiler-entry]: ./media/profiler-settings/configure-profiler-entry.png
[configure-profiler-page]: ./media/profiler-settings/configureBlade.png
[trigger-settings-flyout]: ./media/profiler-settings/CPUTrigger.png
[create-performance-test]: ./media/profiler-settings/new-performance-test.png
[configure-performance-test]: ./media/profiler-settings/configure-performance-test.png
[load-test-queued]: ./media/profiler-settings/load-test-queued.png
[load-test-in-progress]: ./media/profiler-settings/load-test-inprogress.png
[enable-app-insights]: ./media/profiler-settings/enable-app-insights-blade-01.png
[update-site-extension]: ./media/profiler-settings/update-site-extension-01.png
[change-and-save-appinsights]: ./media/profiler-settings/change-and-save-appinsights-01.png
[app-settings-for-profiler]: ./media/profiler-settings/appsettings-for-profiler-01.png
[check-for-extension-update]: ./media/profiler-settings/check-extension-update-01.png
[profiler-timeout]: ./media/profiler-settings/profiler-timeout.png
