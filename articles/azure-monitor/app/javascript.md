---
title: Azure Application Insights pour les applications web JavaScript
description: Obtenir les nombres de sessions et d’affichage de page, les données de client web, les applications monopages (SPA) et les modèles d’utilisation de suivi. Détection des problèmes de performances et des exceptions dans les pages Web JavaScript.
ms.topic: conceptual
ms.date: 08/06/2020
ms.custom: devx-track-js
ms.openlocfilehash: 0588a3eac4ced6cec1e7aea431c6555bbe8bff0a
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97559877"
---
# <a name="application-insights-for-web-pages"></a>Application Insights pour les pages web

Apprenez-en plus sur les performances et l’utilisation de votre page web ou de votre application. Ajoutez [Application Insights](app-insights-overview.md) à votre script de page pour obtenir le minutage des chargements de page et des appels AJAX, le nombre d’exceptions du navigateur et d’échecs d’AJAX et leurs détails, ainsi que les nombres d’utilisateurs et de sessions. Toutes ces données peuvent être segmentées par page, par version de système d’exploitation ou de navigateur client, par emplacement géographique et en fonction d’autres aspects. Vous pouvez définir des alertes en cas de dépassement d’un certain nombre d’échecs ou de ralentissement du chargement des pages. Et en insérant des suivis d’appel dans votre code JavaScript, vous pouvez suivre l’utilisation des différentes fonctionnalités de votre application de page web.

Vous pouvez utiliser Application Insights avec toutes les pages web ; il vous suffit pour cela d’ajouter un court extrait de code JavaScript. Si votre service web est [Java](java-get-started.md) ou [ASP.NET](asp-net.md), vous pouvez utiliser les kits SDK côté serveur conjointement au kit SDK JavaScript côté client pour acquérir une compréhension de bout en bout des performances de votre application.

## <a name="adding-the-javascript-sdk"></a>Ajout du SDK JavaScript

> [!IMPORTANT]
> Les nouvelles régions Azure **exigent** l’utilisation de chaînes de connexion au lieu de clés d’instrumentation. Une [chaîne de connexion](./sdk-connection-string.md?tabs=js) identifie la ressource à laquelle vous souhaitez associer vos données de télémétrie. Elle vous permet également de modifier les points de terminaison que votre ressource utilisera comme destination pour votre télémétrie. Vous devrez copier la chaîne de connexion et l’ajouter au code de votre application ou à une variable d’environnement.

