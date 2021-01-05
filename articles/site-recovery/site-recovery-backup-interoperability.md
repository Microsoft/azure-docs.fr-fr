---
title: Prise en charge de l’utilisation d’Azure Site Recovery avec la sauvegarde Azure
description: Fournit une vue d’ensemble de la façon dont Azure Site Recovery et Sauvegarde Azure peuvent être utilisés conjointement.
author: sideeksh
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 10/15/2019
ms.author: sideeksh
ms.openlocfilehash: c334eee34eb878135d3d81ab15d03618c6604846
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86135174"
---
# <a name="support-for-using-site-recovery-with-azure-backup"></a>Prise en charge de l’utilisation de Site Recovery avec Sauvegarde Azure

Cet article résume la prise en charge de l’utilisation conjointe du [service Site Recovery](site-recovery-overview.md) avec le [service Sauvegarde Azure](../backup/backup-overview.md).

**Action** | **Prise en charge de Site Recovery** | **Détails**
--- | --- | ---
**Déployer les services ensemble** | Prise en charge | Les services sont interopérables et peuvent être configurés conjointement.
**Sauvegarde/restauration de fichier** | Prise en charge | Lorsque les fonctions de sauvegarde et de réplication sont activées pour une machine virtuelle, et que les sauvegardes sont effectuées, il n’existe aucun problème de restauration de fichiers sur les machines virtuelles ou le groupe de machines virtuelles côté source. La réplication se poursuit comme d’habitude, sans que l’intégrité de la réplication s’en trouve modifiée.
**Restauration de disque** | Aucune prise en charge actuelle | Si vous restaurez un disque sauvegardé, vous devez à nouveau désactiver et réactiver la réplication pour la machine virtuelle.
**Restauration de machines virtuelles** | Aucune prise en charge actuelle | Si vous restaurez une machine virtuelle ou un groupe de machines virtuelles, vous devez désactiver et réactiver la réplication pour ces machines virtuelles.  


