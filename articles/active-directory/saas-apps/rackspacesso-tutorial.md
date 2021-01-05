---
title: "Tutoriel : Intégration d'Azure Active Directory à Rackspace SSO | Microsoft Docs"
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Rackspace SSO.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/15/2019
ms.author: jeedes
ms.openlocfilehash: eed0ea075c5e9eb39810259be2aa6781a59f2f22
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92511645"
---
# <a name="tutorial-azure-active-directory-integration-with-rackspace-sso"></a>Tutoriel : Intégration d'Azure Active Directory à Rackspace SSO

Dans ce didacticiel, vous allez apprendre à intégrer Rackspace SSO à Azure Active Directory (Azure AD).
L’intégration de Rackspace SSO à Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Rackspace SSO.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à Rackspace SSO (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour configurer l’intégration d’Azure AD à Rackspace SSO, vous devez disposer des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/)
* Un abonnement Rackspace SSO pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Rackspace prend en charge l’authentification unique lancée par le **fournisseur de services**

## <a name="adding-rackspace-sso-from-the-gallery"></a>Ajout de Rackspace SSO à partir de la galerie

Pour configurer l’intégration de Rackspace SSO à Azure AD, vous devez ajouter Rackspace SSO à partir de la galerie à votre liste d’applications SaaS managées.

