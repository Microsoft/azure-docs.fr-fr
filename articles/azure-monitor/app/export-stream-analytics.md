---
title: Exporter à l’aide de Stream Analytics à partir d’Azure Application Insights | Microsoft Docs
description: Stream Analytics peut transformer, filtrer et acheminer en continu les données que vous exportez depuis Application Insights.
ms.topic: conceptual
ms.date: 01/08/2019
ms.openlocfilehash: e90daf2516b00f7a27ac47e34cfc4ace03fee6e2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90979174"
---
# <a name="use-stream-analytics-to-process-exported-data-from-application-insights"></a>Utiliser Stream Analytics pour traiter des données exportées depuis Application Insights
[Azure Stream Analytics](https://azure.microsoft.com/services/stream-analytics/) est l’outil idéal pour traiter des données [exportées depuis Application Insights](export-telemetry.md). Stream Analytics peut extraire des données de diverses sources. Il peut transformer et filtrer les données, puis les acheminer vers plusieurs récepteurs.

Dans cet exemple, nous allons créer une carte qui récupère des données d’Application Insights, renomme et traite certains champs, puis les dirige vers Power BI.

> [!WARNING]
> Il existe des [méthodes recommandées bien meilleures et plus simples pour afficher les données d’Application Insights dans Power BI](./export-power-bi.md). Le chemin d’accès illustré ici est un exemple pour illustrer comment traiter les données exportées.
> 
> 

![Diagramme de blocs pour l’exportation via SA vers PBI](./media/export-stream-analytics/020.png)

## <a name="create-storage-in-azure"></a>Création d’un stockage dans Azure
Comme l’exportation continue génère toujours des données vers un compte de stockage Azure, vous devez commencer par créer ce stockage.

1. Créez un compte de stockage « classique » dans votre abonnement sur le [portail Azure](https://portal.azure.com).
   
   ![Sur le portail Azure, choisissez Nouveau, Données, Stockage.](./media/export-stream-analytics/030.png)
2. Créez un conteneur.
   
    ![Dans le nouvel emplacement de stockage, sélectionnez Conteneurs, cliquez sur la mosaïque Conteneurs puis sur Ajouter.](./media/export-stream-analytics/040.png)
3. Copiez la clé d’accès au stockage.
   
    Vous en aurez bientôt besoin pour configurer l’entrée dans le service Stream analytics.
   
    ![Dans le stockage, ouvrez Paramètres, Clés et copiez la clé d’accès principale.](./media/export-stream-analytics/045.png)

## <a name="start-continuous-export-to-azure-storage"></a>Démarrer l’exportation continue vers le stockage Azure
[exportation continue](export-telemetry.md) déplace des données d'Application Insights vers le stockage Azure.

1. Sur le portail Azure, accédez à la ressource Application Insights que vous avez créée pour votre application.
   
    ![Sélectionnez Parcourir, Application Insights, puis votre application.](./media/export-stream-analytics/050.png)
2. Créez une exportation continue.
   
    ![Choisissez Paramètres, Exportation continue, Ajouter.](./media/export-stream-analytics/060.png)

    Sélectionnez le compte de stockage que vous avez préalablement créé.

    ![Définissez la destination de l’exportation.](./media/export-stream-analytics/070.png)

    Définissez les types d’événements que vous souhaitez afficher :

    ![Choisissez les types d’événements.](./media/export-stream-analytics/080.png)

1. Laissez les données s'accumuler. Installez-vous confortablement et laissez les utilisateurs utiliser votre application pendant un certain temps. Les données de télémétrie vont vous être transmises et vous permettre d’afficher des graphiques statistiques dans [Metrics explorer](../platform/metrics-charts.md) et des événements dans [Recherche de diagnostic](./diagnostic-search.md). 
   
    Les données seront également exportées vers votre stockage. 
2. Inspectez les données exportées. Dans Visual Studio, sélectionnez **Afficher / Cloud Explorer**, puis ouvrez Azure / Stockage. (Si vous n’avez pas cette option de menu, vous devez installer le SDK Azure : ouvrez la boîte de dialogue Nouveau projet, puis ouvrez Visual C# / Cloud / Obtenir Microsoft Azure SDK pour .NET.)
   
    ![Capture d'écran montrant comment définir les types d'événements que vous souhaitez voir.](./media/export-stream-analytics/04-data.png)
   
    Prenez note de la partie commune du nom du chemin d'accès, qui est dérivée du nom de l'application et de la clé d'instrumentation. 

Les événements sont écrits dans des fichiers blob au format JSON. Chaque fichier peut contenir un ou plusieurs événements. Donc, nous devons lire les données d’événement et filtrer les champs voulus. Nous pourrions effectuer toutes sortes d'opérations avec les données, mais notre objectif aujourd'hui est d'utiliser Stream Analytics pour transmettre les données vers Power BI.

## <a name="create-an-azure-stream-analytics-instance"></a>Création d’une instance Azure Stream Analytics
À partir du [portail Azure](https://portal.azure.com/), sélectionnez le service Azure Stream Analytics et créez une nouvelle tâche Stream Analytics :

![Capture d'écran montrant la page principale de création d'une tâche Stream Analytics sur le portail Azure.](./media/export-stream-analytics/SA001.png)

![Capture d'écran montrant les détails nécessaires à la création d'une tâche Stream Analytics.](./media/export-stream-analytics/SA002.png)

Lors de la création de la tâche, sélectionnez **Accéder à la ressource**.

![Capture d'écran montrant le message reçu au terme du déploiement de la nouvelle tâche Stream Analytics.](./media/export-stream-analytics/SA003.png)

### <a name="add-a-new-input"></a>Ajouter une nouvelle entrée

![Capture d'écran montrant comment ajouter des entrées à la tâche Stream Analytics.](./media/export-stream-analytics/SA004.png)

Définissez-le pour qu’il tienne compte des données de votre objet blob d’exportation continue :

![Capture d'écran montrant la configuration de la tâche Stream Analytics pour tenir compte des données d'un objet blob d'exportation continue.](./media/export-stream-analytics/SA0005.png)

Vous devez maintenant disposer de la clé d’accès principale issue de votre compte de stockage, que vous avez notée précédemment. Définissez-la comme clé de compte de stockage.

### <a name="set-path-prefix-pattern"></a>Définition de la séquence d’octets préfixe du chemin d’accès

**Veillez à définir le format de date sur AAAA-MM-JJ (avec des tirets).**

La séquence d'octets préfixe du chemin d'accès spécifie où Stream Analytics recherche les fichiers d'entrée dans le stockage. Vous devez la configurer pour correspondre au mode de stockage des données de l'exportation continue. Définissez-la comme suit :

`webapplication27_12345678123412341234123456789abcdef0/PageViews/{date}/{time}`

Dans cet exemple :

* `webapplication27` est le nom de la ressource Application Insights, **tout en minuscules**.
* `1234...` est la clé d'instrumentation de la ressource Application Insights, **sans les tirets**. 
* `PageViews` est le type de données que vous souhaitez analyser. Les types disponibles varient selon le filtre que vous définissez dans l’exportation continue. Examinez les données exportées pour voir les autres types disponibles et consultez le [modèle d’exportation de données](export-data-model.md).
* `/{date}/{time}` est une séquence écrite de manière littérale.

> [!NOTE]
> Vérifiez le stockage pour être sûr d'avoir le chemin d'accès approprié.
> 

## <a name="add-new-output"></a>Ajout de la nouvelle sortie
Sélectionnez maintenant votre tâche > **Sorties** > **Ajouter**.

![Capture d'écran montrant comment sélectionner votre tâche Stream Analytics pour ajouter une nouvelle sortie.](./media/export-stream-analytics/SA006.png)


![Sélectionnez le nouveau canal, cliquez sur Sorties, Ajouter, Power BI](./media/export-stream-analytics/SA010.png)

Indiquez votre **compte professionnel ou scolaire** pour autoriser Stream Analytics à accéder à votre ressource Power BI. Inventez ensuite un nom pour la sortie et le jeu de données et la table Power BI cibles.

## <a name="set-the-query"></a>Définir la requête
La requête régit la traduction de l’entrée vers la sortie.

Utilisez la fonction de Test pour vérifier que vous obtenez le résultat approprié. Indiquez les exemples de données que vous avez copiés de la page d'entrées. 

### <a name="query-to-display-counts-of-events"></a>Requête d'affichage du nombre d'événements
Collez cette requête :

```SQL
SELECT
  flat.ArrayValue.name,
  count(*)
INTO
  [pbi-output]
FROM
  [export-input] A
OUTER APPLY GetElements(A.[event]) as flat
GROUP BY TumblingWindow(minute, 1), flat.ArrayValue.name
```

* export-input est l’alias que nous avons donné au flux d’entrée
* pbi-output est l’alias de sortie que nous avons défini
* Nous utilisons [OUTER APPLY GetElements](/stream-analytics-query/apply-azure-stream-analytics), car le nom de l'événement se trouve dans un tableau JSON imbriqué. Ensuite, l’instruction Select récupère le nom de l’événement, ainsi que le nombre d’instances portant ce nom dans la période donnée. La clause [Regrouper par](/stream-analytics-query/group-by-azure-stream-analytics) regroupe les éléments en intervalles d'une minute.

### <a name="query-to-display-metric-values"></a>Requête d'affichage des valeurs de mesure

```SQL
SELECT
  A.context.data.eventtime,
  avg(CASE WHEN flat.arrayvalue.myMetric.value IS NULL THEN 0 ELSE  flat.arrayvalue.myMetric.value END) as myValue
INTO
  [pbi-output]
FROM
  [export-input] A
OUTER APPLY GetElements(A.context.custom.metrics) as flat
GROUP BY TumblingWindow(minute, 1), A.context.data.eventtime
```

* Cette requête explore la télémétrie des mesures pour obtenir l'heure de l'événement et la valeur de mesure. Les valeurs de mesure se trouvent dans un tableau, donc nous utilisons le modèle OUTER APPLY GetElements pour extraire les lignes. Dans ce cas, « myMetric » est le nom de la mesure. 

### <a name="query-to-include-values-of-dimension-properties"></a>Requête d’inclusion de valeurs de propriétés de dimension

```SQL
WITH flat AS (
SELECT
  MySource.context.data.eventTime as eventTime,
  InstanceId = MyDimension.ArrayValue.InstanceId.value,
  BusinessUnitId = MyDimension.ArrayValue.BusinessUnitId.value
FROM MySource
OUTER APPLY GetArrayElements(MySource.context.custom.dimensions) MyDimension
)
SELECT
  eventTime,
  InstanceId,
  BusinessUnitId
INTO AIOutput
FROM flat
```

* Cette requête inclut les valeurs des propriétés de dimension, indépendamment d’une dimension particulière se trouvant à un index fixe dans le tableau de dimension.

## <a name="run-the-job"></a>Exécuter le travail
Vous pouvez sélectionner une date dans le passé à partir de laquelle démarrer la tâche. 

![Sélectionnez la tâche et cliquez sur Interroger. Collez l’exemple ci-dessous.](./media/export-stream-analytics/SA008.png)

Attendez que le travail s’exécute.

## <a name="see-results-in-power-bi"></a>Afficher les résultats dans Power BI
> [!WARNING]
> Il existe des [méthodes recommandées bien meilleures et plus simples pour afficher les données d’Application Insights dans Power BI](./export-power-bi.md). Le chemin d’accès illustré ici est un exemple pour illustrer comment traiter les données exportées.
> 
> 

Ouvrez Power BI avec votre compte professionnel ou scolaire, puis sélectionnez le jeu de données et la table que vous avez définis comme sortie de la tâche Stream Analytics.

![Dans Power BI, sélectionnez votre dataset et vos champs.](./media/export-stream-analytics/200.png)

Vous pouvez maintenant utiliser ce jeu de données dans des rapports et des tableaux de bord dans [Power BI](https://powerbi.microsoft.com).

![Capture d'écran d'un exemple de rapport créé à partir d'un jeu de données dans Power BI.](./media/export-stream-analytics/210.png)

## <a name="no-data"></a>Pas de données ?
* Vérifiez que vous avez [défini le format de date](#set-path-prefix-pattern) correctement sur AAAA-MM-JJ (avec des tirets).

## <a name="video"></a>Vidéo
Noam Ben Zeev montre comment traiter des données exportées à l’aide de Stream Analytics.

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/Export-to-Power-BI-from-Application-Insights/player]
> 
> 

## <a name="next-steps"></a>Étapes suivantes
* [Exportation continue](export-telemetry.md)
* [Référence de modèle de données détaillé pour les valeurs et types de propriétés.](export-data-model.md)
* [Application Insights](./app-insights-overview.md)

