---
title: Comment étiqueter plusieurs termes de glossaire dans une liste de ressources sélectionnées
description: Découvrez les ressources de modification en bloc dans Azure Purview.
author: nayenama
ms.author: nayenama
ms.service: data-catalog
ms.subservice: data-catalog-gen2
ms.topic: conceptual
ms.date: 11/24/2020
ms.openlocfilehash: c42a6894c33993dc9aee5a9fdd10b1c3a3627320
ms.sourcegitcommit: 287c20509c4cf21d20eea4619bbef0746a5cd46e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97372016"
---
# <a name="how-to-bulk-edit-assets-to-tag-glossary-terms"></a>Comment modifier en bloc des ressources pour étiqueter des termes de glossaire

Cet article explique comment étiqueter plusieurs termes de glossaire dans une liste de ressources sélectionnées en une seule action.

### <a name="add-assets-to-view-selected-list-using-search"></a>Ajouter des ressources pour afficher la liste sélectionnée à l’aide de la recherche

1. Recherchez la ressource de données que vous souhaitez ajouter à la liste pour la modifier en bloc.

2. Sur la page des résultats de la recherche, pointez sur la ressource que vous souhaitez ajouter à la liste **Afficher la sélection** de modification en bloc pour voir une case.

   :::image type="content" source="media/how-to-bulk-edit-assets/asset-checkbox.png" alt-text="Capture d'écran de la case.":::

3. Cochez la case pour l’ajouter à la liste **Affiche la sélection** de modification en bloc. Une fois l’ajout effectué, l’icône des éléments sélectionnés s’affiche au bas de la page.

   :::image type="content" source="media/how-to-bulk-edit-assets/selected-list.png" alt-text="Capture d'écran de la liste.":::

4. Répétez les étapes ci-dessus pour ajouter toutes les ressources de données à la liste.

### <a name="add-assets-to-view-selected-list-from-asset-detail-page"></a>Ajouter des ressources pour afficher la liste Afficher la sélection à partir de la page des détails de la ressource

1. Sur la page des détails de la ressource, cochez la case située dans le coin supérieur droit pour ajouter la ressource à la liste **Afficher la sélection** de modification en bloc.

   :::image type="content" source="media/how-to-bulk-edit-assets/asset-list.png" alt-text="Capture d'écran de la ressource.":::

### <a name="bulk-edit-assets-in-the-view-selected-list-to-add-replace-or-remove-glossary-terms"></a>Modifiez en bloc des ressources de la liste Afficher la sélection pour ajouter, remplacer ou supprimer des termes de glossaire.

1. Une fois que vous avez terminé l’identification de toutes les ressources de données qui doivent être modifiées en bloc, sélectionnez la liste **Afficher la sélection** à partir de la page des résultats de la recherche ou de la page des détails de la ressource.

:::image type="content" source="media/how-to-bulk-edit-assets/view-list.png" alt-text="Capture d'écran de la vue.":::

2. Passez en revue la liste et sélectionnez **Supprimer** si vous souhaitez supprimer des éléments.

:::image type="content" source="media/how-to-bulk-edit-assets/remove-list.png" alt-text="Capture d'écran de la suppression.":::

3. Sélectionnez la modification en bloc pour ajouter, supprimer ou remplacer des termes de glossaire pour toutes les ressources sélectionnées.

4. Pour ajouter des termes de glossaire, sélectionnez l’opération **Ajouter**. Sélectionnez tous les termes de glossaire à ajouter dans la nouvelle valeur. Sélectionnez Appliquer lorsque vous avez terminé.
    - L’opération d’ajout ajoute la nouvelle valeur à la liste des termes de glossaire déjà étiquetés aux ressources de données.  
   
    :::image type="content" source="media/how-to-bulk-edit-assets/add-list.png" alt-text="Capture d'écran de l'ajout.":::

5. Pour remplacer les termes du glossaire, sélectionnez l’opération **Remplacer par**. Sélectionnez tous les termes de glossaire à remplacer dans la nouvelle valeur. Sélectionnez Appliquer lorsque vous avez terminé.
    - L’opération de remplacement remplace tous les termes de glossaire pour les ressources de données sélectionnées par les termes sélectionnés dans la nouvelle valeur.
   
6. Pour supprimer des termes de glossaire, sélectionnez l’opération **Supprimer**. Sélectionnez Appliquer lorsque vous avez terminé.
    - L’opération de suppression va supprimer tous les termes de glossaire pour les ressources de données sélectionnées.
   
    :::image type="content" source="media/how-to-bulk-edit-assets/replace-list.png" alt-text="Capture d'écran des termes à supprimer.":::

7. Une fois que vous avez terminé, fermez le panneau de modification en bloc en sélectionnant **Fermer** ou **Supprimer tout et fermer**. Fermer ne supprimera pas les ressources sélectionnées, tandis que Supprimer tout et fermer supprimera toutes les ressources sélectionnées.
    :::image type="content" source="media/how-to-bulk-edit-assets/close-list.png" alt-text="Capture d'écran de la fermeture.":::

   > [!Important]
   > Le nombre recommandé de ressources pour la modification en bloc est de 15. La sélection de plus de 15 éléments peut entraîner des problèmes de performances.
   > La case **Afficher la sélection** ne s’affiche que si au moins une ressource est sélectionnée.


Suivez le [Didacticiel : Créer et importer des termes de glossaire](how-to-create-import-export-glossary.md) pour en savoir plus.
