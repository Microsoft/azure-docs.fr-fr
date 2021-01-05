---
author: trevorbye
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: include
ms.date: 02/20/2020
ms.author: trbye
ms.openlocfilehash: c341ba20ece26e15255faf086e5bd2904fbaa797
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95096025"
---
Dans ce guide de démarrage rapide, vous allez découvrir comment utiliser le SDK Speech Devices pour Android pour créer un produit avec reconnaissance vocale ou pour l’utiliser comme appareil de [transcription de conversation](../conversation-transcription.md).

Ce guide nécessite un compte [Azure Cognitive Services](../overview.md#try-the-speech-service-for-free) avec une ressource de service Speech.

Le code source de l’exemple d’application est fourni avec le Kit de développement logiciel (SDK) Speech Devices. Il est également [disponible sur GitHub](https://github.com/Azure-Samples/Cognitive-Services-Speech-Devices-SDK).

## <a name="prerequisites"></a>Prérequis

Avant de commencer à utiliser le SDK Speech Devices, vous devez :

- Suivre les instructions accompagnant votre [kit de développement](../get-speech-devices-sdk.md) pour mettre l’appareil sous tension.

- Télécharger la dernière version du [SDK Speech Devices](../speech-devices-sdk.md) et extraire le fichier .zip dans votre répertoire de travail.

  > [!NOTE]
  > Ce guide de démarrage rapide part du principe que l’application est extraite dans C:\SDSDK\Android-Sample-Release.

- Pour obtenir une [clé d’abonnement Azure pour le service Speech](../overview.md#try-the-speech-service-for-free)

- Si vous envisagez d'utiliser la transcription de conversation, vous devez utiliser un [microphone circulaire](../get-speech-devices-sdk.md) et cette fonctionnalité n'est actuellement disponible que pour les langues « en-US » et « zh-CN » dans les régions « centralus » et « eastasia ». Vous devez disposer d’une clé Speech dans une de ces régions pour utiliser la transcription de conversation.

- Si vous envisagez d’utiliser le service Speech pour identifier les intentions (ou les actions) à partir des énoncés utilisateur, vous aurez besoin d’un abonnement [LUIS (Language Understanding Service)](../../luis/luis-how-to-azure-subscription.md). Pour plus d’informations sur LUIS et la reconnaissance des intentions, consultez [Reconnaître les intentions vocales avec LUIS, C#](../how-to-recognize-intents-from-speech-csharp.md).

  Vous pouvez [créer un modèle LUIS simple](../../luis/index.yml) ou utiliser l’exemple de modèle LUIS, LUIS-example.json. L’exemple de modèle LUIS est accessible à partir du [site de téléchargement du Kit de développement logiciel (SDK) Speech Devices](https://aka.ms/sdsdk-luis). Pour charger le fichier JSON de votre modèle dans le [portail LUIS](https://www.luis.ai/home), sélectionnez **Importer une nouvelle application**, puis sélectionnez le fichier JSON.

- Installez [Android Studio](https://developer.android.com/studio/) et [Vysor](https://vysor.io/download/) sur votre PC.

## <a name="set-up-the-device"></a>Configurer l’appareil

1. Démarrez Vysor sur votre ordinateur.

   ![Vysor](../media/speech-devices-sdk/qsg-3.png)

1. Votre appareil doit être répertorié sous **Choose a device** (Choisir un appareil). Sélectionnez le bouton **View** (Afficher) en regard de l’appareil.

1. Connectez-vous à votre réseau sans fil en sélectionnant l’icône de dossier, puis sélectionnez **Settings** (Paramètres) > **WLAN** (WLAN).

   ![WLAN Vysor](../media/speech-devices-sdk/qsg-4.png)

   > [!NOTE]
   > Si votre entreprise applique des stratégies relatives à la connexion d’appareils à son système Wi-Fi, vous devez obtenir l’adresse MAC et contacter votre service informatique pour savoir comment vous connecter au système Wi-Fi de votre entreprise.
   >
   > Pour trouver l’adresse MAC du kit de développement, sélectionnez l’icône de dossier de fichiers sur le bureau du kit de développement.
   >
   > ![Dossier de fichiers Vysor](../media/speech-devices-sdk/qsg-10.png)
   >
   > Sélectionnez **Paramètres**. Recherchez « mac address », puis sélectionnez **Mac address** (Adresse MAC) > **Advanced WLAN** (WLAN avancé). Notez l’adresse MAC qui s’affiche au bas de la boîte de dialogue.
   >
   > ![Adresse MAC Vysor](../media/speech-devices-sdk/qsg-11.png)
   >
   > Certaines entreprises peuvent appliquer une durée limite pour la connexion d’un appareil à leur système Wi-Fi. Vous devrez peut-être étendre l’inscription du kit de développement auprès de votre système Wi-Fi au bout d’un nombre de jours spécifique.

## <a name="run-the-sample-application"></a>Exécuter l’exemple d’application

Pour valider la configuration de votre kit de développement, générez et installez l’exemple d’application :

1. Démarrez Android Studio.

1. Sélectionnez **Ouvrir un projet Android Studio**.

   ![Android Studio : ouverture d’un projet existant](../media/speech-devices-sdk/qsg-5.png)

1. Accédez à C:\SDSDK\Android-Sample-Release\example. Sélectionnez **OK** pour ouvrir l’exemple de projet.

1. Configurez Gradle pour référencer le SDK Speech. Les fichiers suivants se trouvent sous **Scripts Gradle** dans Android Studio.

    Mettez à jour **build.gradle(Project:example)** , le bloc allprojects devant correspondre à celui ci-dessous, en ajoutant les lignes Maven.

    ```xml
    allprojects {
        repositories {
            google()
            jcenter()
            mavenCentral()
            maven {
                url 'https://csspeechstorage.blob.core.windows.net/maven/'
            }
        }
    }
    ```

    Mettez à jour **build.gradle(Module:app)** en ajoutant cette ligne à la section des dépendances. 
    
    ```xml
    implementation'com.microsoft.cognitiveservices.speech:client-sdk:1.14.0'
    ```
    
1. Ajoutez votre clé d’abonnement Speech au code source. Si vous voulez essayer la reconnaissance de l’intention, ajoutez également votre clé d’abonnement au [service Language Understanding](https://azure.microsoft.com/services/cognitive-services/language-understanding-intelligent-service/), ainsi que l’ID de l’application.

   Pour Speech et LUIS, vos informations vont dans MainActivity.java :

   ```java
    // Subscription
    private static String SpeechSubscriptionKey = "<enter your subscription info here>";
    private static String SpeechRegion = "westus"; // You can change this if your speech region is different.
    private static String LuisSubscriptionKey = "<enter your subscription info here>";
    private static String LuisRegion = "westus2"; // you can change this, if you want to test the intent, and your LUIS region is different.
    private static String LuisAppId = "<enter your LUIS AppId>";
   ```

   Si vous utilisez la transcription de conversation, vos informations de clé et de région Speech sont aussi nécessaires dans conversation.java :

   ```java
    private static final String CTSKey = "<Conversation Transcription Service Key>";
    private static final String CTSRegion="<Conversation Transcription Service Region>";// Region may be "centralus" or "eastasia"
   ```

1. Le mot clé par défaut est « Computer ». Vous pouvez également essayer l’un des autres mots clés fournis, tels que « Machine » ou « Assistant ». Les fichiers de ressources pour ces mots clés alternatifs figurent dans le dossier keyword du Kit de développement logiciel (SDK) Speech Devices. Par exemple, C:\SDSDK\Android-Sample-Release\keyword\Computer contient les fichiers utilisés pour le mot clé signifiant « Computer ».

   > [!TIP]
   > Vous pouvez également [créer un mot clé personnalisé](../custom-keyword-basics.md).

   Pour utiliser un nouveau mot clé, mettez à jour les deux lignes suivantes dans `MainActivity.java` et copiez l’ensemble de mots clés dans votre application. Par exemple, pour utiliser le mot clé « Machine » du package de mots clés kws-machine.zip :

   - Copiez le package de mots clés dans le dossier "C:\SDSDK\Android-Sample-Release\example\app\src\main\assets\".
   - Mettez à jour le fichier `MainActivity.java` avec le mot clé et le nom de l’ensemble :

     ```java
     private static final String Keyword = "Machine";
     private static final String KeywordModel = "kws-machine.zip" // set your own keyword package name.
     ```

1. Mettez à jour les lignes ci-après, qui contiennent les paramètres géométriques du réseau de microphones :

   ```java
   private static final String DeviceGeometry = "Circular6+1";
   private static final String SelectedGeometry = "Circular6+1";
   ```

   Ce tableau répertorie les valeurs prises en charge :

   | Variable | Signification | Valeurs disponibles |
   | -------- | ------- | ---------------- |
   | `DeviceGeometry` | Configuration physique du micro | Pour un kit de développement circulaire : `Circular6+1` |
   |          |         | Pour un kit de développement linéaire : `Linear4` |
   | `SelectedGeometry` | Configuration du logiciel du micro | Pour un kit de développement circulaire utilisant tous les micros : `Circular6+1` |
   |          |         | Pour un kit de développement circulaire utilisant quatre micros : `Circular3+1` |
   |          |         | Pour un kit de développement linéaire utilisant tous les micros : `Linear4` |
   |          |         | Pour un kit de développement linéaire utilisant deux micros : `Linear2` |

1. Pour générer l’application, dans le menu **Run** (Exécuter), sélectionnez **Run ’app’** (Exécuter « application »). La boîte de dialogue **Select Deployment Target** (Sélectionner la cible de déploiement) s’affiche.

1. Sélectionnez votre appareil, puis sélectionnez **OK** (OK) pour déployer l’application sur l’appareil.

   ![Boîte de dialogue de sélection de la cible de déploiement](../media/speech-devices-sdk/qsg-7.png)

1. L’exemple d’application du Kit de développement logiciel (SDK) Speech Devices démarre et affiche les options suivantes :

   ![Exemple d’application et options du Kit de développement logiciel (SDK) Speech Devices](../media/speech-devices-sdk/qsg-8.png)

1. Essayez la nouvelle démonstration de transcription de conversation. Démarrez la transcription avec « Start Session ». Par défaut, tout le monde est invité. Cependant, si vous avez des signatures vocales des participants, vous pouvez les placer dans un fichier `/video/participants.properties` sur l'appareil. Pour générer la signature vocale, consultez [Transcrire des conversations (SDK)](../how-to-use-conversation-transcription.md).

   ![Application de démonstration de la transcription de conversation](../media/speech-devices-sdk/qsg-15.png)

1. Expérimentez !

## <a name="troubleshooting"></a>Dépannage

Si vous ne pouvez pas vous connecter à Speech Device. Tapez la commande suivante dans une fenêtre d’invite de commandes. Une liste d’appareils est alors retournée :

```powershell
 adb devices
```

> [!NOTE]
> Cette commande utilise Android Debug Bridge, `adb.exe`, qui fait partie intégrante de l’installation d’Android Studio. Cet outil figure dans le répertoire C:\Users\[nom de l’utilisateur]\AppData\Local\Android\Sdk\platform-tools. Vous pouvez ajouter ce répertoire à votre chemin d’accès pour que l’invocation de l’outil `adb` soit plus pratique. Dans le cas contraire, vous devez spécifier le chemin d’accès complet à votre installation d’adb.exe dans chaque commande qui invoque `adb`.
>
> Si vous rencontrez une erreur `no devices/emulators found`, vérifiez que votre câble USB est bien connecté et qu’il s’agit d’un câble de qualité.