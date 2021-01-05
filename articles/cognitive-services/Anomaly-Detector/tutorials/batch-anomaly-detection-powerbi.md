---
title: 'Tutoriel : Visualiser les anomalies à l’aide de la détection par lot et de Power BI'
titleSuffix: Azure Cognitive Services
description: Découvrez comment utiliser l’API Détecteur d’anomalies et Power BI pour visualiser les anomalies dans vos données de série chronologique.
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.subservice: anomaly-detector
ms.topic: tutorial
ms.date: 09/10/2020
ms.author: mbullwin
ms.openlocfilehash: a17301e0807877662ae1bf34ade48e90a1d30c0c
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96006198"
---
# <a name="tutorial-visualize-anomalies-using-batch-detection-and-power-bi"></a>Tutoriel : Visualiser les anomalies à l’aide de la détection par lot et de Power BI

Ce tutoriel vous aide à rechercher par lot des anomalies au sein d’un jeu de données de série chronologique. À l’aide de Power BI Desktop, vous allez prendre un fichier Excel, préparer les données pour l’API Détecteur d’anomalies et visualiser les anomalies statistiques dans toutes ces données.

Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * Utiliser Power BI Desktop pour importer et transformer un jeu de données de série chronologique
> * Intégrer Power BI Desktop à l’API Détecteur d’anomalies pour la détection d’anomalie par lot
> * Visualisez les anomalies détectées dans vos données, notamment les valeurs attendues et observées, et les limites de détection d’anomalie.

