---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à F5 | Microsoft Docs'
description: Dans cet article, découvrez les étapes que vous devez effectuer pour intégrer la touche F5 à Azure Active Directory (Azure AD).
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 11/11/2019
ms.author: jeedes
ms.openlocfilehash: 8d24ed014dd66235383b58cbcb7404aaf00f863e
ms.sourcegitcommit: ce8eecb3e966c08ae368fafb69eaeb00e76da57e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92308957"
---
# <a name="tutorial-azure-active-directory-ad-single-sign-on-sso-integration-with-f5"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory (AD) à F5

Dans ce tutoriel, vous allez découvrir comment intégrer F5 à Azure Active Directory (Azure AD). Quand vous intégrez F5 à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à F5.
* Permettre à vos utilisateurs de se connecter automatiquement à F5 avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement F5 pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

F5 prend en charge l’authentification unique lancée par le **fournisseur de services et le fournisseur d’identité**.

La fonctionnalité d’authentification unique F5 peut être configurée de trois façons différentes :

- [Configurer l’authentification unique F5 pour une application Kerberos avancée](#configure-f5-single-sign-on-for-advanced-kerberos-application)

- [Configurer l’authentification unique F5 pour une application basée sur l’en-tête](headerf5-tutorial.md)

- [Configurer l’authentification unique F5 pour une application Kerberos](kerbf5-tutorial.md)

## <a name="adding-f5-from-the-gallery"></a>Ajout de F5 à partir de la galerie

Pour configurer l’intégration de F5 avec Azure AD, vous devez ajouter F5, disponible dans la galerie, à votre liste d’applications SaaS managées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **F5** dans la zone de recherche.
1. Sélectionnez **F5** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-f5"></a>Configurer et tester l’authentification unique Azure AD pour F5

Configurez et testez l’authentification unique Azure AD avec F5 à l’aide d’un utilisateur de test appelé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir une relation entre un utilisateur Azure AD et l’utilisateur F5 associé.

Pour configurer et tester l’authentification unique Azure AD avec F5, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique F5](#configure-f5-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test F5](#create-f5-test-user)** pour disposer, dans F5, d’un équivalent de B.Simon lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans la page d’intégration de l’application **F5** du [portail Azure](https://portal.azure.com/), recherchez la section **Gérer**, puis sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base**, si vous souhaitez configurer l’application en mode Initié par le **fournisseur d’identité**, entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **Identificateur**, tapez une URL au format suivant : `https://<YourCustomFQDN>.f5.com/`

    b. Dans la zone de texte **URL de réponse**, tapez une URL au format suivant : `https://<YourCustomFQDN>.f5.com/`

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services**, cliquez sur **Définir des URL supplémentaires**, puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion**, tapez une URL au format suivant : `https://<YourCustomFQDN>.f5.com/`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur, l’URL de réponse et l’URL de connexion réels. Pour obtenir ces valeurs, contactez l’[équipe de support technique de F5](https://support.f5.com/csp/knowledge-center/software/BIG-IP?module=BIG-IP%20APM45). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, recherchez **XML de métadonnées de fédération** et sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

1. Dans la section **Configurer F5**, copiez la ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à F5.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **F5**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

## <a name="configure-f5-sso"></a>Configurer l’authentification unique F5

- [Configurer l’authentification unique F5 pour une application basée sur l’en-tête](headerf5-tutorial.md)

- [Configurer l’authentification unique F5 pour une application Kerberos](kerbf5-tutorial.md)

### <a name="configure-f5-single-sign-on-for-advanced-kerberos-application"></a>Configurer l’authentification unique F5 pour une application Kerberos avancée

1. Ouvrez une nouvelle fenêtre de navigateur web et connectez-vous à votre site d’entreprise F5 (Kerberos avancé) en tant qu’administrateur, puis effectuez les étapes suivantes :

1. Vous devez importer le certificat de métadonnées dans F5 (Kerberos avancé) qui sera utilisé plus tard lors du processus d’installation. Accédez à **System > Certificate Management > Traffic Certificate Management >> SSL Certificate Lists** (Système > Gestion des certificats > Gestion des certificats de trafic > Liste des certificats SSL). Cliquez sur **Import** en haut à droite.

    ![Capture d’écran mettant en évidence le bouton Import pour importer le certificat de métadonnées.](./media/advance-kerbf5-tutorial/configure01.png)
 
1. Pour configurer le fournisseur d’identité SAML, accédez à **Access > Federation > SAML Service Provider > Create > From Metadata** (Accès > Fédération > Fournisseur de services SAML > Créer > À partir de métadonnées).

    ![Capture d’écran qui souligne comment créer le fournisseur d’identité SAML à partir de métadonnées.](./media/advance-kerbf5-tutorial/configure02.png)

    ![Capture d’écran montrant l’écran Create New SAML IdP Connector.](./media/advance-kerbf5-tutorial/configure03.png)
 
    ![Configuration de F5 (Kerberos avancé)](./media/advance-kerbf5-tutorial/configure04.png)

    ![Capture d’écran montrant l’écran Single Sign On Service Settings. ](./media/advance-kerbf5-tutorial/configure05.png)
 
1. Spécifiez le certificat chargé à la tâche 3.

    ![Capture d’écran montrant l’écran Edit SAML IdP Connector.](./media/advance-kerbf5-tutorial/configure06.png)

    ![Capture d’écran montrant l’écran Single Logout Service Settings.](./media/advance-kerbf5-tutorial/configure07.png)

 1. Pour configurer le fournisseur de services SAML, accédez à **Access > Federation > SAML Service Federation > Local SP Services > Create** (Accès > Fédération > Fédération de service SAML > Services de fournisseur de services locaux > Créer).

    ![Capture d’écran montrant la page où vous créez un service de fournisseur de services local.](./media/advance-kerbf5-tutorial/configure08.png)
 
1. Cliquez sur **OK**.

1. Sélectionnez la configuration de fournisseur de services, puis cliquez sur **Bind/UnBind IdP Connectors** (Lier/Dissocier les connecteurs IdP).

     ![Capture d’écran montrant le fournisseur de services SAML.](./media/advance-kerbf5-tutorial/configure09.png)
 
 
1. Cliquez sur **Add New Row** (Ajouter une nouvelle ligne) et sélectionnez le connecteur IdP externe (**External IdP connector**) créé à l’étape précédente.

    ![Capture d’écran mettant en évidence le bouton Add New Row.](./media/advance-kerbf5-tutorial/configure10.png)
 
1. Pour configurer l’authentification unique Kerberos, accédez à **Access > Single Sign-on > Kerberos** (Accès > Authentification unique > Kerberos).

    >[!Note]
    >Vous devrez créer et spécifier le compte de délégation Kerberos. Consultez la section KCD (voir l’annexe pour connaître les références de variables).

    • Source de nom d’utilisateur  `session.saml.last.attr.name.http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname`

    • Source de domaine de l’utilisateur  `session.logon.last.domain`

    ![Capture d’écran mettant en évidence Access > Single Sign On.](./media/advance-kerbf5-tutorial/configure11.png)

1. Pour configurer un profil d’accès, accédez à **Access > Profile/Policies > Access Profile (per session policies)** (Accès > Profil/stratégies > Profil d’accès (stratégies par session)).

    ![Capture d’écran mettant en évidence l’onglet Properties sous l’option de menu Profiles/Policies.](./media/advance-kerbf5-tutorial/configure12.png)

    ![Capture d’écran montrant l’onglet SSO/Auth Domains.](./media/advance-kerbf5-tutorial/configure13.png)

    ![Capture d’écran montrant l’onglet Access Policy.](./media/advance-kerbf5-tutorial/configure14.png)

    ![Capture d’écran montrant l’onglet Properties de la stratégie d’accès.](./media/advance-kerbf5-tutorial/configure15.png)

    ![Capture d’écran montrant les propriétés de l’attribution de variable.](./media/advance-kerbf5-tutorial/configure16.png)
 
    * session.logon.last.usernameUPN   expr {[mcget {session.saml.last.identity}]}

    * session.ad.lastactualdomain  TEXT superdemo.live

    ![Capture d’écran montrant les propriétés de requête Active Directory.](./media/advance-kerbf5-tutorial/configure17.png)

    * (userPrincipalName=%{session.logon.last.usernameUPN})

    ![Capture d’écran montrant l’onglet Branch Rules et la règle Check Account.](./media/advance-kerbf5-tutorial/configure18.png)

    ![Capture d’écran montrant les zones de texte de variable personnalisée et d’expression personnalisée.](./media/advance-kerbf5-tutorial/configure19.png)

    * session.logon.last.username  expr { "[mcget {session.ad.last.attr.sAMAccountName}]" }

    ![Capture d’écran montrant les valeurs dans les champs SSO Token Name et SSO Token Password.](./media/advance-kerbf5-tutorial/configure20.png)

    * mcget {session.logon.last.username}
    * mcget {session.logon.last.password}

1. Pour ajouter un nouveau nœud, accédez à **Local Traffic > Nodes > Node List > +** (Trafic local > Nœuds > Liste de nœuds).

    ![Capture d’écran mettant en évidence Local Traffic > Nodes.](./media/advance-kerbf5-tutorial/configure21.png)
 
1. Pour créer un pool, accédez à **Local Traffic > Pools > Pool List > Create** (Trafic local > Pools > Liste de pools > Créer).

     ![Capture d’écran mettant en évidence Local Traffic > Pools.](./media/advance-kerbf5-tutorial/configure22.png)

 1. Pour créer un serveur virtuel, accédez à **Local Traffic > Virtual Servers > Virtual Server List > +** (Trafic local > Serveurs virtuels > Liste des serveurs virtuels).

    ![Capture d’écran mettant en évidence Local Traffic > Virtual Servers.](./media/advance-kerbf5-tutorial/configure23.png)

1. Spécifiez le profil d’accès créé à l’étape précédente.

    ![Capture d’écran montrant où vous spécifiez le profil d’accès que vous avez créé.](./media/advance-kerbf5-tutorial/configure24.png) 

### <a name="setting-up-kerberos-delegation"></a>Configuration de la délégation Kerberos 

>[!Note]
>Pour plus d’informations, cliquez [ici](https://www.f5.com/pdf/deployment-guides/kerberos-constrained-delegation-dg.pdf).

* **Étape 1 : Créer un compte de délégation**

    * Exemple
    ```
    Domain Name : superdemo.live
    Sam Account Name : big-ipuser

    New-ADUser -Name "APM Delegation Account" -UserPrincipalName host/big-ipuser.superdemo.live@superdemo.live -SamAccountName "big-ipuser" -PasswordNeverExpires $true -Enabled $true -AccountPassword (Read-Host -AsSecureString "Password!1234")
    ```

* **Étape 2 : Définir le SPN (sur le compte de délégation APM)**

    *  Exemple
    ```
    setspn –A host/big-ipuser.superdemo.live big-ipuser
    ```

* **Étape 3 : Délégation SPN (pour le compte de service d’application)**

    * Configurez la délégation appropriée pour le compte de délégation F5.
    * Dans l’exemple ci-dessous, le compte de délégation APM est configuré pour KCD pour l’application FRP-App1.superdemo.live.

        ![Capture d’écran montrant les propriétés du compte de délégation APM > onglet Delegation.](./media/advance-kerbf5-tutorial/configure25.png)

1. Fournissez les informations mentionnées dans [ce document de référence](https://techdocs.f5.com/kb/en-us/products/big-ip_apm/manuals/product/apm-authentication-single-sign-on-11-5-0/2.html).

1. Annexe : Mappages de variables BIG-IP SAML – F5 illustrés ci-dessous :

    ![Capture d’écran montrant l’onglet Overview > Active Sessions.](./media/advance-kerbf5-tutorial/configure26.png)

    ![Capture d’écran montrant les variables et les clés de session.](./media/advance-kerbf5-tutorial/configure27.png) 

1. Voici la liste complète des attributs SAML par défaut. GivenName est représenté à l’aide de la chaîne suivante.
`session.saml.last.attr.name.http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname`

| session | Attribut |
| -- | -- |
| eb46b6b6.session.saml.last.assertionID | `<TENANT ID>` |
| eb46b6b6.session.saml.last.assertionIssueInstant  | `<ID>` |
| eb46b6b6.session.saml.last.assertionIssuer | `https://sts.windows.net/<TENANT ID>`/ |
| eb46b6b6.session.saml.last.attr.name.http:\//schemas.microsoft.com/claims/authnmethodsreferences | `http://schemas.microsoft.com/ws/2008/06/identity/authenticationmethod/password` |
| eb46b6b6.session.saml.last.attr.name.http:\//schemas.microsoft.com/identity/claims/displayname | user0 |
| eb46b6b6.session.saml.last.attr.name.http:\//schemas.microsoft.com/identity/claims/identityprovider | `https://sts.windows.net/<TENANT ID>/` |
| eb46b6b6.session.saml.last.attr.name.http:\//schemas.microsoft.com/identity/claims/objectidentifier | `<TENANT ID>` |
| eb46b6b6.session.saml.last.attr.name.http:\//schemas.microsoft.com/identity/claims/tenantid | `<TENANT ID>` |
| eb46b6b6.session.saml.last.attr.name.http:\//schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress | `user0@superdemo.live` |
| eb46b6b6.session.saml.last.attr.name.http:\//schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname | user0 |
| eb46b6b6.session.saml.last.attr.name.http:\//schemas.xmlsoap.org/ws/2005/05/identity/claims/name | `user0@superdemo.live` |
| eb46b6b6.session.saml.last.attr.name.http:\//schemas.xmlsoap.org/ws/2005/05/identity/claims/surname | 0 |
| eb46b6b6.session.saml.last.audience | `https://kerbapp.superdemo.live` |
| eb46b6b6.session.saml.last.authNContextClassRef | urn:oasis:names:tc:SAML:2.0:ac:classes:Password |
| eb46b6b6.session.saml.last.authNInstant | `<ID>` |
| eb46b6b6.session.saml.last.identity | `user0@superdemo.live` |
| eb46b6b6.session.saml.last.inResponseTo | `<TENANT ID>` |
| eb46b6b6.session.saml.last.nameIDValue | `user0@superdemo.live` |
| eb46b6b6.session.saml.last.nameIdFormat | urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress |
| eb46b6b6.session.saml.last.responseDestination | `https://kerbapp.superdemo.live/saml/sp/profile/post/acs` |
| eb46b6b6.session.saml.last.responseId | `<TENANT ID>` |
| eb46b6b6.session.saml.last.responseIssueInstant | `<ID>` |
| eb46b6b6.session.saml.last.responseIssuer | `https://sts.windows.net/<TENANT ID>/` |
| eb46b6b6.session.saml.last.result | 1 |
| eb46b6b6.session.saml.last.samlVersion | 2 |
| eb46b6b6.session.saml.last.sessionIndex | `<TENANT ID>` |
| eb46b6b6.session.saml.last.statusValue | urn:oasis:names:tc:SAML:2.0:status:Success |
| eb46b6b6.session.saml.last.subjectConfirmDataNotOnOrAfter | `<ID>` |
| eb46b6b6.session.saml.last.subjectConfirmDataRecipient | `https://kerbapp.superdemo.live/saml/sp/profile/post/acs` |
| eb46b6b6.session.saml.last.subjectConfirmMethod | urn:oasis:names:tc:SAML:2.0:cm:bearer |
| eb46b6b6.session.saml.last.validityNotBefore | `<ID>` |
| eb46b6b6.session.saml.last.validityNotOnOrAfter | `<ID>` |

### <a name="create-f5-test-user"></a>Créer un utilisateur de test F5

Dans cette section, vous allez créer un utilisateur appelé B.Simon dans F5. Collaborez avec l’ [équipe du support client de F5](https://support.f5.com/csp/knowledge-center/software/BIG-IP?module=BIG-IP%20APM45) pour ajouter des utilisateurs à la plateforme F5. Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique. 

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette F5 dans le volet d’accès, vous devez être connecté automatiquement à l’application F5 pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer F5 avec Azure AD](https://aad.portal.azure.com/)

- [Configurer l’authentification unique F5 pour une application basée sur l’en-tête](headerf5-tutorial.md)

- [Configurer l’authentification unique F5 pour une application Kerberos](kerbf5-tutorial.md)