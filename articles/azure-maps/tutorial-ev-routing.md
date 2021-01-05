---
title: 'Tutoriel : Router les véhicules électriques à l’aide d’Azure Notebooks (Python) avec Microsoft Azure Maps'
description: Tutoriel sur la façon de router les véhicules électriques avec les API de routage Microsoft Azure Maps et Azure Notebooks
author: anastasia-ms
ms.author: v-stharr
ms.date: 12/07/2020
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: mvc, devx-track-python
ms.openlocfilehash: f30b99a1d9c8303d5b2ed4b02819d0ca837946d2
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96905738"
---
# <a name="tutorial-route-electric-vehicles-by-using-azure-notebooks-python"></a>Tutoriel : Router les véhicules électriques avec Azure Notebooks (Python)

Azure Maps est un portefeuille d’API de services géospatiaux intégrées en mode natif dans Azure. Ces API permettent aux développeurs, entreprises et éditeurs de logiciels indépendants de créer des applications utilisant la géolocalisation ainsi que des solutions d’IoT, de mobilité, de logistique et de suivi des ressources. 

Les API REST Azure Maps peuvent être appelées dans des langages tels que Python et R, pour activer l’analyse de données géospatiales et les scénarios de Machine Learning. Azure Maps propose un solide ensemble d’[API de routage](/rest/api/maps/route) qui permettent aux utilisateurs de calculer des itinéraires entre différents points de données. Les calculs sont basés sur différentes conditions, comme le type de véhicule ou la zone accessible. 

Dans ce tutoriel, vous allez aider un conducteur de véhicule électrique dont la batterie est presque vide. Le conducteur doit trouver la borne de recharge la plus proche possible de son véhicule.

Ce didacticiel présente les procédures suivantes :

> [!div class="checklist"]
> * Créer et exécuter un fichier Jupyter Notebook sur [Azure Notebooks](../notebooks/index.yml) dans le cloud.
> * Appeler les API REST Azure Maps en Python.
> * Rechercher une zone accessible en fonction du modèle de consommation du véhicule électrique
> * Rechercher des bornes de recharge de véhicule électrique dans la zone accessible, ou isochrone.
> * Afficher la limite de la zone accessible et les bornes de recharge sur une carte
> * Rechercher et visualiser une route vers la borne de recharge de véhicule électrique la plus proche en temps de conduite.


## <a name="prerequisites"></a>Prérequis 

Pour suivre ce tutoriel, vous devez d’abord créer un compte Azure Maps et obtenir votre clé primaire (clé d’abonnement). 

