---
title: Comprendre l’adresse IP de votre hub IoT | Microsoft Docs
description: Apprenez à interroger l’adresse IP de votre hub IoT et ses propriétés. L’adresse IP de votre hub IoT peut changer dans certains scénarios, par exemple dans le cadre d’une reprise d’activité ou d’un basculement régional.
author: philmea
ms.author: philmea
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 11/21/2019
ms.openlocfilehash: 1bfae8c7afbfdc6e73dd8bb17b94e6543361e9ce
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83848226"
---
# <a name="iot-hub-ip-addresses"></a>Adresses IP IoT Hub

Les préfixes d’adresse IP des points de terminaison publics de l’hub IoT sont publiés régulièrement sous la _balise de service_ [AzureIoTHub](../virtual-network/service-tags-overview.md).

> [!NOTE]
> Pour les appareils déployés au sein des réseaux locaux, Azure IoT Hub prend en charge l’intégration de la connectivité des réseaux virtuels aux points de terminaison privés. Pour plus d’informations, consultez [Prise en charge d’IoT Hub pour les réseaux virtuels](./virtual-network-support.md).


Vous pouvez utiliser ces préfixes d’adresse IP pour contrôler la connectivité entre IoT Hub et vos appareils ou ressources réseau afin d’implémenter un large éventail d’objectifs d’isolement réseau :

| Objectif | Scénarios applicables | Approche |
|------|-----------|----------|
| Vérifiez que vos appareils et services communiquent uniquement avec des points de terminaison IoT Hub | La messagerie [appareil-à-cloud](./iot-hub-devguide-messaging.md)et [cloud-à-appareil](./iot-hub-devguide-messages-c2d.md), les [méthodes directes](./iot-hub-devguide-direct-methods.md), les [jumeaux d’appareil et de module](./iot-hub-devguide-device-twins.md) et les [flux d’appareil](./iot-hub-device-streams-overview.md) | Utilisez les balises de service _AzureIoTHub_ et _EventHub_ pour découvrir IoT Hub et les préfixes d’adresse IP Event Hub, et configurez les règles d’autorisation sur les paramètres de pare-feu des appareils et des services pour ces préfixes d’adresse IP en conséquence ; supprimez le trafic vers les autres adresses IP de destination avec lesquelles vous ne voulez pas que les appareils ou services communiquent. |
| Vérifiez que votre point de terminaison d’appareil IoT Hub reçoit des connexions uniquement à partir de vos appareils et ressources réseau | La messagerie [appareil-à-cloud](./iot-hub-devguide-messaging.md)et [cloud-à-appareil](./iot-hub-devguide-messages-c2d.md), les [méthodes directes](./iot-hub-devguide-direct-methods.md), les [jumeaux d’appareil et de module](./iot-hub-devguide-device-twins.md) et les [flux d’appareil](./iot-hub-device-streams-overview.md) | Utilisez la [fonctionnalité de filtre IP](iot-hub-ip-filtering.md) d’IoT Hub pour autoriser les connexions à partir de vos appareils et des adresses IP de vos ressources réseau (voir la section [limitations](#limitations-and-workarounds)). | 
| Assurez-vous que les ressources de point de terminaison personnalisées de vos itinéraires (comptes de stockage, Service Bus et Event Hub) sont accessibles à partir de vos ressources réseau uniquement | [Routage de messages](./iot-hub-devguide-messages-d2c.md) | Suivez les instructions de votre ressource sur restreindre la connectivité (par exemple, via des [règles de pare-feu](../storage/common/storage-network-security.md), des [liens privés](../private-link/private-endpoint-overview.md) ou des [points de terminaison de service](../virtual-network/virtual-network-service-endpoints-overview.md)). Utilisez des balises de service _AzureIoTHub_ pour découvrir les préfixes d’adresses IP IoT Hub et ajoutez des règles d’autorisation pour ces préfixes IP sur la configuration du pare-feu de votre ressource (voir la section [limitations](#limitations-and-workarounds)). |



## <a name="best-practices"></a>Meilleures pratiques

* Lorsque vous ajoutez des règles d’autorisation dans la configuration du pare-feu de vos appareils, il est préférable de fournir les [ports spécifiques utilisés par les protocoles applicables](./iot-hub-devguide-protocols.md#port-numbers).

* Les préfixes d’adresse IP de l’hub IoT sont susceptibles d’être modifiés. Ces modifications sont publiées régulièrement par le biais de balises de service avant d’être prises en compte. Il est donc important que vous développiez des processus pour récupérer et utiliser régulièrement les balises de service les plus récentes. Ce processus peut être automatisé via l’[API de détection de balises de service](../virtual-network/service-tags-overview.md#service-tags-on-premises). Notez que l’API de détection de balises de service est toujours en préversion et peut, dans certains cas, ne pas générer la liste complète de balises et adresses IP. En attendant la mise à la disposition générale de l'API de détection, pensez à utiliser les [balises de service au format JSON téléchargeable](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files). 

* Utilisez la balise *AzureIoTHub.[nom de région]* pour identifier les préfixes IP utilisés par les points de terminaison de l’hub IoT dans une région spécifique. Pour tenir compte de la récupération d’urgence d’un centre de données ou de [basculement régional](iot-hub-ha-dr.md), assurez-vous que la connectivité aux préfixes IP de la région de géolocalisation de votre IoT Hub est également activée.

* Dans IoT Hub, la configuration de règles de pare-feu peut bloquer la connectivité requise pour exécuter des commandes Azure CLI et PowerShell sur votre instance IoT Hub. Pour l'éviter, vous pouvez ajouter des règles d’autorisation afin de permettre aux préfixes des adresses IP de vos clients de réactiver les clients CLI ou PowerShell pour communiquer avec votre instance IoT Hub.  


## <a name="limitations-and-workarounds"></a>Limitations et solutions de contournement

* La fonctionnalité de filtre IP d’IoT Hub a une limite de 10 règles. Cette limite peut être augmentée sur demande via le support client Azure. 

* Vos [règles de filtrage IP](iot-hub-ip-filtering.md) configurées sont appliquées uniquement sur vos points de terminaison IP IoT Hub et non sur votre point de terminaison Event Hub intégré à IoT Hub. Si vous avez également besoin d’appliquer le filtrage IP sur l’Event Hub dans lequel vos messages sont stockés, vous pouvez utiliser votre propre ressource Event Hub pour configurer directement les règles de filtrage IP souhaitées. Pour ce faire, vous devez configurer votre propre ressource Event Hub et configurer le [routage des messages](./iot-hub-devguide-messages-d2c.md) pour envoyer vos messages à cette ressource plutôt qu’à l’Event Hub intégré à votre IoT Hub. Enfin, comme indiqué dans le tableau ci-dessus, pour activer la fonctionnalité de routage des messages, vous devez également autoriser la connectivité des préfixes d’adresse IP IoT Hub à votre ressource Event Hub configurée.

* Lors du routage vers un compte de stockage, il n’est possible d’autoriser le trafic à partir des préfixes d’adresses IP IoT Hub que lorsque le compte de stockage se trouve dans une autre région que votre IoT Hub.

## <a name="support-for-ipv6"></a>Prise en charge d’IPv6 

IPv6 n’est pas pris en charge sur IoT Hub.
