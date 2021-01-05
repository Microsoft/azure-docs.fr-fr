---
title: Ajouter des outils de migration dans Azure Migrate
description: Découvrez comment ajouter des outils de migration dans Azure Migrate.
author: anvar-ms
ms.author: anvar
ms.manager: bsiva
ms.topic: how-to
ms.date: 11/23/2020
ms.openlocfilehash: 97051e97ec9868f6941b579241e16e62fdd2162b
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96751782"
---
# <a name="add-migration-tools"></a>Ajouter des outils de migration

Cet article explique comment ajouter des outils de migration dans [Azure Migrate](./migrate-services-overview.md).

- Si vous souhaitez ajouter un outil de migration et que vous n’avez pas encore configuré de projet Azure Migrate, suivez cet [article](create-manage-projects.md).
- Si vous avez ajouté un outil ISV pour la migration, [suivez les étapes](prepare-isv-movere.md) de préparation à l’utilisation de l’outil.

## <a name="select-a-migration-scenario"></a>Sélectionnez un scénario de migration

1. Dans le projet Azure Migrate, cliquez sur **Vue d’ensemble**.
2. Sélectionnez le scénario de migration que vous souhaitez utiliser :

    - Pour migrer des machines et des charges de travail vers Azure, sélectionnez **Évaluer et migrer des serveurs**.
    - Pour migrer des machines locales, sélectionnez **Évaluer et migrer des bases de données**.
    - Pour migrer des applications web locales, sélectionnez **Explorer davantage** > **Web Apps**.
    - Pour migrer des données vers Azure à l’aide de Data Box, sélectionnez **Explorer davantage** > **Data Box**.

    ![Options de sélection d’un scénario de migration](./media/how-to-migrate/migrate-scenario.png)


## <a name="select-a-server-migration-tool"></a>Sélectionnez un outil de migration de serveur

1. Ajoutez un outil :

    - Si vous avez créé un projet Azure Migrate à l’aide de l’option **Évaluer et migrer des serveurs** du portail, l’outil Migration de serveur Azure Migrate est automatiquement ajouté au projet. Pour ajouter des outils de migration supplémentaires, dans **Serveurs**, en regard d’**Outils de migration**, sélectionnez **Ajouter d’autres outils**.
    
         ![Bouton permettant d’ajouter d’autres outils de migration](./media/how-to-migrate/add-migration-tools.png)

    - Si vous avez utilisé une autre option pour créer un projet et que vous n’avez pas encore d’outils de migration, dans **Serveurs** > **Outils de migration**, sélectionnez **Cliquez ici**.

    ![Bouton permettant d’ajouter les premiers outils de migration](./media/how-to-migrate/no-migration-tool.png)

2. Dans **Azure Migrate** > **Ajouter des outils**, sélectionnez les outils que vous voulez ajouter. Sélectionnez ensuite **Ajouter un outil**.

    ![Sélectionner des outils d’évaluation dans la liste](./media/how-to-migrate/select-migration-tool.png)


## <a name="select-a-database-migration-tool"></a>Sélectionnez un outil de migration de base de données

Si vous avez créé un projet Azure Migrate à l’aide de l’option **Évaluer et migrer des bases de données** du portail, l’outil Migration de base de données est automatiquement ajouté au projet. 

1. Si l’outil de migration de base de données ne se trouve pas dans le projet, dans **Bases de données** > **Outils d’évaluation**, sélectionnez **Cliquez ici**.
    
    ![Ajouter des outils de migration de base de données](./media/how-to-migrate/no-database-migration-tool.png)


2. Dans **Azure Migrate** > **Ajouter des outils**, sélectionnez l’outil Migration de base de données. Sélectionnez ensuite **Ajouter un outil**.

    ![Sélectionner l’outil de migration de base de données dans la liste](./media/how-to-migrate/select-database-migration-tool.png)

    

## <a name="select-a-web-app-migration-tool"></a>Sélectionnez un outil de migration d’application web

Si vous avez créé un projet Azure Migrate à l’aide de l’option **Explorer davantage** > **Applications web** du portail, l’outil de migration d’application web est automatiquement ajouté au projet. 

1. Si l’outil de migration d’application web ne se trouve pas dans le projet, dans **Web Apps** > **Outils d’évaluation**, sélectionnez **Cliquez ici**.

    ![Ajouter des outils de migration d’application web](./media/how-to-migrate/no-web-app-migration-tool.png)
 

2. Dans **Azure Migrate** > **Ajouter des outils**, sélectionnez l’outil Migration d’application web. Sélectionnez ensuite **Ajouter un outil**.

    ![Sélectionner des outils d’évaluation d’application web dans la liste](./media/how-to-migrate/select-web-app-migration-tool.png)


## <a name="order-an-azure-data-box"></a>Commander une solution Azure Data Box

Pour migrer de grandes quantités de données vers Azure, vous pouvez commander une solution Azure Data Box pour un transfert de données hors connexion.

1. Dans **Vue d’ensemble**, sélectionnez **Explorer davantage**.
2. Dans **Explorer davantage**, sélectionnez **Data Box**.
3. Dans **Bien démarrer avec Data Box**, sélectionnez l’abonnement et le groupe de ressources que vous voulez utiliser lors de la commande d’une solution Data Box.
4. Le **Type de transfert** est une importation vers Azure. Spécifiez le pays dans lequel les données résident et la région Azure vers laquelle vous voulez transférer les données. 
5. Cliquez sur **Appliquer** pour enregistrer les paramètres.

## <a name="next-steps"></a>Étapes suivantes

Essayez une migration à l’aide de l’outil de migration de serveur Azure Migrate pour les machines virtuelles [Hyper-V](tutorial-migrate-hyper-v.md) ou [VMware](tutorial-migrate-vmware.md).
