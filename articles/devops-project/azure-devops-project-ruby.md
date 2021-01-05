---
title: 'Démarrage rapide : Créer un pipeline CI/CD pour Ruby on Rails à l’aide d’Azure DevOps Starter'
description: Azure DevOps Starter permet de démarrer facilement avec Azure. Vous pouvez lancer une application web Ruby sur un service Azure en quelques étapes rapides.
ms.prod: devops
ms.technology: devops-cicd
services: vsts
documentationcenter: vs-devops-build
author: mlearned
manager: gwallace
ms.workload: web
ms.tgt_pltfrm: na
ms.topic: quickstart
ms.date: 03/24/2020
ms.author: mlearned
ms.custom: mvc
ms.openlocfilehash: 9e2f7115560148a1d3b9d2671be54c1ba4d61946
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97588911"
---
# <a name="create-a-cicd-pipeline-for-ruby-on-rails-by-using-azure-devops-starter"></a>Créer un pipeline CI/CD pour Ruby on Rails à l’aide d’Azure DevOps Starter

Configurez l’intégration continue (CI) et la livraison continue (CD) pour votre application Ruby on Rails à l’aide d’Azure DevOps Starter. DevOps Starter simplifie la configuration initiale d’un pipeline de build et de mise en production Azure DevOps.

