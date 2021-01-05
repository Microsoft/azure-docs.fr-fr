---
title: Ajouter un compte professionnel ou scolaire à l’application Microsoft Authenticator - Azure AD
description: Ajoutez votre compte professionnel ou scolaire à l’application Microsoft Authenticator pour vérifier votre identité tout en utilisant la vérification à deux facteurs.
services: active-directory
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: user-help
ms.topic: end-user-help
ms.date: 01/24/2019
ms.author: curtand
ms.reviewer: olhaun
ms.openlocfilehash: e003c45aa1e7d75b709b7fbf99532fb1302fcbb9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88797647"
---
# <a name="add-your-work-or-school-account-to-the-microsoft-authenticator-app"></a>Ajouter votre compte professionnel ou scolaire dans l’application Microsoft Authenticator

Si votre organisation utilise la vérification en deux étapes, vous pouvez configurer votre compte professionnel ou scolaire pour utiliser l’application Microsoft Authenticator comme l’une des méthodes de vérification.

>[!Important]
>Avant de pouvoir ajouter votre compte, vous devez télécharger et installer l’application Microsoft Authenticator. Si ce n’est pas déjà fait, suivez la procédure décrite dans l’article [Télécharger et installer l’application](user-help-auth-app-download-install.md).

## <a name="add-your-work-or-school-account"></a>Ajout de votre compte professionnel ou scolaire

1. Sur votre ordinateur, accédez à la page [Vérification de sécurité supplémentaire](https://account.activedirectory.windowsazure.com/proofup.aspx?proofup=1).

    >[!Note]
    >Si la page **Vérification de sécurité supplémentaire** n’est pas affichée, il se peut que votre administrateur ait activé l’expérience des informations de sécurité (préversion). Si tel est le cas, vous devez suivre les instructions de la section [Configurer les informations de sécurité pour utiliser une application d’authentification](security-info-setup-auth-app.md). Si ce n’est pas le cas, vous devez contacter le support technique de votre organisation pour obtenir de l’aide. Pour en savoir plus sur les informations de sécurité, consultez [Vue d’ensemble des informations de sécurité (préversion)](./security-info-setup-signin.md).

2. Cochez la case **Application Authenticator**, puis sélectionnez **Configurer**.

    La page **Configurer l’application mobile** s’affiche.

    ![Écran qui fournit le code QR](./media/user-help-auth-app-download-install/auth-app-barcode.png)

3. Ouvrez l’application Microsoft Authenticator, sélectionnez **Ajouter un compte** depuis l'icône **Personnaliser et contrôler** dans l’angle supérieur droit, puis sélectionnez **Compte professionnel ou scolaire**.

    >[!Note]
    >S’il s’agit de la première fois que vous configurez l’application Microsoft Authenticator, vous pouvez recevoir une invite vous demandant si vous souhaitez autoriser l’application à accéder à votre appareil photo (iOS) ou à prendre des photos et à enregistrer des vidéos (Android). Vous devez sélectionner **Autoriser** pour que l’application Authenticator puisse accéder à votre appareil photo pour prendre une photo du code QR à l’étape suivante. Si vous n’autorisez pas l’accès à l’appareil photo, vous pouvez toujours installer l’application Authenticator, mais vous devrez ajouter les informations de code manuellement. Pour plus d’informations sur l’ajout manuel du code, consultez [Ajout manuel d'un compte à l'application](user-help-auth-app-add-account-manual.md).

4. Utilisez la caméra de votre appareil pour analyser le code QR à partir de l’écran **Configurer l’application mobile** de votre ordinateur, puis sélectionnez **Terminé**.

    >[!Note]
    >Si la caméra ne peut pas capturer le code QR, vous pouvez ajouter manuellement les informations de votre compte à l’application Microsoft Authenticator pour la vérification en deux étapes. Pour en savoir plus, consultez l’article [Ajouter manuellement un compte](user-help-auth-app-add-account-manual.md).

5. Passez en revue l’écran **Comptes** de l’application sur votre appareil pour vous assurer que votre compte est correct et qu’un code de vérification à six chiffres lui est associé. Pour des raisons de sécurité, le code de vérification change toutes les 30 secondes, ce qui empêche quiconque d’utiliser plusieurs fois un même code.

    ![Écran de comptes](./media/user-help-auth-app-download-install/auth-app-accounts.png)

## <a name="next-steps"></a>Étapes suivantes

- Après avoir ajouté vos comptes à l’application, vous pouvez vous connecter à l’aide de l’application Authenticator sur votre appareil. Pour plus d’informations, consultez l’article [Se connecter à l’aide de l’application Microsoft Authenticator](user-help-auth-app-sign-in.md).

- Pour les appareils exécutant iOS, vous pouvez également sauvegarder les informations d’identification de votre compte et les paramètres de l’application associée, comme l’ordre de vos comptes, dans le cloud. Pour plus d’informations, consultez l’article [Sauvegarder et récupérer des informations d’authentification de compte avec l’application Microsoft Authenticator](user-help-auth-app-backup-recovery.md).