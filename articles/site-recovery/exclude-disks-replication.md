---
title: Exclure des disques de la réplication avec Azure Site Recovery
description: Comme exclure des disques de la réplication vers Azure avec Azure Site Recovery.
ms.topic: conceptual
ms.date: 12/17/2019
ms.openlocfilehash: 15989fbfd65f758eb777c5170c217aba8707e0be
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96008257"
---
# <a name="exclude-disks-from-disaster-recovery"></a>Exclure des disques de la reprise d’activité

Cet article explique comment exclure des disques de la réplication lors de la reprise d’activité d’un site local vers Azure avec [Azure Site Recovery](site-recovery-overview.md). Vous pouvez exclure des disques de la réplication pour plusieurs raisons :

- Pour que l’activité des données sans importance sur le disque exclu ne soient pas répliquée.
- Pour optimiser la bande passante utilisée pour la réplication ou les ressources côté cible.
- Pour économiser des ressources de stockage et réseau en ne répliquant pas les données dont vous n’avez pas besoin.
- Parce que les machines virtuelles Azure ont atteint les limites de réplication Site Recovery.


## <a name="supported-scenarios"></a>Scénarios pris en charge

Vous pouvez exclure des disques de la réplication comme décrit dans le tableau.

**Azure vers Azure** | **VMware vers Azure** | **Hyper-V vers Azure** | **Serveur physique vers Azure**
--- | --- | --- | ---
Oui | Oui | Oui | Oui

## <a name="exclude-limitations"></a>Limites d’exclusion

**Limite** | **Machines virtuelles Azure** | **Machines virtuelles VMware** | **Machines virtuelles Hyper-V**
--- | --- | ---
**Types de disques** | Les disques de base peuvent être exclus de la réplication.<br/><br/> Vous ne pouvez pas exclure de disque de système d’exploitation ni de disque dynamique. Les disques temporaires sont exclus par défaut. | Les disques de base peuvent être exclus de la réplication.<br/><br/> Vous ne pouvez pas exclure de disque de système d’exploitation ni de disque dynamique. | Les disques de base peuvent être exclus de la réplication.<br/><br/> Vous ne pouvez pas exclure les disques de système d’exploitation. Nous vous recommandons de ne pas exclure de disques dynamiques. Site Recovery ne peut pas identifier le disque VHS qui est de type de base ou dynamique dans la machine virtuelle invitée. Si tous les disques de volume dynamique dépendants ne sont pas exclus, le disque dynamique protégé devient un disque défectueux sur la machine virtuelle de basculement, et les données de ce disque ne sont pas accessibles.
**Disque en cours de réplication** | Vous ne pouvez pas exclure un disque en cours de réplication.<br/><br/> Désactivez et réactivez la réplication pour la machine virtuelle. |  Vous ne pouvez pas exclure un disque en cours de réplication. |  Vous ne pouvez pas exclure un disque en cours de réplication.
**Service Mobility (VMware)** | Non pertinent | Vous pouvez exclure les disques uniquement sur les machines virtuelles sur lesquelles le service Mobility est installé.<br/><br/> Cela signifie que vous devez installer manuellement le service Mobility sur les machines virtuelles pour lesquelles vous souhaitez exclure des disques. Vous ne pouvez pas utiliser le mécanisme d’installation Push parce qu’il installe le service Mobility uniquement après l’activation de la réplication. | Non pertinent.
**Ajouter/Supprimer** | Vous pouvez ajouter des disques managés sur des machines virtuelles Azure compatibles avec la réplication et dotées de disques managés. Vous ne pouvez pas supprimer des disques sur des machines virtuelles Azure dont la réplication est activée. | Vous ne pouvez pas ajouter ni supprimer de disques après l’activation de la réplication. Désactivez, puis réactivez la réplication pour ajouter un disque. | Vous ne pouvez pas ajouter ni supprimer de disques après l’activation de la réplication. Désactivez, puis réactivez la réplication.
**Type de basculement** | Si une application a besoin d’un disque que vous avez exclu, vous devez créer le disque manuellement après le basculement pour que l’application répliquée puisse s’exécuter.<br/><br/> Vous pouvez également créer le disque lors du basculement de la machine virtuelle en intégrant Azure Automation dans un plan de récupération. | Si vous excluez un disque dont une application a besoin, créez-le manuellement dans Azure après le basculement. | Si vous excluez un disque dont une application a besoin, créez-le manuellement dans Azure après le basculement.
**Restauration automatique sur site local, disques créés manuellement** | Non pertinent | **Machines virtuelles Windows** : Les disques créés manuellement dans Azure ne sont pas restaurés automatiquement. Par exemple, si vous basculez trois disques et que vous en créez deux directement sur une machine virtuelle Azure, seuls les trois disques qui ont été basculés sont restaurés automatiquement.<br/><br/> **Machines virtuelles Linux** : Les disques créés manuellement dans Azure sont restaurés automatiquement. Par exemple, si vous basculez trois disques et que vous en créez deux sur une machine virtuelle Azure, les cinq disques sont restaurés automatiquement. Vous ne pouvez pas exclure de disques créés manuellement de la restauration automatique. | Les disques créés manuellement dans Azure ne sont pas restaurés automatiquement. Par exemple, si vous basculez trois disques et que vous en créez deux directement sur une machine virtuelle Azure, seuls les trois disques qui ont été basculés sont restaurés automatiquement.
**Restauration automatique sur site local, disques exclus** | Non pertinent | Si vous effectuez une restauration automatique vers l’ordinateur d’origine, la configuration de disque de machine virtuelle de restauration automatique n’inclut pas les disques exclus. Les disques qui étaient exclus de la réplication VMware vers Azure ne sont pas disponibles sur la machine virtuelle de restauration automatique. | Lorsque la restauration automatique est effectuée à l’emplacement Hyper-V d’origine, la configuration de disque de machine virtuelle de restauration automatique reste la même que celle du disque de machine virtuelle source d’origine. Les disques qui étaient exclus de la réplication de site Hyper-V vers Azure sont disponibles sur la machine virtuelle de restauration automatique.



