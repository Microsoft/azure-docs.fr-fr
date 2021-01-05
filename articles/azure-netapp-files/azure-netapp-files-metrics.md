---
title: Métriques pour Azure NetApp Files | Microsoft Docs
description: Azure NetApp Files fournit des métriques sur le stockage alloué, l’utilisation réelle du stockage, les IOPS du volume et la latence. Utilisez ces métriques pour comprendre l’utilisation et les performances.
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
ms.date: 12/04/2020
ms.author: b-juche
ms.openlocfilehash: a17e6cc0479cf8ff2306736994a369d9e44dfdda
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96745942"
---
# <a name="metrics-for-azure-netapp-files"></a>Métriques pour Azure NetApp Files

Azure NetApp Files fournit des métriques sur le stockage alloué, l’utilisation réelle du stockage, les IOPS du volume et la latence. En analysant ces métriques, vous en saurez plus sur le mode d'utilisation et les performances des volumes de vos comptes NetApp.  

## <a name="usage-metrics-for-capacity-pools"></a><a name="capacity_pools"></a>Métriques d'utilisation des pools de capacités

- *Taille allouée au pool*   
    Taille provisionnée du pool.

- *Pool alloué à la taille du volume*  
    Quota total des volumes (en Gio) d’un pool de capacités donné (autrement dit, total des tailles provisionnées pour les volumes du pool de capacités).  
    Il s’agit de la taille que vous avez sélectionnée lors de la création des volumes.  

- *Taille du pool consommée*  
    Espace logique total (en Gio) utilisé sur l’ensemble des volumes d’un pool de capacités.  

- *Taille totale des instantanés du pool*    
    Somme de la taille des instantanés de tous les volumes du pool.

## <a name="usage-metrics-for-volumes"></a><a name="volumes"></a>Métriques d'utilisation des volumes

- *Taille du volume consommée en pourcentage*    
    Pourcentage du volume consommé, y compris les instantanés.  
- *Taille allouée aux volumes*   
    Taille provisionnée d’un volume
- *Taille de quota du volume*    
    Taille de quota (Gio) avec laquelle le volume est approvisionné.   
-  *Taille du volume consommée*  
    Taille logique du volume (octets utilisés).  
    Cette taille inclut l'espace logique utilisé par les captures instantanées et les systèmes de fichiers actifs.  
- *Taille du cliché instantané de volume*   
   Taille de tous les instantanés dans un volume.  

## <a name="performance-metrics-for-volumes"></a>Métriques de performances des volumes

- *Latence de lecture moyenne*   
    Temps moyen des lectures du volume (en millisecondes).
- *Latence d’écriture moyenne*   
    Temps moyen des écritures du volume (en millisecondes).
- *E/S par seconde en lecture*   
    Nombre de lectures sur le volume par seconde.
- *E/S par seconde en écriture*   
    Nombre d’écritures sur le volume par seconde.
<!-- These two metrics are not yet available, until ~ 2020.09
- *Read MiB/s*   
    Read throughput in bytes per second.
- *Write MiB/s*   
    Write throughput in bytes per second.
--> 
<!-- ANF-4128; 2020.07
- *Pool Provisioned Throughput*   
    The total throughput a capacity pool can provide to its volumes based on "Pool Provisioned Size" and "Service Level".
- *Pool Allocated to Volume Throughput*   
    The total throughput allocated to volumes in a given capacity pool (that is, the total of the volumes' allocated throughput in the capacity pool).
-->

<!-- ANF-6443; 2020.11
- *Pool Consumed Throughput*    
    The total throughput being consumed by volumes in a given capacity pool.
-->


## <a name="volume-replication-metrics"></a><a name="replication"></a>Métriques de réplication de volume

> [!NOTE] 
> * La taille de transfert réseau (par exemple, les métriques *Transfert total de la réplication de volume*) peut différer des volumes source ou de destination d’une réplication entre régions. Ce comportement est dû à l’utilisation d’un moteur de réplication efficace pour réduire au maximum le coût du transfert réseau.
> * Les métriques de réplication de volume sont actuellement renseignées pour les volumes de destination de réplication et non la source de la relation de réplication.

- *L’état de la réplication de volume est-il sain*   
    Condition de la relation de réplication. Un état sain est indiqué par `1`. Un état non sain est indiqué par `0`.

- *La réplication de volume est-elle en cours de transfert*    
    L’état de la réplication de volume est-il « En cours de transfert ». 
 
- *Décalage de la réplication de volume*   
    Durée, en secondes, du retard des données présentes sur le miroir par rapport à la source 

- *Durée du dernier transfert de réplication de volume*   
    Durée, en secondes, nécessaire au dernier transfert 

- *Taille du dernier transfert de réplication de volume*    
    Nombre total d’octets transférés dans le cadre du dernier transfert 

- *Progression de la réplication de volume*    
    Le volume total de données transférées pour l’opération de transfert en cours. 

- *Transfert total de la réplication de volume*   
    Les octets cumulés transférés pour la relation. 

## <a name="next-steps"></a>Étapes suivantes

* [Comprendre la hiérarchie de stockage d’Azure NetApp Files](azure-netapp-files-understand-storage-hierarchy.md)
* [Configurer un pool de capacité](azure-netapp-files-set-up-capacity-pool.md)
* [Créer un volume pour Azure NetApp Files](azure-netapp-files-create-volumes.md)
