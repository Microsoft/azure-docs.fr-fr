---
title: Gérer les réplicas en lecture - Portail Azure - Azure Database pour PostgreSQL - Serveur unique
description: Découvrez comment gérer les réplicas en lecture pour Azure Database pour PostgreSQL (serveur unique) à partir du portail Azure.
author: sr-msft
ms.author: srranga
ms.service: postgresql
ms.topic: how-to
ms.date: 11/05/2020
ms.openlocfilehash: 9fdef187e9bdf77b29c548f767a4b4edfeb62f44
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93422176"
---
# <a name="create-and-manage-read-replicas-in-azure-database-for-postgresql---single-server-from-the-azure-portal"></a>Créer et gérer les réplicas en lecture dans Azure Database pour PostgreSQL (serveur unique) à partir du portail Azure

Dans cet article, vous allez apprendre à créer et gérer des réplicas en lecture dans Azure Database pour PostgreSQL à partir du portail Azure. Pour en savoir plus sur les réplicas en lecture, consultez [vue d’ensemble](concepts-read-replicas.md).


## <a name="prerequisites"></a>Prérequis
Un [serveur Azure Database pour PostgreSQL](quickstart-create-server-database-portal.md) qui représente le serveur principal.

## <a name="azure-replication-support"></a>Prise en charge de la réplication Azure

Les [réplicas en lecture](concepts-read-replicas.md) et le [décodage logique](concepts-logical.md) dépendent du journal WAL de Postgres pour obtenir des informations. Ces deux fonctionnalités ont besoin de différents niveaux de journalisation à partir de Postgres. Le décodage logique nécessite un niveau de journalisation plus élevé que les réplicas en lecture.

Pour configurer le niveau de journalisation approprié, utilisez le paramètre de prise en charge de la réplication Azure. La prise en charge de la réplication Azure propose trois options de paramétrage :

* **Désactivé** : place le moins d’informations dans le journal WAL. Ce paramètre n’est pas disponible sur la plupart des serveurs Azure Database pour PostgreSQL.  
* **Réplica** : plus de détails que l’option **Désactivé**. Il s’agit du niveau minimal de journalisation nécessaire pour que les [réplicas en lecture](concepts-read-replicas.md) fonctionnent. Il s’agit du paramètre par défaut sur la plupart des serveurs.
* **Logique** : plus de détails que l’option **Réplica**. Il s’agit du niveau minimal de journalisation pour que le décodage logique fonctionne. Les réplicas en lecture fonctionnent également avec ce paramètre.


> [!NOTE]
> Lors du déploiement de réplicas de lecture pour des charges de travail principales gourmandes en écriture intenses persistantes, le retard de réplication peut continuer à s’allonger et ne jamais pouvoir se réaligner sur le principal. Cela peut également augmenter l’utilisation du stockage sur le serveur principal, car les fichiers WAL ne sont pas supprimés tant qu’ils ne sont pas reçus au niveau du réplica.

## <a name="prepare-the-primary-server"></a>Préparer le serveur principal

1. Dans le portail Azure, sélectionnez un serveur Azure Database pour PostgreSQL à utiliser en tant que serveur maître.

2. Sélectionnez **Réplication** depuis le menu du serveur. Si un support de réplication Azure est réglé au moins sur **Réplica**, vous pouvez créer des réplicas en lecture. 

3. Si le support de réplication Azure n’est pas réglé sur au moins **Réplica**, réglez-le. Sélectionnez **Enregistrer**.

   :::image type="content" source="./media/howto-read-replicas-portal/set-replica-save.png" alt-text="Azure Database pour PostgreSQL – Réplication – Définir un réplica et enregistrer":::

4. Redémarrez le serveur pour appliquer le changement en sélectionnant **Oui**.

   :::image type="content" source="./media/howto-read-replicas-portal/confirm-restart.png" alt-text="Azure Database pour PostgreSQL - Réplication - Confirmer le redémarrage":::

5. Vous recevrez deux notifications du portail Azure une fois que l’opération sera terminée. Une notification concerne la mise à jour du paramètre du serveur. La seconde concerne le redémarrage du serveur qui suit immédiatement.

   :::image type="content" source="./media/howto-read-replicas-portal/success-notifications.png" alt-text="Notifications de réussite":::

