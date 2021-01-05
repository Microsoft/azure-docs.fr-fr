---
title: Modérer du texte avec des listes de termes personnalisées - Content Moderator
titleSuffix: Azure Cognitive Services
description: Utilisez l’API Gestion des listes pour créer des listes de termes personnalisées à utiliser avec l’API Modération de texte.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: conceptual
ms.date: 01/10/2019
ms.author: pafarley
ms.openlocfilehash: f1dfe88741a304da92901d3997c746654336ef54
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85800038"
---
# <a name="moderate-with-custom-term-lists-in-the-api-console"></a>Modérer avec des listes de termes personnalisées dans la console d’API

La liste générale de termes par défaut d’Azure Content Moderator est suffisante pour la plupart des besoins de modération de contenu. Toutefois, vous devrez probablement rechercher des termes spécifiques à votre organisation. Par exemple, vous voudrez peut-être identifier les noms de concurrents pour un examen approfondi. 

Utilisez l’[API Gestion des listes](https://westus.dev.cognitive.microsoft.com/docs/services/57cf755e3f9b070c105bd2c2/operations/57cf755e3f9b070868a1f67f) pour créer des listes de termes personnalisées à utiliser avec l’API Modération de texte. L’opération **Texte - Écran** analyse votre texte pour détecter les propos injurieux et compare également le texte à des listes de blocage personnalisées et partagées.

> [!NOTE]
> Il existe une limite maximale de **5 listes de termes**, chaque liste ne devant **pas dépasser 10 000 termes**.
>

Vous pouvez utiliser l’API Gestion des listes pour effectuer les tâches suivantes :
- Créer une liste.
- Ajouter des termes à une liste.
- Passer au crible un texte pour détecter des termes par rapport aux termes d’une liste.
- Supprimer des termes d’une liste.
- Supprimer une liste.
- Modifier les informations d’une liste.
- Actualiser l’index afin que les changements apportés à la liste soient inclus dans une nouvelle analyse.

## <a name="use-the-api-console"></a>Utiliser la console d’API

Avant de pouvoir tester l’API dans la console en ligne, vous avez besoin de votre clé d’abonnement. Cette clé se trouve sous l’onglet **Paramètres**, dans la zone **Ocp-Apim-Subscription-Key**. Pour plus d’informations, consultez la [vue d’ensemble ](overview.md) de Content Moderator.

## <a name="refresh-search-index"></a>Actualiser l’index de recherche

Une fois les modifications effectuées dans une liste de termes, vous devez actualiser son index pour inclure les modifications aux futures recherches. Cette étape est semblable au fonctionnement d’un moteur de recherche sur votre bureau (s’il est activé) ou d’un moteur de recherche web qui actualise en permanence son index pour inclure les nouveaux fichiers ou les nouvelles pages.

1. Dans la [référence de l’API Gestion de liste de termes](https://westus.dev.cognitive.microsoft.com/docs/services/57cf755e3f9b070c105bd2c2/operations/57cf755e3f9b070868a1f67f), dans le menu gauche, sélectionnez **Listes de termes**, puis **Actualiser l’index de recherche**. 

   La page **Listes de termes - Actualiser l’index de recherche** s’ouvre.

2. Pour l’option **Open API testing console** (Ouvrir la console de test d’API), sélectionnez la région qui décrit le mieux votre emplacement. 

   ![Sélection de la région sur la page Listes de termes - Actualiser l’index de recherche](images/test-drive-region.png)

   La console d’API **Listes de termes - Actualiser l’index de recherche** s’ouvre.

3. Dans le champ **listId**, saisissez l’ID de la liste. Entrez votre clé d’abonnement, puis sélectionnez **Envoyer**.

   ![Champ de contenu de réponse de la console API Listes de termes - Actualiser l’index de recherche](images/try-terms-list-refresh-1.png)

## <a name="create-a-term-list"></a>Créer une liste de termes
1. Accédez à la [référence de l’API Gestion de liste de termes](https://westus.dev.cognitive.microsoft.com/docs/services/57cf755e3f9b070c105bd2c2/operations/57cf755e3f9b070868a1f67f). 

   La page **Listes de termes - Créer** s’ouvre.

2. Pour l’option **Open API testing console** (Ouvrir la console de test d’API), sélectionnez la région qui décrit le mieux votre emplacement. 

   ![Sélection de la région sur la page Listes de termes - Créer](images/test-drive-region.png)

   La console d’API **Listes de termes - Créer** s’ouvre.
 
3. Dans la zone **Ocp-Apim-abonnement-Key**, entrez votre clé d’abonnement.

4. Dans la zone **Corps de la demande**, entrez les valeurs pour **Noms** (par exemple, MyList) et **Description**.

   ![Nom et description du corps de la demande dans la console Listes de termes - Créer](images/try-terms-list-create-1.png)

5. Utilisez les espaces réservées aux paires clé-valeur pour assigner plus de métadonnées descriptives à votre liste.

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

   Ajoutez des métadonnées de liste en tant que paire clé-valeur, pas les termes.
 
6. Sélectionnez **Envoyer**. Votre liste est créée. Notez la valeur **ID** associée à la nouvelle liste. Vous en avez besoin pour d’autres fonctions de gestion de liste de termes.

   ![Console Listes de termes - Créer, la zone Contenu de la réponse affiche l’ID de la liste](images/try-terms-list-create-2.png)
 
7. Ajoutez des termes à MyList. Dans le menu de gauche, sous **Terme**, sélectionnez **Add Term** (Ajouter un terme). 

   La page **Term - Add Term** (Terme - Ajouter un terme) s’ouvre. 

8. Pour l’option **Open API testing console** (Ouvrir la console de test d’API), sélectionnez la région qui décrit le mieux votre emplacement. 

   ![Sélection de la région sur la page Term - Add Term (Terme - Ajouter un terme)](images/test-drive-region.png)

   La console d’API **Term - Add Term** (Terme - Ajouter un terme) s’ouvre.
 
9. Dans la zone **listId**, entrez l’ID de liste que vous avez créé et sélectionnez une valeur de **langue**. Entrez votre clé d’abonnement, puis sélectionnez **Envoyer**.

   ![Paramètres de requête de la console Term - Add Term (Terme - Ajouter un terme)](images/try-terms-list-create-3.png)
 
10. Pour vérifier que le terme a bien été ajouté à la liste, dans le menu gauche, sélectionnez **Terme** puis **Get All Terms** (Obtenir tous les termes). 

    La console d’API **Term - Get All Terms** (Terme - Obtenir tous les termes) s’ouvre.

11. Dans la zone **listId**, entrez l’ID de liste, puis votre clé d’abonnement. Sélectionnez **Envoyer**.

12. Dans la zone **Contenu de la réponse**, vérifiez les termes que vous avez entrés.

    ![Console Term - Get All Terms (Terme - Obtenir tous les termes), la zone Contenu de la réponse répertorie les termes que vous avez entrés](images/try-terms-list-create-4.png)
 
13. Ajoutez quelques termes de plus. Maintenant que vous avez créé une liste de termes personnalisée, essayez [d’analyser du texte](try-text-api.md) en utilisant la liste des termes personnalisée. 

## <a name="delete-terms-and-lists"></a>Supprimer des termes et des listes

La suppression d’un terme ou d’une liste est simple. Vous pouvez utiliser l’API pour effectuer les tâches suivantes :

- Supprimer un terme. (**Terme - Supprimer**)
- Supprimer tous les termes d’une liste sans supprimer la liste. (**Term - Delete All Terms** (Terme - Supprimer tous les termes))
- Supprimer une liste et tout son contenu. (**Listes de termes - Supprimer**)

Cet exemple supprime un seul terme.

1. Dans la [référence de l’API Gestion de liste de termes](https://westus.dev.cognitive.microsoft.com/docs/services/57cf755e3f9b070c105bd2c2/operations/57cf755e3f9b070868a1f67f), dans le menu gauche, sélectionnez **Terme**, puis **Supprimer**. 

   La page **Terme - Supprimer** s’ouvre.

2. Pour l’option **Open API testing console** (Ouvrir la console de test d’API), sélectionnez la région qui décrit le mieux votre emplacement. 

   ![Sélection d’une région sur la page Terme - Supprimer](images/test-drive-region.png)

   La console d’API **Terme - Supprimer** s’ouvre.
  
3. Dans la zone **listId**, entrez l’ID de la liste dans laquelle se trouve le terme à supprimer. Cet ID est le nombre (dans notre exemple, **122**) qui est renvoyé dans la console **Term Lists - Get Details** (Listes de termes - Obtenir les détails) pour MyList. Entrez le terme et sélectionnez une langue.
 
   ![Paramètres de requête de la console Terme - Supprimer](images/try-terms-list-delete-1.png)

4. Entrez votre clé d’abonnement, puis sélectionnez **Envoyer**.

5. Pour vérifier que le terme a été supprimé, utilisez la console **Term Lists - Get All** (Listes de termes - Tout obtenir).

   ![Console Term Lists - Get All (Listes de termes - Tout obtenir), la zone Contenu de la réponse indique que le terme est supprimé](images/try-terms-list-delete-2.png)
 
## <a name="change-list-information"></a>Modifier les informations d’une liste

Vous pouvez modifier le nom et la description d’une liste, et ajouter des éléments de métadonnées.

1. Dans la [référence de l’API Gestion de liste de termes](https://westus.dev.cognitive.microsoft.com/docs/services/57cf755e3f9b070c105bd2c2/operations/57cf755e3f9b070868a1f67f), dans le menu gauche, sélectionnez **Listes de termes**, puis **Mettre à jour les détails**. 

   La page **Listes de termes - Mettre à jour les détails** s’ouvre.

2. Pour l’option **Open API testing console** (Ouvrir la console de test d’API), sélectionnez la région qui décrit le mieux votre emplacement. 

   ![Sélection d’une région sur la page Listes de termes - Mettre à jour les détails](images/test-drive-region.png)

   La console d’API **Listes de termes - Mettre à jour les détails** s’ouvre.

3. Dans la zone **listId**, entrez l’ID de liste, puis votre clé d’abonnement.

4. Dans la zone **Corps de la demande**, faites vos changements, puis sélectionnez **Envoyer**.

   ![Modifications du corps de la demande dans la console Listes de termes - Mettre à jour les détails](images/try-terms-list-change-1.png)
 

## <a name="next-steps"></a>Étapes suivantes

Utilisez l’API REST dans votre code ou commencez par le [guide de démarrage rapide des listes de termes .NET](term-lists-quickstart-dotnet.md) pour en intégrer à votre application.
