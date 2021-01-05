---
title: Modifiez la méthode et les paramètres de vérification en deux étapes – Azure Active Directory
description: Découvrez comment modifier la méthode et les paramètres de vérification de sécurité pour votre compte professionnel ou scolaire à partir de la page de vérification de sécurité supplémentaire.
services: active-directory
author: curtand
manager: daveba
ms.reviewer: richagi
ms.assetid: d3372d9a-9ad1-4609-bdcf-2c4ca9679a3b
ms.workload: identity
ms.service: active-directory
ms.subservice: user-help
ms.topic: end-user-help
ms.date: 07/06/2020
ms.author: curtand
ms.openlocfilehash: e0a6c566e8e0dfb77b5899f735020d0f1facf3d1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88798378"
---
# <a name="change-your-two-factor-verification-method-and-settings"></a>Modifier la méthode et les paramètres de vérification à deux facteurs

Une fois que vous avez configuré vos méthodes de vérification de sécurité pour votre compte professionnel ou scolaire, vous pouvez mettre à jour tous les détails associés, notamment :

- Méthode de vérification de sécurité par défaut

- Détails de la méthode de vérification de sécurité, comme votre numéro de téléphone

- Configuration d’une nouvelle application d’authentification ou suppression d’un appareil de l’application d’authentification

## <a name="using-the-additional-security-verification-page"></a>Utilisation de la page Vérification de sécurité supplémentaire

Si votre organisation vous a fourni des étapes spécifiques expliquant comment activer et gérer la vérification en deux étapes, vous devez d’abord suivre ces instructions. Dans le cas contraire, vous pouvez accéder aux paramètres de méthode de vérification de sécurité à partir de la page [Vérification de sécurité supplémentaire](./multi-factor-authentication-end-user-first-time.md).

>[!Note]
>Si ce que vous voyez à l’écran ne correspond pas à ce qui est abordé dans cet article, cela signifie que votre administrateur a activé l’expérience **Informations de sécurité (préversion)** ou que votre organisation dispose d’un portail personnalisé. Pour en savoir plus sur la nouvelle expérience Informations de sécurité, consultez [Vue d’ensemble des informations de sécurité (préversion)](./security-info-setup-signin.md). Pour plus d’informations sur le portail personnalisé de votre organisation, vous devez contacter le support technique de votre organisation.

### <a name="to-get-to-the-additional-security-verification-page"></a>Pour accéder à la page Vérification de sécurité supplémentaire

