---
title: Surveillance d’un site SharePoint avec Application Insights
description: Démarrage de la surveillance d'une nouvelle application avec une nouvelle clé d'instrumentation
ms.topic: conceptual
ms.date: 09/08/2020
ms.openlocfilehash: f07c317a61c91daf7a469c1f9eaf1fb80d4ecc49
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91825283"
---
# <a name="monitor-a-sharepoint-site-with-application-insights"></a>Surveillance d’un site SharePoint avec Application Insights

Azure Application Insights surveille la disponibilité, les performances et l’utilisation de vos applications. Vous allez maintenant apprendre à le configurer pour un site SharePoint.

> [!NOTE]
> Pour des raisons de sécurité, vous ne pouvez pas ajouter directement le script décrit dans cet article à vos pages web dans le cadre de l’expérience utilisateur moderne de SharePoint. Vous pouvez également vous servir de [ SharePoint Framework (SPFx)](/sharepoint/dev/spfx/extensions/overview-extensions) pour créer une extension personnalisée pouvant être utilisée pour installer Application Insights sur vos sites SharePoint.

## <a name="create-an-application-insights-resource"></a>Création d’une ressource Application Insights dans Azure
Dans le [portail Azure](https://portal.azure.com), créez une ressource Application Insights. Choisissez le type d'application ASP.NET.

![Cliquez sur Propriétés, sélectionnez la clé et appuyez sur Ctrl + C](./media/sharepoint/001.png)

Dans la fenêtre qui s’ouvre, vous trouverez des données relatives à l’utilisation et aux performances de votre application. Vous devriez trouver une vignette sur l’écran d’accueil pour accéder à ces informations, la prochaine fois que vous vous connecterez à Azure. Sinon, cliquez sur Parcourir.

## <a name="add-the-script-to-your-web-pages"></a>Ajout du script dans vos pages web

```HTML
<!-- 
To collect user behavior analytics tools about your application, 
insert the following script into each page you want to track.
Place this code immediately before the closing </head> tag,
and before any other scripts. Your first data will appear 
automatically in just a few seconds.
-->
<script type="text/javascript">
var sdkInstance="appInsightsSDK";window[sdkInstance]="appInsights";var aiName=window[sdkInstance],aisdk=window[aiName]||function(n){var o={config:n,initialize:!0},t=document,e=window,i="script";setTimeout(function(){var e=t.createElement(i);e.src=n.url||"https://az416426.vo.msecnd.net/scripts/b/ai.2.min.js",t.getElementsByTagName(i)[0].parentNode.appendChild(e)});try{o.cookie=t.cookie}catch(e){}function a(n){o[n]=function(){var e=arguments;o.queue.push(function(){o[n].apply(o,e)})}}o.queue=[],o.version=2;for(var s=["Event","PageView","Exception","Trace","DependencyData","Metric","PageViewPerformance"];s.length;)a("track"+s.pop());var r="Track",c=r+"Page";a("start"+c),a("stop"+c);var u=r+"Event";if(a("start"+u),a("stop"+u),a("addTelemetryInitializer"),a("setAuthenticatedUserContext"),a("clearAuthenticatedUserContext"),a("flush"),o.SeverityLevel={Verbose:0,Information:1,Warning:2,Error:3,Critical:4},!(!0===n.disableExceptionTracking||n.extensionConfig&&n.extensionConfig.ApplicationInsightsAnalytics&&!0===n.extensionConfig.ApplicationInsightsAnalytics.disableExceptionTracking)){a("_"+(s="onerror"));var p=e[s];e[s]=function(e,n,t,i,a){var r=p&&p(e,n,t,i,a);return!0!==r&&o["_"+s]({message:e,url:n,lineNumber:t,columnNumber:i,error:a}),r},n.autoExceptionInstrumented=!0}return o}(
{
  instrumentationKey:"INSTRUMENTATION_KEY"
}
);(window[aiName]=aisdk).queue&&0===aisdk.queue.length&&aisdk.trackPageView({});
</script>
```

Insérez-le juste avant la balise &lt;/head&gt; de chaque page que vous voulez suivre. Si votre site Web possède une page maître, vous pouvez y placer le script. Par exemple, dans un projet ASP.NET MVC, vous placeriez le script dans View\Shared\_Layout.cshtml

Le script contient la clé d'instrumentation qui dirige la télémétrie vers votre ressource Application Insights.

### <a name="add-the-code-to-your-site-pages"></a>Ajouter le code aux pages de votre site
#### <a name="on-the-master-page"></a>Sur la page maître
Si vous pouvez modifier la page maître de votre site, vous pourrez surveiller chaque page du site.

Vérifiez la page maître et modifiez-la à l'aide de SharePoint Designer ou de n’importe quel autre éditeur.

![Capture d'écran montrant comment modifier la page maître à l'aide de SharePoint Designer ou d'un autre éditeur.](./media/sharepoint/03-master.png)

Ajoutez le code juste avant la </head> balise. 

![Capture d'écran montrant où ajouter le code sur la page de votre site.](./media/sharepoint/04-code.png)

#### <a name="or-on-individual-pages"></a>Ou sur des pages individuelles
Pour surveiller un ensemble limité de pages, ajoutez le script à chaque page, séparément. 

Insérez un composant Web et incorporez l'extrait de code qu'il contient.

![Capture d'écran montrant comment ajouter le script pour surveiller un ensemble limité de pages.](./media/sharepoint/05-page.png)

## <a name="view-data-about-your-app"></a>Affichage des données relatives à votre application
Redéployez votre application.

Revenez au panneau de votre application dans le [portail Azure](https://portal.azure.com).

Les premiers événements s’affichent dans Search. 

![Capture d'écran montrant les nouvelles données que vous pouvez visualiser dans l'application.](./media/sharepoint/09-search.png)

Après quelques secondes, cliquez sur Actualiser pour obtenir des données supplémentaires.

## <a name="capturing-user-id"></a>Capture des ID d’utilisateur
L’extrait de code d’une page Web standard ne capture pas l’ID d’utilisateur à partir de SharePoint, mais vous pouvez y parvenir en effectuant une simple modification.

1. Copiez la clé d’instrumentation de votre application de la liste déroulante Bases dans Application Insights. 

    ![Capture d'écran montrant comment copier l'instrumentation de l'application à partir de la liste déroulante Essentials d'Application Insights.](./media/sharepoint/02-props.png)

1. Insérez la clé d’instrumentation à l’emplacement « XXXX » dans l’extrait de code ci-dessous. 
2. Incorporez dans votre application SharePoint au lieu de l’extrait de code que vous obtenez à partir du portail.

```


<SharePoint:ScriptLink ID="ScriptLink1" name="SP.js" runat="server" localizable="false" loadafterui="true" /> 
<SharePoint:ScriptLink ID="ScriptLink2" name="SP.UserProfiles.js" runat="server" localizable="false" loadafterui="true" /> 

<script type="text/javascript"> 
var personProperties; 

// Ensure that the SP.UserProfiles.js file is loaded before the custom code runs. 
SP.SOD.executeOrDelayUntilScriptLoaded(getUserProperties, 'SP.UserProfiles.js'); 

function getUserProperties() { 
    // Get the current client context and PeopleManager instance. 
    var clientContext = new SP.ClientContext.get_current(); 
    var peopleManager = new SP.UserProfiles.PeopleManager(clientContext); 

    // Get user properties for the target user. 
    // To get the PersonProperties object for the current user, use the 
    // getMyProperties method. 

    personProperties = peopleManager.getMyProperties(); 

    // Load the PersonProperties object and send the request. 
    clientContext.load(personProperties); 
    clientContext.executeQueryAsync(onRequestSuccess, onRequestFail); 
} 

// This function runs if the executeQueryAsync call succeeds. 
function onRequestSuccess() { 
var appInsights=window.appInsights||function(config){
function s(config){t[config]=function(){var i=arguments;t.queue.push(function(){t[config].apply(t,i)})}}var t={config:config},r=document,f=window,e="script",o=r.createElement(e),i,u;for(o.src=config.url||"//az416426.vo.msecnd.net/scripts/a/ai.0.js",r.getElementsByTagName(e)[0].parentNode.appendChild(o),t.cookie=r.cookie,t.queue=[],i=["Event","Exception","Metric","PageView","Trace"];i.length;)s("track"+i.pop());return config.disableExceptionTracking||(i="onerror",s("_"+i),u=f[i],f[i]=function(config,r,f,e,o){var s=u&&u(config,r,f,e,o);return s!==!0&&t["_"+i](config,r,f,e,o),s}),t
    }({
        instrumentationKey:"XXXX"
    });
    window.appInsights=appInsights;
    appInsights.trackPageView(document.title,window.location.href, {User: personProperties.get_displayName()});
} 

// This function runs if the executeQueryAsync call fails. 
function onRequestFail(sender, args) { 
} 
</script> 


```



## <a name="next-steps"></a>Étapes suivantes
* [Tests Web](./monitor-web-app-availability.md) pour surveiller la disponibilité de votre site.
* [Application Insights](./app-insights-overview.md) pour les autres types d'applications.

<!--Link references-->
