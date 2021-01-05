---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à NetSuite | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et NetSuite.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 11/09/2020
ms.author: jeedes
ms.openlocfilehash: 9272114dd472717e59680b40c07d8435dea19682
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96181966"
---
# <a name="tutorial-integrate-azure-ad-single-sign-on-sso-with-netsuite"></a>Tutoriel : Intégrer l’authentification unique Azure Active Directory à NetSuite

Dans ce tutoriel, vous allez apprendre à intégrer Azure Active Directory (Azure AD) à NetSuite. Quand vous intégrez NetSuite à Azure AD, vous pouvez :

* Contrôler qui a accès à NetSuite dans Azure AD.
* Permettre à vos utilisateurs de se connecter automatiquement à NetSuite avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central, le portail Azure.

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement NetSuite pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test. 

NetSuite prend en charge :

* L’authentification unique lancée par le fournisseur d’identité.
* Le provisionnement d’utilisateurs juste-à-temps (JIT).
* [Le provisionnement automatique des utilisateurs](NetSuite-provisioning-tutorial.md).

> [!NOTE]
> L’identificateur de cette application étant une valeur de chaîne fixe, une seule instance peut être configurée dans un locataire.

## <a name="add-netsuite-from-the-gallery"></a>Ajouter NetSuite à partir de la galerie

Pour configurer l’intégration de NetSuite à Azure AD, ajoutez NetSuite à partir de la galerie à votre liste d’applications SaaS managées en effectuant les actions suivantes :

1. Connectez-vous au portail Azure avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le volet de gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **NetSuite** dans la zone de recherche.
1. Dans le volet de résultats, sélectionnez **NetSuite**, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-netsuite"></a>Configurer et tester l’authentification unique Azure AD pour NetSuite

Configurez et testez l’authentification unique Azure AD avec NetSuite à l’aide d’un utilisateur de test appelé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur NetSuite associé.

Pour configurer et tester l’authentification unique Azure AD avec NetSuite, suivez les indications des sections ci-après :

