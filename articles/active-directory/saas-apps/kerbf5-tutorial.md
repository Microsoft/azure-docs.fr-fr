---
title: 'Tutoriel : Intégration de l’authentification unique Azure AD à F5 | Microsoft Docs'
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
ms.openlocfilehash: 161348f7566ff64858d563f34ad8f3f4c7511adf
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96009158"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-f5"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à F5

Dans ce tutoriel, vous allez découvrir comment intégrer F5 à Azure Active Directory (Azure AD). Quand vous intégrez F5 à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à F5.
* Permettre à vos utilisateurs de se connecter automatiquement à F5 avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).

* Un abonnement F5 pour lequel l’authentification unique est activée

* Le déploiement de la solution conjointe nécessite la licence suivante :
    * F5 BIG-IP&reg; Best Bundle (ou)

    * Licence autonome F5 BIG-IP Access Policy Manager&trade; (APM)

    * Licence de composant additionnel F5 BIG-IP Access Policy Manager&trade; (APM) sur une instance de BIG-IP F5 BIG-IP&reg; Local Traffic Manager&trade; (LTM) existante.

    * Outre la licence ci-dessus, il est possible de bénéficier d’une licence pour le système F5 avec :

        * Un abonnement au filtrage d’URL pour utiliser la base de données de catégories d’URL

        * Un abonnement à F5 IP Intelligence pour détecter et bloquer les attaquants connus et le trafic malveillant

        * Un module de sécurité matériel (HSM) réseau pour protéger et gérer les clés numériques pour une authentification forte

* Un système F5 BIG-IP provisionné avec les modules APM (LTM est facultatif)

