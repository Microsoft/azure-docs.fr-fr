---
title: 'Tutoriel : Déployer votre application ASP.NET et votre code Azure SQL Database en utilisant Azure DevOps Starter'
description: DevOps Starter facilite la prise en main d’Azure. Avec DevOps Starter, vous pouvez déployer votre application ASP.NET et votre code Azure SQL Database en quelques étapes rapides.
ms.author: mlearned
ms.manager: gwallace
ms.prod: devops
ms.technology: devops-cicd
ms.topic: tutorial
ms.date: 03/24/2020
author: mlearned
ms.openlocfilehash: 6a1af644bbd88af5c513ed9a43ce154f285c06df
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91856005"
---
# <a name="tutorial-deploy-your-aspnet-app-and-azure-sql-database-code-by-using-azure-devops-starter"></a>Tutoriel : Déployer votre application ASP.NET et votre code Azure SQL Database en utilisant Azure DevOps Starter

Azure DevOps Starter offre une expérience simplifiée dans laquelle vous pouvez apporter vos code et dépôt Git existants ou choisir un exemple d’application pour créer un pipeline d’intégration continue (CI) et de livraison continue (CD) dans Azure. 

De plus, DevOps Starter :
* Crée automatiquement des ressources Azure, comme une base de données dans Azure SQL Database.
* Crée et configure un pipeline de mise en production dans Azure Pipelines qui inclut un pipeline de build pour CI
* Configurer un pipeline de mise en production pour CD 
* Crée une ressource Azure Application Insights pour la supervision

Ce didacticiel présente les procédures suivantes :

