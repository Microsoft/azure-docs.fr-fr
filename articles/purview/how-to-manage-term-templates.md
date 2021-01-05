---
title: Guide pratique pour gérer les modèles de termes dans un glossaire métier
description: Découvrez comment gérer les modèles de termes pour un glossaire métier dans un catalogue de données Azure Purview.
author: nayenama
ms.author: nayenama
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: conceptual
ms.date: 11/04/2020
ms.openlocfilehash: b1b811d0817d5e23adc208da14719d64d53830dd
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96550558"
---
# <a name="how-to-manage-term-templates-for-business-glossary"></a>Guide pratique pour gérer les modèles de termes dans un glossaire métier

Azure Purview vous permet de créer un glossaire des termes importants pour enrichir vos données. Chaque nouveau terme ajouté au glossaire de votre catalogue de données Purview s’appuie sur un modèle de terme qui détermine ses champs. Cet article explique comment créer un modèle de terme et des attributs personnalisés qui peuvent être associés à des termes de glossaire.

## <a name="manage-term-templates-and-custom-attributes"></a>Gestion des modèles de termes et des attributs personnalisés

Grâce aux modèles de termes, vous pouvez créer des attributs personnalisés, les regrouper et appliquer un modèle lors de la création de termes. Une fois le terme créé, le modèle associé n’est pas modifiable.

1. Sur la page **Termes du glossaire**, sélectionnez **Gérer les modèles de termes**.

   :::image type="content" source="./media/how-to-manage-glossary-term-templates/manage-term-templates.png" alt-text="Capture d’écran de Termes du glossaire > bouton Gérer les modèles de termes":::

2. Sur la page figurent à la fois les attributs système et les attributs personnalisés. Sélectionnez l’onglet **Personnalisé** pour créer ou modifier des modèles de termes.

   :::image type="content" source="./media/how-to-manage-glossary-term-templates/manage-term-custom.png" alt-text="Capture d’écran de Termes du glossaire > page Gérer les modèles de termes":::

3. Sélectionnez **+ Nouveau modèle de terme**, puis entrez le nom et la description du modèle.

   :::image type="content" source="./media/how-to-manage-glossary-term-templates/new-term-template.png" alt-text="Capture d’écran de Termes du glossaire > Gérer les modèles de termes > Nouveau modèle de terme":::

4. Sélectionnez **+ Nouvel attribut** afin de créer un attribut personnalisé pour le modèle de terme. Entrez le nom et la description de l’attribut. Le nom de l’attribut personnalisé doit être unique au sein d’un modèle de terme. Cependant, il peut être réutilisé dans d’autres modèles.

   Choisissez le type de champ dans la liste des options : **Texte**, **Choix unique**, **Choix multiples** ou **Date**. Vous pouvez également indiquer une chaîne de valeur par défaut comme types de champs de texte.  L’attribut peut également être marqué comme **Obligatoire**.

   :::image type="content" source="./media/how-to-manage-glossary-term-templates/new-attribute.png" alt-text="Capture d’écran de Termes du glossaire > page Nouvel attribut":::

5. Une fois tous les attributs personnalisés créés, sélectionnez **Prévisualiser** pour réorganiser la séquence obtenue. Vous pouvez les glisser-déplacer dans l’ordre souhaité.

   :::image type="content" source="./media/how-to-manage-glossary-term-templates/preview-term-template.png" alt-text="Capture d’écran de Termes du glossaire > Prévisualiser le modèle de terme":::

6. Une fois tous les attributs personnalisés définis, sélectionnez **Créer** pour créer un modèle de terme comportant des attributs personnalisés.

   :::image type="content" source="./media/how-to-manage-glossary-term-templates/create-term-template.png" alt-text="Capture d’écran de Termes du glossaire > Nouveau modèle de terme > bouton Créer":::

7. Les attributs personnalisés existants peuvent être marqués comme expirés. Pour cela, cochez **Marquer comme expiré**. Une fois expiré, l’attribut ne pourra pas être réactivé. Il sera grisé pour les termes existants. Les nouveaux termes créés avec ce modèle de terme ne présenteront plus cet attribut.

   :::image type="content" source="./media/how-to-manage-glossary-term-templates/expired-attribute.png" alt-text="Capture d’écran de Termes du glossaire > Modifier l’attribut > Marquer comme expiré":::

## <a name="next-steps"></a>Étapes suivantes

Suivez le [Didacticiel : Création et importation de termes de glossaire](tutorial-import-create-glossary-terms.md) pour en savoir plus.
