---
title: Se connecter une base de connaissances avec Power BI
titleSuffix: Azure Cognitive Search
description: Connectez une base de connaissances Recherche cognitive Azure à Power BI à des fins d’analyse et d’exploration.
author: HeidiSteen
ms.author: heidist
manager: nitinme
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/30/2020
ms.openlocfilehash: 91e75b60f5324288c9f1adac59e31b9c1a1b0e9e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89289169"
---
# <a name="connect-a-knowledge-store-with-power-bi"></a>Connecter une base de connaissances à Power BI

Cet article explique comment connecter et explorer une base de connaissances à l’aide de Power Query dans l’application Power BI Desktop. Vous pouvez commencer plus rapidement avec des modèles, ou créer un tableau de bord personnalisé à partir de rien. Cette courte vidéo ci-dessous montre comment vous pouvez enrichir votre expérience de vos données à l’aide de Recherche cognitive Azure en association avec Power BI.


> [!VIDEO https://www.youtube.com/embed/XWzLBP8iWqg?version=3&start=593&end=663]



+ Suivez les étapes décrites dans [Créer une base de connaissances dans le portail Azure](knowledge-store-create-portal.md) ou [Créer une base de connaissances Recherche cognitive Azure en utilisant REST](knowledge-store-create-rest.md) pour créer l’exemple de base de connaissances utilisé dans cette procédure pas à pas. Vous aurez également besoin du nom du compte de stockage Azure utilisé pour créer la base de connaissances ainsi que de sa clé d’accès dans le portail Azure.

+ [Installer Power BI Desktop](https://powerbi.microsoft.com/downloads/)

## <a name="sample-power-bi-template---azure-portal-only"></a>Exemple de modèle Power BI - Portail Azure uniquement

Lorsque vous créez une [base de connaissances à l’aide du portail Azure](knowledge-store-create-portal.md), vous avez la possibilité de télécharger un [modèle Power BI](https://github.com/Azure-Samples/cognitive-search-templates) dans la deuxième page de l’Assistant **Importer des données**. Ce modèle propose plusieurs visualisations, comme WordCloud et Network Navigator, pour le contenu textuel. 

Cliquez sur **Obtenir le modèle Power BI** dans la page **Ajouter des compétences cognitives** pour récupérer et télécharger le modèle à partir de son emplacement GitHub public. L’Assistant modifie le modèle pour qu’il s’adapte à la forme de vos données, telles qu’elles sont capturées dans les projections de la base de connaissances spécifiées dans l’Assistant. C’est la raison pour laquelle le modèle que vous téléchargez varie chaque fois que vous exécutez l’Assistant, conformément aux entrées de données et sélections de compétences différentes.

![Exemple de modèle Recherche cognitive Azure Power BI](media/knowledge-store-connect-power-bi/powerbi-sample-template-portal-only.png "Exemple de modèle Power BI")

> [!NOTE]
> Bien que le modèle soit téléchargé à mi-parcours de l’Assistant, vous devez attendre que la base de connaissances soit réellement créée dans le stockage Table Azure avant de pouvoir l’utiliser.

## <a name="connect-with-power-bi"></a>Se connecter avec Power BI

1. Démarrez Power BI Desktop, puis cliquez sur **Obtenir les données**.

1. Dans la fenêtre **Obtenir les données**, sélectionnez **Azure**, puis **Stockage Table Azure**.

1. Cliquez sur **Connecter**.

1. Pour **Nom du compte ou URL**, entrez le nom de votre compte Stockage Azure (l’URL complète est créée automatiquement).

1. Si vous y êtes invité, entrez la clé du compte de stockage.

1. Sélectionnez les tables contenant les données d’évaluation des hôtels créées au long des procédures pas à pas précédentes. 

   + Pour la procédure pas à pas du portail, les noms de table sont *hotelReviewsSsDocument*, *hotelReviewsSsEntities*, *hotelReviewsSsKeyPhrases* et *hotelReviewsSsPages*. 
   
   + Pour la procédure pas à pas REST, les noms de table sont *hotelReviewsDocument*, *hotelReviewsPages*, *hotelReviewsKeyPhrases* et *hotelReviewsSentiment*.

1. Cliquez sur **Charger**.

1. Dans le ruban supérieur, cliquez sur **Modifier les requêtes** pour ouvrir l’**Éditeur Power Query**.

   ![Ouvrir Power Query](media/knowledge-store-connect-power-bi/powerbi-edit-queries.png "Ouvrir Power Query")

1. Sélectionnez *hotelReviewsSsDocument*, puis supprimez les colonnes *PartitionKey*, *RowKey* et *Timestamp*. 
   ![Modifier les tables](media/knowledge-store-connect-power-bi/powerbi-edit-table.png "Modifier les tables")

1. En haut à droite de la table, cliquez sur l’icône montrant des flèches opposées pour développer le *Contenu*. Une fois que la liste des colonnes s’affiche, sélectionnez toutes les colonnes, puis désélectionnez les colonnes commençant par « métadonnées ». Cliquez sur **OK** pour afficher les colonnes sélectionnées.

   ![Développer le contenu](media/knowledge-store-connect-power-bi/powerbi-expand-content-table.png "Développer le contenu")

1. Changez le type de données des colonnes suivantes en cliquant en haut à gauche de la colonne sur l’icône ABC-123.

   + Pour *Content.latitude* et *Content.longitude*, sélectionnez **Nombre décimal**.
   + Pour *Content.reviews_date* et *Content.reviews_dateAdded*, sélectionnez **Date/Heure**.

   ![Changer les types de données](media/knowledge-store-connect-power-bi/powerbi-change-type.png "Changer les types de données")

1. Sélectionnez *hotelReviewsSsPages*, puis répétez les étapes 9 et 10 pour supprimer les colonnes et développer le *Contenu*.
1. Remplacez le type de données de *Content.SentimentScore* par **Nombre décimal**.
1. Sélectionnez *hotelReviewsSsKeyPhrases*, puis répétez les étapes 9 et 10 pour supprimer les colonnes et développer le *Contenu*. Il n’y a aucune modification de type de données pour cette table.

1. Dans la barre de commandes, cliquez sur **Fermer et appliquer**.

1. Cliquez sur la vignette Modèle dans le volet de navigation de gauche, puis vérifiez que Power BI montre les relations entre les trois tables.

   ![Valider les relations](media/knowledge-store-connect-power-bi/powerbi-relationships.png "Valider les relations")

1. Double-cliquez sur chaque relation, puis vérifiez que l’option **Direction du filtre croisé** a la valeur **Les deux**.  Cela permet d’actualiser les visuels quand un filtre est appliqué.

1. Cliquez sur la vignette Rapport dans le volet de navigation gauche pour explorer les données dans des visualisations. Pour les champs de texte, les tables et les cartes s’avèrent être des visualisations utiles. Vous pouvez choisir des champs dans chacune des trois tables pour renseigner la table ou la carte. 

<!-- ## Try with larger data sets

We purposely kept the data set small to avoid charges for a demo walkthrough. For a more realistic experience, you can create and then attach a billable Cognitive Services resource to enable a larger number of transactions against the sentiment analyzer, keyphrase extraction, and language detector skills.

Create new containers in Azure Blob storage and upload each CSV file to its own container. Specify one of these containers in the data source creation step in Import data wizard.

| Description | Link |
|-------------|------|
| Free tier   | [HotelReviews_Free.csv](https://knowledgestoredemo.blob.core.windows.net/hotel-reviews/HotelReviews_Free.csv?st=2019-07-29T17%3A51%3A30Z&se=2021-07-30T17%3A51%3A00Z&sp=rl&sv=2018-03-28&sr=c&sig=LnWLXqFkPNeuuMgnohiz3jfW4ijePeT5m2SiQDdwDaQ%3D) |
| Small (500 Records) | [HotelReviews_Small.csv](https://knowledgestoredemo.blob.core.windows.net/hotel-reviews/HotelReviews_Small.csv?st=2019-07-29T17%3A51%3A30Z&se=2021-07-30T17%3A51%3A00Z&sp=rl&sv=2018-03-28&sr=c&sig=LnWLXqFkPNeuuMgnohiz3jfW4ijePeT5m2SiQDdwDaQ%3D) |
| Medium (6000 Records)| [HotelReviews_Medium.csv](https://knowledgestoredemo.blob.core.windows.net/hotel-reviews/HotelReviews_Medium.csv?st=2019-07-29T17%3A51%3A30Z&se=2021-07-30T17%3A51%3A00Z&sp=rl&sv=2018-03-28&sr=c&sig=LnWLXqFkPNeuuMgnohiz3jfW4ijePeT5m2SiQDdwDaQ%3D)
| Large (Full dataset 35000 Records) | [HotelReviews_Large.csv](https://knowledgestoredemo.blob.core.windows.net/hotel-reviews/HotelReviews_Large.csv?st=2019-07-29T17%3A51%3A30Z&se=2021-07-30T17%3A51%3A00Z&sp=rl&sv=2018-03-28&sr=c&sig=LnWLXqFkPNeuuMgnohiz3jfW4ijePeT5m2SiQDdwDaQ%3D). Be aware that very large data sets are expensive to process. This one costs roughly $1000 U.S dollars.|

In the enrichment step of the wizard, attach a billable [Cognitive Services](../cognitive-services/cognitive-services-apis-create-account.md) resource, created at the *S0* tier, in the same region as Azure Cognitive Search to use larger data sets. 

  ![Create a Cognitive Services resource](media/knowledge-store-connect-power-bi/create-cognitive-service.png "Create a Cognitive Services resource") -->

## <a name="clean-up"></a>Nettoyer

Lorsque vous travaillez dans votre propre abonnement, il est recommandé, à la fin de chaque projet, de déterminer si vous avez toujours besoin des ressources que vous avez créées. Les ressources laissées en cours d’exécution peuvent vous coûter de l’argent. Vous pouvez supprimer les ressources une par une, ou choisir de supprimer le groupe de ressources afin de supprimer l’ensemble des ressources.

Vous pouvez rechercher et gérer les ressources dans le portail à l’aide des liens **Toutes les ressources** ou **Groupes de ressources** situés dans le volet de navigation de gauche.

Si vous utilisez un service gratuit, n’oubliez pas que vous êtes limité à trois index, indexeurs et sources de données. Vous pouvez supprimer des éléments un par un dans le portail pour ne pas dépasser la limite.

## <a name="next-steps"></a>Étapes suivantes

Pour apprendre à explorer cette base de connaissances à l’aide de l’Explorateur Stockage, consultez l’article suivant.

> [!div class="nextstepaction"]
> [Voir avec l’Explorateur Stockage](knowledge-store-view-storage-explorer.md)