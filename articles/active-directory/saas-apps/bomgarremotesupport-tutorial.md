---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à BeyondTrust Remote Support | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et BeyondTrust Remote Support.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 11/12/2020
ms.author: jeedes
ms.openlocfilehash: c11d8aaa578006c7dbd96b457399df5b17fd9bd8
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95914990"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-beyondtrust-remote-support"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à BeyondTrust Remote Support

Dans ce tutoriel, vous allez apprendre à intégrer BeyondTrust Remote Support à Azure Active Directory (Azure AD). En intégrant BeyondTrust Remote Support à Azure AD, vous pouvez :

* Contrôler, dans Azure AD, qui a accès à BeyondTrust Remote Support
* Permettre aux utilisateurs de se connecter automatiquement à BeyondTrust Remote Support avec leur compte Azure AD
* Gérer vos comptes à un emplacement central : le Portail Azure.

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement BeyondTrust Remote Support pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* BeyondTrust Remote Support prend en charge l’authentification unique initiée par le **fournisseur de services**
* BeyondTrust Remote Support prend en charge le provisionnement d’utilisateurs **Juste-à-temps**.

## <a name="adding-beyondtrust-remote-support-from-the-gallery"></a>Ajout de BeyondTrust Remote Support à partir de la galerie

Pour configurer l’intégration de BeyondTrust Remote Support à Azure AD, vous devez ajouter BeyondTrust Remote Support à votre liste d’applications SaaS managées à partir de la galerie.

1. Connectez-vous au portail Azure avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **BeyondTrust Remote Support** dans la zone de recherche.
1. Sélectionnez **BeyondTrust Remote Support** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-beyondtrust-remote-support"></a>Configurer et tester l’authentification unique Azure AD pour BeyondTrust Remote Support

Configurez et testez l’authentification unique Azure AD auprès de BeyondTrust Remote Support avec un utilisateur de test appelé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur BeyondTrust Remote Support associé.

