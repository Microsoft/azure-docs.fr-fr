---
title: Besoin d’aide avec le portail Mes applications - Azure Active Directory | Microsoft Docs
description: Obtenez de l’aide sur la connexion et l’exécution de tâches courantes dans le portail Mes applications.
services: active-directory
author: curtand
manager: daveba
ms.service: active-directory
ms.subservice: user-help
ms.workload: identity
ms.topic: end-user-help
ms.date: 03/21/2019
ms.author: curtand
ms.reviewer: kasimpso
ms.custom: user-help, seo-update-azuread-jan
ms.openlocfilehash: 5ae946f82590f2270719570e4b589d49542ffec8
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94833977"
---
# <a name="troubleshoot-problems-with-the-my-apps-portal"></a>Résoudre les problèmes avec le portail Mes applications

Si vous rencontrez des problèmes liés à la connexion ou à l’utilisation du portail **Mes applications**, essayez ces conseils de résolution des problèmes avant de demander de l’aide au support technique ou à votre administrateur.

## <a name="im-having-trouble-installing-the-my-apps-secure-sign-in-extension"></a>Je n’arrive pas à installer l’extension de connexion sécurisée à Mes applications

Si vous n’arrivez pas à installer l’extension de connexion sécurisée à Mes applications :

- Assurez-vous d’utiliser un navigateur pris en charge, par exemple :

    - **Microsoft Edge.** Sur Windows 10 Édition anniversaire ou version ultérieure.

    - **Google Chrome.** Sur Windows 7 ou version ultérieure, et sur Mac OS X ou version ultérieure.

    - **Mozilla Firefox 26.0 ou version ultérieure.** Sur Windows XP SP2 ou version ultérieure, et sur Mac OS X 10.6 ou version ultérieure.

    - **Internet Explorer 11.** Sur Windows 7 ou version ultérieure (prise en charge limitée).

- Assurez-vous que vos paramètres d’extension de navigateur sont activés.

- Essayez de redémarrer votre navigateur et de vous connecter au portail **Mes applications**.

- Essayez d’effacer les cookies de votre navigateur et de vous connecter au portail **Mes applications**.

## <a name="i-cant-sign-in-to-the-my-apps-portal"></a>Je n’arrive pas à me connecter au portail **Mes applications**

Si vous ne parvenez pas à vous connecter au portail **Mes applications**, vous pouvez essayer ce qui suit :

- Vérifiez que vous utilisez la bonne URL. Il doit s’agir de https://myapps.microsoft.com ou d’une page personnalisée pour votre organisation, comme https://myapps.microsoft.com/contoso.com.

- Vérifiez que votre mot de passe est correct et qu’il n’a pas expiré. Pour plus d’informations, consultez [Réinitialiser votre mot de passe professionnel ou scolaire](active-directory-passwords-update-your-own-password.md).

- Assurez-vous que vos informations de vérification sont à jour et exactes. Pour plus d’informations, consultez [Qu’est-ce que l’authentification Azure AD MFA pour moi ?](./multi-factor-authentication-end-user-first-time.md) ou [Modifier les méthodes et les informations de sécurité](./security-info-setup-auth-app.md).

- Ajoutez l’URL du portail **Mes applications** au paramètre **Propriétés Internet > Sécurité > Sites de confiance**.

- Videz le cache de votre navigateur et essayez de vous connecter à nouveau.

## <a name="my-password-isnt-working"></a>Mon mot de passe ne fonctionne pas

Si vous avez oublié votre mot de passe, si votre organisation ne vous en a jamais fourni, si l’accès à votre compte a été bloqué, ou si vous souhaitez modifier votre mot de passe, consultez [J’ai oublié mon mot de passe Azure AD](active-directory-passwords-update-your-own-password.md).

## <a name="i-want-to-be-able-to-reset-my-own-password"></a>Je souhaite pouvoir réinitialiser mon propre mot de passe

Pour pouvoir réinitialiser votre propre mot de passe, votre administrateur doit tout d’abord activer cette fonctionnalité dans votre organisation, puis vous devez mettre à jour et vérifier vos méthodes de vérification. Pour plus d’informations sur la mise à jour de vos méthodes de vérification, consultez [S’inscrire pour la réinitialisation du mot de passe en libre-service](active-directory-passwords-reset-register.md).

## <a name="im-getting-an-access-denied-message-when-i-start-an-app"></a>J’obtiens un message d’accès refusé lorsque je démarre une application

Si vous voyez un message d’**accès refusé** lorsque vous démarrez une application dans le portail **Mes applications**, vous pouvez essayer ce qui suit :

- Assurez-vous que vous avez installé l’[extension de connexion sécurisée à Mes applications](my-apps-portal-end-user-access.md#download-and-install-the-my-apps-secure-sign-in-extension) et que vous utilisez un [navigateur pris en charge](my-apps-portal-end-user-access.md#supported-browsers).

- Assurez-vous que vous utilisez bien la bonne URL pour l’application et que cette URL se trouve dans votre liste **Propriétés Internet > Sécurité > Sites de confiance**.

- Vérifiez que votre mot de passe est correct et qu’il n’a pas expiré. Pour plus d’informations, consultez [Réinitialiser votre mot de passe professionnel ou scolaire](active-directory-passwords-update-your-own-password.md).

- Assurez-vous que vos informations de vérification sont à jour et exactes. Pour plus d’informations, consultez [Qu’est-ce que l’authentification Azure AD MFA pour moi ?](./multi-factor-authentication-end-user-first-time.md) ou [Modifier les méthodes et les informations de sécurité](./security-info-setup-auth-app.md).

- Videz le cache de votre navigateur et essayez de vous connecter à nouveau.

Si après avoir essayé ces solutions, vous ne pouvez toujours pas accéder à votre application, vous devez contacter le support technique de votre organisation pour obtenir une assistance.

## <a name="next-steps"></a>Étapes suivantes

Après vous être connecté au portail **Mes applications**, vous pouvez également mettre à jour votre profil et les informations de votre compte, les informations de votre groupe et les informations de vérification de l’accès (si vous en avez l’autorisation).

- [Accéder aux applications et les utiliser sur le portail Mes applications](my-apps-portal-end-user-access.md).

- [Modifier vos informations de profil](./my-account-portal-settings.md).

- [Consulter et mettre à jour vos informations relatives aux groupes](my-apps-portal-end-user-groups.md).

- [Effectuer vos propres révisions d’accès](my-apps-portal-end-user-access-reviews.md).