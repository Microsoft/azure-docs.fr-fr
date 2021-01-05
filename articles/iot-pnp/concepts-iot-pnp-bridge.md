---
title: Pont IoT Plug-and-Play | Microsoft Docs
description: Découvrez le pont IoT Plug-and-Play et comment l’utiliser pour connecter des appareils existants attachés à une passerelle Windows ou Linux en tant qu’appareils IoT Plug-and-Play.
author: usivagna
ms.author: ugans
ms.date: 09/22/2020
ms.topic: conceptual
ms.service: iot-pnp
services: iot-pnp
ms.custom: mvc
ms.openlocfilehash: 0435fe3946118d59d786dd3e6cec350a5ab4eee4
ms.sourcegitcommit: 2e72661f4853cd42bb4f0b2ded4271b22dc10a52
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92046449"
---
# <a name="iot-plug-and-play-bridge"></a>Pont IoT Plug-and-Play

Le pont IoT Plug-and-Play est une application open source permettant de connecter des appareils existants attachés à une passerelle Windows ou Linux en tant qu’appareils IoT Plug-and-Play. Après l’installation et la configuration de l’application sur votre machine Windows ou Linux, vous pouvez l’utiliser pour connecter des appareils attachés à un hub IoT. Vous pouvez utiliser le pont pour mapper les interfaces IoT Plug-and-Play aux télémétries envoyées par les appareils attachés, utiliser les propriétés des appareils et appeler des commandes.

:::image type="content" source="media/concepts-iot-pnp-bridge/iot-pnp-bridge-high-level.png" alt-text="Sur le côté gauche, deux capteurs existants sont attachés (les deux filaires et sans fil) à un PC Windows ou Linux contenant un pont IoT Plug-and-Play. Le pont Plug-and-Play IoT se connecte ensuite à un hub IoT sur le côté droit":::

Le pont IoT Plug-and-Play peut être déployé comme un exécutable autonome sur tout appareil IoT, PC, serveur ou toute passerelle exécutant Windows 10 ou Linux. Il peut également être compilé dans le code de votre application. Un fichier JSON de configuration simple indique au pont IoT Plug-and-Play que les appareils/périphériques attachés doivent être exposés à Azure.

## <a name="supported-protocols-and-sensors"></a>Protocoles et capteurs pris en charge

Le pont IoT Plug-and-Play prend en charge les types suivants de périphériques par défaut, avec des liens vers la documentation de l’adaptateur :

