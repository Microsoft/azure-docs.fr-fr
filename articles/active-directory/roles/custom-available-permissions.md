---
title: Autorisations de rôle personnalisées pour l’inscription d’applications - Azure AD | Microsoft Docs
description: Déléguez les autorisations de rôle d’administrateur personnalisé pour gérer les inscriptions d’applications.
services: active-directory
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
ms.openlocfilehash: d0e2f520f55b9664d2d0b039867ef7670b190fed
ms.sourcegitcommit: 0d171fe7fc0893dcc5f6202e73038a91be58da03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93377069"
---
# <a name="application-registration-permissions-for-custom-roles-in-azure-active-directory"></a>Autorisations d’inscription d’application pour les rôles personnalisés dans Azure Active Directory

Cet article contient les autorisations d’inscription d’application actuellement disponibles pour les définitions de rôle personnalisées dans Azure Active Directory (Azure AD).

## <a name="permissions-for-managing-single-tenant-applications"></a>Autorisations pour la gestion des applications à locataire unique

Lorsque vous choisissez les autorisations pour votre rôle personnalisé, vous avez la possibilité d’accorder l’accès uniquement pour gérer les applications à locataire unique. Les applications monolocataires sont uniquement disponibles pour les utilisateurs de l’organisation Azure AD dans laquelle l’application est inscrite. Les applications monolocataires sont définies comme ayant des **types de comptes pris en charge** définis sur « Comptes dans ce répertoire d’organisation uniquement ». Dans l’API Graph, les applications monolocataires ont la propriété signInAudience définie sur « AzureADMyOrg ».

Pour accorder l’accès uniquement à la gestion des applications à locataire unique, utilisez les autorisations ci-dessous avec le sous-type **applications.myOrganization**. Par exemple, microsoft.directory/applications.myOrganization/basic/update.

Consultez la [vue d’ensemble des rôles personnalisés](custom-overview.md) pour obtenir une explication des termes généraux du sous-type, de l’autorisation et du jeu de propriétés. Les informations suivantes sont spécifiques aux inscriptions d’applications.

### <a name="create-and-delete"></a>Créer et supprimer

Deux autorisations sont disponibles pour accorder la possibilité de créer des inscriptions d’applications, chacune avec un comportement différent :

#### <a name="microsoftdirectoryapplicationscreateasowner"></a>microsoft.directory/applications/createAsOwner

En attribuant cette autorisation, le créateur est ajouté en tant que premier propriétaire de l'enregistrement de l'application créée. L'enregistrement de l'application créée est comptabilisé dans le quota de 250 objets créés du créateur.

#### <a name="microsoftdirectoryapplicationscreate"></a>microsoft.directory/applications/create

En attribuant cette autorisation, le créateur n’est pas ajouté en tant que premier propriétaire de l'enregistrement de l'application créée. L'enregistrement de l'application créée est comptabilisé dans le quota de 250 objets créés du créateur. Utilisez cette autorisation avec précaution, car rien n’empêche l’intervenant de créer des inscriptions d’applications tant que le quota au niveau du répertoire n’a pas été atteint. Si les deux autorisations sont affectées, l’autorisation est prioritaire.

Si les deux autorisations sont affectées, l’autorisation /create est prioritaire. Bien que l’autorisation /createAsOwner n’ajoute pas automatiquement le créateur comme premier propriétaire, les propriétaires peuvent être spécifiés lors de la création de l’inscription de l’application lors de l’utilisation des API Graph ou des cmdlets PowerShell.

Les autorisations de création autorisent l’accès à la commande **Nouvelle inscription**.

[Ces autorisations accordent l’accès à la commande Nouvelle inscription du portail](./media/custom-available-permissions/new-custom-role.png)

Deux autorisations sont disponibles pour vous permettre de supprimer les inscriptions d’applications :

#### <a name="microsoftdirectoryapplicationsdelete"></a>microsoft.directory/applications/delete

Octroie la possibilité de supprimer des inscriptions d’applications indépendamment du sous-type ; autrement dit, des applications monolocataires et multi-locataires.

#### <a name="microsoftdirectoryapplicationsmyorganizationdelete"></a>microsoft.directory/applications.myOrganization/delete