Si vous ne disposez pas d’un abonnement Azure, vous pouvez en obtenir un gratuitement via [Visual Studio Dev Essentials](https://visualstudio.microsoft.com/dev-essentials/).

## <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

Azure DevOps Starter crée un pipeline CI/CD dans Azure Repos. Vous pouvez créer une organisation Azure DevOps ou utiliser une organisation existante. DevOps Starter crée également des ressources Azure dans l’abonnement Azure de votre choix.

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Dans la zone de recherche, tapez, puis sélectionnez **DevOps Starter**. Cliquez sur le **signe +** pour créer une ressource.

    ![Tableau de bord DevOps Starter](_img/azure-devops-starter-aks/search-devops-starter.png) 

## <a name="select-a-sample-app-and-azure-service"></a>Sélectionner un exemple d’application et un service Azure

1. Sélectionnez l’exemple d’application **Ruby**.

1. Sélectionnez l’infrastructure d’application **Ruby on Rails**. Quand vous avez terminé, sélectionnez **Suivant**.

1. La cible de déploiement par défaut est **Web App sur Linux**.  Si vous le souhaitez, vous pouvez sélectionner **Web App pour conteneurs**. Le framework d’application, que vous avez choisi précédemment, détermine le type de cible de déploiement de service Azure disponible ici. 
    
1. Sélectionnez le service cible, puis **Suivant**.

## <a name="configure-azure-devops-and-an-azure-subscription"></a>Configurer Azure DevOps et un abonnement Azure 

1. Créez gratuitement une organisation Azure DevOps ou choisissez une organisation existante. 

1. Indiquez un nom pour votre projet Azure DevOps. 

1. Sélectionnez votre abonnement et un emplacement Azure, entrez un nom pour votre application, puis sélectionnez **Terminé**.  
    Après quelques minutes, le tableau de bord DevOps Starter est affiché dans le portail Azure. Un exemple d’application est configuré dans un dépôt de votre organisation Azure DevOps, une build est exécutée et votre application est déployée sur Azure. 
    
    Le tableau de bord donne une visibilité sur votre dépôt de code, votre pipeline CI/CD et votre application dans Azure. À droite, sélectionnez **Parcourir** pour voir votre application en cours d’exécution.

    ![Vue du tableau de bord](_img/azure-devops-project-go/dashboardnopreview.png) 

## <a name="commit-your-code-changes-and-execute-the-cicd"></a>Valider vos modifications de code et exécuter la CI/CD

Azure DevOps Starter crée un référentiel Git dans Azure Pipelines ou dans GitHub. Pour voir le dépôt et modifier le code dans votre application, effectuez les étapes suivantes :

1. Dans le tableau de bord DevOps Starter, sur la gauche, sélectionnez le lien correspondant à votre branche principale. Le lien ouvre une vue sur le dépôt Git récemment créé.

1. Pour voir l’URL du clone du dépôt, sélectionnez **Cloner** en haut à droite. Vous pouvez cloner votre dépôt Git dans votre IDE favori. Dans les prochaines étapes, vous allez utiliser le navigateur web pour effectuer des modifications de code directement dans la branche principale et les valider.

1. Sur la gauche, accédez au fichier *app/views/pages/home.html.erb*, puis sélectionnez **Modifier**.

1. Faites une modification dans le fichier. Par exemple, modifiez le texte qui se trouve à l’intérieur d’une des balises div.

1. Sélectionnez **Valider**, puis enregistrez vos modifications.

1. Dans le navigateur, accédez au tableau de bord DevOps Starter. Une build doit être en cours. Les modifications que vous avez apportées font automatiquement l’objet d’une build et d’un déploiement via un pipeline CI/CD.

## <a name="examine-the-azure-pipelines-cicd-pipeline"></a>Examiner le pipeline CI/CD d’Azure Pipelines

Azure DevOps Starter configure automatiquement un pipeline CI/CD complet dans votre organisation Azure DevOps. Explorez et personnalisez le pipeline selon vos besoins. Pour vous familiariser avec les pipelines de build et de mise en production d’Azure DevOps, effectuez les étapes suivantes :

1. Accédez au tableau de bord DevOps Starter.

1. En haut, sélectionnez **Pipelines de build**. Un onglet du navigateur affiche le pipeline de build de votre nouveau projet.

1. Pointez sur le champ **État**, puis sélectionnez les points de suspension (...). Un menu affiche plusieurs options, telles que de la mise en file d’attente d’une nouvelle build, la mise en pause d’une build et la modification du pipeline de build.

1. Sélectionnez **Modifier**.

1. Dans ce volet, vous pouvez examiner les différentes tâches de votre pipeline de build. La build effectue diverses opérations, comme la récupération (fetch) de sources dans le dépôt Git, la restauration de dépendances et la publication de sorties utilisées pour les déploiements.

1. En haut du pipeline de build, sélectionnez le nom du pipeline de build.

1. Remplacez le nom de votre pipeline de build par un nom plus descriptif, sélectionnez **Enregistrer et mettre en file d’attente**, puis sélectionnez **Enregistrer**.

1. Sous le nom de votre pipeline de build, sélectionnez **Historique**. Ce volet montre une piste d’audit des modifications que vous avez apportées récemment à la build. Azure DevOps suit les modifications apportées au pipeline de build et permet de comparer les versions.

1. Sélectionnez **Déclencheurs**.  DevOps Starter crée automatiquement un déclencheur CI tandis que chaque validation dans le dépôt lance une nouvelle build. Si vous le souhaitez, vous pouvez choisir d’inclure ou d’exclure des branches dans le processus CI.

1. Sélectionnez **Rétention**. En fonction de votre scénario, vous pouvez spécifier des stratégies pour conserver ou supprimer un certain nombre de builds.

1. Sélectionnez **Build et mise en production**, puis **Mises en production**.  DevOps Starter crée un pipeline de mise en production pour gérer les déploiements sur Azure.

1. Sélectionnez les points de suspension (...) en regard de votre pipeline de mise en production, puis sélectionnez **Modifier**. Le pipeline de mise en production contient un *pipeline* qui définit le processus de mise en production.

1. Sous **Artefacts**, sélectionnez **Déposer**. Le pipeline de build que vous avez examiné précédemment génère la sortie qui est utilisée pour l’artefact. 

1. À droite de l’icône **Déposer**, sélectionnez **Déclencheur de déploiement continu**. Ce pipeline de mise en production est doté d’un déclencheur de déploiement continu activé qui exécute un déploiement chaque fois qu’un nouvel artefact de build est disponible. Si vous le souhaitez, vous pouvez désactiver le déclencheur afin que vos déploiements nécessitent une exécution manuelle. 

1. À gauche, sélectionnez **Tâches**. Les tâches sont les activités effectuées par votre processus de déploiement. Dans cet exemple, une tâche a été créée pour déployer sur Azure App Service.

1. À droite, sélectionnez **Afficher les mises en production** pour voir l’historique des mises en production.

1. Sélectionnez les points de suspension (...) en regard d’une mise en production, puis sélectionnez **Ouvrir**. Vous pouvez explorer plusieurs menus, comme un résumé des mises en production, les éléments de travail associés et les tests.

1. Sélectionnez **Validations**. Cette vue montre les validations de code qui sont associées à ce déploiement. 

1. Sélectionnez **Journaux d’activité**. Les journaux d’activité contiennent des informations utiles sur le processus de déploiement. Vous pouvez les voir à la fois pendant et après les déploiements.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand ils ne sont plus nécessaires, vous pouvez supprimer l’instance Azure App Service et les ressources associées que vous avez créés dans ce guide de démarrage rapide. Pour ce faire, utilisez la fonctionnalité **Supprimer** du tableau de bord DevOps Starter.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la modification des pipelines de build et de mise en production afin de répondre aux besoins de votre équipe, consultez :

> [!div class="nextstepaction"]
> [Définir votre pipeline de déploiement continu (CD) multiétape](/azure/devops/pipelines/release/define-multistage-release-process?view=vsts)