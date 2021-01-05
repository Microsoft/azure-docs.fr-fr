---
title: Audit Azure SQL pour Azure SQL Database et Azure Synapse Analytics
description: Utilisez la fonctionnalité d’audit d’Azure SQL Database pour effectuer le suivi des événements de base de données dans un journal d’audit.
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.topic: conceptual
author: DavidTrigano
ms.author: datrigan
ms.reviewer: vanto
ms.date: 11/08/2020
ms.custom: azure-synapse, sqldbrb=1
ms.openlocfilehash: b09eb03994098f8cb68033f3c42309a77e15f91c
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96620989"
---
# <a name="auditing-for-azure-sql-database-and-azure-synapse-analytics"></a>Audit pour Azure SQL Database et Azure Synapse Analytics
[!INCLUDE[appliesto-sqldb-asa](../includes/appliesto-sqldb-asa.md)]

L’audit pour Azure [SQL Database](sql-database-paas-overview.md) et [Azure Synapse Analytics](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md) suit les événements de base de données et les écrit dans un journal d’audit dans votre compte de stockage Azure, votre espace de travail Log Analytics ou Event Hubs.

Par ailleurs, l’audit :

- peut vous aider à respecter une conformité réglementaire, à comprendre l’activité de la base de données ainsi qu’à découvrir des discordances et anomalies susceptibles d’indiquer des problèmes pour l’entreprise ou des violations de la sécurité ;

