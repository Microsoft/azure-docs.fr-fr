---
title: Vue d’ensemble du contrôle d’accès à l’espace de travail Synapse
description: Cet article décrit les mécanismes utilisés pour contrôler l’accès à un espace de travail Synapse ainsi que les ressources et les artefacts de code qu’il contient.
services: synapse-analytics
author: billgib
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: security
ms.date: 12/03/2020
ms.author: billgib
ms.reviewer: jrasnick
ms.openlocfilehash: 256fec97819cde0f6f62d59b34416c92e1edfd20
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97109604"
---
# <a name="synapse-access-control"></a>Contrôle d’accès Synapse 

Cet article fournit une vue d’ensemble des mécanismes disponibles pour contrôler l’accès aux données et aux ressources de calcul de Synapse.  

## <a name="overview"></a>Vue d’ensemble

Synapse fournit un système de contrôle d’accès complet et affiné, qui intègre les éléments suivants : 
- **Rôles Azure** pour la gestion des ressources et l’accès aux données dans le stockage 
- **Rôles Synapse** pour la gestion de l’accès direct au code et à l’exécution 
- **Rôles SQL** pour l’accès du plan de données aux données dans les pools SQL 
- **Autorisations Git** pour le contrôle du code source, entre autres la prise en charge de l’intégration et du déploiement continus  

Les rôles Synapse fournissent des ensembles d’autorisations qui peuvent être appliqués à différentes étendues. Grâce à cette granularité, il est facile d’accorder aux administrateurs, développeurs, opérateurs et personnel de sécurité un accès approprié aux données et ressources de calcul.

Le contrôle d’accès peut être simplifié en utilisant des groupes de sécurité correspondant aux rôles de travail des personnes.  Il vous suffit d’ajouter ou de supprimer des utilisateurs dans les groupes de sécurité appropriés pour gérer l’accès.

## <a name="access-control-elements"></a>Éléments du contrôle d’accès

### <a name="creating-and-managing-synapse-compute-resources"></a>Création et gestion des ressources de calcul Synapse

Les rôles Azure sont utilisés pour contrôler la gestion des éléments suivants : 
- Pools SQL dédiés 
- Pools Apache Spark 
- Runtimes d’intégration 

Pour *créer* ces ressources, vous devez être propriétaire ou contributeur Azure sur le groupe de ressources.  Pour les *gérer* une fois créées, vous devez être propriétaire ou contributeur Azure sur le groupe de ressources ou les ressources individuelles. 

### <a name="developing-and-executing-code-in-synapse"></a>Développement et exécution du code dans Synapse 

Synapse prend en charge deux modèles de développement.

