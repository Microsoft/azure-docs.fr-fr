---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à FortiGate SSL VPN | Microsoft Docs'
description: Découvrez les étapes à suivre pour intégrer FortiGate SSL VPN à Azure Active Directory (Azure AD).
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: 18a3d9d5-d81c-478c-be7e-ef38b574cb88
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.topic: tutorial
ms.date: 08/11/2020
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: 021550598452516d45ae67c1139c2f891629a875
ms.sourcegitcommit: 192f9233ba42e3cdda2794f4307e6620adba3ff2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96296571"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-fortigate-ssl-vpn"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à FortiGate SSL VPN

Dans ce tutoriel, vous allez apprendre à intégrer FortiGate SSL VPN à Azure Active Directory (Azure AD). Quand vous intégrez FortiGate SSL VPN à Azure AD, vous pouvez :

* Utiliser Azure AD pour contrôler qui peut accéder à FortiGate SSL VPN.
* Permettre à vos utilisateurs de se connecter automatiquement à FortiGate SSL VPN avec leur compte Azure AD.
* gérer vos comptes à un emplacement central : le portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement FortiGate SSL VPN pour lequel l’authentification unique est activée.

## <a name="tutorial-description"></a>Description du tutoriel

Dans ce tutoriel, vous allez configurer et tester l’authentification SSO Azure AD dans un environnement de test.

FortiGate SSL VPN prend en charge l’authentification unique lancée par le fournisseur de services.

Après avoir configuré FortiGate SSL VPN, vous pouvez appliquer le contrôle de session qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="add-fortigate-ssl-vpn-from-the-gallery"></a>Ajouter FortiGate SSL VPN à partir de la galerie

Pour configurer l’intégration de FortiGate SSL VPN à Azure AD, vous devez ajouter FortiGate SSL VPN, disponible dans la galerie, à votre liste d’applications SaaS gérées :

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire, ou avec un compte Microsoft personnel.
1. Sélectionnez **Azure Active Directory** dans le volet de gauche.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, entrez **FortiGate SSL VPN** dans la zone de recherche.
1. Sélectionnez **FortiGate SSL VPN** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-sso-for-fortigate-ssl-vpn"></a>Configurer et tester l’authentification unique (SSO) Azure AD pour FortiGate SSL VPN

Vous allez configurer et tester l’authentification unique Azure AD avec FortiGate SSL VPN à l’aide d’un utilisateur de test nommé B.Simon. Pour que l’authentification unique fonctionne, vous devez établir une liaison entre un utilisateur Azure AD et l’utilisateur correspondant dans FortiGate SSL VPN.

Pour configurer et tester l’authentification unique Azure AD avec FortiGate SSL VPN, vous allez effectuer les étapes principales suivantes :

