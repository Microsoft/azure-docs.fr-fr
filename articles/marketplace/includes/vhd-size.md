---
title: Fichier include
description: fichier
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: include
author: emuench
ms.author: mingshen
ms.date: 10/09/2020
ms.openlocfilehash: f4e34e850391696506beed9f6f386f85528dff24
ms.sourcegitcommit: b6f3ccaadf2f7eba4254a402e954adf430a90003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92283074"
---
Si vous avez sélectionné l’une des machines virtuelles préconfigurées avec un système d’exploitation (et éventuellement d’autres services), vous avez déjà choisi une taille de machine virtuelle Azure standard. Il est recommandé de démarrer votre solution avec un système d’exploitation préconfiguré. Toutefois, si vous installez un système d’exploitation manuellement, vous devez dimensionner votre VHD principal dans votre image de machine virtuelle. Vérifiez que la taille du disque du système d’exploitation est comprise dans les limites de Linux ou Windows.

| Système d''exploitation | Taille de disque dur virtuel |
| --- | --- |
| Linux | 30 à 1023 Go |
| Windows | 30 à 250 Go |
|

Les images Windows ou SQL Server fournies à titre de base approuvée satisfont déjà les exigences requises. Ne modifiez donc pas le format ou la taille du disque dur virtuel.

Les disques de données peuvent avoir une taille maximale de 1 To. Quand vous choisissez la taille, rappelez-vous que les clients ne peuvent pas redimensionner les disques durs virtuels dans une image lors du déploiement. Créez des disques durs virtuels de disques de données sous forme de disques durs virtuels à format fixe. Ils doivent également être extensibles (fragmentés/dynamiques). À l’origine, les disques de données peuvent être vides ou contenir des données.