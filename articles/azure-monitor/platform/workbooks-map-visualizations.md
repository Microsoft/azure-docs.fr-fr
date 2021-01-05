---
title: Visualisations de classeurs sous forme de cartes Azure Monitor
description: Découvrez les visualisations de classeurs sous forme de cartes Azure Monitor.
services: azure-monitor
author: lgayhardt
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 11/25/2020
ms.author: lagayhar
ms.openlocfilehash: 9b79efeeb90627bee6096c9142d8180896150295
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96439607"
---
# <a name="map-visualization"></a>Visualisation Carte

La visualisation sous forme de carte présente une synthèse des données de monitoring. Elle aide à repérer les problèmes liés à des endroits spécifiques en offrant la possibilité d’agréger toutes les données correspondant à chaque emplacement, pays ou région.

La capture d’écran ci-dessous montre le volume total de transactions et la latence de bout en bout pour différents comptes de stockage. La taille y est déterminée par le nombre total de transactions. Les métriques de couleur présentes sous la carte indiquent la latence de bout en bout. Un premier constat est que le nombre de transactions de la région **USA Ouest** est inférieur à celui de la région **USA Est**, mais que la latence de bout en bout de la première est supérieure à celle de la seconde. Nous pouvons en tirer la conclusion initiale selon laquelle la région **USA Ouest** présente un problème.

![Capture d’écran de la carte des emplacements Azure](./media/workbooks-map-visualizations/map-performance-example.png)

## <a name="adding-a-map"></a>Ajout d’une carte

Une visualisation sous forme de carte est possible si les données/métriques sous-jacentes comportent des informations de latitude/longitude, des informations sur les ressources Azure, des informations d’emplacement Azure, un nom de pays/région ou un code de pays/région.

### <a name="using-azure-location"></a>Avec l’emplacement Azure

1. Basculez le classeur en mode d’édition en sélectionnant l’élément **Modifier** de la barre d’outils.
2. Sélectionnez **Ajouter**, puis *Ajouter une requête*.
3. Remplacez la *Source de données* par `Azure Resource Graph`, puis choisissez n’importe quel abonnement disposant d’un compte de stockage.
4. Entrez la requête ci-dessous pour votre analyse, puis sélectionnez **Exécuter la requête**.

    ```kusto
    where  type =~ 'microsoft.storage/storageaccounts'
    | summarize count() by location
    ```

5. Donnez à *Taille* la valeur `Large`.
6. Donnez à *Visualisation* la valeur `Map`.
7. Tous les paramètres seront remplis automatiquement. En ce qui concerne les paramètres personnalisés, sélectionnez le bouton **Paramètres de la carte** pour ouvrir le volet Paramètres.
8. Voici une capture d’écran de la visualisation sous forme de carte des comptes de stockage de chacune des régions Azure de l’abonnement sélectionné.

![Capture d’écran de la carte des emplacements Azure avec la requête ci-dessus](./media/workbooks-map-visualizations/map-azure-location-example.png)

### <a name="using-azure-resource"></a>Avec la ressource Azure

1. Basculez le classeur en mode d’édition en sélectionnant l’élément **Modifier** de la barre d’outils.
2. Sélectionnez **Ajouter**, puis *Ajouter une métrique*.
3. Utilisez un abonnement disposant de comptes de stockage.
4. Remplacez le *Type de ressource* par `storage account`. Dans *Ressource*, sélectionnez plusieurs comptes de stockage.
5. Sélectionnez **Ajouter une métrique**, puis ajoutez une métrique Transaction.
    1. Espace de noms : `Account`
    2. Métrique : `Transactions`
    3. Agrégation : `Sum`
    
    ![Capture d’écran de la métrique Transaction](./media/workbooks-map-visualizations/map-transaction-metric.png)
1. Sélectionnez **Ajouter une métrique**, puis ajoutez la métrique Success E2E Latency.
    1. Espace de noms : `Account`
    1. Métrique : `Success E2E Latency`
    1. Agrégation : `Average`
    
    ![Capture d’écran de la métrique Success E2E Latency](./media/workbooks-map-visualizations/map-e2e-latency-metric.png)
1. Donnez à *Taille* la valeur `Large`.
1. Donnez à la taille de *Visualisation* la valeur `Map`.
1. Sous **Paramètres de la carte**, définissez les paramètres suivants :
    1. Informations d’emplacement avec : `Azure Resource`
    1. Champ de ressource Azure : `Name`
    1. Dimensionner selon : `microsoft.storage/storageaccounts-Transaction-Transactions`
    1. Agrégation pour l’emplacement : `Sum of values`
    1. Type de coloration : `Heatmap`
    1. Colorer selon : `microsoft.storage/storageaccounts-Transaction-SuccessE2ELatency`
    1. Agrégation pour la couleur : `Sum of values`
    1. Palette de couleurs : `Green to Red`
    1. Valeur minimale : `0`
    1. Valeur de métrique : `microsoft.storage/storageaccounts-Transaction-SuccessE2ELatency`
    1. Agrégation des autres métriques selon : `Sum of values`
    1. Cochez la case **Mise en forme personnalisée**.
    1. Unité : `Milliseconds`
    1. Style : `Decimal`
    1. Nombre maximal de chiffres décimaux : `2`

