---
title: Considérations sur les performances pour Azure NetApp Files | Microsoft Docs
description: Découvrez les performances d’Azure NetApp Files, notamment la relation de quota et la limite de débit, et la façon d’augmenter ou de diminuer de manière dynamique le quota de volume.
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/22/2020
ms.author: b-juche
ms.openlocfilehash: edfebe3d9470defbe70b3694d5574e58ca3b5938
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91325519"
---
# <a name="performance-considerations-for-azure-netapp-files"></a>Considérations sur les performances pour Azure NetApp Files

La [limite de débit](azure-netapp-files-service-levels.md) d’un volume avec Qualité de service automatique est déterminée par une combinaison du quota attribué au volume et du niveau de service sélectionné. Pour les volumes avec une Qualité de service manuelle, la limite de débit peut être définie individuellement. Lorsque vous effectuez des plans de performances concernant Azure NetApp Files, vous devez comprendre plusieurs considérations. 

## <a name="quota-and-throughput"></a>Quota et débit  

La limite de débit n’est qu’un seul déterminant des performances réalisées.  

Les considérations sur les performances de stockage typiques, dont les mélanges de lecture et d’écriture, la taille du transfert, les modèles aléatoires ou séquentiels, et plusieurs autres facteurs contribueront aux performances totales offertes.  

Le débit empirique maximal ayant été observé lors des tests est de 4 500 Mio/s.  Sur le niveau de stockage Premium, un quota de 70,31 Tio de volume avec Qualité de service automatique configure une limite de débit suffisamment élevée pour atteindre ce niveau de performances.  

Dans le cas des volumes avec Qualité de service automatique, si vous envisagez d’attribuer des quotas de volume supérieurs à 70,31 Tio, un quota supplémentaire peut être attribué à un volume pour le stockage de données supplémentaires. Toutefois, le quota ajouté n’entraîne pas une augmentation du débit réel.  

Le même plafond de débit empirique s’applique aux volumes avec une Qualité de service manuelle. Le débit maximal pouvant être attribué à un volume est de 4 500 Mio/s.

## <a name="automatic-qos-volume-quota-and-throughput"></a>Quota et débit des volumes avec Qualité de service automatique

Cette section décrit la gestion des quotas et le débit pour les volumes de type Qualité de service automatique.

### <a name="overprovisioning-the-volume-quota"></a>Sur-approvisionnement du quota de volume

Si le niveau de performance d’une charge de travail est limité par le débit, il est possible de surapprovisionner le quota de volume avec Qualité de service automatique pour déterminer un niveau de débit plus élevé et obtenir des performances accrues.  

Par exemple, si un volume avec Qualité de service automatique dans le niveau de stockage Premium a seulement 500 Gio de données, mais requiert 128 Mio/s de débit, vous pouvez définir le quota à 2 Tio afin que le niveau de débit soit défini en conséquence (64 Mio/s par To * 2 Tio = 128 Mio/s).  

Si vous surappprovisionnez systématiquement un volume pour atteindre un débit plus élevé, envisagez plutôt d’utiliser des volumes avec Qualité de service manuelle ou un niveau de service supérieur.  Dans l’exemple ci-dessus, vous pouvez obtenir la même limite de débit avec moitié moins de quota de volume avec Qualité de service automatique en utilisant le niveau de stockage Ultra à la place (128 Mio/s par Tio * 1 Tio = 128 Mio/s).

### <a name="dynamically-increasing-or-decreasing-volume-quota"></a>Augmentation ou diminution dynamique du volume de quota

Si vos exigences de performances sont temporaires par nature, ou si vous avez augmenté les besoins de performances pour une période fixe, vous pouvez dynamiquement augmenter ou diminuer le quota de volume pour ajuster instantanément la limite de débit.  Tenez compte des points suivants : 

* Le quota de volume peut être augmenté ou diminué sans avoir à suspendre l’E/S, et l’accès au volume n’est pas interrompu ni affecté.  

    Vous pouvez ajuster le quota pendant une transaction active d’E/S par rapport à un volume.  Remarque : ce quota de volume ne peut jamais être diminué sous la quantité de données logiques stockées dans le volume.

* Lorsque le quota de volume est modifié, la modification correspondante de limite de débit est presque instantanée. 

    La modification n’interrompt ni n’affecte l’accès au volume ou l’E/S.  

* L’ajustement du quota de volume peut nécessiter une modification de la taille du pool de capacité.  

    La taille du pool de capacité peut être ajustée dynamiquement et sans affecter la disponibilité des volumes ou des E/S.

## <a name="manual-qos-volume-quota-and-throughput"></a>Quota et débit des volumes avec Qualité de service manuelle 

Si vous utilisez des volumes avec Qualité de service manuelle, vous n’êtes pas obligé de surapprovisionner le quota de volume pour obtenir un débit plus élevé, car le débit peut être attribué indépendamment à chaque volume. Toutefois, vous devez toujours vous assurer que le pool de capacité est préconfiguré avec un débit suffisant pour vos besoins en matière de performances. Le débit d’un pool de capacité est configuré en fonction de sa taille et de son niveau de service. Pour en savoir plus, consultez [Niveaux de service pour Azure NetApp Files](azure-netapp-files-service-levels.md).


## <a name="next-steps"></a>Étapes suivantes

- [Niveaux de service pour Azure NetApp Files](azure-netapp-files-service-levels.md)
- [Test d’évaluation des performances pour Linux](performance-benchmarks-linux.md)