---
title: Préparer des machines pour la migration avec Azure Migrate
description: Découvrez comment préparer des machines locales pour la migration avec Azure Migrate.
author: anvar-ms
ms.author: anvar
ms.manager: bsiva
ms.topic: how-to
ms.date: 06/08/2020
ms.openlocfilehash: 979f40e13aab71f02a316e4ddf60306170166845
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96753924"
---
# <a name="prepare-on-premises-machines-for-migration-to-azure"></a>Préparer des ordinateurs locaux à une migration vers Azure

Cet article explique comment préparer des machines locales avant d’effectuer leur migration vers Azure à l’aide de l’outil [Azure Migrate : Migration de serveur](migrate-services-overview.md#azure-migrate-server-migration-tool).

Dans cet article, vous découvrirez comment :
> [!div class="checklist"]
> * Examiner les limitations de la migration.
> * Sélectionner la méthode de migration des machines virtuelles VMware.
> * Vérifier la configuration requise de l’hyperviseur et du système d’exploitation pour les machines que vous souhaitez migrer.
> * Examiner l’accès aux URL et aux ports pour les ordinateurs que vous voulez migrer.
> * Passer en revue les modifications que vous devrez peut-être apporter avant de commencer la migration.
> * Vérifier la configuration requise des machines virtuelles Azure pour la migration
> * Préparer les ordinateurs de telle sorte que vous puissiez vous connecter aux machines virtuelles Azure après la migration.



## <a name="verify-migration-limitations"></a>Vérifier les limitations de la migration

Le tableau récapitule les limites qui s’appliquent à la détection, à l’évaluation et à la migration pour Azure Migrate. Nous vous recommandons d’évaluer les machines avant leur migration, mais cela n’est pas une obligation.

**Scénario** | **Projet** | **Détection/évaluation** | **Migration**
--- | --- | --- | ---
**Machines virtuelles VMware** | Détectez et évaluez jusqu’à 35 000 machines virtuelles dans un même projet Azure Migrate. | Détectez jusqu’à 10 000 machines virtuelles VMware avec une seule [appliance Azure Migrate](common-questions-appliance.md) pour VMware. | **Migration sans agent** : vous pouvez répliquer simultanément un maximum de 300 machines virtuelles. Pour de meilleures performances, nous vous recommandons de créer plusieurs lots de machines virtuelles s’il y a plus de 50 machines au total.<br/><br/> **Migration basée sur un agent** : vous pouvez faire un [scale-out](./agent-based-migration-architecture.md#performance-and-scaling) de l’[appliance de réplication](migrate-replication-appliance.md) pour répliquer un grand nombre de machines virtuelles.<br/><br/> Dans le portail, vous pouvez sélectionner jusqu’à 10 machines à la fois pour la réplication. Pour répliquer davantage de machines, ajoutez-les par lots de 10.
**Machines virtuelles Hyper-V** | Détectez et évaluez jusqu’à 35 000 machines virtuelles dans un même projet Azure Migrate. | Détectez jusqu’à 5 000 machines virtuelles Hyper-V avec une seule appliance Azure Migrate. | Aucune appliance n’est utilisée pour la migration Hyper-V. Au lieu de cela, le fournisseur de réplication Hyper-V s’exécute sur chaque hôte Hyper-V.<br/><br/> La capacité de réplication dépend de plusieurs facteurs de performance tels que l’évolution des machines virtuelles et la bande passante de chargement pour les données de réplication.<br/><br/> Dans le portail, vous pouvez sélectionner jusqu’à 10 machines à la fois pour la réplication. Pour répliquer davantage de machines, ajoutez-les par lots de 10.
**Machines physiques** | Détectez et évaluez jusqu’à 35 000 machines dans un même projet Azure Migrate. | Détectez jusqu’à 250 serveurs physiques avec une seule appliance Azure Migrate pour les serveurs physiques. | Vous pouvez faire un [scale-out](./agent-based-migration-architecture.md#performance-and-scaling) de l’[appliance de réplication](migrate-replication-appliance.md) pour répliquer un grand nombre de serveurs.<br/><br/> Dans le portail, vous pouvez sélectionner jusqu’à 10 machines à la fois pour la réplication. Pour répliquer davantage de machines, ajoutez-les par lots de 10.

## <a name="select-a-vmware-migration-method"></a>Sélectionner une méthode de migration VMware

Si vous migrez des machines virtuelles VMware vers Azure, [comparez](server-migrate-overview.md#compare-migration-methods) les méthodes de migration sans agent et avec agent afin de déterminer celle qui est appropriée dans votre cas.

## <a name="verify-hypervisor-requirements"></a>Vérifier la configuration requise pour l’hyperviseur

- Vérifiez la configuration requise pour une [migration VMware sans agent](migrate-support-matrix-vmware-migration.md#vmware-requirements-agentless) ou une [migration VMware basée sur un agent](migrate-support-matrix-vmware-migration.md#vmware-requirements-agent-based).
- Vérifiez la configuration requise pour l’[hôte Hyper-V](migrate-support-matrix-hyper-v-migration.md#hyper-v-host-requirements).


## <a name="verify-operating-system-requirements"></a>Vérifier la configuration requise pour le système d’exploitation

Vérifiez les systèmes d’exploitation pris en charge pour la migration :

- Si vous migrez des machines virtuelles VMware ou des machines virtuelles Hyper-V, vérifiez la configuration requise des machines virtuelles VMware pour la migration [sans agent](migrate-support-matrix-vmware-migration.md#vm-requirements-agentless) et la migration [basée sur un agent](migrate-support-matrix-vmware-migration.md#vm-requirements-agent-based), ainsi que la configuration requise des [machines virtuelles Hyper-V](migrate-support-matrix-hyper-v-migration.md#hyper-v-vms).
- Vérifiez quels [systèmes d’exploitation Windows](https://support.microsoft.com/help/2721672/microsoft-server-software-support-for-microsoft-azure-virtual-machines) sont pris en charge dans Azure.
- Vérifiez quelles [distributions Linux](../virtual-machines/linux/endorsed-distros.md) sont prises en charge dans Azure.

## <a name="review-url-and-port-access"></a>Vérifier l’accès aux URL et aux ports

Examinez les URL et les ports utilisés pendant la migration.

**Scénario** | **Détails** |  **URLs** | **Ports**
--- | --- | --- | ---
**Migration VMware sans agent** | Utilise l’[appliance Azure Migrate](migrate-appliance-architecture.md) pour la migration. Rien n’est installé sur les machines virtuelles VMware. | Examinez les [URL](migrate-appliance.md#url-access) de cloud public et du secteur public (Azure Government) qui sont utilisées pour la détection, l’évaluation et la migration avec l’appliance. | [Examinez](migrate-support-matrix-vmware-migration.md#port-requirements-agentless) la configuration requise des ports utilisés pendant la migration sans agent.
**Migration VMware basée sur un agent** | Utilise l’[appliance de réplication](migrate-replication-appliance.md) pour la migration. L’agent du service Mobility est installé sur les machines virtuelles. | Examinez les URL de [cloud public](migrate-replication-appliance.md#url-access) et [Azure Government](migrate-replication-appliance.md#azure-government-url-access) auxquelles l’appliance de réplication doit accéder. | [Examinez](migrate-replication-appliance.md#port-access) les ports utilisés durant la migration basée sur un agent.
**Migration Hyper-V** | Utilise un fournisseur installé sur les hôtes Hyper-V pour la migration. Rien n’est installé sur les machines virtuelles Hyper-V. | Examinez les URL de [cloud public](migrate-support-matrix-hyper-v-migration.md#url-access-public-cloud) et [Azure Government](migrate-support-matrix-hyper-v-migration.md#url-access-azure-government) auxquelles le fournisseur de réplication exécuté sur les hôtes doit accéder. | Le fournisseur de réplication sur l’hôte Hyper-V utilise des connexions sortantes sur le port HTTPS 443 pour envoyer les données de réplication des machines virtuelles.
**Machines physiques** | Utilise l’[appliance de réplication](migrate-replication-appliance.md) pour la migration. L’agent du service Mobility est installé sur les machines physiques. | Examinez les URL de [cloud public](migrate-replication-appliance.md#url-access) et [Azure Government](migrate-replication-appliance.md#azure-government-url-access) auxquelles l’appliance de réplication doit accéder. | [Examinez](migrate-replication-appliance.md#port-access) les ports utilisés durant la migration physique.

## <a name="verify-required-changes-before-migrating"></a>Vérifier les modifications nécessaires avant la migration

Certaines modifications doivent être apportées aux machines virtuelles afin qu’elles puissent être migrées vers Azure.

- Pour certains systèmes d’exploitation, Azure Migrate effectue automatiquement les modifications nécessaires durant le processus de réplication/migration.
- Pour d’autres systèmes d’exploitation, vous devez configurer des paramètres manuellement.
- Il est important d’effectuer cette configuration manuelle avant de commencer la migration. Si vous migrez la machine virtuelle avant d’effectuer la modification, elle risque de ne pas démarrer dans Azure.

Consultez les tableaux pour connaître les modifications à apporter dans votre cas.

### <a name="windows-machines"></a>Machines Windows

Ce tableau récapitule les modifications requises.

**Action** | **VMware (migration sans agent)** | **VMware (migration basée sur un agent)/machines physiques** | **Windows sur Hyper-V** 
--- | --- | --- | ---
**Configurer la stratégie SAN sur OnlineAll**<br/><br/> Ce paramètre garantit que les volumes Windows sur les machines virtuelles Azure utilisent les mêmes affectations de lettre de lecteur que la machine virtuelle locale. | Configuration automatique pour les machines exécutant Windows Server 2008 R2 ou ultérieur.<br/><br/> Configuration manuelle pour les systèmes d’exploitation antérieurs. | Configuration automatique dans la plupart des cas. | Configuration manuelle.
**Installer l’intégration d’invité Hyper-V** | [Installation manuelle](prepare-windows-server-2003-migration.md#install-on-vmware-vms) sur les machines exécutant Windows Server 2003. | [Installation manuelle](prepare-windows-server-2003-migration.md#install-on-vmware-vms) sur les machines exécutant Windows Server 2003. | [Installation manuelle](prepare-windows-server-2003-migration.md#install-on-hyper-v-vms) sur les machines exécutant Windows Server 2003.
**Activer la console série Azure**<br/><br/>[Activez la console](../virtual-machines/troubleshooting/serial-console-windows.md) sur les machines virtuelles Azure pour faciliter le dépannage. Vous n’avez pas besoin de redémarrer la machine virtuelle. La machine virtuelle Azure démarre en utilisant l’image de disque. Le démarrage avec une image de disque équivaut à un redémarrage pour la nouvelle machine virtuelle. | Activation manuelle | Activation manuelle | Activation manuelle
**Se connecter après la migration**<br/><br/> Pour vous connecter après la migration, vous devez effectuer quelques étapes préalablement à la migration. | [Configuration](#prepare-to-connect-to-azure-windows-vms) manuelle. | [Configuration](#prepare-to-connect-to-azure-windows-vms) manuelle. | [Configuration](#prepare-to-connect-to-azure-windows-vms) manuelle.


#### <a name="configure-san-policy"></a>Configurer la stratégie SAN

Par défaut, les machines virtuelles Azure se voient affecter le lecteur D pour une utilisation comme stockage temporaire.

- À la suite de cette affectation de lecteur, tous les autres lecteurs de stockage attachés sont incrémentés d’une lettre.
- Par exemple, si le lecteur de disque de données de votre installation utilise la lettre D pour les installations d’applications, celle-ci est incrémentée pour devenir le lecteur E après la migration de la machine virtuelle vers Azure. 
- Pour empêcher cette affectation automatique et faire en sorte qu’Azure affecte la prochaine lettre de lecteur disponible à son volume temporaire, définissez la stratégie de réseau de zone de stockage (SAN) sur **OnlineAll** :

Configurez ce paramètre manuellement de la façon suivante :

1. Sur l’ordinateur local (pas sur le serveur hôte), ouvrez une invite de commandes avec élévation de privilèges.
2. Entrez **diskpart**.
3. Entrez **SAN**. Si la lettre de lecteur du système d’exploitation invité n’est pas conservée, **Mettre hors connexion tous les disques** ou **Mettre hors connexion les disques partagés** est retourné.
4. À l’invite **DISKPART**, entrez **SAN Policy=OnlineAll**. Ce paramètre est l’assurance que les disques sont mis en ligne et que vous pouvez lire et écrire sur les deux disques.
5. Pendant le test de migration, vous pouvez vérifier que les lettres de lecteurs sont conservées.


### <a name="linux-machines"></a>Machines Linux

Azure Migrate effectue automatiquement ces actions pour les versions suivantes

- Red Hat Enterprise Linux 7.8, 7.7, 7.6, 7.5, 7.4, 7.0, 6.x
- Cent OS 7.7, 7.6, 7.5, 7.4, 6.x
- SUSE Linux Enterprise Server 12 SP1+
- SUSE Linux Enterprise Server 15 SP1
- Ubuntu 19.04, 19.10, 18.04LTS, 16.04LTS, 14.04LTS
- Debian 8, 7
- Oracle Linux 7.7, 7.7-CI

Pour les autres versions, préparez les machines comme cela est résumé dans le tableau.  


**Action** | **Détails** | **Version de Linux**
--- | --- | ---
**Installer Hyper-V Integration Services pour Linux** | Regénérez l’image init Linux pour qu’elle contienne les pilotes Hyper-V nécessaires. La regénération de l’image init est l’assurance que la machine virtuelle démarrera dans Azure. | Ils sont inclus par défaut dans la plupart des nouvelles versions des distributions Linux.<br/><br/> Si ce n’est pas le cas, installez-les manuellement pour toutes les versions autres que celles indiquées ci-dessus.
**Activer la journalisation de la console série Azure** | L’activation de la journalisation de la console vous aide à résoudre les problèmes. Vous n’avez pas besoin de redémarrer la machine virtuelle. La machine virtuelle Azure démarre en utilisant l’image de disque. Le démarrage avec une image de disque équivaut à un redémarrage pour la nouvelle machine virtuelle.<br/><br/> Suivez [ces instructions](../virtual-machines/troubleshooting/serial-console-linux.md) pour l’activer.
**Mettre à jour le fichier de mappage d’appareils** | Mettez à jour le fichier de mappage d’appareils avec les associations entre les volumes et les noms d’appareils de façon à utiliser des identificateurs d’appareils persistants. | Installez-les manuellement pour toutes les versions autres que celles indiquées ci-dessus. (Applicable uniquement dans un scénario VMware basé sur un agent)
**Mettre à jour les entrées fstab** |  Mettez à jour les entrées fstab pour utiliser des identificateurs de volumes persistants.    | Mettez-les à jour manuellement pour toutes les versions autres que celles indiquées ci-dessus.
**Supprimer la règle udev** | Supprimez toutes les règles udev qui réservent des noms d’interface basés sur l’adresse mac, etc. | Supprimez-les manuellement pour toutes les versions autres que celles indiquées ci-dessus.
**Mettre à jour les interfaces réseau** | Mettez à jour les interfaces réseau pour recevoir l’adresse IP basée sur DHCP.nst | Mettez-les à jour manuellement pour toutes les versions autres que celles indiquées ci-dessus.
**Activer ssh** | Vérifiez que ssh est activé et que le service sshd est configuré pour démarrer automatiquement au redémarrage.<br/><br/> Vérifiez que les requêtes de connexion ssh entrantes ne sont pas bloquées par le pare-feu du système d’exploitation ou par des règles de script.| Activez-le manuellement pour toutes les versions autres que celles indiquées ci-dessus.

Le tableau suivant récapitule les étapes accomplies automatiquement pour les systèmes d’exploitation répertoriés ci-dessus.


| Action                                      | Migration VMware basée sur un agent | Migration VMware sans agent | Hyper\-V   |
|---------------------------------------------|-------------------------------|----------------------------|------------|
| Installer Hyper\-V Integration Services pour Linux | Oui                           | Oui                        | Inutile |
| Activer la journalisation de la console série Azure         | Oui                           | Oui                        | Non         |
| Mettre à jour le fichier de mappage d’appareils                      | Oui                           | Non                         | Non         |
| Mettre à jour les entrées fstab                        | Oui                           | Oui                        | Non         |
| Supprimer la règle udev                            | Oui                           | Oui                        | Non         |
| Mettre à jour les interfaces réseau                   | Oui                           | Oui                        | Non         |
| Activer ssh                                  | Non                            | Non                         | Non         |

Apprenez-en davantage sur les étapes nécessaires à l’[exécution d’une machine virtuelle Linux sur Azure](../virtual-machines/linux/create-upload-generic.md) et obtenez des instructions pour certaines distributions Linux très répandues.


## <a name="check-azure-vm-requirements"></a>Exigences des machines virtuelles Azure

Les ordinateurs locaux que vous répliquez dans Azure doivent être conformes la configuration requise des machines virtuelles Azure en ce qui concerne le système d’exploitation et l’architecture, les disques, les paramètres réseau et le nommage des machines virtuelles.

Avant d’effectuer la migration, examinez la configuration requise des machines virtuelles Azure pour la migration [VMware](migrate-support-matrix-vmware-migration.md#azure-vm-requirements), [Hyper-V](migrate-support-matrix-hyper-v-migration.md#azure-vm-requirements) et des [serveurs physiques](migrate-support-matrix-physical-migration.md#azure-vm-requirements).



## <a name="prepare-to-connect-after-migration"></a>Préparer la connexion après la migration

Des machines virtuelles Azure sont créées pendant la migration vers Azure. Après la migration, vous devez pouvoir vous connecter aux nouvelles machines virtuelles Azure. Plusieurs étapes sont nécessaires pour établir une connexion.

### <a name="prepare-to-connect-to-azure-windows-vms"></a>Préparer la connexion aux machines virtuelles Windows Azure

Sur des ordinateurs Windows locaux :

1. Configurez les paramètres Windows. Parmi les paramètres figurent la suppression des routes persistantes statiques ou du proxy WinHTTP.
2. Vérifiez que les [services nécessaires](../virtual-machines/windows/prepare-for-upload-vhd-image.md#check-the-windows-services) sont en cours d’exécution.
3. Activez le bureau à distance (RDP) pour autoriser les connexions à distance à l’ordinateur local. Découvrez comment [utiliser PowerShell pour activer RDP](../virtual-machines/windows/prepare-for-upload-vhd-image.md#update-remote-desktop-registry-settings).
4. Pour accéder à une machine virtuelle Azure via Internet après la migration, dans le Pare-feu Windows de l’ordinateur local, autorisez TCP et UDP dans le profil public, puis définissez RDP comme application autorisée pour tous les profils.
5. Si vous voulez accéder à une machine virtuelle Azure via un réseau privé virtuel (VPN) de site à site après la migration, dans le Pare-feu Windows de l’ordinateur local, autorisez RDP pour les profils de domaine et privé. Découvrez comment [autoriser le trafic RDP](../virtual-machines/windows/prepare-for-upload-vhd-image.md#configure-windows-firewall-rules). 
6. Veillez à ce qu’aucune mise à jour Windows ne soit en attente sur la machine virtuelle locale pendant la migration. Si c’est le cas, les mises à jour risquent de s’installer sur la machine virtuelle Azure après la migration, et vous ne pourrez pas vous y connecter tant que les mises à jour n’auront pas abouti.


### <a name="prepare-to-connect-with-linux-azure-vms"></a>Préparer la connexion aux machines virtuelles Azure Linux

Sur les ordinateurs Linux locaux :

1. Vérifiez que le service Secure Shell est configuré pour démarrer automatiquement au démarrage du système.
2. Vérifiez que les règles de pare-feu autorisent une connexion SSH.

### <a name="configure-azure-vms-after-migration"></a>Configurer les machines virtuelles Azure après la migration

Après la migration, effectuez ces étapes sur les machines virtuelles Azure qui ont été créées :

1. Pour vous connecter à la machine virtuelle via Internet, attribuez-lui une adresse IP publique. Pour la machine virtuelle Azure, vous devez utiliser une adresse IP publique différente de celle que vous avez utilisée pour votre ordinateur local. [Plus d’informations](../virtual-network/virtual-network-public-ip-address.md)
2. Vérifiez que les règles de groupe de sécurité réseau (NSG) de la machine virtuelle autorisent les connexions entrantes sur le port RDP ou SSH.
3. Cochez [diagnostics de démarrage](../virtual-machines/troubleshooting/boot-diagnostics.md#enable-boot-diagnostics-on-existing-virtual-machine) pour afficher la machine virtuelle.


## <a name="next-steps"></a>Étapes suivantes

Choisissez la méthode que vous souhaitez utiliser pour [migrer des machines virtuelles VMware](server-migrate-overview.md) vers Azure, ou lancez la migration des [machines virtuelles Hyper-V](tutorial-migrate-hyper-v.md) ou des [serveurs physiques ou virtualisés ou des machines virtuelles cloud](tutorial-migrate-physical-virtual-machines.md).

## <a name="see-whats-supported"></a>Opérations prises en charge

Pour les machines virtuelles VMware, la migration de serveur prend en charge la [migration avec ou sans agent](server-migrate-overview.md).

- **Machines virtuelles VMware** : Vérifiez la [configuration requise et la prise en charge de la migration](migrate-support-matrix-vmware-migration.md) pour les machines virtuelles VMware.
- **Machines virtuelles Hyper-V** : Vérifiez la [configuration requise et la prise en charge pour la migration](migrate-support-matrix-hyper-v-migration.md) des machines virtuelles Hyper-V.
- **Machines physiques** : Vérifiez la [configuration requise et la prise en charge pour la migration](migrate-support-matrix-physical-migration.md) des ordinateurs physiques locaux et des autres serveurs virtualisés. 
