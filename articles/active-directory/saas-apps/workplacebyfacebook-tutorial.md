---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Workplace by Facebook | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Workplace by Facebook.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/03/2020
ms.author: jeedes
ms.openlocfilehash: f20849e4085adcca909d990a2ae9fb7b9751220b
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92520096"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-workplace-by-facebook"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Workplace by Facebook

Dans ce tutoriel, vous allez apprendre à intégrer Workplace by Facebook à Azure Active Directory (Azure AD). Quand vous intégrez Workplace by Facebook à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Workplace by Facebook
* Autoriser les utilisateurs à se connecter automatiquement à Workplace by Facebook avec leur compte Azure AD
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Abonnement Workplace by Facebook pour lequel l’authentification unique (SSO) est activée.

> [!NOTE]
> Facebook a deux produits, Espace de travail Standard (gratuit) et Espace de travail Premium (payant). N’importe quel abonné de l’Espace de travail Premium peut configurer l’intégration SCIM et SSO sans autre implication sur les coûts ou les licences requises. L’authentification unique et SCIM ne sont pas disponibles dans les instances de l’Espace de travail Standard.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Workplace by Facebook prend en charge l’authentification unique initiée par **SP**
* Workplace by Facebook prend en charge **l’approvisionnement juste-à-temps**
* Workplace by Facebook prend en charge **[l’approvisionnement automatique d’utilisateurs](workplacebyfacebook-provisioning-tutorial.md)**
* L’application mobile Workplace by Facebook peut désormais être configurée avec Azure AD pour activer l’authentification unique. Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.
* Après avoir configuré Workplace by Facebook, vous pouvez appliquer le contrôle de session, qui protège l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrir comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad)

## <a name="adding-workplace-by-facebook-from-the-gallery"></a>Ajout de Workplace by Facebook à partir de la galerie

Pour configurer l’intégration de Workplace by Facebook à Azure AD, vous devez ajouter Workplace by Facebook à partir de la galerie à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Workplace by Facebook** dans la zone de recherche.
1. Sélectionnez **Workplace by Facebook** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-sso-for-workplace-by-facebook"></a>Configurer et tester l’authentification unique Azure AD pour Workplace by Facebook

Configurez et testez l’authentification unique Azure AD avec Workplace by Facebook à l’aide d’un utilisateur de test nommé **B. Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Workplace by Facebook associé.

