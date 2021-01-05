---
title: Guide pratique pour ajouter des jeux de données de référence à un environnement – Azure Time Series Insights | Microsoft Docs
description: Cet article décrit comment ajouter un jeu de données de référence pour augmenter le volume de données dans votre environnement Azure Time Series Insights.
ms.service: time-series-insights
services: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.reviewer: jasonh, kfile
ms.workload: big-data
ms.topic: conceptual
ms.date: 09/30/2020
ms.custom: seodec18
ms.openlocfilehash: d80d97a609aa3a464b9b114439fe7f4058e287c3
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96001828"
---
# <a name="create-a-reference-data-set-for-your-azure-time-series-insights-gen1-environment-using-the-azure-portal"></a>Créer un jeu de données de référence pour votre environnement Azure Time Series Insights Gen1 à l’aide du portail Azure

> [!CAUTION]
> Il s’agit d’un article Gen1.

Cet article décrit comment ajouter un jeu de données de référence à votre environnement Azure Time Series Insights. Les données de référence sont utiles, joignez-les à votre source de données pour augmenter les valeurs.

Un jeu de données de référence est une collection d’éléments qui augmente les événements issus de votre source d’événements. Le moteur d’entrée Azure Time Series Insights associe chaque événement de votre source d’événements à la ligne de données correspondante dans votre jeu de données de référence. Cet événement ajouté est ensuite disponible pour la requête. Cette jointure repose sur les colonnes de clé privée définies dans votre jeu de données de référence.

Les données de référence ne sont pas jointes rétroactivement. Cela signifie que seules les données d’entrée actuelles et futures sont mises en correspondance et jointes à l’ensemble de données de référence, après configuration et téléchargement.

## <a name="video"></a>Vidéo

### <a name="learn-about-time-series-insights-reference-data-modelbr"></a>Découvrez le modèle de données de référence de Time Series Insight.</br>

