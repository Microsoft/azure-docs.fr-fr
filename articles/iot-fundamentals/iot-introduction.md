---
title: Introduction à l’Internet des objets Azure (IoT)
description: Introduction expliquant les principes de base d’Azure IoT et des services IoT avec notamment des exemples qui illustrent l’utilisation de l’IoT.
author: dominicbetts
ms.service: iot-fundamentals
services: iot-fundamentals
ms.topic: overview
ms.date: 01/15/2020
ms.author: dobett
ms.custom:
- amqp
- mqtt
ms.openlocfilehash: d2afd88500801f1e2b1e4da5a1e267bddfa69fc7
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96453018"
---
# <a name="what-is-azure-internet-of-things-iot"></a>Qu’est-ce que l’Internet des objets Azure (IoT) ?

L’Internet des objets Azure (IoT) est une collection de services cloud gérés par Microsoft qui permettent de connecter, superviser et contrôler des milliards de ressources IoT. Plus simplement, une solution IoT est constituée d’un ou plusieurs appareils IoT qui communiquent avec un ou plusieurs services back-end hébergés dans le cloud. 

## <a name="iot-devices"></a>Appareils IoT

Un appareil IoT est généralement constitué d’un circuit avec des capteurs attachés qui se connectent à Internet par WiFi. Par exemple :

* Capteur de pression sur une pompe à huile distante
* Capteurs de température et d’humidité sur un climatiseur
* Accéléromètre dans un ascenseur
* Capteurs de présence dans une salle

Pour créer votre solution, vous disposez d’un large éventail d’appareils de différents fabricants. Pour obtenir la liste des appareils certifiés à utiliser avec Azure IoT Hub, consultez le [catalogue d’appareils Microsoft Azure Certified pour IoT](https://catalog.azureiotsolutions.com/alldevices). Pour le prototypage, vous pouvez utiliser des appareils tels qu’un [DevKit IoT MXChip](https://microsoft.github.io/azure-iot-developer-kit/) ou un [Raspberry Pi](https://www.raspberrypi.org/). Le DevKit dispose de capteurs de température, de pression et d’humidité intégrés ainsi que d’un gyroscope, d’un accéléromètre et d’un magnétomètre. Le Raspberry Pi vous permet d’attacher de nombreux types de capteurs. 

Microsoft fournit des kits [SDK d’appareil](../iot-hub/iot-hub-devguide-sdks.md) open source que vous pouvez utiliser pour créer les applications qui s’exécutent sur vos appareils. Ces kits [SDK simplifient et accélèrent](https://azure.microsoft.com/blog/benefits-of-using-the-azure-iot-sdks-in-your-azure-iot-solution/) le développement de vos solutions IoT.

## <a name="communication"></a>Communication

En règle générale, les appareils IoT envoient la télémétrie des capteurs aux services back-end dans le cloud. Toutefois, d’autres types de communication sont possibles, par exemple un service back-end qui envoie des commandes à vos appareils. Voici quelques exemples de communication appareil-à-cloud et cloud-à-appareil :

* Un camion frigorifique mobile envoie la température à un hub IoT toutes les 5 minutes. 

* Le service back-end envoie une commande à un appareil pour changer sa fréquence d’envoi de la télémétrie en vue de diagnostiquer un problème. 

* Un appareil envoie des alertes basées sur les valeurs lues de ses capteurs. Par exemple, un appareil qui supervise un réacteur discontinu dans une usine chimique envoie une alerte quand la température dépasse une certaine valeur.

* Vos appareils envoient des informations qui s’afficheront dans un tableau de bord et pourront ainsi être vues par des opérateurs humains. Par exemple, dans la salle de commande d’une raffinerie, la température, la pression et les débits de chaque tuyau peuvent être affichés, ce qui permet aux opérateurs de superviser l’installation. 

Les kits [IoT Device SDK](../iot-hub/iot-hub-devguide-sdks.md) et IoT Hub prennent en charge les [protocoles de communication](../iot-hub/iot-hub-devguide-protocols.md) courants comme HTTP, MQTT et AMQP.

Les appareils IoT ont des caractéristiques différentes de celles d’autres clients tels que les navigateurs et les applications mobiles. Les kits SDK d’appareil vous permettent de connecter des appareils au service back-end de manière fiable et sécurisée, ce qui peut parfois représenter un véritable défi.  Plus précisément, les appareils IoT :

* sont souvent des systèmes intégrés, qui ne font appel à aucun opérateur humain (contrairement à un téléphone) ;
* peuvent être déployés sur des sites distants avec un accès physique coûteux ;
* sont accessibles uniquement via le serveur principal de la solution ;
* peuvent avoir des performances et/ou des ressources de traitement limitées ;
* peuvent avoir une connectivité réseau intermittente, lente ou coûteuse ;
* peuvent nécessiter l’utilisation des protocoles d’application personnalisés, propriétaires ou spécifiques à un secteur.

## <a name="back-end-services"></a>Services back-end 

Voici quelques exemples de fonctionnalités fournies par le service back-end dans le cadre d’une solution IoT :

* Recevoir des données de télémétrie à grande échelle de vos appareils et déterminer comment traiter et stocker ces données.
* Analyser la télémétrie pour fournir des insights en temps réel ou en différé.
* Envoyer des commandes depuis le cloud à un appareil spécifique. 
* Provisionner les appareils et contrôler ceux qui peuvent se connecter à votre infrastructure.
* Contrôler l’état de vos appareils et superviser leurs activités.
* Gérer le microprogramme installé sur vos appareils.

Par exemple, dans une solution de supervision à distance pour une station de pompage pétrolière, le service back-end cloud utilise la télémétrie des pompes pour identifier tout comportement anormal. Quand le service back-end identifie une anomalie, il peut renvoyer automatiquement une commande à l’appareil pour effectuer une action corrective. Ce processus génère une boucle de rétroaction automatisée entre l’appareil et le cloud qui augmente considérablement l’efficacité de la solution.

## <a name="azure-iot-examples"></a>Exemples d’utilisation d’Azure IoT

Pour obtenir des exemples concrets d’organisations qui utilisent Azure IoT, consultez les [études de cas techniques Microsoft pour l’IoT](https://microsoft.github.io/techcasestudies/#technology=IoT&sortBy=featured). 

Pour une étude approfondie de l’architecture IoT, consultez [Microsoft Azure IoT Reference Architecture](/azure/architecture/reference-architectures/iot) (Architecture de référence Microsoft Azure IoT).

## <a name="next-steps"></a>Étapes suivantes

Pour des cas d’entreprise concrets et l’architecture utilisée, consultez [Microsoft Azure IoT Technical Case Studies](https://microsoft.github.io/techcasestudies/#technology=IoT&sortBy=featured).

Pour des exemples de projets que vous pouvez essayer avec un kit IoT DevKit, consultez [IoT DevKit Project Catalog](https://microsoft.github.io/azure-iot-developer-kit/docs/projects/). 

Pour une explication plus complète des différents services et de leur utilisation, consultez [Services et technologies Azure IoT](iot-services-and-technologies.md).