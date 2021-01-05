---
title: Problèmes courants avec l’authentification à 2 facteurs de compte – Azure AD
description: Solutions pour certains des problèmes de vérification en deux étapes les plus courants et pour votre compte professionnel ou scolaire.
services: active-directory
author: curtand
manager: daveba
ms.assetid: 8f3aef42-7f66-4656-a7cd-d25a971cb9eb
ms.workload: identity
ms.service: active-directory
ms.subservice: user-help
ms.topic: end-user-help
ms.date: 09/01/2020
ms.author: curtand
ms.reviewer: kexia
ms.custom: contperf-fy21q1
ms.openlocfilehash: caa2c66d66cce09f5b5b0fe0948ab924389d756e
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97033780"
---
# <a name="common-problems-with-two-factor-verification-and-your-work-or-school-account"></a>Problèmes courants avec la vérification en deux étapes et votre compte professionnel ou scolaire

Certains problèmes courants de vérification à deux facteurs semblent se produire plus fréquemment que nous le souhaiterions. Nous avons rédigé cet article pour décrire les correctifs pour les problèmes les plus courants.

Votre organisation Azure Active Directory (Azure AD) peut activer la vérification à deux facteurs pour votre compte. Lorsque la vérification à deux facteurs est activée, la connexion de votre compte nécessite une combinaison des données suivantes :

- Votre nom d’utilisateur
- Votre mot de passe
- Un appareil mobile ou un téléphone

La vérification à deux facteurs est plus sécurisée qu’un simple mot de passe, car elle requiert un élément que vous _connaissez_ et un élément que vous _détenez_. Aucun pirate n’a votre téléphone physique.

>[!Important]
>Si vous êtes un administrateur, vous trouverez plus d’informations sur la méthode à suivre pour configurer et gérer votre environnement Azure AD dans la [documentation relative à Azure AD](../index.yml).

