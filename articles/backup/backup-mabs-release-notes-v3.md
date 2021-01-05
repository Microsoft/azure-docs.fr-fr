---
title: Notes de publication du serveur de sauvegarde Microsoft Azure v3
description: Cet article décrit les informations relatives aux problèmes connus et les solutions de contournement pour le serveur de sauvegarde Microsoft Azure (MABS) v3.
ms.topic: conceptual
ms.date: 06/03/2020
ms.asset: 0c4127f2-d936-48ef-b430-a9198e425d81
ms.openlocfilehash: 87bc415c125a387d98ac88255d77fb1867564acf
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91254259"
---
# <a name="release-notes-for-microsoft-azure-backup-server"></a>Notes de publication du serveur de sauvegarde Microsoft Azure

Cet article décrit les problèmes connus et les solutions de contournement pour le serveur de sauvegarde Microsoft Azure (MABS) V3.

## <a name="backup-and-recovery-fails-for-clustered-workloads"></a>Échecs de sauvegarde et de récupération pour les charges de travail en cluster

**Description :** échecs de la sauvegarde/restauration pour les sources de données en cluster, comme le cluster Hyper-V ou le cluster SQL (SQL Always On), ou Exchange dans le groupe de disponibilité de base de données après la mise à niveau de MABS V2 vers V3.

**Solution de contournement :** pour éviter ce problème, ouvrez SQL Server Management Studio (SSMS) et exécutez le script SQL suivant sur la base de données DPM :

```sql
    IF EXISTS (SELECT * FROM dbo.sysobjects
        WHERE id = OBJECT_ID(N'[dbo].[tbl_PRM_DatasourceLastActiveServerMap]')
        AND OBJECTPROPERTY(id, N'IsUserTable') = 1)
        DROP TABLE [dbo].[tbl_PRM_DatasourceLastActiveServerMap]
        GO

        CREATE TABLE [dbo].[tbl_PRM_DatasourceLastActiveServerMap] (
            [DatasourceId]          [GUID]          NOT NULL,
            [ActiveNode]            [nvarchar](256) NULL,
            [IsGCed]                [bit]           NOT NULL
            ) ON [PRIMARY]
        GO

        ALTER TABLE [dbo].[tbl_PRM_DatasourceLastActiveServerMap] ADD
    CONSTRAINT [pk__tbl_PRM_DatasourceLastActiveServerMap__DatasourceId] PRIMARY KEY NONCLUSTERED
        (
            [DatasourceId]
        )  ON [PRIMARY],

    CONSTRAINT [DF_tbl_PRM_DatasourceLastActiveServerMap_IsGCed] DEFAULT
        (
            0
        ) FOR [IsGCed]
    GO
```

## <a name="upgrade-to-mabs-v3-fails-in-russian-locale"></a>La mise à niveau vers MABS V3 échoue dans les paramètres régionaux russes

**Description :** la mise à niveau de MABS V2 vers V3 dans les paramètres régionaux russes échoue avec le code d’erreur **4387**.

**Solution de contournement :** effectuez les étapes suivantes pour mettre à niveau vers MABS V3 avec le package d’installation russe :

1. [Sauvegardez](/sql/relational-databases/backup-restore/create-a-full-database-backup-sql-server#SSMSProcedure) votre base de données SQL et désinstallez MABS V2 (choisissez de conserver les données protégées pendant la désinstallation).
2. Mettez à niveau vers SQL 2017 (Entreprise) et désinstallez Reporting dans le cadre de la mise à niveau.
3. [Installez](/sql/reporting-services/install-windows/install-reporting-services#install-your-report-server) SQL Server Reporting Services (SSRS).
4. [Installez](/sql/ssms/download-sql-server-management-studio-ssms) SQL Server Management Studio (SSMS).
5. Configurer Reporting à l’aide des paramètres indiqués dans [Configuration de SSRS avec SQL 2017](./backup-azure-microsoft-azure-backup.md#upgrade-mabs).
6. [Installez](backup-azure-microsoft-azure-backup.md) MABS V3.
7. [Restaurez](/sql/relational-databases/backup-restore/restore-a-database-backup-using-ssms) SQL à l’aide de SSMS et exécutez l’outil de synchronisation DPM comme décrit [ici](/system-center/dpm/back-up-the-dpm-server#using-dpmsync).
8. Mettez à jour la propriété « DataBaseVersion » dans la table dbo.tbl_DLS_GlobalSetting à l’aide de la commande suivante :

    ```sql
            UPDATE dbo.tbl_DLS_GlobalSetting
            set PropertyValue = '13.0.415.0'
            where PropertyName = 'DatabaseVersion'
    ```

9. Démarrez le service MSDPM.

## <a name="after-installing-ur1-the-mabs-reports-arent-updated-with-new-rdl-files"></a>Après l’installation d’UR1, les rapports MABS ne sont pas mis à jour avec les nouveaux fichiers RDL

**Description** : Avec UR1, le problème de mise en forme du rapport MABS est résolu grâce à la mise à jour des fichiers RDL. Les nouveaux fichiers RDL ne sont pas automatiquement remplacés par les fichiers existants.

**Solution de contournement** : Pour remplacer les fichiers RDL, procédez comme suit :

1. Sur l’ordinateur MABS, ouvrez l’URL du portail web de SQL Reporting Services.
1. Sur l’URL du portail web, le dossier DPMReports est présent au format **`DPMReports_<GUID>`** .

    >[!NOTE]
    >Il n’existe toujours qu’un seul dossier avec cette convention d’affectation de noms. Si MABS est mis à niveau à partir d’une version antérieure, il peut également y avoir un autre dossier plus ancien, mais vous ne pourrez pas l’ouvrir.

    ![Dossier DPMReports](./media/backup-mabs-release-notes-v3/dpm-reports-folder.png)

1. Sélectionnez et ouvrez le dossier **`DPMReports_<GUID>`** . Les fichiers de rapport individuels sont répertoriés comme indiqué ci-dessous.

    ![Liste des fichiers de rapport individuels](./media/backup-mabs-release-notes-v3/individual-report-files.png)

1. Sélectionnez les fichiers de rapport qui ne se terminent pas par **Report**, cliquez avec le bouton droit sur **Option**, puis sélectionnez **Gérer**.

    ![Sélectionner Gérer pour les fichiers de rapport](./media/backup-mabs-release-notes-v3/manage-files.png)

1. Dans la nouvelle page, sélectionnez l’option **Remplacer** pour remplacer les fichiers par les fichiers de rapport les plus récents.

    Les fichiers de rapport les plus récents se trouvent sous le chemin d’accès `<MABS Installation Directory>\Program Files\Microsoft Azure Backup Server\DPM\DPM\bin\DpmReports`.

    Par exemple : `C:\Program Files\Microsoft Azure Backup Server\DPM\DPM\bin\DpmReports`

    ![Remplacer les fichiers par les fichiers de rapport les plus récents](./media/backup-mabs-release-notes-v3/replace-files.png)

    Une fois les fichiers remplacés, assurez-vous que les champs **Nom** et **Description** sont intacts et ne sont pas vides.

1. Une fois les fichiers remplacés, redémarrez les services MABS et utilisez les fichiers de rapport.

## <a name="next-steps"></a>Étapes suivantes

[Nouveautés dans MABS](backup-mabs-whats-new-mabs.md)
