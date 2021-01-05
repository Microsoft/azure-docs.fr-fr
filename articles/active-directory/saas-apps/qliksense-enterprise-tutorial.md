---
title: 'Tutoriel : Intégration d’Azure Active Directory à Qlik Sense Enterprise | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Qlik Sense Enterprise.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/03/2020
ms.author: jeedes
ms.openlocfilehash: b55e66eaf4bda06369711e389629b6a20765271d
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92522204"
---
# <a name="tutorial-integrate-qlik-sense-enterprise-with-azure-active-directory"></a>Tutoriel : Intégrer Qlik Sense Enterprise avec Azure Active Directory

Dans ce didacticiel, vous allez apprendre à intégrer Qlik Sense Enterprise avec Azure Active Directory (Azure AD). Lorsque vous intégrez Qlik Sense Enterprise avec Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Qlik Sense Enterprise.
* Permettre à vos utilisateurs d’être automatiquement connectés à Qlik Sense Enterprise avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous n’en avez pas, vous pouvez obtenir un essai gratuit d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Un abonnement Qlik Sense Enterprise pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test. 
* Qlik Sense Enterprise prend en charge l’authentification unique initiée par le **fournisseur de services** .
* Qlik Sense Enterprise prend en charge le **provisionnement juste-à-temps**

* Après avoir configuré Qlik Sense Enterprise, vous pouvez appliquer le contrôle de session, qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrir comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad)

## <a name="adding-qlik-sense-enterprise-from-the-gallery"></a>Ajout de Qlik Sense Enterprise à partir de la galerie

Pour configurer l’intégration de Qlik Sense Enterprise à Azure AD, vous devez ajouter Qlik Sense Enterprise depuis la galerie à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la zone de recherche de la section **Ajouter à partir de la galerie** , tapez **Qlik Sense Enterprise** .
1. Sélectionnez **Qlik Sense Enterprise** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Configurez et testez l’authentification unique Azure AD avec Qlik Sense Enterprise à l’aide d’un utilisateur de test nommé **Britta Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Qlik Sense Enterprise associé.

