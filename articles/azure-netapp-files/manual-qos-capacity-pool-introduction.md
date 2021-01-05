---
title: Pool de capacités avec QoS manuelle d’Azure NetApp Files | Microsoft Docs
description: Fournit une présentation du pool de capacités avec QoS manuelle et des références pour obtenir des informations supplémentaires.
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
ms.openlocfilehash: 239b985bf45942afbff8ca49d7e15ead24456304
ms.sourcegitcommit: 83610f637914f09d2a87b98ae7a6ae92122a02f1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91993989"
---
# <a name="manual-qos-capacity-pool"></a>Pool de capacités QoS

Cet article fournit une introduction à la fonctionnalité de pool de capacités avec Qualité de service (QoS) manuelle.

## <a name="how-manual-qos-differs-from-auto-qos"></a>Différences entre la QoS manuelle et la QoS automatique

Le [type de QoS](azure-netapp-files-understand-storage-hierarchy.md#qos_types) est un attribut de pool de capacités. Azure NetApp Files fournit deux types de QoS pour les pools de capacités : automatique (par défaut) et manuelle.  

Au sein d’un pool de capacités avec QoS *manuelle*, vous pouvez attribuer la capacité et le débit d’un volume de manière indépendante. Le débit total de tous les volumes créés avec un pool de capacités de QoS manuel est limité par le débit total du pool. Il est déterminé par la combinaison de la taille du pool et du débit au niveau du service. 

Dans un pool de capacités avec QoS *automatique*, le débit est attribué automatiquement aux volumes du pool, proportionnellement au quota de taille alloué aux volumes.  

Pour plus d’informations sur les types de QoS, consultez [Hiérarchie de stockage d’Azure NetApp Files](azure-netapp-files-understand-storage-hierarchy.md) et [Considérations relatives aux performances pour Azure NetApp Files](azure-netapp-files-performance-considerations.md).

## <a name="example-of-using-manual-qos"></a>Exemple d’utilisation de la QoS manuelle

Lorsque vous utilisez un pool de capacités avec QoS manuelle avec, par exemple, un système SAP HANA, une base de données Oracle ou d’autres charges de travail nécessitant plusieurs volumes, le pool de capacités peut être utilisé pour créer ces volumes d’application.  Chaque volume peut fournir la taille et le débit individuels pour répondre aux besoins de l’application.  Pour plus d’informations sur les avantages, consultez [Exemples de limite de débit de volumes dans un pool de capacités avec QoS manuelle](azure-netapp-files-service-levels.md#throughput-limit-examples-of-volumes-in-a-manual-qos-capacity-pool).  

## <a name="how-to-specify-the-manual-qos-type"></a>Comment spécifier le type QoS manuelle

Lorsque vous [créez un pool de capacités](azure-netapp-files-set-up-capacity-pool.md), vous pouvez spécifier d’utiliser le type QoS manuelle pour le pool de capacités.  Vous pouvez également [modifier un pool de capacités existant](manage-manual-qos-capacity-pool.md#change-to-qos) pour utiliser le type QoS manuelle. 

La définition du type de capacité sur la QoS manuelle est une modification permanente. Vous ne pouvez pas convertir un outil de capacité de type QoS manuelle en pool de capacités avec QoS automatique. 

L’utilisation du type QoS manuelle nécessite que vous [inscriviez la fonctionnalité](manage-manual-qos-capacity-pool.md#register-the-feature).  

## <a name="next-steps"></a>Étapes suivantes

* [Gérer un pool de capacités de QoS manuel](manage-manual-qos-capacity-pool.md)
* [Configurer un pool de capacité](azure-netapp-files-set-up-capacity-pool.md)
* [Hiérarchie de stockage](azure-netapp-files-understand-storage-hierarchy.md) 
* [Niveaux de service pour Azure NetApp Files](azure-netapp-files-service-levels.md)
* [Considérations sur les performances pour Azure NetApp Files](azure-netapp-files-performance-considerations.md)
* [Modèle de coût pour Azure NetApp Files](azure-netapp-files-cost-model.md)
* [Limites des ressources pour Azure NetApp Files](azure-netapp-files-resource-limits.md)
* [Créer un volume NFS](azure-netapp-files-create-volumes.md)
* [Créer un volume SMB](azure-netapp-files-create-volumes-smb.md)
* [Créer un volume à deux protocoles](create-volumes-dual-protocol.md)
* [Métriques pour Azure NetApp Files](azure-netapp-files-metrics.md)
* [Résoudre les problèmes de pools de capacités](troubleshoot-capacity-pools.md)
