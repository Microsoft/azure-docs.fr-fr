---
title: Démarrage rapide – Configurer le suivi des corps Azure Kinect
description: Dans ce guide de démarrage rapide, vous allez configurer le SDK Suivi des corps pour Azure Kinect.
author: qm13
ms.author: quentinm
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: quickstart
keywords: kinect, azure, capteur, accès, profondeur, sdk, corps, suivi, articulation, configuration, cuda, nvidia
ms.openlocfilehash: 2cf4c1097730f88fc4bd66c28e1bdddd7fea8640
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "85277801"
---
# <a name="quickstart-set-up-azure-kinect-body-tracking"></a>Démarrage rapide : Configurer le suivi des corps Azure Kinect

Ce guide de démarrage rapide vous accompagne tout au long du processus de mise en service du suivi des corps sur votre appareil Azure Kinect DK.

## <a name="system-requirements"></a>Configuration système requise

Le PC hôte du SDK Suivi des corps doit être équipé d’un GPU NVIDIA. La configuration recommandée du PC hôte du SDK Suivi des corps est décrite dans la page [Configuration système requise](system-requirements.md).

## <a name="install-software"></a>Installer les logiciels

### <a name="install-the-latest-nvidia-driver"></a>[Installer la dernière version du pilote NVIDIA](https://www.nvidia.com/Download/index.aspx?lang=en-us)

Téléchargez et installez la dernière version du pilote NVIDIA de votre carte graphique. Les pilotes plus anciens risquent de ne pas être compatibles avec les binaires CUDA redistribués avec le SDK Suivi des corps.

### <a name="visual-c-redistributable-for-visual-studio-2015"></a>[Package redistribuable Visual C++ pour Visual Studio 2015](https://www.microsoft.com/en-us/download/details.aspx?id=48145)

Téléchargez et installez Redistributable Visual C++ pour Visual Studio 2015. 

## <a name="set-up-hardware"></a>Configurer le matériel

### <a name="set-up-azure-kinect-dk"></a>[Configurer Azure Kinect DK](set-up-azure-kinect-dk.md)

Lancez la [visionneuse Azure Kinect](azure-kinect-viewer.md) pour vérifier que votre appareil Azure Kinect DK est correctement configuré.

## <a name="download-the-body-tracking-sdk"></a>Télécharger le SDK Suivi des corps
 
1. Sélectionnez le lien de [téléchargement du SDK Suivi des corps](body-sdk-download.md).
2. Installez le SDK Suivi des corps sur votre PC.

## <a name="verify-body-tracking"></a>Vérifier le suivi des corps

Lancez la **visionneuse de suivi des corps Azure Kinect** pour vérifier que le SDK Suivi des corps est correctement configuré. La visionneuse s’installe avec le programme d’installation msi du SDK. Elle se trouve dans le menu de démarrage ou dans `<SDK Installation Path>\tools\k4abt_simple_3d_viewer.exe`.

Si votre GPU n’est pas suffisamment puissant et que vous souhaitez quand même tester le résultat, vous pouvez lancer la **visionneuse de suivi des corps Azure Kinect** sur la ligne de commande en utilisant la commande suivante : `<SDK Installation Path>\tools\k4abt_simple_3d_viewer.exe CPU`

Si tout est correctement configuré, une fenêtre contenant un nuage de points 3D et les corps suivis s’affiche à l’écran.


![Visionneuse 3D du suivi des corps](./media/quickstarts/samples-simple3dviewer.png)

## <a name="examples"></a>Exemples

Les exemples d’utilisation du SDK Suivi des corps se trouvent [ici](https://github.com/microsoft/Azure-Kinect-Samples/tree/master/body-tracking-samples).

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
>[Créer votre première application de suivi des corps](build-first-body-app.md)

