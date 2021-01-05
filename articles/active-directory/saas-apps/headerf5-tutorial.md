---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à F5 | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et F5.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 11/19/2019
ms.author: jeedes
ms.openlocfilehash: 8706ac588f2cc868805de7126140a1edd4b8c735
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92445509"
---
# <a name="tutorial-configure-single-sign-on-sso-between-azure-active-directory-and-f5"></a>Tutoriel : Configurer l’authentification unique entre Azure Active Directory et F5

Dans ce tutoriel, vous allez découvrir comment intégrer F5 à Azure Active Directory (Azure AD). Quand vous intégrez F5 à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à F5.
* Permettre à vos utilisateurs de se connecter automatiquement à F5 avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration d’applications SaaS à l’authentification unique dans Azure AD, consultez [S’authentifier avec l’authentification unique auprès des applications dans Azure Active Directory](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).

* Un abonnement F5 pour lequel l’authentification unique est activée

* Le déploiement de la solution conjointe nécessite la licence suivante :

    * F5 BIG-IP® Best Bundle (ou) 

    * Licence autonome F5 BIG-IP Access Policy Manager™ (APM) 

    * Licence de composant additionnel F5 BIG-IP Access Policy Manager™ (APM) sur une instance existante de BIG-IP F5 BIG-IP® Local Traffic Manager™ (LTM).

    * Outre la licence ci-dessus, il est possible de bénéficier d’une licence pour le système F5 avec : 

        * Un abonnement au filtrage d’URL pour utiliser la base de données de catégories d’URL

        * Un abonnement à F5 IP Intelligence pour détecter et bloquer les attaquants connus et le trafic malveillant
     
        * Un module de sécurité matériel (HSM) réseau pour protéger et gérer les clés numériques pour une authentification forte

* Un système F5 BIG-IP provisionné avec les modules APM (LTM est facultatif)

