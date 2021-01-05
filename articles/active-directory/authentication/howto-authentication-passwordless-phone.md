---
title: Connexion sans mot de passe avec l’application Microsoft Authenticator - Azure Active Directory
description: Activer la connexion sans mot de passe pour Azure AD à l’aide de l’application Microsoft Authenticator (préversion)
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 11/11/2020
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: librown
ms.collection: M365-identity-device-management
ms.openlocfilehash: 35eff46a0470d429c8ec6f364ffa836501c65f47
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96743596"
---
# <a name="enable-passwordless-sign-in-with-the-microsoft-authenticator-app-preview"></a>Activer la connexion sans mot de passe avec l'application Microsoft Authenticator (préversion)

L’application Microsoft Authenticator vous permet de vous connecter à n’importe quel compte Azure AD sans utiliser de mot de passe. Microsoft Authenticator utilise l’authentification par clé pour activer les informations d’identification de l’utilisateur qui sont liées à un appareil, où l’appareil utilise un code confidentiel ou la biométrie. [Windows Hello Entreprise](/windows/security/identity-protection/hello-for-business/hello-identity-verification) utilise une technologie similaire.

Cette technologie d’authentification peut être utilisée sur toutes les plateformes d’appareils, y compris sur les appareils mobiles. Cette technologie peut également être utilisée avec n’importe quelle application ou n’importe quel site web qui s’intègre aux bibliothèques d’authentification Microsoft.

:::image type="content" border="false" source="./media/howto-authentication-passwordless-phone/phone-sign-in-microsoft-authenticator-app.png" alt-text="Exemple de connexion au navigateur demandant à l’utilisateur d’approuver la connexion.":::

Les personnes qui ont activé la connexion par téléphone à partir de l’application Microsoft Authenticator voient un message leur demandant d’appuyer sur un nombre dans leur application. Aucun nom d’utilisateur ou mot de passe n’est demandé. Pour terminer le processus de connexion dans l’application, un utilisateur doit ensuite effectuer les actions suivantes :

1. Choisir le nombre correspondant.
2. Choisissez **Approuver**.
3. Fournir son code confidentiel ou sa biométrie.

## <a name="prerequisites"></a>Prérequis

Pour utiliser la connexion par téléphone sans mot de passe avec l’application Microsoft Authenticator, les prérequis suivants doivent être satisfaits :

- Azure AD Multi-Factor Authentication, avec notifications Push autorisées en tant que méthode de vérification.
- Installation de la dernière version de Microsoft Authenticator sur des appareils exécutant iOS 8.0 ou une version ultérieure, ou Android 6.0 ou une version ultérieure.

> [!NOTE]
> Si vous avez activé la préversion de la connexion sans mot de passe Microsoft Authenticator en utilisant Azure AD PowerShell, elle a été activée pour l’ensemble de votre répertoire. Si vous activez l’utilisation de cette nouvelle méthode, elle remplace la stratégie PowerShell. Nous vous recommandons d’activer cette fonctionnalité pour tous les utilisateurs de votre locataire par le biais du nouveau menu *Méthodes d’authentification*, sinon les utilisateurs ne figurant pas dans la nouvelle stratégie ne pourront plus se connecter sans mot de passe.

## <a name="enable-passwordless-authentication-methods"></a>Activer les méthodes d’authentification sans mot de passe

Pour utiliser l’authentification sans mot de passe dans Azure AD, activez tout d’abord l’expérience d’inscription combinée, puis activez les utilisateurs pour la méthode sans mot de passe.

### <a name="enable-the-combined-registration-experience"></a>Activer l’expérience d’inscription combinée

Les fonctionnalités d’inscription pour les méthodes d’authentification sans mot de passe s’appuient sur la fonctionnalité d'inscription combinée. Pour permettre aux utilisateurs d’effectuer eux-mêmes l’inscription combinée, suivez les étapes pour [activer l’inscription combinée des informations de sécurité](howto-registration-mfa-sspr-combined.md).

### <a name="enable-passwordless-phone-sign-in-authentication-methods"></a>Activer les méthodes d’authentification sans mot de passe par téléphone

Azure AD vous permet de choisir les méthodes d’authentification qui peuvent être utilisées pendant le processus de connexion. Les utilisateurs s’inscrivent ensuite pour les méthodes qu’ils souhaitent utiliser.

Pour activer la méthode d’authentification pour la connexion par téléphone sans mot de passe, effectuez les étapes suivantes :

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte d’*administrateur général*.
1. Recherchez et sélectionnez *Azure Active Directory*, puis accédez à **Sécurité** > **Méthodes d’authentification** > **Stratégie de méthode d’authentification (préversion)**
1. Sous **Connexion par téléphone sans mot de passe**, choisissez les options suivantes :
   1. **Activer** - Oui ou Non
   1. **Cible** - Tous les utilisateurs ou les utilisateurs sélectionnés
1. Pour appliquer la nouvelle stratégie, sélectionnez **Enregistrer**.

## <a name="user-registration-and-management-of-microsoft-authenticator"></a>Inscription des utilisateurs et gestion de Microsoft Authenticator

Les utilisateurs s’inscrivent à la méthode d’authentification sans mot de passe d’Azure AD en procédant comme suit :

