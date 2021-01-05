---
title: Déployer des applications ASP.NET Core dans Azure Kubernetes Service avec Azure DevOps Starter
description: Azure DevOps Starter permet de démarrer facilement avec Azure. Avec Azure DevOps Starter, vous pouvez déployer votre application ASP.NET Core à l’aide d’Azure Kubernetes Service (AKS) en quelques étapes rapides.
ms.author: mlearned
ms.manager: gwallace
ms.prod: devops
ms.technology: devops-cicd
ms.topic: tutorial
ms.date: 03/24/2020
author: mlearned
ms.openlocfilehash: af61254643e9f072995bceed1b6ecf8ecc29e5f8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91855450"
---
# <a name="deploy-aspnet-core-apps-to-azure-kubernetes-service-with-azure-devops-starter"></a>Déployer des applications ASP.NET Core dans Azure Kubernetes Service avec Azure DevOps Starter

Azure DevOps Starter offre une expérience simplifiée dans laquelle vous pouvez apporter vos code et dépôt Git existants ou choisir un exemple d’application pour créer un pipeline d’intégration continue (CI) et de livraison continue (CD) dans Azure. 

De plus, DevOps Starter :

* Crée automatiquement des ressources Azure, telles qu’Azure Kubernetes Service (AKS)
* Crée et configure un pipeline de mise en production dans Azure DevOps qui définit un pipeline de build et de mise en production pour CI/CD.
* Crée une ressource Azure Application Insights pour la supervision
* Permet à [Azure Monitor pour conteneurs](../azure-monitor/insights/container-insights-overview.md) de superviser les performances des charges de travail de conteneur sur le cluster AKS

Ce didacticiel présente les procédures suivantes :