* Bien que facultatif, il est vivement recommandé de déployer les systèmes F5 dans un [groupe d’appareils de synchronisation/basculement](https://techdocs.f5.com/content/techdocs/en-us/bigip-14-1-0/big-ip-device-service-clustering-administration-14-1-0.html) (S/F DG), qui comprend la paire en veille active, avec une adresse IP flottante pour la haute disponibilité. Il est possible de bénéficier d’une redondance d’interface plus poussée en utilisant le protocole LACP (Link Aggregation Control Protocol). LACP gère les interfaces physiques connectées comme une interface virtuelle unique (groupe agrégé) et détecte les échecs d’interface au sein du groupe.

* Pour les applications Kerberos, un compte de service AD local pour la délégation contrainte.  Pour créer un compte de délégation AD, consultez la [documentation F5](https://support.f5.com/csp/article/K43063049).

## <a name="access-guided-configuration"></a>Configuration guidée de l’accès

* La configuration guidée de l’accès est prise en charge sur F5 TMOS version 13.1.0.8 et supérieures. Si votre système BIG-IP exécute une version inférieure à 13.1.0.8, consultez la section **Configuration avancée**.

* La configuration guidée de l’accès présente une expérience utilisateur entièrement nouvelle et simplifiée. Cette architecture à base de workflow propose des étapes de configuration intuitives réentrantes adaptées à la topologie choisie.

* Avant de procéder à la configuration, mettez à niveau la configuration guidée en téléchargeant le dernier pack de cas d’usage à partir de [downloads.f5.com](https://login.f5.com/resource/login.jsp?ctx=719748). Pour effectuer la mise à niveau, suivez la procédure ci-dessous.

    >[!NOTE]
    >Les captures d’écran ci-dessous concernent la dernière version parue (BIG-IP 15.0 avec AGC version 5.0). Les étapes de configuration ci-dessous valent pour ce cas d’usage de la version 13.1.0.8 de BIG-IP à la plus récente.

1. Dans l’interface utilisateur web de F5 BIG-IP, cliquez sur **Access >> Guided Configuration** (Accès >> Configuration guidée).

1. Dans la page **Guided Configuration** (Configuration guidée), cliquez sur **Upgrade Guided Configuration** (Mettre à niveau la configuration guidée) dans l’angle supérieure gauche.

    ![Capture d’écran montrant la page Guided Configuration avec le lien Update Guided Configuration.](./media/headerf5-tutorial/configure14.png) 

1. Dans l’écran contextuel Upgrade Guided Configuration (Mettre à niveau la configuration guidée), sélectionnez **Choose File** (Choisir un fichier) pour charger le pack de cas d’usage téléchargé, puis cliquez sur le bouton **Upload and Install** (Télécharger et installer).

    ![Capture d’écran montrant la boîte de dialogue Upgrade Guided Configuration avec Choose File sélectionné.](./media/headerf5-tutorial/configure15.png) 

1. Une fois la mise à niveau terminée, cliquez sur le bouton **Continue** (Continuer).

    ![Capture d’écran montrant la boîte de dialogue Upgrade Guided Configuration avec un message d’achèvement.](./media/headerf5-tutorial/configure16.png)

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* La fonctionnalité d’authentification unique F5 peut être configurée de trois façons différentes.

- [Configurer l’authentification unique F5 pour une application basée sur l’en-tête](#configure-f5-single-sign-on-for-header-based-application)

- [Configurer l’authentification unique F5 pour une application Kerberos](kerbf5-tutorial.md)

- [Configurer l’authentification unique F5 pour une application Kerberos avancée](advance-kerbf5-tutorial.md)

### <a name="key-authentication-scenarios"></a>Scénarios d’authentification par clé

* En plus de la prise en charge de l’intégration native d’Azure Active Directory pour les protocoles d’authentification modernes que sont notamment Open ID Connect, SAML et WS-Fed, F5 étend l’accès sécurisé aux applications d’authentification héritées pour l’accès interne et externe avec Azure AD, ce qui autorise des scénarios modernes (par exemple, accès sans mot de passe) pour ces applications. Cela inclut :

* Les applications à authentification basée sur l’en-tête

* Les applications à authentification Kerberos

* Les applications à authentification anonyme ou sans authentification intégrée

* Les applications à authentification NTLM (protection avec deux invites pour l’utilisateur)

* Les applications à base de formulaire (protection avec deux invites pour l’utilisateur)

## <a name="adding-f5-from-the-gallery"></a>Ajout de F5 à partir de la galerie

Pour configurer l’intégration de F5 avec Azure AD, vous devez ajouter F5, disponible dans la galerie, à votre liste d’applications SaaS managées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie** , tapez **F5** dans la zone de recherche.
1. Sélectionnez **F5** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-f5"></a>Configurer et tester l’authentification unique Azure AD pour F5

Configurez et testez l’authentification unique Azure AD avec F5 à l’aide d’un utilisateur de test appelé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir une relation entre un utilisateur Azure AD et l’utilisateur F5 associé.

Pour configurer et tester l’authentification unique Azure AD avec F5, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique F5](#configure-f5-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test F5](#create-f5-test-user)** pour disposer, dans F5, d’un équivalent de B.Simon lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans la page d’intégration de l’application **F5** du [portail Azure](https://portal.azure.com/), recherchez la section **Gérer** , puis sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode Initié par le **fournisseur d’identité** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **Identificateur** , tapez une URL au format suivant : `https://<YourCustomFQDN>.f5.com/`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://<YourCustomFQDN>.f5.com/`

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://<YourCustomFQDN>.f5.com/`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur, l’URL de réponse et l’URL de connexion réels. Pour obtenir ces valeurs, contactez l’[équipe de support technique de F5](https://support.f5.com/csp/knowledge-center/software/BIG-IP?module=BIG-IP%20APM45). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **XML de métadonnées de fédération** et **Certificat (en base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

1. Dans la section **Configurer F5** , copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé B. Simon dans le portail Azure.

1. Dans le volet gauche du Portail Azure, sélectionnez **Azure Active Directory** , **Utilisateurs** , puis **Tous les utilisateurs**.
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur** , effectuez les étapes suivantes :
   1. Dans le champ **Nom** , entrez `B.Simon`.  
   1. Dans le champ **Nom de l’utilisateur** , entrez username@companydomain.extension. Par exemple : `B.Simon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ **Mot de passe**.
   1. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à F5.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **F5**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer**.
1. Cliquez sur **Accès conditionnel**.
1. Cliquez sur **Nouvelle stratégie**.
1. Votre application F5 apparaît désormais comme une ressource de la stratégie d’autorité de certification et vous pouvez appliquer n’importe quel accès conditionnel, notamment l’authentification multifacteur, le contrôle d’accès basé sur les appareils ou la stratégie de protection des identités.

## <a name="configure-f5-sso"></a>Configurer l’authentification unique F5

- [Configurer l’authentification unique F5 pour une application Kerberos](kerbf5-tutorial.md)

- [Configurer l’authentification unique F5 pour une application Kerberos avancée](advance-kerbf5-tutorial.md)

### <a name="configure-f5-single-sign-on-for-header-based-application"></a>Configurer l’authentification unique F5 pour une application basé sur l’en-tête

### <a name="guided-configuration"></a>Configuration guidée

1. Ouvrez une nouvelle fenêtre de navigateur web et connectez-vous à votre site d’entreprise F5 (basé sur l’en-tête) en tant qu’administrateur, puis effectuez les étapes suivantes :

1. Accédez à **System > Certificate Management > Traffic Certificate Management > SSL Certificate List** (Système > Gestion des certificats > Gestion des certificats de trafic > Liste des certificats SSL). Sélectionnez **Import** (Importer) en haut à droite. Spécifiez un nom de certificat dans le champ **Certificate Name** (sera référencé plus tard dans la configuration). Dans **Certificate Source** (Source du certificat), sélectionnez Upload File (Charger un fichier), spécifiez le certificat téléchargé à partir d’Azure pendant la configuration de l’authentification unique SAML. Cliquez sur **Importer**.

    ![Capture d’écran montrant la liste de certificats SSL dans laquelle vous sélectionnez le nom du certificat et la source du certificat.](./media/headerf5-tutorial/configure12.png)
 
1. De plus, vous aurez besoin d’un **certificat SSL pour le nom d’hôte de l’application. Accédez à System > Certificate Management > Traffic Certificate Management > SSL Certificate List** (Système > Gestion des certificats > Gestion des certificats de trafic > Liste des certificats SSL). Sélectionnez **Import** (Importer) en haut à droite. **Import Type** (Type d’importation) aura la valeur **PKCS 12 (IIS)** . Spécifiez un nom de clé dans le champ **Key Name** (sera référencé plus tard dans la configuration), puis le fichier PFX. Spécifiez le mot de passe du fichier PFX dans le champ **Password**. Cliquez sur **Importer**.

    >[!NOTE]
    >Dans l’exemple, le nom de notre application est `Headerapp.superdemo.live`, nous utilisons un certificat à caractères génériques, et le nom de clé est `WildCard-SuperDemo.live`.

    ![Capture d’écran montrant la page de source de clé/certificat SSL.](./media/headerf5-tutorial/configure13.png)

1. Nous allons utiliser l’expérience guidée pour configurer l’accès à l’application et la fédération Azure AD. Dans F5 BIG-IP, accédez à l’onglet **Main** (Principal), puis sélectionnez **Access > Guided Configuration > Federation > SAML Service Provider** (Accès > Configuration guidée > Fédération > Fournisseur de services SAML). Cliquez sur **Next** (Suivant) et encore sur **Next** (Suivant) pour commencer la configuration.

    ![Capture d’écran montrant la page Guided Configuration avec Federation sélectionné.](./media/headerf5-tutorial/configure01.png)

    ![Capture d’écran montrant la page de fournisseur de services SAML.](./media/headerf5-tutorial/configure02.png)
 
1. Attribuez un nom à la configuration dans le champ **Configuration Name**. Spécifiez l’ID d’entité dans le champ **Entity ID** (identique à celui que vous avez configuré dans la configuration de l’application Azure AD). Spécifiez le nom d’hôte dans le champ **Host**. Ajoutez une description dans le champ **Description** pour référence. Acceptez les entrées par défaut restantes, puis cliquez sur **Save & Next** (Enregistrer et suivant).

    ![Capture d’écran montrant la page de propriétés du fournisseur de services.](./media/headerf5-tutorial/configure03.png) 

1. Dans cet exemple, nous créons un serveur virtuel dont l’adresse est 192.168.30.20 et le port 443. Spécifiez l’adresse IP du serveur virtuel dans le champ **Destination Address**. En dessous de **Client SSL Profile** (Profil SSL client), sélectionnez Create new (Créer nouveau). Spécifiez le certificat d’application chargé précédemment (le certificat à caractères génériques de cet exemple) et la clé associée, puis cliquez sur **Save & Next** (Enregistrer et suivant).

    >[!NOTE]
    >Dans cet exemple, notre serveur web interne s’exécute sur le port 888 et nous voulons le publier avec 443.

    ![Capture d’écran montrant la page de propriétés de serveur virtuel.](./media/headerf5-tutorial/configure04.png) 

1. Sous **Select method to configure your IdP connector** (Sélectionner une méthode pour configurer votre connecteur IdP), spécifiez Metadata (Métadonnées), cliquez sur Choose File (Choisir un fichier), puis chargez le fichier XML de métadonnées téléchargé précédemment à partir d’Azure AD. Attribuez un nom unique au connecteur IDP SAML dans le champ **Name**. Choisissez le certificat de signature de métadonnées ( **Metadata Signing Certificate** ) qui a été chargé précédemment. Cliquez sur **Save & Next** (Enregistrer et suivant).

    ![Capture d’écran montrant la page de paramètres de connecteur de fournisseur d’identité externe.](./media/headerf5-tutorial/configure05.png)
 
1. Sous **Select a Pool** (Sélectionner un pool), spécifiez **Create New** (Créer nouveau) ou sélectionnez un pool qui existe déjà. Conservez les autres valeurs par défaut. Sous Pool Servers (Serveurs de pool), tapez l’adresse IP sous **IP Address/Node Name** (Adresse IP/Nom du nœud). Spécifiez le **Port**. Cliquez sur **Save & Next** (Enregistrer et suivant).

    ![Capture d’écran montrant la page de propriétés du pool.](./media/headerf5-tutorial/configure06.png)

1. Dans l’écran Single Sign-On Settings (Paramètres d’authentification unique), sélectionnez **Enable Single Sign-On** (Activer l’authentification unique). Sous Selected Single Sign-On Type (Type d’authentification unique sélectionné), choisissez **HTTP header-based** (Basé sur l’en-tête HTTP). Remplacez **session.saml.last.Identity** par **session.saml.last.attr.name.Identity** sous Username Source (Source du nom d’utilisateur). Cette variable est définie à partir du mappage de revendications dans Azure AD. Sous SSO Headers (En-têtes SSO).

    * **HeaderName (Nom d’en-tête) : MyAuthorization**

    * **Header Value (Valeur d’en-tête) :% %{session.saml.last.attr.name.Identity}**

    * Cliquez sur **Save & Next** (Enregistrer et suivant).

    Pour obtenir la liste complète des variables et des valeurs, consultez l’annexe. Vous pouvez ajouter davantage d’en-têtes, si nécessaire.

    >[!NOTE]
    >Le nom du compte est le compte de délégation F5 créé (consultez la documentation F5).

    ![Capture d’écran montrant la page de paramètres d’authentification unique.](./media/headerf5-tutorial/configure07.png) 

1. Dans le cadre de ce guide, nous allons passer les vérifications de point de terminaison.  Consultez la documentation F5 pour plus d’informations. Sélectionnez **Save & Next** (Enregistrer et suivant).

    ![Capture d’écran montrant la page de propriétés des vérifications de points de terminaison.](./media/headerf5-tutorial/configure08.png)

1. Acceptez les valeurs par défaut et cliquez sur **Save & Next** (Enregistrer et suivant). Consultez la documentation F5 pour plus d’informations sur les paramètres de gestion de session SAML.

    ![Capture d’écran montrant la page des paramètres de délai d’attente.](./media/headerf5-tutorial/configure09.png)

1. Passez en revue l’écran récapitulatif, puis sélectionnez **Deploy** (Déployer) pour configurer BIG-IP. Cliquez sur **Finish** (Terminer).

    ![Capture d’écran montrant la page qui indique que votre application est prête à être déployée.](./media/headerf5-tutorial/configure10.png)

    ![Capture d’écran montrant la page qui indique que votre application est déployée.](./media/headerf5-tutorial/configure11.png)

## <a name="advanced-configuration"></a>Configuration avancée

Cette section vous sera utile si vous ne pouvez pas utiliser la configuration guidée ou si vous souhaitez ajouter/modifier des paramètres supplémentaires. Vous aurez besoin d’un certificat TLS/SSL pour le nom d’hôte de l’application.

1. Accédez à **System > Certificate Management > Traffic Certificate Management > SSL Certificate List** (Système > Gestion des certificats > Gestion des certificats de trafic > Liste des certificats SSL). Sélectionnez **Import** (Importer) en haut à droite. **Import Type** (Type d’importation) aura la valeur **PKCS 12 (IIS)** . Spécifiez un nom de clé dans le champ **Key Name** (sera référencé plus tard dans la configuration), puis le fichier PFX. Spécifiez le mot de passe du fichier PFX dans le champ **Password**. Cliquez sur **Importer**.

    >[!NOTE]
    >Dans l’exemple, le nom de notre application est `Headerapp.superdemo.live`, nous utilisons un certificat à caractères génériques, et le nom de clé est `WildCard-SuperDemo.live`.
  
    ![Capture d’écran montrant la page de source de clé/certificat SSL pour la configuration avancée.](./media/headerf5-tutorial/configure17.png)

### <a name="adding-a-new-web-server-to-bigip-f5"></a>Ajout d’un nouveau serveur web à BigIP-F5

1. Cliquez sur **Main > IApps > Application Services > Application > Create** (Principal > IApps > Services d’application > Application > Créer).

1. Indiquez le nom dans le champ **Name** puis, sous **Template** (Modèle), choisissez **F5.http**.
 
    ![Capture d’écran montrant la page de services d’application avec sélection de modèle.](./media/headerf5-tutorial/configure18.png)

1. Nous publierons notre HeaderApp2 en externe en HTTPS dans ce cas. Pour **How should the BIG-IP system handle SSL Traffic?** (Comment le système BIG-IP gère-t-il le trafic SSL ?), nous spécifions **Terminate SSL from Client, Plaintext to servers (SSL Offload)** [Mettre fin à SSL à partir du client, Texte clair vers les serveurs (Déchargement SSL)]. Spécifiez votre certificat et votre clé sous **Which SSL certificate do you want to use?** (Quel certificat SSL voulez-vous utiliser ?) et **Which SSL private key do you want to use?** (Quelle clé privée SSL voulez-vous utiliser ?). Spécifiez l’adresse IP du serveur virtuel sous **What IP Address do you want to use for the Virtual Server?** (Quelle adresse IP voulez-vous utiliser pour le serveur virtuel ?). 

    * **Spécifiez d’autres détails**

        * FQDN  

        * Spécifiez une sortie du pool d’applications ou créez-en un nouveau.

        * Si vous créez un serveur d’applications, spécifiez l’ **adresse IP interne** et le **numéro de port**.

        ![Capture d’écran montrant le volet dans lequel vous pouvez spécifier ces détails.](./media/headerf5-tutorial/configure19.png) 

1. Cliquez sur **Finished** (Terminé).

    ![Capture d’écran montrant la page après l’achèvement.](./media/headerf5-tutorial/configure20.png) 

1. Vérifiez que les propriétés de l’application peuvent être modifiées. Cliquez sur **Main > IApps > Application Services: Applications >> HeaderApp2** (Principal > IApps > Services d’application : Applications >> HeaderApp2). Décochez **Strict Updates** (Mises à jour strictes) (nous modifierons certains paramètres en dehors de l’interface graphique utilisateur). Cliquez sur le bouton **Update** (Mettre à jour).

    ![Capture d’écran montrant la page de services d’application avec l’onglet de propriétés sélectionné.](./media/headerf5-tutorial/configure21.png) 

1. À ce stade, vous devez pouvoir parcourir le serveur virtuel.

### <a name="configuring-f5-as-sp-and-azure-as-idp"></a>Configuration de F5 en tant que fournisseur de services (SP) et Azure en tant que fournisseur d’identité (IDP)

1.  Cliquez sur **Access > Federation> SAML Service Provider > Local SP Service** (Accès > Fédération > Fournisseur de services SAML > Service de fournisseur de services local), puis cliquez sur Create (Créer) ou sur le signe +.

    ![Capture d’écran montrant la page About this BIG I P. ](./media/headerf5-tutorial/configure22.png)

1. Spécifiez les détails du service de fournisseur de services. Spécifiez le nom représentant la configuration du fournisseur de services F5 dans le champ **Name**. Spécifiez l’ID d’entité dans le champ **Entity ID** (généralement identique à l’URL de l’application).

    ![Capture d’écran montrant la page SAML Service Provider avec la boîte de dialogue Create New SAML SP Service.](./media/headerf5-tutorial/configure23.png)

    ![Capture d’écran montrant la boîte de dialogue Create New SAML SP Service avec Endpoint Settings sélectionné.](./media/headerf5-tutorial/configure24.png)

    ![Capture d’écran montrant la boîte de dialogue Create New SAML SP Service avec Security Settings sélectionné.](./media/headerf5-tutorial/configure25.png)

    ![Capture d’écran montrant la boîte de dialogue Create New SAML SP Service avec Authentication Context sélectionné.](./media/headerf5-tutorial/configure26.png)

    ![Capture d’écran montrant la boîte de dialogue Create New SAML SP Service avec Requested Attributes sélectionné.](./media/headerf5-tutorial/configure27.png)

    ![Capture d’écran montrant la boîte de dialogue Edit SAML SP Service avec Advanced Settings sélectionné.](./media/headerf5-tutorial/configure28.png)

### <a name="create-idp-connector"></a>Créer un connecteur IDP

1. Cliquez sur le bouton **Bind/Unbind IdP Connectors** (Lier/Dissocier les connecteurs IdP), sélectionnez **Create New IdP Connector** (Créer un connecteur IdP), puis choisissez l’option **From Metadata** (À partir des métadonnées), puis effectuez les étapes suivantes :
 
    ![Capture d’écran montrant la boîte de dialogue Edit SAML IdPs that use this SP avec l’option Create New IdP Connector sélectionnée.](./media/headerf5-tutorial/configure29.png)

    a. Accédez au fichier metadata.xml téléchargé à partir d’Azure AD et spécifiez un nom de fournisseur d’identité dans le champ **Identity Provider Name**.

    b. Cliquez sur **OK**.

    c. Le connecteur est créé et le certificat est automatiquement prêt à partir du fichier medatada.Xml.
    
    ![Capture d’écran montrant la boîte de dialogue Create New SAML IdP Connector.](./media/headerf5-tutorial/configure30.png)

    d. Configurez F5BIG-IP pour envoyer toutes les demandes à Azure AD.

    e. Cliquez sur **Add New Row** (Ajouter une ligne), choisissez **AzureIDP** (tel qu’il a été créé dans les étapes précédentes, spécifiez 

    f. **Matching Source (Source correspondante)   =  %{session.server.landinguri}** 

    g. **Matching Value (Valeur correspondante)     = /** _

    h. Cliquez sur _ *Update** (Mettre à jour).

    i. Cliquez sur **OK**

    j. **La configuration du fournisseur d’identité SAML est terminée**.
    
    ![Capture d’écran montrant la boîte de dialogue Edit SAML IdPs that use this SP.](./media/headerf5-tutorial/configure31.png)

### <a name="configure-f5-policy-to-redirect-users-to-azure-saml-idp"></a>Configurer la stratégie F5 pour rediriger les utilisateurs vers le fournisseur d’identité SAML Azure

1. Pour configurer la stratégie F5 de façon à rediriger les utilisateurs vers le fournisseur d’identité SAML Azure, effectuez les étapes suivantes :

    a. Cliquez sur **Main > Access > Profile/Policies > Access Profiles** (Principal > Profil/Stratégies > Profils d’accès).

    b. Cliquez sur le bouton **Créer**.

    ![Capture d’écran montrant la page Access Profiles.](./media/headerf5-tutorial/configure32.png)
 
    c. Spécifiez un nom dans le champ **Name** (HeaderAppAzureSAMLPolicy dans l’exemple).

    d. Vous pouvez personnaliser d’autres paramètres ; consultez la documentation F5.

    ![Capture d’écran montrant la page General Properties.](./media/headerf5-tutorial/configure33.png)

    ![Capture d’écran montrant la suite de la page General Properties.](./media/headerf5-tutorial/configure34.png) 

    e. Cliquez sur **Finished** (Terminé).

    f. Une fois la stratégie créée, cliquez sur la stratégie et accédez à l’onglet **Access Policy** (Stratégie d’accès).

    ![Capture d’écran montrant l’onglet Access Policy avec les propriétés générales.](./media/headerf5-tutorial/configure35.png)
 
    g. Dans **Visual Policy Editor** , cliquez sur le lien **Access Policy for Profile** (Stratégie d’accès du profil).

    h. Cliquez sur le signe + dans Visual Policy Editor, choisissez **SAML Auth** (Authentification SAML).

    ![Capture d’écran montrant une stratégie d’accès.](./media/headerf5-tutorial/configure36.png)

    ![Capture d’écran montrant une boîte de dialogue de recherche avec l’authentification SAML sélectionnée.](./media/headerf5-tutorial/configure37.png)
 
    i. Cliquez sur **Add Item** (Ajouter un élément).

    j. Sous **Properties** (Propriétés), spécifiez un nom dans le champ **Name** et, sous **AAA Server** (Serveur AAA), sélectionnez le fournisseur de services configuré précédemment, puis cliquez sur **Save** (Enregistrer).
 
    ![Capture d’écran montrant les propriétés de l’élément, y compris son serveur AAA.](./media/headerf5-tutorial/configure38.png)

    k. La stratégie de base est prête. Vous pouvez personnaliser la stratégie pour incorporer des sources/magasins d’attributs supplémentaires.

    ![Capture d’écran montrant la stratégie personnalisée.](./media/headerf5-tutorial/configure39.png)
 
    l. Veillez à cliquer sur le lien **Apply Access Policy** (Appliquer la stratégie d’accès) dans la partie supérieure.

### <a name="apply-access-profile-to-the-virtual-server"></a>Appliquer le profil d’accès au serveur virtuel

1. Affectez le profil d’accès au serveur virtuel pour que F5 BIG-IP APM applique les paramètres du profil au trafic entrant et exécute la stratégie d’accès définie précédemment.

    a. Cliquez sur **Main** > **Local Traffic** > **Virtual Servers** (Principal > Trafic local > Serveurs virtuels).

    ![Capture d’écran montrant la page Virtual Servers List.](./media/headerf5-tutorial/configure40.png)
 
    b. Cliquez sur le serveur virtuel, accédez à la section **Access Policy** (Stratégie d’accès). Dans la liste déroulante **Access Profil** (Profil d’accès), sélectionnez la stratégie SAML créée (dans l’exemple, HeaderAppAzureSAMLPolicy).

    c. Cliquez sur **Update** (Mettre à jour).
 
    ![Capture d’écran montrant le volet Access Policy.](./media/headerf5-tutorial/configure41.png)

    d. Créez une règle F5 BIG-IP iRule® pour extraire les attributs SAML personnalisés de l’assertion entrante et transmettez-les en tant qu’en-têtes HTTP à l’application de test backend. Cliquez sur **Main > Local Traffic > iRules > iRule List** (Principal > Trafic local > Règles iRule > Liste de règles iRule), puis cliquez sur Create (Créer).

    ![Capture d’écran montrant la liste Local Traffic iRule.](./media/headerf5-tutorial/configure42.png)
 
    e. Collez le texte de la règle F5 BIG-IP iRule en dessous dans la fenêtre de définition.

    ![Capture d’écran montrant la page New iRule.](./media/headerf5-tutorial/configure43.png)
 
    when RULE_INIT {  set static::debug 0  }  when ACCESS_ACL_ALLOWED {

    set AZUREAD_USERNAME [ACCESS::session data get "session.saml.last.attr.name. http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name"] if { $static::debug } { log local0. "AZUREAD_USERNAME = $AZUREAD_USERNAME" } if { !([HTTP::header exists "AZUREAD_USERNAME"]) } { HTTP::header insert "AZUREAD_USERNAME" $AZUREAD_USERNAME }

    set AZUREAD_DISPLAYNAME [ACCESS::session data get "session.saml.last.attr.name. http://schemas.microsoft.com/identity/claims/displayname"] if { $static::debug } { log local0. "AZUREAD_DISPLAYNAME = $AZUREAD_DISPLAYNAME" } if { !([HTTP::header exists "AZUREAD_DISPLAYNAME"]) } { HTTP::header insert "AZUREAD_DISPLAYNAME" $AZUREAD_DISPLAYNAME }

    set AZUREAD_EMAILADDRESS [ACCESS::session data get "session.saml.last.attr.name. http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress"] if { $static::debug } { log local0. "AZUREAD_EMAILADDRESS = $AZUREAD_EMAILADDRESS" } if { !([HTTP::header exists "AZUREAD_EMAILADDRESS"]) } { HTTP::header insert "AZUREAD_EMAILADDRESS" $AZUREAD_EMAILADDRESS }}

    **Exemple de sortie ci-dessous**

    ![Capture d’écran montrant l’exemple de sortie.](./media/headerf5-tutorial/configure44.png)
 
### <a name="create-f5-test-user"></a>Créer un utilisateur de test F5

Dans cette section, vous allez créer un utilisateur appelé B.Simon dans F5. Collaborez avec l’[équipe du support client de F5](https://support.f5.com/csp/knowledge-center/software/BIG-IP?module=BIG-IP%20APM45) pour ajouter des utilisateurs à la plateforme F5. Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique. 

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette F5 dans le volet d’accès, vous devez être connecté automatiquement à l’application F5 pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer F5 avec Azure AD](https://aad.portal.azure.com/)

- [Configurer l’authentification unique F5 pour une application Kerberos](kerbf5-tutorial.md)

- [Configurer l’authentification unique F5 pour une application Kerberos avancée](advance-kerbf5-tutorial.md)