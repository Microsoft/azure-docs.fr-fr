---
title: 'Tutoriel : Déployer votre application Node.js dans une application web Azure à l’aide de DevOps Starter pour GitHub Actions'
description: DevOps Starter facilite la prise en main d’Azure et le déploiement de votre application Node.js dans une application web Azure en quelques étapes rapides.
ms.author: ninallam
ms.prod: devops
ms.technology: devops-cicd
ms.topic: tutorial
ms.date: 08/25/2020
author: ninallam
ms.openlocfilehash: 4caac943df33177728997c74c9ec812105efd62c
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97588775"
---
# <a name="tutorial-deploy-nodejs-app-to-azure-web-app-using-devops-starter-for-github-actions"></a>Tutoriel : Déployer une application Node.js dans une application web Azure à l’aide de DevOps Starter pour GitHub Actions

DevOps Starter pour GitHub Actions offre une expérience simplifiée dans laquelle vous pouvez choisir un exemple d’application pour créer un workflow d’intégration continue (CI) et de livraison continue (CD) à déployer dans Azure. 

En outre, DevOps Starter :
* Crée automatiquement des ressources Azure, telles qu’une nouvelle application web Azure.
* Crée et configure un workflow dans GitHub qui comprend un travail de build pour CI.
* Le workflow contient également un travail de déploiement pour CD. 
* Crée une ressource Azure Application Insights pour la supervision

Ce didacticiel présente les procédures suivantes :

> [!div class="checklist"]
> * Utiliser DevOps Starter pour déployer une application Node.js
> * Configurer GitHub et un abonnement Azure 
> * Examiner le workflow GitHub
> * Valider les modifications apportées à GitHub et les déployer automatiquement dans Azure
> * Configurer la surveillance d’Azure Application Insights
> * Nettoyer les ressources

## <a name="prerequisites"></a>Prérequis

* Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/).

## <a name="use-devops-starter-to-deploy-a-nodejs-app"></a>Utiliser DevOps Starter pour déployer une application Node.js

DevOps Starter crée un workflow dans GitHub. Vous pouvez utiliser une organisation GitHub existante. DevOps Starter crée également des ressources Azure, telles qu’une application web dans l’abonnement Azure de votre choix.

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Dans la zone de recherche, tapez, puis sélectionnez **DevOps Starter**. Cliquez sur le **signe +** pour créer une ressource.

    ![Tableau de bord DevOps Starter](_img/azure-devops-starter-aks/search-devops-starter.png)

1. Assurez-vous que le fournisseur CI/CD est sélectionné en tant que **GitHub Actions**.

    ![CICD_provider](_img/azure-devops-project-nodejs/provider-selection.png)

1. Sélectionnez **Node.js**, puis sélectionnez **Suivant**.

1. Sous **Choisir un framework d’application**, sélectionnez **Express.js**, puis **Suivant**. Le framework d’application, que vous avez choisi à l’étape précédente, détermine le type de cible de déploiement de service Azure disponible ici. 

1. Sélectionnez l’**application web Windows**, puis sélectionnez **Suivant**.

## <a name="configure-github-and-an-azure-subscription"></a>Configurer GitHub et un abonnement Azure

1. **Autorisez** GitHub et sélectionnez une organisation GitHub existante. 

1. Entrez un nom pour votre **Dépôt GitHub**. 

1. Sélectionnez les services de votre abonnement Azure. Si vous le souhaitez, vous pouvez sélectionner **Modifier**, puis entrer plus de détails de configuration, comme l’emplacement des ressources Azure.
 
1. Entrez un nom d’application web, puis sélectionnez **Terminé**. L’application web Azure est prête au bout de quelques minutes. Un exemple d’application Node.js est configuré dans un dépôt de votre organisation GitHub, un workflow est exécuté, et votre application est déployée dans la nouvelle application web Azure créée.

   ![Enter_GH_details](_img/azure-devops-project-nodejs/gh-details.png)    

   Une fois ces opérations terminées, le tableau de bord DevOps Starter s’affiche dans le portail Azure. Vous pouvez également accéder au tableau de bord directement à partir de **Toutes les ressources** dans le portail Azure. 

   Ce tableau de bord permet de visualiser votre dépôt de code GitHub, le workflow CI/CD et votre application en cours d’exécution dans Azure.   

   ![Vue du tableau de bord](_img/azure-devops-project-nodejs/full-dashboard.png)

DevOps Starter configure automatiquement un déclencheur qui déploie les modifications de code dans votre dépôt.
    
## <a name="examine-the-github-workflow"></a>Examiner le workflow GitHub

