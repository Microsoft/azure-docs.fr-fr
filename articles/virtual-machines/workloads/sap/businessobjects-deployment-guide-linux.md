---
title: Déploiement de la plateforme décisionnelle SAP BusinessObjects sur Azure pour Linux | Microsoft Docs
description: Déployer et configurer le plateforme décisionnelle SAP BusinessObjects sur Azure pour Linux
services: virtual-machines-windows,virtual-network,storage,azure-netapp-files,azure-files,mysql
documentationcenter: saponazure
author: dennispadia
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 10/05/2020
ms.author: depadia
ms.openlocfilehash: 647009854ef5a0c0811fc303914f724272f1a3f5
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96486655"
---
# <a name="sap-businessobjects-bi-platform-deployment-guide-for-linux-on-azure"></a>Guide de déploiement de la plateforme SAP BusinessObjects BI pour Linux sur Azure

Cet article décrit la stratégie de déploiement de la plateforme SAP BOBI sur Azure pour Linux. Dans cet exemple, deux machines virtuelles avec Disques managés SSD Premium comme répertoire d’installation sont configurées. Azure Database pour MySQL est utilisé pour la base de données de système de gestion de contenu, et Azure NetApp Files pour le serveur de référentiel de fichiers est partagé entre les deux serveurs. L’application web Java par défaut Tomcat et l’application de plateforme BI sont installées ensemble sur les deux machines virtuelles. Pour équilibrer la charge de la demande de l’utilisateur, Application Gateway est utilisé avec des capacités de déchargement TLS/SSL natives.

Ce type d’architecture est efficace pour un déploiement de petite taille ou un environnement de non production. Pour un déploiement de production ou à grande échelle, vous pouvez avoir des hôtes distincts pour l’application web ainsi que plusieurs hôtes d’applications BOBI permettant au serveur de traiter plus d’informations.

![Déploiement de SAP BOBI sur Azure pour Linux](media/businessobjects-deployment-guide/businessobjects-deployment-linux.png)

Dans cet exemple, la version du produit et la disposition du système de fichiers ci-dessous sont utilisées :

- Plateforme SAP BusinessObjects 4.3
- SUSE Linux Enterprise Server 12 SP5
- Azure Database pour MySQL (version : 8.0.15)
- Connecteur de l’API C MySQL – libmysqlclient (version : 6.1.11)

| Système de fichiers        | Description                                                                                                               | Taille (Go)             | Propriétaire  | Groupe  | Stockage                    |
|--------------------|---------------------------------------------------------------------------------------------------------------------------|-----------------------|--------|--------|----------------------------|
| /usr/sap           | Système de fichiers pour l’installation de l’instance SAP BOBI, application web Tomcat par défaut et pilotes de base de données (le cas échéant) | Recommandations de dimensionnement SAP | bl1adm | sapsys | Disque Premium managé – SSD |
| /usr/sap/frsinput  | Le répertoire de montage est destiné aux fichiers partagés entre tous les hôtes BOBI qui seront utilisés en tant que répertoire de référentiel de fichiers d’entrée  | Besoins de l’entreprise         | bl1adm | sapsys | Azure NetApp Files         |
| /usr/sap/frsoutput | Le répertoire de montage est destiné aux fichiers partagés entre tous les hôtes BOBI qui seront utilisés en tant que répertoire de référentiel de fichiers de sortie | Besoins de l’entreprise         | bl1adm | sapsys | Azure NetApp Files         |

## <a name="deploy-linux-virtual-machine-via-azure-portal"></a>Déployer une machine virtuelle Linux via Portail Azure

Dans cette section, nous allons créer deux machines virtuelles avec l’image du système d’exploitation Linux pour la plateforme SAP BOBI. Les principales étapes pour créer des machines virtuelles sont les suivantes :

