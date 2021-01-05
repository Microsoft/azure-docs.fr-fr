---
title: Afficher l’état d’intégrité de la relation de réplication Azure NetApp Files | Microsoft Docs
description: Décrit comment afficher l’état de la réplication sur le volume source ou le volume de destination d’Azure NetApp Files.
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
ms.topic: how-to
ms.date: 09/16/2020
ms.author: b-juche
ms.openlocfilehash: a14656bc1abc42fdc3e948fdae1ca07469681d21
ms.sourcegitcommit: 30906a33111621bc7b9b245a9a2ab2e33310f33f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/22/2020
ms.locfileid: "95249850"
---
# <a name="display-health-status-of-replication-relationship"></a>Afficher l’état d’intégrité de la relation de réplication 

Vous pouvez afficher l’état de la réplication sur le volume source ou le volume de destination.  

## <a name="display-replication-status"></a>Afficher l’état de la réplication

1. À partir du volume source ou du volume de destination, cliquez sur **Réplication** sous Service de stockage pour l’un des volumes.

    Les informations d’état et d’intégrité de la réplication suivantes s’affichent :  
    * **Type de point de terminaison** : indique si le volume est la source ou la destination de la réplication.
    * **Intégrité** : affiche l’état d’intégrité de la relation de réplication.
    * **État du miroir** : affiche l’une des valeurs suivantes :
        * *Non initialisé* :  
            il s’agit de l’état initial et par défaut lors de la création d’une relation de peering. L’état reste non initialisé jusqu’à ce que l’initialisation se termine correctement.
        * *En miroir* :   
            le volume de destination a été initialisé et est prêt à recevoir les mises à jour de mise en miroir.
        * *Arrêté* :   
            il s’agit de l’état après que vous avez arrêté la relation de peering. Le volume de destination est `‘RW’` et des instantanés sont présents.
    * **État de la relation** : affiche l’une des valeurs suivantes : 
        * *Inactif* :  
            aucune opération de transfert n’est en cours et les transferts ultérieurs ne sont pas désactivés.
        * *Transfert en cours* :  
            une opération de transfert est en cours et les transferts ultérieurs ne sont pas désactivés.
    * **Planification de réplication** : indique la fréquence à laquelle les mises à jour de mise en miroir incrémentielles sont effectuées lorsque l’initialisation (copie de base de référence) est terminée.

    * **Progression totale** : affiche la quantité totale de données, en octets, transférées pour l’opération de transfert en cours. Cette quantité est le nombre réel de bits transférés, et peut différer de l’espace logique signalé par les volumes source et de destination.  

    ![État d’intégrité de la réplication](../media/azure-netapp-files/cross-region-replication-health-status.png)

> [!NOTE] 
> La relation de réplication présente l’état comme étant *non sain* si les travaux de réplication précédents ne sont pas terminés. Cet état est le résultat de transferts de volumes importants durant une fenêtre de transfert plus petite (par exemple un temps de transfert de dix minutes pour un volume important). Dans ce cas, l’état de la relation indique *Transfert en cours* et l’état d’intégrité affiche *non sain*.

## <a name="next-steps"></a>Étapes suivantes  

* [Réplication entre régions](cross-region-replication-introduction.md)
* [Gérer la reprise d’activité après sinistre](cross-region-replication-manage-disaster-recovery.md)
* [Métriques de réplication de volume](azure-netapp-files-metrics.md#replication)
* [Supprimer des volumes ou des réplications de volume](cross-region-replication-delete.md)
* [Résoudre les problèmes de réplication inter-région](troubleshoot-cross-region-replication.md)

