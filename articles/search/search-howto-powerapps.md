---
title: 'Tutoriel : Interroger des données à partir de Power Apps'
titleSuffix: Azure Cognitive Search
description: Instructions pas à pas pour créer une application Power Apps qui se connecte à un index Recherche cognitive Azure, envoie des requêtes et affiche les résultats.
author: luiscabrer
manager: eladz
ms.author: luisca
ms.service: cognitive-search
ms.devlang: rest-api
ms.topic: tutorial
ms.date: 11/17/2020
ms.openlocfilehash: e8c16f02cf6b77fa54d2a19abac48e9914aa99bd
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96008563"
---
# <a name="tutorial-query-a-cognitive-search-index-from-power-apps"></a>Tutoriel : Interroger un index Recherche cognitive à partir de Power Apps

Tirez parti de l’environnement de développement rapide d’applications Power Apps pour créer une application personnalisée permettant de rechercher votre contenu dans Recherche cognitive Azure.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Se connecter à la Recherche cognitive Azure
> * Configurer une demande de requête
> * Visualiser les résultats dans une application canevas

Si vous n’avez pas d’abonnement Azure, ouvrez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

* [Compte Power Apps](https://make.powerapps.com)

* [Index hotels-sample](search-get-started-portal.md) hébergé dans votre service de recherche

* [Clé d’API de requête](search-security-api-keys.md#find-existing-keys)

## <a name="1---create-a-custom-connector"></a>1 – Créer un connecteur personnalisé

Dans Power Apps, un connecteur est une connexion de source de données. Dans cette étape, vous allez créer un connecteur personnalisé pour vous connecter à un index de recherche dans le cloud.

1. [Connectez-vous](https://make.powerapps.com) à Power Apps.

1. À gauche, développez **Données** > **Connecteurs personnalisés**.
 
    :::image type="content" source="./media/search-howto-powerapps/1-2-custom-connector.png" alt-text="Menu du connecteur personnalisé" border="true":::

1. Sélectionnez **+ Nouveau connecteur personnalisé**, puis **Créer entièrement**.

    :::image type="content" source="./media/search-howto-powerapps/1-3-create-blank.png" alt-text="Créer entièrement" border="true":::

1. Donnez un nom à votre connecteur personnalisé (par exemple, *AzureSearchQuery*), puis cliquez sur **Continuer**.

1. Entrez des informations dans la page Général :

   * Couleur d’arrière-plan de l’icône (par exemple, #007ee5)
   * Description (par exemple, « Connecteur vers Recherche cognitive Azure »)
   * Dans l’hôte, vous devez entrer l’URL de votre service de recherche (par exemple, `<yourservicename>.search.windows.net`)
   * Pour l’URL de base, entrez simplement « / »

    :::image type="content" source="./media/search-howto-powerapps/1-5-general-info.png" alt-text="Boîte de dialogue Informations générales" border="true":::

1. Dans la page Sécurité, définissez *Clé de l’API* comme **Type d’authentification**, puis définissez l’étiquette et le nom du paramètre sur *api-key*. Pour **Emplacement du paramètre**, sélectionnez *En-tête* comme indiqué ci-dessous.

    :::image type="content" source="./media/search-howto-powerapps/1-6-authentication-type.png" alt-text="Option Type d’authentification" border="true":::

1. Dans la page Définitions, sélectionnez **+ Nouvelle action** pour créer une action qui interroge l’index. Entrez la valeur « Requête » pour le résumé et le nom de l’ID d’opération. Entrez une description comme *« Interroge l’index de recherche »* .

    :::image type="content" source="./media/search-howto-powerapps/1-7-new-action.png" alt-text="Options Nouvelle action" border="true":::

1. Faites défiler vers le bas. Dans Demandes, cliquez sur le bouton **+ Importer à partir de l’exemple** pour configurer une demande de requête à destination de votre service de recherche :

   * Sélectionnez le verbe `GET`

   * Pour l’URL, entrez un exemple de requête pour votre index de recherche (`search=*` retourne tous les documents, `$select=` vous permet de choisir les champs). La version de l’API est obligatoire. Voici à quoi ressemble une URL entièrement spécifiée : `https://mydemo.search.windows.net/indexes/hotels-sample-index/docs?search=*&$select=HotelName,Description,Address/City&api-version=2020-06-30`

   * Pour En-têtes, tapez `Content-Type`. Vous définirez la valeur sur `application/json` dans une prochaine étape.

     **Power Apps** utilise la syntaxe de l’URL pour extraire les paramètres de la requête : les paramètres search, select et api-version deviennent configurables à mesure que vous progressez dans l’Assistant.

       :::image type="content" source="./media/search-howto-powerapps/1-8-1-import-from-sample.png" alt-text="Importer à partir de l’exemple" border="true":::

1. Cliquez sur **Importer** pour compléter automatiquement la demande. Terminez la définition des métadonnées de paramètre en cliquant sur le symbole **...** en regard de chacun des paramètres. Cliquez sur **Retour** pour retourner à la page Demande après chaque mise à jour de paramètre.

   :::image type="content" source="./media/search-howto-powerapps/1-8-2-import-from-sample.png" alt-text="Boîte de dialogue Importer à partir de l’exemple" border="true":::

1. Pour *search* : définissez `*` comme **valeur par défaut**, définissez **obligatoire** sur *Faux* et définissez la **visibilité** sur *aucune*. 

    :::image type="content" source="./media/search-howto-powerapps/1-10-1-parameter-metadata-search.png" alt-text="Métadonnées du paramètre Search" border="true":::

1. Pour *select* : définissez `HotelName,Description,Address/City` comme **valeur par défaut**, définissez **obligatoire** sur *Faux* et définissez la **visibilité** sur *aucune*.  

    :::image type="content" source="./media/search-howto-powerapps/1-10-4-parameter-metadata-select.png" alt-text="Sélectionner les métadonnées de paramètre" border="true":::

1. Pour *api-version* : définissez `2020-06-30` comme **valeur par défaut**, définissez **obligatoire** sur *Vrai* et définissez la **visibilité** sur *interne*.  

    :::image type="content" source="./media/search-howto-powerapps/1-10-2-parameter-metadata-version.png" alt-text="Métadonnées du paramètre Version" border="true":::

1. Pour *Content-Type* : Défini sur `application/json`.

1. Une fois ces modifications effectuées, basculez sur la vue **Éditeur Swagger**. Dans la section Paramètres, vous devez voir la configuration suivante :

    ```JSON
    parameters:
      - {name: search, in: query, required: false, type: string, default: '*'}
      - {name: $select, in: query, required: false, type: string, default: 'HotelName,Description,Address/City'}
      - {name: api-version, in: query, required: true, type: string, default: '2020-06-30',
        x-ms-visibility: internal}
      - {name: Content-Type, in: header, required: false, type: string}
    ```

1. Revenez à l’Assistant et retournez à l’étape **3. Demande**. Faites défiler vers le bas jusqu’à la section Réponse. Cliquez sur **« Ajouter une réponse par défaut »** . Cela est primordial dans le sens où cela permet à Power Apps de comprendre le schéma de la réponse. 

1. Collez un exemple de réponse. Une façon simple de capturer un exemple de réponse est de passer par l’Explorateur de recherche sur le portail Azure. Dans l’Explorateur de recherche, vous devez entrer la même requête que pour la demande, mais en ajoutant **$top=2** pour limiter les résultats à seulement deux documents : `search=*&$select=HotelName,Description,Address/City&$top=2`. 

   Il suffit de quelques résultats à Power Apps pour détecter le schéma. Vous pouvez maintenant copier la réponse suivante dans l’Assistant, en supposant que utilisez hotels-sample-index.

    ```JSON
    {
        "@odata.context": "https://mydemo.search.windows.net/indexes('hotels-sample-index')/$metadata#docs(*)",
        "value": [
            {
                "@search.score": 1,
                "HotelName": "Arcadia Resort & Restaurant",
                "Description": "The largest year-round resort in the area offering more of everything for your vacation – at the best value!  What can you enjoy while at the resort, aside from the mile-long sandy beaches of the lake? Check out our activities sure to excite both young and young-at-heart guests. We have it all, including being named “Property of the Year” and a “Top Ten Resort” by top publications.",
                "Address": {
                    "City": "Seattle"
                }
            },
            {
                "@search.score": 1,
                "HotelName": "Travel Resort",
                "Description": "The Best Gaming Resort in the area.  With elegant rooms & suites, pool, cabanas, spa, brewery & world-class gaming.  This is the best place to play, stay & dine.",
                "Address": {
                    "City": "Albuquerque"
                }
            }
        ]
    }
    ```

    > [!TIP] 
    > Sachant que vous êtes soumis à une limite de caractères dans la réponse JSON que vous pouvez entrer, vous pouvez simplifier le JSON avant de le coller. Le schéma et le format de la réponse sont plus importants que les valeurs elles-mêmes. Par exemple, le champ Description peut être simplifié à la première phrase.

1. Cliquez sur **Importer** pour ajouter la réponse par défaut.

1. Cliquez sur **Créer un connecteur** en haut à droite pour enregistrer la définition.

1. Cliquez sur **Fermer** pour fermer le connecteur.

## <a name="2---test-the-connection"></a>2 – Tester la connexion

Après avoir créé le connecteur, vous devez le rouvrir à partir de la liste Connecteurs personnalisés de façon à le tester. Si, par la suite, vous effectuez des mises à jour supplémentaires, vous pourrez le tester depuis l’Assistant.

Vous aurez besoin d’une [clé d’API de requête](search-security-api-keys.md#find-existing-keys) pour cette tâche. Chaque fois qu’une connexion est créée, que ce soit pour une série de tests ou pour une inclusion dans une application, le connecteur a besoin de la clé d’API de requête pour se connecter à Recherche cognitive Azure.

1. Tout à gauche, cliquez sur **Connecteurs personnalisés**.

1. Trouvez votre connecteur dans la liste (dans ce tutoriel, il s’agit de « AzureSearchQuery »).

1. Sélectionnez le connecteur, développez la liste d’actions, puis sélectionnez **Afficher les propriétés**.

    :::image type="content" source="./media/search-howto-powerapps/1-11-1-test-connector.png" alt-text="Afficher les propriétés" border="true":::

1. Sélectionnez **Modifier** en haut à droite.

1. Sélectionnez **4. Tester** pour ouvrir la page de test.

1. Dans Opération de test, cliquez sur **+ Nouvelle connexion**.

1. Entrez une clé d’API de requête. Il s’agit d’une requête Recherche cognitive Azure pour un accès en lecture seule à un index. Vous pouvez [trouver la clé](search-security-api-keys.md#find-existing-keys) sur le portail Azure. 

1. Dans Opérations, cliquez sur le bouton **Opération de test**. Si l’opération aboutit, vous obtenez un état 200 et un JSON apparaît dans le corps de la réponse avec une description des résultats de la recherche.

    :::image type="content" source="./media/search-howto-powerapps/1-11-2-test-connector.png" alt-text="Réponse JSON" border="true":::

## <a name="3---visualize-results"></a>3 – Visualiser les résultats

Dans cette étape, vous allez créer une application Power Apps avec une zone de recherche, un bouton de recherche et une zone d’affichage des résultats. L’application Power Apps se connecte au connecteur personnalisé récemment créé pour obtenir les données à partir de Recherche Azure.

1. À gauche, développez **Apps** >  **+ Nouvelle application** > **Canevas**.

    :::image type="content" source="./media/search-howto-powerapps/2-1-create-canvas.png" alt-text="Créer une application canevas" border="true":::

1. Sélectionnez le type d’application. Pour ce tutoriel, créez une **application vide** avec le **Mode téléphone**. **Power Apps Studio** s’affiche.

1. Une fois dans Studio, sélectionnez l’onglet **Sources de données**, puis cliquez sur le nouveau connecteur que vous venez de créer. Dans notre cas, il s’appelle *AzureSearchQuery*. Cliquez sur **Ajouter une connexion**.

   Entrez la clé d’API de requête.

    :::image type="content" source="./media/search-howto-powerapps/2-3-connect-connector.png" alt-text="Connecter le connecteur" border="true":::

    À présent, *AzureSearchQuery* est une source de données qui peut être utilisée à partir de votre application.

1. Sous l’**onglet Insertion**, ajoutez quelques contrôles au canevas.

    :::image type="content" source="./media/search-howto-powerapps/2-4-add-controls.png" alt-text="Insérer des contrôles" border="true":::

1. Insérez les éléments suivants :

   * Une étiquette de texte avec la valeur « Requête : »
   * Un élément d’entrée de texte (appelez-le *txtQuery*, valeur par défaut : «*»)
   * Un bouton avec le texte « Rechercher » 
   * Une galerie verticale appelée (appelez-la *galleryResults*)

    Voici comment se présente le canevas :

    :::image type="content" source="./media/search-howto-powerapps/2-5-controls-layout.png" alt-text="Disposition des contrôles" border="true":::

1. Pour que le **bouton Rechercher** émette une requête, collez l’action suivante dans **OnSelect** :

    ```
    If(!IsBlank(txtQuery.Text),
        ClearCollect(azResult, AzureSearchQuery.Query({search: txtQuery.Text}).value))
    ```

   La capture d’écran suivante affiche la barre de formule pour l’action **OnSelect**.

    :::image type="content" source="./media/search-howto-powerapps/2-6-search-button-event.png" alt-text="Bouton OnSelect" border="true":::

   Cette action fera que le bouton mettra à jour une nouvelle collection appelée *azResult* avec le résultat de la requête de recherche en utilisant le texte de la zone de texte *txtQuery* comme terme de la requête.

   > [!NOTE]
   > Si vous obtenez une erreur de syntaxe de formule de type « La fonction 'ClearCollect' contient des fonctions non valides », essayez ceci :
   > 
   > * Tout d’abord, vérifiez que la référence du connecteur est correcte. Effacez le nom du connecteur et commencez à taper le nom de votre connecteur. Intellisense doit alors vous suggérer le connecteur et le verbe appropriés.
   > 
   > * Si l’erreur persiste, supprimez et recréez le connecteur. S’il existe plusieurs instances d’un connecteur, il se peut que l’application n’utilise pas le bon.
   > 

1. Liez le contrôle Galerie verticale à la collection *azResult* que vous avez créée à l’étape précédente. 

   Sélectionnez le contrôle Galerie, puis effectuez les actions suivantes dans le volet Propriétés.

   * Définissez **Source de données** sur *azResult*.
   * Sélectionnez une **Disposition** qui vous convienne en fonction du type de données de votre index. Dans ce cas, nous avons utilisé la disposition *Title, subtitle and body* (Titre, sous-titre et corps).
   * **Modifiez les champs**, puis sélectionnez les champs que vous souhaitez visualiser.

    Étant donné que nous avons fourni un exemple de résultat lorsque nous avons défini le connecteur, l’application connaît les champs disponibles dans votre index.
    
    :::image type="content" source="./media/search-howto-powerapps/2-7-gallery-select-fields.png" alt-text="Champs de la galerie" border="true":::   
 
1. Appuyez sur la touche **F5** pour afficher un aperçu de l’application.  

    :::image type="content" source="./media/search-howto-powerapps/2-8-3-final.png" alt-text="Application finale" border="true":::    

<!--     Remember that the fields can be set to calculated values.

    For the example, setting using the *"Image, Title and Subtitle"* layout and specifying the *Image* function as the concatenation of the root path for the data and the file name (for instance, `"https://mystore.blob.core.windows.net/multilang/" & ThisItem.metadata_storage_name`) will produce the result below.

    :::image type="content" source="./media/search-howto-powerapps/2-8-2-final.png" alt-text="Final app" border="true":::         -->

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous travaillez dans votre propre abonnement, il est recommandé, à la fin de chaque projet, de déterminer si vous avez toujours besoin des ressources que vous avez créées. Les ressources laissées en cours d’exécution peuvent vous coûter de l’argent. Vous pouvez supprimer les ressources une par une, ou choisir de supprimer le groupe de ressources afin de supprimer l’ensemble des ressources.

Vous pouvez rechercher et gérer les ressources dans le portail à l’aide des liens **Toutes les ressources** ou **Groupes de ressources** situés dans le volet de navigation de gauche.

Si vous utilisez un service gratuit, n’oubliez pas que vous êtes limité à trois index, indexeurs et sources de données. Vous pouvez supprimer des éléments un par un dans le portail pour ne pas dépasser la limite.

## <a name="next-steps"></a>Étapes suivantes

Power Apps permet de développer rapidement des applications personnalisées. Maintenant que vous savez comment vous connecter à un index de recherche, apprenez-en davantage sur la création d’une expérience de visualisation enrichie dans une application Power Apps personnalisée.

> [!div class="nextstepaction"]
> [Catalogue de formations Power Apps](/powerapps/learning-catalog/get-started)