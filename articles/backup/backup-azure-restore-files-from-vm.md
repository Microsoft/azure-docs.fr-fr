---
title: Récupérer des fichiers et des dossiers à partir d’une sauvegarde de machine virtuelle Azure
description: Dans cet article, découvrez comment récupérer des fichiers et des dossiers à partir d’un point de récupération de machine virtuelle Azure.
ms.topic: conceptual
ms.date: 03/12/2020
ms.custom: references_regions
ms.openlocfilehash: 4d34fc48e5d16275d0225a1cef4b5fa63f0b66d6
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97511652"
---
# <a name="recover-files-from-azure-virtual-machine-backup"></a>Récupérer des fichiers à partir d’une sauvegarde de machine virtuelle Azure

La sauvegarde Azure offre la possibilité de restaurer des [machines virtuelles et des disques Azure](./backup-azure-arm-restore-vms.md) à partir de sauvegardes de machines virtuelles Azure, également appelées points de récupération. Cet article explique comment récupérer des fichiers et des dossiers à partir d’une sauvegarde de machine virtuelle Azure. La restauration des fichiers et des dossiers est disponible uniquement pour les machines virtuelles Azure déployées à l’aide du modèle Resource Manager et protégées dans un coffre Recovery Services.


> [!NOTE]
> Cette fonctionnalité est disponible pour les machines virtuelles Azure déployées suivant le modèle Resource Manager et protégées dans un coffre Recovery Services.
> La récupération de fichiers à partir d’une sauvegarde de machine virtuelle chiffrée n’est pas prise en charge.
>

![Workflow de récupération du dossier de fichiers](./media/backup-azure-restore-files-from-vm/file-recovery-1.png)

## <a name="step-1-generate-and-download-script-to-browse-and-recover-files"></a>Étape 1 : Générer et télécharger un script pour parcourir et récupérer des fichiers

Pour restaurer des fichiers ou des dossiers à partir du point de récupération, accédez à la machine virtuelle et procédez comme suit :

