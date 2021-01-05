---
title: Mappage d’index de corps Azure Kinect
description: Découvrez comment interroger un mappage d’index de suivi de corps dans Azure Kinect DK.
author: qm13
ms.author: quentinm
ms.reviewer: yijwan
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: conceptual
keywords: kinect, portage, corps, suivi, index, segmentation, mappage
ms.openlocfilehash: 8c128e59cef515aafb4f59794f0f644b90fd625d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85276528"
---
# <a name="azure-kinect-body-tracking-index-map"></a>Mappage d’index de suivi de corps Azure Kinect

Le mappage d’index de corps comprend le mappage de segmentation d’instance pour chaque corps dans la capture de la caméra de profondeur. Chaque pixel est mappé au pixel correspondant dans l’image de profondeur ou IR. La valeur de chaque pixel représente le corps auquel le pixel appartient. Il peut s’agir de l’arrière-plan (valeur `K4ABT_BODY_INDEX_MAP_BACKGROUND`) ou de l’index d’un `k4abt_body_t`détecté.

![Exemple de mappage d’index de corps](./media/concepts/body-index-map.png)

>[!NOTE]
> L’index du corps est différent de l’ID du corps. Vous pouvez interroger l’ID du corps à partir d’un index de corps donné en appelant l’API [k4abt_frame_get_body_id()](https://microsoft.github.io/Azure-Kinect-Body-Tracking/release/1.x.x/group__btfunctions_ga1d612404d133a279af847974e9359a92.html#ga1d612404d133a279af847974e9359a92).


## <a name="using-body-index-map"></a>Utilisation du mappage d’index de corps

Le mappage d’index de corps est stocké en tant que `k4a_image_t` et a la même résolution que l’image de profondeur ou IR. Chaque pixel est une valeur de 8 bits. Il est possible d’interroger celle-ci à partir d’un `k4abt_frame_t` en appelant `k4abt_frame_get_body_index_map`. Il incombe au développeur de libérer de la mémoire pour le mappage d’index de corps en appelant `k4a_image_release()`.

## <a name="next-steps"></a>Étapes suivantes

[Créer votre première application de suivi de corps](build-first-body-app.md)
