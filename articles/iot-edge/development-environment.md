---
title: Environnement de développement d’Azure IoT Edge | Microsoft Docs
description: Découvrez les systèmes et les outils de développement tiers pris en charge qui vous aideront à créer des modules IoT Edge
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 01/04/2019
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: 4064acd6d2857ebc00a38ea7f6ff5d5e30bbd699
ms.sourcegitcommit: 4f4a2b16ff3a76e5d39e3fcf295bca19cff43540
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93041378"
---
# <a name="prepare-your-development-and-test-environment-for-iot-edge"></a>Préparer votre environnement de développement et de test pour IoT Edge

Azure IoT Edge déplace votre logique métier existante vers des appareils opérant à la périphérie. Pour préparer vos applications et charges de travail à s’exécuter en tant que [modules IoT Edge](iot-edge-modules.md), vous devez les créer comme des conteneurs. Cet article fournit des instructions sur la façon de configurer votre environnement de développement de façon à pouvoir créer une solution IoT Edge. Une fois votre environnement de développement configuré, vous pouvez découvrir comment [Développer vos propres modules IoT Edge](module-development.md).

Dans toute solution IoT Edge, il existe au moins deux types de machines à prendre en compte. L’un est le type d’appareil(s) IoT Edge qui exécute(nt) le module IoT Edge. L’autre est l’ordinateur de développement utilisé pour créer, tester et déployer les modules. Cet article se concentre principalement sur l’ordinateur de développement. À des fins de test, les deux machines peuvent être la même. Vous pouvez exécuter IoT Edge sur votre ordinateur de développement, ainsi que déployer les modules sur celui-ci.

## <a name="operating-system"></a>Système d’exploitation

Azure IoT Edge s’exécute sur un ensemble spécifique de [systèmes d’exploitation pris en charge](support.md). Pour développer pour IoT Edge, vous pouvez utiliser la plupart des systèmes d’exploitation pouvant exécuter un moteur de conteneur. La présence du moteur de conteneur est obligatoire sur l’ordinateur de développement pour générer vos modules en tant que conteneurs et les envoyer à un registre de conteneurs.

