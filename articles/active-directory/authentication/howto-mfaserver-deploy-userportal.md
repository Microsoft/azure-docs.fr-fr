---
title: Portail utilisateur pour Azure MFA Server - Azure Active Directory
description: Prise en main du serveur Azure Multi-Factor Authentication et du portail de l’utilisateur.
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 07/11/2018
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.openlocfilehash: 61cefacc4147ab1eb105af9cb21365129dd1950e
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96742457"
---
# <a name="user-portal-for-the-azure-multi-factor-authentication-server"></a>Portail de l’utilisateur pour le serveur Azure Multi-Factor Authentication

Le portail de l’utilisateur est un site web IIS qui permet aux utilisateurs de s’inscrire dans Azure Multi-Factor Authentication (MFA) et de gérer leurs comptes. L’utilisateur peut modifier son numéro de téléphone ou son code PIN, ou décider de contourner Azure Multi-Factor Authentication lors de sa prochaine connexion.

Les utilisateurs se connectent au portail de l’utilisateur avec leur nom d’utilisateur et leur mot de passe, puis effectuent soit un appel de vérification en deux étapes soit répondent à des questions de sécurité pour s’authentifier. Si l’inscription est autorisée, les utilisateurs configurent leur numéro de téléphone et leur code PIN lors de leur première connexion au portail de l’utilisateur.

Les administrateurs du portail de l’utilisateur peuvent être configurés et autorisés à ajouter de nouveaux utilisateurs et à mettre à jour les utilisateurs existants.

Selon votre environnement, il peut être nécessaire de déployer le portail de l’utilisateur sur le même serveur que le serveur Azure Multi-Factor Authentication ou sur un autre serveur accessible sur Internet.

> [!IMPORTANT]
> Depuis le 1er juillet 2019, Microsoft ne propose plus MFA Server pour les nouveaux déploiements. Les nouveaux clients qui souhaitent imposer une authentification multifacteur lors des événements de connexion doivent utiliser la fonctionnalité d’authentification multifacteur Azure AD basée sur le cloud.
>
> Pour commencer à utiliser l’authentification multifacteur basée sur le cloud, consultez [Tutoriel : Événements de connexion utilisateur sécurisés avec l’authentification multifacteur Azure AD](tutorial-enable-azure-mfa.md).
>
> Les clients existants qui ont activé le serveur MFA avant le 1er juillet 2019 peuvent télécharger la dernière version, les futures mises à jour et générer des informations d’identification d’activation comme d’habitude.

![Page de connexion du portail utilisateur de MFA Server](./media/howto-mfaserver-deploy-userportal/portal.png)

> [!NOTE]
> Le portail de l’utilisateur est disponible uniquement avec le serveur Multi-Factor Authentication. Si vous utilisez Multi-Factor Authentication dans le cloud, invitez vos utilisateurs à se reporter aux articles [Configurer le compte pour la vérification en deux étapes](../user-help/multi-factor-authentication-end-user-first-time.md) ou [Gérer les paramètres de la vérification en deux étapes](../user-help/multi-factor-authentication-end-user-manage-settings.md).

## <a name="install-the-web-service-sdk"></a>Installation du Kit de développement logiciel (SDK) du service web

Quel que soit le scénario, si le Kit de développement logiciel (SDK) Azure Multi-Factor Authentication Web Service **n’est pas déjà installé** sur le serveur Azure Multi-Factor Authentication (MFA), effectuez les étapes suivantes.

1. Ouvrez la console du serveur Azure Multi-Factor Authentication.
2. Accédez au **Kit de développement logiciel (SDK) du service Web**, puis cliquez sur **Installer le kit de développement logiciel du service Web**.
3. À moins que vous n’ayez besoin de les modifier, terminez l’installation à l’aide des paramètres par défaut.
4. Liez un certificat TLS/SSL personnalisé au site dans IIS.

Si vous avez des questions à propos de la configuration du certificat TLS/SSL personnalisé sur un serveur IIS, consultez l’article [Comment configurer SSL sur IIS](/iis/manage/configuring-security/how-to-set-up-ssl-on-iis).

Le Kit de développement logiciel (SDK) Web Service doit être sécurisé avec un certificat TLS/SSL. Un certificat auto-signé peut être ajouté à cet effet. Importez le certificat dans le magasin « Autorités de certification racines de confiance » du compte Ordinateur local sur le serveur web du portail de l’utilisateur, afin qu’il approuve ce certificat lors de l’initialisation de la connexion TLS.

