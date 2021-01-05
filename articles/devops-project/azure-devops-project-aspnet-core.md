---
title: 'Démarrage rapide : Créer un pipeline CI/CD pour .NET avec Azure DevOps Starter'
description: Azure DevOps Starter facilite la prise en main d’Azure. Il vous aide à lancer une application .NET sur le service Azure de votre choix en quelques étapes rapides.
ms.prod: devops
ms.technology: devops-cicd
services: azure-devops-project
documentationcenter: vs-devops-build
author: mlearned
manager: gwallace
editor: ''
ms.assetid: ''
ms.workload: web
ms.tgt_pltfrm: na
ms.topic: quickstart
ms.date: 03/24/2020
ms.author: mlearned
ms.custom: devx-track-csharp, mvc
ms.openlocfilehash: 63d3ac356e4f2c1d17e1511227f5089b7ff4266f
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97589155"
---
# <a name="create-a-cicd-pipeline-for-net-with-azure-devops-starter"></a>Créer un pipeline CI/CD pour .NET avec Azure DevOps Starter

Configurez l’intégration continue (CI) et la livraison continue (CD) pour votre application .NET Core ou ASP.NET avec DevOps Starter. DevOps Starter simplifie la configuration initiale d’un pipeline de build et de mise en production dans Azure Pipelines.

