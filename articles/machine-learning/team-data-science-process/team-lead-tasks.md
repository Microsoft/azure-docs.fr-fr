---
title: Tâches pour responsable d’équipe dans Team Data Science Process
description: Présentation détaillée des tâches d’un responsable d’équipe TDSP (Team Data Science Process)
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: df7d2278487c1b098615a14562c498b9187c56eb
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96000026"
---
# <a name="tasks-for-the-team-lead-on-a-team-data-science-process-team"></a>Tâches du responsable d’équipe TDSP (Team Data Science Process)

Cet article décrit les tâches qu’effectue un *responsable d’équipe* pour son équipe de science des données. L’objectif du responsable d’équipe est d’établir un environnement d’équipe collaboratif standard conforme au processus [TDSP](overview.md) (Team Data Science Process). Ce dernier est conçu pour améliorer la collaboration et l’apprentissage en équipe. 

Le processus TDSP est une méthodologie de science des données agile et itérative destinée à proposer de manière efficace des solutions d’analytique prédictive et des applications intelligentes. Le processus présente les meilleures pratiques et les structures de Microsoft et du secteur.  L’objectif est de réussir l’implémentation d’initiatives de science des données et de tirer pleinement profit des avantages de leurs programmes d’analyse. Pour obtenir une description des rôles et des tâches associées qui incombent aux membres d’une équipe de science des données engagée dans le processus TDSP, consultez [Rôles et tâches du processus TDSP](roles-tasks.md).

Un responsable d’équipe gère une équipe composée de plusieurs scientifiques des données dans la division Science des données d’une grande entreprise. Selon la taille et la structure de la division Science des données, le [responsable de groupe](group-manager-tasks.md) et le responsable d’équipe peuvent être la même personne, ou ils peuvent déléguer leurs tâches à des agents. Cela dit, les tâches confiées restent les mêmes. 

Le schéma suivant présente le workflow des tâches qu’effectue le responsable d’équipe pour mettre en place un environnement d’équipe :

![Workflow des tâches du responsable d’équipe](./media/team-lead-tasks/team-leads-1-creating-teams.png)

1. Créer un **projet d’équipe** dans l’organisation du groupe dans Azure DevOps. 
  
1. Renommer le dépôt par défaut de l’équipe **TeamUtilities**.
  
1. Créer un dépôt **TeamTemplate** dans le projet de l’équipe. 
  
1. Importer le contenu des dépôts **GroupUtilities** et **GroupProjectTemplate** du groupe dans les dépôts **TeamUtilities** et **TeamTemplate**. 
  
1. Configurer le **contrôle de sécurité** en ajoutant des membres à l’équipe et en configurant leurs autorisations.
  
1. Si nécessaire, créer des données d’équipe et des ressources d’analytique :
   - Ajouter des utilitaires propres à l’équipe au dépôt **TeamUtilities**. 
   - Créer un **stockage de fichiers Azure** pour y stocker les ressources de données qui peuvent profiter à toute l’équipe. 
   - Monter le stockage de fichiers Azure sur la machine virtuelle **DSVM** (Data Science Virtual Machine) du responsable d’équipe et y ajouter des ressources de données.

Le tutoriel suivant présente en détail les différentes étapes.

> [!NOTE] 
> Cet article s’appuie sur Azure DevOps et sur une machine virtuelle DSVM pour mettre en place un environnement d’équipe TDSP, car c’est ainsi que Microsoft implémente le processus TDSP. Si votre équipe utilise d’autres plateformes d’hébergement de code ou de développement, les tâches du responsable d’équipe sont les mêmes, mais la façon de les accomplir peut être différente.

## <a name="prerequisites"></a>Prérequis

Ce tutoriel part du principe que les ressources et les autorisations suivantes ont été créées par votre [responsable de groupe](group-manager-tasks.md) :

- **Organisation** Azure DevOps pour votre unité de données
- Les dépôts **GroupProjectTemplate** et **GroupUtilities** remplis avec le contenu des dépôts **ProjectTemplate** et **Utilities** de l’équipe Microsoft TDSP
- Les autorisations sur le compte de votre organisation pour vous permettre de créer des projets et des dépôts pour votre équipe

