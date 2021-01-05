---
title: Gérer un serveur - Portail Azure - Azure Database pour PostgreSQL - Serveur flexible
description: Découvrez comment gérer un serveur Azure Database pour PostgreSQL – Serveur flexible à partir du portail Azure.
author: mksuni
ms.author: sumuth
ms.service: postgresql
ms.topic: how-to
ms.date: 09/22/2020
ms.custom: mvc
ms.openlocfilehash: 1ac418d855696138341115412dc7e2601d4cf3a1
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91961406"
---
# <a name="manage-an-azure-database-for-postgresql---flexible-server-using-the-azure-portal"></a>Gérer un serveur flexible Azure Database pour PostgreSQL à l’aide du portail Azure

> [!IMPORTANT]
> Azure Database pour PostgreSQL – Serveur flexible est en préversion

Cet article vous explique comment gérer votre serveur flexible Azure Database pour PostgreSQL. Les tâches de gestion incluent notamment la mise à l’échelle du calcul et du stockage, la réinitialisation de mot de passe et l’affichage des informations relatives au serveur.

## <a name="sign-in"></a>Se connecter

Connectez-vous au [portail Azure](https://portal.azure.com). Accédez à la ressource de votre serveur flexible dans le portail Azure.

## <a name="scale-compute-and-storage"></a>Mettre à l’échelle le calcul et le stockage

Après la création d’un serveur, vous pouvez modifier les différents [niveaux tarifaires](https://azure.microsoft.com/pricing/details/postgresql/) à mesure que vos besoins évoluent. Vous pouvez également effectuer un scale-up ou un scale-down du calcul et de la mémoire en augmentant ou en diminuant le nombre de vCores.

> [!NOTE]
> Le stockage ne peut pas faire l’objet d’un scale-down vers une valeur inférieure.

1. Dans le Portail Azure, sélectionnez votre serveur. Sélectionnez **Calcul + stockage** dans la section **Paramètres**.
2. Vous pouvez modifier le **Niveau de calcul**, le nombre de **vCores** et le **Stockage** pour effectuer un scale-up du serveur en utilisant un niveau de calcul supérieur, ou effectuer un scale-up au sein du même niveau en augmentant le stockage ou le nombre de vCores à la valeur souhaitée.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/howto-manage-server-portal/scale-server.png" alt-text="mise à l’échelle du stockage du serveur flexible":::

> [!Important]
> - Le stockage ne peut pas faire l’objet d’un scale-down.
> - La mise à l’échelle des vCores entraîne un redémarrage du serveur.

3. Sélectionnez **OK** pour enregistrer les modifications.

## <a name="reset-admin-password"></a>Réinitialiser le mot de passe de l’administrateur

Vous pouvez modifier le mot de passe du rôle d’administrateur à l’aide du Portail Azure.

1. Dans le Portail Azure, sélectionnez votre serveur. Dans la fenêtre **Vue d’ensemble**, sélectionnez **Réinitialiser le mot de passe**.
2. Entrez un nouveau mot de passe et confirmez-le. La zone de texte vous indique les exigences en matière de complexité du mot de passe.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/howto-manage-server-portal/reset-password.png" alt-text="mise à l’échelle du stockage du serveur flexible":::

3. Sélectionnez **Enregistrer** pour enregistrer le nouveau mot de passe.

## <a name="delete-a-server"></a>Supprimer un serveur

Vous pouvez supprimer votre serveur si vous n’en avez plus besoin.

1. Dans le Portail Azure, sélectionnez votre serveur. Dans la fenêtre **Vue d’ensemble**, sélectionnez **Supprimer**.
2. Tapez le nom du serveur dans la zone d’entrée pour confirmer que vous voulez supprimer le serveur.

   :::image type="content" source="./media/howto-manage-server-portal/delete-server.png" alt-text="mise à l’échelle du stockage du serveur flexible":::

   > [!IMPORTANT]
   > La suppression d’un serveur est irréversible.

  > [!div class="mx-imgBorder"]
  > ![supprimer le serveur flexible](./media/howto-manage-server-portal/delete-server.png)  

3. Sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

- [Présentation des concepts de sauvegarde et de restauration](concepts-backup-restore.md)
- [Régler et superviser le serveur](concepts-monitoring.md)
