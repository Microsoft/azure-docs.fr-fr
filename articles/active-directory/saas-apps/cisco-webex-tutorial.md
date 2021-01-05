---
title: 'Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory à Cisco Webex Meetings | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Cisco Webex Meetings.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 08/21/2019
ms.author: jeedes
ms.openlocfilehash: 7a4e3a4a6049b48fa70e5352680bdc7342676c0b
ms.sourcegitcommit: d79513b2589a62c52bddd9c7bd0b4d6498805dbe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/18/2020
ms.locfileid: "97670279"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-cisco-webex-meetings"></a>Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory à Cisco Webex Meetings

Ce tutoriel explique comment intégrer Cisco Webex Meetings à Azure Active Directory (Azure AD). Quand vous intégrez Cisco Webex Meetings à Azure AD, vous pouvez :

* Contrôler qui a accès à Cisco Webex Meetings.
* Permettre à vos utilisateurs de se connecter automatiquement à Cisco Webex Meetings avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Abonnement Cisco Webex Meetings pour lequel l’authentification unique (SSO) est activée

> [!NOTE]
> Cette intégration peut également être utilisée à partir de l’environnement cloud US Government Azure AD. Cette application est disponible dans la Galerie d’applications cloud US Government Azure AD et peut être configurée de la même façon que dans le cloud public.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Cisco Webex Meetings prend en charge l’authentification unique lancée par **le fournisseur de services et le fournisseur d’identité**

* Cisco Webex Meetings prend en charge l’attribution d’utilisateurs **juste-à-temps**.

## <a name="adding-cisco-webex-meetings-from-the-gallery"></a>Ajout de Cisco Webex Meetings à partir de la galerie

Pour configurer l’intégration de Cisco Webex Meetings à Azure AD, vous devez ajouter Cisco Webex Meetings, disponible dans la galerie, à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **Cisco Webex Meetings** dans la zone de recherche.
1. Sélectionnez **Cisco Webex Meetings** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-cisco-webex-meetings"></a>Configurer et tester l’authentification unique Azure AD pour Cisco Webex Meetings

Configurez et testez l’authentification unique Azure AD avec Cisco Webex Meetings pour un utilisateur de test nommé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Cisco Webex Meetings associé.

