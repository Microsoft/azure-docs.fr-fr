---
title: 'Procédure : rechercher dans Data Catalog'
description: Cet article offre une vue d’ensemble de la recherche dans un catalogue de données.
author: chanuengg
ms.author: csugunan
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: conceptual
ms.date: 11/15/2020
ms.openlocfilehash: 87ca3c49d6a96f083cc50f01f5b9a4a739d688a1
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96550171"
---
# <a name="search-the-azure-purview-data-catalog"></a>Effectuer une recherche dans Data Catalog Azure Purview

Cet article explique comment utiliser les différentes fonctionnalités de recherche dans Data Catalog Azure Purview.

## <a name="search-the-catalog-for-assets"></a>Rechercher des ressources dans le catalogue

Les étapes pour effectuer une recherche de ressources sont les suivantes :

1. [Ouvrez la boîte de dialogue de recherche des ressources](#open-the-asset-search-dialog) en sélectionnant **Catalogue de recherche**.
1. [Entrez les termes de recherche](#enter-search-terms) pour rechercher les ressources dont les caractéristiques correspondent aux termes.
1. [Définissez les filtres rapides](#set-quick-filters) pour affiner la recherche.
1. [Démarrez la recherche](#start-the-search) et accédez aux résultats de la recherche.

Peu importe si vous définissez des filtres rapides avant ou après avoir entré des termes de recherche.

S’il n’existe aucun terme de recherche et aucun filtre, les résultats de la recherche incluent toutes les ressources.

### <a name="open-the-asset-search-dialog"></a>Ouvrir la boîte de dialogue de recherche de ressources

Ouvrez la boîte de dialogue de recherche des ressources en sélectionnant **Catalogue de recherche**.

:::image type="content" source="./media/how-to-search-catalog/search-catalog.png" alt-text="Sous « Rechercher dans le catalogue », il existe un volet gauche avec des filtres de recherche et un volet droit avec des recherches récentes." border="true":::

La boîte de dialogue Rechercher affiche les filtres rapides, l’historique de recherche et la liste des ressources récemment consultées.

:::image type="content" source="./media/how-to-search-catalog/asset-search-dialog.png" alt-text="La liste de recherche se trouve dans le volet droit, sous « Rechercher dans le catalogue »." border="true":::

### <a name="enter-search-terms"></a>Entrer les termes de la recherche

Entrez un ou plusieurs termes de la recherche dans **Rechercher dans le catalogue**. À mesure que vous tapez, les termes de recherche correspondants provenant de recherches récentes apparaissent dans **Vos recherches récentes**, les termes de recherche correspondants suggérés apparaissent dans **Suggestions de recherche**, et les ressources de données correspondantes apparaissent dans **Suggestions de ressources**.

:::image type="content" source="./media/how-to-search-catalog/enter-search-terms.png" alt-text="Capture d’écran montrant les résultats d’une recherche entrée dans la zone Catalogue de recherche":::

Les résultats de la recherche incluent uniquement les ressources avec une ou plusieurs caractéristiques qui correspondent aux termes de la recherche. Ces caractéristiques incluent le nom de la ressource, le type de ressource, les classifications et les contacts.

#### <a name="types-of-search-criteria"></a>Types de critères de recherche

Azure portée prend en charge les types de critères de recherche suivants.

> [!Note]
> Spécifiez toujours les opérateurs booléens (**AND**, **OR**, **NOT**) tout en majuscules. Dans le cas contraire, la casse n’a pas d’importance, ni d’espace supplémentaire.

- **hive** : Rechercher des documents contenant **hive**.
- **hive database** : Rechercher des documents qui contiennent exactement **hive database**.
- **hive OR database** : Rechercher des documents qui contiennent **hive** ou **database**, ou les deux.
- **hive AND database**, **hive && database** : Rechercher des documents qui contiennent à la fois **hive** et **database**.
- **hive AND (database OR warehouse)**  : Rechercher des documents qui contiennent **hive** et **database** ou **warehouse**, ou les deux.
- **hive NOT database** : Rechercher des documents qui contiennent **hive**, mais pas **database**.
- **hiv** : Rechercher des documents qui contiennent un mot commençant par **hiv**. Par exemple, **hiv**, **hive**, **hivbar** (* est un caractère générique qui correspond à n’importe quel nombre de caractères).

### <a name="set-quick-filters"></a>Définir des filtres rapides

La liste des résultats de la recherche est basée sur les termes de recherche que vous entrez dans **Rechercher dans le catalogue** et sur les valeurs que vous sélectionnez pour les filtres rapides.

Un filtre rapide limite la liste des résultats de la recherche aux ressources qui ont une valeur sélectionnée d’une caractéristique. Le filtre a une liste déroulante et une zone de texte. La liste déroulante affiche les valeurs de la caractéristique qui se trouvent dans les résultats *actuels* de la recherche. En regard de chaque valeur de la liste figure un comptage du nombre de ressources dans les résultats actuels de la recherche qui ont cette valeur. Si vous sélectionnez une valeur dans la liste, les résultats de la recherche sont limités aux ressources qui ont cette valeur. Vous ne pouvez sélectionner qu’une seule valeur.

Les résultats actuels de la recherche utilisés dans la préparation de la liste déroulante sont déterminés par les éléments suivants :

- Les termes de la recherche entrés dans **Rechercher dans le catalogue**. 
- Les valeurs sélectionnées dans les filtres rapides.

Voici un exemple de filtre rapide « Type de ressource ».

:::image type="content" source="./media/how-to-search-catalog/asset-type-quick-filter.png" alt-text="Exemple de filtre rapide de type de ressource." border="true":::

Vous pouvez entrer du texte dans la zone de texte pour limiter les valeurs de la liste déroulante aux valeurs qui correspondent partiellement ou intégralement au texte. Pour obtenir des exemples d’utilisation de la zone de texte, consultez [Filtre rapide de recherche : filtrer par type de ressource](#search-quick-filter-filter-by-asset-type) et [Filtre rapide de recherche : filtrer par classification](#search-quick-filter-filter-by-classification).

#### <a name="search-quick-filter-filter-by-asset-type"></a>Filtre rapide de recherche : filtrer par type de ressource

Pour filtrer par type de ressource, utilisez le filtre rapide **Type de ressource**. La liste déroulante répertorie les types de ressources trouvés dans les résultats de la recherche actuelle, tels que déterminés par les termes de recherche et les filtres rapides. Pour chaque type, le nombre de ressources de ce type est affiché.

:::image type="content" source="./media/how-to-search-catalog/asset-type-quick-filter.png" alt-text="Les filtres rapides de type de ressource sont mis en surbrillance. Les types de ressources et la quantité de chacun sont affichés." border="true":::

Sélectionnez un type de ressource pour limiter les résultats de la recherche aux ressources de ce type. Vous ne pouvez sélectionner qu’un seul type.

Pour afficher uniquement les types de ressources dont les noms correspondent à une chaîne, entrez la chaîne dans la zone de texte. Par exemple, pour afficher uniquement les types de ressources dont le nom contient **SQL**, entrez **sql**.

:::image type="content" source="./media/how-to-search-catalog/filter-asset-types.png" alt-text="Le volet Filtres rapides contient sql pour « Type de ressource ». La liste des ressources qui contiennent sql affiche trois entrées." border="true":::

Sélectionnez un type de ressource pour limiter les résultats de la recherche aux ressources de ce type. Vous ne pouvez sélectionner qu’un seul type.

#### <a name="search-quick-filter-filter-by-classification"></a>Filtre rapide de recherche : filtrer par classification

Pour filtrer par classification des ressources, utilisez le filtre rapide **Classification**. La liste déroulante affiche les classifications qui ont été affectées à une ou plusieurs ressources dans les résultats de la recherche actuelle, comme déterminé par les termes de recherche et les filtres rapides. Pour chaque classification, le nombre de ressources affectées à cette classification s’affiche.

:::image type="content" source="./media/how-to-search-catalog/classification-quick-filter.png" alt-text="Les filtres rapides de classification sont mis en surbrillance." border="true":::

Sélectionnez une classification pour limiter les résultats de la recherche aux ressources affectées à cette classification. Vous ne pouvez sélectionner qu’une seule classification.

Pour afficher uniquement les classifications dont les noms correspondent à une chaîne, entrez la chaîne dans la zone de texte. Par exemple, pour afficher uniquement les classifications dont le nom contient **numéro**, entrez **numéro**.

:::image type="content" source="./media/how-to-search-catalog/filter-classifications.png" alt-text="Dans le volet Filtres rapides, la classification correspond à « Banque », et les classifications affichées contiennent toutes cette valeur." border="true":::

Sélectionnez une classification pour limiter les résultats de la recherche aux ressources qui ont été affectées à cette classification. Vous ne pouvez pas sélectionner plusieurs classifications.

#### <a name="search-quick-filter-filter-by-contacts"></a>Filtre rapide de recherche : filtrer par contact

Un *contact* est une personne qui est affectée à une ressource en tant que propriétaire ou expert. Lorsque vous affichez les détails de la ressource, les contacts s’affichent sous l’onglet **Contacts**.

Il existe deux façons de rechercher des ressources auxquelles un contact particulier est affecté.

- Entrez tout ou partie du nom du contact dans **Catalogue de recherche** et effectuez une recherche. Les résultats de la recherche incluent les ressources dont les noms correspondent à vos termes de recherche.
- Sélectionnez le contact qui vous intéresse dans le filtre rapide **Contact** et effectuez une recherche.

:::image type="content" source="./media/how-to-search-catalog/contact-quick-filter.png" alt-text="La valeur de Personne dans le volet Filtres rapides est « darren ». Le volet Suggestions comporte trois suggestions." border="true":::

## <a name="search-example"></a>Exemple de recherche

Prenons un exemple hypothétique pour voir comment les termes de la recherche et les filtres rapides interagissent pour déterminer les résultats de la recherche. En particulier, nous allons surveiller le comptage du type de ressource **Stockage Blob Azure**.

- Nous effectuons la première recherche sans entrer aucun terme de recherche ni sélectionner aucune valeur dans les filtres rapides. La recherche porte sur toutes les ressources dans le catalogue. La liste des résultats de la recherche et le filtre rapide **Type de ressource** révèlent :

    - La liste des résultats de la recherche contient 164 230 ressources, qui sont toutes les ressources du catalogue.
    - La liste déroulante **Type de ressource** contient 43 entrées. Il s’agit de tous les types de ressources dans le catalogue. Étant donné que chaque ressource n’a qu’un seul et unique type, la somme du comptage des 43 types de ressources correspond à 164 230.
    - Le type de ressource **Stockage Blob Azure** est la première entrée de la liste déroulante du filtre rapide **Type de ressource**. Les valeurs sont classées par nombre, le plus grand en premier, donc **Stockage Blob Azure** est le type de ressource le plus courant. Son nombre équivaut à 118 174.

- Nous allons maintenant entrer **parquet** dans **Catalogue de recherche** et effectuer une autre recherche. Les résultats de la recherche incluent uniquement les ressources dont les caractéristiques correspondent à **parquet**. Cela réduit tous les comptages, comme suit :

    - La liste des résultats de la recherche contient 493 ressources. Seules 493 des 164 230 ressources du catalogue ont des caractéristiques qui correspondent à « parquet ».
    - La liste déroulante **Type de ressource** contient 15 entrées. Chacune des 493 ressources est de l’un de ces 15 types, et la somme des comptages des 15 types équivaut à 493.
    - Il existe 456 ressources de type **Stockage Blob Azure**. Les 37 autres ressources (493 moins 456) sont de l’un des 14 autres types.

- Nous examinons maintenant la liste déroulante d’un filtre rapide différent, **Classification** :

    - Il existe 12 classifications pour les 493 ressources dans la liste des résultats de la recherche. Les comptages pour les 493 ressources n’arrivent pas à un total de 493, car un nombre quelconque de classifications peut être affecté à une ressource.
    - La classification **Nom de la personne** est affecté aux 36 ressources, plus que toute autre classification.

- Nous sélectionnons la classification **Nom de la personne**. La liste des résultats de la recherche tombe à 36 ressources, comme prévu, car le comptage pour **Nom de la personne** était de 36. Aucun de ces résultats n’est de type **Stockage Blob Azure**.

Nous pouvons conclure qu’il n’y a pas de ressource dont le type est **Stockage Blob Azure** qui corresponde à **parquet** et dont la classification est **Nom de la personne**.

## <a name="start-the-search"></a>Démarrer la recherche

Lorsque vous effectuez une recherche, les termes de recherche que vous entrez dans **Rechercher le catalogue** sont mis en correspondance avec les caractéristiques de la ressource. Ces caractéristiques incluent le nom, le type, la classification et les contacts. Les ressources avec des caractéristiques correspondantes s’affichent dans la liste des résultats de la recherche, sauf si elles sont exclues par un filtre rapide.

Une fois que vous avez entré les termes de la recherche et défini les filtres rapides, démarrez la recherche de l’une des manières suivantes :

- Pour effectuer une recherche en fonction des termes que vous avez entrés, sélectionnez l’icône de recherche (:::image type="icon" source="./media/how-to-search-catalog/search-icon.png":::), appuyez sur **Entrée** ou sélectionnez **Afficher les résultats de la recherche**.
- Pour effectuer une recherche à l’aide des termes d’une recherche précédente, sélectionnez-les dans **Vos recherches récentes**.
- Pour effectuer une recherche à l’aide de termes suggérés, sélectionnez-les dans **Suggestions de recherche**.

Sélectionnez une ressource dans **Suggestions de ressources** pour accéder directement à la page de détails de la ressource. Aucune recherche n’est effectuée.

La liste des résultats pour les suggestions et les recherches de l’utilisateur peut différer légèrement. Les résultats de la liste des suggestions sont basés sur des correspondances approximatives, tandis que les résultats de la recherche initiée par l’utilisateur sont basés sur des correspondances exactes.

Lorsque vous effectuez une recherche, la page **Résultats de la recherche** s’affiche et répertorie les ressources trouvées par la recherche.

:::image type="content" source="./media/how-to-search-catalog/search-results.png" alt-text="Capture d’écran montrant les résultats de la recherche pour la valeur de recherche contoso.":::

Pour afficher les détails d’une ressource, sélectionnez un nom de ressource.

Utilisez les contrôles situés dans la partie inférieure d’une page de résultats de recherche pour accéder à d’autres pages de résultats de recherche.

:::image type="content" source="./media/how-to-search-catalog/page-navigation.png" alt-text="Capture d’écran montrant comment parcourir les pages de résultats de la recherche.":::

### <a name="sort-search-results"></a>Trier les résultats de la recherche

Utilisez **Trier par** pour trier les résultats de la recherche en fonction de la **Pertinence** ou du **Nom**.

:::image type="content" source="./media/how-to-search-catalog/sort-by.png" alt-text="Capture d’écran montrant comment trier les résultats de la recherche. Pour cet exemple, la liste déroulante Trier par est définie sur Pertinence.":::

### <a name="search-results-dynamic-filters"></a>Filtres dynamiques des résultats de la recherche

Le volet **Filtre** dans la page **Résultats de la recherche** contient des filtres qui fournissent un filtrage dynamique des ressources dans la liste des résultats de la recherche. Le filtrage est dynamique dans la mesure où des filtres supplémentaires peuvent apparaître en fonction des sélections de filtre.

Les filtres dynamiques présentent une case à cocher pour chaque valeur dans la liste déroulante. Utilisez ces cases à cocher pour filtrer sur autant de valeurs que vous le souhaitez.

#### <a name="search-results-dynamic-filter-filter-by-asset-type"></a>Filtre dynamique des résultats de la recherche : filtrer par type de ressource

Si vous sélectionnez un type de ressource dans la liste déroulante **Type de ressource**, des filtres dynamiques s’affichent pour vous permettre d’affiner vos résultats de recherche. Les filtres varient en fonction du type de ressource sélectionné. Par exemple, si vous sélectionnez **Azure SQL Database**, des filtres dynamiques s’affichent pour le serveur, la base de données et le schéma. Les valeurs de ces filtres proviennent des ressources figurant dans les résultats de recherche du type sélectionné.

:::image type="content" source="./media/how-to-search-catalog/asset-type-dynamic-filter.png" alt-text="L’élément de filtre Azure SQL Database est le seul élément « Type de ressource » qui est sélectionné. Un résultat de recherche de ce type de ressource est également mis en surbrillance." border="true":::

#### <a name="search-results-dynamic-filter-filter-by-classification"></a>Filtre dynamique des résultats de la recherche : filtrer par classification

Chaque classification de la liste **Classification** s’applique à au moins un élément dans la liste des résultats de recherche. Sélectionnez une ou plusieurs classifications pour limiter les résultats de votre recherche aux ressources des classifications sélectionnées.

:::image type="content" source="./media/how-to-search-catalog/classification-dynamic-filter.png" alt-text="Le filtre Classification de « Résultats de la recherche » est mis en surbrillance." border="true":::

#### <a name="edit-and-delete-search-results-filters"></a>Modifier et supprimer des filtres de résultats de recherche

Pour supprimer un filtre, désactivez la case à cocher en regard du nom du filtre.

### <a name="recently-accessed-assets"></a>Ressources consultées récemment

La section **Consultés récemment** de la zone de recherche étendue affiche les ressources les plus récemment consultées, le cas échéant.

- Sélectionnez **Afficher tout** dans la section **Consultés récemment** pour afficher la liste complète des ressources récemment consultées.

   :::image type="content" source="./media/how-to-search-catalog/get-to-recent-view.png" alt-text="Capture d’écran montrant la section Consultés récemment dans la zone de recherche étendue.":::

   Une liste des ressources consultées récemment s’affiche.

   :::image type="content" source="./media/how-to-search-catalog/recent.png" alt-text="Capture d’écran montrant la liste des ressources récemment consultées.":::

- Pour filtrer par nom, entrez une chaîne de recherche dans **Filtrer par nom**.

- Pour supprimer des éléments dans la liste, sélectionnez-les avec leurs cases à cocher, puis sélectionnez **Supprimer**.

   :::image type="content" source="./media/how-to-search-catalog/remove-from-recent-view.png" alt-text="Capture d’écran montrant comment supprimer des éléments d’une liste de ressources récemment consultées.":::

- Pour effacer l’intégralité de la liste, sélectionnez **Désactiver**.

   :::image type="content" source="./media/how-to-search-catalog/clear-recent-view-selections.png" alt-text="Capture d’écran montrant comment effacer une liste de ressources récemment consultées":::

### <a name="search-assets"></a>Rechercher des ressources

De nombreuses pages autres que **Accueil** ont une zone **Rechercher des ressources** dans leur partie supérieure. Par exemple, voici une page de détails sur les ressources, avec **Rechercher des ressources** mis en surbrillance.

:::image type="content" source="./media/how-to-search-catalog/search-assets.png" alt-text="Capture d’écran montrant une page de détails sur les ressources, avec Rechercher des ressources mis en surbrillance":::

Sélectionnez **Rechercher des ressources** pour lancer une zone de recherche comme celle que vous obtenez à partir de **Catalogue de recherche** dans **Accueil**, avec les mêmes fonctionnalités.

:::image type="content" source="./media/how-to-search-catalog/search-assets-dialog.png" alt-text="Capture d’écran montrant une zone Rechercher des ressources étendue.":::

## <a name="next-steps"></a>Étapes suivantes

- [Comment créer, importer et exporter des termes de glossaire](how-to-create-import-export-glossary.md)
- [Comment gérer les modèles de terme pour le glossaire métier](how-to-manage-term-templates.md)
