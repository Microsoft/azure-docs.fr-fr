---
title: Rôles et tâches du responsable de l’équipe Team Data Science Process
description: Présentation des composants clés, rôles du personnel et tâches associées pour un groupe de science des données.
author: marktab
manager: marktab
editor: marktab
services: machine-learning
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: 4bcbbd9378b6ea861c926eed16d2ceed8131b913
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96000961"
---
# <a name="team-data-science-process-roles-and-tasks"></a>Rôles et tâches du responsable de l’équipe Team Data Science Process

Team Data Science Process (TDSP) est une infrastructure développée par Microsoft qui fournit une méthodologie structurée pour créer efficacement des solutions d’analyse prédictive et des applications intelligentes. Cet article présente les rôles du personnel clé et les tâches qui incombent à une équipe de science des données dans le cadre de ce processus.

Cet article d’introduction contient des liens vers des tutoriels expliquant comment configurer l’environnement TDSP. Ces tutoriels fournissent des instructions détaillées sur l’utilisation d’Azure DevOps Projects, des dépôts Azure Repos et d’Azure Boards.  L’objectif sous-jacent est le passage du concept à la modélisation et au déploiement.

Les tutoriels recourent à Azure DevOps, car c’est la suite utilisée au sein de Microsoft pour l’implémentation du processus TDSP. Azure DevOps facilite la collaboration en intégrant la sécurité basée sur les rôles, la gestion et le suivi des éléments de travail ainsi que l’hébergement de code, le partage et le contrôle de code source. Les tutoriels utilisent également une [machine virtuelle Data Science Virtual Machine (DSVM)](https://aka.ms/dsvm) Azure comme bureau d’analytique, qui a plusieurs outils de science des données courants, préconfigurés et compatibles avec les logiciels Microsoft et services Azure. 

Vous pouvez utiliser les tutoriels pour implémenter le processus TDSP à l’aide d’autres outils et environnements d’hébergement de code, de planification agile et de développement, mais certaines fonctionnalités peuvent ne pas être disponibles.

## <a name="structure-of-data-science-groups-and-teams"></a>Structure des groupes et équipes de science des données

Les fonctions de science des données dans les entreprises sont souvent organisées selon la hiérarchie suivante :

- Groupe de science des données
  - Équipe(s) de science des données au sein du groupe

Ce type de structure comporte des responsables de groupe et des responsables d’équipe. En règle générale, un projet de science des données est effectué par une équipe de science des données. Les équipes de science des données ont des coordinateurs de projet pour les tâches de gestion et de gouvernance de projet ainsi que des ingénieurs et des scientifiques des données individuels pour la réalisation des parties de science des données et d’engineering données du projet. La configuration et la gouvernance initiales du projet sont effectuées par les responsables de groupe, d’équipe ou de projet.

## <a name="definition-and-tasks-for-the-four-tdsp-roles"></a>Définition et tâches pour les quatre rôles TDSP
En supposant que l’unité de science des données se compose d’équipes au sein d’un groupe, il existe quatre rôles distincts pour le personnel TDSP :

1. **Responsable de groupe** : gère l’ensemble de l’unité de science des données dans une entreprise. Une unité de science des données peut avoir plusieurs équipes, chacune travaillant sur plusieurs projets de science des données dans différents secteurs d’activité. Un responsable de groupe peut déléguer ses tâches à un remplaçant, mais les tâches associées au rôle ne changent pas.
   
2. **Responsable d’équipe** : gère une équipe dans l’unité de science des données d’une entreprise. Cette équipe est constituée de plusieurs scientifiques des données. Pour une petite unité de science des données, une même personne peut jouer le rôle de Responsable de groupe et de Responsable d’équipe.
   
3. **Coordinateur de projet** : gère les activités quotidiennes des différents scientifiques des données dans le cadre d’un projet de science des données spécifique.
   
4. **Contributeurs individuels de projet** : scientifiques des données, analystes d’entreprise, ingénieurs de données, architectes et autres personnes qui exécutent un projet de science des données.

> [!NOTE]
> Selon la structure et la taille de l’entreprise, une même personne peut avoir plusieurs rôles ou plusieurs personnes peuvent remplir un même rôle.

### <a name="tasks-to-be-completed-by-the-four-roles"></a>Tâches à effectuer par les quatre rôles

Le diagramme suivant montre les tâches de niveau supérieur pour chaque rôle Team Data Science Process. Ce schéma et la présentation suivante plus détaillée des tâches pour chaque rôle TDSP vous aident à choisir le tutoriel dont vous avez besoin en fonction de vos responsabilités.

![Vue d’ensemble des rôles et tâches](./media/roles-tasks/overview-tdsp-top-level.png)

## <a name="group-manager-tasks"></a>Tâches du responsable de groupe

Le Responsable de groupe ou un administrateur du système TDSP désigné effectue les tâches suivantes afin d’adopter TDSP :

- Crée une **organisation** Azure DevOps et un projet de groupe au sein de l’organisation. 
- Crée un **dépôt de modèle de projet** dans le projet de groupe Azure DevOps et l’alimente à l’aide du dépôt de modèles de projet développé par l’équipe TDSP de Microsoft. Le dépôt de modèles de projet TDSP de Microsoft fournit :
  - une **structure de répertoires normalisée** pour les données, le code et les documents ;
  - un ensemble de **modèles de documents normalisés** pour aider à créer un processus de science des données efficace.
- Crée un **dépôt d’utilitaire** et l’alimente à l’aide du dépôt d’utilitaires développé par l’équipe TDSP de Microsoft. Le dépôt d’utilitaires TDSP de Microsoft fournit un ensemble d’utilitaires pour améliorer l’efficacité du travail d’un scientifique des données. Le dépôt d’utilitaires de Microsoft comprend des utilitaires pour l’exploration interactive des données, l’analyse, la création de rapports ainsi que la modélisation et la création de rapports de base.
- Configure la **stratégie de contrôle de la sécurité** pour le compte de l’organisation.

Pour obtenir des instructions détaillées, consultez [Tâches du responsable de groupe pour une équipe de science des données](group-manager-tasks.md).

## <a name="team-lead-tasks"></a>Tâches du responsable d’équipe

Le Responsable d’équipe ou un administrateur de projet désigné effectue les tâches suivantes afin d’adopter TDSP :

- Crée un **projet** d’équipe dans l’organisation Azure DevOps du groupe.
- Crée le **dépôt de modèles du projet** dans le projet, puis l’alimente à l’aide du dépôt de modèle de projet de groupe configuré par le Responsable de groupe ou un délégué.
- Crée le **dépôt d’utilitaire d’équipe**, l’alimente à l’aide du dépôt d’utilitaire de groupe et y ajoute des utilitaires spécifiques à l’équipe.
- Crée éventuellement un [stockage de fichiers Azure](https://azure.microsoft.com/services/storage/files/) afin de stocker des ressources de données utiles pour l’équipe. D’autres membres de l’équipe peuvent monter ce magasin de fichiers partagés cloud sur leur bureau d’analyse.
- Monte éventuellement le stockage de fichiers Azure sur la machine **DSVM** de l’équipe et y ajoute des ressources de données d’équipe.
- Configure le **contrôle de sécurité** en ajoutant des membres d’équipe et en configurant leurs autorisations.

Pour obtenir des instructions détaillées, consultez [Tâches du responsable d’équipe pour une équipe de science des données](team-lead-tasks.md).


## <a name="project-lead-tasks"></a>Tâches du coordinateur de projet

Le coordinateur de projet effectue les tâches suivantes pour adopter TDSP :

- Crée un **dépôt de projet** dans le projet d’équipe et l’alimente à l’aide du dépôt du modèle de projet.
- Crée éventuellement un **stockage de fichiers Azure** afin de stocker les ressources de données du projet.
- Monte éventuellement le stockage de fichiers Azure sur la machine **DSVM** et y ajoute des ressources de données de projet.
- Configure le **contrôle de sécurité** en ajoutant des membres de projet et en configurant leurs autorisations.

Pour obtenir des instructions détaillées, consultez [Tâches du responsable de projet pour une équipe de science des données](project-lead-tasks.md).

## <a name="project-individual-contributor-tasks"></a>Tâches du contributeur individuel de projet

Le contributeur individuel de projet, généralement un scientifique des données, effectue les tâches suivantes à l’aide de TDSP :

- Clone le **dépôt de projet** configuré par le responsable de projet.
- Monte éventuellement le **stockage de fichiers Azure** partagé de l’équipe et du projet sur leur **Machine virtuelle de science des données** (DSVM).
- Exécute le projet.

Pour obtenir des instructions détaillées sur l’intégration dans un projet, consultez [Tâches d’un contributeur individuel de projet pour une équipe de science des données](project-ic-tasks.md).

## <a name="data-science-project-execution-workflow"></a>Workflow d’exécution du projet de science des données

En suivant les tutoriels appropriés, les scientifiques des données, les responsables de projet et les responsables d’équipe peuvent créer des éléments de travail afin d’effectuer le suivi de toutes les tâches et phases du projet du début à la fin. L’utilisation d’Azure Repos favorise la collaboration entre les scientifiques de données et garantit que les artefacts générés pendant l’exécution du projet sont contrôlés au niveau de la version et partagés par tous les membres du projet. Azure DevOps vous permet de lier vos éléments de travail Azure Boards à vos branches du dépôt Azure Repos et de suivre facilement ce qui a été fait pour un élément de travail.

La figure suivante présente le workflow TDSP pour l’exécution du projet :

![Workflow d’un projet de science des données standard](./media/roles-tasks/overview-project-execute.png)

Les étapes du workflow peuvent être regroupées en trois activités :

- Les coordinateurs de projet effectuent la planification des sprints
- Les scientifiques des données développent des artefacts sur les branches `git` pour traiter les éléments de travail
- Les coordinateurs de projet ou d’autres membres de l’équipe effectuent des révisions du code et fusionnent les branches de travail dans la branche principale

Pour obtenir des instructions détaillées sur le workflow d’exécution du projet, consultez [Développement Agile de projets de science des données](agile-development.md).

## <a name="tdsp-project-template-repository"></a>Dépôt de modèles de projet TDSP

Utilisez le [dépôt de modèles de projet](https://github.com/Azure/Azure-TDSP-ProjectTemplate) de l’équipe TDSP de Microsoft pour gérer la collaboration et l’exécution de projets efficaces. Il offre une structure de répertoires standardisée et des modèles de documents, que vous pouvez utiliser pour vos propres projets TDSP.

## <a name="next-steps"></a>Étapes suivantes

Voici des descriptions plus détaillées des rôles et des tâches définis par le processus TDSP (Team Data Science Process) :

- [Tâches du responsable de groupe pour une équipe de science des données](group-manager-tasks.md)
- [Tâches du responsable d’une équipe de science des données](team-lead-tasks.md)
- [Tâches du coordinateur de projet pour une équipe de science des données](project-lead-tasks.md)
- [Tâches d’un contributeur individuel de projet pour une équipe de science des données](project-ic-tasks.md)
