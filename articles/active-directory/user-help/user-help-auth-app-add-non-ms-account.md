---
title: Ajouter vos comptes non-Microsoft à l’application Microsoft Authenticator – Azure AD
description: Ajoutez vos comptes non-Microsoft, tels que Google ou Facebook, à l’application Microsoft Authenticator pour vérifier votre identité lors de la vérification à deux facteurs.
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
ms.openlocfilehash: cba8837fc738884fc187b13083487c2ab62d25f0
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96020090"
---
# <a name="add-non-microsoft-accounts-to-the-microsoft-authenticator-app"></a>Ajouter vos comptes non-Microsoft à l’application Microsoft Authenticator

Ajoutez vos comptes non-Microsoft, tels que Google, Facebook ou GitHub, à l'application Microsoft Authenticator pour la vérification en deux étapes. L'application Microsoft Authenticator fonctionne avec n'importe quelle application qui utilise la vérification en deux étapes et n'importe quel compte qui prend en charge les mots de passe à usage unique et durée définie (normes TOTP).

>[!Important]
>Avant de pouvoir ajouter votre compte, vous devez télécharger et installer l’application Microsoft Authenticator. Si ce n’est pas déjà fait, suivez la procédure décrite dans l’article [Télécharger et installer l’application](user-help-auth-app-download-install.md).

## <a name="add-personal-accounts"></a>Ajouter des comptes personnels

En règle générale, pour tous vos comptes personnels, vous devez :

1. Vous connecter à votre compte, puis activer la vérification en deux étapes en utilisant votre appareil ou votre ordinateur.

2. Ajouter le compte à l'application Microsoft Authenticator. Dans le cadre de ce processus, vous pouvez être invité à scanner un code QR.

    >[!Note]
    >S’il s’agit de la première fois que vous configurez l’application Microsoft Authenticator, vous pouvez recevoir une invite vous demandant si vous souhaitez autoriser l’application à accéder à votre appareil photo (iOS) ou à prendre des photos et à enregistrer des vidéos (Android). Vous devez sélectionner **Autoriser** pour que l’application Authenticator puisse accéder à votre appareil photo pour prendre une photo du code QR à l’étape suivante. Si vous n’autorisez pas l’accès à l’appareil photo, vous pouvez toujours installer l’application Authenticator, mais vous devrez ajouter les informations de code manuellement. Pour plus d’informations sur l’ajout manuel du code, consultez [Ajouter manuellement un compte à l’application](user-help-auth-app-add-account-manual.md).

Nous vous présentons ici le processus correspondant aux comptes Facebook, Google, GitHub et Amazon, mais ce processus est identique pour d’autres applications, telles qu’Instagram et Adobe.

## <a name="add-your-google-account"></a>Ajout de votre compte Google

Ajoutez votre compte Google en activant la vérification en deux étapes, puis en ajoutant le compte à l’application.

### <a name="turn-on-two-factor-verification"></a>Activation de la vérification en deux étapes

1. Sur votre ordinateur, accédez à https://myaccount.google.com/signinoptions/two-step-verification/enroll-welcome, sélectionnez **Prise en main**, puis vérifiez votre identité.

2. Suivez la procédure décrite sur la page pour activer la vérification en deux étapes pour votre compte Google personnel.

### <a name="add-your-google-account-to-the-app"></a>Ajouter votre compte Google à l'application

