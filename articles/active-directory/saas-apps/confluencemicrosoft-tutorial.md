---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Confluence SAML SSO by Microsoft | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Confluence SAML SSO by Microsoft.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/05/2019
ms.author: jeedes
ms.openlocfilehash: adee491625cec3b4a22cc7b3d1f3ba17595d9e87
ms.sourcegitcommit: d79513b2589a62c52bddd9c7bd0b4d6498805dbe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/18/2020
ms.locfileid: "97673248"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-confluence-saml-sso-by-microsoft"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Confluence SAML SSO by Microsoft

Dans ce tutoriel, vous allez apprendre à intégrer Confluence SAML SSO by Microsoft à Azure Active Directory (Azure AD). Quand vous intégrez Confluence SAML SSO by Microsoft à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Confluence SAML SSO by Microsoft.
* Permettre à vos utilisateurs de se connecter automatiquement à Confluence SAML SSO by Microsoft avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="description"></a>Description :

Utilisez votre compte Microsoft Azure Active Directory avec le serveur Atlassian Confluence pour activer l’authentification unique. Ainsi, l’ensemble des utilisateurs de votre organisation peuvent utiliser les informations d’identification d’Azure AD pour se connecter à l’application Confluence. Ce plug-in utilise SAML 2.0 pour la fédération.

## <a name="prerequisites"></a>Prérequis

Pour configurer l’intégration d’Azure AD à Confluence SAML SSO by Microsoft, vous avez besoin des éléments suivants :

- Un abonnement Azure AD
- Une application serveur Confluence installée sur un serveur Windows 64 bits (local ou dans l’infrastructure IaaS cloud)
- L’activation du HTTPS dans le serveur
- Notez que les versions prises en charge par le plug-in Confluence sont mentionnées dans la section ci-dessous.
- L’accessibilité du serveur Confluence via Internet (particulièrement pour la page de connexion Azure AD pour l’authentification) et la capacité à recevoir le jeton d’Azure AD
- La création d’informations d’identification administrateur dans Confluence
- La désactivation de WebSudo dans Confluence
- La création d’un utilisateur de test dans l’application serveur Confluence

> [!NOTE]
> Il est déconseillé d’utiliser un environnement de production Confluence pour tester les étapes de ce didacticiel. Testez d’abord l’intégration dans l’environnement de développement ou l’environnement intermédiaire de l’application, puis passez à l’environnement de production.

> [!NOTE]
> Cette intégration peut également être utilisée à partir de l’environnement cloud US Government Azure AD. Cette application est disponible dans la Galerie d’applications cloud US Government Azure AD et peut être configurée de la même façon que dans le cloud public.

Pour commencer, vous devez disposer de ce qui suit :

* N’utilisez pas votre environnement de production, sauf si cela est nécessaire.
* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Confluence SAML SSO by Microsoft pour lequel l’authentification unique est activée.

## <a name="supported-versions-of-confluence"></a>Versions de Confluence prises en charge

Les versions suivantes de Confluence sont actuellement prises en charge :

- Confluence : 5.0 à 5.10
- Confluence : 6.0.1 à 6.15.9
- Confluence : 7.0.1 à 7.9.3

> [!NOTE]
> Veuillez noter que nos plug-ins Confluence fonctionnent également sur Ubuntu Version 16.04

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Confluence SAML SSO by Microsoft prend en charge l’authentification unique initiée par le **fournisseur de services**

## <a name="adding-confluence-saml-sso-by-microsoft-from-the-gallery"></a>Ajout de Confluence SAML SSO by Microsoft à partir de la galerie

Pour configurer l’intégration de Confluence SAML SSO by Microsoft à Azure AD, vous devez ajouter Confluence SAML SSO by Microsoft, disponible dans la galerie, à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **Confluence SAML SSO by Microsoft** dans la zone de recherche.
1. Sélectionnez **Confluence SAML SSO by Microsoft** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-confluence-saml-sso-by-microsoft"></a>Configurez et testez l’authentification unique Azure AD pour Confluence SAML SSO by Microsoft

Configurez et testez l’authentification unique Azure AD avec Confluence SAML SSO by Microsoft à l’aide d’un utilisateur de test nommé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Confluence SAML SSO by Microsoft associé.

