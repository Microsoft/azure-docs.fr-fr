---
title: Azure VMware Solution by CloudSimple - Optimiser votre cloud privé CloudSimple pour Oracle RAC
description: Explique comment déployer un nouveau cluster et optimiser une machine virtuelle pour l’installation et la configuration d’Oracle Real Application Clusters (RAC).
author: sharaths-cs
ms.author: b-shsury
ms.date: 08/06/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 980ba86a9916e13dd2ac7639bd06d3ab8546d2f1
ms.sourcegitcommit: 6906980890a8321dec78dd174e6a7eb5f5fcc029
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92424693"
---
# <a name="optimize-your-cloudsimple-private-cloud-for-installing-oracle-rac"></a>Optimiser votre cloud privé CloudSimple pour l’installation d’Oracle RAC

Vous pouvez déployer Oracle Real Application Clusters (RAC) dans votre environnement de cloud privé CloudSimple. Ce guide explique comment déployer un nouveau cluster et optimiser une machine virtuelle pour la solution Oracle RAC. Après avoir effectué les étapes de cette rubrique, vous pouvez installer et configurer Oracle RAC.

## <a name="storage-policy"></a>Stratégie de stockage

La réussite de l’implémentation d’Oracle RAC nécessite un nombre suffisant de nœuds dans le cluster.  Dans la stratégie de stockage vSAN, des défaillances à tolérer (FTT, Failures To Tolerate) sont appliquées aux disques de données utilisés pour le stockage des disques de bases de données, de journaux et de restauration par progression.  Le nombre de nœuds nécessaires pour tolérer efficacement les défaillances est de 2N+1, où N est la valeur de FTT.

Exemple : Si la FTT souhaitée est 2, le nombre total de nœuds dans le cluster doit être égal à 2*2+1 = 5.

## <a name="overview-of-deployment"></a>Vue d’ensemble du déploiement

Les sections suivantes décrivent comment configurer votre environnement de cloud privé CloudSimple pour Oracle RAC.

1. Bonnes pratiques en matière de configuration des disques
2. Déployer un cluster vSphere de cloud privé CloudSimple
3. Configurer la mise en réseau pour Oracle RAC
4. Configurer les stratégies de stockage vSAN
5. Créer des machines virtuelles Oracle et créer des disques de machines virtuelles partagés
6. Configurer des règles d’affinité de machine virtuelle à hôte

## <a name="best-practices-for-disk-configuration"></a>Bonnes pratiques en matière de configuration des disques

Les machines virtuelles Oracle RAC ont plusieurs disques, qui sont utilisés pour une fonction spécifique.  Des disques partagés sont montés sur toutes les machines virtuelles, et utilisés par le cluster RAC Oracle.  Les disques d’installation de système d’exploitation et de logiciels sont montés uniquement sur les machines virtuelles individuelles.  

![Vue d’ensemble des disques de machines virtuelles Oracle RAC](media/oracle-vm-disks-overview.png)

L’exemple suivant utilise les disques définis dans le tableau ci-dessous.

| Disque                                      | Objectif                                       | Disque partagé |
|-------------------------------------------|-----------------------------------------------|-------------|
| Système d''exploitation                                        | Disque de système d’exploitation                         | Non          |
| GRID                                      | Emplacement d’installation du logiciel Oracle Grid     | Non          |
| DATABASE                                  | Emplacement d’installation du logiciel de base de données Oracle | Non          |
| ORAHOME                                   | Emplacement de base des fichiers binaires de base de données Oracle    | Non          |
| DATA1, DATA2, DATA3, DATA4                | Disque où sont stockés les fichiers de base de données Oracle   | Oui         |
| REDO1, REDO2, REDO3, REDO4, REDO5, REDO6  | Disques des journaux de phase de restauration par progression                                | Oui         |
| OCR1, OCR2, OCR3, OCR4, OCR5              | Disques votants                                  | Oui         |
| FRA1, FRA2                                | Disques de la zone de récupération rapide                      | Oui         |

