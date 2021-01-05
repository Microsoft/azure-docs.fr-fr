---
title: Opérations de mise à l’échelle – Portail Azure – Azure Database pour PostgreSQL – Serveur flexible
description: Cet article explique comment effectuer des opérations de mise à l’échelle dans Azure Database pour PostgreSQL à l’aide du portail Azure.
author: sr-msft
ms.author: srranga
ms.service: postgresql
ms.topic: how-to
ms.date: 09/22/2020
ms.openlocfilehash: 1542bba53b51ffdf2129953a81e5d13975ade434
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90930208"
---
# <a name="scale-operations-in-flexible-server"></a>Opérations de mise à l’échelle dans un serveur flexible

> [!IMPORTANT]
> Azure Database pour PostgreSQL – Serveur flexible est disponible en préversion

Cet article explique comment effectuer des opérations de mise à l’échelle pour le calcul et le stockage. Vous serez en mesure de modifier vos niveaux de calcul entre références (SKU) expansibles, à usage général et à mémoire optimisée, notamment de choisir le nombre de vCores approprié pour exécuter votre application. Vous pouvez également effectuer un scale-up de votre stockage. Les IOPS attendues sont affichées en fonction du niveau de calcul, des vCores et de la capacité de stockage. L’estimation du coût est également affichée en fonction de votre sélection.

> [!IMPORTANT]
> Vous ne pouvez pas effectuer un scale-down du stockage.

## <a name="pre-requisites"></a>Conditions préalables

Pour utiliser ce guide pratique, il vous faut :

-   Vous devez disposer d’Azure Database pour PostgreSQL – Serveur flexible. La même procédure s’applique également au serveur flexible configuré avec la redondance de zone.
> [!IMPORTANT]
> En cas de configuration avec une haute disponibilité, vous ne pouvez pas choisir une référence (SKU) expansible. Pendant l’opération de mise à l’échelle, le serveur de secours est mis à l’échelle souhaitée, le serveur principal est basculé et le principal est mis à l’échelle. 

## <a name="scaling-the-compute-tier-and-size"></a>Mise à l’échelle du niveau et de la taille de calcul

Pour choisir le niveau de calcul, procédez comme suit.
 
1.  Dans le  [portail Azure](https://portal.azure.com/), choisissez le serveur flexible à partir duquel vous voulez restaurer la sauvegarde.

2.  Cliquez sur **Calcul + Stockage**.

3.  Une page présentant les paramètres actuels s’affiche.
 :::image type="content" source="./media/how-to-scale-compute-storage-portal/click-compute-storage.png" alt-text="vue calcul + stockage":::

4.  Vous pouvez choisir la classe de calcul parmi les niveaux expansible, à usage général et à mémoire optimisée.
   :::image type="content" source="./media/how-to-scale-compute-storage-portal/list-compute-tiers.png" alt-text="vue calcul + stockage":::


5.  Si les tailles de vCores et de mémoire par défaut vous conviennent, vous pouvez ignorer l’étape suivante.

6.  Si vous souhaitez modifier le nombre de vCores, vous pouvez cliquer sur la liste déroulante **Taille de calcul** et cliquer sur la valeur souhaitée de vCores/mémoire dans la liste.
    
    - Niveau de calcul expansible : :::image type="content" source="./media/how-to-scale-compute-storage-portal/compute-burstable-dropdown.png" alt-text="vue calcul + stockage":::

    - Niveau de calcul à usage général : :::image type="content" source="./media/how-to-scale-compute-storage-portal/compute-general-purpose-dropdown.png" alt-text="vue calcul + stockage":::

    - Niveau de calcul à mémoire optimisée : :::image type="content" source="./media/how-to-scale-compute-storage-portal/compute-memory-optimized-dropdown.png" alt-text="vue calcul + stockage":::

7.  Cliquez sur **Enregistrer**. 
8.  Un message de confirmation s’affiche. Cliquez sur **OK** si vous souhaitez continuer. 
9.  Notification de la progression de la mise à l’échelle.


## <a name="scaling-storage-size"></a>Mise à l’échelle de la taille de stockage

Procédez comme suit pour augmenter la taille de votre stockage.

1.  Dans le  [portail Azure](https://portal.azure.com/), choisissez le serveur flexible dont vous souhaitez augmenter la taille de stockage.
2.  Cliquez sur **Calcul + Stockage**.

3.  Une page présentant les paramètres actuels s’affiche.
   
:::image type="content" source="./media/how-to-scale-compute-storage-portal/click-compute-storage.png" alt-text="vue calcul + stockage":::
4.  Le champ **Taille de stockage en Gio** avec une barre de défilement affiche la taille actuelle.

5.  Faites coulisser la barre sur la taille souhaitée. Le nombre d’IOPS correspondant s’affiche. Les IOPS dépendent du niveau et de la taille de calcul. Les informations de coût sont également affichées. 

 :::image type="content" source="./media/how-to-scale-compute-storage-portal/storage-scaleup.png" alt-text="vue calcul + stockage":::

6.  Si la taille de stockage vous convient, cliquez sur **Enregistrer**. 
7.  Un message de confirmation s’affiche. Cliquez sur **OK** si vous souhaitez continuer. 
8.  Notification de la progression de la mise à l’échelle.

## <a name="next-steps"></a>Étapes suivantes

-   Découvrez la [continuité d’activité](./concepts-business-continuity.md)
-   Découvrez la [haute disponibilité](./concepts-high-availability.md)
-   Découvrez [la sauvegarde et la récupération](./concepts-backup-restore.md)
