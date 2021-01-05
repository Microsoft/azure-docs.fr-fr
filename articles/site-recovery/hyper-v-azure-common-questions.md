---
title: Questions courantes sur la récupération d’urgence pour Hyper-V avec Azure Site Recovery
description: Cet article récapitule les questions courantes sur la configuration de la reprise d’activité de machines virtuelles Hyper-V locales sur Azure à l’aide du site Azure Site Recovery.
ms.date: 11/12/2019
ms.topic: conceptual
ms.openlocfilehash: e4f5ee71fc56cad247b57f94bf08a68bed8775f9
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96754349"
---
# <a name="common-questions---hyper-v-to-azure-disaster-recovery"></a>Questions courantes sur la reprise d’activité d’Hyper-V sur Azure

Cet article fournit des réponses aux questions courantes qui se posent lors de la réplication de machines virtuelles Hyper-V en local vers Azure. 

## <a name="general"></a>Général

### <a name="how-is-site-recovery-priced"></a>Comment les tarifs Azure Site Recovery sont-ils fixés ?
Pour plus d’informations, consultez [Tarification Site Recovery ](https://azure.microsoft.com/pricing/details/site-recovery/).

### <a name="how-do-i-pay-for-azure-vms"></a>Comment payer les machines virtuelles Azure ?
Lors de la réplication, les données sont répliquées vers le stockage Azure et vous ne payez aucune modification de machine virtuelle. Quand vous exécutez un basculement vers Azure, Site Recovery crée automatiquement des machines virtuelles Azure IaaS. Vous êtes ensuite facturé pour les ressources de calcul que vous consommez dans Azure.

### <a name="is-there-any-difference-in-cost-when-replicating-to-general-purpose-v2-storage-account"></a>Existe-t-il une différence de coût en ce qui concerne la réplication vers le compte de stockage à usage général v2 ?

Vous verrez généralement une augmentation du coût des transactions effectuées sur les comptes de stockage GPv2 dans la mesure où Azure Site Recovery comporte un nombre important de transactions. [En savoir plus](../storage/common/storage-account-upgrade.md#pricing-and-billing) pour estimer les modifications.

## <a name="azure"></a>Azure

### <a name="what-do-i-need-in-hyper-v-to-orchestrate-replication-with-site-recovery"></a>Quelle est la configuration requise dans Hyper-V pour orchestrer la réplication avec Site Recovery ?

Ce dont vous avez besoin pour le serveur hôte Hyper-V dépend du scénario de déploiement. Découvrez les prérequis dans :

* [Réplication de machines virtuelles Hyper-V (sans VMM) sur Azure](./hyper-v-azure-tutorial.md)
* [Réplication de machines virtuelles Hyper-V (avec VMM) sur Azure](./hyper-v-vmm-disaster-recovery.md)
* [Réplication de machines virtuelles Hyper-V sur un centre de données secondaire](./hyper-v-vmm-disaster-recovery.md)
* Si vous répliquez sur un centre de données secondaire, consultez l’article [Systèmes d’exploitation invités pris en charge pour les ordinateurs virtuels Hyper-V](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/mt126277(v=ws.11)).
* Si vous effectuez une réplication vers Azure, Site Recovery prend en charge tous les systèmes d’exploitation invités [pris en charge par Azure](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc794868(v=ws.10)).

### <a name="can-i-protect-vms-when-hyper-v-is-running-on-a-client-operating-system"></a>Puis-je protéger des machines virtuelles lorsque Hyper-V est en cours d’exécution sur un système d’exploitation client ?
Non, les machines virtuelles doivent se trouver sur un serveur hôte Hyper-V s’exécutant sur une machine serveur Windows prise en charge. Si vous devez protéger un ordinateur client, vous pouvez le répliquer en tant que machine physique [vers Azure](./vmware-azure-tutorial.md) ou vers un [centre de données secondaire](./vmware-physical-secondary-disaster-recovery.md).

### <a name="do-hyper-v-hosts-need-to-be-in-vmm-clouds"></a>Les hôtes Hyper-V doivent-ils résider dans des clouds VMM ?
Si vous souhaitez effectuer une réplication vers un centre de données secondaire, les machines virtuelles Hyper-V doivent résider sur des serveurs hôtes Hyper-V situés dans un cloud VMM. Si vous souhaitez procéder à une réplication vers Azure, vous pouvez répliquer des machines virtuelles avec ou sans clouds VMM. [En savoir plus](./hyper-v-azure-tutorial.md) sur la réplication Hyper-V dans Azure.


### <a name="can-i-replicate-hyper-v-generation-2-virtual-machines-to-azure"></a>Puis-je répliquer des machines virtuelles de génération 2 Hyper-V vers Azure ?
Oui. Site Recovery les convertit de la génération 2 à la génération 1 pendant le basculement. Au moment de la restauration automatique, la machine est reconvertie en génération 2. [En savoir plus](https://azure.microsoft.com/blog/2015/04/28/disaster-recovery-to-azure-enhanced-and-were-listening/).


### <a name="can-i-deploy-site-recovery-with-vmm-if-i-only-have-one-vmm-server"></a>Puis-je déployer Site Recovery avec VMM si je ne dispose que d’un seul serveur VMM ?

Oui. Vous pouvez répliquer des machines virtuelles Hyper-V dans le cloud VMM vers Azure, ou répliquer entre des clouds VMM sur le même serveur. Pour la réplication entre sites locaux, nous recommandons de disposer d’un serveur VMM dans les sites principaux et secondaires. 

### <a name="what-do-i-need-in-azure"></a>De quoi ai-je besoin dans Azure ?
Vous avez besoin d’un abonnement Azure, d’un coffre Recovery Services, d’un compte de stockage et d’un réseau virtuel. Le coffre, le compte de stockage et le réseau doivent se trouver dans la même région.

### <a name="what-azure-storage-account-do-i-need"></a>De quel compte de stockage Azure ai-je besoin ?
Vous devez disposer d’un compte de stockage LRS ou GRS. Nous vous recommandons d’utiliser un compte GRS, afin que les données soient résilientes si une panne se produit au niveau régional, ou si la région principale ne peut pas être récupérée. Le stockage Premium est pris en charge.

### <a name="does-my-azure-account-need-permissions-to-create-vms"></a>Est-ce que mon compte Azure a besoin d’autorisations pour créer des machines virtuelles ?
Si vous êtes un administrateur d’abonnement, vous disposez des autorisations de réplication nécessaires. Si ce n’est pas le cas, vous avez besoin d’autorisations pour créer une machine virtuelle Azure dans le groupe de ressources et le réseau virtuel que vous spécifiez quand vous configurez Site Recovery et des autorisations pour écrire dans le compte de stockage sélectionné. [Plus d’informations](site-recovery-role-based-linked-access-control.md#permissions-required-to-enable-replication-for-new-virtual-machines)

### <a name="is-replication-data-sent-to-site-recovery"></a>Est-ce que les données de réplication sont envoyées à Site Recovery ?
Non, Site Recovery n’intercepte pas les données répliquées et n’a pas d’informations sur les éléments exécutés sur vos machines virtuelles. Les données de réplication sont échangées entre les hôtes Hyper-V et le stockage Azure. Site Recovery n’a aucun moyen d’intercepter ces données. Seules les métadonnées nécessaires pour coordonner la réplication et le basculement sont envoyées au service Site Recovery.  

Le logiciel Site Recovery est certifié conforme aux normes ISO 27001:2013, 27018, HIPAA et DPA. Il fait actuellement l’objet d’une évaluation de conformité aux exigences SOC2 et JAB FedRAMP.

### <a name="can-we-keep-on-premises-metadata-within-a-geographic-region"></a>Pouvons-nous conserver des métadonnées en local dans une région géographique ?
Oui. Quand vous créez un coffre dans une région, nous garantissons que toutes les métadonnées utilisées par Site Recovery restent dans les limites géographiques de cette région.

### <a name="does-site-recovery-encrypt-replication"></a>Site Recovery chiffre-t-il la réplication ?
Oui, le chiffrement en transit et le [chiffrement dans Azure](../storage/common/storage-service-encryption.md) sont tous deux pris en charge.


## <a name="deployment"></a>Déploiement

### <a name="what-can-i-do-with-hyper-v-to-azure-replication"></a>Que puis-je faire avec la réplication de Hyper-V vers Azure ?

- **Reprise d’activité** : vous pouvez configurer une reprise d’activité complète. Dans ce scénario, vous répliquez des machines virtuelles Hyper-V en local vers le stockage Azure :
    - Vous pouvez répliquer des machines virtuelles dans Azure. Si votre infrastructure locale n’est pas disponible, vous basculez vers Azure.
    - Quand vous exécutez un basculement, les machines virtuelles Azure sont créées à l’aide des données répliquées. Vous pouvez accéder aux applications et aux charges de travail sur les machines virtuelles Azure.
    - Lorsque votre centre de données local est à nouveau disponible, vous pouvez effectuer une restauration automatique d’Azure vers votre site local.
- **Migration** : vous pouvez utiliser Site Recovery pour migrer les machines virtuelles Hyper-V en local vers le stockage Azure. Ensuite, vous effectuez un basculement du site local vers Azure. Après le basculement, vos applications et charges de travail sont disponibles et s’exécutent sur les machines virtuelles Azure.


### <a name="what-do-i-need-on-premises"></a>De quoi ai-je besoin en local ?

Vous avez besoin d’une ou de plusieurs machines virtuelles en cours d’exécution sur un ou plusieurs hôtes Hyper-V autonomes ou en cluster. Vous pouvez également répliquer des machines virtuelles exécutées sur des hôtes gérés par System Center Virtual Machine Manager (VMM).
- Si vous n’exécutez pas VMM, pendant le déploiement de Site Recovery, vous regroupez les hôtes Hyper-V et les clusters dans des sites Hyper-V. Vous installez les agents Site Recovery (fournisseur Azure Site Recovery et agent Recovery Services) sur chaque hôte Hyper-V.
- Si les hôtes Hyper-V sont situés dans un cloud VMM, vous orchestrez la réplication dans VMM. Vous installez le fournisseur Site Recovery sur le serveur VMM et l’agent Recovery Services sur chaque hôte Hyper-V. Vous mappez entre VMM logique/réseaux machines virtuelles et réseaux virtuels Azure.
- [En savoir plus](hyper-v-azure-architecture.md) sur l’architecture Hyper-V vers Azure.

### <a name="can-i-replicate-vms-located-on-a-hyper-v-cluster"></a>Puis-je répliquer des machines virtuelles situées sur un cluster Hyper-V ?

Oui, Site Recovery prend en charge les hôtes Hyper-V en cluster. Notez les points suivants :

- Tous les nœuds du cluster doivent être inscrits dans le même coffre.
- Si vous n’utilisez pas VMM, tous les hôtes Hyper-V du cluster doivent être ajoutés au même site Hyper-V.
- Vous installez le fournisseur Azure Site Recovery et l’agent Recovery Services sur chaque hôte Hyper-V du cluster et ajoutez chaque hôte à un site Hyper-V.
- Aucune étape spécifique ne doit être effectuée sur le cluster.
- Si vous exécutez l’outil Planificateur de déploiement pour Hyper-V, l’outil collecte les données de profil à partir du nœud qui est en cours d’exécution et où la machine virtuelle est également en cours d’exécution. L’outil ne peut pas collecter de données à partir d’un nœud qui est hors tension, mais il suivra ce nœud. Une fois que le nœud est en cours d’exécution, l’outil démarre la collecte des données de profil de machine virtuelle à partir de celui-ci (si la machine virtuelle fait partie de la liste de machines virtuelles du profil et est en cours d’exécution sur le nœud).
- Si une machine virtuelle sur un hôte Hyper-V dans un coffre Site Recovery est migrée vers un autre hôte Hyper-V du même cluster, ou vers un hôte autonome, sa réplication n’est pas affectée. L’hôte Hyper-V doit satisfaire aux [conditions préalables](hyper-v-azure-support-matrix.md#on-premises-servers) et être configuré dans un coffre Site Recovery. 


### <a name="can-i-protect-vms-when-hyper-v-is-running-on-a-client-operating-system"></a>Puis-je protéger des machines virtuelles lorsque Hyper-V est en cours d’exécution sur un système d’exploitation client ?
Non, les machines virtuelles doivent se trouver sur un serveur hôte Hyper-V s’exécutant sur une machine serveur Windows prise en charge. Si vous devez protéger un ordinateur client, vous pouvez [le répliquer en tant que machine physique](physical-azure-disaster-recovery.md) vers Azure.

### <a name="can-i-replicate-hyper-v-generation-2-virtual-machines-to-azure"></a>Puis-je répliquer des machines virtuelles de génération 2 Hyper-V vers Azure ?
Oui. Site Recovery les convertit de la génération 2 à la génération 1 pendant le basculement. Au moment de la restauration automatique, la machine est reconvertie en génération 2.

### <a name="can-i-automate-site-recovery-scenarios-with-an-sdk"></a>Puis-je automatiser des scénarios Site Recovery avec un kit SDK ?
Oui. Vous pouvez automatiser les flux de travail Site Recovery à l’aide de l’API Rest, de PowerShell ou du kit SDK Azure. Scénarios actuellement pris en charge pour la réplication de Hyper-V vers Azure à l’aide de PowerShell :

- [Répliquer Hyper-V sans VMM à l’aide de PowerShell](hyper-v-azure-powershell-resource-manager.md)
- [Réplication Hyper-V sans VMM à l’aide de PowerShell](hyper-v-vmm-powershell-resource-manager.md)

## <a name="replication"></a>Réplication

### <a name="where-do-on-premises-vms-replicate-to"></a>Quelle est la destination de réplication des machines virtuelles en local ?
Les données sont répliquées vers le stockage Azure. Quand vous exécutez un basculement, Site Recovery crée automatiquement des machines virtuelles Azure à partir du compte de stockage.

### <a name="what-apps-can-i-replicate"></a>Quelles applications puis-je répliquer ?
Vous pouvez répliquer toute application ou charge de travail en cours d’exécution sur une machine virtuelle Hyper-V qui est conforme aux [conditions requises de réplication](hyper-v-azure-support-matrix.md#replicated-vms). Site Recovery assure la prise en charge de la réplication compatible avec les applications afin qu’elles puissent faire l’objet d’un basculement et être restaurées dans un état intelligent. Site Recovery s’intègre aux applications Microsoft, notamment à SharePoint, Exchange, Dynamics, SQL Server et Active Directory, et fonctionne en étroite collaboration avec les principaux fournisseurs, notamment Oracle, SAP, IBM et Red Hat. [En savoir plus](site-recovery-workload.md) sur la protection des charges de travail.

### <a name="whats-the-replication-process"></a>Qu’est-ce que le processus de réplication ?

1. Quand la réplication initiale est déclenchée, une capture instantanée des machines virtuelles Hyper-V est effectuée.
2. Les disques durs virtuels sur la machine virtuelle sont répliqués un par un, jusqu’à ce qu’ils soient tous copiés sur Azure. Cela peut prendre un certain temps selon la taille de la machine virtuelle et la bande passante réseau. Découvrez comment augmenter la bande passante réseau.
3. Si des modifications interviennent sur les disques pendant la réplication initiale, le dispositif de suivi de réplication des réplicas Hyper-V assure le suivi des modifications dans des journaux d’activité de réplication Hyper-V (.hrl). Ces fichiers journaux se trouvent dans le même dossier que les disques. À chaque disque correspond un fichier .hrl, qui est envoyé au stockage secondaire. L’instantané et les fichiers journaux consomment des ressources disque pendant la réplication initiale.
4. Lorsque la réplication initiale s’achève, l’instantané de machine virtuelle est supprimé.
5. Toutes les modifications de disque dans le journal sont synchronisées et fusionnées sur le disque parent.
6. Une fois la réplication initiale terminée, la tâche Finaliser la protection sur la machine virtuelle s’exécute. Elle configure les paramètres réseau et autres paramètres de post-réplication, afin que la machine virtuelle soit protégée.
7. À ce stade, vous pouvez vérifier les paramètres de la machine virtuelle pour vous assurer qu’elle est prête pour le basculement. Vous pouvez exécuter une simulation de récupération d’urgence (test de basculement) pour la machine virtuelle, pour vérifier qu’elle bascule comme prévu.
8. La réplication différentielle commence à l’issue de la réplication initiale, selon la stratégie de réplication.
9. Les modifications sont des fichiers .hrl enregistrés. Chaque disque configuré pour la réplication est associé à un fichier .hrl.
10. Le journal est envoyé au compte de stockage du client. Quand un journal est en transit vers Azure, les modifications apportées au disque principal font l’objet d’un suivi dans un autre fichier journal du même dossier.
11. Pendant la réplication différentielle et initiale, vous pouvez surveiller la machine virtuelle dans le portail Azure.

[En savoir plus](hyper-v-azure-architecture.md#replication-process) sur le processus de réplication.

### <a name="can-i-replicate-to-azure-with-a-site-to-site-vpn"></a>Puis-je répliquer vers Azure avec un VPN de site à site ?

Site Recovery réplique les données en local vers le stockage Azure via un point de terminaison public, ou à l’aide du peering Microsoft ExpressRoute. La réplication sur un réseau VPN de site à site n’est pas prise en charge.

### <a name="can-i-replicate-to-azure-with-expressroute"></a>Puis-je répliquer vers Azure avec ExpressRoute ?

Oui, vous pouvez utiliser ExpressRoute pour répliquer des machines virtuelles vers Azure. Site Recovery réplique les données vers un compte de stockage Azure via un point de terminaison public, et vous devez configurer le [peering Microsoft](../expressroute/expressroute-circuit-peerings.md#microsoftpeering) pour cette réplication. Une fois que les machines virtuelles basculent vers un réseau virtuel Azure, vous pouvez y accéder à l’aide du [peering privé](../expressroute/expressroute-circuit-peerings.md#privatepeering).


### <a name="why-cant-i-replicate-over-vpn"></a>Pourquoi ne puis-je pas répliquer via VPN ?

Lorsque vous répliquez vers Azure, le trafic de réplication atteint les points de terminaison publics d’un compte de stockage Azure. Par conséquent, vous pouvez uniquement répliquer via les réseaux Internet publics avec ExpressRoute (peering Microsoft) et VPN ne fonctionne pas. 

### <a name="what-are-the-replicated-vm-requirements"></a>Quelle est la configuration requise d’une machine virtuelle répliquée ?

Pour la réplication, une machine virtuelle Hyper-V doit exécuter un système d’exploitation pris en charge. En outre, la machine virtuelle doit respecter la configuration requise pour les machines virtuelles Azure. [Découvrez-en plus](hyper-v-azure-support-matrix.md#replicated-vms) dans la matrice de prise en charge.

### <a name="why-is-an-additional-standard-storage-account-required-if-i-replicate-my-virtual-machine-disks-to-premium-storage"></a>Pourquoi un compte de stockage standard supplémentaire est-il requis si je réplique les disques de ma machine virtuelle vers un stockage Premium ?

Lorsque vous répliquez vos machines virtuelles/serveurs physiques locaux vers un stockage Premium, toutes les données résidant sur les disques de l’ordinateur protégé sont répliqués sur le compte de stockage Premium. Un compte de stockage standard supplémentaire est requis pour le stockage de journaux de réplication. Une fois la phase initiale de réplication des données du disque terminée, toutes les modifications apportées aux données du disque local sont suivies en continu et stockées sous forme de journaux de réplication dans ce compte de stockage standard supplémentaire.

### <a name="how-often-can-i-replicate-to-azure"></a>À quelle fréquence puis-je répliquer vers Azure ?

Les machines virtuelles Hyper-V peuvent être répliquées toutes les 30 secondes (sauf pour le stockage premium) ou toutes les 5 minutes.

### <a name="can-azure-site-recovery-and-hyper-v-replica-be-configured-together-on-a-hyper-v-machine"></a>Azure Site Recovery et le réplica Hyper-V peuvent-ils être configurés ensemble sur un ordinateur Hyper-V ?

Oui, Azure Site Recovery et le réplica Hyper-V peuvent être configurés ensemble pour un ordinateur. Toutefois, l’ordinateur doit être protégé comme une machine physique et sera répliqué sur Azure à l’aide d’un serveur de configuration/traitement. En savoir plus sur la protection des machines physiques [ici](https://docs.microsoft.com/azure/site-recovery/physical-azure-architecture).

### <a name="can-i-extend-replication"></a>Puis-je étendre la réplication ?
La réplication étendue ou chaînée n’est pas prise en charge. Demandez cette fonctionnalité dans le [forum de commentaires](https://feedback.azure.com/forums/256299-site-recovery/suggestions/6097959).

### <a name="can-i-do-an-offline-initial-replication"></a>Puis-je effectuer une réplication initiale hors connexion ?
Ceci n’est pas pris en charge. Demandez cette fonctionnalité dans le [forum de commentaires](https://feedback.azure.com/forums/256299-site-recovery/suggestions/6227386-support-for-offline-replication-data-transfer-from).

### <a name="can-i-exclude-disks"></a>Puis-je exclure des disques ?
Oui, vous pouvez exclure des disques de la réplication. 

### <a name="can-i-replicate-vms-with-dynamic-disks"></a>Puis-je répliquer des machines virtuelles avec des disques dynamiques ?
Les disques dynamiques peuvent être répliqués. Le disque du système d’exploitation doit être un disque de base.



## <a name="security"></a>Sécurité

### <a name="what-access-does-site-recovery-need-to-hyper-v-hosts"></a>Quel est l’accès aux machines virtuelles Hyper-V dont Site Recovery a besoin ?

Site Recovery doit pouvoir accéder à des hôtes Hyper-V pour répliquer les machines virtuelles que vous sélectionnez. Site Recovery installe les éléments suivants sur les hôtes Hyper-V :

- Si vous n’exécutez pas VMM, le fournisseur Azure Site Recovery et l’agent Recovery Services sont installés sur chaque hôte.
- Si vous exécutez VMM, l’agent Recovery Services est installé sur chaque hôte. Le fournisseur s’exécute sur le serveur VMM.


### <a name="what-does-site-recovery-install-on-hyper-v-vms"></a>Qu’installe Site Recovery sur les machines virtuelles Hyper-V ?

Site Recovery n’installe rien explicitement sur des machines virtuelles Hyper-V activées pour la réplication.




## <a name="failover-and-failback"></a>Basculement et restauration automatique


### <a name="how-do-i-fail-over-to-azure"></a>Comment basculer vers Azure ?

Vous pouvez effectuer un basculement planifié ou non planifié vers Azure à partir de machines virtuelles Hyper-V locales.

- Si vous exécutez un basculement planifié, les machines virtuelles sources sont arrêtées pour éviter toute perte de données.
- Vous pouvez exécuter un basculement non planifié si votre site principal est inaccessible.
- Vous pouvez basculer une seule machine ou créer des plans de récupération pour orchestrer le basculement de plusieurs machines.
- Le basculement comporte deux parties :
    - Une fois la première phase du basculement effectuée, vous pouvez voir les machines virtuelles de réplication créées dans Azure. Si nécessaire, vous pouvez affecter une adresse IP publique à la machine virtuelle.
    - Vous validez ensuite le basculement, pour accéder à la charge de travail à partir de la machine virtuelle Azure de réplication.
   

### <a name="how-do-i-access-azure-vms-after-failover"></a>Comment accéder aux machines virtuelles Azure après un basculement ?
Après un basculement, vous pouvez accéder aux machines virtuelles Azure via une connexion Internet sécurisée, via un réseau privé virtuel de site à site ou via Azure ExpressRoute. Vous devez préparer un certain nombre de choses afin de vous connecter. [Plus d’informations](failover-failback-overview.md#connect-to-azure-after-failover)

### <a name="is-failed-over-data-resilient"></a>Est-ce que les données ayant fait l’objet d’un basculement sont résilientes ?
Azure est conçu pour la résilience. Site Recovery est prévu pour assurer le basculement vers un centre de données Azure secondaire, dans le respect du contrat SLA Azure. En cas de basculement, nous nous assurons que vos métadonnées et vos coffres restent dans la même région géographique que vous avez choisie pour votre coffre.

### <a name="is-failover-automatic"></a>Le basculement est-il automatique ?
Le [basculement](site-recovery-failover.md) n’est pas automatique. Vous lancez les basculements d’un seul clic dans le portail, ou vous pouvez utiliser [PowerShell](/powershell/module/az.recoveryservices) pour déclencher un basculement.

### <a name="how-do-i-fail-back"></a>Comment effectuer une restauration automatique ?

Une fois votre infrastructure locale à nouveau opérationnelle, vous pouvez effectuer la restauration automatique. La restauration automatique s’effectue en trois étapes :

1. Vous lancez un basculement planifié d’Azure vers le site local à l’aide de quelques options différentes :

    - Minimiser le temps d’arrêt : si vous utilisez cette option, Site Recovery synchronise les données avant le basculement. Il recherche les blocs de données changés et les télécharge sur le site local, sans que la machine virtuelle Azure soit arrêtée, ce qui réduit les interruptions de service. Quand vous spécifiez manuellement que le basculement doit avoir lieu, la machine virtuelle Azure est arrêtée, toutes les données différentielles finales sont copiées et le basculement démarre.
    - Téléchargement complet : avec cette option, les données sont synchronisées pendant le basculement. Cette option télécharge la totalité du disque. L’opération est plus rapide, car aucune somme de contrôle n’est calculée, mais un temps d’arrêt a lieu. Utilisez cette option si vous avez exécuté les machines virtuelles Azure de réplication pendant un certain temps ou que la machine virtuelle locale a été supprimée.

2. Vous pouvez choisir d’effectuer une restauration automatique sur la même machine virtuelle ou sur une autre. Vous pouvez spécifier que Site Recovery doit créer la machine virtuelle si elle n’existe pas déjà.
3. Une fois la synchronisation initiale terminée, vous procédez au basculement. Une fois celui-ci terminé, vous pouvez vous connecter à la machine virtuelle locale pour vérifier que tout fonctionne comme prévu. Dans le portail Azure, vous pouvez voir que les machines virtuelles Azure ont été arrêtées.
4. Vous validez le basculement pour terminer, puis accédez à la charge de travail à partir de la machine virtuelle locale.
5. Une fois les charges de travail restaurées automatiquement, vous activez la réplication inverse, afin que les machines virtuelles locales soient répliquées vers Azure à nouveau.

### <a name="can-i-fail-back-to-a-different-location"></a>Puis-je effectuer la restauration à un autre emplacement ?
Oui, si vous effectuez le basculement vers Azure, vous pouvez effectuer la restauration à un autre emplacement si celui d’origine n’est pas disponible. [Plus d’informations](hyper-v-azure-failback.md#fail-back-to-an-alternate-location)
