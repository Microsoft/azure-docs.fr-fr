---
title: Restaurer – Portail Azure – Azure Database pour PostgreSQL – Serveur flexible
description: Cet article explique comment effectuer des opérations de restauration dans Azure Database pour PostgreSQL à l’aide du portail Azure.
author: sr-msft
ms.author: srranga
ms.service: postgresql
ms.topic: how-to
ms.date: 09/22/2020
ms.openlocfilehash: e69bcb3d9e4dca4c45bf9a6fe8ed4d54e7f4a8cd
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90930132"
---
# <a name="point-in-time-restore-of-a-flexible-server"></a>Restauration à un instant dans le passé d’un serveur flexible

> [!IMPORTANT]
> Azure Database pour PostgreSQL – Serveur flexible est en préversion

Cet article indique la procédure pas-à-pas à suivre pour effectuer des restaurations à un instant dans le passé sur un serveur flexible à l’aide de sauvegardes. Vous pouvez effectuer une restauration au point de restauration le plus récent ou à un point de restauration personnalisé de la période de rétention.

## <a name="pre-requisites"></a>Conditions préalables

Pour utiliser ce guide pratique, il vous faut :

-   Vous devez disposer d’Azure Database pour PostgreSQL – Serveur flexible. La même procédure s’applique également au serveur flexible configuré avec la redondance de zone.

## <a name="restoring-to-the-earliest-restore-point"></a>Restauration au point de restauration le plus ancien

Suivez ces étapes pour restaurer votre serveur flexible à l’aide de la plus ancienne sauvegarde existante.

1.  Dans le  [portail Azure](https://portal.azure.com/), choisissez le serveur flexible sur lequel vous voulez restaurer la sauvegarde.

2.  Cliquez sur **Vue d’ensemble** dans le volet gauche, puis cliquez sur **Redémarrer**
   
   :::image type="content" source="./media/how-to-restore-server-portal/restore-overview.png" alt-text="Présentation de la restauration":::

3.  La page Restaurer s’affiche et vous permet de choisir entre Point de restauration le plus ancien et Point de restauration personnalisé.

4.  Sélectionnez **Point de restauration le plus ancien**, puis fournissez un nouveau nom de serveur dans le champ **Restaurer sur un nouveau serveur**. L’horodateur le plus ancien auquel vous pouvez restaurer s’affiche. 
   
   :::image type="content" source="./media/how-to-restore-server-portal/restore-earliest.png" alt-text="Présentation de la restauration":::

5.  Cliquez sur **OK**.

6.  Une notification indique que l’opération de restauration est lancée.

## <a name="restoring-to-a-custom-restore-point"></a>Restauration à un point de restauration personnalisé

Effectuez ces étapes pour restaurer votre serveur flexible à l’aide de la plus ancienne sauvegarde existante.

1.  Dans le  [portail Azure](https://portal.azure.com/), choisissez le serveur flexible sur lequel vous voulez restaurer la sauvegarde.

2.  Dans la page de vue d’ensemble, cliquez sur **Restaurer**.
 :::image type="content" source="./media/how-to-restore-server-portal/restore-overview.png" alt-text="Présentation de la restauration":::
    
3.  La page Restaurer s’affiche et vous permet de choisir entre Point de restauration le plus ancien et Point de restauration personnalisé.

4.  Choisissez **Point de restauration personnalisé**.

5.  Sélectionnez la date et l’heure, puis entrez un nouveau nom de serveur dans le champ **Restaurer sur un nouveau serveur**. 
   
:::image type="content" source="./media/how-to-restore-server-portal/restore-custom.png" alt-text="Présentation de la restauration":::
 
6.  Cliquez sur **OK**.

7.  Une notification indique que l’opération de restauration est lancée.

## <a name="next-steps"></a>Étapes suivantes

-   Découvrir la [continuité de l’activité](./concepts-business-continuity.md)
-   Découvrir la  [haute disponibilité redondante interzone](./concepts-high-availability.md)
-   Découvrir [la sauvegarde et la récupération](./concepts-backup-restore.md)
