---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Salesforce | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Salesforce.
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
ms.openlocfilehash: 625058b131ef5cb6180873806185a3202a766118
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92675572"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-salesforce"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Salesforce

Dans ce tutoriel, vous allez découvrir comment intégrer Salesforce à Azure Active Directory (Azure AD). Quand vous intégrez Salesforce à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Salesforce.
* Permettre à vos utilisateurs de se connecter automatiquement à Salesforce avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Abonnement Salesforce pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Salesforce prend en charge l’authentification unique (SSO) initiée par le **fournisseur de services**

* Salesforce prend en charge le [provisionnement et le déprovisionnement **automatisés** des utilisateurs](salesforce-provisioning-tutorial.md) (recommandé)

* Salesforce prend en charge l’attribution d’utilisateurs **juste-à-temps**

* L’application Salesforce Mobile peut désormais être configurée avec Azure AD pour activer l’authentification unique. Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

## <a name="adding-salesforce-from-the-gallery"></a>Ajout de Salesforce à partir de la galerie

Pour configurer l’intégration de Salesforce avec Azure AD, vous devez ajouter Salesforce à partir de la galerie à votre liste d’applications SaaS gérées.

1. Connectez-vous au portail Azure avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Salesforce** dans la zone de recherche.
1. Sélectionnez **Salesforce** dans le volet des résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-sso-for-salesforce"></a>Configurer et tester l’authentification unique Azure AD pour Salesforce

Configurez et testez l’authentification unique Azure AD avec Salesforce pour un utilisateur de test nommé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Salesforce associé.

