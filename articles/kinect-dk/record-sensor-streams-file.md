---
title: Démarrage rapide – Enregistrer les flux de capteur Azure Kinect dans un fichier
description: Dans ce guide de démarrage rapide, vous allez découvrir comment enregistrer les flux de données du SDK Capteur dans un fichier.
author: tesych
ms.author: tesych
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: quickstart
keywords: azure, kinect, enregistrement, lecture, lecteur, matroska, mkv, flux, profondeur, rvb, caméra, couleur, imu, audio, capteur
ms.openlocfilehash: 3dab147b593bf012bd6cd9c95d0195e84a2cbcf1
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "85277792"
---
# <a name="quickstart-record-azure-kinect-sensor-streams-to-a-file"></a>Démarrage rapide : Enregistrer les flux de capteur Azure Kinect dans un fichier

Ce guide de démarrage rapide fournit des informations sur la façon dont vous pouvez utiliser l’outil [enregistreur Kinect Azure](azure-kinect-recorder.md) pour enregistrer les flux de données du SDK Capteur dans un fichier.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Ce guide de démarrage rapide suppose que :

- Azure Kinect DK est connecté à votre PC hôte et correctement alimenté.
- Vous avez terminé la [configuration](set-up-azure-kinect-dk.md) du matériel.

## <a name="create-recording"></a>Créer un enregistrement

1. Ouvrez une invite de commandes et indiquez le chemin de l’[enregistreur Azure Kinect](azure-kinect-recorder.md), qui figure dans le répertoire des outils installés sous le nom `k4arecorder.exe`. Par exemple : `C:\Program Files\Azure Kinect SDK\tools\k4arecorder.exe`.
2. Faites un enregistrement d’une durée de 5 secondes.

    `k4arecorder.exe -l 5 %TEMP%\output.mkv`

3. Par défaut, l’enregistreur utilise le mode de profondeur NFOV sans compartimentation et génère 1 080 p RVB à 30 i/s avec des données IMU. Pour obtenir une vue d’ensemble complète des options d’enregistrement ainsi que des conseils, consultez [Enregistreur Azure Kinect](azure-kinect-recorder.md).

## <a name="play-back-recording"></a>Lire l’enregistrement

Vous pouvez utiliser la [visionneuse Azure Kinect](azure-kinect-viewer.md) pour lire un enregistrement.

1. Lancez [`k4aviewer.exe`](azure-kinect-viewer.md).
2. Déroulez l’onglet **Open Recording** (Ouvrir un enregistrement) et ouvrez votre enregistrement.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous savez comment enregistrer les flux de capteur dans un fichier, il est temps de

> [!div class="nextstepaction"]
> [Créer votre première application Azure Kinect](build-first-app.md)
