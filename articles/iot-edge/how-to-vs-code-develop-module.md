---
title: Développer et déboguer des modules pour Azure IoT Edge | Microsoft Docs
description: Utiliser Visual Studio Code afin de développer, générer et déboguer un module pour Azure IoT Edge à l'aide de C#, Python, Node.js, Java ou C
services: iot-edge
keywords: ''
author: kgremban
ms.author: kgremban
ms.date: 08/07/2019
ms.topic: conceptual
ms.service: iot-edge
ms.custom: devx-track-js
ms.openlocfilehash: ebc12e6d64d015267497497bebc22c8586adf999
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95995911"
---
# <a name="use-visual-studio-code-to-develop-and-debug-modules-for-azure-iot-edge"></a>Utiliser Visual Studio Code afin de développer et déboguer des modules pour Azure IoT Edge

Vous pouvez transformer votre logique métier en modules pour Azure IoT Edge. Cet article explique comment utiliser Visual Studio Code comme outil principal pour développer et déboguer des modules.

Deux méthodes de débogage de module écrites en C#, Node.js ou Java sont disponibles dans Visual Studio Code : Vous pouvez les associer à un processus dans un conteneur de modules ou lancer le code des modules en mode débogage. Vous ne pouvez déboguer des modules écrits en Python ou en C qu'en les associant à un processus dans des conteneurs Linux amd64.