Dans l’étape précédente, DevOps Starter a automatiquement configuré un workflow GitHub complet. Explorez et personnalisez le workflow selon vos besoins. Effectuez les étapes suivantes pour vous familiariser avec le workflow.

1. À gauche du tableau de bord DevOps Starter, sélectionnez **GitHub workflow**. Ce lien ouvre un onglet de navigateur et le workflow GitHub de votre nouveau projet.
    > [!NOTE]
    > Ne renommez pas le fichier de workflow. Le nom du fichier de workflow doit être **devops-Starter-workflow.yml** pour que le tableau de bord reflète les modifications.

1. Le fichier yaml du workflow contient toutes les actions GitHub nécessaires pour générer et déployer l’application. Cliquez sur l’option **modifier le fichier** pour personnaliser votre fichier de workflow.

1. Sous l’onglet **Code** du dépôt, cliquez sur **commits**. Cette vue montre les validations de code associées au déploiement spécifique.

1. Sous l’onglet **Actions** du dépôt, vous pouvez afficher l’historique de toutes les exécutions de workflow de votre dépôt.

1. Sélectionnez la **dernière exécution** pour afficher tous les travaux qui ont été exécutés dans le workflow.

1. Cliquez sur **jobs** pour voir les journaux détaillés de l’exécution du workflow. Les journaux d’activité contiennent des informations utiles sur le processus de déploiement. Ils peuvent être affichés pendant et après les déploiements.

1. Cliquez sur l’onglet **Pull request** pour voir toutes les demandes de tirage sur votre dépôt.

## <a name="commit-code-changes-and-execute-cicd"></a>Valider les modifications de code et exécuter CI/CD

DevOps Starter crée un dépôt dans GitHub. Pour voir le dépôt et modifier le code dans votre application, suivez les étapes ci-dessous :

1. À gauche du tableau de bord DevOps Starter, sélectionnez le lien correspondant à votre branche primaire. Ce lien ouvre une vue sur le dépôt GitHub récemment créé.

1. Pour afficher l’URL du clone du dépôt, sélectionnez **Cloner** en haut à droite du navigateur. Vous pouvez cloner votre référentiel Git dans votre environnement de développement intégré favori. Dans les prochaines étapes, vous allez utiliser le navigateur web pour effectuer des modifications de code directement dans la branche primaire et les valider.

1. Sur la gauche du navigateur, accédez au fichier **/Application/views/index.pug**.

1. Sélectionnez **Modifier**, puis modifiez le texte.
    Par exemple, modifiez l’une des balises.

1. Sélectionnez **Valider**, puis enregistrez vos modifications.

1. Dans le navigateur, accédez au tableau de bord DevOps Starter.   
Vous devez maintenant voir un travail de build de workflow GitHub en cours. Les modifications que vous venez d’effectuer font automatiquement l’objet d’une build et d’un déploiement via un workflow GitHub.

1. Une fois le déploiement terminé, actualisez votre application pour vérifier vos modifications.

## <a name="configure-azure-application-insights-monitoring"></a>Configurer la surveillance d’Azure Application Insights

Azure Application Insights vous permet d’analyser facilement les performances et l’utilisation de votre application. DevOps Starter configure automatiquement une ressource Application Insights pour votre application. Vous pouvez configurer différentes capacités d’alertes et de surveillance, si besoin.

1. Dans le Portail Azure, accédez au tableau de bord DevOps Starter. 

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
> La procédure suivante supprime définitivement les ressources. La fonctionnalité *Supprimer* détruit les données qui sont créées par le projet dans DevOps Starter et Azure, et vous ne pourrez plus les récupérer. Utilisez cette procédure uniquement après avoir lu attentivement les invites.

1. Dans le Portail Azure, accédez au tableau de bord DevOps Starter.
1. En haut à droite, sélectionnez **Supprimer**. 
1. À l’invite, sélectionnez **Oui** pour *supprimer définitivement* les ressources.

Si vous le souhaitez, vous pouvez modifier le workflow afin qu’il réponde aux besoins de votre équipe. Vous pouvez également utiliser ce modèle CI/CD comme modèle pour vos autres dépôts. 

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Utiliser DevOps Starter pour déployer une application Node.js
> * Configurer GitHub et un abonnement Azure 
> * Examiner le workflow GitHub
> * Valider les modifications apportées à GitHub et les déployer automatiquement dans Azure
> * Configurer la surveillance d’Azure Application Insights
> * Nettoyer les ressources

Pour en savoir plus sur GitHub Actions et les workflows, consultez :

> [!div class="nextstepaction"]
> [Personnaliser le workflow GitHub](https://docs.github.com/actions/configuring-and-managing-workflows/configuring-and-managing-workflow-files-and-runs)
