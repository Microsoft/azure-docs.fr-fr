---
title: Solution Azure VMware – Migration à l’aide d’Azure Data Box
description: Comment utiliser Azure Data Box pour migrer des données en bloc vers une solution Azure VMware.
author: sharaths-cs
ms.author: dikamath
ms.date: 09/27/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: ab772bd9cb415045ef70cb4cf9a518791befb192
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86507656"
---
# <a name="migrating-data-to-azure-vmware-solution-by-using-azure-data-box"></a>Migration de données vers une solution Azure VMware à l’aide d’Azure Data Box

La solution cloud Microsoft Azure Data Box permet d’envoyer plusieurs téraoctets (To) de données vers Azure de façon rapide, économique et fiable. Le transfert de données sécurisé est accéléré par l’envoi à votre adresse d’un appareil de stockage Data Box propriétaire. Chaque appareil de stockage dispose d’une capacité de stockage maximale utile de 80 To et est acheminé vers votre centre de données par un transporteur régional. L'appareil dispose d'un caisson robuste afin de protéger et de sécuriser vos données pendant le transport.

En utilisant la Data Box, vous pouvez migrer en bloc vos données VMware vers votre cloud privé. Les données de votre environnement VMware vSphere local sont copiées sur la Data Box via le protocole NFS (Network File System). La migration de données en bloc consiste à enregistrer une copie à un point dans le temps de machines virtuelles, de la configuration et des données associées sur la Data Box et à l’expédier manuellement à Azure.

Cet article porte sur les points suivants :

* Configuration de la Data Box.
* Copie de données de l’environnement VMware local sur la Data Box via NFS.
* Préparation du retour de la Data Box.
* Préparation des données blob à copier dans la solution Azure VMware.
* Copie des données d’Azure vers votre cloud privé.

## <a name="scenarios"></a>Scénarios

Servez-vous de la Data Box dans les scénarios de migration de données en bloc suivants :

* Migration d’une grande quantité de données de l’environnement local vers la solution Azure VMware. Cette méthode établit une base de référence et synchronise les différences sur le réseau.
* Migration d’un grand nombre de machines virtuelles désactivées (machines virtuelles froides).
* Migration de données de machines virtuelles pour configurer des environnements de développement et de test.
* Migration d’un grand nombre de modèles de machine virtuelle, de fichiers ISO et de disques de machine virtuelle.

## <a name="before-you-begin"></a>Avant de commencer

* Vérifiez les prérequis et [commandez la Data Box](../databox/data-box-deploy-ordered.md) à partir de votre portail Azure. Au moment de passer la commande, vous devez sélectionner un compte de stockage qui permet le stockage d’objets blob. Après avoir reçu l’appareil Data Box, connectez-le à votre réseau local et [configurez-le](../databox/data-box-deploy-set-up.md) avec une adresse IP accessible à partir de votre réseau de gestion vSphere.

* Créez un réseau virtuel et un compte de stockage dans la région où votre solution Azure VMware est provisionnée.

* Créez une [connexion de réseau virtuel Azure](cloudsimple-azure-network-connection.md) entre votre cloud privé et le réseau virtuel où le compte de stockage a été créé en suivant les étapes décrites dans [Connecter un réseau virtuel Azure à CloudSimple à l’aide d’ExpressRoute](virtual-network-connection.md).

## <a name="set-up-data-box-for-nfs"></a>Configurer la Data Box pour NFS

Connectez-vous à l’interface utilisateur web locale de votre Data Box en suivant les étapes décrites dans la section « Se connecter à l’appareil » du [Tutoriel : Branchement des câbles et connexion à une Azure Data Box](../databox/data-box-deploy-set-up.md).  Configurez la Data Box pour autoriser l’accès aux clients NFS :

1. Dans l’interface utilisateur web locale, accédez à la page **Connexion et copie**. En dessous de **Paramètres NFS**, sélectionnez **Accès au client NFS**. 

    ![Configurer l’accès au client NFS 1](media/nfs-client-access.png)

2. Entrez l’adresse IP des hôtes VMware ESXi, puis sélectionnez **Ajouter**. Vous pouvez configurer l’accès pour tous les hôtes de votre cluster vSphere en répétant cette étape. Sélectionnez **OK**.

    ![Configurer l’accès au client NFS 2](media/nfs-client-access2.png)
