---
title: Module IoT Edge en C# pour Azure Stack Edge Pro | Microsoft Docs
description: Découvrez comment développer un module IoT Edge C# pouvant être déployé sur votre appareil Azure Stack Edge Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 08/06/2019
ms.author: alkohli
ms.custom: devx-track-csharp
ms.openlocfilehash: 96a6692524eca3a2845d648ab3df2932d00ce823
ms.sourcegitcommit: a2d8acc1b0bf4fba90bfed9241b299dc35753ee6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/12/2020
ms.locfileid: "91951143"
---
# <a name="develop-a-c-iot-edge-module-to-move-files-with-azure-stack-edge-pro"></a>Développer un module IoT Edge en C# pour déplacer des fichiers avec Azure Stack Edge Pro

Cet article vous guide tout au long de la création d’un module IoT Edge pour un déploiement avec votre appareil Azure Stack Edge Pro. Azure Stack Edge Pro est une solution de stockage qui vous permet de traiter les données et de les envoyer vers Azure via le réseau.

Vous pouvez utiliser les modules Azure IoT Edge avec votre appareil Azure Stack Edge Pro pour transformer les données déplacées vers Azure. Le module de cet article implémente une logique conçue pour copier un fichier à partir d’un partage local vers un partage cloud sur votre appareil Azure Stack Edge Pro.

Dans cet article, vous apprendrez comment :

> [!div class="checklist"]
>
> * Créer un registre de conteneurs pour stocker et gérer vos modules (images Docker).
> * Créer un module IoT Edge à déployer sur votre appareil Azure Stack Edge Pro. 


## <a name="about-the-iot-edge-module"></a>À propos du module IoT Edge

Votre appareil Azure Stack Edge Pro peut déployer et exécuter des modules IoT Edge. Les modules Edge sont essentiellement des conteneurs Docker qui effectuent une tâche spécifique, comme la réception d’un message à partir d’un appareil, la transformation d’un message ou l’envoi d’un message à un IoT Hub. À l’aide de cet article, vous allez créer un module qui copie des fichiers à partir d’un partage local vers un partage cloud sur votre appareil Azure Stack Edge Pro.

1. Les fichiers sont écrits sur le partage local de votre appareil Azure Stack Edge Pro.
2. Le générateur d’événements de fichier crée un événement de fichier pour chaque fichier écrit sur le partage local. Les événements de fichier sont également générés quand un fichier est modifié. Les événements de fichier sont ensuite envoyés au hub IoT Edge (dans le runtime IoT Edge).
3. Le module personnalisé IoT Edge traite l’événement de fichier et crée un objet d’événement de fichier qui contient également le chemin d’accès relatif du fichier. Le module génère un chemin d’accès absolu à l’aide du chemin d’accès relatif et copie le fichier à partir du partage local vers le partage cloud. Le module supprime ensuite le fichier du partage local.

![Fonctionnement du module Azure IoT Edge sur Azure Stack Edge Pro](./media/azure-stack-edge-create-iot-edge-module/how-module-works-1.png)

Une fois que le fichier se trouve dans le partage cloud, ce dernier est automatiquement chargé dans votre compte de stockage Azure.

## <a name="prerequisites"></a>Prérequis

Avant de commencer, assurez-vous que vous disposez  des éléments suivants :

