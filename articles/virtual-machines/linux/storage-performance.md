---
title: Optimiser les performances sur les machines virtuelles Azure de la série Lsv2 – Stockage
description: Découvrez comment optimiser les performances de votre solution sur les machines virtuelles de la série Lsv2 à l’aide d’un exemple Linux.
services: virtual-machines-linux
author: laurenhughes
ms.service: virtual-machines-linux
ms-subservice: sizes
ms.topic: conceptual
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 08/05/2019
ms.author: joelpell
ms.openlocfilehash: 650164556223a73a722bc91ecb31491ee98cb8a5
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91307099"
---
# <a name="optimize-performance-on-the-lsv2-series-linux-virtual-machines"></a>Optimiser les performances sur les machines virtuelles Linux de la série Lsv2

Les machines virtuelles de la série Lsv2 prennent en charge diverses charges de travail qui nécessitant des E/S et des débits élevés au niveau du stockage local pour un large éventail d’applications et de secteurs d’activité.  La série Lsv2 est idéale pour le Big Data, SQL, les bases de données NoSQL, l’entreposage de données et les bases de données transactionnelles volumineuses, notamment Cassandra, MongoDB, Cloudera et Redis.

La conception des machines virtuelles de la série Lsv2 exploite au maximum le processeur AMD EPYC™ 7551 pour offrir des performances optimales entre le processeur, la mémoire, les appareils NVMe et les machines virtuelles. Grâce à une collaboration avec des partenaires autour de Linux, plusieurs builds sont disponibles sur la Place de marché Azure, qui sont optimisées pour les performances de la série Lsv2 :

- Ubuntu 18.04
- Ubuntu 16.04
- RHEL 8.0
- Debian 9
- Debian 10

Cet article fournit des conseils et des suggestions pour faire en sorte que vos charges de travail et vos applications atteignent les performances maximales dans les machines virtuelles. Les informations contenues dans cette page seront constamment mises à jour à mesure que de nouvelles images optimisées Lsv2 seront ajoutées à la Place de marché Azure.

## <a name="amd-eypc-chipset-architecture"></a>Architecture du circuit microprogrammé AMD EYPC™

Les machines virtuelles de la série Lsv2 utilisent des processeurs serveur AMD EYPC™ basés sur la microarchitecture Zen. AMD a développé Infinity Fabric (IF) pour EYPC™ en tant qu’interconnexion scalable pour son modèle NUMA qui peut être utilisé pour les communications au niveau du die, du package et entre plusieurs packages. Par rapport à QPI (Quick-Path Interconnect) et à UPI (Ultra-Path Interconnect), tous deux utilisés sur les processeurs modernes Intel à dies monolithiques, l’architecture AMD constituée de nombreux dies de petite taille NUMA peut offrir des avantages sur le plan des performances, mais créer aussi des difficultés. L’impact réel des contraintes de bande passante et de latence de la mémoire peut varier selon le type des charges de travail exécutées.

## <a name="tips-to-maximize-performance"></a>Conseils pour optimiser les performances

* Si vous chargez un système d’exploitation invité Linux personnalisé pour votre charge de travail, notez que la mise en réseau accélérée est **désactivée** par défaut. Si vous avez l’intention d’activer la mise en réseau accélérée, pour bénéficier de performances optimales, faites-le pendant la création de machines virtuelles.

* Le matériel qui fait tourner les machines virtuelles de la série Lsv2 utilise des appareils NVMe avec huit paires de files d’attente d’E/S. Chaque file d’attente d’E/S d’appareil NVMe est en fait une paire : une file d’attente d’envoi et une file d’attente d’achèvement. Le pilote NVMe est configuré pour optimiser l’utilisation de ces huit paires de files d’attente d’E/S : les E/S sont distribuées selon un programme de tourniquet (round robin). Pour obtenir des performances maximales, exécutez huit tâches par appareil.

* Évitez de combiner des commandes d’administration NVMe (par exemple, une requête d’information NVMe SMART, etc.) avec des commandes d’E/S NVMe pendant que des charges de travail sont actives. Les appareils NVMe Lsv2 reposent sur la technologie Hyper-V NVMe Direct, qui passe en « mode lent » chaque fois que des commandes d’administration NVMe sont en attente. Dans ce cas, les utilisateurs Lsv2 risquent d’observer une baisse considérable des performances des E/S NVMe.

