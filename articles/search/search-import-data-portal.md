---
title: Importer des données dans un index de recherche à l’aide du Portail Azure
titleSuffix: Azure Cognitive Search
description: Découvrez comment utiliser l’Assistant Importer des données dans le portail Azure pour explorer des données Azure à partir de Cosmos DB, de Stockage Blob, de Stockage Table, de SQL Database, de SQL Managed Instance et de SQL Server sur des machines virtuelles Azure.
author: HeidiSteen
manager: nitinme
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/04/2019
ms.openlocfilehash: 7cff009d5d1e187e8d0330fadca530b57b3e3d21
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88935209"
---
# <a name="import-data-wizard-for-azure-cognitive-search"></a>Assistant Importation de données pour la Recherche cognitive Azure

Le tableau de bord Recherche cognitive Azure du Portail Azure propose un Assistant **Importation des données** destiné au prototypage et au chargement d’un index. Cet article s’intéresse aux avantages et aux limitations de l’utilisation de l’Assistant, aux entrées et sorties et à certains aspects liés à l’utilisation. Pour obtenir des conseils pratiques concernant l’utilisation de l’Assistant avec les exemples de données intégrés, consultez le guide de démarrage rapide [Créer un index Recherche cognitive Azure à partir du Portail Azure](search-get-started-portal.md).

Cet Assistant effectue les opérations suivantes :

1 - Connexion à une source de données Azure prise en charge.

2 - Création d’un schéma d’index, déduit par l’échantillonnage des données sources.

3 - (Facultatif) Ajout d’enrichissements de l’IA pour extraire ou générer du contenu et la structure.

4 - Exécution de l’Assistant pour créer des objets, importer des données, définir une planification et d’autres options de configuration.

L’Assistant génère un certain nombre d’objets qui sont enregistrés dans votre service de recherche, auquel vous pouvez accéder par programmation ou avec d’autres outils.

## <a name="advantages-and-limitations"></a>Avantages et limitations

Avant d’écrire du code, vous pouvez utiliser l’Assistant à des fins de prototypage et de test de preuve de concept. L’Assistant se connecte à des sources de données externes, échantillonne les données pour créer un index initial, puis importe les données sous forme de documents JSON dans un index dans la Recherche cognitive Azure. 

L’échantillonnage est le processus par lequel un schéma d’index est déduit et présente quelques limitations. Quand la source de données est créée, l’Assistant choisit un échantillon de documents pour identifier les colonnes qui font partie de la source de données. Tous les fichiers ne sont pas lus, car l’opération pourrait durer des heures avec les sources de données très volumineuses. À partir d’une sélection de documents, les métadonnées sources, comme le nom ou le type de champ, sont utilisées pour créer une collection de champs dans un schéma d’index. Selon la complexité des données sources, vous devrez peut-être modifier le schéma initial dans un souci de précision, ou l’étendre à des fins d’exhaustivité. Vous pouvez faire en sorte que vos modifications soient incorporées dans la page de définition de l’index.

Globalement, les avantages de l’Assistant sont évidents : dans la mesure où les exigences sont respectées, vous pouvez créer un prototype d’index interrogeable en quelques minutes. Certaines complexités liées à l’indexation, comme la mise à disposition des données sous forme de documents JSON, sont gérées par l’Assistant.

En bref, les limitations connues sont les suivantes :

+ L’Assistant ne prend pas en charge l’itération ou la réutilisation. Chaque exécution de l’Assistant donne lieu à la création d’une configuration d’index, d’ensemble de compétences et d’indexeur. Seules les sources de données peuvent être conservées et réutilisées dans l’Assistant. Pour modifier ou affiner d’autres objets, vous devez utiliser les API REST ou le SDK .NET pour récupérer et modifier les structures.

+ Le contenu source doit résider dans une source de données Azure prise en charge.

+ L’échantillonnage porte sur un sous-ensemble des données sources. Pour les sources de données volumineuses, l’Assistant peut ne pas déceler des champs. Vous serez peut-être amené à étendre le schéma ou à corriger les types de données déduits si l’échantillonnage est insuffisant.

+ L’enrichissement par IA, tel que présenté sur le portail, se limite à quelques compétences intégrées. 

