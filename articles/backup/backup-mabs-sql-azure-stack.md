---
title: Sauvegarder des charges de travail SQL Server sur Azure Stack
description: Dans cet article, apprenez comment configurer serveur de Sauvegarde Microsoft Azure (MABS) afin de protéger des bases de données SQL Server sur Azure Stack.
ms.topic: conceptual
ms.date: 06/08/2018
ms.openlocfilehash: 80de7913b010fca69c3703e423109f2ede653590
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91332812"
---
# <a name="back-up-sql-server-on-azure-stack"></a>Sauvegarder SQL Server sur Azure Stack

Utilisez cet article pour configurer Microsoft Azure Backup Server (MABS) afin de protéger des bases de données SQL Server sur Azure Stack.

La gestion de sauvegarde et de récupération de base de données SQL Server dans Azure implique trois étapes :

1. Créer une stratégie de sauvegarde pour protéger les bases de données SQL Server
2. Créer des copies de sauvegarde à la demande
3. Récupérer la base de données à partir de disques et d’Azure

## <a name="prerequisites-and-limitations"></a>Conditions préalables et limitations

* Si vous disposez d’une base de données avec des fichiers sur un partage de fichiers distant, la protection échouera avec l’ID d’erreur 104. MABS ne prend pas en charge la protection des données SQL Server sur un partage de fichiers distant.
* MABS ne peut pas protéger les bases de données stockées sur des partages SMB distants.
* Assurez-vous que les [réplicas de groupe de disponibilité sont configurés en lecture seule](/sql/database-engine/availability-groups/windows/configure-read-only-access-on-an-availability-replica-sql-server).
* Vous devez explicitement ajouter le compte système **NTAuthority\System** au groupe Sysadmin sur SQL Server.
* Lorsque vous effectuez la récupération d’une base de données partiellement autonome à un autre emplacement, vous devez vous assurer que la fonctionnalité [Bases de données autonomes](/sql/relational-databases/databases/migrate-to-a-partially-contained-database#enable) est activée sur l’instance SQL cible.
* Lorsque vous effectuez la récupération d’une base de données de flux de fichiers à un autre emplacement, vous devez vous assurer que la fonctionnalité [Bases de données de flux de fichiers](/sql/relational-databases/blob/enable-and-configure-filestream) est activée sur l’instance SQL cible.
* Protection pour SQL Server AlwaysOn :
  * MABS détecte les groupes de disponibilité lors de l’exécution d’une demande au moment de la création d’un groupe de protection.
  * MABS détecte un basculement et poursuit la protection de la base de données.
  * MABS prend en charge les configurations de cluster multisites d’une instance de SQL Server.
* Lorsque vous protégez des bases de données qui utilisent la fonctionnalité AlwaysOn, MABS présente les limitations suivantes :
  * MABS honorera la stratégie de sauvegarde des groupes de disponibilité définie dans SQL Server en fonction des préférences de sauvegarde, comme suit :
    * Préférer le réplica secondaire : les sauvegardes doivent être effectuées sur un réplica secondaire, sauf lorsque le réplica principal est le seul réplica en ligne. Si plusieurs réplicas secondaires sont disponibles, le nœud ayant la priorité de sauvegarde la plus élevée sera sélectionné pour la sauvegarde. Si seul le réplica principal est disponible, la sauvegarde doit s’effectuer sur le réplica principal.
    * Secondaire uniquement : la sauvegarde ne doit pas être effectuée sur le réplica principal. Si le réplica principal est le seul réplica en ligne, la sauvegarde ne doit pas s’effectuer.
    * Principal : les sauvegardes doivent toujours s’effectuer sur le réplica principal.
    * Sur n’importe quel réplica : les sauvegardes peuvent s’effectuer sur n’importe quel réplica de disponibilité dans le groupe de disponibilité. Le nœud à sauvegarder dépendra des priorités de sauvegarde pour chacun des nœuds.
  * Notez les points suivants :
    * Les sauvegardes peuvent s’effectuer à partir de n’importe quel réplica lisible, c’est-à-dire un réplica principal, secondaire synchrone ou secondaire asynchrone.
    * Si un réplica est exclus de la sauvegarde, par exemple si **Exclure des réplicas** est activé ou marqué comme étant non lisible, alors ce réplica ne sera pas sélectionné pour la sauvegarde, quelle que soit l’option.
    * Si plusieurs réplicas sont disponibles et lisibles, alors le nœud ayant la priorité de sauvegarde la plus élevée sera sélectionné pour la sauvegarde.
    * Si la sauvegarde échoue sur le nœud sélectionné, alors l’opération de sauvegarde échoue.
    * La récupération à l’emplacement d’origine n’est pas prise en charge.
* Problèmes de sauvegarde avec SQL Server 2014 ou versions ultérieures :
  * SQL Server 2014 a ajouté une nouvelle fonctionnalité pour créer une [base de données pour une instance SQL Server locale dans le stockage d’objets BLOB Windows Azure](/sql/relational-databases/databases/sql-server-data-files-in-microsoft-azure). MABS ne peut pas être utilisé pour protéger cette configuration.
  * Il existe certains problèmes connus avec la préférence de sauvegarde « Préférer secondaire » pour l’option SQL AlwaysOn. MABS effectue toujours une sauvegarde de la base de données secondaire. Si aucune base de données secondaire n’est trouvée, la sauvegarde échoue.

## <a name="before-you-start"></a>Avant de commencer

[Installez et préparez un serveur de sauvegarde Azure](backup-mabs-install-azure-stack.md).

## <a name="create-a-backup-policy-to-protect-sql-server-databases-to-azure"></a>Créer une stratégie de sauvegarde pour protéger les bases de données SQL Server dans Azure

1. Dans l’interface utilisateur du serveur de sauvegarde Azure, sélectionnez l’espace de travail **Protection**.

2. Dans le ruban des outils, sélectionnez **Nouveau** pour créer un nouveau groupe de protection.

    ![Créer un groupe de protection](./media/backup-azure-backup-sql/protection-group.png)

    Le serveur de sauvegarde Azure démarre l’Assistant Groupe de protection, qui vous guide au fil des étapes de création d’un **groupe de protection**. Sélectionnez **Suivant**.

3. Dans l’écran **Sélectionner le type de groupe de protection**, sélectionnez **Serveurs**.

    ![Sélectionner le type de groupe de protection - « Servers »](./media/backup-azure-backup-sql/pg-servers.png)

4. Dans l’écran **Sélectionner les membres du groupe**, la liste des membres disponibles affiche différentes sources de données. Sélectionnez **+** pour développer un dossier et afficher ses sous-dossiers. Cochez la case pour sélectionner un élément.

    ![Sélectionner la base de données SQL](./media/backup-azure-backup-sql/pg-databases.png)

    Tous les éléments sélectionnés s’affichent dans la liste Membres sélectionnés. Après avoir sélectionné les serveurs ou les bases de données à protéger, sélectionnez **Suivant**.

5. Dans l’écran **Sélectionner la mode de protection des données**, donnez un nom au groupe de protection, puis cochez la case **Je souhaite une protection en ligne**.

    ![Méthode de Protection des données - disque à court terme et Azure en ligne](./media/backup-azure-backup-sql/pg-name.png)

6. Dans l’écran **Spécifier les objectifs à court terme**, incluez les entrées nécessaires à la création de points de sauvegarde sur disque, puis sélectionnez **Suivant**.

    Dans l’exemple, la **durée de rétention** est définie sur **5 jours** et la **fréquence de synchronisation** sur **15 minutes**, soit la fréquence de sauvegarde. **sauvegarde expresse rapide** est définie sur **20h00**.

    ![Objectifs à court terme](./media/backup-azure-backup-sql/pg-shortterm.png)

   > [!NOTE]
   > Dans l’exemple suivant, un point de sauvegarde est créé tous les jours à 20 h 00 en transférant les données modifiées provenant du point de sauvegarde créé à 20 h 00 la veille. Ce processus est appelé **Sauvegarde expresse rapide**. Les journaux d’activité des transactions sont synchronisés toutes les 15 minutes. Si vous devez récupérer la base de données à 21 h 00, le point est créé à partir des journaux d’activité du dernier point de la sauvegarde complète express (20 h 00 dans ce cas).
   >
   >

7. Sur l’écran **Vérifier l’allocation de disque**, vérifiez l’espace de stockage global disponible et l’espace disque potentiel. Sélectionnez **Suivant**.

8. Dans le champ **Choisir la méthode de création d’un réplica**, choisissez comment créer votre premier point de récupération. Vous pouvez transférer manuellement la sauvegarde initiale (hors réseau) pour éviter l’encombrement de la bande passante ou sur le réseau. Si vous choisissez d’attendre avant de transférer la première sauvegarde, vous pouvez spécifier l’heure du transfert initial. Sélectionnez **Suivant**.

    ![Méthode de réplication initiale](./media/backup-azure-backup-sql/pg-manual.png)

    La copie de sauvegarde initiale nécessite le transfert de la source de données complète (base de données SQL Server) entre le serveur de production (ordinateur SQL Server) et le serveur de sauvegarde Azure. Ces données peuvent être volumineuses et leur transfert sur le réseau peut dépasser la bande passante. C'est la raison pour laquelle vous pouvez choisir de transférer la sauvegarde initiale : **Manuellement** (à l'aide d'un support amovible) afin d'éviter toute congestion de la bande passante, ou **Automatiquement sur le réseau** (à un moment précis).

    Une fois la sauvegarde initiale terminée, le reste des sauvegardes se compose de sauvegardes incrémentielles sur la copie de sauvegarde initiale. Les sauvegardes incrémentielles sont en général très limitées et sont faciles à transférer sur le réseau.

9. Choisissez si vous souhaitez ou non exécuter la vérification de cohérence, puis sélectionnez **Suivant**.

    ![Vérifier la cohérence](./media/backup-azure-backup-sql/pg-consistent.png)

    Le serveur de sauvegarde Azure effectue un contrôle de cohérence pour vérifier l’intégrité du point de sauvegarde. Il calcule la somme de contrôle du fichier de sauvegarde sur le serveur de production (ordinateur SQL Server dans ce scénario) et des données sauvegardée de ce fichier. En cas de conflit, le fichier sauvegardé sur le serveur de sauvegarde Azure est sans doute endommagé. Le serveur de sauvegarde Azure corrige les données sauvegardées en envoyant les blocs correspondant à l’incohérence du contrôle de cohérence. Étant donné que les contrôles de cohérence nécessitent des performances intensives, vous pouvez planifier ces opérations ou les exécuter automatiquement.

10. Pour spécifier la protection des sources de données en ligne, sélectionnez les bases de données à protéger sur Azure et sélectionnez **Suivant**.

    ![Sélectionner les sources de données](./media/backup-azure-backup-sql/pg-sqldatabases.png)

11. Choisissez les planifications de sauvegarde et les stratégies de rétention adaptées aux stratégies d’entreprise.

    ![Planification de sauvegarde et rétention](./media/backup-azure-backup-sql/pg-schedule.png)

    Dans cet exemple, les sauvegardes sont effectuées une fois par jour à 12 h 00 et 20 h 00 (partie inférieure de l’écran)

    > [!NOTE]
    > Il est conseillé de disposer de plusieurs points de récupération à court terme sur disque pour une récupération rapide. Ces points de récupération sont utilisés pour la restauration opérationnelle. Azure constitue un bon emplacement hors site, avec des contrats de niveau de service supérieurs et une disponibilité garantie.
    >
    >

    **Meilleure pratique** : si vous planifiez des sauvegardes sur Azure qui démarreront au terme des sauvegardes sur disque local, les dernières sauvegardes sur disque sont toujours copiées vers Azure.

12. Cliquez sur la planification de stratégie de rétention. Les détails du fonctionnement de la stratégie de rétention sont fournis dans la section [Utilisation d'Azure Backup pour remplacer votre infrastructure sur bande](backup-azure-backup-cloud-as-tape.md).

    ![Stratégie de rétention](./media/backup-azure-backup-sql/pg-retentionschedule.png)

    Dans cet exemple :

    * Les sauvegardes sont effectuées une fois par jour à 12 h 00 et 20 h 00 (partie inférieure de l’écran) et sont conservées pendant 180 jours.
    * La sauvegarde le samedi à 12 h 00 est conservée pendant 104 semaines
    * La sauvegarde le dernier samedi à 12 h 00 est conservée pendant 60 mois
    * La sauvegarde le dernier samedi de mars à 12 h 00 est conservée pendant 10 ans
13. Sélectionnez **Suivant** puis l’option appropriée pour transférer de la copie de sauvegarde initiale vers Azure. Vous pouvez choisir **Automatiquement sur le réseau**.

14. Après avoir passé en revue les détails de la stratégie dans l’écran **Résumé**, sélectionnez **Créer un groupe** pour terminer le workflow. Vous pouvez cliquer sur **Fermer** et superviser la progression du travail dans l’espace de travail Analyse.

    ![Créer un groupe de Protection en cours en progression](./media/backup-azure-backup-sql/pg-summary.png)

## <a name="on-demand-backup-of-a-sql-server-database"></a>Sauvegarde à la demande d’une base de données SQL Server

Alors que les étapes précédentes ont créé une stratégie de sauvegarde, un « point de récupération » est créé uniquement à l’occasion de la première sauvegarde. Au lieu d’attendre que le planificateur arrive, les étapes ci-dessous déclenchent la création manuelle d’un point de récupération.

1. Attendez que l'état du groupe de protection indique **OK** pour la base de données avant de créer le point de récupération.

    ![Membres du groupe de protection](./media/backup-azure-backup-sql/sqlbackup-recoverypoint.png)
2. Cliquez avec le bouton droit sur la base de données, puis sélectionnez **Créer un point de récupération**.

    ![Créer un point de récupération en ligne](./media/backup-azure-backup-sql/sqlbackup-createrp.png)
3. Choisissez **Protection en ligne** dans le menu déroulant, puis sélectionnez **OK** pour lancer la création d’un point de récupération dans Azure.

    ![Créer un point de récupération](./media/backup-azure-backup-sql/sqlbackup-azure.png)
4. Affichez la progression du travail dans l’espace de travail **Surveillance**.

    ![Console de surveillance](./media/backup-azure-backup-sql/sqlbackup-monitoring.png)

## <a name="recover-a-sql-server-database-from-azure"></a>Récupération d'une base de données SQL Server à partir d'Azure

Les étapes suivantes sont nécessaires pour récupérer une entité protégée (base de données SQL Server) à partir d'Azure.

1. Ouvrez la Console de gestion du serveur de sauvegarde Azure. Accédez à l’espace de travail **Récupération** où vous pourrez voir les serveurs protégés. Accédez à la base de données requise (dans ce cas, ReportServer $MSDPM2012). Sélectionnez une heure de **récupération** spécifiée comme point **en ligne**.

    ![Sélectionner un point de récupération](./media/backup-azure-backup-sql/sqlbackup-restorepoint.png)
2. Cliquez avec le bouton droit sur le nom de la base de données, puis sélectionnez **Récupérer**.

    ![Récupérer depuis Azure](./media/backup-azure-backup-sql/sqlbackup-recover.png)
3. MABS affiche les détails du point de récupération. Sélectionnez **Suivant**. Pour remplacer la base de données, sélectionnez le type de récupération **Récupérer l’instance d’origine de SQL Server**. Sélectionnez **Suivant**.

    ![Récupérer à l’emplacement d’origine](./media/backup-azure-backup-sql/sqlbackup-recoveroriginal.png)

    Dans cet exemple, MABS récupère la base de données vers une autre instance SQL Server ou dans un dossier de réseau autonome.

4. Dans l'écran **Spécifier des options de récupération** , vous pouvez sélectionner les options de récupération telles que la limitation de bande passante réseau pour limiter la bande passante utilisée par la récupération. Sélectionnez **Suivant**.

5. Dans l’écran **Résumé** , vous voyez toutes les configurations de récupération fournies jusqu’à présent. Sélectionnez **Récupérer**.

    L’état de récupération indique la base de données en cours de récupération. Vous pouvez sélectionner **Fermer** pour fermer l'Assistant et afficher la progression dans l'espace de travail **Surveillance**.

    ![Initier le processus de récupération](./media/backup-azure-backup-sql/sqlbackup-recoverying.png)

    Une fois la restauration terminée, la base de données restaurée est cohérente avec l’application.

## <a name="next-steps"></a>Étapes suivantes

Consultez l’article [Sauvegarder des fichiers et applications](backup-mabs-files-applications-azure-stack.md).
Consultez l’article [Sauvegarder SharePoint sur Azure Stack](backup-mabs-sharepoint-azure-stack.md).
