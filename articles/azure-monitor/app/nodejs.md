---
title: Surveiller les services Node.js avec Azure Application Insights | Microsoft Docs
description: Analysez les performances et diagnostiquez les problèmes dans les services Node.js avec Application Insights.
ms.topic: conceptual
ms.date: 06/01/2020
ms.custom: devx-track-js
ms.openlocfilehash: 7aea6c03b0ce35fa0e74c39ff5f94f714447ad6f
ms.sourcegitcommit: fec60094b829270387c104cc6c21257826fccc54
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96920588"
---
# <a name="monitor-your-nodejs-services-and-apps-with-application-insights"></a>Surveiller vos services et applications Node.js avec Application Insights

[Application Insights](./app-insights-overview.md) surveille vos services et composants principaux après leur déploiement pour vous permettre de détecter et diagnostiquer rapidement différents problèmes, notamment les problèmes de performances. Vous pouvez utiliser Application Insights pour les services Node.js hébergés dans votre centre de données, dans les machines virtuelles et applications web Azure, et même dans d’autres clouds publics.

Pour recevoir, stocker et explorer vos données d’analyse, incluez le kit de développement logiciel (SDK) dans votre code et configurez une ressource Application Insights correspondante dans Azure. Le kit de développement logiciel (SDK) envoie les données à cette ressource pour une analyse et une exploration plus approfondies.

