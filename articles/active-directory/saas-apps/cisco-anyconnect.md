---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Cisco AnyConnect | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Cisco AnyConnect.
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
ms.openlocfilehash: a89ab7f2304fa51d3e8c7a968d445c9b40a457a3
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92456086"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-cisco-anyconnect"></a>Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory à Cisco AnyConnect

Dans ce tutoriel, vous allez apprendre à intégrer Cisco AnyConnect à Azure Active Directory (Azure AD). Quand vous intégrez Cisco AnyConnect à Azure AD, vous pouvez :

* Contrôler qui dans Azure AD a accès à Cisco AnyConnect.
* Permettre à vos utilisateurs de se connecter automatiquement à Cisco AnyConnect avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Cisco AnyConnect pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Cisco AnyConnect prend en charge l’authentification unique initiée par le **fournisseur d’identité**

## <a name="adding-cisco-anyconnect-from-the-gallery"></a>Ajout de Cisco AnyConnect à partir de la galerie

Pour configurer l’intégration de Cisco AnyConnect à Azure AD, vous devez ajouter Cisco AnyConnect à votre liste d’applications SaaS gérées à partir de la galerie.

1. Connectez-vous au portail Azure avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Cisco AnyConnect** dans la zone de recherche.
1. Sélectionnez **Cisco AnyConnect** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-sso-for-cisco-anyconnect"></a>Configurer et tester l’authentification unique Azure AD pour Cisco AnyConnect

Configurez et testez l’authentification unique Azure AD avec Cisco AnyConnect pour un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur associé dans Cisco AnyConnect.

Pour configurer et tester l’authentification unique Azure AD avec Cisco AnyConnect, effectuez les étapes suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Cisco AnyConnect](#configure-cisco-anyconnect-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test Cisco AnyConnect](#create-cisco-anyconnect-test-user)** pour avoir un équivalent de B.Simon dans Cisco AnyConnect lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le portail Azure, accédez à la page d’intégration de l’application **Cisco AnyConnect** , recherchez la section **Gérer** , puis sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la page **Configurer l’authentification unique avec SAML** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **Identificateur** , tapez une URL au format suivant : `< YOUR CISCO ANYCONNECT VPN VALUE >`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `< YOUR CISCO ANYCONNECT VPN VALUE >`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur et l’URL de réponse réels. Pour obtenir ces valeurs, contactez l’[équipe de support client Cisco AnyConnect](https://www.cisco.com/c/en/us/support/index.html). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (en base64)** et sélectionnez **Télécharger** pour télécharger le fichier de certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer Cisco AnyConnect** , copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

> [!NOTE]
> Si vous voulez intégrer plusieurs TGT du serveur, vous devez ajouter plusieurs instances de l’application Cisco AnyConnect à partir de la galerie. Vous pouvez aussi choisir de télécharger votre propre certificat dans Azure AD pour toutes ces instances d’application. De cette façon, vous pouvez avoir le même certificat pour les applications, mais vous pouvez configurer des identificateurs et des URL de réponse différents pour chaque application.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant un accès à Cisco AnyConnect.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Cisco AnyConnect** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .
1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .
1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez qu’un rôle soit attribué aux utilisateurs, vous pouvez le sélectionner dans la liste déroulante **Sélectionner un rôle** . Si aucun rôle n’a été configuré pour cette application, vous voyez le rôle « Accès par défaut » sélectionné.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-cisco-anyconnect-sso"></a>Configurer l’authentification unique pour Cisco AnyConnect

1. Vous allez faire cela d’abord sur l’interface CLI, mais vous pourrez revenir en arrière et suivre une procédure pas à pas dans ASDM à un autre moment.

1. Connectez-vous à votre appliance VPN ; vous allez utiliser une ASA exécutant le train de code 9.8, et vos clients VPN seront de version 4.6+.

1. Tout d’abord, vous allez créer un Trustpoint et importer notre certificat SAML.

   ```
    config t

    crypto ca trustpoint AzureAD-AC-SAML
      revocation-check none
      no id-usage
      enrollment terminal
      no ca-check
    crypto ca authenticate AzureAD-AC-SAML
    -----BEGIN CERTIFICATE-----
    …
    PEM Certificate Text from download goes here
    …
    -----END CERTIFICATE-----
    quit
   ```

1. Les commandes suivantes vont provisionner votre fournisseur d’identité SAML.

   ```
    webvpn
    saml idp https://sts.windows.net/xxxxxxxxxxxxx/ (This is your Azure AD Identifier from the Set up Cisco AnyConnect section in the Azure portal)
    url sign-in https://login.microsoftonline.com/xxxxxxxxxxxxxxxxxxxxxx/saml2 (This is your Login URL from the Set up Cisco AnyConnect section in the Azure portal)
    url sign-out https://login.microsoftonline.com/common/wsfederation?wa=wsignout1.0 (This is Logout URL from the Set up Cisco AnyConnect section in the Azure portal)
    trustpoint idp AzureAD-AC-SAML
    trustpoint sp (Trustpoint for SAML Requests - you can use your existing external cert here)
    no force re-authentication
    no signature
    base-url https://my.asa.com
    ```

1. Vous pouvez maintenant appliquer l’authentification SAML à une configuration de tunnel VPN.

    ```
    tunnel-group AC-SAML webvpn-attributes
      saml identity-provider https://sts.windows.net/xxxxxxxxxxxxx/
      authentication saml
    end

    write mem
    ```

    > [!NOTE]
    > La configuration du fournisseur d’identité SAML s’accompagne d’une fonctionnalité ; si vous apportez des modifications à la configuration du fournisseur d’identité, vous devez supprimer la configuration du fournisseur d’identité SAML de votre groupe de tunnels et la réappliquer pour que les modifications prennent effet.

### <a name="create-cisco-anyconnect-test-user"></a>Créer un utilisateur de test Cisco AnyConnect

Dans cette section, vous allez créer un utilisateur appelé Britta Simon dans Cisco AnyConnect. Collaborez avec l’[équipe du support technique de Cisco AnyConnect](https://www.cisco.com/c/en/us/support/index.html) pour ajouter les utilisateurs dans la plateforme Cisco AnyConnect. Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester votre configuration de l’authentification unique Azure AD avec les options suivantes.

* Cliquez sur Tester cette application dans le portail Azure : vous devez être connecté automatiquement à l’instance Cisco AnyConnect pour laquelle vous avez configuré l’authentification unique (SSO)
* Vous pouvez utiliser le volet d’accès Microsoft. Le fait de cliquer sur la vignette Cisco AnyConnect dans le volet d’accès doit vous connecter automatiquement à l’application Cisco AnyConnect pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Étapes suivantes

Après avoir configuré Cisco AnyConnect, vous pouvez appliquer le contrôle de session qui protège l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).