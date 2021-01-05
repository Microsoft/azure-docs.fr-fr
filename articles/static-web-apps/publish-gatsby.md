---
title: 'Tutoriel : Publier un site Gatsby dans Azure Static Web Apps'
description: Ce tutoriel vous montre comment déployer une application Gatsby dans Azure Static Web Apps.
services: static-web-apps
author: aaronpowell
ms.service: static-web-apps
ms.topic: tutorial
ms.date: 05/08/2020
ms.author: aapowell
ms.custom: devx-track-js
ms.openlocfilehash: e24a2a19eb39e2c6c7612631ad98f95cb4c5b9ef
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91250162"
---
# <a name="tutorial-publish-a-gatsby-site-to-azure-static-web-apps-preview"></a>Tutoriel : Publier un site Gatsby dans Azure Static Web Apps Preview

Cet article montre comment créer et déployer une application web [Gatsby](https://gatsbyjs.org) dans [Azure Static Web Apps](overview.md). Vous obtenez ainsi un nouveau site Static Web Apps (avec la fonctionnalité GitHub Actions associée) qui vous permet de contrôler la manière dont l’application est générée et publiée.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> - Créer une application Gatsby
> - Configurer un site Azure Static Web Apps
> - Déployer l’application Gatsby sur Azure

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. Si vous n’en avez pas, vous pouvez créer un [compte gratuitement](https://azure.microsoft.com/free/).
- Un compte GitHub. Si vous n’en avez pas, vous pouvez créer un [compte gratuitement](https://github.com/join).
- [Node.js](https://nodejs.org) (déjà installé).

## <a name="create-a-gatsby-app"></a>Créer une application Gatsby

Créez une application Gatsby à l’aide de l’interface de ligne de commande (CLI) Gatsby :

1. Ouvrez un terminal.
1. Utilisez l’outil [npx](https://www.npmjs.com/package/npx) pour créer une application avec l’interface CLI Gatsby. Cela peut prendre quelques minutes.

   ```bash
   npx gatsby new static-web-app
   ```

1. Accédez à l’application nouvellement créée.

   ```bash
   cd static-web-app
   ```

1. Initialiser un dépôt Git

   ```bash
   git init
   git add -A
   git commit -m "initial commit"
   ```

## <a name="push-your-application-to-github"></a>Transmettre votre application à GitHub

Vous devez avoir un référentiel sur GitHub pour créer une ressource Azure Static Web Apps.

1. Créez un référentiel GitHub vide (ne créez pas de fichier README) à partir de [https://github.com/new](https://github.com/new) nommé **gatsby-static-web-app**.

1. Ensuite, ajoutez le référentiel GitHub que vous venez de créer en tant que référentiel distant à votre référentiel local. Veillez à remplacer l’espace réservé `<YOUR_USER_NAME>` par votre nom d’utilisateur GitHub dans la commande suivante.

   ```bash
   git remote add origin https://github.com/<YOUR_USER_NAME>/gatsby-static-web-app
   ```

1. Transmettez votre référentiel local à GitHub.

   ```bash
   git push --set-upstream origin master
   ```

## <a name="deploy-your-web-app"></a>Déployez votre application web

Les étapes suivantes vous montrent comment créer une application Static Web Apps et la déployer dans un environnement de production.

### <a name="create-the-application"></a>Création de l'application

1. Accéder au [portail Azure](https://portal.azure.com)
1. Cliquez sur **Create a Resource** (Créer une ressource).
1. Recherchez **Static Web Apps**.
1. Cliquez sur **Static Web Apps (Preview)** (Static Web Apps (préversion))
1. Cliquez sur **Créer**

   :::image type="content" source="./media/publish-gatsby/create-in-portal.png" alt-text="Création d’une application Static Web Apps (préversion) dans le portail":::

1. Pour _Abonnement_, acceptez l’abonnement qui est listé ou sélectionnez-en un nouveau dans la liste déroulante.

1. Dans _Groupe de ressources_, sélectionnez **Nouveau**. Dans _New resource group name_ (Nom du nouveau groupe de ressources), entrez **gatsby-static-web-app**, puis sélectionnez **OK**.

1. Ensuite, entrez un nom pour votre application dans la zone **Nom**. Les caractères valides sont `a-z`, `A-Z`, `0-9` et `-`.

1. Dans _Region_ (Région), sélectionnez une région disponible proche de vous.

1. Pour _SKU_, sélectionnez **Free** (Gratuit).

   :::image type="content" source="./media/publish-gatsby/basic-app-details.png" alt-text="Création d’une application Static Web Apps (préversion) dans le portail":::

1. Cliquez sur le bouton **Sign in with GitHub** (Se connecter avec GitHub).

1. Sélectionnez l’**organisation** sous laquelle vous avez créé le référentiel.

1. Sélectionnez **gatsby-static-web-app** comme _Repository_ (Référentiel).

1. Pour _Branch_ (Branche), sélectionnez **master** (principale).

   :::image type="content" source="./media/publish-gatsby/completed-github-info.png" alt-text="Création d’une application Static Web Apps (préversion) dans le portail":::

### <a name="build"></a>Build

Vous ajoutez ensuite les paramètres de configuration utilisés par le processus de génération pour générer votre application.

1. Cliquez sur le bouton **Next: Build >** (Suivant : générer >) pour modifier la configuration de la génération.

1. Pour configurer les paramètres de l’étape dans GitHub Actions, définissez _App location_ (Emplacement de l’application) sur **/** .

1. Définissez _App artifact location_ (Emplacement de l’artefact de l’application) sur **public**.

   Il n’est pas nécessaire de fournir une valeur pour _API location_ (Emplacement de l’API), puisque vous ne déployez pas d’API pour le moment.

   :::image type="content" source="./media/publish-gatsby/build-details.png" alt-text="Création d’une application Static Web Apps (préversion) dans le portail":::

### <a name="review-and-create"></a>Examiner et créer

1. Pour vérifier que les informations sont correctes, cliquez sur le bouton **Review + Create** (Vérifier + créer).

1. Pour démarrer la création de l’application Static Web Apps et la configuration d’une action GitHub pour le déploiement, cliquez sur **Create (Créer)** .

1. Une fois le déploiement terminé, cliquez sur **Go to resource** (Accéder à la ressource).

1. Sur l’écran de la ressource, cliquez sur le lien _URL_ pour ouvrir l’application que vous avez déployée. Vous devrez peut-être patienter une ou deux minutes, le temps que l’action GitHub soit terminée.

   :::image type="content" source="./media/publish-gatsby/deployed-app.png" alt-text="Création d’une application Static Web Apps (préversion) dans le portail":::

## <a name="clean-up-resources"></a>Nettoyer les ressources

[!INCLUDE [cleanup-resource](../../includes/static-web-apps-cleanup-resource.md)]

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Ajouter un domaine personnalisé](custom-domain.md)
