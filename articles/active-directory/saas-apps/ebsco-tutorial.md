---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à EBSCO | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et EBSCO.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/11/2019
ms.author: jeedes
ms.openlocfilehash: 2f7f017d773e7708b2206029769df7977b12e6f0
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92454335"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-ebsco"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à EBSCO

Dans ce tutoriel, vous allez découvrir comment intégrer EBSCO à Azure Active Directory (Azure AD). Quand vous intégrez EBSCO à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à EBSCO.
* Permettre à vos utilisateurs de se connecter automatiquement à EBSCO avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement EBSCO pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* EBSCO prend en charge l’authentification unique lancée par **le fournisseur de services et le fournisseur d’identité**
* EBSCO prend en charge le provisionnement d’utilisateurs **juste-à-temps**

> [!NOTE]
> L’identificateur de cette application étant une valeur de chaîne fixe, une seule instance peut être configurée dans un locataire.

## <a name="adding-ebsco-from-the-gallery"></a>Ajout d’EBSCO depuis la galerie

Pour configurer l’intégration d’EBSCO à Azure AD, vous devez ajouter EBSCO, à partir de la galerie, à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **EBSCO** dans la zone de recherche.
1. Sélectionnez **EBSCO** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-ebsco"></a>Configurer et tester l’authentification unique Azure AD pour EBSCO

Configurez et testez l’authentification unique Azure AD avec EBSCO à l’aide d’un utilisateur de test appelé **B. Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur EBSCO associé.

Pour configurer et tester l’authentification unique Azure AD avec EBSCO, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique EBSCO](#configure-ebsco-sso)** pour configurer les paramètres de l’authentification unique côté application.
    * **[Créer un utilisateur de test EBSCO](#create-ebsco-test-user)** pour avoir dans EBSCO un équivalent de B.Simon lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **EBSCO** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode Initié par le **fournisseur d’identité** , entrez les valeurs pour les champs suivants :

    Dans la zone de texte **Identificateur** , tapez une URL :  `pingsso.ebscohost.com`

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `http://search.ebscohost.com/login.aspx?authtype=sso&custid=<unique EBSCO customer ID>&profile=<profile ID>`

    > [!NOTE]
    > La valeur de l’URL de connexion n’est pas réelle. Mettez à jour la valeur avec l’URL de connexion réelle. Pour obtenir ces valeurs, contactez l’[équipe du support EBSCO](mailto:support@ebsco.com). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

    o **Éléments uniques :**  

    o **Custid** = Entrez l’ID client EBSCO unique 

    o **Profile** = Les clients peuvent adapter le lien pour diriger les utilisateurs vers un profil spécifique (selon ce qu’ils achètent auprès d’EBSCO). Ils peuvent entrer un ID de profil spécifique. Les principaux ID sont eds (service de découverte EBSCO) et ehost (bases de données EBSOCOhost). Vous trouverez des instructions les concernant [ici](https://help.ebsco.com/interfaces/EBSCOhost/EBSCOhost_FAQs/How_do_I_set_up_direct_links_to_EBSCOhost_profiles_and_or_databases#profile).

1. L’application EBSCO s’attend à recevoir les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à votre configuration des attributs du jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut.

    ![image](common/default-attributes.png)

    > [!Note]
    > L’attribut **name** est obligatoire et il est mappé sur la **valeur de l’identificateur de nom** dans l’application EBSCO. C’est ajouté par défaut pour vous éviter d’avoir à le faire manuellement.

1. En plus de ce qui précède, l’application EBSCO s’attend à ce que quelques attributs supplémentaires soient repassés dans la réponse SAML. Ces attributs sont également préremplis, mais vous pouvez les examiner pour voir s’ils répondent à vos besoins.

    | Nom | Attribut source|
    | ---------------| --------------- |
    | FirstName   | user.givenname |
    | LastName   | user.surname |
    | E-mail   | user.mail |

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **XML de métadonnées de fédération** et sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

1. Dans la section **Configurer EBSCO** , copiez la ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à EBSCO.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **EBSCO** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-ebsco-sso"></a>Configurer l’authentification unique EBSCO

Pour configurer l’authentification unique côté **EBSCO** , vous devez envoyer le **XML des métadonnées de fédération** téléchargé et les URL copiées correspondantes à partir du portail Azure à l’ [équipe du support EBSCO](mailto:support@ebsco.com). Celles-ci configurent ensuite ce paramètre pour que la connexion SSO SAML soit définie correctement des deux côtés.

### <a name="create-ebsco-test-user"></a>Créer un utilisateur de test EBSCO

Dans le cas d’EBSCO, l’attribution d’utilisateurs est automatique.

**Pour approvisionner un compte d’utilisateur, procédez comme suit :**

Azure AD transmet les données requises à l’application EBSCO. L’attribution d’utilisateurs d’EBSCO peut être automatique ou exiger un formulaire à usage unique. Cela dépend si le client a un grand nombre de comptes EBSCOhost préexistants avec des paramètres personnels enregistrés. Vous pouvez en discuter avec [l’équipe de support technique d’EBSCO](mailto:support@ebsco.com) pendant l’implémentation. Dans les deux cas, le client n’a pas besoin de créer de comptes EBSCOhost avant de procéder aux tests.

   > [!Note]
   > Vous pouvez automatiser la personnalisation et l’attribution d’utilisateurs d’EBSCOhost. Contactez [l’équipe de support technique d’EBSCO](mailto:support@ebsco.com) concernant l’attribution d’utilisateurs juste-à-temps.

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

1. Quand vous cliquez sur la vignette EBSCO dans le volet d’accès, vous devez être connecté automatiquement à votre application EBSCO.
Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

1. Une fois connecté à l’application, cliquez sur le bouton **Sign In** (Se connecter) dans l’angle supérieur droit.

    ![Connexion à EBSCO dans la liste des applications](./media/ebsco-tutorial/tutorial_ebsco_signin.png)

1. Vous recevez une invite à usage unique pour lier la connexion institutionnelle/SAML en choisissant l’option **Link your existing MyEBSCOhost account to your institution account now** (Lier maintenant votre compte MyEBSCOhost existant à votre compte institutionnel) ou **Create a new MyEBSCOhost account and link it to your institution account** (Créer un compte MyEBSCOhost et le lier à votre compte institutionnel). Le compte est utilisé pour la personnalisation de l’application EBSCOhost. Sélectionnez l’option de **création de compte** ; vous verrez alors que le formulaire pour la personnalisation est prérempli avec les valeurs de la réponse saml, comme indiqué dans la capture d’écran ci-dessous. Cliquez sur **Continue** (Continuer) pour enregistrer cette sélection.
    
     ![Utilisateur EBSCO dans la liste des applications](./media/ebsco-tutorial/tutorial_ebsco_user.png)

1. Après avoir effectué la configuration ci-dessus, effacez les cookies et le cache, puis reconnectez-vous. Vous ne devrez pas vous reconnecter manuellement car les paramètres de personnalisation sont mémorisés.

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer EBSCO avec Azure AD](https://aad.portal.azure.com/)