Pour configurer et tester l’authentification unique Azure AD avec Qlik Sense Enterprise, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Qlik Sense Enterprise](#configure-qlik-sense-enterprise-sso)** pour configurer les paramètres de l’authentification unique côté application.
    * **[Créer un utilisateur de test Qlik Sense Enterprise](#create-qlik-sense-enterprise-test-user)** pour avoir un équivalent de Britta Simon dans Qlik Sense Enterprise, lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Qlik Sense Enterprise** , recherchez la section **Gérer** , puis sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet pour **Configuration SAML de base** afin de modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la boîte de dialogue **Configuration SAML de base** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://<Fully Qualified Domain Name>:443{/virtualproxyprefix}/hub`

    b. Dans la zone de texte **Identificateur** , tapez une URL en utilisant l’un des formats suivants :

    ```http
    https://<Fully Qualified Domain Name>.qlikpoc.com
    https://<Fully Qualified Domain Name>.qliksense.com
    ```

    c. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant :

    `https://<Fully Qualified Domain Name>:443{/virtualproxyprefix}/samlauthn/`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL d’authentification, l’identificateur et l’URL de réponse réels décrits plus loin dans ce tutoriel, ou contactez le [support technique de Qlik Sense Enterprise](https://www.qlik.com/us/services/support) pour obtenir ces valeurs. Le port par défaut pour les URL est 443, mais vous pouvez le personnaliser selon les besoins de votre organisation.

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez dans les options proposées le fichier **XML de métadonnées de fédération** correspondant à votre besoin, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé Britta Simon dans le Portail Azure.

1. Dans le volet gauche du Portail Azure, sélectionnez **Azure Active Directory** , **Utilisateurs** , puis **Tous les utilisateurs** .
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur** , effectuez les étapes suivantes :
   1. Dans le champ **Nom** , entrez `Britta Simon`.  
   1. Dans le champ **Nom de l’utilisateur** , entrez username@companydomain.extension. Par exemple : `BrittaSimon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ **Mot de passe** .
   1. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Qlik Sense Enterprise.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Qlik Sense Enterprise** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-qlik-sense-enterprise-sso"></a>Configurer l’authentification unique Qlik Sense Enterprise

1. Préparez le fichier XML de métadonnées de fédération afin de le charger vers le serveur Qlik Sense.

    > [!NOTE]
    > Avant de charger les métadonnées IdP vers le serveur Qlik Sense, vous devez modifier le fichier et supprimer des informations pour vous assurer du bon fonctionnement entre Azure AD et le serveur Qlik Sense.

    ![Capture d’écran affichant une fenêtre Visual Studio Code avec le fichier XML de métadonnées de Fédération.][qs24]

    a. Ouvrez le fichier FederationMetaData.xml téléchargé à partir du portail Azure dans un éditeur de texte.

    b. Recherchez la valeur **RoleDescriptor** .  Il en existe quatre entrées (deux paires de balises d’élément d’ouverture et de fermeture).

    c. Dans le fichier, supprimez les balises RoleDescriptor et toutes les informations insérées entre celles-ci.

    d. Enregistrez le fichier et conservez-le pour l’utiliser ultérieurement dans ce document.

2. Accédez à la console de gestion Qlik (QCM) de Qlik Sense en tant qu’utilisateur pouvant créer des configurations de proxy virtuel.

3. Dans la console QMC, cliquez sur l’élément de menu **Proxys virtuels** .

    ![Capture d’écran affichant les proxys virtuels sélectionnés dans CONFIGURE SYSTEM.][qs6]

4. En bas de l’écran, cliquez sur le bouton **Créer nouveau** .

    ![Capture d’écran montrant l’option Create new.][qs7]

5. L’écran Virtual proxy edit (Modification du proxy virtuel) s’affiche.  Un menu permettant de rendre les options de configuration visibles est affiché sur le côté droit de l’écran.

    ![Capture d’écran affichant la sélection de Identification dans Properties.][qs9]

6. L’option de menu Identification étant sélectionnée, entrez les informations d’identification pour la configuration du proxy virtuel Azure.

    ![Capture d’écran montrant la section Edit virtual proxy Identification, dans laquelle vous pouvez entrer les valeurs décrites.][qs8]  

    a. Le champ **Description** contient un nom convivial pour la configuration du proxy virtuel.  Entrez une valeur pour la description.

    b. Le champ **Préfixe** identifie le point de terminaison proxy virtuel pour la connexion à Qlik Sense avec l’authentification unique Azure AD.  Entrez un nom de préfixe unique pour ce proxy virtuel.

    c. La valeur **Session inactivity timeout** (Délai d’inactivité de session) (minutes) représente le délai d’expiration des connexions via ce proxy virtuel.

    d. La zone **Session cookie header name** (Nom d’en-tête de cookie de session) est le nom du cookie stockant l’identificateur de session pour la session Qlik Sense qu’un utilisateur reçoit après une authentification réussie.  Ce nom doit être unique.

7. Cliquez sur l’option de menu Authentification pour la rendre visible.  L’écran Authentification s’affiche.

    ![Capture d’écran montrant la section Edit virtual proxy Authentication, dans laquelle vous pouvez entrer les valeurs décrites.][qs10]

    a. La liste déroulante **Anonymous access mode (Mode d’accès anonyme)** détermine si des utilisateurs anonymes peuvent accéder à Qlik Sense via le proxy virtuel.  L’option par défaut est No anonymous user (Pas d’utilisateur anonyme).

    b. La liste déroulante **Méthode d’authentification** détermine le schéma d’authentification qui sera utilisé par le proxy virtuel.  Dans la liste déroulante, sélectionnez SAML.  À la suite de cette sélection, d’autres options s’affichent.

    c. Dans le champ **SAML host URI** (URI d’hôte SAML), indiquez le nom d’hôte que les utilisateurs doivent entrer pour accéder à Qlik Sense via ce proxy virtuel SAML.  Le nom d’hôte est l’URI du serveur Qlik Sense.

    d. Dans le champ **SAML entity ID (ID d’entité SAML)** , entrez la valeur indiquée dans le champ SAML host URI (URI d’hôte SAML).

    e. La zone **SAML IdP metadata** (Métadonnées IdP SAML) indique le fichier modifié précédemment dans la section **Modifier les métadonnées de fédération dans la configuration d’Azure AD** .  **Avant de charger les métadonnées IdP, vous devez modifier le fichier** et supprimer des informations pour vous assurer du bon fonctionnement entre Azure AD et le serveur Qlik Sense.  **Reportez-vous aux instructions ci-dessus si le fichier doit encore être modifié.**  Si le fichier a été modifié, cliquez sur le bouton Parcourir et sélectionnez le fichier de métadonnées modifié pour le charger vers la configuration du proxy virtuel.

    f. Entrez le nom d’attribut ou la référence de schéma pour l’attribut SAML représentant **l’ID utilisateur** qu’Azure AD envoie au serveur Qlik Sense.  Les informations de référence de schéma sont disponibles dans les écrans de l’application Azure post-configuration.  Pour utiliser le nom d’attribut, tapez `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`.

    g. Entrez la valeur du **répertoire utilisateur** qui est attaché aux utilisateurs lorsqu’ils s’authentifient sur le serveur Qlik Sense via Azure AD.  Les valeurs codées en dur doivent être entourées de **crochets []** .  Pour utiliser un attribut envoyé dans l’assertion SAML Azure AD, entrez son nom dans cette zone de texte **sans** crochets.

    h. **L’algorithme de signature SAML** définit la signature du certificat du fournisseur de service (dans ce cas le serveur Qlik Sense) pour la configuration du proxy virtuel.  Si le serveur Qlik Sense utilise un certificat approuvé généré à l’aide de Microsoft Enhanced RSA and AES Cryptographic Provider, définissez l’algorithme de signature SAML sur **SHA-256** .

    i. La section SAML attribute mapping (Mappage d’attributs SAML) permet d’envoyer des attributs supplémentaires, comme des groupes, vers Qlik Sense pour les utiliser dans les règles de sécurité.

8. Cliquez sur l’option de menu **ÉQUILIBRAGE DE CHARGE** pour la rendre visible.  L’écran Équilibrage de charge s’affiche.

    ![Capture d’écran affichant l’écran de modification du proxy virtuel pour LOAD BALANCING, dans lequel vous pouvez sélectionner Add new server node.][qs11]

9. Cliquez sur le bouton **Add new server node** (Ajouter un nouveau nœud serveur), sélectionnez un ou plusieurs nœuds de moteur auxquels Qlik Sense enverra des sessions à des fins d’équilibrage de charge, puis cliquez sur le bouton **Ajouter** .

    ![Capture d’écran affichant la boîte de dialogue Add server nodes to load balance on, dans laquelle vous pouvez ajouter des serveurs avec le bouton Add.][qs12]

10. Cliquez sur l’option de menu Avancé pour la rendre visible. L’écran Avancé s’affiche.

    ![Capture d’écran montrant l’écran Edit virtual proxy avec l’option Advanced.][qs13]

    La liste verte Hôte répertorie les noms d’hôte acceptés lors de la connexion au serveur Qlik Sense.  **Entrez le nom d’hôte spécifié par les utilisateurs pour se connecter au serveur Qlik Sense.** Le nom d’hôte est défini sur la même valeur que l’URI d’hôte SAML sans https://.

11. Cliquez sur le bouton **Appliquer** .

    ![Capture d’écran montrant le bouton Apply.][qs14]

12. Cliquez sur OK pour accepter le message d’avertissement indiquant que les proxys liés au proxy virtuel vont être redémarrés.

    ![Capture d’écran affichant le message de confirmation Apply changes to virtual proxy.][qs15]

13. Sur le côté droit de l’écran, le menu Éléments associés s’affiche.  Cliquez sur l’option de menu **Proxys** .

    ![Capture d’écran affichant la sélection Proxies à partir de Associated items.][qs16]

14. L’écran Proxy s’affiche.  Cliquez sur le bouton **Lier** en bas de l’écran pour lier un proxy au proxy virtuel.

    ![Capture d’écran montrant le bouton Link.][qs17]

15. Sélectionnez le nœud proxy qui prendra en charge cette connexion de proxy virtuel et cliquez sur le bouton **Lier** .  Après l’établissement du lien, le proxy est répertorié dans les proxys associés.

    ![Capture d’écran affichant Select proxy services.][qs18]
  
    ![Capture d’écran affichant Associated proxies dans la boîte de dialogue Virtual proxy associated items.][qs19]

16. Après environ cinq à dix secondes, le message Refresh QMC (Actualiser la console QMC) s’affiche.  Cliquez sur le bouton **Refresh QMC** (Actualiser la console QMC).

    ![Capture d’écran affichant le message Your session has ended.][qs20]

17. Lors de l’actualisation de la console QMC, cliquez sur l’élément de menu **Proxys virtuels** . La nouvelle entrée de proxy virtuel SAML est indiquée dans le tableau affiché à l’écran.  Cliquez sur l’entrée de proxy virtuel.

    ![Capture d’écran montrant Virtual proxies avec une seule entrée.][qs51]

18. En bas de l’écran, le bouton Download SP metadata (Télécharger les métadonnées du fournisseur de service) est activé.  Cliquez sur le bouton **Download SP metadata** (Télécharger les métadonnées du fournisseur de service) pour enregistrer les métadonnées dans un fichier.

    ![Capture d’écran montrant le bouton Download SP metadata.][qs52]

19. Ouvrez le fichier de métadonnées du fournisseur de service.  Observez les entrées **entityID** et **AssertionConsumerService** .  Ces valeurs sont équivalentes à celles des champs **Identificateur** , **URL de connexion** et **URL de réponse** dans la configuration d’application Azure AD. Collez ces valeurs dans la section **Domaine et URL Qlik Sense Enterprise** de la configuration de l’application Azure AD si elles ne sont pas identiques. Vous devez ensuite les remplacer dans l’assistant de configuration de l’application Azure AD.

    ![Capture d’écran montrant un éditeur de texte brut avec EntityDescriptor dont entityID et AssertionConsumerService sont mis en évidence.][qs53]

### <a name="create-qlik-sense-enterprise-test-user"></a>Créer un utilisateur de test Qlik Sense Enterprise

Qlik Sense Enterprise prend en charge le **provisionnement juste-à-temps** , les utilisateurs ajoutés automatiquement au dépôt « USERS » de Qlik Sense Enterprise quand ils utilisent la fonctionnalité SSO. Par ailleurs, les clients peuvent utiliser la console QMC et créer un connecteur d’annuaire d’utilisateurs (UDC) pour préremplir les utilisateurs dans Qlik Sense Enterprise à partir du service d’annuaire LDAP de leur choix, comme Active Directory ou autre.

### <a name="test-sso"></a>Tester l’authentification unique (SSO)

Lorsque vous sélectionnez la vignette Qlik Sense Enterprise dans le volet d’accès, vous devez être connecté automatiquement à votre application Qlik Sense Enterprise pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)

<!--Image references-->

[qs6]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_06.png
[qs7]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_07.png
[qs8]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_08.png
[qs9]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_09.png
[qs10]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_10.png
[qs11]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_11.png
[qs12]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_12.png
[qs13]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_13.png
[qs14]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_14.png
[qs15]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_15.png
[qs16]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_16.png
[qs17]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_17.png
[qs18]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_18.png
[qs19]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_19.png
[qs20]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_20.png
[qs24]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_24.png
[qs51]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_51.png
[qs52]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_52.png
[qs53]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_53.png