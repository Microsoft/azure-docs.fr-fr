---
title: Tutoriel - Développer un module pour les appareils Linux à l’aide d’Azure IoT Edge
description: Ce tutoriel vous aide à configurer votre machine de développement et vos ressources cloud afin de développer des modules IoT Edge à l’aide de conteneurs Linux pour des appareils Linux
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 07/30/2020
ms.topic: tutorial
ms.service: iot-edge
services: iot-edge
ms.custom: mvc
ms.openlocfilehash: b352bd92ecc69ca68a6870d3a59ef5e0cdd1daba
ms.sourcegitcommit: fec60094b829270387c104cc6c21257826fccc54
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96920853"
---
# <a name="tutorial-develop-iot-edge-modules-for-linux-devices"></a>Tutoriel : Développer des modules IoT Edge pour les appareils Linux

Utilisez Visual Studio Code pour développer et déployer du code sur des appareils Linux exécutant IoT Edge.

Dans le guide de démarrage rapide, vous avez créé un appareil IoT Edge à l’aide d’une machine virtuelle Linux et déployé un module à partir de la Place de marché Azure. Ce tutoriel vous montre pas à pas comment développer et déployer votre propre code sur un appareil IoT Edge. Cet article constitue un bon préambule aux autres tutoriels qui abordent plus en détail des langages de programmation ou des services Azure spécifiques.

Ce tutoriel utilise l’exemple de déploiement d’un **module C# sur un appareil Linux**. Cet exemple a été choisi, car il s’agit du scénario le plus courant pour les développeurs de solutions IoT Edge. Même si vous prévoyez d’utiliser un autre langage ou de déployer un service Azure, ce didacticiel est également utile pour en savoir plus sur les outils de développement et les concepts. Suivez cette introduction au processus de développement, puis choisissez votre langage ou service Azure préféré pour vous plonger dans les détails.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> * Configurer votre machine de développement
> * Utiliser les outils IoT Edge pour Visual Studio Code afin de créer un projet
> * Générer votre projet sous forme de conteneur et le stocker dans un registre de conteneurs Azure
> * Déployer votre code sur un appareil IoT Edge

## <a name="prerequisites"></a>Prérequis

Une machine de développement :

* Vous pouvez utiliser votre propre ordinateur ou une machine virtuelle, selon vos préférences de développement.
  * Vérifiez que votre ordinateur de développement prend bien en charge la virtualisation imbriquée. Cette fonctionnalité est nécessaire à l’exécution du moteur de conteneur que vous allez installer dans la section suivante.