**Pour ajouter Rackspace SSO à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Rackspace SSO** , sélectionnez **Rackspace SSO** dans le panneau de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![Rackspace SSO dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Rackspace SSO grâce à un utilisateur de test appelé **Britta Simon** .
Quand vous utilisez l’authentification unique avec Rackspace, les utilisateurs Rackspace sont automatiquement créés la première fois qu’ils se connectent au portail Rackspace. 

Pour configurer et tester l’authentification unique Azure AD avec Rackspace SSO, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Rackspace SSO](#configure-rackspace-sso-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer le mappage d’attributs dans le panneau de configuration de Rackspace](#set-up-attribute-mapping-in-the-rackspace-control-panel)** pour attribuer des rôles Rackspace aux utilisateurs Azure AD.
1. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Rackspace SSO, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Rackspace SSO** , sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , chargez le **fichier de métadonnées du fournisseur de services** que vous pouvez télécharger depuis l’ [URL](https://login.rackspace.com/federate/sp.xml) et effectuez les étapes suivantes :

    a. Cliquez sur **Charger un fichier de métadonnées** .

    ![Capture d’écran montrant la Configuration SAML de base avec le lien Charger un fichier de métadonnées.](common/upload-metadata.png)

    b. Cliquez sur le **logo du dossier** pour sélectionner le fichier de métadonnées, puis cliquez sur **Charger** .

    ![Capture d’écran montrant une boîte de dialogue dans laquelle vous pouvez sélectionner et charger un fichier.](common/browse-upload-metadata.png)

    c. Une fois que le fichier de métadonnées est chargé, les URL nécessaires sont renseignées automatiquement.

    d. Dans la zone de texte **URL de connexion** , tapez une URL : `https://login.rackspace.com/federate/`

    ![Informations d’authentification unique dans Domaine et URL Rackspace SSO](common/sp-signonurl.png)   

5. Sur la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le fichier **XML de métadonnées de fédération** en fonction des options définies selon vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

Ce fichier sera chargé sur Rackspace pour renseigner les paramètres de configuration de la fédération des identités nécessaires.

### <a name="configure-rackspace-sso-single-sign-on"></a>Configurer l’authentification unique Rackspace SSO

Pour configurer l’authentification unique du côté **Rackspace SSO**  :

1. Consultez la documentation de [Ajouter un fournisseur d’identité au panneau de configuration](https://developer.rackspace.com/docs/rackspace-federation/gettingstarted/add-idp-cp/)
1. Elle vous guide à travers les étapes permettant de :
    1. Créer un fournisseur d’identité
    1. Spécifiez un domaine de messagerie que les utilisateurs utiliseront pour identifier votre entreprise lors de la connexion.
    1. Chargez le fichier **XML des métadonnées de fédération** précédemment téléchargé à partir du panneau de configuration d’Azure.

Ceci configure correctement les paramètres de l’authentification unique de base nécessaires pour qu’Azure et Rackspace se connectent.

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

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Rackspace SSO.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **Rackspace SSO** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Rackspace SSO** .

    ![Lien Rackspace SSO dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="set-up-attribute-mapping-in-the-rackspace-control-panel"></a>Configurer le mappage d’attributs dans le panneau de configuration de Rackspace

Rackspace utilise une **stratégie de mappage d’attributs** pour affecter des rôles et des groupes Rackspace à vos utilisateurs avec authentification unique. La **stratégie de mappage d’attributs** traduit les revendications SAML Azure AD en champs de configuration des utilisateurs nécessaires à Rackspace. Vous trouverez plus d’informations dans la [documentation sur les concepts de base du mappage d’attributs](https://developer.rackspace.com/docs/rackspace-federation/appendix/map/) de Rackspace. Certaines considérations s’appliquent :

* Si vous voulez affecter différents niveaux d’accès Rackspace avec des groupes Azure AD, vous devez activer la revendication des groupes dans les paramètres d’authentification unique **Rackspace SSO** d’Azure. La **stratégie de mappage d’attributs** est alors utilisée pour mettre en correspondance ces groupes avec les rôles et les groupes Rackspace souhaités :

    ![Les paramètres de revendication des groupes](common/sso-groups-claim.png)

* Par défaut, Azure AD envoie l’UID des groupes Azure AD dans la revendication SAML, et non pas leur nom. Cependant, si vous synchronisez votre annuaire Active Directory avec Azure AD, vous avez la possibilité d’envoyer les noms réels des groupes :

    ![Les paramètres des noms de revendication des groupes](common/sso-groups-claims-names.png)

L’exemple suivant de **stratégie de mappage d’attributs** montre :
1. Définition du nom de l’utilisateur Rackspace sur la revendication SAML `user.name`. Vous pouvez utiliser n’importe quelle revendication, mais il est plus courant de définir ceci sur un champ contenant l’adresse e-mail de l’utilisateur.
1. Définition des rôles Rackspace `admin` et `billing:admin` sur un utilisateur en mettant en correspondance un groupe Azure AD, sur la base du nom de groupe ou de l’UID de groupe. Une *substitution* de `"{0}"` dans le champ `roles` est utilisée, et elle sera remplacée par les résultats des expressions de règle `remote`.
1. Utilisation de la *substitution par défaut* de `"{D}"` pour permettre à Rackspace de récupérer des champs SAML supplémentaires en recherchant des revendications SAML standard et connues dans l’échange SAML.

```yaml
---
mapping:
    rules:
    - local:
        user:
          domain: "{D}"
          name: "{At(http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name)}"
          email: "{D}"
          roles:
              - "{0}"
          expire: "{D}"
      remote:
          - path: |
              (
                if (mapping:get-attributes('http://schemas.microsoft.com/ws/2008/06/identity/claims/groups')='7269f9a2-aabb-9393-8e6d-282e0f945985') then ('admin', 'billing:admin') else (),
                if (mapping:get-attributes('http://schemas.microsoft.com/ws/2008/06/identity/claims/groups')='MyAzureGroup') then ('admin', 'billing:admin') else ()
              )
            multiValue: true
  version: RAX-1
```
> [!TIP]
> Veillez à utiliser un éditeur de texte qui valide la syntaxe YAML lors de la modification de votre fichier de stratégie.

Pour obtenir plus d’exemples, consultez la [documentation des concepts de base du mappage d’attributs](https://developer.rackspace.com/docs/rackspace-federation/appendix/map/) de Rackspace.

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Rackspace SSO dans le volet d’accès, vous devez être connecté automatiquement à l’application Rackspace SSO pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

Vous pouvez également utiliser le bouton **Valider** dans les paramètres d’authentification unique de **Rackspace SSO**  :

   ![Bouton de validation de SSO](common/sso-validate-sign-on.png)

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)