---
title: 'Didacticiel : Intégration d’Azure Active Directory à Keeper Password Manager & Digital Vault | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Keeper Password Manager & Digital Vault.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 11/13/2020
ms.author: jeedes
ms.openlocfilehash: b70c50e7c2900f884dd4d91c6650205bc626326e
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96178039"
---
# <a name="tutorial-azure-active-directory-integration-with-keeper-password-manager--digital-vault"></a>Didacticiel : Intégration d’Azure Active Directory à Keeper Password Manager & Digital Vault

Dans ce didacticiel, vous allez apprendre à intégrer Keeper Password Manager & Digital Vault à Azure Active Directory (Azure AD).
Cette intégration vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Keeper Password Manager & Digital Vault.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à Keeper Password Manager & Digital Vault (authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure.


## <a name="prerequisites"></a>Prérequis

Pour configurer l’intégration d’Azure AD à Keeper Password Manager & Digital Vault, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [essai d’un mois](https://azure.microsoft.com/pricing/free-trial/).
* Un abonnement Keeper Password Manager & Digital Vault pour lequel l’authentification unique (SSO) est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Keeper Password Manager & Digital Vault prend en charge l’authentification unique lancée par le fournisseur de services.

* Keeper Password Manager & Digital Vault prend en charge le provisionnement d’utilisateurs juste-à-temps.

## <a name="add-keeper-password-manager--digital-vault-from-the-gallery"></a>Ajouter de Keeper Password Manager & Digital Vault à partir de la galerie

Pour configurer l’intégration de Keeper Password Manager & Digital Vault à Azure AD, ajoutez l’application à votre liste d’applications SaaS (Software As A Service) managées, à partir de la galerie.

1. Connectez-vous au portail Azure avec un compte professionnel ou scolaire, ou avec un compte personnel Microsoft.
1. Dans le volet de gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans **Ajouter à partir de la galerie**, tapez **Keeper Password Manager & Digital Vault** au niveau de la zone de recherche.
1. Sélectionnez **Keeper Password Manager & Digital Vault** dans le volet des résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-sso-for-keeper-password-manager--digital-vault"></a>Configurer et tester l’authentification unique Azure AD pour Keeper Password Manager & Digital Vault

Configurez et testez l’authentification unique Azure AD avec Keeper Password Manager & Digital Vault au moyen d’un utilisateur de test nommé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Keeper Password Manager & Digital Vault associé.

Pour configurer et tester l’authentification unique Azure AD avec Keeper Password Manager & Digital Vault :

1. [Configurer l’authentification unique Azure AD](#configure-azure-ad-sso) pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.

    * [Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user) pour tester l’authentification unique Azure AD avec Britta Simon.
    * [Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user) pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.

1. [Configurer l’authentification unique Keeper Password Manager & Digital Vault](#configure-keeper-password-manager--digital-vault-sso) pour configurer les paramètres d’authentification unique côté application.
    * [Créer un utilisateur de test Keeper Password Manager & Digital Vault](#create-a-keeper-password-manager--digital-vault-test-user) pour avoir, dans Keeper Password Manager & Digital Vault, un équivalent de Britta Simon lié à la représentation Azure AD associée.
1. [Tester l’authentification unique](#test-sso) pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le portail Azure, accédez à la page d’intégration de l’application **Keeper Password Manager & Digital Vault**, recherchez la section **Gérer**. Sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, sélectionnez l’icône de crayon pour **Configuration SAML de base** afin de modifier les paramètres.

   ![Capture d’écran de Configurer l’authentification unique avec SAML, où l’icône de crayon est mise en évidence.](common/edit-urls.png)

4. Dans la section **Configuration SAML de base**, effectuez les étapes suivantes :

    a. Pour **URL de connexion**, tapez une URL qui utilise le modèle suivant :
    * Pour l’authentification unique cloud : `https://keepersecurity.com/api/rest/sso/saml/sso/<CLOUD_INSTANCE_ID>`
    * Pour l’authentification unique locale : `https://<KEEPER_FQDN>/sso-connect/saml/login`

    b. Pour **Identificateur (ID d’entité)** , tapez une URL dont le modèle est le suivant :
    * Pour l’authentification unique cloud : `https://keepersecurity.com/api/rest/sso/saml/<CLOUD_INSTANCE_ID>`
    * Pour l’authentification unique locale : `https://<KEEPER_FQDN>/sso-connect`

    c. Pour **URL de réponse**, tapez une URL qui utilise le modèle suivant :
    * Pour l’authentification unique cloud : `https://keepersecurity.com/api/rest/sso/saml/sso/<CLOUD_INSTANCE_ID>`
    * Pour l’authentification unique locale : `https://<KEEPER_FQDN>/sso-connect/saml/sso`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion, l’identificateur et l’URL de réponse réels. Pour obtenir ces valeurs, contactez [l’équipe du support technique de Keeper Password Manager & Digital Vault](https://keepersecurity.com/contact.html). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. L’application Keeper Password Manager & Digital Vault attend les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à votre configuration d’attributs de jetons SAML. La capture d’écran suivante montre la liste des attributs par défaut.

    ![Capture d’écran Attributs et revendications de l’utilisateur.](common/default-attributes.png)

1. De plus, l’application Keeper Password Manager & Digital Vault s’attend à ce que quelques attributs supplémentaires soient repassés dans la réponse SAML. C’est ce qui est illustré par le tableau suivant. Ces attributs sont également préremplis, mais vous pouvez les examiner pour voir s’ils répondent à vos besoins.

    | Nom | Attribut source|
    | ------------| --------- |
    | Premier | user.givenname |
    | Dernier | user.surname |
    | E-mail | user.mail |

5. Dans **Configurer l’authentification unique avec SAML**, à la section **Certificat de signature SAML**, sélectionnez **Télécharger**. Le fichier **XML des métadonnées de fédération** est ainsi téléchargé selon vos besoins à partir des options, et enregistré sur votre ordinateur.

    ![Capture d’écran de Certificat de signature SAML, avec mise en évidence de l’option Télécharger.](common/metadataxml.png)

6. Dans **Configurer Keeper Password Manager & Digital Vault**, copiez les URL appropriées, en fonction de vos besoins.

    ![Capture d’écran de Configurer Keeper Password Manager & Digital Vault, avec la mise en évidence des URL.](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD 

Dans cette section, vous créez un utilisateur de test nommé `B.Simon` dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** > **Utilisateurs** > **Tous les utilisateurs**.
1. Sélectionnez **Nouvel utilisateur**, en haut de l’écran.
1. Dans les propriétés **Utilisateur**, effectuez les étapes suivantes :
   1. Pour **Nom**, entrez `B.Simon`.  
   1. Entrez `username@companydomain.extension` comme **Nom d’utilisateur**. Par exemple : `B.Simon@contoso.com`.
   1. Sélectionnez **Afficher le mot de passe** et notez la valeur affichée.
   1. Sélectionnez **Create** (Créer).

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous autorisez B.Simon à utiliser l’authentification unique Azure en accordant l’accès à Keeper Password Manager & Digital Vault.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** > **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **Keeper Password Manager & Digital Vault**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.
1. Sélectionnez **Ajouter un utilisateur**. Dans **Ajouter une attribution**, sélectionnez **Utilisateurs et groupes**.
1. Dans **Utilisateurs et groupes**, sélectionnez **B.Simon** dans la liste des utilisateurs. Choisissez **Sélectionner** au bas de l’écran.
1. Si vous vous attendez à ce qu’un rôle soit attribué aux utilisateurs, vous pouvez le sélectionner dans la liste **Sélectionner un rôle**. Si aucun rôle n’a été configuré pour cette application, le rôle **Accès par défaut** est sélectionné.
1. Sous **Ajouter une attribution**, sélectionnez **Attribuer**.


## <a name="configure-keeper-password-manager--digital-vault-sso"></a>Configurer l’authentification unique Keeper Password Manager & Digital Vault

Configurez l’authentification unique pour l’application en vous reportant aux instructions fournies dans le [Guide de prise en charge de Keeper](https://docs.keeper.io/sso-connect-guide/).

### <a name="create-a-keeper-password-manager--digital-vault-test-user"></a>Créer un utilisateur de test Keeper Password Manager & Digital Vault

Pour permettre aux utilisateurs Azure AD de se connecter à Keeper Password Manager & Digital Vault, vous devez les provisionner. L’application prend en charge le provisionnement d’utilisateurs juste-à-temps ; après l’authentification, les utilisateurs sont créés automatiquement dans l’application. Si vous voulez configurer des utilisateurs manuellement, contactez le [support technique de Keeper](https://keepersecurity.com/contact.html).

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester votre configuration de l’authentification unique Azure AD avec les options suivantes. 

* Dans le portail Azure, sélectionnez **Tester cette application**. Vous êtes redirigé vers l’URL de connexion de Keeper Password Manager & Digital Vault, où vous pouvez lancer la connexion. 

* Vous pouvez accéder directement à l’URL de connexion de l’application, et y lancer la connexion.

* Vous pouvez utiliser le volet d’accès Microsoft. Lorsque vous sélectionnez la vignette **Keeper Password Manager & Digital Vault** dans le volet d’accès, vous êtes redirigé vers l’URL de connexion de l’application. Pour plus d’informations sur le volet d’accès, consultez [Se connecter et démarrer des applications à partir du portail Mes applications](../user-help/my-apps-portal-end-user-access.md).


## <a name="next-steps"></a>Étapes suivantes

Après avoir configuré Keeper Password Manager & Digital Vault, vous pouvez appliquer le contrôle de session. Cela protège l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. Pour plus d’informations, consultez [Découvrir comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).