* Pour définir l’affinité NUMA de leurs applications, les utilisateurs Lsv2 ne doivent pas se fier aux informations NUMA des appareils présentées dans les machines virtuelles pour les lecteurs de données (valeurs 0). Pour de meilleures performances, il est recommandé de répartir les charges de travail entre les UC, dans la mesure du possible.

* La profondeur de file d’attente maximale prise en charge par paire de files d’attente d’E/S pour les appareils NVMe de machines virtuelles Lsv2 est de 1 024 (pour une limite de 32 dans le cas d’Amazon i3). Les utilisateurs Lsv2 doivent limiter leurs charges de travail de benchmarking (synthétique) à une profondeur de file d’attente de 1 024 ou moins pour éviter de déclencher des conditions de file d’attente saturée, ce qui peut faire baisser les performances.

## <a name="utilizing-local-nvme-storage"></a>Utilisation du stockage NVMe local

Le stockage local sur le disque NVMe de 1,92 To est éphémère pour toutes les machines virtuelles Lsv2. Lors d’un redémarrage standard réussi de la machine virtuelle, les données présentes sur le disque NVMe local sont conservées. Les données ne sont pas conservées sur le NVMe si la machine virtuelle est redéployée, désallouée ou supprimée. Les données ne sont pas conservées si la machine virtuelle ou le matériel sur lequel elle s’exécute n’est pas sain à cause d’un autre problème. Quand cela se produit, toutes les données présentes sur l’ancien hôte sont effacées de manière sécurisée.

Dans certains cas, il peut aussi être nécessaire de déplacer la machine virtuelle sur une autre machine hôte, par exemple, à l’occasion d’une opération de maintenance planifiée. Les opérations de maintenance planifiée et certaines défaillances matérielles peuvent être anticipées avec [Scheduled Events](scheduled-events.md). Il est recommandé d’utiliser Scheduled Events pour rester informé des opérations de maintenance et de récupération prévues.

Si un événement de maintenance planifiée exige la récréation de la machine virtuelle sur un nouvel hôte avec des disques locaux vides, les données doivent être resynchronisées (là encore, les données présentes sur l’ancien hôte seront effacées de manière sécurisée). Cela s’explique par le fait que les machines virtuelles de la série Lsv2 ne prennent pas actuellement en charge la migration dynamique sur le disque NVMe local.

Il existe deux modes de maintenance planifiée.

### <a name="standard-vm-customer-controlled-maintenance"></a>Maintenance standard contrôlée par le client de la machine virtuelle

- La machine virtuelle est déplacée sur un hôte mis à jour sur une fenêtre de 30 jours.
- Le risque de perte de données sur le stockage local Lsv2 n’étant pas à écarter, il est recommandé de sauvegarder les données avant l’événement.

### <a name="automatic-maintenance"></a>Maintenance automatique

- Intervient en l’absence de maintenance contrôlée par le client ou en cas de procédure d’urgence à la suite d’un événement de sécurité zero-day, par exemple.
- Destinée à préserver les données client, la machine virtuelle présente un léger risque de blocage ou de redémarrage.
- Le risque de perte de données sur le stockage local Lsv2 n’étant pas à écarter, il est recommandé de sauvegarder les données avant l’événement.

Pour les événements de maintenance à venir, utilisez le processus de maintenance contrôlée afin de sélectionner le moment le plus opportun pour effectuer la mise à jour. Avant l’événement, vous pouvez sauvegarder vos données sur un stockage premium. À l’issue de l’événement de maintenance, vous pouvez renvoyer vos données vers le stockage NVMe local des machines virtuelles Lsv2 actualisées.

Voici quelques scénarios où les données sont conservées sur les disques NVMe locaux :

- La machine virtuelle est en cours d’exécution et saine.
- La machine virtuelle est redémarrée sur place (par vous ou Azure).
- La machine virtuelle est mise en pause (arrêtée sans désallocation).
- La majorité des opérations de maintenance planifiée.

