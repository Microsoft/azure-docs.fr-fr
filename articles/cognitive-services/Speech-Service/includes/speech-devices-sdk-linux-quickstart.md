---
author: trevorbye
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: include
ms.date: 02/20/2020
ms.author: trbye
ms.openlocfilehash: 3c093539301f3f4e3309094f0b53a1f0e8393150
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95097028"
---
Dans ce guide de démarrage rapide, vous allez découvrir comment utiliser le SDK Speech Devices pour Linux pour créer un produit avec reconnaissance vocale ou pour l’utiliser comme appareil de [transcription de conversation](../conversation-transcription.md). Actuellement, seul [Azure Kinect DK](https://azure.microsoft.com/services/kinect-dk/) est pris en charge.

L’application est générée avec le package du SDK Speech et l’IDE Eclipse Java (v4) sur Linux 64 bits (Ubuntu 16.04, Ubuntu 18.04, Debian 9, RHEL 7/8, CentOS 7/8). Elle s’exécute sur un environnement d’exécution Java 8 (JRE) 64 bits.

Ce guide nécessite un compte [Azure Cognitive Services](../overview.md#try-the-speech-service-for-free) avec une ressource de service Speech. 

Le code source de l’[exemple d’application](https://aka.ms/sdsdk-download-JRE) est inclus avec le SDK Speech Devices. Il est également [disponible sur GitHub](https://github.com/Azure-Samples/Cognitive-Services-Speech-Devices-SDK).

## <a name="prerequisites"></a>Prérequis

Ce démarrage rapide nécessite :

* Système d’exploitation : Linux 64 bits (Ubuntu 16.04, Ubuntu 18.04, Debian 9, RHEL 7/8, CentOS 7/8)
* [Azure Kinect DK](https://azure.microsoft.com/services/kinect-dk/)
* [IDE Eclipse Java](https://www.eclipse.org/downloads/)
* [Java 8](https://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html) ou [JDK 8](https://www.oracle.com/technetwork/java/javase/downloads/index.html) uniquement.
* Clé d’abonnement Azure pour le service Speech. [Vous pouvez en obtenir une gratuitement](../overview.md#try-the-speech-service-for-free).
* Téléchargez la dernière version du [SDK Speech Devices](https://aka.ms/sdsdk-download-JRE) pour Java et décompressez le fichier .zip dans votre répertoire de travail.
   > [!NOTE]
   > Ce guide de démarrage rapide part du principe que l’application est extraite dans /home/wcaltest/JRE-Sample-Release.

Vérifiez que ces dépendances sont installées avant de démarrer Eclipse.

* Sur Ubuntu :

  ```sh
  sudo apt-get update
  sudo apt-get install libssl1.0.0 libasound2
  ```

* Sur Debian 9 :

  ```sh
  sudo apt-get update
  sudo apt-get install libssl1.0.2 libasound2
  ```

* Sur RHEL/CentOS :
  
  ```sh
  sudo yum update
  sudo yum install alsa-lib openssl
  ```

  > [!NOTE]
  > - Sur RHEL/CentOS 7, suivez les instructions décrivant [comment configurer RHEL/CentOS 7 pour le Kit de développement logiciel (SDK) Speech](~/articles/cognitive-services/speech-service/how-to-configure-rhel-centos-7.md).
> - Sur RHEL/CentOS 8, suivez les instructions du [Guide pratique pour configurer OpenSSL pour Linux](~/articles/cognitive-services/speech-service/how-to-configure-openssl-linux.md).

La fonctionnalité de transcription de conversation n'est actuellement disponible que pour les langues « en-US » et « zh-CN » dans les régions « USA Centre » et « Asie Est ». Vous devez disposer d’une clé Speech dans une de ces régions pour utiliser la transcription de conversation.

Si vous prévoyez d’utiliser les intentions, vous aurez besoin d’un abonnement [Language Understanding Intelligent Service (LUIS)](../../luis/luis-how-to-azure-subscription.md). Pour plus d’informations sur LUIS et la reconnaissance des intentions, consultez [Reconnaître les intentions vocales avec LUIS, C#](../how-to-recognize-intents-from-speech-csharp.md). Un [exemple de modèle LUIS](https://aka.ms/sdsdk-luis) est disponible pour cette application.

## <a name="create-and-configure-the-project"></a>Créer et configurer le projet

1. Démarrez Eclipse.

1. Dans le champ **Workspace** (Espace de travail) de **Eclipse IDE Launcher**, entrez le nom d’un nouveau répertoire d’espace de travail. Sélectionnez ensuite **Launch** (Lancer).

   ![Capture d’écran montrant l’utilitaire Eclipse Launcher.](../media/speech-devices-sdk/eclipse-launcher-linux.png)

1. La fenêtre principale de l’IDE Eclipse apparaît au bout d’un instant. Fermez l’écran d’accueil s’il en existe un.

1. Dans la barre de menus Eclipse, créez un projet en choisissant **File (Fichier)**  > **New (Nouveau)**  > **Java Project (Projet Java)** . Si ce n’est pas disponible, choisissez **Project** (Projet), puis **Java Project** (Projet Java).

1. L’Assistant **New Java Project** (Nouveau projet Java) démarre. **Accédez**  à l’emplacement de l’exemple de projet. Sélectionnez **Terminer**.

   ![Capture d’écran de l’Assistant New Java Project (Nouveau projet Java)](../media/speech-devices-sdk/eclipse-new-java-project-linux.png)

1. Dans **Package explorer** (l’Explorateur de package), cliquez avec le bouton droit sur votre projet. Choisissez **Configure (Configurer)**  > **Convert to Maven Project (Convertir en projet Maven)** dans le menu contextuel. Sélectionnez **Terminer**.

   ![Capture d’écran de l’explorateur de package](../media/speech-devices-sdk/eclipse-convert-to-maven.png)

1. Ouvrez le fichier pom.xml et modifiez-le.

    À la fin du fichier, avant la balise de fermeture `</project>`, créez des éléments `repositories` et `dependencies`, comme indiqué ici, et vérifiez que le `version` correspond bien à votre version actuelle :
    ```xml    
    <repositories>
         <repository>
             <id>maven-cognitiveservices-speech</id>
             <name>Microsoft Cognitive Services Speech Maven Repository</name>
             <url>https://csspeechstorage.blob.core.windows.net/maven/</url>
         </repository>
    </repositories>
 
    <dependencies>
        <dependency>
             <groupId>com.microsoft.cognitiveservices.speech</groupId>
             <artifactId>client-sdk</artifactId>
             <version>1.14.0</version>
        </dependency>
    </dependencies>
   ```

1. Dans **Package explorer** (l’Explorateur de package), cliquez avec le bouton droit sur votre projet. Choisissez **Properties (Propriétés)** , puis **Run/Debug Settings**  (Paramètres d’exécution/débogage) > **New... (Nouveau...)** > **Java Application** (Application Java). 

1. La fenêtre **Edit Configuration** (Modifier la configuration) apparaît. Dans le champ **Name (Nom)** , entrez **Main (Principal)** et utilisez **Search (Rechercher)** pour la **Main Class (Classe principale)** pour rechercher et sélectionner **com.microsoft.cognitiveservices.speech.samples.FunctionsList**.

   ![Capture d’écran de la modification de la configuration du lancement](../media/speech-devices-sdk/eclipse-edit-launch-configuration-linux.png)

1. Copiez les fichiers binaires audio de votre architecture cible, de **Linux-ARM** ou **Linux-x64** vers l’emplacement du projet Java, par exemple **/home/wcaltest/JRE-Sample-Release**

1. Dans la même fenêtre **Edit Configuration**, sélectionnez la page **Environment** (Environnement) puis **New** (Nouveau). La fenêtre **New Environment Variable** (Nouvelle variable d’environnement) apparaît. Dans le champ **Name** (Nom), entrez **LD_LIBRARY_PATH** et, dans le champ **Value** (Valeur), entrez le dossier contenant les fichiers *.so, par exemple **/home/wcaltest/JRE-Sample-Release**

1. Copiez `kws.table` et `participants.properties` dans le dossier **target/classes** du projet


## <a name="configure-the-sample-application"></a>Configurer l’exemple d’application

1. Ajoutez votre clé d’abonnement Speech au code source. Si vous voulez essayer la reconnaissance de l’intention, ajoutez également votre clé d’abonnement au [service Language Understanding](https://azure.microsoft.com/services/cognitive-services/language-understanding-intelligent-service/), ainsi que l’ID de l’application.

   Pour la reconnaissance vocale et LUIS, vos informations vont dans `FunctionsList.java` :

   ```java
    // Subscription
    private static String SpeechSubscriptionKey = "<enter your subscription info here>";
    private static String SpeechRegion = "westus"; // You can change this if your speech region is different.
    private static String LuisSubscriptionKey = "<enter your subscription info here>";
    private static String LuisRegion = "westus2"; // you can change this, if you want to test the intent, and your LUIS region is different.
    private static String LuisAppId = "<enter your LUIS AppId>";
   ```

    Si vous utilisez la transcription de conversation, vos informations de clé et de région de reconnaissance vocale sont également nécessaires dans `Cts.java` :

   ```java
    private static final String CTSKey = "<Conversation Transcription Service Key>";
    private static final String CTSRegion="<Conversation Transcription Service Region>";// Region may be "centralus" or "eastasia"
    ```

1. Le mot clé par défaut (keyword) est « Computer ». Vous pouvez également essayer l’un des autres mots clés fournis, tels que « Machine » ou « Assistant ». Les fichiers de ressources pour ces mots clés alternatifs figurent dans le dossier keyword du Kit de développement logiciel (SDK) Speech Devices. Par exemple, `/home/wcaltest/JRE-Sample-Release/keyword/Computer` contient les fichiers utilisés pour le mot clé « Computer ».

   > [!TIP]
   > Vous pouvez également [créer un mot clé personnalisé](../custom-keyword-basics.md).

    Pour utiliser un nouveau mot clé, mettez à jour la ligne suivante dans `FunctionsList.java`, puis copiez le mot clé dans votre application. Par exemple, pour utiliser le mot clé « Machine » du package de mots clés `machine.zip` :

   * Copiez le fichier `kws.table` du package zip dans le dossier projet **target/classes**.

   * Mettez à jour le `FunctionsList.java` avec le nom du mot clé :

     ```java
     private static final String Keyword = "Machine";
     ```

## <a name="run-the-sample-application-from-eclipse"></a>Exécuter l’exemple d’application à partir d’Eclipse

1. Dans la barre de menu Eclipse, sélectionnez **Run (Exécuter)**  > **Run (Exécuter)** 

1. L’exemple d’application du Kit de développement logiciel (SDK) Speech Devices démarre et affiche les options suivantes :

   ![Capture d’écran montrant un exemple d’application et d’options du SDK Speech Devices.](../media/speech-devices-sdk/java-sample-app-linux.png)

1. Essayez la nouvelle démonstration de **Transcription de conversation**. Démarrez la transcription avec **Session** > **Démarrer**. Par défaut, tout le monde est invité. Cependant, si vous avez des signatures vocales des participants, vous pouvez les placer dans `participants.properties`, dans le dossier **target/classes** du projet. Pour générer la signature vocale, consultez [Transcrire des conversations (SDK)](../how-to-use-conversation-transcription.md).

   ![Capture d’écran montrant une application de démonstration de la transcription de conversation.](../media/speech-devices-sdk/cts-sample-app-linux.png)

## <a name="create-and-run-standalone-the-application"></a>Créer et exécuter l’application autonome

1. Dans **Package explorer** (l’Explorateur de package), cliquez avec le bouton droit sur votre projet. Choisissez **Export** (Exporter). 
1. La fenêtre **Export** (Exporter) apparaît. Développez **Java**, sélectionnez **Runnable JAR file** (Fichier JAR exécutable), puis sélectionnez **Suivant**.

   ![Capture d’écran montrant la fenêtre d’exportation.](../media/speech-devices-sdk/eclipse-export-linux.png) 

1. La fenêtre **Runnable JAR File Export** (Fichier JAR exécutable) apparaît. Choisissez une **Export destination** (Destination de l’exportation) pour l’application, puis sélectionnez **Finish** (Terminer).
 
   ![Capture d’écran montrant la fenêtre d’exportation d’un fichier JAR exécutable.](../media/speech-devices-sdk/eclipse-export-jar-linux.png)

1. Placez `kws.table` et `participants.properties` dans le dossier de destination sélectionné ci-dessus, car ces fichiers sont nécessaires à l’application.

1. Définissez LD_LIBRARY_LIB sur le dossier contenant les fichiers *.so

     ```bash
     export LD_LIBRARY_PATH=/home/wcaltest/JRE-Sample-Release
     ```

1. Pour exécuter l’application autonome

     ```bash
     java -jar SpeechDemo.jar
     ```