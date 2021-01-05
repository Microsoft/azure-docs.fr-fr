---
title: Avantages de l’utilisation d’Azure NetApp Files avec Oracle Database | Microsoft Docs
description: Décrit les technologies et fournit une comparaison des performances entre Oracle Direct NFS (dNFS) et le client NFS traditionnel. Montre les avantages de l’utilisation de dNFS avec Azure NetApp Files.
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
ms.date: 04/23/2020
ms.author: b-juche
ms.openlocfilehash: 0f431015caf5bb3364fe5628a599f760c6318c47
ms.sourcegitcommit: 50802bffd56155f3b01bfb4ed009b70045131750
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91932496"
---
# <a name="benefits-of-using-azure-netapp-files-with-oracle-database"></a>Avantages de l’utilisation d’Azure NetApp Files avec Oracle Database

Oracle Direct NFS (dNFS) permet d’améliorer les performances par rapport au pilote NFS propre au système d’exploitation. Cet article explique les technologies et fournit une comparaison des performances entre dNFS et le client NFS traditionnel (NFS en mode Noyau). Il montre également les avantages et la facilité inhérents à l’utilisation de dNFS avec Azure NetApp Files.  

## <a name="how-oracle-direct-nfs-works"></a>Fonctionnement d’Oracle Direct NFS

Le récapitulatif suivant explique le fonctionnement d’Oracle Direct NFS à un niveau élevé :

* Oracle Direct NFS contourne le cache des tampons du système d’exploitation. Les données ne sont mises en cache qu’une seule fois dans l’espace utilisateur, ce qui élimine la surcharge des copies en mémoire.  

* Le client NFS traditionnel utilise un seul flux réseau, comme indiqué ci-dessous :    

    ![Client NFS traditionnel utilisant un seul flux réseau](../media/azure-netapp-files/solutions-traditional-nfs-client-using-single-network-flow.png)

    Oracle Direct NFS améliore les performances en équilibrant la charge du trafic réseau entre plusieurs flux réseau. Comme testé et illustré ci-dessous, 650 connexions réseau distinctes ont été établies dynamiquement par la base de données Oracle :  

    ![Oracle Direct NFS améliore les performances](../media/azure-netapp-files/solutions-oracle-direct-nfs-performance-load-balancing.png)

La [FAQ Oracle pour Direct NFS](http://www.orafaq.com/wiki/Direct_NFS) montre qu’Oracle dNFS est un client NFS optimisé. Il fournit un accès rapide et scalable au stockage NFS qui se trouve sur les périphériques de stockage NAS (accessible via TCP/IP). dNFS est intégré au noyau de base de données, tout comme ASM, qui est principalement utilisé avec le stockage DAS ou SAN. Par conséquent, *il est recommandé d’utiliser dNFS lors de l’implémentation du stockage NAS et d’utiliser ASM lors de l’implémentation du stockage SAN.*

dNFS est l’option par défaut dans Oracle 18C.

dNFS est disponible à partir de Oracle Database 11g. Le schéma ci-dessous compare dNFS avec un NFS natif. Lorsque vous utilisez dNFS, une base de données Oracle qui s’exécute sur une machine virtuelle Azure peut générer plus d’E/S que le client NFS natif.

![Comparaison Oracle et Azure NetApp Files entre dNFS et un NFS natif](../media/azure-netapp-files/solutions-oracle-azure-netapp-files-comparing-dnfs-native-nfs.png)

Vous pouvez activer ou désactiver dNFS en exécutant deux commandes et en redémarrant la base de données.

Pour activer :  
`cd $ORACLE_HOME/rdbms/lib ; make -f ins_rdbms.mk dnfs_on`

Pour désactiver :  
`cd $ORACLE_HOME/rdbms/lib ; make -f ins_rdbms.mk dnfs_off`

## <a name="azure-netapp-files-combined-with-oracle-direct-nfs"></a>Azure NetApp Files combiné avec Oracle Direct NFS

Vous pouvez améliorer les performances d’Oracle dNFS avec le service Azure NetApp Files. Le service vous donne un contrôle total sur les performances de votre application. Il peut répondre à des applications extrêmement exigeantes. La combinaison d’Oracle dNFS avec Azure NetApp Files offre un grand avantage pour vos charges de travail.

## <a name="next-steps"></a>Étapes suivantes

- [Architectures de solution avec Azure NetApp Files](azure-netapp-files-solution-architectures.md)
- [Vue d’ensemble des applications et des solutions Oracle sur Azure](../virtual-machines/workloads/oracle/oracle-overview.md)