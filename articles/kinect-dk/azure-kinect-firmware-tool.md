---
title: Outil du microprogramme Azure Kinect
description: Découvrez comment interroger et mettre à jour le microprogramme d’un appareil à l’aide de l’outil de microprogramme Azure Kinect.
author: tesych
ms.author: tesych
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: conceptual
keywords: kinect, microprogramme, mise à jour
ms.openlocfilehash: f8b89751d27fb5a4b18d635f45f63f4f36bd05f6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85276499"
---
# <a name="azure-kinect-dk-firmware-tool"></a>Outil de microprogramme Azure Kinect DK

L’outil de microprogramme Azure Kinect permet d’interroger et de mettre à jour le microprogramme de l’appareil Azure Kinect DK.

## <a name="list-connected-devices"></a>Afficher les unités connectées

Vous pouvez obtenir la liste des appareils connectés à l’aide de l’option -l.  `AzureKinectFirmwareTool.exe -l`

```console
 == Azure Kinect DK Firmware Tool ==
Found 2 connected devices:
0: Device "000036590812"
1: Device "000274185112"
```

## <a name="check-device-firmware-version"></a>Vérifier la version du microprogramme de l’appareil

Vous pouvez vérifier les versions actuelles du microprogramme du premier appareil connecté à l’aide de l’option -q, par exemple, `AzureKinectFirmwareTool.exe -q`.

```console
 == Azure Kinect DK Firmware Tool ==
Device Serial Number: 000036590812
Current Firmware Versions:
    RGB camera firmware:      1.5.92
    Depth camera firmware:    1.5.66
    Depth config file:        6109.7
    Audio firmware:           1.5.14
    Build Config:             Production
    Certificate Type:         Microsoft
```

Si plusieurs appareils sont connectés, vous pouvez spécifier l’appareil que vous souhaitez interroger en ajoutant le numéro de série complet à la commande, par exemple :

`AzureKinectFirmwareTool.exe -q 000036590812`

## <a name="update-device-firmware"></a>Mettre à jour le microprogramme de l’appareil

L’utilisation la plus courante de cet outil est la mise à jour du microprogramme de l’appareil. Effectuez la mise à jour en appelant l’outil à l’aide de l’option `-u`. Une mise à jour du microprogramme peut prendre quelques minutes, en fonction des fichiers de microprogramme à mettre à jour.

Pour obtenir des instructions pas à pas sur la mise à jour du microprogramme, consultez [Mise à jour du microprogramme Azure Kinect](update-device-firmware.md).  

`AzureKinectFirmwareTool.exe -u firmware\AzureKinectDK_Fw_1.5.926614.bin`

Si plusieurs appareils sont connectés, vous pouvez spécifier celui que vous souhaitez interroger en ajoutant son numéro de série complet à la commande.

`AzureKinectFirmwareTool.exe -u firmware\AzureKinectDK_Fw_1.5.926614.bin 000036590812`

## <a name="reset-device"></a>Réinitialisation de l’appareil

Vous pouvez réinitialiser un appareil Azure Kinect DK connecté à l’aide de l’option -r si vous devez faire passer l’appareil dans un état connu.

Si plusieurs appareils sont connectés, vous pouvez spécifier celui que vous souhaitez interroger en ajoutant son numéro de série complet à la commande.

`AzureKinectFirmwareTool.exe -r 000036590812`

## <a name="inspect-firmware"></a>Inspecter le microprogramme

L’inspection du microprogramme vous permet d’obtenir les informations de version à partir d’un fichier bin du microprogramme avant de mettre à jour un appareil réel.

`AzureKinectFirmwareTool.exe -i firmware\AzureKinectDK_Fw_1.5.926614.bin`

```console
 == Azure Kinect DK Firmware Tool ==
Loading firmware package ..\tools\updater\firmware\AzureKinectDK_Fw_1.5.926614.bin.
File size: 1228844 bytes
This package contains:
  RGB camera firmware:      1.5.92
  Depth camera firmware:    1.5.66
  Depth config files: 6109.7 5006.27
  Audio firmware:           1.5.14
  Build Config:             Production
  Certificate Type:         Microsoft
  Signature Type:           Microsoft
```

## <a name="firmware-update-tool-options"></a>Options de l’outil de mise à jour du microprogramme

```console
 == Azure Kinect DK Firmware Tool ==
* Usage Info *
    AzureKinectFirmwareTool.exe <Command> <Arguments>

Commands:
    List Devices: -List, -l
    Query Device: -Query, -q
        Arguments: [Serial Number]
    Update Device: -Update, -u
        Arguments: <Firmware Package Path and FileName> [Serial Number]
    Reset Device: -Reset, -r
        Arguments: [Serial Number]
    Inspect Firmware: -Inspect, -i
        Arguments: <Firmware Package Path and FileName>

    If no Serial Number is provided, the tool will just connect to the first device.

Examples:
    AzureKinectFirmwareTool.exe -List
    AzureKinectFirmwareTool.exe -Update c:\data\firmware.bin 0123456
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
>[Instructions pas à pas pour mettre à jour le microprogramme d’un appareil](update-device-firmware.md)
