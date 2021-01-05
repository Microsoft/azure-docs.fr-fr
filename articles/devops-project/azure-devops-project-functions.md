---
title: 'Tutoriel : Déployer des applications ASP.NET sur Azure Functions avec Azure DevOps Starter'
description: Azure DevOps Starter facilite la prise en main d’Azure. Avec DevOps Starter, vous pouvez déployer votre application ASP.NET sur Azure Functions en quelques étapes rapides.
ms.author: mlearned
ms.manager: gwallace
ms.prod: devops
ms.technology: devops-cicd
ms.topic: tutorial
ms.date: 03/24/2020
author: mlearned
ms.custom: devx-track-csharp
ms.openlocfilehash: 3277fefb5615fd492b2859b741ef8e6f991eaf23
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91355472"
---
# <a name="deploy-to-azure-functions-with-devops-starter"></a>Effectuer un déploiement sur Azure Functions avec DevOps Starter

Azure DevOps Starter offre une expérience simplifiée dans laquelle vous pouvez apporter vos code et dépôt Git existants ou choisir un exemple d’application pour créer un pipeline d’intégration continue (CI) et de livraison continue (CD) dans Azure.

En outre, DevOps Starter :

* Crée automatiquement des ressources Azure comme des fonctions Azure.

* Crée et configure un pipeline de mise en production dans Azure DevOps pour CI/CD.

Ce didacticiel présente les procédures suivantes :

> [!div class="checklist"]
>* Utiliser DevOps Starter pour déployer une application ASP.NET sur Azure Functions
>* Configurer Azure DevOps et un abonnement Azure
>* Examiner la fonction Azure
>* Examiner le pipeline CI
>* Examiner le pipeline CD
>* Valider les modifications dans Git et les déployer automatiquement dans Azure
>* Nettoyer les ressources

