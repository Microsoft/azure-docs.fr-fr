---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à DocuSign | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique (SSO) entre Azure Active Directory et DocuSign.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/09/2020
ms.author: jeedes
ms.openlocfilehash: 00d4381c7af7fdf82ee1e895072d92d1e641f8c4
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92454692"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-docusign"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à DocuSign

Dans ce tutoriel, vous allez découvrir comment intégrer DocuSign à Microsoft Azure Active Directory (Azure AD). Quand vous intégrez DocuSign à Azure AD, vous pouvez :

* Utiliser Azure AD pour contrôler qui a accès à DocuSign.
* Permettre à vos utilisateurs de se connecter automatiquement à DocuSign par le biais de leurs comptes Azure AD.
* gérer vos comptes à un emplacement central : le portail Azure.

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement DocuSign pour lequel l’authentification unique est activée.

> [!NOTE]
> Cette intégration peut également être utilisée à partir de l’environnement cloud US Government Azure AD. Cette application est disponible dans la Galerie d’applications cloud US Government Azure AD et peut être configurée de la même façon que dans le cloud public.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test pour vérifier que :

* DocuSign prend en charge l’authentification unique lancée par le **fournisseur de services** .

* DocuSign prend en charge le provisionnement d’utilisateurs **juste-à-temps** .

* DocuSign prend en charge le [provisionnement automatique d’utilisateurs](./docusign-provisioning-tutorial.md).

* Une fois que vous avez configuré DocuSign, vous pouvez appliquer le contrôle de session, qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrir comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad)

## <a name="adding-docusign-from-the-gallery"></a>Ajout de DocuSign à partir de la galerie

Pour configurer l’intégration de DocuSign à Azure AD, vous devez ajouter DocuSign à partir de la galerie à votre liste d’applications SaaS managées.

1. Connectez-vous au portail Azure soit avec un compte professionnel ou scolaire, soit avec un compte Microsoft personnel.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **DocuSign** dans la zone de recherche.
1. Sélectionnez **DocuSign** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-sso-for-docusign"></a>Configurer et tester l’authentification unique Azure AD pour DocuSign

Configurez et testez l’authentification unique Azure AD avec DocuSign au moyen d’un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur DocuSign correspondant.

Pour configurer et tester l’authentification unique Azure AD avec DocuSign, effectuez les étapes suivantes :