> [!IMPORTANT]
> **Toujours créer un dossier pour les fichiers que vous envisagez de copier sous le partage, puis copier les fichiers dans ce dossier**. Le dossier créé sous les partages d’objets blob de pages et d’objets blob de blocs représente un conteneur dans lequel les données sont chargées en tant qu’objets blob. Vous ne pouvez pas copier de fichiers directement dans le dossier *root* du compte de stockage.

Sous les partages d’objet blob de blocs et d’objet blob de pages, les entités de premier niveau sont des conteneurs et les entités de second niveau sont des objets blob. Sous les partages Azure Files, les entités de premier niveau sont des partages et les entités de second niveau sont des fichiers.

Le tableau suivant montre le chemin UNC aux partages sur votre Data Box et l’URL de chemin Stockage Azure où les données sont chargées. La dernière URL de chemin de Stockage Azure peut être dérivée à partir du chemin de partage UNC.
 
| Blobs et fichiers | Chemin d’accès et URL |
|---------------- | ------------ |
| Objets blob de blocs Azure | <li>Chemin UNC aux partages : `//<DeviceIPAddress>/<StorageAccountName_BlockBlob>/<ContainerName>/files/a.txt`</li><li>URL de Stockage Azure : `https://<StorageAccountName>.blob.core.windows.net/<ContainerName>/files/a.txt`</li> |  
| Objets blob de pages Azure  | <li>Chemin UNC aux partages : `//<DeviceIPAddres>/<StorageAccountName_PageBlob>/<ContainerName>/files/a.txt`</li><li>URL de Stockage Azure : `https://<StorageAccountName>.blob.core.windows.net/<ContainerName>/files/a.txt`</li>   |  
| Azure Files       |<li>Chemin UNC aux partages : `//<DeviceIPAddres>/<StorageAccountName_AzFile>/<ShareName>/files/a.txt`</li><li>URL de Stockage Azure : `https://<StorageAccountName>.file.core.windows.net/<ShareName>/files/a.txt`</li>        |

> [!NOTE]
> Utilisez des objets blob de blocs Azure pour copier des données VMware.

## <a name="mount-the-nfs-share-as-a-datastore-on-your-on-premises-vcenter-cluster-and-copy-the-data"></a>Monter le partage NFS en tant que magasin de données sur un cluster vCenter local et copier les données

Le partage NFS de votre Data Box doit être monté en tant que magasin de données sur votre cluster vCenter local ou votre hôte VMware ESXi pour copier les données dans le magasin de données NFS :

1. Connectez-vous à votre serveur vCenter local.

2. Cliquez avec le bouton droit sur **Datacenter** (Centre de données), puis sélectionnez successivement **Storage** (Stockage), **New Datastore** (Nouveau magasin de données) et **Next** (Suivant).

   ![Ajouter un nouveau magasin de données](media/databox-migration-add-datastore.png)

3. À l’étape 1 de l’Assistant Add Datastore (Ajout d’un magasin de données), sélectionnez **NFS** en dessous de **Type**.

   ![Ajouter un nouveau magasin de données - type](media/databox-migration-add-datastore-type.png)

4. À l’étape 2 de l’Assistant, sélectionnez **NFS 3** pour la version de NFS, puis sélectionnez **Next**.

   ![Ajouter un nouveau magasin de données - Version de NFS](media/databox-migration-add-datastore-nfs-version.png)

5. À l’étape 3 de l’Assistant, spécifiez le nom du magasin de données, le chemin, puis le serveur. Vous pouvez utiliser l’adresse IP de votre Data Box pour le serveur. Le chemin du dossier doit être au format `/<StorageAccountName_BlockBlob>/<ContainerName>/`.

   ![Ajouter un nouveau magasin de données - Configuration de NFS](media/databox-migration-add-datastore-nfs-configuration.png)

6. À l’étape 4 de l’Assistant, sélectionnez les hôtes ESXi où vous voulez monter le magasin de données, puis sélectionnez **Next**.  Dans un cluster, sélectionnez tous les hôtes pour assurer la migration des machines virtuelles.

   ![Ajouter un nouveau magasin de données - Sélectionner les hôtes](media/databox-migration-add-datastore-nfs-select-hosts.png)

7. À l’étape 5 de l’Assistant, examinez le récapitulatif, puis sélectionnez **Finish** (Terminer).