Voici quelques scénarios où les données du client sont effacées par mesure de sécurité :

- La machine virtuelle est redéployée, arrêtée (désallouée) ou supprimée (par vous).
- L’état de la machine virtuelle devient non sain et le service doit être réparé sur un autre nœud en raison d’un problème matériel.
- Nombre limité d’opérations de maintenance planifiée qui nécessitent la réallocation de la machine virtuelle vers un autre hôte pour la maintenance.

Pour en savoir plus sur les options de sauvegarde des données sur le stockage local, consultez [Sauvegarde et récupération d’urgence pour les disques IaaS Azure](../backup-and-disaster-recovery-for-azure-iaas-disks.md).

## <a name="frequently-asked-questions"></a>Forum aux questions

* **Comment lancer le déploiement de machines virtuelles de la série Lsv2 ?**  
   Comme n’importe quelle autre machine virtuelle, utilisez le [portail](quick-create-portal.md), [Azure CLI](quick-create-cli.md) ou [PowerShell](quick-create-powershell.md) pour créer une machine virtuelle.

* **La défaillance d’un disque NVMe unique peut-elle entraîner la défaillance de toutes les machines virtuelles présentes sur l’hôte ?**  
   Si une défaillance de disque est détectée sur le nœud matériel, le matériel est à l’état d’échec. Quand cela se produit, toutes les machines virtuelles du nœud sont automatiquement désallouées et déplacées vers un nœud sain. Pour les machines virtuelles de la série Lsv2, cela signifie que les données du client qui se trouvaient sur le nœud défaillant sont effacées de manière sécurisée et doivent être recréées par le client sur le nouveau nœud. Comme indiqué auparavant, tant que la migration dynamique ne sera pas disponible sur la série Lsv2, les données qui se trouvent sur le nœud défaillant seront préalablement déplacées avec les machines virtuelles pendant leur transfert sur un autre nœud.

* **Dois-je apporter des modifications à rq_affinity pour bénéficier de meilleures performances ?**  
   Le paramètre rq_affinity a une incidence limitée quand le nombre maximal absolu d’opérations d’entrée/sortie par seconde (IOPS) est utilisé. Une fois que tout le reste fonctionne correctement, essayez de définir rq_affinity sur 0 pour voir si cela apporte quelque chose.

* **Ai-je besoin de modifier les paramètres blk_mq ?**  
   RHEL/CentOS 7.x utilise automatiquement blk-mq pour les appareils NVMe. Il n’est pas utile de modifier la configuration ou des paramètres. Le paramètre scsi_mod.use_blk_mq est destiné uniquement à SCSI et était utilisé dans la version préliminaire de Lsv2, car les appareils NVMe étaient présentés dans les machines virtuelles invitées comme des appareils SCSI. Comme les appareils NVMe se présentent désormais en tant qu’appareils NVMe, le paramètre blk-mq SCSI n’a donc aucun intérêt.

* **Ai-je besoin de modifier « fio » ?**  
   Pour obtenir le maximum d’IOPS avec un outil de mesure de performances comme « fio » dans les tailles de machine virtuelle L64v2 et L80v2, définissez « rq_affinity » sur 0 sur chaque appareil NVMe.  Par exemple, cette ligne de commande définit « rq_affinity » sur zéro pour les 10 appareils NVMe d’une machine virtuelle L80v2 :

   ```console
   for i in `seq 0 9`; do echo 0 >/sys/block/nvme${i}n1/queue/rq_affinity; done
   ```

   De même, notez que les meilleures performances sont obtenues quand les E/S se produisent directement au niveau de chaque appareil NVMe brut sans aucun partitionnement, aucun système de fichiers, aucune configuration RAID 0, etc. Avant de démarrer une session de test, vérifiez que la configuration se trouve dans un état nouveau/propre connu en exécutant `blkdiscard` sur chaque appareil NVMe.
   
## <a name="next-steps"></a>Étapes suivantes

* Consultez les spécifications pour toutes les [machines virtuelles à stockage optimisé](../sizes-storage.md) sur Azure
