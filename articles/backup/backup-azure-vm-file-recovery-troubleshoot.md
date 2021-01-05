---
title: Résoudre les problèmes de récupération de fichiers d’une machine virtuelle Azure
description: Résolvez les problèmes propres à la récupération de fichiers et de dossiers à partir d’une sauvegarde de machine virtuelle Azure.
ms.reviewer: srinathv
ms.topic: troubleshooting
ms.date: 07/12/2020
ms.openlocfilehash: 96876415405cc5099b4d9ca209c36086b793d6e0
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97512055"
---
# <a name="troubleshooting-issues-in-file-recovery-of-azure-vm-backup"></a>Résolution des problèmes de récupération de fichiers d’une sauvegarde de machine virtuelle Azure

Cet article présente les étapes de dépannage qui peuvent vous aider à résoudre les erreurs de Sauvegarde Azure liées aux problèmes propres à la récupération de fichiers et de dossiers à partir d’une sauvegarde de machine virtuelle Azure. 

## <a name="exception-caught-while-connecting-to-target"></a>Exception interceptée lors de la connexion à la cible

Cause possible : Le script ne peut pas accéder au point de récupération Action recommandée : Vérifiez si la machine remplit toutes les [conditions d’accès](https://docs.microsoft.com/azure/backup/backup-azure-restore-files-from-vm#step-4-access-requirements-to-successfully-run-the-script).

## <a name="the-target-has-already-been-logged-in-via-an-iscsi-session"></a>La cible a déjà été connectée via une session iSCSI

Cause possible : Le script a déjà été exécuté sur la même machine et les lecteurs ont été attachés.
Action recommandée : Les volumes du point de récupération ont déjà été connectés. Ils ne peuvent pas être montés avec les mêmes lettres de lecteur que celles de la machine virtuelle d’origine. Parcourez tous les volumes disponibles dans l’Explorateur de fichiers.

## <a name="this-script-is-invalid-because-the-disks-have-been-dismounted-via-portalexceeded-the-12-hr-limit-download-a-new-script-from-the-portal"></a>Ce script n’est pas valide, car les disques ont été démontés via le portail/ont dépassé la limite de 12 h. Télécharger un nouveau script à partir du portail

Cause possible : Les disques ont été démontés à partir du portail ou la limite de 12 heures a été dépassée.
Action recommandée : Le script n’est plus valide au-delà de 12 heures après son téléchargement et ne peut pas être exécuté. Accédez au portail et téléchargez un nouveau script pour procéder à la récupération des fichiers.

## <a name="troubleshooting-common-issues"></a>Dépannage des problèmes courants

Cette section décrit les étapes à suivre pour résoudre les problèmes rencontrés pendant le téléchargement et l’exécution du script pour la récupération de fichiers.

## <a name="cannot-download-the-script"></a>Impossible de télécharger le script

Action recommandée :

1. Vérifiez que vous disposez de toutes les [autorisations nécessaires pour télécharger le script](https://docs.microsoft.com/azure/backup/backup-azure-restore-files-from-vm#select-recovery-point-who-can-generate-script).
2. Vérifiez qu’il y a une connectivité aux adresses IP cibles Azure.
Pour vérifier la connexion, exécutez la commande suivante à partir d’une invite de commandes avec élévation de privilèges. 
    - *nslookup download.microsoft.com* ou
    - *ping download.microsoft.com*

## <a name="the-script-downloads-successfully-but-fails-to-run"></a>Le script se télécharge correctement, mais il ne s’exécute pas

### <a name="for-sles-12-sp4-os-linux"></a>Pour le système d’exploitation SLES 12 SP4 (Linux)

Erreur : iscsi_tcp_module introuvable

Cause possible : Pendant l’exécution du script Python pour la récupération au niveau de l’élément sur le système d’exploitation SUSE Linux version 12sp4, une erreur se produit avec l’erreur ***iscsi_tcp module can’t be loaded** _ (impossible de charger le module iscsi_tcp). Le module ILR utilise « iscsi_tcp » pour établir une connexion TCP avec le service de sauvegarde. Or, dans la version 12SP4 de SUSE, iscsi_tcp a été retiré du package open-iscsi, ce qui explique l’échec de l’opération ILR.

Action recommandée :  L’exécution du script de récupération de fichiers n’est pas prise en charge sur les machines virtuelles SUSE 12SP4. Tentez l’opération de restauration sur une version plus ancienne de SUSE 12SP4.

## <a name="the-script-runs-but-connection-to-iscsi-target-failed"></a>Le script s’exécute, mais la connexion à la cible iSCSI échoue

Erreur : Exception interceptée pendant la connexion à la cible

1. Vérifiez que la machine sur laquelle le script est exécuté remplit toutes les [conditions d’accès](https://docs.microsoft.com/azure/backup/backup-azure-restore-files-from-vm#step-4-access-requirements-to-successfully-run-the-script).
2. Vérifiez qu’il y a une connectivité aux adresses IP cibles Azure.
Pour vérifier la connexion, exécutez la commande suivante à partir d’une invite de commandes avec élévation de privilèges. 
    - _nslookup download.microsoft.com* ou<br>
    - *ping download.microsoft.com*
3. Vérifiez qu’il y a un accès au port de sortie iSCSI 3260.
4. Vérifiez qu’aucun pare-feu/groupe de sécurité réseau (NSG) ne bloque le trafic vers les adresses IP cibles Azure ou les URL du service de récupération.
5. Vérifiez si un antivirus bloque l’exécution du script.

## <a name="connected-to-recovery-point-but-disks-did-not-get-attached"></a>Connecté à un point de récupération, mais disques non attachés

Vérifiez que votre [machine permet d’exécuter le script](https://docs.microsoft.com/azure/backup/backup-azure-restore-files-from-vm#step-2-ensure-the-machine-meets-the-requirements-before-executing-the-script)

### <a name="on-windows-vm"></a>Sur une machine virtuelle Windows

**Le pool de stockage sur la machine virtuelle est attaché en mode lecture seule** Pour Windows 2012 R2 et Windows 2016 (avec le pool de stockage), lors de la première exécution du script, il se peut que le pool de stockage attaché à la machine virtuelle passe à l’état lecture seule.

Pour résoudre ce problème, il convient de définir manuellement l’accès en lecture-écriture au pool de stockage pour la première fois et attacher les disques virtuels en suivant les étapes ci-dessous :

1. Définissez l’accès en lecture-écriture.
Accédez à Gestionnaire de serveur > Services de fichiers et de stockage > Volumes > Pools de stockage.

   ![Stockage Windows](./media/backup-azure-restore-files-from-vm/windows-storage-1.png)

2. Dans la fenêtre **Pool de stockage**, cliquez avec le bouton droit sur le pool de stockage disponible, puis sélectionnez l’option **Définir l’accès en lecture-écriture**.

   ![Lecture-écriture du stockage Windows](./media/backup-azure-restore-files-from-vm/windows-storage-read-write-2.png)

3. Une fois le pool de stockage affecté avec un accès en lecture/écriture, nous devons attacher le disque virtuel.
Accédez à l’interface utilisateur du **Gestionnaire de serveur** puis, sous la section Disques virtuels, cliquez avec le bouton droit sur l’option **Attacher le disque virtuel** pour la sélectionner.

   ![Gestionnaire de serveur Disque virtuel](./media/backup-azure-restore-files-from-vm/server-manager-virtual-disk-3.png)

### <a name="on-linux-vm"></a>Sur une machine virtuelle Linux

#### <a name="file-recovery-fails-to-auto-mount-because-disk-does-not-contain-volumes"></a>La récupération de fichiers échoue dans l’opération de montage automatique, car le disque ne contient pas de volumes

Pendant la récupération de fichiers, le service de sauvegarde détecte les volumes et procède au montage automatique. Or, si les disques sauvegardés ont des partitions brutes, ces disques ne sont pas montés automatiquement et le disque de données à récupérer n’est pas visible.

Pour résoudre ce problème, effectuez les étapes décrites dans cet [article](https://docs.microsoft.com/azure/backup/backup-azure-restore-files-from-vm#lvmraid-arrays-for-linux-vms).
 
#### <a name="os-could-not-identify-the-filesystem-causing-linux-file-recovery-to-fail-while-mountings-disks"></a>Le système d’exploitation n’a pas pu identifier le système de fichiers, ce qui a fait échouer la récupération de fichiers Linux lors de la phase de montage des disques

Pendant l’exécution du script de récupération de fichiers, l’attachement du disque de données a échoué avec l’erreur ci-dessous : *Les partitions suivantes n’ont pas pu être montées, car le système d’exploitation n’a pas pu identifier le système de fichiers*

- Pour résoudre ce problème, vérifiez si le volume est chiffré avec une application tierce. Si c’est le cas, le disque ou la machine virtuelle n’apparaît pas comme étant chiffré sur le portail.
1. Connectez-vous à la machine virtuelle sauvegardée et exécutez la commande *lsblk -f*<br>

   ![Disque sans volume](./media/backup-azure-restore-files-from-vm/disk-without-volume-5.png)

2. Vérifiez le système de fichiers et le chiffrement.
3. Si le volume est chiffré, la récupération de fichiers n’est pas prise en charge. [Plus d’informations](https://docs.microsoft.com/azure/backup/backup-support-matrix-iaas#support-for-file-level-restore)

## <a name="disks-are-attached-but-unable-to-mount-volumes"></a>Les disques sont attachés, mais les volumes ne peuvent pas être montés

- Vérifiez que votre [machine permet d’exécuter le script](https://docs.microsoft.com/azure/backup/backup-azure-restore-files-from-vm#step-2-ensure-the-machine-meets-the-requirements-before-executing-the-script).

### <a name="on-windows-vm"></a>Sur une machine virtuelle Windows

Pendant l’exécution du script de récupération de fichiers pour Windows, un message indique que ***0 volumes sont attachés** _, alors que les disques sont détectés dans la console Gestion des disques. Pendant l’attachement de volumes via iSCSI, certains volumes détectés passent à l’état hors connexion. Quand le canal iSCSI communique entre la machine virtuelle et le service, il détecte ces volumes et les met en ligne, mais ils ne sont pas montés.

   ![Disque non attaché](./media/backup-azure-restore-files-from-vm/disk-not-attached-6.png)

Pour identifier et résoudre ce problème, effectuez les étapes suivantes :

1. Accédez à _ *Gestion des disques** en exécutant la commande **diskmgmt** dans la fenêtre cmd.
2. Vérifiez si des disques supplémentaires sont affichés. C’est le cas dans l’exemple ci-dessous avec Disk 2.

   ![Gestion des disques0](./media/backup-azure-restore-files-from-vm/disk-management-7.png)

3. Cliquez avec le bouton droit sur **Volume**, puis sélectionnez **Modifier la lettre de lecteur et les chemins d’accès**.

   ![Gestion des disques1](./media/backup-azure-restore-files-from-vm/disk-management-8.png)

4. Dans la fenêtre **Ajouter une lettre de lecteur ou un chemin d’accès**, sélectionnez **Attribuer la lettre de lecteur suivante**, attribuez un lecteur disponible, puis cliquez sur **OK**. 

   ![Gestion des disques2](./media/backup-azure-restore-files-from-vm/disk-management-9.png)

5. Dans l’Explorateur de fichiers, vous pouvez voir le lecteur.

   ![Gestion des disques3](./media/backup-azure-restore-files-from-vm/disk-management-10.png)

6. Les nouveaux volumes doivent être attachés.  

   ![Échec du montage du disque](./media/backup-azure-restore-files-from-vm/disk-not-mounting-11.png)

7. Dans l’Explorateur de fichiers, les nouveaux volumes seront visibles après la réexécution du script.

   ![Échec du montage du disque1](./media/backup-azure-restore-files-from-vm/disk-not-mounting-12.png)

   ![Échec du montage du disque2](./media/backup-azure-restore-files-from-vm/disk-not-mounting-13.png)

#### <a name="on-linux-vms"></a>Sur les machines virtuelles Linux 

- Vérifiez que votre [machine permet d’exécuter le script](https://docs.microsoft.com/azure/backup/backup-azure-restore-files-from-vm#step-2-ensure-the-machine-meets-the-requirements-before-executing-the-script).
- Si la machine virtuelle Linux protégée utilise des baies LVM et/ou RAID, suivez les étapes listées dans cet [article](https://docs.microsoft.com/azure/backup/backup-azure-restore-files-from-vm#lvmraid-arrays-for-linux-vms).

## <a name="cannot-copy-the-files-from-mounted-volumes"></a>Impossible de copier les fichiers à partir des volumes montés

La copie peut échouer avec une erreur 0x80070780 : Le système ne peut pas accéder au fichier. Dans ce cas, vérifiez si la déduplication de disque est activée sur le serveur source. Vérifiez ensuite que la déduplication est également activée sur les lecteurs du serveur de restauration. Vous pouvez laisser le rôle de déduplication non configuré pour éviter de dédupliquer les lecteurs sur le serveur de restauration.