Ce contenu peut vous aider avec le compte professionnel ou scolaire que votre organisation vous a fourni (par exemple, dritan@contoso.com). Si vous rencontrez des problèmes avec la vérification en deux étapes sur un compte Microsoft personnel, compte que vous avez configuré pour vous-même (par exemple, danielle@outlook.com), consultez [Activation ou désactivation de la vérification en deux étapes de votre compte Microsoft](https://support.microsoft.com/help/4028586/microsoft-account-turning-two-step-verification-on-or-off).

## <a name="i-dont-have-my-mobile-device-with-me"></a>Je n’ai pas mon appareil mobile avec moi

Cela peut arriver. Vous avez laissé votre appareil mobile chez vous, et maintenant vous ne pouvez pas utiliser votre téléphone pour confirmer votre identité. Vous avez peut-être déjà ajouté une autre méthode pour vous connecter à votre compte, par exemple par le biais de votre téléphone professionnel. Le cas échéant, vous pouvez utiliser cette méthode alternative maintenant. Si vous n’avez jamais ajouté une autre méthode de vérification, vous pouvez contacter le support technique de votre organisation pour obtenir de l’aide.

### <a name="to-sign-in-to-your-work-or-school-account-using-another-verification-method"></a>Pour vous connecter à votre compte professionnel ou scolaire à l’aide d’une autre méthode de vérification

1. Connectez-vous à votre compte, mais sélectionnez le lien **Se connecter autrement** dans la page **Vérification en deux étapes**.

    ![Changer de méthode de vérification](./media/multi-factor-authentication-end-user-troubleshoot/two-factor-auth-signin-another-way.png)

    Si vous ne voyez pas le lien **Sign in another way** (Se connecter autrement), cela signifie que vous n’avez pas configuré d’autres méthodes de vérification. Dans ce cas, vous devez contacter votre administrateur pour qu’il vous aide à vous connecter à votre compte.

2. Choisissez votre autre méthode de vérification et poursuivez le processus de vérification à deux facteurs.

## <a name="i-cant-turn-two-factor-verification-off"></a>Je ne peux pas désactiver la vérification en deux étapes

- En revanche, si vous utilisez la vérification à deux facteurs avec un compte personnel pour un service Microsoft, par exemple, alain@outlook.com, vous avez la possibilité [d’activer et de désactiver la fonctionnalité](https://account.live.com/proofs/Manage).

- Si vous utilisez la vérification à deux facteurs avec votre compte professionnel ou scolaire, cela signifie très probablement que votre organisation a décidé de vous contraindre à utiliser cette fonctionnalité de sécurité supplémentaire. Il n’existe aucun moyen de la désactiver individuellement.

Si vous ne parvenez pas désactiver la vérification à deux facteurs, cela peut également être dû aux paramètres de sécurité par défaut qui ont été appliqués au niveau de l'organisation. Pour plus d’informations sur les paramètres de sécurité par défaut, consultez [Présentation des paramètres de sécurité par défaut](../fundamentals/concept-fundamentals-security-defaults.md).

## <a name="my-device-was-lost-or-stolen"></a>Mon appareil a été perdu ou volé

En cas de perte ou de vol de votre appareil mobile, vous pouvez effectuer l’une des actions suivantes :

- Connectez-vous à l’aide d’une autre méthode.
- Demandez au support technique de votre organisation d’effacer vos paramètres.

Nous vous recommandons vivement d’informer le support technique de votre organisation en cas de perte ou de vol de votre téléphone. Le support technique peut apporter les mises à jour appropriées à votre compte. Une fois vos paramètres effacés, la prochaine fois que vous vous connecterez, vous serez invité à vous [réinscrire à la vérification à deux facteurs](multi-factor-authentication-end-user-first-time.md).

## <a name="im-not-receiving-the-verification-code-sent-to-my-mobile-device"></a>Je ne reçois pas le code de vérification envoyé à mon appareil mobile

Le fait de ne pas recevoir votre code de vérification est un problème courant. Le problème est généralement lié à votre appareil mobile et à ses paramètres. Voici quelques actions que vous pouvez essayer.

Procédure | Conseils
--------- | ------------
Utiliser l'application Microsoft Authenticator ou des codes de vérification | Vous recevez le message d'erreur « Vous avez atteint notre limite pour les appels de vérification » ou « Vous avez atteint notre limite pour les codes de vérification par SMS » lors de la connexion. <br/><br/>Microsoft peut limiter les tentatives d'authentification répétées effectuées par le même utilisateur sur une courte période. Cette limitation ne s'applique pas à Microsoft Authenticator ou au code de vérification. Si vous avez atteint ces limites, vous pouvez utiliser l'application Authenticator, le code de vérification ou essayer de vous reconnecter quelques minutes plus tard.
Redémarrer votre appareil mobile | Parfois, votre appareil a simplement besoin d’une actualisation. Lorsque vous redémarrez votre appareil, tous les services et processus en arrière-plan sont terminés. Le redémarrage arrête également les composants principaux de votre appareil. Tout service ou composant est actualisé lorsque vous redémarrez votre appareil.
Vérifier que vos informations de sécurité sont correctes | Vérifiez que les informations relatives à votre méthode de vérification de sécurité sont exactes, en particulier vos numéros de téléphone. Si vous avez entré un numéro de téléphone incorrect, toutes vos alertes seront dirigées vers ce numéro incorrect. Heureusement, cet utilisateur ne pourra rien faire avec les alertes, mais cela ne vous aidera pas non plus à vous connecter à votre compte. Pour vérifier que vos informations sont correctes, consultez les instructions fournies dans l’article [Gérer les paramètres de la vérification à deux facteurs](multi-factor-authentication-end-user-manage-settings.md).
Vérifier que les notifications sont activées | Vérifiez que les notifications sont activées sur votre appareil mobile. Assurez-vous que les modes de notification suivants sont autorisés : <br/><br/> &bull; Appels téléphoniques <br/> &bull; Votre application d’authentification <br/> &bull; Votre application de messagerie texte <br/><br/> Assurez-vous que ces modes créent une alerte qui est _visible_ sur votre appareil.
Vérifier que votre appareil capte un signal et dispose d’une connexion Internet | Vérifiez que vos appels téléphoniques et vos SMS parviennent à votre appareil mobile. Demandez à un ami de vous appeler et de vous envoyer un SMS pour vérifier que vous les recevez bien. Si vous ne recevez aucun appel ou SMS, commencez par vérifier que votre appareil mobile est allumé. Si votre appareil est allumé, mais que vous ne recevez toujours pas d’appel ou de SMS, il y a probablement un problème avec votre réseau. Contactez votre fournisseur. Si vous rencontrez souvent des problèmes liés aux signaux, nous vous recommandons d’installer et d’utiliser l’[application Microsoft Authenticator](user-help-auth-app-download-install.md) sur votre appareil mobile. L’application Authenticator peut générer des codes de sécurité aléatoires pour la connexion, sans nécessiter de signal de réseau mobile ou de connexion Internet.
Désactiver la fonctionnalité Ne pas déranger | Vérifiez que vous n’avez pas activé la fonctionnalité **Ne pas déranger** sur votre appareil mobile. Quand elle est activée, les notifications ne sont pas autorisées à vous alerter sur votre appareil mobile. Pour obtenir des instructions sur la façon de désactiver cette fonctionnalité, consultez le manuel de votre appareil mobile.
Débloquer les numéros de téléphone | Aux États-Unis, les appels vocaux de Microsoft proviennent des numéros suivants : +1 (866) 539 4191, +1 (855) 330 8653 et +1 (877) 668 6536.
Vérifier les paramètres liés à la batterie | Si vous avez configuré l’optimisation de la batterie de façon à empêcher les applications les moins utilisées de rester actives en arrière-plan, cela a probablement affecté votre système de notification. Essayez de désactiver l’optimisation de la batterie pour votre application d’authentification et votre application de messagerie. Essayez ensuite de vous reconnecter à votre compte.
Désactiver les applications de sécurité tierces | Certaines applications de sécurité du téléphone bloquent les SMS et les appels téléphoniques des appelants inconnus. Une application de sécurité peut empêcher votre téléphone de recevoir le code de vérification. Essayez de désactiver les applications de sécurité tierces sur votre téléphone, puis demandez l'envoi d'un autre code de vérification.

## <a name="im-not-being-prompted-for-my-second-verification-information"></a>Je ne suis pas invité à entrer ma deuxième information de vérification

Vous vous connectez à votre compte professionnel ou scolaire à l’aide de vos nom d’utilisateur et mot de passe. Ensuite, vous devriez être invité à fournir vos informations de vérification de sécurité supplémentaires. Si vous ne recevez aucune invitation, c’est peut-être parce que vous n’avez pas encore configuré votre appareil. Votre appareil mobile doit être configuré pour fonctionner avec votre méthode spécifique de vérification de sécurité supplémentaire.

Vous n’avez peut-être pas encore configuré votre appareil. Votre appareil mobile doit être configuré pour fonctionner avec votre méthode spécifique de vérification de la sécurité. Pour savoir comment rendre votre appareil mobile utilisable avec votre méthode de vérification, consultez [Gérer les paramètres de la vérification à deux facteurs](multi-factor-authentication-end-user-manage-settings.md). Si vous savez que vous n’avez pas configuré votre appareil ou votre compte, vous pouvez suivre les étapes décrites dans l’article [Configurer mon compte pour la vérification en deux étapes](multi-factor-authentication-end-user-first-time.md).

## <a name="i-have-a-new-phone-number-and-i-want-to-add-it"></a>J’ai un nouveau numéro de téléphone et je souhaite l’ajouter

Si vous avez un nouveau numéro de téléphone, vous devez mettre à jour les détails de votre méthode de vérification de sécurité. Cela permet à vos invites de vérification d’accéder à l’emplacement approprié. Pour mettre à jour votre méthode de vérification, suivez les étapes décrites dans la section **Ajouter ou modifier votre numéro de téléphone** de l’article [Gérer les paramètres de la vérification à deux facteurs](multi-factor-authentication-end-user-manage-settings.md#add-or-change-your-phone-number).

## <a name="i-have-a-new-mobile-device-and-i-want-to-add-it"></a>J’ai un nouvel appareil mobile et je souhaite l’ajouter

Si vous avez un nouvel appareil mobile, vous devez le configurer pour qu’il fonctionne avec la vérification à deux facteurs. Cette solution comprend plusieurs étapes :

1. Configurez votre appareil pour qu’il fonctionne avec votre compte en suivant les étapes décrites dans l’article [Configurer mon compte pour la vérification à deux facteurs](multi-factor-authentication-end-user-first-time.md).

1. Mettez à jour les informations de votre compte et de votre appareil dans la page **Vérification de sécurité supplémentaire**. Effectuez la mise à jour en supprimant votre ancien appareil et en ajoutant votre nouvel appareil. Pour plus d’informations, consultez l’article [Gérer les paramètres de la vérification à deux facteurs](multi-factor-authentication-end-user-manage-settings.md).

Étapes facultatives :

- Installez l’application Microsoft Authenticator sur votre appareil mobile en procédant de la manière décrite dans l’article [Télécharger et installer l’application Microsoft Authenticator](user-help-auth-app-download-install.md).

- Activez la vérification à deux facteurs pour vos appareils de confiance en suivant les étapes de la section **Activer les invites de vérification à deux facteurs sur un appareil de confiance** de l’article [Gérer les paramètres de la vérification à deux facteurs](multi-factor-authentication-end-user-manage-settings.md#turn-on-two-factor-verification-prompts-on-a-trusted-device).

## <a name="im-having-problems-signing-in-on-my-mobile-device-while-traveling"></a>J’ai des difficultés à me connecter à mon appareil mobile quand je suis en déplacement

Il peut s’avérer plus difficile d’utiliser une méthode de vérification liée à l’appareil mobile, comme un SMS, quand vous êtes dans un autre pays. Il est également possible que des frais d’itinérance vous soient facturés. Dans ce cas, nous vous recommandons d’utiliser l’application Microsoft Authenticator, avec l’option de connexion à un point d’accès Wi-Fi. Pour plus d’informations sur la façon de configurer l’application Microsoft Authenticator sur votre appareil mobile, consultez l’article [Télécharger et installer l’application Microsoft Authenticator](user-help-auth-app-download-install.md).

## <a name="i-cant-get-my-app-passwords-to-work"></a>Mes mots de passe d’application ne fonctionnent pas

Les mots de passe d’application viennent remplacer les mots de passe standard de vos anciennes applications de bureau qui ne prennent pas en charge la vérification à deux facteurs. En premier lieu, vérifiez que vous avez correctement tapé le mot de passe. Si cela ne résout pas le problème, essayez de créer un nouveau mot de passe d’application pour l’application. Pour ce faire, suivez les étapes décrites dans la section **Créer et supprimer des mots de passe d’application à l’aide du portail MyApps** de l’article [Gérer les mots de passe d’application pour la vérification en deux étapes](multi-factor-authentication-end-user-app-passwords.md#create-and-delete-app-passwords-from-the-additional-security-verification-page).

## <a name="i-cant-turn-off-two-factor-verification"></a>Je ne peux pas activer la vérification à deux factors

Si vous utilisez la vérification à deux facteurs avec votre compte professionnel ou scolaire (par exemple, alain@contoso.com), cela signifie très probablement que votre organisation a décidé de vous contraindre à utiliser cette fonctionnalité de sécurité supplémentaire. Comme votre organisation a décidé que cette fonctionnalité était obligatoire, vous n’avez aucun moyen de la désactiver individuellement. En revanche, si vous utilisez la vérification à deux facteurs avec un compte personnel, comme alain@outlook.com, vous avez la possibilité d’activer et de désactiver la fonctionnalité. Pour obtenir des instructions sur la façon de contrôler la vérification en deux étapes pour vos comptes personnels, consultez [Activation ou désactivation de la vérification en deux étapes de votre compte Microsoft](https://support.microsoft.com/help/4028586/microsoft-account-turning-two-step-verification-on-or-off).

Si vous ne parvenez pas désactiver la vérification à deux facteurs, cela peut également être dû aux paramètres de sécurité par défaut qui ont été appliqués au niveau de l'organisation. Pour plus d’informations sur les paramètres de sécurité par défaut, consultez [Présentation des paramètres de sécurité par défaut](../fundamentals/concept-fundamentals-security-defaults.md).

## <a name="i-didnt-find-an-answer-to-my-problem"></a>Je n’ai pas trouvé de réponse à mon problème

Si, malgré ces solutions, vous rencontrez encore des problèmes, contactez le support technique de votre organisation pour obtenir de l’aide.

## <a name="related-articles"></a>Articles connexes

- [Gérer les paramètres de la vérification à deux facteurs](multi-factor-authentication-end-user-manage-settings.md)

- [Configurer mon compte pour la vérification en deux étapes](multi-factor-authentication-end-user-first-time.md)

- [Forum aux questions sur l’application Microsoft Authenticator](user-help-auth-app-faq.md)