Pour pouvoir cloner des dépôts et modifier leur contenu sur votre ordinateur local ou machine virtuelle DSVM, ou pour pouvoir configurer le stockage de fichiers Azure et le monter sur votre machine virtuelle DSVM, vous avez besoin des éléments suivants :

- Un abonnement Azure.
- Git installé sur votre ordinateur. Si vous utilisez une machine virtuelle DSVM, Git est préinstallé. Dans le cas contraire, consultez l’[annexe Plateformes et outils](platforms-and-tools.md#appendix).
- Si vous souhaitez utiliser une machine virtuelle DSVM, l’instance DSVM Windows ou Linux doit être créée et configurée dans Azure. Pour obtenir des informations complémentaires et des instructions, consultez la [documentation sur Data Science Virtual Machine](../data-science-virtual-machine/index.yml).
- Pour une machine virtuelle DSVM Windows, [Git Credential Manager (GCM)](https://github.com/Microsoft/Git-Credential-Manager-for-Windows) installé sur votre ordinateur. Dans le fichier *README.md*, accédez à la section **Télécharger et installer**, puis sélectionnez le **programme d’installation le plus récent**. Téléchargez le programme d’installation *.exe* et exécutez-le. 
- Pour une machine virtuelle DSVM Linux, une clé publique SSH créée dans votre instance DSVM et ajoutée à Azure DevOps. Pour obtenir des informations complémentaires et des instructions, consultez la section **Créer une clé publique SSH** dans l’[annexe Plateformes et outils](platforms-and-tools.md#appendix). 

## <a name="create-a-team-project-and-repositories"></a>Créer un projet d’équipe et des dépôts

Dans cette section, vous allez créer les ressources suivantes dans l’organisation Azure DevOps de votre groupe :

- Le projet **MyTeam** dans Azure DevOps
- Le dépôt **TeamTemplate**
- Le dépôt **TeamUtilities**

Dans ce didacticiel, les noms des dépôts et des répertoires ont été spécifiés en partant de l’hypothèse que vous voulez créer un projet distinct pour votre propre équipe, laquelle dépend d’une plus grande organisation de science des données. Or, le groupe dans son ensemble peut décider de travailler dans le cadre d’un projet unique créé par le responsable de groupe ou l’administrateur de l’organisation. Dans ce cas, toutes les équipes de science des données créent des dépôts pour ce même projet. Ce scénario peut convenir pour :
- Un petit groupe chargé de la science des données qui ne regroupe pas plusieurs équipes de science des données. 
- Un groupe chargé de la science des données plus étoffé qui compte en son sein plusieurs équipes de science des données et qui souhaite néanmoins optimiser la collaboration entre les équipes au moyen d’activités comme la planification sprint au niveau du groupe. 

Si les équipes préfèrent créer leurs propres référentiels pour un même projet de groupe, les responsables d’équipe doivent créer les référentiels nommés *\<TeamName>Template* et *\<TeamName>Utilities*. Exemple : *ModèleÉquipeA* et *UtilitairesÉquipeA*. 

Dans tous les cas, les responsables d’équipe doivent indiquer aux membres de leur équipe quels dépôts de modèles et d’utilitaires ils doivent créer et cloner. Les coordinateurs de projet doivent suivre les instructions de la rubrique [Tâches du coordinateur de projet pour une équipe de science des données](project-lead-tasks.md) pour créer des dépôts de projet, que ce soit pour plusieurs projets distincts ou pour un seul et même projet. 

### <a name="create-the-myteam-project"></a>Créer le projet MyTeam

Pour créer un projet distinct pour votre équipe :

1. Dans votre navigateur web, accédez à la page d’accueil de l’organisation Azure DevOps de votre groupe à l’URL *https:\//\<server name>/\<organization name>* , puis sélectionnez **Nouveau projet**. 
   
   ![Sélectionner Nouveau projet](./media/team-lead-tasks/team-leads-2-create-new-team.png)
   
1. Dans la boîte de dialogue **Créer un projet**, entrez le nom de votre équipe, par exemple *MyTeam*, en dessous de **Nom du projet**, puis sélectionnez **Avancé**. 
   
1. En dessous de **Gestion de version**, sélectionnez **Git**, puis en dessous de **Processus d’élément de travail**, sélectionnez **Agile**. Sélectionnez ensuite **Créer**. 
   
   ![Créer un projet](./media/team-lead-tasks/team-leads-3-create-new-team-2.png)
   
La page **Résumé** du projet d’équipe s’ouvre à l’URL *https:\//\<server name>/\<organization name>/\<team name>* .

### <a name="rename-the-myteam-default-repository-to-teamutilities"></a>Renommer le dépôt par défaut MyTeam TeamUtilities

1. Dans la page **Résumé** de **MyTeam**, en dessous de **Par quel service voulez-vous commencer ?** , sélectionnez **Dépôts**. 
   
   ![Sélectionner Dépôts](./media/team-lead-tasks/team-leads-6-rename-team-project-repo.png)
   
1. Dans la page du dépôt **MyTeam**, sélectionnez le dépôt **MyTeam** en haut de la page, puis sélectionnez **Gérer les dépôts** dans la liste déroulante. 
   
   ![Sélectionner Gérer les dépôts](./media/team-lead-tasks/team-leads-7-rename-team-project-repo-2.png)
1. Dans la page **Paramètres du projet**, sélectionnez **...** en regard du dépôt **MyTeam**, puis sélectionnez **Renommer le dépôt**. 
   
   ![Sélectionner Renommer le dépôt](./media/team-lead-tasks/team-leads-8-rename-team-project-repo-3.png)
   
1. Dans la fenêtre contextuelle **Renommer le dépôt MyTeam**, entrez *TeamUtilities*, puis sélectionnez **Renommer**. 

### <a name="create-the-teamtemplate-repository"></a>Créer le dépôt TeamTemplate

1. Dans la page **Paramètres du projet**, sélectionnez **Nouveau dépôt**. 
   
   ![Sélectionner Nouveau dépôt](./media/team-lead-tasks/team-leads-9-create-team-utilities.png)
   
   Vous pouvez aussi sélectionner **Dépôts** dans le volet de navigation gauche de la page **Résumé** du projet **MyTeam**, sélectionner un dépôt en haut de la page, puis sélectionner **Nouveau dépôt** dans le menu déroulant.
   
1. Dans la boîte de dialogue **Créer un dépôt**, vérifiez que **Git** est sélectionné en dessous de **Type**. Entrez *TeamTemplate* en dessous de **Nom du dépôt**, puis sélectionnez **Créer**.
   
   ![Créer un dépôt](./media/team-lead-tasks/team-leads-10-create-team-utilities-2.png)
   
1. Vérifiez que les deux dépôts **TeamUtilities** et **TeamTemplate** figurent dans la page des paramètres de votre projet. 
   
   ![Les deux dépôts de l’équipe](./media/team-lead-tasks/team-leads-11-two-repo-in-team.png)

### <a name="import-the-contents-of-the-group-common-repositories"></a>Importer le contenu des dépôts communs du groupe

Pour remplir les dépôts de votre équipe avec le contenu des dépôts communs du groupe créés par votre responsable de groupe :

1. Dans la page d’accueil de votre projet **MyTeam**, sélectionnez **Dépôts** dans le volet de navigation gauche. Si vous obtenez un message indiquant que le modèle **MyTeam** est introuvable, sélectionnez le lien dans **. Sinon, accédez à votre dépôt TeamTemplate par défaut.** 
   
   Le dépôt **TeamTemplate** par défaut s’ouvre. 
   
1. Dans la page **TeamTemplate est vide**, sélectionnez **Importer**. 
   
   ![Sélectionner Importer](./media/team-lead-tasks/import-repo.png)
   
1. Dans la boîte de dialogue **Importer un dépôt Git**, sélectionnez **Git** comme **Type de source**, puis entrez l’URL du dépôt de modèles commun de votre groupe en dessous de **URL clone**. L’URL est *https:\//\<server name>/\<organization name>/_git/\<repository name>* . Par exemple : *https:\//dev.azure.com/DataScienceUnit/GroupCommon/_git/GroupProjectTemplate*. 
   
1. Sélectionnez **Importer**. Le contenu du dépôt de modèles de votre groupe est importé dans le dépôt de modèles de votre équipe. 
   
   ![Importer le dépôt de modèles commun du groupe](./media/team-lead-tasks/import-repo-2.png)
   
1. En haut de la page **Dépôts** de votre projet, déroulez la liste et sélectionnez le dépôt **TeamUtilities**.
   
1. Répétez le processus d’importation pour importer le contenu du dépôt d’utilitaires commun de votre groupe, par exemple *GroupUtilities*, dans votre dépôt **TeamUtilities**. 
   
À présent, les deux dépôts de votre équipe contiennent chacun les fichiers du dépôt commun du groupe correspondant. 

### <a name="customize-the-contents-of-the-team-repositories"></a>Personnaliser le contenu des dépôts de l’équipe

Si vous voulez personnaliser le contenu des dépôts de votre équipe pour répondre à ses besoins spécifiques, vous pouvez le faire dès maintenant. Vous pouvez modifier des fichiers, changer la structure des répertoires ou ajouter des fichiers et dossiers.

Pour modifier, charger ou créer des fichiers ou dossiers directement dans Azure DevOps :

1. Dans la page **Résumé** du projet **MyTeam**, sélectionnez **Dépôts**. 
   
1. En haut de la page, sélectionnez le dépôt que vous voulez personnaliser.

1. Dans la structure de répertoires du dépôt, accédez au dossier ou fichier à modifier. 
   
   - Pour créer des dossiers ou des fichiers, sélectionnez la flèche en regard de **Nouveau**. 
     
     ![Créer un fichier](./media/team-lead-tasks/new-file.png)
     
   - Pour charger des fichiers, sélectionnez **Charger le ou les fichiers**. 
     
     ![Charger des fichiers](./media/team-lead-tasks/upload-files.png)
     
   - Pour modifier des fichiers existants, accédez au fichier, puis sélectionnez **Modifier**. 
     
     ![Modifier un fichier](./media/team-lead-tasks/edit-file.png)
     
1. Après avoir ajouté ou modifié des fichiers, sélectionnez **Validation**.
   
   ![Valider les modifications](./media/team-lead-tasks/commit.png)

Pour pouvoir utiliser les dépôts sur votre ordinateur local ou machine virtuelle DSVM, vous devez d’abord copier ou *cloner* les dépôts sur votre ordinateur local, puis valider et envoyer (push) vos modifications aux dépôts partagés de l’équipe. 

Pour cloner les dépôts :

1. Dans la page **Résumé** du projet **MyTeam**, sélectionnez **Dépôts** puis, en haut de la page, sélectionnez le dépôt que vous voulez cloner.
   
1. Dans la page du dépôt, sélectionnez **Cloner** en haut à droite.
   
1. Dans la boîte de dialogue **Cloner le dépôt**, en dessous de **Ligne de commande**, sélectionnez **HTTPS** pour une connexion HTTP ou **SSH** pour une connexion SSH, puis copiez l’URL clone dans votre Presse-papiers.
   
   ![Copier l’URL clone](./media/team-lead-tasks/clone.png)
   
1. Sur votre ordinateur local, créez les répertoires suivants :
   
   - Pour Windows : **C:\GitRepos\MyTeam**
   - Pour Linux, **$home/GitRepos/MyTeam** 
   
1. Passez au répertoire que vous avez créé.
   
1. Dans Git Bash, exécutez la commande `git clone <clone URL>`, où \<clone URL> est l’URL que vous avez copiée à partir de la boîte de dialogue **Cloner**.
   
   Par exemple, utilisez l’une des commandes suivantes pour cloner le dépôt **TeamUtilities** dans le répertoire *MyTeam* de votre ordinateur local. 
   
   **Connexion HTTPS :**
   
   ```bash
   git clone https://DataScienceUnit@dev.azure.com/DataScienceUnit/MyTeam/_git/TeamUtilities
   ```
   
   **Connexion SSH :**
   
   ```bash
   git clone git@ssh.dev.azure.com:v3/DataScienceUnit/MyTeam/TeamUtilities
   ```

Après avoir effectué les modifications voulues dans le clone local de votre répertoire, validez et envoyez (push) les modifications aux répertoires partagés de l’équipe. 

Exécutez les commandes Git Bash suivantes à partir de votre répertoire local **GitRepos\MyTeam\TeamTemplate** ou **GitRepos\MyTeam\TeamUtilities**.

```bash
git add .
git commit -m "push from local"
git push
```

> [!NOTE]
> S’il s’agit de votre première validation dans un dépôt Git, vous pouvez être amené à configurer les paramètres globaux *user.name* et *user.email* avant d’exécuter la commande `git commit`. Exécutez les deux commandes suivantes :
> 
> `git config --global user.name <your name>`
> 
> `git config --global user.email <your email address>`
> 
> Si la validation porte sur plusieurs dépôts Git, utilisez le même nom et la même adresse e-mail pour tous. L’utilisation du même nom et de la même adresse e-mail s’avère pratique quand vous créez des tableaux de bord Power BI afin de suivre vos activités Git dans plusieurs dépôts.

## <a name="add-team-members-and-configure-permissions"></a>Ajouter des membres à l’équipe et configurer des autorisations

Pour ajouter des membres à l’équipe :

1. Dans Azure DevOps, à partir de la page d’accueil du projet **MyTeam**, sélectionnez **Paramètres du projet** dans le volet de navigation gauche. 
   
1. Dans le volet de navigation gauche **Paramètres du projet**, sélectionnez **Équipes** puis, dans la page **Équipes**, sélectionnez l’**équipe MyTeam**. 
   
   ![Configurer des équipes](./media/team-lead-tasks/teams.png)
   
1. Dans la page **Profil de l’équipe**, sélectionnez **Ajouter**.
   
   ![Ajouter à l’équipe MyTeam](./media/team-lead-tasks/add-to-team.png)
   
1. Dans la boîte de dialogue **Ajouter des utilisateurs et des groupes**, recherchez les membres à ajouter au groupe, sélectionnez-les, puis sélectionnez **Enregistrer les modifications**. 
   
   ![Ajoutez des utilisateurs et des groupes](./media/team-lead-tasks/add-users.png)
   

Pour configurer des autorisations pour les membres de l’équipe :

1. Dans le volet de navigation gauche **Paramètres du projet**, sélectionnez **Autorisations**. 
   
1. Dans la page **Autorisations**, sélectionnez le groupe auquel vous voulez ajouter les membres. 
   
1. Dans la page du groupe, sélectionnez **Membres**, puis **Ajouter**. 
   
1. Dans la fenêtre contextuelle **Inviter des membres**, recherchez les membres à ajouter au groupe, sélectionnez-les, puis sélectionnez **Enregistrer**. 
   
   ![Accorder des autorisations aux membres](./media/team-lead-tasks/grant-permissions.png)

## <a name="create-team-data-and-analytics-resources"></a>Créer des données d’équipe et des ressources d’analytique

Bien que cette étape soit facultative, il est très avantageux de partager des données et des ressources d’analytique avec l’ensemble de votre équipe, aussi bien en termes de performances et que de coûts. Les membres de l’équipe peuvent exécuter leurs projets sur les ressources partagées, faires des économies budgétaires et collaborer avec plus d’efficacité. Vous pouvez créer un stockage de fichiers Azure et le monter sur votre machine virtuelle DSVM pour le partager avec les membres de l’équipe. 

Pour savoir comment partager d’autres ressources avec votre équipe, par exemple des clusters Azure HDInsight Spark, consultez [Plateformes et outils](platforms-and-tools.md). Cette rubrique vous conseille sur les ressources à sélectionner pour vos besoins en science des données et propose des liens vers des pages produits et d’autres tutoriels utiles.

>[!NOTE]
> Pour éviter que les données soient transmises à plusieurs centres de données, ce qui peut occasionner des lenteurs et des coûts, veillez à ce que le groupe de ressources Azure, le compte de stockage et la machine virtuelle DSVM soient tous stockés dans la même région Azure. 

### <a name="create-azure-file-storage"></a>Créer un stockage de fichiers Azure

1. Exécutez le script suivant pour créer un stockage de fichiers Azure pour les ressources de données utiles à l’ensemble de l’équipe. Le script vous invite à fournir des informations sur votre abonnement Azure ; préparez-les en vue de les entrer. 

   - Sur un ordinateur Windows, exécutez le script à partir de l’invite de commandes PowerShell :
     
     ```powershell
     wget "https://raw.githubusercontent.com/Azure/Azure-MachineLearning-DataScience/master/Misc/TDSP/CreateFileShare.ps1" -outfile "CreateFileShare.ps1"
     .\CreateFileShare.ps1
     ```
     
   - Sur un ordinateur Linux, exécutez le script à partir de l’interpréteur de commandes Linux :
     
     ```shell
     wget "https://raw.githubusercontent.com/Azure/Azure-MachineLearning-DataScience/master/Misc/TDSP/CreateFileShare.sh"
     bash CreateFileShare.sh
     ```
   
1. Connectez-vous à votre compte Microsoft Azure quand vous y êtes invité, puis sélectionnez l’abonnement que vous souhaitez utiliser.
   
1. Sélectionnez le compte de stockage à utiliser ou créez-en un dans l’abonnement sélectionné. Vous pouvez utiliser des caractères en minuscules, des chiffres et des traits d’union dans le nom du stockage de fichiers Azure.
   
1. Pour faciliter le montage et le partage du stockage, appuyez sur Entrée ou entrez *Y* pour enregistrer les informations sur le stockage de fichiers Azure dans un fichier texte dans le répertoire actif. Vous pouvez archiver ce fichier texte dans votre dépôt **TeamTemplate**, de préférence sous **Docs\DataDictionaries**, pour le rendre accessible à tous les projets de votre équipe. Vous avez aussi besoin des informations du fichier pour monter votre stockage de fichiers Azure sur votre machine virtuelle Azure DSVM dans la section suivante. 
   
### <a name="mount-azure-file-storage-on-your-local-machine-or-dsvm"></a>Monter le stockage de fichiers Azure sur votre ordinateur local ou machine virtuelle DSVM

1. Pour monter votre stockage de fichiers Azure sur votre ordinateur local ou machine virtuelle DSVM, utilisez le script suivant.
   
   - Sur un ordinateur Windows, exécutez le script à partir de l’invite de commandes PowerShell :
     
     ```powershell
     wget "https://raw.githubusercontent.com/Azure/Azure-MachineLearning-DataScience/master/Misc/TDSP/AttachFileShare.ps1" -outfile "AttachFileShare.ps1"
     .\AttachFileShare.ps1
     ```
     
   - Sur un ordinateur Linux, exécutez le script à partir de l’interpréteur de commandes Linux :
     
     ```shell
     wget "https://raw.githubusercontent.com/Azure/Azure-MachineLearning-DataScience/master/Misc/TDSP/AttachFileShare.sh"
     bash AttachFileShare.sh
     ```
   
1. Appuyez sur Entrée ou entrez *Y* pour continuer si vous avez enregistré un fichier d’informations sur le stockage de fichiers Azure à l’étape précédente. Entrez le chemin et le nom complets du fichier que vous avez créé. 
   
   Si vous n’avez pas de fichier d’informations sur le stockage de fichiers Azure, entrez *n*, puis suivez les instructions pour entrer les informations sur votre abonnement, votre compte de stockage Azure et votre stockage de fichiers Azure.
   
1. Entrez le nom d’un lecteur local ou TDSP pour y monter le partage de fichiers. L’écran affiche la liste des noms de lecteurs existants. Indiquez un nom de lecteur qui n’existe pas déjà.
   
1. Vérifiez que le nouveau lecteur et le stockage sont bien montés sur l’ordinateur.

## <a name="next-steps"></a>Étapes suivantes

Les liens ci-dessous donnent accès à des descriptions détaillées des autres rôles et tâches définis par le processus TDSP (Team Data Science Process) :

- [Tâches du responsable de groupe pour une équipe de science des données](group-manager-tasks.md)
- [Tâches du coordinateur de projet pour une équipe de science des données](project-lead-tasks.md)
- [Tâches d’un contributeur individuel de projet pour une équipe de science des données](project-ic-tasks.md)