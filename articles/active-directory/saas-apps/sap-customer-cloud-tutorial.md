---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à SAP Cloud for Customer | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et SAP Cloud pour le client.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/20/2019
ms.author: jeedes
ms.openlocfilehash: 9b244d5e6aa9dd732aa670fd645b6bbc266c9535
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92897277"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-sap-cloud-for-customer"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à SAP Cloud for Customer

Dans ce tutoriel, vous allez apprendre à intégrer SAP Cloud for Customer à Azure Active Directory (Azure AD). Quand vous intégrez SAP Cloud for Customer à Azure AD, vous pouvez :

* Dans Azure AD, contrôler qui a accès à SAP Cloud for Customer.
* Permettre à vos utilisateurs de se connecter automatiquement à SAP Cloud for Customer avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement SAP Cloud for Customer pour lequel l’authentification unique (SSO) est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* SAP Cloud pour le client prend en charge l’authentification unique initiée par le **fournisseur de services**

## <a name="adding-sap-cloud-for-customer-from-the-gallery"></a>Ajout de SAP Cloud pour le client à partir de la galerie

Pour configurer l’intégration de SAP Cloud pour le client à Azure AD, vous devez ajouter SAP Cloud pour le client depuis la galerie à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie** , tapez **SAP Cloud for Customer** dans la zone de recherche.
1. Sélectionnez **SAP Cloud for Customer** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-sap-cloud-for-customer"></a>Configurer et tester l’authentification unique Azure AD pour SAP Cloud for Customer

Configurez et testez l’authentification unique Azure AD avec SAP Cloud for Customer avec un utilisateur de test appelé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur SAP Cloud for Customer associé.

