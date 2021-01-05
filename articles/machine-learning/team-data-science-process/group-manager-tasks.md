---
title: Tâches du responsable de groupe Team Data Science Process
description: Suivez cette présentation détaillée des tâches qu’un responsable de groupe accomplit sur un projet d’équipe de science des données.
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 11/17/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: 7ab6d6511d1e2cec82b321003c9d663249ddcf49
ms.sourcegitcommit: c2dd51aeaec24cd18f2e4e77d268de5bcc89e4a7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94740224"
---
# <a name="team-data-science-process-group-manager-tasks"></a>Tâches du responsable de groupe Team Data Science Process

Cet article décrit les tâches qu’effectue un *responsable de groupe* pour une organisation de science des données. Le responsable de groupe gère l’ensemble de la division Science des données d’une grande entreprise. Une division Science des données peut regrouper plusieurs équipes, chacune travaillant sur divers projets de science des données dans des domaines d’activité distincts. L’objectif du responsable de groupe est de créer un environnement de groupe collaboratif standard conforme au processus [TDSP](overview.md) (Team Data Science Process). Pour obtenir une description de l’ensemble des rôles et des tâches associées qui incombent aux membres d’une équipe de science des données dans le cadre du processus TDSP, consultez [Rôles et tâches du processus TDSP](roles-tasks.md).

Le schéma suivant présente les six principales tâches de configuration du responsable de groupe. Les responsables de groupe peuvent déléguer leurs tâches à des agents, mais les tâches associées au rôle ne changent pas.

![Tâches du responsable de groupe](./media/group-manager-tasks/tdsp-group-manager.png)

1. Créer une **organisation Azure DevOps** pour le groupe.
2. Créer le **projet GroupCommon** par défaut dans l’organisation Azure DevOps.
3. Créer le dépôt **GroupProjectTemplate** dans Azure Repos.
4. Créer le dépôt **GroupUtilities** dans Azure Repos.
5. Importer le contenu des dépôts **ProjectTemplate** et **Utilities** de l’équipe Microsoft TDSP dans les dépôts communs du groupe.
6. Configurer une **appartenance** et des **autorisations** pour permettre aux membres de l’équipe d’accéder au groupe.

Le tutoriel suivant présente en détail les différentes étapes. 

> [!NOTE] 
> Cet article s’appuie sur Azure DevOps pour mettre en place un environnement de groupe TDSP, car c’est ainsi que Microsoft implémente le processus TDSP. Si votre groupe utilise d’autres plateformes d’hébergement de code ou de développement, les tâches du responsable de groupe sont les mêmes, mais la façon de les accomplir peut être différente.

## <a name="create-an-organization-and-project-in-azure-devops"></a>Créer une organisation et un projet dans Azure DevOps

