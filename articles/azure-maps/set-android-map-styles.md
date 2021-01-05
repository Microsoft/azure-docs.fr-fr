---
title: Définir un style de carte à l’aide d'Android SDK Azure Maps
description: Découvrez deux façons de définir le style d’une carte. Découvrez comment utiliser le SDK Microsoft Azure Maps Android dans le fichier layout ou la classe d’activité pour ajuster le style.
author: anastasia-ms
ms.author: v-stharr
ms.date: 11/18/2020
ms.topic: how-to
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.openlocfilehash: 8c7689fb87575ac6e150f793b43f35e8bf6adc83
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96532473"
---
# <a name="set-map-style-using-azure-maps-android-sdk"></a>Définir le style de carte à l’aide d'Android SDK Azure Maps

Cet article vous explique comment définir des styles de carte à l’aide d’Android SDK Azure Maps. Azure Maps offre le choix entre six styles de carte. Pour plus d’informations sur les styles de carte pris en charge, consultez [Styles de carte pris en charge dans Azure Maps](./supported-map-styles.md).

## <a name="prerequisites"></a>Conditions préalables requises

1. [Créer un compte Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account)
2. [Obtenir une clé d’abonnement principale](quick-demo-map-app.md#get-the-primary-key-for-your-account), également appelée clé primaire ou clé d’abonnement.
3. Télécharger et installer le [SDK Azure Maps Android](./how-to-use-android-map-control-library.md).


## <a name="set-map-style-in-the-layout"></a>Définir le style de carte dans la disposition

Vous pouvez définir un style de carte dans le fichier de disposition de votre classe d’activité. Modifiez l'élément `res > layout > activity_main.xml` de sorte qu'il se présente comme suit :

```XML
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
        app:mapcontrol_centerLat="47.602806"
        app:mapcontrol_centerLng="-122.329330"
        app:mapcontrol_zoom="12"
        app:mapcontrol_style="grayscale_dark"
        />

</FrameLayout>
```

L'attribut `mapcontrol_style` ci-dessus définit le style de carte sur **grayscale_dark**.

:::image type="content" source="./media/set-android-map-styles/grayscale-dark.png" border="true" alt-text="Azure Maps : image du mappage présentant le style en tant que grayscale_dark":::

## <a name="set-map-style-in-the-mainactivity-class"></a>Définir le style de carte dans la classe MainActivity

Le style de mappage peut également être défini dans la classe MainActivity. Ouvrez le fichier `java > com.example.myapplication > MainActivity.java`, puis copiez l’extrait de code suivant dans la méthode **onCreate()** . Ce code définit le style de carte sur **satellite_road_labels**.

>[!WARNING]
>Android Studio n’a peut-être pas importé les classes requises.  Par conséquent, le code aura des références insolubles. Pour importer les classes requises, pointez simplement sur chaque référence non résolue et appuyez sur`Alt + Enter` (Option + Retour sur un Mac).

```Java
mapControl.onReady(map -> {

    //Set the camera of the map.
    map.setCamera(center(47.64, -122.33), zoom(14));

    //Set the style of the map.
    map.setStyle((style(SATELLITE_ROAD_LABELS)));
       
});
```

:::image type="content" source="./media/set-android-map-styles/satellite-road-labels.png" border="true" alt-text="Azure Maps : image du mappage présentant le style en tant que satellite_road_labels":::