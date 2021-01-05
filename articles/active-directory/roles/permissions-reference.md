---
title: Description des rôles et des autorisations d’Azure AD - Azure Active Directory | Microsoft Docs
description: Le rôle de l’administrateur peut servir à ajouter des utilisateurs, attribuer des rôles d’administrateur, réinitialiser les mots de passe utilisateur, gérer les licences utilisateur et les domaines, etc.
services: active-directory
author: curtand
manager: daveba
search.appverid: MET150
ms.service: active-directory
ms.workload: identity
ms.subservice: roles
ms.topic: reference
ms.date: 11/05/2020
ms.author: curtand
ms.reviewer: vincesm
ms.custom: it-pro, fasttrack-edit
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1d634978333e2b833324228fb0908cfba61a786d
ms.sourcegitcommit: fec60094b829270387c104cc6c21257826fccc54
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96919740"
---
# <a name="administrator-role-permissions-in-azure-active-directory"></a>Autorisations des rôles d’administrateur dans Azure Active Directory

À l’aide d’Azure Active Directory (Azure AD), vous pouvez désigner des administrateurs limités pour gérer des tâches liées aux identités dans des rôles moins privilégiés. Les administrateurs peuvent se voir attribuer des responsabilités comme l’ajout ou le changement d’utilisateurs, l’attribution de rôles d’administrateur, la réinitialisation des mots de passe utilisateur, la gestion des licences utilisateur et la gestion des noms de domaine. Dans Azure AD, il n’est possible de modifier les [autorisations d’utilisateur par défaut](../fundamentals/users-default-permissions.md) que dans les paramètres utilisateur.

## <a name="limit-use-of-global-administrator"></a>Limiter l’utilisation de l’administrateur général

Les utilisateurs qui sont affectés au rôle d’administrateur d’entreprise peuvent lire et modifier chaque paramètre d’administration de votre organisation Azure AD. Par défaut, quand un utilisateur s’inscrit à un service cloud Microsoft, un locataire Azure AD est créé, et l’utilisateur est promu membre du rôle Administrateur général. Lorsque vous ajoutez un abonnement à un locataire existant, le rôle Administrateur général ne vous est pas attribué. Seuls les administrateurs généraux et les administrateurs disposant d'un rôle privilégié peuvent déléguer des rôles d'administrateur. Pour limiter les risques pour votre entreprise, nous vous recommandons d’attribuer ce rôle à un nombre restreint de personnes de votre organisation.

Nous vous recommandons d'attribuer ce rôle à moins de cinq personnes au sein de votre organisation. Si plus de cinq personnes disposent du rôle d'administrateur général au sein de votre organisation, voici quelques façons de réduire l'utilisation de ce rôle.

### <a name="find-the-role-you-need"></a>Trouvez le rôle dont vous avez besoin

Si vous ne trouvez pas le rôle dont vous avez besoin dans une liste composée de nombreux rôles, Azure AD peut vous montrer des sous-ensembles des rôles en fonction des catégories de rôles. Consultez notre nouveau filtre de **Type** pour [Rôles et administrateurs Azure AD](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RolesAndAdministrators) afin de ne montrer que les rôles du type sélectionné.

### <a name="a-role-exists-now-that-didnt-exist-when-you-assigned-the-global-administrator-role"></a>Il existe maintenant un rôle qui n’existait pas lorsque vous avez attribué le rôle Administrateur général