6. Actualisez la page du portail Azure pour mettre à jour la barre d’outils Réplication. Vous pouvez désormais créer des réplicas en lecture pour ce serveur.
   

## <a name="create-a-read-replica"></a>Créer un réplica en lecture
Pour créer un réplica en lecture, effectuez les étapes suivantes :

1. Sélectionnez un serveur Azure Database pour PostgreSQL à utiliser en tant que serveur principal. 

2. Dans la barre latérale du serveur, sous **PARAMÈTRES**, sélectionnez **Réplication**.

3. Sélectionnez **Ajouter un réplica**.

   :::image type="content" source="./media/howto-read-replicas-portal/add-replica.png" alt-text="Ajouter un réplica":::

4. Entrez un nom pour le réplica en lecture. 

    :::image type="content" source="./media/howto-read-replicas-portal/name-replica.png" alt-text="Nommer le réplica":::

5. Sélectionnez un emplacement pour le réplica. La localisation par défaut est la même que celle du serveur principal.

    :::image type="content" source="./media/howto-read-replicas-portal/location-replica.png" alt-text="Sélectionner un emplacement":::

   > [!NOTE]
   > Pour en savoir plus sur les régions dans lesquelles vous pouvez créer un réplica, consultez l’article [Concepts relatifs aux réplicas en lecture](concepts-read-replicas.md). 

6. Sélectionnez **OK** pour confirmer la création du réplica.

Une fois le réplica en lecture créé, vous pouvez le voir dans la fenêtre **Réplication** :

:::image type="content" source="./media/howto-read-replicas-portal/list-replica.png" alt-text="Voir le nouveau réplica dans la fenêtre Réplication":::
 

