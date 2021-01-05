---
title: Comment rechercher, modifier et supprimer un projet - Custom Translator
titleSuffix: Azure Cognitive Services
description: Custom Translator offre différents moyens pour gérer vos projets efficacement. Vous pouvez créer des projets, effectuer des recherches en fonction de vos critères et modifier vos projets. La suppression d’un projet est également possible dans Custom Translator.
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.date: 08/17/2020
ms.author: swmachan
ms.topic: conceptual
ms.openlocfilehash: 4c48a8a18a3ad0487b4c3840847c7f5edbb6d29f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88510342"
---
# <a name="search-edit-and-delete-projects"></a>Rechercher, modifier et supprimer des projets

Custom Translator offre de nombreux moyens pour gérer vos projets efficacement. Vous pouvez créer de nombreux projets, effectuer des recherches en fonction de vos critères et modifier vos projets. La suppression d’un projet est également possible dans Custom Translator.  

## <a name="search-and-filter-projects"></a>Rechercher et filtrer les projets

L’outil de filtrage vous permet de rechercher des projets selon différentes conditions de filtrage. Il filtre le nom du projet, l’état, la langue source et la langue cible, ainsi que la catégorie du projet.

1. Cliquez sur le bouton de filtre.

    ![Rechercher dans le projet](media/how-to/how-to-search-project.png)

2. Vous pouvez filtrer par l’un (ou l’ensemble) des champs suivants : nom du projet, langue source, langue cible, catégorie et disponibilité du projet.

3. Cliquez sur Appliquer.

    ![Rechercher dans les options de filtre de projet](media/how-to/how-to-search-project-filters.png)

4. Appuyez sur « Effacer » afin d’effacer le filtre et d’afficher tous vos projets.

## <a name="edit-a-project"></a>Modifier un projet

Custom Translator vous permet de modifier le nom et la description d’un projet. Les autres métadonnées du projet comme la catégorie, la langue source et la langue cible ne peuvent pas être modifiées. Les étapes ci-dessous expliquent comment modifier un projet.

1. Cliquez sur l’icône crayon qui apparaît lors du survol d’un projet.

    ![Modifier le projet](media/how-to/how-to-edit-project.png)

2. Dans la boîte de dialogue, vous pouvez modifier le nom ou la description du projet, la description de la catégorie et l’étiquette du projet si aucun modèle n’est déployé. Vous ne pouvez pas modifier la catégorie ni la paire de langues une fois que le projet est créé.

    ![Boîte de dialogue Modifier le projet](media/how-to/how-to-edit-project-dialog.png)

3. Cliquez sur le bouton « Enregistrer ».

## <a name="delete-a-project"></a>Supprimer un projet

Vous pouvez supprimer un projet lorsque vous n’en avez plus besoin. Faites en sorte que le projet ne comporte aucun modèle actif, par exemple à l’état Déployé, Formation soumise, Traitement des données, Déploiement, etc., sinon, l’opération de suppression échoue. Les étapes suivantes décrivent le processus de suppression d’un projet.

1. Placez le pointeur sur n’importe quel enregistrement du projet, puis cliquez sur l’icône Corbeille.

   ![Supprimer le projet](media/how-to/how-to-delete-project.png)

2. Confirmez la suppression. La suppression d’un projet supprimera tous les modèles qui ont été créés dans ce projet. La suppression du projet n’affectera pas vos documents.

   ![Boîte de dialogue de confirmation de suppression](media/how-to/how-to-delete-project-confirm.png)

## <a name="next-steps"></a>Étapes suivantes

- [Chargez des documents](how-to-upload-document.md) pour commencer à construire votre modèle de traduction personnalisé.
