---
title: Tutoriel – Migrer une application Android |Microsoft Azure Maps
description: Tutoriel sur la façon de migrer une application Android de Google Maps vers Microsoft Azure Maps
author: rbrundritt
ms.author: richbrun
ms.date: 08/19/2020
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: cpendle
ms.custom: ''
ms.openlocfilehash: b096b24acd5cf65f6ad3e9eabb1d536b3aae0168
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96187066"
---
# <a name="tutorial---migrate-an-android-app-from-google-maps"></a>Tutoriel – Migrer une application Android à partir de Google Maps

Le kit Android SDK Azure Maps dispose d’une interface d’API qui est similaire au SDK web. Si vous avez effectué le développement avec l’un de ces SDK, un grand nombre des mêmes concepts, bonnes pratiques et architectures s’appliquent. Dans ce didacticiel, vous apprendrez à :

> [!div class="checklist"]
> * Charger une carte
> * Localiser une carte
> * Ajouter des marqueurs, des polylignes et des polygones
> * Superposer une couche de mosaïques
> * Afficher les données du trafic

Android SDK Azure Maps prend en charge une version Android minimale de l’API 21 : Android 5.0.0 (Lollipop).

Tous les exemples sont fournis en Java ; toutefois, vous pouvez également utiliser Kotlin avec le kit Android SDK Azure Maps.

Pour plus d’informations sur le développement avec le kit Android SDK fourni par Azure Maps, consultez les [guides pratiques relatifs au kit Android SDK Azure Maps](how-to-use-android-map-control-library.md).

## <a name="prerequisites"></a>Prérequis 

