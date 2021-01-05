---
title: Gérer l’utilisation et les coûts pour Azure Application Insights | Microsoft Docs
description: Gérer les volumes de données de télémétrie et surveiller les coûts dans Application Insights.
ms.topic: conceptual
ms.custom: devx-track-dotnet
author: DaleKoetke
ms.author: dalek
ms.date: 5/7/2020
ms.reviewer: mbullwin
ms.openlocfilehash: 69ac1e82c267dee521143c4ed5f6c2be4d32e2ea
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96531324"
---
# <a name="manage-usage-and-costs-for-application-insights"></a>Gérer l’utilisation et les coûts pour Application Insights

> [!NOTE]
> Cet article explique en quoi consistent vos coûts pour Application Insights et comment les contrôler.  Un article associé, [Surveiller l’utilisation et l’estimation des coûts](../platform/usage-estimated-costs.md), explique comment visualiser l’utilisation et les coûts estimés avec plusieurs fonctionnalités de surveillance Azure en fonction des différents modèles tarifaires.

Application Insights est conçu pour vous fournir tout ce dont vous avez besoin pour surveiller la disponibilité, les performances et l’utilisation de vos applications web, qu’elles soit hébergées dans Azure ou localement. Application Insights prend en charge les langages et les infrastructures courants, tels que .NET, Java et Node.js, et s’intègre aux processus et outils DevOps tels qu’Azure DevOps, Jira et PagerDuty. Il est important de comprendre ce qui détermine les coûts de surveillance de vos applications. Dans cet article, nous allons examiner les coûts de surveillance des applications et la façon dont vous pouvez les surveiller et les contrôler de manière proactive.

Si vous avez des questions sur le fonctionnement de la tarification d’Application Insights, vous pouvez publier une question dans notre [page Q&A Microsoft](/answers/topics/azure-monitor.html).

## <a name="pricing-model"></a>Modèle de tarification

Les tarifs d’[Azure Application Insights][start] suivent un modèle de **paiement à l’utilisation**, basé sur le volume de données ingérées. Ils peuvent donc varier si la période de conservation des données est plus longue. Chaque ressource d’Application Insights est facturée comme un service distinct et s’ajoute à votre facture d’abonnement Azure. Le volume de données est mesuré comme la taille du package de données JSON non compressé reçu par Application Insights de la part de votre application. Il n’existe aucun frais de volume de données pour l’utilisation du [Flux de métriques temps réel](./live-stream.md).

Les [tests web à plusieurs étapes](./availability-multistep.md) donnent lieu à des frais supplémentaires. Il s’agit de tests web qui exécutent une séquence d’actions. Aucun frais supplémentaire n’est facturé pour les *tests Ping* sur une seule page. Les données de télémétrie des tests Ping et des tests à plusieurs étapes sont facturées comme les autres données de télémétrie de votre application.