![Kit de développement logiciel (SDK) du Service Web d’installation du serveur MFA](./media/howto-mfaserver-deploy-userportal/sdk.png)

## <a name="deploy-the-user-portal-on-the-same-server-as-the-azure-multi-factor-authentication-server"></a>Déploiement du portail de l’utilisateur sur le même serveur qu’Azure Multi-Factor Authentication

La configuration suivante est requise pour installer le portail de l’utilisateur sur le **même serveur** qu’Azure Multi-Factor Authentication :

* IIS, avec ASP.net et IIS 6 avec compatibilité de la métabase (pour IIS 7 ou version ultérieure)
* Un compte avec des droits d’administrateur sur l’ordinateur et le domaine le cas échéant. Le compte doit être autorisé à créer des groupes de sécurité Active Directory.
* Sécurisez le portail de l’utilisateur avec un certificat TLS/SSL.
* Sécurisez le Kit de développement logiciel (SDK) du service web Azure Multi-Factor Authentication avec un certificat TLS/SSL.

Pour déployer le portail de l’utilisateur, procédez comme suit :

1. Ouvrez le serveur Azure Multi-Factor Authentication, cliquez sur l’icône **Portail de l’utilisateur** dans le menu de gauche, puis cliquez sur **Installer le portail de l’utilisateur**.
2. À moins que vous n’ayez besoin de les modifier, terminez l’installation à l’aide des paramètres par défaut.
3. Lier un certificat TLS/SSL personnalisé au site dans IIS

   > [!NOTE]
   > Le certificat TLS/SSL est généralement un certificat TLS/SSL signé publiquement.

4. Ouvrez un navigateur web à partir de n’importe quel ordinateur et accédez à l’URL où le portail de l’utilisateur a été installé (par exemple, `https://mfa.contoso.com/MultiFactorAuth`). Assurez-vous qu'aucun avertissement ou erreur de certificat ne soit affiché.

![Installation du portail de l’utilisateur du serveur MFA](./media/howto-mfaserver-deploy-userportal/install.png)

Si vous avez des questions à propos de la configuration du certificat TLS/SSL personnalisé sur un serveur IIS, consultez l’article [Comment configurer SSL sur IIS](/iis/manage/configuring-security/how-to-set-up-ssl-on-iis).

## <a name="deploy-the-user-portal-on-a-separate-server"></a>Déploiement du portail de l’utilisateur sur un serveur distinct

Si le serveur sur lequel le serveur Azure Multi-Factor Authentication est exécuté n’est pas orienté Internet, vous devez installer le portail de l’utilisateur sur un **serveur accessible sur Internet distinct**.

Si votre organisation utilise l’application Microsoft Authenticator comme méthode de vérification et que vous souhaitez déployer le portail de l’utilisateur sur son propre serveur, respectez la configuration requise suivante :

* Utilisez la version 6.0 ou ultérieure du serveur Azure Multi-Factor Authentication.
* Installez le portail de l’utilisateur sur un serveur web orienté Internet exécutant Microsoft Internet Information Services (IIS) 6.x ou une version ultérieure.
* Lorsque vous utilisez IIS 6.x, vérifiez qu’ASP.NET v2.0.50727 est installé, inscrit et **autorisé**.
* Lorsque vous utilisez IIS 7.x ou une version ultérieure, IIS, y compris l’authentification de base, ASP.NET et IIS 6 avec compatibilité de la métabase.
* Sécurisez le portail de l’utilisateur avec un certificat TLS/SSL.
* Sécurisez le Kit de développement logiciel (SDK) du service web Azure Multi-Factor Authentication avec un certificat TLS/SSL.
* Vérifiez que le portail de l’utilisateur peut se connecter au Kit de développement logiciel (SDK) du service web Azure Multi-Factor Authentication via TLS/SSL.
* Vérifiez que le portail de l’utilisateur peut s’authentifier auprès du Kit de développement logiciel (SDK) du service web Azure Multi-Factor Authentication à l’aide des informations d’identification d’un compte de service membre du groupe de sécurité « PhoneFactor Admins ». Ce compte et ce groupe de service devraient exister dans Active Directory si le serveur Azure Multi-Factor Authentication s’exécute sur un serveur appartenant à un domaine. Ce compte et ce groupe de service existent localement sur le serveur Azure Multi-Factor Authentication s'il n'est pas joint à un domaine.

