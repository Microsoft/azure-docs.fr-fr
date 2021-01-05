---
title: Gérer des groupes de ressources - Portail Azure
description: Utilisez le portail Azure pour gérer vos groupes de ressources dans Azure Resource Manager. Montre comment créer, lister et supprimer des groupes de ressources.
author: mumian
ms.topic: conceptual
ms.date: 03/26/2019
ms.author: jgao
ms.openlocfilehash: 6086dffaefba003461a6edd8177afab05377103d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91371250"
---
# <a name="manage-azure-resource-manager-resource-groups-by-using-the-azure-portal"></a>Gérer les groupes de ressources Azure Resource Manager à l’aide du Portail Azure

Découvrez comment utiliser le [Portail Azure](https://portal.azure.com) avec [Azure Resource Manager](overview.md) pour gérer vos groupes de ressources Azure. Pour la gestion des ressources Azure, consultez [Gérer les ressources Azure à l’aide du Portail Azure](manage-resources-portal.md).

Autres articles sur la gestion des groupes de ressources :

- [Gérer les groupes de ressources Azure à l'aide d'Azure CLI](manage-resources-cli.md)
- [Gérer les groupes de ressources Azure à l’aide d’Azure PowerShell](manage-resources-powershell.md)

[!INCLUDE [Handle personal data](../../../includes/gdpr-intro-sentence.md)]

## <a name="what-is-a-resource-group"></a>Qu’est-ce qu’un groupe de ressources

Un groupe de ressources est un conteneur réunissant les ressources associées d’une solution Azure. Le groupe de ressources peut inclure toutes les ressources de la solution, ou uniquement celles que vous souhaitez gérer en tant que groupe. Pour déterminer comment allouer des ressources aux groupes de ressources, choisissez l’approche la plus pertinente pour votre organisation. En règle générale, il convient d’ajouter des ressources qui partagent le même cycle de vie dans un même groupe de ressources afin de pouvoir facilement les déployer, les mettre à jour et les supprimer en tant que groupe.

Le groupe de ressources stocke des métadonnées sur les ressources. Par conséquent, lorsque vous spécifiez un emplacement pour le groupe de ressources, vous indiquez où stocker ces métadonnées. Pour des raisons de conformité, vous devrez peut-être vous assurer que vos données sont stockées dans une région particulière.


## <a name="create-resource-groups"></a>Créer des groupes de ressources

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Sélectionnez **Groupes de ressources**

    ![ajouter un groupe de ressources](./media/manage-resource-groups-portal/manage-resource-groups-add-group.png)
3. Sélectionnez **Ajouter**.
4. Saisissez les valeurs suivantes :

   - **Abonnement**: Sélectionnez votre abonnement Azure. 
   - **Groupe de ressources** : Entrez un nouveau nom de groupe de ressources. 
   - **Région** : Sélectionnez un emplacement Azure, tel que **USA Centre**.

     ![Créer un groupe de ressources](./media/manage-resource-groups-portal/manage-resource-groups-create-group.png)
5. Sélectionnez **Vérifier + créer**
6. Sélectionnez **Create** (Créer). La création d’un groupe de ressources prend quelques secondes.
7. Sélectionnez **Actualiser** dans le menu supérieur pour actualiser la liste de groupes de ressources, puis sélectionnez le nouveau groupe de ressources créé pour l’ouvrir. Ou bien sélectionnez **Notification** (icône en forme de cloche) en haut, puis sélectionnez **Accéder au groupe de ressources** pour ouvrir le nouveau groupe de ressources créé

    ![accéder au groupe de ressources](./media/manage-resource-groups-portal/manage-resource-groups-add-group-go-to-resource-group.png)

## <a name="list-resource-groups"></a>Répertorier les groupes de ressources

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Pour répertorier les groupes de ressources, sélectionnez **Groupes de ressources**

    ![parcourir les groupes de ressources](./media/manage-resource-groups-portal/manage-resource-groups-list-groups.png)

3. Pour personnaliser les informations affichées pour les groupes de ressources, sélectionnez **Modifier les colonnes**. La capture d’écran suivante montre les colonnes supplémentaires que vous pouvez ajouter à l’affichage :

## <a name="open-resource-groups"></a>Ouvrir des groupes de ressources

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Sélectionnez **Groupes de ressources**.
3. Sélectionnez le groupe de ressources à ouvrir.

## <a name="delete-resource-groups"></a>Supprimer des groupes de ressources

1. Ouvrez le groupe de ressources à supprimer.  Consultez [Ouvrir des groupes de ressources](#open-resource-groups).
2. Sélectionnez **Supprimer le groupe de ressources**.

    ![supprimer un groupe de ressources azure](./media/manage-resource-groups-portal/delete-group.png)

Pour plus d’informations sur l’ordre dans lequel Azure Resource Manager supprime des ressources, voir [Suppression d’un groupe de ressources par Azure Resource Manager](delete-resource-group.md).

## <a name="deploy-resources-to-a-resource-group"></a>Déployer des ressources sur un groupe de ressources

Lorsque vous avez créé un modèle Resource Manager, vous pouvez utiliser le Portail Azure pour déployer vos ressources Azure. Pour la création d’un modèle, voir [Démarrage rapide : Créer et déployer des modèles Azure Resource Manager à l’aide du portail Azure](../templates/quickstart-create-templates-use-the-portal.md). Pour le déploiement d’un modèle à l’aide du portail, voir [Déployer des ressources à l’aide de modèles Resource Manager et du Portail Azure](../templates/deploy-portal.md).

## <a name="move-to-another-resource-group-or-subscription"></a>Déplacer vers un autre groupe de ressources ou abonnement

Vous pouvez déplacer les ressources du groupe vers un autre groupe de ressources. Pour plus d’informations, consultez la page [Déplacement de ressources vers un nouveau groupe de ressources ou un abonnement](move-resource-group-and-subscription.md).

## <a name="lock-resource-groups"></a>Verrouiller des groupes de ressources

Le verrouillage empêche d’autres utilisateurs de votre organisation de supprimer ou de modifier accidentellement des ressources stratégiques, telles qu’une ressource, un groupe de ressources ou un abonnement Azure. 

1. Ouvrez le groupe de ressources à verrouiller.  Consultez [Ouvrir des groupes de ressources](#open-resource-groups).
2. Dans le volet gauche, sélectionnez **Verrous**.
3. Pour ajouter un verrou au groupe de ressources, sélectionnez **Ajouter**.
4. Entrez le **nom du verrou**, le **type de verrou** et des **notes**. Les types de verrous incluent **En lecture seule** et **Supprimer**.

    ![verrouiller un groupe de ressources azure](./media/manage-resource-groups-portal/manage-resource-groups-add-lock.png)

Pour plus d’informations, consultez [Verrouiller les ressources pour empêcher les modifications inattendues](lock-resources.md).

## <a name="tag-resource-groups"></a>Baliser des groupes de ressources

Vous pouvez appliquer des balises à des groupes de ressources pour organiser logiquement vos ressources. Pour plus d’informations, voir [Organisation des ressources Azure à l’aide de balises](tag-resources.md#portal).

## <a name="export-resource-groups-to-templates"></a>Exporter des groupes de ressources dans des modèles

Pour plus d’informations sur l’exportation de modèles, consultez [Single and multi-resource export to template - Portal](../templates/export-template-portal.md) (Exportation de ressource unique ou multiple dans un modèle - Portail).

## <a name="manage-access-to-resource-groups"></a>Gérer l'accès aux groupes de ressources

Le [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../../role-based-access-control/overview.md) vous permet de gérer l’accès aux ressources dans Azure. Pour plus d’informations, consultez [Ajouter ou supprimer des attributions de rôles Azure à l’aide du portail Azure](../../role-based-access-control/role-assignments-portal.md).

## <a name="next-steps"></a>Étapes suivantes

- Pour vous familiariser avec Azure Resource Manager, consultez [Vue d’ensemble d’Azure Resource Manager](overview.md).
- Pour vous familiariser avec la syntaxe des modèles Resource Manager, consultez [Comprendre la structure et la syntaxe des modèles Azure Resource Manager](../templates/template-syntax.md).
- Pour apprendre à développer des modèles, consultez les [tutoriels pas à pas](../index.yml).
- Pour accéder aux schémas liés aux modèles Azure Resource Manager, consultez [Informations de référence sur les modèles](/azure/templates/).
