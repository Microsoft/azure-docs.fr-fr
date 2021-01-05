---
title: Plug-in Angular pour le Kit de développement logiciel (SDK) JavaScript Application Insights
description: Comment installer et utiliser le plug-in Angular pour le Kit de développement logiciel (SDK) JavaScript Application Insights.
services: azure-monitor
author: lgayhardt
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 10/07/2020
ms.author: lagayhar
ms.openlocfilehash: 152ba4b1c8a4e09db0bce759f5b67f577ec5d584
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91843599"
---
# <a name="angular-plugin-for-application-insights-javascript-sdk"></a>Plug-in Angular pour le Kit de développement logiciel (SDK) JavaScript Application Insights

Le plug-in Angular pour le Kit de développement logiciel (SDK) JavaScript Application Insights permet :

- le suivi des modifications d’itinéraires ;
- le calcul des statistiques d’utilisation des composants Angular.

> [!WARNING]
> Le plug-in Angular n’est pas compatible avec ECMAScript 3 (ES3).

## <a name="getting-started"></a>Prise en main

Installez le package npm :

```bash
npm install @microsoft/applicationinsights-angularplugin-js
```

## <a name="basic-usage"></a>Utilisation de base

Configurez une instance d’Application Insights dans le composant d’entrée de votre application :

```js
import { Component, OnInit } from '@angular/core';
import { ApplicationInsights } from '@microsoft/applicationinsights-web';
import { AngularPlugin, AngularPluginService } from '@microsoft/applicationinsights-angularplugin-js';
import { Router } from '@angular/router';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent implements OnInit {
    private appInsights;
    constructor(
        private router: Router,
        private angularPluginService: AngularPluginService 
    ){
        var angularPlugin = new AngularPlugin();
        this.angularPluginService.init(angularPlugin, this.router);
        this.appInsights = new ApplicationInsights({ config: {
        instrumentationKey: 'YOUR_INSTRUMENTATION_KEY_GOES_HERE',
        extensions: [angularPlugin],
        extensionConfig: {
            [angularPlugin.identifier]: { router: this.router }
        }
        } });
    }

    ngOnInit() {
        this.appInsights.loadAppInsights();
    }
}

```

Pour utiliser la méthode `trackMetric` pour suivre l’utilisation du composant Angular, ajoutez `AngularPluginService` en tant que fournisseur dans la liste des fournisseurs dans le fichier `app.module.ts`.

```js
import { AngularPluginService } from '@microsoft/applicationinsights-angularplugin-js';

@NgModule({
    ...
  providers: [ AngularPluginService ],
})
export class AppModule { }
```

Pour suivre la durée de vie d’un composant, appelez `trackMetric` dans la méthode `ngOnDestroy` de ce composant. Lorsque le composant est détruit, il déclenche un événement `trackMetric` qui envoie l’heure à laquelle l’utilisateur est resté sur la page et le nom du composant.

```js
import { Component, OnDestroy, HostListener } from '@angular/core';
import { AngularPluginService } from '@microsoft/applicationinsights-angularplugin-js';

@Component({
  selector: 'app-test',
  templateUrl: './test.component.html',
  styleUrls: ['./test.component.css']
})
export class TestComponent implements OnDestroy {

  constructor(private angularPluginService: AngularPluginService) {}

  @HostListener('window:beforeunload')
  ngOnDestroy() {
    this.angularPluginService.trackMetric();
  }
}
```

## <a name="next-steps"></a>Étapes suivantes

- Pour en savoir plus sur le Kit de développement logiciel (SDK) JavaScript, consultez la [documentation du Kit de développement logiciel (SDK) JavaScript Application Insights](javascript.md)
- [Plug-in Angular sur GitHub](https://github.com/microsoft/ApplicationInsights-JS/tree/master/extensions/applicationinsights-angularplugin-js)