---
title: 'Démarrage rapide : Ajouter l’appel VoIP à une application Android avec Azure Communication Services'
description: Dans ce tutoriel, vous allez apprendre à utiliser la bibliothèque de client Appel Azure Communication Services pour Android.
author: matthewrobertson
ms.author: marobert
ms.date: 08/11/2020
ms.topic: quickstart
ms.service: azure-communication-services
ms.openlocfilehash: 02cf175fc0a29795428ce1b3651469532ff3867c
ms.sourcegitcommit: 6906980890a8321dec78dd174e6a7eb5f5fcc029
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92438562"
---
Dans ce guide de démarrage rapide, vous allez découvrir comment démarrer un appel à l’aide de la bibliothèque de client Appel Azure Communication Services pour Android.

## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- [Android Studio](https://developer.android.com/studio), pour la création de votre application Android
- Une ressource Communication Services déployée. [Créer une ressource Communication Services](../../create-communication-resource.md)
- Un [jeton d’accès utilisateur](../../access-tokens.md) pour votre service Azure Communication

## <a name="setting-up"></a>Configuration

### <a name="create-an-android-app-with-an-empty-activity"></a>Créer une application Android avec une activité vide

Dans Android Studio, sélectionnez Start a new Android Studio project (Commencer un nouveau projet Android Studio).

:::image type="content" source="../media/android/studio-new-project.png" alt-text="Capture d’écran montrant le bouton « Start a new Android Studio Project » sélectionné dans Android Studio.":::

Sélectionnez le modèle de projet « Empty Activity » sous « Phone and Tablet ».

:::image type="content" source="../media/android/studio-blank-activity.png" alt-text="Capture d’écran montrant le bouton « Start a new Android Studio Project » sélectionné dans Android Studio.":::

Sélectionnez la bibliothèque de client minimale « API 26 : Android 8.0 (Oreo) » ou version ultérieure.

:::image type="content" source="../media/android/studio-calling-min-api.png" alt-text="Capture d’écran montrant le bouton « Start a new Android Studio Project » sélectionné dans Android Studio.":::


### <a name="install-the-package"></a>Installer le package

<!-- TODO: update with instructions on how to download, install and add package to project -->
Recherchez votre projet build.gradle et veillez à ajouter `mavenCentral()` à la liste des référentiels sous `buildscript` et `allprojects`.
```groovy
buildscript {
    repositories {
    ...
        mavenCentral()
    ...
    }
}
```

```groovy
allprojects {
    repositories {
    ...
        mavenCentral()
    ...
    }
}
```
Ensuite, dans votre niveau de module build.gradle, ajoutez les lignes suivantes aux sections dependencies et android.

```groovy
android {
    ...
    packagingOptions {
        pickFirst  'META-INF/*'
    }
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}

dependencies {
    ...
    implementation 'com.azure.android:azure-communication-calling:1.0.0-beta.2'
    ...
}
```

### <a name="add-permissions-to-application-manifest"></a>Ajouter des autorisations au manifeste de l’application

Afin de pouvoir demander les autorisations nécessaires pour effectuer un appel, vous devez d’abord les déclarer dans le manifeste de l’application (`app/src/main/AndroidManifest.xml`). Remplacez le contenu du fichier par ce qui suit :

```xml
    <?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.contoso.acsquickstart">

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.RECORD_AUDIO" />
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <!--Our calling client library depends on the Apache HTTP client library.
When targeting Android client library 28+, this library needs to be explicitly referenced.
See https://developer.android.com/about/versions/pie/android-9.0-changes-28#apache-p-->
        <uses-library android:name="org.apache.http.legacy" android:required="false"/>
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
    
```

### <a name="set-up-the-layout-for-the-app"></a>Configurer la disposition de l’application

Deux entrées sont nécessaires : une entrée de texte pour l’ID de l’appelé et un bouton pour établir l’appel. Vous pouvez les ajouter par le biais du concepteur ou en modifiant le fichier xml de disposition. Créez un bouton avec `call_button` comme ID et `callee_id` comme entrée de texte. Accédez à (`app/src/main/res/layout/activity_main.xml`) et remplacez le contenu du fichier par ce qui suit :

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <Button
        android:id="@+id/call_button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="16dp"
        android:text="Call"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent" />

    <EditText
        android:id="@+id/callee_id"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:ems="10"
        android:hint="Callee Id"
        android:inputType="textPersonName"
        app:layout_constraintBottom_toTopOf="@+id/call_button"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />
</androidx.constraintlayout.widget.ConstraintLayout>
```

### <a name="create-the-main-activity-scaffolding-and-bindings"></a>Créer la génération de modèles automatique et les liaisons de l’activité principale

La mise en page étant créée, nous pouvons ajouter les liaisons ainsi que la génération de modèles automatique de base de l’activité. L’activité gère les demandes d’autorisations au moment de l’exécution, la création de l’agent d’appel et l’établissement de l’appel en cas d’appui sur le bouton. Chacune de ces opérations est traitée dans sa propre section. La méthode `onCreate` est substituée pour appeler `getAllPermissions` et `createAgent`, ainsi que pour ajouter les liaisons pour le bouton d’appel. Cela ne se produit qu’une seule fois lors de la création de l’activité. Pour plus d’informations sur `onCreate`, consultez le guide intitulé [Understand the Activity Lifecycle](https://developer.android.com/guide/components/activities/activity-lifecycle) (Présentation du cycle de vie des activités).

Accédez à **MainActivity.java** et remplacez le contenu par le code suivant :

```java
package com.contoso.acsquickstart;

import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.ActivityCompat;

import android.Manifest;
import android.content.pm.PackageManager;
import android.os.Bundle;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Toast;

