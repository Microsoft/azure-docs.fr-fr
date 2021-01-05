---
title: 'Tutoriel : Intégration d’Azure Active Directory à Soloinsight-CloudGate SSO | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Soloinsight-CloudGate SSO.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/06/2019
ms.author: jeedes
ms.openlocfilehash: 193d779a2e9246fe36af4828251a15d593e80ac7
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92514773"
---
# <a name="tutorial-integrate-soloinsight-cloudgate-sso-with-azure-active-directory"></a>Tutoriel : Intégrer Soloinsight-CloudGate SSO dans Azure Active Directory

Dans ce tutoriel, vous allez apprendre à intégrer Soloinsight-CloudGate SSO à Azure Active Directory (Azure AD). Lorsque vous intégrez Soloinsight-CloudGate SSO à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Soloinsight-CloudGate SSO.
* Permettre aux utilisateurs de se connecter automatiquement à Soloinsight-CloudGate SSO avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous n’en avez pas, vous pouvez obtenir un essai gratuit d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Un abonnement Soloinsight-CloudGate SSO pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test. Soloinsight-CloudGate SSO prend en charge l’authentification unique lancée par le **fournisseur de services** .

## <a name="adding-soloinsight-cloudgate-sso-from-the-gallery"></a>Ajout de Soloinsight-CloudGate SSO à partir de la galerie

Pour configurer l’intégration de Soloinsight-CloudGate SSO à Azure AD, vous devez ajouter Soloinsight-CloudGate SSO à partir de la galerie à votre liste d’applications SaaS managées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Soloinsight-CloudGate SSO** dans la zone de recherche.
1. Sélectionnez **Soloinsight-CloudGate SSO** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Configurez et testez l’authentification unique Azure AD avec Soloinsight-CloudGate SSO à l’aide d’un utilisateur de test appelé **Britta Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Soloinsight-CloudGate SSO associé.

