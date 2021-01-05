---
title: Fichier include
description: Fichier include
services: notification-hubs
author: spelluru
ms.service: notification-hubs
ms.topic: include
ms.date: 02/05/2019
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: cd202c98ed605209f5600965ecdb6c0b4c03c17e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "67509111"
---
1. Dans Android Studio, sélectionnez **Outils** dans le menu, puis **SDK Manager**. 
2. Sélectionnez la version cible du Kit de développement logiciel (SDK) Android qui est utilisé dans votre projet. Puis sélectionnez **Afficher les détails du package**. 

    ![Android SDK Manager - sélectionner la version cible](./media/notification-hubs-android-studio-add-google-play-services/notification-hubs-android-studio-sdk-manager.png)
3. Sélectionnez **API Google**, si elles ne sont pas déjà installées.

    ![Android SDK Manager - API Google sélectionnées](./media/notification-hubs-android-studio-add-google-play-services/googole-apis-selected.png)
4. Basculez vers l’onglet **SDK Tools**. Si vous n’avez pas déjà installé un service Google Play, sélectionnez **Google Play Services** comme indiqué dans l’image suivante. Sélectionnez ensuite **Appliquer** pour procéder à l’installation. Notez le chemin du Kit de développement logiciel (SDK). Vous l’utiliserez à une étape suivante.

    ![Android SDK Manager - Google Play Services sélectionné](./media/notification-hubs-android-studio-add-google-play-services/google-play-services-selected.png)
3. Si la boîte de dialogue de **confirmation de la modification** s’affiche, sélectionnez **OK**. Le programme d’installation installe les composants demandés. Sélectionnez **Terminer** une fois que les composants sont installés.
4. Sélectionnez **OK** pour fermer la boîte de dialogue **Paramètres pour les nouveaux projets**.  
5. Ouvrez le fichier build.gradle dans le répertoire **app**, puis ajoutez la ligne suivante sous `dependencies`. 

    ```gradle
    implementation 'com.google.android.gms:play-services-gcm:16.0.0'
    ```
5. Sélectionnez l’icône **Synchroniser maintenant** dans la barre d’outils.

    ![Synchroniser avec Gradle](./media/notification-hubs-android-studio-add-google-play-services/gradle-sync.png)
1. Ouvrez le fichier AndroidManifest.xml, puis ajoutez la balise suivante à la balise *application*.

    ```xml
    <meta-data android:name="com.google.android.gms.version"
         android:value="@integer/google_play_services_version" />
    ```
