---
title: Architecture de la reprise d’activité après sinistre pour Hyper-V dans Azure Site Recovery
description: Cet article fournit une vue d’ensemble des composants et de l’architecture utilisés lors du déploiement de la récupération d’urgence pour des machines virtuelles Hyper-V locales (sans VMM) sur Azure avec le service Azure Site Recovery.
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 11/14/2019
ms.author: raynew
ms.openlocfilehash: c5025b83619b505728bfdf5c4e1ccc81d3bb225e
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97654759"
---
# <a name="hyper-v-to-azure-disaster-recovery-architecture"></a>Architecture pour la récupération d’urgence de Hyper-V vers Azure


Cet article décrit l’architecture et les processus utilisés quand vous répliquez, basculez et récupérez des machines virtuelles Hyper-V entre des hôtes Hyper-V locaux et Azure, à l’aide du service [Azure Site Recovery](site-recovery-overview.md).

Vous pouvez éventuellement gérer les hôtes Hyper-V dans des clouds privés System Center Virtual Machine Manager (VMM).



## <a name="architectural-components---hyper-v-without-vmm"></a>Composants architecturaux : Hyper-V sans VMM

Le tableau et le graphique suivants fournissent une vue d’ensemble des composants utilisés pour la réplication Hyper-V sur Azure, quand les hôtes Hyper-V ne sont pas gérés par VMM.

**Composant** | **Prérequis** | **Détails**
--- | --- | ---
**Microsoft Azure** | Un abonnement Azure, un compte de stockage Azure et un réseau Azure. | Les données répliquées à partir de charges de travail de machines virtuelles locales sont stockées dans le compte de stockage. Les machines virtuelles Azure sont créées avec les données de charge de travail répliquées pendant le basculement à partir de votre site local.<br/><br/> Les machines virtuelles Azure se connectent au réseau virtuel Azure lors de leur création.
**Hyper-V** | Pendant le déploiement de Site Recovery, vous regroupez les hôtes Hyper-V et les clusters dans des sites Hyper-V. Vous installez le fournisseur Azure Site Recovery et l’agent Recovery Services sur chaque hôte Hyper-V autonome ou sur chaque nœud de cluster Hyper-V. | Le fournisseur orchestre la réplication avec Site Recovery via Internet. L’agent Recovery Services gère la réplication des données.<br/><br/> Les communications en provenance du fournisseur et de l’agent sont sécurisées et chiffrées. Les données répliquées dans le stockage Azure sont également chiffrées.
**Machines virtuelles Hyper-V** | Une ou plusieurs machines virtuelles s’exécutant sur Hyper-V. | Rien ne doit explicitement être installé sur les machines virtuelles.


**Architecture Hyper-V vers Azure (sans VMM)**

![Diagramme montrant l’architecture d’un site Hyper-V local vers Azure sans VMM.](./media/hyper-v-azure-architecture/arch-onprem-azure-hypervsite.png)


## <a name="architectural-components---hyper-v-with-vmm"></a>Composants architecturaux : Hyper-V avec VMM

Le tableau et le graphique suivants fournissent une vue d’ensemble des composants utilisés pour la réplication Hyper-V vers Azure, quand les hôtes Hyper-V sont gérés dans des clouds VMM.

**Composant** | **Prérequis** | **Détails**
--- | --- | ---
**Microsoft Azure** | Un abonnement Azure, un compte de stockage Azure et un réseau Azure. | Les données répliquées à partir de charges de travail de machines virtuelles locales sont stockées dans le compte de stockage. Les machines virtuelles Azure sont créées avec les données répliquées lors du basculement à partir de votre site local.<br/><br/> Les machines virtuelles Azure se connectent au réseau virtuel Azure lors de leur création.
**Serveur VMM** | Le serveur VMM a un ou plusieurs clouds contenant des hôtes Hyper-V. | Installez Site Recovery Provider sur le serveur VMM pour organiser la réplication avec Site Recovery, et enregistrez le serveur dans le coffre Recovery Services.
**Hôte Hyper-V** | Un ou plusieurs clusters ou hôtes Hyper-V gérés par VMM. |  Vous installez l’agent Recovery Services sur chaque hôte ou nœud de cluster Hyper-V.
**Machines virtuelles Hyper-V** | Une ou plusieurs machines virtuelles s’exécutant sur le serveur hôte Hyper-V. | Rien ne doit explicitement être installé sur les machines virtuelles.
**Mise en réseau** | Installation des réseaux logiques et des réseaux de machines virtuelles sur le serveur VMM. Le réseau de machines virtuelles doit être lié à un réseau logique lui-même associé au cloud. | Les réseaux de machines virtuelles sont mappés à des réseaux virtuels Azure. Quand les machines virtuelles Azure sont créées après le basculement, elles sont ajoutées au réseau Azure qui est mappé au réseau de machines virtuelles.

