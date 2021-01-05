---
title: Développer et déployer un serveur d’inférence gRPC - Azure
description: Cet article fournit des conseils sur le développement et le déploiement d’un serveur d’inférence gRPC.
ms.topic: how-to
ms.date: 12/02/2020
ms.openlocfilehash: 3f732a7432dacebeeefddd1822fec7d95dfbaa97
ms.sourcegitcommit: cc13f3fc9b8d309986409276b48ffb77953f4458
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97417433"
---
# <a name="how-to-guide--develop-and-deploy-a-grpc-inference-server"></a>Guide pratique - Développer et déployer un serveur d’inférence gRPC

## <a name="overview"></a>Vue d’ensemble

Cet article explique comment vous pouvez encapsuler le ou les modèles IA de votre choix au sein d’un serveur d’inférence gRPC, afin qu’ils puissent être intégrés à LVA (Live Video Analytics) via l’extension Graph. 

## <a name="suggested-pre-reading"></a>Lecture préalable suggérée

* [Extension de graphe multimédia](media-graph-extension-concept.md)
* [Protocole d’extension gRPC](grpc-extension-protocol.md)
* [Schéma des métadonnées d’inférence](inference-metadata-schema.md)
* [Présentation de gRPC](https://www.grpc.io/docs/what-is-grpc/introduction/)
* [Guide du langage proto3](https://developers.google.com/protocol-buffers/docs/proto3)

## <a name="prerequisites"></a>Prérequis

* Un appareil x86-64 ou ARM64 exécutant l’un des [systèmes d’exploitation Linux pris en charge](https://docs.microsoft.com/azure/iot-edge/support#operating-systems) ou un ordinateur Windows.
* [Installation de Docker](https://docs.docker.com/desktop/#download-and-install) sur votre ordinateur.
* Installation d’un [runtime IoT Edge](https://docs.microsoft.com/azure/iot-edge/how-to-install-iot-edge?tabs=linux).

## <a name="grpc-implementation-steps"></a>Étapes d’implémentation gRPC

Pour créer un serveur d’inférence gRPC et l’implémenter en tant qu’extension avec Live Video Analytics, les étapes suivantes sont requises :

### <a name="setup"></a>Paramétrage :

Effectuez les étapes nécessaires pour que le [module Live Video Analytics soit déployé et qu’il fonctionne sur un appareil IoT Edge](deploy-iot-edge-device.md).

### <a name="high-level-implementation-steps"></a>Principales étapes d’implémentation :

1. Choisissez l’un des nombreux langages pris en charge par gRPC : C#, C++, Dart, Go, Java, Node, Objective-C, PHP, Python, Ruby.
1. Implémentez un serveur gRPC qui communiquera avec Live Video Analytics à l’aide de [fichiers proto3](https://github.com/Azure/live-video-analytics/tree/master/contracts/grpc).

    :::image type="content" source="./media/develop-deploy-grpc-inference-srv-how-to/inference-srv-container-process.png" alt-text="Serveur gRPC qui communiquera avec Live Video Analytics à l’aide de fichiers proto3":::

    Au sein de ce service :
    1. Gérez l’échange de messages de description de session entre le serveur et le client.
    1. Gérez les [exemples de messages multimédia](https://github.com/Azure/live-video-analytics/blob/master/contracts/grpc/extension.proto) et retournez des résultats.

        1. Appelez votre moteur d’inférence qui utilise un modèle entraîné pour effectuer des inférences sur les messages entrants.
        1. Recevez des résultats d’inférence du moteur, regroupez-les en tant qu’exemple de fichier multimédia et soumettez-les à Live Video Analytics à l’aide du fichier [inferencing.proto](https://github.com/Azure/live-video-analytics/blob/master/contracts/grpc/inferencing.proto).

            Vous pouvez également appeler une fonction de transformation multimédia sur l’exemple de fichier multimédia.
1. Déployez l’implémentation du serveur gRPC. Pour cela, deux méthodes sont à votre disposition :

    1. Déployez en tant que module IoT colocalisé avec le module Live Video Analytics.
    1. Déployez en tant que module IoT sur un nœud accessible via le réseau (en local ou sur le cloud) qui peut échanger des données avec le module Live Video Analytics.
1. Configurez une topologie de graphe Live Video Analytics à l’aide du module Live Video Analytics et pointez sur le serveur gRPC.

### <a name="recommendation"></a>Recommendation :

En cas de colocalisation sur le même nœud, la mémoire partagée peut être utilisée pour bénéficier de performances optimales. Vous devez pour cela utiliser les fonctionnalités de mémoire partagée Linux exposées par le langage/l’environnement de programmation.

1. Ouvrez le descripteur de la mémoire partagée Linux.
1. Lors de la réception d’une image, accédez au décalage d’adresse dans la mémoire partagée.
1. Accusez réception de la fin du traitement de l’image afin que sa mémoire puisse être récupérée par Live Video Analytics.

## <a name="create-a-grpc-inference-server"></a>Créer un serveur d’inférence gRPC

Vous allez maintenant créer un module IoT Edge (IA externe) qui accepte les images vidéo de Live Video Analytics à l’aide de messages [protobuf](https://github.com/Azure/live-video-analytics/tree/master/contracts/grpc) via la mémoire partagée, classer les images comme « sombres » ou « claires » et retourner les résultats d’inférence au récepteur de messages IoT Hub dans Live Video Analytics à l’aide du [schéma de métadonnées d’inférence](inference-metadata-schema.md).

:::image type="content" source="./media/develop-deploy-grpc-inference-srv-how-to/external-ai.png" alt-text="Créer un module de IoT Edge (IA externe)":::

Ce serveur d’inférence gRPC est une application console .NET Core qui gère les messages [protobuf](https://github.com/Azure/live-video-analytics/tree/master/contracts/grpc) envoyés entre Live Video Analytics et votre IA personnalisée. Voici le flux des messages entre Live Video Analytics et le serveur d’inférence gRPC :

1. Live Video Analytics envoie un descripteur de flux multimédia (voir [extension.proto](https://github.com/Azure/live-video-analytics/blob/master/contracts/grpc/extension.proto)) qui définit les informations de flux multimédia qui seront envoyées, suivi des images vidéo envoyées au serveur en tant que message [protobuf](https://github.com/Azure/live-video-analytics/tree/master/contracts/grpc) sur la session de flux gRPC. 
1. Le serveur valide et accuse réception du descripteur de flux et définit la méthode de transfert de données souhaitée.
1. Live Video Analytics commence alors à envoyer les fichiers MediaSample qui contiennent les images vidéo.
1. Le serveur analyse les images vidéo au fur et à mesure qu’il les reçoit et commence à les traiter à l’aide d’un processeur d’images que vous avez défini.
1. Le serveur retourne ensuite les résultats d’inférence en tant que messages [protobuf](https://github.com/Azure/live-video-analytics/tree/master/contracts/grpc) dès qu’ils sont disponibles. 

    :::image type="content" source="./media/develop-deploy-grpc-inference-srv-how-to/grpc-external-srv.png" alt-text="Créer un serveur d’inférence gRPC":::

Les images vidéo peuvent être transférées via la [mémoire partagée](https://en.wikipedia.org/wiki/Shared_memory#:~:text=In%20computer%20science%2C%20shared%20memory,of%20passing%20data%20between%20programs.) ou peuvent être incorporées dans le message protobuf. Le mode de transfert de données peut être configuré dans la topologie du graphe LVA pour déterminer le mode de transfert des images. Pour cela, vous devez configurer l’élément **dataTransfer** de la propriété MediaGraphGrpcExtension, comme indiqué ci-dessous :

Incorporé :

```
"dataTransfer": {
              "mode": "Embedded"
            }
```

Mémoire partagée :

```
"dataTransfer": {
              "mode": "sharedMemory",
              "SharedMemorySizeMiB": "20"
            }
```

> [!NOTE]
> Lors de la communication sur une mémoire partagée, la valeur de IpcMode doit être définie sur **shareable** et dans le module du serveur gRPC, définissez la valeur de IpcMode sur **container:{NOM_CONTENEUR}** . Ces paramètres doivent être définis dans le fichier manifeste de déploiement utilisé pour déployer les modules dans IoT Hub Azure. Vous trouverez ci-dessous un exemple des options de conteneur à utiliser lors de la configuration des modules IoT Edge.

Module Live Video Analytics :

```
{
    "HostConfig": {
        "LogConfig": {
            "Config": {
                "max-size": "10m",
                "max-file": "10"
            }
        },
        "IpcMode": "shareable"
    }
}
```

Module d’extension gRPC :

```
{
    "HostConfig": {
        "LogConfig": {
            "Config": {
                "max-size": "10m",
                "max-file": "10"
            }
        },
        "IpcMode": "container:lvaEdge"
    }
}
```

> [!NOTE]
> Vérifiez que vous pouvez accéder à la zone de mémoire partagée de « container:lvaEdge » dans grpcExtension.

## <a name="sample-grpc-server"></a>Exemple de serveur gRPC

Pour comprendre la façon dont le serveur gRPC est développé, passons en revue notre exemple de code.

1. Clonez le dépôt à partir du lien GitHub [https://github.com/Azure-Samples/live-video-analytics-iot-edge-csharp](https://github.com/Azure-Samples/live-video-analytics-iot-edge-csharp).
1. Lancez VSCode et accédez au dossier /src/edge/modules/grpcExtension.
1. Nous allons parcourir rapidement les fichiers :

    1. **Program.cs**: il s’agit du point d’entrée de l’application. Il est responsable de l’initialisation et de la gestion du serveur gRPC, qui agira en tant qu’hôte. Dans notre exemple, le port à écouter pour les messages gRPC entrants à partir d’un client gRPC (tel que Live Video Analytics) est spécifié par l’élément de configuration grpcBindings dans AppConfig.json.
    
        ```json    
        {
          "grpcBinding": "tcp://0.0.0.0:5001"
        }
        ```
    1. **Services\MediaGraphExtensionService.cs** : cette classe est chargée de gérer les messages [protobuf](https://github.com/Azure/live-video-analytics/tree/master/contracts/grpc). Elle lit l’image dans le message, appelle le processeur d’images (ImageProcessor) et écrit les résultats de l’inférence.
Maintenant que vous avez configuré et initialisé les connexions de port du serveur gRPC, voyons comment traiter les messages gRPC entrants.

        * Une fois qu’une session gRPC est établie, le tout premier message que le serveur gRPC reçoit du client (Live Video Analytics) est un MediaStreamDescriptor qui est défini dans le fichier [extension.proto](https://github.com/Azure/live-video-analytics/blob/master/contracts/grpc/extension.proto). 

            ```
            message MediaStreamDescriptor {
              oneof stream_identifier {
                GraphIdentifier graph_identifier = 1;     // Media Stream graph identifier
                string extension_identifier = 2;          // Media Stream extension identifier
              }
              MediaDescriptor media_descriptor = 5;       // Session media information.
              // Additional data transfer properties. If none is set, it is assumed
              // that all content will be transferred through messages (embedded transfer).
              oneof data_transfer_properties {
                SharedMemoryBufferTransferProperties shared_memory_buffer_transfer_properties = 10;
              }
            }
            ```
        * Dans notre implémentation de serveur, la méthode `ProcessMediaStreamDescriptor` valide la propriété MediaDescriptor de MediaStreamDescriptor pour un fichier vidéo, puis configure le mode de transfert de données (qui utilise la mémoire partagée ou le mode de transfert d’image incorporée) selon les valeurs que vous spécifiez dans la topologie et le fichier de modèle de déploiement utilisé. 
        * Lors de la réception du message et de la configuration du mode de transfert de données, le serveur gRPC renvoie le message MediaStreamDescriptor au client en tant qu’accusé de réception et établit ainsi une connexion entre le serveur et le client.    
        * Une fois que Live Video Analytics reçoit l’accusé de réception, il commence à transférer le flux multimédia vers le serveur gRPC. Dans notre implémentation de serveur, la méthode `ProcessMediaStream` traitera le MediaStreamMessage entrant. Le MediaStreamMessage est également défini dans le fichier [extension.proto](https://github.com/Azure/live-video-analytics/blob/master/contracts/grpc/extension.proto).

            ```
            message MediaStreamMessage {
              uint64 sequence_number = 1;       // Monotonically increasing directional message identifier starting from 1 when the gRPC connection is created
              uint64 ack_sequence_number = 2;   // 0 if this message is not referencing any sent message.
            
            
              // Possible payloads are strongly defined by the contract below
              oneof payload {
                MediaStreamDescriptor media_stream_descriptor = 5;
                MediaSample media_sample = 6;
              }
            }
            ```
        * Selon la valeur de batchSize dans Appconfig.json, notre serveur continue à recevoir les messages et stocke les images vidéo dans une liste. Une fois la limite de batchSize atteinte, la fonction appelle la fonction ou le fichier qui traitera l’image. Dans notre cas, la méthode appelle un fichier nommé BatchImageProcessor.cs
    1. **Processors\BatchImageProcessor.cs** : cette classe est chargée de traiter la ou les images. Nous avons utilisé un modèle de classification d’image dans cet exemple. Pour chaque image traitée, l’algorithme utilisé est le suivant :

        1. Convertit l’image en tableau d’octets pour traitement. Consultez la méthode : `GetBytes(Bitmap image)`
        
            L’exemple de processeur que nous utilisons ne prend en charge que les trames d’image encodées JPG et « None » comme format de pixel. Si votre processeur personnalisé prend en charge un encodage et/ou un format différents, mettez à jour la méthode `IsMediaFormatSupported` de la classe du processeur.
        1. À l’aide de la classe [ColorMatrix](https://docs.microsoft.com/dotnet/api/system.drawing.imaging.colormatrix?redirectedfrom=MSDN&view=dotnet-plat-ext-3.1&preserve-view=true), convertissez l’image en nuances de gris. Consultez la méthode : `ToGrayScale(Image source)`
        1. Une fois que vous avez obtenu l’image en nuances de gris, calculez la moyenne des octets de nuances de gris.
        1. Si la valeur moyenne est inférieure à 127, classifiez l’image comme étant « sombre », sinon classifiez-le comme étant « claire » avec une valeur de confiance de 1.0. Consultez la méthode : `ProcessImage(List<Image> images)`

    Vous pouvez ajouter votre propre logique de processeur en modifiant cette classe ou en ajoutant une nouvelle classe et en implémentant la méthode :

    ```
    IEnumerable<Inference> ProcessImage(List<Image> images) 
    ```

    Une fois que vous avez ajouté la nouvelle classe, vous devez mettre à jour le fichier MediaGraphExtensionService.cs afin qu’il instancie votre classe et appelle la méthode ProcessImage sur ce dernier pour exécuter votre logique de traitement. 

## <a name="connect-with-live-video-analytics-module"></a>Se connecter avec le module Live Video Analytics

Maintenant que vous avez créé votre module d’extension gRPC, vous allez créer et déployer la topologie de graphe multimédia.

1. À l’aide de Visual Studio Code, suivez [ces instructions](https://docs.microsoft.com/azure/iot-edge/tutorial-develop-for-linux#build-and-push-your-solution) pour vous connecter à Docker.
1. Dans Visual Studio Code, accédez à src/edge. Votre fichier .env et quelques fichiers de modèle de déploiement s’affichent.

    Le modèle de déploiement fait référence au manifeste de déploiement de l’appareil de périphérie. Il inclut des valeurs d’espace réservé. Le fichier .env inclut les valeurs de ces variables.
    
    Sélectionnez ensuite Créer et envoyer (push) la solution IoT Edge. Utilisez src/edge/deployment.grpc.template.json pour cette étape.
        
    :::image type="content" source="./media/develop-deploy-grpc-inference-srv-how-to/build-push-iot-edge-solution.png" alt-text="Se connecter avec le module Live Video Analytics":::
    
    Cette action permet de créer le module de serveur grpc et d’envoyer (via push) l’image à Azure Container Registry.
    Vérifiez que les variables d’environnement CONTAINER_REGISTRY_USERNAME_myacr et CONTAINER_REGISTRY_PASSWORD_myacr sont définies dans le fichier .env.
1. Accédez au dossier src/cloud-to-device-console-app. Ce dernier contient votre fichier appsettings.json et quelques autres fichiers :

    * c2d-console-app.csproj : fichier projet pour Visual Studio Code.
    * operations.json : liste des opérations que vous voulez que le programme exécute.
    * Program.cs : exemple de code de programme. Ce code :

        * Il charge les paramètres de l’application.
        * Invoque les méthodes directes exposées par le module Live Video Analytics sur IoT Edge. Vous pouvez utiliser le module pour analyser des flux vidéo en direct en invoquant ses [méthodes directes](direct-methods.md).
        * S’interrompt pour vous permettre d’examiner la sortie du programme dans la fenêtre TERMINAL et d’examiner les événements qui ont été générés par le module dans la fenêtre SORTIE.
        * Invoque des méthodes directes pour nettoyer des ressources.
1. Modifiez le fichier operations.json :

    * Changez le lien vers la topologie de graphe :

        * `"topologyUrl" : https://raw.githubusercontent.com/Azure/live-video-analytics/master/MediaGraph/topologies/grpcExtension/topology.json`
        * Sous GraphInstanceSet, modifiez le nom de la topologie de graphe pour qu’il corresponde à la valeur dans le lien précédent :<br/>`"topologyName": "InferencingWithGrpcExtension"`
        * Sous GraphTopologyDelete, modifiez le nom :<br/>`"name": "InferencingWithGrpcExtension"`

            La topologie (par exemple, `https://github.com/Azure/live-video-analytics/blob/master/MediaGraph/topologies/grpcExtension/topology.json`) doit définir une adresse d’extension :
    * Paramètre d’adresse d’extension

        ```
        {
            "name": "grpcExtensionAddress",
            "type": "String",
            "description": "gRPC LVA Extension Address",
            "default": "https://<REPLACE-WITH-IP-OR-CONTAINER-NAME>/score"
        },
        ```
    * Configuration

        ```
        {
            "@apiVersion": "1.0",
            "name": "TopologyName",
            "properties": {
            "processors": [
              {
                "@type": "#Microsoft.Media.MediaGraphGrpcExtension",
                "name": "grpcExtension",
                "endpoint": {
                  "@type": "#Microsoft.Media.MediaGraphUnsecuredEndpoint",
                  "url": "${grpcExtensionAddress}",
                  "credentials": {
                    "@type": "#Microsoft.Media.MediaGraphUsernamePasswordCredentials",
                    "username": "${grpcExtensionUserName}",
                    "password": "${grpcExtensionPassword}"
                  }
                },
                "dataTransfer": {
                        "mode": "sharedMemory",
                        "SharedMemorySizeMiB": "5"
                },
                "image": {
                  "scale": {
                    "mode": "${imageScaleMode}",
                    "width": "${frameWidth}",
                    "height": "${frameHeight}"
                  },
                  "format": {
                    "@type": "#Microsoft.Media.MediaGraphImageFormatEncoded",
                    "encoding": "${imageEncoding}",
                    "quality": "${imageQuality}"
                  }
                }
            ]
          }
        }
        ```
    
## <a name="generate-and-deploy-the-iot-edge-deployment-manifest"></a>Générez et déployez le manifeste de déploiement IoT Edge

Le manifeste de déploiement définit les modules qui sont déployés sur un périphérique et les paramètres de configuration de ces modules. Effectuez les étapes suivantes pour générer un manifeste à partir du fichier de modèle, puis déployez-le sur le périphérique.
Cette étape permet de créer le manifeste de déploiement IoT Edge dans src/edge/config/deployment.grpc.amd64.json. Cliquez avec le bouton droit sur ce fichier, puis sélectionnez **Créer un déploiement pour un seul appareil**.

:::image type="content" source="./media/develop-deploy-grpc-inference-srv-how-to/create-deployment-single-device.png" alt-text="Générer et déployer le manifeste de déploiement IoT Edge":::

Visual Studio Code vous demande ensuite de sélectionner un appareil IoT Hub. Sélectionnez votre appareil IoT Edge, à savoir lva-sample-device.
À ce stade, le déploiement des modules de périphérie sur votre appareil IoT Edge a démarré. Après environ 30 secondes, actualisez Azure IoT Hub dans la section inférieure gauche de Visual Studio Code. Vous devez voir qu’un nouveau module a été déployé nommé lvaExtension.

:::image type="content" source="./media/develop-deploy-grpc-inference-srv-how-to/devices.png" alt-text="Un nouveau module A été déployé et il est nommé lvaExtension":::

## <a name="next-steps"></a>Étapes suivantes

Suivez les étapes de **préparation de l’analyse des événements** mentionnées dans le démarrage rapide [Analyser une vidéo en direct avec votre modèle](use-your-model-quickstart.md) pour exécuter l’exemple et interpréter les résultats. Consultez également nos exemples de topologies gRPC : [gRPCExtension](https://github.com/Azure/live-video-analytics/blob/master/MediaGraph/topologies/grpcExtension/topology.json), [CVRWithGrpcExtension](https://github.com/Azure/live-video-analytics/blob/master/MediaGraph/topologies/cvr-with-grpcExtension/topology.json), [EVRtoAssetsByGrpcExtension et [EVROnMotionPlusGrpcExtension](https://github.com/Azure/live-video-analytics/blob/master/MediaGraph/topologies/motion-with-grpcExtension/topology.json).

