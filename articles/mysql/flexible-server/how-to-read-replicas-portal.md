---
title: Gérer des réplicas en lecture – Portail Azure – Azure Database pour MySQL – Serveur flexible
description: Découvrez comment configurer et gérer des réplicas en lecture dans Azure Database pour MySQL – Serveur flexible à l’aide du portail Azure.
author: ambhatna
ms.author: ambhatna
ms.service: mysql
ms.topic: how-to
ms.date: 10/26/2020
ms.openlocfilehash: d33734dc7404e49aed94dffae8644b2bc4386925
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96492826"
---
# <a name="how-to-create-and-manage-read-replicas-in-azure-database-for-mysql-flexible-server-using-the-azure-portal"></a>Comment créer et gérer des réplicas en lecture dans Azure Database pour MySQL – Serveur flexible à l’aide du portail Azure

> [!IMPORTANT]
> La fonctionnalité Réplicas en lecture dans Azure Database pour MySQL – Serveur flexible est disponible en préversion.

Dans cet article, vous allez apprendre à créer et à gérer des réplicas en lecture dans le serveur flexible Azure Database pour MySQL à l’aide du portail Azure.

> [!Note]
> Le réplica n’est pas pris en charge sur les serveurs à une haute disponibilité. 

## <a name="prerequisites"></a>Prérequis

- Un [serveur flexible Azure Database pour MySQL](quickstart-create-server-portal.md) qui sera utilisé comme serveur source.

## <a name="create-a-read-replica"></a>Créer un réplica en lecture

> [!IMPORTANT]
> Lorsque vous créez un réplica pour un serveur source qui n’en a pas, ce dernier commence par redémarrer afin de se préparer à la réplication. Tenez-en compte et effectuez ces opérations en période creuse.

Un serveur réplica en lecture peut être créé en effectuant les étapes suivantes :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

2. Sélectionnez le serveur flexible Azure Database pour MySQL que vous souhaitez utiliser comme source. Cette action ouvre la page **Vue d’ensemble**.

3. Sélectionnez **Réplication** dans le menu, sous **PARAMÈTRES**.

4. Sélectionnez **Ajouter un réplica**.

   :::image type="content" source="./media/how-to-read-replica-portal/add-replica.png" alt-text="Azure Database pour MySQL - Réplication":::

5. Entrez un nom pour le serveur de réplica.

    :::image type="content" source="./media/how-to-read-replica-portal/replica-name.png" alt-text="Azure Database pour MySQL - Nom du réplica":::

6. Sélectionnez **OK** pour confirmer la création du réplica.

> [!NOTE]
> Les réplicas en lecture sont créés avec la même configuration de serveur que le serveur source. La configuration du serveur réplica peut être modifiée après la création de ce dernier. Le serveur réplica est toujours créé dans le même groupe de ressources, le même emplacement et le même abonnement que le serveur source. Si vous souhaitez créer un serveur réplica dans un autre groupe de ressources ou un autre abonnement, vous pouvez [déplacer le serveur réplica](../../azure-resource-manager/management/move-resource-group-and-subscription.md) après sa création. Il est recommandé de maintenir la configuration du serveur réplica à des valeurs égales ou supérieures à celles du serveur source pour garantir que le réplica sera à la hauteur du serveur source.

