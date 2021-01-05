---
title: Développer des modules pour Azure IoT Edge | Microsoft Docs
description: Développer des modules personnalisés pour Azure IoT Edge qui peuvent communiquer avec le runtime et IoT Hub
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 11/10/2020
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: 8907af07fff7b315eec263d38b686c17218ed9d2
ms.sourcegitcommit: 6109f1d9f0acd8e5d1c1775bc9aa7c61ca076c45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94445470"
---
# <a name="develop-your-own-iot-edge-modules"></a>Développer vos propres modules IoT Edge

Les modules Azure IoT Edge peuvent se connecter à d’autres services Azure et contribuer à votre pipeline de données cloud plus volumineuses. Cet article explique comment vous pouvez développer des modules pour communiquer avec le runtime IoT Edge et IoT Hub, et par conséquent, avec le reste du cloud Azure.

## <a name="iot-edge-runtime-environment"></a>Environnement d’exécution IoT Edge

Le runtime IoT Edge fournit l’infrastructure nécessaire pour intégrer les fonctionnalités de plusieurs modules IoT Edge et pour les déployer sur des appareils IoT Edge. Tout programme peut être empaqueté en tant que module IoT Edge. Pour tirer pleinement parti des fonctionnalités de communication et de gestion d’IoT Edge, un programme qui s’exécute dans un module peut utiliser Azure IoT Device SDK pour se connecter au hub IoT Edge local.
::: moniker range=">=iotedge-2020-11"
Les modules peuvent également utiliser n’importe quel client MQTT pour se connecter au répartiteur MQTT du hub IoT Edge local.
::: moniker-end

### <a name="packaging-your-program-as-an-iot-edge-module"></a>Empaquetage de votre programme en tant que module IoT Edge

