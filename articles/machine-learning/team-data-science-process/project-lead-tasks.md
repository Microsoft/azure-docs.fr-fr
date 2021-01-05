---
title: Tâches pour le coordinateur de projet dans Team Data Science Process
description: Présentation détaillée des tâches d’un coordinateur de projet au sein d’une équipe TDSP (Team Data Science Process)
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: 9d9b2546553e03d1555cf4c587d699d9a4ea7e51
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96000978"
---
# <a name="project-lead-tasks-in-the-team-data-science-process"></a>Tâches du coordinateur de projet dans Team Data Science Process

Cet article décrit les tâches qu’un *coordinateur de projets* doit effectuer pour configurer un dépôt pour son équipe de projet dans le processus TDSP ([Team Data Science Process](overview.md)). TDSP est un cadre élaboré par Microsoft qui propose une succession structurée d’activités visant à exécuter avec efficacité des solutions d’analytique prédictive basées sur le cloud. Ce dernier est conçu pour améliorer la collaboration et l’apprentissage en équipe. Pour obtenir une description des rôles et des tâches associées qui incombent aux membres d’une équipe de science des données engagée dans le processus TDSP, consultez [Rôles et tâches du processus TDSP](roles-tasks.md).

Un coordinateur de projet gère les activités quotidiennes des différents scientifiques des données dans le cadre d’un projet de science des données spécifique du processus TDSP. Le diagramme suivant montre le flux de travail pour les tâches du coordinateur de projet :

![Flux des tâches du coordinateur de projet](./media/project-lead-tasks/project-leads-1-tdsp-creating-projects.png)

Ce tutoriel couvre l’étape 1 (Créer un dépôt de projet) et l’étape 2 (Amorcer le dépôt de projet à partir de votre dépôt d’équipe ProjectTemplate). 

Pour l’étape 3 (Créer un élément de travail de fonctionnalité pour le projet) et l’étape 4 (Ajouter des récits pour les phases du projet), consultez [Développement Agile de projets de science des données](agile-development.md).

