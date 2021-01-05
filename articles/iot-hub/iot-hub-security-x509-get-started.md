---
title: Didacticiel pour la sécurité X.509 dans Azure IoT Hub | Microsoft Docs
description: Commencez à utiliser la sécurité X.509 dans votre Azure IoT Hub dans un environnement simulé.
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 08/20/2019
ms.custom:
- amqp
- 'Role: Cloud Development'
- 'Role: IoT Device'
- devx-track-csharp
ms.openlocfilehash: a3e328418a0f111cd0b985310ea6dc497999772d
ms.sourcegitcommit: d76108b476259fe3f5f20a91ed2c237c1577df14
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92909792"
---
# <a name="set-up-x509-security-in-your-azure-iot-hub"></a>Configurer la sécurité X.509 dans votre Azure IoT Hub

Ce didacticiel indique la procédure que vous devez suivre pour sécuriser votre Azure IoT Hub à l’aide de *l’authentification de certificat X.509*. À titre d’illustration, nous utilisons l’outil open source OpenSSL pour créer des certificats localement sur votre machine Windows. Nous vous recommandons de n’utiliser ce didacticiel qu’à des fins de test. Pour un environnement de production, achetez les certificats auprès d’une *autorité de certification racine*. Veillez également à mettre en place une stratégie pour gérer la substitution de certificat à l’expiration d’un certificat d’appareil ou d’autorité de certification.

[!INCLUDE [iot-hub-include-x509-ca-signed-support-note](../../includes/iot-hub-include-x509-ca-signed-support-note.md)]

## <a name="prerequisites"></a>Prérequis

Pour suivre ce didacticiel, vous devez disposer des ressources suivantes :

* Vous avez créé un IoT Hub avec votre abonnement Azure. Pour découvrir la procédure détaillée correspondante, consultez l’article [Création d’un IoT Hub à l’aide du portail Azure](iot-hub-create-through-portal.md).

