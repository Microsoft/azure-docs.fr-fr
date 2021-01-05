---
title: Restauration - Portail Azure - Azure Database pour MySQL - Serveur flexible
description: Cet article explique comment effectuer des opérations de restauration dans Azure Database pour MySQL à l’aide du portail Azure.
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.topic: how-to
ms.date: 09/21/2020
ms.openlocfilehash: 062d53fcb122ebacd004d7dca5e11f5a883354cd
ms.sourcegitcommit: 80034a1819072f45c1772940953fef06d92fefc8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93241954"
---
# <a name="point-in-time-restore-of-a-azure-database-for-mysql---flexible-server-preview"></a>Restauration à un instant dans le passé d’un serveur flexible Azure Database pour MySQL (préversion)


> [!IMPORTANT]
> Azure Database pour MySQL - Serveur flexible est actuellement en préversion publique.

Cet article indique la procédure pas-à-pas à suivre pour effectuer des restaurations à un instant dans le passé sur un serveur flexible à l’aide de sauvegardes.

## <a name="prerequisites"></a>Prérequis

Pour utiliser ce guide pratique, il vous faut :

-   Vous devez disposer d’un serveur flexible Azure Database pour MySQL.

## <a name="restore-to-the-latest-restore-point"></a>Restaurer au dernier point de restauration

Effectuez ces étapes pour restaurer votre serveur flexible à l’aide de la plus ancienne sauvegarde existante.

1.  Dans le [portail Azure](https://portal.azure.com/), choisissez le serveur flexible à partir duquel vous voulez restaurer la sauvegarde.

2.  Cliquez sur **Vue d’ensemble** dans le panneau gauche.

3.  Dans la page de vue d’ensemble, cliquez sur **Restaurer**.

    [Espace réservé]

4.  La page Restaurer s’affiche et vous permet de choisir entre **Dernier point de restauration** et Point de restauration personnalisé.

5.  Sélectionnez **Dernier point de restauration**.


6.  Indiquez un nouveau nom de serveur dans le champ **Restaurer sur un nouveau serveur**.

    :::image type="content" source="./media/concept-backup-restore/restore-blade-latest.png" alt-text="Heure de restauration la plus ancienne":::

8.  Cliquez sur **OK**.

9.  Une notification indique que l’opération de restauration est lancée.

## <a name="restoring-to-a-custom-restore-point"></a>Restauration à un point de restauration personnalisé

Effectuez ces étapes pour restaurer votre serveur flexible à l’aide de la plus ancienne sauvegarde existante.

1.  Dans le [portail Azure](https://portal.azure.com/), choisissez le serveur flexible à partir duquel vous voulez restaurer la sauvegarde.

2.  Dans la page de vue d’ensemble, cliquez sur **Restaurer**.

    [Espace réservé]

3.  La page Restaurer s’affiche et vous permet de choisir entre Point de restauration le plus ancien et Point de restauration personnalisé.

4.  Choisissez **Point de restauration personnalisé**.

5.  Sélectionnez la date et l’heure.

6.  Indiquez un nouveau nom de serveur dans le champ **Restaurer sur un nouveau serveur**.

6.  Indiquez un nouveau nom de serveur dans le champ **Restaurer sur un nouveau serveur**. 
   
    :::image type="content" source="./media/concept-backup-restore/restore-blade-custom.png" alt-text="vue d’ensemble":::
 
7.  Cliquez sur **OK**.

8.  Une notification indique que l’opération de restauration est lancée.

## <a name="next-steps"></a>Étapes suivantes

Espace réservé
