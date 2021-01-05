---
title: Guide de programmabilité U-SQL UDO pour Azure Data Lake
description: En savoir plus sur la programmabilité U-SQL UDO pour Azure Data Lake Analytics afin de créer un script USQL efficace.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 06/30/2017
ms.openlocfilehash: 02360c68e5e830ceee69075fd5532b126d85bec2
ms.sourcegitcommit: df66dff4e34a0b7780cba503bb141d6b72335a96
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96512434"
---
# <a name="u-sql-user-defined-objects-overview"></a>Vue d’ensemble des objets définis par l’utilisateur U-SQL


## <a name="u-sql-user-defined-objects-udo"></a>U-SQL : objets définis par l'utilisateur : UDO
U-SQL vous permet de définir des objets de programmabilité personnalisés, appelés objets définis par l’utilisateur (ou UDO).

Voici une liste d’UDO dans U-SQL :

* Extracteurs définis par l’utilisateur
    * Extraient ligne par ligne
    * Utilisés pour implémenter une extraction de données à partir de fichiers structurés personnalisés

* Générateurs de sortie définis par l’utilisateur
    * Sortent ligne par ligne
    * Utilisés pour sortir des types de données personnalisées ou des formats de fichiers personnalisés

* Processeurs définis par l’utilisateur
    * Prennent une ligne et produisent une ligne
    * Utilisés pour réduire le nombre de colonnes ou produire de nouvelles colonnes avec des valeurs dérivées d’un ensemble de colonnes existant

* Applicateurs définis par l’utilisateur
    * Prennent une ligne et produisent de 0 à n lignes
    * Utilisés avec OUTER/CROSS APPLY

* Combinateurs définis par l’utilisateur
    * Combinent des ensembles de lignes et des jointures définies par l’utilisateur

* Réducteurs définis par l’utilisateur
    * Prennent n lignes et produisent une ligne
    * Utilisés pour réduire le nombre de lignes

UDO est généralement appelé explicitement dans un script U-SQL dans le cadre des instructions U-SQL suivantes :

* EXTRACT
* OUTPUT
* PROCESS
* COMBINE
* REDUCE

> [!NOTE]  
> La consommation des UDO est limitée à 0,5 Go de mémoire.  La limite de mémoire ne s’applique pas aux exécutions locales.

## <a name="next-steps"></a>Étapes suivantes
* [Guide de programmabilité U-SQL - vue d’ensemble](data-lake-analytics-u-sql-programmability-guide.md)
* [Guide de programmabilité U-SQL - UDT et UDAGG](data-lake-analytics-u-sql-programmability-guide-UDT-AGG.md)