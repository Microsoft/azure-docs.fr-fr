---
title: Réponses à des questions fréquentes
description: 'Réponses aux questions courantes sur : les fonctionnalités de la sauvegarde Azure, y compris les coffres Recovery Services ce qu’il peut sauvegarder, son fonctionnement, son chiffrement, et ses limites. '
ms.topic: conceptual
ms.date: 07/07/2019
ms.openlocfilehash: d85866e490b2c56abb7de1e94cd0ffaa8f714615
ms.sourcegitcommit: 4295037553d1e407edeb719a3699f0567ebf4293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96327149"
---
# <a name="azure-backup---frequently-asked-questions"></a>Sauvegarde Azure - Forum Aux Questions

Cet article répond aux questions courantes sur le service Sauvegarde Azure.

## <a name="recovery-services-vault"></a>Coffre Recovery Services

### <a name="is-there-any-limit-on-the-number-of-vaults-that-can-be-created-in-each-azure-subscription"></a>Le nombre de coffres pouvant être créés dans chaque abonnement Azure est-il limité ?

Oui. Vous pouvez créer jusqu’à 500 coffres Recovery Services par région de Sauvegarde Microsoft Azure prise en charge et par abonnement. Au-delà, créez un autre abonnement.

### <a name="are-there-limits-on-the-number-of-serversmachines-that-can-be-registered-against-each-vault"></a>Le nombre de serveurs/ordinateurs pouvant être inscrits dans chaque coffre est-il limité ?

Vous pouvez inscrire jusqu’à 1 000 machines virtuelles Azure par coffre. Si vous utilisez l’agent Sauvegarde Microsoft Azure, vous pouvez inscrire jusqu’à 50 agents MARS par coffre. Et vous pouvez inscrire 50 serveurs MABS/DPM dans un coffre.

### <a name="how-many-datasourcesitems-can-be-protected-in-a-vault"></a>Combien de sources de données/éléments peuvent être protégés dans un coffre ?

Vous pouvez protéger un maximum de 2000 sources de données/éléments sur toutes les charges de travail (machine virtuelle IaaS, SQL ou AFS, par exemple) d'un coffre.
Par exemple, si vous avez déjà protégé 500 machines virtuelles et 400 partages Azure Files dans le coffre, vous ne pouvez y protéger que 1 100 bases de données SQL.

### <a name="how-many-policies-can-i-create-per-vault"></a>Combien de stratégies puis-je créer par coffre ?

Vous ne pouvez avoir que jusqu’à 200 stratégies par coffre.

### <a name="if-my-organization-has-one-vault-how-can-i-isolate-data-from-different-servers-in-the-vault-when-restoring-data"></a>Si mon organisation possède un coffre, comment isoler les données de différents serveurs lors de la restauration des données ?

Les données que vous souhaitez récupérer ensemble doivent utiliser la même phrase secrète lorsque vous configurez la sauvegarde. Si vous souhaitez isoler la récupération vers un serveur ou des serveurs spécifiques, utilisez une phrase secrète unique aux serveurs voulus. Par exemple, les serveurs des ressources humaines peuvent utiliser une phrase secrète de chiffrement, les serveurs de comptabilité peuvent en utiliser une autre et les serveurs de stockage une troisième.

### <a name="can-i-move-my-vault-between-subscriptions"></a>Puis-je déplacer mon coffre d’un abonnement vers un autre ?

Oui. Pour déplacer un coffre Recovery Services, consultez cet [article](backup-azure-move-recovery-services-vault.md)

### <a name="can-i-move-backup-data-to-another-vault"></a>Puis-je déplacer des données de sauvegarde vers un autre coffre ?

Non. Les données de sauvegarde stockées dans un coffre ne peuvent pas être déplacées vers un autre coffre.

### <a name="can-i-change-the-storage-redundancy-setting-after-a-backup"></a>Puis-je modifier le paramètre de redondance du stockage après une sauvegarde ?

Le type de réplication de stockage par défaut est défini sur Stockage géoredondant (GRS). Une fois que vous avez configuré la sauvegarde, l’option de modification est désactivée et non modifiable.

![Type de réplication de stockage](./media/backup-azure-backup-faq/storage-replication-type.png)

