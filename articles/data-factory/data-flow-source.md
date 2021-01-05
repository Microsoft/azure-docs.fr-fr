---
title: Transformation de la source d’un mappage de flux de données
description: Découvrez comment configurer une transformation de la source dans le mappage de flux de données.
author: kromerm
ms.author: makromer
manager: anandsub
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 12/08/2020
ms.openlocfilehash: d375632ad02f9ec7cacf1708ac81c4f257916609
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96929240"
---
# <a name="source-transformation-in-mapping-data-flow"></a>Transformation de la source d’un mappage de flux de données

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Une transformation de la source configure votre source de données pour le flux de données. Lorsque vous concevez des flux de données, la première étape consiste toujours à configurer une transformation de source. Pour ajouter une source, sélectionnez **Ajouter une source** dans le canevas de flux de données.

Chaque flux de données nécessite au moins une transformation de source. Vous pouvez cependant ajouter autant de sources que nécessaire pour effectuer vos transformations de données. Vous pouvez joindre ces sources à l’aide d’une transformation de jointure, de recherche ou d’union.

Chaque transformation de source est associée à un jeu de données ou à un service lié unique. Le jeu de données définit la forme et l’emplacement des données que vous voulez lire ou écrire. Si vous utilisez un jeu de données basé sur des fichiers, vous pouvez utiliser des caractères génériques et des listes de fichiers dans votre source pour utiliser plusieurs fichiers à la fois.

## <a name="inline-datasets"></a>Jeux de données inline

Lorsque vous créez une transformation de source, vous devez d’abord décider si les informations de votre source doivent être définies dans un objet de jeu de données ou dans la transformation de source. La plupart des formats sont uniquement disponibles dans l’un ou l’autre. Pour savoir comment utiliser un connecteur en particulier, référez-vous à la documentation le concernant.

Si un format est pris en charge à la fois inlined et dans les objets de jeu de données, notez que ces deux options présentent des avantages. Les objets de jeu de données sont des entités réutilisables qui peuvent être utilisées dans d’autres flux de données et activités telles que la copie. Ces entités réutilisables sont particulièrement utiles lorsque vous utilisez un schéma renforcé. Les jeux de données ne sont pas basés sur Spark. Il peut arriver que vous deviez remplacer certains paramètres ou la projection de schéma dans la transformation de source.

Les jeux de données inlined sont recommandés lors de l’utilisation de schémas flexibles, d’instances sources uniques ou de sources paramétrables. Si votre source est fortement paramétrée, les jeux de données inlined vous permettront de ne pas créer d’objet « factice ». Les jeux de données inlined sont basés sur Spark et leurs propriétés sont natives au flux de données.

Pour utiliser un jeu de données inlined, sélectionnez le format de votre choix à l’aide du sélecteur **Type de source**. Au lieu de sélectionner un jeu de données source, sélectionnez le service lié auquel vous souhaitez vous connecter.

![Screenshot that shows Inline selected.](media/data-flow/inline-selector.png "Capture d’écran montrant l’option Inlined sélectionnée.")

##  <a name="supported-source-types"></a><a name="supported-sources"></a> Types de sources pris en charge

Le flux de données de mappage suit une approche basée sur l’extraction, le chargement et la transformation (ELT, extract, load, transform) et fonctionne avec des jeux de données *intermédiaires* qui se trouvent tous dans Azure. Actuellement, les jeux de données suivants peuvent être utilisés dans une transformation de source.

