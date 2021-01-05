---
title: Déduplication de lignes et recherche de valeurs Null à l’aide d’extraits de flux de données
description: Découvrez comment dédupliquer facilement des lignes et rechercher des valeurs Null en utilisant des extraits de code dans des flux de données
services: data-factory
author: kromerm
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 09/30/2020
ms.author: makromer
ms.openlocfilehash: cdb522cc4be83eadd2c60c91c7fee33e7ccc039b
ms.sourcegitcommit: fb3c846de147cc2e3515cd8219d8c84790e3a442
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92632445"
---
# <a name="dedupe-rows-and-find-nulls-by-using-data-flow-snippets"></a>Déduplication de lignes et recherche de valeurs Null à l’aide d’extraits de flux de données

[!INCLUDE [appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En utilisant des extraits de code dans des flux de données de mappage, vous pouvez facilement effectuer des tâches courantes telles que la déduplication des données et le filtrage de valeurs Null. Cet article explique comment ajouter facilement ces fonctions à vos pipelines à l’aide d’extraits de script de flux de données.
<br>
> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4GnhH]

## <a name="create-a-pipeline"></a>Créer un pipeline

1. Sélectionnez **Nouveau pipeline**.

1. Ajoutez une activité de flux de données.

1. Sélectionnez l’onglet **Paramètres de la source** , ajoutez une transformation source, puis connectez-la à l’un de vos jeux de données.

    ![Capture d’écran du volet « Paramètres de la source » pour l’ajout d’un type de source.](media/data-flow/snippet-adf-2.png)

    Les extraits de code de vérification de la déduplication et de la valeur Null utilisent des modèles génériques qui tirent parti de la dérive du schéma de flux de données. Les extraits de code fonctionnent avec n’importe quel schéma de votre jeu de données, ou avec des jeux de données qui n’ont pas de schéma prédéfini.

1. Dans la section « Ligne distincte utilisant toutes les colonnes » de [Script de flux de données (DFS)](./data-flow-script.md#distinct-row-using-all-columns), copiez l’extrait de code pour DistinctRows.

1. [Accédez à la page de documentation du script de flux de données et copiez l’extrait de code pour les lignes distinctes.](./data-flow-script.md#distinct-row-using-all-columns)

    ![Capture d’écran d’un extrait de code source.](media/data-flow/snippet-adf-3.png)

1. Dans votre script, après la définition de `source1`, appuyez sur Entrée, puis collez l’extrait de code.

1. Effectuez l'une des opérations suivantes :

   * Connectez cet extrait de code collé à la transformation source que vous avez créée précédemment dans le graphique en saisissant **source1** devant le code collé.

   * Vous pouvez également connecter la nouvelle transformation dans le concepteur en sélectionnant le flux entrant à partir du nouveau nœud de transformation dans le graphique.

     ![Capture d’écran du volet « Paramètres de fractionnement conditionnel ».](media/data-flow/snippet-adf-4.png)

   À présent, votre flux de données supprime les doublons de lignes de votre source à l’aide de la transformation d’agrégation, qui regroupe toutes les lignes à l’aide d’un hachage général pour toutes les valeurs de colonne.
    
1. Ajoutez un extrait de code pour fractionner vos données en un flux qui contient des lignes avec des valeurs Null et un autre flux sans valeurs Null. Pour ce faire :

1. [Revenez à la bibliothèque d’extraits de code et cette fois copiez le code pour les contrôles de valeur Null.](./data-flow-script.md#check-for-nulls-in-all-columns)

   b. Dans le concepteur de flux de données, sélectionnez à nouveau **Script** , puis collez ce nouveau code de transformation en bas. Cette action connecte le script à votre transformation précédente en plaçant le nom de ladite transformation devant l’extrait de code collé.

   Votre graphique de flux de données doit maintenant ressembler à ceci :

    ![Capture d’écran du graphique de flux de données.](media/data-flow/snippet-adf-1.png)

  Vous avez maintenant créé un flux de données fonctionnel avec des déduplications génériques et des contrôles de valeur Null en récupérant les extraits de code existants de la bibliothèque de scripts de flux de données et en les ajoutant à votre conception existante.

## <a name="next-steps"></a>Étapes suivantes

* Créez le reste de votre logique de flux de données à l’aide de [transformations](concepts-data-flow-overview.md) de flux de données de mappage.