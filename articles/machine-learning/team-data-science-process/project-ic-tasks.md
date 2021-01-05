---
title: Tâches pour un contributeur individuel dans TDSP (Team Data Science Process)
description: Séquence détaillée des tâches d’un contributeur individuel sur un projet d’équipe de science des données.
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: 4ecb5fef9c9b14bde72de29a45e29d7e16131bd1
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96000995"
---
# <a name="tasks-for-an-individual-contributor-in-the-team-data-science-process"></a>Tâches pour un contributeur individuel dans TDSP (Team Data Science Process)

Cette rubrique décrit les tâches qu’un *contributeur individuel* doit effectuer pour configurer un projet dans le processus TDSP ([Team Data Science Process](overview.md)). L’objectif est de travailler dans un environnement de collaboration en équipe conforme au processus TDSP. Ce dernier est conçu pour améliorer la collaboration et l’apprentissage en équipe. Pour obtenir une description des rôles des membres de l’équipe de science des données et des tâches qui leur incombent dans le cadre du processus TDSP, consultez [Rôles et tâches du processus TDSP](roles-tasks.md).

Le diagramme suivant montre les tâches que chaque contributeur de projet (scientifique des données) doit effectuer pour configurer son environnement d’équipe. Pour obtenir des instructions sur la façon d’exécuter un projet de science des données dans le cadre du processus TDSP, consultez [Exécution de projets de science des données](./agile-development.md). 

![Tâches d’un contributeur individuel](./media/project-ic-tasks/project-ic-1-tdsp-data-scientist.png)

- **ProjectRepository** désigne le dépôt géré par votre équipe de projet pour partager les modèles et les ressources du projet.
- **TeamUtilities** désigne le dépôt d’utilitaires que votre équipe gère spécifiquement pour votre équipe. 
- **GroupUtilities** désigne le dépôt que votre groupe gère pour partager les utilitaires nécessaires dans l’ensemble du groupe. 

> [!NOTE] 
> Cet article s’appuie sur Azure Repos et sur une machine virtuelle DSVM (Data Science Virtual Machine) pour mettre en place un environnement TDSP, car c’est ainsi que Microsoft implémente le processus TDSP. Si votre équipe utilise d’autres plateformes d’hébergement de code ou de développement, les tâches de chaque contributeur sont les mêmes, mais la façon de les accomplir peut être différente.

## <a name="prerequisites"></a>Prérequis

Ce tutoriel part du principe que les ressources et autorisations suivantes ont été configurées par votre [responsable de groupe](group-manager-tasks.md), votre [chef d’équipe](team-lead-tasks.md), et votre [coordinateur de projet](project-lead-tasks.md) :

- **Organisation** Azure DevOps pour votre unité de science des données
- **Dépôt de projet** configuré par votre coordinateur de projet pour partager des ressources et des modèles de projet
- Dépôts **GroupUtilities** et **TeamUtilities** configurés par le responsable de groupe et le chef d’équipe, le cas échéant
- **Stockage de fichiers** Azure configuré pour les ressources partagées par votre équipe ou projet, le cas échéant
- **Autorisations** permettant de cloner le dépôt de projet et de renvoyer (push) les changements au dépôt 

Pour cloner des référentiels et modifier leur contenu sur votre ordinateur local ou machine virtuelle DSVM, ou pour monter le stockage de fichiers Azure sur votre machine virtuelle DSVM, vous devez suivre cette check-list :

