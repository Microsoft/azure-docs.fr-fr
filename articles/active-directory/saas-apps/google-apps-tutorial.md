---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Google Cloud (G Suite) Connector | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Google Cloud (G Suite) Connector.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/06/2020
ms.author: jeedes
ms.openlocfilehash: 0dd66e246e5e172ad359f5e6e953b360e6e74ebd
ms.sourcegitcommit: ab829133ee7f024f9364cd731e9b14edbe96b496
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/28/2020
ms.locfileid: "97796973"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-google-cloud-g-suite-connector"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Google Cloud (G Suite) Connector

Dans ce tutoriel, vous allez apprendre à intégrer Google Cloud (G Suite) Connector à Azure Active Directory (Azure AD). Lorsque vous intégrez Google Cloud (G Suite) Connector à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Google Cloud (G Suite) Connector.
* Autoriser les utilisateurs à se connecter automatiquement à Google Cloud (G Suite) Connector avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

- Un abonnement Azure AD
- Un abonnement à Google Cloud (G Suite) Connector pour lequel l’authentification unique est activée.
- Un abonnement Google Apps ou Google Cloud Platform.

> [!NOTE]
> Pour tester les étapes de ce didacticiel, nous déconseillons l’utilisation d’un environnement de production. Ce document a été créé à l’aide de la nouvelle expérience utilisateur Authentification unique. Si vous utilisez toujours l’ancienne version, l’installation n’aura pas la même apparence. Vous pouvez activer la nouvelle expérience dans les paramètres d’authentification unique de l’application G Suite. Accédez à **Azure AD, Applications d’entreprise**, sélectionnez **Google Cloud (G Suite) Connector** et **Authentification unique**, puis cliquez sur **Essayer la nouvelle expérience**.

Vous devez en outre suivre les recommandations ci-dessous :

- N’utilisez pas votre environnement de production, sauf si cela est nécessaire.
- Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).

## <a name="frequently-asked-questions"></a>Forum Aux Questions (FAQ)

1. **Q : Cette intégration prend-elle en charge l’intégration SSO Google Cloud Platform à Azure AD ?**

    A : Oui. Google Cloud Platform et Google Apps partagent la même plateforme d’authentification. Pour l’intégration Google Cloud Platform, vous avez donc besoin de configurer la SSO avec Google Apps.

