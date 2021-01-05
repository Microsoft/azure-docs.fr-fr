---
title: Diagnostiquer des exceptions runtime à l’aide d’Azure Application Insights | Microsoft Docs
description: Didacticiel décrivant comment rechercher et diagnostiquer des exceptions runtime dans votre application à l’aide d’Azure Application Insights.
ms.subservice: application-insights
ms.topic: tutorial
author: mrbullwinkle
ms.author: mbullwin
ms.date: 09/19/2017
ms.custom: mvc
ms.openlocfilehash: 057b211179c6932d4214e6118e3fa97b95145ba0
ms.sourcegitcommit: dd45ae4fc54f8267cda2ddf4a92ccd123464d411
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92926629"
---
# <a name="find-and-diagnose-run-time-exceptions-with-azure-application-insights"></a>Rechercher et diagnostiquer des exceptions runtime avec Azure Application Insights

Azure Application Insights collecte la télémétrie de votre application pour identifier et diagnostiquer des exceptions runtime.  Ce didacticiel vous guide dans ce processus avec votre application.  Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Modifier votre projet pour activer le suivi des exceptions
> * Identifier des exceptions pour les différents composants de votre application
> * Afficher les détails d’une exception
> * Télécharger une capture instantanée de l’exception vers Visual Studio à des fins de débogage
> * Analyser les détails des demandes ayant échoué à l’aide du langage de requête
> * Créer un élément de travail pour corriger le code défectueux


## <a name="prerequisites"></a>Conditions préalables requises

Pour suivre ce tutoriel :

