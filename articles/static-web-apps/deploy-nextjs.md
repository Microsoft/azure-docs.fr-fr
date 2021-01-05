---
title: 'Tutoriel : Déployer des sites web Next.js au rendu statique sur Azure Static Web Apps'
description: Générez et déployez des sites dynamiques Next.js avec Azure Static Web Apps.
services: static-web-apps
author: christiannwamba
ms.service: static-web-apps
ms.topic: tutorial
ms.date: 05/08/2020
ms.author: chnwamba
ms.custom: devx-track-js
ms.openlocfilehash: a22d06137c3ec17851280605ac85c94ef8b342cd
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97563073"
---
# <a name="deploy-static-rendered-nextjs-websites-on-azure-static-web-apps-preview"></a>Déployer des sites web Next.js au rendu statique sur Azure Static Web Apps (préversion)

Ce tutoriel permet d’apprendre à déployer un site web statique généré [Next.js](https://nextjs.org) sur [Azure Static Web Apps](overview.md). Pour commencer, vous apprendrez à créer, configurer et déployer une application Next.js. Au cours de ce processus, vous apprendrez aussi à gérer les problèmes courants rencontrés lors de la génération de pages statiques avec Next.js

## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/).
- Un compte GitHub. [Créez un compte gratuitement](https://github.com/join).
- [Node.js](https://nodejs.org) (déjà installé).

## <a name="set-up-a-nextjs-app"></a>Créer une application Next.js

Au lieu d’utiliser l’interface CLI Next.js pour créer une application, vous pouvez utiliser un référentiel de démarrage qui comprend une application Next.js existante. Ce référentiel contient une application Next.js avec des routages dynamiques, qui met en évidence un problème de déploiement courant. Les routages dynamiques ont besoin d’une configuration de déploiement supplémentaire. Nous verrons cela plus tard.

Pour commencer, créez un référentiel sous votre compte GitHub à partir d’un référentiel de modèles.

1. Accédez à [https://github.com/staticwebdev/nextjs-starter/generate](https://github.com/login?return_to=/staticwebdev/nextjs-starter/generate)
1. Nommez le référentiel **nextjs-starter**
1. Ensuite, clonez le nouveau référentiel sur votre ordinateur. Remplacez `<YOUR_GITHUB_ACCOUNT_NAME>` par le nom de votre compte.

    ```bash
    git clone http://github.com/<YOUR_GITHUB_ACCOUNT_NAME>/nextjs-starter
    ```

1. Accédez à l’application Next.js qui vient d’être clonée :

   ```bash
   cd nextjs-starter
   ```

1. Installez des dépendances :

    ```bash
    npm install
    ```

1. Démarrez l’application Next.js dans l’environnement de développement :

    ```bash
    npm run dev
    ```

Accédez à `http://localhost:3000` pour ouvrir l’application, où le site web suivant devrait s’ouvrir dans votre navigateur favori :

:::image type="content" source="media/deploy-nextjs/start-nextjs-app.png" alt-text="Démarrer l’application Next.js":::

Lorsque vous cliquez sur une infrastructure ou une bibliothèque, vous devez voir une page détaillant des informations sur l’élément sélectionné :

:::image type="content" source="media/deploy-nextjs/start-nextjs-details.png" alt-text="Page de détails":::

## <a name="generate-a-static-website-from-nextjs-build"></a>Générer un site web statique à partir du build Next.js

Quand vous générez un site Next.js en utilisant `npm run build`, l’application est générée comme une application web traditionnelle, et non comme un site statique. Pour générer un site statique, utilisez la configuration d’application suivante.

1. Pour configurer des routages statiques, créez un fichier nommé _next.config.js_ à la racine de votre projet, puis ajoutez le code suivant.

    ```javascript
    module.exports = {
      trailingSlash: true,
      exportPathMap: function() {
        return {
          '/': { page: '/' }
        };
      }
    };
    ```
    
      Cette configuration mappe `/` à la page Next.js qui est utilisée pour le routage `/` et qui est le fichier de la page _pages/index.js_.

1. Mettez à jour le script de génération du _package.json_ pour générer également un site statique après la génération avec la commande `next export` : La commande `export` génère un site statique.

    ```json
    "scripts": {
      "dev": "next dev",
      "build": "next build && next export",
    },
    ```

    Maintenant que cette commande est en place, Static Web Apps exécute le script `build` chaque fois que vous transmettez (push) une validation (commit).

1. Générez un site statique :

    ```bash
    npm run build
    ```

    Le site statique est généré et copié dans un dossier _out_ à la racine de votre répertoire de travail.

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

### <a name="create-a-static-app"></a>Créer une application statique

1. Accéder au [portail Azure](https://portal.azure.com)
1. Cliquez sur **Create a Resource** (Créer une ressource).
1. Recherchez **Static Web Apps**.
1. Cliquez sur **Static Web Apps (Preview)** (Static Web Apps (préversion))
1. Cliquez sur **Créer**

1. Sélectionnez un abonnement dans la liste déroulante *Subscription* (Abonnement) ou utilisez la valeur par défaut.
1. Cliquez sur le lien **New (Nouveau)** sous la liste déroulante *Resource group* (Groupe de ressources). Dans *New resource group name* (Nouveau nom de groupe de ressources), tapez **mystaticsite**, puis cliquez sur **OK**.
1. Indiquez ensuite un nom unique pour votre application dans la zone **Name** (Nom). Les caractères valides sont `a-z`, `A-Z`, `0-9` et `-`. Cette valeur est utilisée comme préfixe d’URL pour votre application statique au format `https://<APP_NAME>.azurestaticapps.net`.
1. Dans la liste déroulante *Region* (Région), choisissez une région la plus proche de vous.
1. Dans la liste déroulante SKU, sélectionnez **Free** (Gratuit).

   :::image type="content" source="media/deploy-nextjs/create-static-web-app.png" alt-text="Création d’application web statique":::

### <a name="add-a-github-repository"></a>Ajouter un référentiel GitHub

Le nouveau compte Static Web Apps doit pouvoir accéder au référentiel avec votre application Next.js. Cela lui permet de déployer automatiquement les validations.

1. Cliquez sur le bouton **Sign in with GitHub** (Se connecter avec GitHub).
1. Sélectionnez l’**organisation** sous laquelle vous avez créé le référentiel pour votre projet Next.js. Il peut s’agir de votre nom d’utilisateur GitHub.
1. Recherchez et sélectionnez le nom du référentiel que vous avez créé précédemment.
1. Choisissez **main** (principale) comme branche dans la liste déroulante *Branch* (Branche).

   :::image type="content" source="media/deploy-nextjs/connect-github.png" alt-text="Connexion à GitHub":::

### <a name="configure-the-build-process"></a>Configurer le processus de génération

Azure Static Web Apps est conçu pour effectuer automatiquement des tâches courantes telles que l’installation de modules NPM et l’exécution de `npm run build` lors de chaque déploiement. Cependant, vous devez configurer vous-même certains paramètres, tels que le dossier source de l’application et le dossier de destination du build.

1. Cliquez sur l’onglet **Build** pour configurer le dossier de sortie statique.

   :::image type="content" source="media/deploy-nextjs/build-tab.png" alt-text="Onglet Build":::

2. Dans la zone de texte *App artifact location* (Emplacement de l’artefact d’application), tapez **out**.

### <a name="review-and-create"></a>Examiner et créer

1. Pour vérifier que les informations sont correctes, cliquez sur le bouton **Review + Create** (Vérifier + créer).
1. Pour démarrer la création de la ressource et la configuration d’une action GitHub pour le déploiement, cliquez sur **Create** (Créer).
1. Une fois le déploiement terminé, cliquez sur **Go to resource** (Accéder à la ressource).
1. Dans la fenêtre _Overview_ (Présentation), cliquez sur le lien *URL* pour ouvrir l’application que vous avez déployée. 

Si le site web ne se charge pas immédiatement, c’est que le workflow GitHub Actions est toujours en cours d’exécution en arrière-plan. Lorsqu’il sera terminé, actualisez votre navigateur pour afficher votre application web.
Si le site web ne se charge pas immédiatement, c’est que le workflow GitHub Actions est toujours en cours d’exécution en arrière-plan. Lorsqu’il sera terminé, actualisez votre navigateur pour afficher votre application web.

Vous pouvez vérifier l’état des workflow Actions en accédant aux actions de votre référentiel :

```url
https://github.com/<YOUR_GITHUB_USERNAME>/nextjs-starter/actions
```

### <a name="sync-changes"></a>Synchroniser les modifications

Une fois l’application créée, Azure Static Web Apps a créé un fichier de workflow GitHub Actions dans votre référentiel. Vous devez placer ce fichier dans votre référentiel local pour synchroniser votre historique git.

Revenez au terminal et exécutez la commande suivante : `git pull origin main`.

## <a name="configure-dynamic-routes"></a>Configurer des routages dynamiques

Accédez au site qui vient d’être déployé et cliquez sur l’un des logos de l’infrastructure ou de la bibliothèque. Au lieu d’obtenir une page détaillant des informations, vous obtenez une page d’erreur 404.

:::image type="content" source="media/deploy-nextjs/404-in-production.png" alt-text="Erreur 404 sur les routages dynamiques":::

Cette erreur se produit, car la configuration de l’application fait que Next.js ne génère que la page accueil.

## <a name="generate-static-pages-from-dynamic-routes"></a>Générer des pages statiques à partir de routages dynamiques

1. Mettez à jour le fichier _next.config.js_, pour que Next.js utilise une liste de toutes les données disponibles afin de générer des pages statiques pour chaque infrastructure/bibliothèque :

   ```javascript
   const data = require('./utils/projectsData');

   module.exports = {
     trailingSlash: true,
     exportPathMap: async function () {
       const { projects } = data;
       const paths = {
         '/': { page: '/' },
       };
  
       projects.forEach((project) => {
         paths[`/project/${project.slug}`] = {
           page: '/project/[path]',
           query: { path: project.slug },
         };
       });
  
       return paths;
     },
   };
   ```

   > [!NOTE]
   > La fonction `exportPathMap` est asynchrone. Vous pouvez donc effectuer une requête vers un API dans cette fonction, puis utiliser la liste retournée pour générer les chemins d’accès.

2. Transmettez les nouvelles modifications à votre référentiel GitHub, puis patientez quelques minutes pendant que GitHub Actions génère à nouveau votre site. Une fois la génération terminée, l’erreur 404 disparaît.

   :::image type="content" source="media/deploy-nextjs/404-in-production-fixed.png" alt-text="Résolution de l’erreur 404 sur les routages dynamiques":::

> [!div class="nextstepaction"]
> [Configurer un nom de domaine personnalisé](custom-domain.md)
