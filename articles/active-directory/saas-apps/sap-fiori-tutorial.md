---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à SAP Fiori | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et SAP Fiori.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/05/2019
ms.author: jeedes
ms.openlocfilehash: 35bb05eb019a1284a8a418df0197960130e8d013
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92897294"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-sap-fiori"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à SAP Fiori

Dans ce tutoriel, vous allez apprendre à intégrer SAP Fiori à Azure Active Directory (Azure AD). Quand vous intégrez SAP Fiori à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à SAP Fiori.
* Permettre à vos utilisateurs de se connecter automatiquement à SAP Fiori avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Abonnement SAP Fiori pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* SAP Fiori prend en charge l’authentification unique lancée par le **fournisseur de services**

> [!NOTE]
> Pour l’authentification iFrame lancée par SAP Fiori, nous vous recommandons d’utiliser le paramètre **IsPassive** dans la requête d’authentification (AuthnRequest) SAML pour l’authentification silencieuse. Pour plus de détails sur le paramètre **IsPassive** , reportez-vous aux informations sur l’ [authentification unique SAML Azure AD](../develop/single-sign-on-saml-protocol.md).

## <a name="adding-sap-fiori-from-the-gallery"></a>Ajout de SAP Fiori à partir de la galerie

Pour configurer l’intégration de SAP Fiori à Azure AD, vous devez ajouter SAP Fiori à partir de la galerie à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie** , tapez **SAP Fiori** dans la zone de recherche.
1. Sélectionnez **SAP Fiori** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-sap-fiori"></a>Configurer et tester l’authentification unique Azure AD pour SAP Fiori

Configurez et testez l’authentification unique Azure AD avec SAP Fiori à l’aide d’un utilisateur de test nommé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur SAP Fiori associé.