+ Une [base de connaissances](knowledge-store-concept-intro.md), qui peut être créée par l’Assistant, est limitée à quelques projections par défaut. Si vous voulez enregistrer des documents enrichis créés par l’Assistant, le conteneur d’objets blob et les tables sont fournis avec des noms et une structure par défaut.

<a name="data-source-inputs"></a>

## <a name="data-source-input"></a>Entrée de source de données

L’Assistant **Importation des données** se connecte à une source de données externe en utilisant la logique interne fournie par les indexeurs Recherche cognitive Azure, qui sont capables d’échantillonner la source, lire les métadonnées, décrypter les documents pour en lire le contenu et la structure et sérialiser le contenu sous forme de JSON pour une importation ultérieure dans la Recherche cognitive Azure.

Vous ne pouvez importer des données qu’à partir d’une seule table, d’une vue de base de données ou d’une structure de données équivalente, mais la structure peut inclure des sous-structures hiérarchiques ou imbriquées. Pour plus d’informations, consultez [How to model complex types](search-howto-complex-data-types.md) (Modélisation des types complexes).

Vous devez créer cette table ou vue unique avant d’exécuter l’Assistant et il/elle doit comporter du contenu. Pour des raisons évidentes, exécuter l’Assistant **Importation des données** sur une source de données vide ne présente aucun intérêt.

|  Sélection | Description |
| ---------- | ----------- |
| **Source de données existante** |Si des indexeurs sont déjà définis dans votre service de recherche, il existe peut-être une définition de source de données que vous pouvez réutiliser. Dans la Recherche cognitive Azure, les objets de source de données sont uniquement utilisés par les indexeurs. Vous pouvez créer un objet de source de données par programmation ou via l’Assistant **Importation des données**, puis le réutiliser si nécessaire.|
| **Exemples**| La Recherche cognitive Azure intègre deux exemples de sources de données qui sont utilisées dans des tutoriels et autres guides de démarrage rapide : une base de données immobilières SQL et une base d’hôtels toutes deux hébergées dans Cosmos DB. Pour consulter une procédure pas à pas basée sur l’exemple Hotels, reportez-vous au guide de démarrage rapide [Créer un index sur le portail Azure](search-get-started-portal.md). |
| [**Azure SQL Database ou SQL Managed Instance**](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md) |Le nom du service, les informations d’identification d’un utilisateur de base de données avec autorisation de lecture, ainsi que le nom de la base de données peuvent être spécifiés sur la page ou par le biais d’une chaîne de connexion ADO.NET. Choisissez l’option de chaîne de connexion permettant d’afficher ou de personnaliser les propriétés. <br/><br/>La table ou la vue qui fournit l’ensemble de lignes doit être spécifiée sur la page. Cette option s’affiche une fois que la connexion aboutit : vous pouvez alors faire votre choix dans une liste déroulante.|
| **SQL Server dans les machines virtuelles Azure** |Spécifiez un nom de service complet, un ID d’utilisateur et un mot de passe, ainsi qu’une base de données pour la chaîne de connexion. Afin d’utiliser cette source de données, vous devez avoir préalablement installé un certificat dans le magasin local pour chiffrer la connexion. Pour obtenir des instructions, reportez-vous à [Connexion de machines virtuelles SQL à la Recherche cognitive Azure](search-howto-connecting-azure-sql-iaas-to-azure-search-using-indexers.md). <br/><br/>La table ou la vue qui fournit l’ensemble de lignes doit être spécifiée sur la page. Cette option s’affiche une fois que la connexion aboutit : vous pouvez alors faire votre choix dans une liste déroulante. |
| [**Azure Cosmos DB**](search-howto-index-cosmosdb.md)|La configuration requise inclut le compte, la base de données et la collection. Tous les documents de la collection seront inclus dans l’index. Vous pouvez définir une requête pour aplatir ou filtrer l’ensemble de lignes, ou laisser la requête vide. Aucune requête n’est nécessaire dans cet Assistant.|
| [**Stockage Blob Azure**](search-howto-indexing-azure-blob-storage.md) |La configuration requise inclut le compte de stockage et un conteneur. Si les noms d’objets blob suivent une convention d’affectation de noms virtuelle à des fins de regroupement, vous pouvez indiquer la partie de répertoire virtuel du nom comme dossier sous le conteneur. Consultez la page [Indexation de Stockage Blob](search-howto-indexing-azure-blob-storage.md) pour plus d’informations. |
| [**Stockage Table Azure**](search-howto-indexing-azure-tables.md) |La configuration requise inclut le compte de stockage et un nom de table. Vous pouvez également spécifier une requête pour extraire un sous-ensemble des tables. Consultez la page [Indexation de Stockage Table](search-howto-indexing-azure-tables.md) pour plus d’informations. |

