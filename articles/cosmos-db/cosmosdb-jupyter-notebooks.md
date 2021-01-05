---
title: Introduction à la prise en charge des Jupyter Notebooks intégrés dans Azure Cosmos DB (préversion)
description: Découvrez comment vous pouvez utiliser la prise en charge des Jupyter Notebooks intégrés dans Azure Cosmos DB pour exécuter des requêtes de manière interactive.
ms.service: cosmos-db
ms.topic: overview
ms.date: 05/19/2020
author: deborahc
ms.author: dech
ms.openlocfilehash: 8dca9e3e29796618b905c4d266eb674f82565969
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93097615"
---
# <a name="built-in-jupyter-notebooks-support-in-azure-cosmos-db-preview"></a>Prise en charge des Jupyter Notebooks intégrés dans Azure Cosmos DB (préversion)
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Jupyter Notebook est une application web open-source qui permet de créer et de partager des documents contenant du code en direct, des équations, des visualisations et du texte explicatif. 

Les Jupyter Notebooks intégrés à Azure Cosmos DB sont directement intégrés au portail Azure et vos comptes Azure Cosmos DB, ce qui les rend pratiques et faciles à utiliser. Les développeurs, les scientifiques des données, les ingénieurs et les analystes peuvent utiliser les Jupyter Notebooks pour effectuer une exploration des données, un nettoyage des données, des transformations de données, des simulations numériques, une modélisation statistique, une visualisation des données et du Machine Learning.

:::image type="content" source="./media/cosmosdb-jupyter-notebooks/cosmos-notebooks-overview.png" alt-text="Virtualisations de Jupyter Notebooks dans Azure Cosmos DB":::

Azure Cosmos DB prend en charge les notebooks C# et Python pour toutes les API, y compris Core (SQL), Cassandra, Gremlin, Table et API pour MongoDB. À l’intérieur du notebook, vous pouvez tirer parti des commandes et fonctionnalités intégrées qui facilitent la création de ressources Azure Cosmos DB, le chargement de données, ainsi que la consultation et la visualisation de vos données dans Azure Cosmos DB. 

:::image type="content" source="./media/cosmosdb-jupyter-notebooks/jupyter-notebooks-portal.png" alt-text="Prise en charge des Jupyter Notebooks dans Azure Cosmos DB":::

## <a name="benefits-of-jupyter-notebooks"></a>Avantages des Jupyter Notebooks

À l’origine, les Jupyter Notebooks ont été développés pour les applications de science des données écrites en Python et R. Toutefois, ils peuvent être utilisés de différentes façons pour différents types de projets, notamment :

**Visualisation des données :** Les Jupyter Notebooks vous permettent de visualiser des données sous la forme d’un notebook partagé qui restitue un jeu de données sous forme graphique. Vous pouvez créer des visualisations, apporter des modifications interactives au code partagé et au jeu de données, et partager les résultats.

**Partage du code :** Les services comme GitHub fournissent des méthodes pour partager du code, mais ils sont en grande partie non interactifs. Avec un Jupyter Notebook, vous pouvez voir le code, l’exécuter, puis afficher les résultats directement dans le portail Azure.

**Interactions en direct avec le code :** Le code d’un Jupyter Notebook est dynamique : vous pouvez le modifier et exécuter en temps réel des mises à jour de façon incrémentielle. Vous pouvez également incorporer des contrôles utilisateur (par exemple, des curseurs ou des champs d’entrée de texte) à utiliser comme sources d’entrée pour le code, comme démonstrations ou comme preuves de concepts (POC).

**Documentation des exemples de code et des résultats de l’exploration des données :** Si vous avez un extrait de code et souhaitez expliquer ligne par ligne son fonctionnement, vous pouvez l’incorporer dans un notebook Jupyter. Vous pouvez ajouter l’interactivité et la documentation en même temps.

**Commandes intégrées pour Azure Cosmos DB :** Les commandes magiques intégrées à Azure Cosmos DB facilitent l’interaction avec votre compte. Vous pouvez utiliser des commandes comme %%upload et %%sql pour charger des données dans un conteneur et les interroger à l’aide de la [syntaxe d’API SQL](sql-query-getting-started.md). Vous n’avez pas besoin d’écrire du code personnalisé supplémentaire.

**Environnement « tout dans un emplacement unique » :** Les Jupyter Notebooks combinent du code, du texte enrichi, des images, des vidéos, des animations, des équations mathématiques, des tracés, des cartes, des figures interactives, des widgets et des interfaces graphiques utilisateur dans un document unique.

## <a name="components-of-a-jupyter-notebook"></a>Composants d’un Jupyter Notebook

Les Jupyter Notebooks peuvent inclure plusieurs types de composants, chacun étant organisé en blocs discrets ou cellules :

**Texte et code HTML :** il est possible d’insérer à tout moment dans le document du texte brut ou du texte annoté dans la syntaxe Markdown afin de générer du code HTML. Le style CSS peut également être inclus inline ou ajouté au modèle utilisé pour générer le notebook.

**Code et sortie :** Les Jupyter Notebooks prennent en charge le code Python et C#. Les résultats du code exécuté s’affichent immédiatement après les blocs de code. De plus, les blocs de code peuvent être exécutés plusieurs fois dans l’ordre de votre choix.

**Visualisations :** vous pouvez générer le graphisme et les graphiques à partir du code à l’aide de modules tels que Matplotlib, Plotly ou Bokeh, entre autres. Comme pour la sortie, ces visualisations s’affichent inline en regard du code qui les génère. Comme pour la sortie, ces visualisations s’affichent inline en regard du code qui les génère.

**Multimédia :** Étant donné que les Jupyter Notebooks sont basés sur la technologie web, ils peuvent afficher tous les types de contenu multimédia pris en charge par une page web. Vous pouvez les inclure dans un notebook en tant qu’éléments HTML, ou vous pouvez les générer programmatiquement à l’aide du module `IPython.display`.

**Données :** Vous pouvez importer les données des conteneurs Azure Cosmos et les résultats des requêtes programmatiquement dans un Jupyter Notebook. Utilisez les commandes magiques intégrées pour charger ou interroger des données dans Azure Cosmos DB. 

## <a name="next-steps"></a>Étapes suivantes

Pour bien démarrer avec les Jupyter Notebooks intégrés dans Azure Cosmos DB, consultez les articles suivants :

* [Activer les notebooks dans un compte Azure Cosmos](enable-notebooks.md)
* [Utiliser les fonctionnalités et commandes des notebooks Python](use-python-notebook-features-and-commands.md)
* [Utiliser les fonctionnalités et commandes des notebooks C#](use-csharp-notebook-features-and-commands.md)
