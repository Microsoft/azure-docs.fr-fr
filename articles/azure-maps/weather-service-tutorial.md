---
title: 'Tutoriel : Joindre des données de capteur à des données de prévisions météorologiques à l’aide d’Azure Notebooks (Python) avec Microsoft Azure Maps'
description: Tutoriel montrant comment joindre des données de capteur à des prévisions météorologiques à partir des services Météo de Microsoft Azure Maps avec Azure Notebooks (Python).
author: anastasia-ms
ms.author: v-stharr
ms.date: 12/07/2020
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: mvc, devx-track-python
ms.openlocfilehash: 6d2ede8ab49b22a22d8959ce296182a2210640d0
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96905466"
---
# <a name="tutorial-join-sensor-data-with-weather-forecast-data-by-using-azure-notebooks-python"></a>Tutoriel : Joindre des données de capteur à des données de prévisions météorologiques avec Azure Notebooks (Python)

> [!IMPORTANT]
> Les services Météo d’Azure Maps sont disponibles en préversion publique.
> Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

L’énergie éolienne est une alternative aux énergies fossiles qui permet de lutter contre le changement climatique. Étant donné que le vent n’est, par nature, pas constant, les opérateurs éoliens doivent créer des modèles Machine Learning (ML) pour prédire la capacité d’énergie éolienne. Cette prédiction est nécessaire pour répondre à la demande en électricité et garantir la stabilité du réseau électrique. Dans ce tutoriel, nous allons voir comment les données de prévisions météorologiques Azure Maps sont combinées avec des données de démonstration concernant des relevés météorologiques. Les données de prévisions météorologiques peuvent être demandées en appelant les services Météo d’Azure Maps (préversion).

Ce didacticiel présente les procédures suivantes :

> [!div class="checklist"]
> * Utiliser des fichiers de données dans [Azure Notebooks](../notebooks/index.yml), dans le cloud.
> * Charger les données de démonstration à partir d’un fichier.
> * Appeler les API REST Azure Maps en Python.
> * Afficher les données d’emplacement sur la carte.
> * Enrichissez les données de démonstration avec les données météorologiques issues de l’[API Daily Forecast](/rest/api/maps/weather/getdailyforecastpreview) d’Azure Maps.
> * Créez des graphiques à partir des données de prévisions.


## <a name="prerequisites"></a>Prérequis

Pour suivre ce tutoriel, vous devez d’abord :

