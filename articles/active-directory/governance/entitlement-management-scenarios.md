---
title: Scénarios courants de gestion des droits d’utilisation – Azure AD
description: Découvrez les grandes étapes à suivre dans les scénarios courants de la gestion des droits d’utilisation Azure Active Directory.
services: active-directory
documentationCenter: ''
author: ajburnle
manager: daveba
editor: markwahl-msft
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: compliance
ms.date: 06/18/2020
ms.author: ajburnle
ms.reviewer: mwahl
ms.collection: M365-identity-device-management
ms.openlocfilehash: c0f45a1481661aa350815560d795ab7411f99545
ms.sourcegitcommit: ce8eecb3e966c08ae368fafb69eaeb00e76da57e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92317846"
---
# <a name="common-scenarios-in-azure-ad-entitlement-management"></a>Scénarios courants de la gestion des droits d’utilisation Azure AD

Il y a plusieurs façons de configurer la gestion des droits d’utilisation pour votre organisation. Toutefois, si vous débutez, il est utile de comprendre les scénarios courants pour les administrateurs, les propriétaires de catalogues, les gestionnaires de package d’accès, les approbateurs et les demandeurs.

## <a name="delegate"></a>Déléguer

### <a name="administrator-delegate-management-of-resources"></a>Administrateur : Déléguer la gestion des ressources