1. Accédez à [visualstudio.microsoft.com](https://visualstudio.microsoft.com), sélectionnez **Se connecter** en haut à droite, puis connectez-vous à votre compte Microsoft. 
   
   ![Connexion à votre compte Microsoft](./media/group-manager-tasks/signinvs.png)
   
   Si vous n’avez pas de compte Microsoft, sélectionnez **Inscrivez-vous maintenant**, créez un compte Microsoft, puis connectez-vous en utilisant ce compte. Si votre organisation dispose d’un abonnement Visual Studio, connectez-vous avec les informations d’identification de cet abonnement.
   
1. Une fois que vous êtes connecté, en haut à droite de la page Azure DevOps, sélectionnez **Créer l’organisation**.
   
   ![Créer une organisation](./media/group-manager-tasks/create-organization.png)
   
1. Si vous êtes invité à accepter les conditions d’utilisation du service, la déclaration de confidentialité et le code de conduite, sélectionnez **Continuer**.
   
1. Dans la boîte de dialogue d’inscription, attribuez un nom à votre organisation Azure DevOps et acceptez l’attribution de la région hôte, ou déroulez la liste et sélectionnez une autre région. Sélectionnez **Continuer**. 

1. Sous **Pour commencer, créez un projet**, entrez *GroupCommon*, puis sélectionnez **Créer un projet**. 
   
   ![Créer un projet](./media/group-manager-tasks/create-project.png)

La page **Résumé** du projet **GroupCommon** s’ouvre. L’URL de la page est *https:\//\<servername>/\<organization-name>/GroupCommon*.

![Page Résumé du projet](./media/group-manager-tasks/project-summary.png)

## <a name="set-up-the-group-common-repositories"></a>Créer les dépôts communs du groupe

Azure Repos permet à votre groupe d’héberger les types de dépôt suivants :

- **Dépôts communs du groupe** : dépôts à usage général que plusieurs équipes d’une même division Science des données peuvent adopter pour divers projets de science des données. 
- **Dépôts d’équipe** :  dépôts pour des équipes spécifiques au sein d’une division Science des données. Ces dépôts varient en fonction des besoins d’une équipe et peuvent servir à divers projets au sein d’une même équipe, mais ils ne sont pas suffisamment généraux pour servir à plusieurs équipes au sein d’une division Science des données.
- **Dépôts de projet** : dépôts pour des projets spécifiques. Ces dépôts peuvent ne pas être suffisamment généraux pour servir à plusieurs projets au sein d’une équipe ou à d’autres équipes au sein d’une division Science des données.

Pour créer les dépôts communs de groupe dans votre projet, vous devez : 
- Renommer le dépôt par défaut **GroupCommon** **GroupProjectTemplate**
- Créer un dépôt **GroupUtilities**

### <a name="rename-the-default-project-repository-to-groupprojecttemplate"></a>Renommer le dépôt de projet par défaut GroupProjectTemplate

Pour renommer le dépôt de projet par défaut **GroupCommon** **GroupProjectTemplate** :

1. Dans la page **Résumé** du projet **GroupCommon**, sélectionnez **Dépôts**. Cette action vous dirige vers le dépôt par défaut **GroupCommon** du projet GroupCommon, qui est actuellement vide.
   
1. En haut de la page, cliquez sur la flèche en regard de **GroupCommon**, puis sélectionnez **Gérer les dépôts** dans le menu déroulant.
   
   ![Gérer les dépôts](./media/group-manager-tasks/rename-groupcommon-repo-3.png)
   
1. Dans la page **Paramètres du projet**, sélectionnez **...** en regard de **GroupCommon**, puis **Renommer le dépôt**. 
   
   ![Sélectionner ... puis Renommer le dépôt](./media/group-manager-tasks/rename-groupcommon-repo-4.png)
   
1. Dans la fenêtre contextuelle **Renommer le dépôt GroupCommon**, entrez *GroupProjectTemplate*, puis sélectionnez **Renommer**. 
   
   ![Renommer le dépôt](./media/group-manager-tasks/rename-groupcommon-repo-6.png)

### <a name="create-the-grouputilities-repository"></a>Créer le dépôt GroupUtilities

Pour créer le dépôt **GroupUtilities** :

1. Dans la page **Résumé** du projet **GroupCommon**, sélectionnez **Dépôts**. 
   
1. En haut de la page, cliquez sur la flèche en regard de **GroupProjectTemplate**, puis sélectionnez **Nouveau dépôt** dans le menu déroulant.
   
   ![Sélectionner Nouveau dépôt](./media/group-manager-tasks/create-grouputilities-repo-1.png)
   
1. Dans la boîte de dialogue **Créer un dépôt**, sélectionnez **Git** en guise de **Type**, entrez *GroupUtilities* pour **Nom du dépôt**, puis sélectionnez **Créer**.
   
   ![Créer le dépôt GroupUtilities](./media/group-manager-tasks/create-grouputilities-repo-2.png)
   
1. Dans la page **Paramètres du projet**, sélectionnez **Dépôts** en dessous de **Dépôts** dans le volet de navigation gauche pour afficher les deux dépôts du groupe : **GroupProjectTemplate** et **GroupUtilities**.
   
   ![Les deux dépôts du groupe](./media/group-manager-tasks/two-repositories.png)

## <a name="import-the-microsoft-tdsp-team-repositories"></a>Importer les dépôts de l’équipe Microsoft TDSP

Dans cette partie du tutoriel, vous allez importer le contenu des dépôts **ProjectTemplate** et **Utilities** gérés par l’équipe Microsoft TDSP dans vos dépôts **GroupProjectTemplate** et **GroupUtilities**. 

Pour importer les dépôts de l’équipe TDSP :

1. Dans la page d’accueil du projet **GroupCommon**, sélectionnez **Dépôts** dans le volet de navigation gauche. Le dépôt par défaut **GroupProjectTemplate** s’ouvre. 
   
1. Dans la page **GroupProjectTemplate est vide**, sélectionnez **Importer**. 
   
   ![Sélectionner Importer](./media/group-manager-tasks/import-repo.png)
   
1. Dans la boîte de dialogue **Importer un dépôt Git**, sélectionnez **Git** comme **Type de source**, puis entrez *https:\//github.com/Azure/Azure-TDSP-ProjectTemplate.git* pour l’**URL clone**. Sélectionnez ensuite **Importer**. Le contenu du dépôt ProjectTemplate de l’équipe Microsoft TDSP est importé dans votre dépôt GroupProjectTemplate. 
   
   ![Importer le dépôt de l’équipe Microsoft TDSP](./media/group-manager-tasks/import-repo-2.png)
   
1. En haut de la page **Dépôts**, dans la liste déroulante, sélectionnez le dépôt **GroupUtilities**.
   
Chacun des deux dépôts de votre groupe contient à présent tous les fichiers du dépôt correspondant de l’équipe Microsoft TDSP, à l’exception de ceux du répertoire *.git*. 

## <a name="customize-the-contents-of-the-group-repositories"></a>Personnaliser le contenu des dépôts du groupe

Si vous voulez personnaliser le contenu des dépôts de votre groupe en fonction de ses besoins spécifiques, vous pouvez le faire dès maintenant. Vous pouvez modifier les fichiers, changer la structure des répertoires ou ajouter des fichiers que votre groupe a développés ou qui sont utiles à votre groupe.

### <a name="make-changes-in-azure-repos"></a>Apporter des modifications dans Azure Repos

Pour personnaliser le contenu d’un dépôt :

1. Dans la page **Résumé** du projet **GroupCommon**, sélectionnez **Dépôts**. 
   
1. En haut de la page, sélectionnez le dépôt que vous voulez personnaliser.

1. Dans la structure de répertoires du dépôt, accédez au dossier ou fichier à modifier. 
   
   - Pour créer des dossiers ou des fichiers, sélectionnez la flèche en regard de **Nouveau**. 
     
     ![Créer un fichier](./media/group-manager-tasks/new-file.png)
     
   - Pour charger des fichiers, sélectionnez **Charger le ou les fichiers**. 
     
     ![Charger des fichiers](./media/group-manager-tasks/upload-files.png)
     
   - Pour modifier des fichiers existants, accédez au fichier, puis sélectionnez **Modifier**. 
     
     ![Modifier un fichier](./media/group-manager-tasks/edit-file.png)
     
1. Après avoir ajouté ou modifié des fichiers, sélectionnez **Validation**.
   
   ![Valider les modifications](./media/group-manager-tasks/commit.png)

### <a name="make-changes-using-your-local-machine-or-dsvm"></a>Apporter des modifications à partir de votre ordinateur local ou machine virtuelle DSVM

Si vous souhaitez apporter des modifications à partir de votre ordinateur local ou machine virtuelle DSVM et envoyer (push) ces modifications aux dépôts du groupe, veillez à respecter les prérequis pour utiliser Git et des machines virtuelles DSVM :

- Un abonnement Azure, si vous voulez créer un machine virtuelle DSVM.
- Git installé sur votre ordinateur. Si vous utilisez une machine virtuelle DSVM, Git est préinstallé. Dans le cas contraire, consultez l’[annexe Plateformes et outils](platforms-and-tools.md#appendix).
- Si vous souhaitez utiliser une machine virtuelle DSVM, l’instance DSVM Windows ou Linux doit être créée et configurée dans Azure. Pour obtenir des informations complémentaires et des instructions, consultez la [documentation sur Data Science Virtual Machine](../data-science-virtual-machine/index.yml).
- Pour une machine virtuelle DSVM Windows, [Git Credential Manager (GCM)](https://github.com/Microsoft/Git-Credential-Manager-for-Windows) installé sur votre ordinateur. Dans le fichier *README.md*, accédez à la section **Télécharger et installer**, puis sélectionnez le **programme d’installation le plus récent**. Téléchargez le programme d’installation *.exe* et exécutez-le. 
- Pour une machine virtuelle DSVM Linux, une clé publique SSH créée dans votre instance DSVM et ajoutée à Azure DevOps. Pour obtenir des informations complémentaires et des instructions, consultez la section **Créer une clé publique SSH** dans l’[annexe Plateformes et outils](platforms-and-tools.md#appendix). 

Pour commencer, copiez ou *clonez* le dépôt sur votre ordinateur local. 
   
1. Dans la page **Résumé** du projet **GroupCommon**, sélectionnez **Dépôts** puis, en haut de la page, sélectionnez le dépôt que vous voulez cloner.
   
1. Dans la page du dépôt, sélectionnez **Cloner** en haut à droite.
   
1. Dans la boîte de dialogue **Cloner le dépôt**, sélectionnez **HTTPS** pour une connexion HTTP ou **SSH** pour une connexion SSH, puis copiez l’URL clone figurant sous **Ligne de commande** dans votre Presse-papiers.
   
   ![Cloner le dépôt](./media/group-manager-tasks/clone.png)
   
1. Sur votre ordinateur local, créez les répertoires suivants :
   
   - Pour Windows : **C:\GitRepos\GroupCommon**
   - Pour Linux, **$/GitRepos/GroupCommon** dans votre répertoire de base 
   
1. Passez au répertoire que vous avez créé.
   
1. Dans Git bash, exécutez la commande `git clone <clone URL>.`
   
   Par exemple, chacune des commandes suivantes clone le dépôt **GroupUtilities** dans le répertoire *GroupCommon* de votre ordinateur local. 
   
   **Connexion HTTPS :**
   
   ```bash
   git clone https://DataScienceUnit@dev.azure.com/DataScienceUnit/GroupCommon/_git/GroupUtilities
   ```
   
   **Connexion SSH :**
   
   ```bash
   git clone git@ssh.dev.azure.com:v3/DataScienceUnit/GroupCommon/GroupUtilities
   ```

Après avoir effectué les modifications voulues dans le clone local de votre répertoire, vous pouvez envoyer (push) les modifications aux répertoires communs partagés du groupe. 

Exécutez les commandes Git Bash suivantes à partir de votre répertoire local **GroupProjectTemplate** ou **GroupUtilities**.

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

## <a name="add-group-members-and-configure-permissions"></a>Ajouter des membres au groupe et configurer des autorisations

Pour ajouter des membres au groupe :

1. Dans Azure DevOps, à partir de la page d’accueil du projet **GroupCommon**, sélectionnez **Paramètres du projet** dans le volet de navigation gauche. 
   
1. Dans le volet de navigation gauche **Paramètres du projet**, sélectionnez **Équipes** puis, dans la page **Équipes**, sélectionnez l’**équipe GroupCommon**. 
   
   ![Configurer des équipes](./media/group-manager-tasks/teams.png)
   
1. Dans la page **Profil de l’équipe**, sélectionnez **Ajouter**.
   
   ![Ajouter à l’équipe GroupCommon](./media/group-manager-tasks/add-to-team.png)
   
1. Dans la boîte de dialogue **Ajouter des utilisateurs et des groupes**, recherchez les membres à ajouter au groupe, sélectionnez-les, puis sélectionnez **Enregistrer les modifications**. 
   
   ![Ajoutez des utilisateurs et des groupes](./media/group-manager-tasks/add-users.png)
   

Pour configurer des autorisations pour les membres :

1. Dans le volet de navigation gauche **Paramètres du projet**, sélectionnez **Autorisations**. 
   
1. Dans la page **Autorisations**, sélectionnez le groupe auquel vous voulez ajouter les membres. 
   
1. Dans la page du groupe, sélectionnez **Membres**, puis **Ajouter**. 
   
1. Dans la fenêtre contextuelle **Inviter des membres**, recherchez les membres à ajouter au groupe, sélectionnez-les, puis sélectionnez **Enregistrer**. 
   
   ![Accorder des autorisations aux membres](./media/group-manager-tasks/grant-permissions.png)

## <a name="next-steps"></a>Étapes suivantes

Les liens ci-dessous donnent accès à des descriptions détaillées des autres rôles et tâches dans le processus TDSP (Team Data Science Process) :

- [Tâches du responsable d’une équipe de science des données](team-lead-tasks.md)
- [Tâches du coordinateur de projet pour une équipe de science des données](project-lead-tasks.md)
- [Tâches d’un contributeur individuel de projet pour une équipe de science des données](project-ic-tasks.md)
