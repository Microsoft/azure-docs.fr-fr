---
title: 'Tutoriel : Intégration d’Azure Active Directory à SAP Analytics Cloud | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et SAP Analytics Cloud.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 07/17/2019
ms.author: jeedes
ms.openlocfilehash: 84b0a761b52baca4ae1ab81b3c67f18f6b14db92
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97608831"
---
# <a name="tutorial-integrate-sap-analytics-cloud-with-azure-active-directory"></a>Tutoriel : Intégrer SAP Analytics Cloud à Azure Active Directory

Dans ce tutoriel, vous découvrez comment intégrer SAP Analytics Cloud à Azure Active Directory (Azure AD). Quand vous intégrez SAP Analytics Cloud à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à SAP Analytics Cloud.
* Permettre à vos utilisateurs de se connecter automatiquement à SAP Analytics Cloud avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement SAP Analytics Cloud pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* SAP Analytics Cloud prend en charge l’authentification unique lancée par le **fournisseur de services**

## <a name="adding-sap-analytics-cloud-from-the-gallery"></a>Ajout de SAP Analytics Cloud à partir de la galerie

Pour configurer l’intégration de SAP Analytics Cloud à Azure AD, vous devez ajouter SAP Analytics Cloud depuis la galerie à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **SAP Analytics Cloud** dans la zone de recherche.
1. Sélectionnez **SAP Analytics Cloud** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Configurez et testez l’authentification unique Azure AD avec SAP Analytics Cloud avec un utilisateur de test appelé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur SAP Analytics Cloud associé.

