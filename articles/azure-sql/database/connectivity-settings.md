---
title: Paramètres de connectivité pour Azure SQL Database et Azure Synapse Analytics
description: Cet article explique le choix de la version de TLS (Transport Layer Security), ainsi que les paramètres de proxy ou de redirection pour Azure SQL Database et Azure Synapse Analytics.
services: sql-database
ms.service: sql-database
titleSuffix: Azure SQL Database and Azure Synapse Analytics
ms.topic: how-to
author: rohitnayakmsft
ms.author: rohitna
ms.reviewer: sstein, vanto
ms.date: 07/06/2020
ms.openlocfilehash: e3422f468d1355245fb31e8f04d5f8625f583c37
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96462180"
---
# <a name="azure-sql-connectivity-settings"></a>Paramètres de connectivité pour Azure SQL
[!INCLUDE[appliesto-sqldb-asa](../includes/appliesto-sqldb-asa.md)]

Cet article présente les paramètres qui contrôlent la connectivité au serveur pour Azure SQL Database et Azure Synapse Analytics. Ces paramètres s’appliquent à toutes les bases de données SQL Database et Azure Synapse Analytics associées au serveur.

> [!IMPORTANT]
> Il ne s’applique pas à Azure SQL Managed Instance.

Les paramètres de connectivité sont accessibles à partir de l’écran **Pare-feu et réseaux virtuels**, comme illustré dans la capture d’écran suivante :

 ![Capture d’écran de la fenêtre des paramètres de connectivité.][1]

> [!NOTE]
> Ces paramètres prennent effet immédiatement après leur application. Vos clients risquent de perdre la connexion s’ils ne satisfont pas aux exigences de chaque paramètre.

## <a name="deny-public-network-access"></a>Refuser l’accès au réseau public

Lorsque le paramètre **Refuser l’accès au réseau public** est défini sur **Oui**, seules les connexions via des points de terminaison privés sont autorisées. Lorsque ce paramètre est défini sur **Non** (par défaut), les clients peuvent se connecter à l’aide de points de terminaison publics (avec des règles de pare-feu basées sur l’adresse IP ou sur un réseau virtuel) ou privés (avec Azure Private Link), comme décrit dans la [vue d’ensemble de l’accès réseau](network-access-controls-overview.md).

 ![Diagramme montrant la connectivité selon que le paramètre Refuser l’accès au réseau public est défini sur Oui ou sur Non.][2]

Toute tentative de définition du paramètre **Refuser l’accès au réseau public** sur **Oui** sans aucun point de terminaison privé existant au niveau du serveur logique échouera avec un message d’erreur semblable au message suivant :  

```output
Error 42102
Unable to set Deny Public Network Access to Yes since there is no private endpoint enabled to access the server.
Please set up private endpoints and retry the operation.
```

> [!NOTE]
> Pour définir des règles de pare-feu de réseau virtuel sur un serveur logique déjà configuré avec des points de terminaison privés, définissez le paramètre **Refuser l’accès au réseau public** sur **Non**.

Lorsque le paramètre **Refuser l’accès au réseau public** est défini sur **Oui**, seules les connexions via des points de terminaison privés sont autorisées. Toutes les connexions via des points de terminaison publics seront refusées avec un message d’erreur semblable au message suivant :  

```output
Error 47073
An instance-specific error occurred while establishing a connection to SQL Server. 
The public network interface on this server is not accessible. 
To connect to this server, use the Private Endpoint from inside your virtual network.
```

Lorsque le paramètre **Refuser l’accès au réseau public** est défini sur **Oui**, toute tentative d’ajout ou de mise à jour des règles de pare-feu sera refusée avec un message d’erreur semblable au message suivant :

```output
Error 42101
Unable to create or modify firewall rules when public network interface for the server is disabled. 
To manage server or database level firewall rules, please enable the public network interface.
```

## <a name="change-public-network-access-via-powershell"></a>Modifier l’accès au réseau public via PowerShell

