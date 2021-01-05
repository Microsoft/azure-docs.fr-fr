---
title: Architecture de l’analytique en magasin
description: Apprenez à créer une application d'analytique en magasin à l’aide du modèle d’application Validation dans IoT Central
author: avneets
ms.author: avneets
ms.date: 10/13/2019
ms.topic: overview
ms.service: iot-central
ms.subservice: iot-central-retail
services: iot-central
manager: eliotgra
ms.openlocfilehash: 3486b636325393f9c8e96669ba3d21e7d489efee
ms.sourcegitcommit: fa807e40d729bf066b9b81c76a0e8c5b1c03b536
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97347980"
---
# <a name="in-store-analytics-architecture"></a>Architecture de l'analytique en magasin


Les solutions d'analytique en magasin vous permettent de surveiller différentes conditions d'un environnement de magasin de détail. Ces solutions peuvent être générées à l’aide d’un des modèles d’application dans IoT Central et de l’architecture ci-dessous.


![Analytique en magasin Azure IoT Central](./media/architecture/store-analytics-architecture-frame.png)

- Ensemble de capteurs IoT envoyant des données de télémétrie à un appareil de passerelle
- Les appareils de passerelle envoient des données de télémétrie et des insights agrégés à IoT Central
- Exportation continue des données à destination du service Azure souhaité à des fins de manipulation
- Les données peuvent être structurées dans le format souhaité et envoyées à un service de stockage
- Les applications métier peuvent interroger des données et générer des insights pour les opérations de vente au détail
 
Intéressons-nous aux composants clés qui ont généralement un rôle à jouer dans une solution d'analytique en magasin.

## <a name="condition-monitoring-sensors"></a>Capteurs de surveillance de l’état

Une solution IoT commence par un ensemble de capteurs capturant des signaux significatifs à partir d’un environnement de magasin de détail. Il est représenté par différentes sortes de capteurs à l’extrême gauche du diagramme d’architecture ci-dessus.

## <a name="gateway-devices"></a>Appareils de passerelle

De nombreux capteurs IoT peuvent alimenter directement les signaux bruts vers le cloud ou un appareil de passerelle à proximité. L’appareil de passerelle effectue l’agrégation des données en périphérie avant d’envoyer un récapitulatif des insights à une application IoT Central. Le cas échéant, les appareils de passerelle sont également chargés de relayer les opérations de commande et de contrôle vers les capteurs. 

## <a name="iot-central-application"></a>Application IoT Central

L’application Azure IoT Central ingère les données de différentes sortes de capteurs IoT et d’appareils de passerelle situés dans l’environnement du magasin de détail. Ces données permettent d’obtenir des insights intéressants.

Azure IoT Central offre également une expérience personnalisée au responsable du magasin, en lui permettant de surveiller et de gérer à distance les appareils de l'infrastructure.

## <a name="data-transform"></a>Transformation des données
Au sein d'une solution, l’application Azure IoT Central peut être configurée pour exporter des insights bruts ou agrégés vers un ensemble de services Azure PaaS capables de manipuler les données et d'enrichir ces insights avant de les transférer à une application métier. 

## <a name="business-application"></a>Applications métier
Les données IoT peuvent être utilisées pour alimenter différentes sortes d’applications métier déployées dans un environnement de vente au détail. Un responsable ou un employé de magasin de détail peut utiliser ces applications pour visualiser les insights métier et prendre des mesures significatives en temps réel. Pour savoir comment créer un tableau de bord Power BI en temps réel pour votre équipe de vendeurs, suivez [ce tutoriel](./tutorial-in-store-analytics-create-app.md).

## <a name="next-steps"></a>Étapes suivantes
* Prise en main des modèles d'application [Analytique dans le magasin](https://aka.ms/checkouttemplate) et [Surveillance des conditions d’analytique en magasin](https://aka.ms/conditiontemplate). 
* Suivez un [tutoriel de bout en bout](https://aka.ms/storeanalytics-tutorial) qui vous guide tout au long de la génération d’une solution utilisant l’un des modèles d’application d’analytique en magasin.