**Architecture Hyper-V vers Azure (avec VMM)**

![Diagramme montrant l’architecture d’un site Hyper-V local vers Azure avec VMM.](./media/hyper-v-azure-architecture/arch-onprem-onprem-azure-vmm.png)

## <a name="set-up-outbound-network-connectivity"></a>Configurer la connectivité réseau sortante

Pour que Site Recovery fonctionne comme prévu, vous devez modifier la connectivité réseau sortante pour permettre la réplication de votre environnement.

> [!NOTE]
> Site Recovery ne prend pas en charge l’utilisation d’un proxy d’authentification pour contrôler la connectivité réseau.

### <a name="outbound-connectivity-for-urls"></a>Connectivité sortante pour les URL

Si vous utilisez un proxy de pare-feu basé sur des URL pour contrôler la connectivité sortante, autorisez l’accès à ces URL :

| **Nom**                  | **Commercial**                               | **Secteur public**                                 | **Description** |
| ------------------------- | -------------------------------------------- | ---------------------------------------------- | ----------- |
| Stockage                   | `*.blob.core.windows.net`                  | `*.blob.core.usgovcloudapi.net` | Permet d’écrire les données dans le compte de stockage de cache dans la région source à partir de la machine virtuelle. |
| Azure Active Directory    | `login.microsoftonline.com`                | `login.microsoftonline.us`                   | Fournit l’autorisation et l’authentification aux URL du service Site Recovery. |
| Réplication               | `*.hypervrecoverymanager.windowsazure.com` | `*.hypervrecoverymanager.windowsazure.com`   | Permet à la machine virtuelle de communiquer avec le service Site Recovery. |
| Service Bus               | `*.servicebus.windows.net`                 | `*.servicebus.usgovcloudapi.net`             | Permet à la machine virtuelle d’écrire des données de surveillance et de diagnostic Site Recovery. |


## <a name="replication-process"></a>Processus de réplication

![Diagramme montrant le processus de réplication Hyper-V vers Azure](./media/hyper-v-azure-architecture/arch-hyperv-azure-workflow.png)

**Processus de réplication et de récupération**


### <a name="enable-protection"></a>Activer la protection

1. Une fois que vous activez la protection d’une machine virtuelle Hyper-V, dans le portail Azure ou en local, **l’activation de la protection** démarre.
2. Le travail vérifie que la machine est conforme à la configuration requise, puis appelle la méthode [CreateReplicationRelationship](/windows/win32/hyperv_v2/createreplicationrelationship-msvm-replicationservice), laquelle configure la réplication avec les paramètres que vous avez configurés.
3. Le travail démarre la réplication initiale en appelant la méthode [StartReplication](/windows/win32/hyperv_v2/startreplication-msvm-replicationservice) pour initialiser une réplication complète de la machine virtuelle et envoyer les disques virtuels de la machine virtuelle sur Azure.
4. Vous pouvez surveiller le travail dans l'onglet **Travaux**.      ![Screenshot of the jobs list in the Jobs tab.](media/hyper-v-azure-architecture/image1.png) ![Screenshot of the Enable protection screen with more details.](media/hyper-v-azure-architecture/image2.png)


### <a name="initial-data-replication"></a>Réplication initiale des données

