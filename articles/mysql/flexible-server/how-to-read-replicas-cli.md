---
title: Gérer les réplicas en lecture dans Azure Database pour MySQL – Serveur flexible à l’aide d’Azure CLI.
description: Découvrez comment configurer et gérer des réplicas en lecture dans Azure Database pour MySQL – Serveur flexible à l’aide d’Azure CLI.
author: ambhatna
ms.author: ambhatna
ms.service: mysql
ms.topic: how-to
ms.date: 10/23/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: a7d63fd76a88430495c9f55200308f63b11c89d4
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96494305"
---
# <a name="how-to-create-and-manage-read-replicas-in-azure-database-for-mysql-flexible-server-using-the-azure-cli"></a>Comment créer et gérer des réplicas en lecture dans Azure Database pour MySQL – Serveur flexible à l’aide d’Azure CLI

> [!IMPORTANT]
> La fonctionnalité Réplicas en lecture dans Azure Database pour MySQL – Serveur flexible est disponible en préversion.

Dans cet article, vous allez apprendre à créer et à gérer des réplicas en lecture dans le serveur flexible Azure Database pour MySQL à l’aide d’Azure CLI. Pour en savoir plus sur les réplicas en lecture, consultez [vue d’ensemble](concepts-read-replicas.md).

> [!Note]
> Le réplica n’est pas pris en charge sur les serveurs à une haute disponibilité. 

## <a name="azure-cli"></a>Azure CLI
Vous pouvez créer et gérer des réplicas en lecture à l’aide d’Azure CLI.

### <a name="prerequisites"></a>Prérequis

- [Installation d’Azure CLI 2.0](/cli/azure/install-azure-cli)
- Un [serveur flexible Azure Database pour MySQL](quickstart-create-server-cli.md) qui sera utilisé comme serveur source.

### <a name="create-a-read-replica"></a>Créer un réplica en lecture

> [!IMPORTANT]
> Lorsque vous créez un réplica pour un serveur source qui n’en a pas, ce dernier commence par redémarrer afin de se préparer à la réplication. Tenez-en compte et effectuez ces opérations en période creuse.

Un serveur réplica en lecture peut être créé en utilisant la commande suivante :

```azurecli-interactive
az mysql flexible-server replica create --replica-name mydemoreplicaserver --source-server mydemoserver --resource-group myresourcegroup
``` 

> [!NOTE]
> Les réplicas en lecture sont créés avec la même configuration de serveur que le serveur source. La configuration du serveur réplica peut être modifiée après la création de ce dernier. Le serveur réplica est toujours créé dans le même groupe de ressources, le même emplacement et le même abonnement que le serveur source. Si vous souhaitez créer un serveur réplica dans un autre groupe de ressources ou un autre abonnement, vous pouvez [déplacer le serveur réplica](../../azure-resource-manager/management/move-resource-group-and-subscription.md) après sa création. Il est recommandé de maintenir la configuration du serveur réplica à des valeurs égales ou supérieures à celles du serveur source pour garantir que le réplica sera à la hauteur du serveur source.


### <a name="list-replicas-for-a-source-server"></a>Répertorier les réplicas d'un serveur source

Pour afficher tous les réplicas d'un serveur source donné, exécutez la commande suivante : 

```azurecli-interactive
az mysql flexible-server replica list --server-name mydemoserver --resource-group myresourcegroup
```

### <a name="stop-replication-to-a-replica-server"></a>Arrêter la réplication vers un serveur réplica

> [!IMPORTANT]
> L’arrêt de la réplication vers un serveur est irréversible. Une fois la réplication entre un serveur source et un serveur réplica arrêtée, il est impossible de revenir en arrière. Le serveur réplica devient un serveur autonome et prend désormais en charge la lecture et les écritures. Ce serveur ne peut pas être à nouveau transformé en réplica.

La réplication d'un serveur réplica en lecture peut être arrêtée en utilisant la commande suivante :

```azurecli-interactive
az mysql flexible-server replica stop-replication --replica-name mydemoreplicaserver --resource-group myresourcegroup
```

### <a name="delete-a-replica-server"></a>Supprimer un serveur réplica

La suppression d’un serveur réplica en lecture peut être effectuée en exécutant la commande **[az mysql server delete](/cli/azure/mysql/server)** .

```azurecli-interactive
az mysql flexible-server delete --resource-group myresourcegroup --name mydemoreplicaserver
```

### <a name="delete-a-source-server"></a>Supprimer un serveur source

> [!IMPORTANT]
> La suppression d’un serveur source arrête la réplication vers tous les serveurs réplicas et supprime le serveur source proprement dit. Les serveurs réplicas deviennent des serveurs autonomes qui prennent désormais en charge la lecture et les écritures.

Pour supprimer un serveur source, vous pouvez exécuter la commande **[az mysql flexible-server delete](/cli/azure/mysql/flexible-server)** .

```azurecli-interactive
az mysql flexible-server delete --resource-group myresourcegroup --name mydemoserver
```

## <a name="next-steps"></a>Étapes suivantes

- Découvrir plus en détail les [réplicas en lecture](concepts-read-replicas.md)