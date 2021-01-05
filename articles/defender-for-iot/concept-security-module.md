---
title: Jumeaux d'appareil et de module de sécurité
description: En savoir plus sur le concept des jumeaux de module de sécurité et la manière dont ils sont utilisés dans Defender pour IoT.
services: defender-for-iot
ms.service: defender-for-iot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/24/2019
ms.author: mlottner
ms.openlocfilehash: b33c456d47426a3721e8582f24ffd603db0429c9
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96340031"
---
# <a name="security-module"></a>Module de sécurité

Cet article explique la manière dont Defender pour IoT utilise les jumeaux et les modules d’appareils.

## <a name="device-twins"></a>Jumeaux d’appareil

Pour les solutions IoT intégrées à Azure, les jumeaux d’appareil jouent un rôle essentiel dans la gestion d’appareils et l’automatisation des processus.

Defender pour IoT offre une intégration complète à votre plateforme de gestion d’appareils IoT existants, ce qui vous permet de gérer l’état de sécurité de vos appareils tout en utilisant les fonctionnalités existantes de contrôle des appareils. L’intégration est obtenue par l’utilisation du mécanisme de jumelage IoT Hub.

En savoir plus sur le concept de [jumeaux d’appareils](../iot-hub/iot-hub-devguide-device-twins.md) dans Azure IoT Hub.

## <a name="security-module-twins"></a>Jumeaux de module de sécurité

Defender pour IoT conserve un jumeau de module de sécurité pour chaque appareil du service.
Le jumeau de module de sécurité contient toutes les informations relatives à la sécurité de l’appareil pour chaque appareil spécifique de votre solution.
Les propriétés de sécurité d’appareil sont conservées dans un jumeau de module de sécurité dédié, pour une communication plus sûre et pour permettre des mises à jour et une maintenance nécessitant moins de ressources.

Consultez [Créer un jumeau de module de sécurité](quickstart-create-security-twin.md) et [Configurer des agents de sécurité](how-to-agent-configuration.md) pour découvrir comment créer, personnaliser et configurer le jumeau. Consultez [Comprendre les jumeaux de module IoT Hub](../iot-hub/iot-hub-devguide-module-twins.md) pour en savoir plus sur le concept des jumeaux de module dans IoT Hub.

## <a name="see-also"></a>Voir aussi

- [Vue d’ensemble de Defender pour IoT](overview.md)
- [Déployer des agents de sécurité](how-to-deploy-agent.md)
- [Méthode d'authentification des agents de sécurité](concept-security-agent-authentication-methods.md)