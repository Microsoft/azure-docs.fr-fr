---
title: Fichier include
description: Fichier include
services: virtual-machines
author: roygara
ms.service: virtual-machines
ms.topic: include
ms.date: 07/10/2020
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: 7700b1118c4f04643607f44a474338ffdd5c09e0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86230980"
---
- Ne prend pas en charge les disques Ultra.
- Ne peut pas être activé si Azure Disk Encryption (le chiffrement de machine virtuelle invité à l’aide de BitLocker/VM-decryption) est activé sur vos machines virtuelles ou groupes de machines virtuelles identiques.
- Azure Disk Encryption ne peut pas être activé sur les disques sur lesquels le chiffrement est activé sur l’hôte.
- Le chiffrement peut être activé sur un groupe de machines virtuelles identiques existant. Toutefois, seules les nouvelles machines virtuelles créées après l’activation du chiffrement sont automatiquement chiffrées.
- Les machines virtuelles existantes doivent être libérées et réallouées afin d’être chiffrées.