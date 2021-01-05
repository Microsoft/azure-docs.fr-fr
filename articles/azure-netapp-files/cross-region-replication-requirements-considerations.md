---
title: Configuration requise et considérations relatives à l’utilisation de la réplication inter-région de volume Azure NetApp Files | Microsoft Docs
description: Décrit la configuration requise et les considérations relatives à l’utilisation de la fonctionnalité de réplication inter-région de volume d’Azure NetApp Files.
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
ms.date: 09/16/2020
ms.author: b-juche
ms.openlocfilehash: 7b664dcd1cb12808960ffacf91c6d02d58632c4e
ms.sourcegitcommit: 30906a33111621bc7b9b245a9a2ab2e33310f33f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/22/2020
ms.locfileid: "95243135"
---
# <a name="requirements-and-considerations-for-using-cross-region-replication"></a>Configuration requise et considérations pour la réplication inter-région 

Notez la configuration requise et les considérations suivantes relatives à l’[utilisation de la fonctionnalité de réplication inter-région](cross-region-replication-create-peering.md) d’Azure NetApp Files :  

## <a name="requirements-and-considerations"></a>Conditions requises et éléments à prendre en compte 

* La fonctionnalité de réplication inter-région est actuellement disponible en préversion publique. Vous devez soumettre une demande d’inscription à la liste d’attente pour accéder à la fonctionnalité via la [page de soumission de demande d’inscription à la liste d’attente de réplication inter-région Azure NetApp Files](https://aka.ms/anfcrrpreviewsignup). Attendez de recevoir un e-mail de confirmation officiel de l’équipe Azure NetApp Files avant d’utiliser la fonctionnalité de réplication inter-région.
* La réplication Azure NetApp Files n’est disponible que dans certaines paires de régions fixes. Voir [Paires de régions prises en charge](cross-region-replication-introduction.md#supported-region-pairs). 
* Les volumes SMB et les volumes NFS sont pris en charge. La réplication des volumes SMB nécessite une connexion Active Directory dans les comptes NetApp source et de destination. La connexion Active Directory de destination doit avoir accès aux serveurs DNS ou aux contrôleurs de domaine ADDS qui sont accessibles à partir du sous-réseau délégué dans la région de destination. Pour plus d’informations, consultez [Configuration requise pour les connexions Active Directory](azure-netapp-files-create-volumes-smb.md#requirements-for-active-directory-connections). 
* Le compte de destination doit se trouver dans une région différente du volume source. Vous pouvez également sélectionner un compte NetApp existant dans une autre région.  
* La réplication Azure NetApp Files ne prend actuellement pas en charge plusieurs abonnements. Toutes les réplications doivent être effectuées sous un même abonnement.
* Vous pouvez configurer un maximum de cinq volumes pour la réplication au sein d’un même abonnement par région. Vous pouvez ouvrir un ticket de support pour demander une augmentation du quota par défaut de cinq volumes de destination de réplication (par abonnement dans une région). 
* Il peut y avoir un délai maximal de cinq minutes avant que l’interface reflète un nouvel instantané ajouté sur le volume source.  
* Les topologies en cascade et de type fan-in/out ne sont pas prises en charge.
* La configuration de la réplication de volume pour les volumes sources créés à partir d’un instantané n’est pas prise en charge pour l’instant.
* Une fois que vous avez configuré la réplication inter-région, le processus de réplication crée des *instantanés SnapMirror* pour fournir des références entre le volume source et le volume de destination. Les instantanés SnapMirror sont parcourus automatiquement lorsqu’un nouveau est créé pour chaque transfert incrémentiel. Vous ne pouvez pas supprimer les instantanés SnapMirror tant que la relation de réplication et le volume n’ont pas été supprimés. 
* Vous pouvez supprimer des instantanés manuels sur le volume source d’une relation de réplication lorsque celle-ci est active ou interrompue, et également après la suppression de la relation de réplication. Vous ne pouvez pas supprimer des instantanés manuels du volume de destination tant que la relation de réplication n’est pas rompue.
* Vous ne pouvez pas revenir à un instantané pris avant la création du volume de destination de réplication.

## <a name="next-steps"></a>Étapes suivantes
* [Créer une réplication de volume](cross-region-replication-create-peering.md)
* [Afficher l’état d’intégrité de la relation de réplication](cross-region-replication-display-health-status.md)
* [Gérer la reprise d’activité après sinistre](cross-region-replication-manage-disaster-recovery.md)
* [Métriques de réplication de volume](azure-netapp-files-metrics.md#replication)
* [Supprimer des volumes ou des réplications de volume](cross-region-replication-delete.md)
* [Résoudre les problèmes de réplication inter-région](troubleshoot-cross-region-replication.md)


