---
title: 'Méthodes d’authentification par téléphone : Azure Active Directory'
description: En savoir plus sur l’utilisation des méthodes d'authentification par téléphone dans Azure Active Directory pour contribuer à l’amélioration et à la sécurisation des événements de connexion
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 11/18/2020
ms.author: justinha
author: justinha
manager: daveba
ms.collection: M365-identity-device-management
ms.openlocfilehash: c9d3a00491bc0628244a5a7907f0dee03a5f8d3c
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96744191"
---
# <a name="authentication-methods-in-azure-active-directory---phone-options"></a>Méthodes d’authentification dans Azure Active Directory - Options téléphoniques

Pour une authentification directe à l’aide d’un message texte, vous pouvez [Configurer et activer des utilisateurs pour l’authentification par SMS (préversion)](howto-authentication-sms-signin.md). La connexion par SMS est idéale pour les travailleurs de première ligne. Avec la connexion par SMS, les utilisateurs n’ont pas besoin de connaître le nom d’utilisateur et le mot de passe pour accéder aux applications et aux services. L’utilisateur entre son numéro de téléphone mobile enregistré, reçoit un SMS contenant un code de vérification et l’entre dans l’interface de connexion.

Les utilisateurs peuvent également s'authentifier en utilisant un téléphone mobile ou un téléphone de bureau comme forme secondaire d'authentification lors de l’authentification Azure AD MFA ou de la réinitialisation de mot de passe en libre-service (SSPR).

Pour que tout fonctionne correctement, les numéros de téléphone doivent être au format *+CodePays NuméroTéléphone*, par exemple : *+1 4251234567*.

> [!NOTE]
> Il doit y avoir un espace entre l’indicatif du pays et le numéro de téléphone.
>
> La réinitialisation du mot de passe ne prend pas en charge les extensions de téléphone. Même au format *+1 4251234567X12345*, les extensions sont supprimées avant l’appel.

## <a name="mobile-phone-verification"></a>Vérification du téléphone mobile

Pour Azure AD MFA ou SSPR, les utilisateurs peuvent choisir de recevoir un SMS contenant un code de vérification à entrer dans l'interface de connexion ou de recevoir un appel téléphonique.