* Vous avez installé [Visual Studio 2017 ou Visual Studio 2019](https://www.visualstudio.com/vs/).

## <a name="get-x509-ca-certificates"></a>Obtenir des certificats d’autorité de certification X.509

Pour mettre en place une sécurité basée sur un certificat X.509 dans le IoT Hub, vous devez commencer par une [chaîne d’approbation X.509](https://en.wikipedia.org/wiki/X.509#Certificate_chains_and_cross-certification), qui inclut le certificat racine, ainsi que tous les certificats intermédiaires jusqu’au certificat feuille.

Vous pouvez choisir l’une des méthodes suivantes pour récupérer vos certificats :

* Achetez des certificats X.509 auprès d’une *autorité de certification racine*. Cette méthode est recommandée pour les environnements de production.

* Créez vos propres certificats X.509 à l’aide d’un outil tiers, par exemple [OpenSSL](https://www.openssl.org/). Cette technique se révèle parfaitement adaptée aux scénarios de test et de développement. Consultez [Gestion de certificats d’autorité de certification de test pour des exemples et tutoriels](https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md) pour obtenir des informations sur la génération de certificats d’autorité de certification de test à l’aide de PowerShell ou Bash. Le reste de ce tutoriel utilise des certificats d’autorité de certification de test générés en suivant les instructions fournies dans [Gestion de certificats d’autorité de certification de test pour des exemples et tutoriels](https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md).

* Générez un [certificat d’autorité de certification intermédiaire X.509](iot-hub-x509ca-overview.md#sign-devices-into-the-certificate-chain-of-trust) signé par un certificat d’autorité de certification racine existant et chargez-le sur le hub. Une fois le certificat intermédiaire chargé et vérifié, comme indiqué ci-dessous, il peut être utilisé à la place d’un certificat d’autorité de certification racine mentionné ci-dessous. Des outils tels qu’OpenSSL ([openssl req](https://www.openssl.org/docs/man1.1.0/man1/req.html) et [openssl ca](https://www.openssl.org/docs/man1.1.0/man1/ca.html)) peuvent être utilisés pour générer et signer un certificat d’autorité de certification intermédiaire.

> [!NOTE]
> Ne téléchargez pas la racine tierce si elle ne vous est pas propre, car cela permettrait à d’autres clients du tiers de connecter leurs appareils à votre IoT Hub.

## <a name="register-x509-ca-certificates-to-your-iot-hub"></a>Inscrire des certificats d’autorité de certification X.509 auprès de votre IoT Hub

Cette section décrit la procédure d’ajout d’une nouvelle autorité de certification à votre IoT Hub par le biais du portail. Si vous utilisez l’authentification par certificat d’autorité de certification X.509, veillez à inscrire votre nouveau certificat avant l’expiration de l’actuel dans le cadre de votre stratégie de substitution de certificat.

> [!NOTE]
> Le nombre maximal de certificats d’autorité de certification X.509 pouvant être inscrits sur un hub IoT est 25. Pour plus d’informations, consultez [Quotas et limitations de requêtes Azure IoT Hub](iot-hub-devguide-quotas-throttling.md).

1. Dans le Portail Azure, accédez à votre hub IOT et sélectionnez **Paramètres** > **Certificats** pour le hub.

1. Sélectionnez **Ajouter** pour ajouter un nouveau certificat.

1. Dans **Nom du certificat** , entrez un nom complet convivial et sélectionnez le fichier de certificat que vous avez créé dans la section précédente sur votre ordinateur.

1. Une fois que vous avez obtenu une notification vous informant que votre certificat a été correctement chargé, sélectionnez **Enregistrer**.

    ![Téléchargement d’un certificat](./media/iot-hub-security-x509-get-started/iot-hub-add-cert.png)  

   Votre certificat s’affiche dans la liste des certificats avec l’état **Non vérifié**.

1. Sélectionnez le certificat que vous venez d’ajouter pour afficher les **Détails du certificat** , puis sélectionnez **Générer le code de vérification**.

   ![Vérifier le certificat](./media/iot-hub-security-x509-get-started/copy-verification-code.png)  

1. Copiez le **code de vérification** dans le Presse-papiers. Vous l’utilisez pour valider la propriété du certificat.

1. Suivez l’étape 3 dans [Gestion de certificats d’autorité de certification de test pour des exemples et tutoriels](https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md).  Ce processus signer votre code de vérification avec la clé privée associée à votre certificat d’autorité de certification X.509, ce qui génère une signature. Vous disposez de plusieurs outils pour effectuer ce processus de signature, par exemple OpenSSL. Ce processus est désigné sous le terme de [preuve de possession](https://tools.ietf.org/html/rfc5280#section-3.1).

1. Dans **Détails du certificat** , sous **Fichier .pem ou. cer du certificat de vérification** , recherchez et ouvrez le fichier de signature. Ensuite, sélectionnez **Vérifier**.

   L’état de votre certificat passe à **Vérifié**. Sélectionnez **Actualiser** si le certificat ne se met pas à jour automatiquement.

## <a name="create-an-x509-device-for-your-iot-hub"></a>Créer un appareil X.509 pour votre IoT Hub

1. Dans le Portail Azure, accédez à votre hub IOT, puis sélectionnez **Explorateurs** > **Appareils IOT**.

1. Sélectionnez **Nouveau** pour ajouter un nouvel appareil.

1. Dans **ID d’appareil** , entrez un nom d’affichage convivial. Pour **Type d’authentification** , choisissez **Autorité de certification X.509 signée** , puis sélectionnez **Enregistrer**.

   ![Créer un appareil X.509 dans le portail](./media/iot-hub-security-x509-get-started/new-x509-device.png)

## <a name="authenticate-your-x509-device-with-the-x509-certificates"></a>Authentifier votre appareil X.509 avec les certificats X.509

Pour authentifier votre appareil X.509, vous devez commencer par signer l’appareil avec le certificat d’autorité de certification. La signature des appareils feuilles est généralement effectuée à l’usine de fabrication, dans laquelle les outils de fabrication ont été activés en conséquence. Lorsque l’appareil passe d’un fabricant à un autre, l’action de signature de chaque fabricant est capturée sous la forme d’un certificat intermédiaire dans la chaîne. Le résultat est une chaîne d’approbation comprenant plusieurs certificats, depuis le certificat d’autorité de certification jusqu’au certificat feuille de l’appareil. L’étape 4 dans [Gestion de certificats d’autorité de certification de test pour des exemples et tutoriels](https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md) génère un certificat d’appareil.

Ensuite, nous vous indiquerons comment créer une application C# pour simuler l’appareil X.509 inscrit pour votre IoT Hub. Nous enverrons les valeurs de température et d’humidité de l’appareil simulé sur votre hub. Dans ce didacticiel, nous créerons uniquement l’application de l’appareil. En guide d’exercice, les lecteurs seront chargés de créer l’application de service IoT Hub qui enverra la réponse aux événements envoyés par cet appareil simulé. L’application C# suppose que vous avez suivi les étapes décrites dans [Gestion de certificats d’autorité de certification de test pour des exemples et tutoriels](https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md).

1. Ouvrez Visual Studio, sélectionnez **Créer un nouveau projet** , puis choisissez le modèle de projet **Application console (.NET Framework)** . Sélectionnez **Suivant**.

1. Dans **Configurer votre nouveau projet** , nommez le projet *SimulateX509Device* , puis sélectionnez **Créer**.

   ![Créer un projet d’appareil X.509 dans Visual Studio](./media/iot-hub-security-x509-get-started/create-device-project-vs2019.png)

1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet **SimulateX509Device** , puis sélectionnez **Gérer les packages NuGet**.

1. Dans **Gestionnaire de package NuGet** , sélectionnez **Parcourir** , puis recherchez et choisissez **Microsoft.Azure.Devices.Client**. Sélectionnez **Installer**.

   ![Ajouter le package NuGet device SDK dans Visual Studio](./media/iot-hub-security-x509-get-started/device-sdk-nuget.png)

    Cette opération a pour effet de télécharger, d’installer et d’ajouter une référence au package NuGet Azure IoT device SDK et ses dépendances.

1. Ajoutez les instructions `using` suivantes en haut du fichier **Program.cs** :

    ```csharp
        using Microsoft.Azure.Devices.Client;
        using Microsoft.Azure.Devices.Shared;
        using System.Security.Cryptography.X509Certificates;
    ```

1. Ajoutez les champs suivants à la classe **Program**  :

    ```csharp
        private static int MESSAGE_COUNT = 5;
        private const int TEMPERATURE_THRESHOLD = 30;
        private static String deviceId = "<your-device-id>";
        private static float temperature;
        private static float humidity;
        private static Random rnd = new Random();
    ```

    Utilisez le nom d’appareil convivial que vous avez utilisé à la section précédente à la place de _< your_device_id >_ .

1. Ajoutez la fonction ci-après pour créer des valeurs aléatoires pour la température et l’humidité et pour envoyer ces valeurs au hub :

    ```csharp
    static async Task SendEvent(DeviceClient deviceClient)
    {
        string dataBuffer;
        Console.WriteLine("Device sending {0} messages to IoTHub...\n", MESSAGE_COUNT);

        for (int count = 0; count < MESSAGE_COUNT; count++)
        {
            temperature = rnd.Next(20, 35);
            humidity = rnd.Next(60, 80);
            dataBuffer = string.Format("{{\"deviceId\":\"{0}\",\"messageId\":{1},\"temperature\":{2},\"humidity\":{3}}}", deviceId, count, temperature, humidity);
            Message eventMessage = new Message(Encoding.UTF8.GetBytes(dataBuffer));
            eventMessage.Properties.Add("temperatureAlert", (temperature > TEMPERATURE_THRESHOLD) ? "true" : "false");
            Console.WriteLine("\t{0}> Sending message: {1}, Data: [{2}]", DateTime.Now.ToLocalTime(), count, dataBuffer);

            await deviceClient.SendEventAsync(eventMessage);
        }
    }
    ```

1. Pour finir, ajoutez les lignes de code ci-après à la fonction **Main** en remplaçant les espaces réservés _device-id_ , _your-iot-hub-name_ et _absolute-path-to-your-device-pfx-file_ par les informations requises par votre configuration.

    ```csharp
    try
    {
        var cert = new X509Certificate2(@"<absolute-path-to-your-device-pfx-file>", "1234");
        var auth = new DeviceAuthenticationWithX509Certificate("<device-id>", cert);
        var deviceClient = DeviceClient.Create("<your-iot-hub-name>.azure-devices.net", auth, TransportType.Amqp_Tcp_Only);

        if (deviceClient == null)
        {
            Console.WriteLine("Failed to create DeviceClient!");
        }
        else
        {
            Console.WriteLine("Successfully created DeviceClient!");
            SendEvent(deviceClient).Wait();
        }

        Console.WriteLine("Exiting...\n");
    }
    catch (Exception ex)
    {
        Console.WriteLine("Error in sample: {0}", ex.Message);
    }
    ```

   Ce code se connecte à votre IoT Hub en créant la chaîne de connexion pour votre appareil X.509. Une fois le code connecté, il envoie les événements de température et d’humidité au hub, puis attend la réponse de ce dernier.

1. Exécutez l'application. Étant donné que cette application accède à un fichier *.pfx* , vous devrez peut-être exécuter cette application en tant qu’administrateur.

   1. Créez la solution Visual Studio.

   1. Ouvrez une nouvelle fenêtre d’invite de commandes en utilisant **Exécuter en tant qu’administrateur**.  

   1. Accédez au dossier qui contient votre solution, puis accédez au chemin *bin/Debug* dans le dossier de la solution.

   1. Exécutez l’application **SimulateX509Device. exe** à partir de l’invite de commandes.

   Votre appareil doit alors se connecter correctement au hub et envoyer les événements.

   ![Exécuter l’application de l’appareil](./media/iot-hub-security-x509-get-started/device-app-success.png)

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur la sécurisation de votre solution IoT, consultez :

* [Meilleures pratiques de sécurité pour l’Internet des objets (IoT)](../iot-fundamentals/iot-security-best-practices.md)

* [Architecture de sécurité de l’Internet des objets (IoT)](../iot-fundamentals/iot-security-architecture.md)

* [Sécuriser votre déploiement IoT](../iot-fundamentals/iot-security-deployment.md)

Pour explorer davantage les capacités de IoT Hub, consultez :

* [Déploiement d’une IA sur des appareils de périmètre avec Azure IoT Edge](../iot-edge/quickstart-linux.md)