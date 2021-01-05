---
title: Création et chargement d’un disque dur virtuel Flatcar Container Linux pour Azure
description: Apprenez à créer et à charger un disque dur virtuel contenant un système d’exploitation Flatcar Container Linux.
author: marga-kinvolk
ms.author: danis
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 07/16/2020
ms.reviewer: cynthn
ms.openlocfilehash: 555e53899ed78a5200009d04659e974f8157057e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87268237"
---
# <a name="using-a-prebuilt-flatcar-image-for-azure"></a>Utilisation d’une image Flatcar prédéfinie pour Azure

Vous pouvez télécharger des images de disque dur virtuel Azure prédéfinies de Flatcar Container Linux pour chacun des canaux Flatcar pris en charge :

- [stable](https://stable.release.flatcar-linux.net/amd64-usr/current/flatcar_production_azure_image.vhd.bz2)
- [beta](https://beta.release.flatcar-linux.net/amd64-usr/current/flatcar_production_azure_image.vhd.bz2)
- [alpha](https://alpha.release.flatcar-linux.net/amd64-usr/current/flatcar_production_azure_image.vhd.bz2)
- [edge](https://edge.release.flatcar-linux.net/amd64-usr/current/flatcar_production_azure_image.vhd.bz2)

Cette image est déjà entièrement configurée et optimisée pour s’exécuter sur Azure. Il vous suffit de la décompresser.

## <a name="building-your-own-flatcar-based-virtual-machine-for-azure"></a>Création d’une machine virtuelle Flatcar pour Azure

Vous pouvez également choisir de créer votre propre image Flatcar Container Linux.

Sur n’importe quelle machine Linux, suivez les instructions détaillées dans le [Guide du kit SDK de développement Flatcar Container Linux](https://docs.flatcar-linux.org/os/sdk-modifying-flatcar/). Lors de l’exécution du script `image_to_vm.sh`, veillez à indiquer `--format=azure` pour créer un disque dur virtuel Azure.

## <a name="next-steps"></a>Étapes suivantes

Une fois que vous disposez du fichier. vhd, vous pouvez utiliser le fichier résultant pour créer des machines virtuelles dans Azure. Si c’est la première fois que vous chargez un fichier .vhd sur Azure, consultez [Création d’une machine virtuelle Linux à partir d’un disque personnalisé](upload-vhd.md#option-1-upload-a-vhd).