- **Développement direct Synapse**.  Vous développez et déboguez du code dans Synapse Studio, puis vous le **publiez** pour l’enregistrer et l’exécuter.  Le service Synapse constitue la seule source fiable de modification et d’exécution du code.  Tout travail non publié est perdu quand vous fermez Synapse Studio.  
- **Développement compatible Git**. Vous développez et déboguez du code dans Synapse Studio, puis vous **validez** les modifications apportées à une branche de travail d’un dépôt Git. Le travail effectué sur une ou plusieurs branches est intégré à une branche de collaboration, à partir de laquelle vous **publiez** sur le service.  Le dépôt Git est la seule source fiable pour l’édition de code, et le service celle pour l’exécution. Les modifications doivent être validées dans le dépôt Git, ou publiées dans le service avant de fermer Synapse Studio. [Apprenez-en davantage](https://go.microsoft.com/fwlink/?linkid=2150100) sur l’utilisation de Synapse Analytics avec Git.

Dans les deux modèles de développement, tout utilisateur ayant accès à Synapse Studio peut créer des artefacts de code.  En revanche, vous avez besoin d’autorisations supplémentaires pour publier des artefacts dans le service et lire des artefacts publiés, également pour valider des modifications dans Git, exécuter du code et accéder à des données liées, protégées par des informations d’identification.

### <a name="synapse-roles"></a>Rôles Synapse

Les rôles Synapse servent à contrôler l’accès au service Synapse qui vous permet d’effectuer les opérations suivantes : 
- Lister les artefacts de code publiés 
- Publier les artefacts de code, les services liés et les définitions d’informations d’identification
- Exécuter du code ou des pipelines qui utilisent des ressources de calcul Synapse
- Exécuter du code ou des pipelines qui accèdent à des données liées protégées par des informations d’identification
- Afficher les sorties associées aux artefacts de code publiés
- Superviser l’état des ressources de calcul et afficher les journaux du runtime.

Les rôles Synapse peuvent être affectés au niveau de l’étendue de l’espace de travail ou à des étendues plus fines pour limiter les autorisations accordées à des ressources Synapse spécifiques.

### <a name="git-permissions"></a>Autorisations Git

Lors de l’utilisation du développement compatible Git en mode Git, vos autorisations Git déterminent si vous pouvez lire et valider les modifications apportées aux artefacts de code, y compris les définitions d’informations d’identification et les services liés.   
   
### <a name="accessing-data-in-sql"></a>Accès aux données dans SQL

Lorsque vous travaillez avec des pools SQL dédiés et serverless, l’accès du plan de données est contrôlé à l’aide d’autorisations SQL. 

Le créateur d’un espace de travail est assigné en tant qu’administrateur Active Directory sur l’espace de travail.  Après la création, ce rôle peut être attribué à un autre utilisateur ou à un groupe de sécurité dans le portail Azure.

**Pools SQL serverless** : Les administrateurs Synapse disposent d’autorisations `db_owner` (`DBO`) sur le pool SQL serverless, « intégré ». Pour accorder à d’autres utilisateurs l’accès à des pools SQL serverless, les administrateurs Synapse doivent exécuter des scripts SQL sur chaque pool serverless.  

**Pools SQL dédiés** : l’autorisation d’administrateur Active Directory est accordée au créateur de l’espace de travail et aux identités managées pour les ressources Azure de l’espace de travail.  L’autorisation d’accès aux pools SQL dédiés n’est, sinon, pas octroyée automatiquement. Pour accorder à d’autres utilisateurs ou groupes l’accès aux pools SQL dédiés, l’administrateur Active Directory doit exécuter des scripts SQL sur chaque pool SQL dédié.

Consultez [Guide pratique pour configurer le contrôle d’accès Synapse](./how-to-set-up-access-control.md) qui donne des exemples de scripts SQL permettant d’accorder des autorisations SQL dans les pools SQL.  

 ### <a name="accessing-system-managed-data-in-storage"></a>Accès aux données gérées par le système dans le stockage

Les pools SQL serverless et les tables Apache Spark stockent leurs données dans un conteneur ADLS Gen2 associé à l’espace de travail. Les bibliothèques Apache Spark installées par l’utilisateur sont également gérées dans le même compte de stockage.  Pour activer ces cas d’usage, les identités managées pour les ressources Azure de l’espace de travail et les utilisateurs doivent disposer d’un accès **Contributeur aux données de Stockage Blob** sur ce conteneur de stockage ADLS Gen2 de l’espace de travail.  

## <a name="using-security-groups-as-a-best-practice"></a>Utilisation de groupes de sécurité en guise de bonne pratique

Afin de simplifier la gestion du contrôle d’accès, vous pouvez utiliser des groupes de sécurité pour affecter des rôles à des individus et des groupes. Des groupes de sécurité peuvent être créés pour mettre en miroir les personnes ou les fonctions de tâche de votre organisation qui ont besoin d’accéder à des artefacts ou à des ressources Synapse.  Ces groupes de sécurité basés sur des personnes peuvent ensuite se voir attribuer un ou plusieurs rôles Azure, des rôles Synapse, des autorisations SQL ou des autorisations Git. Avec des groupes de sécurité bien choisis, il est facile d’attribuer les autorisations nécessaires à un utilisateur en l’ajoutant au groupe de sécurité approprié. 

>[!Note]
>Si vous utilisez des groupes de sécurité pour gérer l’accès, une latence supplémentaire est introduite par Azure Active Directory avant que les modifications soient prises en compte. 

## <a name="access-control-enforcement-in-synapse-studio"></a>Application du contrôle d’accès dans Synapse Studio

Synapse Studio se comporte différemment selon vos autorisations et le mode actif :
- **Mode direct Synapse :** Synapse Studio vous empêche d’afficher du contenu publié, de publier du contenu ou d’effectuer d’autres actions si vous ne détenez pas l’autorisation adéquate.  Dans certains cas, vous ne pourrez pas créer les artefacts de code que vous ne pouvez pas utiliser ou enregistrer. 
- **Mode Git :** Si vous disposez d’autorisations Git qui vous permettent de valider les modifications apportées à la branche en cours, l’action de validation sera autorisée même si vous n’avez pas l’autorisation de publier des modifications sur le service direct.  

Dans certains cas, vous êtes autorisé à créer des artefacts de code même sans autorisation de publication ou de validation.  Vous pouvez ainsi exécuter du code (avec les autorisations d’exécution exigées). [Apprenez-en davantage](./synapse-workspace-understand-what-role-you-need.md) sur les rôles nécessaires pour les tâches courantes. 

Si une fonctionnalité est désactivée dans Synapse Studio, une info-bulle indique l’autorisation demandée.  Utilisez le [Guide des rôles RBAC Synapse](./synapse-workspace-synapse-rbac-roles.md#synapse-rbac-actions-and-the-roles-that-permit-them) pour rechercher quel rôle est demandé pour fournir l’autorisation manquante.


## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur le contrôle [RBAC Synapse](./synapse-workspace-synapse-rbac.md)
- En savoir plus sur les [Rôles RBAC Synapse](./synapse-workspace-synapse-rbac-roles.md)
- Découvrir le [Guide pratique pour configurer le contrôle d’accès](./how-to-set-up-access-control.md) d’un espace de travail Synapse à l’aide de groupes de sécurité.
- Découvrir le [Guide pratique pour passer en revue les attributions de rôles RBAC Synapse](./how-to-review-synapse-rbac-role-assignments.md)
- Découvrir le [Guide pratique pour gérer les attributions de rôles RBAC Synapse](./how-to-manage-synapse-rbac-role-assignments.md)