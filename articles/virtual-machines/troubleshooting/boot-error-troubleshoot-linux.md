---
title: Résoudre des erreurs de démarrage sur des machines virtuelles Linux Azure | Microsoft Docs
description: Cet article vous redirige vers les articles permettant de résoudre des erreurs de démarrage sur des machines virtuelles Linux Azure.
services: virtual-machines-linux
documentationCenter: ''
author: vilibert
manager: spogge
editor: ''
ms.service: virtual-machines-linux
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 11/20/2019
ms.author: vilibert
ms.openlocfilehash: 37cb201751f72918838efe5837aa0e357d483f24
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "74408559"
---
# <a name="troubleshoot-azure-linux-virtual-machines-boot-errors"></a>Résoudre des erreurs de démarrage sur des machines virtuelles Linux Azure

Cet article liste les erreurs de démarrage courantes que vous pouvez rencontrer en démarrant une machine virtuelle Linux dans Microsoft Azure. Pour plus d’informations sur les erreurs, consultez les articles de la section **Erreurs de démarrage et solutions**.

## <a name="boot-errors-and-solutions"></a>Erreurs de démarrage et solutions

* [GRUB rescue](troubleshoot-vm-boot-error.md)

## <a name="next-steps"></a>Étapes suivantes

- [Console série de machine virtuelle](serial-console-linux.md)

Résolvez les problèmes d’une machine virtuelle Linux en attachant le disque du système d’exploitation à une machine virtuelle de récupération à l’aide d’Azure :

- [Réparation d’une machine virtuelle Azure](repair-linux-vm-using-azure-virtual-machine-repair-commands.md)

 Échange de disque – Il peut être automatisé avec l’un des éléments suivants :
- [Scripts de récupération Power Shell](https://github.com/Azure/azure-support-scripts/tree/master/VMRecovery/ResourceManager)
- [Scripts de récupération bash](https://github.com/sribs/azure-support-scripts)

- [INTERFACE DE LIGNE DE COMMANDE](troubleshoot-recovery-disks-linux.md)
- [Azure portal](troubleshoot-recovery-disks-portal-linux.md)


## <a name="disk-swap-video"></a>Vidéo sur l’échange de disque :

Si vous n’avez pas accès à GRUB, regardez [cette vidéo](https://youtu.be/m5t0GZ5oGAc) et découvrez comment vous pouvez facilement automatiser la procédure d’échange de disque pour récupérer votre machine virtuelle.

## <a name="unofficial-solution"></a>Solution non officielle

La récupération d’une machine virtuelle peut également être tentée avec le script BETA [ALAR](https://github.com/malachma/azure-auto-recover) non pris en charge.