* Bien que facultatif, il est vivement recommandé de déployer les systèmes F5 dans un [groupe d’appareils de synchronisation/basculement](https://techdocs.f5.com/content/techdocs/en-us/bigip-14-1-0/big-ip-device-service-clustering-administration-14-1-0.html) (S/F DG), qui comprend la paire en veille active, avec une adresse IP flottante pour la haute disponibilité. Il est possible de bénéficier d’une redondance d’interface plus poussée en utilisant le protocole LACP (Link Aggregation Control Protocol). LACP gère les interfaces physiques connectées comme une interface virtuelle unique (groupe agrégé) et détecte les échecs d’interface au sein du groupe.

* Pour les applications Kerberos, un compte de service AD local pour la délégation contrainte.  Pour créer un compte de délégation AD, consultez la [documentation F5](https://support.f5.com/csp/article/K43063049).

## <a name="access-guided-configuration"></a>Configuration guidée de l’accès

* La configuration guidée de l’accès est prise en charge sur F5 TMOS version 13.1.0.8 et ultérieures. Si votre système BIG-IP exécute une version inférieure à 13.1.0.8, consultez la section **Configuration avancée**.

* La configuration guidée de l’accès présente une expérience utilisateur entièrement nouvelle et simplifiée. Cette architecture à base de workflow propose des étapes de configuration intuitives réentrantes adaptées à la topologie choisie.

* Avant de procéder à la configuration, mettez à niveau la configuration guidée en téléchargeant le dernier pack de cas d’usage à partir de [downloads.f5.com](https://login.f5.com/resource/login.jsp?ctx=719748). Pour effectuer la mise à niveau, suivez la procédure ci-dessous.

    >[!NOTE]
    >Les captures d’écran ci-dessous concernent la dernière version parue (BIG-IP 15.0 avec AGC version 5.0). Les étapes de configuration ci-dessous valent pour ce cas d’usage de la version 13.1.0.8 de BIG-IP à la plus récente.

1. Dans l’interface utilisateur web de F5 BIG-IP, cliquez sur **Access >> Guided Configuration** (Accès >> Configuration guidée).

2. Dans la page **Guided Configuration** (Configuration guidée), cliquez sur **Upgrade Guided Configuration** (Mettre à niveau la configuration guidée) dans l’angle supérieure gauche.

    ![Capture d’écran montrant la page « Guided Configuration » avec l’action « Upgrade Guided Configuration » sélectionnée.](./media/kerbf5-tutorial/configure14.png) 

3. Dans l’écran contextuel Upgrade Guided Configuration (Mettre à niveau la configuration guidée), sélectionnez **Choose File** (Choisir un fichier) pour charger le pack de cas d’usage téléchargé, puis cliquez sur le bouton **Upload and Install** (Télécharger et installer).

    ![Capture d’écran montrant l’écran contextuel « Upgrade Guided Configuration » avec les options « Choose File » et « Upload and Install » sélectionnées.](./media/kerbf5-tutorial/configure15.png) 

4. Une fois la mise à niveau terminée, cliquez sur le bouton **Continue** (Continuer).

    ![Capture d’écran montrant la boîte de dialogue « Guided Configuration update is complete » et le bouton « Continue » sélectionné.](./media/kerbf5-tutorial/configure16.png)

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* F5 prend en charge l’authentification unique lancée par le **fournisseur de services et le fournisseur d’identité**.
* La fonctionnalité d’authentification unique F5 peut être configurée de trois façons différentes.

- [Configurer l’authentification unique F5 pour une application Kerberos](#configure-f5-single-sign-on-for-kerberos-application)

- [Configurer l’authentification unique F5 pour une application basée sur l’en-tête](headerf5-tutorial.md)

- [Configurer l’authentification unique F5 pour une application Kerberos avancée](advance-kerbf5-tutorial.md)

### <a name="key-authentication-scenarios"></a>Scénarios d’authentification par clé

En plus de la prise en charge de l’intégration native d’Azure Active Directory pour les protocoles d’authentification modernes que sont notamment Open ID Connect, SAML et WS-Fed, F5 étend l’accès sécurisé aux applications d’authentification héritées pour l’accès interne et externe avec Azure AD, ce qui autorise des scénarios modernes (par exemple, accès sans mot de passe) pour ces applications. Cela inclut :

* Les applications à authentification basée sur l’en-tête

* Les applications à authentification Kerberos

* Les applications à authentification anonyme ou sans authentification intégrée

* Les applications à authentification NTLM (protection avec deux invites pour l’utilisateur)

* Les applications à base de formulaire (protection avec deux invites pour l’utilisateur)

## <a name="adding-f5-from-the-gallery"></a>Ajout de F5 à partir de la galerie

Pour configurer l’intégration de F5 avec Azure AD, vous devez ajouter F5, disponible dans la galerie, à votre liste d’applications SaaS managées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **F5** dans la zone de recherche.
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

1. Dans la page d’intégration de l’application **F5** du [portail Azure](https://portal.azure.com/), recherchez la section **Gérer**, puis sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base**, si vous souhaitez configurer l’application en mode Initié par le **fournisseur d’identité**, entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **Identificateur**, tapez une URL au format suivant : `https://<YourCustomFQDN>.f5.com/`

    b. Dans la zone de texte **URL de réponse**, tapez une URL au format suivant : `https://<YourCustomFQDN>.f5.com/`

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services**, cliquez sur **Définir des URL supplémentaires**, puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion**, tapez une URL au format suivant : `https://<YourCustomFQDN>.f5.com/`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur, l’URL de réponse et l’URL de connexion réels. Pour obtenir ces valeurs, contactez l’[équipe de support technique de F5](https://support.f5.com/csp/knowledge-center/software/BIG-IP?module=BIG-IP%20APM45). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, recherchez **XML de métadonnées de fédération** et **Certificat (en base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

1. Dans la section **Configurer F5**, copiez la ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à F5.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **F5**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.
1. Cliquez sur **Accès conditionnel**.
1. Cliquez sur **Nouvelle stratégie**.
1. Votre application F5 apparaît désormais comme une ressource de la stratégie d’autorité de certification et vous pouvez appliquer n’importe quel accès conditionnel, notamment l’authentification multifacteur, le contrôle d’accès basé sur les appareils ou la stratégie de protection des identités.

## <a name="configure-f5-sso"></a>Configurer l’authentification unique F5

- [Configurer l’authentification unique F5 pour une application basée sur l’en-tête](headerf5-tutorial.md)

- [Configurer l’authentification unique F5 pour une application Kerberos avancée](advance-kerbf5-tutorial.md)

### <a name="configure-f5-single-sign-on-for-kerberos-application"></a>Configurer l’authentification unique F5 pour une application Kerberos

### <a name="guided-configuration"></a>Configuration guidée

1. Ouvrez une nouvelle fenêtre de navigateur web et connectez-vous à votre site d’entreprise F5 (Kerberos) en tant qu’administrateur, puis effectuez les étapes suivantes :

1. Vous devrez importer le certificat de métadonnées dans F5 qui sera utilisé plus tard pendant l’installation.

1. Accédez à **System > Certificate Management > Traffic Certificate Management > SSL Certificate List** (Système > Gestion des certificats > Gestion des certificats de trafic > Liste des certificats SSL). Sélectionnez **Import** (Importer) en haut à droite. Spécifiez un nom de certificat dans le champ **Certificate Name** (sera référencé plus tard dans la configuration). Dans **Certificate Source** (Source du certificat), sélectionnez Upload File (Charger un fichier), spécifiez le certificat téléchargé à partir d’Azure pendant la configuration de l’authentification unique SAML. Cliquez sur **Importer**.

    ![Capture d’écran montrant la page « SSL Certificate/Key Source » avec le nom du certificat (« Certificate Name ») mis en évidence ainsi que l’option « Upload File » et le bouton « Import » sélectionnés.](./media/kerbf5-tutorial/configure01.png) 

1. De plus, vous aurez besoin d’un **certificat SSL pour le nom d’hôte de l’application. Accédez à System > Certificate Management > Traffic Certificate Management > SSL Certificate List** (Système > Gestion des certificats > Gestion des certificats de trafic > Liste des certificats SSL). Sélectionnez **Import** (Importer) en haut à droite. **Import Type** (Type d’importation) aura la valeur **PKCS 12 (IIS)** . Spécifiez un nom de clé dans le champ **Key Name** (sera référencé plus tard dans la configuration), puis le fichier PFX. Spécifiez le mot de passe du fichier PFX dans le champ **Password**. Cliquez sur **Importer**.

    >[!NOTE]
    >Dans l’exemple, le nom de notre application est `Kerbapp.superdemo.live`, nous utilisons un certificat à caractères génériques, et le nom de clé est `WildCard-SuperDemo.live`.

    ![Capture d’écran montrant la page « SSL Certificate/Key Source » avec les valeurs entrées et le bouton « Import » sélectionné.](./media/kerbf5-tutorial/configure02.png) 
 
1. Nous allons utiliser l’expérience guidée pour configurer l’accès à l’application et la fédération Azure AD. Dans F5 BIG-IP, accédez à l’onglet **Main** (Principal), puis sélectionnez **Access > Guided Configuration > Federation > SAML Service Provider** (Accès > Configuration guidée > Fédération > Fournisseur de services SAML). Cliquez sur **Next** (Suivant) et encore sur **Next** (Suivant) pour commencer la configuration.

    ![Capture d’écran montrant la page « Guided Configuration » avec l’icône « Federation » mise en évidence, et le choix « SAML Service Provider » sélectionné.](./media/kerbf5-tutorial/configure03.png) 

    ![Capture d’écran montrant la page « Guided Configuration - SAML Service Provider » avec le bouton « Next » sélectionné.](./media/kerbf5-tutorial/configure04.png)

1. Attribuez un nom à la configuration dans le champ **Configuration Name**. Spécifiez l’ID d’entité dans le champ **Entity ID** (identique à celui que vous avez configuré dans la configuration de l’application Azure AD). Spécifiez le nom d’hôte dans le champ **Host**. Ajoutez une description dans le champ **Description** pour référence. Acceptez les entrées par défaut restantes, puis cliquez sur **Save & Next** (Enregistrer et suivant).

    ![Capture d’écran montrant la section « Service Provider Properties » avec les zones de texte « Host name » et « Description » mises en évidence ainsi que le bouton « Save & Next » sélectionné.](./media/kerbf5-tutorial/configure05.png) 

1. Dans cet exemple, nous créons un serveur virtuel dont l’adresse est 192.168.30.200 et le port 443. Spécifiez l’adresse IP du serveur virtuel dans le champ **Destination Address**. En dessous de **Client SSL Profile** (Profil SSL client), sélectionnez Create new (Créer nouveau). Spécifiez le certificat d’application chargé précédemment (le certificat à caractères génériques de cet exemple) et la clé associée, puis cliquez sur **Save & Next** (Enregistrer et suivant).

    >[!NOTE]
    >Dans cet exemple, notre serveur web interne s’exécute sur le port 80 et nous voulons le publier avec 443.

    ![Capture d’écran montrant la page « Virtual Server Properties » avec la zone de texte « Destination Address » mise en évidence, et le bouton « Save & Next » sélectionné.](./media/kerbf5-tutorial/configure06.png)

1. Sous **Select method to configure your IdP connector** (Sélectionner une méthode pour configurer votre connecteur IdP), spécifiez Metadata (Métadonnées), cliquez sur Choose File (Choisir un fichier), puis chargez le fichier XML de métadonnées téléchargé précédemment à partir d’Azure AD. Attribuez un nom unique au connecteur IDP SAML dans le champ **Name**. Choisissez le certificat de signature de métadonnées (**Metadata Signing Certificate**) qui a été chargé précédemment. Cliquez sur **Save & Next** (Enregistrer et suivant).

    ![Capture d’écran montrant la page « External Identity Provider Connector Settings » avec la zone de texte « Name » mise en évidence, et le bouton « Save & Next » sélectionné.](./media/kerbf5-tutorial/configure07.png)  

1. Sous **Select a Pool** (Sélectionner un pool), spécifiez **Create New** (Créer nouveau) ou sélectionnez un pool qui existe déjà. Conservez les autres valeurs par défaut.    Sous Pool Servers (Serveurs de pool), tapez l’adresse IP sous **IP Address/Node Name** (Adresse IP/Nom du nœud). Spécifiez le **Port**. Cliquez sur **Save & Next** (Enregistrer et suivant).
 
    ![Capture d’écran montrant la page « Pool Properties » avec les zones de texte « IP Address/Node Name » et « Port » mises en évidence, et le bouton « Save & Next » sélectionné.](./media/kerbf5-tutorial/configure08.png)

1. Dans l’écran Single Sign-On Settings (Paramètres d’authentification unique), sélectionnez **Enable Single Sign-On** (Activer l’authentification unique). Sous **Selected Single Sign-On Type** (Type d’authentification unique sélectionné), choisissez **Kerberos**. Remplacez **session.saml.last.Identity** par **session.saml.last.attr.name.Identity** sous **Username Source** (Source du nom d’utilisateur). Cette variable est définie à partir du mappage de revendications dans Azure AD. Sélectionnez **Show Advanced Setting** (Afficher les options avancées). Sous **Kerberos Realm** (Domaine Kerberos), tapez le nom de domaine. Sous **Account Name/ Account Password** (Nom du compte/Mot de passe du compte), spécifiez le compte de délégation APM et le mot de passe. Spécifiez l’adresse IP du contrôleur de domaine dans le champ **KDC**. Cliquez sur **Save & Next** (Enregistrer et suivant).

    ![Capture d’écran montrant la page « Single Sign-On Settings » avec des zones de texte mises en évidence, et le bouton « Save & Next » sélectionné.](./media/kerbf5-tutorial/configure09.png)   

1. Dans le cadre de ce guide, nous allons passer les vérifications de point de terminaison.  Consultez la documentation F5 pour plus d’informations.  Dans l’écran, sélectionnez **Save & Next** (Enregistrer et suivant).

    ![Capture d’écran montrant la page « Endpoint Checks Properties » et le bouton « Save & Next » sélectionné.](./media/kerbf5-tutorial/configure10.png) 

1. Acceptez les valeurs par défaut et cliquez sur **Save & Next** (Enregistrer et suivant). Consultez la documentation F5 pour plus d’informations sur les paramètres de gestion de session SAML.


    ![Capture d’écran montrant la page « Timeout Settings » avec le bouton « Save & Next » sélectionné.](./media/kerbf5-tutorial/configure11.png) 
 
1. Passez en revue l’écran récapitulatif, puis sélectionnez **Deploy** (Déployer) pour configurer BIG-IP.
 
    ![Capture d’écran montrant la page « Your application is ready to be deployed » avec la section « Summary » mise en évidence, et le bouton « Deploy » sélectionné.](./media/kerbf5-tutorial/configure12.png)

1. Une fois l’application configurée, cliquez sur **Finish** (Terminer).

    ![Capture d’écran montrant la page « Your application is deployed » avec le bouton « Finish » sélectionné.](./media/kerbf5-tutorial/configure13.png)

## <a name="advanced-configuration"></a>Configuration avancée

>[!NOTE]
>Pour obtenir des informations de référence, cliquez [ici](https://techdocs.f5.com/kb/en-us/products/big-ip_apm/manuals/product/apm-authentication-single-sign-on-11-5-0/2.html).

### <a name="configuring-an-active-directory-aaa-server"></a>Configuration d’un serveur Active Directory AAA

Vous pouvez configurer un serveur Active Directory AAA dans Access Policy Manager (APM) pour spécifier les contrôleurs de domaine et les informations d’identification dont se servira APM pour authentifier les utilisateurs.

1. Sous l’onglet Main (Principal), cliquez sur **Access Policy > AAA Servers > Active Directory** (Stratégie d’accès > Serveurs AAA > Active Directory). L’écran listant les serveurs Active Directory s’ouvre.

2. Cliquez sur **Créer**. L’écran des propriétés du nouveau serveur s’ouvre.

3. Dans le champ **Name**, tapez un nom unique pour le serveur d’authentification.

4. Dans le champ **Domain Name**, tapez le nom du domaine Windows.

5. Pour le paramètre **Server Connection** (Connexion au serveur), sélectionnez l’une des options suivantes :

   * Sélectionnez **Use Pool** (Utiliser un pool) pour configurer la haute disponibilité pour le serveur AAA.

   * Sélectionnez **Direct** pour configurer le serveur AAA pour une fonctionnalité autonome.

6. Si vous avez sélectionné **Direct**, tapez un nom dans le champ **Domain Controller** (Contrôleur de domaine).

7. Si vous avez sélectionné **Use Pool** (Utiliser un pool), configurez le pool :

   * Tapez un nom dans le champ **Domain Controller Pool Name** (Nom du pool de contrôleurs de domaine).

   * Spécifiez les **contrôleurs de domaine** du pool en tapant l’adresse IP et le nom d’hôte pour chacun d’eux, puis cliquez sur le bouton **Add** (Ajouter).

   * Pour superviser l’intégrité des serveurs AAA, vous pouvez sélectionner un moniteur d’intégrité : seul le moniteur **gateway_icmp** convient dans ce cas ; vous pouvez le sélectionner dans la liste **Server Pool Monitor** (Moniteur de pool de serveurs).

8. Dans le champ **Admin Name** (Nom de l’administrateur), tapez le nom d’un administrateur qui dispose d’autorisations d’administration Active Directory en veillant à respecter la casse. APM utilise les informations contenues dans les champs **Admin Name** (Nom de l’administrateur) et **Admin Password** (Mot de passe de l’administrateur) pour la requête AD. Si Active Directory est configuré pour des requêtes anonymes, vous n’avez pas besoin d’indiquer le nom de l’administrateur. Dans le cas contraire, APM a besoin d’un compte doté de privilèges suffisants pour pouvoir se lier à un serveur Active Directory, récupérer des informations de groupe d’utilisateurs ainsi que les stratégies de mot de passe Active Directory afin de prendre en charge les fonctionnalités liées aux mots de passe. (APM doit extraire les stratégies de mot de passe, par exemple, si vous sélectionnez l’option Prévenir l’utilisateur qu’il doit changer son mot de passe avant qu’il n’expire dans une action de requête AD.) Si vous ne fournissez pas d’informations de compte d’administrateur dans cette configuration, APM utilise le compte d’utilisateur pour récupérer les informations. Ceci fonctionne si le compte d’utilisateur dispose de privilèges suffisants.

9. Dans le champ **Admin Password** (Mot de passe de l’administrateur), tapez le mot de passe d’administrateur associé au nom de domaine.

10. Dans le champ **Verify Admin Password** (Vérifier le mot de passe de l’administrateur), retapez le mot de passe d’administrateur associé au paramètre **Domain Name** (Nom de domaine).

11. Dans le champ **Group Cache Lifetime** (Durée de vie du cache du groupe), tapez le nombre de jours. La durée de vie par défaut est de 30 jours.

12. Dans le champ **Password Security Object Cache Lifetime** (Durée de vie du cache d’objet de sécurité de mot de passe), tapez le nombre de jours. La durée de vie par défaut est de 30 jours.

13. Dans la liste **Kerberos Preauthentication Encryption Type** (Type de chiffrement de la pré-authentification Kerberos), sélectionnez un type de chiffrement. La valeur par défaut est **None** (Aucun). Si vous spécifiez un type de chiffrement, le système BIG-IP inclut des données de pré-authentification Kerberos dans le premier paquet de demande de service d’authentification (AS-REQ).

14. Dans le champ **Timeout**, tapez un délai d’expiration (en secondes) pour le serveur AAA. (Ce paramètre est facultatif.)

15. Cliquez sur **Finished** (Terminé). Le nouveau serveur s’affiche dans la liste. Le nouveau serveur Active Directory est alors ajouté à la liste de serveurs Active Directory.

    ![Capture d’écran montrant les sections « General Properties » et « Configuration ».](./media/kerbf5-tutorial/configure17.png)

### <a name="saml-configuration"></a>Configuration SAML

1. Vous devrez importer le certificat de métadonnées dans F5 qui sera utilisé plus tard pendant l’installation. Accédez à **System > Certificate Management > Traffic Certificate Management > SSL Certificate List** (Système > Gestion des certificats > Gestion des certificats de trafic > Liste des certificats SSL). Sélectionnez **Import** (Importer) en haut à droite.

    ![Capture d’écran montrant la page « Import SSL Certificate/Key Source » avec le bouton « Import » sélectionné.](./media/kerbf5-tutorial/configure18.png)

2. Pour configurer le fournisseur d’identité SAML, **accédez à Access > Federation > SAML: Service Provider > External Idp Connectors** (Accès > Fédération > SAML : Fournisseur de services > Connecteurs IDP externe), puis cliquez sur **Create > From Metadata** (Créer > À partir de métadonnées).

    ![Capture d’écran montrant la page « SAML Service Provider » avec l’option « From Metadata » sélectionnée dans la liste déroulante « Create ».](./media/kerbf5-tutorial/configure19.png)

    ![Capture d’écran montrant la boîte de dialogue « Create New SAML IdP Connector ».](./media/kerbf5-tutorial/configure20.png)

    ![Capture d’écran montrant la fenêtre « Edit SAML IdP Connector » avec l’élément « General Settings » sélectionné.](./media/kerbf5-tutorial/configure21.png)

    ![Capture d’écran montrant la fenêtre « Edit SAML IdP Connector » avec l’élément « Single Sign On Service Settings » sélectionné.](./media/kerbf5-tutorial/configure22.png)

    ![Capture d’écran montrant la fenêtre « Edit SAML IdP Connector » avec l’élément « Security Settings » sélectionné.](./media/kerbf5-tutorial/configure23.png)

    ![Capture d’écran montrant la fenêtre « Edit SAML IdP Connector » avec l’élément « SLO Service Settings » sélectionnée.](./media/kerbf5-tutorial/configure24.png)

1. Pour configurer le fournisseur de services SAML, accédez à **Access > Federation > SAML Service Provider > Local SP Services** (Accès > Fédération > Fournisseur de services SAML > Services SP locaux), puis cliquez sur **Create** (Créer). Fournissez les informations suivantes et cliquez sur **OK**.

    * Type Name (Nom de type) : KerbApp200SAML
    * Entity ID (ID d’entité)* : https://kerb-app.com.cutestat.com
    * SP Name Settings (Paramètres du nom de fournisseur de services)
    * Scheme (Schéma) : https
    * Host (Hôte) : kerbapp200.superdemo.live
    * Description : kerbapp200.superdemo.live

     ![Capture d’écran montrant la fenêtre « Edit SAML SP Service » avec l’élément « General Settings » sélectionné.](./media/kerbf5-tutorial/configure25.png)

     b. Sélectionnez la configuration de fournisseur de services KerbApp200SAML, puis cliquez sur **Bind/UnBind IdP Connectors** (Lier/Dissocier les connecteurs IdP).

     ![Capture d’écran montrant la page « SAML Service Provider - Local SP Services » avec l’option « KerbAPP200 SAML » sélectionnée.](./media/kerbf5-tutorial/configure26.png)

     ![Capture d’écran montrant le bouton « Bind/Unbind IdP Connectors » sélectionné.](./media/kerbf5-tutorial/configure27.png)

     c. Cliquez sur **Add New Row** (Ajouter une nouvelle ligne), sélectionnez le connecteur IdP externe (**External IdP connector**) créé à l’étape précédente, cliquez sur **Update** (Mettre à jour), puis cliquez sur **OK**.

     ![Capture d’écran montrant la fenêtre « Edit SAML IdPs that use this SP » avec le bouton « Add New Row » sélectionné.](./media/kerbf5-tutorial/configure28.png)

1. Pour configurer l’authentification unique Kerberos, accédez à **Access > Single Sign-On > Kerberos** (Accès > Authentification unique > Kerberos), fournissez les informations nécessaires et cliquez sur **Finished** (Terminé).

    >[!Note]
    > Vous devrez créer et spécifier le compte de délégation Kerberos. Consultez la section KCD (voir l’annexe pour obtenir des informations de référence sur les variables).

    * **Username Source** (Source de nom d’utilisateur) : session.saml.last.attr.name.http:\//schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname

    * **User Realm Source** (Source du domaine utilisateur) : session.logon.last.domain

        ![Capture d’écran montrant la page « Single Sign-On - Properties » avec les zones de texte « Username Source » et « User Realm Source » mises en évidence.](./media/kerbf5-tutorial/configure29.png)

1. Pour configurer le profil d’accès, accédez à **Access > Profile/Policies > Access Profile (per session policies)** [Accès > Profil/Stratégies > Profil d’accès (par stratégies de session)], cliquez sur **Create** (Créer), fournissez les informations suivantes et cliquez sur **Finished** (Terminé).

    * Nom : KerbApp200
    * Profil Type (Type de profil) : Tous
    * Profil Scope (Étendue de profil) : Profil
    * Langages : Anglais

        ![Capture d’écran montrant la page « Profiles/Policies - Properties » avec les zones de texte « Name », « Profile Type » et « Languages » mises en évidence.](./media/kerbf5-tutorial/configure30.png)

1. Cliquez sur le nom KerbApp200, fournissez les informations suivantes, puis cliquez sur **Update** (Mettre à jour).

    * Domain Cookie (Cookie du domaine) : superdemo.live
    * SSO Configuration (Configuration SSO) : KerAppSSO_sso

        ![Capture d’écran montrant la page « SSD/Auth Domains » avec la zone de texte « Domain Cookie » et la liste déroulante « SSO Configuration » mises en évidence, et le bouton « Update » sélectionné.](./media/kerbf5-tutorial/configure31.png)

1. Cliquez sur **Stratégie d’accès**, puis choisissez **Modifier la stratégie d’accès** pour le profil « KerbApp200 ».

    ![Capture d’écran montrant la page « Access Policy » avec l’action « Edit Access Policy for Profile KerbApp200 » sélectionnée.](./media/kerbf5-tutorial/configure32.png)

    ![Capture d’écran montrant la page « Access Policy » et la boîte de dialogue « SAML Authentication SP ».](./media/kerbf5-tutorial/configure33.png)

    ![Capture d’écran montrant la page « Access Policy » et la boîte de dialogue « Variable Assign », avec la zone de texte « Assignment » mise en évidence.](./media/kerbf5-tutorial/configure34.png)

    * **session.logon.last.usernameUPN   expr {[mcget {session.saml.last.identity}]}**

    * **session.ad.lastactualdomain  TEXT superdemo.live**

        ![Capture d’écran montrant la page « Access Policy » et la boîte de dialogue « Active Directory », avec la zone de texte « SearchFilter » mise en évidence.](./media/kerbf5-tutorial/configure35.png)

    * **(userPrincipalName=%{session.logon.last.usernameUPN})**

        ![Capture d’écran montrant la page « Access Policy » avec la boîte de dialogue « AD Query - Branch Rules ».](./media/kerbf5-tutorial/configure36.png)

        ![Capture d’écran montrant les zones de texte « Custom Variable » et « Custom Expression » mises en évidence.](./media/kerbf5-tutorial/configure37.png)

    * **session.logon.last.username  expr { "[mcget {session.ad.last.attr.sAMAccountName}]" }**

        ![Capture d’écran montrant la zone de texte « Username from Logon » mise en évidence.](./media/kerbf5-tutorial/configure38.png)

    * **mcget {session.logon.last.username}**
    * **mcget {session.logon.last.password**

1. Pour ajouter un nouveau nœud, accédez à **Local Traffic > Nodes > Node List** (Trafic local > Nœuds > Liste de nœuds), cliquez sur Create (Créer), fournissez les informations suivantes, puis cliquez sur **Finished** (Terminé).

    * Nom : KerbApp200
    * Description : KerbApp200
    * Adresse : 192.168.20.200

        ![Capture d’écran montrant la page « New Node » avec les zones de texte « Name », « Description » et « Address » mises en évidence, et le bouton « Finished » sélectionné.](./media/kerbf5-tutorial/configure39.png)

1. Pour créer un pool, accédez à **Local Traffic > Pools > Pool List** (Trafic local > Pools > Liste de pools), cliquez sur Create (Créer), fournissez les informations suivantes, puis cliquez sur **Finished** (Terminé).

    * Nom : KerbApp200-Pool
    * Description : KerbApp200-Pool
    * Health Monitors (Moniteurs d’intégrité) : http
    * Adresse : 192.168.20.200
    * Service Port (Port du service) : 81

        ![Capture d’écran montrant la page « New Pool » avec des valeurs entrées, et le bouton « Finished » sélectionné.](./media/kerbf5-tutorial/configure40.png)

1. Pour créer un serveur virtuel, accédez à **Local Traffic > Virtual Servers > Virtual Server List > +** (Trafic local > Serveurs virtuels > Liste de serveurs virtuels > +), complétez les informations suivantes, puis cliquez sur **Finished** (Terminé).

    * Nom : KerbApp200
    * Destination Address/Mask (Adresse de destination/Masque) : Host (Host) 192.168.30.200
    * Service Port (Port du service) : Port 443 HTTPS
    * Access Profile (Profil d’accès) : KerbApp200
    * Spécifiez le profil d’accès créé à l’étape précédente

        ![Capture d’écran montrant la page « Virtual Server List » avec les zones de texte « Name », « Destination Address/Mask » et « Service Port » mises en évidence.](./media/kerbf5-tutorial/configure41.png)

        ![Capture d’écran montrant la page « Virtual Server List » avec la liste déroulante « Access Profile » mise en évidence.](./media/kerbf5-tutorial/configure42.png)

### <a name="setting-up-kerberos-delegation"></a>Configuration de la délégation Kerberos 

>[!NOTE]
>Pour obtenir des informations de référence, cliquez [ici](https://www.f5.com/pdf/deployment-guides/kerberos-constrained-delegation-dg.pdf).

*  **Étape 1 :** Créer un compte de délégation

    **Exemple :**
    * Domain Name (Nom du domaine) : **superdemo.live**

    * Sam Account Name (Nom du compte SAM) : **big-ipuser**

    * New-ADUser -Name "APM Delegation Account" -UserPrincipalName host/big-ipuser.superdemo.live@superdemo.live -SamAccountName "big-ipuser" -PasswordNeverExpires $true -Enabled $true -AccountPassword (Read-Host -AsSecureString "Password!1234")

* **Étape 2 :** Définir le SPN (sur le compte de délégation APM)

    **Exemple :**
    * setspn –A **host/big-ipuser.superdemo.live** big-ipuser

* **Étape 3 :** La délégation SPN (pour le compte App Service) configure la délégation appropriée pour le compte de délégation F5.
    Dans l’exemple ci-dessous, le compte de délégation APM est configuré pour KCD pour l’application FRP-App1.superdemo. live.

    ![Configuration F5 (Kerberos)](./media/kerbf5-tutorial/configure43.png)

* Fournissez les détails tels que mentionnées dans le document de référence dans [cette page](https://techdocs.f5.com/kb/en-us/products/big-ip_apm/manuals/product/apm-authentication-single-sign-on-11-5-0/2.html).

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

- [Configurer l’authentification unique F5 pour une application basée sur l’en-tête](headerf5-tutorial.md)

- [Configurer l’authentification unique F5 pour une application Kerberos avancée](advance-kerbf5-tutorial.md)