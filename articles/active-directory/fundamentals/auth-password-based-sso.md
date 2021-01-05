---
title: Authentification par mot de passe avec Azure Active Directory
description: Guide architectural sur l’implémentation de l’authentification par mot de passe avec Azure Active Directory.
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
ms.openlocfilehash: 5bd6a5c8af117bf6cb39969a5f1b1f17ff08681c
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96172820"
---
# <a name="password-based-authentication-with-azure-active-directory"></a>Authentification par mot de passe avec Azure Active Directory

L’authentification unique (SSO) basée sur un mot de passe utilise le processus d’authentification existant pour l’application. Lorsque vous activez l’authentification unique par mot de passe, Azure Active Directory (Azure AD) collecte, chiffre et stocke en toute sécurité les informations d’identification de l’utilisateur dans l’annuaire. Azure AD fournit le nom d’utilisateur et le mot de passe à l’application lorsque l’utilisateur tente de se connecter.

Choisissez l’authentification unique par mot de passe lorsqu’une application s’authentifie avec un nom d’utilisateur et un mot de passe, et non des jetons d’accès et des en-têtes. L’authentification unique par mot de passe prend en charge toutes les applications basées sur le Cloud qui possèdent une page de connexion HTML. 

## <a name="use-when"></a>Cas d'utilisation

Vous devez assurer la protection avec une authentification préalable et fournir une authentification unique via le coffre de mots de passe aux applications Web.

![diagramme architectural](./media/authentication-patterns/password-based-sso-auth.png)


## <a name="components-of-system"></a>Composants du système

* **Utilisateur** : Accède à une application basée sur des formes à partir de My Apps ou en visitant directement le site. 

* **Navigateur Web** : Composant avec lequel l’utilisateur interagit pour accéder à l’URL externe de l’application. L’utilisateur accède à l’application basée sur des formulaires via l’extension MyApps. 

* **extension MyApps**: Identifie l’application SSO basée sur un mot de passe configuré et fournit les informations d’identification au formulaire de connexion. L’extension MyApps est installée sur le navigateur Web. 

* **Azure AD** : Authentifie l’utilisateur.

## <a name="implement-password-based-sso-with-azure-ad"></a>Implémenter l’authentification unique basée sur un mot de passe avec Azure AD

* [Présentation de l’authentification unique basée sur un mot de passe](../manage-apps/what-is-single-sign-on.md) 

* [Configurer l’authentification unique basée sur un mot de passe pour les applications Cloud ](../manage-apps/configure-password-single-sign-on-non-gallery-applications.md)

* [Configurer l’authentification unique basée sur un mot de passe pour les applications locales avec le proxy d’application](../manage-apps/application-proxy-configure-single-sign-on-password-vaulting.md)

