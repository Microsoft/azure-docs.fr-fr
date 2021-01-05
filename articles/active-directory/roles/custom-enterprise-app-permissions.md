---
title: Autorisations d’application pour les rôles personnalisés dans Azure Active Directory | Microsoft Docs
description: Examinez les autorisations d’application d’entreprise pour les rôles Azure AD personnalisés dans le portail Azure, PowerShell ou l’API Graph.
services: active-directory
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: roles
ms.topic: overview
ms.date: 11/04/2020
ms.author: curtand
ms.reviewer: vincesm
ms.custom: it-pro
ms.openlocfilehash: 39d9cf9ed87c8a8f45cb2a6239292562035d31e3
ms.sourcegitcommit: 0d171fe7fc0893dcc5f6202e73038a91be58da03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93379636"
---
# <a name="enterprise-application-permissions-for-custom-roles-in-azure-active-directory"></a>Autorisations d’application d’entreprise pour les rôles personnalisés dans Azure Active Directory

Cet article détaille les autorisations d’application d’entreprise actuellement disponibles pour les définitions de rôles personnalisés dans Azure Active Directory (Azure AD). Dans cet article, vous trouverez des listes d’autorisations pour certains scénarios courants ainsi que la liste complète des autorisations d’application d’entreprise. Les autorisations de proxy d’application ne sont pas déployées dans cette version.

## <a name="required-license-plan"></a>Plan de licence obligatoire