1. Accédez à [https://aka.ms/mysecurityinfo](https://aka.ms/mysecurityinfo) .
1. Connectez-vous, puis ajoutez l’application Authenticator en sélectionnant **Ajouter une méthode > Application d’authentification**, puis **Ajouter**.
1. Suivez les instructions pour installer et configurer l’application Microsoft Authenticator sur votre appareil.
1. Sélectionnez **Terminé** pour terminer la configuration d’Authenticator.
1. Dans **Microsoft Authenticator**, choisissez **Activer la connexion par téléphone** dans le menu déroulant du compte inscrit.
1. Suivez les instructions de l’application pour terminer de vous inscrire à la connexion sans mot de passe.

Une organisation peut demander à ses utilisateurs de se connecter avec leur téléphone, sans utiliser de mot de passe. Pour plus d’informations sur la configuration de l’application Microsoft Authenticator et sur l’activation de la connexion par téléphone, consultez [Vous connecter à vos comptes à l’aide de l’application Microsoft Authenticator](../user-help/user-help-auth-app-sign-in.md).

> [!NOTE]
> Les utilisateurs que la stratégie n’autorise pas à utiliser la connexion par téléphone ne sont plus en mesure de l’activer dans l’application Microsoft Authenticator.

## <a name="sign-in-with-passwordless-credential"></a>Se connecter avec les informations d’identification sans mot de passe

Un utilisateur peut commencer à utiliser la connexion sans mot de passe une fois que toutes les actions suivantes ont été effectuées :

- Un administrateur a activé le locataire de l’utilisateur.
- L’utilisateur a mis à jour son application Microsoft Authenticator pour activer la connexion par téléphone.

La première fois qu’un utilisateur lance le processus de connexion par téléphone, il effectue les étapes suivantes :

1. Entre son nom dans la page de connexion.
2. Sélectionne **Suivant**.
3. Le cas échéant, sélectionne **Autres méthodes de connexion**.
4. Sélectionne **Approuver une demande sur mon application Microsoft Authenticator**.

Un nombre est ensuite présenté à l’utilisateur. L’application invite l’utilisateur à s’authentifier en sélectionnant le nombre approprié au lieu d’entrer un mot de passe.

Une fois que l’utilisateur a utilisé la connexion par téléphone sans mot de passe, l’application continue à le diriger par le biais de cette méthode. Toutefois, l’utilisateur verra l’option permettant de choisir une autre méthode.

:::image type="content" border="false" source="./media/howto-authentication-passwordless-phone/web-sign-in-microsoft-authenticator-app.png" alt-text="Exemple de connexion au navigateur à l’aide de l’application Microsoft Authenticator.":::

## <a name="known-issues"></a>Problèmes connus

Les problèmes connus suivants existent dans l’expérience en préversion actuelle.

### <a name="not-seeing-option-for-passwordless-phone-sign-in"></a>Aucune option pour la connexion par téléphone sans mot de passe n’est visible

Dans un scénario, un utilisateur peut avoir une vérification de connexion par téléphone sans mot de passe sans réponse qui est en attente. Pourtant, l’utilisateur peut tenter de se reconnecter. Dans ce cas, il est possible que l’utilisateur ne voie que la possibilité de saisir un mot de passe.

Pour résoudre ce scénario, vous pouvez utiliser les étapes suivantes :

1. Ouvrez l’application Microsoft Authenticator.
2. Répondez aux invites de notification.

L’utilisateur peut ensuite continuer à utiliser la connexion par téléphone sans mot de passe.

### <a name="federated-accounts"></a>Comptes fédérés

Lorsqu’un utilisateur a activé des informations d’identification sans mot de passe, le processus de connexion Azure AD cesse d’utiliser le login\_hint. Par conséquent, le processus ne dirige plus l’utilisateur vers un emplacement de connexion fédérée.

Cette logique empêche généralement l’utilisateur d’un locataire hybride d’être dirigé vers les services de fédération Active Directory (AD FS) pour la vérification de la connexion. Toutefois, l’utilisateur conserve la possibilité de cliquer sur **Utiliser votre mot de passe à la place**.

### <a name="azure-mfa-server"></a>Serveur Azure MFA

Un utilisateur final peut être en mesure d’utiliser l’authentification multifacteur (MFA) via un serveur Azure MFA local. L’utilisateur peut toujours créer et utiliser une seule paire d’informations d’identification de connexion par téléphone.

Si l’utilisateur tente de mettre à niveau plusieurs installations (supérieures à 5) de l’application Microsoft Authenticator avec ces informations d’identification de connexion par téléphone sans mot de passe, cette modification peut générer une erreur.

### <a name="device-registration"></a>Enregistrement de l’appareil

Avant de pouvoir créer des informations d’identification fortes, vous devez remplir les conditions préalables. L’une des conditions requises est que l’appareil sur lequel l’application Microsoft Authenticator est installée doit être inscrite dans le locataire Azure AD pour un utilisateur individuel.

Actuellement, un appareil ne peut être inscrit que dans un seul locataire. Cette limite signifie qu’un seul compte professionnel ou scolaire de l’application Microsoft Authenticator peut être activé pour la connexion par téléphone.

> [!NOTE]
> L’inscription de l’appareil n’est pas la même que la gestion des appareils ou la gestion des périphériques mobiles (GPM). Elle associe uniquement un ID d’appareil et un identifiant utilisateur dans le répertoire Azure AD.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur l’authentification Azure AD et les méthodes sans mot de passe, consultez les articles suivants :

- [En savoir plus sur le fonctionnement de l’authentification par mot de passe](concept-authentication-passwordless.md)
- [En savoir plus sur l’inscription des appareils](../devices/overview.md#getting-devices-in-azure-ad)
- [En savoir plus sur Azure AD Multi-Factor Authentication](../authentication/howto-mfa-getstarted.md)