### <a name="using-countryregion"></a>Avec le pays/la région

1. Basculez le classeur en mode d’édition en sélectionnant l’élément **Modifier** de la barre d’outils.
2. Sélectionnez **Ajouter*, puis *Ajouter une requête*.
3. Remplacez la *Source de données* par `Log`.
4. Sélectionnez le *Type de ressource* `Application Insights`, puis choisissez une ressource Application Insights contenant des données pageViews.
5. Utilisez l’Éditeur de requête pour entrer le code KQL de votre analyse, puis sélectionnez **Exécuter la requête**.

    ```kusto
    pageViews
    | project duration, itemCount, client_CountryOrRegion
    | limit 20
    ```

6. Définissez les valeurs de taille sur `Large`.
7. Donnez à la visualisation la valeur `Map`.
8. Tous les paramètres seront remplis automatiquement. Pour les paramètres personnalisés, sélectionnez **Paramètres de la carte**.

### <a name="using-latitudelocation"></a>Avec la latitude/l’emplacement

1. Basculez le classeur en mode d’édition en sélectionnant l’élément **Modifier** de la barre d’outils.
2. Sélectionnez **Ajouter*, puis *Ajouter une requête*.
3. Remplacez la *Source de données* par `JSON`.
1. Entrez les données JSON ci-dessous dans l’Éditeur de requête, puis sélectionnez **Exécuter la requête**.
1. Définissez les valeurs *Taille* sur `Large`.
1. Donnez à *Visualisation* la valeur `Map`.
1. Dans **Paramètres de la carte**, définissez *Étiquette de métrique* sur `displayName` sous « Paramètres des métriques », puis sélectionnez **Enregistrer et fermer**.

La visualisation sous forme de carte ci-dessous montre les utilisateurs de chaque position de latitude et de longitude avec l’étiquette sélectionnée pour les métriques.

![Capture d’écran d’une visualisation sous forme de carte montrant les utilisateurs de chaque position de latitude et de longitude avec l’étiquette sélectionnée pour les métriques](./media/workbooks-map-visualizations/map-latitude-longitude-example.png)

## <a name="map-settings"></a>Paramètres de la carte

### <a name="layout-settings"></a>Paramètres de disposition

