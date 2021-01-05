---
title: Guide pratique pour configurer un réseau de microphones - Speech Service
titleSuffix: Azure Cognitive Services
description: Découvrez comment configurer un réseau de microphones de sorte que le SDK Speech Devices puisse l’utiliser.
services: cognitive-services
author: mswellsi
manager: yanbo
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 05/01/2020
ms.author: wellsi
ms.openlocfilehash: cf0580c96f5bf78f0444b2bb39088f2a417fd658
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95025059"
---
# <a name="how-to-configure-a-microphone-array"></a>Guide pratique pour configurer un réseau de microphones

Dans cet article, vous allez apprendre à configurer un [réseau de microphones](./speech-devices-sdk-microphone.md). Cela inclut la définition de l’angle de travail et le mode de sélection du microphone utilisé pour le SDK Speech Devices.

Le fonctionnement du SDK Speech Devices est optimal avec un réseau de microphones conçu conformément à [nos instructions](./speech-devices-sdk-microphone.md). La configuration d’un réseau de microphones peut être fournie par le système d’exploitation ou obtenue par le biais de l’une des méthodes suivantes.

Le SDK Speech Devices prenait initialement en charge les réseaux de microphones en sélectionnant une configuration parmi un ensemble fixe de configurations.

```java
private static String DeviceGeometry = "Circular6+1"; // "Circular6+1", "Linear4",
private static String SelectedGeometry = "Circular6+1"; // "Circular6+1", "Circular3+1", "Linear4", "Linear2"
```

Sur Windows, la configuration des réseaux de microphones est fournie par le pilote audio.

À compter de la version 1.11.0, le SDK Speech Devices prend également en charge la configuration à partir d’un [fichier JSON](https://aka.ms/sdsdk-micarray-json).


## <a name="windows"></a>Windows
Sur Windows, les informations sur la géométrie du réseau de microphones sont obtenues automatiquement auprès du pilote audio. Par conséquent, les propriétés `DeviceGeometry`, `SelectedGeometry` et `MicArrayGeometryConfigFile` sont facultatives. Nous utilisons le [fichier JSON](https://aka.ms/sdsdk-micarray-json) fourni à l’aide de `MicArrayGeometryConfigFile` pour obtenir uniquement la plage de formation de faisceaux.

Si un réseau de microphones est spécifié à l’aide de `AudioConfig::FromMicrophoneInput`, nous utilisons le microphone spécifié. Si aucun microphone n’est spécifié ou si `AudioConfig::FromDefaultMicrophoneInput` est appelé, nous utilisons le microphone par défaut qui est spécifié dans les paramètres Son sur Windows.
La pile audio Microsoft dans le SDK Speech Devices prend uniquement en charge le sous-échantillonnage des taux d’échantillonnage qui sont des multiples entiers de 16 KHz.

## <a name="linux"></a>Linux
Sur Linux, les informations sur la géométrie du microphone doivent être fournies. L’utilisation de `DeviceGeometry` et de `SelectedGeometry` est toujours prise en charge. Elles peuvent également être fournies par le biais du fichier JSON à l’aide de la propriété `MicArrayGeometryConfigFile`. Comme pour Windows, la plage de formation de faisceaux peut être fournie par le fichier JSON.

Si un réseau de microphones est spécifié à l’aide de `AudioConfig::FromMicrophoneInput`, nous utilisons le microphone spécifié. Si aucun microphone n’est spécifié ou si `AudioConfig::FromDefaultMicrophoneInput` est appelé, nous effectuons les enregistrements à partir de l’appareil ALSA nommé *default*. Par défaut, *default* pointe toujours sur la carte 0 de l’appareil 0, mais les utilisateurs peuvent le changer dans le fichier `asound.conf`. 

Dans le SDK Speech Devices, la pile audio Microsoft prend uniquement en charge le sous-échantillonnage des taux d’échantillonnage qui sont des multiples entiers de 16 KHz. De plus, les formats suivants sont pris en charge : valeur float IEEE 32 bits avec primauté des octets de poids faible (Little Endian), entier signé 32 bits avec primauté des octets de poids faible (Little Endian), entier signé 24 bits avec primauté des octets de poids faible (Little Endian), entier signé 16 bits avec primauté des octets de poids faible (Little Endian) et entier signé 8 bits avec primauté des octets de poids faible (Little Endian).

## <a name="android"></a>Android
Actuellement, seul [Roobo v1](./speech-devices-sdk-quickstart.md?pivots=platform-android%253fpivots%253dplatform-android) est pris en charge par le SDK Speech Devices. Le comportement est identique à celui des versions précédentes, excepté que maintenant, la propriété `MicArrayGeometryConfigFile` peut être utilisée pour spécifier le fichier JSON contenant la plage de formation de faisceaux.

## <a name="microphone-array-configuration-json"></a>Fichier JSON de configuration du réseau de microphones

Le fichier JSON pour la configuration de la géométrie du réseau de microphones suit le [schéma JSON](https://aka.ms/sdsdk-micarray-json). Voici quelques exemples qui suivent le schéma.


```json
{
    "micArrayType": "Linear",
    "geometry": "Linear4"
}
```


ou


```json
{
    "micArrayType": "Planar",
    "horizontalAngleBegin": 0,
    "horizontalAngleEnd": 360,
    "numberOfMicrophones": 4,
    "micCoord": [
        {
            "xCoord": 0,
            "yCoord": 0,
            "zCoord": 0
        },
        {
            "xCoord": 40,
            "yCoord": 0,
            "zCoord": 0
        },
        {
            "xCoord": -20,
            "yCoord": -35,
            "zCoord": 0
        },
        {
            "xCoord": -20,
            "yCoord": 35,
            "zCoord": 0
        }
    ]
}
```

ou

```json
{
    "micArrayType": "Linear",
    "horizontalAngleBegin": 70,
    "horizontalAngleEnd": 110
}
```


## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Choisir votre appareil vocal](get-speech-devices-sdk.md)