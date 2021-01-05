---
title: Visualisations
description: Utiliser des notebooks Azure Synapse pour visualiser vos données
author: midesa
ms.author: midesa
ms.reviewer: euang
services: synapse-analytics
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: spark
ms.date: 09/13/2020
ms.openlocfilehash: 73b18d15ad054f1c485d6f61cdefe54993148bc4
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96450539"
---
# <a name="visualize-data"></a>Visualiser les données
Azure Synapse est un service d’analytique intégré qui accélère la génération d’insights dans les entrepôts de données et les systèmes d’analytique de Big Data. La visualisation des données est un composant clé qui permet d’obtenir des informations sur vos données. Elle permet de rendre les données, petites et grandes, plus faciles à comprendre pour les êtres humains. Elle facilite également la détection des modèles, des tendances et des valeurs hors norme dans des groupes de données. 

Lors de l’utilisation d’Apache Spark dans Azure Synapse Analytics, il existe différentes options intégrées pour vous aider à visualiser vos données, notamment des options de graphique pour notebook Synapse, l’accès aux bibliothèques open source populaires et l’intégration à Synapse SQL et à Power BI.

## <a name="notebook-chart-options"></a>Options de graphique du notebook
Lorsque vous utilisez un notebook Azure Synapse, vous pouvez transformer votre vue des résultats tabulaires en graphique personnalisé à l’aide des options de graphique. Ici, vous pouvez visualiser vos données sans avoir à écrire de code. 

### <a name="displaydf-function"></a>Fonction display(df)
La fonction ```display``` vous permet de transformer des requêtes SQL et des DataFrames et RDD Apache Spark en visualisations de données enrichies. La fonction ```display``` peut être utilisée sur des DataFrames ou des RDD créé en PySpark, Scala, Java et .NET.

Pour accéder aux options de graphique :
1. La sortie des commandes magic ```%%sql``` s’affiche par défaut dans l’affichage Table rendu. Vous pouvez également appeler la commande ```display(df)``` sur des DataFrames Spark ou la fonction RDD (Resilient Distributed Datasets) pour produire l’affichage de table rendu. 
   
