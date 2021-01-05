---
title: Développement agile de projets de science des données - Team Data Science Process
description: Réalisez, au sein d’une équipe, un projet de science des données d’une manière systématique, collaborative et avec gestion de versions, à l’aide du processus Team Data Science Process.
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: 113c11b749b42bbaf9e5e40cdf63623288997a92
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89440284"
---
# <a name="agile-development-of-data-science-projects"></a>Développement Agile de projets de science des données

Ce document explique comment les développeurs peuvent réaliser, au sein d’une équipe, un projet de science des données d’une manière systématique, collaborative et avec gestion de versions, à l’aide du [processus TDSP](overview.md) (Team Data Science Process). TDSP est un cadre élaboré par Microsoft qui propose une succession structurée d’activités visant à exécuter avec efficacité des solutions d’analytique prédictive basées sur le cloud. Pour obtenir une description des rôles et des tâches qui leur incombent dans le cadre du processus TDSP, consultez [Rôles et tâches du processus TDSP](roles-tasks.md). 

Cet article fournit des instructions pour les procédures suivantes : 

- Utiliser la *planification sprint* pour les éléments du projet.
- Ajouter des *éléments de travail* à une planification sprint.
- Créer et utiliser un *modèle d’élément de travail dérivé d’agile* qui s’aligne spécifiquement sur les phases du cycle de vie du processus TDSP.

Les instructions qui suivent décrivent les étapes nécessaires pour configurer un environnement d’équipe TDSP à l’aide d’Azure Boards et Azure Repos dans Azure DevOps. Les instructions recourent à Azure DevOps, car c’est la suite utilisée au sein de Microsoft pour l’implémentation du processus TDSP. Si votre groupe utilise une plateforme d’hébergement de code différente, les tâches du responsable d’équipe ne changent généralement pas. Par contre, la façon d’effectuer les tâches diffère. Par exemple, la liaison d’un élément de travail à une branche GIT peut ne pas être la même avec GitHub, comme c’est le cas avec Azure Repos.

La figure suivante montre le workflow classique de planification sprint, de code et de contrôle de code source pour un projet de science des données :

![TDSP (Team Data Science Process)](./media/agile-development/1-project-execute.png)

##  <a name="work-item-types"></a><a name='Terminology-1'></a>Types d’éléments de travail

Dans le framework de planification sprint TDSP, il existe quatre types d’*éléments de travail* fréquemment utilisés : *Fonctionnalités*, *Récits utilisateur*, *Tâches* et *Bogues*. Le backlog de tous les éléments de travail se trouve au niveau du projet, et non au niveau du dépôt Git. 

Voici les définitions des types d’éléments de travail :

- **Caractéristique** : une fonctionnalité correspond à un engagement de projet. Les différents engagements avec un client constituent des fonctionnalités différentes, et il est préférable d’envisager les différentes phases d’un projet en tant que fonctionnalités différentes. Si vous choisissez un format tel que *\<ClientName>-\<EngagementName>* pour nommer vos fonctionnalités, vous pourrez facilement connaître le contexte du projet et de l’engagement grâce au nom qu’il porte.
  
- **Récit utilisateur** : les récits utilisateur sont des éléments de travail nécessaires pour effectuer une fonctionnalité de bout en bout. Voici quelques exemples de récits utilisateur :
  - Obtenir des données 
  - Explorer des données 
  - Générer des fonctionnalités
  - Générer des modèles
  - Rendre les modèles opérationnels 
  - Reformer des modèles
  
- **Tâche** : les tâches sont des éléments de travail assignables qui doivent être réalisées pour effectuer un récit utilisateur spécifique. Par exemple, les tâches du récit utilisateur *Obtention des données* pourraient être les suivantes :
  - Récupérer les informations d’identification SQL Server
  - Charger des données dans Azure Synapse Analytics
  
- **Bogue** : les bogues sont des problèmes dans le code ou les documents existants qui doivent être corrigés pour effectuer une tâche. Si des bogues sont provoqués par des éléments de travail manquants, ils peuvent devenir des récits utilisateur ou des tâches. 

