---
title: 'Script Azure CLI : Télécharger des journaux d’activité de serveurs dans Azure Database pour PostgreSQL'
description: Cet exemple de script Azure CLI montre comment activer et télécharger les journaux d’activité d’un serveur Azure Database pour PostgreSQL.
author: lfittl-msft
ms.author: lufittl
ms.service: postgresql
ms.devlang: azurecli
ms.topic: sample
ms.custom: mvc, devx-track-azurecli
ms.date: 02/28/2018
ms.openlocfilehash: d6ba14a8838d71397a8c2348a03b1d760b3cd739
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94660438"
---
# <a name="enable-and-download-server-slow-query-logs-of-an-azure-database-for-postgresql-server-using-azure-cli"></a>Activer et télécharger les journaux d’activité de requêtes lentes d’un serveur Azure Database pour PostgreSQL à l’aide d’Azure CLI
Cet exemple de script CLI montre comment activer et télécharger les journaux d’activité de requêtes lentes d’un seul serveur Azure Database pour PostgreSQL.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

- Cet article nécessite la version 2.0 ou ultérieure d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="sample-script"></a>Exemple de script
Dans cet exemple de script, modifiez les lignes en surbrillance pour mettre à jour le nom d’utilisateur et le mot de passe d’administrateur et utiliser les vôtres. Dans les commandes `az monitor`, remplacez &lt;log_file_name&gt; par le nom de votre fichier journal de serveur.
[!code-azurecli-interactive[main](../../../cli_scripts/postgresql/server-logs/server-logs.sh?highlight=15-16 "Manipulate with server logs.")]

## <a name="clean-up-deployment"></a>Nettoyer le déploiement
Utilisez la commande suivante pour supprimer le groupe de ressources et toutes les ressources associées après l’exécution du script. 
[!code-azurecli-interactive[main](../../../cli_scripts/postgresql/server-logs/delete-postgresql.sh  "Delete the resource group.")]

## <a name="script-explanation"></a>Explication du script
Ce script utilise les commandes décrites dans le tableau suivant :

| **Commande** | **Remarques** |
|---|---|
| [az group create](/cli/azure/group) | Crée un groupe de ressources dans lequel toutes les ressources sont stockées. |
| [az postgres server create](/cli/azure/postgres/server) | Crée un serveur PostgreSQL qui héberge les bases de données. |
| [az postgres server configuration list](/cli/azure/postgres/server/configuration) | Répertoriez les valeurs de configuration pour un serveur. |
| [az postgres server configuration set](/cli/azure/postgres/server/configuration) | Mettez à jour la configuration d’un serveur. |
| [az postgres server-logs list](/cli/azure/postgres/server-logs) | Répertoriez les fichiers journaux pour un serveur. |
| [az postgres server-logs download](/cli/azure/postgres/server-logs) | Téléchargez les fichiers journaux. |
| [az group delete](/cli/azure/group) | Supprime un groupe de ressources, y compris toutes les ressources imbriquées. |

## <a name="next-steps"></a>Étapes suivantes
- En savoir plus sur Azure CLI : [Documentation d’Azure CLI](/cli/azure)
- Essayez d’autres scripts : [Exemples Azure CLI pour base de données pour PostgreSQL](../sample-scripts-azure-cli.md)
- [Configurer et consulter les journaux d’activité du serveur sur le Portail Azure](../howto-configure-server-logs-in-portal.md)
