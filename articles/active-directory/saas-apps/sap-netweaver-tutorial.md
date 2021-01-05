---
title: 'Tutoriel : Tutoriel : Intégration de l’authentification unique Azure Active Directory à SAP NetWeaver | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et SAP NetWeaver.
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
ms.openlocfilehash: 688c496239becad7d1462f41afee9c6f88af9f95
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92676683"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-sap-netweaver"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à SAP NetWeaver

Dans ce tutoriel, vous allez découvrir comment intégrer SAP NetWeaver à Azure Active Directory (Azure AD). Quand vous intégrez SAP NetWeaver à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à SAP NetWeaver.
* Permettre à vos utilisateurs de se connecter automatiquement à SAP NetWeaver avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement SAP NetWeaver pour lequel l’authentification unique est activée.
* SAP NetWeaver V7.20 (version minimale requise)

## <a name="scenario-description"></a>Description du scénario

* SAP NetWeaver prend en charge **SAML** ( **Authentification unique lancée par le fournisseur de services** ) et **OAuth** . Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test. 

> [!NOTE]
> L’identificateur de cette application étant une valeur de chaîne fixe, une seule instance peut être configurée dans un locataire.

> [!NOTE]
> Configurez l’application dans SAML ou dans OAuth conformément aux exigences de votre organisation. 

## <a name="adding-sap-netweaver-from-the-gallery"></a>Ajout de SAP NetWeaver à partir de la galerie

Pour configurer l’intégration de SAP NetWeaver à Azure AD, vous devez ajouter SAP NetWeaver depuis la galerie à votre liste d’applications SaaS gérées.

1. Connectez-vous au portail Azure avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **SAP NetWeaver** dans la zone de recherche.
1. Sélectionnez **SAP NetWeaver** dans le volet des résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-sso-for-sap-netweaver"></a>Configurer et tester l’authentification unique Azure AD pour NetWeaver

Configurez et testez l’authentification unique Azure AD avec SAP NetWeaver pour un utilisateur de test nommé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur SAP NetWeaver associé.