2. **Q : Les Chromebooks et autres appareils Chrome sont-ils compatibles avec l’authentification unique Azure AD ?**
  
    A : Oui, les utilisateurs peuvent se connecter à leurs appareils Chromebook en entrant leurs informations d’identification Azure AD. Consultez cet [article du support technique Google Cloud (G Suite) Connector](https://support.google.com/chrome/a/answer/6060880) pour en savoir plus sur les raisons de la double demande de saisie des informations d’identification.

3. **Q : Si j’ai activé l’authentification unique, les utilisateurs peuvent-ils utiliser leurs informations d’identification Azure AD pour se connecter à un produit Google, comme Google Classroom, Gmail, Google Drive, YouTube, etc. ?**

    A : Oui, en fonction du produit [Google Cloud (G Suite) Connector](https://support.google.com/a/answer/182442?hl=en&ref_topic=1227583) que vous choisissez d’activer et de désactiver pour votre organisation.

4. **Q : Puis-je activer l’authentification unique pour seulement une partie des utilisateurs de Google Cloud (G Suite) Connector ?**

    A : Non, si vous activez l’authentification unique, l’ensemble des utilisateurs de Google Cloud (G Suite) Connector doivent s’authentifier avec leurs informations d’identification Azure AD. Étant donné que Google Cloud (G Suite) Connector ne prend pas en charge les fournisseurs d’identité multiples, le fournisseur d’identité associé à votre environnement Google Cloud (G Suite) Connector peut être Azure AD ou Google, mais pas les deux.

5. **Q : Si un utilisateur est connecté par le biais de Windows, est-il automatiquement authentifié sur Google Cloud (G Suite) Connector sans qu’il ne lui soit demandé d’entrer un mot de passe ?**

    A : Deux options permettent de prendre en charge ce scénario. Tout d’abord, les utilisateurs peuvent se connecter aux appareils Windows 10 via [Azure Active Directory Join](../devices/overview.md). Sinon, les utilisateurs peuvent se connecter aux appareils Windows joints à un domaine au sein d’un répertoire Active Directory sur lequel est activée l’authentification unique à Azure AD via un déploiement [Active Directory Federation Services (AD FS)](../hybrid/plan-connect-user-signin.md) . Quelle que soit l’option choisie, vous devez effectuer le tutoriel ci-dessous pour activer l’authentification unique entre Azure AD et Google Cloud (G Suite) Connector.

6. **Q : Que dois-je faire si j’obtiens un message d’erreur « adresse e-mail non valide » ?**

    A : Pour cette configuration, l’attribut d’adresse e-mail est obligatoire pour permettre aux utilisateurs de se connecter. Cet attribut ne peut pas être défini manuellement.

    L’attribut d’adresse e-mail est rempli automatiquement pour tout utilisateur disposant d’une licence Exchange valide. Si l’utilisateur n’est pas associé à un attribut d’adresse e-mail, l’erreur est générée dans la mesure où l’application a besoin d’obtenir cet attribut pour accorder l’accès.

    Accédez à portal.office.com avec un compte d’administrateur, puis cliquez sur Centre d’administration, Facturation, Abonnements. Sélectionnez ensuite votre abonnement Microsoft 365, cliquez sur Attribuer à des utilisateurs, sélectionnez les utilisateurs dont vous souhaitez vérifier l’abonnement, puis cliquez sur Modifier les licences dans le volet droit.

    Une fois la licence Microsoft 365 attribuée, vous devrez peut-être patienter quelques minutes avant qu’elle ne soit appliquée. Après cela, l’attribut user.mail est rempli automatiquement et le problème doit être résolu.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Google Cloud (G Suite) Connector prend en charge l’authentification unique lancée par le **fournisseur de services**

* Google Cloud (G Suite) Connector prend en charge le [provisionnement **automatique** des utilisateurs](g-suite-provisioning-tutorial.md)
* Après avoir configuré Google Cloud (G Suite) Connector, vous pouvez appliquer le contrôle de session qui protège l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrir comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad)

## <a name="adding-google-cloud-g-suite-connector-from-the-gallery"></a>Ajout de Google Cloud (G Suite) Connector à partir de la galerie

Pour configurer l’intégration de Google Cloud (G Suite) Connector à Azure AD, vous devez ajouter Google Cloud (G Suite) Connector à votre liste d’applications SaaS managées, à partir de la galerie.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **Google Cloud (G Suite) Connector** dans la zone de recherche.
1. Sélectionnez **Google Cloud (G Suite) Connector** dans le panneau de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-google-cloud-g-suite-connector"></a>Configurer et tester l’authentification unique Azure AD pour Google Cloud (G Suite) Connector

Configurez et testez l’authentification unique Azure AD avec Google Cloud (G Suite) Connector pour un utilisateur de test nommé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur associé dans Google Cloud (G Suite) Connector.

Pour configurer et tester l’authentification unique Azure AD avec Google Cloud (G Suite) Connector, suivez les indications des modules ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Google Cloud (G Suite) Connector](#configure-google-cloud-g-suite-connector-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test Google Cloud (G Suite) Connector](#create-google-cloud-g-suite-connector-test-user)** pour avoir dans Google Cloud (G Suite) Connector un équivalent de B.Simon lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [Portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Google Cloud (G Suite) Connector**, recherchez la section **Gérer** et sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base**, si vous souhaitez procéder à la configuration de **Gmail**, effectuez les étapes suivantes :

    a. Dans la zone de texte **URL de connexion**, tapez une URL au format suivant : `https://www.google.com/a/<yourdomain.com>/ServiceLogin?continue=https://mail.google.com`

    b. Dans la zone de texte **Identificateur**, tapez une URL au format suivant :

    ```http
    google.com/a/<yourdomain.com>
    google.com
    https://google.com
    https://google.com/a/<yourdomain.com>
    ```

    c. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : 

    ```http
    https://www.google.com
    https://www.google.com/a/<yourdomain.com>
    ```

1. Dans la section **Configuration SAML de base**, si vous souhaitez procéder à la configuration de **Google Cloud Platform**, effectuez les étapes suivantes :

    a. Dans la zone de texte **URL de connexion**, tapez une URL au format suivant : `https://www.google.com/a/<yourdomain.com>/ServiceLogin?continue=https://console.cloud.google.com`

    b. Dans la zone de texte **Identificateur**, tapez une URL au format suivant :
    
    ```http
    google.com/a/<yourdomain.com>
    google.com
    https://google.com
    https://google.com/a/<yourdomain.com>
    ```
    
    c. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : 
    
    ```http
    https://www.google.com
    https://www.google.com/a/<yourdomain.com>
    ```

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion et l’identificateur réels. Google Cloud (G Suite) Connector ne fournit pas de valeur ID d’entité/Identificateur dans la configuration de l’authentification unique. Ainsi, quand vous décochez la case **Use a domain specific issuer** (Utiliser un émetteur spécifique de domaine), la valeur Identificateur est `google.com`. Si vous cochez la case **Utiliser un émetteur spécifique de domaine**, la valeur est `google.com/a/<yourdomainname.com>`. Pour cocher/décocher la case **Utiliser un émetteur spécifique de domaine**, vous devez accéder à la section **Configurer l’authentification unique Google Cloud (G Suite) Connector**, plus loin dans le tutoriel. Pour plus d’informations, contactez l’[équipe du support technique Google Cloud (G Suite) Connector](https://www.google.com/contact/).

1. Votre application Google Cloud (G Suite) Connector attend les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à votre configuration d’attributs de jeton SAML. La capture d’écran suivante montre un exemple : La valeur par défaut pour **Identificateur d’utilisateur unique** est **user.userprincipalname**, mais Google Cloud (G Suite) Connector s’attend à ce qu’elle soit mappée sur l’adresse e-mail de l’utilisateur. Pour cela, vous pouvez utiliser l’attribut **user.mail** dans la liste ou utiliser la valeur d’attribut appropriée en fonction de la configuration de votre organisation.

    ![image](common/default-attributes.png)


1. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, recherchez **Certificat (en base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer Google Cloud (G Suite) Connector**, copiez la ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous autorisez B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Google Cloud (G Suite) Connector.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **Google Cloud (G Suite) Connector**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

## <a name="configure-google-cloud-g-suite-connector-sso"></a>Configurer l’authentification unique Google Cloud (G Suite) Connector

1. Ouvrez un nouvel onglet dans votre navigateur et utilisez votre compte d’administrateur pour vous connecter à la [console d’administration de Google Cloud (G Suite) Connector](https://admin.google.com/).

2. Cliquez sur **Sécurité**. Si le lien ne s'affiche pas, il est peut-être masqué par le menu **Autres contrôles** situé en bas de l'écran.

    ![Cliquez sur Sécurité.][10]

3. Sur la page **Sécurité**, cliquez sur **Configurer l'authentification unique (SSO).**

    ![Cliquez sur Authentification unique.][11]

4. Modifiez la configuration comme suit :

    ![Configurer l’authentification unique][12]

    a. Sélectionnez **Configurer l'authentification unique avec un fournisseur d'identité tiers**.

    b. Dans le champ **URL de la page de connexion** de Google Cloud (G Suite) Connector, collez l’**URL de connexion** que vous avez copiée à partir du portail Azure.

    c. Dans le champ **URL de la page de déconnexion** de Google Cloud (G Suite) Connector, collez l’**URL de connexion** que vous avez copiée à partir du portail Azure.

    > [!NOTE]
    > Google cloud (G Suite) est basé sur le protocole de déconnexion SAML. Ainsi, dans le champ **URL de la page de déconnexion**, nous devons utiliser l’URL de déconnexion SAML, c’est-à-dire l’URL de connexion comme valeur de la même façon.

    d. Dans Google Cloud (G Suite) Connector, chargez le certificat que vous avez téléchargé du portail Azure pour l’utiliser comme **Certificat de vérification**.   

    e. Cochez/décochez la case **Utiliser un émetteur spécifique de domaine** conformément à la note mentionnée dans la section **Configuration SAML de base** ci-dessus dans Azure AD.

    f. Dans le champ **Modifier l’URL du mot de passe** de Google Cloud (G Suite) Connector, collez la valeur de **Modifier l’URL du mot de passe** que vous avez copiée du portail Azure.

    g. Cliquez sur **Enregistrer**.

### <a name="create-google-cloud-g-suite-connector-test-user"></a>Créer un utilisateur de test Google Cloud (G Suite) Connector

L’objectif de cette section est de [créer un utilisateur dans Google Cloud (G Suite) Connector](https://support.google.com/a/answer/33310?hl=en), appelé B.Simon. Une fois l’utilisateur créé manuellement dans Google Cloud (G Suite) Connector, il peut se connecter à l’aide de ses informations d’identification Microsoft 365.

Google Cloud (G Suite) Connector prend aussi en charge le provisionnement automatique des utilisateurs. Pour configurer le provisionnement automatique d’utilisateurs, vous devez d’abord [configurer Google Cloud (G Suite) Connector pour le provisionnement automatique d’utilisateurs](g-suite-provisioning-tutorial.md).

> [!NOTE]
> Assurez-vous que votre utilisateur existe déjà dans Google Cloud (G Suite) Connector si le provisionnement dans Azure AD n’a pas été activé avant de tester l’authentification unique.

> [!NOTE]
> Si vous devez créer un utilisateur manuellement, contactez [l’équipe de support Google](https://www.google.com/contact/).

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Lorsque vous cliquez sur la vignette Google Cloud (G Suite) Connector dans le volet d’accès, vous êtes automatiquement connecté à l’application Google Cloud (G Suite) Connector pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Configurer l’approvisionnement de l’utilisateur](g-suite-provisioning-tutorial.md)

- [Essayer Google Cloud (G Suite) Connector avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)

- [Guide pratique pour protéger Google Cloud (G Suite) Connector avec une visibilité et des contrôles avancés](/cloud-app-security/protect-gsuite)

<!--Image references-->

[10]: ./media/google-apps-tutorial/gapps-security.png
[11]: ./media/google-apps-tutorial/security-gapps.png
[12]: ./media/google-apps-tutorial/gapps-sso-config.png