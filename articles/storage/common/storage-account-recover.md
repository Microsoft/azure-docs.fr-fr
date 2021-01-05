---
title: récupérer un compte de stockage supprimé
titleSuffix: Azure Storage
description: Découvrez comment récupérer un compte de stockage supprimé dans le portail Azure.
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 12/11/2020
ms.author: tamram
ms.subservice: common
ms.openlocfilehash: f57cd3361d7888d9d7f747955257d96282274fd6
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97357343"
---
# <a name="recover-a-deleted-storage-account"></a>récupérer un compte de stockage supprimé

Un compte de stockage supprimé peut être récupéré dans certains cas à partir du portail Azure. Pour récupérer un compte de stockage, les conditions suivantes doivent être remplies :

- Le compte de stockage a été supprimé au cours des 14 derniers jours.
- Le compte de stockage a été créé avec le modèle de déploiement Azure Resource Manager.
- Aucun nouveau compte de stockage portant le même nom n’a été créé depuis la suppression du compte d’origine.

Avant de tenter de récupérer un compte de stockage supprimé, assurez-vous que le groupe de ressources pour ce compte existe. Si le groupe de ressources a été supprimé, vous devez le recréer. Il est impossible de récupérer un groupe de ressources. Pour plus d’informations, consultez [Gérer des groupes de ressources](../../azure-resource-manager/management/manage-resource-groups-portal.md).

Si le compte de stockage supprimé a utilisé des clés gérées par le client avec Azure Key Vault et que le coffre de clés a également été supprimé, vous devez restaurer le coffre de clés avant de restaurer le compte de stockage. Pour plus d'informations, consultez [Présentation de la récupération d'Azure Key Vault](../../key-vault/general/key-vault-recovery.md).

> [!IMPORTANT]
> La récupération d’un compte de stockage supprimé n’est pas garantie. La récupération est une tentative d’effort optimal. Microsoft recommande de verrouiller les ressources pour empêcher la suppression accidentelle d’un compte. Pour plus d'informations sur les verrous de ressources, consultez [Verrouiller des ressources pour éviter les modifications](../../azure-resource-manager/management/lock-resources.md).
>
> Une autre bonne pratique pour éviter la suppression accidentelle d’un compte consiste à limiter le nombre d’utilisateurs autorisés à supprimer un compte via le contrôle d’accès en fonction du rôle (Azure RBAC). Pour plus d’informations, consultez [Meilleures pratiques pour Azure RBAC](../../role-based-access-control/best-practices.md).

## <a name="recover-a-deleted-account-from-the-azure-portal"></a>Récupérer un tableau de bord supprimé à partir du portail Azure

Pour récupérer un compte de stockage supprimé à partir d’un autre compte de stockage, procédez comme suit :

1. Accédez à la page de présentation d’un compte de stockage existant dans le portail Azure.
1. Dans la section **Prise en charge + détection des problèmes**, sélectionnez **Récupérer un compte supprimé**.
1. Dans la liste déroulante, sélectionnez le compte à récupérer, comme illustré dans l’image suivante. Si le compte de stockage que vous souhaitez récupérer ne figure pas dans la liste déroulante, il ne peut pas être récupéré.

    :::image type="content" source="media/storage-account-recover/recover-account-portal.png" alt-text="Capture d’écran montrant comment récupérer un compte de stockage dans le portail Azure":::

1. Sélectionnez le bouton **Récupérer** pour restaurer le compte. Le portail affiche une notification indiquant que la récupération est en cours.

## <a name="recover-a-deleted-account-via-a-support-ticket"></a>Récupérer un compte supprimé via un ticket de support

1. Dans le portail Azure, accédez à **Aide + Support**.
1. Sélectionnez **Nouvelle demande de support**.
1. Sous l’onglet **Concepts de base**, dans le champ **Type de problème**, sélectionnez **Technique**.
1. Dans le champ **Abonnement**, sélectionnez l'abonnement qui contenait le compte de stockage supprimé.
1. Dans le champ **Service**, sélectionnez **Gestion des comptes de stockage**.
1. Dans le champ **Ressources**, sélectionnez une ressource de compte de stockage. Le compte de stockage supprimé n’apparaît plus dans la liste.
1. Ajoutez un bref résumé du problème.
1. Dans le champ **Type de problème**, sélectionnez **Suppression et récupération**.
1. Dans le champ **Sous-type de problème**, sélectionnez **Récupérer un compte de stockage supprimé**. L’illustration suivante montre un exemple de l’onglet **Concepts de base** qui est rempli :

    :::image type="content" source="media/storage-account-recover/recover-account-support-basics.png" alt-text="Capture d’écran montrant comment récupérer un compte de stockage via le ticket de support - Onglet Concepts de base":::

1. Ensuite, accédez à l’onglet **Solutions** et sélectionnez **Récupération du compte de stockage contrôlé par le client**, comme illustré dans l’image suivante :

    :::image type="content" source="media/storage-account-recover/recover-account-support-solutions.png" alt-text="Capture d’écran montrant comment récupérer un compte de stockage via le ticket de support - Onglet Solutions":::

1. Dans la liste déroulante, sélectionnez le compte à récupérer, comme illustré dans l’image suivante. Si le compte de stockage que vous souhaitez récupérer ne figure pas dans la liste déroulante, il ne peut pas être récupéré.

    :::image type="content" source="media/storage-account-recover/recover-account-support.png" alt-text="Capture d’écran montrant comment récupérer un compte de stockage via le ticket de support":::

1. Sélectionnez le bouton **Récupérer** pour restaurer le compte. Le portail affiche une notification indiquant que la récupération est en cours.

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble du compte de stockage](storage-account-overview.md)
- [Créez un compte de stockage](storage-account-create.md)
- [Mettre à niveau vers un compte de stockage v2 à usage général](storage-account-upgrade.md)
- [Déplacer un compte Stockage Azure vers une autre région](storage-account-move.md)