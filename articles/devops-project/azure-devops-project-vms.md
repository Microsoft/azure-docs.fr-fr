---
title: 'Tutoriel : Déployer votre application ASP.NET sur des machines virtuelles Azure avec Azure DevOps Starter'
description: DevOps Starter facilite la prise en main d’Azure et le déploiement de votre application ASP.NET sur des machines virtuelles Azure en quelques étapes rapides.
ms.author: mlearned
manager: gwallace
ms.prod: devops
ms.technology: devops-cicd
ms.topic: tutorial
ms.date: 03/24/2020
author: mlearned
ms.openlocfilehash: ce11fe5b65cd49cc880713eb4e47b081b6f3b44f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91855971"
---
# <a name="tutorial-deploy-your-aspnet-app-to-azure-virtual-machines-by-using-azure-devops-starter"></a>Tutoriel : Déployer votre application ASP.NET sur des machines virtuelles Azure avec Azure DevOps Starter

Azure DevOps Starter offre une expérience simplifiée dans laquelle vous pouvez apporter vos code et dépôt Git existants ou choisir un exemple d’application pour créer un pipeline d’intégration continue (CI) et de livraison continue (CD) dans Azure. 

Par ailleurs, DevOps Starter :
* Crée automatiquement des ressources Azure, telles qu’une nouvelle machine virtuelle Azure
* Crée et configure un pipeline de mise en production dans Azure DevOps qui inclut un pipeline de build pour CI
* Configurer un pipeline de mise en production pour CD 
* Crée une ressource Azure Application Insights pour la supervision

Ce didacticiel présente les procédures suivantes :

