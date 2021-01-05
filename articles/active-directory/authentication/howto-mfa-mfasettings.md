---
title: Configurer Azure AD Multi-Factor Authentication - Azure Active Directory
description: Apprenez à configurer les paramètres d'Azure AD Multi-Factor Authentication sur le portail Azure
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 06/05/2020
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.custom: contperf-fy20q4
ms.openlocfilehash: 209125a5cc8d86c1af07966d52681aa91ea7dc19
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97026997"
---
# <a name="configure-azure-ad-multi-factor-authentication-settings"></a>Configurer les paramètres d'Azure AD Multi-Factor Authentication

Pour personnaliser l'expérience de l'utilisateur final avec Azure AD Multi-Factor Authentication (MFA), vous pouvez configurer des options pour les paramètres tels que les seuils de verrouillage de compte ou les alertes et notifications de fraude. Certains paramètres se trouvent sur le portail Azure d'Azure Active Directory (Azure AD), et d'autres sur un portail Azure AD Multi-Factor Authentication distinct.

Les paramètres Azure AD Multi-Factor Authentication suivants sont disponibles sur le portail Azure :

| Fonctionnalité | Description |
| ------- | ----------- |
| [Verrouillage de compte](#account-lockout) | Verrouillez temporairement des comptes pour les empêcher d'utiliser Azure AD Multi-Factor Authentication si un trop grand nombre de tentatives d'authentification successives sont refusées. Cette fonctionnalité s’applique seulement aux utilisateurs qui entrent un code PIN pour s’authentifier. (MFA Server) |
| [Blocage/déblocage des utilisateurs](#block-and-unblock-users) | Empêchez des utilisateurs spécifiques de recevoir des demandes d'Azure AD Multi-Factor Authentication. Toutes les tentatives d’authentification des utilisateurs bloqués sont automatiquement refusées. Ces utilisateurs restent bloqués pendant 90 jours à partir de leur blocage ou jusqu’à ce qu’ils soient débloqués manuellement. |
| [Alerte de fraude](#fraud-alert) | Configurez les paramètres qui permettent aux utilisateurs de signaler les demandes de vérification frauduleuses. |
| [Notifications](#notifications) | Activez les notifications des événements provenant de MFA Server. |
| [Jetons OATH](concept-authentication-oath-tokens.md) | Fonctionnalité utilisée dans les environnements Azure AD MFA basés sur le cloud afin de gérer les jetons OATH des utilisateurs. |
| [Paramètres de l’appel téléphonique](#phone-call-settings) | Configurez les paramètres liés aux appels téléphoniques et aux messages d’accueil pour les environnements cloud et locaux. |
| Fournisseurs | Ceci montre tous les fournisseurs d’authentification existants que vous avez associés à votre compte. À compter du 1er septembre 2018, vous ne pouvez plus créer des fournisseurs d’authentification |

![Portail Azure - Paramètres d’Azure AD Multi-Factor Authentication](./media/howto-mfa-mfasettings/multi-factor-authentication-settings-portal.png)

## <a name="account-lockout"></a>Verrouillage de compte

Pour éviter les tentatives d’authentification multifacteur répétées dans le cadre d’une attaque, utilisez les paramètres de verrouillage de compte pour spécifier le nombre d’échecs de tentatives autorisé avant que le compte ne soit verrouillé pendant un certain temps. Les paramètres de verrouillage de compte sont appliqués uniquement quand un code PIN est entré à l’invite MFA.

Les options suivantes sont disponibles :

* Nombre de refus MFA après lequel déclencher un verrouillage de compte
* Nombre de minutes après lequel le nombre de verrouillages de compte est réinitialisé
* Nombre de minutes avant que le compte soit déverrouillé automatiquement

Pour configurer les paramètres de verrouillage de compte, renseignez les paramètres suivants :

1. Connectez-vous au [portail Azure](https://portal.azure.com) en tant qu’administrateur.
1. Accédez à **Azure Active Directory** > **Sécurité** > **MFA** > **Verrouillage de compte**.
1. Entrez les valeurs requises pour votre environnement, puis sélectionnez **Enregistrer**.

    ![Capture d’écran des paramètres de verrouillage de compte dans le portail Azure](./media/howto-mfa-mfasettings/account-lockout-settings.png)

## <a name="block-and-unblock-users"></a>Bloquer et débloquer des utilisateurs

En cas de perte ou de vol de l'appareil d'un utilisateur, vous pouvez bloquer les tentatives Azure AD Multi-Factor Authentication pour le compte associé. Toutes les tentatives Azure AD Multi-Factor Authentication des utilisateurs bloqués sont automatiquement refusées. La durée de blocage de ces utilisateurs est de 90 jours à partir du moment où ils sont bloqués.

### <a name="block-a-user"></a>Bloquer un utilisateur

Pour bloquer un utilisateur, effectuez les étapes suivantes :

1. Accédez à **Azure Active Directory** > **Sécurité** > **MFA** > **Bloquer/débloquer des utilisateurs**.
1. Sélectionnez **Ajouter** pour bloquer un utilisateur.
1. Sélectionnez **Groupe de réplication**, puis choisissez *Azure par défaut*.

    Entrez le nom d’utilisateur de l’utilisateur bloqué au format `username\@domain.com`, puis entrez un commentaire dans le champ *Raison*.
1. Quand vous êtes prêt, sélectionnez **OK** pour bloquer l’utilisateur.

### <a name="unblock-a-user"></a>Débloquer un utilisateur

Pour débloquer un utilisateur, effectuez les étapes suivantes :

1. Accédez à **Azure Active Directory** > **Sécurité** > **MFA** > **Bloquer/débloquer des utilisateurs**.
1. Dans la colonne *Action* à côté de l’utilisateur à débloquer, sélectionnez **Débloquer**.
1. Entrez un commentaire dans le champ *Motif du déblocage*.
1. Quand vous êtes prêt, sélectionnez **OK** pour débloquer l’utilisateur.

## <a name="fraud-alert"></a>Alerte de fraude

La fonctionnalité d’alerte de fraude permet aux utilisateurs de signaler les tentatives frauduleuses d’accès à leurs ressources. Quand ils reçoivent une invite MFA inconnue et suspecte, les utilisateurs peuvent signaler la tentative de fraude dans l’application Microsoft Authenticator ou par téléphone.

Les options de configuration des alertes de fraude suivantes sont disponibles :

* **Bloquer automatiquement les utilisateurs qui signalent une fraude** : Si un utilisateur signale une fraude, les tentatives d'authentification Azure AD MFA du compte d'utilisateur sont bloquées pendant 90 jours ou jusqu'à ce qu'un administrateur débloque le compte. Un administrateur peut consulter les connexions à l’aide du rapport de connexion et prendre les mesures appropriées pour empêcher les fraudes futures. Un administrateur peut ensuite [débloquer](#unblock-a-user) le compte de l’utilisateur.
* **Code pour signaler une fraude lors du message d’accueil initial** : quand les utilisateurs reçoivent un appel téléphonique pour effectuer l’authentification multifacteur, ils appuient normalement sur **#** pour confirmer leur connexion. Pour signaler une fraude, l’utilisateur doit saisir un code avant d’appuyer sur **#** . Ce code est **0** par défaut, mais vous pouvez le personnaliser.

   > [!NOTE]
   > Le message d’accueil par défaut de Microsoft demande aux utilisateurs d’appuyer sur **0#** pour envoyer une alerte de fraude. Si vous souhaitez utiliser un code autre que **0**, enregistrez et chargez vos propres messages d’accueil vocaux personnalisés avec les instructions appropriées pour vos utilisateurs.

Pour activer et configurer les alertes de fraude, effectuez les étapes suivantes :

1. Accédez à **Azure Active Directory** > **Sécurité** > **MFA** > **Alerte fraude**.
1. Réglez *Autoriser les utilisateurs à envoyer des alertes de fraude* sur **Activé**.
1. Configurez le paramètre *Bloquer automatiquement les utilisateurs qui signalent une fraude* ou *Code pour signaler une fraude lors du message d’accueil initial* selon vos besoins.
1. Quand vous êtes prêt, sélectionnez **Enregistrer**.

### <a name="view-fraud-reports"></a>Afficher les rapports de fraude

Sélectionnez **Azure Active Directory** > **Connexions** > **Informations d’authentification**. Le rapport sur les fraudes fait maintenant partie du rapport des connexions Azure AD standard. Il s’affiche dans **« Détails des résultats »** (accès refusé par MFA, code fraude entré).
 
## <a name="notifications"></a>Notifications

Des notifications par e-mail peuvent être configurées et envoyées quand des utilisateurs signalent des alertes de fraude. Ces notifications sont généralement envoyées aux administrateurs d’identité, car les informations d’identification du compte de l’utilisateur sont susceptibles d’être compromises. L’exemple suivant montre à quoi ressemble un e-mail de notification d’alerte de fraude :

![Exemple d’e-mail de notification d’alerte de fraude](./media/howto-mfa-mfasettings/multi-factor-authentication-fraud-alert-email.png)

Pour configurer les notifications d’alerte de fraude, renseignez les paramètres suivants :

1. Accédez à **Azure Active Directory** > **Sécurité** > **Multi-Factor Authentication** > **Notifications**.
1. Entrez l’adresse e-mail à ajouter dans la zone d’à côté.
1. Pour supprimer une adresse e-mail existante, sélectionnez l’option **...** à côté de l’adresse e-mail souhaitée, puis sélectionnez **Supprimer**.
1. Quand vous êtes prêt, sélectionnez **Enregistrer**.

## <a name="oath-tokens"></a>Jetons OATH

Azure AD prend en charge l’utilisation de jetons OATH-TOTP SHA-1 qui actualisent les codes toutes les 30 ou 60 secondes. Les clients peuvent acheter ces jetons auprès du fournisseur de leur choix.

Les jetons matériels OATH TOTP sont généralement fournis avec une clé secrète, ou valeur initiale, préprogrammée dans le jeton. Ces clés doivent être entrées dans Azure AD comme décrit dans les étapes suivantes. Les clés secrètes sont limitées à 128 caractères et cette limite peut ne pas être compatible avec tous les jetons. La clé secrète peut contenir uniquement les caractères *a à z* ou *A à Z* et les chiffres *1 à 7*, et doit être encodée en *Base32*.

Vous pouvez également configurer des jetons matériels OATH TOTP programmables qui peuvent être réamorcés avec Azure AD dans le processus d’installation des jetons logiciels.

Les jetons matériels OATH sont pris en charge dans le cadre d’une préversion publique. Pour plus d’informations sur les préversions, consultez [Conditions d’utilisation supplémentaires pour les préversions de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)

![Chargement des jetons OATH dans le panneau de jetons OATH MFA](media/concept-authentication-methods/mfa-server-oath-tokens-azure-ad.png)

Après avoir obtenu les jetons, vous devez les charger dans un fichier de valeurs séparées par des virgules (CSV), contenant l'UPN, le numéro de série, la clé secrète, l’intervalle de temps, le fabricant et le modèle, comme indiqué dans l’exemple suivant :

```csv
upn,serial number,secret key,time interval,manufacturer,model
Helga@contoso.com,1234567,1234567abcdef1234567abcdef,60,Contoso,HardwareKey
```

> [!NOTE]
> Veillez à inclure la ligne d’en-tête dans votre fichier CSV.

Une fois que le fichier a été correctement mis en forme au format CSV, l’administrateur peut ensuite se connecter au Portail Azure et accéder à **Azure Active Directory > Sécurité > MFA > Jetons OATH** afin de charger le fichier CSV créé.

L’opération peut prendre plusieurs minutes selon la taille du fichier CSV. Sélectionnez le bouton **Actualiser** pour obtenir l’état actuel. Si le fichier contient des erreurs, vous pouvez télécharger un fichier CSV qui répertorie les erreurs à résoudre. Les noms de champs dans le fichier CSV téléchargé sont différents de ceux de la version chargée.

Une fois que toutes les erreurs ont été résolues, l’administrateur peut activer chaque clé en sélectionnant **Activer** pour le jeton et en entrant l’OTP affiché sur le jeton.

Les utilisateurs peuvent combiner jusqu’à cinq jetons matériels OATH ou des applications d’authentification, comme l’application Microsoft Authenticator, configurées pour une utilisation à tout moment.

## <a name="phone-call-settings"></a>Paramètres de l’appel téléphonique

Si les utilisateurs reçoivent des appels téléphoniques pour les invites MFA, vous pouvez configurer leur expérience, notamment leur ID d’appelant ou le message d’accueil vocal qu’ils entendent.

Aux États-Unis, si vous n’avez pas configuré l’ID de l’appelant MFA, les appels vocaux de Microsoft proviennent des numéros ci-dessous. Si vous utilisez des filtres de courrier indésirable, vous devez exclure ces numéros :

* *+1 (866) 539 4191*
* *+1 (855) 330 8653*
* *+1 (877) 668 6536*

> [!NOTE]
> Lorsque des appels Azure AD Multi-Factor Authentication sont passés sur le réseau téléphonique public, ceux-ci sont parfois acheminés par le biais d'un opérateur qui ne prend pas en charge les ID d'appelant. C'est la raison pour laquelle l'ID de l'appelant n'est pas garanti, même si Azure AD Multi-Factor Authentication l'envoie toujours. Cela s'applique à la fois aux appels téléphoniques et aux SMS fournis par Azure AD Multi-Factor Authentication. Si vous devez confirmer qu'un SMS provient d'Azure AD Multi-Factor Authentication, consultez [Quels sont les codes courts SMS utilisés pour envoyer des SMS ?](multi-factor-authentication-faq.md#what-sms-short-codes-are-used-for-sending-sms-messages-to-my-users)

Pour configurer votre propre numéro d’ID d’appelant, effectuez les étapes suivantes :

1. Accédez à **Azure Active Directory** > **Sécurité** > **MFA** > **Paramètres de l’appel téléphonique**.
1. Définissez le **Numéro d’ID de l’appelant MFA** sur le numéro qui doit s’afficher sur le téléphone des utilisateurs. Seuls les numéros basés aux États-Unis sont autorisés.
1. Quand vous êtes prêt, sélectionnez **Enregistrer**.

### <a name="custom-voice-messages"></a>Messages vocaux personnalisés

Vous pouvez utiliser vos propres enregistrements ou messages d'accueil pour Azure AD Multi-Factor Authentication grâce à la fonctionnalité des messages vocaux personnalisés. Ces messages peuvent servir à compléter ou à remplacer les enregistrements Microsoft par défaut.

Avant de commencer, tenez compte des restrictions suivantes :

* Les formats de fichiers pris en charge sont *.wav* et *.mp3*.
* La taille limite des fichiers est de 1 Mo.
* Les messages d’authentification doivent durer moins de 20 secondes. Les messages de plus de 20 secondes peuvent faire échouer la vérification. L’utilisateur peut ne pas répondre avant la fin du message et de la durée de vérification.

### <a name="custom-message-language-behavior"></a>Comportement lié à la langue du message personnalisé

Quand un message vocal personnalisé est lu à l’utilisateur, la langue du message dépend des facteurs suivants :

* La langue de l’utilisateur actuel.
  * La langue détectée par le navigateur de l’utilisateur.
  * D’autres scénarios d’authentification peuvent se comporter différemment.
* La langue des messages personnalisés disponibles.
  * Cette langue est choisie par l’administrateur, au moment de l’ajout d’un message personnalisé.

Par exemple, s’il n’existe qu’un seul message personnalisé et que la langue est l’allemand :

* Un utilisateur qui s’authentifie en allemand entend le message allemand personnalisé.
* Un utilisateur qui s’authentifie en anglais entend le message anglais standard.

### <a name="custom-voice-message-defaults"></a>Valeurs par défaut des messages vocaux personnalisés

Vous pouvez créer vos propres messages personnalisés sur la base des exemples de scripts ci-dessous. Ces textes de script sont utilisés par défaut si vous ne configurez pas vos propres messages personnalisés :

| Nom du message | Script |
| --- | --- |
| Authentification réussie | Votre connexion a bien été vérifiée. Au revoir. |
| Invite à entrer une extension | Merci d'avoir choisi d'utiliser le système de vérification de connexion de Microsoft. Appuyez sur la touche dièse pour continuer. |
| Confirmation de fraude | Une alerte de fraude a été envoyée. Pour débloquer votre compte, veuillez contacter le support informatique de votre société. |
| Message d'accueil en cas de fraude (standard) | Merci d'avoir choisi d'utiliser le système de vérification de connexion de Microsoft. Appuyez sur la touche dièse pour terminer la vérification. Si vous n’êtes pas à l’origine de cette vérification, quelqu’un tente peut-être d’accéder à votre compte. Veuillez appuyer sur zéro puis sur dièse pour envoyer une alerte de fraude. Cela avertira l’équipe informatique de votre entreprise et bloquera les autres tentatives de vérification. |
| Fraude signalée – Une alerte de fraude a été envoyée. | Pour débloquer votre compte, veuillez contacter le support informatique de votre société. |
| Activation | Merci d'avoir choisi d'utiliser le système de vérification de connexion de Microsoft. Appuyez sur la touche dièse pour terminer la vérification. |
| Nouvelle tentative d'authentification refusée | Vérification refusée. |
| Réessayer (standard) | Merci d'avoir choisi d'utiliser le système de vérification de connexion de Microsoft. Appuyez sur la touche dièse pour terminer la vérification. |
| Message d'accueil (standard) | Merci d'avoir choisi d'utiliser le système de vérification de connexion de Microsoft. Appuyez sur la touche dièse pour terminer la vérification. |
| Message d'accueil (PIN) | Merci d'avoir choisi d'utiliser le système de vérification de connexion de Microsoft. Veuillez entrer votre code PIN suivi de la clé dièse pour terminer votre vérification. |
| Message d'accueil en cas de fraude (PIN) | Merci d'avoir choisi d'utiliser le système de vérification de connexion de Microsoft.  Veuillez entrer votre code PIN suivi de la clé dièse pour terminer votre vérification. Si vous n’êtes pas à l’origine de cette vérification, quelqu’un tente peut-être d’accéder à votre compte. Veuillez appuyer sur zéro puis sur dièse pour envoyer une alerte de fraude. Cela avertira l’équipe informatique de votre entreprise et bloquera les autres tentatives de vérification. |
| Nouvelle tentative (PIN) | Merci d'avoir choisi d'utiliser le système de vérification de connexion de Microsoft. Veuillez entrer votre code PIN suivi de la clé dièse pour terminer votre vérification. |
| Invite à entrer une extension après les chiffres | Si vous utilisez déjà cette extension, appuyez sur la touche dièse pour continuer. |
| Authentification refusée | Désolé, nous ne pouvons pas vous connecter pour le moment. Veuillez réessayer plus tard. |
| Message d'accueil lors de l'activation (standard) | Merci d'avoir choisi d'utiliser le système de vérification de connexion de Microsoft. Appuyez sur la touche dièse pour terminer la vérification. |
| Nouvelle tentative d'activation (standard) | Merci d'avoir choisi d'utiliser le système de vérification de connexion de Microsoft. Appuyez sur la touche dièse pour terminer la vérification. |
| Message d'accueil lors de l'activation (PIN) | Merci d'avoir choisi d'utiliser le système de vérification de connexion de Microsoft. Veuillez entrer votre code PIN suivi de la clé dièse pour terminer votre vérification. |
| Invite à entrer une extension avant les chiffres | Merci d'avoir choisi d'utiliser le système de vérification de connexion de Microsoft. Veuillez transférer cet appel vers l’extension... |

### <a name="set-up-a-custom-message"></a>Configurer un message personnalisé

Pour utiliser vos propres messages personnalisés, effectuez les étapes suivantes :

1. Accédez à **Azure Active Directory** > **Sécurité** > **MFA** > **Paramètres de l’appel téléphonique**.
1. Sélectionnez **Ajouter un message d’accueil**.
1. Choisissez un **Type** de message d’accueil, par exemple *Message d’accueil (standard)* ou *Authentification réussie*.
1. Sélectionnez la **Langue** conformément aux informations de la section précédente, [Comportement lié à la langue du message personnalisé](#custom-message-language-behavior).
1. Recherchez et sélectionnez un fichier audio *.mp3* ou *.wav* à charger.
1. Quand vous êtes prêt, sélectionnez **Ajouter**, puis **Enregistrer**.

## <a name="mfa-service-settings"></a>Paramètres du service MFA

Les paramètres relatifs aux mots de passe des applications, aux adresses IP approuvées, aux options de vérification et à la mémorisation de l'authentification multifacteur pour Azure AD Multi-Factor Authentication se trouvent dans les paramètres de service. Il s’agit d’un portail hérité, qui ne fait pas partie du portail Azure AD standard.

Vous pouvez accéder aux paramètres du service à partir du portail Azure via **Azure Active Directory** > **Sécurité** > **MFA** > **Mise en route** > **Configurer** > **Paramètres de MFA basé dans le cloud supplémentaires**. Une nouvelle fenêtre ou un nouvel onglet s’ouvre avec des options supplémentaires pour les *paramètres de service*.

## <a name="trusted-ips"></a>Adresses IP approuvées

La fonctionnalité _Adresses IP approuvées_ d'Azure AD Multi-Factor Authentication contourne les invites d'authentification multifacteur pour les utilisateurs qui se connectent à partir d'une plage d'adresses IP définie. Vous pouvez définir des plages d'adresses IP approuvées pour vos environnements locaux afin qu'aucune invite Azure AD Multi-Factor Authentication ne soit présentée aux utilisateurs qui se trouvent à l'un de ces emplacements.

> [!NOTE]
> Les adresses IP approuvées peuvent inclure des plages d’adresses IP privées uniquement si vous utilisez le serveur MFA. Dans un environnement Azure AD Multi-Factor Authentication basé sur le cloud, vous pouvez uniquement utiliser des plages d'adresses IP publiques.
>
> Les plages d’adresses IPv6 sont uniquement prises en charge dans l’interface [Emplacement nommé (préversion)](../conditional-access/location-condition.md#preview-features).

Si votre organisation déploie l’extension de serveur NPS pour fournir l’authentification multifacteur aux applications locales, notez que l’adresse IP source apparaît toujours comme étant le serveur NPS qu’emprunte la tentative d’authentification.

| Type de locataire Azure AD | Options de la fonctionnalité Adresses IP approuvées |
|:--- |:--- |
| Adresses IP gérées |**Plage d’adresses IP spécifiques** : Les administrateurs spécifient une plage d’adresses IP pouvant contourner l’authentification multifacteur pour les utilisateurs qui se connectent à partir de l’intranet de l’entreprise. Un maximum de 50 plages d’adresses IP approuvées peuvent être configurées.|
| Adresses IP fédérées |**Tous les utilisateurs fédérés** : Tous les utilisateurs fédérés qui se connectent au sein de l’organisation peuvent contourner l’authentification multifacteur. Les utilisateurs contournent la vérification à l’aide d’une revendication émise par les Services de fédération Active Directory (AD FS).<br/>**Plage d’adresses IP spécifiques** : Les administrateurs spécifient une plage d’adresses IP pouvant contourner l’authentification multifacteur pour les utilisateurs qui se connectent à partir de l’intranet de l’entreprise. |

Le contournement de la vérification des adresses IP approuvées n’est possible qu’au sein de l’intranet de l’entreprise. Si vous sélectionnez l’option **Tous les utilisateurs fédérés** et qu’un utilisateur se connecte en dehors de l’intranet de l’entreprise, celui-ci doit utiliser l’authentification multifacteur. Le processus est le même, même si l’utilisateur présente une revendication AD FS.

### <a name="end-user-experience-inside-of-corpnet"></a>Expérience de l’utilisateur final au sein du réseau d’entreprise

Quand la fonction Adresses IP approuvées est désactivée, l’authentification multifacteur est obligatoire pour les flux de navigateur. Les mots de passe d’application sont requis pour les anciennes applications clientes enrichies.

Avec des adresses IP approuvées, l’authentification multifacteur n’est pas obligatoire pour les flux de navigateur. Les mots de passe d’application ne sont pas obligatoires pour les anciennes applications clientes enrichies, sauf si l’utilisateur avait créé un mot de passe d’application. Dès qu’un mot de passe est utilisé, il reste requis.

### <a name="end-user-experience-outside-corpnet"></a>Expérience de l’utilisateur final en dehors du réseau de l’entreprise

Avec ou sans adresses IP approuvées, l’authentification multifacteur est obligatoire pour les flux de navigateur. Les mots de passe d’application sont requis pour les anciennes applications clientes enrichies.

### <a name="enable-named-locations-by-using-conditional-access"></a>Activer les emplacements nommés à l’aide de l’accès conditionnel

Vous pouvez utiliser les règles d’accès conditionnel pour définir des emplacements nommés de cette façon :

1. Dans le portail Azure, recherchez et sélectionnez **Azure Active Directory**, puis accédez à **Sécurité** > **Accès conditionnel** > **Emplacements nommés**.
1. Sélectionnez **Nouvel emplacement**.
1. Entrez le nom de l’emplacement.
1. Sélectionnez **Marquer comme emplacement approuvé**.
1. Entrez la plage d’adresses IP en notation CIDR pour votre environnement, par exemple *40.77.182.32/27*.
1. Sélectionnez **Create** (Créer).

### <a name="enable-the-trusted-ips-feature-by-using-conditional-access"></a>Activer la fonctionnalité Adresses IP approuvées à l’aide de l’accès conditionnel

Pour activer les adresses IP approuvées à l’aide de stratégies d’accès conditionnel, effectuez les étapes suivantes :

1. Dans le portail Azure, recherchez et sélectionnez **Azure Active Directory**, puis accédez à **Sécurité** >  **Accès conditionnel** > **Emplacements nommés**.
1. Sélectionnez **Configurer des adresses IP approuvées MFA**.
1. Dans la page **Paramètres du service**, sous **Adresses IP approuvées**, choisissez l’une des deux options suivantes :

   * **Pour les requêtes issues d’utilisateurs fédérés provenant de mon intranet** : Pour choisir cette option, activez la case à cocher. Tous les utilisateurs fédérés qui se connectent à partir du réseau d’entreprise contournent l’authentification multifacteur en utilisant une revendication émise par AD FS. Vérifiez qu’AD FS possède une règle pour ajouter la revendication de l’intranet au trafic approprié. Si la règle n’existe pas, créez la règle suivante dans AD FS :

      `c:[Type== "http://schemas.microsoft.com/ws/2012/01/insidecorporatenetwork"] => issue(claim = c);`

   * **Pour les requêtes provenant d’une plage spécifique d’adresses IP** : pour choisir cette option, saisissez les adresses IP dans la zone de texte en notation CIDR.
      * Pour les adresses IP qui se trouvent entre xxx.xxx.xxx.1 et xxx.xxx.xxx.254, utilisez la notation **xxx.xxx.xxx.0/24**.
      * Pour une seule adresse IP, utilisez la notation **xxx.xxx.xxx.xxx/32**.
      * Vous pouvez saisir jusqu’à 50 plages d’adresses IP. Les utilisateurs qui se connectent à partir de ces adresses IP contournent l’authentification multifacteur.

1. Sélectionnez **Enregistrer**.

### <a name="enable-the-trusted-ips-feature-by-using-service-settings"></a>Activer la fonctionnalité Adresses IP approuvées à l’aide des paramètres du service

Si vous ne souhaitez pas utiliser de stratégies d'accès conditionnel pour activer les adresses IP approuvées, configurez les *paramètres de service* d'Azure AD Multi-Factor Authentication comme suit :

1. Dans le portail Azure, recherchez et sélectionnez **Azure Active Directory**, puis choisissez **Utilisateurs**.
1. Sélectionnez **Multi-Factor Authentication**.
1. Sous Multi-Factor Authentication, sélectionnez **Paramètres du service**.
1. Dans la page **Paramètres du service**, sous **Adresses IP approuvées**, choisissez une des deux options suivantes (ou les deux) :

   * **Pour les requêtes issues d’utilisateurs fédérés de mon intranet** : Pour choisir cette option, activez la case à cocher. Tous les utilisateurs fédérés qui se connectent à partir du réseau d’entreprise contournent l’authentification multifacteur en utilisant une revendication émise par AD FS. Vérifiez qu’AD FS possède une règle pour ajouter la revendication de l’intranet au trafic approprié. Si la règle n’existe pas, créez la règle suivante dans AD FS :

      `c:[Type== "http://schemas.microsoft.com/ws/2012/01/insidecorporatenetwork"] => issue(claim = c);`

   * **Pour les requêtes provenant d’une plage spécifique d’adresses IP** : pour choisir cette option, saisissez les adresses IP dans la zone de texte en notation CIDR.
      * Pour les adresses IP qui se trouvent entre xxx.xxx.xxx.1 et xxx.xxx.xxx.254, utilisez la notation **xxx.xxx.xxx.0/24**.
      * Pour une seule adresse IP, utilisez la notation **xxx.xxx.xxx.xxx/32**.
      * Vous pouvez saisir jusqu’à 50 plages d’adresses IP. Les utilisateurs qui se connectent à partir de ces adresses IP contournent l’authentification multifacteur.

1. Sélectionnez **Enregistrer**.

## <a name="verification-methods"></a>Méthodes de vérification

Vous pouvez choisir les méthodes de vérification mises à la disposition des utilisateurs dans le portail des paramètres de service. Lorsque vos utilisateurs inscrivent leurs comptes à Azure AD Multi-Factor Authentication, ils choisissent leur méthode de vérification préférée parmi celles que vous avez activées. Vous trouverez de l’aide sur le processus d’inscription des utilisateurs dans [Configurer mon compte pour l’authentification multifacteur](../user-help/multi-factor-authentication-end-user-first-time.md).

Les méthodes de vérification disponibles sont les suivantes :

| Méthode | Description |
|:--- |:--- |
| Appel vers le téléphone |Passe un appel vocal automatisé. L’utilisateur répond à l’appel et appuie sur la touche # du clavier du téléphone pour s’authentifier. Ce numéro de téléphone n’est pas synchronisé avec Active Directory en local. |
| Message texte vers le téléphone |Envoie un message texte contenant un code de vérification. L’utilisateur est invité à entrer le code de vérification dans l’interface de connexion. On parle alors « SMS unidirectionnel ». Ce terme signifie que l’utilisateur doit renvoyer par SMS dans un code spécifique. Le SMS bidirectionnel est déconseillé et ne sera plus pris en charge après le 14 novembre 2018. Les administrateurs doivent activer une autre méthode pour les utilisateurs qui utilisaient précédemment des SMS bidirectionnels.|
| Notification via une application mobile |Envoie une notification Push sur votre téléphone ou votre appareil inscrit. L’utilisateur consulte la notification et sélectionne **Vérifier** pour terminer la vérification. L’application Microsoft Authenticator est disponible pour [Windows Phone](https://www.microsoft.com/p/microsoft-authenticator/9nblgggzmcj6), [Android](https://go.microsoft.com/fwlink/?Linkid=825072) et [iOS](https://go.microsoft.com/fwlink/?Linkid=825073). |
| Code de vérification provenant d’une application mobile ou d’un jeton matériel |L’application Microsoft Authenticator génère un nouveau code de vérification OATH toutes les 30 secondes. L’utilisateur entre ce code de vérification dans l’interface de connexion. L’application Microsoft Authenticator est disponible pour [Windows Phone](https://www.microsoft.com/p/microsoft-authenticator/9nblgggzmcj6), [Android](https://go.microsoft.com/fwlink/?Linkid=825072) et [iOS](https://go.microsoft.com/fwlink/?Linkid=825073). |

Pour plus d’informations, consultez [Quelles sont les méthodes d’authentification et de vérification disponibles dans Azure AD ?](concept-authentication-methods.md)

### <a name="enable-and-disable-verification-methods"></a>Activer et désactiver des méthodes de vérification

Pour activer ou désactiver les méthodes de vérification, effectuez les étapes suivantes :

1. Dans le portail Azure, recherchez et sélectionnez **Azure Active Directory**, puis choisissez **Utilisateurs**.
1. Sélectionnez **Multi-Factor Authentication**.
1. Sous Multi-Factor Authentication, sélectionnez **Paramètres du service**.
1. Dans la page **Paramètres du service**, sous **Options de vérification**, sélectionnez (ou désélectionnez) les options que vous souhaitez proposer à vos utilisateurs.
1. Cliquez sur **Enregistrer**.

## <a name="remember-multi-factor-authentication"></a>Mémoriser Multi-Factor Authentication

La fonctionnalité de _mémorisation de Multi-Factor Authentication_ permet aux utilisateurs de contourner les vérifications suivantes pendant un nombre spécifié de jours après s’être connectés à un appareil à l’aide de Multi-Factor Authentication. Pour améliorer la convivialité et réduire le nombre de fois qu’un utilisateur doit effectuer une authentification multifacteur sur le même appareil, sélectionnez une durée de 90 jours ou plus.

> [!IMPORTANT]
> Si un appareil ou un compte est compromis, la mémorisation de Multi-Factor Authentication sur des appareils approuvés peut affecter la sécurité. En cas de violation d’un compte d’entreprise ou de perte/vol d’un appareil fiable, vous devez [révoquer les sessions MFA](howto-mfa-userdevicesettings.md).
>
> L’action de restauration révoque le statut approuvé de tous les appareils, et oblige l’utilisateur à procéder de nouveau à l’authentification multifacteur. Vous pouvez également demander aux utilisateurs de restaurer Multi-Factor Authentication sur leurs propres appareils, comme cela est indiqué dans [Gérer vos paramètres pour l’authentification multifacteur](../user-help/multi-factor-authentication-end-user-manage-settings.md#turn-on-two-factor-verification-prompts-on-a-trusted-device).

### <a name="how-the-feature-works"></a>Principe de la fonctionnalité

La fonctionnalité de mémorisation de Multi-Factor Authentication crée un cookie persistant sur le navigateur lorsqu’un utilisateur sélectionne **Don’t ask again for X days (Ne plus me demander pendant X jours)** lors de sa connexion. L’utilisateur n’est pas réinvité à se connecter à Multi-Factor Authentication sur ce navigateur tant que le cookie reste valable. Si l’utilisateur ouvre un autre navigateur sur le même appareil ou s’il efface les cookies, il recevra de nouveau l’invite de vérification.

L’option **Don’t ask again for X days (Ne plus me demander pendant X jours)** ne s’affiche pas dans les applications sans navigateur, qu’elles prennent en charge ou non l’authentification moderne. Ces applications utilisent des _jetons d’actualisation_ qui fournissent de nouveaux jetons d’accès toutes les heures. Quand un jeton d’actualisation est validé, Azure AD vérifie que la dernière authentification multifacteur est intervenue dans le nombre spécifié de jours.

La fonctionnalité réduit le nombre d’authentifications sur les applications web, qui interviennent normalement à chaque fois. La fonctionnalité peut augmenter le nombre d’authentifications pour les clients d’authentification modernes qui, normalement, reçoivent une invite tous les 90 jours, si une durée inférieure est configurée. Elle peut également augmenter le nombre d’authentifications quand elle est combinée avec des stratégies d’accès conditionnel.

> [!IMPORTANT]
> La fonctionnalité de **mémorisation de Multi-Factor Authentication** n’est pas compatible avec la fonctionnalité **Maintenir la connexion** d’AD FS, lorsque les utilisateurs effectuent l’authentification multifacteur pour AD FS via le serveur Azure Multi-Factor Authentication ou avec une solution d’authentification multifacteur tierce.
>
> Si votre utilisateur sélectionne **Maintenir la connexion** dans AD FS et désigne son appareil comme approuvé pour Multi-Factor Authentication, il n’est pas vérifié automatiquement à l’issue du délai en jours spécifiés dans **Mémoriser Multi-Factor Authentication**. Azure AD demande une nouvelle authentification multifacteur, mais AD FS renvoie un jeton avec la revendication Multi-Factor Authentication et la date associée, au lieu d’effectuer de nouveau l’authentification multifacteur. **Cette réaction déclenche une boucle de vérification entre Azure AD et AD FS**.
>
> La fonctionnalité **Mémoriser Multi-Factor Authentication** n’est pas compatible avec les utilisateurs B2B, et n’est pas visible pour ceux-ci lors de la connexion aux locataires invités.
>

### <a name="enable-remember-multi-factor-authentication"></a>Activer Mémoriser Multi-Factor Authentication

Pour activer et configurer l’option permettant aux utilisateurs de mémoriser leur état MFA et de contourner les invites, effectuez les étapes suivantes :

1. Dans le portail Azure, recherchez et sélectionnez **Azure Active Directory**, puis choisissez **Utilisateurs**.
1. Sélectionnez **Multi-Factor Authentication**.
1. Sous Multi-Factor Authentication, sélectionnez **Paramètres du service**.
1. Dans la page **Paramètres du service**, sous **Mémoriser Multi-Factor Authentication**, sélectionnez l’option **Permettre aux utilisateurs de mémoriser l’authentification multifacteur sur des appareils de confiance**.
1. Définissez le nombre de jours pendant lesquels les appareils approuvés peuvent contourner l’authentification multifacteur. Pour une expérience utilisateur optimale, étendez la durée à *90* jours ou plus.
1. Sélectionnez **Enregistrer**.

### <a name="mark-a-device-as-trusted"></a>Marquer un appareil en tant qu’appareil de confiance

Après avoir activé la fonctionnalité de mémorisation de Multi-Factor Authentication, les utilisateurs peuvent marquer un appareil en tant qu’appareil de confiance en sélectionnant l’option **Ne plus me demander**.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les méthodes disponibles dans Azure AD Multi-Factor Authentication, consultez [Quelles sont les méthodes d'authentification et de vérification disponibles dans Azure Active Directory ?](concept-authentication-methods.md)
