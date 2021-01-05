---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Alibaba Cloud Service (Role-based SSO) | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Alibaba Cloud Service (Role-based SSO).
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/08/2020
ms.author: jeedes
ms.openlocfilehash: c9db5ab471f743ffbf3ae4e140ad4cb32d0a3d6c
ms.sourcegitcommit: d79513b2589a62c52bddd9c7bd0b4d6498805dbe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/18/2020
ms.locfileid: "97670190"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-alibaba-cloud-service-role-based-sso"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Alibaba Cloud Service (Role-based SSO)

Dans ce tutoriel, vous allez apprendre à intégrer Alibaba Cloud Service (Role-based SSO) à Azure Active Directory (Azure AD). Quand vous intégrez Alibaba Cloud Service (Role-based SSO) à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Alibaba Cloud Service (Role-based SSO).
* Permettre à vos utilisateurs de se connecter automatiquement à Alibaba Cloud Service (Role-based SSO) avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Alibaba Cloud Service (Role-based SSO) pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Alibaba Cloud Service (Role-based SSO) prend en charge l’authentification unique lancée par le **fournisseur d’identité**

## <a name="adding-alibaba-cloud-service-role-based-sso-from-the-gallery"></a>Ajout d’Alibaba Cloud Service (Role-based SSO) à partir de la galerie

Pour configurer l’intégration d’Alibaba Cloud Service (Role-based SSO) à Azure AD, vous devez ajouter Alibaba Cloud Service (Role-based SSO), disponible dans la galerie, à votre liste d’applications SaaS gérées.

1. Connectez-vous au portail Azure avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **Alibaba Cloud Service (Role-based SSO)** dans la zone de recherche.
1. Sélectionnez **Alibaba Cloud Service (Role-based SSO)** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.
5. Sur la page **Alibaba Cloud Service (Role-based SSO)** , dans le volet de navigation de gauche, cliquez sur **Propriétés**, puis copiez l’**ID d’objet** et enregistrez-le sur votre ordinateur en vue d’une utilisation ultérieure.

    ![Configuration des propriétés](./media/alibaba-cloud-service-role-based-sso-tutorial/Properties.png)


## <a name="configure-and-test-azure-ad-sso-for-alibaba-cloud-service-role-based-sso"></a>Configurer et tester l’authentification unique Azure AD pour Alibaba Cloud Service (Role-based SSO)

Configurez et testez l’authentification unique Azure AD avec Alibaba Cloud Service (Role-based SSO) pour un utilisateur de test appelé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Alibaba Cloud Service (Role-based SSO) associé.

