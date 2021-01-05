---
title: Développer avec Azure IoT Hub pour des appareils contraints avec le kit SDK IoT Hub C
description: Guide du développeur - Aide sur la façon de développer avec les kits SDK Azure IoT pour les appareils contraints.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 05/24/2018
ms.author: robinsh
ms.custom:
- amqp
- mqtt
ms.openlocfilehash: 5a43eb2537ebc09ffcb524a4426d7a8c9bec560b
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96500000"
---
# <a name="develop-for-constrained-devices-using-azure-iot-c-sdk"></a>Développer pour des appareils limités avec le kit SDK Azure IoT C

Le kit SDK C Azure IoT Hub est écrit en C ANSI (C99), ce qui le rend particulièrement bien adapté pour fonctionner sur une variété de plateformes avec une empreinte disque et mémoire faible. La quantité de RAM recommandée est d’au moins 64 Ko, mais l’encombrement mémoire exact varie selon le protocole utilisé, le nombre de connexions ouvertes et la plateforme ciblée.
> [!NOTE]
> * Le SDK C Azure IoT publie régulièrement des informations sur la consommation de ressources pour faciliter le développement.  Visitez notre [dépôt GitHub](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/c_sdk_resource_information.md) et passez en revue le dernier point de référence.
>

Le SDK C est disponible sous forme de package depuis apt-get, NuGet et MBED. Pour cibler des appareils contraints, vous pouvez générer le SDK localement pour votre plateforme cible. Cette documentation explique comment supprimer certaines fonctionnalités pour réduire l’empreinte du kit SDK C avec [cmake](https://cmake.org/). Elle décrit aussi les bonnes pratiques pour les modèles de programmation permettant de travailler avec des appareils contraints.

## <a name="building-the-c-sdk-for-constrained-devices"></a>Génération du SDK C pour les appareils contraints

Générez le kit SDK C pour les appareils limités.

> [!NOTE]
> Le kit SDK Embedded C est une alternative pour les appareils avec contraintes qui prend en charge l’approche BYON (Bring Your Own Network). Les développeurs IoT ont la liberté d’apporter le client MQTT, le protocole TLS et le socket de leur choix pour créer une solution d’appareil. [En savoir plus sur le kit SDK Embedded C](https://github.com/Azure/azure-sdk-for-c/tree/master/sdk/docs/iot).

### <a name="prerequisites"></a>Prérequis

Suivez ce [guide de configuration du SDK C](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/devbox_setup.md) pour préparer votre environnement de développement à la génération du SDK C. Avant de passer à l’étape de génération avec cmake, vous pouvez appeler des indicateurs de cmake pour supprimer des fonctionnalités inutilisées.

### <a name="remove-additional-protocol-libraries"></a>Supprimer des bibliothèques de protocoles supplémentaires

À ce jour, le SDK C prend en charge cinq protocoles : MQTT, MQTT sur WebSocket, AMQPs, AMQP sur WebSocket et HTTPS. La plupart des scénarios nécessitent qu’un à deux protocoles s’exécutent sur un client : vous pouvez donc supprimer du SDK la bibliothèque du protocole que vous n’utilisez pas. Vous trouverez des informations supplémentaires sur le choix du protocole de communication approprié pour votre scénario dans [Choisir un protocole de communication IoT Hub](iot-hub-devguide-protocols.md). Par exemple, MQTT est un protocole léger qui est souvent mieux adapté pour les appareils contraints.

Vous pouvez supprimer les bibliothèques AMQP et HTTP avec la commande cmake suivante :

```
cmake -Duse_amqp=OFF -Duse_http=OFF <Path_to_cmake>
```

### <a name="remove-sdk-logging-capability"></a>Supprimer la fonctionnalité de journalisation du SDK

Le SDK C fournit une journalisation complète qui facilite le débogage. Vous pouvez supprimer la fonctionnalité de journalisation pour les appareils en production avec la commande cmake suivante :

```
cmake -Dno_logging=OFF <Path_to_cmake>
```

### <a name="remove-upload-to-blob-capability"></a>Supprimer la fonctionnalité de chargement d’objets blob

Vous pouvez charger des grands fichiers sur Stockage Azure avec la fonctionnalité intégrée dans le SDK. Azure IoT Hub joue le rôle de répartiteur pour un compte Stockage Azure associé. Vous pouvez utiliser cette fonctionnalité pour envoyer des fichiers multimédias, des grands lots de données de télémétrie et des journaux d’activité. Vous pouvez obtenir plus d’informations dans [Chargement de fichiers avec IoT Hub](iot-hub-devguide-file-upload.md). Si votre application ne nécessite pas cette fonctionnalité, vous pouvez la supprimer avec la commande cmake suivante :

```
cmake -Ddont_use_uploadtoblob=ON <Path_to_cmake>
```

### <a name="running-strip-on-linux-environment"></a>Exécution de strip sur un environnement Linux.

Si vos fichiers binaires s’exécutent sur un système Linux, vous pouvez tirer parti de la [commande strip](https://en.wikipedia.org/wiki/Strip_(Unix)) pour réduire la taille de l’application finale après la compilation.

```
strip -s <Path_to_executable>
```

## <a name="programming-models-for-constrained-devices"></a>Modèles de programmation pour les appareils contraints

Ensuite, regardez les modèles de programmation pour les appareils limités.

### <a name="avoid-using-the-serializer"></a>Éviter d’utiliser le sérialiseur

Le SDK C a un [sérialiseur SDK C](https://github.com/Azure/azure-iot-sdk-c/tree/master/serializer) facultatif, qui vous permet d’utiliser des tables de mappage déclaratives pour définir des méthodes et des propriétés de jumeau d’appareil. Le sérialiseur est conçu pour simplifier le développement, mais il ajoute une charge supplémentaire, ce qui n’est pas optimal pour les appareils contraints. Dans ce cas, vous pouvez envisager d’utiliser les API clientes primitives et d’analyser le JSON avec un analyseur léger, comme [parson](https://github.com/kgabis/parson).

### <a name="use-the-lower-layer-_ll_"></a>Utiliser la couche inférieure (_LL_)

Le SDK C prend en charge deux modèles de programmation. Un ensemble a des API avec un infixe _LL_, qui correspond à la couche inférieure. Cet ensemble d’API est léger et ne déclenche pas de threads de travail, ce qui signifie que l’utilisateur doit contrôler manuellement la planification. Par exemple, pour le client d’appareil, les API _LL_ se trouvent dans ce [fichier d’en-tête](https://github.com/Azure/azure-iot-sdk-c/blob/master/iothub_client/inc/iothub_device_client_ll.h). 

Un autre ensemble d’API sans l’index _LL_ est appelé la couche pratique, où un thread de travail est lancé automatiquement. Par exemple, vous trouverez les API de la couche pratique pour le client d’appareil dans ce [fichier d’en-tête du client d’appareil IoT](https://github.com/Azure/azure-iot-sdk-c/blob/master/iothub_client/inc/iothub_device_client.h). Pour les appareils contraints où chaque thread supplémentaire peut consommer un pourcentage important des ressources système, vous pouvez envisager d’utiliser les API _LL_.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’architecture du SDK C Azure IoT :
-    [Code source du SDK C Azure IoT](https://github.com/Azure/azure-iot-sdk-c/)
-    [Présentation d’Azure IoT device SDK pour C](iot-hub-device-sdk-c-intro.md)
