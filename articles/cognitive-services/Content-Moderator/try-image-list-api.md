---
title: Modérer des images avec des listes personnalisées et la console d'API - Content Moderator
titleSuffix: Azure Content Moderator
description: Vous utilisez l'API Gestion de liste dans Azure Content Moderator pour créer des listes d'images personnalisées.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: conceptual
ms.date: 01/10/2019
ms.author: pafarley
ms.openlocfilehash: 0035d367017c92bd151c27e14d744ef41eace069
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85800146"
---
# <a name="moderate-with-custom-image-lists-in-the-api-console"></a>Modérer avec des listes d’images personnalisées dans la console API

Vous utilisez l’[API Gestion de liste](https://westus.dev.cognitive.microsoft.com/docs/services/57cf755e3f9b070c105bd2c2/operations/57cf755e3f9b070868a1f672) dans Azure Content Moderator pour créer des listes d’images personnalisées. Utilisez les listes d’images personnalisées avec l’API Modération d’image. L’opération de modération d’image évalue votre image. Si vous créez des listes personnalisées, l’opération les compare aussi aux images présentes dans vos listes personnalisées. Vous pouvez utiliser des listes personnalisées pour bloquer ou autoriser l’image.

> [!NOTE]
> Il existe une limite maximale de **5 listes d’images**, chaque liste ne devant **pas dépasser 10 000 images**.
>

Vous utilisez l’API Gestion de liste pour effectuer les tâches suivantes :

- Créer une liste.
- Ajouter des images à une liste.
- Prendre des captures d’écran d’images présentes dans une liste.
- Supprimer des images d’une liste.
- Supprimer une liste.
- Modifier les informations d’une liste.
- Actualiser l’index afin que les changements apportés à la liste soient inclus dans une nouvelle analyse.

## <a name="use-the-api-console"></a>Utiliser la console d’API
Avant de pouvoir tester l’API dans la console en ligne, vous avez besoin de votre clé d’abonnement. Elle se trouve sous l’onglet **Paramètres**, dans la zone **Ocp-Apim-Subscription-Key**. Pour plus d’informations, consultez la [vue d’ensemble ](overview.md) de Content Moderator.

## <a name="refresh-search-index"></a>Actualiser l’index de recherche

Une fois les modifications effectuées à une liste d’images, vous devez actualiser son index pour inclure les modifications aux futures analyses. Cette étape est semblable au fonctionnement d’un moteur de recherche sur votre bureau (s’il est activé) ou d’un moteur de recherche web qui actualise en permanence son index pour inclure les nouveaux fichiers ou les nouvelles pages.

1. Dans la [référence de l’API Gestion de liste d’images](https://westus.dev.cognitive.microsoft.com/docs/services/57cf755e3f9b070c105bd2c2/operations/57cf755e3f9b070868a1f672), dans le menu gauche, sélectionnez **Listes d’images**, puis **Actualiser l’index de recherche**.

   La page **Listes d’images - Actualiser l’index de recherche** s’ouvre.

2. Pour l’option **Open API testing console** (Ouvrir la console de test d’API), sélectionnez la région qui décrit le mieux votre emplacement. 
 
    ![Sélection de la région sur la page Listes d’images - Actualiser l’index de recherche](images/test-drive-region.png)

    La console de l’API **Listes d’images - Actualiser l’index de recherche** s’ouvre.

3. Dans le champ **listId**, saisissez l’ID de la liste. Entrez votre clé d’abonnement, puis sélectionnez **Envoyer**.

   ![Champ de contenu de réponse de la console Listes d’images - Actualiser l’index de recherche](images/try-image-list-refresh-1.png)


## <a name="create-an-image-list"></a>Créer une liste d’image

1. Accédez à la [référence de l’API Gestion de liste d’images](https://westus.dev.cognitive.microsoft.com/docs/services/57cf755e3f9b070c105bd2c2/operations/57cf755e3f9b070868a1f672).

   La page **Listes d’images - Créer** s’ouvre. 

3. Pour l’option **Open API testing console** (Ouvrir la console de test d’API), sélectionnez la région qui décrit le mieux votre emplacement.

   ![Sélection de la région sur la page Listes d’images - Créer](images/test-drive-region.png)

   La console d’API **Listes d’images - Créer** s’ouvre.
 
4. Dans la zone **Ocp-Apim-abonnement-Key**, entrez votre clé d’abonnement.

5. Dans la zone **Corps de la demande**, entrez les valeurs pour **Noms** (par exemple, MyList) et **Description**.

   ![Nom et description du corps de la demande dans la console Listes d’images - Créer](images/try-terms-list-create-1.png)

6. Utilisez les espaces réservées aux paires clé-valeur pour assigner plus de métadonnées descriptives à votre liste.

    ```json
    {
        "Name": "MyExclusionList",
        "Description": "MyListDescription",
        "Metadata": 
        {
            "Category": "Competitors",
            "Type": "Exclude"
        }
    }
    ```

   Ajoutez des métadonnées de liste en tant que paire clé-valeur, pas les images.
 
7. Sélectionnez **Envoyer**. Votre liste est créée. Notez la valeur **ID** associée à la nouvelle liste. Vous en avez besoin pour d’autres fonctions de gestion de liste d’images.

   ![Console Listes d’images - Créer, la zone Contenu de la réponse affiche l’ID de la liste](images/try-terms-list-create-2.png)
 
8. Ajoutez ensuite les images à MyList. Dans le menu gauche, sélectionnez **Image** puis **Ajouter image**.

   La page **Image - Ajouter image** s’ouvre. 

9. Pour l’option **Open API testing console** (Ouvrir la console de test d’API), sélectionnez la région qui décrit le mieux votre emplacement.

   ![Sélection de la région sur la page Image - Ajouter image](images/test-drive-region.png)

   La console d’API **Image - Ajouter image** s’ouvre.
 
10. Dans la zone **listId**, entrez l’ID de liste que vous avez généré, puis l’URL de l’image que vous voulez ajouter. Entrez votre clé d’abonnement, puis sélectionnez **Envoyer**.

11. Pour vérifier que l’image a bien été ajoutée à la liste, dans le menu gauche, sélectionnez **Image** puis **Obtenir tous les ID d’image**.

    La console d’API **Image - Obtenir tous les ID d’image** s’ouvre.
  
12. Dans la zone **listId**, entrez l’ID de liste, puis votre clé d’abonnement. Sélectionnez **Envoyer**.

    ![Console Image - Obtenir tous les ID d’image, la zone Contenu de la réponse répertorie les images que vous avez entrées](images/try-image-list-create-11.png)
 
10. Ajoutez quelques images de plus. Maintenant que vous avez créé une liste d’image personnalisée, essayez d’[évaluer les images](try-image-api.md) avec la liste d’image personnalisée. 

## <a name="delete-images-and-lists"></a>Supprimer des images et des listes

La suppression d’une image ou d’une liste est simple. Vous pouvez utiliser l’API pour effectuer les tâches suivantes :

- Supprimez une image. (**Image - Supprimer**)
- Supprimer toutes les images d’une liste sans supprimer la liste. (**Image - Supprimer toutes les images**)
- Supprimer une liste et tout son contenu. (**Listes d’images - Supprimer**)

Cet exemple supprime une seule image :

1. Dans la [référence de l’API Gestion de liste d’images](https://westus.dev.cognitive.microsoft.com/docs/services/57cf755e3f9b070c105bd2c2/operations/57cf755e3f9b070868a1f672), dans le menu gauche, sélectionnez **Image**, puis **Supprimer**. 

   La page **Image - Supprimer** s’ouvre.

2. Pour l’option **Open API testing console** (Ouvrir la console de test d’API), sélectionnez la région qui décrit le mieux votre emplacement. 

   ![Sélection d’une région sur la page Image - Supprimer](images/test-drive-region.png)
 
   La console d’API **Image - Supprimer** s’ouvre.
 
3. Dans la zone **listId**, entrez l’ID de la liste dans laquelle l’image à supprimer se trouve.  Il s’agit du numéro retourné dans la console **Image - Obtenir tous les ID d’image** de MyList. Entrez ensuite l’**ID d’image** de l’image à supprimer. 

Dans notre exemple, l’ID de la liste est **58953**, la valeur de **ContentSource**. L’ID de l’image est **59021**, la valeur de **ContentIds**.

1. Entrez votre clé d’abonnement, puis sélectionnez **Envoyer**.

1. Pour vérifier que l’image a bien été supprimée, utilisez la console **Image - Obtenir tous les ID d’image**.
 
## <a name="change-list-information"></a>Modifier les informations d’une liste

Vous pouvez modifier le nom et la description d’une liste, et ajouter des éléments de métadonnées.

1. Dans la [référence de l’API Gestion de liste d’images](https://westus.dev.cognitive.microsoft.com/docs/services/57cf755e3f9b070c105bd2c2/operations/57cf755e3f9b070868a1f672), dans le menu gauche, sélectionnez **Listes d’images**, puis **Mettre à jour les détails**. 

   La page **Listes d’images - Mettre à jour les détails** s’ouvre.

2. Pour l’option **Open API testing console** (Ouvrir la console de test d’API), sélectionnez la région qui décrit le mieux votre emplacement.  

    ![Sélection d’une région sur la page Listes d’images - Mettre à jour les détails](images/test-drive-region.png)

    La console d’API **Listes d’images - Mettre à jour les détails** s’ouvre.
 
3. Dans la zone **listId**, entrez l’ID de liste, puis votre clé d’abonnement.

4. Dans la zone **Corps de la demande**, faites vos changements, puis sélectionnez le bouton **Envoyer** sur la page.

   ![Modifications du corps de la demande dans la console Listes d’images - Mettre à jour les détails](images/try-terms-list-change-1.png)
 

## <a name="next-steps"></a>Étapes suivantes

Utilisez l’API REST dans votre code ou commencez par le [guide de démarrage rapide des listes d’images .NET](image-lists-quickstart-dotnet.md) pour en intégrer à votre application.
