---
title: Déléguer des autorisations d’administrateur de gestion des applications – Azure AD | Microsoft Docs
description: Accorder des autorisations pour la gestion de l’accès aux applications dans Azure Active Directory
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: roles
ms.topic: how-to
ms.date: 11/04/2020
ms.author: curtand
ms.reviewer: vincesm
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 9ad21e15d33dabba74d338b339e4b2fdc25df52e
ms.sourcegitcommit: 0d171fe7fc0893dcc5f6202e73038a91be58da03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93379211"
---
# <a name="delegate-app-registration-permissions-in-azure-active-directory"></a>Déléguer des autorisations d’inscription d’application dans Azure Active Directory

Cet article explique comment utiliser des autorisations accordées par des rôles personnalisés dans Azure Active Directory (Azure AD) pour répondre à vos besoins en matière de gestion des applications. Dans Azure AD, vous pouvez déléguer les autorisations de création et de gestion d’applications en procédant ainsi :

- [Restriction des utilisateurs autorisés à créer des applications](#restrict-who-can-create-applications) et à gérer les applications qu’ils créent. Par défaut, dans Azure AD, tous les utilisateurs peuvent inscrire des inscriptions d’applications et gérer tous les aspects des applications qu’ils créent. Cela peut être limité pour autoriser uniquement les personnes qui sont autorisées.
- [Attribution d’un ou plusieurs propriétaires à une application](#assign-application-owners). Il s’agit d’un moyen simple d’accorder à une personne la possibilité de gérer tous les aspects de la configuration Azure AD pour une application spécifique.
- [Attribution d’un rôle d’administrateur intégré](#assign-built-in-application-admin-roles) qui autorise l’accès à la gestion de la configuration dans Azure AD pour toutes les applications. Il s’agit de la méthode recommandée pour permettre aux experts informatiques de gérer des autorisations de configuration d’application étendues sans accorder l’accès pour gérer d’autres parties de Azure AD qui ne sont pas liées à la configuration de l’application.
- [Création d’un rôle personnalisé](#create-and-assign-a-custom-role-preview) qui définit des autorisations très spécifiques et l’affecte à une personne soit au niveau d’une application unique en tant que propriétaire limité ou au niveau de l’étendue du répertoire (toutes les applications) en tant qu’administrateur limité.

Il est important de considérer l’octroi de l’accès à l’aide de l’une des méthodes ci-dessus pour deux raisons. Tout d’abord, la délégation de la possibilité d’effectuer des tâches d’administration réduit la charge de l’administrateur général. Deuxièmement, l’utilisation d’autorisations limitées améliore votre position de sécurité et réduit le risque d’accès non autorisé. Des problèmes et des instructions générales en lien avec la délégation sont abordés dans [Déléguer l’administration dans Azure Active Directory](concept-delegation.md).

## <a name="restrict-who-can-create-applications"></a>Restreindre qui peut créer des applications

Par défaut, dans Azure AD, tous les utilisateurs peuvent inscrire des inscriptions d’applications et gérer tous les aspects des applications qu’ils créent. Tout le monde peut également donner son consentement aux applications qui accèdent aux données de l’entreprise en leur nom. Vous pouvez choisir d’accorder ces autorisations de manière sélective en définissant les commutateurs globaux sur « non » et en ajoutant les utilisateurs sélectionnés au rôle développeur d’applications.

### <a name="to-disable-the-default-ability-to-create-application-registrations-or-consent-to-applications"></a>Pour désactiver la possibilité par défaut de créer des inscriptions d’applications ou de donner leur consentement aux applications

1. Connectez-vous à votre organisation Azure AD avec un compte qui est éligible au rôle d’administrateur d’entreprise dans votre organisation Azure AD.
1. Définissez un ou plusieurs des paramètres suivants :

    - Sur la page [Paramètres utilisateur de votre organisation](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/UserSettings), définissez le paramètre **Les utilisateurs peuvent inscrire des applications** sur Non. Cela désactive la capacité par défaut pour les utilisateurs de créer des inscriptions d’applications.
    - Sur les [paramètres utilisateur pour les applications d’entreprise](https://portal.azure.com/#blade/Microsoft_AAD_IAM/StartboardApplicationsMenuBlade/UserSettings/menuId/), définissez **Les utilisateurs peuvent donner leur consentement aux applications qui accèdent à des données d’entreprise en leur nom** sur Non. Cela désactive la possibilité par défaut pour les utilisateurs de donner leur consentement aux applications qui accèdent aux données de l’entreprise en leur nom.

### <a name="grant-individual-permissions-to-create-and-consent-to-applications-when-the-default-ability-is-disabled"></a>Accorder des autorisations individuelles pour créer des applications et donner leur consentement lorsque la capacité par défaut est désactivée

Affectez le rôle Développeur d’applications pour accorder la possibilité de créer des inscriptions d'applications lorsque le paramètre **Les utilisateurs peuvent inscrire des applications** est défini sur Non. Ce rôle octroie aussi l’autorisation de donner son consentement en son propre nom lorsque le paramètre **Les utilisateurs peuvent autoriser les applications à accéder aux données de l’entreprise en leur nom** est défini sur Non. En guise de comportement du système, lorsqu’un utilisateur crée une nouvelle inscription d’application, il est automatiquement ajouté en tant que premier propriétaire. Les autorisations de propriété offrent à l’utilisateur la possibilité de gérer tous les aspects d’une inscription d’application ou d’une application d’entreprise qu’ils possèdent.

## <a name="assign-application-owners"></a>Assigner les propriétaires de l’application

L'assignation de propriétaires est un moyen simple d'octroyer la possibilité de gérer tous les aspects de la configuration Azure AD pour un enregistrement d'application spécifique ou une application d'entreprise. En guise de comportement du système, lorsqu’un utilisateur crée une nouvelle inscription d’application, il est automatiquement ajouté en tant que premier propriétaire. Les autorisations de propriété offrent à l’utilisateur la possibilité de gérer tous les aspects d’une inscription d’application ou d’une application d’entreprise qu’ils possèdent. Le propriétaire d'origine peut être supprimé et des propriétaires supplémentaires peuvent être ajoutés.

### <a name="enterprise-application-owners"></a>Propriétaires d'applications d'entreprise

En tant que propriétaire, un utilisateur peut gérer la configuration spécifique à l'organisation de l'application d'entreprise, telle que la configuration de la connexion unique, l’approvisionnement et les affectations d'utilisateurs. Un propriétaire peut également ajouter ou supprimer des propriétaires. Contrairement aux administrateurs généraux, les propriétaires ne peuvent gérer que les applications d’entreprise qu’ils possèdent.

Dans certains cas, les applications d'entreprise créées à partir de la galerie d'applications incluent à la fois une application d'entreprise et un enregistrement d'application. Lorsque cela est vrai, l'ajout d'un propriétaire à l'application d'entreprise ajoute automatiquement le propriétaire à l'enregistrement d'application correspondant en tant que propriétaire.

### <a name="to-assign-an-owner-to-an-enterprise-application"></a>Pour affecter un propriétaire à une application d'entreprise

1. Connectez-vous à votre [organisation Azure AD](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview) avec un compte éligible pour l’administrateur d’application ou l’administrateur d’application cloud pour l’organisation.
1. Sur la [page Inscriptions d’applications](https://portal.azure.com/#blade/Microsoft_AAD_IAM/StartboardApplicationsMenuBlade/AllApps/menuId/) de l’organisation, sélectionnez une application pour ouvrir la page Vue d’ensemble de l’application.
1. Sélectionnez **Propriétaires** pour afficher la liste des propriétaires de l’application.
1. Sélectionnez **Ajouter** pour sélectionner un ou plusieurs propriétaires à ajouter à l’application.

> [!IMPORTANT]
> Les utilisateurs et les principaux de service peuvent être des propriétaires d’inscriptions d’applications. Seuls les utilisateurs peuvent être propriétaires d’applications d’entreprise. Les groupes ne peuvent pas être affectés en tant que propriétaires de l’un ou l’autre.
>
> Les propriétaires peuvent ajouter des informations d’identification à une application et utiliser ces informations d’identification pour emprunter l’identité de l’application. L'application peut disposer de plus d'autorisations que le propriétaire, et constituerait donc une élévation de privilèges par rapport à ce à quoi le propriétaire a accès en tant qu'utilisateur ou principal du service. Un propriétaire d’application peut potentiellement créer ou mettre à jour des utilisateurs ou d’autres objets en usurpant l’identité de l’application, en fonction des autorisations de l’application.

## <a name="assign-built-in-application-admin-roles"></a>Affecter des rôles d’administrateur d’application intégrés

Azure AD a un ensemble de rôles d’administrateur intégrés pour accorder l’accès à la gestion de la configuration dans Azure AD pour toutes les applications. Ces rôles sont la méthode recommandée pour accorder aux experts informatiques l’accès pour gérer des autorisations de configuration d’application étendues sans accorder l’accès pour gérer d’autres parties de Azure AD qui ne sont pas liées à la configuration de l’application.

- Administrateur d’application : Les utilisateurs dans ce rôle peuvent créer et gérer tous les aspects des applications d’entreprise, des inscriptions d’application et des paramètres de proxy d’application. Ce rôle permet également de donner son consentement à des autorisations déléguées et des autorisations d’application, à l’exception de Microsoft Graph. Les utilisateurs affectés à ce rôle ne sont pas ajoutés en tant que propriétaires lorsque des inscriptions d’applications ou des applications d’entreprise sont créées.
- Administrateur d’application cloud Les utilisateurs dans ce rôle ont les mêmes autorisations que celles du rôle Administrateur d’application, sans la possibilité de gérer le proxy d’application. Les utilisateurs affectés à ce rôle ne sont pas ajoutés en tant que propriétaires lorsque des inscriptions d’applications ou des applications d’entreprise sont créées.

Pour plus d’informations et pour afficher la description de ces rôles, consultez [Rôles disponibles](permissions-reference.md#available-roles).

Suivez les instructions fournies dans le guide pratique [Attribuer des rôles aux utilisateurs avec Azure Active Directory](../fundamentals/active-directory-users-assign-role-azure-portal.md) pour attribuer des rôles administrateur d’application ou administrateur d’application Cloud.

> [!IMPORTANT]
> Les administrateurs d’applications et les administrateurs d’applications Cloud peuvent ajouter des informations d’identification à une application et utiliser ces informations d’identification pour emprunter l’identité de l’application. L'application peut disposer d'autorisations correspondant à une élévation de privilèges par rapport aux autorisations du rôle admin. Un administrateur de ce rôle peut potentiellement créer ou mettre à jour des utilisateurs ou d’autres objets tout en empruntant l’identité de l’application, en fonction des autorisations de l’application.
> Aucun rôle n’accorde la possibilité de gérer les paramètres d’accès conditionnel.

## <a name="create-and-assign-a-custom-role-preview"></a>Créer et affecter un rôle personnalisé (préversion)

La création de rôles personnalisés et l’attribution de rôles personnalisés sont des étapes distinctes :

- [Créez une *définition de rôle*](custom-create.md) personnalisée et [ajoutez-lui des autorisations à partir d’une liste prédéfinie](custom-available-permissions.md). Il s’agit des mêmes autorisations que celles utilisées dans les rôles intégrés.
- [Créez une *attribution de rôle*](custom-assign-powershell.md) pour attribuer le rôle personnalisé.

Cette séparation vous permet de créer une définition de rôle unique, puis de l'attribuer plusieurs fois à différentes *portées*. Un rôle personnalisé peut être attribué au niveau de l’organisation, ou à l'étendue s'il s'agit d'un seul objet Azure AD. Un exemple d’étendue d’objet est une inscription d’application unique. À l'aide d'étendues différentes, la même définition de rôle peut être attribuée à Sally pour tous les enregistrements d'application de l'organisation, puis à Naveen uniquement pour l'enregistrement de l'application Contoso Expense Reports.

Conseils lors de la création et de l’utilisation de rôles personnalisés pour la délégation de la gestion d’applications :
- Les rôles personnalisés accordent uniquement l’accès dans les panneaux d’inscription d’application les plus récents du portail Azure AD. Ils n’accordent pas d’accès dans les panneaux Inscriptions d’applications héritées.
- Les rôles personnalisés n’accordent pas l’accès au portail Azure AD lorsque le paramètre utilisateur « Restreindre l’accès au portail d’administration Azure AD » est défini sur Oui.
- Les inscriptions d’applications auxquels l’utilisateur a accès en utilisant les attributions de rôles s’affichent uniquement dans l’onglet « Toutes les applications » de la page Inscription d’application. Elles ne s’affichent pas dans l’onglet « Applications détenues ».

Pour plus d’informations sur les principes de base des rôles personnalisés, consultez [Vue d’ensemble des rôles personnalisés](custom-overview.md), ainsi que la façon de [créer un rôle personnalisé](custom-create.md) et d’[attribuer un rôle](custom-assign-powershell.md).

## <a name="next-steps"></a>Étapes suivantes

- [Autorisations et sous-types d’inscription d’application](custom-available-permissions.md)
- [Informations de référence sur le rôle administrateur Azure AD](permissions-reference.md)