> [!IMPORTANT]
> Azure SQL Database prend toujours en charge le module PowerShell Azure Resource Manager, mais tous les développements futurs seront destinés au module Az.Sql. Pour ces cmdlets, voir [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Les arguments des commandes dans le module Az sont sensiblement identiques à ceux des modules AzureRm. Le script suivant nécessite le [module Azure PowerShell](/powershell/azure/install-az-ps).

Le script PowerShell suivant montre comment `Get` et `Set` la propriété **Accès au réseau public** au niveau du serveur :

```powershell
# Get the Public Network Access property
(Get-AzSqlServer -ServerName sql-server-name -ResourceGroupName sql-server-group).PublicNetworkAccess

# Update Public Network Access to Disabled
$SecureString = ConvertTo-SecureString "password" -AsPlainText -Force

Set-AzSqlServer -ServerName sql-server-name -ResourceGroupName sql-server-group -SqlAdministratorPassword $SecureString -PublicNetworkAccess "Disabled"
```

## <a name="change-public-network-access-via-cli"></a>Modifier l’accès au réseau public via CLI

> [!IMPORTANT]
> Tous les scripts évoqués dans cette section nécessitent l’[interface de ligne de commande Azure](/cli/azure/install-azure-cli).

### <a name="azure-cli-in-a-bash-shell"></a>Interface de ligne de commande Azure dans un interpréteur de commandes Bash

Le script CLI suivant montre comment modifier le paramètre **Accès au réseau public** dans un interpréteur de commandes Bash :

```azurecli-interactive

# Get current setting for Public Network Access
az sql server show -n sql-server-name -g sql-server-group --query "publicNetworkAccess"

# Update setting for Public Network Access
az sql server update -n sql-server-name -g sql-server-group --set publicNetworkAccess="Disabled"
```

## <a name="minimal-tls-version"></a>Version TLS minimale 

Le paramètre de version de [TLS (Transport Layer Security)](https://support.microsoft.com/help/3135244/tls-1-2-support-for-microsoft-sql-server) minimale permet aux clients de contrôler la version de TLS que leur base de données SQL utilise.

Actuellement, nous prenons en charge les versions de TLS 1.0, 1.1 et 1.2. La définition d’une version minimale de TLS garantit que les versions de TLS plus récentes sont prises en charge. Par exemple, le choix d’une version de TLS supérieure à 1.1 signifie que seules les connexions avec les versions de TLS 1.1 et 1.2 sont acceptées, les connexions avec la version de TLS 1.0 étant rejetées. Après avoir effectué un test pour vérifier que vos applications prennent en charge la version de TLS 1.2, nous vous recommandons de la définir comme version minimale. Cette version comprend des correctifs de vulnérabilités des versions précédentes et est la version la plus récente de TLS prise en charge dans Azure SQL Database.

> [!IMPORTANT]
> Par défaut, la version minimale de TLS autorise toutes les versions. Une fois que vous avez appliqué une version de TLS, il n’est plus possible de rétablir le paramétrage par défaut.

Pour les clients disposant d’applications qui reposent sur des versions antérieures de TLS, nous vous recommandons de définir la version minimale de TLS en fonction des exigences de vos applications. Pour les clients qui dépendent d’applications pour se connecter à l’aide d’une connexion non chiffrée, nous recommandons de ne pas définir de version minimale de TLS.

Pour plus d’informations, consultez [Considérations relatives au protocole TLS pour la connectivité de SQL Database](connect-query-content-reference-guide.md#tls-considerations-for-database-connectivity).

Une fois la version minimale de TLS définie, les tentatives de connexion de clients qui utilisent une version de TLS inférieure à la version minimale du serveur échouent avec l’erreur suivante :

```output
Error 47072
Login failed with invalid TLS version
```

## <a name="set-the-minimal-tls-version-via-powershell"></a>Définir la version minimale de TLS via PowerShell

> [!IMPORTANT]
> Azure SQL Database prend toujours en charge le module PowerShell Azure Resource Manager, mais tous les développements futurs seront destinés au module Az.Sql. Pour ces cmdlets, voir [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Les arguments des commandes dans le module Az sont sensiblement identiques à ceux des modules AzureRm. Le script suivant nécessite le [module Azure PowerShell](/powershell/azure/install-az-ps).

Le script PowerShell suivant montre comment `Get` et `Set` la propriété **Version TLS minimale** au niveau du serveur logique :

```powershell
# Get the Minimal TLS Version property
(Get-AzSqlServer -ServerName sql-server-name -ResourceGroupName sql-server-group).MinimalTlsVersion

# Update Minimal TLS Version to 1.2
$SecureString = ConvertTo-SecureString "password" -AsPlainText -Force

Set-AzSqlServer -ServerName sql-server-name -ResourceGroupName sql-server-group -SqlAdministratorPassword $SecureString  -MinimalTlsVersion "1.2"
```

## <a name="set-the-minimal-tls-version-via-the-azure-cli"></a>Définir la version minimale de TLS via l’interface de ligne de commande Azure

> [!IMPORTANT]
> Tous les scripts évoqués dans cette section nécessitent l’[interface de ligne de commande Azure](/cli/azure/install-azure-cli).

### <a name="azure-cli-in-a-bash-shell"></a>Interface de ligne de commande Azure dans un interpréteur de commandes Bash

Le script CLI suivant montre comment modifier le paramètre de **version minimale de TLS** dans un interpréteur de commandes Bash :

```azurecli-interactive
# Get current setting for Minimal TLS Version
az sql server show -n sql-server-name -g sql-server-group --query "minimalTlsVersion"

# Update setting for Minimal TLS Version
az sql server update -n sql-server-name -g sql-server-group --set minimalTlsVersion="1.2"
```

## <a name="change-the-connection-policy"></a>Modifier la stratégie de connexion

La [stratégie de connexion](connectivity-architecture.md#connection-policy) détermine la façon dont les clients se connectent à Azure SQL Database.

## <a name="change-the-connection-policy-via-powershell"></a>Modifier la stratégie de connexion via PowerShell

> [!IMPORTANT]
> Azure SQL Database prend toujours en charge le module PowerShell Azure Resource Manager, mais tous les développements futurs seront destinés au module Az.Sql. Pour ces cmdlets, voir [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Les arguments des commandes dans le module Az sont sensiblement identiques à ceux des modules AzureRm. Le script suivant nécessite le [module Azure PowerShell](/powershell/azure/install-az-ps).

Le script PowerShell suivant montre comment modifier la stratégie de connexion à l’aide de PowerShell :

```powershell
# Get SQL Server ID
$sqlserverid=(Get-AzSqlServer -ServerName sql-server-name -ResourceGroupName sql-server-group).ResourceId

# Set URI
$id="$sqlserverid/connectionPolicies/Default"

# Get current connection policy
(Get-AzResource -ResourceId $id -ApiVersion 2014-04-01 -Verbose).Properties.ConnectionType

# Update connection policy
Set-AzResource -ResourceId $id -Properties @{"connectionType" = "Proxy"} -f
```

## <a name="change-the-connection-policy-via-the-azure-cli"></a>Modifier la stratégie de connexion via l’interface de ligne de commande Azure

> [!IMPORTANT]
> Tous les scripts évoqués dans cette section nécessitent l’[interface de ligne de commande Azure](/cli/azure/install-azure-cli).

### <a name="azure-cli-in-a-bash-shell"></a>Interface de ligne de commande Azure dans un interpréteur de commandes Bash

Le script CLI suivant montre comment modifier la stratégie de connexion dans un interpréteur de commandes Bash :

```azurecli-interactive
# Get SQL Server ID
sqlserverid=$(az sql server show -n sql-server-name -g sql-server-group --query 'id' -o tsv)

# Set URI
ids="$sqlserverid/connectionPolicies/Default"

# Get current connection policy
az resource show --ids $ids

# Update connection policy
az resource update --ids $ids --set properties.connectionType=Proxy
```

### <a name="azure-cli-from-a-windows-command-prompt"></a>Azure CLI à partir d’une invite de commandes Windows

Le script d’interface de ligne de commande suivant montre comment modifier la stratégie de connexion à partir d’une invite de commandes Windows (avec l’interface de ligne de commande Azure installée) :

```azurecli
# Get SQL Server ID and set URI
FOR /F "tokens=*" %g IN ('az sql server show --resource-group myResourceGroup-571418053 --name server-538465606 --query "id" -o tsv') do (SET sqlserverid=%g/connectionPolicies/Default)

# Get current connection policy
az resource show --ids %sqlserverid%

# Update connection policy
az resource update --ids %sqlserverid% --set properties.connectionType=Proxy
```

## <a name="next-steps"></a>Étapes suivantes

- Pour obtenir une vue d’ensemble du fonctionnement de la connectivité dans Azure SQL Database, consultez [Architecture de connectivité](connectivity-architecture.md).
- Pour plus d’informations sur la façon de modifier la stratégie de connexion d’un serveur, consultez [conn-policy](/cli/azure/sql/server/conn-policy).

<!--Image references-->
[1]: media/single-database-create-quickstart/manage-connectivity-settings.png
[2]: media/single-database-create-quickstart/manage-connectivity-flowchart.png