import com.azure.android.communication.common.CommunicationUser;
import com.azure.android.communication.common.CommunicationUserCredential;
import com.azure.communication.calling.CallAgent;
import com.azure.communication.calling.CallClient;
import com.azure.communication.calling.StartCallOptions;


import java.util.ArrayList;

public class MainActivity extends AppCompatActivity {
    
    private CallAgent callAgent;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        getAllPermissions();
        createAgent();
        
        // Bind call button to call `startCall`
        Button callButton = findViewById(R.id.call_button);
        callButton.setOnClickListener(l -> startCall());
    }

    /**
     * Request each required permission if the app doesn't already have it.
     */
    private void getAllPermissions() {
        // See section on requesting permissions
    }

    /**
      * Create the call agent for placing calls
      */
    private void createAgent() {
        // See section on creating the call agent
    }

    /**
     * Place a call to the callee id provided in `callee_id` text input.
     */
    private void startCall() {
        // See section on starting the call
    }
}

```

### <a name="request-permissions-at-runtime"></a>Demander des autorisations au moment de l’exécution

Pour Android 6.0 et ultérieur (niveau d’API 23) et `targetSdkVersion` 23 ou plus, les autorisations sont accordées au moment de l’exécution et non lors de l’installation de l’application. Pour prendre cela en charge, vous pouvez implémenter `getAllPermissions` afin d’appeler `ActivityCompat.checkSelfPermission` et `ActivityCompat.requestPermissions` pour chaque autorisation requise.

```java
/**
 * Request each required permission if the app doesn't already have it.
 */
private void getAllPermissions() {
    String[] requiredPermissions = new String[]{Manifest.permission.RECORD_AUDIO, Manifest.permission.CAMERA, Manifest.permission.WRITE_EXTERNAL_STORAGE, Manifest.permission.READ_PHONE_STATE};
    ArrayList<String> permissionsToAskFor = new ArrayList<>();
    for (String permission : requiredPermissions) {
        if (ActivityCompat.checkSelfPermission(this, permission) != PackageManager.PERMISSION_GRANTED) {
            permissionsToAskFor.add(permission);
        }
    }
    if (!permissionsToAskFor.isEmpty()) {
        ActivityCompat.requestPermissions(this, permissionsToAskFor.toArray(new String[0]), 1);
    }
}
```

> [!NOTE]
> Lorsque vous concevez votre application, tenez compte du moment où ces autorisations doivent être demandées. Les autorisations doivent être demandées lorsqu’elles sont nécessaires, et non à l’avance. Pour plus d’informations, consultez le [guide des autorisations Android](https://developer.android.com/training/permissions/requesting).

## <a name="object-model"></a>Modèle objet

Les classes et les interfaces suivantes gèrent certaines des principales fonctionnalités de la bibliothèque de client Azure Communication Services Calling :

| Nom                                  | Description                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| CallClient| CallClient est le point d’entrée principal de la bibliothèque de client Calling.|
| CallAgent | CallAgent sert à démarrer et à gérer les appels. |
| CommunicationUserCredential | CommunicationUserCredential est utilisé comme informations d’identification du jeton pour instancier CallAgent.|

## <a name="create-an-agent-from-the-user-access-token"></a>Créer un agent à partir du jeton d’accès utilisateur

Avec le jeton utilisateur, un agent d’appel authentifié peut être instancié. En général, ce jeton est généré à partir d’un service avec une authentification propre à l’application. Pour plus d’informations sur les jetons d’accès utilisateur, consultez le [guide des jetons d’accès utilisateur](../../access-tokens.md). Pour les besoins de ce guide de démarrage rapide, remplacez `<User_Access_Token>` par un jeton d’accès utilisateur généré pour votre ressource Azure Communication Services.

```java

/**
 * Create the call agent for placing calls
 */
private void createAgent() {
    String userToken = "<User_Access_Token>";

    try {
        CommunicationUserCredential credential = new CommunicationUserCredential(userToken);
        callAgent = new CallClient().createCallAgent(getApplicationContext(), credential).get();
    } catch (Exception ex) {
        Toast.makeText(getApplicationContext(), "Failed to create call agent.", Toast.LENGTH_SHORT).show();
    }
}

```

## <a name="start-a-call-using-the-call-agent"></a>Démarrer un appel à l’aide de l’agent d’appel

L’appel peut être établi par le biais de l’agent d’appel, et cela nécessite simplement de fournir une liste d’ID d’appelés et les options d’appel. Pour les besoins de ce guide de démarrage rapide, nous utiliserons les options d’appel par défaut sans vidéo et un ID d’appelé unique tiré de l’entrée de texte.

```java
/**
 * Place a call to the callee id provided in `callee_id` text input.
 */
private void startCall() {
    EditText calleeIdView = findViewById(R.id.callee_id);
    String calleeId = calleeIdView.getText().toString();
    
    StartCallOptions options = new StartCallOptions();

    callAgent.call(
        getApplicationContext(),
        new CommunicationUser[] {new CommunicationUser(calleeId)},
        options);
}
```


## <a name="launch-the-app-and-call-the-echo-bot"></a>Lancer l’application et appeler le bot d’écho

L’application peut maintenant être lancée à l’aide du bouton « Run App » de la barre d’outils (Maj+F10). Vérifiez que vous êtes en mesure d’établir des appels en appelant `8:echo123`. Un message pré-enregistré est lu, puis votre propre message est répété.

:::image type="content" source="../media/android/quickstart-android-call-echobot.png" alt-text="Capture d’écran montrant le bouton « Start a new Android Studio Project » sélectionné dans Android Studio.":::

## <a name="sample-code"></a>Exemple de code

Vous pouvez télécharger l’exemple d’application à partir de [GitHub](https://github.com/Azure-Samples/communication-services-android-quickstarts/tree/main/Add%20Voice%20Calling).