| Connecteur | Format | Jeu de données/Inlined |
| --------- | ------ | -------------- |
| [Stockage Blob Azure](connector-azure-blob-storage.md#mapping-data-flow-properties) | [Avro](format-avro.md#mapping-data-flow-properties)<br>[Texte délimité](format-delimited-text.md#mapping-data-flow-properties)<br>[Delta](format-delta.md)<br>[Excel](format-excel.md#mapping-data-flow-properties)<br>[JSON](format-json.md#mapping-data-flow-properties) <br>[ORC](format-orc.md#mapping-data-flow-properties)<br/>[Parquet](format-parquet.md#mapping-data-flow-properties)<br>[XML](format-xml.md#mapping-data-flow-properties) | ✓/-<br>✓/-<br>-/✓<br>✓/✓<br/>✓/-<br>✓/✓<br/>✓/-<br>✓/✓ |
| [Azure Cosmos DB (API SQL)](connector-azure-cosmos-db.md#mapping-data-flow-properties) | | ✓/- |
| [Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md#mapping-data-flow-properties) | [Avro](format-avro.md#mapping-data-flow-properties)<br>[Texte délimité](format-delimited-text.md#mapping-data-flow-properties)<br>[Excel](format-excel.md#mapping-data-flow-properties)<br>[JSON](format-json.md#mapping-data-flow-properties)<br>[ORC](format-orc.md#mapping-data-flow-properties)<br/>[Parquet](format-parquet.md#mapping-data-flow-properties)<br>[XML](format-xml.md#mapping-data-flow-properties) | ✓/-<br>✓/-<br>✓/✓<br/>✓/-<br>✓/✓<br/>✓/-<br>✓/✓ |
| [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md#mapping-data-flow-properties) | [Avro](format-avro.md#mapping-data-flow-properties)<br>[Common Data Model](format-common-data-model.md#source-properties)<br>[Texte délimité](format-delimited-text.md#mapping-data-flow-properties)<br>[Delta](format-delta.md)<br>[Excel](format-excel.md#mapping-data-flow-properties)<br>[JSON](format-json.md#mapping-data-flow-properties)<br>[ORC](format-orc.md#mapping-data-flow-properties)<br/>[Parquet](format-parquet.md#mapping-data-flow-properties)<br>[XML](format-xml.md#mapping-data-flow-properties) | ✓/-<br/>-/✓<br>✓/-<br>-/✓<br>✓/✓<br>✓/-<br/>✓/✓<br/>✓/-<br>✓/✓ |
| [Base de données Azure pour PostgreSQL](connector-azure-database-for-postgresql.md) |  | ✓/✓ |
| [Azure SQL Database](connector-azure-sql-database.md#mapping-data-flow-properties) | | ✓/- |
| [Azure SQL Managed Instance (préversion)](connector-azure-sql-managed-instance.md#mapping-data-flow-properties) | | ✓/- |
| [Azure Synapse Analytics](connector-azure-sql-data-warehouse.md#mapping-data-flow-properties) | | ✓/- |
| [Hive](connector-hive.md#mapping-data-flow-properties) | | -/✓ |
| [Snowflake](connector-snowflake.md) | | ✓/✓ |

Les paramètres propres à ces connecteurs se trouvent sous l’onglet **Options de la source**. Vous trouverez des informations et des exemples de scripts de flux de données concernant ces paramètres dans la documentation relative aux connecteurs.

Azure Data Factory a accès à plus de [90 connecteurs natifs](connector-overview.md). Pour inclure dans votre flux de données des données provenant de ces autres sources, utilisez l’outil Copier l’activité pour charger ces données dans l’une des zones de transit prises en charge.

## <a name="source-settings"></a>Paramètres de la source

Une fois que vous avez ajouté une source, configurez-la dans l’onglet **Paramètres de la source**. Sous cet onglet, vous pouvez sélectionner ou créer le jeu de données vers lequel pointe la source. Vous pouvez également sélectionner des options de schéma et d’échantillonnage pour vos données.

Les valeurs de développement des paramètres de jeux de données peuvent être configurées dans les [paramètres de débogage](concepts-data-flow-debug-mode.md). (Le mode débogage doit être activé.)

![Capture d’écran montrant l’onglet Paramètres de la source](media/data-flow/source1.png "Capture d’écran montrant l’onglet Paramètres de la source")

**Nom du flux de sortie** : nom de la transformation de source.

**Type de source** : permet de choisir entre un jeu de données inline et un objet DataSet existant.

**Tester la connexion** : Faites un test pour déterminer si le service Spark du flux de données peut se connecter au service lié utilisé dans votre jeu de données source. Le mode de débogage doit être activé pour que cette fonctionnalité soit activée.

**Dérive de schéma** : La [dérive de schéma](concepts-data-flow-schema-drift.md) est la capacité de Data Factory à gérer nativement des schémas flexibles dans vos flux de données sans avoir besoin de définir explicitement des changements de colonnes.

* Cochez la case **Autoriser la dérive de schéma** si les colonnes sources sont amenées à changer souvent. Ce paramètre autorise tous les champs sources entrants à circuler jusqu’au récepteur, par le biais des transformations.

* La sélection de l’option **Déduire les types de colonnes dérivés** ordonne à Data Factory de détecter et de définir les types de données pour chaque nouvelle colonne découverte. Si cette fonctionnalité est désactivée, toutes les colonnes dérivées seront de type chaîne.

**Valider le schéma :** Quand la case **Valider le schéma** est cochée, l’exécution du flux de données échoue si les données sources entrantes ne correspondent pas au schéma défini du jeu de données.

**Nombre de lignes à ignorer** : Ce champ spécifie le nombre de lignes à ignorer au début du jeu de données.

**Échantillonnage** : Activez **Échantillonnage** pour limiter le nombre de lignes provenant de la source. Utilisez ce paramètre quand vous testez ou échantillonnez des données à partir de votre source à des fins de débogage.

Pour vérifier que votre source est correctement configurée, activez le mode débogage et récupérez un aperçu des données. Pour en savoir plus, consultez [Mode débogage](concepts-data-flow-debug-mode.md).

> [!NOTE]
> Quand le mode débogage est activé, la configuration de la limite du nombre de lignes dans les paramètres de débogage remplace le paramètre d’échantillonnage dans la source lors de l’affichage de l’aperçu des données.

## <a name="source-options"></a>Options de la source

L’onglet **Options de la source** contient des paramètres propres au connecteur et au format choisis. Pour plus d’informations et d’exemples, consultez la [documentation du connecteur](#supported-sources) en question.

## <a name="projection"></a>Projection

Comme les schémas des jeux de données, la projection d’une source définit les colonnes, les types et les formats des données sources. Pour la plupart des types de jeu de données, tels que SQL et Parquet, la projection dans une source est corrigée pour refléter le schéma défini dans un jeu de données. Quand vos fichiers sources ne sont pas fortement typés (par exemple, des fichiers .csv plats et non des fichiers Parquet), vous pouvez définir les types de données pour chaque champ de la transformation de source.

![Screenshot that shows settings on the Projection tab.](media/data-flow/source3.png "Capture d’écran montrant les paramètres sous l’onglet Projection.")

Si votre fichier texte ne comporte aucun schéma défini, sélectionnez **Détecter le type de données** afin que Data Factory échantillonne et déduise les types de données. Sélectionnez **Définir le format par défaut** pour détecter automatiquement les formats de données par défaut.

**Réinitialiser le schéma** réinitialise la projection en fonction de ce qui est défini dans le jeu de données référencé.

Vous pouvez modifier les types de données des colonnes lors d’une transformation de colonne dérivée en aval. Utilisez une transformation de sélection pour modifier les noms de colonnes.

### <a name="import-schema"></a>Importer un schéma

Sélectionnez le bouton **Importer un schéma** sous l’onglet **Projection** pour utiliser un cluster de débogage actif dans le but de créer une projection de schéma. Il est disponible dans chaque type de source. L’importation du schéma ici remplace la projection définie dans le jeu de données. L’objet de jeu de données ne sera pas changé.

L’importation de schéma est utile dans les jeux de données tels qu’Avro et Azure Cosmos DB qui prennent en charge des structures de données complexes et qui ne nécessitent pas l’existence de définitions de schéma dans le jeu de données. Pour les jeux de données inlined, l’importation de schéma est la seule façon de référencer des métadonnées de colonne sans dérive de schéma.

## <a name="optimize-the-source-transformation"></a>Optimiser la transformation de la source

L’onglet **Optimiser** permet de modifier les informations de partition à chaque étape de transformation. Dans la plupart des cas, l’option **Utiliser le partitionnement actuel** sera optimisée afin d’obtenir la structure de partitionnement idéale d’une source.

Si vous lisez des données à partir d’une source Azure SQL Database, c’est probablement le partitionnement **Source** personnalisé qui permettra de lire les données le plus rapidement. Data Factory lit les requêtes volumineuses en établissant des connexions avec votre base de données en parallèle. Ce partitionnement source peut être effectué sur une colonne ou à l’aide d’une requête.

![Screenshot that shows the Source partition settings.](media/data-flow/sourcepart3.png "Capture d’écran montrant les paramètres de partitionnement Source.")

Pour plus d’informations sur l’optimisation dans le mappage de flux de données, consultez la section [Onglet Optimiser](concepts-data-flow-overview.md#optimize).

## <a name="next-steps"></a>Étapes suivantes

Commencez à créer votre flux de données avec une [transformation de colonne dérivée](data-flow-derived-column.md), puis une [transformation de sélection](data-flow-select.md).
