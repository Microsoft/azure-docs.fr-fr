---
title: Gérer les conteneurs de volumes pour les appareils StorSimple de la série 8000
description: Explique comment utiliser la page des conteneurs de volumes du service StorSimple Device Manager pour ajouter, modifier ou supprimer un conteneur de volumes.
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
ms.date: 07/19/2017
ms.author: alkohli
ms.openlocfilehash: 5a6cd904d7baa71c8b383022523909ca30000ce8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85515006"
---
# <a name="use-the-storsimple-device-manager-service-to-manage-storsimple-volume-containers"></a>Utiliser le service StorSimple Device Manager pour gérer les conteneurs de volumes StorSimple

## <a name="overview"></a>Vue d’ensemble
Ce didacticiel explique comment utiliser le service StorSimple Device Manager pour créer et gérer des conteneurs de volumes StorSimple.

Un conteneur de volumes créé dans un appareil Microsoft Azure StorSimple contient un ou plusieurs volumes hébergeant les paramètres de consommation du compte de stockage, du chiffrement et de la bande passante. Un appareil peut comporter plusieurs conteneurs de volumes pour l’ensemble de ses volumes. 

Un conteneur de volumes présente les attributs suivants :

* **Volumes** : les volumes StorSimple à plusieurs niveaux ou épinglés en local et hébergés dans le conteneur de volumes. 
* **Chiffrement** : clé de chiffrement pouvant être définie pour chaque conteneur de volumes. Cette clé est utilisée pour chiffrer les données envoyées depuis votre appareil StorSimple sur le cloud. Une clé AES 256 bits de niveau militaire est utilisée avec la clé saisie par l’utilisateur. Pour sécuriser vos données, nous vous recommandons de toujours activer le chiffrement du stockage cloud.
* **Compte de stockage** : compte de stockage Azure utilisé pour stocker les données. L’ensemble des volumes hébergés dans votre conteneur de volumes partagent ce compte de stockage. Vous pouvez choisir un compte de stockage à partir d’une liste existante ou créer un compte lors de la création du conteneur de volumes, avant de spécifier les informations d’identification d’accès associées à ce compte.
* **Bande passante du cloud** : consommation de bande passante de l’appareil lors de l’envoi de ses données vers le cloud. Vous pouvez appliquer un contrôle de bande passante en définissant une valeur comprise entre 1 et 1 000 Mbits/s lorsque vous créez ce conteneur. Si vous souhaitez que l’appareil consomme toute la bande passante disponible, définissez la valeur **Illimité** dans ce champ. Vous pouvez également créer et appliquer un modèle de bande passante, afin d’allouer la bande passante suivant une planification.

Les procédures suivantes expliquent comment utiliser le panneau **Conteneurs de volumes** StorSimple pour effectuer les opérations courantes ci-dessous :

* Ajouter un conteneur de volumes
* Modifier un conteneur de volumes
* Supprimer un conteneur de volumes

## <a name="add-a-volume-container"></a>Ajouter un conteneur de volumes
Pour ajouter un conteneur de volumes, procédez comme suit.

[!INCLUDE [storsimple-8000-add-volume-container](../../includes/storsimple-8000-create-volume-container.md)]

## <a name="modify-a-volume-container"></a>Modifier un conteneur de volumes
Pour modifier un conteneur de volumes, procédez comme suit.

[!INCLUDE [storsimple-8000-modify-volume-container](../../includes/storsimple-8000-modify-volume-container.md)]

## <a name="delete-a-volume-container"></a>Supprimer un conteneur de volumes
Un conteneur de volumes comporte plusieurs volumes. Il peut être supprimé uniquement après la suppression de l’ensemble des volumes qu’il héberge. Pour supprimer un conteneur de volumes, procédez comme suit.

[!INCLUDE [storsimple-8000-delete-volume-container](../../includes/storsimple-8000-delete-volume-container.md)]

## <a name="next-steps"></a>Étapes suivantes
* En savoir plus sur la [gestion des volumes StorSimple](storsimple-8000-manage-volumes-u2.md). 
* En savoir plus sur l’[utilisation du service StorSimple Device Manager pour gérer votre appareil StorSimple](storsimple-8000-manager-service-administration.md).