1. Tout d’abord, vous avez besoin d’une ressource Application Insights. Si vous ne disposez pas encore d’une ressource ni d’une clé d’instrumentation, suivez les [instructions permettant de créer une ressource](create-new-resource.md).
2. Copiez la _clé d’instrumentation_ (aussi connue sous « iKey ») ou la [chaîne de connexion](#connection-string-setup) de la ressource à laquelle vous souhaitez envoyer vos données de télémétrie JavaScript (de l’étape 1). Vous l’ajouterez au paramètre `instrumentationKey` ou `connectionString` du Kit de développement logiciel (SDK) JavaScript Application Insights.
3. Ajoutez le kit SDK JavaScript Application Insights à votre page web ou à votre application via l’une des deux options suivantes :
    * [Configuration basée sur npm](#npm-based-setup)
    * [Extrait de code JavaScript](#snippet-based-setup)

> [!IMPORTANT]
> N’utilisez qu’une seule méthode pour ajouter le SDK JavaScript à votre application. Si vous utilisez l’installation NPM, n’utilisez pas l’extrait de code, et inversement.

> [!NOTE]
> Le programme d’installation de NPM installe le SDK JavaScript en tant que dépendance à votre projet, en activant IntelliSense, tandis que l’extrait de code extrait le SDK au moment de l’exécution. Les deux prennent en charge les mêmes fonctionnalités. Cependant, les développeurs qui souhaitent des événements et une configuration plus personnalisés choisissent généralement l’installation NPM, tandis que les utilisateurs recherchant une activation rapide de l’analytique web prête à l’emploi choisissent l’extrait de code.

### <a name="npm-based-setup"></a>Configuration basée sur npm

Installez via NPM.

```sh
npm i --save @microsoft/applicationinsights-web
```

> [!Note]
> **Les frappes sont incluses dans ce package**. Vous n’avez donc **pas** besoin d’installer un package de types distinct.
    
```js
import { ApplicationInsights } from '@microsoft/applicationinsights-web'

const appInsights = new ApplicationInsights({ config: {
  instrumentationKey: 'YOUR_INSTRUMENTATION_KEY_GOES_HERE'
  /* ...Other Configuration Options... */
} });
appInsights.loadAppInsights();
appInsights.trackPageView(); // Manually call trackPageView to establish the current user/session/pageview
```

### <a name="snippet-based-setup"></a>Configuration basée sur un extrait

Si votre application n’utilise pas npm, vous pouvez directement instrumenter vos pages web avec Application Insights en collant cet extrait de code en haut de chacune de vos pages. De préférence, il doit s’agir du premier script dans votre section `<head>` pour qu’il puisse surveiller tous les problèmes potentiels liés à l’ensemble de vos dépendances et, en option, toutes les erreurs JavaScript. Si vous utilisez l’application Blazor Server App, ajoutez l’extrait de code en haut du fichier `_Host.cshtml`, dans la section `<head>`.

Pour faciliter le suivi de la version de l’extrait de code que votre application utilise, à partir de la version 2.5.5 l’événement d’affichage de la page inclut une nouvelle balise « ai.internal.snippet » qui contient la version de l’extrait de code identifié.

L’extrait de code actuel (répertorié ci-dessous) sera identifié comme version « 3 ».

```html
<script type="text/javascript">
!function(T,l,y){var S=T.location,u="script",k="instrumentationKey",D="ingestionendpoint",C="disableExceptionTracking",E="ai.device.",I="toLowerCase",b="crossOrigin",w="POST",e="appInsightsSDK",t=y.name||"appInsights";(y.name||T[e])&&(T[e]=t);var n=T[t]||function(d){var g=!1,f=!1,m={initialize:!0,queue:[],sv:"4",version:2,config:d};function v(e,t){var n={},a="Browser";return n[E+"id"]=a[I](),n[E+"type"]=a,n["ai.operation.name"]=S&&S.pathname||"_unknown_",n["ai.internal.sdkVersion"]="javascript:snippet_"+(m.sv||m.version),{time:function(){var e=new Date;function t(e){var t=""+e;return 1===t.length&&(t="0"+t),t}return e.getUTCFullYear()+"-"+t(1+e.getUTCMonth())+"-"+t(e.getUTCDate())+"T"+t(e.getUTCHours())+":"+t(e.getUTCMinutes())+":"+t(e.getUTCSeconds())+"."+((e.getUTCMilliseconds()/1e3).toFixed(3)+"").slice(2,5)+"Z"}(),iKey:e,name:"Microsoft.ApplicationInsights."+e.replace(/-/g,"")+"."+t,sampleRate:100,tags:n,data:{baseData:{ver:2}}}}var h=d.url||y.src;if(h){function a(e){var t,n,a,i,r,o,s,c,p,l,u;g=!0,m.queue=[],f||(f=!0,t=h,s=function(){var e={},t=d.connectionString;if(t)for(var n=t.split(";"),a=0;a<n.length;a++){var i=n[a].split("=");2===i.length&&(e[i[0][I]()]=i[1])}if(!e[D]){var r=e.endpointsuffix,o=r?e.location:null;e[D]="https://"+(o?o+".":"")+"dc."+(r||"services.visualstudio.com")}return e}(),c=s[k]||d[k]||"",p=s[D],l=p?p+"/v2/track":config.endpointUrl,(u=[]).push((n="SDK LOAD Failure: Failed to load Application Insights SDK script (See stack for details)",a=t,i=l,(o=(r=v(c,"Exception")).data).baseType="ExceptionData",o.baseData.exceptions=[{typeName:"SDKLoadFailed",message:n.replace(/\./g,"-"),hasFullStack:!1,stack:n+"\nSnippet failed to load ["+a+"] -- Telemetry is disabled\nHelp Link: https://go.microsoft.com/fwlink/?linkid=2128109\nHost: "+(S&&S.pathname||"_unknown_")+"\nEndpoint: "+i,parsedStack:[]}],r)),u.push(function(e,t,n,a){var i=v(c,"Message"),r=i.data;r.baseType="MessageData";var o=r.baseData;return o.message='AI (Internal): 99 message:"'+("SDK LOAD Failure: Failed to load Application Insights SDK script (See stack for details) ("+n+")").replace(/\"/g,"")+'"',o.properties={endpoint:a},i}(0,0,t,l)),function(e,t){if(JSON){var n=T.fetch;if(n&&!y.useXhr)n(t,{method:w,body:JSON.stringify(e),mode:"cors"});else if(XMLHttpRequest){var a=new XMLHttpRequest;a.open(w,t),a.setRequestHeader("Content-type","application/json"),a.send(JSON.stringify(e))}}}(u,l))}function i(e,t){f||setTimeout(function(){!t&&m.core||a()},500)}var e=function(){var n=l.createElement(u);n.src=h;var e=y[b];return!e&&""!==e||"undefined"==n[b]||(n[b]=e),n.onload=i,n.onerror=a,n.onreadystatechange=function(e,t){"loaded"!==n.readyState&&"complete"!==n.readyState||i(0,t)},n}();y.ld<0?l.getElementsByTagName("head")[0].appendChild(e):setTimeout(function(){l.getElementsByTagName(u)[0].parentNode.appendChild(e)},y.ld||0)}try{m.cookie=l.cookie}catch(p){}function t(e){for(;e.length;)!function(t){m[t]=function(){var e=arguments;g||m.queue.push(function(){m[t].apply(m,e)})}}(e.pop())}var n="track",r="TrackPage",o="TrackEvent";t([n+"Event",n+"PageView",n+"Exception",n+"Trace",n+"DependencyData",n+"Metric",n+"PageViewPerformance","start"+r,"stop"+r,"start"+o,"stop"+o,"addTelemetryInitializer","setAuthenticatedUserContext","clearAuthenticatedUserContext","flush"]),m.SeverityLevel={Verbose:0,Information:1,Warning:2,Error:3,Critical:4};var s=(d.extensionConfig||{}).ApplicationInsightsAnalytics||{};if(!0!==d[C]&&!0!==s[C]){method="onerror",t(["_"+method]);var c=T[method];T[method]=function(e,t,n,a,i){var r=c&&c(e,t,n,a,i);return!0!==r&&m["_"+method]({message:e,url:t,lineNumber:n,columnNumber:a,error:i}),r},d.autoExceptionInstrumented=!0}return m}(y.cfg);(T[t]=n).queue&&0===n.queue.length&&n.trackPageView({})}(window,document,{
src: "https://az416426.vo.msecnd.net/scripts/b/ai.2.min.js", // The SDK URL Source
//name: "appInsights", // Global SDK Instance name defaults to "appInsights" when not supplied
//ld: 0, // Defines the load delay (in ms) before attempting to load the sdk. -1 = block page load and add to head. (default) = 0ms load after timeout,
//useXhr: 1, // Use XHR instead of fetch to report failures (if available),
//crossOrigin: "anonymous", // When supplied this will add the provided value as the cross origin attribute on the script tag 
cfg: { // Application Insights Configuration
    instrumentationKey: "YOUR_INSTRUMENTATION_KEY_GOES_HERE"
    /* ...Other Configuration Options... */
}});
</script>
```

> [!NOTE]
> Pour une meilleure lisibilité et pour réduire les erreurs JavaScript possibles, toutes les options de configuration possibles sont répertoriées sur une nouvelle ligne dans l’extrait de code ci-dessus, si vous ne souhaitez pas modifier la valeur d’une ligne commentée, elle peut être supprimée.


#### <a name="reporting-script-load-failures"></a>Échecs de chargement de script de rapport

Cette version de l’extrait de code détecte et signale les échecs lors du chargement du kit de développement logiciel (SDK) à partir de CDN en tant qu’exception au portail de Azure Monitor (sous le navigateur des échecs &gt; exceptions &gt;), cette exception fournit une visibilité des défaillances de ce type pour vous avertir que votre application ne signale pas les données de télémétrie (ou d’autres exceptions). Ce signal est une mesure importante pour comprendre que vous avez perdu la télémétrie, car le kit de développement logiciel (SDK) n’a pas pu charger ou initialiser, ce qui peut entraîner les opérations suivantes :
- Sous-rapport sur la manière dont les utilisateurs utilisent (ou essaient d’utiliser) votre site ;
- Télémétrie manquante sur la façon dont vos utilisateurs finaux utilisent votre site ;
- Il manque des erreurs JavaScript qui pourraient potentiellement empêcher vos utilisateurs finaux d’utiliser correctement votre site.

Pour plus d’informations sur cette exception, consultez la page de résolution des problèmes [Échec du chargement du kit de développement logiciel](javascript-sdk-load-failure.md).

Le signalement de cet échec en tant qu’exception au portail n’utilise pas l’option de configuration ```disableExceptionTracking``` de la configuration d’Application Insights et, par conséquent, si cet échec se produit, il est toujours signalé par l’extrait de code, même lorsque la prise en charge de window.onerror est désactivée.

La création de rapports sur les échecs de chargement du SDK n’est pas prise en charge spécifiquement sur IE 8 (ou moins). Cela permet de réduire la taille minimisés de l’extrait de code en supposant que la plupart des environnements ne sont pas exclusivement IE 8 ou moins. Si vous avez cette exigence et que vous souhaitez recevoir ces exceptions, vous devez inclure une version Fetch de poly Fill ou créer votre propre extrait de code qui utilise ```XDomainRequest``` au lieu de ```XMLHttpRequest```, il est recommandé d’utiliser le [code source de l’extrait de code fourni](https://github.com/microsoft/ApplicationInsights-JS/blob/master/AISKU/snippet/snippet.js) comme point de départ.

> [!NOTE]
> Si vous utilisez une version précédente de l’extrait de code, il est vivement recommandé d’effectuer la mise à jour vers la dernière version afin que vous receviez ces problèmes précédemment non signalés.

#### <a name="snippet-configuration-options"></a>Options de configuration de l’extrait de code

Toutes les options de configuration ont été déplacées vers la fin du script afin d’éviter d’introduire accidentellement des erreurs JavaScript qui n’entraînent pas l’échec du chargement du kit de développement logiciel (SDK). Ces erreurs pourraient également désactiver la création de rapports de l’échec.

Chaque option de configuration est présentée ci-dessus sur une nouvelle ligne, si vous ne souhaitez pas remplacer la valeur par défaut d’un élément répertoriée comme [facultatif], vous pouvez supprimer cette ligne afin de réduire la taille résultante de la page retournée.

Voici les options de configuration disponibles

| Nom | Type | Description
|------|------|----------------
| src | chaîne **[obligatoire]** | URL complète de l’emplacement à partir duquel charger le kit de développement logiciel (SDK). Cette valeur est utilisée pour l’attribut « src » d’une balise &lt;script/&gt; ajoutée dynamiquement. Vous pouvez utiliser l’emplacement de CDN public ou votre propre hébergement privé.
| name | chaîne *[facultatif]* | Le nom global du Kit de développement logiciel (SDK) initialisé est par défaut `appInsights`. ```window.appInsights``` sera donc une référence à l’instance initialisée. Remarque : Si vous fournissez une valeur de nom ou qu’une instance précédente semble être assignée (via le nom global appInsightsSDK), cette valeur de nom sera également définie dans l’espace de noms global comme ```window.appInsightsSDK=<name value>```, ce qui est requis par le code d’initialisation du kit de développement logiciel (SDK) pour garantir son initialisation et la mise à jour du squelette d’extrait de code et des méthodes de proxy.
| ld | nombre en ms *[facultatif]* | Définit le délai de chargement avant de tenter de charger le kit de développement logiciel (SDK). La valeur par défaut est 0 ms et toute valeur négative ajoute immédiatement une balise de script sue l’&lt;en-tête&gt; région de la page, ce qui bloque l’événement de chargement de la page jusqu’à ce que le script soit chargé (ou échoue).
| useXhr | booléen *[facultatif]* | Ce paramètre est utilisé uniquement pour les échecs de chargement du kit de développement logiciel (SDK). La création de rapports tente d’abord d’utiliser la récupération (fetch) () si elle est disponible, puis de revenir à XHR, en définissant cette valeur sur true pour simplement ignorer la vérification de l’extraction. L’utilisation de cette valeur est requise uniquement si votre application est utilisée dans un environnement où la récupération (fetch) ne parviendrait pas à envoyer les événements d’échec.
| crossOrigin | chaîne *[facultatif]* | Si vous incluez ce paramètre, la balise de script ajoutée pour télécharger le kit de développement logiciel (SDK) inclut l’attribut crossOrigin avec cette valeur de chaîne. Lorsqu’il n’est pas défini (valeur par défaut), aucun attribut crossOrigin n’est ajouté. Les valeurs recommandées ne sont pas définies (valeur par défaut) ; "" ; ou « anonymous » (pour toutes les valeurs valides, consultez [Attribut HTML : documentation `crossorigin`](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/crossorigin))
| cfg | Objet **[obligatoire]** | La configuration est passée au Kit de développement logiciel (SDK) Application Insights pendant l’initialisation.

### <a name="connection-string-setup"></a>Configuration de la chaîne de connexion

Pour la configuration de NPM ou de l’extrait de code, vous pouvez également configurer votre instance d’Application Insights à l’aide d’une chaîne de connexion. Remplacez simplement le champ `instrumentationKey` par le champ `connectionString`.
```js
import { ApplicationInsights } from '@microsoft/applicationinsights-web'

const appInsights = new ApplicationInsights({ config: {
  connectionString: 'YOUR_CONNECTION_STRING_GOES_HERE'
  /* ...Other Configuration Options... */
} });
appInsights.loadAppInsights();
appInsights.trackPageView();
```

### <a name="sending-telemetry-to-the-azure-portal"></a>Envoi de données de télémétrie au portail Azure

Par défaut, le kit SDK JavaScript Application Insights recueille un certain nombre d’éléments de télémétrie qui sont utiles pour déterminer l’intégrité de votre application et l’expérience utilisateur sous-jacente. notamment :

- **Exceptions non interceptées** dans votre application, y compris des informations sur
    - Arborescence des appels de procédure
    - Détails de l’exception et message accompagnant l’erreur
    - Numéros de ligne et de colonne de l’erreur
    - URL où l’erreur a été générée
- **Demandes de dépendance réseau** effectuées par vos demandes **XHR** et **Fetch** d’application (la collection de recherches est désactivée par défaut), y compris des informations sur
    - URL de la source de dépendance
    - Commande et méthode utilisées pour demander la dépendance
    - Durée de la demande
    - Code de résultat et état de réussite de la demande
    - ID (le cas échéant) de l’utilisateur qui effectue la demande
    - Contexte de corrélation (le cas échéant) dans lequel la demande est effectuée
- **Informations utilisateur** (par exemple, emplacement, réseau, adresse IP)
- **Informations sur l’appareil** (par exemple, navigateur, système d’exploitation, version, langue, modèle)
- **Informations de session**

### <a name="telemetry-initializers"></a>Initialiseurs de télémétrie
Les initialiseurs de télémétrie permettent de modifier le contenu des données de télémétrie collectées avant leur envoi à partir du navigateur de l’utilisateur. Ils peuvent également être utilisés pour empêcher l’envoi de certaines données de télémétrie, en retournant `false`. Plusieurs initialiseurs de télémétrie peuvent être ajoutés à votre instance d’Application Insights et ils seront exécutés dans l’ordre où ils sont ajoutés.

L’argument d’entrée pour `addTelemetryInitializer` est un rappel qui prend un [`ITelemetryItem`](https://github.com/microsoft/ApplicationInsights-JS/blob/master/API-reference.md#addTelemetryInitializer) comme argument et retourne un `boolean` ou un `void`. Si `false` est retourné, l’élément de télémétrie n’est pas envoyé, sinon il passe à l’initialiseur de télémétrie suivant, le cas échéant, ou il est envoyé au point de terminaison de collecte de données de télémétrie.

Voici un exemple d’utilisation des initialiseurs de télémétrie :
```ts
var telemetryInitializer = (envelope) => {
  envelope.data.someField = 'This item passed through my telemetry initializer';
};
appInsights.addTelemetryInitializer(telemetryInitializer);
appInsights.trackTrace({message: 'This message will use a telemetry initializer'});

appInsights.addTelemetryInitializer(() => false); // Nothing is sent after this is executed
appInsights.trackTrace({message: 'this message will not be sent'}); // Not sent
```

## <a name="configuration"></a>Configuration
La plupart des champs de configuration sont nommés de façon à pouvoir avoir la valeur false par défaut. Tous les champs sont facultatifs à l’exception de `instrumentationKey`.

| Nom | Default | Description |
|------|---------|-------------|
| instrumentationKey | null | **Obligatoire**<br>Clé d’instrumentation que vous avez obtenue à partir du portail Azure. |
| accountId | null | ID de compte facultatif, si votre application regroupe les utilisateurs dans des comptes. Aucun espace, virgule, point-virgule, signe d’égalité ni barre verticale |
| sessionRenewalMs | 1800000 | Une session est consignée si l’utilisateur est inactif pendant ce laps de temps en millisecondes. La valeur par défaut est 30 minutes |
| sessionExpirationMs | 86400000 | Une session est consignée si elle s’est poursuivie pendant ce laps de temps en millisecondes. La valeur par défaut est 24 heures |
| maxBatchSizeInBytes | 10000 | Taille maximale du lot de données de télémétrie. Si un lot dépasse cette limite, il est immédiatement envoyé et un nouveau lot est démarré |
| maxBatchInterval | 15000 | Durée de mise en lot des données de télémétrie avant l’envoi (en millisecondes) |
| disableExceptionTracking | false | Si la valeur est true, les exceptions ne sont pas collectées automatiquement. La valeur par défaut est false. |
| disableTelemetry | false | Si la valeur est true, les données de télémétrie ne sont pas collectées ni envoyées. La valeur par défaut est false. |
| enableDebug | false | Si la valeur est true, les données de débogage **internes** sont levées en tant qu’exception **au lieu** d’être consignées, quels que soient les paramètres de journalisation du kit SDK. La valeur par défaut est false. <br>**_Remarque :_* _ L’activation de ce paramètre entraîne la suppression de données de télémétrie chaque fois qu’une erreur interne se produit. Cela peut être utile pour identifier rapidement les problèmes liés à votre configuration ou utilisation du kit SDK. Pour ne pas perdre de données de télémétrie pendant le débogage, utilisez `consoleLoggingLevel` ou `telemetryLoggingLevel` à la place de `enableDebug`. |
| loggingLevelConsole | 0 | Consigne les erreurs _ *internes** d’Application Insights dans la console. <br>0 : désactivé, <br>1 : Erreurs critiques uniquement, <br>2 : Tout (erreurs et avertissements) |
| loggingLevelTelemetry | 1 | Envoie les erreurs **internes** d’Application Insights en tant que données de télémétrie. <br>0 : désactivé, <br>1 : Erreurs critiques uniquement, <br>2 : Tout (erreurs et avertissements) |
| diagnosticLogInterval | 10000 | (interne) Intervalle d’interrogation (en ms) pour la file d’attente de journalisation interne |
| samplingPercentage | 100 | Pourcentage d’événements qui seront envoyés. La valeur par défaut est 100, ce qui signifie que tous les événements sont envoyés. Définissez cette option si vous souhaitez conserver votre plafond de données pour les applications à grande échelle. |
| autoTrackPageVisitTime | false | Si la valeur est true, sur une consultation de page, la durée d’affichage de la page instrumentée précédente fait l’objet d’un suivi et est envoyée en tant que données de télémétrie, et un nouveau minuteur est démarré pour la consultation de page en cours. La valeur par défaut est false. |
| disableAjaxTracking | false | Si la valeur est true, les appels Ajax ne sont pas collectés automatiquement. La valeur par défaut est false. |
| disableFetchTracking | true | Si la valeur est true, les demandes Fetch ne sont pas collectées automatiquement. La valeur par défaut est true |
| overridePageViewDuration | false | Si la valeur est true, le comportement par défaut de trackPageView est modifié pour enregistrer la fin de l’intervalle de durée de consultation de page lorsque trackPageView est appelé. Si la valeur est false et qu’aucune durée personnalisée n’est fournie à trackPageView, les performances d’affichage de la page sont calculées à l’aide de l’API de minutage de la navigation. La valeur par défaut est false. |
| maxAjaxCallsPerView | 500 | Valeur par défaut 500 - contrôle le nombre d’appels Ajax qui seront supervisés par affichage de page. Affectez la valeur -1 pour superviser tous les appels Ajax (illimités) dans la page. |
| disableDataLossAnalysis | true | Si la valeur est false, les mémoires tampons d’expéditeur de données de télémétrie internes sont vérifiées au démarrage à la recherche d’éléments qui n’ont pas encore été envoyés. |
| disableCorrelationHeaders | false | Si la valeur est false, le kit SDK ajoute deux en-têtes ('Request-Id' et 'Request-Context') à toutes les demandes de dépendance pour les mettre en corrélation avec les demandes correspondantes côté serveur. La valeur par défaut est false. |
| correlationHeaderExcludedDomains |  | Désactiver les en-têtes de corrélation pour des domaines spécifiques |
| correlationHeaderDomains |  | Activer les en-têtes de corrélation pour des domaines spécifiques |
| disableFlushOnBeforeUnload | false | La valeur par défaut est false. Si la valeur est true, la méthode Flush n’est pas appelée lorsque l’événement onBeforeUnload est déclenché |
| enableSessionStorageBuffer | true | La valeur par défaut est true. Si la valeur est true, la mémoire tampon contenant toutes les données de télémétrie non envoyées est stockée dans le stockage de session. La mémoire tampon est restaurée lors du chargement de la page |
| isCookieUseDisabled | false | La valeur par défaut est false. Si la valeur est true, le kit SDK ne stocke pas ou ne lit pas les données des cookies.|
| cookieDomain | null | Domaine de cookie personnalisé. Cela est utile si vous souhaitez partager des cookies Application Insights entre les sous-domaines. |
| isRetryDisabled | false | La valeur par défaut est false. Si la valeur est false, réessayez avec 206 (succès partiel), 408 (délai d’expiration), 429 (trop de requêtes), 500 (erreur de serveur interne), 503 (service non disponible) et 0 (hors connexion, uniquement si détecté) |
| isStorageUseDisabled | false | Si la valeur est true, le kit SDK ne stocke pas ou ne lit pas les données à partir du stockage local et de session. La valeur par défaut est false. |
| isBeaconApiDisabled | true | Si la valeur est false, le kit SDK envoie toutes les données de télémétrie à l’aide de l’[API Beacon](https://www.w3.org/TR/beacon) |
| onunloadDisableBeacon | false | La valeur par défaut est false. Quand l’onglet est fermé, le SDK envoie toutes les données de télémétrie restantes à l’aide de l’[API Beacon](https://www.w3.org/TR/beacon). |
| sdkExtension | null | Définit le nom de l’extension du kit SDK. Seuls les caractères alphabétiques sont autorisés. Le nom de l’extension est ajouté comme préfixe à la balise 'ai.internal.sdkVersion' (par exemple, 'ext_javascript:2.0.0'). La valeur par défaut est Null. |
| isBrowserLinkTrackingEnabled | false | La valeur par défaut est false. Si la valeur est true, le kit SDK effectue le suivi de toutes les demandes de [lien de navigateur](/aspnet/core/client-side/using-browserlink). |
| appId | null | AppId est utilisé pour la corrélation entre les dépendances AJAX qui se produisent côté client avec les demandes côté serveur. Lorsque l’API Beacon est activée, elle ne peut pas être utilisée automatiquement, mais peut être définie manuellement dans la configuration. La valeur par défaut est Null |
| enableCorsCorrelation | false | Si la valeur est true, le kit SDK ajoute deux en-têtes ('Request-Id' et 'Request-Context') à toutes les demandes CORS pour mettre en corrélation les dépendances AJAX sortantes avec les demandes correspondantes côté serveur. La valeur par défaut est false |
| namePrefix | non défini | Valeur facultative qui sera utilisée comme suffixe de nom pour localStorage et le nom du cookie.
| enableAutoRouteTracking | false | Effectuer le suivi automatique des modifications de route dans les applications monopages (SPA). Si la valeur est true, chaque modification de route envoie un nouveau Pageview à Application Insights. Les modifications des routes de hachage (`example.com/foo#bar`) sont également enregistrées en tant que nouveaux affichages de page.
| enableRequestHeaderTracking | false | Si la valeur est true, les en-têtes de requête d’extraction AJAX & Fetch sont suivis, la valeur par défaut est false.
| enableResponseHeaderTracking | false | Si la valeur est true, les en-têtes de réponse de requête d’extraction AJAX & Fetch sont suivis, la valeur par défaut est false.
| distributedTracingMode | `DistributedTracingModes.AI` | Définit le mode de traçage distribué. Si le mode AI_AND_W3C ou le mode W3C sont définis, les en-têtes de contexte de trace W3C (traceparent/tracestate) sont générés et inclus dans toutes les demandes sortantes. AI_AND_W3C est fourni à des fins de compatibilité descendante avec tous les services instrumentés Application Insights hérités. Consultez l’exemple [ici](./correlation.md#enable-w3c-distributed-tracing-support-for-web-apps).
| enableAjaxErrorStatusText | false | La valeur par défaut est false. Si la valeur est true, inclure le texte des données d’erreur de réponse dans l’événement de dépendance sur les demandes AJAX ayant échoué.
| enableAjaxPerfTracking | false | La valeur par défaut est false. Indicateur pour activer la recherche et l’inclusion de minutages supplémentaires de window.performance du navigateur dans les métriques `ajax` (XHR et fetch) signalées.
| maxAjaxPerfLookupAttempts | 3 | La valeur par défaut est 3. Nombre maximal de fois où la fenêtre est recherchée. les minutages de performances (si disponibles) sont nécessaires, car tous les navigateurs remplissent la fenêtre de performances avant de signaler la fin de la demande XHR et les requêtes de récupération (fetch) sont ajoutées après son achèvement.
| ajaxPerfLookupDelay | 25 | La valeur par défaut est 25 ms. Délai d’attente avant la nouvelle tentative de recherche de minutages de windows.performance pour une requête `ajax`, le temps est exprimé en millisecondes et est transmis directement à setTimeout().
| enableUnhandledPromiseRejectionTracking | false | Si la valeur est true, les rejets de promesse non gérés sont collectés et signalés comme une erreur JavaScript. Quand disableExceptionTracking a la valeur true (ne pas suivre les exceptions), la valeur de configuration est ignorée et les rejets de promesse non gérés ne sont pas signalés.

## <a name="enable-time-on-page-tracking"></a>Activer le suivi du temps sur une page

En définissant `autoTrackPageVisitTime: true`, le temps que passe chaque utilisateur sur chaque page est suivi. Pour chaque nouveau PageView, le temps passé par l’utilisateur sur la page *précédente* est envoyée en tant que [métrique personnalisée](../platform/metrics-custom-overview.md) appelée `PageVisitTime`. Cette métrique personnalisée est affichable dans [Metrics Explorer](../platform/metrics-getting-started.md) en tant que « métrique basée sur le journal ».

## <a name="enable-correlation"></a>Activer la corrélation

La corrélation génère et envoie des données qui activent le suivi distribué et alimente la [cartographie d’application](../app/app-map.md), la [vue de transaction de bout en bout](../app/app-map.md#go-to-details) et d’autres outils de diagnostic.

L’exemple suivant montre toutes les configurations possibles requises pour activer la corrélation, avec les remarques spécifiques aux scénarios ci-dessous :

```javascript
// excerpt of the config section of the JavaScript SDK snippet with correlation
// between client-side AJAX and server requests enabled.
cfg: { // Application Insights Configuration
    instrumentationKey: "YOUR_INSTRUMENTATION_KEY_GOES_HERE"
    disableFetchTracking: false,
    enableCorsCorrelation: true,
    enableRequestHeaderTracking: true,
    enableResponseHeaderTracking: true,
    correlationHeaderExcludedDomains: ['myapp.azurewebsites.net', '*.queue.core.windows.net']
    /* ...Other Configuration Options... */
}});
</script>

``` 

Si l’un de vos serveurs tiers avec lequel le client communique ne peut pas accepter les en-têtes `Request-Id` et `Request-Context`, et que vous ne pouvez pas mettre à jour leur configuration, vous devez les placer dans une liste d’exclusion via la propriété de configuration `correlationHeaderExcludeDomains`. Cette propriété prend en charge les caractères génériques.

Le côté serveur doit être en mesure d’accepter les connexions avec ces en-têtes présents. Selon la configuration de `Access-Control-Allow-Headers` côté serveur, il est souvent nécessaire d’étendre la liste côté serveur en ajoutant manuellement `Request-Id` et `Request-Context`.

Access-Control-Allow-Headers : `Request-Id` `Request-Context`, `<your header>`

> [!NOTE]
> Si vous utilisez des kits de développement logiciel (SDK) OpenTelemtry ou Application Insights publiés en 2020 ou version ultérieure, nous vous recommandons d’utiliser [WC3 TraceContext](https://www.w3.org/TR/trace-context/). Consultez les instructions de configuration [ici](../app/correlation.md#enable-w3c-distributed-tracing-support-for-web-apps).

## <a name="single-page-applications"></a>Applications monopages

Par défaut, ce kit SDK **ne gère pas** les modifications de route basées sur l’état qui se produisent dans les applications monopages. Pour activer le suivi automatique des modifications de route pour votre application monopage, vous pouvez ajouter `enableAutoRouteTracking: true` à la configuration de votre installation.

Actuellement, nous proposons un [plug-in React](javascript-react-plugin.md) distinct que vous pouvez initialiser avec ce SDK. Il effectue également le suivi des modifications de route pour vous, ainsi que la collecte d’autres données de télémétrie spécifiques à React.
> [!NOTE]
> Utilisez `enableAutoRouteTracking: true` uniquement si vous n’utilisez **pas** le plug-in REACT. Tous deux peuvent envoyer de nouveaux PageViews lorsque la route change. Si les deux sont activés, il est possible que des PageViews en double soient envoyés.

## <a name="extensions"></a>Extensions

| Extensions |
|---------------|
| [React](javascript-react-plugin.md)|
| [React Native](javascript-react-native-plugin.md)|
| [Angular](javascript-angular-plugin.md) |

## <a name="explore-browserclient-side-data"></a>Explorer les données côté navigateur/client

Vous pouvez afficher les données côté navigateur/client en accédant à **Métriques** et en ajoutant les métriques individuelles qui vous intéressent :

![Capture d’écran de la page Métriques dans Application Insights montrant des affichages graphiques de données métriques pour une application web.](./media/javascript/page-view-load-time.png)

Vous pouvez également afficher vos données à partir du kit SDK JavaScript via l’expérience du navigateur dans le portail.

Sélectionnez **Navigateur**, puis choisissez **Échecs** ou **Performances**.

![Capture d’écran de la page Navigateur dans Application Insights montrant comment ajouter Échecs du navigateur ou Performances du navigateur aux métriques que vous pouvez afficher pour votre application web.](./media/javascript/browser.png)

### <a name="performance"></a>Performances

![Capture d’écran de la page Performances dans Application Insights montrant des affichages graphiques des métriques Opérations pour une application web.](./media/javascript/performance-operations.png)

### <a name="dependencies"></a>Les dépendances

![Capture d’écran de la page Performances dans Application Insights montrant des affichages graphiques des métriques Dépendances pour une application web.](./media/javascript/performance-dependencies.png)

### <a name="analytics"></a>Analytics

Pour interroger vos données de télémétrie collectées par le kit SDK JavaScript, sélectionnez le bouton **Voir dans les journaux (analytique)** . En ajoutant une instruction `where` égale à `client_Type == "Browser"`, vous verrez uniquement les données du kit SDK JavaScript et toutes les données de télémétrie côté serveur collectées par d’autres kits SDK seront exclues.
 
```kusto
// average pageView duration by name
let timeGrain=5m;
let dataset=pageViews
// additional filters can be applied here
| where timestamp > ago(1d)
| where client_Type == "Browser" ;
// calculate average pageView duration for all pageViews
dataset
| summarize avg(duration) by bin(timestamp, timeGrain)
| extend pageView='Overall'
// render result in a chart
| render timechart
```

### <a name="source-map-support"></a>Prise en charge du mappage de source

La pile d’appels minimisée de vos données de télémétrie d’exception peut être déminimisée dans le portail Azure. Toutes les intégrations existantes dans le panneau Détails des exceptions fonctionnent avec la pile d’appels nouvellement déminimisée.

#### <a name="link-to-blob-storage-account"></a>Lien vers un compte de stockage blob

Vous pouvez lier votre ressource Application Insights à votre propre conteneur de stockage blob Azure pour réduire automatiquement les piles d’appels. Pour commencer, consultez la [prise en charge automatique du mappage de source](./source-map-support.md).

### <a name="drag-and-drop"></a>Glisser-déposer

1. Sélectionnez un élément Télémétrie des exceptions dans le portail Azure pour afficher ses « détails de transaction de bout en bout ».
2. Identifiez les mappages de source qui correspondent à cette pile d’appels. Le mappage de source doit correspondre au fichier source d’un frame de pile, mais avec le suffixe `.map`
3. Glissez-déplacez les mappages de source sur la pile des appels dans le portail Azure ![An animated image showing how to drag and drop source map files from a build folder into the Call Stack window in the Azure portal.](https://i.imgur.com/Efue9nU.gif)

### <a name="application-insights-web-basic"></a>Version web de base d’Application Insights

Pour bénéficier d’une gestion allégée, vous pouvez installer à la place la version de base d’Application Insights
```
npm i --save @microsoft/applicationinsights-web-basic
```
Cette version fournit un strict minimum de fonctionnalités et vous laisse la compléter à votre guise. Par exemple, elle n’effectue pas de collecte automatique (exceptions non interceptées, AJAX, etc.). Les API permettant d’envoyer certains types de données de télémétrie, comme `trackTrace`, `trackException`, etc., ne sont pas incluses dans cette version et vous devrez fournir votre propre wrapper. La seule API disponible est `track`. Vous trouverez un [exemple](https://github.com/Azure-Samples/applicationinsights-web-sample1/blob/master/testlightsku.html) ici.

## <a name="examples"></a>Exemples

Pour obtenir des exemples exécutables, consultez [Exemples du SDK JavaScript Application Insights](https://github.com/Azure-Samples?q=applicationinsights-js-demo).

## <a name="upgrading-from-the-old-version-of-application-insights"></a>Mise à niveau de l’ancienne version d’Application Insights

Dernières modifications dans la version V2 du kit SDK :
- Pour permettre de meilleures signatures d’API, certains appels d’API, tels que trackPageView et trackException, ont été mis à jour. L’exécution dans Internet Explorer 8 ou des versions antérieures du navigateur n’est pas prise en charge.
- L’enveloppe de télémétrie présente des modifications de structure et de nom des champs en raison de mises à jour du schéma de données.
- `context.operation` a été déplacé vers `context.telemetryTrace`. Certains champs ont également été modifiés (`operation.id` --> `telemetryTrace.traceID`).
  - Pour actualiser manuellement l’ID de consultation de page actuel (par exemple, dans les applications SPA), utilisez `appInsights.properties.context.telemetryTrace.traceID = Util.generateW3CId()`.
    > [!NOTE]
    > Pour conserver l’ID de trace unique, dans lequel vous avez déjà utilisé `Util.newId()`, utilisez maintenant `Util.generateW3CId()`. Tous deux finissent par être l’ID d’opération.

Si vous utilisez le kit SDK de PRODUCTION d’Application Insights actuel (1.0.20) et que vous souhaitez voir si le nouveau kit SDK fonctionne dans le runtime, mettez à jour l’URL en fonction de votre scénario de chargement de kit SDK actuel.

- Scénario de téléchargement via CDN : Mettez à jour l’extrait de code que vous utilisez actuellement pour pointer vers l’URL suivante :
   ```
   "https://az416426.vo.msecnd.net/scripts/b/ai.2.min.js"
   ```

- Scénario npm : Appelez `downloadAndSetup` pour télécharger le script ApplicationInsights complet à partir de CDN et l’initialiser avec la clé d’instrumentation :

   ```ts
   appInsights.downloadAndSetup({
     instrumentationKey: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxx",
     url: "https://az416426.vo.msecnd.net/scripts/b/ai.2.min.js"
     });
   ```

Effectuez un test dans l’environnement interne pour vérifier que la télémétrie de surveillance fonctionne comme prévu. Si tout fonctionne, mettez à jour vos signatures d’API conformément à la version V2 du kit SDK et procédez au déploiement dans vos environnements de production.

## <a name="sdk-performanceoverhead"></a>Surcharge/performances du kit SDK

Avec seulement 36 Ko compressés avec gzip et en mettant seulement ~15 ms pour s’initialiser, Application Insights ajoute un temps de chargement négligeable à votre site web. Avec cet extrait de code, les composants minimaux de la bibliothèque sont rapidement chargés. En attendant, le script complet est téléchargé en arrière-plan.

Pendant le téléchargement du script à partir de CDN, tout le suivi de votre page est mis en file d’attente. Une fois que le script téléchargé a fini de s’initialiser de manière asynchrone, tous les événements qui ont été mis en file d’attente font l’objet d’un suivi. Par conséquent, vous ne perdrez aucune donnée de télémétrie au cours du cycle de vie complet de votre page. Ce processus de configuration fournit à votre page un système d’analyse transparent, invisible à vos utilisateurs.

> Résumé :
> - ![version npm](https://badge.fury.io/js/%40microsoft%2Fapplicationinsights-web.svg)
> - ![taille compressée de gzip](https://img.badgesize.io/https://js.monitor.azure.com/scripts/b/ai.2.min.js.svg?compression=gzip)
> - Durée d’initialisation globale de **15 ms**
> - **Aucun** suivi manqué pendant le cycle de vie de la page

## <a name="browser-support"></a>Prise en charge des navigateurs

![Chrome](https://raw.githubusercontent.com/alrra/browser-logos/master/src/chrome/chrome_48x48.png) | ![Firefox](https://raw.githubusercontent.com/alrra/browser-logos/master/src/firefox/firefox_48x48.png) | ![IE](https://raw.githubusercontent.com/alrra/browser-logos/master/src/edge/edge_48x48.png) | ![Opera](https://raw.githubusercontent.com/alrra/browser-logos/master/src/opera/opera_48x48.png) | ![Safari](https://raw.githubusercontent.com/alrra/browser-logos/master/src/safari/safari_48x48.png)
--- | --- | --- | --- | --- |
Chrome version la plus récente ✔ |  Firefox version la plus récente ✔ | IE 9+ et Edge ✔<br>Compatible à IE 8 | Opera version la plus récente ✔ | Safari version la plus récente ✔ |

## <a name="es3ie8-compatibility"></a>Compatibilité avec ES3/IE8

En tant que Kit de développement logiciel (SDK), de nombreux utilisateurs ne peuvent pas contrôler les navigateurs utilisés par leurs clients. Nous devons donc nous assurer que ce SDK continue à « fonctionner » et n’interrompt pas l’exécution de JS lorsqu’il est chargé par un navigateur plus ancien. Bien qu’il soit idéal de ne pas prendre en charge les navigateurs IE8 et ES3 (version antérieure), il existe de nombreux clients/utilisateurs qui continuent à demander des pages qui fonctionnent et comme indiqué, ils peuvent ou non contrôler le navigateur que leurs utilisateurs finaux choisissent d’utiliser.

Cela ne signifie pas que nous ne prenons en charge que l’ensemble commun de fonctionnalités les plus faibles, mais seulement que nous devons assurer la compatibilité du code ES3 et, lors de l’ajout de nouvelles fonctionnalités, vous devez ajouter de nouvelles fonctionnalités de manière à ce qu’elles n’empêchent pas l’analyse JavaScript ES3 et n’aient pas été ajoutées en tant que fonctionnalité facultative.

[Consulter GitHub pour obtenir des informations complètes sur la prise en charge d’IE8](https://github.com/Microsoft/ApplicationInsights-JS#es3ie8-compatibility)

## <a name="open-source-sdk"></a>Kit de développement logiciel (SDK) open source

Le kit SDK JavaScript Application Insights est proposé en open source. Pour afficher le code source ou contribuer au projet, visitez le [dépôt GitHub officiel](https://github.com/Microsoft/ApplicationInsights-JS). 

Pour obtenir les mises à jour et correctifs de bogues les plus récents, [consultez les notes de publication](./release-notes.md).

## <a name="next-steps"></a><a name="next"></a>Étapes suivantes
* [Suivi de l'utilisation](usage-overview.md)
* [Mesures et événements personnalisés](api-custom-events-metrics.md)
* [Développer-mesurer-apprendre](usage-overview.md)
* [Résoudre les problèmes de chargement du SDK](javascript-sdk-load-failure.md)