## <a name="typical-scenarios"></a>Scénarios classiques

Les écritures dans un fichier d’échange (pagefile.sys) et les écritures dans le fichier tempdb de Microsoft SQL Server sont des exemples d’activité des données qui sont de bons candidats pour l’exclusion. Selon la charge de travail et le sous-système de stockage, les fichiers d’échange et tempdb peuvent enregistrer une quantité significative de données d’activité. La réplication de ce type de données vers Azure consomme beaucoup de ressources.

- Pour optimiser la réplication d’une machine virtuelle avec un seul disque virtuel qui comprend à la fois le système d’exploitation et le fichier d’échange, vous pouvez :
    1. Fractionnez le disque virtuel unique en deux disques virtuels. Un disque virtuel comporte le système d’exploitation, et l’autre le fichier d’échange.
    2. Excluez le disque de fichier d’échange de la réplication.

- Pour optimiser la réplication d’un disque qui contient à la fois le fichier Microsoft SQL Server tempdb et le fichier de base de données système, vous pouvez :
    1. Stockez la base de données système et la base de données tempdb sur deux disques différents.
    2. Excluez le disque de base de données tempdb de la réplication.

## <a name="example-1-exclude-the-sql-server-tempdb-disk"></a>Exemple 1 : Exclure le disque de base de données tempdb SQL Server

Nous allons voir comment gérer l’exclusion de disque, le basculement et le basculement d’une machine virtuelle Windows SQL Server source - **SalesDB** _, pour laquelle nous souhaitons exclure tempdb. 

### <a name="exclude-disks-from-replication"></a>Exclure les disques de la réplication

Nous avons ces disques sur la machine virtuelle Windows source SalesDB.

_ *Nom du disque** | **Disque de système d’exploitation invité** | **Lettre de lecteur** | **Type de données de disque**
--- | --- | --- | ---
DB-Disk0-OS | Disk0 | C:\ | Disque de système d’exploitation.
DB-Disk1| Disk1 | D:\ | Base de données système SQL et base de données utilisateur 1.
DB-Disk2 (disque exclu de la protection) | Disk2 | E:\ | Fichiers temporaires.
DB-Disk3 (disque exclu de la protection) | Disk3 | F:\ | Base de données SQL tempdb.<br/><br/> Chemin de dossier - F:\MSSQL\Data\. Notez le chemin du dossier avant le basculement.
DB-Disk4 | Disk4 |G:\ | Base de données utilisateur 2

1. Nous activons la réplication pour la machine virtuelle SalesDB.
2. Nous excluons Disk2 et Disk3 de la réplication, car l’activité des données sur ces disques est temporaire. 


### <a name="handle-disks-during-failover"></a>Gérer les disques pendant le basculement

Étant donné que les disques ne sont pas répliqués, lorsque vous effectuez un basculement vers Azure, ces disques ne sont pas présents sur la machine virtuelle Azure créée après le basculement. Les disques de la machine virtuelle Azure sont décrits dans ce tableau.

