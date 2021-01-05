---
title: Joindre votre appareil professionnel au réseau de votre organisation – AD
description: Apprenez à joindre votre appareil professionnel au réseau de votre organisation.
services: active-directory
author: curtand
manager: daveba
ms.assetid: 54e1b01b-03ee-4c46-bcf0-e01affc0419d
ms.service: active-directory
ms.subservice: user-help
ms.workload: identity
ms.topic: end-user-help
ms.date: 08/03/2018
ms.author: curtand
ms.reviewer: jairoc
ms.openlocfilehash: efb3ee24add847baf8264eccdf71278be5ee9496
ms.sourcegitcommit: f5580dd1d1799de15646e195f0120b9f9255617b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/29/2020
ms.locfileid: "91536968"
---
# <a name="join-your-work-device-to-your-organizations-network"></a>Joindre votre appareil professionnel au réseau de votre organisation
Joignez votre appareil Windows 10 professionnel au réseau de votre organisation afin de pouvoir accéder à des ressources potentiellement restreintes.

## <a name="what-happens-when-you-join-your-device"></a>Ce qui se passe quand vous joignez votre appareil
Quand vous joignez votre appareil Windows 10 au réseau de votre organisation, les actions suivantes se produisent :

- Windows inscrit votre appareil sur le réseau de votre organisation, vous permettant d’accéder à vos ressources à l’aide de votre compte personnel. Une fois votre appareil inscrit, Windows le joint au réseau ; vous pouvez alors utiliser le nom d’utilisateur et le mot de passe de votre organisation pour vous connecter et accéder aux ressources restreintes.

- Selon les choix de votre organisation, vous devrez peut-être configurer une vérification en deux étapes à travers [l’authentification multifacteur](multi-factor-authentication-end-user-first-time.md) ou des [informations de sécurité](./security-info-setup-signin.md).

- Selon les choix de votre organisation, vous pouvez être automatiquement inscrit dans une solution de gestion des appareils mobiles, comme Microsoft Intune. Pour plus d’informations sur l’inscription dans Microsoft Intune, consultez [Inscrire votre appareil sur Intune](/intune-user-help/enroll-your-device-in-intune-all).

- Vous suivrez le processus de connexion en utilisant l’authentification automatique avec votre compte professionnel.

## <a name="to-join-a-brand-new-windows-10-device"></a>Pour joindre un tout nouvel appareil Windows 10
Si votre appareil est tout nouveau et n’a pas encore été configuré, vous pouvez suivre le processus OOBE (Out of Box Experience) de Windows pour le joindre au réseau.

1. Démarrez votre nouvel appareil et commencez le processus OOBE.

2. Dans l’écran **Se connecter avec Microsoft**, tapez votre adresse e-mail professionnelle ou scolaire.

    ![Écran de connexion avec adresse e-mail](./media/user-help-join-device-on-network/join-device-oobe-signin.png)

3. Dans l’écran **Entrez votre mot de passe**, tapez votre mot de passe.

    ![Écran Entrez votre mot de passe](./media/user-help-join-device-on-network/join-device-oobe-password.png)

4. Sur votre appareil mobile, approuvez ce dernier afin qu’il puisse accéder à votre compte. 

    ![Capture d’écran représentant l’écran de notification « Entrez votre mot de passe ».](./media/user-help-join-device-on-network/join-device-oobe-mobile.png)

5. Terminez le processus OOBE, notamment en définissant vos paramètres de confidentialité et en configurant Windows Hello (si nécessaire).

    Votre appareil est maintenant joint au réseau de votre organisation.

## <a name="to-make-sure-youre-joined-new-device"></a>Pour vérifier que vous êtes joint (nouvel appareil)
Vous pouvez vérifier que vous êtes joint en examinant vos paramètres.

1. Ouvrez **Paramètres**, puis sélectionnez **Comptes**.

    ![Comptes dans l’écran Paramètres](./media/user-help-join-device-on-network/join-device-settings-accounts.png)

