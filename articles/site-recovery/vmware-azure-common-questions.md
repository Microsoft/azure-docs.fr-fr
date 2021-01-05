---
title: Questions courantes sur la récupération d’urgence pour VMware avec Azure Site Recovery
description: Obtenez des réponses à des questions courantes sur la récupération d’urgence de machines virtuelles VMware locales sur Azure à l’aide d’Azure Site Recovery.
ms.date: 11/14/2019
ms.topic: conceptual
ms.openlocfilehash: 8f292e7f624b80e8e13514a714c5759d88fbe57c
ms.sourcegitcommit: 8a1ba1ebc76635b643b6634cc64e137f74a1e4da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/09/2020
ms.locfileid: "94379988"
---
# <a name="common-questions-about-vmware-to-azure-replication"></a>Questions courantes sur la réplication de VMware vers Azure

Cet article répond aux questions courantes que vous pouvez vous poser lorsque vous déployez la récupération d’urgence de machines virtuelles VMware locales sur Azure.

## <a name="general"></a>Général

### <a name="what-do-i-need-for-vmware-vm-disaster-recovery"></a>De quoi ai-je besoin pour la récupération d’urgence d’une machine virtuelle VMware ?

[En savoir plus sur les composants impliqués](vmware-azure-architecture.md) dans la récupération d’urgence de machines virtuelles VMware.

### <a name="can-i-use-site-recovery-to-migrate-vmware-vms-to-azure"></a>Puis-je utiliser Site Recovery pour migrer des machines virtuelles VMware vers Azure ?

Oui. En plus d’utiliser Site Recovery pour configurer une récupération d’urgence complète pour des machines virtuelles VMware, vous pouvez également utiliser Site Recovery pour migrer des machines virtuelles VMware locales vers Azure. Dans ce scénario, vous répliquez les machines virtuelles VMware locales vers le Stockage Azure. Ensuite, vous effectuez un basculement du site local vers Azure. Après le basculement, vos applications et charges de travail sont disponibles et s’exécutent sur les machines virtuelles Azure. Le processus est similaire à la configuration d’une récupération d’urgence complète, à ceci près que dans une migration vous ne pouvez pas effectuer une restauration automatique à partir d’Azure.

### <a name="does-my-azure-account-need-permissions-to-create-vms"></a>Est-ce que mon compte Azure a besoin d’autorisations pour créer des machines virtuelles ?

Si vous êtes un administrateur d’abonnement, vous disposez des autorisations de réplication nécessaires. Si vous n’êtes pas un administrateur, vous avez besoin d’autorisations pour effectuer les actions suivantes :

- Créer une machine virtuelle Azure dans le groupe de ressources et le réseau virtuel que vous spécifiez lors de la configuration de Site Recovery.
- Écrire dans le compte de stockage sélectionné ou le disque managé en fonction de votre configuration.

