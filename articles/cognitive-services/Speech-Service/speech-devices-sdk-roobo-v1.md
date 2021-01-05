---
title: SDK Roobo Smart Audio v1 pour Speech Devices - Service Speech
titleSuffix: Azure Cognitive Services
description: Composants requis et instructions pour bien démarrer avec le kit de développement logiciel (SDK) Speech Devices, Roobo Smart Audio Dev Kit v1.
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 07/05/2019
ms.author: erhopf
ms.openlocfilehash: 73eb1225ffc5fd01f9a27ca99ad2b059d45a36cf
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95015288"
---
# <a name="device-roobo-smart-audio-dev-kit"></a>Appareil : Kit de développement puce Roobo Smart Audio

Cet article fournit des informations spécifiques sur l’appareil le kit de développement de Roobo Smart Audio.

## <a name="set-up-the-development-kit"></a>Installer le kit de développement

1. Le kit de développement comporte deux connecteurs micro USB. Le connecteur de gauche sert à alimenter le kit de développement ; il est désigné par le terme Alimentation sur l'image ci-dessous. Le connecteur de droite permet de contrôler le kit de développement ; il est désigné par le terme Débogage sur l'image.

    ![Connexion du kit de développement](media/speech-devices-sdk/qsg-1.png)

1. Alimentez le kit de développement à l'aide d'un câble micro USB pour connecter le port d'alimentation à un PC ou à un adaptateur secteur. Un voyant vert s'allume sous le tableau supérieur.

1. Pour contrôler le kit de développement, connectez le port de débogage à un ordinateur à l'aide d'un second câble micro USB. Pour des communications fiables, un câble de qualité doit être utilisé.

1. Orientez votre kit de développement pour la configuration circulaire ou linéaire.

    |Configuration du kit de développement|Orientation|
    |-----------------------------|------------|
    |Circulaire|Droit, les microphones vers le haut|
    |Linéaire|Sur le côté, les microphones orientés vers vous (comme illustré ci-après)|

    ![Orientation linéaire du kit de développement](media/speech-devices-sdk/qsg-2.png)

1. Installez les certificats et définissez les autorisations du périphérique audio. Tapez les commandes ci-après dans une fenêtre d’invite de commandes :

   ```powershell
   adb push C:\SDSDK\Android-Sample-Release\scripts\roobo_setup.sh /data/
   adb shell
   cd /data/
   chmod 777 roobo_setup.sh
   ./roobo_setup.sh
   exit
   ```

    > [!NOTE]
    > Ces commandes utilisent Android Debug Bridge, `adb.exe`, compris dans l’installation d’Android Studio. Cet outil figure dans le répertoire C:\Users\[nom de l’utilisateur]\AppData\Local\Android\Sdk\platform-tools. Vous pouvez ajouter ce répertoire à votre chemin d’accès pour que l’invocation de l’outil `adb` soit plus pratique. Dans le cas contraire, vous devez spécifier le chemin d’accès complet à votre installation d’adb.exe dans chaque commande qui invoque `adb`.
    >
    > Si vous rencontrez une erreur `no devices/emulators found`, vérifiez que votre câble USB est bien connecté et qu'il s'agit d'un câble de qualité. Vous pouvez utiliser `adb devices` pour vérifier que votre ordinateur peut communiquer avec le kit de développement car il renverra une liste d'appareils.
    >
    > [!TIP]
    > Désactivez le micro et le haut-parleur de votre PC pour vous assurer que vous travaillez avec les micros du kit de développement. De cette façon, vous n’activerez pas l’appareil par accident à cause du son du PC.

1. Si vous souhaitez raccorder un haut-parleur au kit de développement, vous pouvez le connecter à la sortie de la ligne audio. Vous devez choisir un haut-parleur de bonne qualité avec une prise analogique 3,5 mm.

    ![Audio Vysor](media/speech-devices-sdk/qsg-14.png)

## <a name="development-information"></a>Informations sur le développement

Pour plus d’informations sur le développement, consultez le [guide de développement de Roobo](http://dwn.roo.bo/server_upload/ddk/ROOBO%20Dev%20Kit-User%20Guide.pdf).

## <a name="audio"></a>Audio

Roobo fournit un outil qui capture toutes les données audio en mémoire flash. Cet outil peut vous aider à résoudre les problèmes audio. Une version de cet outil est fournie avec chaque configuration de kit de développement. Sur le [site Roobo](http://ddk.roobo.com/), sélectionnez votre appareil, puis sélectionnez le lien **ROOBO Tools** (Outils Roobo) au bas de la page.

## <a name="next-steps"></a>Étapes suivantes

* [Exécuter l’application exemple Android](./speech-devices-sdk-quickstart.md?pivots=platform-android%253fpivots%253dplatform-android)