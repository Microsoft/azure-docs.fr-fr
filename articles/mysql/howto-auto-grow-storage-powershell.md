---
title: Augmenter automatiquement le stockage – Azure PowerShell – Azure Database pour MySQL
description: Cet article explique comment vous pouvez activer la croissance automatique du stockage avec PowerShell dans Azure Database pour MySQL.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: how-to
ms.date: 4/28/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: e8c9f6f66e484fbd9ebe5c15934936d6e5c59073
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94542029"
---
# <a name="auto-grow-storage-in-azure-database-for-mysql-server-using-powershell"></a>Croissance automatique du stockage sur le serveur Azure Database pour MySQL à l’aide de PowerShell

Cet article explique comment vous pouvez configurer l’augmentation d’un stockage de serveur Azure Database pour MySQL sans affecter la charge de travail.

La croissance automatique du stockage évite à votre serveur d’[atteindre la limite de stockage](./concepts-pricing-tiers.md#reaching-the-storage-limit) et de passer en lecture seule. Pour les serveurs disposant de jusqu’à 100 Go de stockage approvisionné, la taille est augmentée de 5 Go lorsque l’espace libre est inférieur à 10 %. Pour les serveurs disposant de plus de 100 Go de stockage approvisionné, la taille est augmentée de 5 % lorsque l’espace libre est inférieur à 10 Go. Des limites de stockage maximales s’appliquent comme spécifié dans la section relative au stockage des [niveaux tarifaires d’Azure Database pour MySQL](./concepts-pricing-tiers.md#storage).

> [!IMPORTANT]
> N’oubliez pas que le stockage peut seulement monter en puissance.

## <a name="prerequisites"></a>Prérequis

Pour utiliser ce guide pratique, il vous faut :

- Le [module Az PowerShell](/powershell/azure/install-az-ps) installé localement ou [Azure Cloud Shell](https://shell.azure.com/) dans le navigateur
- Un [serveur Azure Database pour MySQL](quickstart-create-mysql-server-database-using-azure-powershell.md)

> [!IMPORTANT]
> Tant que le module Az.MySql PowerShell est en préversion, vous devez l’installer séparément du module Az PowerShell à l’aide de la commande suivante : `Install-Module -Name Az.MySql -AllowPrerelease`.
> Une fois que le module Az.MySql PowerShell est en disponibilité générale, il devient partie intégrante des versions futures du module Az PowerShell et disponible en mode natif dans Azure Cloud Shell.

Si vous choisissez d’utiliser PowerShell en local, connectez-vous à votre compte Azure à l’aide de la cmdlet [Connect-AzAccount](/powershell/module/az.accounts/Connect-AzAccount).

## <a name="enable-mysql-server-storage-auto-grow"></a>Activer la croissance automatique du stockage du serveur MySQL

Activez le stockage à croissance automatique du serveur sur un serveur existant avec la commande suivante :

```azurepowershell-interactive
Update-AzMySqlServer -Name mydemoserver -ResourceGroupName myresourcegroup -StorageAutogrow Enabled
```

Activez le stockage à croissance automatique du serveur sur un nouveau serveur avec la commande suivante :

```azurepowershell-interactive
$Password = Read-Host -Prompt 'Please enter your password' -AsSecureString
New-AzMySqlServer -Name mydemoserver -ResourceGroupName myresourcegroup -Sku GP_Gen5_2 -StorageAutogrow Enabled -Location westus -AdministratorUsername myadmin -AdministratorLoginPassword $Password
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Créer et gérer des réplicas en lecture dans Azure Database pour MySQL à l’aide de PowerShell](howto-read-replicas-powershell.md)