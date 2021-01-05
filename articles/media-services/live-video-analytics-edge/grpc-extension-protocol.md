---
title: Protocole d’extension gRPC – Azure
description: Dans cet article, vous allez apprendre à utiliser le protocole d’extension gRPC pour envoyer des messages entre le module Live Video Analytics et votre extension personnalisée d’IA ou de validation croisée (CV).
ms.topic: overview
ms.date: 09/14/2020
ms.openlocfilehash: 7f21ff358b8dd5ac540de8c39c37c52e98977e59
ms.sourcegitcommit: cc13f3fc9b8d309986409276b48ffb77953f4458
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97401625"
---
# <a name="grpc-extension-protocol"></a>Protocole d’extension gRPC

Live Video Analytics sur IoT Edge vous permet d’étendre les fonctionnalités de traitement de graphe multimédia via un [nœud d’extension de graphe](https://review.docs.microsoft.com/en-us/azure/media-services/live-video-analytics-edge/media-graph-extension-concept?branch=release-lva-dec-update). Si vous utilisez le processeur d’extension gRPC comme nœud d’extension, la communication entre le module Live Video Analytics et votre module d’IA ou de validation croisée (CV) se fait via le protocole structuré gRPC, hautement performant.

Dans cet article, vous allez apprendre à utiliser le protocole d’extension gRPC pour envoyer des messages entre le module Live Video Analytics et votre extension personnalisée d’IA ou de validation croisée (CV).

gRPC est un framework RPC moderne, open source et extrêmement performant qui s’exécute dans n’importe quel environnement et prend en charge les communications multiplateforme et interlangage. Le service de transport gRPC utilise le streaming bidirectionnel HTTP/2 entre :

* le client gRPC (Live Video Analytics sur un module IoT Edge) et 
* le serveur gRPC (votre extension personnalisée).

Une session gRPC est une connexion unique du client gRPC au serveur gRPC via le port TCP/TLS. 

Dans une session unique : le client envoie un descripteur de flux multimédia, suivi d’images vidéo, au serveur sous la forme d’un message [protobuf](https://github.com/Azure/live-video-analytics/tree/master/contracts/grpc) via la session de flux gRPC. Le serveur valide le descripteur de flux, analyse l’image vidéo et retourne les résultats d’inférence sous la forme d’un message protobuf. 

Il est fortement recommandé de retourner les réponses à l’aide de documents JSON valides, conformes au schéma préétabli défini comme étant le [modèle d’objet du schéma des métadonnées d’inférence](https://review.docs.microsoft.com/en-us/azure/media-services/live-video-analytics-edge/inference-metadata-schema?branch=release-lva-dec-update). L’interopérabilité avec d’autres composants et les futures fonctionnalités ajoutées au module Live Video Analytics est ainsi mieux garantie.

![Contrat d’extension gRPC](./media/grpc-extension-protocol/grpc.png)

## <a name="implementing-grpc-protocol"></a>Implémentation du protocole gRPC

### <a name="creating-a-grpc-connection"></a>Création d’une connexion gRPC

L’extension personnalisée doit implémenter le service gRPC suivant :

```
service MediaGraphExtension
    {
        rpc ProcessMediaStream(stream MediaStreamMessage) returns (stream MediaStreamMessage);
    }
```

Lorsqu’il est appelé, il ouvre un flux bidirectionnel pour les messages entre l’extension gRPC et le graphe Live Video Analytics. Le premier message envoyé dans ce flux par chaque tiers contient un MediaStreamDescriptor, qui définit les informations qui seront envoyées dans les messages MediaSample suivants.

Par exemple, l’extension de graphe peut envoyer le message (exprimé ici en JSON) pour indiquer qu’elle enverra des images 416x416 encodées en rgb24, incorporées aux messages gRPC, à l’extension personnalisée.

```
 {
    "sequence_number": 1,
    "ack_sequence_number": 0,
    "media_stream_descriptor": 
    {
        "graph_identifier": 
        {
            "media_services_arm_id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/resourceGroupName/providers/microsoft.media/mediaservices/mediaAccountName",
            "graph_instance_name": "mediaGraphName",
            "graph_node_name": "grpcExtension"
        },
        "media_descriptor": 
        {
            "timescale": 90000,
            "video_frame_sample_format": 
            {
                "encoding": "RAW",
                "pixel_format": "RGB24",
                "dimensions": 
                {
                    "width": 416,
                    "height": 416
                },
                "stride_bytes": 1248
            }
        }
    }
}
```

L’extension personnalisée, en réponse, envoie le message suivant pour indiquer qu’elle retourne uniquement des inférences.

```
{
    "sequence_number": 1,
    "ack_sequence_number": 1,
    "media_stream_descriptor": 
    {
        "extension_identifier": "customExtensionName"    
    }
}
```

Maintenant que les deux côtés ont échangé des descripteurs multimédias, Live Video Analytics commence à transmettre des images à l’extension.

> [!NOTE]
> L’implémentation côté serveur gRPC peut être effectuée dans le langage de programmation de votre choix.
### <a name="sequence-numbers"></a>Numéros de séquence

Le nœud d’extension gRPC et l’extension personnalisée conservent un ensemble distinct de numéros de séquence, qui sont affectés à leurs messages. Les numéros de séquence doivent augmenter de façon monotone à partir de 1. `ack_sequence_number` peut être ignoré si aucun message n’est en cours de reconnaissance, ce qui peut se produire lorsque le premier message est envoyé.

Une demande doit être reconnue une fois que le message correspondant a été entièrement traité. Dans le cas d’un transfert de mémoire partagée, cette reconnaissance indique que l’expéditeur peut libérer la mémoire partagée et que le destinataire ne pourra plus y accéder. L’expéditeur doit conserver toute mémoire partagée jusqu’à ce qu’elle soit reconnue.

### <a name="reading-embedded-content"></a>Lecture du contenu incorporé

Le contenu incorporé peut être lu directement à partir du message `MediaSample` via le champ `content_byte`. Les données sont encodées en fonction de `MediaDescriptor`.

### <a name="reading-content-from-shared-memory"></a>Lecture du contenu à partir de la mémoire partagée

Lors du transfert de contenu via la mémoire partagée, l’expéditeur inclut `SharedMemoryBufferTransferProperties` dans son `MediaStreamDescriptor` lorsqu’il établit une session pour la première fois. Cela peut se présenter comme suit (exprimé en JSON) :

```
{
  "handle_name": "inference_client_share_memory_2146989006636459346"
  "length_bytes": 20971520
}
```

Le récepteur ouvre alors le fichier `/dev/shm/inference_client_share_memory_2146989006636459346`. L’expéditeur envoie un message `MediaSample`, qui contient un `ContentReference` faisant référence à un emplacement spécifique dans le fichier.

```
{
    "timestamp": 143598615750000,
    "content_reference": 
    {
        "address_offset": 519168,
        "length_bytes": 173056
    }
}
```

Le récepteur lit ensuite les données à partir de cet emplacement dans le fichier.

Pour que le conteneur Live Video Analytics communique au sujet de la mémoire partagée, le mode IPC du conteneur doit être configuré correctement. Cela peut être fait de nombreuses façons, mais voici quelques configurations recommandées :

* Lors de la communication avec un moteur d’inférence gRPC en cours d’exécution sur l’appareil hôte, le mode IPC doit être défini pour héberger.
* Lors de la communication avec un serveur gRPC en cours d’exécution dans un autre module IoT Edge, le mode IPC doit être défini comme partageable pour le module Live Video Analytics et `container:liveVideoAnalytics` pour l’extension personnalisée, où `liveVideoAnalytics` est le nom du module Live Video Analytics.

Voici à quoi cela peut ressembler dans le jumeau d’appareil à l’aide de la première option ci-dessus.

```
"liveVideoAnalytics": 
{
  "version": "1.0",
  "type": "docker",
  "status": "running",
  "restartPolicy": "always",
  "settings": 
  {
    "image": "mcr.microsoft.com/media/live-video-analytics:1",
    "createOptions": 
      "HostConfig": 
      {
        "IpcMode": "host"
      }
  }
}
```

Pour plus d’informations sur les modes IPC, consultez https://docs.docker.com/engine/reference/run/#ipc-settings---ipc.

## <a name="mediagraph-grpc-extension-contract-definitions"></a>Définitions du contrat d’extension gRPC pour MediaGraph

Cette section définit le contrat gRPC qui définit le flux de données.

### <a name="protocol-messages"></a>Messages de protocole

![Messages de protocole](./media/grpc-extension-protocol/grpc2.png)
 
### <a name="client-authentication"></a>Authentification du client

Les implémenteurs d’extensions personnalisées peuvent valider l’authenticité des connexions gRPC entrantes pour s’assurer qu’elles proviennent du nœud d’extension gRPC. Le nœud fournira une entrée dans les en-têtes de demande à utiliser pour la validation.

Des informations d’identification nom d’utilisateur/mot de passe peuvent être utilisées pour ce faire. Lorsque vous créez un nœud d’extension gRPC, les informations d’identification sont fournies comme ci-dessous :

```
{
  "@type": "#Microsoft.Media.MediaGraphGrpcExtension",
  "name": "{moduleIdentifier}",
  "endpoint": 
  {
    "@type": "#Microsoft.Media.MediaGraphUnsecuredEndpoint",
    "url": "tcp://customExtension:8081",
    "credentials": 
    {
      "@type": "#Microsoft.Media.MediaGraphUsernamePasswordCredentials",
      "username": "username",
      "password": "password"
    }
  }
  // Other fields omitted
}
```

Lorsque la demande gRPC est envoyée, l’en-tête suivant est inclus dans les métadonnées de la demande, imitant l’authentification HTTP de base.

`x-ms-authentication: Basic (Base64 Encoded username:password)`


## <a name="configuring-inference-server-for-each-mediagraph-over-grpc-extension"></a>Configuration du serveur d’inférence pour chaque instance MediaGraph via l’extension gRPC
Quand vous configurez votre serveur d’inférence, vous n’êtes pas obligé d’exposer un nœud pour chaque modèle IA qui est empaqueté dans le serveur d’inférence. Pour une instance de graphe, vous pouvez utiliser à la place la propriété `extensionConfiguration` du nœud `MediaGraphGrpcExtension` et définir la façon de sélectionner le ou les modèles IA. Pendant l’exécution, Live Video Analytics transmet cette chaîne au serveur d’inférence qui peut l’utiliser pour appeler le modèle IA souhaité. Cette propriété `extensionConfiguration` est une propriété facultative, elle est propre au serveur. La propriété peut être utilisée comme indiqué ci-dessous :
```
{
  "@type": "#Microsoft.Media.MediaGraphGrpcExtension",
  "name": "{moduleIdentifier}",
  "endpoint": 
  {
    "@type": "#Microsoft.Media.MediaGraphUnsecuredEndpoint",
    "url": "${grpcExtensionAddress}",
    "credentials": 
    {
      "@type": "#Microsoft.Media.MediaGraphUsernamePasswordCredentials",
      "username": "${grpcExtensionUserName}",
      "password": "${grpcExtensionPassword}"
    }
  },
    // Optional server configuration string. This is server specific 
  "extensionConfiguration": "{Optional extension specific string}",
  "dataTransfer": 
  {
    "mode": "sharedMemory",
    "SharedMemorySizeMiB": "5"
  }
    //Other fields omitted
}
```

## <a name="using-grpc-over-tls"></a>Utilisation de gRPC via TLS

Une connexion gRPC utilisée pour l’inférence peut être sécurisée via TLS. C’est utile quand la sécurité du réseau entre Live Video Analytics et le moteur d’inférence ne peut pas être garantie. TLS chiffre tout le contenu incorporé dans les messages gRPC, ce qui entraîne un temps processeur supplémentaire lors de la transmission des images à un débit élevé.

Les options de vérification IgnoreHostname et IgnoreSignature ne sont pas prises en charge par gRPC. Par conséquent, le certificat de serveur, que le moteur d’inférence présente, doit contenir un CN correspondant exactement à l’adresse IP/nom d’hôte dans l’URL du point de terminaison du nœud d’extension gRPC.

## <a name="next-steps"></a>Étapes suivantes

[En savoir plus sur le schéma des métadonnées d’inférence](inference-metadata-schema.md)
