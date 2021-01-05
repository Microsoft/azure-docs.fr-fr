---
title: 'Tutoriel : Personnaliser l’interface utilisateur'
titleSuffix: Azure AD B2C
description: Ce tutoriel explique comment personnaliser l’interface utilisateur de vos applications dans Azure Active Directory B2C à l’aide du portail Azure.
services: B2C
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: tutorial
ms.date: 07/30/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 036e9e6bf58d837c698a6092770ef581f77bc20c
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97108244"
---
# <a name="tutorial-customize-the-interface-of-user-experiences-in-azure-active-directory-b2c"></a>Tutoriel : Personnaliser l'interface des expériences utilisateur dans Azure Active Directory B2C

Pour des expériences utilisateur plus courantes telles que l’inscription, la connexion et la modification du profil, vous pouvez utiliser des [flux d’utilisateur](user-flow-overview.md) dans Azure Active Directory B2C (Azure AD B2C). Les informations contenues dans ce tutoriel vous aideront à découvrir comment [personnaliser l’interface utilisateur](customize-ui-with-html.md) de ces expériences à l’aide de vos propres fichiers HTML et CSS.

Dans cet article, vous apprendrez comment :

> [!div class="checklist"]
> * Créer des fichiers de personnalisation de l’interface utilisateur
> * Mettre à jour le flux d'utilisateurs pour utiliser les fichiers
> * Tester l’interface utilisateur personnalisée

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

