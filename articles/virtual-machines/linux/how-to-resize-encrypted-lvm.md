---
title: Redimensionner des disques chiffrés à l’aide de Azure Disk Encryption
description: Cet article fournit des instructions pour le redimensionnement de disques chiffrés ADE en utilisant la gestion des volumes logiques.
author: jofrance
ms.service: virtual-machines-linux
ms.subservice: security
ms.topic: article
ms.author: jofrance
ms.date: 09/21/2020
ms.openlocfilehash: 732a2ef3c6c33d0b17deaf3b9bdc5d1ac5e5934c
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96498912"
---
# <a name="how-to-resize-logical-volume-management-devices-that-use-azure-disk-encryption"></a>Comment redimensionner des appareils de gestion de volumes logiques avec Azure Disk Encryption

Dans cet article, vous allez apprendre à redimensionner des disques de données qui utilisent Azure Disk Encryption. Pour redimensionner les disques, vous devez utiliser le gestionnaire de volumes logiques (LVM) sur Linux. Les étapes s’appliquent à plusieurs scénarios.

Vous pouvez utiliser ce processus de redimensionnement dans les environnements suivants :

- Distributions Linux :
    - Red Hat Enterprise Linux (RHEL) 7 ou version ultérieure
    - Ubuntu 16 ou version ultérieure
    - SUSE 12 ou version ultérieure
- Versions Azure Disk Encryption : 
    - Extension une passe
    - Extension double passe

## <a name="prerequisites"></a>Prérequis

Cet article suppose que vous avez :

- Une configuration LVM existante. Pour plus d’informations, consultez [Configurer LVM sur une machine virtuelle Linux](/previous-versions/azure/virtual-machines/linux/configure-lvm).

- Des disques déjà chiffrés par Azure Disk Encryption. Pour plus d’informations, consultez [Configurer LVM et RAID sur des appareils chiffrés](how-to-configure-lvm-raid-on-crypt.md).

- Une expérience d’utilisation de Linux et LVM.