Donne la possibilité de supprimer des inscriptions d’applications limitées à celles qui sont accessibles uniquement aux comptes de votre organisation ou à des applications monolocataires (sous-type myOrganization).

![Ces autorisations accordent l’accès à la commande Supprimer l’inscription d’application](./media/custom-available-permissions/delete-app-registration.png)

> [!NOTE]
> Lors de l’affectation d’un rôle qui contient des autorisations de création, l’attribution de rôle doit être effectuée au niveau de l’étendue du répertoire. Une autorisation de création affectée à une étendue de ressource n’accorde pas la possibilité de créer des inscriptions d’applications.

### <a name="read"></a>Lire

Tous les utilisateurs membres de l’organisation peuvent lire les informations d’inscription de l’application par défaut. Toutefois, cela n’est pas possible pour les utilisateurs invités et les principaux du service d’application. Si vous envisagez d’affecter un rôle à un utilisateur ou une application invité, vous devez inclure les autorisations de lecture appropriées.

#### <a name="microsoftdirectoryapplicationsallpropertiesread"></a>microsoft.directory/applications/allProperties/read

Capacité de lire toutes les propriétés des applications monolocataires et multilocataires, en dehors des propriétés qui ne doivent être lues en aucun cas, telles que les informations d’identification.

#### <a name="microsoftdirectoryapplicationsmyorganizationallpropertiesread"></a>microsoft.directory/applications.myOrganization/allProperties/read

Octroie les mêmes autorisations que microsoft.directory/applications/allProperties/read, mais uniquement pour les applications monolocataires.

#### <a name="microsoftdirectoryapplicationsownersread"></a>microsoft.directory/applications/owners/read

Accorde la possibilité de lire la propriété des propriétaires sur des applications monolocataires et multi-locataires Octroie l’accès à tous les champs de la page propriétaires de l’inscription de l’application :

![Ces autorisations accordent l’accès à la page propriétaires d’inscription des applications.](./media/custom-available-permissions/app-registration-owners.png)

#### <a name="microsoftdirectoryapplicationsstandardread"></a>microsoft.directory/applications/standard/read

Accorde l’accès en lecture des propriétés d’inscription d’application standard. Y sont comprises les propriétés des pages d’inscription d’application.

#### <a name="microsoftdirectoryapplicationsmyorganizationstandardread"></a>microsoft.directory/applications.myOrganization/standard/read

Octroie les mêmes autorisations que microsoft.directory/applications/standard/read, mais uniquement pour les applications monolocataire.

### <a name="update"></a>Update

#### <a name="microsoftdirectoryapplicationsallpropertiesupdate"></a>microsoft.directory/applications/allProperties/update

Capacité de mettre à jour toutes les propriétés des applications multi-locataires et mono-locataires.

#### <a name="microsoftdirectoryapplicationsmyorganizationallpropertiesupdate"></a>microsoft.directory/applications.myOrganization/allProperties/update

Octroie les mêmes autorisations que microsoft.directory/applications/allProperties/update, mais uniquement pour les applications monolocataire.

#### <a name="microsoftdirectoryapplicationsaudienceupdate"></a>microsoft.directory/applications/audience/update

Capacité de mettre à jour la propriété (signInAudience) des types de comptes pris en charge dans les applications à locataires multiples et à locataire unique.

![Cette autorisation accorde l’accès à la propriété des types de compte pris en charge dans l’inscription de l’application sur la page d’authentification](./media/custom-available-permissions/supported-account-types.png)

#### <a name="microsoftdirectoryapplicationsmyorganizationaudienceupdate"></a>microsoft.directory/applications.myOrganization/audience/update

Octroie les mêmes autorisations que microsoft.directory/applications/audience/update, mais uniquement pour les applications monolocataire.

#### <a name="microsoftdirectoryapplicationsauthenticationupdate"></a>microsoft.directory/applications/authentication/update

Possibilité de mettre à jour les propriétés URL de réponse, URL de déconnexion, flux implicite et domaine du serveur de publication sur des applications monolocataires et multi-locataires. Octroie l’accès à tous les champs de la page d’authentification de l’inscription de l’application, à l’exception des types de comptes pris en charge :