[En savoir plus](site-recovery-role-based-linked-access-control.md#permissions-required-to-enable-replication-for-new-virtual-machines) sur les autorisations requises.

### <a name="what-applications-can-i-replicate"></a>Quelles applications puis-je répliquer ?

Vous pouvez répliquer toute application ou charge de travail en cours d’exécution sur une machine virtuelle VMware qui remplit les [conditions requises de réplication](vmware-physical-azure-support-matrix.md#replicated-machines).

- Site Recovery prend en charge la réplication compatible avec les applications afin qu’elles puissent faire l’objet d’un basculement et être restaurées dans un état intelligent.
- Site Recovery s’intègre à des applications Microsoft, telles que SharePoint, Exchange, Dynamics, SQL Server et Active Directory. Il travaille en étroite collaboration avec les principaux fournisseurs, notamment Oracle, SAP, IBM et Red Hat.

[En savoir plus](site-recovery-workload.md) sur la protection des charges de travail.

### <a name="can-i-use-a-guest-os-server-license-on-azure"></a>Puis-je utiliser une licence serveur de système d’exploitation invité sur Azure ?

Oui, les clients Microsoft Software Assurance peuvent utiliser [Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-benefit/) afin de réduire les coûts de licences pour les machines Windows Server qui sont migrées vers Azure ou qui utilisent Azure pour la récupération d’urgence.

## <a name="security"></a>Sécurité

### <a name="what-access-to-vmware-servers-does-site-recovery-need"></a>Quel est l’accès aux serveurs VMware dont Site Recovery a besoin ?

Site Recovery doit pouvoir accéder aux serveurs VMware pour :

- Configurer une machine virtuelle VMware exécutant le serveur de configuration Site Recovery.
- Détecter automatiquement les machines virtuelles pour la réplication.

### <a name="what-access-to-vmware-vms-does-site-recovery-need"></a>Quel est l’accès aux machines virtuelles VMware dont Site Recovery a besoin ?

- Pour effectuer la réplication, le service Mobilité Site Recovery doit être installé et exécuté sur une machine virtuelle VMware. Vous pouvez déployer l’outil manuellement ou vous pouvez spécifier que Site Recovery effectue une installation Push du service lorsque vous activez la réplication pour une machine virtuelle.
- Lors de la réplication, les machines virtuelles communiquent avec Site Recovery comme suit :
    - Les machines virtuelles communiquent avec le serveur de configuration sur le port HTTPS 443 pour la gestion de la réplication.
    - Les machines virtuelles envoient les données de réplication au serveur de processus sur le port HTTPS 9443. (Ce paramètre peut être modifié.)
    - Si vous activez la cohérence multimachine virtuelle, les machines virtuelles communiquent entre elles sur le port 20004.

### <a name="is-replication-data-sent-to-site-recovery"></a>Est-ce que les données de réplication sont envoyées à Site Recovery ?

Non, Site Recovery n’intercepte pas les données répliquées et n’a pas d’informations sur les éléments exécutés sur vos machines virtuelles. Les données de réplication sont échangées entre les hyperviseurs VMware et le Stockage Azure. Site Recovery n’a aucun moyen d’intercepter ces données. Seules les métadonnées nécessaires pour coordonner la réplication et le basculement sont envoyées au service Site Recovery.

Site Recovery est certifié pour ISO 27001:2013 et 27018, HIPAA et DPA. Les évaluations SOC2 et JAB FedRAMP sont en cours.

## <a name="pricing"></a>Tarifs

### <a name="how-do-i-calculate-approximate-charges-for-vmware-disaster-recovery"></a>Comment puis-je calculer les frais approximatifs de la récupération d’urgence VMware ?

Utilisez la [calculatrice de prix](https://aka.ms/asr_pricing_calculator) pour estimer les coûts d’utilisation de Site Recovery.

Pour obtenir une estimation détaillée des coûts, exécutez l’outil Planificateur de déploiement pour [VMware](./site-recovery-deployment-planner.md) et utilisez le [rapport d’estimation des coûts](./site-recovery-vmware-deployment-planner-cost-estimation.md).

### <a name="is-there-any-difference-in-cost-between-replicating-to-storage-or-directly-to-managed-disks"></a>Existe-t-il une différence de coût entre la réplication vers le stockage ou directement vers des disques managés ?

Les disques managés sont facturés légèrement différemment des comptes de stockage. [En savoir plus](https://azure.microsoft.com/pricing/details/managed-disks/) sur la tarification de disque managé.

### <a name="is-there-any-difference-in-cost-when-replicating-to-general-purpose-v2-storage-account"></a>Existe-t-il une différence de coût en ce qui concerne la réplication vers le compte de stockage à usage général v2 ?

Vous verrez généralement une augmentation du coût des transactions effectuées sur les comptes de stockage GPv2 dans la mesure où Azure Site Recovery comporte un nombre important de transactions. [En savoir plus](../storage/common/storage-account-upgrade.md#pricing-and-billing) pour estimer les modifications.

## <a name="mobility-service"></a>Service Mobilité

### <a name="where-can-i-find-the-mobility-service-installers"></a>Où puis-je trouver les programmes d’installation du service Mobilité ?

Les programmes d’installation se trouvent dans le dossier %ProgramData%\ASR\home\svsystems\pushinstallsvc\repository sur le serveur de configuration.

## <a name="how-do-i-install-the-mobility-service"></a>Comment installer le service Mobilité ?

Sur chaque machine virtuelle que vous souhaitez répliquer, installez le service selon l’une des méthodes suivantes :

- [Installation Push](vmware-physical-mobility-service-overview.md#push-installation)
- [Installation manuelle](vmware-physical-mobility-service-overview.md#install-the-mobility-service-using-ui) à partir de l’interface utilisateur ou de PowerShell
- Déploiement à l’aide d’un outil de déploiement tel que [Configuration Manager](vmware-azure-mobility-install-configuration-mgr.md)

## <a name="managed-disks"></a>Disques managés

### <a name="where-does-site-recovery-replicate-data-to"></a>Où Site Recovery réplique-t-il les données ?

Site Recovery réplique des machines virtuelles VMware locales et des serveurs physiques sur des disques managés dans Azure.

- Le serveur de processus Site Recovery écrit les journaux de réplication sur un compte de stockage de cache dans la région cible.
- Ces journaux sont utilisés pour la création de points de récupération sur des disques managés Azure qui présentent le préfixe **asrseeddisk**.
- En cas de basculement, le point de récupération que vous sélectionnez sert à créer un nouveau disque managé cible. Ce disque managé est lié à la machine virtuelle dans Azure.
- Les machines virtuelles qui ont été précédemment répliquées vers un compte de stockage (avant mars 2019) ne sont pas affectées.

### <a name="can-i-replicate-new-machines-to-storage-accounts"></a>Puis-je répliquer de nouvelles machines vers des comptes de stockage ?

Non. À compter de mars 2019, dans le portail Azure, vous ne pouvez répliquer que sur des disques managés Azure.

La réplication de nouvelles machines virtuelles vers un compte de stockage n’est possible qu’à l’aide de PowerShell ([module Az.RecoveryServices version 1.4.5](https://www.powershellgallery.com/packages/Az.RecoveryServices/1.4.5)) ou de l’API REST (version 2018-01-10 ou 2016-08-10). [Découvrez comment](./vmware-azure-disaster-recovery-powershell.md) configurer la réplication à l’aide des commandes PowerShell.

### <a name="what-are-the-benefits-of-replicating-to-managed-disks"></a>Quels sont les avantages de la réplication vers des disques managés ?

[Découvrez comment](https://azure.microsoft.com/blog/simplify-disaster-recovery-with-managed-disks-for-vmware-and-physical-servers/) Site Recovery simplifie la récupération d’urgence avec des disques managés.

### <a name="can-i-change-the-managed-disk-type-after-a-machine-is-protected"></a>Puis-je modifier le type de disque managé une fois qu’une machine est protégée ?

Oui, vous pouvez facilement [modifier le type de disque managé](../virtual-machines/windows/convert-disk-storage.md) pour les réplications en cours. Avant de modifier le type, vérifiez qu’aucune URL de signature d’accès partagé n’est générée sur le disque managé :

1. Accédez à la ressource **Disque managé** sur le portail Azure et vérifiez qu’une bannière d’URL de signature d’accès partagé apparaît dans le panneau **Vue d’ensemble**.
1. Si la bannière est présente, sélectionnez-la pour annuler l’exportation en cours.
1. Modifiez le type du disque dans les minutes qui suivent. Si vous modifiez le type de disque managé, attendez que de nouveaux points de récupération soient générés par Azure Site Recovery.
1. Utilisez les nouveaux points de récupération pour un basculement de test ou un basculement futur.

### <a name="can-i-switch-replication-from-managed-disks-to-unmanaged-disks"></a>Puis-je basculer la réplication de disques managés vers des disques non managés ?

Non. Le basculement entre des disques managés et non managés n’est pas pris en charge.

## <a name="replication"></a>Réplication

### <a name="what-are-the-replicated-vm-requirements"></a>Quelle est la configuration requise d’une machine virtuelle répliquée ?

[En savoir plus](vmware-physical-azure-support-matrix.md#replicated-machines) sur les exigences de prise en charge pour les machines virtuelles et serveurs physiques VMware.

### <a name="how-often-can-i-replicate-to-azure"></a>À quelle fréquence puis-je répliquer vers Azure ?

La réplication est continue quand il s’agit de la réplication des machines virtuelles VMware vers Azure.

### <a name="can-i-extend-replication"></a>Puis-je étendre la réplication ?

La réplication étendue ou chaînée n’est pas prise en charge. Demandez cette fonctionnalité dans le [forum de commentaires](https://feedback.azure.com/forums/256299-site-recovery/suggestions/6097959).

### <a name="can-i-do-an-offline-initial-replication"></a>Puis-je effectuer une réplication initiale hors connexion ?

La réplication hors connexion n’est pas prise en charge. Demandez cette fonctionnalité dans le [forum de commentaires](https://feedback.azure.com/forums/256299-site-recovery/suggestions/6227386-support-for-offline-replication-data-transfer-from).

### <a name="what-is-asrseeddisk"></a>Qu’est-ce qu’asrseeddisk ?

Pour chaque disque source, les données sont répliquées vers un disque managé dans Azure. Ce disque présente le préfixe **asrseeddisk**. Il stocke la copie du disque source et tous les instantanés de point de récupération.

### <a name="can-i-exclude-disks-from-replication"></a>Puis-je exclure des disques de la réplication ?

Oui, vous pouvez exclure des disques.

### <a name="can-i-replicate-vms-that-have-dynamic-disks"></a>Puis-je répliquer des machines virtuelles comprenant des disques dynamiques ?

Les disques dynamiques peuvent être répliqués. Le disque du système d’exploitation doit être un disque de base.

### <a name="if-i-use-replication-groups-for-multi-vm-consistency-can-i-add-a-new-vm-to-an-existing-replication-group"></a>Si j’utilise des groupes de réplication pour la cohérence entre plusieurs machines virtuelles, puis-je ajouter une nouvelle machine virtuelle à un groupe de réplication existant ?

Oui, vous pouvez ajouter de nouvelles machines virtuelles à un groupe de réplication existant lorsque vous activez la réplication. Toutefois :

- Vous ne pouvez pas ajouter une machine virtuelle à un groupe de réplication existant après que la réplication a commencé.
- Vous ne pouvez pas créer un groupe de réplication pour des machines virtuelles existantes.

### <a name="can-i-modify-vms-that-are-replicating-by-adding-or-resizing-disks"></a>Puis-je modifier les machines virtuelles répliquées via l’ajout ou le redimensionnement de disques ?

Pour la réplication VMware vers Azure, vous pouvez modifier la taille de disque des machines virtuelles source. Si vous souhaitez ajouter de nouveaux disques, vous devez ajouter le disque et réactiver la protection de la machine virtuelle.

### <a name="can-i-migrate-on-premises-machines-to-a-new-vcenter-server-without-impacting-ongoing-replication"></a>Puis-je migrer des machines locales vers un nouveau vCenter Server sans que cela compromette la réplication en cours ?

Reportez-vous à nos [conseils](vmware-azure-manage-vcenter.md#migrate-all-vms-to-a-new-server) pour migrer des machines vers un nouveau vCenter

### <a name="can-i-replicate-to-a-cache-or-target-storage-account-that-has-a-virtual-network-with-azure-firewalls-configured-on-it"></a>Puis-je répliquer vers un compte de stockage de cache ou cible dans lequel un réseau virtuel (avec des pare-feux Azure) est configuré ?

Non, Site Recovery ne prend pas en charge la réplication vers un Stockage Azure sur des réseaux virtuels.

### <a name="what-is-the-frequency-of-generation-of-crash-consistent-recovery-points"></a>Quelle est la fréquence de génération de points de récupération cohérents en cas d’incident ?

Site Recovery génère des points de récupération cohérents en cas d’incident toutes les cinq minutes.

## <a name="component-upgrade"></a>Mise à niveau de composant

### <a name="my-version-of-the-mobility-services-agent-or-configuration-server-is-old-and-my-upgrade-failed-what-do-i-do"></a>Ma version de l’agent de services de mobilité ou du serveur de configuration est ancienne et ma mise à niveau a échoué. Que faire ?

Site Recovery suit le modèle de prise en charge N-4. [En savoir plus](./service-updates-how-to.md#support-statement-for-azure-site-recovery) sur la mise à niveau à partir de versions très anciennes.

### <a name="where-can-i-find-the-release-notes-and-update-rollups-for-azure-site-recovery"></a>Où puis-je trouver les notes de publication et correctifs cumulatifs pour Azure Site Recovery ?

[Apprenez-en davantage sur les nouvelles mises à jour](site-recovery-whats-new.md) et [obtenez des informations sur les correctifs cumulatifs](service-updates-how-to.md).

### <a name="where-can-i-find-upgrade-information-for-disaster-recovery-to-azure"></a>Où puis-je trouver des informations de mise à niveau pour la récupération d’urgence vers Azure ?

[Apprenez-en davantage sur la mise à niveau](./service-updates-how-to.md#vmware-vmphysical-server-disaster-recovery-to-azure).

## <a name="do-i-need-to-reboot-source-machines-for-each-upgrade"></a>Dois-je redémarrer les machines sources pour chaque mise à niveau ?

Un redémarrage est recommandé mais non obligatoire pour chaque mise à niveau. [Plus d’informations](./service-updates-how-to.md#reboot-after-mobility-service-upgrade)

## <a name="configuration-server"></a>Serveur de configuration

### <a name="what-does-the-configuration-server-do"></a>Quel fait le serveur de configuration ?

Le serveur de configuration exécute les composants Site Recovery en local, notamment :

- Le serveur de configuration lui-même. Le serveur coordonne la communication entre les composants locaux et Azure, et gère la réplication des données.
- Le serveur de traitement, qui fait office de passerelle de réplication. Ce serveur :
    1. Reçoit les données de réplication.
    2. Optimise les données à l’aide de la mise en cache, la compression et le chiffrement.
    3. Envoie les données vers le Stockage Azure.
  Le serveur de processus installe (type push) le Mobility Service sur les machines virtuelles et effectue la détection automatique sur les machines virtuelles VMware locales.
- Le serveur cible maître, qui gère les données de réplication pendant la restauration automatique à partir d’Azure.

[Découvrez-en plus](vmware-azure-architecture.md) sur les composants et les processus du serveur de configuration.

### <a name="where-do-i-set-up-the-configuration-server"></a>Où configurer le serveur de configuration ?

Vous avez besoin d’une seule machine virtuelle VMware locale hautement disponible pour le serveur de configuration. Pour la récupération d’urgence du serveur physique, installez le serveur de configuration sur une machine physique.

### <a name="what-do-i-need-for-the-configuration-server"></a>De quoi ai-je besoin pour le serveur de configuration ?

Examinez les [conditions préalables](vmware-azure-deploy-configuration-server.md#prerequisites).

### <a name="can-i-manually-set-up-the-configuration-server-instead-of-using-a-template"></a>Puis-je configurer manuellement le serveur de configuration au lieu d’utiliser un modèle ?

Nous vous recommandons de [créer la machine virtuelle du serveur de configuration](vmware-azure-deploy-configuration-server.md) à l’aide de la version la plus récente du modèle OVF (Open Virtualization Format). Si vous ne pouvez pas utiliser le modèle (par exemple, si vous n’avez pas accès au serveur VMware), [téléchargez](physical-azure-set-up-source.md) le fichier d’installation à partir du portail et configurez le serveur de configuration.

### <a name="can-a-configuration-server-replicate-to-more-than-one-region"></a>Un serveur de configuration peut-il répliquer vers plusieurs régions ?

Non. Pour répliquer vers plusieurs régions, vous avez besoin d’un serveur de configuration dans chaque région.

### <a name="can-i-host-a-configuration-server-in-azure"></a>Puis-je héberger un serveur de configuration dans Azure ?

Bien que ce soit possible, la machine virtuelle Azure exécutant le serveur de configuration doit communiquer avec les machines virtuelles et votre infrastructure VMware locales. Cette communication ajoute de la latence et a un impact sur la réplication en cours.

### <a name="how-do-i-update-the-configuration-server"></a>Comment mettre à jour le serveur de configuration ?

[Découvrez](vmware-azure-manage-configuration-server.md#upgrade-the-configuration-server) comment mettre à jour le serveur de configuration.

- Les informations de mise à jour les plus récentes sont disponibles dans la [page relatives aux mises à jour d’Azure](https://azure.microsoft.com/updates/?product=site-recovery).
- Vous pouvez télécharger la dernière version à partir du portail. Vous pouvez également télécharger la dernière version du serveur de configuration directement à partir du [Centre de téléchargement Microsoft](https://aka.ms/asrconfigurationserver).
- Si votre version est antérieure de plus de quatre versions à la version actuelle, consultez la [déclaration de support](./service-updates-how-to.md#support-statement-for-azure-site-recovery) pour obtenir des instructions de mise à niveau.

### <a name="should-i-back-up-the-configuration-server"></a>Dois-je sauvegarder le serveur de configuration ?

Nous vous recommandons d’effectuer des sauvegardes planifiées régulières du serveur de configuration.

- Pour que la restauration automatique réussisse, la machine virtuelle faisant l’objet d’une restauration doit exister dans la base de données du serveur de configuration.
- Le serveur de configuration doit être en cours d’exécution et dans un état connecté.
- [En savoir plus](vmware-azure-manage-configuration-server.md) sur les tâches courantes d’administration du serveur de configuration.

### <a name="when-im-setting-up-the-configuration-server-can-i-download-and-install-mysql-manually"></a>Lorsque je configure le serveur de configuration, puis-je télécharger et installer MySQL manuellement ?

Oui. Téléchargez MySQL et placez-le dans le dossier C:\Temp\ASRSetup. Puis installez-le manuellement. Lorsque vous configurez la machine virtuelle du serveur de configuration et acceptez les termes du contrat, MySQL est répertorié comme **Déjà installé** dans **Télécharger et installer**.

### <a name="can-i-avoid-downloading-mysql-but-let-site-recovery-install-it"></a>Puis-je ne pas télécharger MySQL et laisser Site Recovery l’installer ?

Oui. Téléchargez le programme d’installation MySQL et placez-le dans le dossier C:\Temp\ASRSetup. Lorsque vous configurez la machine virtuelle du serveur de configuration, acceptez les conditions et sélectionnez **Télécharger et installer**. Le portail utilise le programme d’installation que vous avez ajouté pour installer MySQL.

### <a name="can-i-use-the-configuration-server-vm-for-anything-else"></a>Puis-je utiliser la machine virtuelle du serveur de configuration à d’autres fins ?

Non. N’utilisez la machine virtuelle que pour le serveur de configuration.

### <a name="can-i-clone-a-configuration-server-and-use-it-for-orchestration"></a>Puis-je cloner un serveur de configuration et l’utiliser pour l’orchestration ?

Non. Configurez un nouveau serveur de configuration pour éviter les problèmes d’inscription.

### <a name="can-i-change-the-vault-in-which-the-configuration-server-is-registered"></a>Puis-je modifier le coffre dans lequel le serveur de configuration est inscrit ?

Non. Une fois le coffre associé au serveur de configuration, il n’est plus modifiable. [Découvrez](vmware-azure-manage-configuration-server.md#register-a-configuration-server-with-a-different-vault) la procédure d’inscription d’un serveur de configuration auprès d’un autre coffre.

### <a name="can-i-use-the-same-configuration-server-for-disaster-recovery-of-both-vmware-vms-and-physical-servers"></a>Puis-je utiliser le même serveur de configuration à des fins de récupération d’urgence pour les machines virtuelles VMware et les serveurs physiques ?

Oui, mais notez qu’une machine physique peut uniquement être restaurée vers une machine virtuelle VMware.

### <a name="where-can-i-download-the-passphrase-for-the-configuration-server"></a>Où puis-je télécharger la phrase secrète du serveur de configuration ?

[Découvrez](vmware-azure-manage-configuration-server.md#generate-configuration-server-passphrase) comment télécharger la phrase secrète.

### <a name="where-can-i-download-vault-registration-keys"></a>Où puis-je télécharger les clés d’inscription du coffre ?

Dans le coffre Recovery Services, sélectionnez **Serveurs de configuration** dans **Infrastructure Site Recovery** > **Gérer**. Puis, dans **Serveurs**, sélectionnez **Télécharger une clé d’inscription** pour télécharger le fichier d’informations d’identification du coffre.

### <a name="can-a-single-configuration-server-be-used-to-protect-multiple-vcenter-instances"></a>Est-il possible d’utiliser un seul serveur de configuration pour protéger plusieurs instances vCenter ?

Oui, un seul serveur de configuration peut protéger des machines virtuelles sur plusieurs instances vCenter.  Il n’existe pas de limite pour le nombre d’instances de vCenter pouvant être ajoutées au serveur de configuration. Toutefois, des limites s’appliquent quant au nombre de machines virtuelles qu’un seul serveur de configuration peut protéger.

### <a name="can-a-single-configuration-server-protect-multiple-clusters-within-vcenter"></a>Un seul serveur de configuration peut-il protéger plusieurs clusters dans vCenter ?

Oui, Azure Site Recovery peut protéger des machines virtuelles sur différents clusters.

## <a name="process-server"></a>Serveur de traitement

### <a name="why-am-i-unable-to-select-the-process-server-when-i-enable-replication"></a>Pourquoi ne puis-je pas sélectionner le serveur de traitement lorsque j’active la réplication ?

Les mises à jour dans les versions 9.24 et ultérieures affichent désormais l’[intégrité du serveur de traitement lorsque vous activez la réplication](vmware-azure-enable-replication.md#enable-replication). Cette fonctionnalité permet d’éviter une limitation du serveur de traitement et de réduire l’utilisation de serveurs de traitement non sains.

### <a name="how-do-i-update-the-process-server-to-version-924-or-later-for-accurate-health-information"></a>Comment puis-je mettre à jour le serveur de traitement vers la version 9.24 ou ultérieure pour obtenir des informations d’intégrité précises ?

Depuis la [version 9.24](service-updates-how-to.md#links-to-currently-supported-update-rollups), davantage d’alertes ont été ajoutées pour indiquer l’intégrité du serveur de traitement. [Mettez à jour vos composants de Site Recovery vers la version 9.24 ou ultérieure](service-updates-how-to.md#links-to-currently-supported-update-rollups) pour que toutes les alertes soient générées.

### <a name="how-can-i-ensure-high-availability-of-the-process-server"></a>Comment puis-je garantir une haute disponibilité du serveur de traitement ?

En configurant plusieurs serveurs de processus, la conception offre la flexibilité nécessaire pour déplacer des machines protégées d’un serveur de traitement non sain vers un serveur de traitement opérationnel. Le déplacement d’une machine d’un serveur de traitement à un autre doit être initié explicitement/manuellement via les étapes définies ici : [Déplacement de machines virtuelles entre serveurs de traitement](vmware-azure-manage-process-server.md#move-vms-to-balance-the-process-server-load).

## <a name="failover-and-failback"></a>Basculement et restauration automatique

### <a name="can-i-use-the-on-premises-process-server-for-failback"></a>Puis-je utiliser le serveur de traitement local pour la restauration automatique ?

Nous vous recommandons vivement de créer un serveur de traitement dans Azure à des fins de restauration automatique, afin d’éviter les latences de transfert de données. En outre, si vous avez séparé le réseau de machines virtuelles source du réseau accessible sur Azure dans le serveur de configuration, vous devez utiliser le serveur de traitement créé dans Azure pour la restauration automatique.

### <a name="can-i-keep-the-ip-address-on-failover"></a>Puis-je conserver l’adresse IP lors du basculement ?

Oui, vous pouvez conserver l’adresse IP lors du basculement. Veillez à spécifier l’adresse IP cible dans les paramètres **Calcul et réseau** pour la machine virtuelle avant le basculement. Arrêtez également les machines au moment du basculement afin d’éviter des conflits d’adresses IP pendant la restauration automatique.

### <a name="can-i-change-the-target-vm-size-or-vm-type-before-failover"></a>Puis-je modifier la taille de machine virtuelle cible ou le type de machine virtuelle avant le basculement ?

Oui, vous pouvez modifier le type ou la taille de la machine virtuelle à tout moment avant le basculement. Dans le portail, utilisez les paramètres **Calcul et réseau** pour la machine virtuelle répliquée.

### <a name="how-far-back-can-i-recover"></a>Jusqu’à quand peut remonter la récupération ?

Le point de récupération le plus ancien que vous pouvez utiliser pour VMware vers Azure est de 72 heures.

### <a name="how-do-i-access-azure-vms-after-failover"></a>Comment accéder aux machines virtuelles Azure après un basculement ?

Après un basculement, vous pouvez accéder aux machines virtuelles Azure via une connexion Internet sécurisée, via un VPN site à site ou via Azure ExpressRoute. Pour vous connecter, vous devez préparer plusieurs choses. [Plus d’informations](site-recovery-test-failover-to-azure.md#prepare-to-connect-to-azure-vms-after-failover)

### <a name="is-failed-over-data-resilient"></a>Est-ce que les données ayant fait l’objet d’un basculement sont résilientes ?

Azure est conçu pour la résilience. Site Recovery est prévu pour assurer le basculement vers un centre de données Azure secondaire, tel que requis dans le contrat de niveau de service (SLA) Azure. En cas de basculement, nous nous assurons que vos métadonnées et vos coffres restent dans la même région géographique que vous avez choisie pour votre coffre.

### <a name="is-failover-automatic"></a>Le basculement est-il automatique ?

Le [basculement](site-recovery-failover.md) n’est pas automatique. Vous lancez un basculement d’une seule sélection dans le portail ou en utilisant [PowerShell](/powershell/module/az.recoveryservices).

### <a name="can-i-fail-back-to-a-different-location"></a>Puis-je effectuer la restauration à un autre emplacement ?

Oui. Si vous effectuez le basculement vers Azure, vous pouvez effectuer la restauration à un autre emplacement si celui d’origine n’est pas disponible. [Plus d’informations](concepts-types-of-failback.md#alternate-location-recovery-alr)

### <a name="why-do-i-need-a-vpn-or-expressroute-with-private-peering-to-fail-back"></a>Pourquoi ai-je besoin d’un VPN ou d’ExpressRoute avec peering privé pour la restauration ?

Lorsque vous effectuez la restauration à partir d’Azure, les données d’Azure sont recopiées vers votre machine virtuelle locale et un accès privé est requis.


## <a name="automation-and-scripting"></a>Automatisation et scripts

### <a name="can-i-set-up-replication-with-scripting"></a>Puis-je configurer la réplication avec des scripts ?

Oui. Vous pouvez automatiser les flux de travail Site Recovery à l’aide de l’API Rest, de PowerShell ou du Kit de développement logiciel (SDK) Azure. [Plus d’informations](vmware-azure-disaster-recovery-powershell.md)

## <a name="performance-and-capacity"></a>Performances et capacité

### <a name="can-i-throttle-replication-bandwidth"></a>Est-il possible de limiter la bande passante de réplication ?

Oui. [Plus d’informations](site-recovery-plan-capacity-vmware.md)

## <a name="next-steps"></a>Étapes suivantes

- [Examiner](vmware-physical-azure-support-matrix.md) les conditions de prise en charge.
- [Configurer](vmware-azure-tutorial.md) la réplication VMware vers Azure.