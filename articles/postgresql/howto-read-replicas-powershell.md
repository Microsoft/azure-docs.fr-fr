---
title: Gérer des réplicas en lecture - Azure PowerShell - Azure Database pour PostgreSQL
description: Découvrez comment configurer et gérer des réplicas en lecture dans Azure Database pour PostgreSQL à l’aide de PowerShell.
author: sr-msft
ms.author: srranga
ms.service: postgresql
ms.topic: how-to
ms.date: 06/08/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 8bacb7a434cfa04dbdfdaf39d9fd3a0baab5f11a
ms.sourcegitcommit: 3bcce2e26935f523226ea269f034e0d75aa6693a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92489810"
---
# <a name="how-to-create-and-manage-read-replicas-in-azure-database-for-postgresql-using-powershell"></a>Guide pratique pour créer et gérer des réplicas en lecture dans Azure Database pour PostgreSQL à l’aide de PowerShell

Dans cet article, apprenez à créer et à gérer des réplicas en lecture dans le service Azure Database pour PostgreSQL à l’aide de PowerShell. Pour en savoir plus sur les réplicas en lecture, consultez [vue d’ensemble](concepts-read-replicas.md).

## <a name="azure-powershell"></a>Azure PowerShell

Vous pouvez créer et gérer des réplicas en lecture à l’aide de PowerShell.

## <a name="prerequisites"></a>Prérequis

Pour utiliser ce guide pratique, il vous faut :

- Le [module Az PowerShell](/powershell/azure/install-az-ps) installé localement ou [Azure Cloud Shell](https://shell.azure.com/) dans le navigateur
- Un [serveur Azure Database pour PostgreSQL](quickstart-create-postgresql-server-database-using-azure-powershell.md)

> [!IMPORTANT]
> Tant que le module Az.PostgreSql PowerShell est en préversion, vous devez l’installer séparément du module Az PowerShell à l’aide de la commande suivante : `Install-Module -Name Az.PostgreSql -AllowPrerelease`
> Une fois que le module Az.PostgreSql PowerShell est en disponibilité générale, il devient partie intégrante des versions ultérieures du module Az PowerShell et est disponible en mode natif dans Azure Cloud Shell.

Si vous choisissez d’utiliser PowerShell en local, connectez-vous à votre compte Azure à l’aide de la cmdlet [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount).

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

> [!IMPORTANT]
> La fonctionnalité de réplica en lecture est disponible uniquement pour les serveurs Azure Database pour PostgreSQL dans les niveaux tarifaires Usage général ou Mémoire optimisée. Vérifiez que le serveur principal se trouve dans l’un de ces niveaux tarifaires.

### <a name="create-a-read-replica"></a>Créer un réplica en lecture

Un serveur réplica en lecture peut être créé en utilisant la commande suivante :

```azurepowershell-interactive
Get-AzPostgreSqlServer -Name mydemoserver -ResourceGroupName myresourcegroup |
  New-AzPostgreSqlServerReplica -Name mydemoreplicaserver -ResourceGroupName myresourcegroup
```

La commande `New-AzPostgreSqlServerReplica` requiert les paramètres suivants :

| Paramètre | Valeur d'exemple | Description  |
| --- | --- | --- |
| ResourceGroupName |  myResourceGroup |  Groupe de ressources où le serveur réplica est créé.  |
| Nom | mydemoreplicaserver | Nom du nouveau serveur réplica créé. |

Pour créer un réplica en lecture entre régions, utilisez le paramètre **Location** . L’exemple suivant crée un réplica dans la région **USA Ouest** .

```azurepowershell-interactive
Get-AzPostgreSqlServer -Name mrdemoserver -ResourceGroupName myresourcegroup |
  New-AzMariaDServerReplica -Name mydemoreplicaserver -ResourceGroupName myresourcegroup -Location westus
```

Pour en savoir plus sur les régions dans lesquelles vous pouvez créer un réplica, consultez l’article [Concepts relatifs aux réplicas en lecture](concepts-read-replicas.md).

Par défaut, les réplicas en lecture sont créés avec la même configuration de serveur que le serveur principal, sauf si le paramètre **Sku** est spécifié.

> [!NOTE]
> Il est recommandé que la configuration du serveur réplica soit maintenue à des valeurs égales ou supérieures à celles du serveur principal pour garantir que le serveur réplica est à la hauteur du serveur principal.

### <a name="list-replicas-for-a-primary-server"></a>Répertorier les réplicas d'un serveur principal

Pour afficher tous les réplicas d'un serveur principal donné, exécutez la commande suivante :

```azurepowershell-interactive
Get-AzMariaDReplica -ResourceGroupName myresourcegroup -ServerName mydemoserver
```

La commande `Get-AzMariaDReplica` requiert les paramètres suivants :

| Paramètre | Valeur d'exemple | Description  |
| --- | --- | --- |
| ResourceGroupName |  myResourceGroup |  Groupe de ressources dans lequel le serveur réplica sera créé.  |
| ServerName | mydemoserver | Nom ou ID du serveur principal. |

### <a name="delete-a-replica-server"></a>Supprimer un serveur réplica

La suppression d’un serveur réplica en lecture peut être effectuée en exécutant la cmdlet `Remove-AzPostgreSqlServer`.

```azurepowershell-interactive
Remove-AzPostgreSqlServer -Name mydemoreplicaserver -ResourceGroupName myresourcegroup
```

### <a name="delete-a-primary-server"></a>Supprimer un serveur principal

> [!IMPORTANT]
> La suppression d’un serveur principal arrête la réplication vers tous les serveurs réplicas et supprime le serveur principal proprement dit. Les serveurs réplicas deviennent des serveurs autonomes qui prennent désormais en charge la lecture et les écritures.

Pour supprimer un serveur principal, vous pouvez exécuter la cmdlet `Remove-AzPostgreSqlServer`.

```azurepowershell-interactive
Remove-AzPostgreSqlServer -Name mydemoserver -ResourceGroupName myresourcegroup
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Redémarrer un serveur Azure Database pour PostgreSQL à l’aide de PowerShell](howto-restart-server-powershell.md)