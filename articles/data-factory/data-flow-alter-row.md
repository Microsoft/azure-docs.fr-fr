---
title: Transformation de modification de ligne dans le flux de données de mappage
description: Guide pratique pour mettre à jour la cible de base de données à l’aide de la transformation de modification de ligne dans le flux de données de mappage
author: kromerm
ms.author: makromer
ms.reviewer: daperlov
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 05/06/2020
ms.openlocfilehash: c3858756a0140481c0ab249e29c95f76c4b90da5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "82982647"
---
# <a name="alter-row-transformation-in-mapping-data-flow"></a>Transformation de modification de ligne dans le flux de données de mappage

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Utiliser la transformation de Alter Row pour définir des stratégies insert, delete, update et upsert sur les lignes. Vous pouvez ajouter des conditions de type un-à-plusieurs en tant qu’expressions. Ces conditions doivent être spécifiées par ordre de priorité, car chaque ligne sera marquée avec la stratégie liée à la première correspondance. Chacune de ces conditions peut entraîner l'insertion, la mise à jour, la suppression ou l'upsert d'une ligne (ou de plusieurs lignes). Alter Row peut produire des actions DDL et DML sur votre base de données.

![Paramètres d’Alter Row](media/data-flow/alter-row1.png "Paramètres d’Alter Row")

Les transformations Alter Row ne fonctionnent que sur les récepteurs de base de données ou CosmosDB de votre flux de données. Les actions que vous affectez aux lignes (insert, update, delete, upsert) n’ont pas lieu au cours des sessions de débogage. Exécutez une activité Exécuter un flux de données dans un pipeline pour activer les stratégies de modification de ligne sur vos tables de base de données.

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4vJYc]

## <a name="specify-a-default-row-policy"></a>Spécifier une stratégie de ligne par défaut

Créez une transformation Alter Row et spécifiez une stratégie de ligne associée à la condition `true()`. Chaque ligne qui ne correspond à aucune des expressions définies précédemment sera marquée pour la stratégie de ligne spécifiée. Par défaut, chaque ligne qui ne correspond à aucune expression conditionnelle sera marquée pour `Insert`.

![Stratégie de modification de ligne](media/data-flow/alter-row4.png "Stratégie de modification de ligne")

> [!NOTE]
> Pour associer une stratégie à toutes les lignes, vous pouvez créer une condition pour cette stratégie et spécifier la condition comme `true()`.

## <a name="view-policies-in-data-preview"></a>Afficher les stratégies dans l’aperçu des données

Utilisez le [mode de débogage](concepts-data-flow-debug-mode.md) pour afficher les résultats de vos stratégies de modification de ligne dans le volet d’aperçu des données. Un aperçu des données d’une transformation de modification de ligne ne génèrera pas d’actions DDL ni DML sur votre cible.

![Stratégies Alter Row](media/data-flow/alter-row3.png "Stratégies Alter Row")

Chaque stratégie de modification de ligne est représentée par une icône qui indique si une action d’insertion, de mise à jour, upsert ou de suppression se produira. L’en-tête supérieur indique le nombre de lignes affectées par chaque stratégie dans l’aperçu.

## <a name="allow-alter-row-policies-in-sink"></a>Autoriser les stratégies de modification de ligne dans le récepteur

Pour que les stratégies de modification de ligne fonctionnent, le flux de données doit écrire dans une base de données ou un récepteur Cosmos. Dans l’onglet **Paramètres** de votre récepteur, activez les stratégies de modification de ligne autorisées pour ce récepteur.

![Récepteur Alter Row](media/data-flow/alter-row2.png "Récepteur Alter Row")

Le comportement par défaut est d’autoriser uniquement les insertions. Pour autoriser les mises à jour, les opérations upsert ou les suppressions, cochez la case dans le récepteur correspondant à cette condition. Si les mises à jour, les opérations upsert ou les suppressions sont activées, vous devez spécifier les colonnes clés du récepteur sur lesquelles effectuer la correspondance.

> [!NOTE]
> Si vos insertions, mises à jour ou opérations upsert modifient le schéma de la table cible du récepteur, le flux de données échoue. Pour modifier le schéma cible dans votre base de données, choisissez **Recréer la table** en tant qu’action de table. Cela supprime et recrée votre table selon la nouvelle définition de schéma.

