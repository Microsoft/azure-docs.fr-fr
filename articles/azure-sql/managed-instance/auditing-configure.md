---
title: Audit SQL Managed Instance
description: Découvrez comment démarrer avec l’audit Azure SQL Managed Instance à l’aide de T-SQL
services: sql-database
ms.service: sql-managed-instance
ms.subservice: security
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
f1_keywords:
- mi.azure.sqlaudit.general.f1
author: DavidTrigano
ms.author: datrigan
ms.reviewer: vanto
ms.date: 05/26/2020
ms.openlocfilehash: 31a1169ca6c2194b8d5564e5d0df50116dd25084
ms.sourcegitcommit: 4bee52a3601b226cfc4e6eac71c1cb3b4b0eafe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94505663"
---
# <a name="get-started-with-azure-sql-managed-instance-auditing"></a>Bien démarrer avec l’audit Azure SQL Managed Instance
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

L’audit [Azure SQL Managed Instance](sql-managed-instance-paas-overview.md) suit les événements de base de données et les écrit dans un journal d’audit dans votre compte Stockage Azure. Par ailleurs, l’audit :

- peut vous aider à respecter une conformité réglementaire, à comprendre l’activité de la base de données ainsi qu’à découvrir des discordances et anomalies susceptibles d’indiquer des problèmes pour l’entreprise ou des violations de la sécurité ;
- permet et facilite le respect de normes de conformité, même s’il ne garantit pas cette conformité. Pour plus d’informations sur les programmes Azure qui prennent en charge la conformité aux normes, consultez le [Centre de confidentialité Azure](https://gallery.technet.microsoft.com/Overview-of-Azure-c1be3942) où vous trouverez la liste actualisée des certifications de conformité.

## <a name="set-up-auditing-for-your-server-to-azure-storage"></a>Configurer l’audit de votre serveur sur Stockage Azure

La section suivante décrit la configuration de l’audit à l’aide de votre instance Managed Instance.

1. Accédez au [portail Azure](https://portal.azure.com).
2. Créez un **conteneur** Stockage Azure où sont stockés les journaux d’audit.

   1. Accédez au compte Stockage Azure où vous souhaitez stocker vos journaux d’audit.

      > [!IMPORTANT]
      > - Utilisez un compte de stockage dans la même région que l’instance gérée afin d’éviter des lectures/écritures entre régions. 
      > - Si votre compte de stockage se trouve derrière un réseau virtuel ou un pare-feu, consultez [Accorder l’accès à partir d’un réseau virtuel](../../storage/common/storage-network-security.md#grant-access-from-a-virtual-network).
      > - Si vous remplacez la valeur 0 de la période de rétention (rétention illimitée) par une autre valeur, notez que la rétention s’appliquera uniquement aux journaux écrits après la modification de la valeur de rétention (les journaux écrits au cours de la période pendant laquelle la rétention était définie sur illimité sont conservés, même après activation de la rétention).

   1. Dans le compte de stockage, accédez à **Vue d’ensemble**, puis cliquez sur **Objets blob**.

      ![Widget Objets blob Azure](./media/auditing-configure/1_blobs_widget.png)

   1. Dans le menu supérieur, cliquez sur **+ Conteneur** pour créer un conteneur.

      ![Icône de création d’un conteneur d’objets blob](./media/auditing-configure/2_create_container_button.png)

   1. Indiquez un **Nom** pour le conteneur, définissez le **niveau d’accès Public** sur **Privé**, puis cliquez sur **OK**.

      ![Créer une configuration de conteneur d’objets blob](./media/auditing-configure/3_create_container_config.png)

    > [!IMPORTANT]
    > Les clients souhaitant configurer un magasin de journaux immuable pour leurs événements d’audit au niveau du serveur ou de la base de données doivent suivre les [instructions fournies par Stockage Azure](../../storage/blobs/storage-blob-immutability-policies-manage.md#enabling-allow-protected-append-blobs-writes). (Vérifiez que vous avez sélectionné **Autoriser les ajouts supplémentaires** quand vous configurez le stockage d’objets blob immuables.)
  
3. Après avoir créé le conteneur pour les journaux d’audit, vous pouvez le configurer comme cible pour les journaux d’activité de deux façons : [à l’aide de T-SQL](#blobtsql) ou [à l’aide de l’interface utilisateur de SSMS (SQL Server Management Studio)](#blobssms) :

   - <a id="blobtsql"></a>**Configurer le stockage Blob pour les journaux d’audit à l’aide de T-SQL :**

     1. Dans la liste des conteneurs, cliquez sur le conteneur que vous venez de créer, puis sur **Propriétés du conteneur**.

        ![Bouton Propriétés d’un conteneur d’objets blob](./media/auditing-configure/4_container_properties_button.png)

     1. Copiez l’URL du conteneur en cliquant sur l’icône Copier, puis enregistrez-la (par exemple, dans le Bloc-notes) pour l’utiliser ultérieurement. L’URL du conteneur doit être au format `https://<StorageName>.blob.core.windows.net/<ContainerName>`

        ![URL du conteneur d’objets blob à copier](./media/auditing-configure/5_container_copy_name.png)

     1. Générez un **jeton SAS** Stockage Azure pour accorder au compte de stockage des droits d’accès à l’audit d’instance managée :

        - Accédez au compte Stockage Azure dans lequel vous avez créé le conteneur à l’étape précédente.

        - Dans le menu **Paramètres de stockage**, cliquez sur **Signature d’accès partagé**.

          ![Icône Signature d’accès partagé dans le menu des paramètres de stockage](./media/auditing-configure/6_storage_settings_menu.png)

        - Configurez la signature d’accès partagé comme suit :

          - **Services autorisés** : Objet blob

          - **Date de début** : pour éviter tout problème lié au fuseau horaire, utilisez la date de la veille.

          - **Date de fin** : choisissez la date à laquelle ce jeton SAS arrive à expiration.

            > [!NOTE]
            > À l’expiration, renouvelez le jeton afin d’éviter les échecs d’audit.

          - Cliquez sur **Générer une signature d’accès partagé**.

            ![Configuration de SAS](./media/auditing-configure/7_sas_configure.png)

        - Le jeton SAS apparaît en bas. Copiez le jeton en cliquant sur l’icône Copier, puis enregistrez-le (par exemple, dans le Bloc-notes) pour l’utiliser ultérieurement.

          ![Copier le jeton SAS](./media/auditing-configure/8_sas_copy.png)

          > [!IMPORTANT]
          > Supprimez le point d’interrogation (« ? ») au début du jeton.

     1. Connectez-vous à votre instance managée à l’aide de SQL Server Management Studio ou de tout autre outil pris en charge.

     1. Exécutez l’instruction T-SQL suivante pour **créer des informations d’identification** à l’aide de l’URL du conteneur et du jeton SAS créés lors des étapes précédentes :

        ```SQL
        CREATE CREDENTIAL [<container_url>]
        WITH IDENTITY='SHARED ACCESS SIGNATURE',
        SECRET = '<SAS KEY>'
        GO
        ```

     1. Exécutez l’instruction T-SQL suivante pour créer un audit de serveur (choisissez votre propre nom d’audit et utilisez l’URL du conteneur que vous avez créée lors des étapes précédentes). Si aucune valeur n’est spécifiée, la valeur par défaut de `RETENTION_DAYS` est 0 (conservation illimitée) :

        ```SQL
        CREATE SERVER AUDIT [<your_audit_name>]
        TO URL ( PATH ='<container_url>' , RETENTION_DAYS =  integer )
        GO
        ```

     1. Continuez en [créant une spécification d’audit du serveur ou une spécification d’audit de la base de données](#createspec).

   - <a id="blobssms"></a>**Configurer le stockage Blob pour les journaux d’audit à l’aide de SQL Server Management Studio 18 :**

     1. Connectez-vous à l’instance managée à l’aide de l’interface utilisateur de SQL Server Management Studio.

     1. Développez le nœud racine de l’Explorateur d’objets.

     1. Développez le nœud **Sécurité**, cliquez avec le bouton droit sur le nœud **Audits**, puis cliquez sur **Nouvel audit** :

        ![Développer les nœuds Sécurité et Audit](./media/auditing-configure/10_mi_SSMS_new_audit.png)

     1. Vérifiez que **URL** est sélectionné dans **Destination de l’audit**, puis cliquez sur **Parcourir** :

        ![Parcourir le stockage Azure](./media/auditing-configure/11_mi_SSMS_audit_browse.png)

     1. (Facultatif) Connectez-vous à votre compte Azure :

        ![Connexion à Azure](./media/auditing-configure/12_mi_SSMS_sign_in_to_azure.png)

     1. Sélectionnez un abonnement, un compte de stockage et un conteneur d’objets blob dans les menus déroulants, ou cliquez sur **Créer** pour créer votre propre conteneur. Une fois terminé, cliquez sur **OK** :

        ![Sélectionner un abonnement, un compte de stockage et un conteneur d’objets blob Azure](./media/auditing-configure/13_mi_SSMS_select_subscription_account_container.png)

     1. Cliquez sur **OK** dans la boîte de dialogue **Créer un audit**.
     
     1. <a id="createspec"></a>Après avoir configuré le conteneur d’objets blob comme cible pour les journaux d’audit, créez et activez une spécification d’audit du serveur ou une spécification d’audit de la base de données comme vous le feriez pour SQL Server :

   - [CREATE SERVER AUDIT SPECIFICATION (Transact-SQL)](/sql/t-sql/statements/create-server-audit-specification-transact-sql)
   - [CREATE DATABASE AUDIT SPECIFICATION (Transact-SQL)](/sql/t-sql/statements/create-database-audit-specification-transact-sql)

5. Activez l’audit du serveur que vous avez créé à l’étape 3 :

    ```SQL
    ALTER SERVER AUDIT [<your_audit_name>]
    WITH (STATE=ON);
    GO
    ```

Pour toute information supplémentaire :

- [Différences d’audit entre Azure SQL Managed Instance et une base de données dans SQL Server](#auditing-differences-between-databases-in-azure-sql-managed-instance-and-databases-in-sql-server)
- [CREATE SERVER AUDIT](/sql/t-sql/statements/create-server-audit-transact-sql)
- [ALTER SERVER AUDIT](/sql/t-sql/statements/alter-server-audit-transact-sql)

## <a name="set-up-auditing-for-your-server-to-event-hubs-or-azure-monitor-logs"></a>Configurer l’audit de votre serveur sur Event Hubs ou des journaux Azure Monitor

Les journaux d’audit d’une instance managée peuvent être envoyés à Event Hubs ou aux journaux Azure Monitor. Cette section décrit comment configurer cela :

1. Accédez dans le [portail Azure](https://portal.azure.com/) à l’instance managée.

2. Cliquez sur **Paramètres de diagnostic**.

3. Cliquez sur **Activer les diagnostics**. Si les diagnostics sont déjà activés, **+Ajouter un paramètre de diagnostic** s’affiche à la place.

4. Sélectionnez **SQLSecurityAuditEvents** dans la liste des journaux d’activité.

5. Sélectionnez une destination pour les événements d’audit : Event Hubs, les journaux Azure Monitor ou les deux. Configurez les paramètres requis (par exemple, espace de travail Log Analytics) pour chaque cible.

6. Cliquez sur **Enregistrer**.

    ![Configurer les paramètres de diagnostic](./media/auditing-configure/9_mi_configure_diagnostics.png)

7. Connectez-vous à l’instance gérée à l’aide de **SQL Server Management Studio (SSMS)** ou de tout autre client pris en charge.

8. Exécutez l’instruction T-SQL suivante pour créer un audit du serveur :

    ```SQL
    CREATE SERVER AUDIT [<your_audit_name>] TO EXTERNAL_MONITOR;
    GO
    ```

9. Créez et activez une spécification d’audit du serveur ou une spécification d’audit de la base de données comme vous le feriez pour SQL Server :

   - [CREATE SERVER AUDIT SPECIFICATION (Transact-SQL)](/sql/t-sql/statements/create-server-audit-specification-transact-sql)
   - [CREATE DATABASE AUDIT SPECIFICATION (Transact-SQL)](/sql/t-sql/statements/create-database-audit-specification-transact-sql)

10. Activez l’audit du serveur créé à l’étape 8 :

    ```SQL
    ALTER SERVER AUDIT [<your_audit_name>]
    WITH (STATE=ON);
    GO
    ```

## <a name="consume-audit-logs"></a>Utiliser les journaux d’audit

### <a name="consume-logs-stored-in-azure-storage"></a>Utiliser les journaux d’activité stockés dans Stockage Azure

Plusieurs méthodes vous permettent d’afficher des journaux d’activité d’audit d’objets blob.

- Utilisez la fonction système `sys.fn_get_audit_file` pour retourner les données du journal d’audit dans un format tabulaire. Pour plus d’informations sur l’utilisation de cette fonction, consultez la [documentation sys.fn_get_audit_file](/sql/relational-databases/system-functions/sys-fn-get-audit-file-transact-sql).

- Vous pouvez explorer les journaux d’audit avec un outil comme [l’Explorateur de stockage Azure](https://azure.microsoft.com/features/storage-explorer/). Dans Stockage Azure, les journaux d’audit sont enregistrés sous la forme d’une collection de fichiers blob dans un conteneur défini pour stocker les journaux d’audit. Pour plus d’informations sur la hiérarchie du dossier de stockage, sur les conventions de nommage et sur le format des journaux, consultez le [document de référence sur le format des journaux d’audit d’objets blob](../database/audit-log-format.md).

- Pour obtenir la liste complète des méthodes de consommation du journal d’audit, consultez l’article [Bien démarrer avec l’audit Azure SQL Database](../../azure-sql/database/auditing-overview.md).

### <a name="consume-logs-stored-in-event-hubs"></a>Consommer les journaux stockés dans Event Hubs

Pour consommer les données des journaux d’audit à partir d’Event Hubs, vous devez configurer un flux de données destiné à consommer les événements et à les écrire dans une cible. Pour plus d’informations, consultez la documentation d’Azure Event Hubs.

### <a name="consume-and-analyze-logs-stored-in-azure-monitor-logs"></a>Consommer et analyser les journaux stockés dans les journaux Azure Monitor

Si les journaux d’audit sont écrits dans des journaux Azure Monitor, ils sont disponibles dans l’espace de travail Log Analytics où vous pouvez exécuter des recherches avancées sur les données d’audit. Comme point de départ, accédez à l’espace de travail Log Analytics. Dans la section **Général**, cliquez sur **Journaux** et entrez une requête simple, telle que : `search "SQLSecurityAuditEvents"` pour afficher les journaux d’audit.  

Les journaux Azure Monitor vous donnent des insights opérationnels en temps réel à l’aide d’une recherche intégrée et de tableaux de bord personnalisés permettant d’analyser facilement des millions d’enregistrements dans l’ensemble de vos charges de travail et serveurs. Pour plus d’informations utiles sur le langage et les commandes de recherche des journaux Azure Monitor, consultez [Informations de référence sur la recherche dans les journaux Azure Monitor](../../azure-monitor/log-query/log-query-overview.md).

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="auditing-differences-between-databases-in-azure-sql-managed-instance-and-databases-in-sql-server"></a>Différences d’audit entre les bases de données dans Azure SQL Managed Instance et les bases de données dans SQL Server

Les principales différences entre l’audit des bases de données dans Azure SQL Managed Instance et des bases de données dans SQL Server sont les suivantes :

- Avec Azure SQL Managed Instance, l’audit fonctionne au niveau du serveur et stocke les fichiers journaux `.xel` dans le compte Stockage Blob Azure.
- Dans SQL Server, l’audit fonctionne au niveau du serveur, mais stocke les événements dans les journaux des événements du système de fichiers/Windows.

L’audit XEvent dans les instances managées prend en charge les cibles Stockage Blob Azure. Les journaux d’activité de fichiers et de Windows ne sont **pas pris en charge**.

Les principales différences de syntaxe `CREATE AUDIT` pour l’audit du Stockage Blob Azure sont :

- Une nouvelle syntaxe `TO URL` est fournie et vous permet de spécifier l’URL du conteneur Stockage Blob Azure où les fichiers `.xel` sont placés.
- Une nouvelle syntaxe `TO EXTERNAL MONITOR` est fournie pour activer les cibles de journaux Azure Monitor et Event Hub.
- La syntaxe `TO FILE` n’est **pas prise en charge**, car Azure SQL Managed Instance ne peut pas accéder à des partages de fichiers Windows.
- L’option d’arrêt n’est **pas prise en charge**.
- La valeur 0 du paramètre `queue_delay` n’est **pas prise en charge**.

## <a name="next-steps"></a>Étapes suivantes

- Pour obtenir la liste complète des méthodes de consommation du journal d’audit, consultez l’article [Bien démarrer avec l’audit Azure SQL Database](../../azure-sql/database/auditing-overview.md).
- Pour plus d’informations sur les programmes Azure qui prennent en charge la conformité aux normes, consultez le [Centre de confidentialité Azure](https://gallery.technet.microsoft.com/Overview-of-Azure-c1be3942) où vous trouverez la liste actualisée des certifications de conformité.

<!--Image references-->