- permet et facilite le respect de normes de conformité, même s’il ne garantit pas cette conformité. Pour plus d’informations sur les programmes Azure qui prennent en charge la conformité aux normes, consultez le [Centre de confidentialité Azure](https://gallery.technet.microsoft.com/Overview-of-Azure-c1be3942) où vous trouverez la liste actualisée des certifications de conformité Azure SQL.

> [!NOTE]
> Pour plus d’informations sur l’audit d’Azure SQL Managed Instance, consultez l’article suivant [Prise en main de l’audit de SQL Managed Instance](../managed-instance/auditing-configure.md).

## <a name="overview"></a><a id="overview"></a>Vue d’ensemble

Vous pouvez utiliser l’audit SQL Database pour :

- **La rétention** d’une piste d’audit d’événements sélectionnés. Définissez les catégories d'actions et d'événements de base de données à auditer.
- **La génération de rapports** sur les activités de la base de données. Vous pouvez utiliser des rapports préconfigurés et un tableau de bord pour une prise en main rapide de la génération de rapports d’activités et d’événements.
- **L'analyse** des rapports. Vous pouvez repérer les événements suspects, les activités inhabituelles et les tendances.

> [!IMPORTANT]
> L’audit pour Azure SQL Database et Azure Synapse est optimisé pour la disponibilité et les performances. En cas de très haute activité ou de charge réseau élevée, Azure SQL Database ou Azure Synapse permet aux opérations de se poursuivre et peut ne pas enregistrer certains événements audités.

### <a name="auditing-limitations"></a>Limitations de l’audit

- Le **stockage Premium** n'est actuellement **pas pris en charge**.
- **L’espace de noms hiérarchique** pour un **compte de stockage Azure Data Lake Storage Gen2** n’est actuellement **pas pris en charge**.
- L’activation de l’audit sur une instance **Azure Synapse** interrompue n’est pas prise en charge. Pour activer l’audit, reprenez Azure Synapse.

#### <a name="define-server-level-vs-database-level-auditing-policy"></a><a id="server-vs-database-level"></a>Définir une stratégie d’audit au niveau du serveur ou au niveau de la base de données

Une stratégie d’audit peut être définie pour une base de données spécifique ou en tant que stratégie de [serveur](logical-servers.md) par défaut dans Azure (qui héberge SQL Database ou Azure Synapse) :

- Une stratégie de serveur s’applique aux bases de données existantes et à celles qui sont nouvellement créées sur le serveur.

- Si *l’audit du serveur est activé*, il *s’applique toujours à la base de données*. La base de données est auditée, quels que soient les paramètres d’audit de la base de données.

- L’activation de l’audit dans la base de données, en plus de son activation sur le serveur, ne remplace *pas* et ne modifie pas non plus les paramètres d'audit du serveur. Les deux audits coexistent. En d’autres termes, la base de données est auditée deux fois en parallèle (une fois par la stratégie du serveur et une autre fois par la stratégie de la base de données).

   > [!NOTE]
   > Vous devez éviter d’activer à la fois l’audit du serveur et l’audit de la base de données, sauf si :
    >
    > - Vous voulez utiliser un autre *compte de stockage*, une autre *période de rétention* ou un *espace de travail Log Analytics* pour une base de données spécifique.
    > - Vous souhaitez auditer des types ou des catégories d’événements pour une base de données qui sont différents de ceux qui sont audités pour les autres bases de données du serveur. Par exemple, il est possible que des insertions de table doivent être auditées uniquement pour une base de données spécifique.
   >
   > Sinon, nous vous recommandons d’activer uniquement l’audit au niveau du serveur et de laisser l’audit au niveau de la base de données désactivé pour toutes les bases de données.

#### <a name="remarks"></a>Notes

- Les journaux d’audit sont écrits dans des **Blobs d’ajout** dans un stockage Blob Azure avec votre abonnement Azure.
- Les journaux d’audit sont au format .xel et peuvent être ouverts à l’aide de [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms).
- Pour configurer un magasin de journaux immuable pour ses événements d’audit au niveau du serveur ou de la base de données, suivez les [instructions fournies dans le Stockage Azure](../../storage/blobs/storage-blob-immutability-policies-manage.md#enabling-allow-protected-append-blobs-writes). Assurez-vous que vous avez sélectionné **Autoriser les ajouts supplémentaires** lorsque vous configurez le stockage d’objets blob immuables.
- Vous pouvez écrire des journaux d’audit dans un compte Stockage Azure derrière un réseau virtuel ou un pare-feu. Pour obtenir des instructions spécifiques, consultez [Écrire un audit dans un compte de stockage situé derrière un réseau virtuel ou un pare-feu](audit-write-storage-account-behind-vnet-firewall.md).
- Pour plus d’informations sur le format du journal, la hiérarchie du dossier de stockage et les conventions d’affectation de nom,consultez le [document de référence sur le format des journaux d’audit d’objets blob](./audit-log-format.md).
- L’audit sur les [réplicas en lecture seule](read-scale-out.md) est activé automatiquement. Pour plus d’informations sur la hiérarchie des dossiers de stockage, sur les conventions de nommage et sur le format des journaux, consultez la documentation relative au [Format des journaux d’audit SQL Database](audit-log-format.md).
- Quand vous utilisez Azure AD Authentication, les échecs de connexion ne sont *pas* enregistrés dans le journal d’audit SQL. Pour voir les enregistrements d’audit des échecs de connexion, accédez au [portail Azure Active Directory](../../active-directory/reports-monitoring/reference-sign-ins-error-codes.md), qui affiche les détails de ces événements.
- Les connexions sont acheminées par la passerelle vers l’instance spécifique où se trouve la base de données.  Dans le cas des connexions AAD, les informations d’identification sont vérifiées avant toute tentative d’utilisation de cet utilisateur pour se connecter à la base de données demandée.  En cas d’échec, la base de données demandée n’est jamais accessible, de sorte qu’aucun audit n’est effectué.  Dans le cas des connexions SQL, les informations d’identification sont vérifiées sur les données demandées, de sorte qu’elles peuvent être auditées.  Les connexions réussies, qui ont manifestement atteint la base de données, sont auditées dans les deux cas.
- Une fois que vous avez configuré vos paramètres d’audit, vous pouvez activer la nouvelle fonctionnalité de détection des menaces et configurer les adresses e-mail de réception des alertes de sécurité. La détection des menaces vous permet de recevoir des alertes proactives sur des activités anormales de la base de données qui peuvent indiquer des menaces de sécurité potentielles. Pour plus d’informations, consultez [Bien démarrer avec la détection des menaces](threat-detection-overview.md).

## <a name="set-up-auditing-for-your-server"></a><a id="setup-auditing"></a>Configurer l’audit de votre serveur

La stratégie d’audit par défaut inclut toutes les actions et l’ensemble suivant de groupes d’actions, qui feront l’audit de toutes les requêtes et procédures stockées exécutées sur la base de données, ainsi que des connexions réussies et échouées :
  
- BATCH_COMPLETED_GROUP
- SUCCESSFUL_DATABASE_AUTHENTICATION_GROUP
- FAILED_DATABASE_AUTHENTICATION_GROUP
  
Vous pouvez configurer l’audit pour différents types d’actions et groupes d’actions à l’aide de PowerShell, comme décrit dans la section [Gérer l’audit de SQL Database avec Azure PowerShell](#manage-auditing).

L’audit Azure SQL Database et Azure Synapse stocke 4 000 caractères de données pour des champs de caractères dans un enregistrement d’audit. Lorsque l’**instruction** ou les valeurs **data_sensitivity_information** retournées à partir d’une action pouvant être auditée contiennent plus de 4 000 caractères, toutes les données au-delà des 4 000 premiers caractères sont  **tronquées et ne sont pas auditées**.
La section suivante décrit la configuration de l’audit à l’aide du portail Azure.

  > [!NOTE]
  > L’activation de l’audit sur un pool SQL dédié suspendu n’est pas possible. Pour activer l’audit, annulez l’interruption du pool SQL dédié. En savoir plus sur le [pool SQL dédié](../..//synapse-analytics/sql/best-practices-sql-pool.md).

1. Accédez au [portail Azure](https://portal.azure.com).
2. Accédez à **Audit** sous l’en-tête Sécurité dans votre volet **SQL Database** ou **SQL Server**.
3. Si vous préférez définir une stratégie d’audit de serveur, vous pouvez sélectionner le lien **Afficher les paramètres du serveur** dans la page d’audit de la base de données. Vous pouvez alors afficher ou modifier les paramètres d’audit du serveur. Les stratégies d’audit de serveur s’appliquent aux bases de données existantes et à celles qui sont nouvellement créées sur le serveur.

    ![Capture d’écran montrant le lien Afficher les paramètres du serveur mis en évidence sur la page d’audit de la base de données.](./media/auditing-overview/2_auditing_get_started_server_inherit.png)

4. Si vous préférez activer l’audit au niveau base de données, définissez **Audit** sur **ACTIVÉ**. Si l’audit d’objets du serveur est activé, l’audit configuré pour la base de données coexiste avec celui-ci.

5. Vous disposez de plusieurs options pour configurer l’emplacement d’écriture des journaux d’audit. Vous pouvez écrire des journaux dans un compte de stockage Azure ou dans un espace de travail Log Analytics pour qu’ils soient consommés par des journaux d’activité Azure Monitor (préversion), ou dans un Event Hub pour qu’ils soient consommés par cet Event Hub (préversion). Vous pouvez associer ces options comme vous le souhaitez. Les journaux d’audit seront écrits dans chacun des emplacements choisis.
  
   ![Options de stockage](./media/auditing-overview/auditing-select-destination.png)

### <a name="auditing-of-microsoft-support-operations-preview"></a><a id="auditing-of-microsoft-support-operations"></a>Audit des opérations de Support Microsoft (Préversion)

L’audit des opérations de Support Microsoft (Préversion) pour Azure SQL Server vous permet d’auditer les opérations des ingénieurs du support technique de Microsoft quand ils ont besoin d’accéder à votre serveur au cours d’une demande de support. L’utilisation de cette fonctionnalité, ainsi que votre audit, permet une plus grande transparence pour votre personnel, ainsi que la détection des anomalies, la visualisation des tendances et la protection contre la perte de données.

Pour activer l’audit des opérations de Support Microsoft (Préversion), accédez à **Auditing** (Audit) sous l’en-tête Security (Sécurité) de votre volet **Azure SQL Server**, puis basculez **Auditing of Microsoft support operations (Preview)** (Audit des opérations de Microsoft Support (Préversion)) sur **ON** (ACTIF).

  > [!IMPORTANT]
  > L’audit des opérations de Support Microsoft (Préversion) ne prend pas en charge la destination du compte de stockage. Pour activer cette capacité, vous devez configurer un espace de travail Log Analytics ou une destination Event Hub.

![Capture d’écran des opérations de Support Microsoft](./media/auditing-overview/support-operations.png)

Pour passer en revue les journaux d’audit des opérations de Support Microsoft dans votre espace de travail Log Analytics, utilisez la requête suivante :

```kusto
AzureDiagnostics
| where Category == "DevOpsOperationsAudit"
```

### <a name="audit-to-storage-destination"></a><a id="audit-storage-destination"></a>Écriture des journaux d’audit dans un compte de stockage

Pour configurer l’écriture des journaux d’audit dans un compte de stockage, sélectionnez **Stockage**, puis ouvrez **Détails du stockage**. Sélectionnez le compte de stockage Azure dans lequel les journaux d’activité seront enregistrés, puis sélectionnez la période de rétention. Cliquez ensuite sur **OK**. Une fois la période de conservation écoulée, les journaux sont supprimés.

- La valeur par défaut de la période de conservation est 0 (conservation illimitée). Vous pouvez changer cette valeur en déplaçant le curseur **Rétention (jours)** dans **Paramètres de stockage** lors de la configuration du compte de stockage à des fins d’audit.
  - Si vous remplacez la valeur 0 de la période de rétention (rétention illimitée) par une autre valeur, notez que la rétention s’appliquera uniquement aux journaux écrits après la modification de la valeur de rétention (les journaux écrits au cours de la période pendant laquelle la rétention était définie sur illimité sont conservés, même après activation de la rétention).

  ![compte de stockage](./media/auditing-overview/auditing_select_storage.png)

### <a name="audit-to-log-analytics-destination"></a><a id="audit-log-analytics-destination"></a>Écriture des journaux d’audit dans Log Analytics
  
Pour configurer l’écriture des journaux d’audit dans un espace de travail Log Analytics, sélectionnez **Log Analytics (préversion)** , puis ouvrez **Détails de Log Analytics**. Sélectionnez ou créez l’espace de travail Log Analytics où les journaux d’activité doivent être écrits, puis cliquez sur **OK**.

   ![LogAnalyticsworkspace](./media/auditing-overview/auditing_select_oms.png)

Pour plus d’informations sur l’espace de travail Azure Monitor Log Analytics, consultez [Conception de votre déploiement de journaux Azure Monitor](../../azure-monitor/platform/design-logs-deployment.md).
   
### <a name="audit-to-event-hub-destination"></a><a id="audit-event-hub-destination"></a>Écriture des journaux d’audit dans Event Hub

Pour configurer l’écriture des journaux d’audit dans un hub d’événements, sélectionnez **Hub d’événements (préversion)** , puis ouvrez **Détails du hub d’événements**. Sélectionnez le hub d’événements dans lequel les journaux d’activité doivent être écrits, puis cliquez sur **OK**. Veillez à ce que le hub d’événements se trouve dans la même région que votre base de données et votre serveur.

   ![Eventhub](./media/auditing-overview/auditing_select_event_hub.png)

## <a name="analyze-audit-logs-and-reports"></a><a id="subheading-3"></a>Analyse des journaux et des rapports d’audit

Si vous avez choisi d’écrire les journaux d’audit dans des journaux Azure Monitor :

- Utilisez le [portail Azure](https://portal.azure.com). Ouvrez la base de données appropriée. En haut de la page **Audit** de la base de données, sélectionnez **Afficher les journaux d’audit**.

    ![Afficher les journaux d’audit](./media/auditing-overview/auditing-view-audit-logs.png)

- Ensuite, vous avez deux façons d’afficher les journaux :

    Cliquez sur **Log Analytics** en haut de la page **Enregistrements d’audit** pour ouvrir la vue Journaux d’activité dans l’espace de travail Log Analytics, où vous pouvez personnaliser la plage de temps et la requête de recherche.

    ![ouvrir dans l’espace de travail Log Analytics](./media/auditing-overview/auditing-log-analytics.png)

    Cliquer sur **Afficher le tableau de bord** en haut de la page **Enregistrements d’audit** ouvre un tableau de bord affichant les informations des journaux d’audit, dans lequel vous pouvez accéder aux insights de sécurité, l’accès aux données sensibles et plus encore. Ce tableau de bord est conçu pour vous aider à obtenir des Insights sur la sécurité de vos données.
    Vous pouvez également personnaliser l’intervalle de temps et la requête de recherche.
    ![Afficher le tableau de bord Log Analytics](media/auditing-overview/auditing-view-dashboard.png)

    ![Tableau de bord Log Analytics](media/auditing-overview/auditing-log-analytics-dashboard.png)

    ![Insights sur la sécurité Log Analytics](media/auditing-overview/auditing-log-analytics-dashboard-data.png)

- Vous pouvez également accéder aux journaux d’audit à partir du panneau Log Analytics. Ouvrez votre espace de travail Log Analytics, puis, dans la section **Général**, cliquez sur **Journaux d’activité**. Vous pouvez démarrer par une requête simple, telle que : *search "SQLSecurityAuditEvents"* pour afficher les journaux d’audit.
    Vous pouvez également utiliser les [journaux Azure Monitor](../../azure-monitor/log-query/log-query-overview.md) pour exécuter des recherches avancées sur les données de votre journal d’audit. Les journaux Azure Monitor vous donnent des insights opérationnels en temps réel à l’aide d’une recherche intégrée et de tableaux de bord personnalisés permettant d’analyser facilement des millions d’enregistrements dans l’ensemble de vos charges de travail et serveurs. Pour plus d’informations utiles sur le langage et les commandes de recherche des journaux Azure Monitor, consultez [Informations de référence sur la recherche dans les journaux Azure Monitor](../../azure-monitor/log-query/log-query-overview.md).

Si vous avez choisi d’écrire les journaux d’audit dans un hub d’événements :

- Pour utiliser les données des journaux d’audit à partir d’un hub d’événements, vous devez configurer un flux de données destiné à consommer les événements et à les écrire dans une cible. Pour plus d’informations, consultez la [documentation Azure Event Hubs](../index.yml).
- Les journaux d’audit d’Event Hub sont capturés dans le corps d’événements [Apache Avro](https://avro.apache.org/) et stockés en format JSON avec l’encodage UTF-8. Pour lire les journaux d’audit, vous pouvez utiliser les [outils Avro](../../event-hubs/event-hubs-capture-overview.md#use-avro-tools) ou des outils similaires qui traitent ce format.

Si vous choisissez d’écrire les journaux d’audit dans un compte de stockage Azure, plusieurs méthodes existent pour afficher les journaux d’activité :

- Les journaux d’audit sont agrégés dans le compte choisi lors de la configuration. Vous pouvez explorer les journaux d’audit avec un outil comme [l’Explorateur de stockage Azure](https://storageexplorer.com/). Dans le stockage Azure, les journaux d’activité d’audit sont enregistrés sous la forme d’une collection de fichiers d’objets blob dans un conteneur nommé **sqldbauditlogs**. Pour plus d’informations sur la hiérarchie des dossiers de stockage, sur les conventions de nommage et sur le format des journaux, consultez la documentation relative au [Format des journaux d’audit SQL Database](./audit-log-format.md).

- Utilisez le [portail Azure](https://portal.azure.com).  Ouvrez la base de données appropriée. En haut de la page **Audit** de la base de données, cliquez sur **Afficher les journaux d’audit**.

    ![Capture d’écran montrant le bouton Afficher les journaux d’audit mis en évidence sur la page d’audit de la base de données.](./media/auditing-overview/7_auditing_get_started_blob_view_audit_logs.png)

    La page **Enregistrements d’audit** s’ouvre et vous y voyez les journaux d’activité.

  - Vous pouvez afficher des dates spécifiques en cliquant sur **Filtrer** en haut de la page **Enregistrements d’audit**.
  - Vous pouvez basculer entre les enregistrements d’audit qui ont été créés par la *stratégie d’audit de serveur* et la *stratégie d’audit de base de données* en choisissant la **Source de l’audit**.
  - Vous pouvez afficher uniquement les enregistrements d’audit liés aux injections SQL en cochant la case **Afficher uniquement les enregistrements d’audit pour les injections SQL**.

       ![Capture d’écran montrant les options permettant d’afficher les enregistrements d’audit.]( ./media/auditing-overview/8_auditing_get_started_blob_audit_records.png)

- Utilisez la fonction système **sys.fn_get_audit_file** (T-SQL) pour retourner les données du journal d’audit sous un format tabulaire. Pour plus d’informations sur l’utilisation de cette fonction, consultez [sys.fn_get_audit_file](/sql/relational-databases/system-functions/sys-fn-get-audit-file-transact-sql).

- Utilisez **Fusionner les fichiers d’audit** dans SQL Server Management Studio (à partir de SSMS 17) :
    1. Dans le menu SSMS, sélectionnez **Fichier** > **Ouvrir** > **Fusionner les fichiers d’audit**.

        ![Capture d’écran montrant l’option de menu Fusionner les fichiers d’audit.](./media/auditing-overview/9_auditing_get_started_ssms_1.png)
    2. La boîte de dialogue **Ajouter des fichiers d’audit** s’ouvre. Sélectionnez l’une des options **Ajouter** pour choisir de fusionner les fichiers d’audit à partir d’un disque local ou de les importer à partir du stockage Azure. Vous devrez fournir vos informations de stockage Azure et la clé de compte.

    3. Une fois que tous les fichiers à fusionner ont été ajoutés, cliquez sur **OK** pour terminer l’opération de fusion.

    4. Le fichier fusionné s’ouvre dans SSMS, où vous pouvez l’afficher et l’analyser, ainsi que l’exporter vers un fichier XEL ou CSV, ou vers une table.

- Utilisez Power BI. Vous pouvez afficher et analyser les données du journal d’audit dans Power BI. Pour plus d’informations et pour accéder à un modèle téléchargeable, consultez [Analyze audit log data in Power BI](https://techcommunity.microsoft.com/t5/azure-database-support-blog/sql-azure-blob-auditing-basic-power-bi-dashboard/ba-p/368895) (Analyser les données des journaux d’audit dans Power BI).
- Téléchargez les fichiers journaux à partir de votre conteneur Azure Storage Blob via le portail ou avec un outil comme [l’Explorateur de stockage Azure](https://storageexplorer.com/).
  - Une fois que vous avez téléchargé localement un fichier journal, double-cliquez sur le fichier pour ouvrir, afficher et analyser les journaux d’activité dans SSMS.
  - Vous pouvez également télécharger plusieurs fichiers simultanément avec l’Explorateur de stockage Azure. Pour cela, cliquez avec le bouton droit sur un sous-dossier, puis sélectionnez **Enregistrer en tant que** pour l’enregistrer dans un dossier local.

- Autres méthodes :

  - Après avoir téléchargé plusieurs fichiers (ou un sous-dossier contenant des fichiers journaux), vous pouvez les fusionner localement en suivant les instructions relatives à l’option Fusionner les fichiers d’audit dans SSMS, décrites précédemment.
  - Affichez des journaux d’activité d’audit d’objets blob par programmation : [Interrogez des fichiers d’événements étendus](https://sqlscope.wordpress.com/2014/11/15/reading-extended-event-files-using-client-side-tools-only/) avec PowerShell.

## <a name="production-practices"></a><a id="production-practices"></a>Pratiques de production


### <a name="auditing-geo-replicated-databases"></a>Audit des bases de données géorépliquées

Avec les bases de données géorépliquées, lorsque vous activez l’audit dans la base de données primaire, la même stratégie d’audit est appliquée à la base de données secondaire. Il est également possible de configurer l’audit dans la base de données secondaire en activant l’audit dans le **serveur secondaire**, indépendamment de la base de données primaire.

- Au niveau du serveur (**recommandé**) : activez l’audit sur le **serveur principal** et le **serveur secondaire**. Les bases de données primaire et secondaire sont auditées, indépendamment l’une de l’autre, en fonction de leur stratégie de niveau serveur respective.
- Au niveau de la base de données : l’audit au niveau de la base de données ne peut être configuré pour les bases de données secondaires qu’à partir des paramètres d’audit de la base de données primaire.
  - L’audit doit être activé sur la *base de données primaire elle-même*, et non pas sur le serveur.
  - Une fois que l’audit est activé sur la base de données primaire, il est également activé sur la base de données secondaire.

    > [!IMPORTANT]
    > Avec l’audit au niveau de la base de données, les paramètres de stockage de la base de données secondaire sont identiques à ceux de la base de données primaire, ce qui entraîne un trafic entre régions. Il est conseillé d’activer uniquement l’audit d’objets blob au niveau du serveur et de laisser l’audit au niveau de la base de données désactivé pour toutes les bases de données.

### <a name="storage-key-regeneration"></a>Régénération des clés de stockage

Dans un environnement de production, vous allez probablement actualiser périodiquement vos clés de stockage. Si vous écrivez les journaux d’audit dans le stockage Azure, vous devez réenregistrer votre stratégie d’audit lors de l’actualisation de vos clés. Pour ce faire, procédez comme suit :

1. Ouvrez **Détails du stockage**. Dans la zone **Clé d’accès de stockage**, sélectionnez **Secondaire**, puis cliquez sur **OK**. Cliquez ensuite sur **Enregistrer** en haut de la page de configuration de l’audit.

    ![Capture d’écran montrant le processus de sélection d’une clé d’accès de stockage secondaire.](./media/auditing-overview/5_auditing_get_started_storage_key_regeneration.png)
2. Accédez à la page de configuration du stockage, puis regénérez la clé d’accès primaire.

    ![Volet de navigation](./media/auditing-overview/6_auditing_get_started_regenerate_key.png)
3. Revenez à la page de configuration de l’audit, remplacez la clé d’accès de stockage secondaire par la clé primaire, puis cliquez sur **OK**. Cliquez ensuite sur **Enregistrer** en haut de la page de configuration de l’audit.
4. Revenez à la page de configuration du stockage, puis regénérez la clé d’accès secondaire (en préparation du cycle suivant d’actualisation des clés).

## <a name="manage-azure-sql-database-auditing"></a><a id="manage-auditing"></a>Gérer l'audit Azure SQL Database

### <a name="using-azure-powershell"></a>Utilisation de Microsoft Azure PowerShell

**Applets de commande PowerShell (y compris prise en charge de la clause WHERE pour un filtrage supplémentaire)**  :

- [Créer ou mettre à jour une stratégie d’audit de base de données (Set-AzSqlDatabaseAudit)](/powershell/module/az.sql/set-azsqldatabaseaudit)
- [Créer ou mettre à jour une stratégie de serveur (Set-AzSqlServerAudit)](/powershell/module/az.sql/set-azsqlserveraudit)
- [Obtenir une stratégie d’audit de base de données (Get-AzSqlDatabaseAudit)](/powershell/module/az.sql/get-azsqldatabaseaudit)
- [Obtenir une stratégie d’audit de serveur (Get-AzSqlServerAudit)](/powershell/module/az.sql/get-azsqlserveraudit)
- [Supprimer une stratégie d’audit de base de données (Remove-AzSqlDatabaseAudit)](/powershell/module/az.sql/remove-azsqldatabaseaudit)
- [Supprimer une stratégie d’audit de serveur (Remove-AzSqlServerAudit)](/powershell/module/az.sql/remove-azsqlserveraudit)

Pour obtenir un exemple de script, consultez [Configurer l’audit et la détection des menaces avec PowerShell](scripts/auditing-threat-detection-powershell-configure.md).

### <a name="using-rest-api"></a>Utilisation de l'API REST

**API REST** :

- [Create or Update Database Auditing Policy](/rest/api/sql/database%20auditing%20settings/createorupdate)
- [Create or Update Server Auditing Policy](/rest/api/sql/server%20auditing%20settings/createorupdate)
- [Get Database Auditing Policy](/rest/api/sql/database%20auditing%20settings/get)
- [Get Server Auditing Policy](/rest/api/sql/server%20auditing%20settings/get)

Prise en charge de la stratégie étendue avec la clause WHERE pour un filtrage supplémentaire :

- [Créer ou mettre à jour la stratégie d’audit *étendue* de base de données](/rest/api/sql/database%20extended%20auditing%20settings/createorupdate)
- [Créer ou mettre à jour la stratégie d’audit *étendue* de serveur](/rest/api/sql/server%20auditing%20settings/createorupdate)
- [Obtenir la stratégie d’audit *étendue* de base de données](/rest/api/sql/database%20extended%20auditing%20settings/get)
- [Obtenir la stratégie d’audit *étendue* de serveur](/rest/api/sql/server%20auditing%20settings/get)

### <a name="using-azure-resource-manager-templates"></a>Utilisation de modèles Azure Resource Manager

Vous pouvez gérer l’audit Azure SQL Database à l’aide de modèles [Azure Resource Manager](../../azure-resource-manager/management/overview.md), comme indiqué dans ces exemples :

- [Déployer une instance Azure SQL Database avec l’audit activé pour écrire des journaux d’audit dans le compte de stockage blob Azure](https://github.com/Azure/azure-quickstart-templates/tree/master/201-sql-auditing-server-policy-to-blob-storage)
- [Déployer une instance Azure SQL Database avec l’audit activé pour écrire des journaux d’audit dans Log Analytics](https://github.com/Azure/azure-quickstart-templates/tree/master/201-sql-auditing-server-policy-to-oms)
- [Déployer une instance Azure SQL Database avec l’audit activé pour écrire des journaux d’audit dans Event Hubs](https://github.com/Azure/azure-quickstart-templates/tree/master/201-sql-auditing-server-policy-to-eventhub)

> [!NOTE]
> Les exemples liés se trouvent sur un référentiel public externe et sont fournis « en l’état », sans garantie et ne sont pas pris en charge dans n’importe quel service/programme de support Microsoft.