Pour l’étape 5 (Créer et personnaliser les ressources de stockage/d’analyse et le partage), consultez si nécessaire [Créer des données d’équipe et des ressources d’analytique](team-lead-tasks.md#create-team-data-and-analytics-resources).

Pour l’étape 6 (Configurer le contrôle de sécurité du dépôt de projet), consultez [Ajouter des membres à l’équipe et configurer des autorisations](team-lead-tasks.md#add-team-members-and-configure-permissions).

> [!NOTE] 
> Cet article s’appuie sur Azure Repos pour configurer un projet TDSP, car c’est ainsi que Microsoft implémente le processus TDSP. Si votre équipe utilise une autre plateforme d’hébergement de code, les tâches du coordinateur de projet sont les mêmes, mais la façon de les accomplir peut être différente.

## <a name="prerequisites"></a>Prérequis

Ce tutoriel part du principe que les ressources et autorisations suivantes ont été créées par votre [responsable de groupe](group-manager-tasks.md) et votre [chef d’équipe](team-lead-tasks.md) :

- **Organisation** Azure DevOps pour votre unité de données
- **Projet d’équipe** pour votre équipe de science des données
- **Dépôts** de modèle d’équipe et d’utilitaires
- **Autorisations** sur le compte de votre organisation pour vous permettre de créer et de modifier des dépôts pour votre projet

Pour cloner des dépôts et modifier leur contenu sur votre ordinateur local ou votre machine virtuelle DSVM (Data Science Virtual Machine), ou pour configurer le stockage de fichiers Azure et le monter sur votre machine virtuelle DSVM, vous devez également respecter cette liste de vérification :

- Un abonnement Azure.
- Git installé sur votre ordinateur. Si vous utilisez une machine virtuelle DSVM, Git est préinstallé. Dans le cas contraire, consultez l’[annexe Plateformes et outils](platforms-and-tools.md#appendix).
- Si vous souhaitez utiliser une machine virtuelle DSVM, l’instance DSVM Windows ou Linux doit être créée et configurée dans Azure. Pour obtenir des informations complémentaires et des instructions, consultez la [documentation sur Data Science Virtual Machine](../data-science-virtual-machine/index.yml).
- Pour une machine virtuelle DSVM Windows, [Git Credential Manager (GCM)](https://github.com/Microsoft/Git-Credential-Manager-for-Windows) installé sur votre ordinateur. Dans le fichier *README.md*, accédez à la section **Télécharger et installer**, puis sélectionnez le **programme d’installation le plus récent**. Téléchargez le programme d’installation *.exe* et exécutez-le. 
- Pour une machine virtuelle DSVM Linux, une clé publique SSH créée dans votre instance DSVM et ajoutée à Azure DevOps. Pour obtenir des informations complémentaires et des instructions, consultez la section **Créer une clé publique SSH** dans l’[annexe Plateformes et outils](platforms-and-tools.md#appendix). 

## <a name="create-a-project-repository-in-your-team-project"></a>Créer un dépôt de projet dans votre projet d’équipe

Pour créer un dépôt de projet dans le projet **MyTeam** de votre équipe :

1. Accédez à la page **Résumé** de votre équipe via *https:\//\<server name>/\<organization name>/\<team name>* , par exemple, **https:\//dev.azure.com/DataScienceUnit/MyTeam**, puis sélectionnez **Repos** dans le volet de navigation gauche. 
   
1. Sélectionnez le nom du dépôt en haut de la page, puis **Nouveau dépôt** dans la liste déroulante.
   
   ![Sélectionner Nouveau dépôt](./media/project-lead-tasks/project-leads-9-select-repos.png)
   
1. Dans la boîte de dialogue **Créer un dépôt**, vérifiez que **Git** est sélectionné en dessous de **Type**. Entrez *DSProject1* en dessous de **Nom du dépôt**, puis sélectionnez **Créer**.
   
   ![Créer un dépôt](./media/project-lead-tasks/project-leads-3-create-project-repo-2.png)
   
1. Vérifiez que vous pouvez voir le nouveau dépôt **DSProject1** dans la page des paramètres de votre projet. 
   
   ![Dépôt de projet dans les paramètres du projet](./media/project-lead-tasks/project-leads-4-create-project-repo-3.png)

## <a name="import-the-team-template-into-your-project-repository"></a>Importer le modèle d’équipe dans votre dépôt de projet

Pour renseigner votre dépôt de projet avec le contenu de votre dépôt de modèle d’équipe :

1. Dans la page **Résumé** du projet de votre équipe, sélectionnez **Dépôts** dans le volet de navigation de gauche. 
   
1. Sélectionnez le nom du dépôt en haut de la page, puis **DSProject1** dans la liste déroulante.
   
1. Dans la page **DSProject1 est vide**, sélectionnez **Importer**. 
   
   ![Sélectionner Importer](./media/project-lead-tasks/project-leads-5-create-project-repo-4.png)
   
1. Dans la boîte de dialogue **Importer un dépôt Git**, sélectionnez **Git** comme **Type de source**, puis entrez l’URL du dépôt **TeamTemplate** en dessous de **URL clone**. L’URL est *https:\//\<server name>/\<organization name>/\<team name>/_git/\<team template repository name>* . Par exemple : **https:\//dev.azure.com/DataScienceUnit/MyTeam/_git/TeamTemplate**. 
   
1. Sélectionnez **Importer**. Le contenu de votre dépôt de modèle d’équipe est importé dans votre dépôt de projet. 
   
   ![Importer un dépôt de modèle d’équipe](./media/project-lead-tasks/project-leads-6-create-project-repo-5.png)

Si vous avez besoin de personnaliser le contenu de votre dépôt de projet pour répondre aux besoins spécifiques de votre projet, vous pouvez ajouter, supprimer ou modifier des fichiers et des dossiers dans le dépôt. Vous pouvez travailler directement dans Azure Repos ou cloner le dépôt sur votre ordinateur local ou DSVM, apporter des changements, puis valider et envoyer (push) vos mises à jour au dépôt de projet partagé. Suivez les instructions fournies dans [Personnaliser le contenu des dépôts de l’équipe](team-lead-tasks.md#customize-the-contents-of-the-team-repositories).

## <a name="next-steps"></a>Étapes suivantes

Les liens ci-dessous donnent accès à des descriptions détaillées des autres rôles et tâches définis par le processus TDSP (Team Data Science Process) :

- [Tâches du responsable de groupe pour une équipe de science des données](group-manager-tasks.md)
- [Tâches du responsable d’une équipe de science des données](team-lead-tasks.md)
- [Tâches d’un contributeur individuel pour une équipe de science des données](project-ic-tasks.md)