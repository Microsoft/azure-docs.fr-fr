---
title: Autoriser l’accès à Azure App Configuration Azure avec Azure Active Directory
description: Activer Azure RBAC pour autoriser l’accès à votre instance Azure App Configuration
author: AlexandraKemperMS
ms.author: alkemper
ms.date: 05/26/2020
ms.topic: conceptual
ms.service: azure-app-configuration
ms.openlocfilehash: 4768dbe292b7c71770ded1e8ad27025bc9944608
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96930260"
---
# <a name="authorize-access-to-azure-app-configuration-using-azure-active-directory"></a>Autoriser l’accès à Azure App Configuration Azure avec Azure Active Directory
Outre l’utilisation du Hash-based Message Authentication Code (HMAC), Azure App Configuration prend en charge l’utilisation de Azure Active Directory (Azure AD) pour autoriser les requêtes d’instances d’App Configuration.  Azure AD vous permet d’utiliser le contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour accorder des autorisations à un principal de sécurité.  Un utilisateur, une [identité managée](../active-directory/managed-identities-azure-resources/overview.md) ou un [principal du service d’application](../active-directory/develop/app-objects-and-service-principals.md) peuvent être des principaux de sécurité.  Pour en savoir plus sur les rôles et les attributions de rôles, consultez [Comprendre les différents rôles](../role-based-access-control/overview.md).

## <a name="overview"></a>Vue d’ensemble
Les requêtes, effectuées par un principal de sécurité pour accéder à une ressource App Configuration, doivent être autorisées. Avec Azure AD, l’accès à une ressource est un processus en deux étapes :
1. L’identité du principal de sécurité est authentifiée, et un jeton OAuth 2.0 est renvoyé.  Le nom de ressource pour demander un jeton est `https://login.microsoftonline.com/{tenantID}` où `{tenantID}` correspond à l’ID de locataire Azure Active Directory auquel appartient le principal de service.
2. Le jeton est transmis dans une requête adressée au service App Configuration pour autoriser l’accès à la ressource spécifiée.

L’étape d’authentification nécessite qu’une requête d’application contienne un jeton d’accès OAuth 2.0 au moment de l’exécution.  Si une application s’exécute à partir d’une entité Azure telle qu’une application Azure Functions, une application web Azure ou une machine virtuelle Azure, elle peut utiliser une identité managée pour accéder aux ressources.  Pour plus d’informations sur l’authentification des requêtes adressées à Azure App Configuration par une identité managée, consultez [Authentifier l’accès aux ressources Azure App Configuration avec Azure Active Directory et les identités managées pour les ressources Azure](howto-integrate-azure-managed-service-identity.md).

L’étape d’autorisation exige qu’un ou plusieurs rôles Azure soient attribués au principal de sécurité. Azure App Configuration fournit des rôles Azure qui englobent des ensembles d’autorisations pour les ressources App Configuration. Les rôles qui sont attribués à un principal de sécurité déterminent les autorisations fournies au principal. Pour plus d’informations sur les rôles Azure, consultez [Rôles Azure intégrés pour Azure App Configuration](#azure-built-in-roles-for-azure-app-configuration). 

## <a name="assign-azure-roles-for-access-rights"></a>Attribuer des rôles Azure pour les droits d’accès
Azure Active Directory (Azure AD) autorise les droits d’accès aux ressources sécurisées par le biais du [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../role-based-access-control/overview.md).

Lorsqu’un rôle Azure est attribué à un principal de sécurité Azure AD, Azure octroie l’accès à ces ressources pour ce principal de sécurité. L’accès est limité à la ressource App Configuration. Un principal de sécurité Azure AD peut correspondre à un utilisateur, à un principal de service d’application ou à une [identité managée pour les ressources Azure](../active-directory/managed-identities-azure-resources/overview.md).

## <a name="azure-built-in-roles-for-azure-app-configuration"></a>Rôles Azure intégrés pour Azure App Configuration
Azure fournit les rôles Azure intégrés suivants pour autoriser l’accès aux données App Configuration à l’aide d’Azure AD et d’OAuth :

- **Propriétaire des données App Configuration** : Utilisez ce rôle pour accorder un accès en lecture/écriture/suppression aux données App Configuration. Cela n’accorde pas l’accès à la ressource App Configuration.
- **Lecteur des données App Configuration** : Utilisez ce rôle pour accorder un accès en lecture aux données App Configuration. Cela n’accorde pas l’accès à la ressource App Configuration.
- **Collaborateur** : Utilisez ce rôle pour gérer la ressource App Configuration. Alors que les données d’App Configuration sont accessibles à l’aide de clés d’accès, ce rôle n’accorde pas l’accès aux données avec Azure AD.
- **Lecteur** : Utilisez ce rôle pour accorder un accès en lecture à la ressource App Configuration. Cela n’accorde pas l’accès aux clés d’accès de la ressource ni aux données stockées dans App Configuration.

> [!NOTE]
> Actuellement, le portail Azure prend uniquement en charge l’authentification HMAC pour accéder aux données d’App Configuration. L’authentification Azure AD n’est pas prise en charge. Par conséquent, les utilisateurs du portail Azure requièrent que le rôle *Contributeur* récupère les clés d’accès de la ressource App Configuration. L’octroi des rôles *Lecteur de données d’App Configuration* ou *Propriétaire de données d’App Configuration* n’a aucun impact sur l’accès via le portail.

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur l’utilisation des [identités managées](howto-integrate-azure-managed-service-identity.md) pour gérer votre service App Configuration.