2. Une fois que vous disposez d’une vue de table affichée, basculez sur la vue de graphique.
   ![built-in-charts](./media/apache-spark-development-using-notebooks/synapse-built-in-charts.png#lightbox)

3. Vous pouvez désormais personnaliser votre visualisation en spécifiant les valeurs suivantes :
   | Configuration | Description |
   |--|--| 
   | Type de graphique | La fonction ```display``` prend en charge un large éventail de types de graphiques, notamment les graphiques à barres, les nuages de points, les graphiques en courbes, etc. |
   | Clé | Spécifier la plage de valeurs pour l’axe X|
   | Value | Spécifier la plage de valeurs pour l’axe Y |
   | Groupe de séries | Utilisé pour déterminer les groupes de l’agrégation | 
   | Agrégation | Méthode pour agréger des données dans votre visualisation| 
   
   
    > [!NOTE]
    > Par défaut, la fonction ```display(df)``` prend uniquement les 1 000 premières lignes de données pour afficher les graphiques. Vérifiez l’**Agrégation par rapport à tous les résultats** et cliquez sur le bouton **Appliquer**, vous allez appliquer la génération de graphique depuis le jeu de données complet. Un travail Spark est déclenché lorsque le paramètre du graphique change. Notez que l’exécution du calcul et l’affichage du graphique peuvent prendre plusieurs minutes.
   
4. Une fois l’opération terminée, vous pouvez afficher et interagir avec votre visualisation finale.

### <a name="displaydf-statistic-details"></a>Détails des statistiques de display(df)
Vous pouvez utiliser <code>display(df, summary = true)</code> pour vérifier le résumé des statistiques d’un DataFrame Apache Spark donné qui inclut le nom de colonne, le type de colonne, les valeurs uniques et les valeurs manquantes pour chaque colonne. Vous pouvez également sélectionner une colonne spécifique pour afficher ses valeurs minimale, maximale et moyenne, ainsi que son écart type.
    ![built-in-charts-summary](./media/apache-spark-development-using-notebooks/synapse-built-in-charts-summary.png#lightbox)
   
### <a name="displayhtmldf-option"></a>Option displayHTML(df)
Les notebooks Azure Synapse Analytics prennent en charge les graphiques HTML à l’aide de la fonction ```displayHTML```.

L’image suivante est un exemple de création de visualisations à l’aide de [D3.js](https://d3js.org/).

   ![d3-js-example](./media/apache-spark-data-viz/d3-boxplot.png#lightbox)

Exécutez le code suivant pour créer la visualisation ci-dessus.

```python
displayHTML("""<!DOCTYPE html>
<meta charset="utf-8">

<!-- Load d3.js -->
<script src="https://d3js.org/d3.v4.js"></script>

<!-- Create a div where the graph will take place -->
<div id="my_dataviz"></div>
<script>

// set the dimensions and margins of the graph
var margin = {top: 10, right: 30, bottom: 30, left: 40},
  width = 400 - margin.left - margin.right,
  height = 400 - margin.top - margin.bottom;

// append the svg object to the body of the page
var svg = d3.select("#my_dataviz")
.append("svg")
  .attr("width", width + margin.left + margin.right)
  .attr("height", height + margin.top + margin.bottom)
.append("g")
  .attr("transform",
        "translate(" + margin.left + "," + margin.top + ")");

// Create Data
var data = [12,19,11,13,12,22,13,4,15,16,18,19,20,12,11,9]

// Compute summary statistics used for the box:
var data_sorted = data.sort(d3.ascending)
var q1 = d3.quantile(data_sorted, .25)
var median = d3.quantile(data_sorted, .5)
var q3 = d3.quantile(data_sorted, .75)
var interQuantileRange = q3 - q1
var min = q1 - 1.5 * interQuantileRange
var max = q1 + 1.5 * interQuantileRange

// Show the Y scale
var y = d3.scaleLinear()
  .domain([0,24])
  .range([height, 0]);
svg.call(d3.axisLeft(y))

// a few features for the box
var center = 200
var width = 100

// Show the main vertical line
svg
.append("line")
  .attr("x1", center)
  .attr("x2", center)
  .attr("y1", y(min) )
  .attr("y2", y(max) )
  .attr("stroke", "black")

// Show the box
svg
.append("rect")
  .attr("x", center - width/2)
  .attr("y", y(q3) )
  .attr("height", (y(q1)-y(q3)) )
  .attr("width", width )
  .attr("stroke", "black")
  .style("fill", "#69b3a2")

// show median, min and max horizontal lines
svg
.selectAll("toto")
.data([min, median, max])
.enter()
.append("line")
  .attr("x1", center-width/2)
  .attr("x2", center+width/2)
  .attr("y1", function(d){ return(y(d))} )
  .attr("y2", function(d){ return(y(d))} )
  .attr("stroke", "black")
</script>

"""
)

```

## <a name="popular-libraries"></a>Bibliothèques populaires
Lorsqu’il s’agit de la visualisation des données, Python propose plusieurs bibliothèques de graphisme qui sont dotées de nombreuses fonctionnalités différentes. Par défaut, chaque pool Apache Spark dans Azure Synapse Analytics contient un ensemble de bibliothèques open source organisées et populaires. Vous pouvez également ajouter ou gérer des bibliothèques et des versions supplémentaires en utilisant les capacités de gestion de bibliothèque d’Azure Synapse Analytics. 

### <a name="bokeh"></a>Bokeh
Vous pouvez afficher des bibliothèques HTML ou interactives, telle **bokeh**, en utilisant la commande ```displayHTML(df)``` . 

L’image suivante est un exemple de traçage de glyphes sur une carte en utilisant **bokeh**.

   ![bokeh-exemple](./media/apache-spark-development-using-notebooks/synapse-bokeh-image.png#lightbox)

Exécutez l’exemple de code suivant pour dessiner l’image ci-dessus.

```python
from bokeh.plotting import figure, output_file
from bokeh.tile_providers import get_provider, Vendors
from bokeh.embed import file_html
from bokeh.resources import CDN
from bokeh.models import ColumnDataSource

tile_provider = get_provider(Vendors.CARTODBPOSITRON)

# range bounds supplied in web mercator coordinates
p = figure(x_range=(-9000000,-8000000), y_range=(4000000,5000000),
           x_axis_type="mercator", y_axis_type="mercator")
p.add_tile(tile_provider)

# plot datapoints on the map
source = ColumnDataSource(
    data=dict(x=[ -8800000, -8500000 , -8800000],
              y=[4200000, 4500000, 4900000])
)

p.circle(x="x", y="y", size=15, fill_color="blue", fill_alpha=0.8, source=source)

# create an html document that embeds the Bokeh plot
html = file_html(p, CDN, "my plot1")

# display this html
displayHTML(html)
```

### <a name="matplotlib"></a>Matplotlib
Vous pouvez restituer des bibliothèques de traçage standard, comme Matplotlib, en utilisant les fonctions de rendu intégrées pour chaque bibliothèque.

L’image suivante est un exemple de création d’un graphique à barres à l’aide de **Matplotlib**.
   ![Exemple de graphique en courbes.](./media/apache-spark-data-viz/matplotlib-example.png#lightbox)

Exécutez l’exemple de code suivant pour dessiner l’image ci-dessus.

```python
# Bar chart

import matplotlib.pyplot as plt

x1 = [1, 3, 4, 5, 6, 7, 9]
y1 = [4, 7, 2, 4, 7, 8, 3]

x2 = [2, 4, 6, 8, 10]
y2 = [5, 6, 2, 6, 2]

plt.bar(x1, y1, label="Blue Bar", color='b')
plt.bar(x2, y2, label="Green Bar", color='g')
plt.plot()

plt.xlabel("bar number")
plt.ylabel("bar height")
plt.title("Bar Chart Example")
plt.legend()
plt.show()
```

### <a name="additional-libraries"></a>Bibliothèques supplémentaires 
Au-delà de ces bibliothèques, le runtime Azure Synapse Analytics comprend également l’ensemble suivant de bibliothèques qui sont souvent utilisées pour la visualisation de données :
- [Matplotlib](https://matplotlib.org/)
- [Bokeh](https://bokeh.org/)
- [Seaborn](https://seaborn.pydata.org/) 

Vous pouvez consulter la [documentation](./spark/../apache-spark-version-support.md) du runtime Azure Synapse Analytics pour obtenir les informations les plus récentes sur les bibliothèques et versions disponibles.

## <a name="connect-to-power-bi-using-apache-spark--sql-on-demand"></a>Se connecter à Power BI à l’aide d’Apache Spark et du SQL à la demande
Azure Synapse Analytics s’intègre parfaitement à Power BI, permettant aux ingénieurs de données de créer des solutions d’analytique.

Azure Synapse Analytics permet aux différents moteurs de calcul d’espace de travail de partager des bases de données et des tables entre ses pools Spark et le pool SQL serverless. À l’aide du [modèle de métadonnées partagées](https://docs.microsoft.com/azure/synapse-analytics/metadata/overview), vous pouvez interroger vos tables Apache Spark à l’aide du SQL à la demande. Une fois que vous avez terminé, vous pouvez connecter votre point de terminaison SQL à la demande à Power BI pour interroger facilement vos tables Spark synchronisées.


## <a name="next-steps"></a>Étapes suivantes
- Pour plus d’informations sur la configuration du connecteur Spark SQL DW : [Connecteur Synapse SQL](./spark/../synapse-spark-sql-pool-import-export.md)
- Afficher les bibliothèques par défaut : [Runtime Azure Synapse Analytics](../spark/apache-spark-version-support.md)