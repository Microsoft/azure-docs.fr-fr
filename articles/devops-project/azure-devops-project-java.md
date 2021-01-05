---
title: 'Démarrage rapide : Créer un pipeline CI/CD pour Java – Azure DevOps Starter'
description: Apprenez à utiliser l’expérience Azure DevOps Starter simplifiée pour configurer un pipeline d’intégration continue (CI) et de livraison continue (CD) pour votre application Java dans Azure Pipelines.
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
ms.custom: mvc, seo-java-july2019, seo-java-august2019, seo-java-september2019, devx-track-java
ms.openlocfilehash: 077730d63d388566bd842a4ba185bd5fd6637043
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97588996"
---
# <a name="set-up-a-cicd-pipeline-for-a-java-app-with-azure-devops-starter"></a>Configurer un pipeline CI/CD pour une application Java à l’aide d’Azure DevOps Starter

Dans ce guide de démarrage rapide, vous allez utiliser l’expérience Azure DevOps Starter simplifiée pour configurer un pipeline d’intégration continue (CI) et de livraison continue (CD) pour votre application Java dans Azure Pipelines. Vous pouvez utiliser Azure DevOps Starter pour configurer tout ce dont vous avez besoin pour développer, déployer et superviser votre application. 

## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio). 
- Un compte et une organisation [Azure DevOps](https://azure.microsoft.com/services/devops/).

## <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

DevOps Starter crée un pipeline CI/CD dans Azure Pipelines. Vous pouvez créer une organisation Azure DevOps ou utiliser une organisation existante. DevOps Starter crée également des ressources Azure dans l’abonnement Azure de votre choix.

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Dans la zone de recherche, tapez, puis sélectionnez **DevOps Starter**. Cliquez sur le **signe +** pour créer une ressource.

    ![Tableau de bord DevOps Starter](_img/azure-devops-starter-aks/search-devops-starter.png)

## <a name="select-a-sample-application-and-azure-service"></a>Sélectionner un exemple d’application et le service Azure

1. Sélectionnez l’exemple d’application Java. Les exemples Java offrent le choix entre plusieurs infrastructures d'application.

1. L’exemple de framework par défaut est Spring. Laissez le paramètre par défaut, puis sélectionnez **Suivant**.  Web App pour conteneurs est la cible de déploiement par défaut. Le framework d’application, que vous avez choisi précédemment, détermine le type de cible de déploiement de service Azure disponible ici. 

2. Laissez le service par défaut, puis sélectionnez **Suivant**.
 
## <a name="configure-azure-devops-and-an-azure-subscription"></a>Configurer Azure DevOps et un abonnement Azure 

1. Créez une organisation Azure DevOps ou choisissez une organisation existante. 
   
   1. Choisissez un nom pour votre projet. 
   
   1. Sélectionnez votre abonnement et un emplacement Azure, choisissez un nom pour votre application, puis sélectionnez **Terminé**.  
   Après quelques minutes, le tableau de bord DevOps Starter est affiché dans le Portail Azure. Un exemple d’application est configuré dans un dépôt de votre organisation Azure DevOps, une build est exécutée et votre application est déployée sur Azure. Ce tableau de bord donne une visibilité sur votre dépôt de code, le pipeline CI/CD et votre application dans Azure.
   
2. Sélectionnez **Parcourir** pour voir votre application en cours d’exécution.
   
   ![Afficher un tableau de bord d’application dans le portail Azure](_img/azure-devops-project-java/azure-devops-application-dashboard.png) 

DevOps Starter a configuré automatiquement un déclencheur de build et de mise en production CI.  Vous êtes maintenant prêt à collaborer avec une équipe sur une application Java avec un processus CI/CD qui déploie automatiquement votre dernier travail sur votre site web.

## <a name="commit-code-changes-and-execute-cicd"></a>Valider les modifications de code et exécuter CI/CD

DevOps Starter crée un référentiel Git dans Azure Repos ou dans GitHub. Pour voir le dépôt et modifier le code dans votre application, suivez les étapes ci-dessous :

1. À gauche du tableau de bord DevOps Starter, sélectionnez le lien correspondant à votre branche principale. Ce lien ouvre une vue sur le référentiel Git récemment créé.

1. Pour afficher l’URL du clone du dépôt, sélectionnez **Cloner** en haut à droite du navigateur. Vous pouvez cloner votre référentiel Git dans votre environnement de développement intégré favori. Dans les prochaines étapes, vous allez utiliser le navigateur web pour effectuer des modifications de code directement dans la branche principale et les valider.

1. Sur la gauche du navigateur, accédez au fichier **src/main/webapp/index.html**.

1. Sélectionnez **Modifier**, puis modifiez le texte.
    Par exemple, modifiez l’une des balises div.

1. Sélectionnez **Valider**, puis enregistrez vos modifications.

1. Dans le navigateur, accédez au tableau de bord DevOps Starter.   
Vous devez maintenant voir une build en cours. Les modifications que vous venez d’effectuer font automatiquement l’objet d’une build et d’un déploiement via un pipeline CI/CD.

## <a name="examine-the-cicd-pipeline"></a>Examiner le pipeline CI/CD

 Dans l’étape précédente, DevOps Starter a automatiquement configuré un pipeline CI/CD complet. Explorez et personnalisez le pipeline selon vos besoins. Effectuez les étapes suivantes pour vous familiariser avec les pipelines de build et de mise en production.

1. En haut du tableau de bord DevOps Starter, sélectionnez **Pipelines de build**. Ce lien ouvre un onglet du navigateur et le pipeline de build de votre nouveau projet.

1. Pointez sur le champ **État**, puis sélectionnez les points de suspension (...). Cette action ouvre un menu permettant de lancer plusieurs activités, comme mettre une nouvelle build en file d’attente, mettre en pause une build et modifier le pipeline de build.

1. Sélectionnez **Modifier**.

1. Dans ce volet, vous pouvez examiner les différentes tâches de votre pipeline de build. La build effectue différentes tâches, comme la récupération (fetch) de sources dans le dépôt Git, la restauration de dépendances et la publication des sorties utilisées pour les déploiements.

1. En haut du pipeline de build, sélectionnez le nom du pipeline de build.

1. Remplacez le nom de votre pipeline de build par un nom plus descriptif, sélectionnez **Enregistrer et mettre en file d’attente**, puis sélectionnez **Enregistrer**.

1. Sous le nom de votre pipeline de build, sélectionnez **Historique**.   
Dans le volet **Historique**, vous pouvez voir une piste d’audit des modifications que vous avez apportées récemment à la build.  Azure Pipelines gère les modifications apportées au pipeline de build et vous permet de comparer les versions.

1. Sélectionnez **Déclencheurs**.  DevOps Starter a créé automatiquement un déclencheur CI, et chaque validation dans le référentiel lance une nouvelle build.  Vous pouvez éventuellement choisir d’inclure ou d’exclure des branches dans le processus d’intégration continue.

1. Sélectionnez **Rétention**. En fonction de votre scénario, vous pouvez spécifier des stratégies pour conserver ou supprimer un certain nombre de builds.

1. Sélectionnez **Build et mise en production**, puis **Mises en production**.  
 DevOps Starter crée un pipeline de mise en production pour gérer les déploiements sur Azure.

1. Sur la gauche, sélectionnez les points de suspension (...) en regard de votre pipeline de mise en production, puis sélectionnez **Modifier**. Le pipeline de mise en production contient un pipeline qui définit le processus de mise en production.  
    
12. Sous **Artefacts**, sélectionnez **Déposer**. Le pipeline de build que vous avez examiné aux étapes précédentes produit la sortie qui est utilisée pour l’artefact. 

1. En regard de l’icône **Déposer**, sélectionnez le **déclencheur de déploiement continu**. Ce pipeline de mise en production a un déclencheur CD activé, qui effectue un déploiement chaque fois qu’un nouvel artefact de build est disponible. Si vous le souhaitez, vous pouvez désactiver le déclencheur afin que vos déploiements nécessitent une exécution manuelle. 

1. Sur la gauche, sélectionnez **Tâches**. Les tâches sont les activités effectuées par votre processus de déploiement. Dans cet exemple, une tâche a été créée pour déployer sur Azure App Service.

1. Sur la droite, sélectionnez **Afficher les mises en production**. Cette vue affiche un historique des mises en production.

1. Sélectionnez les points de suspension (...) en regard d’une de vos mises en production, puis sélectionnez **Ouvrir**. Il y a plusieurs menus à explorer, comme un résumé des mises en production, les éléments de travail associés et les tests.

1. Sélectionnez **Validations**. Cette vue montre les validations de code associées au déploiement spécifique. 

1. Sélectionnez **Journaux d’activité**. Les journaux d’activité contiennent des informations utiles sur le processus de déploiement. Ils peuvent être affichés pendant et après les déploiements.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’en avez plus besoin, vous pouvez supprimer Azure App Service et les autres ressources associées. Utilisez la fonctionnalité **Supprimer** du tableau de bord DevOps Starter.

## <a name="next-steps"></a>Étapes suivantes

Quand vous avez configuré votre processus CI/CD, les pipelines de build et de mise en production ont été créés automatiquement. Vous pouvez le modifier afin qu’il réponde aux besoins de votre équipe. Pour en savoir plus sur le pipeline CI/CD, consultez :

> [!div class="nextstepaction"]
> [Personnaliser le processus CD](/azure/devops/pipelines/release/define-multistage-release-process?view=vsts)