- Installez [Visual Studio 2019](https://www.visualstudio.com/downloads/) avec les charges de travail suivantes :
    - Développement web et ASP.NET
    - Développement Azure
- Téléchargez et installez le [débogueur d'instantané de Visual Studio](https://aka.ms/snapshotdebugger).
- Activez le [débogueur de capture instantanée de Visual Studio](../app/snapshot-debugger.md).
- Déployez une application .NET pour Azure et [activez le Kit SDK Application Insights](../app/asp-net.md). 
- Le didacticiel effectuant le suivi de l’identification d’une exception dans votre application, modifiez le code dans votre environnement de développement ou de test afin de générer une exception. 

## <a name="log-in-to-azure"></a>Connexion à Azure
Connectez-vous au portail Azure à l’adresse [https://portal.azure.com](https://portal.azure.com).


## <a name="analyze-failures"></a>Analyser les échecs
Application Insights collecte les échecs dans votre application et vous permet d’afficher leur fréquence dans différentes opérations afin de vous aider à concentrer vos efforts sur les échecs dont l’impact est le plus élevé.  Vous pouvez explorer les détails de ces échecs pour en identifier la cause racine.   

1. Sélectionnez **Application Insights**, puis votre abonnement.  
2. Pour ouvrir le panneau **Échecs**, sélectionnez **Échecs** dans le menu **Examiner**, ou cliquez sur le graphique **Demandes ayant échoué**.

    ![Demandes ayant échoué](media/tutorial-runtime-exceptions/failed-requests.png)

3. Le panneau **Demandes ayant échoué** indique le nombre de demandes ayant échoué et le nombre d’utilisateurs affectés pour chaque opération de l’application.  En triant ces informations par utilisateur, vous pouvez identifier les échecs qui ont le plus d’impact sur les utilisateurs.  Dans cet exemple, les opérations **GET Employees/Create** et **GET Customers/Details** sont probablement de bonnes candidates pour examiner la cause en raison du nombre important d’échecs générés et d’utilisateurs affectés.  La sélection d’une opération a pour effet d’afficher des informations supplémentaires sur celle-ci dans le volet droit.

    ![Panneau des demandes ayant échoué](media/tutorial-runtime-exceptions/failed-requests-blade.png)

4. Réduisez la fenêtre de temps pour zoomer sur la période pendant laquelle le taux d’échec présente un pic.

    ![Fenêtre des demandes ayant échoué](media/tutorial-runtime-exceptions/failed-requests-window.png)

5. Consultez les exemples connexes en cliquant sur le bouton avec le nombre de résultats filtrés. Les exemples « suggérés » disposent d’une télémétrie associée à partir de tous les composants, même si l’échantillonnage a été appliqué à l’un d’eux. Cliquez sur un résultat de recherche pour afficher les détails de l’échec.

    ![Exemples de demande ayant échoué](media/tutorial-runtime-exceptions/failed-requests-search.png)

6. Les détails de la demande ayant échoué affiche le diagramme de Gantt qui montre qu’il y avait deux échecs de dépendance dans cette transaction, ce qui est également attribué à plus de 50 % de la durée totale de la transaction. Cette expérience présente toutes les données de télémétrie à travers les composants d’une application distribuée qui sont liés à cet ID d’opération. [En savoir plus sur la nouvelle expérience](../app/transaction-diagnostics.md). Vous pouvez sélectionner n’importe quels éléments pour afficher ses détails sur le côté droit. 

    ![Détails des requêtes ayant échoué](media/tutorial-runtime-exceptions/failed-request-details.png)

7. Le détail des opérations montre également une exception FormatException qui semble être à l’origine de l’échec.  Vous pouvez voir que celle-ci est due à un code postal non valide. Vous pouvez ouvrir la capture instantanée du débogage pour afficher des informations de débogage au niveau du code dans Visual Studio.

    ![Détails de l’exception](media/tutorial-runtime-exceptions/failed-requests-exception.png)

## <a name="identify-failing-code"></a>Identifier le code défaillant
Le débogueur d'instantané collecte des captures instantanées des exceptions les plus fréquentes dans votre application pour vous aider à diagnostiquer leur cause racine en production.  Vous pouvez afficher les captures instantanées de débogage dans le portail pour consulter la pile des appels et inspecter les variables à chaque frame de pile des appels. Par la suite, vous pouvez déboguer le code source en téléchargeant l’instantané et en l’ouvrant dans Visual Studio 2019 Enterprise.

1. Dans les propriétés de l’exception, cliquez sur **Ouvrir l'instantané de débogage**.
2. Le panneau **Déboguer l'instantané** s’ouvre, affichant la pile des appels pour la demande.  Cliquez sur une méthode pour afficher les valeurs de toutes les variables locales au moment de la demande.  En partant de la méthode du haut dans cet exemple, nous pouvons voir les variables locales qui n’ont aucune valeur.

    ![Déboguer l'instantané](media/tutorial-runtime-exceptions/debug-snapshot-01.png)

3. Le premier appel dont les valeurs sont valides est **ValidZipCode**, et nous voyons qu’un code postal a été fourni, qui contient des lettres ne pouvant pas être traduites en chiffres.  Il semble qu’il s’agisse de l’erreur de code à corriger.

    ![Capture d’écran montrant une erreur dans le code qui doit être corrigée.    ](media/tutorial-runtime-exceptions/debug-snapshot-02.png)

4. Vous avez ensuite la possibilité de télécharger cet instantané dans Visual Studio où il est possible de localiser le code réel à corriger. Pour ce faire, cliquez sur **Télécharger l’instantané**.
5. L’instantané est chargé dans Visual Studio.
6. Vous pouvez maintenant exécuter une session de débogage dans Visual Studio Enterprise, qui permet d’identifier rapidement la ligne de code ayant provoqué l’exception.

    ![Exception dans le code](media/tutorial-runtime-exceptions/exception-code.png)


## <a name="use-analytics-data"></a>Utiliser des données Analytics
Toutes les données collectées par Application Insights sont stockées dans Azure Log Analytics qui offre un langage de requête riche permettant d’analyser les données de plusieurs façons.  Nous pouvons utiliser ces données pour analyser les demandes à l’origine de l’exception que nous examinons. 

1. Cliquez sur les informations CodeLens au-dessus du code pour afficher la télémétrie fournie par Application Insights.

    ![Code](media/tutorial-runtime-exceptions/codelens.png)

1. Cliquez sur **Analyser l’impact** pour ouvrir Application Insights Analytics.  Application Insights Analytics comprend plusieurs requêtes qui fournissent des détails sur les demandes ayant échoué, tels que les utilisateurs, les navigateurs et les régions concernés.<br><br>![Capture d’écran montrant la fenêtre Application Insights comprenant plusieurs requêtes.](media/tutorial-runtime-exceptions/analytics.png)<br>

## <a name="add-work-item"></a>Ajouter un élément de travail
Si vous connectez Application Insights à un système de suivi tel que Azure DevOps ou GitHub, vous pouvez créer un élément de travail directement à partir d’Application Insights.

1. Revenez au panneau **Propriétés d'exception** dans Application Insights.
2. Cliquez sur **Nouvel élément de travail**.
3. Le panneau **Nouvel élément de travail** s’ouvre, affichant les détails relatifs à l’exception.  Vous pouvez ajouter des informations avant de l’enregistrer.

    ![Nouvel élément de travail](media/tutorial-runtime-exceptions/new-work-item.png)

## <a name="next-steps"></a>Étapes suivantes
À présent que vous avez appris à identifier les exceptions runtime, passez au didacticiel suivant pour apprendre à identifier et à diagnostiquer les problèmes de performances.

> [!div class="nextstepaction"]
> [Identifier les problèmes de performances](./tutorial-performance.md)