Il est possible qu’un ou plusieurs rôles aient été ajoutés à Azure AD pour fournir des autorisations plus granulaires qui n’étaient pas en option lorsque vous avez attribué à certains utilisateurs le rôle d’administrateur d’entreprise. Au fil du temps, nous déployons des rôles supplémentaires pour des tâches que seul le rôle d’administrateur d’entreprise pouvait accomplir auparavant. Vous pouvez voir ces éléments reflétés dans les [Rôles disponibles](#available-roles) suivants.

## <a name="assign-or-remove-administrator-roles"></a>Attribution ou suppression de rôles d’administrateur

Pour découvrir comment attribuer des rôles d’administrateur à un utilisateur dans Azure Active Directory, consultez l’article [Afficher et attribuer des rôles d’administrateur dans Azure Active Directory](manage-roles-portal.md).

> [!Note]
> Si vous disposez d’une licence Azure AD Premium P2 et que vous êtes déjà un utilisateur Privileged Identity Management (PIM), toutes les tâches de gestion des rôles sont effectuées dans PIM et non dans Azure AD.
>
> ![Rôles Azure AD gérés dans PIM pour les utilisateurs qui utilisent déjà PIM et qui disposent d’une licence Premium P2](./media/permissions-reference/pim-manages-roles-for-p2.png)

## <a name="available-roles"></a>Rôles disponibles

Les rôles d’administrateur disponibles sont les suivants :

### <a name="application-administrator"></a>[Administrateur d’application](#application-administrator-permissions)

Les utilisateurs dans ce rôle peuvent créer et gérer tous les aspects des applications d’entreprise, des inscriptions d’application et des paramètres de proxy d’application. Notez que les utilisateurs affectés à ce rôle ne sont pas ajoutés en tant que propriétaires lorsque des inscriptions d’applications ou des applications d’entreprise sont créées.

Ce rôle permet également de _donner son consentement_ pour des autorisations déléguées et des autorisations d’application, à l’exception des autorisations sur l'API Microsoft Graph.

> [!IMPORTANT]
> Cette exception signifie que vous pouvez toujours accepter les autorisations pour _d’autres applications_ (par exemple les applications Microsoft ou les applications que vous avez inscrites), mais pas les autorisations sur Azure AD proprement dit. Vous pouvez toujours _demander_ ces autorisations dans le cadre de l’inscription de l’application, mais l’_octroi_ de ces autorisations (c’est-à-dire le consentement) requiert un administrateur Azure AD. Cela signifie qu’un utilisateur malveillant ne peut pas élever facilement ses autorisations, par exemple en créant une application capable d’écrire dans le répertoire entier et en lui donnant son consentement, puis, par le biais de des autorisations de cette application, en s’élevant lui-même pour devenir administrateur global.
>
>Ce rôle permet de gérer des informations d’identification d’application. Les utilisateurs auxquels ce rôle a été attribué peuvent ajouter des informations d’identification à une application et les utiliser pour emprunter l’identité de l’application. Si l’identité de l’application a obtenu l’accès à une ressource, telle que la possibilité de créer ou de mettre à jour un utilisateur ou d’autres objets, un utilisateur auquel ce rôle a été attribué peut effectuer ces actions en empruntant l’identité de l’application. Cette capacité à emprunter l’identité de l’application peut correspondre une élévation de privilèges que l’utilisateur réalise dans ses attributions de rôle. Il est important de comprendre que l’affectation d’un utilisateur au rôle Administrateur d’applications lui donne la possibilité d’emprunter l’identité d’une application.

### <a name="application-developer"></a>[Développeur d’applications](#application-developer-permissions)

Les utilisateurs dans ce rôle peuvent créer des inscriptions d’application quand le paramètre « Les utilisateurs peuvent inscrire des applications » est défini sur Non. Ce rôle octroie aussi l’autorisation de donner son consentement en son propre nom lorsque le paramètre « Les utilisateurs peuvent autoriser les applications à accéder aux données de l’entreprise en leur nom » est défini sur Non. Les utilisateurs affectés à ce rôle sont ajoutés en tant que propriétaires lorsque des inscriptions d’applications ou des applications d’entreprise sont créées.

### <a name="authentication-administrator"></a>[Administrateur d’authentification](#authentication-administrator-permissions)

Les utilisateurs dotés de ce rôle peuvent définir ou réinitialiser les informations d’identification sans mot de passe pour certains utilisateurs et peuvent mettre à jour les mots de passe de tous les utilisateurs. Les administrateurs d’authentification peuvent obliger les utilisateurs qui ne sont pas administrateurs ou sont attribués à certains rôles à se réinscrire avec des informations d’identification sans mot de passe existantes (par exemple, MFA, FIDO) et peuvent également révoquer la **mémorisation de l’authentification multifacteur sur l’appareil**, ce qui permet de demander une authentification multifacteur lors de la prochaine connexion. Ces actions s’appliquent uniquement aux utilisateurs qui ne sont pas administrateurs ou qui se voient attribuer un ou plusieurs des rôles suivants :

* Administrateur d’authentification
* Lecteurs de répertoires
* Inviteur
* Lecteur du Centre de messages
* Lecteur de rapports

Le rôle [Administrateur d’authentification privilégié](#privileged-authentication-administrator) a l’autorisation de forcer la réinscription et l’authentification multifacteur pour tous les utilisateurs.

> [!IMPORTANT]
> Les utilisateurs auxquels ce rôle est assigné peuvent changer les informations d’identification des personnes susceptibles d’avoir accès à des informations sensibles ou privées ou à des configurations critiques à l’intérieur et à l’extérieur d’Azure Active Directory. Changer les informations d’identification d’un utilisateur peut signifier la capacité d’assumer l’identité et les autorisations de cet utilisateur. Par exemple :
>
>* Les propriétaires d’inscription d’application et d’application d’entreprise, qui peuvent gérer les informations d’identification des applications qu’ils possèdent. Ces applications peuvent disposer d’autorisations privilégiées dans Azure AD et ailleurs qui ne sont pas accordées aux administrateurs d’authentification. Par ce biais, un administrateur d’authentification peut être en mesure d’assumer l’identité d’un propriétaire d’application, puis d’assumer l’identité d’une application privilégiée en mettant à jour les identifiants de celle-ci.
>* Propriétaires d’abonnement Azure, qui peuvent avoir accès à des informations confidentielles ou privées ou à une configuration critique dans Azure.
>* Propriétaires de groupe de sécurité et de groupe Microsoft 365, qui peuvent gérer l’appartenance à un groupe. Ces groupes peuvent accorder l’accès à des informations sensibles ou privées ou à une configuration critique dans Azure AD et ailleurs.
>* Administrateurs d’autres services en dehors d’Azure AD, tels qu’Exchange Online, le Centre de sécurité et de conformité Office et les systèmes de ressources humaines.
>* Les non-administrateurs comme les cadres supérieurs, les conseillers juridiques et les employés des ressources humaines qui peuvent avoir accès à des informations sensibles ou privées.

### <a name="azure-devops-administrator"></a>[Administrateur Azure DevOps](#azure-devops-administrator-permissions)

Les utilisateurs disposant de ce rôle peuvent gérer la stratégie Azure DevOps pour restreindre la création d’une nouvelle organisation Azure DevOps à un ensemble d’utilisateurs ou de groupes configurables. Les utilisateurs auxquels ce rôle est attribué peuvent gérer cette stratégie par le biais de n’importe quelle organisation Azure DevOps prise en charge par l’organisation Azure AD de l’entreprise. Ce rôle n’accorde aucune autre autorisation spécifique à Azure DevOps (par exemple, administrateur de collections de projets) au sein de l’une des organisations Azure DevOps prises en charge par l’organisation Azure AD de l’entreprise.

Toutes les stratégies Azure DevOps d’entreprise peuvent être gérées par les utilisateurs de ce rôle.

### <a name="azure-information-protection-administrator"></a>[Administrateur Azure Information Protection](#azure-information-protection-administrator-permissions)

Les utilisateurs avec ce rôle ont toutes les autorisations dans le service Azure Information Protection. Ce rôle permet de configurer les étiquettes pour la stratégie Azure Information Protection, de gérer les modèles de protection et d’activer la protection. Il n’octroie aucune autorisation dans Identity Protection Center, Privileged Identity Management, Monitor Microsoft 365 Service Health ni dans le Centre de sécurité et de conformité Office 365.

### <a name="b2c-ief-keyset-administrator"></a>[Administrateur de jeux de clés B2C IEF](#b2c-ief-keyset-administrator-permissions)

L’utilisateur peut créer et gérer des clés et secrets de stratégie pour le chiffrement des jetons, la signatures des jetons et le chiffrement/déchiffrement des revendications.  En ajoutant de nouvelles clés aux conteneurs de clés existants, cet administrateur limité peut substituer des secrets en fonction des besoins sans affecter les applications existantes.  Cet utilisateur peut voir le contenu complet de ces secrets et leurs dates d’expiration, même après leur création.

> [!IMPORTANT]
> Il s’agit d’un rôle sensible.  Le rôle d’administrateur de jeux de clés doit être soigneusement audité et attribué avec précaution pendant les périodes de préproduction et de production.

### <a name="b2c-ief-policy-administrator"></a>[Administrateur de stratégies B2C IEF](#b2c-ief-policy-administrator-permissions)

Les utilisateurs assignés à ce rôle ont la possibilité de créer, lire, mettre à jour et supprimer toutes les stratégies personnalisées dans Azure AD B2C, et donc de contrôler totalement la plateforme Identity Experience Framework dans l’organisation Azure AD B2C concernée. En modifiant les stratégies, cet utilisateur peut établir une fédération directe avec des fournisseurs d’identité externes, modifier le schéma d’annuaire, modifier tout le contenu visible par l’utilisateur (HTML, CSS et JavaScript), modifier les exigences liées à une authentification, créer des utilisateurs, envoyer des données utilisateur à des systèmes externes, notamment des migrations complètes, et modifier toutes les informations utilisateur, notamment des champs sensibles comme les mots de passe et les numéros de téléphone. À l’inverse, ce rôle ne peut pas modifier les clés de chiffrement ou modifier les secrets utilisés pour la fédération dans l’organisation.

> [!IMPORTANT]
> L’administrateur de stratégies B2C IEF est un rôle très sensible qui doit être attribué de manière très limitée pour les organisations en production.  Les activités de ces utilisateurs doivent faire l’objet d’un audit précis, en particulier pour les organisations en production.

### <a name="billing-administrator"></a>[Administrateur de facturation](#billing-administrator-permissions)

Effectue les achats, gère les abonnements, gère les tickets de support et supervise l’intégrité du service.

### <a name="cloud-application-administrator"></a>[Administrateur d’application cloud](#cloud-application-administrator-permissions)

Les utilisateurs dans ce rôle ont les mêmes autorisations que celles du rôle Administrateur d’application, sans la possibilité de gérer le proxy d’application. Ce rôle permet de créer et de gérer tous les aspects des applications d’entreprise et des inscriptions d’applications. Ce rôle permet également de donner son consentement pour des autorisations déléguées et des autorisations d’application, sauf pour Microsoft Graph et Azure AD Graph. Les utilisateurs affectés à ce rôle ne sont pas ajoutés en tant que propriétaires lorsque des inscriptions d’applications ou des applications d’entreprise sont créées.

> [!IMPORTANT]
> Ce rôle permet de gérer des informations d’identification d’application. Les utilisateurs auxquels ce rôle a été attribué peuvent ajouter des informations d’identification à une application et les utiliser pour emprunter l’identité de l’application. Si l’identité de l’application a obtenu l’accès à une ressource, telle que la possibilité de créer ou de mettre à jour un utilisateur ou d’autres objets, un utilisateur auquel ce rôle a été attribué peut effectuer ces actions en empruntant l’identité de l’application. Cette capacité à emprunter l’identité de l’application peut correspondre une élévation de privilèges que l’utilisateur réalise dans ses attributions de rôle. Il est important de comprendre que l’affectation d’un utilisateur au rôle Administrateur d’applications cloud lui donne la possibilité d’emprunter l’identité d’une application.


### <a name="cloud-device-administrator"></a>[Administrateur d’appareil cloud](#cloud-device-administrator-permissions)

Les utilisateurs dans ce rôle peuvent activer, désactiver et supprimer des appareils dans Azure AD, et lire des clés BitLocker Windows 10 (le cas échéant) dans le portail Azure. Ce rôle ne permet pas de gérer d’autres propriétés sur l’appareil.

### <a name="compliance-administrator"></a>[Administrateur de conformité](#compliance-administrator-permissions)

Les utilisateurs avec ce rôle disposent des autorisations pour gérer les fonctionnalités liées à la conformité dans le centre de conformité Microsoft 365, le centre d'administration Microsoft 365, Azure et le Centre de sécurité et conformité Office 365. Les personnes responsables peuvent également gérer toutes les fonctionnalités dans le centre d’administration Exchange et le centre d’administration Teams et Skype Entreprise, et créer des tickets de support pour Azure et Microsoft 365. Pour plus d’informations, consultez [À propos des rôles d’administrateur Microsoft 365](https://support.office.com/article/About-Office-365-admin-roles-da585eea-f576-4f55-a1e0-87090b6aaa9d).

Dans | Peut
----- | ----------
[Centre de conformité Microsoft 365](https://protection.office.com) | Protéger et gérer les données de votre organisation dans les services Microsoft 365<br>Gérer les alertes de conformité
[Compliance Manager](/office365/securitycompliance/meet-data-protection-and-regulatory-reqs-using-microsoft-cloud) | Suivre, affecter et vérifier les activités de conformité réglementaire de votre organisation
[Centre de sécurité et conformité Office 365](https://support.office.com/article/About-Office-365-admin-roles-da585eea-f576-4f55-a1e0-87090b6aaa9d) | Gérer la gouvernance des données<br>Procéder à l'examen juridique des données<br>Gérer une requête d'objet de données<br><br>Ce rôle a les mêmes autorisations que le [RoleGroup Administrateur de conformité](/microsoft-365/security/office-365-security/permissions-in-the-security-and-compliance-center#permissions-needed-to-use-features-in-the-security--compliance-center) dans le contrôle d’accès en fonction du rôle du Centre de sécurité et conformité Office 365.
[Intune](/intune/role-based-access-control) | Afficher toutes les données d’audit Intune
[Cloud App Security](/cloud-app-security/manage-admins) | Dispose d'autorisations en lecture seule et peut gérer les alertes<br>Peut créer et modifier les stratégies de fichier et autoriser des actions de gouvernance de fichier<br>Peut afficher tous les rapports intégrés sous Gestion des données

### <a name="compliance-data-administrator"></a>[Administrateur des données de conformité](#compliance-data-administrator-permissions)

Les utilisateurs dotés de ce rôle sont autorisés à suivre les données dans le centre de conformité Microsoft 365, le centre d’administration Microsoft 365 et Azure. Les utilisateurs peuvent également suivre les données de conformité dans le centre d’administration Exchange, le Gestionnaire de compatibilité, le centre d’administration Teams et Skype Entreprise, et créer des tickets de support pour Azure et Microsoft 365. [Cette documentation](/microsoft-365/security/office-365-security/permissions-in-the-security-and-compliance-center#permissions-needed-to-use-features-in-the-security--compliance-center) contient des détails sur les différences entre Administrateur de conformité et Administrateur des données de conformité.

Dans | Peut
----- | ----------
[Centre de conformité Microsoft 365](https://protection.office.com) | Superviser les stratégies de conformité au sein des services Microsoft 365<br>Gérer les alertes de conformité
[Compliance Manager](/office365/securitycompliance/meet-data-protection-and-regulatory-reqs-using-microsoft-cloud) | Suivre, affecter et vérifier les activités de conformité réglementaire de votre organisation
[Centre de sécurité et conformité Office 365](https://support.office.com/article/About-Office-365-admin-roles-da585eea-f576-4f55-a1e0-87090b6aaa9d) | Gérer la gouvernance des données<br>Procéder à l'examen juridique des données<br>Gérer une requête d'objet de données<br><br>Ce rôle a les mêmes autorisations que le [RoleGroup Administrateur de données de conformité](/microsoft-365/security/office-365-security/permissions-in-the-security-and-compliance-center#permissions-needed-to-use-features-in-the-security--compliance-center) dans le contrôle d’accès en fonction du rôle du Centre de sécurité et conformité Office 365.
[Intune](/intune/role-based-access-control) | Afficher toutes les données d’audit Intune
[Cloud App Security](/cloud-app-security/manage-admins) | Dispose d'autorisations en lecture seule et peut gérer les alertes<br>Peut créer et modifier les stratégies de fichier et autoriser des actions de gouvernance de fichier<br>Peut afficher tous les rapports intégrés sous Gestion des données

### <a name="conditional-access-administrator"></a>[Administrateur de l’accès conditionnel](#conditional-access-administrator-permissions)

Les utilisateurs avec ce rôle ont la possibilité de gérer les paramètres d’accès conditionnel Azure Active Directory.

### <a name="customer-lockbox-access-approver"></a>[Approbateur d’accès Customer Lockbox](#customer-lockbox-access-approver-permissions)

Gère les [requêtes Customer Lockbox](/office365/admin/manage/customer-lockbox-requests) dans votre organisation. Il reçoit par e-mail des notifications ayant trait aux requêtes Customer Lockbox, et peut approuver ou refuser ces requêtes depuis le centre d'administration Microsoft 365. Il peut activer ou désactiver la fonctionnalité Customer Lockbox. Seuls les administrateurs généraux peuvent réinitialiser les mots de passe des personnes dotées de ce rôle.

### <a name="desktop-analytics-administrator"></a>[Administrateur Desktop Analytics](#desktop-analytics-administrator-permissions)


Les utilisateurs avec ce rôle peuvent gérer Desktop Analytics et les services de stratégie et de personnalisation Office. Pour Desktop Analytics, cela inclut la possibilité d’afficher l’inventaire des biens, de créer des plans de déploiement, d’afficher l’état du déploiement et de l’intégrité. Pour le service de stratégie et personnalisation Office, ce rôle permet aux utilisateurs de gérer les stratégies Office.

### <a name="device-administrators"></a>[Administrateurs d’appareils](#device-administrators-permissions)

Ce rôle peut uniquement être attribué comme administrateur local supplémentaire dans [Paramètres de l’appareil](https://aad.portal.azure.com/#blade/Microsoft_AAD_IAM/DevicesMenuBlade/DeviceSettings/menuId/). Les utilisateurs dotés de ce rôle deviennent des administrateurs de l'ordinateur local pour tous les appareils Windows 10 qui sont joints à Azure Active Directory. Ils ne peuvent pas gérer des objets appareil dans Azure Active Directory.

### <a name="directory-readers"></a>[Lecteurs de répertoire](#directory-readers-permissions)

Les utilisateurs de ce rôle peuvent lire des informations de base relatives aux répertoires. Ce rôle doit être utilisé pour :

* Accorder à un ensemble spécifique d’utilisateurs invités un accès en lecture au lieu de l’accorder à tous les utilisateurs invités.
* Accorder à un ensemble spécifique d’utilisateurs non-administrateurs l’accès au portail Azure quand l’option « Restreindre l’accès au portail Azure AD aux seuls administrateurs » a la valeur « Oui ».
* Accorder un accès aux principaux de service au répertoire quand Directory.Read.All n’est pas possible.

### <a name="directory-synchronization-accounts"></a>[Comptes de synchronisation d’annuaires](#directory-synchronization-accounts-permissions)

Ne pas utiliser. Ce rôle est automatiquement attribué au service Azure AD Connect et n’est pas prévu ni pris en charge pour une autre utilisation.

### <a name="directory-writers"></a>[Enregistreurs de répertoire](#directory-writers-permissions)
Les utilisateurs de ce rôle peuvent lire et mettre à jour les informations de base des utilisateurs, des groupes et des principaux de service. Attribuez ce rôle uniquement aux applications qui ne prennent pas en charge [l’infrastructure de consentement](../develop/quickstart-register-app.md). Vous ne devez pas l’affecter à tous les utilisateurs.

### <a name="dynamics-365-administrator--crm-administrator"></a>[Administrateur Dynamics 365/Administrateur CRM](#crm-service-administrator-permissions)

Les utilisateurs auxquels ce rôle est assigné disposent d’autorisations générales dans Microsoft Dynamics 365 Online, lorsque le service est présent. Ils ont aussi la possibilité de gérer les tickets de support et de surveiller l’intégrité du service. Pour plus d’informations, consultez la page [Utiliser le rôle d’administrateur de service pour gérer votre organisation Azure AD](/dynamics365/customer-engagement/admin/use-service-admin-role-manage-tenant).

> [!NOTE]
> Dans l’API Microsoft Graph et Azure AD PowerShell, ce rôle est identifié comme « Administrateur de service Dynamics 365 ». Il est appelé « Administrateur Dynamics 365 » dans le [portail Azure](https://portal.azure.com).

### <a name="exchange-administrator"></a>[Administrateur Exchange](#exchange-service-administrator-permissions)

Les utilisateurs avec ce rôle ont des autorisations générales dans Microsoft Exchange Online, quand le service est présent. Ils ont aussi la possibilité de créer et de gérer tous les groupes Microsoft 365, de gérer les tickets de support et de surveiller l’état d’intégrité des services. Plus d’informations sur les [Rôles d’administrateur dans Microsoft 365](https://support.office.com/article/About-Office-365-admin-roles-da585eea-f576-4f55-a1e0-87090b6aaa9d).

> [!NOTE]
> Dans l’API Microsoft Graph et Azure AD PowerShell, ce rôle est identifié comme « Administrateur de service Exchange ». Il est appelé « Administrateur Exchange » dans le [portail Azure](https://portal.azure.com). Il est « Administrateur Exchange en ligne » dans le [centre d’administration Exchange](https://go.microsoft.com/fwlink/p/?LinkID=529144).


### <a name="external-id-user-flow-administrator"></a>[Administrateur de flux d’utilisateurs ID externe](#external-id-user-flow-administrator-permissions)

Les utilisateurs dotés de ce rôle peuvent créer et gérer des flux d’utilisateurs (également appelés stratégies « intégrées ») sur le Portail Azure. Ces utilisateurs peuvent personnaliser le contenu HTML/CSS/JavaScript, modifier les conditions d’authentification multifacteur, sélectionner des revendications dans le jeton, gérer les connecteurs d’API et configurer les paramètres de session pour tous les flux d’utilisateurs de l’organisation Azure AD. En revanche, ce rôle ne prévoit pas la possibilité de consulter les données utilisateur ni d’apporter des modifications aux attributs inclus dans le schéma de l’organisation. La modification des stratégies Identity Experience Framework (également appelées stratégies personnalisées) sort également de la portée de ce rôle.

### <a name="external-id-user-flow-attribute-administrator"></a>[Administrateur d’attribut de flux d’utilisateurs ID externe](#external-id-user-flow-attribute-administrator-permissions)

Les utilisateurs ayant ce rôle peuvent ajouter ou supprimer des attributs personnalisés disponibles pour tous les flux d’utilisateurs dans l’organisation Azure AD.  Ainsi, ils peuvent modifier ou ajouter de nouveaux éléments au schéma d’utilisateur final, influer sur le comportement de tous les flux d’utilisateurs et entraîner indirectement des modifications des données demandées aux utilisateurs finaux et envoyées sous forme de revendications aux applications.  Ce rôle ne peut pas modifier des flux d’utilisateurs.

### <a name="external-identity-provider-administrator"></a>[Administrateur de fournisseurs d’identité externes](#external-identity-provider-administrator-permissions)

Cet administrateur gère la fédération entre les organisations Azure AD et les fournisseurs d’identité externes.  Avec ce rôle, les utilisateurs peuvent ajouter de nouveaux fournisseurs d’identité et configurer tous les paramètres disponibles (par exemple, chemin d’authentification, ID de service, conteneurs de clés attribués).  Cet utilisateur peut permettre à l’organisation d’approuver des authentifications de fournisseurs d’identité externes.  L’impact produit sur les expériences d’utilisateur final varie selon le type d’organisation :

* Organisations Azure AD pour les employés et les partenaires : L’ajout d’une fédération (par exemple, avec Gmail) impacte immédiatement toutes les invitations d’invités qui ne sont pas encore utilisées. Consultez [Ajout de Google comme fournisseur d’identité pour les utilisateurs invités B2B](../external-identities/google-federation.md).
* Organisations Azure Active Directory B2C : L’ajout d’une fédération (par exemple, avec Facebook ou une autre organisation Azure AD) n’impacte pas immédiatement les flux d’utilisateurs finaux tant que le fournisseur d’identité n’est pas ajouté comme option dans un flux d’utilisateurs (aussi appelé stratégie intégrée). Consultez [Configuration d’un compte Microsoft comme fournisseur d’identité](../../active-directory-b2c/identity-provider-microsoft-account.md) pour obtenir un exemple.  Pour modifier des flux d’utilisateurs, le rôle limité d’« administrateur de flux d’utilisateurs B2C » est nécessaire.

### <a name="global-administrator--company-administrator"></a>[Administrateur général ou Administrateur d’entreprise](#company-administrator-permissions)

Les utilisateurs avec ce rôle ont accès à toutes les fonctionnalités d’administration dans Azure Active Directory, ainsi qu’aux services qui utilisent des identités Azure Active Directory comme le centre de sécurité Microsoft 365, le centre de conformité Microsoft 365, Exchange Online, SharePoint Online et Skype Entreprise Online. Par ailleurs, les administrateurs généraux peuvent [élever leur accès](../../role-based-access-control/elevate-access-global-admin.md) pour gérer tous les abonnements et groupes d’administration Azure. Ils obtiennent ainsi un accès complet à toutes les ressources Azure à l’aide du locataire Azure AD correspondant. La personne qui s’inscrit pour l’organisation Azure AD devient un administrateur général. Une entreprise peut comprendre plusieurs administrateurs généraux. Les administrateurs généraux peuvent réinitialiser le mot de passe des utilisateurs et de tous les autres administrateurs.

> [!NOTE]
> Dans l’API Microsoft Graph et Azure AD PowerShell, ce rôle est identifié comme « Administrateur d’entreprise ». Il est « Administrateur général » dans le [portail Azure](https://portal.azure.com).
>
>

### <a name="global-reader"></a>[Lecteur général](#global-reader-permissions)

Les utilisateurs affectés à ce rôle peuvent lire les paramètres et les informations d’administration entre les services Microsoft 365, mais ne peuvent pas entreprendre d’actions de gestion. Le lecteur général est l’équivalent en lecture seule de l’administrateur général. Affectez un rôle de lecteur général plutôt qu’administrateur général pour la planification, les audits ou les investigations. Utilisez le lecteur global conjointement avec d’autres rôles d’administrateur limités comme celui de l’administrateur Exchange pour faciliter le travail sans affecter le rôle d’administrateur général. Le lecteur général fonctionne avec le centre d’administration Microsoft 365, le centre d’administration Exchange, le centre d’administration SharePoint, le centre d’administration Teams, le centre de sécurité, le centre de conformité, le centre d’administration Azure AD et le centre d’administration de gestion des appareils.

> [!NOTE]
> Le rôle de lecteur général comporte des restrictions pour le moment.
>
>- [Centre d’administration OneDrive](https://admin.onedrive.com/) : Le Centre d’administration OneDrive ne prend pas en charge le rôle de lecteur général.
>- [Centre d’administration M365](https://admin.microsoft.com/Adminportal/Home#/homepage) : Le lecteur général ne peut pas lire les demandes du Customer Lockbox. L’onglet **Demandes du Customer Lockbox** ne figure pas sous **Support** dans le volet gauche du centre d’administration M365.
>- [Centre de sécurité et de conformité Office](https://sip.protection.office.com/homepage) - Le lecteur général ne peut pas lire les journaux d'audit SCC, effectuer une recherche de contenu ou voir le Degré de sécurisation.
>- [Centre d’administration Teams](https://admin.teams.microsoft.com) : Le lecteur général ne peut pas lire le **cycle de vie Teams**, les **analyses et rapports**, la **gestion des appareils téléphoniques IP** et le **catalogue d’applications**.
>- [Privileged Access Management (PAM) ](/office365/securitycompliance/privileged-access-management-overview) ne prend pas en charge le rôle de lecteur général.
>- [Azure Information Protection](/azure/information-protection/what-is-information-protection) : le lecteur général est pris en charge uniquement pour la [création centralisée de rapports](/azure/information-protection/reports-aip) et lorsque votre organisation Azure AD n’est pas sur la [plateforme d’étiquetage unifiée](/azure/information-protection/faqs#how-can-i-determine-if-my-tenant-is-on-the-unified-labeling-platform).
>
> Ces fonctionnalités sont actuellement en cours de développement.
>

### <a name="groups-administrator"></a>[Administrateur de groupes](#groups-administrator-permissions)

Les utilisateurs de ce rôle peuvent créer/gérer des groupes et ses paramètres, comme les stratégies d’attribution de noms et d’expiration. Il est important de comprendre que l’affectation d’un utilisateur à ce rôle lui donne la possibilité de gérer tous les groupes au sein de l’organisation dans diverses charges de travail telles que Teams, SharePoint et Yammer en plus d’Outlook. De même, l’utilisateur peut gérer les différents paramètres des groupes à travers divers portails d’administration, comme le centre d’administration Microsoft, le portail Azure, ainsi que des charges de travail spécifiques telles que Teams et les centres d’administration SharePoint.

### <a name="guest-inviter"></a>[Inviteur d’invités](#guest-inviter-permissions)

les utilisateurs auxquels ce rôle est assigné peuvent gérer les invitations d’utilisateurs invités Azure Active Directory B2B lorsque le paramètre utilisateur **Les membres peuvent inviter** est défini sur Non. Pour plus d’informations sur B2B Collaboration, consultez la page [À propos d’Azure AD B2B Collaboration](../external-identities/what-is-b2b.md). Il n’inclut pas d’autres autorisations.

### <a name="helpdesk-administrator"></a>[Administrateur du support technique](#helpdesk-administrator-permissions)

Les utilisateurs avec ce rôle peuvent changer des mots de passe, rendre non valides les jetons d’actualisation, gérer les demandes de service et superviser l’intégrité du service. L’invalidation d’un jeton d’actualisation oblige l’utilisateur à se reconnecter. Les administrateurs du support peuvent réinitialiser les mots de passe et invalider les jetons d’actualisation des autres utilisateurs qui ne sont pas administrateurs ni dotés des rôles suivants uniquement :

* Lecteurs de répertoires
* Inviteur
* Administrateur du support technique
* Lecteur du Centre de messages
* Administrateur de mots de passe
* Lecteur de rapports

> [!IMPORTANT]
> Les utilisateurs auxquels ce rôle est assigné peuvent changer les mots de passe des personnes susceptibles d’avoir accès à des informations sensibles ou privées ou à des configurations critiques à l’intérieur et à l’extérieur d’Azure Active Directory. Changer le mot de passe d’un utilisateur peut signifier la capacité d’assumer l’identité et les autorisations de cet utilisateur. Par exemple :
>
>- Les propriétaires d’inscription d’application et d’application d’entreprise, qui peuvent gérer les informations d’identification des applications qu’ils possèdent. Ces applications peuvent disposer d’autorisations privilégiées dans Azure AD et ailleurs qui ne sont pas accordées aux administrateurs du support technique. Grâce à ce chemin d’accès, un administrateur de support technique peut être en mesure d’assumer l’identité d’un propriétaire d’application, puis d’assumer l’identité d’une application privilégiée en mettant à jour les identifiants de celle-ci.
>- Propriétaires d’abonnement Azure, qui peuvent avoir accès à des informations confidentielles ou privées ou à une configuration critique dans Azure.
>- Propriétaires de groupe de sécurité et de groupe Microsoft 365, qui peuvent gérer l’appartenance à un groupe. Ces groupes peuvent accorder l’accès à des informations sensibles ou privées ou à une configuration critique dans Azure AD et ailleurs.
>- Administrateurs d’autres services en dehors d’Azure AD, tels qu’Exchange Online, le Centre de sécurité et de conformité Office et les systèmes de ressources humaines.
>- Les non-administrateurs comme les cadres supérieurs, les conseillers juridiques et les employés des ressources humaines qui peuvent avoir accès à des informations sensibles ou privées.

La délégation d’autorisations administratives à des sous-ensembles d’utilisateurs et l’application de stratégies à un sous-ensemble d’utilisateurs sont possibles avec des [unités administratives (maintenant en préversion publique)](administrative-units.md).

Ce rôle était auparavant appelé « administrateur de mot de passe » dans le [portail Azure](https://portal.azure.com/). Le nom « Administrateur du support » dans Azure AD correspond maintenant à celui utilisé dans Azure AD PowerShell et l’API Microsoft Graph.

### <a name="hybrid-identity-administrator"></a>[Administrateur d’identité hybride](#hybrid-identity-administrator-permissions)

Les utilisateurs de ce rôle peuvent créer, gérer et déployer la configuration du provisionnement d’AD vers Azure AD à l’aide du provisionnement cloud, ainsi que gérer les paramètres de fédération. Les utilisateurs peuvent également dépanner et superviser les journaux à l’aide de ce rôle.  

### <a name="insights-administrator"></a>[Administrateur Insights](#insights-administrator-permissions)
Les utilisateurs ayant ce rôle peuvent accéder à l’ensemble des fonctionnalités d’administration de l’[application M365 Insights](https://go.microsoft.com/fwlink/?linkid=2129521). Ce rôle permet de lire les informations d’annuaire, de superviser l’intégrité du service, de soumettre des tickets de support et d’accéder aux paramètres d’administration Insights.

### <a name="insights-business-leader"></a>[Leader d’entreprise Insights](#insights-business-leader-permissions)
Les utilisateurs ayant ce rôle peuvent accéder à un ensemble de tableaux de bord et d’insights par le biais de l’[application M365 Insights](https://go.microsoft.com/fwlink/?linkid=2129521). Ceci inclut l’accès total à tous les tableaux de bord et aux insights présentés et la fonctionnalité d’exploration de données. Les utilisateurs ayant ce rôle n’ont pas accès aux paramètres de configuration du produit. Cet accès est réservé au rôle Administrateur Insights.

### <a name="intune-administrator"></a>[Administrateur Intune](#intune-service-administrator-permissions)

Les utilisateurs avec ce rôle ont des autorisations générales dans Microsoft Intune Online, quand le service est présent. Ce rôle donne aussi la possibilité de gérer les utilisateurs et les appareils afin d’associer la stratégie, ainsi que de créer et de gérer des groupes. Pour plus d’informations, consultez la page [Contrôle d’accès en fonction du rôle (RBAC) avec Microsoft Intune](/intune/role-based-access-control).

Ce rôle peut créer et gérer tous les groupes de sécurité. Cependant, l’administrateur Intune ne dispose pas de droits d’administrateur pour les groupes Office. Cela signifie que l’administrateur ne peut pas mettre à jour les propriétaires ou les membres de tous les groupes Office au sein de l’organisation. Cependant, il peut gérer le groupe Office qu’il crée, ce qui fait partie de ses privilèges d’utilisateur final. Ainsi, tout groupe Office (et non un groupe de sécurité) qu’il crée doit être comptabilisé au titre de son quota de 250.

> [!NOTE]
> Dans l’API Microsoft Graph et Azure AD PowerShell, ce rôle est identifié comme « Administrateur de service Intune ». Il est « Administrateur Intune » dans le [portail Azure](https://portal.azure.com).

### <a name="kaizala-administrator"></a>[Administrateur Kaizala](#kaizala-administrator-permissions)

Les utilisateurs dotés de ce rôle ont des autorisations générales de gérer des paramètres au sein de Microsoft Kaizala, quand le service est présent, et ils peuvent gérer les tickets de support et superviser l’intégrité du service. De plus, l’utilisateur peut accéder à des rapports liés à l’adoption et à l’utilisation de Kaizala par les membres de l’organisation, ainsi qu’à des rapports d’entreprise générés avec les actions Kaizala.

### <a name="license-administrator"></a>[Administrateur de licence](#license-administrator-permissions)

Les utilisateurs dans ce rôle peuvent ajouter, supprimer et mettre à jour des attributions de licence pour les utilisateurs, les groupes (à l’aide de la gestion des licences par groupe) et gérer l’emplacement d’utilisation pour les utilisateurs. Le rôle ne permet pas d’acheter ou de gérer des abonnements, de créer ou de gérer des groupes, ni de créer ou de gérer les utilisateurs au-delà de leur emplacement d’utilisation. Ce rôle n’a pas d'accès pour afficher, créer ou gérer des tickets de support.

### <a name="message-center-privacy-reader"></a>[Lecteur de confidentialité du Centre de messages](#message-center-privacy-reader-permissions)

Les utilisateurs dotés de ce rôle peuvent superviser toutes les notifications dans le Centre de messages, notamment les messages de confidentialité des données. Les lecteurs de confidentialité du Centre de messages reçoivent des notifications par e-mail, notamment celles liées à la confidentialité des données, et ils peuvent se désabonner à l’aide des préférences du Centre de messages. Seuls l’administrateur général et le lecteur de confidentialité du Centre de messages peuvent lire les messages de confidentialité des données. De plus, ce rôle permet de voir les groupes, les domaines et les abonnements. Il n’est pas autorisé à voir, créer ou gérer des demandes de service.

### <a name="message-center-reader"></a>[Lecteur du Centre de messages](#message-center-reader-permissions)

Les utilisateurs titulaires de ce rôle peuvent superviser les notifications et les mises à jour d’avis d’intégrité dans le [Centre de messages](https://support.office.com/article/Message-center-in-Office-365-38FB3333-BFCC-4340-A37B-DEDA509C2093) pour leur organisation sur les services configurés comme Exchange, Intune et Microsoft Teams. Les lecteurs du Centre de messages reçoivent par e-mail des résumés hebdomadaires des publications, des mises à jour, et peuvent partager les messages publiés sur le Centre de messages dans Microsoft 365. Dans Azure AD, les utilisateurs affectés à ce rôle disposeront d’un accès en lecture seule aux services Azure AD comme les utilisateurs et les groupes. Ce rôle n’a pas d'accès pour afficher, créer ou gérer des tickets de support.

### <a name="modern-commerce-user"></a>[Utilisateur de Commerce moderne](#modern-commerce-user-permissions)

Ne pas utiliser. Ce rôle est attribué automatiquement à partir de Commerce, et n’est ni prévu, ni pris en charge pour toute autre utilisation. Consultez les détails ci-dessous.

Le rôle Utilisateur de Commerce moderne donne à certains utilisateurs l’autorisation d’accéder au centre d’administration Microsoft 365 et de consulter les entrées de navigation de gauche pour **Hébergement**, **Facturation** et **Support**. Le contenu disponible dans ces zones est contrôlé par des [rôles spécifiques du commerce](../../cost-management-billing/manage/understand-mca-roles.md) attribués aux utilisateurs pour gérer des produits qu’ils ont achetés pour eux-mêmes ou votre organisation. Cela peut inclure des tâches telles que le paiement de factures, ou l’accès à des comptes de facturation et à des profils de facturation. 

Les utilisateurs ayant le rôle Utilisateur de Commerce moderne disposent généralement d’autorisations d’administration dans d’autres systèmes d’achat Microsoft, mais ne disposent pas des rôles Administrateur général ou Administrateur de facturation utilisés pour accéder au centre d’administration. 

**Quand le rôle Utilisateur de Commerce moderne est-il attribué ?**

* **Achat en libre-service dans le Centre d’administration Microsoft 365** : l’achat en libre-service permet aux utilisateurs de tester de nouveaux produits en les achetant ou en s’y inscrivant eux-mêmes. Ces produits sont gérés dans le Centre d’administration. Les utilisateurs qui effectuent un achat en libre-service se voient attribuer un rôle dans le système Commerce et le rôle Utilisateur de Commerce moderne afin de pouvoir gérer leurs achats dans le centre d’administration. Les administrateurs peuvent bloquer les achats en libre-service (pour Power BI, Power Apps, Power automate) via [PowerShell](/microsoft-365/commerce/subscriptions/allowselfservicepurchase-powershell). Pour plus d’informations, consultez [FAQ sur l’achat en libre-service](/microsoft-365/commerce/subscriptions/self-service-purchase-faq).  
* **Achats sur la Place de marché commerciale de Microsoft** : comme pour l’achat en libre-service, quand un utilisateur achète un produit ou un service sur Microsoft AppSource ou la Place de Marché Azure, le rôle Utilisateur de Commerce moderne lui est attribué s’il n’a pas le rôle Administrateur général ou Administrateur de facturation. Dans certains cas, il peut arriver que des utilisateurs soient empêchés d’effectuer ces achats. Pour plus d’informations, consultez [Place de marché commerciale de Microsoft](../../marketplace/marketplace-faq-publisher-guide.md#what-could-block-a-customer-from-completing-a-purchase).
* **Propositions de Microsoft** : une proposition est une offre formelle de Microsoft à votre organisation pour l’achat de produits et services Microsoft. Quand la personne qui accepte la proposition n’a pas de rôle Administrateur général ou Administrateur de facturation dans Azure AD, elle se voit attribuer un rôle spécifique du commerce pour compléter la proposition et le rôle Utilisateur de Commerce moderne pour accéder au centre d’administration. Quand elle accède au Centre d’administration, elle ne peut utiliser que des fonctionnalités autorisées par son rôle spécifique du commerce.
* **Rôles spécifiques du commerce** : certains utilisateurs se voient attribuer des rôles spécifiques du commerce. Si un utilisateur n’est pas Administrateur général ou Administrateur de facturation, il obtient le rôle Utilisateur de Commerce moderne qui lui permet d’accéder au centre d’administration.  

En cas de suppression de l’attribution du rôle Utilisateur de Commerce moderne à un utilisateur, celui-ci perd son accès au centre d’administration Microsoft 365. S’il gérait des produits pour lui-même ou pour votre organisation, ils ne peut plus les gérer. Cette gestion peut inclure l’attribution de licences, la modification de modes de paiement, le paiement de factures ou d’autres tâches d’administration des abonnements.

### <a name="network-administrator"></a>[Administrateur réseau](#network-administrator-permissions)

Les utilisateurs de ce rôle peuvent passer en revue les recommandations de Microsoft relatives à l’architecture du périmètre réseau basées sur la télémétrie du réseau à partir de leur emplacement utilisateur. Les performances réseau pour Microsoft 365 s’appuient sur une architecture de périmètre de réseau client entreprise soigneuse, qui est généralement propre à la localisation de l’utilisateur. Ce rôle permet de modifier les emplacements utilisateur découverts et la configuration des paramètres réseau pour ces emplacements afin de faciliter les mesures de télémétrie et les suggestions de conception améliorées
### <a name="office-apps-administrator"></a>[Administrateur d’applications Office](#office-apps-administrator-permissions)

Les utilisateurs titulaires de ce rôle peuvent gérer les paramètres cloud des applications Microsoft 365. Cela comprend la gestion des stratégies cloud, la gestion du téléchargement en libre-service et la possibilité d’afficher des rapports relatifs aux applications Office. Ce rôle accorde en outre la possibilité de gérer les tickets de support et de surveiller l’intégrité du service au sein du centre d’administration principal. Les utilisateurs affectés à ce rôle peuvent également gérer la communication de nouvelles fonctionnalités dans les applications Office. 

### <a name="partner-tier1-support"></a>[Prise en charge de niveau 1 de partenaire](#partner-tier1-support-permissions)

Ne pas utiliser. Ce rôle est déconseillé et sera prochainement supprimé d’Azure AD. Il s’adresse à un petit nombre de partenaires revendeurs Microsoft et n’est pas destiné à une utilisation générale.

### <a name="partner-tier2-support"></a>[Prise en charge de niveau 2 de partenaire](#partner-tier2-support-permissions)

Ne pas utiliser. Ce rôle est déconseillé et sera prochainement supprimé d’Azure AD. Il s’adresse à un petit nombre de partenaires revendeurs Microsoft et n’est pas destiné à une utilisation générale.

### <a name="password-administrator"></a>[Administrateur de mots de passe](#password-administrator-permissions)

Les utilisateurs disposant de ce rôle ont une capacité limitée à gérer les mots de passe. Ce rôle n’accorde pas la possibilité de gérer les demandes de service ou de surveiller l’intégrité du service. Les administrateurs de mot de passe peuvent réinitialiser les mots de passe d’autres utilisateurs qui ne sont pas administrateurs ou membres des rôles suivants uniquement :

* Lecteurs de répertoires
* Inviteur
* Administrateur de mots de passe

### <a name="power-bi-administrator"></a>[Administrateur Power BI](#power-bi-service-administrator-permissions)

Les utilisateurs avec ce rôle ont des autorisations générales dans Microsoft Power BI, quand le service est présent, et peuvent gérer les tickets de support et superviser l’intégrité du service. Pour plus d’informations, consultez la page [Présentation du rôle d’administrateur Power BI](/power-bi/service-admin-role).

> [!NOTE]
> Dans l’API Microsoft Graph et Azure AD PowerShell, ce rôle est identifié comme « Administrateur de service Power BI ». Il est appelé « Administrateur Power BI » dans le [portail Azure](https://portal.azure.com).

### <a name="power-platform-administrator"></a>[Administrateur de plateforme Power](#power-platform-administrator-permissions)

Les utilisateurs de ce rôle peuvent créer et gérer tous les aspects des environnements, PowerApps, Flow et les stratégies de protection contre la perte de données. Les utilisateurs avec ce rôle sont aussi en mesure de gérer les tickets de support et de surveiller l’état des services.

### <a name="printer-administrator"></a>[Administrateur d’imprimantes](#printer-administrator-permissions)

Les utilisateurs ayant ce rôle peuvent inscrire des imprimantes et gérer tous les aspects de toutes les configurations d’imprimante dans la solution d’impression universelle Microsoft, y compris les paramètres du connecteur d’impression universel. Ils peuvent consentir à toutes les demandes d’autorisation d’impression déléguée. Les administrateurs d’imprimantes ont également accès aux rapports d’impression.

### <a name="printer-technician"></a>[Technicien en charge des imprimantes](#printer-technician-permissions)

Les utilisateurs ayant ce rôle peuvent inscrire des imprimantes et gérer leur état dans la solution d’impression universelle Microsoft. Ils peuvent également lire toutes les informations du connecteur. Une tâche clé qu’un technicien en charge des imprimantes ne peut pas accomplir est la définition d’autorisations utilisateur sur des imprimantes et le partage d’imprimantes.

### <a name="privileged-authentication-administrator"></a>[Administrateur d’authentification privilégié](#privileged-authentication-administrator-permissions)

Les utilisateurs dotés de ce rôle peuvent définir ou réinitialiser les informations d’identification de tous les utilisateurs, y compris des administrateurs généraux, et mettre à jour les mots de passe de tous les utilisateurs. Les administrateurs d’authentification privilégiés peuvent forcer les utilisateurs à se réinscrire avec des informations d’identification sans mot de passe existantes (par exemple, MFA, FIDO) et à révoquer la mémorisation de l’authentification multifacteur sur l’appareil, ce qui a pour effet de demander une authentification multifacteur lors de la prochaine connexion de tous les utilisateurs. Le rôle [Administrateur de l’authentification](#authentication-administrator) peut forcer la réinscription et l’authentification multifacteur exclusivement pour les utilisateurs non-administrateurs et les utilisateurs attribués aux rôles Azure AD suivants :

* Administrateur d’authentification
* Lecteurs de répertoires
* Inviteur
* Lecteur du Centre de messages
* Lecteur de rapports

### <a name="privileged-role-administrator"></a>[Administrateur de rôle privilégié](#privileged-role-administrator-permissions)

Les utilisateurs avec ce rôle peuvent gérer les attributions de rôles dans Azure Active Directory et Azure AD Privileged Identity Management. Ils peuvent créer et gérer des groupes qui peuvent être affectés à des rôles Azure AD. De plus, ce rôle permet de gérer tous les aspects de Privileged Identity Management et des unités administratives.

> [!IMPORTANT]
> Ce rôle donne la possibilité de gérer les attributions de tous les rôles Azure AD, y compris le rôle d’administrateur général. Ce rôle n’inclut pas d’autres capacités privilégiées dans Azure AD, comme la création ou la mise à jour des utilisateurs. Toutefois, les utilisateurs affectés à ce rôle peuvent s’accorder ou accorder à d’autres des privilèges supplémentaires en leur attribuant des rôles supplémentaires.

### <a name="reports-reader"></a>[Lecteur de rapports](#reports-reader-permissions)

Les utilisateurs avec ce rôle peuvent voir les données de rapports sur l’utilisation et le tableau de bord des rapports dans le centre d’administration Microsoft 365 et le pack du contexte d’adoption dans Power BI. En outre, ce rôle donne accès aux rapports de connexion et aux activités dans Azure AD, ainsi qu’aux données renvoyées par l’API de création de rapports Microsoft Graph. Un utilisateur disposant du rôle Lecteur de rapports peut uniquement accéder au métriques d’utilisation et d’adoption pertinentes. Il n’a pas les autorisations administrateur permettant de configurer les paramètres ou d’accéder aux centres d’administration propres au produit comme Exchange. Ce rôle n’a pas d'accès pour afficher, créer ou gérer des tickets de support.

### <a name="search-administrator"></a>[Administrateur de recherche](#search-administrator-permissions)

Les utilisateurs dotés de ce rôle disposent d’un accès total à toutes les fonctionnalités de gestion de Recherche Microsoft disponibles dans le centre d’administration Microsoft 365. De plus, ces utilisateurs peuvent voir le Centre de messages, superviser l’intégrité du service et créer des demandes de service.

### <a name="search-editor"></a>[Éditeur de recherche](#search-editor-permissions)

Les utilisateurs dotés de ce rôle peuvent créer, gérer et supprimer du contenu pour Recherche Microsoft dans le centre d’administration Microsoft 365, notamment des signets, des questions et réponses, ainsi que des emplacements.

### <a name="security-administrator"></a>[Administrateur de la sécurité](#security-administrator-permissions)

Les utilisateurs disposant de ce rôle sont autorisés à gérer les fonctionnalités liées à la sécurité dans le Centre de sécurité Microsoft 365, Azure Active Directory Identity Protection, Azure Active Directory Authentication, Azure Information Protection et le Centre de sécurité et conformité Office 365. Pour plus d’informations sur les autorisations Office 365, consultez [Autorisations dans le Centre de sécurité et de conformité](https://support.office.com/article/Permissions-in-the-Office-365-Security-Compliance-Center-d10608af-7934-490a-818e-e68f17d0e9c1).

Dans | Peut
--- | ---
[Centre de sécurité Microsoft 365](https://protection.office.com) | Surveiller les stratégies de sécurité au sein des services Microsoft 365<br>Gérer les menaces et les alertes liées à la sécurité<br>Afficher les rapports
Identity Protection Center | Toutes les autorisations du rôle lecteur de sécurité<br>En outre, possibilité d’effectuer toutes les opérations du centre de protection de l'identité, à l’exception de la réinitialisation des mots de passe.
[Privileged Identity Management](../privileged-identity-management/pim-configure.md) | Toutes les autorisations du rôle lecteur de sécurité<br>**Ne peut pas** gérer les paramètres ou les attributions de rôle Azure AD
[Centre de sécurité et conformité Office 365](https://support.office.com/article/About-Office-365-admin-roles-da585eea-f576-4f55-a1e0-87090b6aaa9d) | Gérer les stratégies de sécurité<br>Afficher, examiner et répondre aux menaces de sécurité<br>Afficher les rapports
Azure Advanced Threat Protection | Surveiller et répondre aux activités de sécurité suspectes
Windows Defender ATP et EDR | Attribuer des rôles<br>Gérer des groupes de machines<br>Configurer la détection des menaces liées aux points de terminaison et leur correction automatisée<br>Afficher, examiner et répondre aux alertes
[Intune](/intune/role-based-access-control) | Affiche des informations relatives à l'utilisateur, l'appareil, l'inscription, la configuration et l'application<br>Ne peut apporter de modifications à Intune
[Cloud App Security](/cloud-app-security/manage-admins) | Ajouter des administrateurs, des stratégies et des paramètres, charger des journaux d’activité et effectuer des actions de gouvernance
[Centre de sécurité Azure](../../key-vault/managed-hsm/built-in-roles.md) | peut afficher les stratégies de sécurité, les états de sécurité, les alertes et les recommandations, et ignorer les alertes et les recommandations
[Intégrité des services Microsoft 365](/office365/enterprise/view-service-health) | Afficher l’intégrité des services Microsoft 365
[Verrouillage intelligent](../authentication/howto-password-smart-lockout.md) | Définissez le seuil et la durée des verrouillages lorsque des événements d’échec de connexion se produisent.
[Protection par mot de passe](../authentication/concept-password-ban-bad.md) | Configurez une liste personnalisée de mots de passe interdits ou une protection par mot de passe locale.

### <a name="security-operator"></a>[Opérateur de sécurité](#security-operator-permissions)

Les utilisateurs dotés de ce rôle peuvent gérer les alertes et disposent d’un accès global en lecture seule aux fonctionnalités liées à la sécurité, notamment à toutes les informations disponibles dans le centre de sécurité Microsoft 365, Azure Active Directory, Identity Protection, Privileged Identity Management et le Centre de sécurité et de conformité Office 365. Pour plus d’informations sur les autorisations Office 365, consultez [Autorisations dans le Centre de sécurité et de conformité](/office365/securitycompliance/permissions-in-the-security-and-compliance-center).

Dans | Peut
--- | ---
[Centre de sécurité Microsoft 365](https://protection.office.com) | Toutes les autorisations du rôle lecteur de sécurité<br>Voir, examiner et répondre aux alertes sur les menaces de sécurité
Identity Protection Center | Toutes les autorisations du rôle lecteur de sécurité<br>En outre, possibilité d’effectuer toutes les opérations du centre de protection de l'identité, à l’exception de la réinitialisation des mots de passe.
[Privileged Identity Management](../privileged-identity-management/pim-configure.md) | Toutes les autorisations du rôle lecteur de sécurité
[Centre de sécurité et conformité Office 365](https://support.office.com/article/About-Office-365-admin-roles-da585eea-f576-4f55-a1e0-87090b6aaa9d) | Toutes les autorisations du rôle lecteur de sécurité<br>Voir, examiner et répondre aux alertes de sécurité
Windows Defender ATP et EDR | Toutes les autorisations du rôle lecteur de sécurité<br>Voir, examiner et répondre aux alertes de sécurité
[Intune](/intune/role-based-access-control) | Toutes les autorisations du rôle lecteur de sécurité
[Cloud App Security](/cloud-app-security/manage-admins) | Toutes les autorisations du rôle lecteur de sécurité
[Intégrité des services Microsoft 365](/office365/enterprise/view-service-health) | Afficher l’intégrité des services Microsoft 365

### <a name="security-reader"></a>[Lecteur de sécurité](#security-reader-permissions)

Les utilisateurs dotés de ce rôle ont un accès en lecture seule au niveau global à la fonctionnalité liée à la sécurité, notamment à toutes les informations dans le centre de sécurité Microsoft 365, Azure Active Directory, Identity Protection, Privileged Identity Management. Ils peuvent aussi lire les rapports sur les connexions Azure Active Directory et les journaux d’audit, ainsi que dans le centre de sécurité et de conformité Office 365. Pour plus d’informations sur les autorisations Office 365, consultez [Autorisations dans le Centre de sécurité et de conformité](https://support.office.com/article/Permissions-in-the-Office-365-Security-Compliance-Center-d10608af-7934-490a-818e-e68f17d0e9c1).

Dans | Peut
--- | ---
[Centre de sécurité Microsoft 365](https://protection.office.com) | Afficher les stratégies de sécurité au sein des services Microsoft 365<br>Afficher les menaces et les alertes liées à la sécurité<br>Afficher les rapports
Identity Protection Center | Lire tous les rapports de sécurité et informations de paramètres pour les fonctionnalités de sécurité<br><ul><li>Anti-spam<li>Chiffrement<li>Prévention contre la perte de données<li>Anti-programme malveillant<li>Détection avancée des menaces<li>Anti-hameçonnage<li>Règles du flux de messagerie
[Privileged Identity Management](../privileged-identity-management/pim-configure.md) | A un accès en lecture seule à toutes les informations disponibles dans Azure AD Privileged Identity Management : Stratégies et rapports pour les attributions de rôles Azure AD et révisions de sécurité.<br>**Ne peut pas** s’inscrire auprès d’Azure AD Privileged Identity Management ni y apporter des modifications. Dans le portail Privileged Identity Management ou par le biais de PowerShell, un membre ayant ce rôle peut activer des rôles supplémentaires (par exemple, un administrateur général ou un administrateur de rôle privilégié), s’il est éligible.
[Centre de sécurité et conformité Office 365](https://support.office.com/article/About-Office-365-admin-roles-da585eea-f576-4f55-a1e0-87090b6aaa9d) | Afficher les stratégies de sécurité<br>Afficher et examiner les menaces de sécurité<br>Afficher les rapports
Windows Defender ATP et EDR | Afficher et examiner les alertes. Lorsque vous activez le contrôle d’accès en fonction du rôle dans Windows Defender ATP, les utilisateurs disposant d’autorisations en lecture seule, comme le rôle Lecteur Sécurité Azure AD perdent l’accès jusqu’à ce qu’ils soient affectés à un rôle Windows Defender ATP.
[Intune](/intune/role-based-access-control) | Affiche des informations relatives à l'utilisateur, l'appareil, l'inscription, la configuration et l'application. Ne peut pas apporter de modifications à Intune.
[Cloud App Security](/cloud-app-security/manage-admins) | Dispose d'autorisations en lecture seule et peut gérer les alertes
[Centre de sécurité Azure](../../key-vault/managed-hsm/built-in-roles.md) | peut afficher les recommandations et les alertes, afficher les stratégies de sécurité, afficher les états de la sécurité, mais ne peut pas apporter des modifications
[Intégrité des services Microsoft 365](/office365/enterprise/view-service-health) | Afficher l’intégrité des services Microsoft 365

### <a name="service-support-administrator"></a>[Administrateur de support de service](#service-support-administrator-permissions)

Les utilisateurs titulaires de ce rôle peuvent ouvrir des demandes de support auprès de Microsoft pour les services Azure et Microsoft 365, ainsi que voir le tableau de bord des services et le Centre de messages dans le [portail Azure](https://portal.azure.com) et le [centre d’administration Microsoft 365](https://admin.microsoft.com). Plus d’informations sur les [Rôles d’administrateur](https://support.office.com/article/About-Office-365-admin-roles-da585eea-f576-4f55-a1e0-87090b6aaa9d).

> [!NOTE]
> Auparavant, ce rôle était nommé « Administrateur de services » dans [Portail Azure](https://portal.azure.com) et le [Centre d’administration Microsoft 365](https://admin.microsoft.com). Nous l’avons renommé « Administrateur de support de service » pour l’aligner sur le nom existant dans l’API Microsoft Graph, l’API Azure AD Graph et Azure AD PowerShell.

### <a name="sharepoint-administrator"></a>[Administrateur SharePoint](#sharepoint-service-administrator-permissions)

Les utilisateurs avec ce rôle ont des autorisations générales dans Microsoft SharePoint Online, quand le service est présent. Ils peuvent également créer et gérer tous les groupes Microsoft 365, gérer les tickets de support et superviser l’intégrité des services. Plus d’informations sur les [Rôles d’administrateur](https://support.office.com/article/About-Office-365-admin-roles-da585eea-f576-4f55-a1e0-87090b6aaa9d).

> [!NOTE]
> Dans l’API Microsoft Graph et Azure AD PowerShell, ce rôle est identifié comme « Administrateur de service SharePoint ». Il est « Administrateur SharePoint » dans le [portail Azure](https://portal.azure.com).

> [!NOTE]
> Ce rôle accorde également des autorisations délimitées à l’API Microsoft Graph pour Microsoft Intune, d’où la possibilité de gérer et de configurer des stratégies liées aux ressources SharePoint et Onedrive.

### <a name="skype-for-business--lync-administrator"></a>[Administrateur Skype Entreprise ou Administrateur Lync](#lync-service-administrator-permissions)

Les utilisateurs avec ce rôle ont des autorisations générales dans Microsoft Skype Entreprise, quand le service est présent. Ils peuvent également gérer les attributs utilisateur propres à Skype dans Azure Active Directory. En outre, ce rôle permet de gérer les tickets de support et de surveiller l’intégrité des services, ainsi que d’accéder au Centre d’administration Teams et Skype Entreprise. Le compte doit également disposer d’une licence Teams ; dans le cas contraire, il ne pourra pas exécuter les applets de commande PowerShell Teams. Pour plus d’informations, consultez la page [About the Skype for Business admin role](https://support.office.com/article/about-the-skype-for-business-admin-role-aeb35bda-93fc-49b1-ac2c-c74fbeb737b5) (À propos du rôle d’administrateur Skype Entreprise) ; pour plus d’informations sur les licences Teams, consultez l’article [Licences de compléments pour Skype Entreprise et Microsoft Teams](/skypeforbusiness/skype-for-business-and-microsoft-teams-add-on-licensing/skype-for-business-and-microsoft-teams-add-on-licensing).

> [!NOTE]
> Dans l’API Microsoft Graph et Azure AD PowerShell, ce rôle est identifié comme « Administrateur de service SharePoint ». Il est appelé « Administrateur Skype Entreprise » dans le [portail Azure](https://portal.azure.com/).

### <a name="teams-communications-administrator"></a>[Administrateur des communications Teams](#teams-communications-administrator-permissions)

Les utilisateurs dans ce rôle peuvent gérer les aspects de la charge de travail Microsoft Teams liés à la voix et à la téléphonie. Cela comprend les outils de gestion pour l’attribution des numéros de téléphone, les stratégies de voix et de réunion, ainsi que l’accès total à un ensemble d’outils d’analyse des appels.

### <a name="teams-communications-support-engineer"></a>[Ingénieur de support des communications Teams](#teams-communications-support-engineer-permissions)

Les utilisateurs dans ce rôle peuvent résoudre les problèmes de communication dans Microsoft Teams et Skype Entreprise à l’aide des outils de résolution de problèmes des appels utilisateur dans le centre d’administration Microsoft Teams et Skype Entreprise. Les utilisateurs disposant de ce rôle peuvent visualiser l’intégralité des informations d’enregistrement des appels pour l’ensemble des participants impliqués. Ce rôle n’a pas d'accès pour afficher, créer ou gérer des tickets de support.

### <a name="teams-communications-support-specialist"></a>[Spécialiste du support des communications Teams](#teams-communications-support-specialist-permissions)

Les utilisateurs dans ce rôle peuvent résoudre les problèmes de communication dans Microsoft Teams et Skype Entreprise à l’aide des outils de résolution de problèmes des appels utilisateur dans le centre d’administration Microsoft Teams et Skype Entreprise. Les utilisateurs disposant de ce rôle ne peuvent visualiser les détails utilisateur d’un appel que pour l’utilisateur qu’ils ont spécifiquement recherché. Ce rôle n’a pas d'accès pour afficher, créer ou gérer des tickets de support.

### <a name="teams-devices-administrator"></a>[Administrateur d’appareils Teams](#teams-devices-administrator-permissions)

Les utilisateurs titulaires de ce rôle peuvent gérer des [appareils certifiés par Teams](https://www.microsoft.com/microsoft-365/microsoft-teams/across-devices/devices) à partir du Centre d’administration Teams. Ce rôle permet d’afficher tous les appareils en un seul coup d’œil, avec la possibilité de rechercher et de filtrer les appareils. L’utilisateur peut vérifier les détails de chaque appareil, notamment le compte de connexion, la marque et le modèle de l’appareil. L’utilisateur peut modifier les paramètres sur l’appareil et mettre à jour les versions des logiciels. Ce rôle n’accorde pas d’autorisations pour vérifier l’activité Teams et la qualité d’appel de l’appareil. 

### <a name="teams-service-administrator"></a>[Administrateur du service Teams](#teams-service-administrator-permissions)

Les utilisateurs dans ce rôle peuvent gérer tous les aspects de la charge de travail Microsoft Teams via le centre d’administration Microsoft Teams et Skype Entreprise et les modules PowerShell respectifs. Cela inclut notamment tous les outils de gestion liés à la téléphonie, à la messagerie, aux réunions et aux équipes proprement dites. En outre, ce rôle permet de créer et de gérer tous les groupes Microsoft 365, de gérer les tickets de support et de surveiller l’état d’intégrité des services.

### <a name="user-administrator"></a>[Administrateur d’utilisateurs](#user-administrator-permissions)

Les utilisateurs dotés de ce rôle peuvent créer des utilisateurs, gérer tous les aspects des utilisateurs avec certaines restrictions (voir le tableau) et mettre à jour les stratégies d’expiration de mot de passe. De plus, les utilisateurs dotés de ce rôle peuvent créer et gérer tous les groupes. Ce rôle inclut également la possibilité de créer et de gérer des affichages utilisateur, de gérer les tickets de support et de surveiller l’état d’intégrité des services. Les administrateurs d’utilisateurs n’ont pas l’autorisation de gérer certaines propriétés utilisateur pour les utilisateurs affectés à des rôles d’administrateur. L’utilisateur ayant ce rôle n’est pas autorisé à gérer l’authentification multifacteur. Les rôles qui sont des exceptions à cette restriction sont listés dans le tableau suivant.

| **Permission** | **Peut** |
| --- | --- |
|Autorisations générales|<p>Créer des utilisateurs et des groupes</p><p>Créer et gérer des vues utilisateur</p><p>Gérer les tickets de support Office<p>Mettre à jour les stratégies d’expiration de mot de passe|
| <p>Tous les utilisateurs, notamment les administrateurs</p>|<p>Gérer les licences</p><p>Gérer toutes les propriétés de l’utilisateur, sauf le nom d’utilisateur principal</p>
| Uniquement pour les utilisateurs qui ne sont pas administrateurs ou qui ont l’un des rôles d’administrateur limités suivants :<ul><li>Lecteurs de répertoires<li>Administrateur de groupes<li>Inviteur<li>Administrateur du support technique<li>Lecteur du Centre de messages<li>Administrateur de mots de passe<li>Lecteur de rapports<li>Administrateur d'utilisateurs|<p>Supprimer et restaurer</p><p>Désactiver et activer</p><p>Invalider les jetons d’actualisation</p><p>Gérer toutes les propriétés de l’utilisateur, y compris le nom d’utilisateur principal</p><p>Réinitialiser le mot de passe</p><p>Mettre à jour les clés d’appareil (FIDO)</p>|

> [!IMPORTANT]
> Les utilisateurs auxquels ce rôle est assigné peuvent changer les mots de passe des personnes susceptibles d’avoir accès à des informations sensibles ou privées ou à des configurations critiques à l’intérieur et à l’extérieur d’Azure Active Directory. Changer le mot de passe d’un utilisateur peut signifier la capacité d’assumer l’identité et les autorisations de cet utilisateur. Par exemple :
>
>- Les propriétaires d’inscription d’application et d’application d’entreprise, qui peuvent gérer les informations d’identification des applications qu’ils possèdent. Ces applications peuvent disposer d’autorisations privilégiées dans Azure AD et ailleurs qui ne sont pas accordées aux administrateurs utilisateurs. Grâce à ce chemin d’accès, un administrateur utilisateur peut être en mesure d’assumer l’identité d’un propriétaire d’application, puis d’assumer l’identité d’une application privilégiée en mettant à jour les identifiants de celle-ci.
>- Propriétaires d’abonnement Azure, qui peuvent avoir accès à des informations confidentielles ou privées ou à une configuration critique dans Azure.
>- Propriétaires de groupe de sécurité et de groupe Microsoft 365, qui peuvent gérer l’appartenance à un groupe. Ces groupes peuvent accorder l’accès à des informations sensibles ou privées ou à une configuration critique dans Azure AD et ailleurs.
>- Administrateurs d’autres services en dehors d’Azure AD, tels qu’Exchange Online, le Centre de sécurité et de conformité Office et les systèmes de ressources humaines.
>- Les non-administrateurs comme les cadres supérieurs, les conseillers juridiques et les employés des ressources humaines qui peuvent avoir accès à des informations sensibles ou privées.

## <a name="role-permissions"></a>Autorisations des rôles

Les tableaux suivants décrivent les autorisations spécifiques à chaque rôle dans Azure Active Directory. Certains rôles peuvent disposer d’autorisations supplémentaires dans les services Microsoft en dehors d’Azure Active Directory.

### <a name="application-administrator-permissions"></a>Autorisations d’administrateur d’application

Peut créer et gérer tous les aspects des inscriptions d’applications et des applications d’entreprise.

| **Actions** | **Description** |
| --- | --- |
| microsoft.directory/Application/appProxyAuthentication/update | Mettez à jour les propriétés d’authentification du proxy d’application sur les principaux du service dans Azure Active Directory. |
| microsoft.directory/Application/appProxyUrlSettings/update | Mettez à jour les URL internes et externes du proxy d’application dans Azure Active Directory. |
| microsoft.directory/applications/applicationProxy/read | Lisez toutes les propriétés du proxy d’application. |
| microsoft.directory/applications/applicationProxy/update | Mettez à jour toutes les propriétés du proxy d’application. |
| microsoft.directory/applications/audience/update | Mettez à jour la propriété applications.audience dans Azure Active Directory. |
| microsoft.directory/applications/authentication/update | Mettez à jour la propriété applications.authentication dans Azure Active Directory. |
| microsoft.directory/applications/basic/update | Mettez à jour des propriétés de base sur des applications dans Azure Active Directory. |
| microsoft.directory/applications/create | Créez des applications dans Azure Active Directory. |
| microsoft.directory/applications/credentials/update | Mettez à jour la propriété applications.credentials dans Azure Active Directory. |
| microsoft.directory/applications/delete | Supprimez des applications dans Azure Active Directory. |
| microsoft.directory/applications/owners/update | Mettez à jour la propriété applications.owners dans Azure Active Directory. |
| microsoft.directory/applications/permissions/update | Mettez à jour la propriété applications.permissions dans Azure Active Directory. |
| microsoft.directory/applications/policies/update | Mettez à jour la propriété applications.policies dans Azure Active Directory. |
| microsoft.directory/appRoleAssignments/create | Créez des attributions de rôles d’applications dans Azure Active Directory. |
| microsoft.directory/appRoleAssignments/read | Lisez des attributions de rôles d’applications dans Azure Active Directory. |
| microsoft.directory/appRoleAssignments/update | Mettez à jour des attributions de rôles d’applications dans Azure Active Directory. |
| microsoft.directory/appRoleAssignments/delete | Supprimez des attributions de rôles d’applications dans Azure Active Directory. |
| microsoft.directory/auditLogs/allProperties/read | Lisez toutes les propriétés (y compris les propriétés privilégiées) sur auditLogs dans Azure Active Directory. |
| microsoft.directory/connectorGroups/allProperties/read | Lisez les propriétés du groupe de connecteurs de proxy d’application dans Azure Active Directory. |
| microsoft.directory/connectorGroups/allProperties/update | Mettez à jour toutes les propriétés du groupe de connecteurs de proxy d’application dans Azure Active Directory. |
| microsoft.directory/connectorGroups/create | Créez des groupes de connecteurs de proxy d’application dans Azure Active Directory. |
| microsoft.directory/connectorGroups/delete | Supprimez des groupes de connecteurs de proxy d’application dans Azure Active Directory. |
| microsoft.directory/connectors/allProperties/read | Lisez toutes les propriétés du connecteur de proxy d’application dans Azure Active Directory. |
| microsoft.directory/connectors/create | Créez des connecteurs de proxy d’application dans Azure Active Directory. |
| microsoft.directory/policies/applicationConfiguration/basic/read | Lisez la propriété policies.applicationConfiguration dans Azure Active Directory. |
| microsoft.directory/policies/applicationConfiguration/basic/update | Mettez à jour la propriété policies.applicationConfiguration dans Azure Active Directory. |
| microsoft.directory/policies/applicationConfiguration/create | Créez des stratégies dans Azure Active Directory. |
| microsoft.directory/policies/applicationConfiguration/delete | Supprimez des stratégies dans Azure Active Directory. |
| microsoft.directory/policies/applicationConfiguration/owners/read | Lisez la propriété policies.applicationConfiguration dans Azure Active Directory. |
| microsoft.directory/policies/applicationConfiguration/owners/update | Mettez à jour la propriété policies.applicationConfiguration dans Azure Active Directory. |
| microsoft.directory/policies/applicationConfiguration/policyAppliedTo/read | Lisez la propriété policies.applicationConfiguration dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/appRoleAssignedTo/update | Mettez à jour la propriété servicePrincipals.appRoleAssignedTo dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/appRoleAssignments/update | Mettez à jour la propriété servicePrincipals.appRoleAssignments dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/audience/update | Mettez à jour la propriété servicePrincipals.audience dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/authentication/update | Mettez à jour la propriété servicePrincipals.authentication dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/basic/update | Mettez à jour des propriétés de base sur des principaux de service dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/create | Créez des principaux de service dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/credentials/update | Mettez à jour la propriété servicePrincipals.credentials dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/delete | Supprimez des principaux de service dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/owners/update | Mettez à jour la propriété servicePrincipals.owners dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/permissions/update | Mettez à jour la propriété servicePrincipals.permissions dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/policies/update | Mettez à jour la propriété servicePrincipals.policies dans Azure Active Directory. |
| microsoft.directory/signInReports/allProperties/read | Lisez toutes les propriétés (y compris les propriétés privilégiées) sur signInReports dans Azure Active Directory. |
| microsoft.azure.serviceHealth/allEntities/allTasks | Lisez et configurez Azure Service Health. |
| microsoft.azure.supportTickets/allEntities/allTasks | Créez et gérez les tickets de support Azure. |
| microsoft.office365.serviceHealth/allEntities/allTasks | Lisez et configurez Microsoft 365 Service Health. |
| microsoft.office365.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Office 365. |

### <a name="application-developer-permissions"></a>Autorisations de développeur d’applications

Peut créer des inscriptions d’applications indépendamment du paramètre « Les utilisateurs peuvent inscrire des applications ».

| **Actions** | **Description** |
| --- | --- |
| microsoft.directory/applications/createAsOwner | Créez des applications dans Azure Active Directory. Le créateur est ajouté comme premier propriétaire, et l’objet créé compte dans le quota de 250 objets créés du créateur. |
| microsoft.directory/appRoleAssignments/createAsOwner | Créez des attributions de rôles d’applications dans Azure Active Directory. Le créateur est ajouté comme premier propriétaire, et l’objet créé compte dans le quota de 250 objets créés du créateur. |
| microsoft.directory/oAuth2PermissionGrants/createAsOwner | Créez des octrois d’autorisation OAuth2 dans Azure Active Directory. Le créateur est ajouté comme premier propriétaire, et l’objet créé compte dans le quota de 250 objets créés du créateur. |
| microsoft.directory/servicePrincipals/createAsOwner | Créez des principaux de service dans Azure Active Directory. Le créateur est ajouté comme premier propriétaire, et l’objet créé compte dans le quota de 250 objets créés du créateur. |

### <a name="authentication-administrator-permissions"></a>Autorisations d’administrateur d’authentification

Permet d’afficher, de définir et de réinitialiser les informations de méthode d’authentification pour tout utilisateur non administrateur.

| **Actions** | **Description** |
| --- | --- |
| microsoft.directory/users/invalidateAllRefreshTokens | Invalidez tous les jetons d’actualisation utilisateur dans Azure Active Directory. |
| microsoft.directory/users/strongAuthentication/update | Mettez à jour les propriétés d’authentification forte comme les informations d’identification MFA. |
| microsoft.azure.serviceHealth/allEntities/allTasks | Lisez et configurez Azure Service Health. |
| microsoft.azure.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Azure pour les services au niveau de l’annuaire. |
| microsoft.office365.webPortal/allEntities/basic/read | Lisez les propriétés de base sur toutes les ressources dans microsoft.office365.webPortal. |
| microsoft.office365.serviceHealth/allEntities/allTasks | Lisez et configurez Microsoft 365 Service Health. |
| microsoft.office365.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Office 365. |
| microsoft.directory/users/password/update | Mettez à jour les mots de passe de tous les utilisateurs de l’organisation Microsoft 365. Pour plus d’informations, consultez la documentation en ligne. |

### <a name="azure-devops-administrator-permissions"></a>Autorisations de l’administrateur Azure DevOps

Peut gérer la stratégie et les paramètres de l’organisation Azure DevOps.

> [!NOTE]
> Ce rôle dispose d’autorisations supplémentaires en dehors d’Azure Active Directory. Pour plus d’informations, consultez la [description des rôles](#azure-devops-administrator) ci-dessus.
>
>

| **Actions** | **Description** |
| --- | --- |
| microsoft.azure.devOps/allEntities/allTasks | Lisez et configurez Azure DevOps. |

### <a name="azure-information-protection-administrator-permissions"></a>Autorisations d’administrateur Azure Information Protection

Peut gérer tous les aspects du service Azure Information Protection.

> [!NOTE]
> Ce rôle dispose d’autorisations supplémentaires en dehors d’Azure Active Directory. Pour plus d’informations, consultez la [description des rôles](#) ci-dessus.
>
>

| **Actions** | **Description** |
| --- | --- |
| microsoft.azure.informationProtection/allEntities/allTasks | Gérez tous les aspects d’Azure Information Protection. |
| microsoft.azure.serviceHealth/allEntities/allTasks | Lisez et configurez Azure Service Health. |
| microsoft.azure.supportTickets/allEntities/allTasks | Créez et gérez les tickets de support Azure. |
| microsoft.office365.serviceHealth/allEntities/allTasks | Lisez et configurez Microsoft 365 Service Health. |
| microsoft.office365.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Office 365. |

### <a name="b2c-ief-keyset-administrator-permissions"></a>Autorisations d’administrateur de jeux de clés B2C IEF

Gérez les secrets pour la fédération et le chiffrement dans Identity Experience Framework.

| **Actions** | **Description** |
| --- | --- |
| microsoft.aad.b2c/trustFramework/keySets/allTasks | Lisez et configurez des jeux de clés dans Azure Active Directory B2C. |

### <a name="b2c-ief-policy-administrator-permissions"></a>Autorisations d’administrateur de stratégies B2C IEF

Créez et gérez des stratégies d’infrastructure de confiance dans Identity Experience Framework.

| **Actions** | **Description** |
| --- | --- |
| microsoft.aad.b2c/trustFramework/policies/allTasks | Lisez et configurez des stratégies personnalisées dans Azure Active Directory B2C. |

### <a name="billing-administrator-permissions"></a>Autorisations d’administrateur de facturation

Peut effectuer des tâches de facturation courantes, comme la mise à jour des informations de paiement.

> [!NOTE]
> Ce rôle dispose d’autorisations supplémentaires en dehors d’Azure Active Directory. Pour plus d’informations, consultez la description des rôles ci-dessus.
>
>

| **Actions** | **Description** |
| --- | --- |
| microsoft.directory/organization/basic/update | Mettez à jour des propriétés de base sur une organisation dans Azure Active Directory. |
| microsoft.azure.serviceHealth/allEntities/allTasks | Lisez et configurez Azure Service Health. |
| microsoft.azure.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Azure pour les services au niveau de l’annuaire. |
| microsoft.commerce.billing/allEntities/allTasks | Gérez tous les aspects de la facturation. |
| microsoft.office365.webPortal/allEntities/basic/read | Lisez les propriétés de base sur toutes les ressources dans microsoft.office365.webPortal. |
| microsoft.office365.serviceHealth/allEntities/allTasks | Lisez et configurez Microsoft 365 Service Health. |
| microsoft.office365.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Office 365. |

### <a name="cloud-application-administrator-permissions"></a>Autorisations d’administrateur d’application cloud

Peut créer et gérer tous les aspects des inscriptions d’applications et des applications d’entreprise, à l’exception du proxy d’application.

| **Actions** | **Description** |
| --- | --- |
| microsoft.directory/applications/audience/update | Mettez à jour la propriété applications.audience dans Azure Active Directory. |
| microsoft.directory/applications/authentication/update | Mettez à jour la propriété applications.authentication dans Azure Active Directory. |
| microsoft.directory/applications/basic/update | Mettez à jour des propriétés de base sur des applications dans Azure Active Directory. |
| microsoft.directory/applications/create | Créez des applications dans Azure Active Directory. |
| microsoft.directory/applications/credentials/update | Mettez à jour la propriété applications.credentials dans Azure Active Directory. |
| microsoft.directory/applications/delete | Supprimez des applications dans Azure Active Directory. |
| microsoft.directory/applications/owners/update | Mettez à jour la propriété applications.owners dans Azure Active Directory. |
| microsoft.directory/applications/permissions/update | Mettez à jour la propriété applications.permissions dans Azure Active Directory. |
| microsoft.directory/applications/policies/update | Mettez à jour la propriété applications.policies dans Azure Active Directory. |
| microsoft.directory/appRoleAssignments/create | Créez des attributions de rôles d’applications dans Azure Active Directory. |
| microsoft.directory/appRoleAssignments/update | Mettez à jour des attributions de rôles d’applications dans Azure Active Directory. |
| microsoft.directory/appRoleAssignments/delete | Supprimez des attributions de rôles d’applications dans Azure Active Directory. |
| microsoft.directory/auditLogs/allProperties/read | Lisez toutes les propriétés (y compris les propriétés privilégiées) sur auditLogs dans Azure Active Directory. |
| microsoft.directory/policies/applicationConfiguration/create | Créez des stratégies dans Azure Active Directory. |
| microsoft.directory/policies/applicationConfiguration/basic/read | Lisez la propriété policies.applicationConfiguration dans Azure Active Directory. |
| microsoft.directory/policies/applicationConfiguration/basic/update | Mettez à jour la propriété policies.applicationConfiguration dans Azure Active Directory. |
| microsoft.directory/policies/applicationConfiguration/delete | Supprimez des stratégies dans Azure Active Directory. |
| microsoft.directory/policies/applicationConfiguration/owners/read | Lisez la propriété policies.applicationConfiguration dans Azure Active Directory. |
| microsoft.directory/policies/applicationConfiguration/owners/update | Mettez à jour la propriété policies.applicationConfiguration dans Azure Active Directory. |
| microsoft.directory/policies/applicationConfiguration/policyAppliedTo/read | Lisez la propriété policies.applicationConfiguration dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/appRoleAssignedTo/update | Mettez à jour la propriété servicePrincipals.appRoleAssignedTo dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/appRoleAssignments/update | Mettez à jour la propriété servicePrincipals.appRoleAssignments dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/audience/update | Mettez à jour la propriété servicePrincipals.audience dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/authentication/update | Mettez à jour la propriété servicePrincipals.authentication dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/basic/update | Mettez à jour des propriétés de base sur des principaux de service dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/create | Créez des principaux de service dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/credentials/update | Mettez à jour la propriété servicePrincipals.credentials dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/delete | Supprimez des principaux de service dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/owners/update | Mettez à jour la propriété servicePrincipals.owners dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/permissions/update | Mettez à jour la propriété servicePrincipals.permissions dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/policies/update | Mettez à jour la propriété servicePrincipals.policies dans Azure Active Directory. |
| microsoft.directory/signInReports/allProperties/read | Lisez toutes les propriétés (y compris les propriétés privilégiées) sur signInReports dans Azure Active Directory. |
| microsoft.azure.serviceHealth/allEntities/allTasks | Lisez et configurez Azure Service Health. |
| microsoft.azure.supportTickets/allEntities/allTasks | Créez et gérez les tickets de support Azure. |
| microsoft.office365.serviceHealth/allEntities/allTasks | Lisez et configurez Microsoft 365 Service Health. |
| microsoft.office365.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Office 365. |

### <a name="cloud-device-administrator-permissions"></a>Autorisations d’administrateur d’appareil cloud

Accès total pour gérer des appareils dans Azure AD.

| **Actions** | **Description** |
| --- | --- |
| microsoft.directory/auditLogs/allProperties/read | Lisez toutes les propriétés (y compris les propriétés privilégiées) sur auditLogs dans Azure Active Directory. |
| microsoft.directory/bitlockerKeys/key/read | Lisez les objets et les propriétés des clés BitLocker (y compris la clé de récupération) dans Azure Active Directory. |
| microsoft.directory/devices/delete | Supprimez des appareils dans Azure Active Directory. |
| microsoft.directory/devices/disable | Désactivez des appareils dans Azure Active Directory. |
| microsoft.directory/devices/enable | Activez des appareils dans Azure Active Directory. |
| microsoft.directory/devices/extensionAttributes/update | Mettez à jour toutes les valeurs de la propriété devices.extensionAttributes dans Azure Active Directory. |
| microsoft.directory/signInReports/allProperties/read | Lisez toutes les propriétés (y compris les propriétés privilégiées) sur signInReports dans Azure Active Directory. |
| microsoft.azure.serviceHealth/allEntities/allTasks | Lisez et configurez Azure Service Health. |
| microsoft.office365.serviceHealth/allEntities/allTasks | Lisez et configurez Microsoft 365 Service Health. |

### <a name="company-administrator-permissions"></a>Autorisations d’administrateur de la société

Peut gérer tous les aspects d’Azure AD et des services Microsoft qui utilisent des identités Azure AD. Ce rôle est également connu sous le nom de rôle d’administrateur général. 

> [!NOTE]
> Ce rôle dispose d’autorisations supplémentaires en dehors d’Azure Active Directory. Pour plus d’informations, consultez la description des rôles ci-dessus.
>
>

| **Actions** | **Description** |
| --- | --- |
| microsoft.aad.cloudAppSecurity/allEntities/allTasks | Créez et supprimez toutes les ressources, et lisez et mettez à jour des propriétés standard dans microsoft.aad.cloudAppSecurity. |
| microsoft.directory/administrativeUnits/allProperties/allTasks | Créez et supprimez des unités administratives (administrativeUnits), et lisez et mettez à jour toutes les propriétés dans Azure Active Directory. |
| microsoft.directory/applications/allProperties/allTasks | Créez et supprimez des applications, et lisez et mettez à jour toutes les propriétés dans Azure Active Directory. |
| microsoft.directory/appRoleAssignments/allProperties/allTasks | Créez et supprimez des attributions de rôles d’applications (appRoleAssignments), et lisez et mettez à jour toutes les propriétés dans Azure Active Directory. |
| microsoft.directory/auditLogs/allProperties/read | Lisez toutes les propriétés (y compris les propriétés privilégiées) sur auditLogs dans Azure Active Directory. |
| microsoft.directory/bitlockerKeys/key/read | Lisez les objets et les propriétés des clés BitLocker (y compris la clé de récupération) dans Azure Active Directory. |
| microsoft.directory/contacts/allProperties/allTasks | Créez et supprimez des contacts, et lisez et mettez à jour toutes les propriétés dans Azure Active Directory. |
| microsoft.directory/contracts/allProperties/allTasks | Créez et supprimez des contrats, et lisez et mettez à jour toutes les propriétés dans Azure Active Directory. |
| microsoft.directory/devices/allProperties/allTasks | Créez et supprimez des appareils, et lisez et mettez à jour toutes les propriétés dans Azure Active Directory. |
| microsoft.directory/directoryRoles/allProperties/allTasks | Créez et supprimez des rôles d’annuaire (directoryRoles), et lisez et mettez à jour toutes les propriétés dans Azure Active Directory. |
| microsoft.directory/directoryRoleTemplates/allProperties/allTasks | Créez et supprimez des modèles de rôles d’annuaire (DirectoryRoleTemplates), et lisez et mettez à jour toutes les propriétés dans Azure Active Directory. |
| microsoft.directory/domains/allProperties/allTasks | Créez et supprimez des domaines, et lisez et mettez à jour toutes les propriétés dans Azure Active Directory. |
| microsoft.directory/groups/allProperties/allTasks | Créez et supprimez des groupes, et lisez et mettez à jour toutes les propriétés dans Azure Active Directory. |
| microsoft.directory/groupsAssignableToRoles/allProperties/update | Mettez à jour les groupes dont la propriété isAssignableToRole est définie sur true dans Azure Active Directory. |
| microsoft.directory/groupsAssignableToRoles/create | Créez des groupes dont la propriété isAssignableToRole est définie sur true dans Azure Active Directory. |
| microsoft.directory/groupsAssignableToRoles/delete | Supprimez les groupes dont la propriété isAssignableToRole est définie sur true dans Azure Active Directory. |
| microsoft.directory/groupSettings/allProperties/allTasks | Créez et supprimez des paramètres de groupes (groupSettings), et lisez et mettez à jour toutes les propriétés dans Azure Active Directory. |
| microsoft.directory/groupSettingTemplates/allProperties/allTasks | Créez et supprimez des modèles de paramètres de groupes (groupSettingTemplates), et lisez et mettez à jour toutes les propriétés dans Azure Active Directory. |
| microsoft.directory/loginTenantBranding/allProperties/allTasks | Créez et supprimez une personnalisations de client de connexion (loginTenantBranding), et lisez et mettez à jour toutes les propriétés dans Azure Active Directory. |
| microsoft.directory/oAuth2PermissionGrants/allProperties/allTasks | Créez et supprimez des octrois d’autorisation OAuth2 (oAuth2PermissionGrants), et lisez et mettez à jour toutes les propriétés dans Azure Active Directory. |
| microsoft.directory/organization/allProperties/allTasks | Créez et supprimez des organisations, et lisez et mettez à jour toutes les propriétés dans Azure Active Directory. |
| microsoft.directory/policies/allProperties/allTasks | Créez et supprimez des stratégies, et lisez et mettez à jour toutes les propriétés dans Azure Active Directory. |
| microsoft.directory/roleAssignments/allProperties/allTasks | Créez et supprimez des attributions de rôles (roleAssignments), et lisez et mettez à jour toutes les propriétés dans Azure Active Directory. |
| microsoft.directory/roleDefinitions/allProperties/allTasks | Créez et supprimez des définitions de rôles (roleDefinitions), et lisez et mettez à jour toutes les propriétés dans Azure Active Directory. |
| microsoft.directory/scopedRoleMemberships/allProperties/allTasks | Créez et supprimez des appartenances à un rôle étendu (scopedRoleMemberships), et lisez et mettez à jour toutes les propriétés dans Azure Active Directory. |
| microsoft.directory/serviceAction/activateService | Peut effectuer l’action de service Activateservice dans Azure Active Directory |
| microsoft.directory/serviceAction/disableDirectoryFeature | Peut effectuer l’action de service Disabledirectoryfeature dans Azure Active Directory |
| microsoft.directory/serviceAction/enableDirectoryFeature | Peut effectuer l’action de service Enabledirectoryfeature dans Azure Active Directory |
| microsoft.directory/serviceAction/getAvailableExtentionProperties | Peut effectuer l’action de service Getavailableextentionproperties dans Azure Active Directory |
| microsoft.directory/servicePrincipals/allProperties/allTasks | Créez et supprimez des principaux de service (servicePrincipals), et lisez et mettez à jour toutes les propriétés dans Azure Active Directory. |
| microsoft.directory/signInReports/allProperties/read | Lisez toutes les propriétés (y compris les propriétés privilégiées) sur signInReports dans Azure Active Directory. |
| microsoft.directory/subscribedSkus/allProperties/allTasks | Créez et supprimez des références SKU souscrites, et lisez et mettez à jour toutes les propriétés dans Azure Active Directory. |
| microsoft.directory/users/allProperties/allTasks | Créez et supprimez des utilisateurs, et lisez et mettez à jour toutes les propriétés dans Azure Active Directory. |
| microsoft.directorySync/allEntities/allTasks | Exécutez toutes les actions dans Azure AD Connect. |
| microsoft.aad.directorySync/allEntities/allTasks | Créez et supprimez toutes les ressources, et lisez et mettez à jour des propriétés standard dans microsoft.aad.identityProtection. |
| microsoft.aad.privilegedIdentityManagement/allEntities/read | Lisez toutes les ressources dans microsoft.aad.privilegedIdentityManagement. |
| microsoft.azure.advancedThreatProtection/allEntities/read | Lisez toutes les ressources dans microsoft.azure.advancedThreatProtection. |
| microsoft.azure.informationProtection/allEntities/allTasks | Gérez tous les aspects d’Azure Information Protection. |
| microsoft.azure.serviceHealth/allEntities/allTasks | Lisez et configurez Azure Service Health. |
| microsoft.azure.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Azure pour les services au niveau de l’annuaire. |
| microsoft.commerce.billing/allEntities/allTasks | Gérez tous les aspects de la facturation. |
| microsoft.intune/allEntities/allTasks | Gérez tous les aspects d’Intune. |
| microsoft.office365.complianceManager/allEntities/allTasks | Gérez tous les aspects d’Office 365 Compliance Manager |
| microsoft.office365.desktopAnalytics/allEntities/allTasks | Gérez tous les aspects de Desktop Analytics. |
| microsoft.office365.exchange/allEntities/allTasks | Gérez tous les aspects d’Exchange Online. |
| microsoft.office365.lockbox/allEntities/allTasks | Gérez tous les aspects d’Office 365 Customer Lockbox. |
| microsoft.office365.messageCenter/messages/read | Lisez les messages dans microsoft.office365.messageCenter. |
| microsoft.office365.messageCenter/securityMessages/read | Lisez les messages securityMessages dans microsoft.office365.messageCenter. |
| microsoft.office365.protectionCenter/allEntities/allTasks | Gérez tous les aspects du Centre de protection Office 365. |
| microsoft.office365.securityComplianceCenter/allEntities/allTasks | Créez et supprimez toutes les ressources, et lisez et mettez à jour des propriétés standard dans microsoft.office365.securityComplianceCenter. |
| microsoft.office365.serviceHealth/allEntities/allTasks | Lisez et configurez Microsoft 365 Service Health. |
| microsoft.office365.sharepoint/allEntities/allTasks | Créez et supprimez toutes les ressources, et lisez et mettez à jour des propriétés standard dans microsoft.office365.sharepoint. |
| microsoft.office365.skypeForBusiness/allEntities/allTasks | Gérez tous les aspects de Skype Entreprise Online. |
| microsoft.office365.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Office 365. |
| microsoft.office365.usageReports/allEntities/read | Lisez des rapports d’utilisation Office 365. |
| microsoft.office365.webPortal/allEntities/basic/read | Lisez les propriétés de base sur toutes les ressources dans microsoft.office365.webPortal. |
| microsoft.powerApps.dynamics365/allEntities/allTasks | Gérez tous les aspects de Dynamics 365. |
| microsoft.powerApps.powerBI/allEntities/allTasks | Gérez tous les aspects de Power BI. |
| microsoft.windows.defenderAdvancedThreatProtection/allEntities/read | Lisez toutes les ressources dans microsoft.windows.defenderAdvancedThreatProtection. |

### <a name="compliance-administrator-permissions"></a>Autorisations d’administrateur de conformité

Peut lire et gérer la configuration de la conformité et les rapports dans Azure AD et Microsoft 365.

> [!NOTE]
> Ce rôle dispose d’autorisations supplémentaires en dehors d’Azure Active Directory. Pour plus d’informations, consultez la description des rôles ci-dessus.
>
>

| **Actions** | **Description** |
| --- | --- |
| microsoft.azure.serviceHealth/allEntities/allTasks | Lisez et configurez Azure Service Health. |
| microsoft.azure.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Azure pour les services au niveau de l’annuaire. |
| microsoft.office365.complianceManager/allEntities/allTasks | Gérez tous les aspects d’Office 365 Compliance Manager |
| microsoft.office365.serviceHealth/allEntities/allTasks | Lisez et configurez Microsoft 365 Service Health. |
| microsoft.office365.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Office 365. |
| microsoft.office365.webPortal/allEntities/basic/read | Lisez les propriétés de base sur toutes les ressources dans microsoft.office365.webPortal. |

### <a name="compliance-data-administrator-permissions"></a>Autorisations d’administrateur des données de conformité

Crée et gère le contenu de conformité.

> [!NOTE]
> Ce rôle dispose d’autorisations supplémentaires en dehors d’Azure Active Directory. Pour plus d’informations, consultez la description des rôles ci-dessus.
>
>

| **Actions** | **Description** |
| --- | --- |
| microsoft.directory.cloudAppSecurity/allEntities/allTasks | Lisez et configurez Microsoft Cloud App Security. |
| microsoft.azure.informationProtection/allEntities/allTasks | Gérez tous les aspects d’Azure Information Protection. |
| microsoft.azure.serviceHealth/allEntities/allTasks | Lisez et configurez Azure Service Health. |
| microsoft.azure.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Azure pour les services au niveau de l’annuaire. |
| microsoft.office365.complianceManager/allEntities/allTasks | Gérez tous les aspects d’Office 365 Compliance Manager |
| microsoft.office365.serviceHealth/allEntities/allTasks | Lisez et configurez Microsoft 365 Service Health. |
| microsoft.office365.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Office 365. |
| microsoft.office365.webPortal/allEntities/basic/read | Lisez les propriétés de base sur toutes les ressources dans microsoft.office365.webPortal. |

### <a name="conditional-access-administrator-permissions"></a>Autorisations d’administrateur d’accès conditionnel

Peut gérer les fonctionnalités d’accès conditionnel.

| **Actions** | **Description** |
| --- | --- |
| microsoft.directory/policies/conditionalAccess/basic/read | Lisez la propriété policies.conditionalAccess dans Azure Active Directory. |
| microsoft.directory/policies/conditionalAccess/basic/update | Mettez à jour la propriété policies.conditionalAccess dans Azure Active Directory. |
| microsoft.directory/policies/conditionalAccess/create | Créez des stratégies dans Azure Active Directory. |
| microsoft.directory/policies/conditionalAccess/delete | Supprimez des stratégies dans Azure Active Directory. |
| microsoft.directory/policies/conditionalAccess/owners/read | Lisez la propriété policies.conditionalAccess dans Azure Active Directory. |
| microsoft.directory/policies/conditionalAccess/owners/update | Mettez à jour la propriété policies.conditionalAccess dans Azure Active Directory. |
| microsoft.directory/policies/conditionalAccess/policiesAppliedTo/read | Lisez la propriété policies.conditionalAccess dans Azure Active Directory. |
| microsoft.directory/policies/conditionalAccess/tenantDefault/update | Mettez à jour la propriété policies.conditionalAccess dans Azure Active Directory. |

### <a name="crm-service-administrator-permissions"></a>Autorisations d’administrateur de service CRM

Peut gérer tous les aspects du produit Dynamics 365.

> [!NOTE]
> Ce rôle dispose d’autorisations supplémentaires en dehors d’Azure Active Directory. Pour plus d’informations, consultez la description des rôles ci-dessus.
>
>

| **Actions** | **Description** |
| --- | --- |
| microsoft.azure.serviceHealth/allEntities/allTasks | Lisez et configurez Azure Service Health. |
| microsoft.azure.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Azure pour les services au niveau de l’annuaire. |
| microsoft.powerApps.dynamics365/allEntities/allTasks | Gérez tous les aspects de Dynamics 365. |
| microsoft.office365.webPortal/allEntities/basic/read | Lisez les propriétés de base sur toutes les ressources dans microsoft.office365.webPortal. |
| microsoft.office365.serviceHealth/allEntities/allTasks | Lisez et configurez Microsoft 365 Service Health. |
| microsoft.office365.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Office 365. |

### <a name="customer-lockbox-access-approver-permissions"></a>Autorisations d’approbateur d’accès Customer Lockbox

Peut approuver les demandes de support Microsoft pour accéder aux données organisationnelles du client.

> [!NOTE]
> Ce rôle dispose d’autorisations supplémentaires en dehors d’Azure Active Directory. Pour plus d’informations, consultez la description des rôles ci-dessus.
>
>

| **Actions** | **Description** |
| --- | --- |
| microsoft.office365.webPortal/allEntities/basic/read | Lisez les propriétés de base sur toutes les ressources dans microsoft.office365.webPortal. |
| microsoft.office365.lockbox/allEntities/allTasks | Gérez tous les aspects d’Office 365 Customer Lockbox. |

### <a name="desktop-analytics-administrator-permissions"></a>Autorisations d’administrateur Desktop Analytics

Peut gérer Desktop Analytics et les services de stratégie et de personnalisation Office. Pour Desktop Analytics, cela inclut la possibilité d’afficher l’inventaire des biens, de créer des plans de déploiement, d’afficher l’état du déploiement et de l’intégrité. Pour le service de stratégie et personnalisation Office, ce rôle permet aux utilisateurs de gérer les stratégies Office.

> [!NOTE]
> Ce rôle dispose d’autorisations supplémentaires en dehors d’Azure Active Directory. Pour plus d’informations, consultez la description des rôles ci-dessus.
>
>

| **Actions** | **Description** |
| --- | --- |
| microsoft.azure.serviceHealth/allEntities/allTasks | Lisez et configurez Azure Service Health. |
| microsoft.azure.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Azure pour les services au niveau de l’annuaire. |
| microsoft.office365.webPortal/allEntities/basic/read | Lisez les propriétés de base sur toutes les ressources dans microsoft.office365.webPortal. |
| microsoft.office365.desktopAnalytics/allEntities/allTasks | Gérez tous les aspects de Desktop Analytics. |
| microsoft.office365.serviceHealth/allEntities/allTasks | Lisez et configurez Microsoft 365 Service Health. |
| microsoft.office365.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Office 365. |

### <a name="device-administrators-permissions"></a>Autorisations des administrateurs d’appareils

Les utilisateurs dotés de ce rôle sont ajoutés au groupe d’administrateurs locaux sur des appareils joints à AD Azure.

| **Actions** | **Description** |
| --- | --- |
| microsoft.directory/groupSettings/basic/read | Lisez des propriétés de base sur des paramètres de groupes dans Azure Active Directory. |
| microsoft.directory/groupSettingTemplates/basic/read | Lisez des propriétés de base sur des modèles de paramètres de groupes dans Azure Active Directory. |

### <a name="directory-readers-permissions"></a>Autorisations des lecteurs de répertoire
Peut lire les informations d’annuaire de base. Pour accorder l’accès aux applications, non destiné aux utilisateurs.

| **Actions** | **Description** |
| --- | --- |
| microsoft.directory/administrativeUnits/basic/read | Lisez des propriétés de base sur des unités administratives dans Azure Active Directory. |
| microsoft.directory/administrativeUnits/members/read | Lisez la propriété administrativeUnits.members dans Azure Active Directory. |
| microsoft.directory/applications/basic/read | Lisez des propriétés de base sur des applications dans Azure Active Directory. |
| microsoft.directory/applications/owners/read | Lisez la propriété applications.owners dans Azure Active Directory. |
| microsoft.directory/applications/policies/read | Lisez la propriété applications.policies dans Azure Active Directory. |
| microsoft.directory/contacts/basic/read | Lisez des propriétés de base sur des contacts dans Azure Active Directory. |
| microsoft.directory/contacts/memberOf/read | Lisez la propriété contacts.memberOf dans Azure Active Directory. |
| microsoft.directory/contracts/basic/read | Lisez des propriétés de base sur des contrats dans Azure Active Directory. |
| microsoft.directory/devices/basic/read | Lisez des propriétés de base sur des appareils dans Azure Active Directory. |
| microsoft.directory/devices/memberOf/read | Lisez la propriété devices.memberOf dans Azure Active Directory. |
| microsoft.directory/devices/registeredOwners/read | Lisez la propriété devices.registeredOwners dans Azure Active Directory. |
| microsoft.directory/devices/registeredUsers/read | Lisez la propriété devices.registeredUsers dans Azure Active Directory. |
| microsoft.directory/directoryRoles/basic/read | Lisez des propriétés de base sur des rôles d’annuaire dans Azure Active Directory. |
| microsoft.directory/directoryRoles/eligibleMembers/read | Lisez la propriété directoryRoles.eligibleMembers dans Azure Active Directory. |
| microsoft.directory/directoryRoles/members/read | Lisez la propriété directoryRoles.members dans Azure Active Directory. |
| microsoft.directory/domains/basic/read | Lisez des propriétés de base sur des domaines dans Azure Active Directory. |
| microsoft.directory/groups/appRoleAssignments/read | Lisez la propriété groups.appRoleAssignments dans Azure Active Directory. |
| microsoft.directory/groups/basic/read | Lisez des propriétés de base sur des groupes dans Azure Active Directory. |
| microsoft.directory/groups/memberOf/read | Lisez la propriété groups.memberOf dans Azure Active Directory. |
| microsoft.directory/groups/members/read | Lisez la propriété groups.members dans Azure Active Directory. |
| microsoft.directory/groups/owners/read | Lisez la propriété groups.owners dans Azure Active Directory. |
| microsoft.directory/groups/settings/read | Lisez la propriété groups.settings dans Azure Active Directory. |
| microsoft.directory/groupSettings/basic/read | Lisez des propriétés de base sur des paramètres de groupes dans Azure Active Directory. |
| microsoft.directory/groupSettingTemplates/basic/read | Lisez des propriétés de base sur des modèles de paramètres de groupes dans Azure Active Directory. |
| microsoft.directory/oAuth2PermissionGrants/basic/read | Lisez des propriétés de base sur des octrois d’autorisation OAuth2 dans Azure Active Directory. |
| microsoft.directory/organization/basic/read | Lisez des propriétés de base sur une organisation dans Azure Active Directory. |
| microsoft.directory/organization/trustedCAsForPasswordlessAuth/read | Lisez la propriété organization.trustedCAsForPasswordlessAuth dans Azure Active Directory. |
| microsoft.directory/roleAssignments/basic/read | Lisez des propriétés de base sur roleAssignments dans Azure Active Directory. |
| microsoft.directory/roleDefinitions/basic/read | Lisez des propriétés de base sur roleDefinitions dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/appRoleAssignedTo/read | Lisez la propriété servicePrincipals.appRoleAssignedTo dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/appRoleAssignments/read | Lisez la propriété servicePrincipals.appRoleAssignments dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/basic/read | Lisez des propriétés de base sur des principaux de service dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/memberOf/read | Lisez la propriété servicePrincipals.memberOf dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/oAuth2PermissionGrants/basic/read | Lisez la propriété servicePrincipals.oAuth2PermissionGrants dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/ownedObjects/read | Lisez la propriété servicePrincipals.ownedObjects dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/owners/read | Lisez la propriété servicePrincipals.owners dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/policies/read | Lisez la propriété servicePrincipals.policies dans Azure Active Directory. |
| microsoft.directory/subscribedSkus/basic/read | Lisez des propriétés de base sur des références SKU souscrites dans Azure Active Directory. |
| microsoft.directory/users/appRoleAssignments/read | Lisez la propriété users.appRoleAssignments dans Azure Active Directory. |
| microsoft.directory/users/basic/read | Lisez des propriétés de base sur des utilisateurs dans Azure Active Directory. |
| microsoft.directory/users/directReports/read | Lisez la propriété users.directReports dans Azure Active Directory. |
| microsoft.directory/users/manager/read | Lisez la propriété users.manager dans Azure Active Directory. |
| microsoft.directory/users/memberOf/read | Lisez la propriété users.memberOf dans Azure Active Directory. |
| microsoft.directory/users/oAuth2PermissionGrants/basic/read | Lisez la propriété users.oAuth2PermissionGrants dans Azure Active Directory. |
| microsoft.directory/users/ownedDevices/read | Lisez la propriété users.ownedDevices dans Azure Active Directory. |
| microsoft.directory/users/ownedObjects/read | Lisez la propriété users.ownedObjects dans Azure Active Directory. |
| microsoft.directory/users/registeredDevices/read | Lisez la propriété users.registeredDevices dans Azure Active Directory. |

### <a name="directory-synchronization-accounts-permissions"></a>Autorisations des comptes de synchronisation d’annuaires

Utilisés uniquement par le service Azure AD Connect.

| **Actions** | **Description** |
| --- | --- |
| microsoft.directory/organization/dirSync/update | Mettez à jour la propriété organization.dirSync dans Azure Active Directory. |
| microsoft.directory/policies/create | Créez des stratégies dans Azure Active Directory. |
| microsoft.directory/policies/delete | Supprimez des stratégies dans Azure Active Directory. |
| microsoft.directory/policies/basic/read | Lisez des propriétés de base sur des stratégies dans Azure Active Directory. |
| microsoft.directory/policies/basic/update | Mettez à jour des propriétés de base sur des stratégies dans Azure Active Directory. |
| microsoft.directory/policies/owners/read | Lisez la propriété policies.owners dans Azure Active Directory. |
| microsoft.directory/policies/owners/update | Mettez à jour la propriété policies.owners dans Azure Active Directory. |
| microsoft.directory/policies/policiesAppliedTo/read | Lisez la propriété policies.policiesAppliedTo dans Azure Active Directory. |
| microsoft.directory/policies/tenantDefault/update | Mettez à jour la propriété policies.tenantDefault dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/appRoleAssignedTo/read | Lisez la propriété servicePrincipals.appRoleAssignedTo dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/appRoleAssignedTo/update | Mettez à jour la propriété servicePrincipals.appRoleAssignedTo dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/appRoleAssignments/read | Lisez la propriété servicePrincipals.appRoleAssignments dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/appRoleAssignments/update | Mettez à jour la propriété servicePrincipals.appRoleAssignments dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/audience/update | Mettez à jour la propriété servicePrincipals.audience dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/authentication/update | Mettez à jour la propriété servicePrincipals.authentication dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/basic/read | Lisez des propriétés de base sur des principaux de service dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/basic/update | Mettez à jour des propriétés de base sur des principaux de service dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/create | Créez des principaux de service dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/credentials/update | Mettez à jour la propriété servicePrincipals.credentials dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/memberOf/read | Lisez la propriété servicePrincipals.memberOf dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/oAuth2PermissionGrants/basic/read | Lisez la propriété servicePrincipals.oAuth2PermissionGrants dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/owners/read | Lisez la propriété servicePrincipals.owners dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/owners/update | Mettez à jour la propriété servicePrincipals.owners dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/ownedObjects/read | Lisez la propriété servicePrincipals.ownedObjects dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/permissions/update | Mettez à jour la propriété servicePrincipals.permissions dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/policies/read | Lisez la propriété servicePrincipals.policies dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/policies/update | Mettez à jour la propriété servicePrincipals.policies dans Azure Active Directory. |
| microsoft.directorySync/allEntities/allTasks | Exécutez toutes les actions dans Azure AD Connect. |

### <a name="directory-writers-permissions"></a>Autorisations des enregistreurs de répertoire

Peut lire et écrire des informations d’annuaire de base. Pour accorder l’accès aux applications, non destiné aux utilisateurs.

| **Actions** | **Description** |
| --- | --- |
| microsoft.directory/groups/appRoleAssignments/update | Mettez à jour la propriété groups.appRoleAssignments dans Azure Active Directory. |
| microsoft.directory/groups/assignLicense | Gérez les licences des groupes dans Azure Active Directory. |
| microsoft.directory/groups/basic/update | Mettez à jour des propriétés de base sur des groupes dans Azure Active Directory.  |
| microsoft.directory/groups/classification/update | Mettez à jour la propriété de classification du groupe dans Azure Active Directory. |
| microsoft.directory/groups/create | Créez des groupes dans Azure Active Directory. |
| microsoft.directory/groups/groupType/update | Mettez à jour la propriété groupType d’un groupe dans Azure Active Directory. |
| microsoft.directory/groups/members/update | Mettez à jour la propriété groups.members dans Azure Active Directory. |
| microsoft.directory/groups/owners/update | Mettez à jour la propriété groups.owners dans Azure Active Directory. |
| microsoft.directory/groups/reprocessLicenseAssignment | Traitez à nouveau l’attribution des licences pour un groupe dans Azure Active Directory. |
| microsoft.directory/groups/securityEnabled/update | Mettez à jour la propriété secutiryEnabled d’un groupe dans Azure Active Directory. |
| microsoft.directory/groups/settings/update | Mettez à jour la propriété groups.settings dans Azure Active Directory. |
| microsoft.directory/groups/visibility/update | Mettez à jour la propriété de visibilité du groupe. |
| microsoft.directory/groupSettings/basic/update | Mettez à jour des propriétés de base sur des paramètres de groupe dans Azure Active Directory. |
| microsoft.directory/groupSettings/create | Créez groupSettings dans Azure Active Directory. |
| microsoft.directory/groupSettings/delete | Supprimez des paramètres de groupe dans Azure Active Directory. |
| microsoft.directory/oAuth2PermissionGrants/basic/update | Mettez à jour les propriétés de base d’oAuth2PermissionGrants dans Azure Active Directory. |
| microsoft.directory/oAuth2PermissionGrants/create | Créez des octrois d’autorisation OAuth2 dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/synchronizationCredentials/manage | Gérez les secrets et les informations d’identification de l’approvisionnement des applications. |
| microsoft.directory/servicePrincipals/synchronizationJobs/manage | Démarrez, redémarrez et suspendez les travaux de synchronisation de l’approvisionnement des applications. |
| microsoft.directory/servicePrincipals/synchronizationSchema/manage | Créez et gérez le schéma et les tâches de synchronisation de provisionnement des applications. |
| microsoft.directory/users/appRoleAssignments/update | Mettez à jour la propriété users.appRoleAssignments dans Azure Active Directory. |
| microsoft.directory/users/assignLicense | Gérez les licences sur des utilisateurs dans Azure Active Directory. |
| microsoft.directory/users/basic/update | Mettez à jour des propriétés de base sur des utilisateurs dans Azure Active Directory. |
| microsoft.directory/users/create | Créez des utilisateurs dans Azure Active Directory. |
| microsoft.directory/users/disable | Désactivez un compte d’utilisateur dans Azure Active Directory. |
| microsoft.directory/users/enable | Activez un compte d’utilisateur dans Azure Active Directory. |
| microsoft.directory/users/invalidateAllRefreshTokens | Invalidez tous les jetons d’actualisation utilisateur dans Azure Active Directory, en demandant aux utilisateurs de s’authentifier à nouveau lors de leur prochaine connexion. |
| microsoft.directory/users/manager/update | Mettez à jour la propriété users.manager dans Azure Active Directory. |
| microsoft.directory/users/reprocessLicenseAssignment | Traitez à nouveau l’attribution des licences pour un utilisateur dans Azure Active Directory. |
| microsoft.directory/users/userPrincipalName /update | Mettez à jour la propriété users.userPrincipalName dans Azure Active Directory. |

### <a name="exchange-service-administrator-permissions"></a>Autorisations d’administrateur de service Exchange

Peut gérer tous les aspects du produit Exchange.

> [!NOTE]
> Ce rôle dispose d’autorisations supplémentaires en dehors d’Azure Active Directory. Pour plus d’informations, consultez la description des rôles ci-dessus.
>
>

| **Actions** | **Description** |
| --- | --- |
| microsoft.azure.serviceHealth/allEntities/allTasks | Lisez et configurez Azure Service Health. |
| microsoft.azure.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Azure pour les services au niveau de l’annuaire. |
| microsoft.directory/groups/unified/appRoleAssignments/update | Mettez à jour la propriété groups.unified dans Azure Active Directory. |
| microsoft.directory/groups/unified/basic/update | Mettez à jour les propriétés de base des groupes Microsoft 365. |
| microsoft.directory/groups/unified/create | Créez des groupes Microsoft 365. |
| microsoft.directory/groups/unified/delete | Supprimez des groupes Microsoft 365. |
| microsoft.directory/groups/unified/members/update | Mettez à jour l’appartenance à des groupes Microsoft 365. |
| microsoft.directory/groups/unified/owners/update | Mettez à jour la propriété des groupes Microsoft 365. |
| microsoft.office365.exchange/allEntities/allTasks | Gérez tous les aspects d’Exchange Online. |
| microsoft.office365.network/performance/allProperties/read | Lisez les pages sur les performances réseau dans le Centre d’administration Microsoft 365. |
| microsoft.office365.serviceHealth/allEntities/allTasks | Lisez et configurez Microsoft 365 Service Health. |
| microsoft.office365.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Office 365. |
| microsoft.office365.usageReports/allEntities/read | Lisez des rapports d’utilisation Office 365. |
| microsoft.office365.webPortal/allEntities/basic/read | Lisez les propriétés de base sur toutes les ressources dans microsoft.office365.webPortal. |

### <a name="external-id-user-flow-administrator-permissions"></a>Autorisations d’administrateur de flux d’utilisateurs ID externe

Créez et gérez tous les aspects des flux d’utilisateurs.

| **Actions** | **Description** |
| --- | --- |
| microsoft.aad.b2c/userFlows/allTasks | Lisez et configurez des flux d’utilisateurs dans Azure Active Directory B2C. |

### <a name="external-id-user-flow-attribute-administrator-permissions"></a>Autorisations d’administrateur d’attribut de flux d’utilisateurs ID externe

Créez et gérez le schéma d’attribut disponible pour tous les flux d’utilisateurs.

| **Actions** | **Description** |
| --- | --- |
| microsoft.aad.b2c/userAttributes/allTasks | Lisez et configurez les attributs utilisateur dans Azure Active Directory B2C. |

### <a name="external-identity-provider-administrator-permissions"></a>Autorisations d’administrateur de fournisseurs d’identité externes

Configurez des fournisseurs d’identité pour une utilisation dans une fédération directe.

| **Actions** | **Description** |
| --- | --- |
| microsoft.aad.b2c/identityProviders/allTasks | Lisez et configurez les fournisseurs d’identité dans Azure Active Directory B2C. |

### <a name="global-reader-permissions"></a>Autorisations du lecteur général
Peut lire tous les éléments qu’un administrateur général peut lire, mais ne peut pas les modifier.

> [!NOTE]
> Ce rôle dispose d’autorisations supplémentaires en dehors d’Azure Active Directory. Pour plus d’informations, consultez la [description des rôles](#global-reader) ci-dessus.
>
>

| **Actions** | **Description** |
| --- | --- |
| microsoft.commerce.billing/allEntities/read    | Lisez tous les aspects de la facturation. |
| microsoft.directory/administrativeUnits/basic/read    | Lisez des propriétés de base sur des unités administratives dans Azure Active Directory. |
| microsoft.directory/administrativeUnits/members/read    | Lisez la propriété administrativeUnits.members dans Azure Active Directory. |
| microsoft.directory/applications/basic/read    | Lisez des propriétés de base sur des applications dans Azure Active Directory. |
| microsoft.directory/applications/owners/read    | Lisez la propriété applications.owners dans Azure Active Directory. |
| microsoft.directory/applications/policies/read    | Lisez la propriété applications.policies dans Azure Active Directory. |
| microsoft.directory/bitlockerKeys/key/read | Lisez les objets et les propriétés des clés BitLocker (y compris la clé de récupération) dans Azure Active Directory. |
| microsoft.directory/contacts/basic/read    | Lisez des propriétés de base sur des contacts dans Azure Active Directory. |
| microsoft.directory/contacts/memberOf/read    | Lisez la propriété contacts.memberOf dans Azure Active Directory. |
| microsoft.directory/contracts/basic/read    | Lisez des propriétés de base sur des contrats dans Azure Active Directory. |
| microsoft.directory/devices/basic/read    | Lisez des propriétés de base sur des appareils dans Azure Active Directory. |
| microsoft.directory/devices/memberOf/read    | Lisez la propriété devices.memberOf dans Azure Active Directory. |
| microsoft.directory/devices/registeredOwners/read    | Lisez la propriété devices.registeredOwners dans Azure Active Directory. |
| microsoft.directory/devices/registeredUsers/read    | Lisez la propriété devices.registeredUsers dans Azure Active Directory. |
| microsoft.directory/directoryRoles/basic/read    | Lisez des propriétés de base sur des rôles d’annuaire dans Azure Active Directory. |
| microsoft.directory/directoryRoles/eligibleMembers/read    | Lisez la propriété directoryRoles.eligibleMembers dans Azure Active Directory. |
| microsoft.directory/directoryRoles/members/read    | Lisez la propriété directoryRoles.members dans Azure Active Directory. |
| microsoft.directory/domains/basic/read    | Lisez des propriétés de base sur des domaines dans Azure Active Directory. |
| microsoft.directory/groups/appRoleAssignments/read    | Lisez la propriété groups.appRoleAssignments dans Azure Active Directory. |
| microsoft.directory/groups/basic/read    | Lisez des propriétés de base sur des groupes dans Azure Active Directory. |
| microsoft.directory/groups/hiddenMembers/read    | Lisez la propriété groups.hiddenMembers dans Azure Active Directory. |
| microsoft.directory/groups/memberOf/read    | Lisez la propriété groups.memberOf dans Azure Active Directory. |
| microsoft.directory/groups/members/read    | Lisez la propriété groups.members dans Azure Active Directory. |
| microsoft.directory/groups/owners/read    | Lisez la propriété groups.owners dans Azure Active Directory. |
| microsoft.directory/groups/settings/read    | Lisez la propriété groups.settings dans Azure Active Directory. |
| microsoft.directory/groupSettings/basic/read    | Lisez des propriétés de base sur des paramètres de groupes dans Azure Active Directory. |
| microsoft.directory/groupSettingTemplates/basic/read    | Lisez des propriétés de base sur des modèles de paramètres de groupes dans Azure Active Directory. |
| microsoft.directory/oAuth2PermissionGrants/basic/read    | Lisez des propriétés de base sur des octrois d’autorisation OAuth2 dans Azure Active Directory. |
| microsoft.directory/organization/basic/read    | Lisez des propriétés de base sur une organisation dans Azure Active Directory. |
| microsoft.directory/organization/trustedCAsForPasswordlessAuth/read    | Lisez la propriété organization.trustedCAsForPasswordlessAuth dans Azure Active Directory. |
| microsoft.directory/policies/standard/read    | Lisez les stratégies standard dans Azure Active Directory. |
| microsoft.directory/roleAssignments/basic/read    | Lisez des propriétés de base sur roleAssignments dans Azure Active Directory. |
| microsoft.directory/roleDefinitions/basic/read    | Lisez des propriétés de base sur roleDefinitions dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/appRoleAssignedTo/read    | Lisez la propriété servicePrincipals.appRoleAssignedTo dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/appRoleAssignments/read    | Lisez la propriété servicePrincipals.appRoleAssignments dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/basic/read    | Lisez des propriétés de base sur des principaux de service dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/memberOf/read    | Lisez la propriété servicePrincipals.memberOf dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/oAuth2PermissionGrants/basic/read    | Lisez la propriété servicePrincipals.oAuth2PermissionGrants dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/ownedObjects/read    | Lisez la propriété servicePrincipals.ownedObjects dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/owners/read    | Lisez la propriété servicePrincipals.owners dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/policies/read    | Lisez la propriété servicePrincipals.policies dans Azure Active Directory. |
| microsoft.directory/signInReports/allProperties/read    | Lisez toutes les propriétés (y compris les propriétés privilégiées) sur signInReports dans Azure Active Directory. |
| microsoft.directory/subscribedSkus/basic/read    | Lisez des propriétés de base sur des références SKU souscrites dans Azure Active Directory. |
| microsoft.directory/users/appRoleAssignments/read    | Lisez la propriété users.appRoleAssignments dans Azure Active Directory. |
| microsoft.directory/users/basic/read    | Lisez des propriétés de base sur des utilisateurs dans Azure Active Directory. |
| microsoft.directory/users/directReports/read    | Lisez la propriété users.directReports dans Azure Active Directory. |
| microsoft.directory/users/manager/read    | Lisez la propriété users.manager dans Azure Active Directory. |
| microsoft.directory/users/memberOf/read    | Lisez la propriété users.memberOf dans Azure Active Directory. |
| microsoft.directory/users/oAuth2PermissionGrants/basic/read    | Lisez la propriété users.oAuth2PermissionGrants dans Azure Active Directory. |
| microsoft.directory/users/ownedDevices/read    | Lisez la propriété users.ownedDevices dans Azure Active Directory. |
| microsoft.directory/users/ownedObjects/read    | Lisez la propriété users.ownedObjects dans Azure Active Directory. |
| microsoft.directory/users/registeredDevices/read    | Lisez la propriété users.registeredDevices dans Azure Active Directory. |
| microsoft.directory/users/strongAuthentication/read    | Lisez les propriétés d’authentification forte comme les informations d’identification MFA. |
| microsoft.office365.exchange/allEntities/read    | Lisez tous les aspects d’Exchange Online. |
| microsoft.office365.messageCenter/messages/read    | Lisez les messages dans microsoft.office365.messageCenter. |
| microsoft.office365.messageCenter/securityMessages/read    | Lisez les messages securityMessages dans microsoft.office365.messageCenter. |
| microsoft.office365.network/performance/allProperties/read | Lisez les pages sur les performances réseau dans le Centre d’administration Microsoft 365. |
| microsoft.office365.protectionCenter/allEntities/read    | Lisez tous les aspects du Centre de protection Office 365. |
| microsoft.office365.securityComplianceCenter/allEntities/read    | Lisez toutes les propriétés standard dans microsoft.365.securityComplianceCenter. |
| microsoft.office365.usageReports/allEntities/read    | Lisez des rapports d’utilisation Office 365. |
| microsoft.office365.webPortal/allEntities/standard/read    | Lisez les propriétés standard sur toutes les ressources dans microsoft.office365.webPortal. |

### <a name="groups-administrator-permissions"></a>Autorisations d’administrateur de groupes
Peut gérer tous les aspects des groupes et des paramètres de groupe comme les stratégies d’expiration et de nommage.

| **Actions** | **Description** |
| --- | --- |
| microsoft.directory/groups/basic/read | Lisez des propriétés standard sur des groupes dans Azure Active Directory.  |
| microsoft.directory/groups/basic/update | Mettez à jour des propriétés de base sur des groupes dans Azure Active Directory. |
| microsoft.directory/groups/create | Créez des groupes dans Azure Active Directory. |
| microsoft.directory/groups/createAsOwner | Créez des groupes dans Azure Active Directory. Le créateur est ajouté comme premier propriétaire, et l’objet créé compte dans le quota de 250 objets créés du créateur. |
| microsoft.directory/groups/delete | Supprimez des groupes dans Azure Active Directory. |
| microsoft.directory/groups/hiddenMembers/read | Lisez la propriété groups.hiddenMembers dans Azure Active Directory. |
| microsoft.directory/groups/members/update | Mettez à jour la propriété groups.members dans Azure Active Directory. |
| microsoft.directory/groups/owners/update | Mettez à jour la propriété groups.owners dans Azure Active Directory. |
| microsoft.directory/groups/restore | Restaurez des groupes dans Azure Active Directory. |
| microsoft.directory/groups/settings/update | Mettez à jour la propriété groups.settings dans Azure Active Directory. |
| microsoft.azure.serviceHealth/allEntities/allTasks | Lisez et configurez Azure Service Health. |
| microsoft.azure.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Azure pour les services au niveau de l’annuaire. |
| microsoft.office365.messageCenter/messages/read | Lisez les messages dans microsoft.office365.messageCenter. |
| microsoft.office365.serviceHealth/allEntities/allTasks | Lisez et configurez Microsoft 365 Service Health. |
| microsoft.office365.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Office 365. |

### <a name="guest-inviter-permissions"></a>Autorisations de l’inviteur d’invités
Peut inviter des utilisateurs invités indépendamment du paramètre « Les membres peuvent inviter des invités ».

| **Actions** | **Description** |
| --- | --- |
| microsoft.directory/users/appRoleAssignments/read | Lisez la propriété users.appRoleAssignments dans Azure Active Directory. |
| microsoft.directory/users/basic/read | Lisez des propriétés de base sur des utilisateurs dans Azure Active Directory. |
| microsoft.directory/users/directReports/read | Lisez la propriété users.directReports dans Azure Active Directory. |
| microsoft.directory/users/inviteGuest | Invitez des utilisateurs invités dans Azure Active Directory. |
| microsoft.directory/users/manager/read | Lisez la propriété users.manager dans Azure Active Directory. |
| microsoft.directory/users/memberOf/read | Lisez la propriété users.memberOf dans Azure Active Directory. |
| microsoft.directory/users/oAuth2PermissionGrants/basic/read | Lisez la propriété users.oAuth2PermissionGrants dans Azure Active Directory. |
| microsoft.directory/users/ownedDevices/read | Lisez la propriété users.ownedDevices dans Azure Active Directory. |
| microsoft.directory/users/ownedObjects/read | Lisez la propriété users.ownedObjects dans Azure Active Directory. |
| microsoft.directory/users/registeredDevices/read | Lisez la propriété users.registeredDevices dans Azure Active Directory. |

### <a name="helpdesk-administrator-permissions"></a>Autorisations d’administrateur du support technique

Peut réinitialiser des mots de passe pour les utilisateurs non-administrateurs et les administrateurs du support technique.

| **Actions** | **Description** |
| --- | --- |
| microsoft.directory/devices/bitLockerRecoveryKeys/read | Lisez la propriété devices.bitLockerRecoveryKeys dans Azure Active Directory. |
| microsoft.directory/users/invalidateAllRefreshTokens | Invalidez tous les jetons d’actualisation utilisateur dans Azure Active Directory. |
| microsoft.directory/users/password/update | Mettez à jour des mots de passe pour tous les utilisateurs dans Azure Active Directory. Pour plus d’informations, consultez la documentation en ligne. |
| microsoft.azure.serviceHealth/allEntities/allTasks | Lisez et configurez Azure Service Health. |
| microsoft.azure.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Azure pour les services au niveau de l’annuaire. |
| microsoft.office365.webPortal/allEntities/basic/read | Lisez les propriétés de base sur toutes les ressources dans microsoft.office365.webPortal. |
| microsoft.office365.serviceHealth/allEntities/allTasks | Lisez et configurez Microsoft 365 Service Health. |
| microsoft.office365.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Office 365. |

### <a name="hybrid-identity-administrator-permissions"></a>Autorisations d’administrateur d’identité hybride

Activer, déployer, configurer, gérer, surveiller et résoudre les problèmes liés à l’approvisionnement cloud et aux services d’authentification. 

| **Actions** | **Description** |
| --- | --- |
| microsoft.azure.serviceHealth/allEntities/allTasks | Lisez et configurez Azure Service Health. |
| microsoft.azure.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Azure pour les services au niveau de l’annuaire. |
| microsoft.directory/applications/audience/update  | Mettez à jour la propriété applications.audience dans Azure Active Directory. |
| microsoft.directory/applications/authentication/update | Mettez à jour la propriété applications.authentication dans Azure Active Directory.  |
| microsoft.directory/applications/basic/update | Mettez à jour des propriétés de base sur des applications dans Azure Active Directory. |
| microsoft.directory/applications/create | Créez des applications dans Azure Active Directory. |
| microsoft.directory/applications/credentials/update | Mettez à jour la propriété applications.credentials dans Azure Active Directory. |
| microsoft.directory/applications/delete | Supprimez des applications dans Azure Active Directory. |
| microsoft.directory/applications/owners/update | Mettez à jour la propriété applications.owners dans Azure Active Directory. |
| microsoft.directory/applications/permissions/update | Mettez à jour la propriété applications.permissions dans Azure Active Directory. |
| microsoft.directory/applications/policies/update | Mettez à jour la propriété applications.policies dans Azure Active Directory. |
| microsoft.directory/applicationTemplates/instantiate | Instanciez des applications de la galerie à partir de modèles d’application. |
| microsoft.directory/auditLogs/allProperties/read | Lisez toutes les propriétés (y compris les propriétés privilégiées) sur auditLogs dans Azure Active Directory. |
| microsoft.directory/cloudProvisioning/allProperties/allTasks | Lisez et configurez toutes les propriétés du service d’approvisionnement cloud Azure AD. |
| microsoft.directory/federatedAuthentication/allProperties/allTasks | Gérez tous les aspects des services fédérés Active Directory (ADFS) ou d’un fournisseur de fédération tiers dans Azure AD. |
| microsoft.directory/organization/dirSync/update | Mettez à jour la propriété organization.dirSync dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/audience/update | Mettez à jour la propriété servicePrincipals.audience dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/authentication/update | Mettez à jour la propriété servicePrincipals.authentication dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/basic/update | Mettez à jour des propriétés de base sur des principaux de service dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/create | Créez des principaux de service dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/credentials/update | Mettez à jour la propriété servicePrincipals.credentials dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/delete | Supprimez des principaux de service dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/owners/update | Mettez à jour la propriété servicePrincipals.owners dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/permissions/update | Mettez à jour la propriété servicePrincipals.permissions dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/policies/update | Mettez à jour la propriété servicePrincipals.policies dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/synchronizationJobs/manage | Gérez tous les aspects des tâches de synchronisation dans Azure AD. |
| microsoft.directory/servicePrincipals/synchronizationSchema/manage | Gérez tous les aspects du schéma de synchronisation dans Azure AD. |
| microsoft.directory/servicePrincipals/synchronizationCredentials/manage | Gérez tous les aspects des informations d’identification de synchronisation dans Azure AD. |
| microsoft.directory/servicePrincipals/tag/update | Mettez à jour la propriété servicePrincipals.tag dans Azure Active Directory. |
| microsoft.directory/signInReports/allProperties/read | Lisez toutes les propriétés (y compris les propriétés privilégiées) sur signInReports dans Azure Active Directory. |
| microsoft.office365.messageCenter/messages/read | Lisez les messages dans microsoft.office365.messageCenter. |
| microsoft.office365.serviceHealth/allEntities/allTasks | Lisez et configurez Microsoft 365 Service Health. |
| microsoft.office365.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Office 365. |

### <a name="insights-administrator-permissions"></a>Autorisations du rôle Administrateur Insights

Dispose d’un accès administratif dans l’application Microsoft 365 Insights. 

| **Actions** | **Description** |
| --- | --- |
| microsoft.azure.serviceHealth/allEntities/allTasks | Lisez et configurez Azure Service Health. |
| microsoft.azure.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Azure pour les services au niveau de l’annuaire. |
| microsoft.insights/allEntities/allTasks | Gérez tous les aspects d’Insights. |
| microsoft.office365.serviceHealth/allEntities/allTasks | Lisez et configurez Microsoft 365 Service Health. |
| microsoft.office365.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Office 365. |
| microsoft.office365.webPortal/allEntities/basic/read | Lisez les propriétés de base sur toutes les ressources dans microsoft.office365.webPortal. |

### <a name="insights-business-leader-permissions"></a>Autorisations du rôle Leader d’entreprise Insights

Peut voir et partager des tableaux de bord et des insights par le biais de l’application M365 Insights.

| **Actions** | **Description** |
| --- | --- |
| microsoft.insights/reports/read | Visualisez des rapports et le tableau de bord dans l’application Insights. |
| microsoft.insights/programs/update | Déployez et gérez des programmes dans l’application Insights. |

### <a name="intune-service-administrator-permissions"></a>Autorisations d’administrateur de services Intune

Peut gérer tous les aspects du produit Intune.

> [!NOTE]
> Ce rôle dispose d’autorisations supplémentaires en dehors d’Azure Active Directory. Pour plus d’informations, consultez la description des rôles ci-dessus.
>
>

| **Actions** | **Description** |
| --- | --- |
| microsoft.directory/bitlockerKeys/key/read | Lisez les objets et les propriétés des clés BitLocker (y compris la clé de récupération) dans Azure Active Directory. |
| microsoft.directory/contacts/basic/update | Mettez à jour des propriétés de base sur des contacts dans Azure Active Directory. |
| microsoft.directory/contacts/create | Créez des contacts dans Azure Active Directory. |
| microsoft.directory/contacts/delete | Supprimez des contacts dans Azure Active Directory. |
| microsoft.directory/devices/basic/update | Mettez à jour des propriétés de base sur des appareils dans Azure Active Directory. |
| microsoft.directory/devices/create | Créez des appareils dans Azure Active Directory. |
| microsoft.directory/devices/delete | Supprimez des appareils dans Azure Active Directory. |
| microsoft.directory/devices/disable | Désactivez des appareils dans Azure Active Directory. |
| microsoft.directory/devices/enable | Activez des appareils dans Azure Active Directory. |
| microsoft.directory/devices/extensionAttributes/update | Mettez à jour toutes les valeurs de la propriété devices.extensionAttributes dans Azure Active Directory. |
| microsoft.directory/devices/registeredOwners/update | Mettez à jour la propriété devices.registeredOwners dans Azure Active Directory. |
| microsoft.directory/devices/registeredUsers/update | Mettez à jour la propriété devices.registeredUsers dans Azure Active Directory. |
| microsoft.directory/groups/appRoleAssignments/update | Mettez à jour la propriété groups.appRoleAssignments dans Azure Active Directory. |
| microsoft.directory/groups/basic/update | Mettez à jour des propriétés de base sur des groupes dans Azure Active Directory. |
| microsoft.directory/groups/create | Créez des groupes dans Azure Active Directory. |
| microsoft.directory/groups/createAsOwner | Créez des groupes dans Azure Active Directory. Le créateur est ajouté comme premier propriétaire, et l’objet créé compte dans le quota de 250 objets créés du créateur. |
| microsoft.directory/groups/delete | Supprimez des groupes dans Azure Active Directory. |
| microsoft.directory/groups/hiddenMembers/read | Lisez la propriété groups.hiddenMembers dans Azure Active Directory. |
| microsoft.directory/groups/members/update | Mettez à jour la propriété groups.members dans Azure Active Directory. |
| microsoft.directory/groups/owners/update | Mettez à jour la propriété groups.owners dans Azure Active Directory. |
| microsoft.directory/groups/restore | Restaurez des groupes dans Azure Active Directory. |
| microsoft.directory/groups/settings/update | Mettez à jour la propriété groups.settings dans Azure Active Directory. |
| microsoft.directory/users/appRoleAssignments/update | Mettez à jour la propriété users.appRoleAssignments dans Azure Active Directory. |
| microsoft.directory/users/basic/update | Mettez à jour des propriétés de base sur des utilisateurs dans Azure Active Directory. |
| microsoft.directory/users/manager/update | Mettez à jour la propriété users.manager dans Azure Active Directory. |
| microsoft.azure.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Azure pour les services au niveau de l’annuaire. |
| microsoft.intune/allEntities/allTasks | Gérez tous les aspects d’Intune. |
| microsoft.office365.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Office 365. |
| microsoft.office365.webPortal/allEntities/basic/read | Lisez les propriétés de base sur toutes les ressources dans microsoft.office365.webPortal. |

### <a name="kaizala-administrator-permissions"></a>Autorisations d’administrateur Kaizala

Peut gérer les paramètres de Microsoft Kaizala.

> [!NOTE]
> Ce rôle dispose d’autorisations supplémentaires en dehors d’Azure Active Directory. Pour plus d’informations, consultez la description des rôles ci-dessus.
>
>

| **Actions** | **Description** |
| --- | --- |
| microsoft.office365.serviceHealth/allEntities/allTasks | Lisez et configurez Microsoft 365 Service Health. |
| microsoft.office365.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Office 365. |
| microsoft.office365.webPortal/allEntities/basic/read | Consultez le Centre d’administration Microsoft 365. |

### <a name="license-administrator-permissions"></a>Autorisations d’administrateur de licence

Peut gérer les licences de produit pour les utilisateurs et les groupes.

| **Actions** | **Description** |
| --- | --- |
| microsoft.directory/users/assignLicense | Gérez les licences sur des utilisateurs dans Azure Active Directory. |
| microsoft.directory/users/usageLocation/update | Mettez à jour la propriété users.usageLocation dans Azure Active Directory. |
| microsoft.azure.serviceHealth/allEntities/allTasks | Lisez et configurez Azure Service Health. |
| microsoft.office365.webPortal/allEntities/basic/read | Lisez les propriétés de base sur toutes les ressources dans microsoft.office365.webPortal. |
| microsoft.office365.serviceHealth/allEntities/allTasks | Lisez et configurez Microsoft 365 Service Health. |

### <a name="lync-service-administrator-permissions"></a>Autorisations d’administrateur des services Lync

Peut gérer tous les aspects du produit Skype Entreprise.

> [!NOTE]
> Ce rôle dispose d’autorisations supplémentaires en dehors d’Azure Active Directory. Pour plus d’informations, consultez la description des rôles ci-dessus.
>
>

| **Actions** | **Description** |
| --- | --- |
| microsoft.azure.serviceHealth/allEntities/allTasks | Lisez et configurez Azure Service Health. |
| microsoft.azure.supportTickets/allEntities/allTasks | Créez et gérez les tickets de support Azure. |
| microsoft.office365.serviceHealth/allEntities/allTasks | Lisez et configurez Microsoft 365 Service Health. |
| microsoft.office365.skypeForBusiness/allEntities/allTasks | Gérez tous les aspects de Skype Entreprise Online. |
| microsoft.office365.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Office 365. |
| microsoft.office365.usageReports/allEntities/read    | Lisez des rapports d’utilisation Office 365. |
| microsoft.office365.webPortal/allEntities/basic/read | Lisez les propriétés de base sur toutes les ressources dans microsoft.office365.webPortal. |


### <a name="message-center-privacy-reader-permissions"></a>Autorisations de lecteur de confidentialité du Centre de messages

Peut lire les billets du Centre de messages, les messages de confidentialité des données, les groupes, les domaines et les abonnements.

> [!NOTE]
> Ce rôle dispose d’autorisations supplémentaires en dehors d’Azure Active Directory. Pour plus d’informations, consultez la description des rôles ci-dessus.
>
>

| **Actions** | **Description** |
| --- | --- |
| microsoft.office365.webPortal/allEntities/basic/read | Lisez les propriétés de base sur toutes les ressources dans microsoft.office365.webPortal. |
| microsoft.office365.messageCenter/messages/read | Lisez les messages dans microsoft.office365.messageCenter. |
| microsoft.office365.messageCenter/securityMessages/read | Lisez les messages securityMessages dans microsoft.office365.messageCenter. |

### <a name="message-center-reader-permissions"></a>Autorisations de lecteur du Centre de messages
Peut lire les messages et les mises à jour de son organisation dans le Centre de messages uniquement. 

> [!NOTE]
> Ce rôle dispose d’autorisations supplémentaires en dehors d’Azure Active Directory. Pour plus d’informations, consultez la description des rôles ci-dessus.
>
>

| **Actions** | **Description** |
| --- | --- |
| microsoft.office365.webPortal/allEntities/basic/read | Lisez les propriétés de base sur toutes les ressources dans microsoft.office365.webPortal. |
| microsoft.office365.messageCenter/messages/read | Lisez les messages dans microsoft.office365.messageCenter. |

### <a name="modern-commerce-user-permissions"></a>Autorisations du rôle Utilisateur de Commerce moderne
Peut gérer les achats commerciaux d’une société, d’un service ou d’une équipe. 

> [!NOTE]
> Ce rôle dispose d’autorisations supplémentaires en dehors d’Azure Active Directory. Pour plus d’informations, consultez la description des rôles ci-dessus.
>
>

| **Actions** | **Description** |
| --- | --- |
| microsoft.commerce.billing/partners/read | Consultez la propriété du partenaire de facturation Microsoft 365. |
| microsoft.commerce.volumeLicenseServiceCenter/allEntities/allTasks | Gérez tous les aspects du Centre de services de licences en volume. |
| microsoft.office365.supportTickets/allEntities/allTasks | Créez et consultez vos propres tickets de support Office 365. |
| microsoft.office365.webPortal/allEntities/basic/read | Lisez les propriétés de base sur toutes les ressources dans microsoft.office365.webPortal. |


### <a name="network-administrator-permissions"></a>Autorisations d’administrateur réseau
Peut gérer les emplacements réseau et passer en revue les insights sur la conception réseau de l’entreprise pour les applications Software as a Service Microsoft 365.

> [!NOTE]
> Ce rôle dispose d’autorisations supplémentaires en dehors d’Azure Active Directory. Pour plus d’informations, consultez la description des rôles ci-dessus.
>
>

| **Actions** | **Description** |
| --- | --- |
| microsoft.office365.network/performance/allProperties/read | Lisez les pages sur les performances réseau dans le Centre d’administration M365.  |
| microsoft.office365.network/locations/allProperties/allTasks | Lisez et configurez les propriétés des emplacements réseau pour chaque emplacement. |

### <a name="office-apps-administrator-permissions"></a>Autorisations d’Administrateur des applications Office
Peut gérer les services cloud des applications Office, notamment la gestion des stratégies et des paramètres, et gérer la possibilité de sélectionner, de désélectionner et de publier le contenu de la fonctionnalité « Nouveautés » sur les appareils de l’utilisateur final.

> [!NOTE]
> Ce rôle dispose d’autorisations supplémentaires en dehors d’Azure Active Directory. Pour plus d’informations, consultez la description des rôles ci-dessus.
>
>

| **Actions** | **Description** |
| --- | --- |
| microsoft.azure.serviceHealth/allEntities/allTasks | Lisez et configurez Azure Service Health. |
| microsoft.azure.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Azure pour les services au niveau de l’annuaire. |
| microsoft.office365.messageCenter/messages/read | Lisez les messages dans microsoft.office365.messageCenter. |
| microsoft.office365.serviceHealth/allEntities/allTasks | Lisez et configurez Microsoft 365 Service Health. |
| microsoft.office365.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Office 365. |
| microsoft.office365.userCommunication/allEntities/allTasks | Lisez et mettez à jour la visibilité des messages Nouveautés. |
| microsoft.office365.webPortal/allEntities/basic/read | Lisez les propriétés de base sur toutes les ressources dans microsoft.office365.webPortal. |

### <a name="partner-tier1-support-permissions"></a>Autorisations de prise en charge de niveau 1 de partenaire

Ne pas utiliser - non destiné à une utilisation générale.

> [!NOTE]
> Ce rôle dispose d’autorisations supplémentaires en dehors d’Azure Active Directory. Pour plus d’informations, consultez la description des rôles ci-dessus.
>
>

| **Actions** | **Description** |
| --- | --- |
| microsoft.directory/contacts/basic/update | Mettez à jour des propriétés de base sur des contacts dans Azure Active Directory. |
| microsoft.directory/contacts/create | Créez des contacts dans Azure Active Directory. |
| microsoft.directory/contacts/delete | Supprimez des contacts dans Azure Active Directory. |
| microsoft.directory/groups/create | Créez des groupes dans Azure Active Directory. |
| microsoft.directory/groups/createAsOwner | Créez des groupes dans Azure Active Directory. Le créateur est ajouté comme premier propriétaire, et l’objet créé compte dans le quota de 250 objets créés du créateur. |
| microsoft.directory/groups/members/update | Mettez à jour la propriété groups.members dans Azure Active Directory. |
| microsoft.directory/groups/owners/update | Mettez à jour la propriété groups.owners dans Azure Active Directory. |
| microsoft.directory/users/appRoleAssignments/update | Mettez à jour la propriété users.appRoleAssignments dans Azure Active Directory. |
| microsoft.directory/users/assignLicense | Gérez les licences sur des utilisateurs dans Azure Active Directory. |
| microsoft.directory/users/basic/update | Mettez à jour des propriétés de base sur des utilisateurs dans Azure Active Directory. |
| microsoft.directory/users/delete | Supprimez des utilisateurs dans Azure Active Directory. |
| microsoft.directory/users/invalidateAllRefreshTokens | Invalidez tous les jetons d’actualisation utilisateur dans Azure Active Directory. |
| microsoft.directory/users/manager/update | Mettez à jour la propriété users.manager dans Azure Active Directory. |
| microsoft.directory/users/password/update | Mettez à jour des mots de passe pour tous les utilisateurs dans Azure Active Directory. Pour plus d’informations, consultez la documentation en ligne. |
| microsoft.directory/users/restore | Restaurez des utilisateurs (Users) supprimés dans Azure Active Directory. |
| microsoft.directory/users/userPrincipalName/update | Mettez à jour la propriété users.userPrincipalName dans Azure Active Directory. |
| microsoft.azure.serviceHealth/allEntities/allTasks | Lisez et configurez Azure Service Health. |
| microsoft.azure.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Azure pour les services au niveau de l’annuaire. |
| microsoft.office365.webPortal/allEntities/basic/read | Lisez les propriétés de base sur toutes les ressources dans microsoft.office365.webPortal. |
| microsoft.office365.serviceHealth/allEntities/allTasks | Lisez et configurez Microsoft 365 Service Health. |
| microsoft.office365.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Office 365. |

### <a name="partner-tier2-support-permissions"></a>Autorisations de prise en charge de niveau 2 de partenaire

Ne pas utiliser - non destiné à une utilisation générale.

> [!NOTE]
> Ce rôle dispose d’autorisations supplémentaires en dehors d’Azure Active Directory. Pour plus d’informations, consultez la description des rôles ci-dessus.
>
>

| **Actions** | **Description** |
| --- | --- |
| microsoft.directory/contacts/basic/update | Mettez à jour des propriétés de base sur des contacts dans Azure Active Directory. |
| microsoft.directory/contacts/create | Créez des contacts dans Azure Active Directory. |
| microsoft.directory/contacts/delete | Supprimez des contacts dans Azure Active Directory. |
| microsoft.directory/domains/allTasks | Créez et supprimez des domaines, et lisez et mettez à jour des propriétés standard dans Azure Active Directory. |
| microsoft.directory/groups/create | Créez des groupes dans Azure Active Directory. |
| microsoft.directory/groups/delete | Supprimez des groupes dans Azure Active Directory. |
| microsoft.directory/groups/members/update | Mettez à jour la propriété groups.members dans Azure Active Directory. |
| microsoft.directory/groups/restore | Restaurez des groupes dans Azure Active Directory. |
| microsoft.directory/organization/basic/update | Mettez à jour des propriétés de base sur une organisation dans Azure Active Directory. |
| microsoft.directory/users/appRoleAssignments/update | Mettez à jour la propriété users.appRoleAssignments dans Azure Active Directory. |
| microsoft.directory/users/assignLicense | Gérez les licences sur des utilisateurs dans Azure Active Directory. |
| microsoft.directory/users/basic/update | Mettez à jour des propriétés de base sur des utilisateurs dans Azure Active Directory. |
| microsoft.directory/users/delete | Supprimez des utilisateurs dans Azure Active Directory. |
| microsoft.directory/users/invalidateAllRefreshTokens | Invalidez tous les jetons d’actualisation utilisateur dans Azure Active Directory. |
| microsoft.directory/users/manager/update | Mettez à jour la propriété users.manager dans Azure Active Directory. |
| microsoft.directory/users/password/update | Mettez à jour des mots de passe pour tous les utilisateurs dans Azure Active Directory. Pour plus d’informations, consultez la documentation en ligne. |
| microsoft.directory/users/restore | Restaurez des utilisateurs (Users) supprimés dans Azure Active Directory. |
| microsoft.directory/users/userPrincipalName/update | Mettez à jour la propriété users.userPrincipalName dans Azure Active Directory. |
| microsoft.azure.serviceHealth/allEntities/allTasks | Lisez et configurez Azure Service Health. |
| microsoft.azure.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Azure pour les services au niveau de l’annuaire. |
| microsoft.office365.webPortal/allEntities/basic/read | Lisez les propriétés de base sur toutes les ressources dans microsoft.office365.webPortal. |
| microsoft.office365.serviceHealth/allEntities/allTasks | Lisez et configurez Microsoft 365 Service Health. |
| microsoft.office365.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Office 365. |

### <a name="password-administrator-permissions"></a>Autorisations d’administrateur de mots de passe

Peut réinitialiser les mots de passe pour les utilisateurs non administrateurs et les administrateurs de mot de passe.

| **Actions** | **Description** |
| --- | --- |
| microsoft.directory/users/password/update | Mettez à jour des mots de passe pour tous les utilisateurs dans Azure Active Directory. Pour plus d’informations, consultez la documentation en ligne. |
| microsoft.office365.webPortal/allEntities/basic/read | Lisez les propriétés de base sur toutes les ressources dans microsoft.office365.webPortal. |

### <a name="power-bi-service-administrator-permissions"></a>Autorisations d’administrateur de service Power BI

Peut gérer tous les aspects du produit Power BI.

> [!NOTE]
> Ce rôle dispose d’autorisations supplémentaires en dehors d’Azure Active Directory. Pour plus d’informations, consultez la description des rôles ci-dessus.
>
>
| **Actions** | **Description** |
| --- | --- |
| microsoft.azure.serviceHealth/allEntities/allTasks | Lisez et configurez Azure Service Health. |
| microsoft.azure.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Azure pour les services au niveau de l’annuaire. |
| microsoft.powerApps.powerBI/allEntities/allTasks | Gérez tous les aspects de Power BI. |
| microsoft.office365.webPortal/allEntities/basic/read | Lisez les propriétés de base sur toutes les ressources dans microsoft.office365.webPortal. |
| microsoft.office365.serviceHealth/allEntities/allTasks | Lisez et configurez Microsoft 365 Service Health. |
| microsoft.office365.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Office 365. |


### <a name="power-platform-administrator-permissions"></a>Autorisations de l’administrateur de plateforme Power

Peut créer et gérer tous les aspects de Microsoft Dynamics 365, PowerApps et Power Automate.

> [!NOTE]
> Ce rôle dispose d’autorisations supplémentaires en dehors d’Azure Active Directory. Pour plus d’informations, consultez la description des rôles ci-dessus.
>
>
| **Actions** | **Description** |
| --- | --- |
| microsoft.azure.serviceHealth/allEntities/allTasks | Lisez et configurez Azure Service Health. |
| microsoft.azure.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Azure pour les services au niveau de l’annuaire. |
| microsoft.dynamics365/allEntities/allTasks | Gérez tous les aspects de Dynamics 365. |
| microsoft.flow/allEntities/allTasks | Gérez tous les aspects de Power Automate. |
| microsoft.powerApps/allEntities/allTasks | Gérez tous les aspects de PowerApps. |
| microsoft.office365.webPortal/allEntities/basic/read | Lisez les propriétés de base sur toutes les ressources dans microsoft.office365.webPortal. |
| microsoft.office365.serviceHealth/allEntities/allTasks | Lisez et configurez Microsoft 365 Service Health. |
| microsoft.office365.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Office 365. |

### <a name="printer-administrator-permissions"></a>Autorisations d’administrateur d’imprimantes

Peut gérer tous les aspects des imprimantes et des connecteurs d’imprimantes.

> [!NOTE]
> Ce rôle dispose d’autorisations supplémentaires en dehors d’Azure Active Directory. Pour plus d’informations, consultez la description des rôles ci-dessus.
>
>
| **Actions** | **Description** |
| --- | --- |
| microsoft.azure.print/allEntities/allProperties/allTasks | Créez et supprimez des imprimantes et des connecteurs, et lisez et mettez à jour toutes les propriétés dans Microsoft Print. |

### <a name="printer-technician-permissions"></a>Autorisations de technicien d’imprimantes

Peut inscrire et désinscrire des imprimantes et mettre à jour l’état de l’imprimante.

> [!NOTE]
> Ce rôle dispose d’autorisations supplémentaires en dehors d’Azure Active Directory. Pour plus d’informations, consultez la description des rôles ci-dessus.
>
>
| **Actions** | **Description** |
| --- | --- |
| microsoft.azure.print/connectors/allProperties/read | Lisez toutes les propriétés des connecteurs dans Microsoft Print. |
| microsoft.azure.print/printers/allProperties/read | Lisez toutes les propriétés des imprimantes dans Microsoft Print. |
| microsoft.azure.print/printers/basic/update | Mettez à jour les propriétés de base des imprimantes dans Microsoft Print. |
| microsoft.azure.print/printers/register | Inscrivez des imprimantes dans Microsoft Print. |
| microsoft.azure.print/printers/unregister | Désinscrivez des imprimantes dans Microsoft Print. |

### <a name="privileged-authentication-administrator-permissions"></a>Autorisations d’administrateur d’authentification privilégié

Autorisé à afficher, définir et réinitialiser des informations de méthode d’authentification pour tout utilisateur (administrateur ou non-administrateur).

| **Actions** | **Description** |
| --- | --- |
| microsoft.directory/users/invalidateAllRefreshTokens | Invalidez tous les jetons d’actualisation utilisateur dans Azure Active Directory. |
| microsoft.directory/users/strongAuthentication/update | Mettez à jour les propriétés d’authentification forte comme les informations d’identification MFA. |
| microsoft.azure.serviceHealth/allEntities/allTasks | Lisez et configurez Azure Service Health. |
| microsoft.azure.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Azure pour les services au niveau de l’annuaire. |
| microsoft.office365.webPortal/allEntities/basic/read | Lisez les propriétés de base sur toutes les ressources dans microsoft.office365.webPortal. |
| microsoft.office365.serviceHealth/allEntities/allTasks | Lisez et configurez Microsoft 365 Service Health. |
| microsoft.office365.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Office 365. |
| microsoft.directory/users/password/update | Mettez à jour les mots de passe de tous les utilisateurs de l’organisation Microsoft 365. Pour plus d’informations, consultez la documentation en ligne. |

### <a name="privileged-role-administrator-permissions"></a>Autorisations d’administrateur de rôle privilégié

Peut gérer les attributions de rôles dans Azure AD et tous les aspects de Privileged Identity Management.

> [!NOTE]
> Ce rôle dispose d’autorisations supplémentaires en dehors d’Azure Active Directory. Pour plus d’informations, consultez la description des rôles ci-dessus.
>
>

| **Actions** | **Description** |
| --- | --- |
| microsoft.directory/groupsAssignableToRoles/allProperties/update | Mettez à jour les groupes dont la propriété isAssignableToRole est définie sur true dans Azure Active Directory. |
| microsoft.directory/groupsAssignableToRoles/create | Créez des groupes dont la propriété isAssignableToRole est définie sur true dans Azure Active Directory. |
| microsoft.directory/groupsAssignableToRoles/delete | Supprimez les groupes dont la propriété isAssignableToRole est définie sur true dans Azure Active Directory. |
| microsoft.directory/privilegedIdentityManagement/allEntities/allTasks | Créez et supprimez toutes les ressources, et lisez et mettez à jour des propriétés standard dans microsoft.aad.privilegedIdentityManagement. |
| microsoft.directory/servicePrincipals/appRoleAssignedTo/allTasks | Lisez et configurez la propriété servicePrincipals.appRoleAssignedTo dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/oAuth2PermissionGrants/allTasks | Lisez et configurez la propriété servicePrincipals.oAuth2PermissionGrants dans Azure Active Directory. |
| microsoft.directory/administrativeUnits/allProperties/allTasks | Créez et gérez des unités administratives (y compris les membres). |
| microsoft.directory/roleAssignments/allProperties/allTasks | Créez et gérez des attributions de rôle. |
| microsoft.directory/roleDefinitions/allProperties/allTasks | Créez et gérez des définitions de rôle. |

### <a name="reports-reader-permissions"></a>Autorisations de lecteur de rapports

Peut lire les rapports d’audit et sur les connexions.

> [!NOTE]
> Ce rôle dispose d’autorisations supplémentaires en dehors d’Azure Active Directory. Pour plus d’informations, consultez la description des rôles ci-dessus.
>
>

| **Actions** | **Description** |
| --- | --- |
| microsoft.directory/auditLogs/allProperties/read | Lisez toutes les propriétés (y compris les propriétés privilégiées) sur auditLogs dans Azure Active Directory. |
| microsoft.directory/signInReports/allProperties/read | Lisez toutes les propriétés (y compris les propriétés privilégiées) sur signInReports dans Azure Active Directory. |
| microsoft.azure.serviceHealth/allEntities/allTasks | Lisez et configurez Azure Service Health. |
| microsoft.office365.usageReports/allEntities/read | Lisez des rapports d’utilisation Office 365. |

### <a name="search-administrator-permissions"></a>Autorisations d’administrateur de recherche

Peut créer et gérer tous les aspects des paramètres de Recherche Microsoft.

> [!NOTE]
> Ce rôle dispose d’autorisations supplémentaires en dehors d’Azure Active Directory. Pour plus d’informations, consultez la description des rôles ci-dessus.
>
>

| **Actions** | **Description** |
| --- | --- |
| microsoft.office365.messageCenter/messages/read | Lisez les messages dans microsoft.office365.messageCenter. |
| microsoft.office365.search/allEntities/allProperties/allTasks | Créez et supprimez toutes les ressources, et lisez et mettez à jour toutes les propriétés dans microsoft.office365.search. |
| microsoft.office365.serviceHealth/allEntities/allTasks | Lisez et configurez Microsoft 365 Service Health. |
| microsoft.office365.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Office 365. |
| microsoft.office365.webPortal/allEntities/basic/read | Lisez les propriétés de base sur toutes les ressources dans microsoft.office365.webPortal. |

### <a name="search-editor-permissions"></a>Autorisations d’éditeur de recherche

Peut créer et gérer le contenu éditorial comme les signets, les questions et réponses, les emplacements et les plans d’étage.

> [!NOTE]
> Ce rôle dispose d’autorisations supplémentaires en dehors d’Azure Active Directory. Pour plus d’informations, consultez la description des rôles ci-dessus.
>
>

| **Actions** | **Description** |
| --- | --- |
| microsoft.office365.messageCenter/messages/read | Lisez les messages dans microsoft.office365.messageCenter. |
| microsoft.office365.search/content/allProperties/allTasks | Créez et supprimez du contenu, et lisez et mettez à jour toutes les propriétés dans microsoft.office365.search. |

### <a name="security-administrator-permissions"></a>Autorisations d’administrateur de la sécurité

Peut lire des rapports et des informations de sécurité, ainsi que gérer la configuration dans Azure AD et Microsoft 365.

> [!NOTE]
> Ce rôle dispose d’autorisations supplémentaires en dehors d’Azure Active Directory. Pour plus d’informations, consultez la description des rôles ci-dessus.
>
>

| **Actions** | **Description** |
| --- | --- |
| microsoft.azure.serviceHealth/allEntities/allTasks | Lisez et configurez Azure Service Health. |
| microsoft.azure.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Azure pour les services au niveau de l’annuaire. |
| microsoft.directory/bitlockerKeys/key/read | Lisez les objets et les propriétés des clés BitLocker (y compris la clé de récupération) dans Azure Active Directory. |
| microsoft.directory/applications/policies/update | Mettez à jour la propriété applications.policies dans Azure Active Directory. |
| microsoft.directory/auditLogs/allProperties/read | Lisez toutes les propriétés (y compris les propriétés privilégiées) sur auditLogs dans Azure Active Directory. |
| microsoft.directory/identityProtection/allProperties/read | Lisez toutes les ressources dans microsoft.aad.identityProtection. |
| microsoft.directory/identityProtection/allProperties/update | Mettez à jour toutes les ressources dans microsoft.aad.identityProtection. |
| microsoft.directory/policies/basic/update | Mettez à jour des propriétés de base sur des stratégies dans Azure Active Directory. |
| microsoft.directory/policies/create | Créez des stratégies dans Azure Active Directory. |
| microsoft.directory/policies/delete | Supprimez des stratégies dans Azure Active Directory. |
| microsoft.directory/policies/owners/update | Mettez à jour la propriété policies.owners dans Azure Active Directory. |
| microsoft.directory/policies/tenantDefault/update | Mettez à jour la propriété policies.tenantDefault dans Azure Active Directory. |
| microsoft.directory/privilegedIdentityManagement/allProperties/read | Lisez toutes les ressources dans microsoft.aad.privilegedIdentityManagement. |
| microsoft.directory/servicePrincipals/policies/update | Mettez à jour la propriété servicePrincipals.policies dans Azure Active Directory. |
| microsoft.directory/signInReports/allProperties/read | Lisez toutes les propriétés (y compris les propriétés privilégiées) sur signInReports dans Azure Active Directory. |
| microsoft.office365.protectionCenter/allEntities/read | Lisez tous les aspects du Centre de protection Office 365. |
| microsoft.office365.protectionCenter/allEntities/update | Mettez à jour toutes les ressources dans microsoft.office365.protectionCenter. |
| microsoft.office365.serviceHealth/allEntities/allTasks | Lisez et configurez Microsoft 365 Service Health. |
| microsoft.office365.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Office 365. |
| microsoft.office365.webPortal/allEntities/basic/read | Lisez les propriétés de base sur toutes les ressources dans microsoft.office365.webPortal. |

### <a name="security-operator-permissions"></a>Autorisations d’opérateur de sécurité

Crée et gère les événements de sécurité.

> [!NOTE]
> Ce rôle dispose d’autorisations supplémentaires en dehors d’Azure Active Directory. Pour plus d’informations, consultez la description des rôles ci-dessus.
>
>

| **Actions** | **Description** |
| --- | --- |
| microsoft.azure.advancedThreatProtection/allEntities/read | Lisez et configurez Azure AD Advanced Threat Protection. |
| microsoft.azure.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Azure pour les services au niveau de l’annuaire. |
| microsoft.directory/cloudAppSecurity/allProperties/allTasks | Lisez et configurez Microsoft Cloud App Security. |
| microsoft.directory/identityProtection/allProperties/read | Lisez toutes les ressources dans microsoft.aad.identityProtection. |
| microsoft.directory/privilegedIdentityManagement/allProperties/read | Lisez toutes les ressources dans microsoft.aad.privilegedIdentityManagement. |
| microsoft.intune/allEntities/allTasks | Gérez tous les aspects d’Intune. |
| microsoft.office365.securityComplianceCenter/allEntities/allTasks | Lisez et configurez le Centre de sécurité et de conformité. |
| microsoft.office365.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Office 365. |
| microsoft.windows.defenderAdvancedThreatProtection/allEntities/read | Lisez et configurez Windows Defender Advanced Threat Protection. |

### <a name="security-reader-permissions"></a>Autorisations de lecteur de sécurité

Peut lire des rapports et des informations de sécurité dans Azure AD et Microsoft 365.

> [!NOTE]
> Ce rôle dispose d’autorisations supplémentaires en dehors d’Azure Active Directory. Pour plus d’informations, consultez la description des rôles ci-dessus.
>
>

| **Actions** | **Description** |
| --- | --- |
| microsoft.directory/auditLogs/allProperties/read | Lisez toutes les propriétés (y compris les propriétés privilégiées) sur auditLogs dans Azure Active Directory. |
| microsoft.directory/bitlockerKeys/key/read | Lisez les objets et les propriétés des clés BitLocker (y compris la clé de récupération) dans Azure Active Directory. |
| microsoft.directory/policies/conditionalAccess/basic/read | Lisez la propriété policies.conditionalAccess dans Azure Active Directory. |
| microsoft.directory/signInReports/allProperties/read | Lisez toutes les propriétés (y compris les propriétés privilégiées) sur signInReports dans Azure Active Directory. |
| microsoft.aad.identityProtection/allEntities/read | Lisez toutes les ressources dans microsoft.aad.identityProtection. |
| microsoft.aad.privilegedIdentityManagement/allEntities/read | Lisez toutes les ressources dans microsoft.aad.privilegedIdentityManagement. |
| microsoft.azure.serviceHealth/allEntities/allTasks | Lisez et configurez Azure Service Health. |
| microsoft.office365.webPortal/allEntities/basic/read | Lisez les propriétés de base sur toutes les ressources dans microsoft.office365.webPortal. |
| microsoft.office365.protectionCenter/allEntities/read | Lisez tous les aspects du Centre de protection Office 365. |
| microsoft.office365.serviceHealth/allEntities/allTasks | Lisez et configurez Microsoft 365 Service Health. |

### <a name="service-support-administrator-permissions"></a>Autorisations d’administrateur de support de service

Peut lire des informations sur l’intégrité du service et gérer les tickets de support.

> [!NOTE]
> Ce rôle dispose d’autorisations supplémentaires en dehors d’Azure Active Directory. Pour plus d’informations, consultez la description des rôles ci-dessus.
>
>

| **Actions** | **Description** |
| --- | --- |
| microsoft.azure.serviceHealth/allEntities/allTasks | Lisez et configurez Azure Service Health. |
| microsoft.azure.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Azure pour les services au niveau de l’annuaire. |
| microsoft.office365.webPortal/allEntities/basic/read | Lisez les propriétés de base sur toutes les ressources dans microsoft.office365.webPortal. |
| microsoft.office365.serviceHealth/allEntities/allTasks | Lisez et configurez Microsoft 365 Service Health. |
| microsoft.office365.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Office 365. |

### <a name="sharepoint-service-administrator-permissions"></a>Autorisations d’administrateur de service SharePoint

Peut gérer tous les aspects du service SharePoint.

> [!NOTE]
> Ce rôle dispose d’autorisations supplémentaires en dehors d’Azure Active Directory. Pour plus d’informations, consultez la description des rôles ci-dessus.
>
>

| **Actions** | **Description** |
| --- | --- |
| microsoft.azure.serviceHealth/allEntities/allTasks | Lisez et configurez Azure Service Health. |
| microsoft.azure.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Azure pour les services au niveau de l’annuaire. |
| microsoft.directory/groups/unified/appRoleAssignments/update | Mettez à jour la propriété groups.unified dans Azure Active Directory. |
| microsoft.directory/groups/unified/basic/update | Mettez à jour les propriétés de base des groupes Microsoft 365. |
| microsoft.directory/groups/unified/create | Créez des groupes Microsoft 365. |
| microsoft.directory/groups/unified/delete | Supprimez des groupes Microsoft 365. |
| microsoft.directory/groups/unified/members/update | Mettez à jour l’appartenance à des groupes Microsoft 365. |
| microsoft.directory/groups/unified/owners/update | Mettez à jour la propriété des groupes Microsoft 365. |
| microsoft.office365.network/performance/allProperties/read | Lisez les pages sur les performances réseau dans le Centre d’administration M365. |
| microsoft.office365.serviceHealth/allEntities/allTasks | Lisez et configurez Microsoft 365 Service Health. |
| microsoft.office365.sharepoint/allEntities/allTasks | Créez et supprimez toutes les ressources, et lisez et mettez à jour des propriétés standard dans microsoft.office365.sharepoint. |
| microsoft.office365.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Office 365. |
| microsoft.office365.usageReports/allEntities/read    | Lisez des rapports d’utilisation Office 365. |
| microsoft.office365.webPortal/allEntities/basic/read | Lisez les propriétés de base sur toutes les ressources dans microsoft.office365.webPortal. |

### <a name="teams-communications-administrator-permissions"></a>Autorisations d’administrateur des communications Teams

Peut gérer les fonctionnalités d’appel et de réunion au sein du service Microsoft Teams.

> [!NOTE]
> Ce rôle dispose d’autorisations supplémentaires en dehors d’Azure Active Directory. Pour plus d’informations, consultez la description des rôles ci-dessus.
>
>

| **Actions** | **Description** |
| --- | --- |
| microsoft.azure.serviceHealth/allEntities/allTasks | Lisez et configurez Azure Service Health. |
| microsoft.azure.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Azure pour les services au niveau de l’annuaire. |
| microsoft.office365.webPortal/allEntities/basic/read | Lisez les propriétés de base sur toutes les ressources dans microsoft.office365.webPortal. |
| microsoft.office365.serviceHealth/allEntities/allTasks | Lisez et configurez Microsoft 365 Service Health. |
| microsoft.office365.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Office 365. |
| microsoft.office365.usageReports/allEntities/read | Lisez des rapports d’utilisation Office 365. |
| microsoft.teams/meetings/allProperties/allTasks | Gérez les réunions, notamment les stratégies de réunion, les configurations et les passerelles de conférence. |
| microsoft.teams/voice/allProperties/allTasks | Gérez la voix, notamment les stratégies d’appel, l’inventaire et l’attribution des numéros de téléphone. |
| microsoft.teams/callQuality/allProperties/read | Lisez toutes les données du Tableau de bord Qualité de l’appel (CQD). |

### <a name="teams-communications-support-engineer-permissions"></a>Autorisations d’ingénieur de support des communications Teams

Peut résoudre les problèmes de communication au sein de Teams à l’aide d’outils avancés.

> [!NOTE]
> Ce rôle dispose d’autorisations supplémentaires en dehors d’Azure Active Directory. Pour plus d’informations, consultez la description des rôles ci-dessus.
>
>

| **Actions** | **Description** |
| --- | --- |
| microsoft.azure.serviceHealth/allEntities/allTasks | Lisez et configurez Azure Service Health. |
| microsoft.office365.webPortal/allEntities/basic/read | Lisez les propriétés de base sur toutes les ressources dans microsoft.office365.webPortal. |
| microsoft.office365.serviceHealth/allEntities/allTasks | Lisez et configurez Microsoft 365 Service Health. |
| microsoft.teams/callQuality/allProperties/read | Lisez toutes les données du Tableau de bord Qualité de l’appel (CQD). |

### <a name="teams-communications-support-specialist-permissions"></a>Autorisations de spécialiste du support des communications Teams

Peut résoudre les problèmes de communication au sein de Teams à l’aide d’outils de base.

> [!NOTE]
> Ce rôle dispose d’autorisations supplémentaires en dehors d’Azure Active Directory. Pour plus d’informations, consultez la description des rôles ci-dessus.
>
>

| **Actions** | **Description** |
| --- | --- |
| microsoft.azure.serviceHealth/allEntities/allTasks | Lisez et configurez Azure Service Health. |
| microsoft.office365.webPortal/allEntities/basic/read | Lisez les propriétés de base sur toutes les ressources dans microsoft.office365.webPortal. |
| microsoft.office365.serviceHealth/allEntities/allTasks | Lisez et configurez Microsoft 365 Service Health. |
| microsoft.teams/callQuality/basic/read | Lisez les données de base du Tableau de bord Qualité de l’appel (CQD). |

### <a name="teams-devices-administrator-permissions"></a>Autorisations d’administrateur d’appareil Teams

Peut effectuer des tâches d’administration sur des appareils certifiés Teams.

> [!NOTE]
> Ce rôle dispose d’autorisations supplémentaires en dehors d’Azure Active Directory. Pour plus d’informations, consultez la description des rôles ci-dessus.
>
>

| **Actions** | **Description** |
| --- | --- |
| microsoft.office365.webPortal/allEntities/basic/read | Lisez les propriétés de base sur toutes les ressources dans microsoft.office365.webPortal. |
| microsoft.teams/devices/basic/read | Gérez tous les aspects des appareils certifiés Teams, notamment les stratégies de configuration. |

### <a name="teams-service-administrator-permissions"></a>Autorisations d’administrateur de service Teams

Peut gérer le service Microsoft Teams.

> [!NOTE]
> Ce rôle dispose d’autorisations supplémentaires en dehors d’Azure Active Directory. Pour plus d’informations, consultez la description des rôles ci-dessus.
>
>

| **Actions** | **Description** |
| --- | --- |
| microsoft.azure.serviceHealth/allEntities/allTasks | Lisez et configurez Azure Service Health. |
| microsoft.azure.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Azure pour les services au niveau de l’annuaire. |
| microsoft.directory/groups/hiddenMembers/read | Lisez la propriété groups.hiddenMembers dans Azure Active Directory. |
| microsoft.directory/groups/unified/appRoleAssignments/update | Mettez à jour la propriété groups.unified dans Azure Active Directory. |
| microsoft.directory/groups/unified/basic/update | Mettez à jour les propriétés de base des groupes Microsoft 365. |
| microsoft.directory/groups/unified/create | Créez des groupes Microsoft 365. |
| microsoft.directory/groups/unified/delete | Supprimez des groupes Microsoft 365. |
| microsoft.directory/groups/unified/members/update | Mettez à jour l’appartenance à des groupes Microsoft 365. |
| microsoft.directory/groups/unified/owners/update | Mettez à jour la propriété des groupes Microsoft 365. |
| microsoft.office365.network/performance/allProperties/read | Lisez les pages sur les performances réseau dans le Centre d’administration M365. |
| microsoft.office365.serviceHealth/allEntities/allTasks | Lisez et configurez Microsoft 365 Service Health. |
| microsoft.office365.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Office 365. |
| microsoft.office365.usageReports/allEntities/read | Lisez des rapports d’utilisation Office 365. |
| microsoft.office365.webPortal/allEntities/basic/read | Lisez les propriétés de base sur toutes les ressources dans microsoft.office365.webPortal. |
| microsoft.teams/allEntities/allProperties/allTasks | Gérez toutes les ressources dans Teams. |

### <a name="user-administrator-permissions"></a>Autorisations d’administrateur d’utilisateurs
Peut gérer tous les aspects des utilisateurs et groupes, notamment la réinitialisation des mots de passe pour les administrateurs limités.

| **Actions** | **Description** |
| --- | --- |
| microsoft.directory/appRoleAssignments/create | Créez des attributions de rôles d’applications dans Azure Active Directory. |
| microsoft.directory/appRoleAssignments/delete | Supprimez des attributions de rôles d’applications dans Azure Active Directory. |
| microsoft.directory/appRoleAssignments/update | Mettez à jour des attributions de rôles d’applications dans Azure Active Directory. |
| microsoft.directory/contacts/basic/update | Mettez à jour des propriétés de base sur des contacts dans Azure Active Directory. |
| microsoft.directory/contacts/create | Créez des contacts dans Azure Active Directory. |
| microsoft.directory/contacts/delete | Supprimez des contacts dans Azure Active Directory. |
| microsoft.directory/groups/appRoleAssignments/update | Mettez à jour la propriété groups.appRoleAssignments dans Azure Active Directory. |
| microsoft.directory/groups/basic/update | Mettez à jour des propriétés de base sur des groupes dans Azure Active Directory. |
| microsoft.directory/groups/create | Créez des groupes dans Azure Active Directory. |
| microsoft.directory/groups/createAsOwner | Créez des groupes dans Azure Active Directory. Le créateur est ajouté comme premier propriétaire, et l’objet créé compte dans le quota de 250 objets créés du créateur. |
| microsoft.directory/groups/delete | Supprimez des groupes dans Azure Active Directory. |
| microsoft.directory/groups/hiddenMembers/read | Lisez la propriété groups.hiddenMembers dans Azure Active Directory. |
| microsoft.directory/groups/members/update | Mettez à jour la propriété groups.members dans Azure Active Directory. |
| microsoft.directory/groups/owners/update | Mettez à jour la propriété groups.owners dans Azure Active Directory. |
| microsoft.directory/groups/restore | Restaurez des groupes dans Azure Active Directory. |
| microsoft.directory/groups/settings/update | Mettez à jour la propriété groups.settings dans Azure Active Directory. |
| microsoft.directory/users/appRoleAssignments/update | Mettez à jour la propriété users.appRoleAssignments dans Azure Active Directory. |
| microsoft.directory/users/assignLicense | Gérez les licences sur des utilisateurs dans Azure Active Directory. |
| microsoft.directory/users/basic/update | Mettez à jour des propriétés de base sur des utilisateurs dans Azure Active Directory. |
| microsoft.directory/users/create | Créez des utilisateurs dans Azure Active Directory. |
| microsoft.directory/users/delete | Supprimez des utilisateurs dans Azure Active Directory. |
| microsoft.directory/users/invalidateAllRefreshTokens | Invalidez tous les jetons d’actualisation utilisateur dans Azure Active Directory. |
| microsoft.directory/users/manager/update | Mettez à jour la propriété users.manager dans Azure Active Directory. |
| microsoft.directory/users/password/update | Mettez à jour des mots de passe pour tous les utilisateurs dans Azure Active Directory. Pour plus d’informations, consultez la documentation en ligne. |
| microsoft.directory/users/restore | Restaurez des utilisateurs (Users) supprimés dans Azure Active Directory. |
| microsoft.directory/users/userPrincipalName/update | Mettez à jour la propriété users.userPrincipalName dans Azure Active Directory. |
| microsoft.azure.serviceHealth/allEntities/allTasks | Lisez et configurez Azure Service Health. |
| microsoft.azure.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Azure pour les services au niveau de l’annuaire. |
| microsoft.office365.webPortal/allEntities/basic/read | Lisez les propriétés de base sur toutes les ressources dans microsoft.office365.webPortal. |
| microsoft.office365.serviceHealth/allEntities/allTasks | Lisez et configurez Microsoft 365 Service Health. |
| microsoft.office365.supportTickets/allEntities/allTasks | Créez et gérez des tickets de support Office 365. |

## <a name="role-template-ids"></a>ID des modèles de rôle

Les ID des modèles de rôle sont principalement utilisés par les utilisateurs de l’API Microsoft Graph ou de PowerShell.

DisplayName Graph | Nom d’affichage du portail Azure | directoryRoleTemplateId
----------------- | ------------------------- | -------------------------
Administrateur d’application | Administrateur d’application | 9B895D92-2CD3-44C7-9D02-A6AC2D5EA5C3
Développeur d’applications | Développeur d’applications | CF1C38E5-3621-4004-A7CB-879624DCED7C
Administrateur d’authentification | Administrateur d’authentification | c4e39bd9-1100-46d3-8c65-fb160da0071f
Administrateur Azure DevOps | Administrateur Azure DevOps | e3973bdf-4987-49ae-837a-ba8e231c7286
Administrateur Azure Information Protection | Administrateur Azure Information Protection | 7495fdc4-34c4-4d15-a289-98788ce399fd
Administrateur de jeux de clés B2C IEF | Administrateur de jeux de clés B2C IEF | aaf43236-0c0d-4d5f-883a-6955382ac081
Administrateur de stratégies B2C IEF | Administrateur de stratégies B2C IEF | 3edaf663-341e-4475-9f94-5c398ef6c070
Administrateur de facturation | Administrateur de facturation | b0f54661-2d74-4c50-afa3-1ec803f12efe
Administrateur d'applications cloud | Administrateur d’application cloud | 158c047a-c907-4556-b7ef-446551a6b5f7
Administrateur d’appareil cloud | Administrateur d’appareil cloud | 7698a772-787b-4ac8-901f-60d6b08affd2
Administrateur d’entreprise | Administrateur général | 62e90394-69f5-4237-9190-012177145e10
Administrateur de conformité | Administrateur de conformité | 17315797-102d-40b4-93e0-432062caca18
Administrateur des données de conformité | Administrateur des données de conformité | e6d1a23a-da11-4be4-9570-befc86d067a7
Administrateur de l’accès conditionnel | Administrateur de l’accès conditionnel | b1be1c3e-b65d-4f19-8427-f6fa0d97feb9
Administrateur de services CRM | Administrateur Dynamics 365 | 44367163-eba1-44c3-98af-f5787879f96a
Approbateur d’accès à Customer LockBox | Approbateur d’accès à Customer LockBox | 5c4f9dcd-47dc-4cf7-8c9a-9e4207cbfc91
Administrateur Desktop Analytics | Administrateur Desktop Analytics | 38a96431-2bdf-4b4c-8b6e-5d3d8abac1a4
Administrateurs d’appareils | Administrateurs d’appareils | 9f06204d-73c1-4d4c-880a-6edb90606fd8
Jonction d’appareils | Déprécié | 9c094953-4995-41c8-84c8-3ebb9b32c93f
Gestionnaires d’appareils | Déprécié | 2b499bcd-da44-4968-8aec-78e1674fa64d
Utilisateurs d’appareils | Déprécié | d405c6df-0af8-4e3b-95e4-4d06e542189e
Lecteurs de répertoires | Lecteurs d’annuaires | 88d8e3e3-8f55-4a1e-953a-9b9898b8876b
Comptes de synchronisation d’annuaires | Non affiché, car il ne doit pas être utilisé | d29b2b05-8046-44ba-8758-1e26182fcf32
Enregistreurs de répertoire | Enregistreurs de répertoire | 9360feb5-f418-4baa-8175-e2a00bac4301
Administrateur de services Exchange | Administrateur Exchange | 29232cdf-9323-42fd-ade2-1d097af3e4de
Administrateur de flux d’utilisateurs ID externe | Administrateur de flux d’utilisateurs ID externe | 6e591065-9bad-43ed-90f3-e9424366d2f0
Administrateur d’attribut de flux d’utilisateurs ID externe | Administrateur d’attribut de flux d’utilisateurs ID externe | 0f971eea-41eb-4569-a71e-57bb8a3eff1e
Administrateur de fournisseurs d’identité externes | Administrateur de fournisseurs d’identité externes | be2f45a1-457d-42af-a067-6ec1fa63bc45
Lecteur général | Lecteur général | f2ef992c-3afb-46b9-b7cf-a126ee74c451
Administrateur de groupes | Administrateur de groupes | fdd7a751-b60b-444a-984c-02652fe8fa1c 
Inviteur | Inviteur d’invités | 95e79109-95c0-4d8e-aee3-d01accf2d47b
Administrateur du support technique | Administrateur du support technique | 729827e3-9c14-49f7-bb1b-9608f156bbb8
Administrateur d’identité hybride | Administrateur d’identité hybride | 8ac3fc64-6eca-42ea-9e69-59f4c7b60eb2
Administrateur Insights | Administrateur Insights | eb1f4a8d-243a-41f0-9fbd-c7cdf6c5ef7c
Leader d’entreprise Insights | Leader d’entreprise Insights | 31e939ad-9672-4796-9c2e-873181342d2d
Administrateur de services Intune | Administrateur Intune | 3a2c62db-5318-420d-8d74-23affee5d9d5
Administrateur Kaizala | Administrateur Kaizala | 74ef975b-6605-40af-a5d2-b9539d836353
Administrateur de licence | Administrateur de licence | 4d6ac14f-3453-41d0-bef9-a3e0c569773a
Administrateur de services Lync | Administrateur Skype Entreprise | 75941009-915a-4869-abe7-691bff18279e
Lecteur de confidentialité du Centre de messages | Lecteur de confidentialité du Centre de messages | ac16e43d-7b2d-40e0-ac05-243ff356ab5b
Lecteur du Centre de messages | Lecteur du Centre de messages | 790c1fb9-7f7d-4f88-86a1-ef1f95c05c1b
Utilisateur de Commerce moderne | Utilisateur de Commerce moderne | d24aef57-1500-4070-84db-2666f29cf966
Administrateur réseau | Administrateur réseau | d37c8bed-0711-4417-ba38-b4abe66ce4c2
Administrateur d’applications Office | Administrateur d’applications Office | 2b745bdf-0803-4d80-aa65-822c4493daac
Prise en charge de niveau 1 de partenaire | Non affiché, car il ne doit pas être utilisé | 4ba39ca4-527c-499a-b93d-d9b492c50246
Prise en charge de niveau 2 de partenaire | Non affiché, car il ne doit pas être utilisé | e00e864a-17c5-4a4b-9c06-f5b95a8d5bd8
Administrateur de mots de passe | Administrateur de mots de passe | 966707d0-3269-4727-9be2-8c3a10f19b9d
Administrateur du service Power BI | Administrateur Power BI | a9ea8996-122f-4c74-9520-8edcd192826c
Administrateur de plateforme Power | Administrateur de plateforme Power | 11648597-926c-4cf3-9c36-bcebb0ba8dcc
Administrateur d’imprimantes | Administrateur d’imprimantes | 644ef478-e28f-4e28-b9dc-3fdde9aa0b1f
Technicien en charge des imprimantes | Technicien en charge des imprimantes | e8cef6f1-e4bd-4ea8-bc07-4b8d950f4477
Administrateur d’authentification privilégié | Administrateur d’authentification privilégié | 7be44c8a-adaf-4e2a-84d6-ab2649e08a13
Administrateur de rôle privilégié | Administrateur de rôle privilégié | e8611ab8-c189-46e8-94e1-60213ab1f814
Lecteur de rapports | Lecteur de rapports | 4a5d8f65-41da-4de4-8968-e035b65339cf
Administrateur de recherche | Administrateur de recherche | 0964bb5e-9bdb-4d7b-ac29-58e794862a40
Éditeur de recherche | Éditeur de recherche | 8835291a-918c-4fd7-a9ce-faa49f0cf7d9
Security Administrator | Administrateur de sécurité | 194ae4cb-b126-40b2-bd5b-6091b380977d
Opérateur de sécurité | Opérateur de sécurité | 5f2222b1-57c3-48ba-8ad5-d4759f1fde6f
Lecteur de sécurité | Lecteur de sécurité | 5d6b6bb7-de71-4623-b4af-96380a352509
Administrateur de support de service | Administrateur de support de service | f023fd81-a637-4b56-95fd-791ac0226033
Administrateur de services SharePoint | Administrateur SharePoint | f28a1f50-f6e7-4571-818b-6a12f2af6b6c
Administrateur des communications Teams | Administrateur des communications Teams | baf37b3a-610e-45da-9e62-d9d1e5e8914b
Ingénieur de support des communications Teams | Ingénieur de support des communications Teams | f70938a0-fc10-4177-9e90-2178f8765737
Spécialiste du support des communications Teams | Spécialiste du support des communications Teams | fcf91098-03e3-41a9-b5ba-6f0ec8188a12
Administrateur d’appareils Teams | Administrateur d’appareils Teams | 3d762c5a-1b6c-493f-843e-55a3b42923d4
Administrateur du service Teams | Administrateur du service Teams | 69091246-20e8-4a56-aa4d-066075b2a7a8
Utilisateur | Non affiché, car il ne peut pas être utilisé | a0b1b346-4d3e-4e8b-98f8-753987be4970
Administrateur de compte utilisateur | Administrateur d’utilisateurs | fe930be7-5e62-47db-91af-98c3a49a38b1
Jonction d’appareils d’espace de travail | Déprécié | c34f683f-4d5a-4403-affd-6615e00e3a7f

## <a name="deprecated-roles"></a>Rôles déconseillés

Les rôles suivants ne doivent pas être utilisés. Ils sont déconseillés et seront prochainement supprimés d’Azure AD.

* Administrateur de licences ad hoc
* Jonction d’appareils
* Gestionnaires d’appareils
* Utilisateurs d’appareils
* Créateur d’utilisateur vérifié par e-mail
* Administrateur de boîte aux lettres
* Jonction d’appareils d’espace de travail

## <a name="roles-not-shown-in-the-portal"></a>Rôles non affichés dans le portail

Les rôles retournés par PowerShell ou l’API MS Graph ne sont pas tous visibles dans le portail Azure. Le tableau suivant organise ces différences.

Nom de l’API | Nom du portail Azure | Notes
-------- | ------------------- | -------------
Administrateur d’entreprise | Administrateur général | [Nom modifié pour une plus grande clarté](permissions-reference.md#role-template-ids)
Administrateur de services CRM | Administrateur Dynamics 365 | [Reflète la personnalisation actuelle du produit](permissions-reference.md#role-template-ids)
Jonction d’appareils | Déprécié | [Documentation sur les rôles déconseillés](permissions-reference.md#deprecated-roles)
Gestionnaires d’appareils | Déprécié | [Documentation sur les rôles déconseillés](permissions-reference.md#deprecated-roles)
Utilisateurs d’appareils | Déprécié | [Documentation sur les rôles déconseillés](permissions-reference.md#deprecated-roles)
Comptes de synchronisation d’annuaires | Non affiché, car il ne doit pas être utilisé | [Documentation sur les comptes de synchronisation d’annuaires](permissions-reference.md#directory-synchronization-accounts)
Enregistreurs de répertoire | Non affiché, car il ne doit pas être utilisé | [Documentation sur les enregistreurs d’annuaires](permissions-reference.md#directory-writers)
Utilisateur invité | Non affiché, car il ne peut pas être utilisé  | N/D
Administrateur de services Lync | Administrateur Skype Entreprise | [Reflète la personnalisation actuelle du produit](permissions-reference.md#role-template-ids)
Support de niveau 1 partenaire | Non affiché, car il ne doit pas être utilisé | [Documentation sur la prise en charge du niveau 1 de partenaire](permissions-reference.md#partner-tier1-support)
Support de niveau 2 partenaire | Non affiché, car il ne doit pas être utilisé | [Documentation sur la prise en charge du niveau 2 de partenaire](permissions-reference.md#partner-tier2-support)
Utilisateur invité restreint | Non affiché, car il ne peut pas être utilisé | N/D
Utilisateur | Non affiché, car il ne peut pas être utilisé | N/D
Jonction d’appareils d’espace de travail | Déprécié | [Documentation sur les rôles déconseillés](permissions-reference.md#deprecated-roles)

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur l’affectation d’un utilisateur en tant qu’administrateur d’un abonnement Azure, consultez [Ajouter ou supprimer des attributions de rôle Azure (RBAC Azure)](../../role-based-access-control/role-assignments-portal.md).
* Pour plus d’informations sur la façon dont l’accès aux ressources est contrôlé dans Microsoft Azure, consultez [Présentation des différents rôles](../../role-based-access-control/rbac-and-directory-admin-roles.md)
* Pour plus d’informations sur la relation entre les abonnements et un locataire Azure AD ou pour obtenir des instructions sur l’association ou l’ajout d’un abonnement, consultez [Associer ou ajouter un abonnement Azure à votre locataire Azure Active Directory](../fundamentals/active-directory-how-subscriptions-associated-directory.md).
