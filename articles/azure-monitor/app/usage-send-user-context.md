---
title: ID de contexte utilisateur pour suivre l’activité – Azure Application Insights
description: Suivez les déplacements des utilisateurs dans votre service en affectant une chaîne d’ID unique et permanente à chacun d’eux dans Application Insights.
ms.topic: conceptual
author: NumberByColors
ms.author: daviste
ms.date: 01/03/2019
ms.reviewer: abgreg;mbullwin
ms.openlocfilehash: 46b7479df6d087915cfe81895a786a528da6b9bb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87327903"
---
# <a name="send-user-context-ids-to-enable-usage-experiences-in-azure-application-insights"></a>Envoyer des ID de contexte utilisateur pour activer les expériences d’utilisation dans Azure Application Insights

## <a name="tracking-users"></a>Suivi des utilisateurs

Application Insights vous permet de surveiller et suivre vos utilisateurs par le biais d’un ensemble d’outils d’utilisation du produit :

- [Utilisateurs, sessions, événements](./usage-segmentation.md)
- [Entonnoirs](./usage-funnels.md)
- Cohortes de [rétention](./usage-retention.md)
- [Classeurs](../platform/workbooks-overview.md)

Afin d’assurer le suivi des actions d’un utilisateur au fil du temps, Application Insights nécessite un ID pour chaque utilisateur ou chaque session. Incluez les ID suivants dans chaque vue personnalisée de page ou d’événement personnalisé.

- Utilisateurs, Entonnoirs, Rétention et Cohortes : incluez l’ID d’utilisateur.
- Sessions : incluez l’ID de session.

> [!NOTE]
> Il s’agit d’un article avancé qui décrit les étapes manuelles pour le suivi des activités des utilisateurs avec Application Insights. Avec de nombreuses applications web, **ces étapes peuvent ne pas être obligatoires**, car les SDK côté serveur par défaut conjointement avec le [SDK JavaScript côté client/navigateur ](./website-monitoring.md) sont souvent suffisants pour effectuer un suivi automatique des activités des utilisateurs. Si vous n’avez pas configuré la [supervision côté client](./website-monitoring.md) en plus du SDK côté serveur, effectuez d’abord cette opération et vérifiez si les outils d’analytique de comportement utilisateur fonctionnent comme prévu.

## <a name="choosing-user-ids"></a>Choix d’ID d’utilisateur

Les ID d’utilisateur doivent être conservés d’une session utilisateur à l’autre pour permettre de suivre le comportement des utilisateurs au fil du temps. Il existe différents moyens de conserver l’ID.

- Définition d’un utilisateur dont vous disposez déjà dans votre service.
- Si le service a accès à un navigateur, il peut transmettre au navigateur un cookie contenant un ID. L’ID est conservé tant que le cookie reste dans le navigateur de l’utilisateur.
- Si nécessaire, vous pouvez utiliser un nouvel ID à chaque session, mais les résultats sur les utilisateurs seront limités. Par exemple, vous ne pourrez pas voir l’évolution du comportement de l’utilisateur au fil du temps.

L’ID doit être un identificateur unique ou une autre chaîne assez complexe pour permettre l’identification spécifique de chaque utilisateur. Par exemple, il peut s’agir d’un nombre aléatoire long.

Si l’ID contient des informations personnelles sur l’utilisateur, cette valeur ne peut pas être envoyée à Application Insights en tant qu’ID d’utilisateur. Vous pouvez envoyer un ID en tant qu’[ID d’utilisateur authentifié](./api-custom-events-metrics.md#authenticated-users), mais cela ne satisfait pas les critères requis pour l’ID d’utilisateur dans les scénarios d’usage.

## <a name="aspnet-apps-setting-the-user-context-in-an-itelemetryinitializer"></a>Applications ASP.NET : définition du contexte utilisateur dans ITelemetryInitializer

Créez un initialiseur de télémétrie, comme décrit en détail [ici](./api-filtering-sampling.md#addmodify-properties-itelemetryinitializer). Transmettez l’ID de session via la télémétrie des requêtes et définissez Context.User.Id et Context.Session.Id.

Cet exemple configure l’ID d’utilisateur en tant qu’identificateur expirant à la fin de la session. Si possible, utilisez un ID d’utilisateur qui se conserve d’une session à l’autre.

### <a name="telemetry-initializer"></a>Initialiseur de télémétrie

```csharp
using System;
using System.Web;
using Microsoft.ApplicationInsights.Channel;
using Microsoft.ApplicationInsights.Extensibility;

namespace MvcWebRole.Telemetry
{
  /*
   * Custom TelemetryInitializer that sets the user ID.
   *
   */
  public class MyTelemetryInitializer : ITelemetryInitializer
  {
    public void Initialize(ITelemetry telemetry)
    {
        var ctx = HttpContext.Current;

        // If telemetry initializer is called as part of request execution and not from some async thread
        if (ctx != null)
        {
            var requestTelemetry = ctx.GetRequestTelemetry();
 
            // Set the user and session ids from requestTelemetry.Context.User.Id, which is populated in Application_PostAcquireRequestState in Global.asax.cs.
            if (requestTelemetry != null && !string.IsNullOrEmpty(requestTelemetry.Context.User.Id) &&
                (string.IsNullOrEmpty(telemetry.Context.User.Id) || string.IsNullOrEmpty(telemetry.Context.Session.Id)))
            {
                // Set the user id on the Application Insights telemetry item.
                telemetry.Context.User.Id = requestTelemetry.Context.User.Id;
 
                // Set the session id on the Application Insights telemetry item.
                telemetry.Context.Session.Id = requestTelemetry.Context.User.Id;
            }
        }
    }
  }
}
```

### <a name="globalasaxcs"></a>Global.asax.cs

```csharp
using System.Web;
using System.Web.Http;
using System.Web.Mvc;
using System.Web.Optimization;
using System.Web.Routing;

namespace MvcWebRole.Telemetry
{
    public class MvcApplication : HttpApplication
    {
        protected void Application_Start()
        {
            AreaRegistration.RegisterAllAreas();
            GlobalConfiguration.Configure(WebApiConfig.Register);
            FilterConfig.RegisterGlobalFilters(GlobalFilters.Filters);
            RouteConfig.RegisterRoutes(RouteTable.Routes);
            BundleConfig.RegisterBundles(BundleTable.Bundles);
        }
 
        protected void Application_PostAcquireRequestState()
        {
            var requestTelemetry = Context.GetRequestTelemetry();
 
            if (HttpContext.Current.Session != null && requestTelemetry != null && string.IsNullOrEmpty(requestTelemetry.Context.User.Id))
            {
                requestTelemetry.Context.User.Id = Session.SessionID;
            }
        }
    }
}
```

## <a name="next-steps"></a>Étapes suivantes

- Pour activer les expériences d’utilisation, commencez à envoyer des [événements personnalisés](./api-custom-events-metrics.md#trackevent) ou des [affichages de page](./api-custom-events-metrics.md#page-views).
- Si vous envoyez déjà des événements personnalisés ou des affichages de page, explorez les outils d’utilisation pour savoir comment les utilisateurs emploient votre service.
    - [Vue d’ensemble de l’utilisation](usage-overview.md)
    - [Utilisateurs, Sessions et Événements](usage-segmentation.md)
    - [Entonnoirs](usage-funnels.md)
    - [Rétention](usage-retention.md)
    - [Classeurs](../platform/workbooks-overview.md)