## <a name="copy-data-to-the-data-box-nfs-datastore"></a>Copier des données dans le magasin de données NFS Data Box

Les machines virtuelles peuvent être migrées ou clonées vers le nouveau magasin de données.  Les machines virtuelles non utilisées peuvent éventuellement être migrées vers le magasin de données NFS Data Box à l’aide de l’option **storage vMotion**. Les machines virtuelles actives peuvent être clonées dans le magasin données NFS Data Box.

* Identifiez et listez les machines virtuelles qui peuvent être **déplacées**.
* Identifiez et listez les machines virtuelles qui doivent être **clonées**.

### <a name="move-a-virtual-machine-to-a-data-box-datastore"></a>Déplacer une machine virtuelle vers le magasin de données d’une Data Box

1. Cliquez avec le bouton droit sur la machine virtuelle que vous voulez déplacer vers le magasin de données Data Box, puis sélectionnez **Migrate** (Migrer).

    ![Migrer une machine virtuelle](media/databox-migration-vm-migrate.png)

2. Sélectionnez **Change storage only** (Changer le stockage uniquement) comme type de migration, puis cliquez sur **Next**.

    ![Migrer une machine virtuelle - Stockage seulement](media/databox-migration-vm-migrate-change-storage.png)

3. Sélectionnez **Databox-Datastore** (Magasin de données Data Box) comme destination, puis sélectionnez **Next**.

    ![Migrer une machine virtuelle - Sélectionner un magasin de données](media/databox-migration-vm-migrate-change-storage-select-datastore.png)

4. Vérifiez les informations, puis sélectionnez **Finish** (Terminer).

5. Répétez les étapes 1 à 4 pour d’autres machines virtuelles.

> [!TIP]
> Vous pouvez sélectionner plusieurs machines virtuelles qui se trouvent dans le même état d’alimentation (activé ou désactivé), puis les migrer en bloc.

La machine virtuelle est migrée vers le magasin de données NFS Data Box. Une fois que toutes les machines virtuelles ont été migrées, vous pouvez désactiver (arrêter) les machines virtuelles actives en vue de la migration des données vers la solution Azure VMware.

### <a name="clone-a-virtual-machine-or-a-virtual-machine-template-to-the-data-box-datastore"></a>Cloner une machine virtuelle ou un modèle de machine virtuelle dans le magasin de données Data Box

1. Cliquez avec le bouton droit sur la machine virtuelle ou le modèle de machine virtuelle que vous voulez cloner. Sélectionnez **Clone** (Cloner) > **Clone to Virtual Machine** (Cloner sur une machine virtuelle).

    ![Clone de machine virtuelle](media/databox-migration-vm-clone.png)

2. Sélectionnez un nom pour la machine virtuelle clonée ou le modèle de machine virtuelle.

3. Sélectionnez le dossier où vous voulez placer l’objet cloné, puis sélectionnez **Next**.

4. Sélectionnez le cluster ou la liste de ressources partagées où vous voulez placer l’objet cloné, puis sélectionnez **Next**.

5. Sélectionnez **Databox-Datastore** (Magasin de données Data Box) comme emplacement de stockage, puis sélectionnez **Next**.

    ![Clone de machine virtuelle - Sélectionner un magasin de données](media/databox-migration-vm-clone-select-datastore.png)

6. Si vous voulez personnaliser des options pour l’objet cloné, sélectionnez les options de personnalisation, puis **Next**.

7. Vérifiez les configurations, puis sélectionnez **Finish** (Terminer).

8. Répétez les étapes 1 à 7 pour d’autres machines virtuelles ou modèles de machine virtuelle.

Les machines virtuelles sont clonées et stockées dans le magasin de données NFS Data Box. Une fois les machines virtuelles clonées, vérifiez qu’elles sont arrêtées en vue de la migration des données vers la solution Azure VMware.

### <a name="copy-iso-files-to-the-data-box-datastore"></a>Copier des fichiers ISO dans le magasin de données Data Box

1. À partir de l’interface utilisateur web de votre vCenter local, accédez à **Storage** (Stockage).  Sélectionnez **Databox-Datastore** (Magasin de données DataBox), puis sélectionnez **Files** (Fichiers). Créez un dossier pour le stockage des fichiers ISO.

    ![Copier un fichier ISO - Créer un dossier](media/databox-migration-create-folder.png)

