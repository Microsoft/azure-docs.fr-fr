---
title: Sauvegarder et récupérer des comptes avec l’application Microsoft Authenticator – Azure AD
description: Découvrez comment sauvegarder et récupérer les informations d’identification sauvegardées de votre compte à l’aide de l’application Microsoft Authenticator.
services: active-directory
author: curtand
manager: daveba
ms.subservice: user-help
ms.service: active-directory
ms.workload: identity
ms.topic: end-user-help
ms.date: 06/03/2020
ms.author: curtand
ms.reviewer: olhaun
ms.openlocfilehash: e7608196c962a6918a90b91457f856f3e11b04f0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91530899"
---
# <a name="back-up-and-recover-account-credentials-using-the-microsoft-authenticator-app"></a>Sauvegarder et récupérer des informations d’identification de compte avec l’application Microsoft Authenticator

**S’applique à :**

- appareils iOS exécutant les versions 5.7.0 et ultérieures

- appareils Android exécutant les versions 6.6.0 et ultérieures

L’application Microsoft Authenticator sauvegarde les informations d’identification de votre compte et les paramètres de l’application associée, comme l’ordre de vos comptes, dans le cloud. Après la sauvegarde, vous pouvez également utiliser l’application pour récupérer vos informations sur un nouvel appareil, ce qui évite de se retrouver bloqué ou d’avoir à recréer des comptes.

Chaque emplacement de stockage de sauvegarde exige un seul compte Microsoft personnel, en sachant qu’iOS exige également un compte iCloud. Vous pouvez avoir plusieurs comptes stockés à cet emplacement unique. Par exemple, vous pouvez avoir un compte personnel, un compte professionnel ou scolaire et un compte personnel non-Microsoft comme Facebook, Google, etc.

> [!IMPORTANT]
> Seules les informations d’identification de votre compte personnel et tiers sont stockées, c’est-à-dire votre nom d’utilisateur et le code de vérification de compte indispensable pour prouver votre identité. Nous ne stockons aucune autre information liée à vos comptes, notamment aucune adresse e-mail ni aucun fichier. De plus, nous n’associons pas et ne partageons pas vos comptes de quelque manière que ce soit, avec un autre produit ou service. Enfin, votre administrateur informatique n’aura accès à aucune information sur ces comptes.

## <a name="back-up-your-account-credentials"></a>Sauvegarder les informations d’identification de votre compte

Pour pouvoir sauvegarder vos informations d’identification, vous devez avoir :