Si vous n’êtes pas familiarisé avec les fonctionnalités de débogage de Visual Studio Code, découvrez-en plus sur le [débogage](https://code.visualstudio.com/Docs/editor/debugging).

Cet article fournit des instructions pour le développement et le débogage de modules dans plusieurs langues pour plusieurs architectures. Actuellement, Visual Studio Code prend en charge les modules écrits C#, C, Python, Node.js et Java. Les architectures d’appareils prises en charge sont x64 et ARM32. Pour plus d’informations sur les systèmes d’exploitation, les langages et les architectures pris en charge, consultez [Prise en charge des langages et architectures](module-development.md#language-and-architecture-support).

>[!NOTE]
>La prise en charge du développement et du débogage des appareils Linux ARM64 est disponible en [préversion publique](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Pour plus d’informations, consultez [Développer et déboguer des modules IoT Edge ARM64 dans Visual Studio Code (préversion)](https://devblogs.microsoft.com/iotdev/develop-and-debug-arm64-iot-edge-modules-in-visual-studio-code-preview).

## <a name="prerequisites"></a>Prérequis

Vous pouvez utiliser un ordinateur ou une machine virtuelle Windows, macOS ou Linux comme machine de développement. Sur les ordinateurs Windows, vous pouvez développer des modules Windows ou Linux. Pour développer des modules Windows, utilisez un ordinateur Windows exécutant la version 1809/build 17763 ou une version plus récente. Pour développer des modules Linux, utilisez un ordinateur Windows qui est conforme à la [configuration requise pour Docker Desktop](https://docs.docker.com/docker-for-windows/install/#what-to-know-before-you-install).

Commencez par installer [Visual Studio Code](https://code.visualstudio.com/), puis ajoutez les extensions suivantes :

- [Outils IoT Azure](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
- [Extension Docker](https://marketplace.visualstudio.com/items?itemName=PeterJausovec.vscode-docker)
- Extension(s) Visual Studio spécifique(s) au langage utilisé pour le développement :
  - C#, y compris Azure Functions : [Extension C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)
  - Python : [Extension Python](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
  - Java : [Pack d’extension Java pour Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-pack)
  - C : [Extension C/C++](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools)

Vous devrez également installer d'autres outils spécifiques au langage utilisé pour développer votre module :

- C#, y compris Azure Functions : [Kit de développement logiciel (SDK) .NET Core 2.1](https://www.microsoft.com/net/download)

- Python : [Python](https://www.python.org/downloads/) et [PIP](https://pip.pypa.io/en/stable/installing/#installation) pour l'installation des packages Python (généralement inclus avec votre installation Python).

- Node.js : [Node.js](https://nodejs.org). Vous pourrez également installer [Yeoman](https://www.npmjs.com/package/yo) et le [générateur de module Node.js Azure IoT Edge](https://www.npmjs.com/package/generator-azure-iot-edge-module).

- Java : [Java SE Development Kit 10](/azure/developer/java/fundamentals/java-jdk-long-term-support) et [Maven](https://maven.apache.org/). Vous devrez [définir la variable d'environnement `JAVA_HOME`](https://docs.oracle.com/cd/E19182-01/820-7851/inst_cli_jdk_javahome_t/)de manière à ce qu'elle pointe vers votre installation JDK.

Pour générer et déployer l'image de votre module, vous devez disposer de Docker pour générer l'image du module et d'un registre de conteneurs pour stocker cette image :

- [Docker Community Edition](https://docs.docker.com/install/) sur votre ordinateur de développement

- [Azure Container Registry](../container-registry/index.yml) ou [Docker Hub](https://docs.docker.com/docker-hub/repos/#viewing-repository-tags)

    > [!TIP]
    > Vous pouvez utiliser un registre Docker local pour le prototype et à des fins de test, au lieu d’un registre cloud.

À moins que vous ne développiez votre module en C, vous devez également disposer de l'[outil de développement Azure IoT EdgeHub](https://pypi.org/project/iotedgehubdev/) basé sur Python afin de configurer votre environnement de développement local pour déboguer, exécuter et tester votre solution IoT Edge. Si ce n’est pas déjà fait, installez [Python (2.7/3.6/3.7) et Pip](https://www.python.org/), puis installez **iotedgehubdev** en exécutant la commande suivante dans votre terminal.

   ```cmd
   pip install --upgrade iotedgehubdev
   ```
   
> [!NOTE]
> Actuellement, iotedgehubdev utilise une bibliothèque docker-py qui n’est pas compatible avec Python 3.8.
>
> Si vous avez plusieurs Python, y compris Python 2.7 préinstallé (par exemple, sur Ubuntu ou MacOS), assurez-vous que vous utilisez le bon `pip` ou `pip3` pour installer **iotedgehubdev**.

Pour tester votre module sur un appareil, vous devez disposer d’un hub IoT actif avec au moins un appareil IoT Edge. Pour utiliser votre ordinateur comme appareil IoT Edge, suivez les étapes décrites dans le guide de démarrage rapide pour [Linux](quickstart-linux.md) ou [Windows](quickstart.md). Si vous exécutez le démon IoT Edge sur votre ordinateur de développement, vous devrez peut-être arrêter EdgeHub et EdgeAgent avant de passer à l'étape suivante.

## <a name="create-a-new-solution-template"></a>Créer un modèle de solution

Les étapes suivantes expliquent comment créer un module IoT Edge dans votre langage de développement préféré (y compris Azure Functions, en C#) en utilisant Visual Studio Code et les outils IoT Azure. Vous commencez par créer une solution, puis vous y générez le premier module. Chaque solution peut contenir plusieurs modules.

1. Sélectionnez **Afficher** > **Palette de commandes**.

1. Dans la palette de commandes, entrez et exécutez la commande **Azure IoT Edge: New IoT Edge solution** (Azure IoT Edge : Nouvelle solution IoT Edge).

   ![Exécuter la nouvelle solution IoT Edge](./media/how-to-develop-csharp-module/new-solution.png)

1. Accédez au dossier où vous souhaitez créer la solution, puis choisissez **Sélectionner le dossier**.

1. Entrez un nom pour votre solution.

1. Sélectionnez un modèle de module correspondant à votre langage de développement préféré. Il s'agira du premier module de la solution.

1. Entrez un nom pour votre module. Choisissez un nom qui est unique dans le registre de conteneurs.

1. Indiquez le nom du référentiel d’images du module. Par défaut, Visual Studio Code nomme le module **localhost:5000/<nom de votre module\>** . Remplacez-le par vos propres informations de registre. Si vous utilisez un registre Docker local à des fins de test, **localhost** convient. Si vous utilisez Azure Container Registry, utilisez le serveur de connexion à partir des paramètres de votre registre. Le serveur de connexion se présente sous la forme **_\<registry name\>_ .azurecr.io**. Remplacez uniquement la partie **localhost : 5000** de la chaîne de sorte que le résultat se présente sous la forme **\<*registry name*\>. azurecr.io/ _\<your module name\>_**.

   ![Fourniture du référentiel d’images Docker](./media/how-to-develop-csharp-module/repository.png)

À partir des informations que vous avez fournies, Visual Studio Code crée une solution IoT Edge, puis la charge dans une nouvelle fenêtre.

La solution comprend quatre éléments :

- Un dossier **.vscode**, qui contient les configurations de débogage.

- Un dossier **modules**, qui contient des sous-dossiers pour chaque module.  Dans le dossier de chaque module se trouve fichier, **module.json**, qui contrôle la façon dont les modules sont générés et déployés.  Ce fichier doit être modifié pour remplacer le registre localhost du conteneur de déploiement de module par un registre distant. À ce stade, vous n’avez qu’un seul module.  Vous pouvez cependant en ajouter via la palette de commandes avec la commande **Azure IoT Edge : Ajouter un module IoT Edge**.

- Un fichier **.env**, qui liste vos variables d’environnement. Si Azure Container Registry est votre registre, celui-ci comprend un nom d’utilisateur et un mot de passe Azure Container Registry.

  > [!NOTE]
  > Le fichier d’environnement est créé uniquement si vous fournissez un référentiel d’images pour le module. Si vous avez accepté les valeurs localhost par défaut pour tester et déboguer localement, vous n’avez pas besoin de déclarer des variables d’environnement.

- Un fichier **deployment.template.json**, qui répertorie votre nouveau module, ainsi qu’un exemple de module **SimulatedTemperatureSensor** qui simule les données que vous pouvez utiliser à des fins de test. Pour plus d’informations sur le fonctionnement des manifestes de déploiement, consultez [Découvrez comment utiliser des manifestes de déploiement pour déployer des modules et établir des itinéraires](module-composition.md).

Pour savoir comment le module de température simulée fonctionne, afficher le [code source SimulatedTemperatureSensor.csproj](https://github.com/Azure/iotedge/tree/master/edge-modules/SimulatedTemperatureSensor).

## <a name="add-additional-modules"></a>Ajouter des modules supplémentaires

Pour ajouter des modules supplémentaires à votre solution, exécutez la commande **Azure IoT Edge : Ajouter un module IoT Edge** à partir de la palette de commandes. Vous pouvez également cliquer avec le bouton droit sur le dossier **modules** ou sur le fichier `deployment.template.json` dans l'affichage Explorateur de Visual Studio Code, puis sélectionner **Ajouter un module Edge IoT**.

## <a name="develop-your-module"></a>Développer votre module

Le code du module par défaut fourni avec la solution se trouve à l'emplacement suivant :

- Azure Function (C#) : **modules > *&lt;nom de votre module&gt;*  >  *&lt;nom de votre module&gt;* .cs**
- C# : **modules > *&lt;nom de votre module&gt;* > Program.cs**
- Python : **modules > *&lt;nom de votre module&gt;* > main.py**
- Node.js : **modules > *&lt;nom de votre module&gt;* > app.js**
- Java : **modules > *&lt;nom de votre module&gt;* > src > main > java > com > edgemodulemodules > App.java**
- C : **modules > *&lt;nom de votre module&gt;* > main.c**

Le module et le fichier deployment.template.json sont définis de manière à vous permettre de générer la solution, de l’envoyer vers votre registre de conteneurs et de la déployer sur un appareil pour commencer les tests, sans avoir à utiliser de code. Le module est conçu pour récupérer facilement les entrées d’une source (dans ce cas, le module SimulatedTemperatureSensor qui simule des données) et les acheminer vers IoT Hub.

Lorsque vous êtes prêt à personnaliser le modèle avec votre propre code, utilisez les kits [SDK Azure IoT Hub](../iot-hub/iot-hub-devguide-sdks.md) pour générer des modules répondant aux besoins des solutions IoT, tels que la sécurité, la gestion des appareils et la fiabilité.

## <a name="debug-a-module-without-a-container-c-nodejs-java"></a>Déboguer un module sans conteneur (C#, Node.js, Java)

Si vous développez en C#, Node.js ou Java, l'utilisation d'un objet **ModuleClient** est requise dans le code du module par défaut pour démarrer, exécuter et acheminer les messages. Vous devez également utiliser le canal d'entrée par défaut **input1** pour entreprendre une action lorsque le module reçoit des messages.

### <a name="set-up-iot-edge-simulator-for-iot-edge-solution"></a>Configurer un simulateur IoT Edge pour une solution IoT Edge

Sur votre machine de développement, plutôt que d’installer le démon de sécurité IoT Edge, vous pouvez démarrer le simulateur IoT Edge pour exécuter votre solution IoT Edge.

1. Dans l'Explorateur d'appareils de gauche, cliquez sur l'ID de votre appareil IoT Edge, puis sélectionnez **Configurer le simulateur IoT Edge** pour démarrer le simulateur avec la chaîne de connexion de l'appareil.
1. Vous pouvez constater que le simulateur IoT Edge a été correctement configuré en consultant les détails de la progression dans le terminal intégré.

### <a name="set-up-iot-edge-simulator-for-single-module-app"></a>Configurer un simulateur IoT Edge pour une application à module unique

Pour configurer et démarrer le simulateur, exécutez la commande **Azure IoT Edge : Démarrer le simulateur IoT Edge Hub pour module unique** à partir de la palette de commandes de Visual Studio Code. Lorsque vous y êtes invité, utilisez la valeur **input1** du code du module par défaut (ou la valeur équivalente de votre code) comme nom d'entrée pour votre application. La commande déclenche l'interface CLI **iotedgehubdev**, puis démarre le simulateur IoT Edge ainsi qu'un conteneur de modules d'utilitaires de tests. Vous pouvez voir les sorties ci-dessous dans le terminal intégré si le simulateur a été démarré en mode de module unique. Vous pouvez également voir une commande `curl` pour vous aider à envoyer un message. Vous le réutiliserez ultérieurement.

   ![Configurer un simulateur IoT Edge pour une application à module unique](media/how-to-develop-csharp-module/start-simulator-for-single-module.png)

   Dans Visual Studio Code, vous pouvez utiliser l'affichage Explorateur de Docker pour consulter l'état d'exécution du module.

   ![État du module du simulateur](media/how-to-develop-csharp-module/simulator-status.png)

   Le conteneur **edgeHubDev** est la partie principale du simulateur IoT Edge local. Il peut s'exécuter sur votre ordinateur de développement sans le démon de sécurité IoT Edge et fournit des paramètres d'environnement pour votre application de module native ou vos conteneurs de modules. Le conteneur **input** expose les API REST pour contribuer à relier les messages au canal d'entrée cible de votre module.

### <a name="debug-module-in-launch-mode"></a>Déboguer un module en mode de lancement

1. Préparez votre environnement pour le débogage conformément aux exigences de votre langage de développement, définissez un point d'arrêt dans votre module et sélectionnez la configuration de débogage à utiliser :
   - **C#**
     - Dans le terminal intégré de Visual Studio Code, remplacez le répertoire par le dossier **_&lt;nom de votre module&gt;_* _, puis exécutez la commande suivante pour générer l’application .NET Core.

       ```cmd
       dotnet build
       ```

     - Ouvrez le fichier `Program.cs` et ajoutez un point d'arrêt.

     - Accédez à l’affichage Débogage de Visual Studio Code en sélectionnant _*Affichage > Déboguer**. Sélectionnez la configuration de débogage **_ Débogage local de &lt;nom de votre module&gt; (.NET Core) dans la liste déroulante.

        > [!NOTE]
        > Si votre `TargetFramework` .NET Core ne correspond pas au chemin de votre programme dans `launch.json`, vous devrez le mettre à jour manuellement dans `launch.json` pour qu’il corresponde au `TargetFramework` de votre fichier .csproj afin que Visual Studio Code puisse lancer ce programme.

   - **Node.JS**
     - Dans le terminal intégré de Visual Studio Code, remplacez le répertoire par le dossier **_&lt;nom de votre module&gt;_* _, puis exécutez la commande suivante pour installer les packages Node.

       ```cmd
       npm install
       ```

     - Ouvrez le fichier `app.js` et ajoutez un point d'arrêt.

     - Accédez à l’affichage Débogage de Visual Studio Code en sélectionnant _*Affichage > Déboguer**. Sélectionnez la configuration de débogage **_ Débogage local de &lt;nom de votre module&gt; (Node.js) dans la liste déroulante.
   - **Java**
     - Ouvrez le fichier `App.java` et ajoutez un point d'arrêt.

     - Accédez à l'affichage Débogage de Visual Studio Code en sélectionnant **Affichage > Déboguer**. Sélectionnez la configuration de débogage **_&lt;Débogage local de &gt;_ nom de votre module (Java)** dans la liste déroulante.

1. Cliquez sur **Démarrer le débogage** ou appuyez sur **F5** pour démarrer la session de débogage.

1. Dans le terminal intégré de Visual Studio Code, exécutez la commande suivante pour envoyer un message **Hello World** à votre module. Il s'agit de la commande présentée lors des précédentes étapes de configuration du simulateur IoT Edge.

    ```bash
    curl --header "Content-Type: application/json" --request POST --data '{"inputName": "input1","data":"hello world"}' http://localhost:53000/api/v1/messages
    ```

   > [!NOTE]
   > Si vous utilisez Windows, vérifiez que l'interpréteur de commandes du terminal intégré de Visual Studio Code est **Git Bash** ou **WSL Bash**. Vous ne pouvez pas exécuter la commande `curl` à partir de PowerShell ou d'une invite de commandes.
   > [!TIP]
   > Vous pouvez aussi utiliser [PostMan](https://www.getpostman.com/) ou d’autres outils d’API pour envoyer des messages à la place de `curl`.

1. Les variables apparaissent dans le volet de gauche de l'affichage Débogage de Visual Studio Code.

1. Pour arrêter votre session de débogage, sélectionnez le bouton Arrêter ou appuyez sur **MAJ + F5**, puis exécutez **Azure IoT Edge : Arrêter le simulateur IoT Edge** à partir de la palette de commandes afin d'arrêter le simulateur et de procéder au nettoyage.

## <a name="debug-in-attach-mode-with-iot-edge-simulator-c-nodejs-java-azure-functions"></a>Déboguer en mode Association avec le simulateur IoT Edge (C#, Node.js, Java, Azure Functions)

Votre solution par défaut contient deux modules : un module de capteur de température simulé et un module de canal. Le capteur de température simulé envoie des messages au module de canal, puis les messages sont dirigés vers IoT Hub. Le dossier du module que vous avez créé contient plusieurs fichiers Docker pour différents types de conteneur. Utilisez l'un des fichiers se terminant par l'extension **.debug** pour générer votre module à des fins de test.

Actuellement, le débogage en mode Association est uniquement pris en charge comme suit :

- Les modules C#, y compris ceux d'Azure Functions, prennent en charge le débogage dans les conteneurs Linux amd64
- Les modules Node.js prennent en charge le débogage dans les conteneurs Linux amd64 et arm32v7, ainsi que dans les conteneurs Windows amd64
- Les modules Java prennent en charge le débogage dans les conteneurs Linux amd64 et arm32v7

> [!TIP]
> Vous pouvez basculer d'une option à l'autre pour la plateforme par défaut de votre solution IoT Edge en cliquant sur l'élément correspondant sur la barre d'état de Visual Studio Code.

### <a name="set-up-iot-edge-simulator-for-iot-edge-solution"></a>Configurer un simulateur IoT Edge pour une solution IoT Edge

Sur votre ordinateur de développement, plutôt que d'installer le démon de sécurité IoT Edge, vous pouvez démarrer le simulateur IoT Edge pour exécuter votre solution IoT Edge.

1. Dans l'Explorateur d'appareils de gauche, cliquez sur l'ID de votre appareil IoT Edge, puis sélectionnez **Configurer le simulateur IoT Edge** pour démarrer le simulateur avec la chaîne de connexion de l'appareil.

1. Vous pouvez constater que le simulateur IoT Edge a été correctement configuré en consultant les détails de la progression dans le terminal intégré.

### <a name="build-and-run-container-for-debugging-and-debug-in-attach-mode"></a>Générer et exécuter le conteneur pour le débogage et déboguer en mode d’attachement

1. Ouvrez le fichier de votre module (`Program.cs`, `app.js`, `App.java` ou `<your module name>.cs`) et ajoutez un point d'arrêt.

1. Dans l'affichage Explorateur de Visual Studio Code, cliquez sur le fichier `deployment.debug.template.json` de votre solution, puis sélectionnez **Générer et exécuter la solution IoT Edge dans le simulateur**. Vous pouvez surveiller tous les journaux d’activité du conteneur de module dans la même fenêtre. Vous pouvez aussi accéder à l'affichage Docker pour surveiller l'état du conteneur.

   ![Surveiller des variables](media/how-to-vs-code-develop-module/view-log.png)

1. Accédez à l'affichage Débogage de Visual Studio Code, puis sélectionnez le fichier de configuration du débogage de votre module. Le nom de l'option de débogage doit être semblable à **_&lt;Débogage distant de &gt;_ nom de votre module**

1. Sélectionnez **Démarrer le débogage** ou appuyez sur **F5**. Sélectionnez le processus à attacher.

1. Les variables apparaissent dans le volet de gauche de l'affichage Débogage de Visual Studio Code.

1. Pour arrêter la session de débogage, sélectionnez le bouton Arrêter ou appuyez sur **MAJ + F5**, puis sélectionnez **Azure IoT Edge : Arrêter le simulateur IoT Edge** à partir de la palette de commandes.

> [!NOTE]
> L'exemple précédent montre comment déboguer des modules IoT Edge sur des conteneurs. Il a permis d'ajouter des ports exposés aux paramètres `createOptions` du conteneur de votre module. Une fois le débogage de vos modules terminé, nous vous recommandons de supprimer ces ports exposés pour les modules IoT Edge prêts à entrer en production.
>
> Pour les modules écrits en C#, y compris Azure Functions, cet exemple repose sur la version de débogage de `Dockerfile.amd64.debug`, qui inclut le débogueur en ligne de commande .NET Core (VSDBG) dans l'image de votre conteneur lors de sa création. Une fois le débogage de vos modules C# terminé, nous vous recommandons d’utiliser directement le fichier Dockerfile sans VSDBG pour les modules IoT Edge prêts pour la production.

## <a name="debug-a-module-with-the-iot-edge-runtime"></a>Déboguer un module avec le runtime IoT Edge

Chaque dossier du module contient plusieurs fichiers Docker pour différents types de conteneurs. Utilisez l'un des fichiers se terminant par l'extension **.debug** pour générer votre module à des fins de test.

Lors du débogage de modules à l’aide de cette méthode, vos modules s’exécutent par-dessus le runtime IoT Edge. L’appareil IoT Edge et Visual Studio Code peuvent se trouver sur la même machine. Cependant, Visual Studio Code s’exécute généralement sur la machine de développement, tandis que le runtime et les modules IoT Edge s’exécutent sur une autre machine physique. Pour effectuer un débogage dans Visual Studio Code, vous devez :

- Configurer votre appareil IoT Edge, créer un ou plusieurs modules IoT Edge avec le Dockerfile **.debug**, puis déployer sur un appareil IoT Edge
- Exposer l’adresse IP et le port du module pour que le débogueur puisse être joint
- Mettre à jour `launch.json` afin que Visual Studio Code puisse être joint au processus dans le conteneur présent sur la machine distante. Ce fichier se trouve dans le dossier `.vscode` de votre espace de travail. Il est mis à jour chaque fois que vous ajoutez un module qui prend en charge le débogage.

### <a name="build-and-deploy-your-module-to-the-iot-edge-device"></a>Générer et déployer le module sur l’appareil IoT Edge

1. Dans Visual Studio Code, ouvrez le fichier `deployment.debug.template.json`, qui contient la version de débogage des images de vos modules avec les valeurs `createOptions` appropriées.

1. Si vous développez votre module en Python, procédez comme suit avant de continuer :
   - Ouvrez le fichier `main.py` et ajoutez ce code après la section d'importation :

      ```python
      import ptvsd
      ptvsd.enable_attach(('0.0.0.0',  5678))
      ```

   - Ajoutez la ligne de code unique suivante au rappel que vous souhaitez déboguer :

      ```python
      ptvsd.break_into_debugger()
      ```

     Par exemple, si vous souhaitez déboguer la fonction `receive_message_listener`, insérez cette ligne de code comme indiqué ci-dessous :

      ```python
      def receive_message_listener(client):
          ptvsd.break_into_debugger()
          global RECEIVED_MESSAGES
          while True:
              message = client.receive_message_on_input("input1")   # blocking call
              RECEIVED_MESSAGES += 1
              print("Message received on input1")
              print( "    Data: <<{}>>".format(message.data) )
              print( "    Properties: {}".format(message.custom_properties))
              print( "    Total calls received: {}".format(RECEIVED_MESSAGES))
              print("Forwarding message to output1")
              client.send_message_to_output(message, "output1")
              print("Message successfully forwarded")
      ```

1. Dans la palette de commandes de Visual Studio Code :
   1. Exécutez la commande **Azure IoT Edge : Générer et envoyer (push) la solution IoT Edge**.

   1. Sélectionnez le fichier `deployment.debug.template.json` de votre solution.

1. Dans la section **Appareils Azure IoT Hub** de l'affichage Explorateur de Visual Studio Code :
   1. Cliquez avec le bouton droit sur l’ID de l’appareil IoT Edge, puis sélectionnez **Créer un déploiement pour un seul appareil**.

      > [!TIP]
      > Pour vous assurer que l'appareil choisi est un appareil IoT Edge, sélectionnez-le afin de développer la liste des modules et vérifiez la présence de **$edgeHub** et **$edgeAgent**. Chaque appareil IoT Edge inclut ces deux modules.

   1. Accédez au dossier **config** de votre solution, sélectionnez le fichier `deployment.debug.amd64.json`, puis choisissez **Sélectionner un manifeste de déploiement Edge**.

Vous pouvez alors constater que le déploiement a bien été créé avec un ID de déploiement dans le terminal intégré.

Vous pouvez vérifier l’état de votre conteneur en exécutant la commande `docker ps` dans le terminal. Si Visual Studio Code et le runtime IoT Edge s’exécutent sur la même machine, vous pouvez également vérifier l’état dans la vue Docker de Visual Studio Code.

### <a name="expose-the-ip-and-port-of-the-module-for-the-debugger"></a>Exposer l’adresse IP et le port du module pour le débogueur

Vous pouvez ignorer cette section si vos modules sont exécutés sur la même machine que Visual Studio Code, puisque vous utilisez localhost pour la jonction au conteneur et que vous avez déjà les paramètres de port appropriés dans le Dockerfile **.debug**, les paramètres `createOptions` du module du conteneur et le fichier `launch.json`. Si vos modules et Visual Studio Code ne s’exécutent pas sur la même machine, suivez les étapes relatives à votre langage de développement.

- **C#, y compris Azure Functions**

  [Configurez le canal SSH sur votre machine de développement et un appareil IoT Edge](https://github.com/OmniSharp/omnisharp-vscode/wiki/Attaching-to-remote-processes), puis modifiez le fichier `launch.json` à joindre.

- **Node.JS**

  - Vérifiez que le module à déboguer sur la machine est en cours d’exécution, qu’il est prêt pour la jonction des débogueurs et que le port 9229 est accessible de l’extérieur. Pour cela, ouvrez `http://<target-machine-IP>:9229/json` sur la machine du débogueur. Cette URL doit afficher des informations sur le module Node.js à déboguer.
  
  - Sur la machine de développement, ouvrez Visual Studio Code, puis modifiez `launch.json` de sorte que la valeur d’adresse du profil **_&lt;nom de votre module&gt;_ Remote Debug (Node.js)** (ou du profil **_&lt;nom de votre module&gt;_ Remote Debug (Node.js dans Windows Container)** si le module est exécuté en tant que conteneur Windows) corresponde à l’adresse IP de la machine déboguée.

- **Java**

  - Créez un tunnel SSH vers la machine à déboguer en exécutant `ssh -f <username>@<target-machine> -L 5005:127.0.0.1:5005 -N`.
  
  - Sur votre machine de développement, ouvrez Visual Studio Code, puis modifiez le profil **_&lt;nom de votre module&gt;_ Remote Debug (Java)** dans `launch.json` de manière à pouvoir effectuer la jonction à la machine cible. Pour plus d’informations sur la modification de `launch.json` et sur le débogage Java avec Visual Studio Code, consultez la section sur la [configuration du débogueur](https://code.visualstudio.com/docs/java/java-debugging#_configuration).

- **Python**

  - Vérifiez que le port 5678 de la machine à déboguer est ouvert et accessible.

  - Dans le code `ptvsd.enable_attach(('0.0.0.0', 5678))` que vous avez inséré précédemment dans `main.py`, remplacez **0.0.0.0** par l’adresse IP de la machine à déboguer. Générez, envoyez (push) et déployez à nouveau votre module IoT Edge.

  - Sur votre machine de développement, ouvrez Visual Studio Code, puis modifiez `launch.json` de sorte que la valeur `host` du profil **_&lt;nom de votre module&gt;_ Remote Debug (Python)** utilise l’adresse IP de la machine cible au lieu de `localhost`.

### <a name="debug-your-module"></a>Déboguer votre module

1. Dans l'affichage Débogage de Visual Studio Code, sélectionnez le fichier de configuration du débogage de votre module. Le nom de l'option de débogage doit être semblable à **_&lt;Débogage distant de &gt;_ nom de votre module**

1. Ouvrez le fichier du module correspondant à votre langage de développement et ajoutez un point d'arrêt :

   - **Azure Function (C#)**  : Ajoutez votre point d’arrêt au fichier `<your module name>.cs`.
   - **C#**  : Ajoutez votre point d’arrêt au fichier `Program.cs`.
   - **Node.JS** : Ajoutez votre point d’arrêt au fichier `app.js`.
   - **Java** : Ajoutez votre point d’arrêt au fichier `App.java`.
   - **Python** : Ajoutez votre point d’arrêt au fichier `main.py` dans la méthode de rappel à l’emplacement où vous avez ajouté la ligne `ptvsd.break_into_debugger()`.
   - **C** : Ajoutez votre point d’arrêt au fichier `main.c`.

1. Sélectionnez **Démarrer le débogage** ou **F5**. Sélectionnez le processus à attacher.

1. Les variables apparaissent dans le volet de gauche de l'affichage Débogage de Visual Studio Code.

> [!NOTE]
> L'exemple précédent montre comment déboguer des modules IoT Edge sur des conteneurs. Il a permis d'ajouter des ports exposés aux paramètres `createOptions` du conteneur de votre module. Une fois le débogage de vos modules terminé, nous vous recommandons de supprimer ces ports exposés pour les modules IoT Edge prêts à entrer en production.

## <a name="build-and-debug-a-module-remotely"></a>Générer et déboguer un module à distance

Grâce aux récentes modifications apportées aux moteurs Docker et Moby pour prendre en charge les connexions SSH, ainsi qu'au nouveau paramètre Azure IoT Tools qui permet d'injecter des paramètres d'environnement dans la palette de commandes Visual Studio Code et dans les terminaux Azure IoT Edge, vous pouvez désormais générer et déboguer des modules sur des appareils distants.

Consultez ce [billet de blog destiné aux développeurs IoT](https://devblogs.microsoft.com/iotdev/easily-build-and-debug-iot-edge-modules-on-your-remote-device-with-azure-iot-edge-for-vs-code-1-9-0/) pour obtenir plus d'informations et des instructions pas à pas.

## <a name="next-steps"></a>Étapes suivantes

Une fois votre module généré, découvrez comment [déployer des modules Azure IoT Edge à partir de Visual Studio Code](how-to-deploy-modules-vscode.md).

Pour développer des modules pour vos appareils IoT Edge, consultez [Comprendre et utiliser les kits Azure IoT Hub SDK](../iot-hub/iot-hub-devguide-sdks.md).