1. Créer un abonnement de compte Azure Maps au niveau tarifaire S0 en suivant les instructions fournies dans [Créer un compte](quick-demo-map-app.md#create-an-azure-maps-account).
2. Obtenir la clé primaire d’abonnement destinée à votre compte en suivant les instructions fournies dans [Obtenir la clé primaire](quick-demo-map-app.md#get-the-primary-key-for-your-account).


Pour plus d’informations sur l’authentification dans Azure Maps, voir [Gérer l’authentification dans Azure Maps](./how-to-manage-authentication.md).

Pour vous familiariser avec les notebooks Azure et commencer à les utiliser, suivez les instructions fournies dans [Créer un notebook Azure](./tutorial-ev-routing.md#create-an-azure-notebooks-project).

> [!Note]
> Vous pouvez télécharger le fichier de notebook Jupyter de ce projet dans le [dépôt de notebooks Jupyter Weather Maps](https://github.com/Azure-Samples/Azure-Maps-Jupyter-Notebook/tree/master/AzureMapsJupyterSamples/Tutorials/Analyze%20Weather%20Data).

## <a name="load-the-required-modules-and-frameworks"></a>Charger les modules et les frameworks nécessaires

Pour charger tous les modules et frameworks exigés, exécutez le script suivant :

```python
import pandas as pd
import datetime
from IPython.display import Image, display
!pip install aiohttp
import aiohttp
```

## <a name="import-weather-data"></a>Importer des données météorologiques

Dans le cadre de ce tutoriel, nous utiliserons les relevés de données météorologiques obtenus à partir de capteurs installés sur quatre éoliennes différentes. Les exemples de données sont constitués de relevés météorologiques effectués sur une période de 30 jours. Ces relevés sont collectés par des centres de données météorologiques situés à proximité de chaque éolienne. Les données de démonstration contiennent des données de relevés relatives à la température, à la vitesse du vent et à la direction du vent. Vous pouvez télécharger les données de démonstration [ici](https://github.com/Azure-Samples/Azure-Maps-Jupyter-Notebook/tree/master/AzureMapsJupyterSamples/Tutorials/Analyze%20Weather%20Data/data). Le script ci-dessous importe les données de démonstration dans le notebook Azure.

```python
df = pd.read_csv("./data/weather_dataset_demo.csv")
```

## <a name="request-daily-forecast-data"></a>Demander des données de prévisions quotidiennes

Dans notre scénario, nous souhaitons demander des prévisions quotidiennes pour chaque emplacement de capteur. Le script suivant appelle l’[API de prévision quotidienne](/rest/api/maps/weather/getdailyforecastpreview) des services Météo d’Azure Maps (préversion). Cette API retourne les prévisions météorologiques pour chaque éolienne, pour les 15 prochains jours à compter de la date actuelle.


```python
subscription_key = "Your Azure Maps key"

# Get a lists of unique station IDs and their coordinates 
station_ids = pd.unique(df[['StationID']].values.ravel())
coords = pd.unique(df[['latitude','longitude']].values.ravel())

years,months,days = [],[],[]
dates_check=set()
wind_speeds, wind_direction = [], []

# Call azure maps Weather services (Preview) to get daily forecast data for 15 days from current date
session = aiohttp.ClientSession()
j=-1
for i in range(0, len(coords), 2):
    wind_speeds.append([])
    wind_direction.append([])
    
    query = str(coords[i])+', '+str(coords[i+1])
    forecast_response = await(await session.get("https://atlas.microsoft.com/weather/forecast/daily/json?query={}&api-version=1.0&subscription-key={}&duration=15".format(query, subscription_key))).json()
    j+=1
    for day in range(len(forecast_response['forecasts'])):
            date = forecast_response['forecasts'][day]['date'][:10]
            wind_speeds[j].append(forecast_response['forecasts'][day]['day']['wind']['speed']['value'])
            wind_direction[j].append(forecast_response['forecasts'][day]['day']['windGust']['direction']['degrees'])
            
            if date not in dates_check:
                year,month,day= date.split('-')
                years.append(year)
                months.append(month)
                days.append(day)
                dates_check.add(date)
            
await session.close()
```

Le script ci-dessous affiche l’emplacement des éoliennes sur la carte en appelant le [service Get Map Image](/rest/api/maps/render/getmapimage) d’Azure Maps.

```python
# Render the turbine locations on the map by calling the Azure Maps Get Map Image service
session = aiohttp.ClientSession()

pins="default|la-25+60|ls12|lc003C62|co9B2F15||'Location A'{} {}|'Location B'{} {}|'Location C'{} {}|'Location D'{} {}".format(coords[1],coords[0],coords[3],coords[2],coords[5],coords[4], coords[7],coords[6])

image_response = "https://atlas.microsoft.com/map/static/png?subscription-key={}&api-version=1.0&layer=basic&style=main&zoom=6&center={},{}&pins={}".format(subscription_key,coords[7],coords[6],pins)

static_map_response = await session.get(image_response)

poi_range_map = await static_map_response.content.read()

await session.close()

display(Image(poi_range_map))
```

![Emplacements des éoliennes](./media/weather-service-tutorial/location-map.png)


Nous allons regrouper les données de prévisions avec les données de démonstration en nous basant sur l’ID de station. L’ID de station correspond au centre de données météo. Ce regroupement complète les données de démonstration avec les données de prévisions.

```python
# Group forecasted data for all locations
df = df.reset_index(drop=True)
forecast_data = pd.DataFrame(columns=['StationID','latitude','longitude','Year','Month','Day','DryBulbCelsius','WetBulbFarenheit','WetBulbCelsius','DewPointFarenheit','DewPointCelsius','RelativeHumidity','WindSpeed','WindDirection'])

for i in range(len(station_ids)):
    loc_forecast = pd.DataFrame({'StationID':station_ids[i], 'latitude':coords[0], 'longitude':coords[1], 'Year':years, 'Month':months, 'Day':days, 'WindSpeed':wind_speeds[i], 'WindDirection':wind_direction[i]})
    forecast_data = pd.concat([forecast_data,loc_forecast], axis=0, sort=False)
    
combined_weather_data = pd.concat([df,forecast_data])
grouped_weather_data = combined_weather_data.groupby(['StationID'])
```

Le tableau suivant contient la combinaison des données d’historique et des données de prévisions d’un emplacement d’éolienne.

```python
# Display data for first location
grouped_weather_data.get_group(station_ids[0]).reset_index()
```

<center>

![Données groupées](./media/weather-service-tutorial/grouped-data.png)</center>

## <a name="plot-forecast-data"></a>Créer un graphique à partir des données de prévisions

Nous allons créer un graphique comprenant les valeurs de prévisions et les jours pour lesquels elles sont prévues. Ce tracé nous permet de voir les changements de vitesse et de direction du vent pour les 15 prochains jours.

```python
# Plot wind speed
curr_date = datetime.datetime.now().date()
windsPlot_df = pd.DataFrame({ 'Location A': wind_speeds[0], 'Location B': wind_speeds[1], 'Location C': wind_speeds[2], 'Location D': wind_speeds[3]}, index=pd.date_range(curr_date,periods=15))
windsPlot = windsPlot_df.plot.line()
windsPlot.set_xlabel("Date")
windsPlot.set_ylabel("Wind speed")
```

```python
#Plot wind direction 
windsPlot_df = pd.DataFrame({ 'Location A': wind_direction[0], 'Location B': wind_direction[1], 'Location C': wind_direction[2], 'Location D': wind_direction[3]}, index=pd.date_range(curr_date,periods=15))
windsPlot = windsPlot_df.plot.line()
windsPlot.set_xlabel("Date")
windsPlot.set_ylabel("Wind direction")
```

Les graphiques permettent de visualiser les données de prévisions. Pour connaître les variations prévues concernant la vitesse du vent, consultez le graphique de gauche. Pour connaître les variations prévues concernant la direction du vent, consultez le graphique de droite. Ces données constituent une prédiction pour les 15 prochains jours à compter de la date à laquelle les données sont demandées.

<center>

![Tracé de la vitesse du vent](./media/weather-service-tutorial/speed-date-plot.png) ![Tracé de la direction du vent](./media/weather-service-tutorial/direction-date-plot.png)</center>

Dans ce tutoriel, vous avez appris comment appeler les API REST Azure Maps pour obtenir des données de prévisions météorologiques. Vous avez également appris à visualiser ces données dans des graphiques.

Pour en savoir plus sur l’appel des API REST Azure Maps dans Azure Notebooks, consultez [Router les véhicules électriques avec Azure Notebooks](./tutorial-ev-routing.md).

Pour explorer les API Azure Maps qui sont utilisées dans ce tutoriel, consultez :

* [Daily Forecast](/rest/api/maps/weather/getdailyforecastpreview)
* [Rendu - Obtenir une image de carte](/rest/api/maps/render/getmapimage)

Pour obtenir la liste complète des API REST Azure Maps, consultez [API REST Azure Maps](./consumption-model.md).

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur Azure Notebooks, consultez

> [!div class="nextstepaction"]
> [Azure Notebooks](../notebooks/index.yml)