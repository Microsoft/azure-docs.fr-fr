---
title: Transformation du récepteur dans le flux de données de mappage
description: Apprenez à configurer une transformation de récepteur dans le flux de données de mappage.
author: kromerm
ms.author: makromer
ms.reviewer: daperlov
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 12/08/2020
ms.openlocfilehash: 242249e3ab7fbedf9f19f3cb9a49fc4a8359f4ae
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96929342"
---
# <a name="sink-transformation-in-mapping-data-flow"></a>Transformation du récepteur dans le flux de données de mappage

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Une fois que vous avez terminé de transformer vos données, écrivez-les dans un magasin de destination à l’aide de la transformation du récepteur. Chaque flux de données requiert la transformation d'au moins un récepteur. Vous pouvez cependant écrire dans autant de récepteurs que nécessaire pour terminer votre flux de transformations. Pour écrire dans des récepteurs supplémentaires, créez de nouveaux flux via de nouvelles branches et des fractionnements conditionnels.

Chaque transformation de récepteur est associée exactement à un objet de jeu de données Azure Data Factory ou à un service lié. La transformation du récepteur détermine la forme et l’emplacement des données sur lesquelles vous souhaitez écrire.

## <a name="inline-datasets"></a>Jeux de données inlined

Lors de la création d’une transformation de récepteur, indiquez si vos informations de récepteur sont définies au sein d’un objet de jeu de données ou dans la transformation du récepteur. La plupart des formats sont uniquement disponibles dans l’un ou l’autre. Pour savoir comment utiliser un connecteur en particulier, référez-vous à la documentation le concernant.

Si un format est pris en charge à la fois inlined et dans les objets de jeu de données, notez que ces deux options présentent des avantages. Les objets de jeu de données sont des entités réutilisables qui peuvent être utilisées dans d’autres flux de données et activités telles que la copie. Ces entités réutilisables sont particulièrement utiles lorsque vous utilisez un schéma renforcé. Les jeux de données ne sont pas basés sur Spark. Il peut arriver que vous deviez remplacer certains paramètres ou la projection de schéma dans la transformation du récepteur.

Les jeux de données inlined sont recommandés lorsque vous utilisez des schémas flexibles, d’instances de récepteurs uniques ou de récepteurs paramétrables. Si votre récepteur est fortement paramétrable, les jeux de données inlined vous permettront de ne pas créer d’objet « factice ». Les jeux de données inlined sont basés sur Spark et leurs propriétés sont natives au flux de données.

Pour utiliser un jeu de données inlined, sélectionnez le format de votre choix à l’aide du sélecteur **Type de récepteur**. Au lieu de sélectionner un jeu de données récepteur, sélectionnez le service lié auquel vous souhaitez vous connecter.

![Capture d’écran montrant l’option Inlined sélectionnée.](media/data-flow/inline-selector.png "Capture d’écran montrant l’option Inlined sélectionnée.")

##  <a name="supported-sink-types"></a><a name="supported-sinks"></a> Types de récepteurs pris en charge

Le flux de données de mappage suit une approche basée sur l’extraction, le chargement et la transformation (ELT, extract, load, transform) et fonctionne avec des jeux de données *intermédiaires* qui se trouvent tous dans Azure. Actuellement, les jeux de données suivants peuvent être utilisés dans une transformation de source.