Pour configurer et tester l’authentification unique Azure AD avec Salesforce, effectuez les étapes suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Salesforce](#configure-salesforce-sso)** pour configurer les paramètres de l’authentification unique côté application.
    * **[Créer un utilisateur de test Salesforce](#create-salesforce-test-user)** pour disposer dans Salesforce d’un équivalent de B.Simon lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le portail Azure, accédez à la page d’intégration de l’application **Salesforce** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **URL de connexion** , tapez la valeur au format suivant :

    Compte d’entreprise : `https://<subdomain>.my.salesforce.com`

    Compte de développeur : `https://<subdomain>-dev-ed.my.salesforce.com`
    
    b. Dans la zone de texte **URL de réponse** , tapez la valeur au format suivant :

    Compte d’entreprise : `https://<subdomain>.my.salesforce.com`

    Compte de développeur : `https://<subdomain>-dev-ed.my.salesforce.com`

    c. Dans la zone de texte **Identificateur** , entrez la valeur au format suivant :

    Compte d’entreprise : `https://<subdomain>.my.salesforce.com`

    Compte de développeur : `https://<subdomain>-dev-ed.my.salesforce.com`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion et l’identificateur réels. Pour obtenir ces valeurs, contactez l’[équipe de support technique Salesforce](https://help.salesforce.com/support).

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **XML de métadonnées de fédération** et sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

1. Dans la section **Configurer Salesforce** , copiez la ou les URL appropriées, en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B. Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Salesforce.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Salesforce** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .
1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .
1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez qu’un rôle soit attribué aux utilisateurs, vous pouvez le sélectionner dans la liste déroulante **Sélectionner un rôle** . Si aucun rôle n’a été configuré pour cette application, vous voyez le rôle « Accès par défaut » sélectionné.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-salesforce-sso"></a>Configurer l’authentification unique Salesforce

1. Pour automatiser la configuration dans Salesforce, vous devez installer l’ **extension de navigateur My Apps Secure Sign-in** en cliquant sur **Installer l’extension** .

    ![Extension My apps](common/install-myappssecure-extension.png)

1. Après l’ajout de l’extension au navigateur, cliquez sur **Configurer Salesforce** pour être dirigé vers l’application Salesforce. À partir de là, indiquez les informations d’identification de l’administrateur pour vous connecter à Salesforce. Cette extension de navigateur configure automatiquement l’application et automatise les étapes 3 à 13.

    ![Configuration](common/setup-sso.png)

1. Si vous souhaitez configurer manuellement Salesforce, ouvrez une nouvelle fenêtre de navigateur web, connectez-vous à votre site d’entreprise Salesforce en tant qu’administrateur et effectuez les étapes suivantes :

1. Cliquez sur **Setup** sous **l’icône de paramètres** en haut à droite de la page.

    ![Configurer l’authentification unique – Icône des paramètres](./media/salesforce-tutorial/configure1.png)

1. Dans le volet de navigation, accédez à **SETTINGS** et cliquez sur **Identity** pour développer la section associée. Puis cliquez sur **Paramètres de l’authentification unique** .

    ![Configurer l’authentification unique – Paramètres](./media/salesforce-tutorial/sf-admin-sso.png)

1. Sur la page **Paramètres de l’authentification unique** , cliquez sur le bouton **Modifier** .

    ![Configurer l’authentification unique – Modifier](./media/salesforce-tutorial/sf-admin-sso-edit.png)

    > [!NOTE]
    > Si vous ne pouvez pas activer les paramètres de l’authentification unique pour votre compte Salesforce, il vous faudra peut-être contacter [l’équipe du support technique de Salesforce](https://help.salesforce.com/support).

1. Sélectionnez **SAML activé** , puis cliquez sur **Enregistrer** .

    ![Configurer l’authentification unique – SAML activé](./media/salesforce-tutorial/sf-enable-saml.png)

1. Pour configurer vos paramètres d’authentification unique SAML, cliquez sur **Nouveau à partir d’un fichier de métadonnées** .

    ![Configurer l’authentification unique – Nouveau à partir d’un fichier de métadonnées](./media/salesforce-tutorial/sf-admin-sso-new.png)

1. Cliquez sur **Sélectionner le fichier** pour charger le fichier XML de métadonnées que vous avez téléchargé à partir du Portail Azure, puis cliquez sur **Créer** .

    ![Configurer l’authentification unique – Sélectionner le fichier](./media/salesforce-tutorial/xmlchoose.png)

1. Dans la page **Paramètres d’authentification unique SAML** , les champs sont renseignés automatiquement. Sélectionnez **User Provisioning Enabled** (Attribution d’utilisateurs activée), puis cliquez sur **Enregistrer** .

    ![Configurer l’authentification unique – Provisionnement d’utilisateurs activé](./media/salesforce-tutorial/salesforcexml.png)

1. Dans le volet de navigation de gauche de Salesforce, cliquez sur **Company Settings** pour développer la section associée, puis cliquez sur **My Domain** .

    ![Configurer l’authentification unique – My domain](./media/salesforce-tutorial/sf-my-domain.png)

1. Faites défiler le contenu de la fenêtre jusqu’à la section **Configuration de l’authentification** , puis cliquez sur le bouton **Modifier** .

    ![Configurer l’authentification unique – Configuration de l’authentification](./media/salesforce-tutorial/sf-edit-auth-config.png)

1. Dans la section **Configuration de l’authentification** , cochez **AzureSSO** comme **Service d’authentification** de votre configuration SSO SAML, puis cliquez sur **Enregistrer** .

    ![Configurer l’authentification unique – Service d’authentification](./media/salesforce-tutorial/sf-auth-config.png)

    > [!NOTE]
    > Si plusieurs services d’authentification sont sélectionnés, les utilisateurs sont invités à choisir le service d’authentification qu’ils préfèrent utiliser pour se connecter lors de l’initialisation d’une authentification unique sur votre environnement Salesforce. Si vous ne voulez pas que cela se produise, vous devez **décocher toutes les cases en regard des autres services d’authentification** .

### <a name="create-salesforce-test-user"></a>Créer un utilisateur de test Salesforce

Dans cette section, un utilisateur appelé B.Simon est créé dans Salesforce. Salesforce prend en charge l’approvisionnement juste-à-temps, option activée par défaut. Vous n’avez aucune opération à effectuer dans cette section. Si un utilisateur n’existe pas dans Salesforce, un nouveau est créé lorsque vous tentez d’accéder à Salesforce. Salesforce prend également en charge l’attribution automatique d’utilisateurs. Des informations supplémentaires sur la configuration de l’attribution automatique d’utilisateurs sont disponibles [ici](salesforce-provisioning-tutorial.md).

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester votre configuration de l’authentification unique Azure AD avec les options suivantes. 

1. Cliquez sur **Tester cette application** dans le portail Azure. Cette opération redirige vers l’URL de connexion Salesforce, où vous pouvez lancer le processus de connexion. 

2. Accédez directement à l’URL de connexion Salesforce pour lancer le processus de connexion.

3. Vous pouvez utiliser le volet d’accès Microsoft. Le fait de cliquer sur la vignette Salesforce dans le volet d’accès doit vous connecter automatiquement à l’instance de Salesforce pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="test-sso-for-salesforce-mobile"></a>Tester l’authentification unique pour Salesforce (Mobile)

1. Ouvrez l’application mobile Salesforce. Dans la page de connexion, cliquez sur **Use Custom Domain** (Utiliser un domaine personnalisé).

    ![Application mobile Salesforce – Utiliser un domaine personnalisé](media/salesforce-tutorial/mobile-app1.png)

1. Dans la zone de texte **Custom Domain** (Domaine personnalisé), entrez votre nom de domaine personnalisé inscrit, puis cliquez sur **Continue** (Continuer).

    ![Application mobile Salesforce – Domaine personnalisé](media/salesforce-tutorial/mobile-app2.png)

1. Entrez vos informations d’identification Azure AD pour vous connecter à l’application Salesforce, puis cliquez sur **Next** (Suivant).

    ![Application mobile Salesforce – Informations d’identification Azure AD](media/salesforce-tutorial/mobile-app3.png)

1. Dans la page **Allow access** (Autoriser l’accès), comme indiqué ci-dessous, cliquez sur **Allow** (Autoriser) pour donner accès à l’application Salesforce.

    ![Application mobile Salesforce – Autoriser l’accès](media/salesforce-tutorial/mobile-app4.png)

1. Une fois la connexion réussie, la page d’accueil de l’application s’affiche.

    ![Page d’accueil de l’application mobile Salesforce](media/salesforce-tutorial/mobile-app5.png) ![Application mobile Salesforce](media/salesforce-tutorial/mobile-app6.png)

## <a name="next-steps"></a>Étapes suivantes

Après avoir configuré Salesforce, vous pouvez appliquer le contrôle de session, qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrir comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad)