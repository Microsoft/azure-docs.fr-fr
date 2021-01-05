---
title: Gestion des attributions de rôles RBAC Synapse dans Synapse Studio
description: Cet article explique comment attribuer et révoquer des rôles RBAC Synapse dans des principaux de sécurité AAD
author: billgib
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: security
ms.date: 12/1/2020
ms.author: billgib
ms.reviewer: jrasnick
ms.openlocfilehash: f26b54a18ff137575cac208d70e4693c0b9914a8
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96572284"
---
# <a name="how-to-manage-synapse-rbac-role-assignments-in-synapse-studio"></a>Gestion des attributions de rôles RBAC Synapse dans Synapse Studio

Le contrôle d’accès en fonction du rôle (RBAC) utilise des rôles pour attribuer des autorisations aux utilisateurs, groupes et autres principaux de sécurité afin d’autoriser l’accès et l’utilisation des artefacts de code et des ressources Synapse.  [En savoir plus](./synapse-workspace-synapse-rbac.md)

Cet article explique comment ajouter et supprimer des attributions de rôles RBAC Synapse.

>[!Note]
>- Pour gérer les attributions de rôles RBAC Synapse, vous devez avoir le rôle d’administrateur Synapse sur l’espace de travail ou dans une étendue de niveau inférieur qui inclut les objets que vous souhaitez gérer. Si vous êtes un administrateur Synapse sur l’espace de travail, vous pouvez accorder l’accès à tous les objets de l’espace de travail. 
>- Les **utilisateurs invités** d’un autre locataire Active Directory ne peuvent pas voir ni gérer les attributions de rôles, même s’ils ont le rôle d’administrateur Synapse.
>- Pour vous aider à récupérer l’accès à un espace de travail dans le cas où aucun administrateur Synapse n’est attribué ou mis à votre disposition, les utilisateurs disposant des autorisations nécessaires pour gérer les attributions de rôle **Azure RBAC** sur l’espace de travail peuvent également gérer les attributions de rôle **RBAC Synapse**, en autorisant l’ajout de l’administrateur Synapse ou d’autres attributions de rôles Synapse.
>- L’accès aux pools SQL est géré à l’aide d’autorisations SQL.  À l’exception des rôles Administrateur Synapse et Administrateur Synapse SQL, les rôles RBAC Synapse n’accordent pas l’accès aux pools SQL.

>[!important]
>- L’application des modifications apportées aux attributions de rôles RBAC Synapse peut prendre de 2 à 5 minutes. 
>- Si vous gérez des autorisations RBAC Synapse en modifiant l’appartenance à des groupes de sécurité, les modifications apportées à l’appartenance sont gérées à l’aide d’Azure Active Directory.  L’application des modifications apportées aux appartenances aux groupes peut prendre 10 à 15 minutes ou plus.

## <a name="open-synapse-studio"></a>Ouvrir Synapse Studio  

Pour attribuer un rôle à un utilisateur, un groupe, un principal du service ou une identité managée, [ouvrez Synapse Studio](https://web.azuresynapse.net/) et sélectionnez votre espace de travail. 

![Se connecter à l’espace de travail](./media/common/login-workspace.png) 
 
 Une fois que vous avez ouvert votre espace de travail, développez la section **Sécurité** à gauche, puis sélectionnez **Contrôle d’accès**. 

 ![Sélectionner Contrôle d’accès dans la section Sécurité à gauche](./media/how-to-manage-synapse-rbac-role-assignments/left-nav-security-access-control.png)

L’écran du contrôle d’accès répertorie les attributions de rôles actuelles.  Vous pouvez filtrer la liste par nom de principal ou par e-mail et filtrer de manière sélective les types d’objets, les rôles ou les étendues inclus. À partir de cet écran, vous pouvez ajouter ou supprimer des attributions de rôles.  

## <a name="add-a-synapse-role-assignment"></a>Ajouter une attribution de rôle Synapse

Dans l’écran de contrôle d’accès, sélectionnez **+ Ajouter** pour créer une attribution de rôle.

![Cliquer sur + Ajouter pour créer une attribution de rôle](./media/how-to-manage-synapse-rbac-role-assignments/access-control-add.png)

Sous l’onglet Ajouter une attribution de rôle, vous pouvez créer des attributions de rôles au niveau de l’étendue de l’espace de travail ou de l’étendue de l’élément d’espace de travail. 

## <a name="add-workspace-scoped-role-assignment"></a>Ajouter une attribution de rôle au niveau de l’étendue de l’espace de travail

Tout d’abord, sélectionnez **Espace de travail** comme étendue, puis sélectionnez le rôle **RBAC Synapse**.  Sélectionnez le ou les **principaux** à attribuer au rôle, puis créez les attributions de rôles. 

![Ajouter une attribution de rôle d’espace de travail - sélectionner un rôle](./media/how-to-manage-synapse-rbac-role-assignments/access-control-workspace-role-assignment.png) 

Le rôle attribué s’applique à tous les objets applicables dans l’espace de travail.

## <a name="add-workspace-item-scoped-role-assignment"></a>Ajouter une attribution de rôle au niveau de l’étendue d’élément d’espace de travail

Pour attribuer un rôle à une étendue plus fine, sélectionnez **Élément d’espace de travail** comme étendue, puis sélectionnez l’étendue **Type d’élément**.       

![Ajouter une attribution de rôle d’élément d’espace de travail - sélectionner un type d’élément](./media/how-to-manage-synapse-rbac-role-assignments/access-control-add-workspace-item-assignment-select-item-type.png) 

Sélectionnez l’**élément** spécifique à utiliser comme étendue, puis sélectionnez le **rôle** à attribuer dans la liste déroulante.  La liste déroulante répertorie uniquement les rôles qui sont valides pour le type d’élément sélectionné. [Plus d’informations](https://go.microsoft.com/fwlink/?linkid=2148306)  

![Ajouter une attribution de rôle d’élément d’espace de travail - sélectionner un rôle](./media/how-to-manage-synapse-rbac-role-assignments/access-control-add-workspace-item-assignment-select-role.png) 
 
Ensuite, **sélectionnez le ou les principaux** auxquels le rôle doit être attribué.  Vous pouvez sélectionner plusieurs principaux de manière itérative.  Sélectionnez **Appliquer** pour créer les attributions de rôles.

## <a name="remove-a-synapse-rbac-role-assignment"></a>Supprimer une attribution de rôle RBAC Synapse

Pour révoquer l’accès au rôle RBAC Synapse, supprimez les attributions de rôle appropriées.  Dans l’écran de contrôle d’accès, utilisez les filtres pour localiser les attributions de rôles à supprimer.  Cochez les attributions de rôles, puis sélectionnez **Supprimer l’accès**.   

![Supprimer une attribution de rôle pour retirer l’accès](./media/how-to-manage-synapse-rbac-role-assignments/access-control-remove-access.png)

N’oubliez pas que les modifications apportées aux attributions de rôles sont reflétées en 2 à 5 minutes.   

## <a name="next-steps"></a>Étapes suivantes

[Comprendre les rôles RBAC Synapse requis pour effectuer des tâches courantes](./synapse-workspace-understand-what-role-you-need.md) 
