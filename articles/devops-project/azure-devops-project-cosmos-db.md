---
title: 'Tutoriel : Déployer des applications Node.js alimentées par Azure Cosmos DB avec Azure DevOps Starter'
description: Azure DevOps Starter permet de démarrer facilement avec Azure. Grâce à DevOps Starter, vous pouvez déployer votre application Node.js alimentée par Azure Cosmos DB sur Windows Web App en quelques étapes rapides.
ms.author: mlearned
ms.manager: gwallace
ms.prod: devops
ms.technology: devops-cicd
ms.topic: tutorial
ms.date: 03/24/2020
author: mlearned
ms.custom: devx-track-js
ms.openlocfilehash: de98d9db515e4ed88759efa7f22dddcca06151c8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91854679"
---
# <a name="deploy-nodejs-apps-powered-by-azure-cosmos-db-with-devops-starter"></a>Déployer des applications Node.js alimentées par Azure Cosmos DB avec DevOps Starter

Azure DevOps Starter offre une expérience rationalisée dans laquelle vous pouvez créer un pipeline d’intégration continue (CI) et de déploiement continu (CD) dans Azure. Pour ce faire, utilisez votre code et votre référentiel Git existants ou sélectionnez un exemple d’application.

De plus, DevOps Starter :

* Crée automatiquement des ressources Azure, telles qu’Azure Cosmos DB, Azure Application Insights, Azure App Service et des plans App Service

* Crée et configure un pipeline de mise en production CI/CD dans Azure DevOps

Ce didacticiel présente les procédures suivantes :

> [!div class="checklist"]
> * Utiliser DevOps Starter pour déployer une application Node.js alimentée par Azure Cosmos DB
> * Configurer Azure DevOps et un abonnement Azure
> * Examiner Azure Cosmos DB
> * Examiner le pipeline CI
> * Examiner le pipeline CD
> * Valider les modifications dans Git et les déployer automatiquement dans Azure
> * Nettoyer les ressources

## <a name="prerequisites"></a>Prérequis

