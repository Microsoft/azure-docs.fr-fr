---
title: Fichier include
description: Fichier include
services: azure-resource-manager
author: tfitzmac
ms.service: azure-resource-manager
ms.topic: include
ms.date: 03/19/2020
ms.author: tomfitz
ms.custom: include file
ms.openlocfilehash: a00291182059506aeab9cde965fa4cbd5177ecf7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "80132099"
---
Si un utilisateur ne dispose pas de l’accès requis pour appliquer des étiquettes, vous pouvez lui attribuer le rôle **Contributeur d’étiquette**. Pour plus d’informations, consultez [Didacticiel : Accorder un accès utilisateur aux ressources Azure à l’aide du contrôle RBAC et du portail Azure](../articles/role-based-access-control/quickstart-assign-role-user-portal.md).

1. Pour afficher les balises d’une ressource ou d’un groupe de ressources, recherchez les balises existantes dans la vue d’ensemble. Si vous n’avez préalablement appliqué aucune balise, cette liste est vide.

   ![Afficher les mots clés d’une ressource ou d’un groupe de ressources](./media/resource-manager-tag-resources/view-tags.png)

1. Pour ajouter un mot clé, sélectionnez **Cliquez ici pour ajouter des mots clés**.

1. Donnez un nom et une valeur.

   ![Ajouter un mot clé](./media/resource-manager-tag-resources/add-tag.png)

1. Continuez à ajouter les mots clés dont vous avez besoin. Lorsque vous avez terminé, sélectionnez **Enregistrer**.

   ![Enregistrer les mots clés](./media/resource-manager-tag-resources/save-tags.png)

1. Les mots clés sont maintenant affichés dans la vue d’ensemble.

   ![Afficher les mots clés](./media/resource-manager-tag-resources/view-new-tags.png)

1. Pour ajouter ou supprimer un mot clé, sélectionnez **modifier**.

1. Pour supprimer un mot clé, sélectionnez l’icône de la Corbeille. Sélectionnez ensuite **Enregistrer**.

   ![Supprimer un mot clé](./media/resource-manager-tag-resources/delete-tag.png)

Pour attribuer des balises en bloc à plusieurs ressources :

1. Dans une liste de ressources, cochez la case en regard des ressources auxquelles vous souhaitez attribuer le mot clé. Ensuite, sélectionnez **Attribuer des étiquettes**.

   ![Sélectionner plusieurs ressources](./media/resource-manager-tag-resources/select-multiple-resources.png)

1. Ajoutez des noms et des valeurs. Lorsque vous avez terminé, sélectionnez **Enregistrer**.

   ![Sélectionner Attribuer](./media/resource-manager-tag-resources/select-assign.png)

Pour afficher toutes les ressources ayant un mot clé :

1. Dans le menu du portail Azure, recherchez **étiquettes**. Sélectionnez-le parmi les options disponibles.

   ![Rechercher par mot clé](./media/resource-manager-tag-resources/find-tags-general.png)

1. Sélectionnez le mot clé pour afficher les ressources associées.

   ![Sélectionner le mot clé](./media/resource-manager-tag-resources/select-tag.png)

1. Toutes les ressources associées à ce mot clé sont affichées.

   ![Afficher les ressources par mot clé](./media/resource-manager-tag-resources/view-resources-by-tag.png)
