---
title: 'Tutoriel : Intégration d’Azure Active Directory à Palo Alto Networks Captive Portal | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Palo Alto Networks Captive Portal.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/10/2020
ms.author: jeedes
ms.openlocfilehash: 66e363b43bf9028e3075efa0f5a1f54e6bda190d
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92512685"
---
# <a name="tutorial-azure-active-directory-integration-with-palo-alto-networks-captive-portal"></a>Tutoriel : Intégration d’Azure Active Directory à Palo Alto Networks Captive Portal

Dans ce tutoriel, vous découvrez comment intégrer Palo Alto Networks Captive Portal à Azure Active Directory (Azure AD).
L’intégration de Palo Alto Networks - Captive Portal avec Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Palo Alto Networks Captive Portal.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à Palo Alto Networks Captive Portal (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

## <a name="prerequisites"></a>Prérequis

Pour intégrer Azure AD à Palo Alto Networks Captive Portal, vous avez besoin des éléments suivants :

* Un abonnement Azure Active Directory. Si vous n’avez pas Azure AD, vous pouvez [vous inscrire pour un essai d’un mois](https://azure.microsoft.com/pricing/free-trial/).
* Un abonnement Palo Alto Networks Captive Portal pour lequel l’authentification unique (SSO) est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Palo Alto Networks Captive Portal prend en charge l’authentification unique initiée par le **fournisseur d’identité**
* Palo Alto Networks Captive Portal prend en charge l’attribution d’utilisateurs **juste-à-temps** .

## <a name="adding-palo-alto-networks-captive-portal-from-the-gallery"></a>Ajout de Palo Alto Networks Captive Portal depuis la galerie

Pour configurer l’intégration de Palo Alto Networks - Captive Portal avec Azure AD, vous devez ajouter Palo Alto Networks Captive Portal à partir de la galerie à votre liste d’applications SaaS gérées.

1. Connectez-vous au portail Azure avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , entrez **Palo Alto Networks Captive Portal** dans la zone de recherche.
1. Sélectionnez **Palo Alto Networks Captive Portal** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-sso"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous configurez et vous testez l’authentification unique Azure AD avec Palo Alto Networks Captive Portal sur un utilisateur de test nommé **B.Simon** .
Pour que l’authentification unique fonctionne, une relation doit être établie entre un utilisateur Azure AD et l’utilisateur Palo Alto Networks Captive Portal associé.

Pour configurer et tester l’authentification unique Azure AD avec Palo Alto Networks Captive Portal, procédez comme suit :

1. **[Configurer l’authentification unique (SSO) Azure AD](#configure-azure-ad-sso)** - Permettre aux utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** - Tester l’authentification unique Azure AD avec l’utilisateur B.Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** - Configurer B.Simon pour utiliser l’authentification unique Azure AD.
2. **[Configurer l’authentification unique Palo Alto Networks Captive Portal](#configure-palo-alto-networks-captive-portal-sso)** - Configurer les paramètres de l’authentification unique dans l’application.
    * **[Créer un utilisateur de test Palo Alto Networks Captive Portal](#create-a-palo-alto-networks-captive-portal-test-user)** - Obtenir un équivalent de B.Simon dans Palo Alto Networks Captive Portal lié à la représentation Azure AD associée.
3. **[Tester l’authentification unique](#test-sso)** - Vérifier que la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le portail Azure, dans la page d’intégration de l’application **Palo Alto Networks Captive Portal** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans le volet **Configuration SAML de base** , procédez comme suit :

   1. Pour **Identificateur** , entrez une URL au format `https://<customer_firewall_host_name>/SAML20/SP`.

   2. Pour **URL de réponse** , entrez une URL au format `https://<customer_firewall_host_name>/SAML20/SP/ACS`.

      > [!NOTE]
      > Mettez à jour les valeurs d’espace réservé à cette étape avec les URL d’identificateur et de réponse réelles. Pour obtenir les valeurs réelles, contactez l’[équipe de support client Palo Alto Networks Captive Portal](https://support.paloaltonetworks.com/support).

5. Dans la section **Certificat de signature SAML** , en regard de **XML de métadonnées de fédération** , sélectionnez **Télécharger** . Enregistrez le fichier téléchargé sur votre ordinateur.

    ![Lien de téléchargement du XML de métadonnées de fédération](common/metadataxml.png)

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

Dans cette section, vous autoriserez B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Palo Alto Networks Captive Portal.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Palo Alto Networks Captive Portal** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .
1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .
1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez qu’un rôle soit attribué aux utilisateurs, vous pouvez le sélectionner dans la liste déroulante **Sélectionner un rôle** . Si aucun rôle n’a été configuré pour cette application, vous voyez le rôle « Accès par défaut » sélectionné.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-palo-alto-networks-captive-portal-sso"></a>Configurer l’authentification unique Palo Alto Networks Captive Portal

Ensuite, configurez l’authentification unique dans Palo Alto Networks Captive Portal :

1. Dans une autre fenêtre de navigateur, connectez-vous au site web Palo Alto Networks en tant qu’administrateur.

2. Sélectionnez l’onglet **Appareil** .

    ![Onglet Device (Appareil) sur le site web Palo Alto Networks](./media/paloaltonetworks-captiveportal-tutorial/tutorial_paloaltoadmin_admin1.png)

3. Dans le menu, sélectionnez **SAML Identity Provider** (Fournisseur d’identité SAML), puis sélectionnez **Import** .

    ![Bouton Import](./media/paloaltonetworks-captiveportal-tutorial/tutorial_paloaltoadmin_admin2.png)

4. Dans la boîte de dialogue **SAML Identity Provider Server Profile Import** (Importer le profil de serveur du fournisseur d’identité SAML), effectuez les étapes suivantes :

    ![Configurer l’authentification unique Palo Alto Networks](./media/paloaltonetworks-captiveportal-tutorial/tutorial_paloaltoadmin_admin3.png)

    1. Pour **Profile Name** (Nom du profil), entrez un nom, par exemple, **AzureAD-CaptivePortal** .
    
    2. En regard d’ **Identity Provider Metadata** (Métadonnées du fournisseur d’identité), sélectionnez **Browse** (Parcourir). Sélectionnez le fichier metadata.xml que vous avez téléchargé dans le portail Azure.
    
    3. Sélectionnez **OK** .

### <a name="create-a-palo-alto-networks-captive-portal-test-user"></a>Créer un utilisateur de test Palo Alto Networks Captive Portal

Ensuite, créez un utilisateur nommé *Britta Simon* dans Palo Alto Networks Captive Portal. Palo Alto Networks Captive Portal prend en charge l’attribution d’utilisateurs juste-à-temps, qui est activée par défaut. Vous n’avez pas besoin d’effectuer de tâches dans cette section. S’il n’existe pas encore d’utilisateur dans Palo Alto Networks Captive Portal, il en est créé un après l’authentification.

> [!NOTE]
> Si vous souhaitez créer un utilisateur manuellement, contactez l’[équipe de support client de Palo Alto Networks Captive Portal](https://support.paloaltonetworks.com/support).

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester votre configuration de l’authentification unique Azure AD avec les options suivantes.

Cliquez sur Tester cette application dans le portail Azure. Vous devriez être connecté automatiquement à l’instance Palo Alto Networks Captive Portal pour laquelle vous avez configuré l’authentification unique.

Vous pouvez utiliser le volet d’accès Microsoft. Lorsque vous cliquez sur la vignette Palo Alto Networks Captive Portal dans le panneau d’accès, vous devez être connecté automatiquement à l’application Palo Alto Networks Captive Portal pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Étapes suivantes

Après avoir configuré Palo Alto Networks Captive Portal, vous pouvez appliquer le contrôle de session, qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).