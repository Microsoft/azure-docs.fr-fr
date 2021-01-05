---
title: Qu’est-ce que l’application Microsoft Authenticator ? - Azure Active Directory | Microsoft Docs
description: Découvrez des informations sur l’application Microsoft Authenticator, notamment ce qu’elle est, comment elle fonctionne et quelles informations sont incluses dans cette section du contenu.
services: active-directory
author: curtand
manager: daveba
ms.reviewer: sahenry
ms.service: active-directory
ms.subservice: user-help
ms.workload: identity
ms.topic: end-user-help
ms.date: 01/15/2020
ms.author: curtand
ms.openlocfilehash: 7f4c09a32794e13b0e0041ee916401b714d8509e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90055229"
---
# <a name="what-is-the-microsoft-authenticator-app"></a>Qu’est-ce que l’application Microsoft Authenticator ?

L’application Microsoft Authenticator vous aide à vous connecter à vos comptes si vous utilisez la vérification à deux facteurs. La vérification en deux étapes vous permet d’accéder à vos comptes de façon plus sûre, en particulier quand vous consultez des informations sensibles. Les mots de passe pouvant être oubliés, volés ou compromis, la vérification à deux facteurs est une mesure de sécurité supplémentaire qui renforce la protection de votre compte contre les tentatives d’intrusion.

Vous pouvez utiliser l’application Microsoft Authenticator de plusieurs manières, notamment :

- Répondre à une invite d’authentification après vous être connecté avec vos nom d’utilisateur et mot de passe.

- Vous connecter sans entrer de mot de passe, en utilisant votre nom d’utilisateur, l’application d’authentification, et votre appareil mobile avec votre empreinte digitale, votre visage ou votre code PIN.

- En tant que générateur de code pour tous les comptes prenant en charge les applications d’authentification.

> [!Important]
> L’application Microsoft Authenticator fonctionne avec tout compte utilisant une vérification en deux étapes et prenant en charge les mots de passe à usage unique et durée définie (normes TOTP).
>
>Cet article s’adresse aux utilisateurs qui essaient de télécharger et d’utiliser l’application Microsoft Authenticator comme méthode de vérification de sécurité. Si vous êtes administrateur et que vous recherchez des informations sur la façon d’activer la connexion sans mot de passe à l’aide de l’application Authenticator pour vos employés et d’autres utilisations, consultez [Activer la connexion sans mot de passe avec l’application Microsoft Authenticator (préversion)](../authentication/howto-authentication-passwordless-phone.md).

## <a name="terminology"></a>Terminologie

| Terme|Description|
| ----|-----------|
| Vérification à deux facteurs | Un processus de vérification qui vous oblige à utiliser seulement deux éléments d’information de vérification, comme un mot de passe et un code PIN. L’application Microsoft Authenticator prend en charge à la fois la vérification à deux facteurs standard et la connexion sans mot de passe. |
| Authentification multifacteur (MFA) | La vérification à deux facteurs est une authentification multifacteur, qui vous oblige à utiliser *au moins* deux éléments d’information de vérification, en fonction des exigences de votre organisation. |
| Compte Microsoft (également appelé MSA) | Vous créez vos propres comptes personnels pour accéder à vos produits et services cloud Microsoft orientés consommateurs, comme Outlook, OneDrive, Xbox LIVE ou Microsoft 365. Votre compte Microsoft est créé et stocké dans le système de comptes d’identité des consommateurs de Microsoft. |
| Comptes professionnels ou scolaires | Votre organisation crée votre compte professionnel ou scolaire (comme alain@contoso.com) pour vous permettre d’accéder à des ressources internes et potentiellement restreintes, telles que Microsoft Azure, Windows Intune et Microsoft 365. |
| Code de vérification | Code à six chiffres qui s’affiche dans l’application d’authentification, sous chaque compte ajouté. Le code de vérification change toutes les 30 secondes, ce qui empêche quiconque d’utiliser plusieurs fois un même code. Ce code est également appelé « code secret à usage unique (OTP) ». |

## <a name="how-two-factor-verification-works-with-the-app"></a>Fonctionnement de la vérification à deux facteurs avec l’application

La vérification à deux facteurs fonctionne avec l’application Microsoft Authenticator comme suit :

- **Notification.** Tapez votre nom d’utilisateur et votre mot de passe sur l’appareil où vous vous connectez, pour votre compte professionnel ou scolaire ou pour votre compte Microsoft personnel : l’application Microsoft Authenticator envoie alors une notification vous invitant à **approuver la connexion**. Choisissez **Approuver** si vous reconnaissez la tentative de connexion. Sinon, choisissez **Refuser**. Si vous choisissez **Refuser**, vous pouvez également marquer la demande comme étant frauduleuse.

