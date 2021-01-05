---
title: Présentation de l’adresse IP d’une instance du Service IoT Device Provisioning (DPS) | Microsoft Docs
description: Découvrez comment interroger votre adresse du Service IoT Device Provisioning (DPS) et ses propriétés. L’adresse IP de votre instance DPS peut changer dans certains scénarios, par exemple dans le cadre d’une récupération d’urgence ou d’un basculement régional.
author: wesmc7777
ms.author: wesmc
ms.service: iot-dps
services: iot-dps
ms.topic: conceptual
ms.date: 03/12/2020
ms.openlocfilehash: f6afd5c4cc5aa0215f943979ae91389b39d449f6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "79237453"
---
# <a name="iot-hub-dps-ip-addresses"></a>Adresses IP IoT Hub DPS

Les préfixes d’adresse IP des points de terminaison publics du Service IoT Hub Device Provisioning (DPS) sont publiés régulièrement sous la [balise de service](../virtual-network/service-tags-overview.md) _AzureIoTHub_. Vous pouvez les utiliser pour contrôler la connectivité entre une instance IoT DPS et vos appareils ou ressources réseau afin d’atteindre un large éventail d’objectifs d’isolement réseau :

| Objectif | Approche |
|------|----------|
| Vérifier que vos appareils et services communiquent uniquement avec des points de terminaison IoT Hub DPS | Utilisez la balise de service _AzureIoTHub_ pour découvrir des instances IoT Hub DPS. Configurez des règles d’autorisation sur le paramètre de pare-feu de vos appareils et services pour les préfixes d’adresses IP correspondants. Configurez des règles pour supprimer le trafic vers les adresses IP de destination avec lesquelles vous ne voulez pas que les appareils et les services communiquent. |
| Vérifier que votre point de terminaison IoT Hub DPS ne reçoit que les connexions provenant de vos appareils et ressources réseau | Utilisez la [fonctionnalité de filtre IP](iot-dps-ip-filtering.md) IoT DPS afin de créer des règles de filtre pour les API d’appareil et de service DPS. Ces règles permettent d’autoriser uniquement les connexions provenant de vos appareils et des adresses IP de vos ressources réseau (voir la section [Limitations](#limitations-and-workarounds)). | 




## <a name="best-practices"></a>Meilleures pratiques

* Lorsque vous ajoutez des règles d’autorisation dans la configuration du pare-feu de vos appareils, il est préférable de fournir les [ports spécifiques utilisés par les protocoles applicables](../iot-hub/iot-hub-devguide-protocols.md#port-numbers).

* Les préfixes d’adresse IP des instances IoT DPS peuvent faire l’objet de modifications. Ces modifications sont publiées régulièrement par le biais de balises de service avant d’être prises en compte. Il est donc important que vous développiez des processus pour récupérer et utiliser régulièrement les balises de service les plus récentes. Ce processus peut être automatisé via l’[API de détection de balises de service](../virtual-network/service-tags-overview.md#service-tags-on-premises). L’API de détection de balises de service est toujours en préversion ; il peut arriver, dans certains cas, qu’elle ne génère pas la liste complète des balises et adresses IP. En attendant la mise à la disposition générale de l'API de détection, pensez à utiliser les [balises de service au format JSON téléchargeable](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files). 

* Utilisez la balise *AzureIoTHub.[nom de la région]* pour identifier les préfixes IP utilisés par les points de terminaison DPS dans une région spécifique. Pour tenir compte de la récupération d’urgence d’un centre de données ou d’un [basculement régional](../iot-hub/iot-hub-ha-dr.md), vérifiez que la connectivité aux préfixes IP de la région de géolocalisation de votre instance DPS est également activée.

* Le fait de configurer des règles de pare-feu pour une instance DPS risque de bloquer la connectivité nécessaire pour exécuter des commandes Azure CLI et PowerShell sur cette instance. Pour éviter ces problèmes de connectivité, vous pouvez ajouter des règles d’autorisation de sorte que les préfixes des adresses IP de vos clients permettent aux clients CLI ou PowerShell de communiquer à nouveau avec votre instance DPS.  


## <a name="limitations-and-workarounds"></a>Limitations et solutions de contournement

* La fonctionnalité de filtre IP DPS impose une limite de 100 règles. Cette limite peut être augmentée sur demande via le support client Azure. 

* Vos [règles de filtrage IP](iot-dps-ip-filtering.md) configurées ne sont appliquées que sur vos points de terminaison DPS, et non sur les points de terminaison IoT Hub liés. Le filtrage IP de ces derniers doit être configuré séparément. Pour plus d’informations, consultez [Règles de filtrage IP IoT Hub](../iot-hub/iot-hub-ip-filtering.md).

## <a name="support-for-ipv6"></a>Prise en charge d’IPv6 

IPv6 n’est pas pris en charge à l’heure actuelle sur IoT Hub ou DPS.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les configurations d’adresses IP avec DPS, consultez :

* [Configurer le filtrage d’adresse IP](iot-dps-ip-filtering.md)
