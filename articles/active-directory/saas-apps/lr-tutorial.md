---
title: 'Didacticiel : Intégration d’Azure Active Directory avec LoginRadius | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et LoginRadius.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/14/2019
ms.author: jeedes
ms.openlocfilehash: 832c08123904b9fb889231faa86c1308704a2581
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97606417"
---
# <a name="tutorial-azure-active-directory-integration-with-loginradius"></a>Didacticiel : Intégration d’Azure Active Directory avec LoginRadius

Dans ce didacticiel, vous allez apprendre à intégrer LoginRadius avec Azure Active Directory (Azure AD).

L’intégration de LoginRadius avec Azure AD offre les avantages suivants :

* Il est possible de contrôler dans Azure AD qui a accès à LoginRadius.
* Vous pouvez permettre à vos utilisateurs d’être connectés automatiquement à LoginRadius (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD avec LoginRadius, vous aurez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/)
* Un abonnement LoginRadius pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* LoginRadius prend en charge l’authentification unique lancée par le **fournisseur de services**

## <a name="adding-loginradius-from-the-gallery"></a>Ajout de LoginRadius à partir de la galerie

Pour configurer l’intégration de LoginRadius à Azure AD, vous devez ajouter LoginRadius à votre liste d’applications SaaS gérées à partir de la galerie.

