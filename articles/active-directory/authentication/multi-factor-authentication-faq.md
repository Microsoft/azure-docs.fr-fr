---
title: FAQ sur Microsoft Azure AD Multi-Factor Authentication - Azure Active Directory
description: Questions/Réponses relatives à Azure AD Multi-Factor Authentication.
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 07/14/2020
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1dfe99a2ed57ecc953c69d1bad89e2b2c8816ca3
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96741444"
---
# <a name="frequently-asked-questions-about-azure-ad-multi-factor-authentication"></a>Forum aux questions sur Azure AD Multi-Factor Authentication

Ce forum aux questions répond aux questions courantes sur Azure AD Multi-Factor Authentication et sur l'utilisation du service Multi-Factor Authentication. Il est divisé en questions relatives au service en général, aux modèles de facturation, aux expériences utilisateur et au dépannage.

> [!IMPORTANT]
> À compter du 1er juillet 2019, Microsoft ne propose plus MFA Server pour les nouveaux déploiements. Les nouveaux clients qui souhaitent imposer une authentification multifacteur à leurs utilisateurs doivent utiliser la fonctionnalité Azure AD Multi-Factor Authentication basée sur le cloud. Les clients existants qui ont activé MFA Server avant le 1er juillet peuvent télécharger la dernière version et les futures mises à jour, et générer des informations d’identification d’activation comme d’habitude.
>
> Les informations partagées ci-dessous concernant le serveur Azure Multi-Factor Authentication s’appliquent uniquement aux utilisateurs qui ont déjà le serveur MFA en cours d’exécution.
>
> La gestion des licences basée sur la consommation n’est plus disponible pour les nouveaux clients depuis le 1er septembre 2018.
> Depuis le 1er septembre 2018, il n’est plus possible de créer des fournisseurs d’authentification. Il restera possible d’utiliser et de mettre à jour des fournisseurs d’authentification existants. L’authentification multifacteur restera une fonctionnalité disponible dans les licences Azure AD Premium.

## <a name="general"></a>Général