Pour configurer et tester l’authentification unique Azure AD avec SAP Fiori, effectuez les modules suivants :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique SAP Fiori](#configure-sap-fiori-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test SAP Fiori](#create-sap-fiori-test-user)** pour avoir un équivalent de B.Simon dans SAP Fiori lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Ouvrez une nouvelle fenêtre de navigateur web et connectez-vous à votre site d’entreprise SAP Fiori en tant qu’administrateur.

1. Vérifiez que les services **http** et **https** sont actifs, et que les ports appropriés sont affectés au code de transaction **SMICM**.

1. Connectez-vous à SAP Business Client pour le système SAP **T01** , où l’authentification unique est requise. Ensuite, activez la gestion de session de sécurité HTTP.

    1. Accédez au code de transaction **SICF_SESSIONS**. Tous les paramètres de profil appropriés sont affichés avec les valeurs actuelles. Ils doivent ressembler à l’exemple qui suit :

        ```
        login/create_sso2_ticket = 2
        login/accept_sso2_ticket = 1
        login/ticketcache_entries_max = 1000
        login/ticketcache_off = 0  login/ticket_only_by_https = 0 
        icf/set_HTTPonly_flag_on_cookies = 3
        icf/user_recheck = 0  http/security_session_timeout = 1800
        http/security_context_cache_size = 2500
        rdisp/plugin_auto_logout = 1800
        rdisp/autothtime = 60
        ```

        >[!NOTE]
        > Ajustez les paramètres en fonction des besoins de votre organisation. Les paramètres précédents sont fournis uniquement à titre d’exemple.

    1. Si nécessaire, ajustez les paramètres dans le profil (par défaut) d’instance du système SAP et redémarrez le système SAP.

    1. Double-cliquez sur le client approprié pour activer la session de sécurité HTTP.

        ![La page de valeurs actuelles des paramètres de profil pertinents dans SAP](./media/sapfiori-tutorial/tutorial-sapnetweaver-profileparameter.png)

    1. Activez les services SICF suivants :

        ```
        /sap/public/bc/sec/saml2
        /sap/public/bc/sec/cdc_ext_service
        /sap/bc/webdynpro/sap/saml2
        /sap/bc/webdynpro/sap/sec_diag_tool (This is only to enable / disable trace)
        ```

1. Accédez au code de transaction **SAML2** dans Business Client pour le système SAP [ **T01/122** ]. L’interface utilisateur de configuration s’ouvre dans une nouvelle fenêtre de navigateur. Dans cet exemple, nous utilisons Business Client pour le système SAP 122.

    ![La page de connexion Business Client SAP Fiori](./media/sapfiori-tutorial/tutorial-sapnetweaver-sapbusinessclient.png)

1. Entrez votre nom d’utilisateur et votre mot de passe, puis sélectionnez **Log on** (Connexion).

    ![La page de configuration SAML 2.0 du système ABAP T01/122 dans SAP](./media/sapfiori-tutorial/tutorial-sapnetweaver-userpwd.png)

1. Dans la zone **Provider Name** (Nom du fournisseur), remplacez **T01122** par **http:\//T01122** , puis sélectionnez **Save** (Enregistrer).

    > [!NOTE]
    > Par défaut, le nom du fournisseur est au format \<sid>\<client>. Azure AD attend le nom au format \<protocol>://\<name>. Nous vous conseillons de conserver https\://\<sid>\<client> pour le nom du fournisseur, afin de pouvoir configurer plusieurs moteurs ABAP SAP Fiori dans Azure AD.

    ![Nom du fournisseur mis à jour dans la page de configuration SAML 2.0 du système ABAP T01/122 dans SAP](./media/sapfiori-tutorial/tutorial-sapnetweaver-providername.png)

1. Sélectionnez l’onglet **Local Provider** > **Metadata** (Fournisseur Local>Métadonnées).

1. Dans la boîte de dialogue **SAML 2.0 Metadata** (Métadonnées SAML 2.0), téléchargez le fichier XML de métadonnées généré et enregistrez-le sur votre ordinateur.

    ![Lien de téléchargement des métadonnées dans la boîte de dialogue de métadonnées SAP SAML 2.0](./media/sapfiori-tutorial/tutorial-sapnetweaver-generatesp.png)

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **SAP Fiori** , recherchez la section **Gérer** et sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , si vous disposez d’un **fichier de métadonnées du fournisseur de services** , suivez les étapes ci-dessous :

    a. Cliquez sur **Charger un fichier de métadonnées**.

    ![Charger le fichier de métadonnées](common/upload-metadata.png)

    b. Cliquez sur le **logo du dossier** pour sélectionner le fichier de métadonnées, puis cliquez sur **Charger**.

    ![choisir le fichier de métadonnées](common/browse-upload-metadata.png)

    c. Une fois le fichier de métadonnées chargé, les valeurs **Identificateur** et **URL de réponse** sont renseignées automatiquement dans le volet **Configuration SAML de base**. Dans la zone **URL de connexion** , entrez une URL au format suivant : `https:\//\<your company instance of SAP Fiori\>`.

    > [!NOTE]
    > Quelques clients ont signalé des erreurs de configuration des valeurs **URL de réponse**. Si vous rencontrez ce genre d’erreur, vous pouvez utiliser le script PowerShell suivant afin de définir l’URL de réponse appropriée pour votre instance :
    >
    > ```
    > Set-AzureADServicePrincipal -ObjectId $ServicePrincipalObjectId -ReplyUrls "<Your Correct Reply URL(s)>"
    > ``` 
    > 
    > Vous pouvez définir l’ID d’objet `ServicePrincipal` vous-même avant d’exécuter le script, ou vous pouvez le passer ici.

1. L’application SAP Fiori attend les assertions SAML dans un format spécifique. Configurez les revendications suivantes pour cette application. Pour gérer ces valeurs d’attributs, dans le volet **Configurer l’authentification unique avec SAML** , sélectionnez **Modifier**.

    ![Volet Attributs utilisateur](common/edit-attribute.png)

1. Dans le volet **Attributs et revendications de l’utilisateur** , configurez les attributs de jetons SAML comme illustré dans l’image précédente. Effectuez ensuite les tâches suivantes :

    1. Sélectionnez **Modifier** pour ouvrir le volet **Gérer les revendications des utilisateurs**.

    1. Dans la liste **Transformation** , sélectionnez **ExtractMailPrefix()** .

    1. Dans la liste **Paramètre 1** , sélectionnez **user.userprincipalname**.

    1. Sélectionnez **Enregistrer**.

       ![Volet Gérer les revendications des utilisateurs](./media/sapfiori-tutorial/nameidattribute.png)

       ![Section Transformation dans le volet Gérer les revendications des utilisateurs](./media/sapfiori-tutorial/nameidattribute1.png)
    
1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **XML de métadonnées de fédération** et sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

1. Dans la section **Configurer SAP Fiori** , copiez l’URL ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à SAP Fiori.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **SAP Fiori**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer**.

## <a name="configure-sap-fiori-sso"></a>Configurer l’authentification unique SAP Fiori

1. Connectez-vous au système SAP et accédez au code de transaction **SAML2**. Une nouvelle fenêtre de navigateur s’ouvre avec la page de configuration SAML.

1. Pour configurer des points de terminaison pour un fournisseur d’identité approuvé (Azure AD), accédez à l’onglet **Trusted Providers** (Fournisseurs approuvés).

    ![Onglet Trusted Providers dans SAP](./media/sapfiori-tutorial/tutorial-sapnetweaver-samlconfig.png)

1. Sélectionnez **Add** (Ajouter) puis **Upload Metadata File** (Charger le fichier de métadonnées) dans le menu contextuel.

    ![Options Add et Upload Metadata File dans SAP](./media/sapfiori-tutorial/tutorial-sapnetweaver-uploadmetadata.png)

1. Chargez le fichier de métadonnées que vous avez téléchargé dans le portail Azure. Sélectionnez **Suivant**.

    ![Sélectionnez le fichier de métadonnées à charger dans SAP](./media/sapfiori-tutorial/tutorial-sapnetweaver-metadatafile.png)

1. Dans la page suivante, dans la zone **Alias** , entrez le nom d’alias. Par exemple, **aadsts**. Sélectionnez **Suivant**.

    ![Zone Alias dans SAP](./media/sapfiori-tutorial/tutorial-sapnetweaver-aliasname.png)

1. Vérifiez que la valeur de la zone **Digest Algorithm** est **SHA-256**. Sélectionnez **Suivant**.

    ![Vérifiez la valeur d’algorithme Digest dans SAP](./media/sapfiori-tutorial/tutorial-sapnetweaver-identityprovider.png)

1. Sous **Single Sign-On Endpoints** (Points de terminaison d’authentification unique), sélectionnez **HTTP POST** , puis **suivant**.

    ![Options Single Sign-On Endpoints dans SAP](./media/sapfiori-tutorial/tutorial-sapnetweaver-httpredirect.png)

1. Sous **Single Logout Endpoints** (Points de terminaison de déconnexion unique), sélectionnez **HTTP Redirect** , puis **suivant**.

    ![Options Single Logout Endpoints dans SAP](./media/sapfiori-tutorial/tutorial-sapnetweaver-httpredirect1.png)

1. Sous **Artifact Endpoints** (Points de terminaison d’artefact), sélectionnez **Suivant** pour continuer.

    ![Options Artifact Endpoints dans SAP](./media/sapfiori-tutorial/tutorial-sapnetweaver-artifactendpoint.png)

1. Sous **Authentication Requirements** (Exigences d’authentification), sélectionnez **Finish** (Terminer).

    ![Options Authentication Requirements et option Finish dans SAP](./media/sapfiori-tutorial/tutorial-sapnetweaver-authentication.png)

1. Sélectionnez **Trusted Provider** > **Identity Federation** (Fournisseurs approuvés>Fédération des identités) en bas de la page. Sélectionnez **Modifier**.

    ![Onglets Trusted Provider et Identity Federation dans SAP](./media/sapfiori-tutorial/tutorial-sapnetweaver-trustedprovider.png)

1. Sélectionnez **Ajouter**.

    ![Option Add sous l’onglet Identity Federation](./media/sapfiori-tutorial/tutorial-sapnetweaver-addidentityprovider.png)

1. Dans la boîte de dialogue **Supported NameID Formats** (Formats NameID pris en charge), sélectionnez **Unspecified** (Non spécifié). Sélectionnez **OK**.

    ![Boîte de dialogue Supported NameID Formats dans SAP](./media/sapfiori-tutorial/tutorial-sapnetweaver-nameid.png)

    Les valeurs **user ID Source** (Source d’ID utilisateur) et **user ID mapping mode** (Mode de mappage d’ID utilisateur) déterminent le lien entre l’utilisateur SAP et la revendication Azure AD.  

    **Scenario 1**  : Mappage d’un utilisateur SAP à un utilisateur Azure AD

    1. Dans SAP, sous **Details of NameID Format "Unspecified"** (Détails du format NameID « Unspecified »), notez les détails :

        ![Capture d’écran montrant la boîte de dialogue « Details of NameID Format "Unspecified" » dans SAP.](./media/sapfiori-tutorial/nameiddetails.png)

    1. Dans le portail Azure, sous **Attributs et revendications de l’utilisateur** , notez les revendications requises à partir d’Azure AD.

        ![Capture d’écran montrant la boîte de dialogue « Attributs utilisateur et revendications ».](./media/sapfiori-tutorial/claimsaad1.png)

    **Scénario 2**  : Sélectionner un ID utilisateur SAP en fonction de l’adresse e-mail configurée dans SU01. Dans ce cas, l’ID de l’adresse e-mail doit être configuré dans SU01 pour chaque utilisateur nécessitant l’authentification unique.

    1.  Dans SAP, sous **Details of NameID Format "Unspecified"** (Détails du format NameID « Unspecified »), notez les détails :

        ![Boîte de dialogue Details of NameID Format "Unspecified" dans SAP](./media/sapfiori-tutorial/tutorial-sapnetweaver-nameiddetails1.png)

    1. Dans le portail Azure, sous **Attributs et revendications de l’utilisateur** , notez les revendications requises à partir d’Azure AD.

       ![Boîte de dialogue Attributs et revendications de l’utilisateur dans le portail Azure](./media/sapfiori-tutorial/claimsaad2.png)

1. Sélectionnez **Save** (Enregistrer), puis **Enable** (Activer) pour activer le fournisseur d’identité.

    ![Options Save et Enable dans SAP](./media/sapfiori-tutorial/configuration1.png)

1. À l’invite, sélectionnez **OK**.

    ![Option OK dans la boîte de dialogue SAML 2.0 Configuration dans SAP](./media/sapfiori-tutorial/configuration2.png)

### <a name="create-sap-fiori-test-user"></a>Créer un utilisateur de test SAP Fiori

Dans cette section, vous allez créer un utilisateur appelé Britta Simon dans SAP Fiori. Collaborez avec votre équipe d’experts SAP en interne ou avec le partenaire SAP de votre organisation pour ajouter l’utilisateur à la plateforme SAP Fiori.

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

1. Une fois le fournisseur d’identité Azure AD activé dans SAP Fiori, essayez d’accéder à l’une des URL suivantes pour tester l’authentification unique (vous ne devriez pas être invité à entrer un nom d’utilisateur et un mot de passe) :

    * https:\//\<sapurl\>/sap/bc/bsp/sap/it00/default.htm
    * https:\//\<sapurl\>/sap/bc/bsp/sap/it00/default.htm

    > [!NOTE]
    > Remplacez *sapurl* par le nom d’hôte SAP réel.

1. L’URL de test doit vous mener à la page d’application de test suivante dans SAP. Si la page s’ouvre, cela signifie que l’authentification unique Azure AD est correctement configurée.

    ![Page d’application de test standard dans SAP](./media/sapfiori-tutorial/testingsso.png)

1. Si vous êtes invité à fournir un nom d’utilisateur et un mot de passe, activez la trace pour vous aider à diagnostiquer le problème. Utilisez l’URL suivante pour la trace : https:\//\<sapurl\>/sap/bc/webdynpro/sap/sec_diag_tool?sap-client=122&sap-language=EN#.

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer SAP Fiori avec Azure AD](https://aad.portal.azure.com/)