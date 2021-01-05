---
title: Comment cataloguer des données volumineuses (Big Data) dans Azure Data Catalog ?
description: Article de procédure expliquant comment utiliser Azure Data Catalog avec des sources de données « volumineuses », notamment le Stockage Blob Azure, Azure Data Lake et les fichiers Hadoop HDFS.
author: JasonWHowell
ms.author: jasonh
ms.service: data-catalog
ms.topic: how-to
ms.date: 08/01/2019
ms.openlocfilehash: e79e5e16b300fd02b9c9124f9677747834f22813
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96009430"
---
# <a name="how-to-catalog-big-data-in-azure-data-catalog"></a>Comment cataloguer des données volumineuses (Big Data) dans Azure Data Catalog ?

## <a name="introduction"></a>Introduction

**Microsoft Azure Data Catalog** est un service cloud entièrement géré qui sert de système d'inscription et de détection des sources de données d'entreprise. Il vise essentiellement à aider les utilisateurs à détecter, comprendre et utiliser des sources de données, et à permettre aux organisations de mieux exploiter leurs sources données existantes, y compris le Big Data.

**Azure Data Catalog** prend en charge l’inscription de blobs et de répertoires Stockage Azure ainsi que des fichiers et des répertoires HDFS Hadoop. La nature semi-structurée des sources de données offre une grande flexibilité. Toutefois, pour réellement profiter de leur enregistrement dans **Azure Data Catalog**, les utilisateurs doivent comprendre comment sont organisées les sources de données.

## <a name="directories-as-logical-data-sets"></a>Répertoires sous forme de jeux de données logiques

Un modèle répandu d’organisation de source de données volumineuses consiste à traiter des répertoires sous forme de jeux de données logique. Des répertoires de niveau supérieur sont utilisés pour définir un jeu de données, les sous-dossiers définissent des partitions, et les fichiers qu’ils contiennent stockent les données elles-mêmes.

Voici quelques exemples de ce modèle :

```text
    \vehicle_maintenance_events
        \2013
        \2014
        \2015
            \01
                \2015-01-trailer01.csv
                \2015-01-trailer92.csv
                \2015-01-canister9635.csv
                ...
    \location_tracking_events
        \2013
        ...
```

Dans cet exemple, vehicle_maintenance_events et location_tracking_events représentent les jeux de données logiques. Chacun de ces dossiers contient des fichiers de données organisés par année et par mois en sous-dossiers. Chacun de ces dossiers peut contenir des centaines ou des milliers de fichiers.

Dans ce modèle, l’enregistrement des fichiers individuels auprès d’ **Azure Data Catalog** ne sert sans doute à rien. Au lieu de cela, enregistrez les répertoires qui représentent les jeux de données significatifs pour les utilisateurs travaillant avec ces données.

## <a name="reference-data-files"></a>Référence de fichiers de données

Un modèle complémentaire consiste à stocker des jeux de données de référence sous forme de fichiers individuels. Ces ensembles de données peuvent être considérés comme le côté « émergé » des données volumineuses et sont souvent comparables aux dimensions d’un modèle de données analytiques. Les fichiers de données de référence contiennent des enregistrements utilisés pour offrir un contexte aux lots de fichiers de données stockées ailleurs dans le magasin de données volumineuses.

Voici quelques exemples de ce modèle :

```text
    \vehicles.csv
    \maintenance_facilities.csv
    \maintenance_types.csv
```

Lorsqu’un analyste ou un spécialiste des données travaille avec les données contenues dans des structures de répertoire plus grandes, les données présentes dans ces fichiers de référence fournissent des informations plus détaillées pour les entités référencées uniquement par nom ou ID du jeu de données plus grand.

Dans ce modèle, il est judicieux d’enregistrer les fichiers de données de référence avec **Azure Data Catalog**. Chaque fichier représente un jeu de données, et chacun d’eux peut être annoté et découvert individuellement.

## <a name="alternate-patterns"></a>Modèles alternatifs

Les modèles décrits dans la section précédente représentent deux organisations possibles de magasins de données Big Data, mais chaque implémentation est différente. Quelle que soit la façon dont vos sources de données sont structurées, lors de l’enregistrement des sources de données Big Data auprès de **Azure Data Catalog**, concentrez-vous sur l’enregistrement des fichiers et des répertoires qui représentent les jeux de données qui ont une valeur pour les autres au sein de votre organisation. L’inscription de tous les fichiers et répertoires peut encombrer le catalogue, ce qui complique les opérations de recherche pour les utilisateurs.

## <a name="summary"></a>Résumé

L’inscription des sources de données auprès de **Azure Data Catalog** les facilite leur détection et leur compréhension. En enregistrant et en annotant les fichiers et les répertoires de données volumineuses qui représentent les jeux de données logiques, vous pouvez aider les utilisateurs à trouver et à utiliser les sources de données volumineuses dont ils ont besoin.