Pour configurer et tester l’authentification unique (SSO) Azure AD avec SAP Cloud for Customer, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique SAP Cloud for Customer](#configure-sap-cloud-for-customer-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test SAP Cloud for Customer](#create-sap-cloud-for-customer-test-user)** pour avoir un équivalent de B.Simon dans SAP Cloud for Customer lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **SAP Cloud for Customer** , recherchez la section **Gérer** et sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **URL de connexion** , saisissez une URL au format suivant : `https://<server name>.crm.ondemand.com`

    b. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL au format suivant : `https://<server name>.crm.ondemand.com`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion et l’identificateur réels. Pour obtenir ces valeurs, contactez [l’équipe de support technique SAP Cloud for Customer](https://www.sap.com/about/agreements.sap-cloud-services-customers.html). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. L’application SAP Cloud for Customer attend les assertions SAML dans un certain format, ce qui oblige à ajouter des mappages d’attributs personnalisés à la configuration des attributs du jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut. Cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue Attributs d’utilisateur.

    ![Capture d’écran montrant la boîte de dialogue « Attributs utilisateur » avec l’icône « Modifier » sélectionnée.](common/edit-attribute.png)

1. Dans la section **Attributs utilisateur** de la boîte de dialogue **Attributs et revendications de l’utilisateur** , procédez comme suit :

    a. Cliquez sur l’ **icône Modifier** pour ouvrir la boîte de dialogue **Gérer les revendications des utilisateurs**.

    ![Capture d’écran montrant la boîte de dialogue « Attributs et revendications de l’utilisateur » avec l’icône « Modifier » sélectionnée](./media/sap-customer-cloud-tutorial/tutorial_usermail.png)

    ![image](./media/sap-customer-cloud-tutorial/tutorial_usermailedit.png)

    b. Sélectionnez **Transformation** dans le champ **source**.

    c. Dans la liste **Transformation** , sélectionnez **ExtractMailPrefix()** .

    d. Dans la liste **Paramètre 1** , sélectionnez l’attribut utilisateur que vous souhaitez utiliser pour votre implémentation.
    Par exemple, si vous souhaitez utiliser EmployeeID comme identificateur d’utilisateur unique et que vous avez stocké la valeur d’attribut dans ExtensionAttribute2, sélectionnez user.extensionattribute2.

    e. Cliquez sur **Enregistrer**.

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **XML de métadonnées de fédération** et sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

1. Dans la section **Configurer SAP Cloud for Customer** , copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé B. Simon dans le portail Azure.

1. Dans le volet gauche du Portail Azure, sélectionnez **Azure Active Directory** , **Utilisateurs** , puis **Tous les utilisateurs**.
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur** , effectuez les étapes suivantes :
   1. Dans le champ **Nom** , entrez `B.Simon`.  
   1. Dans le champ **Nom de l’utilisateur** , entrez username@companydomain.extension. Par exemple : `B.Simon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ **Mot de passe**.
   1. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à SAP Cloud for Customer.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **Cloud SAP pour le client**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer**.

## <a name="configure-sap-cloud-for-customer-sso"></a>Configurer l’authentification unique SAP Cloud for Customer

1. Ouvrez une nouvelle fenêtre de navigateur web et connectez-vous à votre site d’entreprise SAP Cloud for Customer en tant qu’administrateur.

2. Dans le côté gauche du menu, cliquez sur **Identity Providers** > **Corporate Identity Providers** > **Add** (Fournisseur d’identité > Fournisseur d’identité d’entreprise > Ajouter) et, dans la fenêtre contextuelle, ajoutez le nom du fournisseur d’identité, tel qu’ **Azure AD** , cliquez sur **Save** (Enregistrer), puis cliquez sur **SAML 2.0 Configuration** (Configuration de SAML 2.0).

    ![Capture d’écran montrant la page « Identity Providers » avec la boîte de dialogue « Add Identity Provider » mise en évidence et le bouton « Save » sélectionné.](./media/sap-customer-cloud-tutorial/configure01.png)

3. Dans la section **SAML 2.0 Configuration** , effectuez les étapes suivantes :

    ![Capture d’écran montrant la section « SAML 2.0 Configuration » avec le bouton « Browse » sélectionné.](./media/sap-customer-cloud-tutorial/configure02.png)

    a. Cliquez sur **Browse** (Parcourir) pour charger le fichier XML de métadonnées de fédération que vous avez téléchargé à partir du portail Azure.

    b. Une fois le fichier XML correctement chargé, les valeurs ci-dessous sont automatiquement renseignées ; cliquez ensuite sur **Save**.

### <a name="create-sap-cloud-for-customer-test-user"></a>Créer un utilisateur de test SAP Cloud pour le client

Pour permettre aux utilisateurs Azure AD de se connecter à SAP Cloud for Customer, vous devez les provisionner dans SAP Cloud for Customer. Dans SAP Cloud for Customer, le provisionnement est une tâche manuelle.

**Pour approvisionner un compte d’utilisateur, procédez comme suit :**

1. Connectez-vous à SAP Cloud for Customer en tant qu’administrateur de la sécurité.

2. Sur le côté gauche du menu, cliquez sur **Users & Authorizations** > **User Management** > **Add User** (Utilisateurs & autorisations > Gestion des utilisateurs > Ajouter un utilisateur).

    ![Capture d’écran montrant la page « User Management » avec le bouton « Add User » sélectionné.](./media/sap-customer-cloud-tutorial/configure03.png)

3. Dans la section **Add New User** (Ajouter un nouvel utilisateur), effectuez les étapes suivantes :

    ![Configuration SAP](./media/sap-customer-cloud-tutorial/configure04.png)

    a. Dans la zone de texte **First Name** (Prénom), entrez le prénom d’un utilisateur, par exemple **B**.

    b. Dans la zone de texte **Last Name** (Nom), entrez le nom de l’utilisateur, par exemple **Simon**.

    c. Dans la zone de texte **E-Mail** , entrez une adresse e-mail d’utilisateur, par exemple `B.Simon@contoso.com`.

    d. Dans la zone de texte **Login Name** (Nom de connexion), entrez le nom de l’utilisateur, par exemple **B.Simon**.

    e. Sélectionnez le type d’utilisateur ( **User Type** ) en fonction de vos besoins.

    f. Sélectionnez l’option **Account Activation** (Activation de compte) en fonction de vos besoins.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Lorsque vous cliquez sur la vignette SAP Cloud pour le client dans le volet d’accès, vous devez être connecté automatiquement à l’application SAP Cloud pour le client pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer SAP Cloud for Customer avec Azure AD](https://aad.portal.azure.com/)