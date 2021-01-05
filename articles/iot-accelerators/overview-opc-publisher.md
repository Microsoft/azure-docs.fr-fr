---
title: Définition d’OPC Publisher - Azure | Microsoft Docs
description: Cet article donne une vue d’ensemble des fonctionnalités d’OPC Publisher. Il permet de publier des données de télémétrie JSON encodées avec une charge utile JSON dans Azure IoT Hub.
author: dominicbetts
ms.author: dobett
ms.date: 06/10/2019
ms.topic: overview
ms.service: industrial-iot
services: iot-industrialiot
manager: philmea
ms.custom:
- amqp
- mqtt
ms.openlocfilehash: 49ca12ed4f408e2a3fce1c8e59f541778f35311e
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91281777"
---
# <a name="what-is-opc-publisher"></a>Qu’est-ce qu’OPC Publisher ?

> [!IMPORTANT]
> Pendant la mise à jour de cet article, consultez [Azure Industrial IoT](https://azure.github.io/Industrial-IoT/) pour obtenir le contenu le plus récent.

OPC Publisher est une implémentation de référence qui montre comment :

- Se connecter à des serveurs OPC UA existants.
- Publier les données de télémétrie encodées JSON des serveurs OPC UA au format Pub/Sub OPC UA sur Azure IoT Hub, à l’aide d’une charge utile JSON.

Vous pouvez utiliser un des protocoles de transport pris en charge par le SDK du client Azure IoT Hub : HTTPS, AMQP et MQTT.

L’implémentation de référence comprend :

- Un *client* OPC UA pour se connecter aux serveurs OPC UA existants sur votre réseau.
- Un *serveur* OPC UA sur le port 62222, que vous pouvez utiliser pour gérer ce qui est publié et qui offre à IoT Hub des méthodes directes pour en faire de même.

Vous pouvez télécharger [l’implémentation de référence d’OPC Publisher](https://github.com/Azure/iot-edge-opc-publisher) à partir de GitHub.

L’application est implémentée avec la technologie .NET Core et peut s’exécuter sur toute plateforme prise en charge par .NET Core.

OPC Publisher implémente la logique de nouvelle tentative pour établir des connexions aux points de terminaison qui ne répondent pas à un certain nombre de demandes de connexion persistante. Cela est utile si, par exemple, si un serveur OPC UA cesse de répondre en raison d’une panne d’alimentation.

Pour chaque intervalle de publication sur un serveur OPC UA, l’application crée un abonnement distinct sur lequel tous les nœuds avec cet intervalle de publication sont mis à jour.

Pour réduire la charge réseau, OPC Publisher prend en charge le traitement par lot des données envoyées à IoT Hub. Ce traitement par lot envoie un paquet à IoT Hub uniquement si la taille de paquet configurée est atteinte.

Cette application utilise la pile de référence OPC UA de l’OPC Foundation dans les packages NuGet. Consultez [https://opcfoundation.org/license/redistributables/1.3/](https://opcfoundation.org/license/redistributables/1.3/) pour les conditions des licences.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez appris ce qu’était OPC Publisher, l’étape suivante suggérée consiste à apprendre comment :

[Configurer OPC Publisher](howto-opc-publisher-configure.md)