- **Code de vérification.** Tapez votre nom d’utilisateur et votre mot de passe sur l’appareil où vous vous connectez, pour votre compte professionnel ou scolaire ou pour votre compte Microsoft personnel, puis copiez le code de vérification associé depuis l’écran **Comptes** de l’application Microsoft Authenticator. Le code de vérification porte également le nom d’authentification du code secret à usage unique (OTP).

- **Connexion sans mot de passe.** Tapez votre nom d’utilisateur et votre mot de passe sur l’appareil où vous vous connectez, pour votre compte professionnel ou scolaire ou pour votre compte Microsoft personnel, puis utilisez votre appareil mobile pour confirmer que c’est bien vous en utilisant votre empreinte digitale, votre visage ou votre code PIN. Pour cette méthode, vous n’avez pas besoin d’entrer votre mot de passe.

### <a name="whether-to-use-your-devices-biometric-capabilities"></a>Utiliser ou non les fonctionnalités de biométrie de votre appareil

Si vous utilisez un code PIN pour effectuer le processus d’authentification, vous pouvez configurer l’application Microsoft Authenticator pour qu’elle utilise à la place les fonctionnalités (biométriques) de reconnaissance faciale ou d’empreinte digitale de votre appareil. Vous pouvez configurer cela la première fois que vous utilisez l’application d’authentification pour vérifier votre compte, en sélectionnant l’option pour utiliser les fonctionnalités biométriques de votre appareil comme moyen d’identification au lieu de votre code PIN.

## <a name="who-decides-if-you-use-this-feature"></a>Qui décide si vous utilisez cette fonctionnalité ?

Selon votre type de compte, votre organisation peut décider que vous devez utiliser la vérification à deux facteurs ou vous pouvez le décider par vous-même.

- **Un compte professionnel ou scolaire.** Si vous utilisez un compte professionnel ou scolaire (par exemple alain@contoso.com), c’est à votre organisation de décider si vous devez utiliser la vérification à deux facteurs, ainsi que les méthodes de vérification spécifiques. Pour plus d’informations sur l’ajout de votre compte professionnel ou scolaire dans l’application Microsoft Authenticator, consultez [Ajouter vos comptes professionnels ou scolaires](user-help-auth-app-add-work-school-account.md).

- **Compte Microsoft personnel.** Vous pouvez choisir de configurer la vérification à deux facteurs pour vos comptes Microsoft personnels (par exemple alain@outlook.com). Pour plus d’informations sur l’ajout de votre compte Microsoft personnel, consultez [Ajouter vos comptes personnels](user-help-auth-app-add-personal-ms-account.md).

- **Compte non-Microsoft.** Vous pouvez choisir de configurer la vérification à deux facteurs pour vos comptes non-Microsoft (par exemple alain@gmail.com). Vos comptes non-Microsoft n’utilisent peut-être pas le terme de « vérification à deux facteurs », mais vous pouvez normalement trouver la fonctionnalité dans les paramètres de la **Sécurité** ou de la **Connexion**. L’application Microsoft Authenticator fonctionne avec tous les comptes qui prennent en charge les standards TOTP. Pour plus d’informations sur l’ajout de vos comptes non-Microsoft, consultez [Ajouter vos comptes non-Microsoft](user-help-auth-app-add-non-ms-account.md).

## <a name="in-this-section"></a>Contenu de cette section

| Article | Description |
| ------ | ------------ |
| [Télécharger et installer l’application](user-help-auth-app-download-install.md) | Décrit où et comment obtenir et installer l’application Microsoft Authenticator pour les appareils exécutant Android et iOS. |
| [Ajouter vos comptes professionnels ou scolaires](user-help-auth-app-add-work-school-account.md) | Décrit comment ajouter vos comptes professionnels ou scolaires, et vos comptes personnels, à l’application Microsoft Authenticator. |
| [Ajouter vos comptes personnels](user-help-auth-app-add-personal-ms-account.md) | Décrit comment ajouter vos comptes Microsoft personnels à l’application Microsoft Authenticator. |
| [Ajouter vos comptes non-Microsoft](user-help-auth-app-add-non-ms-account.md) | Décrit comment ajouter vos comptes non-Microsoft à l’application Microsoft Authenticator. |
| [Ajouter vos comptes manuellement](user-help-auth-app-add-account-manual.md) | Décrit comment ajouter manuellement vos comptes à l’application Microsoft Authenticator si vous ne pouvez pas scanner le code QR fourni. |
| [Se connecter avec l’application](user-help-auth-app-sign-in.md) | Décrit comment se connecter à vos différents comptes avec l’application Microsoft Authenticator.|
| [Sauvegarder et récupérer les informations d’identification du compte](user-help-auth-app-backup-recovery.md) | Fournissez des informations sur la sauvegarde et la récupération de vos informations d’authentification à l’aide de l’application Microsoft Authenticator. |
| [Forum aux questions sur l’application Microsoft Authenticator](user-help-auth-app-faq.md) | Fournit des réponses aux questions les plus fréquentes sur l’application. |