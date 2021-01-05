---
title: Augmenter automatiquement le stockage – Azure PowerShell – Azure Database pour PostgreSQL
description: Cet article explique comment activer la croissance automatique du stockage avec PowerShell dans Azure Database pour PostgreSQL.
author: ambhatna
ms.author: ambhatna
ms.service: postgresql
ms.topic: how-to
ms.date: 06/08/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: bc4655ce6cd572183cd92e1c8b2ac10e613ebd8f
ms.sourcegitcommit: 3bcce2e26935f523226ea269f034e0d75aa6693a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92489963"
---
# <a name="auto-grow-storage-in-azure-database-for-postgresql-server-using-powershell"></a>Croissance automatique du stockage sur le serveur Azure Database pour PostgreSQL à l’aide de PowerShell

Cet article explique comment vous pouvez configurer l’augmentation d’un stockage de serveur Azure Database pour PostgreSQL sans affecter la charge de travail.

La croissance automatique du stockage évite à votre serveur d’[atteindre la limite de stockage](./concepts-pricing-tiers.md#reaching-the-storage-limit) et de passer en lecture seule. Pour les serveurs disposant de jusqu’à 100 Go de stockage approvisionné, la taille est augmentée de 5 Go lorsque l’espace libre est inférieur à 10 %. Pour les serveurs disposant de plus de 100 Go de stockage approvisionné, la taille est augmentée de 5 % lorsque l’espace libre est inférieur à 10 Go. Des limites de stockage maximales s’appliquent comme spécifié dans la section relative au stockage des [niveaux tarifaires d’Azure Database pour PostgreSQL](./concepts-pricing-tiers.md#storage).

> [!IMPORTANT]
> N’oubliez pas que le stockage peut seulement monter en puissance.

## <a name="prerequisites"></a>Prérequis

Pour utiliser ce guide pratique, il vous faut :

- Le [module Az PowerShell](/powershell/azure/install-az-ps) installé localement ou [Azure Cloud Shell](https://shell.azure.com/) dans le navigateur
- Un [serveur Azure Database pour PostgreSQL](quickstart-create-postgresql-server-database-using-azure-powershell.md)

> [!IMPORTANT]
> Tant que le module Az.PostgreSql PowerShell est en préversion, vous devez l’installer séparément du module Az PowerShell à l’aide de la commande suivante : `Install-Module -Name Az.PostgreSql -AllowPrerelease`
> Une fois que le module Az.PostgreSql PowerShell est en disponibilité générale, il devient partie intégrante des versions ultérieures du module Az PowerShell et est disponible en mode natif dans Azure Cloud Shell.

Si vous choisissez d’utiliser PowerShell en local, connectez-vous à votre compte Azure à l’aide de la cmdlet [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount).

## <a name="enable-postgresql-server-storage-auto-grow"></a>Activer la croissance automatique du stockage du serveur PostgreSQL

Activez le stockage à croissance automatique du serveur sur un serveur existant avec la commande suivante :

```azurepowershell-interactive
Update-AzPostgreSqlServer -Name mydemoserver -ResourceGroupName myresourcegroup -StorageAutogrow Enabled
```

Activez le stockage à croissance automatique du serveur sur un nouveau serveur avec la commande suivante :

```azurepowershell-interactive
$Password = Read-Host -Prompt 'Please enter your password' -AsSecureString
New-AzPostgreSqlServer -Name mydemoserver -ResourceGroupName myresourcegroup -Sku GP_Gen5_2 -StorageAutogrow Enabled -Location westus -AdministratorUsername myadmin -AdministratorLoginPassword $Password
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Guide pratique pour créer et gérer des réplicas en lecture dans Azure Database pour PostgreSQL à l’aide de PowerShell](howto-read-replicas-powershell.md)