---
title: Modèle de coût pour Azure NetApp Files | Microsoft Docs
description: Décrit le modèle de coût d’Azure NetApp Files pour faciliter la gestion des dépenses liées à ce service.
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
ms.openlocfilehash: 9c4eebae6909c9ef0969bc85bcb9a985db2a7c02
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91325604"
---
# <a name="cost-model-for-azure-netapp-files"></a>Modèle de coût pour Azure NetApp Files 

Une bonne compréhension du modèle de coût d’Azure NetApp Files vous aidera à gérer vos dépenses liées à ce service. 

Pour connaître le modèle de coût spécifique à la réplication interrégion, consultez la section [Modèle de coût pour la réplication interrégion](cross-region-replication-introduction.md#cost-model-for-cross-region-replication).

## <a name="calculation-of-capacity-consumption"></a>Calcul de la consommation de capacité

Azure NetApp Files est facturé en fonction de la capacité de stockage approvisionnée.  La capacité approvisionnée est allouée par la création de pools de capacité.  Les pools de capacité sont facturés en €/Gio approvisionnés/mois par incréments horaires. La taille minimale d’un pool de capacité est de 4 Tio, et les pools de capacité peuvent ensuite être étendus par incréments de 1 Tio. Des volumes sont créés dans les pools de capacité.  Chaque volume reçoit un quota qui se décrémente de la capacité approvisionnée pour les pools. Le quota pouvant être attribué aux volumes est compris entre 100 Gio et 100 Tio.  

Dans le cas d’un volume actif, la consommation de capacité par rapport au quota repose sur la capacité (effective) logique.

Si la consommation de capacité réelle d’un volume dépasse son quota de stockage, le volume peut continuer à augmenter. Les écritures restent autorisées tant que la taille réelle du volume est inférieure à la limite système (100 Tio).  

La capacité totale utilisée dans un pool de capacité par rapport à la capacité approvisionnée pour ce pool correspond soit à la somme du quota attribué, soit à la somme de la consommation réelle (selon la plus grande de ces valeurs) de tous les volumes au sein du pool : 

   ![Calcul de la capacité totale utilisée](../media/azure-netapp-files/azure-netapp-files-total-used-capacity.png)

Le diagramme ci-dessous illustre ces concepts.  
* Nous disposons d’un pool de capacité avec 4 Tio de capacité approvisionnée.  Ce pool comporte trois volumes.  
    * Le volume 1 a reçu un quota de 2 Tio et présente une consommation de 800 Gio.  
    * Le volume 2 a reçu un quota de 1 Tio et présente une consommation de 100 Gio.  
    * Le volume 3 a reçu un quota de 500 Gio et présente une consommation de 800 Gio (dépassement).  
* Le pool de capacité est limité à 4 Tio de capacité (quantité approvisionnée).  
    3,8 Tio de capacité sont consommés (2 Tio et 1 Tio de quota pour les volumes 1 et 2, et 800 Gio de consommation réelle pour le volume 3). Il reste donc 200 Gio de capacité.

   ![Pool de capacité avec trois volumes](../media/azure-netapp-files/azure-netapp-files-capacity-pool-with-three-vols.png)

## <a name="overage-in-capacity-consumption"></a>Dépassement de la consommation de capacité  

Lorsque la capacité totale utilisée d’un pool dépasse la capacité approvisionnée pour ce pool, les écritures de données restent autorisées.  Après la période de grâce (d’une heure), si la capacité utilisée du pool dépasse encore la capacité approvisionnée pour ce pool, la taille du pool est automatiquement augmentée par incréments de 1 Tio jusqu’à ce que la capacité approvisionnée soit supérieure à la capacité totale utilisée.  Par exemple, dans l’illustration ci-dessus, si le volume 3 continue de croître et que la consommation réelle atteint 1,2 Tio, après la période de grâce, le pool est automatiquement redimensionné à 5 Tio.  La capacité approvisionnée pour le pool (5 Tio) dépasse donc la capacité utilisée (4,2 Tio).  

Bien que la taille du pool de capacité augmente automatiquement pour répondre à la demande du volume, elle n’est pas automatiquement réduite quand la taille du volume diminue. Si vous souhaitez réduire la taille du pool de capacité après une diminution de la taille du volume (par exemple, après le nettoyage des données d’un volume), vous devez _manuellement_ réduire la taille du pool de capacité.

## <a name="manual-changes-of-the-pool-size"></a>Modifications manuelles de la taille d’un pool  

Vous pouvez augmenter ou diminuer manuellement la taille d’un pool. Toutefois, les contraintes suivantes s’appliquent :
* Limites minimales et maximales du service  
    Consultez l’article relatif aux [limites de ressources](azure-netapp-files-resource-limits.md).
* Incrément de 1 Tio après l’achat initial de 4 Tio minimum
* Incrément de facturation d’une heure minimum
* Impossibilité d’abaisser la taille de pool approvisionnée à une valeur inférieure à la capacité totale utilisée dans le pool
* Pour les pools de capacité avec Qualité de service manuelle, la taille du pool ne peut être réduite que si la taille et le niveau de service fournissent plus de débit que le débit réellement attribué à tous les volumes.

## <a name="behavior-of-maximum-size-pool-overage"></a>Comportement en cas de dépassement de la taille maximale d’un pool   

La taille maximale d’un pool de capacité que vous créez ou redimensionnez est de 500 Tio.  Lorsque la capacité totale utilisée dans un pool de capacité dépasse 500 Tio, les situations suivantes se produisent :
* Les écritures de données restent autorisées (tant que le volume est inférieur à la limite système maximale de 100 Tio).
* Après la période de grâce d’une heure, le pool est automatiquement redimensionné par incréments de 1 Tio, jusqu’à ce que la capacité approvisionnée pour le pool dépasse la capacité totale utilisée.
* La capacité supplémentaire approvisionnée et facturée pour le pool supérieure à 500 Tio ne peut pas être utilisée pour attribuer un quota de volume. Elle n’est pas non plus utilisable pour l’extension des limites de qualité de service du niveau de performance.  
    Pour plus d’informations sur les limites du niveau de performance et le dimensionnement de la qualité de service, consultez l’article [Niveaux de service](azure-netapp-files-service-levels.md).

Le diagramme ci-dessous illustre ces concepts :
* Nous disposons d’un pool de capacité avec un niveau de stockage Premium et une capacité de 500 Tio. Ce pool comporte neuf volumes.
    * Les volumes 1 à 8 ont reçu un quota de 60 Tio chacun.  La capacité totale utilisée est de 480 Tio.  
        Chaque volume présente une limite de qualité de service de 3,75 Gio/s de débit (60 Tio * 64 Mio/s).  
    * Le volume 9 a reçu un quota de 20 Tio.  
        Le volume 9 présente une limite de qualité de service de 1,25 Gio/s de débit (20 Tio * 64 Mio/s).
* Le volume 9 correspond à un scénario de dépassement. Sa consommation réelle est de 25 Tio.  
    * Après la période de grâce d’une heure, le pool de capacité doit être redimensionné à 505 Tio.  
        Autrement dit, la capacité totale utilisée est de 8 * le quota de 60 Tio pour les volumes 1 à 8, et 25 Tio de consommation réelle pour le volume 9.
    * La capacité facturée est de 505 Tio.
    * Le quota du volume 9 ne peut être augmenté (car le quota total attribué pour le pool ne peut pas dépasser 500 Tio).
    * Aucun débit de qualité de service supplémentaire ne peut être attribué (car la qualité de service totale pour le pool reste basée sur 500 Tio).

   ![Pool de capacité avec neuf volumes](../media/azure-netapp-files/azure-netapp-files-capacity-pool-with-nine-vols.png)

## <a name="capacity-consumption-of-snapshots"></a>Consommation de capacité des captures instantanées 

La consommation de capacité des captures instantanées dans Azure NetApp Files est facturée au niveau du quota du volume parent.  Par conséquent, elle applique le même taux de facturation que le pool de capacité auquel appartient le volume.  Toutefois, contrairement au volume actif, la consommation des captures instantanées est mesurée sur la base de la capacité incrémentielle consommée.  Les captures instantanées Azure NetApp Files sont différentielles par nature. Selon le taux de modification des données, les captures instantanées consomment souvent une capacité sensiblement inférieure à la capacité logique du volume actif. Par exemple, supposons que vous disposiez d’une capture instantanée d’un volume de 500 Gio contenant uniquement 10 Gio de données différentielles. Dans ce cas, la capacité facturée au niveau du quota de volume pour cette capture instantanée serait de 10 Gio, et non de 500 Gio. 

## <a name="next-steps"></a>Étapes suivantes

* [Tarification Azure NetApp Files](https://azure.microsoft.com/pricing/details/storage/netapp/)
* [Niveaux de service pour Azure NetApp Files](azure-netapp-files-service-levels.md)
* [Limites des ressources pour Azure NetApp Files](azure-netapp-files-resource-limits.md)
* [Modèle de coût pour la réplication inter-région](cross-region-replication-introduction.md#cost-model-for-cross-region-replication)