![Octroie l’accès à l’authentification d’inscription d’application, mais pas aux types de comptes pris en charge](./media/custom-available-permissions/supported-account-types.png)

#### <a name="microsoftdirectoryapplicationsmyorganizationauthenticationupdate"></a>microsoft.directory/applications.myOrganization/authentication/update

Octroie les mêmes autorisations que microsoft.directory/applications/authentication/update, mais uniquement pour les applications monolocataire.

#### <a name="microsoftdirectoryapplicationsbasicupdate"></a>microsoft.directory/applications/basic/update

Possibilité de mettre à jour le nom, le logo, l’URL de la page d’accueil, l’URL des conditions de service et les propriétés de l’URL de la déclaration de confidentialité sur les applications monolocataires et multilocataires. Octroie l’accès à tous les champs de la page de personnalisation de l’inscription d’application :

![Cette autorisation accorde l’accès à la page de personnalisation de l’inscription des applications](./media/custom-available-permissions/app-registration-branding.png)

#### <a name="microsoftdirectoryapplicationsmyorganizationbasicupdate"></a>microsoft.directory/applications.myOrganization/basic/update

Octroie les mêmes autorisations que microsoft.directory/applications/basic/update, mais uniquement pour les applications monolocataire.

#### <a name="microsoftdirectoryapplicationscredentialsupdate"></a>microsoft.directory/applications/credentials/update

Possibilité de mettre à jour les propriétés des certificats et des secrets client sur des applications monolocataires et multi-locataires. Octroie l’accès à tous les champs de la page certificats et secrets d’inscription d’application :

![Cette autorisation accorde l’accès à la page certificats et secrets d’inscription d’application](./media/custom-available-permissions/app-registration-secrets.png)

#### <a name="microsoftdirectoryapplicationsmyorganizationcredentialsupdate"></a>microsoft.directory/applications.myOrganization/credentials/update

Octroie les mêmes autorisations que microsoft.directory/applications/credentials/update, mais uniquement pour les applications à locataire unique.

#### <a name="microsoftdirectoryapplicationsownersupdate"></a>microsoft.directory/applications/owners/update

Possibilité de mettre à jour la propriété du propriétaire sur les applications monolocataires et multi-locataires. Octroie l’accès à tous les champs de la page propriétaires de l’inscription de l’application :

![Ces autorisations accordent l’accès à la page propriétaires d’inscription des applications.](./media/custom-available-permissions/app-registration-owners.png)

#### <a name="microsoftdirectoryapplicationsmyorganizationownersupdate"></a>microsoft.directory/applications.myOrganization/owners/update

Octroie les mêmes autorisations que microsoft.directory/applications/owners/update, mais uniquement pour les applications monolocataire.

#### <a name="microsoftdirectoryapplicationspermissionsupdate"></a>microsoft.directory/applications/permissions/update

Possibilité de mettre à jour les autorisations déléguées, les autorisations d’application, les applications clientes autorisées, les autorisations requises et d’accorder des propriétés de consentement sur des applications monolocataires et multi-locataires. N’accorde pas la possibilité d’effectuer un consentement. Octroie l’accès à tous les champs des autorisations de l’API d’inscription d’application et expose une page API :

![Cette autorisation accorde l’accès à la page Autorisations de l’API d’inscription d’application](./media/custom-available-permissions/app-registration-api-permissions.png)

![Cette autorisation accorde l’accès à la page Exposer une API de l’inscription d’application](./media/custom-available-permissions/app-registration-expose-api.png)

#### <a name="microsoftdirectoryapplicationsmyorganizationpermissionsupdate"></a>microsoft.directory/applications.myOrganization/permissions/update

Octroie les mêmes autorisations que microsoft.directory/applications/permissions/update, mais uniquement pour les applications monolocataire.

## <a name="required-license-plan"></a>Plan de licence obligatoire

[!INCLUDE [License requirement for using custom roles in Azure AD](../../../includes/active-directory-p1-license.md)]

## <a name="next-steps"></a>Étapes suivantes

- Créer des rôles personnalisés à l’aide du [portail Azure, d’Azure AD PowerShell et de l’API Graph](custom-create.md)
- [Afficher les attributions d’un rôle personnalisé](custom-view-assignments.md)