Pour configurer et tester l’authentification unique Azure AD avec Alibaba Cloud Service (Role-based SSO), effectuez les étapes suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
2. **[Configurer l’authentification unique en fonction du rôle dans Alibaba Cloud Service](#configure-role-based-single-sign-on-in-alibaba-cloud-service)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Configurer l’authentification unique Alibaba Cloud Service (Role-based SSO)](#configure-alibaba-cloud-service-role-based-sso-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test Alibaba Cloud Service (Role-based SSO)](#create-alibaba-cloud-service-role-based-sso-test-user)** pour avoir un équivalent de Britta Simon dans Alibaba Cloud Service (Role-based SSO) lié à la représentation Azure AD associée.
3. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le portail Azure, accédez à la page d’intégration de l’application **Alibaba Cloud Service (Role-based SSO)** , recherchez la section **Gérer** et sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base**, si vous disposez d’un **fichier de métadonnées du fournisseur de services**, suivez les étapes ci-dessous :

    a. Cliquez sur **Charger un fichier de métadonnées**.

    b. Cliquez sur le **logo du dossier** pour sélectionner le fichier de métadonnées, puis cliquez sur **Charger**.


    >[!NOTE]
    >1. Pour le site international Alibaba Cloud, téléchargez les métadonnées du fournisseur de services à partir de [ce lien](https://signin.alibabacloud.com/saml-role/sp-metadata.xml).
    > 1. Pour le site Alibaba Cloud Service (CN), téléchargez les métadonnées du fournisseur de services à partir de [ce lien](https://signin.aliyun.com/saml-role/sp-metadata.xml).

    c. Une fois le fichier de métadonnées chargé, les valeurs **Identificateur** et **URL de réponse** sont automatiquement renseignées dans la zone de texte de la section Alibaba Cloud Service (Role-based SSO) :

    > [!Note]
    > Si les valeurs **Identificateur** et **URL de réponse** ne sont pas automatiquement renseignées, renseignez-les manuellement en fonction de vos besoins.

1. Alibaba Cloud Service (Role-based SSO) nécessite que les rôles soient configurés dans Azure AD. La revendication de rôle est préconfigurée. Vous n’avez donc pas à la configurer, mais vous devez toujours créer les rôles dans Azure AD en suivant les instructions de cet [article](https://docs.microsoft.com/azure/active-directory/develop/howto-add-app-roles-in-azure-ad-apps#app-roles-ui).

1. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, recherchez **XML de métadonnées de fédération** et sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

1. Dans la section **Configurer Alibaba Cloud Service (Role-based SSO)** , copiez l’URL ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Alibaba Cloud Service (Role-based SSO).

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **Alibaba Cloud Service (Role-based SSO)** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.
1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.
1. Sous l’onglet **Utilisateurs et groupes**, sélectionnez u2 dans la liste des utilisateurs, puis cliquez sur **Sélectionner**. Cliquez ensuite sur **Affecter**.

    ![Affecter l’utilisateur de test Azure AD 1](./media/alibaba-cloud-service-role-based-sso-tutorial/test01.png)

1. Affichez le rôle affecté et testez Alibaba Cloud Service (Role-based SSO).

    ![Affecter l’utilisateur de test Azure AD 2](./media/alibaba-cloud-service-role-based-sso-tutorial/test02.png)

    >[!NOTE]
    >Une fois l’utilisateur (u2) affecté, le rôle créé est automatiquement associé à l’utilisateur. Si vous avez créé plusieurs rôles, vous devez associer le rôle approprié à l’utilisateur en fonction des besoins. Si vous souhaitez implémenter l’authentification unique en fonction du rôle d’Azure AD pour plusieurs comptes Alibaba Cloud, répétez les étapes précédentes.

## <a name="configure-role-based-single-sign-on-in-alibaba-cloud-service"></a>Configurer l’authentification unique en fonction du rôle dans Alibaba Cloud Service

1. Connectez-vous à la [RAM console](https://account.alibabacloud.com/login/login.htm?oauth_callback=https%3A%2F%2Fram.console.aliyun.com%2F%3Fspm%3Da2c63.p38356.879954.8.7d904e167h6Yg9) (console RAM) d’Alibaba Cloud en utilisant le compte Account1.

2. Dans le volet de navigation de gauche, sélectionnez **SSO**.

3. Sous l’onglet **Role-based SSO** (SSO en fonction du rôle), cliquez sur **Create IdP** (Créer un IdP).

4. Sur la page affichée, entrez `AAD` dans le champ Nom de l’IdP, saisissez une description dans le champ **Note**, cliquez sur **Upload** (Charger) pour charger le fichier de métadonnées de fédération que vous avez téléchargé précédemment, puis cliquez sur **OK**.

5. Une fois le fournisseur d’identité correctement créé, cliquez sur **Create RAM Role** (Créer un rôle de RAM).

6. Dans le champ **RAM Role Name** (Nom du rôle de RAM), entrez `AADrole`, sélectionnez `AAD` dans la liste déroulante **Select IdP** (Sélectionner un IdP), puis cliquez sur OK.

    >[!NOTE]
    >Vous pouvez accorder une autorisation au rôle en fonction des besoins. Après avoir créé le fournisseur d’identité et le rôle correspondant, nous vous recommandons d’enregistrer les ARN du fournisseur d’identité et du rôle en vue d’une utilisation ultérieure. Vous pouvez obtenir les ARNs sur la page d’informations sur le fournisseur d’identité et la page d’informations sur le rôle.

7. Associez le rôle de RAM d’Alibaba Cloud (AADrole) à l’utilisateur Azure AD (u2) : Pour associer le rôle de RAM à l’utilisateur Azure AD, vous devez créer un rôle dans Azure AD en procédant comme suit :

    a. Connectez-vous à [Microsoft Graph Explorer](https://developer.microsoft.com/graph/graph-explorer).

    b. Cliquez sur **Modify permissions** (Modifier les autorisations) pour obtenir les autorisations requises pour la création d’un rôle.

    ![Configuration de Graph 1](./media/alibaba-cloud-service-role-based-sso-tutorial/graph01.png)

    c. Sélectionnez les autorisations suivantes dans la liste, puis cliquez sur **Modify permissions** (Modifier les autorisations) , comme illustré dans la figure suivante.

    ![Configuration de Graph 2](./media/alibaba-cloud-service-role-based-sso-tutorial/graph02.png)

    >[!NOTE]
    >Une fois les autorisations accordées, reconnectez-vous à l’Afficheur Graph.

    d. Dans la page de l’Afficheur Graph, sélectionnez **GET** (OBTENIR) dans la première liste déroulante, puis **beta** (bêta) dans la deuxième. Ensuite, entrez `https://graph.microsoft.com/beta/servicePrincipals` dans le champ en regard des listes déroulantes, puis cliquez sur **Run Query** (Exécuter la requête).

    ![Configuration de Graph 3](./media/alibaba-cloud-service-role-based-sso-tutorial/graph03.png)

    >[!NOTE]
    >Si vous utilisez plusieurs répertoires, vous pouvez entrer `https://graph.microsoft.com/beta/contoso.com/servicePrincipals` dans le champ de la requête.

    e. Dans la section **Response Preview** (Aperçu de la réponse), extrayez la propriété appRoles à partir de « Service Principal » (Principal du service) en vue d’un usage ultérieur.

    ![Configuration de Graph 4](./media/alibaba-cloud-service-role-based-sso-tutorial/graph05.png)

    >[!NOTE]
    >Vous pouvez localiser la propriété appRoles en entrant `https://graph.microsoft.com/beta/servicePrincipals/<objectID>` dans le champ de la requête. Notez que `objectID` est l’ID d’objet que vous avez copié à partir de la page **Propriétés** d’Azure AD.

    f. Revenez à l’Afficheur Graph, modifiez la méthode **GET** (OBTENIR) en **PATCH** (CORRIGER), collez le contenu suivant dans la section **Request Body** (Corps de la demande), puis cliquez sur **Run Query** (Exécuter la requête) :
    ```
    { 
      "appRoles": [
        { 
          "allowedMemberTypes":[
            "User"
          ],
          "description": "msiam_access",
          "displayName": "msiam_access",
          "id": "41be2db8-48d9-4277-8e86-f6d22d35****",
          "isEnabled": true,
          "origin": "Application",
          "value": null
        },
        { "allowedMemberTypes": [
            "User"
        ],
        "description": "Admin,AzureADProd",
        "displayName": "Admin,AzureADProd",
        "id": "68adae10-8b6b-47e6-9142-6476078cdbce",
        "isEnabled": true,
        "origin": "ServicePrincipal",
        "value": "acs:ram::187125022722****:role/aadrole,acs:ram::187125022722****:saml-provider/AAD"
        }
      ]
    }
    ```
    > [!NOTE]
    > `value` correspond aux ARNs du fournisseur d’identité et du rôle que vous avez créés dans la console RAM. Ici, vous pouvez ajouter plusieurs rôles en fonction des besoins. Azure AD enverra la valeur de ces rôles en tant que valeur de revendication dans la réponse SAML. Cependant, vous ne pouvez ajouter de nouveaux rôles qu’après la partie `msiam_access` de l’opération de correction. Pour faciliter le processus de création, nous vous recommandons d’utiliser un générateur d’ID tels que Générateur de GUID pour générer des ID en temps réel.

    g. Une fois le « Service Principal » (principal du service) corrigé avec le rôle requis, associez le rôle à l’utilisateur Azure AD (u2) en suivant la procédure décrite dans la section **Affecter l’utilisateur de test Azure AD** du didacticiel.

### <a name="configure-alibaba-cloud-service-role-based-sso-sso"></a>Configurer l’authentification unique Alibaba Cloud Service (Role-based SSO)

Pour configurer l’authentification unique côté **Alibaba Cloud Service (Role-based SSO)** , vous devez envoyer le fichier **XML des métadonnées de fédération** téléchargé et les URL copiées correspondantes du portail Azure à l’[équipe du support technique Alibaba Cloud Service (Role-based SSO)](https://www.aliyun.com/service/). Celles-ci configurent ensuite ce paramètre pour que la connexion SSO SAML soit définie correctement des deux côtés.

### <a name="create-alibaba-cloud-service-role-based-sso-test-user"></a>Créer un utilisateur de test Alibaba Cloud Service (Role-based SSO)

Dans cette section, vous allez créer un utilisateur nommé Britta Simon dans Alibaba Cloud Service (Role-based SSO). Collaborez avec[l’équipe du support technique d’Alibaba Cloud Service (Role-based SSO)](https://www.aliyun.com/service/) pour ajouter les utilisateurs à la plateforme Alibaba Cloud Service (Role-based SSO). Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Une fois les configurations ci-dessus terminées, testez Alibaba Cloud Service (Role-based SSO) en procédant comme suit :

1. Dans le portail Azure, accédez à la page **Alibaba Cloud Service (Role-based SSO)** , sélectionnez **Single sign-on** (Authentification unique), puis cliquez sur **Test** (Tester).

    ![Configuration de test 1](./media/alibaba-cloud-service-role-based-sso-tutorial/test03.png)

2. Cliquez sur **Se connecter en tant qu’utilisateur actuel**.

    ![Configuration de test 2](./media/alibaba-cloud-service-role-based-sso-tutorial/test04.png)

3. Dans la page de sélection de compte, choisissez u2.

    ![Configuration de test 3](./media/alibaba-cloud-service-role-based-sso-tutorial/test05.png)

4. La page suivante s’affiche, indiquant que l’authentification unique en fonction du rôle a réussi.

    ![Configuration de test 4](./media/alibaba-cloud-service-role-based-sso-tutorial/test06.png)

## <a name="next-steps"></a>Étapes suivantes

Après avoir configuré Alibaba Cloud Service (Role-based SSO), vous pouvez appliquer un contrôle de session, qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).