Vous avez besoin d’un abonnement Azure. Vous pouvez en obtenir un gratuitement via [Visual Studio Dev Essentials](https://visualstudio.microsoft.com/dev-essentials/).

## <a name="use-devops-starter-to-deploy-nodejs-app"></a>Utiliser DevOps Starter pour déployer l’application Node.js

DevOps Starter crée un pipeline CI/CD dans Azure Pipelines. Vous pouvez créer une organisation Azure DevOps ou utiliser une organisation existante. DevOps Starter crée également des ressources Azure, telles qu’Azure Cosmos DB, Application Insights et App Service, et des plans App Service dans l’abonnement Azure de votre choix.

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Dans la zone de recherche, tapez, puis sélectionnez **DevOps Starter**. Cliquez sur le **signe +** pour créer une ressource.

    ![Tableau de bord DevOps Starter](_img/azure-devops-starter-aks/search-devops-starter.png)

1. Sélectionnez **Node.js** comme runtime, puis cliquez sur **Suivant**. Sous **Choisir un framework d’application**, sélectionnez **Express.js**.

1. Activez la section **Ajouter une base de données** pour **Cosmos DB**, puis cliquez sur **Suivant**.

    ![Ajouter une base de données](_img/azure-devops-project-cosmos-db/add-database.png)

    Azure DevOps Starter prend en charge différentes infrastructures d’application, telles qu’**Express.js**, **Sample Node.js app** et **Sail.js**. Dans ce tutoriel, nous allons utiliser **Express.js**.

1. Sélectionnez un service Azure pour déployer l’application, puis cliquez sur **Suivant**. Vos options incluent Windows Web App, Azure Kubernetes Service et Azure Web App pour conteneurs. Dans ce tutoriel, nous allons utiliser **Windows Web App**.

## <a name="configure-azure-devops-and-azure-subscription"></a>Configurer Azure DevOps et l’abonnement Azure

1. Indiquez un nom pour votre projet Azure DevOps.

1. Créez une organisation Azure DevOps ou sélectionnez une organisation existante.

1. Sélectionnez votre abonnement Azure.

1. Pour voir d’autres paramètres de configuration Azure ou pour identifier le niveau tarifaire et l’emplacement, cliquez sur **Paramètres supplémentaires**. Ce volet affiche différentes options pour la configuration du niveau tarifaire et de l’emplacement des services Azure.

1. Quittez la zone de configuration Azure, puis sélectionnez **Terminé**.

1. Le processus se termine au bout de quelques minutes. Un exemple d’application Node.js est configuré dans un référentiel Git dans votre organisation Azure DevOps. Ensuite, Azure Cosmos DB, App Service, un plan App Service et des ressources Application Insights sont créés, ainsi qu’un pipeline CI/CD. Votre application est ensuite déployée sur Azure.

   Une fois tous ces processus terminés, le tableau de bord Azure DevOps Starter s’affiche dans le Portail Azure. Vous pouvez également accéder au tableau de bord DevOps Starter directement à partir de **Toutes les ressources** dans le portail Azure.

   Ce tableau de bord permet de visualiser votre référentiel de code Azure DevOps, votre pipeline CI/CD et votre base de données Azure Cosmos DB. Vous pouvez configurer d’autres options de CI/CD dans votre pipeline Azure DevOps. Sur la droite du tableau de bord, sélectionnez **Azure Cosmos DB** pour afficher ces options.

## <a name="examine-azure-cosmos-db"></a>Examiner Azure Cosmos DB

DevOps Starter configure automatiquement Azure Cosmos DB, que vous pouvez explorer et personnaliser. Pour vous familiariser avec la base de données Azure Cosmos DB, effectuez les étapes suivantes :

1. Accédez au tableau de bord DevOps Starter.

    ![Tableau de bord DevOps Projects](_img/azure-devops-project-cosmos-db/devops-starter-dashboard.png)

1. Sur la droite, sélectionnez Azure Cosmos DB. Un volet s’ouvre pour la base de données Azure Cosmos DB. À partir de cette vue, vous pouvez effectuer diverses actions, par exemple superviser les opérations et effectuer des recherches dans les journaux.

    ![Volet Azure Cosmos DB](_img/azure-devops-project-cosmos-db/cosmos-db.png)

## <a name="examine-the-ci-pipeline"></a>Examiner le pipeline CI

DevOps Starter configure automatiquement un pipeline CI/CD dans votre organisation Azure DevOps. Vous pouvez explorer et personnaliser le pipeline. Pour vous familiariser avec celui-ci, procédez comme suit :

1. Accédez au tableau de bord DevOps Starter.

1. Cliquez sur le lien hypertexte sous **Build**. Un onglet du navigateur affiche le pipeline de build de votre nouveau projet.

    ![Volet Build](_img/azure-devops-project-cosmos-db/build.png)

1. Sélectionnez **Modifier**. Dans ce volet, vous pouvez examiner les différentes tâches de votre pipeline de build. La build effectue différentes tâches, telles que l’extraction du code source à partir du dépôt git, la génération de l’application, l’exécution de tests unitaires et la publication des sorties utilisées pour les déploiements.

1. Sélectionnez **Déclencheurs**. DevOps Starter crée automatiquement un déclencheur CI tandis que chaque validation dans le dépôt lance une nouvelle build. Vous pouvez choisir d’inclure ou d’exclure des branches dans le processus CI.

1. Sélectionnez **Rétention**. En fonction de votre scénario, vous pouvez spécifier des stratégies pour conserver ou supprimer un certain nombre de builds.

1. En haut du pipeline de build, sélectionnez le nom du pipeline de build.

1. Remplacez le nom de votre pipeline de build par un nom plus descriptif, puis sélectionnez **Enregistrer** dans le menu déroulant **Enregistrer et mettre en file d’attente**.

1. Sous le nom de votre pipeline de build, sélectionnez **Historique**. Ce volet montre une piste d’audit des modifications que vous avez apportées récemment à la build. Azure DevOps suit les modifications apportées au pipeline de build et permet de comparer les versions.

## <a name="examine-the-cd-release-pipeline"></a>Examiner le pipeline de mise en production CD

DevOps Starter crée et configure automatiquement les étapes nécessaires au déploiement entre votre organisation Azure DevOps et votre abonnement Azure. Ces étapes comprennent la configuration d’une connexion au service Azure pour authentifier Azure DevOps auprès de votre abonnement Azure. L’automatisation crée également un pipeline de mise en production, qui fournit le déploiement continu vers Azure. Pour en savoir plus sur le pipeline de mise en production, suivez ces étapes :

1. Accédez à **Pipelines**, puis sélectionnez **Mises en production**.

1. Sélectionnez **Modifier**.

1. Sous **Artefacts**, sélectionnez **Déposer**. Le pipeline de build que vous avez examiné aux étapes précédentes produit la sortie qui est utilisée pour l’artefact.

1. À droite de l’icône **Déposer**, sélectionnez **Déclencheur de déploiement continu**. Ce pipeline de mise en production est doté d’un déclencheur de déploiement continu activé, qui exécute un déploiement chaque fois qu’un nouvel artefact de build est disponible. Vous pouvez désactiver le déclencheur afin que vos déploiements s’exécutent manuellement.

1. À droite, sélectionnez la section **Afficher les mises en production** pour voir l’historique des mises en production.

1. Sélectionnez la mise en production pour afficher le pipeline. Sélectionnez n’importe quel environnement pour vérifier le résumé, les validations de la mise en production et les éléments de travail associés.

1. Sélectionnez **Validations**. Cette vue montre les validations de code qui sont associées à ce déploiement. Comparez les mises en production pour afficher les différences de validation entre les déploiements.

1. Sélectionnez **Afficher les journaux**. Les journaux d’activité contiennent des informations utiles sur le processus de déploiement. Vous pouvez les voir à la fois pendant et après les déploiements.

## <a name="commit-code-changes-and-execute-the-cicd-pipeline"></a>Valider les modifications de code et exécuter le pipeline CI/CD

> [!NOTE]
> La procédure suivante teste le pipeline CI/CD en apportant une modification de texte simple.

Vous êtes maintenant prêt à collaborer avec une équipe sur votre application, par le biais d’un processus d’intégration et de déploiement continus (CI/CD) qui déploie votre dernier travail sur votre App Service. Chaque modification apportée au dépôt Git démarre une build dans Azure DevOps tandis qu’un pipeline CD exécute un déploiement dans Azure. Suivez la procédure décrite dans cette section, ou utilisez une autre technique pour valider les modifications dans votre dépôt. Par exemple, vous pouvez cloner le dépôt Git dans votre outil ou environnement de développement intégré favori, puis envoyer (push) les modifications vers ce dépôt.

1. Dans le menu Azure DevOps, sélectionnez **Référentiels**, puis **Fichiers**. Accédez ensuite à votre référentiel.

1. Le référentiel contient déjà du code basé sur le langage de l’application que vous avez sélectionné lors du processus de création. Ouvrez le fichier **Application/views/index.pug**.

1. Sélectionnez **Modifier**, puis apportez une modification à la **ligne numéro 15**. Par exemple, vous pouvez la modifier avec le texte « Mon premier déploiement sur Azure App Service avec Azure Cosmos DB ».

1. En haut à droite, sélectionnez **Valider**, puis **Valider** à nouveau pour envoyer votre modification.

     Au bout de quelques secondes, une build est démarrée dans Azure DevOps tandis qu’une mise en production est exécutée pour déployer les modifications. Supervisez l’état de la build sur le tableau de bord DevOps Starter, ou dans le navigateur avec votre organisation Azure DevOps.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Vous pouvez supprimer les ressources associées que vous avez créées lorsque vous n’en avez plus besoin. Utilisez la fonctionnalité **Supprimer** du tableau de bord DevOps Starter.

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez le modifier afin qu’il réponde aux besoins de votre équipe. Vous pouvez également utiliser ce modèle CI/CD comme modèle pour vos autres pipelines. Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Utiliser DevOps Starter pour déployer une application Node.js alimentée par Azure Cosmos DB
> * Configurer Azure DevOps et un abonnement Azure 
> * Examiner Azure Cosmos DB
> * Examiner le pipeline CI
> * Examiner le pipeline CD
> * Valider les modifications dans Git et les déployer automatiquement dans Azure
> * Nettoyer les ressources

Consultez [Définir votre pipeline de déploiement continu (CD) multiétape](/azure/devops/pipelines/release/define-multistage-release-process?view=azure-devops&viewFallbackFrom=vsts) pour plus d’informations et pour connaître les étapes suivantes.