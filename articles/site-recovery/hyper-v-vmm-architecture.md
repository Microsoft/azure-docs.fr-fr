---
title: Architecture – Reprise d’activité après sinistre pour Hyper-V sur un site secondaire avec Azure Site Recovery
description: Cet article fournit une vue d’ensemble de l’architecture utilisée pour la récupération d’urgence de machines virtuelles Hyper-V locales sur un site System Center VMM secondaire avec Azure Site Recovery.
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 11/12/2019
ms.author: raynew
ms.openlocfilehash: 703a6afdc12c8a9863ff0f480ec7a577ec31ef77
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87495996"
---
# <a name="architecture---hyper-v-replication-to-a-secondary-site"></a>Architecture – Réplication Hyper-V vers un site secondaire

Cet article décrit les composants et les processus impliqués dans la réplication des machines virtuelles Hyper-V locales dans des clouds System Center Virtual Machine Manager (VMM) sur un site VMM secondaire à l’aide du service [Azure Site Recovery](site-recovery-overview.md) dans le portail Azure.
a

## <a name="architectural-components"></a>Composants architecturaux

Le tableau et le graphique suivants fournissent une vue d’ensemble des composants utilisés pour la réplication Hyper-V vers un site secondaire.

**Composant** | **Prérequis** | **Détails**
--- | --- | ---
**Microsoft Azure** | Abonnement Azure | Vous créez un coffre Recovery Services dans l’abonnement Azure pour orchestrer et gérer la réplication entre les emplacements VMM.
**Serveur VMM** | Vous avez besoin d’un emplacement VMM principal et secondaire. | Nous recommandons l’utilisation d’un serveur VMM dans le site principal et un autre dans le site secondaire.
**Serveur Hyper-V** |  Un ou plusieurs serveurs hôtes Hyper-V dans les clouds VMM principaux et secondaires. | Les données sont répliquées entre les serveurs hôtes Hyper-V principal et secondaire via une liaison LAN ou VPN, en utilisant Kerberos ou une authentification par certificat.  
**Machines virtuelles Hyper-V** | Sur le serveur hôte Hyper-V. | Le serveur hôte source doit avoir au moins une machine virtuelle que vous souhaitez répliquer.

**Architecture local vers local**

![Diagramme montrant la protection d’un déploiement de site local vers un site local.](./media/hyper-v-vmm-architecture/arch-onprem-onprem.png)

## <a name="replication-process"></a>Processus de réplication

1. Au moment où la réplication initiale est déclenchée, un [instantané des machines virtuelles Hyper-V](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd560637(v=ws.10)) a lieu.
2. Les disques durs virtuels sur la machine virtuelle sont répliqués un par un vers l’emplacement secondaire.
3. Si des modifications interviennent sur les disques pendant la réplication initiale, le dispositif de suivi de réplication des réplicas Hyper-V assure le suivi des modifications dans des journaux d’activité de réplication Hyper-V (.hrl). Ces fichiers journaux se trouvent dans le même dossier que les disques. À chaque disque correspond un fichier .hrl, qui est envoyé à l’emplacement secondaire. L’instantané et les fichiers journaux consomment des ressources disque pendant la réplication initiale.
4. Quand la réplication initiale s’achève, l’instantané de machine virtuelle est supprimé et la réplication delta commence.
5. Les modifications d’ordre différentiel dans le fichier journal sont synchronisées et fusionnées sur le disque parent.


## <a name="failover-and-failback-process"></a>Processus de basculement et de restauration automatique

- Vous pouvez basculer une seule machine ou créer des plans de récupération pour orchestrer le basculement de plusieurs machines.
- Vous pouvez effectuer un basculement planifié ou non planifié entre des sites locaux. Si vous exécutez un basculement planifié, les machines virtuelles sources sont arrêtées pour éviter toute perte de données.
    - Si vous effectuez un basculement non planifié vers un site secondaire, une fois cette opération effectuée, les machines de l’emplacement secondaire ne sont pas protégées.
    - Si vous avez lancé un basculement planifié, une fois l’opération terminée, les machines de l’emplacement secondaire sont protégées.
- Une fois que le basculement initial s’exécute, validez-le pour accéder à la charge de travail à partir de la machine virtuelle de réplication.
- Quand l’emplacement principal est à nouveau disponible, vous pouvez effectuer une restauration automatique.
    - Vous initiez la réplication inverse, pour démarrer la réplication depuis le site secondaire vers le site principal. La réplication inverse affecte aux machines virtuelles l’état protégé, mais l’emplacement actif est toujours le centre de données secondaire.
    - Pour placer le site principal à l’emplacement actif, lancez un basculement planifié depuis le site secondaire vers le site principal, puis une autre réplication inverse.



## <a name="next-steps"></a>Étapes suivantes


Suivre [le didacticiel](hyper-v-vmm-disaster-recovery.md) montrant comment activer la réplication Hyper-V entre des clouds VMM.
