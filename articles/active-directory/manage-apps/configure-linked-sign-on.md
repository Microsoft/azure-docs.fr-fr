---
title: Comprendre l’authentification liée dans Azure Active Directory
description: Comprendre l’authentification liée dans Azure Active Directory.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: conceptual
ms.workload: identity
ms.date: 07/30/2020
ms.author: kenwith
ms.reviewer: arvinh,luleon
ms.openlocfilehash: 8fc26b00214106aa08e0e6e4c41f9f05aca9344d
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94648036"
---
# <a name="understand-linked-sign-on"></a>Comprendre l’authentification liée

Avec la [série de guides de démarrage rapide](view-applications-portal.md) sur la gestion des applications, vous avez appris à utiliser Azure AD comme fournisseur d’identité (IdP) pour une application. Dans le guide de démarrage rapide, vous configurez l’authentification unique par SAML ou OIDC. Une autre option consiste à utiliser **Authentification liée**. Cet article décrit plus en détail l’option d’authentification liée.

L’option **Authentification liée** vous permet de configurer l’emplacement cible lorsqu’un utilisateur sélectionne l’application dans le portail [Mes applications](https://myapps.microsoft.com/) ou Office 365 de votre organisation.

Voici quelques scénarios courants dans lesquels l’option d’authentification liée est utile :
- Ajouter un lien vers une application web personnalisée qui utilise actuellement la fédération, par exemple les services de fédération Active Directory (AD FS).
- Ajouter des liens ciblés vers des pages SharePoint spécifiques ou d’autres pages web dont vous souhaitez qu’elles s’affichent uniquement sur vos volets d’accès d’utilisateurs.
- Ajoutez un lien vers une application qui ne requiert pas d’authentification. 
 
 L’option **Authentification liée** ne fournit pas de fonctionnalité d’authentification via des informations d’identification Azure AD. Toutefois, vous pouvez toujours utiliser les autres fonctionnalités d’**Applications d’entreprise**. Par exemple, vous pouvez utiliser les journaux d’audit et ajouter un logo et un nom d’application personnalisés.

## <a name="before-you-begin"></a>Avant de commencer

Pour être efficace rapidement, suivez la [série de guides de démarrage rapide](view-applications-portal.md) sur la gestion des applications. Dans le démarrage rapide, où vous configurez l’authentification unique, vous trouverez également l’option **Authentification liée**. 

L’option **Authentification liée** ne fournit pas de fonctionnalité d’authentification via Azure AD. L’option définit simplement l’emplacement auquel les utilisateurs sont envoyés lorsqu’ils sélectionnent l’application sur [Mes applications](https://myapps.microsoft.com/) ou le lanceur d’applications Microsoft 365.  Étant donné que la connexion ne fournit pas de fonctionnalité de connexion via Azure AD, l’accès conditionnel n’est pas disponible pour les applications configurées avec l’authentification unique liée.

> [!IMPORTANT] 
> Dans certains cas, l’option **Authentification unique** n’est pas accessible pour une application sous **Applications d’entreprise**. 
>
> Si l’application a été inscrite à l’aide d’**Inscriptions d’applications**, la capacité d’authentification unique est configurée pour utiliser OIDC OAuth par défaut. Dans ce cas, l’option **Authentification unique** ne s’affiche pas dans le volet de navigation sous **Applications d’entreprise**. Quand vous utilisez **inscriptions d’applications** pour ajouter votre application personnalisée, vous configurez les options dans le fichier du manifeste. Pour en savoir plus sur le fichier manifeste, consultez [Manifeste d’application Azure Active Directory](../develop/reference-app-manifest.md). Pour en savoir plus sur les standards SSO, consultez [Authentification et autorisation avec la plateforme d’identités Microsoft](../develop/authentication-vs-authorization.md#authentication-and-authorization-using-microsoft-identity-platform). 
>
> D’autres scénarios dans lesquels **l’authentification unique** sera absente de la navigation incluent les cas où une application est hébergée dans un autre locataire ou si votre compte ne dispose pas des autorisations requises (administrateur général, administrateur d’application Cloud, administrateur d’application ou propriétaire du principal de service). Les autorisations peuvent également être à l’origine d’un scénario dans lequel vous pouvez ouvrir **l’authentification unique**, mais vous ne pourrez pas l’enregistrer. En savoir plus sur les rôles d’administration d’Azure AD voir (https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-assign-admin-roles).

### <a name="configure-link"></a>Configurer le lien

Pour définir le lien d’une application, sélectionnez **Authentification liée** sur la page **Authentification unique**. Entrez ensuite le lien et sélectionnez **Enregistrer**. Vous avez besoin d’un rappel pour trouver ces options ? Consultez la [série de démarrages rapides](view-applications-portal.md).
 
Une fois que vous avez configuré une application, assignez-lui des utilisateurs et des groupes. Lorsque vous affectez des utilisateurs, vous pouvez contrôler le moment où l’application s’affiche dans [Mes applications](https://myapps.microsoft.com/) ou le lanceur d’applications Microsoft 365.

## <a name="next-steps"></a>Étapes suivantes

- [Affecter des utilisateurs et des groupes à l’application](./assign-user-or-group-access-portal.md)
- [Configurer le provisionnement automatique d’un compte utilisateur](../app-provisioning/configure-automatic-user-provisioning-portal.md)