Pour configurer et tester l’authentification unique Azure AD avec BeyondTrust Remote Support, effectuez les étapes suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique BeyondTrust Remote Support](#configure-beyondtrust-remote-support-sso)** pour configurer les paramètres de l’authentification unique côté application.
    * **[Créer un utilisateur de test BeyondTrust Remote Support](#create-beyondtrust-remote-support-test-user)** pour avoir un équivalent de B.Simon dans BeyondTrust Remote Support lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le portail Azure, accédez à la page d’intégration de l’application **BeyondTrust Remote Support**, recherchez la section **Gérer** et sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base**, entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **URL de connexion**, tapez une URL au format suivant : `https://<HOSTNAME>.bomgar.com/saml`.

    b. Dans la zone de texte **Identificateur**, tapez une URL en utilisant le format suivant : `https://<HOSTNAME>.bomgar.com`

    c. Dans la zone de texte **URL de réponse**, tapez une URL au format suivant : `https://<HOSTNAME>.bomgar.com/saml/sso`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion, l’identificateur et l’URL de réponse réels. Ces valeurs vous seront expliquées plus loin dans le tutoriel.

1. L’application BeyondTrust Remote Support s’attend à recevoir les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à la configuration des attributs du jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut.

    ![image](common/default-attributes.png)

1. En plus de ce qui précède, l’application BeyondTrust Remote Support s’attend à ce que quelques attributs supplémentaires (présentés ci-dessous) soient repassés dans la réponse SAML. Ces attributs sont également préremplis, mais vous pouvez les examiner pour voir s’ils répondent à vos besoins.

    | Nom |  Attribut source|
    | ---------------| ----------|
    | Nom d’utilisateur | user.userprincipalname |
    | FirstName | user.givenname |
    | LastName | user.surname |
    | E-mail | user.mail |
    | Groupes | user.groups |

    > [!NOTE]
    > Quand vous attribuez des groupes Azure AD à l’application BeyondTrust Remote Support, vous devez définir l’option « Groupes renvoyés dans la revendication » (initialement définie sur Aucun) sur SecurityGroup. Les groupes sont importés dans l’application en tant qu’ID d’objet. Pour connaître l’ID d’objet du groupe Azure AD, accédez aux propriétés dans l’interface Azure Active Directory. Vous en aurez besoin pour référencer les groupes Azure AD et les attribuer aux stratégies de groupe appropriées.

1. Quand vous définissez l’identificateur unique de l’utilisateur, vous devez définir son format d’ID de nom sur **Persistant** pour que l’utilisateur soit correctement identifié et associé aux stratégies de groupe appropriées pour les autorisations. Cliquez sur l’icône de modification pour ouvrir la boîte de dialogue **Attributs utilisateur et revendications** et modifier la valeur de l’identificateur unique de l’utilisateur.

1. Dans la section **Gérer la revendication**, cliquez sur **Choisir le format du nom de l’identificateur**. Définissez la valeur sur **Persistant**, puis cliquez sur **Enregistrer**.

    ![Attributs utilisateur et revendications](./media/bomgarremotesupport-tutorial/attribute-unique-user-identifier.png)

1. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, recherchez **XML de métadonnées de fédération** et sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

1. Dans la section **Configurer BeyondTrust Remote Support**, copiez l’URL ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à BeyondTrust Remote Support.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **BeyondTrust Remote Support**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.
1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.
1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez qu’un rôle soit attribué aux utilisateurs, vous pouvez le sélectionner dans la liste déroulante **Sélectionner un rôle** . Si aucun rôle n’a été configuré pour cette application, vous voyez le rôle « Accès par défaut » sélectionné.
1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

## <a name="configure-beyondtrust-remote-support-sso"></a>Configurer l’authentification unique BeyondTrust Remote Support

1. Dans une autre fenêtre de navigateur web, connectez-vous à BeyondTrust Remote Support en tant qu’administrateur.

1. Accédez à **Users & Security** > **Security Providers**.

1. Cliquez sur l’icône **Modifier** dans **Fournisseurs SAML**.

    ![Icône de modification des fournisseurs SAML](./media/bomgarremotesupport-tutorial/saml-providers.png)

1. Développez la section **Paramètres du fournisseur de services**.

1. Cliquez sur **Download Service Provider Metadata** (Télécharger les métadonnées du fournisseur de services) ou copiez les valeurs d’**ID d’entité** et d’**URL ACS** pour les utiliser dans la section **Configuration SAML de base** dans le portail Azure.

    ![Télécharger les métadonnées du fournisseur de services](./media/bomgarremotesupport-tutorial/service-provider-metadata.png)


1. Sous la section Paramètres du fournisseur d’identité, cliquez sur **Charger les métadonnées du fournisseur d’identité** et recherchez le fichier XML de métadonnées que vous avez téléchargé à partir du portail Azure.

1.  Les valeurs d’**ID d’entité**, d’**URL du service d’authentification unique** et de **certificat de serveur** sont automatiquement chargées et vous devez remplacer la **liaison de protocole d’URL d’authentification unique** par **HTTP POST**.

    ![Capture d’écran montrant la section Identity Provider Settings où vous effectuez ces actions.](./media/bomgarremotesupport-tutorial/identity-provider.png)

1. Cliquez sur **Enregistrer**.

### <a name="create-beyondtrust-remote-support-test-user"></a>Créer un utilisateur de test BeyondTrust Remote Support

Nous allons à présent configurer les paramètres de provisionnement d’utilisateurs. Les valeurs utilisées dans cette section sont référencées à partir de la section **Attributs utilisateur et revendications** du portail Azure. Nous les avons configurées comme valeurs par défaut, et elles sont déjà importées au moment de la création. Toutefois, elles peuvent être personnalisées si nécessaire.

![Capture d’écran montrant les paramètres de provisionnement d’utilisateur où vous pouvez configurer des valeurs utilisateur.](./media/bomgarremotesupport-tutorial/user-attribute.png)

> [!NOTE]
> Les groupes et l’attribut d’e-mail ne sont pas nécessaires pour cette implémentation. Si vous utilisez des groupes Azure AD et que vous les attribuez à des stratégies de groupe BeyondTrust Remote Support pour les autorisations, l’ID d’objet du groupe doit être référencé à partir de ses propriétés dans le portail Azure et placé dans la section « Available Groups ». Une fois cette opération effectuée, l’ID d’objet/le groupe AD peuvent être attribués à une stratégie de groupe pour les autorisations.

![Capture d’écran montrant la section IT avec Membership type, Source, Type et Object ID.](./media/bomgarremotesupport-tutorial/config-user2.png)

![Capture d’écran montrant la page des paramètres de base pour une stratégie de groupe.](./media/bomgarremotesupport-tutorial/group-policy.png)

> [!NOTE]
> Une stratégie de groupe par défaut peut également être définie sur le fournisseur de sécurité SAML2. En définissant cette option, vous attribuez à tous les utilisateurs qui s’authentifient avec SAML les autorisations spécifiées dans la stratégie de groupe. La stratégie General Members est incluse dans BeyondTrust Remote Support/Privileged Remote Access avec des autorisations limitées, qui peuvent être utilisées pour tester l’authentification et attribuer les utilisateurs aux stratégies appropriées. Les utilisateurs n’apparaissent dans la liste SAML2 Users (/login > Users & Security) qu’après la première tentative d’authentification réussie. Pour plus d’informations sur les stratégies de groupe, utilisez le lien suivant : `https://www.beyondtrust.com/docs/remote-support/getting-started/admin/group-policies.htm`

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester votre configuration de l’authentification unique Azure AD avec les options suivantes. 

* Cliquez sur **Tester cette application** dans le portail Azure. Vous êtes alors redirigé vers l’URL de connexion BeyondTrust Remote Support, à partir de laquelle vous pouvez lancer le flux de connexion. 

* Accédez directement à l’URL de connexion BeyondTrust Remote Support pour lancer le flux de connexion.

* Vous pouvez utiliser Mes applications de Microsoft. Le fait de cliquer sur la vignette BeyondTrust Remote Support dans Mes applications vous redirige vers l’URL de connexion BeyondTrust Remote Support. Pour plus d’informations sur Mes applications, consultez [Présentation de Mes applications](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="next-steps"></a>Étapes suivantes

Après avoir configuré BeyondTrust Remote Support, vous pouvez appliquer des contrôles de session qui protègent en temps réel contre l’exfiltration et l’infiltration des données sensibles de votre organisation. Les contrôles de session sont étendus à partir de l’accès conditionnel. [Découvrir comment appliquer un contrôle de session avec Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-aad)
