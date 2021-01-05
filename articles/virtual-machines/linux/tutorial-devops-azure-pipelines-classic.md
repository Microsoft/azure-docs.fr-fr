---
title: Tutoriel – Configurer des déploiements Rolling pour les machines virtuelles Linux Azure
description: Dans ce tutoriel, vous allez apprendre à configurer un pipeline de déploiement continu (CD). Ce pipeline met à jour de manière incrémentielle un groupe de machines virtuelles Linux Azure à l’aide de la stratégie de déploiement Rolling.
author: moala
manager: jpconnock
tags: azure-devops-pipelines
ms.assetid: ''
ms.service: virtual-machines-linux
ms.topic: tutorial
ms.tgt_pltfrm: azure-pipelines
ms.workload: infrastructure
ms.date: 4/10/2020
ms.author: moala
ms.custom: devops
ms.openlocfilehash: f77cc85bd62deb2cb2cb74c42cf245a409904b3a
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91978303"
---
# <a name="tutorial---configure-the-rolling-deployment-strategy-for-azure-linux-virtual-machines"></a>Tutoriel – Configurer une stratégie de déploiement propagé (Rolling) pour des machines virtuelles Linux Azure

Azure DevOps est un service Azure intégré qui automatise chaque partie du processus DevOps pour toutes les ressources Azure. Que votre application utilise des machines virtuelles, des applications web, Kubernetes ou toute autre ressource, vous pouvez implémenter une infrastructure sous forme de code (IaC), une intégration continue, des tests continus, une livraison continue et une supervision continue avec Azure et Azure DevOps.

![Le portail Azure avec Azure DevOps sélectionné sous Services](media/tutorial-devops-azure-pipelines-classic/azdevops-view.png)

## <a name="infrastructure-as-a-service-iaas---configure-cicd"></a>Infrastructure en tant que service (IaaS) - Configurer CI/CD

Azure Pipelines fournit un ensemble d’outils d’automatisation CI/CD pour les déploiements sur des machines virtuelles. Vous pouvez configurer un pipeline de livraison continue pour une machine virtuelle Azure à partir du portail Azure.

Cet article explique comment configurer un pipeline CI/CD pour le déploiement de déploiements multi-machine à partir du portail Azure. Le portail Azure prend également en charge d’autres stratégies, telles que [Canary](./tutorial-azure-devops-canary-strategy.md) et [Blue-Green](./tutorial-azure-devops-blue-green-strategy.md).

### <a name="configure-cicd-on-virtual-machines"></a>Configurer les outils CI/CD sur les machines virtuelles

Vous pouvez ajouter des machines virtuelles en tant que cibles à un [groupe de déploiement](/azure/devops/pipelines/release/deployment-groups). Vous pouvez ensuite les cibler pour les mises à jour multi-machine. Une fois que vous avez déployé sur des machines, affichez l’**Historique de déploiement** au sein d’un groupe de déploiement. Cette vue vous permet d’effectuer le suivi de la machine virtuelle au pipeline, puis de valider.

### <a name="rolling-deployments"></a>Déploiements propagés (Rolling)

Dans chaque itération, un déploiement propagé (Rolling) remplace les instances de la version précédente d’une application. Elle les remplace par des instances de la nouvelle version sur un ensemble fixe de machines (ensemble propagé). La procédure suivante explique comment configurer une mise à jour propagée sur des machines virtuelles.

Vous pouvez configurer des mises à jour propagées sur vos « machines virtuelles » dans le portail Azure à l’aide de l’option de livraison continue. Voici la procédure pas à pas :

1. Connectez-vous au portail Azure, puis accédez à une machine virtuelle.
1. Dans le volet le plus à gauche des paramètres de la machine virtuelle, sélectionnez **Livraison continue**. Ensuite, sélectionnez **Configurer**.

   ![Le volet Livraison continue avec le bouton Configurer](media/tutorial-devops-azure-pipelines-classic/azure-devops-configure.png)

1. Dans le volet de configuration, sélectionnez **Organisation Azure DevOps** pour choisir un compte existant ou en créer un nouveau. Sélectionnez ensuite le projet dans lequel vous souhaitez configurer le pipeline.  

   ![Le volet Livraison continue](media/tutorial-devops-azure-pipelines-classic/azure-devops-rolling.png)

