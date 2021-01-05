---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Meraki Dashboard | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Meraki Dashboard.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/07/2020
ms.author: jeedes
ms.openlocfilehash: 0d5b6b7b3cf74ab740dbaeb8d86ab8bbb98e8531
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97357030"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-meraki-dashboard"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Meraki Dashboard

Dans ce tutoriel, vous allez découvrir comment intégrer Meraki Dashboard à Azure Active Directory (Azure AD). Quand vous intégrez Meraki Dashboard à Azure AD, vous pouvez :

* Contrôler qui dans Azure AD a accès à Meraki Dashboard
* Autoriser les utilisateurs à se connecter automatiquement à Meraki Dashboard avec leur compte Azure AD
* Gérer vos comptes à un emplacement central : le Portail Azure.

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Meraki Dashboard pour lequel l’authentification unique (SSO) est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Meraki Dashboard prend en charge l’authentification unique initiée par le **fournisseur d’identité**.

> [!NOTE]
> L’identificateur de cette application étant une valeur de chaîne fixe, une seule instance peut être configurée dans un locataire.

## <a name="adding-meraki-dashboard-from-the-gallery"></a>Ajout de Meraki Dashboard à partir de la galerie

Pour configurer l’intégration de Meraki Dashboard à Azure AD, vous devez ajouter Meraki Dashboard à votre liste d’applications SaaS managées à partir de la galerie.

1. Connectez-vous au portail Azure avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **Meraki Dashboard**  dans la zone de recherche.
1. Sélectionnez **Meraki Dashboard** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-sso-for-meraki-dashboard"></a>Configurer et tester l’authentification unique Azure AD pour Meraki Dashboard

Configurez et testez l’authentification unique Azure AD avec Meraki Dashboard à l’aide d’un utilisateur de test nommé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Meraki Dashboard associé.

