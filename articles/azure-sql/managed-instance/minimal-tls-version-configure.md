---
title: Configurer un version TLS minimale - instance gérée
description: Découvrez comment configurer un version TLS minimale pour les instances gérées
services: sql-database
ms.service: sql-managed-instance
ms.subservice: security
ms.custom: ''
ms.topic: how-to
author: srdan-bozovic-msft
ms.author: srbozovi
ms.reviewer: ''
ms.date: 05/25/2020
ms.openlocfilehash: 17d430946f3cba1aa4680d1eaf8979fa4338bc22
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92788398"
---
# <a name="configure-minimal-tls-version-in-azure-sql-managed-instance"></a>Configurer un version TLS minimale dans Azure SQL Managed Instance
Le paramètre de version minimale [TLS (Transport Layer Security)](https://support.microsoft.com/help/3135244/tls-1-2-support-for-microsoft-sql-server) permet aux clients de contrôler la version de TLS utilisée par leur instance Azure SQL Managed Instance.

À l’heure actuelle, nous prenons en charge TLS 1.0, 1.1 et 1.2. La définition d’une version TLS minimale garantit que les versions TLS ultérieures plus récentes sont prises en charge. Par exemple, si le choix se porte sur une version TLS supérieure à 1.1, seules les connexions avec TLS 1.1 et 1.2 sont acceptées et TLS 1.0 est rejeté. Après le test pour confirmer que vos applications la prennent en charge, nous vous recommandons de définir la version TLS minimale sur 1.2, car elle comprend des correctifs pour les vulnérabilités détectées dans les versions précédentes et est la version la plus récente de TLS prise en charge dans Azure SQL Managed Instance.

Pour les clients disposant d’applications qui reposent sur des versions antérieures de TLS, nous vous recommandons de définir la version TLS minimale conformément aux exigences de vos applications. Pour les clients qui s’appuient sur des applications pour se connecter à l’aide d’une connexion non chiffrée, nous vous recommandons de ne pas définir de version TLS minimale. 

Pour plus d’informations, consultez [Considérations relatives au protocole TLS pour la connectivité de SQL Database](../database/connect-query-content-reference-guide.md#tls-considerations-for-database-connectivity).

Une fois définie la version TLS minimale, les tentatives de connexion à partir de clients qui utilisent une version TLS inférieure à la version TLS minimale du serveur échouent avec l’erreur suivante :

```output
Error 47072
Login failed with invalid TLS version
```

## <a name="set-minimal-tls-version-via-powershell"></a>Définir la version TLS minimale par le biais de PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]
> [!IMPORTANT]
> Le module PowerShell Azure Resource Manager est toujours pris en charge par Azure SQL Database, mais tous les développements futurs sont destinés au module Az.Sql. Pour ces cmdlets, voir [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Les arguments des commandes dans le module Az sont sensiblement identiques à ceux des modules AzureRm. Le script suivant nécessite le [module Azure PowerShell](/powershell/azure/install-az-ps).

Le script PowerShell suivant montre comment `Get` et `Set` la propriété **Version TLS minimale** au niveau de l’instance :

```powershell
#Get the Minimal TLS Version property
(Get-AzSqlInstance -Name sql-instance-name -ResourceGroupName resource-group).MinimalTlsVersion

# Update Minimal TLS Version Property
Set-AzSqlInstance -Name sql-instance-name -ResourceGroupName resource-group -MinimalTlsVersion "1.2"
```

## <a name="set-minimal-tls-version-via-azure-cli"></a>Définir la version TLS minimale par le biais d’Azure CLI

> [!IMPORTANT]
> Tous les scripts de cette section nécessitent [Azure CLI](/cli/azure/install-azure-cli).

### <a name="azure-cli-in-a-bash-shell"></a>Azure CLI dans un interpréteur de commandes Bash

Le script CLI suivant montre comment changer le paramètre **Version TLS minimale** dans un interpréteur de commandes Bash :

```azurecli-interactive
# Get current setting for Minimal TLS Version
az sql mi show -n sql-instance-name -g resource-group --query "minimalTlsVersion"

# Update setting for Minimal TLS Version
az sql mi update -n sql-instance-name -g resource-group --set minimalTlsVersion="1.2"
```