**Disque de système d’exploitation invité** | **Lettre de lecteur** | **Type de données de disque**
--- | --- | ---
Disk0 | C:\ | Disque de système d’exploitation.
Disk1 | E:\ | Stockage temporaire<br/><br/>Azure ajoute ce disque. Comme les disques Disk2 et Disk3 ont été exclus de la réplication, E: est la première lettre de lecteur disponible dans la liste. Azure attribue donc la lettre E: au volume de stockage temporaire. Les autres lettres de lecteur des disques répliqués restent identiques.
Disk2 | D:\ | Base de données système SQL et base de données utilisateur 1
Disk3 | G:\ | Base de données utilisateur 2

Dans notre exemple, étant donné que Disk3, le disque SQL tempdb, a été exclu de la réplication et qu’il n’est pas disponible sur la machine virtuelle Azure, le service SQL est dans l’état arrêté et il a besoin du chemin F:\MSSQL\Data. Vous pouvez créer ce chemin de deux manières : 

- Ajoutez un nouveau disque après le basculement et attribuez-lui le chemin du dossier tempdb.
- Utilisez un disque de stockage temporaire existant pour le chemin du dossier de tempdb.

#### <a name="add-a-new-disk-after-failover"></a>Ajouter un nouveau disque après le basculement

1. Notez le chemin des fichiers SQL tempdb.mdf et tempdb.ldf avant de procéder au basculement.
2. À partir du portail Azure, ajoutez un nouveau disque à la machine virtuelle Azure de basculement. La taille du disque doit être identique (ou supérieure) à celle du disque SQL tempdb source (Disk3).
3. Connectez-vous à la machine virtuelle Azure.
4. À partir de la console de gestion des disques (diskmgmt.msc), initialisez et formatez le disque que vous venez d’ajouter.
5. Attribuez-lui la lettre de lecteur qui était utilisée par le disque SQL tempdb (F:)
6. Créez un dossier de tempdb sur le volume F: (F:\MSSQL\Data).
7. Démarrez le service SQL à partir de la console de service.

#### <a name="use-an-existing-temporary-storage-disk"></a>Utiliser un disque de stockage temporaire existant 

1. Ouvrez une invite de commandes.
2. Exécutez SQL Server en mode de récupération à partir de l’invite de commande.

    ```console
    Net start MSSQLSERVER /f / T3608
    ```

3. Exécutez la commande sqlcmd suivante pour remplacer le chemin de la base de données tempdb par un nouveau chemin.

    ```sql
    sqlcmd -A -S SalesDB        **Use your SQL DBname**
    USE master;     
    GO      
    ALTER DATABASE tempdb       
    MODIFY FILE (NAME = tempdev, FILENAME = 'E:\MSSQL\tempdata\tempdb.mdf');
    GO      
    ALTER DATABASE tempdb       
    MODIFY FILE (NAME = templog, FILENAME = 'E:\MSSQL\tempdata\templog.ldf');       
    GO
    ```

4. Arrêtez le service Microsoft SQL Server.

    ```console
    Net stop MSSQLSERVER
    ```

5. Démarrez le service Microsoft SQL Server.

    ```console
    Net start MSSQLSERVER
    ```

### <a name="vmware-vms-disks-during-failback-to-original-location"></a>Machines virtuelles VMware : disques lors de la restauration automatique à l’emplacement d’origine

Voyons maintenant comment gérer les disques sur les machines virtuelles VMware lorsque vous effectuez une restauration automatique à votre emplacement local d’origine.

- **Disques créés dans Azure** : Dans la mesure où notre exemple utilise une machine virtuelle Windows, les disques que vous créez manuellement dans Azure ne sont pas répliqués sur votre site lorsque vous effectuez une restauration automatique ou reprotégez une machine virtuelle.
- **Disque de stockage temporaire dans Azure** : Le disque de stockage temporaire n’est pas répliqué vers les hôtes locaux.
- **Disques exclus** : Les disques qui étaient exclus de la réplication VMware vers Azure ne sont pas disponibles sur la machine virtuelle locale après la restauration automatique.

Avant de restaurer les machines virtuelles VMware à l’emplacement d’origine, les paramètres du disque de machine virtuelle Azure sont les suivants.