Pour déployer votre programme sur un appareil IoT Edge, il doit d’abord être conteneurisé et exécuté à l’aide d’un moteur compatible avec Docker. IoT Edge utilise [Moby](https://github.com/moby/moby), le projet open source à l’origine de Docker, comme moteur compatible avec Docker. Les mêmes paramètres que ceux que vous utilisez avec Docker peuvent être transmis à vos modules IoT Edge. Pour en savoir plus, consultez [Guide pratique pour configurer les options de création de conteneur pour les modules IoT Edge](how-to-use-create-options.md).

## <a name="using-the-iot-edge-hub"></a>Utilisation du hub IoT Edge

Le hub IoT Edge fournit deux fonctionnalités principales : proxy d’IoT Hub et communications locales.

### <a name="connecting-to-iot-edge-hub-from-a-module"></a>Connexion au hub IoT Edge à partir d’un module

La connexion au hub IoT Edge local à partir d’un module implique les mêmes étapes de connexion que pour tout autre client. Pour plus d’informations, consultez [Connexion au hub IoT Edge](iot-edge-runtime.md#connecting-to-the-iot-edge-hub).

Pour utiliser le routage IoT Edge sur AMQP ou MQTT, vous pouvez utiliser le ModuleClient du Kit de développement logiciel (SDK) Azure IoT. Créez une instance ModuleClient pour connecter votre module au hub IoT Edge en cours d’exécution sur l’appareil, de la même façon que les instances DeviceClient connectent les appareils IoT à IoT Hub. Pour plus d’informations sur la classe ModuleClient et ses méthodes de communication, reportez-vous aux informations de référence sur l’API du langage de votre SDK préféré : [C#](/dotnet/api/microsoft.azure.devices.client.moduleclient), [C](/azure/iot-hub/iot-c-sdk-ref/iothub-module-client-h), [Python](/python/api/azure-iot-device/azure.iot.device.iothubmoduleclient), [Java](/java/api/com.microsoft.azure.sdk.iot.device.moduleclient) ou [Node.js](/javascript/api/azure-iot-device/moduleclient).

<!-- <1.2> -->
::: moniker range=">=iotedge-2020-11"

Pour utiliser le répartiteur MQTT IoT Edge, vous devez apporter votre propre client MQTT et initier vous-même la connexion avec les informations que vous récupérez dans l’API de charge de travail du démon IoT Edge. <!--Need to add details here-->

Pour plus d’informations sur le choix entre le routage ou la publication/l’abonnement avec le répartiteur MQTT, consultez [Communication locale](iot-edge-runtime.md#local-communication).

### <a name="mqtt-broker-primitives"></a>Primitives du répartiteur MQTT

#### <a name="send-a-message-on-a-user-defined-topic"></a>Envoyer un message sur une rubrique définie par l’utilisateur

Grâce au répartiteur MQTT IoT Edge, vous pouvez publier des messages sur n’importe quelle rubrique définie par l’utilisateur. Pour ce faire, autorisez votre module à publier sur des rubriques spécifiques, puis procurez-vous un jeton de l’API de charge de travail à utiliser comme mot de passe lors de la connexion au répartiteur MQTT et, enfin, publiez des messages sur les rubriques autorisées avec le client MQTT de votre choix.

#### <a name="receive-messages-on-a-user-defined-topic"></a>Recevoir des messages sur une rubrique définie par l’utilisateur

Grâce au répartiteur MQTT IoT Edge, la réception de messages est similaire. Tout d’abord, assurez-vous que votre module est autorisé à s’abonner à des rubriques spécifiques, puis récupérez un jeton auprès de l’API de charge de travail à utiliser comme mot de passe lors de la connexion au répartiteur MQTT et, enfin, abonnez-vous aux messages sur les rubriques autorisées avec le client MQTT de votre choix.

::: moniker-end

### <a name="iot-hub-primitives"></a>Primitives IoT Hub

IoT Hub considère une instance de module comme un appareil, dans le sens où :

* Elle peut envoyer des [messages appareil-à-cloud](../iot-hub/iot-hub-devguide-messaging.md).
* Elle peut recevoir des [méthodes directes](../iot-hub/iot-hub-devguide-direct-methods.md) ciblant spécifiquement son identité.
* Elle a un jumeau de module, qui est distinct et isolé du [jumeau d’appareil](../iot-hub/iot-hub-devguide-device-twins.md) et des autres jumeaux de module de cet appareil.

Actuellement, les modules ne peuvent pas recevoir de messages cloud-à-appareil, ni utiliser la fonctionnalité de chargement de fichier.

Lors de l’écriture d’un module, vous pouvez vous connecter au hub IoT Edge et utiliser les primitives IoT Hub comme vous le feriez lors de l’utilisation d’IoT Hub avec une application d’appareil. La seule différence entre les modules IoT Edge et les applications d’appareil IoT est que vous devez vous référer à l’identité du module plutôt qu’à l’identité de l’appareil.

#### <a name="device-to-cloud-messages"></a>Messages appareil-à-cloud

Un module IoT Edge peut envoyer des messages au cloud via le hub IoT Edge qui agit comme un répartiteur local et propage les messages dans le cloud. Pour permettre un traitement complexe des messages appareil-à-cloud, un module IoT Edge peut également intercepter et traiter les messages envoyés par d’autres modules ou appareils à son hub IoT Edge local et envoyer de nouveaux messages avec des données traitées. Il est donc possible de créer des chaînes de modules IoT Edge pour construire des pipelines de traitement locaux.

Pour envoyer des messages de télémétrie appareil-à-cloud à l’aide du routage, utilisez le ModuleClient du Kit de développement logiciel (SDK) Azure IoT. Avec le Kit de développement logiciel (SDK) Azure IoT, chaque module présente le concept de points de terminaison d’*entrée* et de *sortie* de module, qui correspondent à des rubriques MQTT spéciales. Utilisez la méthode `ModuleClient.sendMessageAsync` ; elle enverra des messages sur le point de terminaison de sortie de votre module. Configurez ensuite un itinéraire dans edgeHub pour envoyer ce point de terminaison de sortie à IoT Hub.

<!-- <1.2> -->
::: moniker range=">=iotedge-2020-11"

L’envoi de messages de télémétrie appareil-à-cloud avec le répartiteur MQTT est similaire à la publication de messages sur des rubriques définies par l’utilisateur, mais en utilisant la rubrique spéciale IoT Hub suivante pour votre module : `devices/<device_name>/<module_name>/messages/events`. Les autorisations doivent être configurées de manière appropriée. Le pont MQTT doit également être configuré pour transférer les messages de cette rubrique vers le cloud.

::: moniker-end

Pour traiter les messages à l’aide du routage, commencez par configurer un itinéraire pour envoyer des messages provenant d’un autre point de terminaison (module ou périphérique) au point de terminaison d’entrée de votre module, puis écoutez les messages sur ce dernier. Chaque fois qu’un nouveau message revient, une fonction de rappel est déclenchée par le Kit de développement logiciel (SDK) Azure IoT. Traitez votre message avec cette fonction de rappel et envoyez éventuellement de nouveaux messages dans la file d’attente de point de terminaison de votre module.

<!-- <1.2> -->
::: moniker range=">=iotedge-2020-11"

Le traitement des messages à l’aide du répartiteur MQTT est similaire à l’abonnement à des messages sur des rubriques définies par l’utilisateur, mais en utilisant des rubriques spéciales IoT Edge de la file d’attente de sortie de votre module : `devices/<device_name>/<module_name>/messages/events`. Les autorisations doivent être configurées de manière appropriée. Si vous le souhaitez, vous pouvez envoyer de nouveaux messages dans les rubriques de votre choix.

::: moniker-end

#### <a name="twins"></a>Jumeaux

Les jumeaux sont l’une des primitives fournies par IoT Hub. Il existe des documents JSON qui stockent des informations sur l’état, notamment des métadonnées, des configurations et des conditions. Chaque module ou appareil possède son propre jumeau.

Pour obtenir un jumeau de module avec le Kit de développement logiciel (SDK) Azure IoT, appelez la méthode `ModuleClient.getTwin`.

<!-- <1.2> -->
::: moniker range=">=iotedge-2020-11"

Pour obtenir un jumeau de module avec n’importe quel client MQTT, cela demande un peu plus de travail, car l’obtention d’un jumeau n’est pas un modèle MQTT typique. Le module doit d’abord s’abonner à la rubrique spéciale IoT Hub `$iothub/twin/res/#`. Ce nom de rubrique est hérité d’IoT Hub et tous les appareils/modules doivent s’abonner à la même rubrique. Cela ne signifie pas que les appareils reçoivent le jumeau l’un de l’autre. IoT Hub et edgeHub savent quel jumeau doit être livré et où, même si tous les appareils écoutent le même nom de rubrique. Une fois l’abonnement créé, le module doit demander le jumeau en publiant un message sur la rubrique spéciale IoT Hub suivante avec l’ID de demande `$iothub/twin/GET/?$rid=1234`. Cet ID de demande est un ID arbitraire (c’est-à-dire un GUID) qui sera renvoyé par IoT Hub avec les données demandées. C’est ainsi qu’un client peut jumeler ses demandes aux réponses. Le code résultat est un code d’état de type HTTP, où la réussite est encodée sous le code 200.

::: moniker-end

Pour recevoir un patch de jumeau de module avec le Kit de développement logiciel (SDK) Azure IoT, implémentez une fonction de rappel et inscrivez-la avec la méthode `ModuleClient.moduleTwinCallback` du SDK Azure IoT afin que votre fonction de rappel soit déclenchée chaque fois qu’un patch de jumeau est disponible.

<!-- <1.2> -->
::: moniker range=">=iotedge-2020-11"

Pour recevoir un patch de jumeau de module avec un client MQTT, le processus est très similaire à la réception de jumeaux complets : un client doit s’abonner à la rubrique IoT Hub spéciale `$iothub/twin/PATCH/properties/desired/#`. Une fois l’abonnement créé, lorsque IoT Hub envoie une modification de la section souhaitée du jumeau, le client la reçoit.

::: moniker-end

#### <a name="receive-direct-methods"></a>Recevoir des méthodes directes

Pour recevoir une méthode directe avec le Kit de développement logiciel (SDK) Azure IoT, implémentez une fonction de rappel et inscrivez-la avec la méthode `ModuleClient.methodCallback` du SDK Azure IoT afin que votre fonction de rappel soit déclenchée chaque fois qu’une méthode directe est disponible.

<!-- <1.2> -->
::: moniker range=">=iotedge-2020-11"

Pour recevoir une méthode directe avec un client MQTT, le processus est très similaire à la réception de patchs de jumeau. Le client doit confirmer qu’il a reçu l’appel et peut renvoyer des informations en même temps. La rubrique IoT Hub spéciale à laquelle s’abonner est `$iothub/methods/POST/#`.

::: moniker-end

## <a name="language-and-architecture-support"></a>Prise en charge du langage et de l’architecture

IoT Edge prend en charge plusieurs systèmes d’exploitation, architectures de périphérique et langages de développement afin que vous puissiez générer le scénario qui correspond à vos besoins. Utilisez cette section pour comprendre les options de développement de modules de IoT Edge personnalisés. Vous pouvez en savoir plus sur la prise en charge des outils et les exigences pour chaque langage dans [Préparer votre environnement de développement et de test pour IOT Edge](development-environment.md).

### <a name="linux"></a>Linux

Pour toutes les langues dans le tableau suivant, IoT Edge prend en charge le développement pour les appareils AMD64 et ARM32 Linux.

| Langage de développement | Outils de développement |
| -------------------- | ----------------- |
| C | Visual Studio Code<br>Visual Studio 2017/2019 |
| C# | Visual Studio Code<br>Visual Studio 2017/2019 |
| Java | Visual Studio Code |
| Node.js | Visual Studio Code |
| Python | Visual Studio Code |

>[!NOTE]
>La prise en charge du développement et du débogage des appareils Linux ARM64 est disponible en [préversion publique](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Pour plus d’informations, consultez [Développer et déboguer des modules IoT Edge ARM64 dans Visual Studio Code (préversion)](https://devblogs.microsoft.com/iotdev/develop-and-debug-arm64-iot-edge-modules-in-visual-studio-code-preview).

### <a name="windows"></a>Windows

Pour toutes les langues dans le tableau suivant, IoT Edge prend en charge le développement pour les appareils Windows AMD64.

| Langage de développement | Outils de développement |
| -------------------- | ----------------- |
| C | Visual Studio 2017/2019 |
| C# | Visual Studio Code (aucune fonctionnalité de débogage)<br>Visual Studio 2017/2019 |

## <a name="next-steps"></a>Étapes suivantes

[Préparer votre environnement de développement et de test pour IoT Edge](development-environment.md)

[Utiliser Visual Studio pour développer des modules C# pour IoT Edge](how-to-visual-studio-develop-module.md)

[Utiliser Visual Studio Code pour développer des modules pour IoT Edge](how-to-vs-code-develop-module.md)

[Comprendre et utiliser les kits Azure IoT Hub SDK](../iot-hub/iot-hub-devguide-sdks.md)
