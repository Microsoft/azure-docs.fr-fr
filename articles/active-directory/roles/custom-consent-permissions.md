---
title: Autorisations de consentement d’application pour les rôles personnalisés dans Azure Active Directory | Microsoft Docs
description: Examinez les autorisations de consentement d’application pour les rôles Azure AD personnalisés dans le portail Azure, PowerShell ou l’API Graph.
services: active-directory
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: roles
ms.topic: overview
ms.date: 11/04/2020
ms.author: curtand
ms.reviewer: psignoret
ms.custom: it-pro
ms.openlocfilehash: 358a458698ec1fd8443e15f71a84e057f33af527
ms.sourcegitcommit: 0d171fe7fc0893dcc5f6202e73038a91be58da03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93376729"
---
# <a name="app-consent-permissions-for-custom-roles-in-azure-active-directory"></a>Autorisations de consentement d’application pour les rôles personnalisés dans Azure Active Directory

Cet article contient les autorisations de consentement d’application disponibles pour les définitions de rôle personnalisées dans Azure Active Directory (Azure AD). Dans cet article, vous trouverez les autorisations requises pour certains scénarios courants liés aux autorisations et au consentement d’application.

## <a name="required-license-plan"></a>Plan de licence obligatoire

Pour utiliser cette fonctionnalité, votre organisation Azure AD doit avoir une licence Azure AD Premium P1. Pour trouver la licence appropriée à vos besoins, consultez [Comparaison des fonctionnalités mises à la disposition générale des éditions gratuite, de base et Premium](https://azure.microsoft.com/pricing/details/active-directory/).

## <a name="app-consent-permissions"></a>Autorisations de consentement d’application

Utilisez les autorisations listées dans cet article pour gérer les stratégies de consentement d’application ainsi que l’autorisation d’accorder un consentement aux applications.

> [!NOTE]
> Le portail d’administration Azure AD ne prend pas encore en charge l’ajout des autorisations listées dans cet article à une définition de rôle d’annuaire personnalisé. Vous devez [utiliser Azure AD PowerShell pour créer un rôle d’annuaire personnalisé](custom-create.md#create-a-role-using-powershell) avec les autorisations listées dans cet article.

### <a name="granting-delegated-permissions-to-apps-on-behalf-of-self-user-consent"></a>Octroi d’autorisations déléguées aux applications en son propre nom (consentement de l’utilisateur)

Pour permettre aux utilisateurs d’accorder leur consentement aux applications en leur propre nom (consentement de l’utilisateur), sous réserve d’une stratégie de consentement d’application :

- microsoft.directory/servicePrincipals/managePermissionGrantsForSelf.{id}

Où `{id}` est remplacé par l’ID d’une [stratégie de consentement d’application](../manage-apps/manage-app-consent-policies.md) qui définit les conditions à remplir pour que cette autorisation soit active.

Par exemple, pour permettre aux utilisateurs d’accorder leur consentement pour leur propre compte, en fonction de la stratégie de consentement d’application intégrée ayant l’ID `microsoft-user-default-low`, vous utilisez l’autorisation `...managePermissionGrantsForSelf.microsoft-user-default-low`.

### <a name="granting-permissions-to-apps-on-behalf-of-all-admin-consent"></a>Octroi d’autorisations à des applications pour le compte de tous (consentement administrateur)

Pour déléguer le consentement administrateur au niveau du locataire pour des applications, à la fois pour les autorisations déléguées et les autorisations d’application (rôles d’application) :

- microsoft.directory/servicePrincipals/managePermissionGrantsForAll.{id}

Où `{id}` est remplacé par l’ID d’une [stratégie de consentement d’application](../manage-apps/manage-app-consent-policies.md) qui définit les conditions à remplir pour que cette autorisation soit utilisable.

Par exemple, pour autoriser les destinataires de rôles à accorder un consentement administrateur au niveau du locataire aux applications soumises à une [stratégie de consentement d’application](../manage-apps/manage-app-consent-policies.md) personnalisée ayant l’ID `low-risk-any-app`, vous devez utiliser l’autorisation `microsoft.directory/servicePrincipals/managePermissionGrantsForAll.low-risk-any-app`.

### <a name="managing-app-consent-policies"></a>Gestion des stratégies de consentement d’application

Pour déléguer la création, la mise à jour et la suppression de [stratégies de consentement d’application](../manage-apps/manage-app-consent-policies.md) :

- microsoft.directory/permissionGrantPolicies/create
- microsoft.directory/permissionGrantPolicies/standard/read
- microsoft.directory/permissionGrantPolicies/basic/update
- microsoft.directory/permissionGrantPolicies/delete

## <a name="full-list-of-permissions"></a>Liste complète des autorisations

Autorisation | Description
---------- | -----------
microsoft.directory/servicePrincipals/managePermissionGrantsForSelf.{id} | Octroie la possibilité d’accorder son consentement aux applications en son propre nom (consentement de l’utilisateur), sous réserve de la stratégie de consentement d’application `{id}`.
microsoft.directory/servicePrincipals/managePermissionGrantsForAll.{id} | Octroie l’autorisation d’accorder son consentement aux applications pour le compte de tous (consentement administrateur au niveau du locataire), sous réserve de la stratégie de consentement d’application `{id}`.
microsoft.directory/permissionGrantPolicies/standard/read | Accorde la possibilité de lire les stratégies de consentement d’application.
microsoft.directory/permissionGrantPolicies/basic/update | Accorde la possibilité de mettre à jour les propriétés de base sur les stratégies de consentement d’application existantes.
microsoft.directory/permissionGrantPolicies/create | Accorde la possibilité de créer des stratégies de consentement d’application.
microsoft.directory/permissionGrantPolicies/delete | Accorde la possibilité de supprimer des stratégies de consentement d’application.

## <a name="next-steps"></a>Étapes suivantes

- [Créer des rôles personnalisés à l’aide du portail Azure, d’Azure AD PowerShell et de l’API Graph](custom-create.md)
- [Afficher les attributions d’un rôle personnalisé](../roles/view-assignments.md)
