---
title: 'Didacticiel : Intégration d’Azure Active Directory avec Zscaler ZSCloud | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Zscaler ZSCloud.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/24/2019
ms.author: jeedes
ms.openlocfilehash: ecaf0c68f2234643d2036c67ae61d53f8763f204
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97608967"
---
# <a name="tutorial-azure-active-directory-integration-with-zscaler-zscloud"></a>Didacticiel : Intégration d’Azure Active Directory avec Zscaler ZSCloud

L’objectif de ce didacticiel est de montrer comment intégrer Zscaler ZSCloud avec Azure Active Directory (Azure AD).
Intégrer Zscaler ZSCloud avec Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler l’accès à Zscaler ZSCloud.
* Vous pouvez permettre à vos utilisateurs de se connecter automatiquement à Zscaler ZSCloud (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD avec Zscaler ZSCloud, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/)
* Un abonnement Zscaler ZSCloud pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Zscaler ZSCloud prend en charge l’authentification unique initiée par le **fournisseur de services**

* Zscaler ZSCloud prend en charge l’attribution d’utilisateurs **juste-à-temps**

## <a name="adding-zscaler-zscloud-from-the-gallery"></a>Ajouter Zscaler ZSCloud à partir de la galerie

Pour configurer l’intégration de Zscaler ZSCloud avec Azure AD, vous devez ajouter Zscaler ZSCloud, disponible dans la galerie, à votre liste d’applications SaaS gérées.

