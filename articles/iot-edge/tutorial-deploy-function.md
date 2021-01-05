---
title: 'Tutoriel : Déployer des fonctions Azure en tant que modules - Azure IoT Edge'
description: Dans ce tutoriel, vous allez développer une fonction Azure en tant que module IoT Edge avant de la déployer sur un appareil de périphérie.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 07/29/2020
ms.topic: tutorial
ms.service: iot-edge
services: iot-edge
ms.custom: mvc, devx-track-csharp
ms.openlocfilehash: 5b5d60a014cf63bd28f3097ac3131ad4c7018208
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96621244"
---
# <a name="tutorial-deploy-azure-functions-as-iot-edge-modules"></a>Tutoriel : Déployer des fonctions Azure comme modules IoT Edge

Vous pouvez utiliser Azure Functions pour déployer un code qui implémente votre logique métier directement sur vos appareils Azure IoT Edge. Ce tutoriel vous guide dans la création et le déploiement d’une fonction Azure qui filtre des données de capteur sur l’appareil IoT Edge simulé. Vous utilisez l’appareil IoT Edge simulé que vous avez créé dans les tutoriels Déployer Azure IoT Edge sur un appareil simulé sous [Windows](quickstart.md) ou [Linux](quickstart-linux.md). Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> * Utilisez Visual Studio Code pour créer une fonction Azure.
> * Utilisez VS Code et Docker pour créer une image Docker et la publier sur un registre de conteneurs.
> * Déployez le module du registre de conteneurs sur votre appareil IoT Edge.
> * Affichez les données filtrées.

<center>

![Diagramme - Architecture du tutoriel : indexer et déployer un module de fonction](./media/tutorial-deploy-function/functions-architecture.png)
</center>

La fonction Azure que vous créez dans ce tutoriel filtre les données de température générées par votre appareil. La fonction envoie uniquement des messages en amont au hub Azure IoT quand la température dépasse un seuil spécifié.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

Avant de commencer ce tutoriel, vous devez avoir effectué celui qui précède pour configurer votre environnement de développement pour le développement de conteneur Linux : [Développer des modules IoT Edge pour les appareils Linux](tutorial-develop-for-linux.md). En suivant ce tutoriel, les conditions préalables suivantes doivent être remplies :

