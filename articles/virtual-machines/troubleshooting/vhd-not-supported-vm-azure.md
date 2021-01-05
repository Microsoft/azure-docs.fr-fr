---
title: Le disque dur virtuel n’est pas pris en charge lorsque vous créez une machine virtuelle dans Azure | Microsoft Docs
description: Cet article permet de corriger les erreurs de disque dur virtuel lors de l’exécution d’une machine virtuelle dans Microsoft Azure.
services: virtual-machines
documentationCenter: ''
author: v-miegge
manager: dcscontentpm
editor: ''
ms.service: virtual-machines
ms.assetid: 5488aba9-c3da-435d-b4a5-63470f455b07
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: infrastructure
ms.date: 06/29/2020
ms.author: genli
ms.openlocfilehash: 23f31e8d36d6ba615b21a4043c7d39716fc6d89a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88653492"
---
# <a name="vhd-is-not-supported-when-you-create-a-virtual-machine-in-azure"></a>Le disque dur virtuel n’est pas pris en charge lorsque vous créez une machine virtuelle dans Azure.

Cet article aide à corriger les erreurs de disque dur virtuel lors de l’exécution d’une machine virtuelle dans Windows ou Linux.

## <a name="symptoms"></a>Symptômes

Lorsque vous créez une machine virtuelle dans Microsoft Azure à l’aide d’un disque dur virtuel chargé, le déploiement échoue et retourne le message d’erreur suivant : 

```
New-AzureRmVM : Long running operation failed with status 'Failed'.
ErrorCode: InvalidVhd
ErrorMessage: The specified cookie value in VHD footer indicates that disk 'diskname' with blob https://xxxxxx.blob.core.windows.net/vhds/samplename.vhd is not a supported VHD. Disk is expected to have cookie value 'conectix'.
```

## <a name="cause"></a>Cause

Ce problème se produit pour l’une des raisons suivantes :

- Le disque dur virtuel n’est pas conforme avec l’alignement de 1 Mo (décalage). La taille de disque prise en charge doit être de 1 Mo * N. Par exemple, le disque doit être de 102,401 Mo.
- Le disque dur virtuel est endommagé ou n’est pas pris en charge. 

## <a name="resolution"></a>Résolution

> [!NOTE]
> Pour appliquer le correctif suivant, le client doit effectuer ces étapes avant de charger le disque dur virtuel dans Azure.

Pour résoudre ce problème, redimensionnez le disque pour qu’il soit conforme à un alignement de 1 Mo :

- Pour résoudre le problème dans Windows, utilisez l’[applet de commande PowerShell Resize-VHD](/powershell/module/hyper-v/resize-vhd). Notez que **Resize-VHD** n’est pas une applet de commande Azure PowerShell.

  1. [Installer le rôle Hyper-V sur Windows Server](/windows-server/virtualization/hyper-v/get-started/install-the-hyper-v-role-on-windows-server)
  1. [Convertir le disque virtuel en disque dur virtuel de taille fixe](../windows/prepare-for-upload-vhd-image.md#convert-the-virtual-disk-to-a-fixed-size-vhd)

- Pour résoudre le problème dans Linux, utilisez la [commande qemu-img](../linux/create-upload-generic.md).

Pour plus d’informations sur la création et le chargement d’un disque dur virtuel pour créer une machine virtuelle, consultez les articles suivants :

- [Charger et créer une machine virtuelle Linux à partir d’une image de disque personnalisée à l’aide de la Azure CLI 1.0](../linux/upload-vhd.md)
- [Créer et charger un disque dur virtuel Windows Server dans Azure](../windows/upload-generalized-managed.md)

Des problèmes persistants peuvent indiquer un disque dur virtuel endommagé. Dans ce cas, nous vous recommandons de reconstruire le disque dur virtuel à partir de zéro.

Pour plus d’informations, consultez l’article suivant :

- [À propos du disque dur virtuel](../managed-disks-overview.md)