1. [Configurer l’authentification unique Azure AD](#configure-azure-ad-sso) pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * [Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user) pour tester l’authentification unique Azure AD avec l’utilisateur B.Simon.  
    * [Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user) pour permettre à l’utilisateur B.Simon d’utiliser l’authentification unique Azure AD.
1. [Configurer l’authentification unique NetSuite](#configure-netsuite-sso) pour configurer les paramètres de l’authentification unique côté application.
    * [Créer un utilisateur de test NetSuite](#create-the-netsuite-test-user) pour avoir un équivalent de l’utilisateur B.Simon dans NetSuite qui soit lié à la représentation Azure AD de l’utilisateur.
1. [Tester l’authentification unique (SSO)](#test-sso) pour vérifier que la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Pour activer l’authentification unique Azure AD dans le portail Azure, effectuez les étapes suivantes :

1. Dans le portail Azure, accédez à la page d’intégration de l’application **NetSuite**, recherchez la section **Gérer** et sélectionnez **Authentification unique**.
1. Dans le volet **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, sélectionnez l’icône **Modifier** (« crayon ») en regard de **Configuration SAML de base**.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base**, dans la zone de texte **URL de réponse**, tapez une URL dans l’un des formats suivants :

    ```https
    https://<Instance ID>.NetSuite.com/saml2/acs
    https://<Instance ID>.na1.NetSuite.com/saml2/acs
    https://<Instance ID>.na2.NetSuite.com/saml2/acs
    https://<Instance ID>.sandbox.NetSuite.com/saml2/acs
    https://<Instance ID>.na1.sandbox.NetSuite.com/saml2/acs
    https://<Instance ID>.na2.sandbox.NetSuite.com/saml2/acs
    ```

    * Vous obtiendrez la valeur **<`Instance ID`>** dans la section Configuration de NetSuite, qui est expliquée plus loin dans le tutoriel à l’étape 8 sous Configuration de NetSuite. Vous trouverez le domaine exact (par exemple, system.na0.netsuite.com dans le cas présent).

        ![Capture d’écran affichant la page Configuration SAML dans laquelle vous pouvez obtenir les valeurs.](./media/NetSuite-tutorial/domain-value.png)

        > [!NOTE]
        > Les valeurs des URL précédentes ne sont pas réelles. Mettez-les à jour avec l’URL de réponse réelle. Pour obtenir cette valeur, contactez l’[équipe du support technique NetSuite](http://www.netsuite.com/portal/services/support-services/suitesupport.shtml). Vous pouvez aussi vous reporter aux formats présentés dans la section **Configuration SAML de base** du portail Azure.

1. L’application NetSuite s’attend à recevoir les assertions SAML dans un certain format, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à la configuration des attributs de jetons SAML. La capture d’écran suivante montre la liste des attributs par défaut.

    ![image](common/default-attributes.png)

1. En plus de ce qui précède, l’application NetSuite s’attend à ce que quelques attributs supplémentaires (présentés ci-dessous) soient repassés dans la réponse SAML. Ces attributs sont également préremplis, mais vous pouvez les examiner pour voir s’ils répondent à vos besoins.

    | Nom | Attribut source |
    | ---------------| --------------- |
    | account  | `account id` |

    > [!NOTE]
    > La valeur de l’attribut account n’est pas réelle. Vous mettrez à jour cette valeur, comme expliqué plus loin dans ce tutoriel.

1. Dans la page Configurer l’authentification unique avec SAML, dans la section Certificat de signature SAML, recherchez XML de métadonnées de fédération et sélectionnez Télécharger pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien de téléchargement de certificat](common/metadataxml.png)

1. Dans la section **Configurer NetSuite**, copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé B. Simon sur le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** > **Utilisateurs** > **Tous les utilisateurs**.

1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

1. Dans le volet de propriétés **Utilisateur**, effectuez les étapes suivantes :

   a. Dans la zone **Nom**, entrez **B.Simon**.  
   b. Dans la zone **Nom d’utilisateur**, entrez username@companydomain.extension (par exemple B.Simon@contoso.com).  
   c. Cochez la case **Afficher le mot de passe**, puis notez la valeur affichée dans le champ **Mot de passe**.  
   d. Sélectionnez **Create** (Créer).

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser l’utilisateur B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à NetSuite.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **NetSuite**.
1. Dans le volet de vue d’ensemble, recherchez la section **Gérer**, puis sélectionnez le lien **Utilisateurs et groupes**.
1. Sélectionnez **Ajouter un utilisateur** puis, dans le volet **Ajouter une attribution**, sélectionnez **Utilisateurs et groupes**.
1. Dans la boîte de dialogue **Utilisateurs et groupes**, dans la liste déroulante **Utilisateurs**, sélectionnez **B. Simon**, puis sélectionnez le bouton **Sélectionner** en bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, effectuez les étapes suivantes :

   a. Dans le volet **Sélectionner un rôle**, dans la liste déroulante, sélectionnez le rôle approprié pour l’utilisateur.  
   b. En bas de l’écran, sélectionnez le bouton **Sélectionner**.
1. Dans le volet **Ajouter une attribution**, sélectionnez le bouton **Attribuer**.

## <a name="configure-netsuite-sso"></a>Configurer l’authentification unique NetSuite

1. Ouvrez un nouvel onglet dans votre navigateur et connectez-vous à votre site d’entreprise NetSuite en tant qu’administrateur.

2. Dans la barre de navigation supérieure, sélectionnez **Setup** (Configuration), puis **Company** > **Enable Features** (Société>Activer des fonctionnalités).

    ![Capture d’écran affichant l’élément Enable Features sélectionné à partir de Company.](./media/NetSuite-tutorial/ns-setupsaml.png)

3. Dans la barre d’outils située au milieu de la page, sélectionnez sur **SuiteCloud**.

    ![Capture d’écran montrant la sélection de SuiteCloud.](./media/NetSuite-tutorial/ns-suitecloud.png)

4. Dans la section **Manage Authentication** (Gérer l’authentification), cochez la case **SAML Single Sign-on** (Authentification unique SAML) pour activer l’option du même nom dans NetSuite.

    ![Capture d’écran montrant la section Manage Authentication dans laquelle vous pouvez sélectionner SAML Single Sign-on.](./media/NetSuite-tutorial/ns-ticksaml.png)

5. Dans la barre de navigation supérieure, sélectionnez **Setup**.

    ![Capture d’écran montrant l’élément Setup sélectionné dans la barre de navigation de NETSUITE.](./media/NetSuite-tutorial/ns-setup.png)

6. Dans la liste **Setup Tasks** (Tâches de configuration), sélectionnez **Integration**.

    ![Capture d’écran montrant l’élément Integration sélectionné dans SETUP TASKS.](./media/NetSuite-tutorial/ns-integration.png)

7. Sous **Manage Authentication**, sélectionnez **SAML Single Sign-on**.

    ![Capture d’écran montrant l’option SAML Single Sign-on sélectionnée à partir de l’élément Integration dans SETUP TASKS.](./media/NetSuite-tutorial/ns-saml.png)

8. Dans le volet **SAML Setup** (Configuration SAML), sous **NetSuite Configuration** (Configuration de NetSuite), effectuez les étapes suivantes :

    ![Capture d’écran montrant la section SAML Setup dans laquelle vous pouvez indiquer les valeurs décrites.](./media/NetSuite-tutorial/ns-saml-setup.png)
  
    a. Cochez la case **Primary Authentication Method** (Méthode d’authentification principale).

    b. Sous **SAMLV2 Identity Provider Metadata** (Métadonnées du fournisseur d’identité SAMLV2), sélectionnez **Upload IDP Metadata File** (Charger le fichier de métadonnées IDP), puis **Browse** (Parcourir) pour charger le fichier de métadonnées que vous avez téléchargé à partir du portail Azure.

    c. Sélectionnez **Envoyer**.

9. Dans la barre de navigation supérieure de NetSuite, sélectionnez **Setup**, puis **Company** > **Company Information** (Informations sur la société).

    ![Capture d’écran montrant l’option Company Information sélectionnée à partir de Company.](./media/NetSuite-tutorial/ns-com.png)

    ![Capture d’écran montrant le volet dans lequel vous pouvez entrer les valeurs décrites.](./media/NetSuite-tutorial/ns-account-id.png)

    b. Dans le volet **Company Information**, dans la colonne de droite, copiez la valeur du champ **Account ID** (ID de compte).

    c. Collez la valeur **Account ID** que vous avez copiée à partir du compte NetSuite dans la zone **Valeur de l’attribut** dans Azure AD.

    ![Capture d’écran montrant l’ajout de la valeur d’ID de compte](./media/netsuite-tutorial/attribute-value.png)

10. Avant que les utilisateurs puissent utiliser l’authentification unique dans NetSuite, vous devez d’abord leur affecter les autorisations appropriées dans NetSuite. Pour affecter ces autorisations, effectuez les étapes suivantes :

    a. Dans la barre de navigation supérieure, sélectionnez **Setup**.

    ![Capture d’écran montrant l’élément Setup sélectionné dans la barre de navigation de NETSUITE.](./media/NetSuite-tutorial/ns-setup.png)

    b. Dans le volet gauche, sélectionnez **Users/Roles** (Utilisateurs/rôles), puis **Manage Roles** (Gérer les rôles).

    ![Capture d’écran montrant le volet Manage Roles dans lequel vous pouvez sélectionner New Role.](./media/NetSuite-tutorial/ns-manage-roles.png)

    c. Sélectionnez **New Role** (Nouveau rôle).

    d. Entrez un nom (**Name**) pour le nouveau rôle.

    ![Capture d’écran montrant l’élément Setup Manager dans lequel vous pouvez indiquer le nom du rôle.](./media/NetSuite-tutorial/ns-new-role.png)

    e. Sélectionnez **Enregistrer**.

    f. Dans la barre de navigation supérieure, sélectionnez **Permissions** (Autorisations). Ensuite, sélectionnez **Setup**.

    ![Capture d’écran affichant l’onglet Setup dans lequel vous pouvez indiquer les valeurs décrites.](./media/NetSuite-tutorial/ns-sso.png)

    g. Sélectionnez **SAML Single Sign-on**, puis **Add** (Ajouter).

    h. Sélectionnez **Enregistrer**.

    i. Dans la barre de navigation supérieure, sélectionnez **Setup**, puis **Setup Manager** (Gestionnaire de configuration).

    ![Capture d’écran montrant l’élément Setup sélectionné dans la barre de navigation de NETSUITE.](./media/NetSuite-tutorial/ns-setup.png)

    j. Dans le volet gauche, sélectionnez **Users/Roles**, puis **Manage Users** (Gérer les utilisateurs).

    ![Capture d’écran montrant le volet Manage Users dans lequel vous pouvez sélectionner Suite Demo Team.](./media/NetSuite-tutorial/ns-manage-users.png)

    k. Sélectionnez un utilisateur de test, sélectionnez **Edit** (Modifier), puis sélectionnez l’onglet **Access** (Accès).

    ![Capture d’écran montrant le volet Manage Users dans lequel vous pouvez sélectionner Edit.](./media/NetSuite-tutorial/ns-edit-user.png)

    l. Dans le volet **Roles**, attribuez le rôle que vous avez créé.

    ![Capture d’écran montrant l’option Administrator sélectionnée dans la section Employee.](./media/NetSuite-tutorial/ns-add-role.png)

    m. Sélectionnez **Enregistrer**.

### <a name="create-the-netsuite-test-user"></a>Créer l’tilisateur de test NetSuite

Dans cette section, un utilisateur appelé B.Simon est créé dans NetSuite. NetSuite prend en charge l’attribution d’utilisateurs juste-à-temps, option activée par défaut. Vous n’avez aucune opération à effectuer dans cette section. S’il n’existe pas encore d’utilisateur dans NetSuite, il en est créé un après l’authentification.

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester votre configuration de l’authentification unique Azure AD avec les options suivantes.

- Cliquez sur Tester cette application dans le portail Azure et vous devez être connecté automatiquement à l’instance de NetSuite pour laquelle vous avez configuré l’authentification unique

- Vous pouvez utiliser Mes applications de Microsoft. Quand vous cliquez sur la vignette NetSuite dans Mes applications, vous devez être connecté automatiquement à l’instance de NetSuite pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur Mes applications, consultez [Présentation de Mes applications](../user-help/my-apps-portal-end-user-access.md).


## <a name="next-steps"></a>Étapes suivantes

Après avoir configuré NetSuite, vous pouvez appliquer des contrôles de session qui protègent en temps réel contre l’exfiltration et l’infiltration des données sensibles de votre organisation. Les contrôles de session sont étendus à partir de l’accès conditionnel. [Découvrir comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad)