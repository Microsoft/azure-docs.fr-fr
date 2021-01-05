---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à NetDocuments | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et NetDocuments.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/03/2019
ms.author: jeedes
ms.openlocfilehash: b9a85c7093d0f42ac97c4a548ba6f0feebae8b89
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96350719"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-netdocuments"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à NetDocuments

Dans ce tutoriel, vous allez apprendre à intégrer NetDocuments à Azure Active Directory (Azure AD). Quand vous intégrez NetDocuments à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à NetDocuments.
* Permettre à vos utilisateurs de se connecter automatiquement à NetDocuments avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement NetDocuments pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* NetDocuments prend en charge l’authentification unique initiée par le **fournisseur de services**

## <a name="adding-netdocuments-from-the-gallery"></a>Ajout de NetDocuments à partir de la galerie

Pour configurer l’intégration de NetDocuments à Azure AD, vous devez ajouter NetDocuments à votre liste d’applications SaaS gérées, à partir de la galerie.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **NetDocuments** dans la zone de recherche.
1. Sélectionnez **NetDocuments** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-netdocuments"></a>Configurer et tester l’authentification unique Azure AD pour NetDocuments

Configurez et testez l’authentification unique Azure AD avec NetDocuments à l’aide d’un utilisateur de test nommé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur NetDocuments associé.

Pour configurer et tester l’authentification unique Azure AD avec NetDocuments, effectuez les modules suivants :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique NetDocuments](#configure-netdocuments-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test NetDocuments](#create-netdocuments-test-user)** pour obtenir un équivalent de B.Simon dans NetDocuments lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **NetDocuments**, recherchez la section **Gérer** et sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base**, entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **URL de connexion**, saisissez une URL au format suivant : `https://vault.netvoyage.com/neWeb2/docCent.aspx?whr=<Repository ID>`

    b. Dans la zone de texte **URL de réponse**, tapez une URL au format suivant : `https://vault.netvoyage.com/neWeb2/docCent.aspx?whr=<Repository ID>`
    
    c. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL au format suivant : `http://netdocuments.com/VAULT`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de réponse et l’URL de connexion réelles. L’ID de dépôt est une valeur commençant par **CA-** suivi du code à huit caractères associé à votre dépôt NetDocuments. Pour plus d’informations, consultez le [document sur la prise en charge des identités fédérées NetDocuments](https://support.netdocuments.com/hc/en-us/articles/205220410-Federated-Identity-Login). Vous pouvez aussi contacter l’[équipe de support technique NetDocuments](https://support.netdocuments.com/hc/) pour obtenir ces valeurs si vous avez des difficultés à effectuer la configuration à partir des informations fournies ci-dessus. Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. L’application NetDocuments attend les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à la configuration des attributs du jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut, où **nameidentifier** est mappé avec **user.userprincipalname**. L’application NetDocuments attend le mappage de **nameidentifier** avec **employeeid** ou toute autre revendication applicable à votre organisation en tant que **nameidentifier**. Vous devez donc modifier le mappage d’attributs en cliquant sur l’icône **Modifier** et en procédant aux changements.

    ![image](common/edit-attribute.png)

1. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, recherchez **XML de métadonnées de fédération** et sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

1. Dans la section **Configurer NetDocuments**, copiez l’URL ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé B. Simon dans le portail Azure.

1. Dans le volet gauche du Portail Azure, sélectionnez **Azure Active Directory**, **Utilisateurs**, puis **Tous les utilisateurs**.
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur**, effectuez les étapes suivantes :
   1. Dans le champ **Nom**, entrez `B.Simon`.  
   1. Dans le champ **Nom de l’utilisateur**, entrez username@companydomain.extension. Par exemple : `B.Simon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe**, puis notez la valeur affichée dans le champ **Mot de passe**.
   1. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en accordant l’accès à NetDocuments.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **NetDocuments**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

## <a name="configure-netdocuments-sso"></a>Configurer l’authentification unique NetDocuments

1. Dans une autre fenêtre de navigateur web, connectez-vous à votre site d’entreprise NetDocuments en tant qu’administrateur.

2. En haut à droite, sélectionnez votre nom>**Admin**.

3. Sélectionnez **Centre de sécurité**.
   
    ![Capture d’écran montrant l’élément Security Center sélectionné dans Legal Documents.](./media/netdocuments-tutorial/security-center.png "Security Center")

4. Sélectionnez **Authentification avancée**.
    
    ![Configurer les options d’authentification avancées](./media/netdocuments-tutorial/advance-authentication.png "Configure advanced authentication options")

5.  Sous l’onglet **ID fédéré**, effectuez les étapes suivantes :   
   
    ![Identité fédérée](./media/netdocuments-tutorial/federated-id.png "Identité fédérée")
   
    a. Pour **Federated identity server type**, sélectionnez **Active Directory Federation Services**.
    
    b.  Sélectionnez **Choisir un fichier** pour importer le fichier de métadonnées que vous avez téléchargé à partir du portail Azure.
    
    c.  Sélectionnez **SAVE** (Enregistrer).

### <a name="create-netdocuments-test-user"></a>Créer un utilisateur de test NetDocuments

Pour permettre aux utilisateurs Azure AD de se connecter à NetDocuments, vous devez les configurer dans NetDocuments. Dans le cas de NetDocuments, l’approvisionnement est une tâche manuelle.

**Pour approvisionner un compte d’utilisateur, procédez comme suit :**

1. Connectez-vous à votre site d’entreprise **NetDocuments** en tant qu’administrateur.

2. En haut à droite, sélectionnez votre nom>**Admin**.
   
    ![Administrateur](./media/netdocuments-tutorial/user-admin.png "Admin")

3. Sélectionnez **Utilisateurs et groupes**.
   
    ![Capture d’écran montrant l’élément Users & Groups sélectionné dans Legal Documents.](./media/netdocuments-tutorial/users-groups.png "Référentiel")

4. Dans la zone de texte **Email Address**, tapez l’adresse e-mail d’un compte Azure Active Directory valide à approvisionner, puis cliquez sur **Add User**.
   
    ![Adresse e-mail](./media/netdocuments-tutorial/user-mail.png "Adresse de messagerie")
   
    > [!NOTE]
    > Le titulaire du compte Azure Active Directory recevra un message électronique contenant un lien pour confirmer le compte avant qu’il ne soit activé. Vous pouvez utiliser n’importe quels outils ou API de création de compte utilisateur, fournis par NetDocuments, pour approvisionner des comptes utilisateur Azure Active Directory.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette NetDocuments dans le panneau d’accès doit vous connecter automatiquement à l’application NetDocuments pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer NetDocuments avec Azure AD](https://aad.portal.azure.com/)