---
title: 'Tutoriel : Activer l’authentification dans une application web Python'
titleSuffix: Azure AD B2C
description: Dans ce tutoriel, vous allez découvrir comment utiliser Azure Active Directory B2C afin de fournir une connexion utilisateur pour une application web Flask Python.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.author: mimart
ms.date: 06/12/2020
ms.topic: tutorial
ms.service: active-directory
ms.subservice: B2C
ms.custom: devx-track-python
ms.openlocfilehash: b4455f21ae7243ab7a15e8d746d6674289f9fdb5
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94953013"
---
# <a name="tutorial-enable-authentication-in-a-python-web-application-with-azure-ad-b2c"></a>Tutoriel : Activer l’authentification dans une application web Python avec Azure AD B2C

Ce tutoriel montre comment utiliser Azure Active Directory B2C (Azure AD B2C) pour inscrire et connecter des utilisateurs dans une application web Flask Python.

Dans ce tutoriel, vous allez :

> [!div class="checklist"]
> * Ajouter une URL de réponse à une application inscrite dans votre locataire Azure AD B2C
> * Télécharger un exemple de code depuis GitHub
> * Modifier le code de l’exemple d’application pour qu’il fonctionne avec votre locataire
> * Inscrire en utilisant votre flux utilisateur d’inscription/connexion

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

Avant de poursuivre les étapes de ce tutoriel, vous devez disposer des ressources Azure AD B2C suivantes :

* [Locataire Azure AD B2C](tutorial-create-tenant.md)
* [Application inscrite](tutorial-register-applications.md) dans votre locataire, et son *ID d’application (client)* et sa *clé secrète client*
* [Flux d’utilisateurs créés](tutorial-create-user-flows.md) dans votre locataire

De plus, vous devez disposer des éléments suivants dans votre environnement de développement local :

