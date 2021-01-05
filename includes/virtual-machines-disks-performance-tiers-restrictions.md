---
title: Fichier Include
description: Fichier Include
services: virtual-machines
author: roygara
ms.service: virtual-machines
ms.topic: include
ms.date: 11/20/2020
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: 241a2be834d7828cdb56f59313c0c1a9c478ff77
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96016515"
---
- Cette fonctionnalité est actuellement prise en charge uniquement pour les disques SSD Premium.
- Vous devez libérer votre machine virtuelle ou détacher votre disque d'une machine virtuelle en cours d'exécution avant de pouvoir modifier le niveau du disque.
- Les niveaux de performance P60, P70 et P80 peuvent uniquement être utilisés par des disques d’une taille supérieure à 4 096 Gio.
- Le niveau de performance d'un disque ne peut être rétrogradé qu'une fois toutes les 12 heures.