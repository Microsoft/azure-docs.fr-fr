---
title: Flux de travail GitHub Actions pour Azure Static Web Apps
description: Découvrez comment utiliser les référentiels GitHub pour configurer le déploiement continu sur Azure Static Web Apps.
services: static-web-apps
author: craigshoemaker
ms.service: static-web-apps
ms.topic: conceptual
ms.date: 05/08/2020
ms.author: cshoe
ms.openlocfilehash: 3518935991409d87917582558a34ad7c54841e23
ms.sourcegitcommit: 2989396c328c70832dcadc8f435270522c113229
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92173668"
---
# <a name="github-actions-workflows-for-azure-static-web-apps-preview"></a>Flux de travail GitHub Actions pour Azure Static Web Apps - Préversion

Lorsque vous créez une ressource d’application Azure Static Web Apps, Azure génère un flux de travail GitHub Actions pour contrôler le déploiement continu de l’application. Le flux de travail est piloté par un fichier YAML. Cet article décrit en détail la structure et les options du fichier de flux de travail.

Les déploiements sont initiés par des [déclencheurs](#triggers), qui exécutent des [travaux](#jobs) qui sont définis par des [étapes](#steps) individuelles.

## <a name="file-location"></a>Emplacement du fichier

Quand vous liez votre référentiel GitHub à Azure Static Web Apps, un fichier de flux de travail est ajouté au référentiel.

Procédez comme suit pour afficher le fichier de flux de travail généré.

1. Ouvrez le référentiel de l’application sur GitHub.
1. Dans l’onglet _Code_ , cliquez sur le dossier `.github/workflows`.
1. Cliquez sur le fichier dont le nom ressemble à `azure-static-web-apps-<RANDOM_NAME>.yml`.

Le fichier YAML dans votre référentiel ressemblera à l’exemple suivant :

```yml
name: Azure Static Web Apps CI/CD

on:
  push:
    branches:
    - master
  pull_request:
    types: [opened, synchronize, reopened, closed]
    branches:
    - master

jobs:
  build_and_deploy_job:
    if: github.event_name == 'push' || (github.event_name == 'pull_request' && github.event.action != 'closed')
    runs-on: ubuntu-latest
    name: Build and Deploy Job
    steps:
    - uses: actions/checkout@v2
      with:
        submodules: true
    - name: Build And Deploy
      id: builddeploy
      uses: Azure/static-web-apps-deploy@v0.0.1-preview
      with:
        azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN_MANGO_RIVER_0AFDB141E }}
        repo_token: ${{ secrets.GITHUB_TOKEN }} # Used for GitHub integrations (i.e. PR comments)
        action: 'upload'
        ###### Repository/Build Configurations - These values can be configured to match you app requirements. ######
        app_location: '/' # App source code path
        api_location: 'api' # Api source code path - optional
        app_artifact_location: 'dist' # Built app content directory - optional
        ###### End of Repository/Build Configurations ######

  close_pull_request_job:
    if: github.event_name == 'pull_request' && github.event.action == 'closed'
    runs-on: ubuntu-latest
    name: Close Pull Request Job
    steps:
    - name: Close Pull Request
      id: closepullrequest
      uses: Azure/static-web-apps-deploy@v0.0.1-preview
      with:
        azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN_MANGO_RIVER_0AFDB141E }}
        action: 'close'
```

## <a name="triggers"></a>Déclencheurs

Le [déclencheur](https://help.github.com/actions/reference/events-that-trigger-workflows) GitHub Actions notifie un flux de travail GitHub Actions pour qu’il exécute un travail basé sur des déclencheurs d’événements. Les déclencheurs sont répertoriés à l’aide de la propriété `on` dans le fichier de flux de travail.

```yml
on:
  push:
    branches:
    - master
  pull_request:
    types: [opened, synchronize, reopened, closed]
    branches:
    - master
```

Avec les paramètres associés à la propriété `on`, vous pouvez définir les branches qui déclenchent un travail et définir les déclencheurs à activer pour les différents états de demandes de tirage (pull request).

Dans cet exemple, un flux de travail est démarré lorsque la branche _master_ change. Les modifications qui lancent le flux de travail sont entre autres les validations et les demandes de tirage (pull request) sur la branche choisie.

## <a name="jobs"></a>travaux

Chaque déclencheur d’événements requiert un gestionnaire d’événements. Les [travaux](https://help.github.com/actions/reference/workflow-syntax-for-github-actions#jobs) définissent ce qui se produit lorsqu’un événement est déclenché.

Dans le fichier de flux de travail Static Web Apps, deux travaux sont disponibles.

| Nom  | Description |
|---------|---------|
|`build_and_deploy_job` | S’exécute quand vous envoyez des validations ou que vous ouvrez une demande de tirage (pull request) sur la branche indiquée dans la propriété `on`. |
|`close_pull_request_job` | S’exécute uniquement lorsque vous fermez une requête de tirage qui supprime l’environnement intermédiaire créé à partir des demandes de tirage (pull requests). |

## <a name="steps"></a>Étapes

Les étapes sont les tâches séquentielles des travaux. Chaque étape effectue des actions comme l’installation de dépendances, l’exécution de tests et le déploiement de votre application en production.

Le fichier de flux de travail définit les étapes suivantes.

| Travail  | Étapes  |
|---------|---------|
| `build_and_deploy_job` |<ol><li>Extrait le référentiel dans l’environnement de l’action.<li>Compile et déploie le référentiel sur Azure Static Web Apps.</ol>|
| `close_pull_request_job` | <ol><li>Indique à Azure Static Web Apps qu’une demande de tirage (pull request) a été fermée.</ol>|

## <a name="build-and-deploy"></a>Générer et déployer

L’étape `Build and Deploy` compile et déploie sur votre instance Azure Static Web Apps. Dans la section `with`, vous pouvez personnaliser les valeurs suivantes pour votre déploiement.

```yml
with:
    azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN_MANGO_RIVER_0AFDB141E }}
    repo_token: ${{ secrets.GITHUB_TOKEN }} # Used for GitHub integrations (i.e. PR comments)
    action: 'upload'
    ###### Repository/Build Configurations - These values can be configured to match you app requirements. ######
    app_location: '/' # App source code path
    api_location: 'api' # Api source code path - optional
    app_artifact_location: 'dist' # Built app content directory - optional
    ###### End of Repository/Build Configurations ######
```

| Propriété | Description | Obligatoire |
|---|---|---|
| `app_location` | Emplacement du code de votre application.<br><br>Par exemple, entrez `/` si le code source de votre application se trouve à la racine du référentiel ou `/app` si le code de votre application se trouve dans un répertoire appelé `app`. | Oui |
| `api_location` | Emplacement de votre code Azure Functions.<br><br>Par exemple, entrez `/api` si le code de votre application se trouve dans un dossier appelé `api`. Si aucune application Azure Functions n’est détectée dans le dossier, la création n’échoue pas, le flux de travail suppose que vous ne souhaitez pas d’API. | Non |
| `app_artifact_location` | Emplacement du répertoire de sortie de compilation par rapport à `app_location`.<br><br>Par exemple, si le code source de votre application se trouve dans `/app`, et que le script de compilation place les fichiers dans le dossier `/app/build`, définissez `build` comme valeur `app_artifact_location`. | Non |

Les valeurs `repo_token`, `action`et `azure_static_web_apps_api_token` sont définies pour vous par Azure Static Web Apps et ne doivent pas être modifiées manuellement.

## <a name="custom-build-commands"></a>Commandes de compilation personnalisées

Vous pouvez bénéficier d’un contrôle plus précis sur les commandes qui s’exécutent pendant un déploiement. Les commandes suivantes peuvent être définies dans la section `with` d’un travail.

Le déploiement appelle toujours `npm install` avant toute commande personnalisée.

| Commande            | Description |
|---------------------|-------------|
| `app_build_command` | Définit une commande personnalisée à exécuter pendant le déploiement de l’application de contenu statique.<br><br>Par exemple, pour configurer une build de production pour une application Angular, créez un script NPM nommé `build-prod` pour exécuter `ng build --prod` et entrez `npm run build-prod` comme commande personnalisée. Si elle n’est pas renseignée, le flux de travail tente d’exécuter les commandes `npm run build` ou `npm run build:Azure`.  |
| `api_build_command` | Définit une commande personnalisée à exécuter pendant le déploiement de l’application API Azure Functions. |

## <a name="route-file-location"></a>Emplacement du fichier de routage

Vous pouvez personnaliser le flux de travail pour rechercher le fichier [routes.json](routes.md) dans n’importe quel dossier de votre référentiel. La propriété suivante peut être définie dans la section `with` d’un travail.

| Propriété            | Description |
|---------------------|-------------|
| `routes_location` | Définit l’emplacement du répertoire où se trouve le fichier _routes.json_ . Cet emplacement est relatif par rapport à la racine du référentiel. |

 L’emplacement de votre fichier de _routes.json_ est particulièrement important si l’étape de compilation de votre framework frontal ne déplace pas ce fichier vers `app_artifact_location` par défaut.

## <a name="environment-variables"></a>Variables d'environnement

Vous pouvez définir des variables d’environnement pour votre build via la section `env` de la configuration d’un travail.

```yaml
jobs:
  build_and_deploy_job:
    if: github.event_name == 'push' || (github.event_name == 'pull_request' && github.event.action != 'closed')
    runs-on: ubuntu-latest
    name: Build and Deploy Job
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true
      - name: Build And Deploy
        id: builddeploy
        uses: Azure/static-web-apps-deploy@v0.0.1-preview
        with:
          azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN }}
          repo_token: ${{ secrets.GITHUB_TOKEN }}
          action: "upload"
          ###### Repository/Build Configurations
          app_location: "/"
          api_location: "api"
          app_artifact_location: "public"
          ###### End of Repository/Build Configurations ######
        env: # Add environment variables here
          HUGO_VERSION: 0.58.0
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Passer en revue les demandes de tirage (pull request) dans les environnements de préproduction](review-publish-pull-requests.md)
