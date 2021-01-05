---
title: Traçabilité des données dans Azure Purview (préversion)
description: Décrit les concepts relatifs à la traçabilité des données.
author: chanuengg
ms.author: csugunan
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: conceptual
ms.date: 11/30/2020
ms.openlocfilehash: ceb30cee194994e5b31db312f191194932c937b4
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96550182"
---
# <a name="data-lineage-in-azure-purview-data-catalog-client"></a>Traçabilité des données dans le client Data Catalog Azure Purview

Cet article fournit une vue d’ensemble de la traçabilité des données dans Data Catalog Azure Purview. Il explique également comment les systèmes de données peuvent s’intégrer au catalogue pour capturer la traçabilité des données. Purview peut capturer la traçabilité des données dans différentes parties du patrimoine de données de votre organisation et à différents niveaux de préparation, y compris :

- Données brutes entièrement mises en lots à partir de différentes plateformes
- Données transformées et préparées
- Données utilisées par les plateformes de visualisation

## <a name="use-cases"></a>Scénarios d’utilisation

La traçabilité des données est comprise au sens large comme le cycle de vie qui couvre l’origine des données, et où elle évolue au fil du temps dans le patrimoine des données. Elle est utilisée pour différents types de scénarios rétrospectifs, tels que la résolution des problèmes, le suivi de la cause racine dans les pipelines de données et le débogage. La traçabilité des données est également utilisée pour l’analyse de la qualité des données, la conformité et les scénarios souvent appelés analyse d’impact. La traçabilité est représentée visuellement pour montrer les données migrant de la source vers la destination, notamment la façon dont elles ont été transformées. Étant donné la complexité de la plupart des environnements de données d’entreprise, ces vues peuvent être difficiles à comprendre sans effectuer de consolidation ou de masquage des points de données périphériques.

## <a name="lineage-experience-in-azure-purview-data-catalog"></a>Expérience de traçabilité des données dans Data Catalog Azure Purview

Data Catalog Purview se connectera avec d’autres systèmes de traitement, de stockage et d’analyse des données pour extraire les informations de traçabilité. Les informations sont combinées pour représenter une expérience de traçabilité générique et spécifique aux scénarios dans le catalogue.

:::image type="content" source="media/concept-lineage/lineage-end-end.png" alt-text="traçabilité de bout en bout affichant les données copiées à partir du magasin d’objets BLOB jusqu’au tableau de bord Power BI":::

Votre patrimoine de données peut inclure des systèmes qui exécutent des systèmes d’extraction, de transformation (systèmes ETL/ELT), d’analyse et de visualisation des données. Chacun des systèmes capture des métadonnées statiques et opérationnelles riches qui décrivent l’état et la qualité des données dans la limite du système. L’objectif de la traçabilité dans un catalogue de données consiste à extraire le mouvement, la transformation et les métadonnées opérationnelles à partir de chaque système de données au grain le plus bas possible.

L’exemple suivant est un cas d’utilisation typique de migration de données sur plusieurs systèmes, dans lequel Data Catalog se connecte à chacun des systèmes à des fins de traçabilité.

- Data Factory copie les données d’une zone locale/brute vers une zone d’atterrissage dans le cloud. 
- Les systèmes de traitement de données tels que Synapse, Databricks traitent et transforment les données de la zone d’atterrissage vers la zone organisée à l’aide de notebooks.
- Traitement ultérieur des données dans des modèles analytiques pour optimiser les performances et l’agrégation des requêtes. 
- Les systèmes de visualisation des données utiliseront les jeux de données et les processus via leur méta-modèle pour créer un tableau de bord BI, des expérimentations ML, etc.

## <a name="lineage-granularity"></a>Granularité de la traçabilité

Cette section couvre les détails relatifs à la granularité dont les informations de traçabilité sont collectées par un catalogue de données. Cette granularité peut varier en fonction des systèmes de données.

### <a name="entity-level-lineage-sources--process--targets"></a>Traçabilité au niveau de l’entité : Source(s) > Processus > Cible(s) 

- La traçabilité est représentée sous la forme d’un graphique. Celui-ci contient en général des entités source et cible dans des systèmes de stockage de données qui sont connectés par un processus appelé par un système de calcul. 
- Les systèmes de données se connectent au catalogue de données pour générer et signaler un objet unique référençant l’objet physique du système de données sous-jacent, par exemple : Procédure stockée SQL, notebooks, etc.
- La traçabilité haute fidélité avec des métadonnées supplémentaires telles que la propriété est capturée pour afficher la traçabilité dans un format lisible pour les entités source et cible. Par exemple : traçabilité au niveau de la table Hive au lieu de partitions ou au niveau du fichier.

### <a name="column-or-attribute-level-lineage"></a>Traçabilité au niveau de la colonne ou de l’attribut

Identifiez le ou les attributs d’une entité source utilisée pour créer ou dériver un ou des attributs dans l’entité cible. Le nom de l’attribut source peut être conservé ou renommé dans une cible. Des systèmes tels qu’ADF peuvent effectuer une copie de un à un d’un environnement local vers le cloud. Par exemple : `Table1/ColumnA -> Table2/ColumnA`.

### <a name="process-execution-status"></a>État d’exécution du processus

Pour prendre en charge les scénarios d’analyse de la cause racine et de qualité des données, nous capturons l’état d’exécution des tâches dans les systèmes de traitement des données. Cette exigence n’a rien à avoir avec le remplacement des fonctionnalités d’analyse d’autres systèmes de traitement de données, et l’objectif n’est pas non plus de les remplacer. 

## <a name="summary"></a>Résumé

La traçabilité est une fonctionnalité critique de Data Catalog Purview visant à prendre en charge les scénarios de qualité, d’approbation et d’audit. L’objectif d’un catalogue de données est de créer une infrastructure robuste dans laquelle tous les systèmes de données au sein de votre environnement peuvent se connecter naturellement et créer un rapport de traçabilité. Une fois les métadonnées disponibles, le catalogue de données peut réunir les métadonnées fournies par les systèmes de données pour alimenter les cas d’utilisation de la gouvernance des données.

## <a name="next-steps"></a>Étapes suivantes

* [Démarrage rapide : Créer un compte Azure Purview dans le Portail Azure](create-catalog-portal.md)
* [Démarrage rapide : Créer un compte Azure Purview en utilisant Azure PowerShell ou Azure CLI](create-catalog-powershell.md)
* [Démarrage rapide : Utiliser Purview Studio](use-purview-studio.md)