- Un abonnement Azure.
- Git installé sur votre ordinateur. Si vous utilisez une machine virtuelle DSVM, Git est préinstallé. Dans le cas contraire, consultez l’[annexe Plateformes et outils](platforms-and-tools.md#appendix).
- Si vous souhaitez utiliser une machine virtuelle DSVM, l’instance DSVM Windows ou Linux doit être créée et configurée dans Azure. Pour obtenir des informations complémentaires et des instructions, consultez la [documentation sur Data Science Virtual Machine](../data-science-virtual-machine/index.yml).
- Pour une machine virtuelle DSVM Windows, [Git Credential Manager (GCM)](https://github.com/Microsoft/Git-Credential-Manager-for-Windows) installé sur votre ordinateur. Dans le fichier *README.md*, accédez à la section **Télécharger et installer**, puis sélectionnez le **programme d’installation le plus récent**. Téléchargez le programme d’installation *.exe* et exécutez-le. 
- Pour une machine virtuelle DSVM Linux, une clé publique SSH créée dans votre instance DSVM et ajoutée à Azure DevOps. Pour obtenir des informations complémentaires et des instructions, consultez la section **Créer une clé publique SSH** dans l’[annexe Plateformes et outils](platforms-and-tools.md#appendix). 
- Informations relatives au stockage de fichiers Azure pour tout stockage de ce type à monter sur votre machine virtuelle DSVM. 

## <a name="clone-repositories"></a>Cloner des dépôts

Pour utiliser localement des dépôts et envoyer (push) vos changements aux dépôts d’équipe et de projet partagés, vous devez d’abord copier ou *cloner* les dépôts sur votre ordinateur local. 

1. Dans Azure DevOps, accédez à la page Résumé du projet de votre équipe via *https:\//\<server name>/\<organization name>/\<team name>* , par exemple, **https:\//dev.azure.com/DataScienceUnit/MyTeam**.
   
1. Sélectionnez **Dépôts** dans le volet de navigation gauche puis, en haut de la page, sélectionnez le dépôt à cloner.
   
1. Dans la page du dépôt, sélectionnez **Cloner** en haut à droite.
   
1. Dans la boîte de dialogue **Cloner le dépôt**, sélectionnez **HTTPS** pour une connexion HTTP ou **SSH** pour une connexion SSH, puis copiez l’URL clone figurant sous **Ligne de commande** dans votre Presse-papiers.
   
   ![Cloner le dépôt](./media/project-ic-tasks/clone.png)
   
1. Sur votre ordinateur local ou machine virtuelle DSVM, créez les répertoires suivants :
   
   - Pour Windows : **C:\GitRepos**
   - Pour Linux : **$home/GitRepos**
   
1. Passez au répertoire que vous avez créé.
   
1. Dans Git Bash, exécutez la commande `git clone <clone URL>` pour chaque dépôt à cloner. 
   
   Par exemple, la commande suivante clone le dépôt **TeamUtilities** dans le répertoire *MyTeam* sur votre ordinateur local. 
   
   **Connexion HTTPS :**
   
   ```bash
   git clone https://DataScienceUnit@dev.azure.com/DataScienceUnit/MyTeam/_git/TeamUtilities
   ```
   
   **Connexion SSH :**
   
   ```bash
   git clone git@ssh.dev.azure.com:v3/DataScienceUnit/MyTeam/TeamUtilities
   ```
   
1. Vérifiez que vous pouvez voir les dossiers des dépôts clonés dans votre répertoire de projet local.
   
   ![Trois dossiers de dépôt locaux](./media/project-ic-tasks/project-ic-5-three-repo-cloned-to-ic-linux.png)

## <a name="mount-azure-file-storage-to-your-dsvm"></a>Monter un stockage de fichiers Azure sur votre machine virtuelle DSVM

Si votre équipe ou projet a partagé des ressources dans le stockage de fichiers Azure, montez-le sur votre ordinateur local ou machine virtuelle DSVM. Suivez les instructions fournies dans [Monter le stockage de fichiers Azure sur votre ordinateur local ou machine virtuelle DSVM](team-lead-tasks.md#mount-azure-file-storage-on-your-local-machine-or-dsvm).

## <a name="next-steps"></a>Étapes suivantes

Les liens ci-dessous donnent accès à des descriptions détaillées des autres rôles et tâches définis par le processus TDSP (Team Data Science Process) :

- [Tâches du responsable de groupe pour une équipe de science des données](group-manager-tasks.md)
- [Tâches du responsable d’une équipe de science des données](team-lead-tasks.md)
- [Tâches du coordinateur de projet pour une équipe de science des données](project-lead-tasks.md)