1. Dans la page de sécurité du compte Google sur votre ordinateur (https://myaccount.google.com/security) ), accédez à la section **Ajouter des étapes supplémentaires pour vérifier votre identité** et sélectionnez **Configurer** dans la section **Application Authenticator**.

2. Sur la page **Obtenir les codes de l'application Authenticator**, sélectionnez **Android** ou **iPhone** en fonction de votre type de téléphone, puis **Suivant**.

    Vous recevez un code QR que vous pouvez utiliser pour associer automatiquement votre compte à l’application Microsoft Authenticator. Ne fermez pas cette fenêtre.

3. Ouvrez l’application Microsoft Authenticator, sélectionnez **Ajouter un compte** à partir de l’icône **Personnaliser et contrôler** dans le coin supérieur droit, puis sélectionnez **Autre compte (Google, Facebook, etc.)** .

4. Utilisez l'appareil photo de votre appareil pour scanner le code QR de la page **Configurer Authentificator** de votre ordinateur.

    >[!Note]
    >Si votre appareil photo ne fonctionne pas correctement, vous pouvez entrer le code QR et l'URL manuellement.

5. Examinez la page **Comptes** de l'application Microsoft Authenticator sur votre appareil pour vous assurer que vos informations de compte sont correctes et qu'un code de vérification à six chiffres y est associé.

    Pour des raisons de sécurité, le code de vérification change toutes les 30 secondes, ce qui empêche quiconque d’utiliser plusieurs fois un même code.

6. Sélectionnez **Suivant** sur la page **Configurer Authenticator** de votre ordinateur, entrez le code de vérification à six chiffres fourni dans l'application pour votre compte Google, puis sélectionnez **Vérifier**.

7. Votre compte est vérifié, et vous pouvez sélectionner **Terminé** pour fermer la page **Configurer Authenticator**.

    >[!NOTE]
    >Pour plus d’informations sur la vérification en deux étapes et votre compte Google, consultez [Activer la vérification en deux étapes](https://support.google.com/accounts/answer/185839) et [En savoir plus sur la vérification en deux étapes](https://www.google.com/landing/2step/help.html).

## <a name="add-your-facebook-account"></a>Ajout de votre compte Facebook

Ajoutez votre compte Facebook en activant la vérification en deux étapes, puis en ajoutant le compte à l’application.

### <a name="turn-on-two-factor-verification"></a>Activation de la vérification en deux étapes

1. Sur votre ordinateur, ouvrez Facebook, sélectionnez le menu déroulant dans le coin supérieur droit, puis accédez à **Paramètres** > **Sécurité et connexion**.

    La page **Sécurité et connexion** s’affiche.

2. Accédez à l'option **Utiliser l'authentification à deux facteurs** dans la section **Authentification à deux facteurs**, puis sélectionnez **Modifier**.

    La page **Authentification à deux facteurs** s’affiche.

3. Sélectionnez **Activer**.

### <a name="add-your-facebook-account-to-the-app"></a>Ajouter votre compte Facebook à l'application

1. Sur la page Facebook de votre ordinateur, accédez à la section **Ajouter une sauvegarde**, puis sélectionnez **Configuration** dans la zone **Application d'authentification**.

    Vous recevez un code QR que vous pouvez utiliser pour associer automatiquement votre compte à l’application Microsoft Authenticator. Ne fermez pas cette fenêtre.

2. Ouvrez l’application Microsoft Authenticator, sélectionnez **Ajouter un compte** à partir de l’icône **Personnaliser et contrôler** dans le coin supérieur droit, puis sélectionnez **Autre compte (Google, Facebook, etc.)** .

3. Utilisez l'appareil photo de votre appareil pour scanner le code QR de la page **Authentification à deux facteurs** de votre ordinateur.

    >[!Note]
    >Si votre appareil photo ne fonctionne pas correctement, vous pouvez entrer le code QR et l'URL manuellement.

4. Examinez la page **Comptes** de l'application Microsoft Authenticator sur votre appareil pour vous assurer que vos informations de compte sont correctes et qu'un code de vérification à six chiffres y est associé.

    Pour des raisons de sécurité, le code de vérification change toutes les 30 secondes, ce qui empêche quiconque d’utiliser plusieurs fois un même code.

5. Sélectionnez **Suivant** sur la page **Authentification à deux facteurs** de votre ordinateur, puis entrez le code de vérification à six chiffres fourni dans l'application pour votre compte Facebook.

    Votre compte est vérifié, et vous pouvez dès à présent utiliser l’application pour vérifier votre compte.

    >[!NOTE]
    >Pour plus d’informations sur la vérification en deux étapes et votre compte Facebook, consultez [Présentation et fonctionnement de l’authentification à deux facteurs](https://www.facebook.com/help/148233965247823).

## <a name="add-your-github-account"></a>Ajouter votre compte GitHub

Ajoutez votre compte GitHub en activant la vérification en deux étapes, puis en ajoutant le compte à l’application.

### <a name="turn-on-two-factor-verification"></a>Activation de la vérification en deux étapes

1. Sur votre ordinateur, ouvrez GitHub, sélectionnez votre image dans le coin supérieur droit, puis cliquez sur **Paramètres**.

    La page **Authentification à deux facteurs** s’affiche.

2. Sélectionnez **Sécurité** dans la barre latérale **Paramètres personnels**, puis **Activer l’authentification à deux facteurs** dans la zone **Authentification à deux facteurs** .

### <a name="add-your-github-account-to-the-app"></a>Ajouter votre compte GitHub à l'application

1. Sur la page **Authentification à deux facteurs** de votre ordinateur, sélectionnez **Configurer à l'aide d'une application**.

2. Enregistrez vos codes de récupération afin de pouvoir retourner sur votre compte si vous en perdez l'accès, puis sélectionnez **Suivant**. 

    Pour enregistrer vos codes, téléchargez-les sur votre appareil, imprimez-les ou copiez-les dans un outil de gestion des mots de passe.

3. Sur la page **Authentification à deux facteurs**, sélectionnez **Configurer à l'aide d'une application**.

    La page affiche alors un code QR. Ne fermez pas cette page.

4. Ouvrez l’application Microsoft Authenticator, sélectionnez **Ajouter un compte** à partir de l’icône **Personnaliser et contrôler** dans le coin supérieur droit, sélectionnez **Autre compte (Google, Facebook, etc.)** , puis **Entrer ce code texte** à partir du texte situé en haut de la page.

    L’application Microsoft Authenticator n'est pas en mesure de scanner le code QR et vous devez donc l'entrer manuellement.

5. Entrez un **nom de compte** (par exemple, GitHub), la **clé secrète** de l’étape 4, puis sélectionnez **Terminer**.

6. Sur la page **Authentification à deux facteurs** de votre ordinateur, entrez le code de vérification à six chiffres fourni dans l'application pour votre compte GitHub, puis sélectionnez **Activer**.

    La page **Comptes** de l’application affiche votre nom de compte et un code de vérification à six chiffres. Pour des raisons de sécurité, le code de vérification change toutes les 30 secondes, ce qui empêche quiconque d’utiliser plusieurs fois un même code.

    >[!NOTE]
    >Pour plus d’informations sur la vérification en deux étapes et votre compte GitHub, consultez [À propos de l'authentification à deux facteurs](https://help.github.com/articles/about-two-factor-authentication/).

## <a name="add-your-amazon-account"></a>Ajouter votre compte Amazon

Ajoutez votre compte Amazon en activant la vérification en deux étapes, puis en ajoutant le compte à l’application.

### <a name="turn-on-two-factor-verification"></a>Activation de la vérification en deux étapes

1. Sur votre ordinateur, ouvrez Amazon, sélectionnez le menu déroulant **Compte et listes**, puis cliquez sur **Votre compte**.

2. Sélectionnez **Connexion et sécurité**, connectez-vous à votre compte Amazon, puis sélectionnez **Modifier** dans la zone **Paramètres de sécurité avancés**.

    La page **Paramètres de sécurité avancés** s'affiche.

3. Sélectionnez **Prise en main**.

4. Sélectionnez **Application Authenticator** à partir de la page **Choisir comment recevoir les codes**.

    La page affiche alors un code QR. Ne fermez pas cette page.

5. Ouvrez l’application Microsoft Authenticator, sélectionnez **Ajouter un compte** à partir de l’icône **Personnaliser et contrôler** dans le coin supérieur droit, puis sélectionnez **Autre compte (Google, Facebook, etc.)** .

6. Utilisez l'appareil photo de votre appareil pour scanner le code QR à partir de la page **Choisir comment recevoir les codes** de votre ordinateur.

    >[!Note]
    >Si votre appareil photo ne fonctionne pas correctement, vous pouvez entrer le code QR et l'URL manuellement.

7. Examinez la page **Comptes** de l'application Microsoft Authenticator sur votre appareil pour vous assurer que vos informations de compte sont correctes et qu'un code de vérification à six chiffres y est associé.

    Pour des raisons de sécurité, le code de vérification change toutes les 30 secondes, ce qui empêche quiconque d’utiliser plusieurs fois un même code.

8. Sur la page **Choisir comment recevoir les codes** de votre ordinateur, entrez le code de vérification à six chiffres fourni dans l'application pour votre compte Amazon, puis sélectionnez **Vérifier le code et continuer**.

9. Poursuivez le processus d’inscription en ajoutant notamment une méthode de vérification de sauvegarde comme un message texte, puis sélectionnez **Envoyer le code**.

10. Sur la page **Ajouter une méthode de vérification de sauvegarde** de votre ordinateur, entrez le code de vérification à six chiffres fourni par la méthode de vérification de sauvegarde pour votre compte Amazon, puis sélectionnez **Vérifier le code et continuer**.

11. Sur la page **Presque terminé**, indiquez si vous souhaitez faire de votre ordinateur un appareil de confiance, puis sélectionnez **Compris. Activer la vérification en deux étapes**.

    La page **Paramètres de sécurité avancés** s’ouvre et affiche les détails mis à jour de la vérification en deux étapes.

    >[!NOTE]
    >Pour plus d’informations sur la vérification en deux étapes et votre compte Amazon, consultez [À propos de la vérification en deux étapes](https://www.amazon.com/gp/help/customer/display.html?nodeId=201596330) et [En savoir plus sur la vérification en deux étapes](https://www.amazon.com/gp/help/customer/display.html?nodeId=201962440).

## <a name="next-steps"></a>Étapes suivantes

- Après avoir ajouté vos comptes à l’application, vous pouvez vous connecter à l’aide de l’application Authenticator sur votre appareil. Pour plus d’informations, consultez l’article [Se connecter à l’aide de l’application Microsoft Authenticator](user-help-auth-app-sign-in.md).

- Pour les appareils exécutant iOS, vous pouvez également sauvegarder les informations d’identification de votre compte et les paramètres de l’application associée, comme l’ordre de vos comptes, dans le cloud. Pour plus d’informations, consultez l’article [Sauvegarder et récupérer des informations d’authentification de compte avec l’application Microsoft Authenticator](user-help-auth-app-backup-recovery.md).
