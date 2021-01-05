---
title: Outil de diagnostic et de résolution
description: Découvrez comment résoudre les problèmes liés à votre application dans Azure App Service grâce à l’outil de diagnostic et de résolution du portail Azure.
keywords: app service, azure app service, diagnostics, prise en charge, application web, dépannage, auto-assistance
author: jen7714
ms.topic: article
ms.date: 10/18/2019
ms.author: jennile
ms.custom: seodec18
ms.openlocfilehash: d58341979b0bbe0699a5ca293b20394c43cde1d1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88962806"
---
# <a name="azure-app-service-diagnostics-overview"></a>Présentation des diagnostics Azure App Service

Quand vous exécutez une application web, vous souhaitez anticiper tout problème, des erreurs 500 à l’arrêt de votre site. Avec les diagnostics App Service, vous pouvez résoudre les problèmes de votre application de manière intelligente et interactive sans effectuer de configuration particulière. Si vous rencontrez des problèmes avec votre application, les diagnostics App Service vous en indiquent la nature afin que vous disposiez des informations appropriées pour les résoudre plus facilement et plus rapidement.

Cette fonctionnalité est particulièrement utile quand vous rencontrez des problèmes avec votre application au cours des dernières 24 heures, mais vous pouvez aussi analyser tous les graphiques de diagnostic à tout moment.