Si les utilisateurs ne souhaitent pas que leur numéro de téléphone mobile soit visible dans le répertoire, tout en ayant la possibilité de l’utiliser pour la réinitialisation du mot de passe, les administrateurs ne doivent pas renseigner le numéro de téléphone dans le répertoire. Au lieu de cela, les utilisateurs doivent renseigner leur attribut de **téléphone d’authentification** via l’inscription d’informations de sécurité combinée sur [https://aka.ms/setupsecurityinfo](https://aka.ms/setupsecurityinfo). Les administrateurs peuvent consulter ces informations dans le profil de l’utilisateur, mais elles ne sont pas publiées ailleurs.

:::image type="content" source="media/concept-authentication-methods/user-authentication-methods.png" alt-text="Capture d’écran du Portail Azure affichant les méthodes d’authentification avec un numéro de téléphone renseigné":::

Microsoft ne garantit pas que les notifications Azure AD Multi-Factor Authentication envoyées par SMS ou message vocal proviennent systématiquement du même numéro. Dans l’intérêt de nos utilisateurs, nous pouvons à tout moment ajouter ou supprimer des codes courts pour refléter les ajustements apportés aux itinéraires et améliorer la remise des SMS. Microsoft ne prend pas en charge les codes courts pour les pays/régions en dehors des États-Unis et du Canada.

### <a name="text-message-verification"></a>Vérification des SMS

Avec la vérification par SMS dans le cadre de l'authentification Azure AD MFA et de la réinitialisation SSPR, un SMS contenant un code de vérification est envoyé au numéro de téléphone mobile. Pour terminer le processus de connexion, le code de vérification fourni est entré dans l’interface de connexion.

### <a name="phone-call-verification"></a>Vérification de l’appel téléphonique

Avec la vérification par appel téléphonique dans le cadre de l'authentification Azure AD MFA et de la réinitialisation SSPR, un appel vocal automatique est passé au numéro de téléphone enregistré par l'utilisateur. Pour terminer le processus de connexion, l’utilisateur est invité à saisir le symbole # à l’aide du clavier.

## <a name="office-phone-verification"></a>Vérification du téléphone de bureau

Avec la vérification par appel téléphonique dans le cadre de l'authentification Azure AD MFA et de la réinitialisation SSPR, un appel vocal automatique est passé au numéro de téléphone enregistré par l'utilisateur. Pour terminer le processus de connexion, l’utilisateur est invité à saisir le symbole # à l’aide du clavier.

## <a name="troubleshooting-phone-options"></a>Résolution des problèmes liés aux options du téléphone

Si vous rencontrez des problèmes avec l’authentification par téléphone pour Azure AD, passez en revue les étapes de dépannage suivantes :

* Message d'erreur « Vous avez atteint notre limite pour les appels de vérification » ou « Vous avez atteint notre limite pour les codes de vérification par SMS » lors de la connexion
   * Microsoft peut limiter les tentatives d'authentification répétées effectuées par le même utilisateur sur une courte période. Cette limitation ne s'applique pas à Microsoft Authenticator ou au code de vérification. Si vous avez atteint ces limites, vous pouvez utiliser l'application Authenticator, le code de vérification ou essayer de vous reconnecter quelques minutes plus tard.
* ID de l’appelant bloqué sur un seul appareil.
   * Examinez les numéros bloqués configurés sur l’appareil.
* Numéro de téléphone erroné ou code de pays/région incorrect, ou confusion entre le numéro de téléphone personnel et le numéro de téléphone professionnel.
   * Résolvez les problèmes liés à l’objet utilisateur et aux méthodes d’authentification. Assurez-vous que les numéros de téléphone appropriés sont enregistrés.
* Code PIN entré incorrect.
   * Assurez-vous que l’utilisateur a utilisé le code PIN approprié comme enregistré pour son compte (utilisateurs de MFA Server uniquement).
* Appel transféré vers la messagerie vocale.
   * Vérifiez que le téléphone de l'utilisateur est allumé et que le service est disponible dans sa zone ou utilisez une autre méthode.
* L’utilisateur est bloqué
   * Demander à un administrateur Azure AD de débloquer l’utilisateur dans le Portail Azure.
* L’option SMS n’est pas inscrite sur l’appareil.
   * Demandez à l’utilisateur de changer de méthode ou d’activer les SMS sur l’appareil.
* Fournisseurs de télécommunications en faute, par exemple : aucune entrée téléphone détectée, problèmes de tonalités DTMF manquantes, ID de l’appelant bloqué sur plusieurs appareils, ou SMS bloqués sur plusieurs appareils.
   * Microsoft utilise plusieurs fournisseurs de télécommunications pour acheminer les appels téléphoniques et les messages SMS pour l’authentification. Si vous rencontrez des problèmes parmi ceux mentionnés ci-dessus, demandez à un utilisateur d’essayer d’utiliser la méthode au moins cinq fois en 5 minutes, et soyez en possession de ses informations lorsque vous contactez le support Microsoft.

## <a name="next-steps"></a>Étapes suivantes

Pour commencer, consultez le [tutoriel sur la réinitialisation du mot de passe en libre-service][tutorial-sspr] et l’[authentification multifacteur Azure AD][tutorial-azure-mfa].

Pour en savoir plus sur les concepts de SSPR, consultez [Fonctionnement de la réinitialisation de mot de passe en libre-service dans Azure AD][concept-sspr].

Pour plus d'informations sur les concepts MFA, consultez [Fonctionnement d'Azure AD Multi-Factor Authentication][concept-mfa].

Apprenez-en plus sur la configuration des méthodes d’authentification à l’aide de l’[API REST Microsoft Graph bêta](/graph/api/resources/authenticationmethods-overview?view=graph-rest-beta&preserve-view=true).

<!-- INTERNAL LINKS -->
[tutorial-sspr]: tutorial-enable-sspr.md
[tutorial-azure-mfa]: tutorial-enable-azure-mfa.md
[concept-sspr]: concept-sspr-howitworks.md
[concept-mfa]: concept-mfa-howitworks.md
