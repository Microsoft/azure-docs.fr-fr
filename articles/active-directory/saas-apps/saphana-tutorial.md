---
title: 'Didacticiel : intégration d’Azure Active Directory à SAP HANA | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et SAP HANA.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/27/2018
ms.author: jeedes
ms.openlocfilehash: 748566cf5b25157fcb8a2938a3f4bd307c930874
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95993218"
---
# <a name="tutorial-azure-active-directory-integration-with-sap-hana"></a>Didacticiel : Intégration d’Azure Active Directory à SAP HANA

Dans ce didacticiel, vous allez apprendre à intégrer SAP HANA à Azure Active Directory (Azure AD).
L’intégration de SAP HANA à Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à SAP HANA.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à SAP HANA (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD avec SAP HANA, vous avez besoin des éléments suivants :

- Un abonnement Azure AD
- Un abonnement SAP HANA pour lequel l’authentification unique est activée
- Une instance HANA en cours d’exécution sur une IaaS publique, locale, sur une machine virtuelle Azure ou sur une grande instance SAP dans Azure
- L’interface web d’administration XSA ainsi que HANA Studio, installés sur l’instance HANA

> [!NOTE]
> Nous déconseillons l’utilisation d’un environnement de production de SAP HANA pour tester les étapes de ce didacticiel. Testez d’abord l’intégration dans l’environnement de développement ou l’environnement de préproduction de l’application, puis passez à l’environnement de production.

Pour tester la procédure de ce didacticiel, suivez les recommandations ci-dessous :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Un abonnement SAP HANA pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* SAP HANA prend en charge l’authentification unique initiée par **IDP**
* SAP HANA prend en charge l’attribution d’utilisateurs **juste-à-temps**

## <a name="adding-sap-hana-from-the-gallery"></a>Ajout de SAP HANA à partir de la galerie

Pour configurer l’intégration de SAP HANA à Azure AD, vous devez ajouter SAP HANA à partir de la galerie à votre liste d’applications SaaS gérées.

**Pour ajouter SAP HANA à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise**, puis sélectionnez l’option **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **SAP HANA**, sélectionnez **SAP HANA** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![SAP HANA dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec SAP HANA, avec un utilisateur de test appelé **Britta Simon**.
Pour que l’authentification unique fonctionne, une relation entre l’utilisateur Azure AD et l’utilisateur SAP HANA associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec SAP HANA, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique SAP HANA](#configure-sap-hana-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test SAP HANA](#create-sap-hana-test-user)** pour avoir un équivalent de Britta Simon dans SAP HANA lié à la représentation Azure AD de l’utilisateur.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec SAP HANA, procédez comme suit :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **SAP HANA**, sélectionnez **Authentification unique**.

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique**, sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**.

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Sur la page **Configurer l’authentification unique avec SAML**, effectuez les étapes suivantes :

    ![Informations d’authentification unique dans Domaine et URL SAP HANA](common/idp-intiated.png)

    a. Dans la zone de texte **Identificateur**, tapez : `HA100`

    b. Dans la zone de texte **URL de réponse**, tapez une URL au format suivant : `https://<Customer-SAP-instance-url>/sap/hana/xs/saml/login.xscfunc`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur et l’URL de réponse réels. Pour obtenir ces valeurs, contactez l’[équipe de support technique SAP HANA](https://cloudplatform.sap.com/contact.html). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

5. L’application SAP HANA attend les assertions SAML dans un format spécifique. Configurez les revendications suivantes pour cette application. Vous pouvez gérer les valeurs de ces attributs à partir de la section **Attributs utilisateur** sur la page d’intégration des applications. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur le bouton **Modifier** pour ouvrir la boîte de dialogue **Attributs utilisateur**.

    ![Capture d’écran montrant la section « User Attributes » avec l’icône « Edit » sélectionnée.](common/edit-attribute.png)

6. Dans la section **Attributs utilisateur** de la boîte de dialogue **Attributs et revendications de l’utilisateur**, procédez comme suit :
 
    a. Cliquez sur l’**icône Modifier** pour ouvrir la boîte de dialogue **Gérer les revendications des utilisateurs**.

    ![Capture d’écran montrant la boîte de dialogue « Attributs utilisateur et revendications » avec le bouton « Modifier » sélectionné.](./media/saphana-tutorial/tutorial_usermail.png)

    ![image](./media/saphana-tutorial/tutorial_usermailedit.png)

    b. Dans la liste **Transformation**, sélectionnez **ExtractMailPrefix()** .

    c. Dans la liste **Paramètre 1**, sélectionnez **user.mail**.

    d. Cliquez sur **Enregistrer**.

7. Sur la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, cliquez sur **Télécharger** pour télécharger le fichier **XML de métadonnées de fédération** en fonction des options définies selon vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

### <a name="configure-sap-hana-single-sign-on"></a>Configurer l’authentification unique de SAP HANA

1. Pour configurer l’authentification unique côté SAP HANA, connectez-vous à votre **console web HANA XSA** en accédant au point de terminaison HTTPS correspondant.

    > [!NOTE]
    > Dans la configuration par défaut, l’URL redirige la requête vers un écran d’ouverture de session, où les informations d’identification d’un utilisateur de la base de données SAP HANA agréé sont nécessaires. L’utilisateur qui se connecte doit avoir les autorisations nécessaires pour effectuer des tâches d’administration SAML.

2. Dans l’Interface web XSA, accédez à **SAML Identity Provider** (Fournisseur d’identité SAML). Ensuite, sélectionnez le bouton **+** en bas de l’écran pour afficher le volet **Add Identity Provider Info** (Ajouter les informations du fournisseur d’identité). Ensuite, effectuez les étapes suivantes :

    ![Ajouter un fournisseur d’identité](./media/saphana-tutorial/sap1.png)

    a. Dans le volet **d’ajout des informations sur le fournisseur d’identité**, collez le contenu du fichier XML de métadonnées que vous avez téléchargé à partir du portail Azure dans la zone **Métadonnées**.

    ![Capture d’écran montrant le volet « Add Identity Provider Info » avec les zones « Metadata » et « Name » mises en évidence.](./media/saphana-tutorial/sap2.png)

    b. Si le contenu du document XML est valide, le processus d’analyse extrait les informations requises pour les champs **Subject, Entity ID et Issuer** (Sujet, ID d’entité et Émetteur) dans la zone d’écran **General data** (Données générales). Il extrait également les informations nécessaires pour les champs d’URL dans la zone d’écran **Destination**, tels que les champs **Base URL and SingleSignOn URL (*)** (URL de base et URL d’authentification unique).

    ![Paramètres d’ajout d’un fournisseur d’identité](./media/saphana-tutorial/sap3.png)

    c. Dans la zone **Nom** de la zone d’écran **General Data** (Données générales), entrez un nom pour le nouveau fournisseur d’identité d’authentification unique SAML.

    > [!NOTE]
    > Le nom du fournisseur d’identité SAML est obligatoire et doit être unique. Il apparaît dans la liste des fournisseurs d’identité SAML disponibles qui s’affiche quand vous sélectionnez SAML comme méthode d’authentification pour les applications SAP HANA XS à utiliser. Par exemple, vous pouvez faire cela dans la zone d’écran **Authentication** (Authentification) de l’outil d’administration d’artefact XS.

3. Sélectionnez **Save** (Enregistrer) pour enregistrer les détails du fournisseur d’identité SAML et ajouter le nouveau fournisseur d’identité SAML à la liste des fournisseurs d’identité SAML connus.

    ![Bouton Enregistrer](./media/saphana-tutorial/sap4.png)

4. Dans Studio HANA, dans les propriétés système de l’onglet **Configuration**, filtrez les paramètres en fonction de **saml**. Ensuite, ajustez **assertion_timeout** en remplaçant **10 s** par **120 s**.

    ![Paramètre assertion_timeout](./media/saphana-tutorial/sap7.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD 

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory**, sélectionnez **Utilisateurs**, puis sélectionnez **Tous les utilisateurs**.

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom**, entrez **BrittaSimon**.
  
    b. Dans le champ **Nom d’utilisateur**, tapez **brittasimon\@domainedevotreentreprise.extension**.  
    Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe**, puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à SAP HANA.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, **Toutes les applications**, puis sélectionnez **SAP HANA**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, tapez et sélectionnez **SAP HANA**.

    ![Lien SAP HANA dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes**.

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur**, puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

### <a name="create-sap-hana-test-user"></a>Créer un utilisateur de test SAP HANA

Pour permettre aux utilisateurs Azure AD de se connecter à SAP HANA, vous devez les provisionner dans SAP HANA.
SAP HANA prend en charge le provisionnement **juste-à-temps**, qui est activé par défaut.

Si vous avez besoin de créer un utilisateur manuellement, effectuez les étapes suivantes :

>[!NOTE]
>Vous pouvez changer l’authentification externe dont se sert l’utilisateur. Il peut s’authentifier auprès d’un système externe, tel que Kerberos. Pour plus d’informations sur les identités externes, contactez votre [administrateur de domaine](https://cloudplatform.sap.com/contact.html).

1. Ouvrez [SAP HANA Studio](https://help.sap.com/viewer/a2a49126a5c546a9864aae22c05c3d0e/2.0.01/en-us) en tant qu’administrateur, puis activez l’utilisateur de base de données pour l’authentification unique SAML.

    ![Créer un utilisateur](./media/saphana-tutorial/sap5.png)

2. Cochez la case invisible à gauche de **SAML**, puis sélectionnez le lien **Configurer**.

3. Sélectionnez **Ajouter** pour ajouter le fournisseur d’identité SAML.  Sélectionnez le fournisseur d’identité SAML approprié, puis sélectionnez **OK**.

4. Ajouter **l’identité externe** (dans ce cas, BrittaSimon) ou choisissez **Quelconque**. Sélectionnez ensuite **OK**.

   > [!Note]
   > Si la case **Quelconque** n’est pas cochée, le nom d’utilisateur dans HANA doit correspondre exactement au nom de l’utilisateur dans le nom d’utilisateur principal avant le suffixe de domaine. (Par exemple, BrittaSimon@contoso.com deviendrait BrittaSimon dans HANA.)

5. À des fins de test, affectez tous les rôles **XS** à l’utilisateur.

    ![Attribution de rôles](./media/saphana-tutorial/sap6.png)

    > [!TIP]
    > Vous devez donner les autorisations qui sont appropriées pour vos cas d’usage uniquement.

6. Enregistrez l’utilisateur.

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette SAP HANA dans le panneau d’accès doit vous connecter automatiquement à l’application SAP HANA pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)