---
title: Centre de microtraitement Azure IoT Central | Microsoft Docs
description: Découvrez comment créer une application de centre de microtraitement à l’aide de notre modèle d’application Centre de microtraitement IoT Central.
author: avneet723
ms.author: avneets
ms.date: 10/13/2019
ms.topic: overview
ms.service: iot-central
ms.subservice: iot-central-retail
services: iot-central
manager: eliotgra
ms.openlocfilehash: 9a0d91677770e89f3e249fc699057332cccaa323
ms.sourcegitcommit: fa807e40d729bf066b9b81c76a0e8c5b1c03b536
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97347470"
---
# <a name="micro-fulfillment-center-architecture"></a>Architecture du centre de microtraitement

Les solutions du type Centre de microtraitement vous permettent de vous connecter, de superviser et de gérer numériquement tous les aspects d’un centre de traitement entièrement automatisé en vue de réduire les coûts en éliminant les temps d’arrêt et en augmentant la sécurité et l’efficacité globale. Ces solutions peuvent être générées à l’aide d’un des modèles d’application dans IoT Central et de l’architecture ci-dessous.

![Analytique en magasin Azure IoT Central](./media/architecture/micro-fulfillment-center-architecture-frame.png)

1. Ensemble de capteurs IoT envoyant des données de télémétrie à un appareil de passerelle
2. Les appareils de passerelle envoient des données de télémétrie et des insights agrégés à IoT Central
3. Exportation continue des données à destination du service Azure souhaité à des fins de manipulation
4. Les données peuvent être structurées dans le format souhaité et envoyées à un service de stockage
5. Les applications métier peuvent interroger des données et générer des insights pour les opérations de vente au détail
 
Intéressons-nous aux composants clés qui ont généralement un rôle à jouer dans une solution de type Centre de microtraitement.

## <a name="robotic-carriers"></a>Robots de transport

Une solution de centre de microtraitement dispose généralement d’un grand nombre de robots porteurs qui génèrent différentes sortes de signaux de télémétrie. Ces données peuvent être ingérées par un appareil de passerelle, puis agrégées et envoyées à IoT Central, comme vous pouvez le voir sur la gauche du diagramme d’architecture.  

## <a name="condition-monitoring-sensors"></a>Capteurs de surveillance de l’état

Une solution IoT commence par un ensemble de capteurs qui capturent des données significatives au sein de votre centre de traitement. Il est représenté par différentes sortes de capteurs à l’extrême gauche du diagramme d’architecture ci-dessus.

## <a name="gateway-devices"></a>Appareils de passerelle

De nombreux capteurs IoT peuvent alimenter directement les signaux bruts vers le cloud ou un appareil de passerelle à proximité. L’appareil de passerelle effectue l’agrégation des données en périphérie avant d’envoyer un récapitulatif des insights à une application IoT Central. Le cas échéant, les appareils de passerelle sont également chargés de relayer les opérations de commande et de contrôle vers les capteurs. 

## <a name="iot-central-application"></a>Application IoT Central

L’application Azure IoT Central ingère les données de différentes sortes de capteurs et robots IoT et d’appareils de passerelle situés dans l’environnement du centre de traitement. Ces données permettent d’obtenir des insights intéressants.

Azure IoT Central offre également une expérience personnalisée au responsable du magasin, en lui permettant de surveiller et de gérer à distance les appareils de l'infrastructure.

## <a name="data-transform"></a>Transformation des données
Au sein d’une solution, l’application Azure IoT Central peut être configurée pour exporter des insights bruts ou agrégés vers un ensemble de services Azure PaaS capables de manipuler les données et d’enrichir ces insights avant de les transférer à une application métier. 

## <a name="business-application"></a>Applications métier
Les données IoT peuvent être utilisées pour alimenter différentes sortes d’applications métier déployées dans un environnement de vente au détail. Un responsable ou un employé du centre de traitement peut utiliser ces applications pour visualiser les insights métier et prendre des mesures significatives en temps réel. Pour savoir comment créer un tableau de bord Power BI en temps réel pour votre équipe de vendeurs, suivez [ce tutoriel](./tutorial-in-store-analytics-create-app.md).

## <a name="next-steps"></a>Étapes suivantes
* Pour bien démarrer, consultez le modèle d’application [Déployer et parcourir un modèle d’application du Centre de microtraitement](https://aka.ms/checkouttemplate). 
* Suivez le [tutoriel](https://aka.ms/mfc-tutorial) qui vous guide tout au long des étapes de génération d’une solution à partir du modèle d’application Centre de microtraitement.
