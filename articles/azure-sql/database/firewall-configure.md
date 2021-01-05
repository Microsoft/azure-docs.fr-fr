---
title: Règles de pare-feu IP
description: Configurez des règles de pare-feu IP au niveau serveur pour une base de données dans Azure SQL Database ou un pare-feu Azure Synapse Analytics. Gérez l’accès et configurez des règles de pare-feu IP au niveau de la base de données pour SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: security
titleSuffix: Azure SQL Database and Azure Synapse Analytics
ms.custom: sqldbrb=1, devx-track-azurecli
ms.devlang: ''
ms.topic: conceptual
author: VanMSFT
ms.author: vanto
ms.reviewer: sstein
ms.date: 06/17/2020
ms.openlocfilehash: ce528e268e0ed1e34f53e32196bceef5ad8a2fcb
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96452494"
---
# <a name="azure-sql-database-and-azure-synapse-ip-firewall-rules"></a>Règles de pare-feu IP Azure SQL Database et Azure Synapse
[!INCLUDE[appliesto-sqldb-asa](../includes/appliesto-sqldb-asa.md)]

Lorsque vous créez dans Azure SQL Database ou Azure Synapse Analytics un serveur nommé *mysqlserver*, par exemple, un pare-feu au niveau du serveur bloque tout accès au point de terminaison public du serveur (accessible à l'adresse *mysqlserver.database.windows.net*). Par souci de simplicité, le nom *SQL Database* est utilisé pour désigner à la fois SQL Database et Azure Synapse Analytics.

> [!IMPORTANT]
> Cet article ne s’applique *pas* à *Azure SQL Managed Instance*. Pour plus d'informations sur la configuration réseau, consultez [Connecter votre application à Azure SQL Managed Instance](../managed-instance/connect-application-instance.md).
>
> Azure Synapse ne prend en charge que les règles de pare-feu IP au niveau serveur. Il ne prend pas en charge les règles de pare-feu IP au niveau de la base de données.

## <a name="how-the-firewall-works"></a>Fonctionnement du pare-feu

Les tentatives de connexion à partir d'Internet et d'Azure doivent franchir le pare-feu avant d'atteindre votre serveur ou votre base de données, comme illustré dans le schéma suivant.

   ![Schéma illustrant la configuration du pare-feu][1]

### <a name="server-level-ip-firewall-rules"></a>Règles de pare-feu IP au niveau du serveur

Ces règles permettent aux clients d'accéder à l'ensemble de votre serveur, c'est-à-dire à toutes les bases de données gérées par le serveur. Les règles sont stockées dans la base de données *MASTER*. Vous pouvez avoir un maximum de 128 règles de pare-feu IP au niveau du serveur par serveur. Si le paramètre **Autoriser les services et les ressources Azure à accéder à ce serveur** est activé, cela compte comme une seule règle de pare-feu pour le serveur.
  
Vous pouvez configurer des règles de pare-feu IP au niveau du serveur à l’aide du portail Azure, de PowerShell ou d’instructions Transact-SQL.

- Pour utiliser le portail ou PowerShell, vous devez être le propriétaire ou un contributeur de l’abonnement.
- Pour utiliser Transact-SQL, vous devez vous connecter à la base de données *MASTER* avec un compte principal au niveau du serveur ou en tant qu'administrateur Azure Active Directory. (Une règle de pare-feu IP au niveau du serveur doit d’abord être créée par un utilisateur disposant d’autorisations au niveau d’Azure.)

### <a name="database-level-ip-firewall-rules"></a>Règles de pare-feu IP au niveau de la base de données

Ces règles de pare-feu IP au niveau de la base de données permettent aux clients d'accéder à certaines bases de données (sécurisées). Vous devez créer les règles pour chaque base de données (dont la base de données *MASTER*). Elles y seront stockées de façon individuelle.
  
- Vous pouvez créer et gérer des règles de pare-feu IP au niveau de la base de données pour les bases MASTER et utilisateur uniquement à l’aide d’instructions Transact-SQL et seulement après avoir configuré le premier pare-feu au niveau du serveur.
- Si vous spécifiez dans la règle de pare-feu IP au niveau de la base de données une plage d’adresses IP en dehors de la plage spécifiée dans la règle de pare-feu IP au niveau du serveur, seuls les clients dont les adresses IP appartiennent à la plage au niveau de la base de données peuvent accéder à la base de données.
- Vous pouvez avoir un maximum de 128 règles de pare-feu IP au niveau de la base de données par base de données. Pour plus d’informations sur la configuration des règles de pare-feu IP au niveau de la base de données, consultez l’exemple plus loin dans cet article et [sp_set_database_firewall_rule (Azure SQL Database)](/sql/relational-databases/system-stored-procedures/sp-set-database-firewall-rule-azure-sql-database).

### <a name="recommendations-for-how-to-set-firewall-rules"></a>Recommandations sur la définition des règles de pare-feu

Nous vous recommandons d’utiliser des règles de pare-feu IP au niveau de la base de données autant que possible. Cette pratique améliore la sécurité et la portabilité de votre base de données. Utilisez des règles de pare-feu IP au niveau du serveur pour les administrateurs. Utilisez également ces règles quand de nombreuses bases de données ont les mêmes exigences d’accès et que vous ne souhaitez pas les configurer une à une.

> [!NOTE]
> Pour plus d’informations sur les bases de données portables dans le cadre de la continuité d’activité, consultez [Exigences d’authentification pour la récupération d’urgence](active-geo-replication-security-configure.md).

## <a name="server-level-versus-database-level-ip-firewall-rules"></a>Règles de pare-feu IP au niveau du serveur et au niveau de la base de données

*Les utilisateurs d’une base de données doivent-ils être totalement isolés d’une autre base de données ?*

*Si c’est le cas*, accordez l’accès à l’aide de règles de pare-feu IP au niveau de la base de données. Cette méthode évite d’utiliser des règles de pare-feu IP au niveau du serveur, qui permettent à toutes les bases de données de franchir le pare-feu et réduisent ainsi le champ de vos défenses.

*Les utilisateurs aux adresses IP ont-ils besoin d’accéder à toutes les bases de données ?*

*Si c’est le cas*, utilisez des règles de pare-feu IP au niveau du serveur pour réduire le nombre de règles de pare-feu IP à configurer.

*La personne ou l’équipe qui configure les règles de pare-feu IP a-t-elle seulement accès par le biais du portail Azure, de PowerShell ou de l’API REST ?*

Si c’est le cas, vous devez utiliser des règles de pare-feu IP au niveau du serveur. Les règles de pare-feu IP au niveau de la base de données peuvent être configurées seulement avec Transact-SQL.  

*La personne ou l’équipe qui configure les règles de pare-feu IP a-t-elle l’interdiction d’avoir une autorisation générale au niveau de la base de données ?*

Si c’est le cas, utilisez des règles de pare-feu IP au niveau du serveur. Vous devez disposer au minimum de l’autorisation *CONTROL DATABASE* au niveau de la base de données pour configurer des règles de pare-feu IP au niveau de la base de données à l’aide de Transact-SQL.  

*La personne ou l’équipe qui configure ou vérifie les règles de pare-feu IP gère-t-elle ces règles de façon centralisée pour un grand nombre de bases de données (par exemple, plusieurs centaines) ?*

Si c’est le cas, vos besoins et votre environnement déterminent les bonnes pratiques à mettre en œuvre. Les règles de pare-feu IP au niveau du serveur peuvent être plus faciles à configurer, mais des scripts peuvent configurer les règles au niveau de la base de données. Par ailleurs, même si vous utilisez des règles de pare-feu IP au niveau du serveur, vous devez peut-être vérifier les règles de pare-feu IP au niveau de la base de données pour voir si des utilisateurs avec l’autorisation *CONTROL* sur la base de données créent des règles de pare-feu IP au niveau de la base de données.

*Puis-je utiliser à la fois des règles de pare-feu IP au niveau du serveur et au niveau de la base de données ?*

Oui. Certains utilisateurs comme les administrateurs peuvent avoir besoin de règles de pare-feu IP au niveau du serveur. D’autres utilisateurs, comme les utilisateurs d’une application de base de données, peuvent avoir besoin de règles de pare-feu IP au niveau de la base de données.

### <a name="connections-from-the-internet"></a>Connexions à partir d’Internet

Lorsqu'un ordinateur tente de se connecter à votre serveur à partir d'Internet, le pare-feu vérifie d'abord l'adresse IP d'origine de la demande par rapport aux règles de pare-feu IP au niveau de la base de données pour la base de données demandée par la connexion.

- Si l'adresse est comprise dans une plage spécifiée dans les règles de pare-feu IP au niveau de la base de données, la connexion à la base de données contenant la règle est accordée.
- Dans le cas contraire, le pare-feu vérifie les règles de pare-feu IP au niveau du serveur. Si l’adresse est comprise dans une plage spécifiée dans les règles de pare-feu IP au niveau du serveur, la connexion est accordée. Les règles de pare-feu IP au niveau du serveur s'appliquent à toutes les bases de données gérées par le serveur.  
- Si l’adresse n’est pas comprise dans une plage spécifiée dans les règles de pare-feu IP au niveau du serveur ou de la base de données, la demande de connexion échoue.

> [!NOTE]
> Pour accéder à Azure SQL Database à partir de votre ordinateur local, vérifiez que le pare-feu de votre réseau et de votre ordinateur local autorise les communications sortantes sur le port TCP 1433.

### <a name="connections-from-inside-azure"></a>Connexions à partir d’Azure

Pour autoriser des applications hébergées dans Azure à se connecter à votre serveur SQL, vous devez activer les connexions Azure. Lorsqu'une application Azure tente de se connecter à votre serveur, le pare-feu vérifie que les connexions Azure sont autorisées. Vous pouvez activer la connexion directement à partir du panneau du portail Azure en définissant des règles de pare-feu, ainsi qu'en définissant **Autoriser les services et les ressources Azure à accéder à ce serveur** sur **Activé** dans les paramètres **Pare-feux et réseaux virtuels**. Si la connexion n'est pas autorisée, la demande n'atteint pas le serveur.

> [!IMPORTANT]
> Cette option configure le pare-feu pour autoriser toutes les connexions à partir d’Azure, notamment les connexions issues des abonnements d’autres clients. Si vous sélectionnez cette option, vérifiez que votre connexion et vos autorisations utilisateur limitent l’accès aux seuls utilisateurs autorisés.

## <a name="permissions"></a>Autorisations

Pour pouvoir créer et gérer des règles de pare-feu IP pour Azure SQL Server, vous devez être :

- dans le rôle [Contributeur de SQL Server](../../role-based-access-control/built-in-roles.md#sql-server-contributor)
- dans le rôle [Gestionnaire de sécurité SQL](../../role-based-access-control/built-in-roles.md#sql-security-manager)
- propriétaire de la ressource qui contient SQL Server Azure

## <a name="create-and-manage-ip-firewall-rules"></a>Créer et gérer des règles de pare-feu IP

Vous devez créer le premier paramètre de pare-feu au niveau du serveur à l’aide du [portail Azure](https://portal.azure.com/) ou par programmation avec [Azure PowerShell](/powershell/module/az.sql), l’[interface de ligne de commande Azure](/cli/azure/sql/server/firewall-rule) ou une [API REST](/rest/api/sql/firewallrules/createorupdate) Azure. Vous pouvez utiliser ces méthodes ou Transact-SQL pour créer et gérer d’autres règles de pare-feu IP au niveau du serveur.

> [!IMPORTANT]
> Les règles de pare-feu IP au niveau de la base de données peuvent être créées et gérées seulement avec Transact-SQL.

Pour améliorer les performances, les règles de pare-feu IP au niveau du serveur sont temporairement mises en cache au niveau de la base de données. Pour actualiser le cache, consultez [DBCC FLUSHAUTHCACHE](/sql/t-sql/database-console-commands/dbcc-flushauthcache-transact-sql).

> [!TIP]
> Vous pouvez utiliser l'[Audit de la base de données](../../azure-sql/database/auditing-overview.md) pour vérifier des modifications de pare-feu au niveau serveur et au niveau base de données.

### <a name="use-the-azure-portal-to-manage-server-level-ip-firewall-rules"></a>Gérer des règles de pare-feu IP au niveau du serveur avec le portail Azure

Pour définir une règle de pare-feu IP au niveau du serveur sur le portail Azure, accédez à la page de présentation de votre base de données ou de votre serveur.

> [!TIP]
> Pour accéder à un tutoriel, consultez [Créer une base de données à l'aide du portail Azure](single-database-create-quickstart.md).

#### <a name="from-the-database-overview-page"></a>À partir de la page de présentation de la base de données

1. Pour définir une règle de pare-feu IP au niveau du serveur à partir de la page de présentation de la base de données, cliquez sur **Définir le pare-feu du serveur** dans la barre d’outils comme illustré dans l’image suivante.

    ![Règle de pare-feu IP de serveur](./media/firewall-configure/sql-database-server-set-firewall-rule.png)

    La page **Paramètres de pare-feu** du serveur s'ouvre.

2. Cliquez sur **Ajouter une adresse IP cliente** dans la barre d’outils pour ajouter l’adresse IP de l’ordinateur que vous utilisez, puis cliquez sur **Enregistrer**. Une règle de pare-feu IP au niveau du serveur est créée pour votre adresse IP actuelle.

    ![Règle de pare-feu IP au niveau du serveur définie](./media/firewall-configure/sql-database-server-firewall-settings.png)

#### <a name="from-the-server-overview-page"></a>À partir de la page de présentation du serveur

La page de présentation de votre serveur s’ouvre. Elle affiche le nom de serveur complet (tel que *mynewserver20170403.database.windows.net*) et fournit des options permettant de poursuivre la configuration.

1. Pour définir une règle au niveau du serveur à partir de cette page, sélectionnez **Pare-feu** dans le menu **Paramètres** sur le côté gauche.

2. Cliquez sur **Ajouter une adresse IP cliente** dans la barre d’outils pour ajouter l’adresse IP de l’ordinateur que vous utilisez, puis cliquez sur **Enregistrer**. Une règle de pare-feu IP au niveau du serveur est créée pour votre adresse IP actuelle.

### <a name="use-transact-sql-to-manage-ip-firewall-rules"></a>Gérer des règles de pare-feu IP avec Transact-SQL

| Vue de catalogue ou procédure stockée | Level | Description |
| --- | --- | --- |
| [sys.firewall_rules](/sql/relational-databases/system-catalog-views/sys-firewall-rules-azure-sql-database) |Serveur |Affiche les règles de pare-feu IP au niveau du serveur actuelles |
| [sp_set_firewall_rule](/sql/relational-databases/system-stored-procedures/sp-set-firewall-rule-azure-sql-database) |Serveur |Crée ou met à jour les règles de pare-feu IP au niveau du serveur |
| [sp_delete_firewall_rule](/sql/relational-databases/system-stored-procedures/sp-delete-firewall-rule-azure-sql-database) |Serveur |Supprime des règles de pare-feu IP au niveau du serveur |
| [sys.database_firewall_rules](/sql/relational-databases/system-catalog-views/sys-database-firewall-rules-azure-sql-database) |Base de données |Affiche les règles de pare-feu IP actuelles au niveau de la base de données |
| [sp_set_database_firewall_rule](/sql/relational-databases/system-stored-procedures/sp-set-database-firewall-rule-azure-sql-database) |Base de données |Crée ou met à jour les règles de pare-feu IP au niveau de la base de données |
| [sp_delete_database_firewall_rule](/sql/relational-databases/system-stored-procedures/sp-delete-database-firewall-rule-azure-sql-database) |Bases de données |Supprime des règles de pare-feu IP au niveau de la base de données |

L’exemple suivant examine les règles existantes, active une plage d’adresses IP sur le serveur *Contoso* et supprime une règle de pare-feu IP :

```sql
SELECT * FROM sys.firewall_rules ORDER BY name;
```

Ensuite, ils ajoutent une règle de pare-feu IP au niveau du serveur.

```sql
EXECUTE sp_set_firewall_rule @name = N'ContosoFirewallRule',
   @start_ip_address = '192.168.1.1', @end_ip_address = '192.168.1.10'
```

Pour supprimer une règle de pare-feu IP au niveau du serveur, exécutez la procédure stockée *sp_delete_firewall_rule*. L’exemple suivant supprime la règle *ContosoFirewallRule* :

```sql
EXECUTE sp_delete_firewall_rule @name = N'ContosoFirewallRule'
```

### <a name="use-powershell-to-manage-server-level-ip-firewall-rules"></a>Gérer des règles de pare-feu IP au niveau du serveur avec PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]
> [!IMPORTANT]
> Le module PowerShell Azure Resource Manager est toujours pris en charge par Azure SQL Database, mais tous les développements sont désormais destinés au module Az.Sql. Pour ces cmdlets, voir [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Les arguments des commandes dans les modules Az et AzureRm sont sensiblement identiques.

| Applet de commande | Level | Description |
| --- | --- | --- |
| [Get-AzSqlServerFirewallRule](/powershell/module/az.sql/get-azsqlserverfirewallrule) |Serveur |Retourne les règles de pare-feu au niveau du serveur actuelles |
| [New-AzSqlServerFirewallRule](/powershell/module/az.sql/new-azsqlserverfirewallrule) |Serveur |Crée une règle de pare-feu au niveau du serveur |
| [Set-AzSqlServerFirewallRule](/powershell/module/az.sql/set-azsqlserverfirewallrule) |Serveur |Met à jour les propriétés d’une règle de pare-feu au niveau du serveur existante |
| [Remove-AzSqlServerFirewallRule](/powershell/module/az.sql/remove-azsqlserverfirewallrule) |Serveur |Supprime des règles de pare-feu au niveau du serveur |

L’exemple suivant définit une règle de pare-feu IP au niveau du serveur à l’aide de PowerShell :

```powershell
New-AzSqlServerFirewallRule -ResourceGroupName "myResourceGroup" `
    -ServerName $servername `
    -FirewallRuleName "ContosoIPRange" -StartIpAddress "192.168.1.0" -EndIpAddress "192.168.1.255"
```

> [!TIP]
> Pour $servername, spécifiez le nom du serveur et non le nom DNS complet. Par exemple, spécifiez **mysqldbserver** au lieu de **mysqldbserver.database.windows.net**
>
> Pour obtenir des exemples PowerShell dans le contexte d'un démarrage rapide, consultez [Créer une base de données - PowerShell](powershell-script-content-guide.md) et [Utiliser PowerShell pour créer une base de données unique et configurer une règle de pare-feu IP au niveau du serveur](scripts/create-and-configure-database-powershell.md).

### <a name="use-cli-to-manage-server-level-ip-firewall-rules"></a>Gérer des règles de pare-feu IP au niveau du serveur avec l’interface de ligne de commande

| Applet de commande | Level | Description |
| --- | --- | --- |
|[az sql server firewall-rule create](/cli/azure/sql/server/firewall-rule#az-sql-server-firewall-rule-create)|Serveur|Crée une règle de pare-feu IP de serveur|
|[az sql server firewall-rule list](/cli/azure/sql/server/firewall-rule#az-sql-server-firewall-rule-list)|Serveur|Liste les règles de pare-feu IP sur un serveur|
|[az sql server firewall-rule show](/cli/azure/sql/server/firewall-rule#az-sql-server-firewall-rule-show)|Serveur|Affiche les détails d’une règle de pare-feu IP|
|[az sql server firewall-rule update](/cli/azure/sql/server/firewall-rule##az-sql-server-firewall-rule-update)|Serveur|Met à jour une règle de pare-feu IP|
|[az sql server firewall-rule delete](/cli/azure/sql/server/firewall-rule#az-sql-server-firewall-rule-delete)|Serveur|Supprime une règle de pare-feu IP|

L’exemple suivant définit une règle de pare-feu IP au niveau du serveur à l’aide de l’interface de ligne de commande :

```azurecli-interactive
az sql server firewall-rule create --resource-group myResourceGroup --server $servername \
-n ContosoIPRange --start-ip-address 192.168.1.0 --end-ip-address 192.168.1.255
```

> [!TIP]
> Pour $servername, spécifiez le nom du serveur et non le nom DNS complet. Par exemple, spécifiez **mysqldbserver** au lieu de **mysqldbserver.database.windows.net**
>
> Pour obtenir un exemple basé sur l'interface de ligne de commande dans le contexte d'un démarrage rapide, consultez [Créer une base de données - Azure CLI](az-cli-script-samples-content-guide.md) et [Utiliser Azure CLI pour créer une base de données unique et configurer une règle de pare-feu IP au niveau du serveur](scripts/create-and-configure-database-cli.md).

### <a name="use-a-rest-api-to-manage-server-level-ip-firewall-rules"></a>Gérer des règles de pare-feu IP au niveau du serveur avec une API REST

| API | Level | Description |
| --- | --- | --- |
| [Règles de pare-feu - List by Server](/rest/api/sql/firewallrules/listbyserver) |Serveur |Affiche les règles de pare-feu IP au niveau du serveur actuelles |
| [Règles de pare-feu - Create ou Update](/rest/api/sql/firewallrules/createorupdate) |Serveur |Crée ou met à jour les règles de pare-feu IP au niveau du serveur |
| [Règles de pare-feu - Delete](/rest/api/sql/firewallrules/delete) |Serveur |Supprime des règles de pare-feu IP au niveau du serveur |
| [Règles de pare-feu - Get](/rest/api/sql/firewallrules/get) | Serveur | Obtient les règles de pare-feu IP au niveau du serveur |

## <a name="troubleshoot-the-database-firewall"></a>Résoudre les problèmes liés au pare-feu de base de données

Tenez compte des points suivants lorsque l'accès à Azure SQL Database est anormal.

- **Configuration du pare-feu local :**

  Pour que votre ordinateur puisse accéder à Azure SQL Database, vous devez créer une exception de pare-feu sur votre ordinateur pour le port TCP 1433. Vous devrez peut-être ouvrir des ports supplémentaires pour effectuer des connexions dans la limite du cloud Azure. Pour plus d’informations, consultez la section « SQL Database : exécution externe ou exécution interne de [Ports au-delà de 1433 pour ADO.NET 4.5 et Azure SQL Database](adonet-v12-develop-direct-route-ports.md).

- **Traduction d’adresses réseau :**

  En raison du protocole de traduction d'adresses réseau (NAT, network address translation), l'adresse IP utilisée par votre ordinateur pour se connecter à Azure SQL Database peut être différente de celle indiquée dans les paramètres de configuration IP de votre ordinateur. Pour voir l’adresse IP utilisée par votre ordinateur pour se connecter à Azure :
    1. Connectez-vous au portail.
    1. Accédez à l’onglet **Configurer** du serveur qui héberge votre base de données.
    1. L’**adresse IP du client actif** s’affiche dans la section **Adresses IP autorisées**. Sélectionnez **Ajouter** sous **Adresses IP autorisées** pour que cet ordinateur puisse accéder au serveur.

- **Les modifications apportées à la liste verte n’ont pas encore été prises en compte :**

  Cinq minutes peuvent s'écouler avant que les modifications apportées à la configuration du pare-feu Azure SQL Database soient prises en compte.

- **La connexion n’est pas autorisée ou un mot de passe incorrect a été utilisé :**

  Si une connexion n'a pas d'autorisations sur le serveur ou que le mot de passe est incorrect, la connexion au serveur est refusée. La création d’un paramètre de pare-feu permet uniquement aux clients d’*essayer* de se connecter à votre serveur. Ils doivent toujours fournir les informations d’identification de sécurité nécessaires. Pour plus d'informations sur la préparation des connexions, consultez [Contrôle et octroi de l'accès à la base de données](logins-create-manage.md).

- **Adresse IP dynamique :**

  Si votre connexion Internet utilise un adressage IP dynamique et que le pare-feu demeure infranchissable, essayez l’une des solutions suivantes :
  
  - Demandez à votre fournisseur de services Internet la plage d'adresses IP attribuée aux ordinateurs clients qui accèdent au serveur. Ajoutez cette plage d’adresses IP en tant que règle de pare-feu IP.
  - Récupérez plutôt l’adressage IP statique pour vos ordinateurs clients. Ajoutez les adresses IP en tant que règles de pare-feu IP.

## <a name="next-steps"></a>Étapes suivantes

- Vérifiez que votre environnement de réseau d’entreprise autorise les communications entrantes issues des plages d’adresses IP de calcul (y compris les plages SQL) utilisées par les centres de données Azure. Vous devrez peut-être ajouter ces adresses IP à la liste verte. Consultez [Plages IP de centres de données Microsoft Azure](https://www.microsoft.com/download/details.aspx?id=41653).  
- Pour accéder à un guide de démarrage rapide sur la création d'une règle de pare-feu IP au niveau du serveur, consultez [Créer une base de données unique dans Azure SQL Database](single-database-create-quickstart.md).
- Pour obtenir de l'aide sur la connexion à une base de données dans Azure SQL Database à partir d'applications open source ou tierces, consultez [Exemples de code de démarrage rapide du client pour Azure SQL Database](connect-query-content-reference-guide.md#libraries).
- Pour plus d’informations sur les autres ports que vous devrez peut-être ouvrir, consultez la section « SQL Database : exécution externe ou exécution interne de [Ports au-delà de 1433 pour ADO.NET 4.5 et SQL Database](adonet-v12-develop-direct-route-ports.md).
- Pour obtenir une vue d’ensemble de la sécurité Azure SQL Database, consultez [Sécurisation de votre base de données](security-overview.md).

<!--Image references-->
[1]: ./media/firewall-configure/sqldb-firewall-1.png