L’option Application Insights [Activation des alertes sur les dimensions de métrique personnalisées](./pre-aggregated-metrics-log-metrics.md#custom-metrics-dimensions-and-pre-aggregation) peut également engendrer des coûts supplémentaires, car elle est susceptible d’entraîner la création de métriques pré-agrégation supplémentaires. [Cliquez ici pour en savoir plus](./pre-aggregated-metrics-log-metrics.md) sur les métriques basées sur les journaux et pré-agrégées dans Application Insights et sur les [tarifs](https://azure.microsoft.com/pricing/details/monitor/) des métriques personnalisées Azure Monitor.

### <a name="workspace-based-application-insights"></a>Application Insights basé sur l’espace de travail

Pour les ressources Application Insights qui envoient leurs données à un espace de travail Log Analytics, appelé [ressources Application Insights basées sur l’espace de travail](create-workspace-resource.md), la facturation de l’ingestion et de la conservation des données est effectuée par l’espace de travail où se trouvent les données d’ Application Insights. Cela permet aux clients de tirer parti de toutes les options du [modèle de tarification](../platform/manage-cost-storage.md#pricing-model) Log Analytics qui incluent des réservations de capacité en plus du paiement à l’utilisation. Log Analytics dispose également de plus d’options pour la conservation des données, notamment la [rétention par type de données](../platform/manage-cost-storage.md#retention-by-data-type). Les types de données Application Insights dans l’espace de travail reçoivent 90 jours de rétention sans frais. L’utilisation des tests Web et l’activation des alertes sur les dimensions métriques personnalisées sont toujours signalées par le biais d’Application Insights. Découvrez comment suivre les coûts d’ingestion et de conservation des données dans Log Analytics à l’aide [des coûts d’utilisation et des coûts estimés](../platform/manage-cost-storage.md#understand-your-usage-and-estimate-costs), [Azure Cost Management + facturation](../platform/manage-cost-storage.md#viewing-log-analytics-usage-on-your-azure-bill) et [les requêtes log Analytics](#data-volume-for-workspace-based-application-insights-resources). 

## <a name="estimating-the-costs-to-manage-your-application"></a>Estimation des coûts de gestion de votre application

Si vous n’utilisez pas encore Application Insights, vous pouvez utiliser la [calculatrice de prix Azure Monitor](https://azure.microsoft.com/pricing/calculator/?service=monitor) pour estimer le coût d’utilisation d’Application Insights. Commencez par entrer « Azure Monitor » dans la zone de recherche, puis cliquez sur la vignette Azure Monitor obtenue. Faites défiler la page jusqu’à Azure Monitor et sélectionnez Application Insights dans la liste déroulante Type.  Ici, vous pouvez entrer le nombre de Go de données que vous souhaitez collecter par mois, à savoir la quantité de données qu’Application Insights collectera pour surveiller votre application.

Il existe ici deux approches possibles : utiliser la surveillance par défaut et l’échantillonnage adaptatif, disponible dans le kit SDK ASP.NET, puis estimer votre ingestion de données probable en fonction de ce que d’autres clients similaires ont connu.

### <a name="data-collection-when-using-sampling"></a>Collecte des données dans le cadre de l’échantillonnage

Avec l’[échantillonnage adaptatif](sampling.md#adaptive-sampling) du kit SDK ASP.NET, le volume de données est automatiquement ajusté pour demeurer à un taux de trafic maximal spécifié pour la surveillance Application Insights par défaut. Si l’application génère une faible quantité de données de télémétrie, comme lors du débogage ou en raison d’une faible utilisation, les éléments ne sont pas supprimés par le processeur d’échantillonnage tant que le volume reste inférieur au niveau configuré d’événements par seconde. Pour une application à volume élevé, avec le seuil par défaut de 5 événements par seconde, l’échantillonnage adaptatif limite le nombre d’événements quotidiens à 432 000. En utilisant une taille moyenne d’événement standard de 1 Ko, cela correspond à 13,4 Go de données de télémétrie pour un mois de 31 jours par nœud hébergeant votre application (puisque l’échantillonnage s’effectue localement sur chaque nœud). 

Pour les kits de développement logiciel (SDK) qui ne prennent pas en charge l’échantillonnage adaptatif, vous pouvez utiliser l’[échantillonnage d’ingestion](./sampling.md#ingestion-sampling) où l’échantillonnage a lieu quand les données sont reçues par Application Insights en fonction d’un pourcentage de données à conserver, ou l’[échantillonnage à débit fixe pour les sites web ASP.NET, ASP.NET Core et Java](sampling.md#fixed-rate-sampling) afin de réduire le trafic envoyé à partir de votre serveur web et de vos navigateurs web.

### <a name="learn-from-what-similar-customers-collect"></a>Apprendre de ce que les clients similaires collectent

Dans la calculatrice de prix Azure Monitor pour Application Insights, si vous activez la fonctionnalité « Estimer le volume de données en fonction de l’activité de l’application », vous pouvez fournir des entrées relatives à votre application (demandes par mois et affichages de pages par mois, dans l’éventualité où vous collecterez les données de télémétrie côté client), puis la calculatrice vous indiquera le volume médian et la quantité 90e centile de données collectées par des applications similaires. Ces applications couvrent la plage de configuration d’Application Insights (p. ex., certains ont l’[échantillonnage](./sampling.md) par défaut, d’autres n’ont aucun échantillonnage, etc.) et vous disposez encore du contrôle permettant de réduire le volume de données que vous ingérez par échantillonnage bien au-dessous du niveau médian. Toutefois, il s’agit d’un point de départ pour comprendre ce que les autres clients similaires voient.

## <a name="understand-your-usage-and-estimate-costs"></a>Comprendre votre utilisation et estimer les coûts

Application Insights vous permet de comprendre facilement à combien vont s’élever vos coûts en fonction des modèles d’utilisation récente. Pour commencer, dans le portail Azure, pour la ressource Application Insights, accédez à la page **Utilisation et estimation des coûts** :

![Choisir les tarifs](./media/pricing/pricing-001.png)

R. Consultez votre volume de données pour le mois. Cela inclut toutes les données reçues et conservées (après tous les [échantillonnages](./sampling.md)) provenant de votre serveur, de vos applications clientes et des tests de disponibilité.  
B. Les [tests web multiétapes](./availability-multistep.md) font l’objet d’une facturation distincte. (Cela n’inclut pas les tests de disponibilité simples, qui sont compris dans les frais de volume de données.)  
C. Affichez les tendances des volumes de données pour le mois antérieur.  
D. Utilisez [l’échantillonnage](./sampling.md) d’ingestion des données.
E. Définissez la limite quotidienne de volume de données.  

(Tous les prix indiqués dans les captures d’écran de cet article sont fournis uniquement à titre d’exemple. Pour connaître les prix actuels dans votre devise et pour votre région, consultez la page [Tarification Application Insights][pricing].)

Pour étudier votre utilisation d’Application Insights de façon plus approfondie, ouvrez la page **Métriques**, ajoutez la métrique nommée « Volume de point de données », puis sélectionnez l’option *Appliquer la division* pour répartir les données selon le « type de l’élément de télémétrie ».

Les frais liés à Application Insights sont ajoutés à votre facture Azure. Les informations relatives à votre facture Azure s’affichent dans la section Azure **Cost Management + Facturation** du portail Azure ou sur le [portail de facturation Azure](https://account.windowsazure.com/Subscriptions).  [Pour plus d’informations sur l’utilisation d’Application Insights, voir ci-dessous](#viewing-application-insights-usage-on-your-azure-bill). 

![Dans le menu de gauche, sélectionnez Facturation.](./media/pricing/02-billing.png)

### <a name="using-data-volume-metrics"></a>Utilisation des métriques du volume de données
<a id="understanding-ingested-data-volume"></a>

Pour en savoir plus sur vos volumes de données, sélectionnez **Métriques** pour votre ressource Application Insights, puis ajoutez un nouveau graphique. Pour les métriques du graphique, sous **Métriques basées sur les journaux**, sélectionnez **Volume de point de données**. Cliquez sur **Appliquer la division** et sélectionnez le groupe par **Type de `Telemetryitem`** .

![Utiliser les métriques pour examiner le volume des données](./media/pricing/10-billing.png)

### <a name="queries-to-understand-data-volume-details"></a>Requêtes pour comprendre les détails du volume de données

Il existe deux approches pour examiner les volumes de données dans Application Insights. La première utilise des informations agrégées dans la table `systemEvents` et la deuxième utilise la propriété `_BilledSize`, qui est disponible sur chaque événement ingéré. `systemEvents` n’aura pas d’informations sur la taille des données pour [L’application Insights basée sur l’espace de travail](#data-volume-for-workspace-based-application-insights-resources).

#### <a name="using-aggregated-data-volume-information"></a>Utilisation des informations agrégées de volume de données

Par exemple, vous pouvez utiliser le tableau `systemEvents` pour consulter le volume de données ingéré au cours des dernières 24 heures avec la requête :

```kusto
systemEvents
| where timestamp >= ago(24h)
| where type == "Billing"
| extend BillingTelemetryType = tostring(dimensions["BillingTelemetryType"])
| extend BillingTelemetrySizeInBytes = todouble(measurements["BillingTelemetrySize"])
| summarize sum(BillingTelemetrySizeInBytes)
```

Pour afficher un graphique du volume de données (en octets) par type de données au cours des 30 derniers jours, vous pouvez également utiliser :

```kusto
systemEvents
| where timestamp >= startofday(ago(30d))
| where type == "Billing"
| extend BillingTelemetryType = tostring(dimensions["BillingTelemetryType"])
| extend BillingTelemetrySizeInBytes = todouble(measurements["BillingTelemetrySize"])
| summarize sum(BillingTelemetrySizeInBytes) by BillingTelemetryType, bin(timestamp, 1d) | render barchart  
```

Notez que cette requête peut être utilisée dans une [alerte de journal Azure](../platform/alerts-unified-log.md) pour configurer des alertes sur des volumes de données.  

Pour en savoir plus sur les changements de vos données de télémétrie, nous obtenons le nombre d’événements par type à l’aide de la requête :

```kusto
systemEvents
| where timestamp >= startofday(ago(30d))
| where type == "Billing"
| extend BillingTelemetryType = tostring(dimensions["BillingTelemetryType"])
| summarize count() by BillingTelemetryType, bin(timestamp, 1d)
| render barchart  
```

#### <a name="using-data-size-per-event-information"></a>Utilisation des informations de taille de données par événement

Pour en savoir plus sur la source de vos volumes de données, vous pouvez utiliser la propriété `_BilledSize` qui est présente sur chaque événement ingéré.

Par exemple, pour voir les opérations qui génèrent le plus de volume de données dans les 30 derniers jours, nous pouvons additionner `_BilledSize` pour tous les événements de dépendance :

```kusto
dependencies
| where timestamp >= startofday(ago(30d))
| summarize sum(_BilledSize) by operation_Name
| render barchart  
```

#### <a name="data-volume-for-workspace-based-application-insights-resources"></a>Volume de données pour les ressources d’Application Insights basées sur l’espace de travail

Pour examiner les tendances du volume de données de toutes les ressources [d’Application Insights basées sur l’espace de travail](create-workspace-resource.md) dans un espace de travail pour la semaine passée, accédez à l’espace de travail Log Analytics et exécutez la requête :

```kusto
union (AppAvailabilityResults),
      (AppBrowserTimings),
      (AppDependencies),
      (AppExceptions),
      (AppEvents),
      (AppMetrics),
      (AppPageViews),
      (AppPerformanceCounters),
      (AppRequests),
      (AppSystemEvents),
      (AppTraces)
| where TimeGenerated >= startofday(ago(7d)) and TimeGenerated < startofday(now())
| summarize sum(_BilledSize) by _ResourceId, bin(TimeGenerated, 1d)
| render areachart
```

Pour interroger les tendances du volume de données par type pour une ressource Application Insights basée sur un espace de travail spécifique, dans l’espace de travail Log Analytics, utilisez :

```kusto
union (AppAvailabilityResults),
      (AppBrowserTimings),
      (AppDependencies),
      (AppExceptions),
      (AppEvents),
      (AppMetrics),
      (AppPageViews),
      (AppPerformanceCounters),
      (AppRequests),
      (AppSystemEvents),
      (AppTraces)
| where TimeGenerated >= startofday(ago(7d)) and TimeGenerated < startofday(now())
| where _ResourceId contains "<myAppInsightsResourceName>"
| summarize sum(_BilledSize) by Type, bin(TimeGenerated, 1d)
| render areachart
```

## <a name="viewing-application-insights-usage-on-your-azure-bill"></a>Visualisation de l’utilisation d’Application Insights sur votre facture Azure

Azure fournit de nombreuses fonctionnalités utiles dans le hub [Azure Cost Management + facturation](../../cost-management-billing/costs/quick-acm-cost-analysis.md?toc=/azure/billing/TOC.json). Par exemple, la fonctionnalité « Analyse des coûts » vous permet de voir vos dépenses en ressources Azure. L’ajout d’un filtre par type de ressource (dans microsoft.insights/components for Application Insights) vous permettra de suivre vos dépenses. Ensuite, pour « Regrouper par », sélectionnez « Catégorie du compteur » ou « Compteur ».  Pour les ressources d’Application Insights sur les plans de tarification actuels, la majorité de l’utilisation s’affichera comme Log Analytics pour la Catégorie du compteur puisqu’il existe un seul serveur principal des journaux d’activité pour tous les composants Azure Monitor. 

Vous pouvez mieux comprendre votre utilisation en [téléchargeant votre utilisation à partir du portail Azure](../../cost-management-billing/manage/download-azure-invoice-daily-usage-date.md#download-usage-in-azure-portal).
Dans la feuille de calcul téléchargée, vous pouvez voir l’utilisation par ressource Azure par jour. Dans cette feuille de calcul Excel, vous pouvez trouver l’utilisation de vos ressources Application Insights en filtrant d’abord la colonne « Catégorie du compteur » pour afficher « Application Insights » et « Log Analytics », puis en ajoutant le filtre « contient microsoft.insights/components » sur la colonne « ID d’instance ».  La plus grande part de l’utilisation d’Application Insights est signalée sur des compteurs avec la catégorie du compteur de Log Analytics, car il existe un seul backend de journaux pour tous les composants Azure Monitor.  Seules les ressources Application Insights sur les niveaux tarifaires hérités et les tests web à plusieurs étapes sont signalées avec une catégorie de compteur d’Application Insights.  L’utilisation est indiquée dans la colonne « Quantité consommée » et l’unité pour chaque entrée est affichée dans la colonne « Unité de mesure ».  Plus de détails sont disponibles pour vous aider à [comprendre votre facture Microsoft Azure](../../cost-management-billing/understand/review-individual-bill.md).

## <a name="managing-your-data-volume"></a>Gestion de votre volume de données

Voici quelques suggestions de techniques pour vous aider à gérer le volume de données que vous envoyez :

* **Échantillonnage** : vous pouvez effectuer un échantillonnage pour réduire la quantité de données de télémétrie envoyées à partir de votre serveur et de vos applications clientes, avec une distorsion minimale des métriques. L’échantillonnage est le principal outil dont vous disposez pour ajuster la quantité de données que vous envoyez. Découvrez plus en détail les [fonctionnalités d’échantillonnage](./sampling.md).

* **Limitez les appels AJAX** : vous pouvez [Limiter le nombre d’appels Ajax qui peuvent être signalés](./javascript.md#configuration) dans chaque affichage de page, ou désactiver les rapports Ajax.

* **Désactivez les modules inutiles** : [Modifiez ApplicationInsights.config](./configuration-with-applicationinsights-config.md) pour désactiver les modules de collecte dont vous n’avez pas besoin. Par exemple, vous pouvez décider que les compteurs de performances ou les données de dépendance ne sont pas essentiels.

* **Procédez à la pré-agrégation des métriques** : Si vous placez des appels de TrackMetric dans votre application, vous pouvez réduire le trafic en utilisant la surcharge qui accepte votre calcul de la moyenne et de l’écart type d’un lot de mesures. Une autre possibilité consiste à utiliser un [package de pré-agrégation](https://www.myget.org/gallery/applicationinsights-sdk-labs).
 
* **Limite quotidienne** : au moment où vous créez une ressource Application Insights dans le Portail Azure, la limite quotidienne est définie à 100 Go/jour. Quand vous créez une ressource Application Insights dans Visual Studio, la valeur par défaut est faible (seulement 32,3 Mo/jour). La valeur par défaut de la limite quotidienne est définie pour faciliter les tests. L’idée est que l’utilisateur augmente la limite quotidienne avant de déployer l’application en production. 

    La limite maximale est de 1 000 Go/jour, à moins que vous en demandiez une plus élevée pour les besoins d’une application à fort trafic.
    
    Les e-mails d’avertissement relatifs à la limite quotidienne sont envoyés aux comptes qui sont membres de ces rôles pour votre ressource Application Insights : « ServiceAdmin », « AccountAdmin », « CoAdmin », « Owner ».

    Soyez prudent quand vous définissez la limite quotidienne. Votre objectif doit être de *ne jamais atteindre la limite quotidienne*. Si vous atteignez la limite quotidienne, vous perdez des données pour le reste de la journée, ce qui vous empêche de surveiller votre application. Pour changer la limite quotidienne, utilisez l’option **Limite quotidienne de volume**. Vous pouvez accéder à cette option dans le volet **Utilisation et estimation des coûts** (comme décrit plus loin dans l’article).
    
    Nous avons supprimé la restriction sur certains types d’abonnements qui disposent d’un crédit qui n’a pas pu être utilisé pour Application Insights. Si l’abonnement impose une limite de dépense, la boîte de dialogue Limite quotidienne indique comment la supprimer et permet d’augmenter la limite quotidienne au-delà de 32,3 Mo/jour.
    
* **Limitation** : ce paramètre limite le débit de données à 32 000 événements par seconde, en moyenne sur 1 minute et par clé d’instrumentation. Le volume de données que votre application envoie est évalué à chaque minute. S’il dépasse le taux par seconde moyen de plus d’une minute, le serveur refuse des demandes. Le SDK met les données en mémoire tampon, puis tente de les renvoyer. Il répartit un volume excessif sur plusieurs minutes. Si votre application envoie régulièrement des données au-dessus du taux limite, certaines données sont supprimées. (Les SDK ASP.NET, Java et JavaScript essaient de renvoyer les données de cette manière ; les autres SDK suppriment parfois simplement les données limitées.) En cas de limitation, vous en êtes informé par un avertissement.

## <a name="manage-your-maximum-daily-data-volume"></a>Gérer votre volume de données maximal quotidien

Vous pouvez utiliser la limite quotidienne de volume pour limiter les données collectées. Toutefois, si la limite est atteinte, toutes les données de télémétrie envoyées à partir de votre application sont perdues pour le reste de la journée. Il est *déconseillé* que votre application atteigne la limite quotidienne. Vous ne pouvez pas suivre l’intégrité et les performances de votre application une fois qu’elle a atteint la limite quotidienne.

Au lieu d’utiliser la limite quotidienne de volume, utilisez [l’échantillonnage](./sampling.md) pour régler le volume de données sur le niveau souhaité. Ensuite, n’utilisez la limite quotidienne qu’en « dernier recours », au cas où votre application commencerait soudainement à envoyer des volumes de données de télémétrie beaucoup plus élevés.

### <a name="identify-what-daily-data-limit-to-define"></a>Identifier la limite de données quotidienne à définir

Consultez les informations sur l’utilisation et les coûts estimés d’Application Insights pour comprendre les tendances d’ingestion des données et la limite quotidienne de volume à définir. Effectuez cette opération avec précaution, car vous ne pouvez plus superviser vos ressources une fois la limite atteinte.

### <a name="set-the-daily-cap"></a>Définir la limite quotidienne

Pour changer la limite quotidienne, accédez à la section **Configurer** de votre ressource Application Insights, dans la page **Utilisation et estimation des coûts**, puis sélectionnez **Limite quotidienne**.

![Ajuster la limite du volume quotidien des données de télémétrie](./media/pricing/pricing-003.png)

Pour [modifier la limite quotidienne via Azure Resource Manager](./powershell.md), la propriété à modifier est `dailyQuota`.  Avec Azure Resource Manager, vous pouvez également définir `dailyQuotaResetTime` et `warningThreshold` pour le plafond quotidien.

### <a name="create-alerts-for-the-daily-cap"></a>Créer des alertes pour la limite quotidienne

La limite quotidienne d’Application Insights crée un événement dans le journal d’activité Azure quand les volumes de données ingérées atteignent le niveau d’avertissement ou le plafond quotidien.  Vous pouvez [créer une alerte en fonction de ces événements du journal d’activité](../platform/alerts-activity-log.md#create-with-the-azure-portal). Les noms de signal de ces événements sont :

* Seuil d’avertissement de la limite quotidienne du composant Application Insights atteint

* Limite quotidienne du composant Application Insights atteinte

## <a name="sampling"></a>échantillonnage
L’[échantillonnage](./sampling.md) est une méthode vous permettant de réduire la fréquence d’envoi des données de télémétrie à votre application, tout en conservant la capacité à trouver des événements connexes pendant les recherches de diagnostic. Vous conservez également le décompte des événements corrects.

L’échantillonnage est un moyen efficace de réduire les coûts et de respecter votre quota mensuel. L’algorithme d’échantillonnage conserve les éléments associés à la télémétrie ; ainsi, quand vous utilisez la recherche par exemple, vous pouvez trouver la demande liée à une exception spécifique. L’algorithme conserve également le décompte correct. Cela vous permet de voir les valeurs correctes des taux de demandes, des taux d’exception et des autres compteurs dans Metric Explorer.

Il existe plusieurs formes d’échantillonnage.

* [L’échantillonnage adaptatif](./sampling.md) est la méthode par défaut pour le SDK ASP.NET. L’échantillonnage adaptatif s’ajuste automatiquement au volume de données de télémétrie envoyées par votre application. Il fonctionne automatiquement dans le SDK de votre application web, afin de réduire le trafic de télémétrie sur le réseau. 
* *échantillonnage d’ingestion* est une méthode alternative qui fonctionne au niveau où les données de télémétrie issues de votre application entrent dans le service Application Insights. L’échantillonnage d’ingestion n’affecte pas le volume de télémétrie envoyé depuis votre application, mais il réduit le volume conservé par le service. Vous pouvez utiliser l’échantillonnage d’ingestion pour réduire le quota utilisé par les données de télémétrie provenant des navigateurs et d’autres SDK.

Pour définir l’échantillonnage d’ingestion, accédez au volet **Tarification** :

![Dans le volet Quotas et tarification, sélectionnez la vignette Exemples, puis sélectionnez une fraction d’échantillonnage.](./media/pricing/pricing-004.png)

> [!WARNING]
> Le volet **Échantillonnage des données** contrôle uniquement la valeur d’échantillonnage de l’ingestion. Il ne reflète pas le taux d’échantillonnage qui est appliqué par le SDK Application Insights dans votre application. Si les données de télémétrie entrantes ont déjà été échantillonnées dans le SDK, l’échantillonnage d’ingestion n’est pas appliqué.
>

Pour découvrir le taux d’échantillonnage réel, indépendamment de l’endroit où il a été appliqué, utilisez une [requête Analytics](../log-query/log-query-overview.md). La requête ressemble à ceci :

```kusto
requests | where timestamp > ago(1d)
| summarize 100/avg(itemCount) by bin(timestamp, 1h)
| render areachart
```

Pour chaque enregistrement conservé, `itemCount` indique le nombre d’enregistrements d’origine qu’il représente. Ce nombre est égal à 1 + le nombre d’enregistrements précédents ignorés.

## <a name="change-the-data-retention-period"></a>Changer la période de rétention des données

La rétention par défaut pour les ressources Application Insights est de 90 jours. Différentes périodes de rétention peuvent être sélectionnées pour chaque ressource Application Insights. L’ensemble complet de périodes de conservation disponibles est 30, 60, 90, 120, 180, 270, 365, 550 et 730 jours. [En savoir plus](https://azure.microsoft.com/pricing/details/monitor/) sur la tarification pour une conservation des données plus longue. 

Pour changer le délai de conservation, dans votre ressource Application Insights, accédez à la page **Utilisation et coûts estimés**, puis sélectionnez l’option **Conservation des données** :

![Capture d’écran montrant où modifier la période de rétention des données.](./media/pricing/pricing-005.png)

En cas de diminution de la conservation, une période de grâce de plusieurs jours s’applique avant la suppression des données les plus anciennes.

La conservation peut également être [définie par programmation à l’aide du paramètre `retentionInDays` dans PowerShell](powershell.md#set-the-data-retention). Si vous définissez la conservation des données sur 30 jours, vous pouvez déclencher une purge immédiate des anciennes données avec le paramètre `immediatePurgeDataOn30Days`, ce qui peut être utile pour les scénarios liés à la conformité. Cette fonctionnalité de vidage est exposée uniquement via Azure Resource Manager et doit être utilisée avec une extrême prudence. L’heure de réinitialisation quotidienne pour la limite du volume de données peut être configurée à l’aide d’Azure Resource Manager pour définir le paramètre `dailyQuotaResetTime`.

## <a name="data-transfer-charges-using-application-insights"></a>Frais de transfert de données avec Application Insights

L’envoi de données à Application Insights peut entraîner des frais de bande passante. Comme décrit dans la [page sur les tarifs de bande passante Azure](https://azure.microsoft.com/pricing/details/bandwidth/), le transfert de données entre des services Azure situés dans deux régions différentes est facturé en tant que transfert de données sortant au tarif normal. Le transfert de données entrantes est gratuit. Toutefois, ces frais sont très limités (et ne représentent qu’un faible pourcentage) par rapport aux coûts liés à l’ingestion de données de journal d’Application Insights. Ainsi, la maîtrise des coûts pour Log Analytics doit s’appuyer sur le volume de données ingérées. Vous trouverez des explications à ce sujet [ici](#managing-your-data-volume).

## <a name="limits-summary"></a>Synthèse des limites

[!INCLUDE [application-insights-limits](../../../includes/application-insights-limits.md)]

## <a name="disable-daily-cap-e-mails"></a>Désactiver les e-mails de limite quotidienne

Pour désactiver les e-mails de limite de volume quotidienne, accédez à la section **Configurer** de votre ressource Application Insights dans le volet **Utilisation et estimation des coûts**, puis sélectionnez **Limite quotidienne**. Il existe des paramètres pour envoyer un e-mail quand la limite est atteinte, ainsi que quand un niveau d’avertissement réglable a été atteint. Si vous souhaitez désactiver tous les e-mails liés au plafond de volume quotidien, décochez les deux cases.

## <a name="legacy-enterprise-per-node-pricing-tier"></a>Niveau tarifaire Entreprise existant (par nœud)

Pour les utilisateurs initiaux d’Azure Application Insights, il existe toujours deux niveaux tarifaires possibles : De base et Entreprise. Le niveau tarifaire De base est identique à celui décrit ci-dessus ; il s’agit du niveau par défaut. Il inclut toutes les fonctionnalités du niveau Entreprise, sans frais supplémentaires. La facturation du niveau De base repose essentiellement sur le volume de données ingérées.

> [!NOTE]
> Ces niveaux tarifaires existants ont été renommés. Le niveau tarifaire Entreprise et le niveau tarifaire De base sont désormais nommés **Par nœud** et **Par Go**, respectivement. Ces deux niveaux sont désignés sous leur nouveau nom ci-après et dans le portail Azure.  

Le niveau Par nœud (anciennement Entreprise) donne lieu à une facturation pour chaque nœud, chaque nœud recevant une allocation de données quotidienne. Dans le niveau tarifaire Par nœud, vous êtes facturé pour les données ingérées au-delà de l’allocation incluse. Si vous utilisez Operations Management Suite, vous devez choisir le niveau Par nœud.

Pour connaître les prix actuels dans votre devise et pour votre région, consultez la page [Tarification Application Insights](https://azure.microsoft.com/pricing/details/application-insights/).

> [!NOTE]
> En avril 2018, nous [introduisons](https://azure.microsoft.com/blog/introducing-a-new-way-to-purchase-azure-monitoring-services/) un nouveau modèle tarifaire pour la surveillance Azure. Ce modèle adopte un modèle de « paiement à l’utilisation » simple sur le portefeuille complet de services de surveillance. En savoir plus sur le [nouveau modèle tarifaire](../platform/usage-estimated-costs.md), sur [l’évaluation de l’impact du passage à ce modèle](../platform/usage-estimated-costs.md#understanding-your-azure-monitor-costs) en fonction de vos modèles d’utilisation et sur [l’adoption du nouveau modèle](../platform/usage-estimated-costs.md#azure-monitor-pricing-model)

### <a name="per-node-tier-and-operations-management-suite-subscription-entitlements"></a>Niveau tarifaire Par nœud et droits d’abonnement à Operations Management Suite

Comme [annoncé récemment](/archive/blogs/msoms/azure-application-insights-enterprise-as-part-of-operations-management-suite-subscription), les clients qui achètent Operations Management Suite E1 et E2 peuvent obtenir Application Insights Par nœud comme composant supplémentaire, sans frais supplémentaires. Plus précisément, chaque unité Operations Management Suite E1 et E2 donne droit à un nœud Application Insights du niveau Par nœud. Chaque nœud Application Insights inclut l’intégration de 200 Mo de données maximum par jour (en plus de l’intégration des données Log Analytics), avec une période de rétention des données de 90 jours, sans coût supplémentaire. Le niveau est décrit plus loin dans cet article.

Ce niveau ne s’appliquant qu’aux clients détenant un abonnement Operations Management Suite, ceux sans abonnement Operations Management Suite ne peuvent pas le sélectionner.

> [!NOTE]
> Pour pouvoir bénéficier de ce droit, vos ressources Application Insights doivent faire partie du niveau tarifaire Par nœud. Ce droit s’applique uniquement sous forme de nœuds. Les ressources Application Insights fournies dans le plan Par Go n’en bénéficient pas. Ce droit n’est pas visible dans l’estimation des coûts figurant dans le volet **Utilisation et estimation des coûts**. De plus, si vous déplacez un abonnement vers le nouveau modèle tarifaire de surveillance Azure d’avril 2018, le plan Par Go est le seul niveau disponible. Nous vous déconseillons de déplacer un abonnement vers le nouveau modèle tarifaire pour le monitoring Azure si vous avez un abonnement Operations Management Suite.

### <a name="how-the-per-node-tier-works"></a>Fonctionnement du niveau Par nœud

* Vous payez pour chaque nœud qui envoie des données de télémétrie relatives à une application dans le niveau Par nœud.
  * Un *nœud* correspond à une machine serveur virtuelle ou physique ou à une instance de rôle PaaS (Platform-as-a-Service) qui héberge votre application.
  * Les ordinateurs de développement, les navigateurs clients et les appareils mobiles ne sont pas considérés comme des nœuds.
  * Si votre application comporte plusieurs composants qui envoient des données de télémétrie, comme un service web et un Worker principal, ces composants sont comptabilisés séparément.
  * Les données des [Flux de métriques temps réel](./live-stream.md) ne sont pas comptabilisées dans la tarification. Dans un abonnement, vos frais sont calculés par nœud, et non par application. Si vous disposez de cinq nœuds envoyant des données de télémétrie pour 12 applications, les frais sont calculés pour cinq nœuds.
* Bien que les frais indiqués soient par mois, vous êtes facturé uniquement pour toutes les heures dans lesquelles un nœud envoie des données de télémétrie à partir d’une application. Le tarif horaire est le prix mensuel indiqué divisé par 744 (nombre d’heures dans un mois de 31 jours).
* Une allocation de volume de données de 200 Mo par jour est accordée pour chaque nœud détecté (avec une granularité par heure). Le volume de données alloué inutilisé n’est pas reporté de jour en jour.
  * Si vous choisissez le niveau tarifaire Par nœud, chaque abonnement reçoit un volume quotidien de données en fonction du nombre de nœuds qui envoient des données de télémétrie aux ressources Application Insights dans cet abonnement. Par conséquent, si vous disposez de cinq nœuds qui envoient des données toute la journée, une allocation groupée de 1 Go est appliquée à toutes les ressources Application Insights de cet abonnement. Le fait que certains nœuds envoient plus de données que d’autres a peu d’importance, car les données incluses sont partagées entre tous les nœuds. Si, un jour donné, les ressources Application Insights reçoivent plus de données que le volume quotidien alloué pour cet abonnement, les frais de données de dépassement par Go s’appliquent. 
  * Le volume de données quotidien alloué est calculé comme suit : nombre d’heures quotidiennes (UTC) pendant lesquelles chaque nœud envoie des données de télémétrie, divisé par 24 multiplié par 200 Mo. Par conséquent, si quatre nœuds envoient des données de télémétrie pendant 15 heures sur les 24 heures de la journée, les données incluses pour le jour en question sont calculées comme suit : ((4 &#215; 15) / 24) &#215; 200 Mo = 500 Mo. Au prix de 2,30 USD par Go pour le dépassement de données, les frais s'élèvent à 1,15 USD si les nœuds envoient 1 Go de données dans cette journée.
  * Le volume de données quotidien alloué du niveau Par nœud n’est pas partagé entre les applications pour lesquelles vous avez choisi le niveau Par Go. Le volume alloué inutilisé n’est pas reporté de jour en jour.

### <a name="examples-of-how-to-determine-distinct-node-count"></a>Exemples de détermination du nombre de nœuds distincts

| Scénario                               | Nombre total de nœuds quotidien |
|:---------------------------------------|:----------------:|
| 1 application utilisant 3 instances Azure App Service et 1 serveur virtuel | 4 |
| 3 applications s’exécutant sur 2 machines virtuelles ; ressources Application Insights pour ces applications figurant dans le même abonnement et dans le niveau Par nœud | 2 | 
| 4 applications dont les ressources Applications Insights figurent dans le même abonnement ; chaque application exécutant 2 instances pendant 16 heures creuses, ainsi que 4 instances pendant 8 heures de pointe | 13.33 |
| Services cloud avec 1 rôle de travail et 1 rôle web, chacune exécutant 2 instances | 4 | 
| Cluster Azure Service Fabric de 5 nœuds exécutant 50 microservices ; chaque microservice exécutant 3 instances | 5|

* Le comptage de nœuds précis varie selon le Kit de développement logiciel (SDK) Application Insights que votre application utilise. 
  * Dans les versions 2.2 et ultérieures de ce Kit de développement logiciel, le [Kit de développement logiciel (SDK) principal](https://www.nuget.org/packages/Microsoft.ApplicationInsights/) et le [Kit de développement logiciel (SDK) web](https://www.nuget.org/packages/Microsoft.ApplicationInsights.Web/) Application Insights signalent tous deux chaque hôte d’application en tant que nœud, par exemple le nom d’ordinateur pour le serveur physique et les hôtes de machine virtuelle ou le nom d’instance dans le cas de services cloud.  La seule exception concerne une application qui utilise uniquement [.NET Core](https://dotnet.github.io/) et le Kit de développement logiciel (SDK) principal Application Insights, auquel cas un seul nœud est signalé pour tous les hôtes, car le nom d’hôte n’est pas disponible.
  * Pour les versions antérieures du Kit de développement logiciel (SDK), le [Kit de développement logiciel (SDK) web](https://www.nuget.org/packages/Microsoft.ApplicationInsights.Web/) se comporte comme les nouvelles versions du Kit de développement logiciel (SDK), mais le [Kit de développement logiciel (SDK) principal](https://www.nuget.org/packages/Microsoft.ApplicationInsights/) ne signale qu’un seul nœud, quel que soit le nombre d’hôtes d’application.
  * Si votre application utilise le Kit de développement logiciel (SDK) pour définir l’instance de rôle (**roleInstance**) sur une valeur personnalisée, cette même valeur est utilisée par défaut pour déterminer le nombre de nœuds.
  * Si vous utilisez une nouvelle version du Kit de développement logiciel (SDK) avec une application qui s’exécute à partir d’ordinateurs clients ou d’appareils mobiles, le nombre de nœuds renvoyé peut se révéler important (en raison du grand nombre d’ordinateurs clients ou d’appareils mobiles).

## <a name="automation"></a>Automatisation

Vous pouvez utiliser le service Azure Resource Management pour écrire un script de définition du niveau tarifaire. [Découvrez comment](powershell.md#price).

## <a name="next-steps"></a>Étapes suivantes

* [Échantillonnage](./sampling.md)

[api]: app-insights-api-custom-events-metrics.md
[apiproperties]: app-insights-api-custom-events-metrics.md#properties
[start]: ./app-insights-overview.md
[pricing]: https://azure.microsoft.com/pricing/details/application-insights/
[pricing]: https://azure.microsoft.com/pricing/details/application-insights/