Une fois le serveur réplica créé, il est affiché dans le panneau **Réplication**.

   [:::image type="content" source="./media/how-to-read-replica-portal/list-replica.png" alt-text="Azure Database pour MySQL - Répertorier les réplicas":::](./media/how-to-read-replica-portal/list-replica.png#lightbox)

## <a name="stop-replication-to-a-replica-server"></a>Arrêter la réplication vers un serveur réplica

> [!IMPORTANT]
> L’arrêt de la réplication vers un serveur est irréversible. Une fois la réplication entre un serveur source et un serveur réplica arrêtée, il est impossible de revenir en arrière. Le serveur réplica devient un serveur autonome et prend désormais en charge la lecture et les écritures. Ce serveur ne peut pas être à nouveau transformé en réplica.

Pour arrêter la réplication entre un serveur source et un serveur réplica à partir du portail Azure, effectuez les étapes suivantes :

1. Dans le portail Azure, sélectionnez votre serveur flexible Azure Database pour MySQL source. 

2. Sélectionnez **Réplication** dans le menu, sous **PARAMÈTRES**.

3. Sélectionnez le serveur réplica pour lequel vous souhaitez arrêter la réplication.

   [:::image type="content" source="./media/how-to-read-replica-portal/stop-replication-select.png" alt-text="Azure Database pour MySQL - Sélection du serveur pour lequel arrêter la réplication":::](./media/how-to-read-replica-portal/stop-replication-select.png#lightbox)

4. Sélectionnez **Arrêter la réplication**.

   [:::image type="content" source="./media/how-to-read-replica-portal/stop-replication.png" alt-text="Azure Database pour MySQL - Arrêter la réplication":::](./media/how-to-read-replica-portal/stop-replication.png#lightbox)

5. Validez que vous voulez arrêter la réplication en cliquant sur **OK**.

   [:::image type="content" source="./media/how-to-read-replica-portal/stop-replication-confirm.png" alt-text="Azure Database pour MySQL - Confirmation de l’arrêt de la réplication":::](./media/how-to-read-replica-portal/stop-replication-confirm.png#lightbox)

## <a name="delete-a-replica-server"></a>Supprimer un serveur réplica

Pour supprimer un serveur réplica en lecture du portail Azure, utilisez les étapes suivantes :

1. Dans le portail Azure, sélectionnez votre serveur flexible Azure Database pour MySQL source.

2. Sélectionnez **Réplication** dans le menu, sous **PARAMÈTRES**.

3. Sélectionnez le serveur réplica que vous voulez supprimer.

   [:::image type="content" source="./media/how-to-read-replica-portal/delete-replica-select.png" alt-text="Azure Database pour MySQL - Sélection du serveur réplica à supprimer":::](./media/how-to-read-replica-portal/delete-replica-select.png#lightbox)

4. Sélectionnez **Supprimer le réplica**.

   :::image type="content" source="./media/how-to-read-replica-portal/delete-replica.png" alt-text="Azure Database pour MySQL : Supprimer le réplica":::

5. Tapez le nom du réplica, puis cliquez sur **Supprimer** pour confirmer la suppression de ce dernier.  

   :::image type="content" source="./media/how-to-read-replica-portal/delete-replica-confirm.png" alt-text="Azure Database pour MySQL - Confirmation de la suppression du réplica":::

## <a name="delete-a-source-server"></a>Supprimer un serveur source

> [!IMPORTANT]
> La suppression d’un serveur source arrête la réplication vers tous les serveurs réplicas et supprime le serveur source proprement dit. Les serveurs réplicas deviennent des serveurs autonomes qui prennent désormais en charge la lecture et les écritures.

Pour supprimer un serveur source du portail Azure, procédez comme suit :

1. Dans le portail Azure, sélectionnez votre serveur flexible Azure Database pour MySQL source.

2. Dans la **Vue d’ensemble**, sélectionnez **Supprimer**.

   [:::image type="content" source="./media/how-to-read-replica-portal/delete-master-overview.png" alt-text="Azure Database pour MySQL : Supprimer la source":::](./media/how-to-read-replica-portal/delete-master-overview.png#lightbox)

3. Saisissez le nom du serveur source, puis cliquez sur **Supprimer** pour confirmer la suppression de ce dernier.  

   :::image type="content" source="./media/how-to-read-replica-portal/delete-master-confirm.png" alt-text="Azure Database pour MySQL – Confirmer la suppression de la source":::

## <a name="monitor-replication"></a>Superviser la réplication

1. Dans le [portail Azure](https://portal.azure.com/), sélectionnez le réplica de serveur flexible Azure Database pour MySQL à analyser.

2. Sous la section **Surveillance** de la barre latérale, sélectionnez **Métriques** :

3. Sélectionnez **Décalage de la réplication en secondes** dans la liste déroulante des métriques disponibles.

   [:::image type="content" source="./media/how-to-read-replica-portal/monitor-select-replication-lag.png" alt-text="Sélectionner le décalage de la réplication":::](./media/how-to-read-replica-portal/monitor-select-replication-lag.png#lightbox)

4. Sélectionnez l’intervalle de temps que vous voulez afficher. L’image ci-dessous sélectionne un intervalle de temps de 30 minutes.

   [:::image type="content" source="./media/how-to-read-replica-portal/monitor-replication-lag-time-range.png" alt-text="Sélectionner un intervalle de temps":::](./media/how-to-read-replica-portal/monitor-replication-lag-time-range.png#lightbox)

5. Affichez le décalage de la réplication pour l’intervalle de temps sélectionné. L’image ci-dessous affiche les 30 dernières minutes.

   [:::image type="content" source="./media/how-to-read-replica-portal/monitor-replication-lag-time-range-thirty-mins.png" alt-text="Sélectionner un intervalle de temps de 30 minutes":::](./media/how-to-read-replica-portal/monitor-replication-lag-time-range-thirty-mins.png#lightbox)

## <a name="next-steps"></a>Étapes suivantes

- Découvrir plus en détail les [réplicas en lecture](concepts-read-replicas.md)