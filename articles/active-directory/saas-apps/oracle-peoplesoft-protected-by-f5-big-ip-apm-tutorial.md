---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Oracle PeopleSoft - Protected by F5 BIG-IP APM | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique (SSO) entre Azure Active Directory et Oracle PeopleSoft - Protected by F5 BIG-IP APM.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/14/2020
ms.author: jeedes
ms.openlocfilehash: 3b7c8e024ac8361c08cc41195531a114bb12fcb4
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92522289"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-oracle-peoplesoft---protected-by-f5-big-ip-apm"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Oracle PeopleSoft - Protected by F5 BIG-IP APM

Dans ce tutoriel, vous allez découvrir comment intégrer Oracle PeopleSoft - Protected by F5 BIG-IP APM à Azure Active Directory (Azure AD). Quand vous intégrez Oracle PeopleSoft - Protected by F5 BIG-IP APM à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Oracle PeopleSoft - Protected by F5 BIG-IP APM.
* Autoriser les utilisateurs à se connecter automatiquement à Oracle PeopleSoft - Protected by F5 BIG-IP APM avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.
* Ce tutoriel décrit les instructions pour Oracle PeopleSoft ELM.

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

1. Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
1. Un abonnement compatible avec l’authentification unique (SSO) Oracle PeopleSoft - Protected by F5 BIG-IP APM.

1. Le déploiement de la solution conjointe nécessite la licence suivante :

    1. F5 BIG-IP® Best Bundle (ou) 
    2. Licence autonome F5 BIG-IP Access Policy Manager™ (APM) 
    3. Licence de composant additionnel F5 BIG-IP Access Policy Manager™ (APM) sur une instance existante de BIG-IP F5 BIG-IP® Local Traffic Manager™ (LTM).
    4. Outre la licence ci-dessus, il est possible de bénéficier d’une licence pour le système F5 avec : 
        * Un abonnement au filtrage d’URL pour utiliser la base de données de catégories d’URL 
        * Un abonnement à F5 IP Intelligence pour détecter et bloquer les attaquants connus et le trafic malveillant 
        * Un module de sécurité matériel (HSM) réseau pour protéger et gérer les clés numériques pour une authentification forte
