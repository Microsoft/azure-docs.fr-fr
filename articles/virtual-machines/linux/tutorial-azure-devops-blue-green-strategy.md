---
title: Tutoriel – Configurer des déploiements Canary pour les machines virtuelles Linux Azure
description: Dans ce tutoriel, vous allez apprendre à configurer un pipeline de déploiement continu (CD). Ce pipeline met à jour un groupe de machines virtuelles Linux Azure à l’aide de la stratégie de déploiement Blue-Green.
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
ms.openlocfilehash: f349ff62fe211f0610341864a4c7528ee6bfe9c5
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91961525"
---
# <a name="tutorial---configure-the-blue-green-deployment-strategy-for-azure-linux-virtual-machines"></a>Tutoriel – Configurer une stratégie de déploiement Blue-Green pour des machines virtuelles Linux Azure

## <a name="infrastructure-as-a-service-iaas---configure-cicd"></a>Infrastructure en tant que service (IaaS) - Configurer CI/CD

Azure Pipelines fournit un ensemble d’outils d’automatisation CI/CD pour les déploiements sur des machines virtuelles. Vous pouvez configurer un pipeline de livraison continue pour une machine virtuelle Azure à partir du portail Azure.

Cet article explique comment configurer un pipeline CI/CD qui utilise la stratégie Blue-Green pour les déploiements multi-machine. Le portail Azure prend également en charge d’autres stratégies, telles que [Rolling](./tutorial-devops-azure-pipelines-classic.md) et [Canary](./tutorial-azure-devops-canary-strategy.md).

### <a name="configure-cicd-on-virtual-machines"></a>Configurer les outils CI/CD sur les machines virtuelles

Vous pouvez ajouter des machines virtuelles en tant que cibles à un [groupe de déploiement](/azure/devops/pipelines/release/deployment-groups). Vous pouvez ensuite les cibler pour les mises à jour multi-machine. Une fois que vous avez déployé sur des machines, affichez l’**Historique de déploiement** au sein d’un groupe de déploiement. Cette vue vous permet d’effectuer le suivi de la machine virtuelle au pipeline, puis de valider.

### <a name="blue-green-deployments"></a>Déploiement Blue-Green

Le déploiement Blue-Green réduit les temps d’arrêt à l’aide d’un environnement de secours identique. Un seul environnement est actif à un moment donné.

Pendant la préparation d’une nouvelle mise en production, vous effectuez la dernière étape de tests dans l’environnement « green ». Une fois que le logiciel fonctionne dans l’environnement vert, changez le trafic pour router toutes les requêtes entrantes en direction de l’environnement vert. L’environnement bleu est inactif.

Vous pouvez configurer des déploiements Blue-Green sur vos « machines virtuelles » à partir du portail Azure via l’option de livraison continue. Voici la procédure pas à pas :

1. Connectez-vous au portail Azure, puis accédez à une machine virtuelle.
1. Dans le volet le plus à gauche des paramètres de la machine virtuelle, sélectionnez **Livraison continue**. Ensuite, sélectionnez **Configurer**.

   ![Le volet Livraison continue avec le bouton Configurer](media/tutorial-devops-azure-pipelines-classic/azure-devops-configure.png)

1. Dans le volet de configuration, sélectionnez **Organisation Azure DevOps** pour choisir un compte existant ou en créer un nouveau. Sélectionnez ensuite le projet dans lequel vous souhaitez configurer le pipeline.  

   ![Le volet Livraison continue](media/tutorial-devops-azure-pipelines-classic/azure-devops-rolling.png)

1. Un groupe de déploiement est un ensemble logique de machines cibles de déploiement qui représentent les environnements physiques. Dev, Test, UAT et Production sont des exemples. Vous pouvez créer un groupe de déploiement, ou en sélectionner un qui existe déjà.
1. Sélectionnez le pipeline de build qui publie le package à déployer sur la machine virtuelle. Le script de déploiement deploy.ps1 ou deploy.sh doit se trouver dans le dossier deployscripts qui est situé dans le dossier racine du package publié. Le pipeline exécute ce script de déploiement.
1. Dans **Stratégie de déploiement**, sélectionnez **Blue-Green**.
1. Ajoutez une étiquette « blue » ou « green » aux machines virtuelles qui doivent faire partie des déploiements Blue-Green. Si une machine virtuelle est destinée à un rôle de secours, ajoutez-lui l’étiquette « green ». Sinon, marquez-la comme « blue ».

   ![Le volet Livraison continue, avec la valeur de stratégie de déploiement Blue-Green choisie](media/tutorial-devops-azure-pipelines-classic/azure-devops-blue-green-configure.png)

1. Sélectionnez **OK** pour configurer le pipeline de livraison continue à déployer sur la machine virtuelle.

   ![Le pipeline Blue-Green](media/tutorial-devops-azure-pipelines-classic/azure-devops-blue-green-pipeline.png)

1. Les détails du déploiement de la machine virtuelle s’affichent. Vous pouvez sélectionner le lien pour accéder au pipeline de mise en production dans Azure DevOps. Dans le pipeline de mise en production, sélectionnez **Modifier** pour afficher la configuration du pipeline. Le pipeline comporte les trois phases suivantes :

   1. Cette phase est une phase de groupe de déploiement. Les applications sont déployées sur des machines virtuelles de secours, marquées comme « green ».
   1. Dans cette phase, le pipeline est suspendu et attend une intervention manuelle pour reprendre l’exécution. Les utilisateurs peuvent reprendre l’exécution du pipeline une fois qu’ils ont assuré manuellement la stabilité du déploiement sur les machines virtuelles marquées comme « green ».
   1. Cette phase permute les balises « blue » et « green » dans les machines virtuelles. Cela permet de s’assurer que les machines virtuelles avec des versions d’application plus anciennes sont désormais marquées comme « green ». Lors de la prochaine exécution du pipeline, les applications seront déployées sur ces machines virtuelles.

      ![Volet du groupe de déploiement pour la tâche de déploiement Blue-Green](media/tutorial-devops-azure-pipelines-classic/azure-devops-blue-green-tasks.png)

1. La tâche Exécuter le script de déploiement exécute par défaut le script de déploiement deploy.ps1 ou deploy.sh. Le script se trouve dans le dossier deployscripts du dossier racine du package publié. Vérifiez que le pipeline de build sélectionné publie le déploiement dans le dossier racine du package.

   ![Volet Artefacts montrant deploy.sh dans le dossier deployscripts](media/tutorial-deployment-strategy/package.png)

## <a name="other-deployment-strategies"></a>Autres stratégies de déploiement

- [Configurer la stratégie de déploiement Rolling](./tutorial-devops-azure-pipelines-classic.md)
- [Configurer la stratégie de déploiement Canary](./tutorial-azure-devops-canary-strategy.md)

## <a name="azure-devops-projects"></a>Azure DevOps Projects

Vous pouvez commencer à utiliser Azure facilement. Avec Azure DevOps Projects, commencez à exécuter votre application sur n’importe quel service Azure en seulement trois étapes en sélectionnant :

- Une langue d'application
- Un runtime
- Un service Azure

[Plus d’informations](https://azure.microsoft.com/features/devops-projects/)

## <a name="additional-resources"></a>Ressources supplémentaires

- [Déployer sur des machines virtuelles Azure avec Azure DevOps Projects](../../devops-project/azure-devops-project-vms.md)
- [Implémenter un déploiement continu de votre application sur un groupe de machines virtuelles identiques Azure](/azure/devops/pipelines/apps/cd/azure/deploy-azure-scaleset)