---
title: Configurer la stratégie d’inscription de l’authentification multifacteur – Azure Active Directory Identity Protection
description: Découvrez comment configurer la stratégie d’inscription d’authentification multifacteur Azure AD Identity Protection.
services: active-directory
ms.service: active-directory
ms.subservice: identity-protection
ms.topic: how-to
ms.date: 06/05/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sahandle
ms.collection: M365-identity-device-management
ms.openlocfilehash: 072db1d47abd95844075aeedfeddc4f8cf6bf936
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94835864"
---
# <a name="how-to-configure-the-azure-ad-multi-factor-authentication-registration-policy"></a>Procédure : Configurer la stratégie d'inscription Azure AD Multi-Factor Authentication

Azure AD Identity Protection vous permet de gérer le déploiement de l'inscription Azure AD Multi-Factor Authentication (MFA) en configurant une stratégie d'accès conditionnel pour exiger l'inscription MFA, quelle que soit l'application d'authentification moderne à laquelle vous vous connectez.

## <a name="what-is-the-azure-ad-multi-factor-authentication-registration-policy"></a>Qu'est-ce qu'une stratégie d'inscription Azure AD Multi-Factor Authentication ?

Azure AD Multi-Factor Authentication permet de vérifier votre identité, à l'aide d'une méthode plus sécurisée que la simple fourniture d'un nom d'utilisateur et d'un mot de passe. Ce service fournit une deuxième couche de sécurité pour les connexions utilisateur. Pour que les utilisateurs puissent répondre aux invites d'authentification multifacteur, ils doivent d'abord s'inscrire à Azure AD Multi-Factor Authentication.

Voici les raisons pour lesquelles il est conseillé d'exiger une authentification Azure AD MFA pour les connexions utilisateur :

- Elle fournit une authentification renforcée avec un éventail d’options de vérification.
- Elle joue un rôle clé dans la correction automatique en cas de détections de risques dans Identity Protection.

Pour plus d'informations sur Azure AD Multi-Factor Authentication, consultez [Présentation d'Azure AD Multi-Factor Authentication](../authentication/howto-mfa-getstarted.md).

## <a name="policy-configuration"></a>Configuration de la stratégie

1. Accédez au [portail Azure](https://portal.azure.com).
1. Accédez à **Azure Active Directory** > **Sécurité** > **Identity Protection** > **stratégie d’inscription MFA**.
   1. Sous **Affectations**
      1. **Utilisateurs** : choisissez **Tous les utilisateurs** ou **Sélectionner des personnes et des groupes** si vous limitez votre lancement.
         1. Si vous le souhaitez, vous pouvez exclure des utilisateurs de la stratégie.
   1. Sous **Contrôles**
      1. Assurez-vous que la case **Exiger l'inscription Azure AD MFA** est cochée, puis choisissez **Sélectionner**.
   1. **Appliquer la stratégie** - **Activé**
   1. **Save**

## <a name="user-experience"></a>Expérience utilisateur

Azure Active Directory Identity Protection invite vos utilisateurs à s’inscrire la prochaine fois qu’ils se connectent de façon interactive, et ils disposent de 14 jours pour terminer l’inscription. Pendant cette période de 14 jours, les utilisateurs peuvent contourner l’inscription, mais à la fin de la période, ils devront s’inscrire pour pouvoir aller au bout du processus de connexion.

Pour une obtenir une vue d’ensemble de l’expérience utilisateur, consultez :

- [Expériences de connexion avec Azure AD Identity Protection](concept-identity-protection-user-experience.md).  

## <a name="next-steps"></a>Étapes suivantes

- [Activer les stratégies de connexion et de risque utilisateur](howto-identity-protection-configure-risk-policies.md)

- [Activer la réinitialisation de mot de passe en libre-service Azure AD](../authentication/howto-sspr-deployment.md)

- [Activer l’authentification multifacteur Azure AD](../authentication/howto-mfa-getstarted.md)