* [Comment le serveur Azure Multi-Factor Authentication gère-t-il les données utilisateur ?](#how-does-azure-multi-factor-authentication-server-handle-user-data)
* [Quels sont les codes courts SMS utilisés pour envoyer des SMS à mes utilisateurs ?](#what-sms-short-codes-are-used-for-sending-sms-messages-to-my-users)

### <a name="how-does-azure-multi-factor-authentication-server-handle-user-data"></a>Comment le serveur Azure Multi-Factor Authentication gère-t-il les données utilisateur ?

Avec le serveur Multi-Factor Authentication, les données utilisateur sont uniquement stockées sur les serveurs locaux. Aucune donnée utilisateur persistante n'est stockée dans le cloud. Lorsque l'utilisateur effectue une vérification en deux étapes, le serveur Multi-Factor Authentication envoie les données au service cloud Azure AD Multi-Factor Authentication à des fins d'authentification. La communication entre le serveur Multi-Factor Authentication et le service cloud Multi-Factor Authentication utilise le protocole SSL (Secure Sockets Layer) ou TLS (Transport Layer Security) sur le port 443 sortant.

Lorsque les demandes d’authentification sont envoyées au service cloud, les données collectées pour les rapports d’utilisation et d’authentification. Les champs de données suivants sont inclus dans les journaux d’activité de vérification en deux étapes :

* **ID unique** (soit le nom d’utilisateur soit l’ID de serveur Multi-Factor Authentication local)
* **Prénom et nom** (facultatif)
* **Adresse de messagerie** (facultatif)
* **Numéro de téléphone** (en cas d’utilisation de l’authentification par appel vocal ou SMS)
* **Jeton de l’appareil** (en cas d’utilisation de l’authentification par application mobile)
* **Mode d'authentification**
* **Résultat de l'authentification**
* **Nom du serveur Multi-Factor Authentication**
* **Adresse IP du serveur Multi-Factor Authentication**
* **Adresse IP du client** (si disponible)

Les champs facultatifs peuvent être configurés dans le serveur Multi-Factor Authentication.

Le résultat de la vérification (réussite ou échec) et le motif de refus sont stockés avec les données d’authentification. Celles-ci sont disponibles dans les rapports d’utilisation et d’authentification.

Pour plus d'informations, consultez [Résidence des données et données client pour Azure AD Multi-Factor Authentication](concept-mfa-data-residency.md).

### <a name="what-sms-short-codes-are-used-for-sending-sms-messages-to-my-users"></a>Quels sont les codes courts SMS utilisés pour envoyer des SMS à mes utilisateurs ?

Aux États-Unis, nous utilisons les codes courts SMS suivants :

* *97671*
* *69829*
* *51789*
* *99399*

Au Canada, nous utilisons les codes courts SMS suivants :

* *759731*
* *673801*

Il n'est pas garanti que les notifications Multi-Factor Authentication envoyées par SMS ou message vocal proviennent systématiquement du même numéro. Dans l’intérêt de nos utilisateurs, nous pouvons à tout moment ajouter ou supprimer des codes courts pour refléter les ajustements apportés aux itinéraires et améliorer la remise des SMS.

Nous ne prenons pas en charge les codes courts pour les pays/régions en dehors des États-Unis et du Canada.

## <a name="billing"></a>Facturation

Les réponses à la plupart des questions de facturation sont disponibles sur la [page Tarification de Multi-Factor Authentication](https://azure.microsoft.com/pricing/details/multi-factor-authentication/) ou dans la documentation consacrée aux [Versions et plans de consommation d'Azure AD Multi-Factor Authentication](concept-mfa-licensing.md).

* [L’envoi des appels téléphoniques et des SMS utilisés pour l’authentification est-il facturé à mon organisation ?](#is-my-organization-charged-for-sending-the-phone-calls-and-text-messages-that-are-used-for-authentication)
* [Le modèle de facturation par utilisateur me facture-t-il tous les utilisateurs activés ou seulement ceux ayant effectué la vérification en deux étapes ?](#does-the-per-user-billing-model-charge-me-for-all-enabled-users-or-just-the-ones-that-performed-two-step-verification)
* [Comment fonctionne la facturation Multi-Factor Authentication ?](#how-does-multi-factor-authentication-billing-work)
* [Existe-t-il une version gratuite d'Azure AD Multi-Factor Authentication ?](#is-there-a-free-version-of-azure-ad-multi-factor-authentication)
* [Mon organisation peut-elle passer de la facturation de consommation par utilisateur à la facturation par authentification, et inversement, à tout moment ?](#can-my-organization-switch-between-per-user-and-per-authentication-consumption-billing-models-at-any-time)
* [Mon organisation peut-elle passer de la facturation à la consommation aux abonnements (modèle basé sur licence) à tout moment ?](#can-my-organization-switch-between-consumption-based-billing-and-subscriptions-a-license-based-model-at-any-time)
* [Mon organisation doit-elle utiliser et synchroniser des identités pour utiliser Azure AD Multi-Factor Authentication ?](#does-my-organization-have-to-use-and-synchronize-identities-to-use-azure-ad-multi-factor-authentication)

### <a name="is-my-organization-charged-for-sending-the-phone-calls-and-text-messages-that-are-used-for-authentication"></a>L’envoi des appels téléphoniques et des SMS utilisés pour l’authentification est-il facturé à mon organisation ?

Non, les appels émis et les SMS envoyés aux utilisateurs via Azure AD Multi-Factor Authentication ne sont pas facturés aux organisations. Si vous utilisez un fournisseur MFA par authentification, vous êtes facturé pour chaque authentification, mais pas pour la méthode utilisée.

Les utilisateurs peuvent se voir facturer les appels téléphoniques ou les SMS qu’ils reçoivent, en fonction de leur service téléphonique personnel.

### <a name="does-the-per-user-billing-model-charge-me-for-all-enabled-users-or-just-the-ones-that-performed-two-step-verification"></a>Le modèle de facturation par utilisateur me facture-t-il tous les utilisateurs activés ou seulement ceux ayant effectué la vérification en deux étapes ?

La facturation est basée sur le nombre d’utilisateurs configurés pour l’utilisation de Multi-Factor Authentication, qu’ils aient ou non lancé une vérification en deux étapes ce mois-ci.

### <a name="how-does-multi-factor-authentication-billing-work"></a>Comment fonctionne la facturation Multi-Factor Authentication ?

Lorsque vous créez un fournisseur MFA par utilisateur ou par authentification, l’abonnement Azure de votre organisation est facturé chaque mois en fonction de l’utilisation. Ce modèle de facturation est similaire à celui d’Azure pour l’utilisation de machines virtuelles et de Web Apps.

Lorsque vous souscrivez un abonnement à Azure AD Multi-Factor Authentication, votre organisation paie uniquement les frais de licence annuels de chaque utilisateur. Les licences MFA et Microsoft 365, Azure AD Premium ou les offres groupées Enterprise Mobility + Security sont facturés de cette manière.

Pour plus d'informations, consultez [Guide pratique pour obtenir Azure AD Multi-Factor Authentication](concept-mfa-licensing.md).

### <a name="is-there-a-free-version-of-azure-ad-multi-factor-authentication"></a>Existe-t-il une version gratuite d'Azure AD Multi-Factor Authentication ?

Les paramètres de sécurité par défaut peuvent être activés au niveau Azure AD Free. Avec les paramètres de sécurité par défaut, tous les utilisateurs sont activés pour l'authentification multifacteur à l’aide de l’application Microsoft Authenticator. Il n’est pas possible d’utiliser la vérification par SMS ou appel téléphonique avec les paramètres de sécurité par défaut, mais uniquement l’application Microsoft Authenticator.

Pour plus d’informations, consultez [Présentation des paramètres de sécurité par défaut](../fundamentals/concept-fundamentals-security-defaults.md).

### <a name="can-my-organization-switch-between-per-user-and-per-authentication-consumption-billing-models-at-any-time"></a>Mon organisation peut-elle passer de la facturation de consommation par utilisateur à la facturation par authentification, et inversement, à tout moment ?

Si votre organisation acquiert MFA en tant que service autonome avec une facturation à la consommation, vous choisissez un modèle de facturation lorsque vous créez un fournisseur MFA. Vous ne pouvez pas modifier le modèle de facturation après la création d’un fournisseur MFA. 

Si votre fournisseur MFA n’est *pas* associé à un locataire Azure AD, ou si vous associez le nouveau fournisseur MFA à un autre locataire Azure AD, les paramètres utilisateur et les options de configuration ne sont pas transférés. Par ailleurs, les serveurs Azure MFA existants doivent être réactivés à l’aide des informations d’identification d’activation générées via le nouveau fournisseur MFA. Le fait de réactiver les serveurs MFA afin de les lier au nouveau fournisseur MFA n’a pas de conséquence sur l’authentification des appels téléphoniques et des SMS. Toutefois, les notifications d’applications mobiles cessent de fonctionner pour tous les utilisateurs jusqu’à ce qu’ils réactivent l’application mobile.

Pour plus d’informations sur les fournisseurs MFA, consultez [Prise en main du fournisseur Azure Multi-Factor Auth](concept-mfa-authprovider.md).

### <a name="can-my-organization-switch-between-consumption-based-billing-and-subscriptions-a-license-based-model-at-any-time"></a>Mon organisation peut-elle passer de la facturation à la consommation aux abonnements (modèle basé sur licence) à tout moment ?

Dans certains cas, oui.

Vous pouvez ajouter des licences MFA, si votre répertoire comporte un fournisseur Azure Multi-Factor Authentication *par utilisateur*. Les utilisateurs disposant de licences ne sont pas comptés dans la facturation à la consommation par utilisateur. Les utilisateurs sans licence peuvent toujours être activés pour MFA via le fournisseur MFA. Si vous achetez et affectez des licences à tous les utilisateurs pour utiliser Multi-Factor Authentication, vous pouvez supprimer le fournisseur Azure Multi-Factor Authentication. Vous pouvez toujours créer un autre fournisseur MFA par utilisateur si vous prévoyez d’avoir plus d’utilisateurs que de licences à l’avenir.

Si votre répertoire comporte un fournisseur Azure Multi-Factor Authentication *par authentification*, vous continuez d’être facturé pour chaque authentification, tant que le fournisseur MFA est lié à votre abonnement. Vous pouvez attribuer des licences MFA aux utilisateurs, mais vous continuez d’être facturé pour chaque demande de vérification en deux étapes, qu’elle provienne d’une personne à laquelle une licence MFA a été affectée ou non.

### <a name="does-my-organization-have-to-use-and-synchronize-identities-to-use-azure-ad-multi-factor-authentication"></a>Mon organisation doit-elle utiliser et synchroniser des identités pour utiliser Azure AD Multi-Factor Authentication ?

Azure Active Directory est facultatif et n’est pas requis en cas d’utilisation des modèles de facturation basés sur la consommation par l’organisation. Si votre fournisseur MFA n’est pas lié à un locataire Azure AD, vous pouvez déployer le serveur Azure Multi-Factor Authentication localement uniquement.

Azure Active Directory est requis pour le modèle basé sur licence car les licences sont ajoutées au client Azure AD lorsque vous les achetez et que vous les affectez à des utilisateurs dans le répertoire.

## <a name="manage-and-support-user-accounts"></a>Gérer et prendre en charge les comptes d’utilisateurs

* [Que dois-je dire à mes utilisateurs s’ils ne reçoivent pas de réponse sur leur téléphone ?](#what-should-i-tell-my-users-to-do-if-they-dont-receive-a-response-on-their-phone)
* [Que faire si un de mes utilisateurs ne peut pas accéder à son compte ?](#what-should-i-do-if-one-of-my-users-cant-get-in-to-their-account)
* [Que faire si un de mes utilisateurs perd un téléphone qui utilise des mots de passe d’applications ?](#what-should-i-do-if-one-of-my-users-loses-a-phone-that-is-using-app-passwords)
* [Que se passe-t-il si un utilisateur ne peut pas se connecter à des applications sans navigateur ?](#what-if-a-user-cant-sign-in-to-non-browser-apps)
* [Mes utilisateurs disent que parfois ils ne reçoivent pas de SMS ou que la vérification a expiré.](#my-users-say-that-sometimes-they-dont-receive-the-text-message-or-the-verification-times-out)
* [Puis-je changer le délai d’expiration du système quand mes utilisateurs doivent entrer le code de vérification à partir d’un SMS ?](#can-i-change-the-amount-of-time-my-users-have-to-enter-the-verification-code-from-a-text-message-before-the-system-times-out)
* [Puis-je utiliser des jetons matériels avec le serveur Azure Multi-Factor Authentication ?](#can-i-use-hardware-tokens-with-azure-multi-factor-authentication-server)
* [Puis-je utiliser le serveur Azure Multi-Factor Authentication pour sécuriser les Services Terminal Server ?](#can-i-use-azure-multi-factor-authentication-server-to-secure-terminal-services)
* [J’ai configuré l’ID d’appelant dans le serveur MFA, mais mes utilisateurs continuent de recevoir des appels Multi-Factor Authentication d’un appelant anonyme.](#i-configured-caller-id-in-mfa-server-but-my-users-still-receive-multi-factor-authentication-calls-from-an-anonymous-caller)
* [Pourquoi mes utilisateurs sont invités à inscrire leurs informations de sécurité ?](#why-are-my-users-being-prompted-to-register-their-security-information)

### <a name="what-should-i-tell-my-users-to-do-if-they-dont-receive-a-response-on-their-phone"></a>Que dois-je dire à mes utilisateurs s’ils ne reçoivent pas de réponse sur leur téléphone ?

Demandez à vos utilisateurs d’essayer jusqu’à cinq fois en 5 minutes d’obtenir un appel téléphonique ou un SMS pour l’authentification. Microsoft utilise plusieurs fournisseurs pour distribuer les appels et les SMS. Si cette approche ne fonctionne pas, ouvrez un cas de support afin d'y remédier.

Les applications de sécurité tierces peuvent également bloquer le SMS ou l'appel téléphonique du code de vérification. Si vous utilisez une application de sécurité tierce, essayez de désactiver la protection, puis demandez l’envoi d’un autre code de vérification MFA.

Si les étapes ci-dessus ne fonctionnent pas, vérifiez si les utilisateurs sont configurés pour plusieurs méthodes de vérification. Essayez de vous reconnecter, mais sélectionnez une autre méthode de vérification sur la page de connexion.

Pour plus d’informations, consultez le [Guide de dépannage de l'utilisateur final](../user-help/multi-factor-authentication-end-user-troubleshoot.md).

### <a name="what-should-i-do-if-one-of-my-users-cant-get-in-to-their-account"></a>Que faire si un de mes utilisateurs ne peut pas accéder à son compte ?

Vous pouvez réinitialiser le compte de l’utilisateur en lui demandant d’effectuer à nouveau le processus d’inscription. Pour en savoir plus, consultez [Gestion des paramètres des utilisateurs et des appareils avec Azure AD Multi-Factor Authentication dans le cloud](howto-mfa-userdevicesettings.md).

### <a name="what-should-i-do-if-one-of-my-users-loses-a-phone-that-is-using-app-passwords"></a>Que faire si un de mes utilisateurs perd un téléphone qui utilise des mots de passe d’applications ?

Supprimez tous les mots de passe d’applications de l’utilisateur afin d’empêcher tout accès non autorisé. Une fois que l’utilisateur dispose d’un appareil de remplacement, il peut recréer les mots de passe. Pour en savoir plus, consultez [Gestion des paramètres des utilisateurs et des appareils avec Azure AD Multi-Factor Authentication dans le cloud](howto-mfa-userdevicesettings.md).

### <a name="what-if-a-user-cant-sign-in-to-non-browser-apps"></a>Que se passe-t-il si un utilisateur ne peut pas se connecter à des applications sans navigateur ?

Si votre organisation utilise encore des clients hérités et si vous [avez autorisé l’utilisation de mots de passe d’application](howto-mfa-app-passwords.md), vos utilisateurs ne peuvent pas se connecter à ces clients hérités avec leur nom d’utilisateur et leur mot de passe. Au lieu de cela, ils doivent [configurer des mots de passe d’application](../user-help/multi-factor-authentication-end-user-app-passwords.md). Vos utilisateurs doivent effacer (supprimer) leurs informations de connexion, redémarrer l’application, puis se connecter avec leur nom d’utilisateur et leur *mot de passe d’application* au lieu de leur mot de passe standard.

Si votre organisation ne comporte pas de clients hérités, vous ne devez pas autoriser vos utilisateurs à créer de mots de passe d’application.

> [!NOTE]
> **Authentification moderne pour les clients Office 2013**
>
> Les mots de passe d’application sont nécessaires uniquement pour les applications ne prenant pas en charge l’authentification moderne. Les clients Office 2013 prennent en charge les protocoles d’authentification modernes, mais doivent être configurés. L’authentification moderne est disponible pour tous les clients exécutant la version de mars 2015 ou la dernière mise à jour pour Office 2013. Pour plus d’informations, consultez le billet de blog [Updated Office 365 modern authentication](https://www.microsoft.com/microsoft-365/blog/2015/11/19/updated-office-365-modern-authentication-public-preview/) (Authentification moderne Office 365 mise à jour).

### <a name="my-users-say-that-sometimes-they-dont-receive-the-text-message-or-the-verification-times-out"></a>Mes utilisateurs disent que parfois ils ne reçoivent pas de SMS ou que la vérification a expiré.

La remise de SMS n’est pas garantie en raison de facteurs incontrôlables susceptibles d’affecter la fiabilité du service. Sont notamment en cause le pays ou la région de destination, l’opérateur de téléphonie mobile et la puissance du signal.

Les applications de sécurité tierces peuvent également bloquer le SMS ou l'appel téléphonique du code de vérification. Si vous utilisez une application de sécurité tierce, essayez de désactiver la protection, puis demandez l’envoi d’un autre code de vérification MFA.

Si vos utilisateurs ont souvent des problèmes liés à la fiabilité de la réception des SMS, indiquez leur d’utiliser plutôt l’application Microsoft Authenticator ou l’appel téléphonique. L’application Microsoft Authenticator peut recevoir des notifications à la fois sur des connexions cellulaires et Wi-Fi. En outre, l’application mobile peut générer des codes de vérification même si l’appareil n’a aucun signal. L’application Microsoft Authenticator est disponible pour [Android](https://go.microsoft.com/fwlink/?Linkid=825072), [iOS](https://go.microsoft.com/fwlink/?Linkid=825073) et [Windows Phone](https://www.microsoft.com/p/microsoft-authenticator/9nblgggzmcj6).

### <a name="can-i-change-the-amount-of-time-my-users-have-to-enter-the-verification-code-from-a-text-message-before-the-system-times-out"></a>Puis-je changer le délai d’expiration du système quand mes utilisateurs doivent entrer le code de vérification à partir d’un SMS ?

Oui, dans certains cas.

Pour les SMS unidirectionnels avec un serveur Azure MFA en version 7.0 ou supérieure, vous pouvez configurer le paramètre de délai d’expiration en définissant une clé de Registre. Une fois le SMS envoyé par le service cloud MFA, le code de vérification (ou code secret à usage unique) est retourné au serveur MFA. Le serveur MFA stocke le code en mémoire pendant 300 secondes par défaut. Si l’utilisateur n’entre pas son code avant l’expiration du délai de 300 secondes, son authentification est refusée. Suivez ces étapes pour modifier le paramètre de délai d’expiration par défaut :

1. Atteindre `HKLM\Software\Wow6432Node\Positive Networks\PhoneFactor`.
2. Créez une clé de Registre **DWORD** appelée *pfsvc_pendingSmsTimeoutSeconds* et définissez la durée de stockage en secondes des codes secrets à usage unique du Azure MFA Server.

>[!TIP]
>
> Si vous utilisez plusieurs serveurs MFA, seul celui qui a traité la demande d’authentification d’origine connaît le code de vérification qui a été envoyé à l’utilisateur. Lorsque l’utilisateur entre le code, la demande d’authentification pour le valider doit être envoyée au même serveur. Si la validation du code est envoyée à un autre serveur, l’authentification est refusée.

Si les utilisateurs ne répondent pas au SMS avant la fin du délai d’expiration défini, leur authentification est refusée.

Pour les SMS unidirectionnels avec Azure AD MFA dans le cloud (adaptateur AD FS ou extension NPS (Network Policy Server) compris), vous ne pouvez pas configurer le paramètre de délai d'expiration. Azure AD stocke le code de vérification pendant 180 secondes.

### <a name="can-i-use-hardware-tokens-with-azure-multi-factor-authentication-server"></a>Puis-je utiliser des jetons matériels avec le serveur Azure Multi-Factor Authentication ?

Si vous utilisez le serveur Azure Multi-Factor Authentication, vous pouvez importer des jetons de mots de passe à usage unique et durée définie (TOTP) d’authentification ouverte tierce (OATH), puis les utiliser pour la vérification en deux étapes.

Vous pouvez utiliser des jetons ActiveIdentity, qui sont des jetons OATH TOTP, si vous placez la clé secrète dans un fichier CSV que vous importez sur le serveur Azure Multi-Factor Authentication. Vous pouvez utiliser des jetons OATH avec les services de fédération Active Directory (AD FS), l’authentification par formulaires Internet Information Server (IIS) et le protocole RADIUS (Remote Authentication Dial-In User Service) à partir du moment où le système client peut accepter une entrée utilisateur.

Vous pouvez importer des jetons OATH TOTP tiers avec les formats suivants :  

- Conteneur portable de clé symétrique (PSKC)
- CSV si le fichier contient un numéro de série, une clé secrète au format Base 32 et un intervalle de temps

### <a name="can-i-use-azure-multi-factor-authentication-server-to-secure-terminal-services"></a>Puis-je utiliser le serveur Azure Multi-Factor Authentication pour sécuriser les Services Terminal Server ?

Oui, mais si vous utilisez Windows Server 2012 R2 ou une version ultérieure, vous pouvez le faire uniquement avec la Passerelle des services Bureau à distance (passerelle RD).

Les modifications de sécurité dans Windows Server 2012 R2 ont changé la façon dont le serveur Azure Multi-Factor Authentication se connecte au package de sécurité de l’autorité de sécurité locale (LSA) dans Windows Server 2012 et les versions antérieures. Pour les versions de Terminal Services sous Windows 2012 ou une version antérieure, vous pouvez [sécuriser une application avec l’authentification Windows](howto-mfaserver-windows.md#to-secure-an-application-with-windows-authentication-use-the-following-procedure). Si vous utilisez Windows Server 2012 R2, vous devez utiliser une passerelle RD.

### <a name="i-configured-caller-id-in-mfa-server-but-my-users-still-receive-multi-factor-authentication-calls-from-an-anonymous-caller"></a>J’ai configuré l’ID d’appelant dans le serveur MFA, mais mes utilisateurs continuent de recevoir des appels Multi-Factor Authentication d’un appelant anonyme.

Lorsque des appels Multi-Factor Authentication sont transmis sur le réseau téléphonique public, ils sont parfois acheminés par le biais d’un opérateur qui ne prend pas en charge les ID d’appelant. Compte tenu du comportement de l'opération, l’ID de l’appelant n’est pas garanti, même si le système Multi-Factor Authentication l’envoie toujours.

### <a name="why-are-my-users-being-prompted-to-register-their-security-information"></a>Pourquoi mes utilisateurs sont invités à inscrire leurs informations de sécurité ?

Il existe plusieurs raisons à cela :

- L’utilisateur a été activé pour MFA par son administrateur dans Azure AD, mais n’a encore enregistré aucune information de sécurité pour son compte.
- L’utilisateur a été autorisé à utiliser la réinitialisation du mot de passe en libre-service dans Azure AD. Les informations de sécurité l’aident à réinitialiser son mot de passe ultérieurement en cas d’oubli.
- L’utilisateur a accédé à une application dotée d’une stratégie d’accès conditionnel exigeant la MFA et n’est pas encore enregistré pour celle-ci.
- Les utilisateurs enregistrent un appareil avec Azure AD (y compris Azure AD Join) et votre organisation exige la MFA pour l’enregistrement de l’appareil. Toutefois, l’utilisateur n’a pas encore été enregistré pour la MFA.
- L’utilisateur génère Windows Hello Entreprise dans Windows 10 (qui nécessite MFA) et n’a pas encore été enregistré pour la MFA.
- L’organisation a créé et activé une stratégie d’inscription MFA qui a été appliquée à l’utilisateur.
- L’utilisateur a été précédemment enregistré pour MFA, mais a choisi une méthode de vérification, qui a depuis été désactivée par un administrateur. L’utilisateur doit donc de nouveau passer par l’inscription MFA pour sélectionner une nouvelle méthode de vérification par défaut.

## <a name="errors"></a>Erreurs

* [Que doit faire un utilisateur s’il voit un message d’erreur « La demande d’authentification ne concerne pas un compte activé » quand il utilise les notifications d’application mobile ?](#what-should-users-do-if-they-see-an-authentication-request-is-not-for-an-activated-account-error-message-when-using-mobile-app-notifications)
* [Que doivent faire les utilisateurs s’ils reçoivent un message d’erreur 0x800434D4L quand ils se connectent à une application sans navigateur ?](#what-should-users-do-if-they-see-a-0x800434d4l-error-message-when-signing-in-to-a-non-browser-application)

### <a name="what-should-users-do-if-they-see-an-authentication-request-is-not-for-an-activated-account-error-message-when-using-mobile-app-notifications"></a>Que doit faire un utilisateur s’il voit un message d’erreur « La demande d’authentification ne concerne pas un compte activé » quand il utilise les notifications d’application mobile ?

Demandez à l’utilisateur de suivre la procédure ci-dessous pour supprimer son compte de l'application Microsoft Authenticator, puis l'ajouter à nouveau :

1. Accédez à [son profil du portail Azure](https://account.activedirectory.windowsazure.com/profile/) et connectez-vous avec un compte professionnel.
2. Sélectionnez **Vérification de sécurité supplémentaire**.
3. Supprimez le compte existant de l’application Microsoft Authenticator.
4. Cliquez sur **Configurer** et suivez les instructions pour configurer à nouveau Microsoft Authenticator.

### <a name="what-should-users-do-if-they-see-a-0x800434d4l-error-message-when-signing-in-to-a-non-browser-application"></a>Que doivent faire les utilisateurs s’ils reçoivent un message d’erreur 0x800434D4L quand ils se connectent à une application sans navigateur ?

L’erreur *0x800434D4L* se produit lorsque vous essayez de vous connecter à une application sans navigateur, installée sur un ordinateur local, qui ne fonctionne pas avec des comptes nécessitant une vérification en deux étapes.

La solution consiste à disposer de comptes d’utilisateur distincts pour les opérations d’administration et les autres. Vous pouvez ultérieurement lier les boîtes de réception de votre compte d’administrateur et d’un compte non administrateur pour vous connecter à Outlook à l’aide de votre compte non administrateur. Pour plus d’informations sur cette solution, consultez [Permettre à un administrateur d’ouvrir et d’afficher le contenu de la boîte aux lettres d’un utilisateur](https://help.outlook.com/141/gg709759.aspx?sl=1).

## <a name="next-steps"></a>Étapes suivantes

Si vous ne trouvez pas la réponse à votre question ici, les options de support suivantes sont disponibles :

* Recherchez des solutions aux problèmes techniques courants dans la [base de connaissances du support Microsoft](https://support.microsoft.com).
* Parcourez les forums et recherchez des questions et des réponses techniques de la communauté ou posez votre question dans les [Questions et réponses Azure Active Directory](/answers/topics/azure-active-directory.html).
* Contactez un professionnel Microsoft via le [Support du serveur Azure Multi-Factor Authentication](https://support.microsoft.com/oas/default.aspx?prid=14947). Lorsque vous nous contactez, veillez à inclure autant d’informations que possible concernant votre problème. Vous pouvez notamment préciser la page sur laquelle vous avez rencontré l’erreur, le code d’erreur spécifique, l’ID de session spécifique et l’ID de l’utilisateur qui a vu l’erreur.
* Si vous êtes un client hérité de PhoneFactor et que vous avez des questions ou besoin d’aide pour réinitialiser un mot de passe, utilisez l'adresse e-mail [phonefactorsupport@microsoft.com](mailto:phonefactorsupport@microsoft.com) pour ouvrir une demande de support.