---
title: Utiliser l’API Collecte de données pour créer un pipeline de données
description: L’API Collecte de données HTTP d’Azure Monitor permet d’ajouter des données POST JSON à l’espace de travail Log Analytics à partir de tout client capable d’appeler l’API REST. Cet article explique comment charger automatiquement des données stockées dans des fichiers.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 08/09/2018
ms.openlocfilehash: ac2b79046718fe45ad0dad0396b6f7653efbb779
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86505327"
---
# <a name="create-a-data-pipeline-with-the-data-collector-api"></a>Créer un pipeline de données avec l’API Collecte de données

L’[API Collecte de données d’Azure Monitor ](data-collector-api.md) vous permet d’importer des données de journal d’activité personnalisées dans un espace de travail Log Analytics d’Azure Monitor. Cette opération suppose simplement que les données soient au format JSON et qu’elles soient fractionnées en segments de 30 Mo maximum. Ce mécanisme parfaitement flexible peut être déclenché de nombreuses manières : avec des données directement envoyées à partir de votre application ou bien par des chargements ponctuels ad hoc. Cet article décrit les points de départ pour un scénario courant : télécharger des données stockées dans des fichiers de façon régulière et automatisée. Bien que le pipeline présenté ici ne soit pas le plus performant ou optimisé, il est destiné à servir de point de départ pour vous aider à créer votre propre pipeline de production.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="example-problem"></a>Exemple de problème
Dans le reste de cet article, nous allons examiner les données en mode Page dans Application Insights. Dans notre scénario fictif, nous voulons mettre en corrélation les informations géographiques collectées par défaut par le SDK Application Insights avec des données personnalisées contenant la population de tous les pays/toutes les régions du monde, afin de déterminer dans quelle région nous devons consacrer un maximum de notre budget marketing. 

