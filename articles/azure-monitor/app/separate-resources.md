---
title: 'Guide pratique pour concevoir votre déploiement Application Insights : une ressource vs plusieurs ressources'
description: Télémétrie directe de différentes ressources pour les tampons de développement, de test et de production.
ms.topic: conceptual
ms.date: 05/11/2020
ms.openlocfilehash: 49e9b8920af7333e0d95e23e6e5cf0828d448609
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95536351"
---
# <a name="how-many-application-insights-resources-should-i-deploy"></a>Combien de ressources Application Insights déployer

Lorsque vous développez la prochaine version d’une application web, vous ne souhaitez pas mélanger les télémétries [Application Insights](../../azure-monitor/app/app-insights-overview.md) de la nouvelle version et de la version déjà publiée. Pour éviter toute confusion, envoyez la télémétrie des différentes phases de développement à des ressources Application Insights séparées, avec des clés d’instrumentation (ikeys) distinctes. Pour faciliter la modification de la clé d’instrumentation à mesure qu’une version passe d’une étape à l’autre, il peut être utile de définir l’ikey dans le code plutôt que dans le fichier de configuration.

(Si votre système est un service cloud Azure, il existe [une autre méthode pour configurer des iKeys distinctes](../../azure-monitor/app/cloudservices.md).)

## <a name="about-resources-and-instrumentation-keys"></a>À propos des ressources et des clés d’instrumentation

Lorsque vous définissez un suivi Application Insights pour votre application web, vous créez une *ressource* Application Insights dans Microsoft Azure. Vous ouvrez cette ressource dans le portail Azure pour afficher et analyser la télémétrie recueillie à partir de votre application. Chaque ressource est identifiée par une *clé d’instrumentation* (iKey). Lorsque vous installez le package Application Insights pour surveiller votre application, vous le configurez avec la clé d’instrumentation afin qu’il sache où envoyer la télémétrie.

Chaque ressource Application Insights est accompagnée de métriques prêtes à l’emploi. Si des composants complètement distincts signalent à la même ressource Application Insights, ces métriques peuvent ne pas être pertinentes pour le tableau de bord ou pour les alertes.

### <a name="when-to-use-a-single-application-insights-resource"></a>Quand utiliser une seule ressource Application Insights

-   Pour les composants d’application qui sont déployés ensemble. Habituellement développés par une équipe unique, gérés par le même ensemble d’utilisateurs DevOps/ITOps.
-   S’il semble plus logique d’agréger des indicateurs de performance clés (KPI), tels que les durées de réponse, les taux d’échec dans le tableau de bord, et ainsi de suite, pour tous les composants par défaut (vous pouvez choisir de segmenter par nom de rôle dans l’expérience Metrics Explorer).
-   Si vous n’avez pas besoin de gérer le contrôle d’accès en fonction du rôle Azure (Azure RBAC) différemment entre les composants d’application.
-   Si vous n’avez pas besoin de critères d’alerte de métriques différents entre les composants.
-   Si vous n’avez pas besoin de gérer les exportations continues différemment entre les composants.
-   Si vous n’avez pas besoin de gérer la facturation/les quotas différemment entre les composants.
-   Si cela ne pose pas de problème qu’une clé API ait le même accès aux données à partir de tous les composants. Et que 10 clés API suffisent pour répondre aux besoins de tous.
-   Si cela ne pose pas de problème d’avoir les mêmes paramètres d’intégration d’élément de travail et de détection intelligente parmi tous les rôles.

### <a name="other-things-to-keep-in-mind"></a>Autres points à prendre en compte