|Périphérique|Windows|Linux|
|---------|---------|---------|
|[Bluetooth LE](https://aka.ms/iot-pnp-bridge-bluetooth)       |Oui|Non|
|[Appareils photo](https://aka.ms/iot-pnp-bridge-camera)               |Oui|Non|
|[Modbus](https://aka.ms/iot-pnp-bridge-modbus)                |Oui|Oui|
|[MQTT](https://aka.ms/iot-pnp-bridge-mqtt)                    |Oui|Oui|
|[Série](https://aka.ms/iot-pnp-bridge-serial)                |Oui|Oui|
|[Périphériques USB Windows](https://aka.ms/iot-pnp-bridge-usb)  |Oui|Non applicable|

>[!Important]
>Les développeurs peuvent étendre le pont IoT Plug-and-Play pour prendre en charge des protocoles d’appareils supplémentaires à l’aide des instructions fournies dans la **[documentation pour développeurs de pont IoT Plug-and-Play ici](https://aka.ms/iot-pnp-bridge-dev-doc)** .

## <a name="prerequisites"></a>Prérequis

### <a name="os-platform"></a>Plateforme de système d’exploitation

Les plateformes et versions de système d’exploitation suivantes sont prises en charge :

|Plateforme  |Versions prises en charge  |
|---------|---------|
|Windows 10 |     Toutes les références SKU Windows sont prises en charge. Par exemple : IoT Enterprise, Server, Desktop, IoT Core. *Pour la fonctionnalité d’analyse du fonctionnement de la caméra, 20H1 ou une version ultérieure est recommandé. Toutes les autres fonctionnalités sont disponibles sur toutes les versions de Windows 10.*  |
|Linux     |Testé et pris en charge sur Ubuntu 18.04 ; le fonctionnement des autres distributions n’a pas été testé.         |
||

### <a name="hardware"></a>Matériel

- Toute plateforme matérielle prenant en charge les références et versions de système d’exploitation ci-dessus.
- Les capteurs et périphériques série, USB, Bluetooth et de caméra sont pris en charge en mode natif. Le pont IoT Plug-and-Play peut être étendu pour prendre en charge n’importe quel périphérique ou capteur personnalisé ([Voir la section Périphériques ci-dessus](#iot-plug-and-play-bridge)).

### <a name="development-environment"></a>Environnement de développement

Pour créer, étendre et développer le pont IoT Plug-and-Play, vous aurez besoin de ce qui suit :  

- Un environnement de développement qui prend en charge la compilation C++, par exemple : [Visual Studio (Community, Professional ou Enterprise)](https://visualstudio.microsoft.com/downloads/) : veillez à inclure la charge de travail Développement Desktop en C++ quand vous installez Visual Studio.
- [CMake](https://cmake.org/download/) : lorsque vous installez CMake, sélectionnez l’option `Add CMake to the system PATH`.
- Si vous créez sur Windows, vous devrez également télécharger le kit de développement logiciel (SDK) Windows 17763 : [https://developer.microsoft.com/windows/downloads/windows-10-sdk](https://developer.microsoft.com/windows/downloads/windows-10-sdk)
- [Kit de développement logiciel (SDK) d’appareil Azure IoT Hub C](https://github.com/Azure/azure-iot-sdk-c). Les scripts de génération inclus dans ce référentiel clonent automatiquement le kit de développement logiciel (SDK) Azure IoT C requis pour vous.

### <a name="azure-iot-products-and-tools"></a>Outils et produits Azure IoT

- **Azure IoT Hub** : vous aurez besoin d’un [Azure IoT Hub](../iot-hub/index.yml) dans votre abonnement Azure pour connecter votre appareil. Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer. Si vous n’avez pas de hub IoT, [suivez ces instructions pour en créer un](../iot-hub/iot-hub-create-using-cli.md).

> [!Note]
> IoT Plug-and-Play est actuellement disponible sur les hubs IoT créés dans les régions USA Centre, Europe Nord et Japon Est. La prise en charge d’IoT Plug-and-Play n’est pas incluse dans les hubs IoT de niveau De base. Pour interagir avec votre appareil IoT Plug-and-Play, vous pouvez utiliser l’outil Explorateur Azure IoT. [Téléchargez et installez la dernière version de l’Explorateur Azure IoT](./howto-use-iot-explorer.md) pour votre système d’exploitation.

## <a name="iot-plug-and-play-bridge-architecture"></a>Architecture de pont IoT Plug-and-Play

:::image type="content" source="media/concepts-iot-pnp-bridge/iot-pnp-bridge-components.png" alt-text="Sur le côté gauche, deux capteurs existants sont attachés (les deux filaires et sans fil) à un PC Windows ou Linux contenant un pont IoT Plug-and-Play. Le pont Plug-and-Play IoT se connecte ensuite à un hub IoT sur le côté droit":::

## <a name="download-iot-plug-and-play-bridge"></a>Télécharger le pont IoT Plug-and-Play

Vous pouvez télécharger une version pré-générée du pont avec des adaptateurs pris en charge parmi les [versions du pont IoT Plug-and-Play](https://aka.ms/iot-pnp-bridge-releases) et étendre la liste des ressources pour la version la plus récente. Téléchargez la version la plus récente de l’application pour votre système d’exploitation.

Vous pouvez également télécharger et afficher le code source du [pont IoT Plug-and-Play sur GitHub](https://aka.ms/bridge).

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous disposez d’une vue d’ensemble de l’architecture d’un pont IoT Plug-and-Play, les étapes suivantes permettent d’en savoir plus :

- [Comment utiliser le pont IoT Plug-and-Play](./howto-use-iot-pnp-bridge.md)
- [Consulter les informations de référence pour les développeurs GitHub pour le pont IoT Plug-and-Play](https://aka.ms/iot-pnp-bridge-dev-doc)
- [Pont IoT Plug-and-Play sur GitHub](https://aka.ms/iotplugandplaybridge)