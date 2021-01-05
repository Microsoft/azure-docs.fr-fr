---
title: Script Azure CLI – Modifier les configurations de serveur (PostgreSQL)
description: Cet exemple de script CLI permet de lister toutes les options de configuration de serveur disponibles et met à jour la valeur de l’une d’entre elles.
author: lfittl-msft
ms.author: lufittl
ms.service: postgresql
ms.devlang: azurecli
ms.topic: sample
ms.custom: mvc, devx-track-azurecli
ms.date: 02/28/2018
ms.openlocfilehash: 313c3a59c1b47b4cc6ee6a38e46e85a2343bfb31
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94660523"
---
# <a name="list-and-update-configurations-of-an-azure-database-for-postgresql-server-using-azure-cli"></a>Lister et mettre à jour les configurations d’un serveur Azure Database pour PostgreSQL avec Azure CLI
Cet exemple de script CLI permet de lister tous les paramètres de configuration disponibles, ainsi que leurs valeurs autorisées, pour un serveur Azure Database pour PostgreSQL, et définit *log_retention_days* sur une valeur autre que la valeur par défaut.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

- Cet article nécessite la version 2.0 ou ultérieure d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="sample-script"></a>Exemple de script
Dans cet exemple de script, modifiez les lignes en surbrillance pour mettre à jour le nom d’utilisateur et le mot de passe d’administrateur et utiliser les vôtres.
[!code-azurecli-interactive[main](../../../cli_scripts/postgresql/change-server-configurations/change-server-configurations.sh?highlight=15-16 "List and update configurations of Azure Database for PostgreSQL.")]

## <a name="clean-up-deployment"></a>Nettoyer le déploiement
Utilisez la commande suivante pour supprimer le groupe de ressources et toutes les ressources associées après l’exécution du script. 
[!code-azurecli-interactive[main](../../../cli_scripts/postgresql/change-server-configurations/delete-postgresql.sh  "Delete the resource group.")]

## <a name="script-explanation"></a>Explication du script
Ce script utilise les commandes décrites dans le tableau suivant :

| **Commande** | **Remarques** |
|---|---|
| [az group create](/cli/azure/group) | Crée un groupe de ressources dans lequel toutes les ressources sont stockées. |
| [az postgres server create](/cli/azure/postgres/server) | Crée un serveur PostgreSQL qui héberge les bases de données. |
| [az postgres server configuration list](/cli/azure/postgres/server/configuration) | Liste les configurations d’un serveur Azure Database pour PostgreSQL. |
| [az postgres server configuration set](/cli/azure/postgres/server/configuration) | Met à jour la configuration d’un serveur Azure Database pour PostgreSQL. |
| [az postgres server configuration show](/cli/azure/postgres/server/configuration) | Affiche la configuration d’un serveur Azure Database pour PostgreSQL. |
| [az group delete](/cli/azure/group) | Supprime un groupe de ressources, y compris toutes les ressources imbriquées. |

## <a name="next-steps"></a>Étapes suivantes
- En savoir plus sur Azure CLI : [Documentation d’Azure CLI](/cli/azure)
- Essayez d’autres scripts : [Exemples Azure CLI pour base de données pour PostgreSQL](../sample-scripts-azure-cli.md)
- Pour plus d’informations sur les paramètres de serveur, consultez la page [Guide pratique pour configurer les paramètres de serveur sur le Portail Azure](../howto-configure-server-parameters-using-portal.md).
