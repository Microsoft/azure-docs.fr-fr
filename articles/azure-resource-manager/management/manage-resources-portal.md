---
title: Gérer des ressources – Portail Azure
description: Utilisez le portail Azure et Azure Resource Manager pour gérer vos ressources. Montre comment déployer et supprimer des ressources.
author: mumian
ms.topic: conceptual
ms.date: 02/11/2019
ms.author: jgao
ms.openlocfilehash: e2274a551542b06996941b49e7d047baf3e8b3ca
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92894030"
---
# <a name="manage-azure-resources-by-using-the-azure-portal"></a>Gérer des ressources Azure à l’aide du portail Azure

Découvrez comment utiliser le [Portail Azure](https://portal.azure.com) avec [Azure Resource Manager](overview.md) pour gérer vos ressources Azure. Pour gérer des groupes de ressources, voir [Gérer des groupes de ressources Azure à l’aide du portail Azure](manage-resource-groups-portal.md).

Autres articles sur la gestion des ressources :

- [Gérer des ressources Azure à l’aide d’Azure CLI](manage-resources-cli.md)
- [Gérer des ressources Azure à l’aide d’Azure PowerShell](manage-resources-powershell.md)

[!INCLUDE [Handle personal data](../../../includes/gdpr-intro-sentence.md)]

## <a name="deploy-resources-to-a-resource-group"></a>Déployer des ressources sur un groupe de ressources

Lorsque vous avez créé un modèle Resource Manager, vous pouvez utiliser le Portail Azure pour déployer vos ressources Azure. Pour la création d’un modèle, voir [Démarrage rapide : Créer et déployer des modèles Azure Resource Manager à l’aide du portail Azure](../templates/quickstart-create-templates-use-the-portal.md). Pour le déploiement d’un modèle à l’aide du portail, voir [Déployer des ressources à l’aide de modèles Resource Manager et du Portail Azure](../templates/deploy-portal.md).

## <a name="open-resources"></a>Ressources ouvertes

Les ressources Azure sont organisées par les services Azure et groupes de ressources. Les procédures ci-dessous montrent comment ouvrir un compte de stockage nommé **mystorage0207**. La machine virtuelle réside dans un groupe de ressources appelé **mystorage0207rg**.

Pour ouvrir une ressource par le type de service :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Dans le volet gauche, sélectionnez le service Azure. Dans ce cas, **Comptes de stockage**.  Si vous ne voyez pas le service répertorié, sélectionnez **Tous les services** , puis le type de service.

    ![ouvrir une ressource azure dans le portail](./media/manage-resources-portal/manage-azure-resources-portal-open-service.png)

3. Sélectionnez la ressource à ouvrir.

    ![Capture d’écran mettant en évidence la ressource sélectionnée.](./media/manage-resources-portal/manage-azure-resources-portal-open-resource.png)

    Un compte de stockage ressemble à ceci :

    ![Capture montrant à quoi ressemble un compte de stockage.](./media/manage-resources-portal/manage-azure-resources-portal-open-resource-storage.png)

Pour ouvrir une ressource par groupe de ressources :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Dans le volet gauche, sélectionnez **Groupes de ressources** pour afficher la ressource au sein du groupe.
3. Sélectionnez la ressource à ouvrir. 

## <a name="manage-resources"></a>Gestion des ressources

Lorsque vous visualisez une ressource dans le portail, les options de gestion de cette ressource particulière s’affichent.

![gérer des ressources](./media/manage-resources-portal/manage-azure-resources-portal-manage-resource.png)

La capture d’écran montre les options de gestion pour une machine virtuelle Azure. Vous pouvez effectuer des opérations telles que démarrer, redémarrer et arrêter une machine virtuelle.

## <a name="delete-resources"></a>Supprimer des ressources

1. Ouvrez la ressource dans le portail. Pour savoir comment procéder, voir [Ouvrir des ressources](#open-resources).
2. Sélectionnez **Supprimer**. La capture d’écran suivante montre les options de gestion pour une machine virtuelle.

    ![supprimer des ressources Azure](./media/manage-resources-portal/manage-azure-resources-portal-delete-resource.png)
3. Tapez le nom de la ressource pour confirmer la suppression, puis sélectionnez **Supprimer**.

Pour plus d’informations sur l’ordre dans lequel Azure Resource Manager supprime des ressources, voir [Suppression d’un groupe de ressources par Azure Resource Manager](delete-resource-group.md).

## <a name="move-resources"></a>Déplacer des ressources

1. Ouvrez la ressource dans le portail. Pour savoir comment procéder, voir [Ouvrir des ressources](#open-resources).
2. Sélectionnez **Déplacer**. La capture d’écran suivante montre les options de gestion pour un compte de stockage.

    ![déplacer une ressource azure](./media/manage-resources-portal/manage-azure-resources-portal-move-resource.png)
3. Sélectionnez **Déplacer vers un autre groupe de ressources** ou **Déplacer vers un autre abonnement** en fonction de vos besoins.

Pour plus d’informations, consultez la page [Déplacement de ressources vers un nouveau groupe de ressources ou un abonnement](move-resource-group-and-subscription.md).

## <a name="lock-resources"></a>Verrouiller des ressources

Le verrouillage empêche d’autres utilisateurs de votre organisation de supprimer ou de modifier accidentellement des ressources stratégiques, telles qu’une ressource, un groupe de ressources ou un abonnement Azure. 

1. Ouvrez la ressource dans le portail. Pour savoir comment procéder, voir [Ouvrir des ressources](#open-resources).
2. Sélectionnez **Verrouiller**. La capture d’écran suivante montre les options de gestion pour un compte de stockage.

    ![verrouiller une ressource azure](./media/manage-resources-portal/manage-azure-resources-portal-lock-resource.png)
3. Sélectionnez **Ajouter** , puis spécifiez les propriétés de verrou.

Pour plus d’informations, consultez [Verrouiller des ressources avec Azure Resource Manager](lock-resources.md).

## <a name="tag-resources"></a>Baliser des ressources

Le balisage facilite l’organisation logique des ressources et du groupe de ressources. 

1. Ouvrez la ressource dans le portail. Pour savoir comment procéder, voir [Ouvrir des ressources](#open-resources).
2. Sélectionner **Étiquettes**. La capture d’écran suivante montre les options de gestion pour un compte de stockage.

    ![baliser une ressource azure](./media/manage-resources-portal/manage-azure-resources-portal-tag-resource.png)
3. Spécifiez les propriétés de balise, puis sélectionnez **Enregistrer**.

Pour plus d’informations, voir [Organisation des ressources Azure à l’aide de balises](tag-resources.md#portal).

## <a name="monitor-resources"></a>Surveiller les ressources

Lorsque vous ouvrez une ressource, le portail présente les graphiques et tables par défaut pour la supervision de ce type de ressource. La capture d’écran suivante montre les graphiques pour une machine virtuelle :

![superviser une ressource azure](./media/manage-resources-portal/manage-azure-resources-portal-monitor-resource.png)

Vous pouvez sélectionner l’icône d’épingle dans l’angle supérieur droit des graphiques pour épingler le graphique au tableau de bord. Pour plus d’informations sur l’utilisation des tableaux de bord, consultez [Création et partage de tableaux de bord sur le Portail Azure](../../azure-portal/azure-portal-dashboards.md).

## <a name="manage-access-to-resources"></a>Gérer l’accès aux ressources

Le [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../../role-based-access-control/overview.md) vous permet de gérer l’accès aux ressources dans Azure. Pour plus d’informations, consultez [Ajouter ou supprimer des attributions de rôles Azure à l’aide du portail Azure](../../role-based-access-control/role-assignments-portal.md).

## <a name="next-steps"></a>Étapes suivantes

- Pour vous familiariser avec Azure Resource Manager, consultez [Vue d’ensemble d’Azure Resource Manager](overview.md).
- Pour vous familiariser avec la syntaxe des modèles Resource Manager, consultez [Comprendre la structure et la syntaxe des modèles Azure Resource Manager](../templates/template-syntax.md).
- Pour apprendre à développer des modèles, consultez les [tutoriels pas à pas](../index.yml).
- Pour accéder aux schémas liés aux modèles Azure Resource Manager, consultez [Informations de référence sur les modèles](/azure/templates/).