Les scientifiques des données peuvent se sentir plus à l’aise avec un modèle agile qui remplace les fonctionnalités, récits utilisateur et tâches par des phases et sous-phases du cycle de vie du processus TDSP. Pour créer un modèle dérivé d’agile qui s’aligne spécifiquement sur les phases du cycle de vie du processus TDSP, consultez [Utiliser un modèle de travail TDSP agile](#set-up-agile-dsp-6).

> [!NOTE]
> Le processus TDSP emprunte les concepts de fonctionnalités, de récits utilisateur, de tâches et de bogues à la gestion de code logiciel (SCM). Les définitions des concepts TDSP peuvent varier légèrement de celles données par la gestion de code logiciel.

## <a name="plan-sprints"></a><a name='SprintPlanning-2'></a>Planifier des sprints

De nombreux scientifiques des données sont engagés dans plusieurs projets à la fois, qui peuvent durer des mois et progresser à des rythmes différents. La planification sprint est utile pour la définition des priorités d’un projet, ainsi que pour la planification et l’allocation des ressources. Dans Azure Boards, vous pouvez facilement créer et gérer des éléments de travail pour vos projets, en effectuer le suivi ainsi qu’effectuer une planification sprint en vue de garantir le bon déroulement des projets.

Pour plus d’informations sur la planification des sprints, consultez [Scrum sprints](https://en.wikipedia.org/wiki/Scrum_(software_development)#Sprint). 

Pour plus d’informations sur la planification de sprints dans Azure Boards, consultez [Attribuer des éléments de backlog à un sprint](/azure/devops/boards/sprints/assign-work-sprint). 

## <a name="add-a-feature-to-the-backlog"></a><a name='AddFeature-3'></a>Ajouter une fonctionnalité au backlog 

Une fois votre projet et le dépôt de code du projet créés, vous pouvez ajouter une fonctionnalité au backlog pour représenter le travail de votre projet.

1. Dans la page de votre projet, sélectionnez **Tableaux** > **Backlogs** dans le volet de navigation de gauche. 
   
1. Sous l’onglet **Backlog**, si le type d’élément de travail dans la barre supérieure est **Récits**, ouvrez la liste déroulante et sélectionnez **Fonctionnalités**. Sélectionnez ensuite **Nouvel élément de travail**.
   
   ![Sélectionner un nouvel élément de travail](./media/agile-development/2-sprint-team-overview.png)
   
1. Entrez un titre pour la fonctionnalité, généralement le nom de votre projet, puis sélectionnez **Ajouter en haut**. 
   
   ![Entrer un titre et sélectionner Ajouter en haut](./media/agile-development/3-sprint-team-add-work.png)
   
1. Dans la liste **Backlog**, sélectionnez et ouvrez la nouvelle fonctionnalité. Renseignez la description, attribuez un membre de l’équipe et définissez les paramètres de planification. 
   
   Vous pouvez également lier la fonctionnalité au dépôt du code Azure Repos du projet en sélectionnant **Ajouter un lien** dans la section **Développement**. 
   
   Après avoir modifié la fonctionnalité, sélectionnez **Enregistrer et fermer**.
   
   ![Modifier la fonctionnalité et sélectionner Enregistrer et fermer](./media/agile-development/3a-add-link-repo.png)

## <a name="add-a-user-story-to-the-feature"></a><a name='AddStoryunderfeature-4'></a>Ajouter un récit utilisateur à la fonctionnalité 

Sous la fonctionnalité, vous pouvez ajouter des récits utilisateur pour décrire les principales étapes nécessaires pour terminer le projet. 

Pour ajouter un nouveau récit utilisateur à une fonctionnalité :

1. Sous l’onglet **Backlog**, sélectionnez le symbole **+** à gauche de la fonctionnalité. 
   
   ![Ajouter un nouveau récit utilisateur sous la fonctionnalité](./media/agile-development/4-sprint-add-story.png)
   
1. Donnez un titre au récit utilisateur et modifiez les détails tels que l’affectation, l’état, la description, les commentaires, la planification et la priorité. 
   
   Vous pouvez également lier le récit utilisateur à une branche du dépôt du code Azure Repos du projet en sélectionnant **Ajouter un lien** dans la section **Développement**. Sélectionnez le dépôt et la branche auxquels vous souhaitez lier l’élément de travail, puis sélectionnez **OK**.
   
   ![Ajouter un lien](./media/agile-development/5-sprint-edit-story.png)
   
1. Quand vous avez terminé de modifier le récit utilisateur, sélectionnez **Enregistrer et fermer**. 

## <a name="add-a-task-to-a-user-story"></a><a name='AddTaskunderstory-5'></a>Ajouter une tâche à un récit utilisateur 

Les tâches sont des étapes détaillées qui sont nécessaires à la réalisation d’un récit utilisateur. Une fois que toutes les tâches d’un récit utilisateur sont terminées, le récit utilisateur doit également être terminé. 

Pour ajouter une tâche à un récit utilisateur, sélectionnez le symbole **+** en regard de l’élément de récit utilisateur, puis sélectionnez **Tâche**. Renseignez le titre et les autres informations dans la tâche.

![Ajouter une tâche à un récit utilisateur](./media/agile-development/7-sprint-add-task.png)

Après avoir créé les fonctionnalités, les récits utilisateur et les tâches, vous pouvez les afficher dans les vues **Backlogs** ou **Tableaux** pour suivre leur état.

![Vue Backlogs](./media/agile-development/8-sprint-backlog-view.png)

![Vue Tableaux](./media/agile-development/8a-sprint-board-view.png)

## <a name="use-an-agile-tdsp-work-template"></a><a name='set-up-agile-dsp-6'></a>Utiliser un modèle de travail TDSP agile

Les scientifiques des données peuvent se sentir plus à l’aise avec un modèle agile qui remplace les fonctionnalités, récits utilisateur et tâches par des phases et sous-phases du cycle de vie du processus TDSP. Dans Azure Boards, vous pouvez créer un modèle dérivé d’agile qui utilise les phases du cycle de vie du processus TDSP pour créer et suivre des éléments de travail. Les étapes suivantes décrivent la configuration d’un modèle de processus agile spécifique à la science des données et la création d’éléments de travail de science des données basés sur le modèle.

### <a name="set-up-an-agile-data-science-process-template"></a>Configurer un modèle de processus de science des données agile

1. Dans la page principale de votre organisation Azure DevOps, sélectionnez **Paramètres de l’organisation** dans le volet de navigation gauche. 
   
1. Dans le volet de navigation gauche **Paramètres de l’organisation**, sous **Tableaux**, sélectionnez **Processus**. 
   
1. Dans le volet **Tous les processus**, sélectionnez **...** en regard d’**Agile**, puis sélectionnez **Créer un processus hérité**.
   
   ![Créer un processus hérité à partir d’Agile](./media/agile-development/10-settings.png) 
   
1. Dans la boîte de dialogue **Créer un processus hérité à partir d’Agile**, entrez le nom *AgileDataScienceProcess*, puis sélectionnez **Créer un processus**.
   
   ![Créer un processus AgileDataScienceProcess](./media/agile-development/11-agileds.png)
   
1. Dans **Tous les processus**, sélectionnez le nouveau processus **AgileDataScienceProcess**. 
   
1. Sous l’onglet **Types d’éléments de travail**, désactivez **Épopée**, **Fonctionnalité**, **Récit utilisateur** et **Tâche** en sélectionnant **...** en regard de chaque élément, puis en sélectionnant **Désactiver**. 
   
   ![Désactiver les types d’éléments de travail](./media/agile-development/12-disable.png)
   
1. Dans **Tous les processus**, sélectionnez l’onglet **Niveaux de backlog**. Sous **Backlogs de portefeuille**, sélectionnez **...** en regard d’**Épopée (désactivé)** , puis sélectionnez **Modifier/Renommer**. 
   
1. Dans la boîte de dialogue **Modifier un niveau de backlog** :
   1. Sous **Nom**, remplacez **Épopée** par *Projets TDSP*. 
   1. Sous **Types d’éléments de travail sur ce niveau de backlog**, sélectionnez **Nouveau type d’élément de travail**, entrez *Projet TDSP*, puis sélectionnez **Ajouter**. 
   1. Sous **Type d’élément de travail par défaut**, ouvrez la liste déroulante et sélectionnez **Projet TDSP**. 
   1. Sélectionnez **Enregistrer**.
   
   ![Définir le niveau de backlog du portefeuille](./media/agile-development/13-rename.png)  
   
1. Suivez les mêmes étapes pour renommer **Fonctionnalités** en *Phases TDSP*, puis ajoutez les nouveaux types d’éléments de travail suivants :
   
   - *Présentation de l’entreprise*
   - *Acquisition de données*
   - *Modélisation*
   - *Déploiement*
   
1. Sous **Backlog des exigences**, renommez **Récits** en *Sous-phases TDSP*, ajoutez le nouveau type d’élément de travail *Sous-phase TDSP*, puis définissez le type d’élément de travail par défaut sur **Sous-phase TDSP**.
   
1. Sous **Backlog des itérations**, ajoutez un nouveau type d’élément de travail *Tâche TDSP*, puis définissez-le comme type d’élément de travail par défaut. 
   
Une fois que vous avez effectué ces étapes, les niveaux de backlog doivent ressembler à ceci :
   
 ![Niveaux de backlog du modèle TDSP](./media/agile-development/14-template.png)  

### <a name="create-agile-data-science-process-work-items"></a>Créer des éléments de travail de science des données agile

Vous pouvez utiliser le modèle de processus de science des données pour créer des projets TDSP et suivre les éléments de travail qui correspondent aux phases du cycle de vie du processus TDSP.

1. Dans la page principale de votre organisation Azure DevOps, sélectionnez **Nouveau projet**. 
   
1. Dans la boîte de dialogue **Créer un projet**, donnez un nom à votre projet, puis sélectionnez **Avancé**. 
   
1. Sous **Processus d’élément de travail**, ouvrez la liste déroulante, et sélectionnez **AgileDataScienceProcess**, puis sélectionnez **Créer**.
   
   ![Créer un projet TDSP](./media/agile-development/15-newproject.png)
   
1. Dans le projet créé, sélectionnez **Tableaux** > **Backlogs** dans le volet de navigation de gauche.
   
1. Pour rendre les projets TDSP visibles, sélectionnez l’icône **Configurer les paramètres de l’équipe**. Dans l’écran **Paramètres**, cochez la case **Projets TDSP**, puis sélectionnez **Enregistrer et fermer**.
   
   ![Cocher la case Projets TDSP](./media/agile-development/16-enabledsprojects1.png)
   
1. Pour créer un projet TDSP spécifique à la science des données, sélectionnez **Projets TDSP** dans la barre supérieure, puis sélectionnez **Nouvel élément de travail**. 
   
1. Dans la fenêtre contextuelle, attribuez un nom à l’élément de travail du projet TDSP, puis sélectionnez **Ajouter en haut**.
   
   ![Créer un élément de travail de projet de science des données](./media/agile-development/17-dsworkitems0.png)
   
1. Pour ajouter un élément de travail sous le projet TDSP, sélectionnez le symbole **+** en regard du projet, puis sélectionnez le type d’élément de travail à créer. 
   
   ![Sélectionner un type d’élément de travail de science des données](./media/agile-development/17-dsworkitems1.png)
   
1. Renseignez les détails dans le nouvel élément de travail, puis sélectionnez **Enregistrer et fermer**.
   
1. Sélectionnez de nouveau les symboles **+** en regard des éléments de travail pour ajouter de nouvelles phases, sous-phases et tâches TDSP. 
   
Les éléments de travail du projet de science des données doivent apparaître dans la vue in **Backlogs** :

![18](./media/agile-development/18-workitems1.png)


## <a name="next-steps"></a>Étapes suivantes

[Développement collaboratif avec Git](collaborative-coding-with-git.md) explique comment développer le code de projets de science des données en utilisant Git comme framework de développement de code partagé, et comment lier ces activités de code au travail planifié avec le processus agile.

[Exemples de procédures pas à pas](walkthroughs.md) contient une liste de procédures pas à pas pour des scénarios spécifiques, avec des liens et des descriptions miniatures. Les scénarios liés montrent également comment combiner des outils et des services cloud et locaux dans des workflows ou des pipelines pour créer des applications intelligentes.
  
Ressources supplémentaires sur les processus Agile :

- [Processus Agile](/azure/devops/boards/work-items/guidance/agile-process)
  
- [Types d’éléments de travail et workflow du processus Agile](/azure/devops/boards/work-items/guidance/agile-process-workflow)