## <a name="wizard-output"></a>Sortie de l’Assistant

En arrière-plan, l’Assistant crée, configure et appelle les objets suivants. Une fois l’Assistant exécuté, vous pouvez trouver sa sortie dans les pages du portail. La page Vue d’ensemble de votre service contient des listes d’index, des indexeurs, des sources de données et des ensembles de compétences. Les définitions d’index peuvent être affichées sous forme de JSON complet sur le portail. Pour les autres définitions, vous pouvez utiliser l’[API REST](/rest/api/searchservice/) pour obtenir des objets spécifiques via GET.

| Object | Description | 
|--------|-------------|
| [Source de données](/rest/api/searchservice/create-data-source)  | Conserve les informations de connexion aux données sources, notamment les informations d’identification. Un objet de source de données est utilisé exclusivement avec les indexeurs. | 
| [Index](/rest/api/searchservice/create-index) | Structure de données physique utilisée pour la recherche en texte intégral et d’autres requêtes. | 
| [Ensemble de compétences](/rest/api/searchservice/create-skillset) | Ensemble complet d’instructions destiné à manipuler, transformer et mettre en forme du contenu, notamment en analysant et extrayant des informations de fichiers image. À l’exception des structures très simples et limitées, il comporte une référence à une ressource Cognitive Services qui assure l’enrichissement. Il peut aussi éventuellement contenir une définition de base de connaissances.  | 
| [Indexeur](/rest/api/searchservice/create-indexer)  | Objet de configuration spécifiant une source de données, un index cible, un ensemble de compétences facultatif, une planification facultative et des paramètres de configuration facultatifs pour la gestion des erreurs et l’encodage en base 64. |


## <a name="how-to-start-the-wizard"></a>Comment démarrer l’Assistant

L’Assistant Importation des données se démarre à partir de la barre de commandes dans la page Vue d’ensemble du service.

