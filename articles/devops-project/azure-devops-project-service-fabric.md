---
title: 'Tutoriel : Déployer votre application ASP.NET Core dans Azure Service Fabric à l’aide d’Azure DevOps Starter'
description: Azure DevOps Starter facilite la prise en main d’Azure. Avec Azure DevOps Projects, vous pouvez déployer une application ASP.NET Core dans Azure Service Fabric en quelques étapes rapides.
ms.author: mlearned
ms.manager: gwallace
ms.prod: devops
ms.technology: devops-cicd
ms.topic: tutorial
ms.date: 03/24/2020
author: mlearned
ms.openlocfilehash: 723f46652643883dc8e718468a9ca9eead50e1aa
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91856022"
---
# <a name="tutorial-deploy-your-aspnet-core-app-to-azure-service-fabric-by-using-azure-devops-starter"></a>Tutoriel : Déployer votre application ASP.NET Core dans Azure Service Fabric à l’aide d’Azure DevOps Starter

Azure DevOps Starter offre une expérience simplifiée dans laquelle vous pouvez apporter vos code et référentiel Git existants ou choisir un exemple d’application pour créer un pipeline d’intégration continue (CI) et de livraison continue (CD) dans Azure. 

En outre, DevOps Starter :

* Crée automatiquement des ressources Azure, telles qu’Azure Service Fabric
* Crée et configure un pipeline de mise en production dans Azure DevOps qui définit un pipeline CI/CD
* Crée une ressource Azure Application Insights pour la supervision

Ce didacticiel présente les procédures suivantes :

