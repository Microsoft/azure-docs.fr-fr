---
title: Authentification et autorisations utilisateur Azure Analysis Services | Microsoft Docs
description: Cet article décrit comment Azure Analysis Services utilise Azure Active Directory (Azure AD) pour l’authentification utilisateur et de gestion d’identités.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 12/01/2020
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 551bae56565140da3754e74a23b1cc18087f1171
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96487437"
---
# <a name="authentication-and-user-permissions"></a>Authentification et autorisations utilisateur

Azure Analysis Services utilise Azure Active Directory (Azure AD) pour l’authentification utilisateur et de gestion d’identités. Tout utilisateur qui crée, gère ou se connecte à un serveur Azure Analysis Services doit avoir une identité d’utilisateur valide dans un [client Azure AD](../active-directory/fundamentals/active-directory-whatis.md) dans le même abonnement.

Azure Analysis Services prend en charge [la collaboration Azure AD B2B](../active-directory/external-identities/what-is-b2b.md). Avec B2B, les utilisateurs extérieurs à une organisation peuvent être invités en tant qu’utilisateurs invités dans un répertoire Azure AD. Les invités peuvent être issus d’un autre répertoire client Azure AD ou n’importe quelle adresse e-mail valide. Une fois l’utilisateur invité et accepte l’invitation envoyée par e-mail à partir d’Azure, l’identité de l’utilisateur est ajoutée au répertoire client. Ces identités peuvent être ajoutées aux groupes de sécurité ou en tant que membres d’un rôle d’administrateur de serveur ou de base de données.

![Architecture de l’authentification Azure Analysis Services](./media/analysis-services-manage-users/aas-manage-users-arch.png)

## <a name="authentication"></a>Authentication

Tous les outils et les applications clientes utilisent une ou plusieurs [bibliothèques clientes](/analysis-services/client-libraries?view=azure-analysis-services-current&preserve-view=true) des Services d’analyse (AMO, MSOLAP, ADOMD) pour se connecter à un serveur. 

Les trois bibliothèques clientes prennent en charge les deux flux interactif d’Azure AD et les méthodes d’authentification non interactive. Les deux méthodes non-interactives, les méthodes de mot de passe Active Directory et d’authentification intégrée Active Directory peuvent être utilisées dans les applications qui utilisent AMOND et MSOLAP. Ces deux méthodes n’entraînent jamais l’affichage de boîtes de dialogue contextuelles.

