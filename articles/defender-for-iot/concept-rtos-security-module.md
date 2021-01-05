---
title: Explication conceptuelle des principes de base du module de sécurité pour Azure RTOS
description: Découvrez les principes de base du module de sécurité pour les concepts et le flux de travail Azure RTOS.
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
ms.openlocfilehash: 8f521bd593369509fd520831f90ce9c601227f09
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96340048"
---
# <a name="security-module-for-azure-rtos-preview"></a>Module de sécurité pour Azure RTOS (préversion)

Utilisez cet article mieux comprendre le module de sécurité pour Azure RTOS, y compris les fonctionnalités et les avantages, ainsi que des liens vers des ressources de configuration et de référence pertinentes. 

## <a name="azure-rtos-iot-security-module"></a>Module de sécurité IoT pour Azure RTOS

Le module de sécurité pour Azure RTOS constitue une solution de sécurité complète pour les appareils Azure RTOS dans le cadre de l’offre NetX Duo. Dans l’offre NetX Duo, Azure RTOS est fourni avec le module de sécurité Azure IoT intégré et fournit une couverture contre les menaces courantes sur le système d’exploitation de vos appareils en temps réel, une fois activé. 

Le module de sécurité pour Azure RTOS s’exécute en arrière-plan et offre une expérience utilisateur fluide, tout en envoyant des messages de sécurité à l’aide des connexions uniques de chaque client à leur IoT Hub. Le module de sécurité pour Azure RTOS est activé par défaut.  

## <a name="azure-rtos-netx-duo"></a>Azure RTOS NetX Duo

Azure RTOS NetX Duo est une pile réseau TCP/IP avancée de niveau professionnel, conçue spécifiquement pour les applications IoT et en temps réel profondément intégrées. Azure RTOS NetX Duo est une pile réseau double IPv4 et IPv6 offrant un ensemble complet de protocoles, notamment la sécurité et le cloud. En savoir plus sur les solutions de [NetX Duo d’Azure RTOS](/azure/rtos/netx-duo/).

Le module offre les fonctionnalités suivantes :

- **Détection des activités malveillantes sur les réseaux**
- **Établissement d’une ligne de base du comportement de l’appareil basé sur des alertes personnalisées**
- **Amélioration de l’hygiène de sécurité des appareils**

## <a name="security-module-for-azure-rtos-architecture"></a>Architecture du module de sécurité pour Azure RTOS

Le module de sécurité pour Azure RTOS est initialisé par la plateforme Azure IoT middleware et utilise des clients IoT Hub pour envoyer des données de télémétrie de sécurité au Hub.

:::image type="content" source="media/architecture/security-module-state-diagram.png" alt-text="Diagramme d’état et flux d’informations du module de sécurité Azure IoT":::

Le module de sécurité pour Azure RTOS surveille l’activité et les informations des appareils suivants à l’aide de trois collecteurs :
- Activité réseau **TCP**, **UDP** et **ICM** de l’appareil
- Informations système telles que les versions **ThreadX** et **NetX Duo**
- Événements de pulsation

Chaque collecteur est lié à un groupe de priorité et chaque groupe de priorité a son propre intervalle avec comme valeurs possibles **faible**, **moyenne** et **haute**. Les intervalles affectent l’intervalle de temps pendant lequel les données sont collectées et envoyées.

Chaque intervalle de temps est configurable et les connecteurs IoT peuvent être activés et désactivés afin de [personnaliser votre solution](how-to-azure-rtos-security-module.md). 

## <a name="supported-security-alerts-and-recommendations"></a>Alertes et suggestions de sécurité prises en charge

Le module de sécurité pour Azure RTOS prend en charge des alertes de sécurité et la prise en charge des suggestions spécifiques. Veillez à [passer en revue et à personnaliser les valeurs d’alerte et de suggestion pertinentes](concept-rtos-security-alerts-recommendations.md) pour votre service une fois la configuration initiale effectuée.

## <a name="ready-to-begin"></a>Vous êtes prêt à commencer ?

Le module de sécurité pour Azure RTOS est fourni en téléchargement gratuit pour vos appareils. Le service Defender pour IoT est disponible avec un essai gratuit de 30 jours par abonnement Azure. [Téléchargez le module de sécurité maintenant](https://github.com/azure-rtos/azure-iot-preview/releases) et commençons. 

## <a name="next-steps"></a>Étapes suivantes

- Prise en main du module de sécurité pour Azure RTOS [prérequis et configuration](quickstart-azure-rtos-security-module.md).
- En savoir plus sur les [alertes de sécurité et la prise en charge des suggestions](concept-rtos-security-alerts-recommendations.md) du module de sécurité pour Azure RTOS. 
- [API de référence](azure-rtos-security-module-api.md) pour utiliser le module de sécurité pour Azure RTOS.