- Un [compte Microsoft](https://account.microsoft.com/account) personnel comme votre compte de récupération.

- **Pour iOS uniquement,** vous devez disposer d’un [compte iCloud](https://www.icloud.com/) pour l’emplacement de stockage réel.

### <a name="to-turn-on-cloud-backup-for-ios-devices"></a>Pour activer la sauvegarde cloud pour les appareils iOS

- Sur votre appareil iOS, sélectionnez **Paramètres**, **Sauvegarde**, puis activez **Sauvegarde iCloud**.

    Les informations d’identification de votre compte sont sauvegardées dans votre compte iCloud.

    ![Écran des paramètres iOS affichant l’emplacement des paramètres de sauvegarde iCloud](./media/user-help-auth-app-backup-recovery/backup-and-recovery-turn-on.png)

### <a name="to-turn-on-cloud-backup-for-android-devices"></a>Pour activer la sauvegarde cloud pour les appareils Android

- Sur votre appareil Android, sélectionnez **Paramètres**, **Sauvegarde**, puis activez **Sauvegarde cloud**.

    Les informations d’identification de votre compte sont sauvegardées dans votre compte cloud.

    ![Écran des paramètres Android montrant l’emplacement des paramètres de sauvegarde](./media/user-help-auth-app-backup-recovery/backup-and-recovery-turn-on-android.png)

## <a name="recover-your-account-credentials-on-your-new-device"></a>Récupérer les informations d’identification de votre compte sur votre nouvel appareil

Vous pouvez récupérer les informations d’identification de votre compte à partir de votre compte cloud, mais vous devez d’abord vous assurer que le compte que vous récupérez n’existe pas dans l’application Microsoft Authenticator. Par exemple, si vous récupérez votre compte Microsoft personnel, vous devez vous assurer que vous n’avez pas de compte Microsoft personnel déjà configuré dans l’application Authenticator. Cette vérification est importante, car elle nous permet de ne pas remplacer ou effacer un compte existant par erreur.

### <a name="to-recover-your-information"></a>Pour récupérer vos informations

1. Sur votre appareil mobile, ouvrez l’application Microsoft Authenticator et sélectionnez **Commencer la récupération** au bas de l’écran.

    ![Application Microsoft Authenticator, indiquant où cliquer sur Begin recovery (Lancer la récupération)](./media/user-help-auth-app-backup-recovery/backup-and-recovery-begin-recovery.png)

2. Connectez-vous à votre compte de récupération, à l’aide du compte Microsoft personnel que vous avez utilisé pendant le processus de sauvegarde.

    Les informations d’identification de votre compte sont récupérées sur le nouvel appareil.

Une fois la récupération terminée, vous pouvez constater que les codes de vérification de votre compte Microsoft personnel dans l’application Microsoft Authenticator sont différents sur vos ancien et nouveau téléphones. Ces codes sont différents car chaque appareil a ses propres informations d’identification, mais les deux sont valides et fonctionnent lors de la connexion à l’aide du téléphone associé.

## <a name="recover-accounts-requiring-more-verification"></a>Récupérer des comptes nécessitant davantage de vérifications

Si vous utilisez des notifications Push avec vos comptes personnel, professionnel ou scolaire, une alerte s’affiche à l’écran et vous demande une vérification supplémentaire pour pouvoir récupérer vos informations. Comme les notifications Push requièrent l’utilisation d’une information d’identification liée à votre appareil et ne transitant jamais sur le réseau, vous devez prouver votre identité avant la création de l’information d’identification sur votre appareil.

Pour des comptes Microsoft personnels, vous pouvez prouver votre identité en saisissant votre mot de passe ainsi qu’une autre adresse e-mail ou un autre numéro de téléphone. Pour des comptes professionnels ou scolaires, vous devez lire le code QR que le fournisseur de votre compte vous a transmis.

### <a name="to-provide-more-verification-for-personal-accounts"></a>Pour effectuer une vérification supplémentaire des comptes personnels

1. Dans l’écran **Comptes** de l’application Microsoft Authenticator, appuyez sur le compte à récupérer pour l’ouvrir en plein écran.

    :::image type="content" source="media/user-help-auth-app-backup-recovery/backup-and-recovery-arrow.png" alt-text="Capture d’écran montrant l’application Microsoft Authenticator avec les vignettes de comptes disponibles." border="true":::

1. Appuyez sur la vignette du compte que vous récupérez, puis sur l’option de connexion pour procéder à la récupération. Entrez votre mot de passe, puis confirmez votre adresse e-mail ou votre numéro de téléphone comme vérification supplémentaire.

    :::image type="content" source="media/user-help-auth-app-backup-recovery/backup-and-recovery-codes.png" alt-text="Capture d’écran montrant l’application Microsoft Authenticator avec les vignettes de comptes disponibles." border="true":::

### <a name="to-provide-more-verification-for-work-or-school-accounts"></a>Pour effectuer une vérification supplémentaire des comptes professionnels ou scolaires

1. Dans l’écran **Comptes** de l’application Microsoft Authenticator, appuyez sur le compte à récupérer pour l’ouvrir en plein écran.

    :::image type="content" source="media/user-help-auth-app-backup-recovery/work-or-school-recovery-arrow.png" alt-text="Capture d’écran montrant l’application Microsoft Authenticator avec les vignettes de comptes disponibles." border="true":::

1. Dans l’affichage plein écran, appuyez sur l’option permettant d’analyser un code QR pour effectuer une récupération complète.

    :::image type="content" source="media/user-help-auth-app-backup-recovery/work-or-school-recovery-qr-code.png" alt-text="Capture d’écran montrant l’application Microsoft Authenticator avec les vignettes de comptes disponibles." border="true":::

>[!NOTE]
>Pour plus d’informations sur les codes QR et leur obtention, consultez [Bien démarrer avec l’application Microsoft Authenticator](./user-help-auth-app-download-install.md) ou [Configurer les informations de sécurité pour utiliser une application d’authentification](./security-info-setup-auth-app.md), selon que votre administrateur a activé des informations de sécurité ou non.
>
>S’il s’agit de la première fois que vous configurez l’application Microsoft Authenticator, vous pouvez recevoir une invite vous demandant si vous souhaitez autoriser l’application à accéder à votre appareil photo (iOS) ou à prendre des photos et à enregistrer des vidéos (Android). Vous devez sélectionner **Autoriser** pour que l’application Authenticator puisse accéder à votre appareil photo pour prendre une photo du code QR à l’étape suivante. Si vous n’autorisez pas l’accès à l’appareil photo, vous pouvez toujours installer l’application Authenticator, mais vous devrez ajouter les informations de code manuellement. Pour plus d’informations sur l’ajout manuel du code, consultez [Ajouter manuellement un compte à l’application](user-help-auth-app-add-account-manual.md).

## <a name="troubleshoot-backup-and-recovery-problems"></a>Résoudre les problèmes de sauvegarde et de récupération

Plusieurs raisons peuvent expliquer le fait que votre sauvegarde ne soit pas disponible

- **Modification des systèmes d’exploitation** : Votre sauvegarde est stockée dans le iCloud pour iOS et dans le fournisseur de stockage cloud de Microsoft pour Android. Cela signifie que votre sauvegarde n’est pas disponible si vous passez d’un appareil Android à un appareil iOS ou vice versa. Si vous changez de système d’exploitation, vous devez recréer manuellement vos comptes dans l’application Microsoft Authenticator.

- **Problèmes réseau** : Si vous rencontrez des problèmes réseau, vérifiez que vous êtes connecté au réseau et correctement connecté à votre compte.

- **Problèmes de compte** : Si vous rencontrez des problèmes de compte, vérifiez que vous êtes correctement connecté à votre compte. Pour iOS, cela signifie que vous devez être connecté à iCloud à l’aide du même compte AppleID que votre iPhone.

- **Suppression accidentelle** : Il est possible que vous ayez supprimé votre compte de sauvegarde de votre précédent appareil ou lors de la gestion de votre compte de stockage cloud. Dans ce cas, vous devez recréer manuellement votre compte dans l’application.

- **Comptes Microsoft Authenticator existants** : Si vous avez déjà créé des comptes dans l’application Microsoft Authenticator, elle ne pourra pas récupérer vos comptes sauvegardés. Bloquer la récupération permet d’éviter que les détails de votre compte ne soient remplacés par des informations obsolètes. Dans ce cas, vous devez supprimer les informations d’identification des comptes existants créés dans votre application Authenticator pour pouvoir récupérer votre sauvegarde.

- **La sauvegarde est obsolète** : Si vos informations de sauvegarde sont obsolètes, vous pouvez être invité à actualiser les informations en vous reconnectant à votre compte de récupération Microsoft. Votre compte de récupération correspond au compte Microsoft personnel que vous avez utilisé initialement pour stocker votre sauvegarde. Si une connexion est requise, un point rouge s’affiche dans votre menu ou barre d’action, ou un point d’exclamation vous invite à vous connecter pour terminer la restauration à partir de la sauvegarde. Après avoir sélectionné l’icône appropriée, vous êtes invité à vous reconnecter pour mettre à jour vos informations.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez sauvegardé et récupéré les informations d’identification de votre compte sur votre nouvel appareil, vous pouvez continuer d’utiliser l’application Microsoft Authenticator pour vérifier votre identité. Pour plus d’informations, consultez [Vous connecter à vos comptes à l’aide de l’application Microsoft Authenticator](user-help-sign-in.md).

## <a name="related-articles"></a>Articles connexes

- [Qu’est-ce que l’application Microsoft Authenticator ?](user-help-auth-app-overview.md)

- [Forum aux questions sur l’application Microsoft Authenticator](user-help-auth-app-faq.md)

- [Azure Multi-Factor Authentication](/azure/multi-factor-authentication/)