---
title: Ajouter un calque de mosaïques à une carte à l’aide de l’Android SDK Azure Maps
description: Découvrez comment ajouter une couche de mosaïques à une carte. Consultez un exemple qui utilise le SDK Microsoft Azure Maps Android pour ajouter un calque de radar météo à une carte.
author: anastasia-ms
ms.author: v-stharr
ms.date: 04/26/2019
ms.topic: how-to
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.openlocfilehash: 22618a28f1a87e68c19467aedf639e96ec2fb91e
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96532674"
---
# <a name="add-a-tile-layer-to-a-map-using-the-azure-maps-android-sdk"></a>Ajouter un calque de mosaïques à une carte à l’aide de l’Android SDK Azure Maps

Cet article explique comment afficher un calque de mosaïques sur une carte à l’aide de l’Android SDK Azure Maps. Les couches de mosaïques vous permettent de superposer des images à des mosaïques de carte de base Azure Maps. Vous trouverez plus d’informations sur le système de mosaïques Azure Maps dans la documentation [Niveaux de zoom et grille mosaïque](zoom-levels-and-tile-grid.md).

Un calque de mosaïques charge des mosaïques à partir d’un serveur. Ces images peuvent être prérendues et stockées comme toute autre image sur un serveur à l’aide d’une convention de nommage compréhensible par la couche de mosaïques. Ou ces images peuvent être rendues avec un service dynamique qui génère les images en quasi temps réel. Trois conventions de nommage de service de mosaïques sont prises en charge par la classe TileLayer d’Azure Maps :