> [!div class="checklist"]
> * Utiliser DevOps Starter pour déployer une application ASP.NET Core sur AKS
> * Configurer Azure DevOps et un abonnement Azure 
> * Examiner le cluster AKS
> * Examiner le pipeline CI
> * Examiner le pipeline CD
> * Valider les modifications dans Git et les déployer automatiquement dans Azure
> * Nettoyer les ressources

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Vous pouvez en obtenir un gratuit via [Visual Studio Dev Essentials](https://visualstudio.microsoft.com/dev-essentials/).

## <a name="use-devops-starter-to-deploy-an-aspnet-core-app-to-aks"></a>Utiliser DevOps Starter pour déployer une application ASP.NET Core sur AKS

DevOps Starter crée un pipeline CI/CD dans Azure Pipelines. Vous pouvez créer une organisation Azure DevOps ou utiliser une organisation existante. DevOps Starter crée également des ressources Azure, telles qu’un cluster AKS, dans l’abonnement Azure de votre choix.

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Dans la zone de recherche, tapez, puis sélectionnez **DevOps Starter**. Cliquez sur le **signe +** pour créer une ressource.

    ![Tableau de bord DevOps Starter](_img/azure-devops-starter-aks/search-devops-starter.png)

1. Sélectionnez **.NET**, puis sélectionnez **Suivant**.

1. Sous **Choisir un framework d’application**, sélectionnez **ASP.NET Core**, puis **Suivant**.

1. Sélectionnez **Kubernetes Service**, puis **Suivant**. 

## <a name="configure-azure-devops-and-an-azure-subscription"></a>Configurer Azure DevOps et un abonnement Azure

1. Créez une organisation Azure DevOps ou sélectionnez une organisation existante. 

1. Indiquez un nom pour votre projet Azure DevOps. 

1. Sélectionnez votre abonnement Azure.

1. Pour afficher des paramètres de configuration Azure supplémentaires et déterminer le nombre de nœuds du cluster AKS, sélectionnez **Modifier**. Ce volet regroupe différentes options pour la configuration du type et de l’emplacement des services Azure.
 
1. Quittez la zone de configuration Azure, puis sélectionnez **Terminé**. Le processus est achevé au bout de quelques minutes. Un exemple d’application ASP.NET Core est configuré dans un dépôt Git de votre organisation Azure DevOps, un cluster AKS est créé, un pipeline CI/CD est exécuté et votre application est déployée sur Azure. 

    Une fois toutes ces opérations terminées, le tableau de bord Azure DevOps Starter s’affiche dans le portail Azure. Vous pouvez également accéder au tableau de bord DevOps Starter directement à partir de **Toutes les ressources** dans le portail Azure. 

    Ce tableau de bord permet de visualiser votre dépôt de code Azure DevOps, le pipeline CI/CD et votre cluster AKS. Vous pouvez configurer d’autres options de CI/CD dans votre pipeline Azure DevOps. À droite, sélectionnez **Parcourir** pour voir votre application en cours d’exécution.

## <a name="examine-the-aks-cluster"></a>Examiner le cluster AKS

DevOps Starter configure automatiquement un cluster AKS que vous pouvez explorer et personnaliser. Pour vous familiariser avec le cluster AKS, effectuez les étapes suivantes :

1. Accédez au tableau de bord DevOps Starter.

1. À droite, sélectionnez le service AKS. Un volet s’ouvre pour le cluster AKS. À partir de cette vue, vous pouvez effectuer diverses actions, comme superviser l’intégrité des conteneurs, effectuer des recherches dans les journaux d’activité et ouvrir le tableau de bord Kubernetes.

1. À droite, sélectionnez **Voir le tableau de bord Kubernetes**. Si vous le souhaitez, suivez ces étapes pour ouvrir le tableau de bord Kubernetes.

## <a name="examine-the-ci-pipeline"></a>Examiner le pipeline CI

DevOps Starter configure automatiquement un pipeline CI/CD dans votre organisation Azure DevOps. Vous pouvez explorer et personnaliser le pipeline. Pour vous familiariser avec celui-ci, procédez comme suit :

1. Accédez au tableau de bord DevOps Starter.

1. En haut du tableau de bord DevOps Starter, sélectionnez **Pipelines de build**.  Un onglet du navigateur affiche le pipeline de build de votre nouveau projet.

1. Pointez sur le champ **État**, puis sélectionnez les points de suspension (...).  Un menu affiche plusieurs options, telles que de la mise en file d’attente d’une nouvelle build, la mise en pause d’une build et la modification du pipeline de build.

1. Sélectionnez **Modifier**.

1. Dans ce volet, vous pouvez examiner les différentes tâches de votre pipeline de build. La build effectue diverses opérations, comme la récupération (fetch) de sources dans le dépôt Git, la restauration de dépendances et la publication de sorties utilisées pour les déploiements.

1. En haut du pipeline de build, sélectionnez le nom du pipeline de build.

1. Remplacez le nom de votre pipeline de build par un nom plus descriptif, sélectionnez **Enregistrer et mettre en file d’attente**, puis sélectionnez **Enregistrer**.

1. Sous le nom de votre pipeline de build, sélectionnez **Historique**. Ce volet montre une piste d’audit des modifications que vous avez apportées récemment à la build. Azure DevOps suit les modifications apportées au pipeline de build et permet de comparer les versions.

1. Sélectionnez **Déclencheurs**. DevOps Starter crée automatiquement un déclencheur CI tandis que chaque validation dans le dépôt lance une nouvelle build. Si vous le souhaitez, vous pouvez choisir d’inclure ou d’exclure des branches dans le processus CI.

1. Sélectionnez **Rétention**. En fonction de votre scénario, vous pouvez spécifier des stratégies pour conserver ou supprimer un certain nombre de builds.

## <a name="examine-the-cd-release-pipeline"></a>Examiner le pipeline de mise en production CD

DevOps Starter crée et configure automatiquement les étapes nécessaires au déploiement entre votre organisation Azure DevOps et votre abonnement Azure. Ces étapes comprennent la configuration d’une connexion au service Azure pour authentifier Azure DevOps auprès de votre abonnement Azure. L’automatisation crée également un pipeline de mise en production, qui fournit le déploiement continu vers Azure. Pour en savoir plus sur le pipeline de mise en production, suivez ces étapes :

1. Sélectionnez **Build et mise en production**, puis **Mises en production**.  DevOps Starter crée un pipeline de mise en production pour gérer les déploiements sur Azure.

1. Sélectionnez les points de suspension (...) en regard de votre pipeline de mise en production, puis sélectionnez **Modifier**. Le pipeline de mise en production contient un *pipeline* qui définit le processus de mise en production.

1. Sous **Artefacts**, sélectionnez **Déposer**. Le pipeline de build que vous avez examiné aux étapes précédentes produit la sortie qui est utilisée pour l’artefact. 

1. À droite de l’icône **Déposer**, sélectionnez **Déclencheur de déploiement continu**. Ce pipeline de mise en production est doté d’un déclencheur de déploiement continu activé qui exécute un déploiement chaque fois qu’un nouvel artefact de build est disponible. Si vous le souhaitez, vous pouvez désactiver le déclencheur afin que vos déploiements nécessitent une exécution manuelle. 

1. À droite, sélectionnez **Afficher les mises en production** pour voir l’historique des mises en production.

1. Sélectionnez les points de suspension (...) en regard d’une mise en production, puis sélectionnez **Ouvrir**. Vous pouvez explorer plusieurs menus, comme un résumé des mises en production, les éléments de travail associés et les tests.

1. Sélectionnez **Validations**. Cette vue montre les validations de code qui sont associées à ce déploiement. Comparez les mises en production pour afficher les différences de validation entre les déploiements.

1. Sélectionnez **Journaux d’activité**. Les journaux d’activité contiennent des informations utiles sur le processus de déploiement. Vous pouvez les voir à la fois pendant et après les déploiements.

## <a name="commit-changes-to-azure-repos-and-automatically-deploy-them-to-azure"></a>Valider les modifications apportées à Azure Repos et les déployer automatiquement dans Azure 

 > [!NOTE]
 > La procédure suivante teste le pipeline CI/CD en apportant une modification de texte simple.

Vous êtes maintenant prêt à collaborer avec une équipe sur votre application, par le biais d’un processus d’intégration et de déploiement continus (CI/CD) qui déploie automatiquement votre dernier travail sur votre site web. Chaque modification apportée au dépôt Git démarre une build dans Azure DevOps tandis qu’un pipeline CD exécute un déploiement dans Azure. Suivez la procédure décrite dans cette section, ou utilisez une autre technique pour valider les modifications dans votre dépôt. Par exemple, vous pouvez cloner le dépôt Git dans votre outil ou environnement de développement intégré favori, puis envoyer (push) les modifications vers ce dépôt.

1. Dans le menu Azure DevOps, sélectionnez **Code** > **Fichiers**, puis accédez à votre dépôt.

1. Accédez au répertoire *Views\Home*, sélectionnez les points de suspension (...) situés en regard du fichier *Index.cshtml*, puis sélectionnez **Modifier**.

1. Modifiez le fichier, par exemple en ajoutant du texte dans une des balises div. 

1. En haut à droite, sélectionnez **Valider**, puis **Valider** à nouveau pour transmettre votre modification. Au bout de quelques instants, une build est démarrée dans Azure DevOps tandis qu’une mise en production est exécutée pour déployer les modifications. Supervisez l’état de la build sur le tableau de bord DevOps Starter, ou dans le navigateur avec votre organisation Azure DevOps.

1. Une fois la mise en production terminée, actualisez votre application pour vérifier vos modifications.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous êtes en phase de test, vous pouvez éviter une hausse des frais de facturation en nettoyant vos ressources. Quand ils ne sont plus nécessaires, vous pouvez supprimer le cluster AKS et les ressources associées que vous avez créés dans ce tutoriel. Pour ce faire, utilisez la fonctionnalité **Supprimer** du tableau de bord DevOps Starter.

> [!IMPORTANT]
> La procédure suivante supprime définitivement les ressources. La fonctionnalité *Supprimer* détruit les données qui sont créées par le projet dans DevOps Starter, à la fois dans Azure et dans Azure DevOps, et vous ne pourrez plus les récupérer. Utilisez cette procédure uniquement après avoir lu attentivement les invites.

1. Dans le Portail Azure, accédez au tableau de bord DevOps Starter.
1. En haut à droite, sélectionnez **Supprimer**. 
1. À l’invite, sélectionnez **Oui** pour *supprimer définitivement* les ressources.

## <a name="next-steps"></a>Étapes suivantes

Si vous le souhaitez, vous pouvez modifier ces pipelines de build et de mise en production afin qu’ils répondent aux besoins de votre équipe. Vous pouvez également utiliser ce modèle CI/CD comme modèle pour vos autres pipelines. Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Utiliser DevOps Starter pour déployer une application ASP.NET Core sur AKS
> * Configurer Azure DevOps et un abonnement Azure 
> * Examiner le cluster AKS
> * Examiner le pipeline CI
> * Examiner le pipeline CD
> * Valider les modifications dans Git et les déployer automatiquement dans Azure
> * Nettoyer les ressources

Pour en savoir plus sur l’utilisation du tableau de bord Kubernetes, consultez :

> [!div class="nextstepaction"]
> [Utiliser le tableau de bord Kubernetes](../aks/kubernetes-dashboard.md)