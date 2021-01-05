---
title: Sauvegarder le serveur MABS
description: Découvrez comment sauvegarder le serveur de sauvegarde Microsoft Azure (MABS).
ms.topic: conceptual
ms.date: 09/24/2020
ms.openlocfilehash: 81a6ee005e15b1d7ab7b11a938b8ab14143818f4
ms.sourcegitcommit: 2989396c328c70832dcadc8f435270522c113229
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92172113"
---
# <a name="back-up-the-mabs-server"></a>Sauvegarder le serveur MABS

Pour vous assurer que les données peuvent être récupérées en cas d’échec du serveur de sauvegarde Microsoft Azure (MABS), vous avez besoin d’une stratégie de sauvegarde du serveur MABS. S’il n’est pas sauvegardé, vous devez le reconstruire manuellement après une défaillance, et les points de récupération sur disque ne sont pas récupérables. Vous pouvez sauvegarder les serveurs MABS en sauvegardant la base de données MABS.

## <a name="back-up-the-mabs-database"></a>Sauvegarder la base de données MABS

Dans le cadre de votre stratégie de sauvegarde MABS, vous devez sauvegarder la base de données MABS. La base de données MABS est nommée DPMDB. Cette base de données contient la configuration MABS avec des données sur les sauvegardes de MABS. En cas d’incident, vous pouvez reconstruire la plupart des fonctionnalités d’un serveur MABS à l’aide d’une sauvegarde récente de la base de données. Si vous pouvez restaurer la base de données, des sauvegardes sur bande sont accessibles, et tous les paramètres de groupe de protection ainsi que les planifications de sauvegarde sont conservés. Si les disques du pool de stockage MABS n’ont pas été affectés par la panne, les sauvegardes sur disque sont également utilisables après une reconstruction. Vous pouvez sauvegarder la base de données à l’aide de différentes méthodes.

