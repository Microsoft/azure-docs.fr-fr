---
author: trrwilson
ms.service: cognitive-services
ms.topic: include
ms.date: 10/15/2020
ms.author: travisw
ms.openlocfilehash: 2efe17bdf597b7e3ba7d0277f06d4b24e9b51db3
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96187667"
---
1. Lancez Android Studio, puis sélectionnez **Démarrer un nouveau projet Android Studio** dans la fenêtre d’**accueil**.

    ![Capture d’écran de la fenêtre d’accueil d’Android Studio](../articles/cognitive-services/Speech-Service/media/sdk/qs-java-android-01-start-new-android-studio-project.png)

1. L’Assistant **Choix de votre projet** apparaît. Sélectionnez **Téléphone et tablette** et **Activité vide** dans la zone de sélection des activités. Sélectionnez **Suivant**.

   ![Capture d’écran de l’Assistant Choix de votre projet](../articles/cognitive-services/Speech-Service/media/sdk/qs-java-android-02-target-android-devices.png)

1. Dans l’écran **Configurer votre projet**, entrez *Quickstart* pour **Nom** et *samples.speech.cognitiveservices.microsoft.com* pour **Nom du package**. Sélectionnez ensuite un répertoire de projet. Pour **Niveau d’API minimal**, sélectionnez **API 23 : Android 6.0 (Marshmallow)** . Laissez toutes les autres cases décochées, puis sélectionnez **Terminer**.

   ![Capture d’écran de l’Assistant Configuration de votre projet](../articles/cognitive-services/Speech-Service/media/sdk/qs-java-android-03-create-android-project.png)

Quelques minutes sont nécessaires à Android Studio pour préparer votre nouveau projet Android. Configurez ensuite le projet pour découvrir le kit SDK Speech Azure Cognitive Services et utiliser Java 8.

[!INCLUDE [License notice](cognitive-services-speech-service-license-notice.md)]

La version actuelle du SDK Speech Cognitive Services est 1.13.0.

Le SDK Speech pour Android est empaqueté au format [AAR (bibliothèque Android)](https://developer.android.com/studio/projects/android-library), qui inclut les bibliothèques nécessaires et les autorisations Android requises.
Il est hébergé dans un dépôt Maven sur https:\//csspeechstorage.blob.core.windows.net/maven/.

Configurez votre projet pour utiliser le kit SDK Speech. Ouvrez la fenêtre **Structure du projet** en sélectionnant **Fichier** > **Structure du projet** dans la barre de menus Android Studio. Dans la fenêtre **Structure du projet**, apportez les modifications suivantes :

1. Dans la liste située sur le côté gauche de la fenêtre, sélectionnez **Project** (Projet). Modifiez les paramètres du **dépôt de bibliothèque par défaut** en ajoutant une virgule et l’URL du dépôt Maven, entourée de guillemets simples : 'https:\//csspeechstorage.blob.core.windows.net/maven/'

   ![Capture d’écran de la fenêtre de structure d’un projet](../articles/cognitive-services/Speech-Service/media/sdk/qs-java-android-06-add-maven-repository.png)

1. Dans le même écran, sur le côté gauche, sélectionnez **application**. Ensuite, sélectionnez l’onglet **Dependencies** (Dépendances) en haut de la fenêtre. Sélectionnez le signe plus vert ( **+** ), puis **Library dependency** (Dépendance de bibliothèque) dans le menu déroulant.

   ![Capture d’écran de la dépendance de bibliothèque](../articles/cognitive-services/Speech-Service/media/sdk/qs-java-android-07-add-module-dependency.png)

1. Dans la fenêtre qui s’affiche, entrez le nom et la version du SDK Speech pour Android, *com.microsoft.cognitiveservices.speech:client-sdk:1.13.0*. Sélectionnez ensuite **OK**.
   Le kit SDK Speech devrait à présent apparaître dans la liste des dépendances, comme indiqué ci-dessous :

   ![Capture d’écran du kit SDK Speech dans la liste des dépendances](../articles/cognitive-services/Speech-Service/media/sdk/qs-java-android-08-dependency-added-1.0.0.png)

1. Sélectionnez l’onglet **Properties** (Propriétés). À des fins de **compatibilité source** et **compatibilité cible**, sélectionnez **1.9**.

   ![Capture d’écran de la compatibilité source et de la compatibilité cible](../articles/cognitive-services/Speech-Service/media/sdk/qs-java-android-09-dependency-added.png)

1. Sélectionnez **OK** pour fermer la fenêtre **Structure du projet** et appliquer vos modifications au projet.
