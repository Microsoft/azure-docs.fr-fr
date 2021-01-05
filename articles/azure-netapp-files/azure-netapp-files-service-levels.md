---
title: Niveaux de service pour Azure NetApp Files | Microsoft Docs
description: Décrit les performances de débit des niveaux de service d'Azure NetApp Files.
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
ms.date: 10/12/2020
ms.author: b-juche
ms.openlocfilehash: 7eac6a40476cffe875a03de49c9c9311ffbf4d39
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96017064"
---
# <a name="service-levels-for-azure-netapp-files"></a>Niveaux de service pour Azure NetApp Files
Les niveaux de service sont un attribut d’un pool de capacités. Les niveaux de service sont définis et différenciés par le débit maximum autorisé pour un volume dans le pool de capacités en fonction du quota attribué au volume.

## <a name="supported-service-levels"></a>Niveaux de service pris en charge

Azure NetApp Files prend en charge trois niveaux de service : *Ultra*, *Premium* et *Standard*. 

* <a name="Ultra"></a>Stockage Ultra

    Le niveau de stockage Ultra fournit un débit allant jusqu’à 128 Mio/s pour 1 Tio de capacité provisionnée. 

* <a name="Premium"></a>Stockage Premium

    Le niveau de stockage Premium fournit un débit allant jusqu’à 64 Mio/s pour 1 Tio de capacité provisionnée. 

* <a name="Standard"></a>Stockage Standard

    Le niveau de stockage Standard fournit un débit allant jusqu’à 16 Mio/s pour 1 Tio de capacité provisionnée.

## <a name="throughput-limits"></a>Limites de débit

La limite de débit pour un volume est déterminée par la combinaison des facteurs suivants :
* Le niveau de service du pool de capacités auquel le volume appartient
* Le quota attribué au volume  
* Le type de QoS (*automatique* ou *manuel*) du pool de capacité  

### <a name="throughput-limit-examples-of-volumes-in-an-auto-qos-capacity-pool"></a>Exemples de limite de débit de volumes dans un pool de capacités QoS automatique

Le diagramme suivant montre des exemples de limite de débit de volumes dans un pool de capacités QoS automatique :

![Illustration de niveau de service](../media/azure-netapp-files/azure-netapp-files-service-levels.png)

* Dans l’exemple 1, un volume d’un pool de capacités QoS automatique avec le niveau de stockage Premium et un quota de 2 Tio se voit attribuer une limite de débit de 128 Mio/s (2 Tio * 64 Mio/s). Ce scénario s’applique quelle que soit la taille de pool de capacités ou la consommation de volume réelle.

* Dans l’exemple 2, un volume d’un pool de capacités QoS automatique avec le niveau de stockage Premium et un quota de 100 Gio se voit attribuer une limite de débit de 6,25 Mio/s (0,09765625 Tio * 64 Mio/s). Ce scénario s’applique quelle que soit la taille de pool de capacités ou la consommation de volume réelle.

### <a name="throughput-limit-examples-of-volumes-in-a-manual-qos-capacity-pool"></a>Exemples de limite de débit de volumes dans un pool de capacités QoS manuel 

Si vous utilisez un pool de capacités QoS manuel, vous pouvez affecter la capacité et le débit d’un volume de manière indépendante. Lorsque vous créez un volume dans un pool de capacités QoS manuel, vous pouvez spécifier la valeur de débit (en Mio/S). Le débit total affecté aux volumes dans un pool de capacités QoS manuel dépend de la taille du pool et du niveau de service. La limite est : (taille du pool de capacité en Tio x Débit de niveau de service/Tio). Par exemple, un pool de capacités de 10 Tio avec le niveau de service Ultra a une capacité de débit totale de 1 280 Mio/s (10 Tio x 128 Mio/s/Tio) disponible pour les volumes.

Par exemple, pour un système SAP HANA, ce pool de capacités peut être utilisé pour créer les volumes suivants. Chaque volume fournit la taille individuelle et le débit pour répondre aux besoins de votre application :

* Volume de données SAP HANA : Taille de 4 Tio avec un débit allant jusqu’à 704 Mio/s
* Volume de fichier journal SAP HANA : Taille de 0,5 Tio avec un débit allant jusqu’à 256 Mio/s
* Volume partagé SAP HANA : Taille de 1 Tio avec un débit allant jusqu’à 64 Mio/s
* Volume de sauvegarde SAP HANA : Taille de 4,5 Tio avec un débit allant jusqu’à 256 Mio/s

Le diagramme suivant illustre les scénarios pour les volumes de SAP HANA :

![Scénarios de volume SAP HANA QoS](../media/azure-netapp-files/qos-sap-hana-volume-scenarios.png) 

## <a name="next-steps"></a>Étapes suivantes

- [Tarification Azure NetApp Files](https://azure.microsoft.com/pricing/details/storage/netapp/)
- [Modèle de coût pour Azure NetApp Files](azure-netapp-files-cost-model.md) 
- [Créer un pool de capacités](azure-netapp-files-set-up-capacity-pool.md)
- [Contrat de niveau de service (SLA) pour Azure NetApp Files](https://azure.microsoft.com/support/legal/sla/netapp/)
- [Changer dynamiquement le niveau de service d’un volume](dynamic-change-volume-service-level.md) 
- [Objectifs de niveau de service pour la réplication inter-régions](cross-region-replication-introduction.md#service-level-objectives)