**Disque de système d’exploitation invité** | **Lettre de lecteur** | **Type de données de disque**
--- | --- | ---
Disk0 | C:\ | Disque de système d’exploitation.
Disk1 | E:\ | Stockage temporaire.
Disk2 | D:\ | Base de données système SQL et base de données utilisateur 1.
Disk3 | G:\ | Base de données utilisateur 2.

Après la restauration automatique, les disques de la machine virtuelle VMware à l’emplacement d’origine sont décrits dans le tableau.

**Disque de système d’exploitation invité** | **Lettre de lecteur** | **Type de données de disque**
--- | --- | ---
Disk0 | C:\ | Disque de système d’exploitation.
Disk1 | D:\ | Base de données système SQL et base de données utilisateur 1.
Disk2 | G:\ | Base de données utilisateur 2.


### <a name="hyper-v-vms-disks-during-failback-to-original-location"></a>Machines virtuelles Hyper-V : disques lors de la restauration automatique à l’emplacement d’origine

Voyons maintenant comment gérer les disques sur les machines virtuelles Hyper-V lorsque vous effectuez une restauration automatique à votre emplacement local d’origine.

- **Disques créés dans Azure** : Les disques que vous créez manuellement dans Azure ne sont pas répliqués sur votre site lorsque vous effectuez une restauration automatique ou reprotégez une machine virtuelle.
- **Disque de stockage temporaire dans Azure** : Le disque de stockage temporaire n’est pas répliqué vers les hôtes locaux.
- **Disques exclus** : Après la restauration automatique, la configuration de disque de machine virtuelle est identique à celle d’origine. Les disques qui étaient exclus de la réplication entre Hyper-V et Azure sont disponibles sur la machine virtuelle de restauration automatique.

Avant de restaurer les machines virtuelles Hyper-V à l’emplacement d’origine, les paramètres du disque de machine virtuelle Azure sont les suivants.

**Disque de système d’exploitation invité** | **Lettre de lecteur** | **Type de données de disque**
--- | --- | ---
Disk0 | C:\ | Disque de système d’exploitation.
Disk1 | E:\ | Stockage temporaire.
Disk2 | D:\ | Base de données système SQL et base de données utilisateur 1.
Disk3 | G:\ | Base de données utilisateur 2.

Après le basculement planifié (la restauration automatique) d’Azure vers Hyper-V en local, les disques de la machine virtuelle Hyper-V à l’emplacement d’origine sont décrits dans ce tableau.

**Nom du disque** | **N° de disque de SE invité** | **Lettre de lecteur** | **Type de données de disque**
 --- | --- | --- | ---
DB-Disk0-OS | Disk0 |   C:\ | Disque de système d’exploitation.
DB-Disk1 | Disk1 | D:\ | Base de données système SQL et base de données utilisateur 1.
DB-Disk2 (disque exclu) | Disk2 | E:\ | Fichiers temporaires.
DB-Disk3 (disque exclu) | Disk3 | F:\ | Base de données SQL tempdb<br/><br/> Chemin de dossier (F:\MSSQL\Data\).
DB-Disk4 | Disk4 | G:\ | Base de données utilisateur 2


## <a name="example-2-exclude-the-paging-file-disk"></a>Exemple 2 : Exclure le disque de fichier d’échange

Nous allons voir comment gérer l’exclusion de disque, le basculement et le basculement d’une machine virtuelle Windows source, pour laquelle nous souhaitons exclure le disque de fichier pagefile.sys à la fois sur le lecteur D et sur un autre lecteur.


### <a name="paging-file-on-the-d-drive"></a>Fichier d’échange sur le lecteur D

Nous avons ces disques sur la machine virtuelle source.

**Nom du disque** | **Disque de système d’exploitation invité** | **Lettre de lecteur** | **Type de données de disque**
--- | --- | --- | ---
DB-Disk0-OS | Disk0 | C:\ | Disque de système d’exploitation
DB-Disk1 (à exclure de la réplication) | Disk1 | D:\ | pagefile.sys
DB-Disk2 | Disk2 | E:\ | Données utilisateur 1
DB-Disk3 | Disk3 | F:\ | Données utilisateur 2

Les paramètres du fichier d’échange sur la machine virtuelle source sont les suivants :

![Capture d’écran de la boîte de dialogue Mémoire virtuelle avec la ligne Lecteur D : [volume Pagefile] en surbrillance montrant une Taille de fichier d’échange (Mo) de 3000-7000.](./media/exclude-disks-replication/pagefile-d-drive-source-vm.png)

1. Nous activons la réplication pour la machine virtuelle.
2. Nous excluons DB-Disk1 de la réplication.