> [!div class="checklist"]
> * Utiliser DevOps Starter pour déployer votre application ASP.NET
> * Configurer Azure DevOps et un abonnement Azure 
> * Examiner le pipeline CI
> * Examiner le pipeline CD
> * Valider les modifications apportées à Azure Repos et les déployer automatiquement dans Azure
> * Configurer la surveillance d’Azure Application Insights
> * Nettoyer les ressources

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Vous pouvez en obtenir un gratuit via [Visual Studio Dev Essentials](https://visualstudio.microsoft.com/dev-essentials/).

## <a name="use-devops-starter-to-deploy-your-aspnet-app"></a>Utiliser DevOps Starter pour déployer votre application ASP.NET

Azure DevOps Starter crée un pipeline CI/CD dans Azure Pipelines. Vous pouvez créer une organisation Azure DevOps ou utiliser une organisation existante. Azure DevOps Project crée également des ressources Azure, telles que des machines virtuelles dans l’abonnement Azure de votre choix.

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Dans la zone de recherche, tapez **DevOps Starter**, puis sélectionnez $$$. Cliquez sur **Ajouter** pour en créer une.

    ![Tableau de bord DevOps Starter](_img/azure-devops-starter-aks/search-devops-starter.png)

1. Sélectionnez **.NET**, puis sélectionnez **Suivant**.

1. Sous **Choisir un framework d’application**, sélectionnez **ASP.NET**, puis **Suivant**. Le framework d’application, que vous avez choisi à l’étape précédente, détermine le type de cible de déploiement de service Azure disponible ici. 

1. Sélectionnez la machine virtuelle, puis sélectionnez **Suivant**.

## <a name="configure-azure-devops-and-an-azure-subscription"></a>Configurer Azure DevOps et un abonnement Azure

1. Créez une organisation Azure DevOps ou sélectionnez une organisation existante. 

1. Indiquez un nom pour votre projet Azure DevOps. 

1. Sélectionnez les services de votre abonnement Azure. Si vous le souhaitez, vous pouvez sélectionner **Modifier**, puis entrer plus de détails de configuration, comme l’emplacement des ressources Azure.
 
1. Entrez un nom de machine virtuelle, un nom d’utilisateur et un mot de passe pour votre nouvelle ressource de machine virtuelle Azure, puis sélectionnez **Terminé**. La machine virtuelle Azure est mise en service au bout de quelques minutes. Un exemple d’application ASP.NET est configuré dans un dépôt de votre organisation Azure DevOps, une build et une mise en production sont exécutées, et votre application est déployée sur la nouvelle machine virtuelle Azure créée. 

   Une fois ces opérations terminées, le tableau de bord DevOps Starter s’affiche dans le portail Azure. Vous pouvez également accéder au tableau de bord directement à partir de **Toutes les ressources** dans le portail Azure. 

   Ce tableau de bord permet de visualiser votre dépôt de code Azure DevOps, le pipeline CI/CD et votre application en cours d’exécution dans Azure.   

   ![Vue du tableau de bord](_img/azure-devops-project-vms/vm-starter-dashboard.png)

DevOps Starter configure automatiquement un déclencheur de build et de mise en production CI qui déploie les modifications de code dans votre dépôt. Vous pouvez configurer des options supplémentaires dans Azure DevOps. Pour voir votre application en cours d’exécution, sélectionnez **Parcourir**.
    
## <a name="examine-the-ci-pipeline"></a>Examiner le pipeline CI
 
DevOps Starter a configuré automatiquement un pipeline CI/CD dans Azure Pipelines. Vous pouvez explorer et personnaliser le pipeline. Pour vous familiariser avec le pipeline de build, effectuez les étapes suivantes :

1. En haut du tableau de bord DevOps Starter, sélectionnez **Pipelines de build**. Un onglet du navigateur affiche le pipeline de build de votre nouveau projet.

1. Pointez sur le champ **État**, puis sélectionnez les points de suspension (...). Un menu affiche plusieurs options, telles que de la mise en file d’attente d’une nouvelle build, la mise en pause d’une build et la modification du pipeline de build.

1. Sélectionnez **Modifier**.

1. Dans ce volet, vous pouvez examiner les différentes tâches de votre pipeline de build. La build effectue diverses opérations, comme la récupération (fetch) de sources dans le dépôt Git, la restauration de dépendances et la publication de sorties utilisées pour les déploiements.

1. En haut du pipeline de build, sélectionnez le nom du pipeline de build.

1. Remplacez le nom de votre pipeline de build par un nom plus descriptif, sélectionnez **Enregistrer et mettre en file d’attente**, puis sélectionnez **Enregistrer**.

1. Sous le nom de votre pipeline de build, sélectionnez **Historique**. Ce volet montre une piste d’audit des modifications que vous avez apportées récemment à la build. Azure DevOps suit les modifications apportées au pipeline de build et permet de comparer les versions.

1. Sélectionnez **Déclencheurs**. DevOps Starter crée automatiquement un déclencheur CI tandis que chaque validation dans le dépôt lance une nouvelle build. Si vous le souhaitez, vous pouvez choisir d’inclure ou d’exclure des branches dans le processus CI.

1. Sélectionnez **Rétention**. En fonction de votre scénario, vous pouvez spécifier des stratégies pour conserver ou supprimer un certain nombre de builds.

## <a name="examine-the-cd-pipeline"></a>Examiner le pipeline CD

DevOps Starter crée et configure automatiquement les étapes nécessaires au déploiement entre votre organisation Azure DevOps et votre abonnement Azure. Ces étapes comprennent la configuration d’une connexion au service Azure pour authentifier Azure DevOps auprès de votre abonnement Azure. L’automatisation crée également un pipeline CD qui fournit le déploiement continu à la machine virtuelle Azure. Pour en savoir plus sur le pipeline CD Azure DevOps, effectuez les étapes suivantes :

1. Sélectionnez **Build et mise en production**, puis **Mises en production**.  DevOps Starter crée un pipeline de mise en production pour gérer les déploiements sur Azure.

1. Sélectionnez les points de suspension (...) en regard de votre pipeline de mise en production, puis sélectionnez **Modifier**. Le pipeline de mise en production contient un *pipeline* qui définit le processus de mise en production.

1. Sous **Artefacts**, sélectionnez **Déposer**. Le pipeline de build que vous avez examiné aux étapes précédentes produit la sortie qui est utilisée pour l’artefact. 

1. En regard de l’icône **Déposer**, sélectionnez **Déclencheur de déploiement continu**. Ce pipeline de mise en production est doté d’un déclencheur de déploiement continu activé qui exécute un déploiement chaque fois qu’un nouvel artefact de build est disponible. Si vous le souhaitez, vous pouvez désactiver le déclencheur afin que vos déploiements nécessitent une exécution manuelle. 

1. À gauche, sélectionnez **Tâches**, puis sélectionnez votre environnement. Les tâches correspondent aux activités exécutées par votre processus de déploiement ; elles sont regroupées par phases. Ce pipeline de mise en production se déroule en deux phases :

    - La première phase contient une tâche de déploiement de groupe de ressources Azure qui effectue deux opérations :
     
        - Configurer la machine virtuelle pour le déploiement
        -   Ajouter la nouvelle machine virtuelle à un groupe de déploiement Azure DevOps Le groupe de déploiement de la machine virtuelle dans Azure DevOps gère des groupes logiques de machines cibles de déploiement
     
    - Dans la deuxième phase, une tâche de gestion d’application web IIS crée un site web IIS sur la machine virtuelle. Une seconde tâche de déploiement d’application web IIS est créée pour déployer le site.

1. À droite, sélectionnez **Afficher les mises en production** pour voir l’historique des mises en production.

1. Sélectionnez les points de suspension (...) en regard d’une mise en production, puis sélectionnez **Ouvrir**. Vous pouvez explorer plusieurs menus, comme un résumé des mises en production, les éléments de travail associés et les tests.

1. Sélectionnez **Validations**. Cette vue montre les validations de code qui sont associées à ce déploiement. Comparez les mises en production pour afficher les différences de validation entre les déploiements.

1. Sélectionnez **Journaux d’activité**. Les journaux d’activité contiennent des informations utiles sur le processus de déploiement. Vous pouvez les voir à la fois pendant et après les déploiements.

## <a name="commit-changes-to-azure-repos-and-automatically-deploy-them-to-azure"></a>Valider les modifications apportées à Azure Repos et les déployer automatiquement dans Azure 

Vous êtes maintenant prêt à collaborer avec une équipe sur votre application, par le biais d’un processus d’intégration et de déploiement continus (CI/CD) qui déploie automatiquement votre dernier travail sur votre site web. Chaque modification apportée au dépôt Git démarre une build dans Azure DevOps tandis qu’un pipeline CD exécute un déploiement dans Azure. Suivez la procédure décrite dans cette section, ou utilisez une autre technique pour valider les modifications dans votre dépôt. Les modifications de code lancent le processus d’intégration et de déploiement continus (CI/CD) et déploient automatiquement vos modifications pour le site web IIS sur la machine virtuelle Azure.

1. Dans le volet gauche, sélectionnez **Code**, puis accédez à votre dépôt.

1. Accédez au répertoire *Views\Home*, sélectionnez les points de suspension (...) situés en regard du fichier *Index.cshtml*, puis sélectionnez **Modifier**.

1. Modifiez le fichier, par exemple en ajoutant du texte dans une des balises div. 

1. En haut à droite, sélectionnez **Valider**, puis **Valider** à nouveau pour transmettre votre modification. Au bout de quelques instants, une build est démarrée dans Azure DevOps tandis qu’une mise en production est exécutée pour déployer les modifications. Surveillez l’état de la build dans le tableau de bord DevOps Starter, ou dans le navigateur avec votre organisation Azure DevOps.

1. Une fois la mise en production terminée, actualisez votre application pour vérifier vos modifications.

## <a name="configure-azure-application-insights-monitoring"></a>Configurer la surveillance d’Azure Application Insights

Azure Application Insights vous permet d’analyser facilement les performances et l’utilisation de votre application. DevOps Starter configure automatiquement une ressource Application Insights pour votre application. Vous pouvez configurer différentes capacités d’alertes et de surveillance, si besoin.

1. Dans le portail Azure, accédez au tableau de bord DevOps Starter. 

1. En bas à droite, sélectionnez le lien **Application Insights** pour votre application. Le volet **Application Insights** s’ouvre. Cette vue contient les informations sur la surveillance de l’utilisation, des performances et de la disponibilité de votre application.

   ![Le volet Application Insights](_img/azure-devops-project-github/appinsights.png) 

1. Sélectionnez **Intervalle de temps**, puis **Dernière heure**. Pour filtrer les résultats, sélectionnez **Mettre à jour**. Vous pouvez voir à présent toutes les activités des 60 dernières minutes. 
    
1. Pour quitter l’intervalle de temps, sélectionnez **x**.

1. Sélectionnez **Alertes**, puis **Ajouter une alerte Métrique**. 

1. Entrez un nom pour l’alerte.

1. Dans la liste déroulante **Métrique**, examinez les différents métriques d’alerte. L’alerte par défaut est pour un **temps de réponse serveur supérieur à 1 seconde**. Vous pouvez aisément configurer une variété d’alertes pour améliorer les capacités de surveillance de votre application.

1. Cochez la case **Envoyer un e-mail aux propriétaires, aux contributeurs et aux lecteurs**. Si vous le souhaitez, lorsqu’une alerte est affichée, vous pouvez effectuer des actions supplémentaires en exécutant une application logique Azure.

1. Sélectionnez **OK** pour créer l’alerte. Au bout de quelques instants, l’alerte apparaît comme étant active sur le tableau de bord. 

1. Quittez la zone **Alertes** et revenez au volet **Application Insights**.

1. Sélectionnez **Disponibilité** puis **Ajouter un test**. 

1. Entrez un nom de test, puis sélectionnez **Créer**. Un test Ping basique est créé pour vérifier la disponibilité de votre application. Au bout de quelques instants, les résultats de test sont disponibles, et le tableau de bord Application Insights affiche le statut de disponibilité.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous êtes en phase de test, vous pouvez éviter une hausse des frais de facturation en nettoyant vos ressources. Quand elles ne sont plus nécessaires, vous pouvez supprimer la machine virtuelle Azure et les ressources associées que vous avez créées dans ce tutoriel. Pour ce faire, utilisez la fonctionnalité **Supprimer** du tableau de bord DevOps Starter. 

> [!IMPORTANT]
> La procédure suivante supprime définitivement les ressources. La fonctionnalité *Supprimer* détruit les données qui sont créées par le projet dans DevOps Starter, à la fois dans Azure et dans Azure DevOps, et vous ne pourrez plus les récupérer. Utilisez cette procédure uniquement après avoir lu attentivement les invites.

1. Dans le portail Azure, accédez au tableau de bord DevOps Starter.
1. En haut à droite, sélectionnez **Supprimer**. 
1. À l’invite, sélectionnez **Oui** pour *supprimer définitivement* les ressources.

Si vous le souhaitez, vous pouvez modifier ces pipelines de build et de mise en production afin qu’ils répondent aux besoins de votre équipe. Vous pouvez également utiliser ce modèle CI/CD comme modèle pour vos autres pipelines. 

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Utiliser DevOps Starter pour déployer votre application ASP.NET
> * Configurer Azure DevOps et un abonnement Azure 
> * Examiner le pipeline CI
> * Examiner le pipeline CD
> * Valide les modifications apportées à Azure Repos et les déployer automatiquement dans Azure
> * Configurer la surveillance d’Azure Application Insights
> * Nettoyer les ressources

Pour en savoir plus sur le pipeline CI/CD, consultez :

> [!div class="nextstepaction"]
> [Définir votre pipeline de déploiement continu (CD) multiétape](/azure/devops/pipelines/release/define-multistage-release-process?view=vsts)