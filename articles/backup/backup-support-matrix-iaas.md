---
title: Tableau de prise en charge pour la sauvegarde de machines virtuelles Azure
description: Fournit un récapitulatif des limitations et des paramètres de prise en charge de la sauvegarde de machines virtuelles Azure avec le service Sauvegarde Azure.
ms.topic: conceptual
ms.date: 09/13/2019
ms.custom: references_regions
ms.openlocfilehash: 8cd943a76113fb1680406253a46b8a4f9d32190b
ms.sourcegitcommit: 48cb2b7d4022a85175309cf3573e72c4e67288f5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96853201"
---
# <a name="support-matrix-for-azure-vm-backup"></a>Tableau de prise en charge pour la sauvegarde de machines virtuelles Azure

Vous pouvez utiliser le [service Sauvegarde Azure](backup-overview.md) pour sauvegarder des machines et des charges de travail locales ainsi que des machines virtuelles Azure. Cet article récapitule les paramètres de prise en charge et les limitations de la sauvegarde de machines virtuelles Azure avec Sauvegarde Azure.

Autres tableaux de prise en charge :

- [Tableau de prise en charge générale](backup-support-matrix.md) pour Sauvegarde Azure
- [Tableau de prise en charge](backup-support-matrix-mabs-dpm.md) pour le serveur de sauvegarde Azure ou la sauvegarde System Center Data Protection Manager
- [Tableau de prise en charge](backup-support-matrix-mars-agent.md) de la sauvegarde avec l’agent MARS (Microsoft Azure Recovery Services)

## <a name="supported-scenarios"></a>Scénarios pris en charge

Voici comment vous pouvez sauvegarder et restaurer des machines virtuelles Azure avec le service Sauvegarde Azure.

