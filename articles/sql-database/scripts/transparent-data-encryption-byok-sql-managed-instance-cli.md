---
title: Exemple CLI - Activer le chiffrement TDE avec BYOK - Azure SQL Managed Instance
description: Découvrez comment configurer une entité Azure SQL Managed Instance pour qu’elle commence à utiliser Transparent Data Encryption (TDE) avec BYOK pour le chiffrement au repos à l’aide de PowerShell.
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: ''
ms.devlang: azurecli
ms.topic: conceptual
author: MladjoA
ms.author: mlandzic
ms.reviewer: vanto
ms.date: 11/05/2019
ms.openlocfilehash: 2b948161633569d629dfb048a7d7dee6a9946f43
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91323684"
---
# <a name="manage-transparent-data-encryption-in-a-managed-instance-using-your-own-key-from-azure-key-vault"></a>Gérer Transparent Data Encryption dans une instance gérée à l’aide de votre propre clé Azure Key Vault

Cet exemple de script Azure CLI configure Transparent Data Encryption (TDE) avec une clé gérée par le client pour Azure SQL Managed Instance, en utilisant une clé provenant d’Azure Key Vault. Ceci est souvent désigné sous le nom de scénario BYOK (Bring Your Own Key) pour Transparent Data Encryption. Pour plus d’informations sur TDE avec une clé gérée par le client, consultez [TDE avec BYOK pour Azure SQL](../../azure-sql/database/transparent-data-encryption-byok-overview.md).

Si vous choisissez d’installer et d’utiliser l’interface de ligne de commande localement, vous devez exécuter Azure CLI version 2.0 ou une version ultérieure pour poursuivre la procédure décrite dans cet article. Exécutez `az --version` pour trouver la version. Si vous devez effectuer une installation ou une mise à niveau, consultez [Installer Azure CLI](/cli/azure/install-azure-cli).

## <a name="sample-script"></a>Exemple de script

### <a name="prerequisites"></a>Prérequis

Une instance gérée existante ; consultez [Utiliser Azure CLI pour créer une instance d'Azure SQL Managed Instance](sql-database-create-configure-managed-instance-cli.md).

### <a name="sign-in-to-azure"></a>Connexion à Azure

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

```azurecli-interactive
$subscription = "<subscriptionId>" # add subscription here

az account set -s $subscription # ...or use 'az login'
```

### <a name="run-the-script"></a>Exécuter le script

[!code-azurecli-interactive[main](../../../cli_scripts/sql-database/transparent-data-encryption/setup-tde-byok-sqlmi.sh "Set up BYOK TDE for SQL Managed Instance")]

### <a name="clean-up-deployment"></a>Nettoyer le déploiement

Utilisez la commande suivante pour supprimer le groupe de ressources et toutes les ressources associées.

```azurecli-interactive
az group delete --name $resource
```

## <a name="sample-reference"></a>Informations de référence sur l’exemple

Ce script utilise les commandes suivantes. Chaque commande du tableau renvoie à une documentation spécifique.

| Commande | Description |
|---|---|
| [az sql db](/cli/azure/sql/db) | Commandes de base de données. |
| [az sql failover-group](/cli/azure/sql/failover-group) | Commandes de groupe de basculement. |

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’interface Azure CLI, consultez la [documentation relative à l’interface Azure CLI](/cli/azure).

Vous trouverez des exemples supplémentaires de scripts CLI SQL Database dans [Documentation Azure SQL Database](../../azure-sql/database/az-cli-script-samples-content-guide.md).
