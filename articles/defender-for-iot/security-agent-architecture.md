---
title: Architecture de l’agent de sécurité
description: Familiarisez-vous avec l'architecture des agents de sécurité utilisés dans le service Azure Defender pour IoT.
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
ms.date: 07/26/2019
ms.author: mlottner
ms.openlocfilehash: 64d90b4d9dc3efbe877230bbc20780b1c4f2d213
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90930602"
---
# <a name="security-agent-reference-architecture"></a>Architecture de référence de l’agent de sécurité

Azure Defender pour IoT fournit une architecture de référence pour les agents de sécurité qui consignent, traitent, agrègent et envoient des données de sécurité via IoT Hub.

Les agents de sécurité sont conçus pour fonctionner dans un environnement IoT restreint et sont hautement personnalisables quant aux valeurs qu’ils fournissent par rapport aux ressources qu’ils utilisent.

Les agents de sécurité prennent en charge les fonctionnalités suivantes :

- Collecte d’événements de sécurité bruts à partir du système d’exploitation sous-jacent (Linux, Windows). Pour en savoir plus sur les collecteurs de données de sécurité disponibles, consultez [Configuration de l'agent Defender pour IoT](how-to-agent-configuration.md).

- Regroupez des événements de sécurité bruts dans des messages envoyés via IoT Hub.

- Authentification avec une identité d’appareil existante, ou une identité de module dédiée. Consultez [Méthodes d’authentification de l’agent de sécurité](concept-security-agent-authentication-methods.md) pour en savoir plus.

- Configuration à distance à l’aide du jumeau de module **azureiotsecurity**. Pour plus d'informations, consultez [Configurer un agent Defender pour IoT](how-to-agent-configuration.md).

Les agents de sécurité Defender pour IoT sont développés en tant que projets open source et sont disponibles sur GitHub :

- [Agent Defender pour IoT basé sur C](https://github.com/Azure/Azure-IoT-Security-Agent-C)
- [Agent Defender pour IoT basé sur C#](https://github.com/Azure/Azure-IoT-Security-Agent-CS)

## <a name="agent-supported-platforms"></a>Plateformes prises en charge par l’agent

Defender pour IoT propose différents agents d'installation pour les versions 32 et 64 bits de Windows, ainsi que pour les versions 32 et 64 bits de Linux. Vérifiez que vous avez le programme d’installation d’agent approprié pour chacun de vos appareils conformément au tableau suivant :

| Architecture | Linux | Windows |    Détails|
|----------|----------------------------------------------|-------------|-------------------------------------------|
| 32 bits  | C  | C#  ||
| 64 bits  | C# ou C           | C#      | Nous vous recommandons d’utiliser l’agent C pour les appareils présentant un nombre de ressources d’appareils minimales ou restreintes.|
|

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous vous êtes familiarisé avec l'architecture de l'agent de sécurité Defender pour IoT et avec les programmes d'installation disponibles.

Pour poursuivre votre prise en main du déploiement de Defender pour IoT, lisez les articles suivants :

- Compréhension des [méthodes d’authentification de l’agent de sécurité](concept-security-agent-authentication-methods.md)
- Sélectionner et déployer un [agent de sécurité](how-to-deploy-agent.md)
- Consultez les [Prérequis du service](service-prerequisites.md) Defender pour IoT
- Apprenez à [activer le service Defender pour IoT dans votre hub IoT](quickstart-onboard-iot-hub.md)
- Renseignez-vous sur le service dans la [FAQ Defender pour IoT](resources-frequently-asked-questions.md)