#### <a name="disks-after-failover"></a>Disques après le basculement

Après le basculement, les disques de la machine virtuelle Azure sont décrits dans le tableau.

**Nom du disque** | **Numéro du disque du système d’exploitation invité** | **Lettre de lecteur** | **Type de données sur le disque**
--- | --- | --- | ---
DB-Disk0-OS | Disk0 | C:\ | Disque de système d’exploitation
DB-Disk1 | Disk1 | D:\ | Stockage temporaire/pagefile.sys <br/><br/> Comme DB-Disk1 (D:) a été exclu, D: est la première lettre de lecteur disponible dans la liste.<br/><br/> Azure attribue donc la lettre D: au volume de stockage temporaire.<br/><br/> Étant donné que D: est disponible, le paramètre du fichier d’échange de la machine virtuelle reste le même.
DB-Disk2 | Disk2 | E:\ | Données utilisateur 1
DB-Disk3 | Disk3 | F:\ | Données utilisateur 2

Les paramètres du fichier d’échange sur la machine virtuelle Azure sont les suivants :

![Paramètres du fichier d’échange sur la machine virtuelle Azure](./media/exclude-disks-replication/pagefile-azure-vm-after-failover.png)

### <a name="paging-file-on-another-drive-not-d"></a>Fichier d’échange sur un autre lecteur (autre que D:)

Examinons un exemple dans lequel le fichier d’échange ne se trouve pas sur le lecteur D.  

Nous avons ces disques sur la machine virtuelle source.

**Nom du disque** | **Disque de système d’exploitation invité** | **Lettre de lecteur** | **Type de données de disque**
--- | --- | --- | ---
DB-Disk0-OS | Disk0 | C:\ | Disque de système d’exploitation
DB-Disk1 (à exclure de la réplication) | Disk1 | G:\ | pagefile.sys
DB-Disk2 | Disk2 | E:\ | Données utilisateur 1
DB-Disk3 | Disk3 | F:\ | Données utilisateur 2

Les paramètres du fichier d’échange sur la machine virtuelle locale sont les suivants :

![Paramètres du fichier d’échange sur la machine virtuelle locale](./media/exclude-disks-replication/pagefile-g-drive-source-vm.png)

1. Nous activons la réplication pour la machine virtuelle.
2. Nous excluons DB-Disk1 de la réplication.

#### <a name="disks-after-failover"></a>Disques après le basculement

Après le basculement, les disques de la machine virtuelle Azure sont décrits dans le tableau.

**Nom du disque** | **N° de disque de SE invité** | **Lettre de lecteur** | **Type de données de disque**
--- | --- | --- | ---
DB-Disk0-OS | Disk0  |C:\ | Disque de système d’exploitation
DB-Disk1 | Disk1 | D:\ | Stockage temporaire<br/><br/> Étant donné que D: est la première lettre de lecteur disponible dans la liste, Azure attribue la lettre D: au volume de stockage temporaire.<br/><br/> Pour tous les disques répliqués, la lettre de lecteur reste la même.<br/><br/> Étant donné que le lecteur G: n’est pas disponible, le système utilisera le lecteur C: pour le fichier d’échange.
DB-Disk2 | Disk2 | E:\ | Données utilisateur 1
DB-Disk3 | Disk3 | F:\ | Données utilisateur 2

Les paramètres du fichier d’échange sur la machine virtuelle Azure sont les suivants :

![Capture d’écran de la boîte de dialogue Mémoire virtuelle avec la ligne Lecteur D : en surbrillance montrant un paramètre de Taille de fichier d’échange « Gérée par le système ».](./media/exclude-disks-replication/pagefile-azure-vm-after-failover-2.png)


## <a name="next-steps"></a>Étapes suivantes

- Découvrez plus en détail les instructions pour le disque de stockage temporaire :
    - [Découvrez](https://cloudblogs.microsoft.com/sqlserver/2014/09/25/using-ssds-in-azure-vms-to-store-sql-server-tempdb-and-buffer-pool-extensions/) l’utilisation des disques SSD dans les machines virtuelles Azure pour stocker les extensions du pool de mémoires tampons et TempDB SQL Server
    - [Passez en revue](../azure-sql/virtual-machines/windows/performance-guidelines-best-practices.md) les bonnes pratiques sur les performances de SQL Server dans les machines virtuelles Azure.
- Une fois votre déploiement configuré et effectué, pour en savoir plus sur les différents types de basculement, [cliquez ici](failover-failback-overview.md) .