Une installation du portail de l'utilisateur sur un serveur autre que le serveur Azure Multi-Factor Authentication nécessite les étapes suivantes :

1. **Sur le serveur MFA**, parcourez le chemin d’installation (par exemple : C:\Program Files\Multi-Factor Authentication Server) et copiez le fichier **MultiFactorAuthenticationUserPortalSetup64** dans un emplacement accessible au serveur qui est accessible sur Internet et sur lequel vous l’installerez.
2. **Sur le serveur web accessible sur Internet**, exécutez le fichier d’installation MultiFactorAuthenticationUserPortalSetup64 en tant qu’administrateur, modifiez le site ou modifiez le répertoire virtuel pour un nom court si vous le souhaitez.
3. Liez un certificat TLS/SSL personnalisé au site dans IIS.

   > [!NOTE]
   > Le certificat TLS/SSL est généralement un certificat TLS/SSL signé publiquement.

4. Accédez à **C:\inetpub\wwwroot\MultiFactorAuth**
5. Modifiez le fichier config Web dans le Bloc-notes

    * Recherchez la clé **"USE_WEB_SERVICE_SDK"** et modifiez **value="false"** en **value="true"**
    * Recherchez la clé **"WEB_SERVICE_SDK_AUTHENTICATION_USERNAME"** et modifiez **value=""** en **value="DOMAIN\User"** où DOMAIN\User est un compte de service faisant partie du groupe « PhoneFactor Admins ».
    * Recherchez la clé **"WEB_SERVICE_SDK_AUTHENTICATION_PASSWORD"** et modifiez **value=""** en **value="Password"** où Password est le mot de passe pour le compte de service entré dans la ligne précédente.
    * Recherchez la valeur `https://www.contoso.com/MultiFactorAuthWebServiceSdk/PfWsSdk.asmx` et modifiez cette URL d’espace réservé par l’URL du Kit de développement logiciel (SDK) de service web que nous avons installé à l’étape 2.
    * Enregistrez le fichier config Web et fermez le Bloc-notes.

6. Ouvrez un navigateur web à partir de n’importe quel ordinateur et accédez à l’URL où le portail de l’utilisateur a été installé (par exemple, `https://mfa.contoso.com/MultiFactorAuth`). Assurez-vous qu'aucun avertissement ou erreur de certificat ne soit affiché.

Si vous avez des questions à propos de la configuration du certificat TLS/SSL personnalisé sur un serveur IIS, consultez l’article [Comment configurer SSL sur IIS](/iis/manage/configuring-security/how-to-set-up-ssl-on-iis).

## <a name="configure-user-portal-settings-in-the-azure-multi-factor-authentication-server"></a>Configuration les paramètres du portail de l’utilisateur dans le serveur Azure Multi-Factor Authentication

Maintenant que le portail de l’utilisateur est installé, vous devez configurer le serveur Azure Multi-Factor Authentication pour qu’il travaille avec ce dernier.

1. Dans la console du serveur Azure Multi-Factor Authentication, cliquez sur l’icône **Portail de l’utilisateur**. Dans l’onglet Paramètres, saisissez l’URL qui mène au portail de l’utilisateur dans la zone de texte **URL du portail de l’utilisateur**. Si la fonctionnalité de courrier électronique a été activée, cette URL est insérée dans les courriers électroniques qui sont envoyés aux utilisateurs lorsqu’ils sont importés dans le serveur Azure Multi-Factor Authentication.
2. Choisissez les paramètres que vous souhaitez utiliser dans le portail de l'utilisateur. Par exemple, si les utilisateurs sont autorisés à choisir leurs méthodes d’authentification, vérifiez que l’option **Autoriser les utilisateurs à sélectionner la méthode** est activée, ainsi que les méthodes qu’ils peuvent choisir.
3. Définissez les administrateurs dans l’onglet **Administrateurs**. Vous pouvez créer des autorisations administratives granulaires à l’aide des cases à cocher et des menus déroulants dans les zones Ajouter/Modifier.

Configuration facultative :

- **Questions de sécurité** : définissez les questions de sécurité approuvées pour votre environnement et la langue dans laquelle elles apparaissent.
- **Sessions précédentes** : configurez l’intégration du portail utilisateur avec un site Web basé sur le formulaire à l’aide de MFA.
- **Adresses IP de confiance** : autorisez les utilisateurs à ignorer MFA lorsque vous vous authentifiez à partir d’une liste d’adresses IPs et de plages de confiance.

