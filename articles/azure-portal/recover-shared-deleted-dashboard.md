---
title: Récupérer un tableau de bord supprimé dans le Portail Azure
description: Si vous supprimez un tableau de bord publié dans le Portail Azure, vous pouvez le récupérer.
ms.date: 01/21/2020
ms.topic: troubleshooting
ms.openlocfilehash: 095964691a3cb22f8a805af2e8fe37af4c47cb28
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96745619"
---
# <a name="recover-a-deleted-dashboard-in-the-azure-portal"></a>Récupérer un tableau de bord supprimé dans le Portail Azure

Si vous êtes dans le cloud Azure public et que vous supprimez un tableau de bord _publié_ dans le portail Azure, vous pouvez le récupérer dans les 14 jours suivant la suppression. SI vous êtes dans un cloud Azure Government ou si le tableau de bord n’est pas publié, vous ne pouvez pas le récupérer et vous devez le recréer. Pour plus d’informations sur la publication d’un tableau de bord, consultez [Publier un tableau de bord](azure-portal-dashboard-share-access.md#publish-dashboard). Pour récupérer un tableau de bord publié, procédez comme suit :

1. Dans le menu du Portail Azure, sélectionnez **Groupes de ressources**, puis sélectionnez le groupe de ressources dans lequel vous avez publié le tableau de bord (par défaut, il est nommé **dashboards**).

1. Sous **Journal d’activité**, développez l’opération **Supprimer le tableau de bord**. Sélectionnez l’onglet **Historique des modifications**, puis **\<deleted resource\>** .

    ![Capture d’écran de l’onglet Historique des modifications](media/recover-shared-deleted-dashboard/change-history-tab.png)

1. Sélectionnez et copiez le contenu du volet gauche, puis enregistrez-le dans un fichier texte avec une extension de fichier _.json_. Le portail utilise le fichier JSON pour recréer le tableau de bord.

    ![Capture d’écran de l’éditeur de différences de l’historique des modifications](media/recover-shared-deleted-dashboard/change-history-diff.png)

1. Dans le menu du Portail Azure, sélectionnez **Tableaux de bord**, puis sélectionnez **Charger**.

    ![Capture d’écran du chargement du tableau de bord](media/recover-shared-deleted-dashboard/dashboard-upload.png)

1. Sélectionnez le fichier JSON que vous avez enregistré. Le portail recrée le tableau de bord avec le même nom et les mêmes éléments que le tableau de bord supprimé.

1. Sélectionnez **Partager** pour publier le tableau de bord et rétablir le contrôle d’accès approprié.

    ![Capture d’écran du partage de tableau de bord](media/recover-shared-deleted-dashboard/dashboard-share.png)