2. Nommez le dossier dans lequel les fichiers ISO seront stockés.

3. Double-cliquez sur le dossier nouvellement créé pour l’ouvrir.

4. Sélectionnez **Upload Files** (Charger des fichiers), puis sélectionnez les fichiers ISO que vous voulez charger.
    
    ![Copier un fichier ISO - Charger des fichiers](media/databox-migration-upload-iso.png)

> [!TIP]
> Si votre magasin de données local contient déjà des fichiers ISO, vous pouvez sélectionner les fichiers et **Copy to** pour copier les fichiers dans le magasin de données NFS Data Box.


## <a name="prepare-data-box-for-return"></a>Préparer la Data Box pour le retour

Une fois que l’ensemble des données des machines virtuelles, des données des modèles de machine virtuelle et des fichiers ISO ont été copiés dans le magasin de données NFS Data Box, vous pouvez déconnecter le magasin de données de votre vCenter. Toutes les machines virtuelles et tous les modèles de machine virtuelle doivent être supprimés de l’inventaire avant de déconnecter le magasin de données.

### <a name="remove-objects-from-inventory"></a>Supprimer des objets de l’inventaire

1. À partir de l’interface utilisateur web de votre vCenter local, accédez à **Storage** (Stockage). Sélectionnez **Databox-Datastore** (Magasin de données DataBox), puis **VMs** (Machines virtuelles).

    ![Supprimer les machines virtuelles de l’inventaire – désactivées](media/databox-migration-select-databox-vm.png)

2. Vérifiez que toutes les machines virtuelles sont arrêtées.

3. Sélectionnez toutes les machines virtuelles, cliquez avec le bouton droit, puis sélectionnez **Remove from inventory** (Supprimer de l’inventaire).

    ![Supprimer les machines virtuelles de l’inventaire](media/databox-migration-remove-vm-from-inventory.png)

4. Sélectionnez **VM Templates in Folders** (Modèles de machine virtuelle dans les dossiers), puis répétez l’étape 3.

### <a name="remove-the-data-box-nfs-datastore-from-vcenter"></a>Supprimer le magasin de données NFS Data Box du vCenter

Le magasin de données NFS Data Box doit être déconnecté des hôtes VMware ESXi avant de préparer le retour.

1. À partir de l’interface utilisateur web de votre vCenter local, accédez à **Storage** (Stockage).

2. Cliquez avec le bouton droit sur **Databox-Datastore** (Magasin de données DataBox), puis sélectionnez **Unmount Datastore** (Démonter le magasin).

    ![Démonter le magasin de données Data Box](media/databox-migration-unmount-datastore.png)

3. Sélectionnez tous les hôtes ESXi où le magasin de stockage est monté, puis sélectionnez **OK**.

    ![Démonter le magasin de données Data Box - Sélectionner les hôtes](media/databox-migration-unmount-datastore-select-hosts.png)

4. Examinez et acceptez les avertissements éventuels, puis sélectionnez **OK**.

### <a name="prepare-data-box-for-return-and-then-return-it"></a>Préparer la Data Box pour le retour et la retourner

Suivez les étapes décrites dans l’article [Retourner l’appareil Azure Data Box et vérifier le chargement des données dans Azure](../databox/data-box-deploy-picked-up.md) pour retourner la Data Box. Vérifiez l’état de la copie des données dans votre compte de stockage Azure. Une fois que l’état indique terminé, vous pouvez vérifier les données dans votre compte de stockage Azure.

## <a name="copy-data-from-azure-storage-to-azure-vmware-solution"></a>Copier des données du stockage Azure vers la solution Azure VMware

Les données copiées sur votre appareil Data Box sont disponibles sur votre compte de stockage Azure une fois que l’état de la commande de votre Data Box indique terminé. Les données peuvent dès lors être copiées dans votre solution Azure VMware. Les données du compte de stockage doivent être copiées dans le magasin de données vSAN de votre cloud privé en utilisant le protocole NFS. 

Dans un premier temps, copiez les données du stockage Blob sur un disque managé de machine virtuelle Linux dans Azure avec **AzCopy**. Rendez le disque managé disponible via NFS, montez le partage NFS en tant que magasin de données sur votre cloud privé, puis copiez les données. Cette méthode permet de copier plus rapidement les données sur votre cloud privé.