* Notation de zoom X, Y : selon le niveau de zoom, x correspond à la position de colonne et y à la position de ligne de la mosaïque dans la grille mosaïque.
* Notation Quadkey : combinaison d’informations de zoom x, y au sein d’une valeur de chaîne qui correspond à l’identificateur unique de la mosaïque.
* Rectangle englobant : les coordonnées du rectangle englobant peuvent servir à spécifier une image au format `{west},{south},{east},{north}`, qui est couramment utilisé par [Web Mapping Services (WMS)](https://www.opengeospatial.org/standards/wms).

> [!TIP]
> Un calque de mosaïques est un excellent moyen de visualiser des jeux de données volumineux sur une carte. Non seulement elle peut être générée à partir d’une image, mais les données vectorielles peuvent également être affichées sous la forme d’une couche de mosaïques. En affichant des données vectorielles sous forme de couche de mosaïques, le contrôle de carte doit uniquement charger les mosaïques, qui peuvent être beaucoup plus petites que les données vectorielles qu’elles représentent. Cette technique est couramment utilisée pour afficher des millions de lignes de données sur une carte.

L’URL de la mosaïque passée à une couche de mosaïques doit être l’URL HTTP/HTTPS d’une ressource TileJSON ou d’un modèle d’URL de mosaïque qui utilise les paramètres suivants : 

* `{x}` - position X de la mosaïque. Nécessite également `{y}` et `{z}`.
* `{y}` - position Y de la mosaïque. Nécessite également `{x}` et `{z}`.
* `{z}` - niveau de zoom de la mosaïque. Nécessite également `{x}` et `{y}`.
* `{quadkey}` - identificateur quadkey de la mosaïque basé sur la convention de nommage du système de mosaïques de Bing Maps.
* `{bbox-epsg-3857}` - chaîne de rectangle englobant au format `{west},{south},{east},{north}` du système SRID EPSG 3857.
* `{subdomain}` : espace réservé pour les valeurs de sous-domaine, si la valeur de sous-domaine est spécifiée.

## <a name="prerequisites"></a>Conditions préalables requises

Pour suivre la procédure décrite dans cet article, vous devez installer [Android SDK Azure Maps](./how-to-use-android-map-control-library.md) afin de charger une carte.


## <a name="add-a-tile-layer-to-the-map"></a>Ajouter un calque de mosaïques à une carte

 Cet exemple montre comment créer une couche de mosaïques qui pointe vers un ensemble de mosaïques. Ces mosaïques utilisent le système de mosaïques « x, y, zoom ». La source de cette couche de mosaïques est un calque de radar météo issu de [Iowa Environmental Mesonet of Iowa State University](https://mesonet.agron.iastate.edu/ogc/). 

Vous pouvez ajouter un calque de mosaïques à la carte en suivant les étapes ci-dessous.

1. Modifiez l'élément **res > layout > activity_main.xml** de sorte qu'il se présente comme suit :

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
            app:mapcontrol_centerLat="40.75"
            app:mapcontrol_centerLng="-99.47"
            app:mapcontrol_zoom="3"
            />
    
    </FrameLayout>
    ```

2. Copiez l’extrait de code suivant dans la méthode **onCreate()** de votre classe `MainActivity.java`.

    ```Java
    mapControl.onReady(map -> {
        //Add a tile layer to the map, below the map labels.
        map.layers.add(new TileLayer(
            tileUrl("https://mesonet.agron.iastate.edu/cache/tile.py/1.0.0/nexrad-n0q-900913/{z}/{x}/{y}.png"),
            opacity(0.8f),
            tileSize(256)
        ), "labels");
    });
    ```
    
    L’extrait de code ci-dessus commence par obtenir une instance de contrôle de carte Azure Maps en utilisant la méthode de rappel **onReady()** . Il crée ensuite un objet `TileLayer` et passe une URL de mosaïque **xyz** mise en forme dans l’option `tileUrl`. L’opacité du calque est définie sur `0.8` et, les mosaïques du service de mosaïque utilisé étant des mosaïques de 256 pixels, ces informations sont transmises dans l’option `tileSize`. Le calque de mosaïques est ensuite transmis au gestionnaire de calques Maps.

    Après avoir ajouté l’extrait de code ci-dessus, votre `MainActivity.java` devrait ressembler à celui ci-dessous :
    
    ```Java
    package com.example.myapplication;

    import android.app.Activity;
    import android.os.Bundle;
    import android.support.v7.app.AppCompatActivity;
    import com.microsoft.azure.maps.mapcontrol.layer.TileLayer;
    import java.util.Arrays;
    import java.util.List;
    import com.microsoft.azure.maps.mapcontrol.AzureMaps;
    import com.microsoft.azure.maps.mapcontrol.MapControl;
    import static com.microsoft.azure.maps.mapcontrol.options.TileLayerOptions.tileSize;
    import static com.microsoft.azure.maps.mapcontrol.options.TileLayerOptions.tileUrl;
        
    public class MainActivity extends AppCompatActivity {
    
        static{
            AzureMaps.setSubscriptionKey("<Your Azure Maps subscription key>");
        }
    
        MapControl mapControl;
        @Override
        protected void onCreate(Bundle savedInstanceState) {
    
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
    
            mapControl = findViewById(R.id.mapcontrol);
    
            mapControl.onCreate(savedInstanceState);
    
            mapControl.onReady(map -> {

                //Add a tile layer to the map, below the map labels.
                map.layers.add(new TileLayer(
                    tileUrl("https://mesonet.agron.iastate.edu/cache/tile.py/1.0.0/nexrad-n0q-900913/{z}/{x}/{y}.png"),
                    opacity(0.8f),
                    tileSize(256)
                ), "labels");
            });    
        }
    
        @Override
        public void onResume() {
            super.onResume();
            mapControl.onResume();
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

À ce stade, si vous exécutez votre application, une ligne s'affiche sur la carte comme illustré ci-dessous :

<center>

![Ligne de carte Android](./media/how-to-add-tile-layer-android-map/xyz-tile-layer-android.png)</center>

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les différentes manières de définir des styles de carte, consultez l'article

> [!div class="nextstepaction"]
> [Change map styles in Android maps](./set-android-map-styles.md) (Modifier les styles de carte dans les cartes Android)