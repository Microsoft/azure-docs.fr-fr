---
title: Afficher des données de trafic sur une carte Android | Microsoft Azure Maps
description: Dans cet article, vous allez apprendre à afficher des données de trafic sur une carte en utilisant le kit SDK Android Microsoft Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 11/25/2020
ms.topic: how-to
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.openlocfilehash: 5f7e67d159c2b7dea3ebac7fd4d0856f508cb298
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96532752"
---
# <a name="show-traffic-data-on-the-map-using-azure-maps-android-sdk"></a>Afficher des données de trafic sur la carte à l’aide du kit SDK Android Azure Maps

Les données de flux et d’incidents sont deux types de données de trafic qui peut s’afficher sur la carte. Ce guide vous montre comment afficher ces deux types de données de trafic. Les données d’incidents se composent de données de type point et ligne qui peuvent indiquer des travaux, des routes barrées, des accidents, etc. Les données de flux présentent des métriques sur le flux du trafic routier.

## <a name="prerequisites"></a>Conditions préalables requises

1. [Créer un compte Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account)
2. [Obtenir une clé d’abonnement principale](quick-demo-map-app.md#get-the-primary-key-for-your-account), également appelée clé primaire ou clé d’abonnement.
3. Télécharger et installer le [SDK Azure Maps Android](./how-to-use-android-map-control-library.md).

## <a name="incidents-traffic-data"></a>Données d’incidents de trafic

Vous devez importer les bibliothèques suivantes pour appeler `setTraffic` et `incidents` :

```java
import static com.microsoft.com.azure.maps.mapcontrol.options.TrafficOptions.incidents;
```

 L’exemple de code suivant vous montre comment afficher les données de trafic sur la carte. Une valeur booléenne est transmise à la méthode `incidents`, qui la transmet à la méthode `setTraffic`. 

```java
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    mapControl.getMapAsync(map - > {
        map.setTraffic(incidents(true));
    }
}
```

## <a name="flow-traffic-data"></a>Données de flux de trafic

Vous devez d’abord importer les bibliothèques suivantes pour appeler `setTraffic` et `flow` :

```java
import com.microsoft.azure.maps.mapcontrol.options.TrafficFlow;
import static com.microsoft.azure.maps.mapcontrol.options.TrafficOptions.flow;
```

Utilisez l’extrait de code suivant pour définir les données de flux de trafic. Comme pour le code de la section précédente, la valeur de retour de la méthode `flow` est transmise à la méthode `setTraffic`. Les valeurs qui peuvent être transmises à `flow` sont au nombre de quatre, et chacune d’elles déclenche `flow` pour retourner sa propre valeur. La valeur de retour de `flow` est ensuite transmise sous forme d’argument à `setTraffic`. Ces quatre valeurs sont décrites dans le tableau ci-dessous :

|Valeur de flux | Description|
| :-- | :-- |
| TrafficFlow.NONE | N’affiche pas de données de trafic sur la carte |
| TrafficFlow.RELATIVE | Affiche des données de trafic relatives à la vitesse dans des conditions de circulation routière fluide |
| TrafficFlow.RELATIVE_DELAY | Affiche les zones qui sont plus lentes que le délai moyen attendu |
| TrafficFlow.ABSOLUTE | Affiche la vitesse absolue de tous les véhicules circulant sur la route |

```java
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    mapControl.getMapAsync(map -> {
        map.setTraffic(flow(TrafficFlow.RELATIVE)));
    }
}
```

## <a name="show-incident-traffic-data-by-clicking-a-feature"></a>Afficher les données d’incidents de trafic en cliquant sur une fonctionnalité

Pour obtenir les données d’incidents pour une fonctionnalité déterminée, vous pouvez utiliser le code ci-dessous. Quand l’utilisateur clique sur une fonctionnalité, la logique du code effectue une recherche sur les incidents et génère un message sur l’incident. Un message s’affiche au bas de l’écran avec les détails.

1. Tout d’abord, vous devez modifier `res > layout > activity_main.xml` à la manière de l’exemple ci-dessous. Vous pouvez remplacer `mapcontrol_centerLat`, `mapcontrol_centerLng` et `mapcontrol_zoom` par les valeurs souhaitées. Pour rappel, le niveau de zoom est une valeur comprise entre 0 et 22. Au niveau de zoom 0, le monde entier tient dans une seule mosaïque.

   ```XML
   <?xml version="1.0" encoding="utf-8"?>
   <FrameLayout
       xmlns:android="http://schemas.android.com/apk/res/android"
       xmlns:app="http://schemas.android.com/apk/res-auto"
       android:layout_width="match_parent"
       android:layout_height="match_parent"
       >
    
       <com.microsoft.azure.maps.mapcontrol.MapControl
           android:id="@+id/mapcontrol"
           android:layout_width="match_parent"
           android:layout_height="match_parent"
           app:mapcontrol_centerLat="47.6050"
           app:mapcontrol_centerLng="-122.3344"
           app:mapcontrol_zoom="12"
           />

   </FrameLayout>
   ```

2. Ajoutez le code suivant à votre fichier **MainActivity.java**. Le package étant inclus par défaut, veillez à laisser votre package en haut.

   ```java
   package <yourpackagename>;
   import androidx.appcompat.app.AppCompatActivity;

   import android.os.Bundle;
   import android.widget.Toast;

   import com.microsoft.azure.maps.mapcontrol.AzureMaps;
   import com.microsoft.azure.maps.mapcontrol.MapControl;
   import com.mapbox.geojson.Feature;
   import com.microsoft.azure.maps.mapcontrol.events.OnFeatureClick;

   import com.microsoft.azure.maps.mapcontrol.options.TrafficFlow;
   import static com.microsoft.azure.maps.mapcontrol.options.TrafficOptions.flow;
   import static com.microsoft.azure.maps.mapcontrol.options.TrafficOptions.incidents;

   public class MainActivity extends AppCompatActivity {

       static {
           AzureMaps.setSubscriptionKey("Your Azure Maps Subscription Key");
       }

       MapControl mapControl;

       @Override
       protected void onCreate(Bundle savedInstanceState) {
           super.onCreate(savedInstanceState);
           setContentView(R.layout.activity_main);

           mapControl = findViewById(R.id.mapcontrol);

           mapControl.onCreate(savedInstanceState);

           //Wait until the map resources are ready.
           mapControl.getMapAsync(map -> {

               map.setTraffic(flow(TrafficFlow.RELATIVE));
               map.setTraffic(incidents(true));

               map.events.add((OnFeatureClick) (features) -> {

                   if (features != null && features.size() > 0) {
                       Feature incident = features.get(0);
                       if (incident.properties() != null) {


                           StringBuilder sb = new StringBuilder();
                           String incidentType = incident.getStringProperty("incidentType");
                           if (incidentType != null) {
                               sb.append(incidentType);
                           }
                           if (sb.length() > 0) sb.append("\n");
                           if ("Road Closed".equals(incidentType)) {
                               sb.append(incident.getStringProperty("from"));
                           } else {
                               String description = incident.getStringProperty("description");
                               if (description != null) {
                                   for (String word : description.split(" ")) {
                                       if (word.length() > 0) {
                                           sb.append(word.substring(0, 1).toUpperCase());
                                           if (word.length() > 1) {
                                               sb.append(word.substring(1));
                                           }
                                           sb.append(" ");
                                       }
                                   }
                               }
                           }
                           String message = sb.toString();

                           if (message.length() > 0) {
                               Toast.makeText(this,message,Toast.LENGTH_LONG).show();
                           }
                       }
                   }
               });
           });
       }

       @Override
       public void onResume() {
           super.onResume();
           mapControl.onResume();
       }

       @Override
       protected void onStart(){
           super.onStart();
           mapControl.onStart();
       }

       @Override
       public void onPause() {
           super.onPause();
           mapControl.onPause();
       }

       @Override
       public void onStop() {
           super.onStop();
           mapControl.onStop();
       }

       @Override
       public void onLowMemory() {
           super.onLowMemory();
           mapControl.onLowMemory();
       }

       @Override
       protected void onDestroy() {
           super.onDestroy();
           mapControl.onDestroy();
       }

       @Override
       protected void onSaveInstanceState(Bundle outState) {
           super.onSaveInstanceState(outState);
           mapControl.onSaveInstanceState(outState);
       }
   }
   ```

3. Une fois que vous avez incorporé le code ci-dessus dans votre application, vous pouvez cliquer sur une fonctionnalité pour voir les détails des incidents de trafic. Selon les valeurs de latitude, de longitude et de niveau de zoom que vous avez utilisées dans votre fichier **activity_main.xml**, vous obtenez des résultats similaires à l’image suivante :


    ![Incidents-trafic-sur-la-carte](./media/how-to-show-traffic-android/android-traffic.png)


## <a name="next-steps"></a>Étapes suivantes

Consultez les guides suivants pour savoir comment ajouter des données supplémentaires à votre carte :

> [!div class="nextstepaction"]
> [Ajouter une couche de symboles](how-to-add-symbol-to-android-map.md)

> [!div class="nextstepaction"]
> [Ajouter une couche de mosaïques](how-to-add-tile-layer-android-map.md)

> [!div class="nextstepaction"]
> [Ajouter des formes à une carte Android](how-to-add-shapes-to-android-map.md)

> [!div class="nextstepaction"]
> [Afficher des informations sur les caractéristiques](display-feature-information-android.md)