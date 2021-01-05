---
title: Unités administratives dans Azure Active Directory | Microsoft Docs
description: Utilisez des unités administratives pour une délégation plus précise des autorisations dans Azure Active Directory.
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
ms.service: active-directory
ms.topic: overview
ms.subservice: roles
ms.workload: identity
ms.date: 11/04/2020
ms.author: curtand
ms.reviewer: anandy
ms.custom: oldportal;it-pro;
ms.collection: M365-identity-device-management
ms.openlocfilehash: 792e8cd1e70f901385ed3b225a753024e06f2df0
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93394592"
---
# <a name="administrative-units-in-azure-active-directory"></a>Unités administratives dans Azure Active Directory

Cet article décrit les unités administratives dans Azure Active Directory (Azure AD). Une unité administrative est une ressource Azure AD qui peut être un conteneur pour d’autres ressources Azure AD. Une unité administrative peut contenir seulement des utilisateurs et des groupes.

Les unités administratives limitent les autorisations d’un rôle en fonction du service auquel il appartient au sein de l’organisation. Par exemple, vous pouvez utiliser des unités administratives pour déléguer le rôle [Administrateur du support technique](permissions-reference.md#helpdesk-administrator) aux spécialistes du support régional, pour qu’ils ne puissent s’occuper que des utilisateurs situés dans la région dont ils ont la charge.

## <a name="deployment-scenario"></a>Scénario de déploiement

Il peut être utile de restreindre l’étendue d’administration à l’aide d’unités administratives dans les organisations qui sont composées de divisions indépendantes de tout type. Prenons l’exemple d’une grande université qui se compose de nombreuses écoles autonomes (école de commerce, école d’ingénieurs, etc.). Chaque école a une équipe d’administrateurs informatiques qui contrôlent les accès, gèrent les utilisateurs et définissent les stratégies pour leur école.

Un administrateur central peut :

- Créer un rôle avec des autorisations d’administration seulement sur les utilisateurs Azure AD de l’unité administrative de l’école de commerce.
- Créer une unité administrative pour l’école de commerce.
- Affecter à l’unité administrative seulement les étudiants et le personnel de l’école de commerce.
- Ajouter l’équipe informatique de l’école de commerce au rôle avec son étendue.

## <a name="license-requirements"></a>Conditions de licence :

Pour utiliser des unités administratives, vous devez disposer d’une licence Azure Active Directory Premium pour chaque administrateur d’une unité administrative et de licences Azure Active Directory gratuites pour les membres des unités administratives. Pour plus d'informations, consultez la section [Prise en main d’Azure AD Premium](../fundamentals/active-directory-get-started-premium.md).

## <a name="manage-administrative-units"></a>Gérer des unités administratives

Vous pouvez gérer des unités administratives en utilisant le portail Azure, des applets de commande et des scripts PowerShell, ou Microsoft Graph. Pour plus d'informations, consultez les pages suivantes :

- [Créer, supprimer, remplir et ajouter des rôles aux unités administratives](admin-units-manage.md) : contient des procédures complètes.
- [Utiliser des unités administratives](/powershell/azure/active-directory/working-with-administrative-units?view=azureadps-2.0&preserve-view=true) : explique comment utiliser les unités administratives avec PowerShell.
- [Prise en charge des unités administratives par Graph](/graph/api/resources/administrativeunit?view=graph-rest-1.0&preserve-view=true) : fournit une documentation détaillée sur Microsoft Graph pour les unités administratives.

### <a name="plan-your-administrative-units"></a>Planifier vos unités administratives

Vous pouvez utiliser les unités administratives pour regrouper logiquement des ressources Azure AD. Une organisation dont les membres du service informatique sont répartis dans le monde entier peut créer des unités administratives pour définir des limites géographiques. Autre scénario : dans le cas d’une organisation multinationale composée de plusieurs sous-organisations fonctionnant de manière semi-autonome, une unité administrative peut représenter chacune de ces sous-organisations.

Les critères de création des unités administratives dépendent des exigences spécifiques d’une organisation. Les unités administratives constituent une façon courante de définir la structure des services Microsoft 365. Nous vous recommandons de préparer vos unités administratives en pensant à leur utilisation dans les services Microsoft 365. Vous pouvez tirer le meilleur parti des unités administratives quand vous pouvez associer des ressources communes à Microsoft 365 sous une unité administrative.

Vous pouvez vous attendre à ce que la création d’unités administratives au sein de l’organisation passe par les étapes suivantes :

1. **Adoption initiale** : votre organisation va commencer à créer des unités administratives en fonction de critères initiaux et le nombre d’unités administratives va augmenter à mesure que les critères sont affinés.
1. **Nettoyage** : Une fois que les critères sont définis, les unités administratives qui ne sont plus nécessaires sont supprimées.
1. **Stabilisation** : La structure organisationnelle est définie et le nombre d’unités administratives ne va pas changer de façon significative à court terme.

## <a name="currently-supported-scenarios"></a>Scénarios actuellement pris en charge

Si vous êtes administrateur général ou administrateur avec rôle privilégié, vous pouvez utiliser le portail Azure AD pour :

- Créer des unités administratives
- Ajouter des utilisateurs et des groupes membres d’unités administratives
- Attribuer au personnel informatique des rôles Administrateur limités à une unité administrative.

Ces administrateurs peuvent ensuite utiliser le centre d’administration Microsoft 365 pour la gestion de base des utilisateurs au sein de leur unité administrative. Un administrateur de groupe limité à une unité administrative peut gérer les groupes à l’aide de PowerShell, de Microsoft Graph et du centre d’administration Microsoft 365.

>[!Note]
>Seules les fonctionnalités décrites dans cette section sont disponibles dans le centre d’administration Microsoft 365. Les rôles Azure AD limités à une unité administrative ne peuvent pas bénéficier des fonctionnalités définies au niveau de l’organisation.

Les sections suivantes abordent la prise en charge des scénarios d’unité administrative.

### <a name="administrative-unit-management"></a>Gestion des unités administratives

| Autorisations |   Graph/PowerShell   | Portail Azure AD | Centre d’administration Microsoft 365 |
| --- | --- | --- | --- |
| Création et suppression des unités administratives   |    Prise en charge    |   Prise en charge   |    Non pris en charge |
| Ajout et suppression de membres individuels d’une unité administrative    |   Prise en charge    |   Prise en charge   |    Non pris en charge |
| Ajout et suppression en bloc de membres d’unité administrative avec des fichiers CSV   |    Non pris en charge     |  Prise en charge   |    Pas de prise en charge prévue |
| Affectation d’administrateurs limités à une unité administrative  |     Prise en charge    |   Prise en charge    |   Non pris en charge |
| Ajout et suppression de manière dynamique de membres d’unité administrative en fonction d’attributs | Non pris en charge | Non pris en charge | Non pris en charge

### <a name="user-management"></a>User Management

| Autorisations |   Graph/PowerShell   | Portail Azure AD | Centre d’administration Microsoft 365 |
| --- | --- | --- | --- |
| Gestion limitée à une unité administrative des propriétés, des mots de passe et des licences utilisateur   |    Prise en charge     |  Prise en charge   |   Prise en charge |
| Blocage et déblocage des connexions utilisateur limités à une unité administrative    |   Prise en charge   |    Prise en charge   |    Prise en charge |
| Gestion limitée à une unité administrative des informations d’identification d’authentification multifacteur des utilisateurs   |    Prise en charge   |   Prise en charge   |   Non pris en charge |

### <a name="group-management"></a>Gestion des groupes

| Autorisations |   Graph/PowerShell   | Portail Azure AD | Centre d’administration Microsoft 365 |
| --- | --- | --- | --- |
| Gestion limitée à une unité administrative des propriétés et des membres des groupes     |  Prise en charge   |    Prise en charge    |  Non pris en charge |
| Gestion limitée à une unité administrative des licences de groupe   |    Prise en charge  |    Prise en charge   |   Non pris en charge |

Les unités administratives appliquent l’étendue seulement aux autorisations de gestion. Elles n’empêchent pas les membres ni les administrateurs d’utiliser leurs [autorisations utilisateur par défaut](../fundamentals/users-default-permissions.md) pour parcourir d’autres utilisateurs, groupes ou ressources en dehors de l’unité administrative. Dans le centre d’administration Microsoft 365, les utilisateurs en dehors des unités administratives d’un administrateur limité sont filtrés. Toutefois, vous pouvez parcourir d’autres utilisateurs dans le portail Azure AD, PowerShell et d’autres services Microsoft.

## <a name="next-steps"></a>Étapes suivantes

- [Gestion des unités administratives](admin-units-manage.md)
- [Gérer les utilisateurs dans les unités administratives](admin-units-add-manage-users.md)
- [Gérer les groupes dans les unités administratives](admin-units-add-manage-groups.md)
- [Attribuer des rôles dont l’étendue est délimitée à une unité administrative](admin-units-assign-roles.md)