Si vous ne disposez pas d’un abonnement Azure, vous pouvez en obtenir un gratuitement via [Visual Studio Dev Essentials](https://visualstudio.microsoft.com/dev-essentials/).

## <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

Azure DevOps Starter crée un pipeline CI/CD dans Azure DevOps. Vous pouvez créer une organisation Azure DevOps ou utiliser une organisation existante. DevOps Starter crée également des ressources Azure dans l’abonnement Azure de votre choix.

1. Connectez-vous au [portail Microsoft Azure](https://portal.azure.com).

1. Dans la zone de recherche, tapez, puis sélectionnez **DevOps Starter**. Cliquez sur le **signe +** pour créer une ressource. 

    ![Tableau de bord DevOps Starter](_img/azure-devops-starter-aks/search-devops-starter.png)

## <a name="select-a-sample-application-and-azure-service"></a>Sélectionner un exemple d’application et le service Azure

1. Sélectionnez Exemple d’application **.NET**. Les exemples .NET offrent le choix entre une infrastructure ASP.NET open source et une infrastructure .NET Core multiplateforme.

   ![.NET Framework](_img/azure-devops-project-aspnet-core/select-dotnet.png)

2. Cet exemple est une application MVC ASP.NET Core. Sélectionnez le framework d’application **.NET Core**, puis **Suivant**.    
    
3. Sélectionnez **Application web Windows** comme cible de déploiement, puis sélectionnez **Suivant**. Si vous le souhaitez, vous pouvez choisir d’autres services Azure pour votre déploiement. Le framework d’application, que vous avez choisi précédemment, détermine le type de cible de déploiement de service Azure disponible ici.

## <a name="configure-azure-devops-and-an-azure-subscription"></a>Configurer Azure DevOps et un abonnement Azure 

1. Entrez un **Nom de projet**.

2. Créez gratuitement une **organisation Azure DevOps** ou choisissez une organisation existante dans la liste déroulante.

3. Sélectionnez votre **abonnement Azure**, entrez un nom pour votre **application web** ou prenez la valeur par défaut, puis sélectionnez **Terminé**. Après quelques minutes, la vue d’ensemble du déploiement de DevOps Starter s’affiche dans le portail Azure. 

4. Sélectionnez **Accéder à la ressource** pour afficher le tableau de bord DevOps Starter. Dans le coin supérieur droit, épinglez le **projet** à votre tableau de bord pour y accéder rapidement. Un exemple d’application est défini dans un référentiel dans votre **organisation Azure DevOps**. Un build est exécuté et votre application est déployée sur Azure.

5. Le tableau de bord donne une visibilité sur votre dépôt de code, votre pipeline CI/CD et votre application dans Azure. À droite, sous les ressources Azure, sélectionnez **Parcourir** pour voir votre application en cours d’exécution.

   ![Vue du tableau de bord](_img/azure-devops-project-aspnet-core/dashboardnopreview.png) 

## <a name="commit-code-changes-and-execute-cicd"></a>Valider les modifications de code et exécuter CI/CD

DevOps Starter a créé un dépôt Git dans Azure Repos ou dans GitHub. Pour voir le dépôt et modifier le code dans votre application, suivez les étapes ci-dessous :

1. À gauche du tableau de bord DevOps Starter, sélectionnez le lien correspondant à votre branche **principale**. Ce lien ouvre une vue sur le référentiel Git récemment créé.

2. Dans les prochaines étapes, vous allez utiliser le navigateur web pour effectuer des modifications de code directement dans la branche **principale** et les valider. Vous pouvez également cloner votre référentiel Git dans votre IDE favori en sélectionnant **Cloner** dans la partie supérieure droite de la page du référentiel. 

3. Sur la gauche, parcourez la structure de fichiers de l’application jusqu’à **Application/aspnet-core-dotnet-core/Pages/Index.cshtml**.

4. Sélectionnez **Modifier**, puis modifiez le titre h2. Par exemple, tapez **Prise en main directe d’Azure DevOps Starter** ou autre chose.

      ![Modifications du code](_img/azure-devops-project-aspnet-core/codechange.png)

5. Sélectionnez **Valider**, laissez un commentaire et sélectionnez **Valider** à nouveau.

6. Dans le navigateur, accédez au tableau de bord Azure DevOps Starter.  Vous devez maintenant voir qu’une build est en cours. Les modifications que vous avez apportées font automatiquement l’objet d’une build et d’un déploiement via un pipeline CI/CD.

## <a name="examine-the-cicd-pipeline"></a>Examiner le pipeline CI/CD

Dans l’étape précédente, Azure DevOps Starter a automatiquement configuré un pipeline CI/CD complet. Explorez et personnalisez le pipeline selon vos besoins. Pour vous familiariser avec les pipelines de build et de mise en production d’Azure DevOps, effectuez les étapes suivantes.

1. En haut du tableau de bord DevOps Starter, sélectionnez **Pipelines de build**. Ce lien ouvre un onglet du navigateur et le pipeline de build Azure DevOps pour votre nouveau projet.

1. Sélectionnez les points de suspension (...).  Cette action ouvre un menu permettant de lancer plusieurs activités, comme mettre une nouvelle build en file d’attente, mettre en pause une build et modifier le pipeline de build.

1. Sélectionnez **Modifier**.

    ![Pipeline de build](_img/azure-devops-project-aspnet-core/builddef.png)

1. Dans ce volet, vous pouvez examiner les différentes tâches de votre pipeline de build. La build effectue différentes tâches, comme la récupération (fetch) de sources dans le dépôt Git, la restauration de dépendances et la publication des sorties utilisées pour les déploiements.

1. En haut du pipeline de build, sélectionnez le nom du pipeline de build.

1. Remplacez le nom de votre pipeline de build par un nom plus descriptif, sélectionnez **Enregistrer et mettre en file d’attente**, puis sélectionnez **Enregistrer**.

1. Sous le nom de votre pipeline de build, sélectionnez **Historique**.   
Dans le volet **Historique**, vous pouvez voir une piste d’audit des modifications que vous avez apportées récemment à la build.  Azure Pipelines gère les modifications apportées au pipeline de build et vous permet de comparer les versions.

1. Sélectionnez **Déclencheurs**. DevOps Starter a créé automatiquement un déclencheur CI, et chaque validation dans le dépôt lance une nouvelle build. Vous pouvez éventuellement choisir d’inclure ou d’exclure des branches dans le processus d’intégration continue.

1. Sélectionnez **Rétention**. En fonction de votre scénario, vous pouvez spécifier des stratégies pour conserver ou supprimer un certain nombre de builds.

1. Sélectionnez **Build et mise en production**, puis **Mises en production**.  
DevOps Starter crée un pipeline de mise en production pour gérer les déploiements sur Azure.

1.  Sur la gauche, sélectionnez les points de suspension (...) en regard de votre pipeline de mise en production, puis sélectionnez **Modifier**. Le pipeline de mise en production contient un pipeline qui définit le processus de mise en production.  

1. Sous **Artefacts**, sélectionnez **Déposer**. Le pipeline de build que vous avez examiné aux étapes précédentes génère la sortie utilisée pour l’artefact. 

1. En regard de l’icône **Déposer**, sélectionnez le **déclencheur de déploiement continu**. Ce pipeline de mise en production a un déclencheur CD activé, qui effectue un déploiement chaque fois qu’un nouvel artefact de build est disponible. Si vous le souhaitez, vous pouvez désactiver le déclencheur afin que vos déploiements nécessitent une exécution manuelle.  

1. Sur la gauche, sélectionnez **Tâches**.  Les tâches sont les activités effectuées par votre processus de déploiement. Dans cet exemple, une tâche a été créée pour déployer sur Azure App Service.

1. Sur la droite, sélectionnez **Afficher les mises en production**. Cette vue affiche un historique des mises en production.

1. Sélectionnez les points de suspension (...) en regard d’une de vos mises en production, puis sélectionnez **Ouvrir**. Il y a plusieurs menus à explorer, comme un résumé des mises en production, les éléments de travail associés et les tests.

1. Sélectionnez **Validations**. Cette vue montre les validations de code associées au déploiement spécifique. 

1. Sélectionnez **Journaux d’activité**. Les journaux d’activité contiennent des informations utiles sur le processus de déploiement. Ils peuvent être affichés pendant et après les déploiements.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’en avez plus besoin, vous pouvez supprimer Azure App Service et les autres ressources associées que vous avez créées. Utilisez la fonctionnalité **Supprimer** du tableau de bord DevOps Starter.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la modification des pipelines de build et de mise en production dans le but de répondre aux besoins de votre équipe, consultez ce tutoriel :

> [!div class="nextstepaction"]
> [Personnaliser le processus CD](/azure/devops/pipelines/release/define-multistage-release-process?view=vsts)

## <a name="videos"></a>Videos

> [!VIDEO https://www.youtube.com/embed/itwqMf9aR0w]