![Configuration du portail de l’utilisateur du serveur MFA](./media/howto-mfaserver-deploy-userportal/config.png)

Le serveur Azure Multi-Factor Authentication fournit plusieurs options pour le portail de l'utilisateur. Le tableau suivant fournit une liste de ces options, ainsi qu’une explication de leur utilisation.

| Paramètres du portail de l'utilisateur | Description |
|:--- |:--- |
| URL du portail de l'utilisateur | Entrez l’URL hébergeant le portail. |
| Authentification principale | Indiquez le type d’authentification à utiliser lors de la connexion au portail. Authentification Windows, Radius ou LDAP. |
| Autoriser les utilisateurs à se connecter | Permet aux utilisateurs d’entrer un nom d’utilisateur et un mot de passe dans la page de connexion au portail de l’utilisateur. Si cette option n’est pas sélectionnée, les zones sont grisées. |
| Autoriser l'inscription utilisateur | Permet à l’utilisateur de s’inscrire dans Multi-Factor Authentication grâce à un écran de configuration qui l’invite à entrer des informations supplémentaires, comme son numéro de téléphone. L'invite pour saisir un téléphone de secours permet aux utilisateurs de spécifier un numéro de téléphone secondaire. L’invite pour saisir un jeton OATH tiers permet aux utilisateurs de spécifier un jeton OATH tiers. |
| Autoriser les utilisateurs à lancer le contournement à usage unique | Autorise les utilisateurs à lancer un contournement à usage unique. Si un utilisateur configure cette option, celle-ci prendra effet lors de la prochaine connexion de celui-ci. L'invite à saisir les secondes du contournement affiche une zone qui permet à l'utilisateur de modifier la valeur par défaut de 300 secondes. Dans le cas contraire, le contournement à usage unique n'est valable que 300 secondes. |
| Autoriser les utilisateurs à sélectionner la méthode | Permet aux utilisateurs de spécifier leur méthode de contact principale. Cela peut être un appel téléphonique, un message texte, une application mobile ou un jeton OATH. |
| Autoriser les utilisateurs à sélectionner la langue | Permet aux utilisateurs de modifier la langue utilisée pour l’appel téléphonique, le message texte, l’application mobile ou le jeton OATH. |
| Autoriser les utilisateurs à activer l'application mobile | Permet aux utilisateurs de générer un code d’activation pour terminer le processus d’activation de l’application mobile, utilisé avec le serveur.  Vous pouvez également définir le nombre d’appareils (entre 1 et 10) sur lesquels les utilisateurs peut activer l’application. |
| Utiliser les questions de sécurité de secours | Autorise les questions de sécurité en cas d’échec de la vérification en deux étapes. Vous pouvez spécifier le nombre de questions de sécurité qui doivent être traitées avec succès. |
| Autoriser les utilisateurs à associer un jeton OATH tiers | Permet aux utilisateurs de spécifier un jeton OATH tiers. |
| Utiliser le jeton OATH de secours | Permet d’utiliser un jeton OATH si la vérification en deux étapes échoue. Vous pouvez également spécifier le délai d'expiration de la session en minutes. |
| Activation de la journalisation | Active la journalisation sur le portail de l’utilisateur. Les fichiers journaux se trouvent à l’emplacement suivant : C:\Program Files\Multi-Factor Authentication Server\Logs. |

> [!IMPORTANT]
> Depuis mars 2019, les options d’appel téléphonique ne sont plus disponibles pour les utilisateurs de MFA Server dans des locataires Azure AD gratuits ou à l’essai. Cette modification n’affecte pas les messages SMS. Les appels téléphoniques continueront à être disponibles pour les utilisateurs de locataires Azure AD payants. Ce changement affecte uniquement les locataires Azure AD gratuits ou à l’essai.

L’utilisateur peut voir ces paramètres dès qu’ils sont activés et connectés au portail de l’utilisateur.

![Gérer votre compte MFA Server par l’intermédiaire du portail utilisateur](./media/howto-mfaserver-deploy-userportal/portalsettings.png)

### <a name="self-service-user-enrollment"></a>Inscription utilisateur libre-service

Si vous souhaitez que vos utilisateurs se connectent et s’inscrivent, vous devez sélectionner les options **Autoriser la connexion des utilisateurs** et **Autoriser l’inscription utilisateur** dans l’onglet Paramètres. N’oubliez pas que les paramètres sélectionnés modifieront l’expérience de connexion des utilisateurs.

