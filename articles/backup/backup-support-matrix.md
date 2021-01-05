---
title: Tableau de prise en charge de Sauvegarde Azure
description: Fournit un résumé des limitations et des paramètres de prise en charge pour le service Sauvegarde Azure.
ms.topic: conceptual
ms.date: 02/17/2019
ms.custom: references_regions
ms.openlocfilehash: 70a81b42ad2fb417902a627afb1651d572844648
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96548509"
---
# <a name="support-matrix-for-azure-backup"></a>Matrice de prise en charge pour Sauvegarde Azure

[Sauvegarde Azure](backup-overview.md) permet de sauvegarder des données sur la plateforme cloud Microsoft Azure. Cet article récapitule les limitations et les paramètres généraux de prise en charge pour les scénarios et les déploiements de Sauvegarde Azure.

Autres matrices de prise en charge disponibles :

- [Tableau de prise en charge](backup-support-matrix-iaas.md) de la sauvegarde de machines virtuelles Azure
- Tableau de prise en charge de la sauvegarde à l'aide de [System Center Data Protection Manager (DPM)/Microsoft Azure Backup Server (MABS)](backup-support-matrix-mabs-dpm.md)
- Tableau de prise en charge de la sauvegarde à l'aide de l'[agent MARS (Microsoft Azure Recovery Services)](backup-support-matrix-mars-agent.md)

[!INCLUDE [azure-lighthouse-supported-service](../../includes/azure-lighthouse-supported-service.md)]

## <a name="vault-support"></a>Prise en charge des coffres

Sauvegarde Azure utilise des coffres Recovery Services pour orchestrer et gérer les sauvegardes des types de charges de travail suivants : machines virtuelles Azure, SQL sur des machines virtuelles Azure, SAP HANA sur des machines virtuelles Azure, partages de fichiers Azure et charges de travail locales à l’aide de l’agent Sauvegarde Azure, du serveur de sauvegarde Azure et de System Center DPM. Il utilise également des coffres Recovery Services pour stocker les données de sauvegarde de ces charges de travail.

Le tableau suivant décrit les fonctionnalités des coffres Recovery Services :

**Fonctionnalité** | **Détails**
--- | ---
**Coffres dans l’abonnement** | Jusqu’à 500 coffres Recovery Services dans un même abonnement.
**Machines dans un coffre** | Jusqu’à 2 000 sources de donnée sur toutes les charges de travail (comme les machines virtuelles Azure, les machines virtuelles SQL Server, les serveurs MABS, etc.) peuvent être protégées dans un coffre unique.<br><br>Jusqu’à 1 000 machines virtuelles Azure dans un même coffre.<br/><br/> Jusqu’à 50 serveurs MABS peuvent être inscrits dans un seul coffre.
**Sources de données** | La taille maximale d'une [source de données](./backup-azure-backup-faq.md#how-is-the-data-source-size-determined) est de 54 400 Go. La limite ne s’applique pas aux sauvegardes de machines virtuelles Azure. Aucune limite ne s'applique au volume total de données que vous pouvez sauvegarder dans le coffre.
**Sauvegardes dans le coffre** | **Machines virtuelles Azure :** Une fois par jour.<br/><br/>**Machines protégées par DPM/MABS :** Deux fois par jour.<br/><br/> **Machines sauvegardées directement à l’aide de l’agent MARS :** Trois fois par jour.
**Sauvegardes entre les coffres** | La sauvegarde s’effectue dans une région.<br/><br/> Vous avez besoin d’un coffre dans chaque région Azure qui contient les machines virtuelles que vous souhaitez sauvegarder. Vous ne pouvez pas sauvegarder vers une autre région.
**Déplacer les coffres** | Vous pouvez [déplacer les coffres](./backup-azure-move-recovery-services-vault.md) entre des abonnements ou entre des groupes de ressources dans le même abonnement. En revanche, le déplacement de coffres entre régions n’est pas pris en charge.
**Déplacer des données entre des coffres** | Le déplacement de données sauvegardées entre des coffres n’est pas pris en charge.
**Modifier le type de stockage de coffre** | Vous pouvez modifier le type de réplication de stockage (stockage géoredondant ou stockage localement redondant) pour un coffre avant le stockage des sauvegardes. Une fois que les sauvegardes commencent dans le coffre, le type de réplication ne peut pas être modifié.
**Stockage redondant interzone (ZRS)** | Disponible dans les régions Royaume-Uni Sud (UKS) et Asie Sud-Est (SEA).