Si vous avez déjà configuré la sauvegarde et que vous devez passer de GRS à LRS, consultez [Guide pratique pour passer de GRS à LRS après avoir configuré la sauvegarde](backup-create-rs-vault.md#how-to-change-from-grs-to-lrs-after-configuring-backup).

### <a name="can-i-do-an-item-level-restore-ilr-for-vms-backed-up-to-a-recovery-services-vault"></a>Puis-je effectuer une restauration de niveau élément (ILR) pour les machines virtuelles sauvegardées dans un coffre Recovery Services ?

- La restauration de niveau élément est prise en charge pour les machines virtuelles Azure sauvegardées par la sauvegarde de machine virtuelle Azure. Pour plus d’informations, consultez cet [article](backup-azure-restore-files-from-vm.md)
- La restauration de niveau élément n’est pas prise en charge pour les points de récupération en ligne des machines virtuelles locales sauvegardées par le serveur de sauvegarde Azure (MABS) ou System Center DPM.

### <a name="how-can-i-move-data-from-the-recovery-services-vault-to-on-premises"></a>Comment déplacer des données du coffre Recovery Services vers un emplacement local ?

L’exportation directe de données du coffre Recovery Services vers un emplacement local à l’aide de Data Box n’est pas prise en charge. Les données doivent être restaurées dans un compte de stockage, puis déplacées vers l’emplacement local via [Data Box](../databox/data-box-overview.md) ou [Import/Export](../storage/common/storage-import-export-service.md).

## <a name="azure-backup-agent"></a>Agent Azure Backup

### <a name="where-can-i-find-common-questions-about-the-azure-backup-agent-for-azure-vm-backup"></a>Où puis-je trouver des questions courantes sur l’agent Sauvegarde Azure pour la sauvegarde de machines virtuelles Azure ?

- Pour les agents sur des machines virtuelles Azure, consultez ce [FAQ](backup-azure-vm-backup-faq.md).
- Pour les agents permettant de sauvegarde des dossiers de fichiers Azure, lisez ce [FAQ](backup-azure-file-folder-backup-faq.md).

## <a name="general-backup"></a>Sauvegarde générale

### <a name="are-there-limits-on-backup-scheduling"></a>Existe-t-il des limites sur la planification de la sauvegarde ?

Oui.

- Vous pouvez sauvegarder des machines Windows Server ou Windows jusqu’à trois fois par jour. Vous pouvez définir la stratégie de planification pour des planifications quotidiennes ou hebdomadaires.
- Vous pouvez sauvegarder DPM jusqu’à deux fois par jour. Vous pouvez définir la stratégie de planification pour des sauvegardes quotidiennes, hebdomadaires, mensuelles et annuelles.
- Vous sauvegardez les machines virtuelles Azure une fois par jour.

### <a name="what-operating-systems-are-supported-for-backup"></a>Quels sont les systèmes d’exploitation pris en charge par la sauvegarde ?

Sauvegarde Azure prend en charge les systèmes d’exploitation suivants pour la sauvegarde des dossiers et des fichiers, ainsi que des applications protégées par Azure Backup Server et DPM.

**SE** | **Référence (SKU)** | **Détails**
--- | --- | ---
Station de travail | |
Windows 10 64 bits | Entreprise, Professionnel, Familiale | Les machines virtuelles doivent utiliser les derniers service packs et les dernières mises à jour.
Windows 8.1 64 bits | Entreprise, Professionnel | Les machines virtuelles doivent utiliser les derniers service packs et les dernières mises à jour.
Windows 8 64 bits | Entreprise, Professionnel | Les machines virtuelles doivent utiliser les derniers service packs et les dernières mises à jour.
Windows 7 64 bits | Édition Intégrale, Entreprise, Professionnel, Édition Familiale Premium, Édition Familiale Basique, Édition Starter | Les machines virtuelles doivent utiliser les derniers service packs et les dernières mises à jour.
Serveur | |
Windows Server 2019 64 bits | Standard, Datacenter, Essentials | Avec les derniers Service packs et dernières mises à jour.
Windows Server 2016 64 bits | Standard, Datacenter, Essentials | Avec les derniers Service packs et dernières mises à jour.
Windows Server 2012 R2 64 bits | Standard, Datacenter, Foundation | Avec les derniers Service packs et dernières mises à jour.
Windows Server 2012 64 bits | Datacenter, Foundation, Standard | Avec les derniers Service packs et dernières mises à jour.
Windows Storage Server 2016 64 bits | Standard, Workgroup | Avec les derniers Service packs et dernières mises à jour.
Windows Storage Server 2012 R2 64 bits | Standard, Workgroup, Essential | Avec les derniers Service packs et dernières mises à jour.
Windows Storage Server 2012 64 bits | Standard, Workgroup | Avec les derniers Service packs et dernières mises à jour.
Windows Server 2008 R2 SP1 64 bits | Standard, Entreprise, Datacenter, Foundation | Avec les dernières mises à jour.
Windows Server 2008 64 bits | Standard, Entreprise, Datacenter | Avec les dernières mises à jour.

Sauvegarde Azure ne prend pas en charge les systèmes d’exploitation 32 bits.

Pour les sauvegardes Linux de machine virtuelle Azure, la Sauvegarde Azure prend en charge [la liste de distributions approuvées par Azure](../virtual-machines/linux/endorsed-distros.md), à l’exception de CoreOS Linux et du système d’exploitation 32 bits. Les autres distributions « Bring-Your-Own-Linux » peuvent fonctionner, tant que l’agent de machine virtuelle est disponible sur la machine virtuelle et que Python est pris en charge.

### <a name="are-there-size-limits-for-data-backup"></a>Y-a-t-il une limite de taille pour la sauvegarde de données ?

Les limites de tailles sont les suivantes :

Système d’exploitation/machine | Limite de taille de la source de données
--- | ---
Windows 8 ou version ultérieure | 54 400 Go
Windows 7 |1 700 Go
Windows Server 2012 ou version ultérieure | 54 400 Go
Windows Server 2008, Windows Server 2008 R2 | 1 700 Go
Azure VM | Consultez la [matrice de prise en charge de la sauvegarde de machines virtuelles Azure](./backup-support-matrix-iaas.md#vm-storage-support)

### <a name="how-is-the-data-source-size-determined"></a>Comment la taille de la source de données est-elle déterminée ?

Le tableau suivant explique comment la taille de chaque source de données est déterminée.

**Source de données** | **Détails**
--- | ---
Volume |Quantité de données sauvegardées à partir d’une machine virtuelle à volume unique.
Base de données SQL Server |Taille d’une base de données unique sauvegardée.
SharePoint | Somme des bases de données de contenu et de configuration dans une batterie de serveurs SharePoint sauvegardée.
Exchange |Somme de toutes les bases de données Exchange sur un serveur Exchange en cours de sauvegarde.
État système/récupération complète |Chaque copie individuelle de l’état système/récupération complète de l’ordinateur sauvegardée.

### <a name="is-there-a-limit-on-the-amount-of-data-backed-up-using-a-recovery-services-vault"></a>La quantité de données sauvegardées dans un coffre Recovery Services est-elle limitée ?

Il n’existe aucune limite pour la quantité totale de données que vous pouvez sauvegarder dans un coffre Recovery Services. Les sources de données individuelles (autres que les machines virtuelles Azure) peuvent avoir une taille maximale de 54 400 Go. Pour plus d’informations sur les limites, consultez la [section relative aux limites du coffre dans la matrice de prise en charge](./backup-support-matrix.md#vault-support).

### <a name="why-is-the-size-of-the-data-transferred-to-the-recovery-services-vault-smaller-than-the-data-selected-for-backup"></a>Pourquoi la taille des données transférées dans le coffre Recovery Services est-elle plus réduite que celle des données sélectionnées pour la sauvegarde ?

Les données sauvegardées à partir de l’agent Azure Backup, de DPM et du serveur de sauvegarde Azure sont compressées et chiffrées avant d’être transférées. Une fois la compression et le chiffrement appliqués, les données dans le coffre sont inférieures de 30 à 40 %.

### <a name="can-i-delete-individual-files-from-a-recovery-point-in-the-vault"></a>Puis-je supprimer des fichiers individuels à partir d’un point de récupération dans le coffre ?

Non. La Sauvegarde Azure ne prend pas en charge la suppression ou le vidage des éléments individuels à partir de sauvegardes stockées.

### <a name="if-i-cancel-a-backup-job-after-it-starts-is-the-transferred-backup-data-deleted"></a>Si j’annule une opération de sauvegarde après son démarrage, les données de sauvegarde transférées sont-elles supprimées ?

Non. Toutes les données transférées dans le coffre avant l’annulation de l’opération de sauvegarde sont conservées.

- Azure Backup utilise un mécanisme de point de contrôle pour ajouter occasionnellement des points de contrôle aux données de sauvegarde pendant la sauvegarde.
- En présence de points de contrôle dans les données de sauvegarde le processus de sauvegarde suivant est en mesure de valider l’intégrité des fichiers.
- La prochaine tâche de sauvegarde sera incrémentielle par rapport aux données précédemment sauvegardées. Les sauvegardes incrémentielles transfèrent uniquement les données nouvelles ou modifiées, ce qui équivaut à une meilleure utilisation de la bande passante.

Si vous annulez une tâche de sauvegarde pour une machine virtuelle Azure, toutes les données transférées sont ignorées. La tâche de sauvegarde suivante transfère des données incrémentielles à partir de la dernière sauvegarde réussie.

## <a name="retention-and-recovery"></a>Rétention et récupération

### <a name="are-the-retention-policies-for-dpm-and-windows-machines-without-dpm-the-same"></a>Les stratégies de rétention pour les ordinateurs DPM et Windows sans DPM sont-elles identiques ?

Oui, elles ont toutes les deux des stratégies quotidienne, hebdomadaire, mensuelle et annuelle de rétention.

### <a name="can-i-customize-retention-policies"></a>Puis-je personnaliser les stratégies de rétention ?

Oui, vous pouvez personnaliser les stratégies. Par exemple, vous pouvez configurer une rétention hebdomadaire et quotidienne, mais pas annuelle ou mensuelle.

### <a name="can-i-use-different-times-for-backup-scheduling-and-retention-policies"></a>Puis-je utiliser des heures différentes pour la planification de la sauvegarde et les stratégies de rétention ?

Non. Les stratégies de rétention ne peuvent être appliquées que sur les points de sauvegarde. Par exemple, cette image montre une stratégie de rétention pour les sauvegardes effectuées à 0h00 et 18h00.

![Planification de sauvegarde et rétention](./media/backup-azure-backup-faq/Schedule.png)

### <a name="if-a-backup-is-kept-for-a-long-time-does-it-take-more-time-to-recover-an-older-data-point"></a>Si une sauvegarde est conservée pendant longtemps, faut-il plus de temps pour récupérer un point de données plus ancien ?

Non. Le délai de récupération reste identique pour les points de données récents ou anciens. Chaque point de récupération se comporte comme un point complet.

### <a name="if-each-recovery-point-is-like-a-full-point-does-it-impact-the-total-billable-backup-storage"></a>Si chaque point de récupération est un point complet, cela a-t-il un impact sur la quantité totale de stockage de sauvegarde facturable ?

Les produits avec points de rétention à long terme stockent les données de sauvegarde en tant que points complets.

- Toutefois, même si ces points *occupent* de l’espace de stockage, ils sont plus faciles et plus rapides à récupérer.
- Les copies incrémentielles *occupent moins d’espace de stockage* , mais vous devez restaurer une chaîne de données, ce qui rallonge le temps de récupération.

L’architecture de stockage d’Azure Backup vous offre le meilleur des deux en stockant les données de manière optimale pour des restaurations rapides et des coûts de stockage faibles. Cette approche garantit que votre bande passante entrante et sortante est utilisée de façon efficace. La quantité de stockage de données et le temps nécessaire pour récupérer les données sont tous les deux réduits au minimum. En savoir plus sur [les sauvegardes incrémentielles](backup-architecture.md#backup-types).

### <a name="is-there-a-limit-on-the-number-of-recovery-points-that-can-be-created"></a>Le nombre de points de récupération pouvant être créés est-il limité ?

Vous pouvez créer jusqu’à 9 999 points de récupération par instance protégée. Une instance protégée est un ordinateur, un serveur (physique ou virtuel) ou une charge de travail sauvegardé dans Azure.

- En savoir plus sur la [sauvegarde et la rétention](./backup-support-matrix.md).

### <a name="how-many-times-can-i-recover-data-thats-backed-up-to-azure"></a>Combien de fois puis-je récupérer des données sauvegardées dans Azure ?

Il n’existe aucune limite concernant le nombre de récupérations dans Azure Backup.

### <a name="when-restoring-data-do-i-pay-for-the-egress-traffic-from-azure"></a>En cas de restauration de données, dois-je payer pour le trafic de sortie à partir d’Azure ?

Non. La récupération est gratuite et le trafic sortant ne vous est pas facturé.

### <a name="what-happens-when-i-change-my-backup-policy"></a>Que se passe-t-il lorsque je modifie ma stratégie de sauvegarde ?

Lorsqu’une nouvelle stratégie est appliquée, le planning et la rétention de la nouvelle stratégie sont suivis.

- Si la rétention est étendue, les points de récupération existants sont marqués comme étant à conserver, selon la nouvelle stratégie.
- Si la rétention est réduite, ils sont marqués comme à nettoyer lors de la prochaine tâche de nettoyage et sont ensuite supprimés.

### <a name="how-long-is-data-retained-when-stopping-backups-but-selecting-the-option-to-retain-backup-data"></a>Pendant combien de temps les données sont-elles conservées lors de l’arrêt des sauvegardes, avec l’option de conservation des données de sauvegarde sélectionnée ?

Quand des sauvegardes sont arrêtées et les données conservées, les règles de stratégie existantes pour le nettoyage cessent de s’appliquer et les données sont conservées indéfiniment jusqu’à ce que l’administrateur initialise leur suppression.

## <a name="encryption"></a>Chiffrement

### <a name="is-the-data-sent-to-azure-encrypted"></a>Les données envoyées à Azure sont-elles chiffrées ?

Oui. Les données sont chiffrées sur l’ordinateur local avec AES256. Les données sont envoyées via une connexion HTTPS sécurisée. Les données transmises dans le cloud sont protégées par une liaison HTTPS uniquement entre le stockage et le service de récupération. Le protocole iSCSI sécurise les données transmises entre l’ordinateur de l’utilisateur et le service de récupération. Le tunneling sécurisé est utilisé pour protéger le canal iSCSI.

### <a name="is-the-backup-data-on-azure-encrypted-as-well"></a>Les données de sauvegarde sont-elles également chiffrées dans Azure ?

Oui. Les données dans Azure sont chiffrées au repos.

- Pour la sauvegarde sur site, le chiffrement au repos est assuré à l’aide de la phrase secrète que vous fournissez lorsque vous sauvegardez sur Azure.
- Pour les machines virtuelles Azure, les données sont chiffrées au repos à l’aide de Storage Service Encryption (SSE).

À aucun moment Microsoft ne déchiffre les données de sauvegarde.

### <a name="what-is-the-minimum-length-of-the-encryption-key-used-to-encrypt-backup-data"></a>Quelle est la longueur minimale de la clé de chiffrement utilisée pour chiffrer les données de sauvegarde ?

La clé de chiffrement utilisée par l’agent Microsoft Azure Recovery Services (MARS) est dérivée d’une phrase secrète qui doit comporter au moins 16 caractères. Pour les machines virtuelles Azure, il n’existe aucune limite à la longueur des clés qu’Azure Key Vault utilise.

### <a name="what-happens-if-i-misplace-the-encryption-key-can-i-recover-the-data-can-microsoft-recover-the-data"></a>Que se passe-t-il si j’ai égaré la clé de chiffrement ? Puis-je récupérer les données ? Microsoft peut-il récupérer les données ?

La clé utilisée pour chiffrer les données de sauvegarde est disponible uniquement sur votre site. Microsoft ne conserve pas de copie dans Azure et n'a pas accès à la clé. Si la clé est égarée, Microsoft ne peut pas récupérer les données de sauvegarde.

## <a name="next-steps"></a>Étapes suivantes

Lire les autres forums aux questions :

- [Questions courantes](backup-azure-vm-backup-faq.md) sur la sauvegarde des machines virtuelles Azure.
- [Questions courantes](backup-azure-file-folder-backup-faq.md) sur l’agent Sauvegarde Azure.