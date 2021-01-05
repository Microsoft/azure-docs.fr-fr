---
title: Nouveautés d’Azure Site Recovery
description: Fournit un résumé des nouvelles fonctionnalités et des dernières mises à jour du service Azure Site Recovery.
ms.topic: conceptual
ms.date: 10/22/2020
ms.openlocfilehash: 3667f963eac0ebc719872db00ac069c7da70ec91
ms.sourcegitcommit: b849ecdc8aa97337299b0f09970b7810c59cd044
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96310355"
---
# <a name="whats-new-in-site-recovery"></a>Nouveautés de Site Recovery

Le service [Azure Site Recovery](site-recovery-overview.md) fait l’objet de mises à jour et d’améliorations continues. Pour vous aider à rester à jour, cet article vous donne des informations sur les versions les plus récentes, les nouvelles fonctionnalités et le nouveau contenu. Cette page est régulièrement mise à jour.

Vous pouvez suivre et vous abonner aux notifications de mise à jour Site Recovery dans le canal [Mises à jour Azure](https://azure.microsoft.com/updates/?product=site-recovery).

## <a name="supported-updates"></a>Mises à jour prises en charge

Pour les composants Site Recovery, nous prenons en charge les versions N-4, où N correspond à la dernière version publiée. Celles-ci sont récapitulées dans le tableau ci-dessous.

**Mettre à jour** |  **Installation unifiée** | **OVA du serveur de configuration** | **Agent du service Mobilité** | **Fournisseur Site Recovery** | **Agent Recovery Services**
--- | --- | --- | --- | --- | ---
[Cumul 52](https://support.microsoft.com/help/4597409/)  | 9.39.5796.1 | 5.1.6458.0 | 9.39.5796.1 | 5.1.6458.0  | 2.0.9196.0
[Cumul 51](https://support.microsoft.com/help/4590304)  | 9.38.5761.1 | 5.1.6400.0 | 9.38.5761.1 | 5.1.6400.0  | 2.0.9193.0
[Cumul 50](https://support.microsoft.com/help/4582666/) | 9.37.5724.1 | 5.1.6347.0 | 9.37.5724.1 | 5.1.6347.0  | 2.0.9192.0
[Cumul 49](https://support.microsoft.com/help/4578241/) | 9.36.5696.1 | 5.1.6315.0 | 9.36.5696.1 | 5.1.6315.0 | 2.0.9188.0
[Rollup 48](https://support.microsoft.com/help/4573888/) | 9.35.5659.1 | 5.1.6200.0 | 9.35.5659.1 | 5.1.6200.0 | 2.0.9186.0

[En savoir plus](service-updates-how-to.md) sur l’installation et la prise en charge des mises à jour.


## <a name="updates-november-2020"></a>Mises à jour (novembre 2020)

### <a name="update-rollup-52"></a>Correctif cumulatif 52

Le [correctif cumulatif 52](https://support.microsoft.com/help/4597409/update-rollup-52-for-azure-site-recovery) fournit les mises à jour suivantes :

**Mettre à jour** | **Détails**
--- | ---
**Fournisseurs et agents** | Mises à jour des agents et des fournisseurs Site Recovery comme décrit dans le correctif cumulatif.
**Correctifs/améliorations** | Un certain nombre de correctifs et d’améliorations comme détaillé dans le cumul, y compris la nouvelle prise en charge de Linux pour le service Mobility.
**Récupération d’urgence de machine virtuelle Azure** | Prise en charge des machines virtuelles exécutant RHEL 8.3 et Oracle Linux 7.9
**Récupération d’urgence des machines virtuelles VMware physiques sur Azure** | Prise en charge des machines virtuelles exécutant RHEL 8.3, Oracle Linux 7.9/8.3.

## <a name="updates-october-2020"></a>Mises à jour (octobre 2020)

### <a name="update-rollup-51"></a>Correctif cumulatif 51

Le [correctif cumulatif 51](https://support.microsoft.com/help/4590304/update-rollup-51-for-azure-site-recovery) fournit les mises à jour suivantes :

**Mettre à jour** | **Détails**
--- | ---
**Fournisseurs et agents** | Mises à jour des agents et des fournisseurs Site Recovery comme décrit dans le correctif cumulatif.
**Correctifs/améliorations** | Un certain nombre de correctifs et d’améliorations comme détaillé dans le cumul, y compris la nouvelle prise en charge de Linux pour le service Mobility.

## <a name="updates-september-2020"></a>Mises à jour (septembre 2020)

### <a name="update-rollup-50"></a>Correctif cumulatif 50

Le [correctif cumulatif 50](https://support.microsoft.com/help/4582666/update-rollup-50-for-azure-site-recovery) fournit les mises à jour suivantes :

**Mettre à jour** | **Détails**
--- | ---
**Fournisseurs et agents** | Mises à jour des agents et des fournisseurs Site Recovery comme décrit dans le correctif cumulatif.
**Correctifs/améliorations** | Un certain nombre de correctifs et d’améliorations, comme détaillé dans le correctif cumulatif.

## <a name="updates-august-2020"></a>Mises à jour (août 2020)

### <a name="update-rollup-49"></a>Correctif cumulatif 49

Le [correctif cumulatif 49](https://support.microsoft.com/help/4578241/update-rollup-49-for-azure-site-recovery) fournit les mises à jour suivantes :

**Mettre à jour** | **Détails**
--- | ---
**Fournisseurs et agents** | Mises à jour des agents et des fournisseurs Site Recovery comme décrit dans le correctif cumulatif.
**Correctifs/améliorations** | Un certain nombre de correctifs et d’améliorations comme détaillé dans le cumul, y compris la nouvelle prise en charge de Linux pour le service Mobility.

## <a name="updates-july-2020"></a>Mises à jour (juillet 2020)

### <a name="update-rollup-48"></a>Correctif cumulatif 48

Le [correctif cumulatif 48](https://support.microsoft.com/help/4573888/update-rollup-48-for-azure-site-recovery) fournit les mises à jour suivantes :

**Mettre à jour** | **Détails**
--- | ---
**Fournisseurs et agents** | Mises à jour des agents et des fournisseurs Site Recovery comme décrit dans le correctif cumulatif.
**Correctifs/améliorations** | Un certain nombre de correctifs et d’améliorations, comme détaillé dans le correctif cumulatif.

> [!NOTE]
> Le correctif cumulatif 48 présente un problème connu avec l’activation de la réplication pour les machines Linux chiffrées avec ADE. [Plus d’informations](./azure-to-azure-troubleshoot-errors.md#enable-protection-failed-as-the-installer-is-unable-to-find-the-root-disk-error-code-151137)

### <a name="update-rollup-47"></a>Correctif cumulatif 47

Le [correctif cumulatif 47](https://support.microsoft.com/help/4570609/update-rollup-47-for-azure-site-recovery) fournit les mises à jour suivantes :

**Mettre à jour** | **Détails**
--- | ---
**Fournisseurs et agents** | Mises à jour des agents et des fournisseurs Site Recovery comme décrit dans le correctif cumulatif.
**Correctifs/améliorations** | Un certain nombre de correctifs et d’améliorations, comme détaillé dans le correctif cumulatif.

## <a name="updates-june-2020"></a>Mises à jour (juin 2020)

### <a name="update-rollup-46"></a>Correctif cumulatif 46

Le [correctif cumulatif 46](https://support.microsoft.com/help/4564347/update-rollup-46-for-azure-site-recovery) fournit les mises à jour suivantes :

**Mettre à jour** | **Détails**
--- | ---
**Fournisseurs et agents** | Mises à jour des agents et des fournisseurs Site Recovery comme décrit dans le correctif cumulatif.
**Correctifs/améliorations** | Un certain nombre de correctifs et d’améliorations, comme détaillé dans le correctif cumulatif.

## <a name="updates-march-2020"></a>Mises à jour (mars 2020)

### <a name="update-rollup-45"></a>Correctif cumulatif 45

Le [correctif cumulatif 45](https://support.microsoft.com/help/4550047/update-rollup-45-for-azure-site-recovery) fournit les mises à jour suivantes :

**Mettre à jour** | **Détails**
--- | ---
**Fournisseurs et agents** | Mises à jour des agents et des fournisseurs Site Recovery comme décrit dans le correctif cumulatif.
**Correctifs/améliorations** | Un certain nombre de correctifs et d’améliorations, comme détaillé dans le correctif cumulatif.

## <a name="updates-january-2020"></a>Mises à jour (janvier 2020)

### <a name="update-rollup-44"></a>Correctif cumulatif 44

Le [correctif cumulatif 44](https://support.microsoft.com/help/4538187/update-rollup-44-for-azure-site-recovery) fournit les mises à jour suivantes.

**Mettre à jour** | **Détails**
--- | ---
**Fournisseurs et agents** | Aucune mise à jour des fournisseurs et des agents Site Recovery.
**Correctifs/améliorations** | Un certain nombre de correctifs et d’améliorations, comme détaillé dans le correctif cumulatif.

### <a name="azure-vmware-disaster-recovery"></a>Récupération d’urgence d’Azure VMware

L’offre Machines virtuelles Microsoft Azure prend désormais en charge les machines virtuelles activées pour le chiffrement au repos avec des clés gérées par le client. [Plus d’informations](azure-to-azure-how-to-enable-replication-cmk-disks.md)


### <a name="update-rollup-43"></a>Correctif cumulatif 43

Le [correctif cumulatif 43](https://support.microsoft.com/help/4537047/update-rollup-43-for-azure-site-recovery) fournit les mises à jour suivantes.

**Mettre à jour** | **Détails**
--- | ---
**Fournisseurs et agents** | Mises à jour des agents et des fournisseurs Site Recovery (comme décrit dans le correctif cumulatif)
**Correctifs/améliorations** | Un certain nombre de correctifs et d’améliorations (comme décrit dans le correctif cumulatif)


## <a name="updates-november-2019"></a>Mises à jour (novembre 2019)

### <a name="update-rollup-42"></a>Correctif cumulatif 42

La [mise à jour Cumul 42](https://support.microsoft.com/help/4531426/update-rollup-42-for-azure-site-recovery) fournit les mises à jour suivantes.

**Mettre à jour** | **Détails**
--- | ---
**Fournisseurs et agents** | Mises à jour des agents et des fournisseurs Site Recovery (comme décrit dans le correctif cumulatif)
**Correctifs/améliorations** | Un certain nombre de correctifs et d’améliorations (comme décrit dans le correctif cumulatif)


### <a name="azure-vm-disaster-recovery"></a>Récupération d’urgence de machine virtuelle Azure

Les nouvelles fonctionnalités de la reprise d’activité des machines virtuelles Azure sont résumées dans le tableau.

**Fonctionnalité** | **Détails**
--- | ---
**UEFI** | Site Recovery prend désormais en charge la récupération d’urgence pour les machines virtuelles Azure avec une architecture de démarrage UEFI.
**Linux** | Site Recovery prend désormais en charge les machines virtuelles Azure exécutant Linux avec Azure Disk Encryption (ADE).
**Génération 2** | Toutes les machines virtuelles Azure de 2e génération sont désormais prises en charge pour la récupération d’urgence.
**Régions** | Vous pouvez maintenant activer la récupération d’urgence pour les machines virtuelles Azure dans la zone de géolocalisation norvégienne.

### <a name="vmware-to-azure-disaster-recovery"></a>Récupération d’urgence VMware vers Azure

Les nouvelles fonctionnalités de la reprise d’activité VMware vers Azure sont résumées dans le tableau.

**Fonctionnalité** | **Détails**
--- | ---
**UEFI** | Site Recovery prend désormais en charge la récupération d’urgence pour les machines virtuelles VMware avec une architecture de démarrage UEFI.<br/><br/> Les systèmes d’exploitation pris en charge incluent Windows Server 2019, Windows Server 2016, Windows Server 2012 R2, Windows Server 2012, SLES 12 SP4 et RHEL 8.

## <a name="update-to-servicing-stack-updatesha-2"></a>Mise à jour vers la mise à jour de la pile de maintenance/SHA-2

Pour la récupération d’urgence de machines virtuelles Azure vers une région secondaire ou la récupération d'urgence de machines virtuelles VMware locales ou des serveurs physiques vers Azure, notez les points suivants :

- À partir de la version 9.30.5407.1. de l’extension du service Mobility (pour les machines virtuelles Azure) et de l’agent du service Mobility (pour VMware/ordinateurs physiques), certains systèmes d’exploitation d’ordinateur doivent exécuter la mise à jour de la pile de maintenance et SHA-2. Les détails sont indiqués dans le tableau ci-dessous.
- Installez la mise à jour et l’algorithme SHA-2 conformément à la base de connaissances liée. SHA-1 n’est pas pris en charge à partir de septembre 2019, et si la signature de code SHA-2 n’est pas activée, l’extension de l’agent ne sera pas installée/mise à niveau comme prévu.
- En savoir plus sur la [mise à niveau et la configuration requise pour SHA-2](https://aka.ms/SHA-2KB).

**Système d’exploitation** | **Microsoft Azure** | **Machine virtuelle VMware/ordinateur physique**
--- | --- | ---
**Windows 2008 R2 SP1** | [Mise à jour de la pile de maintenance](https://support.microsoft.com/help/4490628)<br/> [SHA-2](https://support.microsoft.com/help/4474419)| [Mise à jour de la pile de maintenance](https://support.microsoft.com/help/4490628)<br/> [SHA-2](https://support.microsoft.com/help/4474419)
**Windows 2008 SP2** | [Mise à jour de la pile de maintenance](https://support.microsoft.com/help/4493730)<br/> [SHA-2](https://support.microsoft.com/help/4474419)| [Mise à jour de la pile de maintenance](https://support.microsoft.com/help/4493730)<br/> [SHA-2](https://support.microsoft.com/help/4474419)
**Windows 7 SP1** | [Mise à jour de la pile de maintenance](https://support.microsoft.com/help/4490628)<br/> [SHA-2](https://support.microsoft.com/help/4474419)| [Mise à jour de la pile de maintenance](https://support.microsoft.com/help/4490628)<br/> [SHA-2](https://support.microsoft.com/help/4474419).



## <a name="updates-october-2019"></a>Mises à jour (octobre 2019)

### <a name="update-rollup-41"></a>Correctif cumulatif 41

Le [correctif cumulatif 41](https://support.microsoft.com/help/4528026/update-rollup-41-for-azure-site-recovery) fournit les mises à jour suivantes.

**Mettre à jour** | **Détails**
--- | ---
**Fournisseurs et agents** | Mises à jour des agents et des fournisseurs Site Recovery (comme décrit dans le correctif cumulatif)
**Correctifs/améliorations** | Un certain nombre de correctifs et d’améliorations (comme décrit dans le correctif cumulatif)



### <a name="azure-vm-disaster-recovery"></a>Récupération d’urgence de machine virtuelle Azure

Les nouvelles fonctionnalités de la reprise d’activité des machines virtuelles Azure sont résumées dans le tableau.

**Fonctionnalité** | **Détails**
--- | ---
**Paramètres de basculement de test** | Lors de la configuration d’un basculement de test, vous pouvez maintenant configurer les paramètres de la machine virtuelle et du réseau de basculement de test, y compris l’adresse IP, le groupe de sécurité réseau, l’équilibre de charge interne et l’adresse IP publique pour chaque carte réseau. Ces paramètres sont facultatifs et ne modifient pas le comportement actuel. Si vous ne configurez pas ces paramètres, vous pouvez choisir un réseau virtuel Azure au moment du test de basculement. [Plus d’informations](https://azure.microsoft.com/blog/customize-networking-for-dr-drills-azure-site-recovery/)
**Plans de récupération** | Les plans de récupération sont désormais limités à 100 machines virtuelles afin de garantir la fiabilité du basculement.

### <a name="vmware-to-azure-disaster-recovery"></a>Récupération d’urgence VMware vers Azure

Les nouvelles fonctionnalités de la reprise d’activité VMware vers Azure sont résumées dans le tableau.

**Fonctionnalité** | **Détails**
--- | ---
**Plans de récupération** | Les plans de récupération sont désormais limités à 100 machines virtuelles afin de garantir la fiabilité du basculement.


## <a name="updates-september-2019"></a>Mises à jour (septembre 2019)

### <a name="update-rollup-40"></a>Mise à jour Rollup 40

La [mise à jour Rollup 40](https://support.microsoft.com/help/4521530/update-rollup-40-for-azure-site-recovery) fournit les mises à jour suivantes.

**Mettre à jour** | **Détails**
--- | ---
**Fournisseurs et agents** | Mises à jour des agents et des fournisseurs Site Recovery (comme décrit dans le correctif cumulatif)
**Correctifs/améliorations** | Un certain nombre de correctifs et d’améliorations (comme décrit dans le correctif cumulatif)




### <a name="azure-vm-disaster-recovery"></a>Récupération d’urgence de machine virtuelle Azure

Les nouvelles fonctionnalités de la reprise d’activité des machines virtuelles Azure sont résumées dans le tableau.

**Fonctionnalité** | **Détails**
--- | ---
**Nettoyage après restauration automatique** | Après un basculement vers le serveur Azure secondaire, puis la restauration automatique vers la région primaire, Site Recovery nettoie automatiquement les machines de la région secondaire. Il n’est pas nécessaire de supprimer manuellement les machines virtuelles et les cartes réseau.
**Le test de basculement conserve l’adresse IP** | Vous pouvez maintenant conserver l’adresse IP de la machine virtuelle source pendant une reprise d’activité après sinistre et sélectionner une adresse IP statique pour un test de basculement.

### <a name="vmwarephysical-server-disaster-recovery"></a>VMware/récupération d’urgence de serveur physique

Les fonctionnalités ajoutées ce mois-ci sont résumées dans le tableau.

**Fonctionnalité** | **Détails**
--- | ---
Nouvelles alertes du serveur de processus | Nous avons ajouté de nouvelles alertes de serveur de processus. [Plus d’informations](vmware-physical-azure-monitor-process-server.md)

### <a name="hyper-v-disaster-recovery"></a>Reprise d’activité Hyper-V

Les fonctionnalités ajoutées ce mois-ci sont résumées dans le tableau.

**Fonctionnalité** | **Détails**
--- | ---
Compte de stockage | Site Recovery prend désormais en charge l’utilisation de comptes de stockage avec un pare-feu activé pour la reprise d’activité Hyper-V vers Azure.  Vous pouvez sélectionner des comptes de stockage avec pare-feu comme compte cible ou pour le stockage du cache. Si vous utilisez un compte de stockage avec pare-feu, veillez à activer l’option permettant d’autoriser les services Microsoft approuvés.<br/><br/> Cela est pris en charge pour les machines virtuelles Hyper-V avec ou sans System Center VMM.


## <a name="updates-august-2019"></a>Mises à jour (août 2019)

### <a name="update-rollup-39"></a>Correctif cumulatif 39

Le [correctif cumulatif 39](https://support.microsoft.com/help/4517283/update-rollup-39-for-azure-site-recovery) fournit les mises à jour suivantes.

**Mettre à jour** | **Détails**
--- | ---
**Fournisseurs et agents** | Mises à jour des agents et des fournisseurs Site Recovery (comme décrit dans le correctif cumulatif)
**Correctifs/améliorations** | Un certain nombre de correctifs et d’améliorations (comme décrit dans le correctif cumulatif)


### <a name="azure-vm-disaster-recovery"></a>Récupération d’urgence de machine virtuelle Azure

Les nouvelles fonctionnalités de la reprise d’activité des machines virtuelles Azure sont résumées dans le tableau.

**Fonctionnalité** | **Détails**
--- | ---
**Chiffrement sans Azure AD** | Le chiffrement sans une application Azure AD est désormais pris en charge pour la réplication de machines virtuelles Azure vers des disques managés exécutant Windows.
**Ressources réseau pour le basculement** | Lors du basculement vers une autre région, vous pouvez désormais attacher des paramètres de ressource réseau (groupes de sécurité réseau, équilibrage de charge, adresse IP publique) à une machine virtuelle.

## <a name="updates-july-2019"></a>Mises à jour (juillet 2019)

### <a name="update-rollup-38"></a>Correctif cumulatif 38

Le [Correctif cumulatif 38](https://support.microsoft.com/help/4513507/) fournit les mises à jour suivantes.

**Mettre à jour** | **Détails**
--- | ---
**Fournisseurs et agents** | Mises à jour des agents et des fournisseurs Site Recovery (comme décrit dans le correctif cumulatif)
**Correctifs/améliorations** | Un certain nombre de correctifs et d’améliorations (comme décrit dans le correctif cumulatif)


### <a name="general"></a>Général

Site Recovery prend désormais en charge l’utilisation de comptes de stockage v2 à usage général pour le stockage de cache ou le stockage cible. Auparavant, seul v1 était pris en charge.

### <a name="vmware-to-azure-disaster-recovery"></a>Récupération d’urgence VMware vers Azure

Vous pouvez désormais répliquer des disques jusqu’à 8 To lors de la réplication d’une machine virtuelle Azure avec des disques managés.


## <a name="updates-june-2019"></a>Mises à jour (juin 2019)

### <a name="update-rollup-37"></a>Correctif cumulatif 37

Le [Correctif cumulatif 37](https://support.microsoft.com/help/4508614/) fournit les mises à jour suivantes.

**Mettre à jour** | **Détails**
--- | ---
**Fournisseurs et agents** | Mises à jour des agents et des fournisseurs Site Recovery (comme décrit dans le correctif cumulatif)
**Correctifs/améliorations** | Un certain nombre de correctifs et d’améliorations (comme décrit dans le correctif cumulatif)


### <a name="vmwarephysical-server-disaster-recovery"></a>VMware/récupération d’urgence de serveur physique

Les fonctionnalités ajoutées ce mois-ci sont résumées dans le tableau.

**Fonctionnalité** | **Détails**
--- | ---
**Partitions GPT** | À partir du correctif cumulatif 37 (version du service mobilité 9.25.5241.1), jusqu’à cinq partitions GPT sont prises en charge en mode UEFI. Avant cette mise à jour, seules quatre étaient prises en charge.



## <a name="updates-may-2019"></a>Mises à jour (mai 2019)

### <a name="update-rollup-36"></a>Correctif cumulatif 36

Le [Correctif cumulatif 36](https://support.microsoft.com/help/4503156) fournit les mises à jour suivantes.

**Mettre à jour** | **Détails**
--- | ---
**Fournisseurs et agents** | Une mise à jour des agents et des fournisseurs Site Recovery (comme décrit dans le correctif cumulatif)
**Correctifs/améliorations** | Un certain nombre de correctifs et d’améliorations (comme décrit dans le correctif cumulatif)

### <a name="azure-vm-disaster-recovery"></a>Récupération d’urgence de machine virtuelle Azure

Les fonctionnalités ajoutées ce mois-ci sont résumées dans le tableau.

**Fonctionnalité** | **Détails**
--- | ---
**Répliquer les disques ajoutés** | Permet la réplication pour les disques de données ajoutés à une machine virtuelle Azure déjà activée pour la récupération d’urgence. [Plus d’informations](azure-to-azure-enable-replication-added-disk.md)
**Mises à jour automatiques** | Lors de la configuration des mises à jour automatiques de l’extension du service de mobilité qui s’exécute sur des machines virtuelles Azure activées pour la récupération d’urgence, vous pouvez maintenant sélectionner un compte Automation existant au lieu d’utiliser le compte par défaut créé par Site Recovery. [Plus d’informations](azure-to-azure-autoupdate.md)


### <a name="vmwarephysical-server-disaster-recovery"></a>VMware/récupération d’urgence de serveur physique

Les fonctionnalités ajoutées ce mois-ci sont résumées dans le tableau.

**Fonctionnalité** | **Détails**
--- | ---
**Surveillance des serveurs de traitement** | Pour la récupération d’urgence des machines virtuelles VMware et des serveurs physiques locaux, surveillez et résolvez les problèmes liés aux serveurs de traitement grâce à des alertes et rapports améliorés sur l’intégrité des serveurs. [Plus d’informations](vmware-physical-azure-monitor-process-server.md)





## <a name="updates-march-2019"></a>Mises à jour (mars 2019)

### <a name="update-rollup-35"></a>Correctif cumulatif 35

Le [Correctif cumulatif 35](https://support.microsoft.com/en-us/help/4494485/update-rollup-35-for-azure-site-recovery) fournit les mises à jour suivantes.

**Mettre à jour** | **Détails**
--- | ---
**Fournisseurs et agents** | Une mise à jour des agents et des fournisseurs Site Recovery (comme décrit dans le correctif cumulatif)
**Correctifs/améliorations** | Un certain nombre de correctifs et d’améliorations (comme décrit dans le correctif cumulatif)

### <a name="vmwarephysical-server-disaster-recovery"></a>VMware/récupération d’urgence de serveur physique

Les fonctionnalités ajoutées ce mois-ci sont résumées dans le tableau.

**Fonctionnalité** | **Détails**
--- | ---
**Disques managés** | La réplication de machines virtuelles locales VMware et de serveurs physiques s’effectue désormais directement sur les disques managés dans Azure. Les données locales sont envoyées vers un compte de stockage de cache dans Azure et les points de récupération sont créés sur des disques managés à l’emplacement cible. Ainsi, vous n’avez pas besoin de gérer plusieurs comptes de stockage cible.
**Serveur de configuration** | Site Recovery prend désormais en charge les serveurs de configuration avec plusieurs cartes réseau. Ajoutez des adaptateurs supplémentaires sur la machine virtuelle du serveur de configuration avant d’inscrire ce dernier dans le coffre. Ensuite, vous devez réinscrire le serveur dans le coffre.


## <a name="updates-february-2019"></a>Mises à jour (février 2019)

### <a name="update-rollup-34"></a>Correctif cumulatif 34

Le [Correctif cumulatif 34](https://support.microsoft.com/help/4490016/update-rollup-34-for-azure-site-recovery) fournit les mises à jour suivantes.

**Mettre à jour** | **Détails**
--- | ---
**Fournisseurs et agents** | Une mise à jour des agents et des fournisseurs Site Recovery (comme décrit dans le correctif cumulatif).
**Correctifs/améliorations** | Un certain nombre de correctifs et d’améliorations (comme décrit dans le correctif cumulatif).


### <a name="update-rollup-33"></a>Correctif cumulatif 33

Le [Correctif cumulatif 33](https://support.microsoft.com/help/4489582/update-rollup-33-for-azure-site-recovery) fournit les mises à jour suivantes.

**Mettre à jour** | **Détails**
--- | ---
**Fournisseurs et agents** | Une mise à jour des agents et des fournisseurs Site Recovery (comme décrit dans le correctif cumulatif).
**Correctifs/améliorations** | Un certain nombre de correctifs et d’améliorations (comme décrit dans le correctif cumulatif).


### <a name="azure-vm-disaster-recovery"></a>Récupération d’urgence de machine virtuelle Azure

Les fonctionnalités ajoutées ce mois-ci sont résumées dans le tableau.

**Fonctionnalité** | **Détails**
--- | ---
**Mappage réseau** | Pour la récupération d’urgence de machines virtuelles Azure, vous pouvez maintenant utiliser n’importe quel réseau cible disponible lorsque vous activez la réplication.
**SSD Standard** | Vous pouvez désormais configurer la récupération d’urgence pour les machines virtuelles Azure utilisant des [disques SSD standard](../virtual-machines/disks-types.md#standard-ssd).
**Espaces de stockage direct** | Vous pouvez configurer la récupération d’urgence pour les applis s’exécutant sur les applications de machine virtuelle Azure à l’aide de [Storage Spaces Direct](/windows-server/storage/storage-spaces/storage-spaces-direct-overview) pour une haute disponibilité.  L’utilisation de Storage Spaces Direct (S2D) avec Site Recovery offre une protection complète aux charges de travail de machine virtuelle Azure. S2D vous permet d’héberger un cluster invité dans Azure. Cela est particulièrement utile lorsqu’une machine virtuelle héberge une application critique, comme la couche SAP ASCS, SQL Server ou un serveur de fichiers avec montée en puissance parallèle.


### <a name="vmwarephysical-server-disaster-recovery"></a>VMware/récupération d’urgence de serveur physique

Les fonctionnalités ajoutées ce mois-ci sont résumées dans le tableau.

**Fonctionnalité** | **Détails**
--- | ---
**Système de fichiers Linux BTRFS** | Site Recovery prend désormais en charge la réplication des machines virtuelles VMware avec le système de fichiers BTRFS. La réplication n’est pas prise en charge dans les cas suivants :<br/><br/>- Le sous-volume du système de fichiers BTRFS est modifié après l’activation de la réplication.<br/><br/>- Le système de fichiers est réparti sur plusieurs disques.<br/><br/>- Le système de fichiers BTRFS prend en charge RAID.
**Windows Server 2019** | Prise en charge ajoutée pour les machines exécutant Windows Server 2019.


## <a name="updates-january-2019"></a>Mises à jour (janvier 2019)


### <a name="accelerated-networking-azure-vms"></a>Mise en réseau accélérée (machines virtuelles Azure)

L’accélération réseau permet la virtualisation d’E/S de racine unique (SR-IOV) sur une machine virtuelle, améliorant ainsi les performances réseau. Quand vous activez la réplication pour une machine virtuelle Azure, Site Recovery détecte si l’accélération réseau est activée ou non. Si elle l’est, Site Recovery configure automatiquement l’accélération réseau sur la machine virtuelle Azure du réplica cible après le basculement, à la fois sur [Windows](../virtual-network/create-vm-accelerated-networking-powershell.md#enable-accelerated-networking-on-existing-vms) et sur [Linux](../virtual-network/create-vm-accelerated-networking-cli.md#enable-accelerated-networking-on-existing-vms).

[Plus d’informations](azure-vm-disaster-recovery-with-accelerated-networking.md)

### <a name="update-rollup-32"></a>Correctif cumulatif 32

Le [Correctif cumulatif 32](https://support.microsoft.com/help/4485985/update-rollup-32-for-azure-site-recovery) fournit les mises à jour suivantes.

**Mettre à jour** | **Détails**
--- | ---
**Fournisseurs et agents** | Une mise à jour des agents et des fournisseurs Site Recovery (comme décrit dans le correctif cumulatif).
**Correctifs/améliorations** | Un certain nombre de correctifs et d’améliorations (comme décrit dans le correctif cumulatif).

### <a name="azure-vm-disaster-recovery"></a>Récupération d’urgence de machine virtuelle Azure

Les fonctionnalités ajoutées ce mois-ci sont résumées dans le tableau.

**Fonctionnalité** | **Détails**
--- | ---
**Prise en charge de Linux** | Nouvelle prise en charge de RedHat Workstation 6/7, et nouvelles versions de noyau pour Ubuntu, Debian et SUSE.
**Espaces de stockage direct** | Site Recovery prend en charge les machines virtuelles Azure utilisant Storage Spaces Direct (S2D).

### <a name="vmware-vmsphysical-servers-disaster-recovery"></a>Récupération d’urgence des machines virtuelles VMware/serveurs physiques

Les fonctionnalités ajoutées ce mois-ci sont résumées dans le tableau.

**Fonctionnalité** | **Détails**
--- | ---
**Prise en charge de Linux** | Nouvelle prise en charge de RedHat Enterprise Linux 7.6, RedHat Workstation 6/7, Oracle Linux 6.10 et Oracle Linux 7.6, et nouvelles versions de noyau pour Ubuntu, Debian et SUSE.


### <a name="update-rollup-31"></a>Correctif cumulatif 31

Le [Correctif cumulatif 31](https://support.microsoft.com/help/4478871/update-rollup-31-for-azure-site-recovery) fournit les mises à jour suivantes.

**Mettre à jour** | **Détails**
--- | ---
**Fournisseurs et agents** | Une mise à jour des agents et des fournisseurs Site Recovery (comme décrit dans le correctif cumulatif).
**Correctifs/améliorations** | Un certain nombre de correctifs et d’améliorations (comme décrit dans le correctif cumulatif).

### <a name="vmware-vmsphysical-servers-replication"></a>Réplication des machines virtuelles VMware/serveurs physiques

Les fonctionnalités ajoutées ce mois-ci sont résumées dans le tableau.

**Fonctionnalité** | **Détails**
--- | ---
**Prise en charge de Linux** |  Nouvelle prise en charge d’Oracle Linux 6.8, Oracle Linux 6.9 et Oracle Linux 7.0 avec le noyau compatible Red Hat et pour le noyau Unbreakable Enterprise Kernel (UEK) Release 5.
**LVM** | Prend désormais en charge les volumes LVM et LVM2.<br/><br/> Le répertoire /boot des partitions de disque et des volumes LVM est maintenant pris en charge.
**Directories** | Prend désormais en charge les répertoires configurés en tant que partitions distinctes, ou les systèmes de fichiers qui ne se trouvent pas sur le même disque système :<br/><br/> /(root), /boot, /usr, /usr/local, /var, /etc.
**Windows Server 2008** | Prend désormais en charge les disques dynamiques.
**Type de basculement** | Amélioration des temps de basculement pour les machines virtuelles VMware lorsque storvsc et vsbus ne sont pas des pilotes de démarrage.
**Prise en charge du mode UEFI** | Les machines virtuelles Azure ne prennent pas en charge le mode de démarrage UEFI. Vous pouvez désormais migrer les serveurs physiques locaux avec UEFI vers Azure à l’aide de Site Recovery Site Recovery migre le serveur en convertissant le mode de démarrage en BIOS avant la migration. Site Recovery prenait auparavant en charge cette conversion pour les machines virtuelles uniquement. Les serveurs physiques exécutant Windows Server 2012 ou version ultérieure sont pris en charge.

### <a name="azure-vm-disaster-recovery"></a>Récupération d’urgence de machine virtuelle Azure

Les fonctionnalités ajoutées ce mois-ci sont résumées dans le tableau.

**Fonctionnalité** | **Détails**
--- | ---
**Prise en charge de Linux** | Nouvelle prise en charge d’Oracle Linux 6.8, Oracle Linux 6.9 et Oracle Linux 7.0 avec le noyau compatible Red Hat et pour le noyau Unbreakable Enterprise Kernel (UEK) Release 5.
**Système de fichiers Linux BTRFS** | Pris en charge pour les machines virtuelles Azure.
**Machines virtuelles Azure dans les zones de disponibilité** | Vous pouvez activer la réplication vers une autre région pour les machines virtuelles Azure déployées dans les zones de disponibilité. Vous pouvez désormais activer la réplication pour une machine virtuelle Azure et définir comme cible du basculement une instance de machine virtuelle unique, une machine virtuelle dans un groupe à haute disponibilité ou une machine virtuelle dans une zone de disponibilité. Le paramètre n’impacte pas la réplication. [Lisez](https://azure.microsoft.com/blog/disaster-recovery-of-zone-pinned-azure-virtual-machines-to-another-region/) l’annonce.
**Stockage avec pare-feu (portail/PowerShell)** | Nouvelle prise en charge des [comptes de stockage avec pare-feu](../storage/common/storage-network-security.md).<br/><br/> Vous pouvez répliquer des machines virtuelles Azure avec des disques non managés sur des comptes de stockage avec pare-feu sur une autre région Azure pour les scénarios de récupération d’urgence.<br/><br/> Vous pouvez utiliser des comptes de stockage avec pare-feu en tant que comptes de stockage cibles pour des disques non managés.<br/><br/> Fonction prise en charge dans le portail et avec PowerShell.

## <a name="updates-december-2018"></a>Mises à jour (décembre 2018)

### <a name="automatic-updates-for-the-mobility-service-azure-vms"></a>Mises à jour automatiques du service de mobilité (machines virtuelles Azure)

Site Recovery propose désormais une option pour mettre à jour automatiquement l’extension du service Mobilité. L’extension du service Mobilité est installée sur chaque machine virtuelle Azure répliquée par Site Recovery. Quand vous activez la réplication, vous pouvez autoriser Site Recovery à gérer les mises à jour de l’extension.

Les mises à jour ne nécessitent pas le redémarrage de la machine virtuelle et n’affectent pas la réplication. [Plus d’informations](azure-to-azure-autoupdate.md)

### <a name="pricing-calculator-for-azure-vm-disaster-recovery"></a>Calculatrice de prix pour la reprise d’activité de machines virtuelles Azure

La récupération d’urgence des machines virtuelles Azure occasionne des coûts liés aux licences des machines virtuelles ainsi que des frais de stockage et de réseau. Azure fournit une [calculatrice de prix](https://aka.ms/a2a-cost-estimator) pour vous aider à déterminer ces coûts. Site Recovery propose désormais un [exemple d’estimation du prix](https://aka.ms/a2a-cost-estimator) d’un déploiement basé sur une application à trois niveaux comprenant six machines virtuelles avec 12 disques HDD Standard et 6 disques SSD Premium.

- L’exemple suppose des changements de données totalisant 10 Go par jour pour le stockage Standard et 20 Go pour le stockage Premium.
- Pour estimer les coûts de votre propre déploiement, changez les variables.
- Vous pouvez spécifier le nombre de machines virtuelles, le nombre et le type de disques managés ainsi que le taux de changement de données total attendu sur les machines virtuelles.
- Vous pouvez également appliquer un facteur de compression pour estimer les coûts de bande passante.

[Lisez](https://azure.microsoft.com/blog/know-exactly-how-much-it-will-cost-for-enabling-dr-to-your-azure-vm/) l’annonce.


## <a name="updates-october-2018"></a>Mises à jour (octobre 2018)

### <a name="update-rollup-30"></a>Correctif cumulatif 30

Le [Correctif cumulatif 30](https://support.microsoft.com/help/4468181/azure-site-recovery-update-rollup-30) fournit les mises à jour suivantes.

**Mettre à jour** | **Détails**
--- | ---
**Fournisseurs et agents** | Une mise à jour des agents et des fournisseurs Site Recovery (comme décrit dans le correctif cumulatif).
**Correctifs/améliorations** | Un certain nombre de correctifs et d’améliorations (comme décrit dans le correctif cumulatif).

### <a name="azure-vm-disaster-recovery"></a>Récupération d’urgence de machine virtuelle Azure
Les fonctionnalités ajoutées ce mois-ci sont résumées dans le tableau.

**Fonctionnalité** | **Détails**
--- | ---
**Prise en charge des régions** | Site Recovery prend désormais en charge les régions Australie Centre 1 et Australie Centre 2.
**Prise en charge du chiffrement de disque** | Prise en charge ajoutée pour la récupération d’urgence des machines virtuelles Azure chiffrées au moyen d’ADE (Azure Disk Encryption) avec l’application Azure AD. [Plus d’informations](azure-to-azure-how-to-enable-replication-ade-vms.md)
**Exclusion de disque** | Les disques non initialisés sont maintenant automatiquement exclus lors de la réplication de machine virtuelle Azure.
**Stockage avec pare-feu (PowerShell)** | Nouvelle prise en charge des [comptes de stockage avec pare-feu](../storage/common/storage-network-security.md).<br/><br/> Vous pouvez répliquer des machines virtuelles Azure avec des disques non managés sur des comptes de stockage avec pare-feu sur une autre région Azure pour les scénarios de récupération d’urgence.<br/><br/> Vous pouvez utiliser des comptes de stockage avec pare-feu en tant que comptes de stockage cibles pour des disques non managés.<br/><br/> Prise en charge avec PowerShell uniquement.


### <a name="update-rollup-29"></a>Correctif cumulatif 29

Le [Correctif cumulatif 29](https://support.microsoft.com/help/4466466/update-rollup-29-for-azure-site-recovery) fournit les mises à jour suivantes.

**Mettre à jour** | **Détails**
--- | ---
**Fournisseurs et agents** | Une mise à jour des agents et des fournisseurs Site Recovery (comme décrit dans le correctif cumulatif).
**Correctifs/améliorations** | Un certain nombre de correctifs et d’améliorations (comme décrit dans le correctif cumulatif).


## <a name="updates-august-2018"></a>Mises à jour (août 2018)

### <a name="update-rollup-28"></a>Correctif cumulatif 28

Le [Correctif cumulatif 28](https://support.microsoft.com/help/4460079/update-rollup-28-for-azure-site-recovery) fournit les mises à jour suivantes.

**Mettre à jour** | **Détails**
--- | ---
**Fournisseurs et agents** | Une mise à jour des agents et des fournisseurs Site Recovery (comme décrit dans le correctif cumulatif).
**Correctifs/améliorations** | Un certain nombre de correctifs et d’améliorations (comme décrit dans le correctif cumulatif).

### <a name="azure-vm-disaster-recovery"></a>Récupération d’urgence de machine virtuelle Azure
Les fonctionnalités ajoutées ce mois-ci sont résumées dans le tableau.

**Fonctionnalité** | **Détails**
--- | ---
**Prise en charge de Linux** | Nouvelle prise en charge de RedHat Enterprise Linux 6.10 ; CentOS 6.10.<br/><br/>
**Prise en charge du cloud** | Prise en charge de la récupération d’urgence pour les machines virtuelles Azure dans le cloud allemand.
**Récupération d’urgence entre abonnements** | Prise en charge de la réplication des machines virtuelles Azure d’une région vers une autre, dans un autre abonnement du même locataire Azure Active Directory. [Plus d’informations](https://aka.ms/cross-sub-blog)

### <a name="vmware-vmphysical-server-disaster-recovery"></a>Récupération d’urgence des machines virtuelles VMware/serveurs physiques
Les fonctionnalités ajoutées ce mois-ci sont résumées dans le tableau.

**Fonctionnalité** | **Détails**
--- | ---
**Prise en charge de Linux** | Nouvelle prise en charge de RedHat Enterprise Linux 6.10 ; CentOS 6.10.<br/><br/> Les machines virtuelles Linux utilisant le style de partition GPT (GUID Partition Table) en mode de compatibilité avec un BIOS hérité sont désormais prises en charge. Consultez le [FAQ sur les machines virtuelles Azure](../virtual-machines/faq-for-disks.md) pour plus d’informations.
**Récupération d’urgence des machines virtuelles après la migration** | Prise en charge de la récupération d’urgence vers une région secondaire pour une machine virtuelle VMware locale migrée vers Azure, sans avoir à désinstaller le service de mobilité sur la machine virtuelle avant d’activer la réplication.
**Windows Server 2008** | Prise en charge de la migration des machines exécutant Windows Server 2008 R2/2008 64 bits et 32 bits.<br/><br/> Migration uniquement (réplication et basculement). La restauration automatique n’est pas prise en charge.

## <a name="updates-july-2018"></a>Mises à jour (juillet 2018)

### <a name="update-rollup-27-july-2018"></a>Correctif cumulatif 27 (juillet 2018)

Le [Correctif cumulatif 27](https://support.microsoft.com/help/4055712/update-rollup-27-for-azure-site-recovery) fournit les mises à jour suivantes.

**Mettre à jour** | **Détails**
--- | ---
**Fournisseurs et agents** | Une mise à jour des agents et des fournisseurs Site Recovery (comme décrit dans le correctif cumulatif).
**Correctifs/améliorations** | Un certain nombre de correctifs et d’améliorations (comme décrit dans le correctif cumulatif).

### <a name="azure-vm-disaster-recovery"></a>Récupération d’urgence de machine virtuelle Azure

Les fonctionnalités ajoutées ce mois-ci sont résumées dans le tableau.

**Fonctionnalité** | **Détails**
--- | ---
**Prise en charge de Linux** | Nouvelle prise en charge de Red Hat Enterprise Linux 7.5.

### <a name="vmware-vmphysical-server-disaster-recovery"></a>Récupération d’urgence des machines virtuelles VMware/serveurs physiques

Les fonctionnalités ajoutées ce mois-ci sont résumées dans le tableau.

**Fonctionnalité** | **Détails**
--- | ---
**Prise en charge de Linux** | Nouvelle prise en charge de RedHat Enterprise Linux 7.5, SUSE Linux Enterprise Server 12.



## <a name="next-steps"></a>Étapes suivantes

Pour vous tenir informé de nos mises à jour, consultez la page [Mises à jour Azure](https://azure.microsoft.com/updates/?product=site-recovery).