1. [Configurer l’authentification unique Azure AD](#configure-azure-ad-sso) pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. [Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user) pour tester l’authentification unique Azure AD avec B.Simon.
    1. [Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user) pour permettre à B.Simon d’utiliser l’authentification unique Azure AD.
1. [Configurer DocuSign](#configure-docusign-sso) pour configurer les paramètres de l’authentification unique côté application.
    1. [Créer un utilisateur de test DocuSign](#create-docusign-test-user) pour générer un équivalent de B.Simon dans DocuSign lié à la représentation Azure AD associée.
1. [Tester l’authentification unique (SSO)](#test-sso) pour vérifier que la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Pour activer l’authentification unique Azure AD dans le portail Azure, effectuez les étapes suivantes :

1. Dans le portail Azure, accédez à la page d’intégration de l’application **DocuSign** , recherchez la section **Gérer** , puis sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , sélectionnez l’icône de stylet pour **Configuration SAML de base** afin de modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , effectuez les étapes suivantes :

    a. Dans la zone de texte **URL de connexion** , entrez une URL au format suivant :

    `https://<subdomain>.docusign.com/organizations/<OrganizationID>/saml2/login/sp/<IDPID>`

    b. Dans la zone de texte **Identificateur (ID d’entité)** , entrez une URL au format suivant :

    `https://<subdomain>.docusign.com/organizations/<OrganizationID>/saml2`

    c. Dans la zone de texte **URL de réponse** , entrez l’un des modèles d’URL suivants :
    
    | URL de réponse |
    |-------------|
    |`https://<subdomain>.docusign.com/organizations/<OrganizationID>/saml2/login/<IDPID>`|
    |`https://<subdomain>.docusign.net/SAML/`|

    > [!NOTE]
    > Ces valeurs entre crochets sont des espaces réservés. Remplacez ces valeurs par les valeurs réelles de l’URL de connexion, de l’identificateur et de l’URL de réponse. Ces détails sont expliqués dans la section « Afficher les points de terminaison SAML 2.0 » plus loin dans ce tutoriel.

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (Base64)** . Sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer DocuSign** , copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test nommé B.Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , **Utilisateurs** , puis **Tous les utilisateurs** .
1. Sélectionnez **Nouvel utilisateur** , en haut de l’écran.
1. Dans les propriétés **Utilisateur** , effectuez les étapes suivantes :
   1. Dans le champ **Nom** , entrez **B.Simon** .  
   1. Dans le champ **Nom de l’utilisateur** , entrez `<username>@<companydomain>.<extension>`. Par exemple : `B.Simon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ **Mot de passe** .
   1. Sélectionnez **Create** (Créer).

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez accorder à B.Simon l’accès à DocuSign afin que cet utilisateur puisse utiliser l’authentification unique Azure.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **DocuSign** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .
1. Sélectionnez **Ajouter un utilisateur** puis, dans la boîte de dialogue **Ajouter une attribution** , sélectionnez **Utilisateurs et groupes** .
1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B.Simon** dans la liste **Utilisateurs** , puis appuyez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez qu’un rôle soit attribué aux utilisateurs, vous pouvez le sélectionner dans la liste déroulante **Sélectionner un rôle** . Si aucun rôle n’a été configuré pour cette application, vous voyez le rôle « Accès par défaut » sélectionné.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-docusign-sso"></a>Configurer l’authentification unique DocuSign

1. Pour automatiser la configuration dans DocuSign, installez l’extension de navigateur My Apps Secure Sign-in en sélectionnant **Installer l’extension** .

    ![Extension My apps](common/install-myappssecure-extension.png)

2. Après avoir ajouté l’extension au navigateur, sélectionnez **Configurer DocuSign** . Vous êtes dirigé vers l’application DocuSign. À partir de là, indiquez les informations d’identification de l’administrateur pour vous connecter à DocuSign. L’extension de navigateur configure automatiquement l’application et automatise les étapes 3 à 5.

    ![Configuration](common/setup-sso.png)

3. Si vous souhaitez configurer manuellement DocuSign, ouvrez une nouvelle fenêtre de navigateur web et connectez-vous à votre site d’entreprise DocuSign en tant qu’administrateur.

4. Dans le coin supérieur droit de la page, sélectionnez le logo du profil, puis sélectionnez **Go to Admin** .
  
    ![Go to Admin sous le profil][51]

5. Dans la page des solutions de votre domaine, sélectionnez **Domains** .

    ![Domaines/solutions de domaine][50]

6. Dans la section **Domains** , sélectionnez **CLAIM DOMAIN** (Revendiquer un domaine).

    ![Option Claim Domain][52]

7. Dans la boîte de dialogue de **Claim a Domain** (Revendiquer un domaine), dans la zone de texte **Domain Name** (Nom du domaine), indiquez votre domaine d’entreprise, puis cliquez sur **CLAIM** (Revendiquer). Veillez à vérifier le domaine et assurez-vous que son état est actif.

    ![Boîte de dialogue Claim a Domain/Domain Name][53]

8. Dans la page de solutions de domaine, sélectionnez **Identity Providers** (Fournisseurs d’identité).
  
    ![Option Identity Providers][54]

9. Dans la section **Identity Providers** , cliquez sur **ADD IDENTITY PROVIDER** (Ajouter un fournisseur d’identité).

    ![Option Add Identity Provider][55]

10. Dans la page **Identity Provider Settings** (Paramètres du fournisseur d’identité), effectuez les étapes suivantes :

    ![Champs Identity Provider Settings][56]

    a. Dans la zone de texte **Name** (Nom) , tapez le nom unique de votre configuration. N’utilisez pas d’espaces.

    b. Dans la zone **Identity Provider Issuer** (Émetteur du fournisseur d’identité), collez la valeur **Identificateur Azure AD** que vous avez copiée à partir du portail Azure.

    c. Dans la zone **Identity Provider Login URL** (URL de connexion du fournisseur identité), collez la valeur **URL de connexion** que vous avez copiée à partir du portail Azure.

    d. Dans la zone **Identity Provider Logout URL** (URL de déconnexion du fournisseur d’identité), collez la valeur **URL de déconnexion** que vous avez copiée à partir du portail Azure.

    e. Sélectionnez **Signer la demande d’authentification** .

    f. Pour **Send AuthN request by** (Envoyer la requête d’authentification par), sélectionnez **POST** .

    g. Pour **Send logout request by** (Envoyer la requête de déconnexion par), sélectionnez **GET** .

    h. Dans la section **Custom Attribute Mapping** (Mappage des attributs personnalisés), sélectionnez **ADD NEW MAPPING** (Ajouter un nouveau mappage).

       ![Interface utilisateur de mappage des attributs personnalisés][62]

    i. Choisissez le champ que vous voulez mapper à la revendication Azure AD. Dans cet exemple, la revendication **emailaddress** est mappée à la valeur de `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`. Il s’agit du nom de revendication par défaut d’Azure AD pour la revendication d’e-mail. Sélectionnez **SAVE** (Enregistrer).

       ![Champs de mappage des attributs personnalisés][57]

       > [!NOTE]
       > Utilisez l’ **identificateur d’utilisateur** approprié pour mapper l’utilisateur d’Azure AD au mappage utilisateur DocuSign. Sélectionnez le champ correct et entrez la valeur appropriée en fonction des paramètres de votre organisation.

    j. Dans la section **Identity Provider Certificates** (Certificats du fournisseur d’identité), sélectionnez **ADD CERTIFICATE** (Ajouter un certificat), puis chargez le certificat que vous avez téléchargé à partir du portail Azure AD et cliquez sur **SAVE** (Enregistrer).

       ![Identity Provider Certificates/Add Certificate][58]

    k. Dans la section **Identity Providers** (Fournisseurs d’identité), sélectionnez **ACTIONS** , puis **Endpoints** (Points de terminaison).

       ![Identity Providers/Endpoints][59]

    l. Dans la section **View SAML 2.0 Endpoints** (Afficher les points de terminaison SAML 2.0) du portail d’administration DocuSign, effectuez les étapes suivantes :

       ![Afficher les points de terminaison SAML 2.0][60]
       
       1. Copiez la valeur **Service Provider Issuer URL** (URL de l’émetteur du fournisseur d’identité) et collez-la dans la zone **Identificateur** de la section **Configuration SAML de base** du portail Azure.
       
       1. Copiez la valeur **Service Provider Assertion Consumer Service URL** (URL du service consommateur d’assertion) et collez-la dans la zone **URL de réponse** de la section **Configuration SAML de base** dans le Portail Azure.
       
       1. Copiez la valeur **Service Provider Login URL** (URL de connexion du fournisseur de services) et collez-la dans la zone **URL de connexion** de la section **Configuration SAML de base** du portail Azure. La valeur IDPID est indiquée à la fin de **Service Provider Login URL** .

       1. Sélectionnez **Fermer** .

### <a name="create-docusign-test-user"></a>Créer un utilisateur de test DocuSign

Dans cette section, un utilisateur appelé B.Simon est créé dans DocuSign. DocuSign prend en charge le provisionnement d’utilisateurs juste-à-temps, option activée par défaut. Vous n’avez aucune opération à effectuer dans cette section. S’il n’existe pas encore d’utilisateur dans DocuSign, il est créé après l’authentification.

> [!Note]
> Si vous devez créer un utilisateur manuellement, contactez l’[équipe du support DocuSign](https://support.docusign.com/).

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester votre configuration de l’authentification unique Azure AD avec les options suivantes. 

1. Cliquez sur **Tester cette application** dans le portail Azure. Cette opération redirige vers l’URL de connexion DocuSign où vous pouvez lancer le flux de connexion. 

2. Accédez directement à l’URL de connexion DocuSign pour lancer le flux de connexion à partir de là.

3. Vous pouvez utiliser le volet d’accès Microsoft. Le fait de cliquer sur la vignette DocuSign dans le volet d’accès doit vous connecter automatiquement à l’application DocuSign pour laquelle vous avez configuré l’authentification unique (SSO). Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).


## <a name="next-steps"></a>Étapes suivantes

Une fois que vous avez configuré DocuSign, vous pouvez appliquer le contrôle de session, qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrir comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad)

<!--Image references-->

[50]: ./media/docusign-tutorial/tutorial_docusign_18.png
[51]: ./media/docusign-tutorial/tutorial_docusign_21.png
[52]: ./media/docusign-tutorial/tutorial_docusign_22.png
[53]: ./media/docusign-tutorial/tutorial_docusign_23.png
[54]: ./media/docusign-tutorial/tutorial_docusign_19.png
[55]: ./media/docusign-tutorial/tutorial_docusign_20.png
[56]: ./media/docusign-tutorial/tutorial_docusign_24.png
[57]: ./media/docusign-tutorial/tutorial_docusign_25.png
[58]: ./media/docusign-tutorial/tutorial_docusign_26.png
[59]: ./media/docusign-tutorial/tutorial_docusign_27.png
[60]: ./media/docusign-tutorial/tutorial_docusign_28.png
[61]: ./media/docusign-tutorial/tutorial_docusign_29.png
[62]: ./media/docusign-tutorial/tutorial_docusign_30.png