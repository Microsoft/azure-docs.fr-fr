---
title: Définir des rôles Azure pour l’accès d’administration Azure
titleSuffix: Azure Cognitive Search
description: Contrôle d’accès en fonction du rôle Azure (Azure RBAC) sur le portail Azure pour le contrôle et la délégation de tâches d’administration dans le cadre de la gestion de Recherche cognitive Azure.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 10/16/2020
ms.openlocfilehash: ee122727100ec0abad0dfe93b9e5f1be0276cb8e
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92519497"
---
# <a name="set-azure-roles-for-administrative-access-to-azure-cognitive-search"></a>Définir des rôles Azure pour l’accès d’administration à Recherche cognitive Azure

Azure offre un [modèle d’autorisation par rôle global](../role-based-access-control/role-assignments-portal.md) pour tous les services gérés via le portail ou les API Resource Manager. Les rôles Propriétaire, Contributeur et Lecteur définissent le niveau *d’administration des services* pour les utilisateurs, les groupes et les principaux de sécurité Active Directory assignés à chaque rôle. 

> [!Note]
> Il n’existe pas de contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour la sécurisation du contenu sur le service. Vous allez utiliser une clé d’API d’administration ou une clé API de requête pour les demandes authentifiées auprès du service lui-même. Pour un accès en fonction de l’identité sur les résultats de recherche, vous pouvez créer des filtres de sécurité pour ajuster les résultats par identité, en supprimant les documents auxquels le demandeur ne doit pas avoir accès. Pour plus d’informations, consultez [Filtres de sécurité ](search-security-trimming-for-azure-search.md).

## <a name="management-tasks-by-role"></a>Tâches de gestion par rôle

Pour Recherche cognitive Azure, les rôles sont associés à des niveaux d’autorisation qui prennent en charge les tâches de gestion suivantes :

| Role | Tâche |
| --- | --- |
| Propriétaire |Création ou suppression du service ou de tout objet sur le service, y compris les clés API, les index, les indexeurs, les sources de données d’indexeur et les planifications de l’indexeur.<p>Afficher l’état du service, notamment des compteurs et la taille du stockage.<p>Ajout ou suppression d'appartenance à un rôle (seul un Propriétaire peut gérer l'appartenance à un rôle).<p>Les administrateurs d’abonnement et de service appartiennent automatiquement au rôle Propriétaire. |
| Contributeur | Même niveau d’accès que le Propriétaire, à l’exception de la gestion des rôles Azure. Par exemple, un Contributeur peut créer ou supprimer des objets ou afficher et régénérer des [clés API](search-security-api-keys.md), mais il ne peut pas modifier l’appartenance aux rôles.<br><br>[Contributeur du service de recherche](../role-based-access-control/built-in-roles.md#search-service-contributor) est équivalent au rôle intégré de contributeur générique. |
| Lecteur |Affichez les éléments essentiels du service, tels que le point de terminaison de service, l’abonnement, le groupe de ressources, la région, le niveau et la capacité. Vous pouvez également afficher les métriques de service, telles que la moyenne des requêtes par seconde, sous l’onglet Supervision. Les membres de ce rôle ne peuvent pas afficher l’index, l’indexeur, la source de données ni les informations de compétence. Cela comprend les données d’utilisation de ces objets, comme le nombre d’index qui existent sur le service. |

Les rôles n’accordent pas de droits d’accès au point de terminaison de service. Les opérations du service Search telles que la gestion ou le remplissage d'index, tout comme les requêtes de données de recherche, sont contrôlées via des clés api, et non par des rôles. Pour plus d’informations, consultez [Gérer des clés API](search-security-api-keys.md).

## <a name="permissions-table"></a>Tableau des autorisations

Le tableau suivant récapitule les opérations autorisées dans Recherche cognitive Azure, en indiquant la clé qui déverrouille l’accès à une opération particulière.

Les autorisations Azure RBAC s’appliquent aux opérations du portail et à la gestion des services (créer, supprimer ou modifier un service ou ses clés API). Les clés API sont créées après l’existence d’un service et s’appliquent aux opérations de contenu sur le service. En outre, pour les opérations liées au contenu dans le portail, telles que la création ou la suppression d’objets, un propriétaire ou un contributeur Azure RBAC interagit avec le service avec une clé API d’administration implicite.

| Opération | Contrôlé par |
|-----------|-------------------------|
| Créer un service | Autorisations Azure RBAC : Propriétaire ou contributeur |
| Mettre à l’échelle un service | Autorisations Azure RBAC : Propriétaire ou contributeur|
| Supprimer un service | Autorisations Azure RBAC : Propriétaire ou contributeur |
| Gérer les clés d’administration ou de requête | Autorisations Azure RBAC : Propriétaire ou contributeur|
| Afficher les informations de service dans le portail ou une API de gestion | Autorisations Azure RBAC : Propriétaire, Contributeur ou Lecteur  |
| Afficher les informations d’objet et les métriques dans le portail ou une API de gestion | Autorisations Azure RBAC : Propriétaire ou contributeur |
| Créer, modifier et supprimer des objets du service : <br>Index et composants (y compris les définitions de l’analyseur, les profils de scoring et les options CORS), indexeurs, sources de données, synonymes et générateurs de suggestions | Clé d’administration si vous utilisez une API, Propriétaire ou Contributeur Azure RBAC si vous utilisez le portail |
| Interroger un index | Clé d’administration ou de requête si vous utilisez une API, Propriétaire ou Contributeur Azure RBAC si vous utilisez le portail |
| Interroger des informations système à propos d’objets, telles que l’obtention de statistiques, de comptes et de listes d’objets | Clé d’administration si vous utilisez une API, Propriétaire ou Contributeur Azure RBAC si vous utilisez le portail |

## <a name="next-steps"></a>Étapes suivantes

+ [Gestion à l’aide de PowerShell](search-manage-powershell.md) 
+ [Performances et optimisation dans Recherche cognitive Azure](search-performance-optimization.md)
+ [Présentation du contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../role-based-access-control/overview.md).
