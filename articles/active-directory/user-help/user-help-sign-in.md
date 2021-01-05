---
title: Se connecter à l’aide de ses informations d’authentification d’identité – Azure AD
description: Découvrez comment vous connecter en utilisant les différentes méthodes de vérification d’identité dans les informations de sécurité.
services: active-directory
author: curtand
manager: daveba
ms.reviewer: librown
ms.service: active-directory
ms.workload: identity
ms.subservice: user-help
ms.topic: end-user-help
ms.date: 07/18/2019
ms.author: curtand
ms.custom: user-help
ms.openlocfilehash: 42ced6632ebfa56af8fe13a02f531b9835e13df3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88799381"
---
# <a name="sign-in-using-two-step-verification-or-security-info"></a>Se connecter à l’aide de la vérification en deux étapes ou des informations de sécurité

Une fois que vous aurez configuré la vérification en deux étapes ou les informations de sécurité, vous pourrez vous connecter à votre compte à l'aide de la méthode de vérification que vous avez spécifiée.

> [!Note]
> Si vous utilisez toujours la vérification en deux étapes, vous devrez configurer vos méthodes de vérification en suivant les instructions données dans l’article [Configurer mon compte pour la vérification en deux étapes](multi-factor-authentication-end-user-first-time.md).
>
> Si votre administrateur a activé les informations de sécurité, vous devrez définir vos méthodes de vérification en suivant ces articles pas à pas :<ul><li>[Set up security info to use an authentication app](security-info-setup-auth-app.md) (Configurer les informations de sécurité pour utiliser une application d’authentification)</li><li>[Set up security info to use text messaging](security-info-setup-text-msg.md) (Configurer les informations de sécurité pour utiliser la messagerie texte)</li><li>[Set up security info to use a phone call](security-info-setup-phone-number.md) (Configurer les informations de sécurité pour utiliser un appel téléphonique)</li><li>[Configurer les informations de sécurité pour utiliser une clé de sécurité](security-info-setup-security-key.md)</li></ul>

## <a name="sign-in-using-an-authenticator-app-notification-on-your-mobile-device"></a>Se connecter à partir d’une notification de l’application d’authentification sur votre appareil mobile

1. Connectez-vous à votre compte à l'aide de votre nom d'utilisateur et de votre mot de passe.

2. Sélectionnez **Approuver** à partir de la notification d’approbation envoyée sur votre appareil mobile.

## <a name="sign-in-using-an-authenticator-app-code-on-your-mobile-device"></a>Se connecter à partir d’un code de l’application d’authentification sur votre appareil mobile

1. Connectez-vous à votre compte à l'aide de votre nom d'utilisateur et de votre mot de passe.

2. Ouvrez votre application d’authentification et tapez le code généré de manière aléatoire pour votre compte dans la zone **Entrez un code**.

## <a name="sign-in-using-your-phone-number"></a>Se connecter à l’aide de votre numéro de téléphone

1. Connectez-vous à votre compte à l'aide de votre nom d'utilisateur et de votre mot de passe.

2. Décrochez votre téléphone et suivez les instructions.

## <a name="sign-in-using-a-text-message"></a>Se connecter à l’aide d’un SMS

1. Connectez-vous à votre compte à l'aide de votre nom d'utilisateur et de votre mot de passe.

2. Ouvrez le SMS et tapez le code indiqué dans la zone **Entrez un code**.

## <a name="sign-in-using-a-security-key-at-the-lock-screen"></a>Ouvrir une session à l'aide d'une clé de sécurité dans l'écran de verrouillage

1. Après avoir enregistré votre clé de sécurité, sélectionnez l'image de la clé de sécurité dans l'écran de verrouillage Windows 10.

2. Insérez votre clé de sécurité dans le port USB de votre appareil et connectez-vous à Windows à l'aide du code PIN de votre clé de sécurité.

    ![Connexion par clé de sécurité depuis l’écran de verrouillage Windows 10](./media/security-info/security-info-windows-10-lock-screen-security-key.png)

## <a name="sign-in-using-a-security-key-and-the-microsoft-edge-browser"></a>Se connecter à l'aide d'une clé de sécurité et du navigateur Microsoft Edge

1. Après avoir enregistré votre clé de sécurité, ouvrez le navigateur Microsoft Edge.

2. Lorsque vous êtes invité à vous connecter, insérez votre clé de sécurité dans le port USB de votre appareil, puis connectez-vous à Windows à l'aide de votre code PIN.

    ![Se connecter à l'aide d'une clé de sécurité et du navigateur Microsoft Edge](./media/security-info/security-info-edge-security-key.png)

    >[!NOTE]
    >Pour plus d’informations sur la connexion à l’aide de l’application Microsoft Authenticator, consultez l’article [Vous connecter à vos comptes à l’aide de l’application Microsoft Authenticator](user-help-auth-app-sign-in.md).

## <a name="sign-in-using-another-verification-method"></a>Se connecter à l’aide d’une autre méthode de vérification

Si, pour une raison quelconque, vous ne parvenez pas à utiliser votre méthode de connexion principale, vous pouvez utiliser une autre méthode de vérification précédemment configurée.

1. Connectez-vous à votre compte comme vous le faites habituellement, puis cliquez sur le lien **Se connecter d'une autre façon** situé dans la page **Vérification en deux étapes**.

    ![Changer de méthode de vérification](media/security-info/two-factor-auth-signin-another-way.png)

    >[!Note]
    >Si le lien **Se connecter d'une autre façon** ne s'affiche, cela signifie que vous n'avez pas configuré d'autres méthodes de vérification. Contactez votre administrateur pour qu’il vous aide à vous connecter à votre compte. Lorsque c'est chose faite, veillez à ajouter d’autres méthodes de vérification. Pour plus d’informations sur l’ajout de méthodes de vérification, consultez l’article [Gérer les paramètres de la vérification en deux étapes](multi-factor-authentication-end-user-manage-settings.md).
    >
    >Si le lien **Se connecter d'une autre façon** s'affiche, mais que vous ne voyez toujours pas les autres méthodes de vérification, contactez votre administrateur pour qu’il vous aide à vous connecter à votre compte.

2. Choisissez votre méthode de vérification et poursuivez le processus de vérification en deux étapes.

3. Une fois de retour dans votre compte, vous pouvez mettre à jour vos méthodes de vérification (si nécessaire). Pour plus d’informations sur l’ajout ou le changement de méthodes, consultez l’article [Gérer les paramètres de la vérification en deux étapes](multi-factor-authentication-end-user-manage-settings.md).

## <a name="next-steps"></a>Étapes suivantes

- Découvrez les informations de sécurité avec l’article [Security info (preview) overview](./security-info-setup-signin.md) (Vue d’ensemble des informations de sécurité [préversion]).

- Pour plus d’informations sur la vérification en deux étapes, consultez l’article [Vue d’ensemble de la vérification en deux étapes](./multi-factor-authentication-end-user-first-time.md).

- En cas de perte ou d’oubli de votre mot de passe, réinitialisez-le à partir du [portail de réinitialisation de mot de passe](https://passwordreset.microsoftonline.com/).

- Obtenez des conseils de dépannage et de l’aide pour les problèmes de connexion en consultant l’article [Quand vous ne pouvez pas vous connecter à votre compte Microsoft](https://support.microsoft.com/help/12429/microsoft-account-sign-in-cant).