[Créez un flux d'utilisateurs](tutorial-create-user-flows.md) pour permettre aux utilisateurs de s'inscrire et de se connecter à votre application.

## <a name="create-customization-files"></a>Créer des fichiers de personnalisation

Vous créez un conteneur et un compte de stockage Azure, puis vous placez les fichiers HTML et CSS de base dans le conteneur.

### <a name="create-a-storage-account"></a>Créez un compte de stockage.

Bien que vous puissiez stocker vos fichiers de plusieurs façons, pour les besoins de ce tutoriel vous allez les stocker dans le [stockage d’objets blob Azure](../storage/blobs/storage-blobs-introduction.md).

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Veillez à bien utiliser l’annuaire qui contient votre abonnement Azure. Sélectionnez le filtre **Annuaire + abonnement** dans le menu supérieur et sélectionnez l’annuaire qui contient votre abonnement. Cet annuaire est différent de celui qui contient votre locataire Azure B2C.
3. Choisissez Tous les services dans le coin supérieur gauche du portail Azure, puis recherchez et sélectionnez **Comptes de stockage**.
4. Sélectionnez **Ajouter**.
5. Sous **Groupe de ressources**, sélectionnez **Créer**, entrez un nom pour le nouveau groupe de ressources, puis cliquez sur **OK**.
6. Nommez le compte de stockage. Le nom que vous choisissez doit être unique dans Azure, et contenir entre 3 et 24 caractères, uniquement des lettres minuscules et des chiffres.
7. Sélectionnez l’emplacement du compte de stockage ou acceptez l’emplacement par défaut.
8. Acceptez toutes les autres valeurs par défaut, sélectionnez **Vérifier + créer**, puis cliquez sur **Créer**.
9. Une fois le compte de stockage créé, sélectionnez **Accéder à la ressource**.

### <a name="create-a-container"></a>Créez un conteneur.

1. Dans la page de vue d’ensemble du compte de stockage, sélectionnez **Objets blob**.
2. Sélectionnez **Conteneur**, entrez un nom pour le conteneur, choisissez **Blob (accès en lecture anonyme pour les objets blob uniquement)** , puis cliquez sur **OK**.

### <a name="enable-cors"></a>Activez CORS

 Le code Azure AD B2C dans un navigateur utilise une approche moderne et standard pour charger le contenu personnalisé à partir d’une URL que vous spécifiez dans un flux d’utilisateur. Le partage des ressources Cross-Origin (CORS) permet de demander des ressources restreintes dans une page web à partir d’autres domaines.

1. Dans le menu, sélectionnez **CORS**.
2. Pour **Origines autorisées**, entrez `https://your-tenant-name.b2clogin.com`. Remplacez `your-tenant-name` par le nom de votre locataire Azure AD B2C. Par exemple : `https://fabrikam.b2clogin.com`. Vous devez utiliser des minuscules quand vous entrez le nom de votre locataire.
3. Pour **Méthodes autorisées**, sélectionnez `GET`, `PUT` et `OPTIONS`.
4. Pour **En-têtes autorisés**, saisissez un astérisque (*).
5. Pour **En-têtes exposés**, saisissez un astérisque (*).
6. Pour **Âge maximal**, tapez 200.

    ![Page de configuration CORS dans Stockage Blob Azure dans le Portail Azure](./media/tutorial-customize-ui/enable-cors.png)

5. Cliquez sur **Enregistrer**.

### <a name="create-the-customization-files"></a>Créer les fichiers de personnalisation

Pour personnaliser l’interface utilisateur de l’expérience d’inscription, commencez par créer un simple fichier HTML et CSS. Vous pouvez configurer votre code HTML comme vous le souhaitez, mais il doit avoir un élément **div** avec `api` comme identificateur. Par exemple : `<div id="api"></div>`. Azure AD B2C injecte des éléments dans le conteneur `api` quand la page s’affiche.

1. Dans un dossier local, créez le fichier suivant en n’oubliant pas de remplacer `your-storage-account` par le nom du compte de stockage et `your-container` par le nom du conteneur que vous avez créé. Par exemple : `https://store1.blob.core.windows.net/b2c/style.css`.

    ```html
    <!DOCTYPE html>
    <html>
      <head>
        <title>My B2C Application</title>
        <link rel="stylesheet" href="https://your-storage-account.blob.core.windows.net/your-container/style.css">
      </head>
      <body>
        <h1>My B2C Application</h1>
        <div id="api"></div>
      </body>
    </html>
    ```

    Vous pouvez concevoir la page comme bon vous semble, mais l’élément div **api** est obligatoire pour tout fichier de personnalisation HTML que vous créez.

3. Enregistrez le fichier sous *custom-ui.html*.
4. Créez la feuille CSS simple suivante, qui centre tous les éléments dans la page d’inscription ou de connexion, y compris les éléments injectés par Azure AD B2C.

    ```css
    h1 {
      color: blue;
      text-align: center;
    }
    .intro h2 {
      text-align: center;
    }
    .entry {
      width: 300px ;
      margin-left: auto ;
      margin-right: auto ;
    }
    .divider h2 {
      text-align: center;
    }
    .create {
      width: 300px ;
      margin-left: auto ;
      margin-right: auto ;
    }
    ```

5. Enregistrez le fichier sous *style.css*.

### <a name="upload-the-customization-files"></a>Charger les fichiers de personnalisation

Dans ce tutoriel, vous stockez les fichiers que vous avez créés dans le compte de stockage afin qu’Azure AD B2C puisse y accéder.

1. Choisissez **Tous les services** dans le coin supérieur gauche du portail Azure, puis recherchez et sélectionnez **Comptes de stockage**.
2. Sélectionnez le compte de stockage que vous avez créé, sélectionnez **Blobs**, puis sélectionnez le conteneur que vous avez créé.
3. Sélectionnez **Charger**, accédez au fichier *custom-ui.html* et sélectionnez-le, puis cliquez sur **Charger**.

    ![Page Charger l’objet blob dans le portail avec le bouton Charger et Fichiers mis en surbrillance](./media/tutorial-customize-ui/upload-blob.png)

4. Copiez l’URL du fichier que vous avez chargé afin de l’utiliser ultérieurement dans ce tutoriel.
5. Répétez les étapes 3 et 4 pour le fichier *style.css*.

## <a name="update-the-user-flow"></a>Mettre à jour le flux d'utilisateurs

1. Choisissez **Tous les services** dans le coin supérieur gauche du portail Azure, puis recherchez et sélectionnez **Azure AD B2C**.
2. Sélectionnez **Flux d’utilisateurs**, puis le flux utilisateur *B2C_1_signupsignin1*.
3. Sélectionnez **Mises en page**, puis sous **Page unifiée d'inscription ou de connexion**, cliquez sur **Oui** pour **Utiliser un contenu de page personnalisé**.
4. Dans **URI la page personnalisée**, entrez l'URI du fichier *custom-ui.html* notée précédemment.
5. En haut de la page, sélectionnez **Enregistrer**.

## <a name="test-the-user-flow"></a>Tester le flux utilisateur

1. Dans votre locataire Azure AD B2C, sélectionnez **Flux d'utilisateurs** et choisissez le flux d'utilisateurs *B2C_1_signupsignin1*.
2. En haut de la page, cliquez sur **Exécuter le flux d’utilisateur**.
3. Cliquez sur le bouton **Exécuter le flux d’utilisateur**.

    ![Exécuter une page de flux d’utilisateur pour le flux d’utilisateur d’inscription ou de connexion](./media/tutorial-customize-ui/run-user-flow.png)

    Vous devez voir une page semblable à l’exemple suivant avec les éléments centrés conformément au fichier CSS que vous avez créé :

    ![Navigateur web présentant une page d’inscription ou de connexion avec des éléments d’interface utilisateur personnalisés](./media/tutorial-customize-ui/run-now.png)

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à effectuer les opérations suivantes :

> [!div class="checklist"]
> * Créer des fichiers de personnalisation de l’interface utilisateur
> * Mettre à jour le flux d'utilisateurs pour utiliser les fichiers
> * Tester l’interface utilisateur personnalisée

> [!div class="nextstepaction"]
> [Personnaliser l’interface utilisateur dans Azure Active Directory B2C](customize-ui-with-html.md)