### <a name="copy-data-to-your-private-cloud-using-a-linux-virtual-machine-and-managed-disks-and-then-export-as-nfs-share"></a>Copier des données sur votre cloud privé en utilisant une machine virtuelle Linux et des disques managés, puis les exporter en tant que partage NFS

1. Créez une [machine virtuelle Linux](../virtual-machines/linux/quick-create-portal.md) dans Azure, dans la même région que celle où votre compte de stockage a été créé et qui dispose d’une connexion de réseau virtuel Azure à votre cloud privé.

2. Créez un disque managé d’une capacité de stockage supérieure à la quantité de données blob, puis[attachez-le à votre machine virtuelle Linux](../virtual-machines/linux/attach-disk-portal.md).  Si la quantité de données blob est supérieure à la capacité du plus grand disque managé disponible, les données doivent être copiées en plusieurs étapes ou en utilisant plusieurs disques managés.

3. Connectez-vous à la machine virtuelle Linux et montez le disque managé.

4. Installez [AzCopy sur votre machine virtuelle Linux](../storage/common/storage-use-azcopy-v10.md).

5. Téléchargez les données de votre stockage Blob Azure vers le disque managé avec AzCopy.  Syntaxe de la commande : `azcopy copy "https://<storage-account-name>.blob.core.windows.net/<container-name>/*" "<local-directory-path>/"`.  Remplacez `<storage-account-name>` par le nom de votre compte de stockage Azure et `<container-name>` par le conteneur qui contient les données copiées via la Data Box.

6. Installez le serveur NFS sur votre machine virtuelle Linux :

    - Sur une distribution Ubuntu/Debian : `sudo apt install nfs-kernel-server`.
    - Sur une distribution Enterprise Linux : `sudo yum install nfs-utils`.

7. Changez l’autorisation du dossier sur votre disque managé où les données du stockage Blob Azure ont été copiées.  Changez les autorisations pour tous les dossiers que vous voulez exporter en tant que partage NFS.

    ```bash
    chmod -R 755 /<folder>/<subfolder>
    chown nfsnobody:nfsnobody /<folder>/<subfolder>
    ```

8. Attribuez des autorisations pour permettre aux adresses IP clientes d’accéder au partage NFS en modifiant le fichier `/etc/exports`.

    ```bash
    sudo vi /etc/exports
    ```
    
    Entrez les lignes suivantes dans le fichier pour chaque adresse IP de l’hôte ESXi de votre cloud privé.  Si vous créez des partages pour plusieurs dossiers, ajoutez tous les dossiers.

    ```bash
    /<folder>/<subfolder> <ESXiNode1IP>(rw,sync,no_root_squash,no_subtree_check)
    /<folder>/<subfolder> <ESXiNode2IP>(rw,sync,no_root_squash,no_subtree_check)
    .
    .
    ```

9. Exportez les partages NFS en utilisant la commande `sudo exportfs -a`.

10. Redémarrez le serveur du noyau NFS en utilisant la commande `sudo systemctl restart nfs-kernel-server`.


### <a name="mount-the-linux-virtual-machine-nfs-share-as-a-datastore-on-a-private-cloud-vcenter-cluster-and-then-copy-data"></a>Monter le partage NFS de la machine virtuelle Linux en tant que magasin de données sur un cluster vCenter de cloud privé et copier des données

Le partage NFS de votre machine virtuelle Linux doit être monté en tant que magasin de données sur le cluster vCenter de votre cloud privé. Une fois qu’il est monté, les données peuvent être copiées du magasin de données NFS vers le magasin de données vSAN du cloud privé.

1. Connectez-vous au serveur vCenter de votre cloud privé.

2. Cliquez avec le bouton droit sur **Datacenter** (Centre de données), puis sélectionnez successivement **Storage** (Stockage), **New Datastore** (Nouveau magasin de données) et **Next** (Suivant).

   ![Ajouter un nouveau magasin de données](media/databox-migration-add-datastore.png)

3. À l’étape 1 de l’Assistant Add Datastore (Ajout d’un magasin de données), sélectionnez le type **NFS**.

   ![Ajouter un nouveau magasin de données - type](media/databox-migration-add-datastore-type.png)

