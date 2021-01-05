---
title: Module de sécurité Defender pour IoT pour IoT Edge
description: Découvrez l’architecture et les fonctionnalités du module de sécurité Azure Defender pour IoT pour IoT Edge.
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
ms.date: 09/09/2020
ms.author: mlottner
ms.openlocfilehash: 4f6d9f670a1b85e55ccc8f6cb18645b92927221a
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96351637"
---
# <a name="azure-defender-for-iot-edge-security-module"></a>Module de sécurité Azure Defender pour IoT Edge

[Azure IoT Edge](../iot-edge/index.yml) offre de puissantes fonctionnalités de gestion et d’exécution de flux de travail à la périphérie.
Le rôle clé d’IoT Edge dans les environnements IoT le rend particulièrement séduisant pour les personnes malveillantes.

Le module de sécurité Defender pour IoT constitue une solution de sécurité complète pour vos appareils IoT Edge.
Il collecte, agrège et analyse des données de sécurité brutes tirées du système d’exploitation et du système de conteneur pour produire des alertes et des suggestions de sécurité actionnables.

Tout comme les agents de sécurité Defender pour IoT destinés aux appareils IoT, le module Defender pour IoT Edge est hautement personnalisable grâce à son jumeau de module.
Pour plus d’informations, voir [Configurer un agent](how-to-agent-configuration.md).

Le module de sécurité Defender pour IoT pour IoT Edge offre les fonctionnalités suivantes :

- Collecte d’événements de sécurité bruts auprès du système d’exploitation sous-jacent (Linux) et des systèmes de conteneurs IoT Edge.

  Pour en savoir plus sur les collecteurs de données de sécurité disponibles, consultez [Configuration de l'agent Defender pour IoT](how-to-agent-configuration.md).

- Analyse des manifestes de déploiement IoT Edge.

- Agrégation des événements de sécurité bruts dans des messages envoyés par un [hub IoT Edge](../iot-edge/iot-edge-runtime.md#iot-edge-hub).

- Configuration à distance grâce au jumeau de module de sécurité.

  Pour plus d'informations, consultez [Configurer un agent Defender pour IoT](how-to-agent-configuration.md).

Le module de sécurité Defender pour IoT pour IoT Edge s’exécute en mode privilégié sous IoT Edge.
Ce mode est nécessaire pour l’autoriser à effectuer un monitoring du système d’exploitation et d’autres modules IoT Edge.

## <a name="module-supported-platforms"></a>Plateformes prises en charge du module

Le module de sécurité Defender pour IoT pour IoT Edge n’est actuellement disponible que pour Linux.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez découvert l’architecture et les fonctionnalités du module de sécurité Defender pour IoT pour IoT Edge.

Pour poursuivre votre prise en main du déploiement de Defender pour IoT, lisez les articles suivants :

- Déployez le [module de sécurité pour IoT Edge](how-to-deploy-edge.md)
- Découvrez comment [configurer votre module de sécurité](how-to-agent-configuration.md)
- Consultez les [Prérequis du service](service-prerequisites.md) Defender pour IoT
- Apprenez à [activer le service Defender pour IoT dans votre hub IoT](quickstart-onboard-iot-hub.md)
- Renseignez-vous sur le service dans la [FAQ Defender pour IoT](resources-frequently-asked-questions.md)