> [!div class="checklist"]
> * Utiliser Azure DevOps Starter pour déployer votre application ASP.NET et votre code Azure SQL Database
> * Configurer Azure DevOps et un abonnement Azure 
> * Examiner le pipeline CI
> * Examiner le pipeline CD
> * Valider les modifications apportées à Azure Repos et les déployer automatiquement dans Azure
> * Connexion à Azure SQL Database 
> * Nettoyer les ressources

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Vous pouvez en obtenir un gratuit via [Visual Studio Dev Essentials](https://visualstudio.microsoft.com/dev-essentials/).

## <a name="create-a-project-in-devops-projects-for-an-aspnet-app-and-azure-sql-database"></a>Créer un projet dans DevOps Projects pour une application ASP.NET et Azure SQL Database

DevOps Starter crée un pipeline CI/CD dans Azure Pipelines. Vous pouvez créer une organisation Azure DevOps ou utiliser une organisation existante. DevOps Starter crée également des ressources Azure, comme Azure SQL Database, dans l’abonnement Azure de votre choix.

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Dans la zone de recherche, tapez, puis sélectionnez **DevOps Starter**. Cliquez sur le **signe +** pour créer une ressource.

    ![Tableau de bord DevOps Starter](_img/azure-devops-starter-aks/search-devops-starter.png)

1. Sélectionnez **.NET**, puis sélectionnez **Suivant**.

1. Sous **Choisir un framework d’application**, sélectionnez **ASP.NET**.

1. Sélectionnez **Ajouter une base de données**, puis **Suivant**. Le framework d’application, que vous avez choisi à l’étape précédente, détermine le type de cible de déploiement de service Azure disponible ici. 
    
1. Sélectionnez **Suivant**.

## <a name="configure-azure-devops-and-an-azure-subscription"></a>Configurer Azure DevOps et un abonnement Azure

1. Créez une organisation Azure DevOps ou sélectionnez une organisation existante. 

1. Indiquez un nom pour votre projet Azure DevOps. 

1. Sélectionnez les services de votre abonnement Azure. Si vous le souhaitez, pour afficher d’autres paramètres de configuration Azure et identifier le nom d’utilisateur dans la section **Détails de connexion au serveur de base de données**, vous pouvez sélectionner **Modifier**. Enregistrez le nom d’utilisateur pour le réutiliser dans les étapes suivantes de ce tutoriel. Si vous effectuez cette étape facultative, quittez la zone de configuration Azure avant de sélectionner **Terminé**.
 
1. Sélectionnez **Terminé**. Après quelques minutes, le tableau de bord DevOps Starter s’ouvre dans le portail Azure. Vous pouvez également accéder au tableau de bord directement à partir de **Toutes les ressources** dans le portail Azure. À droite, sélectionnez **Parcourir** pour voir votre application en cours d’exécution.
    
## <a name="examine-the-ci-pipeline"></a>Examiner le pipeline CI

DevOps Starter configure automatiquement un pipeline CI/CD complet dans Azure Repos. Vous pouvez explorer et personnaliser le pipeline. Pour vous familiariser avec le pipeline de build d’Azure DevOps, effectuez les étapes suivantes :

1. En haut du tableau de bord DevOps Starter, sélectionnez **Pipelines de build**. Un onglet du navigateur affiche le pipeline de build de votre nouveau projet.

1. Pointez sur le champ **État**, puis sélectionnez les points de suspension (...). Un menu affiche plusieurs options, telles que de la mise en file d’attente d’une nouvelle build, la mise en pause d’une build et la modification du pipeline de build.

1. Sélectionnez **Modifier**.

1. Dans ce volet, vous pouvez examiner les différentes tâches de votre pipeline de build. La build effectue diverses opérations, comme la récupération (fetch) de sources dans le dépôt Git, la restauration de dépendances et la publication de sorties utilisées pour les déploiements.

1. En haut du pipeline de build, sélectionnez le nom du pipeline de build.

1. Remplacez le nom de votre pipeline de build par un nom plus descriptif, sélectionnez **Enregistrer et mettre en file d’attente**, puis sélectionnez **Enregistrer**.

1. Sous le nom de votre pipeline de build, sélectionnez **Historique**. Ce volet montre une piste d’audit des modifications que vous avez apportées récemment à la build. Azure Pipelines suit les modifications apportées au pipeline de build et permet de comparer les versions.

1. Sélectionnez **Déclencheurs**. DevOps Starter crée automatiquement un déclencheur CI, et chaque validation dans le dépôt lance une nouvelle build. Si vous le souhaitez, vous pouvez choisir d’inclure ou d’exclure des branches dans le processus CI.

1. Sélectionnez **Rétention**. En fonction de votre scénario, vous pouvez spécifier des stratégies pour conserver ou supprimer un certain nombre de builds.

## <a name="examine-the-cd-pipeline"></a>Examiner le pipeline CD

DevOps Starter crée et configure automatiquement les étapes nécessaires au déploiement entre votre organisation Azure DevOps et votre abonnement Azure. Ces étapes comprennent la configuration d’une connexion au service Azure pour authentifier Azure DevOps auprès de votre abonnement Azure. L’automatisation crée également un pipeline CD qui fournit le déploiement continu à la machine virtuelle Azure. Pour en savoir plus sur le pipeline CD Azure DevOps, effectuez les étapes suivantes :

1. Sélectionnez **Build et mise en production**, puis **Mises en production**. DevOps Starter crée un pipeline de mise en production pour gérer les déploiements sur Azure.

1. Sélectionnez les points de suspension (...) en regard de votre pipeline de mise en production, puis sélectionnez **Modifier**. Le pipeline de mise en production contient un *pipeline* qui définit le processus de mise en production.

1. Sous **Artefacts**, sélectionnez **Déposer**. Le pipeline de build que vous avez examiné aux étapes précédentes produit la sortie qui est utilisée pour l’artefact. 

1. À droite de l’icône **Déposer**, sélectionnez **Déclencheur de déploiement continu**. Ce pipeline de mise en production est doté d’un déclencheur de déploiement continu activé qui exécute un déploiement chaque fois qu’un nouvel artefact de build est disponible. Si vous le souhaitez, vous pouvez désactiver le déclencheur afin que vos déploiements nécessitent une exécution manuelle. 

    DevOps Starter configure un mot de passe SQL aléatoire et l’utilise pour le pipeline de mise en production.
    
1. À gauche, sélectionnez **Variables**. 

   > [!NOTE]
   > Effectuez l’étape suivante uniquement si vous avez modifié le mot de passe SQL Server. Il existe une seule variable de mot de passe.
  
1. En regard de la zone **Valeur**, sélectionnez l’icône de cadenas, entrez le nouveau mot de passe, puis sélectionnez **Enregistrer**.

1. À gauche, sélectionnez **Tâches**, puis sélectionnez votre environnement. Les tâches correspondent aux activités exécutées par votre processus de déploiement ; elles sont regroupées par phases. Ce pipeline de mise en production est constitué d’une seule phase contenant une tâche de *déploiement d’Azure App Service* et de *déploiement d’Azure SQL Database*.

1. Sélectionnez la tâche *Exécuter Azure SQL* et examinez les diverses propriétés qui sont utilisées pour le déploiement SQL. Sous **Package de déploiement**, la tâche utilise un fichier *SQL DACPAC*.

1. À droite, sélectionnez **Afficher les mises en production** pour voir l’historique des mises en production.

1. Sélectionnez les points de suspension (...) en regard d’une mise en production, puis sélectionnez **Ouvrir**. Vous pouvez explorer plusieurs menus, comme un résumé des mises en production, les éléments de travail associés et les tests.

1. Sélectionnez **Validations**. Cette vue montre les validations de code qui sont associées à ce déploiement. Comparez les mises en production pour afficher les différences de validation entre les déploiements.

1. Sélectionnez **Journaux d’activité**. Les journaux d’activité contiennent des informations utiles sur le processus de déploiement. Vous pouvez les voir à la fois pendant et après les déploiements.

## <a name="commit-changes-to-azure-repos-and-automatically-deploy-them-to-azure"></a>Valider les modifications apportées à Azure Repos et les déployer automatiquement dans Azure 

 > [!NOTE]
 > La procédure suivante teste le pipeline CI/CD avec une modification de texte simple. Si vous le souhaitez, pour tester le processus de déploiement SQL, vous pouvez apporter une modification du schéma SQL Server à la table.

Vous êtes maintenant prêt à collaborer avec une équipe sur votre application, par le biais d’un processus d’intégration et de déploiement continus (CI/CD) qui déploie automatiquement votre dernier travail sur votre site web. Chaque modification apportée au dépôt Git démarre une build dans Azure DevOps tandis qu’un pipeline CD exécute un déploiement dans Azure. Suivez la procédure décrite dans cette section, ou utilisez une autre technique pour valider les modifications dans votre dépôt. Les modifications de code lancent le processus CI/CD et déploient automatiquement vos modifications dans Azure.

1. Dans le volet gauche, sélectionnez **Code**, puis accédez à votre dépôt.

1. Accédez au répertoire *SampleWebApplication\Views\Home*, sélectionnez les points de suspension (...) situés en regard du fichier *Index.cshtml*, puis sélectionnez **Modifier**. 

1. Modifiez le fichier, par exemple en ajoutant du texte dans une des balises div. 

1. En haut à droite, sélectionnez **Valider**, puis **Valider** à nouveau pour transmettre votre modification. Au bout de quelques instants, une build est démarrée dans Azure DevOps tandis qu’une mise en production est exécutée pour déployer les modifications. Supervisez l’état de la build dans le tableau de bord DevOps Starter, ou dans le navigateur avec votre organisation Azure DevOps.

1. Une fois la mise en production terminée, actualisez votre application pour vérifier vos modifications.

## <a name="connect-to-azure-sql-database"></a>Connexion à Azure SQL Database

Vous devez disposer des autorisations appropriées pour vous connecter à Azure SQL Database.

1. Dans le tableau de bord DevOps Starter, sélectionnez **SQL Database** pour accéder à la page de gestion de SQL Database.
   
1. Sélectionnez **Définir le pare-feu du serveur**, puis **Ajouter une adresse IP de client**. 

1. Sélectionnez **Enregistrer**. Votre adresse IP de client est désormais autorisée à accéder à la ressource Azure SQL Server.

1. Revenez au volet **SQL Database**. 

1. À droite, sélectionnez le nom du serveur pour accéder à la page de configuration pour **SQL Server**.

1. Sélectionnez **Réinitialiser le mot de passe**, entrez un mot de passe pour la connexion d’administrateur SQL Server, puis sélectionnez **Enregistrer**. Veillez à conserver ce mot de passe pour l’utiliser ultérieurement dans ce tutoriel.

    Si vous le souhaitez, vous pouvez désormais utiliser les outils clients, comme SQL Server Management Studio ou Visual Studio, pour vous connecter à SQL Server et à Azure SQL Database. Utilisez la propriété **Nom du serveur** pour vous connecter.

    Si vous n’avez pas modifié le nom d’utilisateur de la base de données lors de la configuration initiale du projet dans DevOps Projects, votre nom d’utilisateur correspond à la partie locale de votre adresse e-mail. Par exemple, si votre adresse e-mail est *johndoe\@microsoft.com*, votre nom d’utilisateur est *johndoe*.

   > [!NOTE]
   > Si vous changez votre mot de passe pour la connexion SQL, vous devez changer le mot de passe dans la variable du pipeline de mise en production, comme décrit à la section [Examiner le pipeline CD](#examine-the-cd-pipeline).

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous êtes en phase de test, vous pouvez éviter une hausse des frais de facturation en nettoyant vos ressources. Quand elles ne sont plus nécessaires, vous pouvez supprimer Azure SQL Database et les ressources associées que vous avez créées dans ce tutoriel. Pour ce faire, utilisez la fonctionnalité **Supprimer** du tableau de bord DevOps Starter.

> [!IMPORTANT]
> La procédure suivante supprime définitivement les ressources. La fonctionnalité *Supprimer* détruit les données qui sont créées par le projet dans DevOps Starter, à la fois dans Azure et dans Azure DevOps, et vous ne pourrez plus les récupérer. Utilisez cette procédure uniquement après avoir lu attentivement les invites.

1. Dans le Portail Azure, accédez au tableau de bord DevOps Starter.
2. En haut à droite, sélectionnez **Supprimer**. 
3. À l’invite, sélectionnez **Oui** pour *supprimer définitivement* les ressources.

## <a name="next-steps"></a>Étapes suivantes

Si vous le souhaitez, vous pouvez modifier ces pipelines de build et de mise en production afin qu’ils répondent aux besoins de votre équipe. Vous pouvez également utiliser ce modèle CI/CD comme modèle pour vos autres pipelines. Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Utiliser Azure DevOps Starter pour déployer votre application ASP.NET et votre code Azure SQL Database
> * Configurer Azure DevOps et un abonnement Azure 
> * Examiner le pipeline CI
> * Examiner le pipeline CD
> * Valider les modifications apportées à Azure Repos et les déployer automatiquement dans Azure
> * Connexion à Azure SQL Database 
> * Nettoyer les ressources

Pour en savoir plus sur le pipeline CI/CD, consultez :

> [!div class="nextstepaction"]
> [Définir votre pipeline de déploiement continu (CD) multiétape](/azure/devops/pipelines/release/define-multistage-release-process?view=vsts)

## <a name="videos"></a>Vidéos

> [!VIDEO https://channel9.msdn.com/Events/Build/2018/BRK3308/player]