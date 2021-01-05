---
title: "Tutoriel : Intégration d'Azure Active Directory à SAP Cloud Platform Identity Authentication | Microsoft Docs"
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et SAP Cloud Platform Identity Authentication.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/28/2020
ms.author: jeedes
ms.openlocfilehash: fc5e0270c4533eaae1310dc84b616d675a79c8ea
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92895084"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-sap-cloud-platform-identity-authentication"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à SAP Cloud Platform Identity Authentication

Dans ce tutoriel, vous allez apprendre à intégrer SAP Cloud Platform Identity Authentication à Azure Active Directory (Azure AD). Quand vous intégrez SAP Cloud Platform Identity Authentication à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à SAP Cloud Platform Identity Authentication.
* Permettre à vos utilisateurs de se connecter automatiquement à SAP Cloud Platform Identity Authentication avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement SAP Cloud Platform Identity Authentication pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* SAP Cloud Platform Identity Authentication prend en charge l’authentification unique lancée par le **fournisseur de services** et le **fournisseur d’identité**
* Après avoir configuré SAP Cloud Platform Identity Authentication, vous pouvez appliquer des contrôles de session qui protègent l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Les contrôles de session sont étendus à partir de l’accès conditionnel. [Découvrir comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad)

Avant de rentrer dans les détails techniques, il est essentiel de comprendre les concepts que vous allez voir. SAP Cloud Platform Identity Authentication et les services de fédération Active Directory (AD FS) vous permettent d’implémenter l’authentification unique entre les applications ou les services protégés par Azure AD (comme IdP) et les applications et services SAP protégés par SAP Cloud Platform Identity Authentication.

Actuellement, SAP Cloud Platform Identity Authentication remplit la fonction de fournisseur d’identité proxy pour les applications SAP. Azure Active Directory joue en retour le rôle de fournisseur d’identité principal dans cette configuration. 

Le schéma suivant illustre cette relation :

![Création d’un utilisateur de test Azure AD](./media/sap-hana-cloud-platform-identity-authentication-tutorial/architecture-01.png)

Avec cette configuration, votre client SAP Cloud Platform Identity Authentication est configuré en tant qu’application approuvée dans Azure Active Directory.

Toutes les applications et tous les services SAP que vous souhaitez protéger de cette façon sont ensuite configurés dans la console de gestion SAP Cloud Platform Identity Authentication.

Ainsi, l’autorisation d’accorder l’accès aux services et applications SAP doit avoir lieu dans SAP Cloud Platform Identity Authentication (par opposition à Azure Active Directory).

Quand vous configurez SAP Cloud Platform Identity Authentication en tant qu’application par le biais de la Place de Marché Azure Active Directory, vous n’avez pas besoin de configurer des revendications ou des assertions SAML.

> [!NOTE]
> Actuellement, seule l’authentification unique web a été testée par les deux parties. Les flux nécessaires à la communication application-API ou API-API devraient fonctionner mais n’ont pas encore été testés. Ils seront testés durant les activités à venir.

## <a name="adding-sap-cloud-platform-identity-authentication-from-the-gallery"></a>Ajout de SAP Cloud Platform Identity Authentication à partir de la galerie

Pour configurer l’intégration de SAP Cloud Platform Identity Authentication dans Azure AD, vous devez ajouter SAP Cloud Platform Identity Authentication à votre liste d’applications SaaS managées à partir de la galerie.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **SAP Cloud Platform Identity Authentication** dans la zone de recherche.
1. Sélectionnez **SAP Cloud Platform Identity Authentication** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-single-sign-on-for-sap-cloud-platform-identity-authentication"></a>Configurer et tester l’authentification unique Azure AD pour SAP Cloud Platform Identity Authentication

Configurez et testez l’authentification unique Azure AD avec SAP Cloud Platform Identity Authentication à l’aide d’un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur SAP Cloud Platform Identity Authentication associé.