**Pour ajouter LoginRadius à partir de la galerie :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , sélectionnez l’icône **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise**, puis sélectionnez l’option **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une nouvelle application, sélectionnez le bouton **Nouvelle application** :

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **LoginRadius**, sélectionnez **LoginRadius** dans le volet de résultats, puis sélectionnez le bouton **Ajouter** pour ajouter l’application.

    ![LoginRadius dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec LoginRadius pour un utilisateur de test nommé **Britta Simon**.
Pour que l’authentification unique fonctionne, une relation entre l’utilisateur Azure AD et l’utilisateur LoginRadius associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec LoginRadius, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique LoginRadius](#configure-loginradius-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test LoginRadius](#create-loginradius-test-user)** pour avoir un équivalent de Britta Simon dans LoginRadius qui soit lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec LoginRadius, effectuez ces étapes :

1. Dans la page d’intégration de l’application **LoginRadius** du [portail Azure](https://portal.azure.com/), sélectionnez **Authentification unique**.

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans le volet **Sélectionner une méthode d’authentification unique**, sélectionnez le mode **SAML/WS-Fed** pour activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML**, sélectionnez l’icône **Modifier** pour ouvrir le volet **Configuration SAML de base**.

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** :

   ![Informations sur l’authentification unique dans Domaine et URL LoginRadius](common/sp-identifier.png)

   1. Dans la zone de texte **URL de connexion**, entrez l’URL `https://secure.loginradius.com/login`

   1. Dans la zone de texte **Identificateur (ID d’entité)** , entrez l’URL `https://lr.hub.loginradius.com/`

   1. Dans la zone de texte **URL de réponse (URL Assertion Consumer Service)** , entrez l’URL ACS LoginRadius `https://lr.hub.loginradius.com/saml/serviceprovider/AdfsACS.aspx` 

5. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, sélectionnez **Télécharger** pour télécharger le fichier **XML de métadonnées de fédération** en fonction des options que vous avez définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

6. Dans la section **Configurer LoginRadius**, copiez la ou les URL appropriées en fonction de vos besoins.

   ![Copier les URL de configuration](common/copy-configuration-urls.png)

   - URL de connexion

   - Identificateur Azure AD

   - URL de déconnexion

## <a name="configure-loginradius-single-sign-on"></a>Configurer l’authentification unique LoginRadius

Dans cette section, vous allez activer l’authentification unique Azure AD dans la console d’administration LoginRadius.

1. Connectez-vous à votre compte LoginRadius [Admin Console](https://adminconsole.loginradius.com/login).

2. Accédez à la section **Team Management** (Gestion d’équipe) de [LoginRadius Admin Console](https://secure.loginradius.com/account/team).

3. Sélectionnez l’onglet **Single Sign-On** (Authentification unique), puis **Azure AD** :

   ![Capture d’écran montrant le menu d’authentification unique dans la console de gestion d’équipe LoginRadius](./media/loginradius-tutorial/azure-ad.png)
4. Dans la page de configuration Azure AD, effectuez les étapes suivantes :

   ![Capture d’écran montrant la configuration d’Azure Active Directory dans la console de gestion d’équipe LoginRadius](./media/loginradius-tutorial/single-sign-on.png)

    1. Dans **ID Provider Location** (Emplacement du fournisseur d’ID), entrez le point de terminaison de connexion, que vous obtenez dans votre compte Azure AD.

    1. Dans **ID Provider Logout URL** (URL de déconnexion du fournisseur d’ID), entrez le point de terminaison de déconnexion, que vous obtenez dans votre compte Azure AD.
 
    1. Dans **ID Provider Certificate** (Certificat du fournisseur d’ID), entrez le certificat Azure AD, que vous obtenez dans votre compte Azure AD. Entrez la valeur du certificat avec l’en-tête et le pied de page. Exemple : `-----BEGIN CERTIFICATE-----<certifciate value>-----END CERTIFICATE-----`

    1. Dans **Service Provider Certificate** (Certificat du fournisseur de services) et **Server Provider Certificate Key** (Clé du certificat du fournisseur de services), entrez votre certificat et votre clé. 

       Vous pouvez créer un certificat auto-signé en exécutant les commandes suivantes sur la ligne de commande (Linux/Mac) :

       - Commande pour obtenir la clé de certificat pour le fournisseur de services : `openssl genrsa -out lr.hub.loginradius.com.key 2048`

       - Commande pour obtenir le certificat pour le fournisseur de services : `openssl req -new -x509 -key lr.hub.loginradius.com.key -out lr.hub.loginradius.com.cert -days 3650 -subj /CN=lr.hub.loginradius.com`

       > [!NOTE]
       > Veillez à entrer les valeurs de certificat et de clé de certificat avec l’en-tête et le pied de page :
       > - Exemple de format de valeur de certificat : `-----BEGIN CERTIFICATE-----<certifciate value>-----END CERTIFICATE-----`
       > - Exemple de format de valeur de clé de certificat : `-----BEGIN RSA PRIVATE KEY-----<certifciate key value>-----END RSA PRIVATE KEY-----`

5. Dans la section **Data Mapping** (Mappage de données), sélectionnez les champs (champs de fournisseur de services) et entrez les champs Azure AD correspondants (champs de fournisseur d’identité).

    Voici une liste de quelques noms de champs pour Azure AD.

    | Champs    | Clé de profil                                                          |
    | --------- | -------------------------------------------------------------------- |
    | E-mail     | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress` |
    | FirstName | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname`    |
    | LastName  | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname`      |

    > [!NOTE]
    > Le champ **Email** doit être mappé. Le mappage des champs **FirstName** et **LastName** est facultatif.

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD 

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory**, sélectionnez **Utilisateurs**, puis sélectionnez **Tous les utilisateurs**.

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés **Utilisateur**, effectuez les étapes suivantes.

   ![Boîte de dialogue Utilisateur](common/user-properties.png)

   1. Dans le champ **Nom**, entrez **BrittaSimon**.
  
   1. Dans le champ **Nom de l’utilisateur**, entrez `brittasimon@yourcompanydomain.extension`. Par exemple : BrittaSimon@contoso.com.

   1. Cochez la case **Afficher le mot de passe**, puis notez la valeur affichée dans le champ **Mot de passe**.

   1. Sélectionnez **Create** (Créer).

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à LoginRadius.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, **Toutes les applications**, puis **LoginRadius**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **LoginRadius**.

    ![Lien LoginRadius dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes**.

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Sélectionnez le bouton **Ajouter un utilisateur**, puis sélectionnez **Utilisateurs et groupes** dans le volet **Ajouter une attribution**.

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans le volet **Utilisateurs et groupes**, sélectionnez **Britta Simon** dans la liste **Utilisateurs**, puis choisissez le bouton **Sélectionner** au bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans le volet **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste. Choisissez ensuite le bouton **Sélectionner** en bas de l’écran.

7. Dans le volet **Ajouter une attribution**, sélectionnez le bouton **Attribuer**.

### <a name="create-loginradius-test-user"></a>Créer un utilisateur de test LoginRadius

1. Connectez-vous à votre compte LoginRadius [Admin Console](https://adminconsole.loginradius.com/login).

2. Accédez à la section de gestion de votre équipe dans la console d’administration LoginRadius.

   ![Capture d’écran montrant la console d’administration LoginRadius](./media/loginradius-tutorial/team-management.png)
3. Sélectionnez **Add Team Member** (Ajouter un membre à l’équipe) dans le menu latéral pour ouvrir le formulaire. 

4. Dans le formulaire **Add Team Member** (Ajouter un membre à l’équipe), créez un utilisateur nommé Britta Simon sur votre site LoginRadius en fournissant les détails de l’utilisateur et en lui attribuant les autorisations souhaitées. Pour en savoir plus sur les autorisations en fonction des rôles, consultez la section [Role Access Permissions](https://www.loginradius.com/docs/api/v2/admin-console/team-management/manage-team-members#roleaccesspermissions0) du document LoginRadius relatif à la [gestion des membres d’équipe](https://www.loginradius.com/docs/api/v2/admin-console/team-management/manage-team-members#roleaccesspermissions0). Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

1. Dans un navigateur, accédez à https://accounts.loginradius.com/auth.aspx, puis sélectionnez **Fed SSO log in**.
2. Entrez le nom de votre application LoginRadius, puis sélectionnez **Login** (Connexion).
3. Une fenêtre contextuelle s’ouvre alors pour vous inviter à vous connecter à votre compte Azure AD.
4. Une fois l’authentification effectuée, la fenêtre contextuelle se ferme et vous êtes connecté à la console d’administration LoginRadius.

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)