1. Un système F5 BIG-IP provisionné avec les modules APM (LTM est facultatif) 
1. Bien que facultatif, il est vivement recommandé de déployer les systèmes F5 dans un [groupe d’appareils de synchronisation/basculement](https://techdocs.f5.com/kb/en-us/products/big-ip_ltm/manuals/product/big-ip-device-service-clustering-administration-14-1-0.html) (S/F DG), qui comprend la paire en veille active, avec une adresse IP flottante pour la haute disponibilité. Il est possible de bénéficier d’une redondance d’interface plus poussée en utilisant le protocole LACP (Link Aggregation Control Protocol). LACP gère les interfaces physiques connectées comme une interface virtuelle unique (groupe agrégé) et détecte les échecs d’interface au sein du groupe.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Oracle PeopleSoft - Protected by F5 BIG-IP APM prend en charge l’authentification unique initiée par **le fournisseur de services et le fournisseur d’identité**

## <a name="adding-oracle-peoplesoft---protected-by-f5-big-ip-apm-from-the-gallery"></a>Ajout d’Oracle PeopleSoft - Protected by F5 BIG-IP APM à partir de la galerie

Pour configurer son intégration à Azure AD, vous devez ajouter Oracle PeopleSoft - Protected by F5 BIG-IP APM à votre liste d’applications SaaS gérées, depuis la galerie.

1. Connectez-vous au portail Azure avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Oracle PeopleSoft - Protected by F5 BIG-IP APM** dans la zone de recherche.
1. Sélectionnez **Oracle PeopleSoft - Protected by F5 BIG-IP APM** à partir du volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-sso-for-oracle-peoplesoft---protected-by-f5-big-ip-apm"></a>Configuration et test de l’authentification unique Azure AD pour Oracle PeopleSoft - Protected by F5 BIG-IP APM

Configurez et testez l’authentification unique Azure AD avec Oracle PeopleSoft - Protected by F5 BIG-IP APM à l’aide d’un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Oracle PeopleSoft - Protected by F5 BIG-IP APM associé.

Pour configurer et tester l’authentification unique Azure AD avec Oracle PeopleSoft - Protected by F5 BIG-IP APM, effectuez les étapes suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Oracle PeopleSoft - Protected by F5 BIG-IP APM](#configure-oracle-peoplesoft-protected-by-f5-big-ip-apm-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test Oracle PeopleSoft - Protected by F5 BIG-IP APM](#create-oracle-peoplesoft-protected-by-f5-big-ip-apm-test-user)** pour disposer, dans Oracle PeopleSoft - Protected by F5 BIG-IP APM, d’un équivalent de B. Simon lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le portail Azure, accédez à la page d’intégration de l’application **Oracle PeopleSoft - Protected by F5 BIG-IP APM** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode Initié par le **fournisseur d’identité** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **Identificateur** , tapez une URL au format suivant : `https://<FQDN>.peoplesoft.f5.com`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://<FQDN>.peoplesoft.f5.com/saml/sp/profile/post/acs`

    c. Dans la zone de texte **URL de déconnexion** , tapez une URL au format suivant : `https://<FQDN>.peoplesoft.f5.com/saml/sp/profile/redirect/slr`

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://<FQDN>.peoplesoft.f5.com/`

    > [!NOTE]
    >Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec les valeurs réelles de l’URL de connexion, de l’identificateur, de l’URL de réponse et de l’URL de déconnexion. Pour connaître les valeurs, contactez [l’équipe du support technique d’Oracle PeopleSoft - Protected by F5 BIG-IP APM](https://support.f5.com). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. L’application Oracle PeopleSoft - Protected by F5 BIG-IP APM attend les assertions SAML dans un certain format, ce qui oblige à ajouter des mappages d’attributs personnalisés à la configuration des attributs du jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut.

    ![image](common/default-attributes.png)

1. En plus de ce qui précède, l’application Oracle PeopleSoft - Protected by F5 BIG-IP APM s’attend à ce que quelques attributs supplémentaires, indiqués ci-dessous, soient repassés dans la réponse SAML. Ces attributs sont également préremplis, mais vous pouvez les examiner pour voir s’ils répondent à vos besoins.
    
    | Nom |  Attribut source|
    | ------------------ | --------- |
    | EMPLID | user.employeeid |

1. Dans la page **Configurer l’authentification unique avec SAML** , à la section **Certificat de signature SAML** , téléchargez le fichier **XML des métadonnées de fédération** et le **Certificat (Base64)** , puis enregistrez-les sur votre ordinateur.

    ![Lien Téléchargement de certificat](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/both-certificate.png)

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en accordant l’accès à Oracle PeopleSoft - Protected by F5 BIG-IP APM.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Oracle PeopleSoft - Protected by F5 BIG-IP APM** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .
1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .
1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez qu’un rôle soit attribué aux utilisateurs, vous pouvez le sélectionner dans la liste déroulante **Sélectionner un rôle** . Si aucun rôle n’a été configuré pour cette application, vous voyez le rôle « Accès par défaut » sélectionné.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-oracle-peoplesoft-protected-by-f5-big-ip-apm-sso"></a>Configuration de l’authentification unique Oracle PeopleSoft-Protected by F5 BIG-IP APM

### <a name="f5-saml-sp-configuration"></a>Configuration du fournisseur de services SAML F5

Importez dans F5 le certificat de métadonnées qui sera utilisé plus tard pendant la configuration. Accédez à **System > Certificate Management > Traffic Certificate Management > SSL Certificate List** (Système > Gestion des certificats > Gestion des certificats de trafic > Liste des certificats SSL). Sélectionnez **Import** (Importer) en haut à droite.

![Configuration du fournisseur de services SAML F5](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/sp-configuration.png)

#### <a name="setup-the-saml-idp-connector"></a>Configurer le connecteur du fournisseur d’identité SAML 

1. Accédez à **Access > Federation > SAML: Service Provider > External Idp Connectors** (Accès > Fédération > SAML : Fournisseur de services > Connecteurs IDP externe), puis cliquez sur **Create > From Metadata** (Créer > À partir de métadonnées).

    ![Connecteur IDP SAML F5](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/saml-idp-connector.png)

1. Dans la page suivante, cliquez sur **Browse** (Parcourir) pour charger le fichier xml.

1. Attribuez un nom de fournisseur d’identité valide dans la zone de texte **Identity Provider Name** (Nom du fournisseur d’identité), puis cliquez sur **OK** .

    ![nouveau connecteur IDP SAML](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/new-saml-idp.png)

1. Effectuez les étapes demandées dans l’onglet **Security Settings** (Paramètres de sécurité), puis cliquez sur **OK** .

    ![modifier le connecteur IDP SAML](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/edit-saml-idp.png)

#### <a name="setup-the-saml-sp"></a>Configurer le fournisseur de services SAML

1. Accédez à **Access > Federation > SAML Service Provider > Local SP Services** (Accès > Fédération > Fournisseur de services SAML > Services SP locaux), puis cliquez sur **Create** (Créer). Fournissez les informations suivantes et cliquez sur **OK** .

    * Nom : `<Name>`
    * Entity ID (ID d’entité) : `https://<FQDN>.peoplesoft.f5.com`
    * SP Name Settings (Paramètres du nom de fournisseur de services)
        * Scheme (Schéma) : `https`
        * Hôte : `<FQDN>.peoplesoft.f5.com`
        * Description : `<Description>`

    ![nouveaux services SP SAML](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/new-saml-sp-service.png)

1. Sélectionnez la configuration de fournisseur de services PeopleSoftAppSSO, puis cliquez sur **Bind/UnBind IdP Connectors** (Lier/Dissocier les connecteurs IdP).
Cliquez sur **Add New Row** (Ajouter une nouvelle ligne), sélectionnez le connecteur IdP externe ( **External IdP connector** ) créé à l’étape précédente, cliquez sur **Update** (Mettre à jour), puis cliquez sur **OK** .

    ![créer des services SP SAML](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/edit-saml-idp-use-sp.png)

## <a name="configuring-application"></a>Configuration de l’application

### <a name="create-a-new-pool"></a>Créer un pool
1. Accédez à **Local Traffic > Pools > Pool List** (Trafic local > Pools > Liste de pools), cliquez sur **Create** (Créer), fournissez les informations suivantes, puis cliquez sur **Finished** (Terminé).

    * Nom : `<Name>`
    * Description : `<Description>`
    * Health Monitors (Analyseurs d’intégrité) : `http`
    * Address : `<Address>`
    * Service Port (Port du service) : `<Service Port>`

    ![création d’un pool](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/create-pool.png)


### <a name="create-a-new-client-ssl-profile"></a>Créer un profil SSL client

Accédez à **Local Traffic > Profiles > SSL > Client > +** (Trafic local > Profils > SSL > Client > +), fournissez les informations suivantes, puis cliquez sur **Finished** (Terminé).

* Nom : `<Name>`
* Certificat : `<Certificate>`
* Clé :`<Key>`

![nouveau profil SSL client](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/client-ssl-profile.png)

### <a name="create-a-new-virtual-server"></a>Créer un serveur virtuel

1. Accédez à **Local Traffic > Virtual Servers > Virtual Server List > +** (Trafic local > Serveurs virtuels > Liste de serveurs virtuels > +), complétez les informations suivantes, puis cliquez sur **Finished** (Terminé).
    * Nom : `<Name>`
    * Destination Address/Mask (Adresse de destination/Masque) : `<Address>`
    * Service Port (Port du service) : Port 443 HTTPS
    * HTTP Profile (Client) : http

    ![Créer un serveur virtuel](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/virtual-server-list.png)

1. Renseignez les valeurs suivantes dans la page ci-dessous :

    * SSL Profile (Client) : `<SSL Profile>`
    * Source Address Translation (Traduction d’adresses sources) : Auto Map
    * Access Profile (Profil d’accès) : `<Access Profile>`
    * Default Pool (Pool par défaut) : `<Pool>`


    ![Créer un pool PeopleSoft de serveur virtuel](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/virtual-server-people-soft.png)

## <a name="setting-up-peoplesoft-application-to-support-f5-big-ip-apm-as-the-single-sign-on-solution"></a>Configuration de l’application PeopleSoft pour la prise en charge de F5 Big-IP APM en tant que solution d’authentification unique

>[!Note]
> Référence https://docs.oracle.com/cd/E12530_01/oam.1014/e10356/people.htm

1. Connectez-vous à la console PeopleSoft `https://<FQDN>.peoplesoft.f5.com/:8000/psp/ps/?cmd=start` à l’aide des informations d’identification d’administrateur (par exemple : PS/PS)

    ![Libre-service Gestionnaire](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/people-soft-console.png)

1. Dans l’application PeopleSoft, créez **OAMPSFT** en tant que nouveau profil utilisateur, et associez un rôle de sécurité faible, tel que **PeopleSoft User** (Utilisateur PeopleSoft).
Accédez à **Peopletools > Security > User Profiles > User Profiles** (Peopletools > Sécurité > Profils utilisateur > Profils utilisateur) pour créer un profil utilisateur, par exemple : **OAMPSFT** et ajoutez **PeopleSoft User** (Utilisateur PeopleSoft).

    ![Utilisateur PeopleSoft](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/user-profile.png)

1. Accédez au profil web et entrez **OAMPSFT** en tant que **user ID** (ID utilisateur) pour l’accès public.

    ![Profils utilisateur](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/web-profile-configuration.png)

1. À partir de **PeopleTools Application Designer** (Concepteur d’applications PeopleTools), ouvrez l’enregistrement **FUNCLIB_LDAP** .

    ![Configuration du profil web](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/funclib.png)

1. Mettez à jour l’en-tête utilisateur avec **PS_SSO_UID** pour la fonction **OAMSSO_AUTHENTICATION** .
Dans la fonction **getWWWAuthConfig()** , remplacez la valeur affectée à **&defaultUserId** par **OAMPSFT** que nous avons défini dans le profil web. Enregistrez la définition Record.

    ![OAMSSO_AUTHENTICATION](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/record.png)

1. Accédez à la page **Signon PeopleCode** (PeopleTools, Security, Security Objects, Signon PeopleCode) (PeopleTools, Sécurité, Objets de sécurité, Authentification PeopleCode) et activez la fonction **OAMSSO_AUTHENTICATION** , qui est l’authentification PeopleCode pour l’authentification unique Oracle Access Manager.

    ![OAMSSO_AUTHENTICATION ](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/sign-on-people-soft.png)

## <a name="setting-up-f5-big-ip-apm-to-populate-ps_sso_uid-http-header-with-the-peoplesoft-user-id"></a>Configuration de F5 BIG-IP APM pour renseigner l’en-tête HTTP « PS_SSO_UID » avec l’ID utilisateur PeopleSoft

### <a name="configuring-per-request-policy"></a>Configuration de la stratégie par demande
1. Accédez à **Access > Profile/Policies > Per-Request Policies (Accès > Profil/Stratégies > Stratégie par demande)** , cliquez sur **Create** (Créer), fournissez les informations suivantes et cliquez sur **Finished** (Terminé).

    * Nom : `<Name>`
    * Profil Type (Type de profil) : Tous
    * Languages (Langues) : `<Language>`

    ![Configuration de la stratégie par demande ](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/per-request.png)

1. Cliquez sur **Edit** Per-Request Policy (Modifier la stratégie par demande) `<Name>` ![modifier la stratégie par demande PeopleSoftSSO ](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/people-soft-sso.png)

    `Header Name: <Header Name>`   
    `Header Value: <Header Value>`

### <a name="assign-per-request-policy-to-the-virtual-server"></a>Affecter une stratégie par demande au serveur virtuel

Accédez à **Local Traffic > Virtual Servers > Virtual Server List > PeopleSoftApp** (Trafic local > Serveurs virtuels > Liste des serveurs virtuels > PeopleSoftApp). Spécifiez `<Name>` en tant que stratégie par demande

![PeopleSoftSSO comme stratégie par demande ](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/people-soft-sso-1.png)

## <a name="setting-up-f5-big-ip-apm-to-support-single-logout-from-peoplesoft-application"></a>Configuration de F5 Big-IP APM pour prendre en charge la déconnexion unique à partir de l’application PeopleSoft

Si vous souhaitez ajouter la prise en charge de la déconnexion unique pour tous les utilisateurs PeopleSoft, suivez ces étapes :

1. Déterminer l’URL de déconnexion appropriée pour le portail PeopleSoft
    * Afin de déterminer l’adresse que l’application PeopleSoft utilise pour mettre fin à une session utilisateur, vous devez ouvrir le portail à l’aide d’un navigateur web et activer les outils de débogage du navigateur, comme illustré dans l’exemple ci-dessous :

        ![URL de déconnexion du portail PeopleSoft ](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/sign-out.png)

    * Recherchez l’élément contenant l’ID `PT_LOGOUT_MENU` et enregistrez le chemin de l’URL avec les paramètres de la requête. Dans notre exemple, nous obtenons la valeur suivante : `/psp/ps/?cmd=logout`

1. Créer LTM iRule qui redirigera l’utilisateur vers l’URL de déconnexion APM : `/my.logout.php3`

    * Accédez à **Local Traffic > iRule** (Trafic local > iRule), cliquez sur **Create** (Créer), fournissez les informations suivantes, puis cliquez sur **Finished** (Terminé).

        Nom : `<Name>`  
        Définition :  
                    _when HTTP_REQUEST { switch -glob -- [HTTP::uri] { `/psp/ps/?cmd=logout` { HTTP::redirect `/my.logout.php3` } } }_

1. Affecter au serveur virtuel la règle iRule créée

    * Accédez à **Local Traffic > Virtual Servers > Virtual Server List > PeopleSoftApp > Resources** (Trafic local > Serveurs virtuels > Liste des serveurs virtuels > PeopleSoftApp > Ressources). Cliquez sur le bouton **Manage...** (Gérer) :   

    * Spécifiez `<Name>` comme règle iRule activée, puis cliquez sur **Finished** (Terminé).

        ![_iRule_PeopleSoftApp ](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/irule-people-soft.png)

    * Donnez à la zone de texte **Name** la valeur `<Name>`. 

        ![Règle _iRule_PeopleSoftApp terminée](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/common-irule.png)


### <a name="create-oracle-peoplesoft-protected-by-f5-big-ip-apm-test-user"></a>Créer l’utilisateur test Oracle PeopleSoft-Protected by F5 BIG-IP APM

Dans cette section, vous allez créer un utilisateur appelé B. Simon dans Oracle PeopleSoft-Protected by F5 BIG-IP APM. Collaborez avec [l’équipe du support technique d’Oracle PeopleSoft-Protected by F5 BIG-IP APM](https://support.f5.com) pour ajouter des utilisateurs à la plateforme Oracle PeopleSoft-Protected by F5 BIG-IP APM. Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester votre configuration de l’authentification unique Azure AD avec les options suivantes. 

#### <a name="sp-initiated"></a>Lancée par le fournisseur de services :

* Cliquez sur **Tester cette application** dans le portail Azure. Cette opération redirige vers l’URL de connexion Oracle PeopleSoft-Protected by F5 BIG-IP APM, où vous pouvez lancer le processus de connexion.  

* Accédez directement à l’URL de connexion Oracle PeopleSoft-Protected by F5 BIG-IP APM pour lancer le processus de connexion à partir de là.

#### <a name="idp-initiated"></a>Lancée par le fournisseur d’identité :

* Cliquez sur **Tester cette application** dans le portail Azure : vous devez être connecté automatiquement à l’instance d’Oracle PeopleSoft-Protected by F5 BIG-IP APM pour laquelle vous avez configuré l’authentification unique. 

Vous pouvez aussi utiliser le panneau d’accès Microsoft pour tester l’application dans n’importe quel mode. Si, quand vous cliquez sur la vignette Oracle PeopleSoft-Protected by F5 BIG-IP APM dans le volet d’accès, le mode Fournisseur de services est configuré, vous êtes redirigé vers la page de connexion de l’application pour lancer le processus de connexion ; s’il s’agit du mode Fournisseur d’identité, vous êtes automatiquement connecté à l’instance d’Oracle PeopleSoft-Protected by F5 BIG-IP APM pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).


## <a name="next-steps"></a>Étapes suivantes

Après avoir configuré Oracle PeopleSoft - Protected by F5 BIG-IP APM, vous pouvez appliquer le contrôle de session qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).