Actuellement, les runtimes pris en charge pour les fonctions sont **.NET** et **Node. js**. Nous utilisons le runtime .NET dans le cadre de ce tutoriel pour le déploiement sur Azure Functions. 

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Vous pouvez en obtenir un gratuit par le biais de [Visual Studio Dev Essentials](https://visualstudio.microsoft.com/dev-essentials/)

## <a name="use-devops-starter-to-deploy-an-aspnet-app-to-azure-functions"></a>Utiliser DevOps Starter pour déployer une application ASP.NET sur Azure Functions

DevOps Starter crée un pipeline CI/CD dans Azure Pipelines. Vous pouvez créer une organisation Azure DevOps ou utiliser une organisation existante. DevOps Projects crée également des ressources Azure, telles qu’un hub IoT, dans l’abonnement Azure de votre choix.

1. Connectez-vous au [portail Azure](https://portal.azure.com)

1. Dans la zone de recherche, tapez, puis sélectionnez **DevOps Starter**. Cliquez sur le **signe +** pour créer une ressource.

    ![Tableau de bord DevOps Starter](_img/azure-devops-starter-aks/search-devops-starter.png)

1. Sélectionnez **.NET**, puis sélectionnez **Suivant**. Sous **Choisir un framework d’application**, sélectionnez **ASP.NET** et cliquez sur **Suivant**.

1. Sélectionnez **Application de fonction**, puis **Suivant**.

## <a name="configure-azure-devops-and-azure-subscription"></a>Configurer Azure DevOps et l’abonnement Azure

1. Indiquez un nom pour votre projet Azure DevOps.

1. Créez une organisation Azure DevOps ou sélectionnez une organisation existante.

1. Sélectionnez votre abonnement Azure.

1. Pour voir d’autres paramètres de configuration Azure et pour identifier le niveau tarifaire et l’emplacement, cliquez sur **Paramètres supplémentaires**. Ce volet regroupe différentes options pour la configuration du niveau tarifaire et de l’emplacement des services Azure.

1. Quittez la zone de configuration Azure, puis sélectionnez **Terminé**.

1. Le processus est achevé au bout de quelques minutes. Un exemple d’application ASP.NET est configuré dans un dépôt Git de votre organisation Azure DevOps, une application de fonction, Application Insights est créé, un pipeline CI/CD est exécuté, et votre application est déployée sur Azure.

   Une fois toutes ces opérations terminées, le tableau de bord Azure DevOps Starter s’affiche dans le portail Azure. Vous pouvez également accéder au tableau de bord DevOps Starter directement à partir de **Toutes les ressources** dans le portail Azure.

   Ce tableau de bord permet de visualiser votre dépôt de code Azure DevOps, le pipeline CI/CD et votre fonction Azure. Vous pouvez configurer d’autres options de CI/CD dans votre pipeline Azure DevOps. Sur la droite, sélectionnez **Application de fonction** pour la voir.

## <a name="examine-the-function-app"></a>Examiner l’application de fonction

DevOps Starter configure automatiquement une application de fonction, que vous pouvez explorer et personnaliser. Pour vous familiariser avec l’application de fonction, effectuez les étapes suivantes :

1. Accédez au tableau de bord DevOps Starter.

    ![Tableau de bord DevOps Projects](_img/azure-devops-project-functions/fapp-dashboard.png)

1. Sur la droite, sélectionnez l’application de fonction. Un volet s’ouvre pour l’application de fonction. À partir de cette vue, vous pouvez effectuer diverses actions, par exemple superviser les opérations et effectuer des recherches dans les journaux.

    ![Function App](_img/azure-devops-project-functions/function-app.png)

## <a name="examine-the-ci-pipeline"></a>Examiner le pipeline CI

DevOps Starter configure automatiquement un pipeline CI/CD dans votre organisation Azure DevOps. Vous pouvez explorer et personnaliser le pipeline. Pour vous familiariser avec celui-ci, procédez comme suit :

1. Accédez au tableau de bord DevOps Starter.

1. Cliquez sur le lien hypertexte sous **Build**. Un onglet du navigateur affiche le pipeline de build de votre nouveau projet.

    ![Capture d’écran montrant le tableau de bord DevOps Starter dans le portail Azure avec une flèche pointant vers un lien sous Build dans le pipeline CI/CD.](_img/azure-devops-project-functions/build.png)

1. Sélectionnez **Modifier**. Dans ce volet, vous pouvez examiner les différentes tâches de votre pipeline de build. La build effectue différentes tâches, telles que l’extraction du code source à partir du dépôt git, la génération de l’application, l’exécution de tests unitaires et la publication des sorties utilisées pour les déploiements.

1. Sélectionnez **Déclencheurs**. DevOps Starter crée automatiquement un déclencheur CI tandis que chaque validation dans le dépôt lance une nouvelle build. Si vous le souhaitez, vous pouvez choisir d’inclure ou d’exclure des branches dans le processus CI.

1. Sélectionnez **Rétention**. En fonction de votre scénario, vous pouvez spécifier des stratégies pour conserver ou supprimer un certain nombre de builds.

1. En haut du pipeline de build, sélectionnez le nom du pipeline de build.

1. Remplacez le nom de votre pipeline de build par un nom plus descriptif, puis sélectionnez **Enregistrer** dans le menu déroulant **Enregistrer et mettre en file d’attente**.

1. Sous le nom de votre pipeline de build, sélectionnez **Historique**. Ce volet montre une piste d’audit des modifications que vous avez apportées récemment à la build. Azure DevOps suit les modifications apportées au pipeline de build et permet de comparer les versions.

## <a name="examine-the-cd-release-pipeline"></a>Examiner le pipeline de mise en production CD

DevOps Starter crée et configure automatiquement les étapes nécessaires au déploiement entre votre organisation Azure DevOps et votre abonnement Azure. Ces étapes comprennent la configuration d’une connexion au service Azure pour authentifier Azure DevOps auprès de votre abonnement Azure. L’automatisation crée également un pipeline de mise en production, qui fournit le déploiement continu vers Azure. Pour en savoir plus sur le pipeline de mise en production, suivez ces étapes :

1. Accédez à **Pipelines | Mises en production**.

1. Cliquez sur **Modifier**.

1. Sous **Artefacts**, sélectionnez **Déposer**. Le pipeline de build que vous avez examiné aux étapes précédentes produit la sortie qui est utilisée pour l’artefact.

1. À droite de l’icône **Déposer**, sélectionnez **Déclencheur de déploiement continu**. Ce pipeline de mise en production est doté d’un déclencheur de déploiement continu activé qui exécute un déploiement chaque fois qu’un nouvel artefact de build est disponible. Si vous le souhaitez, vous pouvez désactiver le déclencheur afin que vos déploiements nécessitent une exécution manuelle.

1. À droite, sélectionnez **Afficher les mises en production** pour voir l’historique des mises en production.

1. Cliquez sur la mise en production pour afficher le pipeline. Cliquez sur n’importe quel environnement pour vérifier **la Synthèse, les Validations** de la mise en production et les **Éléments de travail associés**.

1. Sélectionnez **Validations**. Cette vue montre les validations de code qui sont associées à ce déploiement. Comparez les mises en production pour afficher les différences de validation entre les déploiements.

1. Sélectionnez **Afficher les journaux**. Les journaux d’activité contiennent des informations utiles sur le processus de déploiement. Vous pouvez les voir à la fois pendant et après les déploiements.

## <a name="commit-code-changes-and-execute-cicd"></a>Valider les modifications de code et exécuter CI/CD

> [!NOTE]
> La procédure suivante teste le pipeline CI/CD en apportant une modification de texte simple.

Vous êtes maintenant prêt à collaborer avec une équipe sur votre application, par le biais d’un processus d’intégration et de livraison continus (CI/CD) qui déploie automatiquement votre dernier travail sur votre fonction Azure. Chaque modification apportée au dépôt Git démarre une build dans Azure DevOps tandis qu’un pipeline CD exécute un déploiement dans Azure. Suivez la procédure décrite dans cette section, ou utilisez une autre technique pour valider les modifications dans votre dépôt. Par exemple, vous pouvez cloner le dépôt Git dans votre outil ou environnement de développement intégré favori, puis envoyer (push) les modifications vers ce dépôt.

1. Dans le menu Azure DevOps, sélectionnez **Dépôts | Fichiers**, puis accédez à votre dépôt.

1. Le dépôt contient déjà du code appelé **SampleFunctionApp** basé sur le langage de l’application que vous avez sélectionné lors du processus de création. Ouvrez le fichier **application/SampleFunctionApp/fonction1.cs**.

1. Sélectionnez **Modifier**, puis apportez une modification à la **ligne numéro 31**. Par exemple, vous pouvez la mettre à jour pour qu’elle indique **Bonjour. Bienvenue dans Azure Functions via DevOps Starter**

1. En haut à droite, sélectionnez **Valider**, puis **Valider** à nouveau pour transmettre votre modification.

1. Ouvrez le fichier **Application/SampleFunctionApp.Test/Function1TestRunner.cs**. 

1. Sélectionnez **Modifier**, puis apportez une modification à la **ligne numéro 21**. Par exemple, vous pouvez la mettre à jour pour qu’elle indique **Bonjour. Bienvenue dans Azure Functions via Azure DevOps Starter**.

     Au bout de quelques instants, une build est démarrée dans Azure DevOps tandis qu’une mise en production est exécutée pour déployer les modifications. Supervisez l’état de la build sur le tableau de bord DevOps Starter, ou dans le navigateur avec votre organisation Azure DevOps.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’en avez plus besoin, vous pouvez supprimer les ressources associées que vous avez créées. Utilisez la fonctionnalité **Supprimer** du tableau de bord DevOps Starter.

## <a name="next-steps"></a>Étapes suivantes

Si vous le souhaitez, vous pouvez modifier ces pipelines de build et de mise en production afin qu’ils répondent aux besoins de votre équipe. Vous pouvez également utiliser ce modèle CI/CD comme modèle pour vos autres pipelines. Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Utiliser DevOps Starter pour déployer une application ASP.NET Core sur Azure Functions
> * Configurer Azure DevOps et un abonnement Azure 
> * Examiner la fonction Azure
> * Examiner le pipeline CI
> * Examiner le pipeline CD
> * Valider les modifications dans Git et les déployer automatiquement dans Azure
> * Nettoyer les ressources

