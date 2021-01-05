---
title: Tutoriel pour configurer Azure Active Directory B2C avec Nevis
titleSuffix: Azure AD B2C
description: Découvrez comment intégrer l’authentification Azure AD B2C avec Nevis pour une authentification sans mot de passe
services: active-directory-b2c
author: gargi-sinha
manager: martinco
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 11/23/2020
ms.author: gasinh
ms.subservice: B2C
ms.openlocfilehash: 19aa847ce7ce1a6ba727a4733aefcdfad845e8c6
ms.sourcegitcommit: 8c3a656f82aa6f9c2792a27b02bbaa634786f42d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97629389"
---
# <a name="tutorial-to-configure-nevis-with-azure-active-directory-b2c-for-passwordless-authentication"></a>Tutoriel pour configurer Nevis avec Azure Active Directory B2C pour une authentification sans mot de passe

Dans cet exemple de tutoriel, découvrez comment étendre Azure AD B2C avec [Nevis](https://www.nevis.net/solution/authentication-cloud) pour activer l’authentification sans mot de passe. Nevis offre une expérience utilisateur axée sur l’interface mobile, qui est entièrement personnalisée en fonction de votre marque. En combinaison avec l’application Nevis Access, Nevis fournit une authentification forte du client et garantit le respect des obligations relatives aux transactions PSD2 (Payment Services Directive 2).

## <a name="prerequisites"></a>Prérequis

Avant de commencer, vérifiez que vous disposez des éléments suivants :

- Un [compte d’essai](https://www.nevis-security.com/aadb2c/) Nevis

- Un abonnement Azure AD Si vous n’en avez pas, créez un [compte gratuit](https://azure.microsoft.com/free/).

- Un [locataire Azure AD B2C](https://docs.microsoft.com/azure/active-directory-b2c/tutorial-create-tenant) lié à votre abonnement Azure.

- Un environnement Azure AD B2C configuré pour utiliser des [stratégies personnalisées](https://docs.microsoft.com/azure/active-directory-b2c/custom-policy-get-started), si vous souhaitez intégrer Nevis dans votre flux de stratégie d’inscription.

## <a name="scenario-description"></a>Description du scénario

Dans ce scénario, ajoutez une application d’accès entièrement personnalisée en fonction de votre marque à votre application back-end pour offrir une authentification sans mot de passe. La solution se compose des éléments suivants :

- Locataire Azure AD B2C, avec une stratégie de connexion et d’inscription combinée pour votre back-end
- Instance de Nevis avec son API REST pour améliorer Azure AD B2C
- Votre propre application d’accès personnalisée

Le diagramme illustre l’implémentation.

![Présentation générale du flux de connexion par mot de passe avec Azure AD B2C et Nevis](./media/partner-nevis/nevis-architecture-diagram.png)

|Étape | Description |
|:-----| :-----------|
| 1. | Un utilisateur tente de se connecter ou de s’inscrire à une application via la stratégie de connexion et d’inscription d’Azure AD B2C.
| 2. | Durant l’inscription, l’application Nevis Access est inscrite sur l’appareil de l’utilisateur à l’aide d’un code QR. Une clé privée est générée sur l’appareil de l’utilisateur et est utilisée pour signer les requêtes de l’utilisateur.
| 3. |  Azure AD B2C utilise un profil technique RESTful pour démarrer la connexion avec la solution Nevis.
| 4. | La demande de connexion est envoyée à l’application d’accès, sous forme de notification Push, de code QR ou de lien ciblé.
| 5. | L’utilisateur approuve la tentative de connexion à l’aide de sa biométrie. Un message est ensuite retourné à Nevis, qui vérifie la connexion à l’aide de la clé publique stockée.
| 6. | Azure AD B2C envoie une dernière requête à Nevis pour confirmer que la connexion a été correctement effectuée.
| 7. |En fonction du message de réussite/d’échec d’Azure AD B2C, l’utilisateur se voit octroyer ou refuser l’accès à l’application.

## <a name="integrate-your-azure-ad-b2c-tenant"></a>Intégrer votre locataire Azure AD B2C

### <a name="onboard-to-nevis"></a>Intégrer avec Nevis 

[Inscrivez-vous pour obtenir un compte Nevis](https://www.nevis-security.com/aadb2c/).
Vous allez recevoir deux e-mails :

1. Une notification de compte de gestion

2. Une invitation d’application mobile.

### <a name="add-your-azure-ad-b2c-tenant-to-your-nevis-account"></a>Ajouter votre locataire Azure AD B2C à votre compte Nevis

1. À partir de l’e-mail d’essai du compte de gestion Nevis, copiez votre clé de gestion dans le Presse-papiers.

2. Ouvrez https://console.nevis.cloud/ dans un navigateur.

3. Connectez-vous à la console de gestion à l’aide de votre clé.

4. Sélectionnez **Add Instance** (Ajouter une instance)

5. Sélectionnez l’instance créée pour ouvrir cette dernière.

6. Dans la barre de navigation latérale, sélectionnez **Custom Integrations** (Intégrations personnalisées)

7. Sélectionnez **Add custom integration** (Ajouter une intégration personnalisée).

8. Pour Integration Name (Nom de l’intégration), entrez le nom de votre locataire Azure AD B2C.

9. Pour URL/Domain (URL/Domaine), entrez `https://yourtenant.onmicrosoft.com`

10. Sélectionnez **Suivant**.

>[!NOTE]
>Vous aurez besoin du jeton d’accès Nevis plus tard.

11. Sélectionnez **Terminé**.

### <a name="install-the-nevis-access-app-on-your-phone"></a>Installer l’application Nevis Access sur votre téléphone

1. À partir de l’e-mail d’essai de l’application mobile Nevis, ouvrez l’invitation de l’**application Test Flight**.

2. Installez l’application.

3. Suivez les instructions fournies pour installer l’application Nevis Access.

### <a name="integrate-azure-ad-b2c-with-nevis"></a>Intégrer Azure AD B2C avec Nevis

1. Ouvrez le [portail Azure](https://portal.azure.com/).

2. Accédez à votre locataire Azure AD B2C. Vérifiez que vous avez sélectionné le bon locataire, car le locataire Azure AD B2C se trouve généralement dans un locataire distinct.

3. Dans le menu, sélectionnez **IEF (Identity Experience Framework)**

4. Sélectionner **Clés de stratégie**

5. Sélectionnez **Ajouter**, puis créez une clé avec les paramètres suivants :

      a. Dans Options, sélectionnez **Manuel**

      b. Affectez à Nom la valeur **AuthCloudAccessToken**

      c. Collez le **jeton d’accès Nevis** stocké dans le champ Secret

      d. Pour Utilisation de la clé, sélectionnez **Chiffrement**

      e. Sélectionnez **Créer**

### <a name="configure-and-upload-the-nevishtml-to-azure-blob-storage"></a>Configurer et charger le fichier nevis.html sur le Stockage Blob Azure

1. Dans votre IDE (environnement d’identité), accédez au dossier [**policy**](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/Nevis/policy).

2. Ouvrez le fichier [**nevis.html**](https://github.com/azure-ad-b2c/partner-integrations/blob/master/samples/Nevis/policy/nevis.html).

3. Remplacez **authentication_cloud_url** par l’URL de votre console d’administration Nevis - `https://<instance_id>.mauth.nevis.cloud`.

4. **Enregistrez** les changements apportés au fichier.

5. Suivez les [instructions](https://docs.microsoft.com/azure/active-directory-b2c/customize-ui-with-html#2-create-an-azure-blob-storage-account), puis chargez le fichier **nevis.html** dans votre Stockage Blob Azure.

6. Suivez les [instructions](https://docs.microsoft.com/azure/active-directory-b2c/customize-ui-with-html#3-configure-cors) et activez le partage CORS (partage de ressources Cross-Origin) pour ce fichier.

7. Une fois le chargement effectué et le partage CORS activé, sélectionnez le fichier **nevis.html** dans la liste.

8. Sous l’onglet **Vue d’ensemble**, à côté de **URL**, sélectionnez l’icône permettant de **copier le lien**.

9. Ouvrez le lien dans un nouvel onglet de navigateur pour vérifier qu’il affiche une zone grise.

>[!NOTE]
>Vous aurez besoin du lien d’objet blob plus tard.

### <a name="customize-your-trustframeworkbasexml"></a>Personnalisez TrustFrameworkBase.xml

1. Dans votre IDE, accédez au dossier [**policy**](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/Nevis/policy).

2. Ouvrez le fichier [**TrustFrameworkBase.xml**](https://github.com/azure-ad-b2c/partner-integrations/blob/master/samples/Nevis/policy/TrustFrameworkBase.xml).

3. Remplacez **yourtenant** par le nom de votre compte de locataire Azure dans **TenantId**.

4. Remplacez **yourtenant** par le nom de votre compte de locataire Azure dans **PublicPolicyURI**.

5. Remplacez toutes les instances de **authentication_cloud_url** par l’URL de votre console d’administration Nevis

6. **Enregistrez** les changements apportés à votre fichier.

### <a name="customize-your-trustframeworkextensionsxml"></a>Personnalisez TrustFrameworkExtensions.xml

1. Dans votre IDE, accédez au dossier [**policy**](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/Nevis/policy).

2. Ouvrez le fichier [**TrustFrameworkExtensions.xml**](https://github.com/azure-ad-b2c/partner-integrations/blob/master/samples/Nevis/policy/TrustFrameworkExtensions.xml).

3. Remplacez **yourtenant** par le nom de votre compte de locataire Azure dans **TenantId**.

4. Remplacez **yourtenant** par le nom de votre compte de locataire Azure dans **PublicPolicyURI**.

5. Sous **BasePolicy**, dans **TenantId**, remplacez également _yourtenant_ par le nom de votre compte de locataire Azure.

6. Sous **BuildingBlocks**, remplacez **LoadUri** par l’URL du lien d’objet blob de _nevis.html_ dans votre compte Stockage Blob.

7. **Enregistrez** le fichier.

### <a name="customize-your-signuporsigninxml"></a>Personnalisez SignUpOrSignin.xml

1. Dans votre IDE, accédez au dossier [**policy**](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/Nevis/policy).

2. Ouvrez le fichier [**SignUpOrSignin.xml**](https://github.com/azure-ad-b2c/partner-integrations/blob/master/samples/Nevis/policy/SignUpOrSignin.xml).

3. Remplacez **yourtenant** par le nom de votre compte de locataire Azure dans **TenantId**.

4. Remplacez **yourtenant** par le nom de votre compte de locataire Azure dans **PublicPolicyUri**.

5. Sous **BasePolicy**, dans **TenantId**, remplacez également **yourtenant** par le nom de votre compte de locataire Azure.

6. **Enregistrez** le fichier.

### <a name="upload-your-custom-policies-to-azure-ad-b2c"></a>Chargez vos stratégies personnalisées vers Azure AD B2C

1. Ouvrez la page d’accueil de votre [locataire Azure AD B2C](https://portal.azure.com/#blade/Microsoft_AAD_B2CAdmin/TenantManagementMenuBlade/overview).

2. Sélectionnez **Infrastructure d’expérience d’identité**.

3. Sélectionnez **Charger une stratégie personnalisée**.

4. Sélectionnez le fichier **TrustFrameworkBase.xml** que vous avez modifié.

5. Cochez la case **Remplacer la stratégie personnalisée si elle existe déjà**.
6. Sélectionnez **Télécharger**.

7. Répétez les étapes 5 et 6 pour **TrustFrameworkExtensions.xml**.

8. Répétez les étapes 5 et 6 pour **SignUpOrSignin.xml**.

## <a name="test-the-user-flow"></a>Tester le flux utilisateur

### <a name="test-account-creation-and-nevis-access-app-setup"></a>Tester la création de compte et la configuration de l’application Nevis Access

1. Ouvrez la page d’accueil de votre [locataire Azure AD B2C](https://portal.azure.com/#blade/Microsoft_AAD_B2CAdmin/TenantManagementMenuBlade/overview).

2. Sélectionnez **Infrastructure d’expérience d’identité**.

3. Faites défiler l’affichage jusqu’à Stratégies personnalisées, puis sélectionnez **B2C_1A_signup_signin**.

4. Sélectionnez **Exécuter maintenant**.

5. Dans la fenêtre indépendante, sélectionnez **S’inscrire maintenant**.

6. Ajoutez votre adresse e-mail.

7. Sélectionnez **Envoyer le code de vérification**.

8. Copiez le code de vérification à partir de l’e-mail.

9. Sélectionnez **Vérifier**.

10. Remplissez le formulaire avec votre nouveau mot de passe et votre nom d’affichage.

11. Sélectionnez **Create** (Créer).

12. Vous êtes redirigé vers la page d’analyse du code QR.

13. Sur votre téléphone, ouvrez l’**application Nevis Access**.

14. Sélectionnez **Face ID**.

15. Quand l’écran indique **Authenticator registration was successful** (Inscription réussie de l’authentificateur), sélectionnez **Continuer**.

16. Sur votre téléphone, réauthentifiez-vous à l’aide de votre visage.

17. Vous êtes redirigé vers la page de destination [jwt.ms](https://jwt.ms), qui affiche les détails de votre jeton décodé.

### <a name="test-the-pure-passwordless-sign-in"></a>Tester la connexion sans mot de passe pure

1. Sous **Identity Experience Framework**, sélectionnez **B2C_1A_signup_signin**.

2. Sélectionnez **Exécuter maintenant**.

3. Dans la fenêtre indépendante, sélectionnez **Authentification sans mot de passe**.

4. Saisissez votre adresse de messagerie.

5. Sélectionnez **Continuer**.

6. Sur votre téléphone, dans les notifications, sélectionnez la **notification de l’application Nevis Access**.

7. Authentifiez-vous à l’aide de votre visage.

8. Vous êtes automatiquement redirigé vers la page de destination [jwt.ms](https://jwt.ms), qui affiche vos jetons.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez les articles suivants

- [Stratégies personnalisées dans Azure AD B2C](https://docs.microsoft.com/azure/active-directory-b2c/custom-policy-overview)

- [Bien démarrer avec les stratégies personnalisées dans Azure AD B2C](https://docs.microsoft.com/azure/active-directory-b2c/custom-policy-get-started?tabs=applications)