|Méthode de sauvegarde de base de données|Avantages|Inconvénients|
|--------------------------|--------------|-----------------|
|[Sauvegarder sur Azure](#back-up-to-azure)|<li>Facilement configuré et surveillé dans MABS.<li>Fichiers de la base de données de sauvegarde disponibles en plusieurs emplacements.<li>Le stockage dans le cloud fournit une solution robuste pour la récupération d'urgence.<li>Stockage très sûr pour la base de données.<li>Prend en charge 120 points de récupération en ligne.|<li>Nécessite un compte Azure et une configuration MABS supplémentaire. Implique un coût pour le stockage Azure.<li> Requiert une version prise en charge du système Windows Server avec l’agent Azure pour accéder aux sauvegardes MABS stockées dans le coffre de sauvegarde Azure. Il ne peut pas s’agir d’un autre serveur MABS.<li>Cette option est inappropriée si la base de données est hébergée localement et si vous souhaitez activer une protection secondaire. <li>Cela implique un temps supplémentaire de préparation et récupération.|
|[Sauvegarder la base de données en sauvegardant le pool de stockage MABS](#back-up-the-database-by-backing-up-the-mabs-storage-pool)|<li>Simple à configurer et à surveiller.<li>La sauvegarde est conservée sur les disques du pool de stockage MABS et est facile à accéder localement.<li>Les sauvegardes planifiées MABS prennent en charge les sauvegardes complètes rapides 512. Si vous sauvegardez toutes les heures, vous bénéficiez de 21 jours de protection complète.|<li>Cette option ne convient pas pour la récupération d'urgence. En ligne, la récupération peut ne pas fonctionner comme prévu si le disque du pool de stockage ou le serveur MABS échoue.<li>Cette option est inappropriée si la base de données est hébergée localement et si vous souhaitez activer une protection secondaire. <li>Une préparation et des étapes spéciales sont nécessaires pour accéder aux points de récupération si le service ou la console MABS ne s’exécutent pas ou ne fonctionnent pas.|
|[Sauvegarder avec une sauvegarde SQL Server en mode natif sur un disque local](#back-up-with-native-sql-server-backup-to-a-local-disk)|<li>Intégrée à SQL Server.<li>La sauvegarde est conservée sur un disque local qui est facilement accessible.<li>Vous pouvez planifier son exécution aussi souvent que vous le souhaitez.<li>Totalement indépendant de MABS.<li>Vous pouvez planifier un nettoyage de fichiers de sauvegarde.|<li>Cette option ne convient pas pour la récupération d'urgence, à moins que les sauvegardes soient copiées dans un emplacement distant.<li>Nécessite un stockage local pour les sauvegardes, ce qui peut limiter la rétention et la fréquence.|
|[Sauvegarde avec sauvegarde SQL native et protection MABS sur un partage protégé par MABS](#back-up-to-a-share-protected-by-mabs)|<li>Facile à surveiller dans MABS.<li>Fichiers de la base de données de sauvegarde disponibles en plusieurs emplacements.<li>Accès aisé à partir de tout ordinateur Windows sur le réseau.<li>Méthode de récupération probablement la plus rapide.|<li>Prend en charge au maximum 64 points de récupération.<li>Cette option ne convient pas pour une récupération d'urgence de site. Une défaillance de disque du pool de stockage MABS ou du serveur MABS peut entraver les efforts de récupération.<li>Ce n’est pas une option si la base de données MABS est hébergée localement et que vous souhaitez activer la protection secondaire. <li>Une préparation supplémentaire est nécessaire pour la configuration et le test.<li>Une préparation et un temps de récupération supplémentaires sont nécessaires si le serveur MABS lui-même est défaillant, mais que les disques du pool de stockage MABS sont corrects.|

- Si vous sauvegardez à l’aide d’un groupe de protection MABS, nous vous recommandons d'utiliser un groupe de protection unique pour la base de données.

    > [!NOTE]
    > Pour des besoins de restauration, l’installation MABS que vous voulez restaurer avec la base de données MABS doit correspondre à la version de la base de données MABS elle-même.  Par exemple, si la base de données à récupérer est issue d’une installation MABS V3 avec le correctif cumulatif 1, le serveur MABS doit exécuter la même version avec le correctif cumulatif 1. Autrement dit, avant de restaurer la base de données, vous serez peut-être amené à désinstaller et réinstaller MABS avec une version compatible.  Pour vérifier la version de la base de données, vous devrez peut-être la monter manuellement avec un nom de base de données temporaire et exécuter une requête SQL sur la base de données pour consulter le dernier correctif installé, en fonction des versions majeure et mineure.

- Pour vérifier la version de la base de données MABS, procédez comme suit :

    1. Pour exécuter la requête, ouvrez SQL Management Studio, puis connectez-vous à l'instance SQL exécutant la base de données MABS.

    2. Sélectionnez la base de données MABS, puis démarrez une nouvelle requête.

    3. Collez la requête SQL suivante dans le volet de requête, puis exécutez la requête :

        `Select distinct MajorVersionNumber,MinorVersionNumber ,BuildNumber, FileName FROM dbo.tbl\_AM\_AgentPatch order byMajorVersionNumber,MinorVersionNumber,BuildNumber`

    Si aucune valeur n’est retournée dans les résultats de la requête, ou si le serveur MABS a été mis à niveau à partir de versions précédentes mais qu’aucun nouveau correctif cumulatif n’a été installé depuis, il n’y a pas d’entrée pour le principal, mineur pour une installation de base de MABS. Pour vérifier les versions MABS associées aux correctifs cumulatifs, consultez la [Liste des numéros de build pour MABS](https://social.technet.microsoft.com/wiki/contents/articles/36381.microsoft-azure-backup-server-list-of-build-numbers.aspx).

### <a name="back-up-to-azure"></a>Sauvegarde sur Azure

1. Avant de commencer, vous devez exécuter un script pour récupérer le chemin d’accès du point de montage du volume du réplica MABS afin de savoir quel point de récupération contient la sauvegarde MABS. Faites-le après la réplication initiale avec Sauvegarde Azure. Dans le script, remplacez `dplsqlservername%` par le nom de l’instance SQL Server hébergeant la base de données MABS.

    ```SQL
    Select ag.NetbiosName as ServerName,ds.DataSourceName,vol.MountPointPath
    from tbl_IM_DataSource as ds
    join tbl_PRM_LogicalReplica as lr on ds.DataSourceId=lr.DataSourceId
    join tbl_AM_Server as ag on ds.ServerId=ag.ServerId
    join tbl_SPM_Volume as vol on lr.PhysicalReplicaId=vol.VolumeSetID
    and vol.Usage =1
    and lr.Validity in (1,2)
    where ds.datasourcename like '%dpmdb%'
    and servername like '%dpmsqlservername%' --netbios name of server hosting DPMDB
    ```

    Vérifiez que vous disposez du code secret spécifié lors de l’installation de l’agent Azure Recovery Services et que le serveur MABS a été inscrit dans le coffre de Sauvegarde Azure. Vous avez besoin de ce code secret pour restaurer la sauvegarde.

2. Créez un coffre de sauvegarde Azure, téléchargez le fichier d’installation et les informations d’identification du coffre de l’agent de sauvegarde Azure. Exécutez le fichier d’installation pour installer l’agent sur le serveur MABS et utilisez les informations d’identification du coffre pour inscrire le serveur MABS dans le coffre. [Plus d’informations](backup-azure-microsoft-azure-backup.md)

3. Une fois le coffre configuré, configurez un groupe de protection MABS contenant la base de données MABS. Sélectionnez cette option pour la sauvegarder sur disque et sur Azure.

#### <a name="recover-the-mabs-database-from-azure"></a>Récupérer la base de données MABS à partir d’Azure

Vous pouvez récupérer la base de données à partir d’Azure à l’aide d’un serveur MABS inscrit dans le coffre de Sauvegarde Azure, comme suit :

1. Dans la console MABS, sélectionnez **Récupérer** > **Ajouter un MABS externe**.

2. Fournissez les informations d’identification du coffre (téléchargement à partir du coffre de Sauvegarde Azure). Notez que les informations d’identification sont valides uniquement pendant deux jours.

3. Dans **Sélectionner un MABS externe pour la récupération**, sélectionnez le serveur MABS pour lequel vous souhaitez récupérer la base de données, tapez la phrase secrète de chiffrement, puis sélectionnez **OK.**

4. Sélectionnez le point de récupération à utiliser dans la liste des points disponibles. Sélectionnez **Effacer les MABS externes** pour revenir à l’affichage local MABS.

## <a name="back-up-the-mabs-database-to-mabs-storage-pool"></a>Sauvegarder la base de données MABS dans le pool de stockage MABS

> [!NOTE]  
> Cette option s’applique à MABS avec stockage de sauvegarde moderne.

1. Dans la console MABS, sélectionnez **Protection** > **Créer un groupe de protection**.
2. Dans la page **Sélectionner le type de groupe de protection** , sélectionnez **Serveurs**.
3. Dans la page **Sélectionner les membres du groupe**, sélectionnez la **base de données DPM**. Développez le serveur MABS et sélectionnez DPMDB.
4. Dans la page **Sélectionner la méthode de protection des données**, sélectionnez **Je souhaite une protection à court terme sur disque**. Spécifiez les options de stratégie de protection à court terme.
5. Après la réplication initiale de la base de données MABS, exécutez le script SQL suivant :

```SQL
select AG.NetbiosName, DS.DatasourceName, V.AccessPath, LR.PhysicalReplicaId from tbl_IM_DataSource DS
join tbl_PRM_LogicalReplica as LR
on DS.DataSourceId = LR.DataSourceId
join tbl_AM_Server as AG
on DS.ServerId=AG.ServerId
join tbl_PRM_ReplicaVolume RV
on RV.ReplicaId = LR.PhysicalReplicaId
join tbl_STM_Volume V
on RV.StorageId = V.StorageId
where datasourcename like N'%dpmdb%' and ds.ProtectedGroupId is not null
and LR.Validity in (1,2)
and AG.ServerName like N'%<dpmsqlservername>%' -- <dpmsqlservername> is a placeholder, put netbios name of server hosting DPMDB
```

### <a name="recover-mabs-database"></a>Récupérer la base de données MABS

Pour reconstruire votre MABS avec la même base de données, vous devez d’abord récupérer la base de données MABS et la synchroniser avec le MABS récemment installé.

#### <a name="use-the-following-steps"></a>Utiliser les étapes suivantes

1. Ouvrez une invite de commandes d’administration et exécutez `psexec.exe -s powershell.exe` pour démarrer une fenêtre PowerShell dans le contexte système.
2. Indiquez l’emplacement à partir duquel vous voulez récupérer la base de données :

#### <a name="to-copy-the-database-from-the-last-backup"></a>Pour copier la base de données à partir de la dernière sauvegarde

1. Accédez au chemin du VHD de réplica `\<MABSServer FQDN\>\<PhysicalReplicaId\>\<PhysicalReplicaId\>`
2. Montez le **disk0.vhdx** qui y est présent à l’aide de la commande `mount-vhd disk0.vhdx`.
3. Une fois le disque dur virtuel de réplication monté, utilisez `mountvol.exe` pour affecter une lettre de lecteur au volume du réplica à l’aide de l’ID de réplica physique de la sortie du script SQL. Par exemple : `mountvol X: \?\Volume{}\`

#### <a name="to-copy-the-database-from-a-previous-recovery-point"></a>Pour copier la base de données à partir d’un point de récupération précédent

1. Accédez au répertoire de conteneur DPMDB `\<MABSServer FQDN\>\<PhysicalReplicaId\>`. Vous verrez plusieurs répertoires avec des identificateurs GUID uniques en-dessous des points de récupération correspondants pour la base de données MABS. Les autres répertoires représentent un point de récupération ou un PIT.
2. Accédez à n’importe quel chemin d’accès de disque dur virtuel PIT, par exemple `\<MABSServer FQDN\>\<PhysicalReplicaId\>\<PITId\>` et montez le **disk0.vhdx** qui est présent à l’aide de la commande `mount-vhd disk0.vhdx`.
3. Une fois le disque dur virtuel de réplication monté, utilisez `mountvol.exe` pour affecter une lettre de lecteur au volume du réplica à l’aide de l’ID de réplica physique de la sortie du script SQL. Par exemple : `mountvol X: \?\Volume{}\`

   Tous les termes qui apparaissent avec des accolades angulaires dans les étapes ci-dessus sont des espaces réservés. Remplacez-les par les valeurs appropriées comme suit :
   - **ReFSVolume** : chemin d’accès à partir de la sortie du script SQL
   - **FQDN MABSServer** : nom complet du serveur MABS
   - **PhysicalReplicaId** : ID de réplica physique du script SQL
   - **PITId** : identificateur GUID autre que l’ID de réplica physique dans le répertoire de conteneur.
4. Ouvrez une autre invite de commandes d’administration et exécutez `psexec.exe -s cmd.exe` pour démarrer une invite de commandes dans le contexte système.
5. Remplacez le répertoire par le lecteur X : et accédez à l’emplacement des fichiers de base de données MABS.
6. Copiez-les à un emplacement facile à utiliser pour une restauration. Quittez la fenêtre cmd psexec une fois la copie terminée.
7. Accédez à la fenêtre de psexec PowerShell ouverte à l’étape 1, accédez au chemin VHDX et démontez le VHDX à l’aide de la commande `dismount-vhd disk0.vhdx`.
8. Après la réinstallation du serveur MABS, vous pouvez utiliser le DPMDB restauré pour l’attacher au serveur MABS en exécutant la commande `DPMSYNC-RESTOREDB`.
9. Exécutez `DPMSYNC-SYNC` une fois que la commande `DPMSYNC-RESTOREDB` a été exécutée.

### <a name="back-up-the-database-by-backing-up-the-mabs-storage-pool"></a>Sauvegarder la base de données en sauvegardant le pool de stockage MABS

> [!NOTE]
> Cette option s’applique à MABS avec stockage hérité.

Avant de commencer, vous devez exécuter un script pour récupérer le chemin d’accès du point de montage du volume du réplica MABS afin de savoir quel point de récupération contient la sauvegarde MABS. Faites-le après la réplication initiale avec Sauvegarde Azure. Dans le script, remplacez `dplsqlservername%` par le nom de l’instance SQL Server hébergeant la base de données MABS.

```SQL
Select ag.NetbiosName as ServerName,ds.DataSourceName,vol.MountPointPath
from tbl_IM_DataSource as ds
join tbl_PRM_LogicalReplica as lr on ds.DataSourceId=lr.DataSourceId
join tbl_AM_Server as ag on ds.ServerId=ag.ServerId
join tbl_SPM_Volume as vol on lr.PhysicalReplicaId=vol.VolumeSetID
and vol.Usage =1
and lr.Validity in (1,2)
where ds.datasourcename like '%dpmdb%'
and servername like '%dpmsqlservername%' --netbios name of server hosting DPMDB
```

1. Dans la console MABS, sélectionnez **Protection** > **Créer un groupe de protection**.

2. Dans la page **Sélectionner le type de groupe de protection**, sélectionnez **Serveurs**.

3. Dans la page **Sélectionner les membres du groupe**, sélectionnez la base de données MABS. Développez l’élément de serveur MABS et sélectionnez **DPMDB**.

4. Dans la page **Sélectionner la méthode de protection des données**, sélectionnez **Protection à court terme sur disque**. Spécifiez les options de stratégie de protection à court terme. Nous vous recommandons une durée de rétention de deux semaines pour les bases de données MABS.

#### <a name="recover-the-database"></a>Récupération de la base de données

Si le serveur MABS est toujours opérationnel et que le pool de stockage est intact (par exemple, des problèmes avec le service ou la console MABS), copiez la base de données à partir du volume de réplica ou d’un cliché instantané comme suit :

1. Indiquez à partir de quand vous voulez récupérer la base de données.

    - Si vous voulez copier la base de données à partir de la dernière sauvegarde effectuée directement à partir du volume du réplica MABS, exécutez **mountvol.exe** pour attribuer une lettre de lecteur au volume du réplica en utilisant le GUID de la sortie du script SQL. Par exemple : `C:\Mountvol X: \\?\Volume{d7a4fd76\-a0a8\-11e2\-8fd3\-001c23cb7375}\`

    - Si vous souhaitez copier la base de données à partir d’un point de récupération (cliché instantané) précédent, vous devez répertorier tous les clichés instantanés du réplica en utilisant le GUID de volume de la sortie du script SQL. Cette commande répertorie les clichés instantanés pour ce volume : `C:\>Vssadmin list shadows /for\=\\?\Volume{d7a4fd76-a0a8-11e2-8fd3-001c23cb7375}\`. Notez l’heure de création et l’ID du cliché instantané à partir desquels vous souhaitez effectuer la récupération.

2. Utilisez à présent **diskshadow.exe** pour monter le cliché instantané sur une lettre de lecteur inutilisé X : en vous servant de l’ID du cliché instantané pour copier les fichiers de base de données.

3. Ouvrez une invite de commandes d'administration et exécutez `psexec.exe -s cmd.exe` pour démarrer une invite de commandes dans le contexte système qui vous permettra d'être autorisé à naviguer dans le volume du réplica (X:).

4. CD vers X : accédez à l’emplacement des fichiers de base de données MABS. Copiez-les à un emplacement facile à utiliser pour une restauration. Une fois la copie terminée, fermez la fenêtre psexec cmd, puis exécutez **diskshadow.exe** et cessez d’exposer le volume X:.

5. Vous pouvez à présent restaurer les fichiers de base de données à l’aide de SQL Management Studio ou en exécutant la commande **DPMSYNC\-RESTOREDB**.

## <a name="back-up-with-native-sql-server-backup-to-a-local-disk"></a>Sauvegarde SQL Server en mode natif sur disque local

Vous pouvez sauvegarder la base de données MABS sur un disque local avec la sauvegarde SQL Server native, indépendamment de MABS.

- Obtenez un [aperçu](/sql/relational-databases/backup-restore/back-up-and-restore-of-sql-server-databases) de la sauvegarde SQL Server.

- [En savoir plus](/sql/relational-databases/backup-restore/sql-server-backup-and-restore-with-microsoft-azure-blob-storage-service) sur la sauvegarde de SQL Server dans le cloud.

## <a name="back-up-to-a-share-protected-by-mabs"></a>Sauvegarder sur un partage protégé par MABS

Cette option de sauvegarde utilise SQL natif pour enregistrer la base de données MABS sur un partage, protège le partage avec MABS et utilise les versions précédentes de Windows VSS pour faciliter la restauration.

### <a name="before-you-start"></a>Avant de commencer

1. Sur SQL Server, créez un dossier sur un lecteur disposant d’un espace libre suffisant pour contenir une copie unique d’une sauvegarde. Par exemple : `C:\MABSBACKUP`.

1. Partagez le dossier. Par exemple, partagez le dossier `C:\MABSBACKUP` en tant que *DPMBACKUP*.

1. Copiez et collez la commande OSQL ci-dessous dans le Bloc-notes, puis enregistrez-la dans un fichier nommé `C:\MABSACKUP\bkupdb.cmd`. Vérifiez qu’il n’y a pas d’extension .txt. Modifiez les noms SQL_Instance_name et DPMDB_NAME pour qu'ils correspondent à l'instance et au nom de DPMDB utilisés par votre serveur MABS.

    ```SQL
    OSQL -E -S localhost\SQL_INSTANCE_NAME -Q "BACKUP DATABASE DPMDB_NAME TO DISK='C:\DPMBACKUP\dpmdb.bak' WITH FORMAT"
    ```

1. Dans le Bloc-notes, ouvrez le fichier **ScriptingConfig.xml** situé dans le dossier `C:\Program Files\Microsoft System Center\DPM\DPM\Scripting` sur le serveur MABS.

1. Modifiez **ScriptingConfig.xml** et remplacez **DataSourceName=** par la lettre du lecteur contenant le dossier/partage DPMDBBACKUP. Remplacez l’entrée PreBackupScript par le chemin d’accès et le nom complets de **bkupdb.cmd** enregistré à l’étape 3.

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <ScriptConfiguration xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:xsd="http://www.w3.org/2001/XMLSchema"
    xmlns="https://schemas.microsoft.com/2003/dls/ScriptingConfig.xsd">
    <DatasourceScriptConfig DataSourceName="C:">
    <PreBackupScript>C:\MABSDBBACKUP\bkupdb.cmd</PreBackupScript>
    <TimeOut>120</TimeOut>
    </DatasourceScriptConfig>
    </ScriptConfiguration>
    ```

1. Enregistrez les modifications dans **ScriptingConfig.xml**.

1. Protégez le dossier C:\MABSBACKUP ou le partage `\sqlservername\MABSBACKUP` à l’aide de MABS, puis attendez que le réplica initial soit créé. Suite de l’exécution du script de présauvegarde, le dossier C:\MABSBACKUP doit contenir un fichier **dpmdb.bak** qui a été copié dans le réplica DPM.

1. Si vous n’activez pas la récupération libre service, vous devez exécuter des étapes supplémentaires pour partager le dossier MABSBACKUP sur le réplica :

    1. Dans la console MABS > **Protection**, recherchez la source de données MABSBACKUP, puis sélectionnez-la. Dans la section des détails, sélectionnez **Cliquez pour afficher les détails** sur le lien du chemin d'accès du réplica, puis copiez le chemin d’accès dans le Bloc-notes. Supprimez le chemin d'accès de la source et conservez le chemin d'accès de la destination. Le chemin d’accès doit ressembler à `C:\Program Files\Microsoft System Center\DPM\DPM\Volumes\Replica\File System\vol_c9aea05f-31e6-45e5-880c-92ce5fba0a58\454d81a0-0d9d-4e07-9617-d49e3f2aa5de\Full\DPMBACKUP`.

    2. Créez un partage vers ce chemin d’accès en utilisant le nom de partage **MABSSERVERNAME-DPMDB**. Vous pouvez utiliser la commande Net Share ci-dessous à partir d'une invite de commandes d'administration.

        ```cmd
        Net Share MABSSERVERNAME-dpmdb="C:\Program Files\Microsoft System Center\DPM\DPM\Volumes\Replica\File System\vol_c9aea05f-31e6-45e5-880c-92ce5fba0a58\454d81a0-0d9d-4e07-9617-d49e3f2aa5de\Full\DPMBACKUP"
        ```

### <a name="configure-the-backup"></a>Configuration de la sauvegarde

Vous pouvez sauvegarder la base de données MABS comme vous le feriez pour n’importe quelle autre base de données SQL Server à l’aide d’une sauvegarde SQL Server native.

- Obtenez un [aperçu](/sql/relational-databases/backup-restore/back-up-and-restore-of-sql-server-databases) de la sauvegarde SQL Server.

- [En savoir plus](/sql/relational-databases/backup-restore/sql-server-backup-and-restore-with-microsoft-azure-blob-storage-service) sur la sauvegarde de SQL Server dans le cloud.

### <a name="recover-the-mabs-database"></a>Récupérer la base de données MABS

1. Connectez-vous au partage `\\MABSServer\MABSSERVERNAME-dpmdb` via Explorer d’un ordinateur Windows.

2. Cliquez avec le bouton droit sur le fichier **dpmdb.bak** pour afficher ses propriétés. L’onglet **Versions précédentes** affiche toutes les sauvegardes que vous pouvez sélectionner et copier. La toute dernière sauvegarde située dans le dossier C:\MABSBACKUP est également aisément accessible.

3. Si vous devez déplacer un disque de pool de stockage MABS attaché à un réseau SAN vers un autre serveur pour être en mesure de lire le volume du réplica, ou réinstaller Windows pour lire localement des disques attachés, vous devez connaître le chemin d'accès du point de montage du volume du réplica MABS ou le GUID de volume afin de savoir quel volume contient la sauvegarde de base de données. Vous pouvez utiliser le script SQL ci-dessous pour extraire ces informations à tout moment après la protection initiale, mais avant de restaurer. Remplacez `%dpmsqlservername%` par le nom du serveur SQL Server qui héberge la base de données.

    ```sql
    Select ag.NetbiosName as
    ServerName,ds.DataSourceName,vol.MountPointPath,vol.GuidName
    from tbl_IM_DataSource as ds
    join tbl_PRM_LogicalReplica as lr on ds.DataSourceId=lr.DataSourceId
    join tbl_AM_Server as ag on ds.ServerId=ag.ServerId
    join tbl_SPM_Volume as vol on lr.PhysicalReplicaId=vol.VolumeSetID
    and vol.Usage =1
    and lr.Validity in (1,2)
    where ds.datasourcename like '%C:\%' -- volume drive letter for DPMBACKUP
    and servername like '%dpmsqlservername%' --netbios name of server hosting DPMDB

    ```

4. Si vous devez effectuer une récupération après avoir déplacé les disques du pool de stockage MABS ou une reconstruction du serveur MABS :

    1. Vous disposez du GUID du volume. Par conséquent, si ce volume doit être monté sur un autre serveur Windows, ou après une régénération du serveur MABS, utilisez **mountvol.exe** pour lui attribuer une lettre de lecteur en utilisant le GUID du volume figurant dans la sortie du script SQL : `C:\Mountvol X: \\?\Volume{d7a4fd76-a0a8-11e2-8fd3-001c23cb7375}\`.

    2. Partagez à nouveau le dossier MABSBACKUP sur le volume du réplica à l’aide de la lettre du lecteur et de la partie du chemin d’accès du réplica qui représente la structure de dossiers.

        ```cmd
        net share SERVERNAME-DPMDB="X:\454d81a0-0d9d-4e07-9617-d49e3f2aa5de\Full\DPMBACKUP"
        ```

    3. Connectez-vous au partage `\\SERVERNAME\MABSSERVERNAME-dpmdb` via Explorer d’un ordinateur Windows.

    4. Cliquez avec le bouton droit sur le fichier **dpmdb.bak** pour afficher ses propriétés. L’onglet **Versions précédentes** affiche toutes les sauvegardes que vous pouvez sélectionner et copier.

## <a name="using-dpmsync"></a>Utilisation de DPMSync

**DpmSync** est un outil en ligne de commande qui vous permet de synchroniser la base de données MABS avec l’état des disques dans le pool de stockage et avec les agents de protection installés. DpmSync restaure la base de données MABS, synchronise la base de données MABS avec les réplicas dans le pool de stockage, restaure la base de données de rapports et réalloue les réplicas manquants.

### <a name="parameters"></a>Paramètres

| Paramètre      | Description    |
|----------------|-----------------------------|
| **-RestoreDb**                       | Restaure une base de données MABS depuis un emplacement spécifié.|
| **-Sync**                            | Synchronise les bases de données restaurées. Vous devez exécuter **DpmSync –Sync** après avoir restauré les bases de données. Après l’exécution de **DpmSync –Sync**, certains réplicas peuvent toujours être marqués comme manquants. |
| **-DbLoc** *emplacement*                | Identifie l’emplacement de la sauvegarde de la base de données MABS.|
| **-InstanceName** <br/>*server\instance*     | Instance vers laquelle la base de données DPM doit être restaurée.|
| **-ReallocateReplica**         | Réattribue tous les volumes de réplicas manquants sans synchronisation. |
| **-DataCopied**                      | Indique que vous avez terminé de charger les données dans les volumes de réplicas nouvellement alloués. Cela ne s’applique qu’aux ordinateurs clients. |

**Exemple 1 :** Pour restaurer la base de données MABS à partir d’un support de sauvegarde local sur le serveur MABS, exécutez la commande suivante :

```cmd
DpmSync –RestoreDb -DbLoc G:\DPM\Backups\2005\November\DPMDB.bak
```

Après avoir restauré la base de données MABS, exécutez la commande suivante pour synchroniser les bases de données :

```cmd
DpmSync -Sync
```

Après avoir restauré et synchronisé la base de données MABS et avant de restaurer le réplica, exécutez la commande suivante pour réallouer l’espace disque pour le réplica :

```cmd
DpmSync -ReallocateReplica
```

**Exemple 2 :** Pour restaurer la base de données MABS à partir d’une base de données distante, exécutez la commande suivante sur l’ordinateur distant :

```cmd
DpmSync –RestoreDb -DbLoc G:\DPM\Backups\2005\November\DPMDB.bak –InstanceName contoso\ms$dpm
```

Après avoir restauré la base de données MABS, exécutez la commande suivante sur un serveur MABS pour synchroniser les bases de données :

```cmd
DpmSync -Sync
```

Après avoir restauré et synchronisé la base de données MABS et avant de restaurer le réplica, exécutez la commande suivante sur le serveur MABS pour réallouer l’espace disque pour le réplica :

```cmd
DpmSync -ReallocateReplica
```

## <a name="next-steps"></a>Étapes suivantes

- [Matrice de prise en charge MABS](backup-support-matrix-mabs-dpm.md)
- [MABS FAQ](backup-azure-dpm-azure-server-faq.md)