| Connecteur | Format | Jeu de données/Inlined |
| --------- | ------ | -------------- |
| [Stockage Blob Azure](connector-azure-blob-storage.md#mapping-data-flow-properties) | [Avro](format-avro.md#mapping-data-flow-properties) <br>[Texte délimité](format-delimited-text.md#mapping-data-flow-properties) <br>[Delta](format-delta.md) <br>[JSON](format-json.md#mapping-data-flow-properties) <br/>[ORC](format-orc.md#mapping-data-flow-properties)<br>[Parquet](format-parquet.md#mapping-data-flow-properties) | ✓/- <br>✓/- <br>-/✓ <br>✓/- <br>✓/✓<br>✓/- |
| [Azure Cosmos DB (API SQL)](connector-azure-cosmos-db.md#mapping-data-flow-properties) | | ✓/- |
| [Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md#mapping-data-flow-properties) | [Avro](format-avro.md#mapping-data-flow-properties) <br>[Texte délimité](format-delimited-text.md#mapping-data-flow-properties) <br>[JSON](format-json.md#mapping-data-flow-properties) <br/>[ORC](format-orc.md#mapping-data-flow-properties)<br/>[Parquet](format-parquet.md#mapping-data-flow-properties) | ✓/- <br>✓/- <br>✓/- <br>✓/✓<br>✓/- |
| [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md#mapping-data-flow-properties) | [Avro](format-avro.md#mapping-data-flow-properties) <br/>[Common Data Model](format-common-data-model.md#sink-properties)<br>[Texte délimité](format-delimited-text.md#mapping-data-flow-properties) <br>[Delta](format-delta.md) <br>[JSON](format-json.md#mapping-data-flow-properties) <br/>[ORC](format-orc.md#mapping-data-flow-properties)<br/>[Parquet](format-parquet.md#mapping-data-flow-properties) | ✓/- <br>-/✓ <br>✓/- <br>-/✓ <br>✓/-<br>✓/✓ <br>✓/- |
| [Base de données Azure pour PostgreSQL](connector-azure-database-for-postgresql.md) |  | ✓/✓ |
| [Azure SQL Database](connector-azure-sql-database.md#mapping-data-flow-properties) | | ✓/- |
| [Azure SQL Managed Instance (préversion)](connector-azure-sql-managed-instance.md#mapping-data-flow-properties) | | ✓/- |
| [Azure Synapse Analytics](connector-azure-sql-data-warehouse.md#mapping-data-flow-properties) | | ✓/- |
| [Snowflake](connector-snowflake.md) | | ✓/✓ |

Les paramètres spécifiques à ces connecteurs se trouvent dans l’onglet **Paramètres**. Vous trouverez des informations et des exemples de scripts de flux de données concernant ces paramètres dans la documentation relative aux connecteurs.

Azure Data Factory a accès à plus de [90 connecteurs natifs](connector-overview.md). Pour écrire des données sur ces autres sources à partir de votre flux de données, utilisez l’activité de copie pour charger ces données à partir d’un récepteur pris en charge.

## <a name="sink-settings"></a>Paramètres de récepteur

Après avoir ajouté un récepteur, configurez-le via l'onglet **Récepteur**. Sous cet onglet, vous pouvez sélectionner ou créer le jeu de données dans lequel votre récepteur écrira. Les valeurs de développement des paramètres de jeux de données peuvent être configurées dans les [Paramètres de débogage](concepts-data-flow-debug-mode.md). (Le mode débogage doit être activé.)

La vidéo suivant explique un certain nombre d’options de récepteur pour les types de fichiers délimités par du texte,

> [!VIDEO https://www.microsoft.com/videoplayer/embed/RE4tf7T]

![Capture d’écran montrant les paramètres du récepteur.](media/data-flow/sink-settings.png "Capture d’écran montrant les paramètres du récepteur.")

**Dérive de schéma** : La [dérive de schéma](concepts-data-flow-schema-drift.md) est la capacité de Data Factory à gérer nativement des schémas flexibles dans vos flux de données sans avoir besoin de définir explicitement des changements de colonnes. Activez **Autoriser la dérive de schéma** pour écrire des colonnes supplémentaires, en plus de ce qui est défini dans le schéma de données du récepteur.

**Valider le schéma** : Si l’option Valider le schéma est sélectionnée, le flux de données échoue si une colonne du schéma source entrant est introuvable dans la projection source ou que les types de données ne correspondent pas. Utilisez ce paramètre pour que les données sources respectent le contrat de votre projection définie. Il est utile dans les scénarios de source de base de données pour signaler que les noms ou les types de colonne ont changé.

## <a name="cache-sink"></a>Récepteur de cache

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4HKt1]

Un *récepteur de cache* existe lorsqu’un flux de données écrit des données dans le cache Spark et non dans un magasin de données. Dans les flux de données de mappage, vous pouvez référencer ces données à la fois dans le même flux à l’aide d’une *recherche dans le cache*. Cela est utile lorsque vous souhaitez référencer des données dans le cadre d’une expression, mais que vous ne souhaitez pas explicitement y joindre les colonnes. La recherche d’une valeur maximale dans un magasin de données et la mise en correspondance des codes d’erreur avec une base de données de messages d’erreur sont des situations courantes dans lesquelles un récepteur de cache peut être utile. 

Pour écrire dans un récepteur de cache, ajoutez une transformation de récepteur et sélectionnez **Cache** comme type de récepteur. Contrairement à d’autres types de récepteurs, vous ne devez pas sélectionner un jeu de données ou un service lié, car vous n’écrivez pas dans un magasin externe. 

![Sélectionner le récepteur de cache](media/data-flow/select-cache-sink.png "Sélectionner le récepteur de cache")

Dans les paramètres du récepteur, vous pouvez éventuellement spécifier les colonnes clés du récepteur de cache. Elles sont utilisées comme conditions de correspondance lors de l’utilisation de la fonction `lookup()` dans une recherche dans le cache. Si vous spécifiez des colonnes clés, vous ne pouvez pas utiliser la fonction `outputs()` dans une recherche dans le cache. Pour en savoir plus sur la syntaxe de recherche dans le cache, consultez [recherches mises en cache](concepts-data-flow-expression-builder.md#cached-lookup).

![Colonnes clés du récepteur de cache](media/data-flow/cache-sink-key-columns.png "Colonnes clés du récepteur de cache")

Par exemple, si je spécifie une seule colonne clé de `column1` dans un récepteur de cache appelé `cacheExample`, l’appel de `cacheExample#lookup()` aurait un paramètre spécifiant la ligne à laquelle le récepteur de cache doit correspondre. La fonction génère une colonne complexe unique avec des sous-colonnes pour chaque colonne mappée.

> [!NOTE]
> Un récepteur de cache doit se trouver dans un flux de données complètement indépendant de toute transformation qui y fait référence par le biais d’une recherche dans le cache. Un récepteur de cache doit également être le premier récepteur écrit. 

## <a name="field-mapping"></a>Mappages de champs

Comme pour une transformation de sélection (Select), l'onglet **Mappage** du récepteur vous permet de choisir les colonnes entrantes dans lesquelles l'écriture interviendra. Par défaut, toutes les colonnes d'entrée, y compris les colonnes dérivées, sont mappées. Ce comportement est connu sous le nom de *mappage automatique*.

Lorsque vous désactivez le mappage automatique, vous pouvez ajouter des mappages basés sur des colonnes ou des mappages basés sur des règles. Avec le mappage basé sur des règles, vous pouvez écrire des expressions avec des critères spéciaux. Le mappage fixe mappe les noms de colonnes logiques et physiques. Pour plus d'informations sur le mappage basé sur des règles, consultez [Modèles de colonne dans le flux de données de mappage](concepts-data-flow-column-pattern.md#rule-based-mapping-in-select-and-sink).

## <a name="custom-sink-ordering"></a>Ordre des récepteurs personnalisé

Par défaut, les données sont écrites dans plusieurs récepteurs selon un ordre non déterministe. Le moteur d’exécution écrit les données en parallèle à l’issue de la logique de transformation et l’ordre des récepteurs peut varier d’une exécution à l’autre. Pour spécifier l’ordre exact des récepteurs, activez **Ordre des récepteurs personnalisé** sous l’onglet **Général** du flux de données. Quand cette option est activée, les récepteurs sont écrits séquentiellement dans l’ordre croissant.

![Capture d’écran montrant l’Ordre des récepteurs personnalisé.](media/data-flow/custom-sink-ordering.png "Capture d’écran montrant l’Ordre des récepteurs personnalisé.")

> [!NOTE]
> Lors de l’utilisation de [recherches mises en cache](./concepts-data-flow-expression-builder.md#cached-lookup), assurez-vous que l’ordre des récepteurs mis en cache est défini sur 1, le plus bas (ou le premier) dans le classement.

![Ordre des récepteurs personnalisé](media/data-flow/cache-2.png "Ordre des récepteurs personnalisé")

### <a name="sink-groups"></a>Groupes de récepteurs

Vous pouvez regrouper les récepteurs en appliquant le même numéro d’ordre pour une série de récepteurs. ADF traitera ces récepteurs comme des groupes qui peuvent s’exécuter en parallèle. Les options d’exécution en parallèle s’affichent dans l’activité de flux de données du pipeline.

## <a name="error-row-handling"></a>Gestion des lignes d’erreur

Lors de l’écriture dans des bases de données, certaines lignes de données peuvent échouer en raison de contraintes définies par la destination. Par défaut, l’exécution d’un flux de données échouera à la première erreur rencontrée. Dans certains connecteurs, vous pouvez choisir de **Continuer en cas d’erreur**, ce qui permet à votre flux de données de se terminer, même si des lignes individuelles comportent des erreurs. Actuellement, cette fonctionnalité n’est disponible que dans Azure SQL Database. Pour plus d’informations, consultez [Gestion des lignes d’erreurs dans Azure SQL Database](connector-azure-sql-database.md#error-row-handling).

Vous trouverez ci-dessous un didacticiel vidéo sur l’utilisation automatique de la gestion des lignes d’erreur de base de données dans votre transformation du récepteur.

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4IWne]

## <a name="data-preview-in-sink"></a>Aperçu des données dans le récepteur

Lors de la récupération (fetch) d'un aperçu des données sur un cluster de débogage, aucune donnée n'est écrite dans votre récepteur. Une capture instantanée des données est renvoyée, mais rien n'est écrit à l'emplacement de destination. Pour tester l'écriture de données dans votre récepteur, exécutez un débogage de pipeline à partir du canevas du pipeline.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez créé votre flux de données, ajoutez une [activité de flux de données à votre pipeline](concepts-data-flow-overview.md).