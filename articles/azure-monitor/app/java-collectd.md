---
title: Surveiller les performances des applications web Java sur Linux - Azure | Microsoft Docs
description: Surveillance étendue des performances des applications de votre site web Java avec le plug-in collectd pour Application Insights.
ms.topic: conceptual
ms.date: 03/14/2019
author: MS-jgol
ms.custom: devx-track-java
ms.author: jgol
ms.openlocfilehash: 5ec928a0dc3cbcde3c6dd50b1795a05b5e092bde
ms.sourcegitcommit: c4246c2b986c6f53b20b94d4e75ccc49ec768a9a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96601269"
---
# <a name="collectd-linux-performance-metrics-in-application-insights-deprecated"></a>collectd : métriques de performances Linux dans Application Insights [Déconseillé]

> [!IMPORTANT]
> L’**approche recommandée** pour surveiller des applications Java consiste à utiliser l’instrumentation automatique sans modifier le code. Suivez les instructions pour l’ **[agent Application Insights agent Java 3.0](./java-in-process-agent.md)** .

Pour explorer les métriques de performances d’un système Linux dans [Application Insights](./app-insights-overview.md), installez [collectd](https://collectd.org/) et son plug-in Application Insights. Cette solution open source rassemble diverses statistiques concernant le système et le réseau.

De manière générale, on utilise collectd après avoir [instrumenté le service web Java avec Application Insights][java]. Il vous donne davantage de données pour vous aider à améliorer les performances de votre application ou à diagnostiquer les problèmes. 

## <a name="get-your-instrumentation-key"></a>Récupérer votre clé d’instrumentation
Dans le [Portail Microsoft Azure](https://portal.azure.com), ouvrez la ressource [Application Insights](./app-insights-overview.md) dans laquelle vous souhaitez afficher les données. (Ou [créez une ressource](./create-new-resource.md).)

Effectuez une copie de la clé d’instrumentation, qui identifie la ressource.

![Parcourez tous les éléments, ouvrez votre ressource, puis, dans la liste déroulante « Essentials », sélectionnez et copiez la clé d’instrumentation.](./media/java-collectd/instrumentation-key-001.png)

## <a name="install-collectd-and-the-plug-in"></a>Installer le plug-in et collectd
Sur vos ordinateurs serveurs Linux :

1. Installez [collectd](https://collectd.org/) version 5.4.0 ou ultérieure.
2. Téléchargez le [plug-in d'écriture collectd Application Insights](https://github.com/microsoft/ApplicationInsights-Java/tree/master/core/src/main/java/com/microsoft/applicationinsights/internal). Notez le numéro de version.
3. Copiez le fichier JAR du plug-in dans `/usr/share/collectd/java`.
4. Modifiez `/etc/collectd/collectd.conf`:
   * Vérifiez que [le plug-in Java](https://collectd.org/wiki/index.php/Plugin:Java) est activé.
   * Mettez à jour l’élément JVMArg pour java.class.path afin d’inclure le fichier JAR suivant. Mettez à jour le numéro de version afin qu’il corresponde à celui que vous avez téléchargé :
   * `/usr/share/collectd/java/applicationinsights-collectd-1.0.5.jar`
   * Ajoutez cet extrait de code à l’aide de la clé d’instrumentation provenant de votre ressource :

```XML

     LoadPlugin "com.microsoft.applicationinsights.collectd.ApplicationInsightsWriter"
     <Plugin ApplicationInsightsWriter>
        InstrumentationKey "Your key"
     </Plugin>
```

Voici une partie d’un exemple de fichier de configuration :

```XML

    ...
    # collectd plugins
    LoadPlugin cpu
    LoadPlugin disk
    LoadPlugin load
    ...

    # Enable Java Plugin
    LoadPlugin "java"

    # Configure Java Plugin
    <Plugin "java">
      JVMArg "-verbose:jni"
      JVMArg "-Djava.class.path=/usr/share/collectd/java/applicationinsights-collectd-1.0.5.jar:/usr/share/collectd/java/collectd-api.jar"

      # Enabling Application Insights plugin
      LoadPlugin "com.microsoft.applicationinsights.collectd.ApplicationInsightsWriter"

      # Configuring Application Insights plugin
      <Plugin ApplicationInsightsWriter>
        InstrumentationKey "12345678-1234-1234-1234-123456781234"
      </Plugin>

      # Other plugin configurations ...
      ...
    </Plugin>
    ...
```

Configurez d’autres [plug-ins collectd](https://collectd.org/wiki/index.php/Table_of_Plugins)permettant de collecter des données de différentes sources.

Redémarrez collectd selon les instructions de son [manuel](https://collectd.org/wiki/index.php/First_steps).

## <a name="view-the-data-in-application-insights"></a>Visualiser les données dans Application Insights
Dans votre ressource Application Insights, ouvrez [Métriques et ajoutez des graphiques][metrics] en sélectionnant les métriques que vous souhaitez voir dans la catégorie Personnalisé.

Par défaut, les métriques sont agrégés sur toutes les machines hôtes à partir desquelles ils ont été collectés. Pour visualiser les métriques par hôte, dans le panneau des détails du graphique, activez le regroupement, puis choisissez un groupement de type CollectD-Host.

## <a name="to-exclude-upload-of-specific-statistics"></a>Pour exclure le chargement de statistiques spécifiques
Par défaut, le plug-in Application Insights envoie toutes les données collectées par tous les plug-ins de lecture collectd activés. 

Pour exclure des données provenant de plug-ins ou de sources de données spécifiques :

* Modifiez le fichier de configuration. 
* Dans `<Plugin ApplicationInsightsWriter>`, ajoutez les directives suivantes :

| Directive | Résultat |
| --- | --- |
| `Exclude disk` |Exclusion de toutes les données collectées par le plug-in `disk` . |
| `Exclude disk:read,write` |Exclusion des sources nommées `read` et `write` du plug-in `disk`. |

Séparez les directives par un saut de ligne.

## <a name="problems"></a>Des problèmes ?
*Je ne vois pas les données dans le portail*

* Ouvrez [Rechercher][diagnostic] pour voir si les événements bruts sont arrivés. Ils mettent parfois du temps à apparaître dans Metrics Explorer.
* Vous devrez peut-être [définir des exceptions de pare-feu pour les données sortantes](./ip-addresses.md)
* Activez le suivi dans le plug-in Application Insights. Ajoutez la ligne ci-après dans `<Plugin ApplicationInsightsWriter>`:
  * `SDKLogger true`
* Ouvrez un terminal et démarrez collectd en mode détaillé pour vérifier si des problèmes ont été signalés :
  * `sudo collectd -f`

## <a name="known-issue"></a>Problème connu

Le plug-in d’écriture Application Insights n’est pas compatible avec certains plug-ins de lecture. Certains plug-ins envoient parfois « NaN » alors que le plug-in Application Insights s’attend à recevoir un nombre à virgule flottante.

Symptôme : le journal collectd affiche des erreurs incluant « AI: ... SyntaxError: Jeton inattendu N ».

Solution de contournement : exclure les données collectées par les plug-ins d’écriture à l’origine du problème. 

<!--Link references-->

[api]: ./api-custom-events-metrics.md
[apiexceptions]: ./api-custom-events-metrics.md#track-exception
[availability]: ./monitor-web-app-availability.md
[diagnostic]: ./diagnostic-search.md
[eclipse]: app-insights-java-eclipse.md
[java]: java-get-started.md
[javalogs]: java-trace-logs.md
[metrics]: ../platform/metrics-charts.md