![Configuration des disques de machines virtuelles Oracle](media/oracle-vmdk.png)

### <a name="virtual-machine-configuration"></a>Configuration de la machine virtuelle

* Chaque machine virtuelle est configurée avec quatre contrôleurs SCSI.
* Le type de contrôleur SCSI est défini sur VMware paravirtual.
* Plusieurs disques virtuels (.vmdk) sont créés.
* Les disques sont montés sur différents contrôleurs SCSI.
* Le type de partage multi-writer est défini pour les disques de clusters partagés.
* La stratégie de stockage vSAN est définie pour garantir la haute disponibilité des disques.

### <a name="operating-system-and-software-disk-configuration"></a>Configuration des disques de système d’exploitation et de logiciels

Chaque machine virtuelle Oracle est configurée avec plusieurs disques pour le système d’exploitation hôte, l’échange, l’installation de logiciels et d’autres fonctions du système d’exploitation.  Ces disques ne sont pas partagés par les machines virtuelles.  

* Trois disques pour chaque machine virtuelle sont configurés en tant que disques virtuels et montés sur des machines virtuelles Oracle RAC.
    * Disque de système d’exploitation
    * Disque pour le stockage des fichiers d’installation d’Oracle Grid
    * Disque pour le stockage des fichiers d’installation de base de données Oracle
* Les disques peuvent être configurés comme **Thin Provisioned** (alloués dynamiquement).
* Chaque disque est monté sur le premier contrôleur SCSI (SCSI0).  
* Le partage est défini sur **No sharing** (Aucun partage).
* La redondance est définie sur le stockage à l’aide de stratégies vSAN.  

