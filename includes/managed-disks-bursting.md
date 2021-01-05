---
title: Fichier include
description: Fichier include
services: virtual-machines
author: albecker1
ms.service: virtual-machines
ms.topic: include
ms.date: 04/27/2020
ms.author: albecker1
ms.custom: include file
ms.openlocfilehash: 634deb655620f99c7b7b2b55819c58308eaa5a92
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91275153"
---
Sur Azure, nous offrons la possibilité d’optimiser les performances des E/S par seconde et des Mo/s du stockage sur disque, appelée « mode rafale » aussi bien sur les machines virtuelles que sur les disques. Le bursting de disque est utile dans de nombreux scénarios, comme la gestion du trafic de disque inattendu ou les tâches de traitement par lots. Vous pouvez exploiter efficacement le mode rafale au niveau des machines virtuelles et au niveau des disques. Cela vous permet d’obtenir des performances de base de référence et de mode rafale exceptionnelles à la fois sur votre machine virtuelle et sur votre disque. 

Notez que le mode rafale sur les disques et les machines virtuelles sont indépendants l’un de l’autre. Si vous disposez d’un disque en mode rafale, vous n’avez pas besoin d’une machine virtuelle en mode rafale pour permettre la rafale de votre disque. Si vous disposez d’un bursting de machine virtuelle, vous n’avez pas besoin d’un bursting de disque pour permettre la rafale de votre machine virtuelle. 