1. **[Configurez l’authentification unique Azure AD](#configure-azure-ad-sso)** afin d’activer la fonctionnalité pour vos utilisateurs.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD.
    1. **[Octroyez l’accès à l’utilisateur de test](#grant-access-to-the-test-user)** afin d’activer l’authentification unique Azure AD pour cet utilisateur.
1. **[Configurez l’authentification unique FortiGate SSL VPN](#configure-fortigate-ssl-vpn-sso)** côté application.
    1. **Créez un utilisateur de test FortiGate SSL VPN** comme équivalent de la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification SSO](#test-single-sign-on)** , pour vérifier que la configuration fonctionne.

### <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le portail Azure :

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **FortiGate SSL VPN**, dans la section **Gérer**, sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, sélectionnez le bouton représentant un crayon et correspondant à **Configuration SAML de base** pour modifier les paramètres :

   ![Capture d’écran montrant le bouton avec l’icône de crayon pour la modification de la configuration SAML de base.](common/edit-urls.png)

1. Dans la page **Configurer l’authentification unique avec SAML**, entrez les valeurs suivantes :

    a. Dans la zone **URL de connexion**, entrez une URL au format `https://<FQDN>/remote/login`.

    b. Dans la zone **Identificateur**, entrez une URL au format `https://<FQDN>/remote/saml/metadata`.

    c. Dans la zone **URL de réponse**, entrez une URL au format `https://<FQDN>/remote/saml/login`.

    d. Dans la zone **URL de déconnexion**, entrez une URL au format `https://<FQDN>/remote/saml/logout`.

    > [!NOTE]
    > Ces valeurs sont juste des modèles. Vous devez utiliser l’**URL de connexion**, l’**identificateur**, l’**URL de réponse** et l’**URL de déconnexion** réels. Pour obtenir de l’aide, contactez le [support technique Fortinet](https://support.fortinet.com). Vous pouvez aussi consulter les exemples de modèles dans la documentation Fortinet et la section **Configuration SAML de base** sur le portail Azure.

1. L’application FortiGate SSL VPN s’attend à recevoir les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à la configuration. La capture d’écran suivante montre la liste des attributs par défaut.

    ![Capture d’écran montrant les attributs par défaut.](common/default-attributes.png)

1. Les deux revendications supplémentaires exigées par FortiGate SSL VPN sont indiquées dans le tableau suivant. Les noms de ces revendications doivent correspondre aux noms utilisés dans la section **Exécuter la configuration de ligne de commande FortiGate** de ce tutoriel. 

   | Nom |  Attribut source|
   | ------------ | --------- |
   | username | user.userprincipalname |
   | group | user.groups |
   
   Pour créer ces revendications supplémentaires :
   
   1. En regard de **Attributs utilisateur et revendications**, sélectionnez **Modifier**.
   1. Sélectionnez **Ajouter une nouvelle revendication**.
   1. Pour **Nom**, entrez **username**.
   1. Pour **Attribut source**, sélectionnez **user.userprincipalname**.
   1. Sélectionnez **Enregistrer**.
   1. Sélectionnez **Ajouter une revendication de groupe**.
   1. Sélectionnez **Tous les groupes**.
   1. Cochez la case **Personnaliser le nom de la revendication de groupe**.
   1. Pour **Nom**, entrez **group**.
   1. Sélectionnez **Enregistrer**.   

1. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, sélectionnez le lien **Télécharger** en regard de **Certificat (en base64)** pour télécharger le certificat et l’enregistrer sur votre ordinateur :

    ![Capture d’écran montrant le lien de téléchargement du certificat.](common/certificatebase64.png)

1. Dans la section **Configurer FortiGate SSL VPN**, copiez la ou les URL appropriées en fonction de vos besoins :

    ![Capture d’écran montrant les URL de configuration.](common/copy-configuration-urls.png)

#### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test nommé B.Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory**. Sélectionnez **Utilisateurs**, puis **Tous les utilisateurs**.
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur**, effectuez les étapes suivantes :
   1. Dans la zone **Nom**, entrez **B.Simon**.  
   1. Dans la zone **Nom d’utilisateur**, entrez \<username>@\<companydomain>.\<extension>. Par exemple : `B.Simon@contoso.com`.
   1. Sélectionnez **Afficher le mot de passe**, puis notez la valeur affichée dans la zone **Mot de passe**.
   1. Sélectionnez **Create** (Créer).

#### <a name="grant-access-to-the-test-user"></a>Octroyer l’accès à l’utilisateur de test

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à FortiGate SSL VPN.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **FortiGate SSL VPN**.
1. Dans la page de vue d’ensemble de l’application, dans la section **Gérer**, sélectionnez **Utilisateurs et groupes** :

   ![Capture d’écran affichant l’option Utilisateurs et groupes.](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** :

    ![Capture d’écran montrant le bouton Ajouter un utilisateur.](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B.Simon** dans la liste **Utilisateurs**, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste. Cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution**, sélectionnez **Affecter**.

#### <a name="create-a-security-group-for-the-test-user"></a>Créer un groupe de sécurité pour l’utilisateur de test

Dans cette section, vous allez créer un groupe de sécurité dans Azure Active Directory pour l’utilisateur de test. FortiGate utilisera ce groupe de sécurité pour accorder à l’utilisateur l’accès réseau via le VPN.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory**. Sélectionnez ensuite **Groupes**.
1. Sélectionnez **Nouveau groupe** dans la partie supérieure de l’écran.
1. Dans les propriétés du **Nouveau groupe**, effectuez les étapes suivantes :
   1. Dans la liste **Type de groupe**, sélectionnez **Sécurité**.
   1. Dans la zone **Nom du groupe**, entrez **FortiGateAccess**.
   1. Dans la zone **Description du groupe**, entrez **Group for granting FortiGate VPN access** (Groupe pour l’octroi de l’accès à FortiGate VPN).
   1. Pour **Des rôles Azure AD peuvent être attribués au groupe (préversion)** , sélectionnez **Non**.
   1. Dans la zone **Type d’appartenance**, sélectionnez **Affectée**.
   1. Sous **Membres**, sélectionnez **Aucun membre sélectionné**.
   1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B.Simon** dans la liste **Utilisateurs**, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
   1. Sélectionnez **Create** (Créer).
1. Une fois de retour dans la section **Groupes** dans Azure Active Directory, recherchez le groupe **Accès FortiGate** et notez l’**ID d’objet**. Vous en aurez besoin ultérieurement.

### <a name="configure-fortigate-ssl-vpn-sso"></a>Configurer l’authentification unique FortiGate SSL VPN

#### <a name="upload-the-base64-saml-certificate-to-the-fortigate-appliance"></a>Charger le certificat SAML Base64 sur l’appliance FortiGate

Après avoir terminé la configuration SAML de l’application FortiGate dans votre locataire, vous avez téléchargé le certificat SAML encodé en Base64. Vous devez charger ce certificat sur l’appliance FortiGate :

1. Connectez-vous au portail de gestion de votre appliance FortiGate.
1. Dans le volet gauche, sélectionnez **Système**.
1. Sous **Système**, sélectionnez **Certificats**.
1. Sélectionnez **Importer** > **Certificat distant**.
1. Accédez au certificat téléchargé à partir du déploiement de l’application FortiGate dans le locataire Azure, sélectionnez-le, puis sélectionnez **OK**.

Une fois le certificat chargé, notez son nom sous **Système** > **Certificats** > **Certificat distant**. Par défaut, il est nommé REMOTE_Cert_ *N*, où *N* est une valeur entière.

#### <a name="complete-fortigate-command-line-configuration"></a>Effectuer la configuration de ligne de commande de FortiGate

Les étapes suivantes nécessitent que vous configuriez l’URL de déconnexion Azure. Cette URL contient un point d’interrogation (?). Vous devez suivre des étapes spécifiques pour envoyer correctement ce caractère. Vous ne pouvez pas effectuer ces étapes à partir de la console CLI FortiGate. Au lieu de cela, établissez une session SSH pour l’appliance FortiGate à l’aide d’un outil comme PuTTy. Si votre appliance FortiGate est une machine virtuelle Azure, vous pouvez effectuer les étapes suivantes à partir de la console série des machines virtuelles Azure.

Pour effectuer ces étapes, vous avez besoin des valeurs que vous avez enregistrées précédemment :

- L’ID d’entité
- URL de réponse
- URL de déconnexion
- URL de connexion Azure
- Identificateur Azure AD
- URL de déconnexion Azure
- Nom du certificat SAML Base64 (REMOTE_Cert_ *N*)

1. Établissez une session SSH pour votre appliance FortiGate, puis connectez-vous avec un compte administrateur FortiGate.
1. Exécutez les commandes suivantes :

   ```console
    config user saml
    edit azure
    set entity-id <Entity ID>
    set single-sign-on-url <Reply URL>
    set single-logout-url <Logout URL>
    set idp-single-sign-on-url <Azure Login URL>
    set idp-entity-id <Azure AD Identifier>
    set idp-single-logout-url <Azure Logout URL>
    set idp-cert <Base64 SAML Certificate Name>
    set user-name username
    set group-name group
    end

   ```

   > [!NOTE]
   > L’URL de déconnexion Azure contient un caractère `?`. Vous devez entrer une séquence de touches spéciale pour fournir correctement cette URL à la console série FortiGate. L’URL est généralement `https://login.microsoftonline.com/common/wsfederation?wa=wsignout1.0`.
   >
   > Pour entrer l’URL de déconnexion Azure dans la console série, entrez `set idp-single-logout-url https://login.microsoftonline.com/common/wsfederation`.
   > 
   > Sélectionnez ensuite CTRL+V, puis collez le reste de l’URL pour terminer la ligne : `set idp-single-logout-url https://login.microsoftonline.com/common/wsfederation?wa=wsignout1.0`.

#### <a name="configure-fortigate-for-group-matching"></a>Configurer FortiGate pour la correspondance de groupe

Dans cette section, vous allez configurer FortiGate pour reconnaître l’ID d’objet du groupe de sécurité qui inclut dans lequel l’utilisateur de test. Cette configuration permettra à FortiGate de prendre des décisions d’accès en fonction de l’appartenance au groupe.

Pour effectuer ces étapes, vous avez besoin de l’ID d’objet du groupe de sécurité FortiGateAccess que vous avez créé précédemment dans ce tutoriel.

1. Établissez une session SSH pour votre appliance FortiGate, puis connectez-vous avec un compte administrateur FortiGate.
1. Exécutez les commandes suivantes :

   ```
    config user group
    edit FortiGateAccess
    set member azure
    config match
    edit 1
    set server-name azure
    set group-name <Object Id>
    next
    end
    next
    end
   ```

#### <a name="create-a-fortigate-vpn-portals-and-firewall-policy"></a>Créer des portails FortiGate VPN et une stratégie de pare-feu

Dans cette section, vous allez configurer un portail FortiGate VPN et une stratégie de pare-feu qui accorde l’accès au groupe de sécurité FortiGateAccess que vous avez créé précédemment dans ce tutoriel.

Collaborez avec l’[équipe de support FortiGate](mailto:tac_amer@fortinet.com) pour ajouter les portails VPN et la stratégie de pare-feu à la plateforme VPN FortiGate. Vous devez effectuer cette étape avant d’utiliser l’authentification unique.

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester votre configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de sélectionner la vignette FortiGate SSL VPN dans le volet d’accès doit vous connecter automatiquement à l’application FortiGate SSL VPN pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le volet d’accès, consultez [Présentation du volet d’accès](../user-help/my-apps-portal-end-user-access.md).

Pour une expérience utilisateur optimale, Microsoft et FortiGate vous conseillent d’utiliser le client Fortinet VPN, FortiClient.

## <a name="additional-resources"></a>Ressources supplémentaires

- [Tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer FortiGate SSL VPN avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)
