---
title: 'Tutoriel : Ajouter des fournisseurs d’identité à vos applications'
titleSuffix: Azure AD B2C
description: Ce tutoriel explique comment ajouter des fournisseurs d’identité à vos applications dans Azure Active Directory B2C à l’aide du portail Azure.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: tutorial
ms.date: 07/30/2020
ms.author: mimart
ms.subservice: B2C
ms.custom: fasttrack-edit
ms.openlocfilehash: 166bdb7a2cf15a84e1b826a9a798042c568bb227
ms.sourcegitcommit: 4c89d9ea4b834d1963c4818a965eaaaa288194eb
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96608229"
---
# <a name="tutorial-add-identity-providers-to-your-applications-in-azure-active-directory-b2c"></a>Tutoriel : Ajouter des fournisseurs d’identité à vos applications dans Azure Active Directory B2C

Dans vos applications, vous souhaiterez peut-être permettre aux utilisateurs de se connecter avec différents fournisseurs d’identité. Un *fournisseur d’identité* crée, entretient et gère les informations d’identité tout en fournissant des services d’authentification pour les applications. Vous pouvez ajouter des fournisseurs d’identité qui sont pris en charge par Azure Active Directory B2C (Azure AD B2C) à votre [flux d’utilisateurs](user-flow-overview.md) à l’aide du portail Azure.

Dans cet article, vous apprendrez comment :

> [!div class="checklist"]
> * Créer des applications de fournisseurs d’identité
> * Ajouter les fournisseurs d’identité à votre locataire, à la fois dans Facebook et dans Azure Active Directory
> * Ajouter les fournisseurs d’identité à votre flux d’utilisateur

Vous utilisez généralement un seul fournisseur d’identité dans vos applications, mais vous pouvez en ajouter d’autres. Ce tutoriel vous montre comment ajouter un fournisseur d’identité Azure AD et un fournisseur d’identité Facebook à votre application. L’ajout de ces deux fournisseurs d’identité à votre application est facultatif. Vous pouvez également ajouter d’autres fournisseurs d’identité, comme [Amazon](identity-provider-amazon.md), [GitHub](identity-provider-github.md), [Google](identity-provider-google.md), [LinkedIn](identity-provider-linkedin.md), [Microsoft](identity-provider-microsoft-account.md) ou [Twitter](identity-provider-twitter.md).

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