1. Au moment où la réplication initiale est déclenchée, un [instantané des machines virtuelles Hyper-V](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd560637(v=ws.10)) a lieu.
2. Les disques durs virtuels sur la machine virtuelle sont répliqués un par un, jusqu’à ce qu’ils soient tous copiés sur Azure. Cela peut prendre un certain temps selon la taille de la machine virtuelle et la bande passante réseau. [Découvrez comment](https://support.microsoft.com/kb/3056159) augmenter la bande passante réseau.
3. Si des modifications interviennent sur les disques pendant la réplication initiale, le dispositif de suivi de réplication des réplicas Hyper-V assure le suivi des modifications dans des journaux d’activité de réplication Hyper-V (.hrl). Ces fichiers journaux se trouvent dans le même dossier que les disques. À chaque disque correspond un fichier .hrl, qui est envoyé au stockage secondaire. L’instantané et les fichiers journaux consomment des ressources disque pendant la réplication initiale.
4. Lorsque la réplication initiale s’achève, l’instantané de machine virtuelle est supprimé.
5. Les modifications d’ordre différentiel dans le fichier journal sont synchronisées et fusionnées sur le disque parent.


### <a name="finalize-protection-process"></a>Finaliser le processus de protection

1. Une fois la réplication initiale terminée, la tâche **Finaliser la protection sur la machine virtuelle** s’exécute. Elle configure les paramètres réseau et autres paramètres de post-réplication, afin que la machine virtuelle soit protégée.
2. À ce stade, vous pouvez vérifier les paramètres de la machine virtuelle pour vous assurer qu’elle est prête pour le basculement. Vous pouvez exécuter une simulation de récupération d’urgence (test de basculement) pour la machine virtuelle, pour vérifier qu’elle bascule comme prévu. 


## <a name="delta-replication"></a>Réplication différentielle

1. La réplication différentielle commence à l’issue de la réplication initiale, selon la stratégie de réplication.
2. Le dispositif de suivi de réplication des réplicas Hyper-V assure le suivi des modifications sur un disque dur virtuel, dans des fichiers .hrl. Chaque disque configuré pour la réplication est associé à un fichier .hrl.
3. Le journal est envoyé au compte de stockage du client. Quand un journal est en transit vers Azure, les modifications apportées au disque principal font l’objet d’un suivi dans un autre fichier journal du même dossier.
4. Pendant la réplication différentielle et initiale, vous pouvez surveiller la machine virtuelle dans le portail Azure.

### <a name="resynchronization-process"></a>Processus de resynchronisation

1. Si la réplication différentielle échoue et si une réplication complète est exclue (car elle monopoliserait trop de bande passante ou prendrait trop de temps), une resynchronisation se produit au niveau d’une machine virtuelle.
    - Par exemple, si les fichiers .hrl atteignent 50 % de la taille du disque, la machine virtuelle est marquée pour resynchronisation.
    -  Par défaut, la resynchronisation est planifiée pour s’exécuter automatiquement en dehors des heures de bureau.
1.  La resynchronisation envoie uniquement les données différentielles.
    - Elle réduit la quantité de données envoyées en calculant des sommes de contrôle des machines virtuelles source et cible.
    - Elle utilise un algorithme de segmentation de bloc fixe, dans lequel les fichiers source et cible sont divisés en segments fixes.
    - Des sommes de contrôle pour chaque segment sont générés. Ceux-ci sont comparés, afin de savoir quels blocs de la source doivent être appliqués à la cible.
2. Une fois la resynchronisation terminée, la réplication différentielle normale doit reprendre.
3. Si vous ne souhaitez pas attendre la resynchronisation par défaut en dehors des heures de bureau, vous pouvez resynchroniser une machine virtuelle manuellement, par exemple, en cas de panne. Pour ce faire, dans le portail Azure, sélectionnez **Resynchroniser**.

    ![Capture d’écran montrant l’option Resynchroniser.](./media/hyper-v-azure-architecture/image4-site.png)


### <a name="retry-process"></a>Processus de nouvelle tentative

Si une erreur de réplication se produit, une nouvelle tentative intégrée est effectuée. La nouvelle tentative appartient à l’une des catégories décrites dans le tableau suivant.

**Catégorie** | **Détails**
--- | ---
**Erreurs non récupérables** | Aucune nouvelle tentative n’est effectuée. L’état de la machine virtuelle est affiché comme **Critique** et l’intervention de l’administrateur est requise.<br/><br/> Parmi les exemples d’erreurs, citons une chaîne de disques durs virtuels rompue, un état non valide pour les machines virtuelles de réplication, les erreurs d’authentification du réseau, les erreurs d’autorisations et les erreurs de machines virtuelles non trouvées (pour les serveurs Hyper-V autonomes).
**Erreurs récupérables** | De nouvelles tentatives se produisent à chaque intervalle de réplication, à l’aide d’une interruption exponentielle laquelle augmente l’intervalle de récupération de 1, 2, 4, 8 et 10 minutes, dès le début de la première tentative. Si une erreur persiste, effectuez une nouvelle tentative toutes les 30 minutes. Parmi les exemples, citons les erreurs réseau, les erreurs espace disque faible et les conditions de mémoire insuffisante.



## <a name="failover-and-failback-process"></a>Processus de basculement et de restauration automatique

1. Vous pouvez effectuer un basculement planifié ou non planifié vers Azure à partir de machines virtuelles Hyper-V locales. Si vous exécutez un basculement planifié, les machines virtuelles sources sont arrêtées pour éviter toute perte de données. Exécutez un basculement non planifié si votre site principal est inaccessible.
2. Vous pouvez basculer une seule machine ou créer des plans de récupération pour orchestrer le basculement de plusieurs machines.
3. Vous exécutez un basculement. Une fois la première phase du basculement effectuée, vous pouvez voir les machines virtuelles de réplication créées dans Azure. Si nécessaire, vous pouvez affecter une adresse IP publique à la machine virtuelle.
4. Vous validez ensuite le basculement, pour accéder à la charge de travail à partir de la machine virtuelle Azure de réplication.

Une fois votre infrastructure locale à nouveau opérationnelle, vous pouvez effectuer la restauration automatique. La restauration automatique s’effectue en trois étapes :

1. Effectuez un basculement planifié depuis Azure vers le site local :
    - **Minimiser le temps d’arrêt** : si vous utilisez cette option, Site Recovery synchronise les données avant le basculement. Il recherche les blocs de données changés et les télécharge sur le site local, sans que la machine virtuelle Azure soit arrêtée, ce qui réduit les interruptions de service. Quand vous spécifiez manuellement que le basculement doit avoir lieu, la machine virtuelle Azure est arrêtée, toutes les données différentielles finales sont copiées et le basculement démarre.
    - **Téléchargement complet** : avec cette option, les données sont synchronisées pendant le basculement. Cette option télécharge la totalité du disque. L’opération est plus rapide, car aucune somme de contrôle n’est calculée, mais un temps d’arrêt a lieu. Utilisez cette option si vous avez exécuté les machines virtuelles Azure de réplication pendant un certain temps ou que la machine virtuelle locale a été supprimée.
    - **Créer une machine virtuelle** : Vous pouvez choisir d’effectuer une restauration automatique sur la même machine virtuelle ou sur une autre. Vous pouvez spécifier que Site Recovery doit créer la machine virtuelle si elle n’existe pas déjà.

2. Une fois la synchronisation initiale terminée, vous procédez au basculement. Une fois celui-ci terminé, vous pouvez vous connecter à la machine virtuelle locale pour vérifier que tout fonctionne comme prévu. Dans le portail Azure, vous pouvez voir que les machines virtuelles Azure ont été arrêtées.
3.  Ensuite, vous validez le basculement pour terminer, puis accédez à la charge de travail à partir de la machine virtuelle locale.
4. Une fois les charges de travail restaurées automatiquement, vous activez la réplication inverse, afin que les machines virtuelles locales soient répliquées vers Azure à nouveau.



## <a name="next-steps"></a>Étapes suivantes


Suivez [ce didacticiel](tutorial-prepare-azure.md) pour découvrir la réplication Hyper-V vers Azure.


