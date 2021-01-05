---
title: Méthode d’authentification d’application Microsoft Authenticator - Azure Active Directory
description: En savoir plus sur l’utilisation de l’application Microsoft Authenticator dans Azure Active Directory pour contribuer à l’amélioration et à la sécurisation des événements de connexion
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 10/09/2020
ms.author: justinha
author: justinha
manager: daveba
ms.collection: M365-identity-device-management
ms.openlocfilehash: d7309d9d32eba5cae350bbe8a1df325dfb2ab0c8
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96744361"
---
# <a name="authentication-methods-in-azure-active-directory---microsoft-authenticator-app"></a>Méthodes d’authentification dans Azure Active Directory - Application Microsoft Authenticator

L’application Microsoft Authenticator fournit un niveau de sécurité supplémentaire pour votre compte Azure AD professionnel ou scolaire, ou pour votre compte Microsoft. Elle est disponible pour [Android](https://go.microsoft.com/fwlink/?linkid=866594) et [iOS](https://go.microsoft.com/fwlink/?linkid=866594). Avec l'application Microsoft Authenticator, les utilisateurs peuvent s'authentifier sans mot de passe lors de la connexion, ou avec une option de vérification supplémentaire lors de la réinitialisation du mot de passe en libre-service (SSPR) ou d'événements Azure AD Multi-Factor Authentication.

Les utilisateurs peuvent recevoir une notification par le biais de l’application mobile leur permettant d’accepter ou de refuser. Ils peuvent également utiliser l’application Authenticator pour générer un code de vérification OATH pouvant être entré dans une interface de connexion. Si vous activez à la fois la notification et le code de vérification, les utilisateurs s’enregistrant sur l’application Authenticator peuvent utiliser une des deux méthodes pour vérifier leur identité.

Pour utiliser l’application Authenticator dans une invite de connexion au lieu de la combinaison nom d’utilisateur/mot de passe, consultez [Activer la connexion sans mot de passe avec l'application Microsoft Authenticator (préversion)](howto-authentication-passwordless-phone.md).

> [!NOTE]
> Les utilisateurs n’ont pas la possibilité d’enregistrer leur application mobile lorsqu’ils activent la réinitialisation de mot de passe en libre-service. En revanche, les utilisateurs peuvent inscrire leur application mobile sur [https://aka.ms/mfasetup](https://aka.ms/mfasetup) ou dans le cadre de l’inscription d’informations de sécurité combinée sur [https://aka.ms/setupsecurityinfo](https://aka.ms/setupsecurityinfo).

## <a name="passwordless-sign-in"></a>Connexion sans mot de passe

Au lieu d’obtenir une invite de mot de passe après avoir entré un nom d’utilisateur, un utilisateur qui a activé la connexion par téléphone dans l’application Microsoft Authenticator voit s’afficher un message lui demandant d’entrer un nombre dans son application. Lorsque le nombre correct est sélectionné, le processus de connexion est terminé.

![Exemple de connexion dans un navigateur demandant à l’utilisateur d’approuver la connexion](./media/howto-authentication-passwordless-phone/phone-sign-in-microsoft-authenticator-app.png)

Cette méthode d’authentification offre un niveau élevé de sécurité et évite à l’utilisateur de fournir un mot de passe lors de la connexion. La connexion avec le mot de passe à l’aide de l’application Microsoft Authenticator est actuellement disponible en préversion.

Pour commencer à utiliser la connexion sans mot de passe, consultez [Activer la connexion sans mot de passe avec l'application Microsoft Authenticator](howto-authentication-passwordless-phone.md).

## <a name="notification-through-mobile-app"></a>Notification via une application mobile

L’application Authenticator peut aider à empêcher tout accès non autorisé aux comptes et à arrêter les transactions frauduleuses en envoyant une notification à votre smartphone ou tablette. Les utilisateurs voient la notification et, si elle est légitime, sélectionnent **Vérifier**. Sinon, ils peuvent sélectionner **Refuser**.

![Capture d’écran d’exemple de l’invite du navigateur Web affichant la notification de l’application Authenticator permettant de terminer le processus de connexion](media/tutorial-enable-azure-mfa/azure-multi-factor-authentication-browser-prompt.png)

> [!NOTE]
> Si votre organisation dispose de personnel travaillant ou voyageant en Chine, la méthode *Notification via une application mobile* sur les appareils Android ne fonctionne pas dans ce pays ou cette région, car les services Google Play (y compris les notifications Push) sont bloqués dans la région. En revanche, la notification iOS fonctionne. Pour Android, d’autres méthodes d’authentification doivent être proposées à ces utilisateurs.

## <a name="verification-code-from-mobile-app"></a>Code de vérification de l’application mobile

L'application Authenticator peut être utilisée comme jeton logiciel pour générer un code de vérification OATH. Après avoir saisi votre nom d’utilisateur et votre mot de passe, vous entrez le code fourni par l’application Authenticator dans l’interface de connexion. Le code de vérification fournit un deuxième formulaire d’authentification.

Les utilisateurs peuvent combiner jusqu’à cinq jetons matériels OATH ou des applications d’authentification, comme l’application Microsoft Authenticator, configurées pour une utilisation à tout moment.

> [!WARNING]
> Pour garantir le niveau de sécurité le plus élevé quand une seule méthode est requise pour la réinitialisation de mot de passe en libre-service, le code de vérification est la seule option à la disposition des utilisateurs.
>
> Lorsque deux méthodes sont requises, les utilisateurs peuvent effectuer la réinitialisation avec soit une notification, soit un code de vérification en plus de n’importe quelle autre méthode activée.

## <a name="next-steps"></a>Étapes suivantes

Pour commencer à utiliser la connexion sans mot de passe, consultez [Activer la connexion sans mot de passe avec l'application Microsoft Authenticator](howto-authentication-passwordless-phone.md).

Apprenez-en plus sur la configuration des méthodes d’authentification à l’aide de l’[API REST Microsoft Graph bêta](/graph/api/resources/authenticationmethods-overview?view=graph-rest-beta).