[Créez un flux d'utilisateurs](tutorial-create-user-flows.md) pour permettre aux utilisateurs de s'inscrire et de se connecter à votre application.

## <a name="create-applications"></a>Créer des applications

Les applications de fournisseurs d’identité offrent l’identificateur et la clé permettant d’activer la communication avec votre locataire Azure AD B2C. Dans cette section du tutoriel, vous créez une application Azure AD et une application Facebook à partir desquelles vous obtenez les identificateurs et les clés nécessaires pour ajouter les fournisseurs d’identité à votre locataire. Si vous n’ajoutez qu’un des fournisseurs d’identité, il vous suffit de créer l’application pour ce fournisseur.

### <a name="create-an-azure-active-directory-application"></a>Créer une application Azure Active Directory

Pour autoriser la connexion des utilisateurs à partir d’Azure AD, vous devez inscrire une application au sein du locataire Azure AD. Le locataire Azure AD n’est pas identique à votre locataire Azure AD B2C.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Veillez à utiliser l’annuaire qui contient votre locataire Azure AD en sélectionnant le filtre **Annuaire + abonnement** dans le menu du haut et en choisissant l’annuaire qui contient votre locataire Azure AD.
1. Choisissez **Tous les services** dans le coin supérieur gauche du portail Azure, puis recherchez et sélectionnez **Inscriptions d’applications**.
1. Sélectionnez **Nouvelle inscription**.
1. Entrez un nom pour votre application. Par exemple : `Azure AD B2C App`.
1. Acceptez la sélection de **Comptes dans cet annuaire organisationnel uniquement** pour cette application.
1. Pour le champ **URI de redirection**, acceptez la valeur **Web**, puis entrez l’URL suivante en minuscules, en remplaçant `your-B2C-tenant-name` par le nom de votre locataire Azure AD B2C.

    ```
    https://your-B2C-tenant-name.b2clogin.com/your-B2C-tenant-name.onmicrosoft.com/oauth2/authresp
    ```

    Par exemple : `https://contoso.b2clogin.com/contoso.onmicrosoft.com/oauth2/authresp`.

    Toutes les URL doivent désormais utiliser [b2clogin.com](b2clogin.md).

1. Sélectionnez **Inscrire**, puis consignez l’**ID d’application (client)** que vous allez utiliser dans une étape ultérieure.
1. Dans le menu de l’application, sous **Gérer**, sélectionnez **Certificats et secrets**, puis **Nouveau secret client**.
1. Entrez une **Description** pour votre clé secrète client. Par exemple : `Azure AD B2C App Secret`.
1. Sélectionnez la période d’expiration. Pour cette application, acceptez la sélection **Dans 1 an**.
1. Sélectionnez **Ajouter**, puis consignez la valeur de la nouvelle clé secrète client que vous allez utiliser lors d’une étape ultérieure.

### <a name="create-a-facebook-application"></a>Création d’une application Facebook

Pour utiliser un compte Facebook en tant que fournisseur d’identité dans Azure AD B2C, vous devez créer une application dans Facebook. Si vous n’avez pas encore de compte Facebook, vous pouvez en obtenir un à l’adresse [https://www.facebook.com/](https://www.facebook.com/).

1. Connectez-vous à [Facebook pour les développeurs](https://developers.facebook.com/) avec les informations d’identification de votre compte Facebook.
1. Si ce n’est déjà fait, vous devez vous inscrire en tant que développeur Facebook. Pour cela, sélectionnez **Prise en main** dans le coin supérieur droit de la page, acceptez les politiques de Facebook et suivez les étapes d’inscription.
1. Sélectionnez **Mes applications**, puis **Créer une application**.
1. Entrez un **nom d’affichage** et une **adresse e-mail de contact** valide.
1. Cliquez sur **Créer l’ID d’application**. Vous devrez peut-être accepter les politiques de la plateforme Facebook et effectuer une vérification de sécurité en ligne.
1. Sélectionnez **Paramètres** > **Base**.
1. Choisissez une **Catégorie**, par exemple `Business and Pages`. Cette valeur est requise par Facebook, mais n’est pas utilisée par Azure AD B2C.
1. Au bas de la page, sélectionnez **Ajouter une plateforme**, puis sélectionnez **Site web**.
1. Dans **URL du site**, entrez `https://your-tenant-name.b2clogin.com/`, en remplaçant `your-tenant-name` par le nom de votre locataire.
1. Entrez une URL pour l’**URL de stratégie de confidentialité**, par exemple `http://www.contoso.com/`. L’URL de politique de confidentialité est celle d’une page que vous tenez à jour pour fournir des informations de confidentialité pour votre application.
1. Sélectionnez **Enregistrer les modifications**.
1. Consignez la valeur d’**ID d’application** affichée en haut de la page.
1. En regard de **Secret d’application**, sélectionnez **Afficher**, puis consignez cette valeur. Vous avez besoin de l’ID d’application et du secret d’application pour configurer Facebook en tant que fournisseur d’identité dans votre client. Le **Secret d’application** est une information d’identification de sécurité importante que vous devez conserver en lieu sûr.
1. Cliquez sur le signe plus en regard de **PRODUITS**, puis, sous **Connexion Facebook**, sélectionnez **Configurer**.
1. Sous **Connexion Facebook** dans le menu de gauche, sélectionnez **Paramètres**.
1. Dans **URI de redirection OAuth valides**, entrez `https://your-tenant-name.b2clogin.com/your-tenant-name.onmicrosoft.com/oauth2/authresp`. Remplacez `your-tenant-name` par le nom de votre locataire. Sélectionnez **Enregistrer les modifications** en bas de la page.
1. Pour rendre votre application Facebook disponible sur Azure AD B2C, cliquez sur le sélecteur **État** dans la partie supérieure droite de la page et **activez-le** pour rendre l’application publique, puis cliquez sur **Confirmer**. À ce stade, l’état doit passer de **Développement** à **Production**.

## <a name="add-the-identity-providers"></a>Ajouter les fournisseurs d’identité

Après avoir créé l’application pour le fournisseur d’identité que vous souhaitez ajouter, vous ajoutez le fournisseur d’identité à votre locataire.

### <a name="add-the-azure-active-directory-identity-provider"></a>Ajouter le fournisseur d’identité Azure Active Directory

1. Veillez à bien utiliser le répertoire qui contient le locataire Azure AD B2C. Sélectionnez le filtre **Annuaire + abonnement** dans le menu du haut, puis choisissez l’annuaire qui contient votre locataire Azure AD B2C.
1. Choisissez **Tous les services** dans le coin supérieur gauche du portail Azure, puis recherchez et sélectionnez **Azure AD B2C**.
1. Sélectionnez **Fournisseurs d’identité**, puis **Nouveau fournisseur OpenID Connect**.
1. Saisissez un **Nom**. Par exemple, entrez *Contoso Azure AD*.
1. Dans **URL des métadonnées**, entrez l’URL suivante en remplaçant `{tenant}` par le nom de domaine de votre locataire Azure AD :

    ```
    https://login.microsoftonline.com/{tenant}/v2.0/.well-known/openid-configuration
    ```

    Par exemple : `https://login.microsoftonline.com/contoso.onmicrosoft.com/v2.0/.well-known/openid-configuration`.
    Par exemple : `https://login.microsoftonline.com/contoso.com/v2.0/.well-known/openid-configuration`.

1. Dans **ID client**, entrez l’ID d’application que vous avez enregistré précédemment.
1. Dans **Clé secrète client**, entrez la clé secrète client que vous avez enregistrée précédemment.
1. Pour l'**Étendue**, entrez `openid profile`.
1. Conservez les valeurs par défaut pour **Type de réponse** et **Mode de réponse**.
1. (Facultatif) Pour l'**Indication de domaine**, entrez `contoso.com`. Pour plus d’informations, consultez [Configurer la connexion directe avec Azure Active Directory B2C](direct-signin.md#redirect-sign-in-to-a-social-provider).
1. Sous **Mappage des revendications du fournisseur d’identité**, sélectionnez les revendications suivantes :

    * **ID d’utilisateur** : *oid*
    * **Nom d’affichage** : *name*
    * **Prénom** : *given_name*
    * **Nom** : *family_name*
    * **E-mail** : *unique_name*

1. Sélectionnez **Enregistrer**.

### <a name="add-the-facebook-identity-provider"></a>Ajouter le fournisseur d’identité Facebook

1. Sélectionnez **Fournisseurs d’identité**, puis **Facebook**.
1. Saisissez un **Nom**. Par exemple, *Facebook*.
1. Dans **ID client**, entrez l’ID de l’application Facebook que vous avez créée précédemment.
1. Dans **Clé secrète client**, entrez la clé secrète d’application que vous avez consignée.
1. Sélectionnez **Enregistrer**.

## <a name="update-the-user-flow"></a>Mettre à jour le flux d'utilisateurs

Dans le tutoriel que vous avez suivi dans le cadre des prérequis, vous avez créé un flux d’utilisateur pour l’inscription et la connexion nommé *B2C_1_signupsignin1*. Dans cette section, vous ajoutez les fournisseurs d’identité au flux d’utilisateur *B2C_1_signupsignin1*.

1. Sélectionnez **Flux d’utilisateurs**, puis le flux utilisateur *B2C_1_signupsignin1*.
2. Sélectionnez **Fournisseurs d’identité**, sélectionnez les fournisseurs d’identité **Facebook** et **Contoso Azure AD** que vous avez ajoutés.
3. Sélectionnez **Enregistrer**.

## <a name="test-the-user-flow"></a>Tester le flux utilisateur

1. Dans la page Vue d’ensemble du flux d’utilisateur, sélectionnez **Exécuter le flux d’utilisateur**.
1. Pour **Application**, sélectionnez l’application web *webapp1* que vous avez précédemment inscrite. L’**URL de réponse** doit être `https://jwt.ms`.
1. Sélectionnez **Exécuter le flux d’utilisateur**, puis connectez-vous avec un fournisseur d’identité que vous avez précédemment ajouté.
1. Répétez les étapes 1 à 3 pour les autres fournisseurs d’identité que vous avez ajoutés.

Si l’opération de connexion réussit, vous êtes redirigé vers `https://jwt.ms` qui affiche le jeton décodé qui ressemble à ceci :

```json
{
  "typ": "JWT",
  "alg": "RS256",
  "kid": "<key-ID>"
}.{
  "exp": 1562346892,
  "nbf": 1562343292,
  "ver": "1.0",
  "iss": "https://your-b2c-tenant.b2clogin.com/10000000-0000-0000-0000-000000000000/v2.0/",
  "sub": "20000000-0000-0000-0000-000000000000",
  "aud": "30000000-0000-0000-0000-000000000000",
  "nonce": "defaultNonce",
  "iat": 1562343292,
  "auth_time": 1562343292,
  "name": "User Name",
  "idp": "facebook.com",
  "postalCode": "12345",
  "tfp": "B2C_1_signupsignin1"
}.[Signature]
```

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à effectuer les opérations suivantes :

> [!div class="checklist"]
> * Créer des applications de fournisseurs d’identité
> * Ajouter les fournisseurs d’identité dans votre locataire
> * Ajouter les fournisseurs d’identité à votre flux d’utilisateur

Ensuite, apprenez à personnaliser l’interface utilisateur des pages présentées aux utilisateurs dans le cadre de leur expérience d’identité dans vos applications :

> [!div class="nextstepaction"]
> [Personnaliser l’interface utilisateur de vos applications dans Azure Active Directory B2C](tutorial-customize-ui.md)