* Un niveau gratuit ou standard [IoT Hub](../iot-hub/iot-hub-create-through-portal.md) dans Azure.
* Un [appareil Linux exécutant Azure IoT Edge](quickstart-linux.md).
* Un registre de conteneurs tel qu’[Azure Container Registry](../container-registry/index.yml).
* [Visual Studio Code](https://code.visualstudio.com/) configuré avec [Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools).
* [Docker CE](https://docs.docker.com/install/) configuré pour exécuter des conteneurs Linux.

Pour développer un module IoT Edge avec Azure Functions, installez les conditions préalables supplémentaires suivantes sur votre machine de développement :

* [Extension C# pour Visual Studio Code (développée par OmniSharp)](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp).
* [SDK .NET Core 2.1](https://www.microsoft.com/net/download).

## <a name="create-a-function-project"></a>Créer un projet Function

Les outils Azure IoT pour Visual Studio Code que vous avez installés conformément aux prérequis offre des fonctionnalités de gestion, ainsi que certains modèles de code. Dans cette section, vous allez utiliser Visual Studio Code pour créer une solution IoT Edge contenant une fonction Azure.

### <a name="create-a-new-project"></a>Création d'un projet

Créez un modèle de solution de fonction C# que vous pouvez personnaliser avec votre propre code.

1. Ouvrez Visual Studio Code sur votre ordinateur de développement.

2. Ouvrez la palette de commandes VS Code en sélectionnant **Affichage** > **Palette de commandes**.

3. Dans la palette de commandes, entrez et exécutez la commande **Azure IoT Edge: New IoT Edge solution**. Suivez les invites de la palette de commandes pour créer votre solution.

   | Champ | Valeur |
   | ----- | ----- |
   | Sélectionner le dossier | Choisissez l’emplacement sur votre machine de développement pour que VS Code crée les fichiers de la solution. |
   | Provide a solution name (Nommer la solution) | Entrez un nom descriptif pour votre solution, par exemple **FunctionSolution**, ou acceptez le nom par défaut. |
   | Select module template (Sélectionner un modèle de module) | Choisissez **Azure Functions - C#** . |
   | Provide a module name (Nommer le module) | Nommez votre module **CSharpFunction**. |
   | Provide Docker image repository for the module (Indiquer le référentiel d’images Docker pour le module) | Un référentiel d’images comprend le nom de votre registre de conteneurs et celui de votre image conteneur. Votre image conteneur est préremplie à partir de la dernière étape. Remplacez **localhost:5000** par la valeur de **Serveur de connexion** provenant de votre registre de conteneurs Azure. Vous pouvez récupérer le serveur de connexion à partir de la page Vue d’ensemble de votre registre de conteneurs dans le portail Azure. La chaîne finale ressemble à \<registry name\>.azurecr.io/CSharpFunction. |

   ![Fourniture du référentiel d’images Docker](./media/tutorial-deploy-function/repository.png)

### <a name="add-your-registry-credentials"></a>Ajouter les informations d’identification de votre registre

Le fichier d’environnement stocke les informations d’identification de votre registre de conteneurs et les partage avec le runtime IoT Edge. Le runtime a besoin de ces informations d’identification pour extraire vos images privées sur l’appareil IoT Edge.

L’extension IoT Edge tente d’extraire d’Azure vos informations d’identification de registre de conteneurs et de les insérer dans le fichier d’environnement. Vérifiez si vos informations d’identification sont déjà incluses. Si ce n’est pas le cas, veuillez les ajouter maintenant :

1. Dans l’Explorateur VS Code, ouvrez le fichier .env.
2. Mettre à jour les champs avec les valeurs de **nom d’utilisateur** et de **mot de passe** que vous avez copiées à partir de votre registre de conteneurs Azure.
3. Enregistrez ce fichier.

### <a name="select-your-target-architecture"></a>Sélectionner votre architecture cible

Visual Studio Code peut développer des modules C pour les appareils Linux AMD64 et Linux ARM32v7. Vous devez sélectionner l’architecture que vous ciblez avec chaque solution, car le conteneur est généré et s’exécute différemment pour chaque type d’architecture. Linux AMD64 est la valeur par défaut.

1. Ouvrez la palette de commandes et recherchez **Azure IoT Edge: Set Default Target Platform for Edge Solution** (Azure IoT Edge : définir la plateforme cible par défaut pour la solution Edge), ou sélectionnez l’icône de raccourci dans la barre latérale en bas de la fenêtre.

2. Dans la palette de commandes, sélectionnez l’architecture cible dans la liste des options. Pour ce tutoriel, comme nous utilisons une machine virtuelle Ubuntu en tant qu’appareil IoT Edge, nous allons conserver la valeur par défaut **amd64**.

### <a name="update-the-module-with-custom-code"></a>Mettre à jour le module avec du code personnalisé

Nous allons ajouter du code supplémentaire afin que le module traite les messages en périphérie avant leur transfert vers IoT Hub.

1. Dans Visual Studio Code, ouvrez **modules** > **CSharpFunction** > **CSharpFunction.cs**.

1. Remplacez le contenu du fichier **CSharpFunction.cs** par le code suivant. Ce code reçoit les données de télémétrie concernant la température ambiante et la température de l’appareil. Il transmet uniquement le message à IoT Hub si la température de l’appareil dépasse un seuil défini.

   ```csharp
   using System;
   using System.Collections.Generic;
   using System.IO;
   using System.Text;
   using System.Threading.Tasks;
   using Microsoft.Azure.Devices.Client;
   using Microsoft.Azure.WebJobs;
   using Microsoft.Azure.WebJobs.Extensions.EdgeHub;
   using Microsoft.Azure.WebJobs.Host;
   using Microsoft.Extensions.Logging;
   using Newtonsoft.Json;

   namespace Functions.Samples
   {
       public static class CSharpFunction
       {
           [FunctionName("CSharpFunction")]
           public static async Task FilterMessageAndSendMessage(
               [EdgeHubTrigger("input1")] Message messageReceived,
               [EdgeHub(OutputName = "output1")] IAsyncCollector<Message> output,
               ILogger logger)
           {
               const int temperatureThreshold = 20;
               byte[] messageBytes = messageReceived.GetBytes();
               var messageString = System.Text.Encoding.UTF8.GetString(messageBytes);

               if (!string.IsNullOrEmpty(messageString))
               {
                   logger.LogInformation("Info: Received one non-empty message");
                   // Get the body of the message and deserialize it.
                   var messageBody = JsonConvert.DeserializeObject<MessageBody>(messageString);

                   if (messageBody != null && messageBody.machine.temperature > temperatureThreshold)
                   {
                       // Send the message to the output as the temperature value is greater than the threshold.
                       using (var filteredMessage = new Message(messageBytes))
                       {
                            // Copy the properties of the original message into the new Message object.
                            foreach (KeyValuePair<string, string> prop in messageReceived.Properties)
                            {filteredMessage.Properties.Add(prop.Key, prop.Value);}
                            // Add a new property to the message to indicate it is an alert.
                            filteredMessage.Properties.Add("MessageType", "Alert");
                            // Send the message.
                            await output.AddAsync(filteredMessage);
                            logger.LogInformation("Info: Received and transferred a message with temperature above the threshold");
                       }
                   }
               }
           }
       }
       //Define the expected schema for the body of incoming messages.
       class MessageBody
       {
           public Machine machine {get; set;}
           public Ambient ambient {get; set;}
           public string timeCreated {get; set;}
       }
       class Machine
       {
           public double temperature {get; set;}
           public double pressure {get; set;}
       }
       class Ambient
       {
           public double temperature {get; set;}
           public int humidity {get; set;}
       }
   }
   ```

1. Enregistrez le fichier .

## <a name="build-and-push-your-iot-edge-solution"></a>Créer et envoyer (push) votre solution IoT Edge

Dans la section précédente, vous avez créé une solution IoT Edge et modifié **CSharpFunction** pour filtrer les messages dans lesquels la température de machine signalée est sous le seuil acceptable. Vous devez maintenant générer la solution comme image de conteneur et l’envoyer à votre registre de conteneurs.

1. Ouvrez le terminal intégré VS Code en sélectionnant **Affichage** > **Terminal**.

2. Connectez-vous à Docker en entrant la commande suivante dans le terminal. Connectez-vous avec le nom d’utilisateur, le mot de passe et le serveur de connexion de votre registre de conteneurs Azure. Vous pouvez récupérer ces valeurs dans la section **Clés d’accès** de votre registre dans le portail Azure.

   ```bash
   docker login -u <ACR username> -p <ACR password> <ACR login server>
   ```

   Il se peut que vous receviez un avertissement de sécurité recommandant d’utiliser `--password-stdin`. Bien qu’il s’agisse de la bonne pratique recommandée pour les scénarios de production, elle n’est pas pertinente pour ce tutoriel. Pour plus d’informations, consultez les informations de référence sur [docker login](https://docs.docker.com/engine/reference/commandline/login/#provide-a-password-using-stdin).

3. Dans l’Explorateur VS Code, cliquez avec le bouton droit sur le fichier **deployment.template.json** et sélectionnez **Build and Push IoT Edge solution** (Générer et envoyer (push) la solution IoT Edge).

   La commande de génération et d’envoi (push) déclenche trois opérations. Tout d’abord, elle crée un dossier dans la solution appelé **config** contenant le manifeste de déploiement en entier. Il est généré à partir des informations dans le modèle de déploiement et d’autres fichiers de solution. Ensuite, elle exécute `docker build` pour générer l’image de conteneur basée sur le fichier docker correspondant à votre architecture cible. Puis, elle exécute `docker push` pour envoyer (push) le dépôt d’images vers votre registre de conteneurs.

   Ce processus peut prendre plusieurs minutes la première fois, mais il est plus rapide la prochaine fois que vous exécutez les commandes.

## <a name="view-your-container-image"></a>Afficher votre image de conteneur

Visual Studio Code génère un message de réussite quand l’image de conteneur est envoyée au registre de conteneurs. Si vous voulez confirmer la réussite de l’opération pour vous-même, vous pouvez afficher l’image dans le registre.

1. Dans le portail Azure, accédez à votre registre Azure Container Registry.
2. Sélectionnez **Dépôts**.
3. Le référentiel **csharpfunction** doit apparaître dans la liste. Sélectionnez ce référentiel pour afficher plus de détails.
4. Dans la section **Balises**, vous devez voir la balise **0.0.1-amd64**. Cette balise indique la version et la plateforme de l’image que vous avez générée. Ces valeurs sont définies dans le fichier module.json dans le dossier CSharpFunction.

## <a name="deploy-and-run-the-solution"></a>Déployer et exécuter la solution

Vous pouvez utiliser le portail Azure pour déployer votre module de fonction sur un appareil IoT Edge comme vous l’avez fait dans le cadre des guides de démarrage rapide. Vous pouvez également déployer et surveiller des modules à partir de Visual Studio Code. Les sections suivantes utilisent les outils Azure IoT pour VS Code qui figurent dans les prérequis. Installez l’extension maintenant, si ce n’est pas déjà fait.

1. Dans l’Explorateur Visual Studio Code, sous la section **Azure IoT Hub**, développez **Appareils** pour voir votre liste d’appareils IoT.

2. Cliquez avec le bouton droit sur le nom de votre appareil IoT Edge, puis sélectionnez **Créer un déploiement pour un seul appareil**.

3. Accédez au dossier solution qui contient la fonction **CSharpFunction**. Ouvrez le dossier config, sélectionnez le fichier **deployment.amd64.json**, puis cliquez sur **Sélectionner un manifeste de déploiement Edge**.

4. Développez la section **Modules** sous votre appareil pour voir la liste des modules déployés et en cours d’exécution. Cliquez sur le bouton Actualiser. Vous devez voir le nouveau module **CSharpFunction** en cours d’exécution avec le module **SimulatedTemperatureSensor** ainsi que **$edgeAgent** et **$edgeHub**.

    Plusieurs minutes peuvent être nécessaires avant que les nouveaux modules n’apparaissent. Votre appareil IoT Edge doit récupérer ses nouvelles informations de déploiement dans IoT Hub, démarrer les nouveaux conteneurs, puis communiquer son état à IoT Hub.

   ![Afficher les modules déployés dans VS Code](./media/tutorial-deploy-function/view-modules.png)

## <a name="view-the-generated-data"></a>Afficher les données générées

Vous pouvez voir tous les messages arrivant dans votre hub IoT en exécutant **Azure IoT Hub: Start Monitoring Built-in Event Endpoint** (Démarrer la supervision du point de terminaison d’événements intégré) dans la palette de commandes.

Vous pouvez également filtrer pour afficher tous les messages qui arrivent dans votre hub IoT à partir d’un appareil spécifique. Cliquez avec le bouton droit sur l’appareil dans la section **Appareils Azure IoT Hub** et sélectionnez **Start Monitoring Built-in Event Endpoint** (Démarrer la supervision du point de terminaison d’événements intégré).

Pour cesser la surveillance des messages, exécutez la commande **Azure IoT Hub: Stop Monitoring Built-in Event Endpoint** (Arrêter la supervision du point de terminaison d’événements intégré) dans la palette de commandes.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous envisagez de passer à l’article recommandé suivant, vous pouvez conserver les ressources et configurations que vous avez créées afin de les réutiliser. Vous pouvez également continuer à utiliser le même appareil IoT Edge comme appareil de test.

Sinon, vous pouvez supprimer les ressources Azure et les configurations locales que vous avez créées dans cet article pour éviter les frais.

[!INCLUDE [iot-edge-clean-up-cloud-resources](../../includes/iot-edge-clean-up-cloud-resources.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez créé un module de fonction Azure contenant le code pour filtrer les données brutes générées par votre appareil IoT Edge.

Passez aux tutoriels suivants pour en savoir plus sur les autres façons dont Azure IoT Edge peut vous aider à transformer des données en informations métier « en périphérie».

> [!div class="nextstepaction"]
> [Rechercher des moyennes à l’aide d’une fenêtre flottante dans Azure Stream Analytics](tutorial-deploy-stream-analytics.md)
