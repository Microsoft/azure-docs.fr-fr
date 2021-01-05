---
title: Télécharger le Kit de développement logiciel (SDK) du capteur Azure Kinect
description: Découvrez comment télécharger et installer le Kit de développement logiciel (SDK) du capteur Azure Kinect sur Windows et Linux.
author: Brent-A
ms.author: brenta
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: conceptual
keywords: azure, kinect, sdk, mise à jour de téléchargement, dernier, disponible, installer
ms.openlocfilehash: 2fd14781c42192c713d826729f8fab6c698d6321
ms.sourcegitcommit: 2ba6303e1ac24287762caea9cd1603848331dd7a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97505475"
---
# <a name="azure-kinect-sensor-sdk-download"></a>Télécharger le Kit de développement logiciel (SDK) du capteur Azure Kinect

Cette page reprend les liens de téléchargement pour chaque version du Kit de développement logiciel (SDK) du capteur Azure Kinect. Le programme d’installation fournit tous les fichiers nécessaires à développer pour Azure Kinect.

## <a name="azure-kinect-sensor-sdk-contents"></a>Contenus du Kit de développement logiciel (SDK) du capteur Azure Kinect

- En-têtes et bibliothèques pour créer une application à l’aide de l’Azure Kinect DK.
- DLL redistribuables requises par les applications utilisant l’Azure Kinect DK.
- La [Visionneuse Azure Kinect](azure-kinect-viewer.md).
- L’[Enregistreur Azure Kinect](azure-kinect-recorder.md).
- L’[Outil du microprogramme Azure Kinect](azure-kinect-firmware-tool.md).

## <a name="windows-installation-instructions"></a>Instructions d’installation Windows

Vous trouverez des détails sur l’installation des versions les plus récentes et précédentes du Kit de développement logiciel (SDK) et du microprogramme du capteur Azure Kinect [ici](https://github.com/microsoft/Azure-Kinect-Sensor-SDK/blob/develop/docs/usage.md).

Vous trouverez le fichier source [ici](https://github.com/microsoft/Azure-Kinect-Sensor-SDK).

> [!NOTE]
> Lorsque vous installez le Kit de développement logiciel (SDK), rappelez-vous le chemin d’installation. Par exemple, « C:\Program Files\Azure Kinect SDK 1.2 ». Vous trouverez les outils référencés dans les articles à ce chemin.

## <a name="linux-installation-instructions"></a>Instructions d’installation Linux

Actuellement, la seule distribution prise en charge est Ubuntu 18.04. Pour demander la prise en charge d’autres distributions, consultez [cette page](https://aka.ms/azurekinectfeedback).

Tout d’abord, vous devez configurer le [dépôt du package Microsoft](https://packages.microsoft.com/), en suivant les instructions reprises [ici](/windows-server/administration/linux-package-repository-for-microsoft-software).

Vous pouvez maintenant installer les packages nécessaires. Le `k4a-tools` package inclut la [Visionneuse Azure Kinect](azure-kinect-viewer.md), l’[enregistreur Azure Kinect](record-sensor-streams-file.md) et l’[outil du microprogramme Azure Kinect](azure-kinect-firmware-tool.md). Pour l'installer, exécutez

 `sudo apt install k4a-tools`

 Le package `libk4a<major>.<minor>-dev` contient les en-têtes et les fichiers CMake à créer par rapport à `libk4a`.
Le package `libk4a<major>.<minor>` contient les objets partagés nécessaires pour exécuter des exécutables qui dépendent de `libk4a`.

 Les didacticiels de base nécessitent le package `libk4a<major>.<minor>-dev`. Pour l'installer, exécutez

 `sudo apt install libk4a1.1-dev`

Si la commande est réussie, le Kit de développement logiciel (SDK) est prêt à être utilisé.

## <a name="change-log-and-older-versions"></a>Journal des modifications et versions antérieures

Vous trouverez le journal des modifications pour le Kit de développement logiciel (SDK) [ici](https://github.com/microsoft/Azure-Kinect-Sensor-SDK/blob/develop/CHANGELOG.md).

Si vous avez besoin d’une version antérieure du Kit de développement logiciel (SDK) du capteur Azure Kinect, recherchez-la [ici](https://github.com/microsoft/Azure-Kinect-Sensor-SDK/blob/develop/docs/usage.md).

## <a name="next-steps"></a>Étapes suivantes

[Configurer Azure Kinect DK](set-up-azure-kinect-dk.md)