- Un appareil Azure Stack Edge Pro en cours d’exécution.

    - L’appareil doit également être associé à une ressource IoT Hub.
    - Le rôle de calcul Edge est configuré pour l’appareil.
    Pour plus d’informations, accédez à la section [Configurer le calcul](azure-stack-edge-deploy-configure-compute.md#configure-compute) de votre appareil Azure Stack Edge Pro.

- Ressources de développement :

    - [Visual Studio Code](https://code.visualstudio.com/).
    - [Extension C# pour Visual Studio Code (développée par OmniSharp)](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp).
    - [Extension Azure IoT Edge pour Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-edge).
    - [SDK .NET Core 2.1](https://www.microsoft.com/net/download).
    - [Docker CE](https://store.docker.com/editions/community/docker-ce-desktop-windows). Vous devrez peut-être créer un compte pour télécharger et installer le logiciel.

## <a name="create-a-container-registry"></a>Créer un registre de conteneur

Un registre Azure Container Registry est un registre Docker privé dans Azure, dans lequel vous pouvez stocker et gérer vos images conteneurs Docker privées. Azure Container Registry et Docker Hub sont les deux services de registre Docker populaires disponibles dans le cloud. Cet article utilise le service Container Registry.

1. Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).
2. Sélectionnez **Créer une ressource > Conteneurs > Container Registry**. Cliquez sur **Créer**.
3. Procédez de la façon suivante :

   1. Indiquez un **nom de registre** unique dans Azure contenant de 5 à 50 caractères alphanumériques.
   2. Choisissez un **Abonnement**.
   3. Sélectionnez un **groupe de ressources** existant ou créez-en un.
   4. Sélectionnez un **emplacement**. Nous vous recommandons de choisir le même emplacement que celui associé à la ressource Azure Stack Edge.
   5. Basculez **Utilisateur administrateur** sur **Activer**.
   6. Définissez la référence SKU sur **De base**.

      ![Créer un registre de conteneurs](./media/azure-stack-edge-create-iot-edge-module/create-container-registry-1.png)
 
4. Sélectionnez **Create** (Créer).
5. Une fois votre registre de conteneurs créé, accédez à celui-ci, puis sélectionnez **Clés d’accès**.

    ![Obtenir les clés d’accès](./media/azure-stack-edge-create-iot-edge-module/get-access-keys-1.png)
 
6. Copiez les valeurs pour **Serveur de connexion**, **Nom d’utilisateur** et **Mot de passe**. Vous utiliserez ces valeurs plus tard pour publier l’image Docker dans votre registre et ajouter les informations d’identification du registre dans le runtime Azure IoT Edge.


## <a name="create-an-iot-edge-module-project"></a>Créer un projet de module IoT Edge

Les étapes suivantes permettent de créer un projet de module IoT Edge basé sur le Kit de développement logiciel (SDK) .NET Core 2.1. Ce projet utilise Visual Studio Code et l’extension Azure IoT Edge.

### <a name="create-a-new-solution"></a>Créer une solution

Créez un modèle de solution C# que vous pouvez personnaliser avec votre propre code.

1. Dans Visual Studio Code, sélectionnez **Affichage > Palette de commandes** pour ouvrir la palette de commandes VS Code.
2. Dans la palette de commandes, entrez et exécutez la commande **Azure: Sign in** et suivez les instructions pour vous connecter à votre compte Azure. Si vous êtes déjà connecté, vous pouvez ignorer cette étape.
3. Dans la palette de commandes, entrez et exécutez la commande **Azure IoT Edge: New IoT Edge solution**. Dans la palette de commandes, spécifiez les informations suivantes pour créer votre solution :

    1. Sélectionnez le dossier où vous souhaitez créer la solution.
    2. Spécifiez un nom pour votre solution ou acceptez le nom par défaut **EdgeSolution**.
    
        ![Créer une solution 1](./media/azure-stack-edge-create-iot-edge-module/create-new-solution-1.png)

    3. Choisissez **Module C#** comme modèle du module.
    4. Remplacez le nom de module par défaut par le nom que vous souhaitez lui attribuer, dans ce cas, **FileCopyModule**.
    
        ![Créer une solution 2](./media/azure-stack-edge-create-iot-edge-module/create-new-solution-2.png)

    5. Spécifiez le registre de conteneurs que vous avez créé dans la section précédente comme référentiel d’images pour votre premier module. Remplacez **localhost:5000** par la valeur de serveur de connexion que vous avez copiée.

        La chaîne finale ressemble à la suivante : `<Login server name>/<Module name>`. Dans cet exemple, il s’agit de la chaîne `mycontreg2.azurecr.io/filecopymodule`.

        ![Créer une solution 3](./media/azure-stack-edge-create-iot-edge-module/create-new-solution-3.png)

4. Accédez à **Fichier > Ouvrir le dossier**.

    ![Créer une solution 4](./media/azure-stack-edge-create-iot-edge-module/create-new-solution-4.png)

5. Parcourez les dossiers et sélectionnez le dossier **EdgeSolution** créé précédemment. La fenêtre VS Code charge l’espace de travail de votre solution IoT Edge, avec ses cinq composants de niveau supérieur. Cet article ne vous fera pas modifier le dossier **.vscode**, ni les fichiers **.gitignore**, **.env** et **deployment.template.json**.
    
    Le seul composant que vous modifierez est le dossier modules. Ce dossier contient le code C# de votre module et les fichiers Docker servant à générer votre module en tant qu’image de conteneur.

    ![Créer une solution 5](./media/azure-stack-edge-create-iot-edge-module/create-new-solution-5.png)

### <a name="update-the-module-with-custom-code"></a>Mettre à jour le module avec du code personnalisé

1. Dans l’Explorateur VS Code, ouvrez **modules > FileCopyModule > Program.cs**.
2. En haut de l’espace de noms **FileCopyModule**, ajoutez les instructions using suivantes pour les types utilisés ultérieurement. **Microsoft.Azure.Devices.Client.Transport.Mqtt** est un protocole permettant d’envoyer des messages au hub IoT Edge.

    ```
    namespace FileCopyModule
    {
        using Microsoft.Azure.Devices.Client.Transport.Mqtt;
        using Newtonsoft.Json;
    ```
3. Ajouter les variables **InputFolderPath** et **OutputFolderPath** à la classe Program.

    ```
    class Program
        {
            static int counter;
            private const string InputFolderPath = "/home/input";
            private const string OutputFolderPath = "/home/output";
    ```

4. Juste après l’étape précédente, ajoutez la classe **FileEvent** pour définir le corps du message.

    ```
    /// <summary>
    /// The FileEvent class defines the body of incoming messages. 
    /// </summary>
    private class FileEvent
    {
        public string ChangeType { get; set; }

        public string ShareRelativeFilePath { get; set; }

        public string ShareName { get; set; }
    }
    ```

5. Dans la **méthode Init**, le code crée et configure un objet **ModuleClient**. Cet objet permet au module de se connecter au runtime Azure IoT Edge local à l’aide du protocole MQTT pour envoyer et recevoir des messages. La chaîne de connexion utilisée dans la méthode Init est fournie au module par le runtime IoT Edge. Le code enregistre un rappel FileCopy pour recevoir des messages du hub IoT Edge via le point de terminaison **input1**. Remplacez la **méthode Init** par le code suivant.

    ```
    /// <summary>
    /// Initializes the ModuleClient and sets up the callback to receive
    /// messages containing file event information
    /// </summary>
    static async Task Init()
    {
        MqttTransportSettings mqttSetting = new MqttTransportSettings(TransportType.Mqtt_Tcp_Only);
        ITransportSettings[] settings = { mqttSetting };

        // Open a connection to the IoT Edge runtime
        ModuleClient ioTHubModuleClient = await ModuleClient.CreateFromEnvironmentAsync(settings);
        await ioTHubModuleClient.OpenAsync();
        Console.WriteLine("IoT Hub module client initialized.");

        // Register callback to be called when a message is received by the module
        await ioTHubModuleClient.SetInputMessageHandlerAsync("input1", FileCopy, ioTHubModuleClient);
    }
    ```

6. Supprimez le code de **PipeMessage method** (Méthode de message de pipe) et remplacez-le par le code pour **FileCopy**.

    ```
        /// <summary>
        /// This method is called whenever the module is sent a message from the IoT Edge Hub.
        /// This method deserializes the file event, extracts the corresponding relative file path, and creates the absolute input file path using the relative file path and the InputFolderPath.
        /// This method also forms the absolute output file path using the relative file path and the OutputFolderPath. It then copies the input file to output file and deletes the input file after the copy is complete.
        /// </summary>
        static async Task<MessageResponse> FileCopy(Message message, object userContext)
        {
            int counterValue = Interlocked.Increment(ref counter);

            try
            {
                byte[] messageBytes = message.GetBytes();
                string messageString = Encoding.UTF8.GetString(messageBytes);
                Console.WriteLine($"Received message: {counterValue}, Body: [{messageString}]");

                if (!string.IsNullOrEmpty(messageString))
                {
                    var fileEvent = JsonConvert.DeserializeObject<FileEvent>(messageString);

                    string relativeFileName = fileEvent.ShareRelativeFilePath.Replace("\\", "/");
                    string inputFilePath = InputFolderPath + relativeFileName;
                    string outputFilePath = OutputFolderPath + relativeFileName;

                    if (File.Exists(inputFilePath))                
                    {
                        Console.WriteLine($"Moving input file: {inputFilePath} to output file: {outputFilePath}");
                        var outputDir = Path.GetDirectoryName(outputFilePath);
                        if (!Directory.Exists(outputDir))
                        {
                            Directory.CreateDirectory(outputDir);
                        }

                        File.Copy(inputFilePath, outputFilePath, true);
                        Console.WriteLine($"Copied input file: {inputFilePath} to output file: {outputFilePath}");
                        File.Delete(inputFilePath);
                        Console.WriteLine($"Deleted input file: {inputFilePath}");
                    } 
                    else
                    {
                        Console.WriteLine($"Skipping this event as input file doesn't exist: {inputFilePath}");   
                    }
                }
            }
            catch (Exception ex)
            {
                Console.WriteLine("Caught exception: {0}", ex.Message);
                Console.WriteLine(ex.StackTrace);
            }

            Console.WriteLine($"Processed event.");
            return MessageResponse.Completed;
        }
    ```

7. Enregistrez ce fichier.
8. Vous pouvez également [télécharger un exemple de code existant](https://azure.microsoft.com/resources/samples/data-box-edge-csharp-modules/?cdn=disable) pour ce projet. Vous pouvez ensuite valider le fichier que vous avez enregistré dans le fichier **program.cs** dans cet exemple.

## <a name="build-your-iot-edge-solution"></a>Générer votre solution IoT Edge

Dans la section précédente, vous avez créé une solution IoT Edge et ajouté du code à FileCopyModule pour copier des fichiers à partir d’un partage local vers le partage cloud. Vous devez maintenant générer la solution comme image de conteneur et l’envoyer à votre registre de conteneurs.

1. Dans VSCode, accédez à Terminal Server > Nouveau terminal pour ouvrir un nouveau terminal intégré Visual Studio Code.
2. Connectez-vous à Docker en entrant la commande suivante dans le terminal intégré.

    `docker login <ACR login server> -u <ACR username>`

    Utilisez le serveur de connexion et le nom d’utilisateur copiés plus tôt à partir du registre de conteneurs.

    ![Générer et envoyer (push) la solution IoT Edge](./media/azure-stack-edge-create-iot-edge-module/build-iot-edge-solution-1.png)

2. Lorsque vous y êtes invité, renseignez le mot de passe. Vous pouvez également récupérer la valeur du serveur de connexion, le nom d’utilisateur et le mot de passe à partir des **Clés d’accès** de votre registre de conteneurs sur le portail Azure.
 
3. Une fois les informations d’identification fournies, vous pouvez transmettre l’image de votre module au registre Azure Container Registry. Dans l’Explorateur VS Code, cliquez avec le bouton droit sur le fichier **module.json** et sélectionnez **Build and Push IoT Edge solution** (Générer et envoyer (push) la solution IoT Edge).

    ![Créer et envoyer (push) la solution IoT Edge 2](./media/azure-stack-edge-create-iot-edge-module/build-iot-edge-solution-2.png)
 
    Lorsque vous indiquez à Visual Studio Code de générer votre solution, il exécute deux commandes dans le terminal intégré : docker build et docker push. Ces deux commandes génèrent votre code, mettent votre module CSharpModule.dll en conteneur, puis envoient le code au registre de conteneurs que vous avez spécifié lors de l’initialisation de la solution.

    Vous êtes invité à choisir la plateforme du module. Sélectionnez le paramètre *amd64* correspondant à Linux.

    ![Sélectionner une plateforme](./media/azure-stack-edge-create-iot-edge-module/select-platform.png)

    > [!IMPORTANT] 
    > Seuls les modules Linux sont pris en charge.

    L’avertissement suivant peut s’afficher, mais vous pouvez l’ignorer :

    *Program.cs(77,44) : avertissement CS1998 : Cette méthode async n’a pas d’opérateur 'await' et elle s’exécutera de façon synchrone. Utilisez l’opérateur « await » pour attendre les appels d’API non bloquants ou « await Task.Run(…) » pour effectuer un travail utilisant le processeur sur un thread d’arrière-plan.*

4. Vous pouvez afficher l’adresse complète de l’image conteneur avec la balise dans le terminal intégré de VS Code. L’adresse de l’image est créée à partir d’informations contenues dans le fichier module.json, au format `<repository>:<version>-<platform>`. Dans cet article, cela ressemble à ce qui suit : `mycontreg2.azurecr.io/filecopymodule:0.0.1-amd64`.

## <a name="next-steps"></a>Étapes suivantes

Pour déployer et exécuter ce module sur Azure Stack Edge Pro, consultez les étapes décrites dans la section [Ajouter un module](azure-stack-edge-deploy-configure-compute.md#add-a-module).
