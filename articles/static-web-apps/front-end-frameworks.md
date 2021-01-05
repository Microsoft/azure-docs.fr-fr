---
title: Configurer des infrastructures frontales avec Azure Static Web Apps en préversion
description: Paramètres pour les infrastructures frontales populaires nécessaires pour Azure static Web Apps
services: static-web-apps
author: craigshoemaker
ms.service: static-web-apps
ms.topic: conceptual
ms.date: 07/18/2020
ms.author: cshoe
ms.openlocfilehash: 4b1bc58b6b4a87cd6e5e09e83020a38261b8746f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90905371"
---
# <a name="configure-front-end-frameworks-and-libraries-with-azure-static-web-apps-preview"></a>Configurer des infrastructures et bibliothèques frontales avec Azure Static Web Apps en préversion

Azure Static Web Apps nécessite que vous disposiez des valeurs de configuration appropriées dans le [fichier de configuration de build](github-actions-workflow.md) pour votre infrastructure ou bibliothèque frontale.

## <a name="configuration"></a>Configuration

Le tableau suivant répertorie les paramètres d’une série d’infrastructures et de bibliothèques<sup>1</sup>.

Les éléments suivants expliquent la présence des colonnes du tableau :

- **Emplacement de l’artefact de l’application** : Répertorie la valeur de `app_artifact_location`, qui est le [dossier des versions générées des fichiers d’application](github-actions-workflow.md#build-and-deploy).

- **Commande de compilation personnalisée** : Lorsque l’infrastructure requiert une commande différente de `npm run build` ou `npm run azure:build`, vous pouvez définir une [commande de compilation personnalisée](github-actions-workflow.md#custom-build-commands).

| Framework | Emplacement de l’artefact de l’application | Commande de compilation personnalisée |
|--|--|--|
| [Alpine.js](https://github.com/alpinejs/alpine/) | `/` | n/a <sup>2</sup> |
| [Angular](https://angular.io/) | `dist/<APP_NAME>` | `npm run build -- --prod` |
| [Angular Universal](https://angular.io/guide/universal) | `dist/<APP_NAME>/browser` | `npm run prerender` |
| [Aurelia](https://aurelia.io/) | `dist` | n/a |
| [Backbone.js](https://backbonejs.org/) | `/` | n/a |
| [Blazor](https://dotnet.microsoft.com/apps/aspnet/web-apps/blazor) | `wwwroot` | n/a |
| [Ember](https://emberjs.com/) | `dist` | n/a |
| [Flutter](https://flutter.dev/) | `build/web` | `flutter build web` |
| [Framework7](https://framework7.io/) | `www` | `npm run build-prod` |
| [Glimmer](https://glimmerjs.com/) | `dist` | n/a |
| [HTML](https://developer.mozilla.org/docs/Web/HTML) | `/` | n/a |
| [Hyperapp](https://hyperapp.dev/) | `/` | n/a |
| [JavaScript](https://developer.mozilla.org/docs/Web/javascript) | `/` | n/a |
| [JQuery](https://jquery.com/) | `/` | n/a |
| [KnockoutJS](https://knockoutjs.com/) | `dist` | n/a |
| [LitElement](https://lit-element.polymer-project.org/) | `dist` | n/a |
| [Marko](https://markojs.com/) | `public` | n/a |
| [Meteor](https://www.meteor.com/) | `bundle` | n/a |
| [Mithril](https://mithril.js.org/) | `dist` | n/a |
| [Meri](https://www.polymer-project.org/) | `build/default` | n/a |
| [Preact](https://preactjs.com/) | `build` | n/a |
| [React](https://reactjs.org/) | `build` | n/a |
| [Stencil](https://stenciljs.com/) | `www` | n/a |
| [Svelte](https://svelte.dev/) | `public` | n/a |
| [Three.js](https://threejs.org/) | `/` | n/a |
| [TypeScript](https://www.typescriptlang.org/) | `dist` | n/a |
| [Vue.js](https://vuejs.org/) | `dist` | n/a |

<sup>1</sup> Le tableau ci-dessus n’est pas la liste exhaustive des infrastructures et des bibliothèques qui fonctionnent avec Azure Static Web Apps.

<sup>2</sup> Non applicable

## <a name="next-steps"></a>Étapes suivantes

- [Configuration de build et de workflow](github-actions-workflow.md)