2. Sélectionnez **Accès Professionnel ou Scolaire** et vérifiez que vous voyez un texte similaire à **Connecté à *\<your_organization>* Azure AD**.

    ![Capture d’écran représentant la fenêtre « Accès Professionnel ou Scolaire » dans laquelle le compte « Connecté à (votre organisation) Azure AD » est sélectionné.](./media/user-help-join-device-on-network/join-device-oobe-verify.png)


## <a name="to-join-an-already-configured-windows-10-device"></a>Pour joindre un appareil Windows 10 déjà configuré
Si vous avez votre appareil depuis un certain temps et qu’il a déjà été configuré, vous pouvez suivre ces étapes pour le joindre au réseau.

1. Ouvrez **Paramètres**, puis sélectionnez **Comptes**.

2. Sélectionnez **Accès Professionnel ou Scolaire**, puis sélectionnez **Se connecter**.

    ![Accès Professionnel ou scolaire avec liens de connexion](./media/user-help-join-device-on-network/join-device-access-work-school-connect.png)

3. Dans l’écran **Configurer un compte professionnel ou scolaire**, sélectionnez **Joindre cet appareil à Azure Active Directory**.

    ![Écran Configurer un compte professionnel ou scolaire](./media/user-help-join-device-on-network/join-device-setup-join-aad.png)

4. Dans l’écran **Procédons à votre connexion**, tapez votre adresse e-mail (par exemple, alain@contoso.com), puis sélectionnez **Suivant**.

    ![Écran Procédons à votre connexion](./media/user-help-join-device-on-network/join-device-setup-get-signed-in.png)

5. Dans l’écran **Entrez votre mot de passe**, tapez votre mot de passe et sélectionnez **Se connecter**.

    ![Boîte de dialogue Saisie du mot de passe](./media/user-help-join-device-on-network/join-device-setup-password.png)

6. Sur votre appareil mobile, approuvez ce dernier afin qu’il puisse accéder à votre compte. 

    ![Écran de notification par appareil mobile](./media/user-help-join-device-on-network/join-device-setup-mobile.png)

7. Dans l’écran **Vérifiez qu’il s’agit de votre organisation**, passez en revue les informations pour vous assurer qu’elles sont correctes, puis sélectionnez **Joindre**.

    ![Écran de vérification Vérifiez qu’il s’agit de votre organisation](./media/user-help-join-device-on-network/join-device-setup-confirm.png)

8. Dans l’écran **Vous avez terminé**, cliquez sur **Terminé**.

    ![Écran Vous avez terminé](./media/user-help-join-device-on-network/join-device-setup-finish.png)

## <a name="to-make-sure-youre-joined"></a>Pour vérifier que vous êtes joint
Vous pouvez vérifier que vous êtes joint en examinant vos paramètres.

1. Ouvrez **Paramètres**, puis sélectionnez **Comptes**.

    ![Comptes dans l’écran Paramètres](./media/user-help-join-device-on-network/join-device-settings-accounts.png)

2. Sélectionnez **Accès Professionnel ou Scolaire** et vérifiez que vous voyez un texte similaire à **Connecté à *\<your_organization>* Azure AD**.

    ![Écran Accès Professionnel ou Scolaire avec compte contoso connecté](./media/user-help-join-device-on-network/join-device-setup-verify.png)

## <a name="next-steps"></a>Étapes suivantes
Après avoir joint votre appareil au réseau de votre organisation, vous devriez pouvoir accéder à toutes vos ressources à l’aide des informations de votre compte professionnel ou scolaire.

- Si votre organisation vous demande d’inscrire votre appareil personnel, tel que votre téléphone, consultez [Inscrire votre appareil personnel sur le réseau de votre organisation](user-help-register-device-on-network.md).

- Si votre organisation est managée à l’aide de Microsoft Intune et que vous avez des questions sur l’inscription, l'ouverture de session ou tout autre problème lié à Intune, consultez l’[aide utilisateur Intune](/intune-user-help/use-managed-devices-to-get-work-done).