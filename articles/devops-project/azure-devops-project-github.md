---
title: 'Tutoriel : Créer un pipeline CI/CD pour votre code existant à l’aide d’Azure DevOps Starter'
description: Azure DevOps Starter permet de démarrer facilement avec Azure. En quelques étapes rapides, DevOps Projects vous permet d’utiliser votre propre code et votre dépôt GitHub pour lancer une application sur un service Azure.
services: vsts
documentationcenter: vs-devops-build
ms.author: mlearned
ms.manager: gwallace
ms.workload: web
ms.prod: devops
ms.technology: devops-cicd
ms.tgt_pltfrm: na
ms.topic: tutorial
ms.date: 03/24/2020
author: mlearned
ms.custom: mvc
ms.openlocfilehash: 41f2ee9806b8f6818f2b2dd7a48920ac9907ca79
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91854645"
---
# <a name="create-a-cicd-pipeline-for-github-repo-using-azure-devops-starter"></a>Créer un pipeline CI/CD pour un référentiel GitHub à l’aide d’Azure DevOps Starter

Azure DevOps Starter présente un processus simplifié pour la création d’un pipeline d’intégration continue (CI) et de livraison continue (CD) vers Azure. Vous pouvez apporter votre code et dépôt Git existants, ou sélectionner un exemple d’application.

Vous allez effectuer les étapes suivantes :