1. Créez un [groupe de ressources](../../../azure-resource-manager/management/manage-resource-groups-portal.md#create-resource-groups).

2. Créez un [réseau virtuel](../../../virtual-network/quick-create-portal.md#create-a-virtual-network).

   - N’utilisez pas un sous-réseau unique pour tous les services Azure dans le déploiement de la plateforme SAP BI. En fonction de l’architecture de la plateforme SAP BI, vous devez créer plusieurs sous-réseaux. Dans ce déploiement, nous allons créer trois sous-réseaux : un sous-réseau d’application, un sous-réseau de stockage du référentiel de fichiers et un sous-réseau de passerelle applicative.
   - Dans Azure, Application Gateway et Azure NetApp Files doivent toujours se trouver sur des sous-réseaux distincts. Pour plus d’informations, consultez les articles [Azure Application Gateway](../../../application-gateway/configuration-overview.md) et [Consignes pour planifier un réseau Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-network-topologies.md).

3. Créez un groupe à haute disponibilité.

   - Pour obtenir une redondance pour chaque niveau dans le cadre d’un déploiement à plusieurs instances, placez les machines virtuelles de chaque niveau dans un groupe à haute disponibilité. Veillez à séparer les groupes à haute disponibilité pour chaque niveau en fonction de votre architecture.

4. Créez la machine virtuelle 1 (**azusbosl1**).

   - Vous pouvez utiliser l’image personnalisée ou choisir une image à partir de Place de marché Azure. Reportez-vous à [Déploiement d’une machine virtuelle à partir de Place de marché Azure pour SAP](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/virtual-machines/workloads/sap/deployment-guide.md#scenario-1-deploying-a-vm-from-the-azure-marketplace-for-sap) ou [Déploiement d’une machine virtuelle avec une image personnalisée pour SAP](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/virtual-machines/workloads/sap/deployment-guide.md#scenario-2-deploying-a-vm-with-a-custom-image-for-sap) en fonction de vos besoins.

5. Créez la machine virtuelle 2 (**azusbosl2**).
6. Ajoutez un disque SSD Premium. Il sera utilisé comme répertoire d’installation SAP BOBI.

## <a name="provision-azure-netapp-files"></a>Approvisionner Azure NetApp Files

Avant de poursuivre la configuration d’Azure NetApp Files, familiarisez-vous avec la [documentation d’Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-introduction.md).

Azure NetApp Files est disponible dans plusieurs [régions Azure](https://azure.microsoft.com/global-infrastructure/services/?products=netapp). Vérifiez si la région Azure que vous avez sélectionnée est compatible avec Azure NetApp Files.

Pour vérifier la disponibilité d’Azure NetApp Files par région, consultez la page [Disponibilité d’Azure NetApp Files par région Azure](https://azure.microsoft.com/global-infrastructure/services/?products=netapp&regions=all).

Demandez l’intégration à Azure NetApp Files en accédant à [Instructions d’inscription à Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-register.md) avant de déployer Azure NetApp Files.

### <a name="deploy-azure-netapp-files-resources"></a>Déployer des ressources Azure NetApp Files

Les instructions suivantes supposent que vous avez déjà déployé votre [réseau virtuel Azure](../../../virtual-network/virtual-networks-overview.md). Les ressources Azure NetApp Files et les machines virtuelles, où les ressources Azure NetApp Files seront montées, doivent être déployées dans le même réseau virtuel Azure ou dans des réseaux virtuels Azure appairés.

1. Si vous n’avez pas encore déployé les ressources, demandez l’[intégration à Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-register.md).

2. Créez un compte NetApp dans la région Azure sélectionnée en suivant les instructions de la page [Création d’un compte NetApp](../../../azure-netapp-files/azure-netapp-files-create-netapp-account.md).

3. Configurez un pool de capacité Azure NetApp Files en suivant les instructions de la page [Configuration d’un pool de capacité Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-set-up-capacity-pool.md).

   - L’architecture de la plateforme SAP BI présentée dans cet article utilise un seul pool de capacités Azure NetApp Files au niveau de service *Premium*. Pour le serveur de référentiel de fichiers SAP BI sur Azure, nous vous recommandons d’utiliser un [niveau de service](../../../azure-netapp-files/azure-netapp-files-service-levels.md) Azure NetApp Files *Premium* ou *Ultra*.

4. Déléguez un sous-réseau à Azure NetApp Files, comme décrit dans les instructions de la page [Déléguer un sous-réseau à Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-delegate-subnet.md).

5. Déployez des volumes Azure NetApp Files en suivant les instructions de la page [Créer un volume NFS pour Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-create-volumes.md).

   Le volume ANF peut être déployé en tant que NFSv3 et NFSv4.1, car les deux protocoles sont pris en charge pour la plateforme SAP BOBI. Déployez les volumes dans leur sous-réseau Azure NetApp Files respectif. Les adresses IP des volumes Azure NetApp sont attribuées automatiquement.

Gardez à l’esprit que les ressources Azure NetApp Files et les machines virtuelles Azure doivent être dans le même réseau virtuel Azure ou dans des réseaux virtuels Azure appairés. Par exemple, azusbobi-frsinput et azusbobi-frsoutput sont les noms de volume et nfs://10.31.2.4/azusbobi-frsinput et nfs://10.31.2.4/azusbobi-frsoutput sont les chemins d’accès de fichiers pour les volumes Azure NetApp Files.

- Volume azusbobi-frsinput (nfs://10.31.2.4/azusbobi-frsinput)
- Volume azusbobi-frsoutput (nfs://10.31.2.4/azusbobi-frsoutput)

### <a name="important-considerations"></a>Considérations importantes

Lorsque vous créez votre instance Azure NetApp Files pour le serveur de référentiel de fichiers de la plateforme SAP BOBI, tenez compte des points suivants :

1. La taille de pool de capacité minimale est de 4 tébioctets (Tio).
2. La taille de volume minimale est de 100 gibioctets (Gio).
3. Azure NetApp Files et toutes les machines virtuelles où les volumes Azure NetApp Files seront montés doivent être déployés dans le même réseau virtuel Azure ou dans des [réseaux virtuels appairés](../../../virtual-network/virtual-network-peering-overview.md) dans la même région. L’accès à Azure NetApp Files via le peering de réseaux virtuels dans la même région est maintenant pris en charge. L’accès à Azure NetApp via le Peering mondial n’est pas encore pris en charge.
4. Le réseau virtuel sélectionné doit avoir un sous-réseau délégué à Azure NetApp Files.
5. Grâce à la [stratégie d’exportation](../../../azure-netapp-files/azure-netapp-files-configure-export-policy.md) d’Azure NetApp Files, vous pouvez contrôler les clients autorisés, le type d’accès (lecture et écriture, lecture seule, etc.).
6. La fonctionnalité Azure NetApp Files ne tient pas encore compte des zones. Actuellement, la fonctionnalité n’est pas déployée dans toutes les zones de disponibilité d’une région Azure. Méfiez-vous de l’impact potentiel sur les temps de latence dans certaines régions Azure.
7. Les volumes Azure NetApp Files peuvent être déployés en tant que volumes NFSv3 ou NFSv4.1. Les deux protocoles sont pris en charge pour les applications de la plateforme SAP BI.

## <a name="configure-file-systems-on-linux-servers"></a>Configurer des systèmes de fichiers sur des serveurs Linux

Les étapes de cette section utilisent les préfixes suivants :

**[A]**  : L’étape s’applique à tous les hôtes.

### <a name="format-and-mount-sap-file-system"></a>Formater et monter le système de fichiers SAP

1. **[A]** Répertorier tous les disques attachés

    ```bash
    sudo lsblk
    NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
    sda      8:0    0   30G  0 disk
    ├─sda1   8:1    0    2M  0 part
    ├─sda2   8:2    0  512M  0 part /boot/efi
    ├─sda3   8:3    0    1G  0 part /boot
    └─sda4   8:4    0 28.5G  0 part /
    sdb      8:16   0   32G  0 disk
    └─sdb1   8:17   0   32G  0 part /mnt
    sdc      8:32   0  128G  0 disk
    sr0     11:0    1  628K  0 rom  
    # Premium SSD of 128 GB is attached to Virtual Machine, whose device name is sdc
    ```

2. **[A]** Formater le périphérique en mode bloc pour /usr/sap

    ```bash
    sudo mkfs.xfs /dev/sdc
    ```

3. **[A]** Créer un répertoire de point de montage

    ```bash
    sudo mkdir -p /usr/sap
    ```

4. **[A]** Obtenir l’UUID du périphérique en mode bloc

    ```bash
    sudo blkid

    #It will display information about block device. Copy UUID of the formatted block device

    /dev/sdc: UUID="0eb5f6f8-fa77-42a6-b22d-7a9472b4dd1b" TYPE="xfs"
    ```

5. **[A]** Gérer l’entrée de montage du système de fichiers dans /etc/fstab

    ```bash
    sudo echo "UUID=0eb5f6f8-fa77-42a6-b22d-7a9472b4dd1b /usr/sap xfs defaults,nofail 0 2" >> /etc/fstab
    ```

6. **[A]** Monter le système de fichiers

    ```bash
    sudo mount -a

    sudo df -h

    Filesystem                     Size  Used Avail Use% Mounted on
    devtmpfs                       7.9G  8.0K  7.9G   1% /dev
    tmpfs                          7.9G   82M  7.8G   2% /run
    tmpfs                          7.9G     0  7.9G   0% /sys/fs/cgroup
    /dev/sda4                       29G  1.8G   27G   6% /
    tmpfs                          1.6G     0  1.6G   0% /run/user/1000
    /dev/sda3                     1014M   87M  928M   9% /boot
    /dev/sda2                      512M  1.1M  511M   1% /boot/efi
    /dev/sdb1                       32G   49M   30G   1% /mnt
    /dev/sdc                       128G   29G  100G  23% /usr/sap
    ```

### <a name="mount-azure-netapp-files-volume"></a>Monter le volume Azure NetApp Files

1. **[A]** Créer des répertoires de point de montage

   ```bash
   sudo mkdir -p /usr/sap/frsinput
   sudo mkdir -p /usr/sap/frsoutput
   ```

2. **[A]** Configurer le système d’exploitation client pour prendre en charge le montage NFSv4.1 **(applicable uniquement si vous utilisez NFSv4.1)**

   Si vous utilisez des volumes Azure NetApp Files avec le protocole NFSv4.1, exécutez la configuration suivante sur toutes les machines virtuelles, où les volumes Azure NetApp Files NFSv4.1 doivent être montés.

   **Vérifier les paramètres de domaine NFS**

   Assurez-vous que le domaine est configuré en tant que domaine Azure NetApp Files par défaut, c’est-à-dire **defaultv4iddomain.com**, et que le mappage est défini sur **nobody**.

   ```bash
   sudo cat /etc/idmapd.conf
   # Example
   [General]
   Domain = defaultv4iddomain.com
   [Mapping]
   Nobody-User = nobody
   Nobody-Group = nobody
   ```

   > [!Important]
   >
   > Veillez à définir le domaine NFS dans /etc/idmapd.conf sur la machine virtuelle pour qu’il corresponde à la configuration de domaine par défaut sur Azure NetApp Files : **defaultv4iddomain.com**. En cas d’incompatibilité entre la configuration de domaine sur le client NFS (c’est-à-dire, la machine virtuelle) et le serveur NFS, par exemple la configuration Azure NetApp, les autorisations pour les fichiers sur les volumes Azure NetApp montés sur les machines virtuelles s’affichent en tant que « nobody ».

   Vérifier `nfs4_disable_idmapping` Cette option doit avoir la valeur **Y**. Pour créer la structure de répertoire où se trouve `nfs4_disable_idmapping`, exécutez la commande de montage. Vous ne serez pas en mesure de créer manuellement le répertoire sous /sys/modules, car l’accès est réservé pour le noyau/les pilotes.

   ```bash
   # Check nfs4_disable_idmapping
   cat /sys/module/nfs/parameters/nfs4_disable_idmapping

   # If you need to set nfs4_disable_idmapping to Y
   mkdir /mnt/tmp
   mount -t nfs -o sec=sys,vers=4.1 10.31.2.4:/azusbobi-frsinput /mnt/tmp
   umount /mnt/tmp

   echo "Y" > /sys/module/nfs/parameters/nfs4_disable_idmapping

   # Make the configuration permanent
   echo "options nfs nfs4_disable_idmapping=Y" >> /etc/modprobe.d/nfs.conf
   ```

3. **[A]** Ajouter des entrées de montage

   Si vous utilisez NFSv3

   ```bash
   sudo echo "10.31.2.4:/azusbobi-frsinput /usr/sap/frsinput  nfs   rw,hard,rsize=65536,wsize=65536,vers=3" >> /etc/fstab
   sudo echo "10.31.2.4:/azusbobi-frsoutput /usr/sap/frsoutput  nfs   rw,hard,rsize=65536,wsize=65536,vers=3" >> /etc/fstab
   ```

   Si vous utilisez NFSv4.1

   ```bash
   sudo echo "10.31.2.4:/azusbobi-frsinput /usr/sap/frsinput  nfs   rw,hard,rsize=65536,wsize=65536,vers=4.1,sec=sys" >> /etc/fstab
   sudo echo "10.31.2.4:/azusbobi-frsoutput /usr/sap/frsoutput  nfs   rw,hard,rsize=65536,wsize=65536,vers=4.1,sec=sys" >> /etc/fstab
   ```

4. **[A]** Monter des volumes NFS

   ```bash
   sudo mount -a

   sudo df -h

   Filesystem                     Size  Used Avail Use% Mounted on
   devtmpfs                       7.9G  8.0K  7.9G   1% /dev
   tmpfs                          7.9G   82M  7.8G   2% /run
   tmpfs                          7.9G     0  7.9G   0% /sys/fs/cgroup
   /dev/sda4                       29G  1.8G   27G   6% /
   tmpfs                          1.6G     0  1.6G   0% /run/user/1000
   /dev/sda3                     1014M   87M  928M   9% /boot
   /dev/sda2                      512M  1.1M  511M   1% /boot/efi
   /dev/sdb1                       32G   49M   30G   1% /mnt
   /dev/sdc                       128G   29G  100G  23% /usr/sap
   10.31.2.4:/azusbobi-frsinput   101T   18G  100T   1% /usr/sap/frsinput
   10.31.2.4:/azusbobi-frsoutput  100T  512K  100T   1% /usr/sap/frsoutput
   ```

## <a name="configure-cms-database---azure-database-for-mysql"></a>Configurer la base de données CMS – Azure Database pour MySQL

Cette section fournit des informations sur l’approvisionnement d’Azure Database pour MySQL à l’aide de Portail Azure. Elle fournit également des instructions sur la création de bases de données CMS et audit pour la plateforme SAP BOBI et d’un compte d’utilisateur pour accéder à la base de données.

Les instructions s’appliquent uniquement si vous utilisez Azure DB pour MySQL. Pour les autres bases de données, consultez la documentation SAP ou la documentation propre à la base de données.

### <a name="create-an-azure-database-for-mysql"></a>Créer une base de données Azure pour MySQL

Connectez-vous à Portail Azure et suivez les étapes mentionnées dans ce [guide de démarrage rapide d’Azure Database pour MySQL](../../../mysql/quickstart-create-mysql-server-database-using-azure-portal.md). Quelques points à noter lors de l’approvisionnement d’Azure Database pour MySQL :

1. Sélectionnez la même région pour Azure Database pour MySQL que celle où les serveurs d’applications de la plateforme SAP BI s’exécutent.

2. Choisissez une version de base de donnée prise en charge basée sur la [matrice de disponibilité des produits (PAM) pour SAP BI](https://support.sap.com/pam) spécifique à votre version de SAP BOBI. Suivez les mêmes instructions de compatibilité que pour MySQL AB dans la PAM SAP.

3. Dans « Calcul + stockage », sélectionnez **Configurer le serveur**, puis sélectionnez le niveau tarifaire approprié en fonction de la taille de votre sortie.

4. **Croissance automatique du stockage** est activée par défaut. N’oubliez pas que vous pouvez uniquement effectuer un scale-up du [stockage](../../../mysql/concepts-pricing-tiers.md#storage), et non un scale-down.

5. Par défaut, la **période de rétention de sauvegarde** est de sept jours, mais vous pouvez [éventuellement la configurer](../../../mysql/howto-restore-server-portal.md#set-backup-configuration) jusqu’à 35 jours.

6. Les sauvegardes d’Azure Database pour MySQL sont localement redondantes par défaut. Par conséquent, si vous souhaitez effectuer des sauvegardes du serveur dans le stockage géoredondant, sélectionnez **Géographiquement redondant** dans **Options de redondance des sauvegardes**.

> [!NOTE]
> La modification des [options de redondance des sauvegardes](../../../mysql/concepts-backup.md#backup-redundancy-options) après la création du serveur n’est pas prise en charge.

### <a name="configure-connection-security"></a>Configurer la sécurité de la connexion

Par défaut, le serveur créé est protégé par un pare-feu et n’est pas accessible publiquement. Pour fournir l’accès au réseau virtuel sur lequel les serveurs d’applications de la plateforme SAP BI s’exécutent, procédez comme suit :  

1. Accédez à vos ressources de serveur dans le portail Azure et sélectionnez **Sécurité de la connexion** dans le menu de gauche pour votre ressource de serveur.
2. Sélectionnez **Oui** pour **Autoriser l’accès aux services Azure**.
3. Sous Règles de réseau virtuel, sélectionnez **Ajout d’un réseau virtuel existant**. Sélectionnez le réseau virtuel et le sous-réseau du serveur d’applications de la plateforme SAP BI. Vous devez également fournir un accès à Jump Box ou à d’autres serveurs à partir desquels vous pouvez connecter [MySQL Workbench](../../../mysql/connect-workbench.md) à Azure Database pour MySQL. MySQL Workbench sera utilisé pour créer la base de données CMS et audit.
4. Une fois les réseaux virtuels ajoutés, sélectionnez **Enregistrer**.

### <a name="create-cms-and-audit-database"></a>Créer une base de données CMS et audit

1. Téléchargez et installez MySQL Workbench à partir [du site de MySQL](https://dev.mysql.com/downloads/workbench/). Veillez à installer MySQL Workbench sur le serveur qui peut accéder à Azure Database pour MySQL.

2. Connectez-vous au serveur en utilisant MySQL Workbench. Suivez les instructions mentionnées dans cet [article](../../../mysql/connect-workbench.md#get-connection-information). Si le test de connexion réussit, vous obtenez le message suivant :

   ![Connexion à SQL Workbench](media/businessobjects-deployment-guide/businessobjects-sql-workbench.png)

3. Sous l’onglet Requête SQL, exécutez la requête ci-dessous pour créer un schéma pour la base de données CMS et audit.

   ```sql
   # Here cmsbl1 is the database name of CMS database. You can provide the name you want for CMS database.
   CREATE SCHEMA `cmsbl1` DEFAULT CHARACTER SET utf8;

   # auditbl1 is the database name of Audit database. You can provide the name you want for CMS database.
   CREATE SCHEMA `auditbl1` DEFAULT CHARACTER SET utf8;

4. Create user account to connect to schema

   ```sql
   # Create a user that can connect from any host, use the '%' wildcard as a host part
   CREATE USER 'cmsadmin'@'%' IDENTIFIED BY 'password';
   CREATE USER 'auditadmin'@'%' IDENTIFIED BY 'password';

   # Grant all privileges to a user account over a specific database:
   GRANT ALL PRIVILEGES ON cmsbl1.* TO 'cmsadmin'@'%' WITH GRANT OPTION;
   GRANT ALL PRIVILEGES ON auditbl1.* TO 'auditadmin'@'%' WITH GRANT OPTION;

   # Following any updates to the user privileges, be sure to save the changes by issuing the FLUSH PRIVILEGES
   FLUSH PRIVILEGES;
   ```

5. Vérifiez les privilèges et les rôles du compte d’utilisateur MySQL.

   ```sql
   USE sys;
   SHOW GRANTS for 'cmsadmin'@'%';
   +------------------------------------------------------------------------+
   | Grants for cmsadmin@%                                                  |
   +------------------------------------------------------------------------+
   | GRANT USAGE ON *.* TO `cmsadmin`@`%`                                   |
   | GRANT ALL PRIVILEGES ON `cmsbl1`.* TO `cmsadmin`@`%` WITH GRANT OPTION |
   +------------------------------------------------------------------------+

   USE sys;
   SHOW GRANTS FOR 'auditadmin'@'%';
   +----------------------------------------------------------------------------+
   | Grants for auditadmin@%                                                    |
   +----------------------------------------------------------------------------+
   | GRANT USAGE ON *.* TO `auditadmin`@`%`                                     |
   | GRANT ALL PRIVILEGES ON `auditbl1`.* TO `auditadmin`@`%` WITH GRANT OPTION |
   +----------------------------------------------------------------------------+
   ```

### <a name="install-mysql-c-api-connector-libmysqlclient-on-linux-server"></a>Installer le connecteur de l’API C MySQL (libmysqlclient) sur le serveur Linux

Pour que le serveur d’applications SAP BOBI accède à la base de données, il requiert des clients/pilotes de base de données. Le connecteur de l’API C MySQL pour Linux doit être utilisé pour accéder aux bases de données CMS et audit. La connexion ODBC à la base de données CMS n’est pas prise en charge. Cette section fournit des instructions sur la configuration du connecteur de l’API C MySQL sur Linux.

1. Reportez-vous à l’article [Pilotes MySQL et outils de gestion compatibles avec Azure Database pour MySQL](../../../mysql/concepts-compatibility.md), qui décrit les pilotes compatibles avec Azure Database pour MySQL. Vérifiez la présence du pilote **Connecteur MySQL/C (libmysqlclient)** dans l’article.

2. Reportez-vous à ce [lien](https://downloads.mysql.com/archives/c-c/) pour télécharger des pilotes.

3. Sélectionnez le système d’exploitation et téléchargez le package rpm de composant partagé du connecteur MySQL. Dans cet exemple, la version mysql-connector-c-shared-6.1.11 du connecteur est utilisée.

4. Installez le connecteur dans toutes les instances d’application SAP BOBI.

   ```bash
   # Install rpm package
   SLES: sudo zypper install <package>.rpm
   RHEL: sudo yum install <package>.rpm
   ```

5. Vérifiez le chemin d’accès de libmysqlclient.so.

   ```bash
   # Find the location of libmysqlclient.so file
   whereis libmysqlclient

   # sample output
   libmysqlclient: /usr/lib64/libmysqlclient.so
   ```

6. Définissez LD_LIBRARY_PATH pour qu’il pointe vers le répertoire `/usr/lib64` pour le compte d’utilisateur qui sera utilisé pour l’installation.

   ```bash
   # This configuration is for bash shell. If you are using any other shell for sidadm, kindly set environment variable accordingly.
   vi /home/bl1adm/.bashrc

   export LD_LIBRARY_PATH=/usr/lib64
   ```

## <a name="server-preparation"></a>Préparation du serveur

Les étapes de cette section utilisent les préfixes suivants :

**[A]**  : L’étape s’applique à tous les hôtes.

1. **[A]** En fonction de la saveur de Linux (SLES ou RHEL), vous devez définir les paramètres du noyau et installer les bibliothèques requises. Reportez-vous à la section **System requirements** (Configuration système requise) dans [Business Intelligence Platform Installation Guide for Unix](https://help.sap.com/viewer/65018c09dbe04052b082e6fc4ab60030/4.3/en-US) (Guide d’installation de la plateforme Business Intelligence pour UNIX).

2. **[A]** Vérifiez que le fuseau horaire de votre ordinateur est correctement défini. Reportez-vous à la section [Additional Unix and Linux requirements](https://help.sap.com/viewer/65018c09dbe04052b082e6fc4ab60030/4.3/en-US/46b143336e041014910aba7db0e91070.html) (Conditions requises supplémentaires pour UNIX et Linux) dans le guide d’installation.

3. **[A]** Créez un compte d’utilisateur (**bl1** adm) et un groupe (sapsys) sous lesquels les processus d’arrière-plan du logiciel peuvent s’exécuter. Utilisez ce compte pour exécuter l’installation et faire fonctionner le logiciel. Le compte ne nécessite pas de privilèges racine.

4. **[A]** Configurez l’environnement du compte d’utilisateur (**bl1** adm) pour utiliser des paramètres régionaux UTF-8 pris en charge et vérifiez que votre logiciel de console prend en charge les jeux de caractères UTF-8. Pour vous assurer que votre système d’exploitation utilise les paramètres régionaux corrects, définissez les variables d’environnement LC_ALL et LANG sur les paramètres régionaux de votre choix dans votre environnement utilisateur (**bl1** adm).

   ```bash
   # This configuration is for bash shell. If you are using any other shell for sidadm, kindly set environment variable accordingly.
   vi /home/bl1adm/.bashrc

   export LANG=en_US.utf8
   export LC_ALL=en_US.utf8
   ```

5. **[A]** Configurez le compte d’utilisateur (**bl1** adm).

   ```bash
   # Set ulimit for bl1adm to unlimited
   root@azusbosl1:~> ulimit -f unlimited bl1adm
   root@azusbosl1:~> ulimit -u unlimited bl1adm

   root@azusbosl1:~> su - bl1adm
   bl1adm@azusbosl1:~> ulimit -a

   core file size          (blocks, -c) unlimited
   data seg size           (kbytes, -d) unlimited
   scheduling priority             (-e) 0
   file size               (blocks, -f) unlimited
   pending signals                 (-i) 63936
   max locked memory       (kbytes, -l) 64
   max memory size         (kbytes, -m) unlimited
   open files                      (-n) 1024
   pipe size            (512 bytes, -p) 8
   POSIX message queues     (bytes, -q) 819200
   real-time priority              (-r) 0
   stack size              (kbytes, -s) 8192
   cpu time               (seconds, -t) unlimited
   max user processes              (-u) unlimited
   virtual memory          (kbytes, -v) unlimited
   file locks                      (-x) unlimited
   ```

6. Téléchargez et extrayez le média de la plateforme décisionnelle SAP BusinessObjects à partir de SAP Service Marketplace.

## <a name="installation"></a>Installation

Vérifiez les paramètres régionaux pour le compte d’utilisateur **bl1** adm sur le serveur.

```bash
bl1adm@azusbosl1:~> locale
LANG=en_US.utf8
LC_ALL=en_US.utf8
```

Accédez au média de la plateforme décisionnelle SAP BusinessObjects et exécutez la commande ci-dessous avec l’utilisateur **bl1** adm :

```bash
./setup.sh -InstallDir /usr/sap/BL1
```

Suivez le guide d’installation de la [plateforme SAP BOBI](https://help.sap.com/viewer/product/SAP_BUSINESSOBJECTS_BUSINESS_INTELLIGENCE_PLATFORM) pour UNIX, en fonction de votre version. Quelques points à noter lors de l’installation de la plateforme SAP BOBI.

- Sur l’écran **Configure Product Registration** (Configurer l’enregistrement du produit), vous pouvez soit utiliser une clé de licence temporaire pour les solutions SAP BusinessObjects à partir de la note SAP [1288121](https://launchpad.support.sap.com/#/notes/1288121), soit générer une clé de licence dans SAP Service Marketplace.

- Sur l’écran **Select Install Type** (Sélectionner le type d’installation) , sélectionnez **Full** (Installation complète) sur le premier serveur (azusbosl1). Pour l’autre serveur (azusbosl2), sélectionnez **Custom / Expand** (Personnaliser/développer) qui développera la configuration BOBI actuelle.

- Sur l’écran **Select Default or Existing Database** (Sélectionner un base de données existante ou par défaut), sélectionnez **Configure an existing database** (Configurer une base de données existante), ce qui vous invite à sélectionner la base de données CMS et audit. Sélectionnez **MySQL** pour le type de base de données CMS et le type de base de données d’audit.

  Vous pouvez également sélectionner Aucune base de données d’audit si vous ne souhaitez pas configurer l’audit pendant l’installation.

- Sélectionnez les options appropriées sur l’écran **Select Java Web Application Server** (Sélectionner le serveur d’applications web Java) en fonction de votre architecture SAP BOBI. Dans cet exemple, nous avons sélectionné l’option 1, qui installe le serveur Tomcat sur la même plateforme SAP BOBI.

- Entrez les informations de la base de données CMS dans **Configure CMS Repository Database - MySQL** (Configurer la base de données du référentiel CMS – MySQL). Exemple d’entrée pour les informations de base de données CMS pour l’installation de Linux. Azure Database pour MySQL est utilisé sur le port 3306 par défaut.
  
  ![Déploiement de SAP BOBI sur Linux – Base de données CMS](media/businessobjects-deployment-guide/businessobjects-deployment-linux-sql-cms.png)

- (Facultatif) Entrez les informations de la base de données d’audit dans **Configure Audit Repository Database - MySQL** (Configurer la base de données du référentiel d’audit – MySQL). Exemple d’entrée pour les informations de base de données d’audit pour l’installation de Linux.

  ![Déploiement de SAP BOBI sur Linux – Base de données d’audit](media/businessobjects-deployment-guide/businessobjects-deployment-linux-sql-audit.png)

- Suivez les instructions et saisissez les entrées requises pour terminer l’installation.

Pour un déploiement à plusieurs instances, exécutez la configuration de l’installation sur le deuxième hôte (azusbosl2). Sur l’écran **Select Install Type** (Sélectionner le type d’installation), sélectionnez **Custom / Expand** (Personnaliser/développer) qui développera la configuration BOBI actuelle.

Dans l’offre Azure Database pour MySQL, une passerelle est utilisée pour rediriger les connexions vers les instances de serveur. Une fois la connexion établie, le client de MySQL affiche la version de MySQL définie dans la passerelle, et non la version en cours d’exécution sur votre instance de serveur MySQL. Pour déterminer la version de votre instance de serveur MySQL, utilisez la commande `SELECT VERSION();` à l’invite de MySQL. Ainsi, dans la console de gestion centralisée (CMC), vous trouverez une autre version de la base de données qui est essentiellement la version définie sur la passerelle. Pour plus d’informations, consultez [Versions prises en charge du serveur de base de données Azure pour MySQL](../../../mysql/concepts-supported-versions.md).

![Déploiement de SAP BOBI sur Linux – Paramètres de CMC](media/businessobjects-deployment-guide/businessobjects-deployment-linux-sql-cmc.png)

```sql
# Run direct query to the database using MySQL Workbench

select version();

+-----------+
| version() |
+-----------+
| 8.0.15    |
+-----------+
```

## <a name="post-installation"></a>Après l'installation

### <a name="tomcat-clustering---session-replication"></a>Clustering Tomcat – réplication de session

Tomcat prend en charge le clustering d’au moins deux serveurs d’applications pour la réplication et le basculement de session. Les sessions de la plateforme SAP BOBI sont sérialisées, mais une session utilisateur peut basculer en toute transparence vers une autre instance de Tomcat, même en cas de défaillance d’un serveur d’applications.

Par exemple, si un utilisateur est connecté à un serveur web qui échoue alors qu’il navigue dans l’arborescence des dossiers d’une application SAP BI. Avec un cluster correctement configuré, l’utilisateur peut continuer à naviguer dans l’arborescence des dossiers sans être redirigé vers la page de connexion.

Dans la note SAP [2808640](https://launchpad.support.sap.com/#/notes/2808640), les étapes de configuration du clustering Tomcat sont fournies pour l’utilisation de la multidiffusion. Toutefois, dans Azure, la multidiffusion n’est pas prise en charge. Pour que le cluster Tomcat fonctionne dans Azure, vous devez utiliser [StaticMembershipInterceptor](https://tomcat.apache.org/tomcat-8.0-doc/config/cluster-interceptor.html#Static_Membership) (note SAP [2764907](https://launchpad.support.sap.com/#/notes/2764907)). Consultez [Tomcat Clustering using Static Membership for SAP BusinessObjects BI Platform](https://blogs.sap.com/2020/09/04/sap-on-azure-tomcat-clustering-using-static-membership-for-sap-businessobjects-bi-platform/) (Clustering Tomcat à l’aide de l’appartenance statique pour la plateforme décisionnelle SAP BusinessObjects) sur le blog SAP pour configurer le cluster Tomcat dans Azure.

### <a name="load-balancing-web-tier-of-sap-bi-platform"></a>Équilibrage de la charge du niveau web de la plateforme SAP BI

Dans le cadre d’un déploiement multi-instance de SAP BOBI, les serveurs d’applications web Java (niveau web) sont exécutés sur au moins deux hôtes. Pour répartir uniformément la charge des utilisateurs sur les serveurs web, vous pouvez utiliser un équilibreur de charge entre les utilisateurs finaux et les serveurs web. Dans Azure, vous pouvez utiliser Azure Load Balancer ou Azure Application Gateway pour gérer le trafic vers vos serveurs d’applications web. Les détails de chaque offre sont expliqués dans la section suivante.

#### <a name="azure-load-balancer-network-based-load-balancer"></a>Azure Load Balancer (équilibreur de charge basé sur le réseau)

[Azure Load Balancer](../../../load-balancer/load-balancer-overview.md) est un équilibreur de charge de couche 4 (TCP, UDP) à haute performance et à faible latence qui distribue le trafic entre les machines virtuelles saines. Une sonde d’intégrité d’équilibreur de charge surveille un port donné sur chaque machine virtuelle et ne distribue le trafic qu’aux machines virtuelles opérationnelles. Vous pouvez choisir un équilibreur de charge public ou un équilibreur de charge interne, selon que vous souhaitez que la plateforme SAP BI soit accessible ou non à partir d’Internet. Azure Load Balancer est redondant interzone, garantissant une haute disponibilité de la fonctionnalité Zones de disponibilité.

Reportez-vous à la section Équilibreur de charge interne de la figure ci-dessous, où le serveur d’applications web s’exécute sur le port 8080, port HTTP par défaut de Tomcat, qui sera analysé par la sonde d’intégrité. Par conséquent, toute requête entrante émanant des utilisateurs finaux sera redirigée vers les serveurs d’applications web (azusbosl1 ou azusbosl2) dans le pool principal. L’équilibreur de charge ne prend pas en charge la terminaison TLS/SSL (également appelé déchargement TLS/SSL). Si vous utilisez un équilibreur de charge Azure pour distribuer le trafic entre les serveurs web, nous vous recommandons d’utiliser Standard Load Balancer.

> [!NOTE]
> Lorsque des machines virtuelles sans adresse IP publique sont placées dans le pool principal d’Azure Standard Load Balancer interne (aucune adresse IP publique), il n’y a pas de connectivité Internet sortante, sauf si une configuration supplémentaire est effectuée pour autoriser le routage vers des points de terminaison publics. Pour savoir plus en détails comment bénéficier d’une connectivité sortante, voir [Connectivité des points de terminaison publics pour les machines virtuelles avec Azure Standard Load Balancer dans les scénarios de haute disponibilité SAP](high-availability-guide-standard-load-balancer-outbound-connections.md).

![Azure Load Balancer pour équilibrer le trafic entre les serveurs web](media/businessobjects-deployment-guide/businessobjects-deployment-load-balancer.png)

#### <a name="azure-application-gateway-web-application-load-balancer"></a>Passerelle applicative Azure (équilibreur de charge d’application web)

[Azure Application Gateway (AGW)](../../../application-gateway/overview.md) fournit le contrôleur de livraison d’applications (ADC) en tant que service, qui est utilisé pour aider l’application à diriger le trafic des utilisateurs vers un ou plusieurs serveurs d’applications web. Il offre différentes capacités d’équilibrage de charge de couche 7, notamment le déchargement TLS/SSL, le pare-feu d’application web (WAF), l’affinité de session basée sur les cookies et d’autres pour vos applications.

Dans la plateforme SAP BI, la passerelle applicative dirige le trafic web des applications vers les ressources spécifiées dans un pool principal : azusbosl1 ou azusbos2. Vous attribuez un écouteur au port, créez des règles et ajoutez des ressources à un pool principal. Dans la figure ci-dessous, la passerelle applicative avec adresse IP frontale privée (10.31.3.20) agit comme point d’entrée pour les utilisateurs, gère les connexions TLS/SSL (HTTPS-TCP/443) entrantes, déchiffre le TLS/SSL et transmet la requête non chiffrée (HTTP-TCP/8080) aux serveurs du pool principal. Avec la fonctionnalité de terminaison TLS/SSL intégrée, nous devons simplement gérer un certificat TLS/SSL sur la passerelle applicative, ce qui simplifie les opérations.

![Application Gateway pour équilibrer le trafic entre les serveurs web](media/businessobjects-deployment-guide/businessobjects-deployment-application-gateway.png)

Afin de configurer Application Gateway pour le serveur web SAP BOBI, vous pouvez vous reporter à [Load Balancing SAP BOBI Web Servers using Azure Application Gateway](https://blogs.sap.com/2020/09/17/sap-on-azure-load-balancing-web-application-servers-for-sap-bobi-using-azure-application-gateway/) (Équilibrage de la charge des serveurs web SAP BOBI à l’aide d’Azure Application Gateway) sur le blog SAP.

> [!NOTE]
> Nous vous recommandons d’utiliser Azure Application Gateway pour équilibrer la charge du trafic vers le serveur web, car il offre des fonctionnalités similaires comme le déchargement SSL, la gestion centralisée du SSL pour réduire les frais de chiffrement et de déchiffrement sur le serveur, l’algorithme de tourniquet (round robin) pour distribuer le trafic, des capacités de pare-feu d’applications web (WAF), la haute disponibilité, entre autres.

### <a name="sap-businessobjects-bi-platform---back-up-and-restore"></a>Plateforme décisionnelle SAP BusinessObjects – Sauvegarde et restauration

La sauvegarde et la restauration sont un processus qui consiste à créer des copies périodiques des données et des applications vers un emplacement distinct. Il est donc possible de restaurer ou de revenir à l’état antérieur si les données ou les applications d’origine sont perdues ou endommagées. Il s’agit également d’un composant essentiel de toute stratégie de récupération d’urgence.

Pour développer une stratégie complète de sauvegarde et de restauration pour la plateforme SAP BOBI, identifiez les composants qui entraînent un temps d’arrêt du système ou une interruption de l’application. Dans la plateforme SAP BOBI, la sauvegarde des composants suivants est essentielle pour protéger l’application :

- Répertoire d’installation de SAP BOBI (Disques Premium managés)
- Serveur de référentiel de fichiers (Azure NetApp Files ou Fichiers Azure Premium)
- Base de données CMS (Azure Database pour MySQL ou Database sur une machine virtuelle Azure)

La section suivante décrit comment implémenter une stratégie de sauvegarde et de restauration pour chaque composant sur la plateforme SAP BOBI.

#### <a name="backup--restore-for-sap-bobi-installation-directory"></a>Sauvegarder et restaurer le répertoire d’installation de SAP BOBI

Dans Azure, la méthode la plus simple pour sauvegarder des serveurs d’applications et tous les disques attachés consiste à utiliser le service [Sauvegarde Azure](../../../backup/backup-overview.md). Il propose des sauvegardes indépendantes et isolées pour éviter une destruction involontaire des données de vos machines virtuelles. Les sauvegardes sont stockées dans un coffre Recovery Services avec gestion intégrée des points de récupération. La configuration et la mise à l’échelle sont simples, les sauvegardes sont optimisées et peuvent être facilement restaurées le cas échéant.

Dans le cadre du processus de sauvegarde, un instantané est créé et les données sont transférées vers le coffre Recovery Services sans impact sur les charges de travail de production. L’instantané offre différents niveaux de cohérence, comme décrit dans l’article [Cohérence des instantanés](../../../backup/backup-azure-vms-introduction.md#snapshot-consistency). Vous pouvez également choisir de sauvegarder un sous-ensemble des disques de données d’une machine virtuelle en utilisant la fonctionnalité de sauvegarde et restauration sélectives de disques. Pour plus d’informations, consultez le document [Sauvegarde de machine virtuelle Azure](../../../backup/backup-azure-vms-introduction.md) et [Forum aux questions – Sauvegarde de machines virtuelles Azure](../../../backup/backup-azure-vm-backup-faq.md).

#### <a name="backup--restore-for-file-repository-server"></a>Sauvegarder et restaurer le serveur de référentiel de fichiers

Pour **Azure NetApp Files**, vous pouvez créer des instantanés à la demande et planifier un instantané automatique en utilisant des stratégies d’instantané. Les instantanés fournissent une copie à un moment donné de votre volume ANF. Pour plus d’informations, consultez [Gérer les instantanés avec Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-manage-snapshots.md).

La sauvegarde **Azure Files** est intégrée au service [Sauvegarde Azure](../../../backup/backup-overview.md) natif, qui centralise la fonction de sauvegarde et de restauration ainsi que la sauvegarde de machines virtuelles et simplifie le travail d’exploitation. Pour plus d’informations, consultez [Sauvegarde des partages de fichiers Azure](../../../backup/azure-file-share-backup-overview.md) et [Forum aux questions – Sauvegarde des fichiers Azure](../../../backup/backup-azure-files-faq.md).

#### <a name="backup--restore-for-cms-database"></a>Sauvegarder et restaurer la base de données CMS

Azure Database pour MySQL est une offre DBaaS dans Azure qui crée automatiquement des sauvegardes de serveur et les conserve dans un stockage géoredondant ou localement redondant configuré par l’utilisateur. Azure Database pour MySQL effectue des sauvegardes des fichiers de données et du journal des transactions. Selon la taille de stockage maximale prise en charge, elle effectue des sauvegardes complètes et différentielles (serveurs de stockage de 4 To) ou des sauvegardes d’instantané (serveurs de stockage jusqu’à 16 To). Celles-ci vous permettent de restaurer un serveur à n’importe quel point dans le temps pendant la période de conservation des sauvegardes configurée. Par défaut, la période de conservation des sauvegardes est de sept jours, mais vous pouvez [éventuellement la configurer](../../../mysql/howto-restore-server-portal.md#set-backup-configuration) jusqu’à trois jours. Toutes les sauvegardes sont chiffrées à l’aide du chiffrement AES de 256 bits.

Ces fichiers de sauvegarde ne sont pas exposés aux utilisateurs et ne peuvent pas être exportés. Ces sauvegardes sont utilisables uniquement pour les opérations de restauration dans Azure Database pour MySQL. Vous pouvez utiliser [mysqldump](../../../mysql/concepts-migrate-dump-restore.md) pour copier une base de données. Pour plus d’informations, consultez [Sauvegarde et restauration dans Azure Database pour MySQL](../../../mysql/concepts-backup.md).

Pour une base de données installée sur Machines Virtuelles, vous pouvez utiliser les outils de sauvegarde standard ou [Sauvegarde Azure](../../../backup/sap-hana-db-about.md) pour la base de données HANA. De même, si les services et outils Azure ne répondent pas à vos besoins, vous pouvez utiliser d’autres outils de sauvegarde ou un script pour créer une sauvegarde des disques.

## <a name="sap-businessobjects-bi-platform-reliability"></a>Fiabilité de la plateforme décisionnelle SAP BusinessObjects

La plateforme décisionnelle SAP BusinessObjects comprend différents niveaux, qui sont optimisés pour des tâches et des opérations spécifiques. Lorsque le composant d’un niveau quelconque devient indisponible, l’application SAP BOBI devient inaccessible ou certaines fonctionnalités de l’application ne fonctionnent pas. Vous devez donc vous assurer que chaque niveau est conçu pour être fiable afin de garantir le bon fonctionnement de l’application sans aucune interruption de l’activité.

Cette section se concentre sur les options suivantes pour la plateforme SAP BOBI :

- **Haute disponibilité :** Une plateforme à haute disponibilité dispose d’au moins deux copies de chaque élément dans la région Azure pour que l’application reste opérationnelle si l’un des serveurs n’est plus disponible.
- **Reprise d’activité après sinistre :** Il s’agit du processus de restauration des fonctionnalités de votre application en cas de perte irrécupérable, par exemple si l’intégralité de la région Azur devenait indisponible en raison d’une catastrophe naturelle.

L’implémentation de cette solution varie selon la nature de la configuration du système dans Azure. Le client doit donc adapter la solution Haute disponibilité et récupération d’urgence en fonction de ses besoins professionnels.

### <a name="high-availability"></a>Haute disponibilité

Haute disponibilité désigne un ensemble de technologies qui peuvent réduire les interruptions du service informatique en assurant la continuité de l’activité des services ou des applications par le biais de composants redondants, tolérants aux pannes ou protégés par basculement au sein du même centre de données. Dans notre cas, les centres de données se trouvent dans une région Azure. L’article [Scénarios et architecture de haute disponibilité pour SAP](sap-high-availability-architecture-scenarios.md) donne un premier aperçu des différentes techniques de haute disponibilité et des recommandations proposées sur Azure pour les applications SAP, qui compléteront les instructions de cette section.

En fonction du résultat de dimensionnement de la plateforme SAP BOBI, vous devez concevoir le paysage et déterminer la distribution des composants décisionnels entre les machines virtuelles et les sous-réseaux Azure. Le niveau de redondance dans l’architecture distribuée dépend de l’objectif de délai de récupération (RTO) et de l’objectif de point de récupération (RPO) requis par l’entreprise. La plateforme SAP BOBI comprend différents niveaux, et les composants de chaque niveau doivent être conçus de manière à assurer la redondance. Ainsi, en cas de défaillance d’un composant, votre application SAP BOBI sera peu ou pas interrompue. Par exemple,

- Serveurs d’applications redondants, tels que les serveurs d’applications décisionnelles et le serveur web
- Composants uniques, tels que la base de données CMS, le serveur de référentiel de fichiers ou l’équilibreur de charge

La section suivante décrit la façon d’atteindre une haute disponibilité sur chaque composant de la plateforme SAP BOBI.

#### <a name="high-availability-for-application-servers"></a>Haute disponibilité pour les serveurs d’applications

Pour les serveurs d’applications web et décisionnelles, qu’ils soient installés séparément ou ensemble, il n’est pas nécessaire de disposer d’une solution spécifique de haute disponibilité. Vous pouvez obtenir une haute disponibilité par redondance, c’est-à-dire en configurant plusieurs instances de serveurs d’applications web et décisionnelles dans différentes machines virtuelles Azure.

Pour réduire l’impact des temps d’arrêt dus à un ou plusieurs événements, il est recommandé de suivre la pratique de haute disponibilité ci-dessous pour les serveurs d’applications qui s’exécutent sur plusieurs machines virtuelles.

- Utilisez Zones de disponibilité pour protéger les centres de données contre les défaillances.
- Configurez plusieurs machines virtuelles dans un groupe à haute disponibilité pour assurer la redondance.
- Utilisez Disques managés pour les machines virtuelles dans un groupe à haute disponibilité.
- Configurez chaque couche Application dans des groupes à haute disponibilité distincts.

Pour plus d’informations, consultez [Gestion de la disponibilité des machines virtuelles Linux](../../manage-availability.md).

#### <a name="high-availability-for-cms-database"></a>Haute disponibilité pour la base de données CMS

Si vous utilisez le service Azure Database en tant que service (DBaaS) pour la base de données CMS, l’infrastructure de haute disponibilité est fournie par défaut. Il vous suffit de sélectionner la région et les capacités de haute disponibilité, de redondance et de résilience inhérentes au service, sans avoir à configurer de composants supplémentaires. Pour plus d’informations sur le contrat SLA de l’offre DBaaS prise en charge sur Azure, consultez [Haute disponibilité dans Azure Database pour MySQL](../../../mysql/concepts-high-availability.md) et [Haute disponibilité pour Azure SQL Database](../../../azure-sql/database/high-availability-sla.md).

Pour les autres déploiements SGBD pour la base de données CMS, reportez-vous à l’article [Guides de déploiement SGBD pour charge de travail SAP](dbms_guide_general.md), qui fournit des informations sur différents déploiements SGBD et leur approche pour atteindre une haute disponibilité.

#### <a name="high-availability-for-file-repository-server"></a>Haute disponibilité pour le serveur de référentiel de fichiers

Le serveur de référentiel de fichiers (FRS) désigne les répertoires de disque où sont stockés du contenu tels que les rapports, les univers et les connexions. Il est partagé par tous les serveurs d’applications de ce système. Vous devez donc vous assurer qu’il est hautement disponible.

Sur Azure, vous pouvez choisir [Fichiers Azure Premium](../../../storage/files/storage-files-introduction.md) ou [Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-introduction.md) comme partage de fichiers ; les deux sont conçus pour être hautement disponible et hautement durable par nature. Pour plus d’informations, consultez la section [Redondance](../../../storage/files/storage-files-planning.md#redundancy) pour Azure Files.

> [!NOTE]
> Le protocole SMB pour Azure Files est en disponibilité générale, tandis que la prise en charge du protocole NFS pour Azure Files est actuellement en préversion. Pour plus d’informations, consultez [La prise en charge de NFS 4.1 pour Azure Files est désormais disponible en préversion](https://azure.microsoft.com/en-us/blog/nfs-41-support-for-azure-files-is-now-in-preview/).

Ce service de partage de fichiers n’étant pas disponible dans toutes les régions, veillez à consulter le site [Disponibilité des produits par région](https://azure.microsoft.com/en-us/global-infrastructure/services/) pour bénéficier d’informations à jour. Si le service n’est pas disponible dans votre région, vous pouvez créer un serveur NFS à partir duquel vous pourrez partager le système de fichiers avec l’application SAP BOBI. Toutefois, vous devrez également tenir compte de sa haute disponibilité.

#### <a name="high-availability-for-load-balancer"></a>Haute disponibilité pour l’équilibreur de charge

Pour distribuer le trafic sur le serveur web, vous pouvez utiliser Azure Load Balancer ou Azure Application Gateway. La redondance de l’un ou l’autre des équilibreurs de charge peut être obtenue en fonction du niveau tarifaire que vous choisissez pour le déploiement.

- Pour Azure Load Balancer, la redondance peut être obtenue en configurant le serveur frontal de Standard Load Balancer comme redondant interzone. Pour en savoir plus, consultez [Standard Load Balancer et Zones de disponibilité](../../../load-balancer/load-balancer-standard-availability-zones.md).
- Pour Application Gateway, la haute disponibilité peut être obtenue en fonction du type de niveau sélectionné lors du déploiement.
  - Le SKU v1 prend en charge les scénarios de haute disponibilité lorsque vous avez déployé deux instances ou plus. Azure distribue ces instances entre les domaines de mise à jour et d’erreur pour garantir qu'elles n’échouent pas toutes en même temps. Ainsi, grâce à ce SKU, la redondance peut être obtenue au sein de la zone.
  - Le SKU v2 garantit automatiquement que les nouvelles instances sont réparties sur les domaines d’erreur et les domaines de mise à jour. Si vous choisissez une redondance de zone, les instances les plus récentes sont également réparties sur les zones de disponibilité pour offrir une résilience zonale en cas d’échec. Pour plus de détails, consultez [Application Gateway v2 avec mise à l’échelle automatique et redondance interzone](../../../application-gateway/application-gateway-autoscaling-zone-redundant.md).

#### <a name="reference-high-availability-architecture-for-sap-businessobjects-bi-platform"></a>Architecture haute disponibilité de référence pour la plateforme décisionnelle SAP BusinessObjects

L’architecture de référence ci-dessous décrit la configuration de la plateforme SAP BOBI avec un groupe à haute disponibilité, qui fournit la redondance et la disponibilité des machines virtuelles dans la zone. L’architecture présente l’utilisation de différents services Azure, comme Azure Application Gateway, Azure NetApp Files et Azure Database pour MySQL pour la plateforme SAP BOBI, qui offre une redondance intégrée, ce qui réduit la complexité de la gestion de différentes solutions de haute disponibilité.

Dans la figure ci-dessous, le trafic entrant (HTTPS-TCP/443) est équilibré en charge à l’aide du SKU Azure Application Gateway v1, qui est hautement disponible lorsqu’il est déployé sur deux instances ou plus. Plusieurs instances de serveur web, de serveurs d’administration et de serveurs de traitement sont déployées sur des machines virtuelles distinctes pour assurer la redondance, et chaque niveau est déployé dans des groupes à haute disponibilité distincts. Azure NetApp Files disposant d’une redondance intégrée dans le centre de données, vos volumes ANF pour le serveur de référentiel de fichiers sont hautement disponibles. La base de données CMS est approvisionnée sur Azure Database pour MySQL (DBaaS), qui a une haute disponibilité inhérente. Pour plus d’informations, consultez le guide [Haute disponibilité dans Azure Database pour MySQL](../../../mysql/concepts-high-availability.md).

![Redondance de la plateforme décisionnelle SAP BusinessObjects grâce à des groupes à haute disponibilité](media/businessobjects-deployment-guide/businessobjects-deployment-high-availability.png)

L’architecture ci-dessus fournit des informations sur la façon dont le déploiement de SAP BOBI sur Azure peut être effectué. Toutefois, elle ne couvre pas toutes les options de configuration possibles pour la plateforme SAP BOBI sur Azure. Le client peut adapter son déploiement en fonction de ses besoins professionnels, en choisissant différents produits ou services pour différents composants tels que l’équilibreur de charge, le serveur de référentiel de fichiers et le système de gestion de base de données (SGBD).

Dans plusieurs régions Azure, la fonctionnalité Zones de disponibilité est proposée, ce qui signifie que ces régions disposent d’une source d’alimentation électrique, d’un système de refroidissement et d’un réseau indépendants. Cela permet au client de déployer l’application dans deux ou trois zones de disponibilité. Les clients qui souhaitent obtenir une haute disponibilité dans les régions Azure peuvent déployer la plateforme SAP BOBI dans les zones de disponibilité, en veillant à ce que chaque composant de l’application soit redondant interzone.

### <a name="disaster-recovery"></a>Récupération d'urgence

Les instructions de cette section expliquent la stratégie visant à fournir une protection de récupération d’urgence pour la plateforme SAP BOBI. Elles complètent le document [Récupération d’urgence pour SAP](../../../site-recovery/site-recovery-sap.md), qui représente les principales ressources pour l’approche globale de la récupération d’urgence SAP.

#### <a name="reference-disaster-recovery-architecture-for-sap-businessobjects-bi-platform"></a>Architecture de récupération d’urgence de référence pour la plateforme décisionnelle SAP BusinessObjects

Cette architecture de référence exécute un déploiement multi-instance de la plateforme SAP BOBI avec des serveurs d’applications redondants. Pour la récupération d’urgence, vous devez basculer tous les niveaux vers une région secondaire. Chaque couche utilise une stratégie différente pour protéger la récupération d’urgence.

![Récupération d’urgence de la plateforme décisionnelle SAP BusinessObjects](media/businessobjects-deployment-guide/businessobjects-deployment-disaster-recovery.png)

#### <a name="load-balancer"></a>Équilibrage de charge

L’équilibreur de charge est utilisé pour distribuer le trafic sur les serveurs d’applications web de la plateforme SAP BOBI. Pour obtenir la récupération d’urgence pour Azure Application Gateway, implémentez une configuration parallèle de la passerelle applicative sur la région secondaire.

#### <a name="virtual-machines-running-web-and-bi-application-servers"></a>Machines virtuelles exécutant des serveurs d’applications web et décisionnelles

Le service Azure Site Recovery peut être utilisé pour répliquer des machines virtuelles exécutant des serveurs d’applications web et décisionnelles sur la région secondaire. Il réplique les serveurs sur la région secondaire afin que, en cas de catastrophe ou de panne, vous puissiez facilement basculer sur votre environnement répliqué et continuer à travailler.

#### <a name="file-repository-servers"></a>Serveurs de référentiel de fichiers

- **Azure NetApp Files** fournit des volumes NFS et SMB, de sorte que tout outil de copie basé sur des fichiers peut être utilisé pour répliquer les données entre des régions Azure. Pour plus d’informations sur la façon de copier un volume ANF dans une autre région, consultez [Questions fréquentes (FAQ) sur Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-faqs.md#how-do-i-create-a-copy-of-an-azure-netapp-files-volume-in-another-azure-region).

  Vous pouvez utiliser la réplication inter-région Azure NetApp Files, qui est actuellement disponible en [préversion](https://azure.microsoft.com/en-us/blog/azure-netapp-files-cross-region-replication-and-new-enhancements-in-preview/) et qui utilise la technologie NetApp SnapMirror®. Ainsi, seuls les blocs modifiés sont envoyés sur le réseau dans un format fiable et compressé. Cette technologie propriétaire limite le volume de données nécessaire à la réplication dans les différentes régions, ce qui permet d’économiser les coûts de transfert de données. Elle réduit également le temps de réplication, ce qui vous permet d’obtenir un objectif de point de restauration (RPO) inférieur. Pour plus d’informations, consultez [Configuration requise et considérations pour la réplication inter-région](../../../azure-netapp-files/cross-region-replication-requirements-considerations.md).

- **Les fichiers Azure Premium** prennent uniquement en charge le stockage localement redondant (LRS) et le stockage redondant interzone (ZRS). Pour la stratégie de récupération d’urgence des fichiers Azure Premium, vous pouvez utiliser [AzCopy](../../../storage/common/storage-use-azcopy-v10.md) ou [Azure PowerShell](/powershell/module/az.storage/) pour copier vos fichiers vers un autre compte de stockage dans une région différente. Pour plus d’informations, consultez [Récupération d’urgence et basculement de compte de stockage](../../../storage/common/storage-disaster-recovery-guidance.md).

#### <a name="cms-database"></a>Base de données CMS

Azure Database pour MySQL offre plusieurs options de récupération de la base de données en cas de sinistre. Choisissez l’option qui convient le mieux à votre entreprise.

- Activez les réplicas en lecture inter-régions pour améliorer la planification de la continuité d’activité et de la reprise d’activité. Vous pouvez effectuer une réplication du serveur source vers cinq réplicas au maximum. Les réplicas en lecture sont mis à jour de manière asynchrone à l’aide de la technologie de réplication des journaux des transactions de MySQL. Les réplicas sont de nouveaux serveurs que vous gérez de manière similaire aux serveurs Azure Database pour MySQL classiques. Pour plus d’informations sur les réplicas en lecture, les régions disponibles, les restrictions et le basculement, consultez cet [article sur les concepts relatifs aux réplicas en lecture](../../../mysql/concepts-read-replicas.md).

- Utilisez la fonctionnalité de géorestauration d’Azure Database pour MySQL qui restaure le serveur à l’aide de sauvegardes géoredondantes. Ces sauvegardes sont accessibles même lorsque la région dans laquelle votre serveur est hébergé est hors connexion. Vous pouvez effectuer une restauration à partir de ces sauvegardes dans n’importe quelle autre région et remettre votre serveur en ligne.

  > [!NOTE]
  > La géorestauration n’est possible que si vous avez provisionné le serveur avec le stockage de sauvegardes géoredondantes. La modification des **options de redondance des sauvegardes** après la création du serveur n’est pas prise en charge. Pour plus d’informations, consultez l’article [Redondance des sauvegardes](../../../mysql/concepts-backup.md#backup-redundancy-options).

Voici la recommandation pour la récupération d’urgence de chaque niveau utilisé dans cet exemple.

| Niveaux de plateforme SAP BOBI   | Recommandation                                                                                           |
|---------------------------|----------------------------------------------------------------------------------------------------------|
| Azure Application Gateway | Configuration parallèle d’Application Gateway sur la région secondaire                                                |
| Serveurs d’applications web   | Répliquer à l’aide de Site Recovery                                                                         |
| Serveurs d’applications décisionnelles    | Répliquer à l’aide de Site Recovery                                                                         |
| Azure NetApp Files        | Outil de copie basé sur des fichiers pour répliquer des données vers la région secondaire **ou** Réplication inter-région ANF (préversion) |
| Azure Database pour MySQL  | Réplicas en lecture inter-régions **ou** Restaurer une sauvegarde à partir de sauvegardes géoredondantes                             |

## <a name="next-steps"></a>Étapes suivantes

- [Configurer la reprise d’activité pour un déploiement d’application SAP multiniveau](../../../site-recovery/site-recovery-sap.md)
- [Planification et implémentation de machines virtuelles Azure pour SAP](planning-guide.md)
- [Déploiement de machines virtuelles Azure pour SAP](deployment-guide.md)
- [Déploiement SGBD de machines virtuelles Azure pour SAP](./dbms_guide_general.md)