## <a name="on-premises-backup-support"></a>Prise en charge des sauvegardes locales

Voici ce qui est pris en charge si vous voulez sauvegarder des machines locales :

**Machine** | **Éléments sauvegardés** | **Lieu** | **Caractéristiques**
--- | --- | --- | ---
**Sauvegarde directe de l’ordinateur Windows avec l’agent MARS** | Fichiers, dossiers, état du système | Sauvegarde dans le coffre Recovery Services | Sauvegarde trois fois par jour<br/><br/> Pas de sauvegarde tenant compte des applications<br/><br/> Restaurer le fichier, le dossier, le volume
**Sauvegarde directe de l’ordinateur Linux avec l’agent MARS** | Sauvegarde non prise en charge
**Sauvegarde dans DPM** | Fichiers, dossiers, volumes, état du système, données d’application | Sauvegarde dans le stockage DPM local. DPM, puis sauvegarde dans le coffre. | Captures instantanées tenant compte des applications<br/><br/> Précision totale pour la sauvegarde et la récupération<br/><br/> Linux pris en charge pour les machines virtuelles (Hyper-V/VMware)<br/><br/> Oracle non pris en charge
**Sauvegarde dans MABS** | Fichiers, dossiers, volumes, état du système, données d’application | Sauvegarde dans le stockage MABS local. MABS effectue ensuite la sauvegarde dans le coffre. | Captures instantanées tenant compte des applications<br/><br/> Précision totale pour la sauvegarde et la récupération<br/><br/> Linux pris en charge pour les machines virtuelles (Hyper-V/VMware)<br/><br/> Oracle non pris en charge

## <a name="azure-vm-backup-support"></a>Prise en charge de la sauvegarde de machines virtuelles Azure

### <a name="azure-vm-limits"></a>Limites des machines virtuelles Azure

