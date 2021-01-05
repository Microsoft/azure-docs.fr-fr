---
title: Créer des solutions de santé avec Azure IoT Central | Microsoft Docs
description: Apprenez à créer des solutions de santé à l'aide de modèles d'application Azure IoT Central.
author: philmea
ms.author: philmea
ms.date: 09/24/2019
ms.topic: overview
ms.service: iot-central
services: iot-central
manager: eliotgra
ms.openlocfilehash: 2aee5c56b267f9243b69a48dfd0f5f64196c5f23
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "77021484"
---
# <a name="building-healthcare-solutions-with-azure-iot-central"></a>Création de solutions de santé avec Azure IoT Central 



Apprenez à créer des solutions de santé avec Azure IoT Central, à l'aide de modèles d'application.

## <a name="what-is-continuous-patient-monitoring-template"></a>Qu'est-ce que le modèle de surveillance continue des patients ?

Dans l'espace IoT dédié à la santé, la Surveillance continue des patients est un élément clé pour réduire le risque de réadmission, gérer plus efficacement les maladies chroniques et améliorer les résultats des patients. La surveillance continue des patients peut être divisée en deux grandes catégories :

1. La **surveillance des patients hospitalisés** : grâce à des dispositifs médicaux portables et à d'autres appareils disponibles à l'hôpital, les équipes de soins peuvent surveiller les constantes vitales et l'état de santé des patients. Il n'est ainsi plus nécessaire d'envoyer une infirmière au chevet du patient plusieurs fois par jour. Et des notifications permettent aux équipes de soins de déterminer quand un patient a besoin d'une attention particulière, ce qui les aide à mieux hiérarchiser leur temps.
1. La **surveillance à distance des patients** : l’utilisation de dispositifs médicaux portables et des résultats rapportés par les patients pour suivre ceux-ci en dehors de l’hôpital peut réduire le risque de réadmission. Les données des patients atteints de maladies chroniques et des patients en rééducation peuvent être recueillies pour s'assurer qu'ils respectent leur traitement et pour veiller à ce que les alertes de détérioration de leur état soient transmises aux équipes de soins avant que cet état ne devienne critique.

Ce modèle d'application peut être utilisé pour créer des solutions dans les deux catégories de surveillance continue des patients. Voici les avantages :

* Connecter de façon continue différentes sortes de dispositifs médicaux portables à une instance IoT Central
* Surveiller et gérer les appareils pour garantir leur intégrité
* Créer des règles personnalisées pour les données des appareils afin de déclencher les alertes appropriées
* Exporter les données médicales des patients vers l'API Azure pour FHIR, un magasin de données conforme
* Exporter les insights agrégés vers des applications métier existantes ou nouvelles

>[!div class="mx-imgBorder"] 
>![CPM-dashboard](media/in-patient-dashboard.png)

## <a name="next-steps"></a>Étapes suivantes

Pour créer une solution de surveillance continue des patients :

* [Déployer le modèle d'application](tutorial-continuous-patient-monitoring.md)
* [Découvrir un exemple d'architecture](concept-continuous-patient-monitoring-architecture.md)