Pour configurer et tester l’authentification unique Azure AD avec NetWeaver, effectuez les étapes suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer SAP NetWeaver avec SAML](#configure-sap-netweaver-using-saml)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test SAP NetWeaver](#create-sap-netweaver-test-user)** pour avoir un équivalent de B.Simon dans SAP NetWeaver lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.
1. **[Configurer SAP NetWeaver pour OAuth](#configure-sap-netweaver-for-oauth)** pour configurer les paramètres OAuth côté application.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD auprès de SAP NetWeaver, effectuez les étapes suivantes :

1. Ouvrez une nouvelle fenêtre de navigateur web et connectez-vous à votre site d’entreprise SAP NetWeaver en tant qu’administrateur.

1. Vérifiez que les services **http** et **https** sont actifs et que les ports appropriés sont affectés dans le T-Code **SMICM** .

1. Connectez-vous au client d’entreprise de SAP System (T01), où l’authentification unique est nécessaire, et activez HTTP Security Session Management (Gestion des sessions avec la sécurité HTTP).

    a. Accédez au code Transaction **SICF_SESSIONS** . Tous les paramètres de profil appropriés sont affichés avec les valeurs actuelles. Ils se présentent comme ceci :
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
    > Ajustez les paramètres ci-dessus conformément aux besoins de votre organisation. Les paramètres ci-dessus sont fournis ici à titre indicatif.

    b. Si nécessaire, ajustez les paramètres dans l’instance ou le profil par défaut du système SAP, et redémarrez le système SAP.

    c. Double-cliquez sur le client approprié pour activer la session de sécurité HTTP.

    ![Session de sécurité HTTP ](./media/sapnetweaver-tutorial/tutorial_sapnetweaver_profileparameter.png)

    d. Activez les services SICF ci-dessous :
    ```
    /sap/public/bc/sec/saml2
    /sap/public/bc/sec/cdc_ext_service
    /sap/bc/webdynpro/sap/saml2
    /sap/bc/webdynpro/sap/sec_diag_tool (This is only to enable / disable trace)
    ```
1. Accédez au code Transaction **SAML2** dans le client d’entreprise du système SAP [T01/122]. Une interface utilisateur s’ouvre dans un navigateur. Dans cet exemple, nous utilisons 122 comme client d’entreprise SAP.

    ![Code de transaction](./media/sapnetweaver-tutorial/tutorial_sapnetweaver_sapbusinessclient.png)

1. Indiquez vos nom d’utilisateur et mot de passe à entrer dans l’interface utilisateur, puis cliquez sur **Edit** (Modifier).

    ![nom d’utilisateur et mot de passe](./media/sapnetweaver-tutorial/tutorial_sapnetweaver_userpwd.png)

1. Dans **Provider Name** (nom du fournisseur), remplacez T01122 par `http://T01122`, puis cliquez sur **Save** (Enregistrer).

    > [!NOTE]
    > Par défaut, le nom du fournisseur est au format `<sid><client>`, mais Azure AD attend le nom au format `<protocol>://<name>`. Il est donc recommandé de conserver le nom du fournisseur au format `https://<sid><client>` pour autoriser la configuration de plusieurs moteurs ABAP SAP NetWeaver dans Azure AD.

    ![Plusieurs moteurs SAP NetWeaver ABAP](./media/sapnetweaver-tutorial/tutorial_sapnetweaver_providername.png)

1. **Génération des métadonnées du fournisseur de services**  : une fois la configuration du **Local Provider** (fournisseur local) et des **Trusted Providers** (fournisseurs approuvés) terminée dans l’interface utilisateur de SAML 2.0, l’étape suivante consiste à générer le fichier de métadonnées du fournisseur de services qui contient l’ensemble des paramètres, des contextes de l’authentification et des autres configurations dans SAP. Une fois ce fichier généré, nous devons le charger sur Azure AD.

    ![Création de métadonnées du fournisseur de service](./media/sapnetweaver-tutorial/tutorial_sapnetweaver_generatesp.png)

    a. Accédez à l’onglet **Local Provider** (Fournisseur local).

    b. Cliquez sur **Metadata** (Métadonnées).

    c. Enregistrez le **fichier XML de métadonnées** sur votre ordinateur, puis chargez-le dans la section **Configuration SAML de base** pour renseigner automatiquement les valeurs **Identificateur** et **URL de réponse** dans le portail Azure.

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le portail Azure, accédez à la page d’intégration de l’application **SAP NetWeaver** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet pour **Configuration SAML de base** afin de modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode lancé par le **fournisseur d’identité** , effectuez l’étape suivante :

    a. Cliquez sur **Charger le fichier de métadonnées** pour charger le **fichier de métadonnées du fournisseur de services** obtenu précédemment.

    b. Cliquez sur le **logo du dossier** pour sélectionner le fichier de métadonnées, puis cliquez sur **Charger** .

    c. Une fois le fichier de métadonnées chargé, les valeurs **Identificateur** et **URL de réponse** sont automatiquement renseignées dans la zone de texte de la section **Configuration SAML de base** , comme indiqué ci-dessous :

    d. Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://<your company instance of SAP NetWeaver>`.

    > [!NOTE]
    > Quelques clients nous ont signalé une erreur de configuration de l’URL de réponse pour leur instance. Si vous recevez ce type d’erreur, vous pouvez utiliser le script PowerShell suivant en tant que solution de contournement afin de définir l’URL de réponse appropriée pour votre instance :
    > ```
    > Set-AzureADServicePrincipal -ObjectId $ServicePrincipalObjectId -ReplyUrls "<Your Correct Reply URL(s)>"
    > ``` 
    > Vous devez d’abord définir l’ID d’objet ServicePrincipal ou le passer également ici.

1. L’application SAP NetWeaver attend les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à votre configuration des attributs du jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut. Cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue Attributs d’utilisateur.

    ![modifier un attribut](common/edit-attribute.png)

1. Dans la section **Revendications des utilisateurs** de la boîte de dialogue **Attributs utilisateur** , configurez le jeton SAML comme sur l’image ci-dessus et procédez comme suit :

    a. Cliquez sur l’ **icône Modifier** pour ouvrir la boîte de dialogue **Gérer les revendications des utilisateurs** .

    ![icône modifier](./media/sapnetweaver-tutorial/nameidattribute.png)

    ![image](./media/sapnetweaver-tutorial/nameidattribute1.png)

    b. Dans la liste **Transformation** , sélectionnez **ExtractMailPrefix()** .

    c. Dans la liste **Paramètre 1** , sélectionnez **user.userprincipalname** .

    d. Cliquez sur **Enregistrer** .

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **XML de métadonnées de fédération** et sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

   ![Lien Téléchargement de certificat](common/metadataxml.png)

1. Dans la section **Configurer SAP NetWeaver** , copiez la ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à SAP NetWeaver.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **SAP NetWeaver** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .
1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .
1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-sap-netweaver-using-saml"></a>Configurer SAP NetWeaver avec SAML

1. Connectez-vous au système SAP et accédez au code de transaction SAML2. Une nouvelle fenêtre de navigateur s’ouvre avec l’écran de configuration SAML.

2. Pour configurer des points de terminaison pour le fournisseur d’identité approuvé (Azure AD), accédez à l’onglet **Trusted Providers** (Fournisseurs approuvés).

    ![Configurer l’authentification unique – Fournisseurs approuvés](./media/sapnetweaver-tutorial/tutorial_sapnetweaver_samlconfig.png)

3. Appuyez sur **Add** (Ajouter) et sélectionnez **Upload Metadata File** (Charger le fichier de métadonnées) dans le menu contextuel.

    ![Configurer l’authentification unique 2](./media/sapnetweaver-tutorial/tutorial_sapnetweaver_uploadmetadata.png)

4. Chargez le fichier de métadonnées que vous avez téléchargé à partir du portail Azure.

    ![Configurer l’authentification unique 3](./media/sapnetweaver-tutorial/tutorial_sapnetweaver_metadatafile.png)

5. Dans l’écran suivant, tapez le nom d’alias. Par exemple, tapez aadsts, puis appuyez sur **Next** (Suivant) pour continuer.

    ![Configurer l’authentification unique 4](./media/sapnetweaver-tutorial/tutorial_sapnetweaver_aliasname.png)

6. Vérifiez que votre **Digest Algorithm** (Algorithme Digest) est **SHA-256** et qu’aucun changement n’est requis, puis appuyez sur **Next** (Suivant).

    ![Configurer l’authentification unique 5](./media/sapnetweaver-tutorial/tutorial_sapnetweaver_identityprovider.png)

7. Dans **Single Sign-On Endpoints** (Points de terminaison d’authentification unique), utilisez **HTTP POST** et cliquez sur **Next** (Suivant) pour continuer.

    ![Configurer l’authentification unique 6](./media/sapnetweaver-tutorial/tutorial_sapnetweaver_httpredirect.png)

8. Dans **Single Logout Endpoints** (Points de terminaison Single Logout), utilisez **HTTPRedirect** et cliquez sur **Next** (Suivant) pour continuer.

    ![Configurer l’authentification unique 7](./media/sapnetweaver-tutorial/tutorial_sapnetweaver_httpredirect1.png)

9. Dans **Artifact Endpoints** (Points de terminaison d’artefact), appuyez sur **Next** (Suivant) pour continuer.

    ![Configurer l’authentification unique 8](./media/sapnetweaver-tutorial/tutorial_sapnetweaver_artifactendpoint.png)

10. Dans **Authentication Requirements** (Exigences d’authentification), cliquez sur **Finish** (Terminer).

    ![Configurer l’authentification unique 9](./media/sapnetweaver-tutorial/tutorial_sapnetweaver_authentication.png)

11. Accédez à l’onglet **Trusted Providers** (Fournisseurs approuvés) > **Identity Federation** (Fédération des identités), en bas de l’écran. Cliquez sur **Modifier** .

    ![Configurer l’authentification unique 10](./media/sapnetweaver-tutorial/tutorial_sapnetweaver_trustedprovider.png)

12. Cliquez sur **Add** (Ajouter) sous l’onglet **Identity Federation** (Fédération des identités), dans la fenêtre du bas.

    ![Configurer l’authentification unique 11](./media/sapnetweaver-tutorial/tutorial_sapnetweaver_addidentityprovider.png)

13. Dans la fenêtre contextuelle, sélectionnez **Unspecified** (Non spécifié) dans **Supported NameID formats** (Formats d’ID de nom pris en charge), puis cliquez sur OK.

    ![Configurer l’authentification unique 12](./media/sapnetweaver-tutorial/tutorial_sapnetweaver_nameid.png)

14. Notez que les valeurs **user ID Source** (source d’identifiant utilisateur) et **user ID mapping mode** (mode mappage d’identifiant utilisateur) déterminent le lien entre l’utilisateur SAP et la revendication Azure AD.  

    #### <a name="scenario-sap-user-to-azure-ad-user-mapping"></a>Scénario : Mappage d’un utilisateur SAP à un utilisateur Azure AD

    a. Capture d’écran des détails de NameID à partir de SAP.

    ![Configurer l’authentification unique 13](./media/sapnetweaver-tutorial/nameiddetails.png)

    b. Capture d’écran mentionnant les revendications obligatoires d’Azure AD.

    ![Configurer l’authentification unique 14](./media/sapnetweaver-tutorial/claimsaad1.png)

    #### <a name="scenario-select-sap-user-id-based-on-configured-email-address-in-su01-in-this-case-email-id-should-be-configured-in-su01-for-each-user-who-requires-sso"></a>Scénario : Sélectionner un identifiant utilisateur SAP en fonction de l’adresse e-mail configurée dans SU01. Dans ce cas, l’ID de l’adresse e-mail doit être configurée dans su01 pour chaque utilisateur nécessitant l’authentification unique.

    a.  Capture d’écran des détails de NameID à partir de SAP.

    ![Configurer l’authentification unique 15](./media/sapnetweaver-tutorial/tutorial_sapnetweaver_nameiddetails1.png)

    b. Capture d’écran mentionnant les revendications obligatoires d’Azure AD.

    ![Configurer l’authentification unique 16](./media/sapnetweaver-tutorial/claimsaad2.png)

15. Cliquez sur **Save** (Enregistrer), puis sur **Enable** (Activer) pour activer le fournisseur d’identité.

    ![Configurer l’authentification unique 17](./media/sapnetweaver-tutorial/configuration1.png)

16. Cliquez sur **OK** quand vous y êtes invité.

    ![Configurer l’authentification unique 18](./media/sapnetweaver-tutorial/configuration2.png)

    ### <a name="create-sap-netweaver-test-user"></a>Créer un utilisateur de test SAP NetWeaver

    Dans cette section, vous créez un utilisateur appelé B.Simon dans SAP NetWeaver. Collaborez avec votre équipe d’experts SAP en interne ou avec le partenaire SAP de votre organisation pour ajouter les utilisateurs dans la plateforme SAP NetWeaver.

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

1. Une fois le fournisseur d’identité Azure AD activé, essayez d’accéder à l’URL ci-dessous pour vérifier l’authentification unique (aucune invite n’est fournie pour entrer le nom d’utilisateur et le mot de passe)

    `https://<sapurl>/sap/bc/bsp/sap/it00/default.htm`

    (ou) utilisez l’URL ci-dessous.

    `https://<sapurl>/sap/bc/bsp/sap/it00/default.htm`

    > [!NOTE]
    > Remplacez sapurl par le nom d’hôte SAP réel.

2. L’URL ci-dessus doit vous diriger vers l’écran mentionné ci-dessous. Si vous pouvez accéder à la page ci-dessous, cela signifie que l’installation de l’authentification unique Azure AD s’est déroulée correctement.

    ![tester l’authentification unique](./media/sapnetweaver-tutorial/testingsso.png)

3. Si une invite demandant à entrer le nom d’utilisateur et le mot de passe apparaît, diagnostiquez le problème en activant la trace. Pour cela, utilisez l’URL ci-dessous.

    `https://<sapurl>/sap/bc/webdynpro/sap/sec_diag_tool?sap-client=122&sap-language=EN#`

## <a name="configure-sap-netweaver-for-oauth"></a>Configurer SAP NetWeaver pour OAuth

1. Le processus documenté de SAP est disponible à l’emplacement suivant : [NetWeaver Gateway Service Enabling and OAuth 2.0 Scope Creation](https://wiki.scn.sap.com/wiki/display/Security/NetWeaver+Gateway+Service+Enabling+and+OAuth+2.0+Scope+Creation)

2. Accédez à SPRO et recherchez **Activate and Maintain services** (Activer et maintenir les services).

    ![Activer et tenir à jour les services](./media/sapnetweaver-tutorial/oauth01.png)

3. Dans cet exemple, nous voulons connecter le service OData `DAAG_MNGGRP` avec OAuth à l’authentification unique Azure AD. Utilisez la recherche de nom du service technique pour le service `DAAG_MNGGRP` et activez-le s’il n’est pas encore actif (recherchez l’état `green` sous l’onglet des nœuds ICF). Vérifiez si l’alias système (le système back-end connecté, où le service est en cours d’exécution) est correct.

    ![Service OData](./media/sapnetweaver-tutorial/oauth02.png)

    * Cliquez ensuite sur le bouton **OAuth** dans la barre de boutons du haut et affectez `scope` (conservez le nom par défaut proposé).

4. Pour notre exemple, l’étendue est `DAAG_MNGGRP_001` ; elle est générée à partir du nom du service avec un ajout automatique d’un nombre. Vous pouvez utiliser le rapport `/IWFND/R_OAUTH_SCOPES` pour changer le nom de l’étendue ou la créer manuellement.

    ![Configurer OAuth](./media/sapnetweaver-tutorial/oauth03.png)

    > [!NOTE]
    > Message `soft state status is not supported` : vous pouvez l’ignorer sans problème. Pour plus d’informations, cliquez [ici](https://help.sap.com/doc/saphelp_nw74/7.4.16/1e/c60c33be784846aad62716b4a1df39/content.htm?no_cache=true).

### <a name="create-a-service-user-for-the-oauth-20-client"></a>Créer un utilisateur du service pour le client OAuth 2.0

1. OAuth2 utilise un `service ID` pour obtenir le jeton d’accès pour l’utilisateur final en son nom. Restriction importante découlant de la conception d’OAuth : le `OAuth 2.0 Client ID` doit être identique au `username` utilisé par le client OAuth 2.0 pour la connexion lors de la demande d’un jeton d’accès. Par conséquent, pour notre exemple, nous allons inscrire un client OAuth 2.0 avec le nom CLIENT1 et, à titre de prérequis, un utilisateur portant le même nom (CLIENT1) doit exister dans le système SAP : nous allons configurer cet utilisateur pour une utilisation par l’application référencée. 

2. Lors de l’inscription d’un client OAuth, nous utilisons le `SAML Bearer Grant type`.

    >[!NOTE]
    >Pour plus d’informations, consultez le document consacré à l’inscription du client OAuth 2.0 pour le type d’octroi du porteur SAML [ici](https://wiki.scn.sap.com/wiki/display/Security/OAuth+2.0+Client+Registration+for+the+SAML+Bearer+Grant+Type).

3. tcod: SU01 / Créez l’utilisateur CLIENT1 en tant que `System type` et affectez-lui un mot de passe, enregistrez-le en indiquant au programmeur de l’API qu’il faut fournir les informations d’identification et les associer au nom d’utilisateur dans le code appelant. Aucun profil ou rôle ne doit être affecté.

### <a name="register-the-new-oauth-20-client-id-with-the-creation-wizard"></a>Inscrire le nouvel ID de client OAuth 2.0 avec l’Assistant Création

1. Pour inscrire un nouveau **client OAuth 2.0** , démarrez la transaction **SOAUTH2** . La transaction affiche une vue d’ensemble des clients OAuth 2.0 qui ont déjà été inscrits. Choisissez **Create** (Créer) pour démarrer l’Assistant pour le nouveau client OAuth, nommé CLIENT1 dans cet exemple.

2. Accédez à T-Code : **SOAUTH2** et spécifiez la description, puis cliquez sur **Next** (Suivant).

    ![SOAUTH2](./media/sapnetweaver-tutorial/oauth04.png)

    ![ID Client OAuth 2.0](./media/sapnetweaver-tutorial/oauth05.png)

3. Sélectionnez le **fournisseur d’identité SAML2 - Azure AD** déjà ajouté dans la liste déroulante et enregistrez.

    ![Fournisseur d’identité SAML2 – Azure AD 1](./media/sapnetweaver-tutorial/oauth06.png)

    ![Fournisseur d’identité SAML2 – Azure AD 2](./media/sapnetweaver-tutorial/oauth07.png)

    ![Fournisseur d’identité SAML2 – Azure AD 3](./media/sapnetweaver-tutorial/oauth08.png)

4. Cliquez sur **Add** (Ajouter) sous affectation de l’étendue pour ajouter l’étendue créée précédemment: `DAAG_MNGGRP_001`

    ![Étendue](./media/sapnetweaver-tutorial/oauth09.png)

    ![affectation d’étendue](./media/sapnetweaver-tutorial/oauth10.png)

5. Cliquez sur **Finish** (Terminer).

## <a name="next-steps"></a>Étapes suivantes

Après avoir configuré Azure AD SAP NetWeaver, vous pouvez appliquer le contrôle de session, qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrir comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad)