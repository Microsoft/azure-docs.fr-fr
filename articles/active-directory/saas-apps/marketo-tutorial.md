---
title: 'Didacticiel : Intégration d’Azure Active Directory à Marketo | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Marketo.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/04/2020
ms.author: jeedes
ms.openlocfilehash: bd62855aef59439ed476a13770f79ce6d3cf68f8
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96180793"
---
# <a name="tutorial-azure-active-directory-integration-with-marketo"></a>Didacticiel : Intégration d’Azure Active Directory à Marketo

Dans ce didacticiel, vous allez apprendre à intégrer Marketo à Azure Active Directory (Azure AD).
L’intégration de Marketo à Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès Marketo.
* Vous pouvez permettre à vos utilisateurs d’être automatiquement connectés à Marketo (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

## <a name="prerequisites"></a>Prérequis

Pour configurer l’intégration d’Azure AD à Marketo, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Abonnement Marketo pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Marketo prend en charge l’authentification unique initiée par le **fournisseur d’identité**

## <a name="adding-marketo-from-the-gallery"></a>Ajout de Marketo depuis la galerie

Pour configurer l’intégration de Marketo à Azure AD, vous devez ajouter Marketo, depuis la galerie, à votre liste d’applications SaaS gérées.

1. Connectez-vous au portail Azure avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **Marketo** dans la zone de recherche.
1. Sélectionnez **Marketo** dans le volet des résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-sso"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD auprès de Marketo, à l’aide d’un utilisateur de test nommé **Britta Simon**.
Pour que l’authentification unique fonctionne, une relation entre l’utilisateur Azure AD et l’utilisateur Marketo associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD auprès de Marketo, effectuez les étapes suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique (SSO) Azure AD avec Britta Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique (SSO) Azure AD.
2. **[Configurer l’authentification unique Marketo](#configure-marketo-sso)** pour configurer les paramètres d’authentification unique côté application.
    * **[Créer l’utilisateur de test Marketo](#create-marketo-test-user)** pour avoir dans Marketo un équivalent de Britta Simon lié à la représentation Azure AD associée.
3. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le portail Azure, accédez à la page d’intégration de l’application **Marketo**, recherchez la section **Gérer**, puis sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base**, entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **Identificateur**, tapez une URL au format suivant : `https://saml.marketo.com/sp`

    b. Dans la zone de texte **URL de réponse**, tapez une URL au format suivant : `https://login.marketo.com/saml/assertion/\<munchkinid\>`

    c. Dans la zone de texte **État de relais**, entrez une URL en utilisant le modèle suivant : `https://<munchkinid>.marketo.com/`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur, l’URL de réponse et l’état du relais exacts. Pour obtenir ces valeurs, contactez [l’équipe du support technique de Marketo](https://investors.marketo.com/contactus.cfm). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

5. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** en fonction des options définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

6. Dans la section **Configurer Marketo**, copiez l’URL ou les URL appropriées, en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Marketo.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **Marketo**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.
1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.
1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez qu’un rôle soit attribué aux utilisateurs, vous pouvez le sélectionner dans la liste déroulante **Sélectionner un rôle** . Si aucun rôle n’a été configuré pour cette application, vous voyez le rôle « Accès par défaut » sélectionné.
1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

## <a name="configure-marketo-sso"></a>Configurer l’authentification unique Marketo

1. Pour obtenir l’ID Munchkin de votre application, connectez-vous à Marketo à l’aide des informations d’identification d’administrateur, puis effectuez les actions suivantes :
   
    a. Connectez-vous à l’application Marketo à l’aide des informations d’identification de l’administrateur.
   
    b. Dans le volet de navigation supérieur, cliquez sur le bouton **Administrateur**.
   
    ![Configurer l’authentification unique 1](./media/marketo-tutorial/tutorial_marketo_06.png) 
   
    c. Accédez au menu Intégration et cliquez sur le **lien Munchkin**.
   
    ![Configurer l’authentification unique 2](./media/marketo-tutorial/tutorial_marketo_11.png)
   
    d. Copiez l’ID Munchkin indiqué dans l’écran, puis complétez votre URL de réponse dans l’Assistant Configuration Azure AD.
   
    ![Configurer l’authentification unique 3](./media/marketo-tutorial/tutorial_marketo_12.png) 

2. Pour configurer l’authentification unique de l’application, suivez les étapes ci-dessous :
   
    a. Connectez-vous à l’application Marketo à l’aide des informations d’identification de l’administrateur.
   
    b. Dans le volet de navigation supérieur, cliquez sur le bouton **Administrateur**.
   
    ![Configurer l’authentification unique 4](./media/marketo-tutorial/tutorial_marketo_06.png) 
   
    c. Accédez au menu Intégration et cliquez sur **Authentification unique**.
   
    ![Configurer l’authentification unique 5](./media/marketo-tutorial/tutorial_marketo_07.png) 
   
    d. Pour activer les paramètres SAML, cliquez sur le bouton **Modifier**.
   
    ![Configuration de l’authentification unique6](./media/marketo-tutorial/tutorial_marketo_08.png) 
   
    e. **Activez** les paramètres d’authentification unique.
   
    f. Collez la valeur de **Identificateur Azure AD** dans la zone de texte **Issuer ID** (ID de l’émetteur).
   
    g. Dans la zone de texte **ID d’entité**, tapez l’URL `http://saml.marketo.com/sp`.
   
    h. Sélectionnez l’emplacement d’ID utilisateur en tant **qu’élément Identificateur de nom**.
   
    ![Configurer l’authentification unique 7](./media/marketo-tutorial/tutorial_marketo_09.png)
   
    > [!NOTE]
    > Si votre identificateur d’utilisateur n’est pas une valeur UPN, alors changez la valeur dans l’onglet Attribut.
   
    i. Chargez le certificat que vous avez téléchargé à partir de l’Assistant de configuration Azure AD. **Enregistrez** les paramètres.
   
    j. Modifiez les paramètres des pages de redirection.
   
    k. Collez la valeur de l’**URL de connexion** dans la zone de texte **Login URL** (URL de connexion).
   
    l. Collez la valeur de l’**URL de déconnexion** dans la zone de texte **Logout URL** (URL de déconnexion).
   
    m. Dans l’**URL d’erreur**, copiez l’**URL de votre instance Marketo**, puis cliquez sur le bouton **Enregistrer** pour enregistrer les paramètres.
   
    ![Configurer l’authentification unique 8](./media/marketo-tutorial/tutorial_marketo_10.png)

3. Pour activer l’authentification unique pour les utilisateurs, procédez comme suit :
   
    a. Connectez-vous à l’application Marketo à l’aide des informations d’identification de l’administrateur.
   
    b. Dans le volet de navigation supérieur, cliquez sur le bouton **Administrateur**.
   
    ![Configurer l’authentification unique 9](./media/marketo-tutorial/tutorial_marketo_06.png) 
   
    c. Accédez au menu **Sécurité** et cliquez sur **Paramètres de connexion**.
   
    ![Configurer l’authentification unique 10](./media/marketo-tutorial/tutorial_marketo_13.png)
   
    d. Vérifiez l’option **Exiger l’authentification unique** et **enregistrez** les paramètres.
   
    ![Configurer l’authentification unique 11](./media/marketo-tutorial/tutorial_marketo_14.png)


### <a name="create-marketo-test-user"></a>Créer l’utilisateur de test Marketo

Dans cette section, vous allez créer un utilisateur appelé Britta Simon dans Marketo. Pour créer un utilisateur dans la plateforme Marketo, suivez ces étapes.

1. Connectez-vous à l’application Marketo à l’aide des informations d’identification de l’administrateur.

2. Dans le volet de navigation supérieur, cliquez sur le bouton **Administrateur**.
   
    ![utilisateur de test 1](./media/marketo-tutorial/tutorial_marketo_06.png) 

3. Accédez au menu **Sécurité** et cliquez sur **Utilisateurs et rôles**
   
    ![utilisateur de test 2](./media/marketo-tutorial/tutorial_marketo_19.png)  

4. Dans l’onglet Utilisateurs, cliquez sur le lien **Inviter un nouvel utilisateur**
   
    ![utilisateur de test 3](./media/marketo-tutorial/tutorial_marketo_15.png) 

5. Dans l’Assistant d’invitation d’un nouvel utilisateur, renseignez les informations suivantes.
   
    a. Entrez l’adresse **e-mail** de l’utilisateur dans la zone de texte.
   
    ![utilisateur de test 4](./media/marketo-tutorial/tutorial_marketo_16.png)
   
    b. Entrez le **prénom** dans la zone de texte.
   
    c. Entrez le **nom** dans la zone de texte
   
    d. Cliquez sur **Suivant**.

6. Dans l’onglet **Autorisations**, sélectionnez les **rôles d’utilisateur** et cliquez sur **Suivant**
   
    ![utilisateur de test 5](./media/marketo-tutorial/tutorial_marketo_17.png)
7. Pour envoyer l’invitation d’utilisateur, cliquez sur le bouton **Envoyer**
   
    ![utilisateur de test 6](./media/marketo-tutorial/tutorial_marketo_18.png)

8. L’utilisateur recevra une notification par e-mail. Pour activer le compte, il devra cliquer sur le lien et modifier le mot de passe. 

### <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester votre configuration de l’authentification unique Azure AD avec les options suivantes.

1. Cliquez sur Tester cette application dans le portail Azure. Vous êtes alors automatiquement connecté à l’instance de Marketo pour laquelle vous avez configuré l’authentification unique.

1. Vous pouvez utiliser le volet d’accès Microsoft. Le fait de cliquer sur la vignette Marketo dans le volet d’accès doit vous connecter automatiquement à l’instance de Marketo pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Étapes suivantes

Après avoir configuré Marketo, vous pouvez appliquer le contrôle de session, qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).