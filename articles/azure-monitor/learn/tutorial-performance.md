---
title: Diagnostiquer les problèmes de performances à l’aide d’Azure Application Insights | Microsoft Docs
description: Didacticiel vous permettant de rechercher et de diagnostiquer les problèmes de performances dans votre application à l’aide d’Azure Application Insights.
ms.subservice: application-insights
ms.topic: tutorial
author: lgayhardt
ms.author: lagayhar
ms.date: 06/15/2020
ms.custom: mvc
ms.openlocfilehash: df7cfff7d5bf1b89f88105f79c072d1d7e731b31
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96005484"
---
# <a name="find-and-diagnose-performance-issues-with-azure-application-insights"></a>Rechercher et diagnostiquer les problèmes de performances à l’aide d’Azure Application Insights

Azure Application Insights collecte des données de télémétrie à partir de votre application pour analyser son fonctionnement et ses performances.  Vous pouvez utiliser ces informations pour identifier les problèmes qui peuvent se produire ou pour identifier les améliorations apportées à l’application et qui ont le plus d’impact pour les utilisateurs.  Ce didacticiel vous guide dans le processus d’analyse des performances à la fois au niveau des composants serveur de votre application et du point de vue du client.  Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Identifier les performances des opérations côté serveur
> * Analyser les opérations du serveur pour déterminer la cause de la baisse des performances
> * Identifier les opérations les plus lentes côté client
> * Analyser les détails des affichages de page à l’aide du langage de requête


## <a name="prerequisites"></a>Prérequis

Pour suivre ce tutoriel :