Pour configurer et tester l’authentification unique (SSO) Azure AD avec SAP Analytics Cloud, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique SAP Analytics Cloud](#configure-sap-analytics-cloud-sso)**  : pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test SAP Analytics Cloud](#create-sap-analytics-cloud-test-user)**  : pour avoir un équivalent de B.Simon dans SAP Analytics Cloud lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **SAP Analytics Cloud**, recherchez la section **Gérer** et sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de modification/stylet pour **Configuration SAML de base** afin de modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base**, entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **URL de connexion**, entrez une URL au format suivant :

    - `https://<sub-domain>.sapanalytics.cloud/`
    - `https://<sub-domain>.sapbusinessobjects.cloud/`

    b. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL au format suivant :

    - `<sub-domain>.sapbusinessobjects.cloud`
    - `<sub-domain>.sapanalytics.cloud`

    > [!NOTE] 
    > Les valeurs de ces URL servent uniquement à des fins de démonstration. Mettez à jour les valeurs avec les URL de connexion et de l’identificateur réels. Pour obtenir l’URL de connexion, contactez [l’équipe du support client SAP Analytics Cloud](https://help.sap.com/viewer/product/SAP_BusinessObjects_Cloud/release/). Vous pouvez obtenir l’URL de l’identificateur en téléchargeant les métadonnées de SAP Analytics Cloud depuis la console d’administration. Une explication sera fournie plus loin dans ce didacticiel.

4. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, recherchez **XML de métadonnées de fédération** et sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

6. Dans la section **Configurer SAP Analytics Cloud**, copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

### <a name="configure-sap-analytics-cloud-sso"></a>Configurer l’authentification unique SAP Analytics Cloud

1. Dans une autre fenêtre du navigateur web, ouvrez une session sur votre site d’entreprise SAP Analytics Cloud en tant qu’administrateur.

2. Sélectionnez **Menu** > **Système** > **Administration**.
    
    ![Sélectionner Menu, puis Système et Administration](./media/sapboc-tutorial/config1.png)

3. Sous l’onglet **Sécurité**, sélectionnez l’icône (stylet) **Modifier**.
    
    ![Sous l’onglet Sécurité, sélectionner l’icône Modifier](./media/sapboc-tutorial/config2.png)  

4. Sélectionnez **Méthode d’authentification unique SAML (SSO)** comme **Méthode d’authentification**.

    ![Sélectionner Authentification unique SAML comme méthode d’authentification](./media/sapboc-tutorial/config3.png)  

5. Pour télécharger les métadonnées du fournisseur de services (étape 1), sélectionnez **Télécharger**. Dans le fichier de métadonnées, recherchez et copiez la valeur **entityID**. Dans le portail Azure, dans la boîte de dialogue **Configuration SAML de base**, collez la valeur dans la zone **Identificateur**.

    ![Copier et coller la valeur entityID](./media/sapboc-tutorial/config4.png)  

6. Pour télécharger les métadonnées du fournisseur de services (étape 2) dans le fichier que vous avez téléchargé depuis le portail Azure, sous **Charger les métadonnées du fournisseur d’identité**, sélectionnez **Charger**.  

    ![Sous Charger les métadonnées du fournisseur d’identité, sélectionnez Charger](./media/sapboc-tutorial/config5.png)

7. Dans la liste **Attribut utilisateur**, sélectionnez l’attribut utilisateur (étape 3) que vous souhaitez utiliser pour votre mise en œuvre. Cet attribut utilisateur est mappé au fournisseur d’identité. Pour entrer un attribut personnalisé sur la page de l’utilisateur, utilisez l’option **Mappage SAML personnalisé**. Ou bien, vous pouvez sélectionner **E-mail** ou **ID UTILISATEUR** en tant qu’attribut utilisateur. Dans notre exemple, nous avons sélectionné **E-mail**, car nous avons mappé la revendication de l’identificateur d’utilisateur avec l’attribut **userprincipalname** dans la section **Attributs utilisateur et revendications** dans le portail Azure. Ceci fournit un e-mail de l’utilisateur unique, qui est envoyé à l’application SAP Analytics Cloud dans chaque réponse SAML correcte.

    ![Sélectionner un attribut utilisateur](./media/sapboc-tutorial/config6.png)

8. Pour vérifier le compte avec le fournisseur d’identité (étape 4), dans la boîte **Informations d’identification de connexion (e-mail)** , entrez l’adresse e-mail de l’utilisateur. Ensuite, sélectionnez **Vérifier le compte**. Le système ajoute les informations d’identification de connexion au compte utilisateur.

    ![Entrer l’e-mail et sélectionner Vérifier le compte](./media/sapboc-tutorial/config7.png)

9. Sélectionnez l’icône **Enregistrer**.

    ![Icône Enregistrer](./media/sapboc-tutorial/save.png)

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à SAP Analytics Cloud.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **SAP Analytics Cloud**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

### <a name="create-sap-analytics-cloud-test-user"></a>Créer un utilisateur de test SAP Analytics Cloud

Les utilisateurs Azure AD doivent être provisionnés dans SAP Analytics Cloud avant de pouvoir s’y connecter. Dans SAP Analytics Cloud, le provisionnement est une tâche manuelle.

Pour approvisionner un compte d’utilisateur :

1. Connectez-vous en tant qu’administrateur à votre site d’entreprise SAP Analytics Cloud.

2. Sélectionnez **Menu** > **Sécurité** > **Utilisateurs**.

    ![Ajouter un employé](./media/sapboc-tutorial/user1.png)

3. Sur la page **Utilisateurs**, pour ajouter de nouvelles informations de l’utilisateur, sélectionnez **+** . 

    ![Page Ajouter des utilisateurs](./media/sapboc-tutorial/user4.png)

    Effectuez ensuite les tâches suivantes :

    1. Dans la zone **ID UTILISATEUR**, entrez l’ID utilisateur de l’utilisateur, par exemple **B**.

    1. Dans la zone **PRÉNOM**, entrez le prénom de l’utilisateur, par exemple **B**.

    1. Dans la boîte **NOM**, entrez le prénom de l’utilisateur, par exemple **Simon**.

    1. Dans la zone **NOM D’AFFICHAGE**, entrez le nom complet de l’utilisateur, par exemple **B.Simon**.

    1. Dans la boîte **E-MAIL**, entrez l’adresse e-mail de l’utilisateur, par exemple `b.simon@contoso.com`.

    1. Dans la page **Sélectionner des rôles**, sélectionnez le rôle approprié pour l’utilisateur, puis **OK**.

        ![Sélectionner un rôle](./media/sapboc-tutorial/user3.png)

    1. Sélectionnez l’icône **Enregistrer**.

### <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Lorsque vous cliquez sur la vignette SAP Analytics Cloud dans le volet d’accès, vous devez être connecté automatiquement à l’application SAP Analytics Cloud pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)