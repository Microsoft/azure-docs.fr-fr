---
title: Vue d’ensemble de la transformation de flux de données de mappage
description: Vue d’ensemble des différentes transformations disponibles dans le flux de données de mappage
author: dcstwh
ms.author: weetok
manager: anandsub
ms.service: data-factory
ms.topic: conceptual
ms.date: 10/27/2020
ms.openlocfilehash: 9d44890e84e97a413543a4291d1331fee0f04841
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96490888"
---
# <a name="mapping-data-flow-transformation-overview"></a>Vue d’ensemble de la transformation de flux de données de mappage

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)] 

Vous trouverez ci-dessous la liste des transformations actuellement prises en charge dans le flux de données de mappage. Cliquez sur chaque transformation pour découvrir ses détails de configuration.

| Name | Category | Description |
| ---- | -------- | ----------- |
| [Agrégat](data-flow-aggregate.md) | Modificateur de schéma | Définissez les différents types d’agrégations telles que SUM, MIN, MAX et COUNT regroupées par colonnes calculées ou existantes. | 
| [Modification de ligne](data-flow-alter-row.md) | Modificateur de ligne | Définissez des stratégies d’insertion, de suppression, de mise à jour et d’upsert sur des lignes. |
| [Fractionnement conditionnel](data-flow-conditional-split.md) | Entrées/sorties multiples | Routez des lignes de données vers différents flux en fonction de conditions de correspondance. |
| [Colonne dérivée](data-flow-derived-column.md) | Modificateur de schéma | Générez de nouvelles colonnes ou modifiez des champs existants à l’aide du langage d’expression des flux de données. | 
| [Exists](data-flow-exists.md) | Entrées/sorties multiples | Vérifiez si vos données existent dans une autre source ou un autre flux. | 
| [Filter](data-flow-filter.md) | Modificateur de ligne | Filtrez une ligne en fonction d’une condition. |
| [Aplatir](data-flow-flatten.md) | Modificateur de schéma |  Prenez des valeurs de tableau à l’intérieur de structures hiérarchiques, par exemple au format JSON, et déroulez-les en lignes individuelles. |
| [Join](data-flow-join.md) | Entrées/sorties multiples |  Combinez les données de deux sources ou flux. |
| [Lookup](data-flow-lookup.md) | Entrées/sorties multiples | Référencez des données à partir d’une autre source. |
| [Nouvelle branche](data-flow-new-branch.md) | Entrées/sorties multiples | Appliquez plusieurs ensembles d’opérations et de transformations au même flux de données. |
| [Tableau croisé dynamique](data-flow-pivot.md) | Modificateur de schéma | Agrégation où une ou plusieurs colonnes de regroupement voient leurs valeurs de lignes distinctes transformées en colonnes individuelles. |
| [Rank](data-flow-rank.md) | Modificateur de schéma | Générez un classement ordonné en fonction des conditions de tri |
| [Select](data-flow-select.md) | Modificateur de schéma | Créez des alias pour les colonnes et les noms de flux, et supprimez ou réorganisez les colonnes |
| [Section sink](data-flow-sink.md) | - | Destination finale pour vos données |
| [Sort](data-flow-sort.md) | Modificateur de ligne | Triez les lignes entrantes sur le flux de données actuel. |
| [Source](data-flow-source.md) | - | Source de données pour le flux de données |
| [Clé de substitution](data-flow-surrogate-key.md) | Modificateur de schéma | Ajoutez une valeur de clé arbitraire non professionnelle d’incrémentation. |
| [Union](data-flow-union.md) | Entrées/sorties multiples | Combinez plusieurs flux de données verticalement. |
| [Supprimer le tableau croisé dynamique](data-flow-unpivot.md) | Modificateur de schéma | Créer un tableau croisé dynamique pour convertir des colonnes en valeurs de ligne |
| [Window](data-flow-window.md) | Modificateur de schéma |  Définissez des agrégations de colonnes basées sur des fenêtres dans vos flux de données. |
