---
title: Architecture de surveillance continue des patients dans Azure IoT Central | Microsoft Docs
description: Tutoriel - Découvrez une architecture de surveillance continue des patients.
author: philmea
ms.author: philmea
ms.date: 12/11/2020
ms.topic: overview
ms.service: iot-central
services: iot-central
manager: eliotgra
ms.openlocfilehash: dad6327acd24fe7642bf1033b1527ef3e40b772e
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97590101"
---
# <a name="continuous-patient-monitoring-architecture"></a>Architecture du suivi continu des patients

Cet article décrit l’architecture d’une solution créée à partir du modèle d’application de **surveillance continue des patients** :

Vous pouvez créer des solutions de surveillance continue des patients à l’aide du modèle d’application fourni et en utilisant l’architecture décrite ci-dessous comme guide.

:::image type="content" source="media/cpm-architecture.png" alt-text="Architecture du suivi continu des patients":::

## <a name="details"></a>Détails

Cette section décrit plus en détail chaque partie du diagramme de l’architecture :

### <a name="bluetooth-low-energy-ble-medical-devices"></a>Appareils médicaux Bluetooth basse consommation (BLE, Bluetooth Low Energy)

De nombreux appareils médicaux de type wearable utilisés dans les solutions médicales IoT sont des appareils BLE. Ces appareils ne peuvent pas communiquer directement avec le cloud et doivent utiliser une passerelle pour échanger des données avec votre solution cloud. Cette architecture utilise une application de téléphone mobile comme passerelle.

### <a name="mobile-phone-gateway"></a>Passerelle de téléphone portable

La fonction principale de l’application de téléphone mobile est de collecter les données BLE des appareils médicaux et de les communiquer à IoT Central. L’application guide également les patients dans la configuration de l’appareil et leur permet de visualiser leurs données médicales personnelles. D’autres solutions pourraient utiliser une passerelle de tablette ou une passerelle statique dans une chambre d’hôpital. Un exemple d’application mobile open source est disponible pour Android et iOS, que vous pouvez utiliser comme point de départ pour le développement de votre application. Pour plus d’informations, consultez l’[application mobile de surveillance continue des patients IoT Central](/samples/iot-for-all/iotc-cpm-sample/iotc-cpm-sample/).

### <a name="export-to-azure-api-for-fhirreg"></a>Exporter vers l’API Azure pour FHIR&reg;

Azure IoT Central est conforme à HIPAA et certifié HITRUST&reg;. Vous pouvez également envoyer les données médicales des patients à d’autres services en utilisant l’[API Azure pour FHIR](../../healthcare-apis/overview.md). L’API Azure pour FHIR est une API basée sur des standards pour les données médicales cliniques. Le [connecteur Azure IoT pour FHIR](../../healthcare-apis/iot-fhir-portal-quickstart.md) vous permet d’utiliser l’API Azure pour FHIR comme destination d’exportation des données en continu depuis IoT Central.

### <a name="machine-learning"></a>Apprentissage automatique

Utilisez des modèles Machine Learning avec vos données FHIR pour générer des insights et venir en appui des décisions prises par votre équipe soignante. Pour plus d’informations, consultez la [documentation Machine Learning](../../machine-learning/index.yml).

### <a name="provider-dashboard"></a>Tableau de bord de fournisseur

Utilisez les données de la l’API Azure pour FHIR pour créer un tableau de bord des insights sur les patients ou intégrez-les directement dans un dossier médical électronique utilisé par les équipes soignantes. Les équipes soignantes peuvent utiliser le tableau de bord pour assister les patients et identifier les signes précoces avertissant d’une détérioration. Pour plus d’informations, consultez le tutoriel [Créer un tableau de bord de fournisseur Power BI](howto-health-data-triage.md).

## <a name="next-steps"></a>Étapes suivantes

L’étape suggérée suivante est de [Découvrir comment déployer un modèle d’application de surveillance continue des patients](tutorial-continuous-patient-monitoring.md).