-   Vous devrez peut-être ajouter du code personnalisé pour vous assurer que des valeurs significatives sont définies dans l’attribut [Cloud_RoleName](./app-map.md?tabs=net#set-or-override-cloud-role-name). Si aucune valeur significative n’est définie pour cet attribut, *AUCUNE* expérience du portail ne fonctionnera.
- Pour les applications Service Fabric et les services cloud classiques, le SDK lit automatiquement à partir de l’environnement de rôle Azure et les définit. Pour tous les autres types d’applications, vous devrez probablement définir cela de manière explicite.
-   L’expérience de métriques en temps réel ne prend pas en charge le fractionnement par nom de rôle.

## <a name="dynamic-instrumentation-key"></a><a name="dynamic-ikey"></a> Clé d'instrumentation dynamique

Pour faciliter la modification de l’ikey à mesure que le code se déplace entre les phases de production, référencez la clé dynamiquement dans le code au lieu d’utiliser une valeur codée en dur/statique.

Définissez la clé dans une méthode d'initialisation, par exemple global.aspx.cs dans un service ASP.NET :

```csharp
protected void Application_Start()
{
  Microsoft.ApplicationInsights.Extensibility.
    TelemetryConfiguration.Active.InstrumentationKey = 
      // - for example -
      WebConfigurationManager.AppSettings["ikey"];
  ...
```

Dans cet exemple, les ikeys des différentes ressources sont placées dans différentes versions du fichier de configuration web. Le remplacement du fichier de configuration web, que vous pouvez effectuer dans le cadre du script de lancement, remplacera la ressource cible.

### <a name="web-pages"></a>Pages web
L’iKey est également utilisée dans les pages web de votre application, dans le [script que vous avez obtenu à partir du volet de démarrage rapide](../../azure-monitor/app/javascript.md). Au lieu de la coder littéralement dans le script, vous devez la générer à partir de l'état du serveur. Par exemple, dans une application ASP.NET :

```javascript
<script type="text/javascript">
// Standard Application Insights web page script:
var appInsights = window.appInsights || function(config){ ...
// Modify this part:
}({instrumentationKey:  
  // Generate from server property:
  "@Microsoft.ApplicationInsights.Extensibility.
     TelemetryConfiguration.Active.InstrumentationKey"
  }
 )
//...
```

## <a name="create-additional-application-insights-resources"></a>Créer des ressource Application Insights supplémentaires

Pour créer une ressource Applications Insights, suivez le [Guide de création de ressources](./create-new-resource.md).

### <a name="getting-the-instrumentation-key"></a>Récupération de la clé d’instrumentation
La clé d'instrumentation identifie la ressource que vous avez créée.

Vous avez besoin des clés d’instrumentation de toutes les ressources auxquelles votre application envoie des données.

## <a name="filter-on-build-number"></a>Filtrer sur le numéro de build
Quand vous publiez une nouvelle version de votre application, vous voulez pouvoir distinguer la télémétrie des différentes builds.

Vous pouvez définir la propriété Version de l’application pour filtrer les résultats de [recherche](../../azure-monitor/app/diagnostic-search.md) et de [Metrics Explorer](../../azure-monitor/platform/metrics-charts.md).

Il existe plusieurs méthodes de définition de la propriété Version de l’application.

* Définissez directement :

    `telemetryClient.Context.Component.Version = typeof(MyProject.MyClass).Assembly.GetName().Version;`
* Encapsulez cette ligne dans un [initialiseur de télémétrie](../../azure-monitor/app/api-custom-events-metrics.md#defaults) pour vous assurer que toutes les instances de TelemetryClient sont définies de manière cohérente.
* [ASP.NET] Définissez la version dans `BuildInfo.config`. Le module web sélectionnera la version dans le nœud BuildLabel. Incluez ce fichier dans votre projet et n’oubliez pas de définir la propriété Toujours copier dans l’Explorateur de solutions.

    ```XML
    <?xml version="1.0" encoding="utf-8"?>
    <DeploymentEvent xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns="http://schemas.microsoft.com/VisualStudio/DeploymentEvent/2013/06">
      <ProjectName>AppVersionExpt</ProjectName>
      <Build type="MSBuild">
        <MSBuild>
          <BuildLabel kind="label">1.0.0.2</BuildLabel>
        </MSBuild>
      </Build>
    </DeploymentEvent>

    ```
* [ASP.NET] Générez automatiquement BuildInfo.config dans MSBuild. Pour ce faire, ajoutez quelques lignes à votre fichier `.csproj` :

    ```XML
    <PropertyGroup>
      <GenerateBuildInfoConfigFile>true</GenerateBuildInfoConfigFile>    <IncludeServerNameInBuildInfo>true</IncludeServerNameInBuildInfo>
    </PropertyGroup>
    ```

    Cela génère un fichier appelé *Votre_nom_de_projet*.BuildInfo.config. Le processus de publication le renomme en BuildInfo.config.

    L’étiquette de build contient un espace réservé (AutoGen_...) quand vous effectuez la génération avec Visual Studio. Mais quand vous utilisez MSBuild, l’espace réservé est remplacé par le numéro de version correct.

    Pour permettre à MSBuild de générer des numéros de version, définissez la version comme `1.0.*` dans AssemblyReference.cs

## <a name="version-and-release-tracking"></a>Suivi de la version
Pour vérifier la version de l’application, assurez-vous que `buildinfo.config` est généré par votre processus Microsoft Build Engine. Dans votre fichier `.csproj`, ajoutez :  

```XML
<PropertyGroup>
  <GenerateBuildInfoConfigFile>true</GenerateBuildInfoConfigFile>
  <IncludeServerNameInBuildInfo>true</IncludeServerNameInBuildInfo>
</PropertyGroup>
```

Quand il détient les informations de version, le module web Application Insights ajoute automatiquement la **version de l’application** en tant que propriété à chaque élément de télémétrie. Cela vous permet de filtrer par version lorsque vous effectuez des [recherches de diagnostic](../../azure-monitor/app/diagnostic-search.md) ou que vous [explorez les métriques](../../azure-monitor/platform/metrics-charts.md).

Toutefois, notez que le numéro de version de build est uniquement généré par Microsoft Build Engine, et non par la build de développement dans Visual Studio.

### <a name="release-annotations"></a>Annotations de version
Si vous utilisez Azure DevOps, vous pouvez [obtenir un marqueur d’annotation](../../azure-monitor/app/annotations.md) ajouté à vos graphiques lorsque vous publiez une nouvelle version. 

## <a name="next-steps"></a>Étapes suivantes

* [Ressources partagées pour plusieurs rôles](../../azure-monitor/app/app-map.md)
* [Créer un initialiseur de télémétrie pour distinguer des variantes A|B](../../azure-monitor/app/api-filtering-sampling.md#add-properties)