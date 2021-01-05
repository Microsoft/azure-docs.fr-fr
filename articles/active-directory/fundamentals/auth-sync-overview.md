---
title: Vue d’ensemble du protocole d’authentification et de synchronisation Azure Active Directory
description: Guide architectural sur l’intégration d’Azure AD à des protocoles d’authentification et modèles de synchronisation hérités
services: active-directory
author: BarbaraSelden
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: conceptual
ms.date: 10/10/2020
ms.author: baselden
ms.reviewer: ajburnle
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: ab63bc5bd2819a239741da525eebb2404a47bbf9
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92441196"
---
# <a name="azure-active-directory-integrations-with-authentication-and-synchronization-protocols"></a>Intégrations d’Azure Active Directory avec les protocoles d’authentification et de synchronisation

Microsoft Azure Active Directory (Azure AD) permet l’intégration avec de nombreux protocoles d’authentification et de synchronisation. Les intégrations d’authentification vous permettent d’utiliser Azure AD et ses fonctionnalités de sécurité et de gestion avec peu ou pas de modifications apportées à vos applications qui utilisent des méthodes d’authentification héritées. Les intégrations de la synchronisation vous permettent de synchroniser les données des utilisateurs et des groupes avec Azure AD, puis les fonctionnalités de gestion des Azure AD des utilisateurs. Certains modèles de synchronisation activent également l’approvisionnement automatique.

## <a name="legacy-authentication-protocols"></a>Protocoles d’authentification hérités

Le tableau suivant présente l’intégration de l’authentification Azure AD aux protocoles d’authentification hérités et à leurs capacités. Sélectionner le nom d’un protocole d’authentification à afficher

* Description détaillée

* Quand utiliser cette fonctionnalité ?

* Diagramme architectural

* Explication des composants système

* Liens pour l’implémentation de l’intégration

 

| Protocole d’authentification| Authentification| Autorisation| Multi-Factor Authentication| Accès conditionnel |
| - |- | - | - | - |
| [Authentification basée sur l’en-tête](auth-header-based.md)|![coche](./media/authentication-patterns/check.png)| ![coche](./media/authentication-patterns/check.png)| ![coche](./media/authentication-patterns/check.png)| ![coche](./media/authentication-patterns/check.png) |
| [Authentification LDAP](auth-ldap.md)| ![coche](./media/authentication-patterns/check.png)| | |  |
| [Authentification OAuth 2.0](auth-oauth2.md)| ![coche](./media/authentication-patterns/check.png)| ![coche](./media/authentication-patterns/check.png)| ![coche](./media/authentication-patterns/check.png)| ![coche](./media/authentication-patterns/check.png) |
| [Authentification OIDC](auth-oidc.md)| ![coche](./media/authentication-patterns/check.png)| ![coche](./media/authentication-patterns/check.png)| ![coche](./media/authentication-patterns/check.png)| ![coche](./media/authentication-patterns/check.png) |
| [Authentification unique basée sur un mot de passe](auth-password-based-sso.md )| ![coche](./media/authentication-patterns/check.png)| ![coche](./media/authentication-patterns/check.png)| ![coche](./media/authentication-patterns/check.png)| ![coche](./media/authentication-patterns/check.png) |
| [Authentification RADIUS]( auth-radius.md)| ![coche](./media/authentication-patterns/check.png)| | ![coche](./media/authentication-patterns/check.png)| ![coche](./media/authentication-patterns/check.png) |
| [Services de passerelle Bureau à distance](auth-remote-desktop-gateway.md)| ![coche](./media/authentication-patterns/check.png)| ![coche](./media/authentication-patterns/check.png)| ![coche](./media/authentication-patterns/check.png)| ![coche](./media/authentication-patterns/check.png) |
| [Secure Shell (SSH)](auth-ssh.md) |  ![coche](./media/authentication-patterns/check.png)| | ![coche](./media/authentication-patterns/check.png)| ![coche](./media/authentication-patterns/check.png) |
| [Authentification SAML](auth-saml.md)| ![coche](./media/authentication-patterns/check.png)| ![coche](./media/authentication-patterns/check.png)| ![coche](./media/authentication-patterns/check.png)| ![coche](./media/authentication-patterns/check.png) |
| [Authentification Windows - délégation Kerberos contrainte](auth-kcd.md)| ![coche](./media/authentication-patterns/check.png)| ![coche](./media/authentication-patterns/check.png)| ![coche](./media/authentication-patterns/check.png)| ![coche](./media/authentication-patterns/check.png) |


 
## <a name="synchronization-patterns"></a>Modèles de synchronisation

Le tableau suivant présente les modèles d’intégration d’Azure AD avec synchronisation et leurs capacités. Sélectionner le nom d’un modèle à afficher

* Description détaillée

* Quand utiliser cette fonctionnalité ?

* Diagramme architectural

* Explication des composants système

* Liens pour l’implémentation de l’intégration



| Modèle de synchronisation| Synchronisation de répertoires| Approvisionnement d'utilisateurs |
| - | - | - |
| [Synchronisation de répertoires](sync-directory.md)| ![coche](./media/authentication-patterns/check.png)|  |
| [Synchronisation LDAP](sync-ldap.md)| ![coche](./media/authentication-patterns/check.png)|  |
| [Synchronisation SCIM](sync-scim.md)| ![coche](./media/authentication-patterns/check.png)| ![coche](./media/authentication-patterns/check.png) |

