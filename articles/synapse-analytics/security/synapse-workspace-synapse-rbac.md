---
title: Contrôle d’accès en fonction du rôle Synapse
description: Article qui explique le contrôle d’accès en fonction du rôle dans Azure Synapse Analytics
author: billgib
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: security
ms.date: 12/1/2020
ms.author: billgib
ms.reviewer: jrasnick
ms.openlocfilehash: 7972f34bf0d2b93828899903e013c2e35bc997c0
ms.sourcegitcommit: 84e3db454ad2bccf529dabba518558bd28e2a4e6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96523273"
---
# <a name="what-is-synapse-role-based-access-control-rbac"></a>Présentation du contrôle d’accès en fonction du rôle (RBAC) Synapse

RBAC Synapse étend les fonctionnalités de [RBAC Azure](https://docs.microsoft.com/azure/role-based-access-control/overview) pour les espaces de travail Synapse et leur contenu. 

RBAC Azure est utilisé pour gérer les utilisateurs autorisés à créer, mettre à jour ou supprimer l’espace de travail Synapse et ses pools SQL, pools Apache Spark et runtimes d’intégration.

RBAC Synapse permet de désigner les personnes autorisées à :
- Publier des artefacts de code et répertorier ou afficher les artefacts de code publiés , 
- Exécuter du code sur des pools Spark Apache et des runtimes d'intégration ;
- Accéder aux services liés (données) protégés par des informations d’identification ; 
- Surveiller ou annuler l’exécution d’un travail, passer en revue la sortie du travail et les journaux d’exécution.  

>[!Note]
>Même si RBAC Synapse est utilisé pour gérer l’accès à des scripts SQL publiés, il offre uniquement un contrôle d’accès limité aux pools SQL serverless et il n’est _pas_ utilisé pour contrôler l’accès à des pools SQL dédiés.  L’accès aux pools SQL est principalement contrôlé à l’aide de la sécurité SQL.

## <a name="what-can-i-do-with-synapse-rbac"></a>Comment utiliser le contrôle RBAC Synapse ?

Voici quelques exemples de ce que vous pouvez faire avec le contrôle RBAC Synapse :
  - Autoriser un utilisateur à publier des modifications apportées à des notebooks et des travaux Apache Spark sur le service en ligne.
  - Autoriser un utilisateur à exécuter et à annuler des notebooks et des travaux Spark sur un pool Apache Spark spécifique.
  - Autoriser un utilisateur à utiliser des informations d’identification spécifiques afin qu’il puisse exécuter des pipelines sécurisés par l’identité du système de l’espace de travail et accéder aux données des services liés sécurisés avec les informations d’identification. 
  - Autoriser un administrateur à gérer, surveiller et annuler l’exécution d’un travail sur des pools Spark spécifiques.    

## <a name="how-synapse-rbac-works"></a>Fonctionnement de RBAC Synapse
Comme RBAC Azure, RBAC Synapse fonctionne en créant des attributions de rôles. Une attribution de rôle repose sur trois éléments : un principal de sécurité, une définition de rôle et une étendue.  

### <a name="security-principals"></a>Principaux de sécurité

Un _principal de sécurité_ est un utilisateur, un groupe, un principal de service ou une identité gérée.

### <a name="roles"></a>Rôles
 
Un _rôle_  est un ensemble d’autorisations ou d’actions qui peuvent être effectuées sur des types de ressources ou d’artefacts spécifiques.

Synapse fournit des rôles intégrés qui définissent des ensembles d’actions correspondant aux besoins des différents personnages :
- Les administrateurs peuvent obtenir un accès complet pour créer et configurer un espace de travail 
- Les développeurs peuvent créer, mettre à jour et déboguer des scripts SQL, des notebooks, des pipelines et des flux de données, mais ils ne peuvent ni publier ni exécuter ce code sur des données/ressources de calcul de production
- Les opérateurs peuvent surveiller et gérer l'état du système, l'exécution des applications et les journaux de révision, sans avoir accès au code ou aux résultats de l'exécution.
- Le personnel de sécurité peut gérer et configurer les points de terminaison sans avoir accès au code ni aux ressources ou données de calcul.

[En savoir plus](./synapse-workspace-synapse-rbac-roles.md) sur les rôles Synapse intégrés. 

### <a name="scopes"></a>Étendues

Une _étendue_ définit les ressources ou artefacts auxquels l’accès s’applique.  Synapse prend en charge les étendues hiérarchiques.  Les autorisations accordées à une étendue de niveau supérieur sont héritées par les objets d’un niveau inférieur.  Dans RBAC Synapse, l’étendue de niveau supérieur est un espace de travail.  L’attribution d’un rôle à l’étendue de l’espace de travail accorde des autorisations à tous les objets applicables dans cet espace de travail.  

Les étendues actuellement prises en charge au sein d’un espace de travail sont : Pool Apache Spark, runtime d’intégration, service lié et informations d’identification. 

L’accès aux artefacts de code est accordé avec la portée de l’espace de travail.  L’octroi de l’accès aux regroupements d’artefacts au sein d’un espace de travail sera pris en charge dans une version ultérieure.

## <a name="resolving-role-assignments-to-determine-permissions"></a>Résolution des attributions de rôles pour déterminer les autorisations

Une attribution de rôle accorde au principal les autorisations définies par le rôle au niveau de l’étendue spécifiée.

RBAC Synapse est un modèle additif similaire à RBAC Azure. Plusieurs rôles peuvent être attribués à un seul principal et à des étendues différentes. Lors du calcul des autorisations d’un principal de sécurité, le système considère tous les rôles attribués au principal et aux groupes qui incluent directement ou indirectement le principal.  Il prend également en compte l’étendue de chaque attribution pour déterminer les autorisations qui s’appliquent.  

## <a name="enforcing-assigned-permissions"></a>Application des autorisations attribuées

Dans Synapse Studio, des boutons ou des options spécifiques peuvent être grisés, ou une erreur d’autorisation peut être renvoyée lorsque vous tentez d’effectuer une action si vous n’avez pas les autorisations requises. 

En cas de désactivation d’un bouton ou d’une option, le fait de pointer sur le bouton ou l’option affiche une infobulle avec l’autorisation requise.  Contactez un administrateur Synapse pour attribuer un rôle qui accorde l’autorisation requise. Vous pouvez afficher [ici](./synapse-workspace-synapse-rbac-roles.md) les rôles qui fournissent des actions spécifiques.

## <a name="who-can-assign-synapse-rbac-roles"></a>Qui peut attribuer des rôles RBAC Synapse ?

Seul un administrateur Synapse peut attribuer des rôles RBAC Synapse.  Un administrateur Synapse au niveau de l’espace de travail peut accorder l’accès à n’importe quelle étendue.  Un administrateur Synapse dans une étendue de niveau inférieur peut uniquement accorder l’accès à cette étendue. 

Lorsqu’un nouvel espace de travail est créé, le créateur se voit automatiquement attribuer le rôle Administrateur Synapse au niveau de l’étendue de l’espace de travail.   

## <a name="where-do-i-manage-synapse-rbac"></a>Où puis-je gérer RBAC Synapse ?

RBAC Synapse est géré à partir de Synapse Studio à l’aide des outils de contrôle d’accès du hub de gestion. 

## <a name="next-steps"></a>Étapes suivantes

Comprendre les [rôles RBAC Synapse](./synapse-workspace-synapse-rbac-roles.md) intégrés.

Découvrir [comment passer en revue les attributions de rôles RBAC Synapse](./how-to-review-synapse-rbac-role-assignments.md) pour un espace de travail.

Découvrir [comment attribuer des rôles RBAC Synapse](./how-to-manage-synapse-rbac-role-assignments.md)