---
title: Plug-in React pour le Kit de développement logiciel (SDK) JavaScript Application Insights
description: Comment installer et utiliser le plug-in React pour le Kit de développement logiciel (SDK) JavaScript Application Insights.
services: azure-monitor
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 07/28/2020
ms.openlocfilehash: 3348654a83b6d0930d10e1f58e07455623b5861d
ms.sourcegitcommit: f311f112c9ca711d88a096bed43040fcdad24433
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94981083"
---
# <a name="react-plugin-for-application-insights-javascript-sdk"></a>Plug-in React pour le Kit de développement logiciel (SDK) JavaScript Application Insights

Le plug-in React pour le Kit de développement logiciel (SDK) JavaScript Application Insights permet :

- le suivi des modifications d’itinéraires ;
- le calcul des statistiques d’utilisation des composants React.

## <a name="getting-started"></a>Prise en main

Installez le package npm :

```bash

npm install @microsoft/applicationinsights-react-js

```

## <a name="basic-usage"></a>Utilisation de base

Initialisez une connexion à Application Insights :

```javascript
// AppInsights.js
import { ApplicationInsights } from '@microsoft/applicationinsights-web';
import { ReactPlugin } from '@microsoft/applicationinsights-react-js';
import { createBrowserHistory } from 'history';

const browserHistory = createBrowserHistory({ basename: '' });
const reactPlugin = new ReactPlugin();
const appInsights = new ApplicationInsights({
    config: {
        instrumentationKey: 'YOUR_INSTRUMENTATION_KEY_GOES_HERE',
        extensions: [reactPlugin],
        extensionConfig: {
          [reactPlugin.identifier]: { history: browserHistory }
        }
    }
});
appInsights.loadAppInsights();
export { reactPlugin, appInsights };
```

Enveloppez votre composant avec la fonction de composant d’ordre supérieur pour activer Application Insights sur celui-ci :

```javascript
import React from 'react';
import { withAITracking } from '@microsoft/applicationinsights-react-js';
import { reactPlugin, appInsights } from './AppInsights';

// To instrument various React components usage tracking, apply the `withAITracking` higher-order
// component function.

class MyComponent extends React.Component {
    ...
}

export default withAITracking(reactPlugin, appInsights, MyComponent);
```

## <a name="configuration"></a>Configuration