Vous pouvez suivre ce lien pour accéder à la [page Vérification de sécurité supplémentaire](https://account.activedirectory.windowsazure.com/proofup.aspx?proofup=1).

![Page Vérification de sécurité supplémentaire, avec les détails de la méthode de vérification de sécurité disponibles](./media/multi-factor-authentication-end-user-manage-settings/mfa-security-verification-page.png)

Vous pouvez également accéder à la page **Vérification de sécurité supplémentaire** en effectuant les étapes suivantes :

1. Connectez-vous à [https://myapps.microsoft.com](https://myapps.microsoft.com).

1. Sélectionnez votre nom de compte dans la partie supérieure droite, puis sélectionnez **profil**.

1. Sélectionnez **Vérification de sécurité supplémentaire**.  

    ![Lien Mes applications vers la page Vérification de sécurité supplémentaire](./media/multi-factor-authentication-end-user-manage-settings/mfa-myapps-link.png)

>[!Note]
>Pour plus d’informations sur l’utilisation de la section **Mots de passe d’application** de la page **Vérification de sécurité supplémentaire**, consultez [Gérer les mots de passe d’application pour la vérification à deux facteurs](multi-factor-authentication-end-user-app-passwords.md). Les mots de passe d’application ne doivent être utilisés que pour les applications qui ne prennent pas en charge la vérification en deux étapes.

## <a name="change-your-default-security-verification-method"></a>Changer la méthode de vérification de sécurité par défaut

Une fois que vous êtes connecté à votre compte professionnel ou scolaire avec votre nom d’utilisateur et votre mot de passe, la méthode de vérification de sécurité que vous avez choisie vous est présentée automatiquement. En fonction des besoins de votre organisation, il peut s’agir d’un code de vérification ou d’une notification par le biais d’une application d’authentification, d’un SMS ou d’un appel téléphonique.

Si vous souhaitez changer la méthode de vérification de sécurité par défaut utilisée, vous pouvez le faire à partir de cet emplacement.

### <a name="to-change-your-default-security-verification-method"></a>Pour changer la méthode de vérification de sécurité par défaut

1. Dans la page **Vérification de sécurité supplémentaire**, sélectionnez la méthode à utiliser dans la liste **Quelle est votre option préférée ?** Toutes les options sont affichées, mais vous pouvez sélectionner uniquement celles mises à disposition par votre organisation.

    - **M’avertir via l’application** : Vous serez averti par le biais de votre application d’authentification que vous avez une invite de vérification en attente.

    - **Appeler mon téléphone d’authentification** : Vous recevrez un appel téléphonique sur votre appareil mobile vous invitant à vérifier vos informations.

    - **Envoyer un code par SMS à mon téléphone d’authentification** : Vous recevrez un code de vérification dans un SMS sur votre appareil mobile. Vous devez entrer ce code en réponse à l’invite de vérification pour votre compte professionnel ou scolaire.

    - **Appeler mon téléphone de bureau** : Vous recevrez un appel téléphonique sur votre téléphone de bureau vous invitant à vérifier vos informations.

    - **Utiliser le code de vérification fourni par l’application** : Vous utiliserez votre application d’authentification pour obtenir un code de vérification que vous taperez en réponse à l’invite pour votre compte professionnel ou scolaire.

2. Sélectionnez **Enregistrer**.

## <a name="add-or-change-your-phone-number"></a>Ajouter ou modifier votre numéro de téléphone

Vous pouvez ajouter de nouveaux numéros de téléphone ou mettre à jour des numéros existants à partir de la page **Vérification de sécurité supplémentaire**.

>[!Important]
>Nous vous recommandons vivement d’ajouter un numéro de téléphone secondaire pour empêcher le verrouillage de votre compte en cas de perte ou de vol de votre téléphone principal, ou si vous recevez un nouveau téléphone et que vous n’avez plus votre numéro de téléphone principal d’origine.

### <a name="to-change-your-phone-numbers"></a>Pour modifier vos numéros de téléphone

1. Dans la section **Comment aimeriez-vous répondre ?** de la page **Vérification de sécurité supplémentaire**, mettez à jour les informations de numéro de téléphone de votre **Téléphone d’authentification** (votre appareil mobile principal) et de votre **Téléphone de bureau**.

1. Cochez la case à côté de l’option **Téléphone d’authentification secondaire**, puis tapez un numéro de téléphone secondaire auquel vous pouvez recevoir des appels téléphoniques si vous ne pouvez pas accéder à votre appareil principal.

1. Sélectionnez **Enregistrer**.

## <a name="add-a-new-account-to-the-microsoft-authenticator-app"></a>Ajouter un nouveau compte à l’application Microsoft Authenticator

Vous pouvez configurer votre compte professionnel ou scolaire dans l’application Microsoft Authenticator pour [Android](https://play.google.com/store/apps/details?id=com.azure.authenticator) ou [iOS](https://apps.apple.com/app/microsoft-authenticator/id983156458).

Si vous avez déjà configuré votre compte professionnel ou scolaire dans l’application Microsoft Authenticator, vous n’avez pas besoin de le refaire.

1. Dans la section **Comment aimeriez-vous répondre ?** de la page **Vérification de sécurité supplémentaire**, sélectionnez **Configurer une application d’authentification**.

    ![Configurer votre compte professionnel ou scolaire dans l’application Microsoft Authenticator](./media/multi-factor-authentication-end-user-manage-settings/mfa-security-verification-page-auth-app.png)

1. Suivez les instructions à l’écran, notamment concernant l’utilisation de votre appareil mobile pour scanner le code QR, puis sélectionnez **Suivant**.

    Vous serez invité à approuver une notification par le biais de l’application Microsoft Authenticator, afin de vérifier vos informations.

1. Sélectionnez **Enregistrer**.

## <a name="delete-your-account-or-device-from-the-microsoft-authenticator-app"></a>Supprimer votre compte ou appareil de l’application Microsoft Authenticator

Vous pouvez supprimer votre compte de l’application Microsoft Authenticator, et vous pouvez supprimer votre appareil de votre compte professionnel ou scolaire. En règle générale, vous supprimez votre appareil afin de supprimer définitivement un appareil perdu, volé ou ancien de votre compte, et vous supprimez votre compte afin de tenter de résoudre certains problèmes de connexion ou de modifier un compte, par exemple avec un nouveau nom d’utilisateur.

### <a name="to-delete-your-device-from-your-work-or-school-account"></a>Pour supprimer votre appareil de votre compte professionnel ou scolaire

1. Dans la section **Comment aimeriez-vous répondre ?** de la page **Vérification de sécurité supplémentaire**, sélectionnez le bouton **Configurer une application d’authentification**.

1. Sélectionnez **Enregistrer**.

### <a name="to-delete-your-account-from-the-microsoft-authenticator-app"></a>Pour supprimer votre compte de l’application Microsoft Authenticator

À partir de l’application Microsoft Authenticator, sélectionnez le bouton **Supprimer** en regard de l’appareil que vous souhaitez supprimer.

## <a name="turn-on-two-factor-verification-prompts-on-a-trusted-device"></a>Activer les invites de vérification à deux facteurs sur un appareil de confiance

En fonction des paramètres de votre organisation, une case à cocher **Ne plus me le demander pendant X jours** peut s’afficher quand vous effectuez une vérification à deux facteurs sur votre navigateur. Si vous avez sélectionné cette option pour arrêter l’affichage des invites de vérification en deux étapes, et que vous perdez ensuite votre appareil ou que celui-ci est potentiellement compromis, vous devez réactiver les invites de vérification en deux étapes pour vous aider à protéger votre compte. Vous devez activer les invites pour tous vos appareils en même temps. Malheureusement, vous ne pouvez pas réactiver les invites uniquement pour un appareil spécifique.

### <a name="to-turn-two-factor-verification-prompts-back-on-for-your-devices"></a>Pour réactiver les invites de vérification à deux facteurs pour vos appareils

Dans la [page **Vérification de sécurité supplémentaire**](#to-get-to-the-additional-security-verification-page), sélectionnez **Restaurer l’authentification multifacteur sur des appareils précédemment définis comme étant de confiance**. La prochaine fois que vous vous connecterez sur l’un des appareils, vous serez invité à effectuer une vérification à deux facteurs.

## <a name="next-steps"></a>Étapes suivantes

Une fois que vous avez ajouté ou mis à jour vos paramètres de vérification en deux étapes, vous pouvez gérer vos mots de passe d’application, vous connecter ou obtenir de l’aide pour certains problèmes courants liés à la vérification en deux étapes.

- [Gérer les mots de passe d’application pour la vérification à deux facteurs](multi-factor-authentication-end-user-app-passwords.md) pour les applications qui ne prennent pas en charge la vérification à deux facteurs.

- [Comment se connecter à l’aide de la vérification en deux étapes](multi-factor-authentication-end-user-signin.md)

- [Résoudre les problèmes courants de la vérification en deux étapes](multi-factor-authentication-end-user-troubleshoot.md)