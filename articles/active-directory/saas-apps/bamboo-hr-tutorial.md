---
title: 'Didacticiel : Intégration d’Azure Active Directory à BambooHR | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et BambooHR.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 11/02/2020
ms.author: jeedes
ms.openlocfilehash: 3926e98cb55d1afd80caf4af5d67910b66c09cdd
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96180334"
---
# <a name="tutorial-azure-active-directory-integration-with-bamboohr"></a>Didacticiel : Intégration d’Azure Active Directory à BambooHR

Dans ce tutoriel, vous allez apprendre à intégrer BambooHR à Azure Active Directory (Azure AD). Quand vous intégrez BambooHR à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à BambooHR.
* Permettre à vos utilisateurs de se connecter automatiquement à BambooHR avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Abonnement BambooHR pour lequel l’authentification unique (SSO) est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* BambooHR prend en charge l’authentification unique (SSO) initiée par **SP**

> [!NOTE]
> L’identificateur de cette application étant une valeur de chaîne fixe, une seule instance peut être configurée dans un locataire.


## <a name="adding-bamboohr-from-the-gallery"></a>Ajout de BambooHR à partir de la galerie

Pour configurer l’intégration de BambooHR à Azure AD, vous devez ajouter BambooHR, disponible dans la galerie, à votre liste d’applications SaaS gérées.

1. Connectez-vous au portail Azure avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **BambooHR** dans la zone de recherche.
1. Sélectionnez **BambooHR** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-sso-for-bamboohr"></a>Configurer et tester l’authentification unique (SSO) Azure AD pour BambooHR

Configurez et testez l’authentification unique Azure AD avec BambooHR pour un utilisateur de test nommé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur associé dans BambooHR.

Pour configurer et tester l’authentification unique Azure AD avec BambooHR, procédez comme suit :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
2. **[Configurer l’authentification unique BambooHR](#configure-bamboohr-sso)** pour configurer les paramètres de l’authentification unique côté application.
    * **[Créer un utilisateur de test BambooHR](#create-bamboohr-test-user)** pour avoir un équivalent de Britta Simon dans BambooHR lié à la représentation Azure AD de l’utilisateur.
3. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le portail Azure, accédez à la page d’intégration de l’application **BambooHR**, recherchez la section **Gérer** et sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base**, effectuez les étapes suivantes :

    a. Dans la zone de texte **URL de connexion**, saisissez une URL au format suivant : `https://<company>.bamboohr.com`

    b. Dans la zone de texte **URL de réponse**, tapez une URL au format suivant :

    | URL de réponse |
    |-----------|
    | `https://<company>.bamboohr.com/saml/consume.php` |
    | `https://<company>.bamboohr.co.uk/saml/consume.php` |

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de réponse et l’URL de connexion réelles. Pour obtenir les valeurs, contactez l’[équipe du support technique de BambooHR](https://www.bamboohr.com/contact.php). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

4. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** en fonction des options définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

6. Dans la section **Configurer BambooHR**, copiez la ou les URL appropriées, selon vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à BambooHR.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **BambooHR**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.
1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.
1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez qu’un rôle soit attribué aux utilisateurs, vous pouvez le sélectionner dans la liste déroulante **Sélectionner un rôle** . Si aucun rôle n’a été configuré pour cette application, vous voyez le rôle « Accès par défaut » sélectionné.
1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

## <a name="configure-bamboohr-sso"></a>Configurer l’authentification unique BambooHR

1. Dans une nouvelle fenêtre, connectez-vous à votre site d’entreprise BambooHR comme administrateur.

2. Dans la page d’accueil, effectuez les actions suivantes :
   
    ![Page Authentification unique BambooHR](./media/bamboo-hr-tutorial/ic796691.png "Single Sign on")   

    a. Sélectionnez **Applications**.
   
    b. Dans le volet **Applications**, sélectionnez **Authentification unique**.
   
    c. Sélectionnez **Authentification unique SAML**.

3. Dans le volet **Authentification unique SAML**, effectuez les actions suivantes :
   
    ![Volet Authentification unique SAML](./media/bamboo-hr-tutorial/IC796692.png "Authentification unique SAML")
   
    a. Dans la zone **URL de connexion SSO**, collez **l’URL de connexion** que vous avez copiée à partir du Portail Azure à l’étape 6.
      
    b. Dans le Bloc-notes, ouvrez le certificat codé en base 64 téléchargé dans le portail Azure, copiez son contenu, puis collez-le dans la zone **Certificat X.509**.
   
    c. Sélectionnez **Enregistrer**.

### <a name="create-bamboohr-test-user"></a>Créer un utilisateur de test BambooHR

Pour permettre aux utilisateurs Azure AD de se connecter à BambooHR, configurez-les manuellement dans BambooHR en effectuant les actions suivantes :

1. Connectez-vous à votre site **BambooHR** comme administrateur.

2. Dans la barre d’outils en haut, sélectionnez **Paramètres**.
   
    ![Le bouton Paramètres](./media/bamboo-hr-tutorial/IC796694.png "Paramètre")

3. Sélectionnez **Vue d’ensemble**.

4. Dans le volet gauche, sélectionnez **Sécurité** > **Utilisateurs**.

5. Tapez le nom d’utilisateur, le mot de passe et l’adresse e-mail du compte Azure AD valide que vous voulez configurer.

6. Sélectionnez **Enregistrer**.
        
>[!NOTE]
>Pour configurer des comptes d’utilisateur Azure AD, vous pouvez aussi utiliser les outils ou API de création de compte d’utilisateur de BambooHR.

### <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester votre configuration de l’authentification unique Azure AD avec les options suivantes. 

1. Cliquez sur **Tester cette application** dans le portail Azure. Vous êtes alors redirigé vers l’URL de connexion BambooHR, à partir de laquelle vous pouvez démarrer le flux de connexion. 

2. Accédez directement à l’URL de connexion BambooHR pour démarrer le flux de connexion.

3. Vous pouvez utiliser le volet d’accès Microsoft. Le fait de cliquer sur la vignette BambooHR dans le volet d’accès vous redirige vers l’URL de connexion BambooHR. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).


## <a name="next-steps"></a>Étapes suivantes

Après avoir configuré BambooHR, vous pouvez appliquer le contrôle de session, qui protège l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).