Le kit de développement logiciel (SDK) Node.js peut automatiquement analyser les demandes HTTP entrantes et sortantes, les exceptions et certaines mesures du système. À partir de la version 0.20, le kit de développement logiciel (SDK) peut surveiller des [packages tiers](https://github.com/microsoft/node-diagnostic-channel/tree/master/src/diagnostic-channel-publishers#currently-supported-modules) communs comme MongoDB, MySQL et Redis. Tous les événements liés à une demande HTTP entrante sont mis en corrélation pour une résolution des problèmes plus rapide.

Vous pouvez utiliser l’API TelemetryClient pour instrumenter et surveiller manuellement des aspects supplémentaires de votre application et de votre système. L’API TelemetryClient est décrite plus en détail plus loin dans cet article.

## <a name="get-started"></a>Bien démarrer

Terminez les tâches suivantes pour configurer la surveillance d’une application ou d’un service.

### <a name="prerequisites"></a>Prérequis

Avant de commencer, vérifiez que vous disposez d’un abonnement Azure ou [obtenez-en un gratuitement][azure-free-offer]. Si votre organisation possède déjà un abonnement Azure, un administrateur peut suivre [ces instructions][add-aad-user] pour vous y ajouter.

[azure-free-offer]: https://azure.microsoft.com/free/
[add-aad-user]: ../../active-directory/fundamentals/add-users-azure-active-directory.md

### <a name="set-up-an-application-insights-resource"></a><a name="resource"></a> Configurer une ressource Application Insights

1. Connectez-vous au [portail Azure][portal].
2. [Création d’une ressource Application Insights](create-new-resource.md)

### <a name="set-up-the-nodejs-sdk"></a><a name="sdk"></a> Configurer le kit de développement logiciel (SDK) Node.js

Incluez le kit de développement logiciel dans votre application afin qu’il collecte des données.

> [!IMPORTANT]
> Les nouvelles régions Azure **exigent** l’utilisation de chaînes de connexion au lieu de clés d’instrumentation. Une [chaîne de connexion](./sdk-connection-string.md?tabs=nodejs) identifie la ressource à laquelle vous souhaitez associer vos données de télémétrie. Elle vous permet également de modifier les points de terminaison que votre ressource utilisera comme destination pour votre télémétrie. Vous devrez copier la chaîne de connexion et l’ajouter au code de votre application ou à une variable d’environnement.

1. Copiez la clé d’instrumentation de votre ressource (aussi appelée *iClé*) à partir de votre ressource nouvellement créée. Application Insights utilise l’iClé pour mapper les données à votre ressource Azure. Avant que le kit de développement logiciel (SDK) ne puisse utiliser votre iClé, vous devez la spécifier dans une variable d’environnement ou dans votre code.  

   ![Copier la clé d’instrumentation](./media/nodejs/instrumentation-key-001.png)

2. Ajoutez la bibliothèque de kit de développement logiciel (SDK) Node.js aux dépendances de votre application par le biais de package.json. À partir du dossier racine de votre application, exécutez :

   ```bash
   npm install applicationinsights --save
   ```

    > [!NOTE]
    > Si vous utilisez TypeScript, n’installez pas de packages de « typages » distincts. Ce package NPM contient de typages intégrés.

3. Chargez explicitement la bibliothèque dans votre code. Étant donné que le kit de développement logiciel (SDK) injecte l’instrumentation dans beaucoup d’autres bibliothèques, chargez-la dès que possible, même avant les autres instructions `require`.

   ```javascript
   let appInsights = require('applicationinsights');
   ```
4.  Vous pouvez également fournir une iClé par le biais de la variable d’environnement `APPINSIGHTS_INSTRUMENTATIONKEY` au lieu de la transférer manuellement vers `setup()` ou `new appInsights.TelemetryClient()`. Cela vous permet de conserver les iClés en dehors du code source dédié et de spécifier des iClés différentes selon les environnements. Pour configurer manuellement appelez `appInsights.setup('[your ikey]');`.

    Pour des options de configuration supplémentaires, consultez les sections suivantes.

    Vous pouvez essayer le kit de développement logiciel (SDK) sans envoyer de télémétrie en définissant `appInsights.defaultClient.config.disableAppInsights = true`.

5. Commencez à collecter et à envoyer automatiquement des données en appelant `appInsights.start();`.

### <a name="monitor-your-app"></a><a name="monitor"></a> Surveiller votre application

Le Kit de développement logiciel (SDK) recueille automatiquement les données de télémétrie sur le runtime Node.js et certains modules tiers courants. Utilisez votre application pour générer certaines de ces données.

Ensuite, sur le [Portail Azure][portal], accédez à la ressource Application Insights que vous avez créée. Dans la **Vue d’ensemble de la chronologie**, cherchez vos premiers points de données. Pour voir plus de données détaillées, sélectionnez des composants différents dans les graphiques.

Pour consulter la topologie découverte pour votre application, vous pouvez utiliser le [Mappage d’application](app-map.md).

#### <a name="no-data"></a>Pas de données

Étant donné que le kit de développement logiciel (SDK) regroupe les données à envoyer, l’affichage des éléments dans le portail peut prendre un certain temps. Si les données n’apparaissent pas dans votre ressource, essayez ce qui suit :

* Continuez d’utiliser l’application. Prenez plus de mesures pour générer davantage de données de télémétrie.
* Cliquez sur **Actualiser** dans l’affichage des ressources du portail. Les graphiques s’actualisent périodiquement et automatiquement. Les actualiser manuellement les force à s’actualiser immédiatement.
* Vérifiez que les [ports sortants requis](./ip-addresses.md) sont ouverts.
* Utilisez la fonction [Recherche](./diagnostic-search.md) pour chercher des événements spécifiques.
* Consultez les [FAQ][FAQ].

## <a name="basic-usage"></a>Utilisation de base

Pour la collecte des requêtes HTTP prêtes à l’emploi, les événements populaires de la bibliothèque tierce, les exceptions non gérées et les métriques du système :

```javascript

let appInsights = require("applicationinsights");
appInsights.setup("[your ikey]").start();

```

> [!NOTE]
> Si la clé d’instrumentation est définie dans la variable d’environnement `APPINSIGHTS_INSTRUMENTATIONKEY`, `.setup()` peut être appelé sans argument. Cela facilite l’utilisation de différentes iClés pour différents environnements.

Chargez la bibliothèque Application Insights, `require("applicationinsights")`, le plus tôt possible dans vos scripts avant de charger d’autres packages. Cela est nécessaire pour que la bibliothèque Application Insights puisse préparer des packages ultérieurs pour le suivi. Si vous rencontrez des conflits avec d’autres bibliothèques en procédant à une préparation similaire, essayez de charger la bibliothèque Application Insights après celles-ci.

En raison de la façon dont JavaScript gère les rappels, un travail supplémentaire est nécessaire pour effectuer le suivi d’une requête sur les dépendances externes et les rappels ultérieurs. Par défaut, ce suivi supplémentaire est activé ; désactivez-le en appelant `setAutoDependencyCorrelation(false)` comme décrit dans la section [configuration](#sdk-configuration) ci-dessous.

## <a name="migrating-from-versions-prior-to-022"></a>Migration à partir de versions antérieures à 0.22

Il existe des changements cassants entre les versions antérieures à la version 0.22 et les versions ultérieures. Ces changements sont conçus pour apporter une cohérence avec d’autres Kits de développement logiciel (SDK) Application Insights et permettre une extensibilité future.

En général, vous pouvez migrer avec les éléments suivants :

- Remplacez les références à `appInsights.client` par `appInsights.defaultClient`.
- Remplacez les références à `appInsights.getClient()` par `new appInsights.TelemetryClient()`
- Remplacez tous les arguments des méthodes client.track* par un objet unique contenant des propriétés nommées en tant qu’arguments. Consultez les indications de type intégré de l’IDE ou [TelemetryTypes](https://github.com/Microsoft/ApplicationInsights-node.js/tree/develop/Declarations/Contracts/TelemetryTypes) pour l’objet exempté et pour chaque type de données de télémétrie.

Si vous accédez aux fonctions de configuration du Kit de développement logiciel (SDK) sans les chaîner à `appInsights.setup()`, vous pouvez maintenant trouver ces fonctions à `appInsights.Configurations` (par exemple, `appInsights.Configuration.setAutoCollectDependencies(true)`). Passez en revue les modifications apportées à la configuration par défaut dans la section suivante.

## <a name="sdk-configuration"></a>Configuration du kit de développement logiciel (SDK)

L’objet `appInsights` fournit un certain nombre de méthodes de configuration. Elles sont répertoriées dans l’extrait de code suivant avec leurs valeurs par défaut.

```javascript
let appInsights = require("applicationinsights");
appInsights.setup("<instrumentation_key>")
    .setAutoDependencyCorrelation(true)
    .setAutoCollectRequests(true)
    .setAutoCollectPerformance(true, true)
    .setAutoCollectExceptions(true)
    .setAutoCollectDependencies(true)
    .setAutoCollectConsole(true)
    .setUseDiskRetryCaching(true)
    .setSendLiveMetrics(false)
    .setDistributedTracingMode(appInsights.DistributedTracingModes.AI)
    .start();
```

Pour mettre pleinement en corrélation les événements dans un service, veillez à définir `.setAutoDependencyCorrelation(true)`. Lorsque cette option est définie, le kit de développement logiciel (SDK) peut effectuer le suivi de contexte entre les rappels asynchrones dans Node.js.

Passez en revue leurs descriptions dans les indications de type intégré de l’IDE ou [applicationinsights.ts](https://github.com/microsoft/ApplicationInsights-node.js/blob/develop/applicationinsights.ts) pour obtenir des informations détaillées sur ce qu’elles contrôlent, ainsi que sur les arguments secondaires facultatifs.

> [!NOTE]
>  Par défaut `setAutoCollectConsole` est configuré pour *exclure* les appels de `console.log` (et d’autres méthodes de console). Seuls les appels aux enregistreurs d’événements tiers pris en charge (par exemple, Winston et Bunyan) sont collectés. Vous pouvez modifier ce comportement pour inclure des appels aux méthodes `console` à l’aide de `setAutoCollectConsole(true, true)`.

### <a name="sampling"></a>échantillonnage

Par défaut, le Kit de développement logiciel (SDK) envoie toutes les données collectées au service Application Insights. Si vous collectez beaucoup de données, vous pouvez activer l’échantillonnage pour réduire la quantité de données envoyées. Définissez le champ `samplingPercentage` sur l’objet `config` d’un client pour y parvenir. Définir `samplingPercentage` sur 100 (la valeur par défaut) signifie que toutes les données seront envoyées et sur 0 cela signifie que rien ne sera envoyé.

Si vous utilisez la corrélation automatique, toutes les données associées à une requête unique sont incluses ou exclues en tant qu’unité.

Ajoutez du code tel que le suivant pour activer l’échantillonnage :

```javascript
const appInsights = require("applicationinsights");
appInsights.setup("<instrumentation_key>");
appInsights.defaultClient.config.samplingPercentage = 33; // 33% of all telemetry will be sent to Application Insights
appInsights.start();
```

### <a name="multiple-roles-for-multi-components-applications"></a>Plusieurs rôles pour les applications à composants multiples

Si votre application se compose de plusieurs composants que vous souhaitez instrumenter avec la même clé d’instrumentation et que vous voyez toujours ces composants comme des unités distinctes dans le portail, comme s’ils utilisaient des clés d’instrumentation distinctes (par exemple, des nœuds distincts sur la cartographie d’application), vous devrez peut-être configurer manuellement le champ RoleName pour distinguer les données de télémétrie d’autres composants qui envoient des données à votre ressource Application Insights.

Pour définir le champ RoleName, procédez comme suit :

```javascript
const appInsights = require("applicationinsights");
appInsights.setup("<instrumentation_key>");
appInsights.defaultClient.context.tags[appInsights.defaultClient.context.keys.cloudRole] = "MyRoleName";
appInsights.start();
```

### <a name="automatic-third-party-instrumentation"></a>Instrumentation tierce automatique

Pour suivre le contexte entre les appels asynchrones, certaines modifications sont nécessaires dans les bibliothèques tierces telles que MongoDB et Redis. Par défaut, Application Insights utilisera [`diagnostic-channel-publishers`](https://github.com/Microsoft/node-diagnostic-channel/tree/master/src/diagnostic-channel-publishers) pour créer des MonkeyPatch pour certaines de ces bibliothèques. Cela peut être désactivé en définissant la variable d’environnement `APPLICATION_INSIGHTS_NO_DIAGNOSTIC_CHANNEL`.

> [!NOTE]
> En définissant cette variable d’environnement, les événements peuvent ne plus être associés correctement à l’opération appropriée.

 Les MonkeyPatch individuels peuvent être désactivés en définissant la variable d’environnement `APPLICATION_INSIGHTS_NO_PATCH_MODULES` sur une liste séparée par des virgules de packages à désactiver (par exemple, `APPLICATION_INSIGHTS_NO_PATCH_MODULES=console,redis`) pour éviter d’appliquer des correctifs aux packages `console` et `redis`.

Il y a actuellement neuf packages instrumentés : `bunyan`,`console`,`mongodb`,`mongodb-core`,`mysql`,`redis`,`winston`,`pg` et `pg-pool`. Pour plus d’informations sur la version de ces packages corrigés, consultez le fichier [diagnostic-channel-publishers’ README](https://github.com/Microsoft/node-diagnostic-channel/blob/master/src/diagnostic-channel-publishers/README.md).

Les correctifs `bunyan`, `winston` et `console` génèrent des événements de suivi Application Insights selon la `setAutoCollectConsole` activée. Le reste génère des événements de dépendance Application Insights selon que `setAutoCollectDependencies` est activé ou non.

### <a name="live-metrics"></a>Métriques temps réel

Pour activer l’envoi de métriques en temps réel de votre application vers Azure, utilisez `setSendLiveMetrics(true)`. Le filtrage des métriques en temps réel dans le portail n’est actuellement pas pris en charge.

### <a name="extended-metrics"></a>Métriques étendues

> [!NOTE]
> La possibilité d’envoyer des métriques natives étendues a été ajoutée dans la version 1.4.0

Pour activer l’envoi de métriques natives étendues de votre application vers Azure, installez le package de métriques natives distinct. Le Kit de développement logiciel (SDK) se charge automatiquement lorsqu’il est installé et démarre la collecte des métriques natives de Node.js.

```bash
npm install applicationinsights-native-metrics
```

Actuellement, le package de métriques natives effectue la collecte automatique du temps processeur du nettoyage de la mémoire, des cycles de boucle d’événements et de l’utilisation du tas :

- **Nettoyage de la mémoire** : La quantité de temps processeur passée sur chaque type de nettoyage de la mémoire, et le nombre d’occurrences de chaque type.
- **Boucle d’événements** : Le nombre de cycles qui se sont produits et le temps processeur passé au total.
- **Avec ou sans tas** : La quantité d’utilisation de la mémoire de votre application avec ou sans le tas.

### <a name="distributed-tracing-modes"></a>Modes de suivi distribué

Par défaut, le Kit de développement logiciel (SDK) enverra des en-têtes compréhensibles par d’autres applications/services instrumentés avec un Kit de développement logiciel (SDK) Application Insights. Vous pouvez éventuellement activer l’envoi/la réception d’en-têtes [W3C pour le contexte de trace](https://github.com/w3c/trace-context) en plus des en-têtes IA existants, de sorte que vous ne romprez pas la corrélation avec l’un de vos services hérités existants. L’activation des en-têtes W3C permettra à votre application de se mettre en corrélation avec d’autres services qui ne sont pas instrumentés avec Application Insights, mais qui adoptent cette norme W3C.

```Javascript
const appInsights = require("applicationinsights");
appInsights
  .setup("<your ikey>")
  .setDistributedTracingMode(appInsights.DistributedTracingModes.AI_AND_W3C)
  .start()
```

## <a name="telemetryclient-api"></a>API TelemetryClient

Pour une description complète de l’API TelemetryClient, consultez [API Application Insights pour les événements et les mesures personnalisés](./api-custom-events-metrics.md).

Vous pouvez suivre les demandes, les événements, les mesures ou les exceptions à l’aide du kit de développement logiciel (SDK) Application Insights Node.js. L’exemple de code suivant présente certaines des API que vous pouvez utiliser :

```javascript
let appInsights = require("applicationinsights");
appInsights.setup().start(); // assuming ikey in env var. start() can be omitted to disable any non-custom data
let client = appInsights.defaultClient;
client.trackEvent({name: "my custom event", properties: {customProperty: "custom property value"}});
client.trackException({exception: new Error("handled exceptions can be logged with this method")});
client.trackMetric({name: "custom metric", value: 3});
client.trackTrace({message: "trace message"});
client.trackDependency({target:"http://dbname", name:"select customers proc", data:"SELECT * FROM Customers", duration:231, resultCode:0, success: true, dependencyTypeName: "ZSQL"});
client.trackRequest({name:"GET /customers", url:"http://myserver/customers", duration:309, resultCode:200, success:true});

let http = require("http");
http.createServer( (req, res) => {
  client.trackNodeHttpRequest({request: req, response: res}); // Place at the beginning of your request handler
});
```

### <a name="track-your-dependencies"></a>Suivre les dépendances

Utilisez le code suivant pour effectuer le suivi de vos dépendances :

```javascript
let appInsights = require("applicationinsights");
let client = new appInsights.TelemetryClient();

var success = false;
let startTime = Date.now();
// execute dependency call here....
let duration = Date.now() - startTime;
success = true;

client.trackDependency({target:"http://dbname", name:"select customers proc", data:"SELECT * FROM Customers", duration:duration, resultCode:0, success: true, dependencyTypeName: "ZSQL"});;
```

Un exemple d’utilitaire utilisant `trackMetric` pour mesurer le temps nécessaire à la planification de la boucle d’événements :  

```javascript
function startMeasuringEventLoop() {
  var startTime = process.hrtime();
  var sampleSum = 0;
  var sampleCount = 0;

  // Measure event loop scheduling delay
  setInterval(() => {
    var elapsed = process.hrtime(startTime);
    startTime = process.hrtime();
    sampleSum += elapsed[0] * 1e9 + elapsed[1];
    sampleCount++;
  }, 0);

  // Report custom metric every second
  setInterval(() => {
    var samples = sampleSum;
    var count = sampleCount;
    sampleSum = 0;
    sampleCount = 0;

    if (count > 0) {
      var avgNs = samples / count;
      var avgMs = Math.round(avgNs / 1e6);
      client.trackMetric({name: "Event Loop Delay", value: avgMs});
    }
  }, 1000);
}
```

### <a name="add-a-custom-property-to-all-events"></a>Ajouter une propriété personnalisée à tous les événements

Utilisez le code suivant pour ajouter une propriété personnalisée à tous les événements :

```javascript
appInsights.defaultClient.commonProperties = {
  environment: process.env.SOME_ENV_VARIABLE
};
```

### <a name="track-http-get-requests"></a>Suivre les demandes HTTP GET

Utilisez le code suivant pour effectuer le suivi manuel des demandes HTTP GET :

> [!NOTE]
> Toutes les demandes sont suivies par défaut. Pour désactiver la collecte automatique, appelez .setAutoCollectRequests(false) avant d’appeler start().

```javascript
appInsights.defaultClient.trackRequest({name:"GET /customers", url:"http://myserver/customers", duration:309, resultCode:200, success:true});
```

Vous pouvez également suivre les requêtes à l’aide de la méthode `trackNodeHttpRequest` :

```javascript
var server = http.createServer((req, res) => {
  if ( req.method === "GET" ) {
      appInsights.defaultClient.trackNodeHttpRequest({request:req, response:res});
  }
  // other work here....
  res.end();
});
```

### <a name="track-server-startup-time"></a>Suivre le temps de démarrage du serveur

Utilisez le code suivant pour effectuer le suivi du temps de démarrage du serveur :

```javascript
let start = Date.now();
server.on("listening", () => {
  let duration = Date.now() - start;
  appInsights.defaultClient.trackMetric({name: "server startup time", value: duration});
});
```

### <a name="preprocess-data-with-telemetry-processors"></a>Prétraiter des données avec des processeurs de télémétrie

Vous pouvez traiter et filtrer les données collectées avant leur envoi en vue de leur rétention à l’aide de *processeurs de télémétrie*. Les processeurs de télémétrie sont appelés un par un dans l’ordre dans lequel ils ont été ajoutés avant l’envoi de l’élément de télémétrie vers le cloud.

```javascript
public addTelemetryProcessor(telemetryProcessor: (envelope: Contracts.Envelope, context: { http.RequestOptions, http.ClientRequest, http.ClientResponse, correlationContext }) => boolean)
```

Si un processeur de télémétrie retourne la valeur false, cet élément de télémétrie ne sera pas envoyé.

Tous les processeurs de télémétrie reçoivent des données de télémétrie et leur enveloppe à inspecter et à modifier. Ils reçoivent également un objet de contexte. Le contenu de cet objet est défini par le paramètre `contextObjects` lors de l’appel d’une méthode de suivi pour la télémétrie suivie manuellement. Pour les données de télémétrie collectées automatiquement, cet objet est rempli avec les informations de requête disponibles et le contenu des requêtes persistantes, tel qu’il est fourni par `appInsights.getCorrelationContext()` (si la corrélation de dépendance automatique est activée).

Le type de TypeScript pour un processeur de télémétrie est le suivant :

```javascript
telemetryProcessor: (envelope: ContractsModule.Contracts.Envelope, context: { http.RequestOptions, http.ClientRequest, http.ClientResponse, correlationContext }) => boolean;
```

Par exemple, un processeur qui supprime les données de trace des exceptions peut être écrit et ajouté comme suit :

```javascript
function removeStackTraces ( envelope, context ) {
  if (envelope.data.baseType === "Microsoft.ApplicationInsights.ExceptionData") {
    var data = envelope.data.baseData;
    if (data.exceptions && data.exceptions.length > 0) {
      for (var i = 0; i < data.exceptions.length; i++) {
        var exception = data.exceptions[i];
        exception.parsedStack = null;
        exception.hasFullStack = false;
      }
    }
  }
  return true;
}

appInsights.defaultClient.addTelemetryProcessor(removeStackTraces);
```

## <a name="use-multiple-instrumentation-keys"></a>Utilisation de plusieurs clés d’instrumentation

Vous pouvez créer plusieurs ressources Application Insights et envoyer différentes données à chacune d’entre elles à l’aide de leurs clés d’instrumentation respectives (« iClé »).

 Par exemple :

```javascript
let appInsights = require("applicationinsights");

// configure auto-collection under one ikey
appInsights.setup("_ikey-A_").start();

// track some events manually under another ikey
let otherClient = new appInsights.TelemetryClient("_ikey-B_");
otherClient.trackEvent({name: "my custom event"});
```

## <a name="advanced-configuration-options"></a>Options de configuration avancées

L’objet client contient une propriété `config` avec de nombreux paramètres facultatifs pour les scénarios avancés. Ils peuvent être définis comme suit :

```javascript
client.config.PROPERTYNAME = VALUE;
```

Ces propriétés étant spécifiques au client, vous pouvez configurer `appInsights.defaultClient` séparément des clients créés avec `new appInsights.TelemetryClient()`.

| Propriété                        | Description                                                                                                |
| ------------------------------- |------------------------------------------------------------------------------------------------------------|
| instrumentationKey              | Un identificateur pour votre ressource Application Insights.                                                      |
| endpointUrl                     | Le point de terminaison d’ingestion auquel envoyer les charges utiles de télémétrie.                                                      |
| quickPulseHost                  | L’hôte Flux de métriques temps réel auquel envoyer la télémétrie des métriques en temps réel.                                            |
| proxyHttpUrl                    | Un serveur proxy pour le trafic HTTP du Kit de développement logiciel (SDK) (facultatif, extrait par défaut de la variable d’environnement `http_proxy`).     |
| proxyHttpsUrl                   | Un serveur proxy pour le trafic HTTPS du Kit de développement logiciel (SDK) (facultatif, extrait par défaut de la variable d’environnement `https_proxy`).   |
| httpAgent                       | Un agent HTTP à utiliser pour le trafic HTTP du Kit de développement logiciel (SDK) (facultatif, non défini par défaut).                                   |
| httpsAgent                      | Un Agent HTTP à utiliser pour le trafic HTTPS du Kit de développement logiciel (SDK) (facultatif, non défini par défaut).                                 |
| maxBatchSize                    | Le nombre maximal d’éléments de télémétrie à inclure dans une charge utile pour le point de terminaison d’ingestion (`250` par défaut).   |
| maxBatchIntervalMs              | Le délai d’attente maximal pour qu’une charge utile atteigne la maxBatchSize (`15000` par défaut).               |
| disableAppInsights              | Un indicateur spécifiant si la transmission de télémétrie est désactivée (`false` par défaut).                                 |
| samplingPercentage              | Le pourcentage d’éléments de télémétrie suivis qui doivent être transmis (`100` par défaut).                      |
| correlationIdRetryIntervalMs    | Le délai d’attente avant une nouvelle tentative de récupération de l’ID pour la corrélation entre composants (`30000` par défaut).     |
| correlationHeaderExcludedDomains| Une liste des domaines à exclure de l’injection d’en-tête de la corrélation entre composants (par défaut, consultez [Config.ts](https://github.com/Microsoft/ApplicationInsights-node.js/blob/develop/Library/Config.ts)).|

## <a name="next-steps"></a>Étapes suivantes

* [Navigation et tableaux de bord dans le portail Application Insights](./overview-dashboard.md)
* [Écrire des requêtes Analytics via vos données de télémétrie](../log-query/log-analytics-tutorial.md)

<!--references-->

[portal]: https://portal.azure.com/
[FAQ]: ../faq.md