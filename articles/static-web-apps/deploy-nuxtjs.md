---
title: 'Tutoriel : Déployer des sites web Nuxt.js rendus côté serveur sur Azure Static Web Apps'
description: Générez et déployez des sites dynamiques Nuxt.js avec Azure Static Web Apps.
services: static-web-apps
author: christiannwamba
ms.service: static-web-apps
ms.topic: tutorial
ms.date: 05/08/2020
ms.author: chnwamba
ms.custom: devx-track-js
ms.openlocfilehash: 8f21f5fa8ee9035fe594cecff37a63b1ef2115cc
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97563464"
---
# <a name="deploy-server-rendered-nuxtjs-websites-on-azure-static-web-apps-preview"></a>Déployer des sites Web Nuxt.js rendus côté serveur sur Azure Static Web Apps Preview

Ce tutoriel permet d’apprendre à déployer un site web statique généré [Nuxt.js](https://nuxtjs.org) sur [Azure Static Web Apps](overview.md). Pour commencer, vous apprendrez à créer, configurer et déployer une application Nuxt.js. Au cours de ce processus, vous apprendrez aussi à gérer les problèmes courants rencontrés lors de la génération de pages statiques avec Nuxt.js

## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/).
- Un compte GitHub. [Créez un compte gratuitement](https://github.com/join).
- [Node.js](https://nodejs.org) (déjà installé).

## <a name="set-up-a-nuxtjs-app"></a>Créer une application Nuxt.js

Pour créer un projet Nuxt.js, utilisez `create-nuxt-app`. Dans ce tutoriel, au lieu de partir à zéro, vous clonez un référentiel existant. Ce référentiel est créé de telle sorte qu’il constitue un parfait exemple de déploiement d’une application Nuxt.js dynamique en tant que site statique.

1. Créez un référentiel sous votre compte GitHub à partir d’un référentiel de modèles.
1. Accédez à [http://github.com/staticwebdev/nuxtjs-starter/generate](https://github.com/login?return_to=/staticwebdev/nuxtjs-starter/generate)
1. Nommez le référentiel **nuxtjs-Starter**
1. Ensuite, clonez le nouveau référentiel sur votre ordinateur. Veillez à remplacer <YOUR_GITHUB_ACCOUNT_NAME> par le nom de votre compte.

    ```bash
    git clone http://github.com/<YOUR_GITHUB_ACCOUNT_NAME>/nuxtjs-starter
    ```

1. Accédez à l’application Nuxt.js nouvellement clonée :

   ```bash
   cd nuxtjs-starter
   ```

1. Installez des dépendances :

    ```bash
    npm install
    ```

1. Démarrez l’application Nuxt.js dans l’environnement de développement :

    ```bash
    npm run dev
    ```

Accédez à `http://localhost:3000` pour ouvrir l’application, où le site web suivant devrait s’ouvrir dans votre navigateur favori :

:::image type="content" source="media/deploy-nuxtjs/start-nuxtjs-app.png" alt-text="Démarrer l’application Nuxt.js":::

Lorsque vous cliquez sur une infrastructure ou une bibliothèque, vous devez voir une page détaillant des informations sur l’élément sélectionné :

:::image type="content" source="media/deploy-nuxtjs/start-nuxtjs-details.png" alt-text="Page de détails":::

## <a name="generate-a-static-website-from-nuxtjs-build"></a>Générer un site web statique à partir du build Nuxt.js

Quand vous générez un site Nuxt.js en utilisant `npm run build`, l’application est générée comme une application web traditionnelle, et non comme un site statique. Pour générer un site statique, utilisez la configuration d’application suivante.

1. Mettez à jour le script de génération du _package.json_ pour générer uniquement un site statique avec la commande `nuxt generate` :

    ```json
    "scripts": {
      "dev": "nuxt dev",
      "build": "nuxt generate",
    },
    ```

    Maintenant que cette commande est en place, Static Web Apps exécute le script `build` chaque fois que vous transmettez (push) une validation (commit).

1. Générez un site statique :

    ```bash
    npm run build
    ```

    Nuxt.js génère le site statique et le copie dans un dossier _dist_ à la racine de votre répertoire de travail.

    > [!NOTE]
    > Ce dossier est listé dans le fichier _.gitignore_, car il doit être généré par CI/CD lors du déploiement.

## <a name="push-your-static-website-to-github"></a>Transmettre votre site web statique à GitHub

Azure Static Web Apps déploie votre application à partir d’un référentiel GitHub et continue à effectuer cette opération pour chaque validation transmise vers une branche désignée. Utilisez les commandes suivantes pour synchroniser vos modifications avec GitHub.

1. Pour indexer tous les fichiers modifiés :

    ```bash
    git add .
    ```

1. Pour valider toutes les modifications :

    ```bash
    git commit -m "Update build config"
    ```

1. Pour transmettre vos modifications à GitHub :

    ```bash
    git push origin main
    ```

## <a name="deploy-your-static-website"></a>Déployer votre site web statique

La procédure suivante vous indique comment lier Azure Static Web Apps à l’application que vous venez de transmettre à GitHub. Puis, une fois dans Azure, vous pourrez déployer l’application dans un environnement de production.

### <a name="create-an-azure-static-web-apps-preview-resource"></a>Créer une ressource Azure Static Web Apps Preview

1. Accédez au [portail Azure](https://portal.azure.com).
1. Cliquez sur **Create a Resource** (Créer une ressource).
1. Recherchez **Static Web Apps**.
1. Cliquez sur **Static Web Apps (Preview)** (Static Web Apps (Préversion))
1. Cliquez sur **Create** (Créer)

1. Sélectionnez un abonnement dans la liste déroulante *Subscription* (Abonnement) ou utilisez la valeur par défaut.
1. Cliquez sur le lien **New (Nouveau)** sous la liste déroulante *Resource group* (Groupe de ressources). Dans *New resource group name* (Nouveau nom de groupe de ressources), tapez **mystaticsite**, puis cliquez sur **OK**.
1. Indiquez ensuite un nom unique pour votre application dans la zone **Name** (Nom). Les caractères valides sont `a-z`, `A-Z`, `0-9` et `-`. Cette valeur est utilisée comme préfixe d’URL pour votre application statique au format `https://<APP_NAME>.azurestaticapps.net`.
1. Dans la liste déroulante *Region* (Région), choisissez une région la plus proche de vous.
1. Dans la liste déroulante SKU, sélectionnez **Free** (Gratuit).

   :::image type="content" source="media/deploy-nuxtjs/create-static-web-app.png" alt-text="Création d’application web statique":::

### <a name="add-a-github-repository"></a>Ajouter un référentiel GitHub

Le nouveau compte Static Web Apps doit pouvoir accéder au référentiel avec votre application Nuxt.js. Cela lui permet de déployer automatiquement les validations.

1. Cliquez sur le bouton **Sign in with GitHub** (Se connecter avec GitHub).
1. Sélectionnez l’**organisation** sous laquelle vous avez créé le référentiel pour votre projet Nuxt.js. Il peut s’agir de votre nom d’utilisateur GitHub.
1. Recherchez et sélectionnez le nom du référentiel que vous avez créé précédemment.
1. Choisissez **main** (principale) comme branche dans la liste déroulante *Branch* (Branche).

   :::image type="content" source="media/deploy-nuxtjs/connect-github.png" alt-text="Connexion à GitHub":::

### <a name="configure-the-build-process"></a>Configurer le processus de génération

Azure Static Web Apps est conçu pour effectuer automatiquement des tâches courantes telles que l’installation de modules NPM et l’exécution de `npm run build` lors de chaque déploiement. Cependant, vous devez configurer vous-même certains paramètres, tels que le dossier source de l’application et le dossier de destination du build.

1. Cliquez sur l’onglet **Build** pour configurer le dossier de sortie statique.

      :::image type="content" source="media/deploy-nuxtjs/build-tab.png" alt-text="Onglet Build":::

1. Dans la zone de texte *App artifact location* (Emplacement de l’artefact d’application), tapez **dist**.

### <a name="review-and-create"></a>Vérifier et créer

1. Pour vérifier que les informations sont correctes, cliquez sur le bouton **Review + Create** (Vérifier + créer).
1. Pour démarrer la création de la ressource et la configuration d’une action GitHub pour le déploiement, cliquez sur **Create** (Créer).
1. Une fois le déploiement terminé, cliquez sur **Go to resource** (Accéder à la ressource).
1. Dans la fenêtre _Overview_ (Présentation), cliquez sur le lien *URL* pour ouvrir l’application que vous avez déployée. 

Si le site web ne se charge pas immédiatement, c’est que le workflow GitHub Actions est toujours en cours d’exécution en arrière-plan. Lorsqu’il sera terminé, actualisez votre navigateur pour afficher votre application web.

Vous pouvez vérifier l’état des workflow Actions en accédant aux actions de votre référentiel :

```url
https://github.com/<YOUR_GITHUB_USERNAME>/nuxtjs-starter/actions
```

### <a name="sync-changes"></a>Synchroniser les modifications

Une fois l’application créée, Azure Static Web Apps a créé un fichier de workflow GitHub Actions dans votre référentiel. Vous devez placer ce fichier dans votre référentiel local pour synchroniser votre historique git.

Revenez au terminal et exécutez la commande suivante : `git pull origin main`.

## <a name="configure-dynamic-routes"></a>Configurer des routages dynamiques

Accédez au site qui vient d’être déployé et cliquez sur l’un des logos de l’infrastructure ou de la bibliothèque. Au lieu d’obtenir une page détaillant des informations, vous obtenez une page d’erreur 404.

:::image type="content" source="media/deploy-nuxtjs/404-in-production.png" alt-text="Erreur 404 sur les routages dynamiques":::

En effet, Nuxt.js n’a généré que la page d'accueil du site statique. Nuxt.js peut générer des fichiers statiques `.html` équivalents pour chaque fichier de page `.vue`, mais il existe une exception. 

Si la page est une page dynamique, par exemple `_id.vue`, Nuxt.js n’aura pas assez d’informations pour générer un code HTML statique à partir de cette page dynamique. Vous devez alors fournir explicitement les chemins d’accès possibles pour les routages dynamiques.

## <a name="generate-static-pages-from-dynamic-routes"></a>Générer des pages statiques à partir de routages dynamiques

1. Mettez à jour le fichier _nuxt.config.js_, pour que Nuxt.js utilise une liste de toutes les données disponibles afin de générer des pages statiques pour chaque infrastructure/bibliothèque :

   ```javascript
     import { projects } from "./utils/projectsData";

     export default {
       mode: "universal",

       //...truncated

       generate: {
         async routes() {
           const paths = [];

           projects.forEach(project => {
             paths.push(`/project/${project.slug}`);
           });

           return paths;
         }
       }
     };
   ```

   > [!NOTE]
   > `routes` est une fonction asynchrone. Vous pouvez donc effectuer une requête vers un API dans cette fonction, puis utiliser la liste retournée pour générer les chemins d’accès.

2. Transmettez les nouvelles modifications à votre référentiel GitHub, puis patientez quelques minutes pendant que GitHub Actions génère à nouveau votre site. Une fois la génération terminée, l’erreur 404 disparaît.

   :::image type="content" source="media/deploy-nuxtjs/404-in-production-fixed.png" alt-text="Résolution de l’erreur 404 sur les routages dynamiques":::

> [!div class="nextstepaction"]
> [Configurer un nom de domaine personnalisé](custom-domain.md)
