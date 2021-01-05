---
title: 'Tutoriel : Créer un notebook dans Azure Cosmos DB pour analyser et visualiser les données'
description: 'Tutoriel : Découvrez comment utiliser les notebooks Jupyter intégrés pour importer des données dans Azure Cosmos DB, analyser les données et visualiser la sortie.'
author: deborahc
ms.topic: tutorial
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.date: 11/05/2019
ms.author: dech
ms.reviewer: sngun
ms.openlocfilehash: e16a738264e64e37cfa42722832dac7e34fee899
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93339493"
---
# <a name="tutorial-create-a-notebook-in-azure-cosmos-db-to-analyze-and-visualize-the-data"></a>Tutoriel : Créer un notebook dans Azure Cosmos DB pour analyser et visualiser les données
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Cet article explique comment utiliser les notebooks Jupyter intégrés pour importer des exemples de données de vente dans Azure Cosmos DB. Vous verrez comment utiliser les commandes magic SQL et Azure Cosmos DB pour exécuter des requêtes, analyser les données et visualiser les résultats.

## <a name="prerequisites"></a>Prérequis

* [Activer la prise en charge des notebooks lors de la création du compte Azure Cosmos](enable-notebooks.md)

## <a name="create-the-resources-and-import-data"></a>Créer les ressources et importer des données
 
Dans cette section, vous allez créer la base de données Azure Cosmos et le conteneur, puis importer les données de vente dans le conteneur.

1. Accédez à votre compte Azure Cosmos et ouvrez le volet **Data Explorer** (Explorateur de données).

1. Accédez à l’onglet **Notebooks** , sélectionnez `…` en regard de **My Notebooks** et créez un notebook ( **New Notebook** ). Sélectionnez **Python 3** comme noyau par défaut.

   :::image type="content" source="./media/create-notebook-visualize-data/create-new-notebook.png" alt-text="Créer un notebook":::

1. Une fois que vous avez créé un notebook, vous pouvez le renommer par exemple **VisualizeRetailData.ipynb**.

1. Ensuite, vous allez créer une base de données nommée « RetailDemo » et un conteneur nommé « WebsiteData » pour stocker les données de vente. Vous pouvez utiliser /CartID comme clé de partition. Copiez et collez le code suivant dans une nouvelle cellule de votre notebook, puis exécutez-le :

   ```python
   import azure.cosmos
   from azure.cosmos.partition_key import PartitionKey

   database = cosmos_client.create_database_if_not_exists('RetailDemo')
   print('Database RetailDemo created')

   container = database.create_container_if_not_exists(id='WebsiteData', partition_key=PartitionKey(path='/CartID'))
   print('Container WebsiteData created')
   ```

   Pour exécuter une cellule, sélectionnez `Shift + Enter` ou sélectionnez la cellule et choisissez l’option **Run Active Cell** (Exécuter la cellule active) dans la barre de navigation de l’Explorateur de données.

   :::image type="content" source="./media/create-notebook-visualize-data/run-active-cell.png" alt-text="Exécuter la cellule active":::

   La base de données et le conteneur sont créés dans votre compte Azure Cosmos actuel. Le conteneur est provisionné avec 400 RU/s. La sortie suivante s’affiche après la création de la base de données et du conteneur. 

   ```console
    Database RetailDemo created
    Container WebsiteData created
   ```

   Vous pouvez également actualiser l’onglet **Data** et voir les ressources nouvellement créées :

   :::image type="content" source="media/create-notebook-visualize-data/refresh-data-tab.png" alt-text="Actualiser l’onglet des données pour voir le nouveau conteneur":::

1. Ensuite, vous allez importer les exemples de données de vente dans le conteneur Azure Cosmos. Voici le format d’un élément des données de vente :

   ```json
    {
       "CartID":5399,
       "Action":"Viewed",
       "Item":"Cosmos T-shirt",
       "Price":350,
       "UserName":"Demo.User10",
       "Country":"Iceland",
       "EventDate":"2015-06-25T00:00:00",
       "Year":2015,"Latitude":-66.8673,
       "Longitude":-29.8214,
       "Address":"852 Modesto Loop, Port Ola, Iceland",
       "id":"00ffd39c-7e98-4451-9b91-b2bcf2f9a32d"
    }
   ```

   Dans le cadre du tutoriel, les exemples de données de vente sont stockés dans Stockage Blob Azure. Vous pouvez les importer dans le conteneur Azure Cosmos en collant le code suivant dans une nouvelle cellule. Vous pouvez vérifier que les données ont bien été importées en exécutant une requête pour sélectionner le nombre d’éléments.

   ```python
    # Read data from storage
    import urllib.request, json

    with urllib.request.urlopen("https://cosmosnotebooksdata.blob.core.windows.net/notebookdata/websiteData.json") as url:
      data = json.loads(url.read().decode())

    print("Importing data. This will take a few minutes...\n")

    for event in data:
     try: 
        container.upsert_item(body=event)
     except errors.CosmosHttpResponseError as e:
        raise
        
    ## Run a query against the container to see number of documents
    query = 'SELECT VALUE COUNT(1) FROM c'
    result = list(container.query_items(query, enable_cross_partition_query=True))

    print('Container with id \'{0}\' contains \'{1}\' items'.format(container.id, result[0]))
   ```

   Quand vous exécutez la requête précédente, la sortie suivante est retournée :

   ```console
   Importing data. This will take a few minutes...

   Container with id 'WebsiteData' contains '2654' items
   ```