La transformation du récepteur requiert une clé unique ou une série de clés pour l’identification de ligne unique dans votre base de données cible. Pour les récepteurs SQL, définissez les clés sous l’onglet Paramètres du récepteur. Pour CosmosDB, définissez la clé de partition dans les paramètres et définissez également le champ système CosmosDB « ID » dans votre mappage de récepteur. Pour CosmosDB, il est obligatoire d’inclure la colonne système « ID » pour les mises à jour, les opérations upserts et les suppressions.

## <a name="merges-and-upserts-with-azure-sql-database-and-synapse"></a>Fusionne et effectue une opération upsert avec Azure SQL Database et Synapse

Les Data Flows ADF prennent en charge les fusions par rapport au pool de bases de données Azure SQL Database et Synapse (entrepôt de données) avec l’option upsert.

Toutefois, vous pouvez rencontrer des scénarios dans lesquels votre schéma de base de données cible utilisait la propriété d’identité des colonnes clés. ADF vous oblige à identifier les clés que vous allez utiliser pour faire correspondre les valeurs de ligne des mises à jour et des opérations upsert. Toutefois, si la propriété d’identité est définie pour la colonne cible et que vous utilisez la stratégie upsert, la base de données cible ne vous autorisera pas à écrire dans la colonne. Vous pouvez également rencontrer des erreurs lorsque vous essayez d’utiliser une opération upsert par rapport à la colonne de distribution d’une table distribuée.

Voici comment résoudre ce problème :

1. Accédez aux paramètres de transformation du récepteur et définissez « Ignorer l’écriture des colonnes clés ». Cela indique à ADF de ne pas écrire la colonne que vous avez sélectionnée comme valeur de clé pour votre mappage.

2. Si cette colonne clé n’est pas la colonne qui est à l’origine du problème pour les colonnes d’identité, vous pouvez utiliser l’option SQL de prétraitement de transformation du récepteur : ```SET IDENTITY_INSERT tbl_content ON```. Ensuite, désactivez-la à l’aide de la propriété SQL postérieure au traitement : ```SET IDENTITY_INSERT tbl_content OFF```.

3. Pour le cas d’identité et le cas de colonne de distribution, vous pouvez passer de la logique de l’opération upsert à l’utilisation d’une condition de mise à jour distincte et d’une condition d’insertion distincte à l’aide d’une transformation de fractionnement conditionnel. De cette façon, vous pouvez définir le mappage sur le chemin d’accès de la mise à jour pour ignorer le mappage de la colonne clé.

## <a name="data-flow-script"></a>Script de flux de données

### <a name="syntax"></a>Syntaxe

```
<incomingStream>
    alterRow(
           insertIf(<condition>?),
           updateIf(<condition>?),
           deleteIf(<condition>?),
           upsertIf(<condition>?),
        ) ~> <alterRowTransformationName>
```

### <a name="example"></a>Exemple

L’exemple ci-dessous est une transformation de modification de ligne nommée `CleanData` qui prend un flux entrant `SpecifyUpsertConditions` et crée trois conditions de modification de ligne. Dans la transformation précédente, une colonne nommée `alterRowCondition` est calculée pour déterminer si une ligne est ou n’est pas insérée, mise à jour ou supprimée dans la base de données. Si la valeur de la colonne a une valeur de chaîne qui correspond à la règle de modification de ligne, cette stratégie lui est affectée.

Dans l’expérience utilisateur Data Factory, cette transformation se présente comme dans l’image ci-dessous :

![Exemple de modification de ligne](media/data-flow/alter-row4.png "Exemple de modification de ligne")

Le script de flux de données pour cette transformation se trouve dans l’extrait de code ci-dessous :

```
SpecifyUpsertConditions alterRow(insertIf(alterRowCondition == 'insert'),
    updateIf(alterRowCondition == 'update'),
    deleteIf(alterRowCondition == 'delete')) ~> AlterRow
```

## <a name="next-steps"></a>Étapes suivantes

Une fois la transformation Alter Row effectuée, vous souhaiterez peut-être [transférer vos données vers un magasin de données de destination](data-flow-sink.md).
