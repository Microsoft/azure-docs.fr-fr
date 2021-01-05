---
title: Limites système de la gamme StorSimple 8000 | Microsoft Docs
description: Décrit les limites du système et les tailles recommandées pour les connexions et les composants StorSimple série 8000.
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: c7392678-0924-46c6-9c59-1665cb9b6586
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 03/28/2017
ms.author: alkohli
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 70f2d9542082ddf7ecf1d1e7361b0ecdb14c5ef8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "68963385"
---
# <a name="what-are-storsimple-8000-series-system-limits"></a>Quelles sont les limites système de l’appareil StorSimple série 8000 ?

[!INCLUDE [storsimple-8000-eol-banner](../../includes/storsimple-8000-eol-banner.md)]

## <a name="overview"></a>Vue d’ensemble

StorSimple fournit un stockage évolutif et flexible pour votre centre de données. Toutefois, il existe certaines limites dont vous devez tenir compte lorsque vous planifiez, déployez et exploitez votre solution StorSimple. Le tableau suivant décrit ces limites et fournit quelques recommandations afin que vous puissiez tirer parti de votre solution StorSimple.

| Identificateur de la limite | Limite | Commentaires |
| --- | --- | --- |
| Nombre maximal d'informations d'identification du compte de stockage |64 | |
| Nombre maximal de conteneurs de volumes |64 | |
| Nombre maximal de volumes |255 | |
| Nombre maximal de volumes épinglés localement |32 | |
| Nombre maximal de planifications par modèle de bande passante |168 |Une planification par heure, tous les jours de la semaine (24*7). |
| Taille maximale d'un volume à plusieurs niveaux sur les appareils physiques |64 To pour 8100 et 8600 |8100 et 8600 sont des appareils physiques. |
| Taille maximale d'un volume à plusieurs niveaux sur les appareils virtuels dans Azure |30 To pour 8010 <br></br> 64 To pour 8020 |8010 et 8020 sont des appareils virtuels dans Azure qui utilisent le stockage Standard et Premium respectivement. |
| Taille maximale d'un volume épinglé localement sur les appareils physiques |8,5 To pour 8100 <br></br> 22,5 To pour 8600 |8100 et 8600 sont des appareils physiques. |
| Nombre maximal de connexions iSCSI |512 | |
| Nombre maximal de connexions iSCSI à partir d'initiateurs |512 | |
| Nombre maximal d'enregistrements de contrôle d'accès par appareil |64 | |
| Nombre maximal de volumes par stratégie de sauvegarde |20 | |
| Nombre maximal de sauvegardes conservées par planification (dans une stratégie de sauvegarde) |64 | |
| Nombre maximal de planifications par stratégie de sauvegarde |10 | |
| Nombre maximal d'instantanés de n'importe quel type pouvant être conservés par volume |256 |Ce nombre inclut les instantanés locaux et les instantanés cloud. |
| Nombre maximal d'instantanés pouvant être présents dans n'importe quel appareil |10 000 | |
| Nombre maximal de volumes pouvant être traités en parallèle pour sauvegarder, restaurer ou cloner |16 |<ul><li>S’il existe plus de 16 volumes, ils seront traités séquentiellement à mesure que des emplacements se libèrent.</li><li>Il n’est pas possible de générer de nouvelles sauvegardes d’un volume hiérarchisé cloné ou restauré tant que l’opération n’est pas terminée. Toutefois, pour un volume local, les sauvegardes sont autorisées une fois que le volume est en ligne.</li></ul> |
| Temps de récupération d'un clone et d'une restauration pour les volumes à plusieurs niveaux |< 2 minutes |<ul><li>Le volume sera disponible dans les 2 minutes suivant la restauration ou le clonage, quelle que soit la taille du volume.</li><li>Les performances du volume peuvent initialement être inférieures à la normale, car la plupart des données et des métadonnées se trouvent toujours dans le cloud. Les performances vont augmenter au fur et à mesure que les données provenant du cloud se dirigent vers l’appareil StorSimple.</li><li>La durée totale de téléchargement des métadonnées dépend de la taille du volume alloué. Les métadonnées sont automatiquement intégrées à l’appareil en arrière-plan, à une vitesse de 5 minutes par To de données du volume alloué. Cette vitesse peut être affectée par la bande passante Internet vers le cloud.</li><li>La restauration ou le clonage est terminé lorsque toutes les métadonnées se trouvent sur l’appareil.</li><li>Impossible d’effectuer des opérations de sauvegarde tant que la restauration ou le clonage n’est pas totalement terminé(e). |
| Temps de récupération d'une restauration pour les volumes épinglés localement |< 2 minutes |<ul><li>Le volume sera disponible dans les 2 minutes suivant la restauration, quelle que soit la taille du volume.</li><li>Les performances du volume peuvent initialement être inférieures à la normale, car la plupart des données et des métadonnées se trouvent toujours dans le cloud. Les performances vont augmenter au fur et à mesure que les données provenant du cloud se dirigent vers l’appareil StorSimple.</li><li>La durée totale de téléchargement des métadonnées dépend de la taille du volume alloué. Les métadonnées sont automatiquement intégrées à l’appareil en arrière-plan, à une vitesse de 5 minutes par To de données du volume alloué. Cette vitesse peut être affectée par la bande passante Internet vers le cloud.</li><li>Contrairement aux volumes hiérarchisés, dans le cas des volumes épinglés localement, les données du volume sont également téléchargées localement sur l’appareil. L’opération de restauration est terminée lorsque toutes les données du volume ont été transférées vers l’appareil.</li><li>Les opérations de restauration peuvent être longues. La durée totale de restauration dépend de la taille du volume local configuré, de votre bande passante Internet et des données existantes sur l’appareil. Les opérations de sauvegarde sur le volume épinglé localement sont autorisées pendant que l'opération de restauration est en cours. |
| Vitesse de traitement des instantanés cloud |15 minutes/To |<ul><li>Durée minimale nécessaire pour que l’instantané cloud soit prêt pour le chargement (par téraoctet de données d’un volume alloué dans une sauvegarde). </li><li> La durée totale d’un instantané cloud est calculée en ajoutant cette durée à la durée de chargement de l’instantané, qui est affectée par la bande passante Internet vers le cloud. |
| Débit maximal de lecture/d’écriture du client (si traité à partir du niveau SSD)* |920/720 Mbit/s avec une seule interface réseau 10GbE |Jusqu'à 2x avec MPIO et deux interfaces réseau. |
| Débit maximal de lecture/d’écriture du client (si traité à partir du niveau HDD)* |120/250 Mo/s | |
| Débit maximal de lecture/d’écriture du client (si traité à partir du niveau cloud)* pour Update 3 et versions ultérieures** |40/60 Mo/s pour les volumes hiérarchisés<br><br>60/80 Mo/s pour les volumes hiérarchisés avec l’option d’archivage sélectionnée lors de la création du volume |Le débit de lecture dépend de si les clients génèrent et conservent une profondeur de file d’attente d’E/S suffisante. <br><br>La vitesse atteinte dépend de la vitesse du compte de stockage sous-jacent utilisé. |

&#42; Le débit maximal par type d’E/S a été mesuré avec des scénarios de lecture et d’écriture de 100 pour cent. Le débit réel peut être inférieur et varie en fonction du mélange d’E/S et des conditions réseau.

&#42;&#42; Les données de performances avant la version Update 3 peuvent être inférieures.

## <a name="next-steps"></a>Étapes suivantes
Relisez les [Conditions requises pour le système StorSimple](storsimple-8000-system-requirements.md).