1. Un groupe de déploiement est un ensemble logique de machines cibles de déploiement qui représentent les environnements physiques. Dev, Test, UAT et Production sont des exemples. Vous pouvez créer un groupe de déploiement, ou en sélectionner un qui existe déjà.
1. Sélectionnez le pipeline de build qui publie le package à déployer sur la machine virtuelle. Le script de déploiement deploy.ps1 ou deploy.sh doit se trouver dans le dossier deployscripts qui est situé dans le dossier racine du package publié. Le pipeline exécute ce script de déploiement.
1. Dans **Stratégie de déploiement**, sélectionnez **Continue**.
1. Si vous le souhaitez, vous pouvez étiqueter chaque machine avec son rôle. Les balises « web » et « db » sont des exemples. Ces étiquettes vous permettent de cibler uniquement les machines virtuelles qui ont un rôle spécifique.
1. Sélectionnez **OK** pour configurer le pipeline de livraison continue.
1. Une fois la configuration terminée, vous avez un pipeline de livraison continue configuré pour un déploiement sur la machine virtuelle.  

   ![Volet Livraison continue avec l’historique de déploiement](media/tutorial-devops-azure-pipelines-classic/azure-devops-deployment-history.png)

1. Les détails du déploiement de la machine virtuelle s’affichent. Vous pouvez sélectionner le lien pour accéder au pipeline, **Release-1** pour afficher le déploiement, ou **Modifier** pour modifier la définition du pipeline de mise en production.

1. Si vous configurez plusieurs machines virtuelles, répétez les étapes 2 à 4 pour les autres machines virtuelles à ajouter au groupe de déploiement. Si vous sélectionnez un groupe de déploiement qui a déjà une exécution de pipeline, les machines virtuelles sont simplement ajoutées au groupe de déploiement. Aucun nouveau pipeline n’est créé.
1. Une fois la configuration terminée, sélectionnez la définition du pipeline, accédez à l’organisation Azure DevOps, puis sélectionnez **Modifier** pour le pipeline de mise en production.

   ![Modification du pipeline propagé](media/tutorial-devops-azure-pipelines-classic/azure-devops-rolling-pipeline.png)

1. Sélectionnez **1 travail, 1 tâche** dans la phase **dev**. Sélectionnez la phase **Déployer**.

   ![Tâches de pipeline propagé avec tâche de déploiement sélectionnée](media/tutorial-devops-azure-pipelines-classic/azure-devops-rolling-pipeline-tasks.png)

1. Dans le volet de configuration de droite, vous pouvez spécifier le nombre de machines que vous souhaitez déployer en parallèle dans chaque itération. Si vous souhaitez effectuer un déploiement sur plusieurs machines à la fois, vous pouvez spécifier le nombre de machines à l’aide d’un pourcentage avec le curseur.  

1. La tâche Exécuter le script de déploiement exécute par défaut le script de déploiement deploy.ps1 ou deploy.sh. Le script se trouve dans le dossier deployscripts du dossier racine du package publié.

   ![Volet Artefacts montrant deploy.sh dans le dossier deployscripts](media/tutorial-deployment-strategy/package.png)

## <a name="other-deployment-strategies"></a>Autres stratégies de déploiement

- [Configurer la stratégie de déploiement Canary](./tutorial-azure-devops-canary-strategy.md)
- [Configurer la stratégie de déploiement Blue-Green](./tutorial-azure-devops-blue-green-strategy.md)

## <a name="azure-devops-projects"></a>Azure DevOps Projects

Vous pouvez commencer à utiliser Azure facilement. Avec Azure DevOps Projects, commencez à exécuter votre application sur n’importe quel service Azure en seulement trois étapes en sélectionnant :

- Une langue d'application
- Un runtime
- Un service Azure
 
[Plus d’informations](https://azure.microsoft.com/features/devops-projects/)
 
## <a name="additional-resources"></a>Ressources supplémentaires

- [Déployer sur des machines virtuelles Azure avec Azure DevOps Projects](../../devops-project/azure-devops-project-vms.md)
- [Implémenter un déploiement continu de votre application sur un groupe de machines virtuelles identiques Azure](/azure/devops/pipelines/apps/cd/azure/deploy-azure-scaleset)