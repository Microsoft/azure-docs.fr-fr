---
title: SDK Speech Devices, kit de développement Roobo Smart Audio v2 - Speech Service
titleSuffix: Azure Cognitive Services
description: Prérequis et instructions pour bien démarrer avec le SDK Speech Devices, kit de développement Roobo Smart Audio v2.
services: cognitive-services
author: anushapatnala
manager: wellsi
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 02/20/2020
ms.author: v-anusp
ms.openlocfilehash: 2f325c80877068c53d690bd7ff74f768dab3a174
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "80371576"
---
# <a name="device-roobo-smart-audio-dev-kit-v2"></a>Appareil : Kit de développement Roobo Smart Audio v2

Cet article fournit des informations spécifiques à l’appareil pour le kit de développement Roobo Smart Audio 2.

## <a name="set-up-the-development-kit"></a>Installer le kit de développement

1. Le kit de développement comporte deux connecteurs micro USB. Le connecteur de gauche sert à alimenter le kit de développement ; il est désigné par le terme Alimentation sur l'image ci-dessous. Le connecteur de droite permet de contrôler le kit de développement ; il est désigné par le terme Débogage sur l'image. 
    ![Connexion du kit de développement](media/speech-devices-sdk/roobo-v2-connections.png)
1. Alimentez le kit de développement à l'aide d'un câble micro USB pour connecter le port d'alimentation à un PC ou à un adaptateur secteur. Un voyant vert s'allume sous le tableau supérieur.
1. Pour contrôler le kit de développement, connectez le port de débogage à un ordinateur à l'aide d'un second câble micro USB. Il est essentiel d’utiliser un câble de haute qualité pour garantir des communications fiables.
1. Orientez votre kit de développement de façon circulaire : droit, les microphones vers le haut comme indiqué ci-dessus


## <a name="development-information"></a>Informations sur le développement

Pour plus d’informations sur le développement, consultez le [guide de développement de Roobo](http://dwn.roo.bo/server_upload/ddk/ROOBO%20Dev%20Kit-User%20Guide.pdf).

## <a name="audio-recordplay"></a>Enregistrement/lecture audio

Les opérations audio DDK2 peuvent être effectuées des manières suivantes :
* Utilisez des bibliothèques open source ALSA et leurs applications.
* Utilisez l’interface appmainprog pour le développement d’applications. Le framework logiciel relatif à l’audio DDK2 utilise le framework ALSA standard ; vous pouvez utiliser libasound pour développer directement les logiciels. Vous pouvez donc utiliser arecord et aplay d’ALSA directement pour enregistrer et lire des données audio.