- Une expérience de l’utilisation des chemins d’accès */dev/disk/scsi1/* aux disques de données sur Azure. Pour plus d’informations, consultez [Résoudre les problèmes relatifs aux noms d’appareil de machine virtuelle Linux](../troubleshooting/troubleshoot-device-names-problems.md). 

## <a name="scenarios"></a>Scénarios

Les procédures décrites dans cet article s’appliquent aux scénarios suivants :

- Configurations LVM et LVM-on-crypt traditionnelles
- Chiffrement LVM traditionnel 
- LVM-on-crypt 

### <a name="traditional-lvm-and-lvm-on-crypt-configurations"></a>Configurations LVM et LVM-on-crypt traditionnelles

Les configurations LVM et LVM-on-crypt traditionnelles étendent un volume logique (LV) lorsque le groupe de volumes (VG) dispose d’un espace disponible.

### <a name="traditional-lvm-encryption"></a>Chiffrement LVM traditionnel 

Dans le chiffrement LVM traditionnel, les volumes logiques sont chiffrés. L’ensemble du disque n’est pas chiffré.

En utilisant le chiffrement LVM traditionnel, vous pouvez :

- Étendre le volume logique lorsque vous ajoutez un nouveau volume physique (PV).
- Étendre le volume logique lorsque vous redimensionnez un volume physique existant.

### <a name="lvm-on-crypt"></a>LVM-on-crypt 

La méthode recommandée pour le chiffrement de disque est LVM-on-Encrypt. Cette méthode chiffre la totalité du disque, pas seulement le volume logique.

En utilisant LVM-on-crypt, vous pouvez : 

- Étendre le volume logique lorsque vous ajoutez un nouveau volume physique.
- Étendre le volume logique lorsque vous redimensionnez un volume physique existant.

> [!NOTE]
> Nous vous déconseillons de mélanger les chiffrements LVM et LVM-on-crypt traditionnels sur la même machine virtuelle.

Les sections suivantes fournissent des exemples d’utilisation de LVM et de LVM-on-Crypt. Les exemples utilisent des valeurs préexistantes pour les disques, les volumes physiques, les groupes de volumes, les volumes logiques, les systèmes de fichiers, les identificateurs uniques universels (UUID) et les points de montage. Remplacez ces valeurs par vos propres valeurs pour les adapter à votre environnement.

#### <a name="extend-an-lv-when-the-vg-has-available-space"></a>Extension d’un volume logique lorsque le groupe de volumes a un espace disponible

La méthode classique pour redimensionner les volumes logiques consiste à étendre un volume logique lorsque le groupe de volumes a de l’espace disponible. Vous pouvez utiliser cette méthode pour les disques non chiffrés, les volumes chiffrés traditionnellement par LVM et les configurations LVM-on-crypt.

1. Vérifiez la taille actuelle du système de fichiers que vous souhaitez augmenter :

    ``` bash
    df -h /mountpoint
    ```

    ![Capture d’écran montrant le code qui vérifie la taille du système de fichiers. La commande et le résultat sont mis en surbrillance.](./media/disk-encryption/resize-lvm/001-resize-lvm-scenarioa-check-fs.png)

2. Vérifiez que le groupe de volumes a suffisamment d’espace pour augmenter le volume logique :

    ``` bash
    vgs
    ```

    ![Capture d’écran montrant le code qui vérifie l’espace sur le groupe de volumes. La commande et le résultat sont mis en surbrillance.](./media/disk-encryption/resize-lvm/002-resize-lvm-scenarioa-check-vgs.png)

    Vous pouvez aussi utiliser `vgdisplay` :

    ``` bash
    vgdisplay vgname
    ```

    ![Capture d’écran montrant le code d’affichage du groupe de volumes qui vérifie l’espace sur le groupe de volumes. La commande et le résultat sont mis en surbrillance.](./media/disk-encryption/resize-lvm/002-resize-lvm-scenarioa-check-vgdisplay.png)

3. Identifiez les besoins en volume logique à redimensionner :

    ``` bash
    lsblk
    ```

    ![Capture d’écran montrant le résultat de la commande l s b l k. La commande et le résultat sont mis en surbrillance.](./media/disk-encryption/resize-lvm/002-resize-lvm-scenarioa-check-lsblk1.png)

    Pour LVM-on-crypt, la différence est que cette sortie indique que la couche chiffrée est au niveau du disque.

    ![Capture d’écran montrant le résultat de la commande l s b l k. La sortie est mise en surbrillance. Cela affiche la couche chiffrée.](./media/disk-encryption/resize-lvm/002-resize-lvm-scenarioa-check-lsblk2.png)

4. Vérifiez la taille du volume logique :

    ``` bash
    lvdisplay lvname
    ```

    ![Capture d’écran montrant le code qui vérifie la taille du volume logique. La commande et le résultat sont mis en surbrillance.](./media/disk-encryption/resize-lvm/002-resize-lvm-scenarioa-check-lvdisplay01.png)

5. Augmentez la taille du volume logique en utilisant `-r` pour redimensionner le système de fichiers en ligne :

    ``` bash
    lvextend -r -L +2G /dev/vgname/lvname
    ```

    ![Capture d’écran montrant le code qui augmente la taille du volume logique. La commande et les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/003-resize-lvm-scenarioa-resize-lv.png)

6. Vérifiez les nouvelles tailles du volume logique et du système de fichiers :

    ``` bash
    df -h /mountpoint
    ```

    ![Capture d’écran montrant le code qui vérifie la taille du volume logique et du système de fichiers. La commande et le résultat sont mis en surbrillance.](./media/disk-encryption/resize-lvm/004-resize-lvm-scenarioa-check-fs.png)

    La sortie de taille indique que le volume logique et le système de fichiers ont été correctement redimensionnés.

Vous pouvez vérifier à nouveau les informations du volume logique pour confirmer les modifications au niveau du volume logique :

``` bash
lvdisplay lvname
```

![Capture d’écran montrant le code qui confirme les nouvelles tailles. Les tailles sont mises en surbrillance.](./media/disk-encryption/resize-lvm/004-resize-lvm-scenarioa-check-lvdisplay2.png)

#### <a name="extend-a-traditional-lvm-volume-by-adding-a-new-pv"></a>Extension d’un volume LVM traditionnel en ajoutant un nouveau volume physique

Lorsque vous devez ajouter un nouveau disque pour augmenter la taille du groupe de volumes, étendez votre volume LVM traditionnel en ajoutant un nouveau volume physique.

1. Vérifiez la taille actuelle du système de fichiers que vous souhaitez augmenter :

    ``` bash
    df -h /mountpoint
    ```

    ![Capture d’écran montrant le code qui vérifie la taille actuelle d’un système de fichiers. La commande et le résultat sont mis en surbrillance.](./media/disk-encryption/resize-lvm/005-resize-lvm-scenariob-check-fs.png)

2. Vérifiez la configuration actuelle du volume physique :

    ``` bash
    pvs
    ```

    ![Capture d’écran montrant le code qui vérifie la configuration actuelle du volume physique. La commande et le résultat sont mis en surbrillance.](./media/disk-encryption/resize-lvm/006-resize-lvm-scenariob-check-pvs.png)

3. Vérifiez les informations actuelles du groupe de volumes :

    ``` bash
    vgs
    ```

    ![Capture d’écran montrant le code qui vérifie les informations actuelles du groupe de volumes. La commande et le résultat sont mis en surbrillance.](./media/disk-encryption/resize-lvm/007-resize-lvm-scenariob-check-vgs.png)

4. Vérifiez le la liste de disques actuelle. Identifiez les disques de données en vérifiant les appareils dans */dev/disk/azure/scsi1/* .

    ``` bash
    ls -l /dev/disk/azure/scsi1/
    ```

    ![Capture d’écran montrant le code qui vérifie la liste de disques actuelle. La commande et les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/008-resize-lvm-scenariob-check-scs1.png)

5. Vérifiez la sortie de `lsblk` : 

    ``` bash
    lsbk
    ```

    ![Capture d’écran montrant le code qui vérifie la sortie de l s b l k. La commande et les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/008-resize-lvm-scenariob-check-lsblk.png)

6. Connectez le nouveau disque à la machine virtuelle en suivant les instructions fournies dans [Joindre un disque de données à une machine virtuelle Linux](attach-disk-portal.md).

7. Vérifiez la liste de disques et notez le nouveau disque.

    ``` bash
    ls -l /dev/disk/azure/scsi1/
    ```

    ![Capture d’écran montrant le code qui vérifie la liste de disques. Les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/009-resize-lvm-scenariob-check-scsi12.png)

    ``` bash
    lsbk
    ```

    ![Capture d’écran montrant le code qui vérifie la liste de disques avec l s b l k. La commande et le résultat sont mis en surbrillance.](./media/disk-encryption/resize-lvm/009-resize-lvm-scenariob-check-lsblk1.png)

8. Créez un nouveau volume physique en plus du nouveau disque de données :

    ``` bash
    pvcreate /dev/newdisk
    ```

    ![Capture d’écran montrant le code qui crée un nouveau volume physique. Le résultat est mis en surbrillance.](./media/disk-encryption/resize-lvm/010-resize-lvm-scenariob-pvcreate.png)

    Cette méthode utilise la totalité du disque comme un volume physique sans partition. Vous pouvez également utiliser `fdisk` pour créer une partition, puis utiliser cette partition pour `pvcreate`.

9. Vérifiez que le volume physique a été ajouté à la liste des volumes physiques :

    ``` bash
    pvs
    ```

    ![Capture d’écran montrant le code qui vérifie la liste de volumes physiques. Le résultat est mis en surbrillance.](./media/disk-encryption/resize-lvm/011-resize-lvm-scenariob-check-pvs1.png)

10. Étendez le groupe de volumes en y ajoutant le nouveau volume physique :

    ``` bash
    vgextend vgname /dev/newdisk
    ```

    ![Capture d’écran montrant le code qui étend le groupe de volumes. Le résultat est mis en surbrillance.](./media/disk-encryption/resize-lvm/012-resize-lvm-scenariob-vgextend.png)

11. Vérifiez la nouvelle taille du groupe de volumes :

    ``` bash
    vgs
    ```

    ![Capture d’écran montrant le code qui vérifie la taille du groupe de volumes. Les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/013-resize-lvm-scenariob-check-vgs1.png)

12. Utilisez `lsblk` pour identifier le volume logique qui doit être redimensionné :

    ``` bash
    lsblk
    ```

    ![Capture d’écran montrant le code qui identifie le volume local qui doit être redimensionné. Les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/013-resize-lvm-scenariob-check-lsblk1.png)

13. Augmentez la taille du volume logique en utilisant `-r` pour augmenter le système de fichiers en ligne :

    ``` bash
    lvextend -r -L +2G /dev/vgname/lvname
    ```

    ![Capture d’écran montrant le code qui augmente la taille du système de fichiers en ligne. Les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/013-resize-lvm-scenariob-lvextend.png) 

14. Vérifiez les nouvelles tailles du volume logique et du système de fichiers :

    ``` bash
    df -h /mountpoint
    ```

    ![Capture d’écran montrant le code qui vérifie les tailles du volume local et du système de fichiers. La commande et le résultat sont mis en surbrillance.](./media/disk-encryption/resize-lvm/014-resize-lvm-scenariob-check-fs1.png)

    >[!IMPORTANT]
    >Quand Azure Data Encryption est utilisé sur des configurations LVM traditionnelles, la couche chiffrée est créée au niveau du volume logique, et non au niveau du disque.
    >
    >À ce stade, la couche chiffrée est étendue au nouveau disque. Le disque de données réel n’a pas de paramètres de chiffrement au niveau de la plateforme, son état de chiffrement n’est donc pas mis à jour.
    >
    >Voici quelques-unes des raisons pour lesquelles LVM-on-crypt est l’approche recommandée. 

15. Vérifiez les informations de chiffrement à partir du portail :

    ![Capture d’écran montrant les informations de chiffrement dans le portail. Le nom du disque et le chiffrement sont mis en surbrillance.](./media/disk-encryption/resize-lvm/014-resize-lvm-scenariob-check-portal1.png)

    Pour mettre à jour les paramètres de chiffrement sur le disque, ajoutez un nouveau volume logique et activez l’extension sur la machine virtuelle.
    
16. Ajoutez un nouveau volume logique, créez-y un système de fichiers, puis ajoutez-le à `/etc/fstab`.

17. Redéfinissez l’extension de chiffrement. Cette fois, vous allez marquer les paramètres de chiffrement sur le nouveau disque de données au niveau de la plateforme. Voici un exemple d’interface CLI :

    ``` bash
    az vm encryption enable -g ${RGNAME} --name ${VMNAME} --disk-encryption-keyvault "<your-unique-keyvault-name>"
    ```

18. Vérifiez les informations de chiffrement à partir du portail :

    ![Capture d’écran montrant les informations de chiffrement dans le portail. Le nom du disque et les informations de chiffrement sont mis en surbrillance.](./media/disk-encryption/resize-lvm/014-resize-lvm-scenariob-check-portal2.png)

Une fois les paramètres de chiffrement mis à jour, vous pouvez supprimer le nouveau volume logique. Supprimez également l’entrée de `/etc/fstab` et `/etc/crypttab` que vous avez créée.

![Capture d’écran montrant le code qui supprime le nouveau volume logique. L’onglet F S supprimé et l’onglet crypt sont mis en surbrillance.](./media/disk-encryption/resize-lvm/014-resize-lvm-scenariob-delete-fstab-crypttab.png)

Pour terminer le nettoyage, procédez comme suit :

1. Démontez le volume logique :

    ``` bash
    umount /mountpoint
    ```

1. Fermez la couche chiffrée du volume :

    ``` bash
    cryptsetup luksClose /dev/vgname/lvname
    ```

1. Supprimez le volume logique :

    ``` bash
    lvremove /dev/vgname/lvname
    ```

#### <a name="extend-a-traditional-lvm-volume-by-resizing-an-existing-pv"></a>Extension d’un volume LVM traditionnel en redimensionnant un volume physique existant

Dans certains scénarios, vos limitations peuvent vous obliger à redimensionner un disque existant. Voici comment faire :

1. Identifiez vos disques chiffrés :

    ``` bash
    ls -l /dev/disk/azure/scsi1/
    ```

    ![Capture d’écran montrant le code qui identifie les disques chiffrés. Les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/015-resize-lvm-scenarioc-check-scsi1.png)

    ``` bash
    lsblk -fs
    ```

    ![Capture d’écran montrant un autre code qui identifie les disques chiffrés. Les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/015-resize-lvm-scenarioc-check-lsblk.png)

2. Vérifiez les informations du volume physique :

    ``` bash
    pvs
    ```

    ![Capture d’écran montrant le code qui vérifie les informations du volume physique. Les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/016-resize-lvm-scenarioc-check-pvs.png)

    Les résultats dans l’image montrent que tout l’espace de l’ensemble des volumes physiques est actuellement utilisé.

3. Vérifiez les informations du groupe de volumes :

    ``` bash
    vgs
    vgdisplay -v vgname
    ```

    ![Capture d’écran montrant le code qui vérifie les informations du groupe de volumes. Les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/017-resize-lvm-scenarioc-check-vgs.png)

4. Vérifiez le tailles de disque. Vous pouvez utiliser `fdisk` ou `lsblk` pour répertorier les tailles de disque.

    ``` bash
    for disk in `ls -l /dev/disk/azure/scsi1/* | awk -F/ '{print $NF}'` ; do echo "fdisk -l /dev/${disk} | grep ^Disk "; done | bash

    lsblk -o "NAME,SIZE"
    ```

    ![Capture d’écran montrant le code qui vérifie les tailles de disque. Les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/018-resize-lvm-scenarioc-check-fdisk.png)

    Ici, nous avons identifié les volumes physiques associés aux volumes logiques avec `lsblk -fs`. Vous pouvez identifier les associations en exécutant `lvdisplay`.

    ``` bash
    lvdisplay --maps VG/LV
    lvdisplay --maps datavg/datalv1
    ```

    ![Capture d’écran montrant une autre façon d’identifier les associations de volumes physiques avec les volumes locaux. Les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/019-resize-lvm-scenarioc-check-lvdisplay.png)

    Dans ce cas, les quatre disques de données font partie du même groupe de volumes et d’un seul volume logique. Votre configuration peut être différente.

5. Vérifiez l’utilisation du système de fichiers actuel :

    ``` bash
    df -h /datalvm*
    ```

    ![Capture d’écran montrant le code qui vérifie l’utilisation du système de fichiers. La commande et les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/020-resize-lvm-scenarioc-check-df.png)

6. Redimensionnez les disques de données en suivant les instructions présentées dans [Extension d’un disque managé Azure](expand-disks.md#expand-an-azure-managed-disk). Vous pouvez utiliser le portail, l’interface CLI ou PowerShell.

    >[!IMPORTANT]
    >Vous ne pouvez pas redimensionner les disques virtuels pendant l’exécution de la machine virtuelle. Libérez votre machine virtuelle pour cette étape.

7. Démarrez la machine virtuelle et vérifiez les nouvelles tailles avec `fdisk`.

    ``` bash
    for disk in `ls -l /dev/disk/azure/scsi1/* | awk -F/ '{print $NF}'` ; do echo "fdisk -l /dev/${disk} | grep ^Disk "; done | bash

    lsblk -o "NAME,SIZE"
    ```

    ![Capture d’écran montrant le code qui vérifie la taille de disque. Le résultat est mis en surbrillance.](./media/disk-encryption/resize-lvm/021-resize-lvm-scenarioc-check-fdisk1.png)

    Dans ce cas, `/dev/sdd` a été redimensionné de 5 G à 20 G.

8. Vérifiez la taille actuelle du volume physique :

    ``` bash
    pvdisplay /dev/resizeddisk
    ```

    ![Capture d’écran montrant le code qui vérifie la taille du volume physique. Le résultat est mis en surbrillance.](./media/disk-encryption/resize-lvm/022-resize-lvm-scenarioc-check-pvdisplay.png)
    
    Même si le disque a été redimensionné, le volume physique garde la taille précédente.

9. Redimensionnez le volume physique :

    ``` bash
    pvresize /dev/resizeddisk
    ```

    ![Capture d’écran montrant le code qui redimensionne le volume physique. Le résultat est mis en surbrillance.](./media/disk-encryption/resize-lvm/023-resize-lvm-scenarioc-check-pvresize.png)


10. Vérifiez la taille du volume physique :

    ``` bash
    pvdisplay /dev/resizeddisk
    ```

    ![Capture d’écran montrant le code qui vérifie la taille du volume physique. Le résultat est mis en surbrillance.](./media/disk-encryption/resize-lvm/024-resize-lvm-scenarioc-check-pvdisplay1.png)

    Appliquez la même procédure pour tous les disques que vous souhaitez redimensionner.

11. Vérifiez les informations du groupe de volumes.

    ``` bash
    vgdisplay vgname
    ```

    ![Capture d’écran montrant le code qui vérifie les informations pour le groupe de volumes. Le résultat est mis en surbrillance.](./media/disk-encryption/resize-lvm/025-resize-lvm-scenarioc-check-vgdisplay1.png)

    Le groupe de volumes a maintenant suffisamment d’espace pour être alloué aux volumes logiques.

12. Redimensionnez le volume logique :

    ``` bash
    lvresize -r -L +5G vgname/lvname
    lvresize -r -l +100%FREE /dev/datavg/datalv01
    ```

    ![Capture d’écran montrant le code qui redimensionne le volume logique. Les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/031-resize-lvm-scenarioc-check-lvresize1.png)

13. Vérifiez la taille du système de fichiers :

    ``` bash
    df -h /datalvm2
    ```

    ![Capture d’écran montrant le code qui vérifie la taille du système de fichiers. Le résultat est mis en surbrillance.](./media/disk-encryption/resize-lvm/032-resize-lvm-scenarioc-check-df3.png)

#### <a name="extend-an-lvm-on-crypt-volume-by-adding-a-new-pv"></a>Extension d’un volume LVM-on-crypt en ajoutant un nouveau volume physique

Vous pouvez aussi étendre un volume LVM-on-crypt en ajoutant un nouveau volume physique. Cette méthode suit attentivement les étapes décrites dans [Configuration de LVM et RAID sur des appareils chiffrés](how-to-configure-lvm-raid-on-crypt.md#general-steps). Consultez les sections qui expliquent comment ajouter un nouveau disque et le configurer dans une configuration LVM-on-crypt.

Vous pouvez utiliser cette méthode pour ajouter de l’espace à un volume logique existant. Autrement, vous pouvez créer de nouveaux groupes de volumes ou volumes logiques.

1. Vérifiez la taille actuelle de votre groupe de volumes :

    ``` bash
    vgdisplay vgname
    ```

    ![Capture d’écran montrant le code qui vérifie la taille du groupe de volumes. Les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/033-resize-lvm-scenarioe-check-vg01.png)

2. Vérifiez la taille actuelle du système de fichiers et du volume logique que vous souhaitez développer :

    ``` bash
    lvdisplay /dev/vgname/lvname
    ```

    ![Capture d’écran montrant le code qui vérifie la taille du volume local. Les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/034-resize-lvm-scenarioe-check-lv01.png)

    ``` bash
    df -h mountpoint
    ```

    ![Capture d’écran montrant le code qui vérifie la taille du système de fichiers. Le résultat est mis en surbrillance.](./media/disk-encryption/resize-lvm/034-resize-lvm-scenarioe-check-fs01.png)

3. Ajoutez un nouveau disque de données à la machine virtuelle et identifiez-le.

    Avant d’ajouter le nouveau disque, vérifiez les disques :

    ``` bash
    fdisk -l | egrep ^"Disk /"
    ```

    ![Capture d’écran montrant le code qui vérifie la taille des disques. Le résultat est mis en surbrillance.](./media/disk-encryption/resize-lvm/035-resize-lvm-scenarioe-check-newdisk01.png)

    Voici une autre façon de vérifier les disques avant l’ajout d’un nouveau disque :

    ``` bash
    lsblk
    ```

    ![Capture d’écran montrant un autre code qui vérifie la taille des disques. Les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/035-resize-lvm-scenarioe-check-newdisk02.png)

    Pour ajouter le nouveau disque, vous pouvez utiliser PowerShell, l’interface de ligne de commande Azure ou le Portail Azure. Pour plus d’informations, consultez [Attacher un disque de données à une machine virtuelle Linux](attach-disk-portal.md).

    Le schéma de nom de noyau s’applique à l’appareil nouvellement ajouté. Un nouveau disque se voit généralement attribuer la prochaine lettre disponible. Dans ce cas, le disque ajouté est `sdd`.

4. Vérifiez les disques pour vous assurer que le nouveau disque a été ajouté :

    ``` bash
    fdisk -l | egrep ^"Disk /"
    ```

    ![Capture d’écran montrant le code qui répertorie les disques. Les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/036-resize-lvm-scenarioe-check-newdisk02.png)

    ``` bash
    lsblk
    ```

    ![Capture d’écran montrant le disque nouvellement ajouté dans la sortie.](./media/disk-encryption/resize-lvm/036-resize-lvm-scenarioe-check-newdisk03.png)

5. Créez un système de fichiers par-dessus le disque récemment ajouté. Faites correspondre le disque aux appareils liés sur `/dev/disk/azure/scsi1/`.

    ``` bash
    ls -la /dev/disk/azure/scsi1/
    ```

    ![Capture d’écran montrant le code qui crée un système de fichiers. Les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/037-resize-lvm-scenarioe-check-newdisk03.png)

    ``` bash
    mkfs.ext4 /dev/disk/azure/scsi1/${disk}
    ```

    ![Capture d’écran montrant le code supplémentaire qui crée un système de fichiers et associe le disque aux appareils liés. Les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/038-resize-lvm-scenarioe-mkfs01.png)

6. Créez un point de montage temporaire pour le nouveau disque ajouté :

    ``` bash
    newmount=/data4
    mkdir ${newmount}
    ```

7. Ajoutez le système de fichiers récemment créé à `/etc/fstab`.

    ``` bash
    blkid /dev/disk/azure/scsi1/lun4| awk -F\" '{print "UUID="$2" '${newmount}' "$4" defaults,nofail 0 0"}' >> /etc/fstab
    ```

8. Montez le système de fichiers nouvellement créé :

    ``` bash
    mount -a
    ```

9. Vérifiez que le nouveau système de fichiers est monté :

    ``` bash
    df -h
    ```

    ![Capture d’écran montrant le code qui vérifie que le système de fichiers est monté. Le résultat est mis en surbrillance.](./media/disk-encryption/resize-lvm/038-resize-lvm-scenarioe-df.png)

    ``` bash
    lsblk
    ```

    ![Capture d’écran montrant le code supplémentaire qui vérifie que le système de fichiers est monté. Le résultat est mis en surbrillance.](./media/disk-encryption/resize-lvm/038-resize-lvm-scenarioe-lsblk.png)

10. Redémarrez le chiffrement que vous avez démarré précédemment pour les disques de données.

    >[!TIP]
    >Nous vous recommandons d’utiliser `EncryptFormatAll` pour LVM-on-crypt. Dans le cas contraire, vous risquez de voir un double chiffrement lors de la définition de disques supplémentaires.
    >
    >Pour plus d’informations, consultez [Configurer LVM et RAID sur des appareils chiffrés](how-to-configure-lvm-raid-on-crypt.md).

    Voici un exemple :

    ``` bash
    az vm encryption enable \
    --resource-group ${RGNAME} \
    --name ${VMNAME} \
    --disk-encryption-keyvault ${KEYVAULTNAME} \
    --key-encryption-key ${KEYNAME} \
    --key-encryption-keyvault ${KEYVAULTNAME} \
    --volume-type "DATA" \
    --encrypt-format-all \
    -o table
    ```

    Une fois le chiffrement terminé, vous voyez une couche de chiffrement sur le disque que vous venez d’ajouter :

    ``` bash
    lsblk
    ```

    ![Capture d’écran montrant le code qui vérifie la couche de chiffrement. Le résultat est mis en surbrillance.](./media/disk-encryption/resize-lvm/038-resize-lvm-scenarioe-lsblk2.png)

11. Démontez la couche chiffrée du nouveau disque :

    ``` bash
    umount ${newmount}
    ```

12. Vérifiez les informations actuelles du volume physique :

    ``` bash
    pvs
    ```

    ![Capture d’écran montrant le code qui vérifie les informations du volume physique. Le résultat est mis en surbrillance.](./media/disk-encryption/resize-lvm/038-resize-lvm-scenarioe-currentpvs.png)

13. Configurez un volume physique par-dessus la couche chiffrée du disque. Prenez le nom de l’appareil à partir de la commande `lsblk` précédente. Ajoutez un mappeur `/dev/` devant le nom de l’appareil pour créer le volume physique :

    ``` bash
    pvcreate /dev/mapper/mapperdevicename
    ```

    ![Capture d’écran montrant le code qui crée un volume physique sur la couche chiffrée. Les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/038-resize-lvm-scenarioe-pvcreate.png)

    Vous voyez un avertissement sur l’effacement de la signature `ext4 fs` actuelle. Cet avertissement est attendu. Répondez à cette question avec `y`.

14. Vérifiez que le nouveau volume physique a été ajouté à la configuration LVM :

    ``` bash
    pvs
    ```

    ![Capture d’écran montrant le code qui vérifie que le volume physique a été ajouté à la configuration LVM. Le résultat est mis en surbrillance.](./media/disk-encryption/resize-lvm/038-resize-lvm-scenarioe-newpv.png)

15. Ajoutez le nouveau volume physique au groupe de volumes que vous devez augmenter.

    ``` bash
    vgextend vgname /dev/mapper/nameofhenewpv
    ```

    ![Capture d’écran montrant le code qui ajoute un volume physique à un groupe de volumes. Les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/038-resize-lvm-scenarioe-vgextent.png)

16. Vérifiez la nouvelle taille et l’espace libre du groupe de volumes :

    ``` bash
    vgdisplay vgname
    ```

    ![Capture d’écran montrant le code qui vérifie la taille et l’espace libre du groupe de volumes. Les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/038-resize-lvm-scenarioe-vgdisplay.png)

    Notez l’augmentation du nombre de `Total PE` et du `Free PE / Size`.

17. Augmentez la taille du volume logique et du système de fichiers. Utilisez l'option `-r` sur `lvextend`. Dans cet exemple, nous ajoutons l’espace total disponible dans le groupe de volumes au volume logique donné.

    ``` bash
    lvextend -r -l +100%FREE /dev/vgname/lvname
    ```

    ![Capture d’écran montrant le code qui augmente la taille du volume local et du système de fichiers. Les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/038-resize-lvm-scenarioe-lvextend.png)

Suivez les étapes suivantes pour vérifier vos modifications.

1. Vérifiez la taille du volume logique :

    ``` bash
    lvdisplay /dev/vgname/lvname
    ```

    ![Capture d’écran montrant le code qui vérifie la nouvelle taille du volume local. Les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/038-resize-lvm-scenarioe-lvdisplay.png)

1. Vérifiez la nouvelle taille du système de fichiers :

    ``` bash
    df -h mountpoint
    ```

    ![Capture d’écran montrant le code qui vérifie la nouvelle taille du système de fichiers. Le résultat est mis en surbrillance.](./media/disk-encryption/resize-lvm/038-resize-lvm-scenarioe-df1.png)

1. Vérifiez que la couche LVM se trouve au-dessus de la couche chiffrée :

    ``` bash
    lsblk
    ```

    ![Capture d’écran montrant le code qui vérifie que la couche LVM se trouve au-dessus de la couche chiffrée. Le résultat est mis en surbrillance.](./media/disk-encryption/resize-lvm/038-resize-lvm-scenarioe-lsblk3.png)

    Si vous utilisez `lsblk` sans options, les points de montage s’affichent alors plusieurs fois. La commande effectue un tri par appareil et volumes logiques. 

    Pour voudrez peut-être utiliser `lsblk -fs`. Dans cette commande, `-fs` inverse l’ordre de tri afin que les points de montage soient affichés une seule fois. Les disques sont affichés plusieurs fois.

    ``` bash
    lsblk -fs
    ```

    ![Capture d’écran montrant un autre code qui vérifie que la couche LVM se trouve au-dessus de la couche chiffrée. Le résultat est mis en surbrillance.](./media/disk-encryption/resize-lvm/038-resize-lvm-scenarioe-lsblk4.png)

#### <a name="extend-an-lvm-on-a-crypt-volume-by-resizing-an-existing-pv"></a>Extension d’un LVM sur un volume chiffré en redimensionnant un volume physique existant

1. Identifiez vos disques chiffrés :

    ``` bash
    lsblk
    ```

    ![Capture d’écran montrant le code qui identifie les disques chiffrés. Les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/039-resize-lvm-scenariof-lsblk01.png)

    ``` bash
    lsblk -s
    ```

    ![Capture d’écran montrant un autre code qui identifie les disques chiffrés. Les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/040-resize-lvm-scenariof-lsblk012.png)

2. Vérifiez les informations de votre volume physique :

    ``` bash
    pvs
    ```

    ![Capture d’écran montrant le code qui vérifie les informations pour les volumes physiques. Les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/041-resize-lvm-scenariof-pvs.png)

3. Vérifiez les informations de votre groupe de volumes :

    ``` bash
    vgs
    ```

    ![Capture d’écran montrant le code qui vérifie les informations pour les groupes de volumes. Les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/042-resize-lvm-scenariof-vgs.png)

4. Vérifiez les informations de votre volume logique :

    ``` bash
    lvs
    ```

    ![Capture d’écran montrant le code qui vérifie les informations pour le volume local. Le résultat est mis en surbrillance.](./media/disk-encryption/resize-lvm/043-resize-lvm-scenariof-lvs.png)

5. Vérifiez l'utilisation de système de fichiers :

    ``` bash
    df -h /mountpoint(s)
    ```

    ![Capture d’écran montrant le code qui vérifie la quantité de système de fichiers utilisée. Les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/044-resize-lvm-scenariof-fs.png)

6. Vérifiez les tailles de vos disques :

    ``` bash
    fdisk
    fdisk -l | egrep ^"Disk /"
    lsblk
    ```

    ![Capture d’écran montrant le code qui vérifie la taille des disques. Les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/045-resize-lvm-scenariof-fdisk01.png)

7. Redimensionnez le disque de données. Vous pouvez utiliser le portail, l’interface CLI ou PowerShell. Pour plus d’informations, reportez-vous à la section de redimensionnement de disque dans [Développer des disques durs virtuels sur un machine virtuelle Linux](expand-disks.md#expand-an-azure-managed-disk). 

    >[!IMPORTANT]
    >Vous ne pouvez pas redimensionner les disques virtuels pendant l’exécution de la machine virtuelle. Libérez votre machine virtuelle pour cette étape.

8. Vérifiez les tailles de vos disques :

    ``` bash
    fdisk
    fdisk -l | egrep ^"Disk /"
    lsblk
    ```

    ![Capture d’écran montrant le code qui vérifie les tailles des disques. Les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/046-resize-lvm-scenariof-fdisk02.png)

    Dans ce cas, les deux disques ont été redimensionnés de 2 Go à 4 Go. Toutefois, la taille du système de fichiers, du volume logique et du volume physique reste inchangée.

9. Vérifiez la taille actuelle du volume physique. N’oubliez pas que sur LVM-on-crypt, le volume physique est l’appareil `/dev/mapper/`, et non l’appareil `/dev/sd*`.

    ``` bash
    pvdisplay /dev/mapper/devicemappername
    ```

    ![Capture d’écran montrant le code qui vérifie la taille du volume physique actuel. Les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/047-resize-lvm-scenariof-pvs.png)

10. Redimensionnez le volume physique :

    ``` bash
    pvresize /dev/mapper/devicemappername
    ```

    ![Capture d’écran montrant le code qui redimensionne le volume physique. Les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/048-resize-lvm-scenariof-resize-pv.png)

11. Vérifiez la nouvelle taille du volume physique :

    ``` bash
    pvdisplay /dev/mapper/devicemappername
    ```

    ![Capture d’écran montrant le code qui vérifie la taille du volume physique. Les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/049-resize-lvm-scenariof-pv.png)

12. Redimensionnez la couche chiffrée sur le volume physique :

    ``` bash
    cryptsetup resize /dev/mapper/devicemappername
    ```

    Appliquez la même procédure pour tous les disques que vous souhaitez redimensionner.

13. Vérifiez les informations de votre groupe de volumes :

    ``` bash
    vgdisplay vgname
    ```

    ![Capture d’écran montrant le code qui vérifie les informations pour le groupe de volumes. Les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/050-resize-lvm-scenariof-vg.png)

    Le groupe de volumes a maintenant suffisamment d’espace pour être alloué aux volumes logiques.

14. Vérifiez les informations du volume lpgique :

    ``` bash
    lvdisplay vgname/lvname
    ```

    ![Capture d’écran montrant le code qui vérifie les informations pour le volume local. Les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/051-resize-lvm-scenariof-lv.png)

15. Vérifiez l'utilisation de système de fichiers :

    ``` bash
    df -h /mountpoint
    ```

    ![Capture d’écran montrant le code qui vérifie l’utilisation du système de fichiers. Les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/052-resize-lvm-scenariof-fs.png)

16. Redimensionnez le volume logique :

    ``` bash
    lvresize -r -L +2G /dev/vgname/lvname
    ```

    ![Capture d’écran montrant le code qui redimensionne le volume local. Les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/053-resize-lvm-scenariof-lvresize.png)

    Ici, nous utilisons l’option `-r` pour redimensionner également le système de fichiers.

17. Vérifiez les informations du volume lpgique :

    ``` bash
    lvdisplay vgname/lvname
    ```

    ![Capture d’écran montrant le code qui obtient des informations sur le volume local. Les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/054-resize-lvm-scenariof-lvsize.png)

18. Vérifiez l'utilisation de système de fichiers :

    ``` bash
    df -h /mountpoint
    ```

    ![Capture d’écran montrant le code qui vérifie l’utilisation du système de fichiers. Les résultats sont mis en surbrillance.](./media/disk-encryption/resize-lvm/055-resize-lvm-scenariof-fs.png)

Appliquez la même procédure de redimensionnement à tout autre volume logique qui l’exige.

## <a name="next-steps"></a>Étapes suivantes

[Résolution des problèmes Azure Disk Encryption](disk-encryption-troubleshooting.md)