1. Connectez-vous au [portail Azure](https://portal.azure.com). Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.
2. [Créer un compte Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account)
3. [Obtenir une clé d’abonnement principale](quick-demo-map-app.md#get-the-primary-key-for-your-account), également appelée clé primaire ou clé d’abonnement. Pour plus d’informations sur l’authentification dans Azure Maps, voir [Gérer l’authentification dans Azure Maps](how-to-manage-authentication.md).

## <a name="load-a-map"></a>Charger une carte

Le chargement d’une carte dans une application Android à l’aide de Google ou d’Azure Maps nécessite des étapes similaires. Quand vous utilisez l’un ou l’autre des SDK, vous devez :

* Vous procurer une clé API ou d’abonnement pour accéder à l’une ou l’autre plateforme.
* Ajouter du code XML à une activité pour spécifier où la carte doit être affichée et la façon dont elle doit être disposée.
* Substituer toutes les méthodes de cycle de vie de l’activité contenant la vue cartographique aux méthodes correspondantes dans la classe map. En particulier, vous devez substituer les méthodes suivantes :
    * `onCreate(Bundle)`
    * `onStart()`
    * `onResume()`
    * `onPause()`
    * `onStop()`
    * `onDestroy()`
    * `onSaveInstanceState(Bundle)`
    * `onLowMemory()`
* Attendre que la carte soit prête avant d’essayer d’y accéder et de la programmer.

### <a name="before-google-maps"></a>Avant : Google Maps

Pour afficher une carte à l’aide du SDK Google Maps pour Android, il est nécessaire d’effectuer les étapes suivantes :

1. Assurez-vous que les services Google Play sont installés.
2. Ajoutez une dépendance pour le service Google Maps au fichier **gradle.build** du module :

    `implementation 'com.google.android.gms:play-services-maps:17.0.0'`

3. Ajoutez une clé API Google Maps à l’intérieur de la section application du fichier **google\_maps\_api.xml** :

    ```xml
    <meta-data android:name="com.google.android.geo.API_KEY" android:value="YOUR_GOOGLE_MAPS_KEY"/>
    ```

4. Ajoutez un fragment de carte à l’activité principale :

    ```xml
    <com.google.android.gms.maps.MapView
            android:id="@+id/myMap"
            android:layout_width="match_parent"
            android:layout_height="match_parent"/>
    ```

5. Dans le fichier **MainActivity.java**, vous devez importer le SDK Google Maps. Transférez toutes les méthodes de cycle de vie de l’activité contenant la vue cartographique aux éléments correspondants dans la classe map. Récupérez une instance de `MapView` à partir du fragment de carte à l’aide de la méthode `getMapAsync(OnMapReadyCallback)`. `MapView` initialise automatiquement le système maps et l’affichage. Modifiez le fichier **MainActivity.java** comme suit :

    ```java
    import com.google.android.gms.maps.GoogleMap;
    import com.google.android.gms.maps.MapView;
    import com.google.android.gms.maps.OnMapReadyCallback;
 
    import android.support.v7.app.AppCompatActivity;
    import android.os.Bundle;
    
    public class MainActivity extends AppCompatActivity implements OnMapReadyCallback {
    
        MapView mapView;
    
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
    
            mapView = findViewById(R.id.myMap);
    
            mapView.onCreate(savedInstanceState);
            mapView.getMapAsync(this);
        }
    
        @Override
    
        public void onMapReady(GoogleMap map) {
            //Add your post map load code here.
        }
    
        @Override
        public void onResume() {
            super.onResume();
            mapView.onResume();
        }
    
        @Override
        protected void onStart(){
            super.onStart();
            mapView.onStart();
        }
    
        @Override
        public void onPause() {
            super.onPause();
            mapView.onPause();
        }
    
        @Override
        public void onStop() {
            super.onStop();
            mapView.onStop();
        }
    
        @Override
        public void onLowMemory() {
            super.onLowMemory();
            mapView.onLowMemory();
        }
    
        @Override
        protected void onDestroy() {
            super.onDestroy();
            mapView.onDestroy();
        }
    
        @Override
        protected void onSaveInstanceState(Bundle outState) {
            super.onSaveInstanceState(outState);
            mapView.onSaveInstanceState(outState);
        }
    }
    ```

Quand vous exécutez une application, le contrôle de carte se charge comme dans l’image suivante.

![Google Maps simple](media/migrate-google-maps-android-app/simple-google-maps.png)

### <a name="after-azure-maps"></a>Après : Azure Maps

Pour afficher une carte à l’aide du SDK Azure Maps pour Android, il est nécessaire d’effectuer les étapes suivantes :

1. Ouvrez le fichier **build.gradle** de niveau supérieur et ajoutez le code suivant à la section de bloc **all projects** (tous les projets) :

    ```JAVA
    maven {
            url "https://atlas.microsoft.com/sdk/android"
    }
    ```

2. Modifiez l’élément **app/build.gradle** et ajoutez-y le code suivant :

    1. Assurez-vous que la version **minSdkVersion** de votre projet est l’API 21 ou une version ultérieure.

    2. Ajoutez le code suivant à la section Android :

        ```java
        compileOptions {
            sourceCompatibility JavaVersion.VERSION_1_8
            targetCompatibility JavaVersion.VERSION_1_8
        }
        ```

    3. Mettez à jour votre bloc de dépendances. Ajoutez une nouvelle ligne de dépendance d’implémentation pour la dernière version d’Android SDK Azure Maps :

        ```java
        implementation "com.microsoft.azure.maps:mapcontrol:0.2"
        ```

        > [!Note]
        > Android SDK Azure Maps est régulièrement mis à niveau et amélioré. Vous pouvez consulter l’article [Bien démarrer avec le contrôle de carte Android](how-to-use-android-map-control-library.md) pour obtenir le dernier numéro de version d’Azure Maps. Vous pouvez également remplacer le numéro de version « 0.2 » par « 0+ » afin que votre code pointe toujours vers la dernière version.

    4. Accédez à **Fichier** dans la barre d’outils, puis cliquez sur **Synchroniser le projet avec les fichiers Gradle**.

3. Ajoutez un fragment de carte à l’activité principale (resources pwd\> layout \> activity\_main.xml) :

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
            />
    </FrameLayout>
    ```

4. Dans le fichier **MainActivity.java**, vous devez :

    * Importer le SDK Azure Maps.
    * Définir vos informations d’authentification Azure Maps.
    * Obtenir l’instance de contrôle de carte dans la méthode **onCreate**.

     Définissez les informations d’authentification dans la classe `AzureMaps` à l’aide de la méthode `setSubscriptionKey` ou `setAadProperties`. S’agissant d’une mise à jour globale, veillez à ajouter vos informations d’authentification à chaque vue.

    Le contrôle de carte contient ses propres méthodes de cycle de vie pour la gestion du cycle de vie OpenGL d’Android. Ces méthodes doivent être appelées directement à partir de l’activité contenante. Pour appeler correctement les méthodes de cycle de vie du contrôle de carte, vous devez substituer les méthodes de cycle de vie suivantes dans l’activité qui contient le contrôle de carte. Appelez la méthode de contrôle de carte correspondante.

    * `onCreate(Bundle)` 
    * `onStart()` 
    * `onResume()` 
    * `onPause()` 
    * `onStop()` 
    * `onDestroy()` 
    * `onSaveInstanceState(Bundle)` 
    * `onLowMemory()`

    Modifiez le fichier **MainActivity.java** comme suit :

    ```java
    package com.example.myapplication;

    import android.support.v7.app.AppCompatActivity;
    import android.os.Bundle;
    import com.microsoft.azure.maps.mapcontrol.AzureMaps;
    import com.microsoft.azure.maps.mapcontrol.MapControl;
    import com.microsoft.azure.maps.mapcontrol.layer.SymbolLayer;
    import com.microsoft.azure.maps.mapcontrol.options.MapStyle;
    import com.microsoft.azure.maps.mapcontrol.source.DataSource;

    public class MainActivity extends AppCompatActivity {
     
        static {
            AzureMaps.setSubscriptionKey("<Your Azure Maps subscription key>");
        }

        MapControl mapControl;

        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);

            mapControl = findViewById(R.id.mapcontrol);

            mapControl.onCreate(savedInstanceState);
    
            //Wait until the map resources are ready.
            mapControl.onReady(map -> {
                //Add your post map load code here.
    
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

Si vous exécutez votre application, le contrôle de carte se charge comme dans l’image suivante.


![Azure Maps simple](media/migrate-google-maps-android-app/simple-azure-maps.png)

Notez que le contrôle Azure Maps prend davantage en charge le zoom arrière et qu’il fournit un affichage du monde plus étendu.

> [!TIP]
> Si vous utilisez un émulateur Android sur une machine Windows, la carte peut ne pas s’afficher en raison de conflits avec OpenGL et le rendu graphique accéléré par logiciel. Voici comment résoudre éventuellement ce problème. Ouvrez le gestionnaire AVD et sélectionnez l’appareil virtuel à modifier. Faites défiler le panneau **Vérifier la configuration**. Dans la section **Emulated Performance** (Performances émulées), définissez l’option **Graphics** sur **Hardware** (Matériel).

## <a name="localizing-the-map"></a>Localisation de la carte

La localisation est importante si votre audience est répartie dans plusieurs pays/régions ou parle différentes langues.

### <a name="before-google-maps"></a>Avant : Google Maps

Ajoutez le code suivant à la méthode `onCreate` pour définir la langue de la carte. Le code doit être ajouté avant de définir la vue contextuelle de la carte. Le code de langue « fr » limite la langue au français.

```java
String languageToLoad = "fr";
Locale locale = new Locale(languageToLoad);
Locale.setDefault(locale);
Configuration config = new Configuration();
config.locale = locale;
getBaseContext().getResources().updateConfiguration(config,
        getBaseContext().getResources().getDisplayMetrics());
```

Voici un exemple de Google Maps avec la langue définie sur « fr ».

![Localisation de Google Maps](media/migrate-google-maps-android-app/google-maps-localization.png)

### <a name="after-azure-maps"></a>Après : Azure Maps

Azure Maps propose trois méthodes pour définir la langue et la vue locale de la carte. La première option consiste à transmettre les informations de la langue et de la vue locale à la classe `AzureMaps`. Cette option utilise les méthodes statiques `setLanguage` et `setView` globalement. Cela signifie que la langue et la vue locale par défaut sont définis pour tous les contrôles Azure Maps chargés dans votre application. Cet exemple spécifie le français à l’aide du code de langue « fr-FR ».

```java
static {
    //Set your Azure Maps Key.
    AzureMaps.setSubscriptionKey("<Your Azure Maps Key>");

    //Set the language to be used by Azure Maps.
    AzureMaps.setLanguage("fr-FR");

    //Set the regional view to be used by Azure Maps.
    AzureMaps.setView("auto");
}
```

La deuxième option consiste à transmettre les informations de la langue et de la vue locale dans le code XML du contrôle de carte.

```xml
<com.microsoft.azure.maps.mapcontrol.MapControl
    android:id="@+id/myMap"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:mapcontrol_language="fr-FR"
    app:mapcontrol_view="auto"
    />
```

La troisième option consiste à programmer la langue et la vue locale de la carte avec la méthode Maps `setStyle`. Cette option met à jour la langue et la vue locale chaque fois que le code s’exécute.

```java
mapControl.onReady(map -> {
    map.setStyle(StyleOptions.language("fr-FR"));
    map.setStyle(StyleOptions.view("auto"));
});
```

Voici un exemple d’Azure Maps avec la langue définie sur « fr-FR ».

![Localisation de Google Maps](media/migrate-google-maps-android-app/azure-maps-localization.png)

Consultez la liste complète des [langues prises en charge](supported-languages.md).

## <a name="setting-the-map-view"></a>Définition de la vue cartographique

Vous pouvez déplacer programmatiquement les cartes dynamiques dans Azure Maps et Google Maps vers de nouveaux endroits en appelant les méthodes appropriées. Nous allons maintenant faire en sorte que la carte affiche une image aérienne satellite, centrer la carte sur une position avec des coordonnées, et changer le niveau de zoom. Pour cet exemple, nous allons utiliser latitude : 35.0272, longitude : -111.0225, et le niveau de zoom 15.

### <a name="before-google-maps"></a>Avant : Google Maps

La caméra du contrôle de carte Google Maps peut être déplacée programmatiquement avec la méthode `moveCamera`. La méthode `moveCamera` vous permet de spécifier le centre de la carte et un niveau de zoom. La méthode `setMapType` modifie le type de carte à afficher.

```java
@Override
public void onMapReady(GoogleMap googleMap) {
    mapView = googleMap;

    mapView.moveCamera(CameraUpdateFactory.newLatLngZoom(new LatLng(35.0272, -111.0225), 15));
    mapView.setMapType(GoogleMap.MAP_TYPE_SATELLITE);
}
```

![Vue définie de Google Maps](media/migrate-google-maps-android-app/google-maps-set-view.png)

> [!NOTE]
> Google Maps utilise des mosaïques de 256 pixels, tandis qu’Azure Maps utilise une mosaïque de 512 pixels. Cela réduit le nombre de demandes réseau nécessaires à Azure Maps pour charger la même zone réactive que Google Maps. Pour obtenir cette même zone visualisable en tant que carte dans Google Maps, vous devez soustraire le niveau de zoom utilisé dans Google Maps d’une unité lors de l’utilisation d’Azure Maps.

### <a name="after-azure-maps"></a>Après : Azure Maps

Comme indiqué précédemment, pour obtenir la même zone visualisable dans Azure Maps, diminuez le niveau de zoom utilisé dans Google Maps d’une unité. En l’occurrence, utilisez un niveau de zoom de 14.

La vue cartographique initiale peut être définie dans des attributs XML sur le contrôle de carte.

```xml
<com.microsoft.azure.maps.mapcontrol.MapControl
    android:id="@+id/myMap"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:mapcontrol_cameraLat="35.0272"
    app:mapcontrol_cameraLng="-111.0225"
    app:mapcontrol_zoom="14"
    app:mapcontrol_style="satellite"
    />
```

La vue cartographique peut être programmée à l’aide des méthodes Maps `setCamera` et `setStyle`.

```java
mapControl.onReady(map -> {
    //Set the camera of the map.
    map.setCamera(center(35.0272, -111.0225), zoom(14));

    //Set the style of the map.
    map.setStyle(style(MapStyle.SATELLITE));
});
```

![Vue définie d’Azure Maps](media/migrate-google-maps-android-app/azure-maps-set-view.png)

**Ressources supplémentaires :**

* [Styles de carte pris en charge](supported-map-styles.md)

## <a name="adding-a-marker"></a>Ajout d’un marqueur

Les données de points sont souvent affichées sur la carte sous forme d’image. Ces images sont appelées marqueurs, punaises, épingles ou symboles. Les exemples suivants affichent des données de points sous forme de marqueurs sur la carte à la position latitude : 51.5, longitude :  -0.2.

### <a name="before-google-maps"></a>Avant : Google Maps

Avec Google Maps, les marqueurs sont ajoutés à l’aide de la méthode maps `addMarker`.

```java
@Override
public void onMapReady(GoogleMap googleMap) {
    mapView = googleMap;

    mapView.addMarker(new MarkerOptions().position(new LatLng(47.64, -122.33)));
}
```

![Marqueur Google Maps](media/migrate-google-maps-android-app/google-maps-marker.png)

### <a name="after-azure-maps"></a>Après : Azure Maps

Dans Azure Maps, affichez les données de points sur la carte en les ajoutant d’abord à une source de données, puis en connectant cette source de données à un calque de symboles. La source de données optimise la gestion des données spatiales dans le contrôle de carte. Le calque de symboles spécifie comment afficher les données de points à l’aide d’une image ou d’un texte.

```java
mapControl.onReady(map -> {
    //Create a data source and add it to the map.
    DataSource dataSource = new DataSource();
    map.sources.add(dataSource);

    //Create a point feature and add it to the data source.
    dataSource.add(Feature.fromGeometry(Point.fromLngLat(-122.33, 47.64)));

    //Create a symbol layer and add it to the map.
    map.layers.add(new SymbolLayer(dataSource));
});
```

![Marqueur Azure Maps](media/migrate-google-maps-android-app/azure-maps-marker.png)

## <a name="adding-a-custom-marker"></a>Ajout d’un marqueur personnalisé

Les images personnalisées peuvent être utilisées pour représenter des points sur une carte. La carte présentée dans les exemples ci-dessous utilise une image personnalisée pour afficher un point. Le point est à la latitude : 51.5 et à la longitude : -0.2. L’ancre décale la position du marqueur afin que le point de l’icône de punaise s’aligne sur la position correcte sur la carte.

<center>

![image de punaise jaune](media/migrate-google-maps-web-app/yellow-pushpin.png)<br/>
yellow-pushpin.png</center>

Dans les deux exemples, l’image ci-dessus est ajoutée au dossier drawable des ressources d’applications.

### <a name="before-google-maps"></a>Avant : Google Maps

Avec Google Maps, il est possible d’utiliser des images personnalisées pour les marqueurs. Chargez ces images en utilisant l’option `icon` du marqueur. Pour aligner le point de l’image sur la coordonnée, utilisez l’option `anchor`. L’ancre est relative aux dimensions de l’image. En l’occurrence, le point d’ancrage est de 0,2 unité de large et 1 unité de haut.

```java
@Override
public void onMapReady(GoogleMap googleMap) {
    mapView = googleMap;

    mapView.addMarker(new MarkerOptions().position(new LatLng(47.64, -122.33))
    .icon(BitmapDescriptorFactory.fromResource(R.drawable.yellow-pushpin))
    .anchor(0.2f, 1f));
}
```

![Marqueur personnalisé Google Maps](media/migrate-google-maps-android-app/google-maps-custom-marker.png)

### <a name="after-azure-maps"></a>Après : Azure Maps

Dans Azure Maps, les calques de symboles prennent en charge les images personnalisées, mais l’image doit d’abord être chargée dans les ressources de carte et recevoir un ID unique. Ensuite, le calque de symboles doit référencer cet ID. Décalez le symbole pour l’aligner sur le point correct sur l’image à l’aide de l’option `iconOffset`. Le décalage de l’icône est en pixels. Par défaut, le décalage est relatif au centre inférieur de l’image, mais cette valeur de décalage peut être ajustée à l’aide de l’option `iconAnchor`. Cet exemple affecte la valeur `"center"` à l’option `iconAnchor`. Il utilise un décalage d’icône pour déplacer l’image de cinq pixels vers la droite et de 15 pixels vers le haut, afin de l’aligner sur le point de l’image de la punaise.

```java
mapControl.onReady(map -> {
    //Create a data source and add it to the map.
    DataSource dataSource = new DataSource();
    map.sources.add(dataSource);

    //Create a point feature and add it to the data source.
    dataSource.add(Feature.fromGeometry(Point.fromLngLat(-122.33, 47.64)));

    //Load the custom image icon into the map resources.
    map.images.add("my-yellow-pin", R.drawable.yellow_pushpin);

    //Create a symbol that uses the custom image icon and add it to the map.
    map.layers.add(new SymbolLayer(dataSource,
        iconImage("my-yellow-pin"),
        iconAnchor(AnchorType.CENTER
        iconOffset(new Float[]{5f, -15f})));
});
```

![Marqueur personnalisé Azure Maps](media/migrate-google-maps-android-app/azure-maps-custom-marker.png)

## <a name="adding-a-polyline"></a>Ajout d’une polyligne

Les polylignes sont utilisées pour représenter une ligne ou un tracé sur la carte. Les exemples suivants montrent comment créer une polyligne en pointillés sur la carte.

### <a name="before-google-maps"></a>Avant : Google Maps

Avec Google Maps, affichez une polyligne à l’aide de la classe `PolylineOptions`. Ajoutez la polyligne à la carte à l’aide de la méthode `addPolyline`. Définissez la couleur du trait avec l’option `color`. Définissez la largeur du trait avec l’option `width`. Ajoutez un tableau de traits de soulignement avec l’option `pattern`.

```java
@Override
public void onMapReady(GoogleMap googleMap) {
    mapView = googleMap;

    //Create the options for the polyline.
    PolylineOptions lineOptions = new PolylineOptions()
            .add(new LatLng(46, -123))
            .add(new LatLng(49, -122))
            .add(new LatLng(46, -121))
            .color(Color.RED)
            .width(10f)
            .pattern(Arrays.<PatternItem>asList(
                    new Dash(30f), new Gap(30f)));

    //Add the polyline to the map.
    Polyline polyline = mapView.addPolyline(lineOptions);
}
```

![Polyligne Google Maps](media/migrate-google-maps-android-app/google-maps-polyline.png)

### <a name="after-azure-maps"></a>Après : Azure Maps

Dans Azure Maps, les polylignes sont appelées objets `LineString` ou `MultiLineString`. Ajoutez ces objets à une source de données et affichez-les à l’aide d’une couche de lignes. Définissez la largeur du trait à l’aide de l’option `strokeWidth`. Ajoutez un tableau de traits de soulignement à l’aide de l’option `strokeDashArray`.

La largeur du trait et les unités de « pixel » du tableau de traits de soulignement dans le SDK web Azure Maps sont identiques à celles du service Google Maps. Tous deux acceptent les mêmes valeurs et produisent les mêmes résultats.

```java
mapControl.onReady(map -> {
    //Create a data source and add it to the map.
    DataSource dataSource = new DataSource();
    map.sources.add(dataSource);

    //Create an array of points.
    List<Point> points = Arrays.asList(
        Point.fromLngLat(-123, 46),
        Point.fromLngLat(-122, 49),
        Point.fromLngLat(-121, 46));

    //Create a LineString feature and add it to the data source.
    dataSource.add(Feature.fromGeometry(LineString.fromLngLats(points)));

    //Create a line layer and add it to the map.
    map.layers.add(new LineLayer(dataSource,
        strokeColor("red"),
        strokeWidth(4f),
        strokeDashArray(new Float[]{3f, 3f})));
});
```

![Polyligne Azure Maps](media/migrate-google-maps-android-app/azure-maps-polyline.png)

## <a name="adding-a-polygon"></a>Ajout d’un polygone

Les polygones sont utilisés pour représenter une zone sur la carte. Les exemples suivants montrent comment créer un polygone. Ce polygone forme un triangle basé sur la coordonnée du centre de la carte.

### <a name="before-google-maps"></a>Avant : Google Maps

Avec Google Maps, affichez un polygone à l’aide de la classe `PolygonOptions`. Ajoutez le polygone à la carte à l’aide de la méthode `addPolygon`. Définissez les couleurs de remplissage et de trait respectivement à l’aide des options `fillColor` et `strokeColor`. Définissez la largeur du trait à l’aide de l’option `strokeWidth`.

```java
@Override
public void onMapReady(GoogleMap googleMap) {
    mapView = googleMap;

    //Create the options for the polygon.
    PolygonOptions polygonOptions = new PolygonOptions()
            .add(new LatLng(46, -123))
            .add(new LatLng(49, -122))
            .add(new LatLng(46, -121))
            .add(new LatLng(46, -123))  //Close the polygon.
            .fillColor(Color.argb(128, 0, 128, 0))
            .strokeColor(Color.RED)
            .strokeWidth(5f);

    //Add the polygon to the map.
    Polygon polygon = mapView.addPolygon(polygonOptions);
}
```

![Polygone Google Maps](media/migrate-google-maps-android-app/google-maps-polygon.png)

### <a name="after-azure-maps"></a>Après : Azure Maps

Dans Azure Maps, ajoutez des objets `Polygon` et `MultiPolygon` à une source de données, et affichez-les sur la carte à l’aide de couches. Affichez la zone d’un polygone dans une couche de polygones. Affichez le contour d’un polygone à l’aide d’une couche de lignes. Définissez la couleur et la largeur du trait à l’aide des options `strokeColor` et `strokeWidth`.

La largeur du trait et les unités de « pixel » du tableau de traits de soulignement dans le SDK web Azure Maps sont identiques à celles du service Google Maps. Tous deux acceptent les mêmes valeurs et produisent les mêmes résultats.

```java
mapControl.onReady(map -> {
    //Create a data source and add it to the map.
    DataSource dataSource = new DataSource();
    map.sources.add(dataSource);

    //Create an array of point arrays to create polygon rings.
    List<List<Point>> rings = Arrays.asList(Arrays.asList(
        Point.fromLngLat(-123, 46),
        Point.fromLngLat(-122, 49),
        Point.fromLngLat(-121, 46),
        Point.fromLngLat(-123, 46)));

    //Create a Polygon feature and add it to the data source.
    dataSource.add(Feature.fromGeometry(Polygon.fromLngLats(rings)));

    //Add a polygon layer for rendering the polygon area.
    map.layers.add(new PolygonLayer(dataSource,
        fillColor("green"),
        fillOpacity(0.5f)));

    //Add a line layer for rendering the polygon outline.
    map.layers.add(new LineLayer(dataSource,
        strokeColor("red"),
        strokeWidth(2f)));
});
```
![Polygone Azure Maps](media/migrate-google-maps-android-app/azure-maps-polygon.png)

## <a name="overlay-a-tile-layer"></a>Superposer une couche de mosaïques

 Utilisez des couches de mosaïques pour superposer des images qui ont été divisées en images en mosaïque plus petites qui s’alignent sur le système de mosaïque de cartes. Cette approche est un moyen courant de superposer des images de calque ou des jeux de données volumineux. Les couches de mosaïques sont appelés « superpositions d’images » dans Google Maps.

Les exemples suivants superposent une couche de mosaïques radar météo issue de l’Iowa Environmental Mesonet de l’Iowa State University. La taille des mosaïques est de 256 pixels.

### <a name="before-google-maps"></a>Avant : Google Maps

Avec Google Maps, une couche de mosaïques peut être superposée au-dessus de la carte. Utilise la classe `TileOverlayOptions`. Ajoutez la couche de mosaïques à la carte à l’aide de la méthode `addTileLauer`. Pour rendre les mosaïques semi-transparentes, l’option `transparency` est définie sur 0,2 (20 % de transparence).

```java
@Override
public void onMapReady(GoogleMap googleMap) {
    mapView = googleMap;

    //Create the options for the tile layer.
    TileOverlayOptions tileLayer = new TileOverlayOptions()
        .tileProvider(new UrlTileProvider(256, 256) {
            @Override
            public URL getTileUrl(int x, int y, int zoom) {

                try {
                    //Define the URL pattern for the tile images.
                    return new URL(String.format("https://mesonet.agron.iastate.edu/cache/tile.py/1.0.0/nexrad-n0q-900913/%d/%d/%d.png", zoom, x, y));
                }catch (MalformedURLException e) {
                    throw new AssertionError(e);
                }
            }
        }).transparency(0.2f);

    //Add the tile layer to the map.
    mapView.addTileOverlay(tileLayer);
}
```

![Couche de mosaïques Google Maps](media/migrate-google-maps-android-app/google-maps-tile-layer.png)

### <a name="after-azure-maps"></a>Après : Azure Maps

Une couche de mosaïques peut être ajoutée à la carte de la même façon que n’importe quel autre calque. Une URL mise en forme qui a les espaces réservés x, y et zoom (`{x}`, `{y}`, `{z}` respectivement) est utilisée pour indiquer à la couche où accéder aux mosaïques. De plus, les couches de mosaïques dans Azure Maps prennent en charge les espaces réservés `{quadkey}`, `{bbox-epsg-3857}` et `{subdomain}`. Pour rendre la couche de mosaïques semi-transparente, une valeur d’opacité de 0,8 est utilisée. L’opacité et la transparence, bien que similaires, utilisent des valeurs inverses. Pour effectuer une conversion entre les deux options, soustrayez leur valeur du nombre 1.

> [!TIP]
> Dans Azure Maps, il est commode de restituer des couches sous d’autres couches, y compris les couches de la carte de base. De plus, il est souvent souhaitable de restituer des couches de mosaïques sous les étiquettes de carte afin qu’elles soient faciles à lire. La méthode `map.layers.add` prend un deuxième paramètre, qui est l’ID du calque dans lequel insérer le nouveau calque ci-dessous. Pour insérer une couche de mosaïques sous les étiquettes de carte, vous pouvez utiliser le code suivant : `map.layers.add(myTileLayer, "labels");`

```java
mapControl.onReady(map -> {
    //Add a tile layer to the map, below the map labels.
    map.layers.add(new TileLayer(
        tileUrl("https://mesonet.agron.iastate.edu/cache/tile.py/1.0.0/nexrad-n0q-900913/{z}/{x}/{y}.png"),
        opacity(0.8f),
        tileSize(256)
    ), "labels");
});
```

![Couche de mosaïques Azure Maps](media/migrate-google-maps-android-app/azure-maps-tile-layer.png)

## <a name="show-traffic"></a>Afficher le trafic

Azure Maps et Google Maps proposent des options pour superposer les données de trafic.

### <a name="before-google-maps"></a>Avant : Google Maps

Avec Google Maps, vous pouvez superposer les données de flux de trafic sur la carte en passant la valeur true à la méthode `setTrafficEnabled` de la carte.

```java
@Override
public void onMapReady(GoogleMap googleMap) {
    mapView = googleMap;

    mapView.setTrafficEnabled(true);
}
```

![Trafic Google Maps](media/migrate-google-maps-android-app/google-maps-traffic.png)

### <a name="after-azure-maps"></a>Après : Azure Maps

Azure Maps offre plusieurs options différentes pour l’affichage du trafic. Les incidents de trafic, tels que les fermetures de route et les accidents, peuvent être affichés sous forme d’icônes sur la carte. Des routes codées en couleurs et le flux de trafic peuvent être superposés sur la carte. Les couleurs peuvent être modifiées pour tenir compte de la limite de vitesse indiquée, d’un retard prévu normal ou d’un retard absolu. Les données d’incident dans Azure Maps sont mises à jour toutes les minutes, et les données de circulation le sont toutes les deux minutes.

```java
mapControl.onReady(map -> {
    map.setTraffic(
        incidents(true),
        flow(TrafficFlow.RELATIVE));
});
```

![Trafic Azure Maps](media/migrate-google-maps-android-app/azure-maps-traffic.png)

## <a name="next-steps"></a>Étapes suivantes

Découvrez-en plus sur Android SDK Azure Maps :

> [!div class="nextstepaction"]
> [Guide pratique pour utiliser le contrôle de carte Android](how-to-use-android-map-control-library.md)

> [!div class="nextstepaction"]
> [Add a symbol layer to an Android map](how-to-add-symbol-to-android-map.md) (Ajouter une couche de symboles à une carte Android)

> [!div class="nextstepaction"]
> [Ajouter des formes à une carte Android](./how-to-add-shapes-to-android-map.md)

> [!div class="nextstepaction"]
> [Change map styles in Android maps](./set-android-map-styles.md) (Modifier les styles de carte dans les cartes Android)