---
title: Vue d’ensemble de la simulation d’appareil - Azure | Microsoft Docs
description: Description de l’accélérateur de solution Simulation d’appareil et de ses fonctionnalités.
author: dominicbetts
manager: philmea
ms.service: iot-accelerators
services: iot-accelerators
ms.topic: conceptual
ms.custom: mvc
ms.date: 12/03/2018
ms.author: dobett
ms.openlocfilehash: 7be0278fd8870395b8495168eb9723780eba4b26
ms.sourcegitcommit: 48cb2b7d4022a85175309cf3573e72c4e67288f5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96852368"
---
# <a name="device-simulation-solution-accelerator-overview"></a>Vue d’ensemble de l’accélérateur de solution Simulation d’appareil

Dans une solution IoT cloud, vos appareils se connectent à un point de terminaison cloud pour envoyer des données de télémétrie, comme la température, la localisation et l’état. Votre solution utilise cette télémétrie pour vous permettre d’entreprendre des actions ou d’obtenir des insights.

Quand vous développez une solution IoT, l’expérimentation et le test sont des parties essentielles de ce processus. La simulation est un outil important tout au long de ce processus. Avec la simulation d’appareil, vous pouvez :

* Obtenir rapidement un prototype opérationnel, puis itérer en ajustant le comportement de l’appareil simulé à la volée. Ce processus vous permet de tester votre idée avant d’investir dans du matériel coûteux. Vous pouvez créer des appareils personnalisés via l’interface utilisateur web pour générer un prototype d’appareil en quelques secondes.
* Vérifier que la solution fonctionne comme prévu entre un appareil et la solution en simulant des comportements d’appareils réels. Vous pouvez créer des scripts de comportements d’appareils complexes avec JavaScript pour générer une télémétrie simulée réaliste.
* Tester votre solution à différentes échelles en simulant des conditions de charge normales, lors des pics et au-delà. Les tests de mise à l’échelle vous aident aussi à dimensionner correctement les ressources Azure nécessaires à l’exécution de votre solution.

![Exemple de simulation de drone](media/iot-accelerators-device-simulation-overview/dronesimulation.png)

Avec la simulation d’appareil, vous pouvez définir des modèles d’appareils pour simuler vos appareils réels. Ce modèle inclut les formats de message, les propriétés de jumeau et les méthodes. Vous pouvez également simuler des comportements d’appareils complexes avec JavaScript.

Vous pouvez effectuer des simulations pour un à plusieurs milliers d’appareils se connectant à un hub IoT. Pour faciliter le test, vous pouvez si nécessaire déployer un hub IoT avec la simulation d’appareil pour un environnement autonome.

La simulation d’appareil est gratuite. Cependant, la simulation d’appareil se déploie sur votre abonnement Azure dans le cloud et consomme des ressources Azure. Si la simulation d’appareil ne répond pas à vos besoins, le [code source est également disponible sur GitHub](https://github.com/Azure/device-simulation-dotnet) : vous pouvez donc le copier et le modifier.

## <a name="sample-simulations"></a>Exemples de simulations

Quand vous déployez la simulation d’appareil, vous recevez des exemples de simulations et d’appareils. Vous pouvez utiliser ces exemples pour apprendre à utiliser la simulation d’appareil. Pour commencer, exécutez un [exemple de simulation](https://github.com/Azure/device-simulation-dotnet/blob/master/README.md). Vous pouvez également [créer votre propre simulation en utilisant un des nombreux exemples d’appareils fournis](iot-accelerators-device-simulation-create-simulation.md).

![Configuration de la simulation](media/iot-accelerators-device-simulation-overview/samplesimulation1.png)

## <a name="custom-simulated-devices"></a>Appareils simulés personnalisés

Vous pouvez utiliser la simulation d’appareil pour [créer des modèles d’appareils personnalisés](iot-accelerators-device-simulation-create-custom-device.md) à utiliser dans vos simulations. Par exemple, vous pouvez définir un nouveau modèle d’appareil réfrigérateur qui envoie une télémétrie avec la température et l’humidité. Les appareils simulés personnalisés sont idéaux pour des comportements d’appareils simples avec des valeurs de télémétrie qui s’incrémentent ou se décrémentent, ou sont aléatoires.

![Créer un modèle d’appareil](media/iot-accelerators-device-simulation-overview/adddevicemodel.png)

## <a name="advanced-simulated-devices"></a>Appareils simulés avancés

Quand vous avez besoin de contrôler davantage les valeurs de télémétrie envoyées par un appareil, vous pouvez utiliser un modèle d’appareil avancé. Les modèles d’appareils avancés permettent de prendre en charge JavaScript pour manipuler les valeurs de télémétrie envoyées. Par exemple, vous pouvez simuler la température intérieure d’une voiture en stationnement lors d’une chaude journée ensoleillée : la température extérieure augmentant, la température intérieure augmente de façon exponentielle.

Les modèles d’appareils avancés vous permettent de [créer et de charger vos propres modèles d’appareils](iot-accelerators-device-simulation-advanced-device.md), qui sont constitués d’un fichier de définition d’appareil JSON et des fichiers JavaScript correspondants.

Les modèles d’appareils avancés vous permettent de :

* Spécifier le format des messages envoyés depuis l’appareil ainsi que les types de télémétrie.
* Utiliser un script personnalisé pour générer des valeurs de télémétrie qui gèrent l’état de l’appareil au fil du temps.
* Utiliser un script personnalisé pour spécifier la façon dont l’appareil simulé répond aux méthodes.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez découvert l’accélérateur de solution Simulation d’appareil et ses fonctionnalités. Pour déployer l’accélérateur de solution, visitez le dépôt GitHub :

> [!div class="nextstepaction"]
> [Déployer et exécuter une simulation d’appareil IoT dans Azure](https://github.com/Azure/device-simulation-dotnet/blob/master/README.md)