Pour créer un abonnement de compte Azure Maps, suivez les instructions fournies dans [Créer un compte](quick-demo-map-app.md#create-an-azure-maps-account). Vous avez besoin d’un abonnement de compte Azure Maps avec le niveau de prix S1. 

Pour obtenir la clé primaire d’abonnement destinée à votre compte, suivez les instructions fournies dans [Obtenir la clé primaire](quick-demo-map-app.md#get-the-primary-key-for-your-account).

Pour plus d’informations sur l’authentification dans Azure Maps, voir [Gérer l’authentification dans Azure Maps](./how-to-manage-authentication.md).

## <a name="create-an-azure-notebooks-project"></a>Créer un projet Azure Notebooks

Pour suivre ce tutoriel, vous devez créer un projet Azure Notebooks, puis télécharger et exécuter le fichier Jupyter Notebook. Le fichier Jupyter Notebook contient du code Python qui implémente le scénario de ce tutoriel. Pour créer un projet Azure Notebooks et y charger le document Jupyter Notebook, effectuez les étapes suivantes :

1. Accédez à [Azure Notebooks](https://notebooks.azure.com) et connectez-vous. Pour plus d’informations, consultez [Démarrage rapide : Se connecter et définir un identifiant utilisateur](../notebooks/quickstart-sign-in-azure-notebooks.md).
1. En haut de votre page de profil public, sélectionnez **Mes projets**.

    ![Bouton Mes projets](./media/tutorial-ev-routing/myproject.png)

1. Dans la page **Mes projets**, sélectionnez **Nouveau projet**.
 
   ![Bouton Nouveau projet](./media/tutorial-ev-routing/create-project.png)

1. Dans le panneau **Créer un projet**, entrez un nom et un identifiant de projet.
 
    ![Panneau Créer un projet](./media/tutorial-ev-routing/create-project-window.png)

1. Sélectionnez **Create** (Créer).

1. Une fois que votre projet a été créé, téléchargez ce [fichier de document Jupyter Notebook](https://github.com/Azure-Samples/Azure-Maps-Jupyter-Notebook/blob/master/AzureMapsJupyterSamples/Tutorials/EV%20Routing%20and%20Reachable%20Range/EVrouting.ipynb) à partir du [dépôt Azure Maps Jupyter Notebook](https://github.com/Azure-Samples/Azure-Maps-Jupyter-Notebook).

1. Dans la liste des projets de la page **Mes projets**, sélectionnez votre projet, puis sélectionnez **Charger** pour télécharger le fichier de document Jupyter Notebook. 

    ![chargement de Jupyter Notebook](./media/tutorial-ev-routing/upload-notebook.png)

1. Chargez le fichier à partir de votre ordinateur, puis sélectionnez **Terminé**.

1. Une fois le téléchargement effectué, le fichier s’affiche dans votre page de projets. Double-cliquez sur ce fichier pour l’ouvrir en tant que notebook Jupyter.

Essayez de comprendre les fonctionnalités implémentées dans le fichier Jupyter Notebook. Exécutez le code figurant dans le fichier Jupyter Notebook, une cellule à la fois. Vous pouvez exécuter le code dans chaque cellule en sélectionnant le bouton **Exécuter** situé en haut de l’application Jupyter Notebook.

  ![Bouton Exécuter](./media/tutorial-ev-routing/run.png)

## <a name="install-project-level-packages"></a>Installer des packages au niveau du projet

Pour exécuter le code dans Jupyter Notebook, installez les packages au niveau du projet en effectuant les étapes suivantes :

1. Téléchargez le fichier [*requirements.txt*](https://github.com/Azure-Samples/Azure-Maps-Jupyter-Notebook/blob/master/AzureMapsJupyterSamples/Tutorials/EV%20Routing%20and%20Reachable%20Range/requirements.txt) à partir du [dépôt Azure Maps Jupyter Notebook](https://github.com/Azure-Samples/Azure-Maps-Jupyter-Notebook), puis chargez-le dans votre projet.
1. Dans le tableau de bord du projet, sélectionnez **Paramètres du projet**. 
1. Dans le panneau **Paramètres du projet**, sélectionnez l’onglet **Environnement**, puis **Ajouter**.
1. Sous **Étapes de configuration de l’environnement**, effectuez les étapes suivantes :   
    a. Dans la première liste déroulante, sélectionnez **Requirements.txt**.  
    b. Dans la deuxième liste déroulante, sélectionnez votre fichier *Requirements.txt*.  
    c. Dans la troisième liste déroulante, sélectionnez **Python Version 3.6** comme version.
1. Sélectionnez **Enregistrer**.

    ![Installer des packages](./media/tutorial-ev-routing/install-packages.png)

## <a name="load-the-required-modules-and-frameworks"></a>Charger les modules et les frameworks nécessaires

Pour charger tous les modules et frameworks exigés, exécutez le script suivant.

```Python
import time
import aiohttp
import urllib.parse
from IPython.display import Image, display
```

## <a name="request-the-reachable-range-boundary"></a>Demander les limites de la zone accessible

Une société de livraison de colis compte quelques véhicules électriques dans sa flotte. Pendant la journée, les véhicules électriques doivent être rechargés sans avoir à revenir à l’entrepôt. Chaque fois que la charge restante passe sous la barre d’une heure, vous recherchez un ensemble de bornes de recharge dans une zone accessible. En bref, vous recherchez une borne de recharge quand la charge de la batterie est faible. Vous obtenez également des informations sur les limites de cette zone de bornes de recharge. 

Étant donné que la société préfère utiliser les itinéraires offrant un compromis économie-vitesse, le routeType demandé est *eco*. Le script suivant appelle l’[API d’obtention de zone d’itinéraire](/rest/api/maps/route/getrouterange) du service de routage Azure Maps. Il utilise des paramètres pour le modèle de consommation du véhicule. Le script analyse ensuite la réponse pour créer un objet polygone au format geojson, qui représente la zone maximale accessible du véhicule.

Pour déterminer les limites de la zone accessible du véhicule électrique, exécutez le script de la cellule suivante :

```python
subscriptionKey = "Your Azure Maps key"
currentLocation = [34.028115,-118.5184279]
session = aiohttp.ClientSession()

# Parameters for the vehicle consumption model 
travelMode = "car"
vehicleEngineType = "electric"
currentChargeInkWh=45
maxChargeInkWh=80
timeBudgetInSec=550
routeType="eco"
constantSpeedConsumptionInkWhPerHundredkm="50,8.2:130,21.3"


# Get boundaries for the electric vehicle's reachable range.
routeRangeResponse = await (await session.get("https://atlas.microsoft.com/route/range/json?subscription-key={}&api-version=1.0&query={}&travelMode={}&vehicleEngineType={}&currentChargeInkWh={}&maxChargeInkWh={}&timeBudgetInSec={}&routeType={}&constantSpeedConsumptionInkWhPerHundredkm={}"
                                              .format(subscriptionKey,str(currentLocation[0])+","+str(currentLocation[1]),travelMode, vehicleEngineType, currentChargeInkWh, maxChargeInkWh, timeBudgetInSec, routeType, constantSpeedConsumptionInkWhPerHundredkm))).json()

polyBounds = routeRangeResponse["reachableRange"]["boundary"]

for i in range(len(polyBounds)):
    coordList = list(polyBounds[i].values())
    coordList[0], coordList[1] = coordList[1], coordList[0]
    polyBounds[i] = coordList

polyBounds.pop()
polyBounds.append(polyBounds[0])

boundsData = {
               "geometry": {
                 "type": "Polygon",
                 "coordinates": 
                   [
                      polyBounds
                   ]
                }
             }
```

## <a name="search-for-electric-vehicle-charging-stations-within-the-reachable-range"></a>Rechercher des bornes de recharge de véhicule électrique dans la zone accessible

Après avoir déterminé la zone accessible (isochrone) pour le véhicule électrique, vous pouvez rechercher les bornes de recharge dans cette zone. 

Le script suivant appelle l’[API de recherche dans une géométrie](/rest/api/maps/search/postsearchinsidegeometry) Azure Maps. Il recherche les stations de chargement de véhicules électriques, dans les limites de la plage d’accessibilité maximale de la voiture. Ensuite, le script analyse la réponse dans un tableau de lieux accessibles.

Pour rechercher les bornes de recharge de véhicule électrique dans une zone accessible, exécutez le script suivant :

```python
# Search for electric vehicle stations within reachable range.
searchPolyResponse = await (await session.post(url = "https://atlas.microsoft.com/search/geometry/json?subscription-key={}&api-version=1.0&query=electric vehicle station&idxSet=POI&limit=50".format(subscriptionKey), json = boundsData)).json() 

reachableLocations = []
for loc in range(len(searchPolyResponse["results"])):
                location = list(searchPolyResponse["results"][loc]["position"].values())
                location[0], location[1] = location[1], location[0]
                reachableLocations.append(location)
```

## <a name="upload-the-reachable-range-and-charging-points-to-azure-maps-data-service-preview"></a>Charger la zone accessible et les points de recharge sur le service Data d’Azure Maps (préversion)

Vous souhaitez visualiser sur une carte les bornes de recharge et les limites de la zone accessible pour le véhicule électrique. Pour ce faire, chargez les données des limites et des bornes de recharge en tant qu’objets geojson sur le service Data d’Azure Maps (préversion). Utilisez l’[API de chargement des données](/rest/api/maps/data/uploadpreview). 

Pour charger les données des limites et des points de recharge sur le service Data d’Azure Maps, exécutez les deux cellules suivantes :

```python
rangeData = {
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "properties": {},
      "geometry": {
        "type": "Polygon",
        "coordinates": [
          polyBounds
        ]
      }
    }
  ]
}

# Upload the range data to Azure Maps Data service (Preview).
uploadRangeResponse = await session.post("https://atlas.microsoft.com/mapData/upload?subscription-key={}&api-version=1.0&dataFormat=geojson".format(subscriptionKey), json = rangeData)

rangeUdidRequest = uploadRangeResponse.headers["Location"]+"&subscription-key={}".format(subscriptionKey)

while True:
    getRangeUdid = await (await session.get(rangeUdidRequest)).json()
    if 'udid' in getRangeUdid:
        break
    else:
        time.sleep(0.2)
rangeUdid = getRangeUdid["udid"]
```

```python
poiData = {
    "type": "FeatureCollection",
    "features": [
      {
        "type": "Feature",
        "properties": {},
        "geometry": {
            "type": "MultiPoint",
            "coordinates": reachableLocations
        }
    }
  ]
}

# Upload the electric vehicle charging station data to Azure Maps Data service (Preview).
uploadPOIsResponse = await session.post("https://atlas.microsoft.com/mapData/upload?subscription-key={}&api-version=1.0&dataFormat=geojson".format(subscriptionKey), json = poiData)

poiUdidRequest = uploadPOIsResponse.headers["Location"]+"&subscription-key={}".format(subscriptionKey)

while True:
    getPoiUdid = await (await session.get(poiUdidRequest)).json()
    if 'udid' in getPoiUdid:
        break
    else:
        time.sleep(0.2)
poiUdid = getPoiUdid["udid"]
```

## <a name="render-the-charging-stations-and-reachable-range-on-a-map"></a>Afficher les bornes de recharge et la zone accessible sur une carte

Une fois les données chargées dans le service de données, appelez le [service Get Map Image](/rest/api/maps/render/getmapimage) d’Azure Maps. Ce service permet d’afficher les points de recharge et les limites de la zone accessible sur l’image de carte statique. Pour cela, exécutez le script suivant :

```python
# Get boundaries for the bounding box.
def getBounds(polyBounds):
    maxLon = max(map(lambda x: x[0], polyBounds))
    minLon = min(map(lambda x: x[0], polyBounds))

    maxLat = max(map(lambda x: x[1], polyBounds))
    minLat = min(map(lambda x: x[1], polyBounds))
    
    # Buffer the bounding box by 10 percent to account for the pixel size of pins at the ends of the route.
    lonBuffer = (maxLon-minLon)*0.1
    minLon -= lonBuffer
    maxLon += lonBuffer

    latBuffer = (maxLat-minLat)*0.1
    minLat -= latBuffer
    maxLat += latBuffer
    
    return [minLon, maxLon, minLat, maxLat]

minLon, maxLon, minLat, maxLat = getBounds(polyBounds)

path = "lcff3333|lw3|la0.80|fa0.35||udid-{}".format(rangeUdid)
pins = "custom|an15 53||udid-{}||https://raw.githubusercontent.com/Azure-Samples/AzureMapsCodeSamples/master/AzureMapsCodeSamples/Common/images/icons/ev_pin.png".format(poiUdid)

encodedPins = urllib.parse.quote(pins, safe='')

# Render the range and electric vehicle charging points on the map.
staticMapResponse =  await session.get("https://atlas.microsoft.com/map/static/png?api-version=1.0&subscription-key={}&pins={}&path={}&bbox={}&zoom=12".format(subscriptionKey,encodedPins,path,str(minLon)+", "+str(minLat)+", "+str(maxLon)+", "+str(maxLat)))

poiRangeMap = await staticMapResponse.content.read()

display(Image(poiRangeMap))
```

![Carte représentant la zone localisée](./media/tutorial-ev-routing/location-range.png)


## <a name="find-the-optimal-charging-station"></a>Rechercher la borne de recharge optimale

Tout d’abord, vous souhaitez trouver toutes les bornes de recharge potentielles dans la zone accessible. Ensuite, vous souhaitez savoir celles que vous pouvez atteindre en un minimum de temps. 

Le script suivant appelle l’[API de routage par matrice](/rest/api/maps/route/postroutematrix) d’Azure Maps. Elle retourne l’emplacement du véhicule spécifié, le temps de trajet et la distance jusqu’à chaque borne de recharge. Le script de la cellule suivante analyse la réponse pour localiser la borne de recharge accessible la plus proche en temps.

Pour trouver la borne de recharge la plus proche, qui peut être atteinte en moins de temps possible, exécutez le script de la cellule suivante :

```python
locationData = {
            "origins": {
              "type": "MultiPoint",
              "coordinates": [[currentLocation[1],currentLocation[0]]]
            },
            "destinations": {
              "type": "MultiPoint",
              "coordinates": reachableLocations
            }
         }

# Get the travel time and distance to each specified charging station.
searchPolyRes = await (await session.post(url = "https://atlas.microsoft.com/route/matrix/json?subscription-key={}&api-version=1.0&routeType=shortest&waitForResults=true".format(subscriptionKey), json = locationData)).json()

distances = []
for dist in range(len(reachableLocations)):
    distances.append(searchPolyRes["matrix"][0][dist]["response"]["routeSummary"]["travelTimeInSeconds"])

minDistLoc = []
minDistIndex = distances.index(min(distances))
minDistLoc.extend([reachableLocations[minDistIndex][1], reachableLocations[minDistIndex][0]])
closestChargeLoc = ",".join(str(i) for i in minDistLoc)
```

## <a name="calculate-the-route-to-the-closest-charging-station"></a>Calculer l’itinéraire jusqu’à la borne de recharge la plus proche

Maintenant que vous avez trouvé la borne de recharge la plus proche, vous pouvez appeler l’[API d’obtention d’itinéraire](/rest/api/maps/route/getroutedirections) pour demander l’itinéraire détaillé, depuis l’endroit où se trouve le véhicule électrique jusqu’à la borne de recharge.

Pour obtenir l’itinéraire vers la borne de recharge et analyser la réponse qui permet de créer un objet geojson représentant cet itinéraire, exécutez le script de la cellule suivante :

```python
# Get the route from the electric vehicle's current location to the closest charging station. 
routeResponse = await (await session.get("https://atlas.microsoft.com/route/directions/json?subscription-key={}&api-version=1.0&query={}:{}".format(subscriptionKey, str(currentLocation[0])+","+str(currentLocation[1]), closestChargeLoc))).json()

route = []
for loc in range(len(routeResponse["routes"][0]["legs"][0]["points"])):
                location = list(routeResponse["routes"][0]["legs"][0]["points"][loc].values())
                location[0], location[1] = location[1], location[0]
                route.append(location)

routeData = {
         "type": "LineString",
         "coordinates": route
     }
```

## <a name="visualize-the-route"></a>Visualiser l’itinéraire

Pour mieux visualiser l’itinéraire, chargez tout d’abord les données d’itinéraire en tant qu’objet geojson sur le service Data d’Azure Maps (préversion). Pour cela, utilisez l’[API de chargement de données](/rest/api/maps/data/uploadpreview) d’Azure Maps. Ensuite, appelez le service de rendu, l’[API d’obtention d’image de carte](/rest/api/maps/render/getmapimage), pour afficher l’itinéraire sur la carte et le visualiser.

Pour obtenir une image de l’itinéraire sur la carte, exécutez le script suivant :

```python
# Upload the route data to Azure Maps Data service (Preview).
routeUploadRequest = await session.post("https://atlas.microsoft.com/mapData/upload?subscription-key={}&api-version=1.0&dataFormat=geojson".format(subscriptionKey), json = routeData)

udidRequestURI = routeUploadRequest.headers["Location"]+"&subscription-key={}".format(subscriptionKey)

while True:
    udidRequest = await (await session.get(udidRequestURI)).json()
    if 'udid' in udidRequest:
        break
    else:
        time.sleep(0.2)

udid = udidRequest["udid"]

destination = route[-1]

destination[1], destination[0] = destination[0], destination[1]

path = "lc0f6dd9|lw6||udid-{}".format(udid)
pins = "default|codb1818||{} {}|{} {}".format(str(currentLocation[1]),str(currentLocation[0]),destination[1],destination[0])


# Get boundaries for the bounding box.
minLat, maxLat = (float(destination[0]),currentLocation[0]) if float(destination[0])<currentLocation[0] else (currentLocation[0], float(destination[0]))
minLon, maxLon = (float(destination[1]),currentLocation[1]) if float(destination[1])<currentLocation[1] else (currentLocation[1], float(destination[1]))

# Buffer the bounding box by 10 percent to account for the pixel size of pins at the ends of the route.
lonBuffer = (maxLon-minLon)*0.1
minLon -= lonBuffer
maxLon += lonBuffer

latBuffer = (maxLat-minLat)*0.1
minLat -= latBuffer
maxLat += latBuffer

# Render the route on the map.
staticMapResponse = await session.get("https://atlas.microsoft.com/map/static/png?api-version=1.0&subscription-key={}&&path={}&pins={}&bbox={}&zoom=16".format(subscriptionKey,path,pins,str(minLon)+", "+str(minLat)+", "+str(maxLon)+", "+str(maxLat)))

staticMapImage = await staticMapResponse.content.read()

await session.close()
display(Image(staticMapImage))
```

![Carte indiquant l’itinéraire](./media/tutorial-ev-routing/route.png)

Dans ce tutoriel, vous avez découvert comment appeler les API REST Azure Maps directement et comment visualiser les données Azure Maps avec Python.

Pour explorer les API Azure Maps qui sont utilisées dans ce tutoriel, consultez :

* [Obtenir une zone d’itinéraire](/rest/api/maps/route/getrouterange)
* [Envoyer une recherche dans une géométrie](/rest/api/maps/search/postsearchinsidegeometry)
* [Chargement de données](/rest/api/maps/data/uploadpreview)
* [Rendu - Obtenir une image de carte](/rest/api/maps/render/getmapimage)
* [Envoyer une matrice d’itinéraire](/rest/api/maps/route/postroutematrix)
* [Obtenir un itinéraire](/rest/api/maps/route/getroutedirections)
* [API REST Azure Maps](./consumption-model.md)

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur Azure Notebooks, consultez

> [!div class="nextstepaction"]
> [Azure Notebooks](../notebooks/index.yml)