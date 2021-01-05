---
title: Tutoriel – Configurer des déploiements canary pour les machines virtuelles Linux Azure
description: Dans ce tutoriel, vous allez apprendre à configurer un pipeline de déploiement continu (CD). Ce pipeline met à jour un groupe de machines virtuelles Linux Azure à l’aide de la stratégie de déploiement .
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
ms.openlocfilehash: fafe2ba9ae65f95d8a2258ead5715cc0630202ba
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91963480"
---
# <a name="tutorial---configure-the-canary-deployment-strategy-for-azure-linux-virtual-machines"></a>Tutoriel – Configurer la stratégie de déploiement Canary pour des machines virtuelles Linux Azure

## <a name="infrastructure-as-a-service-iaas---configure-cicd"></a>Infrastructure en tant que service (IaaS) - Configurer CI/CD

Azure Pipelines fournit un ensemble d’outils d’automatisation CI/CD pour les déploiements sur des machines virtuelles. Vous pouvez configurer un pipeline de livraison continue pour une machine virtuelle Azure à partir du portail Azure.

Cet article explique comment configurer un pipeline CI/CD qui utilise la stratégie Canary pour les déploiements multi-machine. Le portail Azure prend également en charge d’autres stratégies, telles que [Rolling](./tutorial-devops-azure-pipelines-classic.md) et [Blue-Green](./tutorial-azure-devops-blue-green-strategy.md).

### <a name="configure-cicd-on-virtual-machines"></a>Configurer les outils CI/CD sur les machines virtuelles

Vous pouvez ajouter des machines virtuelles en tant que cibles à un [groupe de déploiement](/azure/devops/pipelines/release/deployment-groups). Vous pouvez ensuite les cibler pour les mises à jour multi-machine. Une fois que vous avez déployé sur des machines, affichez l’**Historique de déploiement** au sein d’un groupe de déploiement. Cette vue vous permet d’effectuer le suivi de la machine virtuelle au pipeline, puis de valider.

### <a name="canary-deployments"></a>Déploiements Canary

Un déploiement Canary réduit les risques en déployant lentement les modifications pour un petit sous-ensemble d’utilisateurs. À mesure que se renforce votre confiance en la nouvelle version, vous pouvez commencer à la mettre en production sur davantage de serveurs de votre infrastructure et y router plus d’utilisateurs.

Vous pouvez configurer des déploiements Canary sur vos « machines virtuelles » à partir du portail Azure via l’option de livraison continue. Voici la procédure pas à pas :

1. Connectez-vous au portail Azure, puis accédez à une machine virtuelle.
1. Dans le volet le plus à gauche des paramètres de la machine virtuelle, sélectionnez **Livraison continue**. Ensuite, sélectionnez **Configurer**.

   ![Le volet Livraison continue avec le bouton Configurer](media/tutorial-devops-azure-pipelines-classic/azure-devops-configure.png)

1. Dans le volet de configuration, sélectionnez **Organisation Azure DevOps** pour choisir un compte existant ou en créer un nouveau. Sélectionnez ensuite le projet dans lequel vous souhaitez configurer le pipeline.  

   ![Le volet Livraison continue](media/tutorial-devops-azure-pipelines-classic/azure-devops-rolling.png)

1. Un groupe de déploiement est un ensemble logique de machines cibles de déploiement qui représentent les environnements physiques. Dev, Test, UAT et Production sont des exemples. Vous pouvez créer un groupe de déploiement, ou en sélectionner un qui existe déjà.
1. Sélectionnez le pipeline de build qui publie le package à déployer sur la machine virtuelle. Le script de déploiement deploy.ps1 ou deploy.sh doit se trouver dans le dossier deployscripts qui est situé dans le dossier racine du package publié. Le pipeline exécute ce script de déploiement.
1. Dans **Stratégie de déploiement**, sélectionnez **Canary**.
1. Ajoutez une étiquette « canary » aux machines virtuelles qui doivent faire partie de déploiements canari. Ajoutez une balise « prod » aux machines virtuelles qui font partie des déploiements effectués après la réussite du déploiement Canary. Les étiquettes vous permettent de cibler uniquement les machines virtuelles qui ont un rôle spécifique.

   ![Le volet Livraison continue, avec la valeur de stratégie de déploiement Canary choisie](media/tutorial-devops-azure-pipelines-classic/azure-devops-configure-canary.png)

1. Sélectionnez **OK** pour configurer le pipeline de livraison continue à déployer sur la machine virtuelle.

   ![Le pipeline de Canary](media/tutorial-devops-azure-pipelines-classic/azure-devops-canary-pipeline.png)

1. Les détails du déploiement de la machine virtuelle s’affichent. Vous pouvez sélectionner le lien pour accéder au pipeline de mise en production dans Azure DevOps. Dans le pipeline de mise en production, sélectionnez **Modifier** pour afficher la configuration du pipeline. Le pipeline comporte les trois phases suivantes :

   1. Cette phase est une phase de groupe de déploiement. Les applications sont déployées sur les machines virtuelles marquées comme « canary ».
   1. Dans cette phase, le pipeline est suspendu et attend une intervention manuelle pour reprendre l’exécution.
   1. Il s’agit à nouveau d’une phase de groupe de déploiement. La mise à jour est désormais déployée sur les machines virtuelles marquées comme « prod ».

      ![Volet du groupe de déploiement pour la tâche de déploiement Canary](media/tutorial-devops-azure-pipelines-classic/azure-devops-canary-task.png)

1. Avant de reprendre l’exécution du pipeline, vérifiez qu’au moins une machine virtuelle porte l’étiquette « prod ». Dans la troisième phase du pipeline, les applications sont déployées uniquement sur les machines virtuelles portant l’étiquette « prod ».

1. La tâche Exécuter le script de déploiement exécute par défaut le script de déploiement deploy.ps1 ou deploy.sh. Le script se trouve dans le dossier deployscripts du dossier racine du package publié. Vérifiez que le pipeline de build sélectionné publie le déploiement dans le dossier racine du package.

   ![Volet Artefacts montrant deploy.sh dans le dossier deployscripts](media/tutorial-deployment-strategy/package.png)

## <a name="other-deployment-strategies"></a>Autres stratégies de déploiement
- [Configurer la stratégie de déploiement Rolling](./tutorial-devops-azure-pipelines-classic.md)
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