Pour configurer et tester l’authentification unique (SSO) Azure AD avec Cisco Webex Meetings, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
2. **[Configurer l’authentification unique Cisco Webex Meetings](#configure-cisco-webex-meetings-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test Cisco Webex Meetings](#create-cisco-webex-meetings-test-user)** pour avoir un équivalent de B.Simon dans Cisco Webex Meetings, lié à la représentation Azure AD associée.
3. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Cisco Webex Meetings**, recherchez la section **Gérer** et sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, vous pouvez configurer l’application en mode lancé par le **fournisseur d’identité** en chargeant le fichier de **métadonnées du fournisseur de services** comme suit :

    a. Cliquez sur **Charger un fichier de métadonnées**.

    b. Cliquez sur le **logo du dossier** pour sélectionner le fichier de métadonnées, puis cliquez sur **Charger**.

    c. Une fois le chargement du fichier de métadonnées du fournisseur de services terminé, les valeurs **Identificateur** et **URL de réponse** sont automatiquement renseignées dans la section **Configuration SAML de base**.

    >[!Note]
    >Vous recevrez le fichier de métadonnées de fournisseur de services à partir de la section **Configurer l’authentification unique Cisco Webex Meetings**, comme décrit plus loin dans le tutoriel. 

1. Si vous souhaitez configurer l’application en mode initié par le **fournisseur de service**, procédez comme suit :  

    a. Dans la section **Configuration SAML de base**, cliquez sur l’icône de crayon/modification.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

    b. Dans la zone de texte **URL de connexion**, tapez l’URL au format suivant : `https://<customername>.my.webex.com`

5. L’application Cisco Webex Meetings s’attend à recevoir les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à la configuration de vos attributs de jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut. Cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue Attributs d’utilisateur.

    ![image](common/edit-attribute.png)

6. En plus de ce qui précède, l’application Cisco Webex Meetings s’attend à ce que quelques attributs supplémentaires soient passés dans la réponse SAML. Dans la section Revendications des utilisateurs de la boîte de dialogue Attributs utilisateur, effectuez les étapes suivantes pour ajouter le jeton SAML, comme indiqué dans le tableau ci-dessous : 

    | Nom | Attribut source|
    | ---------------|  --------- |
    |   firstname    | user.givenname |
    |   lastname    | user.surname |
    |   email       | user.mail |
    |   uid    | user.mail |

    a. Cliquez sur le bouton **Ajouter une nouvelle revendication** pour ouvrir la boîte de dialogue **Gérer les revendications des utilisateurs**.

    b. Dans la zone de texte **Attribut**, indiquez le nom d’attribut pour cette ligne.

    c. Laissez le champ **Espace de noms** vide.

    d. Sélectionnez Source comme **Attribut**.

    e. Dans la liste déroulante **Attribut de la source**, sélectionnez la valeur d’attribut indiquée pour cette ligne.

    f. Cliquez sur **Enregistrer**.

4. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, recherchez **XML de métadonnées de fédération** et sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

6. Dans la section **Configurer Cisco Webex Meetings**, copiez la ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Cisco Webex Meetings.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **Cisco Webex Meetings**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

## <a name="configure-cisco-webex-meetings-sso"></a>Configurer l’authentification unique Cisco Webex Meetings

1. Accédez à l’URL `https://<customername>.webex.com/admin` avec vos informations d’identification d’administration.

2. Accédez à **Common Site Settings** (Paramètres de site courants), puis à **SSO Configuration** (Configuration de l’authentification unique).

    ![Capture d’écran montrant l’administration de Cisco Webex avec Common Site Settings et SSO Configuration sélectionnés.](./media/cisco-webex-tutorial/tutorial-cisco-webex-11.png)

3. Dans la page **Webex Administration** (Administration Webex), effectuez les étapes suivantes :

    ![Capture d’écran montrant la page Webex Administration avec les informations décrites à cette étape.](./media/cisco-webex-tutorial/tutorial-cisco-webex-10.png)

    a. Sélectionnez **SAML 2.0** comme **protocole de fédération**.

    b. Cliquez sur le lien **Import SAML Metadata** (Importer les métadonnées SAML) pour charger le fichier de métadonnées que vous avez téléchargé à partir du portail Azure.

    c. Cliquez sur le bouton **Exporter** pour télécharger le fichier de métadonnées du fournisseur de services, puis chargez-le dans la section **Configuration SAML de base** sur le portail Azure.

    d. Dans la zone de texte **AuthContextClassRef**, tapez `urn:oasis:names:tc:SAML:2.0:ac:classes:unspecified` et, si vous souhaitez activer l’authentification multifacteur à l’aide d’Azure AD, tapez les deux valeurs telles que `urn:oasis:names:tc:SAML:2.0:ac:classes:PasswordProtectedTransport;urn:oasis:names:tc:SAML:2.0:ac:classes:X509`

    e. Sélectionnez **Auto Account Creation** (Création de compte automatique).

    >[!NOTE]
    >Pour activer l’approvisionnement d’utilisateurs **juste-à-temps**, vous devez activer l’option **Auto Account Creation** (Création du compte automatique). En plus de cela, les attributs du jeton SAML doivent être transmis dans la réponse SAML.

    f. Cliquez sur **Enregistrer**.

    >[!NOTE]
    >Cette configuration concerne uniquement les clients qui utilisent l’ID utilisateur Webex dans le format d’e-mail.

### <a name="create-cisco-webex-meetings-test-user"></a>Créer un utilisateur de test Cisco Webex Meetings

L’objectif de cette section est de créer un utilisateur appelé B.Simon dans Cisco Webex Meetings. Cisco Webex Meetings prend en charge l’approvisionnement **juste-à-temps**, option activée par défaut. Vous n’avez aucune opération à effectuer dans cette section. S’il n’existe pas encore d’utilisateur dans Cisco Webex Meetings, il en est créé un quand vous tentez d’y accéder.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette Cisco Webex Meetings dans le panneau d’accès doit vous connecter automatiquement à l’application Cisco Webex Meetings pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer ServiceNow avec Azure AD](https://aad.portal.azure.com)