**Pour ajouter Zscaler ZSCloud à partir de la galerie, réalisez les étapes suivantes :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise**, puis sélectionnez l’option **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Zscaler ZSCloud**, sélectionnez **Zscaler ZSCloud** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![Zscaler ZSCloud dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Zscaler ZSCloud grâce à un utilisateur de test appelé **Britta Simon**.
Pour que l’authentification unique fonctionne, une relation doit être établie entre un utilisateur Azure AD et l’utilisateur Zscaler ZSCloud associé.

Pour configurer et tester l’authentification unique Azure AD avec Zscaler ZSCloud, vous devez compléter les blocs de construction suivants :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Zscaler ZSCloud](#configure-zscaler-zscloud-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Zscaler ZSCloud](#create-zscaler-zscloud-test-user)** pour avoir un équivalent de Britta Simon dans Zscaler ZSCloud qui est lié à la représentation de l’utilisateur Azure AD.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Zscaler ZSCloud, effectuez les étapes suivantes :

1. Sur le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Zscaler ZSCloud**, sélectionnez **Authentification unique**.

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique**, sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**.

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base**, effectuez les étapes suivantes :

    ![Informations d’authentification unique dans Domaine et URL Zscaler ZSCloud](common/sp-signonurl.png)

    Dans la zone de texte **URL de connexion**, entrez l’URL utilisée par vos utilisateurs pour se connecter à votre application Zscaler ZSCloud.

    > [!NOTE]
    > Vous devez remplacer la valeur par l’URL de connexion. Pour obtenir cette valeur, contactez l’[équipe du support technique Zscaler ZSCloud](https://help.zscaler.com/). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

5. Votre application Zscaler ZSCloud s’attend à recevoir les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à votre configuration d’attributs du jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut. Cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Attributs d’utilisateur**.

    ![Capture d’écran montrant User Attributes avec l’icône Edit sélectionnée.](common/edit-attribute.png)

6. En plus de ce qui précède, l’application Zscaler ZSCloud s’attend à ce que quelques attributs supplémentaires soient repassés dans la réponse SAML. Dans la section **Revendications des utilisateurs** de la boîte de dialogue **Attributs utilisateur**, effectuez les étapes suivantes pour ajouter le jeton SAML comme indiqué dans le tableau ci-dessous :
    
    | Name | Attribut source |
    | ---------| ------------ |
    | memberOf | user.assignedroles |

    a. Cliquez sur le bouton **Ajouter une nouvelle revendication** pour ouvrir la boîte de dialogue **Gérer les revendications des utilisateurs**.

    ![Capture d’écran montrant User claims avec l’option permettant d’ajouter une nouvelle revendication.](common/new-save-attribute.png)

    ![Capture d’écran montrant la boîte de dialogue Manage user claims où vous pouvez entrer les valeurs décrites.](common/new-attribute-details.png)

    b. Dans la zone de texte **Attribut**, indiquez le nom d’attribut pour cette ligne.

    c. Laissez le champ **Espace de noms** vide.

    d. Sélectionnez Source comme **Attribut**.

    e. Dans la liste **Attribut de la source**, tapez la valeur d’attribut indiquée pour cette ligne.
    
    f. Cliquez sur **Enregistrer**.

    > [!NOTE]
    > Cliquez [ici](../develop/active-directory-enterprise-app-role-management.md) pour savoir comment configurer un rôle dans Azure AD.

7. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** en fonction des options définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

8. Dans la section **Configurer Zscaler ZSCloud**, copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-zscaler-zscloud-single-sign-on"></a>Configurer l’authentification unique Zscaler ZSCloud

1. Pour automatiser la configuration dans Zscaler ZSCloud, vous devez installer l’**extension de navigateur My Apps Secure Sign-in** en cliquant sur **Installer l’extension**.

    ![Extension My apps](common/install-myappssecure-extension.png)

2. Après l’ajout de l’extension au navigateur, cliquez sur **Configurer Zscaler ZSCloud** pour être dirigé vers l’application Zscaler ZSCloud. À partir de là, indiquez les informations d’identification de l’administrateur pour vous connecter à Zscaler ZSCloud. Cette extension de navigateur configure automatiquement l’application et automatise les étapes 3 à 6.

    ![Configurer l’authentification unique](common/setup-sso.png)

3. Si vous souhaitez configurer manuellement Zscaler ZSCloud, ouvrez une nouvelle fenêtre de navigateur web, connectez-vous à votre site d’entreprise Zscaler ZSCloud en tant qu’administrateur, puis effectuez les étapes suivantes :

4. Sélectionnez **Administration > Authentification > Paramètres d'authentification** et procédez comme suit :
   
    ![Capture d’écran montrant le site Zscaler avec les étapes décrites.](./media/zscaler-zscloud-tutorial/ic800206.png "Administration")

    a. Sous Type d'authentification, choisissez **SAML**.

    b. Cliquez sur **Configure SAML**.

5. Dans la fenêtre **Modifier les paramètres SAML**, procédez comme suit, puis cliquez sur Enregistrer.  
            
    ![Gérer les utilisateurs et l’authentification](./media/zscaler-zscloud-tutorial/ic800208.png "Manage Users & Authentication")
    
    a. Dans la zone de texte **URL du portail SAML**, collez l'**URL de connexion** que vous avez copiée sur le portail Azure.

    b. Dans la zone de texte **Attribut de l'ID de connexion**, entrez **NameID**.

    c. Cliquez sur **Charger** pour charger le certificat de signature SAML Azure que vous avez téléchargé depuis le portail Azure dans le **Certificat SSL public**.

    d. Activez/désactivez **Activer l'approvisionnement automatique SAML**.

    e. Dans la zone de texte **Attribut du nom d'affichage de l'utilisateur**, entrez **displayName** si vous souhaitez activer l'approvisionnement automatique SAML pour les attributs displayName.

    f. Dans la zone de texte **Attribut du nom de groupe**, entrez **memberOf** si vous souhaitez activer l'approvisionnement automatique SAML pour les attributs memberOf.

    g. Dans **Attribut du nom du service**, entrez **department** si vous souhaitez activer l'approvisionnement automatique SAML pour les attributs department.

    h. Cliquez sur **Enregistrer**.

6. Dans la page **Configure User Authentication** , procédez comme suit :

    ![Capture d’écran montrant la boîte de dialogue Configure User Authentication avec l’option Activate sélectionnée.](./media/zscaler-zscloud-tutorial/ic800207.png)

    a. Passez la souris sur le menu **Activation** en bas à gauche.

    b. Cliquez sur **Activer**.

## <a name="configuring-proxy-settings"></a>Configuration des paramètres de proxy
### <a name="to-configure-the-proxy-settings-in-internet-explorer"></a>Pour configurer les paramètres de proxy dans Internet Explorer

1. Démarrez **Internet Explorer**.

2. Pour ouvrir la boîte de dialogue **Options Internet**, sélectionnez **Options Internet** dans le menu **Outils**.   
    
     ![Options Internet](./media/zscaler-zscloud-tutorial/ic769492.png "Options Internet")

3. Cliquez sur l’onglet **Connexions** .   
  
     ![Connexions](./media/zscaler-zscloud-tutorial/ic769493.png "Connexions")

4. Cliquez sur **Paramètres réseau** pour ouvrir la boîte de dialogue **Paramètres réseau**.

5. Dans la section Serveur proxy, procédez comme suit :   
   
    ![Serveur proxy](./media/zscaler-zscloud-tutorial/ic769494.png "Serveur proxy")

    a. Sélectionnez **Utiliser un serveur proxy pour le réseau local**.

    b. Dans la zone de texte Adresse, tapez **gateway.Zscaler ZSCloud.net**.

    c. Dans la zone de texte Port, tapez **80**.

    d. Sélectionnez **Ne pas utiliser de serveur proxy pour les adresses locales**.

    e. Cliquez sur **OK** pour fermer la boîte de dialogue **Paramètres du réseau local**.

6. Cliquez sur **OK** pour fermer la boîte de dialogue **Options Internet**.

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD 

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory**, sélectionnez **Utilisateurs**, puis sélectionnez **Tous les utilisateurs**.

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom**, entrez **BrittaSimon**.
  
    b. Dans le champ **Nom d’utilisateur**, tapez brittasimon@yourcompanydomain.extension. Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe**, puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Zscaler ZSCloud.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, **Toutes les applications**, puis **Zscaler ZSCloud**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Zscaler ZSCloud**.

    ![Lien Zscaler ZSCloud dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes**.

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur**, puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez l’utilisateur tel que **Britta Simon** dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

    ![Capture d’écran montrant la boîte de dialogue Utilisateurs et groupes où vous pouvez sélectionner un utilisateur.](./media/zscaler-zscloud-tutorial/tutorial_zscalerzscloud_users.png)

6. Dans la boîte de dialogue **Sélectionner un rôle**, choisissez le rôle d’utilisateur approprié dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

    ![Capture d’écran montrant la boîte de dialogue Sélectionner un rôle où vous pouvez sélectionner un rôle d’utilisateur.](./media/zscaler-zscloud-tutorial/tutorial_zscalerzscloud_roles.png)

7. Dans la boîte de dialogue **Ajouter une attribution**, sélectionnez le bouton **Attribuer**.

    ![Capture d’écran montrant la boîte de dialogue Add Assignment dans laquelle vous pouvez sélectionner Assign.](./media/zscaler-zscloud-tutorial/tutorial_zscalerzscloud_assign.png)

    >[!NOTE]
    >Le rôle d’accès par défaut n’est pas pris en charge, car il empêcherait le provisionnement. Il ne peut donc pas être sélectionné lors de l’affectation d’utilisateurs.

### <a name="create-zscaler-zscloud-test-user"></a>Créer un utilisateur de test Zscaler ZSCloud

Dans cette section, un utilisateur appelé Britta Simon est créé dans Zscaler ZSCloud. Zscaler ZSCloud prend en charge l’attribution d’utilisateurs juste-à-temps, qui est activée par défaut. Vous n’avez aucune opération à effectuer dans cette section. S’il n’existe pas encore d’utilisateurs dans Zscaler ZSCloud, un utilisateur est créé après l’authentification.

>[!Note]
>Si vous avez besoin de créer un utilisateur manuellement, contactez l’[équipe du support technique Zscaler ZSCloud](https://help.zscaler.com/).

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Zscaler ZSCloud dans le volet d’accès, vous devez vous connecter automatiquement à l’application Zscaler ZSCloud pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)