Pour configurer et tester l’authentification unique Azure AD avec Confluence SAML SSO by Microsoft, effectuez les modules suivants :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Confluence SAML SSO by Microsoft](#configure-confluence-saml-sso-by-microsoft-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test Confluence SAML SSO by Microsoft](#create-confluence-saml-sso-by-microsoft-test-user)** pour avoir un équivalent de B.Simon dans Confluence SAML SSO by Microsoft lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Confluence SAML SSO by Microsoft**, recherchez la section **Gérer** et sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base**, entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **URL de connexion**, tapez une URL au format suivant : `https://<domain:port>/plugins/servlet/saml/auth`.

    b. Dans la zone de texte **Identificateur**, tapez une URL en utilisant le format suivant : `https://<domain:port>/`

    c. Dans la zone de texte **URL de réponse**, tapez une URL au format suivant : `https://<domain:port>/plugins/servlet/saml/auth`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur, l’URL de réponse et l’URL de connexion réels. Le port est facultatif s’il s’agit d’une URL nommée. Ces valeurs sont reçues durant la configuration du plug-in Confluence qui est décrite plus loin dans le didacticiel.

1. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, cliquez sur le bouton Copier pour copier l’**URL des métadonnées de fédération d’application**, puis enregistrez-la sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/copy-metadataurl.png)

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Confluence SAML SSO by Microsoft.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **Confluence SAML SSO by Microsoft**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

## <a name="configure-confluence-saml-sso-by-microsoft-sso"></a>Configurer l’authentification unique Confluence SAML SSO by Microsoft

1. Dans une autre fenêtre de navigateur web, connectez-vous à votre instance de Confluence en tant qu’administrateur.

1. Pointez sur le roue dentée, puis cliquez sur **Modules complémentaires**.

    ![Capture d’écran montrant l’icône « Roue dentée » sélectionnée et « Add-ons » mis en évidence dans le menu déroulant.](./media/confluencemicrosoft-tutorial/addon1.png)