## <a name="get-your-data-into-a-dataframe"></a>Placer vos données dans un DataFrame

Avant d’exécuter des requêtes pour analyser les données, vous pouvez lire les données du conteneur dans un [DataFrame Pandas](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.html) à des fins d’analyse. Utilisez la commande magic SQL suivante pour lire les données dans un DataFrame :

```bash
%%sql --database {database_id} --container {container_id} --output outputDataframeVar 
{Query text}
```

Pour plus d’informations, consultez l’article [Utiliser les fonctionnalités et commandes de notebook intégrées dans Azure Cosmos DB](use-python-notebook-features-and-commands.md). Vous allez exécuter la requête `SELECT c.Action, c.Price as ItemRevenue, c.Country, c.Item FROM c`. Les résultats seront enregistrés dans un DataFrame Pandas nommé df_cosmos. Collez la commande suivante dans une nouvelle cellule de notebook et exécutez-la :

```python
%%sql --database RetailDemo --container WebsiteData --output df_cosmos
SELECT c.Action, c.Price as ItemRevenue, c.Country, c.Item FROM c
```

Dans une nouvelle cellule de notebook, exécutez le code suivant pour lire les 10 premiers éléments de la sortie :

```python
# See a sample of the result
df_cosmos.head(10)
```

:::image type="content" source="./media/create-notebook-visualize-data/run-query-get-top10-items.png" alt-text="Exécuter la requête pour obtenir les 10 premiers articles":::

## <a name="run-queries-and-analyze-your-data"></a>Exécuter des requêtes et analyser vos données

Dans cette section, vous allez exécuter des requêtes sur les données récupérées.

* **Requête 1**  : exécutez une requête Group by sur le DataFrame pour obtenir la somme des chiffres d’affaires pour chaque pays/région et afficher cinq éléments des résultats. Dans une nouvelle cellule de notebook, exécutez le code suivant :

   ```python
   df_revenue = df_cosmos.groupby("Country").sum().reset_index()
   display(df_revenue.head(5))
   ```

   :::image type="content" source="./media/create-notebook-visualize-data/total-sales-revenue-output.png" alt-text="Sortie du chiffre d’affaires total":::

* **Requête 2** : pour obtenir la liste des cinq éléments les plus achetés, ouvrez une nouvelle cellule de notebook et exécutez le code suivant :

   ```python
   import pandas as pd

   ## What are the top 5 purchased items?
   pd.DataFrame(df_cosmos[df_cosmos['Action']=='Purchased'].groupby('Item').size().sort_values(ascending=False).head(5), columns=['Count'])
   ```

   :::image type="content" source="./media/create-notebook-visualize-data/top5-purchased-items.png" alt-text="Cinq articles les plus achetés":::

## <a name="visualize-your-data"></a>Visualiser vos données  

1. Maintenant que vous disposez de vos données de chiffres d’affaires du conteneur Azure Cosmos, vous pouvez visualiser vos données avec la bibliothèque de visualisation de votre choix. Dans ce tutoriel, nous allons utiliser la bibliothèque Bokeh. Ouvrez une nouvelle cellule de notebook et exécutez le code suivant pour installer la bibliothèque Bokeh. Une fois toutes les conditions requises remplies, la bibliothèque est installée.

   ```python
   import sys
   !{sys.executable} -m pip install bokeh --user
   ```

1. Ensuite, préparez le tracé des données sur une carte. Joignez les données dans Azure Cosmos DB aux informations sur les pays/régions qui se trouvent dans Stockage Blob Azure, et convertissez le résultat au format GeoJSON. Copiez le code suivant dans une nouvelle cellule de notebook et exécutez-le.

   ```python
   import urllib.request, json
   import geopandas as gpd

   # Load country/region information for mapping
   countries = gpd.read_file("https://cosmosnotebooksdata.blob.core.windows.net/notebookdata/countries.json")

   # Merge the countries/regions dataframe with our data in Azure Cosmos DB, joining on country/region code
   df_merged = countries.merge(df_revenue, left_on = 'admin', right_on = 'Country', how='left')

   # Convert to GeoJSON so bokeh can plot it
   merged_json = json.loads(df_merged.to_json())
   json_data = json.dumps(merged_json)
   ```