Pour configurer et tester l’authentification unique Azure AD avec Meraki Dashboard, effectuez les étapes suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Meraki Dashboard](#configure-meraki-dashboard-sso)** pour configurer les paramètres de l’authentification unique côté application
    1. **[Créer un utilisateur de test Meraki Dashboard](#create-meraki-dashboard-test-user)** pour avoir un équivalent de B.Simon dans Meraki Dashboard qui soit lié à la représentation Azure AD de l’utilisateur
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le portail Azure, accédez à la page d’intégration de l’application **Meraki Dashboard**, recherchez la section **Gérer** et sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base**, effectuez les étapes suivantes :
     
    Dans la zone de texte **URL de réponse**, tapez une URL au format suivant : `https://n27.meraki.com/saml/login/m9ZEgb/< UNIQUE ID >`

    > [!NOTE]
    > La valeur de l’URL de réponse n’est pas réelle. Mettez à jour cette valeur avec l’URL de réponse réelle. La procédure est expliquée plus loin dans le tutoriel.

1. Cliquez sur le bouton **Enregistrer** .

1. L’application Meraki Dashboard attend les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à la configuration d’attributs de jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut.

    ![image](common/default-attributes.png)

1. En plus de ce qui précède, l’application Meraki Dashboard s’attend à ce que quelques attributs supplémentaires, indiqués ci-après, soient repassés dans la réponse SAML. Ces attributs sont également préremplis, mais vous pouvez les examiner pour voir s’ils répondent à vos besoins.
    
    | Nom | Attribut source|
    | ---------------| --------- |
    | `https://dashboard.meraki.com/saml/attributes/username` | user.userprincipalname |
    | `https://dashboard.meraki.com/saml/attributes/role` | user.assignedroles |

    > [!NOTE]
    > Pour comprendre comment configurer des rôles dans Azure AD, consultez [cette page](https://docs.microsoft.com/azure/active-directory/develop/howto-add-app-roles-in-azure-ad-apps#app-roles-ui).

1. Dans la section **Certificat de signature SAML**, cliquez sur le bouton **Modifier** pour ouvrir la boîte de dialogue **Certificat de signature SAML**.

    ![Modifier le certificat de signature SAML](common/edit-certificate.png)

1. Dans la section **Certificat de signature SAML**, copiez la **valeur de l’empreinte** et enregistrez-la sur votre ordinateur.

    ![Copier la valeur de l’empreinte](common/copy-thumbprint.png)

1. Dans la section **Configurer Meraki Dashboard**, copiez la valeur de l’URL de déconnexion et enregistrez-la sur votre ordinateur.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Meraki Dashboard.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **Meraki Dashboard**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.
1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.
1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

    ![rôle utilisateur](./media/meraki-dashboard-tutorial/user-role.png)

    > [!NOTE]
    > L’option **Sélectionner un rôle** sera désactivée et le rôle par défaut est UTILISATEUR pour l’utilisateur sélectionné.

1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

## <a name="configure-meraki-dashboard-sso"></a>Configurer l’authentification unique Meraki Dashboard

1. Dans une autre fenêtre de navigateur web, connectez-vous à Meraki Dashboard en tant qu’administrateur.

1. Accédez à **Organization** -> **Settings**.

    ![Meraki Dashboard - Onglet Settings (Paramètres)](./media/meraki-dashboard-tutorial/configure-1.png)

1. Sous Authentification, modifiez **SAML SSO** en choisissant **SAML SSO enabled**.

    ![Meraki Dashboard - Authentification](./media/meraki-dashboard-tutorial/configure-2.png)

1. Cliquez sur **Add a SAML IdP** (Ajouter un fournisseur d’identité SAML).

    ![Meraki Dashboard - Add a SAML IdP (Ajouter un fournisseur d’identité SAML)](./media/meraki-dashboard-tutorial/configure-3.png)

1. Collez la valeur **Empreinte** que vous avez copiée à partir du portail Azure dans la zone de texte **X.590 cert SHA1 fingerprint** (Certificat X.590, empreinte digitale SHA-1). Ensuite, cliquez sur **Enregistrer**. Après l’enregistrement, la valeur Consumer URL (URL du consommateur) s’affiche. Copiez la valeur Consumer URL, puis collez-la dans la zone de texte **URL de réponse** de la section **Configuration SAML de base** du portail Azure.

    ![Configuration de Meraki Dashboard](./media/meraki-dashboard-tutorial/configure-4.png)

### <a name="create-meraki-dashboard-test-user"></a>Créer un utilisateur de test Meraki Dashboard

1. Dans une autre fenêtre de navigateur web, connectez-vous à Meraki Dashboard en tant qu’administrateur.

1. Accédez à **Organization** -> **Administrators**.

    ![Meraki Dashboard - Administrators (Administrateurs)](./media/meraki-dashboard-tutorial/user-1.png)

1. Dans la section SAML administrator roles, cliquez sur le bouton **Add SAML role**.

    ![Meraki Dashboard - Bouton Add SAML role (Ajouter un rôle SAML)](./media/meraki-dashboard-tutorial/user-2.png)

1. Entrez le rôle **meraki_full_admin**, choisissez **Full** pour **Organization access**, puis cliquez sur **Create role**. Répétez le processus pour **meraki_readonly_admin** mais en choisissant **Read-only** pour **Organization access**.
 
    ![Meraki Dashboard - Create user (Créer un utilisateur)](./media/meraki-dashboard-tutorial/user-3.png)

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester votre configuration de l’authentification unique Azure AD avec les options suivantes.

* Dans le portail Azure, cliquez sur Tester cette application. Vous êtes alors automatiquement connecté à l’instance de Meraki Dashboard pour laquelle vous avez configuré l’authentification unique.

* Vous pouvez utiliser Mes applications de Microsoft. Quand vous cliquez sur la vignette Meraki Dashboard dans Mes applications, vous devez être connecté automatiquement à l’application Meraki Dashboard pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur Mes applications, consultez [Présentation de Mes applications](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).


## <a name="next-steps"></a>Étapes suivantes

Après avoir configuré Meraki Dashboard, vous pouvez appliquer le contrôle de session qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).