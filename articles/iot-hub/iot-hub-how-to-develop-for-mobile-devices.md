---
title: Développer pour les appareils mobiles à l’aide des Kits de développement logiciels (SDK) Azure IoT | Microsoft Docs
description: 'Guide du développeur : découvrez comment développer pour les appareils mobiles à l’aide des Kits de développement logiciels (SDK) Azure IoT Hub.'
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 04/16/2018
ms.author: robinsh
ms.openlocfilehash: 63fade17d10189cbe8e775b2a859902f1c2bb004
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92151644"
---
# <a name="develop-for-mobile-devices-using-azure-iot-sdks"></a>Développer pour les appareils mobiles à l’aide des Kits de développement logiciels (SDK) Azure IoT

Les « objets » de l’Internet des objets peuvent désigner un large éventail d’appareils dotés de capacités diverses, tels que les capteurs, les microcontrôleurs, les smart devices, les passerelles industrielles et même les appareils mobiles.  Un appareil mobile peut être un appareil IoT qui envoie des données de télémétrie appareil-à-cloud et est géré par le cloud.  Il peut également faire office d’appareil exécutant une application de service principal qui gère les autres appareils IoT.  Dans les deux cas, les [kits SDK Azure IoT Hub](./iot-hub-devguide-sdks.md) peuvent être utilisés pour développer des applications fonctionnant sur appareil mobile.  

## <a name="develop-for-native-ios-platform"></a>Développer pour la plateforme iOS native

Les Kits de développement logiciels (SDK) Azure IoT Hub assurent la prise en charge de la plateforme iOS native via le Kit de développement logiciel (SDK) C Azure IoT Hub.  Celui-ci peut être considéré comme un Kit de développement logiciel (SDK) iOS incorporable dans un projet XCode Swift ou Objective-C.  Le Kit de développement logiciel (SDK) C peut être utilisé de deux façons sur iOS :

* Utiliser les bibliothèques CocoaPod dans le projet XCode directement.  
* Télécharger le code source pour le kit SDK C et générer pour la plateforme iOS en suivant les [instructions de génération](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/devbox_setup.md) pour MacOS.  

Le Kit de développement logiciel (SDK) C Azure IoT Hub est écrit en C99 pour une portabilité maximale sur les différentes plateformes.  Le processus de portage implique l’écriture d’une fine couche d’adoption pour les composants propres à la plateforme en question, disponible ici pour [iOS](https://github.com/Azure/azure-c-shared-utility/tree/master/pal/ios-osx).  Les fonctionnalités du Kit de développement logiciel (SDK) C peuvent être exploitées sur la plateforme iOS, notamment les primitives Azure IoT Hub prises en charge et les fonctionnalités spécifiques du Kit de développement logiciel (SDK), telles que la stratégie de nouvelles tentatives pour la fiabilité du réseau.  L’interface du Kit de développement logiciel (SDK) iOS est également similaire à l’interface du Kit de développement logiciel (SDK) C Azure IoT Hub.  

Les documents qui suivent vous guident dans le développement d’une application d’appareil ou d’une application de service sur un appareil iOS :

* [Démarrage rapide : Send telemetry from a device to an IoT hub (Envoyer des données de télémétrie d’un appareil à un IoT Hub)](quickstart-send-telemetry-ios.md)  
* [Envoyer des messages du cloud à votre appareil avec un hub IoT](iot-hub-ios-swift-c2d.md) 

### <a name="develop-with-azure-iot-hub-cocoapod-libraries"></a>Développer avec les bibliothèques CocoaPod Azure IoT Hub

Les Kits de développement logiciel (SDK) Azure IoT Hub s’accompagnent d’un ensemble de bibliothèques CocoaPod Objective-C pour le développement iOS.  Pour obtenir la dernière liste en date des bibliothèques CocoaPod, consultez [CocoaPods for Microsoft Azure IoT](https://github.com/Azure/azure-iot-sdk-c/blob/master/iothub_client/samples/ios/CocoaPods.md).  Une fois les bibliothèques appropriées incorporées dans votre projet XCode, deux options s’offrent à vous pour écrire le code IoT Hub associé :

* Fonction Objective-C : Si votre projet est écrit en Objective-C, vous pouvez appeler des API directement à partir du Kit de développement logiciel (SDK) C Azure IoT Hub.  Si votre projet est écrit en Swift, vous pouvez appeler `@objc func` avant de créer votre fonction et procéder à l’écriture de toute la logique liée à Azure IoT Hub à l’aide de code C ou Objective-C.  Vous trouverez un ensemble d’exemples illustrant les deux cas de figure dans l’[exemple de dépôt](https://github.com/Azure-Samples/azure-iot-samples-ios).  

* Incorporation d’exemples en C : Si vous avez écrit une application d’appareil en C, vous pouvez la référencer directement dans votre projet XCode :
    * Ajoutez le fichier sample.c à votre projet XCode dans XCode.  
    * Ajoutez le fichier d’en-tête à votre dépendance.  Un fichier d’en-tête est inclus dans l’[exemple de dépôt](https://github.com/Azure-Samples/azure-iot-samples-ios) à titre d’illustration. Pour plus d’informations, consultez la page de documentation d’Apple relative à [Objective-C](https://developer.apple.com/documentation/objectivec).

## <a name="develop-for-android-platform"></a>Développer pour la plateforme Android
Le SDK Java Azure IoT Hub prend en charge la plateforme Android.  Pour la version d’API spécifique testée et la dernière mise à jour correspondante, visitez notre [page de support de la plateforme](iot-hub-device-sdk-platform-support.md).

Les documents qui suivent vous guident dans le développement d’une application d’appareil ou d’une application de service sur un appareil Android à l’aide de Gradle et Android Studio :

* [Démarrage rapide : Send telemetry from a device to an IoT hub (Envoyer des données de télémétrie d’un appareil à un IoT Hub)](quickstart-send-telemetry-android.md)  
* [Démarrage rapide : Contrôler un appareil](quickstart-control-device-android.md) 

## <a name="next-steps"></a>Étapes suivantes

* [Informations de référence sur l’API REST d’IoT Hub](/rest/api/iothub/)
* [Code source du SDK C Azure IoT](https://github.com/Azure/azure-iot-sdk-c)