Pour configurer et tester l’authentification unique Azure AD avec Workplace by Facebook, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
2. **[Configurer Workplace by Facebook](#configure-workplace-by-facebook-sso)** pour configurer les paramètres de l’authentification unique côté application.
    * **[Créer un utilisateur de test Workplace by Facebook](#create-workplace-by-facebook-test-user)** pour avoir dans Workplace by Facebook un équivalent de B. Simon lié à la représentation Azure AD associée.
3. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Workplace by Facebook** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet pour **Configuration SAML de base** afin de modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **URL de connexion** , saisissez une URL au format suivant : `https://<instancename>.facebook.com`

    b. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL au format suivant : `https://www.facebook.com/company/<instanceID>`

    c. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://www.facebook.com/company/<instanceID>`

    > [!NOTE]
    > Il ne s’agit pas des valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion, l’identificateur et l’URL de réponse réels. Pour connaître les valeurs correctes pour votre communauté Workplace, consultez la page d’authentification du tableau de bord Entreprise de Workplace. Ceci est expliqué plus loin dans le tutoriel.

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (Base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer Workplace by Facebook** , copiez la ou les URL appropriées correspondant à vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé B. Simon dans le portail Azure.

1. Dans le volet gauche du Portail Azure, sélectionnez **Azure Active Directory** , **Utilisateurs** , puis **Tous les utilisateurs** .
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur** , effectuez les étapes suivantes :
   1. Dans le champ **Nom** , entrez `B.Simon`.  
   1. Dans le champ **Nom de l’utilisateur** , entrez username@companydomain.extension. Par exemple : `B.Simon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ **Mot de passe** .
   1. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser B. Simon à utiliser l’authentification unique Azure en accordant l’accès à Workplace by Facebook.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Workplace by Facebook** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-workplace-by-facebook-sso"></a>Configurer l’authentification unique Workplace by Facebook

1. Pour automatiser la configuration dans Workplace by Facebook, vous devez installer l’ **extension de navigateur My Apps Secure Sign-in** en cliquant sur **Installer l’extension** .

    ![Extension My apps](common/install-myappssecure-extension.png)

1. Après avoir ajouté l’extension au navigateur, cliquez sur **Configurer Workplace by Facebook** pour être dirigé vers l’application Workplace by Facebook. À partir de là, indiquez les informations d’identification de l’administrateur pour vous connecter à Workplace by Facebook. Cette extension de navigateur configure automatiquement l’application et automatise les étapes 3 à 5.

    ![Configuration](common/setup-sso.png)

1. Si vous souhaitez configurer manuellement Workplace by Facebook, ouvrez une nouvelle fenêtre de navigateur web, connectez-vous à votre site d’entreprise Workplace by Facebook en tant qu’administrateur et effectuez les étapes suivantes :

    > [!NOTE]
    > Dans le cadre du processus d’authentification SAML, Workplace peut utiliser des chaînes de requête d’une taille pouvant aller jusqu’à 2,5 Ko pour transmettre des paramètres à Azure AD.

1. Dans le volet de navigation de gauche, accédez à **Security** > **Authentication** (Sécurité>Authentification).

    ![Panneau d’administration](./media/workplacebyfacebook-tutorial/tutorial-workplace-by-facebook-configure01.png)

    a. Activez l’option **Single-sign on(SSO)** (Authentification unique).
    
    b. Cliquez sur **+Add new SSO Provider** (Ajouter un nouveau fournisseur d’authentification unique).
    > [!NOTE]
    > Veillez également à cocher la case Password (Mot de passe). Lorsqu’ils effectuent la substitution du certificat, les administrateurs peuvent avoir besoin de cette option pour continuer d’avoir accès à Workplace.

1. Sous l’onglet **Authentication** , sélectionnez **Single-Sign On (SSO)** (Authentification unique) et procédez comme suit :

    ![Onglet d’authentification](./media/workplacebyfacebook-tutorial/tutorial-workplace-by-facebook-configure02.png)

    a. Dans **Name of the SSO Provider** (Nom du fournisseur d’authentification unique), entrez le nom de l’instance d’authentification unique, par exemple Azureadsso.

    b. Dans la zone de texte **URL SAML** , collez la valeur **URL de connexion** que vous avez copiée dans le portail Azure.

    c. Dans la zone de texte **SAML Issuer URL** (URL de l’émetteur SAML), collez la valeur de l’ **identifiant Azure AD** que vous avez copiée à partir du portail Azure.

    d. Ouvrez dans le Bloc-notes votre **certificat codé en base 64** , téléchargé à partir du portail Azure, copiez son contenu dans le Presse-papiers, puis collez-le dans la zone de texte **SAML Certificate** (Certificat SAML).

    e. Copiez **Audience URL** pour votre instance et collez-la dans la zone de texte **Identificateur (ID d’entité)** de la section **Configuration SAML de base** du portail Azure.

    f. Copiez **Recipient URL** (URL de destinataire) pour votre instance et collez-la dans la zone de texte **URL d’authentification** de la section **Configuration SAML de base** du portail Azure.

    g. Copiez **ACS (Assertion Consumer Service) URL** pour votre instance et collez cette URL dans la zone de texte **URL de réponse** de la section **Configuration SAML de base** du portail Azure.

    h. Faites défiler l’affichage jusqu’au bas de la section et cliquez sur le bouton **Test SSO** . Une fenêtre contextuelle apparaît, avec la page de connexion Azure AD. Entrez normalement vos informations d’identification pour vous authentifier.

    **Résolution des problèmes :** Vérifiez que l’adresse e-mail retournée par Azure AD est identique au compte Workplace avec lequel vous êtes connecté.

    i. Une fois le test terminé, faites défiler jusqu’au bas de la page et cliquez sur le bouton **Save** (Enregistrer).

    j. La page de connexion à Azure AD sera désormais présentée à tous les utilisateurs de Workplace pour qu’ils s’authentifient.

1. **Redirection de déconnexion SAML (facultatif)**  -

    Vous pouvez choisir de configurer une URL de déconnexion SAML, qui peut être utilisée pour pointer vers la page de déconnexion d’Azure AD. Quand ce paramètre est activé et configuré, l’utilisateur n’est plus dirigé vers la page de déconnexion de Workplace. Au lieu de cela, il est redirigé vers l’URL qui a été ajoutée dans le paramètre SAML Logout Redirect.

### <a name="configuring-reauthentication-frequency"></a>Configuration de la fréquence de réauthentification

Vous pouvez configurer Workplace pour demander une vérification SAML chaque jour, tous les trois jours, toutes les deux semaines, tous les mois ou jamais.

> [!NOTE]
> La valeur minimale pour la vérification SAML sur les applications mobiles est toutes les semaines.

Vous pouvez également forcer une réinitialisation SAML pour tous les utilisateurs en utilisant ce bouton : Require SAML authentication for all users now (Demander une authentification SAML pour tous les utilisateurs).

### <a name="create-workplace-by-facebook-test-user"></a>Créer un utilisateur de test Workplace by Facebook

Dans cette section, un utilisateur appelé B. Simon est créé dans Workplace by Facebook. Workplace by Facebook prend en charge l’approvisionnement juste-à-temps, qui est activé par défaut.

Vous n’avez aucune opération à effectuer dans cette section. S’il n’existe pas d’utilisateur dans Workplace by Facebook, un nouvel utilisateur est créé quand vous tentez d’accéder à Workplace by Facebook.

>[!Note]
>Si vous avez besoin de créer un utilisateur manuellement, contactez l’[équipe de prise en charge des clients Workplace by Facebook](https://www.workplace.com/help/work/).

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette Workplace by Facebook dans le panneau d’accès doit vous connecter automatiquement à l’application Workplace by Facebook pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="test-sso-for-workplace-by-facebook-mobile"></a>Tester l’authentification unique pour Workplace by Facebook (mobile)

1. Ouvrez l’application mobile Workplace by Facebook. Dans la page de connexion, cliquez sur **LOG IN** .

    ![Connexion](./media/workplacebyfacebook-tutorial/test05.png)

2. Entrez votre adresse e-mail professionnelle, puis cliquez sur **CONTINUE** .

    ![L’e-mail](./media/workplacebyfacebook-tutorial/test02.png)

3. Cliquez sur **JUST ONCE** .

    ![L’unique fois](./media/workplacebyfacebook-tutorial/test04.png)

4. Cliquez sur **Autoriser** .

    ![L’autorisation](./media/workplacebyfacebook-tutorial/test03.png)

5. Une fois la connexion réussie, la page d’accueil de l’application s’affiche.    

    ![Page d’accueil](./media/workplacebyfacebook-tutorial/test01.png)

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Configurer l’approvisionnement de l’utilisateur](workplacebyfacebook-provisioning-tutorial.md)

- [Essayer Workplace by Facebook avec Azure AD](https://aad.portal.azure.com)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)