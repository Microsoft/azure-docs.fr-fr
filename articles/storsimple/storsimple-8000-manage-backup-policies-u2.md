---
title: Gestion des stratégies de sauvegarde de la gamme StorSimple 8000 | Microsoft Docs
description: Explique comment utiliser le service StorSimple Manager pour créer et gérer des sauvegardes manuelles, des planifications de sauvegarde et une rétention de sauvegarde sur un appareil de la gamme StorSimple 8000.
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 07/05/2017
ms.author: alkohli
ms.openlocfilehash: 9ec2bbef88ed185c3927676ad2596269783d1850
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85515628"
---
# <a name="use-the-storsimple-device-manager-service-in-azure-portal-to-manage-backup-policies"></a>Utiliser le service StorSimple Device Manager dans le portail Azure pour gérer les stratégies de sauvegarde


## <a name="overview"></a>Vue d’ensemble

Ce didacticiel vous explique comment utiliser le panneau **Stratégies de sauvegarde** du service StorSimple Device Manager pour contrôler les processus de sauvegarde et la rétention de sauvegarde pour vos volumes StorSimple. Il décrit également comment effectuer une sauvegarde manuelle.

Lorsque vous sauvegardez un volume, vous pouvez choisir de créer un instantané local ou un instantané cloud. Si vous sauvegardez un volume épinglé localement, nous vous recommandons de spécifier un instantané cloud. Le fait de prendre un grand nombre d'instantanés locaux d’un volume épinglé localement associé à un jeu de données qui possède une attrition élevée entraîne une situation dans laquelle vous pourriez rapidement manquer d'espace local. Si vous choisissez de prendre des instantanés locaux, nous vous recommandons de prendre moins d'instantanés quotidiens pour sauvegarder l'état le plus récent, de les conserver pendant un jour et de les supprimer.

Lorsque vous prenez un instantané cloud d'un volume épinglé localement, vous copiez uniquement les données modifiées dans le cloud, où elles sont dédupliquées et compressées.

## <a name="the-backup-policy-blade"></a>Le panneau Stratégie de sauvegarde

Le panneau **Stratégies de sauvegarde** de votre appareil StorSimple vous permet de gérer les stratégies de sauvegarde et de planifier les instantanés locaux et cloud. Les stratégies de sauvegarde sont utilisées pour configurer les planifications de sauvegarde et la rétention de sauvegarde pour un ensemble de volumes. Les stratégies de sauvegarde permettent de prendre un instantané de plusieurs volumes simultanément. Ce qui signifie que les sauvegardes créées par une stratégie de sauvegarde sont des copies cohérentes de l’incident.

La liste tabulaire de stratégies de sauvegarde vous permet également de filtrer les stratégies de sauvegarde existantes à l’aide de l’un ou de plusieurs des champs suivants :

* **Nom de la stratégie** : nom associé à la stratégie. Les différents types de stratégies sont les suivants :

  * stratégies planifiées, qui sont explicitement créées par l’utilisateur ;
  * stratégies importées, qui ont été créées dans le gestionnaire d’instantanés StorSimple. Ces éléments présentent une balise décrivant l’hôte du gestionnaire d’instantanés StorSimple depuis lequel les stratégies ont été importées.

  > [!NOTE]
  > Les stratégies de sauvegarde automatiques ou par défaut ne sont plus activées au moment de la création du volume.

* **Dernière sauvegarde réussie** : la date et l’heure de la dernière sauvegarde réussie associée à cette stratégie.

* **Prochaine sauvegarde** : la date et l’heure de la prochaine sauvegarde planifiée qui sera lancée par cette stratégie.

* **Volumes** : les volumes associés à la stratégie. Tous les volumes associés à une stratégie de sauvegarde sont regroupés lors de la création des sauvegardes.

* **Planifications** : le nombre de planifications associées à la stratégie de sauvegarde.

Les opérations fréquentes pouvant être effectuées pour les stratégies de sauvegarde sont les suivantes :

* Ajout d’une stratégie de sauvegarde
* Ajouter ou modifier une planification
* Ajouter ou supprimer un volume
* Supprimer une stratégie de sauvegarde
* Exécuter une sauvegarde manuelle

## <a name="add-a-backup-policy"></a>Ajout d’une stratégie de sauvegarde

Ajoutez une stratégie de sauvegarde pour planifier automatiquement vos sauvegardes. Lorsque vous créez un volume, aucune stratégie de sauvegarde par défaut n’est associée à votre volume. Vous devez ajouter et attribuer une stratégie de sauvegarde pour protéger les données du volume.

Procédez comme suit dans le portail Azure pour ajouter une stratégie de sauvegarde dédiée à votre appareil StorSimple. Après avoir ajouté la stratégie, vous pouvez définir une planification (voir [Ajouter ou modifier une planification](#add-or-modify-a-schedule)).

[!INCLUDE [storsimple-8000-add-backup-policy-u2](../../includes/storsimple-8000-add-backup-policy-u2.md)]

## <a name="add-or-modify-a-schedule"></a>Ajouter ou modifier une planification

Vous pouvez ajouter ou modifier une planification associée à une stratégie de sauvegarde existante sur votre appareil StorSimple. Procédez comme suit dans le portail Azure pour ajouter ou modifier une planification.

[!INCLUDE [storsimple-8000-add-modify-backup-schedule](../../includes/storsimple-8000-add-modify-backup-schedule-u2.md)]


## <a name="add-or-remove-a-volume"></a>Ajouter ou supprimer un volume

Vous pouvez ajouter ou supprimer un volume attribué à une stratégie de sauvegarde sur votre appareil StorSimple. Procédez comme suit dans le portail Azure pour ajouter ou supprimer un volume.

[!INCLUDE [storsimple-8000-add-volume-backup-policy-u2](../../includes/storsimple-8000-add-remove-volume-backup-policy-u2.md)]


## <a name="delete-a-backup-policy"></a>Supprimer une stratégie de sauvegarde

Procédez comme suit dans le portail Azure pour supprimer une stratégie de sauvegarde sur votre appareil StorSimple.

[!INCLUDE [storsimple-8000-delete-backup-policy](../../includes/storsimple-8000-delete-backup-policy.md)]

## <a name="take-a-manual-backup"></a>Exécuter une sauvegarde manuelle

Procédez comme suit dans le portail Azure pour créer une sauvegarde à la demande (manuelle) pour un volume unique.

[!INCLUDE [storsimple-8000-create-manual-backup](../../includes/storsimple-8000-create-manual-backup.md)]

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur l’[utilisation du service StorSimple Device Manager pour gérer votre appareil StorSimple](storsimple-8000-manager-service-administration.md).

