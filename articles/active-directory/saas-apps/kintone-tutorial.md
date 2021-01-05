---
title: 'Didacticiel : intégration d’Azure Active Directory avec Kintone | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Kintone.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/26/2019
ms.author: jeedes
ms.openlocfilehash: 96ea3423d3c2dff2c8ba8c82b4c26d318c47211f
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92459048"
---
# <a name="tutorial-azure-active-directory-integration-with-kintone"></a>Didacticiel : intégration d’Azure Active Directory avec Kintone

L’objectif de ce didacticiel est de vous apprendre à intégrer Kintone avec Azure Active Directory (Azure AD).
Intégrer Kintone avec Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Kintone.
* Vous pouvez permettre à vos utilisateurs de se connecter automatiquement à Kintone (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD avec Kintone, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/)
* Un abonnement Kintone pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Kintone prend en charge l’authentification unique lancée par le **fournisseur de services**

## <a name="adding-kintone-from-the-gallery"></a>Ajout de Kintone à partir de la galerie

Pour configurer l’intégration de Kintone avec Azure AD, vous devez ajouter Kintone, disponible dans la galerie, à votre liste d’applications SaaS gérées.

**Pour ajouter Kintone à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Kintone** , sélectionnez **Kintone** dans le panneau de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![Kintone dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Kintone au moyen d’un utilisateur de test appelé **Britta Simon** .
Pour que l’authentification unique fonctionne, une relation entre un utilisateur Azure AD et l’utilisateur Kintone associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec Kintone, vous devez compléter les blocs de construction suivants :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Kintone](#configure-kintone-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Kintone](#create-kintone-test-user)** pour avoir dans Kintone un équivalent de Britta Simon lié à la représentation Azure AD de l’utilisateur.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Kintone, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), sur la page d’intégration de l’application **Kintone** , sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , effectuez les étapes suivantes :

    ![Informations d’authentification unique dans Domaine et URL Kintone](common/sp-identifier.png)

    a. Dans la zone de texte **URL de connexion** , saisissez une URL au format suivant : `https://<companyname>.kintone.com`

    b. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL au format suivant :
    
    ```http
    https://<companyname>.cybozu.com
    https://<companyname>.kintone.com
    ```

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion et l’identificateur réels. Pour obtenir ces valeurs, contactez [l’équipe de support technique Kintone](https://www.kintone.com/contact/). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

5. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** en fonction des options définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

6. Dans la section **Configurer Kintone** , copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-kintone-single-sign-on"></a>Configurer l’authentification unique Kintone

1. Dans une autre fenêtre de navigateur web, connectez-vous à votre site d’entreprise **Kintone** en tant qu’administrateur.

1. Cliquez sur l’ **icône Paramètres** .

    ![Paramètres](./media/kintone-tutorial/ic785879.png "Paramètres")

1. Cliquez sur **Users & System Administration** .

    ![Users & System Administration](./media/kintone-tutorial/ic785880.png "Users & System Administration") (Administration système et des utilisateurs)

1. Sous **System Administration \> Security** , cliquez sur **Login** .

    ![Connexion](./media/kintone-tutorial/ic785881.png "Connexion")

1. Cliquez sur **Enable SAML authentication** .

    ![Capture d’écran montrant la sélection de l’option « Users & System Administration ».](./media/kintone-tutorial/ic785882.png "Authentification SAML")

1. Dans la section SAML Authentication, procédez comme suit :

    ![Authentification SAML](./media/kintone-tutorial/ic785883.png "Authentification SAML")

    a. Dans la zone de texte **Login URL** (URL de connexion), collez la valeur **URL de connexion** que vous avez copiée dans le portail Azure.

    b. Dans la zone de texte **Logout URL** (URL de déconnexion), collez la valeur de l’ **URL de déconnexion** que vous avez copiée à partir du portail Azure.

    c. Cliquez sur **Parcourir** pour charger le fichier de certificat téléchargé à partir du portail Azure.

    d. Cliquez sur **Enregistrer** .

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , sélectionnez **Utilisateurs** , puis sélectionnez **Tous les utilisateurs** .

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom** , entrez **BrittaSimon** .
  
    b. Dans le champ **Nom d’utilisateur** , tapez `brittasimon@yourcompanydomain.extension`  
    Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant un accès à Kintone.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **Kintone** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Kintone** .

    ![Lien Kintone dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-kintone-test-user"></a>Créer un utilisateur de test Kintone

Pour pouvoir se connecter à Kintone, les utilisateurs Azure AD doivent être provisionnés dans Kintone. Dans le cas de Kintone, l’approvisionnement est une tâche manuelle.

### <a name="to-provision-a-user-account-perform-the-following-steps"></a>Pour approvisionner un compte d’utilisateur, procédez comme suit :

1. Connectez-vous à votre site d’entreprise **Kintone** en tant qu’administrateur.

1. Cliquez sur l’ **icône Paramètres** .

    ![Paramètres](./media/kintone-tutorial/ic785879.png "Paramètres")

1. Cliquez sur **Users & System Administration** .

    ![User & System Administration](./media/kintone-tutorial/ic785880.png "Administration système et utilisateur") (Administration système et des utilisateurs)

1. Sous **User Administration** , cliquez sur **Departments & Users** .

    ![Department & Users](./media/kintone-tutorial/ic785888.png "Département et utilisateurs") (Services et utilisateurs)

1. Cliquez sur **New User** .

    ![Capture d’écran montrant la section « Users » avec l’action « New User » sélectionnée.](./media/kintone-tutorial/ic785889.png "Nouveaux utilisateurs")

1. Dans la section **New User** , procédez comme suit :

    ![New Users](./media/kintone-tutorial/ic785890.png "Nouveaux utilisateurs") (Nouveaux utilisateurs)

    a. Dans les zones de texte correspondantes, tapez un **Nom d’affichage** , un **Nom de connexion** , un **Nouveau mot de passe** , confirmez-le dans **Confirmer le mot de passe** , tapez une **Adresse e-mail** et indiquez d’autres détails d’un compte Azure AD valide que vous voulez provisionner.

    b. Cliquez sur **Enregistrer** .

> [!NOTE]
> Vous pouvez utiliser tout autre outil ou n’importe quelle API de création de compte d’utilisateur fournis par Kintone pour provisionner des comptes d’utilisateurs Azure AD.

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Kintone dans le volet d’accès, vous devez vous connecter automatiquement à l’application Kintone pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)