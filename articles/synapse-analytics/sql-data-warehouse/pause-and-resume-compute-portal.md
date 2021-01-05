---
title: 'Démarrage rapide : Suspendre et reprendre le calcul dans le pool SQL dédié avec le portail Azure'
description: Utilisez le portail Azure pour interrompre le calcul du pool SQL dédié et ainsi réaliser des économies. Reprenez le calcul dès que vous pourrez à nouveau utiliser l’entrepôt de données.
services: synapse-analytics
author: kevinvngo
manager: craigg
ms.service: synapse-analytics
ms.topic: quickstart
ms.subservice: sql-dw
ms.date: 11/23/2020
ms.author: kevin
ms.reviewer: igorstan
ms.custom: seo-lt-2019, azure-synapse
ms.openlocfilehash: 7d258243dae06461d21e9b5f0346e419f034eea9
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97109695"
---
# <a name="quickstart-pause-and-resume-compute-in-dedicated-sql-pool-via-the-azure-portal"></a>Démarrage rapide : Suspendre et reprendre le calcul dans le pool SQL dédié avec le portail Azure

Vous pouvez utiliser le portail Azure pour suspendre et reprendre les ressources de calcul du pool SQL dédié. Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

Connectez-vous au [portail Azure](https://portal.azure.com/).

## <a name="before-you-begin"></a>Avant de commencer

Utilisez [Créer et connecter – Portail](../quickstart-create-sql-pool-portal.md) pour créer un pool SQL dédié nommé **mySampleDataWarehouse**. 

## <a name="pause-compute"></a>Suspension du calcul

Pour réduire les coûts, vous pouvez suspendre et reprendre des ressources de calcul à la demande. Par exemple, si vous n’utilisez pas la base de données pendant la nuit et les week-ends, vous pouvez la suspendre à ces moments et la reprendre pendant la journée.
 
>[!NOTE]
>Aucune ressource de calcul ne vous sera facturée tant que la base de données restera suspendue. Le stockage, en revanche, continuera à occasionner des frais. 

Suivez ces étapes pour mettre en pause un pool SQL dédié :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Accédez à la page de votre **Pool SQL dédié** pour ouvrir le pool SQL. 
3. Remarquez que l’**État** est **En ligne**.

    ![Calcul en ligne](././media/pause-and-resume-compute-portal/compute-online.png)

4. Pour mettre en pause le pool SQL dédié, cliquez sur le bouton **Suspendre**. 
5. Une question de confirmation apparaît, vous demandant si vous souhaitez continuer. Cliquez sur **Oui**.
6. Après quelques instants, **l’État** passe à **Suspension en cours**.

    ![Capture d’écran montrant le portail Azure pour un exemple d’entrepôt de données avec la valeur d’état de Suspension en cours.](./media/pause-and-resume-compute-portal/pausing.png)

7. Une fois l’opération de mise en pause terminée, l’état est **Suspendu** et la case d’option est **Reprendre**.
8. Les ressources de calcul du pool SQL dédié sont maintenant hors connexion. Le calcul ne vous sera pas facturé tant que le service n’aura pas repris.

    ![Calcul hors ligne](././media/pause-and-resume-compute-portal/compute-offline.png)


## <a name="resume-compute"></a>Reprise du calcul

Suivez ces étapes pour reprendre l’exécution d’un pool SQL dédié.

1. Accédez à la page de votre **Pool SQL dédié** pour ouvrir le pool SQL.
3. Sur la page **mySampleDataWarehouse**, **l’État** est **Suspendu**.

    ![Calcul hors ligne](././media/pause-and-resume-compute-portal/compute-offline.png)

1. Pour la reprise du pool SQL, cliquez sur **Reprendre**. 
1. Une question de confirmation apparaît, vous demandant si vous souhaitez le démarrer. Cliquez sur **Oui**.
1. **L’État** passe à **Reprise en cours**.

    ![Capture d’écran montrant le portail Azure pour un exemple d’entrepôt de données, dont le bouton Démarrer est sélectionné et la valeur d’état est Reprise en cours.](./media/pause-and-resume-compute-portal/resuming.png)

1. Une fois le pool SQL de nouveau en ligne, l’état est **En ligne** et la case d’option est **Suspendre**.
1. Les ressources de calcul du pool SQL sont à présent en ligne. Vous pouvez utiliser le service. Les frais de calcul ont repris.

    ![Calcul en ligne](././media/pause-and-resume-compute-portal/compute-online.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Les unités Data Warehouse Unit et les données stockées dans votre pool SQL dédié vous sont facturées. Ces ressources de calcul et de stockage sont facturées séparément. 

- Si vous souhaitez conserver les données dans le stockage, suspendez le calcul.
- Si vous voulez éviter des frais à venir, vous pouvez supprimer le pool SQL dédié. 

Suivez ces étapes pour nettoyer les ressources selon vos besoins.

1. Connectez-vous au [portail Azure](https://portal.azure.com), puis sélectionnez votre pool SQL dédié.

    ![Nettoyer les ressources](./media/pause-and-resume-compute-portal/clean-up-resources.png)

1. Pour suspendre le calcul, cliquez sur le bouton **Suspendre**. 

1. Pour supprimer le pool SQL dédié afin de ne pas être facturé pour le calcul ou le stockage, cliquez sur **Supprimer**.



## <a name="next-steps"></a>Étapes suivantes

Vous venez de suspendre et de reprendre le calcul pour votre pool SQL dédié. Passez à l’article suivant pour en savoir plus sur la façon de [Charger des données dans un pool SQL dédié](load-data-from-azure-blob-storage-using-polybase.md). Pour plus d’informations sur la gestion des capacités de calcul, consultez l’article [Vue d’ensemble de la gestion du calcul](sql-data-warehouse-manage-compute-overview.md). 