1. Téléchargez le plug-in depuis le [Centre de téléchargement Microsoft](https://www.microsoft.com/download/details.aspx?id=56503). Chargez manuellement le plug-in fourni par Microsoft à l’aide du menu **Upload add-on** (Charger le module complémentaire). Le téléchargement du plug-in est couvert dans [Contrat de Services Microsoft](https://www.microsoft.com/servicesagreement/).

    ![Capture d’écran montrant la page « Manage add-ons » avec l’action « Upload add-on » sélectionnée.](./media/confluencemicrosoft-tutorial/addon12.png)

1. Pour exécuter le scénario de proxy inverse Confluence ou le scénario d’équilibreur de charge, effectuez les étapes suivantes :

    > [!NOTE]
    > Vous devez d’abord configurer le serveur à l’aide des instructions ci-dessous, puis installer le plug-in.

    a. Ajoutez l’attribut ci-dessous au port **connecteur** dans le fichier **server.xml** de l’application serveur JIRA.

    `scheme="https" proxyName="<subdomain.domain.com>" proxyPort="<proxy_port>" secure="true"`

    ![Capture d’écran montrant le fichier « server.xml » avec l’attribut ajouté au port « connecteur ».](./media/confluencemicrosoft-tutorial/reverseproxy1.png)

    b. Modifiez l’**URL de base** dans les **paramètres système** en fonction du proxy/de l’équilibreur de charge.

    ![Capture d’écran montrant la page « Administration - Settings » avec « Base URL » mis en évidence.](./media/confluencemicrosoft-tutorial/reverseproxy2.png)

1. Une fois que le plug-in est installé, il s’affiche sous **User Installed** (Installé par l’utilisateur), dans la section **Manage add-ons** (Gérer les modules complémentaires). Cliquez sur **Configurer** pour configurer le nouveau plug-in.

    ![Capture d’écran montrant la section « User Installed » avec le bouton « Configure » mis en évidence.](./media/confluencemicrosoft-tutorial/addon15.png)

1. Effectuez les opérations suivantes dans la page de configuration :

    ![Capture d’écran montrant la page de configuration de l’authentification unique.](./media/confluencemicrosoft-tutorial/addon54.png)

    > [!TIP]
    > Vérifiez qu’un seul certificat est associé à l’application pour éviter toute erreur liée à la résolution des métadonnées. Si plusieurs certificats sont associés, l’administrateur verra un message d’erreur s’afficher lors de la résolution des métadonnées.

    1. Dans la zone de texte **URL des métadonnées**, collez la valeur **URL des métadonnées de fédération de l’application** que vous avez copiée sur le portail Azure, puis cliquez sur le bouton **Résoudre**. L’URL des métadonnées IdP est alors lue et tous les champs sont renseignés.

    1. Copiez les valeurs des champs **Identifier, Reply URL et Sign on URL**, puis collez-les dans les zones de texte **Identificateur, URL de réponse et URL de connexion** correspondantes dans la section **Configuration SAML de base** du portail Azure.

    1. Dans **Login Button Name** (Nom du bouton de connexion), tapez le nom du bouton que les utilisateurs doivent voir sur l’écran de connexion.

    1. Dans **Login Button Description** (Description du bouton de connexion), tapez la description du bouton que les utilisateurs doivent voir sur l’écran de connexion.

    1. Dans **SAML User ID Locations** (Emplacements de l’identificateur d’utilisateur SAML), sélectionnez **User ID is in the NameIdentifier element of the Subject statement** (L’ID utilisateur se trouve dans l’élément NameIdentifier de l’instruction Subject ) ou **User ID is in an Attribute element** (L’identificateur d’utilisateur se trouve dans l’élément Attribute).  Cet ID doit être l’ID d’utilisateur Confluence. Si aucun ID d’utilisateur correspondant n’est trouvé, le système n’autorise pas l’utilisateur à se connecter. 

       > [!Note]
       > Par défaut, l’emplacement de l’identificateur d’utilisateur SAML est défini sur l’élément NameIdentifier. Vous pouvez remplacer cela par une option d’attribut et entrer le nom de l’attribut souhaité.

    1. Si vous sélectionnez l’option **User ID is in an Attribute element** (L’identificateur d’utilisateur se trouve dans l’élément Attribute), dans la zone de texte **Attribute name** (Nom de l’attribut), tapez le nom de l’attribut dans lequel l’ID d’utilisateur doit se trouver. 

    1. Si vous utilisez le domaine fédéré (AD FS, etc.) avec Azure AD, cochez l’option **Enable Home Realm Discovery** (Activer la détection de domaine d’accueil), puis entrez un nom de domaine sous **Domain Name**.

    1. Si la connexion est basée sur AD FS, tapez le nom du domaine dans le champ **Domain Name**.

    1. Cochez l’option **Enable Single Sign out** (Activer la déconnexion unique) si vous souhaitez qu’un utilisateur soit déconnecté d’Azure AD quand il se déconnecte de Confluence. 

    1. Cochez l’option **Force Azure Login** (Forcer la connexion Azure) pour vous connecter avec les informations d’identification Azure AD uniquement.

       > [!Note]
       > Si vous souhaitez activer le formulaire de connexion par défaut pour la connexion d’administrateur dans la page de connexion quand l’option Force Azure Login est activée, ajoutez le paramètre de requête dans l’URL du navigateur.
       > `https://<domain:port>/login.action?force_azure_login=false`

    1. Cliquez sur **Enregistrer** pour enregistrer les paramètres.

       > [!NOTE]
       > Pour plus d’informations sur l’installation et la résolution des problèmes, consultez le [Guide d’administration du connecteur d’authentification unique MS Confluence](./ms-confluence-jira-plugin-adminguide.md). Il existe également une page [FAQ](./ms-confluence-jira-plugin-adminguide.md) pour vous aider.

### <a name="create-confluence-saml-sso-by-microsoft-test-user"></a>Créer un utilisateur de test Confluence SAML SSO by Microsoft

Pour pouvoir se connecter à un serveur local Confluence, les utilisateurs Azure AD doivent être provisionnés dans Confluence SAML SSO by Microsoft. Pour Confluence SAML SSO by Microsoft, l’attribution des utilisateurs se fait manuellement.

**Pour approvisionner un compte d’utilisateur, procédez comme suit :**

1. Connectez-vous à votre serveur local Confluence en tant qu’administrateur.

1. Pointez sur la roue dentée, puis cliquez sur **Gestion des utilisateurs**.

    ![Ajouter un employé](./media/confluencemicrosoft-tutorial/user1.png)

1. Dans la section Utilisateurs, cliquez sur l’onglet **Add users** (Ajouter des utilisateurs). Dans la page de boîte de dialogue **Add a User** (Ajouter un utilisateur), procédez comme suit :

    ![Capture d’écran montrant l’administration Confluence avec l’onglet « Add Users » sélectionné et les informations « Add a User » entrées.](./media/confluencemicrosoft-tutorial/user2.png)

    a. Dans la zone de texte **Username** (Nom d’utilisateur), tapez le nom d’un utilisateur, par exemple B.Simon.

    b. Dans la zone de texte **Full Name** (Nom complet), tapez le nom complet d’un utilisateur, par exemple B.Simon.

    c. Dans la zone de texte **Email** (E-mail), tapez l’adresse e-mail d’un utilisateur, par exemple, B.Simon@contoso.com.

    d. Dans la zone de texte **Password** (Mot de passe), tapez le mot de passe de B.Simon.

    e. Cliquez sur **Confirm Password** (Confirmer le mot de passe), puis entrez de nouveau le mot de passe.

    f. Cliquez sur le bouton **Ajouter**.

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Confluence SAML SSO by Microsoft dans le volet d’accès, vous devez être connecté automatiquement à l’application Confluence SAML SSO by Microsoft pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Confluence SAML SSO by Microsoft avec Azure AD](https://aad.portal.azure.com/)