> [!div class="checklist"]
> * Utiliser DevOps Starter pour créer un pipeline CI/CD
> * Configurer l’accès à votre dépôt GitHub et choisir un framework
> * Configurer Azure DevOps et un abonnement Azure 
> * Valider les modifications apportées à GitHub et les déployer automatiquement dans Azure
> * Examiner le pipeline CI/CD d’Azure Pipelines
> * Nettoyer les ressources

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Vous pouvez en obtenir un gratuit via [Visual Studio Dev Essentials](https://visualstudio.microsoft.com/dev-essentials/).
* Accédez à un dépôt GitHub ou Git externe qui contient .NET, Java, PHP, Node.js, Python ou du code web statique.

## <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

Azure DevOps Starter crée un pipeline CI/CD dans Azure Pipelines. Vous pouvez créer une organisation Azure DevOps ou utiliser une organisation existante. Azure DevOps Starter crée également des ressources Azure dans l’abonnement Azure de votre choix.

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Dans la zone de recherche, tapez, puis sélectionnez **DevOps Starter**. Cliquez sur le **signe +** pour créer une ressource.

    ![Tableau de bord DevOps Starter](_img/azure-devops-starter-aks/search-devops-starter.png)
    
1. Sélectionnez **Apportez votre propre code**, puis sélectionnez **Suivant**.

## <a name="configure-access-to-your-github-repo-and-select-a-framework"></a>Configurer l’accès à votre dépôt GitHub et sélectionner un framework

1. Sélectionnez **GitHub**  ou un dépôt de code **Git** externe. Sélectionnez **GitHub** pour ce tutoriel. Vous pouvez être amené la première fois à vous authentifier auprès de GitHub pour autoriser Azure à accéder à votre dépôt GitHub.

1. Sélectionnez un **Dépôt** et une **Branche**, puis sélectionnez **Suivant**.

1. Si vous utilisez des conteneurs Docker, définissez **Application dockerisée** sur **OUI**. Pour les besoins de ce tutoriel, laissez **NON** sélectionné, puis sélectionnez **Suivant**. Pour plus d’informations sur l’utilisation des conteneurs Docker, pointez sur l’icône **i**.

   ![Sélection de l’infrastructure d’application dans le menu déroulant](_img/azure-devops-project-github/appframework.png)

1. Dans les menus déroulants, sélectionnez un **runtime d’application** et un **framework d’application**, puis sélectionnez **Suivant**. L’infrastructure d’application détermine le type de cible de déploiement de service Azure disponible.

1. Sélectionnez un **service Azure** pour déployer l’application, puis sélectionnez **Suivant**.

## <a name="configure-azure-devops-and-an-azure-subscription"></a>Configurer Azure DevOps et un abonnement Azure

1. Entrez un nom pour **Nom du projet**.

1. Créez gratuitement une organisation dans **Organisation Azure DevOps** ou sélectionnez une organisation existante dans le menu déroulant.

1. Sélectionnez votre abonnement dans **Abonnement Azure**, puis entrez un nom dans **Application web** ou utilisez la valeur par défaut. Sélectionnez un **Emplacement**, puis sélectionnez **Terminé**. Après quelques minutes, la vue d’ensemble du déploiement de DevOps Starter s’affiche dans le Portail Azure.

1. Sélectionnez **Accéder à la ressource** pour afficher le tableau de bord DevOps Starter. Dans le coin supérieur droit, épinglez le **Projet** à votre tableau de bord pour y accéder rapidement. Azure DevOps Starter configure automatiquement un déclencheur de build et de mise en production CI. Votre code reste dans votre dépôt GitHub ou dans un autre dépôt externe, et un exemple d’application est configuré dans un dépôt **Organisation Azure DevOps**. Azure DevOps Starter exécute la build et déploie l’application sur Azure.

   ![Vue du tableau de bord Azure DevOps Projects](_img/azure-devops-project-github/projectsdashboard.png)

1. Le tableau de bord affiche votre dépôt de code, votre pipeline CI/CD et votre application dans Azure. À droite, sous Ressources Azure, sélectionnez **Parcourir** pour voir votre application en cours d’exécution.

## <a name="commit-changes-to-github-and-automatically-deploy-them-to-azure"></a>Valider les modifications apportées à GitHub et les déployer automatiquement dans Azure

Vous êtes désormais prêt à collaborer avec une équipe sur votre application. Le processus CI/CD déploie automatiquement sur votre site web le dernier travail que vous avez effectué. Chaque modification apportée au dépôt GitHub démarre une build dans Azure DevOps tandis qu’un pipeline CD exécute un déploiement dans Azure.

1. À partir de votre tableau de bord DevOps Starter, sélectionnez **Dépôts**. Votre dépôt GitHub s’ouvre dans un nouvel onglet de navigateur. Apportez une modification à votre application, puis sélectionnez **Valider les modifications**.

1. Après quelques instants, une build démarre dans Azure Pipelines. Vous pouvez superviser l’état de la build dans le tableau de bord DevOps Starter. Vous pouvez également le superviser dans votre organisation Azure DevOps en sélectionnant l’onglet **Pipelines de build** dans le tableau de bord DevOps Starter.

## <a name="examine-the-azure-pipelines-cicd-pipeline"></a>Examiner le pipeline CI/CD d’Azure Pipelines

Azure DevOps Starter configure automatiquement un pipeline CI/CD dans Azure Pipelines. Explorez et personnalisez le pipeline selon vos besoins. Pour vous familiariser avec les pipelines de build et de mise en production, effectuez les étapes suivantes :

1. À partir du tableau de bord DevOps Starter, sélectionnez **Pipelines de build**.

1. Après l’ouverture de la page **Azure Pipelines**, vous voyez un historique des builds les plus récentes, et l’état de chaque build.

   ![Page des builds Azure Pipelines](_img/azure-devops-project-github/pipelinesbuildpage.png)

1. Dans le coin supérieur droit de la page **Builds**, vous pouvez sélectionner **Modifier** pour changer la build actuelle, **File d’attente** pour ajouter une nouvelle build, ou le bouton des points de suspension verticaux ( **&#8942;** ) pour ouvrir un menu avec plus d’options. Sélectionnez **Modifier**.

1. La build accomplit diverses opérations, comme la récupération de sources dans le dépôt, la restauration de dépendances et la publication de sorties utilisées pour les déploiements. À droite, sous **Nom**, remplacez le nom du pipeline de build par un nom plus descriptif. Sélectionnez **Enregistrer et mettre en file d’attente**, puis **Enregistrer**. Entrez un commentaire, puis sélectionnez de nouveau **Enregistrer**.

   ![Page des builds Azure DevOps](_img/azure-devops-project-github/buildpage.png)

1. Pour voir une piste d’audit des modifications que vous avez apportées récemment à la build, sélectionnez l’onglet **Historique**.  Azure DevOps effectue le suivi des modifications apportées au pipeline de build et vous permet de comparer les versions.

1. Sélectionnez l’onglet **Déclencheurs**. Azure DevOps Projects crée automatiquement un déclencheur CI avec quelques paramètres par défaut. Vous pouvez définir des déclencheurs, comme **Activer l’intégration continue**, pour exécuter une build chaque fois que vous validez une modification de code. Vous pouvez également définir des déclencheurs pour planifier l’exécution des builds à des moments spécifiques.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin d’Azure App Service et des ressources associées que vous avez créées dans ce tutoriel, vous pouvez les supprimer. Utilisez la fonctionnalité **Supprimer** du tableau de bord DevOps Projects.

## <a name="next-steps"></a>Étapes suivantes

Lorsque vous avez configuré votre processus CI/CD dans ce tutoriel, vous avez automatiquement créé un pipeline de mise en production et de build dans Azure DevOps Projects. Vous pouvez le modifier afin qu’il réponde aux besoins de votre équipe.

Pour en savoir plus sur le pipeline CI/CD, consultez :

> [!div class="nextstepaction"]
> [Définir votre pipeline de déploiement continu (CD) multiétape](/azure/devops/pipelines/release/define-multistage-release-process?view=vsts)

Pour en savoir plus sur la supervision des applications, consultez :
  
 > [!div class="nextstepaction"]
 > [Présentation d’Azure Monitor](../azure-monitor/overview.md)