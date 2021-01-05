---
title: Mettre à jour le microprogramme Azure Kinect DK
description: Découvrez comment mettre à jour le microprogramme d’un appareil Azure Kinect DK à l’aide de l’outil de microprogramme Kinect Azure.
author: tesych
ms.author: tesych
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: conceptual
keywords: kinect, microprogramme, mise à jour, récupération
ms.openlocfilehash: 71557798a584635356cc47fd7c4a0309df4d018a
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94356221"
---
# <a name="update-azure-kinect-dk-firmware"></a>Mettre à jour le microprogramme Azure Kinect DK

Ce document fournit des conseils sur la façon de mettre à jour le microprogramme de votre appareil Azure Kinect DK.

Azure Kinect DK ne met pas à jour le microprogramme automatiquement. Vous pouvez utiliser l’[outil du microprogramme Azure Kinect](azure-kinect-firmware-tool.md) pour mettre à jour le microprogramme manuellement vers la dernière version disponible.

## <a name="prepare-for-firmware-update"></a>Préparer la mise à jour du microprogramme

1. [Télécharger le SDK](sensor-sdk-download.md).
2. Installez le kit SDK.
3. Dans l’emplacement d’installation du Kit de développement logiciel (SDK) sous (emplacement d’installation du SDK)\tools\, vous devriez trouver :

    - AzureKinectFirmwareTool.exe
    - Fichier .bin de microprogramme dans le dossier du microprogramme, par exemple *AzureKinectDK_Fw_1.5.926614. bin*.

4. Connectez votre appareil à l’ordinateur hôte et mettez-le en marche.

> [!IMPORTANT]
> Veillez à ce que les connexions USB et d’alimentation soient correctement établies pendant la mise à jour du microprogramme. La suppression de l’une ou l’autre risque d’altérer le microprogramme.

## <a name="update-device-firmware"></a>Mettre à jour le microprogramme de l’appareil

1. Ouvrez une invite de commandes dans le dossier (emplacement d’installation du SDK)\tools\.
2. Mettre à jour le microprogramme à l’aide de l’outil de microprogramme Azure Kinect

    `AzureKinectFirmwareTool.exe -u <device_firmware_file.bin>`

    Exemple :

    `AzureKinectFirmwareTool.exe -u firmware\AzureKinectDK_Fw_1.5.926614.bin`

3. Attendez la fin de la mise à jour du microprogramme. Cela peut prendre quelques minutes en fonction de la taille de l’image.

### <a name="verify-device-firmware-version"></a>Vérifier la version du microprogramme de l’appareil

1. Vérifiez que le microprogramme est à jour.

    `AzureKinectFirmwareTool.exe -q`

2. Consultez l’exemple suivant.

    ```console
       >AzureKinectFirmwareTool.exe -q
       == Azure Kinect DK Firmware Tool ==
        Device Serial Number: 000805192412
        Current Firmware Versions:
        RGB camera firmware:      1.6.102
        Depth camera firmware:    1.6.75
        Depth config file:        6109.7
        Audio firmware:           1.6.14
        Build Config:             Production
        Certificate Type:         Microsoft
    ```

3. Si vous voyez la sortie ci-dessus, cela signifie que le microprogramme est à jour.

4. Après la mise à jour du microprogramme, vous pouvez exécuter la [visionneuse Azure Kinect](azure-kinect-viewer.md) pour vérifier que tous les capteurs fonctionnent comme prévu.

## <a name="troubleshooting"></a>Dépannage

Les mises à jour du microprogramme peuvent échouer pour plusieurs raisons. En cas d’échec d’une mise à jour de microprogramme, essayez de suivre les étapes d’atténuation suivantes :

1. Réessayez d’exécuter la commande de mise à jour du microprogramme.

2. Vérifiez que l’appareil est toujours connecté en interrogeant la version du microprogramme.        AzureKinectFirmareTool.exe

3. Si tout échoue, suivez les étapes de [récupération](https://support.microsoft.com/help/4494277/reset-azure-kinect-dk) pour revenir au microprogramme d’usine, puis réessayez.

Pour tout problème supplémentaire, consultez les [pages de support de Microsoft](./index.yml)

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
>[Outil du microprogramme Azure Kinect](azure-kinect-firmware-tool.md)