Nous utilisons à cet effet une source de données publique, par exemple les [World Population Prospects de l’ONU](https://esa.un.org/unpd/wpp/). Les données utilisent le schéma simple suivant :

![Exemple de schéma simple](./media/create-pipeline-datacollector-api/example-simple-schema-01.png)

Dans notre exemple, nous partons du principe que nous allons charger un nouveau fichier avec les données de l’année précédente dès qu’elles sont disponibles.

## <a name="general-design"></a>Conception générale
Nous utilisons une logique de type ETL classique pour concevoir notre pipeline. L’architecture se présente comme suit :

![Architecture du pipeline de collecte de données](./media/create-pipeline-datacollector-api/data-pipeline-dataflow-architecture.png)

Cet article n’explique pas comment créer des données ou [les charger dans un compte de stockage Blob Azure](../../storage/blobs/storage-upload-process-images.md). Il décrit le flux déclenché dès qu’un nouveau fichier est chargé dans le blob. À partir de là :

1. Un processus détectera que des nouvelles données ont été chargées.  Notre exemple utilise une [Azure Logic App](../../logic-apps/logic-apps-overview.md), qui dispose d’un déclencheur pour détecter les nouvelles données en cours de chargement dans un blob.

2. Un processeur lit ces nouvelles données et les convertit au format JSON, format requis par Azure Monitor. Dans cet exemple, nous utilisons une [fonction Azure](../../azure-functions/functions-overview.md) pour exécuter notre code de traitement car il s'agit d'un moyen simple et économique. La fonction est déclenchée par la même application logique que nous avons utilisée pour détecter les nouvelles données.

3. Enfin, une fois que l'objet JSON est disponible, il est envoyé à Azure Monitor. La même application logique envoie les données à Azure Monitor en utilisant l'activité de collecte de données intégrée de Log Analytics.

Bien que la configuration détaillée du stockage blob, de Logic App ou d’Azure Function ne soit pas décrite dans le présent article, des instructions détaillées sont disponibles sur les pages produits spécifiques.

Dans le cadre de la surveillance de ce pipeline, nous utilisons Application Insights pour surveiller notre fonction Azure ([voir détails ici](../../azure-functions/functions-monitoring.md)) et Azure Monitor pour surveiller notre application logique ([voir détails ici](../../logic-apps/monitor-logic-apps-log-analytics.md)). 

## <a name="setting-up-the-pipeline"></a>Configuration du pipeline
Pour configurer le pipeline, assurez-vous que vous avez créé et configuré votre conteneur blob. Vérifiez également que l’espace de travail Log Analytics vers lequel vous souhaitez envoyer les données a bien été créé.

## <a name="ingesting-json-data"></a>Ingestion des données JSON
L’ingestion de données JSON s’effectue très simplement avec Logic Apps, et dans la mesure où aucune transformation n’est nécessaire, nous pouvons appliquer l’ensemble du pipeline dans une seule Logic App. Une fois que le conteneur blob et l’espace de travail Log Analytics ont été configurés, créez une Logic App et configurez-la comme suit :

![Exemple de workflow Logic Apps](./media/create-pipeline-datacollector-api/logic-apps-workflow-example-01.png)

Enregistrez votre Logic App et testez-la.

## <a name="ingesting-xml-csv-or-other-formats-of-data"></a>Ingestion de données aux formats XML, CSV ou autres
Logic Apps n’intègre pour le moment aucune fonctionnalité permettant de transformer facilement des données type XML, CSV ou autre au format JSON. Par conséquent, nous devons utiliser un autre moyen pour effectuer cette transformation. Pour cet article, nous utilisons les fonctionnalités de calcul sans serveur d’Azure Functions pour y parvenir facilement et à moindres coûts. 

Dans cet exemple, nous analysons un fichier CSV, mais n’importe quel autre type de fichier peut être traité de la même façon. Modifiez simplement la partie désérialisation de l’Azure Function afin de bien refléter la logique de votre type de données spécifique.

1.  À l’invite, créez une Azure Function à l’aide du runtime v1 et en fonction de la consommation.  Sélectionnez le modèle de **déclencheur HTTP** ciblé sur C# comme point de départ pour configurer vos liaisons comme il se doit. 
2.  Sous l’onglet **Afficher les fichiers** dans le volet droit, créez un fichier appelé **project.json** et collez le code suivant à partir des packages NuGet que nous utilisons :

    ![Exemple de projet Azure Functions](./media/create-pipeline-datacollector-api/functions-example-project-01.png)
    
    ``` JSON
    {
      "frameworks": {
        "net46":{
          "dependencies": {
            "CsvHelper": "7.1.1",
            "Newtonsoft.Json": "11.0.2"
          }  
        }  
       }  
     }  
    ```

3. Dans le volet droit, basculez sur **run.csx** et remplacez le code par défaut par le code suivant. 

    >[!NOTE]
    >Pour votre projet, vous devez remplacer le modèle d’enregistrement (la classe « PopulationRecord ») par votre propre schéma de données.
    >

    ```   
    using System.Net;
    using Newtonsoft.Json;
    using CsvHelper;
    
    class PopulationRecord
    {
        public String Location { get; set; }
        public int Time { get; set; }
        public long Population { get; set; }
    }

    public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, TraceWriter log)
    {
        string filePath = await req.Content.ReadAsStringAsync(); //get the CSV URI being passed from Logic App
        string response = "";

        //get a stream from blob
        WebClient wc = new WebClient();
        Stream s = wc.OpenRead(filePath);         

        //read the stream
        using (var sr = new StreamReader(s))
        {
            var csvReader = new CsvReader(sr);
    
            var records = csvReader.GetRecords<PopulationRecord>(); //deserialize the CSV stream as an IEnumerable
    
            response = JsonConvert.SerializeObject(records); //serialize the IEnumerable back into JSON
        }    

        return response == null
            ? req.CreateResponse(HttpStatusCode.BadRequest, "There was an issue getting data")
            : req.CreateResponse(HttpStatusCode.OK, response);
     }  
    ```

4. Enregistrez votre fonction.
5. Testez la fonction pour vérifier que le code fonctionne correctement. Basculez sur l’onglet **Test** dans le volet droit, en configurant le test comme suit. Placez un lien vers un blob avec des exemples de données dans la zone de texte **Corps de la demande**. Après avoir cliqué sur **Exécuter**, vous devez obtenir une sortie JSON dans la zone **Sortie** :

    ![Code de test Function Apps](./media/create-pipeline-datacollector-api/functions-test-01.png)

Nous devons à présent revenir à la Logic App que nous avons commencé à générer et la modifier pour inclure les données ingérées et converties au format JSON.  À l’aide du Concepteur de vue, définissez la configuration suivante et enregistrez votre Logic App :

![Exemple complet de workflow Logic Apps](./media/create-pipeline-datacollector-api/logic-apps-workflow-example-02.png)

## <a name="testing-the-pipeline"></a>Test du pipeline
Vous pouvez maintenant charger un nouveau fichier dans le blob configuré précédemment et le faire analyser par votre Logic App. Bientôt, vous devriez pouvoir déclencher une nouvelle application logique, appeler votre fonction Azure et réussir à envoyer vos données vers Azure Monitor. 

>[!NOTE]
>La première fois que vous envoyez un nouveau type de données, l'affichage des données dans Azure Monitor peut nécessiter une trentaine de minutes.


## <a name="correlating-with-other-data-in-log-analytics-and-application-insights"></a>Mise en corrélation avec d’autres données dans Log Analytics et Application Insights
Pour atteindre notre objectif, à savoir mettre en corrélation des données en mode Page d’Application Insights avec des données de population reçues à partir de notre source de données personnalisée, exécutez la requête suivante à partir de votre fenêtre Application Insights Analytics ou d’un espace de travail Log Analytics :

``` KQL
app("fabrikamprod").pageViews
| summarize numUsers = count() by client_CountryOrRegion
| join kind=leftouter (
   workspace("customdatademo").Population_CL
) on $left.client_CountryOrRegion == $right.Location_s
| project client_CountryOrRegion, numUsers, Population_d
```

La sortie doit indiquer les deux sources de données, qui sont désormais jointes.  

![Mise en corrélation de données disjointes dans un exemple de résultat de recherche](./media/create-pipeline-datacollector-api/correlating-disjoined-data-example-01.png)

## <a name="suggested-improvements-for-a-production-pipeline"></a>Suggestions d’améliorations pour un pipeline de production
Cet article a présenté un prototype opérationnel et la logique sous-jacente qui peut être appliquée à une véritable solution de production. Pour une telle solution de production, nous recommandons les améliorations suivantes :

* Ajoutez une logique de gestion des erreurs et de nouvelle tentative dans votre Logic App et votre Azure Function.
* Ajoutez une logique pour vous assurer que la limite de 30 Mo par simple appel de l’API d’ingestion Log Analytics n’est pas dépassée. Fractionnez si besoin les données en plus petits segments.
* Configurez une stratégie de nettoyage sur votre stockage blob. Une fois les données envoyées à l’espace de travail Log Analytics, à moins que vous ne souhaitiez conserver les données brutes à des fins d’archivage, rien ne vous oblige à les stocker. 
* Vérifiez que la supervision est activée dans l’ensemble du pipeline, en ajoutant des points de trace et des alertes si besoin.
* Tirez parti du contrôle de code source pour gérer le code pour votre fonction et votre Logic App.
* Assurez-vous que la stratégie de gestion des changements appropriée est bien respectée, de sorte que si le schéma est modifié, la fonction et les Logic Apps soient modifiées en conséquence.
* Si vous téléchargez plusieurs types de données différents, séparez-les dans des dossiers individuels au sein de votre conteneur blob, et créez une logique pour l’appliquer en fonction du type de données. 


## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur l’[API Collecte de données](data-collector-api.md) pour écrire des données dans l’espace de travail Log Analytics à partir d’un client API REST.
