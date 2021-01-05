---
title: Configurer un appareil mobile comme méthode de vérification à deux facteurs - Azure Active Directory | Microsoft Docs
description: Découvrez comment configurer un appareil mobile comme méthode de vérification à deux facteurs.
services: active-directory
author: curtand
manager: daveba
ms.service: active-directory
ms.subservice: user-help
ms.workload: identity
ms.topic: end-user-help
ms.date: 08/12/2019
ms.author: curtand
ms.openlocfilehash: 40af9ac692a226b12f984db447db9635cba655e4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91530253"
---
# <a name="set-up-a-mobile-device-as-your-two-factor-verification-method"></a>Configurer un appareil mobile comme méthode de vérification à deux facteurs

Vous pouvez configurer un appareil mobile comme méthode de vérification à deux facteurs. Votre téléphone mobile peut recevoir un SMS avec un code de vérification ou un appel téléphonique.

>[!Note]
> Si l’option de téléphone d’authentification est grisée, il est possible que votre organisation ne vous autorise pas à utiliser un numéro de téléphone ou SMS à des fins de vérification. Dans ce cas, vous devez choisir une autre méthode ou contacter votre administrateur pour obtenir de l’aide.

## <a name="set-up-your-mobile-device-to-use-a-text-message-as-your-verification-method"></a>Configurer votre appareil mobile pour qu’il utilise un SMS comme méthode de vérification

1. Dans la page **Vérification de sécurité supplémentaire**, sélectionnez **Téléphone d’authentification** dans la zone **Étape 1 : Comment devons-nous vous contacter**, sélectionnez votre pays ou région dans la liste déroulante, puis tapez le numéro de téléphone de votre appareil mobile.

2. Sélectionnez **M’envoyer un code par SMS** à partir de la zone **Méthode**, puis sélectionnez **Suivant**.

    ![Capture d’écran montrant la page « Vérification de sécurité supplémentaire » avec les options « Téléphone d’authentification » et « M’envoyer un code par SMS » sélectionnées.](media/multi-factor-authentication-verification-methods/multi-factor-authentication-text-message.png)

3. Tapez le code de vérification du SMS envoyé par Microsoft dans la zone **Étape 2 : Nous avons envoyé un SMS à votre téléphone**, puis sélectionnez **Vérifier**.

    ![Page Vérification de sécurité supplémentaire, avec téléphone d’authentification et SMS](media/multi-factor-authentication-verification-methods/multi-factor-authentication-text-message-test.png)

4. Dans la zone **Étape 3 : Continuez à utiliser vos applications existantes**, copiez le mot de passe d’application fourni et collez-le à un endroit sûr.

    ![Zone Mots de passe d’application de la page Vérification de sécurité supplémentaire](media/multi-factor-authentication-verification-methods/multi-factor-authentication-app-passwords.png)

    >[!Note]
    >Pour plus d’informations sur l’utilisation du mot de passe d’application avec vos anciennes applications, consultez [Gérer les mots de passe d’application](multi-factor-authentication-end-user-app-passwords.md). Vous devez uniquement utiliser les mots de passe d’application si vous continuez à utiliser des applications plus anciennes qui ne prennent pas en charge la vérification à deux facteurs.

5. Sélectionnez **Terminé**.

## <a name="set-up-your-mobile-device-to-receive-a-phone-call"></a>Configurer votre appareil mobile pour recevoir un appel téléphonique

1. Dans la page **Vérification de sécurité supplémentaire**, sélectionnez **Téléphone d’authentification** dans la zone **Étape 1 : Comment devons-nous vous contacter**, sélectionnez votre pays ou région dans la liste déroulante, puis tapez le numéro de téléphone de votre appareil mobile.

2. Sélectionnez **M’appeler** dans la zone **Méthode**, puis sélectionnez **Suivant**.

    ![Page Vérification de sécurité supplémentaire, avec téléphone d’authentification et appel de téléphone](media/multi-factor-authentication-verification-methods/multi-factor-authentication-phone-call.png)

3. Vous recevez alors un appel téléphonique de Microsoft, vous demandant d’appuyer sur le signe dièse (#) sur votre appareil mobile pour vérifier votre identité.

    ![Test du numéro de téléphone spécifié](media/multi-factor-authentication-verification-methods/multi-factor-authentication-phone-call-test.png)

4. Dans la zone **Étape 3 : Continuez à utiliser vos applications existantes**, copiez le mot de passe d’application fourni et collez-le à un endroit sûr.

    ![Zone Mots de passe d’application de la page Vérification de sécurité supplémentaire](media/multi-factor-authentication-verification-methods/multi-factor-authentication-app-passwords.png)

    >[!Note]
    >Pour plus d’informations sur l’utilisation du mot de passe d’application avec vos anciennes applications, consultez [Gérer les mots de passe d’application](multi-factor-authentication-end-user-app-passwords.md). Vous devez uniquement utiliser les mots de passe d’application si vous continuez à utiliser des applications plus anciennes qui ne prennent pas en charge la vérification à deux facteurs.

5. Sélectionnez **Terminé**.

## <a name="next-steps"></a>Étapes suivantes

Une fois que vous avez configuré votre méthode de vérification à deux facteurs, vous pouvez ajouter des méthodes supplémentaires, gérer vos paramètres et mots de passe d’application, vous connecter ou obtenir de l’aide pour certains problèmes courants liés à la vérification à deux facteurs.

- [Gérer les paramètres de la vérification à deux facteurs](multi-factor-authentication-end-user-manage-settings.md)

- [Gérer les mots de passe d’application](multi-factor-authentication-end-user-app-passwords.md)

- [Se connecter à l’aide de la vérification en deux étapes](multi-factor-authentication-end-user-signin.md)

- [Obtenir de l’aide pour la vérification à deux facteurs](multi-factor-authentication-end-user-troubleshoot.md)
