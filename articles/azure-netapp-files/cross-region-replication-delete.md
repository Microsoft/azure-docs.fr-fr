---
title: Suppression de volumes ou de réplications de volumes pour la 	réplication inter-région Azure NetApp Files | Microsoft Docs
description: Décrit la procédure de suppression d’une connexion de réplication qui n’est plus nécessaire entre les volumes source et de destination.
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
ms.date: 11/18/2020
ms.author: b-juche
ms.openlocfilehash: 5ce7a591acd8203775808457219b0ec392cd696e
ms.sourcegitcommit: 30906a33111621bc7b9b245a9a2ab2e33310f33f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/22/2020
ms.locfileid: "95249889"
---
# <a name="delete-volume-replications-or-volumes"></a>Supprimer des volumes ou des réplications de volume

Cet article explique comment supprimer des réplications de volumes. Il indique également comment supprimer le volume source ou le volume de destination.

## <a name="delete-volume-replications"></a>Suppression de réplications de volumes

Pour mettre fin à la connexion de réplication entre les volumes source et de destination, supprimez la réplication du volume. Vous devez supprimer la réplication du volume de destination. L’opération de suppression ne supprime que l’autorisation pour la réplication ; elle ne supprime pas le volume source ni le volume de destination. 

1. Assurez-vous que le peering de réplication a été interrompu avant de supprimer la réplication de volume. Pour arrêter le peering de réplication : 

    1. Sélectionnez le volume de *destination*. Cliquez sur **Réplication** sous Service de stockage.  

    2.  Vérifiez les champs suivants avant de continuer :  
        * Vérifiez qu’État du miroir indique ***En miroir** _.   
            N’essayez pas d’arrêter le peering de réplication si État du miroir indique _Non initialisé*.
        * Vérifiez qu’État de la relation indique ***Inactif** _.   
            N’essayez pas d’arrêter le peering de réplication si État de la relation indique _Transfert en cours*.   

        Voir [Afficher l’état d’intégrité de la relation de réplication](cross-region-replication-display-health-status.md). 

    3.  Cliquez sur **Arrêter l’appairage**.  

    4.  Tapez **Oui** à l’invite, puis cliquez sur **Arrêter**. 

        ![Arrêter le peering de réplication](../media/azure-netapp-files/cross-region-replication-break-replication-peering.png)


1. Pour supprimer la réplication du volume, sélectionnez **Réplication** sur le volume source ou le volume de destination.  

2. Cliquez sur **Supprimer**.    

3. Pour confirmer la suppression, tapez **Oui** et cliquez sur **Supprimer**.   

    ![Supprimer la réplication](../media/azure-netapp-files/cross-region-replication-delete-replication.png)

## <a name="delete-source-or-destination-volumes"></a>Suppression de volumes source ou de destination

Si vous souhaitez supprimer le volume source ou de destination, vous devez suivre les étapes suivantes dans l’ordre indiqué. Sinon, l’erreur `Volume with replication cannot be deleted` se produit.  

1. Dans le volume de destination, [supprimez la réplication de volume](#delete-volume-replications).   

2. Supprimez le volume source ou de destination selon les besoins. Pour cela, cliquez avec le bouton droit sur le nom du volume, puis sélectionnez **Supprimer**.   

    ![Capture d’écran du menu contextuel d’un volume](../media/azure-netapp-files/cross-region-replication-delete-volume.png)

## <a name="next-steps"></a>Étapes suivantes  

* [Réplication entre régions](cross-region-replication-introduction.md)
* [Configuration requise et considérations pour la réplication entre régions](cross-region-replication-requirements-considerations.md)
* [Afficher l’état d’intégrité de la relation de réplication](cross-region-replication-display-health-status.md)
* [Résoudre les problèmes de réplication entre régions](troubleshoot-cross-region-replication.md)

