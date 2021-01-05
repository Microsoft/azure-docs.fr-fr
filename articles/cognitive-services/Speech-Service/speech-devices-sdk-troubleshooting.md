---
title: Résoudre les problèmes relatifs au SDK Speech Devices - Service Speech
titleSuffix: Azure Cognitive Services
description: Cet article fournit des informations pour vous aider à résoudre des problèmes que vous pourriez rencontrer lors de l’utilisation du kit SDK Speech Devices.
services: cognitive-services
author: mswellsi
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 07/05/2019
ms.author: wellsi
ms.openlocfilehash: c590e0972de289a36890a75b220eddbded701ea8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "74815565"
---
# <a name="troubleshoot-the-speech-devices-sdk"></a>Résoudre les problèmes liés au SDK des dispositifs vocaux

Cet article fournit des informations pour vous aider à résoudre des problèmes que vous pourriez rencontrer lors de l’utilisation du kit SDK Speech Devices.

## <a name="certificate-failures"></a>Erreurs de certificat

Si vous rencontrez des erreurs de certificat lorsque vous utilisez le service Speech, vérifiez que la date et l’heure de votre appareil sont correctes :

1. Accédez à **Settings**. Sous **System** (Système), sélectionnez **Date & time** (Date et heure).

    ![Sélection du paramètre de date et d’heure](media/speech-devices-sdk/qsg-12.png)

1. Laissez l’option **Automatic date & time** (Date et heure automatiques) sélectionnée. Sous **Select time zone** (Sélectionner un fuseau horaire), sélectionnez votre fuseau horaire actuel.

    ![Sélection des options de date et de fuseau horaire](media/speech-devices-sdk/qsg-13.png)

    Lorsque vous voyez que l’heure du kit de développement correspond à l’heure de votre PC, le kit de développement est connecté à Internet.

## <a name="next-steps"></a>Étapes suivantes

* [Examiner les notes de publication](devices-sdk-release-notes.md)