| Name    | Default | Description                                                                                                    |
|---------|---------|----------------------------------------------------------------------------------------------------------------|
| history | null    | Historique du routeur React. Pour plus d’informations, consultez la [documentation du package react-router](https://reactrouter.com/web/api/history). Pour savoir comment accéder à l’objet d’historique en dehors des composants, consultez la [FAQ relative à react-router](https://github.com/ReactTraining/react-router/blob/master/FAQ.md#how-do-i-access-the-history-object-outside-of-components).    |

### <a name="react-components-usage-tracking"></a>Suivi de l’utilisation des composants React

Pour instrumenter le suivi de l’utilisation des composants React, appliquez la fonction de composant d’ordre supérieur `withAITracking`.

Elle mesurera le temps à partir de l’événement `ComponentDidMount` par le biais de l’événement `ComponentWillUnmount`. Toutefois, pour plus de précision, le temps pendant lequel l’utilisateur était inactif sera soustrait : `React Component Engaged Time = ComponentWillUnmount timestamp - ComponentDidMount timestamp - idle time`.

Pour afficher cette métrique dans le portail Azure, vous devez accéder à la ressource Application Insights, sélectionner l’onglet « Métriques » et configurer les graphiques vides pour qu’ils affichent le nom de métrique personnalisé « Temps d’engagement du composant React (secondes) », sélectionner l’agrégation (somme, moyenne, etc.) de votre métrique et appliquer la division par « Nom du composant ».

![Capture d’écran du graphique qui affiche la métrique personnalisée « Temps d’engagement du composant React (secondes) » divisée par « Nom du composant »](./media/javascript-react-plugin/chart.png)

Vous pouvez également exécuter des requêtes personnalisées pour diviser les données d’Application Insights afin de générer des rapports et des visualisations selon vos besoins. Dans le portail Azure, accédez à la ressource Application Insights, sélectionnez « Analyse » dans le menu supérieur de l’onglet Vue d’ensemble et exécutez votre requête.

![Capture d’écran des résultats de la requête de métrique personnalisée.](./media/javascript-react-plugin/query.png)

> [!NOTE]
> Il peut s’écouler jusqu’à 10 minutes avant que de nouvelles mesures personnalisées n’apparaissent dans le portail Azure.

## <a name="using-react-hooks"></a>Utilisation de React Hooks

Les [React Hooks](https://reactjs.org/docs/hooks-reference.html) sont une approche de la gestion de l’état et du cycle de vie dans une application React sans dépendre des composants React basés sur une classe. Le plug-in Application Insights React fournit un certain nombre d’intégrations de Hooks qui fonctionnent de la même façon que l’approche du composant d’ordre supérieur.

### <a name="using-react-context"></a>Utilisation React Context

Les React Hooks pour Application Insights sont conçus pour utiliser [React Context](https://reactjs.org/docs/context.html) comme un aspect contenant. Pour utiliser Context, initialisez Application Insights comme ci-dessus, puis importez l’objet Context :

```javascript
import React from "react";
import { AppInsightsContext } from "@microsoft/applicationinsights-react-js";
import { reactPlugin } from "./AppInsights";

const App = () => {
    return (
        <AppInsightsContext.Provider value={reactPlugin}>
            /* your application here */
        </AppInsightsContext.Provider>
    );
};
```

Ce fournisseur de Context rendra Application Insights disponible en tant que Hook `useContext` dans tous ses composants enfants.

```javascript
import React from "react";
import { useAppInsightsContext } from "@microsoft/applicationinsights-react-js";

const MyComponent = () => {
    const appInsights = useAppInsightsContext();
    
    appInsights.trackMetric("Component 'MyComponent' is in use");
    
    return (
        <h1>My Component</h1>
    );
}
export default MyComponent;
```

### `useTrackMetric`

Le Hook `useTrackMetric` réplique les fonctionnalités du composant d’ordre supérieur `withAITracking` sans ajouter de composant supplémentaire à la structure du composant. Le Hook accepte deux arguments : le premier est l’instance Application Insights (qui peut être obtenue à partir du Hook `useAppInsightsContext`), et le second est un identificateur du composant pour le suivi (p. ex : son nom).

```javascript
import React from "react";
import { useAppInsightsContext, useTrackMetric } from "@microsoft/applicationinsights-react-js";

const MyComponent = () => {
    const appInsights = useAppInsightsContext();
    const trackComponent = useTrackMetric(appInsights, "MyComponent");
    
    return (
        <h1 onHover={trackComponent} onClick={trackComponent}>My Component</h1>
    );
}
export default MyComponent;
```

Il fonctionnera comme le composant d’ordre supérieur, mais répondra aux événements de cycle de vie des Hooks, plutôt qu’au cycle de vie d’un composant. Le Hook doit être fourni explicitement aux événements utilisateur s’il doit s’exécuter sur des interactions particulières.

### `useTrackEvent`

Le Hook `useTrackEvent` est utilisé pour suivre tout événement personnalisé qu’une application peut avoir besoin de suivre, par exemple un clic sur un bouton ou tout autre appel API. Il accepte deux arguments, le premier étant l’instance Application Insights (qui peut être obtenue à partir du Hook `useAppInsightsContext`), et le second étant un nom pour l’événement.

```javascript
import React, { useState, useEffect } from "react";
import { useAppInsightsContext, useTrackEvent } from "@microsoft/applicationinsights-react-js";

const ProductCart = () => {
    const appInsights = useAppInsightsContext();
    const trackCheckout = useTrackEvent(appInsights, "Checkout");
    const trackCartUpdate = useTrackEvent(appInsights, "Cart Updated");
    const [cart, setCart] = useState([]);
    
    useEffect(() => {
        trackCartUpdate({ cartCount: cart.length });
    }, [cart]);
    
    const performCheckout = () => {
        trackCheckout();
        // submit data
    };
    
    return (
        <div>
            <ul>
                <li>Product 1 <button onClick={() => setCart([...cart, "Product 1"])}>Add to Cart</button>
                <li>Product 2 <button onClick={() => setCart([...cart, "Product 2"])}>Add to Cart</button>
                <li>Product 3 <button onClick={() => setCart([...cart, "Product 3"])}>Add to Cart</button>
                <li>Product 4 <button onClick={() => setCart([...cart, "Product 4"])}>Add to Cart</button>
            </ul>
            <button onClick={performCheckout}>Checkout</button>
        </div>
    );
}
export default MyComponent;
```

Lorsque le Hook est utilisé, une charge utile de données peut lui être fournie pour ajouter des données supplémentaires à l’événement lorsqu’il est stocké dans Application Insights.

## <a name="react-error-boundaries"></a>Limites d’erreur de React

Les [limites d’erreur](https://reactjs.org/docs/error-boundaries.html) offrent un moyen efficace de traiter une exception lorsqu’elle se produit dans une application React et, lorsqu’une telle erreur se produit, il est probable que l’exception doive être journalisée. Le plug-in React pour Application Insights fournit un composant Limite d’erreur qui journalise automatiquement l’erreur lorsqu’elle se produit.

```javascript
import React from "react";
import { reactPlugin } from "./AppInsights";
import { AppInsightsErrorBoundary } from "@microsoft/applicationinsights-react-js";

const App = () => {
    return (
        <AppInsightsErrorBoundary onError={() => <h1>I believe something went wrong</h1>} appInsights={reactPlugin}>
            /* app here */
        </AppInsightsErrorBoundary>
    );
};
```

`AppInsightsErrorBoundary` nécessite que deux accessoires lui soient transmis, l’instance `ReactPlugin` créée pour l’application et un composant à restituer lorsqu’une erreur se produit. Lorsqu’une erreur non traitée se produit, `trackException` est appelée avec les informations fournies à la limite d’erreur et le composant `onError` est affiché.

## <a name="sample-app"></a>Exemple d'application

Consultez la [démonstration de React d’Application Insights](https://github.com/Azure-Samples/application-insights-react-demo).

## <a name="next-steps"></a>Étapes suivantes

- Pour en savoir plus sur le Kit de développement logiciel (SDK) JavaScript, consultez la [documentation du Kit de développement logiciel (SDK) JavaScript Application Insights](javascript.md).
- Pour en savoir plus sur le langage de requête Kusto et l’interrogation des données dans Log Analytics, consultez la page [Vue d’ensemble des requêtes de journal](../../azure-monitor/log-query/log-query-overview.md).