Pour utiliser cette fonctionnalité, votre organisation Azure AD doit avoir une licence Azure AD Premium P1. Pour trouver la licence appropriée à vos besoins, consultez [Comparaison des fonctionnalités mises à la disposition générale des éditions gratuite, de base et Premium](https://azure.microsoft.com/pricing/details/active-directory/).

## <a name="enterprise-application-permissions"></a>Autorisations d’application d’entreprise

Pour plus d’informations sur l’utilisation de ces autorisations, consultez [Attribuer des rôles personnalisés pour gérer les applications d’entreprise](custom-enterprise-apps.md)

### <a name="assigning-users-or-groups-to-an-application"></a>Affectation d’utilisateurs ou de groupes à une application

Pour déléguer l’affectation des utilisateurs et groupes autorisés à accéder aux applications d’authentification unique SAML. Autorisations requises

- microsoft.directory/servicePrincipals/appRoleAssignedTo/update

### <a name="creating-gallery-applications"></a>Création d’applications de la galerie

Pour déléguer la création d’applications de la galerie Azure AD comme ServiceNow, F5, Salesforce, entre autres. Autorisations requises :

- microsoft.directory/applicationTemplates/instantiate

### <a name="configuring-basic-saml-urls"></a>Configuration des URL SAML de base

Pour déléguer la lecture et la mise à jour des configurations SAML de base pour les applications d’authentification unique SAML. Autorisations requises :

- microsoft.directory/servicePrincipals/authentication/update
- microsoft.directory/applications.myOrganization/authentication/update

### <a name="rolling-over-or-creating-signing-certs"></a>Substitution ou création de certificats de signature

Pour déléguer la gestion des certificats de signature pour les applications d’authentification unique SAML. Autorisations requises.

microsoft.directory/applications/credentials/update

### <a name="update-expiring-sign-in-cert-notification-email-address"></a>Mettre à jour les adresses e-mail de notification d’expiration des certificats de connexion

Pour déléguer la mise à jour des adresses e-mail de notification d’expiration des certificats de connexion pour les applications d’authentification unique SAML. Autorisations requises :

- microsoft.directory/applications.myOrganization/authentication/update
- microsoft.directory/applications.myOrganization/permissions/update
- microsoft.directory/servicePrincipals/authentication/update
- microsoft.directory/servicePrincipals/basic/update

### <a name="manage-saml-token-signature-and-sign-in-algorithm"></a>Gérer la signature de jeton SAML et l’algorithme de connexion

Pour déléguer la mise à jour de la signature de jeton SAML et de l’algorithme de connexion pour les applications d’authentification unique SAML. Autorisations requises :

- microsoft.directory/applicationPolicies/basic/update
- microsoft.directory/applications/authentication/update
- microsoft.directory/servicePrincipals/policies/update

### <a name="manage-user-attributes-and-claims"></a>Gérer les attributs et revendications utilisateur

Pour déléguer la création, la suppression et la mise à jour des attributs et revendications utilisateur pour les applications d’authentification unique SAML. Autorisations requises :

- microsoft.directory/applicationPolicies/basic/update
- microsoft.directory/applications/authentication/update
- microsoft.directory/servicePrincipals/policies/update

## <a name="app-provisioning-permissions"></a>Autorisations de provisionnement d’application

Toute opération d’écriture, telle que la gestion du travail, du schéma ou des informations d’identification par le biais de l’interface utilisateur, nécessite également les autorisations de lecture pour afficher la page de provisionnement.

La définition de l’étendue sur tous les utilisateurs et groupes ou sur les utilisateurs et groupes affectés nécessite les autorisations synchronizationJob et synchronizationCredentials.

### <a name="turn-on-or-restart-provisioning-jobs"></a>Activer ou redémarrer des travaux de provisionnement

Pour déléguer la capacité à activer, désactiver et redémarrer des travaux de provisionnement. Autorisations requises :

- microsoft.directory/servicePrincipals/synchronizationJobs/manage  

### <a name="configure-the-provisioning-schema"></a>Configurer le schéma de provisionnement  

Pour déléguer les mises à jour du mappage des attributs. Autorisations requises :

- microsoft.directory/servicePrincipals/synchronizationSchema/manage  

### <a name="read-provisioning-settings-associated-with-the-application-object"></a>Lire les paramètres de provisionnement associés à l’objet application

Pour déléguer la capacité à lire les paramètres de provisionnement associés à l’objet. Autorisations requises :

- microsoft.directory/applications/synchronization/standard/read

### <a name="read-provisioning-settings-associated-with-your-service-principal"></a>Lire les paramètres de provisionnement associés au principal de service

Pour déléguer la capacité à lire les paramètres de provisionnement associés au principal de service. Autorisations requises :

- microsoft.directory/servicePrincipals/synchronization/standard/read

### <a name="authorize-application-access-for-provisioning"></a>Autoriser l’accès aux applications pour le provisionnement  

Pour déléguer la capacité à autoriser l’accès aux applications pour le provisionnement. Exemple de jeton du porteur Oauth en entrée. Autorisations requises :

- microsoft.directory/servicePrincipals/synchronizationCredentials/manage

## <a name="full-list-of-permissions"></a>Liste complète des autorisations

Autorisation | Description
---------- | -----------
microsoft.directory/applicationPolicies/allProperties/read | Lire toutes les propriétés des stratégies d’application.
microsoft.directory/applicationPolicies/allProperties/update | Mettre à jour toutes les propriétés des stratégies d’application.
microsoft.directory/applicationPolicies/basic/update | Mettre à jour les propriétés standard des stratégies d’application.
microsoft.directory/applicationPolicies/create | Créer des stratégies d’application.
microsoft.directory/applicationPolicies/createAsOwner | Créer des stratégies d’application. Le créateur est ajouté en tant que premier propriétaire.
microsoft.directory/applicationPolicies/delete | Supprimer des stratégies d’application.
microsoft.directory/applicationPolicies/owners/read | Lire les propriétaires des stratégies d’application.
microsoft.directory/applicationPolicies/owners/update | Mettre à jour les propriétaires des stratégies d’application.
microsoft.directory/applicationPolicies/policyAppliedTo/read | Lire les stratégies d’application appliquées à la liste d’objets.
microsoft.directory/applicationPolicies/standard/read | Lire les propriétés standard des stratégies d’application.
microsoft.directory/servicePrincipals/allProperties/allTasks | Créez et supprimez des principaux de service (servicePrincipals), et lisez et mettez à jour toutes les propriétés dans Azure Active Directory.
microsoft.directory/servicePrincipals/allProperties/read | Lire toutes les propriétés sur les principaux de service.
microsoft.directory/servicePrincipals/allProperties/update | Mettre à jour toutes les propriétés sur les principaux de service.
microsoft.directory/servicePrincipals/appRoleAssignedTo/read | Lire les attributions de rôles des principaux de service.
microsoft.directory/servicePrincipals/appRoleAssignedTo/update | Mettre à jour les attributions de rôles des principaux de service.
microsoft.directory/servicePrincipals/appRoleAssignments/read | Lire les attributions de rôles affectées aux principaux de service.
microsoft.directory/servicePrincipals/audience/update | Mettre à jour les propriétés d’audience sur les principaux de service.
microsoft.directory/servicePrincipals/authentication/update | Mettre à jour les propriétés d’authentification sur les principaux de service.
microsoft.directory/servicePrincipals/basic/update | Mettre à jour les propriétés de base sur les principaux de service.
microsoft.directory/servicePrincipals/create | Créer des principaux de service.
microsoft.directory/servicePrincipals/createAsOwner | Créer des principaux de service. Le créateur est ajouté en tant que premier propriétaire.
microsoft.directory/servicePrincipals/credentials/update | Mettre à jour les propriétés d’identification sur les principaux de service.
microsoft.directory/servicePrincipals/delete | Supprimer des principaux de service.
microsoft.directory/servicePrincipals/disable | Désactiver des principaux de service.
microsoft.directory/servicePrincipals/enable | Activer des principaux de service.
microsoft.directory/servicePrincipals/getPasswordSingleSignOnCredentials | Lire les informations d’identification de l’authentification unique par mot de passe sur les principaux de service.
microsoft.directory/servicePrincipals/managePasswordSingleSignOnCredentials | Gérer les informations d’identification de l’authentification unique par mot de passe sur les principaux de service.
microsoft.directory/servicePrincipals/oAuth2PermissionGrants/read | Lire les autorisations déléguées accordées sur les principaux de service.
microsoft.directory/servicePrincipals/owners/read | Lire les propriétaires sur les principaux de service.
microsoft.directory/servicePrincipals/owners/update | Mettre à jour les propriétaires sur les principaux de service.
microsoft.directory/servicePrincipals/permissions/update |  
microsoft.directory/servicePrincipals/policies/read | Lire les stratégies sur les principaux de service.
microsoft.directory/servicePrincipals/policies/update | Mettre à jour les stratégies sur les principaux de service.
microsoft.directory/servicePrincipals/standard/read | Lire les propriétés standard des principaux de service.
microsoft.directory/servicePrincipals/synchronization/standard/read | Lire les paramètres de provisionnement associés au principal de service.
microsoft.directory/servicePrincipals/tag/update | Mettre à jour la propriété Tags sur les principaux de service.
microsoft.directory/applicationTemplates/instantiate | Instanciez des applications de la galerie à partir de modèles d’application.
microsoft.directory/auditLogs/allProperties/read | Lire les journaux d’audit.
microsoft.directory/signInReports/allProperties/read | Lire les rapports de connexion.
microsoft.directory/applications/synchronization/standard/read | Lire les paramètres de provisionnement associés à l’objet application.
microsoft.directory/servicePrincipals/synchronizationJobs/manage | Gérer tous les aspects de la synchronisation des travaux pour les ressources des principaux de service.
microsoft.directory/servicePrincipals/synchronization/standard/read | Lire les paramètres de provisionnement associés aux principaux de service.
microsoft.directory/servicePrincipals/synchronizationSchema/manage | Gérer tous les aspects de la synchronisation de schéma pour les ressources des principaux de service.
microsoft.directory/provisioningLogs/allProperties/read | Lire toutes les propriétés des journaux de provisionnement.

## <a name="next-steps"></a>Étapes suivantes

- [Créer des rôles personnalisés à l’aide du portail Azure, d’Azure AD PowerShell et de l’API Graph](custom-create.md)
- [Afficher les attributions d’un rôle personnalisé](custom-view-assignments.md)