## <a name="prerequisites"></a>Prérequis
* Un [abonnement Azure](https://azure.microsoft.com/free/cognitive-services)
* [Microsoft Power BI Desktop](https://powerbi.microsoft.com/get-started/), disponible gratuitement
* Un fichier Excel (.xlsx) contenant des points de données de série chronologique. Les exemples de données pour ce guide de démarrage rapide sont disponibles sur [GitHub](https://go.microsoft.com/fwlink/?linkid=2090962)
* Une fois que vous avez votre abonnement Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesAnomalyDetector"  title="Créer une ressource Détecteur d’anomalies"  target="_blank">créez une ressource Détecteur d’anomalies <span class="docon docon-navigate-external x-hidden-focus"></span></a> dans le portail Azure pour obtenir votre clé et votre point de terminaison.
    * Vous aurez besoin de la clé et du point de terminaison de la ressource que vous créez pour connecter votre application à l’API Détecteur d’anomalies. Vous effectuerez cette opération ultérieurement dans le guide de démarrage rapide.

[!INCLUDE [cognitive-services-anomaly-detector-data-requirements](../../../../includes/cognitive-services-anomaly-detector-data-requirements.md)]

## <a name="load-and-format-the-time-series-data"></a>Charger et mettre en forme les données de série chronologique

Pour commencer, ouvrez Power BI Desktop et chargez les données de série chronologique que vous avez téléchargées à partir des prérequis. Ce fichier Excel contient une série de paires valeur/horodatage en temps universel coordonné (UTC).  

> [!NOTE]
> Power BI peut utiliser les données issues d’une multitude de sources, telles que des fichiers .csv, des bases de données SQL, le Stockage Blob Azure et bien plus encore.  

Dans la fenêtre principale de Power BI Desktop, cliquez sur le ruban **Accueil**. Dans le groupe **Données externes** du ruban, ouvrez le menu déroulant **Obtenir des données** et cliquez sur **Excel**.

![Image du bouton « Obtenir des données » dans Power BI](../media/tutorials/power-bi-get-data-button.png)

Une fois que la boîte de dialogue s’est affichée, accédez au dossier où vous avez téléchargé l’exemple de fichier .xlsx et sélectionnez-le. Quand la boîte de dialogue **Navigateur** s’affiche, cliquez sur **Feuille1**, puis sur **Modifier**.

![Image de l’écran « Navigateur » de la source de données dans Power BI](../media/tutorials/navigator-dialog-box.png)

Power BI convertit les horodatages dans la première colonne en type de données `Date/Time`. Ces horodatages doivent être convertis en texte afin d’être envoyés à l’API Détecteur d’anomalies. Si l’éditeur Power Query ne s’ouvre pas automatiquement, cliquez sur **Modifier les requêtes** sous l’onglet Accueil.

Cliquez sur le ruban **Transformer** dans l’éditeur Power Query. Dans le groupe **N’importe quelle colonne**, ouvrez le menu déroulant **Type de données :** et sélectionnez **Texte**.

![Image de la liste déroulante Type de données](../media/tutorials/data-type-drop-down.png)

Quand vous recevez un avis concernant la modification du type de colonne, cliquez sur **Remplacer l’actuel**. Ensuite, cliquez sur **Fermer & appliquer** ou **Appliquer** dans le ruban **Accueil**.

## <a name="create-a-function-to-send-the-data-and-format-the-response"></a>Créer une fonction pour envoyer les données et mettre en forme la réponse

Pour mettre en forme et envoyer le fichier de données à l’API Détecteur d’anomalies, vous pouvez appeler une requête sur la table créée ci-dessus. Dans l’éditeur Power Query, dans le ruban **Accueil**, ouvrez le menu déroulant **Nouvelle source** et cliquez sur **Requête vide**.

Vérifiez que votre nouvelle requête est sélectionnée, puis cliquez sur **Éditeur avancé**.

![Image de l’écran « Éditeur avancé »](../media/tutorials/advanced-editor-screen.png)

Dans l’Éditeur avancé, utilisez l’extrait Power Query M suivant pour extraire les colonnes de la table et les envoyer à l’API. Par la suite, la requête créera une table à partir de la réponse JSON et la renverra. Remplacez la variable `apiKey` par votre clé API Détecteur d’anomalies valide, et `endpoint` par votre point de terminaison. Une fois que vous avez entré la requête dans l’Éditeur avancé, cliquez sur **Terminé**.

```M
(table as table) => let

    apikey      = "[Placeholder: Your Anomaly Detector resource access key]",
    endpoint    = "[Placeholder: Your Anomaly Detector resource endpoint]/anomalydetector/v1.0/timeseries/entire/detect",
    inputTable = Table.TransformColumnTypes(table,{{"Timestamp", type text},{"Value", type number}}),
    jsontext    = Text.FromBinary(Json.FromValue(inputTable)),
    jsonbody    = "{ ""Granularity"": ""daily"", ""Sensitivity"": 95, ""Series"": "& jsontext &" }",
    bytesbody   = Text.ToBinary(jsonbody),
    headers     = [#"Content-Type" = "application/json", #"Ocp-Apim-Subscription-Key" = apikey],
    bytesresp   = Web.Contents(endpoint, [Headers=headers, Content=bytesbody, ManualStatusHandling={400}]),
    jsonresp    = Json.Document(bytesresp),

    respTable = Table.FromColumns({

                     Table.Column(inputTable, "Timestamp")
                     ,Table.Column(inputTable, "Value")
                     , Record.Field(jsonresp, "IsAnomaly") as list
                     , Record.Field(jsonresp, "ExpectedValues") as list
                     , Record.Field(jsonresp, "UpperMargins")as list
                     , Record.Field(jsonresp, "LowerMargins") as list
                     , Record.Field(jsonresp, "IsPositiveAnomaly") as list
                     , Record.Field(jsonresp, "IsNegativeAnomaly") as list

                  }, {"Timestamp", "Value", "IsAnomaly", "ExpectedValues", "UpperMargin", "LowerMargin", "IsPositiveAnomaly", "IsNegativeAnomaly"}
               ),

    respTable1 = Table.AddColumn(respTable , "UpperMargins", (row) => row[ExpectedValues] + row[UpperMargin]),
    respTable2 = Table.AddColumn(respTable1 , "LowerMargins", (row) => row[ExpectedValues] -  row[LowerMargin]),
    respTable3 = Table.RemoveColumns(respTable2, "UpperMargin"),
    respTable4 = Table.RemoveColumns(respTable3, "LowerMargin"),

    results = Table.TransformColumnTypes(

                respTable4,
                {{"Timestamp", type datetime}, {"Value", type number}, {"IsAnomaly", type logical}, {"IsPositiveAnomaly", type logical}, {"IsNegativeAnomaly", type logical},
                 {"ExpectedValues", type number}, {"UpperMargins", type number}, {"LowerMargins", type number}}
              )

 in results
```

Appelez la requête sur votre feuille de données en sélectionnant `Sheet1` sous **Entrer un paramètre**, puis cliquez sur **Appeler**.

![Image de la fonction Appeler](../media/tutorials/invoke-function-screenshot.png)

## <a name="data-source-privacy-and-authentication"></a>Confidentialité de la source de données et authentification

> [!NOTE]
> Veillez à prendre en compte les stratégies de votre organisation en matière d’accès et de confidentialité des données. Pour plus d’informations, consultez [Niveaux de confidentialité Power BI Desktop](/power-bi/desktop-privacy-levels).

Vous pouvez recevoir un message d’avertissement quand vous essayez d’exécuter la requête, car elle utilise une source de données externe.

![Image montrant un avertissement créé par Power BI](../media/tutorials/blocked-function.png)

Pour résoudre ce problème, cliquez sur **Fichier** et **Options et paramètres**. Ensuite, cliquez sur **Options**. Sous **Fichier actuel**, sélectionnez **Confidentialité** et **Ignorer les niveaux de confidentialité et potentiellement améliorer les performances**.

De plus, vous pouvez recevoir un message vous invitant à spécifier la façon dont vous souhaitez vous connecter à l’API.

![Image montrant une requête pour spécifier les informations d’identification de l’accès](../media/tutorials/edit-credentials-message.png)

Pour résoudre ce problème, cliquez sur **Modifier les informations d’identification** dans le message. Quand la boîte de dialogue s’affiche, sélectionnez **Anonyme** pour vous connecter anonymement à l’API. Puis, cliquez sur **Se connecter**.

Ensuite, cliquez sur **Fermer & appliquer** dans le ruban **Accueil** pour appliquer les modifications.

## <a name="visualize-the-anomaly-detector-api-response"></a>Visualiser la réponse de l’API Détecteur d’anomalies

Dans l’écran principal de Power BI, commencez à utiliser les requêtes créées ci-dessus pour visualiser les données. Sélectionnez d’abord **Graphique en courbes** dans **Visualisations**. Ensuite, ajoutez l’horodatage de la fonction appelée à l’**axe** du graphique en courbes. Cliquez dessus avec le bouton droit, puis sélectionnez **Timestamp**.

![Clic droit sur la valeur d’horodatage](../media/tutorials/timestamp-right-click.png)

Ajoutez les champs suivants de la **Fonction appelée** au champ **Valeurs** du graphique. Utilisez la capture d’écran ci-dessous pour vous aider à créer votre graphique.

* Valeur
* UpperMargins
* LowerMargins
* ExpectedValues

![Image des paramètres du graphique](../media/tutorials/chart-settings.png)

Après avoir ajouté les champs, cliquez sur le graphique et redimensionnez-le pour afficher tous les points de données. Votre graphique ressemblera à la capture d’écran ci-dessous :

![Image de la visualisation du graphique](../media/tutorials/chart-visualization.png)

### <a name="display-anomaly-data-points"></a>Afficher les points de données d’anomalie

Sur le côté droit de la fenêtre Power BI, sous le volet **CHAMPS**, cliquez avec le bouton droit sur **Value** sous la **requête de fonction appelée**, puis cliquez sur **Nouvelle mesure rapide**.

![Image du nouvel écran de mesure rapide](../media/tutorials/new-quick-measure.png)

Dans l’écran qui s’affiche, sélectionnez **Valeur filtrée** comme calcul. Affectez la valeur `Sum of Value` à **Valeur de base**. Ensuite, faites glisser `IsAnomaly` du champ **Fonction appelée** vers **Filtre**. Sélectionnez `True` dans le menu déroulant **Filtre**.

![Deuxième image de l’écran Nouvelle mesure rapide](../media/tutorials/new-quick-measure-2.png)

Après avoir cliqué sur **OK**, vous aurez un champ `Value for True` en bas de la liste de vos champs. Cliquez dessus avec le bouton droit et renommez-le **Anomalie**. Ajoutez-le aux **Valeurs** du graphique. Ensuite, sélectionnez l’outil **Format** et affectez **Catégorie** comme type d’axe X.

![Image de la mise en forme de l’axe des X](../media/tutorials/format-x-axis.png)

Appliquez des couleurs à votre graphique en cliquant sur l’outil **Format** et sur **Couleurs des données**. Votre graphique doit ressembler à ceci :

![Image du graphique final](../media/tutorials/final-chart.png)

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
>[Streaming de la détection d’anomalie avec Azure Databricks](anomaly-detection-streaming-databricks.md)