- Installez [Visual Studio 2019](https://www.visualstudio.com/downloads/) avec les charges de travail suivantes :
    - Développement web et ASP.NET
    - Développement Azure
- Déployez une application .NET pour Azure et [activez le Kit SDK Application Insights](../app/asp-net.md).
- [Activer le profileur Application Insights](../app/profiler.md#installation) pour votre application.

## <a name="log-in-to-azure"></a>Connexion à Azure
Connectez-vous au portail Azure à l’adresse [https://portal.azure.com](https://portal.azure.com).

## <a name="identify-slow-server-operations"></a>Identifier les opérations lentes côté serveur
Application Insights collecte des informations sur les performances pour les différentes opérations de votre application. En identifiant ces opérations sur la plus longue période, vous pouvez diagnostiquer les problèmes potentiels ou mieux cibler votre développement actuel afin d’améliorer les performances globales de l’application.

1. Sélectionnez **Application Insights** choisissez votre abonnement.  
1. Pour ouvrir le panneau **Performances**, sélectionnez **Performances** sous le menu **Examiner**, ou cliquez sur le graphique **Temps de réponse du serveur**.

    ![Performances](media/tutorial-performance/1-overview.png)

2. Le panneau **Performances** affiche le nombre et la durée moyenne de chaque opération de l’application.  Vous pouvez utiliser ces informations pour identifier les opérations qui ont le plus d’impact sur les utilisateurs. Dans cet exemple, il peut être intéressant d’examiner les opérations **GET Customers/Details** et **GET Home/Index** en raison de leur durée relativement longue et du nombre d’appels.  D’autres opérations peuvent avoir une durée supérieure, mais elles sont rarement appelées et leur amélioration n’apporterait qu’un résultat négligeable.  

    ![Panneau du serveur de performances](media/tutorial-performance/2-server-operations.png)

3. Le graphique affiche actuellement la durée moyenne des opérations sélectionnées au fil du temps. Vous pouvez basculer vers le 95e centile pour rechercher les problèmes de performances. Ajoutez les opérations qui vous intéressent en les épinglant au graphique.  Cela indique qu’il existe des pics qui méritent un examen approfondi.  Isolez cet élément en réduisant la fenêtre de temps du graphique.

    ![Épingler des opérations](media/tutorial-performance/3-server-operations-95th.png)

4.  Le panneau de performances sur la droite affiche la distribution des durées pour différentes requêtes de l’opération sélectionnée.  Réduisez la fenêtre pour démarrer autour du 95e centile. La carte insights des « 3 principales dépendances » peut vous indiquer d’un coup d’œil que les dépendances externes contribuent probablement aux transactions lentes.  Cliquez sur le bouton avec le nombre d’exemples pour afficher la liste des exemples. Vous pouvez ensuite sélectionner un exemple quelconque pour afficher les détails de la transaction.

5.  Vous pouvez voir d’un coup d’œil que l’appel à la Table Azure Fabrikamaccount contribue le plus à la durée totale de la transaction. Vous pouvez également voir qu’une exception a provoqué son échec. Vous pouvez cliquer sur n’importe quel élément dans la liste pour afficher ses détails sur le côté droit. [En savoir plus sur l’expérience de diagnostic des transactions](../app/transaction-diagnostics.md)

    ![Informations de bout en bout de l’opération](media/tutorial-performance/4-end-to-end.png)
    

6.  Le [**Profileur**](../app/profiler-overview.md) vous aide à en apprendre davantage sur les diagnostics au niveau du code en affichant le code qui s’exécutait pour l’opération et le temps nécessaire à chaque étape. Certaines opérations peuvent ne pas avoir de suivi car le profileur s’exécute périodiquement.  Au fil du temps, d’autres opérations devraient avoir un suivi.  Pour démarrer le profileur pour l’opération, cliquez sur **Suivis du Profileur**.
5.  Le suivi affiche les événements individuels pour chaque opération pour vous permettre d’identifier la cause de la durée de l’opération globale.  Cliquez sur un des exemples en haut de la liste, dont la durée est la plus longue.
6.  Cliquez sur **Chemin réactif** pour mettre en surbrillance le chemin d’accès spécifique aux événements qui contribuent le plus à la durée totale de l’opération.  Dans cet exemple, vous pouvez remarquer que l’appel le plus lent provient de la méthode *FabrikamFiberAzureStorage.GetStorageTableData*. La partie qui prend le plus de temps est la méthode *CloudTable.CreateIfNotExist*. Si cette ligne de code est exécutée chaque fois que la fonction est appelée, les appels réseau inutiles et les ressources du processeur sont consommées. La meilleure façon de corriger votre code est de placer cette ligne dans une méthode de démarrage qui s’exécutera une seule fois.

    ![Détails du profileur](media/tutorial-performance/5-hot-path.png)

7.  Le **conseil relatif aux performances** affiché en haut de l’écran confirme l’hypothèse que la durée excessive est due à l’attente.  Cliquez sur le lien **en attente** pour plus d’informations sur l’interprétation les différents types d’événements.

    ![Conseil relatif aux performances](media/tutorial-performance/6-perf-tip.png)

8.   Pour une analyse plus approfondie, vous pouvez cliquer sur **Télécharger la trace** afin de télécharger la trace. Vous pouvez afficher ces données à l’aide de [PerfView](https://github.com/Microsoft/perfview#perfview-overview).

## <a name="use-logs-data-for-server"></a>Utiliser les données de journaux pour le serveur
 Logs fournit un langage de requête enrichi qui vous permet d’analyser toutes les données collectées par Application Insights. Vous pouvez utiliser ces informations pour effectuer une analyse approfondie des données concernant les requêtes et les performances.

1. Retournez au panneau des détails de l’opération et cliquez sur l’![icône Logs](media/tutorial-performance/app-viewinlogs-icon.png)**Afficher dans Logs (Analytics)**

2. Logs s’ouvre et affiche une requête pour chacune des vues dans le panneau.  Vous pouvez exécuter ces requêtes en tant que telles ou les modifier selon vos besoins.  La première requête indique la durée de cette opération au fil du temps.

    ![requête logs](media/tutorial-performance/7-request-time-logs.png)


## <a name="identify-slow-client-operations"></a>Identifier les opérations lentes côté client
En plus d’identifier les processus côté serveur à optimiser, Application Insights peut effectuer une analyse du point de vue des navigateurs clients.  Cela peut vous aider à identifier les améliorations potentielles des composants clients et même identifier les problèmes avec différents navigateurs ou emplacements.

1. Sélectionnez **Navigateur** sous **Examiner**, puis cliquez sur **Performances du navigateur** ou sélectionnez **Performances** sous **Examiner** et passez à l’onglet **Navigateur** en cliquant sur le bouton bascule serveur/navigateur dans le coin supérieur droit pour ouvrir le résumé des performances du navigateur. Vous obtenez un résumé visuel des différentes données de télémétrie de votre application du point de vue du navigateur.

    ![Résumé du navigateur](media/tutorial-performance/8-browser.png)

2. Sélectionnez l’un des noms d’opération puis cliquez sur le bouton d’exemples bleu dans le coin inférieur droit et sélectionnez une opération. Ceci affiche les détails de transaction de bout en bout, et vous pouvez consulter les **Propriétés des consultations de page** sur la droite. Ceci vous permet d’afficher les détails du client qui demande la page, y compris le type de navigateur et son emplacement. Ces informations peuvent vous aider à déterminer certains types de clients rencontrent des problèmes de performances.

    ![Affichage de la page](media/tutorial-performance/9-page-view-properties.png)

## <a name="use-logs-data-for-client"></a>Utiliser les données de journaux pour le client
Comme avec les données collectées pour les performances du serveur, Application Insights rend toutes les données du client disponibles pour une analyse approfondie à l’aide de Logs.

1. Retournez au résumé du navigateur, puis cliquez sur l’![icône Journaux](media/tutorial-performance/app-viewinlogs-icon.png) **Voir dans les journaux (analytique)**

2. Logs s’ouvre et affiche une requête pour chacune des vues dans le panneau. La première requête indique la durée de différents affichages de page au fil du temps.

    ![Requête Logs](media/tutorial-performance/10-page-view-logs.png)

## <a name="next-steps"></a>Étapes suivantes
Maintenant que vous avez appris à identifier les exceptions d’exécution, passez à l’étape suivante du didacticiel pour apprendre à créer des alertes en réponse aux défaillances.

> [!div class="nextstepaction"]
> [Alerter sur l’intégrité des applications](./tutorial-alert.md)