**Scénario** | **Sauvegarde** | **Agent** |**Restauration**
--- | --- | --- | ---
sauvegarde directe de machines virtuelles Azure  | Sauvegardez la totalité de la machine virtuelle.  | Aucun autre agent n’est nécessaire sur la machine virtuelle Azure. La Sauvegarde Azure installe et utilise une extension l’[agent de machine virtuelle Azure](../virtual-machines/extensions/agent-windows.md) qui s’exécute sur la machine virtuelle. | Restaurez comme ceci :<br/><br/> - **Créez une machine virtuelle de base**. Ceci est pratique si la machine virtuelle n’a pas de configuration spéciale, comme plusieurs adresses IP.<br/><br/> - **Restaurer le disque de la machine virtuelle**. Restaurez le disque. Ensuite, attachez-le à une machine virtuelle existante ou créez une machine virtuelle à partir du disque avec PowerShell.<br/><br/> - **Remplacer un disque de la machine virtuelle**. Si une machine virtuelle existe et qu’elle utilise des disques managés (non chiffrés), vous pouvez restaurer un disque et l’utiliser pour remplacer un disque existant sur la machine virtuelle.<br/><br/> - **Restaurer des fichiers/dossiers spécifiques**. Vous pouvez restaurer des fichiers/dossiers d’une machine virtuelle, au lieu de la totalité de la machine virtuelle.
Sauvegarde directe de machines virtuelles Azure (Windows uniquement)  | Sauvegardez des fichiers/dossiers/volumes spécifiques. | Installation de [l'agent Azure Recovery Services](backup-azure-file-folder-backup-faq.md).<br/><br/> Vous pouvez exécuter l’agent MARS en même temps que l’extension de sauvegarde pour l’agent de machine virtuelle Azure pour sauvegarder la machine virtuelle au niveau des fichiers/dossiers. | Restaurer des fichiers/dossiers spécifiques.
Sauvegarder la machine virtuelle Azure sur le serveur de sauvegarde  | Sauvegarder des fichiers/dossiers/volumes ; état du système/fichiers complets ; données d’application sur System Center DPM ou sur le serveur de sauvegarde Microsoft Azure (MABS).<br/><br/> DPM/MABS effectue ensuite la sauvegarde dans le coffre de sauvegarde. | Installez l’agent de protection DPM/MABS sur la machine virtuelle. L’agent MARS est installé sur DPM/MABS.| Restaurer des fichiers/dossiers/volumes ; état du système/fichiers complets ; données d’application.

En savoir plus sur la sauvegarde [à l’aide d’un serveur de sauvegarde](backup-architecture.md#architecture-back-up-to-dpmmabs) et sur les [prérequis de la prise en charge](backup-support-matrix-mabs-dpm.md).

## <a name="supported-backup-actions"></a>Actions de sauvegarde prises en charge

**Action** | **Support**
--- | ---
Sauvegarder une machine virtuelle arrêtée/hors connexion | Pris en charge.<br/><br/> L’instantané a une cohérence en cas d’incident uniquement, mais pas de cohérence des applications.
Sauvegarder des disques après la migration vers des disques managés | Pris en charge.<br/><br/> La sauvegarde continuera à fonctionner. Aucune action n'est requise.
Sauvegarder des disques managés après l’activation d’un verrou de groupe de ressources | Non pris en charge.<br/><br/> La Sauvegarde Azure ne peut pas supprimer les anciens points de restauration et les sauvegardes commenceront à échouer une fois la limite maximale de points de restauration atteinte.
Modifier la stratégie de sauvegarde pour une machine virtuelle | Pris en charge.<br/><br/> La machine virtuelle est sauvegardée selon la planification et les paramètres de conservation de la nouvelle stratégie. Si les paramètres de conservation sont étendus, les points de récupération existants sont marqués et conservés. S’ils sont réduits, les points de récupération existants sont nettoyés lors de la prochaine tâche de nettoyage et ils sont finalement supprimés.
Annuler un travail de sauvegarde| Pris en charge pendant le processus de capture instantanée.<br/><br/> Non pris en charge quand l’instantané est transféré vers le coffre.
Sauvegarder la machine virtuelle vers une autre région ou un autre abonnement |Non pris en charge.<br><br>Pour réussir la sauvegarde, les machines virtuelles doivent se trouver dans le même abonnement que le coffre de la sauvegarde.
Sauvegardes par jour (via l’extension de machine virtuelle Azure) | Une sauvegarde planifiée par jour.<br/><br/>Le service Sauvegarde Azure prend en charge jusqu’à neuf sauvegardes à la demande par jour, mais Microsoft recommande d’effectuer au maximum quatre sauvegardes quotidiennes à la demande afin de garantir des performances optimales.
Sauvegardes par jour (via l’agent MARS) | Trois sauvegardes planifiées par jour.
Sauvegardes par jour (via DPM/MABS) | Deux sauvegardes planifiées par jour.
Sauvegarde mensuelle/annuelle| Non pris en charge lors de la sauvegarde avec l’extension de machine virtuelle Azure. Seules les sauvegardes quotidiennes et hebdomadaires sont prises en charge.<br/><br/> Vous pouvez configurer la stratégie pour conserver des sauvegardes quotidiennes/hebdomadaires pour une période de conservation mensuelle/annuelle.
Ajustement automatique de l’horloge | Non pris en charge.<br/><br/> La Sauvegarde Azure ne s’ajuste pas automatiquement au changement d’heure lorsque vous sauvegardez une machine virtuelle.<br/><br/>  Modifiez la stratégie manuellement en fonction des besoins.
[Fonctionnalités de sécurité pour la sauvegarde hybride](./backup-azure-security-feature.md) |La désactivation des fonctionnalités de sécurité n’est pas prise en charge.
Sauvegarder la machine virtuelle dont l’heure machine a été modifiée | Non pris en charge.<br/><br/> Si l’heure de la machine est remplacée par une heure ultérieure après l’activation de la sauvegarde pour cette machine virtuelle, et même si le changement d’heure est rétabli, la réussite de la sauvegarde n’est pas garantie.

## <a name="operating-system-support-windows"></a>Prise en charge des systèmes d’exploitation (Windows)

Le tableau suivant récapitule les systèmes d’exploitation pris en charge lors de la sauvegarde de machines virtuelles Azure Windows.

**Scénario** | **Système d’exploitation pris en charge**
--- | ---
Sauvegarder avec l’extension de l’agent de machine virtuelle Azure | - Client Windows 10 (64 bits uniquement) <br/><br/>- Windows Server 2019 (Datacenter/Datacenter Core/Standard) <br/><br/> - Windows Server 2016 (Datacenter/Datacenter Core/Standard) <br/><br/> - Windows Server 2012 R2 (Datacenter/Standard) <br/><br/> - Windows Server 2012 (Datacenter/Standard) <br/><br/> - Windows Server 2008 R2 (RTM et SP1 Standard)  <br/><br/> - Windows Server 2008 (64 bits uniquement)
Sauvegarder avec l’agent MARS | Systèmes d’exploitation [pris en charge](backup-support-matrix-mars-agent.md#supported-operating-systems).
Sauvegarder avec DPM/MABS | Systèmes d’exploitation pris en charge pour la sauvegarde avec [MABS](backup-mabs-protection-matrix.md) et [DPM](/system-center/dpm/dpm-protection-matrix).

Sauvegarde Azure ne prend pas en charge les systèmes d’exploitation 32 bits.

## <a name="support-for-linux-backup"></a>Prise en charge des sauvegardes Linux

Voici ce qui est pris en charge si vous voulez sauvegarder des machines Linux.

**Action** | **Support**
--- | ---
Sauvegarder des machines virtuelles Azure Linux avec l’agent de machine virtuelle Linux Azure Linux | Sauvegarde avec cohérence des fichiers.<br/><br/> Sauvegarde cohérente au niveau application à l’aide de [scripts personnalisés](backup-azure-linux-app-consistent.md).<br/><br/> Pendant la restauration, vous pouvez créer une machine virtuelle, restaurer un disque et l’utiliser pour créer une machine virtuelle, ou restaurer un disque et l’utiliser pour remplacer un disque sur une machine virtuelle existante. Vous pouvez également restaurer des fichiers et des dossiers individuels.
Sauvegarder des machines virtuelles Azure Linux avec l’agent MARS | Non pris en charge.<br/><br/> L’agent MARS ne peut être installé que sur des machines Windows.
Sauvegarder des machines virtuelles Azure Linux avec DPM/MABS | Non pris en charge.

## <a name="operating-system-support-linux"></a>Prise en charge du système d’exploitation (Linux)

Pour les sauvegardes de machines virtuelles Azure Linux, Sauvegarde Azure prend en charge [la liste des distributions Linux approuvées par Azure](../virtual-machines/linux/endorsed-distros.md). Notez les points suivants :

- Sauvegarde Azure ne prend pas en charge CoreOS Linux.
- Sauvegarde Azure ne prend pas en charge les systèmes d’exploitation 32 bits.
- Les autres distributions « Bring-Your-Own-Linux » peuvent fonctionner, dès lors que l’[agent de machine virtuelle Azure pour Linux](../virtual-machines/extensions/agent-linux.md) est disponible sur la machine virtuelle et que Python est pris en charge.
- La Sauvegarde Azure ne prend pas en charge une machine virtuelle Linux configurée avec un proxy si elle n’a pas Python version 2.7 installé.

## <a name="backup-frequency-and-retention"></a>Fréquence et conservation des sauvegardes

**Paramètre** | **Limites**
--- | ---
Nombre maximal de points de récupération par instance protégée (machine/charge de travail) | 9 999.
Délai d’expiration maximal pour un point de récupération | Aucune limite (99 ans).
Fréquence maximale de sauvegarde dans le coffre (extension de machine virtuelle Azure) | Une fois par jour.
Fréquence maximale de sauvegarde dans le coffre (agent MARS) | Trois sauvegardes par jour.
Fréquence de sauvegarde maximale dans DPM/MABS | Toutes les 15 minutes pour SQL Server.<br/><br/> Une fois par heure pour les autres charges de travail.
Conservation des points de récupération | Quotidienne, hebdomadaire, mensuelle et annuelle.
Période de rétention maximale | Dépend de la fréquence de sauvegarde.
Points de récupération sur un disque DPM/MAB | 64 pour les serveurs de fichiers et 448 pour les serveurs d’applications.<br/><br/> Les points de récupération sur bande sont illimités pour les DPM locaux.

## <a name="supported-restore-methods"></a>Méthodes de restauration prises en charge

**Option de restauration** | **Détails**
--- | ---
**Créer une machine virtuelle** | Permet d’avoir rapidement une machine virtuelle de base opérationnelle à partir d’un point de restauration.<br/><br/> Vous pouvez nommer la machine virtuelle, sélectionner le groupe de ressources et le réseau virtuel (VNet) où elle sera placée et spécifier un type de stockage pour la machine virtuelle restaurée. La nouvelle machine virtuelle doit être créée dans la même région que la machine virtuelle source.
**Restaurer un disque** | Restaure un disque de machine virtuelle que vous pouvez ensuite utiliser pour créer une machine virtuelle.<br/><br/> Sauvegarde Azure fournit un modèle pour vous aider à personnaliser et à créer une machine virtuelle. <br/><br> Le travail de restauration génère un modèle que vous pouvez télécharger et utiliser pour spécifier des paramètres de machine virtuelle personnalisés et créer une machine virtuelle.<br/><br/> Les disques sont copiés dans le groupe de ressources spécifié.<br/><br/> Vous pouvez également attacher le disque à une machine virtuelle existante ou créer une machine virtuelle à l’aide de PowerShell.<br/><br/> Cette option est utile si vous souhaitez personnaliser la machine virtuelle, ajouter des paramètres de configuration qui n’existaient pas au moment de la sauvegarde ou encore ajouter des paramètres qui doivent être configurés à l’aide du modèle ou de PowerShell.
**Remplacer l’existant** | Vous pouvez restaurer un disque et l’utiliser pour remplacer un disque sur la machine virtuelle existante.<br/><br/> La machine virtuelle actuelle doit exister. Si elle a été supprimée, vous ne pouvez pas utiliser cette option.<br/><br/> La Sauvegarde Azure prend un instantané de la machine virtuelle existante avant de remplacer le disque et le stocke à l’emplacement intermédiaire spécifié. Les disques existants connectés à la machine virtuelle sont remplacés par le point de restauration sélectionné.<br/><br/> L’instantané est copié dans le coffre et conservé conformément à la stratégie de conservation. <br/><br/> Après l’opération de remplacement du disque, le disque d’origine est conservé dans le groupe de ressources. Vous pouvez choisir de supprimer manuellement les disques d’origine si vous n’en avez pas besoin. <br/><br/>Le remplacement des disques existants est pris en charge pour les machines virtuelles managées non chiffrées et les machines virtuelles [créées à l’aide d’images personnalisées](https://azure.microsoft.com/resources/videos/create-a-custom-virtual-machine-image-in-azure-resource-manager-with-powershell/). Il n’est pas pris en charge pour les disques non managés et les [machines virtuelles généralisées](../virtual-machines/windows/capture-image-resource.md).<br/><br/> Si le point de restauration a plus ou moins de disques que la machine virtuelle actuelle, le nombre de disques du point de restauration reflète alors uniquement la configuration de la machine virtuelle.<br><br> L’option Remplacer l’existant n’est pas prise en charge pour les machines virtuelles avec des ressources liées, par exemple, [Identité managée affectée par l’utilisateur](../active-directory/managed-identities-azure-resources/overview.md) et [Key Vault](../key-vault/general/overview.md).
**Interrégion (région secondaire)** | La restauration interrégion peut être utilisée pour restaurer des machines virtuelles Azure dans la région secondaire, qui est une [région jumelée à Azure](../best-practices-availability-paired-regions.md#what-are-paired-regions).<br><br> Vous pouvez restaurer toutes les machines virtuelles Azure pour le point de récupération sélectionné si la sauvegarde est effectuée dans la région secondaire.<br><br> Cette fonctionnalité est disponible pour les options ci-dessous :<br> <li> [Créer une machine virtuelle](./backup-azure-arm-restore-vms.md#create-a-vm) <br> <li> [Restaurer des disques](./backup-azure-arm-restore-vms.md#restore-disks) <br><br> Nous ne prenons actuellement pas en charge l’option [Remplacer des disques existants](./backup-azure-arm-restore-vms.md#replace-existing-disks).<br><br> Autorisations<br> L’opération de restauration sur la région secondaire peut être effectuée par les administrateurs de sauvegarde et les administrateurs d’applications.

## <a name="support-for-file-level-restore"></a>Prise en charge de la restauration au niveau fichier

**Restauration** | **Pris en charge**
--- | ---
Restauration de fichiers entre systèmes d’exploitation | Vous pouvez restaurer des fichiers sur n’importe quelle machine ayant le même système d’exploitation (ou un système d’exploitation compatible) que la machine virtuelle sauvegardée. Consultez le [Tableau des systèmes d’exploitation compatibles](backup-azure-restore-files-from-vm.md#step-3-os-requirements-to-successfully-run-the-script).
Restauration de fichiers provenant de machines virtuelles chiffrées | Non pris en charge.
Restauration de fichiers provenant de comptes de stockage avec des restrictions réseau | Non pris en charge.
Restauration de fichiers sur des machines virtuelles avec des espaces de stockage Windows | Restauration non prise en charge sur la même machine virtuelle.<br/><br/> Au lieu de cela, restaurez les fichiers sur une machine virtuelle compatible.
Restaurer des fichiers sur une machine virtuelle Linux avec des baies LVM/RAID | Restauration non prise en charge sur la même machine virtuelle.<br/><br/> Restaurer sur une machine virtuelle compatible.
Restaurer des fichiers avec des paramètres réseau spéciaux | Restauration non prise en charge sur la même machine virtuelle. <br/><br/> Restaurer sur une machine virtuelle compatible.

## <a name="support-for-vm-management"></a>Prise en charge de la gestion des machines virtuelles

Le tableau suivant récapitule la prise en charge de la sauvegarde pendant les tâches de gestion des machines virtuelles, comme l’ajout ou le remplacement de disques de machine virtuelle.

**Restauration** | **Pris en charge**
--- | ---
Restaurer entre abonnements/régions/zones. | Non pris en charge.
Restaurer sur une machine virtuelle existante | Utilisez l’option de remplacement de disque.
Restaurer un disque avec un compte de stockage activé pour Azure Storage Service Encryption (SSE) | Non pris en charge.<br/><br/> Restaurez sur un compte où SSE n’est pas est activé.
Restaurer sur des comptes de stockage mixtes |Non pris en charge.<br/><br/> Selon le type de compte de stockage, tous les disques restaurés seront ou Premium ou Standard, mais pas mixtes.
Restaurer une machine virtuelle directement sur un groupe à haute disponibilité | Pour les disques managés, vous pouvez restaurer le disque et utiliser l’option de groupe à haute disponibilité dans le modèle.<br/><br/> Non pris en charge pour les disques non managés. Pour les disques non managés, restaurez le disque, puis créez une machine virtuelle dans le groupe à haute disponibilité.
Restaurer une sauvegarde de machines virtuelles non managées après leur mise à niveau en machines virtuelles managées| Pris en charge.<br/><br/> Vous pouvez restaurer les disques, puis créer une machine virtuelle managée.
Restaurer une machine virtuelle sur un point de restauration antérieur à la migration de la machine virtuelle vers des disques managés | Pris en charge.<br/><br/> Vous restaurez sur des disques non managés (par défaut), vous convertissez les disques restaurés en disques managés, et vous créez une machine virtuelle avec les disques managés.
Restaurer une machine virtuelle qui a été supprimée. | Pris en charge.<br/><br/> Vous pouvez restaurer la machine virtuelle à partir d’un point de récupération.
Restaurer une machine virtuelle contrôleur de domaine  | Pris en charge. Pour plus d’informations, consultez [Restaurer des machines virtuelles contrôleurs de domaine](backup-azure-arm-restore-vms.md#restore-domain-controller-vms).
Restaurer une machine virtuelle dans un autre réseau virtuel |Pris en charge.<br/><br/> Les réseaux virtuels doivent se trouver dans le même abonnement et la même région.

## <a name="vm-compute-support"></a>Prise en charge du calcul de machine virtuelle

**Calcul** | **Support**
--- | ---
Taille de la machine virtuelle |N’importe quelle taille de machine virtuelle Azure avec au moins 2 cœurs d’UC et 1 Go de RAM.<br/><br/> [En savoir plus.](../virtual-machines/sizes.md)
Sauvegarder des machines virtuelles dans des [groupes à haute disponibilité](../virtual-machines/availability.md#availability-sets) | Pris en charge.<br/><br/> Vous ne pouvez pas restaurer une machine virtuelle dans un groupe à haute disponibilité en utilisant l’option de création rapide d’une machine virtuelle. Au lieu de cela, quand vous restaurez la machine virtuelle, restaurez le disque et utilisez-le pour déployer une machine virtuelle, ou bien restaurez un disque et utilisez-le pour remplacer un disque existant.
Sauvegarder des machines virtuelles déployées avec [Hybrid Use Benefit (HUB)](../virtual-machines/windows/hybrid-use-benefit-licensing.md) | Pris en charge.
Sauvegarder des machines virtuelles déployées à partir de la [Place de marché Azure](https://azuremarketplace.microsoft.com/marketplace/apps?filters=virtual-machine-images)<br/><br/> (Publication par Microsoft ou des tiers) |Pris en charge.<br/><br/> La machine virtuelle doit exécuter un système d’exploitation pris en charge.<br/><br/> Lors de la récupération de fichiers sur la machine virtuelle, vous pouvez restaurer seulement sur un système d’exploitation compatible (pas un système d’exploitation antérieur ou ultérieur). Nous ne restaurons pas de machines virtuelles de la Place de marché Azure sauvegardées en tant que machines virtuelles, car celles-ci nécessitent des informations d’achat. Elles sont restaurées uniquement en tant que disques.
Sauvegarder des machines virtuelles déployées à partir d’une image personnalisée (tiers) |Pris en charge.<br/><br/> La machine virtuelle doit exécuter un système d’exploitation pris en charge.<br/><br/> Lors de la récupération de fichiers sur la machine virtuelle, vous pouvez restaurer seulement sur un système d’exploitation compatible (pas un système d’exploitation antérieur ou ultérieur).
Sauvegarder des machines virtuelles migrées vers Azure| Pris en charge.<br/><br/> Pour sauvegarder la machine virtuelle, l’agent de machine virtuelle doit être installé sur la machine migrée.
Cohérence multimachine virtuelle de la sauvegarde | La Sauvegarde Azure ne fournit pas la cohérence des données et des applications sur plusieurs machines virtuelles.
Sauvegarde avec les [Paramètres de diagnostic](../azure-monitor/platform/platform-logs-overview.md)  | Non prise en charge. <br/><br/> Si la restauration de la machine virtuelle Azure avec les paramètres de diagnostic est déclenchée à l’aide de l’option [Créer](backup-azure-arm-restore-vms.md#create-a-vm), la restauration échoue.
Restauration des machines virtuelles épinglées par zone | Prise en charge (pour une machine virtuelle sauvegardée après janvier 2019 et pour laquelle les [zones de disponibilité](https://azure.microsoft.com/global-infrastructure/availability-zones/) sont disponibles).<br/><br/>Nous prenons actuellement en charge la restauration dans la même zone que celle épinglée dans des machines virtuelles. Toutefois, si la zone n’est pas disponible, la restauration échoue.
Machines virtuelles de deuxième génération | Prise en charge <br> Sauvegarde Azure prend en charge la sauvegarde et la restauration des [machines virtuelles de deuxième génération](https://azure.microsoft.com/updates/generation-2-virtual-machines-in-azure-public-preview/). Lorsque ces machines virtuelles sont restaurées à partir du point de récupération, elles sont restaurées sous la forme de [machines virtuelles de deuxième génération](https://azure.microsoft.com/updates/generation-2-virtual-machines-in-azure-public-preview/).
Sauvegarde des machines virtuelles Azure avec des verrous | Non pris en charge pour les machines virtuelles non managées. <br><br> Pris en charge pour les machines virtuelles managées.
[Machines virtuelles Spot](../virtual-machines/spot-vms.md) | Non prise en charge. La sauvegarde Azure restaure les machines virtuelles Sport en tant que machines virtuelles Azure classiques.

## <a name="vm-storage-support"></a>Prise en charge du stockage des machines virtuelles

**Composant** | **Support**
--- | ---
Disques de données de machine virtuelle Azure | Prise en charge de la sauvegarde de machines virtuelles Azure avec jusqu’à 32 disques.<br><br> Prise en charge de la sauvegarde des machines virtuelles Azure avec des disques non managés ou des machines virtuelles classiques avec 16 disques uniquement.
Taille de disque de données | La taille d’un disque individuel peut atteindre jusqu’à 32 To et un maximum de 256 To combinés pour tous les disques d’une machine virtuelle.
Type de stockage | HDD Standard, SSD Standard, SSD Premium.
Disques managés | Pris en charge.
Disques chiffrés | Pris en charge.<br/><br/> Les machines virtuelles Azure activées pour Azure Disk Encryption peuvent être sauvegardées (avec ou sans l’application Azure AD).<br/><br/> Les machines virtuelles chiffrées ne peuvent pas être récupérées au niveau fichier/dossier. Vous devez récupérer la totalité de la machine virtuelle.<br/><br/> Vous pouvez activer le chiffrement sur des machines virtuelles qui sont déjà protégées par Sauvegarde Azure.
Disques avec l’accélérateur d’écriture activé | Depuis le 23 novembre 2020, pris en charge dans les régions Corée Centre (KRC) et Afrique du Sud Nord (SAN).<br/><br/> Sauvegarde Azure sauvegarde les machines virtuelles avec des disques dont l’écriture accélérée (WA) est activée au cours de la sauvegarde.  
Sauvegarder et restaurer des machines virtuelles/disques dédupliqués | Sauvegarde Azure ne prend pas en charge la déduplication. Pour plus d’informations, consultez cet [article](./backup-support-matrix.md#disk-deduplication-support) <br/> <br/>  - Sauvegarde Azure n’effectue pas de déduplication entre les machines virtuelles du coffre Recovery Services <br/> <br/>  - S’il existe des machines virtuelles en état de déduplication pendant la restauration, les fichiers ne peuvent pas être restaurés, car le coffre ne comprend pas le format. Toutefois, vous pouvez effectuer la restauration complète de la machine virtuelle.
Ajouter un disque à une machine virtuelle protégée | Pris en charge.
Redimensionner un disque sur une machine virtuelle protégée | Pris en charge.
Stockage partagé| La sauvegarde des machines virtuelles à l’aide d’un volume partagé de cluster (CSV) ou d’un serveur de fichiers avec montée en puissance parallèle n’est pas prise en charge. En effet, il existe un risque d’échec pour les enregistreurs de volumes partagés de cluster lors de la sauvegarde. Lors de la restauration, les disques contenant des volumes partagés de cluster risquent de ne pas apparaître.
[Disques partagés](../virtual-machines/disks-shared-enable.md) | Non pris en charge.
Disques SSD Ultra | Non pris en charge. Pour plus d’informations, consultez ces [Limitations](selective-disk-backup-restore.md#limitations).

## <a name="vm-network-support"></a>Prise en charge des réseaux de machines virtuelles

**Composant** | **Support**
--- | ---
Nombre d'interfaces réseau (NIC) | Jusqu’au nombre maximal de cartes réseau prises en charge pour une taille spécifique de machine virtuelle Azure.<br/><br/> Les cartes réseau sont créées lors de la création de la machine virtuelle pendant le processus de restauration.<br/><br/> Le nombre de cartes réseau sur la machine virtuelle restaurée correspond au nombre de cartes réseau présentes sur la machine virtuelle quand vous avez activé la protection. La suppression de cartes réseau après l’activation de la protection n’affecte pas leur nombre.
Équilibreur de charge externe/interne |Pris en charge. <br/><br/> [Découvrez plus d’informations](backup-azure-arm-restore-vms.md#restore-vms-with-special-configurations) sur la restauration de machines virtuelles avec des paramètres réseau spéciaux.
Plusieurs adresses IP réservées |Pris en charge. <br/><br/> [Découvrez plus d’informations](backup-azure-arm-restore-vms.md#restore-vms-with-special-configurations) sur la restauration de machines virtuelles avec des paramètres réseau spéciaux.
Machines virtuelles avec plusieurs cartes réseau| Pris en charge. <br/><br/> [Découvrez plus d’informations](backup-azure-arm-restore-vms.md#restore-vms-with-special-configurations) sur la restauration de machines virtuelles avec des paramètres réseau spéciaux.
Machines virtuelles avec des adresses IP publiques| Pris en charge.<br/><br/> Associez une adresse IP publique existante à la carte réseau, ou bien créez une adresse et associez-la à la carte réseau une fois la restauration effectuée.
Groupe de sécurité réseau sur une carte réseau/un sous-réseau. |Pris en charge.
Adresse IP statique | Non pris en charge.<br/><br/> Une machine virtuelle créée à partir d’un point de restauration reçoit une adresse IP dynamique.<br/><br/> Pour les machines virtuelles Classic, vous ne pouvez pas sauvegarder une machine virtuelle avec une adresse IP réservée et sans point de terminaison défini.
Adresse IP dynamique |Pris en charge.<br/><br/> Si la carte réseau sur la machine virtuelle source utilise l’adressage IP dynamique, par défaut, la carte réseau sur la machine virtuelle restaurée l’utilise aussi.
Azure Traffic Manager| Pris en charge.<br/><br/>Si la machine virtuelle sauvegardée est dans Traffic Manager, ajoutez manuellement la machine virtuelle restaurée à la même instance Traffic Manager.
Azure DNS |Pris en charge.
Système DNS personnalisé |Pris en charge.
Connectivité sortante via un proxy HTTP | Pris en charge.<br/><br/> Un proxy authentifié n’est pas pris en charge.
Points de terminaison de service de réseau virtuel| Pris en charge.<br/><br/> Les paramètres du compte de stockage pour le pare-feu et le réseau virtuel doivent autoriser l’accès depuis tous les réseaux.

## <a name="vm-security-and-encryption-support"></a>Prise en charge de la sécurité et du chiffrement des machines virtuelles

La Sauvegarde Azure prend en charge le chiffrement des données en transit et au repos :

Trafic réseau vers Azure :

- Le trafic de sauvegarde entre les serveurs et le coffre Recovery Services est chiffré à l’aide du protocole AES (Advanced Encryption Standard) 256.
- Les données de sauvegarde sont envoyées via une connexion HTTPS sécurisée.
- Les données de sauvegarde sont stockées dans le coffre Recovery Services sous forme chiffrée.
- Vous êtes le seul à connaître la clé de chiffrement pour déverrouiller ces données. Microsoft ne peut déchiffrer les données de sauvegarde à aucun moment.

  > [!WARNING]
  > Une fois le coffre configuré, vous êtes le seul à avoir accès à la clé de chiffrement. Microsoft ne conserve jamais de copie de la clé de chiffrement et n’y a pas accès. Si la clé est égarée, Microsoft ne peut pas récupérer les données de sauvegarde.

Sécurité des données :

- Lors de la sauvegarde de machines virtuelles Azure, vous devez configurer le chiffrement *dans* la machine virtuelle.
- La Sauvegarde Azure prend en charge Azure Disk Encryption, qui utilise BitLocker sur les machines virtuelles Windows et **dm-crypt** sur les machines virtuelles Linux.
- Sur le back end, la sauvegarde Azure utilise le [Chiffrement du service de stockage Azure](../storage/common/storage-service-encryption.md), ce qui protège les données au repos.

**Machine** | **En transit** | **Au repos**
--- | --- | ---
Machines Windows locales sans DPM/MAB | ![Oui][green] | ![Oui][green]
Machines virtuelles Azure | ![Oui][green] | ![Oui][green]
Machines virtuelles locales/Azure avec DPM | ![Oui][green] | ![Oui][green]
Machines virtuelles locales/Azure avec MABS | ![Oui][green] | ![Oui][green]

## <a name="vm-compression-support"></a>Prise en charge de la compression des machines virtuelles

Le service Sauvegarde prend en charge la compression du trafic de sauvegarde, comme décrit dans le tableau suivant. Notez les points suivants :

- Pour les machines virtuelles Azure, l’extension de machine virtuelle lit directement les données à partir du compte de stockage Azure via le réseau de stockage. Il n’est pas nécessaire de compresser ce trafic.
- Si vous utilisez DPM ou MABS, vous pouvez économiser la bande passante en compressant les données avant qu’elles soient sauvegardées dans DPM/MABS.

**Machine** | **Compresser dans MABS/DPM (TCP)** | **Compresser dans le coffre (HTTPS)**
--- | --- | ---
Machines Windows locales sans DPM/MAB | N/D | ![Oui][green]
Machines virtuelles Azure | N/D | N/D
Machines virtuelles locales/Azure avec DPM | ![Oui][green] | ![Oui][green]
Machines virtuelles locales/Azure avec MABS | ![Oui][green] | ![Oui][green]

## <a name="next-steps"></a>Étapes suivantes

- [Sauvegarder des machines virtuelles Azure](backup-azure-arm-vms-prepare.md).
- [Sauvegarder des ordinateurs Windows directement](tutorial-backup-windows-server-to-azure.md), sans serveur de sauvegarde.
- [Configurer MABS](backup-azure-microsoft-azure-backup.md) pour la sauvegarde dans Azure, puis sauvegarder des charges de travail dans MABS.
- [Configurer DPM](backup-azure-dpm-introduction.md) pour la sauvegarde dans Azure, puis sauvegarder des charges de travail dans DPM.

[green]: ./media/backup-support-matrix/green.png
[yellow]: ./media/backup-support-matrix/yellow.png
[red]: ./media/backup-support-matrix/red.png
