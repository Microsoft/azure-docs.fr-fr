---
title: Vue d’ensemble de l’agent de sécurité
description: Prenez un bon départ avec la compréhension, la configuration, le déploiement et l’utilisation des agents de sécurité Azure Defender pour IoT sur vos appareils IoT.
services: defender-for-iot
ms.service: defender-for-iot
documentationcenter: na
author: shhazam-ms
manager: rkarlin
editor: ''
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/27/2019
ms.author: shhazam
ms.openlocfilehash: d3b50d909fb167ee123b548603628bae7d844c5b
ms.sourcegitcommit: 8be279f92d5c07a37adfe766dc40648c673d8aa8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/31/2020
ms.locfileid: "97835906"
---
# <a name="get-started-with-azure-defender-for-iot-device-security-agents"></a>Bien démarrer avec les agents de sécurité d’appareils Azure Defender pour IoT

Les agents de sécurité Defender pour IoT offrent des fonctionnalités de sécurité améliorées, comme la supervision des connexions distantes, les applications actives, les événements de connexion, et des bonnes pratiques en matière de configuration du système d’exploitation. Prenez le contrôle de la protection de l’appareil contre les menaces et de la posture de sécurité avec un seul service.

L’architecture de référence des agents de sécurité Linux et Windows est fournie à la fois en C# et en C.

Les agents de sécurité Defender pour IoT gèrent la collecte d’événements bruts auprès du système d’exploitation, l’agrégation d’événements pour réduire les coûts et la configuration grâce à un jumeau de module. Les messages de sécurité sont envoyés aux services d’analyse de Defender pour l’IoT par le hub IoT.

Utilisez le flux de travail suivant pour déployer et tester vos agents de sécurité Defender pour IoT :

1. [Activer le service Defender pour IoT sur un hub IoT](quickstart-onboard-iot-hub.md)
1. Si aucun appareil n’est inscrit sur votre hub IoT, [Enregistrez un nouvel appareil](../iot-accelerators/quickstart-device-simulation-deploy.md).
1. [Créez un module de sécurité azureiotsecurity](quickstart-create-security-twin.md) pour vos appareils.
1. Pour installer l’agent sur un appareil simulé Azure au lieu d’un appareil réel, [lancez une nouvelle Machine virtuelle Azure](../virtual-machines/linux/quick-create-portal.md) dans une zone disponible.
1. [Déployez un l’agent de sécurité Defender pour IoT](how-to-deploy-linux-cs.md) sur votre appareil IoT ou une nouvelle machine virtuelle.
1. Suivez les instructions de [trigger_events](https://aka.ms/iot-security-github-trigger-events) pour exécuter une simulation d’attaque sans danger.
1. Vérifiez les alertes Defender pour IoT qui surviennent en réponse à l’attaque simulée à l’étape précédente. Commencez la vérification cinq minutes après avoir exécuté le script.
1. Explorez les [alertes](concept-security-alerts.md), les [recommandations](concept-recommendations.md) et la [présentation approfondie de Log Analytics](how-to-security-data-access.md) avec IoT Hub.

## <a name="next-steps"></a>Étapes suivantes

- Configurer une [solution](quickstart-configure-your-solution.md)
- [Créer des modules de sécurité](quickstart-create-security-twin.md)
- Configurer des [alertes personnalisées](quickstart-create-custom-alerts.md)
- [Déployer un agent de sécurité](how-to-deploy-agent.md)