> [!VIDEO <https://www.youtube.com/embed/Z0NuWQUMv1o>]

## <a name="add-a-reference-data-set"></a>Ajouter un jeu de données de référence

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Recherchez votre environnement Azure Time Series Insights existant. Sélectionnez **Toutes les ressources** dans le menu de gauche du portail Azure. Sélectionnez votre environnement Azure Time Series Insights.

1. Sélectionnez la page **Vue d’ensemble**. Développez la section **Éléments principaux** près du haut de la page pour localiser l’**URL de l’Explorateur Azure Time Series Insights** et ouvrir le lien.  

   [![Développez la section Essentials](media/add-reference-data-set/essentials.png)](media/add-reference-data-set/essentials.png#lightbox)

   Affichez l’Explorateur pour votre environnement Azure Time Series Insights.

1. Développez le sélecteur d’environnement dans l’Explorateur Azure Time Series Insights. Choisissez l’environnement actif. Sur la page de l’Explorateur, sélectionnez l’icône des données de référence en haut à droite.

   [![Ajouter des données de référence](media/add-reference-data-set/tsi-select-environment-and-data-icons.png)](media/add-reference-data-set/tsi-select-environment-and-data-icons.png#lightbox)

1. Cliquez sur le bouton **+ Ajouter un jeu de données** pour ajouter un nouveau jeu de données.

   [![Ajouter un jeu de données](media/add-reference-data-set/tsi-add-a-reference-data-set.png)](media/add-reference-data-set/tsi-add-a-reference-data-set.png#lightbox)

1. Sur la page **Nouveau jeu de données de référence**, choisissez le format des données :

   - Choisissez **CSV** pour des données délimitées par des virgules. La première ligne est traitée comme une ligne d’en-tête.
   - Choisissez **Tableau JSON** pour des données au format JSON.

   [![Choisissez le format des données.](media/add-reference-data-set/tsi-select-data-upload-option.png)](media/add-reference-data-set/tsi-select-data-upload-option.png#lightbox)

1. Fournissez les données via l’une des deux méthodes suivantes :

   - Collez les données dans l’éditeur de texte. Ensuite, cliquez sur le bouton **Analyser les données de référence**.
   - Cliquez sur le bouton **Choisir un fichier** pour ajouter des données à partir d’un fichier texte local.

   Par exemple, collez des données CSV : [![Données CSV collées](media/add-reference-data-set/select-csv-and-enter-data.png)](media/add-reference-data-set/select-csv-and-enter-data.png#lightbox)

   Par exemple, collez les données d’un tableau JSON : [![Coller des données JSON](media/add-reference-data-set/select-json-option-and-enter-data.png)](media/add-reference-data-set/select-json-option-and-enter-data.png#lightbox)

   En cas d’erreur lors de l’analyse des valeurs de données, l’erreur s’affiche en rouge au bas de la page : `CSV parsing error, no rows extracted`.

1. Une fois que les données ont été analysées correctement, une grille de données est affichée. Elle contient les colonnes et les lignes qui représentent les données. Passez en revue la grille de données pour garantir son exactitude.

   [![Passer en revue les données de référence](media/add-reference-data-set/review-displayed-data-grid.png)](media/add-reference-data-set/review-displayed-data-grid.png#lightbox)

1. Passez en revue chaque colonne pour comprendre le type de données pris par défaut et le modifier si nécessaire.  Sélectionnez le symbole de type de données dans l’en-tête de colonne : **#** pour double (données numériques), **T|F** pour booléen ou **Abc** pour une chaîne.

   [![Choisissez les types de données dans les en-têtes de colonne.](media/add-reference-data-set/select-column-types.png)](media/add-reference-data-set/select-column-types.png#lightbox)

1. Renommez les en-têtes de colonnes si nécessaire. Le nom de la colonne de clé est nécessaire pour joindre à la propriété correspondante dans votre source d’événements.

   > [!IMPORTANT]
   > Assurez-vous que les noms de colonne de clé de données de référence correspondent exactement au nom d’événement pour vos données entrantes, y compris la casse. Les noms des colonnes ne contenant pas de clés sont utilisés pour augmenter les données entrantes avec les valeurs de données de référence correspondantes.

1. Saisissez une valeur dans le champ **Filtrer les lignes...**  pour passer en revue des lignes spécifiques en fonction des besoins. Ce filtre est utile pour l’examen des données, mais il n’est pas appliqué lors du chargement des données.

1. Nommez le jeu de données en renseignant le champ **Nom du jeu de données** au-dessus de la grille de données.

    [![Nommez le jeu de données.](media/add-reference-data-set/enter-reference-data-set-name.png)](media/add-reference-data-set/enter-reference-data-set-name.png#lightbox)

1. Indiquez la colonne de **clé primaire** du jeu de données en sélectionnant la liste déroulante au-dessus de la grille de données.

    [![Sélectionnez la ou les colonnes clé.](media/add-reference-data-set/select-primary-key-column.png)](media/add-reference-data-set/select-primary-key-column.png#lightbox)

    **(Facultatif)** Sélectionnez le bouton **+** pour ajouter une colonne de clé secondaire, comme une clé primaire composite. Si vous devez annuler la sélection, choisissez la valeur vide de la liste déroulante pour supprimer la clé secondaire.

1. Pour télécharger les données, cliquez sur le bouton **Télécharger les lignes**.

    [![Chargez les lignes et confirmez les données.](media/add-reference-data-set/confirm-upload-reference-data.png)](media/add-reference-data-set/confirm-upload-reference-data.png#lightbox)

    La page confirme la fin du téléchargement et affiche le message **Jeu de données chargé**.

    > [!WARNING]
    > Les colonnes ou propriétés partagées entre les jeux de données de référence affichent une erreur de chargement **Nom de propriété dupliqué**. L’erreur n’empêche pas la réussite du chargement des jeux de données de référence. Elle peut être supprimé en combinant les lignes partageant le nom de propriété dupliqué.

1. Sélectionnez **Ajouter une ligne**, **Importer des lignes en bloc** ou **Ajouter une colonne** pour ajouter plusieurs valeurs de données de référence, si nécessaire.

    [![Ajouter une ligne, Importer des lignes en bloc ou Ajouter une colonne](media/add-reference-data-set/add-row-or-bulk-upload.png)](media/add-reference-data-set/add-row-or-bulk-upload.png#lightbox)

   > [!IMPORTANT]
   > Toute ligne qui partage une clé unique avec une autre ligne voit ses colonnes remplacées par la dernière ligne ajoutée qui partage cette clé unique.

   > [!NOTE]
   > Il **n’est pas** nécessaire que les lignes ajoutées soient  *rectangulaires* ; elles peuvent avoir moins de colonnes, plus de colonnes ou différentes colonnes par rapport aux autres entrées du jeu de données de référence.

## <a name="next-steps"></a>Étapes suivantes

- [Gérez les données de référence](time-series-insights-manage-reference-data-csharp.md) par programme.

- Pour obtenir des informations de référence d’API complètes, lisez le document [API de données de référence](/rest/api/time-series-insights/gen1-reference-data-api).