1. Connectez-vous au [Portail Azure](https://portal.Azure.com), puis, dans le volet à gauche, sélectionnez **Machines virtuelles**. Dans la liste des machines virtuelles, sélectionnez la machine virtuelle pour ouvrir son tableau de bord.

2. Dans le menu de la machine virtuelle, cliquez sur **Sauvegarde** pour ouvrir le tableau de bord Sauvegarde.

    ![Ouvrir l’élément de sauvegarde du coffre Recovery Services.](./media/backup-azure-restore-files-from-vm/open-vault-for-vm.png)

3. Dans le menu du tableau de bord Sauvegarde, sélectionnez **Récupération de fichiers**.

    ![Sélectionner la récupération de fichier](./media/backup-azure-restore-files-from-vm/vm-backup-menu-file-recovery-button.png)

    Le menu **Récupération de fichiers** s’ouvre.

    ![Menu de récupération de fichiers](./media/backup-azure-restore-files-from-vm/file-recovery-blade.png)

4. Dans le menu déroulant **Sélectionner le point de récupération**, sélectionnez le point de récupération qui contient les fichiers dont vous avez besoin. Le dernier point de récupération est sélectionné par défaut.

5. Pour télécharger le logiciel utilisé pour copier des fichiers à partir du point de récupération, sélectionnez **Télécharger l’exécutable** (pour les machines virtuelles Windows Azure) ou **Télécharger le script** (pour les machines virtuelles Linux Azure, un script Python est généré).

    ![Télécharger l’exécutable](./media/backup-azure-restore-files-from-vm/download-executable.png)

    Azure télécharge le fichier exécutable ou le script sur l’ordinateur local.

    ![Message de téléchargement du fichier exécutable ou du script](./media/backup-azure-restore-files-from-vm/run-the-script.png)

    Pour exécuter le fichier exécutable ou le script en tant qu’administrateur, il est recommandé d’enregistrer le fichier téléchargé sur votre ordinateur.

6. Le fichier exécutable ou le script est protégé par mot de passe et nécessite un mot de passe. Dans le menu **Récupération de fichiers**, sélectionnez le bouton Copier pour charger le mot de passe en mémoire.

    ![Mot de passe généré](./media/backup-azure-restore-files-from-vm/generated-pswd.png)


## <a name="step-2-ensure-the-machine-meets-the-requirements-before-executing-the-script"></a>Étape 2 : Vérifier que la machine respecte la configuration requise avant d’exécuter le script

Une fois que le script a été téléchargé, vérifiez que vous disposez de la machine appropriée pour exécuter ce script. La machine virtuelle sur laquelle vous envisagez d’exécuter le script ne doit pas avoir l’une des configurations non prises en charge suivantes. Si c’est le cas, choisissez une autre machine, de préférence dans la même région, qui respecte la configuration requise.  

### <a name="dynamic-disks"></a>Disques dynamiques

Vous ne pouvez pas exécuter le script exécutable sur la machine virtuelle avec l’une des caractéristiques suivantes :

- Volumes qui s’étendent sur plusieurs disques (volumes fractionnés et agrégés par bandes).
- Volumes à tolérance de pannes (volumes RAID-5 ou en miroir) sur des disques dynamiques.

### <a name="windows-storage-spaces"></a>Espaces de stockage Windows

Vous ne pouvez pas exécuter l’exécutable téléchargé sur une machine virtuelle configurée pour des espaces de stockage Windows.

### <a name="virtual-machine-backups-having-large-disks"></a>Sauvegardes de machines virtuelles avec des disques de grande taille

Si la machine sauvegardée possède un grand nombre de disques (>16) ou des disques de grande taille (>4 To chacun), il n’est pas recommandé d’exécuter le script sur la même machine pour la restauration, car cela aura un impact significatif sur la machine virtuelle. Au lieu de cela, il est recommandé d’avoir une machine virtuelle distincte uniquement pour la récupération de fichiers (machines virtuelles Azure VM D2v3), puis de l’arrêter lorsqu’elle n’est pas nécessaire. 

## <a name="step-3-os-requirements-to-successfully-run-the-script"></a>Étape 3 : Configuration requise du système d’exploitation pour exécuter correctement le script

La machine virtuelle sur laquelle vous souhaitez exécuter le script téléchargé doit remplir les conditions suivantes.

### <a name="for-windows-os"></a>Pour système d’exploitation Windows

Le tableau suivant indique la compatibilité entre les systèmes d’exploitation des serveurs et des ordinateurs. Lors de la récupération de fichiers, vous ne pouvez pas restaurer des fichiers sur une version antérieure ou ultérieure du système d’exploitation. Par exemple, vous ne pouvez pas restaurer un fichier à partir d’une machine virtuelle sous Windows Server 2016 sur un ordinateur sous Windows Server 2012 ou Windows 8. Vous pouvez restaurer des fichiers à partir d’une machine virtuelle sur un système d’exploitation de serveur identique, ou sur un système d’exploitation client compatible.

|Système d’exploitation serveur | Système d’exploitation client compatible  |
| --------------- | ---- |
| Windows Server 2019    | Windows 10 |
| Windows Server 2016    | Windows 10 |
| Windows Server 2012 R2 | Windows 8.1 |
| Windows Server 2012    | Windows 8  |
| Windows Server 2008 R2 | Windows 7   |

### <a name="for-linux-os"></a>Pour le système d’exploitation Linux

Sur Linux, le système d’exploitation de l’ordinateur utilisé pour restaurer des fichiers doit prendre en charge le système de fichiers de la machine virtuelle protégée. Lorsque vous sélectionnez un ordinateur pour exécuter le script, vérifiez que l’ordinateur possède un système d’exploitation compatible et utilise l’une des versions identifiées dans le tableau suivant :

|Système d’exploitation Linux | Versions  |
| --------------- | ---- |
| Ubuntu | 12.04 et versions ultérieures |
| CentOS | 6.5 et versions ultérieures  |
| RHEL | 6.7 et versions ultérieures |
| Debian | 7 et versions ultérieures |
| Oracle Linux | 6.4 et versions ultérieures |
| SLES | 12 et versions ultérieures |
| OpenSUSE | 42.2 et versions ultérieures |

> [!NOTE]
> Nous avons rencontré des problèmes lors de l’exécution du script de récupération de fichiers sur les ordinateurs équipés du système d’exploitation SLE 12 SP4 et nous étudions la question avec l’équipe SLES.
> Actuellement, l’exécution du script de récupération de fichier fonctionne sur des ordinateurs équipés des versions de système d’exploitation SLES 12 SP2 et SP3.
>

Le script requiert également les composants Python et bash pour exécuter et établir une connexion sécurisée avec le point de récupération.

|Composant | Version  |
| --------------- | ---- |
| bash | 4 et versions ultérieures |
| python | 2.6.6 et versions ultérieures  |
| TLS | La version 1.2 devrait être prise en charge  |

## <a name="step-4-access-requirements-to-successfully-run-the-script"></a>Étape 4 : Conditions d’accès pour exécuter correctement le script

Si vous exécutez le script sur un ordinateur disposant d’un accès restreint, assurez-vous qu’il a accès aux éléments suivants :

- `download.microsoft.com`
- URL Recovery Services (GEO-NAME fait référence à la région où réside le coffre Recovery Services.)
  - `https://pod01-rec2.GEO-NAME.backup.windowsazure.com` (Pour les régions publiques Azure)
  - `https://pod01-rec2.GEO-NAME.backup.windowsazure.cn` (Pour Azure Chine 21Vianet)
  - `https://pod01-rec2.GEO-NAME.backup.windowsazure.us` (Pour Azure US Government)
  - `https://pod01-rec2.GEO-NAME.backup.windowsazure.de` (Pour Azure Allemagne)
- Ports sortants 53 (DNS), 443, 3260

> [!NOTE]
>
> Le fichier de script que vous avez téléchargé à l’étape 1 [ci-dessus](#step-1-generate-and-download-script-to-browse-and-recover-files) aura l’élément **geo-name** dans le nom du fichier. Utilisez cet élément **geo-name** pour compléter l’URL. Le nom du script téléchargé commence par : \'VMname\'\_\'geoname\'_\'GUID\'.<br><br>
> Ainsi, par exemple, si le nom de fichier du script est *ContosoVM_wcus_12345678*, l’élément **geo-name** est *wcus* et l’URL devient :<br> <https://pod01-rec2.wcus.backup.windowsazure.com>
>

Pour Linux, le script requiert les composants « open-iscsi » et « lshw » pour vous connecter au point de récupération. Si les composants n’existent pas sur l’ordinateur depuis lequel le script est exécuté, le script demande l’autorisation d’installer les composants. Autorisez l’installation des composants nécessaires.

L’accès à `download.microsoft.com` est requis pour télécharger les composants utilisés pour créer un canal sécurisé entre l’ordinateur sur lequel le script est exécuté et les données du point de récupération.


## <a name="step-5-running-the-script-and-identifying-volumes"></a>Étape 5 : Exécution du script et identification des volumes

### <a name="for-windows"></a>Pour Windows

Une fois que vous avez rempli toutes les conditions requises indiquées à l’étape 2, à l’étape 3 et à l’étape 4, copiez le script à partir de l’emplacement téléchargé (généralement le dossier Téléchargements), cliquez avec le bouton droit sur le fichier exécutable ou le script et exécutez-le avec les informations d’identification de l’administrateur. Lorsque vous y êtes invité, tapez le mot de passe ou collez le mot de passe en mémoire et appuyez sur Entrée. Une fois le mot de passe valide entré, le script se connecte au point de récupération.

  ![Sortie de l’exécutable](./media/backup-azure-restore-files-from-vm/executable-output.png)


Lorsque vous exécutez l’exécutable, le système d’exploitation monte les nouveaux volumes et attribue des lettres de lecteur. Vous pouvez utiliser l’Explorateur Windows ou l’Explorateur de fichiers pour parcourir ces lecteurs. Les lettres de lecteur affectées aux volumes peuvent ne pas être les mêmes que celles de la machine virtuelle d’origine. Toutefois, le nom du volume est conservé. Par exemple, si le volume de la machine virtuelle d’origine était « Disque de données (E:`\`) », ce volume peut être connecté sur l’ordinateur local en tant que « Disque de données (n’importe quelle lettre:`\`) ». Parcourez tous les volumes mentionnés dans la sortie du script jusqu’à trouver vos fichiers ou votre dossier.  

   ![Volumes de récupération attachés](./media/backup-azure-restore-files-from-vm/volumes-attached.png)

#### <a name="for-backed-up-vms-with-large-disks-windows"></a>Pour les machines virtuelles sauvegardées avec des disques de grande taille (Windows)

Si le processus de récupération des fichiers est interrompu après l’exécution du script de restauration des fichiers (par exemple, si les disques ne sont jamais montés ou s’ils le sont, mais que les volumes n’apparaissent pas), procédez comme suit :
  
1. Assurez-vous que le système d’exploitation est WS 2012 ou ultérieur.
2. Vérifiez que les clés de Registre sont définies comme indiqué ci-dessous dans le serveur de restauration et assurez-vous de redémarrer le serveur. Le nombre en regard du GUID peut être compris entre 0001 et 0005. Dans l’exemple suivant, il s’agit de 0004. Parcourez le chemin d’accès de la clé de Registre jusqu’à la section des paramètres.

    ![Modifications apportées aux clés de registre](media/backup-azure-restore-files-from-vm/iscsi-reg-key-changes.png)

```registry
- HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Disk\TimeOutValue – change this from 60 to 1200
- HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Control\Class\{4d36e97b-e325-11ce-bfc1-08002be10318}\0003\Parameters\SrbTimeoutDelta – change this from 15 to 1200
- HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Control\Class\{4d36e97b-e325-11ce-bfc1-08002be10318}\0003\Parameters\EnableNOPOut – change this from 0 to 1
- HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Control\Class\{4d36e97b-e325-11ce-bfc1-08002be10318}\0003\Parameters\MaxRequestHoldTime - change this from 60 to 1200
```

### <a name="for-linux"></a>Pour Linux

Pour les machines Linux, un script Python est généré. Téléchargez le script et copiez-le sur le serveur Linux approprié/compatible. Il peut être nécessaire de modifier les autorisations pour l’exécuter avec ```chmod +x <python file name>```. Exécutez ensuite le fichier Python avec ```./<python file name>```.


Sous Linux, les volumes de point de récupération sont montés sur le dossier dans lequel le script est exécuté. Les disques attachés, les volumes et les chemins d’accès de montage correspondants sont affichés en conséquence. Ces chemins de montage sont visibles aux utilisateurs ayant un accès root. Parcourez les volumes mentionnés dans la sortie du script.

  ![Menu Récupération de fichier Linux](./media/backup-azure-restore-files-from-vm/linux-mount-paths.png)


#### <a name="for-backed-up-vms-with-large-disks-linux"></a>Pour les machines virtuelles sauvegardées avec des disques de grande taille (Linux)**

Si le processus de récupération des fichiers est interrompu après l’exécution du script de restauration des fichiers (par exemple, si les disques ne sont jamais montés ou s’ils le sont, mais que les volumes n’apparaissent pas), procédez comme suit :

1. Dans le fichier /etc/iSCSI/iSCSId.conf, modifiez le paramètre à partir de :
    - `node.conn[0].timeo.noop_out_timeout = 5` à `node.conn[0].timeo.noop_out_timeout = 30`
2. Après avoir apporté les modifications ci-dessus, réexécutez le script. En cas de défaillances temporaires, assurez-vous qu’il y ait un intervalle de 20 à 30 minutes entre les réexécutions afin d’éviter que des rafales successives de requêtes n’aient un impact sur la préparation cible. Cet intervalle entre les réexécutions permet de s’assurer que la cible est prête à être connectée à partir du script.
3. Après la récupération de fichier, assurez-vous de revenir au portail et sélectionnez **Démonter les disques** pour les points de récupération où vous n’avez pas pu monter des volumes. Pour l’essentiel, cette étape nettoie les processus et les sessions existantes et augmente les chances de récupération.


#### <a name="lvmraid-arrays-for-linux-vms"></a>LVM/regroupements RAID (pour machines virtuelles Linux)

Sous Linux, le gestionnaire de volumes logiques (LVM) et/ou les regroupements RAID logiciels permettent de gérer des volumes logiques sur plusieurs disques. Si la machine virtuelle Linux protégée utilise LVM et/ou des baies RAID, vous ne pouvez pas exécuter le script sur la même machine virtuelle.<br>
Exécutez plutôt le script sur une autre machine avec un système d’exploitation compatible et qui prend en charge le système de fichiers de la machine virtuelle protégée.<br>
La sortie du script suivante affiche les disques LVM et/ou des baies RAID et les volumes avec le type de partition.

   ![Menu de sortie de LVM Linux](./media/backup-azure-restore-files-from-vm/linux-LVMOutput.png)

Pour mettre ces partitions en ligne, exécutez les commandes dans les sections suivantes.

#### <a name="for-lvm-partitions"></a>Pour les partitions LVM

Une fois le script exécuté, les partitions LVM sont montées dans le(s)volume(s)/disques(s) physiques(s) spécifié(s) dans le résultat du script. Le processus est le suivant :

1. obtenir la liste unique des noms de groupes de volumes à partir des volumes ou disques physiques ;
2. répertorier les volumes logiques dans ces groupes de volumes ;
3. monter les volumes logiques dans un chemin d’accès souhaité.

##### <a name="listing-volume-group-names-from-physical-volumes"></a>Référencement des noms de groupes de volumes à partir des volumes physiques

Listage des noms de groupes de volumes :

```bash
pvs -o +vguuid
```

Cette commande répertorie tous les volumes physiques (y compris ceux présents avant l’exécution du script), leurs noms de groupes de volumes correspondants et les ID d’utilisateurs uniques (UUID) de groupes de volumes. Vous trouverez ci-dessous un exemple de sortie de la commande.

```bash
PV         VG        Fmt  Attr PSize   PFree    VG UUID

  /dev/sda4  rootvg    lvm2 a--  138.71g  113.71g EtBn0y-RlXA-pK8g-de2S-mq9K-9syx-B29OL6

  /dev/sdc   APPvg_new lvm2 a--  <75.00g   <7.50g njdUWm-6ytR-8oAm-8eN1-jiss-eQ3p-HRIhq5

  /dev/sde   APPvg_new lvm2 a--  <75.00g   <7.50g njdUWm-6ytR-8oAm-8eN1-jiss-eQ3p-HRIhq5

  /dev/sdf   datavg_db lvm2 a--   <1.50t <396.50g dhWL1i-lcZS-KPLI-o7qP-AN2n-y2f8-A1fWqN

  /dev/sdd   datavg_db lvm2 a--   <1.50t <396.50g dhWL1i-lcZS-KPLI-o7qP-AN2n-y2f8-A1fWqN
```

La première colonne (PV) affiche le volume physique, les colonnes suivantes affichent le nom, le format, les attributs, la taille, l’espace libre et l’ID unique du groupe de volumes. La sortie de la commande affiche tous les volumes physiques. Reportez-vous à la sortie du script et identifiez les volumes associés à la sauvegarde. Dans l’exemple ci-dessus, la sortie du script aurait montré /dev/sdf et/dev/sdd. Ainsi, le groupe de volumes *datavg_db* appartient au script et le groupe de volumes *Appvg_new* appartient à l’ordinateur. L’idée finale est de s’assurer qu’un nom de groupe de volumes unique a un ID unique.

###### <a name="duplicate-volume-groups"></a>Groupes de volumes en double

Il existe des scénarios dans lesquels les noms de groupes de volumes peuvent avoir 2 UUID après l’exécution du script. Cela signifie que les noms de groupes de volumes dans l’ordinateur sur lequel le script est exécuté et dans la machine virtuelle sauvegardée sont identiques. Ensuite, nous devons renommer les groupes de volumes de machines virtuelles sauvegardés. Examinez l’exemple suivant.

```bash
PV         VG        Fmt  Attr PSize   PFree    VG UUID

  /dev/sda4  rootvg    lvm2 a--  138.71g  113.71g EtBn0y-RlXA-pK8g-de2S-mq9K-9syx-B29OL6

  /dev/sdc   APPvg_new lvm2 a--  <75.00g   <7.50g njdUWm-6ytR-8oAm-8eN1-jiss-eQ3p-HRIhq5

  /dev/sde   APPvg_new lvm2 a--  <75.00g   <7.50g njdUWm-6ytR-8oAm-8eN1-jiss-eQ3p-HRIhq5

  /dev/sdg   APPvg_new lvm2 a--  <75.00g  508.00m lCAisz-wTeJ-eqdj-S4HY-108f-b8Xh-607IuC

  /dev/sdh   APPvg_new lvm2 a--  <75.00g  508.00m lCAisz-wTeJ-eqdj-S4HY-108f-b8Xh-607IuC

  /dev/sdm2  rootvg    lvm2 a--  194.57g  127.57g efohjX-KUGB-ETaH-4JKB-MieG-EGOc-XcfLCt
```

La sortie du script aurait montré /dev/sdg, /dev/sdh et /dev/sdm2 attachés. Par conséquent, les noms de groupes de volumes correspondants sont Appvg_new et rootvg. Toutefois, les mêmes noms sont également présents dans la liste de groupes de volumes de la machine. Nous pouvons vérifier que le nom d’un VG a deux UUID.

Nous devons maintenant renommer les noms de groupes de volumes pour les volumes basés sur un script, par exemple : /dev/sdg, /dev/sdh, /dev/sdm2. Pour renommer le groupe de volumes, utilisez la commande suivante :

```bash
vgimportclone -n rootvg_new /dev/sdm2
vgimportclone -n APPVg_2 /dev/sdg /dev/sdh
```

Nous avons maintenant tous les noms de groupes de volumes avec des ID uniques.

###### <a name="active-volume-groups"></a>Groupes de volumes actifs

Assurez-vous que les groupes de volumes correspondant aux volumes du script sont actifs. La commande suivante permet d’afficher des groupes de volumes actifs. Vérifiez si les groupes de volumes associés au script sont présents dans cette liste.

```bash
vgdisplay -a
```  

Autrement, activez le groupe de volumes à l’aide de la commande suivante.

```bash
#!/bin/bash
vgchange –a y  <volume-group-name>
```

##### <a name="listing-logical-volumes-within-volume-groups"></a>Listage des volumes logiques dans les groupes de volumes

Une fois que nous obtenons la liste active unique des groupes de volumes associés au script, les volumes logiques présents dans ceux-ci peuvent être répertoriés à l’aide de la commande suivante.

```bash
#!/bin/bash
lvdisplay <volume-group-name>
```

Cette commande affiche le chemin d’accès de chaque volume logique sous la forme « VL Chemin d’accès ».

##### <a name="mounting-logical-volumes"></a>Montage de volumes logiques

Pour monter les volumes logiques sur le chemin d’accès de votre choix :

```bash
#!/bin/bash
mount <LV path from the lvdisplay cmd results> </mountpath>
```

> [!WARNING]
> N’utilisez pas la commande « mount -a ». Cette commande monte tous les appareils décrits dans « /etc/fstab ». Cela peut signifier que des périphériques en double peuvent être montés. Les données peuvent être redirigées vers des appareils créés par script, qui ne conservent pas les données et, par conséquent, peuvent entraîner une perte de données.

#### <a name="for-raid-arrays"></a>Pour les tableaux RAID

La commande suivante affiche des détails sur tous les disques RAID :

```bash
#!/bin/bash
mdadm –detail –scan
```

 Le disque RAID pertinent s’affiche en tant que `/dev/mdm/<RAID array name in the protected VM>`

Utilisez la commande de montage si le disque RAID dispose de volumes physiques :

```bash
#!/bin/bash
mount [RAID Disk Path] [/mountpath]
```

Si le disque RAID contient un autre gestionnaire de volume logique configuré, utilisez la procédure précédente pour les partition LVM, en utilisant le nom de volume à la place du nom de disque RAID.

## <a name="step-6-closing-the-connection"></a>Étape 6 : Fermeture de la connexion

Après avoir identifié les fichiers et les avoir copiés dans un emplacement de stockage local, supprimez (ou démontez) les lecteurs supplémentaires. Pour démonter les lecteurs, dans le menu **Récupération de fichier** du portail Azure, cliquez sur **Démonter les disques**.

![Démonter les disques](./media/backup-azure-restore-files-from-vm/unmount-disks3.png)

Une fois les disques démontés, vous recevez un message. L’actualisation de la connexion de sorte que vous pouvez supprimer les disques peut prendre quelques minutes.

Sous Linux, une fois la connexion au point de récupération interrompue, le système d’exploitation ne supprime pas les chemins d’accès de montage correspondants automatiquement. Les chemins d’accès de montage existent en tant que volumes « orphelins » et ils sont visibles, mais génèrent une erreur lorsque vous accédez aux fichiers ou écrivez dessus. Ils peuvent être supprimés manuellement. Le script, lorsqu’il est exécuté, identifie de tels volumes existants à partir des points de récupération précédents et les nettoie après accord.

> [!NOTE]
> Assurez-vous que la connexion est fermée après la restauration des fichiers requis. Cela est important, en particulier dans le cas où l’ordinateur sur lequel le script est exécuté est également configuré pour la sauvegarde. Si la connexion est toujours ouverte, la sauvegarde suivante risque d’échouer avec l’erreur « UserErrorUnableToOpenMount ». Cela se produit parce que les lecteurs/volumes montés sont supposés être disponibles et lorsqu’on y accède, ils peuvent échouer parce que le stockage sous-jacent, c’est-à-dire le serveur cible iSCSI, peut ne pas être disponible. Le nettoyage de la connexion entraînera la suppression de ces lecteurs/volumes et ils ne seront donc pas disponibles pendant la sauvegarde.

## <a name="security"></a>Sécurité

Cette section décrit les diverses mesures de sécurité prises pour l’implémentation de la récupération de fichiers à partir de sauvegardes de machines virtuelles Azure.

### <a name="feature-flow"></a>Flux de fonctionnalité

Cette fonctionnalité a été conçue pour accéder aux données de la machine virtuelle sans avoir à en restaurer l’ensemble ou les disques, avec le nombre réduit d’étapes. L’accès aux données de machine virtuelle est fourni par un script (qui monte le volume de récupération lorsqu’il est exécuté comme indiqué ci-dessous) et constitue la pierre angulaire de toutes les implémentations de sécurité :

  ![Flux de fonctionnalité de sécurité](./media/backup-azure-restore-files-from-vm/vm-security-feature-flow.png)

### <a name="security-implementations"></a>Implémentations de la sécurité

#### <a name="select-recovery-point-who-can-generate-script"></a>Sélectionner le point de récupération (quel utilisateur peut générer un script)

Le script donne accès aux données de la machine virtuelle. Il est donc important de réglementer qui peut les générer en premier lieu. Vous devez vous connecter au portail Azure et disposer de l’[autorisation Azure RBAC](backup-rbac-rs-vault.md#mapping-backup-built-in-roles-to-backup-management-actions) pour générer le script.

La récupération de fichier doit avoir le même niveau d’autorisation que pour la restauration des disques et des machines virtuelles. En d’autres termes, seuls les utilisateurs autorisés peuvent visualiser les données de la machine virtuelle et générer le script.

Le script généré est signé avec le certificat officiel de Microsoft pour le service Sauvegarde Azure. Toute modification du script signifie que la signature est violée, et toute tentative d’exécution du script est mise en évidence par le système d’exploitation comme un risque potentiel.

#### <a name="mount-recovery-volume-who-can-run-script"></a>Volume de récupération de montage (quel utilisateur peut exécuter le script)

Seul un administrateur peut exécuter le script et il doit le faire en mode élevé. Le script exécute uniquement un ensemble d’étapes pré-générée et n’accepte pas d’entrée provenant de sources externes.

Pour exécuter le script, il est nécessaire de disposer d’un mot de passe qui ne sera montré qu’à l’utilisateur autorisé au moment de la génération du script dans le portail Azure ou PowerShell/CLI. Cela permet de s’assurer que l’utilisateur autorisé qui télécharge le script est également responsable de l’exécution du script.

#### <a name="browse-files-and-folders"></a>Parcourir des fichiers et dossiers

Pour parcourir des fichiers et dossiers, le script utilise l’initiateur iSCSI de l’ordinateur et se connecte au point de récupération qui est configuré comme cible iSCSI. Ici, vous pouvez imaginer des scénarios où l’on veut imiter ou usurper l’identité d’un ou de tous les composants.

Nous utilisons un mécanisme d’authentification CHAP mutuel afin que les composants s’authentifient les uns les autres. Cela signifie qu’il est extrêmement difficile pour un initiateur factice de se connecter à la cible iSCSI, et pour une fausse cible de se connecter à l’ordinateur où le script est exécuté.

Le flux de données entre le service de récupération et la machine est protégé par la création d’un tunnel TLS sécurisé sur TCP ([TLS 1.2 doit être pris en charge](#step-3-os-requirements-to-successfully-run-the-script) sur la machine où le script est exécuté).

Tout fichier de liste de contrôle d’accès (ACL) présent dans la machine virtuelle parente ou sauvegardée est également conservé dans le système de fichiers monté.

Le script donne accès en lecture seule à un point de récupération et n’est valide que pendant 12 heures. Si vous souhaitez supprimer l’accès plus tôt, connectez-vous au portail Azure/PowerShell/CLI et **démontez des disques** pour ce point de récupération spécifique. Le script est invalidé immédiatement.


## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [restaurer des fichiers via PowerShell](./backup-azure-vms-automation.md#restore-files-from-an-azure-vm-backup)
- Découvrez comment [restaurer des fichiers via Azure CLI](./tutorial-restore-files.md)
- Une fois la machine virtuelle restaurée, découvrez comment [gérer les sauvegardes](./backup-azure-manage-vms.md)