Par exemple, lorsqu’un utilisateur se connecte au portail de l’utilisateur pour la première fois, il est dirigé vers la page de configuration d’utilisateur d’Azure Multi-Factor Authentication. Selon la façon dont vous avez configuré Azure Multi-Factor Authentication, l'utilisateur peut être en mesure de choisir sa méthode d'authentification.

S’il sélectionne la méthode d’authentification Appel vocal ou si cette méthode a été préconfigurée, la page invite l’utilisateur à entrer son numéro de téléphone principal et son numéro de poste le cas échéant. Il peut également être autorisé à entrer un numéro de téléphone de secours.

![Inscrire des numéros de téléphone principal et de sauvegarde](./media/howto-mfaserver-deploy-userportal/backupphone.png)

Si l’utilisateur doit utiliser un code PIN pour s’authentifier, la page l’invite également à créer un code PIN. Après avoir entré son ou ses numéros de téléphone et son code PIN (le cas échéant), l’utilisateur clique sur le bouton **M’appeler maintenant pour m’authentifier**. Azure Multi-Factor Authentication vérifie le numéro de téléphone principal indiqué en appelant l’utilisateur. L'utilisateur doit répondre à l'appel, entrer son code PIN (le cas échéant) et appuyer sur la touche # pour passer à l'étape suivante du processus d'inscription automatique.

Si l’utilisateur sélectionne la méthode d’authentification par SMS ou si cette méthode a été préconfigurée, la page invite l’utilisateur à saisir son numéro de téléphone mobile. Si l’utilisateur doit utiliser un code PIN pour s’authentifier, la page l’invite également à entrer un code PIN.  Après avoir entré son numéro de téléphone et son code PIN (le cas échéant), l’utilisateur clique sur le bouton **M’envoyer un SMS maintenant pour m’authentifier**. Azure Multi-Factor Authentication envoie un SMS au téléphone mobile de l’utilisateur. L’utilisateur reçoit le SMS avec un code secret à usage unique, puis répond au message en renvoyant ce code et son code PIN (le cas échéant).

![Vérification du portail utilisateur au moyen de SMS](./media/howto-mfaserver-deploy-userportal/text.png)

Si l’utilisateur sélectionne la méthode de vérification Mobile App, la page invite l’utilisateur à installer l’application Microsoft Authenticator sur son appareil et à générer un code d’activation. Une fois l’application installée, l’utilisateur clique sur le bouton Générer le code d’activation.

> [!NOTE]
> Pour utiliser l’application Microsoft Authenticator, l’utilisateur doit activer les notifications Push sur son appareil.

La page affiche alors un code d'activation et une URL, ainsi qu'une image de code-barres. Si l’utilisateur doit utiliser un code PIN pour s’authentifier, la page l’invite également à entrer un code PIN. L’utilisateur saisit le code d’activation et l’URL dans l’application Microsoft Authenticator ou utilise le scanneur de codes-barres pour numériser l’image du code-barres, puis il clique sur le bouton Activer.

Une fois l’activation terminée, l’utilisateur clique sur le bouton **Authenticate Me Now (M’authentifier maintenant)** . Azure Multi-Factor Authentication vérifie l’application mobile de l’utilisateur. L'utilisateur doit entrer son code PIN (le cas échéant) et appuyer sur le bouton Authentifier dans l'application mobile pour passer à l'étape suivante du processus d'inscription automatique.

Si les administrateurs ont configuré le serveur Azure Multi-Factor Authentication pour collecter les questions de sécurité et les réponses, l'utilisateur est alors dirigé vers la page Questions de sécurité. L'utilisateur doit sélectionner quatre questions de sécurité et y répondre.

![Questions de sécurité du portail de l'utilisateur](./media/howto-mfaserver-deploy-userportal/secq.png)

L’inscription automatique de l’utilisateur est à présent terminée et l’utilisateur est connecté au portail de l’utilisateur. Les utilisateurs peuvent se reconnecter au portail de l’utilisateur à tout moment pour modifier leurs numéros de téléphone, leur code PIN, leurs méthodes d’authentification et leurs questions de sécurité, si les administrateurs les y autorisent.

## <a name="next-steps"></a>Étapes suivantes

- [Déployer le service web de l’application mobile du serveur Azure Multi-Factor Authentication](howto-mfaserver-deploy-mobileapp.md)