| Paramètre | Explication |
|:-------------|:-------------|
| `Location info using` | Sélectionnez un moyen d’obtenir l’emplacement des éléments affichés sur la carte. <br> **Latitude/Longitude** : sélectionnez cette option s’il existe des colonnes contenant des informations de latitude et de longitude. Chaque ligne comportant des données de latitude et de longitude sera affichée comme un élément distinct sur la carte. <br> **Emplacement Azure** : sélectionnez cette option s’il existe une colonne contenant des informations d’emplacement Azure (USA Est, Europe Ouest, Inde Centre, etc.). Spécifiez cette colonne. La latitude et la longitude correspondantes seront extraites pour chaque emplacement Azure, et les lignes du même emplacement regroupées (selon l’agrégation spécifiée) pour afficher les emplacements sur la carte. <br> **Ressource Azure** : sélectionnez cette option s’il existe une colonne contenant des informations sur les ressources Azure (compte de stockage, compte CosmosDB, etc.). Spécifiez cette colonne. La latitude et la longitude correspondantes seront extraites pour chaque ressource Azure, et les lignes du même emplacement (Azure) regroupées (selon l’agrégation spécifiée) pour afficher les emplacements sur la carte. <br> **Pays/région** : sélectionnez cette option s’il existe une colonne contenant des informations sur le nom/code de pays/région (US, États-Unis, IN, Inde, CN, Chine). Spécifiez cette colonne. La latitude et la longitude correspondantes seront extraites pour chaque nom/code de pays/région, et les lignes du même nom/code de pays/région regroupées pour afficher les emplacements sur la carte. Le nom et le code du pays ne seront pas regroupés en une seule entité sur la carte.
| `Latitude/Longitude` | Ces deux options seront visibles si la valeur du champ Informations sur l’emplacement est Latitude/Longitude. Sélectionnez la colonne comportant une latitude dans le champ de latitude et une longitude dans le champ de longitude. |
| `Azure location field` | Cette option sera visible si la valeur du champ Informations sur l’emplacement est Emplacement Azure. Sélectionnez la colonne contenant les informations d’emplacement Azure. |
| `Azure resource field` | Cette option sera visible si la valeur du champ Informations sur l’emplacement est Ressource Azure. Sélectionnez la colonne contenant les informations sur les ressources Azure. |
| `Country/Region field` | Cette option sera visible si la valeur du champ Informations sur l’emplacement est Pays ou région. Sélectionnez la colonne contenant les informations de pays/région. |
| `Size by` | Cette option permet de contrôler la taille des éléments affichés sur la carte. La taille dépend de la valeur qui se trouve dans la colonne spécifiée par l’utilisateur. Actuellement, le rayon du cercle est directement proportionnel à la racine carrée de la valeur de la colonne. Si « None… » est sélectionné, tous les cercles affichent la taille de la région par défaut.|
| `Aggregation for location` | Ce champ spécifie le mode d’agrégation des colonnes **Dimensionner selon** associées aux mêmes informations d’emplacement Azure/de ressource Azure/de pays/région. |
| `Minimum region size` | Ce champ spécifie le rayon minimal de l’élément affiché sur la carte. Il est utilisé lorsqu’il existe une différence significative entre les valeurs de la colonne Dimensionner selon, occasionnant un manque de visibilité des petits éléments sur la carte. |
| `Maximum region size` | Ce champ spécifie le rayon maximal de l’élément affiché sur la carte. Il est utilisé lorsque les valeurs de la colonne Dimensionner selon sont très élevées et couvrent une grande zone de la carte.|
| `Default region size` | Ce champ spécifie le rayon par défaut de l’élément affiché sur la carte. Il est utilisé lorsque la colonne Dimensionner selon a la valeur « None… » ou 0.|
| `Minimum value` | Ce champ correspond à la valeur minimale utilisée pour calculer la taille de la région. S’il n’est pas spécifié, il s’agit de la plus petite valeur obtenue après agrégation. |
| `Maximum value` | Ce champ correspond à la valeur maximale utilisée pour calculer la taille de la région. S’il n’est pas spécifié, il s’agit de la plus grande valeur obtenue après agrégation.|
| `Opacity of items on Map` | Ce champ spécifie la transparence des éléments affichés sur la carte. Une opacité de 1 signifie une absence de transparence. Une opacité de 0 indique que les éléments ne seront pas visibles sur la carte. S’il y a trop d’éléments sur la carte, l’opacité peut être définie sur une faible valeur afin que tous les éléments qui se chevauchent soient visibles.|

### <a name="color-settings"></a>Paramètres de couleur

| Type de coloration | Explication |
|:------------- |:-------------|
| `None` | Tous les nœuds ont la même couleur. |
| `Thresholds` | Dans ce type, les couleurs des cellules sont définies par des règles de seuil (par exemple, _Processeur > 90 % => Rouge, 60 % > Processeur > 90 % => Jaune, Processeur < 60 % => Vert_). <ul><li> **Colorer selon** : la valeur de cette colonne sera utilisée par la logique Seuils/Carte thermique.</li> <li>**Agrégation pour la couleur** : ce champ spécifie le mode d’agrégation des colonnes **Colorer selon** associées aux mêmes informations d’emplacement Azure/de ressource Azure/de pays/région. </li> <ul> |
| `Heatmap` | Dans ce type, les cellules sont colorées en fonction de la palette de couleurs et du champ Colorer selon. Elles possèdent également comme seuils les mêmes options **Colorer selon** et **Agrégation pour la couleur**. |

### <a name="metric-settings"></a>Paramètres des métriques
| Paramètre | Explication |
|:------------- |:-------------|
| `Metric Label` | Cette option sera visible si la valeur du champ Informations sur l’emplacement est Latitude/Longitude. À l’aide de cette fonctionnalité, l’utilisateur peut sélectionner l’étiquette à afficher pour les métriques indiquées sous la carte. |
| `Metric Value` | Ce champ spécifie la valeur de métrique à afficher sous la carte. |
| `Create 'Others' group after` | Ce champ spécifie la limite avant création d’un groupe « Autres ». |
| `Aggregate 'Others' metrics by` | Ce champ spécifie l’agrégation utilisée pour le groupe « Autres », s’il est affiché. |
| `Custom formatting` | Utilisez ce champ pour définir les options d’unités, de style et de mise en forme des valeurs numériques. Il s’agit de l’équivalent de la [mise en forme personnalisée de la grille](workbooks-grid-visualizations.md#custom-formatting).|

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment créer des [visualisations en nid d’abeille dans des classeurs](workbooks-honey-comb.md).