1. Visualisez les chiffres d’affaires des différents pays/régions sur une carte du monde en exécutant le code suivant dans une nouvelle cellule de notebook :

   ```python
   from bokeh.io import output_notebook, show
   from bokeh.plotting import figure
   from bokeh.models import GeoJSONDataSource, LinearColorMapper, ColorBar
   from bokeh.palettes import brewer
    
   #Input GeoJSON source that contains features for plotting.
   geosource = GeoJSONDataSource(geojson = json_data)
    
   #Choose our choropleth color palette: https://bokeh.pydata.org/en/latest/docs/reference/palettes.html
   palette = brewer['YlGn'][8]
    
   #Reverse color order so that dark green is highest revenue
   palette = palette[::-1]
    
   #Instantiate LinearColorMapper that linearly maps numbers in a range, into a sequence of colors.
   color_mapper = LinearColorMapper(palette = palette, low = 0, high = 1000)
    
   #Define custom tick labels for color bar.
   tick_labels = {'0': '$0', '250': '$250', '500':'$500', '750':'$750', '1000':'$1000', '1250':'$1250', '1500':'$1500','1750':'$1750', '2000': '>$2000'}
    
   #Create color bar. 
   color_bar = ColorBar(color_mapper=color_mapper, label_standoff=8,width = 500, height = 20,
   border_line_color=None,location = (0,0), orientation = 'horizontal', major_label_overrides = tick_labels)
    
   #Create figure object.
   p = figure(title = 'Sales revenue by country', plot_height = 600 , plot_width = 1150, toolbar_location = None)
   p.xgrid.grid_line_color = None
   p.ygrid.grid_line_color = None
    
   #Add patch renderer to figure. 
   p.patches('xs','ys', source = geosource,fill_color = {'field' :'ItemRevenue', 'transform' : color_mapper},
              line_color = 'black', line_width = 0.25, fill_alpha = 1)
    
   #Specify figure layout.
   p.add_layout(color_bar, 'below')
    
   #Display figure inline in Jupyter Notebook.
   output_notebook()
   
   #Display figure.
   show(p)
   ```

   La sortie affiche la carte du monde avec différentes couleurs. Les couleurs plus sombres représentent les pays/régions dont le chiffre d’affaires est le plus élevé, et les couleurs plus claires le chiffre d’affaires le plus bas.

   :::image type="content" source="./media/create-notebook-visualize-data/countries-revenue-map-visualization.png" alt-text="Visualisation de la carte des chiffres d’affaires par pays/région":::

1. Examinons un autre cas de visualisation des données. Le conteneur WebsiteData contient des enregistrements d’utilisateurs qui ont visualisé un article, l’ont ajouté à leur panier et l’ont acheté. Nous allons tracer le taux de conversion des articles achetés. Exécutez le code suivant dans une nouvelle cellule afin de visualiser le taux de conversion pour chaque article :

   ```python
   from bokeh.io import show, output_notebook
   from bokeh.plotting import figure
   from bokeh.palettes import Spectral3
   from bokeh.transform import factor_cmap
   from bokeh.models import ColumnDataSource, FactorRange
       
   # Get the top 10 items as an array
   top_10_items = df_cosmos[df_cosmos['Action']=='Purchased'].groupby('Item').size().sort_values(ascending=False)[:10].index.values.tolist()
    
   # Filter our data to only these 10 items
   df_top10 = df_cosmos[df_cosmos['Item'].isin(top_10_items)]
    
   # Group by Item and Action, sorting by event count
   df_top10_sorted = df_top10.groupby(['Item', 'Action']).count().rename(columns={'Country':'ResultCount'}, inplace=False).reset_index().sort_values(['Item', 'ResultCount'], ascending = False).set_index(['Item', 'Action'])
    
   # Get sorted X-axis values - this way, we can display the funnel of view -> add -> purchase
   x_axis_values = df_top10_sorted.index.values.tolist()
    
   group = df_top10_sorted.groupby(['Item', 'Action'])
    
   # Specifiy colors for X axis
   index_cmap = factor_cmap('Item_Action', palette=Spectral3, factors=sorted(df_top10.Action.unique()), start=1, end=2)
    
   # Create the plot
    
   p = figure(plot_width=1200, plot_height=500, title="Conversion rate of items from View -> Add to cart -> Purchase", x_range=FactorRange(*x_axis_values), toolbar_location=None, tooltips=[("Number of events", "@ResultCount_max"), ("Item, Action", "@Item_Action")])
    
   p.vbar(x='Item_Action', top='ItemRevenue_max', width=1, source=group,
           line_color="white", fill_color=index_cmap, )
    
   #Configure how the plot looks
   p.y_range.start = 0
   p.x_range.range_padding = 0.05
   p.xgrid.grid_line_color = None
   p.xaxis.major_label_orientation = 1.2
   p.outline_line_color = "black"
   p.xaxis.axis_label = "Item"
   p.yaxis.axis_label = "Count"
    
   #Display figure inline in Jupyter Notebook.
   output_notebook()
    
   #Display figure.
   show(p)
   ```

   :::image type="content" source="./media/create-notebook-visualize-data/visualize-purchase-conversion-rate.png" alt-text="Visualiser le taux de conversion des achats":::

## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus sur les commandes de notebook Python, consultez l’article [Guide pratique pour utiliser les fonctionnalités et commandes de notebook intégrées dans Azure Cosmos DB](use-python-notebook-features-and-commands.md).