* La plupart des systèmes d’exploitation qui exécutent un moteur de conteneur peuvent servir à développer des modules IoT Edge pour appareils Linux. Ce tutoriel utilise un ordinateur Windows, mais souligne des différences connues sur macOS ou Linux.
* Installez [Git](https://git-scm.com/), en vue d’extraire des packages de modèle de module plus loin dans ce tutoriel.  
* [Extension C# pour Visual Studio Code (développée par OmniSharp)](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp).
* [SDK .NET Core 2.1](https://www.microsoft.com/net/download).

Un appareil Azure IoT Edge sur Linux :

* Nous vous déconseillons d’exécuter IoT Edge sur votre machine de développement ; utilisez un appareil distinct à la place. Cette distinction entre machine de développement et appareil IoT Edge reflète plus fidèlement un scénario de déploiement réel et aide à assimiler les différents concepts.
* Si vous ne disposez pas d’un deuxième appareil, suivez l’article de démarrage rapide permettant de créer un appareil IoT Edge dans Azure avec une [machine virtuelle Linux](quickstart-linux.md).

Ressources cloud :

* Un [hub IoT](../iot-hub/iot-hub-create-through-portal.md) de niveau gratuit ou standard dans Azure.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="key-concepts"></a>Concepts clés

Ce tutoriel présente le développement d’un module IoT Edge. Un *module IoT Edge*, ou parfois simplement *module* dans sa forme abrégée, est un conteneur avec du code exécutable. Vous pouvez déployer un ou plusieurs modules sur un appareil IoT Edge. Les modules effectuent des tâches spécifiques comme l’ingestion de données provenant de capteurs, le nettoyage et l’analyse de données ou encore l’envoi de messages à un hub IoT. Pour plus d’informations, consultez [Présentation des modules Azure IoT Edge](iot-edge-modules.md).

Lors du développement d’un module IoT Edge, il est important de comprendre la différence entre la machine de développement et l’appareil IoT Edge cible où le module sera finalement déployé. Le conteneur que vous générez pour le stockage du code du module doit correspondre au système d’exploitation (OS) de l’*appareil cible*. Par exemple, le scénario le plus courant est le développement d’un module sur un ordinateur Windows avec l’intention de cibler un appareil Linux exécutant IoT Edge. Dans ce cas, le système d’exploitation du conteneur est Linux. À mesure que vous parcourez ce tutoriel, gardez à l’esprit la différence entre *système d’exploitation de la machine de développement* et *système d’exploitation du conteneur*.

Ce tutoriel cible les appareils Linux exécutant IoT Edge. Vous pouvez utiliser le système d’exploitation de votre choix, à condition que votre machine de développement exécute des conteneurs Linux. Visual Studio Code étant recommandé pour développer pour les appareils Linux, nous allons l’utiliser dans le cadre de ce tutoriel. Vous pouvez aussi bien utiliser Visual Studio, bien qu’il existe des différences de prise en charge entre les deux outils.

Le tableau suivant liste les scénarios de développement pris en charge pour des **conteneurs Linux** dans Visual Studio Code et Visual Studio.

|   | Visual Studio Code | Visual Studio 2017/2019 |
| - | ------------------ | ------------------ |
| **Architecture d’appareil Linux** | Linux AMD64 <br> Linux ARM32 | Linux AMD64 <br> Linux ARM32 |
| **Services Azure** | Azure Functions <br> Azure Stream Analytics <br> Azure Machine Learning |   |
| **Langues** | C <br> C# <br> Java <br> Node.js <br> Python | C <br> C# |
| **Plus d’informations** | [Azure IoT Edge pour Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-edge) | [Outils Azure IoT Edge pour Visual Studio 2017](https://marketplace.visualstudio.com/items?itemName=vsc-iot.vsiotedgetools) <br> [Outils Azure IoT Edge pour Visual Studio 2019](https://marketplace.visualstudio.com/items?itemName=vsc-iot.vs16iotedgetools) |

>[!NOTE]
>La prise en charge des appareils Linux ARM64 est disponible en [préversion publique](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Pour plus d’informations, consultez [Développer et déboguer des modules IoT Edge ARM64 dans Visual Studio Code (préversion)](https://devblogs.microsoft.com/iotdev/develop-and-debug-arm64-iot-edge-modules-in-visual-studio-code-preview).

Ce tutoriel décrit les étapes de développement pour Visual Studio Code. Si vous préférez utiliser Visual Studio, reportez-vous aux instructions dans [Utiliser Visual Studio 2019 pour développer et déboguer des modules pour Azure IoT Edge](how-to-visual-studio-develop-module.md).
## <a name="install-container-engine"></a>Installer le moteur de conteneur

Les modules IoT Edge étant empaquetés en tant que conteneurs, vous avez besoin d’un moteur de conteneur sur votre machine de développement pour les générer et les gérer. Nous vous recommandons d’utiliser Docker Desktop pour le développement en raison de sa prise en charge des fonctionnalités et de sa popularité. Docker Desktop sur Windows vous permet de basculer entre les conteneurs Linux et Windows, et donc de développer aisément des modules pour différents types d’appareils IoT Edge.

Utilisez la documentation Docker pour effectuer les différentes procédures d’installation sur votre machine de développement :

* [Installer Docker Desktop pour Windows](https://docs.docker.com/docker-for-windows/install/)

  * Quand vous installez Docker Desktop pour Windows, vous êtes invité à indiquer si vous souhaitez utiliser des conteneurs Linux ou Windows. Vous pouvez revenir sur votre décision à tout moment à l’aide d’un simple commutateur. Pour ce tutoriel, nous utilisons des conteneurs Linux, car nos modules ciblent les appareils Linux. Pour plus d’informations, consultez [Switch between Windows and Linux containers](https://docs.docker.com/docker-for-windows/#switch-between-windows-and-linux-containers).

* [Installer Docker Desktop pour Mac](https://docs.docker.com/docker-for-mac/install/)

* Lisez [About Docker CE](https://docs.docker.com/install/) pour obtenir des informations sur l’installation sur plusieurs plateformes Linux.
  * Pour le Sous-système Windows pour Linux (WSL), installez Docker Desktop pour Windows.

## <a name="set-up-vs-code-and-tools"></a>Configurer VS Code et les outils

Utilisez les extensions IoT pour Visual Studio Code pour développer des modules IoT Edge. Ces extensions fournissent des modèles de projet, automatisent la création du manifeste de déploiement et vous permettent de superviser et de gérer des appareils IoT Edge. Dans cette section, vous installez Visual Studio Code et l’extension IoT, puis configurez votre compte Azure pour gérer les ressources IoT Hub à partir de Visual Studio Code.

1. Installez [Visual Studio Code](https://code.visualstudio.com/) sur votre machine de développement.

2. Une fois l’installation terminée, sélectionnez **Affichage** > **Extensions**.

3. Recherchez **Azure IoT Tools**, qui est en fait une collection d’extensions qui vous aident à interagir avec les appareils IoT et IoT Hub ainsi qu’à développer des modules IoT Edge.

4. Sélectionnez **Installer**. Chaque extension incluse s’installe individuellement.

5. Une fois les extensions installées, ouvrez la palette de commandes en sélectionnant **Affichage** > **Palette de commandes**.

6. Dans la palette de commandes, recherchez et sélectionnez **Azure : Sign in**. Suivez les instructions de l’invite pour vous connecter à votre compte Azure.

7. De nouveau dans la palette de commandes, recherchez et sélectionnez **Azure IoT Hub: Select IoT Hub** (Azure IoT Hub : Sélectionner IoT Hub). Suivez les invites pour sélectionner votre abonnement Azure et un hub IoT.

8. Ouvrez la section Explorateur de Visual Studio Code en sélectionnant l’icône dans la barre d’activités sur la gauche, ou en sélectionnant **Affichage** > **Explorateur**.

9. En bas de la section Explorateur, développez le menu **Azure IoT Hub / Appareils** réduit. Vous devez voir les appareils et les appareils IoT Edge associés au hub IoT que vous avez sélectionné via la palette de commandes.

   ![Afficher les appareils dans votre hub IoT](./media/tutorial-develop-for-linux/view-iot-hub-devices.png)

[!INCLUDE [iot-edge-create-container-registry](../../includes/iot-edge-create-container-registry.md)]

## <a name="create-a-new-module-project"></a>Créer un projet de module

L’extension Azure IoT Tools fournit des modèles de projet pour tous les langages de module IoT Edge pris en charge dans Visual Studio Code. Ces modèles ont tous les fichiers et tout le code dont vous avez besoin pour déployer un module opérationnel afin de tester IoT Edge. Vous pouvez également vous servir de ces modèles comme point de départ et les personnaliser avec votre propre logique métier.

Pour ce didacticiel, nous utilisons le modèle de module C#, car il s’agit du modèle le plus couramment utilisé.

### <a name="create-a-project-template"></a>Créer un modèle de projet

Dans la palette de commandes Visual Studio Code, recherchez et sélectionnez **Azure IoT Edge: New IoT Edge solution** (Azure IoT Edge : Nouvelle solution IoT Edge). Suivez les invites et utilisez les valeurs suivantes pour créer votre solution :

   | Champ | Valeur |
   | ----- | ----- |
   | Sélectionner le dossier | Choisissez l’emplacement sur votre machine de développement pour que VS Code crée les fichiers de la solution. |
   | Provide a solution name (Nommer la solution) | Entrez un nom descriptif pour votre solution ou acceptez le nom par défaut (**EdgeSolution**). |
   | Select module template (Sélectionner un modèle de module) | Choisissez **Module C#** . |
   | Provide a module name (Nommer le module) | Acceptez la valeur par défaut **SampleModule**. |
   | Provide Docker image repository for the module (Indiquer le référentiel d’images Docker pour le module) | Un référentiel d’images comprend le nom de votre registre de conteneurs et celui de votre image conteneur. L’image conteneur est préremplie avec le nom que vous avez indiqué à la dernière étape. Remplacez **localhost:5000** par la valeur de **Serveur de connexion** provenant de votre registre de conteneurs Azure. Vous pouvez récupérer la valeur du serveur de connexion à partir de la page Vue d’ensemble de votre registre de conteneurs dans le portail Azure. <br><br> Le référentiel d’images final se présente comme ceci :\<registry name\>.azurecr.io/samplemodule. |

   ![Fourniture du référentiel d’images Docker](./media/tutorial-develop-for-linux/image-repository.png)

Une fois votre nouvelle solution chargée dans la fenêtre Visual Studio Code, prenez un moment pour vous familiariser avec les fichiers qu’elle a créés :

* Le dossier **.vscode** contient un fichier appelé **launch.json**, qui est utilisé pour le débogage des modules.
* Le dossier **modules** contient un dossier pour chaque module dans votre solution. À ce stade, il doit s’agir de **SampleModule** ou du nom que vous avez donné au module. Le dossier SampleModule contient le code de programme principal, les métadonnées du module et plusieurs fichiers Docker.
* Le fichier **.env** contient les informations d’identification liées à votre registre de conteneurs. Ces informations d’identification sont partagées avec votre appareil IoT Edge afin qu’il dispose d’un accès pour extraire les images de conteneur.
* Les fichiers **deployment.debug.template.json** et **deployment.template.json** sont des modèles qui vous aident à créer un manifeste de déploiement. Un *manifeste de déploiement* est un fichier qui définit exactement les modules à déployer sur un appareil, leur configuration et la façon dont ils peuvent communiquer entre eux et avec le cloud. Les fichiers de modèle utilisent des pointeurs pour certaines valeurs. Quand vous transformez le modèle en un manifeste de déploiement réel, les pointeurs sont remplacés par des valeurs provenant d’autres fichiers solution. Recherchez les deux espaces réservés communs dans votre modèle de déploiement :

  * Dans la section des informations d’identification du registre, l’adresse est automatiquement remplie à partir des informations que vous avez fournies quand vous avez créé la solution. Toutefois, le nom d’utilisateur et le mot de passe référencent les variables stockées dans le fichier .env. Cette configuration est une mesure de sécurité, car le fichier .env est ignoré par git, contrairement au modèle de déploiement.
  * Dans la section SampleModule, l’image conteneur n’est pas renseignée même si vous avez fourni le dépôt d’images quand vous avez créé la solution. Cet espace réservé pointe vers le fichier **module.json** dans le dossier SampleModule. Si vous accédez à ce fichier, vous verrez que le champ d’image contient le dépôt, mais également une valeur d’étiquette constituée de la version et de la plateforme du conteneur. Vous pouvez itérer manuellement la version dans le cadre de votre cycle de développement, et vous sélectionnez la plateforme de conteneur à l’aide d’un sélecteur que nous présentons plus loin dans cette section.

### <a name="provide-your-registry-credentials-to-the-iot-edge-agent"></a>Fournir vos informations d’identification de registre à l’agent IoT Edge

Le fichier d’environnement stocke les informations d’identification de votre registre de conteneurs et les partage avec le runtime IoT Edge. Le runtime a besoin de ces informations d’identification pour extraire vos images de conteneur sur l’appareil IoT Edge.

>[!NOTE]
>Si vous n’avez pas remplacé la valeur **localhost:5000** par la valeur du serveur de connexion de votre registre de conteneurs Azure, dans l’étape [**Créer un modèle de projet**](#create-a-project-template), le fichier **.env** et la section registryCredentials du manifeste de déploiement seront manquants. 

L’extension IoT Edge tente d’extraire d’Azure vos informations d’identification de registre de conteneurs et de les insérer dans le fichier d’environnement. Vérifiez si vos informations d’identification sont déjà incluses. Si ce n’est pas le cas, veuillez les ajouter maintenant :

1. Ouvrez le fichier **.env** dans votre solution de module.
2. Ajoutez les valeurs de **nom d’utilisateur** et de **mot de passe** que vous avez copiées à partir de votre registre de conteneurs Azure.
3. Enregistrez les modifications apportées au fichier .env.

### <a name="select-your-target-architecture"></a>Sélectionner votre architecture cible

Visual Studio Code peut développer des modules C# pour les appareils Linux AMD64 et ARM32v7. Vous devez sélectionner l’architecture que vous ciblez avec chaque solution, car cela affecte la façon dont le conteneur est généré et exécuté. Linux AMD64 est la valeur par défaut.

1. Ouvrez la palette de commandes et recherchez **Azure IoT Edge: Set Default Target Platform for Edge Solution** (Azure IoT Edge : définir la plateforme cible par défaut pour la solution Edge), ou sélectionnez l’icône de raccourci dans la barre latérale en bas de la fenêtre.

   ![Sélectionner l’icône de l’architecture dans la barre latérale](./media/tutorial-develop-for-linux/select-architecture.png)

2. Dans la palette de commandes, sélectionnez l’architecture cible dans la liste des options. Pour ce tutoriel, comme nous utilisons une machine virtuelle Ubuntu en tant qu’appareil IoT Edge, nous allons conserver la valeur par défaut **amd64**.

### <a name="review-the-sample-code"></a>Réviser l’exemple de code

Le modèle de solution que vous avez créé inclut un exemple de code pour un module IoT Edge. Cet exemple de module reçoit simplement des messages, puis les transfère. La fonctionnalité de pipeline illustre un concept important dans IoT Edge, à savoir la façon dont les modules communiquent entre eux.

Le code de chaque module peut déclarer plusieurs files d’attente d’*entrée* et de *sortie*. Le hub IoT Edge en cours d’exécution sur l’appareil route les messages depuis la sortie d’un module vers l’entrée d’un ou de plusieurs modules. Le code utilisé pour la déclaration des entrées et des sorties varie, mais le concept est le même pour tous les modules. Pour plus d’informations sur le routage entre des modules, consultez [Déclarer des routes](module-composition.md#declare-routes).

L’exemple de code C# qui est fourni avec le modèle de projet utilise la [classe ModuleClient](/dotnet/api/microsoft.azure.devices.client.moduleclient) du SDK du IoT Hub pour .NET.

1. Ouvrez le fichier **Program.cs**, qui se trouve dans le dossier **modules/SampleModule/** .

2. Dans program.cs, recherchez la méthode **SetInputMessageHandlerAsync**.

3. La méthode [SetInputMessageHandlerAsync](/dotnet/api/microsoft.azure.devices.client.moduleclient.setinputmessagehandlerasync) configure une file d’attente d’entrée pour la réception des messages entrants. Passez en revue cette méthode pour voir comment elle lance une file d’attente d’entrée appelée **input1**.

   ![Rechercher le nom d’entrée dans le constructeur SetInputMessageCallback](./media/tutorial-develop-for-linux/declare-input-queue.png)

4. Recherchez ensuite la méthode **SendEventAsync**.

5. La méthode [SendEventAsync](/dotnet/api/microsoft.azure.devices.client.moduleclient.sendeventasync) traite les messages reçus et configure une file d’attente de sortie pour les transmettre. Passez en revue cette méthode pour voir qu’elle lance une file d’attente de sortie appelée **output1**.

   ![Rechercher le nom de sortie dans SendEventToOutputAsync](./media/tutorial-develop-for-linux/declare-output-queue.png)

6. Ouvrez le fichier **deployment.template.json**.

7. Recherchez la propriété **modules** des propriétés souhaitées $edgeAgent.

   Deux modules doivent y être listés. Le premier est le module **SimulatedTemperatureSensor**, inclus par défaut dans tous les modèles pour fournir des données de température simulée que vous pouvez utiliser pour tester vos modules. L’autre est le module **SampleModule** que vous avez créé dans le cadre de cette solution.

8. En bas du fichier, recherchez les propriétés souhaitées pour le module **$edgeHub**.

   Une des fonctions du module de hub IoT Edge consiste à router les messages entre tous les modules dans un déploiement. Examinez les valeurs dans la propriété **routes**. La route **SampleModuleToIoTHub** utilise un caractère générique (* *\** _) pour indiquer tous les messages provenant des files d’attente de sortie dans le module SampleModule. Ces messages sont passés à _$upstream*, nom réservé pour IoT Hub. L’autre route, **sensorToSampleModule**, prend les messages en provenance du module SimulatedTemperatureSensor et les route vers la file d’attente d’entrée *input1* que vous avez vu initialisée dans le code SampleModule.

   ![Examiner les routes dans deployment.template.json](./media/tutorial-develop-for-linux/deployment-routes.png)

## <a name="build-and-push-your-solution"></a>Générer et envoyer (push) votre solution

Vous avez passé en revue le code du module et le modèle de déploiement pour comprendre certains concepts de déploiement clés. Maintenant, vous êtes prêt à générer l’image conteneur SampleModule et à l’envoyer (push) vers votre registre de conteneurs. Avec l’extension IoT Tools pour Visual Studio Code, cette étape génère également le manifeste de déploiement basé sur les informations contenues dans le fichier de modèle et les informations de module des fichiers solution.

### <a name="sign-in-to-docker"></a>Se connecter à Docker

Fournissez les informations d’identification du registre de conteneurs à Docker afin qu’il puisse envoyer (push) votre image conteneur à stocker dans le registre.

1. Ouvrez le terminal intégré Visual Studio Code en sélectionnant **Affichage** > **Terminal**.

2. Connectez-vous à Docker avec les informations d’identification du registre de conteneurs Azure que vous avez enregistré après avoir créé le registre.

   ```cmd/sh
   docker login -u <ACR username> -p <ACR password> <ACR login server>
   ```

   Il se peut que vous receviez un avertissement de sécurité recommandant d’utiliser `--password-stdin`. Bien qu’il s’agisse de la bonne pratique recommandée pour les scénarios de production, elle n’est pas pertinente pour ce tutoriel. Pour plus d’informations, consultez les informations de référence sur [docker login](https://docs.docker.com/engine/reference/commandline/login/#provide-a-password-using-stdin).
   
3. Connexion à Azure Container Registry

   ```cmd/sh
   az acr login -n <ACR registry name>
   ```

### <a name="build-and-push"></a>Générer et envoyer (push)

Visual Studio Code ayant désormais accès à votre registre de conteneurs, il est temps de transformer le code de la solution en image conteneur.

1. Dans l’Explorateur Visual Studio Code, cliquez avec le bouton droit sur le fichier **deployment.template.json** et sélectionnez **Build and Push IoT Edge Solution** (Générer et envoyer (push) la solution IoT Edge).

   ![Générer et envoyer (push) les modules IoT Edge](./media/tutorial-develop-for-linux/build-and-push-modules.png)

   La commande de génération et d’envoi (push) déclenche trois opérations. Tout d’abord, elle crée un dossier dans la solution appelé **config** contenant les manifestes de déploiement en entier. Il est généré à partir des informations dans le modèle de déploiement et d’autres fichiers de solution. Ensuite, elle exécute `docker build` pour générer l’image de conteneur basée sur le fichier docker correspondant à votre architecture cible. Puis, elle exécute `docker push` pour envoyer (push) le dépôt d’images vers votre registre de conteneurs.

   Ce processus peut prendre plusieurs minutes la première fois, mais il est plus rapide la prochaine fois que vous exécutez les commandes.

2. Ouvrez le fichier **deployment.amd64.json** dans le dossier config créé. Le nom de fichier reflétant l’architecture cible, il est différent si vous avez choisi une autre architecture.

3. Notez que les deux paramètres qui avaient des espaces réservés sont maintenant remplis avec leurs valeurs appropriées. Dans la section **registryCredentials**, les nom d’utilisateur et mot de passe du registre proviennent du fichier .env. Le **SampleModule** a le dépôt d’images complet ; l’étiquette du nom, de la version et de l’architecture provient du fichier module.json.

4. Ouvrez le fichier **module.json** dans le dossier SampleModule.

5. Changez le numéro de version de l’image de module. (La version, pas $schema-version.) Par exemple, incrémentez le numéro de version de correctif en le définissant sur **0.0.2** comme si nous avions apporté une petite correction dans le code du module.

   >[!TIP]
   >Les versions de module activent la gestion de version, ce qui vous permet de tester les modifications sur un petit ensemble d’appareils avant de déployer des mises à jour en production. Si vous n’incrémentez pas la version du module avant d’effectuer les opérations de génération et d’envoi (push), vous remplacez le dépôt dans votre registre de conteneurs.

6. Enregistrez les modifications apportées au fichier module.json.

7. Recliquez avec le bouton droit sur le fichier **deployment.template.json** et resélectionnez **Build and Push IoT Edge Solution** (Générer et envoyer (push) la solution IoT Edge).

8. Rouvrez le fichier **deployment.amd64.json**. Notez qu’aucun fichier n’a été créé quand vous avez réexécuté la commande de génération et d’envoi (push). Au lieu de cela, le même fichier a été mis à jour pour refléter les modifications. L’image SampleModule pointe désormais vers la version 0.0.2 du conteneur.

9. Pour vérifier davantage ce qu’a fait la commande de génération et d’envoi (push), accédez au [portail Azure](https://portal.azure.com), puis à votre registre de conteneurs.

10. Dans votre registre de conteneurs, sélectionnez **Dépôts**, puis **samplemodule**. Vérifiez que les deux versions de l’image ont été envoyées (push) au registre.

    ![Afficher les deux versions de l’image dans le registre de conteneurs](./media/tutorial-develop-for-linux/view-repository-versions.png)

<!--Alternative steps: Use VS Code Docker tools to view ACR images with tags-->

### <a name="troubleshoot"></a>Dépanner

Si vous rencontrez des erreurs lors de la génération et de l’envoi (push) de votre image de module, elles sont souvent liées à la configuration de Docker sur votre machine de développement. Pour passer en revue votre configuration, effectuez les vérifications suivantes :

* Avez-vous exécuté la commande `docker login` en utilisant les informations d’identification que vous avez copiées à partir de votre registre de conteneurs ? Ces informations d’identification sont différentes de celles que vous utilisez pour vous connecter à Azure.
* Votre référentiel de conteneurs est-il correct ? A-t-il les noms de registre de conteneur et de module appropriés ? Ouvrez le fichier **module.json** dans le dossier SampleModule pour vérifier. La valeur du référentiel doit se présenter comme ceci : **\<registry name\>.azurecr.io/samplemodule**.
* Si vous avez utilisé un autre nom que **SampleModule** pour votre module, ce nom est-il cohérent dans l’ensemble de la solution ?
* Votre machine exécute-t-elle le même type de conteneurs que celui que vous générez ? Ce tutoriel concerne les appareils Linux IoT Edge ; Visual Studio Code doit donc indiquer **amd64** ou **arm32v7** dans la barre latérale, et Docker Desktop doit exécuter des conteneurs Linux.  

## <a name="deploy-modules-to-device"></a>Déployer des modules sur un appareil

Vous avez vérifié que les images de conteneur générées sont stockées dans votre registre de conteneurs. Il est donc temps de les déployer sur un appareil. Vérifiez que votre appareil IoT Edge est opérationnel.

1. Dans l’Explorateur Visual Studio Code, sous la section **Azure IoT Hub**, développez **Appareils** pour voir votre liste d’appareils IoT.

2. Cliquez avec le bouton droit sur l’appareil IoT Edge sur lequel vous souhaitez effectuer le déploiement, puis sélectionnez **Create Deployment for Single Device** (Créer un déploiement pour un seul appareil).

   ![Créer un déploiement pour un seul appareil](./media/tutorial-develop-for-linux/create-deployment.png)

3. Dans l’Explorateur de fichiers, accédez au dossier **config**, puis sélectionnez le fichier **deployment.amd64.json**.

   N’utilisez pas le fichier deployment.template.json, qui ne contient pas les informations d’identification du registre de conteneurs ou les valeurs d’image de module. Si vous ciblez un appareil Linux ARM32, le manifeste de déploiement est nommé deployment.arm32v7.json.

4. Développez la section **Modules** sous votre appareil pour voir la liste des modules déployés et en cours d’exécution. Cliquez sur le bouton Actualiser. Vous devez voir les nouveaux modules SimulatedTemperatureSensor et SampleModule en cours d’exécution sur votre appareil.

   Le démarrage des modules peut prendre plusieurs minutes. Le runtime IoT Edge doit recevoir son nouveau manifeste de déploiement, extraire les images de module à partir du runtime du conteneur, puis démarrer chaque nouveau module.

   ![Afficher les modules s’exécutant sur votre appareil IoT Edge](./media/tutorial-develop-for-linux/view-running-modules.png)

## <a name="view-messages-from-device"></a>Afficher les messages provenant de l’appareil

Le code SampleModule reçoit des messages via sa file d’attente d’entrée et les transmet à sa file d’attente de sortie. Le manifeste de déploiement a déclaré des routes qui ont passé les messages depuis SampleModule vers SimulatedTemperatureSensor, puis vers le hub IoT. Les outils Azure IoT pour Visual Studio Code vous permettent de voir les messages à mesure qu’ils arrivent au hub IoT en provenance de vos différents appareils.

1. Dans l’explorateur Visual Studio Code, cliquez avec le bouton droit sur l’appareil IoT Edge que vous souhaitez superviser, puis sélectionnez **Démarrer la supervision du point de terminaison d’événements intégré**.

2. Regardez la fenêtre de sortie dans Visual Studio Code pour voir les messages arrivant à votre hub IoT.

   ![Afficher les messages appareil-à-cloud entrants](./media/tutorial-develop-for-linux/view-d2c-messages.png)

## <a name="view-changes-on-device"></a>Afficher les modifications sur l’appareil

Si vous souhaitez voir ce qui se passe sur votre appareil lui-même, utilisez les commandes de cette section pour inspecter le runtime IoT Edge et les modules en cours d’exécution sur votre appareil.

Les commandes de cette section concernent votre appareil IoT Edge, pas votre machine de développement. Si vous utilisez une machine virtuelle pour votre appareil IoT Edge, connectez-vous-y maintenant. Dans Azure, accédez à la page de vue d’ensemble de la machine virtuelle et sélectionnez **Se connecter** pour accéder à la connexion Secure Shell.

* Affichez tous les modules déployés sur votre appareil et vérifiez leur état :

   ```bash
   iotedge list
   ```

   Vous devez voir quatre modules : les deux modules du runtime IoT Edge, SimulatedTemperatureSensor et SampleModule. Les quatre doivent être listés comme étant en cours d’exécution.

* Examinez les journaux d’activité d’un module spécifique :

   ```bash
   iotedge logs <module name>
   ```

   Les modules IoT Edge respectent la casse.

   Les journaux d’activité de SampleModule et SimulatedTemperatureSensor doivent afficher les messages qu’ils traitent. Le module edgeAgent étant responsable du démarrage des autres modules, ses journaux d’activité contiennent des informations sur l’implémentation du manifeste de déploiement. Si un module n’est pas listé ou n’est pas en cours d’exécution, les journaux d’activité edgeAgent sont susceptibles de contenir des erreurs. Le module edgeHub est responsable des communications entre les modules et le hub IoT. Si les modules sont opérationnels, mais que les messages n’atteignent pas votre hub IoT, les journaux d’activité edgeHub sont susceptibles de contenir des erreurs.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous envisagez de passer à l’article recommandé suivant, vous pouvez conserver les ressources et configurations que vous avez créées afin de les réutiliser. Vous pouvez également continuer à utiliser le même appareil IoT Edge comme appareil de test.

Sinon, vous pouvez supprimer les ressources Azure et les configurations locales que vous avez utilisées dans cet article pour éviter les frais.

[!INCLUDE [iot-edge-clean-up-cloud-resources](../../includes/iot-edge-clean-up-cloud-resources.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez configuré Visual Studio Code sur votre machine de développement et déployé votre premier module IoT Edge à partir de celle-ci. Les concepts de base n’ayant plus de secret pour vous, essayez d’ajouter des fonctionnalités à un module afin qu’il puisse analyser les données transitant par son intermédiaire. Choisissez votre langage préféré :

> [!div class="nextstepaction"]
> [C](tutorial-c-module.md)
> [C#](tutorial-csharp-module.md)
> [Java](tutorial-java-module.md)
> [Node.js](tutorial-node-module.md)
> [Python](tutorial-python-module.md)