Pour configurer et tester l’authentification unique Azure AD avec Soloinsight-CloudGate SSO, suivez les indications des sections ci-dessous :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer Soloinsight-CloudGate SSO](#configure-soloinsight-cloudgate-sso)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Soloinsight-CloudGate SSO](#create-soloinsight-cloudgate-sso-test-user)** pour avoir un équivalent de Britta Simon dans Soloinsight-CloudGate SSO lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [Portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Soloinsight-CloudGate SSO** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet pour **Configuration SAML de base** afin de modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la boîte de dialogue **Configuration SAML de base** , entrez les valeurs pour les champs suivants :

    1. Dans la zone de texte **URL de connexion** , saisissez une URL au format suivant : `https://<SUBDOMAIN>.sigateway.com/login`

    1. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL au format suivant : `https://<SUBDOMAIN>.sigateway.com/process/sso`

   > [!NOTE]
   > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur et l’URL de connexion réels, tel qu’expliqué dans la section **Configurer l’authentification unique Soloinsight-CloudGate SSO** de ce tutoriel.

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (Base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

   ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer Soloinsight-CloudGate SSO** , copiez la ou les URL appropriées en fonction de vos besoins.

   ![Copier les URL de configuration](common/copy-configuration-urls.png)

### <a name="configure-soloinsight-cloudgate-sso"></a>Configurer Soloinsight-CloudGate SSO

1. Pour automatiser la configuration dans Soloinsight-CloudGate SSO, vous devez installer l’ **extension de navigateur My Apps Secure Sign-in** en cliquant sur **Installer l’extension** .

    ![Extension My apps](common/install-myappssecure-extension.png)

2. Après l’ajout de l’extension au navigateur, cliquez sur **Configurer Soloinsight-CloudGate SSO** pour être dirigé vers l’application Soloinsight-CloudGate SSO. À partir de là, indiquez les informations d’identification de l’administrateur pour vous connecter à Soloinsight-CloudGate SSO. Cette extension de navigateur configure automatiquement l’application et automatise les étapes 3 à 8.

    ![Configuration](common/setup-sso.png)

3. Si vous souhaitez configurer manuellement Soloinsight-CloudGate SSO, ouvrez une nouvelle fenêtre de navigateur web, connectez-vous à votre site d’entreprise Soloinsight-CloudGate SSO en tant qu’administrateur, puis effectuez les étapes suivantes :

4. Pour obtenir les valeurs qui doivent être collées dans le portail Azure lors de la configuration SAML de base, connectez-vous au portail web CloudGate à l’aide de vos informations d’identification, puis accédez aux paramètres d’authentification unique à l’emplacement suivant : **Home>Administration>System settings>General** .

    ![Paramètres CloudGate SSO](./media/soloinsight-cloudgate-sso-tutorial/sso-main-settings.png)

5. **URL de consommateur SAML**

    * Copiez les liens indiqués dans les champs **Saml Consumer URL** (URL de consommateur SAML) et **Redirect URL** (URL de redirection), et collez-les dans le Portail Azure dans la section **Configuration SAML de base** , respectivement dans les champs **Identificateur (ID d’entité)** et **URL de réponse** .

        ![SAMLIdentifier](./media/soloinsight-cloudgate-sso-tutorial/saml-identifier.png)

6. **Certificat de signature SAML**

    * Accédez à la source du fichier de certificat (Base64) téléchargé à partir des listes de certificats de signature SAML sur le Portail Azure, et cliquez dessus avec le bouton droit. Choisissez l’option **Edit with Notepad++** (Modifier avec Notepad++) dans la liste. 

        ![SAMLcertificate](./media/soloinsight-cloudgate-sso-tutorial/certificate-file.png)

    * Copiez le contenu dans le fichier de certificat (Base64) Notepad++.

        ![Copie du certificat](./media/soloinsight-cloudgate-sso-tutorial/certificate-copy.png)

    * Collez le contenu dans le champ **Certificate** des paramètres d’authentification unique dans le portail web CloudGate et cliquez sur le bouton Save (Enregistrer).

        ![Portail de certificat](./media/soloinsight-cloudgate-sso-tutorial/certificate-portal.png)

7. **Groupe par défaut**

    * Sélectionnez **Business Admin** (Administrateur d’entreprise) dans la liste déroulante de l’option **Default Group** (Groupe par défaut) dans le portail web CloudGate.

        ![Groupe par défaut](./media/soloinsight-cloudgate-sso-tutorial/default-group.png)

8. **Identificateur AD et URL de connexion**

    * L’ **URL de connexion** copiée à partir des paramètres **Configurer l’authentification unique Soloinsight-CloudGate** dans le Portail Azure doit être entrée dans la section de paramètres d’authentification unique du portail web CloudGate.

    * Collez le lien **URL de connexion** du Portail Azure dans le champ **AD Login URL** (URL de connexion AD) du portail web CloudGate.

    * Collez le lien **Identificateur Azure AD** du Portail Azure dans le champ **AD Identifier** (Identificateur AD) du portail web CloudGate.

        ![Connexion AD](./media/soloinsight-cloudgate-sso-tutorial/ad-login.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé Britta Simon dans le Portail Azure.

1. Dans le volet gauche du Portail Azure, sélectionnez **Azure Active Directory** , **Utilisateurs** , puis **Tous les utilisateurs** .
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur** , effectuez les étapes suivantes :
   1. Dans le champ **Nom** , entrez `Britta Simon`.  
   1. Dans le champ **Nom de l’utilisateur** , entrez username@companydomain.extension. Par exemple : `BrittaSimon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ **Mot de passe** .
   1. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Soloinsight-CloudGate SSO.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Soloinsight-CloudGate SSO** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-soloinsight-cloudgate-sso-test-user"></a>Créer utilisateur de test Soloinsight-CloudGate SSO

Pour créer un utilisateur de test, sélectionnez **Employees** dans le menu principal du portail web CloudGate et renseignez le formulaire Add New employee (Ajouter un nouvel employé). Le niveau d’autorité qui doit être affecté à l’utilisateur de test est **Business Admin** (Administrateur d’entreprise). Cliquez sur **Create** (Créer) une fois que vous avez renseigné tous les champs obligatoires.

![Test de l’employé](./media/soloinsight-cloudgate-sso-tutorial/employee-test.png)

### <a name="test-sso"></a>Tester l’authentification unique (SSO)

Quand vous sélectionnez la vignette Soloinsight-CloudGate SSO dans le volet d’accès, vous devez être connecté automatiquement à l’application Soloinsight-CloudGate SSO pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)