Pour configurer et tester l’authentification unique Azure AD avec SAP Cloud Platform Identity Authentication, vous devez suivre les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique SAP Cloud Platform Identity Authentication](#configure-sap-cloud-platform-identity-authentication-sso)** pour configurer les paramètres de l’authentification unique côté application.
    * **[Créer un utilisateur de test SAP Cloud Platform Identity Authentication](#create-sap-cloud-platform-identity-authentication-test-user)** pour avoir un équivalent de B.Simon dans SAP Cloud Platform Identity Authentication lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **SAP Cloud Platform Identity Authentication** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode lancé par le **fournisseur d’identité** , effectuez les étapes suivantes :

    ![Capture d’écran montrant la section « Configuration SAML de base » avec les zones de texte « Identificateur » et « URL de réponse » mises en évidence, et le bouton « Enregistrer » sélectionné.](common/idp-intiated.png)

    a. Dans la zone de texte **Identificateur** , tapez une URL au format suivant : `<IAS-tenant-id>.accounts.ondemand.com`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://<IAS-tenant-id>.accounts.ondemand.com/saml2/idp/acs/<IAS-tenant-id>.accounts.ondemand.com`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur et l’URL de réponse réels. Pour obtenir ces valeurs, contactez [l’équipe du support technique de SAP Cloud Platform Identity Authentication](https://cloudplatform.sap.com/capabilities/security/trustcenter.html). Si vous ne comprenez pas la valeur d’identificateur, lisez la documentation SAP Cloud Platform Identity Authentication relative à la [configuration du locataire SAML 2.0](https://help.hana.ondemand.com/cloud_identity/frameset.htm?e81a19b0067f4646982d7200a8dab3ca.html).

5. Cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes si vous souhaitez configurer l’application en mode lancé par le **fournisseur de services**  :

    ![Informations d’authentification unique pour Domaine et URL SAP Cloud Platform Identity Authentication](common/metadata-upload-additional-signon.png)

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `{YOUR BUSINESS APPLICATION URL}`

    > [!NOTE]
    > Cette valeur n’est pas la valeur réelle. Mettez-la à jour avec l’URL de connexion réelle. Utilisez l’URL de connexion spécifique de votre application métier. En cas de doute, contactez [l’équipe du support technique de SAP Cloud Platform Identity Authentication](https://cloudplatform.sap.com/capabilities/security/trustcenter.html).

1. L’application SAP Cloud Platform Identity Authentication attend les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à votre configuration d’attributs de jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut.

    ![image](common/default-attributes.png)

1. En plus de ce qui précède, l’application SAP Cloud Platform Identity Authentication s’attend à ce que quelques attributs supplémentaires (présentés ci-dessus) soient repassés dans la réponse SAML. Ces attributs sont également préremplis, mais vous pouvez les examiner pour voir s’ils répondent à vos besoins.

    | Nom | Attribut source|
    | ---------------| --------------- |
    | firstName | user.givenname |

8. Sur la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le fichier **XML des métadonnées** en fonction des options définies, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

9. Dans la section **Configurer SAP Cloud Platform Identity Authentication** , copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à SAP Cloud Platform Identity Authentication.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **SAP Cloud Platform Identity Authentication** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-sap-cloud-platform-identity-authentication-sso"></a>Configurer l’authentification unique SAP Cloud Platform Identity Authentication

1. Pour configurer l’authentification unique sur votre application, accédez à la console d’administration de SAP Cloud Platform Identity Authentication. L’URL suit le modèle suivant : `https://<tenant-id>.accounts.ondemand.com/admin`. Lisez ensuite la documentation relative à SAP Cloud Platform Identity Authentication à l’adresse [Integration with Microsoft Azure AD (Intégration à Microsoft Azure AD)](https://developers.sap.com/tutorials/cp-ias-azure-ad.html).

2. Dans le portail Azure, sélectionnez le bouton **Enregistrer** .

3. Effectuez les étapes suivantes uniquement si vous souhaitez ajouter et activer l’authentification unique pour une autre application SAP. Répétez les étapes décrites dans la section **Ajout de SAP Cloud Platform Identity Authentication à partir de la galerie** .

4. Dans le portail Azure, dans la page d’intégration des applications **SAP Cloud Platform Identity Authentication** , sélectionnez **Authentification liée** .

    ![Configurer l’authentification liée](./media/sap-hana-cloud-platform-identity-authentication-tutorial/linked_sign_on.png)

5. Enregistrez la configuration.

> [!NOTE]
> La nouvelle application utilise la configuration de l’authentification unique de l’application SAP précédente. Veillez à utiliser les mêmes fournisseurs d’identité d’entreprise dans la console d’administration de SAP Cloud Platform Identity Authentication.

### <a name="create-sap-cloud-platform-identity-authentication-test-user"></a>Créer un utilisateur de test SAP Cloud Platform Identity Authentication

Vous n’avez pas besoin de créer un utilisateur dans SAP Cloud Platform Identity Authentication. Les utilisateurs qui se trouvent dans le magasin d’utilisateurs Azure AD peuvent utiliser la fonctionnalité d’authentification unique (SSO).

SAP Cloud Platform Identity Authentication prend en charge l’option de fédération des identités. Cette option permet à l’application de vérifier si les utilisateurs authentifiés par le fournisseur d’identité d’entreprise sont présents dans le magasin d’utilisateurs de SAP Cloud Platform Identity Authentication.

L’option de fédération des identités est désactivée par défaut. Si la fédération des identités est activée, seuls les utilisateurs importés dans SAP Cloud Platform Identity Authentication peuvent accéder à l’application.

Pour savoir comment activer ou désactiver la fédération des identités avec SAP Cloud Platform Identity Authentication, consultez « Activer la fédération des identités avec SAP Cloud Platform Identity Authentication » dans [Configure Identity Federation with the User Store of SAP Cloud Platform Identity Authentication (Configurer la fédération des identités avec le magasin d’utilisateurs de SAP Cloud Platform Identity Authentication)](https://help.sap.com/viewer/6d6d63354d1242d185ab4830fc04feb1/Cloud/c029bbbaefbf4350af15115396ba14e2.html).

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette SAP Cloud Platform Identity Authentication dans le volet d’accès, vous devez vous connecter automatiquement à l’application SAP Cloud Platform Identity Authentication pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer SAP Cloud Platform Identity Authentication avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)

- [Guide pratique pour protéger SAP Cloud Platform Identity Authentication avec une visibilité et des contrôles avancés](/cloud-app-security/proxy-intro-aad)