1. Dans le [portail Azure](https://portal.azure.com), ouvrez la page du service de recherche à partir du tableau de bord, ou [recherchez votre service](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices) dans la liste.

2. En haut de la page de présentation du service, cliquez sur **Importer des données**.

   ![Commande Importer des données dans le portail](./media/search-import-data-portal/import-data-cmd2.png "Démarrer l’Assistant Importer des données")

Vous pouvez aussi lancer l’**Importation des données** à partir d’autres services Azure, dont Azure Cosmos DB, Azure SQL Database, SQL Managed Instance et le stockage Blob Azure. Recherchez **Ajouter Recherche cognitive Azure** dans le volet de navigation de gauche de la page de présentation du service.

<a name="index-definition"></a>

## <a name="how-to-edit-or-finish-an-index-schema-in-the-wizard"></a>Comment modifier ou terminer un schéma d’index dans l’Assistant

L’Assistant génère un index incomplet qui sera rempli avec les documents obtenus à partir de la source de données d’entrée. Pour un index fonctionnel, assurez-vous que les éléments suivants sont bien définis.

1. La liste des champs est-elle complète ? Ajoutez de nouveaux champs qui ont échappé à l’échantillonnage et supprimez ceux qui n’apportent rien à une expérience de recherche ou qui ne seront pas utilisés dans une [expression de filtre](search-query-odata-filter.md) ou un [profil de scoring](index-add-scoring-profiles.md).

1. Le type de données convient-il pour les données entrantes ? La Recherche cognitive Azure prend en charge les [types de données EDM (Entity Data Model)](/rest/api/searchservice/supported-data-types). Pour les données Azure SQL, il existe un [tableau de mappages](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md#TypeMapping) qui présente les valeurs équivalentes. Pour plus d’informations, consultez [Mappages et transformations de champs](search-indexer-field-mappings.md).

1. Avez-vous un champ qui peut faire office de *clé* ? Ce champ doit être Edm.string et doit identifier un document de manière unique. Dans le cas des données relationnelles, elles peuvent être mappées à une clé primaire. Pour les objets blob, il peut s’agir de `metadata-storage-path`. Si des valeurs de champ comportent des espaces ou des tirets, vous devez définir l’option **Clé d’encodage en base-64** dans l’étape **Créer un indexeur**, sous **Options avancées**, pour supprimer le vérification de la validation pour ces caractères.

1. Définissez des attributs pour déterminer comment ce champ est utilisé dans un index. 

   Prenez votre temps dans cette étape, car les attributs déterminent l’expression physique des champs dans l’index. Si, par la suite, vous souhaitez modifier des attributs, même par programmation, vous devrez presque toujours supprimer et regénérer l’index. Les attributs de base comme **Searchable** et **Retrievable** ont un [impact négligeable sur le stockage](search-what-is-an-index.md#index-size). L’activation de filtres et l’utilisation de suggesteurs augmentent les besoins de stockage. 
   
   + **Possibilité de recherche** permet une recherche en texte intégral. Chaque champ utilisé dans les requêtes de forme libre ou dans les expressions de requête doit avoir cet attribut. Les index inversés sont créés pour chaque champ que vous marquez comme **Possibilité de recherche**.

   + **Récupérable** retourne le champ dans les résultats de la recherche. Chaque champ qui fournit du contenu aux résultats de recherche doit avoir cet attribut. La définition de ce champ n’a pas d’incidence notable sur la taille de l’index.

   + **Filtrable** permet de référencer le champ dans les expressions de filtre. Chaque champ utilisé dans une expression **$filter** doit avoir cet attribut. Les expressions de filtre sont des correspondances exactes. Les chaînes de texte demeurant intactes, un stockage supplémentaire est nécessaire pour recevoir le contenu textuel.

   + **À choix multiples** active le champ pour la navigation par facettes. Seuls les champs également marqués comme **Filtrables** peuvent être marqués comme **À choix multiples**.

   + **Triable** permet d’utiliser le champ dans un tri. Chaque champ utilisé dans une expression **$Orderby** doit avoir cet attribut.

1. Avez-vous besoin d’une [analyse lexicale](search-lucene-query-architecture.md#stage-2-lexical-analysis) ? Pour les champs Edm.string de type **Searchable**, vous pouvez définir un **analyseur** si vous voulez des fonctions d’indexation et d’interrogation qui offrent une prise en charge linguistique améliorée. 

   La valeur par défaut est *Standard Lucene*, mais vous pouvez choisir *Microsoft Anglais* si vous souhaitez utiliser l’analyseur de Microsoft pour le traitement lexical avancé, tel que la résolution des formes verbales et nominales irrégulières. Seuls des analyseurs linguistiques peuvent être spécifiés sur le portail. L’utilisation d’un analyseur personnalisé ou d’un analyseur non linguistique, comme Keyword, Pattern, etc., doit s’effectuer par programmation. Pour plus d’informations sur les analyseurs, consultez [Ajouter des analyseurs linguistiques](search-language-support.md).

1. Avez-vous besoin de fonctionnalités TypeAhead matérialisées par la saisie semi-automatique ou les suggestions de résultats ? Cochez la case **Suggesteur** pour activer les [suggestions de requête TypeAhead et la saisie semi-automatique](index-add-suggesters.md) sur les champs sélectionnés. Les suggesteurs s’ajoutent au nombre de termes tokenisés de votre index et occupent donc davantage d’espace de stockage.


## <a name="next-steps"></a>Étapes suivantes

La meilleure façon de comprendre les avantages et les limitations de l’Assistant est de le parcourir pas à pas. Le guide de démarrage rapide suivant vous guide à chaque étape.

> [!div class="nextstepaction"]
> [Créer un index Recherche cognitive Azure à l’aide du Portail Azure](search-get-started-portal.md)