Les applications clientes comme Excel et Power BI Desktop et les outils tels que SSMS et l’extension de projets Analysis Services pour Visual Studio installent les dernières versions des bibliothèques lors de la mise à jour vers la version la plus récente. Les extensions de projets Power BI Desktop, SSMS et Analysis Services sont mises à jour chaque mois. Excel est [mis à jour avec Microsoft 365](https://support.microsoft.com/office/when-do-i-get-the-newest-features-for-microsoft-365-da36192c-58b9-4bc9-8d51-bb6eed468516). Les mises à jour Microsoft 365 sont moins fréquentes. Par ailleurs, certaines organisations utilisent le canal différé, c’est-à-dire que les mises à jour sont reportées de trois mois maximum.

En fonction de l’application cliente ou de l’outil que vous utilisez, le type d’authentification et la façon dont vous vous connectez peuvent être différents. Chaque application peut prendre en charge des fonctionnalités différentes pour la connexion aux services cloud comme Azure Analysis Services.

Power BI Desktop, Visual Studio et SSMS prennent en charge l’authentification universelle Active Directory, une méthode interactive prenant également en charge Azure AD Multi-Factor Authentication (MFA). Azure AD MFA contribue à sécuriser l’accès aux données et aux applications tout en fournissant un processus de connexion simple. Il permet une authentification forte avec plusieurs options de vérification (appel téléphonique, SMS, cartes à puce avec code PIN ou notification d’application mobile). L’authentification multifacteur (MFA) interactive avec Azure AD peut afficher une boîte de dialogue contextuelle de validation. **L’authentification universelle est recommandée**.

Dans le cas d’une connexion à Azure avec un compte Windows et si l’authentification universelle n’est pas sélectionnée ou disponible (Excel), [les services de fédération Active Directory (AD FS)](/windows-server/identity/ad-fs/deployment/how-to-connect-fed-azure-adfs) sont obligatoires. Avec la fédération, les utilisateurs Azure AD et Microsoft 365 s’authentifient avec des informations d’identification locales et peuvent accéder aux ressources Azure.

### <a name="sql-server-management-studio-ssms"></a>SQL Server Management Studio (SSMS)

Les serveurs Azure Analysis Services prennent en charge les connexions depuis [SSMS V17.1](/sql/ssms/download-sql-server-management-studio-ssms) et versions ultérieures à l’aide de l’authentification Windows, l’authentification du mot de passe Active Directory et l’authentification universelle Active Directory. En général, il est recommandé d'utiliser l’authentification universelle Active Directory, car :

*  Elle prend en charge les méthodes d’authentification interactive et non interactive.

*  Elle prend en charge les utilisateurs invités d’Azure B2B dans le client Azure AS. Lors de la connexion à un serveur, les utilisateurs invités doivent sélectionner l’authentification universelle Active Directory.

*  Elle prend en charge authentification multifacteur (MFA). Azure AD MFA permet d’assurer l’accès aux données et applications avec une gamme d’options de vérification simples : appel téléphonique, SMS, cartes à puce avec code PIN ou notification d’application mobile. L’authentification multifacteur (MFA) interactive avec Azure AD peut afficher une boîte de dialogue contextuelle de validation.

### <a name="visual-studio"></a>Visual Studio

Visual Studio se connecte à Azure Analysis Services à l’aide de l’authentification universelle Active Directory avec prise en charge MFA. Les utilisateurs sont invités à se connecter à Azure au premier déploiement. Les utilisateurs doivent se connecter à Azure avec un compte disposant d’autorisations d’administrateur de serveur sur le serveur sur lequel ils sont déployés. Lors de la première connexion à Azure, un jeton est attribué. Le jeton est mis en mémoire cache pour de futures reconnexions.

### <a name="power-bi-desktop"></a>Power BI Desktop

Power BI Desktop se connecte à Azure Analysis Services à l’aide de l’authentification universelle Active Directory avec prise en charge MFA. Les utilisateurs sont invités à se connecter à Azure à la première connexion. Les utilisateurs doivent se connecter à Azure avec un compte inclus dans un administrateur de serveurs ou rôle de bases de données.

### <a name="excel"></a>Excel

Les utilisateurs Excel peuvent se connecter à un serveur en utilisant un compte Windows, un ID d’organisation (e-mail) ou une adresse e-mail externe. Les identités de messagerie externes doivent exister dans Azure AD en tant qu’utilisateur invité.

## <a name="user-permissions"></a>Autorisations utilisateur

**Les administrateurs de serveur** sont spécifiques à une instance de serveur Azure Analysis Services. Ils se connectent avec des outils comme le Portail Azure, SSMS et Visual Studio pour effectuer des tâches telles que l’ajout de bases de données et la gestion des rôles d’utilisateur. Par défaut, l’utilisateur qui crée le serveur dans le portail Azure est automatiquement ajouté en tant qu’administrateur Analysis Services. Les autres administrateurs peuvent être ajoutés à l’aide du portail Azure ou SSMS. Les administrateurs de serveur doivent posséder un compte dans le client Azure AD dans le même abonnement. Pour en savoir plus, consultez [Gérer les administrateurs de serveur](analysis-services-server-admins.md). 

**Les utilisateurs de bases de données** se connectent aux bases de données de modèle à l’aide d’applications clientes comme Excel ou Power BI. Les utilisateurs de bases de données doivent être ajoutés aux rôles de bases de données. Les rôles de bases de données définissent l’administrateur, les processus ou les autorisations de lecture pour une base de données. Il est important de comprendre que le rôle des utilisateurs de bases de données avec des autorisations d’administrateur est différent de celui des administrateurs de serveurs. Toutefois, par défaut, les administrateurs de serveurs sont également administrateurs de bases de données. Pour en savoir plus, consultez [Gérer les rôles et les utilisateurs de bases de données](analysis-services-database-users.md).

**Propriétaires de ressources Azure**. Les propriétaires des ressources gèrent les ressources pour un abonnement Azure. Les propriétaires des ressources peuvent ajouter des identités d’utilisateurs Azure AD au propriétaire ou rôles de contributeur au sein d’un abonnement à l’aide de **Contrôle d’accès** dans le portail Azure ou avec les modèles Azure Resource Manager. 

![Contrôle des accès dans le portail Azure](./media/analysis-services-manage-users/aas-manage-users-rbac.png)

Les rôles à ce niveau s’appliquent aux utilisateurs ou comptes qui doivent effectuer des tâches dans le portail ou à l’aide de modèles Azure Resource Manager. Pour en savoir plus, consultez [Contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../role-based-access-control/overview.md). 

## <a name="database-roles"></a>Rôles de bases de données

 Les rôles définis pour un modèle tabulaire sont des rôles de bases de données. Autrement dit, les rôles contiennent des membres constitués d’utilisateurs d’Azure AD et les groupes de sécurité qui ont des autorisations spécifiques qui définissent l’action que ces membres peuvent effectuer sur une base de données du modèle. Un rôle de bases de données est créé en tant qu’objet distinct dans la base de données et s’applique uniquement à la base de données dans laquelle il est créé.   
  
 Par défaut, lorsque vous créez un projet de modèle tabulaire, le projet de modèle n'a aucun rôle. Les rôles peuvent être définis à l’aide de la boîte de dialogue Gestionnaire de rôles dans Visual Studio. Lorsque les rôles sont définis lors de la conception du projet de modèle, ils sont appliqués uniquement à la base de données de l’espace de travail modèle. Lorsque le modèle est déployé, les mêmes rôles sont appliqués au modèle déployé. Après avoir déployé un modèle, les administrateurs de serveurs et de bases de données peuvent gérer les rôles et les membres à l’aide de SSMS. Pour en savoir plus, consultez [Gérer les rôles et les utilisateurs de bases de données](analysis-services-database-users.md).
  
## <a name="next-steps"></a>Étapes suivantes

[Gérer l’accès aux ressources avec les groupes Azure Active Directory](../active-directory/fundamentals/active-directory-manage-groups.md)   
[Gérer les utilisateurs et rôles de bases de données](analysis-services-database-users.md)  
[Gérer les administrateurs de serveur](analysis-services-server-admins.md)  
[Contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../role-based-access-control/overview.md)