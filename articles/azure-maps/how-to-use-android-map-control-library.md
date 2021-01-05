---
title: Bien démarrer avec Android SDK Azure Maps
description: Familiarisez-vous avec le SDK Microsoft Azure Maps Android. Apprenez à créer un projet dans Android Studio, à installer le Kit de développement logiciel (SDK) et à créer un mappage interactif.
author: anastasia-ms
ms.author: v-stharr
ms.date: 11/18/2020
ms.topic: how-to
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: mvc
ms.openlocfilehash: 1da003bb8d285dbedde87cbc6cd4708fda2dc38b
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96531254"
---
# <a name="getting-started-with-azure-maps-android-sdk"></a>Bien démarrer avec Android SDK Azure Maps

Android SDK Azure Maps est une bibliothèque de cartes de vecteur pour Android. Cet article vous guide tout au long des processus d’installation d’Android SDK Azure Maps et de chargement d’une carte.

## <a name="prerequisites"></a>Prérequis

### <a name="create-an-azure-maps-account"></a>Créer un compte Azure Maps

1. [Créer un compte Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account)
2. [Obtenir une clé d’abonnement principale](quick-demo-map-app.md#get-the-primary-key-for-your-account), également appelée clé primaire ou clé d’abonnement.
Pour plus d’informations sur l’authentification dans Azure Maps, voir [Gérer l’authentification dans Azure Maps](./how-to-manage-authentication.md).
3. [Télécharger et installer Android Studio de Google](https://developer.android.com/studio/).

## <a name="create-a-project-in-android-studio"></a>Créer un projet dans Android Studio

Suivez ces étapes pour créer un projet Android Studio :

1. Lancez Android Studio.
2. Cliquez sur **Créer un projet**.
3. Dans l’onglet **Téléphone et tablette**, sélectionnez **Activité vide**. Cliquez sur **Suivant**.
4. Sous **Configurer votre projet**, sélectionnez `API 21: Android 5.0.0 (Lollipop)` comme SDK minimum.
5. Sélectionnez la langue `Java`.
6. Acceptez la valeur `Name` par défaut pour le projet. Cliquez sur **Terminer**.

Pour obtenir de l’aide supplémentaire sur l’installation d’Android Studio et la création d’un projet, consultez la [documentation Android Studio](https://developer.android.com/studio/intro/).

![Créer un projet dans Android Studio ](./media/how-to-use-android-map-control-library/form-factor-android.png)

## <a name="set-up-a-device"></a>Configurer un appareil

Pour tester votre application pendant le développement, vous pouvez utiliser un téléphone Android ou un émulateur Android.

Pour en savoir plus sur la configuration d’un appareil virtuel Android ou AVD (Android Virtual Device), consultez la [documentation Android Studio](https://developer.android.com/studio/run/managing-avds).

## <a name="install-the-azure-maps-android-sdk"></a>Installer Android SDK Azure Maps

L’étape suivante de la création de votre application consiste à installer Android SDK Azure Maps.

Pour installer le Kit de développement logiciel (SDK), procédez comme suit :

1. Dans l’onglet Projet, développez **Scripts Gradle**. Ouvrez **build.gradle (projet : My_Application)** , puis ajoutez le code suivant à la section **tous les projets**, `repositories` :

    ```
    maven {
            url "https://atlas.microsoft.com/sdk/android"
    }
    ```

2. Ouvrez **build.gradle (module : My_Application)** .

3. Assurez-vous que le paramètre **minSdkVersion** de la section `defaultConfig` est défini sur API 21 ou supérieur.

4. Ajoutez le code suivant à la section Android :

    ```
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
    ```

5. Ajoutez le code suivant à la section `dependencies` :

    ```
    implementation "com.microsoft.azure.maps:mapcontrol:0.6"
    ```

6. Cliquez sur **Fichier** dans la barre d’outils principale, puis sélectionnez **Synchroniser le projet avec les fichiers Gradle**.

7. Ouvrez `res > layout > activity_main.xml`. Cliquez sur l’affichage `Code` dans le coin supérieur droit. Ajoutez le code XML suivant dans l’élément `<androidx.constraintlayout.widget.ConstraintLayout>`.
    
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
            />
    </FrameLayout>
    ```

8. Dans le fichier `java > com.example.myapplication > MainActivity.java`, vous devrez :

    * Ajouter des importations pour le Kit de développement logiciel (SDK) Azure Maps.
    * Définir vos informations d’authentification Azure Maps.
    * Obtenir l’instance de contrôle de carte dans la méthode **onCreate**.

    Pour éviter d’avoir à ajouter des informations d’authentification pour chaque affichage d’application, nous allons définir les informations d’authentification globalement en appelant `AzureMaps.setSubscriptionKey`. Vous pouvez également appeler `AzureMaps.setAadProperties` pour vous identifier à l’aide d’Azure Active Directory.

    Le contrôle de carte remplace les méthodes de cycle de vie suivantes de la classe MainActivity. Ces méthodes sont responsables de la gestion du cycle de vie OpenGL d’Android.

    * onCreate(Bundle)
    * onStart()
    * onResume()
    * onPause()
    * onStop()
    * onDestroy()
    * onSaveInstanceState(Bundle)
    * onLowMemory()

    Modifiez le fichier `MainActivity.java` comme suit :

    ```java
    package com.example.myapplication;

    //For older versions use: import android.support.v7.app.AppCompatActivity; 
    import androidx.appcompat.app.AppCompatActivity;
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

>[!WARNING]
>Android Studio n’a peut-être pas importé les classes requises.  Par conséquent, le code aura des références insolubles. Pour importer les classes requises, pointez simplement sur chaque référence non résolue et appuyez sur`Alt + Enter` (Option + Retour sur un Mac).

La création de l’application par Android Studio prend quelques secondes. Une fois la création terminée, vous pouvez tester votre application dans l’appareil Android émulé. La carte doit ressembler à ceci :

:::image type="content" source="./media/how-to-use-android-map-control-library/android-map.png" border="true" alt-text="Azure Maps dans une application Android":::

## <a name="localizing-the-map"></a>Localisation de la carte

Android SDK Azure Maps propose trois méthodes différentes pour définir la langue et les paramètres régionaux de la carte.

1. Définissez les paramètres linguistiques et régionaux en appelant des méthodes statiques sur la classe AzureMaps.

    ```Java
    static {
        //Set your Azure Maps Key.
        AzureMaps.setSubscriptionKey("<Your Azure Maps Key>");

        //Set the language to be used by Azure Maps.
        AzureMaps.setLanguage("fr-FR");

        //Set the regional view.
        AzureMaps.setLanguage("Auto");
    
    }
    ```

2. Définissez les paramètres linguistiques et régionaux dans le code XML de contrôle de carte.

    ```XML
    <com.microsoft.azure.maps.mapcontrol.MapControl
        android:id="@+id/myMap"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:mapcontrol_language="fr-FR"
        app:mapcontrol_view="Auto"
        />
    ```

3. Définissez les paramètres linguistiques et régionaux en appelant des méthodes sur le contrôle de carte. Cette option vous permet de modifier les paramètres lors de l’exécution.

    ```Java
    mapControl.onReady(map -> {
        map.setStyle(StyleOptions.language("fr-FR"));
        map.setStyle(StyleOptions.view("Auto"));
    
    });
    ```

Voici un exemple d’Azure Maps avec la langue définie sur `fr-FR`.

:::image type="content" source="./media/how-to-use-android-map-control-library/android-localization.png" border="true" alt-text="Azure Maps, image de carte affichant des étiquettes en français":::

La liste complète des langues et des affichages régionaux pris en charge est disponible [ici](supported-languages.md).

## <a name="navigating-the-map"></a>Navigation dans la carte

Il existe plusieurs façons d’effectuer des zooms, des panoramiques, des rotations et des tangages de la carte. Les informations suivantes détaillent les différentes façons de naviguer dans la carte.

**Faire un zoom sur la carte**

- Touchez la carte avec deux doigts et rapprochez-les pour faire un zoom arrière ou écartez-les pour faire un zoom avant.
- Appuyez deux fois sur la carte pour faire un zoom avant d’un niveau.
- Appuyez deux fois avec deux doigts pour faire un zoom arrière d’un niveau de la carte.
- Appuyez deux fois ; sur le second appui, maintenez votre doigt sur la carte et faites glisser vers le haut pour effectuer un zoom avant, ou vers le bas pour effectuer un zoom arrière.

**Effectuer un panoramique sur la carte**

- Touchez la carte et faites-la glisser dans n’importe quelle direction.

**Faire pivoter la carte**

- Touchez la carte avec deux doigts et effectuez une rotation.

**Incliner la carte**

- Touchez la carte avec deux doigts et faites-les glisser ensemble vers le haut ou vers le bas.

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment ajouter des données superposées sur la carte :

> [!div class="nextstepaction"]
> [Add a symbol layer to an Android map](how-to-add-symbol-to-android-map.md) (Ajouter une couche de symboles à une carte Android)

> [!div class="nextstepaction"]
> [Ajouter des formes à une carte Android](./how-to-add-shapes-to-android-map.md)

> [!div class="nextstepaction"]
> [Change map styles in Android maps](./set-android-map-styles.md) (Modifier les styles de carte dans les cartes Android)