![Diagramme illustrant la configuration physique du disque de système d'exploitation Oracle RAC.](media/oracle-vm-os-disks.png)

### <a name="data-disk-configuration"></a>Configuration des disques de données

Les disques de données sont principalement utilisés pour le stockage des fichiers de base de données.  

* Quatre disques sont configurés en tant que disques virtuels et montés sur toutes les machines virtuelles Oracle RAC.
* Chaque disque est monté sur un contrôleur SCSI différent.
* Chaque disque virtuel est configuré au format **Thick Provision Eager Zeroed**.  
* Le partage est défini sur **Multi-writer**.  
* Les disques doivent être configurés en tant que groupe de disques ASM (Automatic Storage Management).  
* La redondance est définie sur le stockage à l’aide de stratégies vSAN.  
* La redondance ASM est définie sur **External**.

![Configuration des groupes de disques de données Oracle RAC](media/oracle-vm-data-disks.png)

### <a name="redo-log-disk-configuration"></a>Configuration des disques des journaux de phase de restauration par progression

Les fichiers journaux de phase de restauration par progression servent à stocker une copie des modifications apportées à la base de données.  Ils sont utilisés quand les données doivent être récupérées après une défaillance.

* Les disques des journaux de phase de restauration par progression doivent être configurés en tant que groupes de disques multiples.  
* Six disques sont créés et montés sur toutes les machines virtuelles Oracle RAC.
* Les disques sont montés sur différents contrôleurs SCSI.
* Chaque disque virtuel est configuré au format **Thick Provision Eager Zeroed**.
* Le partage est défini sur **Multi-writer**.  
* Les disques doivent être configurés dans deux groupes de disques ASM.
* Chaque groupe de disques ASM contient trois disques, qui se trouvent sur des contrôleurs SCSI différents.  
* La redondance ASM est définie sur **Normal**.
* Cinq fichiers journaux de phase de restauration par progression sont créés sur le groupe de journaux de phase de restauration par progression ASM.

```
SQL > alter database add logfile thread 1 ('+ORCLRAC_REDO1','+ORCLRAC_REDO2') size 1G;
SQL > alter database add logfile thread 1 ('+ORCLRAC_REDO1','+ORCLRAC_REDO2') size 1G;
SQL > alter database add logfile thread 1 ('+ORCLRAC_REDO1','+ORCLRAC_REDO2') size 1G;
SQL > alter database add logfile thread 1 ('+ORCLRAC_REDO1','+ORCLRAC_REDO2') size 1G;
SQL > alter database add logfile thread 1 ('+ORCLRAC_REDO1','+ORCLRAC_REDO2') size 1G;
SQL > alter database add logfile thread 2 ('+ORCLRAC_REDO1','+ORCLRAC_REDO2') size 1G;
SQL > alter database add logfile thread 2 ('+ORCLRAC_REDO1','+ORCLRAC_REDO2') size 1G;
SQL > alter database add logfile thread 2 ('+ORCLRAC_REDO1','+ORCLRAC_REDO2') size 1G;
SQL > alter database add logfile thread 2 ('+ORCLRAC_REDO1','+ORCLRAC_REDO2') size 1G;
SQL > alter database add logfile thread 2 ('+ORCLRAC_REDO1','+ORCLRAC_REDO2') size 1G;
```

![Configuration des groupes de disques de journaux de phase de restauration par progression Oracle RAC](media/oracle-vm-redo-log-disks.png)

### <a name="oracle-voting-disk-configuration"></a>Configuration des disque votants Oracle

Les disques votants fournissent une fonctionnalité de disque de quorum comme canal de communication supplémentaire afin d’éviter toute situation de fractionnement.

* Cinq disques sont créés et montés sur toutes les machines virtuelles Oracle RAC.
* Les disques sont montés sur un contrôleur SCSI.
* Chaque disque virtuel est configuré au format **Thick Provision Eager Zeroed**.
* Le partage est défini sur **Multi-writer**.  
* Les disques doivent être configurés dans un même groupe de disques ASM.  
* La redondance ASM est définie sur **High**.

![Configuration des groupes de disques de données Oracle RAC](media/oracle-vm-voting-disks.png)

### <a name="oracle-fast-recovery-area-disk-configuration-optional"></a>Configuration des disques de la zone de récupération rapide Oracle (facultatif)

La zone de récupération rapide (FRA, Fast Recovery Area) est gérée par le système de fichiers par le biais du groupe de disques Oracle ASM.  Elle fournit un emplacement de stockage partagé pour les fichiers de sauvegarde et de récupération. Oracle crée des journaux archivés et des journaux Flashback dans la zone de récupération rapide. Oracle Recovery Manager (RMAN) peut éventuellement stocker ses jeux de sauvegardes et copies d’images dans la zone de récupération rapide, et il l’utilise dans le cadre de la restauration de fichiers pendant la récupération des supports.

* Deux disques sont créés et montés sur toutes les machines virtuelles Oracle RAC.
* Les disques sont montés sur différents contrôleurs SCSI.
* Chaque disque virtuel est configuré au format **Thick Provision Eager Zeroed**.
* Le partage est défini sur **Multi-writer**.  
* Les disques doivent être configurés dans un même groupe de disques ASM.  
* La redondance ASM est définie sur **External**.

![Diagramme illustrant la configuration du groupe de disques votants Oracle RAC.](media/oracle-vm-fra-disks.png)

## <a name="deploy-cloudsimple-private-cloud-vsphere-cluster"></a>Déployer un cluster vSphere de cloud privé CloudSimple

Pour déployer un cluster vSphere sur votre cloud privé, effectuez les étapes suivantes :

1. À partir du portail CloudSimple, [créez un cloud privé](create-private-cloud.md). CloudSimple crée un utilisateur vCenter par défaut nommé « cloudowner » dans chaque nouveau cloud privé. Pour plus d’informations sur l’utilisateur et le modèle d’autorisation par défaut du cloud privé, consultez [Modèle d’autorisation du cloud privé](learn-private-cloud-permissions.md).  Cette étape crée le cluster de gestion principal pour votre cloud privé.

2. À partir du portail CloudSimple, [développez le cloud privé](expand-private-cloud.md) avec un nouveau cluster.  Ce cluster sera utilisé pour déployer Oracle RAC.  Sélectionnez le nombre de nœuds en fonction de la tolérance de panne souhaitée (trois nœuds au minimum).

## <a name="set-up-networking-for-oracle-rac"></a>Configurer la mise en réseau pour Oracle RAC

1. Dans votre cloud privé, [créez deux réseaux locaux virtuels](create-vlan-subnet.md) (un pour le réseau public Oracle et un pour le réseau privé Oracle) et attribuez les CIDR de sous-réseau appropriés.
2. Une fois les réseaux locaux virtuels créés, créez les [groupes de ports distribués sur le cloud privé vCenter](create-vlan-subnet.md#use-vlan-information-to-set-up-a-distributed-port-group-in-vsphere).
3. Configurez une [machine virtuelle de serveur DHCP et DNS](dns-dhcp-setup.md) sur votre cluster de gestion pour l’environnement Oracle.
4. [Configurez le transfert DNS sur le serveur DNS](on-premises-dns-setup.md#create-a-conditional-forwarder) installé dans le cloud privé.

## <a name="set-up-vsan-storage-policies"></a>Configurer les stratégies de stockage vSAN

Les stratégies vSAN définissent les défaillances à tolérer et l’entrelacement de disques pour les données stockées sur les disques de machines virtuelles.  La stratégie de stockage créée doit être appliquée sur les disques de machines virtuelles lors de la création de la machine virtuelle.

1. [Connectez-vous au client vSphere](./vcenter-access.md) de votre cloud privé.
2. Dans le menu supérieur, sélectionnez **Policies and Profiles** (Stratégies et profils).
3. Dans le menu de gauche, sélectionnez **VM Storage Policies** (Stratégies de stockage de machines virtuelles), puis **Create a VM storage Policy** (Créer une stratégie de stockage de machines virtuelles).
4. Entrez un nom explicite pour la stratégie, puis cliquez sur **NEXT**.
5. Dans la section **Policy structure** (Structure de la stratégie), sélectionnez **Enable rules for vSAN storage** (Activer les règles pour le stockage vSAN), puis cliquez sur **NEXT**.
6. Dans la section **vSAN** > **Availability** (Disponibilité), sélectionnez **None** pour Site disaster tolerance (Tolérance pour la reprise d’activité de site). Pour Failures to tolerate, sélectionnez l’option de mise en miroir RAID ( **RAID - Mirroring** ) pour la FTT souhaitée.
    ![Paramètres de vSAN](media/oracle-rac-storage-wizard-vsan.png).
7. Dans la section **Advanced** , sélectionnez le nombre d’agrégations de disques par objet. Pour Object space reservation (Réservation d’espace d’objet), sélectionnez **Thick Provisioned** (Alloué de manière statique). Sélectionnez **Disable object checksum** (Désactiver la somme de contrôle des objets). Cliquez sur **NEXT**.
8. Suivez les instructions à l’écran pour afficher la liste des banques de données vSAN compatibles, passer en revue les paramètres et terminer l’installation.

## <a name="create-oracle-vms-and-create-shared-vm-disks-for-oracle"></a>Créer des machines virtuelles Oracle et créer des disques de machines virtuelles partagés pour Oracle

Pour créer une machine virtuelle pour Oracle, clonez une machine virtuelle existante ou créez-en une nouvelle.  Cette section décrit comment créer une machine virtuelle, puis la cloner pour en créer une deuxième après avoir installé le système d’exploitation de base.  Une fois les machines virtuelles créées, vous pouvez y ajouter des disques.  Le cluster Oracle utilise des disques partagés pour le stockage des données, des journaux et des journaux de phase de restauration par progression.

### <a name="create-vms"></a>Créer des machines virtuelles

1. Dans vCenter, cliquez sur l'icône **Hôtes et clusters**. Sélectionnez le cluster que vous avez créé pour Oracle.
2. Cliquez avec le bouton droit de la souris sur le cluster et sélectionnez **New Virtual Machine** (Nouvelle machine virtuelle).
3. Sélectionnez **Créer une machine virtuelle** , puis cliquez sur **Suivant**.
4. Nommez la machine, sélectionnez l’emplacement de la machine virtuelle Oracle, puis cliquez sur **Next**.
5. Sélectionnez la ressource de cluster, puis cliquez sur **Next**.
6. Sélectionnez la banque de données vSAN pour le cluster, puis cliquez sur **Next**.
7. Conservez la sélection de compatibilité ESXi 6.5 par défaut, puis cliquez sur **Suivant**.
8. Sélectionnez le système d’exploitation invité correspondant à l’ISO de la machine virtuelle que vous créez, puis cliquez sur **Suivant**.
9. Sélectionnez la taille de disque dur requise pour l’installation du système d’exploitation.
10. Pour installer l’application sur un autre appareil, cliquez sur **Add new device** (Ajouter un nouvel appareil).
11. Sélectionnez les options réseau et affectez le groupe de ports distribués créé pour le réseau public.
12. Pour ajouter des interfaces réseau supplémentaires, cliquez sur **Add new device** , puis sélectionnez le groupe de ports distribués créé pour le réseau privé.
13. Pour New DC/DVD Drive (Nouveau lecteur CD/DVD), sélectionnez le fichier ISO de banques de données qui contient l’image ISO de l’installation de système d’exploitation de votre choix. Sélectionnez le fichier que vous avez chargé précédemment dans le dossier des fichiers ISO et des modèles, puis cliquez sur **OK**.
14. Vérifiez les paramètres, puis cliquez sur **OK** pour créer la machine virtuelle.
15. Mettez la machine virtuelle sous tension. Installez le système d’exploitation et toutes les mises à jour requises.

Une fois le système d’exploitation installé, vous pouvez cloner une deuxième machine virtuelle. Cliquez avec le bouton droit sur l’entrée de machine virtuelle, puis sélectionnez l’option Clone.

### <a name="create-shared-disks-for-vms"></a>Créer des disques partagés pour les machines virtuelles

Oracle utilise un disque partagé pour stocker les fichiers de données, journaux, et journaux de phase de restauration par progression.  Vous pouvez créer un disque partagé sur vCenter et le monter sur les deux machines virtuelles.  Pour bénéficier de meilleures performances, placez les disques de données sur différents contrôleurs SCSI. Les étapes ci-dessous montrent comment créer un disque partagé sur vCenter, puis l’attacher à une machine virtuelle. Le client Flash vCenter est utilisé pour modifier les propriétés de la machine virtuelle.

#### <a name="create-disks-on-the-first-vm"></a>Créer des disques sur la première machine virtuelle

1. Dans vCenter, cliquez avec le bouton droit sur l’une des machines virtuelles Oracle, puis sélectionnez **Edit settings**.
2. Dans la section New Device, sélectionnez **SCSI controller** , puis cliquez sur **Add**.
3. Dans la section New Device, sélectionnez **New Hard disk** (Nouveau disque dur), puis cliquez sur **Add**.
4. Développez les propriétés du nouveau disque dur.
5. Spécifiez la taille du disque dur.
6. Spécifiez la stratégie de stockage vSAN que vous avez définie précédemment comme stratégie de stockage de la machine virtuelle.
7. Sélectionnez un dossier de la banque de données vSAN comme emplacement. L’emplacement facilite la navigation et l’attachement des disques à une deuxième machine virtuelle.
8. Pour le provisionnement des disques, sélectionnez **Thick provision eager zeroed**.
9. Pour le partage, spécifiez **Multi-writer**.
10. Pour le nœud d’appareil virtuel, sélectionnez le nouveau contrôleur SCSI créé à l’étape 2.

    ![Capture d'écran mettant en évidence les champs nécessaires à la création de disques sur la première machine virtuelle.](media/oracle-rac-new-hard-disk.png)

Répétez les étapes 2 à 10 pour tous les nouveaux disques requis pour les fichiers de données Oracle, les fichiers journaux et les fichiers journaux de phase de restauration par progression.

#### <a name="attach-disks-to-second-vm"></a>Attacher des disques à la deuxième machine virtuelle

1. Dans vCenter, cliquez avec le bouton droit sur l’une des machines virtuelles Oracle, puis sélectionnez **Edit settings**.
2. Dans la section New Device, sélectionnez **SCSI controller** , puis cliquez sur **Add**.
3. Dans la section New Device, sélectionnez **Existing Hard disk** (Disque dur existant), puis cliquez sur **Add**.
4. Accédez à l’emplacement où le disque a été créé pour la première machine virtuelle et sélectionnez le fichier VMDK.
5. Spécifiez la stratégie de stockage vSAN que vous avez définie précédemment comme stratégie de stockage de la machine virtuelle.
6. Pour le provisionnement des disques, sélectionnez **Thick provision eager zeroed**.
7. Pour le partage, spécifiez **Multi-writer**.
8. Pour le nœud d’appareil virtuel, sélectionnez le nouveau contrôleur SCSI créé à l’étape 2.

    ![Créer des disques sur la première machine virtuelle](media/oracle-rac-existing-hard-disk.png)

Répétez les étapes 2 à 7 pour tous les nouveaux disques requis pour les fichiers de données Oracle, les fichiers journaux et les fichiers journaux de phase de restauration par progression.

## <a name="set-up-vm-host-affinity-rules"></a>Configurer des règles d’affinité de machine virtuelle à hôte

Les règles d’affinité de machine virtuelle à hôte garantissent que la machine virtuelle s’exécute sur l’hôte souhaité.  Vous pouvez définir des règles sur vCenter afin d’être sûr que la machine virtuelle Oracle s’exécute sur l’hôte disposant des ressources adéquates et afin de répondre à des exigences spécifiques en matière de gestion des licences.

1. Dans le portail CloudSimple, [augmentez les privilèges](escalate-private-cloud-privileges.md) de l’utilisateur cloudowner.
2. Connectez-vous au client vSphere de votre cloud privé.
3. Dans le client vSphere, sélectionnez le cluster où les machines virtuelles Oracle sont déployées, puis cliquez sur **Configure**.
4. Sous Configure, sélectionnez **VM/Host Groups** (Groupes d’hôtes/machines virtuelles).
5. Cliquez sur **+**
6. Ajoutez un groupe de machines virtuelles. Sélectionnez **VM group** (Groupe de machines virtuelles) en tant que type. Entrez le nom du groupe. Sélectionnez les machines virtuelles, puis cliquez sur **OK** pour créer le groupe.
6. Ajoutez un groupe hôte. Sélectionnez **Host Group** (Groupe hôte) comme type. Entrez le nom du groupe. Sélectionnez les hôtes sur lesquels les machines virtuelles seront exécutées, puis cliquez sur **OK** pour créer le groupe.
7. Pour créer une règle, cliquez sur **VM/Host rules** (Règles d’hôtes/machines virtuelles).
8. Cliquez sur **+**
9. Entrez un nom pour la règle et cochez **Enable** (Activer).
10. Comme type de règle, sélectionnez **Virtual Machines to Host** (Machines virtuelles à hôte).
11. Sélectionnez le groupe de machines virtuelles qui contient les machines virtuelles Oracle.
12. Sélectionnez **Must run on hosts in this group** (Doit s’exécuter sur des hôtes de ce groupe).
13. Sélectionnez le groupe hôte que vous avez créé.
14. Cliquez sur **OK** pour créer la règle.

## <a name="references"></a>References

* [About vSAN Policies](https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.virtualsan.doc/GUID-08911FD3-2462-4C1C-AE81-0D4DBC8F7990.html) (À propos des stratégies vSAN)
* [VMware Multi-Writer Attribute for Shared VMDKs](https://docs.vmware.com/en/VMware-Cloud-on-AWS/solutions/VMware-Cloud-on-AWS.df6735f8b729fee463802083d46fdc75/GUID-A7642A82B3D6C5F7806DB40A3F2766D9.html) (Attribut Multi-Writer VMware pour les VMDK partagés)
