---
title: Obtenir le Kit de développement logiciel (SDK) de dispositifs vocaux
titleSuffix: Azure Cognitive Services
description: Le service Speech fonctionne avec une large gamme d’appareils et de sources audio. Vous pouvez maintenant faire passer vos applications de reconnaissance vocale au niveau supérieur en y associant du matériel et des logiciels. Dans cet article, vous allez découvrir comment accéder au kit SDK Speech Devices et commencer à développer.
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 04/14/2019
ms.author: erhopf
ms.openlocfilehash: 69898e64934c363a18a3ce2fa5e678116624bd24
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95026367"
---
# <a name="get-the-cognitive-services-speech-devices-sdk"></a>Obtenir le kit de développement logiciel (SDK) Speech Devices de Cognitive Services

Le kit SDK Speech Devices est une bibliothèque préconfigurée conçue pour fonctionner avec des kits de développement spécialisés et différentes configurations de réseau de microphones.

## <a name="choose-a-development-kit"></a>Choisir un kit de développement

|Appareils|Caractéristique|Description|Scénarios|
|--|--|--|--|
|[Kit de développement URbetter](http://www.urbetter.com/products_56/278.html)![URbetter DDK](media/speech-devices-sdk/device-urbetter.jpg)|Ensemble de 7 micros, SoC ARM, Wi-Fi, Ethernet, HDMI, caméra USB. <br>Linux|Un kit SDK Speech Devices de niveau industriel qui adapte l’ensemble Microsoft Mic Array et prend en charge de nombreuses E/S, comme HDMI/Ethernet et d’autres périphériques USB. <br> [Contacter Urbetter](http://www.urbetter.com/products_56/278.html)|Transcription de conversation, enseignement, hôpital, robots, boîte OTT, la agent vocal, Drive Thru|
|[Kit de développement Roobo Smart Audio](http://ddk.roobo.com)<br>[Configuration](speech-devices-sdk-roobo-v1.md) / [Démarrage rapide](./speech-devices-sdk-quickstart.md?pivots=platform-android%253fpivots%253dplatform-android)![Kit de développement Roobo Smart Audio](media/speech-devices-sdk/device-roobo-v1.jpg)|Ensemble de 7 micros, SoC ARM, Wi-Fi, sortie audio, E/S. <br>[Android](./speech-devices-sdk-quickstart.md?pivots=platform-android%253fpivots%253dplatform-android)|Le premier kit SDK Speech Devices à adapter l’ensemble Microsoft Mic Array et le kit SDK de traitement frontal pour développer des scénarios de transcription et de reconnaissance vocale de qualité.|Transcription de conversation, enceinte connectée, agent vocal, technologie portable|
|[Azure Kinect DK](https://azure.microsoft.com/services/kinect-dk/)<br>[Configurer](../../kinect-dk/set-up-azure-kinect-dk.md) / [Démarrage rapide](./speech-devices-sdk-quickstart.md?pivots=platform-windows%253fpivots%253dplatform-windows)![Azure Kinect DK](media/speech-devices-sdk/device-azure-kinect-dk.jpg)|Caméras de profondeur et RVB avec ensemble de 7 micros. <br>[Windows](./speech-devices-sdk-quickstart.md?pivots=platform-windows%253fpivots%253dplatform-windows)/[Linux](./speech-devices-sdk-quickstart.md?pivots=platform-linux%253fpivots%253dplatform-linux)|Kit de développement doté de capteurs d’intelligence artificielle (IA) avancés pour élaborer des modèles vocaux et des modèles de vision par ordinateur sophistiqués. Il associe un ensemble de microphones spatiaux de pointe et une caméra de profondeur à une caméra vidéo et à un capteur d'orientation, le tout au sein d'un petit dispositif doté de différents modes, options et kits SDK pour gérer différents types de calcul.|Transcription de conversation, robotique, bâtiment intelligent|
|Kit de développement Roobo Smart Audio 2<br>[Paramétrage](speech-devices-sdk-roobo-v2.md)<br>![Kit de développement Roobo Smart Audio 2](media/speech-devices-sdk/device-roobo-v2.jpg)|Ensemble de 7 micros, SoC ARM, Wi-Fi, Bluetooth, E/S. <br>Linux|Le kit SDK Speech Devices deuxième génération, qui offre un autre système d’exploitation et des fonctionnalités supplémentaires dans une conception de référence économique.|Transcription de conversation, enceinte connectée, agent vocal, technologie portable|


## <a name="download-the-speech-devices-sdk"></a>Télécharger le kit de développement logiciel (SDK) Speech Devices

Télécharger le [kit SDK Speech Devices](./speech-devices-sdk.md).

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Prise en main du kit de développement logiciel (SDK) Speech Devices](./speech-devices-sdk-quickstart.md?pivots=platform-android)