> [!IMPORTANT]
> Consultez la [section de considérations relatives à la vue d’ensemble de réplica en lecture](concepts-read-replicas.md#considerations).
>
> Avant de modifier un paramètre du serveur principal, remplacez la valeur du paramètre du réplica par une valeur supérieure ou égale. Vous garantissez ainsi l’alignement du réplica sur les changements apportés au serveur maître.

## <a name="stop-replication"></a>Arrêter la réplication
Vous pouvez arrêter la réplication entre un serveur principal et un réplica en lecture.

> [!IMPORTANT]
> Une fois que vous avez arrêté la réplication entre un serveur principal et un réplica en lecture, vous ne pouvez plus l’annuler. Le réplica en lecture devient un serveur autonome qui prend en charge les lectures et les écritures. Le serveur autonome ne peut pas être retransformé en réplica.

Pour arrêter la réplication entre un serveur principal et un réplica en lecture à partir du portail Azure, effectuez les étapes suivantes :

1. Dans le portail Azure, sélectionnez votre serveur Azure Database pour PostgreSQL principal.

2. Dans le menu du serveur, sous **PARAMÈTRES**, sélectionnez **Réplication**.

3. Sélectionnez le serveur réplica dont vous souhaitez arrêter la réplication.

   :::image type="content" source="./media/howto-read-replicas-portal/select-replica.png" alt-text="Sélectionner le réplica":::
 
4. Sélectionnez **Arrêter la réplication**.

   :::image type="content" source="./media/howto-read-replicas-portal/select-stop-replication.png" alt-text="Sélectionner Arrêter la réplication":::
 
5. Sélectionnez **OK** pour arrêter la réplication.

   :::image type="content" source="./media/howto-read-replicas-portal/confirm-stop-replication.png" alt-text="Confirmer l’arrêt de la réplication":::
 

## <a name="delete-a-primary-server"></a>Supprimer un serveur principal
Pour supprimer un serveur principal, suivez les mêmes étapes que celles qui permettent de supprimer un serveur Azure Database pour PostgreSQL autonome. 

> [!IMPORTANT]
> Quand vous supprimez un serveur principal, la réplication est arrêtée sur tous les réplicas en lecture. Les réplicas en lecture deviennent des serveurs autonomes qui prennent désormais en charge les lectures et les écritures.

Pour supprimer un serveur du portail Azure, effectuez les étapes suivantes :

1. Dans le portail Azure, sélectionnez votre serveur Azure Database pour PostgreSQL principal.

2. Ouvrez la page **Vue d’ensemble** du serveur. Sélectionnez **Supprimer**.

   :::image type="content" source="./media/howto-read-replicas-portal/delete-server.png" alt-text="Dans la page de vue d’ensemble du serveur, sélectionner Supprimer pour supprimer le serveur principal":::
 
3. Entrez le nom du serveur principal à supprimer. Sélectionnez **Supprimer** pour confirmer la suppression du serveur principal.

   :::image type="content" source="./media/howto-read-replicas-portal/confirm-delete.png" alt-text="Confirmer la suppression du serveur principal":::
 

## <a name="delete-a-replica"></a>Supprimer un réplica
Vous pouvez supprimer un réplica en lecture de la même façon que vous supprimez un serveur principal.

- Dans le portail Azure, ouvrez la page **Vue d’ensemble** du réplica en lecture. Sélectionnez **Supprimer**.

   :::image type="content" source="./media/howto-read-replicas-portal/delete-replica.png" alt-text="Dans la page de vue d’ensemble du réplica, sélectionner Supprimer pour supprimer le réplica":::
 
Vous pouvez également supprimer le réplica en lecture de la fenêtre **Réplication** en effectuant les étapes suivantes :

1. Dans le portail Azure, sélectionnez votre serveur Azure Database pour PostgreSQL principal.

2. Dans le menu du serveur, sous **PARAMÈTRES**, sélectionnez **Réplication**.

3. Sélectionnez le réplica en lecture à supprimer.

   :::image type="content" source="./media/howto-read-replicas-portal/select-replica.png" alt-text="Sélectionner le réplica à supprimer":::
 
4. Sélectionnez **Supprimer le réplica**.

   :::image type="content" source="./media/howto-read-replicas-portal/select-delete-replica.png" alt-text="Sélectionnez Supprimer le réplica":::
 
5. Entrez le nom du réplica à supprimer. Sélectionnez **Supprimer** pour confirmer la suppression du réplica.

   :::image type="content" source="./media/howto-read-replicas-portal/confirm-delete-replica.png" alt-text="Confirmer la suppression du réplica":::
 

## <a name="monitor-a-replica"></a>Superviser un réplica
Deux métriques sont disponibles pour superviser les réplicas en lecture.

### <a name="max-lag-across-replicas-metric"></a>Métrique Retard maximum entre réplicas
La métrique **Retard maximum entre réplicas** indique le retard en octets entre le serveur principal et le réplica le plus en retard. 

1.  Dans le portail Azure, sélectionnez le serveur Azure Database pour PostgreSQL principal.

2.  Sélectionnez **Métriques**. Dans la fenêtre **Métriques**, sélectionnez **Retard maximum entre réplicas**.

    :::image type="content" source="./media/howto-read-replicas-portal/select-max-lag.png" alt-text="Superviser le retard maximum entre réplicas":::
 
3.  Pour votre **Agrégation**, sélectionnez **Max**.


### <a name="replica-lag-metric"></a>Métrique Retard du réplica
La métrique **Retard du réplica** indique le temps écoulé depuis la dernière transaction réexécutée sur un réplica. S’il n’y a pas de transactions sur votre maître, la métrique reflète ce retard.

1. Dans le portail Azure, sélectionnez le réplica en lecture Azure Database pour PostgreSQL.

2. Sélectionnez **Métriques**. Dans la fenêtre **Métriques**, sélectionnez **Retard du réplica**.

   :::image type="content" source="./media/howto-read-replicas-portal/select-replica-lag.png" alt-text="Superviser le retard du réplica":::
 
3. Pour votre **Agrégation**, sélectionnez **Max**. 
 
## <a name="next-steps"></a>Étapes suivantes
* Découvrez-en plus sur les [réplicas en lecture dans Azure Database pour PostgreSQL](concepts-read-replicas.md).
* Découvrez comment [créer et gérer des réplicas en lecture avec l’interface de ligne de commande Azure et l’API REST](howto-read-replicas-cli.md).