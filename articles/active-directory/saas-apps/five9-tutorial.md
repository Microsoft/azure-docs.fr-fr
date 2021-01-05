---
title: 'Didacticiel : Intégration d’Azure Active Directory avec Five9 Plus Adapter(CTI, agents de centre de contacts) | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Five9 Plus Adapter (CTI, agents de centre de contacts).
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/04/2019
ms.author: jeedes
ms.openlocfilehash: 68de5b11c131fe33252178ebecdeb9c3855fe239
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92453431"
---
# <a name="tutorial-azure-active-directory-integration-with-five9-plus-adapter-cti-contact-center-agents"></a>Didacticiel : Intégration d’Azure Active Directory avec Five9 Plus Adapter(CTI, agents de centre de contacts)

Dans ce didacticiel, vous allez apprendre à intégrer Five9 Plus Adapter (CTI, agents de centre de contacts) à Azure Active Directory (Azure AD).
L’intégration de Five9 Plus Adapter (CTI, agents de centre de contacts) à Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Five9 Plus Adapter (CTI, agents de centre de contacts).
* Vous pouvez autoriser les utilisateurs à se connecter automatiquement à Five9 Plus Adapter (CTI, agents de centre de contacts), par le biais de l’authentification unique, avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD avec Five9 Plus Adapter (CTI, agents de centre de contacts), vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/).
* Abonnement Five9 Plus Adapter (CTI, agents de centre de contacts) pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Five9 Plus Adapter (CTI, agents de centre de contacts) prend en charge l’authentification unique lancée par le **fournisseur d’identité**

## <a name="adding-five9-plus-adapter-cti-contact-center-agents-from-the-gallery"></a>Ajout de Five9 Plus Adapter (CTI, agents de centre de contacts) à partir de la galerie

Pour configurer l’intégration de Five9 Plus Adapter (CTI, agents de centre de contacts) dans Azure AD, vous devez ajouter Five9 Plus Adapter (CTI, agents de centre de contacts) à partir de la galerie à votre liste d’applications SaaS gérées.