Si votre ordinateur de développement ne peut pas exécuter Azure IoT Edge, poursuivez la lecture de cet article pour en savoir plus sur les [outils de test](#testing-tools) qui vous aident à tester et à déboguer localement.

Le système d’exploitation de votre ordinateur de développement ne doit pas nécessairement être le même que celui de votre appareil IoT Edge. En revanche, le système d’exploitation du conteneur doit être identique sur l’ordinateur de développement et l’appareil IoT Edge. Par exemple, vous pouvez développer des modules sur un ordinateur Windows et les déployer sur un appareil Linux. L’ordinateur Windows doit exécuter des conteneurs Linux pour générer les modules pour l’appareil Linux.

## <a name="container-engine"></a>Moteur de conteneur

Le concept central d’IoT Edge est que vous pouvez déployer à distance votre logique métier et cloud sur des appareils en l’empaquetant dans des conteneurs. Pour créer des conteneurs, vous avez besoin d’un moteur de conteneur sur votre ordinateur de développement.

Le seul moteur de conteneur pris en charge pour des appareils IoT Edge en production est Moby. Toutefois, tout moteur de conteneur compatible avec l’Initiative de conteneur ouvert, tel Docker, est capable de créer des images de modules IoT Edge.

## <a name="development-tools"></a>Outils de développement

Visual Studio et Visual Studio Code comprennent des extensions de modules complémentaires pour vous aider à développer des solutions IoT Edge. Ces extensions fournissent des modèles spécifiques du langage pour faciliter la création et le déploiement de nouveaux scénarios IoT Edge. Les extensions Azure IoT Edge pour Visual Studio et Visual Studio Code vous aident à coder, à générer, à déployer et à déboguer vos solutions IoT Edge. Vous pouvez créer une solution IoT Edge entière contenant plusieurs modules. Les extensions mettent à jour automatiquement un modèle de manifeste de déploiement avec chaque nouvel ajout de module. Les extensions vous permettent également de gérer vos appareils IoT à partir de Visual Studio ou de Visual Studio Code. Déployez des modules sur un appareil, surveillez l’état et affichez les messages à mesure qu’ils parviennent à IoT Hub. Les deux extensions utilisent l’[outil de développement IoT EdgeHub](#iot-edgehub-dev-tool) pour permettre l’exécution et le débogage locaux de modules également sur votre ordinateur de développement.

Si vous préférez développer avec d’autres éditeurs ou à partir de l’interface CLI, l’outil de développement Azure IoT Edge fournit des commandes qui vous permettent de développer et tester à partir de la ligne de commande. Vous pouvez créer des scénarios IoT Edge, générer des images de module, exécuter des modules dans un simulateur et surveiller les messages envoyés à IoT Hub.

### <a name="visual-studio-code-extension"></a>Extension de Visual Studio Code

L’extension Azure IoT Edge pour Visual Studio Code fournit des modèles de modules IoT Edge basés sur des langages de programmation, dont C, C#, Java, Node.js et Python, ainsi que des fonctions Azure en C#.

Pour plus d’informations et pour accéder au téléchargement, consultez [Outils Azure IoT pour Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools).

En plus des extensions IoT Edge, il peut s’avérer utile d’installer des extensions supplémentaires pour le développement. Par exemple, vous pouvez utiliser la [Prise en charge de Docker pour Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=PeterJausovec.vscode-docker) afin de gérer vos images, conteneurs et registres. De plus, tous les principaux langages pris en charge comprennent des extensions pour Visual Studio Code qui peuvent vous aider à développer des modules.

#### <a name="prerequisites"></a>Prérequis

Les modèles de modules pour certains langages et services imposent des conditions préalables qui sont nécessaires pour générer les dossiers de projet sur votre ordinateur de développement avec Visual Studio Code.

| Modèle de module | Configuration requise |
| --------------- | ------------ |
| Azure Functions | [SDK .NET Core 2.1](https://www.microsoft.com/net/download) |
| C | [Git](https://git-scm.com/) |
| C# | [SDK .NET Core 2.1](https://www.microsoft.com/net/download) |
| Java | <ul><li>[Java SE Development Kit 10](/azure/developer/java/fundamentals/java-jdk-long-term-support) <li> [Définir la variable d’environnement JAVA_HOME](https://docs.oracle.com/cd/E19182-01/820-7851/inst_cli_jdk_javahome_t/) <li> [Maven](https://maven.apache.org/)</ul> |
| Node.js | <ul><li>[Node.JS](https://nodejs.org/) <li> [Yeoman](https://www.npmjs.com/package/yo) <li> [Générateur de module Node.js Azure IoT Edge](https://www.npmjs.com/package/generator-azure-iot-edge-module)</ul> |
| Python |<ul><li> [Python](https://www.python.org/downloads/) <li> [Pip](https://pip.pypa.io/en/stable/installing/#installation) <li> [Git](https://git-scm.com/) </ul> |

### <a name="visual-studio-20172019-extension"></a>Extension Visual Studio 2017/2019

Les outils Azure IoT Edge pour Visual Studio fournissent un modèle de module IoT Edge basé sur C# et C.

Pour plus d’informations et pour télécharger, voir [Outils Azure IoT Edge pour Visual Studio 2017](https://marketplace.visualstudio.com/items?itemName=vsc-iot.vsiotedgetools) ou [Outils Azure IoT Edge pour Visual Studio 2019](https://marketplace.visualstudio.com/items?itemName=vsc-iot.vs16iotedgetools).

### <a name="iot-edge-dev-tool"></a>Outil de développement IoT Edge

L’outil de développement Azure IoT Edge simplifie le développement IoT Edge avec des fonctionnalités de ligne de commande. Cet outil fournit des commandes CLI pour développer, déboguer et tester des modules. L’outil de développement IoT Edge fonctionne avec votre système de développement, que vous ayez installé manuellement les dépendances sur votre ordinateur ou que vous utilisiez le conteneur de développement IoT Edge.

Pour plus d’informations et pour commencer, voir le [wiki de l’outil de développement IoT Edge](https://github.com/Azure/iotedgedev/wiki).

## <a name="testing-tools"></a>Outils de test

Plusieurs outils de test sont disponibles pour vous aider à simuler des appareils IoT Edge ou à déboguer des modules plus efficacement. Le tableau suivant présente une comparaison générale entre les outils, tandis que les sections individuelles décrivent chaque outil plus précisément.

Seul le runtime IoT Edge est pris en charge pour les déploiements de production, mais les outils suivants vous permettent de simuler ou de créer aisément des appareils IoT Edge à des fins de test et développement. Ces outils ne s’excluent pas mutuellement et peuvent opérer ensemble pour offrir une expérience de développement complète.

| Outil | Également appelé | Plateformes prises en charge | Idéal pour |
| ---- | ------------- | ------------------- | --------- |
| Outil de développement IoT EdgeHub  | iotedgehubdev | Windows, Linux, macOS | Simulation d’appareil pour déboguer des modules. |
| Conteneur de développement IoT Edge | iotedgedev | Windows, Linux, macOS | Développement sans installation de dépendances. |
| Runtime IoT Edge dans un conteneur | iotedgec | Windows, Linux, macOS, ARM | Tests sur un appareil ne pouvant pas prendre en charge le runtime. |
| Conteneur d’appareil IoT Edge | toolboc/azure-iot-edge-device-container | Windows, Linux, macOS, ARM | Tests d’un scénario avec de nombreux appareils IoT Edge à grande échelle. |

### <a name="iot-edgehub-dev-tool"></a>Outil de développement IoT EdgeHub

L’outil de développement hub Azure IoT Edge fournit une expérience de développement et débogage locale. L’outil aide à démarrer des modules IoT Edge sans le runtime IoT Edge de façon à pouvoir créer, développer, tester, exécuter et déboguer des modules et solutions IoT Edge localement. Vous n’êtes pas obligé d’envoyer des images à un registre de conteneurs et de les déployer sur un appareil à des fins de test.

L’outil de développement IoT EdgeHub a été conçu pour fonctionner conjointement avec les extensions Visual Studio et Visual Studio Code, ainsi qu’avec l’outil de développement IoT Edge. Il prend en charge le développement de boucle interne ainsi qu’un test de boucle externe, et s’intègre donc également avec les outils de DevOps.

Pour plus d’informations et pour installer, voir [Outil de développement Azure IoT EdgeHub](https://pypi.org/project/iotedgehubdev/).

### <a name="iot-edge-dev-container"></a>Conteneur de développement IoT Edge

Le conteneur de développement Azure IoT Edge est un conteneur Docker comprenant toutes les dépendances dont vous avez besoin pour le développement IoT Edge. Ce conteneur facilite la prise en main avec tout langage dans lequel vous souhaitez développer, dont C#, Python, Node.js et Java. Il vous suffit d’installer un moteur de conteneur tel que Docker ou Moby pour extraire le conteneur sur votre ordinateur de développement.

Pour plus d’informations, voir [Conteneur de développement Azure IoT Edge](https://github.com/Azure/iotedgedev/wiki/quickstart-with-iot-edge-dev-container).

### <a name="iot-edge-runtime-in-a-container"></a>Runtime IoT Edge dans un conteneur

Le runtime IoT Edge dans un conteneur fournit un runtime complet qui accepte votre chaîne de connexion d’appareil en tant que variable d’environnement. Ce conteneur vous permet de tester des modules et scénarios IoT Edge sur un système ne pouvant pas prendre en charge le runtime en mode natif, comme macOS. Tous les modules que vous déployez démarreront en dehors du conteneur de runtime. Si vous souhaitez que le runtime et les modules déployés coexistent dans le même conteneur, envisagez plutôt le conteneur d’appareil IoT Edge.

Pour plus d’informations, voir [Exécution d’Azure IoT Edge dans un conteneur](https://github.com/Azure/iotedgedev/tree/master/docker/runtime).

### <a name="iot-edge-device-container"></a>Conteneur d’appareil IoT Edge

Le conteneur d’appareil IoT Edge est un appareil IoT Edge complet, prêt pour lancement sur tout ordinateur disposant d’un moteur de conteneur. Le conteneur d’appareil inclut le runtime IoT Edge et un moteur de conteneur. Chaque instance du conteneur est un appareil IoT Edge s’auto-approvisionnant entièrement fonctionnel. Le conteneur d’appareil prend en charge le débogage à distance de modules tant qu’il existe un itinéraire réseau vers le module. Le conteneur d’appareil convient bien pour créer rapidement de grands nombres d’appareils IoT Edge afin de tester des scénarios ou Azure Pipelines à grande échelle. Il prend également en charge le déploiement vers Kubernetes via Helm.

Pour plus d’informations, voir [Conteneur d’appareil Azure IoT Edge](https://github.com/toolboc/azure-iot-edge-device-container).

## <a name="devops-tools"></a>Outils de DevOps

Lorsque vous êtes prêt à développer des solutions à grande échelle pour des scénarios de production d’envergure, tirez parti des principes de DevOps modernes, dont l’automation, la surveillance et les processus d’ingénierie logicielle simplifiée. IoT Edge comprend des extensions pour prendre en charge des outils de DevOps, dont Azure DevOps, Azure DevOps Projects et Jenkins. Si vous souhaitez personnaliser un pipeline existant ou vous servir d’un autre outil de DevOps tel que CircleCI ou TravisCI, vous pouvez le faire avec les fonctionnalités CLI incluses dans l’outil de développement IoT Edge.

Pour plus d’informations, des conseils et des exemples, voir les pages suivantes :

* [Intégration continue et déploiement continu dans Azure IoT Edge](how-to-continuous-integration-continuous-deployment.md)
* [Créer un pipeline CI/CD pour IoT Edge avec Azure DevOps Starter](how-to-devops-starter.md)
* [Plug-in Jenkins pour Azure IoT Edge](https://plugins.jenkins.io/azure-iot-edge)
* [Dépôt GitHub de DevOps IoT Edge](https://github.com/toolboc/IoTEdge-DevOps)