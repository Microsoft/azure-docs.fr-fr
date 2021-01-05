---
title: Plug-in Native React pour le SDK JavaScript Application Insights
description: Comment installer et utiliser le plug-in Native React pour le SDK JavaScript Application Insights.
services: azure-monitor
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 08/06/2020
ms.openlocfilehash: 3dac47aa7cf93ca882c4c1d0d191dabf0eb7178c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88226866"
---
# <a name="native-react-plugin-for-application-insights-javascript-sdk"></a>Plug-in Native React pour le SDK JavaScript Application Insights

Le plug-in Native React pour le SDK JavaScript Application Insights recueille les informations de l’appareil. Par défaut, ce plug-in collecte automatiquement les renseignements suivants :

- **ID d’appareil unique** (également appelé ID d’installation.)
- **Nom du modèle d’appareil** (par exemple, iPhone X, Samsung Galaxy Fold, Huawei P30 Pro, etc.)
- **Type d’appareil** (par exemple, téléphone, tablette, etc.)

## <a name="requirements"></a>Spécifications

Vous devez utiliser une version > = 2.0.0 de `@microsoft/applicationinsights-web`. Ce plug-in fonctionne uniquement dans les applications React Native. Il ne fonctionnera pas avec les [applications qui utilisent le framework Expo](https://docs.expo.io/). Par conséquent, il ne fonctionnera pas avec Create React Native App.

## <a name="getting-started"></a>Prise en main

Installez et liez le package [react-native-device-info](https://www.npmjs.com/package/react-native-device-info). Maintenez le package `react-native-device-info` à jour pour collecter les noms des appareils les plus récents à l’aide de votre application.

```zsh

npm install --save @microsoft/applicationinsights-react-native @microsoft/applicationinsights-web
npm install --save react-native-device-info
react-native link react-native-device-info

```

## <a name="initializing-the-plugin"></a>Initialisation du plugin

Pour utiliser ce plug-in, vous devez construire le plug-in et l’ajouter en tant que `extension` à votre instance Application Insights existante.

```typescript
import { ApplicationInsights } from '@microsoft/applicationinsights-web';
import { ReactNativePlugin } from '@microsoft/applicationinsights-react-native';

var RNPlugin = new ReactNativePlugin();
var appInsights = new ApplicationInsights({
    config: {
        instrumentationKey: 'YOUR_INSTRUMENTATION_KEY_GOES_HERE',
        extensions: [RNPlugin]
    }
});
appInsights.loadAppInsights();

```

## <a name="next-steps"></a>Étapes suivantes

- Pour en savoir plus sur le kit de développement logiciel (SDK) JavaScript, consultez la [documentation du SDK JavaScript Application Insights](javascript.md).
- Pour en savoir plus sur le langage de requête Kusto et l’interrogation des données dans Log Analytics, consultez la page [Vue d’ensemble des requêtes de journal](../../azure-monitor/log-query/log-query-overview.md).