**Pour ajouter Five9 Plus Adapter (CTI, agents de centre de contacts) à partir de la galerie, effectuez les étapes suivantes :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Five9 Plus Adapter (CTI, agents de centre de contacts)** , sélectionnez **Five9 Plus Adapter (CTI, agents de centre de contacts)** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![Five9 Plus Adapter (CTI, agents de centre de contacts) dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Five9 Plus Adapter (CTI, agents de centre de contacts) à l’aide d’un utilisateur de test appelé **Britta Simon** .
Pour que l’authentification unique fonctionne, une relation entre un utilisateur Azure AD et l’utilisateur Five9 Plus Adapter (CTI, agents de centre de contacts) associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec Five9 Plus Adapter (CTI, agents de centre de contacts), vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Five9 Plus Adapter (CTI, agents de centre de contacts)](#configure-five9-plus-adapter-cti-contact-center-agents-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Five9 Plus Adapter (CTI, agents de centre de contacts)](#create-five9-plus-adapter-cti-contact-center-agents-test-user)** pour avoir un équivalent de Britta Simon dans Five9 Plus Adapter (CTI, agents de centre de contacts) lié à la représentation Azure AD de l’utilisateur.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Five9 Plus Adapter (CTI, agents de centre de contacts), effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Five9 Plus Adapter (CTI, agents de centre de contacts)** , sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Sur la page **Configurer l’authentification unique avec SAML** , effectuez les étapes suivantes :

    ![Informations d’authentification unique dans Domaine et URL Five9 Plus Adapter (CTI, agents de centre de contacts)](common/idp-intiated.png)

    a. Dans la zone de texte **Identificateur** , tapez une URL au format suivant :
    
    |    Environnement      |       URL      |
    | :-- | :-- |
    | Pour « Five9 Plus Adapter for Microsoft Dynamics CRM » | `https://app.five9.com/appsvcs/saml/metadata/alias/msdc` |
    | Pour « Five9 Plus Adapter for Zendesk » | `https://app.five9.com/appsvcs/saml/metadata/alias/zd` |
    | Pour « Five9 Plus Adapter for Agent Desktop Toolkit » | `https://app.five9.com/appsvcs/saml/metadata/alias/adt` |

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant :

    |      Environnement     |      URL      |
    | :--                  | :--           |
    | Pour « Five9 Plus Adapter for Microsoft Dynamics CRM » | `https://app.five9.com/appsvcs/saml/SSO/alias/msdc` |
    | Pour « Five9 Plus Adapter for Zendesk » | `https://app.five9.com/appsvcs/saml/SSO/alias/zd` |
    | Pour « Five9 Plus Adapter for Agent Desktop Toolkit » | `https://app.five9.com/appsvcs/saml/SSO/alias/adt` |

6. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** en fonction des options définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

7. Dans la section **Configurer Five9 Plus Adapter (CTI, agents de centre de contacts)** , copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-five9-plus-adapter-cti-contact-center-agents-single-sign-on"></a>Configurer l’authentification unique Five9 Plus Adapter (CTI, agents de centre de contacts)

1. Pour configurer l’authentification unique côté **Five9 Plus Adapter (CTI, agents de centre de contacts)** , vous devez envoyer le **Certificat (en base64)** téléchargé et la ou les URL copiées correspondantes à l’ [équipe du support Five9 Plus Adapter (CTI, agents de centre de contacts)](https://www.five9.com/about/contact). En outre, pour poursuivre la configuration de l’authentification unique, suivez les étapes ci-dessous en fonction de l’adaptateur :

    a. Guide de l’administrateur « Five9 Plus Adapter for Agent Desktop Toolkit » : [https://webapps.five9.com/assets/files/for_customers/documentation/integrations/agent-desktop-toolkit/plus-agent-desktop-toolkit-administrators-guide.pdf](https://webapps.five9.com/assets/files/for_customers/documentation/integrations/agent-desktop-toolkit/plus-agent-desktop-toolkit-administrators-guide.pdf)
    
    b. Guide de l’administrateur « Five9 Plus Adapter for Microsoft Dynamics CRM » : [https://webapps.five9.com/assets/files/for_customers/documentation/integrations/microsoft/microsoft-administrators-guide.pdf](https://webapps.five9.com/assets/files/for_customers/documentation/integrations/microsoft/microsoft-administrators-guide.pdf)
    
    c. Guide de l’administrateur « Five9 Plus Adapter for Zendesk » : [https://webapps.five9.com/assets/files/for_customers/documentation/integrations/zendesk/zendesk-plus-administrators-guide.pdf](https://webapps.five9.com/assets/files/for_customers/documentation/integrations/zendesk/zendesk-plus-administrators-guide.pdf)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD 

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , sélectionnez **Utilisateurs** , puis sélectionnez **Tous les utilisateurs** .

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom** , entrez **BrittaSimon** .
  
    b. Dans le champ **Nom d’utilisateur** , tapez `brittasimon@yourcompanydomain.extension`. Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous autorisez Britta Simon à utiliser l’authentification unique Azure en accordant l’accès à Five9 Plus Adapter (CTI, agents de centre de contacts).

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **Five9 Plus Adapter (CTI, agents de centre de contacts)** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Five9 Plus Adapter (CTI, agents de centre de contacts)** .

    ![Lien Five9 Plus Adapter (CTI, agents de centre de contacts) dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-five9-plus-adapter-cti-contact-center-agents-test-user"></a>Créer un utilisateur de test Five9 Plus Adapter (CTI, agents de centre de contacts)

Dans cette section, vous allez créer un utilisateur appelé Britta Simon dans Five9 Plus Adapter (CTI, agents de centre de contacts). Travaillez avec l’[équipe de support Five9 Plus Adapter (CTI, agents de centre de contacts)](https://www.five9.com/about/contact) pour ajouter les utilisateurs dans la plateforme Five9 Plus Adapter (CTI, agents de centre de contacts). Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique. 

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette Five9 Plus Adapter (CTI, agents de centre de contacts) dans le panneau d’accès doit vous connecter automatiquement à l’application Five9 Plus Adapter (CTI, agents de centre de contacts) pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)