1. [Regardez la vidéo : Délégation du service informatique au responsable de service](https://www.microsoft.com/videoplayer/embed/RE3Lq00)
1. [Déléguer des utilisateurs au rôle créateur de catalogue](entitlement-management-delegate-catalog.md)

### <a name="catalog-creator-delegate-management-of-resources"></a>Créateur de catalogue : Déléguer la gestion des ressources

- [Créer un catalogue](entitlement-management-catalog-create.md#create-a-catalog)

### <a name="catalog-owner-delegate-management-of-resources"></a>Propriétaire de catalogue : Déléguer la gestion des ressources

1. [Ajouter des copropriétaires au catalogue](entitlement-management-catalog-create.md#add-additional-catalog-owners)
1. [Ajouter des ressources au catalogue](entitlement-management-catalog-create.md#add-resources-to-a-catalog)

### <a name="catalog-owner-delegate-management-of-access-packages"></a>Propriétaire de catalogue : Déléguer la gestion des packages d’accès

1. [Regardez la vidéo : Délégation du propriétaire de catalogue au gestionnaire de package d’accès](https://www.microsoft.com/videoplayer/embed/RE3Lq08)
1. [Déléguer des utilisateurs au rôle de gestionnaire de package d’accès](entitlement-management-delegate-managers.md)

## <a name="govern-access-for-users-in-your-organization"></a>Régir l’accès des utilisateurs de l’organisation

### <a name="access-package-manager-allow-employees-in-your-organization-to-request-access-to-resources"></a>Gestionnaire de package d’accès : Autoriser les employés de l’organisation à demander l’accès aux ressources

1. [Créer un package d’accès](entitlement-management-access-package-create.md#start-new-access-package)
1. [Ajouter des groupes, équipes, applications ou sites SharePoint à un package d’accès](entitlement-management-access-package-create.md#resource-roles)
1. [Ajouter une stratégie de demande pour permettre aux utilisateurs du répertoire de demander l’accès](entitlement-management-access-package-create.md#for-users-in-your-directory)
1. [Spécifier des paramètres d’expiration](entitlement-management-access-package-create.md#lifecycle)

### <a name="requestor-request-access-to-resources"></a>Demandeur : Demander l’accès à des ressources

1. [Se connecter au portail Mon Accès](entitlement-management-request-access.md#sign-in-to-the-my-access-portal)
1. Trouver un package d’accès
1. [Demander l’accès](entitlement-management-request-access.md#request-an-access-package)

### <a name="approver-approve-requests-to-resources"></a>Approbateur : Approuver les demandes d’accès aux ressources

1. [Ouvrir une demande dans le portail Mon Accès](entitlement-management-request-approve.md#open-request)
1. [Approuver ou refuser une demande d’accès](entitlement-management-request-approve.md#approve-or-deny-request)

### <a name="requestor-view-the-resources-you-already-have-access-to"></a>Demandeur : Afficher les ressources auxquelles vous avez déjà accès

1. [Se connecter au portail Mon Accès](entitlement-management-request-access.md#sign-in-to-the-my-access-portal)
1. Consulter les packages d’accès actifs

## <a name="govern-access-for-users-outside-your-organization"></a>Régir l’accès des utilisateurs extérieurs à l’organisation

### <a name="administrator-collaborate-with-an-external-partner-organization"></a>Administrateur : Collaborer avec une organisation partenaire externe

1. [En savoir plus sur le fonctionnement de l’accès pour les utilisateurs externes](entitlement-management-external-users.md#how-access-works-for-external-users)
1. [Vérifier les paramètres pour les utilisateurs externes](entitlement-management-external-users.md#settings-for-external-users)
1. [Ajouter une connexion à l’organisation externe](entitlement-management-organization.md)

### <a name="access-package-manager-collaborate-with-an-external-partner-organization"></a>Gestionnaire de package d’accès : Collaborer avec une organisation partenaire externe

1. [Créer un package d’accès](entitlement-management-access-package-create.md#start-new-access-package)
1. [Ajouter des groupes, équipes, applications ou sites SharePoint à un package d’accès](entitlement-management-access-package-resources.md#add-resource-roles)
1. [Ajouter une stratégie de demande pour permettre aux utilisateurs extérieurs au répertoire de demander l’accès](entitlement-management-access-package-request-policy.md#for-users-not-in-your-directory)
1. [Spécifier des paramètres d’expiration](entitlement-management-access-package-create.md#lifecycle)
1. [Copier le lien pour demander le package d’accès](entitlement-management-access-package-settings.md)
1. Envoyer le lien au partenaire externe à des fins de partage avec ses utilisateurs

### <a name="requestor-request-access-to-resources-as-an-external-user"></a>Demandeur : Demander l’accès aux ressources en tant qu’utilisateur externe

1. Rechercher le lien du package d’accès envoyé par le contact
1. [Se connecter au portail Mon Accès](entitlement-management-request-access.md#sign-in-to-the-my-access-portal)
1. [Demander l’accès](entitlement-management-request-access.md#request-an-access-package)

### <a name="approver-approve-requests-to-resources"></a>Approbateur : Approuver les demandes d’accès aux ressources

1. [Ouvrir une demande dans le portail Mon Accès](entitlement-management-request-approve.md#open-request)
1. [Approuver ou refuser une demande d’accès](entitlement-management-request-approve.md#approve-or-deny-request)

### <a name="requestor-view-the-resources-your-already-have-access-to"></a>Demandeur : Afficher les ressources auxquelles vous avez déjà accès

1. [Se connecter au portail Mon Accès](entitlement-management-request-access.md#sign-in-to-the-my-access-portal)
1. Consulter les packages d’accès actifs

## <a name="day-to-day-management"></a>Gestion quotidienne

### <a name="access-package-manager-update-the-resources-for-a-project"></a>Gestionnaire de package d’accès : Mettre à jour les ressources d’un projet

1. [Regardez la vidéo : Gestion quotidienne : Les choses ont changé](https://www.microsoft.com/videoplayer/embed/RE3LD4Z)
1. Ouvrir le package d’accès
1. [Ajouter ou supprimer des groupes, équipes, applications ou sites SharePoint](entitlement-management-access-package-resources.md#add-resource-roles)

### <a name="access-package-manager-update-the-duration-for-a-project"></a>Gestionnaire de package d’accès : Mettre à jour la durée d’un projet

1. [Regardez la vidéo : Gestion quotidienne : Les choses ont changé](https://www.microsoft.com/videoplayer/embed/RE3LD4Z)
1. Ouvrir le package d’accès
1. [Ouvrir les paramètres de cycle de vie](entitlement-management-access-package-lifecycle-policy.md#open-lifecycle-settings)
1. [Mettre à jour les paramètres d’expiration](entitlement-management-access-package-lifecycle-policy.md#lifecycle) 

### <a name="access-package-manager-update-how-access-is-approved-for-a-project"></a>Gestionnaire de package d’accès : Mettre à jour la manière dont l’accès est approuvé pour un projet

1. [Regardez la vidéo : Gestion quotidienne : Les choses ont changé](https://www.microsoft.com/videoplayer/embed/RE3LD4Z)
1. [Ouvrir une stratégie existante de paramètres de demande](entitlement-management-access-package-request-policy.md#open-an-existing-access-package-and-add-a-new-policy-of-request-settings)
1. [Mettre à jour les paramètres d’approbation](entitlement-management-access-package-approval-policy.md#change-approval-settings-of-an-existing-access-package)

### <a name="access-package-manager-update-the-people-for-a-project"></a>Gestionnaire de package d’accès : Mettre à jour les personnes d’un projet

1. [Regardez la vidéo : Gestion quotidienne : Les choses ont changé](https://www.microsoft.com/videoplayer/embed/RE3LD4Z)
1. [Supprimer les utilisateurs qui n’ont plus besoin d’un accès](entitlement-management-access-package-assignments.md)
1. [Ouvrir une stratégie existante de paramètres de demande](entitlement-management-access-package-request-policy.md#open-an-existing-access-package-and-add-a-new-policy-of-request-settings)
1. [Ajouter des utilisateurs qui ont besoin d’un accès](entitlement-management-access-package-request-policy.md#for-users-in-your-directory)

### <a name="access-package-manager-directly-assign-specific-users-to-an-access-package"></a>Gestionnaire de package d’accès : Affecter directement des utilisateurs spécifiques à un package d’accès

1. [Si les utilisateurs ont besoin de différents paramètres de cycle de vie, ajouter une nouvelle stratégie au package d’accès](entitlement-management-access-package-request-policy.md#open-an-existing-access-package-and-add-a-new-policy-of-request-settings)
1. [Affecter directement des utilisateurs spécifiques au package d’accès](entitlement-management-access-package-assignments.md#directly-assign-a-user)

## <a name="assignments-and-reports"></a>Affectations et rapports

### <a name="administrator-view-who-has-assignments-to-an-access-package"></a>Administrateur : Voir qui a des affectations à un package d’accès

1. Ouvrir un package d’accès
1. [Consulter les attributions](entitlement-management-access-package-assignments.md#view-who-has-an-assignment)
1. [Archiver les rapports et les journaux](entitlement-management-logs-and-reporting.md)

### <a name="administrator-view-resources-assigned-to-users"></a>Administrateur : Afficher les ressources affectées aux utilisateurs

1. [Afficher les packages d’accès d’un utilisateur](entitlement-management-reports.md#view-access-packages-for-a-user)
1. [Afficher les affectations de ressources d’un utilisateur](entitlement-management-reports.md#view-resource-assignments-for-a-user)

## <a name="programmatic-administration"></a>Administration programmatique

Vous pouvez également gérer les packages, les catalogues, les stratégies, les requêtes et les affectations d’accès à l’aide de Microsoft Graph.  Un utilisateur doté d’un rôle approprié dans une application qui a l’autorisation déléguée `EntitlementManagement.ReadWrite.All` peut appeler l’[API de gestion des droits d’utilisation](/graph/tutorial-access-package-api?view=graph-rest-beta).

## <a name="next-steps"></a>Étapes suivantes

- [Délégation et rôles](entitlement-management-delegate.md)
- [Processus de requête et notifications par e-mail](entitlement-management-process.md)