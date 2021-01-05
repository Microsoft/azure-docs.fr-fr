---
title: Gérer un serveur - Portail Azure - Serveur flexible Azure Database pour MySQL
description: Découvrez comment gérer un serveur flexible Azure Database pour MySQL à partir du portail Azure.
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.topic: how-to
ms.date: 9/21/2020
ms.openlocfilehash: 7a01863b3a0c29e94550be67ca957655cff32660
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90929859"
---
# <a name="manage-an-azure-database-for-mysql---flexible-server-preview-using-azure-portal"></a>Gérer un serveur flexible Azure Database pour MySQL (préversion) à l’aide du portail Azure


> [!IMPORTANT]
> Azure Database pour MySQL - Serveur flexible est actuellement en préversion publique.

Cet article vous explique comment gérer vos serveurs flexibles Azure Database pour MySQL (préversion). Les tâches de gestion comprennent la mise à l’échelle du calcul et du stockage, la réinitialisation du mot de passe de l’administrateur du serveur et la suppression de votre serveur.

## <a name="sign-in"></a>Se connecter
Connectez-vous au [portail Azure](https://portal.azure.com). Accédez à la ressource de votre serveur flexible dans le portail Azure.

## <a name="scale-compute-and-storage"></a>Mettre à l’échelle le calcul et le stockage

Après la création d’un serveur, vous pouvez modifier les différents [niveaux tarifaires](https://azure.microsoft.com/pricing/details/mysql/) à mesure que vos besoins évoluent. Vous pouvez également effectuer un scale-up ou un scale-down du calcul et de la mémoire en augmentant ou en diminuant le nombre de vCores.

1. Dans le Portail Azure, sélectionnez votre serveur. Sélectionnez **Calcul + Stockage** dans la section **Paramètres**.

2. Vous pouvez changer le **niveau de calcul**, le nombre de **vCores**, le **stockage** pour effectuer un scale-up du serveur en utilisant un niveau de calcul supérieur ou effectuer un scale-up au sein du même niveau en augmentant le stockage ou le nombre de vCores à une valeur de votre choix.

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

   > [!div class="mx-imgBorder"]
   > :::image type="content" source="./media/howto-manage-server-portal/delete-server.png" alt-text="mise à l’échelle du stockage du serveur flexible":::

   > [!NOTE]
   > La suppression d’un serveur est irréversible.

3. Sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes
- [Découvrir comment démarrer ou arrêter un serveur](how-to-stop-start-server-portal.md)
- [Découvrir comment restaurer un serveur](how-to-restore-server-portal.md)
- [Résoudre les problèmes de connexion](how-to-troubleshoot-common-connection-issues.md)