* [Visual Studio Code](https://code.visualstudio.com/) ou un autre éditeur de code
* [Python](https://nodejs.org/en/download/) 2.7+ ou 3+

## <a name="add-a-redirect-uri"></a>Ajouter un URI de redirection

Au cours du deuxième tutoriel que vous avez effectué dans le cadre des prérequis, vous avez inscrit une application web dans Azure AD B2C. Pour permettre la communication avec l’exemple de code de ce tutoriel, ajoutez une URL de réponse (également appelée URI de redirection) à l’inscription d’application.

Pour mettre à jour une application dans votre locataire Azure AD B2C, vous pouvez utiliser notre nouvelle expérience unifiée **Inscriptions d’applications** ou notre expérience héritée **Applications (héritées)** . [En savoir plus sur la nouvelle expérience](./app-registrations-training-guide.md).

#### <a name="app-registrations"></a>[Inscriptions des applications](#tab/app-reg-ga/)

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sélectionnez le filtre **Annuaire et abonnement** dans le menu supérieur, puis l’annuaire qui contient votre locataire Azure AD B2C.
1. Dans le menu de gauche, sélectionnez **Azure AD B2C**. Ou sélectionnez **Tous les services**, puis recherchez et sélectionnez **Azure AD B2C**.
1. Sélectionnez **Inscriptions d’applications**, sélectionnez l’onglet **Applications détenues**, puis l’application *webapp1*.
1. Sous **Gérer**, sélectionnez **Authentification**.
1. Sous **Web**, sélectionnez le lien **Ajouter un URI**, puis entrez `http://localhost:5000/getAToken` dans la zone de texte.
1. Sélectionnez **Enregistrer**.

#### <a name="applications-legacy"></a>[Applications (héritées)](#tab/applications-legacy/)

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Veillez à utiliser l’annuaire qui contient votre locataire Azure AD B2C en sélectionnant le filtre **Annuaire + abonnement** dans le menu du haut et en choisissant l’annuaire qui contient votre locataire.
1. Sélectionnez **Tous les services** en haut à gauche du portail Azure, puis recherchez et sélectionnez **Azure AD B2C**.
1. Sélectionnez **Applications (héritées)** , puis l’application *webapp1*.
1. Sous **URL de réponse**, ajoutez `http://localhost:5000/getAToken`.
1. Sélectionnez **Enregistrer**.
* * *

## <a name="get-the-sample-code"></a>Obtention de l'exemple de code

Dans ce tutoriel, vous configurez un exemple de code que vous téléchargez depuis GitHub pour qu’il fonctionne avec votre locataire B2C. L’exemple montre comment une application web Flask Python peut utiliser Azure AD B2C pour l’inscription et la connexion des utilisateurs.

[Téléchargez une archive .ZIP](https://github.com/Azure-Samples/ms-identity-python-webapp/archive/master.zip) ou clonez le [dépôt de l’exemple de code](https://github.com/Azure-Samples/ms-identity-python-webapp) à partir de GitHub.

```console
git clone https://github.com/Azure-Samples/ms-identity-python-webapp.git
```

## <a name="update-the-sample"></a>Mettre à jour l’exemple

Une fois que vous avez obtenu l’exemple, configurez l’application pour qu’elle utilise votre locataire Azure AD B2C, l’inscription d’application et les flux utilisateur.

Dans le répertoire racine du projet :

1. Renommez le fichier *app_config.py* en *app_config.py.OLD*.
1. Renommez *app_config_b2c.py* en *app_config.py*.

Mettez à jour le fichier *app_config.py* que vous venez de renommer avec les valeurs de votre locataire Azure AD B2C et d’inscription d’application que vous avez créées dans le cadre des prérequis.

1. Ouvrez le fichier *app_config.py* dans votre éditeur.
1. Mettez à jour la valeur `b2c_tenant` avec le nom de votre locataire Azure AD B2C, par exemple *contosob2c*.
1. Mettez à jour chacune des valeurs `*_user_flow` pour qu’elles correspondent aux noms des flux utilisateur que vous avez créés dans le cadre des prérequis.
1. Mettez à jour la valeur `CLIENT_ID` avec l’**ID d’application (client)** de l’application web que vous avez inscrite dans le cadre des prérequis.
1. Mettez à jour la valeur `CLIENT_SECRET` avec la valeur de la **clé secrète client** que vous avez créée dans les prérequis. Pour une sécurité accrue, stockez-la plutôt dans une **variable d’environnement** comme recommandé dans les commentaires.

La section supérieure d’*app_config.py* doit désormais ressembler à l’extrait de code suivant :

```python
import os

b2c_tenant = "contosob2c"
signupsignin_user_flow = "B2C_1_signupsignin1"
editprofile_user_flow = "B2C_1_profileediting1"
resetpassword_user_flow = "B2C_1_passwordreset1"
authority_template = "https://{tenant}.b2clogin.com/{tenant}.onmicrosoft.com/{user_flow}"

CLIENT_ID = "11111111-1111-1111-1111-111111111111" # Application (client) ID of app registration

CLIENT_SECRET = "22222222-2222-2222-2222-222222222222" # Placeholder - for use ONLY during testing.
# In a production app, we recommend you use a more secure method of storing your secret,
# like Azure Key Vault. Or, use an environment variable as described in Flask's documentation:
# https://flask.palletsprojects.com/en/1.1.x/config/#configuring-from-environment-variables
# CLIENT_SECRET = os.getenv("CLIENT_SECRET")
# if not CLIENT_SECRET:
#     raise ValueError("Need to define CLIENT_SECRET environment variable")
```

> [!WARNING]
> Comme indiqué dans les commentaires de l’extrait de code, nous vous recommandons de **ne pas stocker les secrets en texte clair** dans le code de votre application. La variable codée en dur est utilisée dans l’exemple de code *uniquement par souci pratique*. Utilisez plutôt une variable d’environnement ou un magasin de secrets comme Azure Key Vault.

## <a name="run-the-sample"></a>Exécution de l'exemple

1. Dans votre console ou terminal, basculez vers le répertoire contenant l’exemple. Par exemple :

    ```console
    cd ms-identity-python-webapp
    ```
1. Exécutez les commandes suivantes pour installer les packages requis à partir de PyPi et exécuter l’application web sur votre ordinateur local :

    ```console
    pip install -r requirements.txt
    flask run --host localhost --port 5000
    ```

    La fenêtre de console affiche le numéro de port de l’application s’exécutant localement :

    ```console
     * Serving Flask app "app" (lazy loading)
     * Environment: production
       WARNING: This is a development server. Do not use it in a production deployment.
       Use a production WSGI server instead.
     * Debug mode: off
     * Running on http://localhost:5000/ (Press CTRL+C to quit)
    ```

1. Accédez à `http://localhost:5000` pour voir l’application web en cours d’exécution sur votre machine locale.

    :::image type="content" source="media/tutorial-web-app-python/python-flask-web-app-01.png" alt-text="Navigateur web montrant l’application web Flask Python s’exécutant localement":::

### <a name="sign-up-using-an-email-address"></a>S’inscrire au moyen d’une adresse e-mail

Cet exemple d’application prend en charge l’inscription, la connexion et la réinitialisation du mot de passe. Dans ce tutoriel, vous vous inscrivez en utilisant une adresse e-mail.

1. Sélectionnez **Se connecter** pour démarrer le flux utilisateur *B2C_1_signupsignin1* que vous avez spécifié lors d’une étape précédente.
1. Azure AD B2C présente une page de connexion qui inclut un lien d’inscription. Étant donné que vous n’avez pas encore de compte, sélectionnez le lien **Inscrivez-vous maintenant**.
1. Le flux de travail d’inscription présente une page pour collecter et vérifier l’identité de l’utilisateur avec une adresse e-mail. Le flux de travail d’inscription collecte également le mot de passe de l’utilisateur et les attributs demandés qui sont définis dans le flux utilisateur.

    Utilisez une adresse e-mail valide et validez à l’aide d’un code de vérification. Définissez un mot de passe. Entrez des valeurs pour les attributs requis.

    :::image type="content" source="media/tutorial-web-app-python/python-flask-web-app-02.png" alt-text="Page d’inscription affichée par le flux utilisateur Azure AD B2C":::

1. Sélectionnez **Créer** pour créer un compte local dans le répertoire Azure AD B2C.

Quand vous sélectionnez **Créer**, l’application affiche le nom de l’utilisateur connecté.

:::image type="content" source="media/tutorial-web-app-python/python-flask-web-app-03.png" alt-text="Navigateur web montrant l’application web Flask Python avec utilisateur connecté":::

Si vous voulez tester la connexion, sélectionnez le bouton **Logout**, puis sélectionnez **Sign In**, et connectez-vous avec l’adresse e-mail et le mot de passe que vous avez entrés lors de votre inscription.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez configuré une application web Flask Python pour qu’elle fonctionne avec un flux utilisateur dans votre locataire Azure AD B2C pour fournir une fonctionnalité d’inscription et de connexion. Vous avez effectué ces étapes :

> [!div class="checklist"]
> * Ajouter une URL de réponse à une application inscrite dans votre locataire Azure AD B2C
> * Télécharger un exemple de code depuis GitHub
> * Modifier le code de l’exemple d’application pour qu’il fonctionne avec votre locataire
> * Inscrire en utilisant votre flux utilisateur d’inscription/connexion

Découvrez maintenant comment personnaliser l’interface utilisateur des pages de flux utilisateur présentées à vos utilisateurs par Azure AD B2C :

> [!div class="nextstepaction"]
> [Tutoriel : Personnaliser l’interface des expériences utilisateur dans Azure AD B2C](tutorial-customize-ui.md)