**Limite** | **Détails**
--- | ---
**Disques de données de machine virtuelle Azure** | Consultez la [matrice de prise en charge de la sauvegarde de machines virtuelles Azure](./backup-support-matrix-iaas.md#vm-storage-support).
**Taille de disque de données de machine virtuelle Azure** | La taille d’un disque individuel peut atteindre jusqu’à 32 To et un maximum de 256 To combinés pour tous les disques d’une machine virtuelle.

### <a name="azure-vm-backup-options"></a>Options de sauvegarde d’une machine virtuelle Azure

Voici ce qui est pris en charge si vous voulez sauvegarder des machines virtuelles Azure :

**Machine** | **Éléments sauvegardés** | **Lieu** | **Caractéristiques**
--- | --- | --- | ---
**Sauvegarde de machine virtuelle Azure à l’aide de l’extension de machine virtuelle** | Machine virtuelle complète | Sauvegarder vers le coffre. | Extension installée lorsque vous activez la sauvegarde pour une machine virtuelle.<br/><br/> Sauvegarde une fois par jour.<br/><br/> Sauvegarde tenant compte des applications pour les machines virtuelles Windows, sauvegarde cohérente au niveau fichier pour les machines virtuelles Linux. Vous pouvez configurer la cohérence des applications pour les machines Linux à l’aide de scripts personnalisés.<br/><br/> Restaurez une machine virtuelle ou un disque.<br/><br/> Sauvegarde d'une machine virtuelle Azure impossible vers un emplacement local.
**Sauvegarde de machine virtuelle Azure à l’aide de l’agent MARS** | Fichiers, dossiers, état du système | Sauvegarder vers le coffre. | Sauvegarde trois fois par jour.<br/><br/> Si vous souhaitez sauvegarder des fichiers ou dossiers spécifiques plutôt que la machine virtuelle entière, l’agent MARS peut s’exécuter en même temps que l’extension de machine virtuelle .
**Machine virtuelle Azure avec DPM** | Fichiers, dossiers, volumes, état du système, données d’application | Sauvegarde dans le stockage local de la machine virtuelle Azure exécutant DPM. DPM, puis sauvegarde dans le coffre. | Captures instantanées tenant compte des applications.<br/><br/> Précision totale pour la sauvegarde et la récupération.<br/><br/> Linux pris en charge pour les machines virtuelles (Hyper-V/VMware).<br/><br/> Oracle non pris en charge.
**Machine virtuelle Azure avec MABS** | Fichiers, dossiers, volumes, état du système, données d’application | Sauvegarde dans le stockage local de la machine virtuelle Azure exécutant MABS. MABS effectue ensuite la sauvegarde dans le coffre. | Captures instantanées tenant compte des applications.<br/><br/> Précision totale pour la sauvegarde et la récupération.<br/><br/> Linux pris en charge pour les machines virtuelles (Hyper-V/VMware).<br/><br/> Oracle non pris en charge.

## <a name="linux-backup-support"></a>Prise en charge de la sauvegarde Linux

Voici ce qui est pris en charge si vous voulez sauvegarder des machines Linux :

**Type de sauvegarde** | **Linux (approuvé par Azure)**
--- | ---
**Sauvegarde directe de la machine locale exécutant Linux** | Non pris en charge. L’agent MARS ne peut être installé que sur des machines Windows.
**Utilisation de l'extension de l'agent pour sauvegarder la machine virtuelle Azure exécutant Linux** | Sauvegarde cohérente au niveau application à l’aide de [scripts personnalisés](backup-azure-linux-app-consistent.md).<br/><br/> Récupération au niveau fichier.<br/><br/> Restaurer en créant une machine virtuelle à partir d’un point de récupération ou d’un disque.
**Utilisation de DPM pour sauvegarder des machines locales exécutant Linux** | Sauvegarde cohérente au niveau fichier de machines virtuelles invitées Linux sur Hyper-V et VMware.<br/><br/> Restauration de machines virtuelles invitées Linux Hyper-V et VMware.
**Utilisation de MABS pour sauvegarder des machines locales exécutant Linux** | Sauvegarde cohérente au niveau fichier de machines virtuelles invitées Linux sur Hyper-V et VMware.<br/><br/> Restauration de machines virtuelles invitées Linux Hyper-V et VMware.
**Utilisation de MABS ou de DPM pour sauvegarder des machines virtuelles Linux Azure** | Non pris en charge.

## <a name="daylight-saving-time-support"></a>Prise en charge de l’heure d’été

Sauvegarde Azure ne prend pas en charge l’ajustement automatique de l’horloge lors du passage à l’heure d’été pour les sauvegardes de machines virtuelles Azure. Il ne retarde pas et n’avance pas l’heure de la sauvegarde. Pour vous assurer que la sauvegarde s’exécute à l’heure souhaitée, modifiez les stratégies de sauvegarde manuellement en fonction des besoins.

## <a name="disk-deduplication-support"></a>Prise en charge de la déduplication de disque

La prise en charge de la déduplication de disque est la suivante :

- La déduplication de disque est prise en charge localement quand vous utilisez DPM ou MABS pour sauvegarder des machines virtuelles Hyper-V exécutant Windows. Windows Server effectue la déduplication (au niveau de l’hôte) sur les disques durs virtuels attachés en tant que stockage de sauvegarde à la machine virtuelle.
- La déduplication n’est prise en charge dans Azure pour aucun des composants de Sauvegarde. Lorsque DPM et MABS sont déployés dans Azure, les disques de stockage connectés à la machine virtuelle ne peuvent pas être dédupliqués.

## <a name="security-and-encryption-support"></a>Prise en charge de la sécurité et du chiffrement

Sauvegarde Azure prend en charge le chiffrement des données en transit et au repos.

### <a name="network-traffic-to-azure"></a>Trafic réseau vers Azure

- Le trafic de sauvegarde entre les serveurs et le coffre Recovery Services est chiffré à l’aide du protocole AES (Advanced Encryption Standard) 256.
- Les données de sauvegarde sont envoyées via une connexion HTTPS sécurisée.

### <a name="data-security"></a>Sécurité des données

- Les données de sauvegarde sont stockées dans le coffre Recovery Services sous forme chiffrée.
- Quand les données sont sauvegardées à partir de serveurs locaux avec l’agent MARS, les données sont chiffrées avec une phrase secrète avant le chargement vers Sauvegarde Azure et déchiffrées uniquement après avoir été téléchargées à partir de Sauvegarde Azure.
- Lorsque vous sauvegardez des machines virtuelles Azure, vous devez configurer le chiffrement *dans* la machine virtuelle.
- La sauvegarde Azure prend en charge Azure Disk Encryption, qui utilise BitLocker sur les machines virtuelles Windows et **dm-crypt** sur les machines virtuelles Linux.
- Sur le back end, Sauvegarde Azure utilise [Azure Storage Service Encryption](../storage/common/storage-service-encryption.md), ce qui protège les données au repos.

**Machine** | **En transit** | **Au repos**
--- | --- | ---
**Machines Windows locales sans DPM/MABS** | ![Oui][green] | ![Oui][green]
**Machines virtuelles Azure** | ![Oui][green] | ![Oui][green]
**Machines Windows locales/machines virtuelles Azure avec DPM** | ![Oui][green] | ![Oui][green]
**Machines Windows locales ou machines virtuelles Azure avec MABS** | ![Oui][green] | ![Oui][green]

## <a name="compression-support"></a>Prise en charge de la compression

Le service Sauvegarde prend en charge la compression du trafic de sauvegarde, comme décrit dans le tableau suivant.

- Pour les machines virtuelles Azure, l’extension de machine virtuelle lit directement les données à partir du compte de stockage Azure via le réseau de stockage. Il n’est donc pas nécessaire de compresser ce trafic.
- Si vous utilisez DPM ou MABS, vous pouvez économiser la bande passante en compressant les données avant qu’elles soient sauvegardées.

**Machine** | **Compresser dans MABS/DPM (TCP)** | **Compresser dans le coffre (HTTPS)**
--- | --- | ---
**Sauvegarde directe des ordinateurs Windows locaux** | N/D | ![Oui][green]
**Sauvegarde de machines virtuelles Azure à l’aide de l’extension de machine virtuelle** | N/D | N/D
**Sauvegarde sur des machines locales/Azure à l’aide de MABS/DPM** | ![Oui][green] | ![Oui][green]

## <a name="retention-limits"></a>Limites de rétention

**Paramètre** | **Limites**
--- | ---
**Nombre maximum de points de récupération par instance protégée (machine ou charge de travail)** | 9 999
**Délai d’expiration maximal pour un point de récupération** | Aucune limite
**Fréquence de sauvegarde maximale dans DPM/MABS** | Toutes les 15 minutes pour SQL Server<br/><br/> Une fois par heure pour les autres charges de travail
**Fréquence de sauvegarde maximale dans le coffre** | **Machines Windows locales/machines virtuelles Azure exécutant MARS :** trois fois par jour<br/><br/> **DPM/MABS :** deux fois par jour<br/><br/> **Sauvegarde des machines virtuelles Azure :** une fois par jour
**Conservation des points de récupération** | Quotidienne, hebdomadaire, mensuelle, annuelle
**Période de rétention maximale** | Dépend de la fréquence de sauvegarde
**Points de récupération sur un disque DPM/MAB** | 64 pour les serveurs de fichiers ; 448 pour les serveurs d’applications <br/><br/>Points de récupération sur bande illimités pour les DPM locaux

## <a name="cross-region-restore"></a>Restauration inter-régions

Une fonctionnalité de restauration inter-régions a été ajoutée au service Sauvegarde Azure pour renforcer la disponibilité des données et la capacité de résilience, ce qui vous confère un contrôle total pour restaurer des données dans une région secondaire. Pour configurer cette fonctionnalité, consultez l’article [Définir la restauration inter-région](backup-create-rs-vault.md#set-cross-region-restore). Cette fonctionnalité est prise en charge pour les types de gestion suivants :

| Type de gestion des sauvegardes | Prise en charge                                                    | Régions prises en charge |
| ---------------------- | ------------------------------------------------------------ | ----------------- |
| Azure VM               | Oui.   Prise en charge pour les machines virtuelles chiffrées et les machines virtuelles disposant de disques de moins de 4 To | Toutes les régions publiques Azure.  |
| SQL /SAP HANA | Oui                                                          | Toutes les régions publiques, sauf la France |
| Agent MARS/En local  | Non                                                           | N/A               |
| AFS (partages de fichiers Azure)                 | Non                                                           | N/A               |

## <a name="next-steps"></a>Étapes suivantes

- [Passez en revue la matrice de prise en charge](backup-support-matrix-iaas.md) de la sauvegarde des machines virtuelles Azure.

[green]: ./media/backup-support-matrix/green.png
[yellow]: ./media/backup-support-matrix/yellow.png
[red]: ./media/backup-support-matrix/red.png
