---
title: 'Tutoriel : Publier un site Jekyll dans Azure Static Web Apps'
description: Découvrez comment déployer une application Jekyll dans Azure Static Web Apps.
services: static-web-apps
author: craigshoemaker
ms.service: static-web-apps
ms.topic: tutorial
ms.date: 06/08/2020
ms.author: cshoe
ms.openlocfilehash: e3bad12362358620d0f2dc105bb2820dfb691d00
ms.sourcegitcommit: 090ea6e8811663941827d1104b4593e29774fa19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "92000221"
---
# <a name="tutorial-publish-a-jekyll-site-to-azure-static-web-apps-preview"></a>Tutoriel : Publier un site Jekyll dans Azure Static Web Apps (préversion)

Cet article montre comment créer et déployer une application web [Jekyll](https://jekyllrb.com/) dans [Azure Static Web Apps](overview.md).

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> - Créer un site web Jekyll
> - Configurer une application Azure Static Web Apps
> - Déployer l’application Jekyll sur Azure

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

- Installer [Jekyll](https://jekyllrb.com/docs/installation/)
  - Vous pouvez utiliser le sous-système Windows pour Linux et suivre les instructions Ubuntu, si nécessaire.
- Compte Azure avec un abonnement actif. Si vous n’en avez pas, vous pouvez créer un [compte gratuitement](https://azure.microsoft.com/free/).
- Un compte GitHub. Si vous n’en avez pas, vous pouvez créer un [compte gratuitement](https://github.com/join).

## <a name="create-jekyll-app"></a>Créer une application Jekyll

Créez une application Jekyll en utilisant l’interface de ligne de commande (CLI) Jekyll :

1. À partir du terminal, exécutez l’interface CLI Jekyll pour créer une application.

   ```bash
   jekyll new static-app
   ```

1. Accédez à l’application nouvellement créée.

   ```bash
   cd static-app
   ```

1. Initialisez un nouveau dépôt Git.

   ```bash
    git init
   ```

1. Validez les modifications :

   ```bash
   git add -A
   git commit -m "initial commit"
   ```

## <a name="push-your-application-to-github"></a>Transmettre votre application à GitHub

Azure Static Web Apps utilise GitHub pour publier votre site web. Les étapes suivantes vous montrent comment créer un dépôt GitHub.

1. Créez un dépôt GitHub vide (ne créez pas de fichier README) sur [https://github.com/new](https://github.com/new) appelé **Jekyll-static-app**.

1. Ajoutez le référentiel GitHub en tant que référentiel distant pour votre référentiel local. Veillez à remplacer l’espace réservé `<YOUR_USER_NAME>` par votre nom d’utilisateur GitHub dans la commande suivante.

   ```bash
   git remote add origin https://github.com/<YOUR_USER_NAME>/jekyll-static-app
   ```

1. Transmettez votre référentiel local à GitHub.

   ```bash
   git push --set-upstream origin master
   ```

## <a name="deploy-your-web-app"></a>Déployez votre application web

Les étapes suivantes vous montrent comment créer une application Static Web Apps et la déployer dans un environnement de production.

### <a name="create-the-application"></a>Création de l'application

1. Accédez au [portail Azure](https://portal.azure.com).

1. Cliquez sur **Créer une ressource**.

1. Recherchez **Static Web Apps**.

1. Cliquez sur **Static Web Apps (Préversion)** .

1. Cliquez sur **Créer**.

1. Pour **Abonnement**, acceptez l’abonnement qui est listé ou sélectionnez-en un nouveau dans la liste déroulante.

1. Dans _Groupe de ressources_, sélectionnez **Nouveau**. Dans _Nom du nouveau groupe de ressources_, entrez **jekyll-static-app**, puis sélectionnez **OK**.

1. Ensuite, indiquez un nom pour votre application dans la zone _Nom_. Les caractères valides sont `a-z`, `A-Z`, `0-9` et `-`.

1. Dans _Region_ (Région), sélectionnez une région disponible proche de vous.

1. Pour _SKU_, sélectionnez **Free** (Gratuit).

    :::image type="content" source="./media/publish-jekyll/basic-app-details.png" alt-text="Champs renseignés":::

1. Cliquez sur le bouton **Sign in with GitHub** (Se connecter avec GitHub).

1. Sélectionnez l’**organisation** sous laquelle vous avez créé le référentiel.

1. Sélectionnez **jekyll-static-app** comme _Dépôt_.

1. Pour _Branch_ (Branche), sélectionnez **master** (principale).

    :::image type="content" source="./media/publish-jekyll/completed-github-info.png" alt-text="Champs renseignés":::

### <a name="build"></a>Build

Vous ajoutez ensuite les paramètres de configuration utilisés par le processus de génération pour générer votre application. Les paramètres suivants configurent le fichier de workflow d’action GitHub.

1. Cliquez sur le bouton **Next: Build >** (Suivant : Générer >) pour modifier la configuration de la génération.

1. Définissez _App location_ (Emplacement de l’application) sur **/_site**.

1. Laissez l’_emplacement de l’artefact d’application_ vide.

   Il n’est pas nécessaire de fournir une valeur pour _API location_ (Emplacement de l’API), puisque vous ne déployez pas d’API pour le moment.

### <a name="review-and-create"></a>Examiner et créer

1. Pour vérifier que les informations sont correctes, cliquez sur le bouton **Review + Create** (Vérifier + créer).

1. Pour démarrer la création de l’application Azure Static Web Apps et la configuration d’une action GitHub pour le déploiement, cliquez sur **Create** (Créer).

1. Le déploiement échoue tout d’abord, car le fichier de workflow a besoin de certains paramètres spécifiques à Jekyll. Pour ajouter ces paramètres, accédez à votre terminal et tirez (pull) la validation avec l’action GitHub sur votre machine.

   ```bash
   git pull
   ```

1. Ouvrez l’application Jekyll dans un éditeur de texte et ouvrez le fichier _.github/workflows/azure-pages-<NOM_WORKFLOW>.yml_.

1. Après la ligne `- uses: actions/checkout@v2`, ajoutez le bloc de configuration suivant.

    ```yml
    - name: Set up Ruby
      uses: ruby/setup-ruby@ec106b438a1ff6ff109590de34ddc62c540232e0
      with:
        ruby-version: 2.6
    - name: Install dependencies
      run: bundle install
    - name: Jekyll build
      run: jekyll build
    ```

1. Validez le workflow mis à jour, puis transmettez-le à GitHub.

    ```bash
    git add -A
    git commit -m "Updating GitHub Actions workflow"
    git push
    ```

1. Attendez que l’action GitHub se termine.

1. Dans la fenêtre _Vue d’ensemble_ du portail Azure, cliquez sur le lien _URL_ pour ouvrir l’application que vous avez déployée.

   :::image type="content" source="./media/publish-jekyll/deployed-app.png" alt-text="Champs renseignés":::

## <a name="clean-up-resources"></a>Nettoyer les ressources

[!INCLUDE [cleanup-resource](../../includes/static-web-apps-cleanup-resource.md)]

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Ajouter un domaine personnalisé](custom-domain.md)