> [!div class="checklist"]
> * Utiliser DevOps Starter pour créer une application ASP.NET Core et la déployer sur Service Fabric
> * Configurer Azure DevOps et un abonnement Azure 
> * Examiner le pipeline CI
> * Examiner le pipeline CD
> * Valider les modifications dans git et les déployer automatiquement dans Azure
> * Nettoyer les ressources

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Vous pouvez en obtenir un gratuit via [Visual Studio Dev Essentials](https://visualstudio.microsoft.com/dev-essentials/).

## <a name="use-devops-starter-to-create-an-aspnet-core-app-and-deploy-it-to-service-fabric"></a>Utiliser DevOps Starter pour créer une application ASP.NET Core et la déployer sur Service Fabric

DevOps Starter crée un pipeline CI/CD dans Azure Pipelines. Vous pouvez créer une organisation Azure DevOps ou utiliser une organisation existante. DevOps Starter crée également des ressources Azure, telles qu’un cluster Service Fabric, dans l’abonnement Azure de votre choix.

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Dans la zone de recherche, tapez, puis sélectionnez **DevOps Starter**. Cliquez sur le **signe +** pour créer une ressource.

    ![Tableau de bord DevOps Starter](_img/azure-devops-starter-aks/search-devops-starter.png) 

1. Sélectionnez **.NET**, puis sélectionnez **Suivant**.

1. Sous **Choisir un framework d’application**, sélectionnez **ASP.NET Core**, puis **Suivant**.

1. Sélectionnez **Cluster Service Fabric**, puis **Suivant**. 

## <a name="configure-azure-devops-and-an-azure-subscription"></a>Configurer Azure DevOps et un abonnement Azure

1. Créez une organisation Azure DevOps ou sélectionnez une organisation existante. 

1. Indiquez un nom pour votre projet Azure DevOps. 

1. Sélectionnez votre abonnement Azure.

1. Pour afficher d’autres paramètres de configuration Azure et identifier la taille de la machine virtuelle du nœud ainsi que le système d’exploitation du cluster Service Fabric, sélectionnez **Modifier**. Ce volet regroupe différentes options pour la configuration du type et de l’emplacement des services Azure.
 
1. Quittez la zone de configuration Azure, puis sélectionnez **Terminé**.  
    Le processus est achevé au bout de quelques minutes. Un exemple d’application ASP.NET Core est configuré dans un dépôt Git de votre organisation Azure DevOps, un cluster Service Fabric est créé, un pipeline CI/CD est exécuté et votre application est déployée sur Azure. 

    Une fois toutes ces opérations terminées, le tableau de bord DevOps Starter s’affiche dans le Portail Azure. Vous pouvez également accéder au tableau de bord DevOps Starter directement à partir de **Toutes les ressources** dans le Portail Azure. 

    Ce tableau de bord permet de visualiser votre dépôt de code Azure DevOps, le pipeline CI/CD et le cluster Service Fabric. Vous pouvez configurer d’autres options pour votre pipeline CI/CD dans Azure Repos. À droite, sélectionnez **Parcourir** pour voir votre application en cours d’exécution.

## <a name="examine-the-ci-pipeline"></a>Examiner le pipeline CI

DevOps Starter configure automatiquement un pipeline CI/CD dans Azure Pipelines. Vous pouvez explorer et personnaliser le pipeline. Pour vous familiariser avec celui-ci, procédez comme suit :

1. Accédez au tableau de bord DevOps Starter.

1. En haut du tableau de bord DevOps Starter, sélectionnez **Pipelines de build**. Un onglet du navigateur affiche le pipeline de build de votre nouveau projet.

1. Pointez sur le champ **État**, puis sélectionnez les points de suspension (...). Un menu affiche plusieurs options, telles que de la mise en file d’attente d’une nouvelle build, la mise en pause d’une build et la modification du pipeline de build.

1. Sélectionnez **Modifier**.

1. Dans ce volet, vous pouvez examiner les différentes tâches de votre pipeline de build. La build effectue diverses opérations, comme la récupération (fetch) de sources dans le dépôt Git, la restauration de dépendances et la publication de sorties utilisées pour les déploiements.

1. En haut du pipeline de build, sélectionnez le nom du pipeline de build. 

1. Sous le nom de votre pipeline de build, sélectionnez **Historique**. Ce volet montre une piste d’audit des modifications que vous avez apportées récemment à la build. Azure DevOps suit les modifications apportées au pipeline de build et permet de comparer les versions.

1. Sélectionnez **Déclencheurs**. DevOps Starter crée automatiquement un déclencheur CI tandis que chaque validation dans le dépôt lance une nouvelle build. Si vous le souhaitez, vous pouvez choisir d’inclure ou d’exclure des branches dans le processus CI.

1. Sélectionnez **Rétention**. En fonction de votre scénario, vous pouvez spécifier des stratégies pour conserver ou supprimer un certain nombre de builds.

## <a name="examine-the-cd-pipeline"></a>Examiner le pipeline CD

DevOps Starter crée et configure automatiquement les étapes nécessaires au déploiement entre votre organisation Azure DevOps et votre abonnement Azure. Ces étapes comprennent la configuration d’une connexion au service Azure pour authentifier Azure DevOps auprès de votre abonnement Azure. L’automatisation crée également un pipeline de mise en production, qui fournit le déploiement continu vers Azure. Pour en savoir plus sur le pipeline de mise en production, suivez ces étapes :

1. Sélectionnez **Build et mise en production**, puis **Mises en production**. DevOps Starter crée un pipeline de mise en production pour gérer les déploiements sur Azure.

1. Sélectionnez les points de suspension (...) en regard de votre pipeline de mise en production, puis sélectionnez **Modifier**. Le pipeline de mise en production contient un *pipeline* qui définit le processus de mise en production.

1. Sous **Artefacts**, sélectionnez **Déposer**. Le pipeline de build que vous avez examiné précédemment génère la sortie qui est utilisée pour l’artefact. 

1. À droite de l’icône **Déposer**, sélectionnez **Déclencheur de déploiement continu**. Ce pipeline de mise en production est doté d’un déclencheur de déploiement continu activé qui exécute un déploiement chaque fois qu’un nouvel artefact de build est disponible. Si vous le souhaitez, vous pouvez désactiver le déclencheur afin que vos déploiements nécessitent une exécution manuelle. 

1. À droite, sélectionnez **Afficher les mises en production** pour voir l’historique des mises en production.

1. Sélectionnez les points de suspension (...) en regard d’une mise en production, puis sélectionnez **Ouvrir**. Vous pouvez explorer plusieurs menus, comme un résumé des mises en production, les éléments de travail associés et les tests.

1. Sélectionnez **Validations**. Cette vue montre les validations de code qui sont associées à ce déploiement. Comparez les mises en production pour afficher les différences de validation entre les déploiements.

1. Sélectionnez **Journaux d’activité**. Les journaux d’activité contiennent des informations utiles sur le processus de déploiement. Vous pouvez les voir à la fois pendant et après les déploiements.

## <a name="commit-changes-to-git-and-automatically-deploy-them-to-azure"></a>Valider les modifications dans Git et les déployer automatiquement dans Azure 

 > [!NOTE]
 > La procédure suivante teste le pipeline CI/CD en apportant une modification de texte simple.

Vous êtes maintenant prêt à collaborer avec une équipe sur votre application, par le biais d’un processus d’intégration et de déploiement continus (CI/CD) qui déploie automatiquement votre dernier travail sur votre site web. Chaque modification apportée au dépôt Git démarre un build tandis qu’une mise en production déploie ces modifications sur Azure. Suivez la procédure décrite dans cette section, ou utilisez une autre technique pour valider les modifications dans votre dépôt. Par exemple, vous pouvez cloner le dépôt Git dans votre outil ou environnement de développement intégré favori, puis envoyer (push) les modifications vers ce dépôt.

1. Dans le menu Azure DevOps, sélectionnez **Code** > **Fichiers**, puis accédez à votre dépôt.

1. Accédez au répertoire *Views\Home*, sélectionnez les points de suspension (...) situés en regard du fichier *Index.cshtml*, puis sélectionnez **Modifier**.

1. Modifiez le fichier, par exemple en ajoutant du texte dans une des balises div. 

1. En haut à droite, sélectionnez **Valider**, puis **Valider** à nouveau pour transmettre votre modification.  
    Au bout de quelques instants, un build est démarré, puis une mise en production est exécutée pour déployer les modifications. Vous pouvez superviser l’état de la build dans le tableau de bord DevOps Starter ou dans le navigateur avec la journalisation en temps réel d’Azure DevOps.

1. Une fois la mise en production terminée, actualisez votre application pour vérifier vos modifications.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous êtes en phase de test, vous pouvez éviter une hausse des frais de facturation en nettoyant vos ressources. Quand ils ne sont plus nécessaires, vous pouvez supprimer le cluster Azure Service Fabric et les ressources associées que vous avez créés dans ce tutoriel. Pour ce faire, utilisez la fonctionnalité **Supprimer** du tableau de bord DevOps Starter.

> [!IMPORTANT]
> La procédure suivante supprime définitivement les ressources. La fonctionnalité *Supprimer* détruit les données qui sont créées par le projet dans DevOps Starter, à la fois dans Azure et dans Azure DevOps, et vous ne pourrez plus les récupérer. Utilisez cette procédure uniquement après avoir lu attentivement les invites.

1. Dans le Portail Azure, accédez au tableau de bord DevOps Starter.
1. En haut à droite, sélectionnez **Supprimer**. 
1. À l’invite, sélectionnez **Oui** pour *supprimer définitivement* les ressources.

## <a name="next-steps"></a>Étapes suivantes

Si vous le souhaitez, vous pouvez modifier le pipeline CI/CD Azure afin qu’il réponde aux besoins de votre équipe. Vous pouvez également utiliser ce modèle CI/CD comme modèle pour vos autres pipelines. Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Utiliser DevOps Starter pour créer une application ASP.NET Core et la déployer sur Service Fabric
> * Configurer Azure DevOps et un abonnement Azure 
> * Examiner le pipeline CI
> * Examiner le pipeline CD
> * Valider les modifications dans Git et les déployer automatiquement dans Azure
> * Nettoyer les ressources

Pour en savoir plus sur Service Fabric et les microservices, voir :

> [!div class="nextstepaction"]
> [Utiliser une approche de microservices pour la conception d’applications](/azure/devops/pipelines/release/define-multistage-release-process?view=vsts)