4. À l’étape 2 de l’Assistant, sélectionnez **NFS 3** pour la version de NFS, puis sélectionnez **Next**.

   ![Ajouter un nouveau magasin de données - Version de NFS](media/databox-migration-add-datastore-nfs-version.png)

5. À l’étape 3 de l’Assistant, spécifiez le nom du magasin de données, le chemin, puis le serveur.  Vous pouvez utiliser l’adresse IP de votre machine virtuelle Linux pour le serveur.  Le chemin du dossier doit être au format `/<folder>/<subfolder>/`.

   ![Ajouter un nouveau magasin de données - Configuration de NFS](media/databox-migration-add-datastore-nfs-configuration.png)

6. À l’étape 4 de l’Assistant, sélectionnez les hôtes ESXi où vous voulez monter le magasin de données, puis sélectionnez **Next**.  Dans un cluster, sélectionnez tous les hôtes pour assurer la migration des machines virtuelles.

   ![Ajouter un nouveau magasin de données - Sélectionner les hôtes](media/databox-migration-add-datastore-nfs-select-hosts.png)

7. À l’étape 5 de l’Assistant, examinez le récapitulatif, puis sélectionnez **Finish** (Terminer).

### <a name="add-virtual-machines-and-virtual-machine-templates-from-an-nfs-datastore-to-the-inventory"></a>Ajouter des machines virtuelles et des modèles de machine virtuelle d’un magasin de données NFS à l’inventaire

1. À partir de l’interface utilisateur web du vCenter de votre cloud privé, accédez à **Storage** (Stockage).  Sélectionnez un magasin de données NFS de machine virtuelle Linux, puis sélectionnez **Files** (Fichiers).

    ![Sélectionner des fichiers dans un magasin de données NFS](media/databox-migration-datastore-select-files.png)

2. Sélectionnez un dossier qui contient une machine virtuelle ou un modèle de machine virtuelle.  Dans le volet d’informations, sélectionnez un fichier .vmx pour une machine virtuelle ou un fichier .vmtx pour un modèle de machine virtuelle.

3. Sélectionnez **Register VM** (Inscrire la machine virtuelle) pour inscrire la machine virtuelle sur le vCenter de votre cloud privé.

    ![Inscrire une machine virtuelle](media/databox-migration-datastore-register-vm.png)

4. Sélectionnez le centre de ressources, le dossier et le cluster/liste de ressources partagées où vous voulez inscrire la machine virtuelle.

4. Répétez les étapes 3 et 4 pour toutes les machines virtuelles et tous les modèles de machine virtuelle.

5. Accédez au dossier qui contient les fichiers ISO.  Sélectionnez les fichiers ISO et **Copy to** pour copier les fichiers dans un dossier de votre magasin de données vSAN.

Les machines virtuelles et les modèles de machine virtuelle sont désormais disponibles sur le vCenter de votre cloud privé. Ces machines virtuelles doivent être déplacées du magasin de données NFS vers le magasin de données vSAN avant de les activer. Vous pouvez utiliser l’option **storage vMotion** et sélectionner le magasin de données vSAN comme cible pour les machines virtuelles.

Les modèles de machine virtuelle doivent être clonés à partir du magasin de données NFS de votre machine virtuelle Linux vers votre magasin de données vSAN.

### <a name="clean-up-your-linux-virtual-machine"></a>Nettoyer votre machine virtuelle Linux

Une fois que toutes les données ont été copiées sur votre cloud privé, vous pouvez supprimer le magasin de données NFS de votre cloud privé :

1. Vérifiez que toutes les machines virtuelles et tous les modèles sont déplacés et clonés dans votre magasin de données vSAN.

2. Supprimez de l’inventaire tous les modèles de machine virtuelle à partir du magasin de données NFS.

3. Démontez le magasin de données de la machine virtuelle Linux à partir du vCenter de votre cloud privé.

4. Supprimez la machine virtuelle et le disque managé d’Azure.

5. Si vous ne souhaitez pas conserver les données qui ont été transférées par la Data Box dans votre compte de stockage, supprimez le compte de stockage Azure.  
    


## <a name="next-steps"></a>Étapes suivantes

* Apprenez-en davantage sur la [Data Box](../databox/data-box-overview.md).
* Découvrez plus en détail les différentes options de [migration de charges de travail vers votre cloud privé](migrate-workloads.md).
