---
title: Supprimer un sous-réseau après la suppression d’une instance managée de SQL Managed Instance
description: Découvrez comment supprimer un réseau virtuel Azure après la suppression d’une instance managée d’Azure SQL Managed Instance.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: seo-lt-2019, sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: danimir
ms.author: danil
ms.reviewer: douglas, sstein
ms.date: 06/26/2019
ms.openlocfilehash: 4ed8f6dc90debddd17282f8f96962ffd78055030
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92791662"
---
# <a name="delete-a-subnet-after-deleting-a-managed-instance-of-sql-managed-instance"></a>Supprimer un sous-réseau après la suppression d’une instance managée de SQL Managed Instance
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Cet article fournit des instructions sur la suppression manuelle d’un sous-réseau après la suppression de la dernière instance managée d’Azure SQL Managed Instance qui y réside.

Les instances managées sont déployées dans des [clusters virtuels](connectivity-architecture-overview.md#virtual-cluster-connectivity-architecture). Chaque cluster virtuel est associé à un sous-réseau. Par défaut, le cluster virtuel persiste pendant 12 heures après la dernière suppression d’instance, ce qui vous permet de créer plus rapidement des instances managées dans le même sous-réseau. La conservation d’un cluster virtuel vide n’engendre pas de frais. Pendant cette période, le sous-réseau associé au cluster virtuel ne peut pas être supprimé.

Si vous ne souhaitez pas attendre 12 heures et préférez supprimer le cluster virtuel et son sous-réseau avant ce délai, vous pouvez le faire manuellement. Supprimez le cluster virtuel manuellement à l’aide du portail Azure ou l’API de clusters virtuels.

> [!IMPORTANT]
> - Le cluster virtuel ne doit contenir aucune instance managée pour que la suppression aboutisse. 
> - La suppression d’un cluster virtuel est une opération longue qui dure environ 1,5 heure (consultez [Opérations de gestion des instances managées](./sql-managed-instance-paas-overview.md#management-operations) pour avoir des informations à jour sur le délai de suppression d’un cluster virtuel). Le cluster virtuel reste visible sur le portail tant que le processus n’est pas terminé.

## <a name="delete-a-virtual-cluster-from-the-azure-portal"></a>Supprimer un cluster virtuel à partir du portail Azure

Pour supprimer un cluster virtuel à l’aide du portail Azure, recherchez les ressources de cluster virtuel.

![Capture d’écran du portail Azure, avec la zone de recherche en surbrillance](./media/virtual-cluster-delete/virtual-clusters-search.png)

Une fois que vous avez localisé le cluster virtuel à supprimer, sélectionnez cette ressource et sélectionnez **Supprimer** . Vous êtes invité à confirmer la suppression du cluster virtuel.

![Capture d’écran du tableau de bord Clusters virtuels sur le portail Azure, avec l’option Supprimer en surbrillance](./media/virtual-cluster-delete/virtual-clusters-delete.png)

Les notifications du portail Azure vous confirment que la demande de suppression du cluster virtuel a bien été soumise. L’opération de suppression proprement dite dure environ 1,5 heure, durée pendant laquelle le cluster virtuel reste visible sur le portail. Une fois le processus terminé, le cluster virtuel n’est plus visible et le sous-réseau qui lui est associé est libéré pour être réutilisé.

> [!TIP]
> Si aucune instance gérée ne s’affiche dans le cluster virtuel et que vous ne pouvez pas supprimer le cluster virtuel, assurez-vous que vous n’avez pas un déploiement de l’instance en cours. Cela inclut les déploiements démarrés et annulés qui sont toujours en cours. Cela s’explique par le fait que ces opérations continuent d’utiliser le cluster virtuel, ce qui empêche sa suppression. Vous pouvez consulter l’onglet **Déploiements** du groupe de ressources sur lequel l’instance a été déployée pour voir tous les déploiements en cours d’exécution. Dans ce cas, attendez la fin du déploiement, supprimez l’instance managée, puis supprimez le cluster virtuel.

## <a name="delete-a-virtual-cluster-by-using-the-api"></a>Supprimer un cluster virtuel à l’aide de l’API

Pour supprimer un cluster virtuel via l’API, utilisez les paramètres d’URI spécifiés dans la [méthode de suppression de clusters virtuels](/rest/api/sql/virtualclusters/delete).

## <a name="next-steps"></a>Étapes suivantes

- Pour obtenir une vue d’ensemble, consultez [Présentation d’Azure SQL Managed Instance](sql-managed-instance-paas-overview.md).
- Découvrez l’[architecture de connectivité dans SQL Managed Instance](connectivity-architecture-overview.md).
- Apprenez à [modifier un réseau virtuel existant pour SQL Managed Instance](vnet-existing-add-subnet.md).
- Pour accéder à un tutoriel montrant comment créer un réseau virtuel, créer une instance managée et restaurer une base de données à partir d’une sauvegarde, consultez [Créer une instance managée](instance-create-quickstart.md).
- Pour les problèmes liés au DNS, consultez [Configurer un DNS personnalisé](custom-dns-configure.md).