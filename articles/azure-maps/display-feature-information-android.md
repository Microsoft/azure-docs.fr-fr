---
title: Afficher des informations sur les caractéristiques dans le kit Android SDK Azure Maps | Microsoft Azure Maps
description: Découvrez comment afficher des informations lorsque les utilisateurs interagissent avec les fonctionnalités de la carte. Utilisez Android SDK Azure Maps pour afficher des messages Toast et d’autres types de messages.
author: rbrundritt
ms.author: richbrun
ms.date: 08/08/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.openlocfilehash: fabb4cd1e555a7a67a53bf2f5a99d93c87df436c
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96532803"
---
# <a name="display-feature-information"></a>Afficher des informations sur les fonctionnalités

Les données spatiales sont souvent représentées à l’aide de points, de lignes et de polygones. Ces données sont souvent associées à des informations de métadonnées. Par exemple, un point peut représenter l’emplacement d’un magasin et les métadonnées relatives à ce restaurant peuvent être son nom, son adresse et le type de nourriture proposé. Ces métadonnées peuvent être ajoutées en tant que propriétés de ces caractéristiques à l’aide d’un `JsonObject`. Le code suivant crée une caractéristique de point simple avec une propriété `title` dont la valeur est « Hello World! ».

```java
//Create a data source and add it to the map.
DataSource dataSource = new DataSource();
map.sources.add(dataSource);

//Create a point feature with some properties and add it to the data source.
JsonObject properties = new JsonObject();
properties.addProperty("title", "Hello World!");

//Create a point feature, pass in the metadata properties, and add it to the data source.
dataSource.add(Feature.fromGeometry(Point.fromLngLat(-122.33, 47.64), properties));
```

Quand un utilisateur interagit avec une caractéristique sur la carte, des événements peuvent être utilisés pour réagir à ces actions. Un scénario courant consiste à afficher un message constitué des propriétés de métadonnées d’une caractéristique avec laquelle l’utilisateur a interagi. L’événement `OnFeatureClick` est l’événement principal utilisé pour détecter quand l’utilisateur a appuyé sur une caractéristique sur la carte. Il existe également un événement `OnLongFeatureClick`. Quand vous ajoutez l’événement `OnFeatureClick` à la carte, vous pouvez le limiter à un seul calque en transmettant l’ID d’un calque auquel le limiter. Si aucun ID de calque n’est transmis, le fait de cliquer sur n’importe quelle caractéristique de la carte, quelle que soit le calque dans lequel elle se trouve, déclenche cet événement. Le code suivant crée un calque de symboles pour afficher des données de point sur la carte, puis ajoute un événement `OnFeatureClick` et le limite à ce calque de symboles.

```java
//Create a symbol and add it to the map.
SymbolLayer symbolLayer = new SymbolLayer(dataSource);
map.layers.add(symbolLayer);

//Add a feature click event to the map.
map.events.add((OnFeatureClick) (features) -> {
    //Retrieve the title property of the feature as a string.
    String msg = features.get(0).properties().get("title").getAsString();

    //Do something with the message.
}, symbolLayer.getId());    //Limit this event to the symbol layer.
```

## <a name="display-a-toast-message"></a>Afficher un message Toast

Les messages Toast sont l’un des moyens les plus simples de présenter des informations à l’utilisateur. Ils sont disponibles dans toutes les versions d’Android. Ils ne prennent en charge aucun type d’entrée utilisateur et sont affichés uniquement pendant un court instant. Si vous souhaitez fournir rapidement une information à l’utilisateur concernant l’objet sur lequel il a appuyé, un message Toast peut être une bonne option. Le code suivant montre comment utiliser un message Toast avec l’événement `OnFeatureClick`.

```java
//Add a feature click event to the map.
map.events.add((OnFeatureClick) (features) -> {
    //Retrieve the title property of the feature as a string.
    String msg = features.get(0).getStringProperty("title");

    //Display a toast message with the title information.
    Toast.makeText(this, msg, Toast.LENGTH_SHORT).show();
}, symbolLayer.getId());    //Limit this event to the symbol layer.
```

![Animation d’une caractéristique faisant l’objet d’un appui et de l’affichage d’un message Toast](./media/display-feature-information-android/symbol-layer-click-toast-message.gif)

Outre les messages Toast, il existe de nombreuses autres façons de présenter les propriétés de métadonnées d’une caractéristique, par exemple :

- [Widget Snakbar](https://developer.android.com/training/snackbar/showing.html) : les snackbars fournissent des retours d’informations allégés concernant une opération. Ils affichent un bref message en bas de l’écran sur l’appareil mobile et en bas à gauche sur les appareils plus grands. Les snackbars s’affichent au-dessus de tous les autres éléments à l’écran, et un seul peut être affiché à la fois.
- [Boîtes de dialogue](https://developer.android.com/guide/topics/ui/dialogs) : une boîte de dialogue est une petite fenêtre qui invite l’utilisateur à prendre une décision ou à entrer des informations supplémentaires. Une boîte de dialogue ne remplit pas l’écran et est normalement utilisée pour les événements modaux qui exigent que les utilisateurs effectuent une action avant de pouvoir continuer.
- Ajouter un [fragment](https://developer.android.com/guide/components/fragments) à l’activité en cours.
- Accéder à une autre activité ou vue.

## <a name="next-steps"></a>Étapes suivantes

Pour ajouter des données à votre carte :

> [!div class="nextstepaction"]
> [Ajouter une couche de symboles](how-to-add-symbol-to-android-map.md)

> [!div class="nextstepaction"]
> [Ajouter des formes à une carte Android](how-to-add-shapes-to-android-map.md)