Les diagnostics App Service fonctionnent non seulement pour votre application sur Windows, mais également pour les applications exécutées sous [Linux/dans des conteneurs](./overview.md#app-service-on-linux), sous [l’environnement App Service](./environment/intro.md) et dans [Azure Functions](../azure-functions/functions-overview.md).

## <a name="open-app-service-diagnostics"></a>Ouvrir les diagnostics App Service

Pour accéder aux diagnostics App Service, accédez à votre application App Service ou à votre environnement App Service dans le [Portail Azure](https://portal.azure.com). Dans le volet de navigation de gauche, cliquez sur **Diagnostiquer et résoudre les problèmes**.

Pour Azure Functions, accédez à votre application de fonction puis, dans le volet de navigation supérieur, cliquez sur **Fonctionnalités de plateforme** et sélectionnez **Diagnostiquer et résoudre les problèmes** dans la section **Gestion des ressources**.

Sur la page d’accueil des diagnostics App Service, vous pouvez choisir la catégorie qui décrit le mieux le problème rencontré avec votre application. Vous pouvez pour cela utiliser les mots clés dans chaque vignette de page d’accueil. C’est également sur cette page que vous trouverez les **Outils de diagnostic** pour les applications Windows. Consultez la section [Outils de diagnostic (uniquement pour une application Windows)](#diagnostic-tools-only-for-windows-app).

![Page d’accueil](./media/app-service-diagnostics/app-service-diagnostics-homepage-1.png)

> [!NOTE]
> Si votre application est en panne ou s’exécute lentement, vous pouvez [collecter une trace de profilage](https://azure.github.io/AppService/2018/06/06/App-Service-Diagnostics-Profiling-an-ASP.NET-Web-App-on-Azure-App-Service.html) pour identifier la cause racine du problème. Le profilage est léger et est conçu pour les scénarios de production.
>

## <a name="interactive-interface"></a>Interface interactive

Lorsque vous sélectionnez la catégorie de la page d’accueil correspondant le mieux au problème rencontré avec votre application, l’interface interactive des diagnostics App Service, Génie, peut vous aider à diagnostiquer et résoudre le problème. Vous pouvez utiliser les raccourcis de vignette fournis par Génie pour afficher le rapport de diagnostic complet de la catégorie de problème qui vous intéresse. Les raccourcis de vignette vous offrent un moyen d’accès direct aux métriques de diagnostic.

![Raccourcis de vignette](./media/app-service-diagnostics/tile-shortcuts-2.png)

Après avoir cliqué sur ces vignettes, vous pouvez voir une liste des rubriques relatives au problème décrit dans la vignette. Ces rubriques fournissent des extraits du rapport complet, offrant des informations pertinentes. Vous pouvez cliquer sur les rubriques de votre choix pour examiner les problèmes de façon plus approfondie. En outre, vous pouvez cliquer sur **Afficher le rapport complet** pour explorer toutes les rubriques sur une seule page.

![Rubriques](./media/app-service-diagnostics/application-logs-insights-3.png)

![Afficher le rapport complet](./media/app-service-diagnostics/view-full-report-4.png)

## <a name="diagnostic-report"></a>Rapport de diagnostic

Lorsque vous choisissez d’examiner le problème de façon plus approfondie en cliquant sur une rubrique, vous pouvez afficher plus de détails sur la rubrique, s’accompagnant souvent de graphiques et de fichiers Markdown. Le rapport de diagnostic peut représenter un outil puissant pour identifier le problème rencontré avec votre application.

![Rapport de diagnostic](./media/app-service-diagnostics/full-diagnostic-report-5.png)

## <a name="health-checkup"></a>Contrôle d’intégrité

Si vous ignorez ce qui ne va pas avec votre application ou que vous ne savez pas où démarrer la résolution de vos problèmes, le contrôle d’intégrité constitue un bon point de départ. Le contrôle d’intégrité analyse vos applications pour vous donner une vue d’ensemble rapide et interactive qui souligne ce qui est sain et ce qui ne va pas, vous indiquant des pistes de recherche pour mieux comprendre le problème. Son interface intelligente et interactive fournit des recommandations au fil du processus de résolution des problèmes. Le contrôle d’intégrité est intégré à l’expérience Génie pour les applications Windows et au rapport de diagnostic sur les applications web arrêtées pour les applications Linux.

### <a name="health-checkup-graphs"></a>Graphiques de contrôle d’intégrité

Le contrôle d’intégrité offre quatre graphiques différents.

- **Demandes et erreurs :** ce graphique affiche le nombre de demandes effectuées sur les dernières 24 heures, ainsi que les erreurs de serveur HTTP.
- **Performances de l’application :** ce graphique affiche le temps de réponse sur les dernières 24 heures pour différents groupes de centiles.
- **Utilisation de l’UC :** ce graphique affiche le pourcentage global d’utilisation de l’UC par instance sur les dernières 24 heures.  
- **Utilisation de la mémoire :** ce graphique affiche le pourcentage global d’utilisation de la mémoire physique par instance sur les dernières 24 heures.

![Contrôle d’intégrité](./media/app-service-diagnostics/health-checkup-6.png)

### <a name="investigate-application-code-issues-only-for-windows-app"></a>Examiner les problèmes de code d’application (uniquement pour une application Windows)

Dans la mesure où de nombreux problèmes d’application sont liés à des problèmes au niveau du code, les diagnostics App Service s’intègrent à [Application Insights](../azure-monitor/app/app-insights-overview.md) pour mettre en évidence les exceptions et les problèmes de dépendance qui peuvent être corrélés au temps d’arrêt sélectionné. Application Insights doit être activé séparément.

![Application Insights](./media/app-service-diagnostics/application-insights-7.png)

Pour afficher les dépendances et les exceptions d’Application Insights, sélectionnez les raccourcis de vignette **Web App Down** (Application web arrêtée) ou **Web App Slow** (Application web lente).

### <a name="troubleshooting-steps-only-for-windows-app"></a>Procédure de résolution des problèmes (uniquement pour une application Windows)

Si un problème appartenant à une catégorie de problèmes spécifique est détecté au cours des dernières 24 heures, vous pouvez afficher le rapport de diagnostic complet ; en consultant les diagnostics App Service, vous pouvez être invité à lire des conseils et à suivre des étapes de résolution des problèmes supplémentaires qui procurent une expérience plus interactive.

![Application Insights, résolution des problèmes et étapes suivantes](./media/app-service-diagnostics/troubleshooting-and-next-steps-8.png)

## <a name="diagnostic-tools-only-for-windows-app"></a>Outils de diagnostic (uniquement pour une application Windows)

Les Outils de diagnostics incluent des outils de diagnostic plus avancés qui vous aident à examiner les problèmes de code d’application, de lenteur, de chaînes de connexion, etc. Ils incluent également des outils proactifs qui vous aident à atténuer les problèmes liés à l’utilisation de l’UC, aux demandes et à la mémoire.

### <a name="proactive-cpu-monitoring"></a>Surveillance proactive de l’UC

La surveillance proactive de l’UC vous offre un moyen simple et proactif pour intervenir lorsque votre application ou le processus enfant de votre application consomme des ressources d’UC de façon intensive. Vous pouvez définir vos propres règles de seuil d’UC pour atténuer temporairement un problème d’utilisation intensive de l’UC jusqu’à ce que la véritable cause du problème inattendu soit identifiée. Pour plus d’informations, consultez [Atténuer vos problèmes d’UC avant qu’ils ne se produisent](https://azure.github.io/AppService/2019/10/07/Mitigate-your-CPU-problems-before-they-even-happen.html).

![Surveillance proactive de l’UC](./media/app-service-diagnostics/proactive-cpu-monitoring-9.png)

### <a name="auto-healing-and-proactive-auto-healing"></a>Réparation automatique et réparation automatique proactive

La réparation automatique est une action d’atténuation que vous pouvez entreprendre quand votre application présente un comportement inattendu. Vous pouvez définir vos propres règles selon le nombre de demandes, la lenteur des demandes, la limite de mémoire et le code d’état HTTP pour déclencher des actions d’atténuation. Utilisez l’outil pour atténuer temporairement les effets d’un comportement inattendu jusqu’à ce que vous trouviez la cause racine. Pour plus d’informations, consultez [Announcing the new auto healing experience in app service diagnostics](https://azure.github.io/AppService/2018/09/10/Announcing-the-New-Auto-Healing-Experience-in-App-Service-Diagnostics.html).

![Réparation automatique](./media/app-service-diagnostics/auto-healing-10.png)

Comme la supervision proactive du processeur, la réparation automatique proactive constitue une solution idéale pour atténuer un comportement inattendu de votre application. La réparation automatique proactive redémarre votre application quand App Service détermine que votre application est dans un état irrécupérable. Pour plus d’informations, consultez [Présentation de la répartition automatique proactive](https://azure.github.io/AppService/2017/08/17/Introducing-Proactive-Auto-Heal.html).

## <a name="navigator-and-change-analysis-only-for-windows-app"></a>Navigateur et analyse des modifications (uniquement pour les applications Windows)

Dans une grande équipe avec intégration continue où votre application a de nombreuses dépendances, il peut être difficile d’identifier la modification spécifique qui provoque un comportement anormal. Navigateur offre une bonne visibilité de la topologie de votre application en affichant automatiquement une carte des dépendances de votre application et toutes les ressources dans le même abonnement. Navigateur vous permet d’afficher une liste centralisée des modifications apportées par votre application et ses dépendances, et de vous focaliser sur une modification entraînant un comportement anormal. Vous pouvez y accéder par le biais de la vignette **Navigateur** dans la page d’accueil, et vous devez l’activer avant de l’utiliser pour la première fois. Pour plus d’informations, consultez [Get visibility into your app’s dependencies with Navigator](https://azure.github.io/AppService/2019/08/06/Bring-visibility-to-your-app-and-its-dependencies-with-Navigator.html).

![Page par défaut de Navigateur](./media/app-service-diagnostics/navigator-default-page-11.png)

![Affichage des différences](./media/app-service-diagnostics/diff-view-12.png)

L’analyse des modifications d’application est accessible par le biais des raccourcis de vignettes **Modifications d’application** et **Blocages de l’application**dans**Disponibilité et performances** ; ainsi, vous pouvez l’utiliser en même temps que d’autres métriques. Avant d’utiliser la fonctionnalité, vous devez l’activer. Pour plus d’informations, consultez [Announcing the new change analysis experience in App Service Diagnostics](https://azure.github.io/AppService/2019/05/07/Announcing-the-new-change-analysis-experience-in-App-Service-Diagnostics-Analysis.html).

Publiez vos questions ou vos commentaires sur [UserVoice](https://feedback.azure.com/forums/169385-web-apps) en ajoutant « [Diag] » dans le titre.