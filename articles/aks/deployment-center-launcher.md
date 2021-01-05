---
title: Centre de déploiement pour Azure Kubernetes
description: Le centre de déploiement dans Azure DevOps simplifie la configuration d’un pipeline Azure DevOps robuste pour votre application.
ms.author: puagarw
ms.topic: tutorial
ms.date: 07/12/2019
author: pulkitaggarwl
ms.openlocfilehash: 15b0413eabcfae7e3a4b28243caf2a708260ccae
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96932215"
---
# <a name="deployment-center-for-azure-kubernetes"></a>Centre de déploiement pour Azure Kubernetes

Le centre de déploiement dans Azure DevOps simplifie la configuration d’un pipeline Azure DevOps robuste pour votre application. Par défaut, le centre de déploiement configure un pipeline Azure DevOps pour déployer vos mises à jour d’application sur le cluster Kubernetes. Vous pouvez étendre le pipeline Azure DevOps configuré par défaut et ajouter des fonctionnalités enrichies, telles que la possibilité d’obtenir une approbation avant le déploiement, d’approvisionner des ressources Azure supplémentaires, d’exécuter des scripts, de mettre à niveau votre application, voire d’exécuter des tests de validation supplémentaires.

Ce didacticiel présente les procédures suivantes :

> [!div class="checklist"]
> * configurer un pipeline DevOps pour déployer vos mises à jour d’application sur le cluster Kubernetes ;
> * examiner le pipeline d’intégration continue (CI) ;
> * examiner le pipeline de déploiement continu (CD) ;
> * nettoyer les ressources.

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Vous pouvez en obtenir un gratuit via [Visual Studio Dev Essentials](https://visualstudio.microsoft.com/dev-essentials/).

* Un cluster Azure Kubernetes Service (AKS).

## <a name="create-an-aks-cluster"></a>Créer un cluster AKS

1. Connectez-vous à votre [portail Azure](https://portal.azure.com/).

1. Sélectionnez le bouton [Cloud Shell](../cloud-shell/overview.md) du côté droit de la barre de menus sur le portail Azure.

1. Pour créer le cluster AKS, exécutez les commandes suivantes :

    ```azurecli
    # Create a resource group in the South India location:

    az group create --name azooaks --location southindia

    # Create a cluster named azookubectl with one node.

    az aks create --resource-group azooaks --name azookubectl --node-count 1 --enable-addons monitoring --generate-ssh-keys
    ```

## <a name="deploy-application-updates-to-a-kubernetes-cluster"></a>Déployer des mises à jour d’application sur un cluster Kubernetes

1. Accédez au groupe de ressources que vous avez créé dans la section précédente.

1. Sélectionnez le cluster AKS, puis **Centre de déploiement (préversion)** dans le panneau de gauche. Sélectionnez **Prise en main**.

   ![Capture d’écran montrant le portail Azure avec une flèche pointant vers le centre de déploiement.](media/deployment-center-launcher/settings.png)

1. Choisissez l’emplacement du code, puis sélectionnez **Suivant**. Sélectionnez ensuite l’un des référentiels actuellement pris en charge : **[Azure Repos](/azure/devops/repos/index)** ou **GitHub**.

    Azure Repos est un ensemble d’outils de gestion de version qui vous aident à gérer votre code. Quelle que soit la taille de votre projet logiciel, nous vous conseillons d’utiliser la gestion de version au plus tôt.

    - **Azure Repos** : choisissez un dépôt parmi les dépôts existants de votre projet et de votre organisation.

        ![Azure Repos](media/deployment-center-launcher/azure-repos.gif)

    - **GitHub** : autorisez et sélectionnez le dépôt pour votre compte GitHub.

        ![Animation illustrant le processus de sélection de GitHub en tant que source, puis la sélection de votre référentiel.](media/deployment-center-launcher/github.gif)


1. Le centre de déploiement analyse le référentiel et détecte votre Dockerfile. Si vous souhaitez mettre à jour celui-ci, vous pouvez modifier le numéro de port identifié.

    ![Paramètres de l’application](media/deployment-center-launcher/application-settings.png)

    Si le dépôt ne contient pas le Dockerfile, le système affiche un message vous invitant à en valider un.

    ![Capture d’écran montrant le centre de déploiement avec le message Impossible de trouver Dockerfile dans le référentiel.](media/deployment-center-launcher/dockerfile.png)

1. Sélectionnez un registre de conteneurs existant ou créez-en un, puis sélectionnez **Terminer**. Le pipeline automatiquement met une build en file d’attente dans [Azure Pipelines](/azure/devops/pipelines/index).

    Azure Pipelines est un service cloud que vous pouvez utiliser pour générer et tester automatiquement votre projet de code et le mettre à la disposition d’autres utilisateurs. Azure Pipelines combine l’intégration continue (CI) et le déploiement continu (CD) afin de tester et générer votre code de manière constante et cohérente, et de l’expédier vers toute cible.

    ![Container Registry](media/deployment-center-launcher/container-registry.png)

1. Sélectionnez le lien pour afficher le pipeline en cours.

1. Une fois le déploiement terminé, les journaux de réussite s’affichent.

    ![Capture d’écran montrant le centre de déploiement avec la version 1 marquée d’une icône de coche verte.](media/deployment-center-launcher/logs.png)

## <a name="examine-the-ci-pipeline"></a>Examiner le pipeline CI

Le centre de déploiement configure automatiquement le pipeline CI/CD de votre organisation Azure DevOps. Le pipeline peut être exploré et personnalisé.

1. Accédez au tableau de bord du centre de déploiement.  

1. Sélectionnez le numéro de build dans la liste des journaux pour voir le pipeline de build de votre projet.

1. Sélectionnez les points de suspension (...) dans l’angle supérieur droit. Un menu affiche plusieurs options, telles que la mise en file d’attente de nouvelle build, la conservation de build et la modification du pipeline de build. Sélectionnez **Modifier le pipeline**. 

1. Vous pouvez examiner les différentes tâches de votre pipeline de build dans ce volet. La build effectue différentes tâches, telles que la collecte de sources à partir du dépôt Git, la création d’une image, l’envoi (push) d’une image vers le registre de conteneurs et la publication des sorties utilisées pour les déploiements.

1. Sélectionnez le nom du pipeline de build en haut du pipeline.

1. Remplacez le nom de votre pipeline de build par un nom plus descriptif, sélectionnez **Enregistrer et mettre en file d’attente**, puis sélectionnez **Enregistrer**.

1. Sous votre pipeline de build, sélectionnez **Historique**. Ce volet affiche une piste d’audit de vos modifications de build récentes. Azure DevOps supervise les modifications apportées au pipeline de build et vous permet de comparer les versions.

1. Sélectionnez **Déclencheurs**. Vous pouvez inclure ou exclure des branches dans le processus d’intégration continue.

1. Sélectionnez **Rétention**. Vous pouvez spécifier des stratégies pour conserver ou supprimer un certain nombre de builds en fonction de votre scénario.

## <a name="examine-the-cd-pipeline"></a>Examiner le pipeline CD

Le centre de déploiement crée et configure automatiquement la relation entre votre organisation Azure DevOps et votre abonnement Azure. Les étapes impliquées sont la configuration d’une connexion au service Azure pour authentifier votre abonnement Azure auprès d’Azure DevOps. Le processus automatisé crée également un pipeline de mise en production qui effectue une livraison continue à Azure.

1. Sélectionnez **Pipelines**, puis **Mises en production**.

1. Pour modifier le pipeline de mise en production, sélectionnez **Modifier**.

1. Sélectionnez **Déposer** dans la liste **Artefacts**. Lors des étapes précédentes, le pipeline de build que vous avez examiné génère la sortie utilisée pour l’artefact. 

1. Sélectionnez le déclencheur **Déploiement continu** à droite de l’option **Déposer**. Ce pipeline de mise en production est doté d’un déclencheur de déploiement continu activé qui exécute un déploiement chaque fois qu’un nouvel artefact de build est disponible. Vous pouvez également désactiver le déclencheur afin d’exiger une exécution manuelle pour vos déploiements.

1. Pour examiner toutes les tâches de votre pipeline, sélectionnez **Tâches**. La mise en production définit l’environnement Tiller, configure le paramètre `imagePullSecrets`, installe les outils Helm et déploie les graphiques Helm sur le cluster Kubernetes.

1. Pour afficher l’historique des mises en production, sélectionnez **Afficher les mises en production**.

1. Pour afficher le résumé, sélectionnez **Mise en production**. Sélectionnez l’une des étapes pour explorer plusieurs menus, comme une synthèse des mises en production, les éléments de travail associés et les tests. 

1. Sélectionnez **Validations**. Cette vue montre les validations de code liées à ce déploiement. Comparez les mises en production pour voir les différences de validation entre les déploiements.

1. Sélectionnez **Journaux d’activité**. Les journaux contiennent des informations de déploiement utiles, que vous pouvez afficher pendant et après les déploiements.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’en avez plus besoin, vous pouvez supprimer les ressources associées que vous avez créées. Utilisez la fonctionnalité Supprimer du tableau de bord DevOps Projects.

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez le modifier afin qu’il réponde aux besoins de votre équipe. Vous pouvez aussi utiliser ce modèle CI/CD pour vos autres pipelines.
